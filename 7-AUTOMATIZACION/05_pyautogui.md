# Manual de uso: PyAutoGUI

> **Categoría:** Automatización y sistema · **Utilidad:** Automatizar teclado, ratón y pantalla
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y requisitos](#2-instalación-y-requisitos)
3. [Conceptos clave](#3-conceptos-clave)
4. [Seguridad: FAILSAFE y pausas](#4-seguridad-failsafe-y-pausas)
5. [Pantalla y coordenadas](#5-pantalla-y-coordenadas)
6. [Control del ratón](#6-control-del-ratón)
7. [Control del teclado](#7-control-del-teclado)
8. [Capturas de pantalla](#8-capturas-de-pantalla)
9. [Reconocimiento de imágenes](#9-reconocimiento-de-imágenes)
10. [Cuadros de diálogo](#10-cuadros-de-diálogo)
11. [Gestión de ventanas](#11-gestión-de-ventanas)
12. [Técnicas para automatizaciones robustas](#12-técnicas-para-automatizaciones-robustas)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**PyAutoGUI** permite controlar el ratón y el teclado y analizar la pantalla desde Python, simulando las acciones de una persona. Es útil para automatizar tareas repetitivas en **cualquier aplicación de escritorio**, incluidas las que no ofrecen API ni otra forma de integración.

**Casos de uso habituales:**
- Introducir datos en aplicaciones heredadas (ERP, programas antiguos).
- Rellenar formularios repetitivos.
- Automatizar flujos entre varias aplicaciones.
- Tomar capturas de pantalla periódicas.
- Pruebas manuales repetitivas de interfaces.

**Limitaciones:**
- Depende de la posición de los elementos en pantalla, la resolución y el tema visual.
- No «entiende» la aplicación: solo ve píxeles y envía pulsaciones.
- Requiere la pantalla activa y desbloqueada; el equipo no puede usarse mientras se ejecuta.

> Si la aplicación es web, use **Selenium** o **Playwright**; si tiene API o librería, úsela. PyAutoGUI es la última opción, la más frágil.

---

## 2. Instalación y requisitos

```bash
pip install pyautogui
```

Dependencias opcionales recomendadas:

```bash
pip install opencv-python      # Reconocimiento de imágenes con tolerancia (confidence)
pip install pyperclip          # Escribir texto con tildes y ñ mediante el portapapeles
pip install pygetwindow        # Gestión de ventanas (Windows; se instala con pyautogui)
```

Requisitos por sistema:

| Sistema | Requisitos |
|---|---|
| Windows | Ninguno adicional. Para controlar programas ejecutados como administrador, el script también debe ejecutarse como administrador. |
| macOS | Conceder permisos de **Accesibilidad** y **Grabación de pantalla** a la Terminal o al IDE (Ajustes del Sistema → Privacidad y seguridad). |
| Linux | Sesión X11 (no funciona correctamente en Wayland). Instalar `scrot` y `python3-tk`, `python3-dev`. |

```python
import pyautogui
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Coordenadas | `(x, y)` en píxeles. `(0, 0)` es la esquina **superior izquierda**. |
| Resolución | `pyautogui.size()`; las coordenadas válidas van de `0` a `ancho-1` y `alto-1`. |
| FAILSAFE | Mover el ratón a una esquina de la pantalla detiene el script. |
| PAUSE | Pausa automática tras cada acción. |
| Localización por imagen | Buscar un elemento en pantalla a partir de una captura recortada. |
| Escalado (DPI) | En pantallas con escalado (125 %, 150 %), las coordenadas pueden no coincidir con lo que se ve. |

---

## 4. Seguridad: FAILSAFE y pausas

### 4.1 Mecanismo de emergencia

```python
pyautogui.FAILSAFE = True      # Valor por defecto. NO lo desactive.
```

Si el script se descontrola, mueva el ratón rápidamente a **cualquier esquina** de la pantalla: se lanzará `pyautogui.FailSafeException` y el programa se detendrá.

### 4.2 Pausa entre acciones

```python
pyautogui.PAUSE = 0.5          # Medio segundo tras cada llamada
```

Da tiempo a que las aplicaciones respondan y permite al usuario reaccionar.

### 4.3 Pausa inicial

```python
import time

print("El script comenzará en 5 segundos. Sitúe la ventana correcta en primer plano.")
print("Para detenerlo, lleve el ratón a una esquina de la pantalla.")
time.sleep(5)
```

### 4.4 Capturar la interrupción

```python
try:
    ejecutar_automatizacion()
except pyautogui.FailSafeException:
    print("Automatización detenida por el usuario.")
```

---

## 5. Pantalla y coordenadas

```python
ancho, alto = pyautogui.size()          # Resolución de la pantalla principal
x, y = pyautogui.position()             # Posición actual del ratón
pyautogui.onScreen(1500, 900)           # ¿Está dentro de la pantalla?
```

### 5.1 Averiguar coordenadas

Herramienta interactiva incluida (muestra posición y color en tiempo real):

```bash
python -c "import pyautogui; pyautogui.mouseInfo()"
```

Alternativa en consola:

```python
import time
try:
    while True:
        x, y = pyautogui.position()
        r, g, b = pyautogui.pixel(x, y)
        print(f"X={x:5d}  Y={y:5d}  RGB=({r:3d},{g:3d},{b:3d})", end="\r")
        time.sleep(0.1)
except KeyboardInterrupt:
    print("\nFin")
```

### 5.2 Coordenadas relativas

Para que la automatización no dependa de la resolución, calcule posiciones relativas:

```python
ancho, alto = pyautogui.size()
centro = (ancho // 2, alto // 2)
esquina_inf_der = (ancho - 50, alto - 50)
```

O relativas a una ventana (ver sección 11).

### 5.3 Varios monitores

PyAutoGUI trabaja principalmente con la **pantalla principal**. Los monitores secundarios pueden tener coordenadas negativas o mayores que `size()`, y el soporte varía según el sistema.

---

## 6. Control del ratón

### 6.1 Mover

```python
pyautogui.moveTo(500, 300)                          # Instantáneo
pyautogui.moveTo(500, 300, duration=0.5)            # Movimiento visible de 0,5 s
pyautogui.move(100, 0, duration=0.3)                # Relativo a la posición actual
pyautogui.moveTo(500, 300, duration=1, tween=pyautogui.easeInOutQuad)   # Aceleración suave
```

### 6.2 Hacer clic

```python
pyautogui.click()                                   # Clic izquierdo en la posición actual
pyautogui.click(500, 300)                           # Mover y hacer clic
pyautogui.click(500, 300, button="right")           # Botón derecho
pyautogui.click(500, 300, button="middle")
pyautogui.doubleClick(500, 300)
pyautogui.tripleClick(500, 300)                     # Seleccionar una línea de texto
pyautogui.rightClick(500, 300)
pyautogui.click(500, 300, clicks=3, interval=0.25)
```

### 6.3 Pulsar y soltar por separado

```python
pyautogui.mouseDown(400, 300)
pyautogui.moveTo(800, 300, duration=1)
pyautogui.mouseUp()
```

### 6.4 Arrastrar

```python
pyautogui.dragTo(800, 400, duration=1)                  # Arrastra desde la posición actual
pyautogui.drag(200, 0, duration=0.5)                    # Arrastre relativo
pyautogui.dragTo(800, 400, duration=1, button="right")
```

> Algunas aplicaciones necesitan un movimiento lento (`duration` ≥ 0,5) para reconocer el arrastre.

### 6.5 Desplazamiento (*scroll*)

```python
pyautogui.scroll(-500)            # Hacia abajo (negativo)
pyautogui.scroll(500)             # Hacia arriba
pyautogui.scroll(-300, x=800, y=500)
pyautogui.hscroll(200)            # Horizontal (macOS y Linux)
```

---

## 7. Control del teclado

### 7.1 Escribir texto

```python
pyautogui.write("Informe mensual")
pyautogui.write("Texto lento", interval=0.05)   # Pausa entre caracteres
```

> **Limitación importante:** `write()` solo escribe de forma fiable caracteres del teclado inglés. Las tildes, la ñ, `¿`, `¡`, `€` y otros símbolos **no** se escriben correctamente.

### 7.2 Escribir texto en español (portapapeles)

```python
import pyperclip

def escribir(texto: str) -> None:
    pyperclip.copy(str(texto))
    pyautogui.hotkey("ctrl", "v")        # En macOS: "command", "v"

escribir("Señora Muñoz, el pedido nº 1045 asciende a 59,80 €")
```

### 7.3 Pulsar teclas

```python
pyautogui.press("enter")
pyautogui.press("tab")
pyautogui.press(["tab", "tab", "enter"])        # Secuencia
pyautogui.press("down", presses=5, interval=0.1)
```

### 7.4 Combinaciones de teclas

```python
pyautogui.hotkey("ctrl", "c")           # Copiar
pyautogui.hotkey("ctrl", "v")           # Pegar
pyautogui.hotkey("ctrl", "a")           # Seleccionar todo
pyautogui.hotkey("ctrl", "s")           # Guardar
pyautogui.hotkey("alt", "tab")          # Cambiar de ventana
pyautogui.hotkey("alt", "f4")           # Cerrar ventana
pyautogui.hotkey("win", "r")            # Ejecutar (Windows)
pyautogui.hotkey("win", "d")            # Mostrar escritorio
pyautogui.hotkey("ctrl", "shift", "esc")   # Administrador de tareas
pyautogui.hotkey("command", "space")    # Spotlight (macOS)
```

### 7.5 Mantener teclas pulsadas

```python
pyautogui.keyDown("shift")
pyautogui.press(["right"] * 10)         # Selecciona 10 caracteres
pyautogui.keyUp("shift")

with pyautogui.hold("ctrl"):            # Forma más segura: siempre se suelta
    pyautogui.press(["c"])
```

### 7.6 Nombres de teclas

| Grupo | Nombres |
|---|---|
| Edición | `enter`, `tab`, `space`, `backspace`, `delete`, `esc`, `insert` |
| Navegación | `up`, `down`, `left`, `right`, `home`, `end`, `pageup`, `pagedown` |
| Modificadores | `ctrl`, `shift`, `alt`, `altleft`, `altright`, `win`, `command`, `option` |
| Función | `f1` … `f24` |
| Bloqueo | `capslock`, `numlock`, `scrolllock` |
| Otras | `printscreen`, `volumeup`, `volumedown`, `volumemute`, `playpause` |

Lista completa:

```python
print(pyautogui.KEYBOARD_KEYS)
```

---

## 8. Capturas de pantalla

```python
captura = pyautogui.screenshot()                         # Objeto de imagen (Pillow)
captura.save("pantalla.png")

pyautogui.screenshot("pantalla.png")                     # Captura y guarda
pyautogui.screenshot("zona.png", region=(0, 0, 800, 600))   # (x, y, ancho, alto)
```

### 8.1 Leer el color de un píxel

```python
pyautogui.pixel(100, 200)                                # (R, G, B)
pyautogui.pixelMatchesColor(100, 200, (0, 128, 0))
pyautogui.pixelMatchesColor(100, 200, (0, 128, 0), tolerance=10)
```

Uso típico: comprobar si un indicador ha cambiado de color (por ejemplo, un botón activo o un semáforo verde).

### 8.2 Capturas periódicas con fecha

```python
from datetime import datetime
from pathlib import Path
import time

carpeta = Path("capturas")
carpeta.mkdir(exist_ok=True)
for _ in range(10):
    pyautogui.screenshot(carpeta / f"{datetime.now():%Y%m%d_%H%M%S}.png")
    time.sleep(60)
```

---

## 9. Reconocimiento de imágenes

Localiza elementos en pantalla a partir de una imagen de referencia (por ejemplo, un recorte del botón «Aceptar»).

### 9.1 Preparar la imagen de referencia

1. Haga una captura de la pantalla con el elemento visible.
2. Recorte **solo** el elemento (botón, icono, campo) con poco margen.
3. Guárdela en PNG (sin compresión con pérdida).
4. Use la misma resolución, escalado y tema visual que tendrá la ejecución.

### 9.2 Localizar en pantalla

```python
try:
    centro = pyautogui.locateCenterOnScreen("img/boton_aceptar.png", confidence=0.9)
    pyautogui.click(centro)
except pyautogui.ImageNotFoundException:
    print("No se encontró el botón Aceptar")
```

> Desde PyAutoGUI 0.9.41, las funciones de localización lanzan `ImageNotFoundException` cuando no encuentran la imagen. En versiones anteriores devolvían `None`.

| Función | Devuelve |
|---|---|
| `locateOnScreen(img)` | Caja `(left, top, width, height)` |
| `locateCenterOnScreen(img)` | Punto central `(x, y)` |
| `locateAllOnScreen(img)` | Generador con todas las coincidencias |
| `locate(aguja, pajar)` | Buscar una imagen dentro de otra |
| `center(caja)` | Centro de una caja |

### 9.3 Parámetros útiles

```python
pyautogui.locateOnScreen(
    "img/icono.png",
    confidence=0.85,                    # Tolerancia (requiere OpenCV). 1.0 = idéntico
    grayscale=True,                     # Más rápido; ignora diferencias de color
    region=(0, 0, 960, 540),            # Buscar solo en una zona (mucho más rápido)
)
```

### 9.4 Todas las coincidencias

```python
for caja in pyautogui.locateAllOnScreen("img/casilla_vacia.png", confidence=0.9):
    pyautogui.click(pyautogui.center(caja))
```

### 9.5 Esperar a que aparezca un elemento

```python
import time


def esperar_imagen(ruta: str, timeout: float = 15, confidence: float = 0.9,
                   region: tuple | None = None, intervalo: float = 0.5):
    """Espera hasta que la imagen aparezca y devuelve su centro."""
    limite = time.monotonic() + timeout
    while time.monotonic() < limite:
        try:
            return pyautogui.locateCenterOnScreen(ruta, confidence=confidence, region=region)
        except pyautogui.ImageNotFoundException:
            time.sleep(intervalo)
    raise TimeoutError(f"No apareció '{ruta}' en {timeout} s")


def esperar_desaparicion(ruta: str, timeout: float = 30, confidence: float = 0.9) -> None:
    limite = time.monotonic() + timeout
    while time.monotonic() < limite:
        try:
            pyautogui.locateOnScreen(ruta, confidence=confidence)
            time.sleep(0.5)
        except pyautogui.ImageNotFoundException:
            return
    raise TimeoutError(f"'{ruta}' sigue en pantalla tras {timeout} s")


pyautogui.click(esperar_imagen("img/boton_guardar.png"))
esperar_desaparicion("img/cargando.png")
```

---

## 10. Cuadros de diálogo

Ventanas simples para interactuar con el usuario durante la automatización:

```python
pyautogui.alert("Proceso finalizado correctamente.", title="Automatización")

respuesta = pyautogui.confirm("¿Desea continuar con el siguiente lote?", buttons=["Sí", "No"])
if respuesta == "No":
    raise SystemExit

nombre = pyautogui.prompt("Introduzca el código de cliente:", default="C-")
clave = pyautogui.password("Introduzca la contraseña:", mask="*")
```

`prompt` y `password` devuelven `None` si el usuario pulsa *Cancel*.

---

## 11. Gestión de ventanas

En Windows, PyAutoGUI incluye **PyGetWindow** para localizar y manipular ventanas:

```python
import pygetwindow as gw

gw.getAllTitles()                                    # Títulos de todas las ventanas
ventanas = gw.getWindowsWithTitle("Bloc de notas")   # Coincidencia parcial
ventana = ventanas[0]

ventana.activate()          # Traer al frente
ventana.maximize()
ventana.minimize()
ventana.restore()
ventana.resizeTo(1200, 800)
ventana.moveTo(0, 0)
ventana.close()

ventana.left, ventana.top, ventana.width, ventana.height
ventana.isActive, ventana.isMaximized

activa = gw.getActiveWindow()
print(activa.title)
```

### 11.1 Coordenadas relativas a una ventana

```python
def clic_en_ventana(ventana, dx: int, dy: int) -> None:
    """Clic en una posición relativa a la esquina superior izquierda de la ventana."""
    pyautogui.click(ventana.left + dx, ventana.top + dy)

ventana.moveTo(0, 0)
ventana.resizeTo(1280, 800)     # Posición y tamaño fijos → coordenadas reproducibles
clic_en_ventana(ventana, 150, 220)
```

### 11.2 Abrir una aplicación y esperar su ventana

```python
import subprocess, time

subprocess.Popen(["notepad.exe"])

def esperar_ventana(titulo: str, timeout: float = 10):
    limite = time.monotonic() + timeout
    while time.monotonic() < limite:
        ventanas = gw.getWindowsWithTitle(titulo)
        if ventanas:
            return ventanas[0]
        time.sleep(0.3)
    raise TimeoutError(f"No se abrió la ventana '{titulo}'")

ventana = esperar_ventana("Bloc de notas")
ventana.activate()
```

> En macOS y Linux, PyGetWindow tiene soporte limitado. Alternativas: abrir la aplicación con `subprocess` y usar atajos de teclado, o recurrir a herramientas del sistema (AppleScript, `wmctrl`).

---

## 12. Técnicas para automatizaciones robustas

| Problema | Técnica |
|---|---|
| La aplicación tarda en responder | Espere a un elemento visual (`esperar_imagen`) en lugar de usar `sleep` fijo. |
| La ventana cambia de posición | Fije posición y tamaño con PyGetWindow y use coordenadas relativas. |
| Coordenadas distintas en otros equipos | Use reconocimiento de imágenes o atajos de teclado. |
| Texto con caracteres especiales | Use el portapapeles (`pyperclip` + `ctrl+v`). |
| Campos con contenido previo | Seleccione todo (`ctrl+a`) antes de escribir. |
| Navegación entre campos | Prefiera `tab` a clics: no depende de coordenadas. |
| Errores intermitentes | Reintente acciones y registre capturas de pantalla al fallar. |
| Pantalla con escalado (DPI) | Use el escalado al 100 % o compruebe con `mouseInfo()`. |
| Procesos largos | Guarde el progreso (último registro procesado) para poder reanudar. |

### 12.1 Reintentos con captura de error

```python
from functools import wraps
from datetime import datetime


def reintentar(veces: int = 3, espera: float = 1.0):
    def decorador(func):
        @wraps(func)
        def envoltorio(*args, **kwargs):
            for intento in range(1, veces + 1):
                try:
                    return func(*args, **kwargs)
                except (pyautogui.ImageNotFoundException, TimeoutError) as e:
                    if intento == veces:
                        pyautogui.screenshot(f"error_{datetime.now():%H%M%S}.png")
                        raise
                    time.sleep(espera)
        return envoltorio
    return decorador


@reintentar(veces=3)
def pulsar_guardar():
    pyautogui.click(esperar_imagen("img/guardar.png", timeout=5))
```

### 12.2 Atajos de teclado frente a clics

Siempre que la aplicación lo permita, use atajos de teclado: son independientes de la posición y la resolución.

```python
pyautogui.hotkey("ctrl", "n")       # Nuevo registro
pyautogui.hotkey("ctrl", "s")       # Guardar
pyautogui.hotkey("alt", "a")        # Menú Archivo (letra subrayada)
```

---

## 13. Ejemplo práctico completo

Carga masiva de registros desde un CSV en una aplicación de escritorio: activa la ventana, recorre los campos con `tab`, escribe con el portapapeles, guarda, espera la confirmación, registra el resultado, toma capturas si hay errores y permite reanudar.

```python
"""
Carga de clientes desde CSV en una aplicación de escritorio.
Requisitos: pip install pyautogui pyperclip opencv-python
Imágenes de referencia en ./img: confirmacion.png (mensaje "Guardado correctamente")
"""
import csv
import json
import logging
import sys
import time
from datetime import datetime
from pathlib import Path

import pyautogui
import pyperclip

RAIZ = Path(__file__).resolve().parent
CSV_ENTRADA = RAIZ / "clientes.csv"
PROGRESO = RAIZ / "progreso.json"
CAPTURAS = RAIZ / "capturas_error"
IMG_CONFIRMACION = RAIZ / "img" / "confirmacion.png"
TITULO_VENTANA = "Gestión de Clientes"
CAMPOS = ["codigo", "nombre", "nif", "email", "telefono", "direccion"]
PEGAR = ("command", "v") if sys.platform == "darwin" else ("ctrl", "v")

pyautogui.FAILSAFE = True
pyautogui.PAUSE = 0.15

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-7s | %(message)s",
    handlers=[logging.FileHandler(RAIZ / "carga.log", encoding="utf-8"), logging.StreamHandler()],
)
log = logging.getLogger("carga")


def escribir(texto: str) -> None:
    """Sustituye el contenido del campo actual por el texto indicado."""
    pyautogui.hotkey("ctrl", "a")
    pyperclip.copy(texto or "")
    pyautogui.hotkey(*PEGAR)


def activar_ventana() -> None:
    try:
        import pygetwindow as gw
        ventanas = gw.getWindowsWithTitle(TITULO_VENTANA)
        if not ventanas:
            raise RuntimeError(f"No se encuentra la ventana '{TITULO_VENTANA}'")
        v = ventanas[0]
        if v.isMinimized:
            v.restore()
        v.activate()
        time.sleep(0.5)
    except (ImportError, NotImplementedError):
        pyautogui.alert(f"Active manualmente la ventana '{TITULO_VENTANA}' y pulse OK.")
        time.sleep(2)


def esperar_confirmacion(timeout: float = 10) -> bool:
    if not IMG_CONFIRMACION.exists():
        time.sleep(1.5)                     # Sin imagen de referencia: espera fija
        return True
    limite = time.monotonic() + timeout
    while time.monotonic() < limite:
        try:
            pyautogui.locateOnScreen(str(IMG_CONFIRMACION), confidence=0.85, grayscale=True)
            pyautogui.press("enter")        # Cerrar el aviso
            return True
        except pyautogui.ImageNotFoundException:
            time.sleep(0.4)
    return False


def cargar_progreso() -> int:
    if PROGRESO.exists():
        return json.loads(PROGRESO.read_text(encoding="utf-8")).get("ultima_fila", 0)
    return 0


def guardar_progreso(fila: int) -> None:
    PROGRESO.write_text(json.dumps({"ultima_fila": fila, "fecha": datetime.now().isoformat()}),
                        encoding="utf-8")


def alta_cliente(cliente: dict) -> bool:
    pyautogui.hotkey("ctrl", "n")           # Nuevo registro
    time.sleep(0.6)
    for i, campo in enumerate(CAMPOS):
        escribir(cliente.get(campo, "").strip())
        if i < len(CAMPOS) - 1:
            pyautogui.press("tab")
    pyautogui.hotkey("ctrl", "s")           # Guardar
    return esperar_confirmacion()


def main() -> None:
    with CSV_ENTRADA.open(encoding="utf-8-sig", newline="") as f:
        clientes = list(csv.DictReader(f, delimiter=";"))

    inicio = cargar_progreso()
    pendientes = clientes[inicio:]
    if not pendientes:
        pyautogui.alert("No hay registros pendientes.")
        return

    if pyautogui.confirm(
        f"Se cargarán {len(pendientes)} clientes (desde la fila {inicio + 1}).\n\n"
        "Durante el proceso no use el ratón ni el teclado.\n"
        "Para detenerlo, lleve el ratón a una esquina de la pantalla.",
        title="Carga de clientes", buttons=["Comenzar", "Cancelar"],
    ) != "Comenzar":
        return

    activar_ventana()
    CAPTURAS.mkdir(exist_ok=True)
    correctos = errores = 0

    try:
        for n, cliente in enumerate(pendientes, start=inicio + 1):
            if alta_cliente(cliente):
                correctos += 1
                log.info("Fila %d · %s · OK", n, cliente.get("codigo"))
            else:
                errores += 1
                captura = CAPTURAS / f"fila_{n}_{datetime.now():%H%M%S}.png"
                pyautogui.screenshot(str(captura))
                log.error("Fila %d · %s · sin confirmación (captura: %s)", n, cliente.get("codigo"), captura.name)
                pyautogui.press("esc")
            guardar_progreso(n)
    except pyautogui.FailSafeException:
        log.warning("Proceso detenido por el usuario (FAILSAFE)")

    resumen = f"Correctos: {correctos}\nErrores: {errores}\nÚltima fila procesada: {cargar_progreso()}"
    log.info(resumen.replace("\n", " · "))
    pyautogui.alert(resumen, title="Carga finalizada")


if __name__ == "__main__":
    main()
```

Formato del CSV (`clientes.csv`, separado por `;`):

```
codigo;nombre;nif;email;telefono;direccion
C-001;Comercial Norte, S.A.;A12345678;info@norte.es;944000000;Av. Libertad 10, Bilbao
C-002;Distribuciones Muñoz;B87654321;pedidos@munoz.es;954111111;C/ Feria 22, Sevilla
```

---

## 14. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `FailSafeException` inesperada | El ratón llegó a una esquina (también por coordenadas `(0, 0)`). | No haga clic en esquinas; capture la excepción. |
| Tildes y ñ no se escriben | `write()` solo admite caracteres ASCII. | Use `pyperclip` + `ctrl+v`. |
| `ImageNotFoundException` | La imagen no coincide (resolución, tema, escalado). | Recorte de nuevo la imagen, use `confidence` y `grayscale`. |
| `NotImplementedError: confidence keyword argument is only available if OpenCV is installed` | Falta OpenCV. | `pip install opencv-python`. |
| Los clics caen en un sitio distinto | Escalado de pantalla (DPI) o varios monitores. | Use escalado 100 % o verifique con `mouseInfo()`. |
| No hace nada en macOS | Faltan permisos de Accesibilidad. | Concédalos a la Terminal o al IDE y reinicie. |
| No funciona en Linux | Sesión Wayland. | Inicie sesión en X11 o use alternativas (`ydotool`). |
| Las acciones van más rápido que la aplicación | Falta de esperas. | Aumente `PAUSE` y espere a elementos visibles. |
| No interactúa con un programa de administrador (Windows) | Privilegios distintos. | Ejecute el script como administrador. |
| La pantalla se bloquea durante la ejecución | Suspensión o protector de pantalla. | Desactívelos durante el proceso. |

---

## 15. Buenas prácticas

- Mantenga **siempre** `FAILSAFE = True`.
- Antes de automatizar con PyAutoGUI, compruebe si existe una API, una librería o una opción de importación.
- Prefiera atajos de teclado y `tab` a clics por coordenadas.
- Espere a que los elementos aparezcan en lugar de usar pausas fijas largas.
- Use el portapapeles para texto con caracteres especiales.
- Fije la posición y el tamaño de la ventana de trabajo.
- Registre cada acción y haga capturas cuando algo falle.
- Guarde el progreso para poder reanudar tras una interrupción.
- Pruebe primero con pocos registros y supervise la ejecución.
- No deje el equipo desatendido ni con sesión abierta en entornos no seguros mientras se ejecuta.

---

## 16. Referencia rápida

```python
pyautogui.FAILSAFE = True; pyautogui.PAUSE = 0.3
pyautogui.size() / position() / onScreen(x, y) / mouseInfo()

moveTo(x, y, duration) / move(dx, dy) / click(x, y, clicks, button) / doubleClick / rightClick
mouseDown() / mouseUp() / dragTo(x, y, duration) / drag(dx, dy) / scroll(n)

write(texto, interval) / press(tecla | [teclas], presses) / hotkey("ctrl", "c")
keyDown(t) / keyUp(t) / with hold("shift"): ...

screenshot(ruta, region=(x, y, w, h)) / pixel(x, y) / pixelMatchesColor(x, y, rgb, tolerance)
locateOnScreen(img, confidence, grayscale, region) / locateCenterOnScreen / locateAllOnScreen / center
pyautogui.ImageNotFoundException / pyautogui.FailSafeException

alert(msg) / confirm(msg, buttons) / prompt(msg) / password(msg)

import pygetwindow as gw; gw.getWindowsWithTitle("t")[0].activate()
```

---

## 17. Ejercicios propuestos

1. Abra el Bloc de notas, escriba un texto con tildes y guárdelo con un nombre que incluya la fecha.
2. Cree un script que tome una captura de una región de la pantalla cada 5 minutos durante una hora.
3. Automatice el alta de 10 registros en una hoja de cálculo abierta usando solo el teclado.
4. Escriba una función que espere a que un botón cambie de color (píxel) antes de continuar.
5. Añada al ejemplo completo un modo «prueba» que muestre qué haría sin escribir nada.

---

## 18. Referencias

- Documentación oficial: https://pyautogui.readthedocs.io/
- Repositorio: https://github.com/asweigart/pyautogui
- PyGetWindow: https://github.com/asweigart/PyGetWindow
