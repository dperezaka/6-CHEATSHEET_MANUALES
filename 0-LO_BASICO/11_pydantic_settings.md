# Manual de uso: pydantic-settings

> **Categoría:** Configuración · **Utilidad:** Carga la configuración y los secretos desde variables de entorno y archivos `.env`
> **Tipo:** Paquete externo (requiere instalación) · Complemento oficial de Pydantic v2

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primeros pasos](#4-primeros-pasos)
5. [Configuración de la clase (`SettingsConfigDict`)](#5-configuración-de-la-clase-settingsconfigdict)
6. [Variables de entorno: nombres, prefijos y alias](#6-variables-de-entorno-nombres-prefijos-y-alias)
7. [Archivos `.env`](#7-archivos-env)
8. [Tipos complejos y modelos anidados](#8-tipos-complejos-y-modelos-anidados)
9. [Secretos](#9-secretos)
10. [Orden de prioridad y fuentes personalizadas](#10-orden-de-prioridad-y-fuentes-personalizadas)
11. [Archivos TOML, YAML y JSON](#11-archivos-toml-yaml-y-json)
12. [Argumentos de línea de comandos](#12-argumentos-de-línea-de-comandos)
13. [Patrones de uso en una aplicación](#13-patrones-de-uso-en-una-aplicación)
14. [Configuración en las pruebas](#14-configuración-en-las-pruebas)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**pydantic-settings** carga la configuración de una aplicación desde **variables de entorno**, archivos **`.env`**, **directorios de secretos** (Docker, Kubernetes), archivos **TOML/YAML/JSON**, la **línea de comandos** o gestores de secretos en la nube, y la **valida** con Pydantic.

```python
from pydantic_settings import BaseSettings

class Ajustes(BaseSettings):
    base_datos_url: str
    depuracion: bool = False
    trabajadores: int = 4

ajustes = Ajustes()        # Lee BASE_DATOS_URL, DEPURACION, TRABAJADORES del entorno
```

**Ventajas:**
- Sigue la metodología **12-factor**: la configuración vive en el entorno, no en el código.
- Validación y conversión de tipos: `"true"` → `True`, `"8"` → `8`, JSON → listas y diccionarios.
- Falla **al arrancar** si falta algo obligatorio o es incorrecto, no a mitad de ejecución.
- Una única clase documenta toda la configuración, con tipos y valores por defecto.
- Admite secretos sin exponerlos en registros (`SecretStr`).

**Limitaciones:**
- No es un gestor de secretos: los lee, pero no los almacena ni los rota.
- La configuración se lee al crear el objeto; los cambios posteriores del entorno no se reflejan.

| Si necesita… | Opción |
|---|---|
| Configuración tipada y validada desde el entorno | **pydantic-settings** |
| Solo leer un `.env` en `os.environ` | python-dotenv |
| Configuración jerárquica compleja con composición | Hydra / Dynaconf |

---

## 2. Instalación

```bash
uv add pydantic-settings
python -m pip install pydantic-settings
```

Extras opcionales:

| Extra | Habilita |
|---|---|
| `pydantic-settings[yaml]` | Lectura de archivos YAML. |
| `pydantic-settings[toml]` | Lectura de TOML en Python < 3.11. |
| `pydantic-settings[aws-secrets-manager]` | AWS Secrets Manager. |
| `pydantic-settings[azure-key-vault]` | Azure Key Vault. |
| `pydantic-settings[gcp-secret-manager]` | Google Secret Manager. |

> La lectura de archivos `.env` usa `python-dotenv`, que se instala como dependencia.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `BaseSettings` | Clase base: como `BaseModel`, pero rellena los campos desde fuentes externas. |
| Fuente | Origen de valores: argumentos, entorno, `.env`, secretos, archivos, nube. |
| Prioridad | Orden en que se consultan las fuentes; la primera que tiene el valor gana. |
| Prefijo | Texto que se antepone a todos los nombres de variable (`MIAPP_`). |
| Delimitador anidado | Separador para rellenar modelos anidados (`BD__HOST`). |
| Directorio de secretos | Carpeta con un archivo por secreto (`/run/secrets`). |
| `SecretStr` | Tipo que oculta su valor al imprimirlo. |

---

## 4. Primeros pasos

```python
# config.py
from pydantic import Field, PostgresDsn, SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    nombre_app: str = "Facturación"
    entorno: str = "desarrollo"
    depuracion: bool = False
    puerto: int = Field(default=8000, ge=1, le=65535)
    base_datos_url: PostgresDsn
    clave_secreta: SecretStr
    origenes_cors: list[str] = []


ajustes = Ajustes()
```

```bash
# .env
BASE_DATOS_URL=postgresql://app:clave@localhost:5432/facturas
CLAVE_SECRETA=cambie-esto-en-produccion
DEPURACION=true
ORIGENES_CORS=["http://localhost:3000","https://app.empresa.com"]
```

```python
>>> from config import ajustes
>>> ajustes.depuracion
True
>>> ajustes.clave_secreta
SecretStr('**********')
>>> ajustes.origenes_cors
['http://localhost:3000', 'https://app.empresa.com']
```

Si falta un campo obligatorio:

```text
pydantic_core._pydantic_core.ValidationError: 1 validation error for Ajustes
base_datos_url
  Field required [type=missing, input_value={}, input_type=dict]
```

---

## 5. Configuración de la clase (`SettingsConfigDict`)

```python
model_config = SettingsConfigDict(
    env_prefix="FACT_",                 # FACT_PUERTO, FACT_DEPURACION…
    env_file=(".env", ".env.local"),    # Varios archivos; los posteriores tienen prioridad
    env_file_encoding="utf-8",
    env_nested_delimiter="__",          # FACT_BD__HOST → bd.host
    env_ignore_empty=True,              # Trata VAR= como no definida
    case_sensitive=False,               # Por defecto, sin distinguir mayúsculas
    extra="ignore",                     # Ignora variables del .env que no son campos
    secrets_dir="/run/secrets",         # Directorio de secretos
    validate_default=True,
    cli_parse_args=False,               # Leer argumentos de la línea de comandos
)
```

| Opción | Por defecto | Uso |
|---|---|---|
| `env_prefix` | `""` | Evitar colisiones con otras variables del sistema. |
| `env_file` | `None` | Ruta o tupla de rutas a archivos `.env`. |
| `env_nested_delimiter` | `None` | Rellenar submodelos con variables planas. |
| `case_sensitive` | `False` | En Windows, las variables nunca distinguen mayúsculas. |
| `extra` | `"forbid"` | Con `.env` compartidos, use `"ignore"`. |
| `env_ignore_empty` | `False` | Recomendable `True` para no sobrescribir con vacíos. |
| `secrets_dir` | `None` | Ruta (o rutas) con archivos de secretos. |
| `env_parse_none_str` | `None` | Texto que se interpreta como `None` (p. ej. `"null"`). |
| `env_parse_enums` | `True` | Acepta el **nombre** del miembro de un `Enum`. |

---

## 6. Variables de entorno: nombres, prefijos y alias

Por defecto, el nombre de la variable es el del campo (sin distinguir mayúsculas), precedido del prefijo:

```python
class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="FACT_")
    puerto: int = 8000                 # Lee FACT_PUERTO
```

Nombres personalizados con alias (no se les aplica el prefijo):

```python
from pydantic import AliasChoices, Field


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="FACT_")

    redis_url: str = Field(validation_alias="REDIS_URL")                         # Nombre exacto
    base_datos_url: str = Field(validation_alias=AliasChoices("FACT_BD_URL", "DATABASE_URL"))
```

> Útil para leer variables estándar que define la plataforma (`DATABASE_URL`, `PORT`, `REDIS_URL`).

Conversión de tipos desde texto:

| Tipo | Valores aceptados en la variable |
|---|---|
| `bool` | `1`, `true`, `yes`, `on`, `0`, `false`, `no`, `off` (sin distinguir mayúsculas) |
| `int`, `float`, `Decimal` | `8`, `3.5` |
| `list`, `dict`, `set`, modelos | JSON: `["a","b"]`, `{"k": 1}` |
| `Enum` | Valor o nombre del miembro |
| `timedelta` | Segundos (`3600`) o ISO 8601 (`PT1H`) |
| `Path` | Ruta |

---

## 7. Archivos `.env`

```bash
# .env — valores para desarrollo local (NO versionar si contiene secretos)
# Comentarios con almohadilla
FACT_ENTORNO=desarrollo
FACT_DEPURACION=true
FACT_BASE_DATOS_URL="postgresql://app:clave@localhost:5432/facturas"
FACT_SMTP__HOST=localhost
FACT_SMTP__PUERTO=1025
export FACT_LOG_NIVEL=DEBUG      # Se admite el prefijo export
```

Archivos múltiples por entorno:

```python
model_config = SettingsConfigDict(env_file=(".env", ".env.local", ".env.produccion"))
```

Elegir el archivo al crear la instancia:

```python
ajustes = Ajustes(_env_file=".env.pruebas")        # Sobrescribe env_file
ajustes = Ajustes(_env_file=None)                  # Ignorar archivos .env
```

Recomendaciones sobre `.env`:

| Archivo | ¿Versionar? | Contenido |
|---|---|---|
| `.env.example` | **Sí** | Todas las variables con valores ficticios y comentarios. |
| `.env` | No | Valores locales del desarrollador. |
| `.env.local` | No | Sobrescrituras personales. |

```gitignore
.env
.env.*
!.env.example
```

> Las **variables de entorno reales** tienen prioridad sobre el archivo `.env`. Así, en producción se definen en la plataforma y el `.env` solo se usa en local.

---

## 8. Tipos complejos y modelos anidados

```python
from pydantic import BaseModel, SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict


class BaseDatos(BaseModel):
    host: str = "localhost"
    puerto: int = 5432
    nombre: str
    usuario: str
    clave: SecretStr

    @property
    def url(self) -> str:
        return (f"postgresql://{self.usuario}:{self.clave.get_secret_value()}"
                f"@{self.host}:{self.puerto}/{self.nombre}")


class Smtp(BaseModel):
    host: str
    puerto: int = 587
    tls: bool = True


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="FACT_", env_nested_delimiter="__")

    bd: BaseDatos
    smtp: Smtp | None = None
    funciones_activas: set[str] = set()
    limites: dict[str, int] = {}
```

Dos formas equivalentes de rellenar `bd`:

```bash
# Variables planas con el delimitador
FACT_BD__NOMBRE=facturas
FACT_BD__USUARIO=app
FACT_BD__CLAVE=secreta
FACT_BD__PUERTO=5433

# O un JSON completo
FACT_BD='{"nombre": "facturas", "usuario": "app", "clave": "secreta"}'
```

```bash
FACT_FUNCIONES_ACTIVAS=["exportar_pdf","firma_digital"]
FACT_LIMITES={"peticiones_minuto": 120, "tamano_mb": 10}
```

> Si combina el JSON y las variables planas, se fusionan: las planas completan o sobrescriben las claves del JSON.

---

## 9. Secretos

### 9.1 `SecretStr` y `SecretBytes`

```python
from pydantic import SecretStr


class Ajustes(BaseSettings):
    api_token: SecretStr


a = Ajustes()
print(a)                                   # api_token=SecretStr('**********')
cabecera = {"Authorization": f"Bearer {a.api_token.get_secret_value()}"}
```

### 9.2 Directorio de secretos (Docker / Kubernetes)

Docker Swarm y Kubernetes montan cada secreto como un archivo:

```
/run/secrets/
├── bd_clave            # Contenido: la contraseña
└── api_token
```

```python
class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(secrets_dir="/run/secrets")

    bd_clave: SecretStr
    api_token: SecretStr
```

```yaml
# docker-compose.yml
services:
  app:
    image: facturacion:latest
    secrets: [bd_clave, api_token]
secrets:
  bd_clave:
    file: ./secretos/bd_clave.txt
  api_token:
    environment: API_TOKEN
```

### 9.3 Gestores de secretos en la nube

```python
from pydantic_settings import AWSSecretsManagerSettingsSource, BaseSettings, PydanticBaseSettingsSource


class Ajustes(BaseSettings):
    bd_clave: SecretStr

    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings, env_settings,
                                   dotenv_settings, file_secret_settings):
        aws = AWSSecretsManagerSettingsSource(settings_cls, secret_id="produccion/facturacion")
        return (init_settings, env_settings, aws, dotenv_settings, file_secret_settings)
```

Existen fuentes equivalentes para Azure Key Vault (`AzureKeyVaultSettingsSource`) y Google Secret Manager (`GoogleSecretManagerSettingsSource`).

---

## 10. Orden de prioridad y fuentes personalizadas

Orden por defecto (de mayor a menor prioridad):

| # | Fuente | Ejemplo |
|---|---|---|
| 1 | Argumentos de la línea de comandos (si `cli_parse_args`) | `--puerto 9000` |
| 2 | Argumentos al crear la instancia | `Ajustes(puerto=9000)` |
| 3 | Variables de entorno | `FACT_PUERTO=9000` |
| 4 | Archivos `.env` | `FACT_PUERTO=9000` en `.env` |
| 5 | Directorio de secretos | `/run/secrets/fact_puerto` |
| 6 | Valores por defecto del campo | `puerto: int = 8000` |

Cambiar el orden o añadir fuentes:

```python
from pydantic_settings import BaseSettings, PydanticBaseSettingsSource


class Ajustes(BaseSettings):
    ...

    @classmethod
    def settings_customise_sources(
        cls,
        settings_cls: type[BaseSettings],
        init_settings: PydanticBaseSettingsSource,
        env_settings: PydanticBaseSettingsSource,
        dotenv_settings: PydanticBaseSettingsSource,
        file_secret_settings: PydanticBaseSettingsSource,
    ) -> tuple[PydanticBaseSettingsSource, ...]:
        # El .env pasa a tener prioridad sobre las variables de entorno
        return init_settings, dotenv_settings, env_settings, file_secret_settings
```

Fuente propia (por ejemplo, una tabla de configuración en base de datos):

```python
from typing import Any

from pydantic.fields import FieldInfo
from pydantic_settings import PydanticBaseSettingsSource


class FuenteDiccionario(PydanticBaseSettingsSource):
    def __init__(self, settings_cls: type[BaseSettings], datos: dict[str, Any]) -> None:
        super().__init__(settings_cls)
        self.datos = datos

    def get_field_value(self, field: FieldInfo, field_name: str) -> tuple[Any, str, bool]:
        return self.datos.get(field_name), field_name, False

    def __call__(self) -> dict[str, Any]:
        return {k: v for k, v in self.datos.items() if k in self.settings_cls.model_fields}
```

---

## 11. Archivos TOML, YAML y JSON

```python
from pydantic_settings import (BaseSettings, PydanticBaseSettingsSource, SettingsConfigDict,
                               TomlConfigSettingsSource, YamlConfigSettingsSource)


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(toml_file="config.toml", yaml_file="config.yaml", env_prefix="FACT_")

    puerto: int = 8000
    origenes_cors: list[str] = []

    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings, env_settings,
                                   dotenv_settings, file_secret_settings):
        return (
            init_settings,
            env_settings,                          # El entorno sobrescribe los archivos
            dotenv_settings,
            TomlConfigSettingsSource(settings_cls),
            YamlConfigSettingsSource(settings_cls),
            file_secret_settings,
        )
```

```toml
# config.toml
puerto = 8080
origenes_cors = ["https://app.empresa.com"]
```

También están disponibles `JsonConfigSettingsSource` y `PyprojectTomlConfigSettingsSource` (lee una tabla de `pyproject.toml`, p. ej. `[tool.mi_app]`).

---

## 12. Argumentos de línea de comandos

```python
from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict


class Opciones(BaseSettings):
    model_config = SettingsConfigDict(cli_parse_args=True, cli_prog_name="exportar", env_prefix="EXP_")

    origen: str = Field(description="Ruta del archivo de entrada")
    formato: str = Field(default="csv", description="Formato de salida: csv o xlsx")
    limite: int | None = Field(default=None, description="Número máximo de filas")
    detallado: bool = False


if __name__ == "__main__":
    opciones = Opciones()
    print(opciones)
```

```bash
python exportar.py --origen datos.json --formato xlsx --limite 100 --detallado true
EXP_FORMATO=xlsx python exportar.py --origen datos.json      # También desde el entorno
python exportar.py --help
```

> Para aplicaciones de terminal complejas (subcomandos, autocompletado), Typer o Click siguen siendo más adecuados.

---

## 13. Patrones de uso en una aplicación

### 13.1 Instancia única con caché

```python
# config.py
from functools import lru_cache


@lru_cache
def obtener_ajustes() -> Ajustes:
    return Ajustes()
```

```python
# En cualquier módulo
from mi_app.config import obtener_ajustes

ajustes = obtener_ajustes()
```

> Crear la instancia dentro de una función (y no a nivel de módulo) evita leer el entorno al importar, lo que facilita las pruebas.

### 13.2 Inyección en FastAPI

```python
from typing import Annotated

from fastapi import Depends, FastAPI

app = FastAPI()
AjustesDep = Annotated[Ajustes, Depends(obtener_ajustes)]


@app.get("/info")
def info(ajustes: AjustesDep) -> dict[str, str]:
    return {"app": ajustes.nombre_app, "entorno": ajustes.entorno}
```

### 13.3 Validar la configuración al arrancar

```python
import sys

from pydantic import ValidationError


def main() -> None:
    try:
        ajustes = obtener_ajustes()
    except ValidationError as e:
        print("Configuración no válida:", file=sys.stderr)
        for err in e.errors():
            print(f"  - {'.'.join(map(str, err['loc']))}: {err['msg']}", file=sys.stderr)
        sys.exit(78)    # EX_CONFIG
    ...
```

---

## 14. Configuración en las pruebas

```python
import pytest

from mi_app.config import Ajustes, obtener_ajustes


@pytest.fixture
def ajustes_prueba(monkeypatch: pytest.MonkeyPatch, tmp_path) -> Ajustes:
    monkeypatch.setenv("FACT_BD__NOMBRE", "pruebas")
    monkeypatch.setenv("FACT_BD__USUARIO", "test")
    monkeypatch.setenv("FACT_BD__CLAVE", "test")
    monkeypatch.setenv("FACT_ENTORNO", "pruebas")
    obtener_ajustes.cache_clear()
    yield Ajustes(_env_file=None)              # No leer el .env del desarrollador
    obtener_ajustes.cache_clear()


def test_entorno(ajustes_prueba: Ajustes) -> None:
    assert ajustes_prueba.entorno == "pruebas"


def test_construccion_directa() -> None:
    a = Ajustes(_env_file=None, bd={"nombre": "x", "usuario": "u", "clave": "c"})
    assert a.bd.puerto == 5432
```

Con FastAPI, sustituya la dependencia:

```python
app.dependency_overrides[obtener_ajustes] = lambda: Ajustes(_env_file=None, ...)
```

---

## 15. Ejemplo práctico completo

**Configuración de una aplicación de facturación** por entornos (desarrollo, pruebas, producción): modelos anidados, secretos, validación de coherencia según el entorno, comprobación al arrancar con informe legible, generación automática de `.env.example` a partir de la propia clase y volcado seguro (sin secretos) para diagnóstico.

```python
"""
config.py — Configuración de la aplicación con pydantic-settings.

    python config.py comprobar       # Valida la configuración actual
    python config.py ejemplo         # Genera .env.example
    python config.py mostrar         # Muestra la configuración (secretos ocultos)
"""
from __future__ import annotations

import json
import sys
from enum import StrEnum
from functools import lru_cache
from pathlib import Path
from typing import Self

from pydantic import BaseModel, Field, HttpUrl, SecretStr, ValidationError, model_validator
from pydantic_settings import BaseSettings, SettingsConfigDict

PREFIJO = "FACT_"


class Entorno(StrEnum):
    DESARROLLO = "desarrollo"
    PRUEBAS = "pruebas"
    PRODUCCION = "produccion"


class NivelLog(StrEnum):
    DEBUG = "DEBUG"
    INFO = "INFO"
    WARNING = "WARNING"
    ERROR = "ERROR"


class BaseDatos(BaseModel):
    host: str = Field(default="localhost", description="Servidor PostgreSQL")
    puerto: int = Field(default=5432, ge=1, le=65535, description="Puerto PostgreSQL")
    nombre: str = Field(description="Nombre de la base de datos")
    usuario: str = Field(description="Usuario de la base de datos")
    clave: SecretStr = Field(description="Contraseña de la base de datos")
    pool: int = Field(default=5, ge=1, le=100, description="Conexiones simultáneas")

    def url(self) -> str:
        return (f"postgresql+psycopg://{self.usuario}:{self.clave.get_secret_value()}"
                f"@{self.host}:{self.puerto}/{self.nombre}")


class Correo(BaseModel):
    host: str = Field(default="localhost", description="Servidor SMTP")
    puerto: int = Field(default=1025, description="Puerto SMTP")
    usuario: str | None = Field(default=None, description="Usuario SMTP")
    clave: SecretStr | None = Field(default=None, description="Contraseña SMTP")
    remitente: str = Field(default="facturas@empresa.com", description="Dirección del remitente")
    tls: bool = Field(default=False, description="Usar STARTTLS")


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(
        env_prefix=PREFIJO,
        env_file=(".env", ".env.local"),
        env_file_encoding="utf-8",
        env_nested_delimiter="__",
        env_ignore_empty=True,
        extra="ignore",
        secrets_dir=("/run/secrets" if Path("/run/secrets").is_dir() else None),
    )

    entorno: Entorno = Field(default=Entorno.DESARROLLO, description="desarrollo | pruebas | produccion")
    depuracion: bool = Field(default=False, description="Modo depuración")
    log_nivel: NivelLog = Field(default=NivelLog.INFO, description="Nivel de registro")
    clave_secreta: SecretStr = Field(description="Clave para firmar sesiones (mín. 32 caracteres)")
    url_publica: HttpUrl = Field(default=HttpUrl("http://localhost:8000"), description="URL pública")
    origenes_cors: list[str] = Field(default_factory=list, description='JSON: ["https://..."]')
    bd: BaseDatos
    correo: Correo = Field(default_factory=Correo)

    @model_validator(mode="after")
    def coherencia_por_entorno(self) -> Self:
        if self.entorno is Entorno.PRODUCCION:
            problemas = []
            if self.depuracion:
                problemas.append("la depuración debe estar desactivada")
            if len(self.clave_secreta.get_secret_value()) < 32:
                problemas.append("la clave secreta debe tener al menos 32 caracteres")
            if self.url_publica.scheme != "https":
                problemas.append("la URL pública debe usar https")
            if "*" in self.origenes_cors:
                problemas.append("no se permite CORS con '*'")
            if not self.correo.tls:
                problemas.append("el correo debe usar TLS")
            if problemas:
                raise ValueError("producción: " + "; ".join(problemas))
        return self

    def resumen_seguro(self) -> dict[str, object]:
        """Configuración serializable con los secretos ocultos."""
        return json.loads(self.model_dump_json())   # SecretStr se serializa como '**********'


@lru_cache
def obtener_ajustes() -> Ajustes:
    return Ajustes()


# ------------------------------------------------------------------ Utilidades de terminal
def generar_ejemplo(modelo: type[BaseModel], prefijo: str = PREFIJO) -> list[str]:
    lineas: list[str] = []
    for nombre, campo in modelo.model_fields.items():
        variable = f"{prefijo}{nombre}".upper()
        anotacion = campo.annotation
        if isinstance(anotacion, type) and issubclass(anotacion, BaseModel):
            lineas.append(f"\n# --- {nombre} ---")
            lineas += generar_ejemplo(anotacion, f"{variable}__")
            continue
        obligatorio = campo.is_required()
        defecto = "" if obligatorio else campo.get_default(call_default_factory=True)
        if isinstance(defecto, (list, dict, set)):
            defecto = json.dumps(list(defecto) if isinstance(defecto, set) else defecto)
        elif isinstance(defecto, SecretStr) or defecto is None:
            defecto = ""
        marca = " (OBLIGATORIO)" if obligatorio else ""
        lineas.append(f"# {campo.description or nombre}{marca}")
        lineas.append(f"{variable}={defecto}")
    return lineas


def comprobar() -> int:
    try:
        ajustes = obtener_ajustes()
    except ValidationError as e:
        print(f"✖ Configuración no válida ({e.error_count()} errores):", file=sys.stderr)
        for err in e.errors(include_url=False):
            ruta = "__".join(str(p) for p in err["loc"]).upper()
            variable = f"{PREFIJO}{ruta}" if ruta else "(general)"
            print(f"   {variable}: {err['msg']}", file=sys.stderr)
        return 78
    print(f"✔ Configuración válida para el entorno «{ajustes.entorno}»")
    return 0


def main() -> int:
    orden = sys.argv[1] if len(sys.argv) > 1 else "comprobar"
    if orden == "ejemplo":
        Path(".env.example").write_text("\n".join(generar_ejemplo(Ajustes)) + "\n", encoding="utf-8")
        print("✔ .env.example generado")
        return 0
    if orden == "mostrar":
        if (codigo := comprobar()) != 0:
            return codigo
        print(json.dumps(obtener_ajustes().resumen_seguro(), indent=2, ensure_ascii=False))
        return 0
    return comprobar()


if __name__ == "__main__":
    raise SystemExit(main())
```

Uso:

```bash
python config.py ejemplo
cp .env.example .env              # Rellene los valores obligatorios
python config.py comprobar
python config.py mostrar

FACT_ENTORNO=produccion FACT_DEPURACION=true python config.py comprobar
```

Salida del último comando con un `.env` de desarrollo:

```text
✖ Configuración no válida (1 errores):
   (general): Value error, producción: la depuración debe estar desactivada; la clave secreta debe tener al menos 32 caracteres; la URL pública debe usar https; el correo debe usar TLS
```

Fragmento del `.env.example` generado:

```bash
# desarrollo | pruebas | produccion
FACT_ENTORNO=desarrollo
# Modo depuración
FACT_DEPURACION=False
# Clave para firmar sesiones (mín. 32 caracteres) (OBLIGATORIO)
FACT_CLAVE_SECRETA=

# --- bd ---
# Servidor PostgreSQL
FACT_BD__HOST=localhost
# Nombre de la base de datos (OBLIGATORIO)
FACT_BD__NOMBRE=
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Field required` con la variable definida | Prefijo distinto o nombre mal escrito. | Revise `env_prefix`; con alias, el prefijo no se aplica. |
| `Extra inputs are not permitted` | El `.env` contiene variables que no son campos. | `extra="ignore"`. |
| `error parsing value for field "x" from source "EnvSettingsSource"` | Lista o diccionario no escrito en JSON. | Use JSON válido: `["a","b"]`. |
| El `.env` no se lee | Ruta relativa al directorio de trabajo, no al script. | Use una ruta absoluta: `Path(__file__).parent / ".env"`. |
| El valor del `.env` no se aplica | Existe una variable de entorno con el mismo nombre (tiene prioridad). | `env | grep FACT_` y elimine la variable. |
| Submodelo no se rellena desde variables planas | Falta `env_nested_delimiter`. | `env_nested_delimiter="__"`. |
| La contraseña aparece en los registros | Se usó `str` en lugar de `SecretStr`, o se registró `get_secret_value()`. | Use `SecretStr` y no registre su valor. |
| Las pruebas usan el `.env` del desarrollador | La instancia lee `.env` por defecto. | `Ajustes(_env_file=None)` en las pruebas. |
| Los cambios de entorno no se reflejan | Instancia en caché con `lru_cache`. | `obtener_ajustes.cache_clear()`. |
| `ImportError: BaseSettings has been moved` | Código de Pydantic v1. | `from pydantic_settings import BaseSettings`. |

---

## 17. Buenas prácticas

- Una **única clase** de configuración por aplicación, en un módulo `config.py`.
- Use un **prefijo** propio (`MIAPP_`) para evitar colisiones.
- Declare como **obligatorio** (sin valor por defecto) todo lo que no tenga un valor seguro por defecto: es mejor fallar al arrancar.
- Use `SecretStr` para contraseñas, *tokens* y claves; nunca registre `get_secret_value()`.
- Versione `.env.example`; no versione `.env`.
- En producción, use variables de entorno de la plataforma o un gestor de secretos, no archivos `.env`.
- Valide la **coherencia entre campos** según el entorno con `model_validator`.
- Cree la instancia dentro de una función con `lru_cache` y no a nivel de módulo.
- En pruebas, construya la configuración explícitamente y desactive el `.env` (`_env_file=None`).
- Use submodelos (`bd`, `correo`, `cache`) para agrupar la configuración relacionada.

---

## 18. Referencia rápida

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="APP_", env_file=".env", env_nested_delimiter="__",
                                      env_ignore_empty=True, extra="ignore", secrets_dir="/run/secrets")
    campo: int = 8000
    secreto: SecretStr
    sub: SubModelo                          # APP_SUB__CLAVE=valor
    lista: list[str] = []                   # APP_LISTA=["a","b"]
    otro: str = Field(validation_alias="DATABASE_URL")

    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings, env_settings,
                                   dotenv_settings, file_secret_settings): return (...)

Ajustes() · Ajustes(_env_file=None) · Ajustes(_env_file=".env.pruebas")
@lru_cache def obtener_ajustes(): return Ajustes()
```

Prioridad: CLI → argumentos → entorno → `.env` → secretos → valores por defecto.

---

## 19. Ejercicios propuestos

1. Cree la configuración de una aplicación con base de datos y caché Redis como submodelos, y rellénela con variables planas.
2. Añada una fuente TOML con prioridad menor que el entorno y compruebe el orden de prioridad.
3. Monte un secreto en Docker Compose y léalo con `secrets_dir`.
4. Escriba pruebas con `monkeypatch` que verifiquen que en producción se rechaza `depuracion=true`.
5. Amplíe `generar_ejemplo` para que incluya los valores de los `Enum` admitidos en el comentario.

---

## 20. Referencias

- Documentación oficial: https://docs.pydantic.dev/latest/concepts/pydantic_settings/
- Repositorio: https://github.com/pydantic/pydantic-settings
- The Twelve-Factor App (configuración): https://12factor.net/es/config
- Secretos en Docker: https://docs.docker.com/compose/how-tos/use-secrets/
