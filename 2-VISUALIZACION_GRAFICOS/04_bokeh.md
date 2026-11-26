# Manual de uso: Bokeh

> **Categoría:** Visualización de datos · **Utilidad:** Visualizaciones interactivas para navegadores
> **Tipo:** Librería externa (requiere instalación con `pip`)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer gráfico](#4-primer-gráfico)
5. [Glifos: tipos de gráfico](#5-glifos-tipos-de-gráfico)
6. [`ColumnDataSource`: la fuente de datos](#6-columndatasource-la-fuente-de-datos)
7. [Ejes, rangos, formato y estilo](#7-ejes-rangos-formato-y-estilo)
8. [Datos categóricos y colores](#8-datos-categóricos-y-colores)
9. [Herramientas e información emergente](#9-herramientas-e-información-emergente)
10. [Anotaciones](#10-anotaciones)
11. [Disposición: filas, columnas, rejillas y pestañas](#11-disposición-filas-columnas-rejillas-y-pestañas)
12. [Interactividad en el navegador (`CustomJS`)](#12-interactividad-en-el-navegador-customjs)
13. [Aplicaciones con servidor (`bokeh serve`)](#13-aplicaciones-con-servidor-bokeh-serve)
14. [Mapas y teselas](#14-mapas-y-teselas)
15. [Exportar, incrustar y temas](#15-exportar-incrustar-y-temas)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Bokeh** es una librería para crear **visualizaciones interactivas que se ejecutan en el navegador**. Se programa en Python y genera documentos HTML con la librería JavaScript **BokehJS**. Destaca por la interactividad enlazada (selección y zoom compartidos entre gráficos), los *widgets* (deslizadores, desplegables, tablas) y la posibilidad de crear aplicaciones con un **servidor Python** que actualiza los gráficos en tiempo real.

**Ventajas:**
- Interactividad nativa: zoom, desplazamiento, selección, *tooltips*.
- Gráficos **enlazados**: seleccionar puntos en un gráfico los resalta en los demás.
- *Widgets* integrados y lógica en JavaScript (`CustomJS`) sin necesidad de servidor.
- **Bokeh Server** para aplicaciones con lógica en Python y datos en *streaming*.
- Buen rendimiento con volúmenes grandes (WebGL) y actualización incremental de datos.
- Archivos HTML autónomos para compartir.

**Limitaciones:**
- Sintaxis más extensa que Plotly Express: hay que construir cada gráfico paso a paso.
- Menos tipos de gráfico «listos para usar» (no tiene *treemap* ni Sankey nativos).
- Exportar a PNG/SVG requiere Selenium y un navegador.

| Si necesita… | Opción |
|---|---|
| Gráficos enlazados, *widgets* y *streaming* en el navegador | **Bokeh** |
| Gráficos interactivos con muy poco código | Plotly Express |
| Gramática declarativa con selecciones enlazadas | Altair |
| Panel web completo y rápido de construir | Streamlit / Dash |
| Gráficos estáticos de publicación | Matplotlib / Seaborn |

---

## 2. Instalación e importación

```bash
pip install bokeh
pip install pandas                        # Recomendado
pip install selenium                      # Solo para exportar a PNG/SVG
```

Comprobar la instalación:

```bash
python -c "import bokeh; print(bokeh.__version__)"
bokeh info                                # Versiones de Bokeh, Python, Tornado, Node…
```

Importaciones habituales:

```python
from bokeh.plotting import figure, show, output_file, save, curdoc
from bokeh.models import ColumnDataSource, HoverTool, NumeralTickFormatter, Select, CustomJS
from bokeh.layouts import row, column, gridplot, layout
from bokeh.palettes import Category10, Blues9, Viridis256
from bokeh.transform import factor_cmap, linear_cmap, dodge, cumsum
from bokeh.io import output_notebook       # Para Jupyter
```

Dónde se muestran los gráficos:

```python
output_file("grafico.html", title="Mi gráfico")   # Archivo HTML (scripts)
output_notebook()                                 # Dentro de Jupyter
show(p)                                           # Abre el navegador / muestra en el cuaderno
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `figure` | Lienzo de un gráfico con ejes, cuadrícula, herramientas y leyenda. |
| Glifo (*glyph*) | Forma visual que representa los datos: líneas, barras, puntos, rectángulos, cuñas… Se añade con métodos como `p.line()` o `p.vbar()`. |
| `ColumnDataSource` (CDS) | Tabla de datos compartida entre Python y el navegador. Permite enlazar gráficos, *tooltips* y *widgets*. |
| Herramientas (*tools*) | Zoom, desplazamiento, selección, *hover*, guardado… en la barra del gráfico. |
| Modelos (*models*) | Todos los objetos de Bokeh (ejes, leyendas, *widgets*, herramientas) son modelos configurables. |
| Disposición (*layout*) | Organización de varios gráficos y *widgets*: `row`, `column`, `gridplot`, pestañas. |
| `CustomJS` | Código JavaScript ejecutado en el navegador en respuesta a eventos, sin servidor. |
| Bokeh Server | Servidor que mantiene sincronizados los objetos Python y el navegador; permite *callbacks* en Python. |
| Documento (`curdoc`) | Contenedor de todos los modelos de una aplicación con servidor. |

**Dos modos de trabajo:**

| Modo | Lógica | Distribución |
|---|---|---|
| Documento autónomo | Interactividad en JavaScript (`CustomJS`). | Un archivo `.html` que se abre sin servidor. |
| Aplicación de servidor | *Callbacks* en Python (consultas, modelos, cálculos). | `bokeh serve app.py`; requiere un proceso en ejecución. |

---

## 4. Primer gráfico

```python
from bokeh.plotting import figure, output_file, show

meses = ["Ene", "Feb", "Mar", "Abr", "May", "Jun"]
ventas = [120, 135, 128, 160, 172, 190]

output_file("ventas.html", title="Ventas")

p = figure(x_range=meses, title="Ventas del primer semestre",
           width=700, height=350, x_axis_label="Mes", y_axis_label="Miles de €",
           tools="pan,wheel_zoom,box_zoom,reset,save", toolbar_location="above")
p.vbar(x=meses, top=ventas, width=0.7, color="#2E75B6")
p.line(meses, ventas, color="#1F4E78", line_width=2)
p.scatter(meses, ventas, size=8, color="#1F4E78")

show(p)
```

| Parámetro de `figure` | Efecto |
|---|---|
| `width`, `height` | Tamaño en píxeles. |
| `sizing_mode` | `"fixed"`, `"stretch_width"`, `"stretch_both"`, `"scale_width"`: adaptación al contenedor. |
| `x_range`, `y_range` | Rango numérico (`(0, 100)`) o lista de categorías. |
| `x_axis_type` | `"linear"`, `"log"`, `"datetime"`, `"mercator"`. |
| `tools` | Herramientas disponibles (texto separado por comas o lista). |
| `active_scroll` | Herramienta activa con la rueda (`"wheel_zoom"`). |
| `toolbar_location` | `"above"`, `"below"`, `"left"`, `"right"` o `None`. |
| `output_backend` | `"webgl"` para acelerar gráficos con muchos puntos. |

---

## 5. Glifos: tipos de gráfico

### 5.1 Líneas

```python
p.line(x, y, line_width=2, color="navy", legend_label="Ventas")
p.line(x, y2, line_dash="dashed", line_alpha=0.6)
p.step(x, y, mode="center")
p.multi_line(xs=[[1, 2, 3], [1, 2, 3]], ys=[[3, 5, 4], [2, 3, 6]], color=["red", "blue"])
```

### 5.2 Puntos (`scatter`)

```python
p.scatter(x, y, size=10, color="firebrick", alpha=0.6)
p.scatter(x, y, marker="triangle", size=12, fill_color="white", line_color="navy")
```

Marcadores: `"circle"`, `"square"`, `"triangle"`, `"diamond"`, `"inverted_triangle"`, `"star"`, `"hex"`, `"plus"`, `"x"`, `"cross"`…

> Use `p.scatter(...)`, no `p.circle(size=...)`: en versiones recientes `circle` se reserva para círculos con `radius` en unidades de datos.

### 5.3 Barras

```python
p.vbar(x=categorias, top=valores, width=0.8)                  # Verticales
p.hbar(y=categorias, right=valores, height=0.8)               # Horizontales
p.vbar(x=x, top=y, bottom=base, width=0.8)                    # Con base distinta de 0
p.quad(left=bordes[:-1], right=bordes[1:], top=frecuencias, bottom=0)   # Histograma
```

Histograma con NumPy:

```python
import numpy as np
frecuencias, bordes = np.histogram(datos, bins=30)
p.quad(top=frecuencias, bottom=0, left=bordes[:-1], right=bordes[1:],
       fill_color="#2E75B6", line_color="white")
```

### 5.4 Áreas y bandas

```python
p.varea(x=x, y1=0, y2=y, fill_alpha=0.3)                            # Área bajo la curva
p.varea(x=x, y1=y_min, y2=y_max, fill_color="gray", fill_alpha=0.2)  # Banda (intervalo)
p.varea_stack(stackers=["online", "tienda"], x="mes", source=fuente, color=["#2E75B6", "#9DC3E6"])
```

### 5.5 Sectores y anillos

```python
from math import pi
from bokeh.transform import cumsum

datos = pd.DataFrame({"canal": ["Web", "App", "Tienda"], "valor": [45, 30, 25]})
datos["angulo"] = datos["valor"] / datos["valor"].sum() * 2 * pi
datos["color"] = ["#1F4E78", "#2E75B6", "#9DC3E6"]

p = figure(height=350, toolbar_location=None, x_range=(-1, 1), y_range=(-1, 1))
p.annular_wedge(x=0, y=0, inner_radius=0.4, outer_radius=0.8,          # wedge() para sectores
                start_angle=cumsum("angulo", include_zero=True), end_angle=cumsum("angulo"),
                color="color", legend_field="canal", source=ColumnDataSource(datos))
p.axis.visible = False; p.grid.visible = False
```

### 5.6 Rectángulos, mapas de calor e imágenes

```python
p.rect(x="hora", y="dia", width=1, height=1, source=fuente,
       fill_color=linear_cmap("valor", "Blues9", low=0, high=100), line_color=None)
p.image(image=[matriz], x=0, y=0, dw=10, dh=10, palette="Viridis256")
p.hex_tile(q="q", r="r", size=1, source=fuente)                       # Hexágonos
```

### 5.7 Segmentos, texto y otros

```python
p.segment(x0=x, y0=y_min, x1=x, y1=y_max, line_width=2)      # Bigotes, rangos
p.text(x=[1], y=[5], text=["Etiqueta"], text_font_size="10pt")
p.ellipse(x=0, y=0, width=2, height=1, angle=0.3, fill_alpha=0.2)
```

---

## 6. `ColumnDataSource`: la fuente de datos

Es la pieza central de Bokeh: un diccionario de columnas que se envía al navegador. Todos los glifos que la comparten quedan **enlazados** (selección, filtrado, actualización).

```python
from bokeh.models import ColumnDataSource
import pandas as pd

df = pd.DataFrame({"mes": meses, "ventas": ventas, "margen": [0.21, 0.24, 0.22, 0.27, 0.25, 0.3]})
fuente = ColumnDataSource(df)                 # Desde un DataFrame (el índice se añade como "index")
fuente = ColumnDataSource(data={"mes": meses, "ventas": ventas})   # Desde un diccionario

p.vbar(x="mes", top="ventas", width=0.8, source=fuente)        # Columnas por nombre
```

Actualizar los datos (en servidor o desde `CustomJS`):

```python
fuente.data = {"mes": nuevos_meses, "ventas": nuevas_ventas}  # Sustituir todo
fuente.data = ColumnDataSource.from_df(df_filtrado)            # Desde otro DataFrame
fuente.stream({"mes": ["Jul"], "ventas": [205]}, rollover=365) # Añadir filas (streaming)
fuente.patch({"ventas": [(0, 125)]})                           # Modificar valores concretos
```

### 6.1 Vistas y filtros (`CDSView`)

Muestran solo una parte de la fuente sin duplicar datos:

```python
from bokeh.models import CDSView, GroupFilter, BooleanFilter

vista_norte = CDSView(filter=GroupFilter(column_name="region", group="Norte"))
p.scatter("precio", "unidades", source=fuente, view=vista_norte)

vista_altas = CDSView(filter=BooleanFilter((df["ventas"] > 150).tolist()))
```

### 6.2 Selección enlazada

Dos gráficos con la **misma fuente** comparten la selección automáticamente:

```python
fuente = ColumnDataSource(df)
herramientas = "box_select,lasso_select,reset"
p1 = figure(tools=herramientas); p1.scatter("precio", "unidades", source=fuente)
p2 = figure(tools=herramientas); p2.scatter("margen", "unidades", source=fuente)
show(row(p1, p2))          # Seleccionar en p1 resalta los mismos puntos en p2
```

---

## 7. Ejes, rangos, formato y estilo

### 7.1 Títulos y ejes

```python
p.title.text = "Ventas mensuales"
p.title.text_font_size = "14pt"
p.title.align = "left"
p.xaxis.axis_label = "Mes"
p.yaxis.axis_label = "Importe (€)"
p.xaxis.major_label_orientation = 0.8           # Radianes (≈ 45°)
p.axis.minor_tick_line_color = None             # Sin marcas menores
p.yaxis.visible = False
```

### 7.2 Formato de números y fechas

```python
from bokeh.models import NumeralTickFormatter, DatetimeTickFormatter, PrintfTickFormatter

p.yaxis.formatter = NumeralTickFormatter(format="0,0 $")    # 12,500 $
p.yaxis.formatter = NumeralTickFormatter(format="0.0%")     # 25.3%
p.yaxis.formatter = NumeralTickFormatter(format="0.0a")     # 1.2m
p.yaxis.formatter = PrintfTickFormatter(format="%d €")
```

> `NumeralTickFormatter` y `NumberFormatter` no muestran el símbolo «€» escrito dentro del formato (solo interpretan `$`). Para euros, use `PrintfTickFormatter(format="%d €")` o indique la unidad en el título del eje o de la columna.

```python
p.xaxis.formatter = DatetimeTickFormatter(days="%d/%m", months="%b %Y", years="%Y")
```

### 7.3 Rangos

```python
from bokeh.models import Range1d, DataRange1d

p.y_range.start = 0                       # Fijar solo el inicio
p.y_range = Range1d(0, 250)               # Rango fijo
p2 = figure(x_range=p1.x_range)           # Compartir rango: zoom sincronizado
```

### 7.4 Estilo del lienzo y la cuadrícula

```python
p.background_fill_color = "#FAFAFA"
p.border_fill_color = "white"
p.outline_line_color = None
p.xgrid.grid_line_color = None
p.ygrid.grid_line_alpha = 0.4
p.ygrid.grid_line_dash = [4, 4]
p.ygrid.band_fill_color = "#F2F2F2"; p.ygrid.band_fill_alpha = 0.5    # Bandas alternas
```

### 7.5 Leyenda

```python
p.line(x, y1, legend_label="2024")
p.line(x, y2, legend_label="2025")
p.legend.location = "top_left"
p.legend.orientation = "horizontal"
p.legend.click_policy = "hide"            # Clic en la leyenda oculta la serie ("mute" la atenúa)
p.legend.background_fill_alpha = 0.7
p.add_layout(p.legend[0], "right")        # Leyenda fuera del área del gráfico
```

| Parámetro del glifo | Uso |
|---|---|
| `legend_label="Texto"` | Una entrada fija. |
| `legend_field="columna"` | Una entrada por valor de la columna (con `source`). |
| `legend_group="columna"` | Igual, agrupando en Python (más eficiente). |

---

## 8. Datos categóricos y colores

### 8.1 Ejes categóricos

```python
regiones = ["Norte", "Sur", "Este", "Oeste"]
p = figure(x_range=regiones)                              # Orden fijado por la lista
p = figure(y_range=list(reversed(regiones)))              # Horizontal, de arriba abajo
```

### 8.2 Barras agrupadas (`dodge`) y apiladas

```python
from bokeh.transform import dodge

fuente = ColumnDataSource({"region": regiones, "2024": [120, 95, 140, 80], "2025": [135, 110, 150, 92]})
p = figure(x_range=regiones, height=350)
p.vbar(x=dodge("region", -0.2, range=p.x_range), top="2024", width=0.38, source=fuente,
       color="#9DC3E6", legend_label="2024")
p.vbar(x=dodge("region", 0.2, range=p.x_range), top="2025", width=0.38, source=fuente,
       color="#1F4E78", legend_label="2025")

p.vbar_stack(["online", "tienda"], x="region", width=0.8, source=fuente2,
             color=["#2E75B6", "#9DC3E6"], legend_label=["Online", "Tienda"])
```

Categorías anidadas (dos niveles):

```python
from bokeh.models import FactorRange
factores = [("Norte", "2024"), ("Norte", "2025"), ("Sur", "2024"), ("Sur", "2025")]
p = figure(x_range=FactorRange(*factores))
p.vbar(x=factores, top=[120, 135, 95, 110], width=0.9)
```

### 8.3 Colores según los datos

```python
from bokeh.transform import factor_cmap, linear_cmap
from bokeh.palettes import Category10, Blues9, Viridis256
from bokeh.models import ColorBar

# Categórico
p.vbar(x="region", top="ventas", source=fuente,
       fill_color=factor_cmap("region", palette=Category10[4], factors=regiones))

# Continuo con barra de color
mapa = linear_cmap("margen", palette=Viridis256, low=0, high=0.4)
p.scatter("precio", "unidades", color=mapa, size=9, source=fuente)
p.add_layout(ColorBar(color_mapper=mapa.transform, title="Margen"), "right")
```

Paletas: `Category10`, `Category20`, `Colorblind`, `Set2`, `Blues9`, `Greens9`, `RdYlGn11`, `Viridis256`, `Cividis256`… (en `bokeh.palettes`).

---

## 9. Herramientas e información emergente

### 9.1 Herramientas disponibles

| Herramienta | Función |
|---|---|
| `pan` | Desplazar arrastrando. |
| `wheel_zoom`, `box_zoom` | Zoom con rueda o con rectángulo. |
| `xwheel_zoom`, `xpan` | Solo en el eje X (series temporales). |
| `box_select`, `lasso_select`, `tap` | Seleccionar puntos. |
| `hover` | Información emergente. |
| `crosshair` | Cruceta. |
| `reset`, `save`, `undo`, `redo` | Restablecer, descargar PNG, deshacer. |
| `fullscreen`, `copy` | Pantalla completa, copiar imagen. |

```python
p = figure(tools="xpan,xwheel_zoom,box_zoom,reset,save", active_scroll="xwheel_zoom")
p.toolbar.logo = None
p.toolbar.autohide = True
```

### 9.2 *Hover* (`HoverTool`)

```python
from bokeh.models import HoverTool

hover = HoverTool(tooltips=[
    ("Producto", "@producto"),
    ("Precio", "@precio{0,0.00} €"),
    ("Margen", "@margen{0.0%}"),
    ("Fecha", "@fecha{%d/%m/%Y}"),
    ("Columna con espacios", "@{nombre columna}"),
], formatters={"@fecha": "datetime"}, mode="mouse")
p.add_tools(hover)
```

| Sintaxis | Significado |
|---|---|
| `@columna` | Valor de una columna del `ColumnDataSource`. |
| `$x`, `$y` | Coordenadas del cursor. |
| `$index` | Índice de la fila. |
| `{0,0.00}` | Formato numérico (*numeral.js*). |
| `{%d/%m/%Y}` + `formatters` | Formato de fecha. |

Limitar el *hover* a un glifo concreto y modo vertical:

```python
linea = p.line("fecha", "ventas", source=fuente)
p.add_tools(HoverTool(renderers=[linea], mode="vline",
                      tooltips=[("Fecha", "@fecha{%d/%m}"), ("Ventas", "@ventas{0,0} €")],
                      formatters={"@fecha": "datetime"}))
```

*Tooltip* en HTML:

```python
p.add_tools(HoverTool(tooltips="""
    <div style="font-family: Segoe UI; padding: 4px">
      <b>@producto</b><br>
      <span style="color:#1F4E78">@ventas{0,0} €</span>
    </div>"""))
```

---

## 10. Anotaciones

```python
from bokeh.models import Span, BoxAnnotation, Label, LabelSet, Arrow, NormalHead, Whisker, Band

p.add_layout(Span(location=150, dimension="width", line_color="red", line_dash="dashed"))   # Línea horizontal
p.add_layout(BoxAnnotation(left=fecha_ini, right=fecha_fin, fill_color="orange", fill_alpha=0.1))
p.add_layout(BoxAnnotation(top=50, fill_color="red", fill_alpha=0.05))                      # Zona inferior
p.add_layout(Label(x=5, y=190, text="Máximo", text_font_size="9pt", x_offset=5))
p.add_layout(LabelSet(x="mes", y="ventas", text="ventas", source=fuente,
                      text_align="center", y_offset=5, text_font_size="8pt"))               # Valores sobre barras
p.add_layout(Arrow(end=NormalHead(size=8), x_start=3, y_start=200, x_end=5, y_end=190))
p.add_layout(Whisker(base="mes", upper="max", lower="min", source=fuente))                  # Barras de error
p.add_layout(Band(base="x", lower="inf", upper="sup", source=fuente, fill_alpha=0.2))       # Banda de confianza
```

---

## 11. Disposición: filas, columnas, rejillas y pestañas

```python
from bokeh.layouts import row, column, gridplot, layout
from bokeh.models import TabPanel, Tabs, Div

show(row(p1, p2))
show(column(titulo, row(p1, p2), p3, sizing_mode="stretch_width"))
show(gridplot([[p1, p2], [p3, None]], width=400, height=300, toolbar_location="right"))  # Barra común
show(layout([[selector, deslizador], [p1, p2], [tabla]], sizing_mode="stretch_width"))

pestanas = Tabs(tabs=[TabPanel(child=p1, title="Ventas"), TabPanel(child=p2, title="Margen")])
cabecera = Div(text="<h2 style='color:#1F4E78'>Panel de ventas</h2>")
```

> `gridplot` combina las barras de herramientas en una sola; si los gráficos comparten `x_range`, el zoom se sincroniza.

---

## 12. Interactividad en el navegador (`CustomJS`)

Los *widgets* y eventos pueden ejecutar JavaScript directamente en el navegador. El documento sigue siendo un **HTML autónomo**.

### 12.1 *Widgets* principales

```python
from bokeh.models import (Select, MultiChoice, Slider, RangeSlider, DateRangeSlider, CheckboxGroup,
                          RadioButtonGroup, TextInput, Button, Toggle, DataTable, TableColumn)

Select(title="Región", value="Todas", options=["Todas", "Norte", "Sur"])
MultiChoice(title="Productos", value=["A"], options=["A", "B", "C"])
Slider(title="Umbral", start=0, end=100, value=50, step=5)
DateRangeSlider(title="Periodo", start=fecha_min, end=fecha_max, value=(fecha_min, fecha_max))
RadioButtonGroup(labels=["Ventas", "Unidades"], active=0)
CheckboxGroup(labels=["Mostrar media"], active=[0])
```

### 12.2 Enlazar propiedades sin código (`js_link`)

```python
deslizador = Slider(start=1, end=20, value=8, title="Tamaño de punto")
puntos = p.scatter("x", "y", source=fuente, size=8)
deslizador.js_link("value", puntos.glyph, "size")
```

### 12.3 *Callbacks* en JavaScript

```python
from bokeh.models import CustomJS

fuente_completa = ColumnDataSource(df)                   # Todos los datos
fuente_visible = ColumnDataSource(df)                    # Lo que se dibuja

selector = Select(title="Región", value="Todas", options=["Todas", *sorted(df["region"].unique())])
selector.js_on_change("value", CustomJS(
    args={"completa": fuente_completa, "visible": fuente_visible},
    code="""
        const region = cb_obj.value;
        const d = completa.data;
        const nuevo = {};
        for (const k of Object.keys(d)) nuevo[k] = [];
        for (let i = 0; i < d.region.length; i++) {
            if (region === "Todas" || d.region[i] === region) {
                for (const k of Object.keys(d)) nuevo[k].push(d[k][i]);
            }
        }
        visible.data = nuevo;
    """))
```

| Variable en `CustomJS` | Significado |
|---|---|
| `cb_obj` | Objeto que disparó el evento (el *widget*). |
| `args` | Modelos de Python accesibles por nombre en el código JS. |

Eventos de interacción:

```python
from bokeh.events import Tap, SelectionGeometry
fuente.selected.js_on_change("indices", CustomJS(args={"s": fuente}, code="""
    console.log("Seleccionados:", s.selected.indices.length);
"""))
p.js_on_event(Tap, CustomJS(code="console.log(cb_obj.x, cb_obj.y)"))
```

### 12.4 Navegador de rango (`RangeTool`)

```python
from bokeh.models import RangeTool

principal = figure(x_axis_type="datetime", x_range=(fechas[200], fechas[300]), height=300)
principal.line("fecha", "valor", source=fuente)

mini = figure(x_axis_type="datetime", height=110, y_range=principal.y_range, toolbar_location=None)
mini.line("fecha", "valor", source=fuente)
herramienta = RangeTool(x_range=principal.x_range)
mini.add_tools(herramienta)
mini.toolbar.active_multi = "auto"
show(column(principal, mini))
```

---

## 13. Aplicaciones con servidor (`bokeh serve`)

Con el servidor, los *callbacks* se escriben en **Python**: pueden consultar bases de datos, recalcular modelos o leer archivos.

```python
# app.py
import numpy as np
from bokeh.layouts import column
from bokeh.models import ColumnDataSource, Slider
from bokeh.plotting import curdoc, figure

x = np.linspace(0, 10, 300)
fuente = ColumnDataSource({"x": x, "y": np.sin(x)})

p = figure(height=350, sizing_mode="stretch_width", title="Onda")
p.line("x", "y", source=fuente, line_width=2)
frecuencia = Slider(title="Frecuencia", start=0.1, end=5, value=1, step=0.1)

def actualizar(attr, old, new):                       # Se ejecuta en Python
    fuente.data = {"x": x, "y": np.sin(frecuencia.value * x)}

frecuencia.on_change("value", actualizar)

curdoc().add_root(column(frecuencia, p))
curdoc().title = "Aplicación Bokeh"
```

Ejecutar:

```bash
bokeh serve --show app.py                         # http://localhost:5006/app
bokeh serve app.py --port 8080 --allow-websocket-origin=midominio.com
```

Tareas periódicas (*streaming*):

```python
def nuevo_dato():
    fuente.stream({"t": [time.time()], "valor": [leer_sensor()]}, rollover=500)

curdoc().add_periodic_callback(nuevo_dato, 1000)      # Cada 1000 ms
```

| Método de `curdoc()` | Uso |
|---|---|
| `add_root(modelo)` | Añade el contenido de la página. |
| `add_periodic_callback(f, ms)` | Ejecuta `f` periódicamente. |
| `add_timeout_callback(f, ms)` | Ejecuta `f` una vez. |
| `add_next_tick_callback(f)` | Actualiza la interfaz desde otro hilo de forma segura. |

---

## 14. Mapas y teselas

Bokeh dibuja mapas en proyección **Web Mercator**. Convierta latitud/longitud a metros antes de dibujar:

```python
import numpy as np

def a_mercator(lat, lon):
    radio = 6_378_137
    x = np.radians(lon) * radio
    y = np.log(np.tan(np.pi / 4 + np.radians(lat) / 2)) * radio
    return x, y

df["x"], df["y"] = a_mercator(df["lat"], df["lon"])

p = figure(x_axis_type="mercator", y_axis_type="mercator", width=700, height=550,
           x_range=(-1_100_000, 450_000), y_range=(4_250_000, 5_500_000))   # España
p.add_tile("OpenStreetMap Mapnik")       # Otros: "Esri World Imagery", "OpenTopoMap"
p.scatter("x", "y", size="tamano", source=ColumnDataSource(df), fill_alpha=0.7)
```

> Algunos proveedores de teselas (por ejemplo, CartoDB) exigen ahora una clave de API; Bokeh lo indica con un aviso al guardar. OpenStreetMap y Esri pueden usarse sin clave (respete sus condiciones de uso).

Coropletas con GeoJSON:

```python
from bokeh.models import GeoJSONDataSource
with open("regiones.geojson", encoding="utf-8") as f:
    fuente_geo = GeoJSONDataSource(geojson=f.read())
p.patches("xs", "ys", source=fuente_geo, fill_color=linear_cmap("ventas", "Blues9", 0, 1000),
          line_color="white")
```

> Para análisis geográfico intensivo, Folium o Plotly son más directos.

---

## 15. Exportar, incrustar y temas

### 15.1 HTML

```python
from bokeh.plotting import output_file, save
from bokeh.resources import CDN, INLINE

output_file("panel.html", title="Panel")
save(diseno)                                   # Usa la CDN (ligero, requiere internet)
save(diseno, resources=INLINE)                 # Autónomo (funciona sin conexión)
```

### 15.2 Imagen (requiere Selenium y un navegador)

```python
from bokeh.io import export_png, export_svg
export_png(p, filename="grafico.png")
p.output_backend = "svg"; export_svg(p, filename="grafico.svg")
```

### 15.3 Incrustar en una web (Flask, Django)

```python
from bokeh.embed import components, json_item
script, div = components(p)                    # Insertar {{ script|safe }} y {{ div|safe }} en la plantilla
item = json_item(p, "grafico")                 # JSON para cargar con Bokeh.embed.embed_item()
```

### 15.4 Temas

```python
from bokeh.io import curdoc
from bokeh.themes import Theme

curdoc().theme = "light_minimal"               # caliber, dark_minimal, light_minimal, night_sky, contrast

curdoc().theme = Theme(json={"attrs": {
    "figure": {"background_fill_color": "white", "outline_line_color": None, "height": 350},
    "Axis": {"axis_line_color": "#BFBFBF", "major_tick_line_color": "#BFBFBF",
             "minor_tick_line_color": None, "axis_label_text_font_style": "normal"},
    "Grid": {"grid_line_alpha": 0.4, "grid_line_dash": [4, 4]},
    "Title": {"text_font_size": "13pt", "text_color": "#1F4E78"},
}})
```

> El tema se aplica al **guardar** o **mostrar** el documento; defínalo antes de `save()` / `show()`.

---

## 16. Ejemplo práctico completo

**Panel de seguimiento de ventas autónomo (HTML sin servidor)**: con datos diarios de ventas por tienda, construye un panel con selector de región, deslizador de fechas, serie temporal con navegador de rango (`RangeTool`), gráfico de barras por tienda, dispersión precio/unidades con **selección enlazada** a una tabla de datos, y mapa de tiendas sobre teselas. Toda la interactividad se resuelve con `CustomJS`, por lo que el resultado es un único archivo HTML que se abre en cualquier navegador.

```python
"""
Panel interactivo de ventas con Bokeh (HTML autónomo, sin servidor).
Requiere: pip install bokeh pandas numpy
"""
from pathlib import Path

import numpy as np
import pandas as pd
from bokeh.io import curdoc, save
from bokeh.layouts import column, row
from bokeh.models import (BoxAnnotation, ColumnDataSource, CustomJS, DataTable, DateRangeSlider, Div,
                          HoverTool, NumberFormatter, NumeralTickFormatter, RangeTool, Select, Span,
                          TableColumn)
from bokeh.palettes import Category10
from bokeh.plotting import figure, output_file
from bokeh.resources import CDN
from bokeh.themes import Theme
from bokeh.transform import factor_cmap

SALIDA = Path(__file__).resolve().parent / "salida_bokeh"
TIENDAS = pd.DataFrame({
    "tienda": ["Bilbao", "Oviedo", "Madrid", "Valladolid", "Sevilla", "Málaga", "Valencia", "Barcelona"],
    "region": ["Norte", "Norte", "Centro", "Centro", "Sur", "Sur", "Este", "Este"],
    "lat": [43.263, 43.362, 40.417, 41.652, 37.389, 36.721, 39.470, 41.387],
    "lon": [-2.935, -5.849, -3.704, -4.724, -5.984, -4.421, -0.376, 2.169],
})
REGIONES = ["Norte", "Centro", "Sur", "Este"]
COLORES = dict(zip(REGIONES, Category10[4]))
AZUL = "#1F4E78"


# ------------------------------------------------------------------ Datos
def generar_datos(semilla: int = 21) -> tuple[pd.DataFrame, pd.DataFrame]:
    rng = np.random.default_rng(semilla)
    fechas = pd.date_range("2025-01-01", "2025-12-31", freq="D")
    filas = []
    for _, t in TIENDAS.iterrows():
        nivel = rng.uniform(2_000, 6_000)
        estacional = 1 + 0.25 * np.sin(2 * np.pi * (fechas.dayofyear - 100) / 365)
        semanal = np.where(fechas.dayofweek >= 5, 1.3, 1.0)
        filas.append(pd.DataFrame({"fecha": fechas, "tienda": t["tienda"], "region": t["region"],
                                   "ventas": (nivel * estacional * semanal * rng.normal(1, 0.12, len(fechas))).round(2)}))
    diario = pd.concat(filas, ignore_index=True)

    productos = pd.DataFrame({
        "producto": [f"P-{i:03d}" for i in range(1, 121)],
        "region": rng.choice(REGIONES, 120),
        "precio": rng.uniform(5, 250, 120).round(2),
    })
    productos["unidades"] = (8_000 / productos["precio"] ** 0.8 * rng.lognormal(0, 0.35, 120)).round().astype(int)
    productos["ingresos"] = (productos["precio"] * productos["unidades"]).round(2)
    productos["margen"] = rng.normal(0.3, 0.08, 120).clip(0.05, 0.6).round(3)
    return diario, productos


def a_mercator(lat: pd.Series, lon: pd.Series) -> tuple[np.ndarray, np.ndarray]:
    radio = 6_378_137
    return np.radians(lon) * radio, np.log(np.tan(np.pi / 4 + np.radians(lat) / 2)) * radio


# ------------------------------------------------------------------ Tema
def aplicar_tema() -> None:
    curdoc().theme = Theme(json={"attrs": {
        "figure": {"background_fill_color": "white", "border_fill_color": "white",
                   "outline_line_color": None, "min_border_left": 60},
        "Axis": {"axis_line_color": "#BFBFBF", "major_tick_line_color": "#BFBFBF",
                 "minor_tick_line_color": None, "major_label_text_color": "#404040",
                 "axis_label_text_font_style": "normal", "axis_label_text_color": "#404040"},
        "Grid": {"grid_line_alpha": 0.35, "grid_line_dash": [4, 4]},
        "Title": {"text_font_size": "12pt", "text_color": AZUL, "text_font_style": "bold"},
        "Legend": {"background_fill_alpha": 0.8, "border_line_color": None, "label_text_font_size": "9pt"},
    }})


# ------------------------------------------------------------------ Panel
def construir_panel(diario: pd.DataFrame, productos: pd.DataFrame):
    # --- Fuentes de datos
    total_diario = diario.pivot_table(index="fecha", columns="region", values="ventas", aggfunc="sum")
    total_diario["Todas"] = total_diario.sum(axis=1)
    serie = pd.DataFrame({"fecha": total_diario.index})
    for col in total_diario.columns:
        serie[col] = total_diario[col].values
        serie[f"{col}_media"] = total_diario[col].rolling(14, min_periods=1).mean().values
    serie["valor"], serie["media"] = serie["Todas"], serie["Todas_media"]
    fuente_serie = ColumnDataSource(serie)

    fuente_diario = ColumnDataSource(diario.assign(ts=diario["fecha"].dt.as_unit("ms").astype("int64")))  # ms, como el deslizador
    por_tienda = diario.groupby(["tienda", "region"], as_index=False)["ventas"].sum().sort_values("ventas")
    fuente_tiendas = ColumnDataSource(por_tienda)

    fuente_productos = ColumnDataSource(productos)
    tiendas_mapa = TIENDAS.merge(por_tienda[["tienda", "ventas"]], on="tienda")
    tiendas_mapa["x"], tiendas_mapa["y"] = a_mercator(tiendas_mapa["lat"], tiendas_mapa["lon"])
    tiendas_mapa["tamano"] = 12 + 30 * tiendas_mapa["ventas"] / tiendas_mapa["ventas"].max()
    fuente_mapa = ColumnDataSource(tiendas_mapa)

    # --- KPI
    kpi = Div(text="", styles={"font-size": "13px"})

    # --- Serie temporal con navegador de rango
    fechas = serie["fecha"]
    p_serie = figure(height=300, sizing_mode="stretch_width", x_axis_type="datetime",
                     x_range=(fechas.iloc[-90], fechas.iloc[-1]), title="Ventas diarias (media móvil 14 días)",
                     tools="xpan,xwheel_zoom,reset,save", active_scroll="xwheel_zoom", toolbar_location="above")
    p_serie.varea(x="fecha", y1=0, y2="valor", source=fuente_serie, fill_color="#9DC3E6", fill_alpha=0.35)
    linea = p_serie.line("fecha", "media", source=fuente_serie, color=AZUL, line_width=2.5)
    p_serie.yaxis.formatter = NumeralTickFormatter(format="0,0")
    p_serie.yaxis.axis_label = "€"
    p_serie.add_tools(HoverTool(renderers=[linea], mode="vline", formatters={"@fecha": "datetime"},
                                tooltips=[("Fecha", "@fecha{%d/%m/%Y}"), ("Ventas", "@valor{0,0} €"),
                                          ("Media 14 d", "@media{0,0} €")]))
    p_serie.add_layout(BoxAnnotation(left=pd.Timestamp("2025-07-01"), right=pd.Timestamp("2025-08-31"),
                                     fill_color="orange", fill_alpha=0.07))
    media_anual = Span(location=float(serie["valor"].mean()), dimension="width",
                       line_color="#C00000", line_dash="dashed", line_width=1)
    p_serie.add_layout(media_anual)

    p_nav = figure(height=90, sizing_mode="stretch_width", x_axis_type="datetime", y_range=p_serie.y_range,
                   toolbar_location=None, tools="")
    p_nav.line("fecha", "media", source=fuente_serie, color=AZUL)
    p_nav.yaxis.visible = False
    p_nav.ygrid.grid_line_color = None
    navegador = RangeTool(x_range=p_serie.x_range)
    navegador.overlay.fill_color, navegador.overlay.fill_alpha = AZUL, 0.15
    p_nav.add_tools(navegador)

    # --- Barras por tienda
    p_tiendas = figure(y_range=por_tienda["tienda"].tolist(), height=330, sizing_mode="stretch_width",
                       title="Ventas por tienda", tools="hover,save", toolbar_location=None,
                       tooltips=[("Tienda", "@tienda"), ("Región", "@region"), ("Ventas", "@ventas{0,0} €")])
    p_tiendas.hbar(y="tienda", right="ventas", height=0.7, source=fuente_tiendas,
                   fill_color=factor_cmap("region", palette=list(COLORES.values()), factors=REGIONES),
                   line_color=None, legend_field="region")
    p_tiendas.xaxis.formatter = NumeralTickFormatter(format="0.0a")
    p_tiendas.xaxis.axis_label = "€"
    p_tiendas.x_range.start = 0
    p_tiendas.ygrid.grid_line_color = None
    p_tiendas.legend.location = "bottom_right"

    # --- Mapa
    p_mapa = figure(x_axis_type="mercator", y_axis_type="mercator", height=330, sizing_mode="stretch_width",
                    x_range=(-1_050_000, 420_000), y_range=(4_330_000, 5_520_000), title="Mapa de tiendas",
                    tools="pan,wheel_zoom,reset", active_scroll="wheel_zoom")
    p_mapa.add_tile("OpenStreetMap Mapnik")
    circulos = p_mapa.scatter("x", "y", size="tamano", source=fuente_mapa, fill_alpha=0.75, line_color="white",
                              fill_color=factor_cmap("region", palette=list(COLORES.values()), factors=REGIONES))
    p_mapa.add_tools(HoverTool(renderers=[circulos], tooltips=[("Tienda", "@tienda"), ("Ventas", "@ventas{0,0} €")]))
    p_mapa.axis.visible = False

    # --- Dispersión enlazada con tabla
    p_disp = figure(height=360, sizing_mode="stretch_width", x_axis_type="log", y_axis_type="log",
                    title="Productos: precio frente a unidades (seleccione con lazo o rectángulo)",
                    tools="box_select,lasso_select,tap,wheel_zoom,reset", active_drag="box_select",
                    x_axis_label="Precio (€)", y_axis_label="Unidades vendidas")
    puntos = p_disp.scatter("precio", "unidades", source=fuente_productos, size=9, alpha=0.75,
                            color=factor_cmap("region", palette=list(COLORES.values()), factors=REGIONES),
                            selection_color="#C00000", nonselection_alpha=0.15, legend_field="region")
    p_disp.add_tools(HoverTool(renderers=[puntos], tooltips=[("Producto", "@producto"), ("Precio", "@precio{0.00} €"),
                                                             ("Unidades", "@unidades{0,0}"), ("Margen", "@margen{0.0%}")]))
    p_disp.legend.location = "top_right"

    tabla = DataTable(source=fuente_productos, height=360, sizing_mode="stretch_width", index_position=None,
                      columns=[TableColumn(field="producto", title="Producto"),
                               TableColumn(field="region", title="Región"),
                               TableColumn(field="precio", title="Precio (€)", formatter=NumberFormatter(format="0,0.00")),
                               TableColumn(field="unidades", title="Unidades", formatter=NumberFormatter(format="0,0")),
                               TableColumn(field="ingresos", title="Ingresos (€)", formatter=NumberFormatter(format="0,0")),
                               TableColumn(field="margen", title="Margen", formatter=NumberFormatter(format="0.0%"))])
    resumen_seleccion = Div(text="<i>Sin selección: se muestran todos los productos.</i>")

    # --- Controles
    selector = Select(title="Región", value="Todas", options=["Todas", *REGIONES], width=200)
    periodo = DateRangeSlider(title="Periodo", start=fechas.min(), end=fechas.max(),
                              value=(fechas.min(), fechas.max()), format="%d/%m/%Y",
                              sizing_mode="stretch_width")

    # --- Interactividad en JavaScript
    actualizar = CustomJS(
        args={"selector": selector, "periodo": periodo, "serie": fuente_serie, "diario": fuente_diario,
              "tiendas": fuente_tiendas, "p_tiendas": p_tiendas, "kpi": kpi, "media_anual": media_anual},
        code="""
        const region = selector.value;
        const [ini, fin] = periodo.value;
        const fmt = (v) => Math.round(v).toLocaleString("es-ES");

        // Serie temporal: cambiar la columna visible
        const s = serie.data;
        s.valor = s[region].slice();
        s.media = s[region + "_media"].slice();
        serie.change.emit();

        // Agregar ventas por tienda dentro del periodo y la región
        const d = diario.data;
        const total = {}, regionTienda = {};
        let suma = 0, dias = new Set();
        for (let i = 0; i < d.ts.length; i++) {
            if (d.ts[i] < ini || d.ts[i] > fin) continue;
            if (region !== "Todas" && d.region[i] !== region) continue;
            total[d.tienda[i]] = (total[d.tienda[i]] || 0) + d.ventas[i];
            regionTienda[d.tienda[i]] = d.region[i];
            suma += d.ventas[i];
            dias.add(d.ts[i]);
        }
        const orden = Object.keys(total).sort((a, b) => total[a] - total[b]);
        tiendas.data = {tienda: orden, ventas: orden.map(t => total[t]), region: orden.map(t => regionTienda[t])};
        p_tiendas.y_range.factors = orden;

        // Media diaria de referencia e indicadores
        const nDias = Math.max(dias.size, 1);
        media_anual.location = suma / nDias;
        kpi.text = `<div style="display:flex;gap:40px">
            <div><span style="color:#7F7F7F">VENTAS DEL PERIODO</span><br>
                 <b style="font-size:24px;color:#1F4E78">${fmt(suma)} €</b></div>
            <div><span style="color:#7F7F7F">MEDIA DIARIA</span><br>
                 <b style="font-size:24px;color:#1F4E78">${fmt(suma / nDias)} €</b></div>
            <div><span style="color:#7F7F7F">TIENDAS</span><br>
                 <b style="font-size:24px;color:#1F4E78">${orden.length}</b></div>
            <div><span style="color:#7F7F7F">DÍAS</span><br>
                 <b style="font-size:24px;color:#1F4E78">${dias.size}</b></div></div>`;
        """)
    selector.js_on_change("value", actualizar)
    periodo.js_on_change("value_throttled", actualizar)

    fuente_productos.selected.js_on_change("indices", CustomJS(
        args={"fuente": fuente_productos, "resumen": resumen_seleccion},
        code="""
        const idx = fuente.selected.indices;
        if (idx.length === 0) { resumen.text = "<i>Sin selección: se muestran todos los productos.</i>"; return; }
        const d = fuente.data;
        let ingresos = 0, unidades = 0;
        for (const i of idx) { ingresos += d.ingresos[i]; unidades += d.unidades[i]; }
        resumen.text = `<b>${idx.length}</b> productos seleccionados · ` +
            `<b>${Math.round(unidades).toLocaleString("es-ES")}</b> unidades · ` +
            `<b>${Math.round(ingresos).toLocaleString("es-ES")} €</b> de ingresos`;
        """))

    # --- Disposición
    cabecera = Div(text=f"""<div style="background:{AZUL};color:white;padding:14px 20px;border-radius:6px">
        <span style="font-size:22px;font-weight:bold">Panel de ventas 2025</span>
        <span style="opacity:.8;margin-left:12px">Datos simulados · Generado con Bokeh</span></div>""",
                   sizing_mode="stretch_width")
    panel = column(
        cabecera,
        row(selector, periodo, sizing_mode="stretch_width"),
        kpi,
        p_serie, p_nav,
        row(p_tiendas, p_mapa, sizing_mode="stretch_width"),
        row(p_disp, column(resumen_seleccion, tabla, sizing_mode="stretch_width"), sizing_mode="stretch_width"),
        sizing_mode="stretch_width", styles={"max-width": "1300px", "margin": "auto"},
    )
    # Valores iniciales de los indicadores
    total, dias = diario["ventas"].sum(), diario["fecha"].nunique()
    kpi.text = (f'<div style="display:flex;gap:40px">'
                + "".join(f'<div><span style="color:#7F7F7F">{t}</span><br>'
                          f'<b style="font-size:24px;color:{AZUL}">{v}</b></div>'
                          for t, v in [("VENTAS DEL PERIODO", f"{total:,.0f} €".replace(",", ".")),
                                       ("MEDIA DIARIA", f"{total / dias:,.0f} €".replace(",", ".")),
                                       ("TIENDAS", len(TIENDAS)), ("DÍAS", dias)])
                + "</div>")
    return panel


def main() -> None:
    SALIDA.mkdir(exist_ok=True)
    aplicar_tema()
    diario, productos = generar_datos()
    panel = construir_panel(diario, productos)
    ruta = SALIDA / "panel_ventas.html"
    output_file(ruta, title="Panel de ventas 2025")
    save(panel, resources=CDN)
    print(f"Panel generado: {ruta}")


if __name__ == "__main__":
    main()
```

Abra `panel_ventas.html` en el navegador: cambie la región o el periodo, arrastre el navegador de rango bajo la serie temporal y seleccione productos en la dispersión para filtrarlos en la tabla.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| El HTML se abre en blanco | Sin conexión y recursos desde la CDN. | `save(..., resources=INLINE)`. |
| `ValueError: expected an element of ... got 'circle'` / aviso de obsolescencia con `circle(size=...)` | API de marcadores cambiada en Bokeh 3.4+. | Use `p.scatter(..., marker="circle", size=...)`. |
| Las barras categóricas no aparecen | Falta `x_range` / `y_range` con las categorías. | `figure(x_range=lista_categorias)`. |
| `Columns need to be same length` | Columnas de distinta longitud en el `ColumnDataSource`. | Revise los datos antes de asignar `fuente.data`. |
| El *hover* muestra `???` | La columna no existe en la fuente o tiene espacios. | Revise el nombre; use `@{nombre con espacios}`. |
| Fechas como números enormes en el *hover* | Falta el formateador de fechas. | `formatters={"@fecha": "datetime"}` y `@fecha{%d/%m/%Y}`. |
| La leyenda no aparece con `legend_field` | El glifo no usa `source=`. | Pase un `ColumnDataSource` con esa columna. |
| `CustomJS` no hace nada | Error de JavaScript. | Abra la consola del navegador (F12) para ver el mensaje. |
| La selección no se enlaza entre gráficos | Usan fuentes distintas. | Comparta el mismo `ColumnDataSource`. |
| `RuntimeError: ... export_png requires Selenium` | Faltan Selenium y un navegador. | `pip install selenium` e instale Chrome o Firefox. |
| *Callbacks* Python (`on_change`) no se ejecutan | El documento se abrió como HTML estático. | Ejecute la app con `bokeh serve app.py`. |
| El mapa aparece en el océano | Coordenadas en grados en un eje Mercator. | Convierta lat/lon a Web Mercator. |
| El tema no se aplica | Se definió después de `save()`/`show()`. | Asigne `curdoc().theme` antes de guardar. |
| Gráfico lento con muchos puntos | Renderizado con Canvas. | `figure(output_backend="webgl")`. |

---

## 18. Buenas prácticas

- Trabaje siempre con **`ColumnDataSource`**: facilita *tooltips*, enlaces, filtros y actualizaciones.
- Comparta fuentes y rangos (`x_range=p1.x_range`) para crear gráficos enlazados.
- Agregue los datos en Python y envíe al navegador solo lo necesario.
- Decida pronto el modo: **HTML autónomo** con `CustomJS` para compartir; **`bokeh serve`** cuando la lógica requiera Python (bases de datos, modelos).
- Use `sizing_mode="stretch_width"` para paneles adaptables a la pantalla.
- Defina un **tema** (`Theme`) corporativo y aplíquelo a todos los documentos.
- Configure las herramientas según el gráfico: `xwheel_zoom` y `xpan` en series temporales, `box_select` en dispersiones.
- Formatee ejes y *tooltips* con unidades (`NumeralTickFormatter`, `{0,0 €}`).
- Mantenga el código JavaScript de `CustomJS` breve y comentado; si crece mucho, valore pasar a `bokeh serve`.
- Use `output_backend="webgl"` con decenas de miles de puntos.

---

## 19. Referencia rápida

```python
from bokeh.plotting import figure, show, save, output_file, curdoc
p = figure(width=, height=, x_range=, x_axis_type="datetime"|"log"|"mercator", tools=, sizing_mode=)

p.line / scatter(marker=) / vbar(x, top, width) / hbar(y, right, height) / quad / varea / step
p.vbar_stack / varea_stack / wedge / annular_wedge / rect / image / hex_tile / segment / patches / multi_line

fuente = ColumnDataSource(df);  p.line("x", "y", source=fuente)
fuente.data = {...} / fuente.stream({...}, rollover=) / fuente.patch({...})
CDSView(filter=GroupFilter(column_name=, group=))

factor_cmap(col, palette, factors) / linear_cmap(col, palette, low, high) / dodge(col, valor, range=) / cumsum(col)
p.yaxis.formatter = NumeralTickFormatter(format="0,0 €") / DatetimeTickFormatter(months="%b")
p.add_tools(HoverTool(tooltips=[("Nombre", "@col{0,0}")], formatters={"@fecha": "datetime"}, mode="vline"))
p.add_layout(Span / BoxAnnotation / Label / LabelSet / Whisker / Band / ColorBar)
p.legend.location / click_policy="hide";  p.add_tile("OpenStreetMap Mapnik")

row / column / gridplot([[p1, p2]]) / layout / Tabs(tabs=[TabPanel(child=, title=)]) / Div(text=)
Select / Slider / DateRangeSlider / MultiChoice / CheckboxGroup / DataTable(columns=[TableColumn(...)])
widget.js_on_change("value", CustomJS(args={...}, code="...")) / widget.js_link("value", glifo, "prop")
widget.on_change("value", f)  +  curdoc().add_root(...)  +  bokeh serve --show app.py
save(diseno, resources=CDN|INLINE) / export_png(p, filename=) / components(p)
```

---

## 20. Ejercicios propuestos

1. Cree un gráfico de líneas con tres series y una leyenda interactiva que permita ocultarlas (`click_policy="hide"`).
2. Construya dos dispersiones enlazadas con una misma fuente y compruebe que la selección se refleja en ambas.
3. Diseñe un histograma cuyo número de intervalos se controle con un `Slider` mediante `CustomJS`.
4. Desarrolle una aplicación `bokeh serve` que simule la lectura de un sensor cada segundo y muestre los últimos 300 valores.
5. Amplíe el ejemplo completo con una pestaña de mapa de calor (día de la semana × semana del año) que también responda al selector de región.

---

## 21. Referencias

- Documentación oficial: https://docs.bokeh.org/en/latest/
- Guía de usuario: https://docs.bokeh.org/en/latest/docs/user_guide.html
- Galería: https://docs.bokeh.org/en/latest/docs/gallery.html
- *Callbacks* en JavaScript: https://docs.bokeh.org/en/latest/docs/user_guide/interaction/js_callbacks.html
- Bokeh Server: https://docs.bokeh.org/en/latest/docs/user_guide/server.html
- Tutoriales interactivos: https://github.com/bokeh/tutorial
