# Manual de uso: OpenPyXL

> **Categoría:** Ofimática · **Utilidad:** Leer, crear y modificar archivos de Excel (.xlsx)
> **Tipo:** Librería de terceros (`pip install openpyxl`) · **Versión de referencia:** 3.1 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Libros y hojas](#4-libros-y-hojas)
5. [Leer y escribir celdas](#5-leer-y-escribir-celdas)
6. [Recorrer rangos](#6-recorrer-rangos)
7. [Fórmulas](#7-fórmulas)
8. [Formato de números, fechas y texto](#8-formato-de-números-fechas-y-texto)
9. [Estilos: fuentes, rellenos, bordes y alineación](#9-estilos-fuentes-rellenos-bordes-y-alineación)
10. [Dimensiones, combinación e inmovilización](#10-dimensiones-combinación-e-inmovilización)
11. [Tablas de Excel y autofiltro](#11-tablas-de-excel-y-autofiltro)
12. [Formato condicional](#12-formato-condicional)
13. [Validación de datos](#13-validación-de-datos)
14. [Gráficos](#14-gráficos)
15. [Imágenes, comentarios e hipervínculos](#15-imágenes-comentarios-e-hipervínculos)
16. [Nombres definidos](#16-nombres-definidos)
17. [Protección](#17-protección)
18. [Configuración de impresión](#18-configuración-de-impresión)
19. [Archivos grandes: modos de solo lectura y solo escritura](#19-archivos-grandes-modos-de-solo-lectura-y-solo-escritura)
20. [Integración con pandas](#20-integración-con-pandas)
21. [Ejemplo práctico completo](#21-ejemplo-práctico-completo)
22. [Errores frecuentes](#22-errores-frecuentes)
23. [Buenas prácticas](#23-buenas-prácticas)
24. [Referencia rápida](#24-referencia-rápida)
25. [Ejercicios propuestos](#25-ejercicios-propuestos)
26. [Referencias](#26-referencias)

---

## 1. Descripción

**OpenPyXL** lee y escribe archivos de Excel en formato Office Open XML (`.xlsx`, `.xlsm`, `.xltx`, `.xltm`) sin necesidad de tener Excel instalado. Permite generar informes con formato profesional, fórmulas, tablas, gráficos, formato condicional y validaciones, y también leer y modificar libros existentes.

**Ventajas:**
- No requiere Excel: funciona en servidores Linux, Docker y la nube.
- Control detallado de formato, estilos y estructura del libro.
- Lectura y escritura del mismo archivo (modificar plantillas existentes).
- Modos optimizados para archivos muy grandes.

**Limitaciones:**
- **No calcula fórmulas**: las escribe, y Excel (o LibreOffice) las calcula al abrir el archivo.
- No admite el formato antiguo `.xls` (use `xlrd` para leer o convierta a `.xlsx`).
- Puede perder elementos no soportados al guardar un libro existente (macros si no se usa `keep_vba`, segmentaciones, algunos gráficos o formas complejas).
- Para análisis de datos, pandas es más productivo; OpenPyXL destaca en la **presentación**.

| Si necesita… | Opción |
|---|---|
| Crear informes con formato o modificar plantillas `.xlsx` | **OpenPyXL** |
| Solo escribir archivos nuevos, muy rápido y con gráficos avanzados | XlsxWriter |
| Analizar datos tabulares | pandas (usa OpenPyXL por debajo) |
| Controlar Excel abierto (macros, recálculo, Windows/macOS) | xlwings |
| Leer `.xls` antiguos | xlrd / pandas |

---

## 2. Instalación e importación

```bash
pip install openpyxl
pip install pillow        # Opcional: para insertar imágenes
```

```python
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, PatternFill, Border, Side, Alignment, NamedStyle
from openpyxl.utils import get_column_letter, column_index_from_string
```

---

## 3. Conceptos clave

| Objeto | Descripción |
|---|---|
| `Workbook` | El libro (archivo). |
| `Worksheet` | Una hoja del libro. |
| `Cell` | Una celda: `value`, `number_format`, `font`, `fill`, `border`, `alignment`… |
| Coordenadas | `"B3"` (letra de columna + fila) o `(fila=3, columna=2)`; **ambas empiezan en 1**. |
| Rango | `"A1:D10"`; se recorre como tuplas de filas. |

---

## 4. Libros y hojas

```python
from openpyxl import Workbook, load_workbook

wb = Workbook()                         # Libro nuevo con una hoja
ws = wb.active                          # Hoja activa
ws.title = "Ventas"
resumen = wb.create_sheet("Resumen")    # Al final
portada = wb.create_sheet("Portada", 0) # En la primera posición
wb.sheetnames                           # ['Portada', 'Ventas', 'Resumen']
ws = wb["Ventas"]                       # Acceso por nombre
copia = wb.copy_worksheet(ws)           # Copia dentro del mismo libro
wb.move_sheet("Resumen", offset=-1)
del wb["Portada"]                       # o wb.remove(wb["Portada"])
ws.sheet_properties.tabColor = "1F4E78"
ws.sheet_state = "hidden"               # Ocultar ("visible", "hidden", "veryHidden")
wb.active = wb["Resumen"]               # Hoja que se mostrará al abrir
wb.save("informe.xlsx")

wb = load_workbook("plantilla.xlsx")                   # Leer y modificar (fórmulas como texto)
wb = load_workbook("datos.xlsx", data_only=True)       # Valores calculados guardados por Excel
wb = load_workbook("con_macros.xlsm", keep_vba=True)   # Conservar macros
```

> **Guardar con `data_only=True` elimina las fórmulas** (las sustituye por sus valores). Use dos cargas distintas si necesita leer resultados y conservar fórmulas.

---

## 5. Leer y escribir celdas

```python
ws["A1"] = "Informe de ventas"
ws["B2"] = 1250.5
ws["C2"] = datetime.date(2026, 3, 31)
ws["D2"] = True
ws.cell(row=3, column=2, value=980)           # Por índices
valor = ws["B2"].value
c = ws.cell(row=3, column=2)
c.coordinate, c.row, c.column, c.column_letter    # 'B3', 3, 2, 'B'

ws.append(["Fecha", "Producto", "Unidades", "Importe"])   # Añade una fila tras la última usada
ws.append([datetime.date(2026, 1, 5), "Silla", 4, 356.0])

ws.max_row, ws.max_column, ws.dimensions     # Última fila/columna con datos, rango usado ('A1:D2')
ws.insert_rows(2, amount=3) ; ws.delete_rows(5) ; ws.insert_cols(2) ; ws.delete_cols(4, 2)
ws.move_range("A10:D20", rows=-2, cols=1, translate=True)   # translate: ajusta las fórmulas movidas
```

| Tipo Python | En Excel |
|---|---|
| `int`, `float`, `Decimal` | Número. |
| `str` | Texto (si empieza por `=`, fórmula). |
| `datetime`, `date`, `time` | Fecha/hora (con formato de fecha). |
| `bool` | VERDADERO/FALSO. |
| `None` | Celda vacía. |

---

## 6. Recorrer rangos

```python
for fila in ws.iter_rows(min_row=2, max_col=4, values_only=True):    # Tuplas de valores
    fecha, producto, unidades, importe = fila

for fila in ws.iter_rows(min_row=2):                                 # Tuplas de celdas (para modificar)
    for celda in fila:
        if isinstance(celda.value, str):
            celda.value = celda.value.strip()

for columna in ws.iter_cols(min_col=3, max_col=3, min_row=2, values_only=True):
    total = sum(v or 0 for v in columna)

bloque = ws["A1:D10"]                 # Tupla de filas
columna_b = ws["B"] ; fila_1 = ws[1]

# Cabeceras → diccionarios
filas = ws.iter_rows(values_only=True)
cabecera = next(filas)
registros = [dict(zip(cabecera, f)) for f in filas if any(v is not None for v in f)]
```

---

## 7. Fórmulas

```python
ws["E2"] = "=C2*D2"
ws["E10"] = "=SUM(E2:E9)"
ws["F2"] = '=IF(E2>1000,"Alta","Normal")'
ws["G2"] = "=SUMIFS(Datos!$E:$E,Datos!$B:$B,$A2,Datos!$C:$C,G$1)"
ws["H2"] = "=IFERROR(VLOOKUP(A2,Tarifas!$A:$C,3,FALSE),0)"
ws["I2"] = "=_xlfn.XLOOKUP(A2,Tarifas!A:A,Tarifas!C:C)"      # Funciones modernas: prefijo _xlfn.

from openpyxl.worksheet.formula import ArrayFormula
ws["J2"] = ArrayFormula("J2:J10", "=C2:C10*D2:D10")             # Fórmula matricial clásica
```

- Escriba los **nombres de funciones en inglés** y con **comas** como separador, aunque Excel esté en español: Excel los traducirá al abrir (`SUMA`, `;`).
- OpenPyXL no calcula: al leer un libro recién generado con `data_only=True`, las fórmulas devuelven `None` hasta que se abra y guarde en Excel. Para recalcular en un servidor puede usar LibreOffice: `soffice --headless --convert-to xlsx --outdir calculado informe.xlsx`.
- Funciones posteriores a Excel 2010 (`XLOOKUP`, `FILTER`, `UNIQUE`, `IFS`, `TEXTJOIN`…) requieren el prefijo `_xlfn.` para que Excel las reconozca.

---

## 8. Formato de números, fechas y texto

```python
ws["B2"].number_format = "#,##0.00 €"            # Excel aplica los separadores de la configuración regional
ws["C2"].number_format = "0.0%"
ws["D2"].number_format = "DD/MM/YYYY"
ws["E2"].number_format = "#,##0;[Red]-#,##0"     # Negativos en rojo
ws["F2"].number_format = '#,##0,"k"'             # Miles
ws["G2"].number_format = "@"                     # Texto (conserva ceros a la izquierda: códigos postales)

from openpyxl.styles import numbers
numbers.FORMAT_PERCENTAGE_00, numbers.FORMAT_DATE_DDMMYY
```

> En el código de formato, `,` es el separador de miles y `.` el decimal (notación inglesa); Excel lo mostrará como `1.234,56` en una configuración española.

---

## 9. Estilos: fuentes, rellenos, bordes y alineación

```python
from openpyxl.styles import Font, PatternFill, Border, Side, Alignment, NamedStyle

celda = ws["A1"]
celda.font = Font(name="Calibri", size=14, bold=True, italic=False, color="FFFFFF", underline="single")
celda.fill = PatternFill("solid", start_color="1F4E78")
fina = Side(style="thin", color="BFBFBF")
celda.border = Border(left=fina, right=fina, top=fina, bottom=Side(style="medium", color="1F4E78"))
celda.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True, indent=0, text_rotation=0)

# Los estilos son inmutables: para cambiar un atributo, copie con modificaciones
celda.font = celda.font.copy(size=16)     # o Font(**{...})

# Estilo con nombre: más eficiente y reutilizable
encabezado = NamedStyle(name="encabezado")
encabezado.font = Font(bold=True, color="FFFFFF")
encabezado.fill = PatternFill("solid", start_color="1F4E78")
encabezado.alignment = Alignment(horizontal="center", vertical="center")
wb.add_named_style(encabezado)
for c in ws[1]:
    c.style = "encabezado"
```

Aplicar un estilo a un rango:

```python
def estilo_rango(ws, rango: str, **atributos) -> None:
    for fila in ws[rango]:
        for celda in fila:
            for nombre, valor in atributos.items():
                setattr(celda, nombre, valor)

estilo_rango(ws, "A2:F20", border=Border(bottom=fina), alignment=Alignment(vertical="center"))
```

---

## 10. Dimensiones, combinación e inmovilización

```python
ws.column_dimensions["A"].width = 28                   # Ancho en caracteres aproximados
ws.row_dimensions[1].height = 32                        # Alto en puntos
ws.column_dimensions["D"].hidden = True
ws.column_dimensions.group("E", "G", hidden=True)       # Agrupar (esquema)

ws.merge_cells("A1:F1") ; ws.unmerge_cells("A1:F1")     # Solo conserva el valor de la celda superior izquierda
ws.freeze_panes = "B2"                                  # Inmoviliza la fila 1 y la columna A
ws.sheet_view.showGridLines = False
ws.sheet_view.zoomScale = 90

def autoajustar(ws, minimo=8, maximo=60) -> None:
    """Ajuste aproximado del ancho de las columnas al contenido."""
    for columna in ws.columns:
        largo = max((len(str(c.value)) for c in columna if c.value is not None), default=0)
        ws.column_dimensions[columna[0].column_letter].width = max(minimo, min(maximo, largo + 2))
```

---

## 11. Tablas de Excel y autofiltro

Las **tablas** (Insertar → Tabla) aportan estilo con bandas, filtros y referencias estructuradas:

```python
from openpyxl.worksheet.table import Table, TableStyleInfo

tabla = Table(displayName="TablaVentas", ref=f"A1:F{ws.max_row}")
tabla.tableStyleInfo = TableStyleInfo(name="TableStyleMedium2", showRowStripes=True)
ws.add_table(tabla)

ws["H2"] = "=SUM(TablaVentas[Importe])"                 # Referencia estructurada

# Autofiltro simple (sin tabla)
ws.auto_filter.ref = "A1:F200"
```

- `displayName` debe ser único en el libro, sin espacios.
- Las cabeceras deben ser texto y no repetirse.
- Una tabla no puede solaparse con celdas combinadas ni con otro autofiltro.

---

## 12. Formato condicional

```python
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule, IconSetRule, CellIsRule, FormulaRule
from openpyxl.styles import PatternFill, Font

ws.conditional_formatting.add("E2:E100", ColorScaleRule(start_type="min", start_color="F8696B",
                                                        mid_type="percentile", mid_value=50, mid_color="FFEB84",
                                                        end_type="max", end_color="63BE7B"))
ws.conditional_formatting.add("D2:D100", DataBarRule(start_type="min", end_type="max", color="5B9BD5"))
ws.conditional_formatting.add("F2:F100", IconSetRule("3Arrows", "percent", [0, 33, 67]))
ws.conditional_formatting.add("C2:C100", CellIsRule(operator="lessThan", formula=["0"],
                                                    font=Font(color="9C0006"), fill=PatternFill("solid", start_color="FFC7CE")))
ws.conditional_formatting.add("A2:F100", FormulaRule(formula=['$F2="Pendiente"'],
                                                     fill=PatternFill("solid", start_color="FFF2CC")))   # Fila completa
```

> En `FormulaRule`, la fórmula se escribe relativa a la **primera celda** del rango (`$F2` para todo `A2:F100`).

---

## 13. Validación de datos

```python
from openpyxl.worksheet.datavalidation import DataValidation

lista = DataValidation(type="list", formula1='"Pendiente,Enviado,Entregado,Cancelado"', allow_blank=True,
                       showErrorMessage=True, errorTitle="Valor no válido", error="Elija un estado de la lista")
ws.add_data_validation(lista)
lista.add("F2:F500")

desde_rango = DataValidation(type="list", formula1="=Parametros!$A$2:$A$10")   # Lista desde otro rango
importes = DataValidation(type="decimal", operator="between", formula1="0", formula2="100000",
                          promptTitle="Importe", prompt="Introduzca un importe entre 0 y 100.000", showInputMessage=True)
fechas = DataValidation(type="date", operator="greaterThanOrEqual", formula1="DATE(2026,1,1)")
for v in (desde_rango, importes, fechas):
    ws.add_data_validation(v)
importes.add("E2:E500")
```

> Una lista literal (`'"a,b,c"'`) no puede superar 255 caracteres; para listas largas, use un rango.

---

## 14. Gráficos

```python
from openpyxl.chart import BarChart, LineChart, PieChart, ScatterChart, Reference, Series
from openpyxl.chart.label import DataLabelList

grafico = BarChart()
grafico.type = "col"                       # "bar" para horizontales
grafico.grouping = "clustered"             # "stacked", "percentStacked" (con overlap = 100)
grafico.title = "Ventas por región"
grafico.y_axis.title = "Euros"
grafico.y_axis.numFmt = "#,##0"
datos = Reference(ws, min_col=2, max_col=4, min_row=1, max_row=6)      # Incluye la fila de cabecera
categorias = Reference(ws, min_col=1, min_row=2, max_row=6)
grafico.add_data(datos, titles_from_data=True)
grafico.set_categories(categorias)
grafico.width, grafico.height = 18, 9      # Centímetros
grafico.legend.position = "b"
ws.add_chart(grafico, "H2")

linea = LineChart()
linea.add_data(Reference(ws, min_col=5, min_row=1, max_row=13), titles_from_data=True)
linea.set_categories(Reference(ws, min_col=1, min_row=2, max_row=13))
linea.series[0].smooth = False
linea.series[0].marker.symbol = "circle"
linea.y_axis.axId = 200                    # Eje secundario combinado con el de barras:
grafico + linea                            # (grafico += linea tras configurar linea.y_axis.crosses = "max")

tarta = PieChart()
tarta.add_data(Reference(ws, min_col=2, min_row=1, max_row=5), titles_from_data=True)
tarta.set_categories(Reference(ws, min_col=1, min_row=2, max_row=5))
tarta.dataLabels = DataLabelList(); tarta.dataLabels.showPercent = True
```

Tipos disponibles: `BarChart`, `LineChart`, `AreaChart`, `PieChart`, `DoughnutChart`, `ScatterChart`, `BubbleChart`, `RadarChart`, `StockChart`, `SurfaceChart` (y versiones 3D).

> Desde OpenPyXL 3.1, los ejes pueden aparecer ocultos en algunas versiones de Excel; si ocurre, añada `grafico.x_axis.delete = False` y `grafico.y_axis.delete = False`.

---

## 15. Imágenes, comentarios e hipervínculos

```python
from openpyxl.drawing.image import Image as ImagenXL
from openpyxl.comments import Comment

logo = ImagenXL("logo.png"); logo.width, logo.height = 120, 40
ws.add_image(logo, "A1")

ws["B5"].comment = Comment("Incluye devoluciones de marzo", "Control de gestión", width=220, height=80)

ws["A20"].hyperlink = "https://www.ejemplo.es/informe"
ws["A20"].style = "Hyperlink"
ws["A21"].hyperlink = "#'Resumen'!A1"                # Enlace interno a otra hoja
ws["A21"].value = "Ir al resumen"
```

---

## 16. Nombres definidos

```python
from openpyxl.workbook.defined_name import DefinedName

wb.defined_names["IVA"] = DefinedName("IVA", attr_text="Parametros!$B$2")          # OpenPyXL ≥ 3.1
wb.defined_names["Objetivo"] = DefinedName("Objetivo", attr_text="Parametros!$B$3")
ws["G2"] = "=E2*(1+IVA)"

for nombre, definicion in wb.defined_names.items():
    print(nombre, definicion.attr_text, list(definicion.destinations))
```

---

## 17. Protección

```python
ws.protection.sheet = True
ws.protection.password = "clave-hoja"
ws.protection.formatColumns = False              # Permitir cambiar anchos (False = permitido)
from openpyxl.styles import Protection
for fila in ws["C2:C50"]:                        # Celdas editables dentro de una hoja protegida
    for c in fila:
        c.protection = Protection(locked=False)
wb.security.workbookPassword = "clave-libro" ; wb.security.lockStructure = True
```

> La protección de Excel evita modificaciones accidentales, **no es cifrado**. Para confidencialidad, cifre el archivo (por ejemplo, con `msoffcrypto-tool`) o controle el acceso al almacenamiento.

---

## 18. Configuración de impresión

```python
ws.page_setup.orientation = "landscape"
ws.page_setup.paperSize = ws.PAPERSIZE_A4
ws.page_setup.fitToWidth, ws.page_setup.fitToHeight = 1, 0     # Ajustar a 1 página de ancho
ws.sheet_properties.pageSetUpPr.fitToPage = True
ws.print_title_rows = "1:1"                                     # Repetir cabecera en cada página
ws.print_area = "A1:F200"
ws.page_margins.left = ws.page_margins.right = 0.5             # Pulgadas
ws.oddHeader.center.text = "Informe de ventas 2026"
ws.oddFooter.right.text = "Página &P de &N"
ws.print_options.horizontalCentered = True
```

---

## 19. Archivos grandes: modos de solo lectura y solo escritura

```python
# Lectura de archivos muy grandes (cientos de miles de filas): bajo consumo de memoria
wb = load_workbook("enorme.xlsx", read_only=True, data_only=True)
ws = wb["Datos"]
total = sum(fila[4] or 0 for fila in ws.iter_rows(min_row=2, values_only=True))
wb.close()                                        # Imprescindible en modo read_only

# Escritura en streaming
from openpyxl.cell import WriteOnlyCell
wb = Workbook(write_only=True)
ws = wb.create_sheet("Datos")
cabecera = [WriteOnlyCell(ws, value=t) for t in ("Fecha", "Importe")]
for c in cabecera:
    c.font = Font(bold=True)
ws.append(cabecera)
for registro in generador_de_filas():
    ws.append(registro)
wb.save("enorme.xlsx")
```

En modo `write_only` solo se puede añadir con `append` (no acceder a celdas arbitrarias) y el libro se guarda una única vez.

---

## 20. Integración con pandas

```python
import pandas as pd

df = pd.read_excel("ventas.xlsx", sheet_name="Datos", engine="openpyxl")
hojas = pd.read_excel("ventas.xlsx", sheet_name=None)               # Diccionario {hoja: DataFrame}

with pd.ExcelWriter("informe.xlsx", engine="openpyxl") as escritor:
    df.to_excel(escritor, sheet_name="Datos", index=False)
    resumen.to_excel(escritor, sheet_name="Resumen")
    ws = escritor.sheets["Datos"]                                     # Hoja de OpenPyXL para dar formato
    ws.freeze_panes = "A2"

with pd.ExcelWriter("plantilla.xlsx", engine="openpyxl", mode="a", if_sheet_exists="replace") as e:
    df.to_excel(e, sheet_name="Datos", index=False)                   # Sustituir una hoja de un libro existente
```

Patrón habitual: **pandas para calcular, OpenPyXL para presentar**.

---

## 21. Ejemplo práctico completo

**Generador del informe comercial mensual y consolidador de delegaciones**:

1. Genera los libros de cinco delegaciones (como los enviaría cada oficina), con algunos errores típicos: filas vacías, textos con espacios, importes como texto y fechas inválidas.
2. **Consolida** todos los libros leyéndolos en modo `read_only`, valida y normaliza cada fila y registra las incidencias.
3. Construye un **informe profesional** con:
   - **Portada** con índice de hipervínculos y fecha.
   - **Datos**: tabla de Excel con estilo, inmovilización, formatos numéricos, validación del estado y fila resaltada para pedidos pendientes.
   - **Resumen**: matriz delegación × mes con fórmulas `SUMIFS`, totales, variación sobre objetivo (nombre definido `Objetivo`), escala de color, barras de datos e iconos.
   - **Gráficos**: columnas por delegación y línea de evolución mensual.
   - **Parámetros** y **Incidencias**; protección del resumen y configuración de impresión.
4. Comprueba las fórmulas recalculando el libro con LibreOffice, si está instalado.

```python
"""
informe_comercial.py — Consolida libros de delegaciones y genera un informe de Excel profesional.
Requiere: pip install openpyxl     (opcional: LibreOffice para verificar las fórmulas)
Uso: python informe_comercial.py
"""
from __future__ import annotations

import random
import shutil
import subprocess
from dataclasses import dataclass
from datetime import date, datetime
import warnings
from pathlib import Path

from openpyxl import Workbook, load_workbook
from openpyxl.chart import BarChart, LineChart, Reference
from openpyxl.formatting.rule import CellIsRule, ColorScaleRule, DataBarRule, FormulaRule, IconSetRule
from openpyxl.styles import Alignment, Border, Font, NamedStyle, PatternFill, Side
from openpyxl.utils import get_column_letter
from openpyxl.workbook.defined_name import DefinedName
from openpyxl.worksheet.datavalidation import DataValidation
from openpyxl.worksheet.table import Table, TableStyleInfo

CARPETA_ENTRADA = Path("delegaciones")
INFORME = Path("informe_comercial_2026.xlsx")
DELEGACIONES = ["Madrid", "Barcelona", "Valencia", "Sevilla", "Bilbao"]
PRODUCTOS = {"Portátil": 899.0, "Monitor": 219.0, "Teclado": 49.0, "Silla": 189.0, "Impresora": 159.0}
ESTADOS = ["Entregado", "Enviado", "Pendiente", "Cancelado"]
MESES = ["Ene", "Feb", "Mar", "Abr", "May", "Jun", "Jul", "Ago", "Sep", "Oct", "Nov", "Dic"]
AZUL, AZUL_CLARO, GRIS = "1F4E78", "DDEBF7", "F2F2F2"
FINA = Side(style="thin", color="BFBFBF")


def eur(valor: float) -> str:
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# ============================================================ 1. Libros de entrada (simulados)
def generar_delegaciones() -> None:
    CARPETA_ENTRADA.mkdir(exist_ok=True)
    rng = random.Random(2026)
    for d, nombre in enumerate(DELEGACIONES):
        wb = Workbook()
        ws = wb.active
        ws.title = "Pedidos"
        ws.append(["Fecha", "Pedido", "Cliente", "Producto", "Unidades", "Precio", "Estado"])
        for i in range(rng.randint(70, 110)):
            mes = rng.randint(1, 10)
            producto = rng.choice(list(PRODUCTOS))
            fila = [date(2026, mes, rng.randint(1, 28)), f"{nombre[:3].upper()}-{i + 1:04d}",
                    f"Cliente {rng.randint(1, 40):02d}", producto, rng.randint(1, 12),
                    round(PRODUCTOS[producto] * rng.uniform(0.9, 1.05), 2), rng.choices(ESTADOS, [60, 20, 15, 5])[0]]
            if rng.random() < 0.04:
                fila[3] = f"  {producto.lower()} "                    # Texto sin normalizar
            if rng.random() < 0.03:
                fila[5] = f"{fila[5]:.2f}".replace(".", ",")          # Importe como texto
            if rng.random() < 0.02:
                fila[0] = "31/02/2026"                               # Fecha imposible
            ws.append(fila)
            if rng.random() < 0.02:
                ws.append([None] * 7)                                # Fila vacía
        wb.save(CARPETA_ENTRADA / f"pedidos_{nombre.lower()}.xlsx")


# ============================================================ 2. Consolidación y validación
@dataclass
class Pedido:
    delegacion: str
    fecha: date
    pedido: str
    cliente: str
    producto: str
    unidades: int
    precio: float
    estado: str

    @property
    def fila(self) -> list:
        return [self.delegacion, self.fecha, self.pedido, self.cliente, self.producto,
                self.unidades, self.precio, None, self.estado]       # Importe: fórmula en el informe


def normalizar(delegacion: str, registro: dict) -> Pedido:
    fecha = registro["Fecha"]
    if isinstance(fecha, datetime):
        fecha = fecha.date()
    elif isinstance(fecha, str):
        fecha = datetime.strptime(fecha, "%d/%m/%Y").date()          # ValueError si es imposible
    producto = str(registro["Producto"]).strip().capitalize()
    if producto not in PRODUCTOS:
        raise ValueError(f"producto desconocido «{producto}»")
    precio = registro["Precio"]
    if isinstance(precio, str):
        precio = float(precio.replace(".", "").replace(",", "."))
    estado = str(registro["Estado"]).strip().capitalize()
    if estado not in ESTADOS:
        raise ValueError(f"estado no válido «{estado}»")
    return Pedido(delegacion, fecha, str(registro["Pedido"]), str(registro["Cliente"]).strip(),
                  producto, int(registro["Unidades"]), float(precio), estado)


def consolidar() -> tuple[list[Pedido], list[list]]:
    pedidos, incidencias = [], []
    for ruta in sorted(CARPETA_ENTRADA.glob("pedidos_*.xlsx")):
        delegacion = ruta.stem.split("_", 1)[1].capitalize()
        wb = load_workbook(ruta, read_only=True, data_only=True)
        filas = wb["Pedidos"].iter_rows(values_only=True)
        cabecera = next(filas)
        for n, valores in enumerate(filas, start=2):
            if all(v is None for v in valores):
                incidencias.append([ruta.name, n, "Fila vacía", "Descartada"])
                continue
            registro = dict(zip(cabecera, valores))
            try:
                pedido = normalizar(delegacion, registro)
                if registro["Producto"] != pedido.producto or isinstance(registro["Precio"], str):
                    incidencias.append([ruta.name, n, "Formato corregido", f"{registro['Producto']!r} / {registro['Precio']!r}"])
                pedidos.append(pedido)
            except (ValueError, TypeError) as e:
                incidencias.append([ruta.name, n, "Fila rechazada", str(e)])
        wb.close()
    pedidos.sort(key=lambda p: (p.fecha, p.delegacion))
    return pedidos, incidencias


# ============================================================ 3. Informe
def registrar_estilos(wb: Workbook) -> None:
    encabezado = NamedStyle("encabezado")
    encabezado.font = Font(bold=True, color="FFFFFF")
    encabezado.fill = PatternFill("solid", start_color=AZUL)
    encabezado.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)
    encabezado.border = Border(bottom=Side(style="medium", color=AZUL))
    euros = NamedStyle("euros", number_format="#,##0.00 €")
    euros.border = Border(bottom=FINA)
    for estilo in (encabezado, euros):
        wb.add_named_style(estilo)


def titulo(ws, texto: str, subtitulo: str, ancho: int) -> None:
    ws.merge_cells(start_row=1, start_column=1, end_row=1, end_column=ancho)
    ws["A1"] = texto
    ws["A1"].font = Font(size=16, bold=True, color=AZUL)
    ws["A2"] = subtitulo
    ws["A2"].font = Font(italic=True, color="7F7F7F")
    ws.row_dimensions[1].height = 28


def hoja_parametros(wb: Workbook) -> None:
    ws = wb.create_sheet("Parametros")
    titulo(ws, "Parámetros del informe", "Modifique los valores en azul; el resumen se recalcula", 3)
    filas = [("Objetivo mensual por delegación", 14000, "#,##0 €"), ("IVA", 0.21, "0%"), ("Año", 2026, "0")]
    ws.append([])
    ws.append(["Parámetro", "Valor", "Nombre definido"])
    for c in ws[4]:
        c.style = "encabezado"
    for i, ((texto, valor, formato), nombre) in enumerate(zip(filas, ["Objetivo", "IVA", "Anio"]), start=5):
        ws.cell(i, 1, texto)
        celda = ws.cell(i, 2, valor)
        celda.number_format = formato
        celda.font = Font(color="0000FF", bold=True)                 # Convención: azul = dato editable
        ws.cell(i, 3, nombre).font = Font(color="7F7F7F")
        wb.defined_names[nombre] = DefinedName(nombre, attr_text=f"Parametros!$B${i}")
    ws.append([])
    ws.append(["Estados válidos"])
    ws.cell(ws.max_row, 1).font = Font(bold=True)
    inicio = ws.max_row + 1
    for e in ESTADOS:
        ws.append([e])
    wb.defined_names["Estados"] = DefinedName("Estados", attr_text=f"Parametros!$A${inicio}:$A${ws.max_row}")
    ws.column_dimensions["A"].width = 34
    ws.column_dimensions["B"].width = 14
    ws.column_dimensions["C"].width = 18


def hoja_datos(wb: Workbook, pedidos: list[Pedido]) -> int:
    ws = wb.create_sheet("Datos")
    cabecera = ["Delegación", "Fecha", "Pedido", "Cliente", "Producto", "Unidades", "Precio", "Importe", "Estado", "Mes"]
    ws.append(cabecera)
    for i, p in enumerate(pedidos, start=2):
        ws.append(p.fila + [None])
        ws.cell(i, 8, f"=F{i}*G{i}")                                    # Importe
        ws.cell(i, 10, f"=MONTH(B{i})")                                 # Mes (para SUMIFS)
        ws.cell(i, 2).number_format = "DD/MM/YYYY"
        ws.cell(i, 7).number_format = ws.cell(i, 8).number_format = "#,##0.00 €"
    ultima = ws.max_row

    tabla = Table(displayName="TablaPedidos", ref=f"A1:J{ultima}")
    tabla.tableStyleInfo = TableStyleInfo(name="TableStyleMedium2", showRowStripes=True)
    ws.add_table(tabla)
    ws.freeze_panes = "A2"
    for letra, ancho in zip("ABCDEFGHIJ", [13, 12, 12, 12, 12, 10, 12, 14, 12, 6]):
        ws.column_dimensions[letra].width = ancho

    validacion = DataValidation(type="list", formula1="=Estados", showErrorMessage=True,
                                errorTitle="Estado no válido", error="Seleccione un estado de la lista")
    ws.add_data_validation(validacion)
    validacion.add(f"I2:I{ultima + 500}")                               # También para filas futuras
    ws.conditional_formatting.add(f"A2:J{ultima}", FormulaRule(formula=['$I2="Pendiente"'],
                                  fill=PatternFill("solid", start_color="FFF2CC")))
    ws.conditional_formatting.add(f"A2:J{ultima}", FormulaRule(formula=['$I2="Cancelado"'],
                                  font=Font(color="A6A6A6", strike=True)))
    ws.print_title_rows = "1:1"
    ws.page_setup.orientation = "landscape"
    ws.page_setup.fitToWidth, ws.page_setup.fitToHeight = 1, 0
    ws.sheet_properties.pageSetUpPr.fitToPage = True
    return ultima


def hoja_resumen(wb: Workbook, meses: int) -> None:
    ws = wb.create_sheet("Resumen", 1)
    titulo(ws, "Ventas por delegación y mes", "Importe sin IVA de pedidos no cancelados · fórmulas SUMIFS sobre la hoja Datos", meses + 4)
    fila0 = 4
    ws.cell(fila0, 1, "Delegación")
    for m in range(1, meses + 1):
        ws.cell(fila0, m + 1, MESES[m - 1])
    ws.cell(fila0, meses + 2, "Total")
    ws.cell(fila0, meses + 3, "Media mensual")
    ws.cell(fila0, meses + 4, "% objetivo")
    for c in ws[fila0]:
        c.style = "encabezado"

    for i, delegacion in enumerate(DELEGACIONES, start=fila0 + 1):
        ws.cell(i, 1, delegacion).font = Font(bold=True)
        for m in range(1, meses + 1):
            celda = ws.cell(i, m + 1, f'=SUMIFS(TablaPedidos[Importe],TablaPedidos[Delegación],$A{i},'
                                     f'TablaPedidos[Mes],{m},TablaPedidos[Estado],"<>Cancelado")')
            celda.style = "euros"
        ult = get_column_letter(meses + 1)
        ws.cell(i, meses + 2, f"=SUM(B{i}:{ult}{i})").style = "euros"
        ws.cell(i, meses + 3, f"=AVERAGE(B{i}:{ult}{i})").style = "euros"
        pct = ws.cell(i, meses + 4, f"={get_column_letter(meses + 3)}{i}/Objetivo")
        pct.number_format = "0.0%"

    total = fila0 + len(DELEGACIONES) + 1
    ws.cell(total, 1, "Total").font = Font(bold=True)
    for col in range(2, meses + 4):
        letra = get_column_letter(col)
        celda = ws.cell(total, col, f"=SUM({letra}{fila0 + 1}:{letra}{total - 1})")
        celda.style = "euros"
        celda.font = Font(bold=True)
        celda.border = Border(top=Side(style="medium", color=AZUL), bottom=Side(style="double", color=AZUL))
    ws.cell(total, meses + 4, f"={get_column_letter(meses + 3)}{total}/(Objetivo*{len(DELEGACIONES)})").number_format = "0.0%"

    rango_meses = f"B{fila0 + 1}:{get_column_letter(meses + 1)}{total - 1}"
    ws.conditional_formatting.add(rango_meses, ColorScaleRule(start_type="min", start_color="F8696B",
                                  mid_type="percentile", mid_value=50, mid_color="FFEB84",
                                  end_type="max", end_color="63BE7B"))
    col_total = get_column_letter(meses + 2)
    ws.conditional_formatting.add(f"{col_total}{fila0 + 1}:{col_total}{total - 1}",
                                  DataBarRule(start_type="num", start_value=0, end_type="max", color="5B9BD5"))
    col_pct = get_column_letter(meses + 4)
    ws.conditional_formatting.add(f"{col_pct}{fila0 + 1}:{col_pct}{total}",
                                  IconSetRule("3TrafficLights1", "num", [0, 0.9, 1]))
    ws.conditional_formatting.add(f"{col_pct}{fila0 + 1}:{col_pct}{total}",
                                  CellIsRule(operator="lessThan", formula=["0.9"], font=Font(color="C00000", bold=True)))

    ws.column_dimensions["A"].width = 14
    for col in range(2, meses + 5):
        ws.column_dimensions[get_column_letter(col)].width = 13
    ws.freeze_panes = ws.cell(fila0 + 1, 2)
    nota = ws.cell(total + 2, 1, "Semáforo: verde ≥ 100 % del objetivo · ámbar 90–100 % · rojo < 90 %. "
                                 "El objetivo se modifica en la hoja Parametros.")
    nota.font = Font(italic=True, color="7F7F7F")

    # Gráficos
    barras = BarChart()
    barras.type, barras.grouping, barras.overlap = "col", "stacked", 100
    barras.title = "Ventas por delegación (apilado por mes)"
    barras.y_axis.numFmt = "#,##0"
    barras.y_axis.majorGridlines = None
    for m in range(1, meses + 1):
        barras.add_data(Reference(ws, min_col=m + 1, min_row=fila0, max_row=total - 1), titles_from_data=True)
    barras.set_categories(Reference(ws, min_col=1, min_row=fila0 + 1, max_row=total - 1))
    barras.width, barras.height = 16, 9
    barras.x_axis.delete = barras.y_axis.delete = False

    linea = LineChart()
    linea.title = "Evolución mensual total"
    linea.add_data(Reference(ws, min_col=2, max_col=meses + 1, min_row=total), from_rows=True, titles_from_data=False)
    linea.set_categories(Reference(ws, min_col=2, max_col=meses + 1, min_row=fila0))
    linea.series[0].marker.symbol = "circle"
    linea.series[0].smooth = False
    linea.series[0].graphicalProperties.line.solidFill = AZUL
    linea.series[0].graphicalProperties.line.width = 28000
    linea.legend = None
    linea.y_axis.numFmt = "#,##0"
    linea.width, linea.height = 16, 9
    linea.x_axis.delete = linea.y_axis.delete = False
    ws.add_chart(barras, f"A{total + 4}")
    ws.add_chart(linea, f"I{total + 4}")

    ws.protection.sheet = True                                        # Solo lectura (sin contraseña)
    ws.page_setup.orientation = "landscape"
    ws.page_setup.fitToWidth, ws.page_setup.fitToHeight = 1, 1
    ws.sheet_properties.pageSetUpPr.fitToPage = True
    ws.oddFooter.right.text = "Página &P de &N"


def hoja_incidencias(wb: Workbook, incidencias: list[list]) -> None:
    ws = wb.create_sheet("Incidencias")
    ws.append(["Archivo", "Fila", "Tipo", "Detalle"])
    for c in ws[1]:
        c.style = "encabezado"
    for fila in incidencias:
        ws.append(fila)
    ws.conditional_formatting.add(f"A2:D{max(2, ws.max_row)}", FormulaRule(formula=['$C2="Fila rechazada"'],
                                  fill=PatternFill("solid", start_color="FFC7CE")))
    for letra, ancho in zip("ABCD", [26, 6, 18, 60]):
        ws.column_dimensions[letra].width = ancho
    ws.auto_filter.ref = f"A1:D{max(2, ws.max_row)}"


def hoja_portada(wb: Workbook, pedidos: list[Pedido], incidencias: list[list]) -> None:
    ws = wb.active
    ws.title = "Portada"
    ws.sheet_view.showGridLines = False
    ws.sheet_properties.pageSetUpPr.fitToPage = True                   # Portada en una sola página
    ws.page_setup.fitToWidth = ws.page_setup.fitToHeight = 1
    ws.column_dimensions["A"].width = 3
    ws.column_dimensions["B"].width = 36
    ws.column_dimensions["C"].width = 50
    ws["B2"] = "Informe comercial 2026"
    ws["B2"].font = Font(size=22, bold=True, color=AZUL)
    ws["B3"] = f"Generado el {datetime.now():%d/%m/%Y a las %H:%M} · {len(pedidos)} pedidos de {len(DELEGACIONES)} delegaciones"
    ws["B3"].font = Font(color="7F7F7F")
    ws["B5"] = "Contenido"
    ws["B5"].font = Font(bold=True, size=12)
    secciones = [("Resumen", "Matriz delegación × mes, objetivos y gráficos"), ("Datos", "Pedidos consolidados (tabla filtrable)"),
                 ("Incidencias", f"{len(incidencias)} incidencias de validación"), ("Parametros", "Objetivo, IVA y listas")]
    for i, (hoja, descripcion) in enumerate(secciones, start=6):
        celda = ws.cell(i, 2, hoja)
        celda.hyperlink = f"#'{hoja}'!A1"
        celda.style = "Hyperlink"
        ws.cell(i, 3, descripcion)
    for fila in ws["B6:C9"]:
        for c in fila:
            c.border = Border(bottom=FINA)
    ws["B11"] = "Indicadores clave"
    ws["B11"].font = Font(bold=True, size=12)
    indicadores = [("Ventas netas (sin cancelados)", '=SUMIFS(TablaPedidos[Importe],TablaPedidos[Estado],"<>Cancelado")', "#,##0.00 €"),
                   ("Ventas con IVA", "=C12*(1+IVA)", "#,##0.00 €"),
                   ("Pedidos pendientes", '=COUNTIF(TablaPedidos[Estado],"Pendiente")', "0"),
                   ("Ticket medio", '=C12/COUNTIFS(TablaPedidos[Estado],"<>Cancelado")', "#,##0.00 €")]
    for i, (texto, formula, formato) in enumerate(indicadores, start=12):
        ws.cell(i, 2, texto)
        celda = ws.cell(i, 3, formula)
        celda.number_format = formato
        celda.font = Font(bold=True, size=12, color=AZUL)
        celda.alignment = Alignment(horizontal="left")
        celda.fill = PatternFill("solid", start_color=AZUL_CLARO)


def construir_informe(pedidos: list[Pedido], incidencias: list[list]) -> Path:
    wb = Workbook()
    registrar_estilos(wb)
    hoja_portada(wb, pedidos, incidencias)
    hoja_parametros(wb)
    hoja_datos(wb, pedidos)
    hoja_resumen(wb, meses=max(p.fecha.month for p in pedidos))
    hoja_incidencias(wb, incidencias)
    wb.move_sheet("Parametros", offset=len(wb.sheetnames))            # Al final
    for ws in wb.worksheets:
        ws.sheet_properties.tabColor = AZUL if ws.title in ("Portada", "Resumen") else "A6A6A6"
    wb.active = 0
    wb.properties.creator = "Departamento de Control de Gestión"
    wb.properties.title = "Informe comercial 2026"
    wb.save(INFORME)
    return INFORME


# ============================================================ 4. Verificación con LibreOffice (opcional)
def verificar_formulas(ruta: Path, pedidos: list[Pedido]) -> None:
    soffice = shutil.which("soffice") or shutil.which("libreoffice")
    if not soffice:
        print("LibreOffice no está instalado: se omite la verificación de fórmulas.")
        return
    destino = Path("recalculado")
    subprocess.run([soffice, "--headless", "--calc", "--convert-to", "xlsx", "--outdir", str(destino), str(ruta)],
                   check=True, capture_output=True, timeout=180)
    with warnings.catch_warnings():                                   # Avisos por extensiones añadidas por LibreOffice
        warnings.simplefilter("ignore", UserWarning)
        wb = load_workbook(destino / ruta.name, data_only=True)        # Valores calculados por LibreOffice
    ws = wb["Resumen"]
    fila_total = 4 + len(DELEGACIONES) + 1
    col_total = max(p.fecha.month for p in pedidos) + 2
    calculado = ws.cell(fila_total, col_total).value
    esperado = round(sum(p.unidades * p.precio for p in pedidos if p.estado != "Cancelado"), 2)
    errores = [c.coordinate for fila in ws.iter_rows() for c in fila
               if isinstance(c.value, str) and c.value.startswith(("#", "Err:"))]
    print(f"Verificación: total Excel {eur(calculado)} · total Python {eur(esperado)} · "
          f"{'coinciden' if abs(calculado - esperado) < 0.01 else 'NO coinciden'} · errores de fórmula: {errores or 'ninguno'}")
    print(f"Portada · ventas con IVA: {eur(wb['Portada']['C13'].value)} · pendientes: {wb['Portada']['C14'].value}")
    wb.close()


def main() -> int:
    generar_delegaciones()
    pedidos, incidencias = consolidar()
    rechazadas = sum(i[2] == "Fila rechazada" for i in incidencias)
    print(f"Pedidos válidos: {len(pedidos)} · incidencias: {len(incidencias)} (rechazadas: {rechazadas})")
    ruta = construir_informe(pedidos, incidencias)
    print(f"✔ Informe generado: {ruta}")
    verificar_formulas(ruta, pedidos)
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 22. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Las fórmulas devuelven `None` con `data_only=True` | El archivo nunca se calculó (generado por OpenPyXL). | Ábralo y guárdelo en Excel, o recalcule con LibreOffice. |
| Se perdieron las fórmulas del libro | Se guardó un libro cargado con `data_only=True`. | Cargue sin `data_only` para modificar y guardar. |
| `#NAME?` en Excel | Nombre de función en español, `;` como separador o función moderna sin `_xlfn.`. | Funciones en inglés con `,`; prefijo `_xlfn.` en funciones nuevas. |
| Excel indica que el archivo está dañado y lo repara | Tablas solapadas, `displayName` repetido o con espacios, cabeceras duplicadas o no textuales. | Revise la definición de tablas y rangos. |
| `TypeError: cannot set ... ` al modificar un estilo | Los estilos son inmutables. | Asigne un objeto nuevo: `c.font = Font(...)` o `c.font.copy(...)`. |
| `AttributeError: 'ReadOnlyWorksheet' object has no attribute …` | Operación no disponible en modo `read_only`. | Use el modo normal o solo `iter_rows`. |
| `PermissionError` al guardar | El archivo está abierto en Excel. | Ciérrelo o guarde con otro nombre. |
| `InvalidFileException: openpyxl does not support the old .xls file format` | Archivo `.xls`. | Conviértalo a `.xlsx` o léalo con `xlrd`/pandas. |
| Las macros desaparecen | Se cargó un `.xlsm` sin `keep_vba=True`. | `load_workbook(ruta, keep_vba=True)` y guardar como `.xlsm`. |
| Fechas leídas como números | Celda sin formato de fecha. | `openpyxl.utils.datetime.from_excel(valor)`. |
| Ceros a la izquierda perdidos | El valor se escribió como número. | Escriba texto y aplique `number_format = "@"`. |
| Proceso muy lento con muchas celdas | Estilos individuales o modo normal en archivos enormes. | `NamedStyle`, modo `write_only`/`read_only`. |

---

## 23. Buenas prácticas

- Separe **cálculo** (Python/pandas) de **presentación** (OpenPyXL), y deje en Excel fórmulas solo cuando el usuario deba poder recalcular o modificar supuestos.
- Use tablas de Excel y nombres definidos: las fórmulas son más legibles y resistentes a cambios.
- Defina estilos con nombre y una paleta corporativa reutilizable.
- Distinga visualmente datos editables (azul) de cálculos (negro) y proteja las hojas de cálculo.
- No guarde nunca sobre el original de entrada: genere un archivo nuevo.
- Para archivos grandes, utilice `read_only`/`write_only` y cierre los libros (`wb.close()`).
- Verifique los resultados: recalcule con LibreOffice en procesos automáticos y compare con los cálculos en Python.
- Documente en el propio libro (portada, notas) el origen de los datos y la fecha de generación.

---

## 24. Referencia rápida

```python
wb = Workbook() · wb = load_workbook(ruta, data_only=, read_only=, keep_vba=) · wb.save(ruta) · wb.close()
ws = wb.active · wb.create_sheet("Hoja", 0) · wb["Hoja"] · wb.sheetnames · wb.copy_worksheet(ws) · del wb["Hoja"]
ws["A1"] = valor · ws.cell(fila, col, valor) · ws.append([...]) · ws.max_row · ws.iter_rows(min_row=2, values_only=True)
ws["E2"] = "=SUM(B2:D2)" · ws.insert_rows(n) · ws.delete_cols(n) · ws.merge_cells("A1:D1") · ws.freeze_panes = "B2"
c.number_format = "#,##0.00 €" · c.font = Font(bold=True) · c.fill = PatternFill("solid", start_color="1F4E78")
c.border = Border(bottom=Side(style="thin")) · c.alignment = Alignment(horizontal="center", wrap_text=True) · NamedStyle
ws.column_dimensions["A"].width · ws.row_dimensions[1].height · get_column_letter(n) · column_index_from_string("C")
ws.add_table(Table(displayName=, ref=)) · ws.auto_filter.ref · ws.conditional_formatting.add(rango, regla)
DataValidation(type="list", formula1='"a,b"') · BarChart/LineChart/PieChart + Reference + add_data/set_categories + ws.add_chart(g, "H2")
wb.defined_names["N"] = DefinedName("N", attr_text="Hoja!$B$2") · ws.protection.sheet = True · ws.page_setup.orientation
```

---

## 25. Ejercicios propuestos

1. Añada al informe una hoja por delegación generada a partir de una plantilla (`copy_worksheet`) con sus pedidos y su propio gráfico.
2. Implemente una función que compare dos versiones de un libro y genere una hoja con las celdas modificadas (valor anterior y nuevo).
3. Cree una plantilla de presupuesto con celdas editables desbloqueadas, validaciones y una hoja protegida; después, rellénela automáticamente desde un CSV.
4. Lea un libro de 200 000 filas en modo `read_only` y compare tiempo y memoria con el modo normal y con `pandas.read_excel`.
5. Genere con pandas una tabla dinámica de ventas y aplíquele con OpenPyXL formato condicional, totales y un gráfico.

---

## 26. Referencias

- Documentación oficial: https://openpyxl.readthedocs.io/
- Tutorial: https://openpyxl.readthedocs.io/en/stable/tutorial.html
- Gráficos: https://openpyxl.readthedocs.io/en/stable/charts/introduction.html
- Formato condicional: https://openpyxl.readthedocs.io/en/stable/formatting.html
- Rendimiento y modos optimizados: https://openpyxl.readthedocs.io/en/stable/optimized.html
