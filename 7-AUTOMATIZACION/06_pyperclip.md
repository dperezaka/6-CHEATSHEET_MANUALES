# Manual de uso: Pyperclip

> **Categoría:** Automatización y sistema · **Utilidad:** Gestionar el portapapeles
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y requisitos](#2-instalación-y-requisitos)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Transformar el contenido del portapapeles](#5-transformar-el-contenido-del-portapapeles)
6. [Trabajar con datos tabulares](#6-trabajar-con-datos-tabulares)
7. [Integración con PyAutoGUI](#7-integración-con-pyautogui)
8. [Vigilar el portapapeles](#8-vigilar-el-portapapeles)
9. [Más allá del texto: imágenes y archivos](#9-más-allá-del-texto-imágenes-y-archivos)
10. [Ejemplo práctico completo](#10-ejemplo-práctico-completo)
11. [Seguridad y privacidad](#11-seguridad-y-privacidad)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**Pyperclip** es una librería multiplataforma minimalista para **copiar texto al portapapeles** y **leer texto del portapapeles** desde Python. Funciona en Windows, macOS y Linux con la misma interfaz.

**Casos de uso habituales:**
- Copiar automáticamente resultados (contraseñas generadas, enlaces, códigos, informes).
- Transformar el texto copiado (limpiar formato, cambiar mayúsculas, extraer datos) y devolverlo al portapapeles.
- Escribir texto con tildes, ñ y símbolos en automatizaciones con PyAutoGUI.
- Pasar datos entre aplicaciones que no se comunican entre sí.
- Pegar tablas de Excel en Python y viceversa.

> Pyperclip solo gestiona **texto plano**. Para imágenes o archivos se necesitan otras herramientas (sección 9).

---

## 2. Instalación y requisitos

```bash
pip install pyperclip
```

| Sistema | Requisitos |
|---|---|
| Windows | Ninguno. |
| macOS | Ninguno (usa `pbcopy` / `pbpaste`). |
| Linux | Uno de estos: `xclip`, `xsel` (X11) o `wl-clipboard` (Wayland). |

Instalación en Linux (Debian/Ubuntu):

```bash
sudo apt install xclip           # X11
sudo apt install wl-clipboard    # Wayland
```

```python
import pyperclip
```

---

## 3. Conceptos clave

| Función | Descripción |
|---|---|
| `pyperclip.copy(texto)` | Copia texto al portapapeles (sustituye el contenido anterior). |
| `pyperclip.paste()` | Devuelve el texto del portapapeles (`""` si está vacío o no es texto). |
| `pyperclip.PyperclipException` | Excepción si no hay mecanismo de portapapeles disponible. |

> La API es intencionadamente mínima: solo `copy` y `paste`. Funciones como «esperar a que cambie el portapapeles» se implementan fácilmente con un bucle (sección 8).

---

## 4. Operaciones básicas

### 4.1 Copiar y pegar

```python
pyperclip.copy("Hola, mundo")
texto = pyperclip.paste()
print(texto)          # Hola, mundo
```

### 4.2 Copiar otros tipos de datos

`copy` solo acepta texto, números y booleanos (los convierte a texto). Convierta otros tipos explícitamente:

```python
pyperclip.copy(str(12345))
pyperclip.copy(", ".join(["Ana", "Luis", "Marta"]))

import json
pyperclip.copy(json.dumps({"id": 1, "activo": True}, indent=2, ensure_ascii=False))
```

### 4.3 Texto con varias líneas

```python
lineas = ["Pedido 1045 – Enviado", "Pedido 1046 – En preparación", "Pedido 1047 – Entregado"]
pyperclip.copy("\n".join(lineas))
```

> En Windows, algunas aplicaciones esperan saltos de línea `\r\n`. Si el texto pegado aparece en una sola línea, use `"\r\n".join(lineas)`.

### 4.4 Comprobar si hay texto

```python
contenido = pyperclip.paste()
if not contenido.strip():
    print("El portapapeles está vacío o no contiene texto")
```

### 4.5 Conservar y restaurar el contenido original

Cuando un script usa el portapapeles de forma auxiliar, conviene no perder lo que el usuario había copiado:

```python
from contextlib import contextmanager


@contextmanager
def portapapeles_preservado():
    original = pyperclip.paste()
    try:
        yield
    finally:
        pyperclip.copy(original)


with portapapeles_preservado():
    pyperclip.copy("texto temporal")
    # ... pegar en una aplicación ...
```

---

## 5. Transformar el contenido del portapapeles

El patrón más útil: **leer → transformar → copiar**. Se copia un texto, se ejecuta el script y se pega el resultado.

### 5.1 Limpiar texto

```python
import re

texto = pyperclip.paste()
limpio = re.sub(r"[ \t]+", " ", texto)          # Espacios múltiples
limpio = re.sub(r"\n{3,}", "\n\n", limpio)        # Líneas vacías repetidas
limpio = "\n".join(l.strip() for l in limpio.splitlines())
pyperclip.copy(limpio)
```

### 5.2 Unir líneas (texto copiado de un PDF)

```python
texto = pyperclip.paste()
parrafos = re.split(r"\n\s*\n", texto)
pyperclip.copy("\n\n".join(" ".join(p.split()) for p in parrafos))
```

### 5.3 Cambiar mayúsculas y minúsculas

```python
t = pyperclip.paste()
pyperclip.copy(t.upper())
pyperclip.copy(t.lower())
pyperclip.copy(t.title())
pyperclip.copy(t.capitalize())
```

### 5.4 Añadir viñetas o numeración

```python
lineas = [l for l in pyperclip.paste().splitlines() if l.strip()]
pyperclip.copy("\n".join(f"• {l.strip()}" for l in lineas))
pyperclip.copy("\n".join(f"{i}. {l.strip()}" for i, l in enumerate(lineas, 1)))
```

### 5.5 Extraer información

```python
texto = pyperclip.paste()

emails = sorted(set(re.findall(r"[\w.+-]+@[\w-]+\.[\w.-]+", texto)))
telefonos = re.findall(r"(?:\+34\s?)?[6789]\d{2}\s?\d{3}\s?\d{3}", texto)
urls = re.findall(r"https?://\S+", texto)

pyperclip.copy("\n".join(emails))
print(f"{len(emails)} correos copiados al portapapeles")
```

### 5.6 Convertir formatos

```python
# Lista de valores (uno por línea) → lista SQL
valores = [v.strip() for v in pyperclip.paste().splitlines() if v.strip()]
pyperclip.copy("(" + ", ".join(f"'{v}'" for v in valores) + ")")
# ('A001', 'A002', 'A003')

# Texto → slug para URL
import unicodedata
t = unicodedata.normalize("NFKD", pyperclip.paste()).encode("ascii", "ignore").decode()
pyperclip.copy(re.sub(r"[^a-z0-9]+", "-", t.lower()).strip("-"))
```

---

## 6. Trabajar con datos tabulares

Al copiar celdas de Excel, Google Sheets o LibreOffice, el portapapeles contiene texto con **columnas separadas por tabuladores** y **filas separadas por saltos de línea**.

### 6.1 Leer una tabla copiada de Excel

```python
texto = pyperclip.paste()
filas = [linea.split("\t") for linea in texto.rstrip("\r\n").splitlines()]
cabecera, datos = filas[0], filas[1:]
```

### 6.2 Con Pandas

```python
import pandas as pd
from io import StringIO

df = pd.read_csv(StringIO(pyperclip.paste()), sep="\t", decimal=",", thousands=".")
print(df.describe())
```

Pandas incluye funciones directas:

```python
df = pd.read_clipboard(sep="\t", decimal=",")     # Leer
df.to_clipboard(index=False, sep="\t", decimal=",")  # Copiar (se puede pegar en Excel)
```

### 6.3 Copiar resultados para pegarlos en Excel

```python
filas = [["Región", "Ventas"], ["Norte", "13800,00"], ["Sur", "11200,00"]]
pyperclip.copy("\n".join("\t".join(f) for f in filas))
# Al pegar en Excel, cada valor ocupa su propia celda
```

### 6.4 Tabla copiada → Markdown

```python
filas = [l.split("\t") for l in pyperclip.paste().strip().splitlines()]
md = ["| " + " | ".join(filas[0]) + " |", "|" + "---|" * len(filas[0])]
md += ["| " + " | ".join(f) + " |" for f in filas[1:]]
pyperclip.copy("\n".join(md))
```

---

## 7. Integración con PyAutoGUI

`pyautogui.write()` no escribe correctamente tildes, ñ ni muchos símbolos. La solución estándar es copiar el texto y pegarlo.

### 7.1 Escribir cualquier texto

```python
import sys
import pyautogui
import pyperclip

PEGAR = ("command", "v") if sys.platform == "darwin" else ("ctrl", "v")
COPIAR = ("command", "c") if sys.platform == "darwin" else ("ctrl", "c")


def escribir(texto: str) -> None:
    pyperclip.copy(texto)
    pyautogui.hotkey(*PEGAR)

escribir("Atentamente, José Núñez — Dirección Comercial (2026) €")
```

### 7.2 Leer el contenido de un campo de una aplicación

```python
import time

def leer_campo() -> str:
    pyperclip.copy("")                  # Vaciar para detectar si la copia funcionó
    pyautogui.hotkey("ctrl", "a")
    pyautogui.hotkey(*COPIAR)
    time.sleep(0.2)                     # Dar tiempo a la aplicación
    return pyperclip.paste()

valor = leer_campo()
```

### 7.3 Copiar texto seleccionado de cualquier aplicación y procesarlo

```python
def procesar_seleccion(funcion) -> None:
    """Copia la selección actual, la transforma y la pega en su lugar."""
    anterior = pyperclip.paste()
    pyperclip.copy("")
    pyautogui.hotkey(*COPIAR)
    time.sleep(0.2)
    seleccion = pyperclip.paste()
    if seleccion:
        pyperclip.copy(funcion(seleccion))
        pyautogui.hotkey(*PEGAR)
        time.sleep(0.2)
    pyperclip.copy(anterior)

procesar_seleccion(str.upper)
```

> Combinado con la librería **keyboard**, este patrón permite asignar transformaciones a atajos globales (por ejemplo, `Ctrl+Alt+U` → convertir la selección a mayúsculas).

---

## 8. Vigilar el portapapeles

### 8.1 Esperar a que el usuario copie algo

```python
import time


def esperar_nueva_copia(timeout: float | None = None, intervalo: float = 0.3) -> str:
    """Bloquea hasta que el contenido del portapapeles cambie y lo devuelve."""
    inicial = pyperclip.paste()
    limite = None if timeout is None else time.monotonic() + timeout
    while limite is None or time.monotonic() < limite:
        actual = pyperclip.paste()
        if actual != inicial:
            return actual
        time.sleep(intervalo)
    raise TimeoutError(f"No se ha copiado nada en {timeout} s")


print("Copie un texto (Ctrl+C)...")
try:
    nuevo = esperar_nueva_copia(timeout=30)
    print("Ha copiado:", nuevo)
except TimeoutError:
    print("No se ha copiado nada en 30 segundos")
```

### 8.2 Historial del portapapeles

```python
import time
from datetime import datetime

historial: list[tuple[str, str]] = []
anterior = pyperclip.paste()

print("Registrando el portapapeles. Pulse Ctrl+C en la consola para terminar.")
try:
    while True:
        actual = pyperclip.paste()
        if actual and actual != anterior:
            historial.append((datetime.now().strftime("%H:%M:%S"), actual))
            print(f"[{historial[-1][0]}] {actual[:60]!r}")
            anterior = actual
        time.sleep(0.5)
except KeyboardInterrupt:
    print(f"\n{len(historial)} elementos registrados")
```

### 8.3 Reaccionar automáticamente al contenido copiado

```python
import re
import webbrowser

PATRON_PEDIDO = re.compile(r"^PED-\d{6}$")

anterior = ""
while True:
    texto = pyperclip.paste().strip()
    if texto != anterior:
        anterior = texto
        if PATRON_PEDIDO.match(texto):
            webbrowser.open(f"https://intranet.miempresa.com/pedidos/{texto}")
    time.sleep(0.5)
```

---

## 9. Más allá del texto: imágenes y archivos

Pyperclip no gestiona imágenes ni archivos. Alternativas:

### 9.1 Leer una imagen del portapapeles (Windows y macOS)

```python
from PIL import ImageGrab, Image

contenido = ImageGrab.grabclipboard()
if isinstance(contenido, Image.Image):
    contenido.save("captura_portapapeles.png")
elif isinstance(contenido, list):
    print("Archivos copiados:", contenido)      # Rutas de archivos copiados en el explorador
else:
    print("No hay imagen en el portapapeles")
```

### 9.2 Copiar una imagen al portapapeles (Windows)

```python
# pip install pywin32 pillow
import io
import win32clipboard
from PIL import Image

def copiar_imagen(ruta: str) -> None:
    imagen = Image.open(ruta).convert("RGB")
    salida = io.BytesIO()
    imagen.save(salida, "BMP")
    datos = salida.getvalue()[14:]           # Quitar la cabecera BMP
    win32clipboard.OpenClipboard()
    try:
        win32clipboard.EmptyClipboard()
        win32clipboard.SetClipboardData(win32clipboard.CF_DIB, datos)
    finally:
        win32clipboard.CloseClipboard()
```

### 9.3 Texto con formato (HTML) en Windows

Para copiar texto con negritas, enlaces o tablas que se peguen con formato en Word u Outlook, use `win32clipboard` con el formato `HTML Format` (consulte el manual de **PyWin32**).

---

## 10. Ejemplo práctico completo

**Asistente de portapapeles**: un menú en consola que aplica transformaciones habituales al texto copiado. El usuario copia un texto en cualquier aplicación, elige una opción y pega el resultado.

```python
"""
Asistente de portapapeles.
Uso: copie un texto, elija una transformación y pegue el resultado (Ctrl+V).
"""
import json
import re
import unicodedata
from collections.abc import Callable
from datetime import datetime

import pyperclip


# ---------- Transformaciones ----------
def limpiar_espacios(t: str) -> str:
    lineas = [" ".join(l.split()) for l in t.splitlines()]
    return re.sub(r"\n{3,}", "\n\n", "\n".join(lineas)).strip()


def unir_lineas(t: str) -> str:
    parrafos = re.split(r"\n\s*\n", t)
    return "\n\n".join(" ".join(p.split()) for p in parrafos if p.strip())


def sin_tildes(t: str) -> str:
    return unicodedata.normalize("NFKD", t).encode("ascii", "ignore").decode()


def slug(t: str) -> str:
    return re.sub(r"[^a-z0-9]+", "-", sin_tildes(t).lower()).strip("-")


def extraer_emails(t: str) -> str:
    return "\n".join(sorted(set(re.findall(r"[\w.+-]+@[\w-]+\.[\w.-]+", t), key=str.lower)))


def extraer_urls(t: str) -> str:
    return "\n".join(dict.fromkeys(re.findall(r"https?://[^\s<>\"')]+", t)))


def lista_sql(t: str) -> str:
    valores = [v.strip().replace("'", "''") for v in t.splitlines() if v.strip()]
    return "(" + ", ".join(f"'{v}'" for v in valores) + ")"


def tabla_a_markdown(t: str) -> str:
    filas = [l.split("\t") for l in t.strip().splitlines()]
    if not filas:
        return t
    n = max(len(f) for f in filas)
    filas = [f + [""] * (n - len(f)) for f in filas]
    salida = ["| " + " | ".join(filas[0]) + " |", "|" + " --- |" * n]
    salida += ["| " + " | ".join(f) + " |" for f in filas[1:]]
    return "\n".join(salida)


def formatear_json(t: str) -> str:
    return json.dumps(json.loads(t), indent=2, ensure_ascii=False)


def numerar(t: str) -> str:
    lineas = [l.strip() for l in t.splitlines() if l.strip()]
    return "\n".join(f"{i}. {l}" for i, l in enumerate(lineas, 1))


def eliminar_duplicados(t: str) -> str:
    return "\n".join(dict.fromkeys(l for l in t.splitlines() if l.strip()))


def ordenar(t: str) -> str:
    return "\n".join(sorted((l for l in t.splitlines() if l.strip()), key=lambda s: sin_tildes(s).lower()))


OPCIONES: dict[str, tuple[str, Callable[[str], str]]] = {
    "1": ("Limpiar espacios y líneas vacías", limpiar_espacios),
    "2": ("Unir líneas (texto copiado de PDF)", unir_lineas),
    "3": ("MAYÚSCULAS", str.upper),
    "4": ("minúsculas", str.lower),
    "5": ("Tipo Título", str.title),
    "6": ("Quitar tildes", sin_tildes),
    "7": ("Convertir a slug-para-url", slug),
    "8": ("Extraer correos electrónicos", extraer_emails),
    "9": ("Extraer URLs", extraer_urls),
    "10": ("Lista → ('a', 'b', 'c') para SQL", lista_sql),
    "11": ("Tabla de Excel → Markdown", tabla_a_markdown),
    "12": ("Formatear JSON", formatear_json),
    "13": ("Numerar líneas", numerar),
    "14": ("Eliminar líneas duplicadas", eliminar_duplicados),
    "15": ("Ordenar líneas alfabéticamente", ordenar),
}


def vista_previa(t: str, max_lineas: int = 4) -> str:
    lineas = t.splitlines() or [""]
    texto = "\n   ".join(l[:70] for l in lineas[:max_lineas])
    if len(lineas) > max_lineas:
        texto += f"\n   … ({len(lineas)} líneas en total)"
    return texto


def main() -> None:
    historial: list[str] = []
    while True:
        actual = pyperclip.paste()
        print("\n" + "=" * 60)
        print(f" ASISTENTE DE PORTAPAPELES · {datetime.now():%H:%M:%S}")
        print("=" * 60)
        print(f" Contenido actual ({len(actual)} caracteres):\n   {vista_previa(actual)}")
        print("-" * 60)
        for clave, (descripcion, _) in OPCIONES.items():
            print(f" {clave:>2}. {descripcion}")
        print("  d. Deshacer última transformación")
        print("  r. Releer portapapeles")
        print("  q. Salir")
        eleccion = input("\n Opción: ").strip().lower()

        if eleccion == "q":
            break
        if eleccion == "r":
            continue
        if eleccion == "d":
            if historial:
                pyperclip.copy(historial.pop())
                print(" ↩ Restaurado el contenido anterior")
            else:
                print(" No hay nada que deshacer")
            continue
        if eleccion not in OPCIONES:
            print(" Opción no válida")
            continue
        if not actual.strip():
            print(" ⚠ El portapapeles no contiene texto")
            continue

        descripcion, funcion = OPCIONES[eleccion]
        try:
            resultado = funcion(actual)
        except (ValueError, json.JSONDecodeError) as e:
            print(f" ❌ No se pudo aplicar «{descripcion}»: {e}")
            continue

        historial.append(actual)
        pyperclip.copy(resultado)
        print(f" ✅ «{descripcion}» aplicado. Ya puede pegar el resultado (Ctrl+V).")
        print(f"   {vista_previa(resultado)}")


if __name__ == "__main__":
    try:
        pyperclip.paste()
    except pyperclip.PyperclipException as e:
        raise SystemExit(f"No hay acceso al portapapeles: {e}")
    main()
```

---

## 11. Seguridad y privacidad

- El portapapeles es **compartido** por todas las aplicaciones: cualquier programa puede leerlo.
- No deje contraseñas o datos sensibles en el portapapeles más tiempo del necesario. Si su script copia una contraseña, vacíelo después:

```python
import threading

def copiar_temporal(texto: str, segundos: int = 30) -> None:
    pyperclip.copy(texto)
    def limpiar():
        if pyperclip.paste() == texto:     # Solo si el usuario no ha copiado otra cosa
            pyperclip.copy("")
    threading.Timer(segundos, limpiar).start()

copiar_temporal("Cl@ve-Generada-2026", segundos=20)
```

- Windows (`Win + V`) y otros sistemas pueden guardar un **historial del portapapeles** y sincronizarlo entre dispositivos; tenga en cuenta que los datos copiados pueden persistir.
- Los historiales registrados por scripts (sección 8.2) pueden contener información confidencial: no los guarde en disco sin necesidad.

---

## 12. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `PyperclipException: Pyperclip could not find a copy/paste mechanism for your system` | Linux sin `xclip`, `xsel` ni `wl-clipboard`. | Instale uno de ellos. |
| `paste()` devuelve `""` | El portapapeles contiene una imagen o archivos, o está vacío. | Compruebe el contenido; use `ImageGrab` para imágenes. |
| `TypeError` en `copy()` | Se pasó una lista, diccionario u otro objeto. | Convierta a texto (`str`, `join`, `json.dumps`). |
| El texto pegado aparece en una línea (Windows) | Saltos de línea `\n` en aplicaciones antiguas. | Use `\r\n`. |
| Al pegar con PyAutoGUI aparece el texto anterior | La aplicación no ha procesado aún la copia. | Añada una pequeña pausa (`time.sleep(0.1–0.2)`). |
| La lectura con `Ctrl+C` devuelve el contenido viejo | La aplicación tarda en copiar. | Vacíe el portapapeles antes y espere a que cambie. |
| No funciona por SSH o en servidores | No hay entorno gráfico ni portapapeles. | Pyperclip requiere sesión de escritorio. |
| `PyperclipWindowsException` ocasional | Otra aplicación tiene el portapapeles bloqueado. | Reintente tras una breve pausa. |

---

## 13. Buenas prácticas

- Use el patrón **leer → transformar → copiar** para crear pequeñas utilidades muy productivas.
- Restaure el contenido original cuando use el portapapeles como medio auxiliar.
- Añada pausas breves al combinar con PyAutoGUI.
- Convierta los datos a texto explícitamente antes de copiarlos.
- Use tabuladores y saltos de línea para intercambiar tablas con hojas de cálculo.
- No mantenga datos sensibles en el portapapeles; límpielo tras usarlo.
- Para tablas, valore `pandas.read_clipboard()` y `DataFrame.to_clipboard()`.

---

## 14. Referencia rápida

```python
import pyperclip

pyperclip.copy("texto")                       # Copiar
pyperclip.paste()                             # Pegar (leer)
pyperclip.PyperclipException                  # Sin portapapeles disponible
esperar_nueva_copia(timeout=30)               # Función propia (sección 8.1)

# Tablas
"\n".join("\t".join(fila) for fila in filas)  # Para pegar en Excel
pd.read_clipboard(sep="\t") / df.to_clipboard(index=False)

# Imágenes
from PIL import ImageGrab; ImageGrab.grabclipboard()
```

---

## 15. Ejercicios propuestos

1. Cree un script que convierta una lista de nombres copiada en direcciones de correo `nombre.apellido@empresa.com`.
2. Escriba una utilidad que, al copiar una tabla de Excel, calcule la suma de cada columna numérica y copie el resultado.
3. Implemente un generador de contraseñas que copie la contraseña al portapapeles y la borre a los 20 segundos.
4. Registre durante una sesión todas las URLs copiadas y guárdelas en un archivo al terminar.
5. Combine Pyperclip con la librería **keyboard** para convertir el texto seleccionado a mayúsculas con `Ctrl+Alt+U`.

---

## 16. Referencias

- Documentación oficial: https://pyperclip.readthedocs.io/
- Repositorio: https://github.com/asweigart/pyperclip
