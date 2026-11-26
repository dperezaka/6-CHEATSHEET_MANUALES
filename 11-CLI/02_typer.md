# Manual de uso: Typer

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Parsear argumentos, opciones y subcomandos con *type hints*
> **Tipo:** Librería de terceros (`pip install typer`) · **Versión de referencia:** 0.29

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer programa](#4-primer-programa)
5. [Argumentos](#5-argumentos)
6. [Opciones](#6-opciones)
7. [Tipos de datos admitidos](#7-tipos-de-datos-admitidos)
8. [Varios comandos y subcomandos](#8-varios-comandos-y-subcomandos)
9. [Callbacks y contexto](#9-callbacks-y-contexto)
10. [Ayuda y documentación automática](#10-ayuda-y-documentación-automática)
11. [Interacción con el usuario](#11-interacción-con-el-usuario)
12. [Salida, errores y códigos de salida](#12-salida-errores-y-códigos-de-salida)
13. [Integración con Rich](#13-integración-con-rich)
14. [Autocompletado en la shell](#14-autocompletado-en-la-shell)
15. [Pruebas con `CliRunner`](#15-pruebas-con-clirunner)
16. [Empaquetar y distribuir el comando](#16-empaquetar-y-distribuir-el-comando)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**Typer** es una librería para construir aplicaciones de línea de comandos a partir de **funciones Python con anotaciones de tipo**. Los parámetros de la función se convierten automáticamente en argumentos y opciones; los tipos determinan la conversión y la validación, y los *docstrings* generan la ayuda. Está construida sobre **Click** y, si **Rich** está instalado, muestra la ayuda y los errores con formato.

**Ventajas:**
- Muy poco código: la firma de la función *es* la interfaz.
- Validación y conversión automáticas según el tipo (`int`, `Path`, `Enum`, `datetime`…).
- Ayuda (`--help`) generada automáticamente y con formato enriquecido.
- Autocompletado de la shell incluido.
- Compatible con los editores: autocompletado y comprobación de tipos en el código.
- Acceso completo a Click cuando se necesita algo avanzado.

**Limitaciones:**
- Dependencia externa (frente a `argparse`).
- Algunas funciones muy avanzadas exigen recurrir a la API de Click.
- La ayuda con Rich añade un pequeño coste de arranque.

| Si necesita… | Opción |
|---|---|
| CLI moderna con poco código y tipos | **Typer** |
| Control total, plugins y CLIs muy grandes | Click |
| Sin dependencias externas | argparse |
| Exponer código existente sin escribir la CLI | Fire |
| Preguntas interactivas (menús, formularios) | questionary / InquirerPy |

---

## 2. Instalación e importación

```bash
pip install typer          # Incluye Rich y shellingham (autocompletado)
pip install typer-slim     # Versión mínima, sin Rich ni shellingham
```

Importación habitual:

```python
from typing import Annotated
import typer
```

> Typer recomienda declarar argumentos y opciones con `Annotated` (Python ≥ 3.9). Es la sintaxis que se usa en este manual.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Aplicación (`typer.Typer`) | Objeto que agrupa uno o varios comandos. |
| Comando | Función decorada con `@app.command()`. |
| Argumento | Valor posicional: `app.py copiar origen.txt`. Obligatorio por defecto. |
| Opción | Valor con nombre: `--salida informe.csv`, `-v`. Opcional por defecto. |
| Bandera (*flag*) | Opción booleana: `--verbose` / `--no-verbose`. |
| Subcomando | Grupo de comandos anidados: `app.py usuarios crear`. |
| Callback | Función que se ejecuta antes de cualquier comando (opciones globales). |
| Contexto (`typer.Context`) | Objeto con información de la ejecución y datos compartidos entre comandos. |

**Regla general:** un parámetro **sin valor por defecto** es un **argumento** obligatorio; un parámetro **con valor por defecto** es una **opción**.

---

## 4. Primer programa

### 4.1 Con `typer.run` (un solo comando)

```python
import typer

def main(nombre: str, saludo: str = "Hola"):
    """Saluda a una persona."""
    print(f"{saludo}, {nombre}")

if __name__ == "__main__":
    typer.run(main)
```

```bash
python saludo.py Ana                     # Hola, Ana
python saludo.py Ana --saludo Buenas     # Buenas, Ana
python saludo.py --help
```

### 4.2 Con `typer.Typer` (recomendado)

```python
import typer

app = typer.Typer(help="Utilidades de saludo.")

@app.command()
def hola(nombre: str):
    """Saluda a una persona."""
    print(f"Hola, {nombre}")

@app.command()
def adios(nombre: str, formal: bool = False):
    """Se despide de una persona."""
    print(f"Hasta luego, {'Sr./Sra. ' if formal else ''}{nombre}")

if __name__ == "__main__":
    app()
```

```bash
python app.py hola Ana
python app.py adios Ana --formal
```

> Con **un solo** comando registrado, Typer no exige escribir su nombre. Con dos o más, el nombre del comando es obligatorio.

---

## 5. Argumentos

```python
from typing import Annotated
from pathlib import Path

@app.command()
def copiar(
    origen: Annotated[Path, typer.Argument(help="Archivo de origen", exists=True, dir_okay=False)],
    destino: Annotated[Path, typer.Argument(help="Carpeta de destino")] = Path("."),
):
    ...
```

| Parámetro de `typer.Argument` | Efecto |
|---|---|
| `help` | Texto de ayuda. |
| `metavar` | Nombre mostrado en la ayuda (`ARCHIVO`). |
| `envvar` | Leer el valor de una variable de entorno si no se pasa. |
| `show_default` | Mostrar el valor por defecto (`True`, `False` o texto). |
| `exists`, `file_okay`, `dir_okay`, `readable`, `writable`, `resolve_path` | Validaciones de `Path`. |
| `min`, `max`, `clamp` | Rangos numéricos. |
| `hidden` | Ocultar en la ayuda. |

**Número variable de argumentos:**

```python
@app.command()
def borrar(archivos: Annotated[list[Path], typer.Argument(exists=True)]):
    for a in archivos:
        a.unlink()
```

```bash
python app.py borrar a.txt b.txt c.txt
```

**Argumento opcional:**

```python
def saludar(nombre: Annotated[str | None, typer.Argument()] = None):
    print(f"Hola, {nombre or 'mundo'}")
```

---

## 6. Opciones

### 6.1 Declaración básica

```python
@app.command()
def exportar(
    salida: Annotated[Path, typer.Option("--salida", "-o", help="Archivo de salida")] = Path("datos.csv"),
    limite: Annotated[int, typer.Option(min=1, max=10_000, help="Máximo de filas")] = 100,
    verbose: Annotated[bool, typer.Option("--verbose", "-v")] = False,
):
    ...
```

```bash
python app.py exportar -o informe.csv --limite 500 -v
```

### 6.2 Nombres de las opciones

Por defecto, `mi_opcion` se convierte en `--mi-opcion`. Puede indicar nombres explícitos:

```python
usuario: Annotated[str, typer.Option("--user", "-u")] = "admin"
```

### 6.3 Opciones obligatorias

```python
token: Annotated[str, typer.Option(help="Token de acceso")]       # Sin valor por defecto → obligatoria
```

### 6.4 Banderas booleanas

```python
forzar: Annotated[bool, typer.Option("--forzar/--no-forzar")] = False   # Genera ambas formas
seco: Annotated[bool, typer.Option("--dry-run", is_flag=True)] = False  # Solo una forma
```

### 6.5 Opciones múltiples y contadores

```python
etiqueta: Annotated[list[str], typer.Option("--etiqueta", "-e")] = []
# app.py publicar -e python -e cli -e typer  → ['python', 'cli', 'typer']

verbosidad: Annotated[int, typer.Option("--verbose", "-v", count=True)] = 0
# -vvv → 3
```

### 6.6 Variables de entorno, prompts y contraseñas

```python
api_key: Annotated[str, typer.Option(envvar="MI_APP_API_KEY", help="Clave de la API")]
usuario: Annotated[str, typer.Option(prompt="Usuario")]                     # Pregunta si falta
clave: Annotated[str, typer.Option(prompt=True, hide_input=True, confirmation_prompt=True)]
```

| Parámetro de `typer.Option` | Efecto |
|---|---|
| `"--nombre", "-n"` | Nombres largo y corto. |
| `help`, `metavar`, `show_default`, `hidden` | Ayuda. |
| `envvar` | Variable(s) de entorno de respaldo. |
| `prompt`, `hide_input`, `confirmation_prompt` | Pedir el valor interactivamente. |
| `min`, `max`, `clamp` | Rango numérico. |
| `count` | Contar repeticiones (`-vvv`). |
| `case_sensitive` | Para `Enum`: distinguir mayúsculas. |
| `callback` | Función de validación o transformación. |
| `is_eager` | Procesar antes que las demás (p. ej. `--version`). |
| `rich_help_panel` | Agrupar en un panel de la ayuda. |
| `autocompletion` | Función de autocompletado. |

### 6.7 Validación personalizada con `callback`

```python
import re

def validar_email(valor: str) -> str:
    if not re.match(r"^[\w.+-]+@[\w-]+\.[\w.-]+$", valor):
        raise typer.BadParameter("Formato de email no válido.")
    return valor.lower()

@app.command()
def invitar(email: Annotated[str, typer.Option(callback=validar_email)]):
    ...
```

### 6.8 Opción `--version`

```python
__version__ = "1.2.0"

def mostrar_version(valor: bool):
    if valor:
        print(f"mi-app {__version__}")
        raise typer.Exit()

@app.callback()
def principal(
    version: Annotated[bool | None, typer.Option("--version", callback=mostrar_version, is_eager=True)] = None,
):
    """Mi aplicación."""
```

---

## 7. Tipos de datos admitidos

| Tipo | Ejemplo en la línea de comandos | Notas |
|---|---|---|
| `str` | `--nombre Ana` | |
| `int`, `float` | `--edad 30`, `--ratio 0.5` | Admite `min`/`max`. |
| `bool` | `--activo` / `--no-activo` | Bandera. |
| `Path` | `--config ./app.toml` | Validaciones `exists`, `dir_okay`, etc. |
| `Enum` | `--formato csv` | Limita los valores posibles. |
| `Literal["a", "b"]` | `--modo a` | Alternativa a `Enum`. |
| `datetime` | `--desde 2026-01-31` | `formats=["%d/%m/%Y"]` para otros formatos. |
| `uuid.UUID` | `--id 1b4e28ba-…` | |
| `list[T]` | `-e a -e b` / `a b c` | Opciones o argumentos múltiples. |
| `tuple[str, int]` | `--punto x 10` | Número fijo de valores. |
| `typer.FileText` | `--entrada datos.txt` | Archivo ya abierto (`-` = stdin). |

### 7.1 Valores cerrados con `Enum`

```python
from enum import Enum

class Formato(str, Enum):
    csv = "csv"
    json = "json"
    excel = "excel"

@app.command()
def exportar(formato: Formato = Formato.csv):
    print(f"Exportando en {formato.value}")
```

```bash
python app.py exportar --formato json
python app.py exportar --formato pdf
# Error: Invalid value for '--formato': 'pdf' is not one of 'csv', 'json', 'excel'.
```

### 7.2 Fechas

```python
from datetime import datetime

@app.command()
def informe(desde: Annotated[datetime, typer.Option(formats=["%Y-%m-%d", "%d/%m/%Y"])]):
    print(desde.date())
```

### 7.3 Archivos

```python
@app.command()
def contar(entrada: Annotated[typer.FileText, typer.Argument()] = "-"):
    """Cuenta las líneas de un archivo o de la entrada estándar."""
    print(sum(1 for _ in entrada))
```

```bash
python app.py contar datos.txt
cat datos.txt | python app.py contar
```

---

## 8. Varios comandos y subcomandos

### 8.1 Nombre y ayuda de los comandos

```python
@app.command("listar", help="Lista los usuarios.")
def listar_usuarios(): ...

@app.command(hidden=True)          # No aparece en la ayuda
def depurar(): ...

@app.command(deprecated=True)      # Marcado como obsoleto
def antiguo(): ...
```

### 8.2 Subcomandos con `add_typer`

```python
# usuarios.py
import typer
app = typer.Typer(help="Gestión de usuarios.")

@app.command()
def crear(nombre: str): ...

@app.command()
def eliminar(nombre: str): ...
```

```python
# main.py
import typer
import usuarios, informes

app = typer.Typer()
app.add_typer(usuarios.app, name="usuarios")
app.add_typer(informes.app, name="informes")

if __name__ == "__main__":
    app()
```

```bash
python main.py usuarios crear ana
python main.py informes mensual --mes 11
python main.py usuarios --help
```

Estructura recomendada:

```
mi_cli/
├── pyproject.toml
└── src/mi_cli/
    ├── __init__.py
    ├── main.py           # app principal y add_typer
    ├── usuarios.py       # subcomandos de usuarios
    ├── informes.py
    └── servicios/        # Lógica de negocio (sin Typer)
```

---

## 9. Callbacks y contexto

### 9.1 Opciones globales con `@app.callback`

```python
estado = {"verbose": False}

@app.callback()
def principal(verbose: Annotated[bool, typer.Option("--verbose", "-v")] = False):
    """Herramienta de administración."""
    estado["verbose"] = verbose
```

```bash
python app.py --verbose usuarios crear ana     # La opción global va antes del comando
```

### 9.2 Compartir datos con `typer.Context`

```python
@app.callback()
def principal(ctx: typer.Context, config: Path = Path("config.toml")):
    ctx.obj = cargar_configuracion(config)

@app.command()
def sincronizar(ctx: typer.Context):
    configuracion = ctx.obj
    ...
```

### 9.3 Ejecutar algo cuando no se indica comando

```python
@app.callback(invoke_without_command=True)
def principal(ctx: typer.Context):
    if ctx.invoked_subcommand is None:
        print("Use --help para ver los comandos disponibles.")
```

O, más simple: `typer.Typer(no_args_is_help=True)` muestra la ayuda si no hay argumentos.

---

## 10. Ayuda y documentación automática

El *docstring* del comando se convierte en su descripción:

```python
@app.command()
def importar(archivo: Path):
    """
    Importa clientes desde un archivo CSV.

    El archivo debe tener cabecera y usar ';' como separador.
    """
```

### 10.1 Formato enriquecido

```python
app = typer.Typer(rich_markup_mode="rich")       # o "markdown"

@app.command()
def borrar(nombre: str):
    """
    Borra un usuario [bold red]de forma permanente[/bold red].
    """
```

### 10.2 Paneles de ayuda

```python
@app.command()
def desplegar(
    entorno: str,
    replicas: Annotated[int, typer.Option(rich_help_panel="Escalado")] = 1,
    cpu: Annotated[float, typer.Option(rich_help_panel="Escalado")] = 0.5,
    dry_run: Annotated[bool, typer.Option(rich_help_panel="Seguridad")] = False,
):
    ...

@app.command(rich_help_panel="Administración")
def purgar(): ...
```

### 10.3 Epílogo y opciones de la aplicación

```python
app = typer.Typer(
    name="gestor",
    help="Gestor de inventario.",
    epilog="Documentación: https://ejemplo.com/docs",
    no_args_is_help=True,
    add_completion=True,
    context_settings={"help_option_names": ["-h", "--help"]},
    pretty_exceptions_show_locals=False,       # No mostrar variables locales en errores
)
```

### 10.4 Generar documentación en Markdown

```bash
typer mi_cli.main utils docs --name gestor --output DOCS.md
```

---

## 11. Interacción con el usuario

```python
nombre = typer.prompt("Nombre")
edad = typer.prompt("Edad", type=int, default=30)
clave = typer.prompt("Contraseña", hide_input=True, confirmation_prompt=True)

if typer.confirm("¿Eliminar todos los registros?", default=False):
    ...

typer.confirm("¿Continuar?", abort=True)      # Si responde 'no', aborta el programa
```

Barra de progreso básica:

```python
with typer.progressbar(elementos, label="Procesando") as barra:
    for elemento in barra:
        procesar(elemento)
```

Abrir un archivo o URL con la aplicación predeterminada:

```python
typer.launch("https://typer.tiangolo.com")
typer.launch("informe.pdf", locate=True)       # Abre la carpeta que lo contiene
```

> Para menús con flechas, selección múltiple o autocompletado, combine Typer con **questionary** o **InquirerPy**.

---

## 12. Salida, errores y códigos de salida

```python
typer.echo("Mensaje normal")
typer.echo("Error grave", err=True)                      # Salida de error (stderr)
typer.secho("✔ Completado", fg=typer.colors.GREEN, bold=True)
typer.secho("Aviso", fg="yellow")
```

Terminar el programa:

```python
raise typer.Exit()              # Código 0
raise typer.Exit(code=1)        # Código de error
raise typer.Abort()             # Muestra «Aborted!» y sale con código 1
raise typer.BadParameter("El valor no es válido")   # Error de uso (código 2)
```

| Código | Significado habitual |
|---|---|
| `0` | Éxito. |
| `1` | Error general. |
| `2` | Error de uso (argumento u opción no válidos). |
| `130` | Cancelado con Ctrl+C. |

Errores de negocio con mensaje claro:

```python
@app.command()
def eliminar(id_: int):
    try:
        servicio.eliminar(id_)
    except RegistroNoEncontrado:
        typer.secho(f"No existe el registro {id_}.", fg="red", err=True)
        raise typer.Exit(code=1)
```

---

## 13. Integración con Rich

Typer usa Rich para la ayuda y las excepciones. Puede usarlo también para la salida del programa:

```python
from rich.console import Console
from rich.table import Table

consola = Console()
consola_error = Console(stderr=True)

@app.command()
def listar():
    tabla = Table("ID", "Nombre", "Estado", title="Usuarios")
    for u in servicio.listar():
        tabla.add_row(str(u.id), u.nombre, "[green]activo[/]" if u.activo else "[red]inactivo[/]")
    consola.print(tabla)

@app.command()
def sincronizar():
    with consola.status("Sincronizando…"):
        servicio.sincronizar()
    consola.print("[bold green]✔ Sincronización completada[/]")
```

---

## 14. Autocompletado en la shell

Si la aplicación se instala como comando (ver sección 16):

```bash
mi-app --install-completion       # Bash, Zsh, Fish o PowerShell
mi-app --show-completion          # Muestra el script sin instalarlo
```

Autocompletado personalizado de valores:

```python
ENTORNOS = {"dev": "Desarrollo", "pre": "Preproducción", "pro": "Producción"}

def completar_entorno(incompleto: str):
    for nombre, ayuda in ENTORNOS.items():
        if nombre.startswith(incompleto):
            yield (nombre, ayuda)

@app.command()
def desplegar(entorno: Annotated[str, typer.Option(autocompletion=completar_entorno)]):
    ...
```

---

## 15. Pruebas con `CliRunner`

```python
from typer.testing import CliRunner
from mi_cli.main import app

runner = CliRunner()

def test_hola():
    resultado = runner.invoke(app, ["hola", "Ana"])
    assert resultado.exit_code == 0
    assert "Hola, Ana" in resultado.output

def test_opcion_invalida():
    resultado = runner.invoke(app, ["exportar", "--formato", "pdf"])
    assert resultado.exit_code == 2

def test_prompt():
    resultado = runner.invoke(app, ["crear"], input="Ana\ny\n")    # Respuestas simuladas
    assert resultado.exit_code == 0

def test_variable_entorno():
    resultado = runner.invoke(app, ["llamar"], env={"MI_APP_API_KEY": "abc"})
    assert resultado.exit_code == 0
```

> Si la excepción no es esperada, `resultado.exception` contiene el error original.

---

## 16. Empaquetar y distribuir el comando

En `pyproject.toml`:

```toml
[project]
name = "mi-cli"
version = "1.0.0"
dependencies = ["typer>=0.12"]

[project.scripts]
mi-app = "mi_cli.main:app"
```

```bash
pip install -e .          # Instalación en modo desarrollo
mi-app --help

pipx install .            # Instalación aislada para el usuario
uv tool install .         # Alternativa con uv
```

Ejecutar también con `python -m mi_cli`, añadiendo `src/mi_cli/__main__.py`:

```python
from mi_cli.main import app
app(prog_name="mi-app")
```

Ejecutar un script con el comando `typer` sin crear la aplicación:

```bash
typer script.py run --nombre Ana
```

---

## 17. Ejemplo práctico completo

**Gestor de tareas en línea de comandos** con almacenamiento en JSON: subcomandos, opciones globales, `Enum`, fechas, validación, confirmación, salida con Rich, códigos de salida y `--version`.

```python
"""
tareas.py — Gestor de tareas en línea de comandos.
Requiere: pip install typer
Uso: python tareas.py --help
"""
import json
from dataclasses import asdict, dataclass, field
from datetime import date, datetime
from enum import Enum
from pathlib import Path
from typing import Annotated

import typer
from rich.console import Console
from rich.table import Table

__version__ = "1.0.0"

app = typer.Typer(help="Gestor de tareas personal.", no_args_is_help=True, rich_markup_mode="rich")
consola = Console()
error = Console(stderr=True, style="bold red")


# ------------------------------------------------------------------ Modelo y almacenamiento
class Prioridad(str, Enum):
    baja = "baja"
    media = "media"
    alta = "alta"


COLORES = {"baja": "green", "media": "yellow", "alta": "red"}


@dataclass
class Tarea:
    id: int
    titulo: str
    prioridad: str = Prioridad.media.value
    vence: str | None = None
    hecha: bool = False
    etiquetas: list[str] = field(default_factory=list)


class Almacen:
    def __init__(self, ruta: Path):
        self.ruta = ruta
        self.tareas: list[Tarea] = []
        if ruta.exists():
            self.tareas = [Tarea(**t) for t in json.loads(ruta.read_text(encoding="utf-8"))]

    def guardar(self) -> None:
        self.ruta.write_text(json.dumps([asdict(t) for t in self.tareas], ensure_ascii=False, indent=2),
                             encoding="utf-8")

    def siguiente_id(self) -> int:
        return max((t.id for t in self.tareas), default=0) + 1

    def obtener(self, id_: int) -> Tarea:
        for t in self.tareas:
            if t.id == id_:
                return t
        error.print(f"No existe la tarea {id_}.")
        raise typer.Exit(code=1)


# ------------------------------------------------------------------ Opciones globales
def mostrar_version(valor: bool) -> None:
    if valor:
        consola.print(f"tareas {__version__}")
        raise typer.Exit()


@app.callback()
def principal(
    ctx: typer.Context,
    archivo: Annotated[Path, typer.Option("--archivo", "-f", envvar="TAREAS_ARCHIVO",
                                          help="Archivo JSON de tareas.")] = Path("tareas.json"),
    version: Annotated[bool | None, typer.Option("--version", callback=mostrar_version,
                                                 is_eager=True, help="Muestra la versión.")] = None,
):
    """Gestor de tareas personal con almacenamiento en [bold]JSON[/bold]."""
    ctx.obj = Almacen(archivo)


# ------------------------------------------------------------------ Comandos
@app.command()
def nueva(
    ctx: typer.Context,
    titulo: Annotated[str, typer.Argument(help="Descripción de la tarea.")],
    prioridad: Annotated[Prioridad, typer.Option("--prioridad", "-p", case_sensitive=False)] = Prioridad.media,
    vence: Annotated[datetime | None, typer.Option("--vence", "-v", formats=["%Y-%m-%d", "%d/%m/%Y"],
                                                   help="Fecha límite.")] = None,
    etiqueta: Annotated[list[str], typer.Option("--etiqueta", "-e", help="Puede repetirse.")] = [],
):
    """Crea una tarea nueva."""
    almacen: Almacen = ctx.obj
    titulo = titulo.strip()
    if not titulo:
        raise typer.BadParameter("El título no puede estar vacío.", param_hint="TITULO")
    if vence and vence.date() < date.today():
        raise typer.BadParameter("La fecha de vencimiento ya ha pasado.", param_hint="--vence")

    tarea = Tarea(id=almacen.siguiente_id(), titulo=titulo, prioridad=prioridad.value,
                  vence=vence.date().isoformat() if vence else None, etiquetas=etiqueta)
    almacen.tareas.append(tarea)
    almacen.guardar()
    consola.print(f"[green]✔[/] Tarea [bold]#{tarea.id}[/] creada.")


@app.command("listar")
def listar(
    ctx: typer.Context,
    todas: Annotated[bool, typer.Option("--todas", "-a", help="Incluir las completadas.")] = False,
    prioridad: Annotated[Prioridad | None, typer.Option("--prioridad", "-p")] = None,
    etiqueta: Annotated[str | None, typer.Option("--etiqueta", "-e")] = None,
):
    """Muestra las tareas pendientes (o todas)."""
    almacen: Almacen = ctx.obj
    tareas = [t for t in almacen.tareas
              if (todas or not t.hecha)
              and (prioridad is None or t.prioridad == prioridad.value)
              and (etiqueta is None or etiqueta in t.etiquetas)]
    if not tareas:
        consola.print("[dim]No hay tareas que mostrar.[/]")
        raise typer.Exit()

    orden = {"alta": 0, "media": 1, "baja": 2}
    tareas.sort(key=lambda t: (t.hecha, orden[t.prioridad], t.vence or "9999"))

    tabla = Table(title="Tareas", header_style="bold cyan")
    tabla.add_column("#", justify="right")
    tabla.add_column("Título")
    tabla.add_column("Prioridad")
    tabla.add_column("Vence")
    tabla.add_column("Etiquetas", style="dim")
    hoy = date.today().isoformat()
    for t in tareas:
        vence = t.vence or ""
        if t.vence and t.vence < hoy and not t.hecha:
            vence = f"[bold red]{t.vence}[/]"
        titulo = f"[strike dim]{t.titulo}[/]" if t.hecha else t.titulo
        tabla.add_row(str(t.id), titulo, f"[{COLORES[t.prioridad]}]{t.prioridad}[/]", vence,
                      ", ".join(t.etiquetas))
    consola.print(tabla)


@app.command()
def hecha(ctx: typer.Context, ids: Annotated[list[int], typer.Argument(help="Uno o varios identificadores.")]):
    """Marca tareas como completadas."""
    almacen: Almacen = ctx.obj
    for id_ in ids:
        almacen.obtener(id_).hecha = True
    almacen.guardar()
    consola.print(f"[green]✔[/] {len(ids)} tarea(s) completada(s).")


@app.command()
def borrar(
    ctx: typer.Context,
    id_: Annotated[int, typer.Argument(metavar="ID")],
    forzar: Annotated[bool, typer.Option("--forzar", "-y", help="No pedir confirmación.")] = False,
):
    """Elimina una tarea."""
    almacen: Almacen = ctx.obj
    tarea = almacen.obtener(id_)
    if not forzar:
        typer.confirm(f"¿Eliminar «{tarea.titulo}»?", abort=True)
    almacen.tareas.remove(tarea)
    almacen.guardar()
    consola.print(f"[yellow]Tarea #{id_} eliminada.[/]")


@app.command()
def limpiar(ctx: typer.Context):
    """Elimina todas las tareas completadas."""
    almacen: Almacen = ctx.obj
    completadas = [t for t in almacen.tareas if t.hecha]
    if not completadas:
        consola.print("[dim]No hay tareas completadas.[/]")
        raise typer.Exit()
    typer.confirm(f"¿Eliminar {len(completadas)} tarea(s) completada(s)?", abort=True)
    almacen.tareas = [t for t in almacen.tareas if not t.hecha]
    almacen.guardar()
    consola.print("[green]✔ Limpieza completada.[/]")


if __name__ == "__main__":
    app()
```

Uso:

```bash
python tareas.py nueva "Preparar informe trimestral" -p alta -v 2026-12-15 -e trabajo
python tareas.py nueva "Comprar regalo" -e personal
python tareas.py listar
python tareas.py listar -p alta -e trabajo
python tareas.py hecha 1 2
python tareas.py borrar 3 -y
python tareas.py -f otro.json listar --todas
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `Got unexpected extra argument` | Se pasó un argumento que no existe o un valor sin su opción. | Revise `--help`; las opciones necesitan `--nombre valor`. |
| `Missing argument 'NOMBRE'` | Parámetro sin valor por defecto (es obligatorio). | Páselo o asígnele un valor por defecto. |
| Con un solo comando no se reconoce su nombre | Typer omite el nombre si solo hay un comando. | Añada un `@app.callback()` o un segundo comando. |
| La opción global no se aplica | Se escribió después del subcomando. | Opciones del callback **antes** del comando: `app.py --verbose crear`. |
| Valor por defecto mutable compartido (`list = []`) | Comportamiento estándar de Python. | En Typer es seguro porque crea la lista en cada ejecución; en lógica propia, evite listas mutables. |
| `RuntimeError: Type not yet supported` | Tipo no admitido (p. ej. `dict`). | Use tipos simples y conviértalos en la función, o `click_type`/`parser`. |
| Caracteres extraños en la ayuda en Windows antiguo | Consola sin soporte Unicode/ANSI. | Use Windows Terminal o `typer-slim`. |
| `--install-completion` no funciona | El script no está instalado como comando. | Instálelo con `[project.scripts]` y `pip install -e .`. |
| Las excepciones muestran variables con datos sensibles | `pretty_exceptions_show_locals=True` (por defecto). | `typer.Typer(pretty_exceptions_show_locals=False)`. |

---

## 19. Buenas prácticas

- Use `Annotated` para declarar argumentos y opciones: el valor por defecto queda en la firma y el código es más legible.
- Escriba un *docstring* en cada comando y `help` en cada opción: es la documentación del usuario.
- Use `Enum`, `Literal`, `Path` con validaciones y rangos `min`/`max` en lugar de validar a mano.
- Separe la **lógica de negocio** (módulos sin Typer) de la **capa CLI**: facilita las pruebas y la reutilización.
- Use subcomandos (`add_typer`) cuando haya más de 5-6 comandos.
- Devuelva códigos de salida coherentes y escriba los errores en `stderr`.
- Ofrezca `--yes`/`--forzar` en acciones destructivas para poder usarlas en scripts.
- Permita configurar valores sensibles con `envvar` en lugar de escribirlos en la línea de comandos.
- Desactive `pretty_exceptions_show_locals` si maneja credenciales.
- Pruebe la CLI con `CliRunner`.

---

## 20. Referencia rápida

```python
app = typer.Typer(help=..., no_args_is_help=True, rich_markup_mode="rich")
@app.command("nombre") def f(arg: T, opt: T = valor): ...
@app.callback() def principal(ctx: typer.Context, ...): ctx.obj = ...
app.add_typer(sub_app, name="grupo")
app()  ·  typer.run(funcion)

Annotated[T, typer.Argument(help=, metavar=, envvar=, exists=True)]
Annotated[T, typer.Option("--largo", "-c", help=, envvar=, prompt=, hide_input=, min=, max=,
                          count=, callback=, is_eager=, rich_help_panel=, autocompletion=)]
Tipos: str int float bool Path Enum Literal datetime UUID list[T] tuple[...] typer.FileText

typer.echo / secho(texto, fg="green", err=True) · prompt / confirm(abort=True) · progressbar · launch
raise typer.Exit(code=1) · typer.Abort() · typer.BadParameter("mensaje")
from typer.testing import CliRunner; CliRunner().invoke(app, ["cmd", "--opt", "v"], input="…")
[project.scripts] mi-app = "paquete.main:app"
```

---

## 21. Ejercicios propuestos

1. Cree un comando `convertir` que transforme archivos CSV a JSON y viceversa, con el formato como `Enum` y validación de que el archivo de entrada existe.
2. Desarrolle una CLI con subcomandos `usuarios` (crear, listar, eliminar) e `informes` (diario, mensual), con una opción global `--config`.
3. Añada al gestor de tareas un comando `editar` que solo modifique los campos indicados y un comando `exportar` a CSV.
4. Implemente autocompletado personalizado para los identificadores de tarea.
5. Escriba una batería de pruebas con `CliRunner` que cubra los casos de éxito, los errores de validación y las confirmaciones.

---

## 22. Referencias

- Documentación oficial: https://typer.tiangolo.com/
- Tutorial: https://typer.tiangolo.com/tutorial/
- Repositorio: https://github.com/fastapi/typer
- Click (base de Typer): https://click.palletsprojects.com/
