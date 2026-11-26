# Manual de uso: Streamlit

> **Categoría:** Visualización de datos · **Utilidad:** Crear aplicaciones y dashboards de datos rápidamente
> **Tipo:** Librería externa (requiere instalación con `pip`)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Texto y contenido](#5-texto-y-contenido)
6. [Datos: tablas, métricas y edición](#6-datos-tablas-métricas-y-edición)
7. [Gráficos](#7-gráficos)
8. [*Widgets* de entrada](#8-widgets-de-entrada)
9. [Diseño de la página](#9-diseño-de-la-página)
10. [Estado de la sesión y *callbacks*](#10-estado-de-la-sesión-y-callbacks)
11. [Formularios, fragmentos y diálogos](#11-formularios-fragmentos-y-diálogos)
12. [Caché y rendimiento](#12-caché-y-rendimiento)
13. [Archivos, conexiones y secretos](#13-archivos-conexiones-y-secretos)
14. [Aplicaciones de varias páginas](#14-aplicaciones-de-varias-páginas)
15. [Configuración, temas y despliegue](#15-configuración-temas-y-despliegue)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Streamlit** convierte scripts de Python en **aplicaciones web interactivas** con muy poco código. No hay que definir rutas, plantillas HTML ni *callbacks*: se escribe el script de arriba abajo, cada llamada a `st.*` añade un elemento a la página, y cada vez que el usuario cambia un control **el script se vuelve a ejecutar** completo con los nuevos valores.

**Ventajas:**
- Curva de aprendizaje mínima: una aplicación útil en 10–20 líneas.
- Componentes listos: tablas editables, métricas, gráficos, mapas, carga de archivos, chat…
- Compatible con Matplotlib, Plotly, Altair, Bokeh (vía componentes), pydeck, Folium (vía `streamlit-folium`)…
- Recarga automática al guardar el archivo.
- Caché integrada para datos y modelos.
- Despliegue gratuito en Streamlit Community Cloud, o en Docker/servidor propio.

**Limitaciones:**
- El modelo de reejecución completa exige usar **caché** y **estado de sesión** para no repetir cálculos.
- Menos control sobre el diseño exacto que Dash o una aplicación web a medida.
- Cada usuario mantiene una sesión en el servidor (WebSocket): requiere más recursos con muchos usuarios simultáneos.

| Si necesita… | Opción |
|---|---|
| Prototipo, herramienta interna o demo de datos en poco tiempo | **Streamlit** |
| Dashboard de producción con diseño muy controlado | Dash |
| Gráfico interactivo para compartir como archivo | Plotly / Bokeh / Altair |
| Aplicación de escritorio | Tkinter / PySide6 |

---

## 2. Instalación e importación

```bash
pip install streamlit
streamlit hello                         # Aplicación de demostración
```

Comprobar la versión:

```bash
streamlit version
```

Importación habitual:

```python
import streamlit as st
import pandas as pd
import numpy as np
```

Ejecutar una aplicación:

```bash
streamlit run app.py                    # Abre http://localhost:8501
streamlit run app.py --server.port 8080
python -m streamlit run app.py          # Alternativa si «streamlit» no está en el PATH
```

> Una aplicación Streamlit **no** se ejecuta con `python app.py`: hay que usar `streamlit run`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Script de arriba abajo | La página se construye en el orden en que se ejecutan las llamadas `st.*`. |
| Reejecución (*rerun*) | Cada interacción con un *widget* vuelve a ejecutar **todo** el script. |
| *Widget* | Control de entrada (`st.selectbox`, `st.slider`…) que **devuelve** su valor actual. |
| Estado de sesión (`st.session_state`) | Diccionario que conserva valores entre reejecuciones para cada usuario. |
| Caché (`@st.cache_data`, `@st.cache_resource`) | Evita repetir cálculos costosos en cada reejecución. |
| Contenedores | Zonas de la página: `st.sidebar`, `st.columns`, `st.tabs`, `st.expander`, `st.container`. |
| Fragmento (`@st.fragment`) | Parte de la página que se reejecuta sola, sin repetir todo el script. |
| `key` | Identificador único de un *widget*; permite leerlo o modificarlo desde `st.session_state`. |

**El modelo de ejecución:**

```
Usuario mueve un deslizador
        │
        ▼
Streamlit vuelve a ejecutar app.py de principio a fin
        │   - st.slider(...) devuelve el nuevo valor
        │   - las funciones con caché devuelven el resultado guardado
        │   - st.session_state conserva lo almacenado
        ▼
La página se actualiza con el nuevo resultado
```

---

## 4. Primera aplicación

```python
# app.py
import pandas as pd
import streamlit as st

st.set_page_config(page_title="Ventas", page_icon="📊", layout="wide")

st.title("Ventas por región")

df = pd.DataFrame({
    "mes": pd.date_range("2025-01-01", periods=12, freq="MS"),
    "Norte": [120, 135, 128, 160, 172, 190, 185, 170, 165, 180, 195, 210],
    "Sur": [90, 95, 110, 105, 130, 142, 150, 148, 139, 145, 160, 175],
}).set_index("mes")

regiones = st.multiselect("Regiones", df.columns.tolist(), default=df.columns.tolist())
st.line_chart(df[regiones])

st.metric("Ventas totales", f"{df[regiones].sum().sum():,.0f} k€".replace(",", "."))
```

```bash
streamlit run app.py
```

Al cambiar la selección, el script se reejecuta y el gráfico y la métrica se actualizan.

---

## 5. Texto y contenido

```python
st.title("Título principal")
st.header("Encabezado", divider="blue")
st.subheader("Subencabezado")
st.markdown("Texto con **negrita**, *cursiva*, `código` y [enlaces](https://streamlit.io). :blue[Color] y :material/check_circle:")
st.caption("Texto pequeño, para notas o fuentes.")
st.text("Texto de ancho fijo")
st.code("print('hola')", language="python")
st.latex(r"\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i")
st.divider()
st.write("`st.write` muestra casi cualquier cosa:", df, {"clave": "valor"}, 42)
st.json({"region": "Norte", "ventas": 120})
st.image("logo.png", width=200, caption="Logotipo")
st.badge("Nuevo", color="green")
```

Mensajes de estado:

```python
st.success("Datos guardados correctamente.")
st.info("Los datos se actualizan cada hora.")
st.warning("Hay registros sin fecha.")
st.error("No se pudo conectar con la base de datos.")
st.exception(error)                                   # Muestra la traza de una excepción
st.toast("Exportación completada", icon="✅")          # Aviso temporal en una esquina
```

> **Magic:** una variable o expresión suelta en una línea se muestra automáticamente como si se llamara a `st.write()`.

---

## 6. Datos: tablas, métricas y edición

### 6.1 Tablas

```python
st.dataframe(df)                                         # Interactiva: ordenar, buscar, ampliar, descargar
st.dataframe(df, hide_index=True, height=400, width="stretch")
st.table(df.head(10))                                    # Estática
```

### 6.2 Configuración de columnas (`st.column_config`)

```python
st.dataframe(
    df,
    hide_index=True,
    column_config={
        "producto": st.column_config.TextColumn("Producto", width="medium"),
        "precio": st.column_config.NumberColumn("Precio", format="%.2f €"),
        "ventas": st.column_config.NumberColumn("Ventas", format="localized"),
        "margen": st.column_config.ProgressColumn("Margen", min_value=0, max_value=1, format="percent"),
        "evolucion": st.column_config.LineChartColumn("Últimos 12 meses", y_min=0),
        "fecha": st.column_config.DateColumn("Fecha", format="DD/MM/YYYY"),
        "web": st.column_config.LinkColumn("Web"),
        "activo": st.column_config.CheckboxColumn("Activo"),
        "foto": st.column_config.ImageColumn("Foto"),
        "id_interno": None,                                # Ocultar columna
    },
)
```

### 6.3 Tablas editables (`st.data_editor`)

```python
editado = st.data_editor(
    df,
    num_rows="dynamic",                                    # Permite añadir y borrar filas
    disabled=["id"],                                       # Columnas no editables
    column_config={"categoria": st.column_config.SelectboxColumn("Categoría", options=["A", "B", "C"])},
    key="editor",
)
if st.button("Guardar cambios"):
    guardar_en_bd(editado)
```

### 6.4 Selección de filas

```python
evento = st.dataframe(df, on_select="rerun", selection_mode="multi-row", hide_index=True)
filas = evento.selection.rows                              # Índices de las filas seleccionadas
st.write(df.iloc[filas])
```

### 6.5 Métricas

```python
st.metric("Ventas", "1,25 M€", delta="8,3 %")
st.metric("Devoluciones", 42, delta=-5, delta_color="inverse")      # Bajar es bueno
st.metric("Pedidos", "3.420", delta="+120", border=True,
          chart_data=[310, 290, 330, 360, 342], chart_type="area")   # Con minigráfico
```

---

## 7. Gráficos

### 7.1 Gráficos nativos (rápidos)

```python
st.line_chart(df, x="fecha", y=["Norte", "Sur"])
st.area_chart(df, x="fecha", y="ventas", color="region", stack="normalize")
st.bar_chart(df, x="region", y="ventas", color="canal", horizontal=True)
st.scatter_chart(df, x="precio", y="unidades", color="categoria", size="margen")
st.map(df_coordenadas, latitude="lat", longitude="lon", size="ventas", zoom=5)
```

### 7.2 Otras librerías

```python
st.plotly_chart(fig_plotly)                                 # Plotly (ocupa el ancho disponible)
st.altair_chart(grafico_altair)                             # Altair / Vega-Lite
st.vega_lite_chart(df, especificacion_json)
st.pyplot(fig_matplotlib)                                   # Matplotlib / Seaborn
st.pydeck_chart(mapa_pydeck)                                # Mapas 3D
st.graphviz_chart(diagrama)

# Folium (pip install streamlit-folium)
from streamlit_folium import st_folium
st_folium(mapa_folium, width=700, height=450)
```

> Los gráficos se adaptan al ancho del contenedor por defecto (`width="stretch"`). El parámetro antiguo `use_container_width=True` está obsoleto.

### 7.3 Gráficos que responden a la selección

```python
evento = st.plotly_chart(fig, on_select="rerun", selection_mode=("points", "box"), key="grafico")
puntos = evento.selection.points                            # Lista de puntos seleccionados
if puntos:
    st.write("Seleccionados:", [p["x"] for p in puntos])

evento = st.altair_chart(grafico.add_params(seleccion), on_select="rerun")
st.write(evento.selection)
```

---

## 8. *Widgets* de entrada

Cada *widget* **devuelve** su valor actual:

```python
nombre = st.text_input("Nombre", placeholder="Escriba su nombre")
clave = st.text_input("Contraseña", type="password")
comentario = st.text_area("Comentario", height=120)
cantidad = st.number_input("Cantidad", min_value=0, max_value=100, value=10, step=1)
precio = st.number_input("Precio (€)", min_value=0.0, value=9.99, format="%.2f")

region = st.selectbox("Región", ["Norte", "Sur", "Este"], index=0)
productos = st.multiselect("Productos", ["A", "B", "C"], default=["A"])
canal = st.radio("Canal", ["Online", "Tienda"], horizontal=True)
vista = st.segmented_control("Vista", ["Diaria", "Semanal", "Mensual"], default="Mensual")
etiquetas = st.pills("Etiquetas", ["Urgente", "VIP", "Nuevo"], selection_mode="multi")

umbral = st.slider("Umbral", 0, 100, 50)
rango = st.slider("Rango de precios", 0.0, 500.0, (50.0, 300.0))
talla = st.select_slider("Talla", options=["XS", "S", "M", "L", "XL"], value="M")

acepta = st.checkbox("Acepto las condiciones")
activo = st.toggle("Modo detallado")

fecha = st.date_input("Fecha", value=pd.Timestamp("2025-06-30"), format="DD/MM/YYYY")
periodo = st.date_input("Periodo", value=(pd.Timestamp("2025-01-01"), pd.Timestamp("2025-12-31")))
hora = st.time_input("Hora")
color = st.color_picker("Color", "#1F4E78")

if st.button("Calcular", type="primary", icon=":material/calculate:"):
    st.write("Calculando…")

archivo = st.file_uploader("Suba un CSV", type=["csv", "xlsx"])
foto = st.camera_input("Haga una foto")
valoracion = st.feedback("stars")
```

| Parámetro común | Efecto |
|---|---|
| `key="nombre"` | Identificador único (necesario si hay dos *widgets* iguales). |
| `help="Texto"` | Icono de ayuda con explicación. |
| `disabled=True` | Desactivado. |
| `label_visibility="collapsed"` | Oculta la etiqueta. |
| `on_change=funcion` | *Callback* al cambiar el valor (véase la sección 10). |

> `st.button` devuelve `True` **solo en la reejecución inmediata** al clic. Para recordar que se pulsó, guárdelo en `st.session_state`.

---

## 9. Diseño de la página

### 9.1 Configuración general

```python
st.set_page_config(
    page_title="Panel de ventas",          # Título de la pestaña
    page_icon="📊",                         # Emoji, imagen o ":material/..."
    layout="wide",                          # "centered" (por defecto) o "wide"
    initial_sidebar_state="expanded",
)
st.logo("logo.png")                         # Logotipo en la barra lateral y cabecera
```

### 9.2 Barra lateral

```python
with st.sidebar:
    st.header("Filtros")
    region = st.selectbox("Región", regiones)
    fechas = st.date_input("Periodo", value=(inicio, fin))

region = st.sidebar.selectbox("Región", regiones)          # Forma alternativa
```

### 9.3 Columnas

```python
col1, col2, col3 = st.columns(3)
col1.metric("Ventas", "1,2 M€")
col2.metric("Pedidos", "8.540")
col3.metric("Ticket", "140 €")

izquierda, derecha = st.columns([2, 1], gap="large", vertical_alignment="center")   # Proporciones
with izquierda:
    st.plotly_chart(fig)
with derecha:
    st.dataframe(resumen)
```

### 9.4 Pestañas, expansores y ventanas emergentes

```python
resumen, detalle, datos = st.tabs(["📈 Resumen", "🔍 Detalle", "🗂️ Datos"])
with resumen:
    st.line_chart(df)

with st.expander("Ver datos de origen"):
    st.dataframe(df)

with st.popover("Opciones avanzadas"):
    escala_log = st.checkbox("Escala logarítmica")
```

### 9.5 Contenedores

```python
with st.container(border=True):                           # Tarjeta con borde
    st.subheader("Indicadores")
    ...

with st.container(horizontal=True, gap="medium"):         # Elementos en fila
    st.button("Aceptar"); st.button("Cancelar")

marcador = st.empty()                                     # Hueco que se rellena o sustituye después
marcador.info("Cargando…")
marcador.success("Listo")
```

### 9.6 Indicadores de progreso

```python
with st.spinner("Calculando…"):
    resultado = calculo_largo()

barra = st.progress(0, text="Procesando")
for i in range(100):
    barra.progress(i + 1, text=f"Procesando {i + 1} %")

with st.status("Descargando datos…", expanded=True) as estado:
    st.write("Conectando con la API…")
    st.write("Transformando…")
    estado.update(label="Completado", state="complete")
```

---

## 10. Estado de la sesión y *callbacks*

### 10.1 `st.session_state`

Conserva valores entre reejecuciones, **por usuario**:

```python
if "contador" not in st.session_state:                 # Inicializar una sola vez
    st.session_state.contador = 0

if st.button("Sumar"):
    st.session_state.contador += 1

st.write("Contador:", st.session_state.contador)
```

### 10.2 *Widgets* vinculados al estado (`key`)

```python
st.text_input("Cliente", key="cliente")
st.write(st.session_state.cliente)                     # El valor del widget está en el estado

if st.button("Limpiar"):
    st.session_state.cliente = ""                      # Modificar antes de que se dibuje el widget…
    st.rerun()                                         # …o forzar la reejecución
```

### 10.3 *Callbacks* (`on_change`, `on_click`)

Se ejecutan **antes** de la reejecución del script, lo que evita problemas de orden:

```python
def restablecer_filtros():
    st.session_state.region = "Todas"
    st.session_state.categorias = []

st.selectbox("Región", ["Todas", "Norte", "Sur"], key="region")
st.multiselect("Categorías", ["A", "B"], key="categorias")
st.button("Restablecer filtros", on_click=restablecer_filtros)
```

```python
def agregar_tarea(texto: str):
    st.session_state.tareas.append(texto)

st.button("Añadir", on_click=agregar_tarea, args=("Revisar informe",))
```

### 10.4 Parámetros de la URL

```python
region = st.query_params.get("region", "Todas")        # ?region=Norte
st.query_params["region"] = "Sur"                      # Actualiza la URL (enlaces compartibles)
```

---

## 11. Formularios, fragmentos y diálogos

### 11.1 Formularios (`st.form`)

Los *widgets* de un formulario **no** reejecutan el script hasta pulsar el botón de envío:

```python
with st.form("alta_cliente", clear_on_submit=True):
    nombre = st.text_input("Nombre *")
    email = st.text_input("Email *")
    ciudad = st.selectbox("Ciudad", ["Madrid", "Barcelona", "Sevilla"])
    vip = st.checkbox("Cliente VIP")
    enviado = st.form_submit_button("Guardar", type="primary")

if enviado:
    if not nombre or "@" not in email:
        st.error("Revise los campos obligatorios.")
    else:
        guardar_cliente(nombre, email, ciudad, vip)
        st.success(f"Cliente «{nombre}» guardado.")
```

### 11.2 Fragmentos (`@st.fragment`)

Solo se reejecuta la función, no toda la página. Útil para zonas independientes o que se refrescan solas:

```python
@st.fragment
def filtro_tabla(df):
    texto = st.text_input("Buscar en la tabla")
    st.dataframe(df[df["producto"].str.contains(texto, case=False)])

@st.fragment(run_every="10s")                            # Se actualiza cada 10 segundos
def pedidos_en_vivo():
    st.metric("Pedidos de hoy", consultar_pedidos_hoy())

filtro_tabla(df)
pedidos_en_vivo()
```

### 11.3 Diálogos modales (`@st.dialog`)

```python
@st.dialog("Confirmar eliminación")
def confirmar(cliente):
    st.write(f"¿Eliminar a **{cliente}**? Esta acción no se puede deshacer.")
    if st.button("Eliminar", type="primary"):
        eliminar(cliente)
        st.rerun()                                     # Cierra el diálogo y actualiza la página

if st.button("Eliminar cliente"):
    confirmar("Ana López")
```

---

## 12. Caché y rendimiento

Como el script se reejecuta en cada interacción, las operaciones costosas deben almacenarse en caché.

| Decorador | Para | Devuelve |
|---|---|---|
| `@st.cache_data` | Datos: lectura de archivos, consultas, transformaciones, resultados serializables. | Una **copia** del resultado (seguro de modificar). |
| `@st.cache_resource` | Recursos compartidos: conexiones a bases de datos, modelos de ML, clientes de API. | El **mismo objeto** para todos los usuarios. |

```python
@st.cache_data(ttl="1h", show_spinner="Cargando ventas…")
def cargar_ventas(ruta: str) -> pd.DataFrame:
    return pd.read_csv(ruta, parse_dates=["fecha"])

@st.cache_data
def resumir(df: pd.DataFrame, region: str) -> pd.DataFrame:     # Se cachea por cada combinación de argumentos
    return df[df["region"] == region].groupby("mes")["importe"].sum().reset_index()

@st.cache_resource
def conexion_bd():
    import sqlalchemy
    return sqlalchemy.create_engine(st.secrets["db"]["url"])

@st.cache_resource
def cargar_modelo():
    import joblib
    return joblib.load("modelo.pkl")
```

| Opción | Efecto |
|---|---|
| `ttl="10m"` / `ttl=3600` | Caducidad de la caché. |
| `max_entries=100` | Número máximo de resultados guardados. |
| `show_spinner=False` | Sin indicador de carga. |
| Argumento con guion bajo (`_conexion`) | Se excluye del cálculo de la clave de caché. |

Vaciar la caché:

```python
cargar_ventas.clear()           # Una función
st.cache_data.clear()           # Toda la caché de datos
```

> El menú de la aplicación (⋮ → *Clear cache*) y la tecla **C** también vacían la caché durante el desarrollo.

Otras recomendaciones de rendimiento:

- Filtre y agregue en pandas antes de mostrar; no envíe millones de filas a `st.dataframe`.
- Use `st.form` para agrupar varios cambios en una sola reejecución.
- Use `@st.fragment` para que una zona interactiva no reejecute toda la página.

---

## 13. Archivos, conexiones y secretos

### 13.1 Subir y descargar archivos

```python
archivo = st.file_uploader("Suba un archivo de ventas", type=["csv", "xlsx"])
if archivo is not None:
    if archivo.name.endswith(".csv"):
        df = pd.read_csv(archivo, sep=None, engine="python")
    else:
        df = pd.read_excel(archivo)                    # Requiere openpyxl
    st.success(f"{archivo.name}: {len(df)} filas")

st.download_button("Descargar CSV", data=df.to_csv(index=False, sep=";").encode("utf-8-sig"),
                   file_name="ventas.csv", mime="text/csv", icon=":material/download:")
```

Excel en memoria:

```python
import io
buffer = io.BytesIO()
with pd.ExcelWriter(buffer, engine="openpyxl") as escritor:
    df.to_excel(escritor, sheet_name="Ventas", index=False)
st.download_button("Descargar Excel", buffer.getvalue(), "ventas.xlsx")
```

### 13.2 Secretos

Guarde credenciales en `.streamlit/secrets.toml` (no lo suba al repositorio):

```toml
# .streamlit/secrets.toml
api_key = "abc123"

[connections.ventas_db]
url = "postgresql://usuario:clave@servidor:5432/ventas"
```

```python
clave = st.secrets["api_key"]
```

### 13.3 Conexiones a bases de datos

```python
conexion = st.connection("ventas_db", type="sql")              # Requiere sqlalchemy y el driver
df = conexion.query("SELECT * FROM pedidos WHERE fecha >= :inicio", params={"inicio": "2025-01-01"}, ttl="10m")
```

---

## 14. Aplicaciones de varias páginas

Con `st.navigation` y `st.Page` (recomendado):

```
mi_app/
├── app.py
└── paginas/
    ├── resumen.py
    ├── clientes.py
    └── configuracion.py
```

```python
# app.py
import streamlit as st

st.set_page_config(page_title="Gestión comercial", layout="wide")

paginas = {
    "Análisis": [
        st.Page("paginas/resumen.py", title="Resumen", icon=":material/dashboard:", default=True),
        st.Page("paginas/clientes.py", title="Clientes", icon=":material/group:"),
    ],
    "Administración": [
        st.Page("paginas/configuracion.py", title="Configuración", icon=":material/settings:"),
    ],
}
pagina = st.navigation(paginas, position="sidebar")
pagina.run()
```

```python
# Navegar desde código o con enlaces
st.page_link("paginas/clientes.py", label="Ir a clientes", icon=":material/arrow_forward:")
if st.button("Ver detalle"):
    st.switch_page("paginas/clientes.py")
```

> Los datos comunes a varias páginas se comparten mediante `st.session_state` o funciones con caché en un módulo compartido.

---

## 15. Configuración, temas y despliegue

### 15.1 Archivo de configuración

```toml
# .streamlit/config.toml
[theme]
base = "light"
primaryColor = "#1F4E78"
backgroundColor = "#FFFFFF"
secondaryBackgroundColor = "#F4F6F9"
textColor = "#262626"
font = "sans-serif"

[server]
port = 8501
maxUploadSize = 200          # MB

[browser]
gatherUsageStats = false
```

### 15.2 CSS adicional (con moderación)

```python
st.html("""<style>
    [data-testid="stMetricValue"] { color: #1F4E78; }
</style>""")
```

> Los selectores internos pueden cambiar entre versiones; prefiera las opciones de `config.toml`.

### 15.3 Despliegue

| Opción | Cómo |
|---|---|
| Streamlit Community Cloud | Suba el código a GitHub con `requirements.txt` y conéctelo en share.streamlit.io (gratuito para aplicaciones públicas). |
| Servidor propio | `streamlit run app.py --server.port 80 --server.headless true` detrás de Nginx con HTTPS. |
| Docker | Véase abajo. |
| Snowflake / nubes | Streamlit in Snowflake, Azure App Service, Google Cloud Run, AWS… |

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8501
HEALTHCHECK CMD curl --fail http://localhost:8501/_stcore/health || exit 1
CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

Autenticación: Streamlit incluye inicio de sesión con proveedores OpenID Connect (`st.login()`, `st.user`, `st.logout()`), configurado en `secrets.toml`.

### 15.4 Pruebas automatizadas

```python
from streamlit.testing.v1 import AppTest

def test_filtro_region():
    at = AppTest.from_file("app.py").run()
    at.selectbox(key="region").select("Norte").run()
    assert not at.exception
    assert "Norte" in at.markdown[0].value
```

---

## 16. Ejemplo práctico completo

**Aplicación de análisis de ventas con carga de datos propia**: permite subir un CSV o Excel de pedidos (o usar datos de ejemplo), filtra por región, categoría y periodo desde la barra lateral, muestra indicadores con variación y minigráfico, y organiza el análisis en pestañas: evolución temporal (Plotly), *ranking* de productos con selección de filas, análisis por día y hora (Altair), tabla editable de objetivos por región con cálculo de cumplimiento y exportación a CSV y Excel. Usa caché, estado de sesión, formularios, fragmentos y un diálogo de confirmación.

```python
"""
Análisis de ventas con Streamlit.
Requiere: pip install streamlit pandas numpy plotly altair openpyxl
Ejecutar: streamlit run app_ventas.py
"""
import io
from datetime import date

import altair as alt
import numpy as np
import pandas as pd
import plotly.express as px
import streamlit as st

st.set_page_config(page_title="Análisis de ventas", page_icon="📊", layout="wide")

AZUL = "#1F4E78"
COLORES = {"Norte": "#1F4E78", "Centro": "#2E75B6", "Sur": "#C55A11", "Este": "#7F7F7F"}
COLUMNAS_OBLIGATORIAS = {"fecha", "region", "categoria", "producto", "unidades", "importe"}
DIAS = ["Lun", "Mar", "Mié", "Jue", "Vie", "Sáb", "Dom"]


# ------------------------------------------------------------------ Utilidades
def euros(valor: float, decimales: int = 0) -> str:
    return f"{valor:,.{decimales}f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def numero(valor: float) -> str:
    return f"{valor:,.0f}".replace(",", ".")


# ------------------------------------------------------------------ Datos (con caché)
@st.cache_data(show_spinner="Generando datos de ejemplo…")
def datos_ejemplo(n: int = 30_000, semilla: int = 4) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    catalogo = {"Electrónica": ["Portátil", "Móvil", "Auriculares", "Tableta"],
                "Hogar": ["Cafetera", "Lámpara", "Sartén", "Aspiradora"],
                "Moda": ["Zapatillas", "Chaqueta", "Bolso", "Camisa"]}
    precios = {"Portátil": 750, "Móvil": 420, "Auriculares": 60, "Tableta": 300, "Cafetera": 90, "Lámpara": 45,
               "Sartén": 35, "Aspiradora": 180, "Zapatillas": 80, "Chaqueta": 110, "Bolso": 65, "Camisa": 40}
    categoria = rng.choice(list(catalogo), n, p=[0.35, 0.3, 0.35])
    producto = np.array([rng.choice(catalogo[c]) for c in categoria])
    dias = rng.integers(0, 731, n)
    fecha = pd.Timestamp("2024-01-01") + pd.to_timedelta(dias, unit="D") \
        + pd.to_timedelta(np.clip(rng.normal(17, 3, n), 9, 21).round(), unit="h")
    unidades = rng.choice([1, 1, 1, 2, 2, 3], n)
    precio = np.array([precios[p] for p in producto]) * rng.normal(1, 0.08, n) * (1 + dias / 731 * 0.06)
    return pd.DataFrame({"fecha": fecha, "region": rng.choice(list(COLORES), n, p=[0.25, 0.35, 0.22, 0.18]),
                         "categoria": categoria, "producto": producto, "unidades": unidades,
                         "importe": (precio * unidades).round(2)})


@st.cache_data(show_spinner="Leyendo archivo…")
def leer_archivo(contenido: bytes, nombre: str) -> pd.DataFrame:
    buffer = io.BytesIO(contenido)
    df = pd.read_excel(buffer) if nombre.lower().endswith((".xlsx", ".xls")) \
        else pd.read_csv(buffer, sep=None, engine="python", decimal=",")
    df.columns = [c.strip().lower() for c in df.columns]
    faltan = COLUMNAS_OBLIGATORIAS - set(df.columns)
    if faltan:
        raise ValueError(f"Faltan columnas obligatorias: {', '.join(sorted(faltan))}")
    df["fecha"] = pd.to_datetime(df["fecha"], dayfirst=True, errors="coerce")
    return df.dropna(subset=["fecha", "importe"])


def filtrar(df: pd.DataFrame, regiones, categorias, inicio: date, fin: date) -> pd.DataFrame:
    mascara = (df["fecha"].dt.date >= inicio) & (df["fecha"].dt.date <= fin)
    if regiones:
        mascara &= df["region"].isin(regiones)
    if categorias:
        mascara &= df["categoria"].isin(categorias)
    return df[mascara]


# ------------------------------------------------------------------ Estado inicial
if "objetivos" not in st.session_state:
    st.session_state.objetivos = pd.DataFrame({
        "region": list(COLORES), "objetivo_anual": [2_600_000, 3_600_000, 2_300_000, 1_900_000],
        "responsable": ["Ana Ruiz", "Luis Gómez", "Marta Díaz", "Jordi Puig"]})


def restablecer_filtros(df: pd.DataFrame) -> None:
    st.session_state.f_regiones = []
    st.session_state.f_categorias = []
    st.session_state.f_periodo = (date(df["fecha"].max().year, 1, 1), df["fecha"].max().date())


# ------------------------------------------------------------------ Barra lateral
with st.sidebar:
    st.title("📊 Análisis de ventas")
    origen = st.radio("Origen de los datos", ["Datos de ejemplo", "Subir archivo"], horizontal=True)
    if origen == "Subir archivo":
        archivo = st.file_uploader("CSV o Excel", type=["csv", "xlsx"],
                                   help=f"Columnas: {', '.join(sorted(COLUMNAS_OBLIGATORIAS))}")
        if archivo is None:
            st.info("Suba un archivo para continuar.")
            st.stop()
        try:
            df = leer_archivo(archivo.getvalue(), archivo.name)
        except ValueError as error:
            st.error(str(error))
            st.stop()
    else:
        df = datos_ejemplo()

    if "f_periodo" not in st.session_state:
        restablecer_filtros(df)

    st.header("Filtros", divider="blue")
    regiones = st.multiselect("Regiones", sorted(df["region"].unique()), key="f_regiones", placeholder="Todas")
    categorias = st.multiselect("Categorías", sorted(df["categoria"].unique()), key="f_categorias",
                                placeholder="Todas")
    periodo = st.date_input("Periodo", key="f_periodo", format="DD/MM/YYYY",
                            min_value=df["fecha"].min().date(), max_value=df["fecha"].max().date())
    st.button("Restablecer filtros", on_click=restablecer_filtros, args=(df,), icon=":material/restart_alt:")
    st.caption(f"{numero(len(df))} pedidos cargados · del {df['fecha'].min():%d/%m/%Y} al {df['fecha'].max():%d/%m/%Y}")

if not isinstance(periodo, tuple) or len(periodo) != 2:
    st.warning("Seleccione la fecha de inicio y de fin del periodo.")
    st.stop()

inicio, fin = periodo
datos = filtrar(df, regiones, categorias, inicio, fin)
duracion = fin - inicio
anterior = filtrar(df, regiones, categorias, inicio - duracion - pd.Timedelta(days=1).to_pytimedelta(),
                   inicio - pd.Timedelta(days=1).to_pytimedelta())

if datos.empty:
    st.warning("No hay datos para los filtros seleccionados.")
    st.stop()

# ------------------------------------------------------------------ Cabecera e indicadores
st.title("Ventas")
st.caption(f"Del {inicio:%d/%m/%Y} al {fin:%d/%m/%Y} · comparado con los {duracion.days + 1} días anteriores")


def variacion(actual: float, previo: float) -> str | None:
    return None if not previo else f"{(actual / previo - 1) * 100:+.1f} %".replace(".", ",")


semanal = datos.set_index("fecha")["importe"].resample("W").sum()
col1, col2, col3, col4 = st.columns(4)
col1.metric("Ventas", euros(datos["importe"].sum()), variacion(datos["importe"].sum(), anterior["importe"].sum()),
            border=True, chart_data=semanal.tolist(), chart_type="area")
col2.metric("Pedidos", numero(len(datos)), variacion(len(datos), len(anterior)), border=True,
            chart_data=datos.set_index("fecha").resample("W").size().tolist(), chart_type="bar")
ticket, ticket_ant = datos["importe"].mean(), anterior["importe"].mean() if len(anterior) else 0
col3.metric("Ticket medio", euros(ticket, 2), variacion(ticket, ticket_ant), border=True)
col4.metric("Unidades", numero(datos["unidades"].sum()), variacion(datos["unidades"].sum(), anterior["unidades"].sum()),
            border=True)

# ------------------------------------------------------------------ Pestañas
tab_evolucion, tab_productos, tab_patrones, tab_objetivos = st.tabs(
    ["📈 Evolución", "🏷️ Productos", "🕒 Patrones", "🎯 Objetivos"])

with tab_evolucion:
    c1, c2 = st.columns([3, 1], vertical_alignment="center")
    with c2:
        agregacion = st.segmented_control("Agregación", ["Día", "Semana", "Mes"], default="Semana")
        apilado = st.toggle("Áreas apiladas", value=True)
    frecuencia = {"Día": "D", "Semana": "W", "Mes": "MS"}[agregacion or "Semana"]
    serie = datos.set_index("fecha").groupby("region")["importe"].resample(frecuencia).sum().reset_index()
    figura = (px.area if apilado else px.line)(serie, x="fecha", y="importe", color="region",
                                               color_discrete_map=COLORES,
                                               labels={"importe": "Ventas (€)", "fecha": "", "region": "Región"})
    figura.update_layout(hovermode="x unified", separators=",.", height=420, legend_orientation="h",
                         legend_y=1.1, margin={"t": 30})
    figura.update_traces(hovertemplate="%{y:,.0f} €")
    with c1:
        st.plotly_chart(figura, config={"displaylogo": False})

with tab_productos:
    ranking = (datos.groupby(["producto", "categoria"], as_index=False)
               .agg(importe=("importe", "sum"), unidades=("unidades", "sum"), pedidos=("importe", "size"))
               .sort_values("importe", ascending=False, ignore_index=True))
    ranking["cuota"] = ranking["importe"] / ranking["importe"].sum()
    evolucion = (datos.assign(mes=datos["fecha"].dt.to_period("M"))
                 .groupby(["producto", "mes"])["importe"].sum().unstack(fill_value=0))
    ranking["tendencia"] = ranking["producto"].map(lambda p: evolucion.loc[p].round(0).tolist())

    st.caption("Seleccione uno o varios productos para ver su detalle.")
    evento = st.dataframe(
        ranking, hide_index=True, on_select="rerun", selection_mode="multi-row", key="tabla_productos",
        column_config={
            "producto": "Producto", "categoria": "Categoría",
            "importe": st.column_config.NumberColumn("Ventas", format="%.0f €"),
            "unidades": st.column_config.NumberColumn("Unidades", format="localized"),
            "pedidos": st.column_config.NumberColumn("Pedidos", format="localized"),
            "cuota": st.column_config.ProgressColumn("Cuota", format="percent", min_value=0,
                                                     max_value=float(ranking["cuota"].max())),
            "tendencia": st.column_config.AreaChartColumn("Evolución mensual", y_min=0),
        })
    seleccion = ranking.iloc[evento.selection.rows]["producto"].tolist()
    if seleccion:
        detalle = datos[datos["producto"].isin(seleccion)]
        por_region = detalle.pivot_table(index="producto", columns="region", values="importe", aggfunc="sum",
                                         fill_value=0)
        st.bar_chart(por_region, horizontal=True, color=[COLORES[r] for r in por_region.columns],
                     y_label="", x_label="Ventas (€)")

with tab_patrones:
    @st.fragment
    def patrones(datos: pd.DataFrame) -> None:
        metrica = st.radio("Métrica", ["Ventas", "Pedidos"], horizontal=True, key="metrica_patrones")
        agregado = (datos.assign(dia=datos["fecha"].dt.dayofweek.map(dict(enumerate(DIAS))),
                                 hora=datos["fecha"].dt.hour)
                    .groupby(["dia", "hora"], as_index=False)
                    .agg(valor=("importe", "sum" if metrica == "Ventas" else "size")))
        calor = alt.Chart(agregado).mark_rect(cornerRadius=2).encode(
            x=alt.X("hora:O", title="Hora"), y=alt.Y("dia:N", sort=DIAS, title=None),
            color=alt.Color("valor:Q", scale=alt.Scale(scheme="blues"), title=metrica),
            tooltip=[alt.Tooltip("dia:N", title="Día"), alt.Tooltip("hora:O", title="Hora"),
                     alt.Tooltip("valor:Q", title=metrica, format=",.0f")],
        ).properties(height=280)
        st.altair_chart(calor)

    patrones(datos)

with tab_objetivos:
    st.write("Edite los objetivos anuales; el cumplimiento se calcula con las ventas del año en curso.")
    with st.form("form_objetivos"):
        objetivos = st.data_editor(
            st.session_state.objetivos, hide_index=True, disabled=["region"], key="editor_objetivos",
            column_config={"region": "Región",
                           "objetivo_anual": st.column_config.NumberColumn("Objetivo anual", min_value=0,
                                                                           step=50_000, format="%.0f €"),
                           "responsable": "Responsable"})
        if st.form_submit_button("Guardar objetivos", type="primary"):
            st.session_state.objetivos = objetivos
            st.toast("Objetivos actualizados", icon="✅")

    anio = fin.year
    ventas_anio = (df[(df["fecha"].dt.year == anio) & (df["fecha"].dt.date <= fin)]
                   .groupby("region")["importe"].sum())
    fraccion_anio = (fin - date(anio, 1, 1)).days / (date(anio, 12, 31) - date(anio, 1, 1)).days
    cumplimiento = st.session_state.objetivos.assign(
        ventas=lambda d: d["region"].map(ventas_anio).fillna(0),
        cumplimiento=lambda d: d["ventas"] / d["objetivo_anual"],
        previsto=fraccion_anio)
    st.dataframe(cumplimiento, hide_index=True, column_config={
        "region": "Región", "responsable": "Responsable",
        "objetivo_anual": st.column_config.NumberColumn("Objetivo", format="%.0f €"),
        "ventas": st.column_config.NumberColumn(f"Ventas {anio}", format="%.0f €"),
        "cumplimiento": st.column_config.ProgressColumn("Cumplimiento", format="percent", min_value=0, max_value=1),
        "previsto": st.column_config.ProgressColumn("Previsto a la fecha", format="percent", min_value=0, max_value=1),
    })

# ------------------------------------------------------------------ Exportación
st.divider()


@st.dialog("Exportar datos filtrados")
def exportar(datos: pd.DataFrame) -> None:
    st.write(f"Se exportarán **{numero(len(datos))}** pedidos con los filtros actuales.")
    formato = st.radio("Formato", ["CSV (separador ;)", "Excel"], horizontal=True)
    if formato.startswith("CSV"):
        contenido = datos.to_csv(index=False, sep=";", decimal=",").encode("utf-8-sig")
        st.download_button("Descargar CSV", contenido, "ventas_filtradas.csv", "text/csv", type="primary")
    else:
        buffer = io.BytesIO()
        with pd.ExcelWriter(buffer, engine="openpyxl") as escritor:
            datos.to_excel(escritor, sheet_name="Pedidos", index=False)
            st.session_state.objetivos.to_excel(escritor, sheet_name="Objetivos", index=False)
        st.download_button("Descargar Excel", buffer.getvalue(), "ventas_filtradas.xlsx", type="primary")


izquierda, derecha = st.columns([4, 1], vertical_alignment="center")
izquierda.caption("Datos simulados · Aplicación creada con Streamlit")
if derecha.button("Exportar datos", icon=":material/download:"):
    exportar(datos)
```

Ejecute `streamlit run app_ventas.py`, cambie los filtros de la barra lateral, seleccione productos en la pestaña «Productos», edite los objetivos y exporte los datos filtrados.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Warning: to view this Streamlit app on a browser, run it with streamlit run` | Se ejecutó con `python app.py`. | `streamlit run app.py`. |
| `streamlit: command not found` | El ejecutable no está en el PATH. | `python -m streamlit run app.py`. |
| `DuplicateWidgetID` / `StreamlitDuplicateElementId` | Dos *widgets* idénticos sin `key`. | Añada `key` distintos. |
| `StreamlitAPIException: st.session_state.x cannot be modified after the widget with key x is instantiated` | Se cambió el estado de un *widget* después de dibujarlo. | Hágalo en un *callback* (`on_click`) o antes de crear el *widget*. |
| `set_page_config() can only be called once per app page, and must be called as the first Streamlit command` | No es la primera llamada `st.*`. | Muévala al principio del script. |
| La aplicación es lenta en cada clic | Se recargan datos o se recalcula todo. | `@st.cache_data`, `@st.cache_resource`, `st.form`, `@st.fragment`. |
| `UnhashableParamError` en una función con caché | Argumento no *hashable* (conexión, modelo). | Prefije el argumento con `_` (`_conexion`). |
| Los cambios en el archivo de datos no se reflejan | El resultado está en caché. | `ttl=`, `funcion.clear()` o menú *Clear cache*. |
| Un botón «se olvida» tras la siguiente interacción | `st.button` solo es `True` en una reejecución. | Guarde el estado en `st.session_state`. |
| `st.date_input` devuelve una sola fecha a medias | El usuario solo eligió el inicio del rango. | Compruebe que la tupla tiene dos fechas antes de continuar. |
| Aviso de `use_container_width` obsoleto | Parámetro antiguo. | Use `width="stretch"` (o nada: es el valor por defecto). |
| El archivo subido es demasiado grande | Límite de 200 MB. | `server.maxUploadSize` en `config.toml`. |
| Datos de un usuario visibles para otro | Se usaron variables globales o `cache_resource` mutable. | `st.session_state` para datos de usuario; no mute objetos cacheados. |

---

## 18. Buenas prácticas

- Llame a `st.set_page_config()` al principio y use `layout="wide"` para dashboards.
- Coloque los filtros en la **barra lateral** y el análisis en el cuerpo, organizado con columnas y pestañas.
- Cachee la carga de datos (`@st.cache_data`) y los recursos compartidos (`@st.cache_resource`).
- Inicialice `st.session_state` al principio del script y modifíquelo en *callbacks*.
- Use `st.form` cuando varios campos deban aplicarse juntos y `@st.fragment` para zonas independientes.
- Valide los datos de entrada (columnas, tipos, fechas) y detenga la ejecución con un mensaje claro (`st.stop()`).
- Informe del contexto: periodo, filtros activos, número de registros y fuente de los datos.
- Configure el formato español en números y fechas, y use `column_config` en las tablas.
- Guarde credenciales en `secrets.toml` o variables de entorno, nunca en el código.
- Separe la lógica de datos en módulos propios: facilita las pruebas (`AppTest`) y la reutilización.

---

## 19. Referencia rápida

```python
import streamlit as st
st.set_page_config(page_title=, page_icon=, layout="wide")
st.title / header / subheader / markdown / caption / code / latex / divider / write / json / image
st.success / info / warning / error / exception / toast / spinner / progress / status / balloons
st.dataframe(df, hide_index=True, column_config={...}, on_select="rerun") / table / data_editor / metric
st.line_chart / bar_chart / area_chart / scatter_chart / map / plotly_chart / altair_chart / pyplot / pydeck_chart

st.text_input / text_area / number_input / selectbox / multiselect / radio / segmented_control / pills
st.slider / select_slider / checkbox / toggle / date_input / time_input / color_picker / button / feedback
st.file_uploader / download_button / camera_input           (key=, help=, on_change=, disabled=)

st.sidebar / st.columns([2, 1]) / st.tabs([...]) / st.expander / st.popover / st.container(border=True) / st.empty
st.session_state.x / st.query_params / st.rerun() / st.stop()
with st.form("f"): ... st.form_submit_button()
@st.fragment(run_every="10s") / @st.dialog("Título")
@st.cache_data(ttl="1h") / @st.cache_resource / st.secrets["clave"] / st.connection("bd", type="sql")
st.navigation([st.Page("p.py", title=)]).run() / st.page_link / st.switch_page
streamlit run app.py
```

---

## 20. Ejercicios propuestos

1. Cree una calculadora de préstamos con deslizadores (importe, plazo, interés) que muestre la cuota mensual y un gráfico de amortización.
2. Desarrolle un explorador de CSV: subir archivo, elegir columnas en desplegables y generar el gráfico seleccionado (líneas, barras o dispersión).
3. Construya una lista de tareas persistente en `st.session_state` con alta mediante formulario, marcado de completadas y borrado con diálogo de confirmación.
4. Diseñe una aplicación de varias páginas (resumen, detalle, configuración) con `st.navigation` que comparta los filtros entre páginas.
5. Amplíe el ejemplo completo con una pestaña de previsión que ajuste una tendencia lineal por región y escriba pruebas con `AppTest` para los filtros.

---

## 21. Referencias

- Documentación oficial: https://docs.streamlit.io/
- Referencia de la API: https://docs.streamlit.io/develop/api-reference
- Conceptos (ejecución, caché, estado): https://docs.streamlit.io/develop/concepts
- Galería de aplicaciones: https://streamlit.io/gallery
- Community Cloud: https://docs.streamlit.io/deploy/streamlit-community-cloud
- Componentes de la comunidad: https://streamlit.io/components
