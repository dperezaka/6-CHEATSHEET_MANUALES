# Manual de uso: Streamlit

> **Categoría:** Aplicaciones web de datos · **Utilidad:** Paneles e interfaces web interactivas escritas solo en Python
> **Tipo:** Librería de terceros (`pip install streamlit`) · **Versión de referencia:** 1.50 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y ejecución](#2-instalación-y-ejecución)
3. [Modelo de ejecución](#3-modelo-de-ejecución)
4. [Texto y elementos de salida](#4-texto-y-elementos-de-salida)
5. [Datos: tablas, métricas y editor](#5-datos-tablas-métricas-y-editor)
6. [Gráficos](#6-gráficos)
7. [Widgets de entrada](#7-widgets-de-entrada)
8. [Diseño: barra lateral, columnas, pestañas y contenedores](#8-diseño-barra-lateral-columnas-pestañas-y-contenedores)
9. [Estado de sesión (`st.session_state`)](#9-estado-de-sesión-stsession_state)
10. [Formularios](#10-formularios)
11. [Caché](#11-caché)
12. [Archivos: subida y descarga](#12-archivos-subida-y-descarga)
13. [Aplicaciones de varias páginas](#13-aplicaciones-de-varias-páginas)
14. [Fragmentos y diálogos](#14-fragmentos-y-diálogos)
15. [Mensajes de estado y progreso](#15-mensajes-de-estado-y-progreso)
16. [Parámetros de URL](#16-parámetros-de-url)
17. [Conexiones a bases de datos y secretos](#17-conexiones-a-bases-de-datos-y-secretos)
18. [Interfaces de chat](#18-interfaces-de-chat)
19. [Configuración y temas](#19-configuración-y-temas)
20. [Pruebas automatizadas](#20-pruebas-automatizadas)
21. [Despliegue](#21-despliegue)
22. [Ejemplo práctico completo](#22-ejemplo-práctico-completo)
23. [Errores frecuentes](#23-errores-frecuentes)
24. [Buenas prácticas](#24-buenas-prácticas)
25. [Referencia rápida](#25-referencia-rápida)
26. [Ejercicios propuestos](#26-ejercicios-propuestos)
27. [Referencias](#27-referencias)

---

## 1. Descripción

**Streamlit** convierte scripts de Python en **aplicaciones web interactivas** sin escribir HTML, CSS ni JavaScript. Está pensado para analistas, científicos de datos e ingenieros que necesitan compartir paneles, herramientas internas, prototipos de modelos o interfaces de chat de forma rápida.

**Ventajas:**
- Desarrollo extremadamente rápido: un script de pocas líneas ya es una aplicación web.
- Integración directa con pandas, Plotly, Matplotlib, Altair, scikit-learn, etc.
- Recarga automática al guardar el código.
- Widgets, caché, estado, varias páginas, autenticación y despliegue gratuito (Community Cloud).

**Limitaciones:**
- El script se **vuelve a ejecutar completo** con cada interacción: exige usar caché y estado con criterio.
- Control limitado del diseño visual frente a un *framework* web.
- No adecuado para sitios públicos de alto tráfico ni aplicaciones con lógica de interfaz muy compleja.
- Cada usuario mantiene una sesión en el servidor (consumo de memoria).

| Si necesita… | Opción |
|---|---|
| Panel o herramienta interna de datos en horas | **Streamlit** |
| Paneles con control fino de callbacks y diseño | Dash (Plotly) |
| Demos de modelos de IA | Gradio |
| Aplicación web completa (usuarios, API, BD) | Django, FastAPI + frontend |
| Aplicación de escritorio | PySide6, CustomTkinter |

---

## 2. Instalación y ejecución

```bash
pip install streamlit
streamlit hello                         # Aplicación de demostración
streamlit run app.py                    # Abre http://localhost:8501
streamlit run app.py --server.port 8080 --server.headless true
python -m streamlit run app.py          # Alternativa si el comando no está en el PATH
```

```python
# app.py
import streamlit as st

st.title("Mi primera aplicación")
nombre = st.text_input("¿Cómo se llama?")
if nombre:
    st.success(f"Hola, {nombre}")
```

> No ejecute el archivo con `python app.py`: debe lanzarse con `streamlit run`.

---

## 3. Modelo de ejecución

1. Al abrir la página, Streamlit ejecuta el script **de arriba abajo**.
2. Cada interacción con un widget **vuelve a ejecutar todo el script**, y el widget devuelve su nuevo valor.
3. Las variables normales se pierden entre ejecuciones; para conservar datos se usa `st.session_state`.
4. Los cálculos costosos se protegen con `@st.cache_data` / `@st.cache_resource`.

```python
import streamlit as st

contador = 0                       # Se reinicia en cada ejecución
if st.button("Sumar"):
    contador += 1                  # Siempre mostrará 1
st.write(contador)

if "clics" not in st.session_state:
    st.session_state.clics = 0     # Persistente durante la sesión del usuario
if st.button("Sumar (persistente)"):
    st.session_state.clics += 1
st.write(st.session_state.clics)
```

`st.stop()` detiene la ejecución en ese punto; `st.rerun()` fuerza una nueva ejecución inmediata.

---

## 4. Texto y elementos de salida

```python
st.title("Título") ; st.header("Encabezado", divider="blue") ; st.subheader("Subencabezado")
st.markdown("**Negrita**, *cursiva*, `código`, [enlace](https://streamlit.io) y :blue[texto azul]")
st.caption("Texto secundario") ; st.text("Texto de ancho fijo")
st.code("SELECT * FROM ventas", language="sql")
st.latex(r"\bar{x} = \frac{1}{n}\sum x_i")
st.divider()
st.write("Acepta casi todo:", 42, {"clave": "valor"}, df)       # Muestra cada objeto de la forma adecuada
st.json({"pedido": 1042, "estado": "enviado"})
st.image("logo.png", width=160, caption="Logotipo")
st.badge("Nuevo", color="green")                                 # Versiones recientes

"Texto suelto"                     # «Magia»: una expresión aislada se muestra como st.write
```

---

## 5. Datos: tablas, métricas y editor

```python
st.dataframe(df, hide_index=True, height=400, column_config={
    "importe": st.column_config.NumberColumn("Importe", format="%.2f €"),
    "fecha": st.column_config.DateColumn("Fecha", format="DD/MM/YYYY"),
    "progreso": st.column_config.ProgressColumn("Avance", min_value=0, max_value=1),
    "web": st.column_config.LinkColumn("Sitio"),
    "tendencia": st.column_config.LineChartColumn("Últimos 12 meses"),
})
st.table(df.head())                                     # Tabla estática

st.metric("Ventas", "1,2 M€", delta="8,4 %")             # Indicador con variación
st.metric("Devoluciones", "3,1 %", delta="0,4 %", delta_color="inverse")   # Subir es malo

editado = st.data_editor(df, num_rows="dynamic", disabled=["id"], column_config={
    "estado": st.column_config.SelectboxColumn("Estado", options=["Abierto", "Cerrado"], required=True),
    "importe": st.column_config.NumberColumn(min_value=0, step=0.01),
})                                                       # Devuelve el DataFrame modificado por el usuario

seleccion = st.dataframe(df, on_select="rerun", selection_mode="multi-row")
filas = seleccion.selection.rows                         # Índices posicionales de las filas seleccionadas
```

---

## 6. Gráficos

```python
st.line_chart(df, x="fecha", y=["ventas", "objetivo"])        # Gráficos nativos sencillos
st.bar_chart(df, x="region", y="ventas", color="canal", stack=True)
st.area_chart(df) ; st.scatter_chart(df, x="precio", y="unidades", size="margen")
st.map(df_coordenadas)                                        # Columnas lat/lon

import plotly.express as px
fig = px.bar(df, x="region", y="ventas", color="canal")
st.plotly_chart(fig)                                          # Interactivo; ocupa el ancho del contenedor
evento = st.plotly_chart(fig, on_select="rerun")              # Selección de puntos en el gráfico

import matplotlib.pyplot as plt
fig, ax = plt.subplots(); ax.plot([1, 2, 3]); st.pyplot(fig)
st.altair_chart(grafico_altair) ; st.vega_lite_chart(df, especificacion)
```

---

## 7. Widgets de entrada

```python
texto = st.text_input("Cliente", placeholder="Nombre o NIF", max_chars=60)
notas = st.text_area("Observaciones", height=120)
cantidad = st.number_input("Unidades", min_value=1, max_value=500, value=10, step=1)
precio = st.slider("Precio máximo", 0.0, 1000.0, 250.0, step=10.0)
rango = st.slider("Rango", 0, 100, (20, 80))                        # Tupla → doble selector
fecha = st.date_input("Desde", value=date(2026, 1, 1), format="DD/MM/YYYY")
periodo = st.date_input("Periodo", value=(date(2026, 1, 1), date(2026, 3, 31)))
hora = st.time_input("Hora")
region = st.selectbox("Región", ["Norte", "Centro", "Sur"], index=None, placeholder="Elija una región")
regiones = st.multiselect("Regiones", ["Norte", "Centro", "Sur"], default=["Norte"])
canal = st.radio("Canal", ["Tienda", "Web"], horizontal=True)
vista = st.segmented_control("Vista", ["Mensual", "Trimestral", "Anual"], default="Mensual")
etiquetas = st.pills("Etiquetas", ["Urgente", "VIP", "Nuevo"], selection_mode="multi")
activo = st.checkbox("Solo activos", value=True) ; modo = st.toggle("Modo detallado")
color = st.color_picker("Color", "#1F4E78")
talla = st.select_slider("Talla", options=["S", "M", "L", "XL"])
valoracion = st.feedback("stars")

if st.button("Calcular", type="primary", icon=":material/calculate:"):
    ...
st.link_button("Documentación", "https://docs.streamlit.io")
```

Parámetros comunes de los widgets:

| Parámetro | Uso |
|---|---|
| `key` | Identificador único; permite leer/escribir el valor en `st.session_state[key]`. Obligatorio si hay dos widgets iguales. |
| `on_change` / `on_click`, `args`, `kwargs` | Función de retrollamada que se ejecuta **antes** de la nueva ejecución del script. |
| `help` | Texto de ayuda (icono «?»). |
| `disabled` | Desactivar. |
| `label_visibility` | `"visible"`, `"hidden"`, `"collapsed"`. |

---

## 8. Diseño: barra lateral, columnas, pestañas y contenedores

```python
st.set_page_config(page_title="Ventas", page_icon="📊", layout="wide")   # Primera llamada a Streamlit

with st.sidebar:
    st.header("Filtros")
    anio = st.selectbox("Año", [2026, 2025])

c1, c2, c3 = st.columns(3)                       # Columnas iguales
c1.metric("Ventas", "1,2 M€")
izq, der = st.columns([2, 1], gap="large", vertical_alignment="bottom")   # Proporciones

resumen, detalle = st.tabs(["Resumen", "Detalle"])
with resumen:
    st.write("…")

with st.expander("Ver supuestos del cálculo"):
    st.write("…")

with st.container(border=True, height=300):      # Caja con borde y desplazamiento
    st.write("…")

with st.popover("Opciones"):
    st.checkbox("Mostrar totales")

hueco = st.empty()                               # Marcador reemplazable
hueco.info("Calculando…") ; hueco.success("Listo")
```

---

## 9. Estado de sesión (`st.session_state`)

Diccionario persistente **por usuario y pestaña del navegador** durante la sesión:

```python
st.session_state.setdefault("carrito", [])

def anadir(producto: str) -> None:               # Retrollamada: se ejecuta antes de redibujar
    st.session_state.carrito.append(producto)

st.button("Añadir silla", on_click=anadir, args=("Silla",))
st.write(st.session_state.carrito)

st.text_input("Cliente", key="cliente")          # Valor accesible como st.session_state.cliente
if st.button("Limpiar"):
    st.session_state.cliente = ""                # ⚠ Error: no se puede modificar tras crear el widget…
def limpiar(): st.session_state.cliente = ""     # …use una retrollamada
st.button("Limpiar", on_click=limpiar)

del st.session_state["carrito"]                  # Eliminar
```

> Para conservar un valor de widget al cambiar de página, cópielo a una clave propia de `session_state` (los widgets que desaparecen de la pantalla se borran del estado).

---

## 10. Formularios

Agrupan widgets para que el script **solo se ejecute al enviar** (evita recálculos con cada tecla):

```python
with st.form("alta_cliente", clear_on_submit=True, border=True):
    nombre = st.text_input("Nombre")
    email = st.text_input("Correo")
    limite = st.number_input("Límite de crédito", 0, 100_000, 5_000, step=500)
    enviado = st.form_submit_button("Guardar", type="primary")

if enviado:
    if "@" not in email:
        st.error("Correo no válido")
    else:
        guardar_cliente(nombre, email, limite)
        st.success("Cliente guardado")
```

Dentro de un formulario solo `st.form_submit_button` admite retrollamadas.

---

## 11. Caché

```python
@st.cache_data(ttl="1h", show_spinner="Cargando ventas…")        # Datos serializables (DataFrames, listas, dicts)
def cargar_ventas(anio: int) -> pd.DataFrame:
    return pd.read_sql(f"SELECT * FROM ventas WHERE anio = {anio}", motor)

@st.cache_resource                                                # Recursos compartidos: conexiones, modelos
def obtener_modelo():
    return joblib.load("modelo.joblib")

cargar_ventas.clear()          # Vaciar una función ; st.cache_data.clear() para todas
```

| Decorador | Devuelve | Compartido entre usuarios | Uso |
|---|---|---|---|
| `cache_data` | Una **copia** del resultado | Sí | Consultas, lecturas de archivos, transformaciones. |
| `cache_resource` | El **mismo objeto** | Sí | Conexiones a BD, modelos de ML, clientes de API. |

- La clave de la caché son el código de la función y sus argumentos; los argumentos que empiezan por `_` se excluyen (`def f(_conexion, anio)`).
- No modifique el objeto devuelto por `cache_resource` si no es seguro entre hilos.

---

## 12. Archivos: subida y descarga

```python
archivo = st.file_uploader("Suba el CSV de ventas", type=["csv", "xlsx"])
if archivo is not None:
    df = pd.read_csv(archivo, sep=";") if archivo.name.endswith(".csv") else pd.read_excel(archivo)

varios = st.file_uploader("Facturas", type="pdf", accept_multiple_files=True)
foto = st.camera_input("Foto del albarán")

st.download_button("Descargar CSV", df.to_csv(index=False, sep=";").encode("utf-8-sig"),
                   file_name="ventas.csv", mime="text/csv")

buffer = io.BytesIO()
df.to_excel(buffer, index=False)
st.download_button("Descargar Excel", buffer.getvalue(), file_name="ventas.xlsx",
                   mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")
```

El tamaño máximo de subida por defecto es 200 MB (`server.maxUploadSize` en la configuración).

---

## 13. Aplicaciones de varias páginas

Con `st.navigation` (recomendado) se definen las páginas desde un script de entrada:

```python
# app.py
import streamlit as st

def inicio():
    st.title("Inicio")

paginas = st.navigation({
    "General": [st.Page(inicio, title="Inicio", icon=":material/home:", default=True)],
    "Análisis": [st.Page("paginas/ventas.py", title="Ventas", icon=":material/bar_chart:"),
                 st.Page("paginas/clientes.py", title="Clientes", icon=":material/group:")],
})
st.set_page_config(page_title="Portal comercial", layout="wide")
paginas.run()

# Navegar por código y enlaces
st.switch_page("paginas/ventas.py")
st.page_link("paginas/clientes.py", label="Ir a clientes", icon=":material/arrow_forward:")
```

Lo que se escriba en `app.py` antes de `paginas.run()` (barra lateral, autenticación, datos comunes) se comparte en todas las páginas. Alternativa clásica: carpeta `pages/` junto al script principal.

---

## 14. Fragmentos y diálogos

**Fragmentos**: partes de la página que se vuelven a ejecutar de forma independiente (más rápido que ejecutar todo el script), opcionalmente cada cierto tiempo:

```python
@st.fragment(run_every="10s")                    # Se actualiza solo cada 10 s
def panel_en_vivo():
    st.metric("Pedidos en curso", contar_pedidos_abiertos())

@st.fragment
def filtro_local():
    n = st.slider("Filas", 5, 50, 10)            # Solo se re-ejecuta este fragmento
    st.dataframe(df.head(n))

panel_en_vivo()
filtro_local()
```

**Diálogos modales**:

```python
@st.dialog("Detalle del pedido", width="large")
def ver_pedido(id_pedido: int):
    st.write(obtener_pedido(id_pedido))
    if st.button("Cerrar"):
        st.rerun()

if st.button("Ver pedido 1042"):
    ver_pedido(1042)
```

---

## 15. Mensajes de estado y progreso

```python
st.success("Guardado") ; st.info("Información") ; st.warning("Atención") ; st.error("Error")
st.exception(excepcion)
st.toast("Exportación completada", icon="✅")                     # Notificación temporal

with st.spinner("Calculando…"):
    resultado = calculo_largo()

barra = st.progress(0, text="Procesando")
for i, item in enumerate(items, start=1):
    procesar(item)
    barra.progress(i / len(items), text=f"Procesando {i}/{len(items)}")

with st.status("Importando datos…", expanded=True) as estado:
    st.write("Descargando…"); descargar()
    st.write("Validando…"); validar()
    estado.update(label="Importación completada", state="complete", expanded=False)
```

---

## 16. Parámetros de URL

Permiten compartir enlaces con los filtros aplicados (`http://.../?region=Norte&anio=2026`):

```python
region = st.query_params.get("region", "Todas")
st.query_params["anio"] = "2026"                 # Actualiza la URL
st.query_params.update({"region": "Sur"})
st.query_params.clear()
```

Los valores son siempre texto; convierta y valide al leerlos.

---

## 17. Conexiones a bases de datos y secretos

Secretos en `.streamlit/secrets.toml` (no lo suba al repositorio):

```toml
api_clave = "abc123"

[connections.ventas]
url = "postgresql+psycopg://usuario:clave@servidor:5432/ventas"
```

```python
clave = st.secrets["api_clave"]
conexion = st.connection("ventas", type="sql")              # SQLAlchemy por debajo
df = conexion.query("SELECT * FROM pedidos WHERE region = :r", params={"r": "Norte"}, ttl=600)
with conexion.session as s:
    s.execute(text("UPDATE ..."), {...}); s.commit()
```

Autenticación (OpenID Connect: Google, Microsoft…) configurada en `secrets.toml` → `[auth]`:

```python
if not st.user.is_logged_in:
    st.button("Iniciar sesión", on_click=st.login)
    st.stop()
st.write(f"Bienvenido, {st.user.name}")
st.button("Cerrar sesión", on_click=st.logout)
```

---

## 18. Interfaces de chat

```python
st.session_state.setdefault("mensajes", [])
for m in st.session_state.mensajes:
    with st.chat_message(m["rol"]):
        st.markdown(m["texto"])

if pregunta := st.chat_input("Escriba su pregunta"):
    st.session_state.mensajes.append({"rol": "user", "texto": pregunta})
    with st.chat_message("user"):
        st.markdown(pregunta)
    with st.chat_message("assistant"):
        respuesta = st.write_stream(generar_respuesta(pregunta))     # Muestra un generador en streaming
    st.session_state.mensajes.append({"rol": "assistant", "texto": respuesta})
```

---

## 19. Configuración y temas

`.streamlit/config.toml` en la carpeta del proyecto:

```toml
[theme]
base = "light"
primaryColor = "#1F4E78"
backgroundColor = "#FFFFFF"
secondaryBackgroundColor = "#F2F5F9"
textColor = "#1D2939"
font = "sans serif"

[server]
port = 8501
maxUploadSize = 50
headless = true

[client]
toolbarMode = "minimal"
```

CSS adicional (con moderación, puede cambiar entre versiones): `st.html("<style>...</style>")`.

---

## 20. Pruebas automatizadas

`AppTest` ejecuta la aplicación sin navegador y permite simular interacciones:

```python
from streamlit.testing.v1 import AppTest

def test_filtro_region():
    at = AppTest.from_file("app.py", default_timeout=30).run()
    assert not at.exception
    at.selectbox(key="region").select("Norte").run()
    assert at.metric[0].label == "Ventas"
    at.button(key="exportar").click().run()
    assert "Exportación" in at.success[0].value
```

---

## 21. Despliegue

| Opción | Descripción |
|---|---|
| Streamlit Community Cloud | Gratuito, desde un repositorio de GitHub (`requirements.txt` + `app.py`). |
| Snowflake | Streamlit integrado en Snowflake. |
| Docker | Cualquier nube o servidor propio. |
| Servidor interno | `streamlit run` tras un proxy inverso (nginx) con HTTPS. |

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

---

## 22. Ejemplo práctico completo

**Cuadro de mando comercial** en un solo archivo con tres páginas (`st.navigation`):

- **Panel**: filtros en la barra lateral (periodo, regiones, canal) sincronizados con la URL, indicadores con variación frente al periodo anterior, evolución mensual frente a objetivo, ventas por región y canal, *ranking* de productos con selección de filas que abre un **diálogo** de detalle, y un **fragmento** con «pedidos de hoy» que se actualiza cada 30 s.
- **Objetivos**: editor de datos (`st.data_editor`) para fijar el objetivo mensual por región, con validación y guardado en el estado de la sesión; el panel se recalcula con los nuevos objetivos.
- **Datos**: carga de un CSV propio (o datos de demostración), validación de columnas, vista previa y descarga en CSV y Excel.
- Datos cacheados con `@st.cache_data`, formularios, mensajes de estado y formato numérico español.

```python
"""
cuadro_mando.py — Cuadro de mando comercial con Streamlit.
Requiere: pip install streamlit pandas numpy plotly openpyxl
Uso: streamlit run cuadro_mando.py
"""
from __future__ import annotations

import io
from datetime import date, datetime

import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import streamlit as st

REGIONES = ["Norte", "Centro", "Este", "Sur"]
CANALES = ["Tienda", "Web", "Distribuidor"]
PRODUCTOS = {"Portátil": 899, "Monitor": 219, "Teclado": 49, "Silla": 189, "Impresora": 159, "Auriculares": 59}
COLUMNAS = {"fecha", "region", "canal", "producto", "unidades", "importe"}
AZUL, GRIS = "#1F4E78", "#A6A6A6"
COLORES = {"Tienda": "#1F4E78", "Web": "#2E75B6", "Distribuidor": "#9DC3E6"}


# ============================================================ Utilidades
def eur(valor: float, decimales: int = 0) -> str:
    return f"{valor:,.{decimales}f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def pct(valor: float) -> str:
    return f"{valor:+.1%}".replace(".", ",")


@st.cache_data(show_spinner="Generando datos de demostración…")
def datos_demo(semilla: int = 7) -> pd.DataFrame:
    rng = np.random.default_rng(semilla)
    dias = pd.date_range("2025-01-01", date.today(), freq="D")
    n = len(dias) * 12
    fechas = rng.choice(dias, n)
    estacional = 1 + 0.25 * np.sin((pd.DatetimeIndex(fechas).month - 3) / 12 * 2 * np.pi)
    producto = rng.choice(list(PRODUCTOS), n, p=[0.15, 0.2, 0.25, 0.15, 0.1, 0.15])
    unidades = np.maximum(1, (rng.poisson(3, n) * estacional).round()).astype(int)
    precio = np.array([PRODUCTOS[p] for p in producto]) * rng.uniform(0.9, 1.05, n)
    df = pd.DataFrame({"fecha": fechas, "region": rng.choice(REGIONES, n, p=[0.3, 0.35, 0.2, 0.15]),
                       "canal": rng.choice(CANALES, n, p=[0.45, 0.35, 0.2]), "producto": producto,
                       "unidades": unidades, "importe": (unidades * precio).round(2)})
    return df.sort_values("fecha", ignore_index=True)


@st.cache_data
def leer_csv(contenido: bytes) -> pd.DataFrame:
    df = pd.read_csv(io.BytesIO(contenido), sep=None, engine="python", decimal=",")
    df.columns = [c.strip().lower() for c in df.columns]
    faltan = COLUMNAS - set(df.columns)
    if faltan:
        raise ValueError(f"Faltan columnas: {', '.join(sorted(faltan))}")
    df["fecha"] = pd.to_datetime(df["fecha"], dayfirst=True, errors="coerce")
    df = df.dropna(subset=["fecha", "importe"])
    return df[list(COLUMNAS)].sort_values("fecha", ignore_index=True)


def datos() -> pd.DataFrame:
    return st.session_state.get("datos_propios", datos_demo())


def objetivos() -> pd.DataFrame:
    if "objetivos" not in st.session_state:
        st.session_state.objetivos = pd.DataFrame({"region": REGIONES, "objetivo_mensual": [85_000, 100_000, 55_000, 40_000],
                                                   "responsable": ["L. Gómez", "A. Ruiz", "M. Sanz", "J. Pérez"]})
    return st.session_state.objetivos


# ============================================================ Filtros (barra lateral + URL)
def filtros(df: pd.DataFrame) -> tuple[pd.DataFrame, pd.DataFrame, tuple[date, date]]:
    minimo, maximo = df["fecha"].min().date(), df["fecha"].max().date()
    defecto_ini = max(minimo, date(maximo.year, 1, 1))
    qp = st.query_params
    regiones_url = [r for r in qp.get("regiones", "").split(",") if r in REGIONES] or REGIONES

    with st.sidebar:
        st.header("Filtros")
        periodo = st.date_input("Periodo", (defecto_ini, maximo), min_value=minimo, max_value=maximo,
                                format="DD/MM/YYYY", key="periodo")
        regiones = st.multiselect("Regiones", REGIONES, default=regiones_url, key="regiones")
        canal = st.segmented_control("Canal", ["Todos", *CANALES], default=qp.get("canal", "Todos"), key="canal")
        st.caption("Los filtros se guardan en la URL: puede compartir el enlace.")
    if not isinstance(periodo, tuple) or len(periodo) != 2:
        st.info("Seleccione la fecha final del periodo.")
        st.stop()
    if not regiones:
        st.warning("Seleccione al menos una región.")
        st.stop()
    canal = canal or "Todos"
    st.query_params.update({"regiones": ",".join(regiones), "canal": canal})

    inicio, fin = periodo
    duracion = fin - inicio
    base = df[df["region"].isin(regiones) & ((df["canal"] == canal) if canal != "Todos" else True)]
    actual = base[(base["fecha"].dt.date >= inicio) & (base["fecha"].dt.date <= fin)]
    ini_ant = inicio - duracion - pd.Timedelta(days=1)
    anterior = base[(base["fecha"].dt.date >= ini_ant) & (base["fecha"].dt.date < inicio)]
    return actual, anterior, (inicio, fin)


# ============================================================ Página: Panel
@st.dialog("Detalle del producto", width="large")
def detalle_producto(df: pd.DataFrame, producto: str) -> None:
    d = df[df["producto"] == producto]
    c1, c2, c3 = st.columns(3)
    c1.metric("Ventas", eur(d["importe"].sum()))
    c2.metric("Unidades", f"{d['unidades'].sum():,}".replace(",", "."))
    c3.metric("Precio medio", eur(d["importe"].sum() / d["unidades"].sum(), 2))
    tabla = d.pivot_table(index="region", columns="canal", values="importe", aggfunc="sum", fill_value=0)
    st.dataframe(tabla.style.format(lambda v: eur(v)), width="stretch")
    if st.button("Cerrar", type="primary"):
        st.rerun()


@st.fragment(run_every="30s")
def pedidos_de_hoy(df: pd.DataFrame) -> None:
    """Se actualiza solo, sin volver a ejecutar toda la página."""
    hoy = df[df["fecha"].dt.date == df["fecha"].max().date()]
    rng = np.random.default_rng(int(datetime.now().timestamp()) // 30)      # Simula pedidos que llegan
    extra = int(rng.integers(0, 6))
    with st.container(border=True):
        st.markdown(f"**Actividad del último día** · actualizado {datetime.now():%H:%M:%S}")
        c1, c2 = st.columns(2)
        c1.metric("Pedidos", len(hoy) + extra, delta=extra or None)
        c2.metric("Importe", eur(hoy["importe"].sum() + extra * 250))


def pagina_panel() -> None:
    df = datos()
    actual, anterior, (inicio, fin) = filtros(df)
    st.title("Cuadro de mando comercial")
    st.caption(f"Periodo {inicio:%d/%m/%Y} – {fin:%d/%m/%Y} · comparado con el periodo anterior de igual duración")
    if actual.empty:
        st.warning("No hay ventas con los filtros seleccionados.")
        return

    ventas, ventas_ant = actual["importe"].sum(), anterior["importe"].sum()
    pedidos, pedidos_ant = len(actual), len(anterior)
    ticket, ticket_ant = ventas / pedidos, (ventas_ant / pedidos_ant if pedidos_ant else 0)
    meses = max(1, (fin.year - inicio.year) * 12 + fin.month - inicio.month + 1)
    obj = objetivos().set_index("region").loc[sorted(actual["region"].unique()), "objetivo_mensual"].sum() * meses
    variacion = lambda a, b: pct(a / b - 1) if b else None

    k = st.columns(4)
    k[0].metric("Ventas", eur(ventas), variacion(ventas, ventas_ant), border=True)
    k[1].metric("Pedidos", f"{pedidos:,}".replace(",", "."), variacion(pedidos, pedidos_ant), border=True)
    k[2].metric("Ticket medio", eur(ticket, 2), variacion(ticket, ticket_ant), border=True)
    k[3].metric("Cumplimiento del objetivo", f"{ventas / obj:.1%}".replace(".", ","),
                f"{eur(ventas - obj)} vs. objetivo", delta_color="normal", border=True,
                help="Objetivo mensual por región (página Objetivos) × número de meses del periodo")

    izquierda, derecha = st.columns([3, 2])
    with izquierda:
        mensual = actual.set_index("fecha").resample("ME")["importe"].sum().reset_index()
        fig = go.Figure()
        fig.add_bar(x=mensual["fecha"], y=mensual["importe"], name="Ventas", marker_color=AZUL)
        objetivo_mes = objetivos().set_index("region").loc[sorted(actual["region"].unique()), "objetivo_mensual"].sum()
        fig.add_scatter(x=mensual["fecha"], y=[objetivo_mes] * len(mensual), name="Objetivo", mode="lines",
                        line=dict(color="#C00000", dash="dash"))
        fig.update_layout(title="Ventas mensuales frente a objetivo", height=380, margin=dict(t=50, b=10),
                          legend=dict(orientation="h", y=-0.12), separators=",.", yaxis_tickformat=",.0f")
        st.plotly_chart(fig, width="stretch")
    with derecha:
        por_region = actual.groupby(["region", "canal"], as_index=False)["importe"].sum()
        fig = px.bar(por_region, x="importe", y="region", color="canal", orientation="h", color_discrete_map=COLORES,
                     title="Ventas por región y canal", height=380, labels={"importe": "", "region": "", "canal": ""})
        fig.update_layout(margin=dict(t=50, b=10), legend=dict(orientation="h", y=-0.12), separators=",.",
                          yaxis={"categoryorder": "total ascending"})
        st.plotly_chart(fig, width="stretch")

    st.subheader("Ranking de productos")
    st.caption("Seleccione una fila para ver el detalle del producto.")
    ranking = (actual.groupby("producto").agg(ventas=("importe", "sum"), unidades=("unidades", "sum"))
               .sort_values("ventas", ascending=False).reset_index())
    ranking["cuota"] = ranking["ventas"] / ranking["ventas"].sum()
    evolucion = actual.set_index("fecha").groupby("producto").resample("ME")["importe"].sum()
    ranking["tendencia"] = [evolucion.loc[p].tolist() for p in ranking["producto"]]
    seleccion = st.dataframe(
        ranking, hide_index=True, width="stretch", on_select="rerun", selection_mode="single-row", key="ranking",
        column_config={"producto": "Producto",
                       "ventas": st.column_config.NumberColumn("Ventas", format="euro"),
                       "unidades": st.column_config.NumberColumn("Unidades", format="localized"),
                       "cuota": st.column_config.ProgressColumn("Cuota", min_value=0, max_value=float(ranking["cuota"].max()), format="percent"),
                       "tendencia": st.column_config.LineChartColumn("Evolución mensual")})
    if seleccion.selection.rows:
        detalle_producto(actual, ranking.iloc[seleccion.selection.rows[0]]["producto"])

    pedidos_de_hoy(df)


# ============================================================ Página: Objetivos
def pagina_objetivos() -> None:
    st.title("Objetivos por región")
    st.write("Modifique los objetivos y pulse **Guardar**. El panel usa los valores guardados.")
    with st.form("objetivos_form"):
        editado = st.data_editor(
            objetivos(), hide_index=True, width="stretch", disabled=["region"], key="editor_objetivos",
            column_config={"region": "Región",
                           "objetivo_mensual": st.column_config.NumberColumn("Objetivo mensual (€)", min_value=0,
                                                                             step=1000, format="%d", required=True),
                           "responsable": st.column_config.TextColumn("Responsable", max_chars=40)})
        guardar = st.form_submit_button("Guardar", type="primary", icon=":material/save:")
    if guardar:
        if (editado["objetivo_mensual"] <= 0).any():
            st.error("Todos los objetivos deben ser mayores que cero.")
        else:
            st.session_state.objetivos = editado.reset_index(drop=True)
            st.toast("Objetivos guardados", icon="✅")
            st.success(f"Objetivo anual total: {eur(editado['objetivo_mensual'].sum() * 12)}")


# ============================================================ Página: Datos
def pagina_datos() -> None:
    st.title("Datos")
    origen = "propios" if "datos_propios" in st.session_state else "de demostración"
    st.info(f"Se están usando datos **{origen}**.")
    subido = st.file_uploader("Cargar un CSV con columnas: " + ", ".join(sorted(COLUMNAS)), type="csv")
    c1, c2 = st.columns(2)
    if subido is not None and c1.button("Usar este archivo", type="primary"):
        try:
            with st.status("Validando archivo…") as estado:
                df = leer_csv(subido.getvalue())
                st.write(f"{len(df):,} filas válidas".replace(",", "."))
                estado.update(label="Archivo cargado", state="complete")
            st.session_state.datos_propios = df
        except (ValueError, pd.errors.ParserError) as e:
            st.error(f"No se pudo cargar el archivo: {e}")
    if "datos_propios" in st.session_state and c2.button("Volver a los datos de demostración"):
        del st.session_state["datos_propios"]
        st.rerun()

    df = datos()
    st.dataframe(df.tail(200), hide_index=True, width="stretch", height=320,
                 column_config={"fecha": st.column_config.DateColumn("Fecha", format="DD/MM/YYYY"),
                                "importe": st.column_config.NumberColumn("Importe (€)", format="%.2f")})
    st.caption(f"Mostrando las últimas 200 de {len(df):,} filas.".replace(",", "."))

    d1, d2 = st.columns(2)
    d1.download_button("Descargar CSV", df.to_csv(index=False, sep=";", decimal=",").encode("utf-8-sig"),
                       file_name="ventas.csv", mime="text/csv", icon=":material/download:")
    buffer = io.BytesIO()
    with pd.ExcelWriter(buffer, engine="openpyxl") as escritor:
        df.to_excel(escritor, sheet_name="Ventas", index=False)
        objetivos().to_excel(escritor, sheet_name="Objetivos", index=False)
    d2.download_button("Descargar Excel", buffer.getvalue(), file_name="ventas.xlsx", icon=":material/table:",
                       mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")


# ============================================================ Navegación
st.set_page_config(page_title="Cuadro de mando comercial", page_icon="📊", layout="wide")
navegacion = st.navigation([
    st.Page(pagina_panel, title="Panel", icon=":material/dashboard:", default=True),
    st.Page(pagina_objetivos, title="Objetivos", icon=":material/flag:"),
    st.Page(pagina_datos, title="Datos", icon=":material/database:"),
])
navegacion.run()
```

Notas:

- Los formatos `"euro"` y `"localized"` de `column_config` usan la **configuración regional del navegador**: con el navegador en español se mostrará `1.607.570,34 €`.
- `width="stretch"` sustituye al parámetro `use_container_width=True` de versiones anteriores.
- El fragmento «Actividad del último día» se refresca cada 30 s sin recalcular el resto de la página.

Prueba automatizada del ejemplo (`test_cuadro_mando.py`, ejecutar con `pytest`):

```python
from streamlit.testing.v1 import AppTest


def test_panel_y_filtros():
    at = AppTest.from_file("cuadro_mando.py", default_timeout=60).run()
    assert not at.exception
    assert at.title[0].value == "Cuadro de mando comercial"
    assert [m.label for m in at.metric[:4]] == ["Ventas", "Pedidos", "Ticket medio", "Cumplimiento del objetivo"]
    ventas_todas = at.metric[0].value
    at.multiselect(key="regiones").set_value(["Norte"]).run()
    assert at.metric[0].value != ventas_todas
    assert at.query_params["regiones"] == "Norte"


def test_sin_regiones_muestra_aviso():
    at = AppTest.from_file("cuadro_mando.py", default_timeout=60).run()
    at.multiselect(key="regiones").set_value([]).run()
    assert "al menos una región" in at.warning[0].value
```

---

## 23. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Warning: to view this Streamlit app on a browser, run it with …` | Se ejecutó con `python app.py`. | `streamlit run app.py`. |
| `StreamlitDuplicateElementId` / `DuplicateWidgetID` | Dos widgets iguales sin `key` distinta. | Asigne `key` única. |
| `StreamlitAPIException: st.session_state.x cannot be modified after the widget … is instantiated` | Se cambia el valor de un widget después de crearlo. | Modifíquelo en una retrollamada (`on_click`/`on_change`) o antes de crearlo. |
| `set_page_config() can only be called once per app page, and must be called as the first Streamlit command` | Hay llamadas a Streamlit antes. | Muévalo al principio (o tras `st.navigation`, antes de dibujar). |
| La aplicación es lenta en cada interacción | Se recalcula todo en cada ejecución. | `@st.cache_data`, formularios, fragmentos. |
| Los datos cacheados no se actualizan | Caché sin caducidad. | `ttl=`, `función.clear()` o botón de recarga. |
| `UnhashableParamError` en funciones cacheadas | Argumento no *hashable* (conexión, modelo). | Prefije el argumento con `_` o use `hash_funcs`. |
| Valores de widgets perdidos al cambiar de página | Los widgets no visibles se eliminan del estado. | Copie los valores a claves propias de `session_state`. |
| El contador o la lista se reinician | Variables normales en lugar de `session_state`. | `st.session_state`. |
| Cambios en el estado no se reflejan hasta el siguiente clic | Se modifica el estado después de dibujar los widgets. | Use retrollamadas o `st.rerun()`. |
| `DeprecationWarning: use_container_width` | Parámetro sustituido en versiones recientes. | `width="stretch"` (o `"content"`). |
| Archivo grande rechazado al subir | Límite de 200 MB. | `server.maxUploadSize` en `config.toml`. |

---

## 24. Buenas prácticas

- Recuerde el modelo de ejecución: todo el script se repite; organice el código en funciones y cachee lo costoso.
- Use `cache_data` para datos y `cache_resource` para conexiones y modelos; defina `ttl` si los datos cambian.
- Agrupe entradas relacionadas en formularios y aísle zonas interactivas en fragmentos.
- Asigne `key` a los widgets que lea desde el estado o desde pruebas.
- Mantenga los secretos en `secrets.toml` o variables de entorno, nunca en el código ni en el repositorio.
- Use `layout="wide"`, columnas y pestañas para paneles densos; muestre primero los indicadores clave.
- Valide las entradas del usuario (archivos subidos, parámetros de URL) antes de procesarlas.
- Escriba pruebas con `AppTest` y fije versiones en `requirements.txt` para despliegues reproducibles.
- Para muchos usuarios simultáneos, controle la memoria por sesión y cachee resultados compartidos.

---

## 25. Referencia rápida

```python
streamlit run app.py · st.set_page_config(page_title=, layout="wide") · st.title/header/subheader/markdown/caption/write
st.dataframe(df, column_config=, on_select="rerun") · st.data_editor(df, num_rows="dynamic") · st.metric(l, v, delta, border=True)
st.line_chart/bar_chart/scatter_chart/map · st.plotly_chart(fig) · st.pyplot(fig)
st.text_input/number_input/slider/date_input/selectbox/multiselect/radio/checkbox/toggle/segmented_control/pills
st.button(l, on_click=, type="primary") · st.download_button(l, datos, file_name=) · st.file_uploader(l, type=)
st.sidebar · st.columns([2, 1]) · st.tabs([...]) · st.expander · st.container(border=True) · st.popover · st.empty()
st.session_state.clave · key="..." · with st.form("id"): ... st.form_submit_button() · st.rerun() · st.stop()
@st.cache_data(ttl="1h") · @st.cache_resource · @st.fragment(run_every="10s") · @st.dialog("Título")
st.navigation([st.Page(f_o_ruta, title=, icon=)]).run() · st.switch_page · st.page_link · st.query_params
st.success/info/warning/error/toast · st.spinner · st.progress · st.status · st.secrets · st.connection("x", type="sql")
st.chat_input · st.chat_message("user") · st.write_stream(generador) · AppTest.from_file("app.py").run()
```

---

## 26. Ejercicios propuestos

1. Añada al cuadro de mando una página de **previsión** que ajuste una tendencia con NumPy o scikit-learn y muestre el intervalo de confianza en el gráfico mensual.
2. Sustituya los datos de demostración por una base SQLite con `st.connection` y un formulario para registrar pedidos nuevos.
3. Cree una aplicación que reciba un Excel, permita elegir columnas y tipo de gráfico, y descargue el gráfico como PNG.
4. Implemente autenticación sencilla (o `st.login`) y muestre a cada responsable solo los datos de su región.
5. Escriba pruebas con `AppTest` para la página de objetivos que verifiquen que no se guardan objetivos negativos.

---

## 27. Referencias

- Documentación oficial: https://docs.streamlit.io/
- Referencia de la API: https://docs.streamlit.io/develop/api-reference
- Conceptos (modelo de ejecución, caché, estado): https://docs.streamlit.io/develop/concepts
- Galería de aplicaciones: https://streamlit.io/gallery
- Community Cloud (despliegue): https://docs.streamlit.io/deploy/streamlit-community-cloud
