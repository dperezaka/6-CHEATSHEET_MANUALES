# Manual de uso: pytest-cov

> **Categoría:** Pruebas · **Utilidad:** Mide la cobertura de las pruebas
> **Tipo:** *Plugin* de pytest (requiere instalación) · Usa internamente **coverage.py**

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos](#4-primeros-pasos)
5. [Informes](#5-informes)
6. [Cobertura de ramas](#6-cobertura-de-ramas)
7. [Configuración con coverage.py](#7-configuración-con-coveragepy)
8. [Excluir código de la medición](#8-excluir-código-de-la-medición)
9. [Umbral mínimo de cobertura](#9-umbral-mínimo-de-cobertura)
10. [Contextos: qué prueba cubre cada línea](#10-contextos-qué-prueba-cubre-cada-línea)
11. [Ejecución en paralelo, subprocesos y combinación](#11-ejecución-en-paralelo-subprocesos-y-combinación)
12. [Cobertura de los cambios (*diff coverage*)](#12-cobertura-de-los-cambios-diff-coverage)
13. [Integración con CI y servicios externos](#13-integración-con-ci-y-servicios-externos)
14. [Interpretar la cobertura](#14-interpretar-la-cobertura)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**pytest-cov** integra **coverage.py** con pytest para medir qué partes del código se ejecutan durante las pruebas. Indica, por archivo, el porcentaje de líneas (y ramas) cubiertas y **qué líneas concretas no se han probado**.

```text
Name                    Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------------
src/tienda/__init__.py      0      0      0      0   100%
src/tienda/pedidos.py      65      6     10      0    92%   86-91
-------------------------------------------------------------------
TOTAL                      65      6     10      0    92%
```

**Ventajas:**
- Una sola opción (`--cov`) para medir mientras se ejecutan las pruebas.
- Informes en terminal, HTML navegable, XML (Cobertura), JSON, LCOV y Markdown.
- Funciona con `pytest-xdist` (ejecución en paralelo) sin configuración adicional.
- Permite fallar la ejecución si la cobertura baja de un umbral.

**Limitaciones:**
- La cobertura indica qué se **ejecutó**, no qué se **comprobó**: un 100 % no garantiza pruebas buenas.
- Añade una pequeña sobrecarga de tiempo (menor con el núcleo `sysmon` de Python 3.12+).

| Si necesita… | Opción |
|---|---|
| Cobertura al ejecutar pytest | **pytest-cov** |
| Cobertura de cualquier programa o con otro marco de pruebas | coverage.py directamente (`coverage run`) |
| Cobertura solo de las líneas modificadas en una PR | diff-cover |
| Calidad real de las aserciones | Pruebas de mutación (mutmut, cosmic-ray) |

---

## 2. Instalación

```bash
uv add --dev pytest-cov
poetry add --group dev pytest-cov
python -m pip install pytest-cov
```

Instala también `coverage`. Comprobar:

```bash
pytest --version          # Muestra los plugins cargados, incluido pytest-cov
coverage --version
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Sentencia (*statement*) | Línea ejecutable de código. |
| Cobertura de líneas | Porcentaje de sentencias ejecutadas al menos una vez. |
| Cobertura de ramas | Porcentaje de caminos de decisión (`if`/`else`, bucles) recorridos. |
| Rama parcial (*BrPart*) | Decisión en la que solo se ha tomado uno de los caminos. |
| `Miss` | Sentencias no ejecutadas. |
| Fuente (*source*) | Código que se mide (su paquete, no las librerías externas). |
| Archivo de datos | `.coverage`: resultados brutos de la medición. |
| Contexto | Etiqueta que indica qué prueba ejecutó cada línea. |
| Combinar | Unir datos de varias ejecuciones (procesos, versiones de Python). |

---

## 4. Primeros pasos

```bash
pytest --cov                              # Mide todo lo importado bajo el directorio actual
pytest --cov=tienda                       # Mide el paquete «tienda» (recomendado)
pytest --cov=src/tienda                   # Por ruta
pytest --cov=tienda --cov=utilidades      # Varios paquetes
pytest --cov=tienda tests/unitarias/      # Solo con algunas pruebas
pytest --no-cov                           # Desactivar (p. ej. al depurar)
```

> Indique siempre **qué medir** (`--cov=paquete` o `source` en la configuración). Si no, se incluyen archivos de prueba y, a veces, librerías.

Con depuradores (`--pdb`, puntos de ruptura del IDE), la medición puede interferir: use `--no-cov` al depurar.

---

## 5. Informes

```bash
pytest --cov=tienda --cov-report=term                 # Resumen en terminal (por defecto)
pytest --cov=tienda --cov-report=term-missing         # Con las líneas no cubiertas
pytest --cov=tienda --cov-report=term-missing:skip-covered   # Oculta archivos al 100 %
pytest --cov=tienda --cov-report=html                 # htmlcov/index.html
pytest --cov=tienda --cov-report=html:informes/cobertura
pytest --cov=tienda --cov-report=xml                  # coverage.xml (formato Cobertura)
pytest --cov=tienda --cov-report=json                 # coverage.json
pytest --cov=tienda --cov-report=lcov                 # coverage.lcov
pytest --cov=tienda --cov-report=markdown             # coverage.md
pytest --cov=tienda --cov-report=                     # Sin informe (solo genera .coverage)
pytest --cov=tienda --cov-report=term-missing --cov-report=html --cov-report=xml   # Varios
```

| Informe | Uso |
|---|---|
| `term-missing` | Trabajo diario: ver al momento qué falta. |
| `html` | Revisión detallada: código resaltado en verde/rojo/amarillo por línea. |
| `xml` | CI, Codecov, GitLab, SonarQube, Azure DevOps. |
| `json` | Scripts propios (umbrales por módulo, paneles). |
| `lcov` | Extensiones de editor (Coverage Gutters) y Coveralls. |
| `markdown` | Comentarios en *pull requests* y resúmenes de CI. |

Informes desde los datos guardados (sin volver a ejecutar las pruebas):

```bash
coverage report -m
coverage html
coverage report --format=markdown
coverage report --include="src/tienda/pagos/*"
```

---

## 6. Cobertura de ramas

La cobertura de líneas puede ser engañosa:

```python
def descuento(cliente_vip: bool, importe: float) -> float:
    porcentaje = 0.0
    if cliente_vip:
        porcentaje = 0.1
    return importe * (1 - porcentaje)


def test_vip():
    assert descuento(True, 100) == 90
```

Esta prueba ejecuta **todas las líneas** (100 % de líneas), pero nunca prueba el caso `cliente_vip=False`. La cobertura de ramas lo detecta:

```bash
pytest --cov=tienda --cov-branch --cov-report=term-missing
```

```text
Name                Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------------
tienda/precios.py       5      0      2      1    86%   3->5
```

`3->5` significa: «desde la línea 3, el salto a la línea 5 (la condición falsa) nunca se produjo».

> Active **siempre** la cobertura de ramas (`branch = true`).

---

## 7. Configuración con coverage.py

pytest-cov lee la configuración de coverage.py en `pyproject.toml`, `.coveragerc`, `setup.cfg` o `tox.ini`.

```toml
[tool.coverage.run]
source = ["tienda"]                # Qué medir (nombre de paquete o ruta)
branch = true
parallel = false
omit = [
    "*/migraciones/*",
    "*/__main__.py",
    "src/tienda/generado/*",
]
relative_files = true              # Rutas relativas (necesario para combinar entre máquinas)
core = "sysmon"                    # Python 3.12+: medición más rápida

[tool.coverage.report]
show_missing = true
skip_covered = true
skip_empty = true
precision = 1
fail_under = 90
sort = "Cover"
exclude_also = [
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
    "@(abc\\.)?abstractmethod",
    "@overload",
    "if __name__ == .__main__.:",
    "\\.\\.\\.$",
]

[tool.coverage.html]
directory = "informes/cobertura"
title = "Cobertura de Tienda"
show_contexts = true

[tool.coverage.xml]
output = "informes/coverage.xml"

[tool.coverage.json]
output = "informes/coverage.json"
pretty_print = true
```

Opciones de pytest en la configuración para no repetirlas:

```toml
[tool.pytest]
addopts = ["--cov", "--cov-report=term-missing:skip-covered"]
```

> Con `source` definido en la configuración, basta con `--cov` sin argumento.

---

## 8. Excluir código de la medición

### 8.1 Comentario en el código

```python
def depurar_estado():  # pragma: no cover
    print(vars(self))


if sys.platform == "win32":  # pragma: no cover
    import winreg

try:
    import ujson as json
except ImportError:  # pragma: no branch
    import json
```

| Comentario | Efecto |
|---|---|
| `# pragma: no cover` | Excluye la línea (o el bloque que empieza en ella). |
| `# pragma: no branch` | No exige cubrir ambas ramas de esa decisión. |

### 8.2 Patrones globales (`exclude_also`)

Preferible al comentario para casos repetidos: `if TYPE_CHECKING:`, métodos abstractos, `raise NotImplementedError`, cuerpos `...` de protocolos y sobrecargas.

> Excluya solo código que **no tiene sentido** probar. Excluir para «subir el porcentaje» oculta deuda real.

### 8.3 Archivos completos

Use `omit` en `[tool.coverage.run]` para código generado, migraciones o scripts puntuales.

---

## 9. Umbral mínimo de cobertura

```bash
pytest --cov=tienda --cov-fail-under=85
```

```text
FAIL Required test coverage of 85% not reached. Total coverage: 82.35%
```

O en la configuración (`[tool.coverage.report] fail_under = 85`).

Estrategia de umbral para un proyecto existente (**trinquete**):

1. Mida la cobertura actual (p. ej. 63 %).
2. Fije `fail_under` en ese valor.
3. Súbalo cada vez que la cobertura mejore. Nunca lo baje.
4. Exija además una cobertura alta **del código nuevo** (sección 12).

---

## 10. Contextos: qué prueba cubre cada línea

```bash
pytest --cov=tienda --cov-context=test --cov-report=html
```

```toml
[tool.coverage.html]
show_contexts = true
```

En el informe HTML, cada línea indica qué pruebas la ejecutaron. Útil para:

- localizar las pruebas relacionadas con un fragmento de código;
- detectar código cubierto solo por pruebas de integración lentas;
- revisar si una línea crítica está cubierta por pruebas con aserciones relevantes.

Consulta programática:

```python
from coverage import CoverageData

datos = CoverageData()
datos.read()
for archivo in datos.measured_files():
    for linea, contextos in sorted(datos.contexts_by_lineno(archivo).items()):
        print(archivo, linea, contextos)
```

---

## 11. Ejecución en paralelo, subprocesos y combinación

### 11.1 pytest-xdist

Funciona sin configuración: pytest-cov combina automáticamente los datos de cada proceso.

```bash
pytest -n auto --cov=tienda --cov-report=term-missing
```

### 11.2 Subprocesos

Si las pruebas lanzan procesos Python (`subprocess.run([sys.executable, ...])`), active la medición en ellos:

```toml
[tool.coverage.run]
patch = ["subprocess"]
```

> Desde pytest-cov 7, la medición de subprocesos se configura con esta opción de coverage.py (ya no mediante un archivo `.pth`).

### 11.3 Combinar ejecuciones (varias versiones de Python, matrices de CI)

```bash
# En cada trabajo de la matriz
COVERAGE_FILE=.coverage.py312 pytest --cov --cov-report=
COVERAGE_FILE=.coverage.py313 pytest --cov --cov-report=

# En un trabajo final
coverage combine                    # Une .coverage.* en .coverage
coverage report -m
coverage html
```

```toml
[tool.coverage.run]
relative_files = true      # Imprescindible si las rutas difieren entre máquinas

[tool.coverage.paths]      # Equivalencias de rutas al combinar
source = ["src/tienda", "*/site-packages/tienda"]
```

### 11.4 Añadir a datos existentes

```bash
pytest tests/unitarias --cov --cov-report=
pytest tests/integracion --cov --cov-append --cov-report=term-missing
```

---

## 12. Cobertura de los cambios (*diff coverage*)

El porcentaje global se mueve despacio en proyectos grandes. Lo útil en una *pull request* es saber si **el código nuevo** está probado:

```bash
uv add --dev diff-cover
pytest --cov --cov-report=xml
diff-cover coverage.xml --compare-branch=origin/main --fail-under=90
diff-cover coverage.xml --compare-branch=origin/main --format markdown:diff-cover.md
```

```text
Diff Coverage
Diff: origin/main...HEAD, staged and unstaged changes
-------------
src/tienda/pagos.py (85.7%): Missing lines 42,47
src/tienda/pedidos.py (100%)
-------------
Total:   21 lines
Missing: 2 lines
Coverage: 90%
```

---

## 13. Integración con CI y servicios externos

### 13.1 GitHub Actions con resumen en la ejecución

```yaml
- uses: astral-sh/setup-uv@v6
- run: uv sync --locked
- run: uv run pytest --cov --cov-report=xml --cov-report=markdown:coverage.md
- if: always()
  run: cat coverage.md >> "$GITHUB_STEP_SUMMARY"
- uses: actions/upload-artifact@v4
  with:
    name: cobertura
    path: coverage.xml
```

### 13.2 Codecov

```yaml
- uses: codecov/codecov-action@v5
  with:
    files: coverage.xml
    token: ${{ secrets.CODECOV_TOKEN }}
```

### 13.3 GitLab CI (cobertura en las *merge requests*)

```yaml
pruebas:
  script:
    - uv run pytest --cov --cov-report=term --cov-report=xml
  coverage: '/TOTAL.*\s(\d+(?:\.\d+)?)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```

GitLab muestra en el *diff* de la MR qué líneas están cubiertas.

### 13.4 Editor

La extensión **Coverage Gutters** (VS Code) muestra la cobertura junto al código a partir de `coverage.lcov` o `coverage.xml`:

```bash
pytest --cov --cov-report=lcov
```

---

## 14. Interpretar la cobertura

| Cobertura | Lectura orientativa |
|---|---|
| < 50 % | Gran parte del código no se ejecuta en las pruebas: riesgo alto al cambiarlo. |
| 50-80 % | Cobertura parcial; revise qué módulos críticos quedan fuera. |
| 80-95 % | Objetivo razonable para la mayoría de proyectos. |
| > 95 % | Adecuado para librerías y lógica crítica (pagos, seguridad). |

Lo que la cobertura **no** dice:

```python
def test_calcular_total():
    calcular_total([Linea("A", 1, 10.0)])      # 100 % de cobertura… sin ninguna aserción
```

- Use la cobertura para encontrar **código no probado**, no como medida de calidad.
- Revise las **ramas parciales**: suelen ser casos límite olvidados (errores, valores vacíos).
- Para medir la calidad de las aserciones, use **pruebas de mutación** (`mutmut`): modifican el código y comprueban si alguna prueba falla.

---

## 15. Ejemplo práctico completo

**Política de cobertura por módulo**: además del umbral global, el equipo exige más cobertura en los módulos críticos. El script lee `coverage.json`, aplica umbrales por patrón de ruta (líneas y ramas), muestra una tabla y genera un resumen Markdown. Se usa sobre el proyecto `tienda` del manual de pytest.

Configuración (`pyproject.toml`):

```toml
[tool.pytest]
testpaths = ["tests"]
addopts = ["-ra", "--import-mode=importlib", "--cov", "--cov-report=term-missing:skip-covered",
           "--cov-report=json"]
markers = ["lento: pruebas que tardan más de 1 s"]

[tool.coverage.run]
source = ["tienda"]
branch = true

[tool.coverage.report]
fail_under = 85
exclude_also = ["if TYPE_CHECKING:", "raise NotImplementedError", "\\.\\.\\.$"]

[tool.coverage.json]
output = "coverage.json"

# Umbrales propios por módulo (los lee el script)
[[tool.politica-cobertura.modulo]]
patron = "*/pagos/*"
lineas = 95
ramas = 90

[[tool.politica-cobertura.modulo]]
patron = "*/pedidos.py"
lineas = 95
ramas = 85

[[tool.politica-cobertura.modulo]]
patron = "*"
lineas = 70
ramas = 60
```

Script (`scripts/politica_cobertura.py`):

```python
"""
Aplica umbrales de cobertura por módulo a partir de coverage.json.

    uv run pytest                                    # Genera coverage.json
    uv run python scripts/politica_cobertura.py [--resumen cobertura.md]
"""
from __future__ import annotations

import argparse
import fnmatch
import json
import sys
import tomllib
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class Regla:
    patron: str
    lineas: float
    ramas: float


@dataclass(frozen=True)
class Resultado:
    archivo: str
    lineas: float
    ramas: float | None
    regla: Regla
    pendientes: list[int]

    @property
    def cumple(self) -> bool:
        ok_ramas = self.ramas is None or self.ramas >= self.regla.ramas
        return self.lineas >= self.regla.lineas and ok_ramas


def cargar_reglas(pyproject: Path) -> list[Regla]:
    datos = tomllib.loads(pyproject.read_text(encoding="utf-8"))
    modulos = datos.get("tool", {}).get("politica-cobertura", {}).get("modulo", [])
    if not modulos:
        sys.exit("No hay reglas en [[tool.politica-cobertura.modulo]]")
    return [Regla(m["patron"], float(m["lineas"]), float(m.get("ramas", 0))) for m in modulos]


def porcentaje(cubiertas: int, total: int) -> float | None:
    return None if total == 0 else 100 * cubiertas / total


def evaluar(cobertura: dict, reglas: list[Regla]) -> list[Resultado]:
    resultados = []
    for archivo, info in sorted(cobertura["files"].items()):
        s = info["summary"]
        regla = next(r for r in reglas if fnmatch.fnmatch(archivo, r.patron))
        lineas = porcentaje(s["covered_lines"], s["num_statements"])
        ramas = porcentaje(s.get("covered_branches", 0), s.get("num_branches", 0))
        resultados.append(Resultado(archivo, 100.0 if lineas is None else lineas, ramas, regla,
                                    info.get("missing_lines", [])))
    return resultados


def rangos(lineas: list[int]) -> str:
    if not lineas:
        return ""
    tramos, inicio, previo = [], lineas[0], lineas[0]
    for n in lineas[1:] + [None]:  # type: ignore[list-item]
        if n != (previo + 1 if previo is not None else None):
            tramos.append(str(inicio) if inicio == previo else f"{inicio}-{previo}")
            inicio = n
        previo = n
    return ", ".join(tramos)


def tabla_markdown(resultados: list[Resultado], total: float) -> str:
    filas = ["## 📊 Cobertura por módulo\n", f"**Total del proyecto:** {total:.1f} %\n",
             "| | Archivo | Líneas | Ramas | Mínimo (líneas/ramas) | Sin cubrir |",
             "|---|---|---:|---:|---:|---|"]
    for r in resultados:
        ramas = "-" if r.ramas is None else f"{r.ramas:.1f} %"
        filas.append(f"| {'✅' if r.cumple else '❌'} | `{r.archivo}` | {r.lineas:.1f} % | {ramas} "
                     f"| {r.regla.lineas:.0f} / {r.regla.ramas:.0f} | {rangos(r.pendientes)} |")
    return "\n".join(filas) + "\n"


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("--json", type=Path, default=Path("coverage.json"))
    parser.add_argument("--pyproject", type=Path, default=Path("pyproject.toml"))
    parser.add_argument("--resumen", type=Path)
    args = parser.parse_args()

    if not args.json.exists():
        sys.exit(f"No existe {args.json}: ejecute antes «pytest --cov --cov-report=json»")
    cobertura = json.loads(args.json.read_text(encoding="utf-8"))
    resultados = evaluar(cobertura, cargar_reglas(args.pyproject))
    total = cobertura["totals"]["percent_covered"]

    ancho = max(len(r.archivo) for r in resultados)
    print(f"{'':2}{'Archivo':<{ancho}}  {'Líneas':>7}  {'Ramas':>7}  {'Mínimo':>9}")
    for r in resultados:
        ramas = "-" if r.ramas is None else f"{r.ramas:.1f}%"
        print(f"{'✔' if r.cumple else '✖'} {r.archivo:<{ancho}}  {r.lineas:>6.1f}%  {ramas:>7}  "
              f"{r.regla.lineas:>4.0f}/{r.regla.ramas:<4.0f}")
    fallos = [r for r in resultados if not r.cumple]
    print(f"\nTotal: {total:.1f} % · {len(fallos)} módulos por debajo de su umbral")

    if args.resumen:
        args.resumen.write_text(tabla_markdown(resultados, total), encoding="utf-8")
    return 1 if fallos else 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Ejecución sobre el proyecto de ejemplo (todas las pruebas):

```text
$ uv run pytest -q
================================ tests coverage ================================
_______________ coverage: platform linux, python 3.13.18-final-0 _______________

Name    Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------
TOTAL      65      0     10      0   100%

2 files skipped due to complete coverage.
Coverage JSON written to file coverage.json
Required test coverage of 85.0% reached. Total coverage: 100.00%
15 passed in 0.58s

$ uv run python scripts/politica_cobertura.py
  Archivo                  Líneas    Ramas     Mínimo
✔ src/tienda/__init__.py   100.0%        -    70/60
✔ src/tienda/pedidos.py    100.0%   100.0%    95/85

Total: 100.0 % · 0 módulos por debajo de su umbral
```

Si se elimina la prueba `test_exportar_csv`, el total sigue por encima del umbral global (85 %), pero el módulo crítico `pedidos.py` baja de su mínimo y la política lo detecta:

```text
$ uv run pytest -q --deselect tests/test_servicio.py::test_exportar_csv

Name                    Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------------
src/tienda/pedidos.py      65      6     10      0    92%   86-91
-------------------------------------------------------------------
TOTAL                      65      6     10      0    92%

1 file skipped due to complete coverage.
Coverage JSON written to file coverage.json
Required test coverage of 85.0% reached. Total coverage: 92.00%
14 passed, 1 deselected in 0.58s

$ uv run python scripts/politica_cobertura.py --resumen cobertura.md
  Archivo                  Líneas    Ramas     Mínimo
✔ src/tienda/__init__.py   100.0%        -    70/60
✖ src/tienda/pedidos.py     90.8%   100.0%    95/85

Total: 92.0 % · 1 módulos por debajo de su umbral
```

Este es exactamente el caso que un umbral global no detecta: la exportación a CSV ha quedado sin probar (líneas 86-91) y el porcentaje total apenas lo refleja.

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `CoverageWarning: No data was collected` | `--cov` apunta a un paquete que no se importa o con otro nombre. | Use el nombre importable (`tienda`), no la carpeta del proyecto; verifique que las pruebas lo importan. |
| `Module X was previously imported, but not measured` | El paquete se importó antes de empezar la medición (p. ej. en un *plugin* o `conftest` de nivel superior). | Mida por ruta (`source = ["src/tienda"]`) o evite importarlo antes. |
| Aparecen los archivos de prueba en el informe | No se definió `source`. | `source = ["tienda"]` o `--cov=tienda`. |
| Cobertura 0 % con el paquete instalado (no editable) | Se mide `src/` pero se ejecuta la copia de `site-packages`. | Instale en editable o configure `[tool.coverage.paths]`. |
| Los puntos de ruptura del IDE no se detienen | Interferencia del trazador de coverage. | `--no-cov` al depurar. |
| Cobertura distinta en local y CI | Pruebas omitidas por plataforma o dependencias opcionales. | Combine los datos de toda la matriz antes de aplicar el umbral. |
| `Can't combine line data with arc data` | Mezcla de ejecuciones con y sin `--cov-branch`. | Use `branch = true` en todas. |
| No se mide el código de subprocesos | Falta la configuración de subprocesos. | `[tool.coverage.run] patch = ["subprocess"]`. |
| `FAIL Required test coverage of X% not reached` | La cobertura total está por debajo de `fail_under`. | Añada pruebas o ajuste el umbral según la estrategia de trinquete. |

---

## 17. Buenas prácticas

- Defina `source` y `branch = true` en `pyproject.toml`; ejecute con `--cov` sin más argumentos.
- Use `term-missing:skip-covered` en local y `xml`/`json`/`markdown` en CI.
- Fije un **umbral global** con estrategia de trinquete y exija cobertura alta en el **código nuevo** (diff-cover).
- Aplique umbrales más altos en los **módulos críticos**.
- Excluya con `exclude_also` lo que no tiene sentido probar; evite `# pragma: no cover` salvo casos justificados.
- Revise el informe HTML y las ramas parciales: indican casos límite sin probar.
- Combine la cobertura de todas las versiones de Python y plataformas antes de evaluarla.
- No persiga el 100 % a toda costa: priorice pruebas con aserciones significativas.
- Use contextos (`--cov-context=test`) para saber qué pruebas cubren el código crítico.

---

## 18. Referencia rápida

```bash
pytest --cov[=paquete] [--cov-branch] [--cov-report=term-missing[:skip-covered]|html|xml|json|lcov|markdown]
pytest --cov --cov-fail-under=90  ·  --cov-append  ·  --cov-context=test  ·  --no-cov
coverage report -m  ·  coverage html  ·  coverage combine  ·  coverage erase
diff-cover coverage.xml --compare-branch=origin/main --fail-under=90
```

```toml
[tool.coverage.run]     source = ["paquete"] · branch = true · omit = [...] · patch = ["subprocess"]
[tool.coverage.report]  fail_under = 90 · show_missing = true · skip_covered = true · exclude_also = [...]
[tool.coverage.html]    directory = "htmlcov" · show_contexts = true
```

```python
# pragma: no cover   ·   # pragma: no branch
```

---

## 19. Ejercicios propuestos

1. Mida la cobertura de un proyecto propio con y sin `--cov-branch` y explique la diferencia en tres archivos.
2. Abra el informe HTML, localice una rama parcial y escriba la prueba que la cubre.
3. Configure `exclude_also` para excluir `if TYPE_CHECKING:` y los métodos abstractos; compare el porcentaje antes y después.
4. Configure diff-cover en una *pull request* de prueba y verifique que falla si el código nuevo no está probado.
5. Amplíe el script de política para comparar con el `coverage.json` de la rama principal y señalar los módulos cuya cobertura ha bajado.

---

## 20. Referencias

- pytest-cov: https://pytest-cov.readthedocs.io/
- coverage.py: https://coverage.readthedocs.io/
- Configuración de coverage.py: https://coverage.readthedocs.io/en/latest/config.html
- Cobertura de ramas: https://coverage.readthedocs.io/en/latest/branch.html
- diff-cover: https://pypi.org/project/diff-cover/
- mutmut (pruebas de mutación): https://mutmut.readthedocs.io/
