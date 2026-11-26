# Manual de uso: pipx

> **Categoría:** Herramientas de terminal · **Utilidad:** Instalar aplicaciones de línea de comandos, cada una en un entorno aislado
> **Tipo:** Paquete externo (requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Instalar aplicaciones](#4-instalar-aplicaciones)
5. [Ejecutar sin instalar (`pipx run`)](#5-ejecutar-sin-instalar-pipx-run)
6. [Consultar, actualizar y desinstalar](#6-consultar-actualizar-y-desinstalar)
7. [Inyectar paquetes en una aplicación](#7-inyectar-paquetes-en-una-aplicación)
8. [Versiones de Python](#8-versiones-de-python)
9. [Ubicaciones y variables de entorno](#9-ubicaciones-y-variables-de-entorno)
10. [Scripts con dependencias en línea (PEP 723)](#10-scripts-con-dependencias-en-línea-pep-723)
11. [Distribuir su propia herramienta para pipx](#11-distribuir-su-propia-herramienta-para-pipx)
12. [pipx frente a `uv tool`](#12-pipx-frente-a-uv-tool)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**pipx** instala **aplicaciones de línea de comandos** escritas en Python (por ejemplo `ruff`, `black`, `httpie`, `poetry`, `cookiecutter`) de forma que:

- cada aplicación vive en **su propio entorno virtual**, sin conflictos de dependencias entre ellas;
- sus ejecutables quedan disponibles **globalmente** en el `PATH`;
- no se toca el Python del sistema.

**Ventajas:**
- Elimina los conflictos entre herramientas globales.
- Evita el error `externally-managed-environment` de las distribuciones modernas.
- Permite ejecutar una herramienta una sola vez sin instalarla (`pipx run`).
- Actualización y desinstalación limpias.

**Limitaciones:**
- Solo sirve para **aplicaciones** con ejecutables, no para librerías que vaya a importar en su código.
- Más lento que `uv tool`, su equivalente moderno.

| Si necesita… | Opción |
|---|---|
| Instalar una herramienta de terminal global | **pipx** / `uv tool install` |
| Usar una librería en su proyecto | pip / uv / Poetry dentro de un entorno virtual |
| Ejecutar una herramienta una vez | `pipx run` / `uvx` |
| Fijar la versión de una herramienta por proyecto | Dependencia de desarrollo del proyecto o pre-commit |

---

## 2. Instalación

| Sistema | Comando |
|---|---|
| macOS (Homebrew) | `brew install pipx` |
| Debian / Ubuntu | `sudo apt install pipx` |
| Fedora | `sudo dnf install pipx` |
| Windows (Scoop) | `scoop install pipx` |
| Cualquiera (con pip, en el usuario) | `python -m pip install --user pipx` |

Añada el directorio de ejecutables al `PATH`:

```bash
pipx ensurepath
```

Cierre y vuelva a abrir la terminal, y compruebe:

```bash
pipx --version
```

Autocompletado en la terminal:

```bash
pipx completions        # Muestra las instrucciones para su shell
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Aplicación | Paquete de PyPI que define ejecutables (*entry points*). |
| Entorno de la aplicación | Entorno virtual propio, en `~/.local/share/pipx/venvs/<app>` (Linux). |
| Directorio de binarios | Carpeta con enlaces a los ejecutables, añadida al `PATH` (`~/.local/bin`). |
| `pipx run` | Ejecuta una aplicación en un entorno temporal en caché. |
| Inyección | Añadir paquetes extra al entorno de una aplicación (plugins). |
| Entorno compartido | Entorno con pip que pipx reutiliza para crear los demás. |

---

## 4. Instalar aplicaciones

```bash
pipx install ruff
pipx install httpie
pipx install "poetry==2.4.1"                      # Versión exacta
pipx install "black>=24,<25"                      # Rango
pipx install "mkdocs[i18n]"                       # Con extras
pipx install git+https://github.com/org/herramienta.git
pipx install ./mi_herramienta                     # Ruta local
pipx install -e ./mi_herramienta                  # Editable (desarrollo)
pipx install --suffix=@1 "poetry==1.8.5"          # Dos versiones a la vez: poetry y poetry@1
pipx install --force ruff                         # Reinstalar
```

Paquetes con varios ejecutables o ejecutables de dependencias:

```bash
pipx install --include-deps jupyter               # Expone también los ejecutables de las dependencias
```

Tras instalar, el comando está disponible en cualquier terminal:

```bash
ruff --version
http GET https://httpbin.org/get
```

---

## 5. Ejecutar sin instalar (`pipx run`)

```bash
pipx run cowsay -t "Hola"
pipx run cookiecutter gh:org/plantilla
pipx run --spec "black==24.10.0" black --check .
pipx run --spec httpie http GET example.com       # El paquete y el comando tienen nombres distintos
pipx run --python 3.12 pycowsay hola
pipx run --no-cache herramienta                   # Ignorar la caché
```

> Los entornos de `pipx run` se guardan en caché unos días: la segunda ejecución es inmediata.

Úselo para herramientas que necesita ocasionalmente (generadores de proyectos, conversores, utilidades puntuales) sin acumular instalaciones.

---

## 6. Consultar, actualizar y desinstalar

```bash
pipx list                         # Aplicaciones, versiones y ejecutables
pipx list --short                 # Solo nombre y versión
pipx list --json                  # Para scripts

pipx upgrade ruff                 # Actualizar una
pipx upgrade-all                  # Actualizar todas
pipx upgrade --include-injected httpie   # Incluir los paquetes inyectados

pipx uninstall ruff
pipx uninstall-all

pipx reinstall ruff               # Recrear el entorno (p. ej., tras actualizar Python)
pipx reinstall-all
```

Ejecutar pip dentro del entorno de una aplicación:

```bash
pipx runpip httpie list
pipx runpip httpie install -U httpie-oauth
```

---

## 7. Inyectar paquetes en una aplicación

Algunas aplicaciones aceptan *plugins* que deben instalarse **en su mismo entorno**:

```bash
pipx install poetry
pipx inject poetry poetry-plugin-export

pipx install mkdocs
pipx inject mkdocs mkdocs-material mkdocstrings[python]

pipx install httpie
pipx inject httpie httpie-jwt-auth

pipx inject --include-apps ansible-core ansible-lint     # Exponer también sus ejecutables
pipx uninject mkdocs mkdocstrings
```

---

## 8. Versiones de Python

Por defecto, pipx usa el Python con el que se instaló. Para elegir otro:

```bash
pipx install --python python3.12 herramienta
pipx install --python 3.12 herramienta              # pipx busca el intérprete
pipx install --python 3.13 --fetch-missing-python herramienta    # Lo descarga si falta
```

Definir el Python por defecto:

```bash
export PIPX_DEFAULT_PYTHON=/usr/bin/python3.13
```

Tras actualizar el Python base (por ejemplo, de 3.12 a 3.13), los entornos dejan de funcionar:

```bash
pipx reinstall-all
```

---

## 9. Ubicaciones y variables de entorno

```bash
pipx environment                 # Muestra todas las rutas y variables
```

| Variable | Uso | Valor por defecto (Linux) |
|---|---|---|
| `PIPX_HOME` | Entornos de las aplicaciones | `~/.local/share/pipx` |
| `PIPX_BIN_DIR` | Ejecutables | `~/.local/bin` |
| `PIPX_MAN_DIR` | Páginas de manual | `~/.local/share/man` |
| `PIPX_DEFAULT_PYTHON` | Python para nuevas instalaciones | El de pipx |
| `PIPX_GLOBAL_HOME` / `PIPX_GLOBAL_BIN_DIR` | Instalaciones globales (`--global`) | `/opt/pipx`, `/usr/local/bin` |

Instalación para todos los usuarios de la máquina:

```bash
sudo pipx install --global ruff
```

Uso de un índice privado:

```bash
pipx install --index-url https://pypi.empresa.local/simple herramienta-interna
pipx install --pip-args="--trusted-host pypi.empresa.local" herramienta-interna
```

---

## 10. Scripts con dependencias en línea (PEP 723)

`pipx run` ejecuta scripts que declaran sus dependencias en un bloque de metadatos al inicio del archivo:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "httpx>=0.27",
#   "rich>=13",
# ]
# ///
import httpx
from rich import print

respuesta = httpx.get("https://api.github.com/repos/pypa/pipx")
datos = respuesta.json()
print(f"[bold]pipx[/bold] tiene [green]{datos['stargazers_count']}[/green] estrellas")
```

```bash
pipx run estrellas.py
```

pipx crea un entorno temporal con las dependencias indicadas y ejecuta el script. Es ideal para compartir utilidades de un solo archivo.

---

## 11. Distribuir su propia herramienta para pipx

Para que su paquete sea instalable con pipx, basta con declarar **scripts** en `pyproject.toml`:

```toml
[project]
name = "limpia-csv"
version = "1.0.0"
requires-python = ">=3.12"
dependencies = ["typer>=0.12", "rich>=13"]

[project.scripts]
limpia-csv = "limpia_csv.cli:app"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

```bash
pipx install .                                       # Desde la carpeta del proyecto
pipx install git+https://github.com/org/limpia-csv   # Desde el repositorio
pipx install limpia-csv                              # Desde PyPI, una vez publicado
```

---

## 12. pipx frente a `uv tool`

| Acción | pipx | uv |
|---|---|---|
| Instalar | `pipx install ruff` | `uv tool install ruff` |
| Ejecutar sin instalar | `pipx run ruff` | `uvx ruff` (`uv tool run ruff`) |
| Listar | `pipx list` | `uv tool list` |
| Actualizar todas | `pipx upgrade-all` | `uv tool upgrade --all` |
| Plugins | `pipx inject app plugin` | `uv tool install app --with plugin` |
| Ejecutar script PEP 723 | `pipx run script.py` | `uv run script.py` |

> Ambos son válidos. Si ya usa **uv** en sus proyectos, `uv tool` evita tener otra herramienta y es considerablemente más rápido.

---

## 13. Ejemplo práctico completo

**Aprovisionar la estación de trabajo de un desarrollador** con un conjunto estándar de herramientas a partir de un archivo de configuración versionado. El script instala lo que falta, actualiza lo existente, aplica los *plugins* y genera un informe.

Archivo `herramientas.toml`:

```toml
[herramientas.ruff]
version = ">=0.8"

[herramientas.poetry]
version = ">=2,<3"
plugins = ["poetry-plugin-export"]

[herramientas.mkdocs]
plugins = ["mkdocs-material", "mkdocstrings[python]"]

[herramientas.httpie]

[herramientas.pre-commit]

[herramientas.cookiecutter]
python = "3.12"
```

Script `aprovisionar.py`:

```python
"""
Instala y actualiza con pipx las herramientas declaradas en herramientas.toml.
Requiere Python 3.11+ (tomllib) y pipx en el PATH.
"""
from __future__ import annotations

import json
import shutil
import subprocess
import sys
import tomllib
from dataclasses import dataclass, field
from pathlib import Path


@dataclass
class Herramienta:
    nombre: str
    version: str = ""
    python: str | None = None
    plugins: list[str] = field(default_factory=list)

    @property
    def especificacion(self) -> str:
        return f"{self.nombre}{self.version}"


def pipx(*args: str, capturar: bool = False) -> subprocess.CompletedProcess[str]:
    return subprocess.run(["pipx", *args], check=True, text=True, capture_output=capturar)


def instaladas() -> dict[str, dict]:
    datos = json.loads(pipx("list", "--json", capturar=True).stdout)
    return {
        nombre: info["metadata"]
        for nombre, info in datos.get("venvs", {}).items()
    }


def cargar(ruta: Path) -> list[Herramienta]:
    config = tomllib.loads(ruta.read_text(encoding="utf-8"))
    return [Herramienta(nombre=n, **(opciones or {})) for n, opciones in config["herramientas"].items()]


def asegurar(h: Herramienta, actuales: dict[str, dict]) -> str:
    if h.nombre in actuales:
        pipx("upgrade", "--include-injected", h.nombre)
        accion = "actualizada"
    else:
        args = ["install", h.especificacion]
        if h.python:
            args += ["--python", h.python]
        pipx(*args)
        accion = "instalada"

    inyectados = set(actuales.get(h.nombre, {}).get("injected_packages", {}))
    pendientes = [p for p in h.plugins if p.split("[")[0] not in inyectados]
    if pendientes:
        pipx("inject", h.nombre, *pendientes)
        accion += f" (+{len(pendientes)} plugins)"
    return accion


def main() -> int:
    if shutil.which("pipx") is None:
        print("✖ pipx no está instalado. Consulte https://pipx.pypa.io/")
        return 1

    herramientas = cargar(Path(__file__).with_name("herramientas.toml"))
    actuales = instaladas()
    errores: list[str] = []

    for h in herramientas:
        try:
            print(f"✔ {h.nombre:<14} {asegurar(h, actuales)}")
        except subprocess.CalledProcessError as e:
            print(f"✖ {h.nombre:<14} error (código {e.returncode})")
            errores.append(h.nombre)

    print("\nResumen:")
    for nombre, meta in sorted(instaladas().items()):
        version = meta["main_package"]["package_version"]
        print(f"  {nombre:<14} {version}")

    if errores:
        print(f"\n{len(errores)} herramientas con errores: {', '.join(errores)}")
    return 1 if errores else 0


if __name__ == "__main__":
    sys.exit(main())
```

Uso:

```bash
pipx ensurepath
python aprovisionar.py
```

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `command not found` tras instalar | `~/.local/bin` no está en el `PATH`. | `pipx ensurepath` y reabrir la terminal. |
| `No apps associated with package` | El paquete es una librería, no tiene ejecutables. | Instálelo en un entorno virtual del proyecto; o use `--include-deps` si el ejecutable es de una dependencia. |
| Las herramientas fallan tras actualizar Python | Los entornos apuntan al intérprete antiguo. | `pipx reinstall-all`. |
| `pipx run` ejecuta el comando equivocado | El nombre del paquete y del ejecutable difieren. | `pipx run --spec paquete comando`. |
| Un *plugin* no tiene efecto | Se instaló con pip fuera del entorno de la aplicación. | `pipx inject app plugin`. |
| `externally-managed-environment` al instalar pipx | Se intentó `pip install pipx` en el Python del sistema. | Instale pipx con el gestor de paquetes del sistema. |
| Conflicto de nombres entre dos versiones | Se instaló la misma app dos veces. | Use `--suffix` para mantener ambas. |

---

## 15. Buenas prácticas

- Use pipx (o `uv tool`) para **todas** las herramientas de terminal globales; nunca `sudo pip install`.
- No instale con pipx las herramientas cuya versión deba coincidir en todo el equipo (Ruff, mypy, pytest): declárelas como dependencias de desarrollo del proyecto o en `pre-commit`.
- Use `pipx run` para herramientas ocasionales.
- Use `pipx inject` para los *plugins*; no use `pip` directamente en sus entornos.
- Ejecute `pipx upgrade-all` periódicamente y `pipx reinstall-all` tras actualizar Python.
- Versione la lista de herramientas del equipo (como en el ejemplo) para aprovisionar equipos nuevos.
- Si ya trabaja con uv, prefiera `uv tool` y `uvx` por velocidad y para usar una sola herramienta.

---

## 16. Referencia rápida

```bash
pipx ensurepath
pipx install app / "app==1.2" / "app[extra]" / git+URL / -e ./ruta  [--python 3.12] [--suffix=@1]
pipx run [--spec paquete] comando [args]   ·   pipx run script.py
pipx list [--short | --json]
pipx upgrade app / upgrade-all
pipx inject app plugin  ·  uninject app plugin
pipx runpip app list
pipx reinstall-all
pipx uninstall app / uninstall-all
pipx environment
```

---

## 17. Ejercicios propuestos

1. Instale `ruff` y `httpie` con pipx y compruebe con `pipx list` que cada uno tiene su propio entorno.
2. Ejecute `cowsay` con `pipx run` sin instalarlo y localice el entorno en caché.
3. Instale MkDocs e inyecte `mkdocs-material`. Cree un sitio de prueba y sírvalo.
4. Escriba un script con metadatos PEP 723 que consulte una API pública y ejecútelo con `pipx run`.
5. Amplíe `aprovisionar.py` para desinstalar las herramientas que ya no aparecen en `herramientas.toml`.

---

## 18. Referencias

- Documentación oficial: https://pipx.pypa.io/
- Repositorio: https://github.com/pypa/pipx
- PEP 723 (metadatos de scripts en línea): https://peps.python.org/pep-0723/
- Herramientas con uv: https://docs.astral.sh/uv/guides/tools/
