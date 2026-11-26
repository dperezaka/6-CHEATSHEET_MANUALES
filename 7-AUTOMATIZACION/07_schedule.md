# Manual de uso: Schedule

> **Categoría:** Automatización y sistema · **Utilidad:** Programar tareas periódicas
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Sintaxis de programación](#5-sintaxis-de-programación)
6. [Pasar argumentos a las tareas](#6-pasar-argumentos-a-las-tareas)
7. [Gestión de tareas](#7-gestión-de-tareas)
8. [Zonas horarias](#8-zonas-horarias)
9. [Gestión de errores](#9-gestión-de-errores)
10. [Tareas largas y concurrencia](#10-tareas-largas-y-concurrencia)
11. [Ejecución permanente del programador](#11-ejecución-permanente-del-programador)
12. [Schedule frente a otras alternativas](#12-schedule-frente-a-otras-alternativas)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**Schedule** permite programar la ejecución periódica de funciones de Python con una sintaxis muy legible, cercana al lenguaje natural:

```python
schedule.every().monday.at("08:30").do(enviar_informe)
```

**Casos de uso habituales:**
- Copias de seguridad diarias.
- Envío de informes o recordatorios a una hora fija.
- Comprobación periódica de servidores, precios o servicios.
- Descarga o sincronización de datos cada cierto tiempo.
- Limpieza de archivos temporales.

**Características:**
- Ligera, sin dependencias y fácil de aprender.
- Se ejecuta **dentro del proceso de Python**: el script debe permanecer en ejecución.
- No guarda las tareas en disco: si el programa se detiene, no se ejecutan.

---

## 2. Instalación e importación

```bash
pip install schedule
pip install pytz          # Opcional: necesario para indicar zonas horarias en .at()
```

```python
import schedule
import time
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Job` | Tarea programada: qué función ejecutar y cuándo. |
| `Scheduler` | Programador que almacena las tareas (por defecto se usa uno global). |
| `every()` | Inicia la definición de una tarea periódica. |
| `.do(funcion, *args)` | Asocia la función a la tarea. |
| `run_pending()` | Ejecuta las tareas cuyo momento ha llegado. **Debe llamarse en un bucle.** |
| `idle_seconds()` | Segundos hasta la siguiente tarea. |
| Etiquetas (*tags*) | Permiten agrupar, consultar y cancelar tareas. |

**Funcionamiento:** Schedule no crea hilos ni procesos. El bucle principal llama a `run_pending()` periódicamente; esta función comprueba qué tareas toca ejecutar y las ejecuta **una tras otra**, en el mismo hilo.

---

## 4. Operaciones básicas

### 4.1 Primer programa

```python
import time
import schedule


def tarea():
    print("Ejecutando tarea...")


schedule.every(10).seconds.do(tarea)

while True:
    schedule.run_pending()
    time.sleep(1)
```

### 4.2 Ejemplos de programación

```python
schedule.every(10).minutes.do(tarea)
schedule.every().hour.do(tarea)
schedule.every().day.at("09:00").do(tarea)
schedule.every().monday.do(tarea)
schedule.every().wednesday.at("13:15").do(tarea)
schedule.every().minute.at(":17").do(tarea)          # En el segundo 17 de cada minuto
schedule.every().hour.at(":30").do(tarea)            # A y media de cada hora
schedule.every(5).to(10).minutes.do(tarea)           # Intervalo aleatorio entre 5 y 10 minutos
schedule.every().day.at("08:30").until("2026-12-31 23:59").do(tarea)
```

### 4.3 Ejecutar una vez al iniciar y luego periódicamente

`every()` programa la **primera** ejecución tras el primer intervalo. Para ejecutar también al arrancar:

```python
schedule.every(1).hours.do(tarea)
schedule.run_all()                 # Ejecuta todas las tareas ahora
```

O solo una concreta:

```python
trabajo = schedule.every(1).hours.do(tarea)
trabajo.run()
```

---

## 5. Sintaxis de programación

### 5.1 Unidades de tiempo

| Singular (con `every()`) | Plural (con `every(n)`) |
|---|---|
| `.second` | `.seconds` |
| `.minute` | `.minutes` |
| `.hour` | `.hours` |
| `.day` | `.days` |
| `.week` | `.weeks` |

```python
schedule.every().minute.do(tarea)      # Cada minuto
schedule.every(15).minutes.do(tarea)   # Cada 15 minutos
```

> Usar singular con `every(n)` (por ejemplo `every(3).minute`) produce un error.

### 5.2 Días de la semana

`.monday`, `.tuesday`, `.wednesday`, `.thursday`, `.friday`, `.saturday`, `.sunday`

```python
schedule.every().friday.at("17:00").do(cerrar_semana)
```

> No se pueden combinar varios días en una sola tarea: cree una tarea por día (ver 5.5).

### 5.3 Hora concreta con `.at()`

| Unidad | Formato de `.at()` | Ejemplo |
|---|---|---|
| `day` / día de la semana | `"HH:MM"` o `"HH:MM:SS"` | `.day.at("08:30")` |
| `hour` | `":MM"` o `"MM:SS"` | `.hour.at(":15")` |
| `minute` | `":SS"` | `.minute.at(":30")` |

### 5.4 Fecha o tiempo límite con `.until()`

```python
from datetime import datetime, timedelta, time as hora

schedule.every(1).hours.until("18:30").do(tarea)                       # Hasta hoy a las 18:30
schedule.every(1).hours.until("2026-12-31 18:30").do(tarea)            # Hasta una fecha
schedule.every(1).hours.until(timedelta(hours=8)).do(tarea)            # Durante 8 horas
schedule.every(1).hours.until(hora(18, 30)).do(tarea)
schedule.every(1).hours.until(datetime(2026, 12, 31, 18, 30)).do(tarea)
```

### 5.5 Días laborables

```python
for dia in ("monday", "tuesday", "wednesday", "thursday", "friday"):
    getattr(schedule.every(), dia).at("08:00").do(informe_diario).tag("laborables")
```

O comprobando dentro de la función:

```python
from datetime import date

def informe_si_laborable():
    if date.today().weekday() < 5:          # 0 = lunes … 4 = viernes
        informe_diario()

schedule.every().day.at("08:00").do(informe_si_laborable)
```

### 5.6 Días concretos del mes

Schedule no admite «el día 1 de cada mes» directamente. Compruébelo dentro de la tarea:

```python
def cierre_mensual():
    if date.today().day == 1:
        generar_cierre()

schedule.every().day.at("06:00").do(cierre_mensual)
```

Último día del mes:

```python
from datetime import timedelta

def si_ultimo_dia_mes():
    if (date.today() + timedelta(days=1)).day == 1:
        generar_cierre()
```

---

## 6. Pasar argumentos a las tareas

### 6.1 Argumentos posicionales y con nombre

```python
def saludar(nombre, saludo="Hola"):
    print(f"{saludo}, {nombre}")

schedule.every(2).seconds.do(saludar, "Ana")
schedule.every(5).seconds.do(saludar, "Luis", saludo="Buenos días")
```

### 6.2 Con `functools.partial` o `lambda`

```python
from functools import partial

schedule.every().day.at("07:00").do(partial(copia_seguridad, origen="C:/Datos", destino="D:/Backup"))
schedule.every(10).minutes.do(lambda: comprobar("https://miempresa.com"))
```

### 6.3 Decorador `@repeat`

```python
from schedule import every, repeat


@repeat(every(10).minutes)
def comprobar_servidor():
    ...


@repeat(every().day.at("08:00"), "equipo@empresa.com")
def enviar_resumen(destinatario):
    ...
```

Las tareas decoradas se registran al importar el módulo; solo falta el bucle con `run_pending()`.

---

## 7. Gestión de tareas

### 7.1 Consultar tareas

```python
schedule.get_jobs()                 # Lista de tareas
schedule.next_run()                 # datetime de la próxima ejecución
schedule.idle_seconds()             # Segundos hasta la próxima ejecución

trabajo = schedule.every().day.at("09:00").do(tarea)
trabajo.next_run                    # Próxima ejecución de esta tarea
trabajo.last_run                    # Última ejecución
print(trabajo)                      # Descripción legible
```

### 7.2 Etiquetas

```python
schedule.every().day.at("08:00").do(informe_ventas).tag("informes", "diario")
schedule.every().monday.do(informe_semanal).tag("informes", "semanal")
schedule.every(5).minutes.do(monitor).tag("monitorizacion")

schedule.get_jobs("informes")       # Tareas con la etiqueta "informes"
schedule.clear("informes")          # Cancelar todas las tareas con esa etiqueta
```

### 7.3 Cancelar tareas

```python
trabajo = schedule.every(10).seconds.do(tarea)
schedule.cancel_job(trabajo)        # Cancelar una tarea

schedule.clear()                    # Cancelar todas
```

### 7.4 Tareas que se ejecutan una sola vez

Si la función devuelve `schedule.CancelJob`, la tarea se elimina tras ejecutarse:

```python
def recordatorio_unico():
    print("Recordatorio: reunión a las 10:00")
    return schedule.CancelJob

schedule.every().day.at("09:50").do(recordatorio_unico)
```

### 7.5 Limitar el número de ejecuciones

```python
def con_limite(funcion, veces: int):
    contador = {"n": 0}
    def envoltorio(*args, **kwargs):
        funcion(*args, **kwargs)
        contador["n"] += 1
        if contador["n"] >= veces:
            return schedule.CancelJob
    return envoltorio

schedule.every(1).minutes.do(con_limite(tarea, veces=5))
```

### 7.6 Varios programadores independientes

```python
programador_a = schedule.Scheduler()
programador_b = schedule.Scheduler()

programador_a.every(5).seconds.do(tarea_a)
programador_b.every(1).minutes.do(tarea_b)

while True:
    programador_a.run_pending()
    programador_b.run_pending()
    time.sleep(1)
```

---

## 8. Zonas horarias

Por defecto, `.at()` usa la **hora local del equipo**. Se puede indicar otra zona horaria:

```python
schedule.every().day.at("09:00", "Europe/Madrid").do(tarea)
schedule.every().day.at("09:00", "America/New_York").do(informe_eeuu)
```

> El uso de zonas horarias en `.at()` requiere la librería **pytz**: `pip install pytz`. Sin ella se produce `ModuleNotFoundError: No module named 'pytz'`.

Recomendación para servidores (que suelen estar en UTC): indique siempre la zona horaria de negocio.

---

## 9. Gestión de errores

Si una tarea lanza una excepción, **se detiene todo el bucle** y el programa termina. Proteja las tareas:

### 9.1 Decorador de protección

```python
import functools
import logging

log = logging.getLogger("tareas")


def proteger(funcion):
    @functools.wraps(funcion)
    def envoltorio(*args, **kwargs):
        try:
            return funcion(*args, **kwargs)
        except Exception:
            log.exception("Error en la tarea %s", funcion.__name__)
    return envoltorio


@proteger
def tarea_fragil():
    1 / 0


schedule.every(5).seconds.do(tarea_fragil)        # El error se registra y el programa continúa
```

### 9.2 Reintentos

```python
def con_reintentos(veces: int = 3, espera: float = 10):
    def decorador(funcion):
        @functools.wraps(funcion)
        def envoltorio(*args, **kwargs):
            for intento in range(1, veces + 1):
                try:
                    return funcion(*args, **kwargs)
                except Exception as e:
                    log.warning("%s · intento %d/%d fallido: %s", funcion.__name__, intento, veces, e)
                    if intento < veces:
                        time.sleep(espera)
            log.error("%s ha fallado %d veces", funcion.__name__, veces)
        return envoltorio
    return decorador
```

### 9.3 Registrar duración de cada tarea

```python
def medir(funcion):
    @functools.wraps(funcion)
    def envoltorio(*args, **kwargs):
        inicio = time.perf_counter()
        try:
            return funcion(*args, **kwargs)
        finally:
            log.info("%s completada en %.2f s", funcion.__name__, time.perf_counter() - inicio)
    return envoltorio
```

---

## 10. Tareas largas y concurrencia

Schedule ejecuta las tareas **de forma secuencial**. Si una tarea tarda 10 minutos, el resto espera. Soluciones:

### 10.1 Ejecutar cada tarea en un hilo

```python
import threading


def en_hilo(funcion):
    @functools.wraps(funcion)
    def envoltorio(*args, **kwargs):
        hilo = threading.Thread(target=funcion, args=args, kwargs=kwargs, daemon=True)
        hilo.start()
    return envoltorio


schedule.every(1).minutes.do(en_hilo(descarga_lenta))
```

### 10.2 Evitar ejecuciones solapadas

Si una tarea aún no ha terminado cuando llega su siguiente ejecución:

```python
def sin_solapamiento(funcion):
    cerrojo = threading.Lock()

    @functools.wraps(funcion)
    def envoltorio(*args, **kwargs):
        if not cerrojo.acquire(blocking=False):
            log.warning("%s sigue en ejecución; se omite esta vez", funcion.__name__)
            return
        try:
            return funcion(*args, **kwargs)
        finally:
            cerrojo.release()
    return envoltorio


schedule.every(1).minutes.do(en_hilo(sin_solapamiento(sincronizar)))
```

### 10.3 Cola de trabajos con un hilo trabajador

```python
import queue

cola = queue.Queue()

def trabajador():
    while True:
        funcion = cola.get()
        try:
            funcion()
        finally:
            cola.task_done()

threading.Thread(target=trabajador, daemon=True).start()

schedule.every(10).seconds.do(cola.put, tarea_a)
schedule.every(30).seconds.do(cola.put, tarea_b)
```

### 10.4 Programador en segundo plano

Para mantener el programa principal libre (por ejemplo, en una aplicación con interfaz):

```python
def iniciar_programador(intervalo: float = 1) -> threading.Event:
    detener = threading.Event()

    def bucle():
        while not detener.is_set():
            schedule.run_pending()
            detener.wait(intervalo)

    threading.Thread(target=bucle, daemon=True).start()
    return detener

detener = iniciar_programador()
# ... resto de la aplicación ...
detener.set()                     # Para finalizar
```

---

## 11. Ejecución permanente del programador

### 11.1 Bucle principal eficiente

```python
while True:
    schedule.run_pending()
    espera = schedule.idle_seconds()
    if espera is None:                 # No quedan tareas
        break
    time.sleep(max(0, min(espera, 60)))
```

### 11.2 Finalización ordenada (Ctrl+C)

```python
try:
    while True:
        schedule.run_pending()
        time.sleep(1)
except KeyboardInterrupt:
    log.info("Programador detenido por el usuario")
```

### 11.3 Mantener el script siempre activo

Schedule necesita que el proceso esté en ejecución. Opciones según el sistema:

**Windows — Programador de tareas** (iniciar el script al arrancar el equipo):

```bat
schtasks /Create /TN "MisTareasPython" /SC ONSTART /RU SYSTEM ^
         /TR "\"C:\Python312\pythonw.exe\" \"C:\scripts\programador.py\""
```

> `pythonw.exe` ejecuta el script sin ventana de consola.

**Linux — servicio systemd** (`/etc/systemd/system/programador.service`):

```ini
[Unit]
Description=Programador de tareas Python
After=network-online.target

[Service]
WorkingDirectory=/opt/tareas
ExecStart=/opt/tareas/.venv/bin/python programador.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now programador
journalctl -u programador -f         # Ver registros
```

**Docker**: ejecute el script como proceso principal del contenedor con `restart: unless-stopped`.

> Si el equipo se suspende o el programa se reinicia, las ejecuciones perdidas **no se recuperan**.

---

## 12. Schedule frente a otras alternativas

| Herramienta | Ventajas | Inconvenientes | Recomendada para |
|---|---|---|---|
| **Schedule** | Muy sencilla, legible, sin dependencias. | Requiere proceso activo; sin persistencia; secuencial. | Scripts y tareas ligeras. |
| **APScheduler** | Expresiones cron, persistencia en BD, hilos/procesos, recuperación de ejecuciones perdidas. | Más compleja. | Aplicaciones con requisitos avanzados. |
| **Programador de tareas (Windows)** / **cron (Linux)** | Del sistema; no necesita proceso activo; sobrevive a reinicios. | Configuración fuera de Python. | Ejecutar scripts a horas fijas. |
| **Celery Beat** | Distribuido, escalable. | Requiere broker (Redis/RabbitMQ). | Sistemas en producción a gran escala. |
| **Airflow / Prefect** | Flujos de datos complejos, dependencias, interfaz web. | Infraestructura pesada. | Orquestación de procesos de datos. |

> Si solo necesita ejecutar un script a una hora fija, el **Programador de tareas de Windows** o **cron** suelen ser más robustos que mantener un proceso de Python siempre activo.

---

## 13. Ejemplo práctico completo

Programador de tareas de oficina con: configuración centralizada, registro en archivo, protección frente a errores, ejecución en hilos sin solapamiento, tareas de días laborables, monitorización de una web, limpieza de temporales, informe de estado y finalización ordenada.

```python
"""
Programador de tareas de oficina.
Requisitos: pip install schedule requests pytz
"""
import functools
import logging
import shutil
import signal
import threading
import time
from datetime import date, datetime, timedelta
from logging.handlers import RotatingFileHandler
from pathlib import Path

import requests
import schedule

# ---------------------------------------------------------------- Configuración
RAIZ = Path(__file__).resolve().parent
CARPETA_DATOS = RAIZ / "datos"
CARPETA_BACKUP = RAIZ / "backups"
CARPETA_TEMP = RAIZ / "temp"
URL_MONITOR = "https://www.python.org"
ZONA = "Europe/Madrid"
DIAS_LABORABLES = ("monday", "tuesday", "wednesday", "thursday", "friday")

# ---------------------------------------------------------------- Registro
(RAIZ / "logs").mkdir(exist_ok=True)
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-7s | %(threadName)-12s | %(message)s",
    handlers=[
        RotatingFileHandler(RAIZ / "logs" / "programador.log", maxBytes=2_000_000,
                            backupCount=5, encoding="utf-8"),
        logging.StreamHandler(),
    ],
)
log = logging.getLogger("programador")


# ---------------------------------------------------------------- Decoradores
def tarea(funcion):
    """Ejecuta la función en un hilo, evita solapamientos, mide la duración y captura errores."""
    cerrojo = threading.Lock()

    @functools.wraps(funcion)
    def ejecutar():
        if not cerrojo.acquire(blocking=False):
            log.warning("«%s» sigue en ejecución; se omite", funcion.__name__)
            return
        inicio = time.perf_counter()
        try:
            log.info("▶ Inicio «%s»", funcion.__name__)
            funcion()
            log.info("✔ Fin «%s» (%.1f s)", funcion.__name__, time.perf_counter() - inicio)
        except Exception:
            log.exception("✖ Error en «%s»", funcion.__name__)
        finally:
            cerrojo.release()

    @functools.wraps(funcion)
    def lanzar():
        threading.Thread(target=ejecutar, name=funcion.__name__[:12], daemon=True).start()

    return lanzar


# ---------------------------------------------------------------- Tareas
@tarea
def copia_seguridad():
    if not CARPETA_DATOS.exists():
        log.warning("No existe la carpeta de datos: %s", CARPETA_DATOS)
        return
    CARPETA_BACKUP.mkdir(exist_ok=True)
    nombre = CARPETA_BACKUP / f"datos_{datetime.now():%Y%m%d_%H%M}"
    archivo = shutil.make_archive(str(nombre), "zip", CARPETA_DATOS)
    log.info("Copia creada: %s", Path(archivo).name)

    copias = sorted(CARPETA_BACKUP.glob("datos_*.zip"), reverse=True)
    for antigua in copias[7:]:
        antigua.unlink()
        log.info("Copia antigua eliminada: %s", antigua.name)


@tarea
def monitorizar_web():
    try:
        r = requests.get(URL_MONITOR, timeout=10)
        ms = r.elapsed.total_seconds() * 1000
        if r.ok:
            log.info("Web OK · %d · %.0f ms", r.status_code, ms)
        else:
            log.error("Web con error · código %d", r.status_code)
    except requests.RequestException as e:
        log.error("Web no accesible: %s", e)


@tarea
def limpiar_temporales():
    if not CARPETA_TEMP.exists():
        return
    limite = time.time() - 7 * 86400
    eliminados = 0
    for archivo in CARPETA_TEMP.rglob("*"):
        if archivo.is_file() and archivo.stat().st_mtime < limite:
            archivo.unlink(missing_ok=True)
            eliminados += 1
    log.info("Temporales eliminados: %d", eliminados)


@tarea
def informe_diario():
    ayer = date.today() - timedelta(days=1)
    log.info("Generando informe del %s ...", ayer.strftime("%d/%m/%Y"))
    time.sleep(2)                                   # Simula la generación
    log.info("Informe enviado")


@tarea
def cierre_mensual():
    if date.today().day != 1:
        return
    log.info("Ejecutando cierre del mes anterior")


def estado():
    log.info("Tareas programadas: %d · próxima: %s", len(schedule.get_jobs()),
             schedule.next_run().strftime("%d/%m %H:%M:%S") if schedule.next_run() else "—")
    for j in sorted(schedule.get_jobs(), key=lambda j: j.next_run):
        log.info("   %-22s → %s  %s", j.job_func.__name__, j.next_run.strftime("%d/%m %H:%M"),
                 sorted(j.tags))


# ---------------------------------------------------------------- Programación
def programar():
    schedule.every().day.at("02:00", ZONA).do(copia_seguridad).tag("mantenimiento")
    schedule.every(5).minutes.do(monitorizar_web).tag("monitorizacion")
    schedule.every().sunday.at("03:00", ZONA).do(limpiar_temporales).tag("mantenimiento")
    schedule.every().day.at("06:00", ZONA).do(cierre_mensual).tag("informes")
    for dia in DIAS_LABORABLES:
        getattr(schedule.every(), dia).at("08:30", ZONA).do(informe_diario).tag("informes")
    schedule.every().hour.do(estado).tag("sistema")


# ---------------------------------------------------------------- Bucle principal
def main():
    detener = threading.Event()

    def senal(signum, _frame):
        log.info("Señal %s recibida; deteniendo…", signum)
        detener.set()

    signal.signal(signal.SIGINT, senal)
    signal.signal(signal.SIGTERM, senal)

    programar()
    log.info("Programador iniciado")
    monitorizar_web()                                # Comprobación inicial
    estado()

    while not detener.is_set():
        schedule.run_pending()
        espera = schedule.idle_seconds() or 1
        detener.wait(timeout=max(0.5, min(espera, 30)))

    schedule.clear()
    log.info("Programador detenido")


if __name__ == "__main__":
    main()
```

---

## 14. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| Las tareas no se ejecutan nunca | Falta el bucle con `run_pending()`. | Añada `while True: schedule.run_pending(); time.sleep(1)`. |
| El programa termina tras un error en una tarea | Excepción no capturada. | Proteja las tareas con `try/except` o un decorador. |
| `IntervalError: Use seconds instead of second` | Unidad en singular con `every(n)`. | Use plural: `every(5).seconds`. |
| `ScheduleValueError: Invalid time format` | Formato incorrecto en `.at()`. | Use `"HH:MM"` para días, `":MM"` para horas, `":SS"` para minutos. |
| La tarea se ejecuta a una hora distinta | El servidor usa otra zona horaria (UTC). | Indique la zona en `.at("09:00", "Europe/Madrid")`. |
| `ModuleNotFoundError: No module named 'pytz'` | Se indicó una zona horaria en `.at()`. | `pip install pytz`. |
| Una tarea retrasa a las demás | Ejecución secuencial. | Ejecute las tareas largas en hilos. |
| Ejecuciones solapadas | La tarea dura más que su intervalo. | Use un cerrojo (`Lock`) para omitir ejecuciones. |
| La tarea no se ejecutó durante la noche | El equipo se suspendió o el script se cerró. | Ejecútelo como servicio o use el programador del sistema. |
| `.do(tarea())` ejecuta la tarea solo una vez al inicio | Se pasó el resultado en lugar de la función. | Pase la función sin paréntesis: `.do(tarea)`. |
| CPU alta | Bucle sin pausa. | Incluya `time.sleep()` en el bucle. |

---

## 15. Buenas prácticas

- Proteja **todas** las tareas frente a excepciones y registre los errores.
- Use `logging` con archivo rotativo para tener un historial de ejecuciones.
- Ejecute las tareas largas en hilos y evite solapamientos.
- Indique la zona horaria en tareas a hora fija.
- Use etiquetas para organizar y gestionar grupos de tareas.
- Centralice la programación en una función (`programar()`) y la configuración en constantes.
- Implemente una finalización ordenada (Ctrl+C, señales).
- Ejecute el programador como servicio para que sobreviva a reinicios.
- Si necesita persistencia o recuperación de ejecuciones perdidas, valore **APScheduler** o el programador del sistema operativo.

---

## 16. Referencia rápida

```python
schedule.every(n).seconds | minutes | hours | days | weeks .do(func, *args, **kwargs)
schedule.every().second | minute | hour | day | week .do(func)
schedule.every().monday ... sunday .at("HH:MM", "Europe/Madrid").do(func)
schedule.every().hour.at(":30") / every().minute.at(":15")
schedule.every(5).to(10).minutes.do(func)
.until("18:00" | "2026-12-31 18:00" | timedelta | datetime)
.tag("etiqueta", ...)

schedule.run_pending() / run_all() / idle_seconds() / next_run()
schedule.get_jobs(tag) / clear(tag) / cancel_job(job)
return schedule.CancelJob                    # Ejecutar una sola vez
@repeat(every(10).minutes)                   # Decorador
schedule.Scheduler()                         # Programador independiente
```

---

## 17. Ejercicios propuestos

1. Programe una tarea que, cada día laborable a las 9:00, muestre un resumen del espacio libre en disco.
2. Cree un monitor que compruebe tres URLs cada 2 minutos y registre los tiempos de respuesta en un CSV.
3. Implemente un recordatorio que se ejecute una sola vez en una fecha concreta y se cancele después.
4. Ejecute una copia de seguridad cada noche y elimine automáticamente las copias de más de 30 días.
5. Configure el script del ejemplo completo como servicio del sistema (systemd o Programador de tareas).

---

## 18. Referencias

- Documentación oficial: https://schedule.readthedocs.io/
- Repositorio: https://github.com/dbader/schedule
- APScheduler: https://apscheduler.readthedocs.io/
