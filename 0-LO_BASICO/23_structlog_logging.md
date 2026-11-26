# Manual de uso: structlog y logging

> **Categoría:** Registro · **Utilidad:** Registros estructurados para producción
> **Tipo:** `logging`: librería estándar (no requiere instalación) · `structlog`: paquete externo

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [logging: uso básico](#4-logging-uso-básico)
5. [logging: niveles y cuándo usarlos](#5-logging-niveles-y-cuándo-usarlos)
6. [logging: jerarquía de *loggers*](#6-logging-jerarquía-de-loggers)
7. [logging: *handlers*, formateadores y filtros](#7-logging-handlers-formateadores-y-filtros)
8. [logging: configuración con `dictConfig`](#8-logging-configuración-con-dictconfig)
9. [logging: excepciones, contexto y rendimiento](#9-logging-excepciones-contexto-y-rendimiento)
10. [logging en librerías](#10-logging-en-librerías)
11. [Registros estructurados (JSON)](#11-registros-estructurados-json)
12. [structlog: primeros pasos](#12-structlog-primeros-pasos)
13. [structlog: procesadores](#13-structlog-procesadores)
14. [structlog: contexto ligado y `contextvars`](#14-structlog-contexto-ligado-y-contextvars)
15. [structlog: integración con logging](#15-structlog-integración-con-logging)
16. [Pruebas de los registros](#16-pruebas-de-los-registros)
17. [Registros en producción](#17-registros-en-producción)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

Los **registros** (*logs*) son la principal fuente de información sobre lo que hace una aplicación en producción: qué peticiones atiende, qué errores se producen y en qué contexto.

- **`logging`** es el sistema de registro de la librería estándar. Todas las librerías de Python lo usan, así que cualquier aplicación lo necesita configurado.
- **structlog** añade **registros estructurados**: cada evento es un conjunto de pares clave-valor (no solo un texto), con contexto que se arrastra automáticamente (usuario, petición, pedido) y salida legible en desarrollo y **JSON** en producción.

```python
# logging clásico: texto libre, difícil de filtrar
log.info("Pedido %s creado para %s por %.2f €", 1042, "ana@ejemplo.com", 59.90)
# 2026-11-26 10:30:00 INFO tienda Pedido 1042 creado para ana@ejemplo.com por 59.90 €

# structlog: evento + datos
log.info("pedido_creado", pedido_id=1042, cliente="ana@ejemplo.com", total=59.90)
# {"event": "pedido_creado", "pedido_id": 1042, "cliente": "ana@ejemplo.com", "total": 59.9,
#  "level": "info", "timestamp": "2026-11-26T10:30:00Z", "request_id": "7f3a…"}
```

**Ventajas de los registros estructurados:**
- Búsqueda y filtrado precisos en Elasticsearch/OpenSearch, Loki, Datadog, CloudWatch, Google Cloud Logging…
- Métricas y alertas a partir de los registros (`count by event`, `avg(total)`).
- Contexto completo sin repetir datos en cada llamada.

| Si necesita… | Opción |
|---|---|
| Registro básico, scripts, librerías | **logging** |
| Aplicaciones y servicios en producción con registros estructurados | **structlog** (sobre logging) |
| Solo JSON con logging, sin cambiar las llamadas | `python-json-logger` |
| API sencilla con configuración mínima | loguru |
| Trazas distribuidas y métricas además de registros | OpenTelemetry |

---

## 2. Instalación

```bash
# logging: incluido en Python
uv add structlog
uv add rich                  # Opcional: trazas de excepciones coloreadas en desarrollo
python -m pip install structlog
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Logger* | Objeto con nombre que emite registros (`logging.getLogger("tienda.pedidos")`). |
| Nivel | Gravedad: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. |
| Registro (*LogRecord*) | Un evento: mensaje, nivel, hora, módulo, línea, excepción… |
| *Handler* | Destino del registro: consola, archivo, red, cola. |
| Formateador | Convierte el registro en texto (o JSON). |
| Filtro | Decide si un registro pasa o lo modifica. |
| Propagación | Los registros suben a los *loggers* padre hasta el raíz. |
| Evento (structlog) | Nombre de lo que ocurrió (`pedido_creado`), más sus datos. |
| Procesador (structlog) | Función que añade, modifica o elimina datos del evento. |
| Contexto ligado | Datos asociados al *logger* o a la petición que se añaden a cada evento. |
| Renderizador | Último procesador: produce el texto final (consola legible o JSON). |

---

## 4. logging: uso básico

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

log = logging.getLogger(__name__)          # Un logger por módulo, con el nombre del módulo


def procesar(pedido_id: int) -> None:
    log.debug("Inicio del proceso de %s", pedido_id)        # No se muestra (nivel INFO)
    log.info("Procesando pedido %s", pedido_id)
    log.warning("Stock bajo para el pedido %s", pedido_id)
```

```text
2026-11-26 10:30:00 INFO     tienda.pedidos: Procesando pedido 1042
2026-11-26 10:30:00 WARNING  tienda.pedidos: Stock bajo para el pedido 1042
```

> `basicConfig` solo surte efecto la **primera vez** (si el *logger* raíz no tiene *handlers*). Use `force=True` para reconfigurar.

Atributos útiles del formato:

| Atributo | Contenido |
|---|---|
| `%(asctime)s` | Fecha y hora. |
| `%(levelname)s` | Nivel. |
| `%(name)s` | Nombre del *logger*. |
| `%(message)s` | Mensaje. |
| `%(module)s`, `%(funcName)s`, `%(lineno)d` | Origen en el código. |
| `%(process)d`, `%(thread)d`, `%(threadName)s` | Proceso e hilo. |
| `%(taskName)s` | Tarea de asyncio (Python 3.12+). |

---

## 5. logging: niveles y cuándo usarlos

| Nivel | Valor | Uso |
|---|---|---|
| `DEBUG` | 10 | Detalle para diagnosticar (valores intermedios, consultas). Desactivado en producción. |
| `INFO` | 20 | Hitos normales del negocio (pedido creado, tarea terminada, servicio arrancado). |
| `WARNING` | 30 | Algo inesperado pero recuperable (reintento, configuración por defecto, uso obsoleto). |
| `ERROR` | 40 | Fallo de una operación (no se pudo enviar el correo, petición fallida). |
| `CRITICAL` | 50 | Fallo grave del servicio (no hay base de datos, no puede arrancar). |

```python
log.setLevel(logging.DEBUG)                    # Umbral del logger
logging.getLogger("httpx").setLevel(logging.WARNING)   # Silenciar una librería ruidosa
log.isEnabledFor(logging.DEBUG)                # ¿Se emitiría un DEBUG?
```

> Un registro se emite si supera **el nivel del *logger*** y **el del *handler***.

---

## 6. logging: jerarquía de *loggers*

Los nombres con puntos forman un árbol: `tienda.pedidos` es hijo de `tienda`, que es hijo del raíz.

```
root  (nivel WARNING por defecto)
├── tienda                (INFO)
│   ├── tienda.pedidos
│   └── tienda.pagos      (DEBUG)
├── httpx                 (WARNING)
└── sqlalchemy.engine     (WARNING)
```

- Un *logger* sin nivel propio hereda el **nivel efectivo** del padre.
- Los registros **se propagan** hacia arriba y los gestionan los *handlers* de cada antecesor.
- Configure los *handlers* en el **raíz** (o en el *logger* de la aplicación) y use `getLogger(__name__)` en cada módulo.

```python
logging.getLogger("tienda.pagos").setLevel(logging.DEBUG)   # Más detalle solo en pagos
logging.getLogger("tienda.legado").propagate = False        # No propagar (con handler propio)
```

---

## 7. logging: *handlers*, formateadores y filtros

```python
import logging
import sys
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

consola = logging.StreamHandler(sys.stdout)
consola.setLevel(logging.INFO)
consola.setFormatter(logging.Formatter("%(levelname)s %(name)s: %(message)s"))

archivo = RotatingFileHandler("tienda.log", maxBytes=10_000_000, backupCount=5, encoding="utf-8")
archivo.setLevel(logging.DEBUG)
archivo.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(name)s:%(lineno)d %(message)s"))

raiz = logging.getLogger()
raiz.setLevel(logging.DEBUG)
raiz.addHandler(consola)
raiz.addHandler(archivo)
```

*Handlers* de la librería estándar:

| *Handler* | Destino |
|---|---|
| `StreamHandler` | Consola (`stderr` por defecto). |
| `FileHandler` | Archivo. |
| `RotatingFileHandler` | Archivo con rotación por tamaño. |
| `TimedRotatingFileHandler` | Archivo con rotación por tiempo (diaria, semanal). |
| `QueueHandler` + `QueueListener` | Envío a una cola (no bloquea el hilo principal). |
| `SysLogHandler` | Syslog. |
| `SMTPHandler` | Correo electrónico (para errores críticos). |
| `HTTPHandler` | Servicio HTTP. |
| `NullHandler` | Descarta (para librerías). |

Filtros:

```python
class SoloPagos(logging.Filter):
    def filter(self, record: logging.LogRecord) -> bool:
        return record.name.startswith("tienda.pagos")


class AnadirEntorno(logging.Filter):
    def __init__(self, entorno: str) -> None:
        super().__init__()
        self.entorno = entorno

    def filter(self, record: logging.LogRecord) -> bool:
        record.entorno = self.entorno            # Añade un atributo usable en el formato
        return True


consola.addFilter(AnadirEntorno("produccion"))
```

---

## 8. logging: configuración con `dictConfig`

Configuración declarativa (también cargable desde YAML/TOML):

```python
import logging.config

CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,          # ¡Importante! No silenciar loggers ya creados
    "formatters": {
        "simple": {"format": "%(asctime)s %(levelname)-8s %(name)s: %(message)s"},
    },
    "filters": {},
    "handlers": {
        "consola": {
            "class": "logging.StreamHandler",
            "stream": "ext://sys.stdout",
            "formatter": "simple",
            "level": "INFO",
        },
    },
    "loggers": {
        "tienda": {"level": "DEBUG"},
        "httpx": {"level": "WARNING"},
        "uvicorn.access": {"level": "WARNING"},
    },
    "root": {"level": "INFO", "handlers": ["consola"]},
}

logging.config.dictConfig(CONFIG)
```

> `disable_existing_loggers` vale `True` por defecto: desactiva todos los *loggers* creados antes de la configuración (por ejemplo, al importar módulos). Casi siempre debe ser `False`.

Registro no bloqueante con cola (Python 3.12+):

```python
CONFIG["handlers"]["cola"] = {
    "class": "logging.handlers.QueueHandler",
    "handlers": ["consola"],                  # Se ejecutan en un hilo aparte
    "respect_handler_level": True,
}
CONFIG["root"]["handlers"] = ["cola"]
logging.config.dictConfig(CONFIG)
logging.getHandlerByName("cola").listener.start()
```

---

## 9. logging: excepciones, contexto y rendimiento

### 9.1 Excepciones

```python
try:
    cobrar(pedido)
except PasarelaError:
    log.exception("Error al cobrar el pedido %s", pedido.id)   # Nivel ERROR + traza completa
    raise

log.error("Fallo no crítico", exc_info=True)        # Traza con otro nivel
log.warning("Reintento", stack_info=True)           # Pila actual (sin excepción)
```

> Use `log.exception` **dentro** de un bloque `except`. No registre y relance la misma excepción en cada nivel: se duplica la traza. Regístrela donde se gestiona.

### 9.2 Formato perezoso

```python
log.debug("Datos: %s", datos)                 # ✔ Solo se formatea si DEBUG está activo
log.debug(f"Datos: {datos}")                  # ✖ Se formatea siempre (y Ruff G004 lo señala)

if log.isEnabledFor(logging.DEBUG):           # Para cálculos costosos
    log.debug("Resumen: %s", calcular_resumen_costoso())
```

### 9.3 Datos adicionales

```python
log.info("Pedido creado", extra={"pedido_id": 1042, "cliente": "ana@ejemplo.com"})
```

`extra` añade atributos al `LogRecord` (útil con formateadores JSON). Para contexto que se repite en muchas llamadas, `LoggerAdapter` o, mejor, structlog con `contextvars`.

---

## 10. logging en librerías

Si escribe una **librería**, no configure el registro (eso es decisión de la aplicación):

```python
# mi_libreria/__init__.py
import logging

logging.getLogger(__name__).addHandler(logging.NullHandler())
```

- Use `getLogger(__name__)` y emita registros normalmente.
- No llame a `basicConfig` ni añada *handlers* propios.
- No use `print` para avisos: `warnings.warn` para usos obsoletos y `logging` para eventos.

---

## 11. Registros estructurados (JSON)

En producción, la salida debe ser **una línea JSON por evento** en `stdout`, para que la plataforma la recoja e indexe:

```json
{"timestamp": "2026-11-26T09:30:00.123Z", "level": "info", "logger": "tienda.pedidos",
 "event": "pedido_creado", "pedido_id": 1042, "total": 59.9, "request_id": "7f3a9c"}
```

Opciones:

| Opción | Cuándo |
|---|---|
| **structlog** con `JSONRenderer` | Aplicaciones nuevas o donde se pueda cambiar la forma de registrar. |
| `python-json-logger` | Mantener `logging` puro y solo cambiar el formateador. |
| Formateador propio | Requisitos muy específicos. |

Formateador JSON mínimo con la librería estándar:

```python
import json
import logging
from datetime import datetime, UTC

CAMPOS_ESTANDAR = set(vars(logging.makeLogRecord({}))) | {"message", "asctime", "taskName"}


class FormateadorJSON(logging.Formatter):
    def format(self, record: logging.LogRecord) -> str:
        datos = {
            "timestamp": datetime.fromtimestamp(record.created, UTC).isoformat(timespec="milliseconds"),
            "level": record.levelname.lower(),
            "logger": record.name,
            "event": record.getMessage(),
            **{k: v for k, v in vars(record).items() if k not in CAMPOS_ESTANDAR},
        }
        if record.exc_info:
            datos["exception"] = self.formatException(record.exc_info)
        return json.dumps(datos, ensure_ascii=False, default=str)
```

---

## 12. structlog: primeros pasos

Sin configuración, structlog ya produce una salida legible en la consola:

```python
import structlog

log = structlog.get_logger()

log.info("pedido_creado", pedido_id=1042, total=59.90)
log.warning("stock_bajo", sku="TEC-01", unidades=2)
```

```text
2026-11-26 10:30:00 [info     ] pedido_creado                  pedido_id=1042 total=59.9
2026-11-26 10:30:00 [warning  ] stock_bajo                     sku=TEC-01 unidades=2
```

Configuración explícita (desarrollo legible / producción JSON):

```python
import logging
import sys

import structlog


def configurar(json: bool, nivel: int = logging.INFO) -> None:
    procesadores = [
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso", utc=True),
        structlog.processors.StackInfoRenderer(),
    ]
    if json:
        procesadores += [
            structlog.processors.ExceptionRenderer(
                structlog.tracebacks.ExceptionDictTransformer(show_locals=False)),
            structlog.processors.JSONRenderer(),
        ]
    else:
        procesadores += [structlog.dev.ConsoleRenderer()]

    structlog.configure(
        processors=procesadores,
        wrapper_class=structlog.make_filtering_bound_logger(nivel),
        logger_factory=structlog.PrintLoggerFactory(sys.stdout),
        cache_logger_on_first_use=True,
    )
```

Convenciones:

- El **primer argumento** es el nombre del **evento**, corto y estable (`pedido_creado`, `pago_rechazado`), no una frase con datos.
- Los datos van como **argumentos con nombre**.
- Use los mismos nombres de campo en todo el proyecto (`pedido_id`, no a veces `id_pedido`).

---

## 13. structlog: procesadores

Cada procesador recibe `(logger, nombre_metodo, evento: dict)` y devuelve el diccionario modificado. Se ejecutan en orden; el último **renderiza**.

| Procesador | Función |
|---|---|
| `contextvars.merge_contextvars` | Añade el contexto de la petición/tarea actual. |
| `processors.add_log_level` | Añade `level`. |
| `stdlib.add_logger_name` | Añade `logger` (con integración stdlib). |
| `processors.TimeStamper(fmt="iso", utc=True)` | Añade `timestamp`. |
| `processors.CallsiteParameterAdder([...])` | Archivo, función, línea, proceso, hilo. |
| `processors.StackInfoRenderer()` | Pila si se pasa `stack_info=True`. |
| `processors.dict_tracebacks` | Excepción como estructura JSON (**incluye variables locales**: en producción use `ExceptionRenderer(ExceptionDictTransformer(show_locals=False))`). |
| `processors.format_exc_info` | Excepción como texto. |
| `processors.EventRenamer("message")` | Renombrar `event` (algunas plataformas esperan `message`). |
| `processors.UnicodeDecoder()` | Bytes a texto. |
| `dev.ConsoleRenderer()` | Salida coloreada y legible (desarrollo). |
| `processors.JSONRenderer()` | JSON (producción). |
| `processors.LogfmtRenderer()` | Formato `clave=valor`. |

Procesador propio para **ocultar datos sensibles**:

```python
SENSIBLES = {"password", "contrasena", "token", "api_key", "authorization", "tarjeta", "iban"}


def ocultar_sensibles(logger, metodo, evento: dict) -> dict:
    for clave in list(evento):
        if clave.lower() in SENSIBLES:
            evento[clave] = "***"
    return evento
```

Procesador que descarta eventos (por ejemplo, comprobaciones de salud):

```python
from structlog import DropEvent


def descartar_salud(logger, metodo, evento: dict) -> dict:
    if evento.get("path") == "/salud" and metodo == "info":
        raise DropEvent
    return evento
```

---

## 14. structlog: contexto ligado y `contextvars`

### 14.1 `bind`: contexto en un *logger*

```python
log = structlog.get_logger().bind(servicio="pagos", version="1.4.0")
log_pedido = log.bind(pedido_id=1042)

log_pedido.info("cobro_iniciado", importe=59.90)
log_pedido.info("cobro_completado", referencia="PAY-889")
# Ambos eventos incluyen servicio, version y pedido_id

log_pedido.unbind("pedido_id")
log.new(usuario="ana")         # Contexto nuevo (descarta el anterior)
```

### 14.2 `contextvars`: contexto por petición o tarea

Asocia datos a la **petición actual** (también en código asíncrono) sin pasar el *logger* de función en función:

```python
from structlog.contextvars import bind_contextvars, bound_contextvars, clear_contextvars

clear_contextvars()
bind_contextvars(request_id="7f3a9c", usuario_id=17)

servicio.crear_pedido(...)        # Todos los registros dentro incluyen request_id y usuario_id

with bound_contextvars(pedido_id=1042):     # Solo dentro del bloque
    log.info("pedido_validado")
```

Middleware para FastAPI/Starlette:

```python
import time
import uuid

from starlette.middleware.base import BaseHTTPMiddleware
from structlog.contextvars import bind_contextvars, clear_contextvars

log = structlog.get_logger("tienda.http")


class RegistroPeticiones(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        clear_contextvars()
        request_id = request.headers.get("x-request-id") or uuid.uuid4().hex
        bind_contextvars(request_id=request_id, metodo=request.method, ruta=request.url.path)
        inicio = time.perf_counter()
        try:
            respuesta = await call_next(request)
        except Exception:
            log.exception("peticion_fallida")
            raise
        respuesta.headers["x-request-id"] = request_id
        log.info("peticion_atendida", estado=respuesta.status_code,
                 duracion_ms=round((time.perf_counter() - inicio) * 1000, 1))
        return respuesta
```

---

## 15. structlog: integración con logging

En una aplicación real conviven sus registros (structlog) y los de las librerías (logging: uvicorn, SQLAlchemy, httpx). Lo ideal es que **todos** salgan con el mismo formato. Para ello, structlog se configura para entregar sus eventos a `logging`, y un `ProcessorFormatter` formatea ambos tipos de registro:

```python
import logging
import sys

import structlog

COMUNES = [
    structlog.contextvars.merge_contextvars,
    structlog.stdlib.add_log_level,
    structlog.stdlib.add_logger_name,
    structlog.processors.TimeStamper(fmt="iso", utc=True),
    structlog.stdlib.ExtraAdder(),               # Incluye los «extra=» de logging
]

structlog.configure(
    processors=[*COMUNES, structlog.stdlib.ProcessorFormatter.wrap_for_formatter],
    logger_factory=structlog.stdlib.LoggerFactory(),
    wrapper_class=structlog.stdlib.BoundLogger,
    cache_logger_on_first_use=True,
)

formateador = structlog.stdlib.ProcessorFormatter(
    foreign_pre_chain=COMUNES,                  # Procesa también los registros de logging
    processors=[
        structlog.stdlib.ProcessorFormatter.remove_processors_meta,
        structlog.processors.ExceptionRenderer(
            structlog.tracebacks.ExceptionDictTransformer(show_locals=False)),
        structlog.processors.JSONRenderer(),
    ],
)

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(formateador)
raiz = logging.getLogger()
raiz.handlers = [handler]
raiz.setLevel(logging.INFO)
```

```python
structlog.get_logger("tienda").info("pedido_creado", pedido_id=1)
logging.getLogger("httpx").warning("Reintentando petición")
```

```json
{"pedido_id": 1, "event": "pedido_creado", "level": "info", "logger": "tienda", "timestamp": "2026-11-26T09:30:00.120Z"}
{"event": "Reintentando petición", "level": "warning", "logger": "httpx", "timestamp": "2026-11-26T09:30:00.121Z"}
```

Para que uvicorn use esta configuración, arránquelo con `log_config=None` (o `--log-config` apuntando a una configuración vacía) y desactive su *logger* de acceso si lo sustituye por el middleware.

---

## 16. Pruebas de los registros

### 16.1 structlog: `capture_logs`

```python
from structlog.testing import capture_logs


def test_registra_pedido_creado():
    with capture_logs() as registros:
        servicio.crear("ana@ejemplo.com", lineas)
    assert {"event": "pedido_creado", "log_level": "info", "pedido_id": 1,
            "total": 59.9} in registros
```

### 16.2 logging: fixture `caplog` de pytest

```python
import logging


def test_aviso_stock(caplog):
    with caplog.at_level(logging.WARNING, logger="tienda.pedidos"):
        procesar_sin_stock()
    assert "Stock insuficiente" in caplog.text
    assert caplog.records[0].levelno == logging.WARNING
```

> Pruebe los registros **importantes para operación** (errores, eventos de negocio, auditoría), no todos.

---

## 17. Registros en producción

| Práctica | Detalle |
|---|---|
| Salida a `stdout` en JSON | La plataforma (Docker, Kubernetes, nube) recoge y envía los registros. Nada de archivos dentro del contenedor. |
| Identificador de correlación | `request_id` (o el `trace_id` de OpenTelemetry) en cada evento para seguir una petición entre servicios. |
| Nombres de eventos estables | `pedido_creado`, no frases que cambian. Permiten métricas y alertas. |
| Esquema de campos coherente | Mismos nombres y tipos en todo el sistema (`usuario_id` siempre entero). |
| Niveles bien elegidos | `INFO` para negocio, `WARNING` para lo recuperable, `ERROR` para fallos; `DEBUG` desactivado. |
| Sin datos sensibles | Nunca contraseñas, *tokens*, tarjetas o datos personales innecesarios (RGPD). Procesador de ocultación como red de seguridad. |
| Volumen controlado | Muestreo de eventos muy frecuentes; no registrar en bucles internos. |
| Hora en UTC e ISO 8601 | Evita ambigüedades entre servidores y zonas horarias. |
| Nivel configurable | Por variable de entorno (`LOG_NIVEL=DEBUG`) sin redesplegar código. |
| Excepciones estructuradas | Trazas en JSON para buscar por tipo de excepción, **sin variables locales**. |

---

## 18. Ejemplo práctico completo

**Módulo de registro para un servicio**: una función `configurar_registro()` que unifica structlog y logging, con salida legible en desarrollo y JSON en producción (según el entorno), ocultación de datos sensibles, contexto por petición con `contextvars`, campos de origen opcionales, silenciado de librerías ruidosas y pruebas con `capture_logs`. Se usa desde un servicio de pagos de ejemplo.

### 18.1 `src/tienda/registro.py`

```python
"""Configuración centralizada de registros (structlog + logging)."""
from __future__ import annotations

import logging
import os
import sys
from collections.abc import MutableMapping
from typing import Any

import structlog
from structlog.typing import EventDict, Processor, WrappedLogger

CLAVES_SENSIBLES = {"password", "contrasena", "token", "api_key", "authorization", "tarjeta", "iban", "cvv"}
LIBRERIAS_RUIDOSAS = {"httpx": logging.WARNING, "httpcore": logging.WARNING, "urllib3": logging.WARNING,
                      "asyncio": logging.WARNING, "uvicorn.access": logging.WARNING}


def ocultar_sensibles(_: WrappedLogger, __: str, evento: EventDict) -> EventDict:
    """Sustituye por *** los valores de claves sensibles, también en diccionarios anidados."""
    def limpiar(datos: MutableMapping[str, Any]) -> None:
        for clave, valor in datos.items():
            if clave.lower() in CLAVES_SENSIBLES:
                datos[clave] = "***"
            elif isinstance(valor, MutableMapping):
                limpiar(valor)
    limpiar(evento)
    return evento


def anadir_servicio(nombre: str, version: str) -> Processor:
    def procesador(_: WrappedLogger, __: str, evento: EventDict) -> EventDict:
        evento.setdefault("servicio", nombre)
        evento.setdefault("version", version)
        return evento
    return procesador


def configurar_registro(
    *,
    json: bool | None = None,
    nivel: str | int | None = None,
    servicio: str = "tienda",
    version: str = "dev",
    origen: bool = False,
) -> None:
    """Configura structlog y logging para que todos los registros compartan formato.

    Args:
        json: Salida JSON (producción). Por defecto, JSON salvo que ``ENTORNO=desarrollo``
            o la salida sea una terminal interactiva.
        nivel: Nivel mínimo. Por defecto, la variable ``LOG_NIVEL`` o ``INFO``.
        servicio: Nombre del servicio que se añade a cada evento.
        version: Versión del servicio.
        origen: Añadir archivo, función y línea de cada evento.
    """
    if json is None:
        json = os.getenv("ENTORNO", "produccion") != "desarrollo" and not sys.stdout.isatty()
    nivel = logging.getLevelName(nivel or os.getenv("LOG_NIVEL", "INFO")) if not isinstance(nivel, int) else nivel

    comunes: list[Processor] = [
        structlog.contextvars.merge_contextvars,
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso", utc=True),
        structlog.stdlib.ExtraAdder(),
        anadir_servicio(servicio, version),
        ocultar_sensibles,
    ]
    if origen:
        comunes.append(structlog.processors.CallsiteParameterAdder(
            {structlog.processors.CallsiteParameter.FILENAME,
             structlog.processors.CallsiteParameter.FUNC_NAME,
             structlog.processors.CallsiteParameter.LINENO}))

    # Excepciones como JSON SIN variables locales: podrían contener datos sensibles
    excepciones = structlog.processors.ExceptionRenderer(
        structlog.tracebacks.ExceptionDictTransformer(show_locals=False))
    renderizador: list[Processor] = (
        [excepciones, structlog.processors.JSONRenderer(ensure_ascii=False)]
        if json else [structlog.dev.ConsoleRenderer(colors=sys.stdout.isatty())]
    )

    structlog.configure(
        processors=[structlog.stdlib.filter_by_level, *comunes,
                    structlog.stdlib.ProcessorFormatter.wrap_for_formatter],
        logger_factory=structlog.stdlib.LoggerFactory(),
        wrapper_class=structlog.stdlib.BoundLogger,
        cache_logger_on_first_use=True,
    )

    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(structlog.stdlib.ProcessorFormatter(
        foreign_pre_chain=comunes,
        processors=[structlog.stdlib.ProcessorFormatter.remove_processors_meta, *renderizador],
    ))
    raiz = logging.getLogger()
    raiz.handlers = [handler]
    raiz.setLevel(nivel)
    for nombre, nivel_libreria in LIBRERIAS_RUIDOSAS.items():
        logging.getLogger(nombre).setLevel(nivel_libreria)
```

### 18.2 Uso en un servicio (`src/tienda/pagos.py`)

```python
from __future__ import annotations

import uuid
from decimal import Decimal

import structlog
from structlog.contextvars import bound_contextvars

log = structlog.get_logger(__name__)


class PagoRechazado(Exception):
    pass


def cobrar(pedido_id: int, importe: Decimal, tarjeta: str) -> str:
    with bound_contextvars(pedido_id=pedido_id):
        log.info("cobro_iniciado", importe=float(importe), tarjeta=tarjeta)   # «tarjeta» se oculta
        if importe > 1000:
            log.warning("cobro_rechazado", motivo="importe_maximo", limite=1000)
            raise PagoRechazado("Importe superior al máximo permitido")
        referencia = f"PAY-{uuid.uuid4().hex[:8]}"
        log.info("cobro_completado", referencia=referencia)
        return referencia
```

### 18.3 Programa de demostración (`demo_registro.py`)

```python
import logging
from decimal import Decimal

import structlog
from structlog.contextvars import bind_contextvars, clear_contextvars

from tienda.pagos import PagoRechazado, cobrar
from tienda.registro import configurar_registro

configurar_registro(json=True, servicio="tienda-api", version="1.4.0")
log = structlog.get_logger("tienda.demo")

clear_contextvars()
bind_contextvars(request_id="7f3a9c", usuario_id=17)

cobrar(1042, Decimal("59.90"), tarjeta="4111111111111111")
try:
    cobrar(1043, Decimal("1500"), tarjeta="5500000000000004")
except PagoRechazado:
    log.exception("pedido_no_cobrado", pedido_id=1043)

logging.getLogger("libreria.externa").warning("Mensaje de una librería que usa logging")
log.debug("no_se_muestra")                 # Nivel INFO por defecto
```

```text
$ ENTORNO=produccion python demo_registro.py
{"importe": 59.9, "tarjeta": "***", "event": "cobro_iniciado", "usuario_id": 17, "request_id": "7f3a9c", "pedido_id": 1042, "level": "info", "logger": "tienda.pagos", "timestamp": "2026-11-26T16:57:51.603082Z", "servicio": "tienda-api", "version": "1.4.0"}
{"referencia": "PAY-ca359e3b", "event": "cobro_completado", "usuario_id": 17, "request_id": "7f3a9c", "pedido_id": 1042, "level": "info", "logger": "tienda.pagos", "timestamp": "2026-11-26T16:57:51.603319Z", "servicio": "tienda-api", "version": "1.4.0"}
{"importe": 1500.0, "tarjeta": "***", "event": "cobro_iniciado", "usuario_id": 17, "request_id": "7f3a9c", "pedido_id": 1043, "level": "info", "logger": "tienda.pagos", "timestamp": "2026-11-26T16:57:51.603428Z", "servicio": "tienda-api", "version": "1.4.0"}
{"motivo": "importe_maximo", "limite": 1000, "event": "cobro_rechazado", "usuario_id": 17, "request_id": "7f3a9c", "pedido_id": 1043, "level": "warning", "logger": "tienda.pagos", "timestamp": "2026-11-26T16:57:51.603490Z", "servicio": "tienda-api", "version": "1.4.0"}
{"pedido_id": 1043, "event": "pedido_no_cobrado", "usuario_id": 17, "request_id": "7f3a9c", "level": "error", "logger": "tienda.demo", "timestamp": "2026-11-26T16:57:51.603569Z", "servicio": "tienda-api", "version": "1.4.0", "exception": [{"exc_type": "PagoRechazado", "exc_value": "Importe superior al máximo permitido", "frames": ["…"]}]}
{"event": "Mensaje de una librería que usa logging", "usuario_id": 17, "request_id": "7f3a9c", "level": "warning", "logger": "libreria.externa", "timestamp": "2026-11-26T16:57:51.603732Z", "servicio": "tienda-api", "version": "1.4.0"}
```

Observe que: la tarjeta aparece como `***`; el `request_id` y el `usuario_id` se añaden a todos los eventos, incluido el de la librería que usa `logging`; `pedido_id` se añade solo dentro del bloque `bound_contextvars`; y la excepción es una estructura JSON consultable (aquí abreviada), **sin variables locales**.

> ⚠ **Variables locales en las trazas.** `dict_tracebacks` incluye por defecto las variables locales de cada marco de la pila. Al probar este ejemplo con esa opción, el número de tarjeta completo aparecía dentro de la traza de la excepción, saltándose el procesador de ocultación. Por eso `configurar_registro` usa `ExceptionDictTransformer(show_locals=False)`.

En desarrollo (`configurar_registro(json=False)`), la misma ejecución se muestra en columnas legibles y con colores.

### 18.4 Pruebas (`tests/test_registro.py`)

```python
from decimal import Decimal

import pytest
from structlog.testing import capture_logs

from tienda.pagos import PagoRechazado, cobrar
from tienda.registro import ocultar_sensibles


def test_cobro_registra_eventos():
    with capture_logs() as registros:
        referencia = cobrar(1, Decimal("10"), tarjeta="4111111111111111")
    eventos = [r["event"] for r in registros]
    assert eventos == ["cobro_iniciado", "cobro_completado"]
    assert registros[1]["referencia"] == referencia


def test_cobro_rechazado_registra_aviso():
    with capture_logs() as registros, pytest.raises(PagoRechazado):
        cobrar(2, Decimal("5000"), tarjeta="4111111111111111")
    assert registros[-1] == {"event": "cobro_rechazado", "log_level": "warning",
                             "motivo": "importe_maximo", "limite": 1000}


def test_ocultar_sensibles_anidados():
    evento = {"event": "x", "token": "abc", "cliente": {"iban": "ES00", "nombre": "Ana"}}
    assert ocultar_sensibles(None, "info", evento) == {
        "event": "x", "token": "***", "cliente": {"iban": "***", "nombre": "Ana"}}
```

```text
$ uv run pytest tests/test_registro.py -q
...                                                                      [100%]
3 passed in 0.05s
```

> `capture_logs` captura los eventos **antes** de los procesadores, por eso en las pruebas la tarjeta aparece sin ocultar y el nivel figura como `log_level`. La ocultación se prueba por separado.

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| No aparece ningún registro | Nivel raíz `WARNING` por defecto, o no se configuró ningún *handler*. | `basicConfig(level=logging.INFO)` o la configuración del proyecto. |
| Registros duplicados | Varios *handlers* (p. ej. en el *logger* y en el raíz) con propagación. | Un solo punto de configuración; `propagate = False` si hay *handler* propio. |
| `basicConfig` no tiene efecto | Algún módulo ya configuró el raíz. | `basicConfig(..., force=True)` o `dictConfig`. |
| Los *loggers* de módulos importados dejan de emitir | `dictConfig` con `disable_existing_loggers=True` (por defecto). | Ponga `False`. |
| `TypeError: not all arguments converted during string formatting` | Mezcla de `%s` y argumentos incorrectos. | Revise el número de `%s` y argumentos. |
| Los registros de las librerías no salen en JSON | structlog configurado sin integración con `logging`. | `ProcessorFormatter` con `foreign_pre_chain` (sección 15). |
| El contexto de una petición aparece en otra | No se limpiaron las `contextvars`. | `clear_contextvars()` al inicio de cada petición/tarea. |
| Registros fuera de orden o perdidos en contenedores | Salida con búfer. | `PYTHONUNBUFFERED=1`; escribir en `stdout`. |
| La configuración de structlog no cambia en las pruebas | `cache_logger_on_first_use=True` con *loggers* ya usados. | `structlog.reset_defaults()` en las pruebas o configure antes de crear *loggers*. |
| Datos sensibles en los registros | Se registraron objetos completos o cabeceras. | Registre campos concretos y use el procesador de ocultación. |
| Datos sensibles dentro de las trazas JSON | `dict_tracebacks` incluye las variables locales de la pila. | `ExceptionRenderer(ExceptionDictTransformer(show_locals=False))`. |

---

## 20. Buenas prácticas

- Un *logger* por módulo: `logging.getLogger(__name__)` o `structlog.get_logger(__name__)`.
- **Configuración en un único punto** al arrancar la aplicación; nunca en librerías (solo `NullHandler`).
- En producción: **JSON a `stdout`**, hora UTC ISO 8601 y nivel configurable por variable de entorno.
- Unifique structlog y `logging` para que los registros de las librerías tengan el mismo formato.
- Eventos con **nombre estable** y datos como pares clave-valor; nombres de campo coherentes.
- Contexto por petición con `contextvars` (`request_id`, `usuario_id`) y limpieza al empezar cada petición.
- `log.exception` dentro de `except`, una sola vez por error, donde se gestiona.
- Formato perezoso en `logging` (`%s`), nunca *f-strings* en las llamadas.
- **Nunca** registre contraseñas, *tokens*, tarjetas ni datos personales innecesarios; añada un procesador de ocultación.
- Silencie librerías ruidosas por nivel, no eliminando sus registros.
- Pruebe los registros relevantes para operación con `capture_logs` o `caplog`.

---

## 21. Referencia rápida

```python
# logging
log = logging.getLogger(__name__)
log.debug/info/warning/error/critical("texto %s", valor) · log.exception("…") (en except)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(name)s: %(message)s", force=True)
logging.config.dictConfig({...,"disable_existing_loggers": False})
logging.getLogger("libreria").setLevel(logging.WARNING)
logging.getLogger(__name__).addHandler(logging.NullHandler())      # En librerías

# structlog
log = structlog.get_logger(__name__)
log.info("evento", clave=valor) · log.bind(k=v) · log.exception("evento")
structlog.configure(processors=[...], wrapper_class=..., logger_factory=..., cache_logger_on_first_use=True)
structlog.contextvars.clear_contextvars() / bind_contextvars(k=v) / bound_contextvars(k=v)
structlog.processors.TimeStamper(fmt="iso", utc=True) · add_log_level · ExceptionRenderer(ExceptionDictTransformer(show_locals=False)) · JSONRenderer()
structlog.dev.ConsoleRenderer() · structlog.stdlib.ProcessorFormatter(foreign_pre_chain=[...], processors=[...])
from structlog.testing import capture_logs
```

---

## 22. Ejercicios propuestos

1. Configure `logging` con `dictConfig` para enviar `INFO` a la consola y `DEBUG` a un archivo rotativo, con `httpx` limitado a `WARNING`.
2. Escriba un formateador JSON para `logging` que incluya los campos de `extra` y compárelo con el de structlog.
3. Añada el middleware de `request_id` a una API FastAPI y verifique que todos los registros de una petición comparten el identificador.
4. Amplíe `ocultar_sensibles` para enmascarar también números de tarjeta que aparezcan dentro de textos (dejando visibles los últimos cuatro dígitos).
5. Envíe los registros JSON de un contenedor a Grafana Loki (o al servicio de su nube) y cree una consulta que cuente `cobro_rechazado` por `motivo`.

---

## 23. Referencias

- `logging`, documentación: https://docs.python.org/es/3/library/logging.html
- Guía práctica de logging: https://docs.python.org/es/3/howto/logging.html
- Recetario de logging: https://docs.python.org/es/3/howto/logging-cookbook.html
- structlog, documentación: https://www.structlog.org/
- structlog y la librería estándar: https://www.structlog.org/en/stable/standard-library.html
- structlog, pruebas: https://www.structlog.org/en/stable/testing.html
- OpenTelemetry para Python: https://opentelemetry.io/docs/languages/python/
