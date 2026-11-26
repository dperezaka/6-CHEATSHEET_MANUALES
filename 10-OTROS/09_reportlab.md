# Manual de uso: ReportLab

> **Categoría:** Utilidades generales · **Utilidad:** Generar documentos PDF
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Nivel bajo: Canvas](#4-nivel-bajo-canvas)
5. [Nivel alto: Platypus](#5-nivel-alto-platypus)
6. [Estilos de texto](#6-estilos-de-texto)
7. [Tablas](#7-tablas)
8. [Imágenes, gráficos y códigos](#8-imágenes-gráficos-y-códigos)
9. [Operaciones avanzadas](#9-operaciones-avanzadas)
10. [Ejemplo práctico completo](#10-ejemplo-práctico-completo)
11. [Errores frecuentes](#11-errores-frecuentes)
12. [Buenas prácticas](#12-buenas-prácticas)
13. [Referencia rápida](#13-referencia-rápida)
14. [Ejercicios propuestos](#14-ejercicios-propuestos)
15. [Referencias](#15-referencias)

---

## 1. Descripción

**ReportLab** es la librería de referencia en Python para generar documentos PDF desde código. Se utiliza para crear facturas, informes, certificados, etiquetas, catálogos y cualquier documento que deba producirse automáticamente.

Ofrece dos niveles de trabajo:

| Nivel | Módulo | Uso |
|---|---|---|
| **Bajo** | `canvas` | Dibujo con coordenadas exactas (etiquetas, certificados, plantillas fijas). |
| **Alto** | `platypus` | Documentos con flujo de contenido: párrafos, tablas, saltos de página automáticos (informes, facturas largas). |

---

## 2. Instalación e importación

```bash
pip install reportlab
```

```python
from reportlab.lib.pagesizes import A4, landscape
from reportlab.lib.units import cm, mm
from reportlab.lib import colors
from reportlab.pdfgen import canvas
from reportlab.platypus import (
    SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle,
    Image, PageBreak, KeepTogether,
)
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.enums import TA_CENTER, TA_RIGHT, TA_JUSTIFY
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Unidad base | El **punto** (1 pt = 1/72 pulgada). Use `cm` y `mm` para mayor claridad. |
| Origen de coordenadas | Esquina **inferior izquierda** de la página. |
| `A4` | Tupla `(595.27, 841.89)` puntos. |
| *Flowable* | Elemento que fluye en el documento (párrafo, tabla, imagen…). |
| *Story* | Lista de flowables que forman el documento. |
| Plantilla de página | Define márgenes, encabezados y pies. |

---

## 4. Nivel bajo: Canvas

### 4.1 Documento mínimo

```python
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import A4

c = canvas.Canvas("hola.pdf", pagesize=A4)
ancho, alto = A4

c.drawString(2 * cm, alto - 2 * cm, "Hola, ReportLab")
c.showPage()       # Finaliza la página
c.save()           # Guarda el archivo
```

### 4.2 Texto

```python
c.setFont("Helvetica-Bold", 18)
c.drawString(2*cm, alto - 3*cm, "Título a la izquierda")
c.drawCentredString(ancho / 2, alto - 4*cm, "Texto centrado")
c.drawRightString(ancho - 2*cm, alto - 5*cm, "Texto a la derecha")

c.setFillColor(colors.HexColor("#1F4E78"))
c.setFont("Helvetica", 11)
```

Fuentes estándar incluidas: `Helvetica`, `Helvetica-Bold`, `Helvetica-Oblique`, `Times-Roman`, `Times-Bold`, `Courier`, `Courier-Bold`, `Symbol`, `ZapfDingbats`.

### 4.3 Texto de varias líneas

```python
texto = c.beginText(2*cm, alto - 7*cm)
texto.setFont("Helvetica", 11)
texto.setLeading(15)                     # Interlineado
for linea in ["Primera línea", "Segunda línea", "Tercera línea"]:
    texto.textLine(linea)
c.drawText(texto)
```

### 4.4 Formas y líneas

```python
c.setStrokeColor(colors.grey)
c.setLineWidth(1)
c.line(2*cm, alto - 9*cm, ancho - 2*cm, alto - 9*cm)

c.setFillColor(colors.lightblue)
c.rect(2*cm, alto - 13*cm, 6*cm, 3*cm, stroke=1, fill=1)
c.roundRect(9*cm, alto - 13*cm, 6*cm, 3*cm, radius=10, stroke=1, fill=0)
c.circle(ancho / 2, alto - 16*cm, 1.5*cm, stroke=1, fill=1)
c.ellipse(2*cm, alto - 20*cm, 8*cm, alto - 18*cm)
```

### 4.5 Imágenes

```python
c.drawImage("logo.png", 2*cm, alto - 4*cm, width=4*cm, height=2*cm,
            preserveAspectRatio=True, mask="auto")
```

### 4.6 Varias páginas y numeración

```python
for n in range(1, 4):
    c.drawString(2*cm, alto - 2*cm, f"Contenido de la página {n}")
    c.drawRightString(ancho - 2*cm, 1.5*cm, f"Página {n}")
    c.showPage()
c.save()
```

### 4.7 Metadatos del documento

```python
c.setTitle("Informe anual")
c.setAuthor("Departamento Financiero")
c.setSubject("Resultados 2026")
```

---

## 5. Nivel alto: Platypus

### 5.1 Documento básico

```python
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet

doc = SimpleDocTemplate(
    "informe.pdf", pagesize=A4,
    leftMargin=2*cm, rightMargin=2*cm, topMargin=2*cm, bottomMargin=2*cm,
    title="Informe", author="Mi Empresa",
)
estilos = getSampleStyleSheet()

story = [
    Paragraph("Informe trimestral", estilos["Title"]),
    Spacer(1, 0.5*cm),
    Paragraph("1. Introducción", estilos["Heading2"]),
    Paragraph("Este documento resume los resultados del tercer trimestre. " * 10,
              estilos["BodyText"]),
]

doc.build(story)
```

Los saltos de página se gestionan automáticamente.

### 5.2 Flowables principales

| Flowable | Función |
|---|---|
| `Paragraph(texto, estilo)` | Párrafo con ajuste de línea y marcado. |
| `Spacer(ancho, alto)` | Espacio vertical. |
| `Table(datos)` | Tabla. |
| `Image(ruta, ancho, alto)` | Imagen. |
| `PageBreak()` | Salto de página forzado. |
| `KeepTogether([...])` | Mantiene elementos en la misma página. |
| `ListFlowable` | Listas con viñetas o numeradas. |
| `HRFlowable` | Línea horizontal. |

### 5.3 Marcado dentro de `Paragraph`

```python
Paragraph(
    'Texto en <b>negrita</b>, <i>cursiva</i>, <u>subrayado</u>, '
    '<font color="red" size="12">color</font>, '
    'H<sub>2</sub>O, x<super>2</super> y '
    '<a href="https://www.python.org" color="blue">enlaces</a>.<br/>'
    'Nueva línea.',
    estilos["Normal"],
)
```

> Los caracteres `&`, `<` y `>` en textos dinámicos deben escaparse (`from xml.sax.saxutils import escape`).

### 5.4 Listas

```python
from reportlab.platypus import ListFlowable, ListItem

lista = ListFlowable(
    [ListItem(Paragraph(t, estilos["Normal"])) for t in ["Ventas", "Costes", "Margen"]],
    bulletType="bullet", start="•", leftIndent=15,
)
numerada = ListFlowable(
    [Paragraph(t, estilos["Normal"]) for t in ["Paso uno", "Paso dos"]],
    bulletType="1",
)
```

---

## 6. Estilos de texto

### 6.1 Estilos predefinidos

`Title`, `Heading1` … `Heading6`, `Normal`, `BodyText`, `Italic`, `Bullet`, `Code`, `Definition`.

### 6.2 Crear estilos propios

```python
estilos.add(ParagraphStyle(
    name="Cuerpo",
    parent=estilos["Normal"],
    fontName="Helvetica",
    fontSize=10.5,
    leading=15,
    alignment=TA_JUSTIFY,
    spaceAfter=8,
))

estilos.add(ParagraphStyle(
    name="TituloSeccion",
    parent=estilos["Heading2"],
    textColor=colors.HexColor("#1F4E78"),
    spaceBefore=12,
    spaceAfter=6,
))

estilos.add(ParagraphStyle(name="Derecha", parent=estilos["Normal"], alignment=TA_RIGHT))
```

| Propiedad | Descripción |
|---|---|
| `fontName`, `fontSize` | Fuente y tamaño. |
| `leading` | Interlineado (≈ 1,2–1,4 × tamaño). |
| `alignment` | `TA_LEFT`, `TA_CENTER`, `TA_RIGHT`, `TA_JUSTIFY`. |
| `textColor`, `backColor` | Colores. |
| `spaceBefore`, `spaceAfter` | Espaciado vertical. |
| `leftIndent`, `firstLineIndent` | Sangrías. |

### 6.3 Fuentes TrueType (tildes, ñ y diseño propio)

```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

pdfmetrics.registerFont(TTFont("DejaVu", "DejaVuSans.ttf"))
pdfmetrics.registerFont(TTFont("DejaVu-Bold", "DejaVuSans-Bold.ttf"))

estilo = ParagraphStyle("Propio", fontName="DejaVu", fontSize=11)
```

> Las fuentes estándar admiten los caracteres del español. Registre fuentes TrueType para otros alfabetos o para usar la tipografía corporativa.

---

## 7. Tablas

### 7.1 Tabla básica

```python
datos = [
    ["Producto", "Unidades", "Precio", "Total"],
    ["Teclado", 2, "29,90 €", "59,80 €"],
    ["Ratón", 5, "14,50 €", "72,50 €"],
]
tabla = Table(datos, colWidths=[7*cm, 3*cm, 3*cm, 3*cm])
```

### 7.2 Estilo de tabla

```python
tabla.setStyle(TableStyle([
    # (comando, (col_inicio, fila_inicio), (col_fin, fila_fin), valor)
    ("BACKGROUND", (0, 0), (-1, 0), colors.HexColor("#1F4E78")),
    ("TEXTCOLOR", (0, 0), (-1, 0), colors.white),
    ("FONTNAME", (0, 0), (-1, 0), "Helvetica-Bold"),
    ("ALIGN", (1, 0), (-1, -1), "RIGHT"),
    ("VALIGN", (0, 0), (-1, -1), "MIDDLE"),
    ("GRID", (0, 0), (-1, -1), 0.5, colors.grey),
    ("ROWBACKGROUNDS", (0, 1), (-1, -1), [colors.white, colors.HexColor("#F2F2F2")]),
    ("BOTTOMPADDING", (0, 0), (-1, -1), 6),
    ("TOPPADDING", (0, 0), (-1, -1), 6),
]))
```

> Los índices negativos funcionan como en Python: `-1` es la última fila o columna.

| Comando | Efecto |
|---|---|
| `BACKGROUND` | Color de fondo. |
| `TEXTCOLOR` | Color del texto. |
| `FONTNAME`, `FONTSIZE` | Fuente y tamaño. |
| `ALIGN`, `VALIGN` | Alineación horizontal y vertical. |
| `GRID`, `BOX`, `INNERGRID` | Bordes. |
| `LINEABOVE`, `LINEBELOW` | Líneas horizontales. |
| `SPAN` | Combinar celdas. |
| `ROWBACKGROUNDS` | Colores alternos por fila. |
| `LEFTPADDING`, `TOPPADDING`… | Márgenes internos. |

### 7.3 Celdas con texto largo

Use `Paragraph` dentro de las celdas para que el texto se ajuste:

```python
datos = [
    ["Concepto", "Descripción"],
    ["Servicio", Paragraph("Mantenimiento anual de equipos informáticos, incluidas "
                           "revisiones trimestrales y soporte remoto.", estilos["Normal"])],
]
```

### 7.4 Repetir la cabecera en cada página

```python
tabla = Table(datos, repeatRows=1)
```

---

## 8. Imágenes, gráficos y códigos

### 8.1 Imágenes en Platypus

```python
from reportlab.platypus import Image
from reportlab.lib.utils import ImageReader

def imagen_proporcional(ruta, ancho):
    iw, ih = ImageReader(ruta).getSize()
    return Image(ruta, width=ancho, height=ancho * ih / iw)

story.append(imagen_proporcional("grafico.png", 15*cm))
```

### 8.2 Insertar un gráfico de Matplotlib

```python
import io
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(6, 3))
ax.bar(["Q1", "Q2", "Q3", "Q4"], [120, 135, 150, 170])
buffer = io.BytesIO()
fig.savefig(buffer, format="png", dpi=200, bbox_inches="tight")
plt.close(fig)
buffer.seek(0)

story.append(Image(buffer, width=15*cm, height=7.5*cm))
```

### 8.3 Gráficos nativos (vectoriales)

```python
from reportlab.graphics.shapes import Drawing
from reportlab.graphics.charts.barcharts import VerticalBarChart

dibujo = Drawing(400, 200)
grafico = VerticalBarChart()
grafico.x, grafico.y, grafico.width, grafico.height = 50, 30, 320, 150
grafico.data = [[120, 135, 150, 170]]
grafico.categoryAxis.categoryNames = ["Q1", "Q2", "Q3", "Q4"]
grafico.bars[0].fillColor = colors.HexColor("#1F4E78")
dibujo.add(grafico)
story.append(dibujo)
```

### 8.4 Códigos QR y de barras

```python
from reportlab.graphics.barcode import qr, code128
from reportlab.graphics.shapes import Drawing

codigo_qr = qr.QrCodeWidget("https://www.miempresa.com/factura/2026-101")
b = codigo_qr.getBounds()
tam = 3*cm
d = Drawing(tam, tam, transform=[tam / (b[2] - b[0]), 0, 0, tam / (b[3] - b[1]), 0, 0])
d.add(codigo_qr)
story.append(d)

story.append(code128.Code128("FAC2026101", barHeight=1.5*cm, barWidth=1.2))
```

---

## 9. Operaciones avanzadas

### 9.1 Encabezado y pie de página

```python
def encabezado_pie(c, doc):
    c.saveState()
    ancho, alto = A4
    c.setFont("Helvetica-Bold", 9)
    c.drawString(2*cm, alto - 1.3*cm, "MI EMPRESA, S.L.")
    c.setStrokeColor(colors.HexColor("#1F4E78"))
    c.line(2*cm, alto - 1.5*cm, ancho - 2*cm, alto - 1.5*cm)

    c.setFont("Helvetica", 8)
    c.drawString(2*cm, 1.2*cm, "Documento confidencial")
    c.drawRightString(ancho - 2*cm, 1.2*cm, f"Página {doc.page}")
    c.restoreState()

doc.build(story, onFirstPage=encabezado_pie, onLaterPages=encabezado_pie)
```

### 9.2 «Página X de Y»

```python
from reportlab.pdfgen.canvas import Canvas


class CanvasNumerado(Canvas):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._paginas = []

    def showPage(self):
        self._paginas.append(dict(self.__dict__))
        self._startPage()

    def save(self):
        total = len(self._paginas)
        for estado in self._paginas:
            self.__dict__.update(estado)
            self.setFont("Helvetica", 8)
            self.drawRightString(A4[0] - 2*cm, 0.8*cm, f"Página {self._pageNumber} de {total}")
            super().showPage()
        super().save()

doc.build(story, canvasmaker=CanvasNumerado)
```

### 9.3 Orientación horizontal y tamaños de página

```python
from reportlab.lib.pagesizes import A4, A3, A5, LETTER, landscape, portrait

SimpleDocTemplate("horizontal.pdf", pagesize=landscape(A4))
canvas.Canvas("etiqueta.pdf", pagesize=(10*cm, 5*cm))     # Tamaño personalizado
```

### 9.4 Varias columnas y plantillas propias

```python
from reportlab.platypus import BaseDocTemplate, Frame, PageTemplate

doc = BaseDocTemplate("columnas.pdf", pagesize=A4)
ancho_col = (A4[0] - 5*cm) / 2
frames = [
    Frame(2*cm, 2*cm, ancho_col, A4[1] - 4*cm, id="izq"),
    Frame(3*cm + ancho_col, 2*cm, ancho_col, A4[1] - 4*cm, id="der"),
]
doc.addPageTemplates([PageTemplate(id="DosColumnas", frames=frames)])
doc.build(story)
```

### 9.5 Generar el PDF en memoria (descarga web)

```python
import io

buffer = io.BytesIO()
SimpleDocTemplate(buffer, pagesize=A4).build(story)
pdf_bytes = buffer.getvalue()
```

### 9.6 Marca de agua

```python
def marca_agua(c, doc):
    c.saveState()
    c.setFont("Helvetica-Bold", 70)
    c.setFillColor(colors.Color(0.8, 0.8, 0.8, alpha=0.3))
    c.translate(A4[0] / 2, A4[1] / 2)
    c.rotate(45)
    c.drawCentredString(0, 0, "BORRADOR")
    c.restoreState()
```

---

## 10. Ejemplo práctico completo

Generador de facturas con logotipo opcional, datos de cliente, líneas, totales con IVA, pie de página y código QR.

```python
from dataclasses import dataclass
from datetime import date
from decimal import Decimal, ROUND_HALF_UP
from pathlib import Path
from xml.sax.saxutils import escape

from reportlab.graphics.barcode import qr
from reportlab.graphics.shapes import Drawing
from reportlab.lib import colors
from reportlab.lib.enums import TA_RIGHT
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import ParagraphStyle, getSampleStyleSheet
from reportlab.lib.units import cm
from reportlab.platypus import Image, Paragraph, SimpleDocTemplate, Spacer, Table, TableStyle

AZUL = colors.HexColor("#1F4E78")
GRIS = colors.HexColor("#F2F2F2")


@dataclass
class Linea:
    concepto: str
    unidades: int
    precio: Decimal


def euros(valor: Decimal) -> str:
    v = valor.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def generar_qr(texto: str, tam=2.8 * cm) -> Drawing:
    widget = qr.QrCodeWidget(texto)
    x1, y1, x2, y2 = widget.getBounds()
    d = Drawing(tam, tam, transform=[tam / (x2 - x1), 0, 0, tam / (y2 - y1), 0, 0])
    d.add(widget)
    return d


def pie(c, doc):
    c.saveState()
    c.setFont("Helvetica", 8)
    c.setFillColor(colors.grey)
    c.drawCentredString(A4[0] / 2, 1.2 * cm,
                        "Mi Empresa, S.L. · CIF B12345678 · C/ Mayor 1, 28013 Madrid · info@miempresa.com")
    c.drawRightString(A4[0] - 2 * cm, 1.2 * cm, f"Pág. {doc.page}")
    c.restoreState()


def crear_factura(numero: str, cliente: dict, lineas: list[Linea], salida: Path, logo: Path | None = None):
    est = getSampleStyleSheet()
    est.add(ParagraphStyle("Der", parent=est["Normal"], alignment=TA_RIGHT))
    est.add(ParagraphStyle("Peq", parent=est["Normal"], fontSize=9, leading=12))

    story = []

    # Cabecera: logo + datos de la factura
    izquierda = Image(str(logo), width=4 * cm, height=2 * cm) if logo and logo.exists() \
        else Paragraph("<b><font size=16 color='#1F4E78'>MI EMPRESA</font></b>", est["Normal"])
    derecha = Paragraph(
        f"<font size=18 color='#1F4E78'><b>FACTURA</b></font><br/>"
        f"Nº <b>{escape(numero)}</b><br/>Fecha: {date.today():%d/%m/%Y}", est["Der"])
    cab = Table([[izquierda, derecha]], colWidths=[8.5 * cm, 8.5 * cm])
    cab.setStyle(TableStyle([("VALIGN", (0, 0), (-1, -1), "TOP")]))
    story += [cab, Spacer(1, 0.8 * cm)]

    # Datos del cliente
    datos_cliente = "<br/>".join(escape(str(cliente[k])) for k in ("nombre", "nif", "direccion"))
    caja = Table([[Paragraph(f"<b>Facturar a:</b><br/>{datos_cliente}", est["Peq"])]],
                 colWidths=[17 * cm])
    caja.setStyle(TableStyle([
        ("BACKGROUND", (0, 0), (-1, -1), GRIS),
        ("BOX", (0, 0), (-1, -1), 0.5, colors.lightgrey),
        ("LEFTPADDING", (0, 0), (-1, -1), 10), ("TOPPADDING", (0, 0), (-1, -1), 8),
        ("BOTTOMPADDING", (0, 0), (-1, -1), 8),
    ]))
    story += [caja, Spacer(1, 0.8 * cm)]

    # Líneas
    filas = [["Concepto", "Uds.", "Precio", "Importe"]]
    base = Decimal("0")
    for l in lineas:
        importe = l.precio * l.unidades
        base += importe
        filas.append([Paragraph(escape(l.concepto), est["Peq"]), l.unidades,
                      euros(l.precio), euros(importe)])

    t = Table(filas, colWidths=[9.5 * cm, 2 * cm, 2.75 * cm, 2.75 * cm], repeatRows=1)
    t.setStyle(TableStyle([
        ("BACKGROUND", (0, 0), (-1, 0), AZUL),
        ("TEXTCOLOR", (0, 0), (-1, 0), colors.white),
        ("FONTNAME", (0, 0), (-1, 0), "Helvetica-Bold"),
        ("ALIGN", (1, 0), (-1, -1), "RIGHT"),
        ("VALIGN", (0, 0), (-1, -1), "MIDDLE"),
        ("ROWBACKGROUNDS", (0, 1), (-1, -1), [colors.white, GRIS]),
        ("LINEBELOW", (0, -1), (-1, -1), 0.8, AZUL),
        ("TOPPADDING", (0, 0), (-1, -1), 6), ("BOTTOMPADDING", (0, 0), (-1, -1), 6),
    ]))
    story += [t, Spacer(1, 0.5 * cm)]

    # Totales
    iva = base * Decimal("0.21")
    total = base + iva
    totales = Table([
        ["Base imponible", euros(base)],
        ["IVA (21 %)", euros(iva)],
        ["TOTAL", euros(total)],
    ], colWidths=[4 * cm, 3.5 * cm], hAlign="RIGHT")
    totales.setStyle(TableStyle([
        ("ALIGN", (0, 0), (-1, -1), "RIGHT"),
        ("FONTNAME", (0, -1), (-1, -1), "Helvetica-Bold"),
        ("FONTSIZE", (0, -1), (-1, -1), 12),
        ("BACKGROUND", (0, -1), (-1, -1), AZUL),
        ("TEXTCOLOR", (0, -1), (-1, -1), colors.white),
        ("TOPPADDING", (0, 0), (-1, -1), 5), ("BOTTOMPADDING", (0, 0), (-1, -1), 5),
    ]))
    story += [totales, Spacer(1, 1 * cm)]

    # Forma de pago + QR
    pago = Paragraph(
        "<b>Forma de pago:</b> transferencia bancaria a 30 días.<br/>"
        "IBAN: ES12 3456 7890 1234 5678 9012<br/>"
        f"Concepto: {escape(numero)}", est["Peq"])
    bloque = Table([[pago, generar_qr(f"https://miempresa.com/facturas/{numero}")]],
                   colWidths=[13.5 * cm, 3.5 * cm])
    bloque.setStyle(TableStyle([("VALIGN", (0, 0), (-1, -1), "MIDDLE")]))
    story.append(bloque)

    doc = SimpleDocTemplate(str(salida), pagesize=A4, leftMargin=2 * cm, rightMargin=2 * cm,
                            topMargin=2 * cm, bottomMargin=2.2 * cm,
                            title=f"Factura {numero}", author="Mi Empresa, S.L.")
    doc.build(story, onFirstPage=pie, onLaterPages=pie)


if __name__ == "__main__":
    crear_factura(
        numero="2026-0101",
        cliente={"nombre": "Comercial Ejemplo, S.A.", "nif": "A87654321",
                 "direccion": "Av. de la Constitución 25, 41001 Sevilla"},
        lineas=[
            Linea("Licencia anual de software de gestión", 3, Decimal("249.00")),
            Linea("Horas de consultoría e implantación", 12, Decimal("55.00")),
            Linea("Formación presencial (jornada completa)", 1, Decimal("390.00")),
        ],
        salida=Path("factura_2026-0101.pdf"),
        logo=Path("logo.png"),
    )
    print("Factura generada.")
```

---

## 11. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| El texto aparece en la parte inferior | El origen está abajo a la izquierda. | Calcule `y = alto - margen`. |
| PDF vacío o incompleto | Falta `showPage()` / `save()`. | Llame a ambos al terminar con Canvas. |
| `ValueError: paragraph text ... parse error` | Caracteres `&`, `<`, `>` sin escapar. | Use `escape()` de `xml.sax.saxutils`. |
| Caracteres como cuadrados negros | La fuente no tiene esos glifos. | Registre una fuente TrueType adecuada. |
| `LayoutError: Flowable too large` | Elemento más alto que la página. | Reduzca la imagen o divida la tabla. |
| Texto de celda que se sale | Cadena simple en la celda. | Use `Paragraph` dentro de la celda. |
| Imagen deformada | Ancho y alto no proporcionales. | Calcule el alto a partir de la proporción real. |

---

## 12. Buenas prácticas

- Use **Platypus** para documentos con contenido variable y **Canvas** para plantillas fijas.
- Trabaje con unidades `cm` / `mm` en lugar de puntos.
- Centralice colores, fuentes y estilos en un módulo propio para mantener la identidad corporativa.
- Escape siempre el texto dinámico que se inserta en `Paragraph`.
- Use `Decimal` para importes y redondee al final.
- Use `repeatRows=1` en tablas largas.
- Rellene los metadatos (`title`, `author`) para mejorar la accesibilidad y la indexación.

---

## 13. Referencia rápida

```python
# Canvas
c = canvas.Canvas("x.pdf", pagesize=A4); c.setFont("Helvetica", 12)
c.drawString(x, y, t) / drawCentredString / drawRightString / drawImage / line / rect / circle
c.showPage(); c.save()

# Platypus
doc = SimpleDocTemplate("x.pdf", pagesize=A4, leftMargin=2*cm, ...)
story = [Paragraph(t, estilo), Spacer(1, 12), Table(datos), Image(r, w, h), PageBreak()]
doc.build(story, onFirstPage=f, onLaterPages=f)

estilos = getSampleStyleSheet(); ParagraphStyle(name, parent, fontSize, leading, alignment)
TableStyle([("GRID", (0,0), (-1,-1), 0.5, colors.grey), ...])
pdfmetrics.registerFont(TTFont("Nombre", "fuente.ttf"))
```

---

## 14. Ejercicios propuestos

1. Genere un certificado de asistencia en formato horizontal con nombre, curso, fecha y firma.
2. Cree un informe de varias páginas a partir de un CSV con tabla de datos, gráfico de Matplotlib y numeración «Página X de Y».
3. Produzca una hoja de etiquetas de envío (3 × 8 por página A4) con código de barras.
4. Genere un catálogo de productos con imagen, descripción y precio, dos productos por fila.

---

## 15. Referencias

- Documentación oficial: https://docs.reportlab.com/
- Guía de usuario (PDF): https://www.reportlab.com/docs/reportlab-userguide.pdf
