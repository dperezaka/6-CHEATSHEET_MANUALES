# Manual de uso: Watchdog

> **Categoría:** Automatización y sistema · **Utilidad:** Detectar cambios en archivos y carpetas
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tipos de eventos](#4-tipos-de-eventos)
5. [Operaciones básicas](#5-operaciones-básicas)
6. [Filtrar eventos](#6-filtrar-eventos)
7. [Vigilar varias carpetas](#7-vigilar-varias-carpetas)
8. [Problemas habituales y técnicas](#8-problemas-habituales-y-técnicas)
9. [Procesamiento en segundo plano](#9-procesamiento-en-segundo-plano)
10. [Observadores alternativos](#10-observadores-alternativos)
11. [Herramienta de línea de comandos](#11-herramienta-de-línea-de-comandos)
12. [Ejemplo práctico completo](#12-ejemplo-práctico-completo)
13. [Errores frecuentes](#13-errores-frecuentes)
14. [Buenas prácticas](#14-buenas-prácticas)
15. [Referencia rápida](#15-referencia-rápida)
16. [Ejercicios propuestos](#16-ejercicios-propuestos)
17. [Referencias](#17-referencias)

---

## 1. Descripción

**Watchdog** vigila carpetas del sistema de archivos y avisa en tiempo real cuando se **crean, modifican, mueven o eliminan** archivos y carpetas. Usa los mecanismos nativos de cada sistema operativo, por lo que es eficiente y no necesita comprobar la carpeta continuamente.

**Casos de uso habituales:**
- Procesar automáticamente archivos que llegan a una carpeta (facturas, pedidos, exportaciones).
- Carpetas de «entrada» para conversiones automáticas (PDF, imágenes, CSV → Excel).
- Sincronización o copias de seguridad al detectar cambios.
- Recargar una aplicación o regenerar un sitio cuando cambia el código.
- Auditoría de cambios en carpetas compartidas.

| Sistema | Mecanismo nativo |
|---|---|
| Windows | `ReadDirectoryChangesW` |
| Linux | `inotify` |
| macOS | `FSEvents` |
| Otros / redes | Sondeo periódico (`PollingObserver`) |

---

## 2. Instalación e importación

```bash
pip install watchdog
```

Con la herramienta de línea de comandos `watchmedo`:

```bash
pip install "watchdog[watchmedo]"
```

```python
from watchdog.observers import Observer
from watchdog.events import (
    FileSystemEventHandler, PatternMatchingEventHandler, RegexMatchingEventHandler,
    FileSystemEvent,
)
```

---

## 3. Conceptos clave

| Componente | Función |
|---|---|
| **Observer** | Vigila una o varias rutas en un hilo propio y genera eventos. |
| **Event handler** | Clase con métodos que reaccionan a cada tipo de evento. |
| **Event** | Objeto con la información del cambio (`event_type`, `src_path`, `dest_path`, `is_directory`). |
| `schedule()` | Asocia un manejador a una ruta en el observador. |
| `recursive` | Si se vigilan también las subcarpetas. |

**Flujo de trabajo:**

```
Crear manejador → Crear Observer → observer.schedule(manejador, ruta) → observer.start()
→ (el programa sigue en ejecución) → observer.stop() → observer.join()
```

---

## 4. Tipos de eventos

| Método del manejador | Se ejecuta cuando… | Clase del evento |
|---|---|---|
| `on_created` | Se crea un archivo o carpeta. | `FileCreatedEvent` / `DirCreatedEvent` |
| `on_modified` | Se modifica el contenido o metadatos. | `FileModifiedEvent` / `DirModifiedEvent` |
| `on_deleted` | Se elimina. | `FileDeletedEvent` / `DirDeletedEvent` |
| `on_moved` | Se mueve o renombra (incluye `dest_path`). | `FileMovedEvent` / `DirMovedEvent` |
| `on_closed` | Se cierra un archivo tras escribir en él (Linux). | `FileClosedEvent` |
| `on_closed_no_write` | Se cierra un archivo sin escribir (Linux). | `FileClosedNoWriteEvent` |
| `on_opened` | Se abre un archivo (Linux). | `FileOpenedEvent` |
| `on_any_event` | Cualquiera de los anteriores. | — |

Atributos del evento:

```python
event.event_type      # 'created', 'modified', 'deleted', 'moved', 'closed', 'opened'
event.src_path        # Ruta de origen (str)
event.dest_path       # Ruta de destino (solo en 'moved')
event.is_directory    # True si es una carpeta
event.is_synthetic    # True si lo generó watchdog (no el sistema)
```

---

## 5. Operaciones básicas

### 5.1 Primer vigilante

```python
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler


class Manejador(FileSystemEventHandler):
    def on_created(self, event):
        print(f"Creado:     {event.src_path}")

    def on_modified(self, event):
        print(f"Modificado: {event.src_path}")

    def on_deleted(self, event):
        print(f"Eliminado:  {event.src_path}")

    def on_moved(self, event):
        print(f"Movido:     {event.src_path} → {event.dest_path}")


if __name__ == "__main__":
    ruta = "."
    observador = Observer()
    observador.schedule(Manejador(), ruta, recursive=True)
    observador.start()
    print(f"Vigilando {ruta!r}. Pulse Ctrl+C para detener.")
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observador.stop()
    observador.join()
```

### 5.2 Con gestor de contexto

```python
observador = Observer()
observador.schedule(Manejador(), "entrada", recursive=False)
with observador:                     # start() al entrar; stop() al salir
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        pass
observador.join()
```

### 5.3 Ignorar eventos de carpetas

```python
class SoloArchivos(FileSystemEventHandler):
    def on_any_event(self, event):
        if event.is_directory:
            return
        print(event.event_type, event.src_path)
```

### 5.4 Trabajar con `pathlib`

```python
from pathlib import Path

class Manejador(FileSystemEventHandler):
    def on_created(self, event):
        ruta = Path(event.src_path)
        print(ruta.name, ruta.suffix, ruta.parent)
```

> En versiones recientes `src_path` puede ser `str` o `bytes`; convierta con `os.fsdecode()` si necesita asegurar texto.

---

## 6. Filtrar eventos

### 6.1 `PatternMatchingEventHandler` (comodines)

```python
from watchdog.events import PatternMatchingEventHandler


class ManejadorExcel(PatternMatchingEventHandler):
    def __init__(self):
        super().__init__(
            patterns=["*.xlsx", "*.xls", "*.csv"],      # Incluir
            ignore_patterns=["~$*", "*.tmp"],           # Excluir (temporales de Office)
            ignore_directories=True,
            case_sensitive=False,
        )

    def on_created(self, event):
        print("Nueva hoja de cálculo:", event.src_path)
```

### 6.2 `RegexMatchingEventHandler` (expresiones regulares)

```python
from watchdog.events import RegexMatchingEventHandler


class ManejadorFacturas(RegexMatchingEventHandler):
    def __init__(self):
        super().__init__(
            regexes=[r".*[/\\]FAC-\d{4}-\d{5}\.pdf$"],   # FAC-2026-00123.pdf
            ignore_regexes=[r".*[/\\]\..*"],              # Ocultos
            ignore_directories=True,
            case_sensitive=False,
        )

    def on_created(self, event):
        print("Factura recibida:", event.src_path)
```

### 6.3 Filtrar tipos de evento en el observador (watchdog ≥ 5)

```python
from watchdog.events import FileCreatedEvent, FileMovedEvent

observador.schedule(manejador, "entrada", event_filter=[FileCreatedEvent, FileMovedEvent])
```

### 6.4 Filtrar manualmente

```python
EXTENSIONES = {".pdf", ".docx"}
IGNORAR = {".git", "__pycache__", "node_modules"}

class Manejador(FileSystemEventHandler):
    def on_any_event(self, event):
        ruta = Path(event.src_path)
        if event.is_directory or ruta.suffix.lower() not in EXTENSIONES:
            return
        if any(parte in IGNORAR for parte in ruta.parts):
            return
        ...
```

---

## 7. Vigilar varias carpetas

```python
observador = Observer()
observador.schedule(ManejadorExcel(), "C:/Datos/Entrada", recursive=False)
observador.schedule(ManejadorFacturas(), "C:/Contabilidad/Recibidas", recursive=True)
observador.schedule(Manejador(), "D:/Compartida", recursive=True)
observador.start()
```

Gestionar las vigilancias:

```python
vigilancia = observador.schedule(Manejador(), "temporal")
observador.unschedule(vigilancia)       # Dejar de vigilar una ruta
observador.unschedule_all()
```

---

## 8. Problemas habituales y técnicas

### 8.1 Eventos duplicados

Guardar un archivo suele generar **varios** eventos (`created` + uno o más `modified`). Muchos editores además escriben en un temporal y lo renombran. Solución: **antirrebote** (*debounce*), es decir, procesar solo cuando el archivo lleve un tiempo sin cambios.

```python
import threading


class ManejadorConAntirrebote(FileSystemEventHandler):
    def __init__(self, procesar, espera: float = 2.0):
        self.procesar = procesar
        self.espera = espera
        self._temporizadores: dict[str, threading.Timer] = {}
        self._cerrojo = threading.Lock()

    def _programar(self, ruta: str):
        with self._cerrojo:
            if ruta in self._temporizadores:
                self._temporizadores[ruta].cancel()
            t = threading.Timer(self.espera, self._ejecutar, args=[ruta])
            self._temporizadores[ruta] = t
            t.start()

    def _ejecutar(self, ruta: str):
        with self._cerrojo:
            self._temporizadores.pop(ruta, None)
        self.procesar(ruta)

    def on_created(self, event):
        if not event.is_directory:
            self._programar(event.src_path)

    def on_modified(self, event):
        if not event.is_directory:
            self._programar(event.src_path)

    def on_moved(self, event):
        if not event.is_directory:
            self._programar(event.dest_path)
```

### 8.2 Archivos que aún se están escribiendo

Un evento `created` llega en cuanto aparece el archivo, **antes** de que termine de copiarse (archivos grandes, descargas, copias por red). Espere a que el tamaño se estabilice:

```python
import os
import time


def esperar_archivo_estable(ruta: str, intervalo: float = 1.0, comprobaciones: int = 3,
                            timeout: float = 300) -> bool:
    limite = time.monotonic() + timeout
    anterior, estables = -1, 0
    while time.monotonic() < limite:
        try:
            tamano = os.path.getsize(ruta)
        except FileNotFoundError:
            return False
        if tamano == anterior and tamano > 0:
            estables += 1
            if estables >= comprobaciones:
                return True
        else:
            estables = 0
        anterior = tamano
        time.sleep(intervalo)
    return False
```

En Windows, puede comprobarse además que el archivo no esté bloqueado:

```python
def archivo_disponible(ruta: str) -> bool:
    try:
        with open(ruta, "rb+"):
            return True
    except (PermissionError, OSError):
        return False
```

### 8.3 Temporales de aplicaciones

| Aplicación | Archivos temporales |
|---|---|
| Microsoft Office | `~$nombre.xlsx`, `~WRL0001.tmp` |
| LibreOffice | `.~lock.nombre.ods#` |
| Navegadores | `.crdownload`, `.part`, `.download` |
| Editores | `.swp`, `~`, `.tmp`, `4913` |

Ignórelos con `ignore_patterns`.

### 8.4 Bucles infinitos

Si el manejador escribe archivos en la **misma carpeta que vigila**, generará nuevos eventos que volverán a activar el manejador. Soluciones:
- Escriba los resultados en otra carpeta.
- Mueva los archivos procesados a una subcarpeta excluida.
- Ignore las extensiones o nombres que genera el propio script.

### 8.5 Archivos existentes al arrancar

Watchdog solo informa de los cambios **posteriores** al inicio. Procese al arrancar los archivos que ya estaban:

```python
for ruta in Path("entrada").glob("*.csv"):
    procesar(str(ruta))
observador.start()
```

---

## 9. Procesamiento en segundo plano

Los métodos del manejador se ejecutan en el **hilo del observador**. Si el procesamiento es lento, los eventos se acumulan. Use una cola y un hilo trabajador:

```python
import queue
import threading

cola: "queue.Queue[str]" = queue.Queue()


class Encolador(PatternMatchingEventHandler):
    def __init__(self):
        super().__init__(patterns=["*.pdf"], ignore_directories=True)

    def on_created(self, event):
        cola.put(event.src_path)

    def on_moved(self, event):
        cola.put(event.dest_path)


def trabajador():
    while True:
        ruta = cola.get()
        try:
            if esperar_archivo_estable(ruta):
                procesar_pdf(ruta)
        except Exception as e:
            print(f"Error procesando {ruta}: {e}")
        finally:
            cola.task_done()


for _ in range(2):                      # Dos trabajadores en paralelo
    threading.Thread(target=trabajador, daemon=True).start()
```

---

## 10. Observadores alternativos

```python
from watchdog.observers.polling import PollingObserver

observador = PollingObserver(timeout=5)       # Comprueba cada 5 segundos
```

Use `PollingObserver` cuando:
- Vigila **unidades de red** (SMB/CIFS, NFS), carpetas de Docker montadas o sistemas virtualizados, donde los eventos nativos no siempre llegan.
- El mecanismo nativo no está disponible.

> El sondeo consume más recursos en carpetas con muchos archivos. Ajuste el `timeout` según la necesidad.

En Linux, `inotify` tiene un límite de carpetas vigiladas. Si se supera:

```bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 11. Herramienta de línea de comandos

`watchmedo` permite vigilar y reaccionar sin escribir código:

```bash
# Mostrar los cambios en la consola
watchmedo log --patterns="*.py;*.txt" --recursive .

# Ejecutar un comando al detectar cambios
watchmedo shell-command --patterns="*.py" --recursive \
    --command='echo "Cambio en ${watch_src_path}"' .

# Ejecutar las pruebas cada vez que cambia el código
watchmedo shell-command --patterns="*.py" --recursive --drop \
    --command="pytest -q" src/

# Reiniciar un programa cuando cambia el código
watchmedo auto-restart --patterns="*.py" --recursive -- python servidor.py
```

---

## 12. Ejemplo práctico completo

**Carpeta de entrada automática**: los archivos que se dejan en `entrada/` se procesan según su tipo (CSV → Excel resumen, imágenes → miniaturas, PDF → archivado por fecha), con antirrebote, espera a que la copia termine, cola de procesamiento, gestión de errores y registro.

```python
"""
Procesador automático de carpeta de entrada.
Requisitos: pip install watchdog pandas openpyxl pillow
"""
import logging
import queue
import shutil
import threading
import time
from datetime import datetime
from pathlib import Path

from watchdog.events import PatternMatchingEventHandler
from watchdog.observers import Observer

RAIZ = Path(__file__).resolve().parent
ENTRADA = RAIZ / "entrada"
PROCESADOS = RAIZ / "procesados"
ERRORES = RAIZ / "errores"
SALIDA = RAIZ / "salida"
for carpeta in (ENTRADA, PROCESADOS, ERRORES, SALIDA):
    carpeta.mkdir(exist_ok=True)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-7s | %(threadName)-10s | %(message)s",
    handlers=[logging.FileHandler(RAIZ / "procesador.log", encoding="utf-8"), logging.StreamHandler()],
)
log = logging.getLogger("procesador")


# ------------------------------------------------------------------ Utilidades
def esperar_estable(ruta: Path, intervalo: float = 0.5, comprobaciones: int = 3, timeout: float = 120) -> bool:
    limite = time.monotonic() + timeout
    anterior, estables = -1, 0
    while time.monotonic() < limite:
        try:
            tamano = ruta.stat().st_size
        except FileNotFoundError:
            return False
        if tamano == anterior and tamano > 0:
            estables += 1
            if estables >= comprobaciones:
                try:
                    with ruta.open("rb"):
                        return True
                except PermissionError:
                    estables = 0
        else:
            estables = 0
        anterior = tamano
        time.sleep(intervalo)
    return False


def mover_unico(origen: Path, carpeta: Path) -> Path:
    carpeta.mkdir(parents=True, exist_ok=True)
    destino = carpeta / origen.name
    n = 1
    while destino.exists():
        destino = carpeta / f"{origen.stem}_{n}{origen.suffix}"
        n += 1
    return Path(shutil.move(origen, destino))


# ------------------------------------------------------------------ Procesadores
def procesar_csv(ruta: Path) -> str:
    import pandas as pd
    df = pd.read_csv(ruta, sep=None, engine="python")
    numericas = df.select_dtypes("number")
    destino = SALIDA / f"{ruta.stem}_resumen.xlsx"
    with pd.ExcelWriter(destino) as excel:
        df.to_excel(excel, sheet_name="Datos", index=False)
        if not numericas.empty:
            numericas.describe().round(2).to_excel(excel, sheet_name="Resumen")
    return f"{len(df)} filas → {destino.name}"


def procesar_imagen(ruta: Path) -> str:
    from PIL import Image, ImageOps
    with Image.open(ruta) as img:
        img = ImageOps.exif_transpose(img).convert("RGB")
        img.thumbnail((800, 800))
        destino = SALIDA / f"{ruta.stem}_800.jpg"
        img.save(destino, quality=85, optimize=True)
    return f"miniatura → {destino.name}"


def procesar_pdf(ruta: Path) -> str:
    fecha = datetime.fromtimestamp(ruta.stat().st_mtime)
    carpeta = SALIDA / "pdf" / f"{fecha:%Y}" / f"{fecha:%m}"
    carpeta.mkdir(parents=True, exist_ok=True)
    shutil.copy2(ruta, carpeta / ruta.name)
    return f"archivado en pdf/{fecha:%Y/%m}"


PROCESADORES = {
    ".csv": procesar_csv,
    ".jpg": procesar_imagen, ".jpeg": procesar_imagen, ".png": procesar_imagen, ".webp": procesar_imagen,
    ".pdf": procesar_pdf,
}


# ------------------------------------------------------------------ Cola y trabajador
cola: "queue.Queue[Path]" = queue.Queue()
pendientes: set[Path] = set()
cerrojo = threading.Lock()


def encolar(ruta: Path) -> None:
    with cerrojo:
        if ruta in pendientes:
            return                         # Antirrebote: ya está en cola
        pendientes.add(ruta)
    cola.put(ruta)


def trabajador() -> None:
    while True:
        ruta = cola.get()
        try:
            if not esperar_estable(ruta):
                log.warning("Archivo no disponible o incompleto: %s", ruta.name)
                continue
            procesador = PROCESADORES.get(ruta.suffix.lower())
            if procesador is None:
                log.info("Tipo no soportado, se mueve a errores: %s", ruta.name)
                mover_unico(ruta, ERRORES)
                continue
            inicio = time.perf_counter()
            resultado = procesador(ruta)
            mover_unico(ruta, PROCESADOS / f"{datetime.now():%Y-%m-%d}")
            log.info("✔ %s · %s (%.1f s)", ruta.name, resultado, time.perf_counter() - inicio)
        except Exception:
            log.exception("✖ Error procesando %s", ruta.name)
            if ruta.exists():
                mover_unico(ruta, ERRORES)
        finally:
            with cerrojo:
                pendientes.discard(ruta)
            cola.task_done()


# ------------------------------------------------------------------ Manejador
class ManejadorEntrada(PatternMatchingEventHandler):
    def __init__(self):
        super().__init__(
            patterns=["*"],
            ignore_patterns=["~$*", ".~lock*", "*.tmp", "*.part", "*.crdownload", ".*"],
            ignore_directories=True,
            case_sensitive=False,
        )

    def on_created(self, event):
        encolar(Path(event.src_path))

    def on_moved(self, event):                 # Archivos renombrados dentro de la carpeta
        destino = Path(event.dest_path)
        if destino.parent == ENTRADA:
            encolar(destino)


# ------------------------------------------------------------------ Programa principal
def main() -> None:
    for _ in range(2):
        threading.Thread(target=trabajador, name="trabajador", daemon=True).start()

    existentes = [p for p in ENTRADA.iterdir() if p.is_file() and not p.name.startswith((".", "~$"))]
    for p in existentes:
        encolar(p)
    if existentes:
        log.info("%d archivos existentes en cola", len(existentes))

    observador = Observer()
    observador.schedule(ManejadorEntrada(), str(ENTRADA), recursive=False)
    observador.start()
    log.info("Vigilando %s · tipos: %s · Ctrl+C para detener", ENTRADA, ", ".join(sorted(PROCESADORES)))

    try:
        while observador.is_alive():
            observador.join(timeout=1)
    except KeyboardInterrupt:
        log.info("Deteniendo…")
    finally:
        observador.stop()
        observador.join()
        cola.join()                            # Terminar lo que quede en cola
        log.info("Procesador detenido")


if __name__ == "__main__":
    main()
```

---

## 13. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| El programa termina inmediatamente | El hilo principal finaliza tras `start()`. | Mantenga un bucle (`while True: time.sleep(1)`) o `observer.join()`. |
| Se procesa el mismo archivo varias veces | Varios eventos por cada guardado. | Use antirrebote o un conjunto de pendientes. |
| `PermissionError` al abrir el archivo | Aún se está copiando o lo tiene abierto otra aplicación. | Espere a que el tamaño sea estable y el archivo esté libre. |
| Archivo vacío o incompleto | Se procesó en cuanto se creó. | Use `esperar_estable` antes de procesar. |
| Bucle infinito de eventos | El script escribe en la carpeta vigilada. | Escriba en otra carpeta o ignore esos archivos. |
| No detecta cambios en una unidad de red | Los eventos nativos no se propagan. | Use `PollingObserver`. |
| `OSError: inotify watch limit reached` (Linux) | Demasiadas carpetas vigiladas. | Aumente `fs.inotify.max_user_watches`. |
| No detecta archivos que ya estaban | Watchdog solo informa de cambios posteriores. | Procese los existentes al arrancar. |
| El manejador bloquea otros eventos | Procesamiento lento en el hilo del observador. | Use una cola y trabajadores. |
| Se procesan temporales de Office | No se excluyen `~$*`. | Añádalos a `ignore_patterns`. |

---

## 14. Buenas prácticas

- Mantenga los manejadores **ligeros**: encole el trabajo y procéselo en otro hilo.
- Aplique antirrebote y espere a que los archivos estén completos antes de procesarlos.
- Excluya temporales y archivos ocultos.
- No escriba en la carpeta vigilada; mueva los archivos procesados a otra ubicación.
- Separe los resultados en carpetas de **procesados** y **errores**.
- Procese los archivos existentes al arrancar.
- Use `PollingObserver` para unidades de red.
- Registre cada evento y cada error con `logging`.
- Ejecute el vigilante como servicio para que esté siempre activo.

---

## 15. Referencia rápida

```python
from watchdog.observers import Observer
from watchdog.observers.polling import PollingObserver
from watchdog.events import FileSystemEventHandler, PatternMatchingEventHandler, RegexMatchingEventHandler

class M(FileSystemEventHandler):
    def on_created(self, e): ...     # e.src_path, e.is_directory
    def on_modified(self, e): ...
    def on_deleted(self, e): ...
    def on_moved(self, e): ...       # e.dest_path
    def on_closed(self, e): ...      # Linux
    def on_any_event(self, e): ...   # e.event_type

PatternMatchingEventHandler(patterns=["*.csv"], ignore_patterns=["~$*"], ignore_directories=True, case_sensitive=False)

obs = Observer()
w = obs.schedule(M(), "ruta", recursive=True)
obs.start(); ...; obs.stop(); obs.join()
obs.unschedule(w)
```

```bash
watchmedo log --patterns="*.py" --recursive .
watchmedo shell-command --patterns="*.py" --recursive --command="pytest -q" .
watchmedo auto-restart --patterns="*.py" --recursive -- python app.py
```

---

## 16. Ejercicios propuestos

1. Vigile la carpeta de descargas y mueva automáticamente los PDF a *Documentos/PDF*.
2. Cree un registro de auditoría (CSV) con todos los cambios de una carpeta compartida: fecha, tipo de evento, ruta y tamaño.
3. Convierta automáticamente a PDF los `.docx` que se dejen en una carpeta, usando LibreOffice con `subprocess`.
4. Implemente un sincronizador sencillo que replique en otra carpeta los archivos creados, modificados y eliminados.
5. Use `watchmedo` para ejecutar las pruebas de un proyecto cada vez que cambie un archivo `.py`.

---

## 17. Referencias

- Documentación oficial: https://python-watchdog.readthedocs.io/
- Repositorio: https://github.com/gorakhargosh/watchdog
