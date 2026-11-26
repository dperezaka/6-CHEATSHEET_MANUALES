# Manual de uso: Pyright y ty

> **Categoría:** Comprobación de tipos · **Utilidad:** Alternativas a mypy, más rápidas
> **Tipo:** Paquetes externos (requieren instalación) · Pyright: Microsoft, estable · ty: Astral, en fase beta

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Pyright: uso básico](#4-pyright-uso-básico)
5. [Pyright: configuración](#5-pyright-configuración)
6. [Pyright: modos y reglas de diagnóstico](#6-pyright-modos-y-reglas-de-diagnóstico)
7. [Pyright: comentarios y supresión de errores](#7-pyright-comentarios-y-supresión-de-errores)
8. [Pyright: completitud de tipos de una librería](#8-pyright-completitud-de-tipos-de-una-librería)
9. [ty: uso básico](#9-ty-uso-básico)
10. [ty: configuración](#10-ty-configuración)
11. [ty: reglas, supresión y servidor de lenguaje](#11-ty-reglas-supresión-y-servidor-de-lenguaje)
12. [Diferencias de comportamiento con mypy](#12-diferencias-de-comportamiento-con-mypy)
13. [Integración con editores](#13-integración-con-editores)
14. [Integración con pre-commit y CI](#14-integración-con-pre-commit-y-ci)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**Pyright** y **ty** son comprobadores de tipos estáticos para Python, alternativos a mypy, que priorizan la **velocidad** y la **experiencia en el editor**.

| | **Pyright** | **ty** |
|---|---|---|
| Desarrollador | Microsoft | Astral (creadores de uv y Ruff) |
| Lenguaje | TypeScript (Node.js) | Rust |
| Madurez | Estable, muy extendido | Beta (versiones `0.0.x`); recomendado para usuarios dispuestos a reportar problemas |
| Editor | Motor de **Pylance** (VS Code) | Servidor de lenguaje propio + extensión de VS Code |
| Configuración | `pyrightconfig.json` o `[tool.pyright]` | `ty.toml` o `[tool.ty]` |
| Fortalezas | Inferencia muy completa, cumplimiento riguroso de la especificación | Velocidad extrema, diagnósticos muy legibles, análisis incremental |

**Ventajas comunes frente a mypy:**
- Mucho más rápidos, especialmente en proyectos grandes y en el editor.
- Comprueban el cuerpo de funciones sin anotar (inferencia de tipos de retorno).
- Diagnósticos con contexto detallado.

**Limitaciones:**
- Sin sistema de *plugins* como el de mypy (Django, por ejemplo, se apoya en stubs).
- Pueden discrepar de mypy en casos límite: elija **uno** como referencia para la CI.
- ty aún no implementa la especificación de tipos al completo; revise su estado antes de adoptarlo como única barrera en CI.

| Si necesita… | Opción |
|---|---|
| Comprobación rápida y estable en VS Code y CI | **Pyright** |
| Máxima velocidad, ecosistema uv/Ruff, aceptando una herramienta en beta | **ty** |
| *Plugins* (Django) o adopción gradual por módulo muy fina | mypy |

---

## 2. Instalación

### 2.1 Pyright

Pyright necesita Node.js; el paquete de PyPI lo descarga automáticamente:

```bash
uv add --dev pyright                   # Dependencia del proyecto (recomendado)
python -m pip install pyright
npm install -g pyright                 # Directamente con npm
```

Variante comunitaria **basedpyright** (sin Node.js, con reglas adicionales y funciones de Pylance):

```bash
uv add --dev basedpyright
```

### 2.2 ty

```bash
uv add --dev ty                        # Dependencia del proyecto
uv tool install ty                     # Herramienta global
uvx ty check                           # Ejecutar sin instalar
```

Comprobar:

```bash
pyright --version
ty --version
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Modo de comprobación | Nivel de exigencia global (Pyright: `basic`, `standard`, `strict`). |
| Regla de diagnóstico | Tipo de error con nombre propio y gravedad configurable. |
| Gravedad | `error`, `warning`, `information` o desactivado (`none`/`ignore`). |
| Servidor de lenguaje (LSP) | Proceso que da diagnósticos, autocompletado e información en el editor. |
| Inferencia de retorno | Deducir el tipo de retorno de una función sin anotación. |
| Entorno de Python | Intérprete y `site-packages` donde se buscan las librerías. |
| Completitud de tipos | Porcentaje de la API pública de una librería con tipos conocidos. |

---

## 4. Pyright: uso básico

```bash
pyright                              # Todo el proyecto (según la configuración)
pyright src/ tests/
pyright archivo.py
pyright --watch                      # Vuelve a comprobar al guardar
pyright --pythonversion 3.12
pyright --pythonpath .venv/bin/python
pyright --outputjson > resultado.json
pyright --level error                # Solo errores, sin avisos
pyright --stats                      # Tiempos y número de archivos
```

Ejemplo de salida:

```text
/proyecto/src/inventario.py
  /proyecto/src/inventario.py:19:7 - error: "stock" is not a known attribute of "None" (reportOptionalMemberAccess)
  /proyecto/src/inventario.py:24:12 - error: Argument of type "Literal['5']" cannot be assigned to parameter "unidades" of type "int"
    "Literal['5']" is not assignable to "int" (reportArgumentType)
2 errors, 0 warnings, 0 informations
```

Pyright detecta automáticamente `.venv` en la raíz del proyecto.

---

## 5. Pyright: configuración

En `pyproject.toml`:

```toml
[tool.pyright]
include = ["src", "tests"]
exclude = ["**/__pycache__", "build", "docs"]
pythonVersion = "3.12"
venvPath = "."
venv = ".venv"
typeCheckingMode = "strict"
reportMissingTypeStubs = "warning"
reportUnnecessaryTypeIgnoreComment = "error"
reportImplicitOverride = "error"

executionEnvironments = [
  { root = "src" },
  { root = "tests", extraPaths = ["src"], reportPrivateUsage = false },
]
```

O en `pyrightconfig.json` (tiene prioridad sobre `pyproject.toml`):

```json
{
  "include": ["src", "tests"],
  "pythonVersion": "3.12",
  "venvPath": ".",
  "venv": ".venv",
  "typeCheckingMode": "strict"
}
```

| Opción | Uso |
|---|---|
| `include` / `exclude` / `ignore` | Qué se comprueba (`ignore`: se analiza pero no se informa). |
| `pythonVersion` / `pythonPlatform` | Versión y plataforma objetivo. |
| `venvPath` + `venv` | Entorno virtual con las dependencias. |
| `extraPaths` | Rutas adicionales de importación. |
| `stubPath` | Carpeta de stubs propios (por defecto `typings/`). |
| `strict` | Lista de rutas que se comprueban en modo estricto. |
| `executionEnvironments` | Configuración distinta por carpeta. |

---

## 6. Pyright: modos y reglas de diagnóstico

| Modo | Uso |
|---|---|
| `off` | Solo errores de sintaxis e importación. |
| `basic` | Errores evidentes; adecuado para empezar. |
| `standard` | Por defecto desde Pyright 1.1.339; buen equilibrio. |
| `strict` | Exige tipos completos; prohíbe `Any` implícito y tipos desconocidos. |

Aplicar estricto solo a parte del proyecto:

```toml
[tool.pyright]
typeCheckingMode = "standard"
strict = ["src/mi_app/dominio", "src/mi_app/api"]
```

Reglas más útiles (se pueden fijar a `"error"`, `"warning"`, `"information"`, `"none"`, `true` o `false`):

| Regla | Detecta |
|---|---|
| `reportOptionalMemberAccess` | Acceso a atributos de un valor que puede ser `None`. |
| `reportArgumentType` | Argumentos de tipo incorrecto. |
| `reportAttributeAccessIssue` | Atributos inexistentes. |
| `reportMissingImports` | Módulos que no se encuentran. |
| `reportMissingTypeStubs` | Librerías sin stubs. |
| `reportUnknownMemberType` / `…ParameterType` / `…VariableType` | Tipos desconocidos (`Unknown`) en modo estricto. |
| `reportPrivateUsage` | Uso de miembros `_privados` desde fuera. |
| `reportUnusedImport` / `reportUnusedVariable` | Código sin usar. |
| `reportUnnecessaryIsInstance` | `isinstance` siempre verdadero. |
| `reportUnnecessaryTypeIgnoreComment` | `# type: ignore` sobrantes. |
| `reportImplicitOverride` | Métodos redefinidos sin `@override`. |
| `reportMatchNotExhaustive` | `match` que no cubre todos los casos. |
| `reportShadowedImports` | Archivos propios que ocultan módulos estándar (p. ej. `random.py`). |
| `reportDeprecated` | Uso de APIs marcadas con `@deprecated`. |

---

## 7. Pyright: comentarios y supresión de errores

```python
valor = cliente.dato_dinamico          # pyright: ignore[reportAttributeAccessIssue]
x = libreria.funcion()                 # type: ignore[import]   (estándar, también lo acepta)
```

Comentarios a nivel de archivo (al principio del módulo):

```python
# pyright: strict
# pyright: basic
# pyright: reportPrivateUsage=false, reportUnknownMemberType=warning
```

Revelar tipos inferidos:

```python
from typing import reveal_type

reveal_type(datos)        # information: Type of "datos" is "dict[str, list[int]]"
```

> Active `reportUnnecessaryTypeIgnoreComment` para que no se acumulen supresiones obsoletas. Prefiera `# pyright: ignore[regla]` a `# type: ignore`: es más específico.

---

## 8. Pyright: completitud de tipos de una librería

Para autores de paquetes: mide qué parte de la API pública tiene tipos conocidos.

```bash
pyright --verifytypes mi_paquete
pyright --verifytypes mi_paquete --ignoreexternal     # Ignora tipos de dependencias
pyright --verifytypes mi_paquete --outputjson
```

```text
Symbols exported by "mi_paquete": 128
  With known type: 121
  With ambiguous type: 2
  With unknown type: 5
Type completeness score: 94.5%
```

Generar stubs para una librería sin tipos:

```bash
pyright --createstub libreria_antigua       # Crea typings/libreria_antigua/*.pyi
```

---

## 9. ty: uso básico

```bash
ty check                               # Proyecto actual (detecta pyproject.toml y .venv)
ty check src/ tests/
ty check archivo.py
ty check --python-version 3.12
ty check --python .venv                # Entorno con las dependencias
ty check --watch                       # Modo vigilancia
ty check --output-format concise       # Una línea por diagnóstico
ty check --output-format github        # Anotaciones para GitHub Actions
ty check --error possibly-unresolved-reference --ignore unresolved-import
ty check --exit-zero                   # No falla nunca (solo informar)
```

Ejemplo de diagnóstico:

```text
error[invalid-argument-type]: Argument to function `reponer` is incorrect
  --> src/inventario.py:24:31
   |
24 |     reponer(productos, "TEC-01", "5")
   |                                  ^^^ Expected `int`, found `Literal["5"]`
   |
info: Function defined here
  --> src/inventario.py:17:5
   |
17 | def reponer(productos: list[Producto], nombre: str, unidades: int) -> int:
   |     ^^^^^^^                                         ------------- Parameter declared here
   |
info: rule `invalid-argument-type` is enabled by default

Found 1 diagnostic
```

---

## 10. ty: configuración

En `pyproject.toml` (`[tool.ty]`) o en `ty.toml` (mismas claves, sin el prefijo `tool.ty`):

```toml
[tool.ty.environment]
python-version = "3.12"
python = ".venv"
root = ["src"]                      # Raíces de los módulos propios

[tool.ty.src]
include = ["src", "tests"]
exclude = ["src/mi_app/generado"]

[tool.ty.rules]
possibly-unresolved-reference = "error"
redundant-cast = "warn"
unused-ignore-comment = "warn"

[tool.ty.analysis]
allowed-unresolved-imports = ["libreria_sin_instalar.*"]

[tool.ty.terminal]
output-format = "concise"
error-on-warning = false

[[tool.ty.overrides]]
include = ["tests/**"]

[tool.ty.overrides.rules]
unresolved-attribute = "warn"
```

| Sección | Uso |
|---|---|
| `[tool.ty.environment]` | Versión y plataforma de Python, intérprete, rutas de búsqueda. |
| `[tool.ty.src]` | Archivos incluidos y excluidos (patrones estilo `.gitignore`). |
| `[tool.ty.rules]` | Gravedad de cada regla: `error`, `warn` o `ignore` (`all` aplica a todas). |
| `[tool.ty.analysis]` | Comportamiento del análisis (importaciones permitidas sin resolver, comentarios `type: ignore`…). |
| `[tool.ty.terminal]` | Formato de salida y si los avisos hacen fallar la ejecución. |
| `[[tool.ty.overrides]]` | Reglas distintas para ciertas rutas. |

> ty está en desarrollo activo: consulte la referencia de configuración de su versión antes de copiar opciones.

---

## 11. ty: reglas, supresión y servidor de lenguaje

### 11.1 Reglas frecuentes

| Regla | Detecta |
|---|---|
| `invalid-argument-type` | Argumentos de tipo incorrecto. |
| `invalid-return-type` | Retorno incompatible con la anotación. |
| `invalid-assignment` | Asignación incompatible. |
| `unresolved-attribute` | Atributos inexistentes. |
| `missing-union-attribute` | Atributo que falta en algún miembro de una unión (p. ej. `None`). |
| `unsupported-operator` | Operaciones no admitidas entre tipos (`int + str`). |
| `unresolved-import` | Módulos no encontrados. |
| `unresolved-reference` | Nombres no definidos. |
| `possibly-unresolved-reference` | Nombres que podrían no estar definidos en algún camino. |
| `call-non-callable` | Llamar a algo que no es invocable. |
| `missing-argument` / `unknown-argument` | Argumentos que faltan o sobran. |
| `division-by-zero` | División por cero detectable. |
| `redundant-cast` | `cast()` innecesario. |
| `unused-ignore-comment` | Supresiones que ya no hacen falta. |

Liste todas con su descripción en la documentación (*Rules reference*).

### 11.2 Supresión

```python
valor = objeto.dinamico          # ty: ignore[unresolved-attribute]
otro = libreria.f()              # type: ignore[import]   (respetado por defecto)
```

```python
from typing import no_type_check

@no_type_check
def dinamica(*args, **kwargs): ...
```

### 11.3 Servidor de lenguaje

```bash
ty server                         # Lo arranca el editor automáticamente
```

- **VS Code**: extensión oficial «ty» (`astral-sh.ty`). Desactive el servidor de Pylance para evitar diagnósticos duplicados (`"python.languageServer": "None"`).
- **Neovim, Helix, Zed, Emacs**: configure `ty server` como servidor LSP para Python.

Ofrece diagnósticos en vivo, ir a la definición, buscar referencias, autocompletado, información al pasar el ratón, *inlay hints* y renombrado.

---

## 12. Diferencias de comportamiento con mypy

```python
def sin_anotar(x):
    return x.upper() + 1          # mypy (por defecto): no comprueba el cuerpo
                                  # Pyright/ty: analizan el cuerpo con los tipos inferidos


def doble(n: int):                # Sin anotación de retorno
    return n * 2

reveal_type(doble(2))             # mypy: Any (en modo no estricto) · Pyright: int · ty: según return-type-inference
```

| Aspecto | mypy | Pyright | ty |
|---|---|---|---|
| Funciones sin anotar | Ignoradas por defecto | Analizadas | Analizadas |
| Tipo de retorno no anotado | `Any` | Inferido | Configurable |
| Lista vacía `x = []` | Exige anotación | Infiere por uso | Infiere |
| *Plugins* | Sí | No | No |
| Supresión propia | `# type: ignore[codigo]` | `# pyright: ignore[regla]` | `# ty: ignore[regla]` |
| Velocidad relativa (proyecto grande) | 1× | ~3-5× | ~10-60× |

> Las cifras de velocidad dependen del proyecto y de la caché; tómelas como orientación.

---

## 13. Integración con editores

### 13.1 VS Code con Pylance (Pyright)

```json
// .vscode/settings.json
{
  "python.analysis.typeCheckingMode": "standard",
  "python.analysis.diagnosticMode": "workspace",
  "python.analysis.inlayHints.functionReturnTypes": true,
  "python.analysis.inlayHints.variableTypes": true
}
```

> Si el proyecto tiene `[tool.pyright]` o `pyrightconfig.json`, Pylance usa esa configuración y la del archivo tiene prioridad.

### 13.2 VS Code con ty

```json
{
  "python.languageServer": "None",
  "ty.diagnosticMode": "workspace"
}
```

### 13.3 PyCharm

PyCharm tiene su propio motor de tipos. Para usar Pyright o ty, configúrelos como servidor LSP mediante el *plugin* correspondiente o ejecútelos como herramienta externa.

---

## 14. Integración con pre-commit y CI

### 14.1 pre-commit (*hooks* locales, usan el entorno del proyecto)

```yaml
repos:
  - repo: local
    hooks:
      - id: pyright
        name: pyright
        entry: uv run pyright
        language: system
        types: [python]
        pass_filenames: false
      - id: ty
        name: ty
        entry: uv run ty check
        language: system
        types: [python]
        pass_filenames: false
```

### 14.2 GitHub Actions

```yaml
- uses: astral-sh/setup-uv@v6
- run: uv sync --locked
- run: uv run pyright --outputjson > pyright.json || true
- run: uv run pyright
- run: uv run ty check --output-format github
```

> Elija **un** comprobador como barrera obligatoria. Puede ejecutar un segundo en modo informativo (`--exit-zero`) mientras lo evalúa.

---

## 15. Ejemplo práctico completo

**Comparativa automatizada de comprobadores**: un proyecto de ejemplo con errores intencionados y un script que ejecuta mypy, Pyright y ty, mide su tiempo y resume cuántos diagnósticos produce cada uno por archivo. Sirve para evaluar la adopción en un proyecto real.

Código de prueba (`src/demo/pedidos.py`):

```python
from dataclasses import dataclass


@dataclass
class Pedido:
    id: int
    importe: float
    cliente: str | None = None


def total(pedidos: list[Pedido]) -> float:
    return sum(p.importe for p in pedidos)


def email_cliente(p: Pedido) -> str:
    return p.cliente.lower() + "@empresa.com"          # Error: cliente puede ser None


def sin_anotar(datos):
    return len(datos) + " unidades"                      # int + str: solo Pyright/ty lo analizan


def descuento(p: Pedido, porcentaje: float) -> float:
    if porcentaje > 1:
        resultado = p.importe * (1 - porcentaje / 100)
    return resultado                                     # Posiblemente sin definir


total([Pedido(1, "100")])                                # Error: str no es float
```

Configuración (`pyproject.toml`):

```toml
[project]
name = "demo"
version = "0.1.0"
requires-python = ">=3.12"

[dependency-groups]
dev = ["mypy>=1.13", "pyright>=1.1.390", "ty"]

[tool.mypy]
files = ["src"]
enable_error_code = ["possibly-undefined"]

[tool.pyright]
include = ["src"]
typeCheckingMode = "standard"

[tool.ty.src]
include = ["src"]
```

Script (`comparar_comprobadores.py`):

```python
"""Ejecuta mypy, Pyright y ty sobre el proyecto y compara resultados y tiempos."""
from __future__ import annotations

import json
import os
import re
import shutil
import subprocess
import time
from collections import Counter
from dataclasses import dataclass, field


@dataclass
class Resultado:
    herramienta: str
    segundos: float = 0.0
    por_archivo: Counter[str] = field(default_factory=Counter)
    disponible: bool = True

    @property
    def total(self) -> int:
        return sum(self.por_archivo.values())


def cronometrar(cmd: list[str]) -> tuple[subprocess.CompletedProcess[str], float]:
    inicio = time.perf_counter()
    proceso = subprocess.run(cmd, capture_output=True, text=True)
    return proceso, time.perf_counter() - inicio


def ejecutar_mypy() -> Resultado:
    r = Resultado("mypy")
    proceso, r.segundos = cronometrar(["mypy", "--no-pretty", "--no-error-summary", "--no-incremental"])
    for linea in proceso.stdout.splitlines():
        if m := re.match(r"^([^:]+):\d+: error:", linea):
            r.por_archivo[m[1]] += 1
    return r


def ejecutar_pyright() -> Resultado:
    r = Resultado("pyright")
    proceso, r.segundos = cronometrar(["pyright", "--outputjson"])
    datos = json.loads(proceso.stdout or "{}")
    for d in datos.get("generalDiagnostics", []):
        if d.get("severity") == "error":
            r.por_archivo[os.path.relpath(d["file"])] += 1
    return r


def ejecutar_ty() -> Resultado:
    r = Resultado("ty")
    proceso, r.segundos = cronometrar(["ty", "check", "--output-format", "concise", "--exit-zero"])
    for linea in proceso.stdout.splitlines():
        if m := re.match(r"^([^:]+):\d+:\d+: error", linea):
            r.por_archivo[m[1]] += 1
    return r


def main() -> None:
    ejecutores = {"mypy": ejecutar_mypy, "pyright": ejecutar_pyright, "ty": ejecutar_ty}
    resultados = []
    for nombre, funcion in ejecutores.items():
        if shutil.which(nombre) is None:
            resultados.append(Resultado(nombre, disponible=False))
            continue
        resultados.append(funcion())

    archivos = sorted({a for r in resultados for a in r.por_archivo})
    ancho = max([len(a) for a in archivos] + [20])
    print(f"{'Archivo':<{ancho}}" + "".join(f"{r.herramienta:>10}" for r in resultados))
    for archivo in archivos:
        print(f"{archivo:<{ancho}}" + "".join(f"{r.por_archivo.get(archivo, 0):>10}" for r in resultados))
    print("-" * (ancho + 10 * len(resultados)))
    print(f"{'Total de errores':<{ancho}}" + "".join(
        f"{(r.total if r.disponible else '-'):>10}" for r in resultados))
    print(f"{'Tiempo (s)':<{ancho}}" + "".join(
        f"{(f'{r.segundos:.2f}' if r.disponible else '-'):>10}" for r in resultados))


if __name__ == "__main__":
    main()
```

Uso:

```bash
uv sync
uv run python comparar_comprobadores.py
```

Salida obtenida (mypy 1.x, Pyright 1.1.x, ty 0.0.x; los tiempos dependen del equipo):

```text
Archivo                   mypy   pyright        ty
src/demo/pedidos.py          3         4         3
--------------------------------------------------
Total de errores             3         4         3
Tiempo (s)                0.67      0.63      0.03
```

Interpretación:

- **mypy** no informa del error de `sin_anotar` porque, por defecto, no comprueba funciones sin anotaciones.
- **Pyright** detecta los cuatro problemas, incluida la variable `resultado` posiblemente sin definir.
- **ty** detecta `sin_anotar`, pero con su configuración por defecto no trata como error la variable posiblemente sin definir; puede endurecerse en `[tool.ty.rules]`.

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Import "x" could not be resolved` (Pyright) / `unresolved-import` (ty) | No encuentra el entorno con las dependencias. | Configure `venvPath`/`venv` (Pyright) o `environment.python` (ty), o cree `.venv` en la raíz. |
| `Stub file not found for "x"` (Pyright) | Librería sin stubs. | Instale `types-x` o rebaje `reportMissingTypeStubs`. |
| Pylance y la CI dan resultados distintos | Pylance usa otra versión de Pyright o la configuración del usuario. | Defina la configuración en `pyproject.toml` y fije la versión de Pyright en el proyecto. |
| Avalancha de `reportUnknownMemberType` | Modo `strict` con librerías sin tipos. | Use `standard` globalmente y `strict` solo en sus módulos. |
| Diagnósticos duplicados en VS Code | Pylance y ty activos a la vez. | Desactive el servidor de lenguaje de Pylance. |
| ty no reconoce una construcción válida | Funcionalidad aún no implementada (beta). | Suprima con `# ty: ignore[regla]`, actualice ty o informe del problema. |
| Pyright descarga Node.js en cada CI | Instalación desde PyPI sin caché. | Cachee `~/.cache/pyright-python` o use basedpyright. |
| `# type: ignore` no suprime el error en Pyright | Activada `enableTypeIgnoreComments = false`. | Use `# pyright: ignore[regla]`. |

---

## 17. Buenas prácticas

- Elija **un comprobador de referencia** para la CI y configúrelo en `pyproject.toml`.
- Fije la versión del comprobador como dependencia de desarrollo; en ty, es especialmente importante por su ritmo de cambios.
- Use el mismo comprobador en el editor y en la CI para evitar discrepancias.
- Empiece en modo `standard` (Pyright) o con las reglas por defecto (ty) y endurezca por carpetas.
- Use supresiones específicas (`# pyright: ignore[regla]`, `# ty: ignore[regla]`) y active los avisos de supresiones sin usar.
- Si publica una librería, mida su completitud con `pyright --verifytypes` y añada `py.typed`.
- Evalúe ty en paralelo (modo informativo) antes de convertirlo en barrera obligatoria.
- Mantenga las anotaciones conformes a la especificación estándar: así podrá cambiar de comprobador sin reescribir código.

---

## 18. Referencia rápida

```bash
# Pyright
pyright [rutas] [--watch] [--pythonversion 3.12] [--outputjson] [--level error]
pyright --verifytypes paquete  ·  pyright --createstub libreria

# ty
ty check [rutas] [--python .venv] [--python-version 3.12] [--watch]
ty check --output-format concise|github  ·  --error regla  ·  --ignore regla  ·  --exit-zero
ty server
```

```toml
[tool.pyright]  include, exclude, pythonVersion, venvPath, venv, typeCheckingMode, strict = [...], reportX = "error"
[tool.ty.environment]  python-version, python, root
[tool.ty.src]  include, exclude
[tool.ty.rules]  regla = "error" | "warn" | "ignore"
[[tool.ty.overrides]]  include = [...]  + [tool.ty.overrides.rules]
```

```python
# pyright: strict   ·   # pyright: ignore[regla]   ·   # ty: ignore[regla]   ·   reveal_type(x)
```

---

## 19. Ejercicios propuestos

1. Ejecute Pyright en modo `basic`, `standard` y `strict` sobre un proyecto propio y compare el número de diagnósticos.
2. Configure `executionEnvironments` para que `tests/` use reglas más relajadas que `src/`.
3. Instale ty, configure `[tool.ty]` y compare sus diagnósticos con los de Pyright en el ejemplo del manual.
4. Mida la completitud de tipos de una librería propia con `--verifytypes` y súbala por encima del 95 %.
5. Configure VS Code para usar ty como servidor de lenguaje y evalúe la experiencia frente a Pylance.

---

## 20. Referencias

- Pyright, documentación: https://microsoft.github.io/pyright/
- Pyright, configuración: https://microsoft.github.io/pyright/#/configuration
- basedpyright: https://docs.basedpyright.com/
- ty, documentación: https://docs.astral.sh/ty/
- ty, configuración: https://docs.astral.sh/ty/reference/configuration/
- ty, reglas: https://docs.astral.sh/ty/reference/rules/
- Anuncio de la beta de ty: https://astral.sh/blog/ty
