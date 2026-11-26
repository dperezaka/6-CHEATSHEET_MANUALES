# Manual de uso: mypy

> **Categoría:** Comprobación de tipos · **Utilidad:** Verifica los tipos sin ejecutar el código
> **Tipo:** Paquete externo (requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos](#4-primeros-pasos)
5. [Cómo leer los errores](#5-cómo-leer-los-errores)
6. [Configuración](#6-configuración)
7. [Modo estricto y opciones importantes](#7-modo-estricto-y-opciones-importantes)
8. [Configuración por módulo](#8-configuración-por-módulo)
9. [Librerías sin tipos e importaciones](#9-librerías-sin-tipos-e-importaciones)
10. [Silenciar errores de forma controlada](#10-silenciar-errores-de-forma-controlada)
11. [Herramientas de depuración](#11-herramientas-de-depuración)
12. [Plugins](#12-plugins)
13. [Adopción gradual en un proyecto existente](#13-adopción-gradual-en-un-proyecto-existente)
14. [Rendimiento: caché y demonio](#14-rendimiento-caché-y-demonio)
15. [Integración con editores, pre-commit y CI](#15-integración-con-editores-pre-commit-y-ci)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**mypy** es el comprobador de tipos estático de referencia para Python. Lee las anotaciones de tipo (*type hints*) y analiza el código **sin ejecutarlo** para detectar errores como llamar a un método inexistente, pasar un `str` donde se espera un `int` o olvidar comprobar un `None`.

```python
def iva(base: float) -> float:
    return base * 0.21

iva("100")      # mypy: Argument 1 to "iva" has incompatible type "str"; expected "float"
```

**Ventajas:**
- Proyecto de referencia, desarrollado junto a la especificación de tipos de Python.
- Muy configurable: permite adoptar los tipos módulo a módulo.
- Sistema de *plugins* para librerías dinámicas (Pydantic, Django, SQLAlchemy).
- Amplia documentación y comunidad.

**Limitaciones:**
- Más lento que Pyright y ty en proyectos grandes (mitigado por la caché y el demonio `dmypy`).
- Por defecto no comprueba el cuerpo de funciones sin anotar.

| Si necesita… | Opción |
|---|---|
| Comprobador de referencia, con *plugins* y adopción gradual | **mypy** |
| Más velocidad e integración con VS Code (Pylance) | Pyright |
| Velocidad máxima, ecosistema Astral (uv, Ruff) | ty (en beta) |

---

## 2. Instalación

Instálelo como dependencia de desarrollo del proyecto (para que todo el equipo use la misma versión):

```bash
uv add --dev mypy
poetry add --group dev mypy
python -m pip install mypy
```

Comprobar:

```bash
mypy --version
```

> mypy debe ejecutarse en el **mismo entorno** donde están instaladas las dependencias del proyecto, para poder leer sus tipos.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Análisis estático | Comprobación sin ejecutar el código. |
| Función anotada | Tiene al menos una anotación; mypy comprueba su cuerpo. |
| Función sin anotar | mypy no comprueba su cuerpo (salvo `check_untyped_defs`). |
| `Any` implícito | Tipo asignado a lo que no está anotado; desactiva la comprobación. |
| Código de error | Identificador del tipo de error: `[arg-type]`, `[attr-defined]`… |
| Stub | Archivo `.pyi` con las firmas tipadas de un módulo. |
| Inferencia | mypy deduce tipos de variables a partir de su valor inicial. |
| Modo estricto | Conjunto de opciones que exige anotaciones completas y prohíbe `Any` implícito. |

---

## 4. Primeros pasos

```bash
mypy archivo.py
mypy src/                         # Una carpeta
mypy src/ tests/
mypy -p mi_paquete                # Un paquete por nombre
mypy -m mi_paquete.modulo         # Un módulo
mypy                              # Usa «files» de la configuración
mypy --strict src/
```

Ejemplo:

```python
# inventario.py
from dataclasses import dataclass


@dataclass
class Producto:
    nombre: str
    stock: int


def buscar(productos: list[Producto], nombre: str) -> Producto | None:
    for p in productos:
        if p.nombre == nombre:
            return p
    return None


def reponer(productos: list[Producto], nombre: str, unidades: int) -> int:
    p = buscar(productos, nombre)
    p.stock += unidades           # ← Error: p puede ser None
    return p.stock
```

```text
$ mypy inventario.py
inventario.py:19: error: Item "None" of "Producto | None" has no attribute "stock"  [union-attr]
inventario.py:20: error: Item "None" of "Producto | None" has no attribute "stock"  [union-attr]
Found 2 errors in 1 file (checked 1 source file)
```

Corrección:

```python
def reponer(productos: list[Producto], nombre: str, unidades: int) -> int:
    p = buscar(productos, nombre)
    if p is None:
        raise KeyError(f"Producto no encontrado: {nombre}")
    p.stock += unidades
    return p.stock
```

---

## 5. Cómo leer los errores

```text
inventario.py:19: error: Item "None" of "Producto | None" has no attribute "stock"  [union-attr]
└─ archivo ──┘└┘ └ nivel ┘└──────────────── mensaje ────────────────────────────┘ └ código ─┘
            línea
```

Códigos de error más frecuentes:

| Código | Significado | Ejemplo |
|---|---|---|
| `arg-type` | Argumento de tipo incorrecto. | `iva("100")` |
| `return-value` | El retorno no coincide con la anotación. | `-> int` devolviendo `str` |
| `assignment` | Asignación incompatible. | `x: int = "a"` |
| `union-attr` | Atributo sobre una unión que incluye `None` u otro tipo. | `p.stock` con `p: Producto | None` |
| `attr-defined` | Atributo o método inexistente. | `lista.push(1)` |
| `call-arg` | Número o nombre de argumentos incorrecto. | `f(1, 2, 3)` con dos parámetros |
| `index` | Índice no válido. | `d[1]` con `d: dict[str, int]` |
| `operator` | Operación no admitida. | `"a" + 1` |
| `no-untyped-def` | Función sin anotaciones (modo estricto). | `def f(x):` |
| `import-untyped` | Librería instalada sin tipos. | `import requests` sin `types-requests` |
| `import-not-found` | Módulo no encontrado. | Error de nombre o de entorno |
| `override` | Firma incompatible con la clase base. | Subclase que cambia tipos de parámetros |
| `misc` | Otros errores. | — |

Opciones para mejorar la salida:

```bash
mypy --pretty src/                  # Muestra el fragmento de código con el error marcado
mypy --show-error-context src/      # Indica en qué función/clase está
mypy --show-column-numbers src/
mypy --show-error-code-links src/   # Enlace a la documentación de cada código
mypy -O json src/                   # Salida JSON (integraciones)
```

---

## 6. Configuración

mypy lee la configuración de `pyproject.toml` (`[tool.mypy]`), `mypy.ini`, `.mypy.ini` o `setup.cfg`.

```toml
[tool.mypy]
python_version = "3.12"
files = ["src", "tests"]
strict = true
pretty = true
show_error_context = true
warn_unreachable = true
enable_error_code = ["ignore-without-code", "redundant-expr", "truthy-bool", "possibly-undefined"]
plugins = ["pydantic.mypy"]
exclude = ["^build/", "^docs/"]

[[tool.mypy.overrides]]
module = ["tests.*"]
disallow_untyped_defs = false

[[tool.mypy.overrides]]
module = ["weasyprint.*", "reportlab.*"]
ignore_missing_imports = true
```

> En la línea de comandos las opciones usan guiones (`--warn-unreachable`); en la configuración, guiones bajos (`warn_unreachable`).

---

## 7. Modo estricto y opciones importantes

`strict = true` activa, entre otras:

| Opción | Efecto |
|---|---|
| `disallow_untyped_defs` | Toda función debe estar anotada. |
| `disallow_incomplete_defs` | No se admiten anotaciones parciales. |
| `check_untyped_defs` | Comprueba también el cuerpo de funciones sin anotar. |
| `disallow_any_generics` | Prohíbe `list` sin parámetro (exige `list[int]`). |
| `disallow_untyped_calls` | Prohíbe llamar a funciones sin anotar desde código anotado. |
| `disallow_untyped_decorators` | Los decoradores deben estar tipados. |
| `no_implicit_reexport` | Un módulo solo reexporta lo que declare (`__all__` o `import x as x`). |
| `warn_return_any` | Avisa al devolver `Any` desde una función tipada. |
| `warn_unused_ignores` | Avisa de `# type: ignore` que ya no son necesarios. |
| `warn_redundant_casts` | Avisa de `cast()` innecesarios. |
| `strict_equality` | Detecta comparaciones imposibles (`"a" == 1`). |
| `extra_checks` | Comprobaciones adicionales de seguridad. |

Opciones adicionales recomendadas (no incluidas en `strict`):

| Opción | Efecto |
|---|---|
| `warn_unreachable` | Detecta código que nunca se ejecuta. |
| `enable_error_code = ["ignore-without-code"]` | Obliga a indicar el código en `# type: ignore[...]`. |
| `enable_error_code = ["possibly-undefined"]` | Variables que podrían no estar definidas. |
| `enable_error_code = ["truthy-bool"]` | Comprobaciones `if objeto:` sobre objetos siempre verdaderos. |
| `enable_error_code = ["explicit-override"]` | Exige `@override` al redefinir métodos. |
| `disallow_any_explicit` | Prohíbe escribir `Any` (muy estricto). |

---

## 8. Configuración por módulo

Las secciones `[[tool.mypy.overrides]]` aplican opciones a módulos concretos (admiten comodines):

```toml
[[tool.mypy.overrides]]
module = ["mi_app.legado.*"]
ignore_errors = true                  # Código heredado que aún no se revisa

[[tool.mypy.overrides]]
module = ["mi_app.api.*", "mi_app.dominio.*"]
disallow_any_explicit = true          # Núcleo más estricto

[[tool.mypy.overrides]]
module = ["tests.*"]
disallow_untyped_defs = false
```

Comentarios en línea al inicio de un archivo:

```python
# mypy: disable-error-code="no-untyped-def"
# mypy: warn-unreachable
```

---

## 9. Librerías sin tipos e importaciones

### 9.1 Diagnóstico

```text
error: Library stubs not installed for "requests"  [import-untyped]
note: Hint: "python3 -m pip install types-requests"
```

### 9.2 Soluciones, por orden de preferencia

1. **Actualizar la librería**: muchas incluyen ya `py.typed`.
2. **Instalar los stubs** de typeshed:

   ```bash
   uv add --dev types-requests types-PyYAML
   mypy --install-types --non-interactive     # Instala automáticamente los que falten
   ```

3. **Escribir stubs propios** en una carpeta y apuntar a ella:

   ```toml
   [tool.mypy]
   mypy_path = "stubs"
   ```

4. **Ignorar** las importaciones de esa librería concreta (nunca de forma global):

   ```toml
   [[tool.mypy.overrides]]
   module = ["libreria_antigua.*"]
   ignore_missing_imports = true
   ```

### 9.3 Estructura `src/` y rutas

Con la estructura `src/`, instale el proyecto en modo editable (`uv sync`, `pip install -e .`) o indique la ruta:

```toml
[tool.mypy]
mypy_path = "src"
explicit_package_bases = true
```

### 9.4 Comprobar con otro entorno

```bash
mypy --python-executable .venv/bin/python src/
```

---

## 10. Silenciar errores de forma controlada

```python
resultado = libreria.funcion()  # type: ignore[no-untyped-call]
valor = config["puerto"]        # type: ignore[index]  # Validado en cargar_config()
```

> **Siempre** indique el código entre corchetes y, si no es evidente, el motivo. Active `warn_unused_ignores` para eliminar los que dejen de ser necesarios.

Otras herramientas:

```python
from typing import cast, TYPE_CHECKING

datos = cast(dict[str, int], json.loads(texto))   # Afirmación sin comprobación real

if TYPE_CHECKING:
    ...                                            # Solo lo analiza mypy
else:
    ...                                            # Solo se ejecuta
```

Excluir una función del análisis:

```python
from typing import no_type_check


@no_type_check
def magia_dinamica(*args, **kwargs): ...
```

---

## 11. Herramientas de depuración

### 11.1 `reveal_type` y `reveal_locals`

Muestran el tipo que mypy ha inferido (no hace falta importarlos para mypy; en ejecución, `typing.reveal_type` existe desde 3.11):

```python
from typing import reveal_type

datos = {"a": [1, 2], "b": [3]}
reveal_type(datos)          # note: Revealed type is "dict[str, list[int]]"

def f(x: int | str) -> None:
    if isinstance(x, int):
        reveal_type(x)      # note: Revealed type is "int"
    reveal_locals()
```

### 11.2 Comprobar un fragmento rápidamente

```bash
mypy -c "x: int = 'a'"
```

### 11.3 Informes de cobertura de tipos

```bash
uv add --dev lxml
mypy src/ --html-report informe_tipos/       # Porcentaje de líneas tipadas por módulo
mypy src/ --txt-report informe_tipos/
mypy src/ --any-exprs-report informe_tipos/  # Expresiones de tipo Any
```

### 11.4 Verificar stubs frente al código real

```bash
python -m mypy.stubtest mi_paquete          # Compara los .pyi con el módulo en ejecución
```

---

## 12. Plugins

Algunas librerías generan atributos dinámicamente; sus *plugins* enseñan a mypy a entenderlas:

| Librería | Configuración |
|---|---|
| Pydantic | `plugins = ["pydantic.mypy"]` |
| Django | `uv add --dev django-stubs[compatible-mypy]` y `plugins = ["mypy_django_plugin.main"]` |
| SQLAlchemy 2 | No necesita *plugin*: usa `Mapped[...]` tipado. |
| NumPy | `numpy.typing` incluido; sin *plugin* en versiones recientes. |

Configuración del *plugin* de Pydantic:

```toml
[tool.mypy]
plugins = ["pydantic.mypy"]

[tool.pydantic-mypy]
init_forbid_extra = true
init_typed = true
warn_required_dynamic_aliases = true
```

---

## 13. Adopción gradual en un proyecto existente

Estrategia recomendada para un proyecto grande sin tipos:

| Fase | Acción |
|---|---|
| 1 | Configuración básica sin `strict`; corregir errores reales (`mypy src/`). |
| 2 | Añadir anotaciones a las funciones públicas de los módulos más usados. |
| 3 | Activar `strict` y desactivarlo por módulo con `overrides` para lo pendiente. |
| 4 | Ir eliminando módulos de la lista de excepciones. Prohibir excepciones nuevas en CI. |
| 5 | Añadir `warn_unreachable`, `ignore-without-code` y demás comprobaciones extra. |

Configuración típica de la fase 3:

```toml
[tool.mypy]
strict = true
files = ["src"]

# Módulos pendientes de tipar. ¡Esta lista solo debe disminuir!
[[tool.mypy.overrides]]
module = [
    "mi_app.informes.*",
    "mi_app.importadores.excel",
    "mi_app.utilidades.fechas",
]
disallow_untyped_defs = false
disallow_untyped_calls = false
disallow_incomplete_defs = false
```

Herramientas que ayudan a añadir anotaciones:

```bash
uvx autotyping --safe src/          # Añade anotaciones evidentes (-> None, -> bool…)
uvx monkeytype run script.py        # Infiere tipos ejecutando el código
uvx monkeytype apply mi_app.modulo
```

---

## 14. Rendimiento: caché y demonio

mypy guarda resultados en `.mypy_cache/` y solo vuelve a analizar lo que cambia. Añádalo a `.gitignore`.

```bash
mypy --cache-dir=/tmp/mypy-cache src/
mypy --sqlite-cache src/                 # Caché en SQLite (más eficiente en proyectos grandes)
mypy --no-incremental src/               # Desactivar la caché
```

Demonio para respuestas casi instantáneas en proyectos grandes:

```bash
dmypy run -- src/          # Arranca el demonio y comprueba
dmypy check src/           # Siguientes comprobaciones (muy rápidas)
dmypy status
dmypy stop
```

Compilado con *mypyc*, las *wheels* de PyPI ya son nativas y mucho más rápidas que la versión en Python puro.

---

## 15. Integración con editores, pre-commit y CI

### 15.1 VS Code

Extensión oficial **Mypy Type Checker** (`ms-python.mypy-type-checker`):

```json
{
  "mypy-type-checker.args": ["--config-file=pyproject.toml"],
  "mypy-type-checker.importStrategy": "fromEnvironment"
}
```

### 15.2 pre-commit

Como los *hooks* de pre-commit se ejecutan en un entorno aislado, es más fiable un *hook* local que use el entorno del proyecto:

```yaml
- repo: local
  hooks:
    - id: mypy
      name: mypy
      entry: uv run mypy
      language: system
      types: [python]
      pass_filenames: false
```

### 15.3 GitHub Actions

```yaml
- uses: astral-sh/setup-uv@v6
- run: uv sync --locked
- run: uv run mypy
```

Cachear `.mypy_cache` acelera las ejecuciones:

```yaml
- uses: actions/cache@v4
  with:
    path: .mypy_cache
    key: mypy-${{ runner.os }}-${{ hashFiles('uv.lock') }}
```

---

## 16. Ejemplo práctico completo

**Puerta de calidad de tipos para un proyecto en migración**: configuración estricta con lista de módulos pendientes y un script que ejecuta mypy, agrupa errores por módulo y código, compara con una línea base guardada y falla solo si **aparecen errores nuevos**. Permite activar mypy hoy en un proyecto con cientos de errores sin bloquear al equipo.

Configuración:

```toml
[tool.mypy]
python_version = "3.12"
files = ["src"]
strict = true
warn_unreachable = true
enable_error_code = ["ignore-without-code", "possibly-undefined"]
show_error_codes = true
```

Script `scripts/mypy_linea_base.py`:

```python
"""
Ejecuta mypy y compara los errores con una línea base.

    python scripts/mypy_linea_base.py              # Comprueba (falla si hay errores nuevos)
    python scripts/mypy_linea_base.py --actualizar # Guarda la situación actual como línea base
"""
from __future__ import annotations

import argparse
import json
import re
import subprocess
import sys
from collections import Counter
from dataclasses import dataclass
from pathlib import Path

LINEA_BASE = Path(".mypy-linea-base.json")
PATRON = re.compile(r"^(?P<archivo>[^:]+):(?P<linea>\d+): error: (?P<mensaje>.*?)\s+\[(?P<codigo>[\w-]+)\]$")


@dataclass(frozen=True, slots=True)
class ErrorMypy:
    archivo: str
    linea: int
    codigo: str
    mensaje: str

    @property
    def clave(self) -> str:
        # Sin número de línea: así mover código no cuenta como error nuevo
        return f"{self.archivo}|{self.codigo}|{self.mensaje}"


def ejecutar_mypy(argumentos: list[str]) -> list[ErrorMypy]:
    proceso = subprocess.run(
        [sys.executable, "-m", "mypy", "--no-pretty", "--no-error-summary", *argumentos],
        capture_output=True, text=True,
    )
    if proceso.returncode not in (0, 1):
        print(proceso.stdout, proceso.stderr, sep="\n")
        raise SystemExit(f"mypy terminó con un error interno (código {proceso.returncode})")
    errores = []
    for linea in proceso.stdout.splitlines():
        if m := PATRON.match(linea):
            errores.append(ErrorMypy(m["archivo"], int(m["linea"]), m["codigo"], m["mensaje"]))
    return errores


def cargar_linea_base() -> Counter[str]:
    if not LINEA_BASE.exists():
        return Counter()
    return Counter(json.loads(LINEA_BASE.read_text(encoding="utf-8")))


def guardar_linea_base(errores: list[ErrorMypy]) -> None:
    contador = Counter(e.clave for e in errores)
    LINEA_BASE.write_text(json.dumps(dict(sorted(contador.items())), indent=2, ensure_ascii=False) + "\n",
                          encoding="utf-8")
    print(f"✔ Línea base guardada: {len(errores)} errores en {len({e.archivo for e in errores})} archivos")


def resumen(errores: list[ErrorMypy]) -> None:
    por_codigo = Counter(e.codigo for e in errores)
    por_archivo = Counter(e.archivo for e in errores)
    print("\nErrores por código:")
    for codigo, n in por_codigo.most_common(10):
        print(f"  {codigo:<22} {n:>5}")
    print("\nArchivos con más errores:")
    for archivo, n in por_archivo.most_common(10):
        print(f"  {archivo:<50} {n:>5}")


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("--actualizar", action="store_true", help="Guarda la situación actual como línea base")
    parser.add_argument("rutas", nargs="*", default=[])
    args = parser.parse_args()

    errores = ejecutar_mypy(args.rutas)
    if args.actualizar:
        guardar_linea_base(errores)
        return 0

    base = cargar_linea_base()
    actuales = Counter(e.clave for e in errores)
    nuevos_claves = actuales - base
    corregidos = sum((base - actuales).values())

    nuevos = [e for e in errores if nuevos_claves[e.clave] > 0]
    resumen(errores)
    print(f"\nTotal: {len(errores)} · Línea base: {sum(base.values())} · Corregidos: {corregidos}")

    if nuevos:
        print(f"\n✖ {len(nuevos)} errores NUEVOS respecto a la línea base:")
        for e in nuevos:
            print(f"  {e.archivo}:{e.linea}: {e.mensaje} [{e.codigo}]")
        return 1
    if corregidos:
        print("\n✔ Sin errores nuevos. Ejecute con --actualizar para registrar las mejoras.")
    else:
        print("\n✔ Sin errores nuevos.")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Uso:

```bash
uv run python scripts/mypy_linea_base.py --actualizar   # Una vez: registra la deuda actual
git add .mypy-linea-base.json
uv run python scripts/mypy_linea_base.py                # En CI: falla solo con errores nuevos
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Library stubs not installed for "X"` | Librería sin tipos. | `types-X` o `ignore_missing_imports` para ese módulo. |
| `Cannot find implementation or library stub for module named "mi_app"` | mypy no encuentra el paquete propio. | Instale el proyecto en editable o configure `mypy_path`/`explicit_package_bases`. |
| `Duplicate module named "X"` | Dos archivos con el mismo nombre de módulo (p. ej. `tests/conftest.py` en varias carpetas). | Añada `__init__.py` o use `explicit_package_bases`. |
| `Function is missing a type annotation` | Modo estricto con funciones sin anotar. | Anote, o relaje en `overrides` temporalmente. |
| `Need type annotation for "x"` | Colección vacía sin tipo: `x = []`. | `x: list[str] = []`. |
| `Incompatible types in assignment` al reutilizar una variable | Variable con dos tipos distintos. | Use otro nombre o anote la unión. |
| `Returning Any from function declared to return "int"` | Valor de una librería sin tipos o `json.loads`. | Valide/convierta el valor o use `cast` justificado. |
| `unused "type: ignore" comment` | El error ya se corrigió. | Elimine el comentario. |
| Resultados distintos en local y en CI | Versiones diferentes de mypy o de las dependencias. | Fije mypy en el bloqueo y ejecute en el entorno del proyecto. |
| El error no aparece en una función | La función no está anotada. | Anótela o active `check_untyped_defs`. |

---

## 18. Buenas prácticas

- Fije la versión de mypy como dependencia de desarrollo y ejecútelo en el entorno del proyecto.
- Configure mypy en `pyproject.toml` y defina `files` para que `mypy` sin argumentos haga lo correcto.
- Objetivo: `strict = true`. En proyectos existentes, llegue a él con `overrides` por módulo.
- Use `# type: ignore[codigo]` siempre con código, y active `warn_unused_ignores` e `ignore-without-code`.
- Nunca active `ignore_missing_imports` de forma global; hágalo por librería.
- Use `reveal_type` para entender lo que infiere mypy antes de añadir un `cast`.
- Active el *plugin* de Pydantic o Django si usa esas librerías.
- Ejecute mypy en CI y en el editor; en pre-commit, con un *hook* local.
- En proyectos grandes, use `dmypy` en local y cachee `.mypy_cache` en CI.

---

## 19. Referencia rápida

```bash
mypy [rutas] | -p paquete | -m modulo | -c "código"
mypy --strict --warn-unreachable --pretty --show-error-context
mypy --install-types --non-interactive
mypy --html-report dir/  ·  -O json
dmypy run -- src/  ·  dmypy check src/  ·  dmypy stop
python -m mypy.stubtest paquete
```

```toml
[tool.mypy]
strict = true · files = ["src"] · plugins = ["pydantic.mypy"] · warn_unreachable = true
enable_error_code = ["ignore-without-code"]
[[tool.mypy.overrides]] module = ["x.*"] · ignore_missing_imports = true
```

```python
reveal_type(x) · # type: ignore[codigo] · cast(T, x) · if TYPE_CHECKING: · @no_type_check
```

---

## 20. Ejercicios propuestos

1. Ejecute mypy sobre un proyecto propio sin configuración y clasifique los errores por código.
2. Active `strict` y use `overrides` para que el proyecto pase; documente la lista de módulos pendientes.
3. Escriba un stub `.pyi` para un módulo interno sin anotaciones y verifíquelo con `stubtest`.
4. Configure el *plugin* de Pydantic y compruebe qué errores detecta al crear modelos con campos erróneos.
5. Integre el script de línea base en un flujo de GitHub Actions y verifique que falla al introducir un error nuevo.

---

## 21. Referencias

- Documentación oficial: https://mypy.readthedocs.io/
- Opciones de configuración: https://mypy.readthedocs.io/en/stable/config_file.html
- Códigos de error: https://mypy.readthedocs.io/en/stable/error_code_list.html
- Hoja de referencia de tipos: https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html
- Adopción en código existente: https://mypy.readthedocs.io/en/stable/existing_code.html
- typeshed (stubs): https://github.com/python/typeshed
