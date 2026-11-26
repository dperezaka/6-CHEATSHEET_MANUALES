# Manual de uso: GitHub Actions y GitLab CI

> **Categoría:** Integración continua · **Utilidad:** Ejecuta pruebas y comprobaciones en cada cambio subido
> **Tipo:** Servicios de las plataformas GitHub y GitLab (configuración en YAML dentro del repositorio)

---

## Índice

1. [Descripción](#1-descripción)
2. [Requisitos y puesta en marcha](#2-requisitos-y-puesta-en-marcha)
3. [Conceptos clave](#3-conceptos-clave)
4. [GitHub Actions: estructura de un *workflow*](#4-github-actions-estructura-de-un-workflow)
5. [GitHub Actions: eventos (`on`)](#5-github-actions-eventos-on)
6. [GitHub Actions: trabajos, pasos y matrices](#6-github-actions-trabajos-pasos-y-matrices)
7. [GitHub Actions: caché, artefactos y servicios](#7-github-actions-caché-artefactos-y-servicios)
8. [GitHub Actions: secretos, permisos y entornos](#8-github-actions-secretos-permisos-y-entornos)
9. [GitHub Actions: reutilización](#9-github-actions-reutilización)
10. [GitLab CI: estructura de `.gitlab-ci.yml`](#10-gitlab-ci-estructura-de-gitlab-ciyml)
11. [GitLab CI: reglas, dependencias y matrices](#11-gitlab-ci-reglas-dependencias-y-matrices)
12. [GitLab CI: caché, artefactos e informes](#12-gitlab-ci-caché-artefactos-e-informes)
13. [GitLab CI: variables, entornos y reutilización](#13-gitlab-ci-variables-entornos-y-reutilización)
14. [Publicar en PyPI sin *tokens* (publicación de confianza)](#14-publicar-en-pypi-sin-tokens-publicación-de-confianza)
15. [Seguridad de la cadena de CI](#15-seguridad-de-la-cadena-de-ci)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

La **integración continua** (CI) ejecuta automáticamente, en servidores limpios, las comprobaciones del proyecto cada vez que se sube un cambio: pruebas, *linters*, tipos, seguridad, construcción del paquete o de la imagen Docker y, si todo es correcto, el despliegue o la publicación (CD).

| | **GitHub Actions** | **GitLab CI/CD** |
|---|---|---|
| Archivo | `.github/workflows/*.yml` (varios) | `.gitlab-ci.yml` (uno, con `include`) |
| Unidad | *Workflow* → *jobs* → *steps* | *Pipeline* → *stages* → *jobs* |
| Reutilización | *Actions* del Marketplace, *workflows* reutilizables | `include`, plantillas, componentes CI/CD, `extends` |
| Ejecutores | *Runners* de GitHub o propios | *Runners* compartidos de GitLab o propios |
| Imagen | Máquina virtual (Ubuntu, Windows, macOS) o contenedor | Contenedor Docker por trabajo (`image`) |
| Informes integrados | Anotaciones, resumen del trabajo, *Code scanning* (SARIF) | Pruebas JUnit, cobertura, calidad de código, SAST en la MR |

**Ventajas comunes:**
- Cada cambio se valida igual, sin depender del equipo del desarrollador.
- Los resultados aparecen en la *pull/merge request* y pueden bloquear la fusión.
- Matrices para probar varias versiones de Python y sistemas operativos en paralelo.
- Despliegues reproducibles y auditables.

**Limitaciones:**
- Tiempo de ejecución limitado o de pago según el plan (los *runners* propios lo evitan).
- La configuración YAML crece rápido: hay que organizarla y reutilizarla.
- Un *pipeline* mal protegido es un riesgo de seguridad (secretos, publicación).

---

## 2. Requisitos y puesta en marcha

| | GitHub Actions | GitLab CI |
|---|---|---|
| Activar | Crear un archivo en `.github/workflows/` | Crear `.gitlab-ci.yml` en la raíz |
| Ver ejecuciones | Pestaña **Actions** | **Build → Pipelines** |
| Secretos | **Settings → Secrets and variables → Actions** | **Settings → CI/CD → Variables** |
| Protección de ramas | **Settings → Branches / Rulesets** (exigir comprobaciones) | **Settings → Repository → Protected branches** + «Pipelines must succeed» |
| Validar localmente | `actionlint`, `act` (ejecución local) | **Build → Pipeline editor** (validación y visualización), `glab ci lint` |

Herramientas de validación:

```bash
uvx --from actionlint-py actionlint            # Valida los workflows de GitHub
glab ci lint                                   # Valida .gitlab-ci.yml (CLI de GitLab)
```

---

## 3. Conceptos clave

| Concepto | GitHub Actions | GitLab CI |
|---|---|---|
| Configuración | *Workflow* | *Pipeline* |
| Disparador | Evento (`on`) | Fuente del *pipeline* + `rules` / `workflow:rules` |
| Agrupación | *Job* (paralelos por defecto) | *Stage* (secuenciales) que contiene *jobs* (paralelos) |
| Paso | *Step* (`run` o `uses`) | Líneas de `script` |
| Dependencia | `needs` | `needs` (DAG) o el orden de `stages` |
| Matriz | `strategy.matrix` | `parallel:matrix` |
| Máquina | `runs-on` | `image` + `tags` del *runner* |
| Caché | `actions/cache` o la caché de las acciones `setup-*` | `cache` |
| Resultados | `actions/upload-artifact` | `artifacts` |
| Secretos | `secrets.NOMBRE` | Variables protegidas/enmascaradas |
| Despliegue | `environment` con aprobaciones | `environment` + trabajos manuales/protegidos |

---

## 4. GitHub Actions: estructura de un *workflow*

```yaml
# .github/workflows/ci.yml
name: CI                                  # Nombre visible

on:                                       # Cuándo se ejecuta
  push:
    branches: [main]
  pull_request:

permissions:                              # Permisos mínimos del token
  contents: read

concurrency:                              # Cancela ejecuciones anteriores de la misma rama
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:                                      # Variables para todos los trabajos
  UV_FROZEN: "1"

jobs:
  pruebas:                                # Identificador del trabajo
    name: Pruebas
    runs-on: ubuntu-latest                # Máquina
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4         # Acción reutilizable
      - uses: astral-sh/setup-uv@v6
        with:
          enable-cache: true
      - name: Instalar dependencias       # Comando de terminal
        run: uv sync --locked
      - name: Pruebas
        run: uv run pytest
```

Expresiones y contextos:

| Expresión | Valor |
|---|---|
| `${{ github.ref_name }}` | Rama o etiqueta. |
| `${{ github.sha }}` | *Commit*. |
| `${{ github.event_name }}` | Evento que disparó la ejecución. |
| `${{ runner.os }}` | Sistema del *runner*. |
| `${{ matrix.python }}` | Valor de la matriz. |
| `${{ secrets.PYPI_TOKEN }}` | Secreto. |
| `${{ vars.ENTORNO }}` | Variable de configuración (no secreta). |
| `${{ needs.build.outputs.version }}` | Salida de otro trabajo. |
| `${{ hashFiles('uv.lock') }}` | Hash de archivos (claves de caché). |

Funciones de estado para condiciones: `success()`, `failure()`, `always()`, `cancelled()`.

---

## 5. GitHub Actions: eventos (`on`)

```yaml
on:
  push:
    branches: [main, "release/**"]
    tags: ["v*"]
    paths:                                # Solo si cambian estos archivos
      - "src/**"
      - "tests/**"
      - "pyproject.toml"
      - "uv.lock"
    paths-ignore: ["docs/**", "*.md"]
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
  schedule:
    - cron: "17 3 * * 1-5"                # Laborables a las 03:17 UTC
  workflow_dispatch:                      # Botón «Run workflow» con parámetros
    inputs:
      entorno:
        type: choice
        options: [staging, produccion]
        default: staging
  release:
    types: [published]
  workflow_call:                          # Invocable como workflow reutilizable
```

> En repositorios públicos, `pull_request` desde *forks* se ejecuta **sin acceso a secretos** y con permisos de solo lectura. No use `pull_request_target` para ejecutar código de la PR: es una vía clásica de robo de secretos.

---

## 6. GitHub Actions: trabajos, pasos y matrices

### 6.1 Dependencias, condiciones y salidas

```yaml
jobs:
  calidad:
    runs-on: ubuntu-latest
    steps: [...]

  pruebas:
    needs: calidad                         # Espera a «calidad»
    runs-on: ubuntu-latest
    steps: [...]

  version:
    runs-on: ubuntu-latest
    outputs:
      valor: ${{ steps.leer.outputs.version }}
    steps:
      - uses: actions/checkout@v4
      - id: leer
        run: echo "version=$(grep -m1 '^version' pyproject.toml | cut -d'"' -f2)" >> "$GITHUB_OUTPUT"

  publicar:
    needs: [pruebas, version]
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - run: echo "Publicando ${{ needs.version.outputs.valor }}"
```

### 6.2 Matrices

```yaml
pruebas:
  runs-on: ${{ matrix.os }}
  strategy:
    fail-fast: false                       # No cancelar el resto si una falla
    max-parallel: 6
    matrix:
      os: [ubuntu-latest, windows-latest, macos-latest]
      python: ["3.12", "3.13", "3.14"]
      exclude:
        - os: macos-latest
          python: "3.12"
      include:
        - os: ubuntu-latest
          python: "3.15"
          experimental: true
  continue-on-error: ${{ matrix.experimental == true }}
  steps:
    - uses: actions/checkout@v4
    - uses: astral-sh/setup-uv@v6
      with:
        python-version: ${{ matrix.python }}
    - run: uv run pytest
```

### 6.3 Archivos especiales del *runner*

| Archivo | Uso |
|---|---|
| `$GITHUB_OUTPUT` | Salidas de un paso (`echo "clave=valor" >> "$GITHUB_OUTPUT"`). |
| `$GITHUB_ENV` | Variables para los pasos siguientes. |
| `$GITHUB_PATH` | Añadir directorios al `PATH`. |
| `$GITHUB_STEP_SUMMARY` | Markdown que aparece en el resumen de la ejecución. |

Anotaciones en el código desde un comando:

```bash
echo "::error file=src/app.py,line=10::Mensaje de error"
echo "::warning::Aviso general"
```

Muchas herramientas las generan directamente: `ruff check --output-format github`, `ty check --output-format github`, `pytest` con `pytest-github-actions-annotate-failures`.

---

## 7. GitHub Actions: caché, artefactos y servicios

### 7.1 Caché

```yaml
- uses: astral-sh/setup-uv@v6
  with:
    enable-cache: true                     # Caché de uv basada en uv.lock

- uses: actions/cache@v4                   # Caché genérica
  with:
    path: |
      ~/.cache/pre-commit
      .mypy_cache
    key: herramientas-${{ runner.os }}-${{ hashFiles('.pre-commit-config.yaml', 'uv.lock') }}
    restore-keys: herramientas-${{ runner.os }}-
```

### 7.2 Artefactos

```yaml
- uses: actions/upload-artifact@v4
  if: always()                             # Subir también si las pruebas fallan
  with:
    name: informes-${{ matrix.python }}
    path: |
      coverage.xml
      junit.xml
    retention-days: 14

# En otro trabajo
- uses: actions/download-artifact@v4
  with:
    pattern: informes-*
    merge-multiple: true
```

### 7.3 Servicios (bases de datos para pruebas de integración)

```yaml
integracion:
  runs-on: ubuntu-latest
  services:
    postgres:
      image: postgres:17
      env:
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
        POSTGRES_DB: tienda
      ports: ["5432:5432"]
      options: >-
        --health-cmd "pg_isready -U test" --health-interval 5s --health-timeout 5s --health-retries 10
    redis:
      image: redis:7
      ports: ["6379:6379"]
  env:
    DATABASE_URL: postgresql://test:test@localhost:5432/tienda
    REDIS_URL: redis://localhost:6379/0
  steps:
    - uses: actions/checkout@v4
    - uses: astral-sh/setup-uv@v6
    - run: uv run pytest -m integracion
```

---

## 8. GitHub Actions: secretos, permisos y entornos

### 8.1 Secretos y variables

```yaml
env:
  API_URL: ${{ vars.API_URL }}                 # Variable no secreta
steps:
  - run: ./desplegar.sh
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}      # Solo en el paso que lo necesita
```

> Los secretos se enmascaran en los registros, pero **no** imprima ni transforme secretos (codificarlos en base64 los dejaría visibles).

### 8.2 Permisos del `GITHUB_TOKEN`

```yaml
permissions: {}                       # Por defecto, ninguno (a nivel de workflow)

jobs:
  pruebas:
    permissions:
      contents: read
  seguridad:
    permissions:
      contents: read
      security-events: write          # Subir SARIF a Code scanning
  publicar:
    permissions:
      id-token: write                 # Publicación de confianza (OIDC)
      contents: write                 # Crear la release
```

### 8.3 Entornos con aprobación

En **Settings → Environments** cree `produccion` con revisores obligatorios, ramas permitidas y secretos propios:

```yaml
desplegar:
  needs: pruebas
  runs-on: ubuntu-latest
  environment:
    name: produccion
    url: https://tienda.empresa.com
  steps:
    - run: ./desplegar.sh
      env:
        CLAVE: ${{ secrets.CLAVE_PRODUCCION }}     # Secreto del entorno
```

El trabajo queda en espera hasta que un revisor lo aprueba.

---

## 9. GitHub Actions: reutilización

### 9.1 *Workflow* reutilizable

```yaml
# .github/workflows/_pruebas.yml
on:
  workflow_call:
    inputs:
      python:
        type: string
        default: "3.13"
    secrets:
      CODECOV_TOKEN:
        required: false

jobs:
  pruebas:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
        with:
          python-version: ${{ inputs.python }}
      - run: uv run pytest
```

```yaml
# .github/workflows/ci.yml
jobs:
  pruebas:
    uses: ./.github/workflows/_pruebas.yml        # u «org/repo/.github/workflows/x.yml@v1»
    with:
      python: "3.14"
    secrets: inherit
```

### 9.2 Acción compuesta (pasos reutilizables)

```yaml
# .github/actions/preparar-python/action.yml
name: Preparar Python
description: Instala uv y las dependencias del proyecto
inputs:
  python:
    default: "3.13"
runs:
  using: composite
  steps:
    - uses: astral-sh/setup-uv@v6
      with:
        python-version: ${{ inputs.python }}
        enable-cache: true
    - run: uv sync --locked --all-extras
      shell: bash
```

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: ./.github/actions/preparar-python
    with:
      python: "3.14"
```

### 9.3 Dependabot (actualizaciones automáticas)

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: uv
    directory: /
    schedule: { interval: weekly }
    groups:
      desarrollo:
        dependency-type: development
  - package-ecosystem: github-actions
    directory: /
    schedule: { interval: weekly }
  - package-ecosystem: docker
    directory: /
    schedule: { interval: weekly }
```

---

## 10. GitLab CI: estructura de `.gitlab-ci.yml`

```yaml
stages: [calidad, pruebas, construccion, despliegue]

default:
  image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim
  interruptible: true                     # Se cancela si llega un pipeline más nuevo
  timeout: 15m

variables:
  UV_CACHE_DIR: .uv-cache
  UV_LINK_MODE: copy

workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS
      when: never                         # Evita pipelines duplicados (rama + MR)
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_COMMIT_TAG

lint:
  stage: calidad
  script:
    - uv run --frozen ruff check --output-format=gitlab --output-file=gl-code-quality-report.json . || (cat gl-code-quality-report.json; exit 1)
    - uv run --frozen ruff format --check .
  artifacts:
    when: always
    reports:
      codequality: gl-code-quality-report.json

pruebas:
  stage: pruebas
  script:
    - uv sync --locked
    - uv run pytest --junitxml=junit.xml --cov --cov-report=xml --cov-report=term
  coverage: '/TOTAL.*\s(\d+(?:\.\d+)?)%/'
  artifacts:
    when: always
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```

Variables predefinidas más útiles:

| Variable | Valor |
|---|---|
| `CI_COMMIT_BRANCH` / `CI_COMMIT_TAG` | Rama o etiqueta. |
| `CI_COMMIT_SHA` / `CI_COMMIT_SHORT_SHA` | *Commit*. |
| `CI_DEFAULT_BRANCH` | Rama principal. |
| `CI_PIPELINE_SOURCE` | Origen: `push`, `merge_request_event`, `schedule`, `web`… |
| `CI_MERGE_REQUEST_TARGET_BRANCH_NAME` | Rama destino de la MR. |
| `CI_PROJECT_DIR` | Directorio del repositorio en el *runner*. |
| `CI_REGISTRY_IMAGE` | Ruta de la imagen en el registro del proyecto. |
| `CI_JOB_TOKEN` | *Token* temporal del trabajo. |

---

## 11. GitLab CI: reglas, dependencias y matrices

### 11.1 `rules`

```yaml
docs:
  stage: construccion
  script: [uv run mkdocs build --strict]
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      changes: [docs/**/*, mkdocs.yml]
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

desplegar:
  stage: despliegue
  script: [./desplegar.sh]
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual                        # Requiere pulsar un botón
    - when: never
```

### 11.2 `needs`: grafo de dependencias

```yaml
pruebas:
  stage: pruebas
  needs: []                               # Empieza sin esperar a la etapa anterior

construir:
  stage: construccion
  needs: [lint, pruebas]                  # Solo depende de estos trabajos
```

### 11.3 Matrices

```yaml
pruebas:
  stage: pruebas
  image: ghcr.io/astral-sh/uv:python${PYTHON}-bookworm-slim
  parallel:
    matrix:
      - PYTHON: ["3.12", "3.13", "3.14"]
        BD: [sqlite]
      - PYTHON: ["3.13"]
        BD: [postgres]
  script:
    - uv run pytest
```

### 11.4 Servicios

```yaml
integracion:
  services:
    - name: postgres:17
      alias: bd
  variables:
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    POSTGRES_DB: tienda
    DATABASE_URL: postgresql://test:test@bd:5432/tienda
  script:
    - uv run pytest -m integracion
```

---

## 12. GitLab CI: caché, artefactos e informes

```yaml
.cache-uv: &cache-uv
  key:
    files: [uv.lock]
  paths: [.uv-cache]
  policy: pull-push                       # «pull» en trabajos que solo leen

pruebas:
  cache: *cache-uv
  after_script:
    - uv cache prune --ci
  artifacts:
    when: always
    expire_in: 2 weeks
    paths: [htmlcov/]
    reports:
      junit: junit.xml
      coverage_report: { coverage_format: cobertura, path: coverage.xml }
```

Informes que GitLab integra en la *merge request*:

| `reports` | Generado por | Se muestra |
|---|---|---|
| `junit` | `pytest --junitxml=junit.xml` | Pestaña de pruebas con fallos. |
| `coverage_report` (cobertura) | `pytest --cov --cov-report=xml` | Cobertura línea a línea en el *diff*. |
| `codequality` | `ruff check --output-format=gitlab` | Avisos de calidad en el *diff*. |
| `sast` | Plantilla SAST (incluye Bandit/Semgrep) | Panel de seguridad. |
| `dependency_scanning` / `cyclonedx` | Plantillas / `pip-audit -f cyclonedx-json` | Vulnerabilidades y SBOM. |
| `dotenv` | Archivo `.env` | Variables para trabajos posteriores. |

---

## 13. GitLab CI: variables, entornos y reutilización

### 13.1 Variables y secretos

En **Settings → CI/CD → Variables**: marque **Masked** (oculta en registros) y **Protected** (solo en ramas/etiquetas protegidas). Tipo **File** para certificados o archivos de credenciales.

### 13.2 Entornos

```yaml
desplegar_staging:
  stage: despliegue
  environment:
    name: staging
    url: https://staging.tienda.empresa.com
  script: [./desplegar.sh staging]
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

desplegar_produccion:
  stage: despliegue
  environment:
    name: produccion
    url: https://tienda.empresa.com
  script: [./desplegar.sh produccion]
  rules:
    - if: $CI_COMMIT_TAG
      when: manual
```

Con **entornos protegidos** (planes de pago), solo ciertos usuarios pueden ejecutar el despliegue.

### 13.3 Reutilización: `extends`, anclas e `include`

```yaml
.python:                                  # Plantilla oculta (empieza por punto)
  image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim
  before_script:
    - uv sync --locked
  cache:
    key: { files: [uv.lock] }
    paths: [.uv-cache]

tipos:
  extends: .python
  script: [uv run mypy]

include:
  - local: ci/seguridad.yml               # Archivo del propio repositorio
  - project: plataforma/plantillas-ci     # Otro proyecto
    ref: v2.1.0
    file: python.yml
  - template: Jobs/SAST.gitlab-ci.yml     # Plantillas de GitLab
  - component: $CI_SERVER_FQDN/plataforma/componentes/python-tests@1.4.0
    inputs:
      python_version: "3.13"
```

---

## 14. Publicar en PyPI sin *tokens* (publicación de confianza)

La **publicación de confianza** (*Trusted Publishing*) permite que la CI publique en PyPI mediante OIDC, **sin guardar *tokens***. Se configura una vez en PyPI (**Your project → Publishing → Add a trusted publisher**) indicando el repositorio, el *workflow* o el trabajo y, opcionalmente, el entorno.

### 14.1 GitHub Actions

```yaml
publicar:
  needs: construir
  runs-on: ubuntu-latest
  environment: pypi
  permissions:
    id-token: write
  steps:
    - uses: actions/download-artifact@v4
      with: { name: dist, path: dist }
    - uses: pypa/gh-action-pypi-publish@release/v1
```

### 14.2 GitLab CI

```yaml
publicar:
  stage: despliegue
  image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim
  environment: pypi
  id_tokens:
    PYPI_ID_TOKEN:
      aud: pypi
  script:
    - uv build
    - uv publish --trusted-publishing always
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
```

> Las acciones oficiales generan además **atestaciones** (PEP 740) que vinculan el paquete publicado con el *commit* y el *workflow* que lo construyó.

---

## 15. Seguridad de la cadena de CI

- **Permisos mínimos**: `permissions: {}` por defecto en GitHub y ampliar solo por trabajo.
- **Fije las acciones de terceros por *hash***: `uses: org/accion@<sha-completo> # v1.2.3` (las etiquetas se pueden mover). Dependabot actualiza los *hash*.
- **Secretos solo donde se necesitan**, a nivel de paso o de entorno; nunca en `env` global.
- No ejecute código de PR externas con secretos (`pull_request_target`, *pipelines* de *forks* con variables protegidas).
- Use **entornos protegidos** y aprobaciones para producción y publicación.
- Prefiera **OIDC** (publicación de confianza, credenciales temporales de AWS/GCP/Azure) a claves estáticas.
- Revise los *workflows* con **zizmor** (auditoría de seguridad de GitHub Actions) y **actionlint**.
- Separe construcción y publicación en trabajos distintos: el que publica no ejecuta código del proyecto.

```bash
uvx zizmor .github/workflows/
```

---

## 16. Ejemplo práctico completo

***Pipeline* completo para un paquete Python con uv** en ambas plataformas: calidad (Ruff, mypy), seguridad (Bandit, pip-audit), pruebas en matriz con cobertura combinada, construcción y verificación del paquete, documentación, y publicación en PyPI con publicación de confianza al crear una etiqueta `vX.Y.Z`.

### 16.1 GitHub Actions (`.github/workflows/ci.yml`)

```yaml
name: CI

on:
  push:
    branches: [main]
    tags: ["v*.*.*"]
  pull_request:
  schedule:
    - cron: "23 4 * * 1"                  # Lunes: detectar roturas por dependencias nuevas
  workflow_dispatch:

permissions: {}

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

env:
  FORCE_COLOR: "1"
  UV_LOCKED: "1"

jobs:
  calidad:
    name: Lint y tipos
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@v6
        with:
          enable-cache: true
      - run: uv sync --all-extras
      - name: Ruff
        run: |
          uv run ruff check --output-format github .
          uv run ruff format --check --diff .
      - name: mypy
        run: uv run mypy

  seguridad:
    name: Seguridad
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@v6
      - name: Bandit (SARIF)
        run: uvx --from "bandit[sarif,toml]" bandit -c pyproject.toml -r src -f sarif -o bandit.sarif --exit-zero
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: bandit.sarif
      - name: Bandit (barrera)
        run: uvx --from "bandit[toml]" bandit -c pyproject.toml -r src -ll -ii -q
      - name: pip-audit
        run: |
          uv export --no-emit-project --no-hashes -o requisitos.txt
          uvx pip-audit -r requisitos.txt --no-deps --disable-pip --desc -f markdown -o auditoria.md \
            || { cat auditoria.md >> "$GITHUB_STEP_SUMMARY"; exit 1; }

  pruebas:
    name: Pruebas (${{ matrix.os }}, ${{ matrix.python }})
    runs-on: ${{ matrix.os }}
    timeout-minutes: 20
    permissions:
      contents: read
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest]
        python: ["3.12", "3.13", "3.14"]
        include:
          - { os: windows-latest, python: "3.13" }
          - { os: macos-latest, python: "3.13" }
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@v6
        with:
          enable-cache: true
          python-version: ${{ matrix.python }}
      - run: uv sync --all-extras
      - name: pytest
        run: uv run pytest --cov --cov-report= --junitxml=junit.xml
        env:
          COVERAGE_FILE: .coverage.${{ matrix.os }}.${{ matrix.python }}
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: cobertura-${{ matrix.os }}-${{ matrix.python }}
          path: .coverage.*
          include-hidden-files: true
          retention-days: 3

  cobertura:
    name: Cobertura combinada
    needs: pruebas
    if: always() && needs.pruebas.result != 'cancelled'
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@v6
      - uses: actions/download-artifact@v4
        with:
          pattern: cobertura-*
          merge-multiple: true
      - name: Combinar e informar
        run: |
          uvx coverage combine
          uvx coverage report --format=markdown >> "$GITHUB_STEP_SUMMARY"
          uvx coverage report --fail-under=90

  construir:
    name: Construir paquete
    needs: [calidad, seguridad, pruebas]
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
          fetch-depth: 0                  # Versión dinámica desde etiquetas Git
      - uses: astral-sh/setup-uv@v6
      - run: uv build
      - run: uvx twine check --strict dist/*
      - name: Instalar y probar la rueda
        run: |
          uv venv /tmp/prueba
          uv pip install --python /tmp/prueba dist/*.whl
          /tmp/prueba/bin/python -c "import tienda; print('Importación correcta')"
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  publicar:
    name: Publicar en PyPI
    needs: construir
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    environment:
      name: pypi
      url: https://pypi.org/project/tienda/
    permissions:
      id-token: write
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - uses: pypa/gh-action-pypi-publish@release/v1
```

### 16.2 GitLab CI (`.gitlab-ci.yml`)

```yaml
stages: [calidad, pruebas, construccion, publicacion]

workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS
      when: never
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
    - if: $CI_PIPELINE_SOURCE == "schedule"

default:
  image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim
  interruptible: true
  timeout: 20m

variables:
  UV_CACHE_DIR: $CI_PROJECT_DIR/.uv-cache
  UV_LINK_MODE: copy
  UV_LOCKED: "1"

.uv:
  cache:
    key:
      files: [uv.lock]
      prefix: uv-$CI_JOB_IMAGE
    paths: [.uv-cache]
  before_script:
    - uv sync --all-extras
  after_script:
    - uv cache prune --ci

# ---------------------------------------------------------------- Calidad
lint:
  stage: calidad
  extends: .uv
  needs: []
  script:
    - uv run ruff format --check .
    - uv run ruff check --output-format=gitlab --output-file=gl-code-quality-report.json . || FALLO=1
    - uv run ruff check --output-format=concise .
    - exit ${FALLO:-0}
  artifacts:
    when: always
    reports:
      codequality: gl-code-quality-report.json

tipos:
  stage: calidad
  extends: .uv
  needs: []
  script:
    - uv run mypy

seguridad:
  stage: calidad
  needs: []
  script:
    - uvx --from "bandit[toml]" bandit -c pyproject.toml -r src -ll -ii -q
    - uv export --no-emit-project --no-hashes -o requisitos.txt
    - uvx pip-audit -r requisitos.txt --no-deps --disable-pip --desc
    - uvx pip-audit -r requisitos.txt --no-deps --disable-pip -f cyclonedx-json -o sbom.json || true
  artifacts:
    when: always
    paths: [sbom.json]
    reports:
      cyclonedx: sbom.json

# ---------------------------------------------------------------- Pruebas
pruebas:
  stage: pruebas
  extends: .uv
  needs: []
  image: ghcr.io/astral-sh/uv:python${PYTHON}-bookworm-slim
  parallel:
    matrix:
      - PYTHON: ["3.12", "3.13", "3.14"]
  variables:
    COVERAGE_FILE: .coverage.$PYTHON
  script:
    - uv run pytest --cov --cov-report= --junitxml=junit-$PYTHON.xml
  artifacts:
    when: always
    expire_in: 3 days
    paths: [.coverage.*]
    reports:
      junit: junit-*.xml

cobertura:
  stage: pruebas
  needs: [pruebas]
  script:
    - uvx coverage combine
    - uvx coverage xml
    - uvx coverage report --fail-under=90
  coverage: '/TOTAL.*\s(\d+(?:\.\d+)?)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

# ---------------------------------------------------------------- Construcción y publicación
construir:
  stage: construccion
  needs: [lint, tipos, seguridad, cobertura]
  variables:
    GIT_DEPTH: "0"
  script:
    - uv build
    - uvx twine check --strict dist/*
    - uv venv /tmp/prueba && uv pip install --python /tmp/prueba dist/*.whl
    - /tmp/prueba/bin/python -c "import tienda"
  artifacts:
    paths: [dist/]
    expire_in: 1 week

publicar:
  stage: publicacion
  needs: [construir]
  environment:
    name: pypi
    url: https://pypi.org/project/tienda/
  id_tokens:
    PYPI_ID_TOKEN:
      aud: pypi
  script:
    - uv publish --trusted-publishing always dist/*
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
```

### 16.3 Proteger la rama principal

| Plataforma | Configuración |
|---|---|
| GitHub | **Settings → Rules → Rulesets**: rama `main`, exigir PR, exigir las comprobaciones «Lint y tipos», «Seguridad», «Cobertura combinada» y «Construir paquete», prohibir *push* forzado. |
| GitLab | **Settings → Merge requests**: «Pipelines must succeed»; **Repository → Protected branches**: `main` sin *push* directo; reglas de aprobación. |

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| El *workflow* no aparece / no se ejecuta | YAML inválido, ruta distinta de `.github/workflows/` o evento que no coincide. | `actionlint`; revise `on` y los filtros `branches`/`paths`. |
| `Resource not accessible by integration` | El `GITHUB_TOKEN` no tiene el permiso necesario. | Añada el permiso concreto en `permissions` del trabajo. |
| Los secretos llegan vacíos en PR de *forks* | Comportamiento de seguridad de GitHub. | Diseñe los trabajos de PR sin secretos. |
| La caché nunca se reutiliza | Clave que cambia en cada ejecución. | Base la clave en `hashFiles` de los archivos de bloqueo. |
| GitLab ejecuta dos *pipelines* (rama y MR) | Falta la regla que evita duplicados. | Use la combinación de `workflow:rules` del ejemplo. |
| `This job is stuck because you don't have any active runners` | No hay *runners* disponibles o los `tags` no coinciden. | Active los *runners* compartidos o registre uno con esos `tags`. |
| `coverage combine` no encuentra archivos | Los artefactos `.coverage.*` son ocultos. | `include-hidden-files: true` (GitHub) o rutas explícitas en `artifacts:paths` (GitLab). |
| La versión del paquete sale `0.0.0` | Clonado superficial sin etiquetas (versión dinámica desde Git). | `fetch-depth: 0` (GitHub) o `GIT_DEPTH: "0"` (GitLab). |
| `invalid-publisher` al publicar en PyPI | La configuración del editor de confianza no coincide (nombre del *workflow*, entorno). | Revise repositorio, archivo de *workflow* / trabajo y entorno en PyPI. |
| Pruebas que pasan en local y fallan en Windows | Rutas, finales de línea o codificación. | `pathlib`, `newline=""`, `encoding="utf-8"`; incluya Windows en la matriz. |

---

## 18. Buenas prácticas

- Ejecute en CI **lo mismo** que en local: los mismos comandos (`uv run …`, `nox`, `pre-commit`) y las mismas versiones (archivo de bloqueo).
- Falle rápido: trabajos de calidad cortos en paralelo con las pruebas, `concurrency`/`interruptible` para cancelar ejecuciones obsoletas.
- Use **matrices** para versiones de Python y sistemas operativos; `fail-fast: false` para ver todos los fallos.
- Cachee las dependencias con claves basadas en el bloqueo.
- Publique informes (JUnit, cobertura, calidad, SARIF) para verlos en la PR/MR.
- **Permisos mínimos**, secretos por entorno, acciones fijadas por *hash* y OIDC en lugar de claves.
- Proteja la rama principal exigiendo que el *pipeline* pase.
- Programe una ejecución periódica para detectar roturas por cambios externos (dependencias, vulnerabilidades).
- Separe construir de publicar; publique solo desde etiquetas y con aprobación.
- Valide los archivos de CI con `actionlint`/`zizmor` (GitHub) o el editor de *pipelines* (GitLab).

---

## 19. Referencia rápida

```yaml
# GitHub Actions
on: { push: { branches: [main], tags: ["v*"] }, pull_request: {}, schedule: [{ cron: "0 3 * * 1" }], workflow_dispatch: {} }
permissions: {}
jobs:
  x:
    runs-on: ubuntu-latest
    needs: [y]
    if: github.event_name == 'push'
    strategy: { fail-fast: false, matrix: { python: ["3.12", "3.13"] } }
    permissions: { contents: read }
    environment: produccion
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
        with: { python-version: "${{ matrix.python }}", enable-cache: true }
      - run: uv run pytest
        env: { TOKEN: "${{ secrets.TOKEN }}" }
```

```yaml
# GitLab CI
stages: [a, b]
workflow: { rules: [{ if: $CI_PIPELINE_SOURCE == "merge_request_event" }, { if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH }] }
default: { image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim }
trabajo:
  stage: a
  extends: .plantilla
  needs: []
  parallel: { matrix: [{ PYTHON: ["3.12", "3.13"] }] }
  script: [uv run pytest]
  rules: [{ if: $CI_COMMIT_TAG, when: manual }]
  cache: { key: { files: [uv.lock] }, paths: [.uv-cache] }
  artifacts: { reports: { junit: junit.xml } }
  environment: produccion
```

---

## 20. Ejercicios propuestos

1. Cree un *workflow* de GitHub Actions que ejecute Ruff y pytest en tres versiones de Python con caché de uv.
2. Traduzca ese *workflow* a `.gitlab-ci.yml` con `parallel:matrix` e informes JUnit y de cobertura en la MR.
3. Añada un trabajo de integración con PostgreSQL como servicio en ambas plataformas.
4. Configure la publicación de confianza en TestPyPI y publique una versión desde una etiqueta.
5. Audite sus *workflows* con `actionlint` y `zizmor`, y fije todas las acciones de terceros por *hash*.

---

## 21. Referencias

- GitHub Actions, documentación: https://docs.github.com/es/actions
- Sintaxis de *workflows*: https://docs.github.com/es/actions/reference/workflows-and-actions/workflow-syntax
- Seguridad en GitHub Actions: https://docs.github.com/es/actions/reference/security/secure-use
- GitLab CI/CD, documentación: https://docs.gitlab.com/ci/
- Referencia de `.gitlab-ci.yml`: https://docs.gitlab.com/ci/yaml/
- uv en CI: https://docs.astral.sh/uv/guides/integration/github/ · https://docs.astral.sh/uv/guides/integration/gitlab/
- Publicación de confianza en PyPI: https://docs.pypi.org/trusted-publishers/
- actionlint: https://github.com/rhysd/actionlint · zizmor: https://docs.zizmor.sh/
