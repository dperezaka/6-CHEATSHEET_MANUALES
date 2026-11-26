# Manual de uso: Bandit

> **Categoría:** Seguridad · **Utilidad:** Detecta vulnerabilidades en tu código
> **Tipo:** Paquete externo (requiere instalación) · Proyecto de PyCQA

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos](#4-primeros-pasos)
5. [Severidad y confianza](#5-severidad-y-confianza)
6. [Formatos de salida](#6-formatos-de-salida)
7. [Configuración](#7-configuración)
8. [Supresión de hallazgos (`# nosec`)](#8-supresión-de-hallazgos-nosec)
9. [Línea base](#9-línea-base)
10. [Catálogo de comprobaciones](#10-catálogo-de-comprobaciones)
11. [Hallazgos frecuentes y cómo corregirlos](#11-hallazgos-frecuentes-y-cómo-corregirlos)
12. [Bandit y las reglas `S` de Ruff](#12-bandit-y-las-reglas-s-de-ruff)
13. [Integración con pre-commit y CI](#13-integración-con-pre-commit-y-ci)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

**Bandit** es una herramienta de **análisis estático de seguridad** (SAST) para código Python. Recorre el árbol sintáctico (AST) de cada archivo y aplica un conjunto de comprobaciones que detectan patrones inseguros habituales:

- contraseñas y claves escritas en el código;
- inyección de comandos (`subprocess` con `shell=True`, `os.system`);
- inyección SQL por concatenación de cadenas;
- deserialización insegura (`pickle`, `yaml.load`);
- criptografía débil (MD5, SHA-1, `random` para tokens);
- verificación TLS desactivada, peticiones sin tiempo de espera;
- `eval`, `exec`, `assert` usado para controles de seguridad…

**Ventajas:**
- Específico para Python, sencillo de ejecutar e integrar.
- Clasifica cada hallazgo por **severidad** y **confianza**.
- Salida en formatos estándar (JSON, SARIF) para CI y paneles de seguridad.
- Admite línea base para adoptarlo en proyectos existentes.

**Limitaciones:**
- Análisis por patrones dentro de cada archivo: no sigue el flujo de datos entre funciones ni módulos.
- Produce falsos positivos (revise cada hallazgo) y no detecta todos los fallos lógicos.
- No analiza dependencias (use **pip-audit**) ni secretos en el historial de Git (use gitleaks o similares).

| Si necesita… | Opción |
|---|---|
| Patrones inseguros en el código propio | **Bandit** (o reglas `S` de Ruff) |
| Vulnerabilidades conocidas en dependencias | pip-audit |
| Análisis de flujo de datos más profundo | Semgrep, CodeQL |
| Secretos en el repositorio y su historial | gitleaks, trufflehog |

---

## 2. Instalación

```bash
uv add --dev "bandit[toml]"           # [toml]: leer configuración de pyproject.toml
python -m pip install "bandit[toml]"
pipx install bandit
```

Extras disponibles:

| Extra | Habilita |
|---|---|
| `toml` | Configuración en `pyproject.toml` (necesario en Python < 3.11). |
| `sarif` | Salida en formato SARIF (GitHub Code Scanning). |
| `baseline` | Utilidad `bandit-baseline` para comparar con una rama. |

Comprobar:

```bash
bandit --version
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Comprobación (*test*) | Regla con un identificador: `B602` (subprocess con `shell=True`). |
| Hallazgo (*issue*) | Coincidencia de una comprobación en una línea de código. |
| Severidad | Gravedad potencial: `LOW`, `MEDIUM`, `HIGH`. |
| Confianza | Certeza de que es un problema real: `LOW`, `MEDIUM`, `HIGH`. |
| CWE | Identificador del catálogo *Common Weakness Enumeration* asociado. |
| Perfil | Conjunto de comprobaciones a ejecutar u omitir. |
| `# nosec` | Comentario que marca un hallazgo como revisado y aceptado. |
| Línea base | Resultado guardado; solo se informan los hallazgos nuevos. |

---

## 4. Primeros pasos

```bash
bandit -r src/                       # Analiza la carpeta de forma recursiva
bandit archivo.py
bandit -r . -x ./tests,./.venv       # Excluye rutas
bandit -r src -n 3                   # Muestra 3 líneas de contexto
bandit -r src -q                     # Solo resultados (sin cabecera ni progreso)
bandit -r src --exit-zero            # No falla aunque haya hallazgos
```

Ejemplo:

```python
# app.py
import subprocess

import requests

API_PASSWORD = "sk-live-123456"


def descargar(url: str) -> bytes:
    return requests.get(url, verify=False).content


def comprimir(nombre: str) -> None:
    subprocess.call(f"zip -r {nombre}.zip {nombre}", shell=True)
```

```text
$ bandit app.py
>> Issue: [B404:blacklist] Consider possible security implications associated with the subprocess module.
   Severity: Low   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   Location: ./app.py:2:0
>> Issue: [B105:hardcoded_password_string] Possible hardcoded password: 'sk-live-123456'
   Severity: Low   Confidence: Medium
   CWE: CWE-259 (https://cwe.mitre.org/data/definitions/259.html)
   Location: ./app.py:6:15
>> Issue: [B501:request_with_no_cert_validation] Call to requests with verify=False disabling SSL certificate checks, security issue.
   Severity: High   Confidence: High
   CWE: CWE-295 (https://cwe.mitre.org/data/definitions/295.html)
   Location: ./app.py:10:11
>> Issue: [B113:request_without_timeout] Call to requests without timeout
   Severity: Medium   Confidence: Low
   CWE: CWE-400 (https://cwe.mitre.org/data/definitions/400.html)
   Location: ./app.py:10:11
>> Issue: [B602:subprocess_popen_with_shell_equals_true] subprocess call with shell=True identified, security issue.
   Severity: High   Confidence: High
   CWE: CWE-78 (https://cwe.mitre.org/data/definitions/78.html)
   Location: ./app.py:14:4

Run metrics:
	Total issues (by severity):  Low: 2  Medium: 1  High: 2
	Total issues (by confidence): Low: 1  Medium: 1  High: 3
```

> El código de salida es **1** si hay hallazgos (dentro de los umbrales configurados) y **0** si no.

---

## 5. Severidad y confianza

Filtrar por nivel mínimo:

```bash
bandit -r src -l          # Severidad ≥ LOW (todas)
bandit -r src -ll         # Severidad ≥ MEDIUM
bandit -r src -lll        # Solo HIGH
bandit -r src -i / -ii / -iii                 # Lo mismo para la confianza
bandit -r src --severity-level medium --confidence-level high
```

Matriz orientativa de actuación:

| | Confianza ALTA | Confianza MEDIA | Confianza BAJA |
|---|---|---|---|
| **Severidad ALTA** | Corregir de inmediato | Corregir | Revisar |
| **Severidad MEDIA** | Corregir | Revisar | Revisar |
| **Severidad BAJA** | Revisar | Revisar o aceptar | Aceptar con justificación |

> Una política habitual en CI es **bloquear** severidad media o alta con confianza media o alta, e informar del resto.

---

## 6. Formatos de salida

```bash
bandit -r src -f json -o bandit.json
bandit -r src -f sarif -o bandit.sarif         # Requiere bandit[sarif]
bandit -r src -f html -o bandit.html
bandit -r src -f csv -o bandit.csv
bandit -r src -f xml -o bandit.xml             # JUnit
bandit -r src -f screen                        # Por defecto en terminal
bandit -r src -f custom --msg-template "{abspath}:{line}: {test_id} [{severity}] {msg}"
```

Estructura JSON de un hallazgo:

```json
{
  "filename": "src/app.py",
  "line_number": 13,
  "test_id": "B602",
  "test_name": "subprocess_popen_with_shell_equals_true",
  "issue_severity": "HIGH",
  "issue_confidence": "HIGH",
  "issue_cwe": {"id": 78, "link": "https://cwe.mitre.org/data/definitions/78.html"},
  "issue_text": "subprocess call with shell=True identified, security issue.",
  "code": "13     subprocess.call(f\"zip -r {nombre}.zip {nombre}\", shell=True)\n",
  "more_info": "https://bandit.readthedocs.io/en/latest/plugins/b602_subprocess_popen_with_shell_equals_true.html"
}
```

---

## 7. Configuración

### 7.1 En `pyproject.toml`

```toml
[tool.bandit]
exclude_dirs = ["tests", ".venv", "build", "migraciones"]
skips = ["B101"]                 # assert (habitual en pruebas)
# tests = ["B201", "B301"]       # Ejecutar solo estas comprobaciones

[tool.bandit.any_other_function_with_shell_equals_true]
no_shell = ["os.execl", "os.execle", "os.execv", "os.spawnl"]
```

> Debe indicar el archivo explícitamente: `bandit -c pyproject.toml -r src`.

### 7.2 En YAML (`.bandit.yml` o `bandit.yaml`)

```yaml
exclude_dirs: ["tests", ".venv"]
skips: ["B101", "B311"]

hardcoded_tmp_directory:
  tmp_dirs: ["/tmp", "/var/tmp", "/dev/shm"]

assert_used:
  skips: ["*/tests/*", "*/test_*.py"]
```

```bash
bandit -c .bandit.yml -r src
bandit-config-generator -o .bandit.yml    # Genera una plantilla con todas las opciones
```

### 7.3 Opciones de línea de comandos equivalentes

```bash
bandit -r src -s B101,B311              # Omitir comprobaciones
bandit -r src -t B602,B608              # Solo estas
bandit -r src -x "./tests/*,./scripts/*"
```

---

## 8. Supresión de hallazgos (`# nosec`)

Cuando un hallazgo es un falso positivo o un riesgo aceptado, márquelo **indicando la comprobación y el motivo**:

```python
import subprocess

# El comando es fijo y no incluye datos del usuario
subprocess.run(["git", "rev-parse", "HEAD"], check=True)    # (no genera hallazgo: sin shell)

resultado = subprocess.run("ls -la /var/log", shell=True)    # nosec B602 - comando fijo sin entrada externa

hash_cache = hashlib.md5(contenido, usedforsecurity=False)   # Mejor que nosec: indica la intención
```

| Forma | Efecto |
|---|---|
| `# nosec B602` | Suprime solo B602 en esa línea (**recomendado**). |
| `# nosec B602, B607` | Varias comprobaciones. |
| `# nosec` | Suprime todo en la línea (evítelo: oculta hallazgos futuros). |

Auditar las supresiones:

```bash
bandit -r src --ignore-nosec           # Ignora los nosec y muestra todo
grep -rn "nosec" src/                  # Revise periódicamente cada supresión
```

---

## 9. Línea base

Para adoptar Bandit en un proyecto con hallazgos existentes sin bloquear al equipo:

```bash
bandit -r src -f json -o .bandit-baseline.json --exit-zero    # Guardar la situación actual
bandit -r src -b .bandit-baseline.json                        # Solo informa de hallazgos nuevos
```

> La línea base compara por archivo, comprobación y código afectado. Si modifica una línea con un hallazgo, este puede volver a aparecer como nuevo: es intencionado, para que se revise.

Comparar con la rama principal (extra `baseline`):

```bash
bandit-baseline -r src -ii -ll     # Compara HEAD con el commit anterior
```

---

## 10. Catálogo de comprobaciones

| Rango | Categoría | Ejemplos |
|---|---|---|
| B1xx | Varios | B101 `assert`, B102 `exec`, B103 permisos inseguros, B104 escuchar en `0.0.0.0`, B105-B107 contraseñas fijas, B108 directorio temporal fijo, B110/B112 `try/except: pass/continue`, B113 peticiones sin *timeout* |
| B2xx | Configuración de aplicaciones | B201 Flask con `debug=True`, B202 `tarfile.extractall` sin filtro |
| B3xx | Llamadas a funciones inseguras | B301 `pickle`, B303 MD5/SHA1 (obsoleta, ahora B324), B306 `mktemp`, B307 `eval`, B310 `urlopen` con esquemas arbitrarios, B311 `random`, B314-B320 XML inseguro, B324 hash débil |
| B4xx | Importaciones de módulos inseguros | B403 `pickle`, B404 `subprocess`, B405-B411 XML, B413 `pycrypto` |
| B5xx | Criptografía y TLS | B501 `verify=False`, B502-B504 versiones SSL/TLS antiguas, B505 claves débiles, B506 `yaml.load`, B507 SSH sin verificar el *host*, B508/B509 SNMP inseguro |
| B6xx | Inyección | B601 *paramiko*, B602-B607 *subprocess* y *shell*, B608 SQL por concatenación, B609 comodines, B610/B611 Django `extra`/`RawSQL`, B612 `logging.config.listen`, B613 caracteres bidireccionales (*trojan source*), B614 `torch.load` |
| B7xx | Plantillas y XSS | B701 Jinja2 con `autoescape=False`, B702 Mako, B703 Django `mark_safe`, B704 `markupsafe.Markup` |

Listado completo con su versión instalada:

```bash
bandit -h          # Al final de la ayuda aparecen todas las comprobaciones
```

---

## 11. Hallazgos frecuentes y cómo corregirlos

| Código | Inseguro | Seguro |
|---|---|---|
| B105/B106 | `CLAVE = "abc123"` | `CLAVE = os.environ["CLAVE"]` o pydantic-settings con `SecretStr` |
| B602 | `subprocess.run(f"convert {f}", shell=True)` | `subprocess.run(["convert", f], check=True)` |
| B605 | `os.system("rm " + ruta)` | `Path(ruta).unlink()` |
| B608 | `cur.execute(f"SELECT * FROM t WHERE id = {id_}")` | `cur.execute("SELECT * FROM t WHERE id = ?", (id_,))` |
| B301 | `pickle.loads(datos_recibidos)` | `json.loads(datos_recibidos)` + validación con Pydantic |
| B506 | `yaml.load(texto)` | `yaml.safe_load(texto)` |
| B307 | `eval(expresion)` | `ast.literal_eval(expresion)` o un analizador propio |
| B311 | `random.choice(caracteres)` para *tokens* | `secrets.token_urlsafe(32)` |
| B324 | `hashlib.md5(clave)` | `hashlib.sha256(...)`; para contraseñas, `argon2`/`bcrypt`; si no es seguridad, `usedforsecurity=False` |
| B501 | `requests.get(url, verify=False)` | `requests.get(url, timeout=10)` (o `verify="/ruta/ca.pem"`) |
| B113 | `requests.get(url)` | `requests.get(url, timeout=10)` |
| B108 | `open("/tmp/datos.txt", "w")` | `tempfile.NamedTemporaryFile()` / `tempfile.mkdtemp()` |
| B104 | `app.run(host="0.0.0.0")` | Escuchar en `127.0.0.1` salvo que se necesite (contenedores) |
| B201 | `app.run(debug=True)` | `debug` controlado por configuración y desactivado en producción |
| B701 | `Environment(autoescape=False)` | `Environment(autoescape=select_autoescape())` |
| B202 | `tar.extractall(destino)` | `tar.extractall(destino, filter="data")` |
| B101 | `assert usuario.es_admin` | `if not usuario.es_admin: raise PermissionError` (los `assert` se eliminan con `-O`) |

---

## 12. Bandit y las reglas `S` de Ruff

Ruff implementa la mayoría de las comprobaciones de Bandit como reglas `S` (`flake8-bandit`): `S105` equivale a `B105`, `S602` a `B602`, `S608` a `B608`, etc.

| | Bandit | Ruff `S` |
|---|---|---|
| Velocidad | Buena | Muy alta |
| Severidad y confianza | Sí | No (todas son avisos) |
| Formatos SARIF/HTML | Sí | SARIF, JSON, GitLab… |
| Línea base | Sí | Mediante `--add-noqa` |
| Integración en el editor | Limitada | Sí (servidor de lenguaje) |
| Cobertura de comprobaciones | Completa | Casi completa |

Estrategia recomendada:

- **Ruff `S`** en el editor y en pre-commit: respuesta inmediata al escribir.
- **Bandit** en CI: informe con severidad/confianza en SARIF para el panel de seguridad y auditorías.

Las supresiones son distintas: Ruff usa `# noqa: S602` y Bandit `# nosec B602`. Si usa ambos, puede necesitar las dos en la misma línea.

---

## 13. Integración con pre-commit y CI

### 13.1 pre-commit

```yaml
- repo: https://github.com/PyCQA/bandit
  rev: 1.9.4
  hooks:
    - id: bandit
      args: ["-c", "pyproject.toml", "-ll", "-ii"]
      additional_dependencies: ["bandit[toml]"]
```

### 13.2 GitHub Actions con *Code Scanning* (SARIF)

```yaml
name: Seguridad
on: [push, pull_request]

jobs:
  bandit:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
      - run: uvx --from "bandit[sarif,toml]" bandit -c pyproject.toml -r src -f sarif -o bandit.sarif --exit-zero
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: bandit.sarif
      - run: uvx --from "bandit[toml]" bandit -c pyproject.toml -r src -ll -ii    # Barrera
```

Los hallazgos aparecen en la pestaña **Security → Code scanning** y como comentarios en las *pull requests*.

### 13.3 GitLab CI

GitLab incluye Bandit en su plantilla SAST:

```yaml
include:
  - template: Jobs/SAST.gitlab-ci.yml
```

---

## 14. Ejemplo práctico completo

**Auditoría de seguridad de un servicio** con código vulnerable, su versión corregida y un script de CI que ejecuta Bandit, aplica una política (bloquear severidad media o alta con confianza media o alta), respeta una línea base y genera un resumen Markdown para la *pull request*.

### 14.1 Código vulnerable (`src/servicio/exportador.py`)

```python
import hashlib
import os
import pickle
import random
import sqlite3
import string
import subprocess

import requests
import yaml

DB_PASSWORD = "Sup3rS3creta!"


def cargar_config(ruta):
    with open(ruta) as f:
        return yaml.load(f, Loader=yaml.Loader)


def buscar_cliente(conexion: sqlite3.Connection, nombre: str):
    consulta = "SELECT * FROM clientes WHERE nombre = '%s'" % nombre
    return conexion.execute(consulta).fetchall()


def generar_token() -> str:
    return "".join(random.choice(string.ascii_letters) for _ in range(32))


def huella(contenido: bytes) -> str:
    return hashlib.md5(contenido).hexdigest()


def comprimir(carpeta: str) -> None:
    os.system(f"tar czf /tmp/{carpeta}.tgz {carpeta}")


def enviar(url: str, datos: dict) -> None:
    requests.post(url, json=datos, verify=False)


def restaurar(datos: bytes):
    return pickle.loads(datos)


def ejecutar(comando: str) -> str:
    return subprocess.check_output(comando, shell=True, text=True)
```

```text
$ bandit -q -r src/servicio/exportador.py -f custom --msg-template "{line}: {test_id} [{severity}/{confidence}] {msg}"
3: B403 [LOW/HIGH] Consider possible security implications associated with pickle module.
7: B404 [LOW/HIGH] Consider possible security implications associated with the subprocess module.
12: B105 [LOW/MEDIUM] Possible hardcoded password: 'Sup3rS3creta!'
17: B506 [MEDIUM/HIGH] Use of unsafe yaml load. Allows instantiation of arbitrary objects. Consider yaml.safe_load().
21: B608 [MEDIUM/LOW] Possible SQL injection vector through string-based query construction.
26: B311 [LOW/HIGH] Standard pseudo-random generators are not suitable for security/cryptographic purposes.
30: B324 [HIGH/HIGH] Use of weak MD5 hash for security. Consider usedforsecurity=False
34: B605 [HIGH/HIGH] Starting a process with a shell, possible injection detected, security issue.
38: B501 [HIGH/HIGH] Call to requests with verify=False disabling SSL certificate checks, security issue.
38: B113 [MEDIUM/LOW] Call to requests without timeout
42: B301 [MEDIUM/HIGH] Pickle and modules that wrap it can be unsafe when used to deserialize untrusted data, possible security issue.
46: B602 [HIGH/HIGH] subprocess call with shell=True identified, security issue.
```

### 14.2 Código corregido

```python
import hashlib
import json
import os
import secrets
import shutil
import sqlite3
import tempfile
from pathlib import Path

import httpx
import yaml
from pydantic import BaseModel

DB_PASSWORD = os.environ["DB_PASSWORD"]                         # B105: el secreto viene del entorno


def cargar_config(ruta: Path) -> dict:
    with ruta.open(encoding="utf-8") as f:
        return yaml.safe_load(f)                                 # B506


def buscar_cliente(conexion: sqlite3.Connection, nombre: str) -> list[tuple]:
    return conexion.execute(
        "SELECT * FROM clientes WHERE nombre = ?", (nombre,)     # B608: consulta parametrizada
    ).fetchall()


def generar_token() -> str:
    return secrets.token_urlsafe(32)                             # B311


def huella(contenido: bytes) -> str:
    return hashlib.sha256(contenido).hexdigest()                 # B324


def comprimir(carpeta: Path) -> Path:
    destino = Path(tempfile.mkdtemp()) / carpeta.name            # Directorio temporal seguro
    return Path(shutil.make_archive(str(destino), "gztar", root_dir=carpeta))   # B605: sin shell


def enviar(url: str, datos: dict) -> None:
    httpx.post(url, json=datos, timeout=10).raise_for_status()   # B501, B113


class Estado(BaseModel):
    version: int
    elementos: list[str]


def restaurar(datos: bytes) -> Estado:
    return Estado.model_validate(json.loads(datos))              # B301: JSON validado, no pickle
```

```text
$ bandit -q -r src/servicio/exportador.py
(sin hallazgos)
```

> Se elimina `ejecutar()`: una función que ejecuta comandos arbitrarios no se corrige, se rediseña (lista blanca de operaciones concretas).

### 14.3 Política de CI (`scripts/politica_bandit.py`)

```python
"""
Ejecuta Bandit, aplica la política de seguridad del proyecto y genera un resumen Markdown.

    python scripts/politica_bandit.py src               # Comprueba
    python scripts/politica_bandit.py src --resumen resumen.md
"""
from __future__ import annotations

import argparse
import json
import subprocess
import sys
from collections import Counter
from pathlib import Path

NIVELES = {"LOW": 1, "MEDIUM": 2, "HIGH": 3}
SEVERIDAD_MINIMA = "MEDIUM"      # Bloquea severidad >= MEDIUM…
CONFIANZA_MINIMA = "MEDIUM"      # …con confianza >= MEDIUM
LINEA_BASE = Path(".bandit-baseline.json")


def ejecutar_bandit(rutas: list[str]) -> dict:
    cmd = [sys.executable, "-m", "bandit", "-r", *rutas, "-f", "json", "-q", "--exit-zero"]
    if Path("pyproject.toml").exists():
        cmd[3:3] = ["-c", "pyproject.toml"]
    if LINEA_BASE.exists():
        cmd += ["-b", str(LINEA_BASE)]
    proceso = subprocess.run(cmd, capture_output=True, text=True, check=False)
    if not proceso.stdout.strip():
        sys.exit(f"Bandit no produjo resultados:\n{proceso.stderr}")
    return json.loads(proceso.stdout)


def bloquea(h: dict) -> bool:
    return (NIVELES[h["issue_severity"]] >= NIVELES[SEVERIDAD_MINIMA]
            and NIVELES[h["issue_confidence"]] >= NIVELES[CONFIANZA_MINIMA])


def resumen_markdown(hallazgos: list[dict], bloqueantes: list[dict]) -> str:
    por_sev = Counter(h["issue_severity"] for h in hallazgos)
    lineas = [
        "## 🔒 Análisis de seguridad (Bandit)\n",
        f"| Severidad | Hallazgos |\n|---|---:|",
        *(f"| {s} | {por_sev.get(s, 0)} |" for s in ("HIGH", "MEDIUM", "LOW")),
        "",
    ]
    if bloqueantes:
        lineas += [f"### ✖ {len(bloqueantes)} hallazgos bloqueantes\n",
                   "| Archivo | Línea | Regla | Sev./Conf. | Descripción |", "|---|---:|---|---|---|"]
        for h in sorted(bloqueantes, key=lambda x: (-NIVELES[x["issue_severity"]], x["filename"])):
            lineas.append(f"| `{h['filename']}` | {h['line_number']} | [{h['test_id']}]({h['more_info']}) "
                          f"| {h['issue_severity']}/{h['issue_confidence']} | {h['issue_text']} |")
    else:
        lineas.append("✔ Sin hallazgos bloqueantes.")
    return "\n".join(lineas) + "\n"


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("rutas", nargs="+")
    parser.add_argument("--resumen", type=Path, help="Archivo Markdown de salida")
    args = parser.parse_args()

    informe = ejecutar_bandit(args.rutas)
    hallazgos = informe.get("results", [])
    bloqueantes = [h for h in hallazgos if bloquea(h)]

    for h in hallazgos:
        marca = "✖" if h in bloqueantes else "·"
        print(f"{marca} {h['filename']}:{h['line_number']} {h['test_id']} "
              f"[{h['issue_severity']}/{h['issue_confidence']}] {h['issue_text']}")
    for error in informe.get("errors", []):
        print(f"⚠ No se pudo analizar {error['filename']}: {error['reason']}")

    texto = resumen_markdown(hallazgos, bloqueantes)
    if args.resumen:
        args.resumen.write_text(texto, encoding="utf-8")
    print(f"\n{len(hallazgos)} hallazgos, {len(bloqueantes)} bloqueantes "
          f"(política: severidad ≥ {SEVERIDAD_MINIMA} y confianza ≥ {CONFIANZA_MINIMA})")
    return 1 if bloqueantes else 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Integración en GitHub Actions, publicando el resumen en la página de la ejecución:

```yaml
- run: uv run python scripts/politica_bandit.py src --resumen resumen.md
- if: always()
  run: cat resumen.md >> "$GITHUB_STEP_SUMMARY"
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| La configuración de `pyproject.toml` se ignora | Bandit no lo lee por defecto. | `bandit -c pyproject.toml ...` e instale `bandit[toml]`. |
| Cientos de `B101` en las pruebas | `assert` es normal en pytest. | Excluya `tests` o omita B101 para esa ruta. |
| `Unknown test found in profile` | Identificador de comprobación inexistente o retirado. | Revise la lista con `bandit -h`. |
| `-f sarif` no disponible | Falta el extra. | `pip install "bandit[sarif]"`. |
| Se analiza `.venv` y tarda mucho | No se excluyó el entorno. | `exclude_dirs = [".venv"]` o `-x ./.venv`. |
| Un `# nosec` no tiene efecto | Comentario en otra línea de una expresión multilínea. | Colóquelo en la primera línea de la llamada. |
| Falsos positivos de B608 en consultas con parámetros | Construcción de SQL con f-strings para nombres de tabla. | Use listas blancas para identificadores y `# nosec B608` justificado. |
| `B404`/`B403` en cualquier uso de `subprocess`/`pickle` | Avisos informativos de importación (severidad baja). | Omítalos (`skips`) si la política solo bloquea niveles medio/alto. |

---

## 16. Buenas prácticas

- Ejecute Bandit en **CI en cada cambio**, con una política clara de qué bloquea.
- Complemente con las reglas `S` de Ruff en el editor para detectar los problemas al escribir.
- Excluya entornos virtuales y código generado; trate las pruebas con reglas específicas.
- Use `# nosec <código>` **siempre con código y motivo**, y revíselos periódicamente con `--ignore-nosec`.
- En proyectos existentes, empiece con una **línea base** y redúzcala progresivamente.
- Publique los resultados en SARIF para tener un histórico en el panel de seguridad.
- Combine Bandit con **pip-audit** (dependencias) y un detector de secretos (gitleaks).
- No confíe solo en herramientas: las revisiones de código y el modelado de amenazas siguen siendo necesarios.

---

## 17. Referencia rápida

```bash
bandit -r src [-c pyproject.toml] [-x ./tests] [-s B101] [-t B602,B608]
bandit -r src -ll -ii                      # Severidad y confianza ≥ MEDIUM
bandit -r src -f json|sarif|html|csv|xml|custom -o salida
bandit -r src -b .bandit-baseline.json     # Solo hallazgos nuevos
bandit -r src --ignore-nosec               # Mostrar también los suprimidos
bandit -r src --exit-zero  ·  -q  ·  -n 3
```

```toml
[tool.bandit]
exclude_dirs = ["tests", ".venv"]
skips = ["B101"]
```

```python
codigo_aceptado()  # nosec B602 - motivo
```

---

## 18. Ejercicios propuestos

1. Ejecute Bandit sobre un proyecto propio con `-ll -ii` y clasifique los hallazgos en «corregir», «revisar» y «aceptar».
2. Corrija un caso de inyección SQL (B608) usando consultas parametrizadas y compruebe que el hallazgo desaparece.
3. Configure la subida SARIF a GitHub Code Scanning y verifique que los hallazgos aparecen en una *pull request*.
4. Cree una línea base y demuestre que un hallazgo nuevo hace fallar la CI mientras los antiguos no.
5. Compare los resultados de Bandit con `ruff check --select S` sobre el mismo código y documente las diferencias.

---

## 19. Referencias

- Documentación oficial: https://bandit.readthedocs.io/
- Lista de comprobaciones: https://bandit.readthedocs.io/en/latest/plugins/index.html
- Configuración: https://bandit.readthedocs.io/en/latest/config.html
- Repositorio: https://github.com/PyCQA/bandit
- Reglas `S` de Ruff: https://docs.astral.sh/ruff/rules/#flake8-bandit-s
- CWE (catálogo de debilidades): https://cwe.mitre.org/
- OWASP Top 10: https://owasp.org/www-project-top-ten/
