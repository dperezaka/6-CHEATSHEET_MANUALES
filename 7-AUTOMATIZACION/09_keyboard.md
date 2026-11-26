# Manual de uso: Keyboard

> **Categoría:** Automatización y sistema · **Utilidad:** Automatizar y detectar pulsaciones de teclado
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y requisitos](#2-instalación-y-requisitos)
3. [Conceptos clave](#3-conceptos-clave)
4. [Simular pulsaciones](#4-simular-pulsaciones)
5. [Escribir texto](#5-escribir-texto)
6. [Atajos de teclado globales](#6-atajos-de-teclado-globales)
7. [Abreviaturas (expansión de texto)](#7-abreviaturas-expansión-de-texto)
8. [Detectar teclas](#8-detectar-teclas)
9. [Esperar teclas](#9-esperar-teclas)
10. [Bloquear y reasignar teclas](#10-bloquear-y-reasignar-teclas)
11. [Grabar y reproducir](#11-grabar-y-reproducir)
12. [Nombres de teclas](#12-nombres-de-teclas)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Uso responsable y legal](#14-uso-responsable-y-legal)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

**keyboard** permite **simular** pulsaciones de teclado y **detectar** las teclas que pulsa el usuario a nivel global, es decir, aunque la ventana activa sea otra aplicación.

**Casos de uso habituales:**
- **Atajos globales**: ejecutar funciones de Python con una combinación de teclas desde cualquier aplicación.
- **Expansión de texto**: escribir `@@firma` y que se sustituya por la firma completa.
- Automatizar secuencias de teclas.
- Reasignar o bloquear teclas.
- Atajos para detener o pausar otras automatizaciones (PyAutoGUI, bucles largos).

**Diferencias con PyAutoGUI:**

| | keyboard | PyAutoGUI |
|---|---|---|
| Simular teclado | ✔ | ✔ |
| Detectar teclas pulsadas / atajos globales | ✔ | ✘ |
| Escribir texto con tildes y ñ | ✔ (Windows) | ✘ |
| Ratón y pantalla | ✘ (ver librería `mouse`) | ✔ |
| Soporte macOS | Limitado / experimental | ✔ |

> El proyecto original `keyboard` no recibe actualizaciones frecuentes, aunque sigue siendo muy utilizado. Para proyectos nuevos multiplataforma, valore también **pynput**.

---

## 2. Instalación y requisitos

```bash
pip install keyboard
```

| Sistema | Requisitos |
|---|---|
| **Windows** | Ninguno. Para interactuar con aplicaciones ejecutadas como administrador, el script también debe ejecutarse como administrador. |
| **Linux** | Ejecutar como **root** (`sudo`), ya que accede directamente a `/dev/input`. |
| **macOS** | Soporte experimental; requiere permisos de Accesibilidad y suele necesitar `sudo`. Se recomienda `pynput`. |

```python
import keyboard
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Hook* (gancho) | Función que se ejecuta con cada evento de teclado del sistema. |
| *Hotkey* | Combinación de teclas que dispara una función (`ctrl+alt+p`). |
| *Hotstring* / abreviatura | Texto que se sustituye automáticamente al escribirlo. |
| Evento | Objeto `KeyboardEvent` con `name`, `scan_code`, `event_type` (`down`/`up`) y `time`. |
| *suppress* | Impide que la pulsación llegue a la aplicación activa (solo Windows de forma fiable). |
| Hilo de escucha | keyboard escucha en un hilo propio: el programa principal debe permanecer activo. |

---

## 4. Simular pulsaciones

### 4.1 Pulsar una tecla o combinación

```python
keyboard.send("enter")                 # Pulsar y soltar
keyboard.send("ctrl+c")                # Combinación
keyboard.send("ctrl+shift+esc")
keyboard.send("alt+tab")
keyboard.send("windows+d")             # Mostrar escritorio
```

`press_and_release` es equivalente a `send`.

### 4.2 Secuencias de combinaciones

```python
keyboard.send("ctrl+a, ctrl+c")        # Seleccionar todo y copiar
keyboard.send("alt, a, g")             # Menú Archivo → Guardar (según aplicación)
```

### 4.3 Mantener y soltar

```python
keyboard.press("shift")
for _ in range(5):
    keyboard.send("right")             # Selecciona 5 caracteres
keyboard.release("shift")
```

### 4.4 Repetir con pausas

```python
import time

for _ in range(10):
    keyboard.send("down")
    time.sleep(0.1)
```

---

## 5. Escribir texto

### 5.1 `write`

```python
keyboard.write("Informe mensual de ventas")
keyboard.write("Texto lento", delay=0.05)                       # Pausa entre caracteres
keyboard.write("Atentamente, José Núñez — 59,80 € ¿Alguna duda?")  # Admite Unicode
```

> En Windows, `write` escribe correctamente tildes, ñ y símbolos. En Linux depende de la distribución de teclado configurada; si falla, use el portapapeles (`pyperclip` + `ctrl+v`).

### 5.2 Escribir varias líneas

```python
keyboard.write("Primera línea\nSegunda línea\n")
```

> Algunas aplicaciones (chats, formularios web) envían el mensaje al recibir `Enter`. Use `shift+enter` para saltos de línea en ese caso:

```python
for i, linea in enumerate(texto.splitlines()):
    if i:
        keyboard.send("shift+enter")
    keyboard.write(linea)
```

---

## 6. Atajos de teclado globales

### 6.1 Registrar un atajo

```python
def saludar():
    print("¡Atajo pulsado!")

keyboard.add_hotkey("ctrl+alt+h", saludar)

print("Pulse Ctrl+Alt+H (Esc para salir)")
keyboard.wait("esc")                    # Mantiene el programa activo
```

### 6.2 Pasar argumentos

```python
keyboard.add_hotkey("ctrl+shift+f1", print, args=("Opción 1",))
keyboard.add_hotkey("ctrl+shift+f2", lambda: abrir_carpeta("C:/Informes"))
```

### 6.3 Opciones de `add_hotkey`

```python
keyboard.add_hotkey(
    "ctrl+shift+s",
    funcion,
    args=(),
    suppress=True,            # La combinación no llega a la aplicación activa (Windows)
    timeout=1,                # Tiempo máximo entre teclas en secuencias ("ctrl+k, ctrl+c")
    trigger_on_release=False, # Ejecutar al soltar en lugar de al pulsar
)
```

### 6.4 Secuencias de atajos (tipo editor de código)

```python
keyboard.add_hotkey("ctrl+k, ctrl+c", comentar)     # Pulsar Ctrl+K y después Ctrl+C
```

### 6.5 Eliminar atajos

```python
atajo = keyboard.add_hotkey("ctrl+alt+x", funcion)
keyboard.remove_hotkey(atajo)          # También acepta la cadena "ctrl+alt+x"
keyboard.unhook_all_hotkeys()
```

### 6.6 Funciones largas en atajos

Los atajos se ejecutan en el hilo de escucha. Si la función tarda, **bloquea** la detección de teclas. Ejecútela en un hilo aparte:

```python
import threading

def en_hilo(funcion):
    return lambda: threading.Thread(target=funcion, daemon=True).start()

keyboard.add_hotkey("ctrl+alt+r", en_hilo(generar_informe))
```

### 6.7 Atajo de emergencia para otros scripts

```python
import os

keyboard.add_hotkey("ctrl+alt+q", lambda: os._exit(0))    # Detiene el programa inmediatamente
```

Pausar y reanudar un proceso largo:

```python
import threading

pausado = threading.Event()

def alternar_pausa():
    if pausado.is_set():
        pausado.clear(); print("▶ Reanudado")
    else:
        pausado.set(); print("⏸ Pausado")

keyboard.add_hotkey("ctrl+alt+p", alternar_pausa)

for registro in registros:
    while pausado.is_set():
        time.sleep(0.2)
    procesar(registro)
```

---

## 7. Abreviaturas (expansión de texto)

### 7.1 `add_abbreviation`

Al escribir la abreviatura seguida de **espacio** (o Enter/Tab), se sustituye por el texto completo:

```python
keyboard.add_abbreviation("@@mail", "ana.lopez@miempresa.com")
keyboard.add_abbreviation("@@tel", "+34 600 123 456")
keyboard.add_abbreviation("@@dir", "C/ Mayor 1, 28013 Madrid")
keyboard.add_abbreviation("@@firma", "Un saludo,\nAna López\nDepartamento Comercial")

keyboard.wait()
```

> Use un prefijo poco habitual (`@@`, `;;`, `//`) para evitar sustituciones accidentales.

### 7.2 Abreviaturas dinámicas (fecha, hora)

`add_abbreviation` solo admite texto fijo. Para contenido dinámico, use `add_word_listener`:

```python
from datetime import datetime


def expandir(abreviatura: str, generar):
    def accion():
        for _ in range(len(abreviatura) + 1):          # Borrar abreviatura + espacio
            keyboard.send("backspace")
        keyboard.write(generar())
    keyboard.add_word_listener(abreviatura, accion, triggers=["space"], match_suffix=False)


expandir("@@hoy", lambda: datetime.now().strftime("%d/%m/%Y"))
expandir("@@hora", lambda: datetime.now().strftime("%H:%M"))
expandir("@@ts", lambda: datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
```

### 7.3 Eliminar

```python
keyboard.remove_abbreviation("@@mail")
keyboard.remove_word_listener("@@hoy")
```

---

## 8. Detectar teclas

### 8.1 Comprobar si una tecla está pulsada

```python
if keyboard.is_pressed("shift"):
    print("Shift pulsado")

if keyboard.is_pressed("ctrl+alt"):
    ...
```

Uso típico en bucles:

```python
while True:
    if keyboard.is_pressed("esc"):
        print("Detenido")
        break
    trabajar()
    time.sleep(0.05)
```

### 8.2 Reaccionar a cualquier pulsación

```python
def al_pulsar(evento):
    print(f"{evento.name!r:15} scan={evento.scan_code:4} {evento.event_type}")

keyboard.on_press(al_pulsar)
keyboard.wait("esc")
```

| Función | Escucha |
|---|---|
| `on_press(func)` | Todas las teclas al pulsar. |
| `on_release(func)` | Todas las teclas al soltar. |
| `on_press_key("a", func)` | Una tecla concreta al pulsar. |
| `on_release_key("a", func)` | Una tecla concreta al soltar. |
| `hook(func)` | Todos los eventos (pulsar y soltar). |
| `hook_key("a", func)` | Todos los eventos de una tecla. |

### 8.3 Leer la siguiente tecla

```python
evento = keyboard.read_event()          # Bloquea hasta el siguiente evento
tecla = keyboard.read_key()             # Nombre de la siguiente tecla pulsada
atajo = keyboard.read_hotkey()          # Combinación completa, p. ej. 'ctrl+shift+a'
print("Ha pulsado:", atajo)
```

Útil para que el usuario **configure** su propio atajo:

```python
print("Pulse la combinación que desea usar:")
atajo = keyboard.read_hotkey(suppress=False)
keyboard.add_hotkey(atajo, funcion)
print(f"Atajo «{atajo}» configurado")
```

### 8.4 Eliminar escuchas

```python
gancho = keyboard.on_press(al_pulsar)
keyboard.unhook(gancho)
keyboard.unhook_all()                   # Elimina todo: atajos, abreviaturas, ganchos
```

---

## 9. Esperar teclas

```python
keyboard.wait()                         # Bloquea indefinidamente (mantiene vivos los atajos)
keyboard.wait("esc")                    # Hasta que se pulse Esc
keyboard.wait("ctrl+alt+q")             # Hasta una combinación
```

Esperar a que el usuario esté listo:

```python
print("Sitúe el cursor en el primer campo y pulse F8 para comenzar...")
keyboard.wait("f8")
iniciar_automatizacion()
```

---

## 10. Bloquear y reasignar teclas

### 10.1 Bloquear una tecla

```python
keyboard.block_key("caps lock")          # Desactiva Bloq Mayús
keyboard.unblock_key("caps lock")
```

### 10.2 Reasignar teclas

```python
keyboard.remap_key("caps lock", "esc")          # Bloq Mayús funciona como Esc
keyboard.remap_hotkey("ctrl+alt+c", "ctrl+c")   # Una combinación produce otra
```

> El bloqueo y la reasignación solo funcionan mientras el script está en ejecución, y de forma fiable sobre todo en **Windows**.

### 10.3 Suprimir una combinación

```python
keyboard.add_hotkey("windows+l", lambda: print("Bloqueo interceptado"), suppress=True)
```

> Algunas combinaciones del sistema (`Ctrl+Alt+Supr`) no pueden interceptarse.

---

## 11. Grabar y reproducir

```python
print("Grabando. Pulse Esc para terminar.")
eventos = keyboard.record(until="esc")

print("Reproduciendo en 3 segundos...")
time.sleep(3)
keyboard.play(eventos, speed_factor=1.0)     # 2.0 = el doble de rápido
```

Grabación manual:

```python
keyboard.start_recording()
time.sleep(10)
eventos = keyboard.stop_recording()
```

Obtener el texto escrito en una grabación:

```python
for texto in keyboard.get_typed_strings(eventos):
    print(texto)
```

---

## 12. Nombres de teclas

| Grupo | Nombres |
|---|---|
| Modificadores | `ctrl`, `shift`, `alt`, `alt gr`, `windows` (`win`), `left ctrl`, `right shift`… |
| Edición | `enter`, `tab`, `space`, `backspace`, `delete`, `insert`, `esc` |
| Navegación | `up`, `down`, `left`, `right`, `home`, `end`, `page up`, `page down` |
| Función | `f1` … `f24` |
| Bloqueo | `caps lock`, `num lock`, `scroll lock` |
| Multimedia | `volume up`, `volume down`, `volume mute`, `play/pause media`, `next track` |
| Otras | `print screen`, `menu`, `pause` |
| Letras y números | `a` … `z`, `0` … `9` |
| Teclado numérico | Use el código de escaneo para distinguirlas. |

> Los nombres dependen de la distribución de teclado y del sistema. Para averiguar el nombre exacto de una tecla:

```python
print(keyboard.read_key())
```

En combinaciones, las teclas cuyo nombre contiene `+` o `,` se escriben como `plus` y `comma`.

---

## 13. Ejemplo práctico completo

**Asistente de productividad** que se ejecuta en segundo plano y ofrece:
- Abreviaturas fijas y dinámicas (fecha, hora).
- Atajos para transformar el texto seleccionado (mayúsculas, minúsculas, tipo título, quitar tildes).
- Atajo para insertar un texto desde un menú de plantillas.
- Atajo para mostrar la ayuda y otro para salir.
- Registro de uso.

```python
"""
Asistente de productividad con atajos globales.
Requisitos: pip install keyboard pyperclip
Windows: ejecutar normalmente. Linux: ejecutar con sudo.
"""
import logging
import threading
import time
import unicodedata
from datetime import datetime

import keyboard
import pyperclip

logging.basicConfig(level=logging.INFO, format="%(asctime)s | %(message)s", datefmt="%H:%M:%S")
log = logging.getLogger("asistente")

# ------------------------------------------------------------- Configuración
ABREVIATURAS = {
    ";;mail": "ana.lopez@miempresa.com",
    ";;tel": "+34 600 123 456",
    ";;dir": "C/ Mayor 1, 28013 Madrid",
    ";;cif": "B12345678",
    ";;iban": "ES12 3456 7890 1234 5678 9012",
}

ABREVIATURAS_DINAMICAS = {
    ";;hoy": lambda: datetime.now().strftime("%d/%m/%Y"),
    ";;hora": lambda: datetime.now().strftime("%H:%M"),
    ";;ts": lambda: datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
}

PLANTILLAS = {
    "1": ("Saludo formal", "Estimado/a cliente:\n\nGracias por ponerse en contacto con nosotros. "),
    "2": ("Despedida", "\n\nQuedamos a su disposición para cualquier consulta.\n\nUn cordial saludo,\nAna López"),
    "3": ("Recepción de pedido", "Hemos recibido su pedido correctamente y lo estamos preparando. "
                                 "Le informaremos en cuanto se realice el envío."),
    "4": ("Solicitud de documentación", "Para continuar con la gestión, le rogamos que nos envíe "
                                        "copia de su DNI y del último recibo."),
}

ATAJOS_TEXTO = {
    "ctrl+alt+u": ("MAYÚSCULAS", str.upper),
    "ctrl+alt+l": ("minúsculas", str.lower),
    "ctrl+alt+t": ("Tipo Título", str.title),
    "ctrl+alt+n": ("Quitar tildes",
                   lambda t: unicodedata.normalize("NFKD", t).encode("ascii", "ignore").decode()),
}

cerrojo = threading.Lock()                 # Evita ejecuciones simultáneas


# ------------------------------------------------------------- Utilidades
def pegar(texto: str) -> None:
    """Escribe un texto mediante el portapapeles, conservando el contenido original."""
    original = pyperclip.paste()
    pyperclip.copy(texto)
    time.sleep(0.05)
    keyboard.send("ctrl+v")
    time.sleep(0.15)
    pyperclip.copy(original)


def obtener_seleccion() -> str:
    original = pyperclip.paste()
    pyperclip.copy("")
    keyboard.send("ctrl+c")
    for _ in range(20):                    # Esperar hasta 1 s a que la aplicación copie
        time.sleep(0.05)
        texto = pyperclip.paste()
        if texto:
            return texto
    pyperclip.copy(original)
    return ""


def en_hilo(funcion):
    def lanzar(*args):
        def ejecutar():
            if not cerrojo.acquire(blocking=False):
                return
            try:
                time.sleep(0.1)            # Esperar a que se suelten las teclas del atajo
                funcion(*args)
            except Exception:
                log.exception("Error en %s", funcion.__name__)
            finally:
                cerrojo.release()
        threading.Thread(target=ejecutar, daemon=True).start()
    return lanzar


# ------------------------------------------------------------- Acciones
def transformar_seleccion(nombre: str, funcion) -> None:
    original = pyperclip.paste()
    texto = obtener_seleccion()
    if not texto:
        log.info("No hay texto seleccionado")
        return
    pyperclip.copy(funcion(texto))
    keyboard.send("ctrl+v")
    time.sleep(0.15)
    pyperclip.copy(original)
    log.info("Transformación aplicada: %s (%d caracteres)", nombre, len(texto))


def menu_plantillas() -> None:
    print("\n=== PLANTILLAS ===")
    for clave, (nombre, _) in PLANTILLAS.items():
        print(f"  {clave}. {nombre}")
    print("  Esc. Cancelar\nPulse un número...")
    while True:
        tecla = keyboard.read_key(suppress=False)
        if tecla == "esc":
            print("Cancelado")
            return
        if tecla in PLANTILLAS:
            keyboard.send("backspace")     # Borra el número escrito en la aplicación
            nombre, texto = PLANTILLAS[tecla]
            pegar(texto)
            log.info("Plantilla insertada: %s", nombre)
            return


def expandir_dinamica(abreviatura: str, generar) -> None:
    for _ in range(len(abreviatura) + 1):
        keyboard.send("backspace")
    keyboard.write(generar())
    log.info("Abreviatura dinámica: %s", abreviatura)


def mostrar_ayuda() -> None:
    print("\n" + "=" * 55)
    print(" ASISTENTE DE PRODUCTIVIDAD · ATAJOS DISPONIBLES")
    print("=" * 55)
    for atajo, (nombre, _) in ATAJOS_TEXTO.items():
        print(f"  {atajo:<14} {nombre} (texto seleccionado)")
    print(f"  {'ctrl+alt+p':<14} Menú de plantillas")
    print(f"  {'ctrl+alt+a':<14} Mostrar esta ayuda")
    print(f"  {'ctrl+alt+q':<14} Salir")
    print("\n Abreviaturas (escriba y pulse espacio):")
    for abrev in list(ABREVIATURAS) + list(ABREVIATURAS_DINAMICAS):
        print(f"  {abrev}")
    print("=" * 55)


# ------------------------------------------------------------- Registro
def registrar() -> None:
    for abrev, texto in ABREVIATURAS.items():
        keyboard.add_abbreviation(abrev, texto)

    for abrev, generar in ABREVIATURAS_DINAMICAS.items():
        keyboard.add_word_listener(abrev, lambda a=abrev, g=generar: expandir_dinamica(a, g),
                                   triggers=["space"], match_suffix=False)

    for atajo, (nombre, funcion) in ATAJOS_TEXTO.items():
        keyboard.add_hotkey(atajo, en_hilo(transformar_seleccion), args=(nombre, funcion))

    keyboard.add_hotkey("ctrl+alt+p", en_hilo(menu_plantillas))
    keyboard.add_hotkey("ctrl+alt+a", mostrar_ayuda)


def main() -> None:
    registrar()
    mostrar_ayuda()
    log.info("Asistente en ejecución. Pulse Ctrl+Alt+Q para salir.")
    keyboard.wait("ctrl+alt+q")
    keyboard.unhook_all()
    log.info("Asistente finalizado")


if __name__ == "__main__":
    main()
```

> Para que se inicie con Windows, cree un acceso directo a `pythonw.exe ruta\asistente.py` en la carpeta de inicio (`Win + R` → `shell:startup`).

---

## 14. Uso responsable y legal

La capacidad de registrar todas las pulsaciones del teclado puede utilizarse de forma indebida.

- **No registre** las pulsaciones de otras personas sin su conocimiento y consentimiento expreso: puede constituir un delito contra la intimidad y vulnerar la normativa de protección de datos (RGPD).
- No almacene en disco lo que escribe el usuario (podría incluir contraseñas y datos personales).
- Limite las escuchas a lo necesario: atajos y abreviaturas concretos, en lugar de `hook` sobre todas las teclas.
- Los antivirus pueden detectar como sospechosos los programas que capturan el teclado; es un comportamiento esperado.
- En entornos corporativos, consulte las políticas de seguridad antes de desplegar herramientas de este tipo.

---

## 15. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `ImportError: You must be root to use this library on linux` | Linux sin privilegios. | Ejecute con `sudo` (o use `pynput`). |
| El programa termina y los atajos dejan de funcionar | El hilo principal finaliza. | Use `keyboard.wait()` o un bucle. |
| Los atajos no funcionan en algunas aplicaciones | Aplicación ejecutada como administrador. | Ejecute el script como administrador. |
| Las teclas del atajo interfieren con la acción | El usuario aún mantiene pulsadas las teclas. | Añada una pausa breve antes de actuar o use `trigger_on_release=True`. |
| Se dejan de detectar teclas mientras se ejecuta una acción | Función larga en el hilo de escucha. | Ejecute la acción en otro hilo. |
| `ValueError: Key name ... is not mapped to any known key` | Nombre de tecla incorrecto o dependiente del idioma. | Averigüe el nombre con `keyboard.read_key()`. |
| Tildes incorrectas con `write` (Linux) | Distribución de teclado distinta. | Use el portapapeles. |
| La abreviatura no se expande | Falta el espacio final o se escribió en mayúsculas. | Termine con espacio; respete mayúsculas. |
| Teclas «pegadas» tras un error | Se interrumpió entre `press` y `release`. | Use `try/finally` para liberar, o `keyboard.release("ctrl")`. |
| No funciona en macOS | Soporte limitado. | Use `pynput`. |

---

## 16. Buenas prácticas

- Use combinaciones poco habituales (`ctrl+alt+…`) para evitar conflictos con otras aplicaciones.
- **Teclado español:** en Windows, `Ctrl+Alt` equivale a `AltGr`. Evite combinaciones que producen caracteres (`ctrl+alt+1` → `|`, `ctrl+alt+2` → `@`, `ctrl+alt+3` → `#`, `ctrl+alt+e` → `€`), o dejará de poder escribirlos. Prefiera letras sin uso con AltGr o `ctrl+shift+…`.
- Ejecute en hilos las acciones de los atajos que puedan tardar.
- Incluya siempre un atajo para **salir** y otro para **mostrar la ayuda**.
- Libere las teclas pulsadas manualmente en bloques `try/finally`.
- Use el portapapeles para insertar textos largos (es más rápido y fiable que `write`).
- Restaure el contenido del portapapeles tras usarlo.
- Llame a `keyboard.unhook_all()` al terminar.
- Respete la privacidad: no registre ni almacene pulsaciones de forma general.

---

## 17. Referencia rápida

```python
keyboard.send("ctrl+c") / press("shift") / release("shift")
keyboard.write("texto", delay=0.02)
keyboard.is_pressed("esc")

keyboard.add_hotkey("ctrl+alt+x", func, args=(), suppress=False, trigger_on_release=False)
keyboard.remove_hotkey(h) / unhook_all_hotkeys()
keyboard.add_abbreviation(";;mail", "texto") / remove_abbreviation(";;mail")
keyboard.add_word_listener("palabra", func, triggers=["space"], match_suffix=False)

keyboard.on_press(func) / on_release(func) / on_press_key("a", func) / hook(func)
keyboard.unhook(h) / unhook_all()
keyboard.read_key() / read_hotkey() / read_event()
keyboard.wait() / wait("esc")

keyboard.block_key("caps lock") / remap_key("caps lock", "esc") / remap_hotkey(a, b)
eventos = keyboard.record(until="esc"); keyboard.play(eventos, speed_factor=1)
```

---

## 18. Ejercicios propuestos

1. Cree un atajo que inserte la fecha y hora actuales en cualquier aplicación.
2. Desarrolle un expansor de texto que lea las abreviaturas desde un archivo JSON editable.
3. Añada a una automatización con PyAutoGUI atajos para pausar (`ctrl+alt+p`) y detener (`ctrl+alt+q`).
4. Cree un atajo que copie el texto seleccionado, lo busque en Google y abra el navegador.
5. Implemente un contador de pulsaciones por minuto que muestre solo estadísticas (sin almacenar las teclas).

---

## 19. Referencias

- Repositorio y documentación: https://github.com/boppreh/keyboard
- Alternativa multiplataforma (pynput): https://pynput.readthedocs.io/
