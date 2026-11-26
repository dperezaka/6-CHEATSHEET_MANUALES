# Manual de uso: Ruff

> **Categoría:** Linter y formateador · **Utilidad:** Analiza el estilo y los errores, ordena los *imports* y formatea; sustituye a flake8, isort y black
> **Tipo:** Herramienta externa (binario escrito en Rust, distribuido en PyPI)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Analizar código: `ruff check`](#4-analizar-código-ruff-check)
5. [Correcciones automáticas](#5-correcciones-automáticas)
6. [Formatear código: `ruff format`](#6-formatear-código-ruff-format)
7. [Configuración general](#7-configuración-general)
8. [Selección de reglas](#8-selección-de-reglas)
9. [Familias de reglas recomendadas](#9-familias-de-reglas-recomendadas)
10. [Ajustes por regla y por archivo](#10-ajustes-por-regla-y-por-archivo)
11. [Ordenar *imports* (sustituto de isort)](#11-ordenar-imports-sustituto-de-isort)
12. [Supresión de avisos (`noqa`)](#12-supresión-de-avisos-noqa)
13. [Notebooks de Jupyter](#13-notebooks-de-jupyter)
14. [Integración con editores](#14-integración-con-editores)
15. [Integración con pre-commit y CI](#15-integración-con-pre-commit-y-ci)
16. [Migrar desde flake8, isort y black](#16-migrar-desde-flake8-isort-y-black)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**Ruff** es un *linter* (analizador de código) y **formateador** para Python, desarrollado por **Astral**. En una sola herramienta, extremadamente rápida, reúne lo que antes requería varias:

| Herramienta anterior | Equivalente en Ruff |
|---|---|
| flake8 y sus *plugins* (bugbear, comprehensions, simplify…) | `ruff check` |
| isort | `ruff check --select I --fix` |
| black | `ruff format` |
| pyupgrade | Reglas `UP` |
| pydocstyle | Reglas `D` |
| autoflake | Reglas `F401`, `F841` con `--fix` |
| Partes de pylint y bandit | Reglas `PL`, `S` |

**Ventajas:**
- Entre 10 y 100 veces más rápido que flake8 o black: analiza proyectos grandes en milisegundos.
- Más de 800 reglas, muchas con corrección automática.
- Configuración única en `pyproject.toml`.
- Formato compatible con black al 99,9 %.
- Servidor de lenguaje integrado para el editor.

**Limitaciones:**
- No comprueba tipos (use mypy, Pyright o ty).
- No implementa todas las reglas de pylint (las que requieren análisis de tipos entre módulos).

---

## 2. Instalación

Como dependencia de desarrollo del proyecto (recomendado, para fijar la versión):

```bash
uv add --dev ruff
poetry add --group dev ruff
python -m pip install ruff
```

Como herramienta global o puntual:

```bash
uv tool install ruff
uvx ruff check .
pipx install ruff
```

Comprobar:

```bash
ruff version
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Regla | Comprobación concreta, con un código: `F401` (*import* sin usar). |
| Prefijo | Grupo de reglas: `F` (Pyflakes), `E` (pycodestyle), `B` (bugbear)… |
| Diagnóstico | Problema encontrado: archivo, línea, código y mensaje. |
| Corrección segura | Cambio automático que no altera el comportamiento. |
| Corrección no segura | Cambio que podría alterar el comportamiento; requiere `--unsafe-fixes`. |
| Formateador | Reescribe el estilo (espacios, comillas, saltos) sin cambiar la lógica. |
| `noqa` | Comentario que silencia un diagnóstico en una línea. |
| Modo *preview* | Reglas y comportamientos nuevos aún no estabilizados. |

---

## 4. Analizar código: `ruff check`

```bash
ruff check                          # Directorio actual (recursivo)
ruff check src/ tests/
ruff check archivo.py
ruff check --select E,F,I           # Solo esas reglas (sobrescribe la configuración)
ruff check --extend-select B        # Añade reglas a las configuradas
ruff check --ignore E501
ruff check --statistics             # Recuento por regla
ruff check --watch                  # Vuelve a analizar al guardar
ruff check --output-format concise  # Una línea por diagnóstico
ruff check --output-format json     # Para integraciones
ruff check --output-format github   # Anotaciones en GitHub Actions
ruff check --exit-zero              # No falla aunque haya diagnósticos
```

Ejemplo de salida:

```text
src/facturas/servicio.py:3:8: F401 [*] `os` imported but unused
  |
1 | import json
2 | import logging
3 | import os
  |        ^^ F401
  |
  = help: Remove unused import: `os`

src/facturas/servicio.py:14:5: B006 Do not use mutable data structures for argument defaults
   |
14 | def crear(lineas=[]):
   |                  ^^ B006
   |
   = help: Replace with `None`; initialize within function

Found 2 errors.
[*] 1 fixable with the `--fix` option.
```

Obtener información de una regla:

```bash
ruff rule F401              # Explicación, ejemplos y si es corregible
ruff rule --all             # Todas las reglas
ruff linter                 # Lista de familias (prefijos)
```

---

## 5. Correcciones automáticas

```bash
ruff check --fix                    # Aplica las correcciones seguras
ruff check --fix --unsafe-fixes     # También las no seguras (revise el resultado)
ruff check --diff                   # Muestra los cambios sin aplicarlos
ruff check --fix --show-fixes       # Lista lo que ha corregido
ruff check --select I --fix         # Solo ordenar imports
ruff check --add-noqa               # Añade «# noqa» a todos los diagnósticos actuales
```

Controlar qué se corrige:

```toml
[tool.ruff.lint]
fixable = ["ALL"]
unfixable = ["F401"]            # No eliminar imports automáticamente (p. ej. en __init__.py)

[tool.ruff.lint.extend-safe-fixes]  # Convertir correcciones no seguras en seguras
# extend-safe-fixes = ["UP"]
```

---

## 6. Formatear código: `ruff format`

```bash
ruff format                     # Formatea todo el proyecto
ruff format src/archivo.py
ruff format --check             # Falla si algo no está formateado (CI)
ruff format --diff              # Muestra los cambios sin aplicarlos
ruff format --line-length 100
```

Configuración del formateador:

```toml
[tool.ruff.format]
quote-style = "double"              # "single" o "preserve"
indent-style = "space"              # "tab"
line-ending = "auto"                # "lf", "cr-lf", "native"
skip-magic-trailing-comma = false   # Respeta la coma final para forzar una línea por elemento
docstring-code-format = true        # Formatea el código de ejemplo dentro de los docstrings
docstring-code-line-length = "dynamic"
```

La **coma final mágica**: si una lista termina en coma, el formateador mantiene un elemento por línea:

```python
# Se compacta
colores = ["rojo", "verde", "azul"]

# Se mantiene expandida gracias a la coma final
colores = [
    "rojo",
    "verde",
    "azul",
]
```

Desactivar el formato en un fragmento:

```python
# fmt: off
matriz = [
    1, 0, 0,
    0, 1, 0,
    0, 0, 1,
]
# fmt: on

tabla = {"a": 1,   "b": 2}  # fmt: skip
```

> Ejecute primero `ruff check --fix` y después `ruff format`: algunas correcciones pueden dejar código que el formateador reorganiza.

---

## 7. Configuración general

Ruff lee `pyproject.toml` (`[tool.ruff]`), `ruff.toml` o `.ruff.toml` (sin el prefijo `tool.ruff`).

```toml
[tool.ruff]
line-length = 100
target-version = "py312"           # Se deduce de requires-python si no se indica
src = ["src", "tests"]             # Para clasificar imports propios
extend-exclude = ["migraciones", "src/generado"]
respect-gitignore = true
preview = false
fix = false                        # Si es true, «ruff check» corrige siempre
show-fixes = true
output-format = "full"

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "UP"]
ignore = []

[tool.ruff.format]
quote-style = "double"
```

| Opción | Uso |
|---|---|
| `line-length` | Longitud máxima (formateador y regla `E501`). Por defecto 88. |
| `target-version` | Versión mínima de Python: afecta a `UP` y al formateador. |
| `exclude` / `extend-exclude` | Rutas excluidas (`extend-` añade a las predeterminadas). |
| `src` | Carpetas con código propio (para `isort`). |
| `extend` | Heredar de otro archivo de configuración: `extend = "../ruff-base.toml"`. |
| `required-version` | Versión mínima de Ruff para el proyecto. |

Configuración jerárquica: Ruff usa el archivo de configuración **más cercano** a cada archivo analizado. Un subdirectorio puede tener su propio `ruff.toml` con `extend = "../pyproject.toml"`.

---

## 8. Selección de reglas

| Opción | Efecto |
|---|---|
| `select` | Conjunto de reglas activas (sustituye al predeterminado `["E4", "E7", "E9", "F"]`). |
| `extend-select` | Añade reglas al conjunto. |
| `ignore` | Desactiva reglas concretas. |
| `per-file-ignores` | Desactiva reglas en rutas concretas. |

Los selectores admiten distintos niveles:

```toml
select = [
    "E",        # Toda la familia pycodestyle-errores
    "E5",       # Solo E5xx
    "E501",     # Una regla concreta
    "ALL",      # Todas (combinar con ignore; nuevas versiones pueden añadir reglas)
]
```

> Elegir `"ALL"` es exhaustivo pero cada actualización de Ruff puede activar reglas nuevas. En proyectos de equipo, una lista explícita es más predecible.

---

## 9. Familias de reglas recomendadas

| Prefijo | Origen | Detecta |
|---|---|---|
| `F` | Pyflakes | Imports y variables sin usar, nombres no definidos. **Imprescindible.** |
| `E`, `W` | pycodestyle | Errores y avisos de estilo PEP 8. |
| `I` | isort | Orden de los *imports*. |
| `B` | flake8-bugbear | Errores sutiles: argumentos mutables por defecto, `except:` amplios… |
| `UP` | pyupgrade | Sintaxis antigua actualizable (`List[int]` → `list[int]`, f-strings). |
| `SIM` | flake8-simplify | Código simplificable. |
| `C4` | flake8-comprehensions | Comprensiones innecesarias o mejorables. |
| `N` | pep8-naming | Convenciones de nombres. |
| `S` | flake8-bandit | Problemas de seguridad (contraseñas fijas, `eval`, SQL concatenado). |
| `PT` | flake8-pytest-style | Estilo de las pruebas con pytest. |
| `RUF` | Ruff | Reglas propias (ambigüedad de caracteres, `mutable-class-default`…). |
| `PL` | Pylint | Complejidad, número de argumentos, comparaciones mágicas. |
| `D` | pydocstyle | Estilo de los *docstrings*. |
| `ANN` | flake8-annotations | Funciones sin anotaciones de tipo. |
| `T20` | flake8-print | `print()` olvidados. |
| `ERA` | eradicate | Código comentado. |
| `DTZ` | flake8-datetimez | `datetime` sin zona horaria. |
| `PTH` | flake8-use-pathlib | `os.path` sustituible por `pathlib`. |
| `TRY` | tryceratops | Malas prácticas con excepciones. |
| `PERF` | Perflint | Patrones lentos. |
| `FURB` | refurb | Modernizaciones idiomáticas. |
| `ASYNC` | flake8-async | Errores en código asíncrono (llamadas bloqueantes). |
| `C90` | mccabe | Complejidad ciclomática. |

Configuraciones de partida:

```toml
# Básica (cualquier proyecto)
select = ["E", "W", "F", "I", "B", "UP"]

# Recomendada (proyecto profesional)
select = ["E", "W", "F", "I", "B", "UP", "SIM", "C4", "N", "S", "PT", "RUF", "PL", "DTZ", "PTH", "PERF"]
ignore = ["PLR2004"]          # Números mágicos en comparaciones (demasiado ruidosa)
```

---

## 10. Ajustes por regla y por archivo

```toml
[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]                         # Reexportaciones
"tests/**/*.py" = ["S101", "PLR2004", "ANN"]     # assert, números mágicos, anotaciones
"scripts/*.py" = ["T201"]                        # print permitido
"migraciones/*.py" = ["E501"]

[tool.ruff.lint.pydocstyle]
convention = "google"                            # "numpy" o "pep257"

[tool.ruff.lint.mccabe]
max-complexity = 10

[tool.ruff.lint.pylint]
max-args = 6
max-branches = 12

[tool.ruff.lint.flake8-bugbear]
extend-immutable-calls = ["fastapi.Depends", "fastapi.Query"]   # Evita falsos B008 en FastAPI

[tool.ruff.lint.pep8-naming]
classmethod-decorators = ["pydantic.field_validator", "pydantic.model_validator"]

[tool.ruff.lint.flake8-tidy-imports]
ban-relative-imports = "all"

[tool.ruff.lint.flake8-tidy-imports.banned-api]
"requests".msg = "Use httpx, que es la librería estándar del proyecto."
"datetime.datetime.utcnow".msg = "Use datetime.now(UTC)."
```

---

## 11. Ordenar *imports* (sustituto de isort)

```bash
ruff check --select I --fix
```

```toml
[tool.ruff.lint]
extend-select = ["I"]

[tool.ruff.lint.isort]
known-first-party = ["facturas"]
known-third-party = ["fastapi"]
section-order = ["future", "standard-library", "third-party", "first-party", "local-folder"]
combine-as-imports = true
force-sort-within-sections = false
lines-after-imports = 2
required-imports = ["from __future__ import annotations"]
```

Resultado:

```python
from __future__ import annotations

import json
import logging
from pathlib import Path

import httpx
from pydantic import BaseModel

from facturas.modelos import Factura
from facturas.servicio import calcular_total

from .utilidades import formatear
```

---

## 12. Supresión de avisos (`noqa`)

```python
import os  # noqa: F401                        # Una regla
x = eval(expr)  # noqa: S307, PGH001           # Varias
resultado = funcion_antigua()  # noqa          # Todas (evítelo)
```

Todo el archivo (al principio):

```python
# ruff: noqa: E501
# ruff: noqa                                   # Desactiva Ruff en todo el archivo
```

Mantenimiento de las supresiones:

```toml
[tool.ruff.lint]
extend-select = ["RUF100"]          # Avisa de «noqa» que ya no son necesarios
external = ["V"]                    # Códigos de otras herramientas que no debe marcar como desconocidos
```

```bash
ruff check --extend-select RUF100 --fix     # Elimina los noqa sobrantes
```

---

## 13. Notebooks de Jupyter

Ruff analiza y formatea archivos `.ipynb` por defecto:

```bash
ruff check notebooks/
ruff format notebooks/
```

```toml
[tool.ruff.lint.per-file-ignores]
"*.ipynb" = ["E402", "T201", "B018"]     # Imports en celdas, print, expresiones sueltas
```

---

## 14. Integración con editores

### 14.1 VS Code

Extensión oficial **Ruff** (`charliermarsh.ruff`):

```json
// .vscode/settings.json
{
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  },
  "ruff.importStrategy": "fromEnvironment"
}
```

### 14.2 PyCharm

Desde PyCharm 2025.1 hay integración nativa con Ruff (*Settings → Tools → Ruff*); en versiones anteriores, *plugin* «Ruff».

### 14.3 Otros editores

`ruff server` es un servidor LSP estándar: configúrelo en Neovim, Helix, Zed, Emacs o Sublime Text.

---

## 15. Integración con pre-commit y CI

### 15.1 pre-commit

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.16.9                    # Use la misma versión que en el proyecto
    hooks:
      - id: ruff-check
        args: [--fix]
      - id: ruff-format
```

> Ponga `ruff-check` con `--fix` **antes** de `ruff-format`. En versiones antiguas del repositorio de *hooks*, el identificador era `ruff` en lugar de `ruff-check`.

### 15.2 GitHub Actions

```yaml
- uses: astral-sh/ruff-action@v3       # Instala y ejecuta Ruff
  with:
    args: "check --output-format github"
- uses: astral-sh/ruff-action@v3
  with:
    args: "format --check --diff"
```

O con el entorno del proyecto:

```yaml
- run: uv run ruff check --output-format github
- run: uv run ruff format --check --diff
```

### 15.3 GitLab CI (informe de calidad de código)

```yaml
ruff:
  script:
    - uv run ruff check --output-format gitlab > gl-code-quality-report.json
  artifacts:
    reports:
      codequality: gl-code-quality-report.json
```

---

## 16. Migrar desde flake8, isort y black

| Antes | Ahora |
|---|---|
| `flake8 .` | `ruff check` |
| `isort .` | `ruff check --select I --fix` |
| `black .` | `ruff format` |
| `black --check .` | `ruff format --check` |
| `.flake8` / `setup.cfg [flake8]` | `[tool.ruff.lint]` |
| `max-line-length` | `line-length` |
| `extend-ignore = E203` | No necesario: Ruff no emite E203 incompatible con el formateador |
| `per-file-ignores` | `[tool.ruff.lint.per-file-ignores]` |
| `[tool.isort] profile = "black"` | No necesario: compatible por defecto |
| `[tool.black] line-length` | `[tool.ruff] line-length` |

Pasos:

1. Traduzca la configuración (la herramienta `flake8-to-ruff` ayuda con `.flake8`).
2. Ejecute `ruff check --statistics` para ver el volumen de avisos.
3. Ejecute `ruff format` y haga un *commit* separado solo de formato.
4. Añada ese *commit* a `.git-blame-ignore-revs` para no contaminar `git blame`.
5. Sustituya los *hooks* de pre-commit y los pasos de CI.

```bash
git rev-parse HEAD >> .git-blame-ignore-revs
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

---

## 17. Ejemplo práctico completo

**Adoptar Ruff en un proyecto existente** con un conjunto de reglas profesional: configuración completa, un archivo «antes» con problemas típicos, el resultado tras las correcciones automáticas y un script que convierte la salida JSON de Ruff en un informe Markdown por familia de reglas para revisar con el equipo.

### 17.1 Configuración (`pyproject.toml`)

```toml
[tool.ruff]
line-length = 100
target-version = "py312"
src = ["src", "tests"]
extend-exclude = ["src/facturas/generado"]

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "UP", "SIM", "C4", "N", "S", "PT", "RUF", "PL", "DTZ", "PTH", "T20"]
ignore = ["PLR2004"]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101", "PLR0913"]
"scripts/**" = ["T201"]

[tool.ruff.lint.isort]
known-first-party = ["facturas"]

[tool.ruff.lint.pylint]
max-args = 6

[tool.ruff.format]
docstring-code-format = true
```

### 17.2 Código antes (`src/facturas/informe.py`)

```python
import os, sys
from typing import List, Dict, Optional
import json
from datetime import datetime

def generar(facturas: List[Dict], destino: Optional[str] = None, filtros = {}):
    print("Generando informe...")
    total = 0
    for f in facturas:
        if f.get("pagada") == True:
            total = total + f["importe"]
    resultado = dict([(f["numero"], f["importe"]) for f in facturas])
    fecha = datetime.now()
    if destino != None:
        with open(os.path.join(destino, "informe.json"), "w") as fh:
            fh.write(json.dumps({"total": total, "detalle": resultado, "fecha": str(fecha)}))
    try:
        enviar(resultado)
    except:
        pass
    return total
```

```bash
$ ruff check src/facturas/informe.py --statistics
2	UP035 	[ ] deprecated-import
2	UP006 	[*] non-pep585-annotation
1	E722  	[ ] bare-except
1	DTZ005	[ ] call-datetime-now-without-tzinfo
1	B006  	[ ] mutable-argument-default
1	UP045 	[*] non-pep604-annotation-optional
1	S110  	[ ] try-except-pass
1	F821  	[ ] undefined-name
1	C404  	[ ] unnecessary-list-comprehension-dict
1	I001  	[*] unsorted-imports
1	F401  	[*] unused-import
Found 13 errors.
[*] 5 fixable with the `--fix` option (2 hidden fixes can be enabled with the `--unsafe-fixes` option).
```

### 17.3 Código después (`ruff check --fix --unsafe-fixes` + correcciones manuales + `ruff format`)

Las reglas sin corrección automática (`B006`, `E722`, `S110`, `DTZ005`, `F821`) se resuelven a mano. Además, se aprovecha para sustituir `print` por `logging` y `os.path` por `pathlib`. El resultado pasa `ruff check` y `ruff format --check` sin avisos:

```python
import json
import logging
from datetime import UTC, datetime
from pathlib import Path

from facturas.envio import enviar

log = logging.getLogger(__name__)


def generar(
    facturas: list[dict], destino: str | None = None, filtros: dict | None = None
) -> float:
    log.info("Generando informe...")
    filtros = filtros or {}
    total = sum(f["importe"] for f in facturas if f.get("pagada"))
    resultado = {f["numero"]: f["importe"] for f in facturas}
    fecha = datetime.now(UTC)
    if destino is not None:
        datos = {"total": total, "detalle": resultado, "fecha": fecha.isoformat()}
        (Path(destino) / "informe.json").write_text(json.dumps(datos), encoding="utf-8")
    try:
        enviar(resultado)
    except ConnectionError:
        log.exception("No se pudo enviar el informe")
    return total
```

### 17.4 Informe por familias (`scripts/informe_ruff.py`)

```python
"""
Genera un informe Markdown con los diagnósticos de Ruff agrupados por familia.

    uv run python scripts/informe_ruff.py > informe-ruff.md
"""
from __future__ import annotations

import json
import re
import subprocess
import sys
from collections import Counter, defaultdict
from pathlib import Path


def ejecutar_ruff(rutas: list[str]) -> list[dict]:
    proceso = subprocess.run(
        [sys.executable, "-m", "ruff", "check", "--output-format", "json", "--exit-zero", *rutas],
        capture_output=True, text=True, check=True,
    )
    return json.loads(proceso.stdout)


def familia(codigo: str | None) -> str:
    if not codigo:
        return "Sintaxis"
    return re.match(r"[A-Z]+", codigo).group(0)  # type: ignore[union-attr]


def main() -> None:
    diagnosticos = ejecutar_ruff(sys.argv[1:] or ["."])
    por_familia: dict[str, Counter[str]] = defaultdict(Counter)
    corregibles = 0
    archivos: Counter[str] = Counter()
    descripcion: dict[str, str] = {}

    for d in diagnosticos:
        codigo = d.get("code") or "-"
        por_familia[familia(d.get("code"))][codigo] += 1
        descripcion.setdefault(codigo, d["message"])
        archivos[str(Path(d["filename"]).relative_to(Path.cwd()))] += 1
        corregibles += d.get("fix") is not None

    print("# Informe de Ruff\n")
    print(f"- **Diagnósticos:** {len(diagnosticos)}")
    print(f"- **Corregibles automáticamente:** {corregibles}")
    print(f"- **Archivos afectados:** {len(archivos)}\n")

    print("## Por familia\n")
    print("| Familia | Total | Reglas más frecuentes |")
    print("|---|---:|---|")
    for fam, contador in sorted(por_familia.items(), key=lambda x: -sum(x[1].values())):
        top = ", ".join(f"`{c}` ({n})" for c, n in contador.most_common(3))
        print(f"| {fam} | {sum(contador.values())} | {top} |")

    print("\n## Detalle de reglas\n")
    print("| Regla | Veces | Ejemplo de mensaje |")
    print("|---|---:|---|")
    total_por_regla = Counter({c: n for cont in por_familia.values() for c, n in cont.items()})
    for codigo, n in total_por_regla.most_common():
        mensaje = descripcion[codigo].replace("|", "\\|")
        print(f"| `{codigo}` | {n} | {mensaje} |")

    print("\n## Archivos con más diagnósticos\n")
    for archivo, n in archivos.most_common(10):
        print(f"- `{archivo}`: {n}")


if __name__ == "__main__":
    main()
```

Flujo de adopción:

```bash
uv add --dev ruff
uv run python scripts/informe_ruff.py > informe-ruff.md     # Revisar con el equipo
uv run ruff check --fix                                     # Correcciones seguras
uv run ruff format                                          # Commit separado de formato
uv run ruff check --add-noqa                                # Opcional: congelar lo pendiente
pre-commit install                                          # A partir de aquí, sin regresiones
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ruff failed: Failed to parse pyproject.toml` | Opción inexistente o mal ubicada (p. ej. `select` fuera de `[tool.ruff.lint]`). | Revise la documentación de configuración de su versión. |
| Avisos de opciones obsoletas en la raíz de `[tool.ruff]` | Opciones de *lint* fuera de `[tool.ruff.lint]`. | Muévalas a `[tool.ruff.lint]`. |
| `ruff format` y `ruff check` se contradicen (E501, comillas) | Reglas de estilo incompatibles con el formateador. | Ignore `E501` o deje que el formateador decida; Ruff avisa de conflictos. |
| Los imports propios se clasifican como de terceros | Ruff no sabe qué es código propio. | `src = ["src"]` o `known-first-party`. |
| `F401` elimina un import necesario | Reexportación en `__init__.py`. | Use `__all__` o `import x as x`, o `per-file-ignores`. |
| `B008` en parámetros de FastAPI | `Depends()` en valores por defecto. | `extend-immutable-calls` o use `Annotated`. |
| Resultados distintos en local y en CI | Versiones diferentes de Ruff. | Fije la versión en el proyecto y en pre-commit. |
| Tras actualizar Ruff aparecen avisos nuevos | `select = ["ALL"]` o reglas que salieron de *preview*. | Lista explícita de reglas; revise las notas de la versión. |
| Un archivo no se analiza | Está en `.gitignore` o en `exclude`. | `ruff check --show-files` para verlo; `--no-respect-gitignore`. |

---

## 19. Buenas prácticas

- Instale Ruff como **dependencia de desarrollo** y fije la misma versión en pre-commit.
- Use `ruff check` + `ruff format`; elimine flake8, isort y black del proyecto.
- Configure una **lista explícita** de reglas y amplíela gradualmente.
- Incluya siempre `F`, `E`, `I`, `B` y `UP`; para seguridad, `S`.
- Use `per-file-ignores` para pruebas y scripts en lugar de relajar todo el proyecto.
- Supresiones con código concreto (`# noqa: S101`) y `RUF100` para limpiar las sobrantes.
- Formatee y corrija al guardar en el editor; compruebe en pre-commit y CI.
- Haga el primer formateo masivo en un *commit* aislado y regístrelo en `.git-blame-ignore-revs`.
- Revise `ruff rule <código>` antes de ignorar una regla: la explicación suele incluir el motivo y la alternativa.

---

## 20. Referencia rápida

```bash
ruff check [rutas] [--fix] [--unsafe-fixes] [--diff] [--watch] [--statistics]
ruff check --select I --fix                 # Ordenar imports
ruff check --output-format concise|json|github|gitlab
ruff check --add-noqa  ·  --extend-select RUF100 --fix
ruff format [rutas] [--check] [--diff]
ruff rule F401  ·  ruff linter  ·  ruff server  ·  ruff version
```

```toml
[tool.ruff]               line-length, target-version, src, extend-exclude
[tool.ruff.lint]          select, extend-select, ignore, fixable, unfixable
[tool.ruff.lint.per-file-ignores]   "tests/**" = ["S101"]
[tool.ruff.lint.isort]    known-first-party
[tool.ruff.format]        quote-style, docstring-code-format
```

```python
# noqa: F401   ·   # ruff: noqa: E501   ·   # fmt: off / # fmt: on   ·   # fmt: skip
```

---

## 21. Ejercicios propuestos

1. Ejecute `ruff check --statistics --select ALL` en un proyecto propio e identifique las cinco reglas con más avisos.
2. Migre un proyecto que use flake8, isort y black a Ruff y compare los tiempos de ejecución.
3. Configure `banned-api` para prohibir `requests` y `datetime.utcnow` en el proyecto.
4. Configure VS Code para corregir y formatear al guardar con Ruff.
5. Amplíe el script de informe para generar también un CSV y un gráfico de barras por familia.

---

## 22. Referencias

- Documentación oficial: https://docs.astral.sh/ruff/
- Reglas: https://docs.astral.sh/ruff/rules/
- Configuración: https://docs.astral.sh/ruff/settings/
- Formateador: https://docs.astral.sh/ruff/formatter/
- Integraciones (editores, pre-commit, CI): https://docs.astral.sh/ruff/integrations/
- ruff-pre-commit: https://github.com/astral-sh/ruff-pre-commit
