# Manual de uso: tqdm

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Barras de progreso rápidas y ligeras para bucles e iterables
> **Tipo:** Librería de terceros (`pip install tqdm`) · **Versión de referencia:** 4.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Uso básico](#4-uso-básico)
5. [Parámetros principales](#5-parámetros-principales)
6. [Control manual de la barra](#6-control-manual-de-la-barra)
7. [Información dinámica: descripción y *postfix*](#7-información-dinámica-descripción-y-postfix)
8. [Formato personalizado (`bar_format`)](#8-formato-personalizado-bar_format)
9. [Imprimir y registrar sin romper la barra](#9-imprimir-y-registrar-sin-romper-la-barra)
10. [Barras anidadas y múltiples](#10-barras-anidadas-y-múltiples)
11. [Hilos y procesos](#11-hilos-y-procesos)
12. [asyncio](#12-asyncio)
13. [Archivos, descargas y bytes](#13-archivos-descargas-y-bytes)
14. [Integración con pandas](#14-integración-con-pandas)
15. [Jupyter y `tqdm.auto`](#15-jupyter-y-tqdmauto)
16. [tqdm en la línea de comandos](#16-tqdm-en-la-línea-de-comandos)
17. [Rendimiento y entornos sin terminal](#17-rendimiento-y-entornos-sin-terminal)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**tqdm** (del árabe *taqaddum*, «progreso») añade una barra de progreso a cualquier bucle con un cambio mínimo: envolver el iterable con `tqdm(...)`. Muestra porcentaje, elementos procesados, velocidad, tiempo transcurrido y tiempo restante estimado, con un coste de rendimiento muy bajo (del orden de decenas de nanosegundos por iteración).

**Ventajas:**
- Uso inmediato: `for x in tqdm(datos):`.
- Muy ligera y rápida; sin dependencias obligatorias.
- Funciona en terminal, Jupyter, scripts redirigidos y como comando de *shell*.
- Integraciones con pandas, `concurrent.futures`, `asyncio`, `logging` y Keras.
- Estima automáticamente velocidad y tiempo restante.

**Limitaciones:**
- Salida centrada en barras: no ofrece tablas, paneles ni pantallas complejas.
- Personalización visual limitada frente a Rich.
- Las barras múltiples pueden descolocarse en algunas terminales o IDE.

| Si necesita… | Opción |
|---|---|
| Barra rápida en un bucle con mínimo código | **tqdm** |
| Barras con columnas personalizadas dentro de una interfaz rica | Rich (`Progress`) |
| Indicador de actividad sin total conocido | yaspin / halo |
| Panel completo a pantalla completa | Textual |

---

## 2. Instalación e importación

```bash
pip install tqdm
```

```python
from tqdm import tqdm, trange
from tqdm.auto import tqdm            # Elige automáticamente terminal o Jupyter
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `tqdm(iterable)` | Envuelve un iterable y actualiza la barra en cada iteración. |
| `total` | Número total de pasos. Se deduce de `len()` si el iterable lo admite. |
| `update(n)` | Avanza manualmente `n` pasos. |
| `desc` | Texto que precede a la barra. |
| `postfix` | Información adicional al final de la barra (métricas). |
| `unit` | Unidad de cada paso (`it`, `B`, `archivo`, `fila`…). |
| `position` | Fila de la barra cuando hay varias. |
| `leave` | Si la barra permanece al terminar. |

Anatomía de la barra:

```
Procesando:  45%|████▌     | 450/1000 [00:09<00:11, 49.8archivo/s, errores=2]
└── desc ──┘ └%┘ └─ bar ─┘ └n/total┘ └trans.< rest., velocidad┘ └─ postfix ─┘
```

---

## 4. Uso básico

```python
import time
from tqdm import tqdm, trange

for archivo in tqdm(archivos):
    procesar(archivo)

for i in trange(100, desc="Calculando"):         # trange(n) = tqdm(range(n))
    time.sleep(0.01)

resultados = [transformar(x) for x in tqdm(datos, desc="Transformando")]
```

Con iterables sin longitud (generadores), indique el total si lo conoce:

```python
def leer_lineas(ruta):
    with open(ruta, encoding="utf-8") as f:
        yield from f

total = sum(1 for _ in open("datos.csv", encoding="utf-8"))
for linea in tqdm(leer_lineas("datos.csv"), total=total, unit="línea"):
    ...
```

Sin `total`, tqdm muestra un contador y la velocidad, pero no porcentaje ni tiempo restante.

Iterar con índice o con varios iterables:

```python
from tqdm.contrib import tenumerate, tzip, tmap

for i, fila in tenumerate(filas):
    ...
for a, b in tzip(lista_a, lista_b):
    ...
resultados = list(tmap(procesar, elementos))
```

---

## 5. Parámetros principales

```python
tqdm(
    iterable,
    desc="Importando",          # Texto inicial
    total=1000,                 # Total de pasos
    unit="fila",                # Unidad
    unit_scale=True,            # 1500 → 1.50k
    unit_divisor=1000,          # 1024 para bytes binarios
    leave=True,                 # Mantener la barra al terminar
    ncols=100,                  # Ancho total (o dynamic_ncols=True)
    colour="green",             # Color de la barra (nombre o '#rrggbb')
    ascii=False,                # True o " ▏▎▍▌▋▊▉█" para caracteres personalizados
    position=0,                 # Línea (barras múltiples)
    initial=0,                  # Valor inicial (reanudar)
    mininterval=0.1,            # Segundos mínimos entre refrescos
    miniters=None,              # Iteraciones mínimas entre refrescos
    smoothing=0.3,              # 0 = media global, 1 = velocidad instantánea
    delay=0,                    # Segundos antes de mostrar la barra
    disable=False,              # Desactivar (None: desactiva si no hay terminal)
    file=None,                  # Flujo de salida (por defecto sys.stderr)
    postfix=None,               # Diccionario o texto inicial de postfix
    bar_format=None,            # Formato completo (sección 8)
)
```

| Parámetro | Uso típico |
|---|---|
| `desc` | Identificar la tarea. |
| `unit` + `unit_scale` | Mostrar `1.2M filas` o `35.4MB`. |
| `leave=False` | Barras temporales (bucles internos). |
| `delay=2` | No mostrar barra en tareas que terminan en menos de 2 s. |
| `disable=None` | Desactivar automáticamente cuando la salida no es una terminal. |
| `colour` | Distinguir barras (`"green"`, `"#E07B00"`). |

---

## 6. Control manual de la barra

Cuando el progreso no corresponde a iterar una colección (bytes, pasos de un proceso, resultados que llegan de forma irregular):

```python
with tqdm(total=100, desc="Migración", unit="%") as barra:
    crear_tablas();      barra.update(20)
    copiar_datos();      barra.update(60)
    crear_indices();     barra.update(15)
    verificar();         barra.update(5)
```

Sin `with`, cierre la barra explícitamente:

```python
barra = tqdm(total=len(tareas))
try:
    for t in tareas:
        ejecutar(t)
        barra.update()
finally:
    barra.close()
```

Otros métodos útiles:

| Método | Efecto |
|---|---|
| `update(n=1)` | Avanza `n` pasos. |
| `reset(total=None)` | Reinicia la barra (opcionalmente con otro total). |
| `refresh()` | Fuerza el redibujado. |
| `clear()` | Borra la barra de la pantalla. |
| `close()` | Finaliza la barra. |
| `unpause()` | Reanuda el cálculo de tiempos tras una pausa. |
| `n`, `total` | Valor actual y total (modificables; llame a `refresh()`). |
| `format_dict` | Diccionario con los valores actuales (velocidad, tiempos…). |

Total desconocido al principio:

```python
barra = tqdm(desc="Buscando archivos", unit="archivo")
for ruta in recorrer_carpetas():
    barra.update()
barra.total = barra.n
barra.refresh()
barra.close()
```

---

## 7. Información dinámica: descripción y *postfix*

```python
barra = tqdm(lotes, desc="Entrenando")
for epoca, lote in enumerate(barra, 1):
    perdida, precision = entrenar(lote)
    barra.set_description(f"Época {epoca}")
    barra.set_postfix(perdida=f"{perdida:.4f}", precision=f"{precision:.1%}")
```

```
Época 7:  70%|███████   | 7/10 [00:14<00:06, 2.01s/it, perdida=0.2134, precision=91.2%]
```

```python
barra.set_postfix({"errores": 3, "omitidos": 12}, refresh=False)
barra.set_postfix_str("último: informe_2026.pdf")
barra.set_description_str("Procesando sin ':' final")
```

> Llamar a `set_postfix` en cada iteración tiene coste: con millones de iteraciones, actualícelo cada cierto número de pasos o use `refresh=False`.

---

## 8. Formato personalizado (`bar_format`)

```python
FORMATO = "{desc}: {percentage:3.0f}%|{bar:30}| {n_fmt}/{total_fmt} [{elapsed}<{remaining}, {rate_fmt}{postfix}]"
for x in tqdm(datos, bar_format=FORMATO, desc="Exportando"):
    ...
```

| Campo | Contenido |
|---|---|
| `{l_bar}` | Parte izquierda por defecto (`desc: 45%|`). |
| `{bar}` / `{bar:30}` / `{bar:-10}` | La barra (ancho fijo o ancho restante menos N). |
| `{r_bar}` | Parte derecha por defecto. |
| `{n}`, `{n_fmt}` | Pasos completados. |
| `{total}`, `{total_fmt}` | Total. |
| `{percentage}` | Porcentaje (número). |
| `{elapsed}`, `{remaining}` | Tiempo transcurrido y restante. |
| `{elapsed_s}`, `{remaining_s}` | Los mismos en segundos. |
| `{rate}`, `{rate_fmt}`, `{rate_inv_fmt}` | Velocidad. |
| `{desc}`, `{unit}`, `{postfix}` | Descripción, unidad y *postfix*. |
| `{eta}` | Hora estimada de finalización (`datetime`). |

Ejemplos:

```python
# Solo contador y tiempo, sin barra
bar_format="{desc}: {n_fmt} elementos [{elapsed}, {rate_fmt}]"

# Barra con colores ANSI y hora estimada de finalización
bar_format="{l_bar}\x1b[32m{bar}\x1b[0m| {n_fmt}/{total_fmt} · fin aprox. {eta:%H:%M}"
```

---

## 9. Imprimir y registrar sin romper la barra

`print()` normal se mezcla con la barra. Use `tqdm.write`:

```python
for archivo in tqdm(archivos):
    if not valido(archivo):
        tqdm.write(f"⚠ Archivo omitido: {archivo}")
```

Con `logging`:

```python
import logging
from tqdm.contrib.logging import logging_redirect_tqdm

logging.basicConfig(level=logging.INFO)
log = logging.getLogger("importador")

with logging_redirect_tqdm():
    for fila in tqdm(filas):
        if fila.incompleta:
            log.warning("Fila %s incompleta", fila.id)
```

Redirigir todo `stdout` temporalmente:

```python
import contextlib, sys

@contextlib.contextmanager
def stdout_por_tqdm():
    original = sys.stdout
    class Escritor:
        def write(self, texto):
            if texto.rstrip():
                tqdm.write(texto.rstrip(), file=original)
        def flush(self):
            original.flush()
    sys.stdout = Escritor()
    try:
        yield original
    finally:
        sys.stdout = original
```

---

## 10. Barras anidadas y múltiples

### 10.1 Bucles anidados

```python
for cliente in tqdm(clientes, desc="Clientes", position=0):
    for factura in tqdm(cliente.facturas, desc="Facturas", position=1, leave=False):
        procesar(factura)
```

`leave=False` en la barra interna evita que se acumulen líneas.

### 10.2 Barras paralelas con posición fija

```python
from threading import Thread

def trabajo(n, posicion):
    for _ in tqdm(range(n), desc=f"Hilo {posicion}", position=posicion, leave=True):
        time.sleep(0.01)

hilos = [Thread(target=trabajo, args=(200 + i * 50, i)) for i in range(4)]
for h in hilos: h.start()
for h in hilos: h.join()
```

> Las barras múltiples dependen de que la terminal admita movimiento del cursor. En algunos IDE (consola de PyCharm, salida de VS Code no interactiva) pueden mostrarse en líneas repetidas.

---

## 11. Hilos y procesos

### 11.1 `thread_map` y `process_map`

Equivalen a `ThreadPoolExecutor.map` / `ProcessPoolExecutor.map` con barra incluida:

```python
from tqdm.contrib.concurrent import thread_map, process_map

paginas = thread_map(descargar, urls, max_workers=8, desc="Descargando")          # E/S: hilos
resultados = process_map(calcular, datos, max_workers=4, chunksize=50, desc="Cálculo")  # CPU: procesos
```

> `process_map` debe llamarse dentro de `if __name__ == "__main__":` (requisito de `multiprocessing` en Windows y macOS).

### 11.2 `concurrent.futures` con `as_completed`

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor(max_workers=8) as ex:
    futuros = {ex.submit(descargar, url): url for url in urls}
    resultados, errores = {}, {}
    for futuro in tqdm(as_completed(futuros), total=len(futuros), desc="Descargas"):
        url = futuros[futuro]
        try:
            resultados[url] = futuro.result()
        except Exception as e:
            errores[url] = e
```

### 11.3 Bloqueo compartido en `multiprocessing`

```python
from multiprocessing import Pool, RLock, freeze_support

def trabajo(args):
    indice, n = args
    for _ in tqdm(range(n), desc=f"Proceso {indice}", position=indice, leave=False):
        time.sleep(0.001)
    return indice

if __name__ == "__main__":
    freeze_support()
    tqdm.set_lock(RLock())
    with Pool(4, initializer=tqdm.set_lock, initargs=(tqdm.get_lock(),)) as pool:
        list(pool.imap_unordered(trabajo, [(i, 1000) for i in range(4)]))
```

---

## 12. asyncio

```python
import asyncio
from tqdm.asyncio import tqdm as atqdm

async def descargar(url):
    await asyncio.sleep(0.1)
    return url

async def main(urls):
    # Equivalente a asyncio.gather con barra
    resultados = await atqdm.gather(*(descargar(u) for u in urls), desc="Descargas")

    # Procesar según terminan
    for corrutina in atqdm.as_completed([descargar(u) for u in urls], desc="Según llegan"):
        r = await corrutina

    # Iterar un generador asíncrono
    async for elemento in atqdm(generador_asincrono(), total=100):
        ...

asyncio.run(main([f"https://api.ejemplo.com/{i}" for i in range(50)]))
```

---

## 13. Archivos, descargas y bytes

### 13.1 Unidades de bytes

```python
tqdm(total=tamano_bytes, unit="B", unit_scale=True, unit_divisor=1024, desc="Copiando")
# 34.5MB/120MB [00:03<00:07, 11.2MB/s]
```

### 13.2 Copia de archivos con progreso

```python
from pathlib import Path

def copiar(origen: Path, destino: Path, bloque: int = 1024 * 1024) -> None:
    with origen.open("rb") as fo, destino.open("wb") as fd, \
         tqdm(total=origen.stat().st_size, unit="B", unit_scale=True, unit_divisor=1024,
              desc=origen.name) as barra:
        while datos := fo.read(bloque):
            fd.write(datos)
            barra.update(len(datos))
```

### 13.3 Envolver un archivo (`wrapattr`)

```python
import shutil

with open("origen.iso", "rb") as fo, open("copia.iso", "wb") as fd:
    with tqdm.wrapattr(fo, "read", total=Path("origen.iso").stat().st_size, desc="Copia") as fo_progreso:
        shutil.copyfileobj(fo_progreso, fd)
```

### 13.4 Descarga HTTP (con `requests`)

```python
import requests

def descargar(url: str, destino: Path) -> None:
    with requests.get(url, stream=True, timeout=30) as r:
        r.raise_for_status()
        total = int(r.headers.get("content-length", 0)) or None
        with destino.open("wb") as f, tqdm(total=total, unit="B", unit_scale=True,
                                           unit_divisor=1024, desc=destino.name) as barra:
            for bloque in r.iter_content(chunk_size=65536):
                barra.update(f.write(bloque))
```

### 13.5 Callbacks de librerías (`CallbackIOWrapper`, `update_to`)

```python
import urllib.request

class BarraDescarga(tqdm):
    def update_to(self, bloques=1, tam_bloque=1, total=None):
        if total is not None:
            self.total = total
        self.update(bloques * tam_bloque - self.n)

with BarraDescarga(unit="B", unit_scale=True, unit_divisor=1024, desc="archivo.zip") as b:
    urllib.request.urlretrieve(url, "archivo.zip", reporthook=b.update_to)
```

---

## 14. Integración con pandas

```python
import pandas as pd
from tqdm import tqdm

tqdm.pandas(desc="Normalizando")                 # Registra los métodos progress_*

df["email"] = df["email"].progress_apply(str.lower)
df["puntuacion"] = df.progress_apply(calcular_puntuacion, axis=1)
resumen = df.groupby("region").progress_apply(agregar)
df["categoria"] = df["codigo"].progress_map(CATEGORIAS)
```

Lectura por bloques de un CSV grande:

```python
total_filas = sum(1 for _ in open("ventas.csv", encoding="utf-8")) - 1
partes = []
with tqdm(total=total_filas, unit="fila", unit_scale=True, desc="Leyendo") as barra:
    for bloque in pd.read_csv("ventas.csv", chunksize=50_000):
        partes.append(limpiar(bloque))
        barra.update(len(bloque))
df = pd.concat(partes, ignore_index=True)
```

---

## 15. Jupyter y `tqdm.auto`

```python
from tqdm.auto import tqdm, trange       # Widget en Jupyter, texto en terminal
from tqdm.notebook import tqdm           # Forzar el widget de Jupyter
```

El widget requiere `ipywidgets` (`pip install ipywidgets`). Sin él, `tqdm.auto` muestra la versión de texto.

---

## 16. tqdm en la línea de comandos

tqdm puede insertarse en una tubería de *shell* para mostrar el progreso de datos que pasan por ella:

```bash
# Contar líneas mostrando el progreso
cat registros.log | python -m tqdm | wc -l

# Comprimir mostrando bytes procesados
tar -cf - carpeta/ | python -m tqdm --bytes --total $(du -sb carpeta | cut -f1) | gzip > copia.tar.gz

# Con total y descripción
seq 1 1000000 | python -m tqdm --total 1000000 --desc "Números" --unit num > /dev/null
```

Opciones: los mismos nombres que en Python (`--desc`, `--total`, `--unit`, `--unit_scale`, `--ncols`…), más `--bytes`, `--delim` y `--update`.

---

## 17. Rendimiento y entornos sin terminal

### 17.1 Rendimiento

- tqdm solo redibuja cada `mininterval` segundos (0,1 s por defecto), por lo que su coste por iteración es muy bajo.
- En bucles de millones de iteraciones muy rápidas, aumente `miniters` o `mininterval`, o actualice manualmente por lotes:

```python
with tqdm(total=len(datos), mininterval=0.5) as barra:
    for i in range(0, len(datos), 10_000):
        procesar_lote(datos[i:i + 10_000])
        barra.update(min(10_000, len(datos) - i))
```

### 17.2 Logs, CI y redirección

```python
import sys
tqdm(datos, disable=not sys.stderr.isatty())     # Desactivar si no hay terminal
tqdm(datos, disable=None)                         # Igual, automáticamente
tqdm(datos, mininterval=30, ascii=True)           # En CI: pocas líneas y solo ASCII
```

### 17.3 Configuración por variables de entorno

Cualquier parámetro puede fijarse con `TQDM_<PARÁMETRO>` sin tocar el código:

```bash
TQDM_DISABLE=1 python script.py              # Desactiva todas las barras
TQDM_MININTERVAL=5 python script.py          # Refresco cada 5 s (logs de CI)
TQDM_NCOLS=80 TQDM_ASCII=1 python script.py
```

---

## 18. Ejemplo práctico completo

**Procesador de imágenes por lotes** (simulado sin dependencias): recorre una carpeta, procesa los archivos en paralelo con hilos, muestra barra global con métricas en el *postfix*, barra de bytes para la copia de resultados, registra advertencias con `logging_redirect_tqdm`, desactiva las barras si no hay terminal y genera un informe final.

```python
"""
lotes.py — Procesamiento por lotes de archivos con barras de progreso.
Requiere: pip install tqdm
Uso: python lotes.py <carpeta_origen> <carpeta_destino> [--hilos N] [--demo]
"""
import argparse
import hashlib
import logging
import random
import shutil
import sys
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass
from pathlib import Path

from tqdm import tqdm
from tqdm.contrib.logging import logging_redirect_tqdm

log = logging.getLogger("lotes")
EXTENSIONES = {".jpg", ".jpeg", ".png", ".gif", ".webp"}
SIN_TERMINAL = not sys.stderr.isatty()


@dataclass
class Resultado:
    origen: Path
    destino: Path | None
    bytes_entrada: int
    bytes_salida: int = 0
    error: str | None = None


# ------------------------------------------------------------------ Preparación
def crear_demo(carpeta: Path, n: int = 120) -> None:
    """Genera archivos de prueba con tamaños aleatorios."""
    carpeta.mkdir(parents=True, exist_ok=True)
    for i in tqdm(range(n), desc="Creando demo", unit="archivo", leave=False, disable=SIN_TERMINAL):
        ext = random.choice([".jpg", ".png", ".webp", ".txt"])
        (carpeta / f"imagen_{i:04d}{ext}").write_bytes(random.randbytes(random.randint(20_000, 400_000)))


def buscar(origen: Path) -> list[Path]:
    encontrados = []
    with tqdm(desc="Buscando archivos", unit="archivo", disable=SIN_TERMINAL) as barra:
        for ruta in origen.rglob("*"):
            if ruta.is_file() and ruta.suffix.lower() in EXTENSIONES:
                encontrados.append(ruta)
                barra.update()
    return encontrados


# ------------------------------------------------------------------ Procesamiento
def procesar(ruta: Path, destino: Path) -> Resultado:
    """Simula una transformación: lee, 'comprime' y escribe el resultado."""
    datos = ruta.read_bytes()
    resultado = Resultado(ruta, None, len(datos))
    time.sleep(random.uniform(0.01, 0.06))                      # Trabajo simulado
    if random.random() < 0.04:
        resultado.error = "formato dañado"
        return resultado
    salida = destino / f"{ruta.stem}_{hashlib.md5(datos).hexdigest()[:8]}.bin"
    salida.write_bytes(datos[: int(len(datos) * random.uniform(0.4, 0.8))])
    resultado.destino, resultado.bytes_salida = salida, salida.stat().st_size
    return resultado


def procesar_todo(archivos: list[Path], destino: Path, hilos: int) -> list[Resultado]:
    destino.mkdir(parents=True, exist_ok=True)
    resultados: list[Resultado] = []
    ahorro = errores = 0
    with logging_redirect_tqdm(), ThreadPoolExecutor(max_workers=hilos) as ex, \
            tqdm(total=len(archivos), desc="Procesando", unit="img", colour="green",
                 dynamic_ncols=True, disable=SIN_TERMINAL) as barra:
        futuros = {ex.submit(procesar, a, destino): a for a in archivos}
        for futuro in as_completed(futuros):
            try:
                r = futuro.result()
            except Exception as e:                                # Error inesperado
                r = Resultado(futuros[futuro], None, 0, error=str(e))
            resultados.append(r)
            if r.error:
                errores += 1
                log.warning("%s: %s", r.origen.name, r.error)
            else:
                ahorro += r.bytes_entrada - r.bytes_salida
            barra.set_postfix(errores=errores, ahorro=f"{ahorro / 1_048_576:.1f}MB", refresh=False)
            barra.update()
    return resultados


def empaquetar(resultados: list[Resultado], destino: Path) -> Path:
    """Copia los resultados a un único archivo mostrando el progreso en bytes."""
    validos = [r for r in resultados if r.destino]
    total = sum(r.bytes_salida for r in validos)
    paquete = destino / "paquete.dat"
    with paquete.open("wb") as fd, tqdm(total=total, unit="B", unit_scale=True, unit_divisor=1024,
                                        desc="Empaquetando", colour="cyan", disable=SIN_TERMINAL) as barra:
        for r in validos:
            with r.destino.open("rb") as fo:
                with tqdm.wrapattr(fo, "read", total=r.bytes_salida, leave=False,
                                   desc=r.destino.name[:20], disable=SIN_TERMINAL) as fo_progreso:
                    shutil.copyfileobj(fo_progreso, fd, length=64 * 1024)
            barra.update(r.bytes_salida)
    return paquete


# ------------------------------------------------------------------ Informe
def informe(resultados: list[Resultado], segundos: float, paquete: Path) -> None:
    ok = [r for r in resultados if not r.error]
    entrada = sum(r.bytes_entrada for r in ok)
    salida = sum(r.bytes_salida for r in ok)
    print("\nResumen")
    print("-------")
    print(f"Procesados correctamente : {len(ok)}/{len(resultados)}")
    print(f"Errores                  : {len(resultados) - len(ok)}")
    print(f"Tamaño original          : {entrada / 1_048_576:8.2f} MB")
    print(f"Tamaño resultante        : {salida / 1_048_576:8.2f} MB "
          f"({(1 - salida / entrada) if entrada else 0:.0%} de ahorro)")
    print(f"Tiempo total             : {segundos:8.2f} s ({len(resultados) / segundos:.1f} img/s)")
    print(f"Paquete                  : {paquete}")


def main() -> int:
    parser = argparse.ArgumentParser(description="Procesamiento de imágenes por lotes.")
    parser.add_argument("origen", type=Path)
    parser.add_argument("destino", type=Path)
    parser.add_argument("--hilos", type=int, default=8)
    parser.add_argument("--demo", action="store_true", help="genera archivos de prueba en ORIGEN")
    args = parser.parse_args()
    logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")

    if args.demo:
        crear_demo(args.origen)
    if not args.origen.is_dir():
        parser.error(f"la carpeta {args.origen} no existe")

    archivos = buscar(args.origen)
    if not archivos:
        log.error("No se encontraron imágenes en %s", args.origen)
        return 1

    inicio = time.perf_counter()
    try:
        resultados = procesar_todo(archivos, args.destino, args.hilos)
        paquete = empaquetar(resultados, args.destino)
    except KeyboardInterrupt:
        tqdm.write("\nProceso interrumpido por el usuario.")
        return 130
    informe(resultados, time.perf_counter() - inicio, paquete)
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

Uso:

```bash
python lotes.py ./fotos ./salida --demo --hilos 8
TQDM_DISABLE=1 python lotes.py ./fotos ./salida        # Sin barras (p. ej., en CI)
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| La barra se repite en muchas líneas | `print()` dentro del bucle o IDE sin soporte de retorno de carro. | `tqdm.write()`, `logging_redirect_tqdm()` o `dynamic_ncols=True`. |
| Sin porcentaje ni tiempo restante | El iterable no tiene `len()` y no se indicó `total`. | Pase `total=`. |
| `TypeError: 'module' object is not callable` | `import tqdm` y luego `tqdm(...)`. | `from tqdm import tqdm`. |
| La barra no llega al 100 % | Se interrumpió el bucle o el `total` era mayor. | Ajuste el total o use `with` para cerrar la barra. |
| Barras anidadas acumuladas | La barra interna conserva su línea. | `leave=False` en las internas. |
| Barras desordenadas con procesos | Sin bloqueo compartido ni `position`. | `tqdm.set_lock(RLock())` e `initializer`. |
| Los logs de CI se llenan de líneas | Refresco frecuente sin terminal. | `disable=None`, `mininterval=30` o `TQDM_DISABLE=1`. |
| Widget no visible en Jupyter | Falta `ipywidgets`. | `pip install ipywidgets`. |
| `RuntimeError` con `process_map` en Windows | Falta la protección del punto de entrada. | `if __name__ == "__main__":`. |
| Velocidad errática | Iteraciones de duración muy irregular. | Ajuste `smoothing` (0 para media global). |

---

## 20. Buenas prácticas

- Envuelva el iterable directamente (`tqdm(datos)`) siempre que sea posible.
- Use `with tqdm(...) as barra:` en el modo manual para garantizar el cierre.
- Indique `desc`, `unit` y `total`: hacen la barra informativa.
- Use `leave=False` en barras internas o temporales.
- Imprima con `tqdm.write()` y redirija `logging` con `logging_redirect_tqdm`.
- Use `disable=None` (o una opción `--quiet`) para no ensuciar logs cuando no hay terminal.
- Prefiera `thread_map`/`process_map` para paralelizar con barra en una sola línea.
- No actualice el *postfix* en cada iteración de bucles muy rápidos.
- Si la aplicación ya usa Rich para su salida, use también las barras de Rich para mantener un estilo coherente.

---

## 21. Referencia rápida

```python
from tqdm import tqdm, trange; from tqdm.auto import tqdm
for x in tqdm(it, desc=, total=, unit=, unit_scale=, leave=, position=, colour=, disable=None): ...
trange(n) · tenumerate(it) · tzip(a, b) · tmap(f, it)
with tqdm(total=N) as b: b.update(n); b.set_description(t); b.set_postfix(k=v); b.reset(); b.refresh()
tqdm.write("msg") · with logging_redirect_tqdm(): ...
bar_format="{l_bar}{bar}| {n_fmt}/{total_fmt} [{elapsed}<{remaining}, {rate_fmt}{postfix}]"
thread_map(f, it, max_workers=) · process_map(f, it, max_workers=, chunksize=)
await tqdm.asyncio.tqdm.gather(*corrutinas) · tqdm.asyncio.tqdm.as_completed(...)
tqdm(total=bytes, unit="B", unit_scale=True, unit_divisor=1024) · tqdm.wrapattr(f, "read", total=)
tqdm.pandas(); df.progress_apply(f) · df.progress_map(f)
python -m tqdm --total N --desc X --bytes · TQDM_DISABLE=1 · TQDM_MININTERVAL=5
```

---

## 22. Ejercicios propuestos

1. Añada barras de progreso a un script que calcule el hash SHA-256 de todos los archivos de una carpeta, mostrando bytes procesados y velocidad.
2. Cree una función de copia de carpetas que muestre una barra global por archivos y una barra interna por bytes para cada archivo grande.
3. Use `thread_map` para comprobar el estado HTTP de una lista de URLs y muestre en el *postfix* cuántas han fallado.
4. Procese un CSV de un millón de filas por bloques con pandas mostrando filas por segundo.
5. Modifique el ejemplo completo para que acepte `--quiet` y, en ese caso, desactive todas las barras sin cambiar el resto del código.

---

## 23. Referencias

- Documentación oficial: https://tqdm.github.io/
- Repositorio: https://github.com/tqdm/tqdm
- Ejemplos: https://github.com/tqdm/tqdm/tree/master/examples
