# Manual de uso: Altair

> **Categoría:** Visualización de datos · **Utilidad:** Visualización declarativa de datos
> **Tipo:** Librería externa (requiere instalación con `pip`). Basada en Vega-Lite.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer gráfico](#4-primer-gráfico)
5. [Marcas (`mark_*`)](#5-marcas-mark_)
6. [Codificación (`encode`) y tipos de datos](#6-codificación-encode-y-tipos-de-datos)
7. [Escalas, ejes, leyendas y formato](#7-escalas-ejes-leyendas-y-formato)
8. [Transformaciones](#8-transformaciones)
9. [Composición: capas, concatenación, *facets* y repetición](#9-composición-capas-concatenación-facets-y-repetición)
10. [Interactividad: parámetros y selecciones](#10-interactividad-parámetros-y-selecciones)
11. [Controles: desplegables, deslizadores y casillas](#11-controles-desplegables-deslizadores-y-casillas)
12. [Mapas](#12-mapas)
13. [Temas y configuración](#13-temas-y-configuración)
14. [Datos grandes](#14-datos-grandes)
15. [Guardar y exportar](#15-guardar-y-exportar)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Altair** es una librería de **visualización declarativa**: en lugar de indicar *cómo* dibujar cada elemento, se describe *qué* se quiere representar (qué columna va en cada eje, qué define el color, cómo se agregan los datos) y Altair genera el gráfico. Produce especificaciones **Vega-Lite** (formato JSON) que se dibujan en el navegador.

Se basa en la **gramática de gráficos**: todo gráfico se compone de **datos + marca + codificación**.

**Ventajas:**
- Sintaxis concisa, coherente y fácil de leer.
- Interactividad potente con muy poco código: selecciones, filtros cruzados y gráficos enlazados.
- Composición sencilla: capas (`+`), lado a lado (`|`), uno sobre otro (`&`).
- Agregaciones y transformaciones integradas (`sum`, `mean`, `bin`, `timeUnit`, ventanas móviles…).
- Resultado portable: HTML, JSON, PNG, SVG o PDF.
- Excelente integración con Jupyter, Streamlit y Dash.

**Limitaciones:**
- Por defecto, los datos se incrustan en el gráfico: con más de 5000 filas lanza un error (configurable; véase la sección 14).
- Menos tipos de gráfico especializados que Plotly (sin 3D, Sankey ni *treemap* nativos).
- La personalización extrema (posiciones exactas, anotaciones complejas) es más limitada que en Matplotlib.

| Si necesita… | Opción |
|---|---|
| Exploración rápida e interactividad enlazada con poco código | **Altair** |
| Gran variedad de gráficos interactivos | Plotly |
| *Widgets* y aplicaciones con servidor | Bokeh / Dash / Streamlit |
| Gráficos estáticos con control total | Matplotlib |

---

## 2. Instalación e importación

```bash
pip install altair                    # Librería principal (incluye pandas y jsonschema)
pip install vl-convert-python         # Exportar a PNG, SVG y PDF
pip install "vegafusion[embed]"       # Opcional: datos grandes (véase la sección 14)
```

Comprobar la versión:

```bash
python -c "import altair as alt; print(alt.__version__)"
```

Importaciones habituales:

```python
import altair as alt
import pandas as pd
```

Dónde se muestran los gráficos:

| Entorno | Cómo |
|---|---|
| Jupyter / JupyterLab / VS Code | Automático: deje el gráfico al final de la celda. |
| Script | `grafico.save("grafico.html")` y abrir el archivo, o `grafico.show()`. |
| Streamlit | `st.altair_chart(grafico, use_container_width=True)` |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Chart` | Objeto principal: se crea a partir de un DataFrame (`alt.Chart(df)`). |
| Marca (*mark*) | Forma geométrica que representa cada dato: `mark_bar`, `mark_line`, `mark_point`… |
| Codificación (*encoding*) | Asociación entre columnas y **canales visuales**: `x`, `y`, `color`, `size`, `shape`, `tooltip`… |
| Tipo de dato | Cómo interpretar una columna: **Q** cuantitativo, **N** nominal, **O** ordinal, **T** temporal, **G** geográfico. |
| Transformación | Cálculo sobre los datos dentro del gráfico: filtrar, agregar, calcular, agrupar en intervalos. |
| Parámetro (`param`) | Valor variable (por ejemplo, la selección del usuario o un deslizador) que modifica el gráfico. |
| Composición | Combinar gráficos: capas (`+`), horizontal (`\|`), vertical (`&`), *facets*, repetición. |
| Especificación Vega-Lite | JSON que describe el gráfico. Altair la genera; el navegador la dibuja. |

**La gramática en una línea:**

```python
alt.Chart(datos).mark_bar().encode(x="region:N", y="sum(importe):Q", color="canal:N")
#        └ datos ┘ └ marca ┘ └──────────────── codificación ─────────────────────┘
```

Ver la especificación generada:

```python
print(grafico.to_json(indent=2))
```

---

## 4. Primer gráfico

```python
import altair as alt
import pandas as pd

ventas = pd.DataFrame({
    "mes": pd.date_range("2025-01-01", periods=6, freq="MS"),
    "importe": [120, 135, 128, 160, 172, 190],
})

grafico = (
    alt.Chart(ventas, title="Ventas del primer semestre")
    .mark_line(point=True)
    .encode(
        x=alt.X("mes:T", title="Mes", timeUnit="yearmonth"),
        y=alt.Y("importe:Q", title="Ventas (miles de €)"),
        tooltip=["mes:T", "importe:Q"],
    )
    .properties(width=600, height=300)
)
grafico.save("ventas.html")
```

Dos formas de escribir la codificación:

```python
.encode(x="region:N", y="importe:Q")                                    # Abreviada
.encode(x=alt.X("region:N", sort="-y", title="Región"),                  # Completa (permite opciones)
        y=alt.Y("importe:Q", axis=alt.Axis(format=",.0f")))
```

---

## 5. Marcas (`mark_*`)

| Marca | Uso |
|---|---|
| `mark_bar()` | Barras, histogramas, Gantt. |
| `mark_line()` | Líneas (series temporales). |
| `mark_area()` | Áreas, áreas apiladas, bandas. |
| `mark_point()`, `mark_circle()`, `mark_square()` | Dispersión. |
| `mark_tick()` | Marcas de guion (distribuciones compactas). |
| `mark_rect()` | Mapas de calor. |
| `mark_rule()` | Líneas de referencia, barras de error. |
| `mark_text()` | Etiquetas y valores. |
| `mark_arc()` | Sectores y anillos. |
| `mark_boxplot()`, `mark_errorbar()`, `mark_errorband()` | Estadísticos. |
| `mark_geoshape()` | Mapas. |
| `mark_image()` | Imágenes. |
| `mark_trail()` | Líneas de grosor variable. |

Propiedades fijas (no dependen de los datos) dentro de la marca:

```python
mark_bar(color="#1F4E78", opacity=0.8, cornerRadiusTopLeft=3, cornerRadiusTopRight=3)
mark_line(strokeWidth=2.5, strokeDash=[4, 2], interpolate="monotone", point=True)
mark_point(filled=True, size=80, shape="diamond")
mark_text(align="left", dx=4, fontSize=11, fontWeight="bold")
mark_arc(innerRadius=60, outerRadius=110)                # Anillo
mark_area(line=True, opacity=0.3)
```

### Ejemplos rápidos

```python
# Barras horizontales ordenadas
alt.Chart(df).mark_bar().encode(x="sum(importe):Q", y=alt.Y("producto:N", sort="-x"))

# Barras agrupadas (xOffset)
alt.Chart(df).mark_bar().encode(x="region:N", y="sum(importe):Q", color="anio:N", xOffset="anio:N")

# Áreas apiladas normalizadas (100 %)
alt.Chart(df).mark_area().encode(x="mes:T", y=alt.Y("importe:Q", stack="normalize"), color="canal:N")

# Histograma
alt.Chart(df).mark_bar().encode(x=alt.X("importe:Q", bin=alt.Bin(maxbins=30)), y="count():Q")

# Mapa de calor
alt.Chart(df).mark_rect().encode(x="hours(fecha):O", y="day(fecha):O", color="sum(importe):Q")

# Anillo
alt.Chart(df).mark_arc(innerRadius=50).encode(theta="sum(importe):Q", color="canal:N")

# Diagrama de caja
alt.Chart(df).mark_boxplot(extent="min-max").encode(x="region:N", y="importe:Q")

# Gantt
alt.Chart(tareas).mark_bar().encode(x="inicio:T", x2="fin:T", y="tarea:N", color="responsable:N")
```

---

## 6. Codificación (`encode`) y tipos de datos

### 6.1 Tipos de datos

| Código | Tipo | Ejemplo | Escala por defecto |
|---|---|---|---|
| `Q` | Cuantitativo | Importe, temperatura | Continua |
| `N` | Nominal | Región, producto | Categórica sin orden |
| `O` | Ordinal | Talla (S, M, L), nivel | Categórica ordenada |
| `T` | Temporal | Fecha | Tiempo |
| `G` | Geográfico | Geometría GeoJSON | — |

> Con DataFrames de pandas, Altair deduce el tipo, pero **indíquelo siempre** (`"importe:Q"`): evita sorpresas, sobre todo con años o códigos numéricos que en realidad son categorías.

### 6.2 Canales de codificación

| Canal | Uso |
|---|---|
| `x`, `y` | Posición. |
| `x2`, `y2` | Posición final (rangos, Gantt, bandas). |
| `xOffset`, `yOffset` | Desplazamiento dentro de una categoría (barras agrupadas). |
| `color`, `fill`, `stroke` | Color. |
| `opacity`, `size`, `shape`, `strokeDash` | Opacidad, tamaño, forma, tipo de línea. |
| `text` | Texto (con `mark_text`). |
| `tooltip` | Información emergente. |
| `theta`, `radius` | Ángulo y radio (sectores, radiales). |
| `row`, `column`, `facet` | Paneles por categoría. |
| `order` | Orden de apilado o de dibujo. |
| `detail` | Agrupar sin asignar un canal visual. |
| `href` | Enlace al hacer clic. |
| `latitude`, `longitude` | Posición geográfica. |

### 6.3 Agregaciones en la codificación

```python
y="sum(importe):Q"        # Suma
y="mean(importe):Q"       # Media
y="median(importe):Q"
y="count():Q"             # Número de filas
y="distinct(cliente):Q"   # Valores distintos
y="max(importe):Q" / "min(importe):Q" / "stdev(importe):Q" / "q1(importe):Q" / "q3(importe):Q"
```

### 6.4 Unidades de tiempo

```python
x="month(fecha):O"          # Ene, Feb… (agrupa meses de todos los años)
x="yearmonth(fecha):T"      # Ene 2025, Feb 2025…
x="day(fecha):O"            # Día de la semana
x="hours(fecha):O"          # Hora del día
x="yearquarter(fecha):O"    # Trimestre
x="week(fecha):O"           # Semana del año
```

### 6.5 *Tooltip*

```python
tooltip=[
    alt.Tooltip("producto:N", title="Producto"),
    alt.Tooltip("sum(importe):Q", title="Ventas", format=",.0f"),
    alt.Tooltip("fecha:T", title="Fecha", format="%d/%m/%Y"),
    alt.Tooltip("margen:Q", format=".1%"),
]
```

---

## 7. Escalas, ejes, leyendas y formato

### 7.1 Escalas

```python
alt.Y("importe:Q", scale=alt.Scale(domain=[0, 250]))                     # Rango fijo
alt.Y("importe:Q", scale=alt.Scale(zero=False))                          # No empezar en 0
alt.X("poblacion:Q", scale=alt.Scale(type="log"))                        # Logarítmica
alt.Color("margen:Q", scale=alt.Scale(scheme="blues"))                   # Esquema de color
alt.Color("variacion:Q", scale=alt.Scale(scheme="redblue", domainMid=0)) # Divergente
alt.Color("canal:N", scale=alt.Scale(domain=["Online", "Tienda"],
                                      range=["#2E75B6", "#C00000"]))      # Colores fijos
alt.Size("ventas:Q", scale=alt.Scale(range=[20, 600]))
```

Esquemas de color habituales: `"category10"`, `"tableau10"`, `"set2"`, `"blues"`, `"greens"`, `"viridis"`, `"redblue"`, `"redyellowgreen"`.

### 7.2 Ejes

```python
alt.X("fecha:T", axis=alt.Axis(format="%b %Y", labelAngle=-45, tickCount=6, grid=False, title=None))
alt.Y("importe:Q", axis=alt.Axis(format=",.0f", titleColor="#1F4E78", labelExpr="datum.label + ' €'"))
alt.Y("margen:Q", axis=alt.Axis(format="%"))
alt.X("region:N", sort=["Norte", "Sur", "Este", "Oeste"])        # Orden manual
alt.X("region:N", sort="-y")                                      # Por valor descendente
alt.Y("producto:N", sort=alt.EncodingSortField("importe", op="sum", order="descending"))
```

### 7.3 Leyendas

```python
alt.Color("canal:N", legend=alt.Legend(title="Canal", orient="top", direction="horizontal"))
alt.Color("canal:N", legend=None)                                 # Sin leyenda
```

### 7.4 Título y tamaño

```python
alt.Chart(df, title=alt.Title("Ventas 2025", subtitle="Datos provisionales", anchor="start"))
grafico.properties(width=600, height=300)
grafico.properties(width="container")                             # Ancho del contenedor (HTML, Streamlit)
```

### 7.5 Formato en español

```python
alt.renderers.set_embed_options(
    formatLocale={"decimal": ",", "thousands": ".", "grouping": [3], "currency": ["", " €"]},
    timeFormatLocale={
        "dateTime": "%A, %e de %B de %Y, %X", "date": "%d/%m/%Y", "time": "%H:%M:%S",
        "periods": ["AM", "PM"],
        "days": ["domingo", "lunes", "martes", "miércoles", "jueves", "viernes", "sábado"],
        "shortDays": ["dom", "lun", "mar", "mié", "jue", "vie", "sáb"],
        "months": ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio",
                   "agosto", "septiembre", "octubre", "noviembre", "diciembre"],
        "shortMonths": ["ene", "feb", "mar", "abr", "may", "jun", "jul", "ago", "sep", "oct", "nov", "dic"],
    },
)
```

> Esta configuración se aplica a la visualización en Jupyter. Al guardar en HTML, pásela con `grafico.save("g.html", embed_options={...})`.

---

## 8. Transformaciones

Las transformaciones se calculan **en el navegador**, dentro del gráfico. Para datos grandes, es preferible calcular en pandas antes.

```python
alt.Chart(df).transform_filter(alt.datum.importe > 100)                     # Filtrar
alt.Chart(df).transform_filter(alt.FieldOneOfPredicate(field="region", oneOf=["Norte", "Sur"]))
alt.Chart(df).transform_calculate(margen="datum.beneficio / datum.importe") # Nueva columna (expresión JS)
alt.Chart(df).transform_aggregate(total="sum(importe)", groupby=["region"])
alt.Chart(df).transform_bin("rango", "importe", bin=alt.Bin(maxbins=20))
alt.Chart(df).transform_timeunit(mes="yearmonth(fecha)")
alt.Chart(df).transform_fold(["online", "tienda"], as_=["canal", "importe"])  # Ancho → largo
alt.Chart(df).transform_joinaggregate(total_region="sum(importe)", groupby=["region"])
alt.Chart(df).transform_lookup(lookup="id", from_=alt.LookupData(otro_df, "id", ["nombre"]))
alt.Chart(df).transform_regression("precio", "unidades", method="linear")     # linear, log, exp, poly…
alt.Chart(df).transform_loess("fecha", "importe", bandwidth=0.3)
alt.Chart(df).transform_density("importe", groupby=["canal"], as_=["importe", "densidad"])
```

Ventanas (medias móviles, acumulados, *ranking*):

```python
alt.Chart(df).transform_window(
    media_7d="mean(importe)", frame=[-6, 0], sort=[{"field": "fecha"}]          # Media móvil 7 días
).transform_window(
    acumulado="sum(importe)", sort=[{"field": "fecha"}]                         # Suma acumulada
).transform_window(
    puesto="rank()", sort=[{"field": "importe", "order": "descending"}], groupby=["region"]
)
```

*Top N* por categoría:

```python
(alt.Chart(df)
 .transform_aggregate(total="sum(importe)", groupby=["producto"])
 .transform_window(puesto="rank()", sort=[{"field": "total", "order": "descending"}])
 .transform_filter(alt.datum.puesto <= 10)
 .mark_bar().encode(x="total:Q", y=alt.Y("producto:N", sort="-x")))
```

---

## 9. Composición: capas, concatenación, *facets* y repetición

### 9.1 Capas (`+` o `alt.layer`)

```python
base = alt.Chart(df).encode(x="mes:T")
barras = base.mark_bar(color="#9DC3E6").encode(y="sum(importe):Q")
linea = base.mark_line(color="#C00000").encode(y="mean(objetivo):Q")
etiquetas = base.mark_text(dy=-6).encode(y="sum(importe):Q", text=alt.Text("sum(importe):Q", format=",.0f"))
grafico = barras + linea + etiquetas
```

Línea de referencia:

```python
regla = alt.Chart(pd.DataFrame({"y": [150]})).mark_rule(strokeDash=[4, 4], color="gray").encode(y="y:Q")
grafico = barras + regla
```

Doble eje Y:

```python
alt.layer(barras, linea).resolve_scale(y="independent")
```

### 9.2 Concatenación (`|`, `&`)

```python
fila = grafico1 | grafico2                     # Horizontal (alt.hconcat)
columna = grafico1 & grafico2                  # Vertical (alt.vconcat)
panel = (g1 | g2) & g3
alt.concat(g1, g2, g3, g4, columns=2)          # Rejilla
```

### 9.3 *Facets* (paneles por categoría)

```python
alt.Chart(df).mark_line().encode(x="fecha:T", y="importe:Q").properties(width=200, height=120) \
    .facet(facet="region:N", columns=2)

alt.Chart(df).mark_bar().encode(x="mes:O", y="sum(importe):Q", row="canal:N", column="region:N")
```

### 9.4 Repetición

```python
alt.Chart(df).mark_point().encode(
    x=alt.X(alt.repeat("column"), type="quantitative"),
    y=alt.Y(alt.repeat("row"), type="quantitative"),
).properties(width=150, height=150).repeat(row=["precio", "unidades"], column=["margen", "descuento"])
```

### 9.5 Escalas compartidas o independientes

```python
(g1 | g2).resolve_scale(y="shared")             # Mismo eje Y (por defecto en facets)
(g1 | g2).resolve_scale(color="independent")    # Leyendas de color separadas
```

---

## 10. Interactividad: parámetros y selecciones

La interactividad se basa en **parámetros**: valores que cambian con la acción del usuario y se usan para resaltar, filtrar o modificar otros gráficos.

### 10.1 Zoom y desplazamiento

```python
grafico.interactive()                           # Zoom con rueda y desplazamiento con arrastre
grafico.interactive(bind_y=False)               # Solo en el eje X
```

### 10.2 Selección de puntos (`selection_point`)

```python
seleccion = alt.selection_point(fields=["region"])          # Clic (Mayús + clic para varios)

alt.Chart(df).mark_bar().encode(
    x="region:N", y="sum(importe):Q",
    color=alt.when(seleccion).then(alt.value("#1F4E78")).otherwise(alt.value("lightgray")),
).add_params(seleccion)
```

| Opción | Efecto |
|---|---|
| `fields=["col"]` | Selecciona todos los datos con el mismo valor en esa columna. |
| `on="pointerover"` | Selecciona al pasar el ratón (en vez de hacer clic). |
| `bind="legend"` | Selecciona haciendo clic en la leyenda. |
| `value=[{"region": "Norte"}]` | Valor inicial. |
| `empty=False` (en `alt.when`) | Sin selección, no se resalta nada. |

### 10.3 Selección de intervalo (`selection_interval`)

```python
pincel = alt.selection_interval(encodings=["x"])           # Arrastrar en horizontal

alt.Chart(df).mark_area().encode(x="fecha:T", y="sum(importe):Q").add_params(pincel)
```

### 10.4 Expresiones condicionales: `alt.when`

```python
color = alt.when(seleccion).then("canal:N").otherwise(alt.value("lightgray"))
opacidad = alt.when(seleccion).then(alt.value(1)).otherwise(alt.value(0.2))
tamano = alt.when(alt.datum.importe > 150).then(alt.value(120)).otherwise(alt.value(30))
```

> `alt.when(...).then(...).otherwise(...)` sustituye a la antigua `alt.condition(...)`, que sigue funcionando.

### 10.5 Filtro cruzado entre gráficos

```python
pincel = alt.selection_interval(encodings=["x"])

linea = (alt.Chart(df).mark_line()
         .encode(x="fecha:T", y="sum(importe):Q")
         .add_params(pincel)
         .properties(width=700, height=200))

barras = (alt.Chart(df).mark_bar()
          .encode(x="sum(importe):Q", y=alt.Y("region:N", sort="-x"))
          .transform_filter(pincel)                       # Solo los datos del periodo seleccionado
          .properties(width=700, height=150))

linea & barras
```

### 10.6 Resaltado al pasar el ratón con regla vertical

```python
cercano = alt.selection_point(nearest=True, on="pointerover", fields=["fecha"], empty=False)
base = alt.Chart(df).encode(x="fecha:T")
lineas = base.mark_line().encode(y="importe:Q", color="region:N")
puntos = lineas.mark_point().encode(opacity=alt.when(cercano).then(alt.value(1)).otherwise(alt.value(0)))
regla = base.mark_rule(color="gray").encode(
    opacity=alt.when(cercano).then(alt.value(0.5)).otherwise(alt.value(0)),
    tooltip=["fecha:T", "importe:Q", "region:N"],
).add_params(cercano)
lineas + puntos + regla
```

---

## 11. Controles: desplegables, deslizadores y casillas

Un parámetro se puede **vincular** a un control HTML con `bind`:

```python
# Desplegable que filtra por región
opciones = [None, "Norte", "Sur", "Este", "Oeste"]
desplegable = alt.binding_select(options=opciones, labels=["Todas", *opciones[1:]], name="Región: ")
sel_region = alt.selection_point(fields=["region"], bind=desplegable)

alt.Chart(df).mark_point().encode(x="precio:Q", y="unidades:Q").add_params(sel_region).transform_filter(sel_region)
```

```python
# Deslizador que modifica un umbral
umbral = alt.param(name="umbral", value=100,
                   bind=alt.binding_range(min=0, max=300, step=10, name="Umbral (€): "))

alt.Chart(df).mark_bar().encode(
    x="producto:N", y="importe:Q",
    color=alt.when(alt.datum.importe >= umbral).then(alt.value("#2E7D32")).otherwise(alt.value("#C00000")),
).add_params(umbral)
```

| Control | Función |
|---|---|
| `alt.binding_select(options=[...])` | Desplegable. |
| `alt.binding_radio(options=[...])` | Botones de opción. |
| `alt.binding_range(min, max, step)` | Deslizador. |
| `alt.binding_checkbox()` | Casilla de verificación. |
| `alt.binding(input="text")` | Campo de texto. |

Usar un parámetro en una transformación o en una expresión:

```python
alt.Chart(df).transform_filter(alt.datum.importe >= umbral)
alt.Chart(df).transform_calculate(ajustado=alt.datum.importe * factor)
alt.Chart(df).mark_point(size=alt.ExprRef(tamano_param.name))
```

---

## 12. Mapas

### 12.1 Puntos sobre un mapa base

```python
from vega_datasets import data                      # pip install vega_datasets (datos de ejemplo)
mundo = alt.topo_feature(data.world_110m.url, "countries")

fondo = alt.Chart(mundo).mark_geoshape(fill="#EEEEEE", stroke="white")
puntos = alt.Chart(tiendas).mark_circle().encode(
    longitude="lon:Q", latitude="lat:Q", size="ventas:Q", color="region:N", tooltip=["tienda", "ventas"])

(fondo + puntos).project(type="mercator", scale=2000, center=[-3.7, 40.2]).properties(width=600, height=450)
```

### 12.2 Coropletas desde un GeoJSON

```python
import json
with open("provincias.geojson", encoding="utf-8") as f:
    geo = json.load(f)

regiones = alt.Data(values=geo["features"])
alt.Chart(regiones).mark_geoshape(stroke="white").encode(
    color=alt.Color("ventas:Q", scale=alt.Scale(scheme="blues")),
    tooltip=["properties.nombre:N", "ventas:Q"],
).transform_lookup(
    lookup="properties.codigo", from_=alt.LookupData(df_ventas, "codigo", ["ventas"])
).project("mercator")
```

Con **GeoPandas**, el `GeoDataFrame` se pasa directamente: `alt.Chart(gdf).mark_geoshape()`.

Proyecciones: `"mercator"`, `"equalEarth"`, `"albersUsa"`, `"naturalEarth1"`, `"orthographic"`…

---

## 13. Temas y configuración

### 13.1 Temas incluidos

```python
print(alt.theme.names())             # dark, excel, fivethirtyeight, ggplot2, googlecharts, powerbi, quartz, vox…
alt.theme.enable("powerbi")
```

### 13.2 Tema corporativo propio

```python
@alt.theme.register("corporativo", enable=True)
def tema_corporativo():
    return alt.theme.ThemeConfig({
        "config": {
            "font": "Segoe UI",
            "view": {"stroke": None, "continuousWidth": 600, "continuousHeight": 300},
            "title": {"fontSize": 16, "color": "#1F4E78", "anchor": "start", "subtitleColor": "#7F7F7F"},
            "axis": {"labelColor": "#404040", "titleColor": "#404040", "gridColor": "#EEEEEE",
                     "domainColor": "#BFBFBF", "tickColor": "#BFBFBF"},
            "range": {"category": ["#1F4E78", "#2E75B6", "#9DC3E6", "#C00000", "#7F7F7F"]},
            "legend": {"orient": "top", "title": None},
            "bar": {"color": "#2E75B6"},
            "line": {"strokeWidth": 2.5},
        }
    })
```

### 13.3 Configuración de un gráfico concreto

```python
grafico.configure_axis(grid=False, labelFontSize=11) \
       .configure_view(strokeWidth=0) \
       .configure_title(fontSize=16, anchor="start") \
       .configure_legend(orient="bottom")
```

> Los métodos `configure_*` solo pueden aplicarse al gráfico **final** (el de nivel superior), no a los componentes antes de combinarlos.

---

## 14. Datos grandes

Por defecto, Altair incrusta todos los datos en la especificación y limita a **5000 filas** para no generar archivos enormes.

| Estrategia | Código |
|---|---|
| **Agregar antes en pandas** (recomendada) | `df.groupby(...).sum()` y graficar el resultado. |
| Usar VegaFusion (agrega en Python) | `pip install "vegafusion[embed]"` y `alt.data_transformers.enable("vegafusion")` |
| Quitar el límite | `alt.data_transformers.disable_max_rows()` (archivos HTML más pesados). |
| Guardar los datos aparte | `alt.data_transformers.enable("json")` (crea un .json junto al gráfico). |
| Pasar una URL | `alt.Chart("https://.../datos.csv")` (el navegador descarga los datos). |

> Con VegaFusion, las transformaciones (`transform_filter`, agregaciones…) se ejecutan en Python y solo se envían al navegador los datos ya resumidos.

---

## 15. Guardar y exportar

```python
grafico.save("grafico.html")                                 # Interactivo (requiere internet: usa la CDN)
grafico.save("grafico.html", inline=True)                    # Autónomo, sin internet (requiere vl-convert)
grafico.save("grafico.png", scale_factor=2)                  # Imagen (requiere vl-convert-python)
grafico.save("grafico.svg")
grafico.save("grafico.pdf")
grafico.save("grafico.json")                                 # Especificación Vega-Lite

especificacion = grafico.to_dict()                           # Diccionario
html = grafico.to_html()                                     # Texto HTML
```

> Las imágenes estáticas (PNG, SVG, PDF) pierden la interactividad: los controles y selecciones no aparecen.

Integración con otras herramientas:

| Entorno | Cómo |
|---|---|
| Streamlit | `st.altair_chart(grafico, use_container_width=True)` (las selecciones pueden devolverse a Python con `on_select`). |
| Dash | Componente `dash-vega-components` o `html.Iframe(srcDoc=grafico.to_html())`. |
| Web propia | Inserte `grafico.to_json()` con la librería JavaScript `vega-embed`. |
| Jupyter interactivo | `alt.JupyterChart(grafico)` permite leer las selecciones desde Python. |

---

## 16. Ejemplo práctico completo

**Explorador interactivo de ventas con filtros cruzados**: con datos diarios de ventas por tienda y categoría, construye un panel en el que un **pincel sobre la serie temporal** filtra el resto de gráficos, un **clic en una región** la resalta en todos ellos, un **desplegable** elige la categoría y un **deslizador** fija el objetivo diario. Incluye una media móvil, un *ranking* de tiendas, un mapa de calor día × mes y una dispersión de pedidos. Se aplica un tema corporativo y el resultado se guarda como HTML interactivo y como PNG estático.

```python
"""
Explorador interactivo de ventas con Altair.
Requiere: pip install altair pandas numpy vl-convert-python
"""
from pathlib import Path

import altair as alt
import numpy as np
import pandas as pd

SALIDA = Path(__file__).resolve().parent / "salida_altair"
REGIONES = ["Norte", "Centro", "Sur", "Este"]
COLORES = ["#1F4E78", "#2E75B6", "#C55A11", "#7F7F7F"]
TIENDAS = {"Norte": ["Bilbao", "Oviedo"], "Centro": ["Madrid", "Valladolid"],
           "Sur": ["Sevilla", "Málaga"], "Este": ["Valencia", "Barcelona"]}
CATEGORIAS = ["Electrónica", "Hogar", "Moda"]


# ------------------------------------------------------------------ Tema y formato
@alt.theme.register("corporativo", enable=True)
def tema_corporativo() -> alt.theme.ThemeConfig:
    return alt.theme.ThemeConfig({
        "config": {
            "font": "Segoe UI, Arial, sans-serif",
            "view": {"stroke": None},
            "title": {"fontSize": 13, "color": "#1F4E78", "anchor": "start", "fontWeight": "bold",
                      "subtitleColor": "#7F7F7F", "subtitleFontSize": 11},
            "axis": {"labelColor": "#404040", "titleColor": "#595959", "gridColor": "#EEEEEE",
                     "domainColor": "#BFBFBF", "tickColor": "#BFBFBF", "labelFontSize": 10,
                     "titleFontWeight": "normal"},
            "legend": {"labelFontSize": 10, "titleFontSize": 10, "orient": "right"},
            "range": {"category": COLORES},
            "concat": {"spacing": 25},
        }
    })


FORMATO_ES = {
    "formatLocale": {"decimal": ",", "thousands": ".", "grouping": [3], "currency": ["", " €"]},
    "timeFormatLocale": {
        "dateTime": "%A, %e de %B de %Y, %X", "date": "%d/%m/%Y", "time": "%H:%M:%S", "periods": ["AM", "PM"],
        "days": ["domingo", "lunes", "martes", "miércoles", "jueves", "viernes", "sábado"],
        "shortDays": ["dom", "lun", "mar", "mié", "jue", "vie", "sáb"],
        "months": ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto",
                   "septiembre", "octubre", "noviembre", "diciembre"],
        "shortMonths": ["ene", "feb", "mar", "abr", "may", "jun", "jul", "ago", "sep", "oct", "nov", "dic"],
    },
}


# ------------------------------------------------------------------ Datos
def generar_datos(semilla: int = 5) -> pd.DataFrame:
    """Ventas diarias por tienda y categoría (ya agregadas: ~8.800 filas)."""
    rng = np.random.default_rng(semilla)
    fechas = pd.date_range("2025-01-01", "2025-12-31", freq="D")
    filas = []
    for region, tiendas in TIENDAS.items():
        for tienda in tiendas:
            nivel = rng.uniform(0.6, 1.4)
            for categoria, peso, ticket in zip(CATEGORIAS, [1.4, 0.8, 1.0], [120, 55, 45]):
                estacional = 1 + 0.3 * np.sin(2 * np.pi * (fechas.dayofyear - (80 if categoria == "Moda" else 300)) / 365)
                semanal = np.where(fechas.dayofweek >= 5, 1.35, 1.0)
                ventas = 1_000 * nivel * peso * estacional * semanal * rng.normal(1, 0.15, len(fechas)).clip(0.3)
                filas.append(pd.DataFrame({"fecha": fechas, "region": region, "tienda": tienda,
                                           "categoria": categoria, "ventas": ventas.round(0),
                                           "pedidos": rng.poisson(ventas / (ticket * rng.uniform(0.85, 1.15)))}))
    return pd.concat(filas, ignore_index=True)


# ------------------------------------------------------------------ Panel
def construir_panel(df: pd.DataFrame) -> alt.VConcatChart:
    # --- Parámetros interactivos
    pincel = alt.selection_interval(encodings=["x"], name="periodo")
    sel_region = alt.selection_point(fields=["region"], bind="legend", name="region_sel")
    sel_categoria = alt.selection_point(
        fields=["categoria"], name="categoria_sel",
        bind=alt.binding_select(options=[None, *CATEGORIAS], labels=["Todas", *CATEGORIAS], name="Categoría: "))
    objetivo = alt.param(name="objetivo", value=30_000,
                         bind=alt.binding_range(min=15_000, max=45_000, step=1_000, name="Objetivo diario (€): "))

    base = alt.Chart(df).transform_filter(sel_categoria)
    color_region = alt.Color("region:N", title="Región (clic para filtrar)", sort=REGIONES)
    opacidad = alt.when(sel_region).then(alt.value(1)).otherwise(alt.value(0.15))

    # --- 1. Serie temporal con pincel, media móvil y objetivo
    diario = (base.transform_filter(sel_region)
              .transform_aggregate(ventas="sum(ventas)", groupby=["fecha"])
              .transform_window(media_14="mean(ventas)", frame=[-13, 0], sort=[{"field": "fecha"}]))
    area = diario.mark_area(color="#9DC3E6", opacity=0.5).encode(
        x=alt.X("fecha:T", title=None, axis=alt.Axis(format="%b", tickCount=12)),
        y=alt.Y("ventas:Q", title="Ventas diarias (€)", axis=alt.Axis(format="~s")),
    )
    media = diario.mark_line(color="#1F4E78", strokeWidth=2).encode(
        x="fecha:T", y="media_14:Q",
        tooltip=[alt.Tooltip("fecha:T", title="Fecha", format="%d/%m/%Y"),
                 alt.Tooltip("ventas:Q", title="Ventas", format=",.0f"),
                 alt.Tooltip("media_14:Q", title="Media 14 días", format=",.0f")])
    linea_objetivo = alt.Chart().mark_rule(color="#C00000", strokeDash=[5, 4]).encode(
        y=alt.datum(objetivo, type="quantitative"))
    serie = (alt.layer(area.add_params(pincel, sel_categoria, objetivo), media, linea_objetivo)
             .properties(width=820, height=200,
                         title=alt.Title("Evolución diaria de ventas",
                                         subtitle="Arrastre sobre el gráfico para seleccionar un periodo · "
                                                  "línea roja: objetivo diario")))

    # --- 2. Ventas por región (barras) — clic o leyenda
    por_region = base.transform_filter(pincel).mark_bar(cornerRadiusEnd=3).encode(
        x=alt.X("sum(ventas):Q", title="Ventas (€)", axis=alt.Axis(format="~s")),
        y=alt.Y("region:N", title=None, sort=REGIONES),
        color=color_region, opacity=opacidad,
        tooltip=[alt.Tooltip("region:N", title="Región"),
                 alt.Tooltip("sum(ventas):Q", title="Ventas", format=",.0f"),
                 alt.Tooltip("sum(pedidos):Q", title="Pedidos", format=",.0f")],
    ).add_params(sel_region).properties(width=360, height=170, title="Ventas por región")

    # --- 3. Ranking de tiendas
    tiendas = (base.transform_filter(pincel).transform_filter(sel_region)
               .mark_bar(cornerRadiusEnd=3).encode(
                   x=alt.X("sum(ventas):Q", title="Ventas (€)", axis=alt.Axis(format="~s")),
                   y=alt.Y("tienda:N", sort="-x", title=None),
                   color=color_region,
                   tooltip=[alt.Tooltip("tienda:N", title="Tienda"),
                            alt.Tooltip("sum(ventas):Q", title="Ventas", format=",.0f")])
               .properties(width=380, height=170, title="Ranking de tiendas"))

    # --- 4. Mapa de calor día de la semana × mes
    calor = (base.transform_filter(sel_region)
             .transform_filter(pincel)
             .mark_rect(cornerRadius=2).encode(
                 x=alt.X("month(fecha):O", title=None),
                 y=alt.Y("day(fecha):O", title=None),
                 color=alt.Color("mean(ventas):Q", title="Venta media (€)", scale=alt.Scale(scheme="blues"),
                                 legend=alt.Legend(orient="right", format="~s")),
                 tooltip=[alt.Tooltip("month(fecha):O", title="Mes"), alt.Tooltip("day(fecha):O", title="Día"),
                          alt.Tooltip("mean(ventas):Q", title="Venta media", format=",.0f")])
             .properties(width=360, height=170, title="Venta media por tienda: día de la semana × mes"))

    # --- 5. Pedidos frente a ticket medio, con recta de regresión
    puntos_base = (base.transform_filter(pincel)
                   .transform_aggregate(ventas="sum(ventas)", pedidos="sum(pedidos)",
                                        groupby=["tienda", "region", "categoria"])
                   .transform_calculate(ticket="datum.ventas / datum.pedidos"))
    puntos = puntos_base.mark_point(filled=True, size=140, stroke="white", strokeWidth=1).encode(
        x=alt.X("pedidos:Q", title="Pedidos", scale=alt.Scale(zero=False)),
        y=alt.Y("ticket:Q", title="Ticket medio (€)", scale=alt.Scale(zero=False)),
        color=color_region, opacity=opacidad,
        shape=alt.Shape("categoria:N", title="Categoría"),
        tooltip=["tienda:N", "categoria:N", alt.Tooltip("pedidos:Q", format=",.0f"),
                 alt.Tooltip("ticket:Q", title="Ticket medio", format=",.2f")])
    regresion = puntos_base.transform_regression("pedidos", "ticket").mark_line(
        color="#7F7F7F", strokeDash=[3, 3]).encode(x="pedidos:Q", y="ticket:Q")
    dispersion = (puntos + regresion).properties(width=380, height=170,
                                                 title="Pedidos frente a ticket medio (tienda × categoría)")

    # --- 6. Indicadores calculados según los filtros
    kpi_base = base.transform_filter(pincel).transform_filter(sel_region).transform_aggregate(
        total="sum(ventas)", pedidos="sum(pedidos)", dias="distinct(fecha)")
    kpis = alt.hconcat(*[
        kpi_base.transform_calculate(valor=expresion).mark_text(
            fontSize=24, fontWeight="bold", color="#1F4E78", align="left", x=0, y=26).encode(
            text=alt.Text("valor:Q", format=formato)).properties(
            width=190, height=40, title=alt.Title(titulo, fontSize=11, color="#7F7F7F", fontWeight="normal"))
        for titulo, expresion, formato in [
            ("VENTAS", "datum.total", ",.0f"),
            ("PEDIDOS", "datum.pedidos", ",.0f"),
            ("TICKET MEDIO (€)", "datum.total / datum.pedidos", ",.2f"),
            ("VENTA MEDIA DIARIA (€)", "datum.total / datum.dias", ",.0f"),
        ]], spacing=15)

    panel = alt.vconcat(
        kpis,
        serie,
        alt.hconcat(por_region, tiendas),
        alt.hconcat(calor, dispersion),
    ).properties(
        title=alt.Title("Explorador de ventas 2025", fontSize=20,
                        subtitle="Datos simulados · los filtros se aplican a todos los gráficos"),
    ).resolve_legend(color="shared", shape="independent").configure_view(stroke=None)
    return panel


def main() -> None:
    SALIDA.mkdir(exist_ok=True)
    df = generar_datos()
    alt.data_transformers.disable_max_rows()         # ~8.800 filas: ya están agregadas por día
    panel = construir_panel(df)
    panel.save(SALIDA / "explorador_ventas.html", embed_options=FORMATO_ES)
    print(f"Panel interactivo: {SALIDA / 'explorador_ventas.html'}")
    try:
        panel.save(SALIDA / "explorador_ventas.png", scale_factor=1.5)
        print("Imagen estática generada.")
    except Exception as error:
        print(f"PNG no generado ({error.__class__.__name__}): pip install vl-convert-python")


if __name__ == "__main__":
    main()
```

Abra `explorador_ventas.html`: arrastre sobre la serie temporal para elegir un periodo, haga clic en la leyenda de regiones, cambie la categoría con el desplegable y mueva el deslizador del objetivo.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `MaxRowsError: The number of rows in your dataset is greater than the maximum allowed (5000)` | Límite de filas por defecto. | Agregue en pandas, use VegaFusion o `alt.data_transformers.disable_max_rows()`. |
| El gráfico no se muestra en un script | Los scripts no tienen visor. | `grafico.save("g.html")` o `grafico.show()`. |
| El HTML guardado se ve en blanco sin internet | Carga Vega desde la CDN. | `grafico.save("g.html", inline=True)`. |
| `ValueError: ... vl-convert-python` al guardar PNG/SVG | Falta el conversor. | `pip install vl-convert-python`. |
| Ejes con fechas como números | Tipo no indicado o columna de texto. | Convierta con `pd.to_datetime` y use `:T`. |
| Los años (2024, 2025) aparecen como 2.024 | Tratados como cuantitativos. | Use `"anio:O"` o `"anio:N"`. |
| Barras ordenadas alfabéticamente | Orden por defecto. | `sort="-x"`, `sort="-y"` o una lista explícita. |
| `SchemaValidationError` | Opción no válida o en el sitio equivocado. | Lea el mensaje: indica la propiedad y las opciones permitidas. |
| `configure_*` falla en un gráfico compuesto | Se aplicó a un subgráfico. | Aplique `configure_*` solo al gráfico final. |
| `Javascript Error: Duplicate signal name` | Mismo parámetro añadido dos veces. | Añada cada `param`/selección con `add_params` una sola vez. |
| Una selección no filtra el otro gráfico | Falta `transform_filter(seleccion)` en el gráfico destino. | Añádalo al gráfico que debe reaccionar. |
| Nombres de columna con puntos o corchetes no funcionan | Vega los interpreta como rutas anidadas. | Renombre las columnas o escápelas con `\\.`. |
| Desaparece la leyenda en un panel compuesto | Las escalas de color compartidas combinan sus leyendas; `legend=None` en un gráfico la oculta para todos. | Quite `legend=None` o use `resolve_scale(color="independent")`. |
| `AltairDeprecationWarning` con `alt.themes` | API antigua. | Use `alt.theme.enable` / `@alt.theme.register`. |

---

## 18. Buenas prácticas

- Indique siempre el **tipo de dato** (`:Q`, `:N`, `:O`, `:T`).
- Prepare los datos en formato **largo** (una fila por observación) y agréguelos en pandas cuando sean grandes.
- Defina un gráfico **base** con los datos y codificaciones comunes y derive de él las capas.
- Use `alt.when(...)` para resaltar en lugar de eliminar: el contexto ayuda a interpretar.
- Combine pincel (`selection_interval`) y `transform_filter` para crear filtros cruzados entre gráficos.
- Registre un **tema** corporativo una sola vez en lugar de repetir `configure_*`.
- Configure el formato español (`formatLocale`, `timeFormatLocale`) al guardar en HTML.
- Ordene las categorías por valor (`sort="-x"`) salvo que tengan un orden natural.
- Añada *tooltips* con títulos y formato.
- Guarde en HTML para compartir la versión interactiva y en PNG/SVG para informes.

---

## 19. Referencia rápida

```python
import altair as alt
alt.Chart(df).mark_bar|line|area|point|circle|rect|rule|text|arc|boxplot|geoshape(...).encode(
    x=alt.X("col:Q|N|O|T", title=, sort=, scale=alt.Scale(...), axis=alt.Axis(format=)),
    y="sum(col):Q", color="col:N", size=, shape=, opacity=, text=, theta=, tooltip=[...],
    xOffset=, row=, column=,
).properties(width=, height=, title=)

transform_filter / calculate / aggregate / window / bin / timeunit / fold / lookup / regression / loess
Agregaciones: sum, mean, median, count(), distinct, min, max · Tiempo: yearmonth(), month(), day(), hours()

g1 + g2 (capas) / g1 | g2 (horizontal) / g1 & g2 (vertical) / .facet(...) / .repeat(...) / .resolve_scale(...)
sel = alt.selection_point(fields=[...], bind="legend") / alt.selection_interval(encodings=["x"])
p = alt.param(value=, bind=alt.binding_range(min=, max=) | binding_select(options=[...]))
alt.when(sel).then(...).otherwise(...) / .add_params(sel) / .transform_filter(sel) / .interactive()

alt.theme.enable("powerbi") / @alt.theme.register("mio", enable=True)
alt.data_transformers.enable("vegafusion") / disable_max_rows()
g.save("g.html" | "g.png" | "g.svg" | "g.json", embed_options={...}, scale_factor=2)
```

---

## 20. Ejercicios propuestos

1. Cree un gráfico de líneas con una línea por región y una regla vertical que muestre los valores de todas las regiones al pasar el ratón.
2. Represente un histograma de importes y una dispersión enlazados: al seleccionar un intervalo en el histograma, la dispersión muestra solo esos pedidos.
3. Construya un gráfico de barras de las 10 subcategorías con más ventas usando `transform_window` y `rank()`.
4. Diseñe un mapa de calor de correlaciones con etiquetas de texto en cada celda (capa de `mark_rect` y `mark_text`).
5. Amplíe el ejemplo completo con un deslizador que defina el número de días de la media móvil y una comparación interanual 2024–2025.

---

## 21. Referencias

- Documentación oficial: https://altair-viz.github.io/
- Galería de ejemplos: https://altair-viz.github.io/gallery/index.html
- Interactividad: https://altair-viz.github.io/user_guide/interactions.html
- Transformaciones: https://altair-viz.github.io/user_guide/transform/index.html
- Especificación Vega-Lite: https://vega.github.io/vega-lite/docs/
- Formatos numéricos (d3-format): https://d3js.org/d3-format
