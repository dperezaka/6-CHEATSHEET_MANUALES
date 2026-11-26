# Manual de uso: venv

> **Categoría:** Entornos virtuales · **Utilidad:** Crear entornos aislados por proyecto
> **Tipo:** Módulo de la librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y comprobación](#2-instalación-y-comprobación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Crear un entorno virtual](#4-crear-un-entorno-virtual)
5. [Activar y desactivar](#5-activar-y-desactivar)
6. [Trabajar sin activar](#6-trabajar-sin-activar)
7. [Estructura interna del entorno](#7-estructura-interna-del-entorno)
8. [Opciones de creación](#8-opciones-de-creación)
9. [Elegir la versión de Python](#9-elegir-la-versión-de-python)
10. [Integración con editores e IDE](#10-integración-con-editores-e-ide)
11. [Detectar el entorno desde Python](#11-detectar-el-entorno-desde-python)
12. [Recrear, mover y eliminar entornos](#12-recrear-mover-y-eliminar-entornos)
13. [Crear entornos desde código](#13-crear-entornos-desde-código)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

**venv** crea **entornos virtuales**: directorios con su propio intérprete de Python (enlazado al original) y su propio `site-packages`. Cada proyecto tiene así sus dependencias aisladas, sin interferir con otros proyectos ni con el Python del sistema.

**Ventajas:**
- Incluido en la librería estándar desde Python 3.3.
- Ligero y rápido de crear.
- Funciona igual en Windows, macOS y Linux.
- Es el formato estándar que reconocen pip, uv, Poetry, los IDE y las herramientas de CI.

**Limitaciones:**
- No instala ni gestiona versiones de Python: usa una que ya exista en el sistema.
- No gestiona dependencias ni archivos de bloqueo (se combina con pip o se sustituye por uv/Poetry).
- Los entornos no son portables: no se pueden copiar a otra ruta ni a otra máquina.

| Si necesita… | Opción |
|---|---|
| Aislar dependencias, sin herramientas adicionales | **venv** |
| Crear entornos mucho más rápido | `uv venv` |
| Gestionar entorno + dependencias + bloqueo | Poetry / uv |
| Varias versiones de Python | pyenv / `uv python` / lanzador `py` (Windows) |
| Entornos con paquetes no Python (CUDA, GDAL…) | conda / mamba / pixi |

---

## 2. Instalación y comprobación

venv viene con Python. Compruebe que está disponible:

```bash
python -m venv --help
```

En Debian/Ubuntu puede venir en un paquete aparte:

```bash
sudo apt install python3-venv          # o python3.13-venv para una versión concreta
```

> En este manual se usa `python`. En Linux/macOS puede ser `python3`; en Windows, `py`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Entorno virtual | Directorio con un intérprete y un `site-packages` propios. |
| Intérprete base | Python a partir del cual se creó el entorno (`sys.base_prefix`). |
| Activación | Modifica el `PATH` de la terminal para que `python` y `pip` apunten al entorno. |
| `site-packages` | Carpeta donde se instalan los paquetes del entorno. |
| `pyvenv.cfg` | Archivo de configuración que identifica el directorio como entorno virtual. |
| Paquetes del sistema | Paquetes del intérprete base; no son visibles por defecto. |

**Idea clave:** un entorno virtual es **desechable**. Lo que se conserva y versiona es la **lista de dependencias** (`pyproject.toml`, `requirements.txt`), no el entorno.

---

## 4. Crear un entorno virtual

```bash
cd mi_proyecto
python -m venv .venv
```

> El nombre `.venv` es la convención más extendida: lo detectan automáticamente VS Code, PyCharm, uv, Poetry y muchas otras herramientas, y queda oculto en Linux/macOS.

Añada el entorno a `.gitignore`:

```gitignore
.venv/
```

> Desde Python 3.13, `venv` crea automáticamente un `.gitignore` dentro del entorno que excluye todo su contenido.

---

## 5. Activar y desactivar

| Sistema / terminal | Activar |
|---|---|
| Linux / macOS (bash, zsh) | `source .venv/bin/activate` |
| fish | `source .venv/bin/activate.fish` |
| csh / tcsh | `source .venv/bin/activate.csh` |
| Windows (PowerShell) | `.venv\Scripts\Activate.ps1` |
| Windows (cmd) | `.venv\Scripts\activate.bat` |
| Windows (Git Bash) | `source .venv/Scripts/activate` |

Al activar, el indicador de la terminal muestra el nombre del entorno:

```text
(.venv) usuario@equipo:~/mi_proyecto$
```

Comprobar:

```bash
which python          # Linux/macOS → /home/usuario/mi_proyecto/.venv/bin/python
where python          # Windows
python -c "import sys; print(sys.prefix)"
```

Desactivar:

```bash
deactivate
```

### 5.1 PowerShell: política de ejecución

Si PowerShell bloquea el script de activación:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

---

## 6. Trabajar sin activar

Activar es una comodidad, no un requisito. Puede invocar directamente el intérprete del entorno:

```bash
.venv/bin/python script.py                    # Linux/macOS
.venv/bin/python -m pip install requests
.venv/bin/pytest

.venv\Scripts\python.exe script.py            # Windows
```

> En scripts de automatización, `Makefile`, tareas programadas (cron) y servicios (systemd), **no active**: use la ruta absoluta del intérprete. Es más explícito y no depende de la terminal.

```ini
# /etc/systemd/system/mi_app.service
[Service]
WorkingDirectory=/opt/mi_app
ExecStart=/opt/mi_app/.venv/bin/python -m mi_app
```

```cron
0 3 * * * /opt/mi_app/.venv/bin/python /opt/mi_app/tareas/limpieza.py
```

---

## 7. Estructura interna del entorno

```
.venv/
├── pyvenv.cfg              # Configuración del entorno
├── bin/                    # Scripts/ en Windows
│   ├── activate            # Scripts de activación
│   ├── python -> python3.13
│   ├── pip
│   └── (ejecutables de los paquetes instalados: pytest, ruff…)
├── include/
└── lib/python3.13/site-packages/   # Lib/site-packages en Windows
```

Contenido de `pyvenv.cfg`:

```ini
home = /usr/local/bin
include-system-site-packages = false
version = 3.13.1
executable = /usr/local/bin/python3.13
command = /usr/local/bin/python3.13 -m venv /home/usuario/mi_proyecto/.venv
```

> El campo `home` apunta al intérprete base. Si ese Python se desinstala o actualiza a otra versión menor, el entorno deja de funcionar: hay que recrearlo.

---

## 8. Opciones de creación

```bash
python -m venv .venv --prompt mi-app             # Nombre mostrado en la terminal
python -m venv .venv --prompt .                  # Usa el nombre de la carpeta actual
python -m venv .venv --upgrade-deps              # Actualiza pip al crear
python -m venv .venv --without-pip               # Sin pip (si va a usar uv, por ejemplo)
python -m venv .venv --system-site-packages      # Ve los paquetes del Python base
python -m venv .venv --clear                     # Vacía un entorno existente y lo recrea
python -m venv .venv --upgrade                   # Actualiza tras subir de parche el Python base
python -m venv .venv --copies                    # Copia binarios en lugar de enlaces
```

| Opción | Cuándo usarla |
|---|---|
| `--prompt` | Distinguir varios entornos en la terminal. |
| `--upgrade-deps` | Siempre al crear: evita trabajar con un pip antiguo. |
| `--system-site-packages` | Aprovechar paquetes del sistema difíciles de compilar (p. ej. en una Raspberry Pi). |
| `--clear` | Empezar de cero sin borrar la carpeta a mano. |
| `--copies` | Sistemas de archivos sin enlaces simbólicos. |

---

## 9. Elegir la versión de Python

venv usa el Python con el que se ejecuta. Para otra versión, invoque ese intérprete:

```bash
python3.12 -m venv .venv                   # Linux/macOS
py -3.12 -m venv .venv                     # Windows (lanzador py)
py --list                                  # Windows: versiones instaladas
~/.pyenv/versions/3.12.8/bin/python -m venv .venv    # Con pyenv
```

Si necesita instalar versiones de Python:

| Herramienta | Comando |
|---|---|
| pyenv (Linux/macOS) | `pyenv install 3.13` |
| Python install manager (Windows) | `py install 3.13` |
| uv (todas) | `uv python install 3.13` y `uv venv --python 3.13` |

---

## 10. Integración con editores e IDE

### 10.1 VS Code

Detecta `.venv` automáticamente. Para fijarlo:

```json
// .vscode/settings.json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true
}
```

O bien: `Ctrl+Shift+P` → **Python: Select Interpreter**.

### 10.2 PyCharm

**Settings → Project → Python Interpreter → Add Interpreter → Existing** y seleccione `.venv/bin/python`.

### 10.3 Activación automática al entrar en la carpeta

Con **direnv** (Linux/macOS):

```bash
# .envrc
source .venv/bin/activate
```

```bash
direnv allow
```

---

## 11. Detectar el entorno desde Python

```python
import os
import sys


def en_entorno_virtual() -> bool:
    return sys.prefix != sys.base_prefix


print("Intérprete:", sys.executable)
print("Prefijo del entorno:", sys.prefix)
print("Python base:", sys.base_prefix)
print("¿Entorno virtual?", en_entorno_virtual())
print("Variable VIRTUAL_ENV:", os.environ.get("VIRTUAL_ENV"))
```

> `VIRTUAL_ENV` solo existe si el entorno se **activó**. La comprobación fiable es `sys.prefix != sys.base_prefix`.

Proteger un script para que no se ejecute fuera del entorno:

```python
import sys

if sys.prefix == sys.base_prefix:
    sys.exit("Ejecute este script dentro del entorno virtual del proyecto.")
```

---

## 12. Recrear, mover y eliminar entornos

### 12.1 Eliminar

```bash
deactivate                 # Si está activo
rm -rf .venv               # Linux/macOS
rmdir /s /q .venv          # Windows (cmd)
Remove-Item -Recurse -Force .venv    # PowerShell
```

### 12.2 Recrear

```bash
rm -rf .venv
python -m venv .venv --upgrade-deps
.venv/bin/python -m pip install -r requirements.txt     # o: pip install -e ".[dev]"
```

### 12.3 Mover o copiar

**No mueva ni copie** un entorno virtual: los scripts de `bin/` contienen rutas absolutas. Recrée el entorno en la nueva ubicación a partir de la lista de dependencias.

### 12.4 Actualizar la versión de Python

- Cambio de **parche** (3.13.0 → 3.13.1): `python -m venv --upgrade .venv`.
- Cambio de **versión menor** (3.12 → 3.13): recree el entorno.

---

## 13. Crear entornos desde código

El módulo expone la clase `venv.EnvBuilder`:

```python
import os
import subprocess
import venv
from pathlib import Path

ruta = Path(".venv")
constructor = venv.EnvBuilder(with_pip=True, upgrade_deps=True, prompt="demo", clear=True)
constructor.create(ruta)

python = ruta / ("Scripts/python.exe" if os.name == "nt" else "bin/python")
subprocess.run([str(python), "-m", "pip", "install", "requests"], check=True)
```

Personalizar la creación heredando de `EnvBuilder`:

```python
import subprocess
import venv


class ConstructorConDependencias(venv.EnvBuilder):
    def __init__(self, requisitos: str, **kwargs):
        super().__init__(with_pip=True, **kwargs)
        self.requisitos = requisitos

    def post_setup(self, context) -> None:
        subprocess.run([context.env_exec_cmd, "-m", "pip", "install", "-r", self.requisitos], check=True)


ConstructorConDependencias("requirements.txt", upgrade_deps=True).create(".venv")
```

---

## 14. Ejemplo práctico completo

**Script de arranque multiplataforma** (`bootstrap.py`) que cualquier miembro del equipo puede ejecutar tras clonar el repositorio. Comprueba la versión de Python, crea o repara `.venv`, instala dependencias solo si han cambiado y muestra un resumen. Solo usa la librería estándar.

```python
"""
bootstrap.py — Prepara el entorno de desarrollo del proyecto.

Uso:
    python bootstrap.py            # Crear o actualizar el entorno
    python bootstrap.py --recrear  # Borrarlo y crearlo desde cero
"""
from __future__ import annotations

import argparse
import hashlib
import os
import shutil
import subprocess
import sys
import venv
from pathlib import Path

RAIZ = Path(__file__).resolve().parent
VENV = RAIZ / ".venv"
REQUISITOS = [RAIZ / "requirements.txt", RAIZ / "requirements-dev.txt"]
MARCA = VENV / ".huella-dependencias"
PYTHON_MINIMO = (3, 12)


def python_del_entorno() -> Path:
    return VENV / ("Scripts/python.exe" if os.name == "nt" else "bin/python")


def ejecutar(*args: str) -> None:
    print("  $", " ".join(args))
    subprocess.run(args, check=True)


def huella() -> str:
    h = hashlib.sha256()
    h.update(sys.version.encode())
    for archivo in REQUISITOS:
        if archivo.exists():
            h.update(archivo.read_bytes())
    return h.hexdigest()


def entorno_valido() -> bool:
    if not python_del_entorno().exists():
        return False
    resultado = subprocess.run([str(python_del_entorno()), "-c", "import sys; print(sys.version_info[:2])"],
                               capture_output=True, text=True)
    return resultado.returncode == 0 and resultado.stdout.strip() == str(sys.version_info[:2])


def crear_entorno() -> None:
    print(f"→ Creando entorno en {VENV.name}/ con Python {sys.version.split()[0]}")
    venv.EnvBuilder(with_pip=True, upgrade_deps=True, clear=True, prompt=RAIZ.name).create(VENV)


def instalar_dependencias() -> None:
    if MARCA.exists() and MARCA.read_text() == huella():
        print("✔ Dependencias al día")
        return
    print("→ Instalando dependencias")
    py = str(python_del_entorno())
    for archivo in REQUISITOS:
        if archivo.exists():
            ejecutar(py, "-m", "pip", "install", "-q", "-r", str(archivo))
    if (RAIZ / "pyproject.toml").exists():
        ejecutar(py, "-m", "pip", "install", "-q", "--no-deps", "-e", str(RAIZ))
    ejecutar(py, "-m", "pip", "check")
    MARCA.write_text(huella())


def resumen() -> None:
    activar = r".venv\Scripts\Activate.ps1" if os.name == "nt" else "source .venv/bin/activate"
    print("\n✔ Entorno listo")
    print(f"  Intérprete: {python_del_entorno()}")
    print(f"  Activar:    {activar}")


def main() -> int:
    parser = argparse.ArgumentParser(description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter)
    parser.add_argument("--recrear", action="store_true", help="Borra el entorno y lo crea de nuevo")
    args = parser.parse_args()

    if sys.version_info < PYTHON_MINIMO:
        print(f"✖ Se requiere Python {'.'.join(map(str, PYTHON_MINIMO))} o superior.")
        return 1
    if sys.prefix != sys.base_prefix:
        print("✖ Ejecute este script con el Python del sistema, no desde un entorno virtual.")
        return 1

    if args.recrear and VENV.exists():
        print("→ Eliminando entorno anterior")
        shutil.rmtree(VENV)
    if not entorno_valido():
        crear_entorno()
    try:
        instalar_dependencias()
    except subprocess.CalledProcessError as e:
        print(f"✖ Falló la instalación (código {e.returncode}).")
        return e.returncode
    resumen()
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Uso:

```bash
python bootstrap.py              # Primera vez: crea e instala
python bootstrap.py              # Siguientes: solo instala si cambian los requisitos
python bootstrap.py --recrear    # Desde cero
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `The virtual environment was not created successfully because ensurepip is not available` | Falta el paquete `python3-venv` (Debian/Ubuntu). | `sudo apt install python3-venv`. |
| `Activate.ps1 cannot be loaded because running scripts is disabled` | Política de ejecución de PowerShell. | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`. |
| `ModuleNotFoundError` con el paquete ya instalado | Se instaló en otro entorno o se ejecuta con otro intérprete. | Compruebe `sys.executable`; use `python -m pip`. |
| `bad interpreter: No such file or directory` | Entorno movido, copiado o Python base desinstalado. | Recrear el entorno. |
| El entorno se ha subido al repositorio | Falta `.venv/` en `.gitignore`. | Añádalo y elimínelo del índice: `git rm -r --cached .venv`. |
| `source: command not found` (Windows) | Sintaxis de Linux en cmd/PowerShell. | Use `.venv\Scripts\activate`. |
| El IDE no ve los paquetes | El IDE usa otro intérprete. | Seleccione `.venv` como intérprete del proyecto. |
| Tras actualizar Python el entorno falla | Cambio de versión menor del intérprete base. | Recrear el entorno. |
| `pip` instala en el sistema aun con el entorno activo | Alias de shell o `pip` de otra ruta. | Use `python -m pip` y revise `which pip`. |

---

## 16. Buenas prácticas

- Un entorno por proyecto, en la raíz y con el nombre **`.venv`**.
- No versione el entorno: añada `.venv/` a `.gitignore`.
- Versione la lista de dependencias y trate el entorno como desechable.
- Cree siempre con `--upgrade-deps` para tener un pip actualizado.
- En scripts, servicios y tareas programadas, use la ruta del intérprete del entorno en lugar de activar.
- No mueva ni copie entornos: recréelos.
- Recree el entorno al cambiar de versión menor de Python.
- Configure pip con `require-virtualenv = true` para evitar instalaciones accidentales fuera del entorno.
- Si la velocidad de creación o instalación es un problema, use `uv venv` y `uv pip`, compatibles con el mismo formato.

---

## 17. Referencia rápida

```bash
python -m venv .venv [--prompt nombre] [--upgrade-deps] [--clear] [--system-site-packages]
source .venv/bin/activate            # Linux/macOS
.venv\Scripts\Activate.ps1           # Windows PowerShell
deactivate
.venv/bin/python -m pip install -r requirements.txt     # Sin activar
python -c "import sys; print(sys.prefix != sys.base_prefix)"    # ¿Estoy en un entorno?
rm -rf .venv && python -m venv .venv                    # Recrear
```

```python
import venv
venv.EnvBuilder(with_pip=True, upgrade_deps=True, clear=True, prompt="demo").create(".venv")
```

---

## 18. Ejercicios propuestos

1. Cree dos proyectos con entornos distintos e instale versiones diferentes de `requests` en cada uno. Compruebe que no interfieren.
2. Inspeccione `pyvenv.cfg` y la carpeta `bin/` (o `Scripts/`). Explique qué ocurre si mueve el entorno a otra ruta.
3. Escriba un script que se niegue a ejecutarse fuera de un entorno virtual.
4. Configure un servicio o tarea programada que use el intérprete del entorno sin activarlo.
5. Amplíe `bootstrap.py` para aceptar `--python 3.12` y relanzarse con esa versión usando `py` (Windows) o `python3.12` (Linux/macOS).

---

## 19. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/venv.html
- Tutorial de entornos virtuales: https://docs.python.org/es/3/tutorial/venv.html
- Guía de empaquetado (entornos): https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/
- PEP 405 (entornos virtuales): https://peps.python.org/pep-0405/
