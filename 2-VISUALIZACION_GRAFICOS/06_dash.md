# Manual de uso: Dash

> **Categoría:** Visualización de datos · **Utilidad:** Crear dashboards interactivos
> **Tipo:** Librería externa (requiere instalación con `pip`). *Framework* web de Plotly, basado en Flask y React.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Componentes HTML (`dash.html`)](#5-componentes-html-dashhtml)
6. [Componentes interactivos (`dash.dcc`)](#6-componentes-interactivos-dashdcc)
7. [*Callbacks*: la lógica de la aplicación](#7-callbacks-la-lógica-de-la-aplicación)
8. [Gráficos interactivos y filtrado cruzado](#8-gráficos-interactivos-y-filtrado-cruzado)
9. [Tablas de datos](#9-tablas-de-datos)
10. [Diseño y estilo](#10-diseño-y-estilo)
11. [Estado, almacenamiento y actualización periódica](#11-estado-almacenamiento-y-actualización-periódica)
12. [*Callbacks* avanzados](#12-callbacks-avanzados)
13. [Aplicaciones de varias páginas](#13-aplicaciones-de-varias-páginas)
14. [Rendimiento](#14-rendimiento)
15. [Despliegue en producción](#15-despliegue-en-producción)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Dash** es un *framework* para crear **aplicaciones web analíticas y dashboards** escribiendo solo Python. Lo desarrolla la empresa de Plotly: los gráficos son figuras de Plotly, el servidor es **Flask** y la interfaz del navegador está hecha con **React**, pero no es necesario conocer HTML, CSS ni JavaScript para usarlo.

Una aplicación Dash tiene dos partes: el **diseño** (*layout*), que describe qué se ve, y los ***callbacks***, funciones de Python que se ejecutan cuando el usuario interactúa y actualizan la página.

**Ventajas:**
- Aplicaciones web completas en Python puro.
- Integración nativa con Plotly (todos sus gráficos y su interactividad).
- Control fino del diseño (HTML y CSS) y de qué se actualiza en cada interacción.
- Modelo **reactivo** sin estado en el servidor: escala a muchos usuarios.
- Ecosistema amplio: Dash Bootstrap Components, Dash AG Grid, Dash Mantine Components, Dash Leaflet…
- Adecuado para producción (es una aplicación Flask: se despliega con Gunicorn, Docker, etc.).

**Limitaciones:**
- Más código que Streamlit para aplicaciones sencillas.
- Hay que pensar en términos de entradas y salidas (*callbacks*), lo que requiere cierta planificación.
- El estado se gestiona de forma explícita (`dcc.Store`), no con variables globales.

| Si necesita… | Opción |
|---|---|
| Dashboard de producción con diseño a medida y muchos usuarios | **Dash** |
| Prototipo o herramienta interna con muy poco código | Streamlit |
| Gráfico interactivo para compartir como archivo | Plotly (HTML) / Bokeh / Altair |
| Aplicación web general (formularios, usuarios, base de datos) | Django / Flask / FastAPI |

---

## 2. Instalación e importación

```bash
pip install dash                              # Incluye plotly y flask
pip install pandas                            # Recomendado
pip install dash-bootstrap-components         # Opcional: diseño con Bootstrap
pip install dash-ag-grid                      # Opcional: tablas avanzadas
```

Comprobar la versión:

```bash
python -c "import dash; print(dash.__version__)"
```

Importaciones habituales:

```python
from dash import Dash, html, dcc, dash_table, Input, Output, State, callback, ctx, no_update, Patch
import plotly.express as px
import pandas as pd
```

| Módulo | Contenido |
|---|---|
| `dash.html` | Etiquetas HTML como componentes: `html.Div`, `html.H1`, `html.P`… |
| `dash.dcc` | Componentes interactivos: `dcc.Graph`, `dcc.Dropdown`, `dcc.Slider`… |
| `dash.dash_table` | Tabla de datos interactiva (`DataTable`). |
| `Input`, `Output`, `State` | Definen las conexiones de los *callbacks*. |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Aplicación (`Dash`) | Objeto principal: `app = Dash(__name__)`. |
| Diseño (`app.layout`) | Árbol de componentes que forman la página. |
| Componente | Elemento de la interfaz (`html.Div`, `dcc.Graph`…) con propiedades como `id`, `children`, `style`, `value`. |
| `id` | Identificador único de un componente; necesario para usarlo en *callbacks*. |
| *Callback* | Función decorada con `@callback` que se ejecuta cuando cambia una propiedad de **entrada** y devuelve nuevos valores para propiedades de **salida**. |
| `Input` | Propiedad que, al cambiar, dispara el *callback*. |
| `Output` | Propiedad que el *callback* actualiza. |
| `State` | Propiedad que se lee pero **no** dispara el *callback* (p. ej., el texto de un formulario hasta pulsar «Enviar»). |
| `dcc.Store` | Almacén de datos en el navegador para compartir información entre *callbacks*. |

**Flujo de una aplicación:**

```
Usuario cambia el desplegable ──► Input("selector", "value")
                                         │
                                 @callback (Python, en el servidor)
                                         │
                       Output("grafico", "figure") ──► El gráfico se redibuja
```

> **Regla de oro:** los *callbacks* no deben modificar variables globales. Cada usuario comparte el mismo proceso del servidor; el estado de cada usuario debe viajar en las entradas y salidas (o en `dcc.Store`).

---

## 4. Primera aplicación

```python
# app.py
from dash import Dash, html, dcc, Input, Output, callback
import plotly.express as px
import pandas as pd

df = pd.DataFrame({
    "mes": ["Ene", "Feb", "Mar", "Abr", "May", "Jun"] * 2,
    "region": ["Norte"] * 6 + ["Sur"] * 6,
    "ventas": [120, 135, 128, 160, 172, 190, 90, 95, 110, 105, 130, 142],
})

app = Dash(__name__, title="Ventas")

app.layout = html.Div([
    html.H1("Ventas por región"),
    dcc.Dropdown(id="selector-region", options=["Norte", "Sur"], value="Norte", clearable=False),
    dcc.Graph(id="grafico-ventas"),
])


@callback(Output("grafico-ventas", "figure"), Input("selector-region", "value"))
def actualizar_grafico(region):
    datos = df[df["region"] == region]
    return px.line(datos, x="mes", y="ventas", markers=True, title=f"Ventas · {region}")


if __name__ == "__main__":
    app.run(debug=True)            # http://127.0.0.1:8050
```

Ejecutar:

```bash
python app.py
```

| Opción de `app.run` | Efecto |
|---|---|
| `debug=True` | Recarga automática al guardar y panel de errores en el navegador. |
| `port=8051` | Puerto (por defecto 8050). |
| `host="0.0.0.0"` | Accesible desde otros equipos de la red. |

> En versiones antiguas se usaba `app.run_server()`; hoy es `app.run()`.

---

## 5. Componentes HTML (`dash.html`)

Cada etiqueta HTML tiene su componente. El contenido va en `children` (primer argumento) y los estilos en `style` (diccionario con nombres en *camelCase*):

```python
html.Div([
    html.H1("Título", style={"color": "#1F4E78", "marginBottom": "4px"}),
    html.P(["Texto con ", html.B("negrita"), " y ", html.A("enlace", href="https://plotly.com")]),
    html.Hr(),
    html.Ul([html.Li("Elemento 1"), html.Li("Elemento 2")]),
    html.Img(src="/assets/logo.png", style={"height": "40px"}),
    html.Button("Actualizar", id="boton", n_clicks=0),
], className="contenedor", style={"padding": "20px", "fontFamily": "Segoe UI"})
```

| Componente | HTML |
|---|---|
| `html.Div`, `html.Span` | Contenedores |
| `html.H1` … `html.H6`, `html.P`, `html.B`, `html.Small` | Texto |
| `html.A`, `html.Img`, `html.Iframe` | Enlaces, imágenes, marcos |
| `html.Ul`, `html.Li`, `html.Table`, `html.Tr`, `html.Td` | Listas y tablas simples |
| `html.Button`, `html.Label` | Botón, etiqueta |
| `html.Details`, `html.Summary` | Sección desplegable |

> `style` usa *camelCase* (`fontSize`, `backgroundColor`); `className` aplica clases CSS.

---

## 6. Componentes interactivos (`dash.dcc`)

```python
dcc.Dropdown(id="region", options=["Norte", "Sur", "Este"], value="Norte", clearable=False)
dcc.Dropdown(id="productos", options=[{"label": "Teclado", "value": "TEC"},
                                      {"label": "Ratón", "value": "RAT"}], multi=True, placeholder="Productos…")
dcc.RadioItems(id="metrica", options=["Ventas", "Unidades"], value="Ventas", inline=True)
dcc.Checklist(id="opciones", options=["Mostrar media", "Escala log"], value=[])
dcc.Slider(id="umbral", min=0, max=100, step=5, value=50, marks={0: "0", 50: "50", 100: "100"})
dcc.RangeSlider(id="rango", min=0, max=1000, value=[100, 800], tooltip={"placement": "bottom"})
dcc.Input(id="texto", type="text", placeholder="Buscar…", debounce=True)
dcc.Input(id="numero", type="number", min=0, max=100, step=1)
dcc.Textarea(id="comentario", style={"width": "100%"})
dcc.DatePickerRange(id="fechas", start_date="2025-01-01", end_date="2025-12-31", display_format="DD/MM/YYYY")
dcc.DatePickerSingle(id="fecha", date="2025-06-30")
dcc.Upload(id="subir", children=html.Div(["Arrastre o ", html.A("seleccione un archivo")]))
dcc.Download(id="descarga")
dcc.Tabs(id="pestanas", value="resumen", children=[dcc.Tab(label="Resumen", value="resumen"),
                                                  dcc.Tab(label="Detalle", value="detalle")])
dcc.Loading(dcc.Graph(id="grafico"), type="circle")          # Indicador de carga
dcc.Markdown("**Negrita**, *cursiva* y [enlaces](https://dash.plotly.com)")
dcc.Store(id="datos-filtrados")
dcc.Interval(id="reloj", interval=5_000)                     # Cada 5 s
dcc.Location(id="url")
dcc.Graph(id="grafico", figure=fig, config={"displaylogo": False})
```

| Componente | Propiedad principal |
|---|---|
| `Dropdown`, `RadioItems`, `Checklist`, `Slider`, `Input` | `value` |
| `RangeSlider` | `value` (lista `[mín, máx]`) |
| `DatePickerRange` | `start_date`, `end_date` |
| `Tabs` | `value` |
| `Graph` | `figure`, `clickData`, `selectedData`, `hoverData`, `relayoutData` |
| `Upload` | `contents`, `filename` |
| `Interval` | `n_intervals` |
| `Store` | `data` |
| `html.Button` | `n_clicks` |

---

## 7. *Callbacks*: la lógica de la aplicación

### 7.1 Varias entradas y salidas

```python
@callback(
    Output("grafico", "figure"),
    Output("total", "children"),
    Input("region", "value"),
    Input("fechas", "start_date"),
    Input("fechas", "end_date"),
)
def actualizar(region, inicio, fin):
    datos = df[(df["region"] == region) & df["fecha"].between(inicio, fin)]
    figura = px.bar(datos, x="fecha", y="ventas")
    return figura, f"{datos['ventas'].sum():,.0f} €".replace(",", ".")
```

> Los argumentos de la función se reciben **en el mismo orden** que los `Input`/`State`, y se devuelven tantos valores como `Output`.

### 7.2 `State`: leer sin disparar

```python
@callback(
    Output("resultado", "children"),
    Input("boton-enviar", "n_clicks"),
    State("nombre", "value"),
    State("email", "value"),
    prevent_initial_call=True,           # No ejecutar al cargar la página
)
def enviar(n_clicks, nombre, email):
    return f"Gracias, {nombre}. Le escribiremos a {email}."
```

### 7.3 No actualizar una salida (`no_update`) y cancelar (`PreventUpdate`)

```python
from dash import no_update
from dash.exceptions import PreventUpdate

@callback(Output("grafico", "figure"), Output("aviso", "children"), Input("producto", "value"))
def actualizar(producto):
    if not producto:
        raise PreventUpdate                         # No cambia nada
    datos = df[df["producto"] == producto]
    if datos.empty:
        return no_update, "Sin datos para ese producto"   # Mantiene el gráfico actual
    return px.line(datos, x="fecha", y="ventas"), ""
```

### 7.4 Saber qué entrada disparó el *callback* (`ctx`)

```python
from dash import ctx

@callback(Output("contador", "children"),
          Input("sumar", "n_clicks"), Input("restar", "n_clicks"), Input("reiniciar", "n_clicks"))
def contar(*_):
    if ctx.triggered_id == "sumar":
        ...
    elif ctx.triggered_id == "reiniciar":
        ...
```

### 7.5 Opciones dependientes (desplegables en cascada)

```python
@callback(Output("tienda", "options"), Output("tienda", "value"), Input("region", "value"))
def actualizar_tiendas(region):
    tiendas = sorted(df.loc[df["region"] == region, "tienda"].unique())
    return tiendas, tiendas[0] if tiendas else None
```

### 7.6 Descargar archivos

```python
@callback(Output("descarga", "data"), Input("boton-csv", "n_clicks"), prevent_initial_call=True)
def descargar(_):
    return dcc.send_data_frame(df.to_csv, "ventas.csv", index=False, sep=";", decimal=",")
    # Excel: dcc.send_data_frame(df.to_excel, "ventas.xlsx", sheet_name="Ventas")   (requiere openpyxl)
```

### 7.7 Subir archivos

```python
import base64, io

@callback(Output("vista-previa", "children"), Input("subir", "contents"), State("subir", "filename"),
          prevent_initial_call=True)
def leer_archivo(contenido, nombre):
    _, datos_b64 = contenido.split(",")
    datos = base64.b64decode(datos_b64)
    df_subido = pd.read_csv(io.StringIO(datos.decode("utf-8")), sep=None, engine="python")
    return f"{nombre}: {len(df_subido)} filas, {len(df_subido.columns)} columnas"
```

---

## 8. Gráficos interactivos y filtrado cruzado

`dcc.Graph` expone como propiedades las acciones del usuario sobre el gráfico:

| Propiedad | Contenido |
|---|---|
| `clickData` | Punto en el que se hizo clic. |
| `hoverData` | Punto bajo el ratón. |
| `selectedData` | Puntos seleccionados con lazo o rectángulo. |
| `relayoutData` | Zoom o desplazamiento aplicado (rango de ejes). |

Filtrado cruzado: un clic en una barra filtra otro gráfico:

```python
@callback(Output("detalle", "figure"), Input("barras-region", "clickData"))
def filtrar_por_region(click):
    region = click["points"][0]["x"] if click else None
    datos = df if region is None else df[df["region"] == region]
    return px.line(datos.groupby("fecha", as_index=False)["ventas"].sum(), x="fecha", y="ventas",
                   title=f"Evolución · {region or 'todas las regiones'}")
```

> Use `custom_data` en Plotly Express para recuperar columnas adicionales en `clickData`: `px.scatter(..., custom_data=["id"])` → `click["points"][0]["customdata"][0]`.

Configuración del gráfico:

```python
dcc.Graph(id="g", figure=fig, config={"displaylogo": False, "locale": "es",
                                      "modeBarButtonsToRemove": ["lasso2d"]},
          style={"height": "400px"})
```

Actualización parcial con `Patch` (sin reenviar toda la figura):

```python
from dash import Patch

@callback(Output("grafico", "figure"), Input("color", "value"), prevent_initial_call=True)
def cambiar_color(color):
    figura = Patch()
    figura["data"][0]["marker"]["color"] = color
    return figura
```

---

## 9. Tablas de datos

### 9.1 `dash_table.DataTable`

```python
from dash import dash_table
from dash.dash_table.Format import Format, Group, Scheme, Symbol

dash_table.DataTable(
    id="tabla",
    data=df.to_dict("records"),
    columns=[
        {"name": "Producto", "id": "producto"},
        {"name": "Ventas", "id": "ventas", "type": "numeric",
         "format": Format(precision=0, scheme=Scheme.fixed, group=Group.yes, group_delimiter=".",
                          decimal_delimiter=",", symbol=Symbol.yes, symbol_suffix=" €")},
        {"name": "Margen", "id": "margen", "type": "numeric", "format": {"specifier": ".1%"}},
    ],
    page_size=15,
    sort_action="native",                 # Ordenar al hacer clic en la cabecera
    filter_action="native",               # Fila de filtros
    row_selectable="multi",
    export_format="xlsx",                 # Botón de exportación
    style_table={"overflowX": "auto"},
    style_header={"backgroundColor": "#1F4E78", "color": "white", "fontWeight": "bold"},
    style_cell={"fontFamily": "Segoe UI", "padding": "6px", "textAlign": "left"},
    style_data_conditional=[
        {"if": {"row_index": "odd"}, "backgroundColor": "#F5F7FA"},
        {"if": {"filter_query": "{margen} < 0.1", "column_id": "margen"}, "color": "#C00000", "fontWeight": "bold"},
    ],
)
```

| Propiedad de salida | Contenido |
|---|---|
| `data` | Filas actuales (también tras editar). |
| `selected_rows` | Índices de las filas seleccionadas. |
| `active_cell` | Celda activa (`row`, `column_id`). |
| `derived_virtual_data` | Filas visibles tras ordenar y filtrar. |

### 9.2 Dash AG Grid (tablas avanzadas)

```python
import dash_ag_grid as dag

dag.AgGrid(
    id="grid",
    rowData=df.to_dict("records"),
    columnDefs=[{"field": "producto", "filter": True},
                {"field": "ventas", "type": "numericColumn",
                 "valueFormatter": {"function": "d3.format(',.0f')(params.value) + ' €'"}}],
    defaultColDef={"sortable": True, "resizable": True, "floatingFilter": True},
    dashGridOptions={"pagination": True, "paginationPageSize": 20},
    className="ag-theme-alpine",
)
```

> AG Grid ofrece agrupación, columnas fijas, edición avanzada y rendimiento con decenas de miles de filas.

---

## 10. Diseño y estilo

### 10.1 Carpeta `assets`

Dash carga automáticamente los archivos `.css` y `.js` de la carpeta `assets/` situada junto a la aplicación:

```
mi_dash/
├── app.py
└── assets/
    ├── estilos.css        # Se aplica automáticamente
    ├── logo.png           # Accesible en /assets/logo.png
    └── favicon.ico        # Icono de la pestaña
```

```css
/* assets/estilos.css */
body { font-family: "Segoe UI", Arial, sans-serif; background: #F4F6F9; margin: 0; }
.tarjeta { background: white; border-radius: 8px; padding: 16px; box-shadow: 0 1px 3px rgba(0,0,0,.08); }
.kpi-valor { font-size: 28px; font-weight: 700; color: #1F4E78; }
```

### 10.2 Rejillas con CSS en línea

```python
html.Div([tarjeta1, tarjeta2, tarjeta3, tarjeta4],
         style={"display": "grid", "gridTemplateColumns": "repeat(4, 1fr)", "gap": "16px"})
html.Div([izquierda, derecha], style={"display": "flex", "gap": "16px", "flexWrap": "wrap"})
```

### 10.3 Dash Bootstrap Components

```python
import dash_bootstrap_components as dbc

app = Dash(__name__, external_stylesheets=[dbc.themes.FLATLY])

app.layout = dbc.Container([
    dbc.Row(dbc.Col(html.H2("Panel de ventas"), width=12)),
    dbc.Row([
        dbc.Col(dbc.Card(dbc.CardBody([html.H6("Ventas"), html.H3(id="kpi-ventas")])), md=3),
        dbc.Col(dbc.Card(dbc.CardBody([html.H6("Pedidos"), html.H3(id="kpi-pedidos")])), md=3),
    ], className="g-3"),
    dbc.Row([dbc.Col(dcc.Graph(id="g1"), lg=8), dbc.Col(dcc.Graph(id="g2"), lg=4)]),
], fluid=True)
```

Temas disponibles: `BOOTSTRAP`, `FLATLY`, `LUX`, `MINTY`, `CERULEAN`, `DARKLY`, `CYBORG`… Componentes: `Card`, `Navbar`, `Modal`, `Alert`, `Badge`, `Button`, `Tabs`, `Offcanvas`, `Toast`…

### 10.4 Plantilla de gráficos coherente

```python
import plotly.io as pio
pio.templates.default = "plotly_white"
```

---

## 11. Estado, almacenamiento y actualización periódica

### 11.1 `dcc.Store`: compartir datos entre *callbacks*

```python
app.layout = html.Div([..., dcc.Store(id="datos-filtrados", storage_type="memory")])

@callback(Output("datos-filtrados", "data"), Input("region", "value"), Input("fechas", "start_date"))
def filtrar(region, inicio):
    return df[(df["region"] == region) & (df["fecha"] >= inicio)].to_dict("records")

@callback(Output("grafico", "figure"), Input("datos-filtrados", "data"))
def dibujar(registros):
    return px.bar(pd.DataFrame(registros), x="fecha", y="ventas")
```

| `storage_type` | Duración |
|---|---|
| `"memory"` | Hasta recargar la página (por defecto). |
| `"session"` | Hasta cerrar la pestaña. |
| `"local"` | Permanente en el navegador. |

> `dcc.Store` guarda JSON en el navegador: evite almacenar grandes volúmenes (más de unos pocos MB). Guarde identificadores o resúmenes y recupere los datos en el servidor.

### 11.2 Actualización periódica (`dcc.Interval`)

```python
dcc.Interval(id="refresco", interval=30_000, n_intervals=0)       # Cada 30 s

@callback(Output("kpi-pedidos", "children"), Input("refresco", "n_intervals"))
def refrescar(_):
    return f"{consultar_pedidos_hoy():,}"
```

---

## 12. *Callbacks* avanzados

### 12.1 Componentes dinámicos (`ALL`, `MATCH`)

Cuando el número de componentes cambia (filtros añadidos por el usuario), use identificadores de diccionario:

```python
from dash import ALL, MATCH

html.Button("Añadir filtro", id="anadir")
html.Div(id="contenedor-filtros", children=[])

@callback(Output("contenedor-filtros", "children"), Input("anadir", "n_clicks"),
          State("contenedor-filtros", "children"), prevent_initial_call=True)
def anadir_filtro(n, filtros):
    filtros.append(dcc.Dropdown(id={"tipo": "filtro", "indice": n}, options=columnas))
    return filtros

@callback(Output("resumen", "children"), Input({"tipo": "filtro", "indice": ALL}, "value"))
def leer_filtros(valores):
    return f"Filtros activos: {[v for v in valores if v]}"
```

### 12.2 *Callbacks* en el navegador (`clientside_callback`)

Para acciones sencillas sin viajar al servidor (más rápidas):

```python
from dash import clientside_callback

clientside_callback(
    """function(n) { return n % 2 === 1 ? {"display": "none"} : {"display": "block"}; }""",
    Output("panel-filtros", "style"),
    Input("boton-ocultar", "n_clicks"),
)
```

### 12.3 *Callbacks* largos con indicador de progreso

```python
@callback(
    Output("resultado", "children"),
    Input("calcular", "n_clicks"),
    running=[(Output("calcular", "disabled"), True, False)],   # Desactiva el botón mientras se ejecuta
    prevent_initial_call=True,
)
def calcular(_):
    return modelo_costoso()
```

> Para tareas de varios minutos, use `background=True` con un gestor de tareas (`DiskcacheManager` o `CeleryManager`).

### 12.4 Encadenar *callbacks*

La salida de un *callback* puede ser la entrada de otro (región → tiendas → gráfico). Dash resuelve el orden automáticamente. Evite ciclos (A actualiza B y B actualiza A) salvo que use el mismo *callback* con `ctx.triggered_id`.

---

## 13. Aplicaciones de varias páginas

```
mi_app/
├── app.py
└── pages/
    ├── inicio.py
    ├── ventas.py
    └── clientes.py
```

```python
# app.py
from dash import Dash, html, dcc, page_container, page_registry

app = Dash(__name__, use_pages=True)
app.layout = html.Div([
    html.Nav([dcc.Link(p["name"], href=p["relative_path"], style={"marginRight": "16px"})
              for p in page_registry.values()]),
    page_container,                         # Aquí se muestra la página activa
])

if __name__ == "__main__":
    app.run(debug=True)
```

```python
# pages/ventas.py
from dash import html, register_page

register_page(__name__, path="/ventas", name="Ventas", order=1)

layout = html.Div([html.H2("Ventas"), ...])          # Puede ser también una función layout(**params)
```

> Los *callbacks* de cada página se definen en su propio archivo con `@callback` (no `@app.callback`).

---

## 14. Rendimiento

| Técnica | Cómo |
|---|---|
| Cargar los datos una sola vez | Léalos al iniciar el módulo, no dentro de cada *callback*. |
| Agregar antes de graficar | Envíe al navegador resúmenes, no millones de filas. |
| Caché de resultados | `flask_caching` con `@cache.memoize(timeout=300)`. |
| Actualizaciones parciales | `Patch()` en lugar de reconstruir la figura. |
| Evitar disparos innecesarios | `prevent_initial_call=True`, `State`, `dcc.Input(debounce=True)`. |
| Lógica sencilla en el navegador | `clientside_callback`. |
| Gráficos con muchos puntos | `render_mode="webgl"` / `go.Scattergl`. |
| Tareas largas | `background=True` con Diskcache o Celery. |

Caché con `flask-caching`:

```python
from flask_caching import Cache

cache = Cache(app.server, config={"CACHE_TYPE": "SimpleCache", "CACHE_DEFAULT_TIMEOUT": 300})

@cache.memoize()
def consultar_ventas(region: str, inicio: str, fin: str) -> pd.DataFrame:
    return pd.read_sql(...)
```

---

## 15. Despliegue en producción

`app.run(debug=True)` es solo para desarrollo. En producción, sirva la aplicación Flask subyacente (`app.server`) con un servidor WSGI:

```python
# app.py
app = Dash(__name__)
server = app.server            # Objeto Flask para Gunicorn
```

```bash
pip install gunicorn
gunicorn app:server --workers 4 --bind 0.0.0.0:8050          # Linux / macOS
pip install waitress
waitress-serve --port=8050 app:server                         # Windows
```

Docker:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8050
CMD ["gunicorn", "app:server", "--workers", "4", "--bind", "0.0.0.0:8050"]
```

Lista de comprobación:

- Desactive `debug` y no exponga trazas de error.
- Use variables de entorno para credenciales (`os.environ["DB_URL"]`).
- Añada autenticación (proxy inverso, `dash-auth` para casos sencillos o el sistema corporativo).
- Sirva detrás de un proxy (Nginx) con HTTPS.
- Si los datos cambian, recárguelos periódicamente o use caché con caducidad.

---

## 16. Ejemplo práctico completo

**Dashboard comercial**: aplicación de una sola página con filtros de región, categoría y periodo; cuatro indicadores clave con variación frente al periodo anterior; evolución temporal con agregación diaria, semanal o mensual; gráfico de barras por tienda que **filtra por clic** el resto del panel; mapa de calor de ventas por día y hora; tabla de detalle con formato condicional; exportación a CSV del resultado filtrado y botón para limpiar la selección. Usa `dcc.Store` para compartir el estado del filtro por clic y estilos CSS incrustados (sin dependencias adicionales).

```python
"""
Dashboard comercial con Dash.
Requiere: pip install dash pandas numpy
Ejecutar: python dashboard.py  →  http://127.0.0.1:8050
"""
import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import plotly.io as pio
from dash import Dash, Input, Output, State, callback, ctx, dash_table, dcc, html, no_update
from dash.dash_table.Format import Format, Group, Scheme

AZUL, AZUL_CLARO, ROJO, GRIS = "#1F4E78", "#9DC3E6", "#C00000", "#7F7F7F"
COLORES_REGION = {"Norte": "#1F4E78", "Centro": "#2E75B6", "Sur": "#C55A11", "Este": "#7F7F7F"}
TIENDAS = {"Norte": ["Bilbao", "Oviedo"], "Centro": ["Madrid", "Valladolid"],
           "Sur": ["Sevilla", "Málaga"], "Este": ["Valencia", "Barcelona"]}
CATEGORIAS = ["Electrónica", "Hogar", "Moda", "Deporte"]
DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]

pio.templates["panel"] = go.layout.Template(layout={
    "font": {"family": "Segoe UI, Arial, sans-serif", "size": 12, "color": "#262626"},
    "colorway": list(COLORES_REGION.values()), "separators": ",.",
    "plot_bgcolor": "white", "paper_bgcolor": "white",
    "margin": {"l": 50, "r": 20, "t": 50, "b": 40},
    "title": {"font": {"size": 15, "color": AZUL}, "x": 0.01},
    "xaxis": {"showgrid": False, "linecolor": "#D9D9D9"}, "yaxis": {"gridcolor": "#EEEEEE"},
    "legend": {"orientation": "h", "y": 1.08, "x": 0, "title": None},
})
pio.templates.default = "plotly_white+panel"


# ------------------------------------------------------------------ Datos (se cargan una vez)
def generar_pedidos(n: int = 40_000, semilla: int = 8) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    regiones = rng.choice(list(TIENDAS), n, p=[0.25, 0.35, 0.22, 0.18])
    tiendas = np.array([rng.choice(TIENDAS[r]) for r in regiones])
    dias = rng.integers(0, 731, n)
    fechas = pd.Timestamp("2024-01-01") + pd.to_timedelta(dias, unit="D")
    hora = np.clip(rng.normal(17, 3.5, n).round(), 9, 21).astype(int)
    categoria = rng.choice(CATEGORIAS, n, p=[0.3, 0.25, 0.3, 0.15])
    base = np.select([categoria == "Electrónica", categoria == "Hogar", categoria == "Deporte"], [140, 70, 60], 45)
    tendencia = 1 + dias / 730 * 0.15
    importe = (base * tendencia * rng.lognormal(0, 0.45, n)).round(2)
    return pd.DataFrame({
        "pedido": np.arange(100_000, 100_000 + n), "fecha": fechas.normalize() + pd.to_timedelta(hora, unit="h"),
        "region": regiones, "tienda": tiendas, "categoria": categoria, "importe": importe,
        "margen": (importe * rng.normal(0.3, 0.06, n).clip(0.05, 0.6)).round(2),
    }).sort_values("fecha", ignore_index=True)


DF = generar_pedidos()
FECHA_MIN, FECHA_MAX = DF["fecha"].min().date(), DF["fecha"].max().date()


def euros(valor: float, decimales: int = 0) -> str:
    return f"{valor:,.{decimales}f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def filtrar(region, categorias, inicio, fin, tienda=None) -> pd.DataFrame:
    datos = DF[DF["fecha"].between(pd.Timestamp(inicio), pd.Timestamp(fin) + pd.Timedelta(days=1))]
    if region and region != "Todas":
        datos = datos[datos["region"] == region]
    if categorias:
        datos = datos[datos["categoria"].isin(categorias)]
    if tienda:
        datos = datos[datos["tienda"] == tienda]
    return datos


# ------------------------------------------------------------------ Componentes de diseño
ESTILOS = """
body { margin: 0; background: #F4F6F9; font-family: 'Segoe UI', Arial, sans-serif; color: #262626; }
.cabecera { background: #1F4E78; color: white; padding: 14px 28px; display: flex; align-items: baseline; gap: 14px; }
.cabecera h1 { margin: 0; font-size: 22px; } .cabecera span { opacity: .75; font-size: 13px; }
.contenido { max-width: 1400px; margin: 0 auto; padding: 18px 24px; }
.tarjeta { background: white; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,.08); padding: 14px 16px; }
.filtros { display: grid; grid-template-columns: 1fr 2fr 1.4fr auto auto; gap: 16px; align-items: end; }
.filtros label { font-size: 12px; color: #595959; display: block; margin-bottom: 4px; }
.kpis { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; margin: 16px 0; }
.kpi-titulo { font-size: 12px; color: #7F7F7F; text-transform: uppercase; letter-spacing: .5px; }
.kpi-valor { font-size: 26px; font-weight: 700; color: #1F4E78; margin: 4px 0; }
.kpi-delta { font-size: 12px; }
.rejilla { display: grid; grid-template-columns: 2fr 1fr; gap: 16px; margin-bottom: 16px; }
.boton { background: #1F4E78; color: white; border: none; border-radius: 6px; padding: 9px 14px; cursor: pointer; }
.boton.secundario { background: white; color: #1F4E78; border: 1px solid #1F4E78; }
.aviso { font-size: 13px; color: #C55A11; margin-top: 8px; min-height: 18px; }
@media (max-width: 1000px) { .filtros, .kpis, .rejilla { grid-template-columns: 1fr; } }
"""


def tarjeta_kpi(id_: str, titulo: str) -> html.Div:
    return html.Div([html.Div(titulo, className="kpi-titulo"),
                     html.Div(id=f"{id_}-valor", className="kpi-valor"),
                     html.Div(id=f"{id_}-delta", className="kpi-delta")], className="tarjeta")


app = Dash(__name__, title="Dashboard comercial")
server = app.server
app.index_string = app.index_string.replace("{%css%}", "{%css%}<style>" + ESTILOS + "</style>")

app.layout = html.Div([
    html.Div([html.H1("Dashboard comercial"), html.Span("Datos simulados 2024–2025")], className="cabecera"),
    html.Div([
        # --- Filtros
        html.Div([
            html.Div([html.Label("Región"),
                      dcc.Dropdown(id="f-region", options=["Todas", *TIENDAS], value="Todas", clearable=False)]),
            html.Div([html.Label("Categorías"),
                      dcc.Dropdown(id="f-categorias", options=CATEGORIAS, multi=True, placeholder="Todas")]),
            html.Div([html.Label("Periodo"),
                      dcc.DatePickerRange(id="f-fechas", start_date="2025-01-01", end_date=str(FECHA_MAX),
                                          min_date_allowed=str(FECHA_MIN), max_date_allowed=str(FECHA_MAX),
                                          display_format="DD/MM/YYYY", first_day_of_week=1)]),
            html.Button("Limpiar tienda", id="b-limpiar", className="boton secundario"),
            html.Button("Exportar CSV", id="b-csv", className="boton"),
        ], className="tarjeta filtros"),
        html.Div(id="aviso-tienda", className="aviso"),

        # --- Indicadores
        html.Div([tarjeta_kpi("k-ventas", "Ventas"), tarjeta_kpi("k-pedidos", "Pedidos"),
                  tarjeta_kpi("k-ticket", "Ticket medio"), tarjeta_kpi("k-margen", "Margen")], className="kpis"),

        # --- Gráficos
        html.Div([
            html.Div([dcc.RadioItems(id="f-agregacion", inline=True, value="W",
                                     options=[{"label": " Día ", "value": "D"}, {"label": " Semana ", "value": "W"},
                                              {"label": " Mes ", "value": "MS"}],
                                     style={"float": "right", "fontSize": "13px"}),
                      dcc.Loading(dcc.Graph(id="g-evolucion", config={"displaylogo": False}))], className="tarjeta"),
            html.Div(dcc.Graph(id="g-tiendas", config={"displaylogo": False}), className="tarjeta"),
        ], className="rejilla"),
        html.Div([
            html.Div(dcc.Graph(id="g-calor", config={"displaylogo": False}), className="tarjeta"),
            html.Div(dcc.Graph(id="g-categorias", config={"displaylogo": False}), className="tarjeta"),
        ], className="rejilla"),

        # --- Tabla
        html.Div([
            html.H3("Detalle por tienda y categoría", style={"color": AZUL, "fontSize": "15px", "marginTop": 0}),
            dash_table.DataTable(
                id="t-detalle", page_size=12, sort_action="native", filter_action="native",
                style_table={"overflowX": "auto"},
                style_header={"backgroundColor": AZUL, "color": "white", "fontWeight": "bold", "border": "none"},
                style_cell={"fontFamily": "Segoe UI, Arial", "fontSize": "13px", "padding": "6px 10px", "textAlign": "left",
                            "border": "none", "borderBottom": "1px solid #EEEEEE"},
                style_cell_conditional=[{"if": {"column_id": c}, "textAlign": "right"}
                                        for c in ["pedidos", "ventas", "ticket", "margen_pct"]],
                style_data_conditional=[
                    {"if": {"row_index": "odd"}, "backgroundColor": "#F8F9FB"},
                    {"if": {"filter_query": "{margen_pct} < 0.27", "column_id": "margen_pct"},
                     "color": ROJO, "fontWeight": "bold"},
                    {"if": {"filter_query": "{margen_pct} >= 0.32", "column_id": "margen_pct"},
                     "color": "#2E7D32", "fontWeight": "bold"},
                ],
                columns=[
                    {"name": "Región", "id": "region"}, {"name": "Tienda", "id": "tienda"},
                    {"name": "Categoría", "id": "categoria"},
                    {"name": "Pedidos", "id": "pedidos", "type": "numeric",
                     "format": Format(group=Group.yes, group_delimiter=".")},
                    {"name": "Ventas (€)", "id": "ventas", "type": "numeric",
                     "format": Format(precision=0, scheme=Scheme.fixed, group=Group.yes,
                                      group_delimiter=".", decimal_delimiter=",")},
                    {"name": "Ticket (€)", "id": "ticket", "type": "numeric",
                     "format": Format(precision=2, scheme=Scheme.fixed, decimal_delimiter=",")},
                    {"name": "Margen", "id": "margen_pct", "type": "numeric",
                     "format": Format(precision=1, scheme=Scheme.percentage, decimal_delimiter=",")},
                ]),
        ], className="tarjeta"),
        dcc.Store(id="s-tienda"),
        dcc.Download(id="d-csv"),
    ], className="contenido"),
])


# ------------------------------------------------------------------ Callbacks
@callback(
    Output("s-tienda", "data"),
    Input("g-tiendas", "clickData"), Input("b-limpiar", "n_clicks"), Input("f-region", "value"),
    prevent_initial_call=True,
)
def seleccionar_tienda(click, _limpiar, _region):
    """Guarda la tienda pulsada en el gráfico de barras; se limpia con el botón o al cambiar de región."""
    if ctx.triggered_id == "g-tiendas" and click:
        return click["points"][0]["y"]
    return None


@callback(
    Output("k-ventas-valor", "children"), Output("k-ventas-delta", "children"),
    Output("k-pedidos-valor", "children"), Output("k-pedidos-delta", "children"),
    Output("k-ticket-valor", "children"), Output("k-ticket-delta", "children"),
    Output("k-margen-valor", "children"), Output("k-margen-delta", "children"),
    Output("aviso-tienda", "children"),
    Input("f-region", "value"), Input("f-categorias", "value"),
    Input("f-fechas", "start_date"), Input("f-fechas", "end_date"), Input("s-tienda", "data"),
)
def actualizar_kpis(region, categorias, inicio, fin, tienda):
    actual = filtrar(region, categorias, inicio, fin, tienda)
    duracion = pd.Timestamp(fin) - pd.Timestamp(inicio) + pd.Timedelta(days=1)
    anterior = filtrar(region, categorias, pd.Timestamp(inicio) - duracion,
                       pd.Timestamp(inicio) - pd.Timedelta(days=1), tienda)

    def resumen(d: pd.DataFrame) -> dict:
        ventas = d["importe"].sum()
        return {"ventas": ventas, "pedidos": len(d), "ticket": ventas / len(d) if len(d) else 0,
                "margen": d["margen"].sum() / ventas if ventas else 0}

    a, b = resumen(actual), resumen(anterior)

    def delta(clave: str, en_puntos: bool = False):
        if not b[clave]:
            return html.Span("Sin periodo anterior comparable", style={"color": GRIS})
        cambio = (a[clave] - b[clave]) * 100 if en_puntos else (a[clave] / b[clave] - 1) * 100
        color, flecha = ("#2E7D32", "▲") if cambio >= 0 else (ROJO, "▼")
        unidad = " p.p." if en_puntos else " %"
        return html.Span(f"{flecha} {abs(cambio):.1f}".replace(".", ",") + f"{unidad} vs. periodo anterior",
                         style={"color": color})

    aviso = f"Filtrando por la tienda «{tienda}» (clic en el gráfico de tiendas)." if tienda else ""
    return (euros(a["ventas"]), delta("ventas"),
            f"{a['pedidos']:,}".replace(",", "."), delta("pedidos"),
            euros(a["ticket"], 2), delta("ticket"),
            f"{a['margen']:.1%}".replace(".", ","), delta("margen", en_puntos=True),
            aviso)


@callback(
    Output("g-evolucion", "figure"), Output("g-tiendas", "figure"),
    Output("g-calor", "figure"), Output("g-categorias", "figure"), Output("t-detalle", "data"),
    Input("f-region", "value"), Input("f-categorias", "value"), Input("f-fechas", "start_date"),
    Input("f-fechas", "end_date"), Input("f-agregacion", "value"), Input("s-tienda", "data"),
)
def actualizar_graficos(region, categorias, inicio, fin, agregacion, tienda):
    datos = filtrar(region, categorias, inicio, fin)
    seleccion = datos if not tienda else datos[datos["tienda"] == tienda]

    # Evolución
    serie = (seleccion.set_index("fecha").groupby("region")["importe"]
             .resample(agregacion).sum().reset_index())
    nombre = {"D": "diarias", "W": "semanales", "MS": "mensuales"}[agregacion]
    fig_evol = px.area(serie, x="fecha", y="importe", color="region", color_discrete_map=COLORES_REGION,
                       title=f"Ventas {nombre}" + (f" · {tienda}" if tienda else ""),
                       labels={"importe": "Ventas (€)", "fecha": ""})
    fig_evol.update_traces(line_width=1.5, hovertemplate="%{x|%d/%m/%Y}<br>%{y:,.0f} €")
    fig_evol.update_layout(hovermode="x unified", height=340, legend_title_text=None)

    # Tiendas (clic para filtrar)
    por_tienda = datos.groupby(["tienda", "region"], as_index=False)["importe"].sum().sort_values("importe")
    colores = [COLORES_REGION[r] if (tienda is None or t == tienda) else "#D9D9D9"
               for t, r in zip(por_tienda["tienda"], por_tienda["region"])]
    fig_tiendas = go.Figure(go.Bar(x=por_tienda["importe"], y=por_tienda["tienda"], orientation="h",
                                   marker_color=colores, text=[euros(v) for v in por_tienda["importe"]],
                                   textposition="inside", insidetextanchor="end",
                                   hovertemplate="%{y}: %{x:,.0f} €<extra></extra>"))
    fig_tiendas.update_layout(title="Ventas por tienda (clic para filtrar)", height=340,
                              xaxis={"showticklabels": False, "showgrid": False}, yaxis={"showgrid": False},
                              margin={"l": 90})

    # Mapa de calor día × hora
    calor = (seleccion.assign(dia=seleccion["fecha"].dt.dayofweek, hora=seleccion["fecha"].dt.hour)
             .pivot_table(index="dia", columns="hora", values="importe", aggfunc="sum", fill_value=0)
             .reindex(range(7), fill_value=0))
    fig_calor = px.imshow(calor.values / 1000, x=[f"{h}h" for h in calor.columns], y=DIAS,
                          color_continuous_scale="Blues", aspect="auto",
                          labels={"color": "Miles de €", "x": "Hora", "y": ""},
                          title="Ventas por día de la semana y hora")
    fig_calor.update_layout(height=320)

    # Categorías (anillo)
    por_categoria = seleccion.groupby("categoria", as_index=False)["importe"].sum()
    fig_cat = px.pie(por_categoria, names="categoria", values="importe", hole=0.55,
                     title="Reparto por categoría",
                     color_discrete_sequence=[AZUL, "#2E75B6", AZUL_CLARO, "#C55A11"])
    fig_cat.update_traces(textinfo="percent", hovertemplate="%{label}: %{value:,.0f} €<extra></extra>")
    fig_cat.update_layout(height=320, legend={"orientation": "v", "y": 0.5, "x": 1})

    # Tabla
    tabla = (seleccion.groupby(["region", "tienda", "categoria"], as_index=False)
             .agg(pedidos=("pedido", "count"), ventas=("importe", "sum"), margen=("margen", "sum")))
    tabla["ticket"] = (tabla["ventas"] / tabla["pedidos"]).round(2)
    tabla["margen_pct"] = (tabla["margen"] / tabla["ventas"]).round(4)
    tabla = tabla.drop(columns="margen").sort_values("ventas", ascending=False)

    return fig_evol, fig_tiendas, fig_calor, fig_cat, tabla.to_dict("records")


@callback(
    Output("d-csv", "data"),
    Input("b-csv", "n_clicks"),
    State("f-region", "value"), State("f-categorias", "value"),
    State("f-fechas", "start_date"), State("f-fechas", "end_date"), State("s-tienda", "data"),
    prevent_initial_call=True,
)
def exportar_csv(_n, region, categorias, inicio, fin, tienda):
    datos = filtrar(region, categorias, inicio, fin, tienda)
    if datos.empty:
        return no_update
    return dcc.send_data_frame(datos.to_csv, f"pedidos_{inicio}_{fin}.csv",
                               index=False, sep=";", decimal=",", encoding="utf-8-sig")


if __name__ == "__main__":
    app.run(debug=True)
```

Ejecute `python dashboard.py` y abra http://127.0.0.1:8050. Pruebe a cambiar los filtros, hacer clic en una tienda del gráfico de barras y exportar el resultado.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ObsoleteAttributeException: app.run_server has been replaced by app.run` | API antigua. | Use `app.run(debug=True)`. |
| `DuplicateIdError` | Dos componentes con el mismo `id`. | Asigne identificadores únicos. |
| `A nonexistent object was used in an Input/Output` | El `id` del *callback* no existe en el *layout*. | Revise el `id`; para componentes creados dinámicamente, `app = Dash(..., suppress_callback_exceptions=True)`. |
| `Duplicate callback outputs` | Dos *callbacks* escriben la misma salida. | Agrúpelos en uno con `ctx.triggered_id` o use `allow_duplicate=True` en el `Output`. |
| `Callback error: ... expected N outputs, got M` | Número de valores devueltos distinto de los `Output`. | Devuelva una tupla con un valor por salida. |
| El *callback* se ejecuta al cargar y falla con `None` | Ejecución inicial con valores vacíos. | `prevent_initial_call=True` o compruebe `None` y use `PreventUpdate`. |
| `TypeError: Object of type Timestamp is not JSON serializable` | Se devolvieron tipos no serializables. | Convierta: `df.to_dict("records")` con fechas como texto, `int(valor)`. |
| `Circular dependency` | Un *callback* depende de su propia salida. | Combine la lógica en un solo *callback*. |
| Los datos de un usuario aparecen a otro | Se usaron variables globales modificables. | Guarde el estado en `dcc.Store` o en las entradas/salidas. |
| El CSS de `assets/` no se aplica | Carpeta mal ubicada o con otro nombre. | `assets/` junto al archivo que crea `Dash(__name__)`. |
| `Address already in use` | El puerto 8050 está ocupado. | `app.run(port=8051)` o cierre el proceso anterior. |
| La aplicación va lenta | Lectura de datos en cada *callback* o figuras enormes. | Cargue los datos una vez, agregue, use caché y `Patch`. |
| El selector de fechas devuelve texto | `DatePickerRange` entrega cadenas `"AAAA-MM-DD"`. | Conviértalas con `pd.Timestamp(valor)`. |

---

## 18. Buenas prácticas

- Separe **datos**, **diseño** y ***callbacks*** (en aplicaciones grandes, en módulos distintos).
- Asigne `id` descriptivos y coherentes (`f-region`, `g-evolucion`, `k-ventas`).
- Cargue y prepare los datos una sola vez al iniciar; filtre en los *callbacks*.
- No modifique variables globales dentro de *callbacks*: use `dcc.Store`.
- Agrupe en un *callback* las salidas que dependen de las mismas entradas para evitar cálculos repetidos.
- Use `State` y `prevent_initial_call=True` para evitar ejecuciones innecesarias.
- Defina una plantilla de Plotly y una hoja de estilos comunes para todo el panel.
- Muestre indicadores de carga (`dcc.Loading`) en cálculos de más de un segundo.
- Informe al usuario de los filtros activos y ofrezca una forma de limpiarlos.
- En producción: Gunicorn/Waitress, `debug=False`, HTTPS, autenticación y caché.

---

## 19. Referencia rápida

```python
from dash import Dash, html, dcc, dash_table, Input, Output, State, callback, ctx, no_update, Patch, ALL, MATCH
app = Dash(__name__, title=, external_stylesheets=[...], use_pages=False, suppress_callback_exceptions=False)
server = app.server
app.layout = html.Div([...])
app.run(debug=True, port=8050)

html.Div / H1 / P / Span / Button(n_clicks=0) / Img / A / Hr       (children, id, style={}, className)
dcc.Graph(figure=, config=) / Dropdown(options, value, multi) / RadioItems / Checklist / Slider / RangeSlider
dcc.Input(debounce=True) / DatePickerRange(start_date, end_date) / Tabs + Tab / Loading / Store(data) / Interval
dcc.Upload(contents) / Download + dcc.send_data_frame(df.to_csv, "f.csv") / Markdown / Location / Link
dash_table.DataTable(data=df.to_dict("records"), columns=[...], sort_action="native", filter_action="native")

@callback(Output(id, prop), Input(id, prop), State(id, prop), prevent_initial_call=True)
def f(entrada, estado): return valor           # o tupla con varios valores
raise PreventUpdate / return no_update / ctx.triggered_id / Patch() / Output(..., allow_duplicate=True)
dcc.Graph: clickData, hoverData, selectedData, relayoutData
clientside_callback("function(x) {...}", Output(...), Input(...))
register_page(__name__, path="/ventas") + page_container
gunicorn app:server --workers 4
```

---

## 20. Ejercicios propuestos

1. Cree una aplicación con un desplegable de países y un gráfico de líneas de su población histórica.
2. Añada desplegables en cascada: continente → país → indicador.
3. Construya un formulario de alta de registros con validación y botón «Guardar» (`State`) que añada filas a una tabla.
4. Diseñe una aplicación con `dcc.Upload` que cargue un CSV, muestre las columnas en desplegables y genere un gráfico con las columnas elegidas.
5. Convierta el ejemplo completo en una aplicación de varias páginas (resumen, tiendas y clientes) con Dash Bootstrap Components y despliéguela con Gunicorn en Docker.

---

## 21. Referencias

- Documentación oficial: https://dash.plotly.com/
- Tutorial: https://dash.plotly.com/tutorial
- Componentes básicos (`dcc`): https://dash.plotly.com/dash-core-components
- *Callbacks*: https://dash.plotly.com/basic-callbacks
- DataTable: https://dash.plotly.com/datatable
- Dash Bootstrap Components: https://www.dash-bootstrap-components.com/
- Dash AG Grid: https://dash.plotly.com/dash-ag-grid
