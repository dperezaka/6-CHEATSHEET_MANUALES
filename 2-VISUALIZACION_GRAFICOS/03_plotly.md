# Manual de uso: Plotly

> **Categoría:** Visualización de datos · **Utilidad:** Gráficos interactivos
> **Tipo:** Librería externa (requiere instalación con `pip`)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer gráfico](#4-primer-gráfico)
5. [Plotly Express: gráficos básicos](#5-plotly-express-gráficos-básicos)
6. [Gráficos estadísticos y de distribución](#6-gráficos-estadísticos-y-de-distribución)
7. [Gráficos jerárquicos y de flujo](#7-gráficos-jerárquicos-y-de-flujo)
8. [Graph Objects: control total](#8-graph-objects-control-total)
9. [Diseño (*layout*), ejes y leyenda](#9-diseño-layout-ejes-y-leyenda)
10. [Información emergente (*hover*) y texto](#10-información-emergente-hover-y-texto)
11. [Varios gráficos en una figura (`make_subplots`)](#11-varios-gráficos-en-una-figura-make_subplots)
12. [Interactividad: botones, deslizadores y animaciones](#12-interactividad-botones-deslizadores-y-animaciones)
13. [Mapas](#13-mapas)
14. [Temas y plantillas](#14-temas-y-plantillas)
15. [Exportar: HTML, imagen e integración](#15-exportar-html-imagen-e-integración)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Plotly** es una librería para crear **gráficos interactivos** que se visualizan en el navegador: el usuario puede hacer zoom, desplazarse, ver el valor exacto de cada punto al pasar el ratón (*tooltip*), ocultar series desde la leyenda y descargar el gráfico como imagen. Internamente genera gráficos con **plotly.js**, una librería JavaScript, por lo que el resultado puede guardarse como un archivo HTML autónomo.

**Ventajas:**
- Interactividad completa sin escribir JavaScript.
- **Plotly Express** crea gráficos complejos con una línea a partir de un DataFrame.
- Más de 40 tipos de gráfico: financieros, mapas, 3D, jerárquicos (*treemap*, *sunburst*), Sankey…
- Exporta a HTML (para compartir) y a PNG, SVG o PDF.
- Base de **Dash**, el *framework* de paneles web de la misma empresa.
- Funciona en Jupyter, scripts, Streamlit y aplicaciones web.

**Limitaciones:**
- Los archivos HTML pesan unos 3–4 MB si incluyen la librería (se reduce usando la CDN).
- Con cientos de miles de puntos el navegador se ralentiza (use `Scattergl` o agregue los datos).
- La exportación a imagen requiere el paquete `kaleido` y un navegador Chrome/Chromium.

| Si necesita… | Opción |
|---|---|
| Gráficos interactivos para compartir o explorar | **Plotly** |
| Gráficos estáticos para impresión | Matplotlib / Seaborn |
| Interactividad enlazada entre gráficos en el navegador con servidor propio | Bokeh |
| Gramática declarativa y selecciones enlazadas | Altair |
| Panel web completo con filtros y lógica en Python | Dash (usa Plotly) / Streamlit |

---

## 2. Instalación e importación

```bash
pip install plotly                 # Librería principal
pip install pandas                 # Recomendado: Plotly Express trabaja con DataFrames
pip install kaleido                # Exportar a PNG, SVG, PDF
pip install nbformat               # Mostrar gráficos en Jupyter
```

> `kaleido` necesita Chrome o Chromium instalado. Si no lo tiene, ejecute `plotly_get_chrome` en la terminal para descargar una copia.

Importaciones habituales:

```python
import plotly.express as px                    # Interfaz rápida
import plotly.graph_objects as go              # Interfaz detallada
from plotly.subplots import make_subplots      # Varios gráficos
import plotly.io as pio                        # Configuración, temas y exportación
```

Elegir dónde se muestran los gráficos:

```python
pio.renderers.default = "browser"     # Abre en el navegador (scripts)
pio.renderers.default = "notebook"    # Jupyter clásico
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Figura (`go.Figure`) | Objeto que contiene todo el gráfico: datos (`data`) y diseño (`layout`). |
| Traza (*trace*) | Una serie de datos con su tipo de representación: `go.Scatter`, `go.Bar`, `go.Pie`… Una figura puede tener varias. |
| Diseño (`layout`) | Todo lo que no son datos: título, ejes, leyenda, márgenes, colores de fondo, anotaciones. |
| Plotly Express (`px`) | Funciones de alto nivel que reciben un DataFrame y devuelven un `go.Figure` ya configurado. |
| Graph Objects (`go`) | Clases de bajo nivel para construir la figura traza a traza. |
| *Hover* | Información que aparece al pasar el ratón por un punto. |
| Plantilla (*template*) | Tema visual reutilizable (colores, fuentes, fondo). |
| *Facet* | División automática en paneles por una variable categórica (`facet_col`, `facet_row`). |

**Estructura de una figura** (es un diccionario anidado):

```python
fig = px.bar(x=["A", "B"], y=[3, 5])
print(fig)              # data=[Bar(...)], layout=Layout(...)
fig.to_dict()           # Diccionario completo
fig.data[0]             # Primera traza
fig.layout.title        # Título
```

**¿Express o Graph Objects?**

| Plotly Express (`px`) | Graph Objects (`go`) |
|---|---|
| Una línea por gráfico desde un DataFrame. | Control total, traza a traza. |
| Colores, leyendas y *facets* automáticos. | Necesario para tipos no disponibles en `px` (velas, Sankey, indicadores, tablas). |
| Recomendado como **punto de partida**. | Recomendado para **ajustes finos** y figuras compuestas. |

> Ambos se combinan: cree la figura con `px` y ajústela con los métodos `update_*` o añádale trazas `go`.

---

## 4. Primer gráfico

```python
import pandas as pd
import plotly.express as px

ventas = pd.DataFrame({
    "mes": ["Ene", "Feb", "Mar", "Abr", "May", "Jun"],
    "importe": [120, 135, 128, 160, 172, 190],
})

fig = px.line(ventas, x="mes", y="importe", markers=True,
              title="Ventas del primer semestre",
              labels={"mes": "Mes", "importe": "Ventas (miles de €)"})
fig.show()                               # Abre el gráfico interactivo
fig.write_html("ventas.html")            # Guarda un archivo para compartir
```

Barra de herramientas del gráfico (esquina superior derecha): descargar PNG, zoom, desplazar, selección, restablecer ejes.

---

## 5. Plotly Express: gráficos básicos

Todas las funciones de `px` comparten parámetros: `data_frame`, `x`, `y`, `color`, `symbol`, `size`, `facet_col`, `facet_row`, `hover_name`, `hover_data`, `text`, `labels`, `title`, `template`, `width`, `height`.

### 5.1 Líneas y áreas

```python
px.line(df, x="fecha", y="importe", color="region", markers=True)
px.line(df, x="fecha", y=["online", "tienda"])                    # Varias columnas (formato ancho)
px.area(df, x="fecha", y="importe", color="canal")                 # Áreas apiladas
```

### 5.2 Barras

```python
px.bar(df, x="region", y="importe", color="canal")                          # Apiladas
px.bar(df, x="region", y="importe", color="canal", barmode="group")          # Agrupadas
px.bar(df, x="importe", y="producto", orientation="h", text_auto=".2s")      # Horizontales con valor
px.bar(df, x="region", y="importe", color="canal", barmode="relative",
       pattern_shape="canal")                                                # Con tramas
```

| `barmode` | Resultado |
|---|---|
| `"relative"` | Apiladas (por defecto); los negativos van hacia abajo. |
| `"group"` | Agrupadas una junto a otra. |
| `"overlay"` | Superpuestas (use `opacity`). |

### 5.3 Dispersión y burbujas

```python
px.scatter(df, x="precio", y="unidades", color="categoria", size="margen",
           hover_name="producto", size_max=40, opacity=0.8)
px.scatter(df, x="precio", y="unidades", trendline="ols")         # Recta de regresión (requiere statsmodels)
px.scatter(df, x="precio", y="unidades", log_x=True, marginal_y="histogram")
```

### 5.4 Sectores y anillos

```python
px.pie(df, names="canal", values="importe", hole=0.45)            # hole > 0 → anillo
```

### 5.5 Mapa de calor e imágenes

```python
px.imshow(tabla, text_auto=".0f", color_continuous_scale="Blues", aspect="auto")
px.density_heatmap(df, x="hora", y="dia", z="importe", histfunc="sum")
```

### 5.6 Paneles por categoría (*facets*)

```python
px.scatter(df, x="precio", y="unidades", color="canal",
           facet_col="region", facet_col_wrap=2, height=600)
fig.for_each_annotation(lambda a: a.update(text=a.text.split("=")[-1]))   # Quitar "region="
```

### 5.7 Colores

```python
px.bar(df, x="region", y="importe", color="canal",
       color_discrete_map={"Online": "#2E75B6", "Tienda": "#C00000"})              # Categóricos fijos
px.bar(df, ..., color_discrete_sequence=px.colors.qualitative.Safe)               # Paleta categórica
px.scatter(df, ..., color="margen", color_continuous_scale="Viridis")             # Escala continua
px.bar(df, ..., category_orders={"region": ["Norte", "Sur", "Este", "Oeste"]})    # Orden
```

Paletas disponibles: `px.colors.qualitative` (`Plotly`, `D3`, `Set2`, `Safe`, `Bold`…), `px.colors.sequential` (`Blues`, `Viridis`…), `px.colors.diverging` (`RdBu`, `Tealrose`…).

---

## 6. Gráficos estadísticos y de distribución

```python
px.histogram(df, x="importe", nbins=40, color="canal", barmode="overlay", opacity=0.6)
px.histogram(df, x="region", y="importe", histfunc="sum")              # Agregación (suma)
px.histogram(df, x="importe", marginal="box", histnorm="percent")      # Con diagrama de caja marginal
px.box(df, x="region", y="importe", color="canal", points="outliers")
px.violin(df, x="region", y="importe", box=True, points="all")
px.strip(df, x="region", y="importe")
px.ecdf(df, x="importe", color="canal")
px.scatter_matrix(df, dimensions=["precio", "unidades", "margen"], color="canal")
px.parallel_coordinates(df, dimensions=["precio", "unidades", "margen"], color="margen")
```

---

## 7. Gráficos jerárquicos y de flujo

### 7.1 Treemap y sunburst

```python
px.treemap(df, path=[px.Constant("Total"), "region", "categoria", "producto"],
           values="importe", color="margen", color_continuous_scale="RdYlGn")
px.sunburst(df, path=["region", "categoria"], values="importe")
px.icicle(df, path=["region", "categoria"], values="importe")
```

### 7.2 Embudo (*funnel*)

```python
etapas = pd.DataFrame({"etapa": ["Visitas", "Carrito", "Pago", "Compra"],
                       "usuarios": [10_000, 2_400, 900, 720]})
px.funnel(etapas, x="usuarios", y="etapa")
```

### 7.3 Diagrama de Gantt (`timeline`)

```python
tareas = pd.DataFrame({
    "tarea": ["Análisis", "Diseño", "Desarrollo", "Pruebas"],
    "inicio": ["2025-01-06", "2025-01-20", "2025-02-03", "2025-03-10"],
    "fin": ["2025-01-24", "2025-02-07", "2025-03-14", "2025-03-28"],
    "responsable": ["Ana", "Luis", "Ana", "Marta"],
})
fig = px.timeline(tareas, x_start="inicio", x_end="fin", y="tarea", color="responsable")
fig.update_yaxes(autorange="reversed")            # Primera tarea arriba
```

### 7.4 Sankey (solo `go`)

```python
fig = go.Figure(go.Sankey(
    node={"label": ["Web", "App", "Compra", "Abandono"], "pad": 20},
    link={"source": [0, 0, 1, 1], "target": [2, 3, 2, 3], "value": [300, 700, 250, 250]},
))
```

---

## 8. Graph Objects: control total

### 8.1 Construir una figura traza a traza

```python
import plotly.graph_objects as go

fig = go.Figure()
fig.add_trace(go.Bar(x=meses, y=ventas, name="Ventas", marker_color="#9DC3E6"))
fig.add_trace(go.Scatter(x=meses, y=objetivo, name="Objetivo", mode="lines",
                         line={"color": "#C00000", "dash": "dash", "width": 2}))
fig.update_layout(title="Ventas frente a objetivo", yaxis_title="€")
fig.show()
```

| Traza | Uso |
|---|---|
| `go.Scatter` | Líneas (`mode="lines"`), puntos (`"markers"`), ambos (`"lines+markers"`), texto. |
| `go.Scattergl` | Igual que `Scatter`, acelerado por GPU (>100 000 puntos). |
| `go.Bar`, `go.Histogram`, `go.Box`, `go.Violin` | Barras y distribuciones. |
| `go.Pie`, `go.Heatmap`, `go.Contour` | Sectores, mapas de calor, contornos. |
| `go.Candlestick`, `go.Ohlc` | Gráficos financieros. |
| `go.Indicator` | Indicadores KPI, velocímetros (*gauge*). |
| `go.Table` | Tabla de datos. |
| `go.Waterfall` | Cascada (puente de resultados). |
| `go.Sankey` | Flujos. |

### 8.2 Modificar una figura existente

```python
fig.update_traces(marker_line_width=0, opacity=0.9)                       # Todas las trazas
fig.update_traces(line_width=3, selector={"name": "Objetivo"})            # Solo una
fig.update_layout(title_text="Nuevo título", height=500)
fig.update_xaxes(tickangle=-45)
fig.update_yaxes(tickprefix="€ ", separatethousands=True)
fig.for_each_trace(lambda t: t.update(name=t.name.upper()))
```

> Notación con guion bajo: `marker_color="red"` equivale a `marker={"color": "red"}`.

### 8.3 Ejemplos con `go`

Indicadores KPI:

```python
fig = go.Figure(go.Indicator(
    mode="number+delta", value=1_250_000,
    number={"prefix": "€", "valueformat": ",.0f"},
    delta={"reference": 1_100_000, "relative": True, "valueformat": ".1%"},
    title={"text": "Ventas anuales"},
))
```

Velas japonesas:

```python
fig = go.Figure(go.Candlestick(x=df["fecha"], open=df["apertura"], high=df["max"],
                               low=df["min"], close=df["cierre"]))
fig.update_layout(xaxis_rangeslider_visible=False)
```

Cascada:

```python
fig = go.Figure(go.Waterfall(
    x=["Ingresos", "Coste ventas", "Gastos", "Impuestos", "Beneficio"],
    y=[1000, -450, -300, -60, 0],
    measure=["absolute", "relative", "relative", "relative", "total"],
))
```

---

## 9. Diseño (*layout*), ejes y leyenda

### 9.1 Títulos, tamaño y márgenes

```python
fig.update_layout(
    title={"text": "Ventas 2025<br><sup>Datos provisionales</sup>", "x": 0.02},
    width=900, height=500,
    margin={"l": 40, "r": 20, "t": 80, "b": 40},
    font={"family": "Segoe UI, Arial", "size": 12, "color": "#262626"},
    plot_bgcolor="white", paper_bgcolor="white",
)
```

### 9.2 Ejes

```python
fig.update_xaxes(title_text="Mes", showgrid=False, tickangle=-30)
fig.update_yaxes(title_text="Importe", rangemode="tozero", gridcolor="#EEEEEE",
                 ticksuffix=" €", tickformat=",.0f")
fig.update_yaxes(type="log")                       # Escala logarítmica
fig.update_xaxes(range=[0, 100])                   # Rango fijo
fig.update_xaxes(categoryorder="total descending") # Ordenar categorías por valor
```

Formatos de número (sintaxis d3-format):

| `tickformat` | Ejemplo |
|---|---|
| `",.0f"` | 12,500 |
| `".1%"` | 25.3 % |
| `".2s"` | 1.2M |
| `"%d/%m/%Y"` (fechas) | 15/03/2025 |
| `"%b %Y"` (fechas) | Mar 2025 |

Separadores españoles (punto de miles y coma decimal):

```python
fig.update_layout(separators=",.")       # Primero el decimal, después el de miles
```

### 9.3 Doble eje Y

```python
fig = make_subplots(specs=[[{"secondary_y": True}]])
fig.add_trace(go.Bar(x=meses, y=ventas, name="Ventas"), secondary_y=False)
fig.add_trace(go.Scatter(x=meses, y=margen, name="Margen %"), secondary_y=True)
fig.update_yaxes(title_text="€", secondary_y=False)
fig.update_yaxes(title_text="%", secondary_y=True)
```

### 9.4 Leyenda

```python
fig.update_layout(legend={"orientation": "h", "yanchor": "bottom", "y": 1.02, "x": 0,
                          "title": None, "bgcolor": "rgba(0,0,0,0)"})
fig.update_layout(showlegend=False)
fig.update_traces(showlegend=False, selector={"name": "Objetivo"})
```

### 9.5 Líneas, zonas y anotaciones

```python
fig.add_hline(y=150, line_dash="dash", line_color="gray", annotation_text="Objetivo")
fig.add_vline(x="2025-03-15", line_dash="dot")
fig.add_vrect(x0="2025-07-01", x1="2025-08-31", fillcolor="orange", opacity=0.1,
              annotation_text="Verano", line_width=0)
fig.add_annotation(x="May", y=190, text="Máximo", showarrow=True, arrowhead=2)
fig.add_shape(type="rect", x0=0, y0=0, x1=1, y1=1, xref="paper", yref="paper", line_color="#DDD")
```

> `xref="paper"` / `yref="paper"` usan coordenadas relativas a la figura (0 a 1) en lugar de las de los datos.

---

## 10. Información emergente (*hover*) y texto

```python
px.scatter(df, x="precio", y="unidades", hover_name="producto",
           hover_data={"margen": ":.1%", "precio": ":.2f", "categoria": True, "id": False})
```

Plantilla personalizada:

```python
fig.update_traces(
    hovertemplate="<b>%{x}</b><br>Ventas: %{y:,.0f} €<br>Cliente: %{customdata[0]}<extra></extra>",
    customdata=df[["cliente"]],
)
fig.update_layout(hovermode="x unified")        # Un solo cuadro con todas las series
```

| Variable | Significado |
|---|---|
| `%{x}`, `%{y}` | Valores del punto |
| `%{text}` | Texto asignado |
| `%{customdata[i]}` | Columna adicional |
| `%{y:,.2f}` | Valor con formato |
| `<extra></extra>` | Oculta el nombre de la traza en el cuadro |

Texto sobre los elementos:

```python
px.bar(df, x="region", y="importe", text_auto=".3s")
fig.update_traces(texttemplate="%{y:,.0f} €", textposition="outside")
```

| `hovermode` | Comportamiento |
|---|---|
| `"closest"` | El punto más cercano (defecto). |
| `"x"` / `"x unified"` | Todos los puntos con la misma X (separados / en un cuadro). |
| `False` | Sin *hover*. |

---

## 11. Varios gráficos en una figura (`make_subplots`)

```python
from plotly.subplots import make_subplots

fig = make_subplots(
    rows=2, cols=2,
    subplot_titles=("Evolución", "Por región", "Distribución", "Canal"),
    specs=[[{"colspan": 2}, None],
           [{"type": "xy"}, {"type": "domain"}]],        # "domain" para sectores
    vertical_spacing=0.12, horizontal_spacing=0.08,
)
fig.add_trace(go.Scatter(x=meses, y=ventas, name="Ventas"), row=1, col=1)
fig.add_trace(go.Histogram(x=importes, name="Tickets"), row=2, col=1)
fig.add_trace(go.Pie(labels=canales, values=valores, hole=0.5), row=2, col=2)
fig.update_layout(height=700, showlegend=False)
fig.update_yaxes(title_text="€", row=1, col=1)
```

| `type` en `specs` | Trazas |
|---|---|
| `"xy"` | Scatter, Bar, Histogram, Box… (por defecto) |
| `"domain"` | Pie, Sunburst, Treemap, Indicator, Table |
| `"geo"` / `"map"` | Mapas |
| `"polar"`, `"scene"` (3D) | Polares y 3D |

Reutilizar trazas de Plotly Express en `make_subplots`:

```python
fig_px = px.bar(df, x="region", y="importe", color="canal")
for traza in fig_px.data:
    fig.add_trace(traza, row=1, col=2)
```

---

## 12. Interactividad: botones, deslizadores y animaciones

### 12.1 Selector de rango en fechas

```python
fig.update_xaxes(
    rangeslider_visible=True,
    rangeselector={"buttons": [
        {"count": 1, "label": "1M", "step": "month", "stepmode": "backward"},
        {"count": 6, "label": "6M", "step": "month", "stepmode": "backward"},
        {"count": 1, "label": "Año", "step": "year", "stepmode": "todate"},
        {"step": "all", "label": "Todo"},
    ]},
)
```

### 12.2 Menú desplegable para mostrar u ocultar series

```python
regiones = ["Norte", "Sur", "Este"]
for r in regiones:
    fig.add_trace(go.Scatter(x=fechas, y=datos[r], name=r))

botones = [{"label": "Todas", "method": "update", "args": [{"visible": [True] * len(regiones)}]}]
for i, r in enumerate(regiones):
    visibles = [j == i for j in range(len(regiones))]
    botones.append({"label": r, "method": "update",
                    "args": [{"visible": visibles}, {"title": f"Ventas · {r}"}]})

fig.update_layout(updatemenus=[{"buttons": botones, "x": 1, "y": 1.15, "xanchor": "right"}])
```

| `method` | Modifica |
|---|---|
| `"restyle"` | Propiedades de las trazas (datos, visibilidad, color). |
| `"relayout"` | Propiedades del diseño (título, ejes). |
| `"update"` | Ambas a la vez. |
| `"animate"` | Lanza fotogramas de una animación. |

### 12.3 Animaciones con Plotly Express

```python
px.scatter(df, x="pib_per_capita", y="esperanza_vida", size="poblacion", color="continente",
           animation_frame="anio", animation_group="pais", log_x=True,
           range_x=[100, 100_000], range_y=[25, 90])
px.bar(df, x="region", y="importe", animation_frame="mes", range_y=[0, 200])
```

> Fije siempre `range_x` / `range_y` en las animaciones; si no, los ejes cambian en cada fotograma y la comparación pierde sentido.

---

## 13. Mapas

### 13.1 Puntos sobre un mapa base (`scatter_map`)

Usa mapas de teselas de MapLibre (OpenStreetMap y similares) sin necesidad de clave:

```python
tiendas = pd.DataFrame({
    "tienda": ["Madrid", "Barcelona", "Sevilla", "Valencia"],
    "lat": [40.4168, 41.3874, 37.3891, 39.4699],
    "lon": [-3.7038, 2.1686, -5.9845, -0.3763],
    "ventas": [520, 480, 210, 260],
})
fig = px.scatter_map(tiendas, lat="lat", lon="lon", size="ventas", color="ventas",
                     hover_name="tienda", zoom=4.5, center={"lat": 40, "lon": -3.5},
                     map_style="carto-positron", size_max=35)
```

Estilos de mapa: `"open-street-map"`, `"carto-positron"`, `"carto-darkmatter"`, `"basic"`, `"streets"`, `"outdoors"`, `"satellite"`…

### 13.2 Mapas de coropletas (regiones coloreadas)

```python
import json
with open("provincias.geojson", encoding="utf-8") as f:
    geo = json.load(f)

px.choropleth_map(df, geojson=geo, locations="codigo", featureidkey="properties.cod_prov",
                  color="ventas", color_continuous_scale="Blues",
                  map_style="carto-positron", zoom=4.5, center={"lat": 40, "lon": -3.5})
```

Países del mundo (sin GeoJSON, con códigos ISO-3):

```python
px.choropleth(df, locations="iso3", color="ventas", hover_name="pais", projection="natural earth")
```

### 13.3 Densidad

```python
px.density_map(df, lat="lat", lon="lon", z="importe", radius=15, zoom=5, map_style="carto-positron")
```

> En versiones recientes las funciones `*_mapbox` se han sustituido por `scatter_map`, `choropleth_map`, `density_map` y `line_map`, que no requieren *token* de Mapbox.

---

## 14. Temas y plantillas

```python
import plotly.io as pio

print(list(pio.templates))           # plotly, plotly_white, plotly_dark, simple_white, ggplot2, seaborn, presentation, none
pio.templates.default = "plotly_white"
px.bar(df, ..., template="simple_white")             # Solo para un gráfico
```

Plantilla corporativa propia:

```python
pio.templates["corporativo"] = go.layout.Template(
    layout={
        "font": {"family": "Segoe UI, Arial", "size": 12, "color": "#262626"},
        "colorway": ["#1F4E78", "#2E75B6", "#9DC3E6", "#C00000", "#7F7F7F"],
        "title": {"font": {"size": 18, "color": "#1F4E78"}, "x": 0.02},
        "plot_bgcolor": "white",
        "xaxis": {"showgrid": False, "linecolor": "#BFBFBF"},
        "yaxis": {"gridcolor": "#EEEEEE", "zeroline": False},
        "hoverlabel": {"font_family": "Segoe UI, Arial"},
        "separators": ",.",
    }
)
pio.templates.default = "plotly_white+corporativo"      # Combinar plantillas con "+"
```

---

## 15. Exportar: HTML, imagen e integración

### 15.1 HTML interactivo

```python
fig.write_html("grafico.html")                                    # Autónomo (~3,5 MB, funciona sin internet)
fig.write_html("grafico.html", include_plotlyjs="cdn")            # Ligero (~10 KB, requiere internet)
fig.write_html("grafico.html", config={"displaylogo": False, "locale": "es"})
html = fig.to_html(full_html=False, include_plotlyjs=False)       # Fragmento <div> para insertar en una web
```

Varios gráficos en un solo HTML:

```python
with open("informe.html", "w", encoding="utf-8") as f:
    f.write("<html><head><meta charset='utf-8'></head><body>")
    for i, figura in enumerate([fig1, fig2, fig3]):
        f.write(figura.to_html(full_html=False, include_plotlyjs="cdn" if i == 0 else False))
    f.write("</body></html>")
```

### 15.2 Imagen estática (requiere `kaleido`)

```python
fig.write_image("grafico.png", width=1200, height=600, scale=2)   # scale=2 → doble resolución
fig.write_image("grafico.svg")
fig.write_image("grafico.pdf")
bytes_png = fig.to_image(format="png")
```

### 15.3 Configuración de la barra de herramientas

```python
fig.show(config={
    "displayModeBar": True,
    "displaylogo": False,
    "modeBarButtonsToRemove": ["lasso2d", "select2d"],
    "toImageButtonOptions": {"format": "png", "filename": "ventas", "scale": 2},
    "scrollZoom": True,
})
```

### 15.4 Integración con otras herramientas

| Entorno | Cómo |
|---|---|
| Jupyter / VS Code | `fig.show()` o dejar `fig` al final de la celda. |
| Streamlit | `st.plotly_chart(fig, use_container_width=True)` |
| Dash | `dcc.Graph(figure=fig)` |
| Flask / Django | `fig.to_html(full_html=False)` en la plantilla. |
| pandas | `pd.options.plotting.backend = "plotly"` → `df.plot()` devuelve figuras Plotly. |

---

## 16. Ejemplo práctico completo

**Informe interactivo de ventas en un único HTML**: genera datos diarios de ventas por región y canal, y construye un informe con indicadores KPI, una serie temporal con selector de rango y media móvil, un menú desplegable para filtrar por región, un *treemap* de categorías, un panel de distribución, un mapa de tiendas y un diagrama de cascada de resultados. Todo se exporta a un archivo HTML que se puede compartir por correo, y opcionalmente a PNG.

```python
"""
Informe interactivo de ventas con Plotly.
Requiere: pip install plotly pandas numpy      (opcional: pip install kaleido para PNG)
"""
from pathlib import Path

import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import plotly.io as pio
from plotly.subplots import make_subplots

SALIDA = Path(__file__).resolve().parent / "salida_plotly"
PALETA = ["#1F4E78", "#2E75B6", "#9DC3E6", "#C00000", "#7F7F7F", "#C55A11"]
TIENDAS = pd.DataFrame({
    "region": ["Norte", "Norte", "Centro", "Centro", "Sur", "Sur", "Este", "Este"],
    "tienda": ["Bilbao", "Oviedo", "Madrid", "Valladolid", "Sevilla", "Málaga", "Valencia", "Barcelona"],
    "lat": [43.263, 43.362, 40.417, 41.652, 37.389, 36.721, 39.470, 41.387],
    "lon": [-2.935, -5.849, -3.704, -4.724, -5.984, -4.421, -0.376, 2.169],
})
CATEGORIAS = {"Electrónica": ["Portátiles", "Móviles", "Audio"],
              "Hogar": ["Cocina", "Decoración"],
              "Moda": ["Calzado", "Ropa", "Complementos"]}


# ------------------------------------------------------------------ Configuración
def configurar_plantilla() -> None:
    pio.templates["corporativo"] = go.layout.Template(layout={
        "font": {"family": "Segoe UI, Arial, sans-serif", "size": 12, "color": "#262626"},
        "colorway": PALETA,
        "title": {"font": {"size": 17, "color": "#1F4E78"}, "x": 0.01, "xanchor": "left"},
        "plot_bgcolor": "white", "paper_bgcolor": "white",
        "xaxis": {"showgrid": False, "linecolor": "#BFBFBF", "ticks": "outside", "tickcolor": "#BFBFBF"},
        "yaxis": {"gridcolor": "#EEEEEE", "zeroline": False},
        "legend": {"orientation": "h", "y": 1.02, "yanchor": "bottom", "x": 0, "title": None},
        "margin": {"l": 60, "r": 30, "t": 90, "b": 50},
        "separators": ",.",
        "hoverlabel": {"bgcolor": "white", "font_size": 12},
    })
    pio.templates.default = "plotly_white+corporativo"


# ------------------------------------------------------------------ Datos
def generar_datos(semilla: int = 3) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    fechas = pd.date_range("2024-01-01", "2025-12-31", freq="D")
    filas = []
    for _, t in TIENDAS.iterrows():
        nivel = rng.uniform(0.6, 1.5)
        for categoria, subcategorias in CATEGORIAS.items():
            for sub in subcategorias:
                base = nivel * rng.uniform(300, 900)
                margen_base = rng.uniform(0.18, 0.38)
                tendencia = np.linspace(1, rng.uniform(1.05, 1.35), len(fechas))
                estacional = 1 + 0.3 * np.sin(2 * np.pi * (fechas.dayofyear - 80) / 365)
                fin_semana = np.where(fechas.dayofweek >= 5, 1.35, 1.0)
                ruido = rng.normal(1, 0.15, len(fechas)).clip(0.4)
                importe = base * tendencia * estacional * fin_semana * ruido
                filas.append(pd.DataFrame({
                    "fecha": fechas, "region": t["region"], "tienda": t["tienda"],
                    "categoria": categoria, "subcategoria": sub,
                    "canal": rng.choice(["Tienda", "Online"], len(fechas), p=[0.65, 0.35]),
                    "importe": importe.round(2),
                    "margen_pct": rng.normal(margen_base, 0.04, len(fechas)).clip(0.05, 0.5).round(3),
                }))
    return pd.concat(filas, ignore_index=True)


# ------------------------------------------------------------------ Figuras
def figura_kpis(df: pd.DataFrame) -> go.Figure:
    actual, anterior = df[df["fecha"].dt.year == 2025], df[df["fecha"].dt.year == 2024]
    kpis = [
        ("Ventas 2025", actual["importe"].sum(), anterior["importe"].sum(), " €", ",.0f"),
        ("Venta media diaria por tienda",
         actual.groupby(["fecha", "tienda"])["importe"].sum().mean(),
         anterior.groupby(["fecha", "tienda"])["importe"].sum().mean(), " €", ",.0f"),
        ("Margen medio", (actual["margen_pct"] * actual["importe"]).sum() / actual["importe"].sum() * 100,
         (anterior["margen_pct"] * anterior["importe"]).sum() / anterior["importe"].sum() * 100, " %", ".1f"),
        ("Peso del canal online", actual.loc[actual["canal"] == "Online", "importe"].sum()
         / actual["importe"].sum() * 100, anterior.loc[anterior["canal"] == "Online", "importe"].sum()
         / anterior["importe"].sum() * 100, " %", ".1f"),
    ]
    fig = make_subplots(rows=1, cols=4, specs=[[{"type": "domain"}] * 4])
    for i, (titulo, valor, referencia, sufijo, formato) in enumerate(kpis, start=1):
        fig.add_trace(go.Indicator(
            mode="number+delta", value=valor,
            number={"suffix": sufijo,
                    "valueformat": formato, "font": {"size": 34, "color": "#1F4E78"}},
            delta={"reference": referencia, "relative": True, "valueformat": ".1%",
                   "increasing": {"color": "#2E7D32"}, "decreasing": {"color": "#C00000"}},
            title={"text": f"{titulo}<br><span style='font-size:11px;color:#7F7F7F'>vs. 2024</span>"},
        ), row=1, col=i)
    fig.update_layout(height=200, margin={"t": 60, "b": 10}, title="Indicadores clave")
    return fig


def figura_evolucion(df: pd.DataFrame) -> go.Figure:
    regiones = sorted(df["region"].unique())
    diario = df.groupby(["fecha", "region"])["importe"].sum().unstack()
    diario["Total"] = diario.sum(axis=1)
    series = ["Total", *regiones]

    fig = go.Figure()
    for nombre in series:
        visible = nombre == "Total"
        fig.add_trace(go.Scatter(x=diario.index, y=diario[nombre], name=f"{nombre} (diario)",
                                 mode="lines", line={"width": 1, "color": "#9DC3E6"}, visible=visible,
                                 hovertemplate="%{x|%d/%m/%Y}<br>%{y:,.0f} €<extra></extra>"))
        fig.add_trace(go.Scatter(x=diario.index, y=diario[nombre].rolling(28, min_periods=1).mean(),
                                 name=f"{nombre} (media 28 días)", mode="lines",
                                 line={"width": 2.5, "color": "#1F4E78"}, visible=visible,
                                 hovertemplate="%{x|%d/%m/%Y}<br>Media: %{y:,.0f} €<extra></extra>"))

    botones = []
    for i, nombre in enumerate(series):
        visibles = [j // 2 == i for j in range(len(series) * 2)]
        botones.append({"label": nombre, "method": "update",
                        "args": [{"visible": visibles}, {"title.text": f"Evolución diaria de ventas · {nombre}"}]})

    fig.update_layout(
        title="Evolución diaria de ventas · Total", height=480, hovermode="x unified", showlegend=False,
        updatemenus=[{"buttons": botones, "x": 1, "xanchor": "right", "y": 1.18, "yanchor": "top",
                      "bgcolor": "white", "bordercolor": "#BFBFBF"}],
    )
    fig.update_yaxes(ticksuffix=" €", tickformat=",.0f", rangemode="tozero")
    fig.update_xaxes(rangeslider={"visible": True, "thickness": 0.06}, rangeselector={
        "buttons": [{"count": 3, "label": "3M", "step": "month", "stepmode": "backward"},
                    {"count": 6, "label": "6M", "step": "month", "stepmode": "backward"},
                    {"count": 1, "label": "Año en curso", "step": "year", "stepmode": "todate"},
                    {"step": "all", "label": "Todo"}]})
    return fig


def figura_treemap(df: pd.DataFrame) -> go.Figure:
    datos = (df[df["fecha"].dt.year == 2025]
             .assign(margen=lambda d: d["importe"] * d["margen_pct"])
             .groupby(["categoria", "subcategoria", "region"], as_index=False)[["importe", "margen"]].sum())
    datos["margen_pct"] = datos["margen"] / datos["importe"] * 100
    fig = px.treemap(datos, path=[px.Constant("Total"), "categoria", "subcategoria", "region"],
                     values="importe", color="margen_pct", color_continuous_scale="RdYlGn",
                     color_continuous_midpoint=datos["margen_pct"].median(),
                     title="Ventas 2025 por categoría (tamaño) y margen (color)")
    fig.update_traces(hovertemplate="<b>%{label}</b><br>Ventas: %{value:,.0f} €"
                                    "<br>Margen: %{color:.1f} %<extra></extra>",
                      texttemplate="%{label}<br>%{value:.3s} €", marker_line_color="white")
    fig.update_layout(height=520, coloraxis_colorbar={"title": "Margen %"}, margin={"t": 70})
    return fig


def figura_distribucion(df: pd.DataFrame) -> go.Figure:
    mensual = (df.assign(mes=df["fecha"].dt.to_period("M").dt.to_timestamp())
               .groupby(["mes", "canal"], as_index=False)["importe"].sum())
    diario_tienda = df.groupby(["fecha", "tienda", "region"], as_index=False)["importe"].sum()

    fig = make_subplots(rows=1, cols=2, column_widths=[0.6, 0.4], horizontal_spacing=0.1,
                        subplot_titles=("Ventas mensuales por canal", "Ventas diarias por tienda"))
    for canal, color in [("Tienda", "#1F4E78"), ("Online", "#9DC3E6")]:
        d = mensual[mensual["canal"] == canal]
        fig.add_trace(go.Bar(x=d["mes"], y=d["importe"], name=canal, marker_color=color,
                             hovertemplate="%{x|%b %Y}<br>" + canal + ": %{y:,.0f} €<extra></extra>"),
                      row=1, col=1)
    orden = diario_tienda.groupby("tienda")["importe"].median().sort_values().index
    for tienda in orden:
        d = diario_tienda[diario_tienda["tienda"] == tienda]
        fig.add_trace(go.Box(x=d["importe"], name=tienda, marker_color="#2E75B6", boxpoints=False,
                             showlegend=False), row=1, col=2)
    fig.update_layout(barmode="stack", height=440, title="Distribución de las ventas")
    fig.update_yaxes(tickformat=".2s", ticksuffix=" €", row=1, col=1)
    fig.update_xaxes(tickformat=".2s", ticksuffix=" €", row=1, col=2)
    return fig


def figura_mapa(df: pd.DataFrame) -> go.Figure:
    por_tienda = (df[df["fecha"].dt.year == 2025].groupby("tienda", as_index=False)["importe"].sum()
                  .merge(TIENDAS, on="tienda"))
    fig = px.scatter_map(por_tienda, lat="lat", lon="lon", size="importe", color="region",
                         hover_name="tienda", hover_data={"importe": ":,.0f", "lat": False, "lon": False,
                                                         "region": True},
                         size_max=38, zoom=4.6, center={"lat": 40.0, "lon": -3.6},
                         map_style="carto-positron", title="Ventas 2025 por tienda",
                         labels={"importe": "Ventas (€)", "region": "Región"})
    fig.update_layout(height=520, margin={"l": 0, "r": 0, "t": 70, "b": 0})
    return fig


def figura_cascada(df: pd.DataFrame) -> go.Figure:
    ventas = df.loc[df["fecha"].dt.year == 2025, "importe"].sum()
    margen = (df.loc[df["fecha"].dt.year == 2025, "importe"] * df.loc[df["fecha"].dt.year == 2025, "margen_pct"]).sum()
    coste_ventas = ventas - margen
    personal, alquileres, marketing = ventas * 0.09, ventas * 0.05, ventas * 0.03
    beneficio = margen - personal - alquileres - marketing
    importes = [ventas, -coste_ventas, margen, -personal, -alquileres, -marketing, beneficio]
    fig = go.Figure(go.Waterfall(
        x=["Ventas", "Coste de ventas", "Margen bruto", "Personal", "Alquileres", "Marketing", "Beneficio"],
        y=[ventas, -coste_ventas, 0, -personal, -alquileres, -marketing, 0],   # Los totales se calculan solos
        measure=["absolute", "relative", "total", "relative", "relative", "relative", "total"],
        text=[f"{v / 1e6:,.2f} M€".replace(".", ",") for v in importes], textposition="outside",
        hovertemplate="%{x}: %{text}<extra></extra>",
        increasing={"marker": {"color": "#2E75B6"}}, decreasing={"marker": {"color": "#C00000"}},
        totals={"marker": {"color": "#1F4E78"}}, connector={"line": {"color": "#BFBFBF"}},
    ))
    fig.update_layout(title=f"Cuenta de resultados simplificada 2025 · beneficio {beneficio / ventas:.1%}".replace(".", ","),
                      height=440, showlegend=False)
    fig.update_yaxes(tickformat=".2s", ticksuffix=" €", range=[0, ventas * 1.12])
    return fig


# ------------------------------------------------------------------ Informe
def exportar_informe(figuras: list[go.Figure], ruta: Path) -> None:
    configuracion = {"displaylogo": False, "locale": "es",
                     "toImageButtonOptions": {"format": "png", "scale": 2}}
    partes = [f.to_html(full_html=False, include_plotlyjs="cdn" if i == 0 else False, config=configuracion)
              for i, f in enumerate(figuras)]
    html = f"""<!DOCTYPE html>
<html lang="es"><head><meta charset="utf-8"><title>Informe de ventas</title>
<style>
  body {{ font-family: 'Segoe UI', Arial, sans-serif; background: #F4F6F9; margin: 0; color: #262626; }}
  header {{ background: #1F4E78; color: white; padding: 18px 32px; }}
  header h1 {{ margin: 0; font-size: 24px; }} header p {{ margin: 4px 0 0; opacity: .8; }}
  main {{ max-width: 1200px; margin: 0 auto; padding: 20px; }}
  .tarjeta {{ background: white; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,.08);
             margin-bottom: 20px; padding: 8px; }}
</style></head><body>
<header><h1>Informe de ventas 2024–2025</h1>
<p>Generado el {pd.Timestamp.now():%d/%m/%Y %H:%M} · Datos simulados</p></header>
<main>{''.join(f'<section class="tarjeta">{p}</section>' for p in partes)}</main>
</body></html>"""
    ruta.write_text(html, encoding="utf-8")


def main() -> None:
    configurar_plantilla()
    SALIDA.mkdir(exist_ok=True)
    df = generar_datos()
    figuras = [figura_kpis(df), figura_evolucion(df), figura_treemap(df),
               figura_distribucion(df), figura_mapa(df), figura_cascada(df)]
    exportar_informe(figuras, SALIDA / "informe_ventas.html")
    print(f"Informe interactivo: {SALIDA / 'informe_ventas.html'}")

    try:                                                     # Imagen estática opcional
        figuras[3].write_image(SALIDA / "distribucion.png", width=1200, height=450, scale=2)
        print("Imagen PNG generada.")
    except Exception as error:                               # kaleido o Chrome no disponibles
        print(f"PNG no generado ({error.__class__.__name__}). Instale kaleido para exportar imágenes.")


if __name__ == "__main__":
    main()
```

Abra `informe_ventas.html` en cualquier navegador: todos los gráficos son interactivos (zoom, *tooltips*, filtro por región, selector de rango y descarga de imagen).

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `fig.show()` no muestra nada en un script | Renderizador no adecuado. | `pio.renderers.default = "browser"` o guarde con `write_html`. |
| Gráfico en blanco en Jupyter | Falta `nbformat` o la extensión. | `pip install --upgrade nbformat` y reinicie el núcleo. |
| `ValueError: Image export using the "kaleido" engine requires the kaleido package` | `kaleido` no instalado. | `pip install kaleido`. |
| `ChromeNotFoundError` al exportar imagen | `kaleido` necesita Chrome/Chromium. | Instale Chrome o ejecute `plotly_get_chrome`. |
| `ValueError: Value of 'x' is not the name of a column` | Nombre de columna incorrecto. | Revise `df.columns`. |
| `AttributeError: module 'plotly.express' has no attribute 'scatter_mapbox'` | Funciones Mapbox retiradas en versiones recientes. | Use `px.scatter_map`, `px.choropleth_map`, `px.density_map`. |
| El HTML ocupa varios MB | Incluye la librería completa. | `include_plotlyjs="cdn"`. |
| El HTML con `cdn` no se ve sin internet | La librería se descarga al abrirlo. | Use `include_plotlyjs=True` para uso sin conexión. |
| `ValueError: Trace type 'pie' is not compatible with subplot type 'xy'` | Falta `specs` en `make_subplots`. | `specs=[[{"type": "domain"}]]` para sectores, indicadores o tablas. |
| El navegador va muy lento | Demasiados puntos o trazas. | `go.Scattergl` / `render_mode="webgl"`, agregue o muestree los datos. |
| `trendline="ols"` falla | Falta `statsmodels`. | `pip install statsmodels`. |
| Títulos de *facets* con «region=Norte» | Comportamiento por defecto. | `fig.for_each_annotation(lambda a: a.update(text=a.text.split("=")[-1]))`. |
| Las categorías aparecen en orden incorrecto | Orden de aparición en los datos. | `category_orders={...}` o `update_xaxes(categoryorder=...)`. |
| Los ejes cambian en cada fotograma de una animación | Rango automático. | Fije `range_x` y `range_y`. |

---

## 18. Buenas prácticas

- Empiece con **Plotly Express** y recurra a `go` solo para lo que `px` no ofrezca.
- Defina una **plantilla corporativa** y actívela con `pio.templates.default` en lugar de repetir estilos.
- Personalice el `hovertemplate`: muestre solo la información útil, con unidades y formato.
- Use `hovermode="x unified"` en series temporales con varias líneas.
- Fije colores por categoría con `color_discrete_map` para mantener la coherencia entre gráficos.
- Prefiera `include_plotlyjs="cdn"` para compartir por correo o web; use la versión autónoma solo si el destinatario puede no tener conexión.
- Agregue los datos antes de graficar (`groupby`) en lugar de enviar millones de filas al navegador.
- Configure `separators=",."` y `locale: "es"` para mostrar números y fechas con formato español.
- Desactive elementos innecesarios de la barra de herramientas (`displaylogo`, `modeBarButtonsToRemove`).
- Encapsule cada figura en una función que devuelva `go.Figure`: podrá reutilizarla en HTML, Dash o Streamlit sin cambios.

---

## 19. Referencia rápida

```python
import plotly.express as px, plotly.graph_objects as go, plotly.io as pio
from plotly.subplots import make_subplots

px.line / bar / scatter / area / pie / histogram / box / violin / ecdf / imshow / density_heatmap
px.treemap(path=[...]) / sunburst / funnel / timeline / scatter_matrix / parallel_coordinates
px.scatter_map / choropleth_map / density_map / choropleth(locations=iso3)
   parámetros: x, y, color, size, facet_col, hover_name, hover_data, text_auto, labels, title,
               color_discrete_map, category_orders, animation_frame, template

go.Figure() + fig.add_trace(go.Scatter / Bar / Pie / Heatmap / Indicator / Waterfall / Candlestick / Sankey / Table)
make_subplots(rows, cols, specs=[[{"type": "xy"}, {"type": "domain"}]], subplot_titles=)

fig.update_layout(title=, height=, legend=, hovermode="x unified", separators=",.", updatemenus=[...])
fig.update_xaxes(...) / update_yaxes(tickformat=",.0f", ticksuffix=" €") / update_traces(hovertemplate=)
fig.add_hline / add_vline / add_vrect / add_annotation / add_shape

pio.templates.default = "plotly_white"
fig.show() / write_html("f.html", include_plotlyjs="cdn") / write_image("f.png", scale=2) / to_html(full_html=False)
```

---

## 20. Ejercicios propuestos

1. Represente la cotización diaria de una acción con velas japonesas, volumen en un segundo panel y selector de rango.
2. Cree un *sunburst* de ventas por continente, país y producto, coloreado por crecimiento interanual.
3. Construya una animación de burbujas que muestre la evolución anual de varios indicadores por país.
4. Diseñe un gráfico con un menú desplegable que cambie entre ver ventas, unidades y margen.
5. Amplíe el ejemplo completo con un mapa de coropletas por provincia a partir de un GeoJSON y una tabla `go.Table` con las 10 subcategorías de mayor crecimiento.

---

## 21. Referencias

- Documentación oficial (Python): https://plotly.com/python/
- Plotly Express: https://plotly.com/python/plotly-express/
- Referencia de figuras: https://plotly.com/python/reference/
- Formato del *hover*: https://plotly.com/python/hover-text-and-formatting/
- Exportación de imágenes estáticas: https://plotly.com/python/static-image-export/
- Formatos numéricos (d3-format): https://d3js.org/d3-format
