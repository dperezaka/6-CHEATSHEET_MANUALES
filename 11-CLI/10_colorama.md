# Manual de uso: colorama

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Colores ANSI compatibles en Windows, macOS y Linux
> **Tipo:** Librería de terceros (`pip install colorama`) · **Versión de referencia:** 0.4.6

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Inicialización](#4-inicialización)
5. [Colores de texto y de fondo](#5-colores-de-texto-y-de-fondo)
6. [Estilos y restablecimiento](#6-estilos-y-restablecimiento)
7. [Movimiento del cursor y borrado](#7-movimiento-del-cursor-y-borrado)
8. [Funciones auxiliares de uso habitual](#8-funciones-auxiliares-de-uso-habitual)
9. [Logging con colores](#9-logging-con-colores)
10. [Desactivar el color: redirección, `NO_COLOR` y opciones](#10-desactivar-el-color-redirección-no_color-y-opciones)
11. [Colores de 256 y 24 bits](#11-colores-de-256-y-24-bits)
12. [Pruebas](#12-pruebas)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**colorama** hace que las **secuencias de escape ANSI** (los códigos que cambian color y estilo del texto en la terminal) funcionen de forma uniforme en Windows, macOS y Linux. En Unix no hace nada especial, porque las terminales ya entienden ANSI; en Windows, activa el modo de procesamiento ANSI de la consola o, en versiones antiguas, traduce las secuencias a llamadas de la API de Windows. Además, ofrece **constantes legibles** (`Fore.RED`, `Back.WHITE`, `Style.BRIGHT`) para no escribir los códigos a mano.

**Ventajas:**
- Mínima y sin dependencias.
- Compatibilidad con la consola clásica de Windows (cmd.exe, PowerShell 5).
- Constantes simples para colores, fondos, estilos y cursor.
- Elimina automáticamente los códigos de color cuando la salida se redirige a un archivo (en Windows).
- Muy extendida: muchas librerías la usan internamente.

**Limitaciones:**
- Solo 16 colores mediante constantes (los de 256 o 24 bits se escriben a mano).
- No ofrece tablas, paneles, barras de progreso ni marcado.
- Desarrollo poco activo: la versión 0.4.6 es estable desde 2022.
- En terminales modernas (Windows Terminal, VS Code, macOS, Linux) la compatibilidad ya existe sin colorama; su utilidad principal son las constantes y la consola clásica de Windows.

| Si necesita… | Opción |
|---|---|
| Solo colores simples y compatibilidad con Windows antiguo | **colorama** |
| Salida rica (tablas, paneles, marcado, progreso) | Rich |
| Colores dentro de una CLI de Click | `click.style` / `click.secho` |
| Barras de progreso | tqdm |

---

## 2. Instalación e importación

```bash
pip install colorama
```

```python
from colorama import Fore, Back, Style, Cursor, just_fix_windows_console
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Secuencia ANSI | Texto especial (`\x1b[31m`) que la terminal interpreta como «cambia a rojo». |
| `Fore` | Constantes de color del texto. |
| `Back` | Constantes de color de fondo. |
| `Style` | Intensidad (`BRIGHT`, `DIM`, `NORMAL`) y `RESET_ALL`. |
| `Cursor` | Secuencias para mover el cursor. |
| `just_fix_windows_console()` | Activa ANSI en Windows sin alterar el resto de plataformas (recomendado). |
| `init()` | Método clásico: envuelve `sys.stdout`/`sys.stderr` para convertir o eliminar ANSI. |
| `autoreset` | Restablece el estilo automáticamente después de cada `print`. |

Las constantes son simplemente **textos**:

```python
>>> from colorama import Fore
>>> Fore.RED
'\x1b[31m'
```

Por eso se concatenan con el texto: `print(Fore.RED + "Error" + Style.RESET_ALL)`.

---

## 4. Inicialización

### 4.1 Recomendado: `just_fix_windows_console()` (0.4.6+)

```python
from colorama import just_fix_windows_console, Fore, Style

just_fix_windows_console()          # Seguro de llamar en todas las plataformas
print(Fore.GREEN + "Correcto" + Style.RESET_ALL)
```

- En Windows 10/11 activa el procesamiento ANSI nativo de la consola.
- En consolas Windows antiguas, envuelve la salida para traducir los códigos.
- En macOS/Linux, y si la salida no es una terminal, no hace nada.
- Se puede llamar varias veces sin efectos secundarios.

### 4.2 Método clásico: `init()`

```python
from colorama import init, deinit, reinit

init()                       # Envuelve stdout y stderr
init(autoreset=True)         # Además, añade RESET_ALL tras cada escritura
init(strip=True)             # Elimina siempre los códigos ANSI
init(convert=False)          # No convertir a llamadas de la API de Windows
deinit()                     # Restaura los flujos originales
reinit()                     # Vuelve a aplicar la configuración de init()
```

| Parámetro de `init` | Efecto |
|---|---|
| `autoreset=False` | Si es `True`, restablece estilos al final de cada `print`. |
| `strip=None` | Elimina los códigos ANSI. `None`: automático (en Windows, si no es terminal). |
| `convert=None` | Traduce ANSI a llamadas Win32. `None`: automático. |
| `wrap=True` | Si es `False`, no envuelve `sys.stdout`/`sys.stderr`. |

> `init()` sustituye `sys.stdout` y `sys.stderr` por envoltorios. Esto puede interferir con otras librerías o con pruebas; por eso, en código nuevo se recomienda `just_fix_windows_console()`.

### 4.3 Uso temporal con gestor de contexto

```python
from colorama import colorama_text

with colorama_text():        # init() al entrar, deinit() al salir
    print(Fore.CYAN + "Solo aquí se inicializa" + Style.RESET_ALL)
```

---

## 5. Colores de texto y de fondo

### 5.1 Constantes disponibles

| `Fore` / `Back` | Variante clara (`*_EX`) |
|---|---|
| `BLACK` | `LIGHTBLACK_EX` (gris) |
| `RED` | `LIGHTRED_EX` |
| `GREEN` | `LIGHTGREEN_EX` |
| `YELLOW` | `LIGHTYELLOW_EX` |
| `BLUE` | `LIGHTBLUE_EX` |
| `MAGENTA` | `LIGHTMAGENTA_EX` |
| `CYAN` | `LIGHTCYAN_EX` |
| `WHITE` | `LIGHTWHITE_EX` |
| `RESET` | — (vuelve al color por defecto) |

### 5.2 Ejemplos

```python
from colorama import Fore, Back, Style

print(Fore.RED + "Texto rojo")
print(Back.GREEN + "Fondo verde")
print(Fore.WHITE + Back.RED + Style.BRIGHT + " ERROR " + Style.RESET_ALL + " No se pudo conectar")
print(Fore.LIGHTBLACK_EX + "Texto gris (secundario)" + Fore.RESET)
print(f"{Fore.CYAN}Usuario:{Fore.RESET} ana   {Fore.CYAN}Rol:{Fore.RESET} admin")
```

> El aspecto exacto de cada color depende del tema de la terminal: `YELLOW` puede verse marrón u ocre en algunas consolas, y los colores `LIGHT*` pueden no distinguirse de los normales.

---

## 6. Estilos y restablecimiento

| Constante | Efecto |
|---|---|
| `Style.BRIGHT` | Texto brillante / negrita (según la terminal). |
| `Style.DIM` | Texto tenue (no admitido en la consola clásica de Windows). |
| `Style.NORMAL` | Intensidad normal. |
| `Style.RESET_ALL` | Restablece color, fondo y estilo. |
| `Fore.RESET` / `Back.RESET` | Restablece solo el color o solo el fondo. |

```python
print(Style.BRIGHT + "Título" + Style.NORMAL + " texto normal")
print(Style.DIM + "nota al pie" + Style.RESET_ALL)
```

**Regla fundamental:** termine siempre con `Style.RESET_ALL` (o use `autoreset=True`). Si no, el color se mantiene en las líneas siguientes e incluso en el *prompt* de la terminal tras salir del programa.

```python
try:
    print(Fore.YELLOW + "Procesando…", end="")
    procesar()
finally:
    print(Style.RESET_ALL)
```

> colorama **no** ofrece cursiva ni subrayado como constantes. Pueden escribirse con su código ANSI (`"\x1b[3m"` cursiva, `"\x1b[4m"` subrayado) en terminales que los admitan.

---

## 7. Movimiento del cursor y borrado

```python
from colorama import Cursor, ansi

print(Cursor.UP(2) + "Sobrescribe dos líneas más arriba")
print(Cursor.FORWARD(10) + "Desplazado 10 columnas")
print(Cursor.POS(5, 3) + "En la columna 5, fila 3")       # Posición absoluta (x, y), desde 1

print(ansi.clear_screen())             # Borra la pantalla (equivale a '\x1b[2J')
print(ansi.clear_line(), end="\r")     # Borra la línea actual
```

| Secuencia | Efecto |
|---|---|
| `Cursor.UP(n)`, `DOWN(n)`, `FORWARD(n)`, `BACK(n)` | Mueve el cursor `n` posiciones. |
| `Cursor.POS(x, y)` | Posición absoluta. |
| `ansi.clear_screen(modo=2)` | Borra la pantalla (0: hasta el final, 1: hasta el inicio, 2: toda). |
| `ansi.clear_line(modo=2)` | Borra la línea. |

Actualizar una línea en el sitio (contador simple):

```python
import time
for i in range(101):
    print(f"\r{Fore.CYAN}Progreso:{Fore.RESET} {i:3d} %", end="", flush=True)
    time.sleep(0.02)
print()
```

---

## 8. Funciones auxiliares de uso habitual

Encapsular los colores en funciones mantiene el código limpio y coherente:

```python
from colorama import Fore, Back, Style, just_fix_windows_console

just_fix_windows_console()

def color(texto: str, fore: str = "", back: str = "", estilo: str = "") -> str:
    return f"{estilo}{fore}{back}{texto}{Style.RESET_ALL}"

def ok(msg: str) -> None:
    print(color("✔ ", Fore.GREEN, estilo=Style.BRIGHT) + msg)

def aviso(msg: str) -> None:
    print(color("⚠ ", Fore.YELLOW, estilo=Style.BRIGHT) + msg)

def error(msg: str) -> None:
    import sys
    print(color(" ERROR ", Fore.WHITE, Back.RED, Style.BRIGHT) + " " + msg, file=sys.stderr)

def titulo(texto: str) -> None:
    print("\n" + color(texto.upper(), Fore.CYAN, estilo=Style.BRIGHT))
    print(color("─" * len(texto), Fore.CYAN))

def resaltar(texto: str, palabra: str) -> str:
    return texto.replace(palabra, color(palabra, Fore.BLACK, Back.YELLOW))
```

```python
titulo("Comprobaciones")
ok("Conexión con la base de datos")
aviso("Espacio en disco por debajo del 15 %")
error("No se encontró config.toml")
print(resaltar("El servidor srv-01 no responde", "srv-01"))
```

Tabla sencilla con estados de color (cuidado con el ancho: los códigos ANSI no ocupan espacio visible):

```python
import re
ANSI = re.compile(r"\x1b\[[0-9;]*m")

def ancho_visible(texto: str) -> int:
    return len(ANSI.sub("", texto))

def rellenar(texto: str, ancho: int) -> str:
    return texto + " " * (ancho - ancho_visible(texto))

filas = [("web", color("OK", Fore.GREEN)), ("api", color("LENTO", Fore.YELLOW)), ("db", color("CAÍDO", Fore.RED))]
for servicio, estado in filas:
    print(rellenar(servicio, 10) + estado)
```

---

## 9. Logging con colores

```python
import logging
from colorama import Fore, Style, just_fix_windows_console

just_fix_windows_console()

class FormatoColor(logging.Formatter):
    COLORES = {
        logging.DEBUG: Fore.LIGHTBLACK_EX,
        logging.INFO: Fore.CYAN,
        logging.WARNING: Fore.YELLOW,
        logging.ERROR: Fore.RED,
        logging.CRITICAL: Fore.WHITE + "\x1b[41m" + Style.BRIGHT,     # Fondo rojo
    }

    def format(self, registro: logging.LogRecord) -> str:
        texto = super().format(registro)
        return f"{self.COLORES.get(registro.levelno, '')}{texto}{Style.RESET_ALL}"

manejador = logging.StreamHandler()
manejador.setFormatter(FormatoColor("%(asctime)s %(levelname)-8s %(message)s", "%H:%M:%S"))
logging.basicConfig(level=logging.DEBUG, handlers=[manejador])

log = logging.getLogger("app")
log.debug("Detalle técnico")
log.info("Servicio iniciado")
log.warning("Uso de memoria elevado")
log.error("Fallo al guardar")
```

> Use colores solo en el manejador de consola; los archivos de log deben guardarse sin códigos ANSI.

---

## 10. Desactivar el color: redirección, `NO_COLOR` y opciones

Los códigos ANSI en un archivo o en una tubería resultan molestos (`[31mError[0m`). Criterios habituales para desactivar el color:

```python
import os
import sys

def usar_color(flujo=sys.stdout) -> bool:
    if os.environ.get("NO_COLOR"):                   # Convención https://no-color.org
        return False
    if os.environ.get("FORCE_COLOR"):
        return True
    return hasattr(flujo, "isatty") and flujo.isatty()
```

Patrón con colores condicionales:

```python
from colorama import Fore, Style

class Colores:
    def __init__(self, activo: bool):
        self.ok = Fore.GREEN if activo else ""
        self.error = Fore.RED if activo else ""
        self.aviso = Fore.YELLOW if activo else ""
        self.reset = Style.RESET_ALL if activo else ""

C = Colores(usar_color())
print(f"{C.ok}Correcto{C.reset}")
```

Eliminar códigos de un texto ya formateado:

```python
import re
sin_color = re.sub(r"\x1b\[[0-9;]*[A-Za-z]", "", texto_con_color)
```

> Con `init(strip=True)`, colorama elimina los códigos de toda la salida automáticamente.

---

## 11. Colores de 256 y 24 bits

colorama no tiene constantes para estos colores, pero se pueden escribir directamente; `just_fix_windows_console()` permite que funcionen en Windows 10/11.

```python
def fg256(n: int) -> str:
    return f"\x1b[38;5;{n}m"

def bg256(n: int) -> str:
    return f"\x1b[48;5;{n}m"

def fg_rgb(r: int, g: int, b: int) -> str:
    return f"\x1b[38;2;{r};{g};{b}m"

def fg_hex(hex_: str) -> str:
    hex_ = hex_.lstrip("#")
    return fg_rgb(*(int(hex_[i:i + 2], 16) for i in (0, 2, 4)))

print(fg256(208) + "Naranja (256 colores)" + Style.RESET_ALL)
print(fg_hex("#1F78B4") + "Azul corporativo (24 bits)" + Style.RESET_ALL)

# Paleta de 256 colores
for i in range(256):
    print(bg256(i) + f" {i:3d} " + Style.RESET_ALL, end="\n" if (i + 1) % 16 == 0 else "")
```

> La consola clásica de Windows (anterior a Windows 10) solo admite los 16 colores básicos.

---

## 12. Pruebas

Al probar funciones que imprimen con color, compare el texto sin códigos o desactive el color:

```python
import re

ANSI = re.compile(r"\x1b\[[0-9;]*m")

def test_mensaje_ok(capsys):
    ok("Copia completada")
    salida = ANSI.sub("", capsys.readouterr().out)
    assert salida.strip() == "✔ Copia completada"

def test_color_aplicado(capsys):
    ok("x")
    assert Fore.GREEN in capsys.readouterr().out
```

> Evite llamar a `init()` en las pruebas: sustituye `sys.stdout` y puede interferir con la captura de salida de `pytest`. `just_fix_windows_console()` no tiene ese problema.

---

## 13. Ejemplo práctico completo

**Diagnóstico del entorno de desarrollo**: comprueba la versión de Python, herramientas instaladas, espacio en disco, variables de entorno y archivos de configuración; muestra resultados con colores, alineación correcta pese a los códigos ANSI, resumen final y código de salida. Respeta `NO_COLOR`, la redirección a archivo y la opción `--sin-color`.

```python
"""
diagnostico.py — Diagnóstico del entorno de desarrollo con salida en color.
Requiere: pip install colorama
Uso: python diagnostico.py [--sin-color] [--detalle]
"""
import argparse
import os
import platform
import re
import shutil
import subprocess
import sys
from dataclasses import dataclass
from pathlib import Path
from typing import Callable

from colorama import Back, Fore, Style, just_fix_windows_console

ANSI = re.compile(r"\x1b\[[0-9;]*m")


# ------------------------------------------------------------------ Paleta
class Paleta:
    def __init__(self, activa: bool):
        self.activa = activa

    def __call__(self, texto: str, *codigos: str) -> str:
        if not self.activa:
            return texto
        return "".join(codigos) + texto + Style.RESET_ALL

    def ok(self, t): return self(t, Fore.GREEN, Style.BRIGHT)
    def aviso(self, t): return self(t, Fore.YELLOW, Style.BRIGHT)
    def error(self, t): return self(t, Fore.RED, Style.BRIGHT)
    def tenue(self, t): return self(t, Fore.LIGHTBLACK_EX)
    def titulo(self, t): return self(f" {t} ", Fore.WHITE, Back.BLUE, Style.BRIGHT)
    def dato(self, t): return self(t, Fore.CYAN)


def rellenar(texto: str, ancho: int) -> str:
    return texto + " " * max(0, ancho - len(ANSI.sub("", texto)))


# ------------------------------------------------------------------ Comprobaciones
@dataclass
class Resultado:
    nombre: str
    estado: str            # "ok" | "aviso" | "error"
    detalle: str
    sugerencia: str = ""


def comprobar_python() -> Resultado:
    v = sys.version_info
    texto = f"{v.major}.{v.minor}.{v.micro} ({platform.python_implementation()})"
    if v < (3, 10):
        return Resultado("Python", "error", texto, "Actualice a Python 3.10 o superior.")
    if v < (3, 12):
        return Resultado("Python", "aviso", texto, "Se recomienda Python 3.12 o superior.")
    return Resultado("Python", "ok", texto)


def comprobar_herramienta(nombre: str, argumento: str = "--version", obligatoria: bool = True) -> Callable[[], Resultado]:
    def comprobar() -> Resultado:
        ruta = shutil.which(nombre)
        if not ruta:
            return Resultado(nombre, "error" if obligatoria else "aviso", "no encontrado",
                             f"Instale {nombre} y compruebe que está en el PATH.")
        try:
            salida = subprocess.run([ruta, argumento], capture_output=True, text=True, timeout=5)
            version = (salida.stdout or salida.stderr).strip().splitlines()[0][:60]
        except (OSError, subprocess.TimeoutExpired, IndexError):
            version = "versión desconocida"
        return Resultado(nombre, "ok", version)
    return comprobar


def comprobar_disco() -> Resultado:
    uso = shutil.disk_usage(Path.cwd())
    libre_gb, porcentaje = uso.free / 1024**3, uso.free / uso.total
    texto = f"{libre_gb:.1f} GB libres ({porcentaje:.0%})"
    if porcentaje < 0.05:
        return Resultado("Disco", "error", texto, "Libere espacio: menos del 5 % disponible.")
    if porcentaje < 0.15:
        return Resultado("Disco", "aviso", texto, "Queda menos del 15 % de espacio.")
    return Resultado("Disco", "ok", texto)


def comprobar_entorno_virtual() -> Resultado:
    if sys.prefix != getattr(sys, "base_prefix", sys.prefix):
        return Resultado("Entorno virtual", "ok", Path(sys.prefix).name)
    return Resultado("Entorno virtual", "aviso", "no activo",
                     "Cree uno con 'python -m venv .venv' y actívelo.")


def comprobar_variables(*nombres: str) -> Callable[[], Resultado]:
    def comprobar() -> Resultado:
        faltan = [n for n in nombres if not os.environ.get(n)]
        if faltan:
            return Resultado("Variables de entorno", "aviso", f"faltan: {', '.join(faltan)}",
                             "Defínalas en su perfil o en un archivo .env.")
        return Resultado("Variables de entorno", "ok", f"{len(nombres)} definidas")
    return comprobar


def comprobar_archivos(*rutas: str) -> Callable[[], Resultado]:
    def comprobar() -> Resultado:
        faltan = [r for r in rutas if not Path(r).exists()]
        if faltan:
            return Resultado("Archivos del proyecto", "error", f"faltan: {', '.join(faltan)}",
                             "Ejecute el diagnóstico desde la raíz del proyecto.")
        return Resultado("Archivos del proyecto", "ok", ", ".join(rutas))
    return comprobar


COMPROBACIONES: list[tuple[str, list[Callable[[], Resultado]]]] = [
    ("Sistema", [comprobar_python, comprobar_disco, comprobar_entorno_virtual]),
    ("Herramientas", [comprobar_herramienta("git"), comprobar_herramienta("pip"),
                      comprobar_herramienta("docker", obligatoria=False),
                      comprobar_herramienta("node", obligatoria=False)]),
    ("Proyecto", [comprobar_variables("HOME" if os.name != "nt" else "USERPROFILE", "EDITOR"),
                  comprobar_archivos("pyproject.toml", "README.md")]),
]


# ------------------------------------------------------------------ Presentación
def mostrar(resultados_por_grupo: dict[str, list[Resultado]], c: Paleta, detalle: bool) -> None:
    iconos = {"ok": c.ok("✔ OK   "), "aviso": c.aviso("⚠ AVISO"), "error": c.error("✖ ERROR")}
    print(c.titulo("DIAGNÓSTICO DEL ENTORNO"), c.tenue(f"{platform.system()} {platform.release()}"))
    for grupo, resultados in resultados_por_grupo.items():
        print("\n" + c(grupo, Style.BRIGHT))
        print(c.tenue("─" * 70))
        for r in resultados:
            print(f"  {iconos[r.estado]}  {rellenar(c.dato(r.nombre), 24)}{r.detalle}")
            if r.sugerencia and (detalle or r.estado == "error"):
                print(f"  {' ' * 9}{c.tenue('↳ ' + r.sugerencia)}")


def resumen(todos: list[Resultado], c: Paleta) -> int:
    cuenta = {e: sum(r.estado == e for r in todos) for e in ("ok", "aviso", "error")}
    print("\n" + c.tenue("═" * 70))
    linea = (f"{c.ok(str(cuenta['ok']) + ' correctas')} · "
             f"{c.aviso(str(cuenta['aviso']) + ' avisos')} · "
             f"{c.error(str(cuenta['error']) + ' errores')}")
    print(f"  Resultado: {linea}")
    if cuenta["error"]:
        print("  " + c(" ENTORNO NO VÁLIDO ", Fore.WHITE, Back.RED, Style.BRIGHT))
        return 1
    if cuenta["aviso"]:
        print("  " + c(" ENTORNO VÁLIDO CON AVISOS ", Fore.BLACK, Back.YELLOW))
        return 0
    print("  " + c(" ENTORNO CORRECTO ", Fore.BLACK, Back.GREEN))
    return 0


def main() -> int:
    parser = argparse.ArgumentParser(description="Diagnóstico del entorno de desarrollo.")
    parser.add_argument("--sin-color", action="store_true", help="desactiva los colores")
    parser.add_argument("--detalle", action="store_true", help="muestra todas las sugerencias")
    args = parser.parse_args()

    just_fix_windows_console()
    color_activo = (not args.sin_color and not os.environ.get("NO_COLOR")
                    and (sys.stdout.isatty() or bool(os.environ.get("FORCE_COLOR"))))
    c = Paleta(color_activo)

    por_grupo = {grupo: [f() for f in funciones] for grupo, funciones in COMPROBACIONES}
    mostrar(por_grupo, c, args.detalle)
    return resumen([r for rs in por_grupo.values() for r in rs], c)


if __name__ == "__main__":
    sys.exit(main())
```

Uso:

```bash
python diagnostico.py
python diagnostico.py --detalle
python diagnostico.py > informe.txt          # Sin códigos de color en el archivo
NO_COLOR=1 python diagnostico.py
```

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Aparecen textos como `←[31m` o `[31m` en Windows | Consola sin ANSI activado. | Llame a `just_fix_windows_console()` al inicio. |
| El color «se queda» en las líneas siguientes o en el *prompt* | Falta `Style.RESET_ALL`. | Restablezca siempre o use `autoreset=True`. |
| Códigos ANSI en archivos de log o al redirigir | Se imprime color sin comprobar la salida. | Compruebe `isatty()`, `NO_COLOR` o use `init(strip=True)`. |
| Columnas desalineadas | `len()` cuenta los caracteres de los códigos ANSI. | Calcule el ancho visible eliminando los códigos. |
| `Style.DIM` no tiene efecto | No admitido en la consola clásica de Windows. | Use `Fore.LIGHTBLACK_EX` para texto secundario. |
| Captura de salida rota en pruebas | `init()` sustituye `sys.stdout`. | Use `just_fix_windows_console()` o `deinit()`. |
| `AttributeError: … has no attribute 'ITALIC'` | colorama no define cursiva ni subrayado. | Use el código ANSI directamente (`\x1b[3m`, `\x1b[4m`). |
| Colores distintos en cada terminal | Cada tema define su propia paleta. | No transmita información solo con el color: añada texto o iconos. |

---

## 15. Buenas prácticas

- Llame a `just_fix_windows_console()` una vez, al inicio del programa.
- Centralice los colores en funciones o en una clase «paleta» con nombres semánticos (`ok`, `error`, `aviso`).
- Restablezca siempre el estilo al final de cada texto coloreado.
- Desactive el color cuando la salida no sea una terminal, si existe `NO_COLOR` o si el usuario lo pide.
- No use el color como único indicador: acompáñelo de iconos o palabras (accesibilidad y archivos de log).
- Escriba los errores en `stderr`.
- Si la aplicación necesita tablas, paneles o barras de progreso, use Rich en lugar de construirlos a mano.

---

## 16. Referencia rápida

```python
from colorama import Fore, Back, Style, Cursor, just_fix_windows_console, init, deinit, colorama_text
just_fix_windows_console()                        # Recomendado
init(autoreset=True, strip=None, convert=None, wrap=True) · deinit() · with colorama_text(): ...

Fore/Back: BLACK RED GREEN YELLOW BLUE MAGENTA CYAN WHITE RESET + LIGHT*_EX
Style: BRIGHT DIM NORMAL RESET_ALL
Cursor.UP(n) DOWN(n) FORWARD(n) BACK(n) POS(x, y) · ansi.clear_screen() · ansi.clear_line()

print(Fore.RED + Back.WHITE + Style.BRIGHT + "texto" + Style.RESET_ALL)
256 colores: "\x1b[38;5;{n}m" / fondo "\x1b[48;5;{n}m" · RGB: "\x1b[38;2;{r};{g};{b}m"
Quitar ANSI: re.sub(r"\x1b\[[0-9;]*[A-Za-z]", "", texto) · Respetar NO_COLOR e isatty()
```

---

## 17. Ejercicios propuestos

1. Cree una función `imprimir_diff(antes, despues)` que muestre las líneas añadidas en verde y las eliminadas en rojo (puede usar `difflib`).
2. Implemente un `grep` simplificado que resalte con fondo amarillo las coincidencias de un patrón en un archivo.
3. Añada a un script existente un formateador de `logging` con colores por nivel que solo se active cuando la salida sea una terminal.
4. Muestre la paleta completa de 16 colores combinando cada `Fore` con cada `Back` en una cuadrícula.
5. Amplíe el ejemplo completo con una comprobación de puertos ocupados (`socket`) y una opción `--json` que imprima los resultados sin color.

---

## 18. Referencias

- Repositorio y documentación: https://github.com/tartley/colorama
- PyPI: https://pypi.org/project/colorama/
- Secuencias ANSI (referencia): https://en.wikipedia.org/wiki/ANSI_escape_code
- Convención NO_COLOR: https://no-color.org/
