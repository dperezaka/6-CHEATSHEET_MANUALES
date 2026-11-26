# Manual de uso: Click

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Construir CLIs robustas, componibles y extensibles
> **Tipo:** Librería de terceros (`pip install click`) · **Versión de referencia:** 8.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer comando](#4-primer-comando)
5. [Opciones](#5-opciones)
6. [Argumentos](#6-argumentos)
7. [Tipos de parámetros](#7-tipos-de-parámetros)
8. [Grupos y subcomandos](#8-grupos-y-subcomandos)
9. [Contexto y objetos compartidos](#9-contexto-y-objetos-compartidos)
10. [Validación y callbacks](#10-validación-y-callbacks)
11. [Variables de entorno y configuración](#11-variables-de-entorno-y-configuración)
12. [Utilidades de terminal](#12-utilidades-de-terminal)
13. [Errores y códigos de salida](#13-errores-y-códigos-de-salida)
14. [Extensibilidad: tipos, comandos y plugins](#14-extensibilidad-tipos-comandos-y-plugins)
15. [Comandos encadenados](#15-comandos-encadenados)
16. [Autocompletado en la shell](#16-autocompletado-en-la-shell)
17. [Pruebas con `CliRunner`](#17-pruebas-con-clirunner)
18. [Distribución](#18-distribución)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**Click** (*Command Line Interface Creation Kit*) es una librería para crear aplicaciones de línea de comandos mediante **decoradores**. Es la base de herramientas muy extendidas como Flask, Black o pip-tools, y también de **Typer**. Destaca por su capacidad para construir CLIs grandes, anidadas y extensibles con plugins.

**Ventajas:**
- Composición: grupos de comandos anidados a cualquier nivel.
- Validación y conversión de tipos integradas (`Path`, `File`, `Choice`, rangos, fechas…).
- Ayuda generada automáticamente.
- Utilidades de terminal: colores, prompts, barras de progreso, paginador, editor externo.
- Herramientas de prueba (`CliRunner`) integradas.
- Muy extensible: tipos, comandos, grupos y carga diferida de subcomandos.
- Estable, madura y muy documentada.

**Limitaciones:**
- Más verbosa que Typer: los parámetros se declaran dos veces (decorador y firma).
- No usa las anotaciones de tipo para inferir parámetros.
- La ayuda no tiene formato enriquecido sin complementos (`rich-click`).

| Si necesita… | Opción |
|---|---|
| CLI grande, extensible, con plugins | **Click** |
| CLI moderna con *type hints* y poco código | Typer |
| Sin dependencias externas | argparse |
| Exponer código existente rápidamente | Fire |
| Ayuda con formato Rich manteniendo Click | `rich-click` |

---

## 2. Instalación e importación

```bash
pip install click
```

```python
import click
```

Comprobar la versión:

```bash
python -c "from importlib.metadata import version; print(version('click'))"
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Comando (`@click.command`) | Función ejecutable desde la línea de comandos. |
| Grupo (`@click.group`) | Comando que contiene otros comandos (subcomandos). |
| Opción (`@click.option`) | Parámetro con nombre (`--salida`, `-o`). Opcional por defecto. |
| Argumento (`@click.argument`) | Parámetro posicional. Obligatorio por defecto. Sin texto de ayuda propio. |
| Tipo de parámetro (`ParamType`) | Convierte y valida el texto recibido (`click.INT`, `click.Path`…). |
| Contexto (`click.Context`) | Estado de la ejecución; transporta datos entre grupo y subcomandos (`ctx.obj`). |
| Callback | Función que valida o transforma un parámetro. |
| `CliRunner` | Ejecuta comandos en pruebas sin abrir un proceso. |

**Orden de los decoradores:** el decorador `@click.command()` va **arriba**; los parámetros, debajo. Los parámetros aparecen en la ayuda en el mismo orden en que se declaran.

---

## 4. Primer comando

```python
import click

@click.command()
@click.option("--veces", "-n", default=1, show_default=True, help="Número de saludos.")
@click.argument("nombre")
def hola(veces, nombre):
    """Saluda a NOMBRE varias veces."""
    for _ in range(veces):
        click.echo(f"Hola, {nombre}")

if __name__ == "__main__":
    hola()
```

```bash
python hola.py Ana -n 3
python hola.py --help
```

```
Usage: hola.py [OPTIONS] NOMBRE

  Saluda a NOMBRE varias veces.

Options:
  -n, --veces INTEGER  Número de saludos.  [default: 1]
  --help               Show this message and exit.
```

> El tipo de la opción se infiere del valor por defecto (`default=1` → `INTEGER`).

---

## 5. Opciones

### 5.1 Nombres

```python
@click.option("--salida", "-o")               # Parámetro: salida
@click.option("--max-filas")                  # Parámetro: max_filas
@click.option("-u", "--user", "usuario")      # Parámetro con nombre explícito: usuario
```

### 5.2 Obligatorias y con valor por defecto

```python
@click.option("--token", required=True, help="Token de acceso.")
@click.option("--puerto", default=8080, show_default=True, type=int)
@click.option("--fecha", default=lambda: date.today().isoformat(), show_default="hoy")
```

### 5.3 Banderas

```python
@click.option("--verbose", "-v", is_flag=True, help="Modo detallado.")
@click.option("--color/--no-color", default=True, help="Salida con color.")
@click.option("--json", "formato", flag_value="json", help="Salida JSON.")
@click.option("--csv", "formato", flag_value="csv", default=True, help="Salida CSV.")
```

### 5.4 Múltiples valores

```python
@click.option("--etiqueta", "-e", multiple=True)              # -e a -e b → ('a', 'b')
@click.option("--punto", nargs=2, type=float)                  # --punto 1.5 2.0 → (1.5, 2.0)
@click.option("--verbose", "-v", count=True)                   # -vvv → 3
```

### 5.5 Prompts y contraseñas

```python
@click.option("--nombre", prompt="Nombre del usuario")
@click.option("--clave", prompt=True, hide_input=True, confirmation_prompt=True)
@click.password_option()                                       # Atajo equivalente
@click.confirmation_option(prompt="¿Eliminar la base de datos?")
```

### 5.6 Opciones predefinidas

```python
@click.version_option("1.2.0", prog_name="gestor")              # --version
@click.help_option("-h", "--help")                              # Añade -h
```

O para toda la aplicación:

```python
CONTEXTO = {"help_option_names": ["-h", "--help"], "max_content_width": 100}

@click.group(context_settings=CONTEXTO)
def cli(): ...
```

| Parámetro de `click.option` | Efecto |
|---|---|
| `default`, `show_default` | Valor por defecto y si se muestra en la ayuda. |
| `required` | Obligatoria. |
| `type` | Tipo de dato (sección 7). |
| `help`, `metavar`, `hidden` | Ayuda. |
| `is_flag`, `flag_value` | Banderas. |
| `multiple`, `nargs`, `count` | Varios valores. |
| `prompt`, `hide_input`, `confirmation_prompt` | Pedir el valor interactivamente. |
| `envvar`, `show_envvar` | Variable de entorno. |
| `callback`, `is_eager`, `expose_value` | Validación y procesamiento anticipado. |
| `shell_complete` | Autocompletado personalizado. |

---

## 6. Argumentos

```python
@click.command()
@click.argument("origen", type=click.Path(exists=True, dir_okay=False))
@click.argument("destino", type=click.Path(file_okay=False), default=".")
def copiar(origen, destino):
    """Copia ORIGEN en la carpeta DESTINO."""
```

**Número variable:**

```python
@click.command()
@click.argument("archivos", nargs=-1, type=click.Path(exists=True), required=True)
def comprimir(archivos):
    for a in archivos:
        ...
```

```bash
python app.py comprimir *.log
```

> Los argumentos **no admiten** `help=`: documéntelos en el *docstring* del comando, en mayúsculas.

**Separar opciones de argumentos que empiezan por guion:**

```bash
python app.py buscar -- -texto-con-guion
```

---

## 7. Tipos de parámetros

| Tipo | Ejemplo | Notas |
|---|---|---|
| `str`, `int`, `float`, `bool` | `type=int` | También `click.STRING`, `click.INT`… |
| `click.Choice([...])` | `--formato csv` | Valores cerrados. Acepta `Enum` (8.2+). |
| `click.IntRange(1, 10)` | `--nivel 5` | `clamp=True` ajusta al límite en lugar de fallar. |
| `click.FloatRange(0, 1)` | `--ratio 0.3` | |
| `click.Path(...)` | `--config app.toml` | `exists`, `file_okay`, `dir_okay`, `writable`, `resolve_path`, `path_type=Path`. |
| `click.File("r")` | `--entrada datos.txt` | Archivo abierto automáticamente; `-` = stdin/stdout. |
| `click.DateTime(formats)` | `--desde 2026-01-31` | Devuelve `datetime`. |
| `click.UUID` | `--id …` | |
| `click.Tuple([str, int])` | `--par x 3` | Tipos distintos por posición. |

### 7.1 `Choice`

```python
@click.option("--formato", type=click.Choice(["csv", "json", "xlsx"], case_sensitive=False),
              default="csv", show_default=True)
```

Con `Enum`:

```python
import enum

class Entorno(enum.Enum):
    DESARROLLO = "dev"
    PRODUCCION = "pro"

@click.option("--entorno", type=click.Choice(Entorno, case_sensitive=False))
def desplegar(entorno: Entorno): ...
```

### 7.2 `Path` y `File`

```python
from pathlib import Path

@click.option("--config", type=click.Path(exists=True, dir_okay=False, path_type=Path))
@click.option("--salida", type=click.File("w", encoding="utf-8"), default="-")
def procesar(config: Path, salida):
    salida.write("resultado\n")          # Escribe en el archivo o en stdout
```

> `click.File` abre el archivo de forma **diferida** cuando es de escritura: no se crea si el comando falla antes de usarlo (`lazy=True`).

---

## 8. Grupos y subcomandos

### 8.1 Grupo básico

```python
@click.group()
def cli():
    """Herramienta de administración."""

@cli.command()
def iniciar():
    """Inicia el servicio."""

@cli.command()
@click.option("--forzar", is_flag=True)
def detener(forzar):
    """Detiene el servicio."""

if __name__ == "__main__":
    cli()
```

```bash
python admin.py iniciar
python admin.py detener --forzar
```

### 8.2 Grupos anidados

```python
@cli.group()
def usuarios():
    """Gestión de usuarios."""

@usuarios.command("crear")
@click.argument("nombre")
def usuarios_crear(nombre): ...

@usuarios.command("listar")
def usuarios_listar(): ...
```

```bash
python admin.py usuarios crear ana
```

### 8.3 Comandos en varios módulos

```python
# comandos/bd.py
import click

@click.group()
def bd():
    """Operaciones de base de datos."""

@bd.command()
def migrar(): ...
```

```python
# main.py
import click
from comandos.bd import bd
from comandos.usuarios import usuarios

@click.group()
def cli(): ...

cli.add_command(bd)
cli.add_command(usuarios)
cli.add_command(otra_funcion, name="alias")
```

### 8.4 Ejecutar el grupo sin subcomando

```python
@click.group(invoke_without_command=True)
@click.pass_context
def cli(ctx):
    if ctx.invoked_subcommand is None:
        click.echo(ctx.get_help())
```

---

## 9. Contexto y objetos compartidos

### 9.1 `pass_context` y `ctx.obj`

```python
@click.group()
@click.option("--config", type=click.Path(path_type=Path), default="config.toml")
@click.option("--verbose", "-v", is_flag=True)
@click.pass_context
def cli(ctx, config, verbose):
    ctx.ensure_object(dict)
    ctx.obj["config"] = cargar(config)
    ctx.obj["verbose"] = verbose

@cli.command()
@click.pass_context
def sincronizar(ctx):
    if ctx.obj["verbose"]:
        click.echo("Sincronizando…")
```

### 9.2 `pass_obj`

```python
@cli.command()
@click.pass_obj
def estado(obj):
    click.echo(obj["config"])
```

### 9.3 Decorador propio con `make_pass_decorator` (recomendado)

```python
class Aplicacion:
    def __init__(self, config: Path, verbose: bool):
        self.config = config
        self.verbose = verbose

pass_app = click.make_pass_decorator(Aplicacion)

@click.group()
@click.option("--config", default="config.toml", type=click.Path(path_type=Path))
@click.option("--verbose", "-v", is_flag=True)
@click.pass_context
def cli(ctx, config, verbose):
    ctx.obj = Aplicacion(config, verbose)

@cli.command()
@pass_app
def estado(app: Aplicacion):
    click.echo(f"Configuración: {app.config}")
```

### 9.4 Invocar otro comando

```python
@cli.command()
@click.pass_context
def reiniciar(ctx):
    ctx.invoke(detener, forzar=True)
    ctx.invoke(iniciar)
```

### 9.5 Liberar recursos al terminar

```python
@click.group()
@click.pass_context
def cli(ctx):
    conexion = conectar()
    ctx.call_on_close(conexion.close)
    # o bien: ctx.with_resource(conectar())  → gestor de contexto
    ctx.obj = conexion
```

---

## 10. Validación y callbacks

### 10.1 Callback de validación

```python
import re

def validar_email(ctx, param, valor):
    if valor is None:
        return valor
    if not re.match(r"^[\w.+-]+@[\w-]+\.[\w.-]+$", valor):
        raise click.BadParameter("formato no válido.")
    return valor.lower()

@click.command()
@click.option("--email", callback=validar_email, required=True)
def invitar(email): ...
```

```
Error: Invalid value for '--email': formato no válido.
```

### 10.2 Opciones mutuamente excluyentes

Click no las incluye de serie; se resuelven en el propio comando:

```python
@click.command()
@click.option("--id", "id_", type=int)
@click.option("--nombre")
def buscar(id_, nombre):
    if (id_ is None) == (nombre is None):
        raise click.UsageError("Indique exactamente una opción: --id o --nombre.")
```

> Para grupos de opciones y exclusiones declarativas existe el paquete `click-option-group`.

### 10.3 Opción procesada antes que las demás (`is_eager`)

```python
def listar_plantillas(ctx, param, valor):
    if not valor or ctx.resilient_parsing:
        return
    click.echo("basica\nempresa\ninforme")
    ctx.exit()

@click.option("--listar-plantillas", is_flag=True, callback=listar_plantillas,
              expose_value=False, is_eager=True)
```

---

## 11. Variables de entorno y configuración

### 11.1 `envvar` por opción

```python
@click.option("--api-key", envvar="MIAPP_API_KEY", required=True, show_envvar=True)
```

### 11.2 Prefijo automático para todas las opciones

```python
if __name__ == "__main__":
    cli(auto_envvar_prefix="MIAPP")
# --puerto del comando servir → MIAPP_SERVIR_PUERTO
```

### 11.3 Valores por defecto desde un archivo (`default_map`)

```python
import tomllib

def cargar_config(ctx, param, ruta):
    if ruta and Path(ruta).exists():
        with open(ruta, "rb") as f:
            ctx.default_map = tomllib.load(f)
    return ruta

@click.group()
@click.option("--config", default="miapp.toml", callback=cargar_config, is_eager=True, expose_value=False)
def cli(): ...
```

```toml
# miapp.toml
[servir]
puerto = 9000
host = "0.0.0.0"
```

**Prioridad:** línea de comandos > variable de entorno > `default_map` > `default`.

### 11.4 Carpeta de configuración del usuario

```python
carpeta = Path(click.get_app_dir("MiApp"))
# Windows: %APPDATA%\MiApp · macOS: ~/Library/Application Support/MiApp · Linux: ~/.config/miapp
```

---

## 12. Utilidades de terminal

### 12.1 Salida y colores

```python
click.echo("Texto")                                  # Seguro con Unicode, bytes y tuberías
click.echo("Error", err=True)                        # stderr
click.secho("✔ Completado", fg="green", bold=True)
click.echo(click.style("Aviso", fg="yellow") + " revise la configuración")
click.echo("sin salto", nl=False)
```

Colores: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, sus variantes `bright_*`, códigos 0-255 o tuplas RGB. Estilos: `bold`, `dim`, `underline`, `italic`, `blink`, `reverse`, `strikethrough`.

> Click elimina los códigos de color automáticamente si la salida no es una terminal.

### 12.2 Entrada

```python
nombre = click.prompt("Nombre")
edad = click.prompt("Edad", type=click.IntRange(0, 120), default=30)
formato = click.prompt("Formato", type=click.Choice(["csv", "json"]), default="csv")

if click.confirm("¿Continuar?", default=True):
    ...
click.confirm("¿Borrar?", abort=True)                 # Aborta si responde 'no'

tecla = click.getchar()                               # Lee una sola tecla
click.pause("Pulse una tecla para continuar…")
```

### 12.3 Barra de progreso

```python
with click.progressbar(archivos, label="Procesando", show_pos=True,
                       item_show_func=lambda a: a.name if a else "") as barra:
    for archivo in barra:
        procesar(archivo)

with click.progressbar(length=total_bytes, label="Descargando") as barra:
    for bloque in descargar():
        barra.update(len(bloque))
```

### 12.4 Paginador, editor, archivos y pantalla

```python
click.echo_via_pager("\n".join(lineas))               # Como 'less'
texto = click.edit("# Escriba el mensaje\n")          # Abre $EDITOR; None si no guarda
click.launch("https://example.com")                   # Navegador / aplicación predeterminada
click.launch("informe.pdf", locate=True)              # Abrir la carpeta
click.clear()                                         # Limpiar la pantalla
ancho, alto = shutil.get_terminal_size()
```

### 12.5 Abrir archivos de forma segura

```python
with click.open_file(ruta, "w", atomic=True) as f:    # Escribe en un temporal y lo renombra
    f.write(contenido)
```

---

## 13. Errores y códigos de salida

| Excepción | Uso | Código |
|---|---|---|
| `click.ClickException("msg")` | Error general con mensaje. | 1 |
| `click.UsageError("msg")` | Uso incorrecto; muestra el *usage*. | 2 |
| `click.BadParameter("msg")` | Valor de parámetro no válido. | 2 |
| `click.FileError(ruta, hint)` | Error al abrir un archivo. | 1 |
| `click.Abort()` | Cancelación; muestra «Aborted!». | 1 |
| `ctx.exit(código)` / `sys.exit()` | Salida explícita. | El indicado |

Excepción propia:

```python
class ErrorAplicacion(click.ClickException):
    exit_code = 3

    def show(self, file=None):
        click.secho(f"✖ {self.message}", fg="red", err=True)
```

Errores de negocio:

```python
try:
    servicio.eliminar(id_)
except NoEncontrado:
    raise click.ClickException(f"No existe el registro {id_}.")
```

---

## 14. Extensibilidad: tipos, comandos y plugins

### 14.1 Tipo de parámetro propio

```python
class TipoTamano(click.ParamType):
    """Acepta tamaños como 10K, 5M o 2G y devuelve bytes."""
    name = "tamaño"
    UNIDADES = {"K": 1024, "M": 1024**2, "G": 1024**3}

    def convert(self, valor, param, ctx):
        if isinstance(valor, int):
            return valor
        try:
            unidad = valor[-1].upper()
            if unidad in self.UNIDADES:
                return int(float(valor[:-1]) * self.UNIDADES[unidad])
            return int(valor)
        except (ValueError, IndexError):
            self.fail(f"{valor!r} no es un tamaño válido (ej.: 10K, 5M, 2G).", param, ctx)

TAMANO = TipoTamano()

@click.option("--max", "maximo", type=TAMANO, default="10M")
```

### 14.2 Grupo con alias y orden personalizado

```python
class GrupoConAlias(click.Group):
    ALIAS = {"ls": "listar", "rm": "eliminar"}

    def get_command(self, ctx, nombre):
        return super().get_command(ctx, self.ALIAS.get(nombre, nombre))

    def list_commands(self, ctx):
        return list(self.commands)          # Orden de declaración, no alfabético

@click.group(cls=GrupoConAlias)
def cli(): ...
```

### 14.3 Carga diferida de subcomandos

Para CLIs grandes, importa cada subcomando solo cuando se usa (arranque más rápido):

```python
import importlib

class GrupoDiferido(click.Group):
    def __init__(self, *args, comandos_diferidos: dict[str, str], **kw):
        super().__init__(*args, **kw)
        self.diferidos = comandos_diferidos          # {"bd": "miapp.comandos.bd:bd"}

    def list_commands(self, ctx):
        return sorted([*super().list_commands(ctx), *self.diferidos])

    def get_command(self, ctx, nombre):
        if nombre in self.diferidos:
            modulo, attr = self.diferidos[nombre].split(":")
            return getattr(importlib.import_module(modulo), attr)
        return super().get_command(ctx, nombre)

@click.group(cls=GrupoDiferido, comandos_diferidos={"bd": "miapp.comandos.bd:bd"})
def cli(): ...
```

### 14.4 Plugins mediante *entry points*

```toml
# pyproject.toml del plugin
[project.entry-points."miapp.plugins"]
informes = "miapp_informes:grupo"
```

```python
from importlib.metadata import entry_points

for ep in entry_points(group="miapp.plugins"):
    cli.add_command(ep.load(), name=ep.name)
```

---

## 15. Comandos encadenados

Permite ejecutar varios subcomandos en una sola llamada, al estilo de una tubería:

```python
@click.group(chain=True)
def imagen():
    """Procesa imágenes encadenando operaciones."""

@imagen.command()
@click.argument("ruta")
def abrir(ruta):
    return lambda _: Image.open(ruta)

@imagen.command()
@click.option("--ancho", type=int, required=True)
def redimensionar(ancho):
    return lambda img: img.resize((ancho, int(img.height * ancho / img.width)))

@imagen.command()
@click.argument("ruta")
def guardar(ruta):
    return lambda img: img.save(ruta) or img

@imagen.result_callback()
def ejecutar(operaciones):
    resultado = None
    for op in operaciones:
        resultado = op(resultado)
```

```bash
python imagen.py abrir foto.jpg redimensionar --ancho 800 guardar mini.jpg
```

---

## 16. Autocompletado en la shell

Con el programa instalado como comando `miapp`:

```bash
# Bash (~/.bashrc)
eval "$(_MIAPP_COMPLETE=bash_source miapp)"
# Zsh (~/.zshrc)
eval "$(_MIAPP_COMPLETE=zsh_source miapp)"
# Fish (~/.config/fish/completions/miapp.fish)
_MIAPP_COMPLETE=fish_source miapp | source
```

Autocompletado de valores personalizados:

```python
from click.shell_completion import CompletionItem

def completar_entorno(ctx, param, incompleto):
    entornos = {"dev": "Desarrollo", "pre": "Preproducción", "pro": "Producción"}
    return [CompletionItem(k, help=v) for k, v in entornos.items() if k.startswith(incompleto)]

@click.option("--entorno", shell_complete=completar_entorno)
```

---

## 17. Pruebas con `CliRunner`

```python
from click.testing import CliRunner
from miapp.main import cli

def test_hola():
    runner = CliRunner()
    resultado = runner.invoke(cli, ["hola", "Ana", "-n", "2"])
    assert resultado.exit_code == 0
    assert resultado.output.count("Hola, Ana") == 2

def test_prompt_y_entorno():
    runner = CliRunner()
    resultado = runner.invoke(cli, ["crear"], input="Ana\ny\n", env={"MIAPP_TOKEN": "x"})
    assert resultado.exit_code == 0

def test_archivos():
    runner = CliRunner()
    with runner.isolated_filesystem():             # Carpeta temporal
        Path("datos.csv").write_text("a;b\n1;2\n")
        resultado = runner.invoke(cli, ["importar", "datos.csv"])
        assert resultado.exit_code == 0

def test_error_inesperado():
    resultado = CliRunner().invoke(cli, ["fallar"], catch_exceptions=False)   # Propaga la excepción
```

| Atributo de `Result` | Contenido |
|---|---|
| `exit_code` | Código de salida. |
| `output` | Salida (stdout y stderr mezclados). |
| `stdout`, `stderr` | Salidas por separado. |
| `exception` | Excepción no controlada, si la hubo. |

---

## 18. Distribución

```toml
[project]
name = "miapp"
version = "1.0.0"
dependencies = ["click>=8.1"]

[project.scripts]
miapp = "miapp.main:cli"
```

```bash
pip install -e .
miapp --help
pipx install .            # Instalación aislada
```

Ejecutar con `python -m miapp` (`src/miapp/__main__.py`):

```python
from miapp.main import cli
cli(prog_name="miapp")
```

---

## 19. Ejemplo práctico completo

**Herramienta de copias de seguridad** con grupos anidados, objeto de aplicación compartido, configuración TOML con `default_map`, tipo propio, validación, barra de progreso, confirmaciones, colores y errores personalizados.

```python
"""
respaldo.py — Copias de seguridad comprimidas de carpetas.
Requiere: pip install click
Uso: python respaldo.py --help
"""
import fnmatch
import json
import tomllib
import zipfile
from dataclasses import dataclass
from datetime import datetime
from pathlib import Path

import click

__version__ = "1.0.0"
CONTEXTO = {"help_option_names": ["-h", "--help"], "max_content_width": 100}


# ------------------------------------------------------------------ Tipos y errores
class TipoTamano(click.ParamType):
    name = "tamaño"
    UNIDADES = {"K": 1024, "M": 1024**2, "G": 1024**3}

    def convert(self, valor, param, ctx):
        if isinstance(valor, int):
            return valor
        try:
            u = valor[-1].upper()
            return int(float(valor[:-1]) * self.UNIDADES[u]) if u in self.UNIDADES else int(valor)
        except (ValueError, IndexError):
            self.fail(f"{valor!r} no es un tamaño válido (ej.: 500K, 10M).", param, ctx)


class ErrorRespaldo(click.ClickException):
    def show(self, file=None):
        click.secho(f"✖ {self.message}", fg="red", err=True)


def formato_tamano(n: int) -> str:
    for unidad in ("B", "KB", "MB", "GB"):
        if n < 1024:
            return f"{n:.1f} {unidad}" if unidad != "B" else f"{n} B"
        n /= 1024
    return f"{n:.1f} TB"


# ------------------------------------------------------------------ Aplicación
@dataclass
class Aplicacion:
    destino: Path
    verbose: bool

    @property
    def indice(self) -> Path:
        return self.destino / "indice.json"

    def leer_indice(self) -> list[dict]:
        return json.loads(self.indice.read_text(encoding="utf-8")) if self.indice.exists() else []

    def escribir_indice(self, datos: list[dict]) -> None:
        self.indice.write_text(json.dumps(datos, ensure_ascii=False, indent=2), encoding="utf-8")

    def log(self, mensaje: str) -> None:
        if self.verbose:
            click.secho(mensaje, dim=True)


pass_app = click.make_pass_decorator(Aplicacion)


def cargar_config(ctx: click.Context, param, ruta: str | None):
    if ruta and Path(ruta).is_file():
        with open(ruta, "rb") as f:
            ctx.default_map = tomllib.load(f)
    return ruta


@click.group(context_settings=CONTEXTO)
@click.option("--config", type=click.Path(dir_okay=False), default="respaldo.toml",
              callback=cargar_config, is_eager=True, expose_value=False,
              help="Archivo de configuración TOML.", show_default=True)
@click.option("--destino", "-d", type=click.Path(file_okay=False, path_type=Path),
              default=Path("respaldos"), envvar="RESPALDO_DESTINO", show_default=True,
              help="Carpeta donde se guardan las copias.")
@click.option("--verbose", "-v", is_flag=True, help="Muestra información detallada.")
@click.version_option(__version__, prog_name="respaldo")
@click.pass_context
def cli(ctx: click.Context, destino: Path, verbose: bool):
    """Crea, lista, restaura y purga copias de seguridad comprimidas."""
    destino.mkdir(parents=True, exist_ok=True)
    ctx.obj = Aplicacion(destino, verbose)


# ------------------------------------------------------------------ Comandos
@cli.command()
@click.argument("origen", type=click.Path(exists=True, file_okay=False, path_type=Path))
@click.option("--nombre", "-n", help="Nombre de la copia (por defecto, el de la carpeta).")
@click.option("--excluir", "-x", multiple=True, metavar="PATRÓN",
              help="Patrón de archivos a excluir (repetible), p. ej. '*.tmp'.")
@click.option("--max-archivo", type=TipoTamano(), default="100M", show_default=True,
              help="Omitir archivos mayores que este tamaño.")
@pass_app
def crear(app: Aplicacion, origen: Path, nombre: str | None, excluir: tuple[str, ...], max_archivo: int):
    """Crea una copia comprimida de la carpeta ORIGEN."""
    nombre = nombre or origen.resolve().name
    archivos, omitidos = [], 0
    for ruta in origen.rglob("*"):
        if not ruta.is_file():
            continue
        relativa = ruta.relative_to(origen).as_posix()
        if any(fnmatch.fnmatch(relativa, p) or fnmatch.fnmatch(ruta.name, p) for p in excluir):
            app.log(f"  excluido: {relativa}")
            continue
        if ruta.stat().st_size > max_archivo:
            omitidos += 1
            app.log(f"  demasiado grande: {relativa}")
            continue
        archivos.append(ruta)

    if not archivos:
        raise ErrorRespaldo("No hay archivos que copiar con los filtros indicados.")

    marca = datetime.now().strftime("%Y%m%d-%H%M%S")
    zip_ruta = app.destino / f"{nombre}-{marca}.zip"
    with zipfile.ZipFile(zip_ruta, "w", zipfile.ZIP_DEFLATED) as zf, \
            click.progressbar(archivos, label=f"Comprimiendo {nombre}", show_pos=True) as barra:
        for ruta in barra:
            zf.write(ruta, ruta.relative_to(origen))

    indice = app.leer_indice()
    indice.append({"archivo": zip_ruta.name, "nombre": nombre, "origen": str(origen.resolve()),
                   "fecha": datetime.now().isoformat(timespec="seconds"),
                   "archivos": len(archivos), "tamano": zip_ruta.stat().st_size})
    app.escribir_indice(indice)

    click.secho(f"✔ Copia creada: {zip_ruta.name} ({formato_tamano(zip_ruta.stat().st_size)})",
                fg="green", bold=True)
    if omitidos:
        click.secho(f"  {omitidos} archivo(s) omitido(s) por tamaño.", fg="yellow")


@cli.command("listar")
@click.option("--nombre", "-n", help="Filtrar por nombre de copia.")
@pass_app
def listar(app: Aplicacion, nombre: str | None):
    """Muestra las copias disponibles."""
    copias = [c for c in app.leer_indice() if nombre is None or c["nombre"] == nombre]
    if not copias:
        click.secho("No hay copias registradas.", dim=True)
        return
    click.secho(f"{'#':>3}  {'Fecha':<19}  {'Nombre':<20}  {'Archivos':>8}  {'Tamaño':>10}", bold=True)
    for i, c in enumerate(copias, 1):
        click.echo(f"{i:>3}  {c['fecha']:<19}  {c['nombre']:<20}  {c['archivos']:>8}  "
                   f"{formato_tamano(c['tamano']):>10}")


@cli.command()
@click.argument("numero", type=click.IntRange(min=1))
@click.argument("destino", type=click.Path(file_okay=False, path_type=Path))
@click.option("--sobrescribir", is_flag=True, help="Permite restaurar en una carpeta con contenido.")
@pass_app
def restaurar(app: Aplicacion, numero: int, destino: Path, sobrescribir: bool):
    """Restaura la copia NUMERO (ver 'listar') en la carpeta DESTINO."""
    copias = app.leer_indice()
    if numero > len(copias):
        raise click.BadParameter(f"solo hay {len(copias)} copia(s).", param_hint="NUMERO")
    copia = copias[numero - 1]
    if destino.exists() and any(destino.iterdir()) and not sobrescribir:
        raise ErrorRespaldo(f"La carpeta {destino} no está vacía. Use --sobrescribir.")
    if sobrescribir:
        click.confirm(f"Se sobrescribirán archivos en {destino}. ¿Continuar?", abort=True)

    with zipfile.ZipFile(app.destino / copia["archivo"]) as zf:
        miembros = zf.infolist()
        with click.progressbar(miembros, label="Restaurando") as barra:
            for m in barra:
                zf.extract(m, destino)
    click.secho(f"✔ {len(miembros)} archivo(s) restaurado(s) en {destino}", fg="green")


@cli.command()
@click.option("--conservar", "-k", type=click.IntRange(min=1), default=5, show_default=True,
              help="Número de copias más recientes a conservar por nombre.")
@click.option("--yes", "-y", is_flag=True, help="No pedir confirmación.")
@pass_app
def purgar(app: Aplicacion, conservar: int, yes: bool):
    """Elimina las copias antiguas y conserva las más recientes."""
    indice = sorted(app.leer_indice(), key=lambda c: c["fecha"], reverse=True)
    vistos: dict[str, int] = {}
    conservadas, eliminar = [], []
    for c in indice:
        vistos[c["nombre"]] = vistos.get(c["nombre"], 0) + 1
        (conservadas if vistos[c["nombre"]] <= conservar else eliminar).append(c)

    if not eliminar:
        click.secho("No hay copias que purgar.", dim=True)
        return
    for c in eliminar:
        click.echo(f"  - {c['archivo']}")
    if not yes:
        click.confirm(f"¿Eliminar {len(eliminar)} copia(s)?", abort=True)
    for c in eliminar:
        (app.destino / c["archivo"]).unlink(missing_ok=True)
    app.escribir_indice(sorted(conservadas, key=lambda c: c["fecha"]))
    click.secho(f"✔ {len(eliminar)} copia(s) eliminada(s).", fg="green")


if __name__ == "__main__":
    cli(auto_envvar_prefix="RESPALDO")
```

Ejemplo de `respaldo.toml`:

```toml
destino = "D:/Respaldos"

[crear]
excluir = ["*.tmp", "node_modules/*", ".git/*"]
max_archivo = "50M"

[purgar]
conservar = 10
```

Uso:

```bash
python respaldo.py crear ./proyecto -x "*.log" -x ".venv/*"
python respaldo.py listar
python respaldo.py restaurar 1 ./restaurado
python respaldo.py purgar -k 3 -y
python respaldo.py -v --destino /tmp/copias crear ./documentos --max-archivo 5M
```

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: f() got an unexpected keyword argument 'x'` | El nombre del parámetro de la función no coincide con el de la opción. | `--mi-opcion` → `mi_opcion`, o indique el nombre explícito como último argumento. |
| `TypeError: f() missing 1 required positional argument` | Falta un decorador o `pass_context`. | Revise que cada parámetro tenga su decorador. |
| `TypeError: __init__() got an unexpected keyword argument 'help'` | `help` en un `@click.argument`. | Documente los argumentos en el *docstring*. |
| `No such command 'x'` | Subcomando no registrado o con otro nombre (`_` → `-`). | `cli.add_command(...)`; Click convierte `mi_cmd` en `mi-cmd`. |
| `AttributeError: 'NoneType' object has no attribute…` en `ctx.obj` | El grupo no inicializó `ctx.obj`. | `ctx.ensure_object(dict)` o asignar `ctx.obj`. |
| La opción global no se reconoce | Escrita después del subcomando. | Opciones del grupo antes del subcomando. |
| `Error: Got unexpected extra argument` | Argumento sobrante o valor de opción con guion inicial. | Use `--` para separar o entrecomille. |
| `RuntimeError: Click will abort further execution because Python was configured to use ASCII` | Locale ASCII en Linux antiguo. | `export LC_ALL=C.UTF-8` (Python ≥ 3.7 suele resolverlo). |
| Los colores no aparecen al redirigir la salida | Click los elimina fuera de una terminal. | Es el comportamiento esperado; `color=True` en `echo` para forzarlos. |
| El *docstring* se muestra mal formateado | Click reajusta los párrafos. | Use `\b` en una línea sola antes del bloque que no debe reajustarse. |

---

## 21. Buenas prácticas

- Use `click.echo` en lugar de `print`: gestiona codificaciones, tuberías y colores correctamente.
- Agrupe los comandos en módulos y regístrelos con `add_command`; use carga diferida en CLIs grandes.
- Comparta el estado con un objeto propio y `make_pass_decorator`, no con variables globales.
- Use los tipos de Click (`Path`, `Choice`, `IntRange`, `File`) en lugar de validar manualmente.
- Lance `ClickException`/`UsageError` en lugar de imprimir errores y llamar a `sys.exit`.
- Ofrezca `--yes` en los comandos con confirmación para que puedan usarse en scripts.
- Permita configurar opciones por variables de entorno y archivo de configuración (`default_map`).
- Mantenga la lógica de negocio fuera de las funciones decoradas.
- Pruebe con `CliRunner` e `isolated_filesystem`.
- Si quiere ayuda con formato Rich sin cambiar de librería, use `rich-click`.

---

## 22. Referencia rápida

```python
@click.command() / @click.group(chain=False, invoke_without_command=False, cls=..., context_settings={...})
@click.option("--largo", "-c", "nombre", default=, type=, required=, help=, show_default=,
              is_flag=, flag_value=, multiple=, nargs=, count=, prompt=, hide_input=,
              envvar=, callback=, is_eager=, expose_value=, shell_complete=)
@click.argument("nombre", type=, nargs=-1, required=, default=)
@click.pass_context / @click.pass_obj / pass_x = click.make_pass_decorator(Clase)
@click.version_option("1.0") · @click.confirmation_option() · @click.password_option()
grupo.add_command(cmd, name=) · ctx.invoke(cmd, **kw) · ctx.obj · ctx.default_map · ctx.call_on_close(f)

Tipos: Choice IntRange FloatRange Path(path_type=Path) File("r") DateTime UUID Tuple ParamType
click.echo / secho / style · prompt / confirm(abort=True) · progressbar · echo_via_pager · edit · launch
click.getchar · pause · clear · get_app_dir · open_file(atomic=True)
ClickException · UsageError · BadParameter · Abort · FileError
from click.testing import CliRunner; r = CliRunner().invoke(cli, [...], input=, env=)
cli(auto_envvar_prefix="APP")
```

---

## 23. Ejercicios propuestos

1. Cree una herramienta `notas` con los subcomandos `nueva` (abre el editor con `click.edit`), `listar` (con paginador) y `buscar`.
2. Implemente un tipo de parámetro `TipoDuracion` que acepte valores como `30s`, `5m` o `2h` y devuelva segundos.
3. Añada al ejemplo completo un subcomando `verificar` que compruebe la integridad de cada ZIP (`ZipFile.testzip`).
4. Convierta el ejemplo completo para que sus subcomandos se carguen de forma diferida desde módulos separados.
5. Escriba pruebas con `CliRunner` e `isolated_filesystem` para `crear`, `listar` y `purgar`.

---

## 24. Referencias

- Documentación oficial: https://click.palletsprojects.com/
- Repositorio: https://github.com/pallets/click
- rich-click (ayuda con formato Rich): https://github.com/ewels/rich-click
- click-option-group: https://github.com/click-contrib/click-option-group
