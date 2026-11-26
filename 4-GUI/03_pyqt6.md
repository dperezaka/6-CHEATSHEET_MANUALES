# Manual de uso: PyQt6

> **Categoría:** Interfaces gráficas (GUI) · **Utilidad:** Aplicaciones de escritorio profesionales
> **Tipo:** Librería externa · Binding de **Qt 6** desarrollado por Riverbank Computing

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Widgets principales](#5-widgets-principales)
6. [Layouts](#6-layouts)
7. [Señales y slots](#7-señales-y-slots)
8. [QMainWindow: menús, barras y acciones](#8-qmainwindow-menús-barras-y-acciones)
9. [Diálogos](#9-diálogos)
10. [Modelo/vista: tablas y listas](#10-modelovista-tablas-y-listas)
11. [Estilos (QSS) y temas](#11-estilos-qss-y-temas)
12. [Hilos: QThread y QThreadPool](#12-hilos-qthread-y-qthreadpool)
13. [Qt Designer](#13-qt-designer)
14. [Configuración persistente (QSettings)](#14-configuración-persistente-qsettings)
15. [Otras funcionalidades](#15-otras-funcionalidades)
16. [Distribución y licencia](#16-distribución-y-licencia)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**PyQt6** permite desarrollar aplicaciones de escritorio con **Qt 6**, uno de los frameworks de interfaces gráficas más completos y utilizados en la industria (se usa en aplicaciones como VirtualBox, Autodesk Maya, el cliente de Dropbox o VLC).

**Características:**
- Multiplataforma: Windows, macOS y Linux con aspecto nativo.
- Cientos de widgets: tablas con modelos de datos, árboles, pestañas, gráficos, editor de texto enriquecido, navegador web, multimedia…
- Arquitectura **modelo/vista** para manejar grandes volúmenes de datos.
- Sistema de **señales y slots** para comunicar componentes.
- Estilos con **QSS** (sintaxis similar a CSS).
- Diseñador visual **Qt Designer**.
- Soporte para hilos, red, bases de datos SQL, impresión, internacionalización y accesibilidad.

**PyQt6 frente a PySide6:**

| Aspecto | PyQt6 | PySide6 |
|---|---|---|
| Desarrollador | Riverbank Computing | The Qt Company (oficial) |
| Licencia | **GPL v3** o comercial | **LGPL v3** (uso comercial más flexible) |
| Señales | `pyqtSignal`, `pyqtSlot` | `Signal`, `Slot` |
| API | Prácticamente idéntica | Prácticamente idéntica |

> El código es casi intercambiable. Si va a distribuir software comercial de código cerrado sin comprar una licencia, valore PySide6 (ver sección 16).

---

## 2. Instalación e importación

```bash
pip install PyQt6
pip install pyqt6-tools          # Opcional: Qt Designer y otras herramientas (según plataforma)
```

Módulos adicionales que se instalan por separado:

```bash
pip install PyQt6-Charts             # Gráficos (QtCharts)
pip install PyQt6-WebEngine          # Navegador web integrado
```

Importaciones habituales:

```python
import sys
from PyQt6.QtWidgets import (
    QApplication, QMainWindow, QWidget, QDialog, QLabel, QPushButton, QLineEdit, QTextEdit,
    QComboBox, QCheckBox, QRadioButton, QSpinBox, QDoubleSpinBox, QDateEdit, QSlider, QProgressBar,
    QVBoxLayout, QHBoxLayout, QGridLayout, QFormLayout, QGroupBox, QTabWidget, QSplitter,
    QTableView, QTableWidget, QTableWidgetItem, QListWidget, QTreeWidget, QHeaderView,
    QMessageBox, QFileDialog, QInputDialog, QStatusBar, QToolBar, QDialogButtonBox,
)
from PyQt6.QtCore import Qt, QTimer, QThread, pyqtSignal, pyqtSlot, QObject, QSettings, QSize, QDate
from PyQt6.QtGui import QAction, QIcon, QFont, QKeySequence, QColor, QPixmap
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `QApplication` | Objeto único que gestiona la aplicación y el bucle de eventos. |
| `QWidget` | Clase base de todos los elementos visuales; un `QWidget` sin padre es una ventana. |
| `QMainWindow` | Ventana principal con menú, barras de herramientas, barra de estado y widget central. |
| `QDialog` | Ventana de diálogo (modal o no). |
| Layout | Organiza los widgets y los adapta al tamaño (`QVBoxLayout`, `QGridLayout`…). |
| Señal | Evento que emite un objeto (`clicked`, `textChanged`…). |
| Slot | Función que se ejecuta al recibir una señal. |
| Padre (*parent*) | Jerarquía de objetos: al destruirse el padre, se destruyen los hijos. |
| Modelo/vista | Separación entre los datos (modelo) y su presentación (vista). |
| Enumeraciones | En PyQt6 se usan con su nombre completo: `Qt.AlignmentFlag.AlignCenter`. |

> **Diferencia importante con PyQt5:** en PyQt6 las enumeraciones se escriben completas (`Qt.AlignmentFlag.AlignRight`, `QMessageBox.StandardButton.Yes`), `exec_()` pasa a ser `exec()` y `QAction` está en `QtGui`.

---

## 4. Primera aplicación

### 4.1 Ventana mínima

```python
import sys
from PyQt6.QtWidgets import QApplication, QLabel

app = QApplication(sys.argv)
etiqueta = QLabel("Hola, PyQt6")
etiqueta.resize(300, 100)
etiqueta.show()
sys.exit(app.exec())
```

### 4.2 Ventana con clase (estructura recomendada)

```python
import sys
from PyQt6.QtWidgets import QApplication, QWidget, QVBoxLayout, QLabel, QLineEdit, QPushButton


class Ventana(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Saludo")
        self.resize(360, 160)

        self.entrada = QLineEdit(placeholderText="Escriba su nombre")
        self.resultado = QLabel()
        boton = QPushButton("Saludar")
        boton.clicked.connect(self.saludar)
        self.entrada.returnPressed.connect(self.saludar)

        layout = QVBoxLayout(self)
        layout.addWidget(self.entrada)
        layout.addWidget(boton)
        layout.addWidget(self.resultado)

    def saludar(self):
        nombre = self.entrada.text().strip() or "desconocido"
        self.resultado.setText(f"Hola, <b>{nombre}</b>")


if __name__ == "__main__":
    app = QApplication(sys.argv)
    ventana = Ventana()
    ventana.show()
    sys.exit(app.exec())
```

> Solo debe existir **una** `QApplication`. Créela antes que cualquier widget.

### 4.3 Propiedades comunes de las ventanas

```python
ventana.setWindowTitle("Título")
ventana.resize(800, 600)
ventana.setMinimumSize(600, 400)
ventana.setWindowIcon(QIcon("icono.png"))
ventana.showMaximized()
ventana.setWindowFlag(Qt.WindowType.WindowStaysOnTopHint, True)
```

### 4.4 Argumentos en el constructor

PyQt6 permite asignar propiedades como argumentos con nombre:

```python
QLineEdit(placeholderText="Buscar…", maxLength=50, clearButtonEnabled=True)
QPushButton("Aceptar", default=True, minimumWidth=100)
```

---

## 5. Widgets principales

### 5.1 Texto y botones

```python
etiqueta = QLabel("Texto")
etiqueta.setAlignment(Qt.AlignmentFlag.AlignCenter)
etiqueta.setWordWrap(True)
etiqueta.setText("<h2>Título</h2><p>Admite <i>HTML</i> básico</p>")
etiqueta.setPixmap(QPixmap("logo.png").scaledToWidth(160, Qt.TransformationMode.SmoothTransformation))

boton = QPushButton("Guardar")
boton.setIcon(QIcon("iconos/guardar.png"))
boton.setEnabled(False)
boton.setToolTip("Guarda los cambios (Ctrl+S)")
boton.setCheckable(True)            # Botón de alternancia
```

### 5.2 Entrada de datos

```python
campo = QLineEdit()
campo.setPlaceholderText("Email")
campo.setEchoMode(QLineEdit.EchoMode.Password)
campo.setMaxLength(100)
campo.setInputMask("99999")                 # Código postal: 5 dígitos
campo.text(); campo.setText("valor"); campo.clear()

texto = QTextEdit()                          # Texto enriquecido multilínea
texto.setPlainText("Contenido")
texto.toPlainText(); texto.toHtml()
# QPlainTextEdit: más eficiente para textos largos sin formato (logs)

combo = QComboBox()
combo.addItems(["Madrid", "Sevilla", "Valencia"])
combo.addItem("Bilbao", userData=48)         # Texto visible + dato asociado
combo.setEditable(True)
combo.currentText(); combo.currentData(); combo.currentIndex()
combo.setCurrentText("Sevilla")

casilla = QCheckBox("Cliente VIP")
casilla.isChecked(); casilla.setChecked(True)

numero = QSpinBox(minimum=0, maximum=1000, singleStep=5, suffix=" uds")
importe = QDoubleSpinBox(minimum=0, maximum=99999, decimals=2, suffix=" €")
fecha = QDateEdit(calendarPopup=True, date=QDate.currentDate(), displayFormat="dd/MM/yyyy")
fecha.date().toPyDate()                      # → datetime.date

deslizador = QSlider(Qt.Orientation.Horizontal, minimum=0, maximum=100)
progreso = QProgressBar(minimum=0, maximum=100, value=40)
```

### 5.3 Validadores

```python
from PyQt6.QtGui import QIntValidator, QDoubleValidator, QRegularExpressionValidator
from PyQt6.QtCore import QRegularExpression

QLineEdit(validator=QIntValidator(0, 120))
QLineEdit(validator=QRegularExpressionValidator(QRegularExpression(r"^\d{8}[A-Za-z]$")))   # DNI
```

### 5.4 Contenedores

```python
grupo = QGroupBox("Datos del cliente")
pestanas = QTabWidget()
pestanas.addTab(QWidget(), "General")
pestanas.addTab(QWidget(), "Pedidos")

divisor = QSplitter(Qt.Orientation.Horizontal)     # Paneles redimensionables
divisor.addWidget(panel_izquierdo)
divisor.addWidget(panel_derecho)
divisor.setSizes([250, 750])
```

Otros: `QScrollArea` (desplazamiento), `QStackedWidget` (varias páginas, una visible), `QDockWidget` (paneles acoplables), `QFrame`.

---

## 6. Layouts

Los layouts colocan los widgets y los adaptan al redimensionar la ventana. **Evite las posiciones fijas** (`move`, `setGeometry`).

```python
# Vertical y horizontal
vertical = QVBoxLayout()
vertical.addWidget(QLabel("Arriba"))
vertical.addWidget(QLabel("Abajo"))

botones = QHBoxLayout()
botones.addStretch()                       # Empuja los botones a la derecha
botones.addWidget(QPushButton("Cancelar"))
botones.addWidget(QPushButton("Aceptar"))
vertical.addLayout(botones)

# Rejilla
rejilla = QGridLayout()
rejilla.addWidget(QLabel("Nombre"), 0, 0)
rejilla.addWidget(QLineEdit(), 0, 1)
rejilla.addWidget(QTextEdit(), 1, 0, 1, 2)    # fila, columna, filas que ocupa, columnas que ocupa
rejilla.setColumnStretch(1, 1)

# Formulario (etiqueta + campo)
formulario = QFormLayout()
formulario.addRow("Nombre:", QLineEdit())
formulario.addRow("Email:", QLineEdit())
formulario.addRow("Ciudad:", QComboBox())

widget.setLayout(vertical)        # O pasar el widget al constructor: QVBoxLayout(widget)
```

Ajustes:

```python
layout.setContentsMargins(12, 12, 12, 12)
layout.setSpacing(8)
layout.addWidget(widget, stretch=1)        # Proporción del espacio
layout.addSpacing(20)
layout.addStretch(1)
widget.setSizePolicy(QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Fixed)
```

---

## 7. Señales y slots

### 7.1 Conectar señales

```python
boton.clicked.connect(self.guardar)
campo.textChanged.connect(self.validar)                    # Recibe el texto
combo.currentIndexChanged.connect(self.al_cambiar_ciudad)  # Recibe el índice
casilla.toggled.connect(lambda marcado: print(marcado))
boton.clicked.connect(lambda: self.eliminar(registro_id))  # Con argumentos propios
```

Señales habituales:

| Widget | Señales |
|---|---|
| `QPushButton` | `clicked`, `pressed`, `toggled` |
| `QLineEdit` | `textChanged`, `textEdited`, `returnPressed`, `editingFinished` |
| `QComboBox` | `currentIndexChanged`, `currentTextChanged`, `activated` |
| `QCheckBox` | `toggled`, `stateChanged` |
| `QSpinBox` | `valueChanged` |
| `QTableView` | `clicked`, `doubleClicked`; `selectionModel().selectionChanged` |
| `QTabWidget` | `currentChanged` |
| `QTimer` | `timeout` |

### 7.2 Desconectar y bloquear

```python
boton.clicked.disconnect(self.guardar)
campo.blockSignals(True); campo.setText("x"); campo.blockSignals(False)   # Cambiar sin emitir señales
```

### 7.3 Señales propias

```python
from PyQt6.QtCore import QObject, pyqtSignal


class Carrito(QObject):
    total_cambiado = pyqtSignal(float)              # Tipo del argumento
    producto_anadido = pyqtSignal(str, int)

    def __init__(self):
        super().__init__()
        self._total = 0.0

    def anadir(self, nombre: str, unidades: int, precio: float):
        self._total += unidades * precio
        self.producto_anadido.emit(nombre, unidades)
        self.total_cambiado.emit(self._total)


carrito = Carrito()
carrito.total_cambiado.connect(lambda t: etiqueta_total.setText(f"{t:.2f} €"))
```

> Las señales propias se declaran como **atributos de clase** en una clase que herede de `QObject` (o de cualquier widget).

### 7.4 Temporizadores

```python
self.temporizador = QTimer(self)
self.temporizador.timeout.connect(self.actualizar_reloj)
self.temporizador.start(1000)          # Cada segundo

QTimer.singleShot(3000, self.ocultar_aviso)     # Una sola vez, a los 3 s
```

### 7.5 Eventos (sobrescribir métodos)

```python
class Ventana(QMainWindow):
    def closeEvent(self, evento):
        if self.hay_cambios:
            r = QMessageBox.question(self, "Salir", "Hay cambios sin guardar. ¿Salir de todos modos?")
            if r != QMessageBox.StandardButton.Yes:
                evento.ignore()
                return
        evento.accept()

    def keyPressEvent(self, evento):
        if evento.key() == Qt.Key.Key_Escape:
            self.close()
        super().keyPressEvent(evento)

    def resizeEvent(self, evento):
        self.statusBar().showMessage(f"{self.width()} × {self.height()}")
        super().resizeEvent(evento)
```

---

## 8. QMainWindow: menús, barras y acciones

### 8.1 Acciones

Una `QAction` representa una orden reutilizable en menús, barras de herramientas y atajos:

```python
accion_guardar = QAction(QIcon("iconos/guardar.png"), "&Guardar", self)
accion_guardar.setShortcut(QKeySequence.StandardKey.Save)       # Ctrl+S / Cmd+S
accion_guardar.setStatusTip("Guardar el documento")
accion_guardar.triggered.connect(self.guardar)

accion_salir = QAction("Salir", self, shortcut="Ctrl+Q", triggered=self.close)
```

### 8.2 Menús, barra de herramientas y barra de estado

```python
class Principal(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setCentralWidget(QTextEdit())

        menu = self.menuBar()
        archivo = menu.addMenu("&Archivo")
        archivo.addAction(accion_guardar)
        archivo.addSeparator()
        archivo.addAction(accion_salir)

        recientes = archivo.addMenu("Recientes")         # Submenú

        barra = self.addToolBar("Principal")
        barra.setIconSize(QSize(20, 20))
        barra.addAction(accion_guardar)
        barra.setMovable(False)

        self.statusBar().showMessage("Listo", 3000)      # 3 segundos
        self.etiqueta_estado = QLabel("0 registros")
        self.statusBar().addPermanentWidget(self.etiqueta_estado)
```

### 8.3 Menú contextual

```python
tabla.setContextMenuPolicy(Qt.ContextMenuPolicy.CustomContextMenu)
tabla.customContextMenuRequested.connect(self.menu_contextual)

def menu_contextual(self, posicion):
    menu = QMenu(self)
    menu.addAction("Editar", self.editar)
    menu.addAction("Eliminar", self.eliminar)
    menu.exec(self.tabla.viewport().mapToGlobal(posicion))
```

### 8.4 Atajos sin acción

```python
from PyQt6.QtGui import QShortcut
QShortcut(QKeySequence("Ctrl+F"), self, activated=self.buscador.setFocus)
```

---

## 9. Diálogos

### 9.1 Cuadros de mensaje

```python
QMessageBox.information(self, "Información", "Datos guardados correctamente.")
QMessageBox.warning(self, "Aviso", "Hay campos obligatorios vacíos.")
QMessageBox.critical(self, "Error", "No se pudo conectar con el servidor.")

r = QMessageBox.question(self, "Eliminar", "¿Eliminar el registro seleccionado?",
                         QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No,
                         QMessageBox.StandardButton.No)
if r == QMessageBox.StandardButton.Yes:
    ...
```

### 9.2 Archivos, carpetas y entradas

```python
ruta, _ = QFileDialog.getOpenFileName(self, "Abrir", "", "CSV (*.csv);;Excel (*.xlsx);;Todos (*)")
rutas, _ = QFileDialog.getOpenFileNames(self, "Seleccionar imágenes", "", "Imágenes (*.png *.jpg)")
ruta, _ = QFileDialog.getSaveFileName(self, "Guardar", "informe.pdf", "PDF (*.pdf)")
carpeta = QFileDialog.getExistingDirectory(self, "Elegir carpeta")

texto, ok = QInputDialog.getText(self, "Nuevo proyecto", "Nombre:")
numero, ok = QInputDialog.getInt(self, "Unidades", "Cantidad:", value=1, min=1, max=999)
opcion, ok = QInputDialog.getItem(self, "Ciudad", "Seleccione:", ["Madrid", "Sevilla"], editable=False)
```

> Compruebe siempre si el usuario canceló (`ruta == ""` o `ok == False`).

### 9.3 Diálogo propio

```python
class DialogoCliente(QDialog):
    def __init__(self, padre=None, datos: dict | None = None):
        super().__init__(padre)
        self.setWindowTitle("Cliente")
        datos = datos or {}

        self.nombre = QLineEdit(datos.get("nombre", ""))
        self.email = QLineEdit(datos.get("email", ""))
        formulario = QFormLayout()
        formulario.addRow("Nombre *", self.nombre)
        formulario.addRow("Email *", self.email)

        botones = QDialogButtonBox(QDialogButtonBox.StandardButton.Ok | QDialogButtonBox.StandardButton.Cancel)
        botones.accepted.connect(self.validar)
        botones.rejected.connect(self.reject)

        layout = QVBoxLayout(self)
        layout.addLayout(formulario)
        layout.addWidget(botones)

    def validar(self):
        if not self.nombre.text().strip():
            QMessageBox.warning(self, "Falta información", "El nombre es obligatorio.")
            return
        self.accept()

    def datos(self) -> dict:
        return {"nombre": self.nombre.text().strip(), "email": self.email.text().strip()}


dialogo = DialogoCliente(self)
if dialogo.exec() == QDialog.DialogCode.Accepted:
    print(dialogo.datos())
```

---

## 10. Modelo/vista: tablas y listas

Qt ofrece dos enfoques:

| Enfoque | Clases | Uso |
|---|---|---|
| **Basado en elementos** | `QTableWidget`, `QListWidget`, `QTreeWidget` | Pocos datos, sencillo. |
| **Modelo/vista** | `QTableView` + modelo (`QAbstractTableModel`, `QStandardItemModel`, `QSqlTableModel`) | Muchos datos, filtros, ordenación, varias vistas de los mismos datos. |

### 10.1 `QTableWidget` (sencillo)

```python
tabla = QTableWidget(0, 3)
tabla.setHorizontalHeaderLabels(["Producto", "Unidades", "Precio"])
for fila, (nombre, uds, precio) in enumerate(datos):
    tabla.insertRow(fila)
    tabla.setItem(fila, 0, QTableWidgetItem(nombre))
    tabla.setItem(fila, 1, QTableWidgetItem(str(uds)))
    tabla.setItem(fila, 2, QTableWidgetItem(f"{precio:.2f} €"))
tabla.horizontalHeader().setSectionResizeMode(0, QHeaderView.ResizeMode.Stretch)
tabla.setSortingEnabled(True)
```

### 10.2 Modelo propio con `QAbstractTableModel`

```python
from PyQt6.QtCore import QAbstractTableModel, QModelIndex, Qt


class ModeloProductos(QAbstractTableModel):
    COLUMNAS = ["Código", "Nombre", "Categoría", "Precio", "Stock"]

    def __init__(self, productos: list[dict] | None = None):
        super().__init__()
        self.productos = productos or []

    def rowCount(self, padre=QModelIndex()):
        return len(self.productos)

    def columnCount(self, padre=QModelIndex()):
        return len(self.COLUMNAS)

    def data(self, indice, rol=Qt.ItemDataRole.DisplayRole):
        if not indice.isValid():
            return None
        p = self.productos[indice.row()]
        clave = ["codigo", "nombre", "categoria", "precio", "stock"][indice.column()]
        valor = p[clave]
        if rol == Qt.ItemDataRole.DisplayRole:
            return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".") if clave == "precio" else str(valor)
        if rol == Qt.ItemDataRole.UserRole:                  # Valor sin formato (para ordenar)
            return valor
        if rol == Qt.ItemDataRole.TextAlignmentRole and clave in ("precio", "stock"):
            return Qt.AlignmentFlag.AlignRight | Qt.AlignmentFlag.AlignVCenter
        if rol == Qt.ItemDataRole.ForegroundRole and clave == "stock" and valor == 0:
            return QColor("#C00000")
        return None

    def headerData(self, seccion, orientacion, rol=Qt.ItemDataRole.DisplayRole):
        if rol == Qt.ItemDataRole.DisplayRole and orientacion == Qt.Orientation.Horizontal:
            return self.COLUMNAS[seccion]
        return None

    # Modificar datos notificando a la vista
    def anadir(self, producto: dict):
        fila = len(self.productos)
        self.beginInsertRows(QModelIndex(), fila, fila)
        self.productos.append(producto)
        self.endInsertRows()

    def eliminar(self, fila: int):
        self.beginRemoveRows(QModelIndex(), fila, fila)
        del self.productos[fila]
        self.endRemoveRows()

    def actualizar(self, fila: int, producto: dict):
        self.productos[fila] = producto
        self.dataChanged.emit(self.index(fila, 0), self.index(fila, self.columnCount() - 1))

    def reemplazar_todo(self, productos: list[dict]):
        self.beginResetModel()
        self.productos = productos
        self.endResetModel()
```

> **Regla clave:** cualquier cambio en los datos del modelo debe ir acompañado de las llamadas `begin…/end…` o de la señal `dataChanged`, para que la vista se actualice.

| Rol | Uso |
|---|---|
| `DisplayRole` | Texto que se muestra. |
| `EditRole` | Valor al editar. |
| `ToolTipRole` | Texto emergente. |
| `TextAlignmentRole` | Alineación. |
| `ForegroundRole` / `BackgroundRole` | Colores de texto y fondo. |
| `FontRole` | Fuente. |
| `DecorationRole` | Icono. |
| `UserRole` | Datos propios. |

### 10.3 Vista, filtro y ordenación

```python
from PyQt6.QtCore import QSortFilterProxyModel

modelo = ModeloProductos(productos)
proxy = QSortFilterProxyModel()
proxy.setSourceModel(modelo)
proxy.setFilterCaseSensitivity(Qt.CaseSensitivity.CaseInsensitive)
proxy.setFilterKeyColumn(-1)                    # Buscar en todas las columnas
proxy.setSortRole(Qt.ItemDataRole.UserRole)     # Ordenar por valor real (no por texto formateado)

vista = QTableView()
vista.setModel(proxy)
vista.setSortingEnabled(True)
vista.setSelectionBehavior(QTableView.SelectionBehavior.SelectRows)
vista.setSelectionMode(QTableView.SelectionMode.SingleSelection)
vista.setAlternatingRowColors(True)
vista.verticalHeader().setVisible(False)
vista.horizontalHeader().setSectionResizeMode(1, QHeaderView.ResizeMode.Stretch)

buscador.textChanged.connect(proxy.setFilterFixedString)
```

Obtener la fila seleccionada en el modelo original:

```python
indices = vista.selectionModel().selectedRows()
if indices:
    fila_original = proxy.mapToSource(indices[0]).row()
    producto = modelo.productos[fila_original]
```

### 10.4 Edición directa en la tabla

```python
def flags(self, indice):
    base = super().flags(indice)
    if indice.column() in (3, 4):                     # Precio y stock editables
        return base | Qt.ItemFlag.ItemIsEditable
    return base

def setData(self, indice, valor, rol=Qt.ItemDataRole.EditRole):
    if rol != Qt.ItemDataRole.EditRole:
        return False
    clave = ["codigo", "nombre", "categoria", "precio", "stock"][indice.column()]
    try:
        self.productos[indice.row()][clave] = float(valor) if clave == "precio" else int(valor)
    except ValueError:
        return False
    self.dataChanged.emit(indice, indice)
    return True
```

### 10.5 Bases de datos con `QtSql`

```python
from PyQt6.QtSql import QSqlDatabase, QSqlTableModel

bd = QSqlDatabase.addDatabase("QSQLITE")
bd.setDatabaseName("tienda.db")
bd.open()

modelo = QSqlTableModel(db=bd)
modelo.setTable("productos")
modelo.setEditStrategy(QSqlTableModel.EditStrategy.OnManualSubmit)
modelo.select()
vista.setModel(modelo)
# modelo.submitAll() guarda los cambios; modelo.revertAll() los descarta
```

---

## 11. Estilos (QSS) y temas

### 11.1 Estilo Fusion

```python
app.setStyle("Fusion")          # Aspecto moderno y uniforme en todos los sistemas
```

### 11.2 Hojas de estilo (QSS)

```python
app.setStyleSheet("""
    QWidget { font-family: "Segoe UI"; font-size: 10pt; }
    QPushButton {
        background-color: #1F4E78; color: white; border: none;
        padding: 6px 14px; border-radius: 4px;
    }
    QPushButton:hover { background-color: #2A6496; }
    QPushButton:disabled { background-color: #A0AEC0; }
    QPushButton#peligro { background-color: #C0392B; }
    QLineEdit, QComboBox { padding: 4px; border: 1px solid #C5CCD3; border-radius: 4px; }
    QLineEdit:focus { border-color: #1F4E78; }
    QHeaderView::section { background: #1F4E78; color: white; padding: 5px; border: none; }
    QTableView { gridline-color: #E2E8F0; selection-background-color: #D6E4F0; selection-color: black; }
""")

boton_borrar.setObjectName("peligro")          # Selector por nombre: QPushButton#peligro
```

Selectores QSS: `Clase`, `Clase#nombre`, `Clase:hover`, `Clase:checked`, `Clase:disabled`, `Clase::subcontrol`, `Padre > Hijo`, `[propiedad="valor"]`.

> Guarde los estilos en un archivo `.qss` y cárguelos con `app.setStyleSheet(Path("estilo.qss").read_text(encoding="utf-8"))`.

### 11.3 Modo oscuro con paleta

```python
from PyQt6.QtGui import QPalette

def modo_oscuro(app: QApplication) -> None:
    app.setStyle("Fusion")
    p = QPalette()
    oscuro, texto = QColor(45, 45, 48), QColor(230, 230, 230)
    p.setColor(QPalette.ColorRole.Window, oscuro)
    p.setColor(QPalette.ColorRole.WindowText, texto)
    p.setColor(QPalette.ColorRole.Base, QColor(30, 30, 30))
    p.setColor(QPalette.ColorRole.AlternateBase, oscuro)
    p.setColor(QPalette.ColorRole.Text, texto)
    p.setColor(QPalette.ColorRole.Button, oscuro)
    p.setColor(QPalette.ColorRole.ButtonText, texto)
    p.setColor(QPalette.ColorRole.Highlight, QColor(42, 130, 218))
    p.setColor(QPalette.ColorRole.HighlightedText, Qt.GlobalColor.white)
    app.setPalette(p)
```

> En Qt 6.5+ también puede forzar el esquema: `app.styleHints().setColorScheme(Qt.ColorScheme.Dark)`. Librerías como **qdarktheme** o **qt-material** ofrecen temas completos.

---

## 12. Hilos: QThread y QThreadPool

Las tareas lentas en el hilo principal congelan la interfaz. **Nunca modifique widgets desde otro hilo**: comuníquese mediante señales.

### 12.1 Trabajador con `QThread`

```python
class Trabajador(QObject):
    progreso = pyqtSignal(int)
    terminado = pyqtSignal(list)
    error = pyqtSignal(str)

    def __init__(self, ruta: str):
        super().__init__()
        self.ruta = ruta
        self._cancelar = False

    @pyqtSlot()
    def ejecutar(self):
        try:
            filas = []
            lineas = Path(self.ruta).read_text(encoding="utf-8").splitlines()
            for i, linea in enumerate(lineas, 1):
                if self._cancelar:
                    break
                filas.append(linea.split(";"))
                self.progreso.emit(int(i * 100 / len(lineas)))
            self.terminado.emit(filas)
        except Exception as e:
            self.error.emit(str(e))

    def cancelar(self):
        self._cancelar = True


# En la ventana:
def importar(self, ruta):
    self.hilo = QThread()
    self.trabajador = Trabajador(ruta)
    self.trabajador.moveToThread(self.hilo)

    self.hilo.started.connect(self.trabajador.ejecutar)
    self.trabajador.progreso.connect(self.barra.setValue)
    self.trabajador.terminado.connect(self.importacion_terminada)
    self.trabajador.error.connect(lambda m: QMessageBox.critical(self, "Error", m))
    for senal in (self.trabajador.terminado, self.trabajador.error):
        senal.connect(self.hilo.quit)
    self.hilo.finished.connect(self.trabajador.deleteLater)
    self.hilo.finished.connect(self.hilo.deleteLater)

    self.hilo.start()
```

> Guarde las referencias (`self.hilo`, `self.trabajador`): si son variables locales, Python las elimina y la aplicación puede cerrarse inesperadamente.

### 12.2 Tareas sueltas con `QThreadPool`

```python
from PyQt6.QtCore import QRunnable, QThreadPool


class SenalesTarea(QObject):
    resultado = pyqtSignal(object)


class Tarea(QRunnable):
    def __init__(self, funcion, *args):
        super().__init__()
        self.funcion, self.args = funcion, args
        self.senales = SenalesTarea()

    def run(self):
        self.senales.resultado.emit(self.funcion(*self.args))


tarea = Tarea(descargar_tipos_de_cambio, "EUR")
tarea.senales.resultado.connect(self.mostrar_tipos)
QThreadPool.globalInstance().start(tarea)
```

---

## 13. Qt Designer

**Qt Designer** permite diseñar ventanas arrastrando widgets y guarda el resultado en archivos `.ui` (XML).

### 13.1 Obtener Qt Designer

- Con `pip install pyqt6-tools` (comando `pyqt6-tools designer`, según disponibilidad para su plataforma).
- Con `pip install PySide6` (comando `pyside6-designer`), que genera archivos `.ui` compatibles.
- Instalando Qt desde qt.io.

### 13.2 Cargar un `.ui` en tiempo de ejecución

```python
from PyQt6 import uic


class Ventana(QMainWindow):
    def __init__(self):
        super().__init__()
        uic.loadUi("ventana.ui", self)           # Los widgets quedan como atributos (self.botonGuardar…)
        self.botonGuardar.clicked.connect(self.guardar)
```

### 13.3 Convertir el `.ui` a código Python

```bash
pyuic6 ventana.ui -o ui_ventana.py
```

```python
from ui_ventana import Ui_MainWindow


class Ventana(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.botonGuardar.clicked.connect(self.guardar)
```

> No edite el archivo generado: se sobrescribe al regenerarlo. Ponga la lógica en su propia clase.

---

## 14. Configuración persistente (QSettings)

Guarda preferencias en el lugar adecuado de cada sistema (registro de Windows, `~/.config` en Linux, `plist` en macOS):

```python
QApplication.setOrganizationName("MiEmpresa")
QApplication.setApplicationName("GestorInventario")

ajustes = QSettings()
ajustes.setValue("ventana/geometria", self.saveGeometry())
ajustes.setValue("ventana/estado", self.saveState())           # Barras y paneles
ajustes.setValue("ultima_carpeta", "C:/Datos")
ajustes.setValue("modo_oscuro", True)

geometria = ajustes.value("ventana/geometria")
if geometria:
    self.restoreGeometry(geometria)
carpeta = ajustes.value("ultima_carpeta", "", type=str)
oscuro = ajustes.value("modo_oscuro", False, type=bool)
```

> Indique `type=` al leer: algunos sistemas devuelven los valores como texto.

---

## 15. Otras funcionalidades

| Módulo / clase | Uso |
|---|---|
| `QtCharts` (`PyQt6-Charts`) | Gráficos de barras, líneas, sectores. |
| Matplotlib (`FigureCanvasQTAgg`) | Gráficos científicos dentro de la aplicación. |
| `QtWebEngineWidgets` | Navegador web integrado (`QWebEngineView`). |
| `QtPrintSupport` | Impresión y exportación a PDF (`QPrinter`). |
| `QtMultimedia` | Audio y vídeo. |
| `QtNetwork` | Peticiones HTTP, sockets. |
| `QSystemTrayIcon` | Icono en la bandeja del sistema. |
| `QClipboard` | Portapapeles (`QApplication.clipboard()`). |
| `QTranslator` | Traducción de la interfaz. |
| `QFileSystemWatcher` | Vigilar cambios en archivos. |

Gráfico de Matplotlib:

```python
from matplotlib.backends.backend_qtagg import FigureCanvasQTAgg
from matplotlib.figure import Figure

figura = Figure(figsize=(5, 3))
lienzo = FigureCanvasQTAgg(figura)
ax = figura.add_subplot()
ax.bar(["Norte", "Sur"], [120, 95])
layout.addWidget(lienzo)
lienzo.draw()
```

Exportar un documento a PDF:

```python
from PyQt6.QtPrintSupport import QPrinter
from PyQt6.QtGui import QTextDocument

documento = QTextDocument()
documento.setHtml("<h1>Informe</h1><p>Contenido…</p>")
impresora = QPrinter(QPrinter.PrinterMode.HighResolution)
impresora.setOutputFormat(QPrinter.OutputFormat.PdfFormat)
impresora.setOutputFileName("informe.pdf")
documento.print(impresora)
```

---

## 16. Distribución y licencia

### 16.1 Crear un ejecutable

```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name Inventario --icon icono.ico main.py
```

Alternativas: **Nuitka** (compila a C, ejecutables más rápidos), **briefcase**, **cx_Freeze**.

### 16.2 Licencia

- PyQt6 se distribuye bajo **GPL v3**: si distribuye una aplicación que lo usa, debe distribuirla también bajo una licencia compatible con la GPL (con su código fuente), **o** adquirir una licencia comercial de Riverbank.
- El uso interno en una empresa (sin distribuir el software a terceros) no obliga a publicar el código.
- PySide6 (LGPL) permite distribuir aplicaciones de código cerrado cumpliendo sus condiciones (enlazado dinámico, permitir sustituir la librería).

> Esta información es orientativa; consulte los términos de licencia o asesoramiento legal para casos concretos.

---

## 17. Ejemplo práctico completo

**Gestor de inventario** con: `QMainWindow`, modelo propio (`QAbstractTableModel`) con filtro y ordenación, resaltado de stock bajo, buscador y filtro por categoría, diálogo de alta/edición con validadores, eliminación con confirmación, importación de CSV en un `QThread` con barra de progreso, exportación a CSV, acciones con atajos, menú contextual, barra de estado, modo oscuro y configuración persistente con `QSettings`.

```python
"""
Gestor de inventario con PyQt6.
Requisitos: pip install PyQt6
"""
import csv
import sys
import time
from dataclasses import asdict, dataclass, fields
from pathlib import Path

from PyQt6.QtCore import (QAbstractTableModel, QModelIndex, QObject, QSettings, QSortFilterProxyModel, Qt,
                          QThread, pyqtSignal, pyqtSlot)
from PyQt6.QtGui import QAction, QColor, QFont, QKeySequence, QPalette
from PyQt6.QtWidgets import (QAbstractItemView, QApplication, QComboBox, QDialog, QDialogButtonBox,
                             QDoubleSpinBox, QFileDialog, QFormLayout, QHBoxLayout, QHeaderView, QLabel,
                             QLineEdit, QMainWindow, QMenu, QMessageBox, QProgressBar, QSpinBox, QTableView,
                             QVBoxLayout, QWidget)

CATEGORIAS = ["Periféricos", "Pantallas", "Equipos", "Audio", "Accesorios"]
STOCK_MINIMO = 10


@dataclass
class Producto:
    codigo: str
    nombre: str
    categoria: str
    precio: float
    stock: int


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# ------------------------------------------------------------------ Modelo
class ModeloInventario(QAbstractTableModel):
    COLUMNAS = [("codigo", "Código"), ("nombre", "Nombre"), ("categoria", "Categoría"),
                ("precio", "Precio"), ("stock", "Stock"), ("valor", "Valor")]
    cambiado = pyqtSignal()

    def __init__(self, productos: list[Producto] | None = None):
        super().__init__()
        self.productos = productos or []

    def rowCount(self, padre=QModelIndex()):
        return 0 if padre.isValid() else len(self.productos)

    def columnCount(self, padre=QModelIndex()):
        return len(self.COLUMNAS)

    def _valor(self, p: Producto, clave: str):
        return round(p.precio * p.stock, 2) if clave == "valor" else getattr(p, clave)

    def data(self, indice, rol=Qt.ItemDataRole.DisplayRole):
        if not indice.isValid():
            return None
        p = self.productos[indice.row()]
        clave = self.COLUMNAS[indice.column()][0]
        valor = self._valor(p, clave)
        if rol == Qt.ItemDataRole.DisplayRole:
            return euros(valor) if clave in ("precio", "valor") else str(valor)
        if rol in (Qt.ItemDataRole.UserRole, Qt.ItemDataRole.EditRole):
            return valor
        if rol == Qt.ItemDataRole.TextAlignmentRole and clave in ("precio", "stock", "valor"):
            return Qt.AlignmentFlag.AlignRight | Qt.AlignmentFlag.AlignVCenter
        if rol == Qt.ItemDataRole.ForegroundRole and clave == "stock" and p.stock < STOCK_MINIMO:
            return QColor("#E74C3C")
        if rol == Qt.ItemDataRole.FontRole and clave == "stock" and p.stock < STOCK_MINIMO:
            f = QFont(); f.setBold(True); return f
        if rol == Qt.ItemDataRole.ToolTipRole and p.stock < STOCK_MINIMO:
            return f"Stock por debajo del mínimo ({STOCK_MINIMO})"
        return None

    def headerData(self, seccion, orientacion, rol=Qt.ItemDataRole.DisplayRole):
        if orientacion == Qt.Orientation.Horizontal and rol == Qt.ItemDataRole.DisplayRole:
            return self.COLUMNAS[seccion][1]
        return None

    def existe(self, codigo: str) -> bool:
        return any(p.codigo == codigo for p in self.productos)

    def anadir(self, producto: Producto) -> None:
        fila = len(self.productos)
        self.beginInsertRows(QModelIndex(), fila, fila)
        self.productos.append(producto)
        self.endInsertRows()
        self.cambiado.emit()

    def actualizar(self, fila: int, producto: Producto) -> None:
        self.productos[fila] = producto
        self.dataChanged.emit(self.index(fila, 0), self.index(fila, self.columnCount() - 1))
        self.cambiado.emit()

    def eliminar(self, fila: int) -> None:
        self.beginRemoveRows(QModelIndex(), fila, fila)
        del self.productos[fila]
        self.endRemoveRows()
        self.cambiado.emit()

    def reemplazar(self, productos: list[Producto]) -> None:
        self.beginResetModel()
        self.productos = productos
        self.endResetModel()
        self.cambiado.emit()


class FiltroInventario(QSortFilterProxyModel):
    def __init__(self):
        super().__init__()
        self.texto = ""
        self.categoria = "Todas"
        self.solo_stock_bajo = False
        self.setSortRole(Qt.ItemDataRole.UserRole)

    def establecer(self, texto=None, categoria=None, solo_stock_bajo=None):
        if texto is not None:
            self.texto = texto.lower()
        if categoria is not None:
            self.categoria = categoria
        if solo_stock_bajo is not None:
            self.solo_stock_bajo = solo_stock_bajo
        self.invalidate()

    def filterAcceptsRow(self, fila, padre):
        p = self.sourceModel().productos[fila]
        return ((self.categoria == "Todas" or p.categoria == self.categoria)
                and (not self.solo_stock_bajo or p.stock < STOCK_MINIMO)
                and (self.texto in p.codigo.lower() or self.texto in p.nombre.lower()))


# ------------------------------------------------------------------ Importación en segundo plano
class Importador(QObject):
    progreso = pyqtSignal(int)
    terminado = pyqtSignal(list, list)          # productos, errores

    def __init__(self, ruta: str):
        super().__init__()
        self.ruta = ruta

    @pyqtSlot()
    def ejecutar(self):
        productos, errores = [], []
        with open(self.ruta, encoding="utf-8-sig", newline="") as f:
            filas = list(csv.DictReader(f, delimiter=";"))
        for i, fila in enumerate(filas, 1):
            try:
                productos.append(Producto(fila["codigo"].strip().upper(), fila["nombre"].strip(),
                                          fila["categoria"].strip(),
                                          float(fila["precio"].replace(",", ".")), int(fila["stock"])))
            except (KeyError, ValueError) as e:
                errores.append(f"Línea {i + 1}: {e}")
            time.sleep(0.002)                   # Simula un proceso lento
            self.progreso.emit(int(i * 100 / max(1, len(filas))))
        self.terminado.emit(productos, errores)


# ------------------------------------------------------------------ Diálogo
class DialogoProducto(QDialog):
    def __init__(self, padre=None, producto: Producto | None = None, codigos_existentes=()):
        super().__init__(padre)
        self.setWindowTitle("Editar producto" if producto else "Nuevo producto")
        self.setMinimumWidth(380)
        self.codigos_existentes = set(codigos_existentes)
        self.edicion = producto is not None

        self.codigo = QLineEdit(producto.codigo if producto else "", maxLength=10, enabled=not self.edicion)
        self.nombre = QLineEdit(producto.nombre if producto else "")
        self.categoria = QComboBox()
        self.categoria.addItems(CATEGORIAS)
        if producto:
            self.categoria.setCurrentText(producto.categoria)
        self.precio = QDoubleSpinBox(minimum=0, maximum=99_999, decimals=2, suffix=" €",
                                     value=producto.precio if producto else 0)
        self.stock = QSpinBox(minimum=0, maximum=1_000_000, value=producto.stock if producto else 0)
        self.error = QLabel(styleSheet="color: #E74C3C")

        formulario = QFormLayout()
        for etiqueta, widget in [("Código *", self.codigo), ("Nombre *", self.nombre), ("Categoría", self.categoria),
                                 ("Precio", self.precio), ("Stock", self.stock)]:
            formulario.addRow(etiqueta, widget)

        botones = QDialogButtonBox(QDialogButtonBox.StandardButton.Save | QDialogButtonBox.StandardButton.Cancel)
        botones.accepted.connect(self.validar)
        botones.rejected.connect(self.reject)

        layout = QVBoxLayout(self)
        layout.addLayout(formulario)
        layout.addWidget(self.error)
        layout.addWidget(botones)

    def validar(self):
        codigo = self.codigo.text().strip().upper()
        if not codigo or not self.nombre.text().strip():
            self.error.setText("El código y el nombre son obligatorios.")
        elif not self.edicion and codigo in self.codigos_existentes:
            self.error.setText(f"Ya existe un producto con el código {codigo}.")
        else:
            self.accept()

    def producto(self) -> Producto:
        return Producto(self.codigo.text().strip().upper(), self.nombre.text().strip(),
                        self.categoria.currentText(), self.precio.value(), self.stock.value())


# ------------------------------------------------------------------ Ventana principal
class VentanaPrincipal(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Gestor de inventario")
        self.resize(980, 620)
        self.ajustes = QSettings()
        self.hay_cambios = False

        self.modelo = ModeloInventario(self._datos_ejemplo())
        self.filtro = FiltroInventario()
        self.filtro.setSourceModel(self.modelo)
        self.modelo.cambiado.connect(self._al_cambiar_modelo)

        self._crear_acciones()
        self._crear_menu_y_barras()
        self._crear_interfaz()
        self._restaurar_ajustes()
        self._actualizar_estado()

    @staticmethod
    def _datos_ejemplo() -> list[Producto]:
        return [Producto("TEC-01", "Teclado mecánico", "Periféricos", 59.90, 120),
                Producto("RAT-01", "Ratón inalámbrico", "Periféricos", 24.50, 8),
                Producto("MON-27", "Monitor 27\"", "Pantallas", 219.00, 35),
                Producto("POR-14", "Portátil 14\"", "Equipos", 899.00, 4),
                Producto("AUR-BT", "Auriculares BT", "Audio", 79.00, 60),
                Producto("CAB-HD", "Cable HDMI 2 m", "Accesorios", 9.90, 500)]

    # ---------- Construcción
    def _crear_acciones(self):
        self.acc_nuevo = QAction("Nuevo", self, shortcut=QKeySequence.StandardKey.New, triggered=self.nuevo)
        self.acc_editar = QAction("Editar", self, shortcut="Ctrl+E", triggered=self.editar, enabled=False)
        self.acc_eliminar = QAction("Eliminar", self, shortcut=QKeySequence.StandardKey.Delete,
                                    triggered=self.eliminar, enabled=False)
        self.acc_importar = QAction("Importar CSV…", self, shortcut="Ctrl+I", triggered=self.importar)
        self.acc_exportar = QAction("Exportar CSV…", self, shortcut="Ctrl+Shift+E", triggered=self.exportar)
        self.acc_oscuro = QAction("Modo oscuro", self, checkable=True, toggled=self.aplicar_tema)
        self.acc_salir = QAction("Salir", self, shortcut=QKeySequence.StandardKey.Quit, triggered=self.close)

    def _crear_menu_y_barras(self):
        archivo = self.menuBar().addMenu("&Archivo")
        archivo.addActions([self.acc_importar, self.acc_exportar])
        archivo.addSeparator()
        archivo.addAction(self.acc_salir)
        productos = self.menuBar().addMenu("&Productos")
        productos.addActions([self.acc_nuevo, self.acc_editar, self.acc_eliminar])
        self.menuBar().addMenu("&Ver").addAction(self.acc_oscuro)

        barra = self.addToolBar("Principal")
        barra.setObjectName("barra_principal")
        barra.setMovable(False)
        barra.addActions([self.acc_nuevo, self.acc_editar, self.acc_eliminar])
        barra.addSeparator()
        barra.addActions([self.acc_importar, self.acc_exportar])

        self.etiqueta_estado = QLabel()
        self.statusBar().addPermanentWidget(self.etiqueta_estado)
        self.barra_progreso = QProgressBar(maximumWidth=180, visible=False)
        self.statusBar().addPermanentWidget(self.barra_progreso)

    def _crear_interfaz(self):
        self.buscador = QLineEdit(placeholderText="Buscar por código o nombre…", clearButtonEnabled=True)
        self.buscador.textChanged.connect(lambda t: self.filtro.establecer(texto=t))
        self.combo_categoria = QComboBox()
        self.combo_categoria.addItems(["Todas"] + CATEGORIAS)
        self.combo_categoria.currentTextChanged.connect(lambda c: self.filtro.establecer(categoria=c))
        self.combo_stock = QComboBox()
        self.combo_stock.addItems(["Todo el stock", f"Stock bajo (< {STOCK_MINIMO})"])
        self.combo_stock.currentIndexChanged.connect(lambda i: self.filtro.establecer(solo_stock_bajo=i == 1))

        filtros = QHBoxLayout()
        filtros.addWidget(self.buscador, stretch=1)
        filtros.addWidget(QLabel("Categoría:"))
        filtros.addWidget(self.combo_categoria)
        filtros.addWidget(self.combo_stock)

        self.tabla = QTableView()
        self.tabla.setModel(self.filtro)
        self.tabla.setSortingEnabled(True)
        self.tabla.sortByColumn(1, Qt.SortOrder.AscendingOrder)
        self.tabla.setSelectionBehavior(QAbstractItemView.SelectionBehavior.SelectRows)
        self.tabla.setSelectionMode(QAbstractItemView.SelectionMode.SingleSelection)
        self.tabla.setEditTriggers(QAbstractItemView.EditTrigger.NoEditTriggers)
        self.tabla.setAlternatingRowColors(True)
        self.tabla.verticalHeader().setVisible(False)
        cabecera = self.tabla.horizontalHeader()
        cabecera.setSectionResizeMode(QHeaderView.ResizeMode.ResizeToContents)
        cabecera.setSectionResizeMode(1, QHeaderView.ResizeMode.Stretch)
        self.tabla.doubleClicked.connect(self.editar)
        self.tabla.selectionModel().selectionChanged.connect(self._al_cambiar_seleccion)
        self.tabla.setContextMenuPolicy(Qt.ContextMenuPolicy.CustomContextMenu)
        self.tabla.customContextMenuRequested.connect(self._menu_contextual)

        central = QWidget()
        layout = QVBoxLayout(central)
        layout.addLayout(filtros)
        layout.addWidget(self.tabla)
        self.setCentralWidget(central)

    # ---------- Utilidades
    def _fila_seleccionada(self) -> int | None:
        indices = self.tabla.selectionModel().selectedRows()
        return self.filtro.mapToSource(indices[0]).row() if indices else None

    def _al_cambiar_seleccion(self):
        hay = self._fila_seleccionada() is not None
        self.acc_editar.setEnabled(hay)
        self.acc_eliminar.setEnabled(hay)

    def _al_cambiar_modelo(self):
        self.hay_cambios = True
        self.setWindowTitle("Gestor de inventario *")
        self._actualizar_estado()

    def _actualizar_estado(self):
        productos = self.modelo.productos
        valor = sum(p.precio * p.stock for p in productos)
        bajos = sum(p.stock < STOCK_MINIMO for p in productos)
        self.etiqueta_estado.setText(f"{len(productos)} productos · valor {euros(valor)} · {bajos} con stock bajo")

    def _menu_contextual(self, pos):
        if self.tabla.indexAt(pos).isValid():
            menu = QMenu(self)
            menu.addActions([self.acc_editar, self.acc_eliminar])
            menu.exec(self.tabla.viewport().mapToGlobal(pos))

    # ---------- Acciones
    def nuevo(self):
        dialogo = DialogoProducto(self, codigos_existentes=[p.codigo for p in self.modelo.productos])
        if dialogo.exec() == QDialog.DialogCode.Accepted:
            self.modelo.anadir(dialogo.producto())
            self.statusBar().showMessage("Producto añadido", 3000)

    def editar(self):
        fila = self._fila_seleccionada()
        if fila is None:
            return
        dialogo = DialogoProducto(self, self.modelo.productos[fila])
        if dialogo.exec() == QDialog.DialogCode.Accepted:
            self.modelo.actualizar(fila, dialogo.producto())
            self.statusBar().showMessage("Producto actualizado", 3000)

    def eliminar(self):
        fila = self._fila_seleccionada()
        if fila is None:
            return
        p = self.modelo.productos[fila]
        r = QMessageBox.question(self, "Eliminar", f"¿Eliminar «{p.nombre}» ({p.codigo})?",
                                 QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No,
                                 QMessageBox.StandardButton.No)
        if r == QMessageBox.StandardButton.Yes:
            self.modelo.eliminar(fila)

    def importar(self):
        carpeta = self.ajustes.value("ultima_carpeta", "", type=str)
        ruta, _ = QFileDialog.getOpenFileName(self, "Importar inventario", carpeta, "CSV (*.csv)")
        if ruta:
            self.ajustes.setValue("ultima_carpeta", str(Path(ruta).parent))
            self.iniciar_importacion(ruta)

    def iniciar_importacion(self, ruta: str):
        self.acc_importar.setEnabled(False)
        self.barra_progreso.setValue(0)
        self.barra_progreso.setVisible(True)

        self.hilo = QThread(self)
        self.importador = Importador(ruta)
        self.importador.moveToThread(self.hilo)
        self.hilo.started.connect(self.importador.ejecutar)
        self.importador.progreso.connect(self.barra_progreso.setValue)
        self.importador.terminado.connect(self._importacion_terminada)
        self.importador.terminado.connect(self.hilo.quit)
        self.hilo.finished.connect(self.importador.deleteLater)
        self.hilo.start()

    def _importacion_terminada(self, productos: list, errores: list):
        self.barra_progreso.setVisible(False)
        self.acc_importar.setEnabled(True)
        existentes = {p.codigo: i for i, p in enumerate(self.modelo.productos)}
        nuevos = actualizados = 0
        for p in productos:
            if p.codigo in existentes:
                self.modelo.actualizar(existentes[p.codigo], p)
                actualizados += 1
            else:
                self.modelo.anadir(p)
                nuevos += 1
        mensaje = f"Importación finalizada: {nuevos} nuevos, {actualizados} actualizados."
        if errores:
            mensaje += f"\n\n{len(errores)} líneas con errores:\n" + "\n".join(errores[:5])
        self.statusBar().showMessage(mensaje.splitlines()[0], 5000)
        QMessageBox.information(self, "Importación", mensaje)

    def exportar(self, ruta: str | None = None):
        if ruta is None:
            ruta, _ = QFileDialog.getSaveFileName(self, "Exportar", "inventario.csv", "CSV (*.csv)")
        if not ruta:
            return
        with open(ruta, "w", encoding="utf-8-sig", newline="") as f:
            escritor = csv.DictWriter(f, fieldnames=[c.name for c in fields(Producto)], delimiter=";")
            escritor.writeheader()
            for fila in range(self.filtro.rowCount()):              # Exporta lo que se ve (filtrado y ordenado)
                p = self.modelo.productos[self.filtro.mapToSource(self.filtro.index(fila, 0)).row()]
                escritor.writerow({**asdict(p), "precio": f"{p.precio:.2f}".replace(".", ",")})
        self.hay_cambios = False
        self.setWindowTitle("Gestor de inventario")
        self.statusBar().showMessage(f"Exportado a {Path(ruta).name}", 4000)

    def aplicar_tema(self, oscuro: bool):
        app = QApplication.instance()
        app.setStyle("Fusion")
        if not oscuro:
            app.setPalette(app.style().standardPalette())
            return
        p = QPalette()
        base, texto = QColor(43, 43, 46), QColor(225, 225, 225)
        for rol, color in [(QPalette.ColorRole.Window, base), (QPalette.ColorRole.WindowText, texto),
                           (QPalette.ColorRole.Base, QColor(30, 30, 32)), (QPalette.ColorRole.AlternateBase, base),
                           (QPalette.ColorRole.Text, texto), (QPalette.ColorRole.Button, base),
                           (QPalette.ColorRole.ButtonText, texto), (QPalette.ColorRole.ToolTipBase, base),
                           (QPalette.ColorRole.ToolTipText, texto),
                           (QPalette.ColorRole.Highlight, QColor(42, 130, 218)),
                           (QPalette.ColorRole.HighlightedText, QColor("white"))]:
            p.setColor(rol, color)
        app.setPalette(p)

    # ---------- Persistencia y cierre
    def _restaurar_ajustes(self):
        if geometria := self.ajustes.value("geometria"):
            self.restoreGeometry(geometria)
        if estado := self.ajustes.value("estado"):
            self.restoreState(estado)
        self.acc_oscuro.setChecked(self.ajustes.value("modo_oscuro", False, type=bool))

    def closeEvent(self, evento):
        if self.hay_cambios:
            r = QMessageBox.question(self, "Salir", "Hay cambios sin exportar. ¿Salir de todos modos?")
            if r != QMessageBox.StandardButton.Yes:
                evento.ignore()
                return
        self.ajustes.setValue("geometria", self.saveGeometry())
        self.ajustes.setValue("estado", self.saveState())
        self.ajustes.setValue("modo_oscuro", self.acc_oscuro.isChecked())
        evento.accept()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    app.setOrganizationName("MiEmpresa")
    app.setApplicationName("GestorInventario")
    app.setStyle("Fusion")
    ventana = VentanaPrincipal()
    ventana.show()
    sys.exit(app.exec())
```

Formato del CSV de importación (`;` como separador):

```
codigo;nombre;categoria;precio;stock
WEB-HD;Webcam HD;Periféricos;49,00;30
RAT-01;Ratón inalámbrico;Periféricos;22,90;150
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `QWidget: Must construct a QApplication before a QWidget` | Se creó un widget antes que `QApplication`. | Cree `QApplication(sys.argv)` primero. |
| La ventana aparece y desaparece | La variable de la ventana es local y se elimina. | Guarde la referencia (`self.ventana = …` o variable en `main`). |
| `AttributeError: type object 'Qt' has no attribute 'AlignCenter'` | Enumeraciones al estilo PyQt5. | Use `Qt.AlignmentFlag.AlignCenter`. |
| `AttributeError: 'QApplication' object has no attribute 'exec_'` | Método de PyQt5. | Use `app.exec()`. |
| `ImportError: cannot import name 'QAction' from 'PyQt6.QtWidgets'` | `QAction` se movió a `QtGui`. | `from PyQt6.QtGui import QAction`. |
| `qt.qpa.plugin: Could not load the Qt platform plugin "xcb"` (Linux) | Faltan librerías del sistema. | Instale `libxcb-cursor0` y dependencias de Qt. |
| La interfaz se congela | Tarea larga en el hilo principal. | `QThread` / `QThreadPool` con señales. |
| `QObject::setParent: Cannot set parent, new parent is in a different thread` / cierres inesperados | Se modificaron widgets desde otro hilo. | Comuníquese solo mediante señales. |
| `QThread: Destroyed while thread is still running` | Se perdió la referencia al hilo. | Guárdela como atributo y espere a que termine. |
| La tabla no se actualiza al cambiar los datos | No se notificó a la vista. | Use `beginInsertRows/endInsertRows`, `dataChanged.emit()` o `beginResetModel/endResetModel`. |
| La fila editada no es la seleccionada | Se usó el índice del proxy en el modelo original. | Convierta con `proxy.mapToSource(indice)`. |
| `TypeError: ... has unexpected type` en señales | El tipo emitido no coincide con el declarado. | Revise `pyqtSignal(tipo)` y `emit(...)`. |
| El estilo QSS no se aplica a un widget | Selector incorrecto o falta `setObjectName`. | Revise el selector (`QPushButton#nombre`). |

---

## 19. Buenas prácticas

- Organice cada ventana y diálogo en su propia clase; separe la interfaz de la lógica de negocio.
- Use **layouts** siempre; evite posiciones y tamaños fijos.
- Use `QAction` para órdenes compartidas por menús, barras y atajos.
- Para listas y tablas con datos reales, use **modelo/vista** con `QSortFilterProxyModel`.
- Ejecute las tareas lentas en hilos y comuníquese mediante **señales**.
- Use `QSettings` para recordar el tamaño de la ventana y las preferencias del usuario.
- Use el estilo `Fusion` y hojas de estilo QSS en archivos separados para un aspecto coherente.
- Valide los datos en los diálogos y confirme las acciones destructivas.
- Diseñe las ventanas complejas con **Qt Designer** y mantenga la lógica en sus clases.
- Revise los requisitos de licencia antes de distribuir la aplicación.

---

## 20. Referencia rápida

```python
app = QApplication(sys.argv); v = Ventana(); v.show(); sys.exit(app.exec())

QLabel / QPushButton / QLineEdit / QTextEdit / QComboBox / QCheckBox / QRadioButton
QSpinBox / QDoubleSpinBox / QDateEdit / QSlider / QProgressBar / QGroupBox / QTabWidget / QSplitter
QVBoxLayout / QHBoxLayout / QGridLayout / QFormLayout · addWidget / addLayout / addStretch

widget.senal.connect(slot) · senal = pyqtSignal(tipo) · senal.emit(valor) · QTimer.singleShot(ms, f)
QAction(texto, padre, shortcut=, triggered=) · menuBar().addMenu() · addToolBar() · statusBar().showMessage()
QMessageBox.information/question · QFileDialog.getOpenFileName · QInputDialog.getText · QDialog.exec()

QAbstractTableModel: rowCount, columnCount, data(indice, rol), headerData, flags, setData
QSortFilterProxyModel: setSourceModel, setFilterFixedString, mapToSource · QTableView.setModel
QThread + QObject.moveToThread + señales · QThreadPool.globalInstance().start(QRunnable)
QSettings().setValue / value(clave, defecto, type=) · app.setStyle("Fusion") · app.setStyleSheet(qss)
uic.loadUi("x.ui", self) · pyuic6 x.ui -o ui_x.py
```

---

## 21. Ejercicios propuestos

1. Cree un formulario de registro con validadores (DNI, email, código postal) que active el botón «Guardar» solo cuando todos los campos sean válidos.
2. Muestre un CSV grande en un `QTableView` con un modelo propio, filtro de texto y ordenación numérica correcta.
3. Diseñe la ventana del ejemplo completo en Qt Designer y cárguela con `uic.loadUi`.
4. Añada una pestaña con un gráfico de Matplotlib del valor del inventario por categoría, que se actualice al cambiar los datos.
5. Implemente la descarga de un archivo grande en un `QThread` con barra de progreso y botón de cancelar.

---

## 22. Referencias

- Documentación de PyQt6: https://www.riverbankcomputing.com/static/Docs/PyQt6/
- Documentación de Qt 6 (C++, aplicable a PyQt): https://doc.qt.io/qt-6/
- Tutoriales (Python GUIs): https://www.pythonguis.com/pyqt6-tutorial/
