# Manual de uso: Docker

> **Categoría:** Despliegue · **Utilidad:** Empaqueta la aplicación con su entorno de forma reproducible
> **Tipo:** Plataforma externa (requiere instalación: Docker Engine o Docker Desktop)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Comandos esenciales](#4-comandos-esenciales)
5. [El `Dockerfile`](#5-el-dockerfile)
6. [Imágenes Python: elección de la base](#6-imágenes-python-elección-de-la-base)
7. [Dockerfile para Python con uv (multietapa)](#7-dockerfile-para-python-con-uv-multietapa)
8. [Caché de construcción y `.dockerignore`](#8-caché-de-construcción-y-dockerignore)
9. [Seguridad de la imagen](#9-seguridad-de-la-imagen)
10. [Configuración, secretos y volúmenes](#10-configuración-secretos-y-volúmenes)
11. [Docker Compose](#11-docker-compose)
12. [Desarrollo con contenedores](#12-desarrollo-con-contenedores)
13. [Registros y etiquetado](#13-registros-y-etiquetado)
14. [Construcción multiplataforma y en CI](#14-construcción-multiplataforma-y-en-ci)
15. [Depuración y operación](#15-depuración-y-operación)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Docker** empaqueta una aplicación junto con **todo lo que necesita para ejecutarse** —intérprete de Python, dependencias, librerías del sistema, configuración— en una **imagen**. Esa imagen se ejecuta igual en el portátil del desarrollador, en la CI y en producción, como un **contenedor** aislado.

**Ventajas:**
- Reproducibilidad: misma imagen en todos los entornos; se acabó el «en mi equipo funciona».
- Aislamiento de dependencias, incluidas las del sistema (`libpq`, `libmagic`, fuentes, etc.).
- Despliegue estándar en cualquier plataforma: Kubernetes, ECS, Cloud Run, Azure Container Apps, servidores propios.
- Entornos de desarrollo completos (aplicación + base de datos + caché) con un comando (`docker compose up`).

**Limitaciones:**
- Curva de aprendizaje (capas, redes, volúmenes, seguridad).
- Las imágenes deben mantenerse: parches de seguridad de la base y de las dependencias.
- En macOS y Windows se ejecuta en una máquina virtual: el acceso a archivos montados es más lento.

| Si necesita… | Opción |
|---|---|
| Empaquetar y desplegar servicios | **Docker** (imágenes OCI) |
| Orquestar muchos contenedores en producción | Kubernetes, ECS, Nomad |
| Alternativa sin demonio, compatible | Podman |
| Distribuir una herramienta de terminal a usuarios | pipx / `uv tool`, PyInstaller |

---

## 2. Instalación

| Sistema | Opción |
|---|---|
| Windows / macOS | **Docker Desktop** (incluye Compose y Buildx). En Windows, con WSL 2. |
| Linux | **Docker Engine** desde el repositorio oficial de Docker. |
| Alternativas | Podman, Rancher Desktop, OrbStack (macOS), Colima. |

Linux (Debian/Ubuntu), script oficial:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker "$USER"          # Usar docker sin sudo (cierre sesión y vuelva a entrar)
```

Comprobar:

```bash
docker version
docker compose version
docker run --rm hello-world
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Imagen | Plantilla inmutable de solo lectura (sistema de archivos + metadatos). |
| Capa | Cada instrucción del `Dockerfile` crea una capa; se cachean y comparten. |
| Contenedor | Instancia en ejecución de una imagen, con una capa escribible efímera. |
| `Dockerfile` | Receta para construir una imagen. |
| Registro | Almacén de imágenes: Docker Hub, GHCR, GitLab Registry, ECR, ACR, Artifact Registry. |
| Etiqueta (*tag*) | Nombre de versión de una imagen: `tienda:1.4.0`. |
| *Digest* | Identificador inmutable por contenido: `tienda@sha256:…`. |
| Volumen | Almacenamiento persistente fuera del contenedor. |
| Red | Conecta contenedores entre sí (por nombre de servicio). |
| Docker Compose | Define y ejecuta aplicaciones de varios contenedores con un YAML. |
| BuildKit / Buildx | Motor de construcción moderno: caché avanzada, secretos, multiplataforma. |

---

## 4. Comandos esenciales

### 4.1 Imágenes

```bash
docker build -t tienda:dev .                 # Construir desde el Dockerfile del directorio
docker build -t tienda:dev -f docker/Dockerfile --target produccion .
docker images                                # Listar (docker image ls)
docker image inspect tienda:dev
docker history tienda:dev                    # Capas y tamaño de cada una
docker pull python:3.13-slim
docker push ghcr.io/empresa/tienda:1.4.0
docker rmi tienda:dev
docker image prune                           # Eliminar imágenes sin etiqueta
```

### 4.2 Contenedores

```bash
docker run --rm -it python:3.13-slim python           # Interactivo, se elimina al salir
docker run -d --name api -p 8000:8000 tienda:dev       # En segundo plano, puerto 8000
docker run --rm -e ENTORNO=pruebas --env-file .env tienda:dev
docker run --rm -v "$PWD/datos:/app/datos" tienda:dev  # Montar una carpeta
docker ps                                    # En ejecución (docker ps -a: todos)
docker logs -f api                           # Registros en vivo
docker exec -it api bash                     # Abrir una terminal dentro
docker stop api && docker rm api
docker stats                                 # Consumo de CPU y memoria
docker system df                             # Espacio usado
docker system prune -a --volumes             # Limpieza completa (¡cuidado!)
```

| Opción de `docker run` | Efecto |
|---|---|
| `--rm` | Elimina el contenedor al terminar. |
| `-d` | En segundo plano. |
| `-it` | Interactivo con terminal. |
| `-p 8080:8000` | Publica el puerto 8000 del contenedor en el 8080 del equipo. |
| `-e CLAVE=valor` / `--env-file` | Variables de entorno. |
| `-v origen:destino` | Volumen o carpeta montada. |
| `--name` | Nombre del contenedor. |
| `--memory 512m --cpus 1` | Límites de recursos. |
| `--read-only` | Sistema de archivos de solo lectura. |
| `--init` | Proceso init mínimo (gestiona señales y procesos zombi). |
| `--user 1000:1000` | Usuario con el que se ejecuta. |

---

## 5. El `Dockerfile`

| Instrucción | Uso |
|---|---|
| `FROM imagen AS nombre` | Imagen base (y nombre de la etapa). |
| `WORKDIR /app` | Directorio de trabajo. |
| `COPY origen destino` | Copiar archivos del contexto de construcción. |
| `COPY --from=etapa origen destino` | Copiar desde otra etapa o imagen. |
| `RUN comando` | Ejecutar un comando durante la construcción (crea una capa). |
| `ENV CLAVE=valor` | Variable de entorno (construcción y ejecución). |
| `ARG CLAVE=valor` | Variable solo de construcción (`--build-arg`). |
| `EXPOSE 8000` | Documenta el puerto (no lo publica). |
| `USER app` | Usuario para las instrucciones siguientes y la ejecución. |
| `HEALTHCHECK CMD …` | Comprobación de salud del contenedor. |
| `ENTRYPOINT ["…"]` | Ejecutable principal. |
| `CMD ["…"]` | Argumentos por defecto (o comando si no hay `ENTRYPOINT`). |
| `LABEL clave=valor` | Metadatos (OCI: versión, repositorio, licencia). |

Ejemplo mínimo (válido, pero mejorable: ver sección 7):

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "-m", "tienda"]
```

> Use siempre la **forma exec** (`["python", "-m", "tienda"]`) en `CMD`/`ENTRYPOINT`. La forma de terminal (`CMD python -m tienda`) lanza `/bin/sh` como proceso principal y la aplicación no recibe la señal de parada (`SIGTERM`).

---

## 6. Imágenes Python: elección de la base

| Imagen | Tamaño aprox. | Notas |
|---|---|---|
| `python:3.13` | ~1 GB | Debian completo con compiladores. Para etapas de construcción. |
| `python:3.13-slim` | ~120-150 MB | Debian mínimo. **Opción por defecto recomendada.** |
| `python:3.13-alpine` | ~50 MB | musl en lugar de glibc: muchas *wheels* no sirven y hay que compilar; problemas sutiles. Evítela para Python salvo motivo claro. |
| `ghcr.io/astral-sh/uv:python3.13-bookworm-slim` | ~170 MB | `slim` con uv incluido: cómoda para construir. |
| *Distroless* / Chainguard / Docker Hardened Images | Muy pequeñas | Sin terminal ni gestor de paquetes: menor superficie de ataque; más difíciles de depurar. |

Fije la versión con precisión adecuada:

```dockerfile
FROM python:3.13-slim                         # Recibe parches 3.13.x automáticamente
FROM python:3.13.9-slim-bookworm              # Versión exacta y distribución fija
FROM python:3.13-slim@sha256:<digest>         # Inmutable (Renovate/Dependabot lo actualizan)
```

---

## 7. Dockerfile para Python con uv (multietapa)

La construcción **multietapa** separa la etapa que instala dependencias (con herramientas de compilación, caché, uv) de la imagen final (solo lo necesario para ejecutar):

```dockerfile
# syntax=docker/dockerfile:1

# ---------------------------------------------------------------- Etapa 1: construcción
FROM ghcr.io/astral-sh/uv:python3.13-bookworm-slim AS construccion

ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    UV_NO_DEV=1

WORKDIR /app

# 1) Dependencias: capa que solo se reconstruye si cambia el bloqueo
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project

# 2) Código del proyecto
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-editable

# ---------------------------------------------------------------- Etapa 2: ejecución
FROM python:3.13-slim-bookworm AS produccion

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/app/.venv/bin:$PATH"

RUN groupadd --system --gid 1000 app \
 && useradd --system --uid 1000 --gid app --no-create-home app

WORKDIR /app
COPY --from=construccion --chown=app:app /app/.venv /app/.venv

USER 1000:1000                      # UID numérico: verificable por Kubernetes (runAsNonRoot)
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD ["python", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/salud', timeout=2)"]

CMD ["uvicorn", "tienda.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

| Variable | Motivo |
|---|---|
| `UV_COMPILE_BYTECODE=1` | Compila `.pyc` en la construcción: arranque más rápido. |
| `UV_LINK_MODE=copy` | La caché está en un montaje: copiar en lugar de enlazar. |
| `UV_NO_DEV=1` | No instalar dependencias de desarrollo. |
| `--no-editable` | Instala el proyecto como paquete normal: la etapa final no necesita el código fuente. |
| `PYTHONDONTWRITEBYTECODE=1` | No escribir `.pyc` en ejecución (ya están compilados). |
| `PYTHONUNBUFFERED=1` | Registros sin búfer: aparecen al momento en `docker logs`. |

Con pip en lugar de uv (mismo patrón):

```dockerfile
FROM python:3.13-slim AS construccion
RUN python -m venv /venv
ENV PATH="/venv/bin:$PATH"
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip pip install --require-hashes -r requirements.txt
COPY . .
RUN pip install --no-deps .

FROM python:3.13-slim
COPY --from=construccion /venv /venv
ENV PATH="/venv/bin:$PATH"
...
```

Dependencias del sistema (por ejemplo, para compilar o para `psycopg`):

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends libpq5 \
 && rm -rf /var/lib/apt/lists/*
```

> Instale compiladores (`build-essential`, `gcc`) **solo en la etapa de construcción**; en la final, solo las librerías de ejecución.

---

## 8. Caché de construcción y `.dockerignore`

### 8.1 Orden de las instrucciones

Docker reutiliza una capa si la instrucción y sus archivos no han cambiado. **Copie primero lo que cambia poco** (archivo de bloqueo) y después lo que cambia mucho (código):

```dockerfile
COPY pyproject.toml uv.lock ./          # Cambia poco → capa de dependencias reutilizada
RUN uv sync --locked --no-install-project
COPY src ./src                          # Cambia a menudo → solo se repite esta parte
```

### 8.2 Montajes de caché (BuildKit)

```dockerfile
RUN --mount=type=cache,target=/root/.cache/uv uv sync --locked
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked apt-get update && apt-get install -y ...
```

### 8.3 `.dockerignore`

Excluye archivos del **contexto de construcción** (más rápido y sin filtrar secretos a la imagen):

```gitignore
# .dockerignore
.git
.github
.venv
**/__pycache__
**/*.pyc
.pytest_cache
.mypy_cache
.ruff_cache
.coverage*
htmlcov
dist
build
*.egg-info
.env
.env.*
!.env.example
docs/_build
site
node_modules
Dockerfile*
compose*.yaml
```

---

## 9. Seguridad de la imagen

| Práctica | Cómo |
|---|---|
| Usuario sin privilegios | `USER 1000:1000` (UID numérico; nunca ejecutar como `root`). |
| Base mínima y actualizada | `slim`, *distroless* o *hardened*; reconstruir periódicamente. |
| Sin secretos en la imagen | Nada de `ENV CLAVE=...`, `ARG TOKEN`, ni `COPY .env`. Use secretos de construcción y de ejecución. |
| Dependencias fijadas | Archivo de bloqueo con hashes. |
| Escaneo de vulnerabilidades | `docker scout cves`, Trivy, Grype. |
| Sistema de archivos de solo lectura | `--read-only` + `tmpfs` para lo que deba escribirse. |
| Capacidades mínimas | `--cap-drop ALL`, `--security-opt no-new-privileges`. |
| Firmar y generar SBOM | `docker buildx build --sbom=true --provenance=true`; firma con Cosign. |

Secretos durante la construcción (p. ej., un índice privado), sin dejarlos en ninguna capa:

```dockerfile
RUN --mount=type=secret,id=indice,env=UV_INDEX_INTERNO_PASSWORD \
    uv sync --locked
```

```bash
docker build --secret id=indice,env=TOKEN_INDICE -t tienda .
```

Escanear la imagen:

```bash
docker scout quickview tienda:dev
docker scout cves tienda:dev --only-severity critical,high
trivy image --severity HIGH,CRITICAL --exit-code 1 tienda:dev
```

Analizar el `Dockerfile`:

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
docker build --check .                        # Comprobaciones de buenas prácticas de BuildKit
```

---

## 10. Configuración, secretos y volúmenes

### 10.1 Configuración por variables de entorno

La imagen debe ser **la misma** en todos los entornos; lo que cambia se pasa al ejecutar:

```bash
docker run -e TIENDA_ENTORNO=produccion -e TIENDA_BD__HOST=bd.interna tienda:1.4.0
docker run --env-file produccion.env tienda:1.4.0
```

Combine con **pydantic-settings** para validarla al arrancar.

### 10.2 Secretos en ejecución

- **Docker Compose / Swarm**: `secrets`, montados en `/run/secrets/<nombre>` (sección 11).
- **Kubernetes**: *Secrets* montados como archivos o variables.
- **Nube**: gestores de secretos (AWS Secrets Manager, Azure Key Vault…) e identidades del servicio.

### 10.3 Volúmenes

```bash
docker volume create datos-bd
docker run -v datos-bd:/var/lib/postgresql/data postgres:17      # Volumen con nombre (persistente)
docker run -v "$PWD/config:/app/config:ro" tienda:dev            # Carpeta del equipo, solo lectura
docker run --tmpfs /tmp tienda:dev                               # En memoria
docker volume ls · docker volume inspect datos-bd · docker volume rm datos-bd
```

> Los contenedores son **efímeros**: todo lo que se escriba fuera de un volumen se pierde al eliminarlos.

---

## 11. Docker Compose

Define varios servicios, sus redes y volúmenes en `compose.yaml`:

```yaml
services:
  api:
    build:
      context: .
      target: produccion
    image: tienda:dev
    ports: ["8000:8000"]
    env_file: [.env]
    environment:
      TIENDA_BD__HOST: bd
      TIENDA_REDIS_URL: redis://cache:6379/0
    depends_on:
      bd:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped

  bd:
    image: postgres:17
    environment:
      POSTGRES_DB: tienda
      POSTGRES_USER: tienda
      POSTGRES_PASSWORD_FILE: /run/secrets/bd_clave
    secrets: [bd_clave]
    volumes: [datos-bd:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U tienda -d tienda"]
      interval: 5s
      timeout: 3s
      retries: 10

  cache:
    image: redis:7-alpine

volumes:
  datos-bd:

secrets:
  bd_clave:
    file: ./secretos/bd_clave.txt
```

Comandos:

```bash
docker compose up -d                      # Construir (si hace falta) y arrancar
docker compose up -d --build              # Forzar reconstrucción
docker compose ps
docker compose logs -f api
docker compose exec api bash
docker compose run --rm api pytest        # Comando puntual en un contenedor nuevo
docker compose down                       # Parar y eliminar contenedores y red
docker compose down -v                    # …y también los volúmenes (¡borra datos!)
docker compose config                     # Configuración final (tras variables y combinaciones)
docker compose --profile herramientas up  # Activar servicios de un perfil
```

| Clave | Uso |
|---|---|
| `build` / `image` | Construir desde un `Dockerfile` o usar una imagen. |
| `ports` | Publicar puertos en el equipo. |
| `environment` / `env_file` | Variables de entorno. |
| `depends_on` + `condition` | Orden de arranque y espera a que el servicio esté sano. |
| `volumes` | Volúmenes y carpetas montadas. |
| `secrets` | Secretos como archivos en `/run/secrets`. |
| `healthcheck` | Comprobación de salud. |
| `profiles` | Servicios opcionales (`herramientas`, `monitorizacion`). |
| `restart` | Política de reinicio. |
| `deploy.resources.limits` | Límites de CPU y memoria. |

Los servicios se comunican por **nombre** en una red interna: la API se conecta a `bd:5432`, no a `localhost`.

---

## 12. Desarrollo con contenedores

### 12.1 Archivo de sobrescritura para desarrollo

`compose.override.yaml` se combina automáticamente con `compose.yaml`:

```yaml
services:
  api:
    build:
      target: desarrollo
    command: ["uvicorn", "tienda.main:app", "--host", "0.0.0.0", "--reload"]
    environment:
      TIENDA_ENTORNO: desarrollo
      TIENDA_DEPURACION: "true"
    develop:
      watch:
        - action: sync                    # Copia los cambios de código al contenedor
          path: ./src
          target: /app/src
        - action: rebuild                 # Reconstruye si cambian las dependencias
          path: uv.lock
```

```bash
docker compose watch                      # Arranca y sincroniza los cambios en vivo
```

### 12.2 Ejecutar pruebas en contenedores

```bash
docker compose run --rm api uv run pytest
docker compose -f compose.yaml -f compose.test.yaml run --rm pruebas
```

### 12.3 Dev Containers

VS Code y otros editores pueden abrir el proyecto **dentro** de un contenedor (`.devcontainer/devcontainer.json`), con las mismas herramientas para todo el equipo.

---

## 13. Registros y etiquetado

```bash
docker login ghcr.io -u USUARIO --password-stdin <<< "$TOKEN"
docker tag tienda:dev ghcr.io/empresa/tienda:1.4.0
docker tag tienda:dev ghcr.io/empresa/tienda:1.4
docker push ghcr.io/empresa/tienda:1.4.0
docker push ghcr.io/empresa/tienda:1.4
```

Estrategia de etiquetas:

| Etiqueta | Uso |
|---|---|
| `1.4.0` | Versión exacta (inmutable por convención). |
| `1.4`, `1` | Última versión de esa rama (se mueven). |
| `sha-3f2a1bc` | *Commit* exacto (trazabilidad). |
| `main` / `edge` | Última construcción de la rama principal. |
| `latest` | Evítela en despliegues: no indica qué versión se ejecuta. |

> En producción, despliegue por **digest** (`@sha256:…`) o por etiqueta inmutable.

Etiquetas OCI con metadatos:

```dockerfile
ARG VERSION=dev
LABEL org.opencontainers.image.title="tienda" \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.source="https://github.com/empresa/tienda" \
      org.opencontainers.image.licenses="MIT"
```

---

## 14. Construcción multiplataforma y en CI

### 14.1 Multiplataforma (Intel y ARM)

```bash
docker buildx create --use --name multi
docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/empresa/tienda:1.4.0 --push .
```

### 14.2 GitHub Actions

```yaml
imagen:
  runs-on: ubuntu-latest
  permissions:
    contents: read
    packages: write
  steps:
    - uses: actions/checkout@v4
    - uses: docker/setup-buildx-action@v3
    - uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    - id: meta
      uses: docker/metadata-action@v5
      with:
        images: ghcr.io/${{ github.repository }}
        tags: |
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha
          type=ref,event=branch
    - uses: docker/build-push-action@v6
      with:
        push: ${{ github.event_name != 'pull_request' }}
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        platforms: linux/amd64,linux/arm64
        cache-from: type=gha
        cache-to: type=gha,mode=max
        sbom: true
        provenance: mode=max
```

### 14.3 GitLab CI

```yaml
imagen:
  image: docker:27
  services: [docker:27-dind]
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"
    - docker buildx create --use
    - docker buildx build --push
        --cache-from type=registry,ref=$CI_REGISTRY_IMAGE:cache
        --cache-to type=registry,ref=$CI_REGISTRY_IMAGE:cache,mode=max
        -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
```

---

## 15. Depuración y operación

```bash
docker logs --tail 100 -f api                 # Últimas líneas y seguimiento
docker exec -it api sh                        # Terminal en un contenedor en marcha
docker run --rm -it --entrypoint sh tienda:dev   # Terminal en una imagen sin arrancar la app
docker inspect api --format '{{.State.Health.Status}}'
docker inspect api --format '{{json .State.Health}}' | python -m json.tool
docker cp api:/app/registro.log .             # Copiar archivos desde el contenedor
docker events                                 # Eventos del demonio
docker debug api                              # Herramientas de depuración en imágenes mínimas (Docker Desktop)
docker build --progress=plain --no-cache .    # Construcción detallada y sin caché
```

Parada ordenada: Docker envía `SIGTERM` y, tras 10 s, `SIGKILL`. La aplicación debe terminar las peticiones en curso y cerrar conexiones al recibir `SIGTERM` (Uvicorn y Gunicorn lo hacen). Si su proceso no lo gestiona, use `--init` o `tini`.

Registros: escriba en **stdout/stderr** (no en archivos) en formato estructurado (JSON), y deje que la plataforma los recoja.

---

## 16. Ejemplo práctico completo

**API FastAPI de pedidos lista para producción**: `Dockerfile` multietapa con uv (etapas de construcción, desarrollo, pruebas y producción), usuario sin privilegios, comprobación de salud, Compose con PostgreSQL y secretos, sobrescritura para desarrollo con recarga en caliente y un `Makefile` con las operaciones habituales.

### 16.1 Estructura

```
tienda/
├── Dockerfile
├── .dockerignore
├── compose.yaml
├── compose.override.yaml
├── Makefile
├── pyproject.toml
├── uv.lock
├── secretos/bd_clave.txt          # No versionado
└── src/tienda/
    ├── __init__.py
    ├── config.py
    └── main.py
```

### 16.2 Aplicación (`src/tienda/main.py`)

```python
import logging
import os
import signal
from contextlib import asynccontextmanager
from pathlib import Path

import psycopg
from fastapi import FastAPI, HTTPException

logging.basicConfig(level=os.getenv("LOG_NIVEL", "INFO"), format="%(asctime)s %(levelname)s %(name)s %(message)s")
log = logging.getLogger("tienda")


def url_bd() -> str:
    clave_archivo = Path(os.getenv("BD_CLAVE_ARCHIVO", "/run/secrets/bd_clave"))
    clave = clave_archivo.read_text().strip() if clave_archivo.exists() else os.getenv("BD_CLAVE", "")
    return (f"postgresql://{os.getenv('BD_USUARIO', 'tienda')}:{clave}"
            f"@{os.getenv('BD_HOST', 'localhost')}:{os.getenv('BD_PUERTO', '5432')}/{os.getenv('BD_NOMBRE', 'tienda')}")


@asynccontextmanager
async def ciclo_vida(app: FastAPI):
    app.state.conexion = await psycopg.AsyncConnection.connect(url_bd(), autocommit=True)
    await app.state.conexion.execute(
        "CREATE TABLE IF NOT EXISTS pedidos (id SERIAL PRIMARY KEY, cliente TEXT NOT NULL, total NUMERIC(10,2))"
    )
    log.info("Conectado a la base de datos")
    yield
    await app.state.conexion.close()
    log.info("Conexión cerrada; parada ordenada")


app = FastAPI(title="Tienda", lifespan=ciclo_vida)


@app.get("/salud")
async def salud() -> dict[str, str]:
    try:
        await app.state.conexion.execute("SELECT 1")
    except Exception as e:
        raise HTTPException(status_code=503, detail="Base de datos no disponible") from e
    return {"estado": "ok"}


@app.post("/pedidos", status_code=201)
async def crear(cliente: str, total: float) -> dict[str, object]:
    cur = await app.state.conexion.execute(
        "INSERT INTO pedidos (cliente, total) VALUES (%s, %s) RETURNING id", (cliente, total)
    )
    (id_,) = await cur.fetchone()
    return {"id": id_, "cliente": cliente, "total": total}


@app.get("/pedidos")
async def listar() -> list[dict[str, object]]:
    cur = await app.state.conexion.execute("SELECT id, cliente, total FROM pedidos ORDER BY id")
    return [{"id": i, "cliente": c, "total": float(t)} for i, c, t in await cur.fetchall()]
```

### 16.3 `Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1
ARG PYTHON=3.13

# ================================================================ Base de construcción
FROM ghcr.io/astral-sh/uv:python${PYTHON}-bookworm-slim AS base
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy UV_PYTHON_DOWNLOADS=never
WORKDIR /app

# ================================================================ Dependencias de producción
FROM base AS construccion
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-dev --no-install-project
COPY pyproject.toml uv.lock README.md ./
COPY src ./src
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-dev --no-editable

# ================================================================ Desarrollo (con dependencias dev)
FROM base AS desarrollo
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project
COPY . .
RUN --mount=type=cache,target=/root/.cache/uv uv sync --locked
ENV PATH="/app/.venv/bin:$PATH" PYTHONUNBUFFERED=1
CMD ["uvicorn", "tienda.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]

# ================================================================ Pruebas
FROM desarrollo AS pruebas
CMD ["pytest", "-q"]

# ================================================================ Producción
FROM python:${PYTHON}-slim-bookworm AS produccion
ARG VERSION=dev
LABEL org.opencontainers.image.title="tienda" \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.source="https://github.com/empresa/tienda"

ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1 PATH="/app/.venv/bin:$PATH"
RUN groupadd --system --gid 1000 app && useradd --system --uid 1000 --gid app --no-create-home app
WORKDIR /app
COPY --from=construccion --chown=app:app /app/.venv /app/.venv
USER 1000:1000

EXPOSE 8000
HEALTHCHECK --interval=15s --timeout=3s --start-period=20s --retries=3 \
  CMD ["python", "-c", "import urllib.request,sys; sys.exit(urllib.request.urlopen('http://127.0.0.1:8000/salud', timeout=2).status != 200)"]

CMD ["uvicorn", "tienda.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2", \
     "--proxy-headers", "--forwarded-allow-ips", "*", "--timeout-graceful-shutdown", "20"]
```

### 16.4 `compose.yaml` (base, como en producción)

```yaml
name: tienda

services:
  api:
    build:
      context: .
      target: produccion
      args:
        VERSION: ${VERSION:-dev}
    image: ghcr.io/empresa/tienda:${VERSION:-dev}
    ports: ["8000:8000"]
    environment:
      BD_HOST: bd
      BD_USUARIO: tienda
      BD_NOMBRE: tienda
      BD_CLAVE_ARCHIVO: /run/secrets/bd_clave
      LOG_NIVEL: INFO
    secrets: [bd_clave]
    depends_on:
      bd:
        condition: service_healthy
    read_only: true
    tmpfs: [/tmp]
    security_opt: ["no-new-privileges:true"]
    cap_drop: [ALL]
    deploy:
      resources:
        limits: { cpus: "1.0", memory: 512M }
    restart: unless-stopped
    stop_grace_period: 25s

  bd:
    image: postgres:17
    environment:
      POSTGRES_DB: tienda
      POSTGRES_USER: tienda
      POSTGRES_PASSWORD_FILE: /run/secrets/bd_clave
    secrets: [bd_clave]
    volumes: [datos-bd:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U tienda -d tienda"]
      interval: 5s
      timeout: 3s
      retries: 10
    restart: unless-stopped

  adminer:
    image: adminer:5
    ports: ["8080:8080"]
    profiles: [herramientas]

volumes:
  datos-bd:

secrets:
  bd_clave:
    file: ./secretos/bd_clave.txt
```

### 16.5 `compose.override.yaml` (solo desarrollo)

```yaml
services:
  api:
    build:
      target: desarrollo
    image: tienda:desarrollo
    read_only: false
    environment:
      LOG_NIVEL: DEBUG
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app/src
        - action: rebuild
          path: uv.lock
```

### 16.6 `Makefile`

```makefile
VERSION ?= $(shell git describe --tags --always --dirty)
IMAGEN  ?= ghcr.io/empresa/tienda

.PHONY: secretos dev prod pruebas logs escanear publicar limpiar

secretos:
	@mkdir -p secretos && test -f secretos/bd_clave.txt || python3 -c "import secrets; print(secrets.token_urlsafe(24))" > secretos/bd_clave.txt

dev: secretos                ## Desarrollo con recarga en caliente
	docker compose watch

prod: secretos               ## Pila como en producción (sin override)
	VERSION=$(VERSION) docker compose -f compose.yaml up -d --build --wait
	curl -fsS localhost:8000/salud && echo

pruebas:                     ## Pruebas dentro del contenedor
	docker build --target pruebas -t tienda:pruebas .
	docker run --rm tienda:pruebas

logs:
	docker compose logs -f api

escanear:                    ## Vulnerabilidades de la imagen
	docker build --target produccion -t $(IMAGEN):$(VERSION) .
	docker scout cves $(IMAGEN):$(VERSION) --only-severity critical,high --exit-code

publicar:
	docker buildx build --platform linux/amd64,linux/arm64 --target produccion \
	  --build-arg VERSION=$(VERSION) -t $(IMAGEN):$(VERSION) --sbom=true --provenance=true --push .

limpiar:
	docker compose down --remove-orphans
```

Uso:

```bash
make dev                                   # http://localhost:8000/docs, recarga al guardar
make prod                                  # Imagen de producción + PostgreSQL, espera a que estén sanos
curl -X POST "localhost:8000/pedidos?cliente=Ana&total=59.90"
curl localhost:8000/pedidos
docker compose --profile herramientas up -d adminer   # Interfaz web de la BD en :8080
docker compose ps                          # Columna STATUS: (healthy)
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Cannot connect to the Docker daemon` | El servicio de Docker no está arrancado o falta permiso. | Inicie Docker Desktop / `sudo systemctl start docker`; añada el usuario al grupo `docker`. |
| La API no conecta con la BD (`localhost`) | Dentro del contenedor, `localhost` es el propio contenedor. | Use el nombre del servicio (`bd`). |
| `port is already allocated` | El puerto del equipo está en uso. | Cambie el puerto publicado (`-p 8001:8000`) o pare el otro proceso. |
| Cada construcción reinstala todas las dependencias | Se copia todo el código antes de instalar dependencias. | Copie primero `pyproject.toml`/`uv.lock` (sección 8). |
| La imagen pesa más de 1 GB | Base completa, compiladores o cachés en la imagen final. | Multietapa, base `slim`, `--no-cache`, `.dockerignore`. |
| `exec format error` | Imagen de otra arquitectura (ARM/Intel). | `--platform linux/amd64` o construya multiplataforma. |
| `Permission denied` al escribir | Usuario sin privilegios sin permisos en la carpeta. | `--chown` en `COPY`, volumen con el propietario correcto o escriba en `/tmp`. |
| El contenedor tarda 10 s en pararse | El proceso no recibe/gestiona `SIGTERM` (forma *shell* de `CMD`). | Forma exec en `CMD`; `--init` si hace falta. |
| No aparecen los registros | Salida con búfer. | `PYTHONUNBUFFERED=1`. |
| Error al compilar una dependencia en Alpine | musl: no hay *wheel* compatible. | Use una base Debian `slim`. |
| `depends_on` no espera a que la BD esté lista | Solo espera a que arranque el contenedor. | `condition: service_healthy` con `healthcheck`. |
| Los datos desaparecen | Se escribieron fuera de un volumen o se usó `down -v`. | Volúmenes con nombre; cuidado con `-v`. |

---

## 18. Buenas prácticas

- **Multietapa**: herramientas de construcción en una etapa; en la final, solo el entorno virtual y el código.
- Base `python:3.X-slim` fijada (versión o *digest*) y reconstrucción periódica para recibir parches.
- Instalación **desde el archivo de bloqueo** (`uv sync --locked`), con caché de BuildKit.
- Ordene las instrucciones de menos a más cambiante y mantenga un `.dockerignore` completo.
- Ejecute como **usuario sin privilegios**, con `CMD` en forma exec y `HEALTHCHECK`.
- **Una imagen para todos los entornos**: configuración por variables de entorno, secretos por montajes.
- Nunca incluya secretos en la imagen (ni en `ENV`, `ARG` o capas intermedias).
- Escanee las imágenes en CI, genere SBOM y procedencia, y firme las de producción.
- Registros a stdout en formato estructurado; un proceso principal por contenedor.
- Use Compose para desarrollo y pruebas locales; `depends_on` con `service_healthy`.
- Despliegue por etiqueta inmutable o *digest*, nunca `latest`.

---

## 19. Referencia rápida

```bash
docker build -t img:tag [--target etapa] [--build-arg K=V] [--secret id=x,env=Y] .
docker run --rm [-d] [-it] [-p 8000:8000] [-e K=V] [--env-file f] [-v vol:/ruta] [--name n] img:tag
docker ps [-a] · logs -f n · exec -it n sh · stop n · rm n · inspect n
docker images · rmi img · tag img nueva · push img · pull img · history img
docker volume ls/create/rm · docker system df · docker system prune
docker compose up -d [--build] [--wait] · down [-v] · ps · logs -f s · exec s sh · run --rm s cmd · watch · config
docker buildx build --platform linux/amd64,linux/arm64 --push -t img:tag .
docker scout cves img · trivy image img
```

```dockerfile
FROM python:3.13-slim AS etapa · WORKDIR /app · COPY --from=etapa --chown=app:app a b
RUN --mount=type=cache,target=/root/.cache/uv uv sync --locked
ENV PYTHONUNBUFFERED=1 · USER app · EXPOSE 8000 · HEALTHCHECK CMD [...] · CMD ["exec", "form"]
```

---

## 20. Ejercicios propuestos

1. Construya una imagen de una aplicación propia con un `Dockerfile` simple y después con el patrón multietapa; compare el tamaño con `docker images` y las capas con `docker history`.
2. Modifique solo un archivo de código y mida el tiempo de reconstrucción antes y después de reordenar las instrucciones.
3. Cree un `compose.yaml` con la aplicación, PostgreSQL y Redis, con `healthcheck` y secretos.
4. Escanee la imagen con Docker Scout o Trivy, corrija las vulnerabilidades altas actualizando la base o las dependencias.
5. Configure GitHub Actions para construir una imagen multiplataforma, etiquetarla con la versión y publicarla en GHCR con SBOM.

---

## 21. Referencias

- Documentación oficial: https://docs.docker.com/
- Referencia del `Dockerfile`: https://docs.docker.com/reference/dockerfile/
- Buenas prácticas de construcción: https://docs.docker.com/build/building/best-practices/
- Docker Compose: https://docs.docker.com/compose/
- uv en Docker: https://docs.astral.sh/uv/guides/integration/docker/
- Imágenes oficiales de Python: https://hub.docker.com/_/python
- Hadolint: https://github.com/hadolint/hadolint
- Trivy: https://trivy.dev/
