# Manual de uso: python-dotenv

> **Categoría:** Utilidades generales · **Utilidad:** Gestionar variables de entorno
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Sintaxis del archivo .env](#4-sintaxis-del-archivo-env)
5. [Operaciones básicas](#5-operaciones-básicas)
6. [Conversión y validación de valores](#6-conversión-y-validación-de-valores)
7. [Varios entornos](#7-varios-entornos)
8. [Operaciones avanzadas](#8-operaciones-avanzadas)
9. [Integración con otras herramientas](#9-integración-con-otras-herramientas)
10. [Ejemplo práctico completo](#10-ejemplo-práctico-completo)
11. [Seguridad](#11-seguridad)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**python-dotenv** lee pares `CLAVE=valor` de un archivo `.env` y los carga como **variables de entorno**. Permite separar la configuración (contraseñas, tokens, URLs de bases de datos, modos de depuración) del código fuente, siguiendo la metodología *The Twelve-Factor App*.

**Ventajas:**
- Las credenciales no se escriben en el código ni se suben al repositorio.
- Cada entorno (desarrollo, pruebas, producción) usa su propia configuración sin cambiar el código.
- En producción se pueden usar variables de entorno reales del sistema sin modificar nada.

---

## 2. Instalación e importación

```bash
pip install python-dotenv
```

Con la herramienta de línea de comandos:

```bash
pip install "python-dotenv[cli]"
```

El paquete se instala como `python-dotenv`, pero se importa como `dotenv`:

```python
import os
from dotenv import load_dotenv, dotenv_values, find_dotenv, set_key, unset_key
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Variable de entorno | Valor de configuración disponible para un proceso (`os.environ`). |
| Archivo `.env` | Archivo de texto con variables `CLAVE=valor`. |
| `load_dotenv()` | Carga el `.env` en `os.environ`. |
| `dotenv_values()` | Lee el `.env` y devuelve un diccionario, **sin** modificar el entorno. |
| `override` | Indica si el `.env` sobrescribe variables ya existentes en el sistema. |
| `.env.example` | Plantilla con las claves necesarias, sin valores secretos (sí se sube al repositorio). |

> **Prioridad por defecto:** las variables reales del sistema prevalecen sobre las del archivo `.env` (`override=False`).

---

## 4. Sintaxis del archivo .env

```bash
# Comentario de línea completa
APP_NOMBRE=Mi Aplicación
APP_DEBUG=true
APP_PUERTO=8000

# Comillas (necesarias si hay espacios al inicio/final o caracteres especiales)
DB_PASSWORD="p@ss w0rd#123"
MENSAJE='Texto literal con $DOLAR sin expandir'

# Valores vacíos
API_OPCIONAL=

# Referencias a otras variables (expansión)
DB_HOST=localhost
DB_PUERTO=5432
DB_NOMBRE=empresa
DATABASE_URL=postgresql://admin:${DB_PASSWORD}@${DB_HOST}:${DB_PUERTO}/${DB_NOMBRE}

# Valor multilínea
CLAVE_PRIVADA="-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASC...
-----END PRIVATE KEY-----"

# Sintaxis compatible con shell
export ENTORNO=desarrollo

# Comentario al final de línea (requiere espacio antes de #)
TIMEOUT=30 # segundos
```

| Regla | Detalle |
|---|---|
| Formato | `CLAVE=valor`, una por línea. |
| Espacios | Se ignoran alrededor del `=`. |
| Comillas dobles | Permiten `\n`, escapes y expansión `${VAR}`. |
| Comillas simples | Valor literal, sin expansión. |
| Comentarios | `#` al inicio de línea o tras un espacio. |
| Tipos | Todos los valores se leen como **texto**. |
| Nombres | Por convención, en MAYÚSCULAS con guiones bajos. |

---

## 5. Operaciones básicas

### 5.1 Cargar el archivo `.env`

```python
import os
from dotenv import load_dotenv

load_dotenv()                            # Busca .env en el directorio actual y superiores

token = os.getenv("API_TOKEN")
debug = os.getenv("APP_DEBUG")
```

### 5.2 Leer variables

```python
os.getenv("APP_PUERTO")                  # Devuelve None si no existe
os.getenv("APP_PUERTO", "8000")          # Valor por defecto
os.environ["API_TOKEN"]                  # KeyError si no existe (útil para obligatorias)
os.environ.get("API_TOKEN")              # Equivalente a getenv
"API_TOKEN" in os.environ                # Comprobar existencia
```

### 5.3 Ruta explícita al archivo

```python
from pathlib import Path

RAIZ = Path(__file__).resolve().parent
load_dotenv(RAIZ / ".env")
```

> Indicar la ruta evita problemas cuando el script se ejecuta desde otra carpeta (tareas programadas, servicios, IDE).

### 5.4 Sobrescribir variables existentes

```python
load_dotenv(override=True)               # El .env prevalece sobre el sistema
```

### 5.5 Leer sin modificar el entorno

```python
from dotenv import dotenv_values

config = dotenv_values(".env")           # dict[str, str | None]
print(config["APP_NOMBRE"])
```

### 5.6 Comprobar si se cargó

```python
if not load_dotenv():
    print("Aviso: no se encontró el archivo .env")
```

---

## 6. Conversión y validación de valores

Todas las variables son **cadenas de texto**. Conviértalas explícitamente.

### 6.1 Funciones de conversión

```python
import os


def env_str(nombre: str, defecto: str | None = None, obligatoria: bool = False) -> str | None:
    valor = os.getenv(nombre, defecto)
    if obligatoria and not valor:
        raise RuntimeError(f"Falta la variable de entorno obligatoria: {nombre}")
    return valor


def env_bool(nombre: str, defecto: bool = False) -> bool:
    valor = os.getenv(nombre)
    if valor is None:
        return defecto
    return valor.strip().lower() in {"1", "true", "yes", "si", "sí", "on"}


def env_int(nombre: str, defecto: int) -> int:
    valor = os.getenv(nombre)
    try:
        return int(valor) if valor else defecto
    except ValueError:
        raise RuntimeError(f"{nombre} debe ser un número entero (valor actual: {valor!r})")


def env_list(nombre: str, separador: str = ",") -> list[str]:
    valor = os.getenv(nombre, "")
    return [x.strip() for x in valor.split(separador) if x.strip()]
```

Uso:

```python
DEBUG = env_bool("APP_DEBUG")
PUERTO = env_int("APP_PUERTO", 8000)
HOSTS = env_list("ALLOWED_HOSTS")              # ALLOWED_HOSTS=localhost,127.0.0.1
TOKEN = env_str("API_TOKEN", obligatoria=True)
```

> **Cuidado:** `bool("false")` es `True` en Python, porque cualquier cadena no vacía es verdadera. Use siempre una función de conversión.

### 6.2 Validar al arrancar

```python
OBLIGATORIAS = ["DATABASE_URL", "API_TOKEN", "SECRET_KEY"]
faltan = [v for v in OBLIGATORIAS if not os.getenv(v)]
if faltan:
    raise SystemExit(f"Configuración incompleta. Faltan: {', '.join(faltan)}")
```

---

## 7. Varios entornos

### 7.1 Un archivo por entorno

```
proyecto/
├── .env                 # Valores comunes / locales (no se sube)
├── .env.development     # Desarrollo
├── .env.test            # Pruebas
├── .env.production      # Producción (normalmente no existe: se usan variables del servidor)
└── .env.example         # Plantilla documentada (sí se sube)
```

```python
import os
from dotenv import load_dotenv

entorno = os.getenv("APP_ENV", "development")
load_dotenv(f".env.{entorno}")           # Específico del entorno
load_dotenv(".env")                      # Valores comunes (no sobrescribe los anteriores)
```

### 7.2 Combinar varias fuentes con prioridad

```python
from dotenv import dotenv_values

config = {
    **dotenv_values(".env.shared"),       # 1. Valores compartidos
    **dotenv_values(".env.secret"),       # 2. Secretos locales
    **os.environ,                         # 3. Variables del sistema (máxima prioridad)
}
```

### 7.3 Archivo `.env.example`

```bash
# ============================================
#  Configuración de la aplicación
#  Copie este archivo como .env y complete los valores
# ============================================

# Entorno: development | test | production
APP_ENV=development
APP_DEBUG=false
APP_PUERTO=8000

# Base de datos (obligatorio)
DATABASE_URL=postgresql://usuario:clave@localhost:5432/basedatos

# Claves de servicios externos (obligatorio)
API_TOKEN=
SECRET_KEY=

# Correo (opcional)
SMTP_HOST=
SMTP_USUARIO=
SMTP_CLAVE=
```

---

## 8. Operaciones avanzadas

### 8.1 Modificar el archivo `.env` desde código

```python
from dotenv import set_key, unset_key, get_key

set_key(".env", "ULTIMA_EJECUCION", "2026-11-24T21:45:00")
get_key(".env", "ULTIMA_EJECUCION")
unset_key(".env", "VARIABLE_OBSOLETA")
```

### 8.2 Buscar el archivo automáticamente

```python
from dotenv import find_dotenv

ruta = find_dotenv(usecwd=True)                      # Busca desde el directorio actual
ruta = find_dotenv(".env.local", raise_error_if_not_found=True)
load_dotenv(ruta)
```

### 8.3 Leer desde una cadena o flujo

```python
from io import StringIO

contenido = StringIO("CLAVE=valor\nOTRA=123")
load_dotenv(stream=contenido)
```

### 8.4 Desactivar la expansión de variables

```python
load_dotenv(interpolate=False)       # ${VAR} se conserva literalmente
```

### 8.5 Línea de comandos

```bash
dotenv list                          # Mostrar variables del .env
dotenv list --format=json
dotenv get API_TOKEN
dotenv set APP_DEBUG true
dotenv unset APP_DEBUG
dotenv run -- python script.py       # Ejecutar un programa con el .env cargado
dotenv -f .env.test run -- pytest    # Usar otro archivo
```

### 8.6 Ocultar secretos al mostrar la configuración

```python
SENSIBLES = ("PASSWORD", "CLAVE", "SECRET", "TOKEN", "KEY")

def mostrar_config(config: dict) -> None:
    for clave, valor in sorted(config.items()):
        if any(s in clave.upper() for s in SENSIBLES) and valor:
            valor = valor[:2] + "****"
        print(f"{clave:<20} = {valor}")
```

---

## 9. Integración con otras herramientas

### 9.1 Pydantic Settings (validación tipada)

```python
from pydantic import SecretStr, PostgresDsn
from pydantic_settings import BaseSettings, SettingsConfigDict


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8",
                                      env_prefix="APP_", extra="ignore")

    debug: bool = False
    puerto: int = 8000
    database_url: PostgresDsn
    api_token: SecretStr


ajustes = Ajustes()                  # Lee, convierte y valida automáticamente
```

### 9.2 Django

```python
# settings.py
from pathlib import Path
import os
from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")

SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
DEBUG = os.getenv("DJANGO_DEBUG", "false").lower() == "true"
ALLOWED_HOSTS = os.getenv("DJANGO_ALLOWED_HOSTS", "localhost").split(",")
```

### 9.3 Flask

Flask carga automáticamente `.env` y `.flaskenv` si python-dotenv está instalado y se usa el comando `flask run`.

### 9.4 FastAPI / Uvicorn

```bash
uvicorn main:app --env-file .env
```

### 9.5 Docker y Docker Compose

```yaml
# docker-compose.yml
services:
  app:
    build: .
    env_file:
      - .env
```

> En contenedores, las variables llegan al proceso directamente; `load_dotenv()` no sobrescribe esas variables, por lo que el mismo código funciona en local y en Docker.

### 9.6 Pruebas con pytest

```python
def test_modo_debug(monkeypatch):
    monkeypatch.setenv("APP_DEBUG", "true")
    assert env_bool("APP_DEBUG") is True
```

---

## 10. Ejemplo práctico completo

Módulo de configuración centralizado, tipado y validado, reutilizable en cualquier proyecto.

**`.env`**

```bash
APP_ENV=development
APP_NOMBRE=Gestor de Pedidos
APP_DEBUG=true
APP_PUERTO=8080
ALLOWED_HOSTS=localhost,127.0.0.1

DB_HOST=localhost
DB_PUERTO=5432
DB_NOMBRE=pedidos
DB_USUARIO=admin
DB_CLAVE="S3gura#2026"

SMTP_HOST=smtp.miempresa.com
SMTP_PUERTO=587
SMTP_USUARIO=notificaciones@miempresa.com
SMTP_CLAVE=clave-smtp

API_TOKEN=tok_abcdef1234567890
```

**`config.py`**

```python
"""Configuración centralizada de la aplicación."""
from __future__ import annotations

import os
from dataclasses import dataclass, field
from pathlib import Path

from dotenv import load_dotenv

RAIZ = Path(__file__).resolve().parent
ENTORNO = os.getenv("APP_ENV", "development")

# 1) Archivo específico del entorno  2) Archivo general. El sistema siempre prevalece.
load_dotenv(RAIZ / f".env.{ENTORNO}")
load_dotenv(RAIZ / ".env")


class ErrorConfiguracion(RuntimeError):
    pass


def _obligatoria(nombre: str) -> str:
    valor = os.getenv(nombre)
    if not valor:
        raise ErrorConfiguracion(f"Falta la variable obligatoria {nombre}")
    return valor


def _bool(nombre: str, defecto: bool = False) -> bool:
    valor = os.getenv(nombre)
    return defecto if valor is None else valor.strip().lower() in {"1", "true", "yes", "si", "sí", "on"}


def _int(nombre: str, defecto: int) -> int:
    valor = os.getenv(nombre)
    if not valor:
        return defecto
    try:
        return int(valor)
    except ValueError as e:
        raise ErrorConfiguracion(f"{nombre} debe ser un entero (valor: {valor!r})") from e


def _lista(nombre: str) -> list[str]:
    return [x.strip() for x in os.getenv(nombre, "").split(",") if x.strip()]


@dataclass(frozen=True)
class BaseDatos:
    host: str
    puerto: int
    nombre: str
    usuario: str
    clave: str = field(repr=False)                # No aparece al imprimir

    @property
    def url(self) -> str:
        return f"postgresql://{self.usuario}:{self.clave}@{self.host}:{self.puerto}/{self.nombre}"


@dataclass(frozen=True)
class Correo:
    host: str | None
    puerto: int
    usuario: str | None
    clave: str | None = field(repr=False)

    @property
    def activo(self) -> bool:
        return bool(self.host and self.usuario and self.clave)


@dataclass(frozen=True)
class Configuracion:
    entorno: str
    nombre: str
    debug: bool
    puerto: int
    hosts: list[str]
    bd: BaseDatos
    correo: Correo
    api_token: str = field(repr=False)

    @property
    def es_produccion(self) -> bool:
        return self.entorno == "production"


def cargar() -> Configuracion:
    errores = []

    def intentar(func, *args, **kwargs):
        try:
            return func(*args, **kwargs)
        except ErrorConfiguracion as e:
            errores.append(str(e))
            return None

    cfg = Configuracion(
        entorno=ENTORNO,
        nombre=os.getenv("APP_NOMBRE", "Aplicación"),
        debug=_bool("APP_DEBUG"),
        puerto=intentar(_int, "APP_PUERTO", 8000),
        hosts=_lista("ALLOWED_HOSTS") or ["localhost"],
        bd=BaseDatos(
            host=os.getenv("DB_HOST", "localhost"),
            puerto=intentar(_int, "DB_PUERTO", 5432),
            nombre=intentar(_obligatoria, "DB_NOMBRE"),
            usuario=intentar(_obligatoria, "DB_USUARIO"),
            clave=intentar(_obligatoria, "DB_CLAVE"),
        ),
        correo=Correo(
            host=os.getenv("SMTP_HOST"),
            puerto=intentar(_int, "SMTP_PUERTO", 587),
            usuario=os.getenv("SMTP_USUARIO"),
            clave=os.getenv("SMTP_CLAVE"),
        ),
        api_token=intentar(_obligatoria, "API_TOKEN"),
    )

    if cfg.es_produccion and cfg.debug:
        errores.append("APP_DEBUG no puede estar activo en producción")

    if errores:
        raise ErrorConfiguracion("Configuración no válida:\n  - " + "\n  - ".join(errores))
    return cfg


config = cargar()
```

**`main.py`**

```python
from config import config

print(f"Iniciando {config.nombre} [{config.entorno}] en el puerto {config.puerto}")
print(config)                                    # Las claves no se muestran
print("Base de datos:", config.bd.host, config.bd.nombre)
print("Correo activo:", config.correo.activo)
```

---

## 11. Seguridad

### 11.1 Excluir `.env` del control de versiones

`.gitignore`:

```gitignore
# Variables de entorno
.env
.env.*
!.env.example
```

### 11.2 Si un `.env` se ha subido por error

1. **Revoque y regenere inmediatamente** todas las claves y tokens expuestos.
2. Elimine el archivo del repositorio: `git rm --cached .env` y confirme el cambio.
3. Tenga en cuenta que el archivo sigue en el historial: límpielo con herramientas como `git filter-repo` y considere los secretos comprometidos igualmente.

### 11.3 Otras recomendaciones

- Restrinja los permisos del archivo en servidores: `chmod 600 .env`.
- No imprima ni registre en logs los valores secretos.
- En producción, use variables del sistema o gestores de secretos (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault, secretos de GitHub Actions).
- No envíe archivos `.env` por correo o chat.

---

## 12. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `os.getenv()` devuelve `None` | No se llamó a `load_dotenv()` o no encuentra el archivo. | Llame a `load_dotenv()` al inicio con la ruta explícita. |
| Funciona en el IDE pero no al ejecutarlo desde otra carpeta | Búsqueda relativa al directorio actual. | Use `Path(__file__).resolve().parent / ".env"`. |
| El valor del `.env` no se aplica | Ya existe una variable del sistema con ese nombre. | Use `override=True` o elimine la variable del sistema. |
| `APP_DEBUG=false` se interpreta como verdadero | `bool("false")` es `True`. | Use una función de conversión. |
| `ModuleNotFoundError: No module named 'dotenv'` | Paquete no instalado. | `pip install python-dotenv` (no `pip install dotenv`). |
| Contraseña cortada | Contiene `#` sin comillas. | Entrecomille el valor: `CLAVE="abc#123"`. |
| `${VAR}` no se expande | Comillas simples o `interpolate=False`. | Use comillas dobles o sin comillas. |
| Caracteres extraños | Archivo guardado con BOM o codificación distinta. | Guárdelo en UTF-8 sin BOM o use `encoding="utf-8-sig"`. |

---

## 13. Buenas prácticas

- Llame a `load_dotenv()` **una sola vez**, al principio, en el punto de entrada o en un módulo `config.py`.
- Centralice la lectura de variables en un único módulo; el resto del código importa la configuración.
- Valide las variables obligatorias al arrancar y falle con un mensaje claro.
- Convierta los tipos explícitamente (o use Pydantic Settings).
- Mantenga un `.env.example` actualizado y documentado.
- No suba nunca `.env` al repositorio.
- Use prefijos (`APP_`, `DB_`, `SMTP_`) para organizar las variables.
- En producción, prefiera variables del sistema o gestores de secretos.

---

## 14. Referencia rápida

```python
from dotenv import load_dotenv, dotenv_values, find_dotenv, set_key, get_key, unset_key

load_dotenv()                                   # Carga .env (no sobrescribe)
load_dotenv(ruta, override=True, encoding="utf-8", interpolate=True)
dotenv_values(".env")                           # dict sin tocar os.environ
find_dotenv(usecwd=True)
set_key(".env", "CLAVE", "valor") / get_key(...) / unset_key(...)

os.getenv("CLAVE", "defecto") / os.environ["CLAVE"]
```

```bash
dotenv list | get CLAVE | set CLAVE valor | unset CLAVE | run -- comando
```

---

## 15. Ejercicios propuestos

1. Cree un `.env` y un `.env.example` para un proyecto que use base de datos, correo SMTP y una API externa.
2. Escriba un módulo `config.py` que valide las variables obligatorias y muestre todas las que faltan a la vez.
3. Configure tres entornos (`development`, `test`, `production`) y compruebe que se cargan los valores correctos según `APP_ENV`.
4. Migre un script que tenga contraseñas escritas en el código para que use variables de entorno.
5. Reescriba la configuración con `pydantic-settings` y compare ambos enfoques.

---

## 16. Referencias

- Documentación oficial: https://saurabh-kumar.com/python-dotenv/
- Repositorio: https://github.com/theskumar/python-dotenv
- The Twelve-Factor App (configuración): https://12factor.net/es/config
