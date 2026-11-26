# Manual de uso: Matplotlib

> **Categoría:** Visualización de datos · **Utilidad:** Crear gráficos de todo tipo
> **Tipo:** Librería externa (requiere instalación con `pip`)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer gráfico](#4-primer-gráfico)
5. [Tipos de gráfico básicos](#5-tipos-de-gráfico-básicos)
6. [Personalización de ejes, títulos y leyendas](#6-personalización-de-ejes-títulos-y-leyendas)
7. [Colores, estilos y marcadores](#7-colores-estilos-y-marcadores)
8. [Varios gráficos en una figura](#8-varios-gráficos-en-una-figura)
9. [Gráficos avanzados](#9-gráficos-avanzados)
10. [Anotaciones y texto](#10-anotaciones-y-texto)
11. [Fechas y series temporales](#11-fechas-y-series-temporales)
12. [Estilos globales y `rcParams`](#12-estilos-globales-y-rcparams)
13. [Integración con pandas](#13-integración-con-pandas)
14. [Guardar y exportar](#14-guardar-y-exportar)
15. [Gráficos 3D y animaciones](#15-gráficos-3d-y-animaciones)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Matplotlib** es la librería de referencia para crear gráficos en Python. Produce gráficos estáticos de calidad de publicación (PNG, PDF, SVG) y es la base sobre la que se construyen otras librerías como **Seaborn** o la función `plot()` de **pandas**.

**Ventajas:**
- Control total sobre cada elemento del gráfico.
- Enorme variedad de tipos de gráfico: líneas, barras, dispersión, histogramas, mapas de calor, 3D…
- Exporta a formatos vectoriales (PDF, SVG) aptos para informes e impresión.
- Muy documentada y con una comunidad amplia.
- Se integra con pandas, NumPy, Jupyter y aplicaciones de escritorio (Tkinter, Qt).

**Limitaciones:**
- La sintaxis es extensa: un gráfico pulido requiere bastantes líneas.
- Los gráficos son estáticos (la interactividad es limitada fuera de Jupyter).
- El aspecto por defecto es correcto pero sobrio; conviene aplicar un estilo.

| Si necesita… | Opción |
|---|---|
| Gráficos estáticos para informes, control total | **Matplotlib** |
| Gráficos estadísticos con poco código | Seaborn |
| Gráficos interactivos (zoom, *tooltips*) | Plotly / Bokeh |
| Visualización declarativa basada en gramática | Altair |
| Paneles web interactivos | Dash / Streamlit |
| Mapas interactivos | Folium |

---

## 2. Instalación e importación

```bash
pip install matplotlib
```

Comprobar la versión instalada:

```bash
python -c "import matplotlib; print(matplotlib.__version__)"
```

Importaciones habituales:

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import matplotlib.dates as mdates
import numpy as np
```

> `pyplot` (alias `plt`) es la interfaz principal. Los submódulos `ticker` y `dates` se usan para dar formato a los ejes.

En **Jupyter**, los gráficos se muestran automáticamente. Para gráficos interactivos dentro del cuaderno:

```python
%matplotlib widget        # Requiere: pip install ipympl
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Figura (`Figure`) | El lienzo completo. Puede contener uno o varios gráficos. |
| Ejes (`Axes`) | Un gráfico individual dentro de la figura, con su sistema de coordenadas. |
| Eje (`Axis`) | Cada uno de los ejes X e Y, con sus marcas (*ticks*) y etiquetas. |
| Artista (`Artist`) | Cualquier elemento visible: líneas, textos, rectángulos, leyendas… |
| Interfaz `pyplot` | Funciones como `plt.plot()` que actúan sobre la figura «actual». Cómoda para pruebas rápidas. |
| Interfaz orientada a objetos | Se trabaja con objetos `fig` y `ax` de forma explícita. **Recomendada.** |
| *Backend* | Motor que dibuja la figura: en pantalla (TkAgg, QtAgg) o a archivo (Agg). |

**Anatomía de una figura:**

```
Figure ─┬─ Axes (gráfico 1) ─┬─ título, líneas, barras, leyenda…
        │                    ├─ XAxis (marcas, etiqueta)
        │                    └─ YAxis (marcas, etiqueta)
        └─ Axes (gráfico 2) ─── …
```

**Las dos interfaces:**

```python
# Interfaz pyplot (implícita): rápida, para pruebas
plt.plot([1, 2, 3], [4, 1, 6])
plt.title("Prueba")
plt.show()

# Interfaz orientada a objetos (explícita): recomendada
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 1, 6])
ax.set_title("Prueba")
plt.show()
```

> **Use siempre la interfaz orientada a objetos** (`fig, ax = plt.subplots()`) en scripts y aplicaciones: es más clara y evita errores al trabajar con varias figuras.

---

## 4. Primer gráfico

### 4.1 Gráfico mínimo

```python
import matplotlib.pyplot as plt

meses = ["Ene", "Feb", "Mar", "Abr", "May", "Jun"]
ventas = [120, 135, 128, 160, 172, 190]

fig, ax = plt.subplots(figsize=(8, 4))     # Ancho x alto en pulgadas
ax.plot(meses, ventas, marker="o")
ax.set_title("Ventas del primer semestre")
ax.set_xlabel("Mes")
ax.set_ylabel("Ventas (miles de €)")
plt.show()
```

### 4.2 Tamaño y resolución

```python
fig, ax = plt.subplots(figsize=(10, 5), dpi=100)   # 1000 x 500 píxeles
fig.set_size_inches(12, 6)                         # Cambiar después
```

### 4.3 Mostrar, guardar y cerrar

```python
plt.show()                          # Muestra la ventana (bloquea hasta cerrarla)
fig.savefig("grafico.png")          # Guarda en archivo
plt.close(fig)                      # Libera memoria (imprescindible en bucles)
```

---

## 5. Tipos de gráfico básicos

### 5.1 Líneas (`plot`)

```python
x = np.linspace(0, 10, 100)
ax.plot(x, np.sin(x), label="Seno")
ax.plot(x, np.cos(x), label="Coseno", linestyle="--")
ax.legend()
```

### 5.2 Barras verticales y horizontales (`bar`, `barh`)

```python
regiones = ["Norte", "Sur", "Este", "Oeste"]
importes = [120, 95, 140, 80]

ax.bar(regiones, importes, color="#1F4E78")
ax.barh(regiones, importes)                    # Horizontales
```

Barras agrupadas:

```python
x = np.arange(len(regiones))
ancho = 0.38
ax.bar(x - ancho / 2, [120, 95, 140, 80], ancho, label="2024")
ax.bar(x + ancho / 2, [135, 110, 150, 92], ancho, label="2025")
ax.set_xticks(x, regiones)
ax.legend()
```

Barras apiladas:

```python
ax.bar(regiones, [50, 40, 70, 30], label="Online")
ax.bar(regiones, [70, 55, 70, 50], bottom=[50, 40, 70, 30], label="Tienda")
```

Etiquetas de valor sobre las barras:

```python
barras = ax.bar(regiones, importes)
ax.bar_label(barras, fmt="%.0f", padding=3)
```

### 5.3 Dispersión (`scatter`)

```python
ax.scatter(precio, unidades, s=tamano, c=margen, cmap="viridis", alpha=0.7, edgecolors="white")
fig.colorbar(ax.collections[0], ax=ax, label="Margen (%)")
```

| Parámetro | Efecto |
|---|---|
| `s` | Tamaño de los puntos (número o lista). |
| `c` | Color (uno fijo o una lista de valores numéricos). |
| `cmap` | Mapa de colores cuando `c` es numérico. |
| `alpha` | Transparencia (0 a 1). |

### 5.4 Histograma (`hist`)

```python
datos = np.random.default_rng(42).normal(50, 10, 1000)
ax.hist(datos, bins=30, color="#2E75B6", edgecolor="white")
ax.hist(datos, bins=30, density=True, histtype="step")      # Normalizado, solo contorno
```

### 5.5 Sectores (`pie`) y anillo

```python
ax.pie([45, 30, 25], labels=["A", "B", "C"], autopct="%1.1f %%", startangle=90)
ax.pie([45, 30, 25], labels=["A", "B", "C"], wedgeprops={"width": 0.4})   # Anillo (donut)
ax.set_aspect("equal")
```

> Los gráficos de sectores son difíciles de comparar con más de 4–5 categorías: use barras en su lugar.

### 5.6 Caja (`boxplot`) y violín (`violinplot`)

```python
ax.boxplot([grupo_a, grupo_b, grupo_c], tick_labels=["A", "B", "C"])
ax.violinplot([grupo_a, grupo_b, grupo_c], showmedians=True)
```

### 5.7 Área (`fill_between`, `stackplot`)

```python
ax.fill_between(x, y_min, y_max, alpha=0.3)                 # Banda (p. ej., intervalo)
ax.stackplot(meses, online, tienda, labels=["Online", "Tienda"])
```

### 5.8 Escalón y tallo

```python
ax.step(x, y, where="mid")
ax.stem(x, y)
```

---

## 6. Personalización de ejes, títulos y leyendas

### 6.1 Títulos y etiquetas

```python
ax.set_title("Ventas mensuales", fontsize=14, fontweight="bold", loc="left")
ax.set_xlabel("Mes")
ax.set_ylabel("Importe (€)")
fig.suptitle("Informe anual 2025", fontsize=16)          # Título de toda la figura
```

Forma abreviada:

```python
ax.set(title="Ventas", xlabel="Mes", ylabel="Importe (€)", xlim=(0, 12), ylim=(0, 200))
```

### 6.2 Límites y escalas

```python
ax.set_xlim(0, 12)
ax.set_ylim(bottom=0)                 # Solo el límite inferior
ax.set_yscale("log")                  # Escala logarítmica
ax.invert_yaxis()                     # Invertir eje (útil en rankings)
ax.margins(x=0.02)                    # Margen interior
```

### 6.3 Marcas (*ticks*) y formato de números

```python
import matplotlib.ticker as mticker

ax.set_xticks([0, 3, 6, 9, 12])
ax.set_xticks(range(6), ["Ene", "Feb", "Mar", "Abr", "May", "Jun"])
ax.tick_params(axis="x", rotation=45, labelsize=9)

ax.yaxis.set_major_formatter(mticker.StrMethodFormatter("{x:,.0f} €"))
ax.yaxis.set_major_formatter(mticker.PercentFormatter(xmax=1))      # 0.25 → 25 %
ax.yaxis.set_major_locator(mticker.MultipleLocator(50))             # Marca cada 50
```

Formato con separadores españoles (punto de miles, coma decimal):

```python
def euros(valor, _):
    return f"{valor:,.0f} €".replace(",", ".")

ax.yaxis.set_major_formatter(mticker.FuncFormatter(euros))
```

### 6.4 Cuadrícula y bordes

```python
ax.grid(True, axis="y", linestyle=":", alpha=0.6)
ax.set_axisbelow(True)                              # Cuadrícula detrás de los datos
ax.spines[["top", "right"]].set_visible(False)      # Quitar bordes superior y derecho
```

### 6.5 Leyenda

```python
ax.plot(x, y1, label="2024")
ax.plot(x, y2, label="2025")
ax.legend(loc="upper left", frameon=False, ncols=2, title="Año")
ax.legend(loc="center left", bbox_to_anchor=(1, 0.5))      # Fuera del gráfico, a la derecha
```

| `loc` | Posición |
|---|---|
| `"best"` | Automática (por defecto) |
| `"upper right"`, `"upper left"` | Esquinas superiores |
| `"lower center"`, `"center"` | Centrada abajo / en el centro |

### 6.6 Doble eje Y

```python
fig, ax1 = plt.subplots()
ax1.bar(meses, ventas, color="#9DC3E6", label="Ventas")
ax2 = ax1.twinx()
ax2.plot(meses, margen, color="#C00000", marker="o", label="Margen %")
ax1.set_ylabel("Ventas (€)"); ax2.set_ylabel("Margen (%)")
```

---

## 7. Colores, estilos y marcadores

### 7.1 Especificar colores

```python
ax.plot(x, y, color="steelblue")         # Nombre
ax.plot(x, y, color="#1F4E78")           # Hexadecimal
ax.plot(x, y, color=(0.1, 0.3, 0.5))     # RGB (0–1)
ax.plot(x, y, color="C2")                # Tercer color del ciclo por defecto
```

### 7.2 Estilo de línea y marcadores

```python
ax.plot(x, y, color="#1F4E78", linewidth=2, linestyle="--", marker="o", markersize=6,
        markerfacecolor="white", markeredgewidth=1.5)
ax.plot(x, y, "r--o")                    # Forma abreviada: color, línea, marcador
```

| Línea | Significado | Marcador | Significado |
|---|---|---|---|
| `"-"` | Continua | `"o"` | Círculo |
| `"--"` | Discontinua | `"s"` | Cuadrado |
| `":"` | Punteada | `"^"` | Triángulo |
| `"-."` | Raya y punto | `"x"`, `"+"` | Aspa, cruz |

### 7.3 Mapas de color (*colormaps*)

| Tipo | Ejemplos | Uso |
|---|---|---|
| Secuencial | `viridis`, `Blues`, `Greens` | Valores de menor a mayor. |
| Divergente | `RdBu`, `coolwarm`, `RdYlGn` | Valores alrededor de un centro (p. ej., variación ±). |
| Cualitativo | `tab10`, `Set2`, `Paired` | Categorías sin orden. |

```python
colores = plt.get_cmap("tab10").colors          # Tupla de 10 colores
cmap = plt.get_cmap("Blues")
color = cmap(0.7)                               # Color al 70 % de la escala
```

> Prefiera mapas **perceptualmente uniformes** (`viridis`, `cividis`) y evite `jet`: distorsiona la lectura y no es apto para daltonismo.

---

## 8. Varios gráficos en una figura

### 8.1 Rejilla regular (`subplots`)

```python
fig, axs = plt.subplots(2, 2, figsize=(10, 7), sharex=True, layout="constrained")
axs[0, 0].plot(x, y1); axs[0, 0].set_title("Ventas")
axs[0, 1].bar(regiones, importes)
axs[1, 0].hist(datos, bins=20)
axs[1, 1].scatter(precio, unidades)

for ax in axs.flat:                       # Recorrer todos los ejes
    ax.grid(alpha=0.3)
```

> `layout="constrained"` ajusta automáticamente los márgenes para que no se solapen títulos y etiquetas. Alternativa clásica: `fig.tight_layout()`.

### 8.2 Disposición irregular (`subplot_mosaic`)

```python
fig, axs = plt.subplot_mosaic([["principal", "principal", "lateral"],
                               ["inferior1", "inferior2", "lateral"]],
                              figsize=(12, 7), layout="constrained")
axs["principal"].plot(x, y)
axs["lateral"].barh(regiones, importes)
```

### 8.3 `GridSpec` (proporciones personalizadas)

```python
fig = plt.figure(figsize=(10, 6), layout="constrained")
gs = fig.add_gridspec(2, 3, width_ratios=[2, 1, 1], height_ratios=[3, 1])
ax_grande = fig.add_subplot(gs[0, :2])
ax_derecha = fig.add_subplot(gs[:, 2])
ax_abajo = fig.add_subplot(gs[1, :2])
```

### 8.4 Gráfico dentro de otro (*inset*)

```python
detalle = ax.inset_axes([0.6, 0.6, 0.35, 0.35])      # x, y, ancho, alto (fracción)
detalle.plot(x[:20], y[:20])
```

---

## 9. Gráficos avanzados

### 9.1 Mapa de calor (`imshow`)

```python
matriz = np.random.default_rng(0).random((5, 7))
im = ax.imshow(matriz, cmap="Blues", aspect="auto")
ax.set_xticks(range(7), ["L", "M", "X", "J", "V", "S", "D"])
ax.set_yticks(range(5), ["Sem 1", "Sem 2", "Sem 3", "Sem 4", "Sem 5"])
fig.colorbar(im, ax=ax, label="Intensidad")

for i in range(matriz.shape[0]):                     # Valores en cada celda
    for j in range(matriz.shape[1]):
        ax.text(j, i, f"{matriz[i, j]:.2f}", ha="center", va="center", fontsize=8)
```

### 9.2 Contornos (`contour`, `contourf`)

```python
X, Y = np.meshgrid(np.linspace(-3, 3, 100), np.linspace(-3, 3, 100))
Z = np.exp(-(X**2 + Y**2))
cs = ax.contourf(X, Y, Z, levels=15, cmap="viridis")
ax.contour(X, Y, Z, levels=15, colors="white", linewidths=0.5)
fig.colorbar(cs, ax=ax)
```

### 9.3 Barras de error

```python
ax.errorbar(x, media, yerr=desviacion, fmt="o", capsize=4)
ax.bar(categorias, media, yerr=desviacion, capsize=4)
```

### 9.4 Gráfico polar (radar)

```python
categorias = ["Precio", "Calidad", "Servicio", "Plazo", "Garantía"]
valores = [4, 3, 5, 2, 4]
angulos = np.linspace(0, 2 * np.pi, len(categorias), endpoint=False).tolist()
valores += valores[:1]; angulos += angulos[:1]                  # Cerrar el polígono

fig, ax = plt.subplots(subplot_kw={"projection": "polar"})
ax.plot(angulos, valores, marker="o")
ax.fill(angulos, valores, alpha=0.25)
ax.set_xticks(angulos[:-1], categorias)
```

### 9.5 Líneas y zonas de referencia

```python
ax.axhline(150, color="gray", linestyle="--", linewidth=1, label="Objetivo")
ax.axvline(5, color="red", linestyle=":")
ax.axhspan(100, 150, color="green", alpha=0.08)                  # Banda horizontal
ax.axvspan(3, 5, color="orange", alpha=0.1)                      # Periodo destacado
```

---

## 10. Anotaciones y texto

```python
ax.text(2, 150, "Lanzamiento", fontsize=9, color="gray")
ax.text(0.98, 0.95, "Fuente: ERP", transform=ax.transAxes, ha="right", va="top", fontsize=8)

ax.annotate("Máximo histórico",
            xy=(5, 190),                      # Punto señalado
            xytext=(3, 205),                  # Posición del texto
            arrowprops={"arrowstyle": "->", "color": "black"},
            fontsize=9)
```

| Sistema de coordenadas | Significado |
|---|---|
| Por defecto | Coordenadas de los datos. |
| `transform=ax.transAxes` | Fracción del gráfico (0,0 abajo-izquierda; 1,1 arriba-derecha). |
| `transform=fig.transFigure` | Fracción de la figura completa. |

Fórmulas matemáticas (sintaxis LaTeX, sin necesidad de instalar LaTeX):

```python
ax.set_title(r"$y = \alpha \cdot e^{-\beta x}$")
```

---

## 11. Fechas y series temporales

Matplotlib reconoce objetos `datetime`, `date` y fechas de pandas/NumPy.

```python
import matplotlib.dates as mdates
import pandas as pd

fechas = pd.date_range("2025-01-01", periods=365, freq="D")
valores = np.random.default_rng(1).normal(0, 1, 365).cumsum()

fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(fechas, valores)
ax.xaxis.set_major_locator(mdates.MonthLocator())
ax.xaxis.set_major_formatter(mdates.DateFormatter("%b"))
fig.autofmt_xdate()                       # Inclina las etiquetas de fecha
```

Formato automático y conciso (recomendado):

```python
localizador = mdates.AutoDateLocator()
ax.xaxis.set_major_locator(localizador)
ax.xaxis.set_major_formatter(mdates.ConciseDateFormatter(localizador))
```

Nombres de los meses en español:

```python
import locale
locale.setlocale(locale.LC_TIME, "es_ES.UTF-8")    # En Windows: "Spanish_Spain"
```

---

## 12. Estilos globales y `rcParams`

### 12.1 Hojas de estilo incluidas

```python
print(plt.style.available)               # Lista de estilos
plt.style.use("seaborn-v0_8-whitegrid")
plt.style.use("ggplot")

with plt.style.context("dark_background"):  # Solo para un bloque
    fig, ax = plt.subplots()
```

### 12.2 Configuración global con `rcParams`

```python
plt.rcParams.update({
    "figure.figsize": (10, 5),
    "figure.dpi": 100,
    "font.family": "sans-serif",
    "font.size": 10,
    "axes.titlesize": 13,
    "axes.titleweight": "bold",
    "axes.spines.top": False,
    "axes.spines.right": False,
    "axes.grid": True,
    "grid.alpha": 0.3,
    "axes.prop_cycle": plt.cycler(color=["#1F4E78", "#2E75B6", "#9DC3E6", "#C00000", "#7F7F7F"]),
    "savefig.bbox": "tight",
})
```

### 12.3 Hoja de estilo corporativa

Guarde la configuración en un archivo `corporativo.mplstyle`:

```
figure.figsize : 10, 5
font.size      : 10
axes.spines.top   : False
axes.spines.right : False
axes.prop_cycle : cycler('color', ['1F4E78', '2E75B6', '9DC3E6', 'C00000'])
```

Y aplíquela con `plt.style.use("ruta/corporativo.mplstyle")`.

---

## 13. Integración con pandas

Los `DataFrame` y `Series` de pandas tienen un método `.plot()` que usa Matplotlib y devuelve un objeto `Axes`:

```python
import pandas as pd

df = pd.DataFrame({"mes": meses, "online": [50, 60, 55, 70, 80, 85],
                   "tienda": [70, 75, 73, 90, 92, 105]}).set_index("mes")

ax = df.plot(kind="bar", stacked=True, figsize=(8, 4), title="Ventas por canal")
ax.set_ylabel("Miles de €")
```

| `kind` | Gráfico |
|---|---|
| `"line"` | Líneas (por defecto) |
| `"bar"`, `"barh"` | Barras |
| `"hist"` | Histograma |
| `"box"` | Caja |
| `"scatter"` | Dispersión (requiere `x=` e `y=`) |
| `"pie"` | Sectores |
| `"area"` | Área apilada |

Combinar con ejes propios:

```python
fig, axs = plt.subplots(1, 2, figsize=(12, 4))
df["online"].plot(ax=axs[0], marker="o", title="Online")
df["tienda"].plot(kind="bar", ax=axs[1], title="Tienda")
```

---

## 14. Guardar y exportar

```python
fig.savefig("informe.png", dpi=200, bbox_inches="tight")          # Imagen de alta calidad
fig.savefig("informe.pdf")                                        # Vectorial (informes)
fig.savefig("informe.svg")                                        # Vectorial (web, edición)
fig.savefig("transparente.png", transparent=True)
fig.savefig("informe.png", facecolor="white")                     # Fondo blanco explícito
```

| Formato | Uso recomendado |
|---|---|
| PNG | Presentaciones, web, correo. Use `dpi=150–300`. |
| PDF | Informes e impresión (vectorial, no pierde calidad). |
| SVG | Web y edición posterior (Illustrator, Inkscape). |
| JPG | Evítelo: introduce artefactos en líneas y texto. |

Varias páginas en un solo PDF:

```python
from matplotlib.backends.backend_pdf import PdfPages

with PdfPages("informe_completo.pdf") as pdf:
    for region in regiones:
        fig, ax = plt.subplots()
        ...
        pdf.savefig(fig)
        plt.close(fig)
```

Generar imágenes en un servidor (sin pantalla):

```python
import matplotlib
matplotlib.use("Agg")          # Antes de importar pyplot
import matplotlib.pyplot as plt
```

Obtener la imagen en memoria (para incrustarla en HTML, correo o una API):

```python
import io, base64

buffer = io.BytesIO()
fig.savefig(buffer, format="png", bbox_inches="tight")
imagen_b64 = base64.b64encode(buffer.getvalue()).decode()
html = f'<img src="data:image/png;base64,{imagen_b64}">'
```

---

## 15. Gráficos 3D y animaciones

### 15.1 Gráficos 3D

```python
fig = plt.figure(figsize=(8, 6))
ax = fig.add_subplot(projection="3d")

X, Y = np.meshgrid(np.linspace(-3, 3, 50), np.linspace(-3, 3, 50))
Z = np.sin(np.sqrt(X**2 + Y**2))
ax.plot_surface(X, Y, Z, cmap="viridis", edgecolor="none")
ax.set(xlabel="X", ylabel="Y", zlabel="Z")
ax.view_init(elev=30, azim=45)            # Ángulo de cámara
```

> Los gráficos 3D estáticos son difíciles de leer. Para explorar datos en 3D, Plotly es una mejor opción.

### 15.2 Animaciones

```python
from matplotlib.animation import FuncAnimation

fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 200)
linea, = ax.plot(x, np.sin(x))

def actualizar(fotograma):
    linea.set_ydata(np.sin(x + fotograma / 10))
    return linea,

animacion = FuncAnimation(fig, actualizar, frames=100, interval=40, blit=True)
animacion.save("onda.gif", writer="pillow", fps=25)       # MP4: writer="ffmpeg"
```

> Mantenga una referencia a la animación (`animacion = ...`): si no, el recolector de basura la elimina y no se reproduce.

---

## 16. Ejemplo práctico completo

**Informe de ventas automatizado**: genera datos de ventas, construye un panel de cuatro gráficos con estilo corporativo (evolución mensual con objetivo, ventas por región, distribución de tickets y mapa de calor por día de la semana), añade indicadores clave y exporta el resultado a PNG y a un PDF de varias páginas (una por región).

```python
"""
Informe de ventas con Matplotlib.
Requiere: pip install matplotlib numpy pandas
"""
from pathlib import Path

import matplotlib

matplotlib.use("Agg")                      # Sin ventana: genera archivos
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import numpy as np
import pandas as pd
from matplotlib.backends.backend_pdf import PdfPages

SALIDA = Path(__file__).resolve().parent / "salida"
PALETA = ["#1F4E78", "#2E75B6", "#9DC3E6", "#C00000", "#7F7F7F"]
MESES = ["Ene", "Feb", "Mar", "Abr", "May", "Jun", "Jul", "Ago", "Sep", "Oct", "Nov", "Dic"]
DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]
OBJETIVO_MENSUAL = 95_000


# ------------------------------------------------------------------ Estilo
def aplicar_estilo() -> None:
    plt.rcParams.update({
        "font.family": "sans-serif",
        "font.size": 9,
        "axes.titlesize": 11,
        "axes.titleweight": "bold",
        "axes.titlelocation": "left",
        "axes.spines.top": False,
        "axes.spines.right": False,
        "axes.grid": True,
        "axes.grid.axis": "y",
        "grid.alpha": 0.3,
        "axes.prop_cycle": plt.cycler(color=PALETA),
        "savefig.dpi": 200,
        "savefig.bbox": "tight",
    })


def formato_euros(valor: float, _=None) -> str:
    """Formato español: 12.500 €  ·  1,2 M€"""
    if abs(valor) >= 1_000_000:
        return f"{valor / 1_000_000:.1f} M€".replace(".", ",")
    return f"{valor:,.0f} €".replace(",", ".")


# ------------------------------------------------------------------ Datos
def generar_datos(semilla: int = 7) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    fechas = pd.date_range("2025-01-01", "2025-12-31", freq="D")
    regiones = ["Norte", "Sur", "Este", "Oeste"]
    n = 12_000
    df = pd.DataFrame({
        "fecha": rng.choice(fechas, n),
        "region": rng.choice(regiones, n, p=[0.32, 0.22, 0.28, 0.18]),
        "importe": rng.lognormal(mean=4.3, sigma=0.6, size=n).round(2),
    })
    estacionalidad = 1 + 0.25 * np.sin((df["fecha"].dt.month - 3) / 12 * 2 * np.pi)
    df["importe"] = (df["importe"] * estacionalidad).round(2)
    df["mes"] = df["fecha"].dt.month
    df["dia_semana"] = df["fecha"].dt.dayofweek
    df["semana"] = df["fecha"].dt.isocalendar().week.astype(int)
    return df


# ------------------------------------------------------------------ Gráficos
def grafico_evolucion(ax: plt.Axes, df: pd.DataFrame) -> None:
    mensual = df.groupby("mes")["importe"].sum().reindex(range(1, 13), fill_value=0)
    ax.fill_between(range(12), mensual.values, alpha=0.12, color=PALETA[0])
    ax.plot(range(12), mensual.values, marker="o", markersize=4, linewidth=2, label="Ventas")
    ax.axhline(OBJETIVO_MENSUAL, color=PALETA[3], linestyle="--", linewidth=1, label="Objetivo")

    maximo = int(mensual.values.argmax())
    ax.annotate(f"Máximo\n{formato_euros(mensual.iloc[maximo])}",
                xy=(maximo, mensual.iloc[maximo]), xytext=(maximo - 2.5, mensual.max() * 1.04),
                arrowprops={"arrowstyle": "->", "color": "#404040"}, fontsize=8)

    ax.set_xticks(range(12), MESES)
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(formato_euros))
    ax.set_ylim(0, mensual.max() * 1.25)
    ax.set_title("Evolución mensual de ventas")
    ax.legend(loc="lower right", frameon=False, ncols=2)


def grafico_regiones(ax: plt.Axes, df: pd.DataFrame) -> None:
    por_region = df.groupby("region")["importe"].sum().sort_values()
    colores = [PALETA[2]] * (len(por_region) - 1) + [PALETA[0]]     # Destacar la mejor
    barras = ax.barh(por_region.index, por_region.values, color=colores)
    ax.bar_label(barras, labels=[formato_euros(v) for v in por_region.values], padding=4, fontsize=8)
    ax.xaxis.set_major_formatter(mticker.FuncFormatter(formato_euros))
    ax.xaxis.set_major_locator(mticker.MaxNLocator(5))
    ax.set_xlim(0, por_region.max() * 1.25)
    ax.grid(axis="y", visible=False)
    ax.grid(axis="x", visible=True)
    ax.set_title("Ventas por región")


def grafico_distribucion(ax: plt.Axes, df: pd.DataFrame) -> None:
    importes = df["importe"]
    limite = importes.quantile(0.99)
    ax.hist(importes[importes <= limite], bins=40, color=PALETA[1], edgecolor="white")
    media, mediana = importes.mean(), importes.median()
    ax.axvline(media, color=PALETA[3], linestyle="--", linewidth=1, label=f"Media: {formato_euros(media)}")
    ax.axvline(mediana, color="#404040", linestyle=":", linewidth=1, label=f"Mediana: {formato_euros(mediana)}")
    ax.xaxis.set_major_formatter(mticker.FuncFormatter(formato_euros))
    ax.set_title("Distribución del importe por ticket")
    ax.set_ylabel("Nº de tickets")
    ax.legend(frameon=False)


def grafico_calor(fig: plt.Figure, ax: plt.Axes, df: pd.DataFrame) -> None:
    tabla = df.pivot_table(index="dia_semana", columns="mes", values="importe",
                           aggfunc="sum", fill_value=0).reindex(index=range(7), columns=range(1, 13), fill_value=0)
    im = ax.imshow(tabla.values / 1000, cmap="Blues", aspect="auto")
    ax.set_xticks(range(12), MESES, fontsize=8)
    ax.set_yticks(range(7), DIAS, fontsize=8)
    ax.grid(False)
    cb = fig.colorbar(im, ax=ax, shrink=0.85)
    cb.set_label("Miles de €", fontsize=8)
    ax.set_title("Ventas por día de la semana y mes")


def indicadores(fig: plt.Figure, df: pd.DataFrame) -> None:
    kpis = [("Ventas totales", formato_euros(df["importe"].sum())),
            ("Nº de tickets", f"{len(df):,}".replace(",", ".")),
            ("Ticket medio", formato_euros(df["importe"].mean())),
            ("Meses sobre objetivo",
             f"{(df.groupby('mes')['importe'].sum() >= OBJETIVO_MENSUAL).sum()} de 12")]
    for i, (titulo, valor) in enumerate(kpis):
        x = 0.02 + i * 0.245
        fig.text(x, 0.905, titulo.upper(), fontsize=8, color="#7F7F7F")
        fig.text(x, 0.87, valor, fontsize=15, fontweight="bold", color=PALETA[0])


# ------------------------------------------------------------------ Informe
def crear_panel(df: pd.DataFrame, titulo: str) -> plt.Figure:
    fig = plt.figure(figsize=(14, 9))
    fig.suptitle(titulo, x=0.02, y=0.98, ha="left", fontsize=17, fontweight="bold", color="#262626")
    indicadores(fig, df)

    gs = fig.add_gridspec(2, 2, top=0.82, bottom=0.06, left=0.06, right=0.97, hspace=0.35, wspace=0.18)
    grafico_evolucion(fig.add_subplot(gs[0, 0]), df)
    grafico_regiones(fig.add_subplot(gs[0, 1]), df)
    grafico_distribucion(fig.add_subplot(gs[1, 0]), df)
    grafico_calor(fig, fig.add_subplot(gs[1, 1]), df)
    fig.text(0.97, 0.01, "Fuente: datos simulados · Generado con Matplotlib",
             ha="right", fontsize=7, color="#7F7F7F")
    return fig


def main() -> None:
    aplicar_estilo()
    SALIDA.mkdir(exist_ok=True)
    df = generar_datos()

    fig = crear_panel(df, "Informe de ventas 2025")
    fig.savefig(SALIDA / "informe_ventas.png")
    plt.close(fig)

    with PdfPages(SALIDA / "informe_por_region.pdf") as pdf:
        fig = crear_panel(df, "Informe de ventas 2025 · Total")
        pdf.savefig(fig); plt.close(fig)
        for region in sorted(df["region"].unique()):
            fig = crear_panel(df[df["region"] == region], f"Informe de ventas 2025 · {region}")
            pdf.savefig(fig); plt.close(fig)
        info = pdf.infodict()
        info["Title"] = "Informe de ventas 2025"
        info["Author"] = "Departamento de Análisis"

    print(f"Informe generado en: {SALIDA}")


if __name__ == "__main__":
    main()
```

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'matplotlib'` | Librería no instalada en el entorno activo. | `pip install matplotlib` en el mismo entorno que ejecuta el script. |
| El gráfico no aparece | Falta `plt.show()` en un script. | Añada `plt.show()` al final (en Jupyter no es necesario). |
| La imagen guardada sale en blanco | `savefig()` se llamó después de `show()`. | Llame a `savefig()` **antes** de `show()`. |
| Etiquetas cortadas al guardar | Márgenes insuficientes. | `bbox_inches="tight"` o `layout="constrained"`. |
| Títulos y etiquetas se solapan | Varios gráficos sin ajuste de márgenes. | `plt.subplots(..., layout="constrained")` o `fig.tight_layout()`. |
| `RuntimeWarning: More than 20 figures have been opened` | Figuras creadas en un bucle sin cerrarlas. | `plt.close(fig)` tras guardar cada figura. |
| Memoria que crece sin parar en un servidor | Mismo caso anterior. | `plt.close(fig)` y backend `Agg`. |
| `UserWarning: FigureCanvasAgg is non-interactive` | Se llama a `show()` con el backend `Agg`. | Use `savefig()` o cambie de backend. |
| `TclError: no display name` / falla en servidor | Backend gráfico sin pantalla. | `matplotlib.use("Agg")` antes de importar `pyplot`. |
| Caracteres como «€», «ñ» o tildes aparecen como cuadros | La fuente no tiene esos glifos. | Use una fuente con soporte (`DejaVu Sans`, `Arial`). |
| `ValueError: x and y must have same first dimension` | Listas de distinta longitud. | Compruebe `len(x) == len(y)`. |
| Fechas superpuestas en el eje X | Demasiadas etiquetas. | `fig.autofmt_xdate()` o `ConciseDateFormatter`. |
| Los colores no coinciden entre gráficos | El ciclo de colores avanza en cada serie. | Asigne colores explícitos con un diccionario `{categoría: color}`. |
| `plt.title()` modifica otro gráfico | La interfaz `pyplot` actúa sobre los «ejes actuales». | Use la interfaz orientada a objetos (`ax.set_title`). |

---

## 18. Buenas prácticas

- Use la **interfaz orientada a objetos** (`fig, ax = plt.subplots()`) en lugar de `plt.plot()` en scripts.
- Defina el estilo una sola vez (`rcParams` o una hoja `.mplstyle`) y reutilícelo en todos los informes.
- Elija el gráfico según el mensaje: **líneas** para evolución, **barras** para comparar, **dispersión** para relaciones, **histograma** para distribuciones.
- Empiece el eje Y en cero en los gráficos de barras para no exagerar diferencias.
- Elimine elementos innecesarios: bordes superior y derecho, cuadrículas intensas, leyendas redundantes.
- Ponga títulos que expliquen la conclusión («Las ventas crecen un 12 %»), no solo el contenido («Ventas»).
- Indique siempre las **unidades** en los ejes y la **fuente** de los datos.
- Use paletas aptas para daltonismo (`viridis`, `cividis`, `tab10`) y no confíe solo en el color para distinguir series.
- Guarde en **PDF o SVG** para informes y en **PNG con `dpi` ≥ 150** para presentaciones.
- Cierre las figuras (`plt.close(fig)`) cuando genere muchas imágenes en bucle.
- Encapsule cada gráfico en una función que reciba un `ax`: facilita reutilizarlo en paneles distintos.

---

## 19. Referencia rápida

```python
fig, ax = plt.subplots(figsize=(10, 5), layout="constrained")       # Figura y ejes
fig, axs = plt.subplots(2, 2) / plt.subplot_mosaic([["a", "b"], ["c", "c"]])

ax.plot(x, y) / bar / barh / scatter / hist / pie / boxplot / violinplot / fill_between / stackplot
ax.imshow(m) / contourf(X, Y, Z) / errorbar(x, y, yerr) / step / stem
ax.axhline(y) / axvline(x) / axhspan / axvspan / text / annotate / bar_label

ax.set(title=, xlabel=, ylabel=, xlim=, ylim=) / set_xticks(pos, etiquetas) / set_yscale("log")
ax.legend(loc=) / grid(alpha=) / spines[["top", "right"]].set_visible(False) / twinx()
ax.yaxis.set_major_formatter(mticker.FuncFormatter(f)) / tick_params(rotation=45)
fig.colorbar(im, ax=ax) / fig.suptitle(t) / fig.autofmt_xdate()

plt.style.use("ggplot") / plt.rcParams.update({...})
fig.savefig("g.png", dpi=200, bbox_inches="tight") / plt.show() / plt.close(fig)
df.plot(kind="bar", ax=ax)
```

---

## 20. Ejercicios propuestos

1. Represente la evolución diaria de la temperatura de una ciudad durante un año, con una banda que muestre el rango entre mínima y máxima y una media móvil de 7 días.
2. Cree un gráfico de barras agrupadas que compare las ventas de cuatro productos en tres años, con etiquetas de valor y formato en euros.
3. Construya un panel 2 × 2 con `subplot_mosaic` que combine un histograma, un diagrama de caja, una dispersión y un gráfico de sectores sobre el mismo conjunto de datos.
4. Diseñe una hoja de estilo `.mplstyle` corporativa y aplíquela a tres gráficos distintos.
5. Amplíe el ejemplo completo para añadir una página de comparación interanual (2024 frente a 2025) y una animación GIF de la evolución mensual.

---

## 21. Referencias

- Documentación oficial: https://matplotlib.org/stable/
- Galería de ejemplos: https://matplotlib.org/stable/gallery/index.html
- Guía de inicio rápido: https://matplotlib.org/stable/users/explain/quick_start.html
- Mapas de color: https://matplotlib.org/stable/users/explain/colors/colormaps.html
- Hoja de referencia (*cheatsheet*): https://matplotlib.org/cheatsheets/
