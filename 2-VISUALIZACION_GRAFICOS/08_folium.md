# Manual de uso: Folium

> **Categoría:** Visualización de datos · **Utilidad:** Crear mapas interactivos
> **Tipo:** Librería externa (requiere instalación con `pip`). Basada en Leaflet.js.

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer mapa](#4-primer-mapa)
5. [Mapas base (teselas)](#5-mapas-base-teselas)
6. [Marcadores, iconos, *popups* y *tooltips*](#6-marcadores-iconos-popups-y-tooltips)
7. [Formas: círculos, líneas y polígonos](#7-formas-círculos-líneas-y-polígonos)
8. [GeoJSON y estilos según los datos](#8-geojson-y-estilos-según-los-datos)
9. [Mapas de coropletas](#9-mapas-de-coropletas)
10. [Escalas de color y leyendas](#10-escalas-de-color-y-leyendas)
11. [Capas y control de capas](#11-capas-y-control-de-capas)
12. [*Plugins*](#12-plugins)
13. [Trabajar con GeoPandas](#13-trabajar-con-geopandas)
14. [Integración con otras herramientas](#14-integración-con-otras-herramientas)
15. [Guardar, exportar y rendimiento](#15-guardar-exportar-y-rendimiento)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**Folium** permite crear **mapas web interactivos** desde Python. Genera mapas con **Leaflet.js**, la librería JavaScript de mapas más utilizada, y los guarda como archivos HTML que se abren en cualquier navegador: el usuario puede desplazarse, hacer zoom, pulsar en los elementos para ver información y activar o desactivar capas.

**Ventajas:**
- Sintaxis sencilla: un mapa con marcadores en pocas líneas.
- Múltiples mapas base (OpenStreetMap, satélite, topográfico, gris…).
- Soporte directo de **GeoJSON**, **TopoJSON** y **GeoPandas**.
- Mapas de coropletas, mapas de calor, agrupación de marcadores, rutas, dibujo, búsqueda…
- *Popups* con HTML, tablas e incluso gráficos.
- Resultado en un único archivo HTML fácil de compartir o incrustar.

**Limitaciones:**
- Los datos se incrustan en el HTML: con decenas de miles de geometrías el archivo pesa mucho y el navegador se ralentiza.
- La interactividad es de visualización: no hay *callbacks* a Python (salvo integrado en Streamlit o Dash).
- No realiza análisis espacial (para eso se usan GeoPandas o Shapely).
- Los mapas base requieren conexión a internet.

| Si necesita… | Opción |
|---|---|
| Mapa web interactivo para compartir, con capas y *popups* | **Folium** |
| Mapa integrado con otros gráficos interactivos | Plotly (`scatter_map`, `choropleth_map`) |
| Mapas en una aplicación con lógica en Python | Streamlit + `streamlit-folium` / Dash Leaflet |
| Mapas estáticos para informes | GeoPandas + Matplotlib |
| Grandes volúmenes o 3D | pydeck / kepler.gl |

---

## 2. Instalación e importación

```bash
pip install folium
pip install pandas                      # Recomendado
pip install geopandas                   # Opcional: datos geográficos (shapefiles, GeoPackage…)
pip install streamlit-folium            # Opcional: mapas en Streamlit
```

Comprobar la versión:

```bash
python -c "import folium; print(folium.__version__)"
```

Importaciones habituales:

```python
import folium
from folium import plugins
import branca.colormap as cm             # Escalas de color (se instala con folium)
import pandas as pd
```

Visualización:

| Entorno | Cómo |
|---|---|
| Jupyter / VS Code | Deje el objeto `mapa` al final de la celda. |
| Script | `mapa.save("mapa.html")` y abra el archivo en el navegador. |
| Streamlit | `st_folium(mapa)` (paquete `streamlit-folium`). |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `folium.Map` | Mapa base: centro, zoom inicial y teselas. |
| Teselas (*tiles*) | Imágenes del mapa base que se descargan de un servidor (OpenStreetMap, Esri…). |
| Coordenadas | Siempre en el orden **[latitud, longitud]** (sistema WGS84, EPSG:4326). |
| Zoom | Nivel de detalle: 1 (mundo) – 18/19 (calle). España completa ≈ 6; una ciudad ≈ 12. |
| Elemento | Cualquier objeto añadido al mapa: marcadores, formas, capas GeoJSON… Se añade con `.add_to(mapa)`. |
| *Popup* | Ventana que aparece al **hacer clic** en un elemento. |
| *Tooltip* | Texto que aparece al **pasar el ratón**. |
| `FeatureGroup` | Grupo de elementos que se activa o desactiva como una capa. |
| GeoJSON | Formato estándar de geometrías (puntos, líneas, polígonos) con propiedades. |
| Coropleta | Mapa en el que cada zona se colorea según un valor. |

> **Atención al orden:** Folium usa `[lat, lon]`, pero GeoJSON almacena las coordenadas como `[lon, lat]`. Confundirlos sitúa los puntos en el lugar equivocado (a menudo en el océano).

---

## 4. Primer mapa

```python
import folium

mapa = folium.Map(location=[40.4168, -3.7038], zoom_start=6)

folium.Marker(
    location=[40.4168, -3.7038],
    popup="<b>Madrid</b><br>Sede central",
    tooltip="Madrid",
    icon=folium.Icon(color="blue", icon="building", prefix="fa"),
).add_to(mapa)

folium.Marker([41.3874, 2.1686], popup="Barcelona", tooltip="Barcelona").add_to(mapa)

mapa.save("mapa.html")
```

| Parámetro de `folium.Map` | Efecto |
|---|---|
| `location` | Centro `[lat, lon]`. |
| `zoom_start` | Zoom inicial. |
| `tiles` | Mapa base (véase la sección 5). |
| `min_zoom`, `max_zoom` | Límites de zoom. |
| `control_scale=True` | Muestra la escala. |
| `width`, `height` | Tamaño (`"100%"`, `600`). |
| `prefer_canvas=True` | Mejora el rendimiento con muchos elementos vectoriales. |
| `zoom_control=False` | Oculta los botones de zoom. |
| `max_bounds=True` + `min_lat`… | Limita el desplazamiento a una zona. |

Ajustar la vista a un conjunto de puntos:

```python
mapa.fit_bounds([[36.0, -9.5], [43.8, 3.3]])                    # [suroeste, noreste]
mapa.fit_bounds(df[["lat", "lon"]].values.tolist())             # Para que todos los puntos sean visibles
```

---

## 5. Mapas base (teselas)

```python
folium.Map(location=[40, -3.7], zoom_start=6, tiles="OpenStreetMap")          # Por defecto
folium.Map(location=[40, -3.7], zoom_start=6, tiles="Esri.WorldGrayCanvas")   # Gris claro, ideal para datos
folium.Map(location=[40, -3.7], zoom_start=6, tiles="Esri.WorldImagery")      # Satélite
folium.Map(location=[40, -3.7], zoom_start=6, tiles="OpenTopoMap")            # Topográfico
folium.Map(location=[40, -3.7], zoom_start=6, tiles=None)                     # Sin mapa base
```

Folium utiliza el catálogo de proveedores de **xyzservices**: cualquier nombre como `"Esri.WorldStreetMap"` o `"CyclOSM"` es válido.

> Algunos proveedores (por ejemplo, CartoDB o Stadia) exigen ahora una **clave de API**; Folium muestra un aviso si falta. OpenStreetMap, OpenTopoMap y los mapas de Esri funcionan sin clave (respete sus condiciones de uso y la atribución).

Varios mapas base seleccionables:

```python
mapa = folium.Map(location=[40, -3.7], zoom_start=6, tiles=None)
folium.TileLayer("Esri.WorldGrayCanvas", name="Gris").add_to(mapa)
folium.TileLayer("OpenStreetMap", name="Callejero", show=False).add_to(mapa)
folium.TileLayer("Esri.WorldImagery", name="Satélite", show=False).add_to(mapa)
folium.LayerControl().add_to(mapa)
```

> Si no indica `show=False`, el último mapa base añadido es el que se ve al abrir el mapa.

Teselas personalizadas (servidor propio o WMS):

```python
folium.TileLayer(
    tiles="https://{s}.servidor.com/{z}/{x}/{y}.png",
    attr="© Mi organización", name="Mapa corporativo",
).add_to(mapa)

folium.WmsTileLayer(
    url="https://www.ign.es/wms-inspire/pnoa-ma",            # Ortofoto del IGN (España)
    layers="OI.OrthoimageCoverage", name="PNOA", fmt="image/png", transparent=True,
    attr="© Instituto Geográfico Nacional",
).add_to(mapa)
```

---

## 6. Marcadores, iconos, *popups* y *tooltips*

### 6.1 Marcadores con icono

```python
folium.Marker(
    [37.3891, -5.9845],
    tooltip="Tienda Sevilla",
    popup="Ventas 2025: 1,2 M€",
    icon=folium.Icon(color="red", icon="shopping-cart", prefix="fa", icon_color="white"),
).add_to(mapa)
```

| Parámetro de `folium.Icon` | Valores |
|---|---|
| `color` | `red`, `blue`, `green`, `purple`, `orange`, `darkred`, `lightred`, `beige`, `darkblue`, `darkgreen`, `cadetblue`, `darkpurple`, `white`, `pink`, `lightblue`, `lightgreen`, `gray`, `black`, `lightgray` |
| `icon` | Nombre del icono (Bootstrap Glyphicons o Font Awesome). |
| `prefix` | `"glyphicon"` (por defecto) o `"fa"` (Font Awesome). |
| `icon_color` | Color del símbolo (cualquier color CSS). |

### 6.2 Marcadores circulares (tamaño según el dato)

```python
for _, fila in df.iterrows():
    folium.CircleMarker(
        location=[fila["lat"], fila["lon"]],
        radius=4 + fila["ventas"] / df["ventas"].max() * 20,     # Radio en píxeles
        color="white", weight=1,
        fill=True, fill_color="#1F4E78", fill_opacity=0.75,
        tooltip=f"{fila['tienda']}: {fila['ventas']:,.0f} €",
    ).add_to(mapa)
```

> `CircleMarker` usa un radio en **píxeles** (constante al hacer zoom); `Circle` usa **metros** (crece al acercarse).

### 6.3 Iconos personalizados

```python
folium.Marker([40.41, -3.70], icon=folium.CustomIcon("logo.png", icon_size=(32, 32))).add_to(mapa)

folium.Marker([40.41, -3.70], icon=folium.DivIcon(
    html='<div style="background:#1F4E78;color:white;border-radius:12px;padding:2px 8px;'
         'font:bold 12px Arial;white-space:nowrap">1,2 M€</div>',
    icon_size=(70, 20), icon_anchor=(35, 10),
)).add_to(mapa)
```

### 6.4 *Popups* con HTML

```python
html = f"""
<div style="font-family:Arial; width:220px">
  <h4 style="margin:0 0 6px;color:#1F4E78">{tienda}</h4>
  <table style="width:100%;font-size:12px">
    <tr><td>Ventas</td><td style="text-align:right"><b>{ventas:,.0f} €</b></td></tr>
    <tr><td>Empleados</td><td style="text-align:right">{empleados}</td></tr>
  </table>
</div>"""
folium.Marker([lat, lon], popup=folium.Popup(html, max_width=260)).add_to(mapa)
```

| Opción de `folium.Popup` | Efecto |
|---|---|
| `max_width` | Ancho máximo en píxeles. |
| `show=True` | Abierto al cargar el mapa. |
| `sticky=True` | Permanece abierto al hacer clic en otro lugar. |
| `lazy=True` | Se genera al abrirlo (mejora el rendimiento con muchos marcadores). |

*Popup* con un gráfico (Vega-Lite / Altair):

```python
grafico = alt.Chart(df_tienda).mark_bar().encode(x="mes:O", y="ventas:Q").properties(width=250, height=120)
folium.Marker([lat, lon], popup=folium.Popup().add_child(folium.VegaLite(grafico, width=300, height=160))).add_to(mapa)
```

### 6.5 *Tooltips*

```python
folium.Marker([lat, lon], tooltip=folium.Tooltip("<b>Madrid</b>", sticky=True)).add_to(mapa)
folium.Marker([lat, lon], tooltip=folium.Tooltip("Madrid", permanent=True, direction="right")).add_to(mapa)  # Etiqueta fija
```

---

## 7. Formas: círculos, líneas y polígonos

```python
# Círculo en metros (área de influencia de 25 km)
folium.Circle([40.4168, -3.7038], radius=25_000, color="#1F4E78", fill=True, fill_opacity=0.1,
              tooltip="Radio de reparto: 25 km").add_to(mapa)

# Línea (ruta)
ruta = [[40.4168, -3.7038], [39.4699, -0.3763], [41.3874, 2.1686]]
folium.PolyLine(ruta, color="#C00000", weight=4, opacity=0.8, dash_array="8,6", tooltip="Ruta logística").add_to(mapa)

# Polígono
zona = [[40.50, -3.80], [40.50, -3.60], [40.35, -3.60], [40.35, -3.80]]
folium.Polygon(zona, color="green", fill=True, fill_color="green", fill_opacity=0.2, popup="Zona norte").add_to(mapa)

# Rectángulo
folium.Rectangle([[40.30, -3.90], [40.55, -3.50]], color="orange", fill=False).add_to(mapa)
```

| Opción de estilo | Efecto |
|---|---|
| `color`, `weight`, `opacity` | Color, grosor y opacidad del borde. |
| `fill`, `fill_color`, `fill_opacity` | Relleno. |
| `dash_array` | Línea discontinua (`"5,5"`). |
| `line_cap`, `line_join` | Extremos y uniones (`"round"`). |

---

## 8. GeoJSON y estilos según los datos

### 8.1 Cargar un GeoJSON

```python
folium.GeoJson("provincias.geojson", name="Provincias").add_to(mapa)      # Ruta, URL o diccionario
```

### 8.2 Estilo, resaltado, *tooltip* y *popup*

```python
import json
with open("provincias.geojson", encoding="utf-8") as f:
    geo = json.load(f)

def estilo(feature):
    poblacion = feature["properties"]["poblacion"]
    return {"fillColor": "#C00000" if poblacion > 1_000_000 else "#9DC3E6",
            "color": "white", "weight": 1, "fillOpacity": 0.7}

folium.GeoJson(
    geo,
    name="Provincias",
    style_function=estilo,
    highlight_function=lambda f: {"weight": 3, "color": "#1F4E78", "fillOpacity": 0.9},
    tooltip=folium.GeoJsonTooltip(fields=["nombre", "poblacion"], aliases=["Provincia", "Población"],
                                  localize=True, sticky=True),
    popup=folium.GeoJsonPopup(fields=["nombre", "capital"], aliases=["Provincia", "Capital"]),
    zoom_on_click=True,
).add_to(mapa)
```

> `localize=True` muestra los números con el formato del navegador (en un navegador en español: `1.234.567`).

### 8.3 Puntos de un GeoJSON con marcadores propios

```python
folium.GeoJson(
    geo_puntos,
    marker=folium.CircleMarker(radius=6, fill=True, fill_opacity=0.8),
    style_function=lambda f: {"fillColor": "red" if f["properties"]["estado"] == "Cerrada" else "green",
                              "color": "white"},
).add_to(mapa)
```

### 8.4 TopoJSON

```python
folium.TopoJson(json.load(open("municipios.topojson")), "objects.municipios", name="Municipios").add_to(mapa)
```

---

## 9. Mapas de coropletas

`folium.Choropleth` une un GeoJSON con una tabla de datos y colorea cada zona.

```python
import pandas as pd

datos = pd.DataFrame({"codigo": ["28", "08", "41", "46"], "ventas": [5.2, 4.1, 1.8, 2.3]})

folium.Choropleth(
    geo_data="provincias.geojson",
    data=datos,
    columns=["codigo", "ventas"],               # [clave, valor]
    key_on="feature.properties.cod_prov",       # Ruta a la clave dentro del GeoJSON
    fill_color="Blues",                         # Paleta ColorBrewer
    fill_opacity=0.75, line_opacity=0.3,
    nan_fill_color="#EEEEEE",                   # Zonas sin datos
    bins=[0, 1, 2, 3, 4, 6],                    # Límites de las clases (o un número)
    legend_name="Ventas 2025 (M€)",
    highlight=True,
    name="Ventas por provincia",
).add_to(mapa)
```

| Parámetro | Descripción |
|---|---|
| `columns` | Columna de unión y columna de valor del DataFrame. |
| `key_on` | Propiedad del GeoJSON que coincide con la clave (`feature.properties.xxx` o `feature.id`). |
| `fill_color` | Paleta ColorBrewer: `Blues`, `Greens`, `YlOrRd`, `BuPu`, `RdYlGn`, `PuBu`… |
| `bins` | Número de clases o lista de límites. |
| `use_jenks=True` | Clases por cortes naturales (requiere `jenkspy`). |

> El tipo de la clave debe coincidir: `"08"` (texto) no es igual a `8` (número). Normalice con `df["codigo"] = df["codigo"].astype(str).str.zfill(2)`.

Añadir *tooltip* a una coropleta (los valores deben estar en las propiedades del GeoJSON):

```python
coropleta = folium.Choropleth(...).add_to(mapa)
for feature in coropleta.geojson.data["features"]:
    codigo = feature["properties"]["cod_prov"]
    feature["properties"]["ventas"] = float(datos.set_index("codigo")["ventas"].get(codigo, 0))
folium.GeoJsonTooltip(["nombre", "ventas"], aliases=["Provincia", "Ventas (M€)"]).add_to(coropleta.geojson)
```

Alternativa más flexible: `folium.GeoJson` con `style_function` y una escala de `branca` (sección 10), que permite *tooltips* y estilos libres en un solo paso.

---

## 10. Escalas de color y leyendas

```python
import branca.colormap as cm

escala = cm.LinearColormap(["#DEEBF7", "#9ECAE1", "#3182BD", "#08519C"], vmin=0, vmax=6,
                           caption="Ventas (M€)")
escala = cm.linear.YlOrRd_09.scale(0, 100)                     # Paletas predefinidas
escala = cm.StepColormap(["#2E7D32", "#FBC02D", "#C00000"], index=[0, 50, 80, 100],
                         vmin=0, vmax=100, caption="Ocupación (%)")
escala.caption = "Ocupación (%)"

color = escala(42)                                             # → '#...'
escala.add_to(mapa)                                            # Leyenda en el mapa
```

Uso con GeoJSON:

```python
folium.GeoJson(geo, style_function=lambda f: {
    "fillColor": escala(f["properties"]["ventas"]), "color": "white", "weight": 1, "fillOpacity": 0.75,
}).add_to(mapa)
```

Leyenda categórica personalizada (HTML):

```python
leyenda = """
<div style="position: fixed; bottom: 30px; left: 30px; z-index: 9999; background: white;
            padding: 10px 14px; border-radius: 6px; box-shadow: 0 1px 4px rgba(0,0,0,.3); font: 12px Arial">
  <b>Tipo de tienda</b><br>
  <i style="background:#1F4E78;width:10px;height:10px;display:inline-block;border-radius:50%"></i> Propia<br>
  <i style="background:#C55A11;width:10px;height:10px;display:inline-block;border-radius:50%"></i> Franquicia
</div>"""
mapa.get_root().html.add_child(folium.Element(leyenda))
```

---

## 11. Capas y control de capas

```python
tiendas = folium.FeatureGroup(name="Tiendas", show=True).add_to(mapa)
almacenes = folium.FeatureGroup(name="Almacenes", show=False).add_to(mapa)   # Oculta al inicio

folium.Marker([40.41, -3.70], tooltip="Tienda Madrid").add_to(tiendas)
folium.Marker([40.30, -3.75], tooltip="Almacén Getafe", icon=folium.Icon(color="gray")).add_to(almacenes)

folium.LayerControl(collapsed=False, position="topright").add_to(mapa)     # Siempre al final
```

| Tipo de capa | Uso |
|---|---|
| `folium.FeatureGroup` | Agrupa elementos y aparece en el control de capas. |
| `folium.TileLayer(overlay=False)` | Mapa base (selección excluyente). |
| `plugins.FeatureGroupSubGroup` | Subgrupos dentro de un grupo (p. ej., subcapas de un `MarkerCluster`). |
| `plugins.GroupedLayerControl` | Control de capas agrupado por categorías, con opciones excluyentes. |

> `folium.LayerControl()` debe añadirse **después** de todas las capas.

---

## 12. *Plugins*

```python
from folium import plugins
```

### 12.1 Agrupación de marcadores (`MarkerCluster`)

```python
cluster = plugins.MarkerCluster(name="Clientes", options={"maxClusterRadius": 50}).add_to(mapa)
for _, c in clientes.iterrows():
    folium.Marker([c["lat"], c["lon"]], tooltip=c["nombre"]).add_to(cluster)

plugins.FastMarkerCluster(clientes[["lat", "lon"]].values.tolist(), name="Clientes").add_to(mapa)  # Miles de puntos
```

### 12.2 Mapa de calor (`HeatMap`)

```python
plugins.HeatMap(
    clientes[["lat", "lon", "importe"]].values.tolist(),       # [lat, lon, peso]
    name="Densidad de ventas", radius=18, blur=15, min_opacity=0.3,
    gradient={0.2: "blue", 0.5: "lime", 0.8: "orange", 1: "red"},
).add_to(mapa)
```

Mapa de calor animado en el tiempo:

```python
datos_por_mes = [grupo[["lat", "lon"]].values.tolist() for _, grupo in df.groupby("mes")]
plugins.HeatMapWithTime(datos_por_mes, index=[str(m) for m in df["mes"].unique()], radius=15).add_to(mapa)
```

### 12.3 Controles de interfaz

```python
plugins.Fullscreen(position="topright", title="Pantalla completa").add_to(mapa)
plugins.MiniMap(toggle_display=True, position="bottomright").add_to(mapa)
plugins.MeasureControl(primary_length_unit="kilometers", primary_area_unit="hectares").add_to(mapa)
plugins.MousePosition(position="bottomleft", separator=" | ", num_digits=5).add_to(mapa)
plugins.LocateControl().add_to(mapa)                         # Botón «mi ubicación»
plugins.Geocoder(collapsed=True).add_to(mapa)                # Buscador de direcciones (servicio externo)
plugins.Draw(export=True).add_to(mapa)                       # Dibujar formas y exportarlas a GeoJSON
plugins.ScrollZoomToggler().add_to(mapa)                     # Activar/desactivar zoom con la rueda
```

### 12.4 Búsqueda en una capa

```python
capa_tiendas = folium.GeoJson(geo_tiendas, name="Tiendas",
                              tooltip=folium.GeoJsonTooltip(["nombre"])).add_to(mapa)
plugins.Search(layer=capa_tiendas, search_label="nombre", placeholder="Buscar tienda…", collapsed=False).add_to(mapa)
```

### 12.5 Otros *plugins* útiles

| *Plugin* | Uso |
|---|---|
| `AntPath` | Línea animada (rutas, flujos). |
| `PolyLineTextPath` | Texto a lo largo de una línea. |
| `TimestampedGeoJson` | Elementos que aparecen según la fecha (animación temporal). |
| `DualMap` | Dos mapas sincronizados lado a lado. |
| `SideBySideLayers` | Comparar dos capas con una cortinilla. |
| `BeautifyIcon` | Iconos con forma, borde y número. |
| `TagFilterButton` | Filtrar marcadores por etiquetas. |
| `Realtime` | Actualizar una capa periódicamente desde una URL. |

---

## 13. Trabajar con GeoPandas

GeoPandas lee formatos geográficos (Shapefile, GeoPackage, GeoJSON, KML…) y Folium dibuja sus `GeoDataFrame` directamente.

```python
import geopandas as gpd

provincias = gpd.read_file("provincias.gpkg").to_crs(epsg=4326)    # Folium necesita WGS84 (EPSG:4326)
provincias = provincias.merge(ventas, on="codigo", how="left")

folium.GeoJson(
    provincias,
    style_function=lambda f: {"fillColor": escala(f["properties"]["ventas"] or 0),
                              "color": "white", "weight": 0.8, "fillOpacity": 0.75},
    tooltip=folium.GeoJsonTooltip(["nombre", "ventas"], aliases=["Provincia", "Ventas"]),
).add_to(mapa)
```

Mapa rápido con `explore()` (usa Folium internamente):

```python
mapa = provincias.explore(column="ventas", cmap="Blues", tooltip=["nombre", "ventas"],
                          tiles="Esri.WorldGrayCanvas", legend=True, name="Ventas")
mapa.save("ventas_provincias.html")
```

Reducir el peso de geometrías complejas:

```python
provincias["geometry"] = provincias.simplify(tolerance=0.01, preserve_topology=True)
```

Convertir un DataFrame con lat/lon en GeoDataFrame:

```python
gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df["lon"], df["lat"]), crs="EPSG:4326")
```

---

## 14. Integración con otras herramientas

| Entorno | Cómo |
|---|---|
| Jupyter | Deje `mapa` al final de la celda. |
| Streamlit | `from streamlit_folium import st_folium` → `datos = st_folium(mapa, width=800, height=500)`; devuelve el último clic, la vista y los objetos dibujados. |
| Dash | `html.Iframe(srcDoc=mapa.get_root().render(), width="100%", height="600")` o la librería `dash-leaflet`. |
| Flask / Django | `mapa.get_root().render()` o `mapa._repr_html_()` dentro de la plantilla. |
| Informes HTML | Incruste el archivo con `<iframe src="mapa.html">`. |

Ejemplo en Streamlit:

```python
import streamlit as st
from streamlit_folium import st_folium

mapa = folium.Map([40.4, -3.7], zoom_start=6)
...
resultado = st_folium(mapa, width=900, height=520, returned_objects=["last_object_clicked_tooltip"])
if resultado and resultado.get("last_object_clicked_tooltip"):
    st.write("Ha seleccionado:", resultado["last_object_clicked_tooltip"])
```

---

## 15. Guardar, exportar y rendimiento

### 15.1 Guardar

```python
mapa.save("mapa.html")                        # Archivo autónomo (las teselas se cargan de internet)
html = mapa.get_root().render()               # Texto HTML completo
```

Título de la pestaña del navegador y CSS adicional:

```python
mapa.get_root().header.add_child(folium.Element("<title>Red de tiendas</title>"))
mapa.get_root().header.add_child(folium.Element("<style>.leaflet-tooltip{font-family:Arial}</style>"))
```

Título visible sobre el mapa:

```python
titulo = '<h3 style="position:fixed;top:10px;left:60px;z-index:9999;background:white;padding:6px 12px;' \
         'border-radius:6px;box-shadow:0 1px 4px rgba(0,0,0,.3);font-family:Arial;margin:0">Red de tiendas</h3>'
mapa.get_root().html.add_child(folium.Element(titulo))
```

### 15.2 Imagen estática

Folium no exporta imágenes directamente. Opciones:

- Capturar el HTML con un navegador sin interfaz (Selenium o Playwright).
- Para mapas estáticos de informe, usar **GeoPandas + Matplotlib** (`gdf.plot(column="ventas")`) con `contextily` para el mapa base.

### 15.3 Rendimiento

| Situación | Solución |
|---|---|
| Miles de marcadores | `FastMarkerCluster` o `MarkerCluster`; `CircleMarker` con `prefer_canvas=True`. |
| Muchos puntos solo para ver densidad | `HeatMap`. |
| Polígonos muy detallados | Simplifique las geometrías (`simplify`) y elimine propiedades innecesarias. |
| Archivo HTML pesado | Reduzca decimales de coordenadas (5 decimales ≈ 1 m de precisión). |
| *Popups* complejos en muchos elementos | `folium.Popup(..., lazy=True)`. |
| Capas poco usadas | Créelas con `show=False`. |

---

## 16. Ejemplo práctico completo

**Mapa de la red comercial**: a partir de datos de tiendas, almacenes y clientes, construye un mapa interactivo con varios mapas base; tiendas con icono según su tipo y *popup* con tabla de indicadores; áreas de influencia; almacenes y rutas de reparto; clientes agrupados (`MarkerCluster`); mapa de calor de ventas; una **rejilla hexagonal** coloreada por facturación (coropleta construida en Python con GeoJSON y una escala de `branca`); leyendas, título, control de capas, búsqueda de tiendas, minimapa, pantalla completa y medición de distancias. El resultado es un único HTML.

```python
"""
Mapa interactivo de la red comercial con Folium.
Requiere: pip install folium pandas numpy
"""
import math
from pathlib import Path

import branca.colormap as cm
import folium
import numpy as np
import pandas as pd
from folium import plugins

SALIDA = Path(__file__).resolve().parent / "salida_folium"
AZUL, NARANJA, GRIS = "#1F4E78", "#C55A11", "#595959"

TIENDAS = pd.DataFrame([
    ("Madrid Centro", "Propia", 40.4168, -3.7038, 1_850_000, 24), ("Madrid Norte", "Propia", 40.4800, -3.6900, 1_240_000, 16),
    ("Barcelona", "Propia", 41.3874, 2.1686, 1_720_000, 22), ("Valencia", "Franquicia", 39.4699, -0.3763, 980_000, 12),
    ("Sevilla", "Propia", 37.3891, -5.9845, 1_050_000, 14), ("Málaga", "Franquicia", 36.7213, -4.4214, 760_000, 9),
    ("Bilbao", "Propia", 43.2630, -2.9350, 890_000, 11), ("Zaragoza", "Franquicia", 41.6488, -0.8891, 640_000, 8),
    ("Valladolid", "Franquicia", 41.6523, -4.7245, 520_000, 7), ("A Coruña", "Franquicia", 43.3623, -8.4115, 470_000, 6),
    ("Murcia", "Franquicia", 37.9922, -1.1307, 590_000, 8), ("Palma", "Propia", 39.5696, 2.6502, 710_000, 9),
], columns=["tienda", "tipo", "lat", "lon", "ventas", "empleados"])

ALMACENES = pd.DataFrame([
    ("Almacén Centro (Getafe)", 40.3057, -3.7329, ["Madrid Centro", "Madrid Norte", "Valladolid", "Zaragoza", "Bilbao", "A Coruña"]),
    ("Almacén Este (Sagunto)", 39.6766, -0.2733, ["Valencia", "Barcelona", "Murcia", "Palma"]),
    ("Almacén Sur (Antequera)", 37.0194, -4.5612, ["Sevilla", "Málaga"]),
], columns=["almacen", "lat", "lon", "destinos"])


# ------------------------------------------------------------------ Datos
def generar_clientes(n: int = 4_000, semilla: int = 12) -> pd.DataFrame:
    """Clientes repartidos alrededor de las tiendas, más cerca cuanto más vende la tienda."""
    rng = np.random.default_rng(semilla)
    pesos = TIENDAS["ventas"] / TIENDAS["ventas"].sum()
    origen = TIENDAS.iloc[rng.choice(len(TIENDAS), n, p=pesos)].reset_index(drop=True)
    distancia_km = rng.gamma(2.0, 9.0, n)
    angulo = rng.uniform(0, 2 * np.pi, n)
    lat = origen["lat"] + distancia_km / 111 * np.sin(angulo)
    lon = origen["lon"] + distancia_km / (111 * np.cos(np.radians(origen["lat"]))) * np.cos(angulo)
    return pd.DataFrame({
        "cliente": [f"C{i:05d}" for i in range(n)], "tienda": origen["tienda"],
        "lat": lat.round(5), "lon": lon.round(5),
        "importe": rng.lognormal(5.6, 0.7, n).round(2),
        "segmento": rng.choice(["Particular", "Empresa"], n, p=[0.8, 0.2]),
    })


def rejilla_hexagonal(clientes: pd.DataFrame, radio_km: float = 12) -> dict:
    """Agrupa los clientes en hexágonos y devuelve un GeoJSON con la facturación de cada celda."""
    lat0 = clientes["lat"].mean()
    km_lon = 111 * math.cos(math.radians(lat0))
    x, y = clientes["lon"] * km_lon, clientes["lat"] * 111                     # Proyección local en km
    ancho, alto = math.sqrt(3) * radio_km, 1.5 * radio_km
    fila = np.round(y / alto).astype(int)
    columna = np.round((x - (fila % 2) * ancho / 2) / ancho).astype(int)
    agrupado = (clientes.assign(fila=fila, columna=columna)
                .groupby(["fila", "columna"]).agg(importe=("importe", "sum"), clientes=("cliente", "count"))
                .reset_index())

    features = []
    for _, celda in agrupado.iterrows():
        cx = celda["columna"] * ancho + (celda["fila"] % 2) * ancho / 2
        cy = celda["fila"] * alto
        vertices = [[round((cx + radio_km * math.cos(math.radians(60 * k + 30))) / km_lon, 5),
                     round((cy + radio_km * math.sin(math.radians(60 * k + 30))) / 111, 5)] for k in range(7)]
        features.append({"type": "Feature",
                         "geometry": {"type": "Polygon", "coordinates": [vertices]},     # GeoJSON: [lon, lat]
                         "properties": {"importe": round(float(celda["importe"]), 0),
                                        "clientes": int(celda["clientes"]),
                                        "importe_txt": f"{celda['importe']:,.0f} €".replace(",", ".")}})
    return {"type": "FeatureCollection", "features": features}


def euros(valor: float) -> str:
    return f"{valor:,.0f} €".replace(",", ".")


# ------------------------------------------------------------------ Capas
def popup_tienda(t: pd.Series, clientes: pd.DataFrame) -> folium.Popup:
    propios = clientes[clientes["tienda"] == t["tienda"]]
    filas = [("Tipo", t["tipo"]), ("Ventas 2025", euros(t["ventas"])), ("Empleados", t["empleados"]),
             ("Ventas por empleado", euros(t["ventas"] / t["empleados"])),
             ("Clientes", f"{len(propios):,}".replace(",", ".")),
             ("% empresas", f"{(propios['segmento'] == 'Empresa').mean():.0%}")]
    tabla = "".join(f"<tr><td style='color:#7F7F7F;padding:2px 8px 2px 0'>{k}</td>"
                    f"<td style='text-align:right;font-weight:bold'>{v}</td></tr>" for k, v in filas)
    html = (f"<div style='font-family:Arial;font-size:12px;width:230px'>"
            f"<div style='color:{AZUL};font-size:15px;font-weight:bold;margin-bottom:6px'>{t['tienda']}</div>"
            f"<table style='width:100%'>{tabla}</table></div>")
    return folium.Popup(html, max_width=260)


def capa_tiendas(mapa: folium.Map, clientes: pd.DataFrame) -> folium.GeoJson:
    grupo = folium.FeatureGroup(name="🏬 Tiendas", show=True).add_to(mapa)
    for _, t in TIENDAS.iterrows():
        color = "darkblue" if t["tipo"] == "Propia" else "orange"
        folium.Marker([t["lat"], t["lon"]], tooltip=f"<b>{t['tienda']}</b> · {euros(t['ventas'])}",
                      popup=popup_tienda(t, clientes),
                      icon=folium.Icon(color=color, icon="store", prefix="fa")).add_to(grupo)

    # Capa GeoJSON invisible para el buscador
    geo = {"type": "FeatureCollection", "features": [
        {"type": "Feature", "geometry": {"type": "Point", "coordinates": [t["lon"], t["lat"]]},
         "properties": {"tienda": t["tienda"]}} for _, t in TIENDAS.iterrows()]}
    return folium.GeoJson(geo, name="Buscador", control=False,
                          marker=folium.CircleMarker(radius=1, opacity=0, fill_opacity=0)).add_to(mapa)


def capa_influencia(mapa: folium.Map) -> None:
    grupo = folium.FeatureGroup(name="⭕ Áreas de influencia", show=False).add_to(mapa)
    for _, t in TIENDAS.iterrows():
        radio = 10_000 + 25_000 * math.sqrt(t["ventas"] / TIENDAS["ventas"].max())
        folium.Circle([t["lat"], t["lon"]], radius=radio, color=AZUL if t["tipo"] == "Propia" else NARANJA,
                      weight=1, fill=True, fill_opacity=0.08,
                      tooltip=f"{t['tienda']}: radio {radio / 1000:.0f} km").add_to(grupo)


def capa_logistica(mapa: folium.Map) -> None:
    grupo = folium.FeatureGroup(name="🚚 Almacenes y rutas", show=True).add_to(mapa)
    posiciones = TIENDAS.set_index("tienda")[["lat", "lon"]]
    for _, a in ALMACENES.iterrows():
        folium.Marker([a["lat"], a["lon"]], tooltip=a["almacen"],
                      icon=folium.Icon(color="gray", icon="warehouse", prefix="fa")).add_to(grupo)
        for destino in a["destinos"]:
            lat, lon = posiciones.loc[destino]
            distancia = 6371 * 2 * math.asin(math.sqrt(
                math.sin(math.radians(lat - a["lat"]) / 2) ** 2
                + math.cos(math.radians(a["lat"])) * math.cos(math.radians(lat))
                * math.sin(math.radians(lon - a["lon"]) / 2) ** 2))
            plugins.AntPath([[a["lat"], a["lon"]], [lat, lon]], color=GRIS, pulse_color="#BFBFBF",
                            weight=2, delay=1200, dash_array=[8, 20],
                            tooltip=f"{a['almacen']} → {destino}: {distancia:,.0f} km en línea recta").add_to(grupo)


def capa_clientes(mapa: folium.Map, clientes: pd.DataFrame) -> None:
    cluster = plugins.MarkerCluster(name="👥 Clientes (agrupados)", show=False,
                                    options={"maxClusterRadius": 45, "disableClusteringAtZoom": 13}).add_to(mapa)
    for _, c in clientes.iterrows():
        folium.CircleMarker([c["lat"], c["lon"]], radius=4, weight=0, fill=True, fill_opacity=0.8,
                            fill_color=AZUL if c["segmento"] == "Particular" else NARANJA,
                            tooltip=f"{c['cliente']} · {c['segmento']} · {euros(c['importe'])}").add_to(cluster)


def capa_calor(mapa: folium.Map, clientes: pd.DataFrame) -> None:
    plugins.HeatMap(clientes[["lat", "lon", "importe"]].values.tolist(), name="🔥 Mapa de calor de ventas",
                    radius=16, blur=18, min_opacity=0.25, show=False,
                    gradient={0.2: "#2C7BB6", 0.45: "#ABD9E9", 0.65: "#FFFFBF", 0.85: "#FDAE61", 1: "#D7191C"}
                    ).add_to(mapa)


def capa_hexagonos(mapa: folium.Map, clientes: pd.DataFrame) -> None:
    geo = rejilla_hexagonal(clientes)
    valores = [f["properties"]["importe"] for f in geo["features"]]
    tope = float(np.quantile(valores, 0.95))                    # Evita que unos pocos valores extremos dominen
    escala = cm.LinearColormap(["#EFF3FF", "#BDD7E7", "#6BAED6", "#3182BD", "#08519C"], vmin=0, vmax=tope,
                               caption="Facturación por hexágono de 12 km (€)")
    folium.GeoJson(
        geo, name="⬢ Facturación por zona",
        style_function=lambda f: {"fillColor": escala(min(f["properties"]["importe"], tope)), "color": "white",
                                  "weight": 0.6, "fillOpacity": 0.75},
        highlight_function=lambda f: {"weight": 2.5, "color": AZUL, "fillOpacity": 0.9},
        tooltip=folium.GeoJsonTooltip(["importe_txt", "clientes"], aliases=["Facturación", "Clientes"],
                                      localize=True),
    ).add_to(mapa)
    escala.add_to(mapa)


def elementos_fijos(mapa: folium.Map) -> None:
    titulo = f"""
    <div style="position:fixed;top:12px;left:55px;z-index:9999;background:white;padding:8px 14px;
                border-radius:6px;box-shadow:0 1px 5px rgba(0,0,0,.25);font-family:Arial">
      <div style="font-size:17px;font-weight:bold;color:{AZUL}">Red comercial 2025</div>
      <div style="font-size:11px;color:#7F7F7F">{len(TIENDAS)} tiendas · {len(ALMACENES)} almacenes ·
        ventas totales {euros(TIENDAS['ventas'].sum())}</div>
    </div>"""
    leyenda = f"""
    <div style="position:fixed;bottom:28px;left:12px;z-index:9999;background:white;padding:8px 12px;
                border-radius:6px;box-shadow:0 1px 5px rgba(0,0,0,.25);font:12px Arial;line-height:20px">
      <b>Leyenda</b><br>
      <i class="fa fa-store" style="color:{AZUL};width:16px"></i> Tienda propia<br>
      <i class="fa fa-store" style="color:#F69730;width:16px"></i> Franquicia<br>
      <i class="fa fa-warehouse" style="color:{GRIS};width:16px"></i> Almacén<br>
      <span style="color:{AZUL}">●</span> Cliente particular &nbsp;
      <span style="color:{NARANJA}">●</span> Empresa
    </div>"""
    raiz = mapa.get_root()
    raiz.html.add_child(folium.Element(titulo))
    raiz.html.add_child(folium.Element(leyenda))
    raiz.header.add_child(folium.Element("<title>Red comercial 2025</title>"))


# ------------------------------------------------------------------ Mapa
def construir_mapa() -> folium.Map:
    clientes = generar_clientes()
    mapa = folium.Map(location=[40.0, -3.6], zoom_start=6, tiles=None, control_scale=True, prefer_canvas=True)
    folium.TileLayer("Esri.WorldGrayCanvas", name="Mapa gris").add_to(mapa)
    folium.TileLayer("OpenStreetMap", name="Callejero", show=False).add_to(mapa)
    folium.TileLayer("Esri.WorldImagery", name="Satélite", show=False).add_to(mapa)

    capa_hexagonos(mapa, clientes)
    capa_calor(mapa, clientes)
    capa_influencia(mapa)
    capa_logistica(mapa)
    capa_clientes(mapa, clientes)
    buscador = capa_tiendas(mapa, clientes)

    plugins.Search(layer=buscador, search_label="tienda", placeholder="Buscar tienda…",
                   collapsed=True, position="topright", search_zoom=12).add_to(mapa)
    plugins.Fullscreen(position="topright", title="Pantalla completa", title_cancel="Salir").add_to(mapa)
    plugins.MiniMap(toggle_display=True, position="bottomright", width=130, height=130).add_to(mapa)
    plugins.MeasureControl(position="topleft", primary_length_unit="kilometers",
                           secondary_length_unit=None, primary_area_unit="hectares").add_to(mapa)
    plugins.MousePosition(position="bottomright", separator=" · ", num_digits=4, prefix="Lat/Lon:").add_to(mapa)
    elementos_fijos(mapa)
    folium.LayerControl(collapsed=False, position="topright").add_to(mapa)
    mapa.fit_bounds([[TIENDAS["lat"].min() - 0.5, TIENDAS["lon"].min() - 0.5],
                     [TIENDAS["lat"].max() + 0.5, TIENDAS["lon"].max() + 0.5]])
    return mapa


def main() -> None:
    SALIDA.mkdir(exist_ok=True)
    ruta = SALIDA / "red_comercial.html"
    construir_mapa().save(ruta)
    print(f"Mapa generado: {ruta} ({ruta.stat().st_size / 1_048_576:.1f} MB)")


if __name__ == "__main__":
    main()
```

Abra `red_comercial.html`: active y desactive capas en el panel superior derecho, pulse en una tienda para ver sus indicadores, busque una tienda por nombre y mida distancias con la regla.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Los puntos aparecen en el océano o en otro continente | Latitud y longitud invertidas. | Folium: `[lat, lon]`; GeoJSON: `[lon, lat]`. |
| El mapa se ve gris, sin mapa base | Sin conexión o proveedor que exige clave. | Compruebe la conexión; use OpenStreetMap o Esri. |
| `UserWarning: CartoDB tiles now require an API key` | CartoDB exige clave. | Indique la clave o cambie a `"Esri.WorldGrayCanvas"`. |
| La coropleta sale toda negra o gris | `key_on` no coincide con las claves del DataFrame. | Revise la ruta `feature.properties.x` y el tipo (`"08"` frente a `8`). |
| `ValueError: Cannot render objects with any missing geometries` | Geometrías nulas en el GeoDataFrame. | `gdf = gdf[gdf.geometry.notna()]`. |
| Geometrías desplazadas al usar GeoPandas | Sistema de referencia distinto de WGS84. | `gdf.to_crs(epsg=4326)`. |
| `TypeError: Object of type int64 is not JSON serializable` | Tipos de NumPy en propiedades o *popups*. | Convierta con `int()` / `float()`. |
| El HTML pesa decenas de MB y va lento | Demasiados elementos o geometrías detalladas. | Simplifique, use `FastMarkerCluster`, `HeatMap` o reduzca decimales. |
| `LayerControl` no muestra todas las capas | Se añadió antes que algunas capas. | Añádalo al final. |
| Los iconos de Font Awesome no aparecen | Falta `prefix="fa"` o el nombre no existe. | `folium.Icon(icon="store", prefix="fa")`. |
| El *tooltip* de `GeoJsonTooltip` falla | El campo no existe en `properties`. | Revise los nombres de propiedades del GeoJSON. |
| El mapa no se muestra en Jupyter | Cuaderno no confiable o salida bloqueada. | *File → Trust Notebook* o guarde a HTML. |
| Caracteres extraños (tildes, «€») | Codificación al leer el GeoJSON. | `open(..., encoding="utf-8")`. |

---

## 18. Buenas prácticas

- Centre el mapa y ajuste el zoom a los datos (`fit_bounds`).
- Use un mapa base **neutro** (gris) cuando el protagonista sean los datos; ofrezca otros como alternativa.
- Agrupe los elementos en `FeatureGroup` con nombres claros y añada `LayerControl` al final.
- Muestre ocultas al inicio las capas pesadas o secundarias (`show=False`).
- Use `tooltip` para la información esencial y `popup` para el detalle.
- Represente magnitudes con tamaño (`CircleMarker`) o color (coropletas), no con iconos repetidos.
- Elija clases de color con sentido (cuantiles, cortes naturales o umbrales de negocio) e incluya siempre la **leyenda**.
- Normalice las claves de unión (códigos con ceros a la izquierda, mayúsculas, tildes) antes de una coropleta.
- Simplifique geometrías y limite el número de elementos para que el HTML sea ligero.
- Indique la fuente de los datos y respete la atribución de los mapas base.

---

## 19. Referencia rápida

```python
import folium; from folium import plugins; import branca.colormap as cm
m = folium.Map(location=[lat, lon], zoom_start=6, tiles="OpenStreetMap"|"Esri.WorldGrayCanvas"|None, control_scale=True)
folium.TileLayer("Esri.WorldImagery", name="Satélite").add_to(m)

folium.Marker([lat, lon], tooltip=, popup=folium.Popup(html, max_width=), icon=folium.Icon(color=, icon=, prefix="fa"))
folium.CircleMarker([lat, lon], radius=px, fill=True) / folium.Circle([lat, lon], radius=metros)
folium.PolyLine(puntos) / Polygon / Rectangle / DivIcon(html=) / CustomIcon("img.png")
folium.GeoJson(geo, style_function=, highlight_function=, tooltip=folium.GeoJsonTooltip([...]), popup=GeoJsonPopup)
folium.Choropleth(geo_data=, data=df, columns=[clave, valor], key_on="feature.properties.x", fill_color="Blues", bins=)
cm.LinearColormap([...], vmin=, vmax=, caption=) / cm.StepColormap(..., index=[...]) → escala(valor), escala.add_to(m)

folium.FeatureGroup(name=, show=).add_to(m) / folium.LayerControl(collapsed=False).add_to(m)   # al final
plugins.MarkerCluster / FastMarkerCluster / HeatMap / HeatMapWithTime / AntPath / Search(layer=, search_label=)
plugins.Fullscreen / MiniMap / MeasureControl / MousePosition / LocateControl / Draw(export=True) / Geocoder
m.get_root().html.add_child(folium.Element(html)) / m.fit_bounds([[s, o], [n, e]]) / m.save("mapa.html")
gdf.to_crs(epsg=4326).explore(column=, cmap=)  /  st_folium(m)  (streamlit-folium)
```

---

## 20. Ejercicios propuestos

1. Cree un mapa con los monumentos de una ciudad, cada uno con icono, *tooltip* y un *popup* con foto y descripción.
2. Represente estaciones meteorológicas con `CircleMarker` cuyo color dependa de la temperatura (escala lineal con leyenda).
3. Construya un mapa de coropletas de población por provincia a partir de un GeoJSON público y un CSV, con *tooltip* y clases por cuantiles.
4. Dibuje la ruta de un viaje con `PolyLine`, marque las paradas y añada el plugin de medición de distancias.
5. Amplíe el ejemplo completo con una capa temporal (`TimestampedGeoJson`) que muestre la apertura de tiendas por año y un `DualMap` que compare 2024 y 2025.

---

## 21. Referencias

- Documentación oficial: https://python-visualization.github.io/folium/latest/
- Guía de usuario: https://python-visualization.github.io/folium/latest/user_guide.html
- *Plugins*: https://python-visualization.github.io/folium/latest/user_guide/plugins.html
- Leaflet.js: https://leafletjs.com/
- Proveedores de teselas (xyzservices): https://xyzservices.readthedocs.io/
- Paletas ColorBrewer: https://colorbrewer2.org/
- `streamlit-folium`: https://folium.streamlit.app/
