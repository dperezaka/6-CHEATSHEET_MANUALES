# Manual de uso: Plotly

> **Categoría:** Ciencia de datos · **Utilidad:** Gráficos interactivos para web, informes y paneles
> **Tipo:** Librería de terceros (`pip install plotly`) · **Versión de referencia:** 5.x – 7.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Plotly Express: gráficos en una línea](#4-plotly-express-gráficos-en-una-línea)
5. [Tipos de gráfico de Plotly Express](#5-tipos-de-gráfico-de-plotly-express)
6. [Graph Objects: control detallado](#6-graph-objects-control-detallado)
7. [Diseño (`layout`): títulos, ejes, leyendas y márgenes](#7-diseño-layout-títulos-ejes-leyendas-y-márgenes)
8. [Información emergente (*hover*)](#8-información-emergente-hover)
9. [Varios gráficos: facetas y `make_subplots`](#9-varios-gráficos-facetas-y-make_subplots)
10. [Plantillas, colores y formato numérico](#10-plantillas-colores-y-formato-numérico)
11. [Anotaciones, formas y líneas de referencia](#11-anotaciones-formas-y-líneas-de-referencia)
12. [Interactividad: selectores de rango, botones, deslizadores y animaciones](#12-interactividad-selectores-de-rango-botones-deslizadores-y-animaciones)
13. [Mapas](#13-mapas)
14. [Indicadores y gráficos jerárquicos](#14-indicadores-y-gráficos-jerárquicos)
15. [Exportar: HTML, imágenes y JSON](#15-exportar-html-imágenes-y-json)
16. [Integración: Jupyter, Streamlit y Dash](#16-integración-jupyter-streamlit-y-dash)
17. [Rendimiento con muchos datos](#17-rendimiento-con-muchos-datos)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Plotly** es una librería para crear **gráficos interactivos** que se visualizan en el navegador: el usuario puede acercar, desplazarse, ver información al pasar el ratón, ocultar series desde la leyenda, cambiar de vista con botones o reproducir animaciones. Los gráficos se guardan como **HTML autónomo** (se abren sin Python ni servidor) o como imagen estática, y se integran directamente en Jupyter, Streamlit y Dash.

**Ventajas:**
- Interactividad completa sin escribir JavaScript.
- **Plotly Express**: gráficos complejos en una sola línea a partir de un DataFrame.
- Gran variedad: estadísticos, financieros, mapas, 3D, jerárquicos, indicadores.
- Exportación a HTML para compartir por correo o publicar en una web.
- Base de **Dash** para crear aplicaciones analíticas.

**Limitaciones:**
- Los HTML pesan (~3-4 MB si incluyen la librería JavaScript).
- Con cientos de miles de puntos, el navegador se ralentiza (hay soluciones: WebGL, agregación).
- La exportación a imagen requiere el paquete **Kaleido** (y un navegador Chrome/Chromium).
- Para figuras de publicación con control absoluto, Matplotlib sigue siendo más preciso.

| Si necesita… | Opción |
|---|---|
| Gráficos interactivos para web, informes HTML o paneles | **Plotly** |
| Gráficos estáticos para documentos e impresión | Matplotlib |
| Exploración estadística rápida estática | Seaborn |
| Aplicación de datos completa en Python | Streamlit / Dash (con Plotly) |

---

## 2. Instalación e importación

```bash
pip install plotly
pip install kaleido          # Exportar a PNG, SVG o PDF
pip install pandas           # Plotly Express trabaja con DataFrames
```

```python
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import plotly.io as pio
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Figure` | Objeto gráfico completo: datos (*traces*) + diseño (*layout*). |
| *Trace* | Una serie de datos dibujada con un tipo concreto (`Scatter`, `Bar`, `Pie`…). |
| *Layout* | Todo lo que no son datos: títulos, ejes, leyenda, márgenes, plantilla, anotaciones. |
| Plotly Express (`px`) | API de alto nivel: una función por tipo de gráfico, a partir de un DataFrame. |
| Graph Objects (`go`) | API de bajo nivel: construcción explícita de *traces* y *layout*. |
| Plantilla (*template*) | Conjunto de estilos predefinido (`plotly_white`, `simple_white`…). |
| *Hover* | Información emergente al pasar el ratón. |
| *Frames* | Fotogramas de una animación. |

Una figura es, internamente, un diccionario serializable a JSON:

```python
fig = px.bar(x=["A", "B"], y=[3, 5])
print(fig.to_dict()["data"][0]["type"])   # 'bar'
```

---

## 4. Plotly Express: gráficos en una línea

```python
import pandas as pd
import plotly.express as px

df = pd.DataFrame({
    "mes": pd.date_range("2026-01-01", periods=12, freq="MS").repeat(3),
    "region": ["Norte", "Sur", "Este"] * 12,
    "ventas": [120, 95, 140, 130, 90, 150, 125, 99, 160, 140, 105, 170,
               150, 110, 175, 160, 120, 180, 170, 115, 190, 165, 130, 200,
               180, 140, 210, 175, 135, 205, 190, 150, 220, 210, 160, 240],
})

fig = px.line(df, x="mes", y="ventas", color="region", markers=True,
              title="Ventas mensuales por región", labels={"ventas": "Ventas (k€)", "mes": ""},
              template="plotly_white")
fig.show()                                     # Abre el navegador (o se muestra en Jupyter)
fig.write_html("ventas.html")                  # Archivo interactivo autónomo
```

Parámetros comunes de Plotly Express:

| Parámetro | Uso |
|---|---|
| `x`, `y` | Columnas de los ejes. |
| `color` | Color por categoría o escala continua. |
| `size`, `symbol`, `line_dash`, `pattern_shape` | Otras variables visuales. |
| `facet_col`, `facet_row`, `facet_col_wrap` | Paneles por categoría. |
| `hover_name`, `hover_data` | Contenido del *hover*. |
| `text`, `text_auto` | Etiquetas sobre los elementos. |
| `labels` | Renombrar columnas en ejes y leyendas. |
| `title`, `template`, `height`, `width` | Presentación. |
| `category_orders` | Orden de las categorías. |
| `color_discrete_sequence` / `color_discrete_map` | Colores por categoría. |
| `color_continuous_scale` | Escala continua. |
| `animation_frame`, `animation_group` | Animación. |
| `log_x`, `log_y`, `range_x`, `range_y` | Escalas y rangos. |

---

## 5. Tipos de gráfico de Plotly Express

```python
# Básicos
px.line(df, x=, y=, color=)            ; px.area(df, x=, y=, color=)
px.bar(df, x=, y=, color=, barmode="group" | "stack" | "relative", text_auto=True)
px.scatter(df, x=, y=, color=, size=, hover_name=, trendline="ols")     # trendline requiere statsmodels
px.pie(df, names=, values=, hole=0.45)  # Con hole → anillo
px.funnel(df, x=, y=)                   ; px.timeline(df, x_start=, x_end=, y=)   # Diagrama de Gantt

# Distribuciones
px.histogram(df, x=, color=, nbins=40, marginal="box", histnorm="percent")
px.box(df, x=, y=, color=, points="outliers") ; px.violin(df, x=, y=, box=True)
px.strip(df, x=, y=) ; px.ecdf(df, x=, color=)
px.density_heatmap(df, x=, y=, nbinsx=20) ; px.density_contour(df, x=, y=)

# Matrices y relaciones múltiples
px.imshow(tabla, text_auto=".1f", color_continuous_scale="RdBu_r", aspect="auto")   # Mapa de calor
px.scatter_matrix(df, dimensions=["a", "b", "c"], color="grupo")
px.parallel_coordinates(df, color="puntuacion") ; px.parallel_categories(df)

# Jerárquicos
px.treemap(df, path=[px.Constant("Total"), "region", "categoria"], values="ventas")
px.sunburst(df, path=["region", "categoria"], values="ventas")
px.icicle(df, path=["region", "categoria"], values="ventas")

# 3D y polares
px.scatter_3d(df, x=, y=, z=, color=) ; px.line_polar(df, r=, theta=, line_close=True)
```

---

## 6. Graph Objects: control detallado

```python
import plotly.graph_objects as go

fig = go.Figure()
fig.add_trace(go.Scatter(x=meses, y=ventas, mode="lines+markers", name="Ventas",
                         line={"color": "#1F4E78", "width": 3}, marker={"size": 7}))
fig.add_trace(go.Bar(x=meses, y=pedidos, name="Pedidos", marker_color="#E07B00", opacity=0.6, yaxis="y2"))
fig.update_layout(
    title={"text": "Ventas y pedidos", "x": 0.02},
    yaxis={"title": "Ventas (€)"},
    yaxis2={"title": "Pedidos", "overlaying": "y", "side": "right", "showgrid": False},
    legend={"orientation": "h", "y": 1.1},
    template="plotly_white",
)
```

Tipos de *trace* habituales: `Scatter` (líneas y puntos), `Bar`, `Histogram`, `Box`, `Violin`, `Pie`, `Heatmap`, `Contour`, `Candlestick`/`Ohlc` (financieros), `Waterfall`, `Funnel`, `Indicator`, `Treemap`, `Sunburst`, `Sankey`, `Table`, `Scattergl` (WebGL), `Scattergeo`, `Choropleth`, `Scattermap`, `Surface`, `Scatter3d`.

Modificar figuras existentes (también las creadas con Plotly Express):

```python
fig.update_traces(marker_line_width=0, selector={"type": "bar"})
fig.update_traces(line_width=3, selector={"name": "Norte"})
fig.for_each_trace(lambda t: t.update(name=t.name.upper()))
fig.update_xaxes(showgrid=False, tickangle=-45)
fig.update_yaxes(rangemode="tozero", ticksuffix=" €")
fig.data[0].visible = "legendonly"         # Oculta una serie (se activa desde la leyenda)
```

Gráfico de cascada (*waterfall*) y Sankey:

```python
go.Figure(go.Waterfall(x=["Ingresos", "Costes", "Impuestos", "Beneficio"], y=[500, -320, -45, None],
                       measure=["absolute", "relative", "relative", "total"],
                       text=["500", "-320", "-45", "135"], connector={"line": {"color": "gray"}}))

go.Figure(go.Sankey(node={"label": ["Web", "Tienda", "Compra", "Abandono"]},
                    link={"source": [0, 0, 1, 1], "target": [2, 3, 2, 3], "value": [60, 40, 30, 5]}))
```

---

## 7. Diseño (`layout`): títulos, ejes, leyendas y márgenes

```python
fig.update_layout(
    title={"text": "<b>Informe de ventas</b><br><sup>Datos hasta noviembre de 2026</sup>", "x": 0.02},
    font={"family": "Inter, Arial", "size": 13, "color": "#1F2933"},
    height=500, width=None,                                   # width=None: se adapta al contenedor
    margin={"l": 40, "r": 20, "t": 80, "b": 40},
    legend={"title": None, "orientation": "h", "yanchor": "bottom", "y": 1.02, "xanchor": "right", "x": 1},
    hovermode="x unified",                                    # Un único cuadro para todas las series
    plot_bgcolor="white", paper_bgcolor="white",
    bargap=0.25, barmode="group",
    separators=",.",                                          # Decimal «,» y miles «.» (formato español)
)
fig.update_xaxes(title=None, showline=True, linecolor="#ccc", ticks="outside", dtick="M1", tickformat="%b\n%Y")
fig.update_yaxes(title="Importe", gridcolor="#eee", zeroline=False, tickformat=",.0f", ticksuffix=" €")
fig.update_yaxes(type="log")                                  # Escala logarítmica
fig.update_xaxes(categoryorder="total descending")            # Ordenar categorías por valor
```

> `separators=",."` hace que los números se muestren con coma decimal y punto de miles en todo el gráfico.

---

## 8. Información emergente (*hover*)

```python
fig = px.scatter(df, x="precio", y="unidades", color="categoria", hover_name="producto",
                 hover_data={"precio": ":.2f", "unidades": True, "categoria": False, "proveedor": True})

fig.update_traces(hovertemplate="<b>%{hovertext}</b><br>Precio: %{x:,.2f} €<br>Unidades: %{y}<extra></extra>")

# Datos adicionales con customdata
fig.update_traces(customdata=df[["margen", "stock"]],
                  hovertemplate="%{x}: %{y:,.0f} €<br>Margen: %{customdata[0]:.1%}<br>Stock: %{customdata[1]}<extra></extra>")
fig.update_layout(hovermode="x unified", hoverlabel={"bgcolor": "white", "font_size": 12})
```

> `<extra></extra>` elimina el cuadro lateral con el nombre de la serie. Los formatos siguen la sintaxis de d3: `:,.2f`, `:.1%`, `|%d/%m/%Y` para fechas.

---

## 9. Varios gráficos: facetas y `make_subplots`

### 9.1 Facetas con Plotly Express

```python
fig = px.histogram(df, x="importe", facet_col="region", facet_col_wrap=3, color="canal", height=600)
fig.for_each_annotation(lambda a: a.update(text=a.text.split("=")[-1]))   # Títulos sin «region=»
fig.update_yaxes(matches=None)                                             # Ejes Y independientes
```

### 9.2 `make_subplots`

```python
from plotly.subplots import make_subplots

fig = make_subplots(
    rows=2, cols=2,
    specs=[[{"colspan": 2}, None], [{"type": "xy"}, {"type": "domain"}]],   # domain: pie, indicador, sunburst
    subplot_titles=("Evolución", "Por región", "Por canal"),
    vertical_spacing=0.12, row_heights=[0.6, 0.4],
)
fig.add_trace(go.Scatter(x=meses, y=ventas, name="Ventas"), row=1, col=1)
fig.add_trace(go.Bar(x=regiones, y=totales, name="Regiones"), row=2, col=1)
fig.add_trace(go.Pie(labels=canales, values=cuotas, hole=0.5), row=2, col=2)
fig.update_layout(height=700, showlegend=False)

# Eje Y secundario
fig = make_subplots(specs=[[{"secondary_y": True}]])
fig.add_trace(go.Bar(x=m, y=pedidos, name="Pedidos"), secondary_y=False)
fig.add_trace(go.Scatter(x=m, y=ticket, name="Ticket medio"), secondary_y=True)
```

Reutilizar *traces* de Plotly Express en `make_subplots`:

```python
for trace in px.line(df, x="mes", y="ventas", color="region").data:
    fig.add_trace(trace, row=1, col=1)
```

---

## 10. Plantillas, colores y formato numérico

```python
pio.templates.default = "plotly_white"     # Para todas las figuras
# Disponibles: plotly, plotly_white, plotly_dark, ggplot2, seaborn, simple_white, presentation, none

# Plantilla corporativa propia
pio.templates["corporativa"] = go.layout.Template(
    layout={"font": {"family": "Inter, Arial", "size": 13},
            "colorway": ["#1F4E78", "#E07B00", "#2CA02C", "#C00000", "#7F7F7F"],
            "separators": ",.",
            "xaxis": {"showgrid": False}, "yaxis": {"gridcolor": "#EEE"}})
pio.templates.default = "plotly_white+corporativa"          # Combinar plantillas

px.colors.qualitative.Set2 ; px.colors.sequential.Blues ; px.colors.diverging.RdBu
px.bar(df, x="region", y="ventas", color="region",
       color_discrete_map={"Norte": "#1F4E78", "Sur": "#E07B00", "Este": "#2CA02C"})
px.scatter(df, x="x", y="y", color="valor", color_continuous_scale="Viridis", range_color=[0, 100])
```

Formato de números y fechas (sintaxis d3):

| Formato | Ejemplo |
|---|---|
| `,.0f` | 12.345 (con `separators=",."`) |
| `.1%` | 25,3 % |
| `.2s` | 1,2k · 3,4M |
| `$,.2f` | Moneda (use `ticksuffix=" €"` para euros) |
| `%d/%m/%Y` | 25/11/2026 |
| `%b %Y` | nov 2026 (idioma con `config={"locale": "es"}` en HTML) |

---

## 11. Anotaciones, formas y líneas de referencia

```python
fig.add_hline(y=150, line_dash="dash", line_color="red", annotation_text="Objetivo", annotation_position="top left")
fig.add_vline(x="2026-11-27", line_dash="dot", annotation_text="Black Friday")
fig.add_vrect(x0="2026-07-01", x1="2026-08-31", fillcolor="orange", opacity=0.1, line_width=0,
              annotation_text="Verano")
fig.add_hrect(y0=0, y1=50, fillcolor="red", opacity=0.05)
fig.add_annotation(x="2026-06-01", y=210, text="Máximo", showarrow=True, arrowhead=2, ax=-40, ay=-30)
fig.add_annotation(text="Fuente: ERP", xref="paper", yref="paper", x=1, y=-0.15, showarrow=False,
                   font={"size": 10, "color": "gray"})
fig.add_shape(type="rect", x0=1, x1=2, y0=0, y1=5, line={"color": "black"})
```

> Con `xref="paper"` / `yref="paper"`, las coordenadas van de 0 a 1 respecto al área del gráfico (útil para textos fijos).

---

## 12. Interactividad: selectores de rango, botones, deslizadores y animaciones

### 12.1 Selector de rango de fechas

```python
fig.update_xaxes(
    rangeslider_visible=True,
    rangeselector={"buttons": [
        {"count": 1, "label": "1 m", "step": "month", "stepmode": "backward"},
        {"count": 6, "label": "6 m", "step": "month", "stepmode": "backward"},
        {"count": 1, "label": "Año", "step": "year", "stepmode": "todate"},
        {"step": "all", "label": "Todo"},
    ]},
)
```

### 12.2 Botones y menús desplegables

```python
fig = go.Figure([go.Bar(x=regiones, y=ventas, name="Ventas"),
                 go.Bar(x=regiones, y=margen, name="Margen", visible=False)])
fig.update_layout(updatemenus=[{
    "type": "buttons", "direction": "right", "x": 0, "y": 1.15,
    "buttons": [
        {"label": "Ventas", "method": "update", "args": [{"visible": [True, False]}, {"yaxis.title": "€"}]},
        {"label": "Margen", "method": "update", "args": [{"visible": [False, True]}, {"yaxis.title": "%"}]},
    ],
}])
```

| `method` | Modifica |
|---|---|
| `restyle` | Propiedades de los *traces* (datos, visibilidad, color). |
| `relayout` | Propiedades del *layout* (títulos, rangos). |
| `update` | Ambas cosas. |
| `animate` | Reproduce fotogramas. |

### 12.3 Animaciones

```python
fig = px.scatter(df, x="pib_per_capita", y="esperanza_vida", size="poblacion", color="continente",
                 hover_name="pais", animation_frame="anio", animation_group="pais",
                 log_x=True, size_max=55, range_x=[200, 100_000], range_y=[25, 90])
fig.layout.updatemenus[0].buttons[0].args[1]["frame"]["duration"] = 400      # Velocidad
```

> Fije `range_x`/`range_y` en las animaciones: si no, los ejes cambian en cada fotograma.

---

## 13. Mapas

```python
# Coropletas de países (códigos ISO-3)
px.choropleth(df, locations="iso3", color="ventas", hover_name="pais", color_continuous_scale="Blues")

# Regiones propias con GeoJSON (p. ej., provincias o comunidades autónomas)
import json
geo = json.load(open("provincias.geojson", encoding="utf-8"))
px.choropleth(df, geojson=geo, locations="codigo", featureidkey="properties.cod_prov", color="ventas",
              fitbounds="locations")

# Puntos sobre mapa base (teselas de OpenStreetMap; requiere conexión al visualizar)
px.scatter_map(df, lat="lat", lon="lon", size="ventas", color="tipo", hover_name="tienda",
               zoom=5, center={"lat": 40.4, "lon": -3.7}, map_style="open-street-map")
px.density_map(df, lat="lat", lon="lon", z="ventas", radius=20)
```

> Desde Plotly 5.24 las funciones de mapa con teselas son `scatter_map`, `density_map` y `choropleth_map` (basadas en MapLibre); las antiguas `*_mapbox` están obsoletas.

---

## 14. Indicadores y gráficos jerárquicos

```python
fig = go.Figure(go.Indicator(
    mode="number+delta+gauge", value=1_264_000,
    number={"prefix": "", "suffix": " €", "valueformat": ",.0f"},
    delta={"reference": 1_150_000, "relative": True, "valueformat": ".1%"},
    gauge={"axis": {"range": [0, 1_500_000]}, "bar": {"color": "#1F4E78"},
           "threshold": {"value": 1_200_000, "line": {"color": "red", "width": 3}}},
    title={"text": "Facturación vs. objetivo"},
))

px.treemap(df, path=[px.Constant("Empresa"), "region", "categoria", "producto"], values="ventas",
           color="margen", color_continuous_scale="RdYlGn", color_continuous_midpoint=0.25)
```

---

## 15. Exportar: HTML, imágenes y JSON

```python
fig.write_html("grafico.html")                                     # Autónomo (~3,5 MB, funciona sin Internet)
fig.write_html("grafico.html", include_plotlyjs="cdn")             # Ligero (~10 KB, requiere Internet al abrirlo)
fig.write_html("grafico.html", full_html=False, include_plotlyjs=False)   # Fragmento para incrustar en una web
fig.write_html("grafico.html", config={"displaylogo": False, "locale": "es",
                                       "modeBarButtonsToRemove": ["lasso2d", "select2d"],
                                       "toImageButtonOptions": {"format": "png", "scale": 2}})
html = fig.to_html(include_plotlyjs="cdn")                         # Como texto

fig.write_image("grafico.png", width=1200, height=600, scale=2)    # Requiere kaleido
fig.write_image("grafico.svg") ; fig.write_image("grafico.pdf")
fig.write_json("grafico.json") ; fig = pio.read_json("grafico.json")
```

Varias figuras en un único informe HTML:

```python
partes = [f.to_html(full_html=False, include_plotlyjs="cdn" if i == 0 else False) for i, f in enumerate(figuras)]
Path("informe.html").write_text(f"<html><head><meta charset='utf-8'></head><body>{''.join(partes)}</body></html>",
                                encoding="utf-8")
```

> Kaleido 1.x usa un navegador Chrome/Chromium instalado para generar imágenes. Si no hay ninguno, `plotly_get_chrome` (incluido con Kaleido) lo descarga.

---

## 16. Integración: Jupyter, Streamlit y Dash

```python
# Jupyter / VS Code: la figura se muestra al ser la última expresión de la celda o con fig.show()
pio.renderers.default = "notebook"        # browser, png, svg, notebook_connected, vscode…

# Streamlit
import streamlit as st
st.plotly_chart(fig, use_container_width=True)

# Dash (aplicación web interactiva con callbacks)
from dash import Dash, dcc, html
app = Dash(__name__)
app.layout = html.Div([dcc.Graph(figure=fig)])
app.run(debug=True)
```

---

## 17. Rendimiento con muchos datos

- Use *traces* WebGL: `px.scatter(..., render_mode="webgl")` o `go.Scattergl`.
- Agregue antes de dibujar (por día, por hora, histogramas) en lugar de enviar millones de puntos.
- Para series temporales largas, reduzca la resolución (`resample`) o use `plotly-resampler`.
- Use `include_plotlyjs="cdn"` cuando el archivo se vaya a abrir con conexión a Internet.
- Evite `hover_data` con muchas columnas en figuras grandes.

---

## 18. Ejemplo práctico completo

**Panel interactivo de ventas en un único archivo HTML**: genera datos de ventas diarias por región, canal y categoría; construye indicadores con variación interanual, una serie temporal con selector de rango, media móvil y anotaciones; barras con botones para cambiar de métrica; un *treemap* de la composición de ventas; un mapa de calor día × mes; una dispersión animada mes a mes; y una tabla de las principales categorías. Usa una plantilla corporativa con formato numérico español y guarda el panel en HTML (autónomo o ligero) y la figura principal en PNG.

```python
"""
panel_interactivo.py — Panel de ventas interactivo con Plotly (HTML autónomo).
Requiere: pip install plotly pandas numpy    (kaleido opcional para exportar PNG)
Uso: python panel_interactivo.py [--ligero] [--png]
"""
import argparse
from datetime import date
from pathlib import Path

import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import plotly.io as pio
from plotly.subplots import make_subplots

COLORES = ["#1F4E78", "#E07B00", "#2CA02C", "#C00000", "#7F7F7F", "#8E6BBF"]
pio.templates["corporativa"] = go.layout.Template(layout={
    "font": {"family": "Inter, Segoe UI, Arial", "size": 13, "color": "#1F2933"},
    "colorway": COLORES, "separators": ",.",
    "title": {"x": 0.01, "font": {"size": 17}},
    "xaxis": {"showgrid": False, "linecolor": "#CCD3DB"},
    "yaxis": {"gridcolor": "#EEF1F4", "zeroline": False},
    "legend": {"orientation": "h", "yanchor": "bottom", "y": 1.02, "x": 0, "title": None},
    "margin": {"l": 50, "r": 20, "t": 70, "b": 40},
    "hoverlabel": {"bgcolor": "white"},
})
pio.templates.default = "plotly_white+corporativa"
MESES = ["Ene", "Feb", "Mar", "Abr", "May", "Jun", "Jul", "Ago", "Sep", "Oct", "Nov", "Dic"]
DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]


# ------------------------------------------------------------------ Datos
def generar_datos(semilla: int = 21) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    fechas = pd.date_range("2025-01-01", "2026-11-24", freq="D")
    regiones = {"Norte": 1.0, "Sur": 0.8, "Este": 1.2, "Oeste": 0.7, "Centro": 1.4}
    categorias = {"Periféricos": (0.30, 0.32), "Monitores": (0.25, 0.18), "Almacenamiento": (0.20, 0.22),
                  "Redes": (0.15, 0.27), "Accesorios": (0.10, 0.45)}
    canales = ["Web", "Tienda", "Marketplace"]
    filas = []
    t = np.arange(len(fechas))
    tendencia = 1 + t / len(fechas) * 0.35
    estacional = 1 + 0.2 * np.sin(2 * np.pi * (fechas.dayofyear - 80) / 365) + \
        np.where((fechas.month == 11) & (fechas.day >= 20), 0.8, 0) + np.where(fechas.month == 12, 0.4, 0)
    semanal = np.where(fechas.dayofweek >= 5, 0.7, 1.0)
    for region, peso_r in regiones.items():
        for categoria, (peso_c, margen) in categorias.items():
            for canal, peso_k in zip(canales, [0.5, 0.35, 0.15]):
                base = 900 * peso_r * peso_c * peso_k * 5
                ventas = base * tendencia * estacional * semanal * rng.lognormal(0, 0.25, len(fechas))
                filas.append(pd.DataFrame({"fecha": fechas, "region": region, "categoria": categoria,
                                           "canal": canal, "ventas": ventas.round(2),
                                           "margen": (ventas * rng.normal(margen, 0.03, len(fechas))).round(2),
                                           "pedidos": rng.poisson(ventas / 65)}))
    return pd.concat(filas, ignore_index=True)


# ------------------------------------------------------------------ Figuras
def fig_indicadores(df: pd.DataFrame) -> go.Figure:
    hoy = df["fecha"].max()
    actual = df[df["fecha"].dt.year == hoy.year]
    anterior = df[(df["fecha"].dt.year == hoy.year - 1) & (df["fecha"].dt.dayofyear <= hoy.dayofyear)]
    kpis = [
        ("Ventas acumuladas", actual["ventas"].sum(), anterior["ventas"].sum(), " €", ",.0f", True),
        ("Margen", actual["margen"].sum() / actual["ventas"].sum() * 100,
         anterior["margen"].sum() / anterior["ventas"].sum() * 100, " %", ".1f", False),   # Diferencia en puntos
        ("Pedidos", actual["pedidos"].sum(), anterior["pedidos"].sum(), "", ",.0f", True),
        ("Ticket medio", actual["ventas"].sum() / actual["pedidos"].sum(),
         anterior["ventas"].sum() / anterior["pedidos"].sum(), " €", ",.2f", True),
    ]
    fig = make_subplots(rows=1, cols=4, specs=[[{"type": "domain"}] * 4])
    for i, (titulo, valor, referencia, sufijo, formato, relativo) in enumerate(kpis, 1):
        fig.add_trace(go.Indicator(
            mode="number+delta", value=valor,
            title={"text": f"<b>{titulo}</b><br><span style='font-size:12px;color:#7F8C99'>"
                           f"vs. mismo periodo de {hoy.year - 1}</span>", "font": {"size": 16}},
            number={"suffix": sufijo, "valueformat": formato, "font": {"size": 34}},
            delta={"reference": referencia, "relative": relativo,
                   "valueformat": ".1%" if relativo else "+.2f", "suffix": "" if relativo else " p.p."}),
            row=1, col=i)
    fig.update_layout(height=190, margin={"t": 60, "b": 10})
    return fig


def fig_evolucion(df: pd.DataFrame) -> go.Figure:
    diario = df.groupby("fecha", as_index=False)[["ventas", "pedidos"]].sum()
    diario["media_28"] = diario["ventas"].rolling(28, center=True).mean()
    fig = go.Figure()
    fig.add_trace(go.Scatter(x=diario["fecha"], y=diario["ventas"], name="Diario", mode="lines",
                             line={"color": "#B8C4D0", "width": 1},
                             customdata=diario[["pedidos"]],
                             hovertemplate="%{x|%d/%m/%Y}<br>Ventas: %{y:,.0f} €<br>Pedidos: %{customdata[0]:,}<extra></extra>"))
    fig.add_trace(go.Scatter(x=diario["fecha"], y=diario["media_28"], name="Media móvil 28 días", mode="lines",
                             line={"color": COLORES[0], "width": 3}, hovertemplate="Media: %{y:,.0f} €<extra></extra>"))
    for anio in (2025, 2026):
        fig.add_vrect(x0=f"{anio}-11-20", x1=f"{anio}-11-30", fillcolor=COLORES[1], opacity=0.12, line_width=0,
                      annotation_text="Campaña", annotation_position="top left", annotation_font_size=11)
    pico = diario.loc[diario["ventas"].idxmax()]
    fig.add_annotation(x=pico["fecha"], y=pico["ventas"], text=f"Máximo: {pico['ventas']:,.0f} €".replace(",", "."),
                       showarrow=True, arrowhead=2, ax=-60, ay=-25)
    fig.update_xaxes(rangeslider={"visible": True, "thickness": 0.06}, rangeselector={"buttons": [
        {"count": 1, "label": "1 m", "step": "month", "stepmode": "backward"},
        {"count": 3, "label": "3 m", "step": "month", "stepmode": "backward"},
        {"count": 1, "label": "Año en curso", "step": "year", "stepmode": "todate"},
        {"step": "all", "label": "Todo"}]})
    fig.update_yaxes(title="Ventas diarias", ticksuffix=" €", tickformat=",.0f")
    fig.update_xaxes(rangeselector={"x": 0, "y": 1.06})
    fig.update_layout(title="Evolución de las ventas", height=500, hovermode="x unified", margin={"t": 100},
                      legend={"x": 1, "xanchor": "right", "y": 1.06})
    return fig


def fig_regiones(df: pd.DataFrame) -> go.Figure:
    tabla = (df[df["fecha"].dt.year == 2026].groupby(["region", "canal"], as_index=False)
             .agg(ventas=("ventas", "sum"), margen=("margen", "sum"), pedidos=("pedidos", "sum")))
    tabla["margen_pct"] = tabla["margen"] / tabla["ventas"]
    orden = tabla.groupby("region")["ventas"].sum().sort_values(ascending=False).index.tolist()
    metricas = [("ventas", "Ventas (€)", ",.0f", " €"), ("pedidos", "Pedidos", ",.0f", ""),
                ("margen_pct", "Margen (%)", ".1%", "")]
    fig = go.Figure()
    canales = tabla["canal"].unique()
    for m, (col, _, formato, sufijo) in enumerate(metricas):
        for k, canal in enumerate(canales):
            datos = tabla[tabla["canal"] == canal].set_index("region").loc[orden]
            fig.add_trace(go.Bar(x=orden, y=datos[col], name=canal, marker_color=COLORES[k], visible=(m == 0),
                                 legendgroup=canal, showlegend=True,
                                 hovertemplate=f"%{{x}} · {canal}<br>%{{y:{formato}}}{sufijo}<extra></extra>"))
    n = len(canales)
    botones = []
    for m, (_, titulo, formato, sufijo) in enumerate(metricas):
        visible = [m == i // n for i in range(len(metricas) * n)]
        botones.append({"label": titulo, "method": "update",
                        "args": [{"visible": visible},
                                 {"yaxis": {"title": titulo, "tickformat": formato, "ticksuffix": sufijo,
                                            "gridcolor": "#EEF1F4"},
                                  "barmode": "relative" if m == 2 else "group"}]})
    fig.update_layout(title="Regiones y canales (2026)", barmode="group", height=450, margin={"t": 110},
                      yaxis={"title": "Ventas (€)", "tickformat": ",.0f", "ticksuffix": " €"},
                      updatemenus=[{"type": "buttons", "direction": "right", "x": 1, "xanchor": "right", "y": 1.28,
                                    "buttons": botones, "showactive": True}])
    return fig


def fig_composicion(df: pd.DataFrame) -> go.Figure:
    tabla = (df[df["fecha"].dt.year == 2026].groupby(["region", "categoria"], as_index=False)
             .agg(ventas=("ventas", "sum"), margen=("margen", "sum")))
    tabla["margen_pct"] = tabla["margen"] / tabla["ventas"] * 100
    fig = px.treemap(tabla, path=[px.Constant("Total 2026"), "region", "categoria"], values="ventas",
                     color="margen_pct", color_continuous_scale="RdYlGn", color_continuous_midpoint=27,
                     custom_data=["margen_pct"])
    fig.update_traces(texttemplate="<b>%{label}</b><br>%{value:,.0f} €<br>%{percentRoot:.1%}",
                      hovertemplate="<b>%{label}</b><br>Ventas: %{value:,.0f} €<br>Margen: %{customdata[0]:.1f} %<extra></extra>")
    fig.update_layout(title="Composición de las ventas (color = margen %)", height=480,
                      coloraxis_colorbar={"title": "Margen %"}, margin={"t": 60, "l": 10, "r": 10, "b": 10})
    return fig


def fig_calor(df: pd.DataFrame) -> go.Figure:
    d = df[df["fecha"].dt.year == 2026].groupby("fecha", as_index=False)["ventas"].sum()
    tabla = d.pivot_table(index=d["fecha"].dt.dayofweek, columns=d["fecha"].dt.month, values="ventas", aggfunc="mean")
    fig = px.imshow(tabla.values / 1000, x=[MESES[m - 1] for m in tabla.columns], y=DIAS,
                    color_continuous_scale="Blues", text_auto=".1f", aspect="auto",
                    labels={"color": "k€/día", "x": "", "y": ""})
    fig.update_traces(hovertemplate="%{y} · %{x}<br>Venta media: %{z:.1f} k€<extra></extra>")
    fig.update_layout(title="Venta media diaria por día de la semana y mes (k€, 2026)", height=380)
    return fig


def fig_animada(df: pd.DataFrame) -> go.Figure:
    mensual = (df.assign(mes=df["fecha"].dt.to_period("M").astype(str))
               .groupby(["mes", "categoria"], as_index=False)
               .agg(ventas=("ventas", "sum"), margen=("margen", "sum"), pedidos=("pedidos", "sum")))
    mensual["margen_pct"] = mensual["margen"] / mensual["ventas"] * 100
    mensual["ticket"] = mensual["ventas"] / mensual["pedidos"]
    fig = px.scatter(mensual, x="ticket", y="margen_pct", size="ventas", color="categoria", hover_name="categoria",
                     animation_frame="mes", animation_group="categoria", size_max=60,
                     range_x=[mensual["ticket"].min() * 0.9, mensual["ticket"].max() * 1.1],
                     range_y=[mensual["margen_pct"].min() - 3, mensual["margen_pct"].max() + 3],
                     labels={"ticket": "Ticket medio (€)", "margen_pct": "Margen (%)", "mes": "Mes"})
    fig.update_layout(title="Evolución mensual por categoría (tamaño = ventas)", height=540,
                      legend={"orientation": "v", "x": 1.02, "y": 1, "yanchor": "top"})
    fig.layout.updatemenus[0].buttons[0].args[1]["frame"]["duration"] = 350
    return fig


def fig_tabla(df: pd.DataFrame) -> go.Figure:
    d = df[df["fecha"].dt.year == 2026]
    t = (d.groupby("categoria").agg(ventas=("ventas", "sum"), margen=("margen", "sum"), pedidos=("pedidos", "sum"))
         .assign(margen_pct=lambda x: x["margen"] / x["ventas"], ticket=lambda x: x["ventas"] / x["pedidos"])
         .sort_values("ventas", ascending=False).reset_index())
    formato = lambda v, f: f"{v:{f}}".replace(",", "X").replace(".", ",").replace("X", ".")
    fig = go.Figure(go.Table(
        header={"values": ["<b>Categoría</b>", "<b>Ventas</b>", "<b>Margen</b>", "<b>Pedidos</b>", "<b>Ticket medio</b>"],
                "fill_color": COLORES[0], "font": {"color": "white"}, "align": ["left", "right"]},
        cells={"values": [t["categoria"], [formato(v, ",.0f") + " €" for v in t["ventas"]],
                          [formato(v * 100, ".1f") + " %" for v in t["margen_pct"]],
                          [formato(v, ",.0f") for v in t["pedidos"]], [formato(v, ",.2f") + " €" for v in t["ticket"]]],
               "fill_color": [["#F7F9FB", "white"] * len(t)], "align": ["left", "right"], "height": 28}))
    fig.update_layout(title="Resumen por categoría (2026)", height=290, margin={"t": 50, "b": 10})
    return fig


# ------------------------------------------------------------------ Informe HTML
def componer_html(figuras: list[go.Figure], ligero: bool) -> str:
    config = {"displaylogo": False, "locale": "es", "responsive": True,
              "toImageButtonOptions": {"format": "png", "scale": 2}}
    partes = []
    for i, fig in enumerate(figuras):
        incluir = ("cdn" if ligero else True) if i == 0 else False
        partes.append(f'<section class="tarjeta">{fig.to_html(full_html=False, include_plotlyjs=incluir, config=config)}</section>')
    return f"""<!DOCTYPE html>
<html lang="es"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
<title>Panel de ventas</title>
<script src="https://cdn.plot.ly/plotly-locale-es-latest.js"></script>
<style>
 body{{font-family:Inter,Segoe UI,Arial,sans-serif;background:#F3F5F8;margin:0;color:#1F2933}}
 header{{background:#1F4E78;color:white;padding:18px 28px}} header h1{{margin:0;font-size:24px}}
 header p{{margin:4px 0 0;opacity:.8}} main{{max-width:1300px;margin:auto;padding:18px;display:grid;gap:16px;
 grid-template-columns:1fr 1fr}} .tarjeta{{background:white;border-radius:10px;box-shadow:0 1px 3px rgba(0,0,0,.08);padding:6px}}
 .tarjeta:nth-child(1),.tarjeta:nth-child(2),.tarjeta:nth-child(7){{grid-column:1/-1}}
 @media(max-width:900px){{main{{grid-template-columns:1fr}}}}
</style></head><body>
<header><h1>Panel de ventas</h1><p>Datos simulados hasta el {date.today():%d/%m/%Y} · Pase el ratón sobre los gráficos,
use la leyenda para ocultar series y el selector para cambiar el periodo.</p></header>
<main>{''.join(partes)}</main></body></html>"""


def main() -> int:
    p = argparse.ArgumentParser(description="Panel interactivo de ventas con Plotly.")
    p.add_argument("--salida", type=Path, default=Path("panel_ventas.html"))
    p.add_argument("--ligero", action="store_true", help="cargar plotly.js desde CDN (archivo pequeño)")
    p.add_argument("--png", action="store_true", help="exportar también la evolución en PNG (requiere kaleido)")
    args = p.parse_args()

    df = generar_datos()
    figuras = [fig_indicadores(df), fig_evolucion(df), fig_regiones(df), fig_composicion(df),
               fig_calor(df), fig_animada(df), fig_tabla(df)]
    args.salida.write_text(componer_html(figuras, args.ligero), encoding="utf-8")
    print(f"✔ Panel guardado en {args.salida} ({args.salida.stat().st_size / 1024:,.0f} KB)")
    if args.png:
        figuras[1].write_image(args.salida.with_suffix(".png"), width=1400, height=600, scale=2)
        print(f"✔ Imagen guardada en {args.salida.with_suffix('.png')}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ValueError: Image export using the "kaleido" engine requires the kaleido package` | Falta Kaleido. | `pip install kaleido`. |
| Kaleido no encuentra Chrome | Kaleido 1.x necesita un navegador. | Instale Chrome/Chromium o ejecute `plotly_get_chrome`. |
| `fig.show()` no hace nada | Renderizador inadecuado (servidor, script). | `fig.write_html(...)` o `pio.renderers.default = "browser"`. |
| El HTML se ve en blanco sin Internet | Se usó `include_plotlyjs="cdn"`. | Use `include_plotlyjs=True` (archivo autónomo). |
| `ValueError: Value of 'x' is not the name of a column` | Columna inexistente. | Revise `df.columns`. |
| Las categorías aparecen en orden alfabético | Orden por defecto. | `category_orders={"col": [...]}` o `categoryorder`. |
| Los ejes «saltan» durante la animación | Rangos automáticos por fotograma. | Fije `range_x` y `range_y`. |
| Números con punto decimal | Formato anglosajón por defecto. | `separators=",."` en el *layout*. |
| `ValueError: Trace type 'pie' is not compatible with subplot type 'xy'` | Tipo de subgráfico incorrecto. | `specs=[[{"type": "domain"}]]`. |
| Navegador muy lento | Demasiados puntos. | WebGL (`render_mode="webgl"`), agregación. |
| `trendline` falla | Falta statsmodels. | `pip install statsmodels`. |
| Avisos de funciones `*_mapbox` obsoletas | Sustituidas por las basadas en MapLibre. | `scatter_map`, `density_map`, `choropleth_map`. |

---

## 20. Buenas prácticas

- Empiece con Plotly Express y ajuste con `update_layout`/`update_traces`; use Graph Objects para lo que Express no cubra.
- Defina una plantilla corporativa una vez (colores, fuente, `separators=",."`).
- Cuide el *hover*: muestre la información útil con formato y elimine lo redundante (`<extra></extra>`).
- Ordene categorías con criterio (por valor o lógico) y fije rangos en animaciones.
- No abuse de la interactividad: cada botón o selector debe responder a una pregunta concreta.
- Agregue los datos antes de dibujar y use WebGL con muchos puntos.
- Elija el modo de exportación según el uso: HTML autónomo para enviar, CDN para publicar, PNG/PDF para documentos.
- Desactive el logotipo y botones innecesarios de la barra de herramientas con `config`.

---

## 21. Referencia rápida

```python
px.line · bar(text_auto=, barmode=) · scatter(size=, trendline=) · area · pie(hole=) · histogram(marginal=) · box · violin
px.imshow(text_auto=) · treemap(path=) · sunburst · funnel · timeline · scatter_map · choropleth · scatter_3d
px.*(df, x=, y=, color=, facet_col=, hover_name=, hover_data=, labels=, animation_frame=, category_orders=, template=)
go.Figure() · add_trace(go.Scatter|Bar|Pie|Indicator|Table|Waterfall|Sankey|Heatmap) · make_subplots(rows, cols, specs=)
fig.update_layout(title=, height=, legend=, hovermode="x unified", separators=",.", updatemenus=[...])
fig.update_traces(hovertemplate="%{y:,.0f} €<extra></extra>") · update_xaxes(rangeslider_visible=True) · update_yaxes(tickformat=)
fig.add_hline · add_vline · add_vrect · add_annotation · pio.templates.default = "plotly_white"
fig.show() · write_html(include_plotlyjs="cdn"|True) · to_html(full_html=False) · write_image("x.png", scale=2)
```

---

## 22. Ejercicios propuestos

1. Cree un gráfico de velas (*candlestick*) con los precios diarios de un valor bursátil simulado, con volumen en un subgráfico inferior.
2. Construya un embudo de conversión (visitas → carrito → pago → compra) por canal con `px.funnel`.
3. Añada al panel del ejemplo un desplegable para filtrar la serie temporal por región.
4. Represente en un mapa de puntos las tiendas de una cadena con tamaño proporcional a las ventas y color según el margen.
5. Convierta el panel en una aplicación de Streamlit con filtros de fecha y región.

---

## 23. Referencias

- Documentación oficial (Python): https://plotly.com/python/
- Plotly Express: https://plotly.com/python/plotly-express/
- Referencia de figuras: https://plotly.com/python/reference/
- Exportación de imágenes (Kaleido): https://plotly.com/python/static-image-export/
- Dash: https://dash.plotly.com/
