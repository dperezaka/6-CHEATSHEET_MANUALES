# Manual de uso: PySide6

> **Categoría:** Interfaces gráficas (GUI) · **Utilidad:** Interfaces gráficas con Qt
> **Tipo:** Librería externa · Binding **oficial** de Qt 6 (The Qt Company), licencia **LGPL v3**

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Widgets y layouts](#5-widgets-y-layouts)
6. [Señales, slots y propiedades](#6-señales-slots-y-propiedades)
7. [Ventana principal y navegación](#7-ventana-principal-y-navegación)
8. [Diálogos](#8-diálogos)
9. [Bases de datos con QtSql](#9-bases-de-datos-con-qtsql)
10. [Gráficos con QtCharts](#10-gráficos-con-qtcharts)
11. [Estilos, temas e iconos](#11-estilos-temas-e-iconos)
12. [Concurrencia](#12-concurrencia)
13. [Qt Designer y archivos .ui](#13-qt-designer-y-archivos-ui)
14. [Recursos, configuración y traducción](#14-recursos-configuración-y-traducción)
15. [Introducción a Qt Quick (QML)](#15-introducción-a-qt-quick-qml)
16. [Distribución](#16-distribución)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**PySide6** (también llamado *Qt for Python*) es el binding **oficial** de Qt 6 para Python, desarrollado por The Qt Company. Permite crear aplicaciones de escritorio profesionales y multiplataforma con todo el ecosistema Qt: widgets, modelo/vista, gráficos, bases de datos, multimedia, red, impresión, navegador web y Qt Quick (QML).

**Por qué elegir PySide6:**
- **Licencia LGPL**: permite desarrollar y distribuir aplicaciones comerciales de código cerrado (cumpliendo las condiciones de la LGPL), sin comprar licencia.
- Mantenido por los creadores de Qt, sincronizado con cada versión de Qt.
- El paquete incluye los módulos **QtCharts**, **QtSql**, **QtWebEngine**, **QtMultimedia**… y herramientas como **Qt Designer** (`pyside6-designer`) y el empaquetador `pyside6-deploy`.
- API prácticamente idéntica a PyQt6: la documentación y los ejemplos de uno sirven para el otro.

**Diferencias con PyQt6:**

| PyQt6 | PySide6 |
|---|---|
| `pyqtSignal`, `pyqtSlot`, `pyqtProperty` | `Signal`, `Slot`, `Property` |
| `from PyQt6 import uic; uic.loadUi()` | `QUiLoader().load()` |
| `pyuic6` | `pyside6-uic` |
| `pyrcc6` (eliminado en PyQt6) | `pyside6-rcc` |
| GPL / comercial | LGPL |

> Este manual complementa al de **PyQt6** (cuyos conceptos de widgets, layouts, modelo/vista y QSS son aplicables). Aquí se profundiza en lo específico de PySide6 y en **QtSql**, **QtCharts**, navegación y despliegue.

---

## 2. Instalación e importación

```bash
pip install pyside6
```

El paquete incluye herramientas de línea de comandos:

| Comando | Uso |
|---|---|
| `pyside6-designer` | Qt Designer (diseño visual de ventanas). |
| `pyside6-uic` | Convierte `.ui` en código Python. |
| `pyside6-rcc` | Compila archivos de recursos `.qrc`. |
| `pyside6-lupdate` / `pyside6-linguist` | Traducciones. |
| `pyside6-deploy` | Empaqueta la aplicación en un ejecutable. |
| `pyside6-qml` | Ejecuta archivos QML. |

> Si solo necesita los módulos básicos y quiere una instalación más ligera: `pip install pyside6-essentials` (sin `pyside6-addons`).

Importaciones habituales:

```python
import sys
from PySide6.QtWidgets import (
    QApplication, QMainWindow, QWidget, QDialog, QLabel, QPushButton, QLineEdit, QComboBox,
    QDateEdit, QDoubleSpinBox, QVBoxLayout, QHBoxLayout, QFormLayout, QGridLayout,
    QTableView, QHeaderView, QMessageBox, QFileDialog, QStackedWidget, QListWidget,
)
from PySide6.QtCore import Qt, Signal, Slot, Property, QObject, QTimer, QDate, QSettings, QThreadPool, QRunnable
from PySide6.QtGui import QAction, QIcon, QKeySequence, QColor, QFont
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `QApplication` | Gestiona la aplicación y el bucle de eventos (una sola instancia). |
| `QWidget` / `QMainWindow` / `QDialog` | Ventanas y componentes visuales. |
| Layouts | Organizan y adaptan los widgets al tamaño. |
| `Signal` / `Slot` | Comunicación entre objetos. |
| `Property` | Propiedades Qt (usadas en QML y animaciones). |
| Modelo/vista | Datos (modelo) separados de su presentación (vista). |
| `QtSql` | Acceso a bases de datos con modelos listos para tablas. |
| `QtCharts` | Gráficos integrados en la interfaz. |
| Qt Quick / QML | Lenguaje declarativo para interfaces fluidas y táctiles. |

**Enumeraciones:** igual que en PyQt6, use el nombre completo (`Qt.AlignmentFlag.AlignCenter`). PySide6 también acepta la forma corta (`Qt.AlignCenter`) por compatibilidad, pero se recomienda la completa.

---

## 4. Primera aplicación

```python
import sys
from PySide6.QtWidgets import QApplication, QWidget, QVBoxLayout, QLabel, QLineEdit, QPushButton


class Conversor(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Conversor de temperatura")
        self.celsius = QLineEdit(placeholderText="Grados Celsius")
        self.resultado = QLabel("—")
        boton = QPushButton("Convertir")
        boton.clicked.connect(self.convertir)
        self.celsius.returnPressed.connect(self.convertir)

        layout = QVBoxLayout(self)
        layout.addWidget(self.celsius)
        layout.addWidget(boton)
        layout.addWidget(self.resultado)

    def convertir(self):
        try:
            c = float(self.celsius.text().replace(",", "."))
        except ValueError:
            self.resultado.setText("Introduzca un número válido")
            return
        self.resultado.setText(f"{c * 9 / 5 + 32:.1f} °F")


if __name__ == "__main__":
    app = QApplication(sys.argv)
    ventana = Conversor()
    ventana.show()
    sys.exit(app.exec())
```

---

## 5. Widgets y layouts

Los widgets y layouts son los mismos que en PyQt6. Resumen de los más usados:

| Categoría | Widgets |
|---|---|
| Texto | `QLabel`, `QLineEdit`, `QTextEdit`, `QPlainTextEdit` |
| Botones | `QPushButton`, `QToolButton`, `QCheckBox`, `QRadioButton` |
| Selección | `QComboBox`, `QSpinBox`, `QDoubleSpinBox`, `QSlider`, `QDial` |
| Fechas | `QDateEdit`, `QTimeEdit`, `QDateTimeEdit`, `QCalendarWidget` |
| Listas y tablas | `QListWidget`, `QTableWidget`, `QTreeWidget`, `QListView`, `QTableView`, `QTreeView` |
| Contenedores | `QGroupBox`, `QTabWidget`, `QStackedWidget`, `QScrollArea`, `QSplitter`, `QDockWidget`, `QFrame` |
| Progreso | `QProgressBar`, `QProgressDialog` |

Layouts: `QVBoxLayout`, `QHBoxLayout`, `QGridLayout`, `QFormLayout` (ver manual de PyQt6, sección 6).

### 5.1 Fechas

```python
fecha = QDateEdit(calendarPopup=True, displayFormat="dd/MM/yyyy")
fecha.setDate(QDate.currentDate())
fecha.setMaximumDate(QDate.currentDate())
fecha.date().toPython()                # → datetime.date  (en PyQt6: toPyDate())

calendario = QCalendarWidget()
calendario.selectionChanged.connect(lambda: print(calendario.selectedDate().toString("dd/MM/yyyy")))
```

> PySide6 usa `toPython()` para convertir tipos de Qt (`QDate`, `QDateTime`, `QTime`) a tipos de Python.

### 5.2 Campo numérico con formato español

```python
from PySide6.QtCore import QLocale

importe = QDoubleSpinBox(minimum=0, maximum=1_000_000, decimals=2, suffix=" €")
importe.setLocale(QLocale(QLocale.Language.Spanish, QLocale.Country.Spain))   # Coma decimal
importe.setGroupSeparatorShown(True)                                          # 1.234,56 €
```

Establecer el idioma por defecto de toda la aplicación:

```python
QLocale.setDefault(QLocale(QLocale.Language.Spanish, QLocale.Country.Spain))
```

---

## 6. Señales, slots y propiedades

### 6.1 Señales propias

```python
from PySide6.QtCore import QObject, Signal, Slot


class Contador(QObject):
    valor_cambiado = Signal(int)
    limite_alcanzado = Signal()

    def __init__(self, limite: int = 10):
        super().__init__()
        self._valor = 0
        self.limite = limite

    @Slot()
    def incrementar(self):
        self._valor += 1
        self.valor_cambiado.emit(self._valor)
        if self._valor >= self.limite:
            self.limite_alcanzado.emit()


contador = Contador(3)
contador.valor_cambiado.connect(lambda v: print("Valor:", v))
contador.limite_alcanzado.connect(lambda: print("¡Límite!"))
boton.clicked.connect(contador.incrementar)
```

Señales con varios argumentos o sobrecargas:

```python
guardado = Signal(str, float)                  # emit("TEC-01", 59.9)
datos_listos = Signal(list)
mensaje = Signal((str,), (str, int))           # Sobrecarga
```

> Las señales se declaran como **atributos de clase** de una clase que herede de `QObject`. El decorador `@Slot()` es opcional, pero mejora el rendimiento y es obligatorio para exponer métodos a QML.

### 6.2 Tipos de conexión

```python
emisor.senal.connect(receptor.metodo)                                    # Automática
emisor.senal.connect(receptor.metodo, Qt.ConnectionType.QueuedConnection)  # En el bucle de eventos del receptor
emisor.senal.connect(receptor.metodo, Qt.ConnectionType.SingleShotConnection)   # Solo una vez (Qt 6)
```

### 6.3 Propiedades (`Property`)

```python
from PySide6.QtCore import Property


class Termostato(QObject):
    temperatura_cambiada = Signal(float)

    def __init__(self):
        super().__init__()
        self._temperatura = 21.0

    def _leer(self) -> float:
        return self._temperatura

    def _escribir(self, valor: float) -> None:
        if valor != self._temperatura:
            self._temperatura = valor
            self.temperatura_cambiada.emit(valor)

    temperatura = Property(float, _leer, _escribir, notify=temperatura_cambiada)
```

Las propiedades Qt permiten animarlas con `QPropertyAnimation` y enlazarlas desde QML.

### 6.4 Animaciones

```python
from PySide6.QtCore import QPropertyAnimation, QEasingCurve, QRect

animacion = QPropertyAnimation(panel, b"geometry")
animacion.setDuration(300)
animacion.setStartValue(QRect(0, 0, 0, 400))
animacion.setEndValue(QRect(0, 0, 220, 400))
animacion.setEasingCurve(QEasingCurve.Type.OutCubic)
animacion.start()
```

---

## 7. Ventana principal y navegación

### 7.1 Navegación con barra lateral y `QStackedWidget`

Patrón habitual en aplicaciones modernas: una lista de secciones a la izquierda y las páginas a la derecha.

```python
class Principal(QMainWindow):
    def __init__(self):
        super().__init__()
        self.menu_lateral = QListWidget()
        self.menu_lateral.setFixedWidth(180)
        self.paginas = QStackedWidget()

        for nombre, pagina in [("Inicio", PaginaInicio()), ("Clientes", PaginaClientes()),
                               ("Informes", PaginaInformes())]:
            self.menu_lateral.addItem(nombre)
            self.paginas.addWidget(pagina)

        self.menu_lateral.currentRowChanged.connect(self.paginas.setCurrentIndex)
        self.menu_lateral.setCurrentRow(0)

        central = QWidget()
        layout = QHBoxLayout(central)
        layout.setContentsMargins(0, 0, 0, 0)
        layout.addWidget(self.menu_lateral)
        layout.addWidget(self.paginas, 1)
        self.setCentralWidget(central)
```

### 7.2 Paneles acoplables (`QDockWidget`)

```python
from PySide6.QtWidgets import QDockWidget

panel = QDockWidget("Propiedades", self)
panel.setWidget(QLabel("Contenido del panel"))
panel.setObjectName("panel_propiedades")            # Necesario para saveState/restoreState
self.addDockWidget(Qt.DockWidgetArea.RightDockWidgetArea, panel)
self.menuBar().addMenu("Ver").addAction(panel.toggleViewAction())    # Mostrar/ocultar
```

### 7.3 Acciones, menús y barra de estado

Igual que en PyQt6:

```python
accion = QAction(QIcon.fromTheme("document-save"), "&Guardar", self)
accion.setShortcut(QKeySequence.StandardKey.Save)
accion.triggered.connect(self.guardar)
self.menuBar().addMenu("&Archivo").addAction(accion)
self.addToolBar("Principal").addAction(accion)
self.statusBar().showMessage("Listo", 3000)
```

### 7.4 Icono en la bandeja del sistema

```python
from PySide6.QtWidgets import QSystemTrayIcon, QMenu

bandeja = QSystemTrayIcon(QIcon("icono.png"), app)
menu = QMenu()
menu.addAction("Mostrar", ventana.show)
menu.addAction("Salir", app.quit)
bandeja.setContextMenu(menu)
bandeja.show()
bandeja.showMessage("Recordatorio", "Tiene 3 tareas pendientes", QSystemTrayIcon.MessageIcon.Information, 5000)
```

---

## 8. Diálogos

```python
QMessageBox.information(self, "Información", "Operación completada.")
r = QMessageBox.question(self, "Confirmar", "¿Eliminar el registro?")
if r == QMessageBox.StandardButton.Yes: ...

ruta, _ = QFileDialog.getOpenFileName(self, "Abrir", "", "CSV (*.csv)")
ruta, _ = QFileDialog.getSaveFileName(self, "Guardar", "informe.pdf", "PDF (*.pdf)")
```

### 8.1 Diálogo de progreso

```python
from PySide6.QtWidgets import QProgressDialog

progreso = QProgressDialog("Procesando archivos…", "Cancelar", 0, len(archivos), self)
progreso.setWindowModality(Qt.WindowModality.WindowModal)
progreso.setMinimumDuration(500)                  # Solo aparece si tarda más de 0,5 s
for i, archivo in enumerate(archivos):
    if progreso.wasCanceled():
        break
    procesar(archivo)
    progreso.setValue(i + 1)
    QApplication.processEvents()
```

### 8.2 Botones traducidos

Los botones estándar (*Yes*, *No*, *Cancel*) se muestran en el idioma del sistema si se carga la traducción de Qt:

```python
from PySide6.QtCore import QTranslator, QLibraryInfo

traductor = QTranslator()
traductor.load("qtbase_es", QLibraryInfo.path(QLibraryInfo.LibraryPath.TranslationsPath))
app.installTranslator(traductor)
```

---

## 9. Bases de datos con QtSql

`QtSql` conecta con SQLite, PostgreSQL, MySQL y ODBC, y ofrece **modelos listos** para mostrar y editar tablas.

### 9.1 Conectar

```python
from PySide6.QtSql import QSqlDatabase, QSqlQuery

bd = QSqlDatabase.addDatabase("QSQLITE")       # QPSQL, QMYSQL, QODBC…
bd.setDatabaseName("gastos.db")
if not bd.open():
    raise RuntimeError(bd.lastError().text())

QSqlDatabase.drivers()                         # Controladores disponibles
```

> Debe existir una `QApplication` (o `QCoreApplication`) antes de crear la conexión. Los controladores de PostgreSQL y MySQL requieren las librerías cliente instaladas en el sistema.

### 9.2 Consultas con `QSqlQuery`

```python
consulta = QSqlQuery()
consulta.exec("""
    CREATE TABLE IF NOT EXISTS gastos (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        fecha TEXT NOT NULL, concepto TEXT NOT NULL,
        categoria TEXT NOT NULL, importe REAL NOT NULL CHECK (importe > 0))
""")

# Parámetros (evitan inyección SQL)
consulta.prepare("INSERT INTO gastos (fecha, concepto, categoria, importe) VALUES (?, ?, ?, ?)")
for valor in ("2026-11-24", "Material de oficina", "Oficina", 45.90):
    consulta.addBindValue(valor)
if not consulta.exec():
    print(consulta.lastError().text())

consulta.prepare("SELECT categoria, SUM(importe) FROM gastos WHERE fecha >= :desde GROUP BY categoria")
consulta.bindValue(":desde", "2026-01-01")
consulta.exec()
while consulta.next():
    print(consulta.value(0), consulta.value(1))
```

### 9.3 `QSqlTableModel`: tabla editable sin escribir SQL

```python
from PySide6.QtSql import QSqlTableModel

modelo = QSqlTableModel(db=bd)
modelo.setTable("gastos")
modelo.setEditStrategy(QSqlTableModel.EditStrategy.OnManualSubmit)
modelo.setSort(1, Qt.SortOrder.DescendingOrder)          # Ordenar por fecha
modelo.setFilter("categoria = 'Viajes'")                 # WHERE (sin la palabra WHERE)
modelo.select()

for col, titulo in enumerate(["Id", "Fecha", "Concepto", "Categoría", "Importe"]):
    modelo.setHeaderData(col, Qt.Orientation.Horizontal, titulo)

vista = QTableView()
vista.setModel(modelo)
vista.hideColumn(0)

# Guardar o descartar los cambios hechos en la tabla
modelo.submitAll()
modelo.revertAll()
```

| Estrategia | Cuándo se guardan los cambios |
|---|---|
| `OnFieldChange` | Al cambiar cada celda. |
| `OnRowChange` | Al cambiar de fila. |
| `OnManualSubmit` | Al llamar a `submitAll()` (recomendado: permite deshacer). |

Insertar y eliminar registros con el modelo:

```python
registro = modelo.record()
registro.setGenerated("id", False)          # Lo genera la base de datos
registro.setValue("fecha", "2026-11-24")
registro.setValue("concepto", "Taxi aeropuerto")
registro.setValue("categoria", "Viajes")
registro.setValue("importe", 32.50)
modelo.insertRecord(-1, registro)
modelo.submitAll()

modelo.removeRow(fila)
modelo.submitAll()
```

> Si `submitAll()` devuelve `False`, consulte `modelo.lastError().text()`.

### 9.4 Otros modelos

| Modelo | Uso |
|---|---|
| `QSqlQueryModel` | Resultado de una consulta (solo lectura), útil para informes. |
| `QSqlTableModel` | Una tabla editable. |
| `QSqlRelationalTableModel` | Tabla con claves foráneas: muestra el nombre relacionado y ofrece desplegables. |

```python
from PySide6.QtSql import QSqlQueryModel
resumen = QSqlQueryModel()
resumen.setQuery("SELECT categoria, ROUND(SUM(importe), 2) FROM gastos GROUP BY categoria ORDER BY 2 DESC", bd)
```

---

## 10. Gráficos con QtCharts

`QtCharts` está incluido en PySide6 y permite gráficos interactivos integrados en la interfaz.

### 10.1 Gráfico circular

```python
from PySide6.QtCharts import QChart, QChartView, QPieSeries
from PySide6.QtGui import QPainter

serie = QPieSeries()
for categoria, total in [("Oficina", 320.5), ("Viajes", 845.0), ("Comidas", 410.2)]:
    porcion = serie.append(categoria, total)
    porcion.setLabel(f"{categoria} ({total:.0f} €)")
serie.setLabelsVisible(True)
serie.setHoleSize(0.35)                     # Gráfico de anillo

grafico = QChart()
grafico.addSeries(serie)
grafico.setTitle("Gastos por categoría")
grafico.setAnimationOptions(QChart.AnimationOption.SeriesAnimations)
grafico.legend().setAlignment(Qt.AlignmentFlag.AlignRight)

vista = QChartView(grafico)
vista.setRenderHint(QPainter.RenderHint.Antialiasing)
layout.addWidget(vista)
```

### 10.2 Gráfico de barras

```python
from PySide6.QtCharts import QBarSeries, QBarSet, QBarCategoryAxis, QValueAxis

conjunto = QBarSet("Gastos")
conjunto.append([520, 610, 480, 730])
conjunto.setColor(QColor("#1F4E78"))
serie = QBarSeries()
serie.append(conjunto)

grafico = QChart()
grafico.addSeries(serie)
eje_x = QBarCategoryAxis()
eje_x.append(["Ago", "Sep", "Oct", "Nov"])
eje_y = QValueAxis()
eje_y.setLabelFormat("%.0f €")
grafico.addAxis(eje_x, Qt.AlignmentFlag.AlignBottom)
grafico.addAxis(eje_y, Qt.AlignmentFlag.AlignLeft)
serie.attachAxis(eje_x)
serie.attachAxis(eje_y)
grafico.legend().hide()
```

### 10.3 Gráfico de líneas

```python
from PySide6.QtCharts import QLineSeries

linea = QLineSeries(name="Acumulado")
for i, valor in enumerate([520, 1130, 1610, 2340]):
    linea.append(i, valor)
```

### 10.4 Actualizar un gráfico

```python
serie.clear()
for categoria, total in nuevos_datos:
    serie.append(categoria, total)
```

### 10.5 Tema del gráfico y exportación

```python
grafico.setTheme(QChart.ChartTheme.ChartThemeDark)          # Light, BlueCerulean, Dark, Qt…
vista.grab().save("grafico.png")                            # Guardar como imagen
```

---

## 11. Estilos, temas e iconos

### 11.1 Estilo y esquema de color

```python
app.setStyle("Fusion")
app.styleHints().setColorScheme(Qt.ColorScheme.Dark)        # Qt 6.8+: Light / Dark / Unknown (sistema)
```

### 11.2 Hojas de estilo QSS

```python
app.setStyleSheet(Path("estilo.qss").read_text(encoding="utf-8"))
```

```css
/* estilo.qss */
QListWidget#menuLateral { background: #1F2A37; color: #E5E7EB; border: none; font-size: 11pt; }
QListWidget#menuLateral::item { padding: 10px 16px; }
QListWidget#menuLateral::item:selected { background: #2563EB; color: white; }
QPushButton { padding: 6px 14px; border-radius: 4px; background: #2563EB; color: white; border: none; }
QPushButton:hover { background: #1D4ED8; }
QPushButton[secundario="true"] { background: transparent; color: #2563EB; border: 1px solid #2563EB; }
QLabel#titulo { font-size: 18pt; font-weight: bold; }
```

```python
boton.setProperty("secundario", True)        # Selector por propiedad
etiqueta.setObjectName("titulo")
```

### 11.3 Iconos

```python
QIcon("iconos/guardar.svg")                                  # SVG escala sin perder calidad
QIcon.fromTheme("document-save")                             # Iconos del sistema (Linux)
QIcon.fromTheme(QIcon.ThemeIcon.DocumentSave)                # Qt 6.7+: iconos estándar multiplataforma
self.style().standardIcon(QStyle.StandardPixmap.SP_DialogSaveButton)   # Iconos del estilo
```

---

## 12. Concurrencia

### 12.1 `QThreadPool` + `QRunnable` con señales

```python
class Senales(QObject):
    resultado = Signal(object)
    error = Signal(str)
    progreso = Signal(int)


class Tarea(QRunnable):
    def __init__(self, funcion, *args, **kwargs):
        super().__init__()
        self.funcion, self.args, self.kwargs = funcion, args, kwargs
        self.senales = Senales()

    @Slot()
    def run(self):
        try:
            self.senales.resultado.emit(self.funcion(*self.args, progreso=self.senales.progreso, **self.kwargs))
        except Exception as e:
            self.senales.error.emit(str(e))


def importar_csv(ruta, progreso):
    filas = Path(ruta).read_text(encoding="utf-8").splitlines()
    for i, _ in enumerate(filas, 1):
        progreso.emit(int(i * 100 / len(filas)))
    return filas


tarea = Tarea(importar_csv, "datos.csv")
tarea.senales.progreso.connect(barra.setValue)
tarea.senales.resultado.connect(self.mostrar)
QThreadPool.globalInstance().start(tarea)
```

> Nunca modifique widgets desde otro hilo; emita señales y deje que el hilo principal actualice la interfaz.

### 12.2 `QThread` con trabajador

El patrón `QObject.moveToThread` es el mismo que en PyQt6 (sección 12 de su manual), cambiando `pyqtSignal` por `Signal`.

### 12.3 Integración con asyncio (`QtAsyncio`)

```python
import PySide6.QtAsyncio as QtAsyncio

async def cargar():
    ...

QtAsyncio.run(cargar(), handle_sigint=True)     # Bucle de eventos de Qt compatible con asyncio
```

---

## 13. Qt Designer y archivos .ui

```bash
pyside6-designer                   # Abre Qt Designer
```

### 13.1 Cargar el `.ui` en tiempo de ejecución

```python
from PySide6.QtUiTools import QUiLoader
from PySide6.QtCore import QFile

cargador = QUiLoader()
archivo = QFile("ventana.ui")
archivo.open(QFile.OpenModeFlag.ReadOnly)
ventana = cargador.load(archivo)
archivo.close()

ventana.botonGuardar.clicked.connect(guardar)     # Acceso a los widgets por su objectName
ventana.show()
```

> `QUiLoader` devuelve un widget nuevo (no rellena `self` como `uic.loadUi` de PyQt). Para usarlo dentro de una clase, añada el widget cargado al layout de su ventana.

### 13.2 Generar código Python (recomendado)

```bash
pyside6-uic ventana.ui -o ui_ventana.py
```

```python
from ui_ventana import Ui_MainWindow


class Ventana(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.botonGuardar.clicked.connect(self.guardar)
```

### 13.3 Proyectos con `pyproject.toml`

Qt Creator y `pyside6-project` gestionan proyectos que convierten automáticamente `.ui`, `.qrc` y traducciones:

```bash
pyside6-project build
pyside6-project run
```

---

## 14. Recursos, configuración y traducción

### 14.1 Recursos embebidos (`.qrc`)

Para incluir iconos e imágenes dentro de la aplicación:

```xml
<!-- recursos.qrc -->
<RCC>
  <qresource prefix="/">
    <file>iconos/guardar.svg</file>
    <file>iconos/logo.png</file>
  </qresource>
</RCC>
```

```bash
pyside6-rcc recursos.qrc -o recursos_rc.py
```

```python
import recursos_rc                       # noqa: F401 — registra los recursos
QIcon(":/iconos/guardar.svg")            # Ruta de recurso con ":"
```

### 14.2 Configuración persistente

```python
app.setOrganizationName("MiEmpresa")
app.setApplicationName("Gastos")
ajustes = QSettings()
ajustes.setValue("ventana/geometria", ventana.saveGeometry())
ventana.restoreGeometry(ajustes.value("ventana/geometria", b""))
```

### 14.3 Traducción de la interfaz

```python
self.setWindowTitle(self.tr("Expenses"))          # Textos marcados con tr()
```

```bash
pyside6-lupdate main.py -ts traducciones/app_es.ts
pyside6-linguist traducciones/app_es.ts           # Traducir
pyside6-lrelease traducciones/app_es.ts           # Compilar a .qm
```

---

## 15. Introducción a Qt Quick (QML)

Además de widgets, Qt ofrece **Qt Quick**: interfaces declarativas con QML, animaciones fluidas y aspecto moderno, adecuadas para aplicaciones táctiles o con diseño muy personalizado.

```qml
// principal.qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    visible: true
    width: 360; height: 220
    title: "Contador"

    Column {
        anchors.centerIn: parent
        spacing: 12
        Label { text: "Pulsaciones: " + contador.valor; font.pixelSize: 20 }
        Button { text: "Sumar"; onClicked: contador.incrementar() }
    }
}
```

```python
import sys
from PySide6.QtCore import QObject, Signal, Slot, Property
from PySide6.QtGui import QGuiApplication
from PySide6.QtQml import QQmlApplicationEngine


class Contador(QObject):
    valor_cambiado = Signal()

    def __init__(self):
        super().__init__()
        self._valor = 0

    @Property(int, notify=valor_cambiado)
    def valor(self):
        return self._valor

    @Slot()
    def incrementar(self):
        self._valor += 1
        self.valor_cambiado.emit()


app = QGuiApplication(sys.argv)
motor = QQmlApplicationEngine()
contador = Contador()
motor.rootContext().setContextProperty("contador", contador)
motor.load("principal.qml")
sys.exit(app.exec())
```

| Qt Widgets | Qt Quick (QML) |
|---|---|
| Aplicaciones de escritorio tradicionales, formularios, tablas. | Interfaces fluidas, animadas, táctiles o de diseño libre. |
| Todo en Python. | Interfaz en QML, lógica en Python. |
| Aspecto nativo. | Aspecto personalizado. |

---

## 16. Distribución

### 16.1 `pyside6-deploy`

Herramienta oficial basada en Nuitka:

```bash
pyside6-deploy main.py                 # Genera pysidedeploy.spec y el ejecutable
pyside6-deploy -c pysidedeploy.spec    # Volver a generar con la configuración guardada
```

### 16.2 PyInstaller

```bash
pyinstaller --onefile --windowed --name Gastos --icon icono.ico main.py
```

> PySide6 ocupa bastante espacio. Excluya los módulos no utilizados o use `pyside6-essentials` para reducir el tamaño.

### 16.3 Licencia LGPL: puntos clave

- Puede distribuir aplicaciones comerciales de código cerrado.
- Debe permitir que el usuario sustituya la librería PySide6/Qt (enlace dinámico; no la incruste de forma que no pueda reemplazarse).
- Debe incluir el aviso de licencia LGPL y el acceso al código de PySide6/Qt (o indicar dónde obtenerlo).

> Información orientativa; consulte los términos de la licencia para su caso concreto.

---

## 17. Ejemplo práctico completo

**Control de gastos** con: navegación lateral (`QListWidget` + `QStackedWidget`), base de datos SQLite gestionada con `QSqlTableModel` (alta, edición en tabla, borrado, guardar/descartar), filtros por mes y categoría, panel de resumen con tarjetas y gráficos `QtCharts` (anillo y barras), formato monetario español, exportación a CSV, estilos QSS y configuración persistente.

```python
"""
Control de gastos con PySide6 (QtSql + QtCharts).
Requisitos: pip install pyside6
"""
import csv
import random
import sys
from datetime import date, timedelta
from pathlib import Path

from PySide6.QtCharts import (QBarCategoryAxis, QBarSeries, QBarSet, QChart, QChartView, QPieSeries,
                              QValueAxis)
from PySide6.QtCore import QDate, QLocale, QSettings, QSortFilterProxyModel, Qt, Signal
from PySide6.QtGui import QAction, QColor, QKeySequence, QPainter
from PySide6.QtSql import QSqlDatabase, QSqlQuery, QSqlTableModel
from PySide6.QtWidgets import (QAbstractItemView, QApplication, QComboBox, QDateEdit, QDialog,
                               QDialogButtonBox, QDoubleSpinBox, QFileDialog, QFormLayout, QFrame,
                               QGridLayout, QHBoxLayout, QHeaderView, QLabel, QLineEdit, QListWidget,
                               QMainWindow, QMessageBox, QPushButton, QStackedWidget, QTableView,
                               QVBoxLayout, QWidget)

RUTA_BD = Path(__file__).resolve().parent / "gastos.db"
CATEGORIAS = ["Oficina", "Viajes", "Comidas", "Software", "Formación", "Otros"]
COLORES = ["#2563EB", "#16A34A", "#F59E0B", "#DC2626", "#7C3AED", "#64748B"]
LOCALE = QLocale(QLocale.Language.Spanish, QLocale.Country.Spain)
MESES = ["", "Ene", "Feb", "Mar", "Abr", "May", "Jun", "Jul", "Ago", "Sep", "Oct", "Nov", "Dic"]

ESTILO = """
QMainWindow { background: #F3F4F6; }
QListWidget#menu { background: #111827; color: #D1D5DB; border: none; font-size: 11pt; padding-top: 12px; }
QListWidget#menu::item { padding: 12px 18px; }
QListWidget#menu::item:selected { background: #2563EB; color: white; }
QLabel#titulo { font-size: 18pt; font-weight: 600; color: #111827; }
QFrame#tarjeta { background: white; border-radius: 8px; border: 1px solid #E5E7EB; }
QLabel#kpiValor { font-size: 18pt; font-weight: 700; color: #111827; }
QLabel#kpiTitulo { color: #6B7280; }
QPushButton { background: #2563EB; color: white; border: none; border-radius: 4px; padding: 6px 14px; }
QPushButton:hover { background: #1D4ED8; }
QPushButton[secundario="true"] { background: white; color: #2563EB; border: 1px solid #2563EB; }
QTableView { background: white; gridline-color: #E5E7EB; selection-background-color: #DBEAFE; selection-color: black; }
QHeaderView::section { background: #F9FAFB; padding: 6px; border: none; border-bottom: 1px solid #E5E7EB; font-weight: 600; }
"""


def euros(valor: float) -> str:
    return LOCALE.toCurrencyString(valor, "€")


# ------------------------------------------------------------------ Base de datos
def abrir_bd() -> QSqlDatabase:
    bd = QSqlDatabase.addDatabase("QSQLITE")
    bd.setDatabaseName(str(RUTA_BD))
    if not bd.open():
        raise RuntimeError(bd.lastError().text())
    q = QSqlQuery(bd)
    q.exec("""CREATE TABLE IF NOT EXISTS gastos (
                id INTEGER PRIMARY KEY AUTOINCREMENT, fecha TEXT NOT NULL, concepto TEXT NOT NULL,
                categoria TEXT NOT NULL, importe REAL NOT NULL CHECK (importe > 0))""")
    q.exec("SELECT COUNT(*) FROM gastos")
    if q.next() and q.value(0) == 0:
        random.seed(3)
        conceptos = {"Oficina": ["Papelería", "Tóner", "Mobiliario"], "Viajes": ["Tren", "Hotel", "Taxi"],
                     "Comidas": ["Comida cliente", "Catering"], "Software": ["Licencia anual", "Suscripción nube"],
                     "Formación": ["Curso online", "Congreso"], "Otros": ["Mensajería", "Varios"]}
        bd.transaction()
        q.prepare("INSERT INTO gastos (fecha, concepto, categoria, importe) VALUES (?, ?, ?, ?)")
        for _ in range(160):
            cat = random.choice(CATEGORIAS)
            for v in ((date(2026, 11, 24) - timedelta(days=random.randint(0, 200))).isoformat(),
                      random.choice(conceptos[cat]), cat, round(random.uniform(8, 450), 2)):
                q.addBindValue(v)
            q.exec()
        bd.commit()
    return bd


def consulta(sql: str, params: dict | None = None) -> list[tuple]:
    q = QSqlQuery()
    q.prepare(sql)
    for clave, valor in (params or {}).items():
        q.bindValue(clave, valor)
    if not q.exec():
        raise RuntimeError(q.lastError().text())
    filas, n = [], q.record().count()
    while q.next():
        filas.append(tuple(q.value(i) for i in range(n)))
    return filas


# ------------------------------------------------------------------ Diálogo de alta
class DialogoGasto(QDialog):
    def __init__(self, padre=None):
        super().__init__(padre)
        self.setWindowTitle("Nuevo gasto")
        self.fecha = QDateEdit(QDate.currentDate(), calendarPopup=True, displayFormat="dd/MM/yyyy")
        self.fecha.setMaximumDate(QDate.currentDate())
        self.concepto = QLineEdit(placeholderText="Descripción del gasto")
        self.categoria = QComboBox()
        self.categoria.addItems(CATEGORIAS)
        self.importe = QDoubleSpinBox(minimum=0.01, maximum=100_000, decimals=2, suffix=" €")
        self.importe.setLocale(LOCALE)
        self.importe.setGroupSeparatorShown(True)

        formulario = QFormLayout()
        formulario.addRow("Fecha", self.fecha)
        formulario.addRow("Concepto *", self.concepto)
        formulario.addRow("Categoría", self.categoria)
        formulario.addRow("Importe", self.importe)
        botones = QDialogButtonBox(QDialogButtonBox.StandardButton.Save | QDialogButtonBox.StandardButton.Cancel)
        botones.accepted.connect(self._validar)
        botones.rejected.connect(self.reject)
        layout = QVBoxLayout(self)
        layout.addLayout(formulario)
        layout.addWidget(botones)

    def _validar(self):
        if len(self.concepto.text().strip()) < 3:
            QMessageBox.warning(self, "Datos incompletos", "Indique un concepto (mínimo 3 caracteres).")
            return
        self.accept()

    def valores(self) -> dict:
        return {"fecha": self.fecha.date().toString("yyyy-MM-dd"), "concepto": self.concepto.text().strip(),
                "categoria": self.categoria.currentText(), "importe": round(self.importe.value(), 2)}


# ------------------------------------------------------------------ Página de movimientos
class PaginaMovimientos(QWidget):
    datos_cambiados = Signal()

    def __init__(self, bd: QSqlDatabase):
        super().__init__()
        self.modelo = QSqlTableModel(self, bd)
        self.modelo.setTable("gastos")
        self.modelo.setEditStrategy(QSqlTableModel.EditStrategy.OnManualSubmit)
        self.modelo.setSort(1, Qt.SortOrder.DescendingOrder)
        for col, titulo in enumerate(["Id", "Fecha", "Concepto", "Categoría", "Importe (€)"]):
            self.modelo.setHeaderData(col, Qt.Orientation.Horizontal, titulo)
        self.modelo.select()
        self.modelo.dataChanged.connect(lambda *_: self._marcar_pendiente(True))

        self.buscador = QSortFilterProxyModel(self)
        self.buscador.setSourceModel(self.modelo)
        self.buscador.setFilterKeyColumn(2)
        self.buscador.setFilterCaseSensitivity(Qt.CaseSensitivity.CaseInsensitive)

        titulo = QLabel("Movimientos", objectName="titulo")
        self.filtro_texto = QLineEdit(placeholderText="Buscar concepto…", clearButtonEnabled=True)
        self.filtro_texto.textChanged.connect(self.buscador.setFilterFixedString)
        self.filtro_categoria = QComboBox()
        self.filtro_categoria.addItems(["Todas las categorías"] + CATEGORIAS)
        self.filtro_mes = QComboBox()
        self.filtro_mes.addItem("Todos los meses", "")
        for i in range(6):
            d = QDate.currentDate().addMonths(-i)
            self.filtro_mes.addItem(f"{MESES[d.month()]} {d.year()}", d.toString("yyyy-MM"))
        self.filtro_categoria.currentIndexChanged.connect(self.aplicar_filtros)
        self.filtro_mes.currentIndexChanged.connect(self.aplicar_filtros)

        filtros = QHBoxLayout()
        filtros.addWidget(self.filtro_texto, 1)
        filtros.addWidget(self.filtro_categoria)
        filtros.addWidget(self.filtro_mes)

        self.tabla = QTableView()
        self.tabla.setModel(self.buscador)
        self.tabla.hideColumn(0)
        self.tabla.setSortingEnabled(True)
        self.tabla.sortByColumn(1, Qt.SortOrder.DescendingOrder)        # Más recientes primero
        self.tabla.setSelectionBehavior(QAbstractItemView.SelectionBehavior.SelectRows)
        self.tabla.setAlternatingRowColors(True)
        self.tabla.verticalHeader().hide()
        self.tabla.horizontalHeader().setSectionResizeMode(2, QHeaderView.ResizeMode.Stretch)

        self.boton_nuevo = QPushButton("+ Nuevo gasto")
        self.boton_borrar = QPushButton("Eliminar")
        self.boton_borrar.setProperty("secundario", True)
        self.boton_guardar = QPushButton("Guardar cambios", enabled=False)
        self.boton_descartar = QPushButton("Descartar", enabled=False)
        self.boton_descartar.setProperty("secundario", True)
        self.total = QLabel()
        self.boton_nuevo.clicked.connect(self.nuevo)
        self.boton_borrar.clicked.connect(self.eliminar)
        self.boton_guardar.clicked.connect(self.guardar)
        self.boton_descartar.clicked.connect(self.descartar)

        botones = QHBoxLayout()
        for b in (self.boton_nuevo, self.boton_borrar):
            botones.addWidget(b)
        botones.addStretch()
        botones.addWidget(self.total)
        botones.addSpacing(20)
        botones.addWidget(self.boton_descartar)
        botones.addWidget(self.boton_guardar)

        layout = QVBoxLayout(self)
        layout.setContentsMargins(24, 20, 24, 20)
        layout.addWidget(titulo)
        layout.addLayout(filtros)
        layout.addWidget(self.tabla)
        layout.addLayout(botones)
        self._actualizar_total()

    def aplicar_filtros(self):
        condiciones = []
        if self.filtro_categoria.currentIndex() > 0:
            condiciones.append(f"categoria = '{self.filtro_categoria.currentText()}'")   # Valores de una lista fija
        if mes := self.filtro_mes.currentData():
            condiciones.append(f"substr(fecha, 1, 7) = '{mes}'")
        self.modelo.setFilter(" AND ".join(condiciones))
        self.modelo.select()
        self._actualizar_total()

    def _actualizar_total(self):
        total = sum(self.modelo.record(i).value("importe") or 0 for i in range(self.modelo.rowCount()))
        self.total.setText(f"{self.modelo.rowCount()} movimientos · <b>{euros(total)}</b>")

    def _marcar_pendiente(self, pendiente: bool):
        self.boton_guardar.setEnabled(pendiente)
        self.boton_descartar.setEnabled(pendiente)

    def nuevo(self):
        dialogo = DialogoGasto(self)
        if dialogo.exec() != QDialog.DialogCode.Accepted:
            return
        registro = self.modelo.record()
        registro.setGenerated("id", False)
        for campo, valor in dialogo.valores().items():
            registro.setValue(campo, valor)
        self.modelo.insertRecord(-1, registro)
        self.guardar()

    def eliminar(self):
        filas = sorted({self.buscador.mapToSource(i).row() for i in self.tabla.selectionModel().selectedRows()},
                       reverse=True)
        if not filas:
            return
        if QMessageBox.question(self, "Eliminar", f"¿Eliminar {len(filas)} movimiento(s)?") \
                != QMessageBox.StandardButton.Yes:
            return
        for fila in filas:
            self.modelo.removeRow(fila)
        self.guardar()

    def guardar(self):
        if not self.modelo.submitAll():
            QMessageBox.critical(self, "Error al guardar", self.modelo.lastError().text())
            self.modelo.revertAll()
        self.modelo.select()
        self._marcar_pendiente(False)
        self._actualizar_total()
        self.datos_cambiados.emit()

    def descartar(self):
        self.modelo.revertAll()
        self._marcar_pendiente(False)

    def exportar(self, ruta: str):
        with open(ruta, "w", encoding="utf-8-sig", newline="") as f:
            w = csv.writer(f, delimiter=";")
            w.writerow(["Fecha", "Concepto", "Categoría", "Importe"])
            for fila in range(self.buscador.rowCount()):
                datos = [self.buscador.index(fila, c).data() for c in range(1, 5)]
                datos[3] = f"{datos[3]:.2f}".replace(".", ",")
                w.writerow(datos)


# ------------------------------------------------------------------ Página de resumen
class Tarjeta(QFrame):
    def __init__(self, titulo: str):
        super().__init__(objectName="tarjeta")
        layout = QVBoxLayout(self)
        layout.addWidget(QLabel(titulo, objectName="kpiTitulo"))
        self.valor = QLabel("—", objectName="kpiValor")
        layout.addWidget(self.valor)


class PaginaResumen(QWidget):
    def __init__(self):
        super().__init__()
        rejilla = QGridLayout(self)
        rejilla.setContentsMargins(24, 20, 24, 20)
        rejilla.setSpacing(14)
        rejilla.addWidget(QLabel("Resumen", objectName="titulo"), 0, 0, 1, 4)

        self.tarjetas = {c: Tarjeta(t) for c, t in [("mes", "Gasto este mes"), ("anterior", "Mes anterior"),
                                                    ("media", "Media mensual (6 meses)"), ("mayor", "Mayor categoría")]}
        for i, t in enumerate(self.tarjetas.values()):
            rejilla.addWidget(t, 1, i)

        self.grafico_categorias = QChartView(renderHint=QPainter.RenderHint.Antialiasing)
        self.grafico_meses = QChartView(renderHint=QPainter.RenderHint.Antialiasing)
        rejilla.addWidget(self.grafico_categorias, 2, 0, 1, 2)
        rejilla.addWidget(self.grafico_meses, 2, 2, 1, 2)
        rejilla.setRowStretch(2, 1)
        self.actualizar()

    def actualizar(self):
        hoy = QDate.currentDate()
        mes_actual, mes_anterior = hoy.toString("yyyy-MM"), hoy.addMonths(-1).toString("yyyy-MM")
        desde = hoy.addMonths(-5).toString("yyyy-MM-01")

        por_mes = dict(consulta("SELECT substr(fecha,1,7), SUM(importe) FROM gastos WHERE fecha >= :d "
                                "GROUP BY 1 ORDER BY 1", {":d": desde}))
        por_cat = consulta("SELECT categoria, SUM(importe) FROM gastos WHERE substr(fecha,1,7) = :m "
                           "GROUP BY 1 ORDER BY 2 DESC", {":m": mes_actual})

        actual, anterior = por_mes.get(mes_actual, 0), por_mes.get(mes_anterior, 0)
        self.tarjetas["mes"].valor.setText(euros(actual))
        variacion = f" ({(actual / anterior - 1):+.0%})" if anterior else ""
        self.tarjetas["anterior"].valor.setText(euros(anterior) + variacion)
        self.tarjetas["media"].valor.setText(euros(sum(por_mes.values()) / max(1, len(por_mes))))
        self.tarjetas["mayor"].valor.setText(por_cat[0][0] if por_cat else "—")

        # Anillo por categoría
        serie = QPieSeries()
        serie.setHoleSize(0.4)
        for i, (cat, total) in enumerate(por_cat):
            porcion = serie.append(f"{cat}  {euros(total)}", total)
            porcion.setColor(QColor(COLORES[CATEGORIAS.index(cat) % len(COLORES)]))
        grafico = QChart()
        grafico.addSeries(serie)
        grafico.setTitle(f"Por categoría · {MESES[hoy.month()]} {hoy.year()}")
        grafico.legend().setAlignment(Qt.AlignmentFlag.AlignRight)
        grafico.setAnimationOptions(QChart.AnimationOption.SeriesAnimations)
        self.grafico_categorias.setChart(grafico)

        # Barras por mes
        meses = [hoy.addMonths(-i) for i in range(5, -1, -1)]
        conjunto = QBarSet("Gasto")
        conjunto.append([por_mes.get(m.toString("yyyy-MM"), 0) for m in meses])
        conjunto.setColor(QColor("#2563EB"))
        barras = QBarSeries()
        barras.append(conjunto)
        grafico2 = QChart()
        grafico2.addSeries(barras)
        grafico2.setTitle("Últimos 6 meses")
        eje_x = QBarCategoryAxis()
        eje_x.append([MESES[m.month()] for m in meses])
        eje_y = QValueAxis()
        eje_y.setLabelFormat("%.0f €")
        grafico2.addAxis(eje_x, Qt.AlignmentFlag.AlignBottom)
        grafico2.addAxis(eje_y, Qt.AlignmentFlag.AlignLeft)
        barras.attachAxis(eje_x)
        barras.attachAxis(eje_y)
        grafico2.legend().hide()
        self.grafico_meses.setChart(grafico2)


# ------------------------------------------------------------------ Ventana principal
class VentanaPrincipal(QMainWindow):
    def __init__(self, bd: QSqlDatabase):
        super().__init__()
        self.setWindowTitle("Control de gastos")
        self.resize(1100, 680)
        self.ajustes = QSettings()

        self.menu = QListWidget(objectName="menu")
        self.menu.setFixedWidth(190)
        self.menu.addItems(["Resumen", "Movimientos"])
        self.paginas = QStackedWidget()
        self.resumen = PaginaResumen()
        self.movimientos = PaginaMovimientos(bd)
        self.paginas.addWidget(self.resumen)
        self.paginas.addWidget(self.movimientos)
        self.menu.currentRowChanged.connect(self.paginas.setCurrentIndex)
        self.movimientos.datos_cambiados.connect(self.resumen.actualizar)

        central = QWidget()
        layout = QHBoxLayout(central)
        layout.setContentsMargins(0, 0, 0, 0)
        layout.setSpacing(0)
        layout.addWidget(self.menu)
        layout.addWidget(self.paginas, 1)
        self.setCentralWidget(central)

        archivo = self.menuBar().addMenu("&Archivo")
        archivo.addAction(QAction("Nuevo gasto", self, shortcut=QKeySequence.StandardKey.New,
                                  triggered=lambda: (self.menu.setCurrentRow(1), self.movimientos.nuevo())))
        archivo.addAction(QAction("Exportar CSV…", self, shortcut="Ctrl+E", triggered=self.exportar))
        archivo.addSeparator()
        archivo.addAction(QAction("Salir", self, shortcut=QKeySequence.StandardKey.Quit, triggered=self.close))

        self.menu.setCurrentRow(int(self.ajustes.value("pagina", 0)))
        if geometria := self.ajustes.value("geometria"):
            self.restoreGeometry(geometria)
        self.statusBar().showMessage(f"Base de datos: {RUTA_BD.name}")

    def exportar(self):
        ruta, _ = QFileDialog.getSaveFileName(self, "Exportar", "gastos.csv", "CSV (*.csv)")
        if ruta:
            self.movimientos.exportar(ruta)
            self.statusBar().showMessage(f"Exportado a {Path(ruta).name}", 4000)

    def closeEvent(self, evento):
        self.ajustes.setValue("geometria", self.saveGeometry())
        self.ajustes.setValue("pagina", self.menu.currentRow())
        super().closeEvent(evento)


if __name__ == "__main__":
    app = QApplication(sys.argv)
    app.setOrganizationName("MiEmpresa")
    app.setApplicationName("ControlGastos")
    app.setStyle("Fusion")
    app.setStyleSheet(ESTILO)
    QLocale.setDefault(LOCALE)
    ventana = VentanaPrincipal(abrir_bd())
    ventana.show()
    sys.exit(app.exec())
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ImportError: DLL load failed` (Windows) | Instalación dañada o conflicto con otra instalación de Qt/PyQt. | Reinstale en un entorno virtual limpio. |
| `qt.qpa.plugin: Could not load the Qt platform plugin "xcb"` (Linux) | Faltan librerías del sistema. | `sudo apt install libxcb-cursor0` y dependencias. |
| `AttributeError: 'PySide6.QtCore' has no attribute 'pyqtSignal'` | Código de PyQt. | Use `Signal` y `Slot`. |
| `QSqlDatabase: QSQLITE driver not loaded` / `requires a QCoreApplication` | Conexión creada antes que la aplicación. | Cree `QApplication` antes de `addDatabase`. |
| `QSqlDatabasePrivate::removeDatabase: connection is still in use` | Se cerró la base de datos con modelos o consultas activos. | Elimine modelos y consultas antes de cerrar. |
| `submitAll()` devuelve `False` | Violación de restricciones o error SQL. | Consulte `modelo.lastError().text()`. |
| `ModuleNotFoundError: No module named 'PySide6.QtCharts'` | Instalación `pyside6-essentials` sin complementos. | Instale `pyside6` (completo) o `pyside6-addons`. |
| `QUiLoader` no conecta los slots con `self` | Devuelve un widget nuevo. | Acceda a los widgets del objeto devuelto o use `pyside6-uic`. |
| La interfaz se congela | Tarea larga en el hilo principal. | `QThreadPool` / `QThread` + señales. |
| `RuntimeError: Internal C++ object already deleted` | Se usó un objeto Qt ya destruido (su padre se eliminó). | Mantenga las referencias y revise la jerarquía de padres. |
| La ventana se cierra al crearla | No se guardó la referencia a la ventana. | Asígnela a una variable que permanezca viva. |
| Los textos de los botones estándar en inglés | No se cargó la traducción de Qt. | Instale `QTranslator` con `qtbase_es`. |

---

## 19. Buenas prácticas

- Use PySide6 cuando necesite distribuir software comercial con licencia LGPL o prefiera el binding oficial.
- Organice la aplicación en ventanas, páginas y componentes; separe la interfaz del acceso a datos.
- Use **layouts** siempre y `QStackedWidget` para la navegación entre secciones.
- Aproveche **QtSql** para tablas de bases de datos y **QtCharts** para gráficos integrados.
- Use `OnManualSubmit` en modelos SQL editables para permitir guardar o descartar.
- Ejecute las tareas lentas fuera del hilo principal y comuníquese con señales.
- Configure el idioma (`QLocale`, `QTranslator`) para formatos y textos en español.
- Centralice los estilos en un archivo QSS y use `objectName`/propiedades como selectores.
- Guarde preferencias y geometría con `QSettings`.
- Diseñe ventanas complejas en Qt Designer y genere el código con `pyside6-uic`.

---

## 20. Referencia rápida

```python
app = QApplication(sys.argv); v.show(); sys.exit(app.exec())
Signal(tipo) / @Slot() / Property(tipo, getter, setter, notify=senal)
QDate.toPython() · QLocale(QLocale.Language.Spanish, QLocale.Country.Spain).toCurrencyString(v, "€")
QStackedWidget + QListWidget.currentRowChanged → setCurrentIndex
QDockWidget · QSystemTrayIcon · QProgressDialog · QPropertyAnimation

bd = QSqlDatabase.addDatabase("QSQLITE"); bd.setDatabaseName(r); bd.open()
q = QSqlQuery(); q.prepare(sql); q.bindValue(":p", v) | addBindValue(v); q.exec(); while q.next(): q.value(i)
m = QSqlTableModel(db=bd); m.setTable(t); m.setFilter(cond); m.setSort(col, orden); m.select(); m.submitAll() / revertAll()

QPieSeries().append(etiqueta, valor) · QBarSet + QBarSeries + QBarCategoryAxis + QValueAxis · QLineSeries
QChart().addSeries(s); QChartView(chart).setRenderHint(QPainter.RenderHint.Antialiasing)

QUiLoader().load(QFile("x.ui")) · pyside6-uic x.ui -o ui_x.py · pyside6-rcc r.qrc -o r_rc.py
pyside6-designer · pyside6-deploy main.py · QSettings · app.setStyle("Fusion") · app.setStyleSheet(qss)
```

---

## 21. Ejercicios propuestos

1. Convierta el ejemplo de PyQt6 (gestor de inventario) a PySide6 cambiando solo las importaciones y las señales.
2. Cree una agenda de contactos con `QSqlRelationalTableModel` donde la ciudad sea una clave foránea mostrada como desplegable.
3. Añada al control de gastos un gráfico de líneas con el gasto acumulado del año y un selector de año.
4. Diseñe la ventana de alta de gastos en Qt Designer, conviértala con `pyside6-uic` y úsela en la aplicación.
5. Empaquete el ejemplo completo con `pyside6-deploy` y compruebe el tamaño del ejecutable resultante.

---

## 22. Referencias

- Documentación oficial de Qt for Python: https://doc.qt.io/qtforpython-6/
- Tutoriales oficiales: https://doc.qt.io/qtforpython-6/tutorials/index.html
- Documentación de Qt 6: https://doc.qt.io/qt-6/
- Comparativa PySide6 / PyQt6: https://www.pythonguis.com/faq/pyqt6-vs-pyside6/
