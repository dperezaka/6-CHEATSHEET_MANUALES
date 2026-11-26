# Manual de uso: python-docx

> **Categoría:** Utilidades generales · **Utilidad:** Crear y modificar documentos Word
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Formato de texto y párrafos](#5-formato-de-texto-y-párrafos)
6. [Estilos](#6-estilos)
7. [Tablas](#7-tablas)
8. [Imágenes, saltos y secciones](#8-imágenes-saltos-y-secciones)
9. [Leer y modificar documentos existentes](#9-leer-y-modificar-documentos-existentes)
10. [Operaciones avanzadas](#10-operaciones-avanzadas)
11. [Ejemplo práctico completo](#11-ejemplo-práctico-completo)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**python-docx** permite crear, leer y modificar documentos de Microsoft Word (`.docx`) desde Python, sin necesidad de tener Word instalado.

**Casos de uso habituales:**
- Generar contratos, cartas, informes y certificados a partir de datos.
- Rellenar plantillas corporativas automáticamente.
- Extraer texto y tablas de documentos existentes.
- Modificar documentos por lotes (reemplazar textos, actualizar datos).

> Solo admite el formato `.docx` (Word 2007 en adelante), no el antiguo `.doc`.

---

## 2. Instalación e importación

```bash
pip install python-docx
```

El paquete se instala como `python-docx`, pero se importa como `docx`:

```python
from docx import Document
from docx.shared import Pt, Cm, Inches, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH, WD_BREAK
from docx.enum.table import WD_TABLE_ALIGNMENT
from docx.enum.section import WD_ORIENT
```

---

## 3. Conceptos clave

Un documento Word se organiza en una jerarquía:

```
Document
 ├── Sections        (configuración de página, encabezados, pies)
 ├── Paragraphs      (párrafos)
 │    └── Runs       (fragmentos de texto con el mismo formato)
 └── Tables
      └── Rows → Cells → Paragraphs → Runs
```

| Elemento | Descripción |
|---|---|
| `Document` | El documento completo. |
| `Paragraph` | Un párrafo (incluye títulos y elementos de lista). |
| `Run` | Fragmento de texto con formato uniforme. Un párrafo puede tener varios. |
| `Style` | Estilo con nombre (`Normal`, `Heading 1`, `List Bullet`…). |
| `Section` | Parte del documento con su propia configuración de página. |

**Unidades:** `Pt` (puntos), `Cm` (centímetros), `Inches` (pulgadas), `Mm` (milímetros).

---

## 4. Operaciones básicas

### 4.1 Crear un documento

```python
doc = Document()

doc.add_heading("Informe de actividad", level=0)      # Título principal
doc.add_heading("1. Introducción", level=1)
doc.add_paragraph("Este documento resume la actividad del mes de noviembre.")

doc.save("informe.docx")
```

### 4.2 Niveles de título

```python
doc.add_heading("Título del documento", level=0)   # Estilo "Title"
doc.add_heading("Capítulo", level=1)               # "Heading 1"
doc.add_heading("Apartado", level=2)               # "Heading 2"
doc.add_heading("Subapartado", level=3)
```

> Usar los niveles de título correctamente permite generar un índice automático en Word.

### 4.3 Listas

```python
doc.add_paragraph("Primer punto", style="List Bullet")
doc.add_paragraph("Segundo punto", style="List Bullet")
doc.add_paragraph("Subpunto", style="List Bullet 2")

doc.add_paragraph("Paso uno", style="List Number")
doc.add_paragraph("Paso dos", style="List Number")
```

### 4.4 Texto con varios formatos en un párrafo

```python
p = doc.add_paragraph("El importe total asciende a ")
p.add_run("1.250,00 €").bold = True
p.add_run(", con vencimiento el ")
fecha = p.add_run("15/12/2026")
fecha.italic = True
p.add_run(".")
```

### 4.5 Salto de página

```python
doc.add_page_break()
```

---

## 5. Formato de texto y párrafos

### 5.1 Formato de caracteres (`Run`)

```python
run = p.add_run("Texto destacado")
run.bold = True
run.italic = True
run.underline = True
run.font.name = "Calibri"
run.font.size = Pt(12)
run.font.color.rgb = RGBColor(0x1F, 0x4E, 0x78)
run.font.highlight_color = 7          # WD_COLOR_INDEX.YELLOW
run.font.strike = True
run.font.superscript = True
run.font.all_caps = True
```

### 5.2 Formato de párrafo

```python
p = doc.add_paragraph("Párrafo con formato.")
fmt = p.paragraph_format

p.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY        # LEFT, CENTER, RIGHT, JUSTIFY
fmt.space_before = Pt(6)
fmt.space_after = Pt(12)
fmt.line_spacing = 1.15
fmt.left_indent = Cm(1)
fmt.right_indent = Cm(1)
fmt.first_line_indent = Cm(0.75)
fmt.keep_with_next = True                       # No separar del siguiente párrafo
fmt.page_break_before = False
```

### 5.3 Salto de línea dentro de un párrafo

```python
run = p.add_run("Primera línea")
run.add_break()                                 # Salto de línea
p.add_run("Segunda línea")
run.add_break(WD_BREAK.PAGE)                    # Salto de página dentro del run
```

### 5.4 Tabulaciones

```python
from docx.enum.text import WD_TAB_ALIGNMENT, WD_TAB_LEADER

p = doc.add_paragraph()
p.paragraph_format.tab_stops.add_tab_stop(Cm(15), WD_TAB_ALIGNMENT.RIGHT, WD_TAB_LEADER.DOTS)
p.add_run("Capítulo 1\t5")
```

---

## 6. Estilos

### 6.1 Modificar el estilo base del documento

```python
estilo = doc.styles["Normal"]
estilo.font.name = "Calibri"
estilo.font.size = Pt(11)
estilo.paragraph_format.space_after = Pt(6)
estilo.paragraph_format.line_spacing = 1.15
```

### 6.2 Personalizar títulos

```python
h1 = doc.styles["Heading 1"]
h1.font.name = "Calibri"
h1.font.size = Pt(16)
h1.font.color.rgb = RGBColor(0x1F, 0x4E, 0x78)
```

### 6.3 Crear un estilo nuevo

```python
from docx.enum.style import WD_STYLE_TYPE

nota = doc.styles.add_style("Nota", WD_STYLE_TYPE.PARAGRAPH)
nota.base_style = doc.styles["Normal"]
nota.font.size = Pt(9)
nota.font.italic = True
nota.font.color.rgb = RGBColor(0x59, 0x59, 0x59)

doc.add_paragraph("Texto de nota al margen.", style="Nota")
```

### 6.4 Listar estilos disponibles

```python
for s in doc.styles:
    print(s.type, s.name)
```

> La forma más eficaz de conseguir un diseño corporativo es **partir de una plantilla `.docx`** que ya tenga los estilos definidos: `Document("plantilla.docx")`.

---

## 7. Tablas

### 7.1 Crear una tabla

```python
datos = [
    ("Producto", "Unidades", "Importe"),
    ("Teclado", "2", "59,80 €"),
    ("Monitor", "1", "189,00 €"),
]

tabla = doc.add_table(rows=1, cols=3)
tabla.style = "Light Grid Accent 1"
tabla.alignment = WD_TABLE_ALIGNMENT.CENTER

cab = tabla.rows[0].cells
for i, texto in enumerate(datos[0]):
    cab[i].text = texto

for fila in datos[1:]:
    celdas = tabla.add_row().cells
    for i, valor in enumerate(fila):
        celdas[i].text = valor
```

Estilos de tabla habituales: `Table Grid`, `Light Shading Accent 1`, `Light List Accent 1`, `Light Grid Accent 1`, `Medium Shading 1 Accent 1`, `Light Grid`.

### 7.2 Acceder a celdas

```python
tabla.cell(0, 1).text
tabla.rows[1].cells[2].text = "60,00 €"
len(tabla.rows), len(tabla.columns)
```

### 7.3 Formato de celdas

```python
from docx.enum.table import WD_CELL_VERTICAL_ALIGNMENT

celda = tabla.cell(0, 0)
celda.vertical_alignment = WD_CELL_VERTICAL_ALIGNMENT.CENTER

parrafo = celda.paragraphs[0]
parrafo.alignment = WD_ALIGN_PARAGRAPH.RIGHT
parrafo.runs[0].font.bold = True
```

### 7.4 Ancho de columnas

```python
anchos = [Cm(9), Cm(3), Cm(4)]
tabla.autofit = False
for fila in tabla.rows:
    for celda, ancho in zip(fila.cells, anchos):
        celda.width = ancho
```

### 7.5 Combinar celdas

```python
a = tabla.cell(0, 0)
b = tabla.cell(0, 2)
combinada = a.merge(b)
combinada.text = "Resumen de ventas"
```

### 7.6 Color de fondo de una celda

python-docx no tiene una propiedad directa; se modifica el XML:

```python
from docx.oxml import OxmlElement
from docx.oxml.ns import qn

def color_celda(celda, hex_color: str):
    tcPr = celda._tc.get_or_add_tcPr()
    sombreado = OxmlElement("w:shd")
    sombreado.set(qn("w:val"), "clear")
    sombreado.set(qn("w:color"), "auto")
    sombreado.set(qn("w:fill"), hex_color)
    tcPr.append(sombreado)

for celda in tabla.rows[0].cells:
    color_celda(celda, "1F4E78")
```

### 7.7 Repetir la fila de cabecera en cada página

```python
def repetir_cabecera(fila):
    trPr = fila._tr.get_or_add_trPr()
    elemento = OxmlElement("w:tblHeader")
    elemento.set(qn("w:val"), "true")
    trPr.append(elemento)

repetir_cabecera(tabla.rows[0])
```

---

## 8. Imágenes, saltos y secciones

### 8.1 Insertar imágenes

```python
doc.add_picture("grafico.png", width=Cm(15))         # Mantiene la proporción
doc.paragraphs[-1].alignment = WD_ALIGN_PARAGRAPH.CENTER

# Imagen dentro de un párrafo (en línea con texto)
p = doc.add_paragraph()
p.add_run().add_picture("logo.png", height=Cm(1.5))
```

### 8.2 Insertar un gráfico de Matplotlib

```python
import io
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(7, 3.5))
ax.plot(["Ene", "Feb", "Mar"], [10, 14, 12], marker="o")
buffer = io.BytesIO()
fig.savefig(buffer, format="png", dpi=200, bbox_inches="tight")
plt.close(fig)
buffer.seek(0)

doc.add_picture(buffer, width=Cm(15))
```

### 8.3 Configuración de página

```python
seccion = doc.sections[0]
seccion.page_height = Cm(29.7)        # A4
seccion.page_width = Cm(21)
seccion.top_margin = Cm(2.5)
seccion.bottom_margin = Cm(2.5)
seccion.left_margin = Cm(2.5)
seccion.right_margin = Cm(2.5)
```

### 8.4 Orientación horizontal y nuevas secciones

```python
from docx.enum.section import WD_SECTION

nueva = doc.add_section(WD_SECTION.NEW_PAGE)
nueva.orientation = WD_ORIENT.LANDSCAPE
nueva.page_width, nueva.page_height = nueva.page_height, nueva.page_width
```

### 8.5 Encabezado y pie de página

```python
seccion = doc.sections[0]

encabezado = seccion.header
encabezado.paragraphs[0].text = "MI EMPRESA, S.L. — Informe confidencial"
encabezado.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.RIGHT

pie = seccion.footer
pie.paragraphs[0].text = "C/ Mayor 1, 28013 Madrid · www.miempresa.com"
pie.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.CENTER

seccion.different_first_page_header_footer = True     # Portada sin encabezado
```

---

## 9. Leer y modificar documentos existentes

### 9.1 Leer el texto

```python
doc = Document("contrato.docx")

for p in doc.paragraphs:
    print(f"[{p.style.name}] {p.text}")

texto_completo = "\n".join(p.text for p in doc.paragraphs)
```

### 9.2 Leer tablas

```python
for t in doc.tables:
    for fila in t.rows:
        print([celda.text for celda in fila.cells])
```

Convertir una tabla en DataFrame:

```python
import pandas as pd

t = doc.tables[0]
filas = [[c.text for c in f.cells] for f in t.rows]
df = pd.DataFrame(filas[1:], columns=filas[0])
```

### 9.3 Obtener títulos (estructura del documento)

```python
for p in doc.paragraphs:
    if p.style.name.startswith("Heading"):
        nivel = p.style.name.replace("Heading ", "")
        print("  " * (int(nivel) - 1) + p.text)
```

### 9.4 Propiedades del documento

```python
props = doc.core_properties
props.title = "Contrato de servicios"
props.author = "Departamento Legal"
props.subject = "Cliente ABC"
props.keywords = "contrato, servicios, 2026"
print(props.created, props.last_modified_by)
```

### 9.5 Eliminar un párrafo

```python
def eliminar_parrafo(parrafo):
    elemento = parrafo._element
    elemento.getparent().remove(elemento)
```

---

## 10. Operaciones avanzadas

### 10.1 Rellenar plantillas con marcadores

Cree en Word una plantilla con marcadores como `{{NOMBRE}}`, `{{FECHA}}`, `{{IMPORTE}}`.

```python
def reemplazar_en_parrafo(parrafo, reemplazos: dict):
    texto = parrafo.text
    if not any(clave in texto for clave in reemplazos):
        return
    for clave, valor in reemplazos.items():
        texto = texto.replace(clave, str(valor))
    # Conserva el formato del primer run y elimina el resto
    for run in parrafo.runs[1:]:
        run._element.getparent().remove(run._element)
    if parrafo.runs:
        parrafo.runs[0].text = texto
    else:
        parrafo.add_run(texto)


def rellenar_plantilla(doc, reemplazos: dict):
    for p in doc.paragraphs:
        reemplazar_en_parrafo(p, reemplazos)
    for t in doc.tables:
        for fila in t.rows:
            for celda in fila.cells:
                for p in celda.paragraphs:
                    reemplazar_en_parrafo(p, reemplazos)
    for s in doc.sections:
        for p in s.header.paragraphs + s.footer.paragraphs:
            reemplazar_en_parrafo(p, reemplazos)
```

> Word puede dividir un marcador como `{{NOMBRE}}` en varios *runs* si se editó a trozos. La función anterior lo resuelve uniendo el texto del párrafo, a costa de aplicar a todo el párrafo el formato del primer fragmento. Para plantillas complejas, considere la librería **docxtpl** (plantillas Jinja2 sobre Word).

### 10.2 Hipervínculos

```python
def agregar_hipervinculo(parrafo, url: str, texto: str):
    r_id = parrafo.part.relate_to(
        url, "http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink",
        is_external=True,
    )
    hiper = OxmlElement("w:hyperlink")
    hiper.set(qn("r:id"), r_id)
    run = OxmlElement("w:r")
    rPr = OxmlElement("w:rPr")
    estilo = OxmlElement("w:rStyle")
    estilo.set(qn("w:val"), "Hyperlink")
    rPr.append(estilo)
    run.append(rPr)
    t = OxmlElement("w:t")
    t.text = texto
    run.append(t)
    hiper.append(run)
    parrafo._p.append(hiper)

p = doc.add_paragraph("Más información en ")
agregar_hipervinculo(p, "https://www.python.org", "python.org")
```

### 10.3 Numeración de páginas en el pie

```python
def campo_numero_pagina(parrafo):
    run = parrafo.add_run()
    for tipo, texto in (("begin", None), (None, "PAGE"), ("end", None)):
        if tipo:
            fld = OxmlElement("w:fldChar")
            fld.set(qn("w:fldCharType"), tipo)
            run._r.append(fld)
        else:
            instr = OxmlElement("w:instrText")
            instr.set(qn("xml:space"), "preserve")
            instr.text = texto
            run._r.append(instr)

p = doc.sections[0].footer.add_paragraph("Página ")
p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
campo_numero_pagina(p)
```

### 10.4 Convertir a PDF

python-docx no exporta a PDF. Opciones:

```bash
# Con LibreOffice instalado (multiplataforma)
soffice --headless --convert-to pdf informe.docx
```

```python
# Con Word instalado (Windows/macOS)
# pip install docx2pdf
from docx2pdf import convert
convert("informe.docx", "informe.pdf")
```

### 10.5 Guardar en memoria (descarga web)

```python
import io
buffer = io.BytesIO()
doc.save(buffer)
datos = buffer.getvalue()
```

---

## 11. Ejemplo práctico completo

Generación masiva de cartas personalizadas a partir de un CSV, con encabezado, tabla de detalle y firma.

```python
import csv
from datetime import date
from pathlib import Path

from docx import Document
from docx.enum.table import WD_TABLE_ALIGNMENT
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml import OxmlElement
from docx.oxml.ns import qn
from docx.shared import Cm, Pt, RGBColor

AZUL = RGBColor(0x1F, 0x4E, 0x78)
MESES = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio",
         "agosto", "septiembre", "octubre", "noviembre", "diciembre"]


def fecha_larga(d: date) -> str:
    return f"{d.day} de {MESES[d.month - 1]} de {d.year}"


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def fondo_celda(celda, color: str):
    shd = OxmlElement("w:shd")
    shd.set(qn("w:val"), "clear")
    shd.set(qn("w:fill"), color)
    celda._tc.get_or_add_tcPr().append(shd)


def configurar(doc: Document):
    normal = doc.styles["Normal"]
    normal.font.name = "Calibri"
    normal.font.size = Pt(11)
    normal.paragraph_format.space_after = Pt(6)

    s = doc.sections[0]
    s.page_width, s.page_height = Cm(21), Cm(29.7)
    for m in ("top_margin", "bottom_margin", "left_margin", "right_margin"):
        setattr(s, m, Cm(2.5))

    cab = s.header.paragraphs[0]
    r = cab.add_run("MI EMPRESA, S.L.")
    r.bold, r.font.color.rgb, r.font.size = True, AZUL, Pt(14)
    cab.add_run("\nC/ Mayor 1 · 28013 Madrid · info@miempresa.com").font.size = Pt(8)

    pie = s.footer.paragraphs[0]
    pie.text = "Este documento ha sido generado automáticamente."
    pie.alignment = WD_ALIGN_PARAGRAPH.CENTER
    pie.runs[0].font.size = Pt(8)


def crear_carta(cliente: dict, destino: Path):
    doc = Document()
    configurar(doc)

    p = doc.add_paragraph(f"Madrid, {fecha_larga(date.today())}")
    p.alignment = WD_ALIGN_PARAGRAPH.RIGHT

    doc.add_paragraph(f"{cliente['nombre']}\n{cliente['direccion']}\n{cliente['ciudad']}")

    asunto = doc.add_paragraph()
    asunto.add_run("Asunto: ").bold = True
    asunto.add_run("Recordatorio de facturas pendientes")
    asunto.paragraph_format.space_before = Pt(18)

    doc.add_paragraph(f"Estimado/a {cliente['contacto']}:")
    cuerpo = doc.add_paragraph(
        "Le informamos de que, según nuestros registros, a fecha de hoy figuran "
        "pendientes de pago las siguientes facturas:"
    )
    cuerpo.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY

    facturas = [f.split(":") for f in cliente["facturas"].split(";")]
    tabla = doc.add_table(rows=1, cols=3)
    tabla.style = "Table Grid"
    tabla.alignment = WD_TABLE_ALIGNMENT.CENTER
    for celda, texto in zip(tabla.rows[0].cells, ("Factura", "Vencimiento", "Importe")):
        celda.text = texto
        celda.paragraphs[0].runs[0].bold = True
        celda.paragraphs[0].runs[0].font.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
        fondo_celda(celda, "1F4E78")

    total = 0.0
    for numero, vencimiento, importe in facturas:
        fila = tabla.add_row().cells
        fila[0].text, fila[1].text, fila[2].text = numero, vencimiento, euros(float(importe))
        fila[2].paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.RIGHT
        total += float(importe)

    fila = tabla.add_row().cells
    fila[0].merge(fila[1]).text = "TOTAL PENDIENTE"
    fila[2].text = euros(total)
    for c in (fila[0], fila[2]):
        c.paragraphs[0].runs[0].bold = True
    fila[2].paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.RIGHT

    for f in tabla.rows:
        for c, ancho in zip(f.cells, (Cm(5), Cm(5), Cm(5))):
            c.width = ancho

    doc.add_paragraph().paragraph_format.space_after = Pt(0)
    texto = doc.add_paragraph(
        "Le rogamos que proceda a su abono en un plazo de 15 días mediante transferencia "
        "a la cuenta ES12 3456 7890 1234 5678 9012. Si ya ha realizado el pago, "
        "le pedimos que ignore esta comunicación."
    )
    texto.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY

    doc.add_paragraph("Atentamente,").paragraph_format.space_before = Pt(18)
    firma = doc.add_paragraph()
    firma.paragraph_format.space_before = Pt(36)
    firma.add_run("Laura Martín\n").bold = True
    firma.add_run("Departamento de Administración")

    doc.core_properties.title = f"Recordatorio de pago — {cliente['nombre']}"
    doc.core_properties.author = "Mi Empresa, S.L."
    doc.save(destino)


def main():
    entrada = Path("clientes.csv")
    if not entrada.exists():
        entrada.write_text(
            "nombre,contacto,direccion,ciudad,facturas\n"
            '"Comercial Norte, S.A.",Sr. Pérez,Av. Libertad 10,48001 Bilbao,'
            '"F-101:15/10/2026:1250.00;F-118:30/10/2026:480.50"\n'
            "Distribuciones Sur,Sra. Gómez,C/ Feria 22,41003 Sevilla,F-120:05/11/2026:899.90\n",
            encoding="utf-8",
        )

    salida = Path("cartas")
    salida.mkdir(exist_ok=True)
    with entrada.open(encoding="utf-8", newline="") as f:
        for cliente in csv.DictReader(f):
            nombre_archivo = "".join(c for c in cliente["nombre"] if c.isalnum() or c == " ")
            ruta = salida / f"Recordatorio - {nombre_archivo.strip()}.docx"
            crear_carta(cliente, ruta)
            print(f"✓ {ruta}")


if __name__ == "__main__":
    main()
```

---

## 12. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'docx'` | Paquete no instalado. | `pip install python-docx` (no `pip install docx`). |
| `PackageNotFoundError` | Archivo `.doc` antiguo o ruta incorrecta. | Convierta a `.docx` y revise la ruta. |
| `KeyError: "no style with name 'Light Grid Accent 1'"` | El estilo no existe en el documento/plantilla. | Liste los estilos o use `Table Grid`. |
| El reemplazo de marcadores no funciona | El marcador está dividido en varios *runs*. | Reemplace a nivel de párrafo (ver 10.1) o use docxtpl. |
| La fuente no cambia en textos asiáticos o especiales | Falta definir la fuente East Asian. | Ajuste `rPr.rFonts` con `qn("w:eastAsia")`. |
| El ancho de columnas no se aplica | `autofit` activo o ancho solo en columnas. | Asigne `cell.width` a cada celda. |
| `PermissionError` al guardar | El archivo está abierto en Word. | Cierre el documento antes de guardar. |

---

## 13. Buenas prácticas

- Parta de una **plantilla `.docx`** con los estilos corporativos ya definidos.
- Use estilos de título (`Heading 1`, `Heading 2`…) en lugar de formato manual: permiten generar índices y facilitan la navegación.
- Separe los datos (CSV, base de datos) de la lógica de generación del documento.
- Encapsule las manipulaciones XML (colores de celda, hipervínculos) en funciones reutilizables.
- Para plantillas con bucles y condiciones, valore **docxtpl**.
- Revise el resultado en Word y en LibreOffice si los destinatarios usan distintos programas.

---

## 14. Referencia rápida

```python
doc = Document() | Document("plantilla.docx")
doc.add_heading(texto, level=0..9)
p = doc.add_paragraph(texto, style="List Bullet")
r = p.add_run(texto); r.bold = True; r.font.size = Pt(12); r.font.color.rgb = RGBColor(...)
p.alignment = WD_ALIGN_PARAGRAPH.CENTER; p.paragraph_format.space_after = Pt(6)
t = doc.add_table(rows, cols); t.style = "Table Grid"; t.add_row(); t.cell(r, c).text
doc.add_picture(ruta, width=Cm(15)); doc.add_page_break()
s = doc.sections[0]; s.header / s.footer / s.orientation / s.left_margin
doc.styles["Normal"].font.name = "Calibri"
doc.core_properties.title = "..."
doc.save("salida.docx")
```

---

## 15. Ejercicios propuestos

1. Genere un informe mensual con portada, índice de apartados, tabla de datos y gráfico de Matplotlib.
2. Cree una plantilla de contrato con marcadores y genere un contrato por cada fila de un CSV.
3. Extraiga todas las tablas de un documento Word y guárdelas en un archivo Excel, una por hoja.
4. Escriba un script que reemplace un término (por ejemplo, el nombre antiguo de la empresa) en todos los `.docx` de una carpeta.

---

## 16. Referencias

- Documentación oficial: https://python-docx.readthedocs.io/
- docxtpl (plantillas Jinja2): https://docxtpl.readthedocs.io/
