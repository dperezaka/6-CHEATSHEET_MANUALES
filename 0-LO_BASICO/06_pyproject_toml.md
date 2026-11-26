# Manual de uso: pyproject.toml

> **Categoría:** Configuración · **Utilidad:** Archivo estándar con los metadatos, las dependencias y los ajustes de las herramientas
> **Tipo:** Estándar del ecosistema Python (PEP 517, 518, 621, 639, 735)

---

## Índice

1. [Descripción](#1-descripción)
2. [Ubicación y herramientas que lo usan](#2-ubicación-y-herramientas-que-lo-usan)
3. [Conceptos clave](#3-conceptos-clave)
4. [Sintaxis TOML esencial](#4-sintaxis-toml-esencial)
5. [La tabla `[build-system]`](#5-la-tabla-build-system)
6. [La tabla `[project]`: metadatos](#6-la-tabla-project-metadatos)
7. [Dependencias y extras](#7-dependencias-y-extras)
8. [Grupos de dependencias (`[dependency-groups]`)](#8-grupos-de-dependencias-dependency-groups)
9. [Puntos de entrada y scripts](#9-puntos-de-entrada-y-scripts)
10. [Campos dinámicos](#10-campos-dinámicos)
11. [Configuración de los *backends*](#11-configuración-de-los-backends)
12. [La tabla `[tool]`: configuración de herramientas](#12-la-tabla-tool-configuración-de-herramientas)
13. [Leer `pyproject.toml` desde Python](#13-leer-pyprojecttoml-desde-python)
14. [Validar el archivo](#14-validar-el-archivo)
15. [Migrar desde `setup.py` y `setup.cfg`](#15-migrar-desde-setuppy-y-setupcfg)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**`pyproject.toml`** es el archivo de configuración estándar de un proyecto Python. En un solo lugar reúne:

- **cómo se construye** el paquete (`[build-system]`);
- **qué es** el proyecto: nombre, versión, autores, licencia, dependencias (`[project]`);
- **las dependencias de desarrollo** (`[dependency-groups]`);
- **la configuración de las herramientas**: Ruff, mypy, pytest, coverage, uv, Poetry… (`[tool.*]`).

**Ventajas:**
- Formato declarativo: no ejecuta código, a diferencia de `setup.py`.
- Independiente de la herramienta: pip, uv, Poetry, Hatch y PDM leen la misma tabla `[project]`.
- Sustituye a `setup.py`, `setup.cfg`, `requirements.txt` (para dependencias directas), `pytest.ini`, `mypy.ini`, `.coveragerc`, `.isort.cfg`, etc.

**Limitaciones:**
- No es un archivo de bloqueo: declara rangos, no versiones exactas (eso es `uv.lock`, `poetry.lock` o `pylock.toml`).
- Algunas herramientas aún requieren su propio archivo (p. ej. `.pre-commit-config.yaml`, `tox.ini` en configuraciones antiguas).

---

## 2. Ubicación y herramientas que lo usan

El archivo se coloca en la **raíz del proyecto**:

```
mi_proyecto/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── mi_paquete/
│       └── __init__.py
└── tests/
```

| Herramienta | Tabla que lee |
|---|---|
| pip, build | `[build-system]`, `[project]` |
| uv | `[project]`, `[dependency-groups]`, `[tool.uv]` |
| Poetry 2 | `[project]`, `[tool.poetry]` |
| Hatch / hatchling | `[project]`, `[tool.hatch]` |
| Ruff | `[tool.ruff]` |
| mypy | `[tool.mypy]` |
| Pyright | `[tool.pyright]` |
| ty | `[tool.ty]` |
| pytest | `[tool.pytest.ini_options]` (o `[tool.pytest]` en pytest 9+) |
| coverage | `[tool.coverage.*]` |
| Bandit | `[tool.bandit]` |
| Hypothesis | — (perfiles en código) |
| tox | `[tool.tox]` |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Frontend* de construcción | Herramienta que pide construir el paquete (pip, uv, build). |
| *Backend* de construcción | Librería que genera el *wheel* y el *sdist* (hatchling, setuptools…). |
| Metadatos estáticos | Valores escritos directamente en `[project]`. |
| Metadatos dinámicos | Valores que calcula el *backend* (p. ej. la versión desde Git). |
| Dependencias | Lo que el paquete necesita para funcionar (se publican). |
| Extras | Dependencias opcionales para los usuarios (`paquete[pdf]`). |
| Grupos de dependencias | Dependencias de desarrollo (no se publican). |
| Punto de entrada | Ejecutable o *plugin* que expone el paquete. |

---

## 4. Sintaxis TOML esencial

```toml
# Comentario
cadena = "texto"
literal = 'C:\ruta\sin\escapes'
multilinea = """
Varias
líneas"""
entero = 42
decimal = 3.14
booleano = true
fecha = 2026-11-26
lista = ["a", "b", "c"]
lista_larga = [
    "uno",
    "dos",      # Se admite coma final
]
tabla_en_linea = { nombre = "Ana", email = "ana@ejemplo.com" }

[tabla]
clave = "valor"

[tabla.subtabla]          # Equivale a tabla = { subtabla = { ... } }
otra = 1

[[lista_de_tablas]]       # Cada [[...]] añade un elemento a la lista
nombre = "primero"

[[lista_de_tablas]]
nombre = "segundo"
```

> Una tabla no puede declararse dos veces. Las claves con puntos (`a.b = 1`) y las tablas (`[a]`) deben ser coherentes entre sí.

---

## 5. La tabla `[build-system]`

Indica qué *backend* construye el paquete (PEP 517/518):

```toml
[build-system]
requires = ["hatchling>=1.26"]
build-backend = "hatchling.build"
```

| *Backend* | `requires` | `build-backend` | Indicado para |
|---|---|---|---|
| Hatchling | `["hatchling"]` | `hatchling.build` | Uso general, recomendado por la guía oficial. |
| uv_build | `["uv_build>=0.9,<0.14"]` | `uv_build` | Proyectos gestionados con uv (Python puro). |
| setuptools | `["setuptools>=75"]` | `setuptools.build_meta` | Proyectos existentes, extensiones C. |
| Poetry | `["poetry-core>=2,<3"]` | `poetry.core.masonry.api` | Proyectos gestionados con Poetry. |
| Flit | `["flit_core>=3.10,<4"]` | `flit_core.buildapi` | Paquetes simples de Python puro. |
| PDM | `["pdm-backend"]` | `pdm.backend` | Proyectos gestionados con PDM. |
| maturin | `["maturin>=1.7,<2"]` | `maturin` | Extensiones en Rust. |
| scikit-build-core | `["scikit-build-core"]` | `scikit_build_core.build` | Extensiones C/C++ con CMake. |

> Si el proyecto no se va a empaquetar (una aplicación que solo se ejecuta), `[build-system]` puede omitirse con uv; con Poetry, use `package-mode = false`.

---

## 6. La tabla `[project]`: metadatos

```toml
[project]
name = "gestor-facturas"                 # Nombre en PyPI (se normaliza: _ . - equivalen)
version = "1.4.0"                        # PEP 440
description = "Generación y envío de facturas electrónicas"
readme = "README.md"                     # O { file = "README.rst", content-type = "text/x-rst" }
requires-python = ">=3.12"
license = "MIT"                          # Expresión SPDX (PEP 639)
license-files = ["LICENSE", "AUTHORS*"]
authors = [
    { name = "Ana García", email = "ana@empresa.com" },
    { name = "Equipo Finanzas" },
]
maintainers = [{ name = "Luis Pérez", email = "luis@empresa.com" }]
keywords = ["facturación", "pdf", "factura-e"]
classifiers = [
    "Development Status :: 5 - Production/Stable",
    "Intended Audience :: Developers",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.12",
    "Programming Language :: Python :: 3.13",
    "Typing :: Typed",
    "Operating System :: OS Independent",
]

[project.urls]
Homepage = "https://empresa.com/facturas"
Documentation = "https://docs.empresa.com/facturas"
Repository = "https://github.com/empresa/gestor-facturas"
Issues = "https://github.com/empresa/gestor-facturas/issues"
Changelog = "https://github.com/empresa/gestor-facturas/blob/main/CHANGELOG.md"
```

| Campo | Obligatorio | Notas |
|---|---|---|
| `name` | Sí | Debe ser único en PyPI si se publica. |
| `version` | Sí (o `dynamic`) | Formato PEP 440: `1.0.0`, `1.0.0rc1`, `1.0.0.post1`. |
| `requires-python` | Recomendado | Limita las versiones de Python compatibles. |
| `license` | Recomendado | Expresión SPDX: `"MIT"`, `"Apache-2.0"`, `"MIT OR Apache-2.0"`. |
| `readme` | Recomendado | Se muestra como descripción en PyPI. |
| `classifiers` | Opcional | Lista oficial en https://pypi.org/classifiers/. |

> La forma antigua `license = { text = "MIT" }` y los clasificadores `License :: ...` están **obsoletos** con PEP 639. Use la cadena SPDX y `license-files`.

---

## 7. Dependencias y extras

```toml
[project]
dependencies = [
    "httpx>=0.27,<1",
    "pydantic>=2.9",
    "jinja2~=3.1",
    "tomli>=2 ; python_version < '3.11'",
    "pywin32>=308 ; sys_platform == 'win32'",
]

[project.optional-dependencies]
pdf = ["weasyprint>=63"]
excel = ["openpyxl>=3.1"]
todo = ["gestor-facturas[pdf,excel]"]     # Un extra puede incluir otros
```

Instalación:

```bash
pip install gestor-facturas
pip install "gestor-facturas[pdf]"
pip install "gestor-facturas[todo]"
```

Reglas de estilo para las restricciones:

| Tipo de proyecto | Recomendación |
|---|---|
| Librería | Límites inferiores (`>=`) y superiores solo si hay incompatibilidad conocida. |
| Aplicación | Rangos razonables en `pyproject.toml` + versiones exactas en el archivo de bloqueo. |

---

## 8. Grupos de dependencias (`[dependency-groups]`)

Estándar **PEP 735** para dependencias que **no** forman parte del paquete publicado (pruebas, linters, documentación):

```toml
[dependency-groups]
test = ["pytest>=8.3", "pytest-cov>=6", "hypothesis>=6.120"]
lint = ["ruff>=0.8", "mypy>=1.13"]
docs = ["mkdocs-material>=9.5", "mkdocstrings[python]>=0.27"]
dev = [
    { include-group = "test" },
    { include-group = "lint" },
]
```

```bash
uv sync --group docs           # uv
pip install --group dev        # pip 25.1+
poetry install --with dev      # Poetry (también admite [tool.poetry.group.*])
```

| | `optional-dependencies` (extras) | `dependency-groups` |
|---|---|---|
| ¿Se publican en PyPI? | Sí | No |
| ¿Para quién? | Usuarios del paquete | Desarrolladores del proyecto |
| ¿Instalan el proyecto? | Sí | No necesariamente |

---

## 9. Puntos de entrada y scripts

```toml
[project.scripts]                       # Ejecutables de terminal
facturar = "gestor_facturas.cli:main"
facturas-admin = "gestor_facturas.admin:app"

[project.gui-scripts]                   # Aplicaciones gráficas (sin consola en Windows)
facturas-gui = "gestor_facturas.gui:iniciar"

[project.entry-points."pytest11"]       # Plugins para otras aplicaciones
facturas = "gestor_facturas.pytest_plugin"

[project.entry-points."gestor_facturas.exportadores"]   # Sistema de plugins propio
csv = "gestor_facturas.exportar:ExportadorCSV"
```

Leer *plugins* registrados por otros paquetes:

```python
from importlib.metadata import entry_points

for ep in entry_points(group="gestor_facturas.exportadores"):
    clase = ep.load()
    print(ep.name, clase)
```

---

## 10. Campos dinámicos

Algunos campos pueden calcularlos el *backend*. Deben declararse en `dynamic`:

```toml
[project]
name = "gestor-facturas"
dynamic = ["version"]

# Hatchling: versión desde un archivo
[tool.hatch.version]
path = "src/gestor_facturas/__about__.py"
```

Versión desde las etiquetas de Git:

```toml
[build-system]
requires = ["hatchling", "hatch-vcs"]
build-backend = "hatchling.build"

[project]
dynamic = ["version"]

[tool.hatch.version]
source = "vcs"
```

```toml
# Equivalente con setuptools
[build-system]
requires = ["setuptools>=75", "setuptools-scm>=8"]
build-backend = "setuptools.build_meta"

[project]
dynamic = ["version"]

[tool.setuptools_scm]
```

Leer la versión en tiempo de ejecución, sin duplicarla:

```python
from importlib.metadata import version, PackageNotFoundError

try:
    __version__ = version("gestor-facturas")
except PackageNotFoundError:          # Ejecución sin instalar
    __version__ = "0.0.0"
```

---

## 11. Configuración de los *backends*

### 11.1 Hatchling

```toml
[tool.hatch.build.targets.wheel]
packages = ["src/gestor_facturas"]

[tool.hatch.build.targets.sdist]
include = ["src", "tests", "README.md", "LICENSE"]
exclude = ["*.log", "/.github"]
```

### 11.2 setuptools

```toml
[tool.setuptools.packages.find]
where = ["src"]

[tool.setuptools.package-data]
gestor_facturas = ["plantillas/*.html", "py.typed"]
```

### 11.3 uv_build

```toml
[tool.uv.build-backend]
module-name = "gestor_facturas"
module-root = "src"
```

### 11.4 Incluir archivos de datos

Coloque los recursos dentro del paquete y léalos con `importlib.resources`:

```python
from importlib.resources import files

plantilla = files("gestor_facturas").joinpath("plantillas/factura.html").read_text(encoding="utf-8")
```

---

## 12. La tabla `[tool]`: configuración de herramientas

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM", "S"]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101"]

[tool.mypy]
strict = true
python_version = "3.12"

[[tool.mypy.overrides]]
module = ["weasyprint.*"]
ignore_missing_imports = true

[tool.pytest.ini_options]
minversion = "8.0"
testpaths = ["tests"]
addopts = ["-ra", "--strict-markers", "--import-mode=importlib"]
markers = ["lento: pruebas que tardan más de 1 s"]

[tool.coverage.run]
source = ["gestor_facturas"]
branch = true

[tool.coverage.report]
fail_under = 85
show_missing = true
exclude_also = ["if TYPE_CHECKING:", "raise NotImplementedError"]

[tool.bandit]
exclude_dirs = ["tests"]
```

> Cada herramienta documenta su propia tabla. Consulte el manual de cada una para las opciones completas.

---

## 13. Leer `pyproject.toml` desde Python

`tomllib` forma parte de la librería estándar desde Python 3.11 (solo lectura):

```python
import tomllib
from pathlib import Path

with Path("pyproject.toml").open("rb") as f:      # ¡Modo binario!
    datos = tomllib.load(f)

proyecto = datos["project"]
print(proyecto["name"], proyecto.get("version"))
print("Dependencias:", proyecto.get("dependencies", []))
print("Longitud de línea Ruff:", datos.get("tool", {}).get("ruff", {}).get("line-length"))
```

Para **escribir** TOML conservando comentarios y formato, use `tomlkit`:

```python
import tomlkit
from pathlib import Path

ruta = Path("pyproject.toml")
doc = tomlkit.parse(ruta.read_text(encoding="utf-8"))
doc["project"]["version"] = "1.5.0"
ruta.write_text(tomlkit.dumps(doc), encoding="utf-8")
```

Analizar especificadores de dependencias con `packaging`:

```python
from packaging.requirements import Requirement

req = Requirement("httpx[http2]>=0.27,<1 ; python_version >= '3.12'")
print(req.name, req.extras, req.specifier, req.marker)
print(req.specifier.contains("0.28.1"))          # True
```

---

## 14. Validar el archivo

```bash
uvx validate-pyproject pyproject.toml        # Valida contra los esquemas oficiales
uv build                                     # Construir es la prueba definitiva
uvx twine check dist/*                       # Verifica metadatos antes de publicar
poetry check                                 # En proyectos Poetry
```

Para validación en cada *commit*, añada `validate-pyproject` a pre-commit:

```yaml
- repo: https://github.com/abravalheri/validate-pyproject
  rev: v0.23
  hooks:
    - id: validate-pyproject
```

Editores: VS Code con la extensión **Even Better TOML** muestra errores y autocompletado usando el esquema de SchemaStore.

---

## 15. Migrar desde `setup.py` y `setup.cfg`

| `setup.py` / `setup.cfg` | `pyproject.toml` |
|---|---|
| `name=` | `[project] name` |
| `version=` | `[project] version` o `dynamic` |
| `install_requires=` | `[project] dependencies` |
| `extras_require=` | `[project.optional-dependencies]` |
| `python_requires=` | `[project] requires-python` |
| `entry_points={"console_scripts": [...]}` | `[project.scripts]` |
| `packages=find_packages("src")` | `[tool.setuptools.packages.find] where = ["src"]` |
| `package_data=` | `[tool.setuptools.package-data]` |
| `tests_require=` | `[dependency-groups] test` |
| `setup.cfg [flake8]`, `[isort]` | `[tool.ruff]` |
| `pytest.ini`, `setup.cfg [tool:pytest]` | `[tool.pytest.ini_options]` |
| `mypy.ini` | `[tool.mypy]` |
| `.coveragerc` | `[tool.coverage.*]` |

Herramienta de conversión automática:

```bash
uvx ini2toml setup.cfg -o pyproject.toml
```

> Si el proyecto tiene extensiones C que requieren lógica en `setup.py`, puede mantener un `setup.py` mínimo **solo** para ellas y declarar el resto en `pyproject.toml`.

---

## 16. Ejemplo práctico completo

**`pyproject.toml` de referencia** para una librería tipada con CLI, extras, grupos de desarrollo y configuración de todas las herramientas del flujo de calidad, seguido de un script que lo audita.

```toml
# ============================================================ Construcción
[build-system]
requires = ["hatchling>=1.26", "hatch-vcs>=0.4"]
build-backend = "hatchling.build"

# ============================================================ Metadatos
[project]
name = "gestor-facturas"
dynamic = ["version"]
description = "Generación, validación y envío de facturas electrónicas"
readme = "README.md"
requires-python = ">=3.12"
license = "MIT"
license-files = ["LICENSE"]
authors = [{ name = "Equipo Finanzas", email = "finanzas@empresa.com" }]
keywords = ["facturas", "pdf", "contabilidad"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Programming Language :: Python :: 3.12",
    "Programming Language :: Python :: 3.13",
    "Programming Language :: Python :: 3.14",
    "Typing :: Typed",
]
dependencies = [
    "httpx>=0.27,<1",
    "pydantic>=2.9,<3",
    "pydantic-settings>=2.6",
    "jinja2>=3.1",
    "typer>=0.12",
    "structlog>=24.4",
]

[project.optional-dependencies]
pdf = ["weasyprint>=63"]
excel = ["openpyxl>=3.1"]

[project.scripts]
facturar = "gestor_facturas.cli:app"

[project.entry-points."gestor_facturas.exportadores"]
csv = "gestor_facturas.exportar:ExportadorCSV"
json = "gestor_facturas.exportar:ExportadorJSON"

[project.urls]
Repository = "https://github.com/empresa/gestor-facturas"
Documentation = "https://empresa.github.io/gestor-facturas"

# ============================================================ Desarrollo
[dependency-groups]
test = ["pytest>=8.3", "pytest-cov>=6", "hypothesis>=6.120", "respx>=0.21"]
lint = ["ruff>=0.8", "mypy>=1.13", "bandit>=1.8", "pip-audit>=2.7"]
docs = ["mkdocs-material>=9.5", "mkdocstrings[python]>=0.27"]
dev = [{ include-group = "test" }, { include-group = "lint" }, "pre-commit>=4", "nox>=2024.10"]

# ============================================================ Backend
[tool.hatch.version]
source = "vcs"

[tool.hatch.build.targets.wheel]
packages = ["src/gestor_facturas"]

# ============================================================ uv
[tool.uv]
default-groups = ["dev"]

# ============================================================ Ruff
[tool.ruff]
line-length = 100
src = ["src", "tests"]

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "UP", "SIM", "S", "N", "RUF", "PT"]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101", "S105"]

[tool.ruff.format]
quote-style = "double"

# ============================================================ mypy
[tool.mypy]
strict = true
files = ["src", "tests"]
plugins = ["pydantic.mypy"]

[[tool.mypy.overrides]]
module = ["weasyprint.*"]
ignore_missing_imports = true

# ============================================================ pytest y coverage
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = ["-ra", "--strict-markers", "--strict-config", "--import-mode=importlib",
           "--cov", "--cov-report=term-missing"]
markers = ["lento: pruebas que tardan más de 1 s", "red: requieren acceso a red"]
filterwarnings = ["error"]

[tool.coverage.run]
source = ["gestor_facturas"]
branch = true

[tool.coverage.report]
fail_under = 90
show_missing = true
skip_covered = true
exclude_also = ["if TYPE_CHECKING:", "@overload", "raise NotImplementedError"]

# ============================================================ Seguridad
[tool.bandit]
exclude_dirs = ["tests", ".venv"]
```

Script de auditoría (`scripts/auditar_pyproject.py`) que revisa las convenciones del equipo:

```python
"""Comprueba que pyproject.toml cumple las convenciones del equipo."""
from __future__ import annotations

import sys
import tomllib
from pathlib import Path

from packaging.requirements import InvalidRequirement, Requirement
from packaging.specifiers import SpecifierSet

OBLIGATORIOS = ["name", "description", "readme", "requires-python", "license", "authors"]
HERRAMIENTAS = ["ruff", "mypy", "pytest", "coverage"]


def auditar(ruta: Path) -> list[str]:
    datos = tomllib.loads(ruta.read_text(encoding="utf-8"))
    problemas: list[str] = []
    proyecto = datos.get("project", {})

    if "build-system" not in datos:
        problemas.append("Falta la tabla [build-system].")
    for campo in OBLIGATORIOS:
        if campo not in proyecto:
            problemas.append(f"Falta [project].{campo}.")
    if "version" not in proyecto and "version" not in proyecto.get("dynamic", []):
        problemas.append("La versión no está definida ni declarada como dinámica.")
    if isinstance(proyecto.get("license"), dict):
        problemas.append("Use una expresión SPDX en 'license' (PEP 639), no una tabla.")

    rp = proyecto.get("requires-python")
    if rp and "3.11" in SpecifierSet(rp):
        problemas.append(f"requires-python ({rp}) admite Python 3.11; la política exige >=3.12.")

    for dep in proyecto.get("dependencies", []):
        try:
            req = Requirement(dep)
        except InvalidRequirement as e:
            problemas.append(f"Dependencia no válida «{dep}»: {e}")
            continue
        if not req.specifier and req.url is None:
            problemas.append(f"Dependencia sin restricción de versión: {req.name}")

    herramientas = datos.get("tool", {})
    for nombre in HERRAMIENTAS:
        if nombre not in herramientas:
            problemas.append(f"Falta la configuración [tool.{nombre}].")
    return problemas


def main() -> int:
    ruta = Path(sys.argv[1] if len(sys.argv) > 1 else "pyproject.toml")
    problemas = auditar(ruta)
    for p in problemas:
        print(f"✖ {p}")
    print("✔ pyproject.toml correcto" if not problemas else f"\n{len(problemas)} problemas encontrados")
    return 1 if problemas else 0


if __name__ == "__main__":
    raise SystemExit(main())
```

```bash
uv run --with packaging scripts/auditar_pyproject.py
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TOMLDecodeError: Cannot overwrite a value` | Tabla o clave declarada dos veces. | Unifique la tabla en un solo bloque. |
| `TypeError: File must be opened in binary mode` | `tomllib.load()` con archivo en modo texto. | Abra con `"rb"` o use `tomllib.loads(texto)`. |
| `project.license must be valid exactly by one definition` | Formato de licencia incorrecto para el *backend*. | Use `license = "MIT"` con un *backend* actualizado. |
| `Field "version" declared as dynamic but is defined` | Versión en `version` y en `dynamic` a la vez. | Elija uno de los dos. |
| `ModuleNotFoundError` al ejecutar el script instalado | La ruta de `[project.scripts]` es incorrecta. | Formato `paquete.modulo:funcion`, con la ruta real del módulo. |
| El *wheel* no incluye el paquete | El *backend* no encuentra el código en `src/`. | Configure `packages`/`where`/`module-root` del *backend*. |
| La configuración de una herramienta se ignora | Nombre de tabla incorrecto (p. ej. `[tool.pytest]` en pytest < 9). | Consulte el nombre exacto en la documentación de la herramienta. |
| `invalid pyproject.toml config: project.dependencies[0]` | Especificador mal escrito. | Use la sintaxis PEP 508: `"paquete>=1.0"`. |

---

## 18. Buenas prácticas

- Use **un solo `pyproject.toml`** como fuente de verdad: metadatos, dependencias y configuración de herramientas.
- Declare siempre `[build-system]` y `requires-python`.
- Use la **estructura `src/`** para evitar importar el código sin instalarlo.
- Licencia como expresión **SPDX** y `license-files` (PEP 639).
- Separe extras (para usuarios) de grupos de dependencias (para desarrolladores).
- No duplique la versión: use `dynamic` o léala con `importlib.metadata`.
- Mantenga las dependencias directas en `pyproject.toml` y las versiones exactas en el archivo de bloqueo.
- Ordene el archivo por secciones con comentarios separadores.
- Valide el archivo en CI (`validate-pyproject`, `uv build`, `twine check`).
- Añada `py.typed` y el clasificador `Typing :: Typed` si el paquete incluye anotaciones de tipo.

---

## 19. Referencia rápida

```toml
[build-system]       requires = ["hatchling"]; build-backend = "hatchling.build"
[project]            name, version | dynamic, description, readme, requires-python,
                     license = "MIT", license-files, authors, classifiers, dependencies
[project.optional-dependencies]   extra = ["paquete>=1"]
[project.scripts]                 comando = "paquete.modulo:funcion"
[project.entry-points."grupo"]    nombre = "paquete.modulo:objeto"
[project.urls]                    Repository = "https://..."
[dependency-groups]               dev = ["pytest", { include-group = "lint" }]
[tool.<herramienta>]              Configuración de Ruff, mypy, pytest, coverage, uv…
```

```python
import tomllib; datos = tomllib.load(open("pyproject.toml", "rb"))
from importlib.metadata import version; version("paquete")
```

---

## 20. Ejercicios propuestos

1. Convierta un proyecto con `setup.py`, `setup.cfg` y `pytest.ini` a un único `pyproject.toml`.
2. Configure la versión dinámica desde etiquetas de Git y verifique que `uv build` genera el *wheel* con la versión correcta.
3. Defina un sistema de *plugins* con `entry-points` y cree un segundo paquete que registre un *plugin*.
4. Amplíe el script de auditoría para comprobar que todos los extras tienen al menos una dependencia con restricción.
5. Escriba un script con `tomlkit` que incremente la versión de parche conservando los comentarios del archivo.

---

## 21. Referencias

- Guía oficial «Writing your pyproject.toml»: https://packaging.python.org/en/latest/guides/writing-pyproject-toml/
- Especificación de metadatos: https://packaging.python.org/en/latest/specifications/pyproject-toml/
- PEP 517 / 518 / 621: https://peps.python.org/pep-0621/
- PEP 639 (licencias): https://peps.python.org/pep-0639/
- PEP 735 (grupos de dependencias): https://peps.python.org/pep-0735/
- Lista de clasificadores: https://pypi.org/classifiers/
- Especificación TOML: https://toml.io/es/
