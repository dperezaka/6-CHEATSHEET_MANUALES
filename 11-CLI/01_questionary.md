# Manual de uso: questionary

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Crear prompts interactivos en terminal
> **Tipo:** Librería de terceros (`pip install questionary`) · **Versión de referencia:** 2.1

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer ejemplo](#4-primer-ejemplo)
5. [Tipos de pregunta](#5-tipos-de-pregunta)
6. [Opciones de las listas: `Choice` y `Separator`](#6-opciones-de-las-listas-choice-y-separator)
7. [Validación de respuestas](#7-validación-de-respuestas)
8. [Formularios y preguntas encadenadas](#8-formularios-y-preguntas-encadenadas)
9. [Preguntas condicionales](#9-preguntas-condicionales)
10. [Estilos y personalización](#10-estilos-y-personalización)
11. [Cancelación con Ctrl+C](#11-cancelación-con-ctrlc)
12. [Uso con asyncio](#12-uso-con-asyncio)
13. [Integración con otras librerías](#13-integración-con-otras-librerías)
14. [Pruebas automatizadas](#14-pruebas-automatizadas)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**questionary** es una librería para crear preguntas interactivas en la terminal: campos de texto, contraseñas, confirmaciones, menús que se recorren con las flechas, selección múltiple, autocompletado y selección de rutas. Está construida sobre **prompt_toolkit**, de la que hereda el control del teclado y la compatibilidad multiplataforma.

**Ventajas:**
- API muy sencilla: una función por tipo de pregunta.
- Menús navegables con flechas, atajos numéricos y búsqueda.
- Validación integrada, con mensajes de error en línea.
- Formularios declarativos (diccionarios) con preguntas condicionales.
- Soporte nativo de `asyncio`.
- Funciona en Windows, macOS y Linux.

**Limitaciones:**
- Requiere una terminal interactiva (TTY): no funciona con entrada redirigida ni en la mayoría de entornos de CI.
- No parsea argumentos de línea de comandos (para eso se combina con Typer, Click o argparse).
- No ofrece búsqueda difusa (*fuzzy*) avanzada: para ello existe InquirerPy.
- No está pensada para interfaces a pantalla completa (use Textual).

| Si necesita… | Opción |
|---|---|
| Preguntas interactivas sencillas y formularios | **questionary** |
| Menús con búsqueda difusa y muchas opciones | InquirerPy |
| Argumentos y subcomandos (`app.py crear --nombre X`) | Typer / Click / argparse |
| Prompts mínimos junto a salida con formato | Rich (`Prompt`, `Confirm`) |
| Aplicación completa a pantalla completa | Textual |

---

## 2. Instalación e importación

```bash
pip install questionary
```

Comprobar la instalación:

```bash
python -c "import questionary; print(questionary.__version__)"
```

Importación habitual:

```python
import questionary
from questionary import Choice, Separator, Style, Validator, ValidationError
```

> questionary instala automáticamente su dependencia, `prompt_toolkit`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Pregunta (`Question`) | Objeto devuelto por `questionary.text()`, `select()`, etc. **No se muestra hasta llamar a `.ask()`.** |
| `.ask()` | Muestra la pregunta y devuelve la respuesta. Devuelve `None` si el usuario pulsa Ctrl+C. |
| `.unsafe_ask()` | Igual que `ask()`, pero lanza `KeyboardInterrupt` al cancelar. |
| `.ask_async()` | Versión asíncrona (`await`). |
| `Choice` | Opción de una lista con texto visible, valor interno, estado y descripción. |
| Validador | Función o clase que acepta o rechaza la respuesta antes de continuar. |
| Formulario | Conjunto de preguntas que se responden en orden y devuelven un diccionario. |
| `qmark` | Símbolo que precede a la pregunta (por defecto `?`). |
| Estilo | Colores y formato de cada parte de la pregunta (`Style`). |

```python
pregunta = questionary.text("¿Nombre?")   # Se crea, pero no se muestra
respuesta = pregunta.ask()                # Ahora se muestra y espera
```

---

## 4. Primer ejemplo

```python
import questionary

nombre = questionary.text("¿Cómo se llama?").ask()
lenguaje = questionary.select(
    "¿Cuál es su lenguaje favorito?",
    choices=["Python", "JavaScript", "Go", "Rust"],
).ask()

if questionary.confirm(f"¿Confirma que prefiere {lenguaje}?").ask():
    print(f"Hola, {nombre}. Buena elección.")
```

Resultado en la terminal:

```
? ¿Cómo se llama? Ana
? ¿Cuál es su lenguaje favorito? (Use arrow keys)
 » Python
   JavaScript
   Go
   Rust
? ¿Confirma que prefiere Python? (Y/n)
```

---

## 5. Tipos de pregunta

### 5.1 Texto (`text`)

```python
nombre = questionary.text("Nombre del proyecto:", default="mi-proyecto").ask()

descripcion = questionary.text(
    "Descripción:",
    multiline=True,                               # Varias líneas
    instruction="(Esc + Enter para terminar)",
).ask()
```

### 5.2 Contraseña (`password`)

```python
clave = questionary.password("Contraseña:").ask()     # Se muestra con asteriscos
```

### 5.3 Confirmación (`confirm`)

```python
continuar = questionary.confirm("¿Desea continuar?", default=False).ask()   # True / False

# auto_enter=False obliga a pulsar Enter tras la tecla y/n
questionary.confirm("¿Borrar todo?", default=False, auto_enter=False).ask()
```

### 5.4 Selección única (`select`)

```python
entorno = questionary.select(
    "Entorno de despliegue:",
    choices=["desarrollo", "pruebas", "producción"],
    default="pruebas",
    use_shortcuts=True,          # Atajos 1, 2, 3… (máximo 36 opciones)
    use_search_filter=True,      # Escribir para filtrar (requiere use_jk_keys=False)
    use_jk_keys=False,
    instruction="(flechas o escriba para filtrar)",
).ask()
```

| Opción | Efecto |
|---|---|
| `default` | Opción preseleccionada. |
| `use_shortcuts` | Muestra un número o letra por opción. |
| `use_arrow_keys` | Navegación con flechas (activa por defecto). |
| `use_jk_keys` | Navegación con `j`/`k` estilo vim (activa por defecto). |
| `use_search_filter` | Filtra las opciones al escribir. Incompatible con `use_jk_keys=True`. |
| `use_indicator` | Muestra un indicador `●`/`○` junto a cada opción. |
| `pointer` | Símbolo del cursor (por defecto `»`). |
| `show_description` | Muestra la descripción de la `Choice` resaltada. |

### 5.5 Selección por número (`rawselect`)

```python
formato = questionary.rawselect("Formato de salida:", choices=["CSV", "JSON", "Excel"]).ask()
```

```
? Formato de salida:
  1) CSV
  2) JSON
  3) Excel
  Answer: 2
```

### 5.6 Selección múltiple (`checkbox`)

```python
modulos = questionary.checkbox(
    "Módulos a instalar:",
    choices=["autenticación", "pagos", "informes", "notificaciones"],
    validate=lambda r: True if r else "Seleccione al menos uno.",
).ask()                                            # Devuelve una lista
```

Teclas: **Espacio** marca/desmarca, **a** marca todas, **i** invierte la selección, **Enter** confirma.

### 5.7 Autocompletado (`autocomplete`)

```python
pais = questionary.autocomplete(
    "País:",
    choices=["España", "Argentina", "México", "Colombia", "Chile"],
    meta_information={"España": "Europa", "México": "América del Norte"},
    ignore_case=True,
    match_middle=True,           # Coincide aunque el texto esté en medio
    validate=lambda t: t in ["España", "Argentina", "México", "Colombia", "Chile"] or "País no válido",
).ask()
```

> `autocomplete` **sugiere**, pero acepta cualquier texto. Si solo admite valores de la lista, añada `validate`.

### 5.8 Ruta (`path`)

```python
carpeta = questionary.path("Carpeta de salida:", only_directories=True, default="./salida").ask()

archivo = questionary.path(
    "Archivo de configuración:",
    file_filter=lambda p: p.endswith((".toml", ".yaml")) or Path(p).is_dir(),
).ask()
```

Pulse **Tab** para autocompletar la ruta.

### 5.9 Pausa (`press_any_key_to_continue`)

```python
questionary.press_any_key_to_continue("Pulse una tecla para volver al menú…").ask()
```

### 5.10 Imprimir con estilo (`print`)

```python
questionary.print("✔ Operación completada", style="bold fg:ansigreen")
questionary.print("Aviso: sin conexión", style="italic fg:#d78700")
```

---

## 6. Opciones de las listas: `Choice` y `Separator`

`Choice` separa el **texto visible** del **valor devuelto** y permite deshabilitar o premarcar opciones.

```python
from questionary import Choice, Separator

plan = questionary.select(
    "Seleccione un plan:",
    choices=[
        Choice("Básico (5 €/mes)", value="basico", description="1 usuario, 10 GB"),
        Choice("Profesional (15 €/mes)", value="pro", description="5 usuarios, 100 GB"),
        Separator(),
        Choice("Empresa", value="empresa", disabled="Contacte con ventas"),
    ],
).ask()
# plan == "pro"
```

| Parámetro | Efecto |
|---|---|
| `title` | Texto visible. |
| `value` | Valor devuelto (si se omite, el título). Puede ser cualquier objeto. |
| `checked` | Premarcada (solo `checkbox`). |
| `disabled` | Texto que explica por qué no se puede elegir. |
| `shortcut_key` | Tecla de atajo personalizada. |
| `description` | Texto de ayuda mostrado al resaltar la opción. |

**Separadores** con texto:

```python
choices = [Separator("── Bases de datos ──"), "PostgreSQL", "MySQL",
           Separator("── Cachés ──"), "Redis", "Memcached"]
```

**Valores como objetos:**

```python
usuarios = repo.listar()
usuario = questionary.select(
    "Usuario:", choices=[Choice(u.nombre, value=u) for u in usuarios]
).ask()                               # Devuelve el objeto, no el texto
```

**Premarcar en `checkbox`:**

```python
questionary.checkbox("Notificar por:", choices=[
    Choice("Email", checked=True), Choice("SMS"), Choice("Push", checked=True),
]).ask()
```

---

## 7. Validación de respuestas

### 7.1 Con una función

La función devuelve `True` si la respuesta es válida, o un **texto con el mensaje de error**:

```python
def validar_edad(texto: str) -> bool | str:
    if not texto.isdigit():
        return "Introduzca un número entero."
    if not 18 <= int(texto) <= 120:
        return "La edad debe estar entre 18 y 120."
    return True

edad = int(questionary.text("Edad:", validate=validar_edad).ask())
```

### 7.2 Con una clase `Validator`

Útil para reutilizar validaciones o indicar la posición del error:

```python
import re
from questionary import Validator, ValidationError

class ValidadorEmail(Validator):
    PATRON = re.compile(r"^[\w.+-]+@[\w-]+\.[\w.-]+$")

    def validate(self, documento):
        if not self.PATRON.match(documento.text):
            raise ValidationError(message="Email no válido",
                                  cursor_position=len(documento.text))

email = questionary.text("Email:", validate=ValidadorEmail).ask()
```

### 7.3 Validación en `checkbox`

En `checkbox` la función recibe la **lista** de opciones marcadas:

```python
questionary.checkbox(
    "Elija entre 1 y 3 opciones:",
    choices=["A", "B", "C", "D", "E"],
    validate=lambda sel: 1 <= len(sel) <= 3 or "Debe elegir entre 1 y 3.",
).ask()
```

> La validación solo comprueba; la **conversión** (a `int`, `Path`, `date`…) debe hacerse después de `ask()`.

---

## 8. Formularios y preguntas encadenadas

### 8.1 `questionary.form`

Agrupa preguntas y devuelve un diccionario:

```python
respuestas = questionary.form(
    nombre=questionary.text("Nombre del proyecto:"),
    licencia=questionary.select("Licencia:", choices=["MIT", "Apache-2.0", "GPL-3.0"]),
    tests=questionary.confirm("¿Incluir pruebas?"),
).ask()

# {'nombre': 'demo', 'licencia': 'MIT', 'tests': True}
```

### 8.2 `questionary.prompt` (formato diccionario)

Permite definir preguntas como datos (por ejemplo, cargadas de un JSON o YAML):

```python
preguntas = [
    {"type": "text", "name": "nombre", "message": "Nombre:"},
    {"type": "select", "name": "rol", "message": "Rol:", "choices": ["admin", "editor", "lector"]},
    {"type": "confirm", "name": "activo", "message": "¿Activo?", "default": True},
]
respuestas = questionary.prompt(preguntas)
```

| Clave | Uso |
|---|---|
| `type` | `text`, `password`, `confirm`, `select`, `rawselect`, `checkbox`, `autocomplete`, `path`, `print`, `press_any_key_to_continue` |
| `name` | Clave del diccionario de respuestas. |
| `message` | Texto de la pregunta (puede ser una función que recibe las respuestas previas). |
| `default` | Valor por defecto. |
| `choices` | Opciones (puede ser una función). |
| `validate` | Validador. |
| `filter` | Función que transforma la respuesta antes de guardarla. |
| `when` | Función que decide si la pregunta se muestra. |

```python
{"type": "text", "name": "puerto", "message": "Puerto:", "default": "8080",
 "validate": lambda t: t.isdigit() or "Número no válido", "filter": int}
```

> `prompt()` devuelve `{}` si se cancela con Ctrl+C (o lanza `KeyboardInterrupt` con `unsafe_prompt()`).

---

## 9. Preguntas condicionales

### 9.1 Con `when`

```python
preguntas = [
    {"type": "confirm", "name": "usar_bd", "message": "¿Usar base de datos?"},
    {"type": "select", "name": "motor", "message": "Motor:",
     "choices": ["PostgreSQL", "MySQL", "SQLite"],
     "when": lambda r: r["usar_bd"]},
    {"type": "text", "name": "host", "message": "Host:", "default": "localhost",
     "when": lambda r: r.get("motor") in ("PostgreSQL", "MySQL")},
]
respuestas = questionary.prompt(preguntas)
```

### 9.2 Con `skip_if` (API de objetos)

```python
es_admin = questionary.confirm("¿Es administrador?").ask()
clave = questionary.password("Clave de administración:").skip_if(not es_admin, default=None).ask()
```

### 9.3 Mensajes y opciones dinámicas

```python
{"type": "select", "name": "ciudad",
 "message": lambda r: f"Ciudad de {r['pais']}:",
 "choices": lambda r: CIUDADES_POR_PAIS[r["pais"]]}
```

---

## 10. Estilos y personalización

### 10.1 Estilo global

```python
from questionary import Style

estilo = Style([
    ("qmark", "fg:#1F78B4 bold"),        # Símbolo inicial
    ("question", "bold"),                # Texto de la pregunta
    ("answer", "fg:#2CA02C bold"),       # Respuesta ya enviada
    ("pointer", "fg:#1F78B4 bold"),      # Cursor del menú
    ("highlighted", "fg:#1F78B4 bold"),  # Opción resaltada
    ("selected", "fg:#2CA02C"),          # Opción marcada (checkbox)
    ("separator", "fg:#808080"),
    ("instruction", "fg:#808080 italic"),
    ("text", ""),
    ("disabled", "fg:#858585 italic"),
])

questionary.select("Opción:", choices=["A", "B"], style=estilo).ask()
```

Formato de los valores: `fg:#rrggbb`, `bg:#rrggbb`, `fg:ansired`, `bold`, `italic`, `underline`, `reverse`, `noinherit`.

### 10.2 Símbolos

```python
questionary.text("Nombre:", qmark="›").ask()
questionary.select("Opción:", choices=["A", "B"], qmark="◆", pointer="→").ask()
questionary.checkbox("Opciones:", choices=["A", "B"], qmark="☐").ask()
```

### 10.3 Títulos con formato dentro de una `Choice`

```python
Choice(title=[("class:text", "Servidor "), ("fg:ansigreen", "● activo")], value="srv1")
```

---

## 11. Cancelación con Ctrl+C

Por defecto, `ask()` captura **Ctrl+C**, muestra «Cancelled by user» y devuelve `None`. Compruébelo siempre:

```python
nombre = questionary.text("Nombre:").ask()
if nombre is None:
    raise SystemExit(1)                     # El usuario canceló
```

Para gestionar la cancelación con excepciones (más limpio en flujos largos):

```python
try:
    nombre = questionary.text("Nombre:").unsafe_ask()
    email = questionary.text("Email:").unsafe_ask()
except KeyboardInterrupt:
    print("\nOperación cancelada.")
    raise SystemExit(130)
```

Mensaje personalizado al cancelar:

```python
questionary.text("Nombre:").ask(kbi_msg="Cancelado.")
```

> Atención: una respuesta vacía en `text` devuelve `""`, no `None`. Distinga ambos casos.

---

## 12. Uso con asyncio

```python
import asyncio
import questionary

async def main():
    nombre = await questionary.text("Nombre:").ask_async()
    confirmado = await questionary.confirm("¿Continuar?").ask_async()
    if confirmado:
        await procesar(nombre)

asyncio.run(main())
```

Para formularios: `await questionary.form(...).ask_async()` y `await questionary.prompt_async(preguntas)`.

> No llame a `ask()` (síncrono) dentro de un bucle de eventos en ejecución: provoca `RuntimeError`. Use siempre `ask_async()`.

---

## 13. Integración con otras librerías

### 13.1 Con Typer: pedir solo lo que falta

```python
import typer, questionary

app = typer.Typer()

@app.command()
def crear(nombre: str = typer.Option(None), entorno: str = typer.Option(None)):
    nombre = nombre or questionary.text("Nombre:").unsafe_ask()
    entorno = entorno or questionary.select("Entorno:", ["dev", "prod"]).unsafe_ask()
    typer.echo(f"Creando {nombre} en {entorno}")
```

Así el comando funciona tanto en modo interactivo como en scripts (`crear --nombre x --entorno dev`).

### 13.2 Con Rich: salida con formato

```python
from rich.console import Console
from rich.table import Table

consola = Console()
seleccion = questionary.checkbox("Servicios:", choices=["web", "api", "worker"]).ask()

tabla = Table("Servicio", "Estado")
for s in seleccion:
    tabla.add_row(s, "[green]reiniciado[/]")
consola.print(tabla)
```

### 13.3 Detectar si hay terminal interactiva

```python
import sys

if not sys.stdin.isatty():
    sys.exit("Este comando requiere una terminal interactiva. Use --no-interactivo.")
```

---

## 14. Pruebas automatizadas

questionary se basa en prompt_toolkit, que permite simular la entrada de teclado:

```python
from prompt_toolkit.input import create_pipe_input
from prompt_toolkit.output import DummyOutput
import questionary

def test_select():
    with create_pipe_input() as entrada:
        entrada.send_text("\x1b[B\r")           # Flecha abajo + Enter
        resultado = questionary.select(
            "Opción:", choices=["A", "B", "C"], input=entrada, output=DummyOutput()
        ).ask()
    assert resultado == "B"
```

| Secuencia | Tecla |
|---|---|
| `"\r"` | Enter |
| `"\x1b[A"` / `"\x1b[B"` | Flecha arriba / abajo |
| `" "` | Espacio |
| `"\x03"` | Ctrl+C |

Otra opción, más sencilla, es separar la lógica de las preguntas y **simular (`mock`) las funciones** de questionary con `unittest.mock.patch`.

---

## 15. Ejemplo práctico completo

**Asistente de creación de proyectos**: pregunta los datos de un proyecto Python con validación, preguntas condicionales y opciones con descripción, muestra un resumen, pide confirmación y genera la estructura de carpetas.

```python
"""
Asistente interactivo para crear proyectos Python.
Requiere: pip install questionary
"""
import re
import sys
from datetime import date
from pathlib import Path

import questionary
from questionary import Choice, Separator, Style

ESTILO = Style([
    ("qmark", "fg:#1F78B4 bold"),
    ("question", "bold"),
    ("answer", "fg:#2CA02C bold"),
    ("pointer", "fg:#1F78B4 bold"),
    ("highlighted", "fg:#1F78B4 bold"),
    ("selected", "fg:#2CA02C"),
    ("instruction", "fg:#808080 italic"),
])

PATRON_NOMBRE = re.compile(r"^[a-z][a-z0-9_-]{2,39}$")
PATRON_EMAIL = re.compile(r"^[\w.+-]+@[\w-]+\.[\w.-]+$")

LICENCIAS = {
    "MIT": "Permisiva y breve",
    "Apache-2.0": "Permisiva, con cesión de patentes",
    "GPL-3.0": "Copyleft: las obras derivadas deben ser libres",
    "Propietaria": "Sin licencia pública",
}


# ------------------------------------------------------------------ Validadores
def validar_nombre(texto: str) -> bool | str:
    if not PATRON_NOMBRE.match(texto):
        return "Use 3-40 caracteres: minúsculas, números, '-' o '_', empezando por letra."
    if Path(texto).exists():
        return f"Ya existe una carpeta llamada «{texto}»."
    return True


def validar_version(texto: str) -> bool | str:
    return bool(re.fullmatch(r"3\.(1[0-4])", texto)) or "Indique una versión entre 3.10 y 3.14."


# ------------------------------------------------------------------ Preguntas
def preguntar() -> dict:
    preguntas = [
        {"type": "text", "name": "nombre", "message": "Nombre del proyecto:",
         "validate": validar_nombre},
        {"type": "text", "name": "descripcion", "message": "Descripción breve:",
         "default": "Proyecto Python"},
        {"type": "text", "name": "autor", "message": "Autor:"},
        {"type": "text", "name": "email", "message": "Email del autor:",
         "validate": lambda t: bool(PATRON_EMAIL.match(t)) or "Email no válido",
         "filter": str.lower},
        {"type": "select", "name": "tipo", "message": "Tipo de proyecto:",
         "choices": [
             Choice("Librería", value="libreria", description="Paquete reutilizable"),
             Choice("Aplicación de consola", value="cli", description="Con punto de entrada"),
             Choice("API web", value="api", description="Servicio HTTP"),
         ]},
        {"type": "select", "name": "framework", "message": "Framework web:",
         "choices": ["FastAPI", "Flask", "Django"],
         "when": lambda r: r["tipo"] == "api"},
        {"type": "text", "name": "python", "message": "Versión mínima de Python:",
         "default": "3.12", "validate": validar_version},
        {"type": "select", "name": "licencia", "message": "Licencia:",
         "choices": [Choice(k, description=v) for k, v in LICENCIAS.items()]},
        {"type": "checkbox", "name": "extras", "message": "Herramientas adicionales:",
         "choices": [
             Separator("── Calidad ──"),
             Choice("pytest", checked=True), Choice("ruff", checked=True), Choice("mypy"),
             Separator("── Automatización ──"),
             Choice("pre-commit"), Choice("GitHub Actions", value="github-actions"),
         ]},
        {"type": "confirm", "name": "git", "message": "¿Inicializar un repositorio git?",
         "default": True},
    ]
    return questionary.unsafe_prompt(preguntas, style=ESTILO)


# ------------------------------------------------------------------ Generación
def generar(datos: dict) -> Path:
    raiz = Path(datos["nombre"])
    paquete = datos["nombre"].replace("-", "_")
    (raiz / "src" / paquete).mkdir(parents=True)
    (raiz / "src" / paquete / "__init__.py").write_text('__version__ = "0.1.0"\n', encoding="utf-8")

    if "pytest" in datos["extras"]:
        (raiz / "tests").mkdir()
        (raiz / "tests" / "test_basico.py").write_text(
            f"import {paquete}\n\n\ndef test_version():\n    assert {paquete}.__version__\n",
            encoding="utf-8")

    dependencias = {"api": [datos.get("framework", "").lower()], "cli": ["typer"]}.get(datos["tipo"], [])
    (raiz / "pyproject.toml").write_text(
        "[project]\n"
        f'name = "{datos["nombre"]}"\nversion = "0.1.0"\n'
        f'description = "{datos["descripcion"]}"\n'
        f'authors = [{{ name = "{datos["autor"]}", email = "{datos["email"]}" }}]\n'
        f'requires-python = ">={datos["python"]}"\n'
        f'license = "{datos["licencia"]}"\n'
        f"dependencies = {dependencias}\n", encoding="utf-8")

    (raiz / "README.md").write_text(
        f"# {datos['nombre']}\n\n{datos['descripcion']}\n\n"
        f"© {date.today().year} {datos['autor']}\n", encoding="utf-8")

    if datos["git"]:
        (raiz / ".gitignore").write_text("__pycache__/\n.venv/\n*.egg-info/\ndist/\n", encoding="utf-8")
    return raiz


def mostrar_resumen(datos: dict) -> None:
    questionary.print("\nResumen del proyecto", style="bold underline")
    for clave, valor in datos.items():
        if isinstance(valor, list):
            valor = ", ".join(valor) or "—"
        questionary.print(f"  {clave:<12} ", style="fg:#808080", end="")
        questionary.print(str(valor), style="bold")
    print()


# ------------------------------------------------------------------ Programa principal
def main() -> int:
    if not sys.stdin.isatty():
        print("Este asistente requiere una terminal interactiva.", file=sys.stderr)
        return 2

    questionary.print("Asistente de nuevos proyectos\n", style="bold fg:#1F78B4")
    try:
        datos = preguntar()
        mostrar_resumen(datos)
        if not questionary.confirm("¿Crear el proyecto?", style=ESTILO).unsafe_ask():
            questionary.print("Operación cancelada.", style="fg:ansiyellow")
            return 1
    except KeyboardInterrupt:
        questionary.print("\nOperación cancelada por el usuario.", style="fg:ansiyellow")
        return 130

    ruta = generar(datos)
    questionary.print(f"✔ Proyecto creado en {ruta.resolve()}", style="bold fg:ansigreen")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| La pregunta no aparece | Falta `.ask()`. | `questionary.text("…").ask()`. |
| `TypeError: '>=' not supported between 'NoneType'…` | El usuario canceló y `ask()` devolvió `None`. | Compruebe `None` o use `unsafe_ask()` con `try/except`. |
| `OSError` / `NoConsoleScreenBufferError` / el programa se cuelga | No hay terminal interactiva (CI, tubería, IDE sin TTY). | Compruebe `sys.stdin.isatty()` y ofrezca opciones por argumentos. |
| `RuntimeError: This event loop is already running` | `ask()` dentro de código `async`. | Use `await …ask_async()`. |
| `ValueError` al activar `use_search_filter` | Se mantiene `use_jk_keys=True`. | Añada `use_jk_keys=False`. |
| `autocomplete` acepta valores fuera de la lista | Solo sugiere, no restringe. | Añada `validate`. |
| La validación de `checkbox` no hace nada | Se espera recibir texto, pero recibe una lista. | Escriba el validador para una lista. |
| Se devuelve el texto en lugar del valor | Opciones como `str`. | Use `Choice(titulo, value=...)`. |
| Los colores no se ven en Windows antiguo | Consola sin soporte ANSI. | Use Windows Terminal o PowerShell 7. |
| `ValueError` con `use_shortcuts` | Más de 36 opciones. | Desactive los atajos o use `use_search_filter`. |

---

## 17. Buenas prácticas

- Compruebe siempre si la respuesta es `None` o use `unsafe_ask()` dentro de un `try/except KeyboardInterrupt`.
- Use `Choice(titulo, value=...)` para separar el texto mostrado del valor que usa el programa.
- Valide durante la pregunta (`validate`) y convierta después (`int()`, `Path()`), no al revés.
- Ofrezca valores por defecto razonables para que el usuario pueda avanzar con Enter.
- Combine questionary con un parser de argumentos (Typer/Click) para que la herramienta también funcione **sin interacción** en scripts y CI.
- Defina el estilo una sola vez y reutilícelo en todas las preguntas.
- Muestre un resumen y pida confirmación antes de acciones destructivas o irreversibles.
- Prefiera `select` frente a `text` cuando el conjunto de respuestas válidas sea cerrado.
- Mantenga las preguntas en una función o estructura de datos separada de la lógica, para poder probarla.

---

## 18. Referencia rápida

```python
questionary.text(msg, default="", validate=f, multiline=False).ask()
questionary.password(msg).ask()
questionary.confirm(msg, default=True).ask()                       # bool
questionary.select(msg, choices=[...], default=x, use_shortcuts=True).ask()
questionary.rawselect(msg, choices=[...]).ask()
questionary.checkbox(msg, choices=[...], validate=f).ask()         # list
questionary.autocomplete(msg, choices=[...], match_middle=True).ask()
questionary.path(msg, only_directories=True).ask()
questionary.press_any_key_to_continue().ask()
questionary.print(texto, style="bold fg:ansigreen")

Choice(titulo, value=v, checked=True, disabled="motivo", description="ayuda") · Separator("texto")
questionary.form(a=q1, b=q2).ask()                                 # dict
questionary.prompt([{"type", "name", "message", "when", "filter", "validate"}])
pregunta.skip_if(condicion, default=None)
.ask() → None si Ctrl+C · .unsafe_ask() → KeyboardInterrupt · await .ask_async()
Style([("qmark", "fg:#1F78B4 bold"), ("answer", "fg:#2CA02C"), ...])
```

---

## 19. Ejercicios propuestos

1. Cree un conversor de unidades que pregunte la magnitud (`select`), la unidad de origen y de destino, y el valor con validación numérica.
2. Desarrolle un menú principal en bucle (`select`) para una agenda de contactos con las opciones *Añadir*, *Buscar* (`autocomplete`), *Eliminar* (con confirmación) y *Salir*.
3. Cargue un cuestionario desde un archivo JSON y ejecútelo con `questionary.prompt`, incluyendo preguntas condicionales con `when`.
4. Combine Typer y questionary en un comando que pida interactivamente solo los parámetros que no se hayan pasado por línea de comandos.
5. Escriba pruebas con `create_pipe_input` para el asistente del ejemplo completo.

---

## 20. Referencias

- Documentación oficial: https://questionary.readthedocs.io/
- Repositorio: https://github.com/tmbo/questionary
- Ejemplos oficiales: https://github.com/tmbo/questionary/tree/master/examples
- prompt_toolkit (base de questionary): https://python-prompt-toolkit.readthedocs.io/
