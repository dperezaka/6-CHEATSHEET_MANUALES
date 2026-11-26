# Manual de uso: yaspin y halo

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Mostrar *spinners* animados durante tareas o esperas
> **Tipo:** Librerías de terceros (`pip install yaspin` / `pip install halo`) · **Versiones de referencia:** yaspin 3.x · halo 0.0.31

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [yaspin: uso básico](#4-yaspin-uso-básico)
5. [yaspin: spinners, colores y posición](#5-yaspin-spinners-colores-y-posición)
6. [yaspin: actualizar texto, escribir y finalizar](#6-yaspin-actualizar-texto-escribir-y-finalizar)
7. [yaspin: temporizador, ocultar y pedir datos](#7-yaspin-temporizador-ocultar-y-pedir-datos)
8. [yaspin: decoradores y señales](#8-yaspin-decoradores-y-señales)
9. [halo: uso básico](#9-halo-uso-básico)
10. [halo: estados finales, animaciones y opciones](#10-halo-estados-finales-animaciones-y-opciones)
11. [Comparativa yaspin / halo](#11-comparativa-yaspin--halo)
12. [Spinners con hilos, procesos y asyncio](#12-spinners-con-hilos-procesos-y-asyncio)
13. [Entornos sin terminal, Windows y CI](#13-entornos-sin-terminal-windows-y-ci)
14. [Pruebas](#14-pruebas)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**yaspin** (*Yet Another Terminal Spinner*) y **halo** son librerías que muestran un **indicador animado** (*spinner*) en la terminal mientras el programa realiza una tarea cuya duración se desconoce: una consulta a una API, una conexión a una base de datos, una compilación o una instalación. Al terminar, el *spinner* se sustituye por un símbolo de resultado (✔, ✖, ⚠, ℹ) y un mensaje.

El *spinner* se anima en un **hilo en segundo plano**, de modo que el código principal sigue ejecutándose sin cambios.

**yaspin**
- Mantenida activamente, con anotaciones de tipo.
- Más de 80 animaciones incluidas (las de *cli-spinners*).
- Temporizador integrado, ocultar/mostrar, gestión de señales, decorador que inyecta el *spinner*.

**halo**
- API muy expresiva: `succeed()`, `fail()`, `warn()`, `info()`.
- Animaciones del texto largo (`bounce`, `marquee`) y soporte de Jupyter.
- **Sin mantenimiento desde 2020** (versión 0.0.31). Funciona, pero se recomienda yaspin para proyectos nuevos.

**Limitaciones comunes:**
- No muestran progreso cuantitativo (para eso: tqdm o Rich `Progress`).
- Imprimir con `print()` mientras giran descoloca la línea.
- Animan incluso cuando la salida no es una terminal, por lo que conviene desactivarlas en CI o redirecciones.

| Si necesita… | Opción |
|---|---|
| *Spinner* ligero y mantenido | **yaspin** |
| *Spinner* con métodos `succeed`/`warn`/`info` y animación del texto | **halo** |
| Progreso con porcentaje y tiempo restante | tqdm / Rich `Progress` |
| *Spinner* integrado con salida enriquecida | Rich (`console.status`) |

---

## 2. Instalación e importación

```bash
pip install yaspin
pip install halo
```

```python
from yaspin import yaspin, kbi_safe_yaspin, inject_spinner
from yaspin.spinners import Spinners

from halo import Halo
```

> halo instala como dependencias `colorama`, `termcolor`, `log_symbols`, `spinners` y `six`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| *Spinner* | Secuencia de fotogramas (`⠋⠙⠹⠸…`) que se muestra en bucle. |
| Intervalo | Milisegundos entre fotogramas. |
| Texto | Mensaje que acompaña al *spinner* y puede cambiar durante la tarea. |
| Estado final | Símbolo y texto que sustituyen al *spinner* al terminar (`ok`, `fail`…). |
| Hilo de animación | Hilo en segundo plano que redibuja la línea. |
| Gestor de contexto | `with yaspin(...)` / `with Halo(...)`: inicia y detiene automáticamente. |
| Escritura segura | `sp.write()` (yaspin) / `stop_and_persist()` (halo) para imprimir sin romper la animación. |

---

## 4. yaspin: uso básico

### 4.1 Gestor de contexto (recomendado)

```python
import time
from yaspin import yaspin

with yaspin(text="Conectando con el servidor", color="cyan") as sp:
    time.sleep(2)                     # Tarea larga
    sp.ok("✔")                        # Sustituye el spinner por ✔
```

```
✔ Conectando con el servidor
```

### 4.2 Inicio y parada manual

```python
sp = yaspin(text="Descargando datos")
sp.start()
try:
    descargar()
    sp.ok("✔")
except Exception:
    sp.fail("✖")
    raise
finally:
    sp.stop()                         # Seguro aunque ya se haya llamado a ok/fail
```

### 4.3 Como decorador

```python
@yaspin(text="Generando informe…")
def generar_informe():
    time.sleep(3)

generar_informe()
```

---

## 5. yaspin: spinners, colores y posición

### 5.1 Elegir la animación

```python
from yaspin.spinners import Spinners

with yaspin(Spinners.dots12, text="Procesando"): ...
with yaspin(Spinners.line, text="Compatible con consolas antiguas"): ...
with yaspin(Spinners.bouncingBar, text="Esperando"): ...
with yaspin(Spinners.earth, text="Sincronizando"): ...
```

Algunas animaciones: `dots`, `dots2`…`dots12`, `line`, `pipe`, `simpleDots`, `star`, `arc`, `circle`, `circleHalves`, `squareCorners`, `toggle`, `arrow3`, `bouncingBar`, `bouncingBall`, `clock`, `earth`, `moon`, `runner`, `pong`, `shark`, `weather`. Para listarlas:

```python
print(Spinners._fields)
```

### 5.2 Spinner personalizado

```python
from yaspin import Spinner

reloj_arena = Spinner(["⏳", "⌛"], 400)                    # (fotogramas, intervalo en ms)
puntos = Spinner([".  ", ".. ", "...", "   "], 250)

with yaspin(puntos, text="Cargando"):
    time.sleep(2)
```

### 5.3 Color, fondo y atributos

```python
with yaspin(text="Compilando", color="yellow", on_color="on_blue", attrs=["bold"]) as sp:
    ...
sp.color = "green"                   # Cambiar durante la ejecución
```

- Colores: `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white` (y variantes `light_*`).
- Fondos: `on_red`, `on_green`…
- Atributos: `bold`, `dark`, `underline`, `blink`, `reverse`, `concealed`.

Atajos encadenables:

```python
with yaspin(text="Analizando").white.bold.shark.on_blue as sp:
    ...
```

> El color se aplica solo al *spinner*, no al texto. Para colorear el texto, inclúyale códigos ANSI (por ejemplo, con `colorama`).

### 5.4 Posición y sentido

```python
yaspin(text="Texto a la izquierda del spinner", side="right")
yaspin(text="Animación invertida", reversal=True)
yaspin(text="Esperando", ellipsis="…")           # Puntos suspensivos animados tras el texto
```

---

## 6. yaspin: actualizar texto, escribir y finalizar

### 6.1 Cambiar el texto durante la tarea

```python
with yaspin(text="Preparando") as sp:
    for i, paso in enumerate(pasos, 1):
        sp.text = f"[{i}/{len(pasos)}] {paso.descripcion}"
        paso.ejecutar()
    sp.ok("✔")
```

### 6.2 Escribir líneas sin romper la animación

```python
with yaspin(text="Instalando paquetes") as sp:
    for paquete in paquetes:
        instalar(paquete)
        sp.write(f"  ✓ {paquete}")       # Imprime encima y el spinner sigue debajo
    sp.ok("✔")
```

```
  ✓ requests
  ✓ pandas
  ✓ numpy
✔ Instalando paquetes
```

### 6.3 Finalizar con distintos símbolos

```python
sp.ok("✔")                 # Éxito
sp.fail("✖")               # Fallo
sp.ok("⚠")                 # Cualquier texto/símbolo sirve como marca final
sp.text = "Completado en 3,2 s"; sp.ok("✔")     # Cambiar el texto antes de finalizar
sp.stop()                  # Detener sin dejar línea final
```

Con colores en la marca final:

```python
from colorama import Fore, Style
sp.ok(Fore.GREEN + "✔" + Style.RESET_ALL)
sp.fail(Fore.RED + "✖" + Style.RESET_ALL)
```

---

## 7. yaspin: temporizador, ocultar y pedir datos

### 7.1 Temporizador

```python
with yaspin(text="Consultando la API", timer=True) as sp:
    consultar()
    sp.ok("✔")
# ✔ Consultando la API (0:00:02.41)

print(sp.elapsed_time)        # Segundos transcurridos (float)
```

### 7.2 Ocultar temporalmente

```python
with yaspin(text="Procesando") as sp:
    procesar_parte_1()
    with sp.hidden():                                  # Oculta el spinner dentro del bloque
        respuesta = input("¿Continuar con la parte 2? [s/n] ")
    if respuesta.lower() == "s":
        procesar_parte_2()
    sp.ok("✔")
```

Equivalente manual: `sp.hide()` … `sp.show()`.

> Use siempre `hidden()` para pedir datos al usuario o para imprimir varias líneas seguidas.

---

## 8. yaspin: decoradores y señales

### 8.1 Inyectar el spinner en la función

```python
from yaspin import inject_spinner
from yaspin.core import Yaspin

@inject_spinner(Spinners.dots, text="Procesando…", timer=True)
def procesar(sp: Yaspin, registros: list) -> int:
    for i, r in enumerate(registros, 1):
        sp.text = f"Procesando {i}/{len(registros)}"
        guardar(r)
    sp.ok("✔")
    return len(registros)

total = procesar(registros)            # El spinner se pasa automáticamente
```

### 8.2 Ctrl+C limpio

Por defecto, al pulsar Ctrl+C puede quedar la línea del *spinner* a medias. `kbi_safe_yaspin` detiene la animación correctamente:

```python
from yaspin import kbi_safe_yaspin

with kbi_safe_yaspin(text="Pulse Ctrl+C para cancelar") as sp:
    tarea_larga()
```

### 8.3 Gestores de señales personalizados (`sigmap`)

```python
import signal
from yaspin.core import fancy_handler

def al_interrumpir(signum, frame, spinner):
    spinner.fail("✖")
    spinner.stop()
    raise SystemExit(130)

with yaspin(sigmap={signal.SIGINT: al_interrumpir}, text="Migrando base de datos") as sp:
    migrar()
    sp.ok("✔")

# Manejador predefinido con mensaje decorado
with yaspin(sigmap={signal.SIGINT: fancy_handler}, text="Trabajando"):
    ...
```

> `sigmap` solo puede usarse desde el hilo principal.

---

## 9. halo: uso básico

### 9.1 Gestor de contexto

```python
from halo import Halo

with Halo(text="Cargando configuración", spinner="dots", color="cyan"):
    cargar()
# Al salir del bloque, el spinner simplemente desaparece
```

### 9.2 Inicio y estados finales

```python
spinner = Halo(text="Conectando", spinner="dots")
spinner.start()
try:
    conectar()
    spinner.succeed("Conexión establecida")       # ✔ verde
except TimeoutError:
    spinner.warn("Conexión lenta, reintentando")    # ⚠ amarillo
except Exception as e:
    spinner.fail(f"Error de conexión: {e}")         # ✖ rojo
```

### 9.3 Decorador

```python
@Halo(text="Calculando estadísticas", spinner="dots")
def calcular():
    time.sleep(2)
```

---

## 10. halo: estados finales, animaciones y opciones

### 10.1 Estados finales

| Método | Símbolo |
|---|---|
| `succeed(texto)` | ✔ (verde) |
| `fail(texto)` | ✖ (rojo) |
| `warn(texto)` | ⚠ (amarillo) |
| `info(texto)` | ℹ (azul) |
| `stop_and_persist(symbol=, text=)` | Símbolo y texto personalizados |
| `stop()` | Detiene y borra la línea |
| `clear()` | Borra la línea sin detener |

```python
spinner.stop_and_persist(symbol="🚀", text="Despliegue completado")
```

### 10.2 Parámetros

```python
Halo(
    text="Procesando",
    spinner="dots",                 # Nombre o diccionario {"interval": 100, "frames": [...]}
    color="cyan",                   # Color del spinner
    text_color="white",             # Color del texto
    animation="bounce",             # Texto largo: 'bounce' o 'marquee'
    placement="left",               # 'left' o 'right'
    interval=-1,                    # ms entre fotogramas (-1: el del spinner)
    enabled=True,                   # False: no muestra nada (útil en CI)
    stream=sys.stdout,              # Flujo de salida
)
```

Spinner propio:

```python
Halo(text="Esperando", spinner={"interval": 200, "frames": ["◐", "◓", "◑", "◒"]})
```

### 10.3 Cambiar propiedades en ejecución

```python
spinner.text = "Paso 2 de 3"
spinner.color = "magenta"
spinner.spinner = "line"
```

### 10.4 Jupyter

```python
from halo import HaloNotebook

with HaloNotebook(text="Entrenando modelo"):
    entrenar()
```

---

## 11. Comparativa yaspin / halo

| Aspecto | yaspin | halo |
|---|---|---|
| Mantenimiento | Activo | Sin actualizaciones desde 2020 |
| Anotaciones de tipo | Sí | No |
| Estados finales | `ok(símbolo)`, `fail(símbolo)` | `succeed`, `fail`, `warn`, `info`, `stop_and_persist` |
| Escribir sin romper | `sp.write()` | No tiene equivalente directo (detener, imprimir, reiniciar) |
| Ocultar para pedir datos | `sp.hidden()` | `stop()` + `start()` |
| Temporizador | `timer=True` | No |
| Color del texto | No (solo el spinner) | `text_color` |
| Animación del texto largo | No | `bounce`, `marquee` |
| Desactivar | Condicional manual | `enabled=False` |
| Señales (Ctrl+C) | `kbi_safe_yaspin`, `sigmap` | Básico |
| Jupyter | Detecta el entorno | `HaloNotebook` |

**Recomendación:** yaspin para proyectos nuevos; halo si ya está en uso o se necesita `warn/info` o color de texto sin código adicional.

---

## 12. Spinners con hilos, procesos y asyncio

El *spinner* ya usa su propio hilo; el trabajo puede hacerse en el hilo principal o en otros:

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with yaspin(text="Comprobando servidores", timer=True) as sp:
    with ThreadPoolExecutor(max_workers=10) as ex:
        futuros = {ex.submit(comprobar, s): s for s in servidores}
        caidos = 0
        for i, f in enumerate(as_completed(futuros), 1):
            if not f.result():
                caidos += 1
                sp.write(f"  ✖ {futuros[f]} no responde")
            sp.text = f"Comprobando servidores ({i}/{len(servidores)})"
    (sp.fail if caidos else sp.ok)("✖" if caidos else "✔")
```

Con `asyncio`, el *spinner* funciona igual (su hilo no bloquea el bucle de eventos):

```python
import asyncio

async def main():
    with yaspin(text="Descargando en paralelo") as sp:
        resultados = await asyncio.gather(*(descargar(u) for u in urls))
        sp.ok("✔")
    return resultados
```

> No inicie un *spinner* dentro de cada proceso de `multiprocessing`: varios procesos escribiendo en la misma línea producen una salida ilegible. Use un único *spinner* en el proceso principal.

---

## 13. Entornos sin terminal, Windows y CI

Ambas librerías **siguen animando** aunque la salida se redirija a un archivo o se ejecute en CI, lo que llena los logs de caracteres de control. Desactívelas cuando no haya terminal:

```python
import sys
from contextlib import nullcontext

def spinner(texto: str):
    if sys.stdout.isatty():
        return yaspin(text=texto, timer=True)
    print(f"• {texto}…")
    return nullcontext(_SpinnerNulo())

class _SpinnerNulo:
    text = ""
    def ok(self, s="✔"): print(f"  {s} hecho")
    def fail(self, s="✖"): print(f"  {s} error")
    def write(self, t): print(t)
    def hidden(self): return nullcontext()
```

Con halo es más sencillo:

```python
Halo(text="Procesando", enabled=sys.stdout.isatty())
```

**Windows:**
- En Windows Terminal y PowerShell 7 los *spinners* Unicode funcionan correctamente.
- En la consola clásica (cmd.exe con fuente de mapa de bits) pueden verse caracteres extraños: use `Spinners.line` (`-\|/`) o `spinner="line"` en halo.
- Si los colores aparecen como códigos (`←[36m`), llame a `colorama.just_fix_windows_console()` al inicio.

---

## 14. Pruebas

Lo más práctico es **no** probar la animación sino la lógica, inyectando el *spinner* o desactivándolo:

```python
from unittest.mock import MagicMock

def importar(registros, sp):
    for r in registros:
        sp.text = f"Importando {r}"
    sp.ok("✔")
    return len(registros)

def test_importar():
    sp = MagicMock()
    assert importar(["a", "b"], sp) == 2
    sp.ok.assert_called_once_with("✔")
```

Capturar la salida real de yaspin:

```python
import io
from yaspin import yaspin

def test_salida():
    flujo = io.StringIO()
    with yaspin(text="Tarea", stream=flujo) as sp:
        sp.ok("✔")
    assert "✔ Tarea" in flujo.getvalue()
```

---

## 15. Ejemplo práctico completo

**Asistente de configuración de proyecto**: ejecuta una secuencia de pasos (comprobaciones, creación de entorno virtual, instalación simulada de dependencias, plantillas, repositorio git) mostrando un *spinner* por paso con temporizador, mensajes intermedios con `write`, una pregunta con el *spinner* oculto, gestión de Ctrl+C, colores en los resultados y modo sin animación cuando no hay terminal. Incluye la versión equivalente de un paso con halo.

```python
"""
configurar.py — Asistente de configuración de proyectos con spinners.
Requiere: pip install yaspin colorama   (halo opcional para la demostración final)
Uso: python configurar.py <nombre_proyecto> [--sin-git] [--rapido]
"""
import argparse
import random
import shutil
import subprocess
import sys
import time
import venv
from contextlib import nullcontext
from pathlib import Path
from typing import Callable

from colorama import Fore, Style, just_fix_windows_console
from yaspin import kbi_safe_yaspin
from yaspin.spinners import Spinners

just_fix_windows_console()
INTERACTIVO = sys.stdout.isatty()


def marca(simbolo: str, color: str) -> str:
    return f"{color}{simbolo}{Style.RESET_ALL}" if INTERACTIVO else simbolo


OK, FALLO, AVISO = marca("✔", Fore.GREEN), marca("✖", Fore.RED), marca("⚠", Fore.YELLOW)


class ErrorPaso(Exception):
    """Error controlado de un paso, con mensaje para el usuario."""


# ------------------------------------------------------------------ Spinner con modo sin terminal
class SpinnerTexto:
    """Sustituto sin animación para CI, archivos o tuberías."""

    def __init__(self, texto: str):
        self.text = texto
        print(f"• {texto}…", flush=True)

    def write(self, linea: str) -> None:
        print(linea, flush=True)

    def ok(self, simbolo: str = "✔") -> None:
        print(f"  {simbolo} {self.text}", flush=True)

    fail = ok

    def hidden(self):
        return nullcontext()


def spinner(texto: str):
    if INTERACTIVO:
        return kbi_safe_yaspin(Spinners.dots, text=texto, color="cyan", timer=True)
    return nullcontext(SpinnerTexto(texto))


# ------------------------------------------------------------------ Pasos
class Asistente:
    def __init__(self, nombre: str, usar_git: bool, rapido: bool):
        self.nombre = nombre
        self.raiz = Path(nombre).resolve()
        self.usar_git = usar_git
        self.pausa = 0.1 if rapido else 1.0
        self.avisos: list[str] = []

    def esperar(self, factor: float = 1.0) -> None:
        time.sleep(self.pausa * factor * random.uniform(0.6, 1.4))

    def comprobar_requisitos(self, sp) -> None:
        if sys.version_info < (3, 10):
            raise ErrorPaso("Se necesita Python 3.10 o superior.")
        sp.write(f"  {OK} Python {sys.version.split()[0]}")
        self.esperar(0.5)
        if self.usar_git and not shutil.which("git"):
            self.usar_git = False
            self.avisos.append("git no está instalado: se omite el repositorio.")
            sp.write(f"  {AVISO} git no encontrado (se omitirá)")
        elif self.usar_git:
            sp.write(f"  {OK} git disponible")
        if self.raiz.exists() and any(self.raiz.iterdir()):
            with sp.hidden():
                try:
                    respuesta = input(f"{AVISO} La carpeta {self.raiz.name} no está vacía. ¿Continuar? [s/N] ")
                except EOFError:                       # Sin entrada disponible (CI, tubería)
                    respuesta = "n"
            if respuesta.strip().lower() != "s":
                raise ErrorPaso("Operación cancelada por el usuario.")

    def crear_estructura(self, sp) -> None:
        for carpeta in ("src/" + self.nombre.replace("-", "_"), "tests", "docs"):
            sp.text = f"Creando {carpeta}/"
            (self.raiz / carpeta).mkdir(parents=True, exist_ok=True)
            self.esperar(0.3)
        sp.text = "Estructura de carpetas creada"

    def crear_entorno(self, sp) -> None:
        destino = self.raiz / ".venv"
        if destino.exists():
            sp.text = "Entorno virtual existente reutilizado"
            return
        venv.EnvBuilder(with_pip=False).create(destino)
        sp.text = "Entorno virtual creado en .venv"

    def instalar_dependencias(self, sp) -> None:
        paquetes = ["requests", "typer", "rich", "pydantic", "pytest", "ruff"]
        for i, paquete in enumerate(paquetes, 1):
            sp.text = f"Instalando dependencias ({i}/{len(paquetes)}): {paquete}"
            self.esperar(0.6)
            if paquete == "pydantic" and random.random() < 0.3:
                sp.write(f"  {AVISO} {paquete}: reintento tras error de red")
                self.avisos.append(f"{paquete} necesitó un reintento.")
                self.esperar(0.6)
            sp.write(f"  {OK} {paquete}")
        sp.text = f"{len(paquetes)} dependencias instaladas (simulado)"

    def generar_archivos(self, sp) -> None:
        paquete = self.nombre.replace("-", "_")
        archivos = {
            "pyproject.toml": f'[project]\nname = "{self.nombre}"\nversion = "0.1.0"\nrequires-python = ">=3.10"\n',
            "README.md": f"# {self.nombre}\n\nProyecto generado automáticamente.\n",
            f"src/{paquete}/__init__.py": '__version__ = "0.1.0"\n',
            "tests/test_version.py": f"from {paquete} import __version__\n\n\ndef test_version():\n    assert __version__\n",
            ".gitignore": ".venv/\n__pycache__/\n*.egg-info/\ndist/\n",
        }
        for ruta, contenido in archivos.items():
            sp.text = f"Generando {ruta}"
            (self.raiz / ruta).write_text(contenido, encoding="utf-8")
            self.esperar(0.2)
        sp.text = f"{len(archivos)} archivos generados"

    def iniciar_git(self, sp) -> None:
        if not self.usar_git:
            sp.text = "Repositorio git omitido"
            return
        for comando in (["git", "init", "-q"], ["git", "add", "."]):
            resultado = subprocess.run(comando, cwd=self.raiz, capture_output=True, text=True)
            if resultado.returncode != 0:
                raise ErrorPaso(f"Fallo en '{' '.join(comando)}': {resultado.stderr.strip()}")
        sp.text = "Repositorio git inicializado"

    # ---------- Orquestación
    def ejecutar(self) -> int:
        pasos: list[tuple[str, Callable]] = [
            ("Comprobando requisitos", self.comprobar_requisitos),
            ("Creando estructura", self.crear_estructura),
            ("Creando entorno virtual", self.crear_entorno),
            ("Instalando dependencias", self.instalar_dependencias),
            ("Generando archivos", self.generar_archivos),
            ("Inicializando git", self.iniciar_git),
        ]
        print(Style.BRIGHT + f"\nConfigurando el proyecto «{self.nombre}»\n" + Style.RESET_ALL)
        inicio = time.perf_counter()
        for i, (titulo, funcion) in enumerate(pasos, 1):
            with spinner(f"[{i}/{len(pasos)}] {titulo}") as sp:
                try:
                    funcion(sp)
                    sp.text = f"[{i}/{len(pasos)}] {sp.text}"
                    sp.ok(OK)
                except ErrorPaso as e:
                    sp.text = f"[{i}/{len(pasos)}] {titulo}: {e}"
                    sp.fail(FALLO)
                    return 1

        print(f"\n{marca('Proyecto listo', Fore.GREEN + Style.BRIGHT)} "
              f"en {time.perf_counter() - inicio:.1f} s → {self.raiz}")
        for aviso in self.avisos:
            print(f"  {AVISO} {aviso}")
        activar = r".venv\Scripts\activate" if sys.platform == "win32" else "source .venv/bin/activate"
        print(f"\nSiguientes pasos:\n  cd {self.nombre}\n  {activar}")
        return 0


def demostracion_halo() -> None:
    """El mismo patrón con halo (si está instalado)."""
    try:
        from halo import Halo
    except ImportError:
        return
    with Halo(text="Verificando con halo", spinner="dots", enabled=INTERACTIVO) as h:
        time.sleep(0.5)
        h.succeed("Verificación final completada (halo)")


def main() -> int:
    parser = argparse.ArgumentParser(description="Asistente de configuración de proyectos.")
    parser.add_argument("nombre", help="nombre del proyecto (minúsculas y guiones)")
    parser.add_argument("--sin-git", action="store_true")
    parser.add_argument("--rapido", action="store_true", help="reduce las esperas simuladas")
    args = parser.parse_args()
    try:
        codigo = Asistente(args.nombre, not args.sin_git, args.rapido).ejecutar()
    except KeyboardInterrupt:
        print(f"\n{FALLO} Configuración interrumpida.")
        return 130
    if codigo == 0:
        demostracion_halo()
    return codigo


if __name__ == "__main__":
    sys.exit(main())
```

Salida (resumida):

```
Configurando el proyecto «mi-api»

✔ [1/6] Comprobando requisitos (0:00:01.12)
✔ [2/6] Estructura de carpetas creada (0:00:00.91)
✔ [3/6] Entorno virtual creado en .venv (0:00:00.34)
  ✔ requests
  ✔ typer
  ...
✔ [4/6] 6 dependencias instaladas (simulado) (0:00:03.80)
✔ [5/6] 5 archivos generados (0:00:01.02)
✔ [6/6] Repositorio git inicializado (0:00:00.05)

Proyecto listo en 7.3 s → /home/ana/mi-api
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| La línea del *spinner* se mezcla con otras salidas | Se usa `print()` mientras gira. | `sp.write()` o `with sp.hidden():` (yaspin); detener y reiniciar (halo). |
| `input()` queda oculto o la animación pisa la pregunta | El *spinner* sigue activo. | `with sp.hidden(): input(...)`. |
| El log de CI se llena de caracteres extraños | La animación continúa sin terminal. | Desactívela si `not sys.stdout.isatty()` (`enabled=False` en halo). |
| `UserWarning: color … not supported when output stream is not a TTY` | yaspin con color y salida redirigida. | No pase `color` cuando no hay terminal. |
| Queda una línea a medias tras Ctrl+C | Interrupción durante la animación. | `kbi_safe_yaspin` o `sigmap`. |
| Caracteres raros en cmd.exe | Fuente sin glifos Unicode. | `Spinners.line` / `spinner="line"` o Windows Terminal. |
| `ValueError: signal only works in main thread` | `sigmap` usado fuera del hilo principal. | Configure las señales en el hilo principal. |
| El símbolo final aparece sin color | `ok()`/`fail()` reciben solo el texto. | Pase el símbolo con códigos ANSI (colorama). |
| Varias barras/spinners se pisan | Dos *spinners* activos a la vez. | Use uno solo; para varias tareas, Rich `Progress`. |

---

## 17. Buenas prácticas

- Use *spinners* solo para esperas de duración desconocida; si conoce el total, use una barra de progreso.
- Use siempre el gestor de contexto (`with`) para garantizar que la animación se detiene.
- Actualice `sp.text` con el paso en curso para dar contexto («3/6 Instalando…»).
- Finalice siempre con un estado claro (`ok`/`fail` o `succeed`/`fail`/`warn`).
- Imprima con `sp.write()` y pida datos dentro de `sp.hidden()`.
- Desactive la animación cuando la salida no sea una terminal.
- Prefiera yaspin en proyectos nuevos por su mantenimiento activo.
- Si la aplicación ya usa Rich, `console.status()` evita añadir otra dependencia.

---

## 18. Referencia rápida

```python
# yaspin
from yaspin import yaspin, kbi_safe_yaspin, inject_spinner, Spinner
from yaspin.spinners import Spinners
with yaspin(Spinners.dots, text=, color=, on_color=, attrs=[], side="left", reversal=False,
            timer=True, ellipsis="…", sigmap={}, stream=) as sp:
    sp.text = "…"; sp.color = "green"; sp.write("línea"); with sp.hidden(): input()
    sp.ok("✔") / sp.fail("✖") / sp.stop()
sp.elapsed_time · yaspin(...).bold.cyan.dots12 · Spinner(["⏳","⌛"], 400)
@yaspin(text=) · @inject_spinner(text=) def f(sp, ...) · kbi_safe_yaspin(...)

# halo
from halo import Halo, HaloNotebook
with Halo(text=, spinner="dots"|{"interval", "frames"}, color=, text_color=, animation="bounce"|"marquee",
          placement="left", enabled=sys.stdout.isatty()) as h: ...
h.start() · h.text = "…" · h.succeed(t) · h.fail(t) · h.warn(t) · h.info(t)
h.stop_and_persist(symbol="🚀", text=) · h.stop() · h.clear() · @Halo(text=)
```

---

## 19. Ejercicios propuestos

1. Envuelva una consulta lenta (simulada con `time.sleep`) en un *spinner* de yaspin con temporizador y finalice con ✔ o ✖ según el resultado.
2. Cree una función `paso(texto)` que devuelva un *spinner* de yaspin en terminal o una versión de texto plano en CI, y úsela en un script de despliegue.
3. Escriba un comprobador de URLs que use un único *spinner* actualizando el texto con el progreso y `sp.write()` para las que fallen.
4. Reproduzca el ejemplo completo con halo, usando `warn()` para los pasos con avisos.
5. Diseñe un *spinner* personalizado con fotogramas propios y compárelo visualmente con `Spinners.dots` y `Spinners.line`.

---

## 20. Referencias

- yaspin: https://github.com/pavdmyt/yaspin
- halo: https://github.com/manrajgrover/halo
- Catálogo de animaciones (cli-spinners): https://github.com/sindresorhus/cli-spinners
