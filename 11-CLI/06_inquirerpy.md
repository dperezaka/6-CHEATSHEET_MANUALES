# Manual de uso: InquirerPy

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Menús y preguntas interactivas en terminal con búsqueda difusa
> **Tipo:** Librería de terceros (`pip install InquirerPy`) · **Versión de referencia:** 0.3

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Las dos API: `inquirer` y `prompt`](#4-las-dos-api-inquirer-y-prompt)
5. [Tipos de pregunta](#5-tipos-de-pregunta)
6. [Búsqueda difusa (`fuzzy`)](#6-búsqueda-difusa-fuzzy)
7. [Opciones: `Choice`, `Separator` y opciones dinámicas](#7-opciones-choice-separator-y-opciones-dinámicas)
8. [Validación](#8-validación)
9. [Transformar y filtrar respuestas](#9-transformar-y-filtrar-respuestas)
10. [Preguntas opcionales y cancelación](#10-preguntas-opcionales-y-cancelación)
11. [Atajos de teclado](#11-atajos-de-teclado)
12. [Estilos](#12-estilos)
13. [API clásica: preguntas condicionales](#13-api-clásica-preguntas-condicionales)
14. [Uso con asyncio](#14-uso-con-asyncio)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**InquirerPy** es una librería para crear preguntas interactivas en la terminal, inspirada en Inquirer.js y construida sobre **prompt_toolkit**. Además de los tipos habituales (texto, contraseña, confirmación, listas, selección múltiple, rutas), destaca por su pregunta **`fuzzy`**, que filtra listas largas con **búsqueda difusa** mientras se escribe, y por su alto grado de personalización (atajos de teclado, modo *vi*, bordes, alturas, marcadores).

**Ventajas:**
- Búsqueda difusa integrada, eficaz con miles de opciones.
- Pregunta numérica con flechas, límites y decimales.
- Selección múltiple en casi cualquier lista (`multiselect=True`).
- Atajos de teclado configurables y modo *vi*.
- Validadores predefinidos (vacío, número, ruta, contraseña).
- Dos estilos de API: objetos (`inquirer`) y diccionarios (`prompt`).

**Limitaciones:**
- Requiere una terminal interactiva (no funciona con entrada redirigida ni en CI).
- Desarrollo poco activo en los últimos años (la versión 0.3.4 es estable y funcional).
- No parsea argumentos de línea de comandos.

| Si necesita… | Opción |
|---|---|
| Listas largas con búsqueda difusa | **InquirerPy** |
| Preguntas sencillas con API mínima | questionary |
| Control de bajo nivel del teclado y la pantalla | prompt_toolkit |
| Argumentos y subcomandos | Typer / Click / argparse |
| Aplicación completa a pantalla completa | Textual |

---

## 2. Instalación e importación

```bash
pip install InquirerPy
```

```python
from InquirerPy import inquirer, prompt, get_style
from InquirerPy.base.control import Choice
from InquirerPy.separator import Separator
from InquirerPy.validator import EmptyInputValidator, NumberValidator, PathValidator, PasswordValidator
```

> El nombre del paquete en `pip` y en `import` es **InquirerPy** (con mayúsculas).

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Pregunta | Objeto creado con `inquirer.text()`, `inquirer.select()`, etc. |
| `.execute()` | Muestra la pregunta y devuelve la respuesta. |
| `.execute_async()` | Versión asíncrona. |
| `Choice` | Opción con valor interno (`value`), texto visible (`name`) y estado (`enabled`). |
| `Separator` | Línea separadora no seleccionable. |
| `validate` | Función o `Validator` que acepta o rechaza la respuesta. |
| `filter` | Transforma el **valor devuelto** (no lo que se muestra). |
| `transformer` | Transforma lo que se **muestra** tras responder (no el valor). |
| `mandatory` | Si es `False`, la pregunta se puede omitir (Ctrl+Z) y devuelve `None`. |
| `keybindings` | Diccionario para redefinir las teclas de cada acción. |

---

## 4. Las dos API: `inquirer` y `prompt`

### 4.1 API de objetos (`inquirer`) — recomendada

```python
from InquirerPy import inquirer

nombre = inquirer.text(message="Nombre:").execute()
lenguaje = inquirer.select(message="Lenguaje:", choices=["Python", "Go", "Rust"]).execute()
seguir = inquirer.confirm(message="¿Continuar?", default=True).execute()
```

Ventajas: autocompletado del editor, comprobación de tipos y control del flujo con Python normal (`if`, bucles, funciones).

### 4.2 API clásica (`prompt`) — diccionarios

```python
from InquirerPy import prompt

preguntas = [
    {"type": "input", "name": "nombre", "message": "Nombre:"},
    {"type": "list", "name": "lenguaje", "message": "Lenguaje:", "choices": ["Python", "Go", "Rust"]},
    {"type": "confirm", "name": "seguir", "message": "¿Continuar?", "default": True},
]
respuestas = prompt(preguntas)
# {'nombre': 'Ana', 'lenguaje': 'Python', 'seguir': True}
```

Ventajas: preguntas definidas como datos (por ejemplo, desde JSON/YAML) y condiciones `when` declarativas.

| `inquirer.*` | `type` en `prompt` |
|---|---|
| `text` | `"input"` |
| `secret` | `"password"` |
| `number` | `"number"` |
| `confirm` | `"confirm"` |
| `select` | `"list"` |
| `rawlist` | `"rawlist"` |
| `expand` | `"expand"` |
| `checkbox` | `"checkbox"` |
| `fuzzy` | `"fuzzy"` |
| `filepath` | `"filepath"` |

---

## 5. Tipos de pregunta

### 5.1 Texto (`text`)

```python
from prompt_toolkit.completion import WordCompleter

ciudad = inquirer.text(
    message="Ciudad:",
    default="Madrid",
    completer=WordCompleter(["Madrid", "Barcelona", "Sevilla", "Valencia"], ignore_case=True),
    long_instruction="Pulse Tab para autocompletar",
).execute()

notas = inquirer.text(message="Notas:", multiline=True,
                      instruction="(Esc + Enter para terminar)").execute()
```

El completador también puede ser un diccionario (incluso anidado):

```python
inquirer.text(message="Comando:", completer={"git": {"commit": None, "push": None}, "docker": None})
```

### 5.2 Contraseña (`secret`)

```python
clave = inquirer.secret(
    message="Contraseña:",
    validate=PasswordValidator(length=8, cap=True, number=True, special=True,
                               message="Mín. 8 caracteres con mayúscula, número y símbolo"),
    transformer=lambda _: "[oculta]",
).execute()
```

### 5.3 Número (`number`)

```python
edad = inquirer.number(message="Edad:", min_allowed=0, max_allowed=120, default=30).execute()

precio = inquirer.number(
    message="Precio:",
    float_allowed=True,
    decimal_symbol=",",           # Separador decimal mostrado
    min_allowed=0,
    validate=EmptyInputValidator("Indique un precio"),
    filter=float,
).execute()
```

Teclas: **↑/↓** incrementan o decrementan; **-** cambia el signo. Devuelve texto salvo que use `filter` (`int`/`float`).

### 5.4 Confirmación (`confirm`)

```python
inquirer.confirm(message="¿Eliminar el archivo?", default=False).execute()

# Letras personalizadas (p. ej. español: s/n)
inquirer.confirm(message="¿Continuar?", confirm_letter="s", reject_letter="n",
                 transformer=lambda r: "Sí" if r else "No").execute()
```

### 5.5 Lista (`select`)

```python
entorno = inquirer.select(
    message="Entorno:",
    choices=["desarrollo", "pruebas", "producción"],
    default="pruebas",
    pointer="➜",
    border=True,
    cycle=True,                    # Al llegar al final, vuelve al principio
    max_height="50%",              # Altura máxima (líneas o porcentaje de la terminal)
).execute()

servicios = inquirer.select(
    message="Servicios a reiniciar:",
    choices=["web", "api", "worker", "scheduler"],
    multiselect=True,              # Selección múltiple con Espacio
    instruction="(Espacio: marcar · Enter: confirmar)",
).execute()                        # Lista
```

### 5.6 Lista numerada (`rawlist`)

```python
formato = inquirer.rawlist(message="Formato:", choices=["CSV", "JSON", "Excel"]).execute()
```

```
? Formato:
  1) CSV
❯ 2) JSON
  3) Excel
  Answer: 2
```

Se navega con flechas o pulsando el número.

### 5.7 Lista con teclas (`expand`)

Cada opción tiene una tecla; `h` muestra la lista completa:

```python
from InquirerPy.prompts.expand import ExpandChoice

accion = inquirer.expand(
    message="El archivo ya existe:",
    choices=[
        ExpandChoice("sobrescribir", name="Sobrescribir", key="o"),
        ExpandChoice("renombrar", name="Renombrar", key="r"),
        ExpandChoice("omitir", name="Omitir", key="s"),
    ],
    default="s",
).execute()
```

### 5.8 Casillas (`checkbox`)

```python
modulos = inquirer.checkbox(
    message="Módulos:",
    choices=[Choice("auth", name="Autenticación", enabled=True),
             Choice("pagos", name="Pagos"),
             Choice("informes", name="Informes", enabled=True)],
    enabled_symbol="◉",
    disabled_symbol="○",
    validate=lambda r: len(r) >= 1,
    invalid_message="Seleccione al menos un módulo",
).execute()
```

### 5.9 Ruta (`filepath`)

```python
from pathlib import Path

archivo = inquirer.filepath(
    message="Archivo CSV:",
    default=str(Path.cwd()) + "/",
    validate=PathValidator(is_file=True, message="El archivo no existe"),
    only_files=True,
    filter=Path,
).execute()

carpeta = inquirer.filepath(message="Carpeta:", only_directories=True,
                            validate=PathValidator(is_dir=True)).execute()
```

Pulse **Tab** para autocompletar.

---

## 6. Búsqueda difusa (`fuzzy`)

Permite escribir fragmentos (no necesariamente consecutivos) para filtrar las opciones. Por ejemplo, `mdr` encuentra «**M**a**dr**id».

```python
paises = ["Alemania", "Argentina", "Australia", "Bélgica", "Brasil", "Canadá", "Chile",
          "Colombia", "España", "Estados Unidos", "Francia", "México", "Perú", "Portugal"]

pais = inquirer.fuzzy(
    message="País:",
    choices=paises,
    max_height="40%",
    border=True,
    info=True,                     # Muestra «coincidencias/total»
    match_exact=False,             # Ctrl+F alterna a coincidencia exacta (si se configura)
).execute()
```

### 6.1 Selección múltiple con búsqueda

```python
etiquetas = inquirer.fuzzy(
    message="Etiquetas:",
    choices=lista_de_500_etiquetas,
    multiselect=True,
    instruction="(Tab: marcar · Enter: confirmar)",
    validate=lambda r: len(r) <= 5,
    invalid_message="Máximo 5 etiquetas",
).execute()
```

> En `fuzzy`, la tecla **Espacio** se escribe en el buscador; para marcar opciones se usa **Tab** (marca y baja) o **Shift+Tab** (marca y sube).

### 6.2 Listas muy grandes

`fuzzy` admite miles de elementos con buen rendimiento. `choices` también puede ser una **función**: se ejecuta al crear la pregunta y recibe las respuestas previas de la sesión (útil en la API clásica para generar las opciones a partir de ellas).

```python
def cargar_productos(_):
    return [Choice(p.id, name=f"{p.codigo} · {p.nombre}") for p in repo.todos()]

producto = inquirer.fuzzy(message="Producto:", choices=cargar_productos).execute()
```

### 6.3 Alternar coincidencia exacta

```python
inquirer.fuzzy(
    message="Archivo:",
    choices=archivos,
    keybindings={"toggle-exact": [{"key": "c-t"}]},
    long_instruction="Ctrl+T: búsqueda exacta / difusa",
).execute()
```

---

## 7. Opciones: `Choice`, `Separator` y opciones dinámicas

### 7.1 `Choice`

```python
from InquirerPy.base.control import Choice
from InquirerPy.separator import Separator

plan = inquirer.select(
    message="Plan:",
    choices=[
        Choice(value="basico", name="Básico (5 €/mes)"),
        Choice(value="pro", name="Profesional (15 €/mes)"),
        Separator(),
        Choice(value=None, name="Cancelar"),
    ],
    default="pro",
).execute()
```

| Parámetro | Efecto |
|---|---|
| `value` | Valor devuelto (cualquier objeto). |
| `name` | Texto mostrado (si se omite, `str(value)`). |
| `enabled` | Premarcada en selección múltiple / `checkbox`. |

También se aceptan diccionarios: `{"name": "Básico", "value": "basico"}`.

> ⚠ Si el `value` es una instancia de una **`dataclass`**, InquirerPy la devuelve convertida en **diccionario** (internamente usa `dataclasses.asdict`). Use como valor un identificador (nombre, id) y recupere el objeto después, o use clases normales.

```python
por_id = {p.id: p for p in productos}            # productos: lista de dataclasses
elegido = por_id[inquirer.select(message="Producto:",
                                 choices=[Choice(p.id, name=p.nombre) for p in productos]).execute()]
```

### 7.2 Separadores con texto

```python
choices = [Separator("── Bases de datos ──"), "PostgreSQL", "MySQL",
           Separator("── Cachés ──"), "Redis"]
```

### 7.3 Opciones dependientes de respuestas anteriores

```python
pais = inquirer.select(message="País:", choices=list(CIUDADES)).execute()
ciudad = inquirer.fuzzy(message="Ciudad:", choices=CIUDADES[pais]).execute()
```

En la API clásica, `choices` puede ser una función que recibe el diccionario de respuestas:

```python
{"type": "list", "name": "ciudad", "message": "Ciudad:",
 "choices": lambda r: CIUDADES[r["pais"]]}
```

---

## 8. Validación

### 8.1 Función + mensaje

```python
codigo = inquirer.text(
    message="Código postal:",
    validate=lambda t: t.isdigit() and len(t) == 5,
    invalid_message="Debe tener 5 dígitos",
).execute()
```

### 8.2 Validadores predefinidos

| Validador | Uso |
|---|---|
| `EmptyInputValidator(message)` | No admite respuestas vacías. |
| `NumberValidator(message, float_allowed=False)` | Solo números. |
| `PathValidator(message, is_file=False, is_dir=False)` | Ruta existente (archivo o carpeta). |
| `PasswordValidator(message, length, cap, special, number)` | Requisitos de contraseña. |

```python
puerto = inquirer.text(message="Puerto:", validate=NumberValidator("Número no válido"), filter=int).execute()
```

### 8.3 Validador propio

```python
import re
from InquirerPy.validator import Validator, ValidationError

class ValidadorEmail(Validator):
    def validate(self, documento):
        if not re.match(r"^[\w.+-]+@[\w-]+\.[\w.-]+$", documento.text):
            raise ValidationError(message="Email no válido", cursor_position=len(documento.text))

email = inquirer.text(message="Email:", validate=ValidadorEmail()).execute()
```

> En `checkbox`, `select(multiselect=True)` y `fuzzy(multiselect=True)`, la función de validación recibe la **lista** de opciones marcadas.

---

## 9. Transformar y filtrar respuestas

| Parámetro | Afecta a | Ejemplo |
|---|---|---|
| `filter` | El **valor devuelto** por `execute()`. | Convertir a `int`, `Path`, minúsculas… |
| `transformer` | El **texto mostrado** después de responder. | Ocultar, resumir, traducir. |

```python
servidores = inquirer.checkbox(
    message="Servidores:",
    choices=["srv-01", "srv-02", "srv-03", "srv-04"],
    transformer=lambda r: f"{len(r)} servidor(es) seleccionado(s)",   # Lo que se ve
    filter=lambda r: [s.upper() for s in r],                          # Lo que se devuelve
).execute()
```

```
? Servidores: 3 servidor(es) seleccionado(s)
```

---

## 10. Preguntas opcionales y cancelación

### 10.1 Ctrl+C

Por defecto, **Ctrl+C lanza `KeyboardInterrupt`**:

```python
try:
    nombre = inquirer.text(message="Nombre:").execute()
except KeyboardInterrupt:
    print("Cancelado.")
    raise SystemExit(130)
```

Con `raise_keyboard_interrupt=False`, Ctrl+C **omite** la pregunta (devuelve `None`) y Ctrl+D la interrumpe.

### 10.2 Preguntas que se pueden omitir

```python
telefono = inquirer.text(
    message="Teléfono (opcional):",
    mandatory=False,                         # Ctrl+Z omite la pregunta
    long_instruction="Ctrl+Z para omitir",
).execute()                                  # None si se omite
```

Con `mandatory=True` (por defecto), intentar omitir muestra `mandatory_message`.

### 10.3 Imprimir sin romper la pregunta

```python
from InquirerPy.utils import color_print

color_print([("#2CA02C", "✔ "), ("", "Configuración cargada")])
color_print([("bold red", "Error: "), ("", "sin conexión")])
```

---

## 11. Atajos de teclado

### 11.1 Teclas por defecto

| Acción | Teclas |
|---|---|
| Responder | Enter |
| Subir / bajar | ↑ / ↓, Ctrl+P / Ctrl+N (o `k`/`j` con `vi_mode=True`) |
| Marcar (listas) | Espacio |
| Marcar y bajar / subir | Tab / Shift+Tab |
| Marcar o desmarcar todas | Ctrl+R / Alt+R |
| Marcar todas | Ctrl+A / Alt+A |
| Omitir (si `mandatory=False`) | Ctrl+Z |
| Interrumpir | Ctrl+C |

### 11.2 Redefinir teclas

```python
inquirer.checkbox(
    message="Opciones:",
    choices=["A", "B", "C"],
    keybindings={
        "toggle-all-true": [{"key": "a"}],
        "toggle-all-false": [{"key": "n"}],
        "skip": [{"key": "escape"}],
        "interrupt": [{"key": "c-q"}],
    },
).execute()
```

### 11.3 Teclas con funciones propias

```python
pregunta = inquirer.select(message="Archivo:", choices=archivos)

@pregunta.register_kb("c-d")
def _(evento):
    """Ctrl+D: muestra detalles del elemento resaltado sin salir de la pregunta."""
    actual = pregunta.content_control.selection["value"]     # Elemento resaltado
    print(f"\n{detalles(actual)}")

resultado = pregunta.execute()
```

### 11.4 Modo *vi*

```python
inquirer.select(message="Opción:", choices=[...], vi_mode=True).execute()
```

---

## 12. Estilos

```python
from InquirerPy import get_style

estilo = get_style({
    "questionmark": "#1F78B4 bold",
    "answermark": "#2CA02C",
    "answer": "#2CA02C bold",
    "input": "#98c379",
    "question": "bold",
    "instruction": "#808080 italic",
    "long_instruction": "#808080",
    "pointer": "#1F78B4 bold",
    "checkbox": "#2CA02C",
    "marker": "#e5c07b",
    "fuzzy_prompt": "#c678dd",
    "fuzzy_match": "#c678dd bold",
    "fuzzy_border": "#4b5263",
    "separator": "#808080",
    "skipped": "#5c6370",
    "validator": "bg:#C00000 #ffffff",
}, style_override=False)          # False: completa el tema por defecto en lugar de sustituirlo

inquirer.select(message="Opción:", choices=["A", "B"], style=estilo).execute()
```

Símbolos:

```python
inquirer.text(message="Nombre:", qmark="›", amark="✔").execute()
inquirer.select(message="Opción:", choices=[...], pointer="→", marker="●", marker_pl=" ").execute()
```

Estilo global mediante variables de entorno (sin tocar el código):

```bash
export INQUIRERPY_STYLE_QUESTIONMARK="#1F78B4"
export INQUIRERPY_STYLE_POINTER="#1F78B4 bold"
```

---

## 13. API clásica: preguntas condicionales

```python
from InquirerPy import prompt

preguntas = [
    {"type": "confirm", "name": "bd", "message": "¿Usar base de datos?", "default": True},
    {"type": "list", "name": "motor", "message": "Motor:",
     "choices": ["PostgreSQL", "MySQL", "SQLite"],
     "when": lambda r: r["bd"]},
    {"type": "input", "name": "host", "message": "Host:", "default": "localhost",
     "when": lambda r: r.get("motor") in ("PostgreSQL", "MySQL")},
    {"type": "number", "name": "puerto", "message": "Puerto:",
     "default": lambda r: 5432 if r.get("motor") == "PostgreSQL" else 3306,
     "when": lambda r: r.get("motor") in ("PostgreSQL", "MySQL"), "filter": int},
]
respuestas = prompt(preguntas, style={"questionmark": "#1F78B4"}, vi_mode=False)
```

Las claves admitidas en cada diccionario son las mismas que los parámetros de la función `inquirer.*` correspondiente, más `type`, `name` y `when`.

---

## 14. Uso con asyncio

```python
import asyncio
from InquirerPy import inquirer

async def main():
    nombre = await inquirer.text(message="Nombre:").execute_async()
    entorno = await inquirer.select(message="Entorno:", choices=["dev", "pro"]).execute_async()
    await desplegar(nombre, entorno)

asyncio.run(main())
```

Con la API clásica: `from InquirerPy import prompt_async` y `await prompt_async(preguntas)`.

---

## 15. Ejemplo práctico completo

**Explorador de procesos y servicios simulado**: menú principal en bucle, búsqueda difusa sobre cientos de elementos, selección múltiple, preguntas numéricas, validación, confirmaciones con letras en español, atajos personalizados y estilo propio.

```python
"""
admin_servicios.py — Consola interactiva de administración de servicios (simulada).
Requiere: pip install InquirerPy
"""
import random
import sys
from dataclasses import dataclass, field

from InquirerPy import get_style, inquirer
from InquirerPy.base.control import Choice
from InquirerPy.separator import Separator
from InquirerPy.utils import color_print
from InquirerPy.validator import EmptyInputValidator

ESTILO = get_style({
    "questionmark": "#1F78B4 bold", "answermark": "#2CA02C", "answer": "#2CA02C bold",
    "pointer": "#1F78B4 bold", "instruction": "#808080 italic", "long_instruction": "#808080",
    "fuzzy_match": "#E07B00 bold", "fuzzy_prompt": "#1F78B4", "marker": "#2CA02C",
}, style_override=False)

COMUNES = {"style": ESTILO, "qmark": "›", "amark": "✔"}


# ------------------------------------------------------------------ Datos simulados
@dataclass
class Servicio:
    nombre: str
    equipo: str
    activo: bool = True
    replicas: int = 1
    etiquetas: list[str] = field(default_factory=list)

    @property
    def etiqueta(self) -> str:
        estado = "●" if self.activo else "○"
        return f"{estado} {self.nombre:<28} {self.equipo:<10} ×{self.replicas}"


def generar_servicios(n: int = 300) -> list[Servicio]:
    random.seed(7)
    dominios = ["pagos", "usuarios", "pedidos", "catalogo", "envios", "informes", "auth", "busqueda"]
    tipos = ["api", "worker", "cron", "gateway", "cache", "db-proxy"]
    equipos = ["backend", "datos", "plataforma", "producto"]
    return [Servicio(f"{random.choice(dominios)}-{random.choice(tipos)}-{i:03d}",
                     random.choice(equipos), random.random() > 0.15, random.randint(1, 6),
                     random.sample(["critico", "interno", "beta", "legacy"], k=random.randint(0, 2)))
            for i in range(n)]


SERVICIOS = generar_servicios()
POR_NOMBRE = {s.nombre: s for s in SERVICIOS}


# ------------------------------------------------------------------ Utilidades
def elegir_servicios(mensaje: str, multiple: bool = True, solo_activos: bool | None = None) -> list[Servicio]:
    candidatos = [s for s in SERVICIOS if solo_activos is None or s.activo == solo_activos]
    if not candidatos:
        color_print([("#E07B00", "No hay servicios en ese estado.")])
        return []
    # El valor de cada opción es el nombre (no la dataclass: InquirerPy la convertiría en dict)
    resultado = inquirer.fuzzy(
        message=mensaje,
        choices=[Choice(s.nombre, name=s.etiqueta) for s in candidatos],
        multiselect=multiple,
        max_height="50%",
        border=True,
        info=True,
        instruction="(escriba para filtrar)",
        long_instruction="Tab: marcar · Ctrl+A: marcar todos · Ctrl+Z: volver" if multiple else "Ctrl+Z: volver",
        mandatory=False,
        validate=(lambda r: len(r) > 0) if multiple else None,
        invalid_message="Marque al menos uno con Tab",
        transformer=(lambda r: f"{len(r)} servicio(s)") if multiple else None,
        **COMUNES,
    ).execute()
    if resultado is None:
        return []
    nombres = resultado if multiple else [resultado]
    return [POR_NOMBRE[n] for n in nombres]


def confirmar(mensaje: str) -> bool:
    return inquirer.confirm(message=mensaje, default=False, confirm_letter="s", reject_letter="n",
                            transformer=lambda r: "Sí" if r else "No", **COMUNES).execute()


# ------------------------------------------------------------------ Acciones
def accion_detalle() -> None:
    for s in elegir_servicios("Servicio:", multiple=False):
        color_print([("bold", f"\n{s.nombre}\n")])
        for clave, valor in [("Equipo", s.equipo), ("Estado", "activo" if s.activo else "detenido"),
                             ("Réplicas", s.replicas), ("Etiquetas", ", ".join(s.etiquetas) or "—")]:
            color_print([("#808080", f"  {clave:<10}"), ("", str(valor))])
        print()


def accion_detener() -> None:
    seleccion = elegir_servicios("Servicios a detener:", solo_activos=True)
    criticos = [s for s in seleccion if "critico" in s.etiquetas]
    if not seleccion:
        return
    if criticos:
        color_print([("#C00000 bold", f"⚠ {len(criticos)} servicio(s) marcados como críticos.")])
    if confirmar(f"¿Detener {len(seleccion)} servicio(s)?"):
        for s in seleccion:
            s.activo = False
        color_print([("#2CA02C", f"✔ {len(seleccion)} servicio(s) detenido(s).\n")])


def accion_iniciar() -> None:
    seleccion = elegir_servicios("Servicios a iniciar:", solo_activos=False)
    for s in seleccion:
        s.activo = True
    if seleccion:
        color_print([("#2CA02C", f"✔ {len(seleccion)} servicio(s) iniciado(s).\n")])


def accion_escalar() -> None:
    seleccion = elegir_servicios("Servicio a escalar:", multiple=False, solo_activos=True)
    if not seleccion:
        return
    s = seleccion[0]
    replicas = inquirer.number(
        message=f"Réplicas para {s.nombre} (actual: {s.replicas}):",
        default=s.replicas, min_allowed=1, max_allowed=20,
        validate=EmptyInputValidator("Indique un número"),
        long_instruction="↑/↓ para ajustar · máximo 20",
        filter=int, **COMUNES,
    ).execute()
    if replicas == s.replicas:
        color_print([("#808080", "Sin cambios.\n")])
        return
    s.replicas = replicas
    color_print([("#2CA02C", f"✔ {s.nombre} escalado a {replicas} réplica(s).\n")])


def accion_etiquetar() -> None:
    seleccion = elegir_servicios("Servicios a etiquetar:")
    if not seleccion:
        return
    actuales = {e for s in seleccion for e in s.etiquetas}
    etiquetas = inquirer.checkbox(
        message="Etiquetas:",
        choices=[Choice(e, enabled=e in actuales) for e in ["critico", "interno", "beta", "legacy"]],
        enabled_symbol="◉", disabled_symbol="○",
        keybindings={"toggle-all-true": [{"key": "a"}], "toggle-all-false": [{"key": "n"}]},
        long_instruction="Espacio: marcar · a: todas · n: ninguna",
        **COMUNES,
    ).execute()
    for s in seleccion:
        s.etiquetas = list(etiquetas)
    color_print([("#2CA02C", f"✔ Etiquetas actualizadas en {len(seleccion)} servicio(s).\n")])


def accion_resumen() -> None:
    activos = sum(s.activo for s in SERVICIOS)
    por_equipo: dict[str, int] = {}
    for s in SERVICIOS:
        por_equipo[s.equipo] = por_equipo.get(s.equipo, 0) + s.replicas * s.activo
    color_print([("bold", "\nResumen\n")])
    color_print([("", f"  Servicios: {len(SERVICIOS)} · "), ("#2CA02C", f"{activos} activos"),
                 ("", " · "), ("#C00000", f"{len(SERVICIOS) - activos} detenidos")])
    for equipo, replicas in sorted(por_equipo.items()):
        color_print([("#808080", f"  {equipo:<12}"), ("", f"{replicas:>4} réplicas activas")])
    print()


# ------------------------------------------------------------------ Menú principal
MENU = [
    Choice(accion_resumen, name="Resumen"),
    Choice(accion_detalle, name="Ver detalle de un servicio"),
    Separator(),
    Choice(accion_iniciar, name="Iniciar servicios"),
    Choice(accion_detener, name="Detener servicios"),
    Choice(accion_escalar, name="Escalar un servicio"),
    Choice(accion_etiquetar, name="Etiquetar servicios"),
    Separator(),
    Choice(None, name="Salir"),
]


def main() -> int:
    if not sys.stdin.isatty():
        print("Se requiere una terminal interactiva.", file=sys.stderr)
        return 2
    color_print([("#1F78B4 bold", "Consola de servicios"), ("#808080", f"  ({len(SERVICIOS)} servicios)\n")])
    try:
        while True:
            accion = inquirer.select(message="¿Qué desea hacer?", choices=MENU, cycle=True,
                                     pointer="➜", **COMUNES).execute()
            if accion is None:
                return 0
            accion()
    except KeyboardInterrupt:
        color_print([("#E07B00", "\nSesión finalizada.")])
        return 130


if __name__ == "__main__":
    sys.exit(main())
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'inquirerpy'` | Importación en minúsculas. | `from InquirerPy import inquirer`. |
| La pregunta no aparece | Falta `.execute()`. | `inquirer.text(...).execute()`. |
| `KeyboardInterrupt` sin capturar al pulsar Ctrl+C | Comportamiento por defecto. | `try/except KeyboardInterrupt` o `raise_keyboard_interrupt=False`. |
| En `fuzzy` la tecla Espacio no marca | Espacio se escribe en el buscador. | Use **Tab** para marcar. |
| Se recibe un `dict` en lugar del objeto elegido | El `value` de la `Choice` era una `dataclass`. | Use un identificador como valor y recupere el objeto. |
| `number` devuelve texto | Comportamiento por defecto. | `filter=int` o `filter=float`. |
| Ctrl+Z no omite la pregunta | `mandatory=True` por defecto. | `mandatory=False`. |
| La validación de una lista múltiple falla | La función recibe una lista, no texto. | Escriba el validador para listas. |
| El estilo sustituye todos los colores | `get_style` usa `style_override=True` por defecto. | `style_override=False`. |
| `RuntimeError: This event loop is already running` | `execute()` dentro de código `async`. | `await ...execute_async()`. |
| Errores en CI o con tuberías | No hay TTY. | Compruebe `sys.stdin.isatty()` y ofrezca argumentos alternativos. |

---

## 17. Buenas prácticas

- Use la API de objetos (`inquirer.*`) para lógica compleja y la API clásica (`prompt`) para cuestionarios definidos como datos.
- Prefiera `fuzzy` frente a `select` a partir de unas 15-20 opciones.
- Use `Choice(value=objeto, name=texto)` para trabajar con objetos en lugar de textos.
- Distinga `filter` (valor devuelto) de `transformer` (texto mostrado).
- Añada `long_instruction` con las teclas disponibles, sobre todo en selecciones múltiples.
- Defina el estilo y los parámetros comunes una vez y reutilícelos (`**COMUNES`).
- Marque como `mandatory=False` las preguntas opcionales e indique cómo omitirlas.
- Capture `KeyboardInterrupt` en el punto de entrada para terminar de forma limpia.
- Combine InquirerPy con Typer o Click para que la herramienta también funcione de forma no interactiva.

---

## 18. Referencia rápida

```python
inquirer.text(message, default, completer, multiline, validate, invalid_message, filter, transformer)
inquirer.secret(message, validate=PasswordValidator(...))
inquirer.number(message, min_allowed, max_allowed, float_allowed, default, filter=int)
inquirer.confirm(message, default, confirm_letter="s", reject_letter="n")
inquirer.select(message, choices, default, multiselect, border, cycle, max_height, pointer)
inquirer.rawlist(...) · inquirer.expand(choices=[ExpandChoice(v, name, key)])
inquirer.checkbox(message, choices, enabled_symbol, disabled_symbol, validate)
inquirer.fuzzy(message, choices, multiselect, info, border, match_exact, max_height)
inquirer.filepath(message, only_files, only_directories, validate=PathValidator(is_file=True))
.execute() · await .execute_async()

Choice(value, name=, enabled=) · Separator("texto")
Comunes: qmark amark instruction long_instruction style vi_mode keybindings mandatory
         raise_keyboard_interrupt transformer filter validate invalid_message
prompt([{"type": "input|password|number|confirm|list|rawlist|expand|checkbox|fuzzy|filepath",
         "name", "message", "when", ...}])
get_style({...}, style_override=False) · color_print([(estilo, texto), ...])
pregunta.register_kb("c-d")(funcion)
```

---

## 19. Ejercicios propuestos

1. Cree un selector de archivos del proyecto con `fuzzy` que liste todos los archivos `.py` recursivamente y muestre el número de líneas del elegido.
2. Desarrolle un formulario de alta de empleados con `prompt` (API clásica), incluyendo preguntas condicionales según el tipo de contrato.
3. Implemente un atajo `Ctrl+D` en una lista de productos que muestre el precio del elemento resaltado sin salir de la pregunta.
4. Añada al ejemplo completo una acción «Buscar por etiqueta» que filtre primero con `checkbox` y después con `fuzzy`.
5. Construya un configurador de servidores con `number` (CPU, RAM) y `expand` (acción ante conflicto), mostrando un resumen final.

---

## 20. Referencias

- Documentación oficial: https://inquirerpy.readthedocs.io/
- Repositorio: https://github.com/kazhala/InquirerPy
- Ejemplos: https://github.com/kazhala/InquirerPy/tree/master/examples
- prompt_toolkit: https://python-prompt-toolkit.readthedocs.io/
