# Manual de uso: PySide6

> **Categoría:** Interfaces gráficas · **Utilidad:** Aplicaciones de escritorio profesionales con Qt
> **Tipo:** Librería de terceros (`pip install PySide6`) · **Versión de referencia:** 6.5 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Primera aplicación](#3-primera-aplicación)
4. [Señales y ranuras](#4-señales-y-ranuras)
5. [Widgets básicos](#5-widgets-básicos)
6. [Disposiciones (*layouts*)](#6-disposiciones-layouts)
7. [Ventana principal: menús, barras de herramientas y paneles](#7-ventana-principal-menús-barras-de-herramientas-y-paneles)
8. [Diálogos](#8-diálogos)
9. [Formularios y validación](#9-formularios-y-validación)
10. [Arquitectura modelo/vista](#10-arquitectura-modelovista)
11. [Estilos: Fusion, paletas y hojas QSS](#11-estilos-fusion-paletas-y-hojas-qss)
12. [Hilos y tareas en segundo plano](#12-hilos-y-tareas-en-segundo-plano)
13. [Temporizadores y eventos](#13-temporizadores-y-eventos)
14. [Configuración persistente con `QSettings`](#14-configuración-persistente-con-qsettings)
15. [Bases de datos](#15-bases-de-datos)
16. [Gráficos: Qt Charts y Matplotlib](#16-gráficos-qt-charts-y-matplotlib)
17. [Qt Designer y archivos `.ui`](#17-qt-designer-y-archivos-ui)
18. [Pruebas automatizadas](#18-pruebas-automatizadas)
19. [Distribución](#19-distribución)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**PySide6** es el enlace oficial de Python para **Qt 6** (proyecto *Qt for Python*, mantenido por The Qt Company). Qt es uno de los marcos de interfaces gráficas más completos: *widgets* nativos en Windows, macOS y Linux, arquitectura modelo/vista, gráficos, multimedia, red, bases de datos, impresión, internacionalización y QML para interfaces declarativas.

**Ventajas:**
- Catálogo muy amplio de componentes (tablas, árboles, pestañas, paneles acoplables, editores de texto enriquecido…).
- Aspecto nativo y alto rendimiento (C++ por debajo).
- Arquitectura sólida: señales y ranuras, modelo/vista, hilos.
- Herramientas: Qt Designer (diseño visual), Qt Linguist (traducción), `pyside6-deploy`.
- Licencia **LGPL**: puede usarse en aplicaciones comerciales cumpliendo sus condiciones.

**Limitaciones:**
- Curva de aprendizaje mayor que Tkinter o CustomTkinter.
- Paquete pesado (varios cientos de MB instalado; ejecutables de decenas de MB).
- Documentación principal orientada a C++ (aunque la de Python es cada vez más completa).

| Si necesita… | Opción |
|---|---|
| Aplicación de escritorio profesional y compleja | **PySide6** |
| Misma API con licencia GPL/comercial de Riverbank | PyQt6 |
| Interfaz sencilla sin dependencias | Tkinter |
| Interfaz sencilla con aspecto moderno | CustomTkinter |
| Interfaz web para análisis de datos | Streamlit |

---

## 2. Instalación e importación

```bash
pip install PySide6
```

El paquete incluye los módulos principales, Qt Designer (`pyside6-designer`), el compilador de interfaces (`pyside6-uic`), el de recursos (`pyside6-rcc`) y la herramienta de distribución (`pyside6-deploy`).

```python
from PySide6.QtCore import Qt, Signal, Slot, QTimer, QSettings, QThreadPool
from PySide6.QtGui import QAction, QIcon, QKeySequence, QFont, QColor
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, QPushButton, QLabel, QVBoxLayout
```

| Módulo | Contenido |
|---|---|
| `QtCore` | Núcleo: señales, temporizadores, hilos, modelos, configuración, fechas. |
| `QtGui` | Acciones, iconos, fuentes, colores, imágenes, atajos. |
| `QtWidgets` | *Widgets* y ventanas. |
| `QtCharts` | Gráficos de líneas, barras, sectores… |
| `QtSql` | Acceso a bases de datos (SQLite, PostgreSQL, MySQL…). |
| `QtNetwork`, `QtWebEngineWidgets`, `QtMultimedia`, `QtPrintSupport` | Red, navegador integrado, audio/vídeo, impresión. |

---

## 3. Primera aplicación

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget, QLineEdit, QPushButton, QLabel, QVBoxLayout


class Saludo(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Saludo")
        self.entrada = QLineEdit(placeholderText="Escriba su nombre")
        self.boton = QPushButton("Saludar")
        self.resultado = QLabel()

        disposicion = QVBoxLayout(self)
        for w in (self.entrada, self.boton, self.resultado):
            disposicion.addWidget(w)

        self.boton.clicked.connect(self.saludar)
        self.entrada.returnPressed.connect(self.saludar)

    def saludar(self):
        self.resultado.setText(f"Hola, {self.entrada.text().strip() or 'desconocido'}")


if __name__ == "__main__":
    app = QApplication(sys.argv)          # Una única instancia por proceso, antes de cualquier widget
    ventana = Saludo()
    ventana.resize(320, 140)
    ventana.show()
    sys.exit(app.exec())                  # Bucle de eventos
```

> En PySide6 las propiedades pueden pasarse como argumentos con nombre al constructor (`QLineEdit(placeholderText="…")`), lo que equivale a llamar a `setPlaceholderText`.

---

## 4. Señales y ranuras

Los objetos Qt emiten **señales** cuando ocurre algo; se conectan a **ranuras** (cualquier función o método).

```python
boton.clicked.connect(self.guardar)
entrada.textChanged.connect(lambda texto: etiqueta.setText(texto.upper()))
deslizador.valueChanged.connect(barra.setValue)          # De widget a widget
boton.clicked.disconnect(self.guardar)
```

Señales propias:

```python
from PySide6.QtCore import QObject, Signal, Slot

class Temporizador(QObject):
    progreso = Signal(int)                 # Tipos de los argumentos
    terminado = Signal(str, float)

    def ejecutar(self):
        for i in range(101):
            self.progreso.emit(i)
        self.terminado.emit("OK", 1.5)

t = Temporizador()
t.progreso.connect(barra.setValue)
t.terminado.connect(lambda estado, segundos: print(estado, segundos))
```

- El decorador `@Slot(tipo)` es opcional; mejora ligeramente el rendimiento y documenta la firma.
- Las conexiones entre hilos son seguras: si el emisor vive en otro hilo, la ranura se ejecuta en el hilo del receptor (conexión en cola).

---

## 5. Widgets básicos

| *Widget* | Uso | Señal habitual |
|---|---|---|
| `QLabel` | Texto o imagen (admite HTML sencillo). | — |
| `QPushButton`, `QToolButton` | Botones. | `clicked` |
| `QLineEdit` | Texto de una línea. | `textChanged`, `returnPressed`, `editingFinished` |
| `QTextEdit`, `QPlainTextEdit` | Texto multilínea (enriquecido / plano). | `textChanged` |
| `QSpinBox`, `QDoubleSpinBox` | Números con flechas. | `valueChanged` |
| `QComboBox` | Lista desplegable (editable opcionalmente). | `currentIndexChanged`, `currentTextChanged` |
| `QCheckBox`, `QRadioButton` | Opciones. | `toggled`, `checkStateChanged` |
| `QSlider`, `QProgressBar` | Rango de valores / progreso. | `valueChanged` |
| `QDateEdit`, `QCalendarWidget` | Fechas. | `dateChanged` |
| `QListWidget`, `QTableWidget`, `QTreeWidget` | Listas/tablas sencillas con elementos. | `itemSelectionChanged` |
| `QTabWidget`, `QStackedWidget`, `QGroupBox`, `QScrollArea`, `QSplitter` | Contenedores. | `currentChanged` |

```python
combo = QComboBox()
combo.addItems(["Madrid", "Sevilla"])
combo.addItem("Bilbao", userData=48)              # Dato asociado
combo.currentData() ; combo.setCurrentText("Sevilla")

precio = QDoubleSpinBox(minimum=0, maximum=99_999, decimals=2, suffix=" €", singleStep=0.5)
fecha = QDateEdit(calendarPopup=True, displayFormat="dd/MM/yyyy")
fecha.setDate(QDate.currentDate())
fecha.date().toPython()                           # → datetime.date

etiqueta = QLabel("<b>Total:</b> 120 €")
etiqueta.setAlignment(Qt.AlignmentFlag.AlignRight | Qt.AlignmentFlag.AlignVCenter)
etiqueta.setWordWrap(True)
```

---

## 6. Disposiciones (*layouts*)

Las disposiciones colocan y redimensionan los *widgets* automáticamente:

```python
from PySide6.QtWidgets import QHBoxLayout, QVBoxLayout, QGridLayout, QFormLayout

fila = QHBoxLayout()
fila.addWidget(QLabel("Buscar:"))
fila.addWidget(buscador, stretch=1)              # Ocupa el espacio sobrante
fila.addStretch()                                # Espacio flexible
fila.addWidget(boton)

rejilla = QGridLayout()
rejilla.addWidget(QLabel("Nombre"), 0, 0)
rejilla.addWidget(nombre, 0, 1)
rejilla.addWidget(notas, 1, 0, 1, 2)             # Fila 1, columna 0, ocupa 1 fila y 2 columnas
rejilla.setColumnStretch(1, 1)

principal = QVBoxLayout(self)                    # Asignar al widget contenedor
principal.addLayout(fila)
principal.addLayout(rejilla)
principal.setContentsMargins(12, 12, 12, 12)
principal.setSpacing(8)
```

Para dividir zonas redimensionables por el usuario, use `QSplitter(Qt.Orientation.Horizontal)` con `addWidget`.

---

## 7. Ventana principal: menús, barras de herramientas y paneles

`QMainWindow` aporta barra de menús, barras de herramientas, barra de estado, paneles acoplables y un *widget* central.

```python
from PySide6.QtGui import QAction, QKeySequence
from PySide6.QtWidgets import QMainWindow, QDockWidget, QStyle

class Principal(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Editor")
        self.setCentralWidget(QPlainTextEdit())

        abrir = QAction(self.style().standardIcon(QStyle.StandardPixmap.SP_DialogOpenButton), "&Abrir…", self)
        abrir.setShortcut(QKeySequence.StandardKey.Open)        # Ctrl+O según la plataforma
        abrir.setStatusTip("Abrir un archivo")
        abrir.triggered.connect(self.abrir)

        menu = self.menuBar().addMenu("&Archivo")
        menu.addAction(abrir)
        menu.addSeparator()
        menu.addAction("Salir", QKeySequence("Ctrl+Q"), self.close)

        barra = self.addToolBar("Principal")
        barra.addAction(abrir)

        self.statusBar().showMessage("Listo", 3000)            # Mensaje temporal (ms)
        self.statusBar().addPermanentWidget(QLabel("UTF-8"))

        panel = QDockWidget("Propiedades", self)
        panel.setWidget(QListWidget())
        self.addDockWidget(Qt.DockWidgetArea.RightDockWidgetArea, panel)
        self.menuBar().addMenu("&Ver").addAction(panel.toggleViewAction())   # Mostrar/ocultar
```

Acciones marcables: `accion.setCheckable(True)` y señal `toggled(bool)`. Agrupe opciones excluyentes con `QActionGroup`. Menú contextual: `widget.setContextMenuPolicy(Qt.ContextMenuPolicy.ActionsContextMenu)` y `widget.addAction(accion)`.

---

## 8. Diálogos

```python
from PySide6.QtWidgets import QMessageBox, QFileDialog, QInputDialog

QMessageBox.information(self, "Exportación", "Archivo guardado correctamente.")
QMessageBox.warning(self, "Atención", "Hay campos vacíos.")
respuesta = QMessageBox.question(self, "Eliminar", "¿Eliminar 3 registros?",
                                 QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No)
if respuesta == QMessageBox.StandardButton.Yes: ...

ruta, _ = QFileDialog.getOpenFileName(self, "Abrir", "", "CSV (*.csv);;Todos (*)")
ruta, _ = QFileDialog.getSaveFileName(self, "Guardar como", "informe.csv", "CSV (*.csv)")
carpeta = QFileDialog.getExistingDirectory(self, "Carpeta de destino")

texto, ok = QInputDialog.getText(self, "Categoría", "Nombre:")
numero, ok = QInputDialog.getInt(self, "Unidades", "Cantidad:", value=1, minValue=1, maxValue=999)
```

Diálogo propio:

```python
from PySide6.QtWidgets import QDialog, QDialogButtonBox

class DialogoNombre(QDialog):
    def __init__(self, padre=None):
        super().__init__(padre)
        self.setWindowTitle("Nuevo cliente")
        self.nombre = QLineEdit()
        botones = QDialogButtonBox(QDialogButtonBox.StandardButton.Ok | QDialogButtonBox.StandardButton.Cancel)
        botones.accepted.connect(self.accept)
        botones.rejected.connect(self.reject)
        capa = QVBoxLayout(self)
        capa.addWidget(self.nombre)
        capa.addWidget(botones)

dialogo = DialogoNombre(self)
if dialogo.exec() == QDialog.DialogCode.Accepted:      # Modal: espera a que se cierre
    print(dialogo.nombre.text())
```

---

## 9. Formularios y validación

```python
from PySide6.QtCore import QRegularExpression
from PySide6.QtGui import QRegularExpressionValidator, QIntValidator
from PySide6.QtWidgets import QFormLayout, QCompleter

formulario = QFormLayout()
codigo = QLineEdit(placeholderText="ABC-123")
codigo.setValidator(QRegularExpressionValidator(QRegularExpression(r"[A-Z]{3}-\d{3}")))
codigo.setInputMask(">AAA-999;_")                 # Alternativa: máscara de entrada (mayúsculas automáticas)
cp = QLineEdit(); cp.setValidator(QIntValidator(1000, 52999))
ciudad = QLineEdit(); ciudad.setCompleter(QCompleter(["Madrid", "Málaga", "Murcia"]))
formulario.addRow("Código:", codigo)
formulario.addRow("Código postal:", cp)
formulario.addRow("&Ciudad:", ciudad)            # & = atajo Alt+C que enfoca el campo

codigo.hasAcceptableInput()                       # True si cumple el validador por completo
```

Patrón recomendado: conecte `textChanged`/`valueChanged` de los campos a un método que compruebe la validez, active o desactive el botón *Aceptar* y marque los campos erróneos (por ejemplo, con una propiedad dinámica y QSS: `campo.setProperty("error", True)`).

---

## 10. Arquitectura modelo/vista

Para tablas con muchos datos, separe los **datos** (modelo) de su **presentación** (vista). La vista solo pide al modelo lo que necesita mostrar.

### 10.1 Modelo de tabla propio

```python
from PySide6.QtCore import QAbstractTableModel, QModelIndex, Qt

class ModeloClientes(QAbstractTableModel):
    COLUMNAS = ["Nombre", "Ciudad", "Saldo"]

    def __init__(self, filas: list[dict]):
        super().__init__()
        self.filas = filas

    def rowCount(self, padre=QModelIndex()):
        return 0 if padre.isValid() else len(self.filas)

    def columnCount(self, padre=QModelIndex()):
        return 0 if padre.isValid() else len(self.COLUMNAS)

    def data(self, indice, rol=Qt.ItemDataRole.DisplayRole):
        fila, clave = self.filas[indice.row()], ["nombre", "ciudad", "saldo"][indice.column()]
        valor = fila[clave]
        if rol == Qt.ItemDataRole.DisplayRole:
            return f"{valor:,.2f} €" if clave == "saldo" else valor
        if rol == Qt.ItemDataRole.EditRole:
            return valor
        if rol == Qt.ItemDataRole.ForegroundRole and clave == "saldo" and valor < 0:
            return QColor("#B42318")
        if rol == Qt.ItemDataRole.TextAlignmentRole and clave == "saldo":
            return Qt.AlignmentFlag.AlignRight | Qt.AlignmentFlag.AlignVCenter
        return None

    def headerData(self, seccion, orientacion, rol=Qt.ItemDataRole.DisplayRole):
        if rol == Qt.ItemDataRole.DisplayRole and orientacion == Qt.Orientation.Horizontal:
            return self.COLUMNAS[seccion]
        return None

    def agregar(self, fila: dict):
        self.beginInsertRows(QModelIndex(), len(self.filas), len(self.filas))
        self.filas.append(fila)
        self.endInsertRows()
```

| Rol | Uso |
|---|---|
| `DisplayRole` | Texto mostrado. |
| `EditRole` | Valor para el editor (y para ordenar, si así se configura). |
| `ToolTipRole`, `StatusTipRole` | Ayudas. |
| `ForegroundRole`, `BackgroundRole`, `FontRole` | Formato. |
| `TextAlignmentRole` | Alineación. |
| `DecorationRole` | Icono. |
| `CheckStateRole` | Casilla de verificación. |
| `UserRole` (+ n) | Datos propios (identificadores, objetos). |

Para que una celda sea **editable**, implemente `flags()` (añadiendo `Qt.ItemFlag.ItemIsEditable`) y `setData()`, que debe emitir `dataChanged`. Antes y después de cambiar la estructura llame a `beginInsertRows/endInsertRows`, `beginRemoveRows/endRemoveRows` o `beginResetModel/endResetModel`.

### 10.2 Vista, ordenación y filtrado

```python
from PySide6.QtCore import QSortFilterProxyModel
from PySide6.QtWidgets import QTableView, QHeaderView, QAbstractItemView

proxy = QSortFilterProxyModel()
proxy.setSourceModel(modelo)
proxy.setFilterCaseSensitivity(Qt.CaseSensitivity.CaseInsensitive)
proxy.setFilterKeyColumn(-1)                         # Buscar en todas las columnas
proxy.setSortRole(Qt.ItemDataRole.EditRole)          # Ordenar por el valor, no por el texto formateado
buscador.textChanged.connect(proxy.setFilterFixedString)

tabla = QTableView()
tabla.setModel(proxy)
tabla.setSortingEnabled(True)
tabla.setSelectionBehavior(QAbstractItemView.SelectionBehavior.SelectRows)
tabla.setAlternatingRowColors(True)
tabla.horizontalHeader().setSectionResizeMode(0, QHeaderView.ResizeMode.Stretch)
tabla.verticalHeader().hide()

filas = {proxy.mapToSource(i).row() for i in tabla.selectionModel().selectedRows()}   # Índices reales
```

Para filtros con varias condiciones, cree una subclase de `QSortFilterProxyModel` y sobrescriba `filterAcceptsRow(fila, padre)`.

---

## 11. Estilos: Fusion, paletas y hojas QSS

```python
app.setStyle("Fusion")                               # Estilo homogéneo en todas las plataformas
app.styleHints().setColorScheme(Qt.ColorScheme.Dark)  # Forzar oscuro (Qt 6.8+); Qt.ColorScheme.Unknown = sistema
```

Las **hojas de estilo QSS** usan una sintaxis similar a CSS:

```python
app.setStyleSheet("""
    QPushButton { background: #1F4E78; color: white; border-radius: 6px; padding: 6px 14px; }
    QPushButton:hover { background: #2E75B6; }
    QPushButton:disabled { background: #A0A7B0; }
    QLineEdit[error="true"] { border: 1px solid #B42318; }      /* Propiedad dinámica */
    QTableView { gridline-color: #E4E7EC; selection-background-color: #2E75B6; }
    QHeaderView::section { font-weight: bold; padding: 4px; }
    #titulo { font-size: 18pt; font-weight: 600; }              /* Por objectName */
""")
titulo.setObjectName("titulo")
campo.setProperty("error", True); campo.style().unpolish(campo); campo.style().polish(campo)   # Reaplicar estilo
```

---

## 12. Hilos y tareas en segundo plano

Las operaciones largas en el hilo principal congelan la interfaz. **Nunca** modifique *widgets* desde otro hilo: emita señales y deje que el hilo principal actualice la interfaz.

### 12.1 `QRunnable` + `QThreadPool` (tareas puntuales)

```python
from PySide6.QtCore import QObject, QRunnable, QThreadPool, Signal

class SenalesTarea(QObject):
    progreso = Signal(int)
    terminado = Signal(object)
    error = Signal(str)

class Tarea(QRunnable):
    def __init__(self, funcion, *args):
        super().__init__()
        self.funcion, self.args = funcion, args
        self.senales = SenalesTarea()

    def run(self):                                      # Se ejecuta en otro hilo
        try:
            resultado = self.funcion(*self.args, progreso=self.senales.progreso.emit)
            self.senales.terminado.emit(resultado)
        except Exception as e:
            self.senales.error.emit(str(e))

tarea = Tarea(procesar_archivo, ruta)
tarea.senales.progreso.connect(barra.setValue)
tarea.senales.terminado.connect(mostrar_resultado)
QThreadPool.globalInstance().start(tarea)
```

### 12.2 `QThread` con un objeto trabajador (procesos de larga duración)

```python
hilo = QThread()
trabajador = Trabajador()                       # QObject con señales y un método ejecutar()
trabajador.moveToThread(hilo)
hilo.started.connect(trabajador.ejecutar)
trabajador.terminado.connect(hilo.quit)
trabajador.terminado.connect(trabajador.deleteLater)
hilo.finished.connect(hilo.deleteLater)
hilo.start()
```

---

## 13. Temporizadores y eventos

```python
reloj = QTimer(self, interval=1000)
reloj.timeout.connect(self.actualizar_hora)
reloj.start()
QTimer.singleShot(500, self.cargar_datos)          # Una sola vez, tras 500 ms

class Ventana(QMainWindow):
    def closeEvent(self, evento):                   # Al cerrar
        if self.hay_cambios and QMessageBox.question(self, "Salir", "¿Salir sin guardar?") != QMessageBox.StandardButton.Yes:
            evento.ignore()
            return
        evento.accept()

    def keyPressEvent(self, evento):
        if evento.key() == Qt.Key.Key_Escape:
            self.buscador.clear()
        else:
            super().keyPressEvent(evento)
```

Para capturar eventos de otros *widgets* sin crear subclases, use `widget.installEventFilter(self)` y sobrescriba `eventFilter(objeto, evento)`.

---

## 14. Configuración persistente con `QSettings`

Guarda preferencias en el lugar estándar de cada sistema (registro de Windows, `.plist` en macOS, `.conf` en Linux):

```python
QApplication.setOrganizationName("MiEmpresa")
QApplication.setApplicationName("Inventario")
ajustes = QSettings()

ajustes.setValue("ventana/geometria", self.saveGeometry())
ajustes.setValue("ventana/estado", self.saveState())          # Barras y paneles
ajustes.setValue("tema", "oscuro")

if (geo := ajustes.value("ventana/geometria")) is not None:
    self.restoreGeometry(geo)
tema = ajustes.value("tema", "claro")
limite = ajustes.value("limite", 100, type=int)              # Indique el tipo: en Linux todo se lee como texto
```

---

## 15. Bases de datos

Puede usar `sqlite3`/SQLAlchemy en un modelo propio o el módulo **QtSql**, que ofrece modelos conectados directamente a tablas:

```python
from PySide6.QtSql import QSqlDatabase, QSqlTableModel, QSqlQuery

bd = QSqlDatabase.addDatabase("QSQLITE")
bd.setDatabaseName("inventario.db")
bd.open()
QSqlQuery("CREATE TABLE IF NOT EXISTS productos (id INTEGER PRIMARY KEY, nombre TEXT, stock INTEGER)")

modelo = QSqlTableModel(db=bd)
modelo.setTable("productos")
modelo.setEditStrategy(QSqlTableModel.EditStrategy.OnManualSubmit)
modelo.select()
tabla.setModel(modelo)              # Edición directa en la tabla
modelo.submitAll()                  # Guardar cambios
```

---

## 16. Gráficos: Qt Charts y Matplotlib

```python
from PySide6.QtCharts import QChart, QChartView, QBarSeries, QBarSet, QBarCategoryAxis, QValueAxis
from PySide6.QtGui import QPainter

serie = QBarSeries()
ventas = QBarSet("Ventas"); ventas.append([120, 95, 140])
serie.append(ventas)
grafico = QChart(); grafico.addSeries(serie); grafico.setTitle("Ventas por región")
eje_x = QBarCategoryAxis(); eje_x.append(["Norte", "Centro", "Sur"])
grafico.addAxis(eje_x, Qt.AlignmentFlag.AlignBottom); serie.attachAxis(eje_x)
eje_y = QValueAxis(); grafico.addAxis(eje_y, Qt.AlignmentFlag.AlignLeft); serie.attachAxis(eje_y)
vista = QChartView(grafico); vista.setRenderHint(QPainter.RenderHint.Antialiasing)
```

Con Matplotlib:

```python
from matplotlib.backends.backend_qtagg import FigureCanvasQTAgg, NavigationToolbar2QT
from matplotlib.figure import Figure
lienzo = FigureCanvasQTAgg(Figure(figsize=(5, 3), layout="constrained"))
ejes = lienzo.figure.add_subplot()
ejes.plot([1, 2, 3], [4, 1, 5]); lienzo.draw_idle()
capa.addWidget(NavigationToolbar2QT(lienzo, self)); capa.addWidget(lienzo)
```

---

## 17. Qt Designer y archivos `.ui`

Qt Designer permite diseñar ventanas visualmente (`pyside6-designer`). El archivo `.ui` resultante se puede usar de dos maneras:

```bash
pyside6-uic ventana.ui -o ui_ventana.py          # Opción 1: generar código Python
```

```python
from ui_ventana import Ui_MainWindow
class Ventana(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super().__init__()
        self.setupUi(self)                          # Crea los widgets definidos en Designer
        self.botonGuardar.clicked.connect(self.guardar)

# Opción 2: cargar el .ui en tiempo de ejecución
from PySide6.QtUiTools import QUiLoader
ventana = QUiLoader().load("ventana.ui")
```

Recursos (iconos, imágenes) empaquetados: archivo `.qrc` compilado con `pyside6-rcc recursos.qrc -o recursos_rc.py`.

---

## 18. Pruebas automatizadas

Con **pytest-qt** (`pip install pytest-qt`):

```python
def test_saludo(qtbot):
    ventana = Saludo()
    qtbot.addWidget(ventana)
    qtbot.keyClicks(ventana.entrada, "Ana")
    qtbot.mouseClick(ventana.boton, Qt.MouseButton.LeftButton)
    assert ventana.resultado.text() == "Hola, Ana"

def test_senal(qtbot, trabajador):
    with qtbot.waitSignal(trabajador.terminado, timeout=5000):
        trabajador.iniciar()
```

En servidores sin pantalla (integración continua), ejecute con `QT_QPA_PLATFORM=offscreen`.

---

## 19. Distribución

```bash
pyside6-deploy main.py                        # Herramienta oficial (basada en Nuitka); genera pysidedeploy.spec
pyinstaller --windowed --name Inventario main.py    # Alternativa
```

Incluya solo los módulos que use: PyInstaller detecta los módulos Qt importados. Recuerde que la licencia LGPL exige permitir al usuario sustituir las bibliotecas Qt (la distribución en carpeta, no en un único archivo, lo facilita).

---

## 20. Ejemplo práctico completo

**Gestor de inventario** de escritorio con:

- Ventana principal con menús, barra de herramientas con iconos estándar, barra de estado con indicadores y panel acoplable con un **gráfico de Qt Charts** (valor del stock por categoría).
- **Modelo de tabla propio** con formato condicional (productos bajo mínimo en rojo), edición en línea de stock y precio, ordenación por valor y **filtro combinado** (texto, categoría y «solo bajo mínimo»).
- **Diálogo de alta** con validadores, autocompletado y botón *Aceptar* que solo se activa cuando el formulario es válido.
- Persistencia en **SQLite**, importación/exportación **CSV en segundo plano** (`QThreadPool`) con barra de progreso.
- Tema claro/oscuro con una paleta Fusion (más QSS ligero) y preferencias (geometría, tema) en `QSettings`.

```python
"""
inventario.py — Gestor de inventario de escritorio con PySide6.
Requiere: pip install PySide6
Uso: python inventario.py
"""
from __future__ import annotations

import csv
import sqlite3
import sys
import time
from dataclasses import astuple, dataclass, fields
from pathlib import Path

from PySide6.QtCharts import QBarCategoryAxis, QBarSet, QChart, QChartView, QHorizontalBarSeries, QValueAxis
from PySide6.QtCore import (QAbstractTableModel, QModelIndex, QObject, QRegularExpression, QRunnable, QSettings,
                            QSortFilterProxyModel, Qt, QThreadPool, Signal)
from PySide6.QtGui import QAction, QColor, QKeySequence, QPainter, QPalette, QRegularExpressionValidator
from PySide6.QtWidgets import (QAbstractItemView, QApplication, QCheckBox, QComboBox, QCompleter, QDialog,
                               QDialogButtonBox, QDockWidget, QDoubleSpinBox, QFileDialog, QFormLayout, QHBoxLayout,
                               QHeaderView, QLabel, QLineEdit, QMainWindow, QMessageBox, QProgressBar, QSpinBox,
                               QStyle, QTableView, QVBoxLayout, QWidget)

BD = Path("inventario.db")
CATEGORIAS = ["Electrónica", "Oficina", "Mobiliario", "Limpieza", "Almacén"]


# ============================================================ Datos
@dataclass
class Producto:
    codigo: str
    nombre: str
    categoria: str
    stock: int
    minimo: int
    precio: float

    @property
    def bajo_minimo(self) -> bool:
        return self.stock < self.minimo

    @property
    def valor(self) -> float:
        return self.stock * self.precio


class Almacen:
    """Repositorio SQLite (sin dependencias de Qt)."""

    def __init__(self, ruta: Path):
        self.con = sqlite3.connect(ruta)
        self.con.execute("""CREATE TABLE IF NOT EXISTS productos (
            codigo TEXT PRIMARY KEY, nombre TEXT NOT NULL, categoria TEXT NOT NULL,
            stock INTEGER NOT NULL CHECK (stock >= 0), minimo INTEGER NOT NULL, precio REAL NOT NULL)""")
        if self.con.execute("SELECT COUNT(*) FROM productos").fetchone()[0] == 0:
            self.guardar_todos(datos_ejemplo())

    def todos(self) -> list[Producto]:
        return [Producto(*f) for f in self.con.execute("SELECT * FROM productos ORDER BY codigo")]

    def guardar(self, p: Producto) -> None:
        with self.con:
            self.con.execute("INSERT OR REPLACE INTO productos VALUES (?, ?, ?, ?, ?, ?)", astuple(p))

    def guardar_todos(self, productos: list[Producto]) -> None:
        with self.con:
            self.con.executemany("INSERT OR REPLACE INTO productos VALUES (?, ?, ?, ?, ?, ?)",
                                 [astuple(p) for p in productos])

    def eliminar(self, codigos: list[str]) -> None:
        with self.con:
            self.con.executemany("DELETE FROM productos WHERE codigo = ?", [(c,) for c in codigos])


def datos_ejemplo() -> list[Producto]:
    filas = [("ELE-001", "Monitor 27\"", "Electrónica", 14, 5, 219.0), ("ELE-002", "Teclado inalámbrico", "Electrónica", 3, 10, 39.9),
             ("ELE-003", "Ratón ergonómico", "Electrónica", 22, 10, 29.5), ("OFI-001", "Papel A4 (caja)", "Oficina", 40, 20, 24.0),
             ("OFI-002", "Bolígrafos (pack 50)", "Oficina", 6, 8, 12.5), ("OFI-003", "Archivadores", "Oficina", 55, 15, 3.2),
             ("MOB-001", "Silla de oficina", "Mobiliario", 4, 2, 189.0), ("MOB-002", "Mesa regulable", "Mobiliario", 1, 2, 420.0),
             ("LIM-001", "Gel hidroalcohólico", "Limpieza", 30, 12, 4.8), ("ALM-001", "Cajas de cartón", "Almacén", 120, 50, 1.1)]
    return [Producto(*f) for f in filas]


# ============================================================ Modelo de tabla
class ModeloProductos(QAbstractTableModel):
    COLUMNAS = [("codigo", "Código"), ("nombre", "Nombre"), ("categoria", "Categoría"),
                ("stock", "Stock"), ("minimo", "Mínimo"), ("precio", "Precio"), ("valor", "Valor")]
    EDITABLES = {"stock", "precio"}
    cambiado = Signal()

    def __init__(self, almacen: Almacen):
        super().__init__()
        self.almacen = almacen
        self.productos = almacen.todos()

    def rowCount(self, padre=QModelIndex()):
        return 0 if padre.isValid() else len(self.productos)

    def columnCount(self, padre=QModelIndex()):
        return 0 if padre.isValid() else len(self.COLUMNAS)

    def data(self, indice, rol=Qt.ItemDataRole.DisplayRole):
        if not indice.isValid():
            return None
        p = self.productos[indice.row()]
        clave = self.COLUMNAS[indice.column()][0]
        valor = getattr(p, clave)
        if rol == Qt.ItemDataRole.DisplayRole:
            if clave in ("precio", "valor"):
                return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")
            return str(valor)
        if rol in (Qt.ItemDataRole.EditRole, Qt.ItemDataRole.UserRole):
            return valor
        if rol == Qt.ItemDataRole.TextAlignmentRole and clave in ("stock", "minimo", "precio", "valor"):
            return int(Qt.AlignmentFlag.AlignRight | Qt.AlignmentFlag.AlignVCenter)
        if rol == Qt.ItemDataRole.ForegroundRole and p.bajo_minimo and clave in ("stock", "nombre"):
            return QColor("#D92D20")
        if rol == Qt.ItemDataRole.ToolTipRole and p.bajo_minimo:
            return f"Bajo mínimo: faltan {p.minimo - p.stock} unidades"
        return None

    def headerData(self, seccion, orientacion, rol=Qt.ItemDataRole.DisplayRole):
        if orientacion == Qt.Orientation.Horizontal and rol == Qt.ItemDataRole.DisplayRole:
            return self.COLUMNAS[seccion][1]
        return None

    def flags(self, indice):
        base = super().flags(indice)
        if self.COLUMNAS[indice.column()][0] in self.EDITABLES:
            return base | Qt.ItemFlag.ItemIsEditable
        return base

    def setData(self, indice, valor, rol=Qt.ItemDataRole.EditRole):
        if rol != Qt.ItemDataRole.EditRole:
            return False
        p = self.productos[indice.row()]
        clave = self.COLUMNAS[indice.column()][0]
        try:
            nuevo = int(valor) if clave == "stock" else round(float(valor), 2)
        except (TypeError, ValueError):
            return False
        if nuevo < 0:
            return False
        setattr(p, clave, nuevo)
        self.almacen.guardar(p)
        fila = indice.row()
        self.dataChanged.emit(self.index(fila, 0), self.index(fila, self.columnCount() - 1))   # Toda la fila (color, valor)
        self.cambiado.emit()
        return True

    # --- Operaciones
    def agregar(self, p: Producto) -> None:
        self.almacen.guardar(p)
        self.beginInsertRows(QModelIndex(), len(self.productos), len(self.productos))
        self.productos.append(p)
        self.endInsertRows()
        self.cambiado.emit()

    def eliminar(self, filas: list[int]) -> None:
        self.almacen.eliminar([self.productos[f].codigo for f in filas])
        for f in sorted(filas, reverse=True):              # De abajo arriba para no desplazar índices
            self.beginRemoveRows(QModelIndex(), f, f)
            del self.productos[f]
            self.endRemoveRows()
        self.cambiado.emit()

    def recargar(self) -> None:
        self.beginResetModel()
        self.productos = self.almacen.todos()
        self.endResetModel()
        self.cambiado.emit()

    def existe(self, codigo: str) -> bool:
        return any(p.codigo == codigo for p in self.productos)


class FiltroProductos(QSortFilterProxyModel):
    def __init__(self):
        super().__init__()
        self.texto, self.categoria, self.solo_bajo = "", "Todas", False
        self.setSortRole(Qt.ItemDataRole.UserRole)        # Ordena por valor numérico, no por texto

    def configurar(self, texto=None, categoria=None, solo_bajo=None) -> None:
        self.beginFilterChange()
        if texto is not None:
            self.texto = texto.lower()
        if categoria is not None:
            self.categoria = categoria
        if solo_bajo is not None:
            self.solo_bajo = solo_bajo
        self.endFilterChange()

    def filterAcceptsRow(self, fila, padre):
        p = self.sourceModel().productos[fila]
        return ((self.texto in p.codigo.lower() or self.texto in p.nombre.lower())
                and (self.categoria == "Todas" or p.categoria == self.categoria)
                and (not self.solo_bajo or p.bajo_minimo))


# ============================================================ Diálogo de alta
class DialogoProducto(QDialog):
    def __init__(self, padre, existe):
        super().__init__(padre)
        self.setWindowTitle("Nuevo producto")
        self.existe = existe
        self.codigo = QLineEdit(placeholderText="ABC-123")
        self.codigo.setValidator(QRegularExpressionValidator(QRegularExpression(r"[A-Z]{3}-\d{3}")))
        self.nombre = QLineEdit()
        self.categoria = QComboBox(editable=True)
        self.categoria.addItems(CATEGORIAS)
        self.categoria.completer().setCompletionMode(QCompleter.CompletionMode.PopupCompletion)
        self.stock = QSpinBox(maximum=100_000)
        self.minimo = QSpinBox(maximum=100_000, value=5)
        self.precio = QDoubleSpinBox(maximum=1_000_000, decimals=2, suffix=" €")
        self.aviso = QLabel()
        self.aviso.setStyleSheet("color: #D92D20")

        formulario = QFormLayout()
        for etiqueta, campo in [("&Código:", self.codigo), ("&Nombre:", self.nombre), ("C&ategoría:", self.categoria),
                                ("&Stock:", self.stock), ("&Mínimo:", self.minimo), ("&Precio:", self.precio)]:
            formulario.addRow(etiqueta, campo)
        self.botones = QDialogButtonBox(QDialogButtonBox.StandardButton.Ok | QDialogButtonBox.StandardButton.Cancel)
        self.botones.accepted.connect(self.accept)
        self.botones.rejected.connect(self.reject)

        capa = QVBoxLayout(self)
        capa.addLayout(formulario)
        capa.addWidget(self.aviso)
        capa.addWidget(self.botones)
        for senal in (self.codigo.textChanged, self.nombre.textChanged, self.precio.valueChanged):
            senal.connect(self.validar)
        self.validar()

    def validar(self) -> bool:
        problemas = []
        if not self.codigo.hasAcceptableInput():
            problemas.append("código con formato ABC-123")
        elif self.existe(self.codigo.text()):
            problemas.append("el código ya existe")
        if len(self.nombre.text().strip()) < 3:
            problemas.append("nombre (mín. 3 caracteres)")
        if self.precio.value() <= 0:
            problemas.append("precio mayor que 0")
        self.aviso.setText("Falta: " + ", ".join(problemas) if problemas else "")
        self.botones.button(QDialogButtonBox.StandardButton.Ok).setEnabled(not problemas)
        return not problemas

    def producto(self) -> Producto:
        return Producto(self.codigo.text(), self.nombre.text().strip(), self.categoria.currentText().strip(),
                        self.stock.value(), self.minimo.value(), self.precio.value())


# ============================================================ Tareas en segundo plano
class SenalesTarea(QObject):
    progreso = Signal(int)
    terminado = Signal(object)
    error = Signal(str)


class Tarea(QRunnable):
    def __init__(self, funcion, *args):
        super().__init__()
        self.funcion, self.args = funcion, args
        self.senales = SenalesTarea()

    def run(self):
        try:
            self.senales.terminado.emit(self.funcion(*self.args, progreso=self.senales.progreso.emit))
        except Exception as e:  # noqa: BLE001 — se informa a la interfaz
            self.senales.error.emit(f"{type(e).__name__}: {e}")


def exportar_csv(productos: list[Producto], ruta: str, progreso) -> str:
    nombres = [f.name for f in fields(Producto)]
    with open(ruta, "w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.writer(f, delimiter=";")
        escritor.writerow(nombres)
        for i, p in enumerate(productos, start=1):
            escritor.writerow(astuple(p))
            time.sleep(0.05)                                  # Simula una operación lenta
            progreso(int(i / len(productos) * 100))
    return f"Exportados {len(productos)} productos a {Path(ruta).name}"


def importar_csv(ruta: str, progreso) -> list[Producto]:
    with open(ruta, encoding="utf-8-sig", newline="") as f:
        filas = list(csv.DictReader(f, delimiter=";"))
    productos = []
    for i, fila in enumerate(filas, start=1):
        productos.append(Producto(fila["codigo"], fila["nombre"], fila["categoria"], int(fila["stock"]),
                                  int(fila["minimo"]), float(fila["precio"])))
        progreso(int(i / len(filas) * 100))
    return productos


# ============================================================ Gráfico
class GraficoCategorias(QChartView):
    def __init__(self):
        self.grafico = QChart()
        self.grafico.setTitle("Valor del stock por categoría")
        self.grafico.legend().hide()
        super().__init__(self.grafico)
        self.setRenderHint(QPainter.RenderHint.Antialiasing)
        self.setMinimumWidth(320)

    def actualizar(self, productos: list[Producto], oscuro: bool) -> None:
        totales: dict[str, float] = {}
        for p in productos:
            totales[p.categoria] = totales.get(p.categoria, 0) + p.valor
        orden = sorted(totales, key=totales.get)
        self.grafico.removeAllSeries()
        for eje in self.grafico.axes():
            self.grafico.removeAxis(eje)
        conjunto = QBarSet("Valor")
        conjunto.append([totales[c] for c in orden])
        conjunto.setColor(QColor("#2E75B6"))
        serie = QHorizontalBarSeries()
        serie.append(conjunto)
        self.grafico.addSeries(serie)
        eje_y = QBarCategoryAxis()
        eje_y.append(orden)
        eje_x = QValueAxis()
        eje_x.setRange(0, max(totales.values(), default=1) * 1.3)
        eje_x.setLabelFormat("%.0f")                     # Formato printf: evite caracteres no ASCII (p. ej. €)
        eje_x.setTitleText("Euros")
        eje_x.setTickCount(4)
        self.grafico.addAxis(eje_y, Qt.AlignmentFlag.AlignLeft)
        self.grafico.addAxis(eje_x, Qt.AlignmentFlag.AlignBottom)
        serie.attachAxis(eje_y)
        serie.attachAxis(eje_x)
        self.grafico.setTheme(QChart.ChartTheme.ChartThemeDark if oscuro else QChart.ChartTheme.ChartThemeLight)
        conjunto.setColor(QColor("#5B9BD5" if oscuro else "#1F4E78"))


# ============================================================ Ventana principal
QSS = """
QTableView { gridline-color: palette(midlight); }
QHeaderView::section { background: palette(button); padding: 5px; border: none; font-weight: 600; }
QLabel#resumen { color: palette(placeholder-text); }
"""


def paleta(oscuro: bool) -> QPalette:
    """Paleta para el estilo Fusion: modo oscuro completo sin reescribir cada widget en QSS."""
    if not oscuro:
        return QApplication.style().standardPalette()
    p = QPalette()
    R = QPalette.ColorRole
    for rol, color in [(R.Window, "#1E2227"), (R.WindowText, "#E6E8EB"), (R.Base, "#252A30"),
                       (R.AlternateBase, "#2B3139"), (R.Text, "#E6E8EB"), (R.Button, "#2F3540"),
                       (R.ButtonText, "#E6E8EB"), (R.Highlight, "#2E75B6"), (R.HighlightedText, "#FFFFFF"),
                       (R.ToolTipBase, "#2F3540"), (R.ToolTipText, "#E6E8EB"), (R.PlaceholderText, "#8A94A3"),
                       (R.Mid, "#39404A"), (R.Midlight, "#39404A"), (R.Dark, "#15181C"), (R.Light, "#4A525E")]:
        p.setColor(rol, QColor(color))
    p.setColor(QPalette.ColorGroup.Disabled, R.ButtonText, QColor("#6B7380"))
    p.setColor(QPalette.ColorGroup.Disabled, R.Text, QColor("#6B7380"))
    p.setColor(QPalette.ColorGroup.Disabled, R.WindowText, QColor("#6B7380"))
    return p


class VentanaInventario(QMainWindow):
    def __init__(self, ruta_bd: Path = BD):
        super().__init__()
        self.setWindowTitle("Inventario")
        self.resize(1150, 620)
        self.ajustes = QSettings()
        self.pool = QThreadPool.globalInstance()
        self.tarea_actual: Tarea | None = None

        self.modelo = ModeloProductos(Almacen(ruta_bd))
        self.filtro = FiltroProductos()
        self.filtro.setSourceModel(self.modelo)

        self._crear_centro()
        self._crear_acciones()
        self._crear_panel()
        self._crear_estado()

        self.modelo.cambiado.connect(self.actualizar_resumen)
        self.tabla.selectionModel().selectionChanged.connect(self._seleccion)
        self.aplicar_tema(self.ajustes.value("tema", "claro") == "oscuro")
        if (geometria := self.ajustes.value("geometria")) is not None:
            self.restoreGeometry(geometria)
        self.actualizar_resumen()
        self._seleccion()

    # --- Construcción
    def _crear_centro(self) -> None:
        self.buscador = QLineEdit(placeholderText="Buscar por código o nombre…", clearButtonEnabled=True)
        self.combo_categoria = QComboBox()
        self.combo_categoria.addItems(["Todas", *CATEGORIAS])
        self.solo_bajo = QCheckBox("Solo bajo mínimo")
        self.buscador.textChanged.connect(lambda t: self.filtro.configurar(texto=t))
        self.combo_categoria.currentTextChanged.connect(lambda c: self.filtro.configurar(categoria=c))
        self.solo_bajo.toggled.connect(lambda v: self.filtro.configurar(solo_bajo=v))
        for senal in (self.buscador.textChanged, self.combo_categoria.currentTextChanged, self.solo_bajo.toggled):
            senal.connect(self.actualizar_resumen)

        barra = QHBoxLayout()
        barra.addWidget(self.buscador, stretch=1)
        barra.addWidget(QLabel("Categoría:"))
        barra.addWidget(self.combo_categoria)
        barra.addWidget(self.solo_bajo)

        self.tabla = QTableView()
        self.tabla.setModel(self.filtro)
        self.tabla.setSortingEnabled(True)
        self.tabla.sortByColumn(0, Qt.SortOrder.AscendingOrder)
        self.tabla.setAlternatingRowColors(True)
        self.tabla.setSelectionBehavior(QAbstractItemView.SelectionBehavior.SelectRows)
        self.tabla.setEditTriggers(QAbstractItemView.EditTrigger.DoubleClicked | QAbstractItemView.EditTrigger.EditKeyPressed)
        self.tabla.verticalHeader().hide()
        cabecera = self.tabla.horizontalHeader()
        cabecera.setSectionResizeMode(QHeaderView.ResizeMode.ResizeToContents)
        cabecera.setSectionResizeMode(1, QHeaderView.ResizeMode.Stretch)

        self.resumen = QLabel(objectName="resumen")
        centro = QWidget()
        capa = QVBoxLayout(centro)
        capa.addLayout(barra)
        capa.addWidget(self.tabla)
        capa.addWidget(self.resumen)
        self.setCentralWidget(centro)

    def _accion(self, texto, icono, atajo, funcion, consejo="") -> QAction:
        accion = QAction(self.style().standardIcon(icono), texto, self)
        if atajo:
            accion.setShortcut(atajo)
        accion.setStatusTip(consejo or texto.replace("&", "").rstrip("…"))
        accion.triggered.connect(funcion)
        return accion

    def _crear_acciones(self) -> None:
        P = QStyle.StandardPixmap
        self.acc_nuevo = self._accion("&Nuevo producto…", P.SP_FileIcon, QKeySequence.StandardKey.New, self.nuevo)
        self.acc_eliminar = self._accion("&Eliminar", P.SP_TrashIcon, QKeySequence.StandardKey.Delete, self.eliminar,
                                         "Eliminar los productos seleccionados")
        self.acc_importar = self._accion("&Importar CSV…", P.SP_DialogOpenButton, "Ctrl+I", self.importar)
        self.acc_exportar = self._accion("E&xportar CSV…", P.SP_DialogSaveButton, "Ctrl+E", self.exportar)
        self.acc_salir = self._accion("&Salir", P.SP_DialogCloseButton, QKeySequence.StandardKey.Quit, self.close)
        self.acc_oscuro = QAction("Tema &oscuro", self, checkable=True)
        self.acc_oscuro.toggled.connect(self.aplicar_tema)

        archivo = self.menuBar().addMenu("&Archivo")
        archivo.addActions([self.acc_importar, self.acc_exportar])
        archivo.addSeparator()
        archivo.addAction(self.acc_salir)
        self.menuBar().addMenu("&Edición").addActions([self.acc_nuevo, self.acc_eliminar])
        self.menu_ver = self.menuBar().addMenu("&Ver")
        self.menu_ver.addAction(self.acc_oscuro)

        barra = self.addToolBar("Principal")
        barra.setObjectName("barra_principal")              # Necesario para saveState()
        barra.setToolButtonStyle(Qt.ToolButtonStyle.ToolButtonTextBesideIcon)
        barra.addActions([self.acc_nuevo, self.acc_eliminar])
        barra.addSeparator()
        barra.addActions([self.acc_importar, self.acc_exportar])
        self.tabla.addActions([self.acc_nuevo, self.acc_eliminar])
        self.tabla.setContextMenuPolicy(Qt.ContextMenuPolicy.ActionsContextMenu)

    def _crear_panel(self) -> None:
        self.grafico = GraficoCategorias()
        panel = QDockWidget("Análisis", self, objectName="panel_analisis")
        panel.setWidget(self.grafico)
        self.addDockWidget(Qt.DockWidgetArea.RightDockWidgetArea, panel)
        self.menu_ver.addAction(panel.toggleViewAction())

    def _crear_estado(self) -> None:
        self.progreso = QProgressBar(maximumWidth=180, visible=False)
        self.indicador_bajo = QLabel()
        self.statusBar().addPermanentWidget(self.progreso)
        self.statusBar().addPermanentWidget(self.indicador_bajo)

    # --- Lógica de interfaz
    def actualizar_resumen(self, *_):
        productos = self.modelo.productos
        visibles = self.filtro.rowCount()
        bajos = sum(p.bajo_minimo for p in productos)
        total = sum(p.valor for p in productos)
        self.resumen.setText(f"Mostrando {visibles} de {len(productos)} productos · valor total del inventario: "
                             + f"{total:,.2f} €".replace(",", "X").replace(".", ",").replace("X", "."))
        self.indicador_bajo.setText(f"⚠ {bajos} bajo mínimo" if bajos else "✔ Stock correcto")
        self.indicador_bajo.setStyleSheet(f"color: {'#D92D20' if bajos else '#079455'}; padding: 0 8px;")
        self.grafico.actualizar(productos, self.acc_oscuro.isChecked())

    def _seleccion(self, *_):
        self.acc_eliminar.setEnabled(self.tabla.selectionModel().hasSelection())

    def filas_seleccionadas(self) -> list[int]:
        return sorted({self.filtro.mapToSource(i).row() for i in self.tabla.selectionModel().selectedRows()})

    def nuevo(self) -> None:
        dialogo = DialogoProducto(self, self.modelo.existe)
        if dialogo.exec() == QDialog.DialogCode.Accepted:
            p = dialogo.producto()
            self.modelo.agregar(p)
            self.statusBar().showMessage(f"Producto {p.codigo} añadido", 4000)

    def eliminar(self) -> None:
        filas = self.filas_seleccionadas()
        if not filas:
            return
        nombres = ", ".join(self.modelo.productos[f].codigo for f in filas[:5]) + ("…" if len(filas) > 5 else "")
        if QMessageBox.question(self, "Eliminar", f"¿Eliminar {len(filas)} producto(s)?\n{nombres}") \
                == QMessageBox.StandardButton.Yes:
            self.modelo.eliminar(filas)
            self.statusBar().showMessage(f"{len(filas)} producto(s) eliminados", 4000)

    def exportar(self, ruta: str | None = None) -> None:
        ruta = ruta or QFileDialog.getSaveFileName(self, "Exportar inventario", "inventario.csv", "CSV (*.csv)")[0]
        if ruta:
            self._lanzar(Tarea(exportar_csv, list(self.modelo.productos), ruta), self._mensaje)

    def importar(self, ruta: str | None = None) -> None:
        ruta = ruta or QFileDialog.getOpenFileName(self, "Importar inventario", "", "CSV (*.csv)")[0]
        if ruta:
            self._lanzar(Tarea(importar_csv, ruta), self._importados)

    def _mensaje(self, texto: str) -> None:
        self.statusBar().showMessage(texto, 5000)

    def _error(self, texto: str) -> None:
        QMessageBox.critical(self, "Error", texto)

    def _importados(self, productos: list[Producto]) -> None:
        self.modelo.almacen.guardar_todos(productos)
        self.modelo.recargar()
        self.statusBar().showMessage(f"Importados {len(productos)} productos (los códigos existentes se actualizan)", 5000)

    def _lanzar(self, tarea: Tarea, al_terminar) -> None:
        for accion in (self.acc_importar, self.acc_exportar):
            accion.setEnabled(False)
        self.progreso.setValue(0)
        self.progreso.setVisible(True)
        tarea.senales.progreso.connect(self.progreso.setValue)
        tarea.senales.terminado.connect(al_terminar)
        tarea.senales.error.connect(self._error)       # Métodos (no lambdas): se ejecutan en el hilo principal
        for senal in (tarea.senales.terminado, tarea.senales.error):
            senal.connect(self._fin_tarea)
        self.tarea_actual = tarea                          # Mantener viva la tarea (y sus señales) hasta el final
        self.pool.start(tarea)

    def _fin_tarea(self, *_):
        self.tarea_actual = None
        self.progreso.setVisible(False)
        for accion in (self.acc_importar, self.acc_exportar):
            accion.setEnabled(True)

    def aplicar_tema(self, oscuro: bool) -> None:
        if self.acc_oscuro.isChecked() != oscuro:
            self.acc_oscuro.setChecked(oscuro)               # Vuelve a llamar a este método
            return
        app = QApplication.instance()
        app.setPalette(paleta(oscuro))
        app.setStyleSheet(QSS)
        self.ajustes.setValue("tema", "oscuro" if oscuro else "claro")
        self.actualizar_resumen()

    def closeEvent(self, evento) -> None:
        self.pool.waitForDone(3000)                           # No cerrar con una exportación a medias
        self.ajustes.setValue("geometria", self.saveGeometry())
        evento.accept()


def main() -> int:
    app = QApplication(sys.argv)
    app.setOrganizationName("MiEmpresa")
    app.setApplicationName("Inventario")
    app.setStyle("Fusion")
    ventana = VentanaInventario()
    ventana.show()
    return app.exec()


if __name__ == "__main__":
    sys.exit(main())
```

Aspectos destacados:

- El modelo devuelve el valor numérico en `UserRole` y el proxy ordena por ese rol: «1.100,00 €» se ordena correctamente frente a «219,00 €».
- `setData` emite `dataChanged` para **toda la fila**: al editar el stock se recalculan el color y la columna *Valor*.
- El filtro combinado se implementa en `filterAcceptsRow` y se recalcula con `beginFilterChange()`/`endFilterChange()` (en versiones anteriores a 6.10 use `invalidateFilter()`).
- Las tareas en segundo plano no tocan la interfaz: comunican progreso y resultado mediante señales conectadas a **métodos** de la ventana (una `lambda` sin receptor se ejecutaría en el hilo de trabajo).

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `QWidget: Must construct a QApplication before a QWidget` | Se creó un *widget* antes de `QApplication`. | Cree `QApplication` al inicio. |
| La ventana aparece y desaparece | No se guarda la referencia y el recolector la destruye. | Asígnela a una variable o atributo (`self.ventana`). |
| `RuntimeError: Internal C++ object … already deleted` | Se usa un objeto Qt ya destruido (p. ej., tras `deleteLater`). | No conserve referencias a objetos eliminados; compruebe el ciclo de vida. |
| La interfaz se congela | Trabajo pesado en el hilo principal. | `QThreadPool`/`QThread` y señales. |
| Bloqueo o cierre al actualizar desde otro hilo | Se modifican *widgets* fuera del hilo principal. | Emita señales; la ranura actualiza la interfaz. |
| Una `lambda` conectada a una señal emitida desde otro hilo no actualiza la interfaz | Sin objeto receptor, la `lambda` se ejecuta en el hilo emisor. | Conecte a métodos de un `QObject` del hilo principal. |
| La tabla no se actualiza tras cambiar los datos | No se notificó a la vista. | `beginInsertRows`/`endInsertRows`, `dataChanged.emit`, `beginResetModel`. |
| Ordenación alfabética de números («10» < «9») | El proxy ordena por `DisplayRole` formateado. | `proxy.setSortRole(...)` a un rol con el valor numérico. |
| Fila equivocada al eliminar con filtro | Se usan índices del proxy sobre el modelo de origen. | `proxy.mapToSource(indice)`. |
| `AttributeError: … has no attribute 'AlignRight'` | Enumeraciones antiguas (Qt 5). | Nombres completos: `Qt.AlignmentFlag.AlignRight`. |
| `QSettings` devuelve `'true'` (texto) | En Linux/INI los valores se leen como texto. | `ajustes.value("clave", False, type=bool)`. |
| `saveState()` avisa por `objectName` | Barras/paneles sin nombre. | `setObjectName(...)` en barras y paneles. |
| `qt.qpa.plugin: Could not load the Qt platform plugin "xcb"` | Faltan bibliotecas del sistema o no hay pantalla. | Instale `libxcb-cursor0` (Linux) o use `QT_QPA_PLATFORM=offscreen` en servidores. |

---

## 22. Buenas prácticas

- Separe la lógica de negocio (sin Qt) de la interfaz: facilita las pruebas y la reutilización.
- Use **modelo/vista** para tablas con datos reales; `QTableWidget` solo para casos pequeños.
- Centralice las operaciones en **acciones** (`QAction`) compartidas por menús, barras y menús contextuales, y active/desactive según el contexto.
- Nunca bloquee el bucle de eventos; comunique hilos mediante señales.
- Use disposiciones (*layouts*), no posiciones absolutas: la interfaz se adaptará a tamaños, escalas e idiomas.
- Guarde preferencias y geometría con `QSettings`; asigne `objectName` a barras y paneles.
- Valide los formularios en tiempo real y desactive *Aceptar* hasta que los datos sean correctos.
- Utilice `app.setStyle("Fusion")` y QSS moderado para un aspecto coherente; evite estilos excesivos que rompan la accesibilidad.
- Automatice pruebas con pytest-qt en modo `offscreen`.

---

## 23. Referencia rápida

```python
app = QApplication(sys.argv); app.setStyle("Fusion"); ventana.show(); sys.exit(app.exec())
objeto.senal.connect(funcion) · Signal(int, str) · senal.emit(...) · QTimer.singleShot(ms, f)
QVBoxLayout(widget) · QHBoxLayout · QGridLayout.addWidget(w, fila, col, nfil, ncol) · QFormLayout.addRow("Etiqueta", w)
QMainWindow: setCentralWidget · menuBar().addMenu · addToolBar · statusBar().showMessage · addDockWidget
QAction(icono, "Texto", self, shortcut=, checkable=) .triggered .toggled · self.style().standardIcon(QStyle.StandardPixmap.SP_…)
QMessageBox.question/information/warning/critical · QFileDialog.getOpenFileName/getSaveFileName · QInputDialog.getText
QDialog.exec() == QDialog.DialogCode.Accepted · QDialogButtonBox(Ok | Cancel).accepted/.rejected
QAbstractTableModel: rowCount, columnCount, data(indice, rol), headerData, flags, setData · beginInsertRows/endInsertRows
QSortFilterProxyModel: setSourceModel, setFilterFixedString, setSortRole, filterAcceptsRow, mapToSource
QTableView: setModel, setSortingEnabled, setSelectionBehavior, selectionModel().selectedRows()
QThreadPool.globalInstance().start(QRunnable) · QThread + moveToThread · QSettings().setValue/value(type=)
```

---

## 24. Ejercicios propuestos

1. Añada al inventario un historial de movimientos (entradas y salidas) en una segunda pestaña con su propio modelo.
2. Sustituya el modelo propio por `QSqlTableModel` y compare el código necesario.
3. Implemente un delegado (`QStyledItemDelegate`) que muestre el stock como una barra de progreso respecto al mínimo.
4. Diseñe en Qt Designer el diálogo de alta, cárguelo con `pyside6-uic` y conecte su validación.
5. Escriba pruebas con pytest-qt que verifiquen el filtrado, la validación del diálogo y la exportación.

---

## 25. Referencias

- Documentación oficial de Qt for Python: https://doc.qt.io/qtforpython-6/
- Tutoriales oficiales: https://doc.qt.io/qtforpython-6/tutorials/index.html
- Programación modelo/vista: https://doc.qt.io/qt-6/model-view-programming.html
- Referencia de hojas de estilo QSS: https://doc.qt.io/qt-6/stylesheet-reference.html
- pytest-qt: https://pytest-qt.readthedocs.io/
