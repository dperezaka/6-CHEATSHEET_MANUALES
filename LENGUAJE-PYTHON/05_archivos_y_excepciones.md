# Manual de uso: Archivos, excepciones y registro en Python

> **Categoría:** Programación · **Utilidad:** Leer y escribir archivos, gestionar errores y registrar eventos
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Conceptos clave](#2-conceptos-clave)
3. [Excepciones](#3-excepciones)
4. [Excepciones propias](#4-excepciones-propias)
5. [Rutas con `pathlib`](#5-rutas-con-pathlib)
6. [Archivos de texto](#6-archivos-de-texto)
7. [CSV](#7-csv)
8. [JSON](#8-json)
9. [Otros formatos](#9-otros-formatos)
10. [Gestores de contexto](#10-gestores-de-contexto)
11. [Operaciones con archivos y carpetas](#11-operaciones-con-archivos-y-carpetas)
12. [Registro de eventos (`logging`)](#12-registro-de-eventos-logging)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

Casi todo programa real lee o escribe datos y debe reaccionar ante situaciones imprevistas: un archivo que no existe, un dato mal formado, una conexión que falla. Python ofrece para ello:

- **Excepciones** (`try` / `except`) para detectar y gestionar errores.
- **`pathlib`** para trabajar con rutas de forma portable.
- Módulos **`csv`**, **`json`**, **`sqlite3`**, **`zipfile`**… para formatos habituales.
- **`logging`** para registrar lo que ocurre durante la ejecución.

---

## 2. Conceptos clave

| Concepto | Descripción |
|---|---|
| Excepción | Objeto que representa un error durante la ejecución. |
| Lanzar (`raise`) | Generar una excepción. |
| Capturar (`except`) | Interceptar una excepción para gestionarla. |
| *Traceback* | Informe que muestra la cadena de llamadas que llevó al error. |
| Ruta | Ubicación de un archivo: absoluta (`C:\datos\a.txt`) o relativa (`datos/a.txt`). |
| Codificación | Forma de convertir texto en bytes. Use **UTF-8**. |
| Modo de apertura | `r` (leer), `w` (escribir), `a` (añadir), `b` (binario)… |
| Gestor de contexto | Objeto usado con `with` que libera recursos automáticamente. |
| Serialización | Convertir objetos a un formato almacenable (JSON, CSV…). |
| *Log* | Registro de eventos con nivel, fecha y mensaje. |

---

## 3. Excepciones

### 3.1 Estructura completa

```python
try:
    edad = int(input("Edad: "))
    ratio = 100 / edad
except ValueError:
    print("Debe introducir un número entero.")
except ZeroDivisionError:
    print("La edad no puede ser cero.")
else:
    print(f"Resultado: {ratio:.2f}")        # Solo si NO hubo excepción
finally:
    print("Fin del cálculo")                 # Siempre se ejecuta
```

| Bloque | Se ejecuta… |
|---|---|
| `try` | Siempre: contiene el código que puede fallar. |
| `except` | Si se produce la excepción indicada. |
| `else` | Si no se produjo ninguna excepción. |
| `finally` | Siempre, haya o no error (liberar recursos). |

### 3.2 Capturar varias excepciones y acceder al error

```python
try:
    valor = datos["importe"] / datos["unidades"]
except (KeyError, ZeroDivisionError) as e:
    print(f"Dato incorrecto: {type(e).__name__}: {e}")
```

### 3.3 Excepciones integradas más comunes

| Excepción | Cuándo se produce |
|---|---|
| `ValueError` | Valor del tipo correcto pero inválido (`int("abc")`). |
| `TypeError` | Operación con un tipo inadecuado (`"a" + 1`). |
| `KeyError` | Clave inexistente en un diccionario. |
| `IndexError` | Índice fuera de rango. |
| `AttributeError` | Atributo o método inexistente. |
| `ZeroDivisionError` | División entre cero. |
| `FileNotFoundError` | Archivo o carpeta inexistente. |
| `PermissionError` | Sin permisos sobre el archivo. |
| `IsADirectoryError` | Se esperaba un archivo y es una carpeta. |
| `UnicodeDecodeError` | Codificación del archivo incorrecta. |
| `OSError` | Error genérico del sistema (padre de los errores de archivo). |
| `ImportError` / `ModuleNotFoundError` | Módulo no encontrado. |
| `KeyboardInterrupt` | El usuario pulsó `Ctrl+C`. |

> Jerarquía: `BaseException` → `Exception` → (`ValueError`, `OSError`, `LookupError` → `KeyError`/`IndexError`, …). Capturar una clase padre captura también sus hijas.

### 3.4 Lanzar excepciones

```python
def retirar(saldo: float, importe: float) -> float:
    if importe <= 0:
        raise ValueError(f"Importe no válido: {importe}")
    if importe > saldo:
        raise ValueError("Saldo insuficiente")
    return saldo - importe
```

### 3.5 Relanzar y encadenar

```python
try:
    config = json.loads(texto)
except json.JSONDecodeError as e:
    raise ConfiguracionInvalida("El archivo de configuración no es JSON válido") from e
```

```python
except OSError:
    logger.exception("Fallo al guardar")
    raise                                     # Relanza la misma excepción
```

### 3.6 Grupos de excepciones (`except*`)

Para gestionar varios errores simultáneos (por ejemplo, en tareas concurrentes):

```python
try:
    raise ExceptionGroup("Errores de validación", [ValueError("email"), KeyError("nombre")])
except* ValueError as grupo:
    print("Valores incorrectos:", grupo.exceptions)
except* KeyError as grupo:
    print("Campos ausentes:", grupo.exceptions)
```

### 3.7 Pedir perdón o pedir permiso

```python
# LBYL (Look Before You Leap): comprobar antes
if clave in datos:
    valor = datos[clave]

# EAFP (Easier to Ask Forgiveness than Permission): estilo habitual en Python
try:
    valor = datos[clave]
except KeyError:
    valor = None
```

> El estilo **EAFP** es idiomático en Python, sobre todo con archivos: entre comprobar que existe y abrirlo, otro proceso podría borrarlo.

---

## 4. Excepciones propias

```python
class ErrorAplicacion(Exception):
    """Base de todas las excepciones de la aplicación."""


class ClienteNoEncontrado(ErrorAplicacion):
    def __init__(self, id_cliente: int) -> None:
        super().__init__(f"No existe el cliente con id {id_cliente}")
        self.id_cliente = id_cliente


class DatosInvalidos(ErrorAplicacion):
    def __init__(self, errores: dict[str, str]) -> None:
        super().__init__("; ".join(f"{campo}: {msg}" for campo, msg in errores.items()))
        self.errores = errores


try:
    raise ClienteNoEncontrado(42)
except ErrorAplicacion as e:              # Captura cualquier error propio
    print(e)
```

> Defina una excepción base por aplicación o librería: permite capturar todos sus errores sin capturar errores de programación (`TypeError`, `NameError`…).

---

## 5. Rutas con `pathlib`

`pathlib` es la forma moderna y portable (Windows/Linux/macOS) de manejar rutas.

```python
from pathlib import Path

base = Path(__file__).resolve().parent         # Carpeta del script actual
datos = base / "datos" / "ventas.csv"           # Unir rutas con /
Path.home()                                     # Carpeta del usuario
Path.cwd()                                      # Carpeta de trabajo actual

datos.name          # 'ventas.csv'
datos.stem          # 'ventas'
datos.suffix        # '.csv'
datos.parent        # .../datos
datos.with_suffix(".xlsx")
datos.with_stem("ventas_2026")

datos.exists(), datos.is_file(), datos.is_dir()
datos.stat().st_size                            # Tamaño en bytes
datos.stat().st_mtime                           # Fecha de modificación (timestamp)
```

### 5.1 Listar y buscar archivos

```python
carpeta = Path("informes")
for archivo in carpeta.iterdir():               # Contenido directo
    print(archivo.name)

list(carpeta.glob("*.pdf"))                     # PDF de la carpeta
list(carpeta.rglob("*.csv"))                    # CSV en todas las subcarpetas
sorted(carpeta.glob("*.log"), key=lambda p: p.stat().st_mtime, reverse=True)   # Más recientes primero
```

### 5.2 Lectura y escritura rápidas

```python
texto = Path("nota.txt").read_text(encoding="utf-8")
Path("salida.txt").write_text("Contenido", encoding="utf-8")
bytes_ = Path("imagen.png").read_bytes()
```

> Indique **siempre** `encoding="utf-8"`. Sin él, Windows puede usar otra codificación y provocar `UnicodeDecodeError` con tildes y eñes.

---

## 6. Archivos de texto

### 6.1 `open` y modos

```python
with open("registro.txt", "r", encoding="utf-8") as f:
    contenido = f.read()
```

| Modo | Descripción |
|---|---|
| `"r"` | Lectura (por defecto). Error si no existe. |
| `"w"` | Escritura. **Crea o vacía** el archivo. |
| `"a"` | Añade al final. Lo crea si no existe. |
| `"x"` | Creación exclusiva. Error si ya existe. |
| `"b"` | Binario (`"rb"`, `"wb"`): imágenes, PDF, ZIP… |
| `"+"` | Lectura y escritura (`"r+"`). |

### 6.2 Lectura

```python
with open("datos.txt", encoding="utf-8") as f:
    todo = f.read()                  # Todo el contenido
with open("datos.txt", encoding="utf-8") as f:
    lineas = f.readlines()           # Lista de líneas (con \n)
with open("datos.txt", encoding="utf-8") as f:
    for linea in f:                  # ✔ Línea a línea: eficiente con archivos grandes
        procesar(linea.rstrip("\n"))
```

### 6.3 Escritura

```python
with open("informe.txt", "w", encoding="utf-8") as f:
    f.write("Informe de ventas\n")
    f.writelines(f"{p}: {v}\n" for p, v in ventas.items())
    print("Total:", total, file=f)   # print también escribe en archivos
```

### 6.4 Codificaciones

```python
open("excel_es.csv", encoding="utf-8-sig")       # UTF-8 con BOM (lo genera y lo espera Excel)
open("antiguo.txt", encoding="cp1252")           # Windows Europa occidental
open("dudoso.txt", encoding="utf-8", errors="replace")   # Sustituye bytes inválidos por �
```

---

## 7. CSV

### 7.1 Lectura

```python
import csv

with open("clientes.csv", newline="", encoding="utf-8-sig") as f:
    lector = csv.DictReader(f, delimiter=";")    # Cada fila es un diccionario
    for fila in lector:
        print(fila["nombre"], fila["email"])
```

```python
with open("datos.csv", newline="", encoding="utf-8") as f:
    lector = csv.reader(f)
    cabecera = next(lector)
    filas = list(lector)                          # Listas de cadenas
```

### 7.2 Escritura

```python
clientes = [{"nombre": "Ana", "email": "ana@ejemplo.com", "saldo": 120.5}]

with open("salida.csv", "w", newline="", encoding="utf-8-sig") as f:
    escritor = csv.DictWriter(f, fieldnames=["nombre", "email", "saldo"], delimiter=";")
    escritor.writeheader()
    escritor.writerows(clientes)
```

> - Abra los CSV con `newline=""` para evitar líneas en blanco en Windows.
> - Excel en español espera **`;`** como separador y **`utf-8-sig`** para mostrar bien las tildes.
> - El módulo `csv` devuelve siempre **cadenas**: convierta los números explícitamente.

### 7.3 Detectar el separador

```python
with open("desconocido.csv", newline="", encoding="utf-8") as f:
    dialecto = csv.Sniffer().sniff(f.read(2048))
    f.seek(0)
    lector = csv.reader(f, dialecto)
```

---

## 8. JSON

```python
import json

datos = {"cliente": "Ana", "pedidos": [1001, 1002], "vip": True, "saldo": None}

# Objeto Python → texto JSON
texto = json.dumps(datos, ensure_ascii=False, indent=2)

# Texto JSON → objeto Python
datos = json.loads(texto)

# Archivos
with open("datos.json", "w", encoding="utf-8") as f:
    json.dump(datos, f, ensure_ascii=False, indent=2)

with open("datos.json", encoding="utf-8") as f:
    datos = json.load(f)
```

| Python | JSON |
|---|---|
| `dict` | objeto `{}` |
| `list`, `tuple` | array `[]` |
| `str` | cadena |
| `int`, `float` | número |
| `True` / `False` | `true` / `false` |
| `None` | `null` |

### 8.1 Tipos no soportados (fechas, `Decimal`, objetos)

```python
from datetime import date
from decimal import Decimal

def serializar(obj):
    if isinstance(obj, (date, datetime)):
        return obj.isoformat()
    if isinstance(obj, Decimal):
        return str(obj)
    raise TypeError(f"Tipo no serializable: {type(obj).__name__}")

json.dumps({"fecha": date.today(), "importe": Decimal("9.99")}, default=serializar)
```

> `ensure_ascii=False` conserva las tildes en lugar de escribir `\u00e1`.

---

## 9. Otros formatos

| Formato | Módulo / librería |
|---|---|
| Configuración `.ini` | `configparser` (estándar) |
| Configuración `.toml` (lectura) | `tomllib` (estándar) |
| ZIP | `zipfile` (estándar) |
| Base de datos SQLite | `sqlite3` (estándar) |
| Objetos Python (binario) | `pickle` (estándar) ⚠ solo con datos de confianza |
| Excel `.xlsx` | `openpyxl`, `pandas` |
| YAML | `PyYAML` |
| PDF | `pypdf` |

```python
import tomllib
with open("config.toml", "rb") as f:          # tomllib exige modo binario
    config = tomllib.load(f)

import zipfile
with zipfile.ZipFile("copia.zip", "w", zipfile.ZIP_DEFLATED) as z:
    for archivo in Path("informes").glob("*.pdf"):
        z.write(archivo, arcname=archivo.name)
```

> ⚠ **Nunca** cargue con `pickle` archivos de origen desconocido: pueden ejecutar código arbitrario.

---

## 10. Gestores de contexto

`with` garantiza que el recurso se libera (archivo cerrado, conexión terminada, bloqueo liberado) **aunque se produzca una excepción**.

```python
with open("a.txt", encoding="utf-8") as origen, open("b.txt", "w", encoding="utf-8") as destino:
    destino.write(origen.read().upper())
```

### 10.1 Crear un gestor de contexto

```python
from contextlib import contextmanager
import time

@contextmanager
def cronometro(nombre: str):
    inicio = time.perf_counter()
    try:
        yield
    finally:
        print(f"{nombre}: {time.perf_counter() - inicio:.3f} s")

with cronometro("Importación"):
    importar_datos()
```

### 10.2 Utilidades de `contextlib`

```python
from contextlib import suppress

with suppress(FileNotFoundError):             # Ignora un error concreto
    Path("temporal.tmp").unlink()
```

> `Path.unlink(missing_ok=True)` también evita el error si el archivo no existe.

### 10.3 Escritura segura (atómica)

Evita dejar un archivo corrupto si el programa falla a mitad de escritura:

```python
import os, tempfile

def escribir_seguro(ruta: Path, contenido: str) -> None:
    with tempfile.NamedTemporaryFile("w", encoding="utf-8", dir=ruta.parent, delete=False) as tmp:
        tmp.write(contenido)
    os.replace(tmp.name, ruta)                # Sustitución atómica
```

---

## 11. Operaciones con archivos y carpetas

```python
import shutil
from pathlib import Path

Path("salida/2026/11").mkdir(parents=True, exist_ok=True)   # Crea toda la ruta
Path("viejo.txt").rename("nuevo.txt")
Path("borrar.txt").unlink(missing_ok=True)
Path("carpeta_vacia").rmdir()

shutil.copy2("origen.pdf", "copias/")         # Copia conservando fechas
shutil.copytree("proyecto", "proyecto_copia")
shutil.move("descarga.zip", "archivo/")
shutil.rmtree("temporal")                     # ⚠ Borra la carpeta y todo su contenido
shutil.make_archive("copia_2026", "zip", "datos")
shutil.disk_usage("/").free                   # Espacio libre en bytes
```

### 11.1 Archivos temporales

```python
import tempfile

with tempfile.TemporaryDirectory() as carpeta:
    ruta = Path(carpeta) / "intermedio.csv"
    ...                                        # Se borra al salir del bloque
```

---

## 12. Registro de eventos (`logging`)

Sustituya los `print` de diagnóstico por `logging`: permite niveles, fecha, archivo de destino y desactivación sin tocar el código.

### 12.1 Configuración básica

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    handlers=[logging.FileHandler("app.log", encoding="utf-8"), logging.StreamHandler()],
)

logger = logging.getLogger(__name__)          # Un logger por módulo

logger.debug("Detalle para depurar")
logger.info("Importados %d registros", 250)   # Formato con %: se evalúa solo si se emite
logger.warning("Fila %d sin email", 17)
logger.error("No se pudo conectar con %s", host)

try:
    1 / 0
except ZeroDivisionError:
    logger.exception("Error en el cálculo")   # Incluye el traceback completo
```

| Nivel | Uso |
|---|---|
| `DEBUG` | Información detallada para desarrollo. |
| `INFO` | Confirmación de que todo funciona según lo previsto. |
| `WARNING` | Algo inesperado, pero el programa continúa. |
| `ERROR` | Una operación ha fallado. |
| `CRITICAL` | Error grave: el programa puede no continuar. |

### 12.2 Rotación de archivos de registro

```python
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

RotatingFileHandler("app.log", maxBytes=5_000_000, backupCount=5, encoding="utf-8")
TimedRotatingFileHandler("app.log", when="midnight", backupCount=30, encoding="utf-8")
```

> Configure `logging` **una sola vez**, en el punto de entrada (`main`). En los módulos, use solo `logging.getLogger(__name__)`.

---

## 13. Ejemplo práctico completo

**Importador de pedidos**: lee todos los CSV de una carpeta, valida cada fila con excepciones propias, separa registros válidos y rechazados, genera un resumen JSON y un CSV de errores, mueve los archivos procesados y deja trazabilidad en un *log*. Para probarlo, el script crea datos de ejemplo la primera vez.

```python
"""
Importador de pedidos desde CSV con validación, informes y registro.
Uso: python importador.py [carpeta_entrada]
"""
import csv
import json
import logging
import re
import shutil
import sys
from dataclasses import asdict, dataclass
from datetime import date, datetime
from decimal import Decimal, InvalidOperation
from pathlib import Path

BASE = Path(__file__).resolve().parent
ENTRADA = Path(sys.argv[1]) if len(sys.argv) > 1 else BASE / "entrada"
PROCESADOS = BASE / "procesados"
SALIDA = BASE / "salida"
PATRON_EMAIL = re.compile(r"^[\w.+-]+@[\w-]+\.[\w.-]+$")
CAMPOS = ["pedido", "fecha", "email", "producto", "unidades", "precio"]

logger = logging.getLogger("importador")


# ------------------------------------------------------------------ Excepciones
class ErrorImportacion(Exception):
    """Base de los errores del importador."""


class FilaInvalida(ErrorImportacion):
    def __init__(self, archivo: str, linea: int, motivo: str) -> None:
        super().__init__(f"{archivo}:{linea} → {motivo}")
        self.archivo, self.linea, self.motivo = archivo, linea, motivo


class CabeceraInvalida(ErrorImportacion):
    pass


# ------------------------------------------------------------------ Modelo
@dataclass(frozen=True)
class Pedido:
    pedido: str
    fecha: date
    email: str
    producto: str
    unidades: int
    precio: Decimal

    @property
    def importe(self) -> Decimal:
        return self.precio * self.unidades


def validar_fila(fila: dict[str, str], archivo: str, linea: int) -> Pedido:
    def error(motivo: str) -> FilaInvalida:
        return FilaInvalida(archivo, linea, motivo)

    vacios = [c for c in CAMPOS if not (fila.get(c) or "").strip()]
    if vacios:
        raise error(f"campos vacíos: {', '.join(vacios)}")
    try:
        fecha = datetime.strptime(fila["fecha"].strip(), "%d/%m/%Y").date()
    except ValueError:
        raise error(f"fecha no válida '{fila['fecha']}' (formato dd/mm/aaaa)") from None
    if fecha > date.today():
        raise error("la fecha es futura")
    email = fila["email"].strip().lower()
    if not PATRON_EMAIL.match(email):
        raise error(f"email no válido '{email}'")
    try:
        unidades = int(fila["unidades"])
        precio = Decimal(fila["precio"].replace(",", "."))
    except (ValueError, InvalidOperation):
        raise error("unidades o precio no numéricos") from None
    if unidades <= 0 or precio < 0:
        raise error("unidades o precio fuera de rango")
    return Pedido(fila["pedido"].strip(), fecha, email, fila["producto"].strip(), unidades, precio)


# ------------------------------------------------------------------ Proceso
def leer_archivo(ruta: Path) -> tuple[list[Pedido], list[FilaInvalida]]:
    validos, errores = [], []
    with ruta.open(newline="", encoding="utf-8-sig") as f:
        lector = csv.DictReader(f, delimiter=";")
        faltan = set(CAMPOS) - set(lector.fieldnames or [])
        if faltan:
            raise CabeceraInvalida(f"{ruta.name}: faltan columnas {sorted(faltan)}")
        for n, fila in enumerate(lector, start=2):          # La línea 1 es la cabecera
            try:
                validos.append(validar_fila(fila, ruta.name, n))
            except FilaInvalida as e:
                logger.warning(str(e))
                errores.append(e)
    return validos, errores


def guardar_informes(pedidos: list[Pedido], errores: list[FilaInvalida], marca: str) -> None:
    SALIDA.mkdir(exist_ok=True)
    resumen = {
        "fecha_proceso": datetime.now().isoformat(timespec="seconds"),
        "pedidos_validos": len(pedidos),
        "filas_rechazadas": len(errores),
        "importe_total": str(sum((p.importe for p in pedidos), Decimal("0"))),
        "clientes": len({p.email for p in pedidos}),
        "pedidos": [asdict(p) | {"importe": p.importe} for p in pedidos],
    }
    ruta_json = SALIDA / f"resumen_{marca}.json"
    ruta_json.write_text(json.dumps(resumen, ensure_ascii=False, indent=2, default=str), encoding="utf-8")
    logger.info("Resumen guardado en %s", ruta_json.name)

    if errores:
        ruta_err = SALIDA / f"errores_{marca}.csv"
        with ruta_err.open("w", newline="", encoding="utf-8-sig") as f:
            w = csv.writer(f, delimiter=";")
            w.writerow(["archivo", "linea", "motivo"])
            w.writerows((e.archivo, e.linea, e.motivo) for e in errores)
        logger.info("Errores guardados en %s", ruta_err.name)


def crear_datos_ejemplo() -> None:
    ENTRADA.mkdir(exist_ok=True)
    filas = [
        ["P-001", "03/11/2026", "ana@ejemplo.com", "Teclado", "2", "59,90"],
        ["P-002", "05/11/2026", "luis@ejemplo", "Monitor", "1", "249"],
        ["P-003", "31/02/2026", "eva@ejemplo.com", "Ratón", "3", "18,5"],
        ["P-004", "10/11/2026", "eva@ejemplo.com", "Portátil", "1", "899"],
        ["P-005", "12/11/2026", "", "Cable", "4", "6"],
        ["P-006", "15/11/2026", "ana@ejemplo.com", "Webcam", "dos", "45"],
    ]
    with (ENTRADA / "pedidos_noviembre.csv").open("w", newline="", encoding="utf-8-sig") as f:
        w = csv.writer(f, delimiter=";")
        w.writerow(CAMPOS)
        w.writerows(filas)


def main() -> int:
    logging.basicConfig(level=logging.INFO, format="%(asctime)s | %(levelname)-7s | %(message)s",
                        datefmt="%H:%M:%S",
                        handlers=[logging.FileHandler(BASE / "importador.log", encoding="utf-8"),
                                  logging.StreamHandler()])
    if not ENTRADA.exists() or not any(ENTRADA.glob("*.csv")):
        logger.info("No hay archivos de entrada: se crean datos de ejemplo en %s", ENTRADA)
        crear_datos_ejemplo()

    archivos = sorted(ENTRADA.glob("*.csv"))
    logger.info("Archivos a procesar: %d", len(archivos))
    todos, errores = [], []
    PROCESADOS.mkdir(exist_ok=True)

    for ruta in archivos:
        try:
            validos, fallidos = leer_archivo(ruta)
        except CabeceraInvalida as e:
            logger.error("Archivo descartado: %s", e)
            continue
        except (OSError, UnicodeDecodeError):
            logger.exception("No se pudo leer %s", ruta.name)
            continue
        todos += validos
        errores += fallidos
        logger.info("%s: %d válidos, %d rechazados", ruta.name, len(validos), len(fallidos))
        shutil.move(ruta, PROCESADOS / ruta.name)

    guardar_informes(todos, errores, datetime.now().strftime("%Y%m%d_%H%M%S"))
    logger.info("Proceso terminado: %d pedidos importados, %d filas con errores", len(todos), len(errores))
    return 1 if errores else 0


if __name__ == "__main__":
    sys.exit(main())
```

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `FileNotFoundError: [Errno 2] No such file or directory` | Ruta relativa a una carpeta de trabajo distinta de la esperada. | Construya rutas desde `Path(__file__).resolve().parent`. |
| `UnicodeDecodeError: 'utf-8' codec can't decode byte` | Archivo en otra codificación (p. ej. `cp1252`). | Indique la codificación correcta o `errors="replace"`. |
| Tildes mal mostradas en Excel | CSV en UTF-8 sin BOM. | Use `encoding="utf-8-sig"`. |
| Líneas en blanco entre filas de un CSV (Windows) | Falta `newline=""`. | `open(..., newline="")`. |
| Se pierde el contenido del archivo | Se abrió con `"w"` en lugar de `"a"`. | Use `"a"` para añadir. |
| `PermissionError` | Archivo abierto en otro programa (Excel) o sin permisos. | Ciérrelo o compruebe los permisos. |
| `TypeError: Object of type date is not JSON serializable` | Tipos no soportados por JSON. | Use `default=` con una función de conversión. |
| `json.JSONDecodeError` | JSON mal formado (comillas simples, comas finales). | Valide el archivo; JSON exige comillas dobles. |
| `SyntaxError` en una ruta de Windows | `"C:\nuevo"` interpreta `\n`. | Use `Path`, `r"C:\nuevo"` o `/`. |
| Errores ocultos que nadie detecta | `except:` o `except Exception: pass`. | Capture excepciones concretas y regístrelas. |
| No se crea el archivo de *log* | `basicConfig` se llamó después de otro `logging`. | Configure `logging` al principio de `main()`. |

---

## 15. Buenas prácticas

- Use siempre `with` para abrir archivos y otros recursos.
- Indique `encoding="utf-8"` (o `utf-8-sig` para CSV destinados a Excel).
- Use `pathlib` en lugar de concatenar cadenas para las rutas.
- Capture **excepciones concretas**; nunca use `except:` vacío ni silencie errores con `pass`.
- Mantenga el bloque `try` lo más corto posible: solo las instrucciones que pueden fallar.
- Cree una jerarquía de excepciones propias para los errores del dominio.
- Use `raise ... from e` para conservar la causa original.
- Valide los datos externos (archivos, formularios, API) en cuanto entren al programa.
- Lea los archivos grandes línea a línea o con generadores.
- Use `logging` en lugar de `print` para diagnóstico y auditoría; `logger.exception` dentro de `except`.
- Escriba de forma atómica los archivos críticos (configuración, datos).

---

## 16. Referencia rápida

```python
try: ... except (A, B) as e: ... else: ... finally: ...
raise ValueError("msg") / raise NuevaExc("msg") from e / raise
class MiError(Exception): ...

p = Path(__file__).resolve().parent / "datos" / "a.csv"
p.name .stem .suffix .parent .exists() .is_file() .mkdir(parents=True, exist_ok=True) .glob("*.csv") .rglob()
p.read_text(encoding="utf-8") / p.write_text(txt, encoding="utf-8") / p.unlink(missing_ok=True)
with open(p, "r|w|a|rb", encoding="utf-8", newline="") as f: f.read() / for l in f / f.write()
csv.DictReader(f, delimiter=";") / csv.DictWriter(f, fieldnames=[...]).writeheader() .writerows()
json.load(f) / json.dump(obj, f, ensure_ascii=False, indent=2, default=str) / json.loads / json.dumps
shutil.copy2 / move / rmtree / make_archive   ·   tempfile.TemporaryDirectory()
logging.basicConfig(level=logging.INFO, format=..., handlers=[...]); log = logging.getLogger(__name__)
log.info("x=%s", x) / log.warning / log.error / log.exception
```

---

## 17. Ejercicios propuestos

1. Escriba un programa que cuente las líneas, palabras y caracteres de todos los `.txt` de una carpeta y muestre un resumen ordenado.
2. Cree una función que convierta un CSV a JSON y otra que haga la conversión inversa, gestionando codificación y separador.
3. Desarrolle un organizador de descargas que mueva los archivos a subcarpetas por extensión (`pdf/`, `imagenes/`, `comprimidos/`…) y registre cada movimiento.
4. Implemente un gestor de configuración que lea un `config.json`, aplique valores por defecto y lance una excepción propia si faltan claves obligatorias.
5. Amplíe el ejemplo completo para detectar pedidos duplicados (mismo número de pedido en distintos archivos) y añadirlos al informe de errores.

---

## 18. Referencias

- Errores y excepciones (tutorial): https://docs.python.org/es/3/tutorial/errors.html
- Excepciones integradas: https://docs.python.org/es/3/library/exceptions.html
- `pathlib`: https://docs.python.org/es/3/library/pathlib.html
- `csv`: https://docs.python.org/es/3/library/csv.html
- `json`: https://docs.python.org/es/3/library/json.html
- `logging` (guía práctica): https://docs.python.org/es/3/howto/logging.html
