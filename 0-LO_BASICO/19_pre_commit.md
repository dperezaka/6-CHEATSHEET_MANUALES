# Manual de uso: pre-commit

> **Categoría:** Automatización · **Utilidad:** Ejecuta las comprobaciones antes de cada *commit*
> **Tipo:** Herramienta externa (requiere instalación) · Gestor de *hooks* de Git multilenguaje

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos](#4-primeros-pasos)
5. [El archivo `.pre-commit-config.yaml`](#5-el-archivo-pre-commit-configyaml)
6. [Opciones de los *hooks*](#6-opciones-de-los-hooks)
7. [*Hooks* recomendados para Python](#7-hooks-recomendados-para-python)
8. [*Hooks* locales](#8-hooks-locales)
9. [Etapas: `commit-msg`, `pre-push` y otras](#9-etapas-commit-msg-pre-push-y-otras)
10. [Ejecutar, omitir y actualizar](#10-ejecutar-omitir-y-actualizar)
11. [Crear y publicar un *hook* propio](#11-crear-y-publicar-un-hook-propio)
12. [pre-commit en CI](#12-pre-commit-en-ci)
13. [Rendimiento y caché](#13-rendimiento-y-caché)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

**pre-commit** gestiona *hooks* de Git: programas que se ejecutan automáticamente en ciertos momentos (antes de un *commit*, al escribir el mensaje, antes de un *push*). Con un único archivo de configuración versionado, todo el equipo ejecuta las mismas comprobaciones —formato, *linters*, tipos, seguridad, archivos grandes, secretos— **antes** de que el código llegue al repositorio.

```text
$ git commit -m "Añade exportación a CSV"
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
ruff check...............................................................Failed
- hook id: ruff-check
- files were modified by this hook

Fixed 2 errors:
- src/tienda/exportar.py:
    1 × F401 (unused-import)
    1 × I001 (unsorted-imports)

ruff format..............................................................Passed
```

**Ventajas:**
- Configuración única y versionada: las mismas reglas para todos.
- Cada *hook* se instala en un **entorno aislado**, con su versión fijada.
- Solo analiza los **archivos modificados**: rápido.
- Multilenguaje: Python, Node, Rust, Go, Docker, scripts del sistema.
- Gran catálogo de *hooks* listos para usar.

**Limitaciones:**
- Los *hooks* locales se pueden saltar (`--no-verify`): **la CI debe repetir las comprobaciones**.
- La primera ejecución descarga e instala los entornos (después se reutilizan).
- Las comprobaciones lentas (pruebas completas) no son adecuadas para cada *commit*.

---

## 2. Instalación

```bash
uv tool install pre-commit          # Recomendado (global, aislado)
pipx install pre-commit
uv add --dev pre-commit             # Como dependencia de desarrollo del proyecto
brew install pre-commit
```

Comprobar:

```bash
pre-commit --version
```

> **prek** es una reimplementación en Rust, compatible con la misma configuración y bastante más rápida: `uv tool install prek` y use `prek` en lugar de `pre-commit`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Hook* de Git | Script que Git ejecuta en un evento (`pre-commit`, `commit-msg`, `pre-push`…). |
| Repositorio de *hooks* | Repositorio Git que define *hooks* en `.pre-commit-hooks.yaml`. |
| `rev` | Versión (etiqueta o *commit*) del repositorio de *hooks* que se usa. |
| `id` | Identificador del *hook* dentro de su repositorio. |
| Etapa (*stage*) | Momento en que se ejecuta el *hook*. |
| Archivos preparados | Los archivos en el área de preparación (`git add`); son los que se analizan. |
| Entorno del *hook* | Entorno aislado donde se instala cada *hook* (en `~/.cache/pre-commit`). |
| *Hook* local | Definido en el propio proyecto (`repo: local`). |

**Comportamiento clave:** si un *hook* **modifica** archivos (p. ej. el formateador), el *commit* se detiene. Revise los cambios, añádalos con `git add` y repita el *commit*.

---

## 4. Primeros pasos

1. Cree `.pre-commit-config.yaml` en la raíz del repositorio:

   ```yaml
   repos:
     - repo: https://github.com/pre-commit/pre-commit-hooks
       rev: v6.0.0
       hooks:
         - id: trailing-whitespace
         - id: end-of-file-fixer
         - id: check-yaml
         - id: check-added-large-files

     - repo: https://github.com/astral-sh/ruff-pre-commit
       rev: v0.16.9
       hooks:
         - id: ruff-check
           args: [--fix]
         - id: ruff-format
   ```

2. Instale los *hooks* en el repositorio local:

   ```bash
   pre-commit install
   ```

3. Ejecútelos sobre todo el código la primera vez:

   ```bash
   pre-commit run --all-files
   ```

4. A partir de aquí, se ejecutan solos en cada `git commit`.

Generar una configuración de ejemplo:

```bash
pre-commit sample-config > .pre-commit-config.yaml
```

> Cada miembro del equipo debe ejecutar `pre-commit install` una vez tras clonar. Puede automatizarlo en el script de arranque del proyecto o en `git config --global init.templateDir` (sección 10).

---

## 5. El archivo `.pre-commit-config.yaml`

```yaml
# Opciones globales
minimum_pre_commit_version: "4.0.0"
default_language_version:
  python: python3.13
default_install_hook_types: [pre-commit, commit-msg, pre-push]
default_stages: [pre-commit]
fail_fast: false                       # true: detenerse en el primer fallo
exclude: ^(docs/_build/|src/tienda/generado/)

repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: check-toml
      - id: check-json
        exclude: ^\.vscode/            # Exclusión específica

  - repo: local                        # Hooks definidos en el proyecto
    hooks:
      - id: mi-comprobacion
        name: Mi comprobación
        entry: python scripts/comprobar.py
        language: system
        types: [python]

  - repo: meta                         # Hooks de autocomprobación de pre-commit
    hooks:
      - id: check-hooks-apply
      - id: check-useless-excludes
```

| Opción global | Uso |
|---|---|
| `minimum_pre_commit_version` | Versión mínima de pre-commit requerida. |
| `default_language_version` | Versión de Python (u otros lenguajes) para los *hooks*. |
| `default_install_hook_types` | Tipos de *hook* que instala `pre-commit install`. |
| `default_stages` | Etapas por defecto de los *hooks*. |
| `files` / `exclude` | Expresiones regulares (Python) de archivos incluidos/excluidos globalmente. |
| `fail_fast` | Detener tras el primer *hook* que falle. |

Validar la configuración:

```bash
pre-commit validate-config
pre-commit validate-manifest .pre-commit-hooks.yaml     # Para repositorios de hooks
```

---

## 6. Opciones de los *hooks*

```yaml
- id: mypy
  name: Comprobación de tipos         # Nombre mostrado
  args: [--strict]                    # Argumentos adicionales
  files: ^src/                        # Solo archivos que coincidan (regex)
  exclude: ^src/legado/
  types: [python]                     # Por tipo de archivo (todos deben cumplirse)
  types_or: [python, pyi]             # Al menos uno
  exclude_types: [markdown]
  stages: [pre-push]                  # En qué etapa
  additional_dependencies: ["pydantic>=2.9", "types-requests"]
  pass_filenames: false               # No pasar la lista de archivos
  always_run: true                    # Ejecutar aunque no haya archivos coincidentes
  require_serial: true                # No dividir en lotes paralelos
  verbose: true                       # Mostrar la salida aunque pase
  log_file: .pre-commit-mypy.log
  alias: tipos                        # Nombre alternativo para «pre-commit run tipos»
```

Tipos de archivo frecuentes (`types`): `python`, `pyi`, `jupyter`, `yaml`, `toml`, `json`, `markdown`, `shell`, `dockerfile`, `text`, `binary`, `executable`, `file`. Consulte los de un archivo con:

```bash
python -m identify.cli src/tienda/pedidos.py
# ["file", "non-executable", "python", "text"]
```

---

## 7. *Hooks* recomendados para Python

| Repositorio | *Hooks* | Función |
|---|---|---|
| `pre-commit/pre-commit-hooks` | `trailing-whitespace`, `end-of-file-fixer`, `mixed-line-ending` | Espacios y finales de línea. |
| | `check-yaml`, `check-toml`, `check-json` | Sintaxis de archivos de configuración. |
| | `check-added-large-files` | Impide subir archivos grandes por error. |
| | `check-merge-conflict` | Marcas de conflicto sin resolver. |
| | `detect-private-key` | Claves privadas en el código. |
| | `debug-statements` | `breakpoint()`, `pdb` olvidados. |
| | `check-case-conflict` | Nombres que chocan en sistemas sin distinción de mayúsculas. |
| | `no-commit-to-branch` | Impide *commits* directos a `main`. |
| `astral-sh/ruff-pre-commit` | `ruff-check`, `ruff-format` | *Lint* y formato. |
| `astral-sh/uv-pre-commit` | `uv-lock`, `uv-export` | Mantener `uv.lock` sincronizado con `pyproject.toml`. |
| `pre-commit/mirrors-mypy` | `mypy` | Tipos (mejor como *hook* local, ver sección 8). |
| `PyCQA/bandit` | `bandit` | Seguridad del código. |
| `gitleaks/gitleaks` | `gitleaks` | Secretos (*tokens*, claves) en los cambios. |
| `abravalheri/validate-pyproject` | `validate-pyproject` | Validez de `pyproject.toml`. |
| `codespell-project/codespell` | `codespell` | Erratas en código y documentación. |
| `kynan/nbstripout` | `nbstripout` | Elimina las salidas de los *notebooks*. |
| `compilerla/conventional-pre-commit` | `conventional-pre-commit` | Formato de mensajes de *commit*. |
| `hadolint/hadolint` | `hadolint` | Buenas prácticas en `Dockerfile`. |
| `rhysd/actionlint` | `actionlint` | Flujos de GitHub Actions. |

---

## 8. *Hooks* locales

### 8.1 Usar las herramientas del entorno del proyecto

Para herramientas que necesitan las dependencias del proyecto (mypy, pytest, pyright), lo más fiable es ejecutarlas con el entorno del propio proyecto:

```yaml
- repo: local
  hooks:
    - id: mypy
      name: mypy
      entry: uv run mypy
      language: system
      types: [python]
      pass_filenames: false            # mypy analiza el proyecto según su configuración
      require_serial: true

    - id: pytest-rapidas
      name: pruebas rápidas
      entry: uv run pytest -q -m "not lento" -x
      language: system
      pass_filenames: false
      stages: [pre-push]
```

| `language` | Significado |
|---|---|
| `system` | Usa el programa del sistema/entorno tal cual (`entry`). |
| `python` | pre-commit crea un entorno Python e instala `additional_dependencies`. |
| `script` | Ejecuta un script del repositorio. |
| `pygrep` | Busca una expresión regular en los archivos (sin escribir código). |
| `fail` | Falla siempre si hay archivos coincidentes (prohibir archivos). |
| `docker_image` | Ejecuta una imagen Docker. |
| `node`, `rust`, `golang`… | Otros lenguajes. |

### 8.2 Comprobaciones sin código con `pygrep` y `fail`

```yaml
- repo: local
  hooks:
    - id: sin-print
      name: no usar print() en src/
      entry: '^\s*print\('
      language: pygrep
      files: ^src/
      types: [python]

    - id: todo-con-ticket
      name: los TODO deben referenciar un ticket (TODO(PROY-123))
      entry: 'TODO(?!\([A-Z]+-\d+\))'
      language: pygrep
      types: [python]

    - id: sin-env
      name: no versionar archivos .env
      entry: Los archivos .env no deben versionarse
      language: fail
      files: (^|/)\.env(\..*)?$
      exclude: \.env\.example$
```

### 8.3 Script propio en Python

```yaml
- repo: local
  hooks:
    - id: cabecera-licencia
      name: cabecera de licencia
      entry: python scripts/hooks/cabecera.py
      language: python                 # Entorno aislado con additional_dependencies
      additional_dependencies: []
      types: [python]
```

El script recibe la lista de archivos como argumentos y debe devolver un código distinto de 0 si falla.

---

## 9. Etapas: `commit-msg`, `pre-push` y otras

| Etapa | Momento | Uso típico |
|---|---|---|
| `pre-commit` | Antes de crear el *commit* | Formato, *lint*, comprobaciones rápidas. |
| `commit-msg` | Tras escribir el mensaje | Formato del mensaje (Conventional Commits). |
| `prepare-commit-msg` | Antes de abrir el editor | Plantilla de mensaje, número de *ticket* desde la rama. |
| `pre-push` | Antes de `git push` | Pruebas rápidas, tipos, auditorías. |
| `post-checkout` / `post-merge` | Tras cambiar de rama o fusionar | Sincronizar dependencias (`uv sync`). |
| `manual` | Solo con `pre-commit run --hook-stage manual` | Comprobaciones lentas a demanda. |

Instalar los tipos de *hook* necesarios:

```bash
pre-commit install --hook-type pre-commit --hook-type commit-msg --hook-type pre-push
```

O, mejor, declárelos en la configuración (`default_install_hook_types`) y ejecute solo `pre-commit install`.

Validar mensajes de *commit*:

```yaml
- repo: https://github.com/compilerla/conventional-pre-commit
  rev: v4.2.0
  hooks:
    - id: conventional-pre-commit
      stages: [commit-msg]
      args: [--strict, feat, fix, docs, refactor, test, chore, ci, perf]
```

```text
$ git commit -m "cambios varios"
Conventional Commit......................................................Failed
[Bad commit message] >> cambios varios
Your commit message does not follow Conventional Commits formatting
```

Sincronizar dependencias al cambiar de rama:

```yaml
- repo: local
  hooks:
    - id: uv-sync
      name: sincronizar dependencias
      entry: uv sync --quiet
      language: system
      pass_filenames: false
      always_run: true
      stages: [post-checkout, post-merge]
```

---

## 10. Ejecutar, omitir y actualizar

```bash
pre-commit run                          # Hooks sobre los archivos preparados
pre-commit run --all-files              # Sobre todo el repositorio
pre-commit run ruff-check               # Un hook concreto
pre-commit run ruff-check --files src/a.py src/b.py
pre-commit run --from-ref origin/main --to-ref HEAD   # Archivos cambiados respecto a main
pre-commit run --hook-stage pre-push --all-files
pre-commit run --show-diff-on-failure   # Muestra los cambios hechos por los hooks
```

Omitir *hooks* de forma puntual:

```bash
SKIP=mypy,pytest-rapidas git commit -m "wip: ..."     # Omitir algunos
git commit --no-verify -m "..."                        # Omitir todos (úselo con criterio)
```

Actualizar las versiones de los *hooks*:

```bash
pre-commit autoupdate                   # Sube «rev» a la última etiqueta de cada repositorio
pre-commit autoupdate --freeze          # Usa el hash del commit (más seguro) con la etiqueta en comentario
pre-commit autoupdate --repo https://github.com/astral-sh/ruff-pre-commit
```

Instalación y limpieza:

```bash
pre-commit install                      # Instalar en .git/hooks
pre-commit install --install-hooks      # Además, preparar ya todos los entornos
pre-commit uninstall
pre-commit clean                        # Vaciar la caché de entornos
pre-commit gc                           # Eliminar entornos que ya no se usan
```

Activar pre-commit automáticamente en todos los repositorios que clone:

```bash
git config --global init.templateDir ~/.git-template
pre-commit init-templatedir ~/.git-template
```

---

## 11. Crear y publicar un *hook* propio

Un repositorio de *hooks* necesita un archivo `.pre-commit-hooks.yaml` y el código del *hook*:

```
hooks-empresa/
├── .pre-commit-hooks.yaml
├── pyproject.toml
└── src/hooks_empresa/
    ├── __init__.py
    └── cabecera.py
```

```yaml
# .pre-commit-hooks.yaml
- id: cabecera-licencia
  name: Cabecera de licencia
  description: Comprueba que los archivos Python tienen la cabecera de licencia de la empresa.
  entry: cabecera-licencia
  language: python
  types: [python]
```

```toml
# pyproject.toml
[project]
name = "hooks-empresa"
version = "1.0.0"
requires-python = ">=3.12"

[project.scripts]
cabecera-licencia = "hooks_empresa.cabecera:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

Probar el *hook* antes de publicarlo:

```bash
cd otro-proyecto
pre-commit try-repo ../hooks-empresa cabecera-licencia --all-files
```

Publicar: suba el repositorio, cree una etiqueta (`git tag v1.0.0 && git push --tags`) y úselo:

```yaml
- repo: https://github.com/empresa/hooks-empresa
  rev: v1.0.0
  hooks:
    - id: cabecera-licencia
```

---

## 12. pre-commit en CI

Los *hooks* locales se pueden omitir, así que **la CI debe ejecutarlos también**.

### 12.1 GitHub Actions

```yaml
name: pre-commit
on: [pull_request, push]

jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
      - uses: actions/cache@v4
        with:
          path: ~/.cache/pre-commit
          key: pre-commit-${{ hashFiles('.pre-commit-config.yaml') }}
      - run: uv sync --locked                       # Para los hooks locales que usan «uv run»
      - run: uvx pre-commit run --all-files --show-diff-on-failure
```

### 12.2 pre-commit.ci

Servicio gratuito para proyectos de código abierto: ejecuta los *hooks* en cada *pull request*, **corrige automáticamente** (añade un *commit* con los arreglos) y abre PR semanales con `autoupdate`.

```yaml
ci:
  autofix_commit_msg: "style: correcciones automáticas de pre-commit"
  autoupdate_schedule: monthly
  autoupdate_commit_msg: "chore: actualizar hooks de pre-commit"
  skip: [mypy, pytest-rapidas]          # Hooks locales que necesitan el entorno del proyecto
```

### 12.3 GitLab CI

```yaml
pre-commit:
  image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim
  variables:
    PRE_COMMIT_HOME: ${CI_PROJECT_DIR}/.cache/pre-commit
  cache:
    key:
      files: [.pre-commit-config.yaml]
    paths: [.cache/pre-commit]
  before_script:
    - apt-get update && apt-get install -y --no-install-recommends git
  script:
    - uvx pre-commit run --all-files --show-diff-on-failure
```

---

## 13. Rendimiento y caché

- Los entornos se guardan en `~/.cache/pre-commit` (variable `PRE_COMMIT_HOME`) y se reutilizan entre proyectos.
- En el día a día solo se analizan los archivos preparados: la mayoría de *hooks* tardan milisegundos.
- Mantenga en `pre-commit` solo comprobaciones **rápidas** (< 2-3 s en total). Mueva lo lento (pruebas, mypy de todo el proyecto, auditorías) a `pre-push`, `manual` o a la CI.
- Use herramientas rápidas (Ruff en lugar de flake8 + isort + black).
- Con **prek** (compatible con la misma configuración) la instalación de entornos y la ejecución son notablemente más rápidas.
- `require_serial: false` (por defecto) permite a pre-commit dividir los archivos en lotes paralelos.

---

## 14. Ejemplo práctico completo

**Configuración completa para un proyecto Python con uv**: higiene de archivos, Ruff, validación de `pyproject.toml`, bloqueo sincronizado, secretos, seguridad, reglas propias con `pygrep`/`fail`, tipos y pruebas rápidas antes del *push*, mensajes de *commit* convencionales, más un *hook* local en Python que impide fijar versiones sin límite superior… y la integración en CI.

### 14.1 `.pre-commit-config.yaml`

```yaml
minimum_pre_commit_version: "4.0.0"
default_install_hook_types: [pre-commit, commit-msg, pre-push]
default_language_version:
  python: python3.13
exclude: ^(src/tienda/generado/|tests/datos/)

repos:
  # ------------------------------------------------------------ Higiene de archivos
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
        args: [--markdown-linebreak-ext=md]
      - id: end-of-file-fixer
      - id: mixed-line-ending
        args: [--fix=lf]
      - id: check-yaml
      - id: check-toml
      - id: check-json
      - id: check-merge-conflict
      - id: check-case-conflict
      - id: check-added-large-files
        args: [--maxkb=500]
      - id: detect-private-key
      - id: debug-statements
      - id: no-commit-to-branch
        args: [--branch, main]

  # ------------------------------------------------------------ Python
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.16.9
    hooks:
      - id: ruff-check
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/abravalheri/validate-pyproject
    rev: v0.24.1
    hooks:
      - id: validate-pyproject

  - repo: https://github.com/astral-sh/uv-pre-commit
    rev: 0.13.10
    hooks:
      - id: uv-lock

  # ------------------------------------------------------------ Seguridad
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.28.0
    hooks:
      - id: gitleaks

  - repo: https://github.com/PyCQA/bandit
    rev: 1.9.4
    hooks:
      - id: bandit
        args: [-c, pyproject.toml, -ll, -ii, -q]
        additional_dependencies: ["bandit[toml]"]
        files: ^src/

  # ------------------------------------------------------------ Reglas del proyecto
  - repo: local
    hooks:
      - id: sin-print
        name: no usar print() en src/ (use logging)
        entry: '^\s*print\('
        language: pygrep
        files: ^src/
        types: [python]

      - id: todo-con-ticket
        name: TODO con ticket, p. ej. TODO(TIE-123)
        entry: '#\s*TODO(?!\([A-Z]+-\d+\))'
        language: pygrep
        types: [python]

      - id: sin-env
        name: no versionar archivos .env
        entry: "Los archivos .env no se versionan; use .env.example"
        language: fail
        files: (^|/)\.env(\.[^/]*)?$
        exclude: \.env\.example$

      - id: dependencias-acotadas
        name: dependencias con versión mínima
        entry: python scripts/hooks/dependencias_acotadas.py
        language: python
        files: ^pyproject\.toml$

      # -------------------------------------------------------- Antes del push
      - id: mypy
        name: mypy (proyecto completo)
        entry: uv run mypy
        language: system
        types: [python]
        pass_filenames: false
        require_serial: true
        stages: [pre-push]

      - id: pytest-rapidas
        name: pruebas rápidas
        entry: uv run pytest -q -x -m "not lento" --no-cov
        language: system
        pass_filenames: false
        always_run: true
        stages: [pre-push]

  # ------------------------------------------------------------ Mensajes de commit
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v4.2.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: [--strict, feat, fix, docs, refactor, test, chore, ci, perf, build]

  # ------------------------------------------------------------ Autocomprobación
  - repo: meta
    hooks:
      - id: check-hooks-apply
      - id: check-useless-excludes

ci:
  skip: [mypy, pytest-rapidas, uv-lock]
```

### 14.2 *Hook* local (`scripts/hooks/dependencias_acotadas.py`)

```python
"""
Comprueba que todas las dependencias de [project] declaran una versión mínima (>= o ==)
y que no se usan especificadores sin versión ni referencias a ramas de Git.

Uso como hook de pre-commit: recibe la ruta de pyproject.toml como argumento.
"""
from __future__ import annotations

import re
import sys
import tomllib
from pathlib import Path

NOMBRE = re.compile(r"^\s*([A-Za-z0-9][A-Za-z0-9._-]*)(\[[^\]]*\])?\s*(.*)$")
RAMA_GIT = re.compile(r"git\+[^@]+(@(main|master|develop|HEAD))?$")


def problemas_de(dependencia: str) -> str | None:
    requisito = dependencia.split(";", 1)[0].strip()
    if " @ " in requisito:
        url = requisito.split(" @ ", 1)[1]
        if RAMA_GIT.search(url):
            return "referencia Git sin etiqueta ni commit fijo"
        return None
    m = NOMBRE.match(requisito)
    if not m:
        return "formato no reconocido"
    especificador = m.group(3).strip().strip("()")
    if not especificador:
        return "sin versión mínima"
    if not any(op in especificador for op in (">=", "==", "~=", ">")):
        return f"sin límite inferior ({especificador})"
    return None


def revisar(ruta: Path) -> list[str]:
    datos = tomllib.loads(ruta.read_text(encoding="utf-8"))
    proyecto = datos.get("project", {})
    todas = list(proyecto.get("dependencies", []))
    for extra, deps in proyecto.get("optional-dependencies", {}).items():
        todas += deps
    errores = []
    for dep in todas:
        if motivo := problemas_de(dep):
            errores.append(f"{ruta}: «{dep}» → {motivo}")
    return errores


def main(argv: list[str]) -> int:
    errores = [e for ruta in argv for e in revisar(Path(ruta))]
    for e in errores:
        print(e)
    if errores:
        print("\nDeclare una versión mínima, p. ej.: \"httpx>=0.27\".")
    return 1 if errores else 0


if __name__ == "__main__":
    raise SystemExit(main(sys.argv[1:]))
```

### 14.3 Puesta en marcha y uso diario

```bash
uv tool install pre-commit
pre-commit install                        # Instala pre-commit, commit-msg y pre-push
pre-commit run --all-files                # Primera pasada sobre todo el código
git add -A && git commit -m "style: aplicar pre-commit a todo el proyecto"
git rev-parse HEAD >> .git-blame-ignore-revs
```

Ejemplo de *commit* que se bloquea:

```text
$ git commit -m "añade cálculo de envío"
trim trailing whitespace.................................................Passed
...
ruff check...............................................................Passed
ruff format..............................................................Passed
no usar print() en src/ (use logging)....................................Failed
- hook id: sin-print

src/tienda/envio.py:14:    print("coste", coste)

dependencias con versión mínima..........................................Failed
- hook id: dependencias-acotadas
- exit code: 1

pyproject.toml: «rich» → sin versión mínima

Declare una versión mínima, p. ej.: "httpx>=0.27".

Conventional Commit......................................................Failed
[Bad commit message] >> añade cálculo de envío
```

Tras corregir:

```bash
git commit -m "feat(envio): añade cálculo del coste de envío"
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Los *hooks* no se ejecutan al hacer *commit* | No se ejecutó `pre-commit install` en ese clon. | `pre-commit install` (o plantilla global de Git). |
| `files were modified by this hook` y el *commit* se cancela | Un *hook* corrigió archivos. | Revise, `git add` y repita el *commit*. Es el comportamiento esperado. |
| `An unexpected error has occurred: CalledProcessError` al instalar un *hook* | Fallo al crear el entorno (red, versión de Python inexistente). | Revise `default_language_version`; `pre-commit clean` y reintente. |
| `Executable 'uv' not found` en un *hook* `system` | La herramienta no está en el `PATH` del proceso de Git (p. ej. desde un IDE). | Use la ruta completa o asegúrese de que el IDE hereda el `PATH`. |
| mypy en pre-commit no encuentra las dependencias | El *hook* `mirrors-mypy` usa un entorno aislado. | Añada `additional_dependencies` o use un *hook* local con `uv run mypy`. |
| Resultados distintos entre pre-commit y la CI | Versiones diferentes de la herramienta. | Use la misma versión en `rev` que en las dependencias del proyecto. |
| `check-useless-excludes` falla | Un `exclude` no coincide con ningún archivo. | Corrija o elimine la exclusión. |
| *Hooks* muy lentos | Comprobaciones pesadas en la etapa `pre-commit`. | Muévalas a `pre-push`, `manual` o a la CI. |
| `rev` apunta a una rama (`main`) | Configuración no reproducible. | Use etiquetas o *hashes*; `pre-commit autoupdate --freeze`. |

---

## 16. Buenas prácticas

- Versione `.pre-commit-config.yaml` y documente `pre-commit install` en el README o en el script de arranque.
- Fije `rev` en **etiquetas o *hashes***, nunca en ramas; actualice periódicamente con `autoupdate` (o pre-commit.ci / Renovate).
- Mantenga la etapa `pre-commit` **rápida**; lo lento, en `pre-push` o en la CI.
- Use *hooks* que **corrigen** (Ruff con `--fix`, formateadores) para reducir fricción.
- Ejecute **siempre** pre-commit también en la CI: los *hooks* locales se pueden omitir.
- Para herramientas que necesitan el entorno del proyecto (mypy, pytest), use *hooks* locales con `uv run`.
- Incluya detección de **secretos** (gitleaks, `detect-private-key`) y bloquee archivos `.env`.
- Codifique las convenciones del equipo con `pygrep` y `fail` antes de escribir scripts.
- Haga la primera aplicación masiva en un *commit* aislado y regístrelo en `.git-blame-ignore-revs`.
- Active los *hooks* meta (`check-hooks-apply`, `check-useless-excludes`) para mantener limpia la configuración.

---

## 17. Referencia rápida

```bash
pre-commit install [--hook-type commit-msg --hook-type pre-push] [--install-hooks]
pre-commit run [id] [--all-files] [--files f1 f2] [--from-ref A --to-ref B] [--hook-stage pre-push]
pre-commit autoupdate [--freeze] [--repo URL]
pre-commit validate-config  ·  try-repo RUTA id  ·  clean  ·  gc  ·  uninstall
SKIP=id1,id2 git commit ...   ·   git commit --no-verify
```

```yaml
default_install_hook_types: [pre-commit, commit-msg, pre-push]
repos:
  - repo: URL            # o «local» o «meta»
    rev: vX.Y.Z
    hooks:
      - id: nombre
        args: [...] · files: regex · exclude: regex · types: [python]
        stages: [pre-push] · additional_dependencies: [...] · pass_filenames: false
        entry: comando · language: system|python|pygrep|fail|script
```

---

## 18. Ejercicios propuestos

1. Añada pre-commit a un proyecto propio con los *hooks* básicos y Ruff, y aplíquelo a todo el código.
2. Cree un *hook* `pygrep` que prohíba `datetime.utcnow()` y otro `fail` que impida versionar archivos `.sqlite`.
3. Configure `commit-msg` con Conventional Commits y compruebe que rechaza mensajes no conformes.
4. Escriba un *hook* en Python que verifique que cada módulo de `src/` tiene su archivo de pruebas, y publíquelo como repositorio de *hooks* con `try-repo`.
5. Configure pre-commit en GitHub Actions con caché y mida el tiempo con y sin ella.

---

## 19. Referencias

- Documentación oficial: https://pre-commit.com/
- Catálogo de *hooks*: https://pre-commit.com/hooks.html
- pre-commit-hooks: https://github.com/pre-commit/pre-commit-hooks
- pre-commit.ci: https://pre-commit.ci/
- ruff-pre-commit: https://github.com/astral-sh/ruff-pre-commit
- uv-pre-commit: https://github.com/astral-sh/uv-pre-commit
- prek: https://github.com/j178/prek
- Conventional Commits: https://www.conventionalcommits.org/es/
