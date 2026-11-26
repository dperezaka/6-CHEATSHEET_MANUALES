# Manual de uso: pip-audit

> **Categoría:** Seguridad · **Utilidad:** Detecta vulnerabilidades conocidas en las dependencias
> **Tipo:** Paquete externo (requiere instalación) · Proyecto de la PyPA, desarrollado con Trail of Bits y Google

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Auditar el entorno actual](#4-auditar-el-entorno-actual)
5. [Auditar archivos de requisitos y proyectos](#5-auditar-archivos-de-requisitos-y-proyectos)
6. [Auditar proyectos con uv y Poetry](#6-auditar-proyectos-con-uv-y-poetry)
7. [Leer los resultados](#7-leer-los-resultados)
8. [Formatos de salida y SBOM](#8-formatos-de-salida-y-sbom)
9. [Corregir vulnerabilidades](#9-corregir-vulnerabilidades)
10. [Ignorar vulnerabilidades de forma controlada](#10-ignorar-vulnerabilidades-de-forma-controlada)
11. [Fuentes de vulnerabilidades e índices privados](#11-fuentes-de-vulnerabilidades-e-índices-privados)
12. [Integración con pre-commit y CI](#12-integración-con-pre-commit-y-ci)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**pip-audit** analiza las **dependencias** de un proyecto Python (directas y transitivas) y las compara con bases de datos públicas de vulnerabilidades conocidas (*PyPI Advisory Database* y **OSV**). Para cada paquete vulnerable indica el identificador del aviso (PYSEC, CVE, GHSA) y las **versiones que lo corrigen**.

```text
$ pip-audit -r requirements.txt --no-deps --disable-pip
Found 5 known vulnerabilities in 1 package
Name     Version ID              Fix Versions
-------- ------- --------------- ------------
requests 2.19.0  PYSEC-2018-28   2.20.0
requests 2.19.0  PYSEC-2023-74   2.31.0
requests 2.19.0  PYSEC-2026-1873 2.32.0
requests 2.19.0  PYSEC-2026-1872 2.32.4
requests 2.19.0  PYSEC-2026-2275 2.33.0
```

**Ventajas:**
- Herramienta oficial de la PyPA, gratuita y sin cuenta.
- Audita entornos, `requirements.txt`, proyectos con `pyproject.toml` y archivos `pylock.toml`.
- Puede **corregir** automáticamente actualizando a versiones seguras.
- Genera **SBOM** (*Software Bill of Materials*) en formato CycloneDX.
- Salida JSON y Markdown para CI e informes.

**Limitaciones:**
- Solo detecta vulnerabilidades **conocidas y publicadas**: no analiza el código (use **Bandit**).
- No detecta paquetes maliciosos que aún no se han reportado.
- Necesita acceso a Internet (o a un espejo) para consultar las bases de datos.

| Si necesita… | Opción |
|---|---|
| Vulnerabilidades conocidas en dependencias Python | **pip-audit** |
| Patrones inseguros en su propio código | Bandit / Ruff `S` |
| Actualizaciones automáticas por *pull request* | Dependabot / Renovate |
| Análisis multilenguaje y de contenedores | OSV-Scanner, Trivy, Grype |

---

## 2. Instalación

```bash
uv add --dev pip-audit
pipx install pip-audit             # Como herramienta global
uvx pip-audit                      # Sin instalar
python -m pip install pip-audit
```

Comprobar:

```bash
pip-audit --version
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Vulnerabilidad conocida | Fallo de seguridad publicado con un identificador. |
| PYSEC | Identificador de la *PyPI Advisory Database* (`PYSEC-2023-74`). |
| CVE | Identificador universal (`CVE-2023-32681`). |
| GHSA | Identificador de GitHub Security Advisories. |
| OSV | Base de datos abierta de vulnerabilidades (osv.dev) que agrega varias fuentes. |
| Versión de corrección | Primera versión que soluciona la vulnerabilidad. |
| Dependencia transitiva | Dependencia de una dependencia. |
| SBOM | Inventario de componentes del software (CycloneDX). |

---

## 4. Auditar el entorno actual

Sin argumentos, pip-audit analiza los paquetes instalados en el entorno de Python activo:

```bash
source .venv/bin/activate
pip-audit
pip-audit --local               # Solo paquetes del entorno, no los del sistema
pip-audit --skip-editable       # Omite el propio proyecto instalado en modo editable
pip-audit -v                    # Más detalle
```

Auditar el entorno de otro intérprete (pip-audit instalado de forma global):

```bash
PIPAPI_PYTHON_LOCATION=.venv/bin/python pip-audit
```

> Auditar el entorno incluye lo que haya instalado a mano. Para CI, es más fiable auditar el **archivo de bloqueo** (sección 6).

---

## 5. Auditar archivos de requisitos y proyectos

```bash
pip-audit -r requirements.txt                      # Resuelve e instala en un entorno temporal
pip-audit -r requirements.txt -r requirements-dev.txt
pip-audit -r requirements.txt --no-deps            # Las versiones ya están fijadas (bloqueo completo)
pip-audit -r requirements.txt --require-hashes     # Exige hashes en todas las líneas
pip-audit -r requirements.txt --disable-pip        # No usa pip (requiere --no-deps o --require-hashes)
pip-audit .                                        # Proyecto local (lee pyproject.toml)
pip-audit --locked .                               # Lee pylock.toml del proyecto
```

| Opción | Cuándo |
|---|---|
| `--no-deps` | El archivo ya contiene **todas** las dependencias con versión exacta (generado por un bloqueo). Más rápido y preciso. |
| `--require-hashes` | Archivos con hashes: garantiza que se audita exactamente lo que se instalará. |
| `--disable-pip` | Evita crear un entorno temporal; combinado con lo anterior, la auditoría es casi instantánea. |

---

## 6. Auditar proyectos con uv y Poetry

La forma más precisa es exportar el **bloqueo** y auditarlo sin resolver de nuevo:

### 6.1 uv

```bash
uv export --format requirements-txt --no-emit-project --no-dev -o /tmp/req.txt
pip-audit -r /tmp/req.txt --no-deps --disable-pip

# En una línea
uv export --no-emit-project --no-hashes | uvx pip-audit -r /dev/stdin --no-deps --disable-pip
```

> Versiones recientes de uv incluyen `uv audit` (en *preview*), que audita el bloqueo directamente. Consulte su estado antes de sustituir pip-audit en CI.

### 6.2 Poetry

```bash
poetry self add poetry-plugin-export
poetry export -f requirements.txt --only main -o /tmp/req.txt
pip-audit -r /tmp/req.txt --no-deps --disable-pip --require-hashes
```

### 6.3 Incluir las dependencias de desarrollo

Las herramientas de desarrollo también se ejecutan en los equipos y en CI. Audítelas por separado:

```bash
uv export --no-emit-project --only-dev -o /tmp/req-dev.txt
pip-audit -r /tmp/req-dev.txt --no-deps --disable-pip
```

---

## 7. Leer los resultados

```bash
pip-audit -r req.txt --desc            # Añade la descripción de cada vulnerabilidad
pip-audit -r req.txt --aliases         # Añade CVE y GHSA equivalentes
pip-audit -r req.txt --desc --aliases -f markdown
```

```text
$ pip-audit -r requirements.txt --no-deps --disable-pip --aliases      # jinja2==3.1.2, requests==2.31.0
Found 8 known vulnerabilities in 2 packages
Name     Version ID              Fix Versions Aliases
-------- ------- --------------- ------------ -----------------------------------
jinja2   3.1.2   PYSEC-2026-1473 3.1.3        GHSA-h5c8-rqwp-cp95, CVE-2024-22195
jinja2   3.1.2   PYSEC-2026-1474 3.1.4        CVE-2024-34064, GHSA-h75v-3vvj-5mfj
jinja2   3.1.2   PYSEC-2026-1475 3.1.5        GHSA-q2x7-8rv6-6q7h, CVE-2024-56326
jinja2   3.1.2   PYSEC-2026-1472 3.1.5        CVE-2024-56201, GHSA-gmj6-6f8f-6699
jinja2   3.1.2   PYSEC-2026-1471 3.1.6        CVE-2025-27516, GHSA-cpwx-vrp4-4pq7
requests 2.31.0  PYSEC-2026-1873 2.32.0       GHSA-9wx4-h78v-vm56, CVE-2024-35195
requests 2.31.0  PYSEC-2026-1872 2.32.4       GHSA-9hjg-9r4m-mvj7, CVE-2024-47081
requests 2.31.0  PYSEC-2026-2275 2.33.0       CVE-2026-25645, GHSA-gc5v-m9x4-r6x2
```

> Algunas versiones de pip-audit repiten filas cuando un aviso llega por varias vías; el recuento puede incluir esas repeticiones. El script del ejemplo práctico las elimina. Con `--no-deps`, pip-audit también avisa de que es preferible fijar las dependencias con hashes.

| Columna | Significado |
|---|---|
| Name / Version | Paquete y versión auditada. |
| ID | Identificador principal del aviso. |
| Fix Versions | Versiones que corrigen el fallo (vacío: aún no hay corrección). |
| Aliases | Otros identificadores del mismo fallo. |

Códigos de salida:

| Código | Significado |
|---|---|
| `0` | Sin vulnerabilidades conocidas. |
| `1` | Se encontraron vulnerabilidades (o hubo un error). |

Opciones relacionadas:

```bash
pip-audit --strict           # Falla si algún paquete no se pudo auditar (p. ej. no está en PyPI)
pip-audit --progress-spinner off
pip-audit -o informe.json -f json    # Escribir a un archivo
```

---

## 8. Formatos de salida y SBOM

```bash
pip-audit -f columns                       # Por defecto
pip-audit -f json -o auditoria.json
pip-audit -f markdown -o auditoria.md
pip-audit -f cyclonedx-json -o sbom.json   # SBOM CycloneDX
pip-audit -f cyclonedx-xml -o sbom.xml
```

Estructura JSON:

```json
{
  "dependencies": [
    {
      "name": "requests",
      "version": "2.19.0",
      "vulns": [
        {
          "id": "PYSEC-2018-28",
          "fix_versions": ["2.20.0"],
          "aliases": ["CVE-2018-18074", "GHSA-x84v-xcm2-53pg"],
          "description": "The Requests package before 2.20.0 for Python sends an HTTP Authorization header..."
        }
      ]
    },
    {"name": "idna", "version": "3.10", "vulns": []}
  ],
  "fixes": []
}
```

> El **SBOM** se exige cada vez más en contratos públicos y en normativas como la *Cyber Resilience Act* europea. Generarlo en cada versión publicada es una buena práctica.

---

## 9. Corregir vulnerabilidades

### 9.1 Corrección automática

```bash
pip-audit --fix --dry-run        # Muestra qué actualizaría
pip-audit --fix                  # Actualiza los paquetes del entorno a versiones seguras
pip-audit -r requirements.txt --fix    # Modifica el archivo de requisitos
```

> `--fix` actúa sobre el entorno o el archivo de requisitos, **no** sobre `uv.lock` o `poetry.lock`. En esos proyectos, actualice con su gestor.

### 9.2 Con uv y Poetry

```bash
uv lock --upgrade-package requests          # Actualiza el bloqueo
uv add "requests>=2.32.4"                   # Si la restricción impide llegar a la versión segura

poetry update requests
poetry add "requests>=2.32.4"
```

### 9.3 Vulnerabilidad en una dependencia transitiva

```bash
uv tree --invert --package urllib3       # ¿Qué paquete la trae?
```

Opciones:

1. Actualizar el paquete que la trae, si su nueva versión exige una versión segura.
2. Añadir una **restricción** en el proyecto:

   ```toml
   [tool.uv]
   constraint-dependencies = ["urllib3>=2.5"]
   ```

3. Con pip: archivo de restricciones (`-c constraints.txt`).

### 9.4 Sin versión de corrección

Si `Fix Versions` está vacío: lea el aviso, evalúe si el código vulnerable se usa en su proyecto, aplique mitigaciones, valore alternativas y documente la decisión (sección 10).

---

## 10. Ignorar vulnerabilidades de forma controlada

```bash
pip-audit --ignore-vuln PYSEC-2023-74
pip-audit --ignore-vuln GHSA-9wx4-h78v-vm56 --ignore-vuln CVE-2024-22195
```

> Ignorar debe ser una **decisión documentada y temporal**: quién la tomó, por qué (el código vulnerable no se usa, no es explotable en su contexto…) y hasta cuándo. El ejemplo práctico incluye un archivo de excepciones con fecha de caducidad.

---

## 11. Fuentes de vulnerabilidades e índices privados

```bash
pip-audit -s pypi        # PyPI Advisory Database (por defecto)
pip-audit -s osv         # OSV (incluye avisos de GitHub y otras fuentes)
pip-audit -s esms        # Ecosyste.ms
```

| Fuente | Características |
|---|---|
| `pypi` | Consulta la API JSON de PyPI; rápida. |
| `osv` | Base agregada (PyPA, GitHub, otras); suele tener avisos antes. |

> Puede ejecutar dos auditorías (una con cada fuente) para mayor cobertura.

Índices privados y proxies:

```bash
pip-audit -r requirements.txt --index-url https://pypi.empresa.local/simple/
pip-audit -r requirements.txt --extra-index-url https://pypi.empresa.local/simple/
pip-audit --cert /etc/ssl/certs/empresa.pem
pip-audit --timeout 30
```

Los paquetes internos no aparecen en las bases públicas: pip-audit los indica como «no auditables». Use `--strict` solo si todos sus paquetes son públicos.

Caché local de resultados:

```bash
pip-audit --cache-dir ~/.cache/pip-audit
```

---

## 12. Integración con pre-commit y CI

### 12.1 pre-commit

```yaml
- repo: https://github.com/pypa/pip-audit
  rev: v2.10.1
  hooks:
    - id: pip-audit
      args: ["-r", "requirements.txt", "--no-deps", "--disable-pip"]
```

> Requiere red y tarda unos segundos: valore ejecutarlo solo cuando cambien las dependencias (`files: ^(uv\.lock|requirements.*\.txt)$`) o solo en CI.

### 12.2 GitHub Actions (acción oficial)

```yaml
- uses: actions/checkout@v4
- uses: pypa/gh-action-pip-audit@v1.1.0
  with:
    inputs: requirements.txt
```

### 12.3 GitHub Actions con uv y ejecución programada

Las vulnerabilidades se publican a diario, aunque su código no cambie: programe una auditoría periódica.

```yaml
name: Auditoría de dependencias
on:
  push:
    paths: ["uv.lock", "pyproject.toml"]
  pull_request:
  schedule:
    - cron: "17 6 * * 1-5"         # Laborables a las 06:17 UTC

jobs:
  auditoria:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
      - run: uv export --no-emit-project --no-hashes -o req.txt
      - run: uvx pip-audit -r req.txt --no-deps --disable-pip --desc -f markdown -o auditoria.md
      - if: always()
        run: cat auditoria.md >> "$GITHUB_STEP_SUMMARY"
```

### 12.4 GitLab CI

```yaml
auditoria:
  image: ghcr.io/astral-sh/uv:python3.13-bookworm-slim
  script:
    - uv export --no-emit-project --no-hashes -o req.txt
    - uvx pip-audit -r req.txt --no-deps --disable-pip -f cyclonedx-json -o sbom.json
  artifacts:
    paths: [sbom.json]
```

---

## 13. Ejemplo práctico completo

**Auditoría con política de excepciones caducables**: el script exporta el bloqueo de uv, ejecuta pip-audit, aplica un archivo de excepciones versionado (cada excepción con responsable, motivo y fecha de caducidad), falla si hay vulnerabilidades no aceptadas o excepciones caducadas, y genera un informe Markdown y un SBOM.

Archivo de excepciones (`seguridad/excepciones.toml`):

```toml
# Vulnerabilidades aceptadas temporalmente. Cada entrada caduca y debe revisarse.

[[excepcion]]
id = "PYSEC-2018-28"
paquete = "requests"
responsable = "ana.garcia@empresa.com"
motivo = "Solo se usa contra un servicio interno sin redirecciones; actualización planificada en el sprint 48."
caduca = 2026-12-31

[[excepcion]]
id = "GHSA-xxxx-yyyy-zzzz"
paquete = "libreria-ejemplo"
responsable = "luis.perez@empresa.com"
motivo = "Sin versión corregida; la función afectada no se usa."
caduca = 2026-10-01
```

Script (`scripts/auditar_dependencias.py`):

```python
"""
Audita las dependencias del proyecto con pip-audit y aplica la política de excepciones.

    python scripts/auditar_dependencias.py                     # Bloqueo de uv (uv.lock)
    python scripts/auditar_dependencias.py -r requirements.txt # Archivo de requisitos
    python scripts/auditar_dependencias.py --sbom sbom.json --informe auditoria.md
"""
from __future__ import annotations

import argparse
import json
import subprocess
import sys
import tempfile
import tomllib
from dataclasses import dataclass
from datetime import date
from pathlib import Path

EXCEPCIONES = Path("seguridad/excepciones.toml")


@dataclass(frozen=True)
class Vulnerabilidad:
    paquete: str
    version: str
    id: str
    alias: tuple[str, ...]
    correccion: tuple[str, ...]
    descripcion: str

    @property
    def identificadores(self) -> set[str]:
        return {self.id, *self.alias}


@dataclass(frozen=True)
class Excepcion:
    id: str
    paquete: str
    responsable: str
    motivo: str
    caduca: date

    @property
    def vigente(self) -> bool:
        return self.caduca >= date.today()


def exportar_bloqueo() -> Path:
    destino = Path(tempfile.mkdtemp()) / "requisitos.txt"
    subprocess.run(["uv", "export", "--no-emit-project", "--no-hashes", "--quiet", "-o", str(destino)],
                   check=True)
    return destino


def ejecutar_pip_audit(requisitos: Path, sbom: Path | None) -> list[Vulnerabilidad]:
    base = [sys.executable, "-m", "pip_audit", "-r", str(requisitos), "--no-deps", "--disable-pip",
            "--desc", "--aliases", "--progress-spinner", "off"]
    proceso = subprocess.run([*base, "-f", "json"], capture_output=True, text=True)
    if not proceso.stdout.strip():
        sys.exit(f"pip-audit falló:\n{proceso.stderr}")
    if sbom:
        subprocess.run([*base, "-f", "cyclonedx-json", "-o", str(sbom)], capture_output=True, check=False)

    vistas: set[tuple[str, str]] = set()
    resultado = []
    for dep in json.loads(proceso.stdout)["dependencies"]:
        for v in dep.get("vulns", []):
            if (dep["name"], v["id"]) in vistas:
                continue
            vistas.add((dep["name"], v["id"]))
            resultado.append(Vulnerabilidad(dep["name"], dep["version"], v["id"], tuple(v.get("aliases", [])),
                                            tuple(v.get("fix_versions", [])), v.get("description", "")))
    return resultado


def cargar_excepciones() -> list[Excepcion]:
    if not EXCEPCIONES.exists():
        return []
    datos = tomllib.loads(EXCEPCIONES.read_text(encoding="utf-8"))
    return [Excepcion(**e) for e in datos.get("excepcion", [])]


def informe_markdown(pendientes: list[Vulnerabilidad], aceptadas: list[tuple[Vulnerabilidad, Excepcion]],
                     caducadas: list[Excepcion]) -> str:
    lineas = ["## 🛡️ Auditoría de dependencias\n",
              f"- Vulnerabilidades sin aceptar: **{len(pendientes)}**",
              f"- Aceptadas temporalmente: **{len(aceptadas)}**",
              f"- Excepciones caducadas: **{len(caducadas)}**\n"]
    if pendientes:
        lineas += ["### Vulnerabilidades a corregir\n", "| Paquete | Versión | Aviso | Corregida en |",
                   "|---|---|---|---|"]
        lineas += [f"| {v.paquete} | {v.version} | {v.id} ({', '.join(v.alias) or '-'}) "
                   f"| {', '.join(v.correccion) or 'sin corrección'} |" for v in pendientes]
    if aceptadas:
        lineas += ["\n### Aceptadas temporalmente\n", "| Paquete | Aviso | Responsable | Caduca | Motivo |",
                   "|---|---|---|---|---|"]
        lineas += [f"| {v.paquete} | {v.id} | {e.responsable} | {e.caduca} | {e.motivo} |" for v, e in aceptadas]
    if caducadas:
        lineas += ["\n### Excepciones caducadas (revisar)\n"]
        lineas += [f"- {e.id} ({e.paquete}) — caducó el {e.caduca}, responsable {e.responsable}" for e in caducadas]
    return "\n".join(lineas) + "\n"


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("-r", "--requisitos", type=Path, help="Archivo de requisitos (por defecto, uv.lock)")
    parser.add_argument("--sbom", type=Path, help="Genera un SBOM CycloneDX en esta ruta")
    parser.add_argument("--informe", type=Path, help="Genera un informe Markdown en esta ruta")
    args = parser.parse_args()

    requisitos = args.requisitos or exportar_bloqueo()
    vulnerabilidades = ejecutar_pip_audit(requisitos, args.sbom)
    excepciones = cargar_excepciones()
    caducadas = [e for e in excepciones if not e.vigente]
    vigentes = [e for e in excepciones if e.vigente]

    pendientes, aceptadas = [], []
    for v in vulnerabilidades:
        excepcion = next((e for e in vigentes if e.id in v.identificadores and e.paquete == v.paquete), None)
        (aceptadas.append((v, excepcion)) if excepcion else pendientes.append(v))

    for v in pendientes:
        correccion = ", ".join(v.correccion) or "sin corrección publicada"
        print(f"✖ {v.paquete} {v.version}: {v.id} → actualizar a {correccion}")
    for v, e in aceptadas:
        print(f"· {v.paquete} {v.version}: {v.id} aceptada hasta {e.caduca} ({e.responsable})")
    for e in caducadas:
        print(f"⚠ Excepción caducada: {e.id} ({e.paquete}), caducó el {e.caduca}")

    if args.informe:
        args.informe.write_text(informe_markdown(pendientes, aceptadas, caducadas), encoding="utf-8")

    total = len(pendientes) + len(caducadas)
    print(f"\n{len(vulnerabilidades)} vulnerabilidades · {len(aceptadas)} aceptadas · "
          f"{len(pendientes)} pendientes · {len(caducadas)} excepciones caducadas")
    return 1 if total else 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Uso y salida real (noviembre de 2026, con un `requirements.txt` que fija `requests==2.19.0` e `idna==2.7`):

```bash
python scripts/auditar_dependencias.py -r requirements.txt --informe auditoria.md --sbom sbom.json
```

```text
✖ requests 2.19.0: PYSEC-2023-74 → actualizar a 2.31.0
✖ requests 2.19.0: PYSEC-2026-1873 → actualizar a 2.32.0
✖ requests 2.19.0: PYSEC-2026-1872 → actualizar a 2.32.4
✖ requests 2.19.0: PYSEC-2026-2275 → actualizar a 2.33.0
✖ idna 2.7: PYSEC-2024-60 → actualizar a 3.7
✖ idna 2.7: PYSEC-2026-215 → actualizar a 3.15
· requests 2.19.0: PYSEC-2018-28 aceptada hasta 2026-12-31 (ana.garcia@empresa.com)
⚠ Excepción caducada: GHSA-xxxx-yyyy-zzzz (libreria-ejemplo), caducó el 2026-10-01

7 vulnerabilidades · 1 aceptadas · 6 pendientes · 1 excepciones caducadas
```

> La lista concreta de avisos cambia con el tiempo: cada día se publican vulnerabilidades nuevas.

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `--disable-pip requires --no-deps or --require-hashes` | Combinación de opciones incompleta. | Añada `--no-deps` (con bloqueo completo) o `--require-hashes`. |
| `Dependency not found on PyPI and could not be audited` | Paquete interno o instalado desde Git. | Normal en paquetes privados; omítalo con `--skip-editable` o evite `--strict`. |
| Tarda mucho con `-r` | Resuelve e instala en un entorno temporal. | Exporte un bloqueo completo y use `--no-deps --disable-pip`. |
| Resultados distintos entre ejecuciones | Se publicaron avisos nuevos o cambió la fuente. | Es esperado; fije la fuente (`-s`) si necesita comparar. |
| `ConnectionError` / tiempo de espera | Sin acceso a PyPI u osv.dev. | Configure proxy, `--cert` o `--timeout`. |
| Audita el entorno equivocado | pip-audit instalado en otro entorno. | Use `-r` con el bloqueo, o `PIPAPI_PYTHON_LOCATION`. |
| `--fix` no cambia `uv.lock` | `--fix` solo modifica entornos y `requirements.txt`. | Actualice con `uv lock --upgrade-package`. |
| Avisos duplicados con IDs distintos | El mismo fallo publicado en varias bases. | Use `--aliases` para identificarlos. |

---

## 15. Buenas prácticas

- Audite el **archivo de bloqueo** (no el entorno de un desarrollador) con `--no-deps --disable-pip`.
- Ejecute la auditoría en CI **en cada cambio de dependencias** y además de forma **programada** (diaria).
- Audite también las dependencias de desarrollo: se ejecutan en equipos y servidores de CI.
- Documente cada vulnerabilidad aceptada con responsable, motivo y **fecha de caducidad**.
- Combine pip-audit con Dependabot o Renovate para recibir las actualizaciones como *pull requests*.
- Genere y archive un **SBOM** CycloneDX con cada versión publicada.
- Use `--desc --aliases` en los informes para facilitar la evaluación.
- Complemente con Bandit (código propio), un detector de secretos y el escaneo de la imagen Docker.

---

## 16. Referencia rápida

```bash
pip-audit                                     # Entorno actual
pip-audit -r req.txt [--no-deps] [--disable-pip] [--require-hashes]
pip-audit .  ·  pip-audit --locked .
uv export --no-emit-project --no-hashes | pip-audit -r /dev/stdin --no-deps --disable-pip
pip-audit --desc --aliases -f columns|json|markdown|cyclonedx-json|cyclonedx-xml -o archivo
pip-audit --fix [--dry-run]
pip-audit --ignore-vuln ID
pip-audit -s pypi|osv  ·  --index-url URL  ·  --strict  ·  --skip-editable
```

---

## 17. Ejercicios propuestos

1. Cree un `requirements.txt` con versiones antiguas de `requests`, `jinja2` y `urllib3` y audítelo con ambas fuentes (`pypi` y `osv`). Compare los resultados.
2. Corrija las vulnerabilidades encontradas con `--fix --dry-run` y después con su gestor de dependencias.
3. Localice con `uv tree --invert` qué paquete trae una dependencia transitiva vulnerable y fuércela a una versión segura con `constraint-dependencies`.
4. Programe una auditoría diaria en GitHub Actions que publique el informe en el resumen de la ejecución.
5. Genere un SBOM CycloneDX y ábralo en una herramienta de visualización (p. ej. CycloneDX Sunshine).

---

## 18. Referencias

- Repositorio y documentación: https://github.com/pypa/pip-audit
- Acción de GitHub: https://github.com/pypa/gh-action-pip-audit
- PyPI Advisory Database: https://github.com/pypa/advisory-database
- OSV: https://osv.dev/
- CycloneDX (SBOM): https://cyclonedx.org/
- Guía de seguridad de la cadena de suministro (OpenSSF): https://openssf.org/
