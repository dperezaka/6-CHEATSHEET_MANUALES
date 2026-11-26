# Manual de uso: PyAutoGUI

> **Categoría:** Automatización · **Utilidad:** Control del ratón, el teclado y la pantalla (RPA de escritorio)
> **Tipo:** Librería de terceros (`pip install pyautogui`) · **Versión de referencia:** 0.9.54 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y requisitos por sistema](#2-instalación-y-requisitos-por-sistema)
3. [Seguridad: *fail-safe* y pausas](#3-seguridad-fail-safe-y-pausas)
4. [Coordenadas y pantalla](#4-coordenadas-y-pantalla)
5. [Control del ratón](#5-control-del-ratón)
6. [Control del teclado](#6-control-del-teclado)
7. [Texto con tildes y caracteres especiales](#7-texto-con-tildes-y-caracteres-especiales)
8. [Capturas de pantalla y color de píxeles](#8-capturas-de-pantalla-y-color-de-píxeles)
9. [Reconocimiento de imágenes en pantalla](#9-reconocimiento-de-imágenes-en-pantalla)
10. [Cuadros de diálogo](#10-cuadros-de-diálogo)
11. [Gestión de ventanas](#11-gestión-de-ventanas)
12. [Esperas y sincronización](#12-esperas-y-sincronización)
13. [Diseño de un robot robusto](#13-diseño-de-un-robot-robusto)
14. [Pantallas de alta resolución y varios monitores](#14-pantallas-de-alta-resolución-y-varios-monitores)
15. [Alternativas más fiables](#15-alternativas-más-fiables)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**PyAutoGUI** permite controlar el ratón y el teclado, tomar capturas de pantalla y localizar imágenes en ella, de forma multiplataforma (Windows, macOS y Linux con X11). Es la herramienta habitual para **automatizar aplicaciones de escritorio** que no ofrecen API, interfaz de línea de comandos ni acceso a datos: aplicaciones heredadas, ERP de escritorio, formularios repetitivos, pruebas visuales o tareas administrativas.

**Ventajas:**
- API mínima y fácil de aprender.
- Funciona con cualquier aplicación visible en pantalla, sin integración previa.
- Multiplataforma.
- Incluye reconocimiento de imágenes y protección de emergencia (*fail-safe*).

**Limitaciones:**
- **Frágil**: depende de la resolución, la escala, el tema, la posición de las ventanas y los tiempos de respuesta.
- Ocupa el ratón y el teclado: el equipo no se puede usar mientras el robot trabaja.
- No «entiende» la interfaz (no lee controles, solo píxeles).
- No funciona con la pantalla bloqueada ni en Wayland (Linux) sin configuración adicional.

| Si necesita automatizar… | Opción preferible |
|---|---|
| Una aplicación sin API, «a ciegas» | **PyAutoGUI** |
| Páginas web | Playwright, Selenium |
| Aplicaciones Windows por sus controles (más robusto) | pywinauto, uiautomation |
| Excel, Word, Outlook | openpyxl, python-docx, pywin32 (COM) |
| Archivos y datos | pandas, pathlib, APIs |
| Atajos de teclado globales y escucha de eventos | pynput, keyboard |

> Antes de automatizar la interfaz gráfica, compruebe si la aplicación ofrece API, importación de archivos, base de datos o línea de comandos: siempre será más rápido y fiable.

---

## 2. Instalación y requisitos por sistema

```bash
pip install pyautogui
pip install opencv-python pyperclip      # Opcionales: búsqueda con tolerancia y portapapeles
```

| Sistema | Requisitos adicionales |
|---|---|
| Windows | Ninguno. |
| macOS | Conceder permisos de **Accesibilidad** y **Grabación de pantalla** a la terminal/IDE (Ajustes del Sistema → Privacidad y seguridad). |
| Linux (X11) | `sudo apt install python3-tk python3-dev scrot xclip`. No compatible con Wayland: inicie sesión en «Xorg». |

```python
import pyautogui
```

---

## 3. Seguridad: *fail-safe* y pausas

Un robot descontrolado puede hacer clic y escribir en cualquier sitio. PyAutoGUI incluye dos mecanismos:

```python
pyautogui.FAILSAFE = True      # Por defecto: mover el ratón a una ESQUINA de la pantalla detiene el programa
pyautogui.PAUSE = 0.15         # Pausa automática (s) tras CADA llamada a PyAutoGUI
```

- Al activarse el *fail-safe* se lanza `pyautogui.FailSafeException`: captúrela para cerrar ordenadamente (guardar el progreso, registrar).
- **No desactive** `FAILSAFE` salvo en entornos sin usuario (máquinas virtuales dedicadas).
- `PAUSE` da tiempo a la aplicación para reaccionar; ajústelo según su velocidad.

---

## 4. Coordenadas y pantalla

El origen `(0, 0)` es la esquina **superior izquierda** del monitor principal; `x` crece hacia la derecha e `y` hacia abajo.

```python
ancho, alto = pyautogui.size()             # Resolución del monitor principal
x, y = pyautogui.position()                # Posición actual del ratón
pyautogui.onScreen(1500, 900)              # ¿Está dentro de la pantalla?
```

Herramienta para obtener coordenadas mientras diseña el robot:

```bash
python -m pyautogui        # Muestra en tiempo real la posición del ratón y el color RGB (Ctrl+C para salir)
```

---

## 5. Control del ratón

```python
pyautogui.moveTo(500, 300, duration=0.3)                      # Posición absoluta (duration: movimiento visible)
pyautogui.moveRel(100, 0)                                    # Relativo (alias: move)
pyautogui.click()                                            # En la posición actual
pyautogui.click(500, 300)                                    # Mover y hacer clic
pyautogui.click(500, 300, clicks=2, interval=0.1)            # Doble clic (o doubleClick)
pyautogui.rightClick(500, 300) ; pyautogui.middleClick() ; pyautogui.tripleClick(500, 300)   # Triple: seleccionar línea
pyautogui.mouseDown(button="left") ; pyautogui.mouseUp()
pyautogui.dragTo(800, 300, duration=0.5, button="left")      # Arrastrar (selecciones, archivos)
pyautogui.dragRel(0, 200, duration=0.5)
pyautogui.scroll(-5)                                         # Rueda: negativo = abajo
pyautogui.hscroll(5)                                         # Horizontal (Linux/macOS)

pyautogui.moveTo(500, 300, duration=0.6, tween=pyautogui.easeInOutQuad)   # Movimiento suavizado
```

---

## 6. Control del teclado

```python
pyautogui.write("Informe mensual", interval=0.03)     # Teclea caracteres (solo los del teclado; ver §7)
pyautogui.press("enter")
pyautogui.press("tab", presses=3, interval=0.1)
pyautogui.press(["down", "down", "enter"])
pyautogui.hotkey("ctrl", "s")                         # Combinación (se sueltan en orden inverso)
pyautogui.hotkey("ctrl", "shift", "esc")
with pyautogui.hold("shift"):                         # Mantener pulsada mientras se ejecuta el bloque
    pyautogui.press(["right"] * 5)
pyautogui.keyDown("alt") ; pyautogui.press("tab") ; pyautogui.keyUp("alt")
pyautogui.KEYBOARD_KEYS                                # Lista de nombres de teclas válidos
```

| Tecla | Nombre |
|---|---|
| Intro, Escape, Tabulador, Retroceso, Suprimir | `enter`, `esc`, `tab`, `backspace`, `delete` |
| Flechas | `up`, `down`, `left`, `right` |
| Inicio, Fin, Re Pág, Av Pág | `home`, `end`, `pageup`, `pagedown` |
| Modificadores | `ctrl`, `shift`, `alt`, `win`, `command` (macOS), `option` |
| Funciones | `f1` … `f24` |
| Otras | `space`, `printscreen`, `capslock`, `insert` |

> El atajo de copiar/pegar es `ctrl` en Windows/Linux y `command` en macOS: use `"command" if sys.platform == "darwin" else "ctrl"`.

---

## 7. Texto con tildes y caracteres especiales

`write()` simula pulsaciones de teclas y **no** escribe correctamente caracteres como `á`, `ñ`, `ü`, `€` o `¿` (dependen de la distribución del teclado). La solución fiable es copiar al portapapeles y pegar:

```python
import sys
import pyperclip

PEGAR = ("command", "v") if sys.platform == "darwin" else ("ctrl", "v")

def escribir(texto: str) -> None:
    """Escribe cualquier texto: ASCII tecleando y el resto pegando desde el portapapeles."""
    if texto.isascii():
        pyautogui.write(texto, interval=0.01)
    else:
        pyperclip.copy(texto)
        pyautogui.hotkey(*PEGAR)
```

---

## 8. Capturas de pantalla y color de píxeles

```python
captura = pyautogui.screenshot()                              # Imagen de Pillow
pyautogui.screenshot("evidencia.png")                         # Guardar
zona = pyautogui.screenshot(region=(100, 200, 400, 300))      # (x, y, ancho, alto)

pyautogui.pixel(250, 400)                                     # (R, G, B)
pyautogui.pixelMatchesColor(250, 400, (40, 167, 69), tolerance=20)   # ¿Color esperado?
```

Buscar un color en una región con NumPy (rápido y sin imágenes de referencia):

```python
import numpy as np

def buscar_color(color, region=None, tolerancia=12):
    """Centro de la zona de píxeles con el color dado, o None."""
    img = np.asarray(pyautogui.screenshot(region=region).convert("RGB")).astype(int)
    coincide = (np.abs(img - color) <= tolerancia).all(axis=2)
    ys, xs = np.nonzero(coincide)
    if xs.size == 0:
        return None
    dx, dy = (region[0], region[1]) if region else (0, 0)
    return int(xs.mean()) + dx, int(ys.mean()) + dy
```

---

## 9. Reconocimiento de imágenes en pantalla

Localiza en pantalla una imagen de referencia (un botón, un icono, un logotipo) capturada previamente:

```python
try:
    caja = pyautogui.locateOnScreen("boton_guardar.png", confidence=0.85, grayscale=True,
                                    region=(0, 0, 1200, 800))       # confidence requiere OpenCV
    centro = pyautogui.center(caja)
    pyautogui.click(centro)
except pyautogui.ImageNotFoundException:
    print("No se encontró el botón")

pyautogui.locateCenterOnScreen("icono.png", confidence=0.9)        # Directamente el centro
list(pyautogui.locateAllOnScreen("casilla.png", confidence=0.9))   # Todas las apariciones
pyautogui.click("boton_guardar.png")                                # Atajo: localizar y hacer clic
```

Consejos para imágenes de referencia:

- Recorte el **mínimo distintivo** (el texto del botón, no el botón entero con su fondo).
- Captúrelas en el mismo equipo, resolución, escala y tema en que se ejecutará el robot.
- Use `confidence` (0.8–0.95) y `grayscale=True` para tolerar pequeñas diferencias.
- Limite la búsqueda con `region`: es mucho más rápida y evita falsos positivos.

> Desde PyScreeze 0.1.30, si no encuentra la imagen lanza `ImageNotFoundException` (antes devolvía `None`).

---

## 10. Cuadros de diálogo

PyAutoGUI incluye diálogos sencillos (basados en Tkinter) para interactuar con el usuario:

```python
pyautogui.alert("Proceso terminado", title="Robot")
if pyautogui.confirm("¿Iniciar la carga de 120 registros?", buttons=["Iniciar", "Cancelar"]) == "Iniciar": ...
usuario = pyautogui.prompt("Usuario del ERP:")
clave = pyautogui.password("Contraseña:", mask="*")
```

---

## 11. Gestión de ventanas

En **Windows**, PyAutoGUI incorpora PyGetWindow:

```python
ventanas = pyautogui.getWindowsWithTitle("Bloc de notas")
if ventanas:
    v = ventanas[0]
    v.activate() ; v.maximize() ; v.restore() ; v.minimize()
    v.moveTo(0, 0) ; v.resizeTo(1200, 800)
    print(v.left, v.top, v.width, v.height, v.isActive)
pyautogui.getActiveWindow()
```

En macOS y Linux, use alternativas: `pywinctl` (multiplataforma), `wmctrl`/`xdotool` (Linux) o AppleScript (macOS).

Para lanzar aplicaciones: `subprocess.Popen(["notepad.exe"])`, `os.startfile("informe.xlsx")` (Windows) o `subprocess.Popen(["open", "-a", "TextEdit"])` (macOS).

---

## 12. Esperas y sincronización

El error más habitual es actuar antes de que la aplicación esté lista. Evite `time.sleep()` fijos largos: **espere a una condición** con tiempo máximo.

```python
import time

def esperar(condicion, timeout=10.0, intervalo=0.25, descripcion="condición"):
    """Repite condicion() hasta que devuelva un valor verdadero o venza el tiempo."""
    limite = time.monotonic() + timeout
    while time.monotonic() < limite:
        resultado = condicion()
        if resultado:
            return resultado
        time.sleep(intervalo)
    raise TimeoutError(f"Tiempo agotado esperando: {descripcion}")

def imagen_visible(ruta):
    try:
        return pyautogui.locateCenterOnScreen(ruta, confidence=0.9)
    except pyautogui.ImageNotFoundException:
        return None

centro = esperar(lambda: imagen_visible("ventana_cargada.png"), timeout=30, descripcion="ventana principal")
```

---

## 13. Diseño de un robot robusto

| Principio | Cómo aplicarlo |
|---|---|
| **Anclas, no coordenadas absolutas** | Localice un elemento distintivo (imagen o color) y calcule el resto relativo a él. |
| **Teclado antes que ratón** | `Tab`, atajos y teclas de acceso son más estables que clics en posiciones. |
| **Esperar condiciones** | Confirme cada paso (pantalla cargada, mensaje de éxito) antes del siguiente. |
| **Verificar el resultado** | Compruebe un indicador de éxito o error tras cada operación. |
| **Idempotencia y reanudación** | Registre qué registros se procesaron para poder reanudar tras un fallo. |
| **Evidencias** | Captura de pantalla y registro (`logging`) en cada error. |
| **Modo simulación** | Opción para recorrer los datos sin actuar sobre la pantalla. |
| **Entorno controlado** | Resolución, escala, idioma y tema fijos; ventana en posición conocida; notificaciones desactivadas. |

---

## 14. Pantallas de alta resolución y varios monitores

- **Windows con escala > 100 %**: las coordenadas pueden no coincidir con las capturas. Ejecute el script con reconocimiento de DPI (`ctypes.windll.shcore.SetProcessDpiAwareness(2)` al inicio) o use escala 100 %.
- **macOS Retina**: las capturas tienen el doble de píxeles que las coordenadas lógicas; divida entre 2 las posiciones obtenidas de las capturas (`pyautogui.screenshot().size` frente a `pyautogui.size()`).
- **Varios monitores**: PyAutoGUI trabaja de forma fiable con el principal. Para capturas de otros monitores, `mss` es más adecuado (`pip install mss`).

---

## 15. Alternativas más fiables

```python
# Windows: pywinauto controla los elementos de la interfaz por nombre, no por píxeles
from pywinauto import Application
app = Application(backend="uia").start("notepad.exe")
app.window(title_re=".*Bloc de notas").child_window(control_type="Edit").type_keys("Hola", with_spaces=True)
```

Combine herramientas: PyAutoGUI para los pasos sin alternativa y librerías específicas para el resto (archivos, web, Office).

---

## 16. Ejemplo práctico completo

**Robot de alta de clientes en una aplicación de escritorio heredada**. Se compone de dos programas:

1. `app_altas.py`: simula la aplicación heredada (formulario Tkinter sin API) con validación de datos, que guarda los registros en `registros_app.csv`. Muestra un mensaje **verde** si el alta es correcta y **rojo** si hay errores.
2. `robot_altas.py`: lee `clientes.csv` y da de alta cada cliente en la aplicación:
   - Localiza la ventana mediante un **ancla de color** (la cabecera corporativa) y verifica que está lista.
   - Escribe los datos con el teclado (navegación con `Tab`), usando el portapapeles para tildes y «ñ».
   - Pulsa *Guardar* localizando el botón por su color y **verifica** el resultado leyendo el color del mensaje.
   - Registra cada operación, guarda capturas de los errores, permite **reanudar** y tiene **modo simulación**.

### 16.1 Aplicación de prueba (`app_altas.py`)

```python
"""
app_altas.py — Aplicación de escritorio «heredada» de alta de clientes (para practicar automatización).
Requiere: Python con Tkinter.  Uso: python app_altas.py
"""
import csv
import re
import tkinter as tk
from datetime import datetime
from pathlib import Path
from tkinter import ttk

CABECERA, VERDE, ROJO, BOTON = "#1F4E78", "#28A745", "#DC3545", "#2E8B57"
REGISTRO = Path("registros_app.csv")


class AppAltas(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("ERP Clásico — Alta de clientes")
        self.geometry("520x400+80+60")
        self.resizable(False, False)
        tk.Label(self, text="  ALTA DE CLIENTES", bg=CABECERA, fg="white", anchor="w",
                 font=("Arial", 15, "bold"), height=2).pack(fill="x")

        marco = tk.Frame(self, padx=20, pady=12)
        marco.pack(fill="x")
        self.campos: dict[str, tk.Entry | ttk.Combobox] = {}
        for fila, (clave, texto) in enumerate([("nombre", "Nombre / Razón social"), ("nif", "NIF / CIF"),
                                               ("email", "Correo electrónico"), ("ciudad", "Ciudad"),
                                               ("limite", "Límite de crédito (€)")]):
            tk.Label(marco, text=texto, anchor="w", width=22).grid(row=fila, column=0, pady=5, sticky="w")
            if clave == "ciudad":
                campo = ttk.Combobox(marco, values=["Madrid", "Barcelona", "Valencia", "Sevilla", "Bilbao"], width=28)
            else:
                campo = tk.Entry(marco, width=31)
            campo.grid(row=fila, column=1, pady=5)
            self.campos[clave] = campo

        tk.Button(self, text="Guardar", bg=BOTON, fg="white", activebackground=BOTON, width=14,
                  font=("Arial", 11, "bold"), command=self.guardar).pack(pady=6)
        self.estado = tk.Label(self, text="", height=2, font=("Arial", 10, "bold"), fg="white", bg=self.cget("bg"))
        self.estado.pack(fill="x", padx=20, pady=6)
        self.bind("<Control-s>", lambda e: self.guardar())
        self.bind("<Escape>", lambda e: self.reiniciar())
        self.limpiar()

    def limpiar(self):
        for campo in self.campos.values():
            campo.delete(0, "end")
        self.campos["nombre"].focus_set()

    def reiniciar(self):
        self.estado.config(text="", bg=self.cget("bg"))
        self.limpiar()

    def guardar(self):
        self.after(400, self._guardar)                   # Simula la latencia de la aplicación

    def _guardar(self):
        datos = {k: c.get().strip() for k, c in self.campos.items()}
        errores = []
        if len(datos["nombre"]) < 3:
            errores.append("nombre")
        if not re.fullmatch(r"[0-9A-Z]\d{7}[0-9A-Z]", datos["nif"].upper()):
            errores.append("NIF")
        if not re.fullmatch(r"[^@\s]+@[^@\s]+\.[a-z]{2,}", datos["email"].lower()):
            errores.append("correo")
        try:
            limite = float(datos["limite"].replace(".", "").replace(",", "."))
        except ValueError:
            errores.append("límite")
        if errores:
            self.estado.config(text="Error: revise " + ", ".join(errores), bg=ROJO)
            return
        nuevo = not REGISTRO.exists()
        with REGISTRO.open("a", newline="", encoding="utf-8") as f:
            escritor = csv.writer(f, delimiter=";")
            if nuevo:
                escritor.writerow(["fecha", *datos])
            escritor.writerow([datetime.now().isoformat(timespec="seconds"), *datos.values()])
        self.estado.config(text=f"Cliente guardado: {datos['nombre']}", bg=VERDE)
        self.limpiar()
        self.after(1500, lambda: self.estado.config(text="", bg=self.cget("bg")))


if __name__ == "__main__":
    AppAltas().mainloop()
```

### 16.2 Robot (`robot_altas.py`)

```python
"""
robot_altas.py — Robot de escritorio que da de alta clientes en app_altas.py a partir de un CSV.
Requiere: pip install pyautogui pyperclip numpy
Uso:
    python robot_altas.py                 # Lanza la aplicación y procesa clientes.csv
    python robot_altas.py --simulacion    # Recorre y valida los datos sin tocar la pantalla
    python robot_altas.py --sin-lanzar    # La aplicación ya está abierta
Detención de emergencia: mover el ratón a una esquina de la pantalla.
"""
from __future__ import annotations

import argparse
import csv
import json
import logging
import subprocess
import sys
import time
from datetime import datetime
from pathlib import Path

import numpy as np
import pyautogui
import pyperclip

# ---------------------------------------------------------------- Configuración
CLIENTES = Path("clientes.csv")
PROGRESO = Path("robot_progreso.json")
EVIDENCIAS = Path("evidencias")
COLOR_CABECERA = (0x1F, 0x4E, 0x78)
COLOR_BOTON = (0x2E, 0x8B, 0x57)
COLOR_OK, COLOR_ERROR = (0x28, 0xA7, 0x45), (0xDC, 0x35, 0x45)
CAMPOS = ["nombre", "nif", "email", "ciudad", "limite"]          # Orden de tabulación en la aplicación
TAMANO_VENTANA = (520, 400)
PEGAR = ("command", "v") if sys.platform == "darwin" else ("ctrl", "v")

pyautogui.FAILSAFE = True
pyautogui.PAUSE = 0.08

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)-7s %(message)s", datefmt="%H:%M:%S",
                    handlers=[logging.StreamHandler(), logging.FileHandler("robot.log", encoding="utf-8")])
log = logging.getLogger("robot")


# ---------------------------------------------------------------- Utilidades de pantalla
def esperar(condicion, timeout: float = 10.0, intervalo: float = 0.2, descripcion: str = "condición"):
    limite = time.monotonic() + timeout
    while time.monotonic() < limite:
        if resultado := condicion():
            return resultado
        time.sleep(intervalo)
    raise TimeoutError(f"Tiempo agotado esperando: {descripcion}")


def buscar_color(color, region=None, tolerancia: int = 10, minimo: int = 30):
    """Devuelve la caja (x0, y0, x1, y1) de los píxeles con ese color, o None."""
    img = np.asarray(pyautogui.screenshot(region=region).convert("RGB")).astype(np.int16)
    ys, xs = np.nonzero((np.abs(img - np.array(color)) <= tolerancia).all(axis=2))
    if xs.size < minimo:
        return None
    dx, dy = (region[0], region[1]) if region else (0, 0)
    return xs.min() + dx, ys.min() + dy, xs.max() + dx, ys.max() + dy


def centro(caja) -> tuple[int, int]:
    return (int(caja[0] + caja[2]) // 2, int(caja[1] + caja[3]) // 2)


def escribir(texto: str) -> None:
    if texto.isascii():
        pyautogui.write(texto, interval=0.01)
    else:                                                          # Tildes, ñ, €…: portapapeles
        pyperclip.copy(texto)
        pyautogui.hotkey(*PEGAR)


def evidencia(nombre: str) -> Path:
    EVIDENCIAS.mkdir(exist_ok=True)
    ruta = EVIDENCIAS / f"{datetime.now():%Y%m%d_%H%M%S}_{nombre}.png"
    pyautogui.screenshot(str(ruta))
    return ruta


# ---------------------------------------------------------------- Robot
class RobotAltas:
    def __init__(self, simulacion: bool = False):
        self.simulacion = simulacion
        self.region: tuple[int, int, int, int] | None = None       # Zona de la ventana (x, y, ancho, alto)
        self.boton: tuple[int, int] | None = None

    def localizar_aplicacion(self, timeout: float = 20) -> None:
        cabecera = esperar(lambda: buscar_color(COLOR_CABECERA, minimo=2000), timeout, descripcion="ventana de la aplicación")
        x0, y0 = int(cabecera[0]), int(cabecera[1])
        self.region = (x0, y0, TAMANO_VENTANA[0], TAMANO_VENTANA[1])
        caja_boton = esperar(lambda: buscar_color(COLOR_BOTON, self.region, minimo=200), 5, descripcion="botón Guardar")
        self.boton = centro(caja_boton)
        log.info("Aplicación localizada en %s · botón Guardar en %s", (x0, y0), self.boton)

    def zona_estado(self) -> tuple[int, int, int, int]:
        return (self.region[0], self.boton[1] + 15, self.region[2], 70)

    def mensaje(self) -> str | None:
        """Lee el mensaje de la aplicación por su color: "ok", "error" o None."""
        if buscar_color(COLOR_OK, self.zona_estado(), minimo=200):
            return "ok"
        if buscar_color(COLOR_ERROR, self.zona_estado(), minimo=200):
            return "error"
        return None

    def preparar_formulario(self) -> None:
        x, y = self.region[0] + 40, self.region[1] + 15              # Clic en la cabecera: activa la ventana
        pyautogui.click(x, y)
        pyautogui.press("esc")                                       # La aplicación limpia el formulario y el mensaje
        esperar(lambda: self.mensaje() is None, 5, descripcion="formulario limpio")   # Evita leer un mensaje anterior

    def alta(self, cliente: dict[str, str]) -> tuple[bool, str]:
        self.preparar_formulario()
        for i, campo in enumerate(CAMPOS):
            escribir(cliente[campo])
            if i < len(CAMPOS) - 1:
                pyautogui.press("tab")
        pyautogui.click(*self.boton)
        resultado = esperar(self.mensaje, timeout=8, descripcion="mensaje de la aplicación")
        if resultado == "ok":
            return True, "guardado"
        ruta = evidencia(f"error_{cliente['nif'] or 'sin_nif'}")
        return False, f"rechazado por la aplicación (captura: {ruta.name})"

    def ejecutar(self, clientes: list[dict[str, str]]) -> dict:
        hechos = set(json.loads(PROGRESO.read_text())["hechos"]) if PROGRESO.exists() else set()
        pendientes = [c for c in clientes if c["nif"] not in hechos]
        log.info("Clientes: %d · ya procesados: %d · pendientes: %d", len(clientes), len(hechos), len(pendientes))
        resumen = {"ok": 0, "rechazados": 0, "detalle": []}
        if not self.simulacion:
            self.localizar_aplicacion()
        try:
            for n, cliente in enumerate(pendientes, start=1):
                if self.simulacion:
                    ok, motivo = True, "simulado"
                else:
                    ok, motivo = self.alta(cliente)
                resumen["ok" if ok else "rechazados"] += 1
                resumen["detalle"].append({"nif": cliente["nif"], "nombre": cliente["nombre"], "ok": ok, "motivo": motivo})
                (log.info if ok else log.warning)("[%d/%d] %s · %s", n, len(pendientes), cliente["nombre"], motivo)
                hechos.add(cliente["nif"])
                if not self.simulacion:
                    PROGRESO.write_text(json.dumps({"hechos": sorted(hechos)}))
        except pyautogui.FailSafeException:
            log.error("Detenido por el usuario (fail-safe). Se reanudará desde el siguiente cliente pendiente.")
        except TimeoutError as e:
            log.error("%s · captura: %s", e, evidencia("timeout").name)
        return resumen


def crear_clientes_ejemplo(ruta: Path) -> None:
    filas = [
        ("Ferretería Núñez S.L.", "B12345678", "compras@ferrenunez.es", "Valencia", "15.000"),
        ("Óptica Martín", "12345678Z", "info@opticamartin.es", "Madrid", "3.500,50"),
        ("Construcciones Peña e Hijos", "A87654321", "admin@cpena.es", "Sevilla", "40.000"),
        ("Librería Ágora", "X1234567L", "correo-sin-arroba.es", "Bilbao", "2.000"),     # Correo no válido
        ("Distribuciones Ibáñez", "B11223344", "pedidos@dibanez.es", "Barcelona", "22.750"),
    ]
    with ruta.open("w", newline="", encoding="utf-8") as f:
        escritor = csv.writer(f, delimiter=";")
        escritor.writerow(CAMPOS)
        escritor.writerows(filas)


def main() -> int:
    parser = argparse.ArgumentParser(description="Robot de alta de clientes")
    parser.add_argument("--simulacion", action="store_true")
    parser.add_argument("--sin-lanzar", action="store_true")
    parser.add_argument("--reiniciar", action="store_true", help="Olvida el progreso guardado")
    args = parser.parse_args()

    if not CLIENTES.exists():
        crear_clientes_ejemplo(CLIENTES)
    if args.reiniciar:
        PROGRESO.unlink(missing_ok=True)
    with CLIENTES.open(encoding="utf-8") as f:
        clientes = list(csv.DictReader(f, delimiter=";"))

    proceso = None
    if not (args.simulacion or args.sin_lanzar):
        proceso = subprocess.Popen([sys.executable, str(Path(__file__).with_name("app_altas.py"))])
    try:
        resumen = RobotAltas(simulacion=args.simulacion).ejecutar(clientes)
    finally:
        if proceso:
            time.sleep(0.5)
            proceso.terminate()

    log.info("Resultado: %d guardados, %d rechazados", resumen["ok"], resumen["rechazados"])
    Path("robot_resultado.json").write_text(json.dumps(resumen, ensure_ascii=False, indent=2), encoding="utf-8")
    return 0 if resumen["rechazados"] == 0 else 1


if __name__ == "__main__":
    raise SystemExit(main())
```

Ejecución: sitúe `app_altas.py` y `robot_altas.py` en la misma carpeta, ejecute `python robot_altas.py` y no use el ratón ni el teclado hasta que termine. El cliente con el correo incorrecto será rechazado por la aplicación y quedará registrado con su captura de pantalla en `evidencias/`. Una segunda ejecución no repetirá los clientes ya procesados (use `--reiniciar` para empezar de cero).

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `pyautogui.FailSafeException` | El ratón llegó a una esquina (a propósito o por un movimiento del robot). | Comportamiento esperado: capture la excepción; evite mover el ratón a las esquinas. |
| `ImageNotFoundException` | La imagen no está visible o difiere (escala, tema, resolución). | Recapture la referencia, use `confidence` y `grayscale`, espere a que aparezca. |
| `NotImplementedError: confidence keyword … requires OpenCV` | Falta OpenCV. | `pip install opencv-python`. |
| Clics desplazados respecto a lo esperado | Escala de pantalla (DPI) o Retina. | Escala 100 % o reconocimiento de DPI; dividir entre 2 en Retina. |
| Se escriben caracteres incorrectos (tildes, «ñ», «@») | `write()` depende de la distribución del teclado. | Pegar desde el portapapeles (`pyperclip`). |
| El texto se escribe en otra ventana | La aplicación no tenía el foco. | Active la ventana (clic en zona segura o `activate()`) y verifique antes de escribir. |
| Acciones antes de que la aplicación responda | Falta de sincronización. | Esperas por condición; aumente `PAUSE`. |
| No hace nada en macOS | Sin permisos de Accesibilidad/Grabación de pantalla. | Concédalos a la terminal o IDE y reinicie. |
| `Exception: To take screenshots, you must install … gnome-screenshot` (Linux) | Falta herramienta de captura. | `sudo apt install scrot` (X11) o `gnome-screenshot`. |
| No funciona en Linux moderno | Sesión Wayland. | Inicie sesión con Xorg. |
| Funciona en un equipo y falla en otro | Resolución, escala, tema o idioma distintos. | Estandarice el entorno y use anclas relativas. |

---

## 18. Buenas prácticas

- Use PyAutoGUI como **último recurso**: busque antes API, archivos de importación o automatización específica.
- Mantenga `FAILSAFE` activado y una `PAUSE` razonable; avise al usuario antes de empezar.
- Navegue con el teclado siempre que pueda y use anclas (imagen o color) en lugar de coordenadas fijas.
- Espere condiciones con tiempo máximo y verifique el resultado de cada operación.
- Registre todo con `logging`, guarde capturas de los errores y permita reanudar.
- Separe datos, configuración (colores, rutas, tiempos) y lógica del robot.
- Pruebe primero en modo simulación y con pocos registros.
- Ejecute en un entorno estable (máquina virtual dedicada, resolución fija, notificaciones desactivadas).
- No almacene contraseñas en el código: use variables de entorno o el almacén de credenciales del sistema (`keyring`).

---

## 19. Referencia rápida

```python
pyautogui.FAILSAFE = True · pyautogui.PAUSE = 0.1 · pyautogui.size() · pyautogui.position() · python -m pyautogui
moveTo(x, y, duration=) · moveRel(dx, dy) · click(x, y, clicks=, button=) · doubleClick · rightClick · dragTo · scroll(n)
write("texto", interval=) · press("enter", presses=) · hotkey("ctrl", "s") · with hold("shift"): ... · keyDown/keyUp
screenshot(ruta, region=(x, y, w, h)) · pixel(x, y) · pixelMatchesColor(x, y, (r, g, b), tolerance=)
locateOnScreen(img, confidence=, grayscale=, region=) · locateCenterOnScreen · locateAllOnScreen · center(caja)
alert · confirm(buttons=) · prompt · password · getWindowsWithTitle(t)[0].activate() (Windows)
pyperclip.copy(texto) + hotkey("ctrl", "v")   # Texto con tildes
```

---

## 20. Ejercicios propuestos

1. Sustituya la localización por color del ejemplo por imágenes de referencia (`locateOnScreen`) y compare la robustez al cambiar el tema del sistema.
2. Añada al robot un informe final en Excel con los resultados y las rutas de las capturas.
3. Implemente reintentos: si un alta falla por tiempo de espera, reintente hasta dos veces antes de marcarla como error.
4. Automatice la exportación de un informe desde una aplicación de escritorio real de su entorno (menú → Exportar → guardar con un nombre de archivo con la fecha).
5. Escriba un script que grabe la posición del ratón y las pulsaciones de una tarea (con `pynput`) y genere el código PyAutoGUI equivalente.

---

## 21. Referencias

- Documentación oficial: https://pyautogui.readthedocs.io/
- Repositorio: https://github.com/asweigart/pyautogui
- PyScreeze (reconocimiento de imágenes): https://github.com/asweigart/pyscreeze
- pywinauto (automatización de Windows por controles): https://pywinauto.readthedocs.io/
- *Automate the Boring Stuff with Python*, capítulo de control del teclado y el ratón: https://automatetheboringstuff.com/
