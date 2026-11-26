# Manual de uso: python-pptx

> **Categoría:** Utilidades generales · **Utilidad:** Crear presentaciones PowerPoint
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Texto y formato](#5-texto-y-formato)
6. [Formas e imágenes](#6-formas-e-imágenes)
7. [Tablas](#7-tablas)
8. [Gráficos nativos](#8-gráficos-nativos)
9. [Leer y modificar presentaciones](#9-leer-y-modificar-presentaciones)
10. [Operaciones avanzadas](#10-operaciones-avanzadas)
11. [Ejemplo práctico completo](#11-ejemplo-práctico-completo)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**python-pptx** permite crear, leer y modificar presentaciones de PowerPoint (`.pptx`) desde Python, sin necesidad de tener PowerPoint instalado.

**Casos de uso habituales:**
- Generar automáticamente informes periódicos (ventas, KPIs, resultados).
- Crear presentaciones a partir de datos de Excel, bases de datos o APIs.
- Aplicar la plantilla corporativa de forma consistente.
- Extraer el texto de presentaciones existentes.
- Actualizar cifras y gráficos en presentaciones recurrentes.

---

## 2. Instalación e importación

```bash
pip install python-pptx
```

Se importa como `pptx`:

```python
from pptx import Presentation
from pptx.util import Inches, Cm, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR
from pptx.enum.shapes import MSO_SHAPE
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE, XL_LEGEND_POSITION
```

---

## 3. Conceptos clave

```
Presentation
 ├── slide_layouts      (diseños disponibles en la plantilla)
 ├── slides             (diapositivas)
 │    └── shapes        (formas: cuadros de texto, imágenes, tablas, gráficos…)
 │         └── text_frame → paragraphs → runs
 └── slide_master       (patrón de diapositivas)
```

| Concepto | Descripción |
|---|---|
| `Presentation` | La presentación completa. |
| `SlideLayout` | Diseño base (Título, Título y contenido, En blanco…). |
| `Slide` | Diapositiva concreta. |
| `Shape` | Cualquier elemento colocado en la diapositiva. |
| *Placeholder* | Marcador de posición definido por el diseño (título, contenido). |
| `TextFrame` | Contenedor de texto de una forma. |
| EMU | Unidad interna (914 400 EMU = 1 pulgada). Use `Cm`, `Inches` o `Pt`. |

### 3.1 Diseños de la plantilla por defecto

| Índice | Diseño |
|---|---|
| 0 | Título |
| 1 | Título y contenido |
| 2 | Encabezado de sección |
| 3 | Dos contenidos |
| 4 | Comparación |
| 5 | Solo título |
| 6 | En blanco |
| 7 | Contenido con título |
| 8 | Imagen con título |

> Los índices pueden cambiar en plantillas personalizadas. Consulte la sección 9.3 para listarlos.

---

## 4. Operaciones básicas

### 4.1 Crear una presentación

```python
prs = Presentation()

# Formato panorámico 16:9 (la plantilla por defecto es 4:3)
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)
```

### 4.2 Diapositiva de título

```python
diapo = prs.slides.add_slide(prs.slide_layouts[0])
diapo.shapes.title.text = "Resultados del tercer trimestre"
diapo.placeholders[1].text = "Dirección Comercial · Noviembre 2026"
```

### 4.3 Diapositiva con viñetas

```python
diapo = prs.slides.add_slide(prs.slide_layouts[1])
diapo.shapes.title.text = "Puntos clave"

cuerpo = diapo.placeholders[1].text_frame
cuerpo.text = "Las ventas crecieron un 12 %"            # Primer párrafo

p = cuerpo.add_paragraph()
p.text = "Destaca la región Norte"
p.level = 1                                              # Sangría (subnivel)

p = cuerpo.add_paragraph()
p.text = "El margen se mantiene estable"
```

### 4.4 Diapositiva en blanco con cuadro de texto

```python
diapo = prs.slides.add_slide(prs.slide_layouts[6])
cuadro = diapo.shapes.add_textbox(Cm(2), Cm(2), Cm(20), Cm(3))
cuadro.text_frame.text = "Texto libre en cualquier posición"
```

### 4.5 Guardar

```python
prs.save("presentacion.pptx")
```

---

## 5. Texto y formato

### 5.1 Formato de párrafo y fuente

```python
tf = cuadro.text_frame
tf.word_wrap = True

p = tf.paragraphs[0]
p.text = "Título personalizado"
p.alignment = PP_ALIGN.CENTER
p.space_after = Pt(12)
p.line_spacing = 1.2

fuente = p.font
fuente.name = "Calibri"
fuente.size = Pt(32)
fuente.bold = True
fuente.italic = False
fuente.color.rgb = RGBColor(0x1F, 0x4E, 0x78)
```

### 5.2 Varios formatos en un mismo párrafo

```python
p = tf.add_paragraph()
r1 = p.add_run()
r1.text = "Crecimiento: "
r2 = p.add_run()
r2.text = "+12 %"
r2.font.bold = True
r2.font.color.rgb = RGBColor(0x00, 0x8A, 0x3E)
```

### 5.3 Márgenes, anclaje y autoajuste

```python
from pptx.enum.text import MSO_AUTO_SIZE

tf.margin_left = tf.margin_right = Cm(0.3)
tf.vertical_anchor = MSO_ANCHOR.MIDDLE        # TOP, MIDDLE, BOTTOM
tf.auto_size = MSO_AUTO_SIZE.TEXT_TO_FIT_SHAPE
```

### 5.4 Hipervínculos

```python
r = p.add_run()
r.text = "Ver informe completo"
r.hyperlink.address = "https://intranet.miempresa.com/informe-q3"
```

### 5.5 Notas del orador

```python
diapo.notes_slide.notes_text_frame.text = "Recordar mencionar la campaña de Navidad."
```

---

## 6. Formas e imágenes

### 6.1 Insertar imágenes

```python
diapo.shapes.add_picture("logo.png", Cm(28), Cm(0.5), height=Cm(1.5))   # Mantiene proporción
diapo.shapes.add_picture("grafico.png", Cm(2), Cm(4), width=Cm(20))
```

### 6.2 Formas automáticas

```python
forma = diapo.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, Cm(2), Cm(5), Cm(8), Cm(4))

forma.fill.solid()
forma.fill.fore_color.rgb = RGBColor(0x1F, 0x4E, 0x78)
forma.line.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
forma.line.width = Pt(1.5)
forma.shadow.inherit = False                 # Quitar sombra

forma.text_frame.text = "12.500 €"
forma.text_frame.paragraphs[0].font.size = Pt(28)
forma.text_frame.paragraphs[0].font.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
forma.text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
```

Formas habituales: `RECTANGLE`, `ROUNDED_RECTANGLE`, `OVAL`, `RIGHT_ARROW`, `CHEVRON`, `PENTAGON`, `ISOSCELES_TRIANGLE`, `STAR_5_POINT`, `CLOUD`.

### 6.3 Líneas y conectores

```python
from pptx.enum.shapes import MSO_CONNECTOR

linea = diapo.shapes.add_connector(MSO_CONNECTOR.STRAIGHT, Cm(2), Cm(3), Cm(31), Cm(3))
linea.line.color.rgb = RGBColor(0x1F, 0x4E, 0x78)
linea.line.width = Pt(2)
```

### 6.4 Posición, tamaño y rotación

```python
forma.left, forma.top = Cm(5), Cm(6)
forma.width, forma.height = Cm(10), Cm(3)
forma.rotation = 15
```

### 6.5 Fondo de la diapositiva

```python
fondo = diapo.background.fill
fondo.solid()
fondo.fore_color.rgb = RGBColor(0xF2, 0xF2, 0xF2)
```

---

## 7. Tablas

### 7.1 Crear una tabla

```python
datos = [
    ["Región", "Q3", "Q4", "Variación"],
    ["Norte", "12.500 €", "13.800 €", "+10,4 %"],
    ["Sur", "9.800 €", "11.200 €", "+14,3 %"],
    ["Este", "14.300 €", "13.900 €", "-2,8 %"],
]

filas, columnas = len(datos), len(datos[0])
grafico_tabla = diapo.shapes.add_table(filas, columnas, Cm(2), Cm(4), Cm(24), Cm(6))
tabla = grafico_tabla.table

for i, fila in enumerate(datos):
    for j, valor in enumerate(fila):
        tabla.cell(i, j).text = valor
```

### 7.2 Formato de tabla

```python
tabla.columns[0].width = Cm(8)
for j in range(1, columnas):
    tabla.columns[j].width = Cm(5.33)

for j in range(columnas):
    celda = tabla.cell(0, j)
    celda.fill.solid()
    celda.fill.fore_color.rgb = RGBColor(0x1F, 0x4E, 0x78)
    p = celda.text_frame.paragraphs[0]
    p.font.bold = True
    p.font.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
    p.alignment = PP_ALIGN.CENTER

for i in range(1, filas):
    for j in range(1, columnas):
        tabla.cell(i, j).text_frame.paragraphs[0].alignment = PP_ALIGN.RIGHT
    tabla.cell(i, 0).vertical_anchor = MSO_ANCHOR.MIDDLE
```

### 7.3 Combinar celdas

```python
tabla.cell(0, 0).merge(tabla.cell(0, 1))
```

### 7.4 Desactivar bandas y estilos de cabecera

```python
tabla.first_row = True
tabla.horz_banding = True
```

---

## 8. Gráficos nativos

Los gráficos nativos son **editables** en PowerPoint (el usuario puede modificar sus datos).

### 8.1 Gráfico de barras

```python
datos_grafico = CategoryChartData()
datos_grafico.categories = ["Norte", "Sur", "Este", "Oeste"]
datos_grafico.add_series("Q3", (12500, 9800, 14300, 8700))
datos_grafico.add_series("Q4", (13800, 11200, 13900, 10100))

marco = diapo.shapes.add_chart(
    XL_CHART_TYPE.COLUMN_CLUSTERED, Cm(2), Cm(4), Cm(22), Cm(12), datos_grafico
)
grafico = marco.chart
grafico.has_legend = True
grafico.legend.position = XL_LEGEND_POSITION.BOTTOM
grafico.legend.include_in_layout = False
grafico.has_title = True
grafico.chart_title.text_frame.text = "Ventas por región"
```

### 8.2 Tipos de gráfico más usados

| Constante | Tipo |
|---|---|
| `COLUMN_CLUSTERED` | Columnas agrupadas |
| `COLUMN_STACKED` | Columnas apiladas |
| `BAR_CLUSTERED` | Barras horizontales |
| `LINE_MARKERS` | Líneas con marcadores |
| `PIE` | Circular |
| `DOUGHNUT` | Anillo |
| `AREA` | Área |
| `XY_SCATTER` | Dispersión (usa `XyChartData`) |

### 8.3 Personalizar el gráfico

```python
from pptx.enum.chart import XL_LABEL_POSITION

# Colores de las series
colores = [RGBColor(0x1F, 0x4E, 0x78), RGBColor(0x9D, 0xC3, 0xE6)]
for serie, color in zip(grafico.series, colores):
    serie.format.fill.solid()
    serie.format.fill.fore_color.rgb = color

# Etiquetas de datos
plot = grafico.plots[0]
plot.has_data_labels = True
plot.data_labels.number_format = '#,##0 "€"'
plot.data_labels.number_format_is_linked = False
plot.data_labels.font.size = Pt(9)
plot.gap_width = 80

# Ejes
eje_y = grafico.value_axis
eje_y.has_major_gridlines = True
eje_y.tick_labels.font.size = Pt(10)
eje_y.maximum_scale = 16000
grafico.category_axis.tick_labels.font.size = Pt(11)
```

### 8.4 Gráfico circular con porcentajes

```python
d = CategoryChartData()
d.categories = ["Online", "Tienda", "Distribuidores"]
d.add_series("Canal", (0.45, 0.35, 0.20))

g = diapo.shapes.add_chart(XL_CHART_TYPE.PIE, Cm(2), Cm(4), Cm(14), Cm(12), d).chart
g.plots[0].has_data_labels = True
g.plots[0].data_labels.number_format = "0%"
g.plots[0].data_labels.number_format_is_linked = False
g.plots[0].data_labels.position = XL_LABEL_POSITION.OUTSIDE_END
```

### 8.5 Actualizar los datos de un gráfico existente

```python
nuevos = CategoryChartData()
nuevos.categories = ["Norte", "Sur", "Este", "Oeste"]
nuevos.add_series("Q1 2027", (14000, 11500, 15000, 10500))
grafico.replace_data(nuevos)
```

### 8.6 Gráficos de Matplotlib como imagen

```python
import io
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(10, 5), dpi=200)
ax.plot(["Ene", "Feb", "Mar"], [10, 14, 12], marker="o")
buffer = io.BytesIO()
fig.savefig(buffer, format="png", bbox_inches="tight", transparent=True)
plt.close(fig)
buffer.seek(0)
diapo.shapes.add_picture(buffer, Cm(2), Cm(4), width=Cm(22))
```

---

## 9. Leer y modificar presentaciones

### 9.1 Abrir una presentación

```python
prs = Presentation("existente.pptx")
print(len(prs.slides), "diapositivas")
```

### 9.2 Extraer todo el texto

```python
for num, diapo in enumerate(prs.slides, start=1):
    print(f"--- Diapositiva {num} ---")
    for forma in diapo.shapes:
        if forma.has_text_frame:
            for p in forma.text_frame.paragraphs:
                texto = "".join(r.text for r in p.runs)
                if texto.strip():
                    print("  " * p.level + texto)
        if forma.has_table:
            for fila in forma.table.rows:
                print(" | ".join(c.text for c in fila.cells))
    if diapo.has_notes_slide:
        print("Notas:", diapo.notes_slide.notes_text_frame.text)
```

### 9.3 Inspeccionar diseños y marcadores de una plantilla

```python
prs = Presentation("plantilla_corporativa.pptx")
for i, layout in enumerate(prs.slide_layouts):
    print(i, layout.name)
    for ph in layout.placeholders:
        print(f"    idx={ph.placeholder_format.idx}  tipo={ph.placeholder_format.type}  nombre={ph.name}")
```

### 9.4 Reemplazar texto conservando el formato

```python
def reemplazar(prs, reemplazos: dict):
    for diapo in prs.slides:
        for forma in diapo.shapes:
            marcos = []
            if forma.has_text_frame:
                marcos.append(forma.text_frame)
            if forma.has_table:
                marcos += [c.text_frame for f in forma.table.rows for c in f.cells]
            for tf in marcos:
                for p in tf.paragraphs:
                    for r in p.runs:
                        for clave, valor in reemplazos.items():
                            if clave in r.text:
                                r.text = r.text.replace(clave, str(valor))

reemplazar(prs, {"{{TRIMESTRE}}": "Q4 2026", "{{VENTAS}}": "48.900 €"})
```

### 9.5 Eliminar una diapositiva

```python
def eliminar_diapositiva(prs, indice: int):
    lista = prs.slides._sldIdLst
    sld_id = lista[indice]
    prs.part.drop_rel(sld_id.rId)
    lista.remove(sld_id)
```

### 9.6 Reordenar diapositivas

```python
def mover_diapositiva(prs, origen: int, destino: int):
    lista = prs.slides._sldIdLst
    elemento = lista[origen]
    lista.remove(elemento)
    lista.insert(destino, elemento)
```

---

## 10. Operaciones avanzadas

### 10.1 Usar una plantilla corporativa

```python
prs = Presentation("plantilla_corporativa.pptx")      # Hereda colores, fuentes y patrones
diapo = prs.slides.add_slide(prs.slide_layouts[1])
```

> Elimine de la plantilla las diapositivas de ejemplo y conserve solo los diseños del patrón.

### 10.2 Rellenar marcadores por índice

```python
diapo = prs.slides.add_slide(prs.slide_layouts[8])     # Imagen con título
diapo.placeholders[0].text = "Nuevo producto"
diapo.placeholders[1].insert_picture("producto.jpg")   # Marcador de imagen
diapo.placeholders[2].text = "Disponible a partir de enero"
```

### 10.3 Funciones auxiliares reutilizables

```python
def tarjeta_kpi(diapo, x, y, titulo, valor, variacion, color=RGBColor(0x1F, 0x4E, 0x78)):
    caja = diapo.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, x, y, Cm(7), Cm(4))
    caja.fill.solid()
    caja.fill.fore_color.rgb = color
    caja.line.fill.background()
    caja.shadow.inherit = False

    tf = caja.text_frame
    tf.vertical_anchor = MSO_ANCHOR.MIDDLE
    lineas = [(titulo, 12, False), (valor, 28, True), (variacion, 12, False)]
    for i, (texto, tam, negrita) in enumerate(lineas):
        p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
        p.text = texto
        p.alignment = PP_ALIGN.CENTER
        p.font.size = Pt(tam)
        p.font.bold = negrita
        p.font.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
```

### 10.4 Convertir a PDF o imágenes

python-pptx no exporta a PDF. Opción con LibreOffice:

```bash
soffice --headless --convert-to pdf presentacion.pptx
```

### 10.5 Guardar en memoria

```python
import io
buffer = io.BytesIO()
prs.save(buffer)
```

---

## 11. Ejemplo práctico completo

Generación automática de una presentación de resultados (16:9) a partir de un DataFrame: portada, KPIs, gráfico nativo, tabla y conclusiones.

```python
import pandas as pd
from pptx import Presentation
from pptx.chart.data import CategoryChartData
from pptx.dml.color import RGBColor
from pptx.enum.chart import XL_CHART_TYPE, XL_LEGEND_POSITION
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import MSO_ANCHOR, PP_ALIGN
from pptx.util import Cm, Pt

AZUL = RGBColor(0x1F, 0x4E, 0x78)
AZUL_CLARO = RGBColor(0x9D, 0xC3, 0xE6)
GRIS = RGBColor(0x59, 0x59, 0x59)
BLANCO = RGBColor(0xFF, 0xFF, 0xFF)
VERDE = RGBColor(0x00, 0x8A, 0x3E)
ROJO = RGBColor(0xC0, 0x00, 0x00)


def euros(v: float) -> str:
    return f"{v:,.0f} €".replace(",", ".")


def texto(diapo, x, y, w, h, contenido, tam=18, negrita=False, color=GRIS, alinear=PP_ALIGN.LEFT):
    cuadro = diapo.shapes.add_textbox(x, y, w, h)
    tf = cuadro.text_frame
    tf.word_wrap = True
    p = tf.paragraphs[0]
    p.text = contenido
    p.alignment = alinear
    p.font.size, p.font.bold, p.font.color.rgb = Pt(tam), negrita, color
    return cuadro


def cabecera(prs, diapo, titulo):
    barra = diapo.shapes.add_shape(MSO_SHAPE.RECTANGLE, 0, 0, prs.slide_width, Cm(2.4))
    barra.fill.solid()
    barra.fill.fore_color.rgb = AZUL
    barra.line.fill.background()
    texto(diapo, Cm(1.2), Cm(0.5), Cm(30), Cm(1.5), titulo, 26, True, BLANCO)


def tarjeta(diapo, x, y, titulo, valor, variacion):
    caja = diapo.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, x, y, Cm(9.5), Cm(4.5))
    caja.fill.solid()
    caja.fill.fore_color.rgb = RGBColor(0xF2, 0xF2, 0xF2)
    caja.line.fill.background()
    caja.shadow.inherit = False
    tf = caja.text_frame
    tf.vertical_anchor = MSO_ANCHOR.MIDDLE
    for i, (t, tam, neg, col) in enumerate([
        (titulo, 13, False, GRIS),
        (valor, 30, True, AZUL),
        (variacion, 13, True, VERDE if not variacion.startswith("-") else ROJO),
    ]):
        p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
        p.text, p.alignment = t, PP_ALIGN.CENTER
        p.font.size, p.font.bold, p.font.color.rgb = Pt(tam), neg, col


def generar(df: pd.DataFrame, salida: str):
    prs = Presentation()
    prs.slide_width, prs.slide_height = Cm(33.867), Cm(19.05)       # 16:9
    blanco = prs.slide_layouts[6]

    # 1. Portada
    d = prs.slides.add_slide(blanco)
    d.background.fill.solid()
    d.background.fill.fore_color.rgb = AZUL
    texto(d, Cm(2), Cm(6.5), Cm(30), Cm(3), "Resultados comerciales Q4 2026", 44, True, BLANCO)
    texto(d, Cm(2), Cm(9.8), Cm(30), Cm(1.5), "Dirección Comercial · Noviembre 2026", 20, False, AZUL_CLARO)

    # 2. KPIs
    q3, q4 = df["Q3"].sum(), df["Q4"].sum()
    var = (q4 - q3) / q3
    mejor = df.assign(v=(df.Q4 - df.Q3) / df.Q3).sort_values("v", ascending=False).iloc[0]

    d = prs.slides.add_slide(blanco)
    cabecera(prs, d, "Indicadores clave")
    tarjeta(d, Cm(1.5), Cm(4.5), "Ventas Q4", euros(q4), f"{var:+.1%} vs. Q3".replace(".", ","))
    tarjeta(d, Cm(12.2), Cm(4.5), "Ventas Q3", euros(q3), "Trimestre anterior")
    tarjeta(d, Cm(22.9), Cm(4.5), "Mejor región", mejor["Region"],
            f"{mejor['v']:+.1%}".replace(".", ","))
    d.notes_slide.notes_text_frame.text = "Destacar el crecimiento global y la región líder."

    # 3. Gráfico
    d = prs.slides.add_slide(blanco)
    cabecera(prs, d, "Ventas por región")
    datos = CategoryChartData()
    datos.categories = df["Region"].tolist()
    datos.add_series("Q3", df["Q3"].tolist())
    datos.add_series("Q4", df["Q4"].tolist())
    g = d.shapes.add_chart(XL_CHART_TYPE.COLUMN_CLUSTERED, Cm(1.5), Cm(3.2),
                           Cm(31), Cm(15), datos).chart
    g.has_legend = True
    g.legend.position = XL_LEGEND_POSITION.BOTTOM
    g.legend.include_in_layout = False
    for serie, color in zip(g.series, (AZUL_CLARO, AZUL)):
        serie.format.fill.solid()
        serie.format.fill.fore_color.rgb = color
    plot = g.plots[0]
    plot.gap_width = 70
    plot.has_data_labels = True
    plot.data_labels.number_format = '#,##0" €"'
    plot.data_labels.number_format_is_linked = False
    plot.data_labels.font.size = Pt(10)

    # 4. Tabla
    d = prs.slides.add_slide(blanco)
    cabecera(prs, d, "Detalle por región")
    filas = len(df) + 2
    t = d.shapes.add_table(filas, 4, Cm(3), Cm(4), Cm(28), Cm(1.2) * filas).table
    cab = ["Región", "Q3", "Q4", "Variación"]
    for j, v in enumerate(cab):
        c = t.cell(0, j)
        c.text = v
        c.fill.solid()
        c.fill.fore_color.rgb = AZUL
        c.text_frame.paragraphs[0].font.color.rgb = BLANCO
        c.text_frame.paragraphs[0].font.bold = True
    for i, fila in enumerate(df.itertuples(), start=1):
        variacion = (fila.Q4 - fila.Q3) / fila.Q3
        valores = [fila.Region, euros(fila.Q3), euros(fila.Q4), f"{variacion:+.1%}".replace(".", ",")]
        for j, v in enumerate(valores):
            t.cell(i, j).text = v
            p = t.cell(i, j).text_frame.paragraphs[0]
            p.font.size = Pt(14)
            if j:
                p.alignment = PP_ALIGN.RIGHT
            if j == 3:
                p.font.color.rgb = VERDE if variacion >= 0 else ROJO
    total = ["TOTAL", euros(q3), euros(q4), f"{var:+.1%}".replace(".", ",")]
    for j, v in enumerate(total):
        c = t.cell(filas - 1, j)
        c.text = v
        c.text_frame.paragraphs[0].font.bold = True
        if j:
            c.text_frame.paragraphs[0].alignment = PP_ALIGN.RIGHT

    # 5. Conclusiones
    d = prs.slides.add_slide(blanco)
    cabecera(prs, d, "Conclusiones y próximos pasos")
    cuadro = d.shapes.add_textbox(Cm(2), Cm(4), Cm(30), Cm(13))
    tf = cuadro.text_frame
    tf.word_wrap = True
    puntos = [
        (f"Crecimiento global del {var:.1%} respecto al trimestre anterior.".replace(".", ",", 1), 0),
        (f"{mejor['Region']} lidera el crecimiento.", 0),
        ("Reforzar la campaña de Navidad en las regiones con menor crecimiento.", 0),
        ("Revisar precios y promociones en la región Este.", 1),
        ("Presentar el plan comercial 2027 antes del 15 de diciembre.", 0),
    ]
    for i, (t_, nivel) in enumerate(puntos):
        p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
        p.text = ("• " if nivel == 0 else "– ") + t_
        p.level = nivel
        p.space_after = Pt(14)
        p.font.size = Pt(22 if nivel == 0 else 18)
        p.font.color.rgb = GRIS

    prs.save(salida)


if __name__ == "__main__":
    ventas = pd.DataFrame({
        "Region": ["Norte", "Sur", "Este", "Oeste"],
        "Q3": [12500, 9800, 14300, 8700],
        "Q4": [13800, 11200, 13900, 10100],
    })
    generar(ventas, "resultados_q4.pptx")
    print("Presentación generada: resultados_q4.pptx")
```

---

## 12. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'pptx'` | Paquete no instalado. | `pip install python-pptx`. |
| `KeyError` al acceder a `placeholders[1]` | El diseño no tiene ese marcador. | Inspeccione los marcadores (sección 9.3). |
| `IndexError` en `slide_layouts` | La plantilla tiene menos diseños. | Liste los diseños disponibles. |
| Diapositivas en formato 4:3 | Plantilla por defecto. | Ajuste `slide_width` y `slide_height`. |
| El texto se sale de la forma | Texto demasiado largo. | Use `word_wrap` y reduzca el tamaño o el contenido. |
| `AttributeError: 'Picture' object has no attribute 'text_frame'` | No todas las formas tienen texto. | Compruebe `shape.has_text_frame`. |
| No se puede abrir el archivo `.ppt` | Formato antiguo. | Convierta a `.pptx`. |

---

## 13. Buenas prácticas

- Use la **plantilla corporativa** como base en lugar de dar formato manual a cada elemento.
- Defina colores, fuentes y tamaños como constantes.
- Cree funciones auxiliares (cabecera, tarjeta KPI, tabla) para mantener la coherencia.
- Prefiera **gráficos nativos** (editables) a imágenes cuando el destinatario necesite modificarlos.
- Limite el texto por diapositiva: una idea principal y pocas viñetas.
- Incluya notas del orador con los mensajes clave.
- Separe la obtención de datos de la generación de la presentación.

---

## 14. Referencia rápida

```python
prs = Presentation() | Presentation("plantilla.pptx")
prs.slide_width = Cm(33.867); prs.slide_height = Cm(19.05)   # 16:9
d = prs.slides.add_slide(prs.slide_layouts[i])
d.shapes.title.text; d.placeholders[idx].text
d.shapes.add_textbox(x, y, w, h).text_frame
tf.paragraphs[0].text / .font.size / .alignment / tf.add_paragraph() / p.level
d.shapes.add_picture(ruta, x, y, width=...)
d.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, x, y, w, h)
d.shapes.add_table(filas, cols, x, y, w, h).table.cell(i, j).text
d.shapes.add_chart(XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, w, h, CategoryChartData())
d.notes_slide.notes_text_frame.text = "..."
prs.save("salida.pptx")
```

---

## 15. Ejercicios propuestos

1. Genere una presentación con una diapositiva por producto a partir de un Excel (imagen, nombre, precio y descripción).
2. Lea un `.pptx` existente y exporte todo su texto y sus notas a un archivo Markdown.
3. Cree una presentación mensual de KPIs que actualice automáticamente los gráficos con `replace_data`.
4. Construya un organigrama sencillo con formas y conectores a partir de una lista de empleados y responsables.

---

## 16. Referencias

- Documentación oficial: https://python-pptx.readthedocs.io/
