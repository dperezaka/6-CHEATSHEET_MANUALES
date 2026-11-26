# Manual de uso: PyWin32

> **Categoría:** Automatización y sistema · **Utilidad:** Interactuar con funciones de Windows
> **Tipo:** Librería externa · **Sistema:** solo Windows

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Automatización de Office con COM](#4-automatización-de-office-con-com)
5. [Excel](#5-excel)
6. [Word](#6-word)
7. [Outlook](#7-outlook)
8. [Ventanas y mensajes](#8-ventanas-y-mensajes)
9. [Portapapeles](#9-portapapeles)
10. [Archivos, carpetas y accesos directos](#10-archivos-carpetas-y-accesos-directos)
11. [Procesos, servicios y sistema](#11-procesos-servicios-y-sistema)
12. [Registro de Windows](#12-registro-de-windows)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**PyWin32** (*Python for Windows Extensions*) da acceso desde Python a la **API de Windows (Win32)** y a la tecnología **COM**, que permite controlar aplicaciones como Excel, Word, Outlook, PowerPoint o Access.

**Casos de uso habituales:**
- Automatizar Microsoft Office: abrir libros de Excel, ejecutar macros, generar informes, convertir documentos a PDF, enviar correos con Outlook.
- Gestionar ventanas: buscar, activar, mover, minimizar, cerrar.
- Portapapeles avanzado (imágenes, HTML).
- Crear accesos directos, consultar carpetas especiales del sistema.
- Gestionar procesos, servicios y el registro de Windows.
- Crear servicios de Windows escritos en Python.

**¿Cuándo usar PyWin32 frente a otras librerías?**

| Necesidad | Opción recomendada |
|---|---|
| Leer/escribir archivos `.xlsx` sin abrir Excel | openpyxl / pandas |
| Crear `.docx` sin Word | python-docx |
| Recalcular fórmulas, ejecutar macros VBA, actualizar tablas dinámicas o conexiones | **PyWin32 (Excel COM)** |
| Convertir Word/Excel/PowerPoint a PDF con fidelidad exacta | **PyWin32** |
| Enviar o leer correos con la cuenta de Outlook del usuario | **PyWin32 (Outlook COM)** |
| Controlar ventanas y funciones nativas de Windows | **PyWin32** |

> La automatización COM **requiere tener Office instalado** y abre realmente las aplicaciones (de forma visible u oculta). No está pensada para servidores sin interfaz.

---

## 2. Instalación e importación

```bash
pip install pywin32
```

Si aparecen errores de DLL tras la instalación, ejecute (como administrador):

```bash
python -m pywin32_postinstall -install
```

Módulos principales:

```python
import win32com.client as win32        # Automatización COM (Office)
import win32gui                        # Ventanas
import win32con                        # Constantes de Windows
import win32api                        # Funciones generales de la API
import win32clipboard                  # Portapapeles
import win32process                    # Procesos
import win32service, win32serviceutil  # Servicios
import winreg                          # Registro (librería estándar)
import pythoncom                       # Inicialización COM en hilos
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| **COM** | Tecnología de Windows que permite a un programa controlar otro mediante objetos. |
| **ProgID** | Identificador de una aplicación COM: `"Excel.Application"`, `"Word.Application"`, `"Outlook.Application"`. |
| `Dispatch` | Crea o conecta con un objeto COM (enlace dinámico). |
| `gencache.EnsureDispatch` | Igual, pero genera envoltorios con constantes y autocompletado (enlace temprano). |
| `DispatchEx` | Crea siempre una **instancia nueva** de la aplicación. |
| `GetActiveObject` | Conecta con una instancia ya abierta. |
| Modelo de objetos | Jerarquía de la aplicación (Excel: `Application → Workbooks → Workbook → Worksheets → Range`). Es el mismo que se usa en VBA. |
| **HWND** | Identificador numérico de una ventana. |
| Constantes | Valores numéricos de Office/Windows (`xlUp = -4162`, `wdFormatPDF = 17`). |

> **Regla de oro:** lo que se puede hacer con VBA se puede hacer con PyWin32 usando el mismo modelo de objetos. La grabadora de macros de Excel/Word es una excelente forma de descubrir qué objetos y métodos usar.

---

## 4. Automatización de Office con COM

### 4.1 Iniciar una aplicación

```python
import win32com.client as win32

excel = win32.Dispatch("Excel.Application")                 # Usa una instancia existente o crea una
excel = win32.DispatchEx("Excel.Application")               # Siempre una instancia nueva (recomendado en scripts)
excel = win32.gencache.EnsureDispatch("Excel.Application")  # Con constantes y nombres en caché
```

### 4.2 Conectar con una instancia abierta

```python
excel = win32.GetActiveObject("Excel.Application")
for libro in excel.Workbooks:
    print(libro.Name)
```

### 4.3 Opciones habituales

```python
excel.Visible = False             # Ejecutar en segundo plano
excel.DisplayAlerts = False       # No mostrar avisos (sobrescribir, guardar cambios…)
excel.ScreenUpdating = False      # Más rápido
```

### 4.4 Constantes

```python
from win32com.client import constants
excel = win32.gencache.EnsureDispatch("Excel.Application")
constants.xlUp           # -4162 (requiere EnsureDispatch)
```

O defínalas manualmente (más robusto):

```python
XL_UP = -4162
XL_OPENXML_WORKBOOK = 51     # .xlsx
XL_TYPE_PDF = 0
WD_FORMAT_PDF = 17
OL_MAIL_ITEM = 0
```

### 4.5 Cerrar correctamente (fundamental)

Si el script falla sin cerrar la aplicación, queda un proceso `EXCEL.EXE` oculto en memoria. Use siempre `try/finally`:

```python
excel = win32.DispatchEx("Excel.Application")
excel.Visible = False
excel.DisplayAlerts = False
try:
    libro = excel.Workbooks.Open(r"C:\Datos\ventas.xlsx")
    # ... trabajo ...
    libro.Close(SaveChanges=True)
finally:
    excel.Quit()
    del excel
```

Gestor de contexto reutilizable:

```python
from contextlib import contextmanager


@contextmanager
def aplicacion_office(progid: str, visible: bool = False):
    app = win32.DispatchEx(progid)
    app.Visible = visible
    app.DisplayAlerts = False
    try:
        yield app
    finally:
        app.Quit()

with aplicacion_office("Excel.Application") as excel:
    ...
```

### 4.6 Rutas

COM requiere **rutas absolutas** con barras invertidas:

```python
from pathlib import Path
ruta = str(Path("datos/ventas.xlsx").resolve())
```

---

## 5. Excel

### 5.1 Abrir, crear y guardar libros

```python
libro = excel.Workbooks.Open(r"C:\Datos\ventas.xlsx")
libro = excel.Workbooks.Open(r"C:\Datos\ventas.xlsx", ReadOnly=True)
libro = excel.Workbooks.Add()

libro.Save()
libro.SaveAs(r"C:\Salida\informe.xlsx", FileFormat=51)     # 51 = .xlsx
libro.SaveAs(r"C:\Salida\informe.xlsm", FileFormat=52)     # 52 = con macros
libro.SaveAs(r"C:\Salida\datos.csv", FileFormat=6)         # 6 = CSV
libro.Close(SaveChanges=False)
```

### 5.2 Hojas

```python
hoja = libro.Worksheets("Ventas")
hoja = libro.Worksheets(1)                  # Los índices COM empiezan en 1
hoja = libro.ActiveSheet

nueva = libro.Worksheets.Add()
nueva.Name = "Resumen"
hoja.Copy(After=libro.Worksheets(libro.Worksheets.Count))
libro.Worksheets("Temporal").Delete()

for h in libro.Worksheets:
    print(h.Name)
```

### 5.3 Leer y escribir celdas

```python
hoja.Range("A1").Value = "Informe de ventas"
hoja.Cells(2, 1).Value = 1250.75            # Fila 2, columna 1
valor = hoja.Range("B5").Value

hoja.Range("C2").Formula = "=SUM(B2:B100)"
hoja.Range("D2").FormulaLocal = "=SUMA(B2:B100)"     # Fórmula en el idioma de Excel
```

### 5.4 Leer y escribir rangos completos (rápido)

Leer y escribir celda a celda es muy lento. Trabaje con rangos completos:

```python
datos = hoja.Range("A1:D100").Value          # Tupla de tuplas

filas = [
    ("Producto", "Unidades", "Precio"),
    ("Teclado", 10, 29.9),
    ("Ratón", 25, 14.5),
]
hoja.Range(hoja.Cells(1, 1), hoja.Cells(len(filas), len(filas[0]))).Value = filas
```

### 5.5 Última fila y columna con datos

```python
XL_UP, XL_TO_LEFT = -4162, -4159
ultima_fila = hoja.Cells(hoja.Rows.Count, 1).End(XL_UP).Row
ultima_col = hoja.Cells(1, hoja.Columns.Count).End(XL_TO_LEFT).Column
rango_usado = hoja.UsedRange
```

### 5.6 Formato

```python
cab = hoja.Range("A1:D1")
cab.Font.Bold = True
cab.Font.Color = 0xFFFFFF                    # Formato BGR: 0xBBGGRR
cab.Interior.Color = 0x784E1F                # Azul #1F4E78 en BGR
cab.HorizontalAlignment = -4108              # xlCenter

hoja.Range("C2:C100").NumberFormat = '#,##0.00 "€"'          # Sintaxis inglesa (siempre válida)
hoja.Range("D2:D100").NumberFormatLocal = "#.##0,00 €"        # Sintaxis del idioma de Excel
hoja.Columns("A:D").AutoFit()
hoja.Range("A1:D100").Borders.LineStyle = 1
```

> Excel usa colores en formato **BGR**. Conversión: `def rgb(r, g, b): return r + g * 256 + b * 65536`.

### 5.7 Operaciones exclusivas de Excel (motivo principal para usar COM)

```python
libro.RefreshAll()                                   # Actualizar conexiones y tablas dinámicas
excel.CalculateUntilAsyncQueriesDone()               # Esperar a las consultas
excel.Calculate()                                    # Recalcular fórmulas

excel.Application.Run("MiLibro.xlsm!Modulo1.GenerarInforme")     # Ejecutar una macro VBA
excel.Application.Run("Actualizar", "2026", 11)                  # Macro con argumentos

libro.ExportAsFixedFormat(0, r"C:\Salida\informe.pdf")           # Exportar a PDF
hoja.ExportAsFixedFormat(0, r"C:\Salida\hoja.pdf")
hoja.PrintOut()                                                  # Imprimir
```

### 5.8 Integración con pandas

```python
import pandas as pd

valores = hoja.UsedRange.Value
df = pd.DataFrame(valores[1:], columns=valores[0])

# DataFrame → Excel
datos = [df.columns.tolist()] + df.astype(object).where(df.notna(), None).values.tolist()
hoja.Range(hoja.Cells(1, 1), hoja.Cells(len(datos), len(datos[0]))).Value = datos
```

---

## 6. Word

### 6.1 Abrir y crear documentos

```python
word = win32.DispatchEx("Word.Application")
word.Visible = False
word.DisplayAlerts = 0

doc = word.Documents.Open(r"C:\Plantillas\contrato.docx")
doc = word.Documents.Add()
doc = word.Documents.Add(Template=r"C:\Plantillas\carta.dotx")
```

### 6.2 Escribir texto

```python
rango = doc.Content
rango.Text = "Título del documento\n"
rango.InsertAfter("Primer párrafo de contenido.\n")

parrafo = doc.Paragraphs(1)
parrafo.Range.Font.Size = 18
parrafo.Range.Font.Bold = True
parrafo.Style = "Título 1"           # Nombre del estilo en el idioma de Word ("Heading 1" en inglés)
```

### 6.3 Buscar y reemplazar (plantillas)

```python
def reemplazar(doc, buscar: str, reemplazo: str) -> None:
    rango = doc.Content
    rango.Find.Execute(
        FindText=buscar, ReplaceWith=reemplazo,
        Replace=2,                  # wdReplaceAll
        MatchCase=True, Forward=True, Wrap=1,
    )

datos = {"{{NOMBRE}}": "Ana López", "{{FECHA}}": "24/11/2026", "{{IMPORTE}}": "1.250,00 €"}
for clave, valor in datos.items():
    reemplazar(doc, clave, valor)
```

> `Find.Execute` admite como máximo 255 caracteres en el texto de reemplazo.

### 6.4 Tablas

```python
tabla = doc.Tables.Add(doc.Paragraphs(doc.Paragraphs.Count).Range, NumRows=3, NumColumns=3)
tabla.Cell(1, 1).Range.Text = "Producto"
tabla.Borders.Enable = True
tabla.Rows(1).Range.Font.Bold = True
```

### 6.5 Guardar y convertir

```python
WD_FORMAT_PDF = 17
doc.SaveAs2(r"C:\Salida\contrato_ana.docx")
doc.SaveAs2(r"C:\Salida\contrato_ana.pdf", FileFormat=WD_FORMAT_PDF)
doc.ExportAsFixedFormat(r"C:\Salida\contrato.pdf", 17)
doc.Close(SaveChanges=False)
word.Quit()
```

### 6.6 Convertir por lotes Word → PDF

```python
from pathlib import Path

def word_a_pdf(carpeta: Path) -> None:
    word = win32.DispatchEx("Word.Application")
    word.Visible = False
    try:
        for docx in carpeta.glob("*.docx"):
            if docx.name.startswith("~$"):
                continue
            doc = word.Documents.Open(str(docx.resolve()), ReadOnly=True)
            try:
                doc.SaveAs2(str(docx.with_suffix(".pdf").resolve()), FileFormat=17)
            finally:
                doc.Close(False)
    finally:
        word.Quit()
```

PowerPoint funciona igual (`"PowerPoint.Application"`, `Presentations.Open`, `SaveAs(ruta, 32)` para PDF).

---

## 7. Outlook

### 7.1 Enviar un correo

```python
outlook = win32.Dispatch("Outlook.Application")
correo = outlook.CreateItem(0)              # 0 = olMailItem

correo.To = "cliente@ejemplo.com"
correo.CC = "comercial@miempresa.com"
correo.BCC = "archivo@miempresa.com"
correo.Subject = "Informe mensual de ventas"
correo.HTMLBody = """
<p>Buenos días:</p>
<p>Adjuntamos el <b>informe de ventas</b> de noviembre.</p>
<p>Un saludo,<br>Ana López</p>
"""
correo.Attachments.Add(r"C:\Salida\informe.pdf")

correo.Display()      # Mostrar para revisar antes de enviar
# correo.Send()       # Enviar directamente
```

> Con `HTMLBody` la firma predeterminada no se añade automáticamente. Para conservarla, llame primero a `correo.Display()`, lea `correo.HTMLBody` (que ya contiene la firma) y anteponga su texto.

### 7.2 Elegir la cuenta de envío

```python
for cuenta in outlook.Session.Accounts:
    print(cuenta.SmtpAddress)

cuenta = next(c for c in outlook.Session.Accounts if c.SmtpAddress == "ventas@miempresa.com")
correo.SendUsingAccount = cuenta
```

> Si la asignación no tiene efecto (ocurre en algunas versiones con `Dispatch`), use la alternativa de bajo nivel: `correo._oleobj_.Invoke(64209, 0, 8, 0, cuenta)` (64209 es el identificador de la propiedad `SendUsingAccount`).

### 7.3 Leer la bandeja de entrada

```python
ns = outlook.GetNamespace("MAPI")
bandeja = ns.GetDefaultFolder(6)            # 6 = Bandeja de entrada

mensajes = bandeja.Items
mensajes.Sort("[ReceivedTime]", True)        # Más recientes primero

for msg in list(mensajes)[:10]:
    if msg.Class != 43:                      # 43 = MailItem (excluye citas, informes…)
        continue
    print(msg.ReceivedTime, "|", msg.SenderName, "|", msg.Subject, "| no leído" if msg.UnRead else "")
```

### 7.4 Filtrar mensajes

```python
no_leidos = bandeja.Items.Restrict("[UnRead] = True")
de_cliente = bandeja.Items.Restrict("[SenderEmailAddress] = 'cliente@ejemplo.com'")
recientes = bandeja.Items.Restrict("[ReceivedTime] >= '11/01/2026 00:00'")    # Formato mm/dd/aaaa
```

### 7.5 Guardar adjuntos

```python
from pathlib import Path

destino = Path(r"C:\Facturas\Recibidas")
destino.mkdir(parents=True, exist_ok=True)

for msg in bandeja.Items.Restrict("[UnRead] = True"):
    if "factura" not in (msg.Subject or "").lower():
        continue
    for adjunto in msg.Attachments:
        if adjunto.FileName.lower().endswith(".pdf"):
            adjunto.SaveAsFile(str(destino / adjunto.FileName))
    msg.UnRead = False
```

### 7.6 Carpetas, mover mensajes y calendario

```python
procesados = bandeja.Folders("Procesados")           # Subcarpeta existente
msg.Move(procesados)

calendario = ns.GetDefaultFolder(9)                   # 9 = Calendario
cita = outlook.CreateItem(1)                          # 1 = olAppointmentItem
cita.Subject = "Revisión de resultados"
cita.Start = "2026-12-01 10:00"
cita.Duration = 60
cita.Location = "Sala 2"
cita.Save()
```

| Carpeta | Código |
|---|---|
| Bandeja de entrada | 6 |
| Elementos enviados | 5 |
| Elementos eliminados | 3 |
| Borradores | 16 |
| Calendario | 9 |
| Contactos | 10 |
| Tareas | 13 |

> Outlook puede mostrar avisos de seguridad cuando un programa externo accede a direcciones o envía correos, según la configuración del antivirus y de la organización.

---

## 8. Ventanas y mensajes

### 8.1 Cuadros de mensaje nativos

```python
import win32api, win32con

win32api.MessageBox(0, "Proceso completado correctamente.", "Automatización", win32con.MB_ICONINFORMATION)

r = win32api.MessageBox(0, "¿Desea continuar?", "Confirmar", win32con.MB_YESNO | win32con.MB_ICONQUESTION)
if r == win32con.IDYES:
    ...
```

### 8.2 Buscar ventanas

```python
import win32gui

hwnd = win32gui.FindWindow(None, "Sin título: Bloc de notas")   # Por título exacto
hwnd = win32gui.FindWindow("Notepad", None)                      # Por clase
hwnd = win32gui.GetForegroundWindow()                            # Ventana activa
print(win32gui.GetWindowText(hwnd), win32gui.GetClassName(hwnd))
```

Enumerar todas las ventanas visibles:

```python
def listar_ventanas() -> list[tuple[int, str]]:
    ventanas = []
    def callback(hwnd, _):
        if win32gui.IsWindowVisible(hwnd) and win32gui.GetWindowText(hwnd):
            ventanas.append((hwnd, win32gui.GetWindowText(hwnd)))
    win32gui.EnumWindows(callback, None)
    return ventanas

def buscar_ventana(texto: str) -> int | None:
    return next((h for h, t in listar_ventanas() if texto.lower() in t.lower()), None)
```

### 8.3 Manipular ventanas

```python
win32gui.ShowWindow(hwnd, win32con.SW_RESTORE)        # Restaurar
win32gui.ShowWindow(hwnd, win32con.SW_MAXIMIZE)
win32gui.ShowWindow(hwnd, win32con.SW_MINIMIZE)
win32gui.SetForegroundWindow(hwnd)                    # Traer al frente
win32gui.MoveWindow(hwnd, 0, 0, 1280, 800, True)      # x, y, ancho, alto
izq, arr, der, abj = win32gui.GetWindowRect(hwnd)
win32gui.PostMessage(hwnd, win32con.WM_CLOSE, 0, 0)   # Cerrar
```

> `SetForegroundWindow` puede fallar si el script no es el proceso activo (restricción de Windows). Un truco habitual es enviar antes una pulsación de `Alt`:
> `win32api.keybd_event(win32con.VK_MENU, 0, 0, 0); win32api.keybd_event(win32con.VK_MENU, 0, win32con.KEYEVENTF_KEYUP, 0)`

### 8.4 Obtener el proceso de una ventana

```python
import win32process
_, pid = win32process.GetWindowThreadProcessId(hwnd)
```

---

## 9. Portapapeles

```python
import win32clipboard as cb

# Copiar texto (Unicode)
cb.OpenClipboard()
try:
    cb.EmptyClipboard()
    cb.SetClipboardText("Texto con tildes y ñ", cb.CF_UNICODETEXT)
finally:
    cb.CloseClipboard()

# Leer texto
cb.OpenClipboard()
try:
    if cb.IsClipboardFormatAvailable(cb.CF_UNICODETEXT):
        texto = cb.GetClipboardData(cb.CF_UNICODETEXT)
finally:
    cb.CloseClipboard()
```

### 9.1 Copiar una imagen

```python
import io
from PIL import Image

def copiar_imagen(ruta: str) -> None:
    salida = io.BytesIO()
    Image.open(ruta).convert("RGB").save(salida, "BMP")
    datos = salida.getvalue()[14:]           # Quitar la cabecera del archivo BMP
    cb.OpenClipboard()
    try:
        cb.EmptyClipboard()
        cb.SetClipboardData(cb.CF_DIB, datos)
    finally:
        cb.CloseClipboard()
```

### 9.2 Lista de archivos copiados en el Explorador

```python
cb.OpenClipboard()
try:
    if cb.IsClipboardFormatAvailable(cb.CF_HDROP):
        archivos = cb.GetClipboardData(cb.CF_HDROP)     # Tupla de rutas
finally:
    cb.CloseClipboard()
```

---

## 10. Archivos, carpetas y accesos directos

### 10.1 Carpetas especiales

```python
from win32com.shell import shell, shellcon

escritorio = shell.SHGetFolderPath(0, shellcon.CSIDL_DESKTOPDIRECTORY, None, 0)
documentos = shell.SHGetFolderPath(0, shellcon.CSIDL_PERSONAL, None, 0)
appdata = shell.SHGetFolderPath(0, shellcon.CSIDL_APPDATA, None, 0)
inicio = shell.SHGetFolderPath(0, shellcon.CSIDL_STARTUP, None, 0)       # Inicio automático
```

> Útil cuando las carpetas del usuario están redirigidas (OneDrive, red corporativa) y `Path.home() / "Documents"` no es correcto.

### 10.2 Crear accesos directos

```python
def crear_acceso_directo(destino: str, ruta_lnk: str, argumentos: str = "",
                         carpeta_trabajo: str = "", icono: str = "", descripcion: str = "") -> None:
    shell_ws = win32.Dispatch("WScript.Shell")
    acceso = shell_ws.CreateShortCut(ruta_lnk)
    acceso.TargetPath = destino
    acceso.Arguments = argumentos
    acceso.WorkingDirectory = carpeta_trabajo
    acceso.Description = descripcion
    if icono:
        acceso.IconLocation = icono
    acceso.Save()

# Ejecutar un script de Python al iniciar sesión
import sys
crear_acceso_directo(
    destino=sys.executable.replace("python.exe", "pythonw.exe"),
    ruta_lnk=str(Path(inicio) / "Asistente.lnk"),
    argumentos=r'"C:\scripts\asistente.py"',
    carpeta_trabajo=r"C:\scripts",
)
```

### 10.3 Enviar a la papelera

```python
from win32com.shell import shell, shellcon

def a_papelera(ruta: str) -> None:
    shell.SHFileOperation((0, shellcon.FO_DELETE, ruta, None,
                           shellcon.FOF_ALLOWUNDO | shellcon.FOF_NOCONFIRMATION | shellcon.FOF_SILENT,
                           None, None))
```

### 10.4 Atributos de archivo

```python
win32api.SetFileAttributes(r"C:\Datos\config.ini", win32con.FILE_ATTRIBUTE_HIDDEN)
win32api.SetFileAttributes(r"C:\Datos\config.ini", win32con.FILE_ATTRIBUTE_NORMAL)
attrs = win32api.GetFileAttributes(r"C:\Datos\config.ini")
es_solo_lectura = bool(attrs & win32con.FILE_ATTRIBUTE_READONLY)
```

### 10.5 Abrir e imprimir con la aplicación asociada

```python
win32api.ShellExecute(0, "open", r"C:\Salida\informe.pdf", None, ".", 1)
win32api.ShellExecute(0, "print", r"C:\Salida\informe.pdf", None, ".", 0)
```

---

## 11. Procesos, servicios y sistema

### 11.1 Información del sistema

```python
win32api.GetComputerName()
win32api.GetUserName()
win32api.GetSystemMetrics(win32con.SM_CXSCREEN), win32api.GetSystemMetrics(win32con.SM_CYSCREEN)
win32api.GetLogicalDriveStrings().split("\x00")[:-1]          # ['C:\\', 'D:\\']
```

### 11.2 Procesos con WMI

```python
wmi = win32.GetObject("winmgmts:")
for p in wmi.InstancesOf("Win32_Process"):
    if p.Name.lower() == "excel.exe":
        print(p.ProcessId, p.Name, p.CommandLine)
```

Consultas WMI útiles:

```python
for disco in wmi.ExecQuery("SELECT * FROM Win32_LogicalDisk WHERE DriveType = 3"):
    print(disco.DeviceID, int(disco.FreeSpace) // 1024**3, "GB libres")

for so in wmi.ExecQuery("SELECT * FROM Win32_OperatingSystem"):
    print(so.Caption, so.Version, so.LastBootUpTime)
```

### 11.3 Terminar procesos

```python
import win32process

def terminar_proceso(pid: int) -> None:
    manejador = win32api.OpenProcess(win32con.PROCESS_TERMINATE, False, pid)
    try:
        win32api.TerminateProcess(manejador, 0)
    finally:
        win32api.CloseHandle(manejador)
```

> Útil para cerrar procesos de Excel huérfanos que hayan quedado de ejecuciones fallidas (con precaución: también cerraría los del usuario).

### 11.4 Servicios de Windows

```python
import win32serviceutil

win32serviceutil.QueryServiceStatus("Spooler")[1]     # 4 = en ejecución, 1 = detenido
win32serviceutil.StopService("Spooler")               # Requiere administrador
win32serviceutil.StartService("Spooler")
win32serviceutil.RestartService("Spooler")
```

### 11.5 Bloquear, apagar y reiniciar

```python
import ctypes
ctypes.windll.user32.LockWorkStation()                        # Bloquear sesión

win32api.InitiateSystemShutdown(None, "Reinicio programado", 60, True, True)   # Requiere privilegios
win32api.AbortSystemShutdown(None)
```

### 11.6 Registro de eventos de Windows

```python
import win32evtlogutil, win32evtlog

win32evtlogutil.ReportEvent("MiAplicacion", 1000,
                            eventType=win32evtlog.EVENTLOG_INFORMATION_TYPE,
                            strings=["Copia de seguridad completada"])
```

---

## 12. Registro de Windows

El módulo estándar `winreg` (incluido en Python) permite leer y modificar el registro.

```python
import winreg

# Leer un valor
with winreg.OpenKey(winreg.HKEY_CURRENT_USER, r"Control Panel\International") as clave:
    formato_fecha, _ = winreg.QueryValueEx(clave, "sShortDate")
    print(formato_fecha)                # dd/MM/yyyy

# Escribir configuración propia
with winreg.CreateKey(winreg.HKEY_CURRENT_USER, r"Software\MiEmpresa\MiApp") as clave:
    winreg.SetValueEx(clave, "UltimaEjecucion", 0, winreg.REG_SZ, "2026-11-24 22:00")
    winreg.SetValueEx(clave, "Version", 0, winreg.REG_DWORD, 3)

# Programas que se inician con el usuario
with winreg.OpenKey(winreg.HKEY_CURRENT_USER,
                    r"Software\Microsoft\Windows\CurrentVersion\Run") as clave:
    i = 0
    while True:
        try:
            nombre, valor, _ = winreg.EnumValue(clave, i)
            print(f"{nombre}: {valor}")
            i += 1
        except OSError:
            break
```

> ⚠ Modificar claves del sistema (`HKEY_LOCAL_MACHINE`) puede dejar Windows inestable. Limítese a claves de su propia aplicación en `HKEY_CURRENT_USER\Software\...` y haga una copia de seguridad antes de cambios importantes.

---

## 13. Ejemplo práctico completo

**Informe mensual automatizado**: abre un libro de Excel con conexiones de datos, lo actualiza, ejecuta una macro opcional, lee el resumen, exporta la hoja a PDF y prepara un correo en Outlook con el PDF adjunto y un resumen en el cuerpo. Gestiona errores, cierra Office siempre y registra cada paso.

```python
"""
Informe mensual con Excel + Outlook (PyWin32).
Requisitos: Windows, Microsoft Office, pip install pywin32
"""
import logging
from datetime import date
from html import escape
from pathlib import Path

import pythoncom
import pywintypes
import win32com.client as win32

# --------------------------------------------------------------- Configuración
RAIZ = Path(__file__).resolve().parent
LIBRO = RAIZ / "datos" / "ventas.xlsx"           # Libro con conexiones/tablas dinámicas
HOJA_INFORME = "Informe"
RANGO_RESUMEN = "A1:C10"                          # Tabla de resumen en la hoja
MACRO = None                                      # p. ej. "ventas.xlsm!Modulo1.Preparar"
SALIDA = RAIZ / "salida"
DESTINATARIOS = "direccion@miempresa.com"
CC = "comercial@miempresa.com"
ENVIAR_DIRECTAMENTE = False                       # False = mostrar el correo para revisarlo

XL_TYPE_PDF = 0
XL_QUALITY_STANDARD = 0
OL_MAIL_ITEM = 0
MESES = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto",
         "septiembre", "octubre", "noviembre", "diciembre"]

logging.basicConfig(
    level=logging.INFO, format="%(asctime)s | %(levelname)-7s | %(message)s",
    handlers=[logging.FileHandler(RAIZ / "informe.log", encoding="utf-8"), logging.StreamHandler()],
)
log = logging.getLogger("informe")


# --------------------------------------------------------------- Excel
def generar_pdf_y_resumen() -> tuple[Path, list[tuple]]:
    if not LIBRO.exists():
        raise FileNotFoundError(LIBRO)
    SALIDA.mkdir(exist_ok=True)
    hoy = date.today()
    pdf = SALIDA / f"Informe_ventas_{hoy:%Y_%m}.pdf"

    excel = win32.DispatchEx("Excel.Application")
    excel.Visible = False
    excel.DisplayAlerts = False
    excel.ScreenUpdating = False
    libro = None
    try:
        log.info("Abriendo %s", LIBRO.name)
        libro = excel.Workbooks.Open(str(LIBRO.resolve()), UpdateLinks=0)

        log.info("Actualizando conexiones y tablas dinámicas…")
        libro.RefreshAll()
        excel.CalculateUntilAsyncQueriesDone()
        excel.CalculateFull()

        if MACRO:
            log.info("Ejecutando macro %s", MACRO)
            excel.Application.Run(MACRO)

        hoja = libro.Worksheets(HOJA_INFORME)
        valores = hoja.Range(RANGO_RESUMEN).Value
        resumen = [fila for fila in valores if any(c not in (None, "") for c in fila)]

        log.info("Exportando %s a PDF", HOJA_INFORME)
        hoja.ExportAsFixedFormat(Type=XL_TYPE_PDF, Filename=str(pdf.resolve()),
                                 Quality=XL_QUALITY_STANDARD, IncludeDocProperties=True,
                                 IgnorePrintAreas=False, OpenAfterPublish=False)
        libro.Save()
        return pdf, resumen
    finally:
        if libro is not None:
            libro.Close(SaveChanges=False)
        excel.ScreenUpdating = True
        excel.Quit()
        log.info("Excel cerrado")


# --------------------------------------------------------------- Outlook
def formatear(valor) -> str:
    if isinstance(valor, float):
        return f"{valor:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")
    return escape("" if valor is None else str(valor))


def tabla_html(filas: list[tuple]) -> str:
    if not filas:
        return "<p><i>Sin datos de resumen.</i></p>"
    cab, *cuerpo = filas
    estilo_th = "background:#1F4E78;color:#fff;padding:6px 10px;text-align:left"
    estilo_td = "padding:5px 10px;border-bottom:1px solid #ddd"
    html = ['<table style="border-collapse:collapse;font-family:Calibri,Arial;font-size:11pt">']
    html.append("<tr>" + "".join(f'<th style="{estilo_th}">{formatear(c)}</th>' for c in cab) + "</tr>")
    for fila in cuerpo:
        html.append("<tr>" + "".join(
            f'<td style="{estilo_td};text-align:{"right" if isinstance(c, (int, float)) else "left"}">'
            f"{formatear(c)}</td>" for c in fila) + "</tr>")
    html.append("</table>")
    return "".join(html)


def preparar_correo(pdf: Path, resumen: list[tuple]) -> None:
    hoy = date.today()
    mes = f"{MESES[hoy.month - 1]} de {hoy.year}"

    outlook = win32.Dispatch("Outlook.Application")
    correo = outlook.CreateItem(OL_MAIL_ITEM)
    correo.To = DESTINATARIOS
    correo.CC = CC
    correo.Subject = f"Informe de ventas · {mes}"
    correo.Attachments.Add(str(pdf.resolve()))

    correo.Display()                          # Carga la firma predeterminada en HTMLBody
    firma = correo.HTMLBody
    cuerpo = (
        '<div style="font-family:Calibri,Arial;font-size:11pt">'
        "<p>Buenos días:</p>"
        f"<p>Adjunto el informe de ventas correspondiente a <b>{mes}</b>. "
        "A continuación se muestra el resumen:</p>"
        f"{tabla_html(resumen)}"
        "<p>Quedo a vuestra disposición para cualquier aclaración.</p></div>"
    )
    correo.HTMLBody = cuerpo + firma

    if ENVIAR_DIRECTAMENTE:
        correo.Send()
        log.info("Correo enviado a %s", DESTINATARIOS)
    else:
        log.info("Correo preparado para revisión")


# --------------------------------------------------------------- Principal
def main() -> int:
    pythoncom.CoInitialize()                  # Necesario si se ejecuta en un hilo o servicio
    try:
        pdf, resumen = generar_pdf_y_resumen()
        log.info("PDF generado: %s (%d filas de resumen)", pdf.name, len(resumen))
        preparar_correo(pdf, resumen)
        return 0
    except pywintypes.com_error as e:
        detalle = e.excepinfo[2] if e.excepinfo else e.strerror
        log.error("Error de Office (COM): %s", detalle)
    except Exception:
        log.exception("Error inesperado")
    finally:
        pythoncom.CoUninitialize()
    return 1


if __name__ == "__main__":
    raise SystemExit(main())
```

> Programe este script con el **Programador de tareas de Windows** (con la opción «Ejecutar solo cuando el usuario haya iniciado sesión», ya que Office necesita una sesión interactiva).

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'win32com'` | PyWin32 no instalado o sistema no Windows. | `pip install pywin32` (solo Windows). |
| `ImportError: DLL load failed` | Instalación incompleta. | `python -m pywin32_postinstall -install` como administrador. |
| `pywintypes.com_error: (-2147221005, 'Invalid class string')` | La aplicación (Office) no está instalada o el ProgID es incorrecto. | Compruebe la instalación y el nombre (`Excel.Application`). |
| `com_error ... 'Sorry, we couldn't find ...'` | Ruta relativa o con `/`. | Use rutas absolutas: `str(Path(...).resolve())`. |
| Procesos `EXCEL.EXE` acumulados | El script falló sin llamar a `Quit()`. | Use `try/finally`; cierre los procesos huérfanos. |
| `AttributeError: '<win32com.gen_py...>' object has no attribute 'CLSIDToClassMap'` | Caché de `gencache` dañada. | Elimine la carpeta `%LOCALAPPDATA%\Temp\gen_py` y vuelva a ejecutar. |
| `CoInitialize has not been called` | Uso de COM en un hilo secundario. | Llame a `pythoncom.CoInitialize()` en ese hilo. |
| `com_error: 'Call was rejected by callee'` | Office está ocupado (diálogo abierto, cálculo). | Cierre diálogos, desactive alertas y reintente tras una pausa. |
| Los colores salen cambiados | Excel usa formato BGR. | Convierta: `r + g*256 + b*65536`. |
| `Find.Execute` no reemplaza textos largos | Límite de 255 caracteres. | Divida el texto o use rangos. |
| Las fórmulas en español no funcionan con `.Formula` | `.Formula` espera la sintaxis inglesa. | Use `.FormulaLocal` o las funciones en inglés (`SUM`). |
| Falla al ejecutarse desde una tarea programada sin sesión | Office necesita sesión de usuario interactiva. | Ejecute la tarea con la sesión iniciada. |

---

## 15. Buenas prácticas

- Use `DispatchEx` en scripts para no interferir con las ventanas de Office que el usuario tenga abiertas.
- Desactive `Visible`, `DisplayAlerts` y `ScreenUpdating` en procesos automáticos.
- **Cierre siempre** las aplicaciones en un bloque `finally`.
- Lea y escriba rangos completos en Excel en lugar de celda a celda.
- Defina las constantes que use como valores con nombre en su código.
- Use rutas absolutas en todas las llamadas COM.
- Para leer o generar archivos de Office sin necesidad de la aplicación, prefiera openpyxl, python-docx o python-pptx.
- Use la grabadora de macros de Office para descubrir los objetos y métodos necesarios.
- En Outlook, revise el correo (`Display()`) antes de activar el envío automático.
- Registre cada paso y capture `pywintypes.com_error` para mostrar mensajes claros.

---

## 16. Referencia rápida

```python
import win32com.client as win32, win32gui, win32con, win32api, win32clipboard, pythoncom, winreg

# Office (COM)
app = win32.DispatchEx("Excel.Application" | "Word.Application" | "PowerPoint.Application")
app = win32.GetActiveObject("Excel.Application")
app.Visible = False; app.DisplayAlerts = False; ...; app.Quit()

# Excel
lb = excel.Workbooks.Open(ruta_abs); h = lb.Worksheets("Hoja")
h.Range("A1").Value / h.Cells(f, c).Value / h.Range("A1:D10").Value = filas
lb.RefreshAll(); excel.Application.Run("Libro!Macro"); h.ExportAsFixedFormat(0, pdf)
lb.SaveAs(ruta, FileFormat=51); lb.Close(SaveChanges=False)

# Word
doc = word.Documents.Open(ruta); doc.Content.Find.Execute(FindText=, ReplaceWith=, Replace=2)
doc.SaveAs2(pdf, FileFormat=17); doc.Close(False)

# Outlook
ol = win32.Dispatch("Outlook.Application"); m = ol.CreateItem(0)
m.To / m.Subject / m.HTMLBody / m.Attachments.Add(ruta) / m.Display() / m.Send()
ol.GetNamespace("MAPI").GetDefaultFolder(6).Items.Restrict("[UnRead] = True")

# Ventanas
win32gui.FindWindow(None, "Título") / EnumWindows(cb, None) / SetForegroundWindow(h) / ShowWindow(h, SW_MAXIMIZE)
win32api.MessageBox(0, "texto", "título", win32con.MB_OK)

# Registro
with winreg.OpenKey(winreg.HKEY_CURRENT_USER, r"Software\...") as k: winreg.QueryValueEx(k, "nombre")
```

---

## 17. Ejercicios propuestos

1. Convierta a PDF todos los libros de Excel de una carpeta, exportando solo la primera hoja de cada uno.
2. Recorra la bandeja de entrada de Outlook, guarde los adjuntos PDF de correos con «factura» en el asunto y mueva los correos a una carpeta *Procesados*.
3. Genere contratos en Word a partir de una plantilla con marcadores y un CSV de clientes, y guárdelos en PDF.
4. Cree un script que liste las ventanas abiertas, permita elegir una por número y la maximice y traiga al frente.
5. Programe el ejemplo completo para que se ejecute el primer día laborable de cada mes.

---

## 18. Referencias

- Repositorio y documentación: https://github.com/mhammond/pywin32
- Modelo de objetos de Excel (VBA): https://learn.microsoft.com/es-es/office/vba/api/overview/excel
- Modelo de objetos de Word (VBA): https://learn.microsoft.com/es-es/office/vba/api/overview/word
- Modelo de objetos de Outlook (VBA): https://learn.microsoft.com/es-es/office/vba/api/overview/outlook
- `winreg`: https://docs.python.org/es/3/library/winreg.html
