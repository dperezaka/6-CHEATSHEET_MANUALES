# Manual de uso: Logging

> **Categoría:** Utilidades generales · **Utilidad:** Registrar eventos y errores
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Niveles de registro](#4-niveles-de-registro)
5. [Operaciones básicas](#5-operaciones-básicas)
6. [Configuración intermedia](#6-configuración-intermedia)
7. [Configuración avanzada](#7-configuración-avanzada)
8. [Ejemplo práctico completo](#8-ejemplo-práctico-completo)
9. [Errores frecuentes](#9-errores-frecuentes)
10. [Buenas prácticas](#10-buenas-prácticas)
11. [Referencia rápida](#11-referencia-rápida)
12. [Ejercicios propuestos](#12-ejercicios-propuestos)
13. [Referencias](#13-referencias)

---

## 1. Descripción

El módulo `logging` proporciona un sistema flexible para registrar lo que ocurre durante la ejecución de un programa: información de seguimiento, advertencias, errores y excepciones. A diferencia de `print()`, permite:

- Clasificar los mensajes por **nivel de gravedad**.
- Enviarlos a varios destinos (consola, archivos, correo, servicios externos).
- Añadir fecha, módulo, línea y otros datos de contexto automáticamente.
- Activar o desactivar mensajes sin modificar el código.

---

## 2. Instalación e importación

No requiere instalación.

```python
import logging
```

---

## 3. Conceptos clave

| Componente | Función |
|---|---|
| **Logger** | Objeto al que se envían los mensajes (`logging.getLogger(__name__)`). |
| **Handler** | Decide **dónde** se envían: consola, archivo, red… |
| **Formatter** | Decide el **formato** del mensaje. |
| **Filter** | Permite filtrar mensajes con reglas personalizadas. |
| **Level** | Nivel mínimo de gravedad que se procesa. |

**Flujo:** `logger.info("...")` → el logger comprueba el nivel → lo pasa a sus handlers → cada handler aplica su formato y lo escribe.

Los loggers forman una **jerarquía** por nombre con puntos: `app`, `app.bd`, `app.api`. Los mensajes se propagan hacia el logger raíz.

---

## 4. Niveles de registro

| Nivel | Valor | Cuándo usarlo |
|---|---|---|
| `DEBUG` | 10 | Detalle técnico para depuración. |
| `INFO` | 20 | Confirmación de que todo funciona según lo esperado. |
| `WARNING` | 30 | Algo inesperado, pero el programa continúa. *(Nivel por defecto)* |
| `ERROR` | 40 | Un fallo impide realizar una operación. |
| `CRITICAL` | 50 | Error grave; el programa puede no continuar. |

Solo se muestran los mensajes con nivel **igual o superior** al configurado.

---

## 5. Operaciones básicas

### 5.1 Uso inmediato

```python
logging.warning("Espacio en disco bajo")      # Se muestra
logging.info("Proceso iniciado")              # NO se muestra (nivel por defecto WARNING)
```

### 5.2 Configuración básica con `basicConfig`

```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.debug("No se muestra")
logging.info("Aplicación iniciada")
logging.warning("Configuración no encontrada, usando valores por defecto")
logging.error("No se pudo conectar con la base de datos")
logging.critical("Fallo crítico del sistema")
```

Salida:

```
2026-11-24 21:40:12 | INFO     | root | Aplicación iniciada
2026-11-24 21:40:12 | WARNING  | root | Configuración no encontrada, usando valores por defecto
```

> `basicConfig` solo surte efecto la **primera vez** que se llama. Llámelo al inicio del programa.

### 5.3 Guardar en un archivo

```python
logging.basicConfig(
    filename="app.log",
    filemode="a",                  # "a" añadir, "w" sobrescribir
    encoding="utf-8",
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)
```

### 5.4 Loggers con nombre (recomendado)

```python
logger = logging.getLogger(__name__)

def procesar_pedido(pedido_id):
    logger.info("Procesando pedido %s", pedido_id)
```

Cada módulo usa su propio logger y el nombre aparece en el registro, lo que facilita saber de dónde viene cada mensaje.

### 5.5 Mensajes con variables

```python
logger.info("Usuario %s ha iniciado sesión desde %s", usuario, ip)
```

> Use el formato `%s` con argumentos en lugar de f-strings: el texto solo se construye si el mensaje se va a mostrar.

### 5.6 Registrar excepciones con su traza

```python
try:
    resultado = 10 / 0
except ZeroDivisionError:
    logger.exception("Error al calcular el resultado")   # Nivel ERROR + traza completa
```

Equivalente con otro nivel:

```python
logger.warning("Fallo no crítico", exc_info=True)
```

---

## 6. Configuración intermedia

### 6.1 Atributos disponibles para el formato

| Atributo | Contenido |
|---|---|
| `%(asctime)s` | Fecha y hora |
| `%(levelname)s` | Nivel |
| `%(name)s` | Nombre del logger |
| `%(message)s` | Mensaje |
| `%(module)s` | Módulo |
| `%(funcName)s` | Función |
| `%(lineno)d` | Línea |
| `%(filename)s` | Archivo |
| `%(process)d` / `%(thread)d` | Proceso / hilo |

### 6.2 Consola y archivo a la vez, con niveles distintos

```python
logger = logging.getLogger("app")
logger.setLevel(logging.DEBUG)

formato = logging.Formatter(
    "%(asctime)s | %(levelname)-8s | %(name)s:%(lineno)d | %(message)s",
    datefmt="%d/%m/%Y %H:%M:%S",
)

consola = logging.StreamHandler()
consola.setLevel(logging.INFO)
consola.setFormatter(formato)

archivo = logging.FileHandler("app.log", encoding="utf-8")
archivo.setLevel(logging.DEBUG)
archivo.setFormatter(formato)

logger.addHandler(consola)
logger.addHandler(archivo)

logger.debug("Solo en el archivo")
logger.info("En consola y en archivo")
```

### 6.3 Rotación de archivos por tamaño

```python
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    "app.log", maxBytes=5 * 1024 * 1024, backupCount=5, encoding="utf-8"
)
# Genera app.log, app.log.1, …, app.log.5 (máx. 5 MB cada uno)
```

### 6.4 Rotación por tiempo

```python
from logging.handlers import TimedRotatingFileHandler

handler = TimedRotatingFileHandler(
    "app.log", when="midnight", interval=1, backupCount=30, encoding="utf-8"
)
# Un archivo por día, conserva 30 días
```

### 6.5 Silenciar librerías ruidosas

```python
logging.getLogger("urllib3").setLevel(logging.WARNING)
logging.getLogger("sqlalchemy.engine").setLevel(logging.WARNING)
```

---

## 7. Configuración avanzada

### 7.1 Configuración centralizada con `dictConfig`

```python
import logging.config

CONFIG_LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "estandar": {
            "format": "%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
            "datefmt": "%Y-%m-%d %H:%M:%S",
        },
    },
    "handlers": {
        "consola": {
            "class": "logging.StreamHandler",
            "level": "INFO",
            "formatter": "estandar",
        },
        "archivo": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "DEBUG",
            "formatter": "estandar",
            "filename": "app.log",
            "maxBytes": 5_000_000,
            "backupCount": 3,
            "encoding": "utf-8",
        },
        "errores": {
            "class": "logging.FileHandler",
            "level": "ERROR",
            "formatter": "estandar",
            "filename": "errores.log",
            "encoding": "utf-8",
        },
    },
    "loggers": {
        "app": {"level": "DEBUG", "handlers": ["consola", "archivo", "errores"], "propagate": False},
    },
    "root": {"level": "WARNING", "handlers": ["consola"]},
}

logging.config.dictConfig(CONFIG_LOGGING)
logger = logging.getLogger("app")
```

Esta configuración puede guardarse en un archivo JSON o YAML y cargarse al inicio.

### 7.2 Filtros personalizados

```python
class SoloErroresDeBD(logging.Filter):
    def filter(self, record: logging.LogRecord) -> bool:
        return record.name.startswith("app.bd")

handler.addFilter(SoloErroresDeBD())
```

### 7.3 Añadir contexto con `extra` y `LoggerAdapter`

```python
formato = logging.Formatter("%(asctime)s [%(usuario)s] %(message)s")

logger.info("Pedido creado", extra={"usuario": "ana"})

adaptador = logging.LoggerAdapter(logger, {"usuario": "ana"})
adaptador.info("Pedido creado")
```

### 7.4 Formato JSON (para sistemas de monitorización)

```python
import json

class FormatoJSON(logging.Formatter):
    def format(self, record):
        datos = {
            "fecha": self.formatTime(record, "%Y-%m-%dT%H:%M:%S"),
            "nivel": record.levelname,
            "logger": record.name,
            "mensaje": record.getMessage(),
            "modulo": record.module,
            "linea": record.lineno,
        }
        if record.exc_info:
            datos["excepcion"] = self.formatException(record.exc_info)
        return json.dumps(datos, ensure_ascii=False)
```

### 7.5 Enviar errores críticos por correo

```python
from logging.handlers import SMTPHandler

correo = SMTPHandler(
    mailhost=("smtp.empresa.com", 587),
    fromaddr="alertas@empresa.com",
    toaddrs=["soporte@empresa.com"],
    subject="Error crítico en la aplicación",
    credentials=("usuario", "clave"),
    secure=(),
)
correo.setLevel(logging.CRITICAL)
```

### 7.6 Registrar excepciones no capturadas

```python
import sys

def registrar_excepcion(tipo, valor, traza):
    if issubclass(tipo, KeyboardInterrupt):
        sys.__excepthook__(tipo, valor, traza)
        return
    logging.getLogger("app").critical("Excepción no controlada", exc_info=(tipo, valor, traza))

sys.excepthook = registrar_excepcion
```

---

## 8. Ejemplo práctico completo

Estructura de proyecto con configuración centralizada y loggers por módulo.

**`config_logging.py`**

```python
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path


def configurar_logging(nivel_consola=logging.INFO, carpeta="logs"):
    Path(carpeta).mkdir(exist_ok=True)

    formato = logging.Formatter(
        "%(asctime)s | %(levelname)-8s | %(name)-15s | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

    consola = logging.StreamHandler()
    consola.setLevel(nivel_consola)
    consola.setFormatter(formato)

    general = RotatingFileHandler(f"{carpeta}/app.log", maxBytes=2_000_000,
                                  backupCount=5, encoding="utf-8")
    general.setLevel(logging.DEBUG)
    general.setFormatter(formato)

    errores = logging.FileHandler(f"{carpeta}/errores.log", encoding="utf-8")
    errores.setLevel(logging.ERROR)
    errores.setFormatter(formato)

    raiz = logging.getLogger()
    raiz.setLevel(logging.DEBUG)
    raiz.handlers.clear()                 # Evita duplicados si se llama dos veces
    for h in (consola, general, errores):
        raiz.addHandler(h)

    logging.getLogger("urllib3").setLevel(logging.WARNING)
```

**`pedidos.py`**

```python
import logging

logger = logging.getLogger(__name__)


def procesar(pedido: dict) -> float:
    logger.debug("Datos recibidos: %s", pedido)
    if pedido["unidades"] <= 0:
        logger.warning("Pedido %s con unidades no válidas", pedido["id"])
        return 0.0
    total = pedido["unidades"] * pedido["precio"]
    logger.info("Pedido %s procesado: %.2f €", pedido["id"], total)
    return total
```

**`main.py`**

```python
import logging
from config_logging import configurar_logging
import pedidos

configurar_logging()
logger = logging.getLogger("main")

lista = [
    {"id": 1, "unidades": 3, "precio": 19.9},
    {"id": 2, "unidades": 0, "precio": 9.5},
    {"id": 3, "unidades": 2},                      # Falta el precio
]

logger.info("Inicio del procesamiento de %d pedidos", len(lista))
total = 0.0
for p in lista:
    try:
        total += pedidos.procesar(p)
    except KeyError as e:
        logger.exception("Pedido %s incompleto: falta %s", p.get("id"), e)

logger.info("Procesamiento finalizado. Total: %.2f €", total)
```

---

## 9. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| No aparecen los mensajes `INFO` o `DEBUG` | Nivel por defecto `WARNING`. | Configure `level=logging.DEBUG` o `INFO`. |
| `basicConfig` no tiene efecto | Ya se configuró antes (o lo hizo una librería). | Llámelo al inicio o use `force=True`. |
| Mensajes duplicados | Handlers añadidos varias veces o propagación al raíz. | Use `handlers.clear()` o `propagate=False`. |
| Tildes incorrectas en el archivo | Codificación por defecto del sistema. | Indique `encoding="utf-8"`. |
| Se pierde la traza del error | Uso de `logger.error(str(e))`. | Use `logger.exception(...)` dentro del `except`. |
| `KeyError` con `extra` | El formato usa un campo que no siempre se envía. | Envíe siempre el campo o use un `Filter` que lo añada. |

---

## 10. Buenas prácticas

- Use `logger = logging.getLogger(__name__)` en cada módulo.
- Configure el logging **una sola vez**, en el punto de entrada de la aplicación.
- Las librerías **no** deben configurar handlers; solo crear loggers.
- Elija el nivel adecuado: no registre todo como `ERROR`.
- No registre contraseñas, tokens ni datos personales sensibles.
- Use rotación de archivos para evitar que los logs crezcan sin límite.
- Sustituya los `print()` de depuración por `logger.debug()`.

---

## 11. Referencia rápida

```python
logging.basicConfig(level=..., format=..., filename=..., encoding="utf-8", force=True)
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
logger.debug / info / warning / error / critical("Texto %s", valor)
logger.exception("Texto")                       # Dentro de except
handler = logging.StreamHandler() | FileHandler(...) | RotatingFileHandler(...)
handler.setLevel(...); handler.setFormatter(logging.Formatter("..."))
logger.addHandler(handler)
logging.config.dictConfig({...})
```

---

## 12. Ejercicios propuestos

1. Configure una aplicación que muestre `INFO` en consola y guarde `DEBUG` en un archivo con rotación diaria.
2. Cree un decorador `@registrar` que anote en el log la entrada, salida y duración de cualquier función.
3. Implemente un formateador JSON y compruebe que cada línea del archivo es un JSON válido.
4. Escriba un script que lea `app.log` y cuente cuántos mensajes hay de cada nivel.

---

## 13. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/logging.html
- Guía práctica (Logging HOWTO): https://docs.python.org/es/3/howto/logging.html
