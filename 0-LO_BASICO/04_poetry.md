# Manual de uso: Poetry

> **Categoría:** Gestión de proyectos · **Utilidad:** Gestionar dependencias con archivo de bloqueo, el entorno y la publicación de paquetes
> **Tipo:** Herramienta externa (requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear un proyecto](#4-crear-un-proyecto)
5. [El archivo `pyproject.toml` en Poetry](#5-el-archivo-pyprojecttoml-en-poetry)
6. [Gestionar dependencias](#6-gestionar-dependencias)
7. [Grupos de dependencias y extras](#7-grupos-de-dependencias-y-extras)
8. [El archivo de bloqueo](#8-el-archivo-de-bloqueo)
9. [Entornos virtuales](#9-entornos-virtuales)
10. [Ejecutar comandos y scripts](#10-ejecutar-comandos-y-scripts)
11. [Versiones del proyecto](#11-versiones-del-proyecto)
12. [Construir y publicar paquetes](#12-construir-y-publicar-paquetes)
13. [Repositorios privados y credenciales](#13-repositorios-privados-y-credenciales)
14. [Configuración](#14-configuración)
15. [Plugins y exportación](#15-plugins-y-exportación)
16. [Poetry en CI y Docker](#16-poetry-en-ci-y-docker)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**Poetry** es una herramienta de gestión de proyectos Python que unifica en un solo flujo:

- la **declaración de dependencias** en `pyproject.toml`;
- su **resolución y bloqueo** en `poetry.lock` (reproducibilidad exacta);
- la **creación y gestión del entorno virtual**;
- la **construcción y publicación** de paquetes en PyPI o repositorios privados.

**Ventajas:**
- Resolutor de dependencias robusto y archivo de bloqueo multiplataforma.
- Grupos de dependencias (desarrollo, pruebas, documentación).
- Publicación integrada, sin necesidad de `build` ni `twine`.
- Desde la versión 2.0 usa la tabla estándar `[project]` (PEP 621).
- Proyecto maduro, con amplia adopción y documentación.

**Limitaciones:**
- No instala versiones de Python (requiere pyenv, `py` o similar).
- La resolución es más lenta que la de **uv**.
- Parte de su configuración sigue en `[tool.poetry]`, específica de la herramienta.

| Si necesita… | Opción |
|---|---|
| Proyecto con bloqueo, entorno y publicación, herramienta consolidada | **Poetry** |
| Lo mismo, mucho más rápido y gestionando también Python | uv |
| Solo instalar paquetes | pip |
| Solo herramientas globales | pipx |

---

## 2. Instalación

Poetry debe instalarse **aislado** de sus proyectos. Opciones recomendadas:

```bash
pipx install poetry                     # Recomendada
uv tool install poetry                  # Alternativa con uv
```

Instalador oficial:

```bash
curl -sSL https://install.python-poetry.org | python3 -                       # Linux/macOS
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -   # Windows
```

Comprobar y actualizar:

```bash
poetry --version
pipx upgrade poetry          # Si se instaló con pipx
poetry self update           # Si se usó el instalador oficial
```

Autocompletado:

```bash
poetry completions bash >> ~/.bash_completion
poetry completions zsh > ~/.zfunc/_poetry
```

> ⚠ No instale Poetry con `pip install poetry` dentro del entorno de un proyecto: sus dependencias se mezclarían con las del proyecto.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `pyproject.toml` | Declara metadatos y dependencias (rangos aceptables). |
| `poetry.lock` | Versiones exactas y hashes de **todas** las dependencias, incluidas las transitivas. |
| Grupo | Conjunto de dependencias opcionales para una tarea (`dev`, `test`, `docs`). |
| Extra | Dependencias opcionales que los **usuarios** del paquete pueden pedir (`paquete[pdf]`). |
| Entorno | Entorno virtual gestionado por Poetry para el proyecto. |
| Modo *package* | El proyecto es un paquete instalable (por defecto). |
| Modo *non-package* | Solo gestiona dependencias (aplicaciones, scripts): `package-mode = false`. |
| Fuente | Repositorio del que se descargan paquetes (PyPI u otros). |

**Flujo general:** declarar en `pyproject.toml` → `poetry lock` resuelve → `poetry install`/`poetry sync` instala exactamente lo bloqueado.

---

## 4. Crear un proyecto

### 4.1 Proyecto nuevo

```bash
poetry new facturacion                 # Estructura con src/ y tests/
poetry new --flat facturacion          # Sin carpeta src/
```

Estructura generada:

```
facturacion/
├── pyproject.toml
├── README.md
├── src/
│   └── facturacion/
│       └── __init__.py
└── tests/
    └── __init__.py
```

### 4.2 Proyecto existente

```bash
cd proyecto_existente
poetry init                            # Asistente interactivo
poetry init --no-interaction --name mi-app --python ">=3.12"
```

### 4.3 Importar `requirements.txt`

```bash
poetry add $(sed 's/#.*//' requirements.txt | xargs)          # Linux/macOS
```

---

## 5. El archivo `pyproject.toml` en Poetry

Desde Poetry 2.0, los metadatos van en la tabla estándar `[project]` y lo específico de Poetry en `[tool.poetry]`:

```toml
[project]
name = "facturacion"
version = "0.1.0"
description = "Generación y envío de facturas"
authors = [{ name = "Ana García", email = "ana@empresa.com" }]
readme = "README.md"
license = "MIT"
requires-python = ">=3.12"
dependencies = [
    "httpx (>=0.27,<1.0)",
    "pydantic (>=2.9,<3.0)",
    "jinja2 (>=3.1,<4.0)",
]

[project.optional-dependencies]
pdf = ["weasyprint (>=63,<64)"]

[project.scripts]
facturar = "facturacion.cli:main"

[project.urls]
repository = "https://github.com/empresa/facturacion"

[tool.poetry]
packages = [{ include = "facturacion", from = "src" }]

[tool.poetry.group.dev.dependencies]
pytest = "^8.3"
ruff = "^0.8"
mypy = "^1.13"

[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

### 5.1 Restricciones de versión de Poetry

| Sintaxis | Significado | Equivale a |
|---|---|---|
| `^1.2.3` | Compatible (no cambia el primer dígito distinto de cero) | `>=1.2.3,<2.0.0` |
| `^0.2.3` | | `>=0.2.3,<0.3.0` |
| `~1.2.3` | Cambios de parche | `>=1.2.3,<1.3.0` |
| `1.2.*` | Comodín | `>=1.2,<1.3` |
| `>=1.2,<2` | Rango explícito (PEP 440) | — |

> En `[project].dependencies` se usa la sintaxis estándar PEP 508 (`"httpx (>=0.27,<1.0)"`). Los operadores `^` y `~` solo son válidos en las tablas `[tool.poetry...]`.

### 5.2 Proyecto que no es un paquete

Para una aplicación o conjunto de scripts que no se va a publicar:

```toml
[tool.poetry]
package-mode = false
```

---

## 6. Gestionar dependencias

```bash
poetry add httpx                          # Última versión compatible, con restricción ^
poetry add "httpx@^0.27"                  # Restricción explícita
poetry add "httpx>=0.27,<1"
poetry add "fastapi[standard]"            # Con extras
poetry add git+https://github.com/org/repo.git#v1.2.0
poetry add ./libs/utilidades --editable   # Ruta local editable
poetry add --group dev pytest ruff        # A un grupo
poetry add --optional pdf weasyprint      # Como extra del paquete

poetry remove httpx
poetry remove --group dev ruff

poetry update                             # Actualiza todo dentro de las restricciones
poetry update httpx pydantic              # Solo algunos
poetry show                               # Lista de dependencias instaladas
poetry show --tree                        # Árbol de dependencias
poetry show --outdated                    # Desactualizadas
poetry show --top-level                   # Solo las directas
poetry show httpx                         # Detalle de un paquete
poetry search fastapi                     # (Solo en índices que lo admitan)
```

Dependencias condicionales:

```toml
[project]
dependencies = [
    "tomli (>=2.0) ; python_version < '3.11'",
    "uvloop (>=0.21) ; sys_platform != 'win32'",
]
```

---

## 7. Grupos de dependencias y extras

### 7.1 Grupos (para el desarrollo del proyecto)

```toml
[tool.poetry.group.dev.dependencies]
ruff = "^0.8"
mypy = "^1.13"

[tool.poetry.group.test.dependencies]
pytest = "^8.3"
pytest-cov = "^6.0"

[tool.poetry.group.docs]
optional = true                  # No se instala salvo que se pida

[tool.poetry.group.docs.dependencies]
mkdocs-material = "^9.5"
```

```bash
poetry install                              # Principal + grupos no opcionales
poetry install --with docs                  # Añadir un grupo opcional
poetry install --without test,docs          # Excluir grupos
poetry install --only main                  # Solo producción
poetry install --only test                  # Solo un grupo
```

### 7.2 Extras (para los usuarios del paquete)

```toml
[project.optional-dependencies]
pdf = ["weasyprint (>=63,<64)"]
excel = ["openpyxl (>=3.1,<4)"]
```

```bash
poetry install --extras pdf
poetry install --all-extras
pip install "facturacion[pdf]"              # Lo que hará el usuario final
```

| | Grupos | Extras |
|---|---|---|
| ¿Para quién? | Desarrolladores del proyecto | Usuarios del paquete |
| ¿Se publican? | No | Sí |
| Ejemplo | pytest, ruff | soporte PDF, Excel |

---

## 8. El archivo de bloqueo

`poetry.lock` registra la versión exacta y los hashes de cada paquete. Garantiza que todos los equipos y entornos instalan exactamente lo mismo.

```bash
poetry lock                    # Resuelve y actualiza el bloqueo (respeta lo ya fijado)
poetry lock --regenerate       # Resuelve desde cero
poetry check --lock            # Verifica que el bloqueo está sincronizado con pyproject.toml
poetry install                 # Instala lo bloqueado (añade lo que falte)
poetry sync                    # Instala lo bloqueado y ELIMINA lo que sobre
```

> **Versione siempre `poetry.lock`** en aplicaciones. En librerías también es recomendable para que el equipo y la CI usen las mismas versiones (los usuarios de la librería no lo usan).

### 8.1 Protección frente a ataques a la cadena de suministro

Desde Poetry 2.4 puede exigir que una versión lleve un número mínimo de días publicada antes de considerarla:

```bash
poetry config solver.min-release-age 7
poetry config solver.min-release-age-exclude "mi-paquete-interno"
```

---

## 9. Entornos virtuales

Poetry crea y usa automáticamente un entorno para el proyecto.

```bash
poetry env info                  # Información del entorno actual
poetry env info --path           # Solo la ruta
poetry env list                  # Entornos del proyecto
poetry env use python3.13        # Usar (o crear) un entorno con esa versión
poetry env use 3.12
poetry env remove python3.12     # Eliminar
poetry env remove --all
```

### 9.1 Entorno dentro del proyecto (`.venv`)

Recomendado para que los IDE lo detecten automáticamente:

```bash
poetry config virtualenvs.in-project true
```

### 9.2 Activar el entorno

El comando `poetry shell` se retiró en Poetry 2.0 (está disponible como *plugin*). Use:

```bash
eval $(poetry env activate)                   # bash/zsh
Invoke-Expression (poetry env activate)       # PowerShell
```

O simplemente `poetry run …` (sección 10), o active `.venv` como cualquier entorno virtual.

---

## 10. Ejecutar comandos y scripts

```bash
poetry run python script.py
poetry run pytest -q
poetry run ruff check .
poetry run facturar --mes 2026-11          # Script declarado en [project.scripts]
```

`poetry run` ejecuta el comando dentro del entorno del proyecto sin necesidad de activarlo.

### 10.1 Tareas del proyecto

Poetry no incluye un ejecutor de tareas. Opciones habituales:

- `Makefile` o `justfile` que invoquen `poetry run`.
- El *plugin* `poethepoet`:

```toml
[tool.poe.tasks]
test = "pytest -q"
lint = ["_ruff", "_mypy"]
_ruff = "ruff check ."
_mypy = "mypy src"
```

```bash
poetry run poe test
```

---

## 11. Versiones del proyecto

```bash
poetry version                  # Muestra: facturacion 0.1.0
poetry version --short          # 0.1.0
poetry version patch            # 0.1.0 → 0.1.1
poetry version minor            # 0.1.1 → 0.2.0
poetry version major            # 0.2.0 → 1.0.0
poetry version prerelease       # 1.0.0 → 1.0.1a0
poetry version 2.0.0            # Valor explícito
```

Leer la versión desde el código sin duplicarla:

```python
from importlib.metadata import version

__version__ = version("facturacion")
```

---

## 12. Construir y publicar paquetes

```bash
poetry build                          # Genera dist/*.whl y dist/*.tar.gz
poetry build --format wheel
poetry check                          # Valida pyproject.toml

poetry publish                        # Sube dist/ a PyPI
poetry publish --build                # Construye y publica
poetry publish -r testpypi            # A otro repositorio
poetry publish --dry-run
```

Configurar el token de PyPI:

```bash
poetry config pypi-token.pypi pypi-AgEIcHlwaS5vcmc...
```

Publicar primero en TestPyPI:

```bash
poetry config repositories.testpypi https://test.pypi.org/legacy/
poetry config pypi-token.testpypi pypi-...
poetry publish --build -r testpypi
```

> En CI, prefiera la **publicación de confianza** (*Trusted Publishing*) de PyPI mediante OIDC, que evita guardar tokens.

---

## 13. Repositorios privados y credenciales

### 13.1 Añadir una fuente

```bash
poetry source add --priority=supplemental interno https://pypi.empresa.local/simple/
poetry source add --priority=primary interno https://pypi.empresa.local/simple/
poetry source show
```

```toml
[[tool.poetry.source]]
name = "interno"
url = "https://pypi.empresa.local/simple/"
priority = "supplemental"
```

| Prioridad | Comportamiento |
|---|---|
| `primary` | Se consulta antes que PyPI (si hay alguna `primary`, PyPI deja de ser la principal). |
| `supplemental` | Solo se consulta si el paquete no está en las primarias. |
| `explicit` | Solo para los paquetes que la indiquen expresamente. |

Fijar un paquete a una fuente (evita la confusión de dependencias):

```toml
[tool.poetry.dependencies]
utilidades-internas = { version = "^2.1", source = "interno" }
```

### 13.2 Credenciales

```bash
poetry config http-basic.interno usuario contraseña
```

O con variables de entorno (ideal en CI):

```bash
export POETRY_HTTP_BASIC_INTERNO_USERNAME=usuario
export POETRY_HTTP_BASIC_INTERNO_PASSWORD=token
```

Poetry guarda las credenciales en el llavero del sistema cuando está disponible.

---

## 14. Configuración

```bash
poetry config --list                                   # Ver toda la configuración
poetry config virtualenvs.in-project true              # .venv en el proyecto
poetry config virtualenvs.prefer-active-python true    # Usar el Python activo
poetry config installer.max-workers 8
poetry config --local virtualenvs.in-project true      # Solo para este proyecto (poetry.toml)
poetry config --unset virtualenvs.in-project
```

Cualquier opción puede definirse por variable de entorno con el prefijo `POETRY_`:

```bash
export POETRY_VIRTUALENVS_IN_PROJECT=true
export POETRY_NO_INTERACTION=1
```

Exigir una versión de Poetry para el proyecto:

```toml
[tool.poetry]
requires-poetry = ">=2.0"
```

---

## 15. Plugins y exportación

```bash
poetry self add poetry-plugin-export          # Exportar a requirements.txt
poetry self add poetry-plugin-shell           # Recuperar «poetry shell»
poetry self show plugins
```

Exportar el bloqueo a `requirements.txt` (útil para Docker o herramientas que solo leen ese formato):

```bash
poetry export -f requirements.txt --output requirements.txt
poetry export --only main --without-hashes -o requirements.txt
poetry export --with dev -o requirements-dev.txt
```

Declarar los *plugins* que necesita el proyecto:

```toml
[tool.poetry.requires-plugins]
poetry-plugin-export = ">=1.8"
```

---

## 16. Poetry en CI y Docker

### 16.1 GitHub Actions

```yaml
jobs:
  pruebas:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.13"
      - run: pipx install poetry
      - run: poetry config virtualenvs.in-project true
      - uses: actions/cache@v4
        with:
          path: .venv
          key: venv-${{ runner.os }}-${{ hashFiles('poetry.lock') }}
      - run: poetry check --lock
      - run: poetry sync --with test
      - run: poetry run pytest
```

### 16.2 Docker (multietapa)

```dockerfile
FROM python:3.13-slim AS builder
ENV POETRY_NO_INTERACTION=1 \
    POETRY_VIRTUALENVS_IN_PROJECT=true \
    POETRY_VIRTUALENVS_CREATE=true
RUN pip install --no-cache-dir poetry
WORKDIR /app
COPY pyproject.toml poetry.lock ./
RUN poetry sync --only main --no-root
COPY src ./src
RUN poetry sync --only main

FROM python:3.13-slim
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv
COPY --from=builder /app/src /app/src
ENV PATH="/app/.venv/bin:$PATH"
CMD ["facturar"]
```

> Copiar primero `pyproject.toml` y `poetry.lock` permite que Docker reutilice la capa de dependencias mientras estas no cambien.

---

## 17. Ejemplo práctico completo

**Librería publicable** `conversor-divisas` con CLI, extras opcionales, grupos de desarrollo y un flujo completo: crear, desarrollar, probar, versionar y publicar.

### 17.1 Crear el proyecto

```bash
poetry new conversor-divisas
cd conversor-divisas
poetry config --local virtualenvs.in-project true
poetry env use 3.13
poetry add "httpx>=0.27,<1" "typer>=0.12,<1"
poetry add --optional cache "diskcache>=5.6,<6"
poetry add --group dev pytest pytest-cov ruff mypy respx
```

### 17.2 `pyproject.toml` resultante (ajustado)

```toml
[project]
name = "conversor-divisas"
version = "0.1.0"
description = "Conversión de divisas con tipos de cambio del BCE"
authors = [{ name = "Equipo Finanzas", email = "finanzas@empresa.com" }]
readme = "README.md"
license = "MIT"
requires-python = ">=3.12"
dependencies = [
    "httpx (>=0.27,<1)",
    "typer (>=0.12,<1)",
]

[project.optional-dependencies]
cache = ["diskcache (>=5.6,<6)"]

[project.scripts]
divisas = "conversor_divisas.cli:app"

[tool.poetry]
packages = [{ include = "conversor_divisas", from = "src" }]

[tool.poetry.group.dev.dependencies]
pytest = "^8.3"
pytest-cov = "^6.0"
ruff = "^0.8"
mypy = "^1.13"
respx = "^0.21"

[tool.pytest.ini_options]
addopts = "-q --cov=conversor_divisas --cov-report=term-missing"
testpaths = ["tests"]

[tool.ruff]
line-length = 100

[tool.mypy]
strict = true

[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

### 17.3 Código

`src/conversor_divisas/servicio.py`:

```python
from __future__ import annotations

import xml.etree.ElementTree as ET
from decimal import ROUND_HALF_EVEN, Decimal

import httpx

URL_BCE = "https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml"
NS = {"e": "http://www.ecb.int/vocabulary/2002-08-01/eurofxref"}


class DivisaDesconocida(ValueError):
    pass


def obtener_tipos(cliente: httpx.Client | None = None) -> dict[str, Decimal]:
    """Descarga los tipos de cambio del día respecto al euro."""
    cliente = cliente or httpx.Client(timeout=10)
    respuesta = cliente.get(URL_BCE)
    respuesta.raise_for_status()
    raiz = ET.fromstring(respuesta.content)
    tipos = {"EUR": Decimal("1")}
    for nodo in raiz.iterfind(".//e:Cube[@currency]", NS):
        tipos[nodo.attrib["currency"]] = Decimal(nodo.attrib["rate"])
    return tipos


def convertir(importe: Decimal, origen: str, destino: str, tipos: dict[str, Decimal]) -> Decimal:
    origen, destino = origen.upper(), destino.upper()
    for divisa in (origen, destino):
        if divisa not in tipos:
            raise DivisaDesconocida(divisa)
    resultado = importe / tipos[origen] * tipos[destino]
    return resultado.quantize(Decimal("0.01"), rounding=ROUND_HALF_EVEN)
```

`src/conversor_divisas/cli.py`:

```python
from decimal import Decimal
from importlib.metadata import version

import typer

from .servicio import DivisaDesconocida, convertir, obtener_tipos

app = typer.Typer(help="Conversor de divisas (tipos del BCE).")


@app.command()
def main(
    importe: str = typer.Argument(..., help="Importe, p. ej. 100.50"),
    origen: str = typer.Argument(..., help="Divisa de origen, p. ej. EUR"),
    destino: str = typer.Argument(..., help="Divisa de destino, p. ej. USD"),
) -> None:
    """Convierte IMPORTE de ORIGEN a DESTINO."""
    typer.echo(f"conversor-divisas {version('conversor-divisas')}", err=True)
    try:
        resultado = convertir(Decimal(importe.replace(",", ".")), origen, destino, obtener_tipos())
    except DivisaDesconocida as e:
        typer.secho(f"Divisa desconocida: {e}", fg="red", err=True)
        raise typer.Exit(1) from e
    typer.echo(f"{importe} {origen.upper()} = {resultado} {destino.upper()}")
```

`tests/test_servicio.py`:

```python
from decimal import Decimal

import httpx
import pytest
import respx

from conversor_divisas.servicio import URL_BCE, DivisaDesconocida, convertir, obtener_tipos

XML = b"""<?xml version="1.0"?>
<gesmes:Envelope xmlns:gesmes="http://www.gesmes.org/xml/2002-08-01"
                 xmlns="http://www.ecb.int/vocabulary/2002-08-01/eurofxref">
  <Cube><Cube time="2026-11-26">
    <Cube currency="USD" rate="1.0500"/><Cube currency="GBP" rate="0.8400"/>
  </Cube></Cube>
</gesmes:Envelope>"""


@respx.mock
def test_obtener_tipos() -> None:
    respx.get(URL_BCE).mock(return_value=httpx.Response(200, content=XML))
    tipos = obtener_tipos()
    assert tipos["USD"] == Decimal("1.0500")
    assert tipos["EUR"] == 1


def test_convertir() -> None:
    tipos = {"EUR": Decimal(1), "USD": Decimal("1.05"), "GBP": Decimal("0.84")}
    assert convertir(Decimal("100"), "eur", "usd", tipos) == Decimal("105.00")
    assert convertir(Decimal("105"), "USD", "GBP", tipos) == Decimal("84.00")


def test_divisa_desconocida() -> None:
    with pytest.raises(DivisaDesconocida):
        convertir(Decimal(1), "EUR", "XXX", {"EUR": Decimal(1)})
```

### 17.4 Desarrollo, publicación y uso

```bash
poetry install --all-extras         # Instala todo, incluido el propio paquete en editable
poetry run pytest
poetry run ruff check . && poetry run mypy src
poetry run divisas 100 EUR USD

poetry version minor                # 0.1.0 → 0.2.0
poetry check --lock
poetry build
poetry publish -r testpypi          # Verificar en TestPyPI
poetry publish                      # Publicar en PyPI

pipx install "conversor-divisas[cache]"     # Lo que hará un usuario
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `pyproject.toml changed significantly since poetry.lock was last generated` | Se editó `pyproject.toml` sin regenerar el bloqueo. | `poetry lock`. |
| `SolverProblemError` / `version solving failed` | Restricciones incompatibles (a menudo `requires-python` demasiado amplio). | Ajuste `requires-python` al rango que admiten sus dependencias o relaje restricciones. |
| `The command "shell" does not exist` | `poetry shell` se retiró en 2.0. | `eval $(poetry env activate)` o `poetry self add poetry-plugin-shell`. |
| `The command "export" does not exist` | Falta el *plugin*. | `poetry self add poetry-plugin-export`. |
| `No file/folder found for package` | La estructura no coincide con `packages`. | Revise `packages = [{ include = ..., from = "src" }]` o use `package-mode = false`. |
| El IDE no encuentra los paquetes | El entorno está en la caché de Poetry. | `poetry config virtualenvs.in-project true` y recrear, o seleccionar `poetry env info --path`. |
| `Current Python version is not allowed by the project` | El Python activo no cumple `requires-python`. | `poetry env use 3.13`. |
| Poetry se rompe tras actualizar dependencias del proyecto | Poetry se instaló dentro del entorno del proyecto. | Desinstálelo de ahí e instálelo con pipx. |
| Instalación lenta en CI | Sin caché. | Cachee `.venv` con clave basada en `poetry.lock`. |

---

## 19. Buenas prácticas

- Instale Poetry con **pipx** o `uv tool`, nunca dentro del proyecto.
- Use la tabla estándar **`[project]`** para los metadatos.
- **Versione `poetry.lock`** y verifique en CI con `poetry check --lock`.
- Use `poetry sync` en CI y Docker para que el entorno coincida exactamente con el bloqueo.
- Configure `virtualenvs.in-project = true` para que el entorno sea visible para el IDE.
- Separe dependencias en grupos (`dev`, `test`, `docs`) y marque como opcionales los que no se usan a diario.
- Fije las dependencias internas a su fuente privada para evitar la confusión de dependencias.
- Active `solver.min-release-age` en proyectos sensibles a la cadena de suministro.
- Ajuste `requires-python` a la realidad del proyecto: un rango demasiado amplio provoca conflictos de resolución.
- Lea la versión con `importlib.metadata.version()` en lugar de duplicarla en el código.

---

## 20. Referencia rápida

```bash
poetry new nombre / init
poetry add paquete [--group dev] [--optional extra]  ·  remove paquete
poetry lock [--regenerate]  ·  check --lock
poetry install [--with g] [--without g] [--only main] [--all-extras]  ·  sync
poetry update [paquete]
poetry show [--tree | --outdated | --top-level]
poetry run comando
poetry env use 3.13 / info / list / remove
eval $(poetry env activate)
poetry version patch|minor|major
poetry build  ·  publish [--build] [-r repo]
poetry source add --priority=supplemental nombre URL
poetry config clave valor [--local]
poetry self add plugin  ·  export -o requirements.txt
```

---

## 21. Ejercicios propuestos

1. Cree un proyecto con `poetry new`, añada `requests` y `pytest` (grupo `dev`) e inspeccione `poetry.lock`.
2. Convierta un proyecto existente con `requirements.txt` a Poetry y compruebe con `poetry show --tree` el árbol de dependencias.
3. Defina un grupo opcional `docs` con MkDocs y verifique que `poetry install` no lo instala salvo con `--with docs`.
4. Publique el ejemplo del conversor en TestPyPI e instálelo con pipx desde allí.
5. Escriba un `Dockerfile` multietapa que use `poetry sync --only main` y compare el tamaño de la imagen con una sin multietapa.

---

## 22. Referencias

- Documentación oficial: https://python-poetry.org/docs/
- Gestión de dependencias: https://python-poetry.org/docs/managing-dependencies/
- Configuración: https://python-poetry.org/docs/configuration/
- Repositorios: https://python-poetry.org/docs/repositories/
- Blog de versiones: https://python-poetry.org/blog/
- *Plugin* de exportación: https://github.com/python-poetry/poetry-plugin-export
