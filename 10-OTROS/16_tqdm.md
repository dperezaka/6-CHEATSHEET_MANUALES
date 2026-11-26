# Manual de uso: TQDM

> **Categoría:** Utilidades generales · **Utilidad:** Mostrar barras de progreso
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Personalización](#5-personalización)
6. [Control manual de la barra](#6-control-manual-de-la-barra)
7. [Barras anidadas y múltiples](#7-barras-anidadas-y-múltiples)
8. [Integración con Pandas](#8-integración-con-pandas)
9. [Descargas y archivos](#9-descargas-y-archivos)
10. [Concurrencia](#10-concurrencia)
11. [Entornos especiales](#11-entornos-especiales)
12. [Ejemplo práctico completo](#12-ejemplo-práctico-completo)
13. [Errores frecuentes](#13-errores-frecuentes)
14. [Buenas prácticas](#14-buenas-prácticas)
15. [Referencia rápida](#15-referencia-rápida)
16. [Ejercicios propuestos](#16-ejercicios-propuestos)
17. [Referencias](#17-referencias)

---

## 1. Descripción

**tqdm** (del árabe *taqaddum*, «progreso») añade barras de progreso a bucles e iteraciones con una sola línea de código. Muestra el porcentaje completado, el número de elementos procesados, la velocidad, el tiempo transcurrido y el tiempo restante estimado.

```
Procesando: 62%|██████▏   | 620/1000 [00:03<00:01, 205.4it/s]
```

**Casos de uso habituales:**
- Procesamiento de archivos por lotes.
- Descargas de datos.
- Operaciones sobre DataFrames.
- Entrenamiento de modelos de Machine Learning.
- Cualquier script que tarde más de unos segundos.

Su sobrecarga es mínima (del orden de decenas de nanosegundos por iteración), por lo que puede usarse incluso en bucles muy rápidos.

---

## 2. Instalación e importación

```bash
pip install tqdm
```

```python
from tqdm import tqdm, trange
from tqdm.auto import tqdm          # Elige automáticamente la versión de consola o de Jupyter
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `tqdm(iterable)` | Envuelve un iterable y muestra el progreso al recorrerlo. |
| `trange(n)` | Atajo de `tqdm(range(n))`. |
| `total` | Número total de pasos (necesario si el iterable no tiene longitud). |
| `update(n)` | Avanza la barra manualmente `n` pasos. |
| `desc` | Texto descriptivo a la izquierda de la barra. |
| `postfix` | Información adicional a la derecha (métricas, estado). |
| `unit` | Unidad mostrada (`it`, `archivos`, `B`…). |

### 3.1 Partes de la barra

```
Descripción: 45%|████▌     | 450/1000 [00:02<00:03, 180.12archivos/s, errores=3]
│            │   │          │         │      │        │                 └─ postfix
│            │   │          │         │      │        └─ velocidad y unidad
│            │   │          │         │      └─ tiempo restante estimado
│            │   │          │         └─ tiempo transcurrido
│            │   │          └─ completados / total
│            │   └─ barra
│            └─ porcentaje
└─ desc
```

---

## 4. Operaciones básicas

### 4.1 Envolver un bucle

```python
import time
from tqdm import tqdm

for i in tqdm(range(100)):
    time.sleep(0.02)
```

### 4.2 `trange`

```python
from tqdm import trange

for i in trange(100, desc="Calculando"):
    time.sleep(0.02)
```

### 4.3 Recorrer listas y otros iterables

```python
archivos = ["a.csv", "b.csv", "c.csv", "d.csv"]
for archivo in tqdm(archivos, desc="Procesando archivos", unit="archivo"):
    procesar(archivo)
```

### 4.4 Con `enumerate` y `zip`

```python
for i, elemento in enumerate(tqdm(datos)):
    ...

for a, b in tqdm(zip(lista_a, lista_b), total=len(lista_a)):
    ...
```

> Coloque `tqdm` **dentro** de `enumerate` para que la barra conozca el total. Con `zip` o generadores, indique `total` explícitamente.

### 4.5 Generadores (longitud desconocida)

```python
def leer_registros():
    for linea in open("datos.txt", encoding="utf-8"):
        yield linea

# Sin total: solo muestra el contador y la velocidad
for r in tqdm(leer_registros(), desc="Leyendo"):
    ...

# Con total conocido de antemano: barra completa
total = sum(1 for _ in open("datos.txt", encoding="utf-8"))
for r in tqdm(leer_registros(), total=total, desc="Leyendo"):
    ...
```

### 4.6 Listas por comprensión

```python
resultados = [transformar(x) for x in tqdm(datos, desc="Transformando")]
```

---

## 5. Personalización

### 5.1 Parámetros más utilizados

```python
for _ in tqdm(
    range(500),
    desc="Importando",        # Texto descriptivo
    unit="reg",               # Unidad
    unit_scale=True,          # 1500 → 1.5k
    total=500,                # Total de pasos
    ncols=90,                 # Ancho total en caracteres
    colour="green",           # Color de la barra
    leave=True,               # Mantener la barra al terminar (False la borra)
    mininterval=0.1,          # Intervalo mínimo de refresco (segundos)
    initial=0,                # Valor inicial (para reanudar)
    disable=False,            # Desactivar la barra
    position=0,               # Línea (para barras múltiples)
    dynamic_ncols=True,       # Adaptar al ancho de la terminal
    ascii=False,              # True: usa caracteres ASCII (#) en vez de bloques
    smoothing=0.3,            # Suavizado de la velocidad (0 = media global, 1 = instantánea)
):
    time.sleep(0.005)
```

### 5.2 Colores

```python
tqdm(range(100), colour="green")
tqdm(range(100), colour="#1F4E78")
```

Colores con nombre: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`.

### 5.3 Formato personalizado de la barra

```python
formato = "{desc}: {percentage:3.0f}%|{bar:30}| {n_fmt}/{total_fmt} [{elapsed} < {remaining}]"
for _ in tqdm(range(200), desc="Exportando", bar_format=formato):
    time.sleep(0.01)
```

| Variable | Contenido |
|---|---|
| `{desc}` | Descripción |
| `{percentage}` | Porcentaje |
| `{bar}` / `{bar:30}` | Barra (con ancho opcional) |
| `{n}` / `{n_fmt}` | Elementos completados |
| `{total}` / `{total_fmt}` | Total |
| `{elapsed}` | Tiempo transcurrido |
| `{remaining}` | Tiempo restante |
| `{rate_fmt}` | Velocidad |
| `{postfix}` | Información adicional |
| `{unit}` | Unidad |

### 5.4 Información dinámica: descripción y *postfix*

```python
barra = tqdm(clientes, desc="Enviando correos")
errores = 0
for cliente in barra:
    barra.set_description(f"Enviando a {cliente['nombre'][:15]:<15}")
    if not enviar(cliente):
        errores += 1
    barra.set_postfix(errores=errores, ultimo=cliente["id"])
```

Salida:

```
Enviando a Ana López      : 40%|████      | 40/100 [00:04<00:06, 9.8it/s, errores=2, ultimo=1040]
```

### 5.5 Unidades de bytes (descargas, copias)

```python
tqdm(total=tamano_bytes, unit="B", unit_scale=True, unit_divisor=1024)
# 12.3M/45.0M [00:03<00:08, 4.10MB/s]
```

---

## 6. Control manual de la barra

Cuando el progreso no corresponde a un bucle sobre un iterable.

### 6.1 Uso con `with` y `update`

```python
with tqdm(total=100, desc="Proceso por fases") as barra:
    cargar_datos()
    barra.update(20)

    limpiar_datos()
    barra.update(30)

    generar_informe()
    barra.update(50)
```

### 6.2 Avanzar por bloques

```python
total_registros = 10_000
tamano_lote = 500

with tqdm(total=total_registros, desc="Insertando en BD", unit="reg") as barra:
    for inicio in range(0, total_registros, tamano_lote):
        lote = registros[inicio:inicio + tamano_lote]
        insertar(lote)
        barra.update(len(lote))
```

### 6.3 Métodos útiles

```python
barra.update(n)                 # Avanzar n pasos
barra.set_description("Texto")
barra.set_postfix(clave=valor)
barra.set_postfix_str("texto libre")
barra.reset(total=nuevo_total)  # Reiniciar (reutilizar la barra)
barra.refresh()                 # Forzar redibujado
barra.clear()                   # Borrar de la pantalla
barra.close()                   # Cerrar (automático con with)
barra.n                         # Progreso actual
barra.total                     # Total
barra.format_dict["elapsed"]    # Tiempo transcurrido (segundos)
```

### 6.4 Escribir mensajes sin romper la barra

```python
for archivo in tqdm(archivos):
    if not archivo.exists():
        tqdm.write(f"⚠ No encontrado: {archivo}")     # En lugar de print()
```

---

## 7. Barras anidadas y múltiples

### 7.1 Bucles anidados

```python
for carpeta in tqdm(carpetas, desc="Carpetas", position=0):
    archivos = list(carpeta.glob("*.csv"))
    for archivo in tqdm(archivos, desc=f"  {carpeta.name}", position=1, leave=False):
        procesar(archivo)
```

- `position`: línea en la que se dibuja cada barra.
- `leave=False`: la barra interna desaparece al terminar cada iteración.

### 7.2 Varias barras independientes

```python
from tqdm import tqdm

descarga = tqdm(total=100, desc="Descarga", position=0)
proceso = tqdm(total=100, desc="Proceso ", position=1)

for _ in range(100):
    descarga.update(1)
    proceso.update(1)
    time.sleep(0.02)

descarga.close()
proceso.close()
```

---

## 8. Integración con Pandas

### 8.1 `progress_apply` y `progress_map`

```python
import pandas as pd
from tqdm.auto import tqdm

tqdm.pandas(desc="Calculando")                    # Registrar la integración (una vez)

df["resultado"] = df["texto"].progress_apply(limpiar_texto)
df["total"] = df.progress_apply(lambda fila: fila["uds"] * fila["precio"], axis=1)
df["categoria"] = df["codigo"].progress_map(mapa_categorias)
```

### 8.2 Operaciones agrupadas

```python
resumen = df.groupby("region").progress_apply(calcular_indicadores)
```

### 8.3 Leer un CSV grande por bloques

```python
from pathlib import Path

ruta = Path("ventas_grandes.csv")
total_filas = sum(1 for _ in ruta.open(encoding="utf-8")) - 1
tamano = 50_000

partes = []
with tqdm(total=total_filas, desc="Leyendo CSV", unit="filas", unit_scale=True) as barra:
    for bloque in pd.read_csv(ruta, chunksize=tamano):
        partes.append(bloque[bloque["importe"] > 0])
        barra.update(len(bloque))

df = pd.concat(partes, ignore_index=True)
```

> Para operaciones vectorizadas de Pandas (`df["a"] * df["b"]`) no se necesita barra: son instantáneas. `progress_apply` es útil cuando se aplica una función Python fila a fila.

---

## 9. Descargas y archivos

### 9.1 Descarga de archivos con `requests`

```python
import requests
from tqdm import tqdm


def descargar(url: str, destino: str) -> None:
    with requests.get(url, stream=True, timeout=30) as r:
        r.raise_for_status()
        total = int(r.headers.get("content-length", 0))
        with open(destino, "wb") as f, tqdm(
            total=total, unit="B", unit_scale=True, unit_divisor=1024,
            desc=destino, colour="cyan",
        ) as barra:
            for bloque in r.iter_content(chunk_size=8192):
                f.write(bloque)
                barra.update(len(bloque))
```

### 9.2 Copiar un archivo grande

```python
from pathlib import Path


def copiar_con_progreso(origen: Path, destino: Path, bloque: int = 1024 * 1024) -> None:
    tamano = origen.stat().st_size
    with origen.open("rb") as fo, destino.open("wb") as fd, \
         tqdm(total=tamano, unit="B", unit_scale=True, unit_divisor=1024, desc=origen.name) as barra:
        while datos := fo.read(bloque):
            fd.write(datos)
            barra.update(len(datos))
```

### 9.3 Envolver un archivo (lectura con progreso)

```python
from tqdm.utils import CallbackIOWrapper

tamano = Path("datos.bin").stat().st_size
with open("datos.bin", "rb") as f, tqdm(total=tamano, unit="B", unit_scale=True) as barra:
    lector = CallbackIOWrapper(barra.update, f, "read")
    contenido = lector.read()
```

---

## 10. Concurrencia

### 10.1 Hilos y procesos con `tqdm.contrib.concurrent`

```python
from tqdm.contrib.concurrent import thread_map, process_map

# E/S (descargas, peticiones HTTP): hilos
resultados = thread_map(descargar_pagina, urls, max_workers=8, desc="Descargando")

# Cálculo intensivo: procesos
resultados = process_map(calcular, datos, max_workers=4, chunksize=100, desc="Calculando")
```

> `process_map` debe ejecutarse dentro de `if __name__ == "__main__":` en Windows y macOS.

### 10.2 Con `concurrent.futures` (orden de finalización)

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor(max_workers=8) as ex:
    futuros = {ex.submit(consultar_api, id_): id_ for id_ in ids}
    resultados = {}
    for futuro in tqdm(as_completed(futuros), total=len(futuros), desc="Consultando API"):
        id_ = futuros[futuro]
        try:
            resultados[id_] = futuro.result()
        except Exception as e:
            tqdm.write(f"Error en {id_}: {e}")
```

### 10.3 Código asíncrono

```python
import asyncio
from tqdm.asyncio import tqdm as atqdm


async def main():
    tareas = [consultar(i) for i in range(100)]
    resultados = await atqdm.gather(*tareas, desc="Peticiones")

    async for elemento in atqdm(generador_asincrono(), desc="Leyendo"):
        ...

asyncio.run(main())
```

---

## 11. Entornos especiales

### 11.1 Jupyter Notebook / JupyterLab

```python
from tqdm.notebook import tqdm         # Barra gráfica (widget)
from tqdm.auto import tqdm             # Recomendado: detecta el entorno automáticamente
```

> Si la barra no aparece en Jupyter, instale `ipywidgets`: `pip install ipywidgets`.

### 11.2 Integración con `logging`

```python
import logging
from tqdm.contrib.logging import logging_redirect_tqdm

log = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO)

with logging_redirect_tqdm():
    for i in tqdm(range(100)):
        if i % 25 == 0:
            log.info("Punto de control %d", i)     # No rompe la barra
```

### 11.3 Desactivar la barra (CI, logs, modo silencioso)

```python
import sys

silencioso = not sys.stderr.isatty()               # No es una terminal interactiva
for x in tqdm(datos, disable=silencioso):
    ...
```

O mediante variable de entorno (tqdm ≥ 4.66):

```bash
TQDM_DISABLE=1 python script.py
```

### 11.4 Integración con Rich

```python
from tqdm.rich import tqdm      # Barra dibujada con Rich (experimental)
```

### 11.5 Línea de comandos

tqdm también funciona en *pipes* de la terminal:

```bash
seq 1000000 | python -m tqdm --total 1000000 | wc -l
tar -zcf - carpeta/ | python -m tqdm --bytes --total $(du -sb carpeta/ | cut -f1) > copia.tgz
```

---

## 12. Ejemplo práctico completo

Procesador por lotes de archivos CSV: lectura con barras anidadas, validación, registro de avisos sin romper las barras, consolidación y resumen final.

```python
import csv
import logging
import random
import time
from pathlib import Path

from tqdm import tqdm
from tqdm.contrib.logging import logging_redirect_tqdm

logging.basicConfig(level=logging.INFO, format="%(levelname)s | %(message)s")
log = logging.getLogger("lotes")

ENTRADA = Path("lotes")
SALIDA = Path("consolidado.csv")


def generar_demo(n_archivos: int = 6, filas: int = 300) -> None:
    rng = random.Random(7)
    ENTRADA.mkdir(exist_ok=True)
    for i in range(1, n_archivos + 1):
        with (ENTRADA / f"ventas_{i:02d}.csv").open("w", newline="", encoding="utf-8") as f:
            w = csv.writer(f)
            w.writerow(["id", "cliente", "importe"])
            for j in range(rng.randint(filas // 2, filas)):
                importe = rng.choice([round(rng.uniform(5, 500), 2), "", "N/A"])
                w.writerow([f"{i}-{j}", f"Cliente {rng.randint(1, 80)}", importe])


def contar_filas(ruta: Path) -> int:
    with ruta.open(encoding="utf-8") as f:
        return sum(1 for _ in f) - 1


def procesar() -> dict:
    archivos = sorted(ENTRADA.glob("*.csv"))
    estadisticas = {"archivos": 0, "validas": 0, "descartadas": 0, "importe": 0.0}

    with logging_redirect_tqdm(), SALIDA.open("w", newline="", encoding="utf-8") as salida:
        escritor = csv.writer(salida)
        escritor.writerow(["id", "cliente", "importe", "origen"])

        barra_archivos = tqdm(archivos, desc="Archivos", unit="arch", position=0, colour="green")
        for archivo in barra_archivos:
            total = contar_filas(archivo)
            descartadas_archivo = 0

            with archivo.open(encoding="utf-8") as f:
                filas = tqdm(csv.DictReader(f), total=total, desc=f"  {archivo.name}",
                             unit="fila", position=1, leave=False, colour="cyan")
                for fila in filas:
                    time.sleep(0.001)                               # Simula trabajo
                    try:
                        importe = float(fila["importe"])
                    except ValueError:
                        descartadas_archivo += 1
                        continue
                    escritor.writerow([fila["id"], fila["cliente"], f"{importe:.2f}", archivo.name])
                    estadisticas["validas"] += 1
                    estadisticas["importe"] += importe
                    filas.set_postfix(descartadas=descartadas_archivo)

            estadisticas["archivos"] += 1
            estadisticas["descartadas"] += descartadas_archivo
            if descartadas_archivo > total * 0.3:
                log.warning("%s: %d de %d filas descartadas (%.0f %%)",
                            archivo.name, descartadas_archivo, total, 100 * descartadas_archivo / total)
            barra_archivos.set_postfix(validas=estadisticas["validas"],
                                       importe=f"{estadisticas['importe']:,.0f}€")

    return estadisticas


def main() -> None:
    if not ENTRADA.exists():
        generar_demo()

    inicio = time.perf_counter()
    e = procesar()
    duracion = time.perf_counter() - inicio

    total = e["validas"] + e["descartadas"]
    print("\n" + "=" * 50)
    print(f" Archivos procesados : {e['archivos']}")
    print(f" Filas válidas       : {e['validas']:,}".replace(",", "."))
    print(f" Filas descartadas   : {e['descartadas']:,} ({e['descartadas'] / total:.1%})".replace(",", "."))
    print(f" Importe total       : {e['importe']:,.2f} €".replace(",", "X").replace(".", ",").replace("X", "."))
    print(f" Tiempo              : {duracion:.1f} s")
    print(f" Resultado           : {SALIDA}")
    print("=" * 50)


if __name__ == "__main__":
    main()
```

---

## 13. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| La barra no muestra porcentaje ni tiempo restante | El iterable no tiene longitud (generador, `zip`). | Indique `total=`. |
| Se imprimen muchas barras en líneas nuevas | Uso de `print()` dentro del bucle. | Use `tqdm.write()` o `logging_redirect_tqdm()`. |
| Barras anidadas desordenadas | Falta `position` o `leave=False`. | Asigne `position` a cada nivel y `leave=False` a las internas. |
| Barras duplicadas en Jupyter | Uso de `tqdm` de consola en notebook. | Use `from tqdm.auto import tqdm`. |
| La barra no se cierra al producirse un error | Barra creada manualmente sin cerrar. | Use `with tqdm(...) as barra:`. |
| `AttributeError: 'Series' object has no attribute 'progress_apply'` | No se registró la integración. | Llame a `tqdm.pandas()` antes. |
| Barra que «salta» en archivos de log o CI | Salida no interactiva. | Use `disable=True` o `TQDM_DISABLE=1`. |
| `enumerate(tqdm(...))` correcto pero `tqdm(enumerate(...))` sin total | `enumerate` oculta la longitud. | Coloque `tqdm` dentro de `enumerate` o indique `total`. |
| Caracteres extraños en la barra (Windows antiguo) | La consola no muestra bloques Unicode. | Use `ascii=True` o Windows Terminal. |

---

## 14. Buenas prácticas

- Añada barras de progreso a cualquier proceso que tarde más de unos segundos.
- Use descripciones claras (`desc`) y unidades significativas (`unit="archivo"`).
- Indique siempre `total` cuando el iterable no tenga longitud.
- Use `with` para barras manuales: garantiza su cierre aunque haya errores.
- Use `tqdm.write()` o `logging_redirect_tqdm()` para mensajes dentro del bucle.
- Muestre métricas útiles con `set_postfix` (errores, valores medios, último elemento).
- Use `tqdm.auto` para que el mismo código funcione en consola y Jupyter.
- Desactive las barras en entornos no interactivos (servidores, CI, tareas programadas).
- No cree una barra para cada iteración de un bucle muy interno: agrupe en lotes.

---

## 15. Referencia rápida

```python
from tqdm import tqdm, trange
from tqdm.auto import tqdm

for x in tqdm(iterable, desc="Texto", total=n, unit="it", colour="green", leave=True): ...
for i in trange(n): ...

with tqdm(total=n) as b:
    b.update(k); b.set_description("..."); b.set_postfix(clave=valor)

tqdm.write("mensaje")
tqdm(total=bytes, unit="B", unit_scale=True, unit_divisor=1024)
tqdm(..., position=1, leave=False)                # Barras anidadas
tqdm(..., bar_format="{desc}: {percentage:3.0f}%|{bar}| {n_fmt}/{total_fmt}")

tqdm.pandas(); df["c"].progress_apply(f); df.progress_apply(f, axis=1)
from tqdm.contrib.concurrent import thread_map, process_map
from tqdm.contrib.logging import logging_redirect_tqdm
from tqdm.asyncio import tqdm as atqdm; await atqdm.gather(*tareas)
```

---

## 16. Ejercicios propuestos

1. Añada una barra de progreso a un script que redimensione todas las imágenes de una carpeta, mostrando el nombre del archivo actual y el número de errores.
2. Descargue varios archivos en paralelo con `thread_map` y muestre una barra por descarga en bytes.
3. Procese un CSV de un millón de filas por bloques con Pandas mostrando el progreso en filas.
4. Cree barras anidadas para recorrer carpetas → archivos → líneas, borrando las barras internas al terminar.
5. Integre `logging` con tqdm en un proceso por lotes y compruebe que los avisos no rompen las barras.

---

## 17. Referencias

- Documentación oficial: https://tqdm.github.io/
- Repositorio: https://github.com/tqdm/tqdm
