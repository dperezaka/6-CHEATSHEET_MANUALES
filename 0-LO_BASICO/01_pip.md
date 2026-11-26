# Manual de uso: pip

> **Categoría:** Instalación de paquetes · **Utilidad:** Instalar paquetes de PyPI
> **Tipo:** Herramienta incluida con Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y comprobación](#2-instalación-y-comprobación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Instalar paquetes](#4-instalar-paquetes)
5. [Especificadores de versión](#5-especificadores-de-versión)
6. [Actualizar y desinstalar](#6-actualizar-y-desinstalar)
7. [Consultar lo instalado](#7-consultar-lo-instalado)
8. [Archivos de requisitos](#8-archivos-de-requisitos)
9. [Restricciones y bloqueo de versiones](#9-restricciones-y-bloqueo-de-versiones)
10. [Instalación desde otras fuentes](#10-instalación-desde-otras-fuentes)
11. [Índices privados y configuración](#11-índices-privados-y-configuración)
12. [Instalación segura y reproducible](#12-instalación-segura-y-reproducible)
13. [Caché y entornos sin conexión](#13-caché-y-entornos-sin-conexión)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

**pip** es el instalador oficial de paquetes de Python. Descarga paquetes del **Python Package Index (PyPI)** u otros orígenes, resuelve sus dependencias y los instala en el entorno de Python activo.

**Ventajas:**
- Viene incluido con Python 3.4 y versiones posteriores.
- Es el estándar de facto: toda la documentación del ecosistema lo usa como referencia.
- Admite archivos de requisitos, índices privados, repositorios Git y rutas locales.
- Resolutor de dependencias estricto: detecta conflictos antes de instalar.

**Limitaciones:**
- No crea entornos virtuales (use **venv**) ni gestiona versiones de Python.
- No genera archivos de bloqueo nativos con hashes (se complementa con `pip-tools` o se sustituye por **uv**/**Poetry**).
- Más lento que alternativas escritas en Rust, como **uv**.

| Si necesita… | Opción |
|---|---|
| Instalar paquetes en un entorno existente | **pip** |
| Aislar las dependencias de cada proyecto | venv + pip |
| Instalar herramientas de terminal globales | pipx / `uv tool` |
| Archivo de bloqueo y gestión del proyecto completo | Poetry / uv |
| Máxima velocidad con la misma interfaz | `uv pip` |

---

## 2. Instalación y comprobación

pip se instala con Python. Compruebe la versión:

```bash
python -m pip --version
# pip 25.x from /ruta/al/entorno/lib/python3.13/site-packages/pip (python 3.13)
```

> Utilice siempre **`python -m pip`** en lugar de `pip` a secas: garantiza que se usa el pip del intérprete que espera, sobre todo cuando hay varias versiones de Python instaladas.

Si pip no está disponible:

```bash
python -m ensurepip --upgrade
```

En Linux, algunas distribuciones lo distribuyen aparte:

```bash
sudo apt install python3-pip        # Debian / Ubuntu
sudo dnf install python3-pip        # Fedora
```

Actualizar pip:

```bash
python -m pip install --upgrade pip
```

En Windows, con el lanzador `py`:

```powershell
py -3.13 -m pip install requests
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| PyPI | Repositorio público de paquetes (https://pypi.org). |
| Paquete de distribución | Archivo que se descarga e instala: *wheel* (`.whl`) o *sdist* (`.tar.gz`). |
| Wheel | Formato precompilado: se instala rápido, sin compilar. |
| sdist | Código fuente: puede requerir compilar extensiones (C, Rust…). |
| Dependencia | Paquete que otro paquete necesita para funcionar. |
| Especificador de versión | Regla que limita las versiones aceptables (`>=2.0,<3`). |
| Entorno | Intérprete de Python con su propio `site-packages`. |
| Archivo de requisitos | Lista de paquetes a instalar (`requirements.txt`). |
| Extra | Dependencias opcionales de un paquete (`paquete[extra]`). |

> **Regla de oro:** instale siempre dentro de un **entorno virtual**, nunca en el Python del sistema.

---

## 4. Instalar paquetes

```bash
python -m pip install requests                 # Última versión compatible
python -m pip install requests==2.32.3         # Versión exacta
python -m pip install "requests>=2.31,<3"      # Rango (use comillas en la terminal)
python -m pip install requests httpx rich      # Varios paquetes
python -m pip install "fastapi[standard]"      # Con extras
python -m pip install --pre paquete            # Incluir versiones preliminares
python -m pip install --user paquete           # En el directorio del usuario (evítelo; use venv)
```

Opciones útiles:

| Opción | Efecto |
|---|---|
| `-U`, `--upgrade` | Actualiza si ya está instalado. |
| `--no-deps` | No instala las dependencias. |
| `--force-reinstall` | Reinstala aunque ya esté. |
| `--only-binary :all:` | Solo acepta *wheels* (evita compilar). |
| `--no-binary paquete` | Fuerza la compilación desde el código fuente. |
| `-q` / `-v` | Menos / más información en pantalla. |
| `--dry-run` | Muestra lo que haría sin instalar nada. |
| `--report informe.json` | Genera un informe JSON de lo que se instalaría. |

---

## 5. Especificadores de versión

| Especificador | Significado | Ejemplo |
|---|---|---|
| `==` | Versión exacta | `django==5.1.4` |
| `!=` | Excluir una versión | `numpy!=2.0.0` |
| `>=`, `<=`, `>`, `<` | Comparaciones | `pandas>=2.2,<3` |
| `~=` | Compatible (mismo nivel superior) | `~=2.31` equivale a `>=2.31,<3` |
| `==X.*` | Comodín | `flask==3.*` |
| `;` marcador | Condición de entorno | `pywin32; sys_platform == "win32"` |

Marcadores de entorno habituales:

```text
tomli>=2.0 ; python_version < "3.11"
uvloop ; sys_platform != "win32"
colorama ; os_name == "nt"
```

> Para **aplicaciones**, fije versiones exactas (`==`) en un archivo bloqueado. Para **librerías**, declare rangos amplios para no provocar conflictos a sus usuarios.

---

## 6. Actualizar y desinstalar

```bash
python -m pip install -U requests                    # Actualizar un paquete
python -m pip list --outdated                        # Ver paquetes desactualizados
python -m pip uninstall requests                     # Desinstalar (pide confirmación)
python -m pip uninstall -y requests httpx            # Sin confirmación
python -m pip uninstall -y -r requirements.txt       # Desinstalar una lista
```

> `pip uninstall` **no** elimina las dependencias que se instalaron junto al paquete. Para un entorno limpio, lo más fiable es recrear el entorno virtual.

Estrategia de actualización:

```bash
python -m pip install -U --upgrade-strategy only-if-needed paquete   # Por defecto
python -m pip install -U --upgrade-strategy eager paquete            # Actualiza también dependencias
```

---

## 7. Consultar lo instalado

```bash
python -m pip list                       # Paquetes instalados
python -m pip list --format=json         # En JSON (para scripts)
python -m pip list --not-required        # Solo los de nivel superior
python -m pip show requests              # Detalles: versión, ubicación, dependencias
python -m pip show -f requests           # Incluye los archivos instalados
python -m pip freeze                     # Formato requirements (paquete==versión)
python -m pip check                      # Verifica dependencias incompatibles
python -m pip inspect > entorno.json     # Informe completo del entorno
python -m pip index versions requests    # Versiones disponibles en el índice
```

Salida de `pip show`:

```text
Name: requests
Version: 2.32.3
Summary: Python HTTP for Humans.
Location: /proyecto/.venv/lib/python3.13/site-packages
Requires: certifi, charset-normalizer, idna, urllib3
Required-by: httpie
```

---

## 8. Archivos de requisitos

### 8.1 Formato

```text
# requirements.txt
requests==2.32.3
pydantic>=2.9,<3
rich~=13.9
tomli ; python_version < "3.11"

# Incluir otro archivo
-r requirements-base.txt

# Restricciones
-c constraints.txt

# Índice alternativo
--index-url https://pypi.org/simple
--extra-index-url https://pypi.empresa.local/simple
```

### 8.2 Uso

```bash
python -m pip install -r requirements.txt
python -m pip freeze > requirements.txt      # Volcar el entorno actual
```

### 8.3 Separar por propósito

```
requirements/
├── base.txt          # Dependencias de producción
├── dev.txt           # -r base.txt + pytest, ruff, mypy…
└── docs.txt          # -r base.txt + mkdocs…
```

```text
# requirements/dev.txt
-r base.txt
pytest==8.3.4
ruff==0.8.4
mypy==1.13.0
```

> `pip freeze` incluye **todo** lo instalado, incluidas las dependencias transitivas y paquetes instalados a mano. Para separar dependencias directas de las bloqueadas, use `pip-tools` (sección 9) o declare las dependencias en `pyproject.toml`.

---

## 9. Restricciones y bloqueo de versiones

### 9.1 Archivos de restricciones (`-c`)

Limitan versiones **sin** pedir que se instale el paquete:

```text
# constraints.txt
urllib3<2.3
numpy==2.1.3
```

```bash
python -m pip install -r requirements.txt -c constraints.txt
```

### 9.2 Bloqueo con `pip-tools`

`pip-tools` separa las dependencias declaradas (`.in`) de las versiones exactas resueltas (`.txt`):

```bash
python -m pip install pip-tools
```

```text
# requirements.in
fastapi
sqlalchemy>=2
```

```bash
pip-compile requirements.in --generate-hashes -o requirements.txt
pip-sync requirements.txt            # Deja el entorno exactamente igual al archivo
pip-compile --upgrade-package fastapi requirements.in   # Actualizar uno
```

### 9.3 Archivo de bloqueo estándar (`pylock.toml`)

El estándar **PEP 751** define `pylock.toml` como formato común de bloqueo. Versiones recientes de pip pueden generarlo de forma experimental:

```bash
python -m pip lock -r requirements.txt -o pylock.toml
```

> Si necesita bloqueo reproducible como parte del flujo normal, valore **uv** o **Poetry**, que lo integran de forma nativa.

---

## 10. Instalación desde otras fuentes

```bash
# Proyecto local (lee pyproject.toml)
python -m pip install .
python -m pip install -e .                   # Modo editable (desarrollo)
python -m pip install -e ".[dev,test]"       # Editable con extras

# Repositorio Git
python -m pip install "git+https://github.com/org/repo.git"
python -m pip install "git+https://github.com/org/repo.git@v1.2.0"          # Etiqueta
python -m pip install "git+https://github.com/org/repo.git@main#subdirectory=paquete"

# Archivos
python -m pip install ./dist/mi_paquete-1.0.0-py3-none-any.whl
python -m pip install https://ejemplo.com/paquete-1.0.tar.gz

# Referencia directa en requirements (PEP 508)
mi-paquete @ git+https://github.com/org/repo.git@v1.2.0
```

> El **modo editable** (`-e`) enlaza el código fuente: los cambios se reflejan sin reinstalar. Es el modo habitual al desarrollar un paquete.

---

## 11. Índices privados y configuración

### 11.1 Índices privados

```bash
python -m pip install --index-url https://pypi.empresa.local/simple paquete-interno
python -m pip install --extra-index-url https://pypi.empresa.local/simple paquete-interno
```

> ⚠ **Riesgo de confusión de dependencias:** con `--extra-index-url`, pip busca en **todos** los índices y elige la versión más alta. Un atacante puede publicar en PyPI un paquete con el mismo nombre que uno interno y una versión mayor. Para paquetes internos, use `--index-url` con un proxy que controle (Artifactory, Nexus, devpi) o reserve los nombres en PyPI.

### 11.2 Archivo de configuración

| Sistema | Ubicación (usuario) |
|---|---|
| Linux | `~/.config/pip/pip.conf` |
| macOS | `~/Library/Application Support/pip/pip.conf` o `~/.config/pip/pip.conf` |
| Windows | `%APPDATA%\pip\pip.ini` |
| Entorno virtual | `<venv>/pip.conf` (o `pip.ini` en Windows) |

```ini
[global]
index-url = https://pypi.empresa.local/simple
timeout = 30
disable-pip-version-check = true

[install]
require-virtualenv = true
```

```bash
python -m pip config list                                # Ver la configuración efectiva
python -m pip config set global.require-virtualenv true  # Modificar
python -m pip config debug                               # Ver qué archivos se leen
```

### 11.3 Variables de entorno

Cualquier opción se puede definir como `PIP_<OPCIÓN>`:

```bash
export PIP_REQUIRE_VIRTUALENV=true
export PIP_INDEX_URL=https://pypi.empresa.local/simple
export PIP_NO_CACHE_DIR=1
```

### 11.4 Credenciales

No escriba contraseñas en `requirements.txt` ni en la configuración versionada. Use un archivo `~/.netrc` o `keyring`:

```text
# ~/.netrc
machine pypi.empresa.local
login usuario
password TOKEN_SECRETO
```

---

## 12. Instalación segura y reproducible

### 12.1 Exigir entorno virtual

```bash
python -m pip config set global.require-virtualenv true
```

Así pip rechaza instalar fuera de un entorno virtual.

### 12.2 Python «gestionado externamente» (PEP 668)

En distribuciones Linux modernas y en Homebrew, pip rechaza instalar en el Python del sistema:

```text
error: externally-managed-environment
```

**Solución correcta:** cree un entorno virtual o use pipx para herramientas. No use `--break-system-packages` salvo en contenedores desechables.

### 12.3 Verificación de hashes

```text
# requirements.txt con hashes (generado por pip-compile --generate-hashes)
requests==2.32.3 \
    --hash=sha256:70761cfe03c773ceb22aa2f671b4757976145175cdfca038c02654d061d6dcc6
```

```bash
python -m pip install --require-hashes -r requirements.txt
```

Si cualquier archivo descargado no coincide con su hash, la instalación falla.

### 12.4 Comprobar vulnerabilidades

```bash
python -m pip install pip-audit
pip-audit -r requirements.txt
```

---

## 13. Caché y entornos sin conexión

```bash
python -m pip cache dir                  # Ubicación de la caché
python -m pip cache info
python -m pip cache list
python -m pip cache purge                # Vaciar
python -m pip install --no-cache-dir x   # Instalar sin usar la caché (útil en Docker)
```

Instalación sin conexión:

```bash
# En una máquina con Internet
python -m pip download -r requirements.txt -d ./paquetes

# En la máquina sin Internet
python -m pip install --no-index --find-links ./paquetes -r requirements.txt
```

Generar *wheels* propias:

```bash
python -m pip wheel -r requirements.txt -w ./wheels
```

---

## 14. Ejemplo práctico completo

**Preparar un proyecto reproducible** con dependencias separadas para producción y desarrollo, bloqueadas con hashes y verificadas.

Estructura:

```
inventario/
├── requirements/
│   ├── base.in
│   ├── base.txt          # Generado
│   ├── dev.in
│   └── dev.txt           # Generado
├── src/inventario/
│   └── __init__.py
├── pyproject.toml
└── Makefile
```

Dependencias declaradas:

```text
# requirements/base.in
httpx>=0.27
pydantic>=2.9
rich>=13
```

```text
# requirements/dev.in
-c base.txt
pytest>=8
ruff
mypy
pip-audit
```

Automatización con `Makefile`:

```makefile
PY := .venv/bin/python

.PHONY: venv lock install dev audit clean

venv:
	python3 -m venv .venv
	$(PY) -m pip install -U pip pip-tools

lock:
	$(PY) -m piptools compile --generate-hashes --strip-extras \
	    requirements/base.in -o requirements/base.txt
	$(PY) -m piptools compile --generate-hashes --strip-extras \
	    requirements/dev.in -o requirements/dev.txt

install:
	$(PY) -m pip install --require-hashes -r requirements/base.txt
	$(PY) -m pip install --no-deps -e .

dev:
	$(PY) -m piptools sync requirements/base.txt requirements/dev.txt
	$(PY) -m pip install --no-deps -e .

audit:
	$(PY) -m pip_audit -r requirements/base.txt --require-hashes

clean:
	rm -rf .venv
```

Flujo de trabajo:

```bash
make venv          # Crea el entorno e instala pip-tools
make lock          # Resuelve y bloquea versiones con hashes
make dev           # Sincroniza el entorno de desarrollo
make audit         # Comprueba vulnerabilidades conocidas
python -m pip check
```

Script de verificación del entorno (`scripts/verificar_entorno.py`):

```python
"""Comprueba que el entorno coincide con requirements/base.txt."""
import re
import sys
from importlib.metadata import PackageNotFoundError, version
from pathlib import Path

PATRON = re.compile(r"^([A-Za-z0-9_.\-]+)==([^\s;\\]+)")


def leer_bloqueo(ruta: Path) -> dict[str, str]:
    esperados: dict[str, str] = {}
    for linea in ruta.read_text(encoding="utf-8").splitlines():
        if m := PATRON.match(linea.strip()):
            esperados[m.group(1).lower()] = m.group(2)
    return esperados


def main() -> int:
    if sys.prefix == sys.base_prefix:
        print("✖ No está dentro de un entorno virtual.")
        return 2
    errores = 0
    for paquete, esperado in leer_bloqueo(Path("requirements/base.txt")).items():
        try:
            instalado = version(paquete)
        except PackageNotFoundError:
            print(f"✖ {paquete}: no instalado (esperado {esperado})")
            errores += 1
            continue
        if instalado != esperado:
            print(f"✖ {paquete}: {instalado} ≠ {esperado}")
            errores += 1
    print("✔ Entorno correcto" if not errores else f"{errores} discrepancias")
    return 1 if errores else 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `pip: command not found` | pip no está en el `PATH` o no está instalado. | Use `python -m pip` o `python -m ensurepip`. |
| `error: externally-managed-environment` | Python del sistema protegido (PEP 668). | Cree un entorno virtual o use pipx. |
| `ERROR: Could not find a version that satisfies the requirement` | Nombre erróneo, versión inexistente o incompatible con su Python/plataforma. | Revise el nombre y `pip index versions paquete`. |
| `ResolutionImpossible` | Dependencias con versiones incompatibles entre sí. | Relaje restricciones o use `-c` para fijar una versión común. |
| `Failed building wheel for …` | Falta compilador o cabeceras del sistema. | Instale una versión con *wheel* o las herramientas de compilación. |
| `Permission denied` | Intento de instalar en el Python del sistema. | Use un entorno virtual; nunca `sudo pip`. |
| El paquete instalado no se encuentra al importar | Se instaló en otro intérprete. | Use `python -m pip` con el mismo `python` que ejecuta el código. |
| `THESE PACKAGES DO NOT MATCH THE HASHES` | Archivo alterado o hash desactualizado. | Regenere el bloqueo; nunca desactive la verificación sin investigar. |
| `SSL: CERTIFICATE_VERIFY_FAILED` | Proxy corporativo o certificados del sistema. | Configure `cert` con el certificado de la empresa; evite `--trusted-host`. |
| Instalación muy lenta | Compilación desde sdist, red lenta. | Use *wheels*, caché local o `uv pip`. |

---

## 16. Buenas prácticas

- Use siempre `python -m pip` en lugar de `pip`.
- Trabaje **siempre** dentro de un entorno virtual; active `require-virtualenv`.
- Nunca use `sudo pip install`.
- Declare las dependencias directas en `pyproject.toml` o `.in` y bloquee las versiones exactas en un archivo aparte.
- En producción, instale con `--require-hashes` desde un archivo bloqueado.
- Separe las dependencias de producción y de desarrollo.
- Revise periódicamente `pip list --outdated` y `pip-audit`.
- No guarde credenciales en archivos versionados.
- Para índices privados, prefiera `--index-url` con un proxy controlado a `--extra-index-url`.
- Para proyectos nuevos, valore **uv**, que ofrece la misma interfaz (`uv pip`) con bloqueo y mucha más velocidad.

---

## 17. Referencia rápida

```bash
python -m pip install paquete / "paquete>=1,<2" / "paquete[extra]" / -e . / -r requirements.txt
python -m pip install -U paquete                      # Actualizar
python -m pip uninstall -y paquete
python -m pip list [--outdated | --not-required]
python -m pip show paquete
python -m pip freeze > requirements.txt
python -m pip check
python -m pip download -r req.txt -d dir  ·  install --no-index --find-links dir
python -m pip install --require-hashes -r requirements.txt
python -m pip config list / set clave valor
python -m pip cache purge
```

---

## 18. Ejercicios propuestos

1. Cree un entorno virtual, instale `requests` y `rich`, y genere un `requirements.txt` con `pip freeze`. Explique qué paquetes aparecen y por qué.
2. Escriba un `requirements.txt` con un marcador de entorno que instale `tomli` solo en Python < 3.11.
3. Use `pip-tools` para bloquear dependencias con hashes y compruebe que `--require-hashes` falla si modifica un hash.
4. Prepare una instalación sin conexión: descargue los paquetes en una carpeta e instálelos en otro entorno con `--no-index`.
5. Configure pip para exigir entorno virtual y verifique que rechaza instalar fuera de él.

---

## 19. Referencias

- Documentación oficial: https://pip.pypa.io/
- Especificadores de versión: https://packaging.python.org/en/latest/specifications/version-specifiers/
- Guía de instalación de paquetes: https://packaging.python.org/es/latest/tutorials/installing-packages/
- pip-tools: https://pip-tools.readthedocs.io/
- PEP 668 (entornos gestionados externamente): https://peps.python.org/pep-0668/
- PEP 751 (`pylock.toml`): https://peps.python.org/pep-0751/
