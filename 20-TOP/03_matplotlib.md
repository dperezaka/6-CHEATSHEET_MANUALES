# Manual de uso: Matplotlib

> **Categoría:** Ciencia de datos · **Utilidad:** Gráficos estáticos de calidad para informes y publicaciones
> **Tipo:** Librería de terceros (`pip install matplotlib`) · **Versión de referencia:** 3.8 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave: `Figure` y `Axes`](#3-conceptos-clave-figure-y-axes)
4. [Las dos interfaces: `pyplot` y orientada a objetos](#4-las-dos-interfaces-pyplot-y-orientada-a-objetos)
5. [Tipos de gráfico](#5-tipos-de-gráfico)
6. [Personalizar ejes, títulos y leyendas](#6-personalizar-ejes-títulos-y-leyendas)
7. [Colores, estilos de línea y marcadores](#7-colores-estilos-de-línea-y-marcadores)
8. [Varios gráficos en una figura](#8-varios-gráficos-en-una-figura)
9. [Texto, anotaciones y líneas de referencia](#9-texto-anotaciones-y-líneas-de-referencia)
10. [Formato de los ejes: números, porcentajes y fechas](#10-formato-de-los-ejes-números-porcentajes-y-fechas)
11. [Mapas de color y datos 2D](#11-mapas-de-color-y-datos-2d)
12. [Estilos y configuración global (`rcParams`)](#12-estilos-y-configuración-global-rcparams)
13. [Guardar figuras](#13-guardar-figuras)
14. [Gráficos 3D](#14-gráficos-3d)
15. [Animaciones](#15-animaciones)
16. [Integración con pandas, Jupyter y aplicaciones](#16-integración-con-pandas-jupyter-y-aplicaciones)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**Matplotlib** es la librería de visualización más extendida de Python. Permite crear gráficos estáticos (y, en menor medida, animados e interactivos) con un control total de cada elemento: ejes, escalas, colores, tipografía, anotaciones. Es la base sobre la que se construyen **Seaborn** y los gráficos de **pandas**, y exporta a PNG, SVG, PDF y otros formatos con calidad de publicación.

**Ventajas:**
- Control absoluto de cada detalle del gráfico.
- Enorme variedad de tipos de gráfico.
- Exportación vectorial (PDF, SVG) y de mapa de bits (PNG) en alta resolución.
- Integración con NumPy, pandas, Jupyter, Tkinter, PySide6 y Streamlit.
- Madura, estable y ampliamente documentada.

**Limitaciones:**
- La API es extensa y algo verbosa para gráficos estadísticos complejos.
- Interactividad limitada en comparación con Plotly.
- Estilo por defecto correcto pero sobrio: conviene personalizarlo.

| Si necesita… | Opción |
|---|---|
| Gráficos estáticos con control total para informes | **Matplotlib** |
| Gráficos estadísticos rápidos y atractivos a partir de DataFrames | Seaborn |
| Gráficos interactivos para web o paneles | Plotly |
| Paneles web de datos | Streamlit (+ cualquiera de las anteriores) |

---

## 2. Instalación e importación

```bash
pip install matplotlib
```

```python
import matplotlib.pyplot as plt
import matplotlib as mpl
import numpy as np
```

En scripts que solo guardan imágenes (servidores, tareas programadas), use un *backend* sin ventana:

```python
import matplotlib
matplotlib.use("Agg")            # Antes de importar pyplot
import matplotlib.pyplot as plt
```

---

## 3. Conceptos clave: `Figure` y `Axes`

```
┌──────────────────── Figure (lienzo completo) ────────────────────┐
│  suptitle                                                         │
│  ┌──────────── Axes (un gráfico) ────────────┐                    │
│  │ title                                     │                    │
│  │ y ▲          ── línea (Line2D)            │  legend            │
│  │ l │      ●  marcadores                    │                    │
│  │ a │  ▇▇ barras (Rectangle)                │                    │
│  │ b └────────────────────────► xlabel       │                    │
│  │      ticks / ticklabels                   │                    │
│  └───────────────────────────────────────────┘                    │
└───────────────────────────────────────────────────────────────────┘
```

| Elemento | Descripción |
|---|---|
| `Figure` | Lienzo que contiene uno o varios gráficos. |
| `Axes` | Un gráfico concreto con sus ejes X/Y, título, leyenda… |
| `Axis` | Cada eje (`ax.xaxis`, `ax.yaxis`) con sus marcas (*ticks*). |
| *Artist* | Todo lo que se dibuja (líneas, textos, rectángulos…). |
| *Backend* | Motor de dibujo: ventana (TkAgg, QtAgg) o archivo (Agg, PDF, SVG). |

---

## 4. Las dos interfaces: `pyplot` y orientada a objetos

### 4.1 Interfaz `pyplot` (rápida, para exploración)

```python
x = np.linspace(0, 10, 100)
plt.plot(x, np.sin(x), label="seno")
plt.plot(x, np.cos(x), label="coseno")
plt.title("Funciones trigonométricas")
plt.xlabel("x"); plt.ylabel("y")
plt.legend()
plt.show()
```

### 4.2 Interfaz orientada a objetos (recomendada)

```python
fig, ax = plt.subplots(figsize=(8, 4.5), layout="constrained")
ax.plot(x, np.sin(x), label="seno")
ax.plot(x, np.cos(x), label="coseno", linestyle="--")
ax.set(title="Funciones trigonométricas", xlabel="x", ylabel="y", xlim=(0, 10))
ax.legend()
fig.savefig("trigonometria.png", dpi=150)
plt.show()
```

> Use la interfaz orientada a objetos (`fig, ax = plt.subplots()`) en scripts, funciones y aplicaciones: es explícita y evita confusiones sobre «qué gráfico está activo».

| `pyplot` | Objeto |
|---|---|
| `plt.title()` | `ax.set_title()` |
| `plt.xlabel()` / `plt.ylabel()` | `ax.set_xlabel()` / `ax.set_ylabel()` |
| `plt.xlim()` / `plt.ylim()` | `ax.set_xlim()` / `ax.set_ylim()` |
| `plt.xticks()` | `ax.set_xticks()` / `ax.set_xticklabels()` |
| `plt.legend()` | `ax.legend()` |
| `plt.savefig()` | `fig.savefig()` |

---

## 5. Tipos de gráfico

```python
fig, ax = plt.subplots()

ax.plot(x, y, marker="o")                                   # Líneas
ax.step(x, y, where="mid")                                  # Escalones
ax.scatter(x, y, s=tamanos, c=valores, cmap="viridis", alpha=0.7)   # Dispersión
ax.bar(categorias, valores, color="#1F77B4")                # Barras verticales
ax.barh(categorias, valores)                                # Barras horizontales
ax.bar(x, a, label="A"); ax.bar(x, b, bottom=a, label="B")  # Apiladas
ax.hist(datos, bins=30, density=True, alpha=0.6)            # Histograma
ax.boxplot([g1, g2, g3], tick_labels=["A", "B", "C"])       # Cajas y bigotes
ax.violinplot([g1, g2, g3])                                 # Violín
ax.pie(valores, labels=etiquetas, autopct="%1.1f%%", startangle=90)   # Tarta
ax.fill_between(x, y_min, y_max, alpha=0.2)                 # Área / banda de confianza
ax.stackplot(x, serie1, serie2, serie3, labels=["a", "b", "c"])        # Áreas apiladas
ax.errorbar(x, y, yerr=error, fmt="o", capsize=4)           # Barras de error
ax.imshow(matriz, cmap="magma")                             # Imagen / mapa de calor
ax.contour(X, Y, Z, levels=10) ; ax.contourf(X, Y, Z)       # Curvas de nivel
ax.hexbin(x, y, gridsize=30)                                # Densidad en hexágonos
ax.stem(x, y) ; ax.eventplot(eventos) ; ax.ecdf(datos)      # Otros
```

### 5.1 Barras agrupadas con etiquetas de valor

```python
categorias = ["Norte", "Sur", "Este", "Oeste"]
v2025 = [120, 95, 140, 80]
v2026 = [135, 90, 160, 101]
x = np.arange(len(categorias))
ancho = 0.38

fig, ax = plt.subplots(layout="constrained")
b1 = ax.bar(x - ancho / 2, v2025, ancho, label="2025", color="#9DB4C0")
b2 = ax.bar(x + ancho / 2, v2026, ancho, label="2026", color="#1F4E78")
ax.bar_label(b1, padding=2, fontsize=8)
ax.bar_label(b2, padding=2, fontsize=8, fmt="%.0f k€")
ax.set_xticks(x, categorias)
ax.set_ylabel("Ventas (k€)")
ax.legend(frameon=False)
ax.spines[["top", "right"]].set_visible(False)
```

### 5.2 Gráfico de anillo (*donut*)

```python
fig, ax = plt.subplots()
cuñas, *_ = ax.pie(valores, labels=etiquetas, autopct="%1.0f%%", pctdistance=0.8,
                   wedgeprops={"width": 0.4, "edgecolor": "white"}, startangle=90)
ax.text(0, 0, f"{sum(valores)}\ntotal", ha="center", va="center", fontsize=14, weight="bold")
ax.set_aspect("equal")
```

---

## 6. Personalizar ejes, títulos y leyendas

```python
ax.set_title("Ventas mensuales", fontsize=14, fontweight="bold", loc="left", pad=12)
fig.suptitle("Informe 2026", fontsize=16)
ax.set_xlabel("Mes") ; ax.set_ylabel("Importe (€)")
ax.set_xlim(0, 12) ; ax.set_ylim(bottom=0)
ax.set_xscale("log") ; ax.set_yscale("symlog")
ax.set_xticks([1, 4, 7, 10], ["Ene", "Abr", "Jul", "Oct"])
ax.tick_params(axis="x", rotation=45, labelsize=9)
ax.invert_yaxis()                                  # Útil en barras horizontales ordenadas
ax.grid(True, axis="y", linestyle=":", alpha=0.6)
ax.set_axisbelow(True)                             # Rejilla detrás de las barras
ax.spines[["top", "right"]].set_visible(False)     # Quitar bordes
ax.margins(x=0.02)
ax.set_aspect("equal")

ax.legend(loc="upper left", ncols=2, frameon=False, title="Región")
ax.legend(loc="center left", bbox_to_anchor=(1.01, 0.5))     # Fuera del gráfico, a la derecha
fig.legend(loc="outside lower center", ncols=3)              # Leyenda común (layout="constrained")

ax2 = ax.twinx()                                   # Segundo eje Y (p. ej., unidades frente a importe)
ax2.plot(x, unidades, color="tab:orange")
ax2.set_ylabel("Unidades", color="tab:orange")
```

---

## 7. Colores, estilos de línea y marcadores

```python
ax.plot(x, y, color="#1F4E78", linewidth=2, linestyle="--", marker="o", markersize=5,
        markerfacecolor="white", markeredgecolor="#1F4E78", alpha=0.9, label="Serie A")
ax.plot(x, y, "r--o")                              # Formato abreviado: color, línea y marcador
```

| Propiedad | Valores |
|---|---|
| `color` | Nombre (`"red"`, `"tab:blue"`), hexadecimal (`"#1F4E78"`), RGB (`(0.1, 0.3, 0.5)`), ciclo (`"C0"`…`"C9"`). |
| `linestyle` | `"-"`, `"--"`, `"-."`, `":"`, `"None"`. |
| `marker` | `"o"`, `"s"`, `"^"`, `"D"`, `"x"`, `"+"`, `"*"`, `"."`. |
| `linewidth`, `markersize`, `alpha`, `zorder` | Grosor, tamaño, transparencia, orden de dibujo. |

Paletas de colores:

```python
colores = plt.colormaps["tab10"].colors                    # Cualitativa (categorías)
colores = plt.colormaps["viridis"](np.linspace(0, 1, 5))   # 5 colores de una secuencial
```

> Para categorías, use paletas cualitativas (`tab10`, `Set2`); para magnitudes, secuenciales (`viridis`, `Blues`); para valores alrededor de un centro, divergentes (`RdBu`, `coolwarm`). Evite `jet`: distorsiona la percepción.

---

## 8. Varios gráficos en una figura

### 8.1 Rejilla regular

```python
fig, axs = plt.subplots(2, 3, figsize=(12, 7), layout="constrained", sharex=True)
axs[0, 0].plot(x, y)
axs[1, 2].bar(cats, vals)
for ax in axs.flat:
    ax.grid(alpha=0.3)
```

### 8.2 Diseños irregulares con `subplot_mosaic`

```python
fig, axs = plt.subplot_mosaic(
    [["evolucion", "evolucion", "kpi"],
     ["barras",    "tarta",     "kpi"]],
    figsize=(12, 7), layout="constrained",
    width_ratios=[1, 1, 0.6],
)
axs["evolucion"].plot(meses, ventas)
axs["barras"].bar(cats, vals)
axs["tarta"].pie(vals)
axs["kpi"].axis("off")
```

### 8.3 `GridSpec` y ejes insertados

```python
fig = plt.figure(figsize=(10, 6), layout="constrained")
gs = fig.add_gridspec(2, 2, height_ratios=[2, 1])
ax_principal = fig.add_subplot(gs[0, :])
ax_a = fig.add_subplot(gs[1, 0]); ax_b = fig.add_subplot(gs[1, 1])

detalle = ax_principal.inset_axes([0.6, 0.55, 0.35, 0.4])     # Gráfico dentro de otro
```

> Use `layout="constrained"` (o `fig.tight_layout()`) para que títulos y etiquetas no se solapen.

---

## 9. Texto, anotaciones y líneas de referencia

```python
ax.text(0.02, 0.95, "Fuente: ERP interno", transform=ax.transAxes, fontsize=8, color="gray", va="top")
ax.annotate("Máximo histórico", xy=(x_max, y_max), xytext=(x_max - 3, y_max * 1.1),
            arrowprops={"arrowstyle": "->", "color": "black"}, fontsize=9)
ax.axhline(objetivo, color="red", linestyle="--", linewidth=1, label="Objetivo")
ax.axvline(fecha_lanzamiento, color="gray", alpha=0.5)
ax.axvspan(inicio_campana, fin_campana, color="orange", alpha=0.1)   # Zona sombreada
ax.axhspan(0, 50, color="red", alpha=0.05)
```

Fórmulas con notación LaTeX (sin necesidad de tener LaTeX instalado):

```python
ax.set_title(r"$y = \alpha \cdot e^{-\lambda t}$")
```

> `transform=ax.transAxes` usa coordenadas relativas al gráfico (0–1), útil para textos fijos independientes de los datos.

---

## 10. Formato de los ejes: números, porcentajes y fechas

```python
from matplotlib.ticker import FuncFormatter, PercentFormatter, MaxNLocator, StrMethodFormatter
import matplotlib.dates as mdates

ax.yaxis.set_major_formatter(StrMethodFormatter("{x:,.0f} €"))
ax.yaxis.set_major_formatter(FuncFormatter(lambda v, _: f"{v / 1000:.0f} k€"))
ax.yaxis.set_major_formatter(PercentFormatter(xmax=1))            # 0.25 → 25 %
ax.xaxis.set_major_locator(MaxNLocator(integer=True))             # Solo enteros

# Fechas
ax.plot(fechas, valores)                                           # fechas: datetime o datetime64
localizador = mdates.AutoDateLocator()
ax.xaxis.set_major_locator(localizador)
ax.xaxis.set_major_formatter(mdates.ConciseDateFormatter(localizador))   # Etiquetas compactas
ax.xaxis.set_major_locator(mdates.MonthLocator(interval=2))
ax.xaxis.set_major_formatter(mdates.DateFormatter("%b %Y"))
```

Separador decimal español:

```python
mpl.rcParams["axes.formatter.use_locale"] = True
import locale; locale.setlocale(locale.LC_NUMERIC, "es_ES.UTF-8")   # Si la configuración regional está instalada
# Alternativa portable:
ax.yaxis.set_major_formatter(FuncFormatter(lambda v, _: f"{v:,.0f}".replace(",", ".")))
```

---

## 11. Mapas de color y datos 2D

```python
matriz = np.random.default_rng(0).random((6, 8))
fig, ax = plt.subplots()
im = ax.imshow(matriz, cmap="YlGnBu", vmin=0, vmax=1, aspect="auto")
fig.colorbar(im, ax=ax, label="Intensidad", shrink=0.8)
ax.set_xticks(range(8), [f"Col {i}" for i in range(8)], rotation=45)
for (i, j), valor in np.ndenumerate(matriz):                       # Valores dentro de las celdas
    ax.text(j, i, f"{valor:.2f}", ha="center", va="center", fontsize=7,
            color="white" if valor > 0.6 else "black")

# Superficies de funciones
X, Y = np.meshgrid(np.linspace(-3, 3, 200), np.linspace(-3, 3, 200))
Z = np.exp(-(X**2 + Y**2)) * np.cos(2 * X)
cs = ax.contourf(X, Y, Z, levels=20, cmap="RdBu_r")
ax.contour(X, Y, Z, levels=cs.levels[::4], colors="k", linewidths=0.5)

from matplotlib.colors import LogNorm, TwoSlopeNorm
ax.imshow(datos, norm=LogNorm())                       # Escala logarítmica
ax.imshow(anomalias, cmap="RdBu_r", norm=TwoSlopeNorm(vcenter=0))   # Centrado en 0
```

---

## 12. Estilos y configuración global (`rcParams`)

```python
print(plt.style.available)
plt.style.use("seaborn-v0_8-whitegrid")      # También: "ggplot", "bmh", "fivethirtyeight", "dark_background"

with plt.style.context("dark_background"):  # Solo dentro del bloque
    fig, ax = plt.subplots()

mpl.rcParams.update({
    "figure.figsize": (9, 5),
    "figure.dpi": 110,
    "font.family": "DejaVu Sans",
    "font.size": 10,
    "axes.titlesize": 13,
    "axes.titleweight": "bold",
    "axes.spines.top": False,
    "axes.spines.right": False,
    "axes.grid": True,
    "grid.alpha": 0.3,
    "axes.prop_cycle": mpl.cycler(color=["#1F4E78", "#E07B00", "#2CA02C", "#C00000", "#7F7F7F"]),
    "savefig.bbox": "tight",
})
```

Hoja de estilo propia reutilizable (`corporativo.mplstyle`):

```
axes.prop_cycle: cycler('color', ['1F4E78', 'E07B00', '2CA02C', 'C00000'])
axes.spines.top: False
axes.spines.right: False
font.size: 10
```

```python
plt.style.use("corporativo.mplstyle")
```

---

## 13. Guardar figuras

```python
fig.savefig("grafico.png", dpi=200, bbox_inches="tight")              # Mapa de bits
fig.savefig("grafico.svg")                                             # Vectorial (web)
fig.savefig("grafico.pdf")                                             # Vectorial (impresión)
fig.savefig("grafico.png", transparent=True, facecolor="white")
plt.close(fig)                                                          # Liberar memoria

from matplotlib.backends.backend_pdf import PdfPages
with PdfPages("informe.pdf") as pdf:                                   # PDF de varias páginas
    for figura in figuras:
        pdf.savefig(figura)
    pdf.infodict()["Title"] = "Informe mensual"

import io
buffer = io.BytesIO(); fig.savefig(buffer, format="png"); buffer.seek(0)   # En memoria (web, correo, Excel)
```

> En bucles que generan muchas figuras, cierre cada una con `plt.close(fig)`; si no, la memoria crece y aparece el aviso «More than 20 figures have been opened».

---

## 14. Gráficos 3D

```python
fig = plt.figure(figsize=(8, 6))
ax = fig.add_subplot(projection="3d")
ax.plot_surface(X, Y, Z, cmap="viridis", edgecolor="none")
ax.scatter(xs, ys, zs, c=zs)
ax.set(xlabel="X", ylabel="Y", zlabel="Z")
ax.view_init(elev=30, azim=45)
```

---

## 15. Animaciones

```python
from matplotlib.animation import FuncAnimation

fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 200)
linea, = ax.plot(x, np.sin(x))

def actualizar(fotograma):
    linea.set_ydata(np.sin(x + fotograma / 10))
    return linea,

anim = FuncAnimation(fig, actualizar, frames=100, interval=40, blit=True)
anim.save("onda.gif", writer="pillow", fps=25)     # MP4: writer="ffmpeg" (requiere ffmpeg)
```

---

## 16. Integración con pandas, Jupyter y aplicaciones

```python
ax = df.plot(x="fecha", y=["ventas", "objetivo"], figsize=(10, 4))   # pandas devuelve un Axes
ax.set_ylabel("€")
df.groupby("region")["importe"].sum().sort_values().plot.barh(ax=axs[1])
```

Jupyter: `%matplotlib inline` (estático, por defecto) o `%matplotlib widget` (interactivo, con `ipympl`).

Tkinter / PySide6: incruste la figura con `FigureCanvasTkAgg` o `FigureCanvasQTAgg` (use `matplotlib.figure.Figure` en lugar de `pyplot`). Streamlit: `st.pyplot(fig)`.

---

## 17. Ejemplo práctico completo

**Panel de ventas en una sola figura y un informe PDF de varias páginas**: genera datos simulados de ventas diarias, construye un panel con `subplot_mosaic` (evolución con media móvil, objetivo y anotación del máximo; barras comparativas con etiquetas; anillo de cuota por canal; histograma de importes con la mediana; mapa de calor día de la semana × mes; dispersión precio–unidades con recta de ajuste; tarjeta de indicadores), aplica un estilo corporativo propio y exporta PNG, SVG y un PDF de varias páginas.

```python
"""
panel_ventas.py — Panel de ventas e informe PDF con Matplotlib.
Requiere: pip install matplotlib numpy
Uso: python panel_ventas.py [--mostrar]
"""
import argparse
from datetime import date, timedelta
from pathlib import Path

import matplotlib

matplotlib.use("Agg")
import matplotlib.dates as mdates
import matplotlib.pyplot as plt
import numpy as np
from matplotlib.backends.backend_pdf import PdfPages
from matplotlib.ticker import FuncFormatter, PercentFormatter

AZUL, NARANJA, VERDE, ROJO, GRIS = "#1F4E78", "#E07B00", "#2CA02C", "#C00000", "#8A8F98"
MESES = ["Ene", "Feb", "Mar", "Abr", "May", "Jun", "Jul", "Ago", "Sep", "Oct", "Nov", "Dic"]
DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]


def estilo_corporativo() -> None:
    plt.rcParams.update({
        "figure.dpi": 110, "font.size": 9, "axes.titlesize": 11, "axes.titleweight": "bold",
        "axes.titlelocation": "left", "axes.spines.top": False, "axes.spines.right": False,
        "axes.grid": True, "grid.alpha": 0.25, "grid.linestyle": ":", "axes.axisbelow": True,
        "axes.prop_cycle": plt.cycler(color=[AZUL, NARANJA, VERDE, ROJO, GRIS]),
        "legend.frameon": False, "savefig.bbox": "tight",
    })


def euros(v: float, _=None) -> str:
    if abs(v) >= 1_000_000:
        return f"{v / 1_000_000:.1f} M€".replace(".", ",")
    if abs(v) >= 1_000:
        return f"{v / 1_000:.0f} k€"
    return f"{v:.0f} €"


# ------------------------------------------------------------------ Datos simulados
def generar_datos(semilla: int = 3) -> dict:
    rng = np.random.default_rng(semilla)
    dias = np.arange(np.datetime64("2026-01-01"), np.datetime64("2027-01-01"))
    t = np.arange(dias.size)
    dia_semana = (dias.astype("datetime64[D]").view("int64") - 4) % 7          # 0 = lunes
    estacional = 1 + 0.25 * np.sin(2 * np.pi * (t - 60) / 365) + np.where(np.isin(t // 30, [10, 11]), 0.3, 0)
    semanal = np.where(dia_semana >= 5, 0.6, 1.0)
    ventas = (9000 + t * 8) * estacional * semanal * rng.lognormal(0, 0.12, dias.size)
    meses = dias.astype("datetime64[M]").astype(int) % 12
    canales = {"Web": 0.46, "Tienda": 0.34, "Teléfono": 0.12, "Marketplace": 0.08}
    precios = rng.uniform(10, 300, 400)
    unidades = np.clip(900 / precios ** 0.6 * rng.lognormal(0, 0.25, 400), 1, None)
    return {"dias": dias, "ventas": ventas, "dia_semana": dia_semana, "meses": meses,
            "canales": canales, "importes": rng.lognormal(4.2, 0.7, 5000),
            "precios": precios, "unidades": unidades,
            "regiones": {"Norte": (1.21, 1.35), "Sur": (0.98, 0.95), "Este": (1.40, 1.58), "Oeste": (0.82, 0.97),
                         "Centro": (1.66, 1.71)}}


# ------------------------------------------------------------------ Gráficos (uno por función)
def grafico_evolucion(ax, d) -> None:
    fechas = d["dias"].astype("datetime64[D]").astype(object)
    media = np.convolve(d["ventas"], np.ones(28) / 28, mode="same")
    ax.plot(fechas, d["ventas"], color=GRIS, linewidth=0.6, alpha=0.6, label="Diario")
    ax.plot(fechas[14:-14], media[14:-14], color=AZUL, linewidth=2.2, label="Media móvil 28 días")
    objetivo = 12_000
    ax.axhline(objetivo, color=ROJO, linestyle="--", linewidth=1, label="Objetivo diario")
    ax.fill_between(fechas[14:-14], media[14:-14], objetivo, where=media[14:-14] >= objetivo,
                    color=VERDE, alpha=0.15, interpolate=True, label="Por encima del objetivo")
    ax.axvspan(date(2026, 11, 20), date(2026, 12, 2), color=NARANJA, alpha=0.12)
    ax.text(date(2026, 11, 21), ax.get_ylim()[1] * 0.97, "Campaña", color=NARANJA, fontsize=8, va="top")
    i = int(np.argmax(d["ventas"]))
    ax.annotate(f"Máximo: {euros(d['ventas'][i])}\n{fechas[i]:%d/%m}", xy=(fechas[i], d["ventas"][i]),
                xytext=(-170, -25), textcoords="offset points", fontsize=8,
                arrowprops={"arrowstyle": "->", "color": "black", "lw": 0.8})
    localizador = mdates.MonthLocator()
    ax.xaxis.set_major_locator(localizador)
    ax.xaxis.set_major_formatter(FuncFormatter(lambda v, _: MESES[mdates.num2date(v).month - 1]))
    ax.yaxis.set_major_formatter(FuncFormatter(euros))
    ax.set_title("Evolución de las ventas diarias")
    ax.legend(loc="upper left", ncols=4, fontsize=8)


def grafico_regiones(ax, d) -> None:
    nombres = list(d["regiones"])
    v25, v26 = np.array(list(d["regiones"].values())).T
    orden = np.argsort(v26)
    y = np.arange(len(nombres))
    alto = 0.38
    ax.barh(y - alto / 2, v25[orden], alto, color="#B8C7D6", label="2025")
    barras = ax.barh(y + alto / 2, v26[orden], alto, color=AZUL, label="2026")
    variacion = (v26[orden] - v25[orden]) / v25[orden]
    textos = ax.bar_label(barras, labels=[f"{v:+.0%}" for v in variacion], padding=3, fontsize=8)
    for texto, v in zip(textos, variacion):                 # Un color por etiqueta
        texto.set_color(VERDE if v >= 0 else ROJO)
    ax.set_yticks(y, np.array(nombres)[orden])
    ax.xaxis.set_major_formatter(FuncFormatter(lambda v, _: f"{v:.1f} M€".replace(".", ",")))
    ax.set_xlim(0, v26.max() * 1.25)
    ax.grid(axis="y", visible=False)
    ax.set_title("Ventas por región")
    ax.legend(loc="lower right", fontsize=8)


def grafico_canales(ax, d) -> None:
    etiquetas, cuotas = zip(*d["canales"].items())
    cuñas, _, textos = ax.pie(cuotas, labels=etiquetas, autopct="%1.0f%%", pctdistance=0.78, startangle=90,
                              counterclock=False, wedgeprops={"width": 0.42, "edgecolor": "white"},
                              textprops={"fontsize": 8})
    for t in textos:
        t.set_color("white")
        t.set_fontweight("bold")
    ax.text(0, 0, "Canal", ha="center", va="center", fontsize=10, fontweight="bold")
    ax.set_title("Cuota por canal")
    ax.set_aspect("equal")


def grafico_histograma(ax, d) -> None:
    importes = d["importes"]
    ax.hist(importes, bins=np.logspace(np.log10(importes.min()), np.log10(importes.max()), 40),
            color=AZUL, alpha=0.8, edgecolor="white", linewidth=0.4)
    ax.set_xscale("log")
    mediana = np.median(importes)
    ax.axvline(mediana, color=NARANJA, linewidth=1.5)
    ax.text(mediana * 1.1, ax.get_ylim()[1] * 0.95, f"Mediana {mediana:.0f} €", color=NARANJA, fontsize=8,
            va="top", bbox={"facecolor": "white", "edgecolor": "none", "alpha": 0.8, "pad": 1})
    ax.xaxis.set_major_formatter(FuncFormatter(lambda v, _: f"{v:,.0f} €".replace(",", ".")))
    ax.set_title("Importe por pedido")
    ax.set_ylabel("Pedidos")


def grafico_calor(ax, fig, d) -> None:
    matriz = np.zeros((7, 12))
    np.add.at(matriz, (d["dia_semana"], d["meses"]), d["ventas"])
    conteo = np.zeros((7, 12))
    np.add.at(conteo, (d["dia_semana"], d["meses"]), 1)
    media = matriz / conteo
    im = ax.imshow(media, cmap="YlGnBu", aspect="auto")
    ax.set_xticks(range(12), MESES, fontsize=7)
    ax.set_yticks(range(7), DIAS, fontsize=7)
    ax.grid(False)
    for (i, j), v in np.ndenumerate(media):
        ax.text(j, i, f"{v / 1000:.0f}", ha="center", va="center", fontsize=6,
                color="white" if v > np.percentile(media, 70) else "black")
    fig.colorbar(im, ax=ax, shrink=0.85, format=FuncFormatter(euros)).ax.tick_params(labelsize=7)
    ax.set_title("Venta media por día y mes (k€)")


def grafico_dispersion(ax, d) -> None:
    x, y = d["precios"], d["unidades"]
    puntos = ax.scatter(x, y, c=x * y, cmap="viridis", s=14, alpha=0.75, edgecolors="none")
    coef = np.polyfit(np.log(x), np.log(y), 1)                    # Ajuste potencial: log y = b·log x + a
    xs = np.linspace(x.min(), x.max(), 200)
    ax.plot(xs, np.exp(coef[1]) * xs ** coef[0], color=ROJO, linewidth=1.5,
            label=f"Elasticidad ≈ {coef[0]:.2f}")
    ax.set(xscale="log", yscale="log", xlabel="Precio (€)", ylabel="Unidades/mes")
    ax.set_title("Precio frente a unidades vendidas")
    ax.legend(fontsize=8)
    return puntos


def tarjeta_kpis(ax, d) -> None:
    ax.axis("off")
    total = d["ventas"].sum()
    mensual = np.bincount(d["meses"], weights=d["ventas"])
    kpis = [("Facturación anual", euros(total), AZUL),
            ("Mejor mes", f"{MESES[int(mensual.argmax())]} · {euros(mensual.max())}", VERDE),
            ("Peor mes", f"{MESES[int(mensual.argmin())]} · {euros(mensual.min())}", ROJO),
            ("Venta media diaria", euros(d["ventas"].mean()), NARANJA),
            ("Días sobre objetivo", f"{np.mean(d['ventas'] >= 12_000):.0%}", GRIS)]
    ax.set_title("Indicadores clave")
    for i, (titulo, valor, color) in enumerate(kpis):
        y = 0.88 - i * 0.19
        ax.add_patch(plt.Rectangle((0, y - 0.07), 0.03, 0.14, color=color, transform=ax.transAxes))
        ax.text(0.07, y + 0.035, titulo, transform=ax.transAxes, fontsize=8, color=GRIS, va="center")
        ax.text(0.07, y - 0.035, valor, transform=ax.transAxes, fontsize=12, fontweight="bold", va="center")


# ------------------------------------------------------------------ Composición y exportación
def panel(d) -> plt.Figure:
    fig, axs = plt.subplot_mosaic(
        [["evolucion", "evolucion", "evolucion", "kpis"],
         ["regiones", "canales", "histograma", "kpis"],
         ["calor", "calor", "dispersion", "dispersion"]],
        figsize=(16, 11), layout="constrained", width_ratios=[1, 1, 1, 0.75], height_ratios=[1.1, 1, 1],
    )
    fig.suptitle("Panel de ventas 2026", fontsize=18, fontweight="bold", x=0.01, ha="left")
    fig.text(0.99, 0.995, "Datos simulados · Elaborado con Matplotlib", ha="right", va="top", fontsize=8, color=GRIS)
    grafico_evolucion(axs["evolucion"], d)
    grafico_regiones(axs["regiones"], d)
    grafico_canales(axs["canales"], d)
    grafico_histograma(axs["histograma"], d)
    grafico_calor(axs["calor"], fig, d)
    puntos = grafico_dispersion(axs["dispersion"], d)
    fig.colorbar(puntos, ax=axs["dispersion"], label="Ingresos (precio × unidades)", shrink=0.85)
    tarjeta_kpis(axs["kpis"], d)
    return fig


def figura_individual(funcion, d, titulo: str) -> plt.Figure:
    fig, ax = plt.subplots(figsize=(11, 6), layout="constrained")
    if funcion is grafico_calor:
        funcion(ax, fig, d)
    else:
        funcion(ax, d)
    fig.suptitle(titulo, fontsize=14, fontweight="bold", x=0.01, ha="left")
    return fig


def main() -> int:
    p = argparse.ArgumentParser(description="Panel e informe de ventas con Matplotlib.")
    p.add_argument("--salida", type=Path, default=Path("salida_graficos"))
    p.add_argument("--mostrar", action="store_true", help="abrir el panel en una ventana")
    args = p.parse_args()
    if args.mostrar:
        plt.switch_backend("TkAgg")
    args.salida.mkdir(exist_ok=True)
    estilo_corporativo()
    datos = generar_datos()

    fig = panel(datos)
    fig.savefig(args.salida / "panel_ventas.png", dpi=150)
    fig.savefig(args.salida / "panel_ventas.svg")

    with PdfPages(args.salida / "informe_ventas.pdf") as pdf:
        pdf.savefig(fig)
        for funcion, titulo in [(grafico_evolucion, "1. Evolución diaria"), (grafico_regiones, "2. Regiones"),
                                (grafico_calor, "3. Estacionalidad"), (grafico_dispersion, "4. Precio y demanda")]:
            pagina = figura_individual(funcion, datos, titulo)
            pdf.savefig(pagina)
            plt.close(pagina)
        info = pdf.infodict()
        info["Title"], info["Author"] = "Informe de ventas 2026", "Departamento de Análisis"

    if args.mostrar:
        plt.show()
    plt.close("all")
    print(f"✔ Archivos generados en {args.salida}/: panel_ventas.png, panel_ventas.svg, informe_ventas.pdf")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| No se muestra ninguna ventana | Falta `plt.show()` o el *backend* es `Agg`. | Llame a `plt.show()`; en servidores, guarde con `savefig`. |
| La imagen guardada está en blanco | `savefig` después de `plt.show()` (la figura ya se cerró). | Guarde antes de mostrar. |
| Títulos y etiquetas solapados o cortados | Diseño sin ajustar. | `layout="constrained"` o `bbox_inches="tight"`. |
| `ValueError: x and y must have same first dimension` | Longitudes distintas. | Compruebe `len(x) == len(y)`. |
| Fechas ilegibles en el eje X | Demasiadas etiquetas. | `ConciseDateFormatter`, `MonthLocator`, rotación. |
| `RuntimeWarning: More than 20 figures have been opened` | Figuras sin cerrar en un bucle. | `plt.close(fig)`. |
| `UserWarning: Glyph … missing from font` | Fuente sin ese carácter (emojis, símbolos). | Cambie `font.family` o elimine el carácter. |
| El color de un mapa de calor no refleja bien los datos | Escala lineal con valores extremos. | `vmin`/`vmax`, `LogNorm`, `TwoSlopeNorm`. |
| `TclError: no display name` | Ventanas en un servidor sin entorno gráfico. | `matplotlib.use("Agg")`. |
| La leyenda tapa los datos | Posición automática insuficiente. | `loc=`, `bbox_to_anchor`, `fig.legend(loc="outside …")`. |
| `TypeError: 'labels' is deprecated` en `boxplot` | Renombrado a `tick_labels` (3.9+). | Use `tick_labels=`. |

---

## 19. Buenas prácticas

- Use la interfaz orientada a objetos (`fig, ax = plt.subplots()`) y funciones que reciban un `ax`.
- Elija el gráfico según el mensaje: evolución → líneas; comparación → barras; distribución → histograma o cajas; relación → dispersión; composición → barras apiladas (mejor que tartas con muchas porciones).
- Etiquete ejes con unidades, añada título y fuente de los datos.
- Elimine elementos innecesarios (bordes superior y derecho, rejillas densas) y resalte lo importante con color.
- Use paletas adecuadas y accesibles (`viridis`, `cividis`, `tab10`) y no dependa solo del color.
- Centralice el estilo con `rcParams` o una hoja `.mplstyle`.
- Guarde en formato vectorial (PDF/SVG) para impresión y PNG con `dpi` ≥ 150 para pantallas.
- Cierre las figuras en bucles y use el *backend* `Agg` en servidores.

---

## 20. Referencia rápida

```python
fig, ax = plt.subplots(figsize=(8, 5), layout="constrained") · fig, axs = plt.subplots(2, 2) · plt.subplot_mosaic([...])
ax.plot · scatter · bar · barh · hist · boxplot · violinplot · pie · fill_between · stackplot · errorbar · imshow · contourf
ax.set(title=, xlabel=, ylabel=, xlim=, ylim=, xscale=) · set_xticks(pos, etiquetas) · tick_params · grid · legend
ax.bar_label · annotate · text(transform=ax.transAxes) · axhline · axvline · axvspan · twinx · inset_axes
FuncFormatter · PercentFormatter · StrMethodFormatter · mdates.ConciseDateFormatter · MonthLocator
fig.colorbar(im, ax=ax) · plt.colormaps["viridis"] · LogNorm · TwoSlopeNorm
plt.style.use("…") · mpl.rcParams.update({...}) · fig.suptitle · fig.savefig("x.png", dpi=200) · PdfPages · plt.close(fig)
matplotlib.use("Agg") · FuncAnimation · add_subplot(projection="3d")
```

---

## 21. Ejercicios propuestos

1. Represente la temperatura máxima y mínima diaria de un año con una banda sombreada entre ambas y marque los días por encima de 35 °C.
2. Cree un gráfico de barras horizontales ordenado con los 10 productos más vendidos, con etiquetas de valor y el primero resaltado en otro color.
3. Construya una figura 2×2 con histograma, diagrama de cajas, gráfico de violín y ECDF de una misma variable.
4. Genere un PDF de varias páginas con un gráfico por región a partir de un DataFrame de pandas.
5. Anime la evolución de una cartera de inversión día a día y guárdela como GIF.

---

## 22. Referencias

- Documentación oficial: https://matplotlib.org/stable/
- Galería de ejemplos: https://matplotlib.org/stable/gallery/index.html
- Guía rápida (*cheatsheets*): https://matplotlib.org/cheatsheets/
- Elección de mapas de color: https://matplotlib.org/stable/users/explain/colors/colormaps.html
- Diseño con `subplot_mosaic`: https://matplotlib.org/stable/users/explain/axes/mosaic.html
