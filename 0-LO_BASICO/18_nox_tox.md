# Manual de uso: nox y tox

> **Categoría:** Pruebas · **Utilidad:** Ejecutan las pruebas en varias versiones de Python
> **Tipo:** Paquetes externos (requieren instalación) · nox: configuración en Python · tox: configuración declarativa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [nox: primer `noxfile.py`](#4-nox-primer-noxfilepy)
5. [nox: sesiones, versiones y parametrización](#5-nox-sesiones-versiones-y-parametrización)
6. [nox: la API de `session`](#6-nox-la-api-de-session)
7. [nox: ejecución desde la terminal](#7-nox-ejecución-desde-la-terminal)
8. [nox: integración con uv](#8-nox-integración-con-uv)
9. [tox: configuración básica](#9-tox-configuración-básica)
10. [tox: entornos, factores y etiquetas](#10-tox-entornos-factores-y-etiquetas)
11. [tox: ejecución desde la terminal](#11-tox-ejecución-desde-la-terminal)
12. [tox: integración con uv (`tox-uv`)](#12-tox-integración-con-uv-tox-uv)
13. [Obtener las versiones de Python](#13-obtener-las-versiones-de-python)
14. [nox y tox en CI](#14-nox-y-tox-en-ci)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**nox** y **tox** automatizan la ejecución de tareas del proyecto (pruebas, *linters*, comprobación de tipos, documentación, construcción) en **entornos virtuales aislados y reproducibles**, y permiten repetir cada tarea en **varias versiones de Python** con un solo comando.

```bash
nox                     # Ejecuta todas las sesiones configuradas
tox                     # Ejecuta todos los entornos configurados
```

| | **nox** | **tox** |
|---|---|---|
| Configuración | `noxfile.py` (código Python) | `tox.toml`, `tox.ini` o `[tool.tox]` en `pyproject.toml` |
| Estilo | Imperativo: funciones con lógica, condiciones, bucles | Declarativo: listas de dependencias y comandos |
| Flexibilidad | Muy alta | Alta, con convenciones |
| Antigüedad | Desde 2016 | Desde 2010; muy extendido en librerías |
| Integración con uv | Nativa (`venv_backend="uv"`) | *Plugin* `tox-uv` |
| Paralelismo | Mediante CI o herramientas externas | Integrado (`tox p`) |

**Ventajas comunes:**
- El mismo comando en local y en CI: «funciona en mi equipo» deja de ser un problema.
- Cada tarea en un entorno limpio con sus dependencias exactas.
- Matriz de versiones de Python (y de dependencias) sin configuración repetida.
- Prueban el paquete **instalado**, detectando errores de empaquetado.

**Limitaciones:**
- Crear entornos lleva tiempo (mucho menos con uv).
- Necesitan que las versiones de Python estén instaladas (o un gestor que las descargue).

| Si necesita… | Opción |
|---|---|
| Lógica compleja en las tareas, configuración en Python | **nox** |
| Configuración declarativa, estándar en muchas librerías | **tox** |
| Solo atajos de comandos, sin matriz de versiones | Makefile, `just`, `poethepoet` |

---

## 2. Instalación

Instálelos como **herramientas globales** (gestionan sus propios entornos):

```bash
uv tool install nox
uv tool install tox --with tox-uv
pipx install nox
pipx install tox && pipx inject tox tox-uv
```

Comprobar:

```bash
nox --version
tox --version
```

Ejecución puntual sin instalar:

```bash
uvx nox
uvx --with tox-uv tox
```

---

## 3. Conceptos clave

| Concepto | nox | tox |
|---|---|---|
| Unidad de trabajo | **Sesión** (función decorada) | **Entorno** (sección de configuración) |
| Archivo | `noxfile.py` | `tox.toml` / `tox.ini` / `pyproject.toml` |
| Versión de Python | `python=["3.12", "3.13"]` | Factores `py312`, `py313` |
| Dependencias | `session.install(...)` | `deps`, `extras`, `dependency_groups` |
| Comandos | `session.run(...)` | `commands` |
| Argumentos extra | `session.posargs` | `{posargs}` |
| Reutilizar entornos | `-r` / `reuse_venv` | Por defecto (se recrean si cambian las dependencias) |
| Agrupar tareas | `tags` | `labels` |

---

## 4. nox: primer `noxfile.py`

```python
# noxfile.py
import nox


@nox.session(python=["3.12", "3.13", "3.14"])
def tests(session: nox.Session) -> None:
    """Ejecuta las pruebas en cada versión de Python."""
    session.install(".[test]")
    session.run("pytest", *session.posargs)


@nox.session
def lint(session: nox.Session) -> None:
    """Analiza el código con Ruff."""
    session.install("ruff")
    session.run("ruff", "check", ".")
    session.run("ruff", "format", "--check", ".")
```

```text
$ nox
nox > Running session tests-3.12
nox > Creating virtual environment (virtualenv) using python3.12 in .nox/tests-3-12
nox > python -m pip install '.[test]'
nox > pytest
...
nox > Session tests-3.12 was successful in 9 seconds.
nox > Running session tests-3.13
...
nox > Ran multiple sessions:
nox > * tests-3.12: success
nox > * tests-3.13: success
nox > * tests-3.14: success
nox > * lint: success
```

Los entornos se crean en `.nox/` (añádalo a `.gitignore`).

---

## 5. nox: sesiones, versiones y parametrización

### 5.1 Opciones del decorador

```python
@nox.session(
    python=["3.12", "3.13"],        # Versiones (None: la del propio nox)
    venv_backend="uv",              # "virtualenv" (defecto), "venv", "uv", "conda", "none"
    reuse_venv=True,                # Reutilizar el entorno entre ejecuciones
    tags=["calidad"],               # Etiquetas para seleccionar sesiones
    default=True,                   # Si False, solo se ejecuta cuando se pide expresamente
    name="pruebas",                 # Nombre distinto al de la función
)
def tests(session): ...
```

### 5.2 Parametrización (matriz de dependencias)

```python
@nox.session(python=["3.12", "3.13"])
@nox.parametrize("django", ["4.2", "5.2"])
def tests_django(session: nox.Session, django: str) -> None:
    session.install(f"django~={django}.0", ".[test]")
    session.run("pytest", "tests/django")
```

```bash
nox -l
# * tests_django-3.12(django='4.2')
# * tests_django-3.12(django='5.2')
# * tests_django-3.13(django='4.2')
# * tests_django-3.13(django='5.2')
```

Parámetros con identificadores y combinaciones:

```python
@nox.session
@nox.parametrize(
    ("bd", "url"),
    [
        nox.param("sqlite", "sqlite:///:memory:", id="sqlite"),
        nox.param("postgres", "postgresql://test@localhost/test", id="postgres"),
    ],
)
def integracion(session, bd, url):
    session.install(".[test]", *(["psycopg[binary]"] if bd == "postgres" else []))
    session.run("pytest", "tests/integracion", env={"DATABASE_URL": url})
```

### 5.3 Opciones globales

```python
nox.options.sessions = ["lint", "tests"]          # Sesiones por defecto
nox.options.default_venv_backend = "uv"
nox.options.reuse_existing_virtualenvs = True
nox.options.error_on_missing_interpreters = True  # Falla si falta una versión de Python
nox.needs_version = ">=2025.5.1"
```

### 5.4 Leer las versiones de `pyproject.toml`

```python
from nox.project import load_toml, python_versions

PYPROJECT = load_toml("pyproject.toml")
PYTHONS = python_versions(PYPROJECT)              # A partir de los clasificadores


@nox.session(python=PYTHONS)
def tests(session): ...
```

---

## 6. nox: la API de `session`

| Método / atributo | Uso |
|---|---|
| `session.install(*paquetes)` | Instala con pip (o uv) en el entorno de la sesión. |
| `session.install("-e", ".[test]")` | Instalación editable con extras. |
| `session.install("-r", "requirements.txt")` | Desde un archivo. |
| `session.run(*cmd, env={}, silent=False, success_codes=[0])` | Ejecuta un comando del entorno. |
| `session.run_install(...)` | Como `run`, pero se omite con `--no-install`. |
| `session.run_always(...)` | Se ejecuta incluso con `--install-only`. |
| `session.posargs` | Argumentos tras `--` en la línea de comandos. |
| `session.python` | Versión de Python de la sesión. |
| `session.env` | Variables de entorno de la sesión (diccionario). |
| `session.chdir(ruta)` | Cambia de directorio. |
| `session.notify("otra")` | Programa otra sesión para después. |
| `session.log(...)` / `session.warn(...)` | Mensajes. |
| `session.skip("motivo")` | Omite la sesión. |
| `session.error("motivo")` | Falla la sesión. |
| `session.create_tmp()` | Directorio temporal de la sesión. |
| `session.interactive` | `True` si se ejecuta en una terminal interactiva. |
| `session.venv_backend` | *Backend* usado. |

```python
@nox.session
def docs(session: nox.Session) -> None:
    session.install(".[docs]")
    if "--servir" in session.posargs:
        session.run("mkdocs", "serve")
    else:
        session.run("mkdocs", "build", "--strict")


@nox.session(python=False)                          # Sin entorno: usa las herramientas del sistema
def limpiar(session: nox.Session) -> None:
    session.run("rm", "-rf", "dist", "build", ".coverage", "htmlcov", external=True)
```

> `external=True` es necesario para ejecutar programas que no están en el entorno virtual de la sesión.

---

## 7. nox: ejecución desde la terminal

```bash
nox                              # Sesiones por defecto
nox -l                           # Lista de sesiones (* = se ejecuta por defecto)
nox -s tests                     # Una sesión (todas sus versiones)
nox -s tests-3.13                # Una versión concreta
nox -s lint tests                # Varias
nox -t calidad                   # Por etiqueta
nox -k "tests and not django"    # Por expresión
nox -p 3.13                      # Solo sesiones con esa versión
nox -s tests -- -x -k carrito    # Argumentos para la sesión (posargs)
nox -r                           # Reutilizar entornos (más rápido)
nox -R                           # Reutilizar y no reinstalar (--no-install)
nox --install-only              # Solo preparar entornos (útil para cachear en CI)
nox --forcecolor / --nocolor
nox --report informe.json        # Resultados en JSON
nox -f otro_noxfile.py
```

---

## 8. nox: integración con uv

```python
nox.options.default_venv_backend = "uv"           # Entornos e instalación con uv: mucho más rápido


@nox.session(python=["3.12", "3.13", "3.14"], venv_backend="uv")
def tests(session: nox.Session) -> None:
    # Instala exactamente lo bloqueado en uv.lock (grupo «test») en el entorno de la sesión
    session.run_install(
        "uv", "sync", "--locked", "--group", "test",
        env={"UV_PROJECT_ENVIRONMENT": session.virtualenv.location},
    )
    session.run("pytest", *session.posargs)
```

> Con `venv_backend="uv"`, uv puede **descargar** automáticamente las versiones de Python que falten.

El `noxfile.py` puede declarar sus propias dependencias (PEP 723) y ejecutarse con `uv run noxfile.py`:

```python
#!/usr/bin/env -S uv run --script
# /// script
# dependencies = ["nox>=2025.5.1"]
# ///
import nox
...
if __name__ == "__main__":
    nox.main()
```

---

## 9. tox: configuración básica

tox 4 admite tres formatos. El recomendado para proyectos nuevos es **`tox.toml`** (o `[tool.tox]` en `pyproject.toml`).

### 9.1 `tox.toml`

```toml
requires = ["tox>=4.24", "tox-uv>=1.20"]
env_list = ["3.12", "3.13", "3.14", "lint", "type"]
skip_missing_interpreters = true

[env_run_base]                       # Configuración común a todos los entornos
description = "Pruebas con {base_python}"
extras = ["test"]
commands = [["pytest", { replace = "posargs", default = ["tests"], extend = true }]]

[env.lint]
description = "Análisis con Ruff"
skip_install = true
deps = ["ruff"]
commands = [
    ["ruff", "check", "."],
    ["ruff", "format", "--check", "."],
]

[env.type]
description = "Comprobación de tipos"
deps = ["mypy"]
commands = [["mypy", "src"]]
```

### 9.2 `tox.ini` (formato clásico, aún muy extendido)

```ini
[tox]
requires = tox>=4.24
env_list = py312, py313, py314, lint, type
skip_missing_interpreters = true

[testenv]
description = Pruebas con {base_python}
extras = test
commands = pytest {posargs:tests}

[testenv:lint]
skip_install = true
deps = ruff
commands =
    ruff check .
    ruff format --check .

[testenv:type]
deps = mypy
commands = mypy src
```

### 9.3 Opciones más usadas

| Opción | Uso |
|---|---|
| `env_list` | Entornos que se ejecutan por defecto. |
| `base_python` | Versión de Python del entorno (se deduce del nombre `py313`/`3.13`). |
| `deps` | Dependencias adicionales. |
| `extras` | Extras del proyecto a instalar (`.[test]`). |
| `dependency_groups` | Grupos PEP 735 (`[dependency-groups]`). |
| `skip_install` | No instala el proyecto (herramientas como Ruff). |
| `package` | `"wheel"`, `"sdist"`, `"editable"`, `"skip"`. |
| `commands` | Comandos a ejecutar. |
| `pass_env` / `set_env` | Variables de entorno que se heredan / se fijan. |
| `allowlist_externals` | Programas externos permitidos (`make`, `docker`…). |
| `depends` | Orden entre entornos (p. ej. combinar cobertura al final). |
| `labels` | Grupos de entornos. |
| `parallel_show_output` | Mostrar la salida en ejecución paralela. |

> Por defecto tox construye el paquete y lo instala (como haría un usuario). Esto detecta errores de empaquetado que no aparecen al ejecutar desde el código fuente.

---

## 10. tox: entornos, factores y etiquetas

### 10.1 Factores (formato ini)

Los nombres separados por guiones combinan **factores**:

```ini
[tox]
env_list = py{312,313}-django{42,52}, lint

[testenv]
deps =
    django42: django~=4.2.0
    django52: django~=5.2.0
commands = pytest {posargs}
```

Genera `py312-django42`, `py312-django52`, `py313-django42`, `py313-django52`.

### 10.2 Matrices en `tox.toml`

```toml
env_list = [
    { product = [["3.12", "3.13"], ["django42", "django52"]] },
    "lint",
]

[env_run_base]
deps = [
    { replace = "if", condition = "factor.django42", then = ["django~=4.2.0"] },
    { replace = "if", condition = "factor.django52", then = ["django~=5.2.0"] },
]
```

### 10.3 Etiquetas

```toml
[labels]
calidad = ["lint", "type", "seguridad"]
pruebas = ["3.12", "3.13", "3.14"]
```

```bash
tox -m calidad
```

### 10.4 Dependencias entre entornos

```toml
[env.cobertura]
depends = ["3.12", "3.13", "3.14"]
skip_install = true
deps = ["coverage[toml]"]
commands = [["coverage", "combine"], ["coverage", "report", "--fail-under=90"]]
```

---

## 11. tox: ejecución desde la terminal

```bash
tox                              # Entornos de env_list
tox list                         # Entornos disponibles con su descripción (tox l)
tox -e lint                      # Uno
tox -e 3.13,lint                 # Varios
tox -m calidad                   # Por etiqueta
tox -f django52                  # Por factor
tox p                            # En paralelo (tox run-parallel)
tox p -p 4                       # Con 4 procesos
tox -e 3.13 -- -x -k carrito     # Argumentos para {posargs}
tox -r                           # Recrear los entornos
tox --skip-pkg-install           # No reconstruir/instalar el paquete
tox config -e 3.13               # Ver la configuración efectiva de un entorno
tox devenv -e 3.13 .venv         # Crear un entorno de desarrollo a partir de la configuración
tox exec -e 3.13 -- python -c "import sys; print(sys.version)"
```

Los entornos se guardan en `.tox/` (añádalo a `.gitignore`).

---

## 12. tox: integración con uv (`tox-uv`)

```bash
uv tool install tox --with tox-uv
```

Con el *plugin* instalado, tox crea los entornos e instala con uv automáticamente. Opciones adicionales:

```toml
requires = ["tox>=4.24", "tox-uv>=1.20"]

[env_run_base]
runner = "uv-venv-lock-runner"       # Instala desde uv.lock
dependency_groups = ["test"]
uv_sync_flags = ["--locked"]
```

| Opción | Uso |
|---|---|
| `runner = "uv-venv-lock-runner"` | Sincroniza el entorno con `uv.lock` (reproducible). |
| `uv_seed = true` | Incluir pip en los entornos. |
| `uv_resolution = "lowest-direct"` | Probar con las **versiones mínimas** declaradas de las dependencias. |

> `uv_resolution = "lowest-direct"` es muy útil en librerías: verifica que los límites inferiores (`httpx>=0.27`) son reales.

---

## 13. Obtener las versiones de Python

nox y tox necesitan encontrar los intérpretes:

| Método | Cómo |
|---|---|
| uv (recomendado) | `uv python install 3.12 3.13 3.14`; con `venv_backend="uv"`/`tox-uv` se descargan solas. |
| pyenv | `pyenv install 3.12 3.13 3.14` y `pyenv global 3.13 3.12 3.14`. |
| Windows | `py install 3.12` (Python install manager); tox y nox usan el lanzador `py`. |
| CI | `actions/setup-python` con varias versiones o `astral-sh/setup-uv`. |
| Contenedor | Imagen con varias versiones (p. ej. `ghcr.io/astral-sh/uv`). |

Con `skip_missing_interpreters` (tox) o sin `error_on_missing_interpreters` (nox), las versiones que falten se omiten con un aviso; en CI conviene que fallen.

---

## 14. nox y tox en CI

### 14.1 GitHub Actions con matriz

```yaml
jobs:
  pruebas:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python: ["3.12", "3.13", "3.14"]
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
        with:
          python-version: ${{ matrix.python }}
      - run: uvx nox -s tests-${{ matrix.python }}
      # Con tox: uvx --with tox-uv tox -e ${{ matrix.python }}

  calidad:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
      - run: uvx nox -t calidad
```

Acción oficial de nox (instala nox y todas las versiones de Python):

```yaml
- uses: wntrblm/nox@2026.10.20
  with:
    python-versions: "3.12, 3.13, 3.14"
- run: nox
```

### 14.2 GitLab CI

```yaml
pruebas:
  image: ghcr.io/astral-sh/uv:bookworm-slim
  parallel:
    matrix:
      - PYTHON: ["3.12", "3.13", "3.14"]
  script:
    - uvx nox -s tests-$PYTHON
```

> Separe la matriz en trabajos de CI (paralelismo y resultados por versión) y use nox/tox dentro de cada uno: así los desarrolladores ejecutan **exactamente** lo mismo en local.

---

## 15. Ejemplo práctico completo

**`noxfile.py` de un proyecto profesional** con uv: pruebas en tres versiones de Python con cobertura paralela y combinación final, pruebas con las versiones mínimas de las dependencias, *lint*, tipos, seguridad, documentación, construcción y verificación del paquete, y una sesión de desarrollo. Incluye el `tox.toml` equivalente para comparar.

### 15.1 `noxfile.py`

```python
"""
Tareas del proyecto «tienda».

    nox -l                    # Lista de sesiones
    nox                       # Calidad + pruebas en todas las versiones + cobertura
    nox -s tests-3.13 -- -x   # Pruebas en 3.13, parar en el primer fallo
    nox -t calidad            # lint, tipos y seguridad
"""
from __future__ import annotations

import shutil
from pathlib import Path

import nox

nox.needs_version = ">=2025.5.1"
nox.options.default_venv_backend = "uv"
nox.options.reuse_existing_virtualenvs = True
nox.options.error_on_missing_interpreters = True

PYTHONS = ["3.12", "3.13", "3.14"]
PRINCIPAL = "3.13"
RAIZ = Path(__file__).parent


def sincronizar(session: nox.Session, *grupos: str, extras: bool = True) -> None:
    """Instala en la sesión exactamente lo bloqueado en uv.lock."""
    argumentos = ["uv", "sync", "--locked", "--no-default-groups"]
    for grupo in grupos:
        argumentos += ["--group", grupo]
    if extras:
        argumentos.append("--all-extras")
    session.run_install(*argumentos, env={"UV_PROJECT_ENVIRONMENT": session.virtualenv.location,
                                          "UV_PYTHON": str(session.python)})


# ------------------------------------------------------------------ Calidad
@nox.session(python=PRINCIPAL, tags=["calidad"])
def lint(session: nox.Session) -> None:
    """Ruff: análisis y formato."""
    session.install("ruff")
    fix = "--fix" in session.posargs
    session.run("ruff", "check", ".", *(["--fix"] if fix else []))
    session.run("ruff", "format", *([] if fix else ["--check"]), ".")


@nox.session(python=PRINCIPAL, tags=["calidad"])
def tipos(session: nox.Session) -> None:
    """mypy en modo estricto."""
    sincronizar(session, "lint", "test")
    session.run("mypy", *session.posargs)


@nox.session(python=PRINCIPAL, tags=["calidad"])
def seguridad(session: nox.Session) -> None:
    """Bandit sobre el código y pip-audit sobre el bloqueo."""
    session.install("bandit[toml]", "pip-audit")
    session.run("bandit", "-c", "pyproject.toml", "-r", "src", "-ll", "-ii", "-q")
    requisitos = Path(session.create_tmp()) / "requisitos.txt"
    session.run("uv", "export", "--no-emit-project", "--no-hashes", "--quiet", "-o", str(requisitos),
                external=True)
    session.run("pip-audit", "-r", str(requisitos), "--no-deps", "--disable-pip", "--progress-spinner", "off")


# ------------------------------------------------------------------ Pruebas
@nox.session(python=PYTHONS, tags=["pruebas"])
def tests(session: nox.Session) -> None:
    """pytest con cobertura (un archivo de datos por versión)."""
    sincronizar(session, "test")
    session.run(
        "pytest", "--cov", "--cov-report=", *session.posargs,
        env={"COVERAGE_FILE": f".coverage.{session.python}"},
    )
    session.notify("cobertura")


@nox.session(python=PYTHONS[0], tags=["pruebas"])
def tests_minimas(session: nox.Session) -> None:
    """Pruebas con las versiones MÍNIMAS declaradas de las dependencias."""
    session.run_install("uv", "pip", "install", "--resolution", "lowest-direct", ".", "--group", "test",
                        env={"VIRTUAL_ENV": session.virtualenv.location})
    session.run("pytest", "-q", "-p", "no:cacheprovider", *session.posargs)


@nox.session(python=PRINCIPAL, default=False)
def cobertura(session: nox.Session) -> None:
    """Combina la cobertura de todas las versiones y aplica el umbral."""
    session.install("coverage[toml]")
    if not list(RAIZ.glob(".coverage.*")):
        session.skip("No hay datos de cobertura: ejecute antes la sesión «tests»")
    session.run("coverage", "combine")
    session.run("coverage", "report", "--fail-under=90")
    session.run("coverage", "xml")
    if session.interactive:
        session.run("coverage", "html")
        session.log("Informe HTML en htmlcov/index.html")


# ------------------------------------------------------------------ Documentación y paquete
@nox.session(python=PRINCIPAL, tags=["docs"])
def docs(session: nox.Session) -> None:
    """Construye la documentación (o la sirve con «-- serve»)."""
    sincronizar(session, "docs")
    if session.posargs and session.posargs[0] == "serve":
        session.run("mkdocs", "serve")
    else:
        session.run("mkdocs", "build", "--strict")


@nox.session(python=PRINCIPAL, tags=["paquete"])
def construir(session: nox.Session) -> None:
    """Construye el paquete, verifica sus metadatos y prueba a instalarlo."""
    shutil.rmtree(RAIZ / "dist", ignore_errors=True)
    session.run("uv", "build", external=True)
    session.install("twine")
    session.run("twine", "check", "--strict", *map(str, (RAIZ / "dist").glob("*")))
    rueda = next((RAIZ / "dist").glob("*.whl"))
    session.install(str(rueda))
    session.run("python", "-c", "import tienda, importlib.metadata as m; print(m.version('tienda'))")


# ------------------------------------------------------------------ Utilidades
@nox.session(python=False, default=False)
def limpiar(session: nox.Session) -> None:
    """Elimina artefactos generados."""
    for patron in ["dist", "build", "htmlcov", ".coverage*", "coverage.xml", "site", ".nox", "*.egg-info"]:
        for ruta in RAIZ.glob(patron):
            session.log(f"Eliminando {ruta.name}")
            shutil.rmtree(ruta, ignore_errors=True) if ruta.is_dir() else ruta.unlink()
```

```text
$ nox -l
Sessions defined in /proyecto/noxfile.py:

* lint [calidad] -> Ruff: análisis y formato.
* tipos [calidad] -> mypy en modo estricto.
* seguridad [calidad] -> Bandit sobre el código y pip-audit sobre el bloqueo.
* tests-3.12 [pruebas] -> pytest con cobertura (un archivo de datos por versión).
* tests-3.13 [pruebas] -> pytest con cobertura (un archivo de datos por versión).
* tests-3.14 [pruebas] -> pytest con cobertura (un archivo de datos por versión).
* tests_minimas [pruebas] -> Pruebas con las versiones MÍNIMAS declaradas de las dependencias.
- cobertura -> Combina la cobertura de todas las versiones y aplica el umbral.
* docs [docs] -> Construye la documentación (o la sirve con «-- serve»).
* construir [paquete] -> Construye el paquete, verifica sus metadatos y prueba a instalarlo.
- limpiar -> Elimina artefactos generados.

sessions marked with * are selected, sessions marked with - are skipped.
```

`tox list` con el `tox.toml` equivalente (sección 15.3):

```text
default environments:
lint      -> Ruff: análisis y formato
tipos     -> mypy en modo estricto
seguridad -> Bandit y pip-audit
3.12      -> Pruebas con 3.12
3.13      -> Pruebas con 3.13
3.14      -> Pruebas con 3.14
minimas   -> Pruebas con las versiones mínimas de las dependencias
cobertura -> Combina la cobertura y aplica el umbral

additional environments:
docs      -> Documentación
```

### 15.2 Configuración de cobertura para combinar

```toml
[tool.coverage.run]
source = ["tienda"]
branch = true
relative_files = true

[tool.coverage.paths]
source = ["src/tienda", "*/site-packages/tienda"]
```

### 15.3 `tox.toml` equivalente (con `tox-uv`)

```toml
requires = ["tox>=4.24", "tox-uv>=1.20"]
env_list = ["lint", "tipos", "seguridad", "3.12", "3.13", "3.14", "minimas", "cobertura"]
skip_missing_interpreters = false

[env_run_base]
description = "Pruebas con {base_python}"
runner = "uv-venv-lock-runner"
dependency_groups = ["test"]
extras = ["pdf", "excel"]
set_env = { COVERAGE_FILE = "{work_dir}/.coverage.{env_name}" }
commands = [["pytest", "--cov", "--cov-report=", { replace = "posargs", extend = true }]]

[env.minimas]
description = "Pruebas con las versiones mínimas de las dependencias"
base_python = ["3.12"]
runner = "uv-venv-runner"
uv_resolution = "lowest-direct"
set_env = {}
commands = [["pytest", "-q", { replace = "posargs", extend = true }]]

[env.cobertura]
description = "Combina la cobertura y aplica el umbral"
depends = ["3.12", "3.13", "3.14"]
skip_install = true
deps = ["coverage[toml]"]
set_env = { COVERAGE_FILE = "{work_dir}/.coverage" }
commands = [
    ["coverage", "combine", "{work_dir}"],
    ["coverage", "report", "--fail-under=90"],
    ["coverage", "xml"],
]

[env.lint]
description = "Ruff: análisis y formato"
skip_install = true
deps = ["ruff"]
commands = [["ruff", "check", "."], ["ruff", "format", "--check", "."]]

[env.tipos]
description = "mypy en modo estricto"
dependency_groups = ["lint", "test"]
commands = [["mypy"]]

[env.seguridad]
description = "Bandit y pip-audit"
skip_install = true
deps = ["bandit[toml]", "pip-audit"]
allowlist_externals = ["uv"]
commands = [
    ["bandit", "-c", "pyproject.toml", "-r", "src", "-ll", "-ii", "-q"],
    ["uv", "export", "--no-emit-project", "--no-hashes", "--quiet", "-o", "{env_tmp_dir}/req.txt"],
    ["pip-audit", "-r", "{env_tmp_dir}/req.txt", "--no-deps", "--disable-pip"],
]

[env.docs]
description = "Documentación"
dependency_groups = ["docs"]
commands = [["mkdocs", "build", "--strict"]]

[labels]
calidad = ["lint", "tipos", "seguridad"]
pruebas = ["3.12", "3.13", "3.14", "minimas"]
```

```bash
tox p -m pruebas          # Pruebas en paralelo
tox -e cobertura          # Combinar
tox -m calidad
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Python interpreter 3.12 not found` (nox) / `skipped because could not find python interpreter` (tox) | Versión de Python no instalada. | `uv python install 3.12` o use el *backend* uv; en CI, instale todas las versiones. |
| `Command X not found in the virtualenv` (nox) / `X is not allowed, use allowlist_externals` (tox) | Programa externo no permitido. | `external=True` (nox) o `allowlist_externals` (tox). |
| Cambios en las dependencias no se reflejan | Entorno reutilizado. | `nox` sin `-R`, o `tox -r`. |
| Las pruebas pasan en tox pero fallan instaladas (o al revés) | Diferencias entre el código fuente y el paquete construido (archivos de datos no incluidos). | Revise la configuración del *backend*; es justo lo que tox debe detectar. |
| `coverage combine` no encuentra datos | Archivos `.coverage.*` en otra ruta o con rutas absolutas distintas. | `COVERAGE_FILE` y `relative_files = true` + `[tool.coverage.paths]`. |
| tox lee una configuración inesperada | Hay varios archivos (`tox.ini` y `tox.toml`). | Mantenga uno solo; `tox config` muestra la efectiva. |
| `{posargs}` no recibe argumentos | Faltan los dos guiones `--`. | `tox -e 3.13 -- -x`, `nox -s tests -- -x`. |
| Ejecución muy lenta | Entornos recreados en cada ejecución con pip. | uv como *backend* y reutilización de entornos en local. |

---

## 17. Buenas prácticas

- Use nox **o** tox (no ambos) como única fuente de verdad de las tareas del proyecto.
- Defina sesiones/entornos para **todas** las comprobaciones: pruebas, *lint*, tipos, seguridad, documentación, construcción.
- Use **uv** como *backend*: los entornos se crean en segundos y las versiones de Python se descargan solas.
- Instale desde el **archivo de bloqueo** para que local y CI usen las mismas versiones.
- Pruebe en **todas las versiones** de `requires-python` y, en librerías, también con las **versiones mínimas** de las dependencias.
- Combine la cobertura de todas las versiones antes de aplicar el umbral.
- En CI, use la matriz del proveedor y llame a nox/tox dentro de cada trabajo.
- Documente cada sesión con un *docstring* (nox) o `description` (tox): aparece en `nox -l` / `tox list`.
- Añada `.nox/` y `.tox/` a `.gitignore`.

---

## 18. Referencia rápida

```python
import nox
nox.options.default_venv_backend = "uv"; nox.options.sessions = [...]
@nox.session(python=["3.12", "3.13"], tags=["x"], reuse_venv=True, default=True)
@nox.parametrize("arg", ["a", "b"])
def s(session): session.install(".[test]"); session.run("pytest", *session.posargs); session.notify("otra")
```

```bash
nox [-l] [-s sesion[-3.13]] [-t etiqueta] [-k expr] [-p 3.13] [-r|-R] [-- args]
tox [list] [-e env1,env2] [-m etiqueta] [-f factor] [p] [-r] [config -e env] [-- args]
```

```toml
# tox.toml
env_list = ["3.12", "3.13", "lint"]
[env_run_base]  extras, deps, dependency_groups, commands = [["pytest", { replace = "posargs", extend = true }]]
[env.lint]      skip_install = true · deps = ["ruff"] · commands = [["ruff", "check", "."]]
[labels]        calidad = ["lint", "tipos"]
```

---

## 19. Ejercicios propuestos

1. Cree un `noxfile.py` con sesiones `tests` (tres versiones de Python) y `lint`, y ejecútelo con el *backend* uv.
2. Parametrice una sesión para probar contra dos versiones mayores de una dependencia (p. ej. Pydantic o Django).
3. Traduzca su `noxfile.py` a `tox.toml` y compare la legibilidad y el tiempo de ejecución con `tox p`.
4. Añada una sesión que pruebe con `--resolution lowest-direct` y corrija los límites inferiores que fallen.
5. Configure GitHub Actions con una matriz de sistemas operativos y versiones que llame a la sesión correspondiente.

---

## 20. Referencias

- nox, documentación: https://nox.thea.codes/
- nox, referencia de `Session`: https://nox.thea.codes/en/stable/usage.html
- tox, documentación: https://tox.wiki/
- tox, configuración: https://tox.wiki/en/latest/config.html
- tox-uv: https://github.com/tox-dev/tox-uv
- uv con nox: https://docs.astral.sh/uv/
