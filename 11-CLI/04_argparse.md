# Manual de uso: argparse

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Parsear argumentos de línea de comandos sin dependencias
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Importación](#2-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer programa](#4-primer-programa)
5. [Argumentos posicionales](#5-argumentos-posicionales)
6. [Opciones](#6-opciones)
7. [Acciones (`action`)](#7-acciones-action)
8. [Tipos, valores permitidos y número de valores](#8-tipos-valores-permitidos-y-número-de-valores)
9. [Grupos de argumentos y exclusión mutua](#9-grupos-de-argumentos-y-exclusión-mutua)
10. [Subcomandos](#10-subcomandos)
11. [Personalizar la ayuda](#11-personalizar-la-ayuda)
12. [Validación y errores](#12-validación-y-errores)
13. [Configuración desde archivos y variables de entorno](#13-configuración-desde-archivos-y-variables-de-entorno)
14. [Técnicas avanzadas](#14-técnicas-avanzadas)
15. [Pruebas](#15-pruebas)
16. [Distribución](#16-distribución)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**argparse** es el módulo de la librería estándar de Python para interpretar los argumentos de la línea de comandos (`sys.argv`). Convierte y valida los valores, genera la ayuda (`-h`/`--help`) y los mensajes de error, y admite subcomandos al estilo de `git commit` o `pip install`.

**Ventajas:**
- Incluido en Python: **sin dependencias**, ideal para scripts que se distribuyen tal cual.
- Estable y compatible con todas las versiones actuales de Python.
- Ayuda y mensajes de error automáticos.
- Subcomandos, grupos de argumentos y opciones mutuamente excluyentes.
- Muy configurable mediante acciones, tipos y formateadores personalizados.

**Limitaciones:**
- Más verboso que Typer o Click.
- No usa las anotaciones de tipo de las funciones.
- Sin colores en la ayuda hasta Python 3.14.
- No lee variables de entorno ni archivos de configuración de forma nativa (se implementa a mano).

| Si necesita…                                     | Opción       |
| ------------------------------------------------ | ------------ |
| Script sin dependencias externas                 | **argparse** |
| CLI moderna con *type hints*                     | Typer        |
| CLI grande y extensible con plugins              | Click        |
| Exponer funciones existentes sin escribir la CLI | Fire         |

---

## 2. Importación

```python
import argparse
```

No requiere instalación. Las funciones descritas indican la versión mínima de Python cuando no están disponibles en todas.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `ArgumentParser` | Objeto que define la interfaz y analiza los argumentos. |
| `add_argument()` | Declara un argumento posicional u opción. |
| `parse_args()` | Analiza `sys.argv[1:]` y devuelve un `Namespace`. |
| `Namespace` | Objeto con un atributo por argumento (`args.salida`). |
| Argumento posicional | Se identifica por su posición: `copiar origen destino`. Obligatorio por defecto. |
| Opción | Empieza por `-` o `--`: `--salida informe.csv`. Opcional por defecto. |
| Acción (`action`) | Qué hacer con el valor: guardarlo, contar, añadir a una lista… |
| `dest` | Nombre del atributo en el `Namespace`. |
| Subparser | Analizador independiente para cada subcomando. |

---

## 4. Primer programa

```python
import argparse

parser = argparse.ArgumentParser(description="Saluda a una persona.")
parser.add_argument("nombre", help="nombre de la persona")
parser.add_argument("-n", "--veces", type=int, default=1, help="número de saludos (por defecto: %(default)s)")
parser.add_argument("-v", "--verbose", action="store_true", help="modo detallado")
args = parser.parse_args()

for _ in range(args.veces):
    print(f"Hola, {args.nombre}")
```

```bash
python hola.py Ana -n 2
python hola.py -h
```

```
usage: hola.py [-h] [-n VECES] [-v] nombre

Saluda a una persona.

positional arguments:
  nombre               nombre de la persona

options:
  -h, --help           show this help message and exit
  -n, --veces VECES    número de saludos (por defecto: 1)
  -v, --verbose        modo detallado
```

**Estructura recomendada** (facilita las pruebas):

```python
def crear_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(...)
    ...
    return parser

def main(argv: list[str] | None = None) -> int:
    args = crear_parser().parse_args(argv)
    ...
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 5. Argumentos posicionales

```python
parser.add_argument("origen", help="archivo de origen")
parser.add_argument("destino", nargs="?", default=".", help="carpeta de destino (opcional)")
parser.add_argument("archivos", nargs="+", help="uno o más archivos")
```

Los posicionales se asignan en el orden en que se declaran. El nombre del primer argumento de `add_argument` es el atributo del `Namespace`.

```python
parser.add_argument("archivo_entrada", metavar="ENTRADA")    # args.archivo_entrada; en la ayuda: ENTRADA
```

---

## 6. Opciones

### 6.1 Nombres corto y largo

```python
parser.add_argument("-o", "--salida")                # args.salida
parser.add_argument("--max-filas", type=int)         # args.max_filas (guion → guion bajo)
parser.add_argument("-u", "--user", dest="usuario")  # args.usuario
```

### 6.2 Obligatorias y valores por defecto

```python
parser.add_argument("--token", required=True, help="token de acceso")
parser.add_argument("--puerto", type=int, default=8080)
```

> Las opciones obligatorias son poco habituales en el estilo Unix: si un dato es imprescindible, suele ser un **posicional**.

### 6.3 Valores por defecto globales

```python
parser.set_defaults(modo="normal", reintentos=3)
```

### 6.4 Prefijos y abreviaturas

argparse acepta por defecto abreviaturas no ambiguas (`--verb` por `--verbose`). Para desactivarlo:

```python
parser = argparse.ArgumentParser(allow_abbrev=False)
```

---

## 7. Acciones (`action`)

| Acción | Efecto | Ejemplo |
|---|---|---|
| `"store"` (por defecto) | Guarda el valor. | `--salida x.csv` |
| `"store_const"` | Guarda `const`. | `--rapido` → `modo="rapido"` |
| `"store_true"` / `"store_false"` | Guarda `True` / `False`. | `--verbose` |
| `"append"` | Añade cada aparición a una lista. | `-e a -e b` → `['a', 'b']` |
| `"append_const"` | Añade `const` a una lista. | Varias banderas a una misma lista. |
| `"extend"` | Extiende la lista con varios valores. | `--tags a b --tags c` → `['a', 'b', 'c']` |
| `"count"` | Cuenta apariciones. | `-vvv` → `3` |
| `"version"` | Muestra la versión y sale. | `--version` |
| `"help"` | Muestra la ayuda y sale. | `-h` |
| `argparse.BooleanOptionalAction` | Crea `--x` y `--no-x`. | `--color` / `--no-color` |

```python
parser.add_argument("-v", "--verbose", action="count", default=0)
parser.add_argument("-e", "--etiqueta", action="append", default=[])
parser.add_argument("--tags", nargs="+", action="extend", default=[])
parser.add_argument("--color", action=argparse.BooleanOptionalAction, default=True)
parser.add_argument("--version", action="version", version="%(prog)s 1.2.0")
parser.add_argument("--json", dest="formato", action="store_const", const="json", default="texto")
```

### 7.1 Acción personalizada

```python
class AccionClaveValor(argparse.Action):
    """Convierte --set clave=valor en un diccionario."""

    def __call__(self, parser, namespace, valores, option_string=None):
        destino = getattr(namespace, self.dest, None) or {}
        for par in valores:
            if "=" not in par:
                parser.error(f"{option_string}: se esperaba clave=valor, recibido {par!r}")
            clave, valor = par.split("=", 1)
            destino[clave] = valor
        setattr(namespace, self.dest, destino)

parser.add_argument("--set", nargs="+", action=AccionClaveValor, metavar="CLAVE=VALOR")
# --set host=localhost puerto=5432  →  {'host': 'localhost', 'puerto': '5432'}
```

---

## 8. Tipos, valores permitidos y número de valores

### 8.1 `type`

`type` es cualquier función que reciba un texto y devuelva el valor convertido:

```python
from pathlib import Path
from datetime import date

parser.add_argument("--limite", type=int)
parser.add_argument("--ratio", type=float)
parser.add_argument("--config", type=Path)
parser.add_argument("--desde", type=date.fromisoformat)          # 2026-01-31
parser.add_argument("--entrada", type=argparse.FileType("r", encoding="utf-8"))   # '-' = stdin
```

> **Atención:** `type=bool` no funciona como se espera (`bool("False")` es `True`). Use `store_true` o `BooleanOptionalAction`.

### 8.2 Tipos con validación

```python
def entero_positivo(texto: str) -> int:
    try:
        valor = int(texto)
    except ValueError:
        raise argparse.ArgumentTypeError(f"{texto!r} no es un entero")
    if valor <= 0:
        raise argparse.ArgumentTypeError(f"debe ser mayor que 0 (recibido {valor})")
    return valor

def archivo_existente(texto: str) -> Path:
    ruta = Path(texto)
    if not ruta.is_file():
        raise argparse.ArgumentTypeError(f"no existe el archivo {texto!r}")
    return ruta

parser.add_argument("--hilos", type=entero_positivo, default=4)
parser.add_argument("entrada", type=archivo_existente)
```

```
error: argument --hilos: debe ser mayor que 0 (recibido -2)
```

### 8.3 `choices`

```python
parser.add_argument("--formato", choices=["csv", "json", "xlsx"], default="csv")
parser.add_argument("--nivel", type=int, choices=range(1, 6))
```

Con `Enum`:

```python
from enum import Enum

class Entorno(Enum):
    dev = "dev"
    pro = "pro"

parser.add_argument("--entorno", type=Entorno, choices=list(Entorno),
                    metavar="{" + ",".join(e.value for e in Entorno) + "}")
```

### 8.4 `nargs`

| Valor | Significado | Resultado |
|---|---|---|
| `N` (entero) | Exactamente N valores. | Lista |
| `"?"` | 0 o 1 valor (usa `default` o `const`). | Valor |
| `"*"` | 0 o más valores. | Lista |
| `"+"` | 1 o más valores. | Lista |
| `argparse.REMAINDER` | El resto de la línea, sin analizar. | Lista |

```python
parser.add_argument("--punto", nargs=2, type=float, metavar=("X", "Y"))
parser.add_argument("--log", nargs="?", const="app.log", default=None)
# (sin --log) → None · --log → 'app.log' · --log x.log → 'x.log'
```

---

## 9. Grupos de argumentos y exclusión mutua

### 9.1 Grupos para la ayuda

```python
red = parser.add_argument_group("conexión", "Parámetros del servidor")
red.add_argument("--host", default="localhost")
red.add_argument("--puerto", type=int, default=5432)

salida = parser.add_argument_group("salida")
salida.add_argument("-o", "--salida")
salida.add_argument("--formato", choices=["csv", "json"])
```

### 9.2 Opciones mutuamente excluyentes

```python
grupo = parser.add_mutually_exclusive_group(required=True)
grupo.add_argument("--id", type=int)
grupo.add_argument("--nombre")
```

```
error: argument --nombre: not allowed with argument --id
```

```python
verbosidad = parser.add_mutually_exclusive_group()
verbosidad.add_argument("-q", "--quiet", action="store_true")
verbosidad.add_argument("-v", "--verbose", action="count", default=0)
```

---

## 10. Subcomandos

### 10.1 Estructura básica

```python
parser = argparse.ArgumentParser(prog="gestor")
parser.add_argument("-v", "--verbose", action="store_true")      # Opción global

subparsers = parser.add_subparsers(dest="comando", required=True, title="comandos")

p_crear = subparsers.add_parser("crear", help="crea un usuario")
p_crear.add_argument("nombre")
p_crear.add_argument("--admin", action="store_true")

p_listar = subparsers.add_parser("listar", aliases=["ls"], help="lista los usuarios")
p_listar.add_argument("--activos", action="store_true")

args = parser.parse_args()
```

```bash
gestor crear ana --admin
gestor -v ls --activos
```

### 10.2 Asociar una función a cada subcomando (recomendado)

```python
def cmd_crear(args): ...
def cmd_listar(args): ...

p_crear.set_defaults(func=cmd_crear)
p_listar.set_defaults(func=cmd_listar)

args = parser.parse_args()
args.func(args)
```

### 10.3 Opciones compartidas con `parents`

```python
comun = argparse.ArgumentParser(add_help=False)
comun.add_argument("--formato", choices=["tabla", "json"], default="tabla")
comun.add_argument("--limite", type=int, default=50)

subparsers.add_parser("usuarios", parents=[comun], help="lista usuarios")
subparsers.add_parser("pedidos", parents=[comun], help="lista pedidos")
```

### 10.4 Subcomandos anidados

```python
p_bd = subparsers.add_parser("bd", help="operaciones de base de datos")
sub_bd = p_bd.add_subparsers(dest="accion", required=True)
sub_bd.add_parser("migrar").set_defaults(func=cmd_migrar)
sub_bd.add_parser("respaldar").set_defaults(func=cmd_respaldar)
# gestor bd migrar
```

### 10.5 Subcomandos obsoletos (Python 3.13+)

```python
subparsers.add_parser("antiguo", deprecated=True)         # Muestra un aviso al usarlo
parser.add_argument("--viejo", deprecated=True)
```

---

## 11. Personalizar la ayuda

### 11.1 Parámetros de `ArgumentParser`

```python
parser = argparse.ArgumentParser(
    prog="gestor",
    description="Gestor de inventario.",
    epilog="Ejemplo: gestor importar productos.csv --formato csv",
    formatter_class=argparse.RawDescriptionHelpFormatter,
    add_help=True,
    allow_abbrev=False,
    fromfile_prefix_chars="@",
    exit_on_error=True,
)
```

### 11.2 Formateadores

| Clase | Efecto |
|---|---|
| `HelpFormatter` | Por defecto: reajusta todos los textos. |
| `RawDescriptionHelpFormatter` | Respeta los saltos de línea de `description` y `epilog`. |
| `RawTextHelpFormatter` | Respeta los saltos en todos los textos de ayuda. |
| `ArgumentDefaultsHelpFormatter` | Añade «(default: …)» a cada opción. |
| `MetavarTypeHelpFormatter` | Usa el nombre del tipo como *metavar*. |

Combinar dos formateadores:

```python
class Formato(argparse.ArgumentDefaultsHelpFormatter, argparse.RawDescriptionHelpFormatter):
    pass

parser = argparse.ArgumentParser(formatter_class=Formato)
```

### 11.3 Variables en los textos de ayuda

```python
parser.add_argument("--hilos", type=int, default=4,
                    help="número de hilos (por defecto: %(default)s, tipo: %(type)s)")
```

Variables disponibles: `%(prog)s`, `%(default)s`, `%(type)s`, `%(choices)s`, `%(dest)s`, `%(metavar)s`. Para escribir un `%` literal, use `%%`.

### 11.4 Ocultar argumentos

```python
parser.add_argument("--depurar", action="store_true", help=argparse.SUPPRESS)
```

### 11.5 Traducir los textos fijos

Los textos internos («usage:», «options:», «error:») vienen en inglés. Se pueden traducir con `gettext` o, de forma sencilla, cambiando los títulos:

```python
parser = argparse.ArgumentParser(add_help=False)
parser.add_argument("-h", "--help", action="help", help="muestra esta ayuda y sale")
parser._optionals.title = "opciones"
parser._positionals.title = "argumentos"
```

### 11.6 Colores y sugerencias (Python 3.14+)

```python
parser = argparse.ArgumentParser(color=True, suggest_on_error=True)
# error: argument comando: invalid choice: 'crar', maybe you meant 'crear'?
```

---

## 12. Validación y errores

### 12.1 `parser.error()`

Muestra el *usage*, el mensaje y termina con código **2**:

```python
args = parser.parse_args()
if args.desde and args.hasta and args.desde > args.hasta:
    parser.error("--desde no puede ser posterior a --hasta")
```

### 12.2 Códigos de salida

| Situación | Código |
|---|---|
| `-h` / `--version` | 0 |
| Error de argumentos (`parser.error`) | 2 |
| Errores de la aplicación | Los que defina (1 habitualmente) |

### 12.3 Capturar los errores en lugar de salir (Python 3.9+)

```python
parser = argparse.ArgumentParser(exit_on_error=False)
try:
    args = parser.parse_args(["--hilos", "x"])
except argparse.ArgumentError as e:
    print(f"Error: {e}")
```

> `exit_on_error=False` no cubre todos los casos (p. ej. argumentos obligatorios ausentes en algunas versiones). Para un control total, herede de `ArgumentParser` y redefina `error()`.

### 12.4 Argumentos desconocidos

```python
args, resto = parser.parse_known_args()      # No falla con argumentos no declarados
```

---

## 13. Configuración desde archivos y variables de entorno

### 13.1 Argumentos desde un archivo (`@archivo`)

```python
parser = argparse.ArgumentParser(fromfile_prefix_chars="@")
```

```
# opciones.txt (un argumento por línea)
--host
db.ejemplo.com
--puerto
5432
```

```bash
python app.py @opciones.txt --verbose
```

### 13.2 Variables de entorno como valor por defecto

```python
import os

parser.add_argument("--api-key", default=os.environ.get("MIAPP_API_KEY"),
                    help="clave de la API (o variable MIAPP_API_KEY)")
args = parser.parse_args()
if not args.api_key:
    parser.error("falta --api-key o la variable MIAPP_API_KEY")
```

### 13.3 Archivo TOML de configuración

```python
import tomllib

def cargar_config(argv=None):
    previo = argparse.ArgumentParser(add_help=False)
    previo.add_argument("--config", type=Path, default=Path("app.toml"))
    conocidos, resto = previo.parse_known_args(argv)

    parser = argparse.ArgumentParser(parents=[previo])
    parser.add_argument("--host", default="localhost")
    parser.add_argument("--puerto", type=int, default=8080)

    if conocidos.config.is_file():
        with conocidos.config.open("rb") as f:
            parser.set_defaults(**tomllib.load(f))       # El archivo sustituye los valores por defecto
    return parser.parse_args(resto)                      # La línea de comandos tiene prioridad
```

**Prioridad resultante:** línea de comandos > archivo > variable de entorno > valor por defecto.

---

## 14. Técnicas avanzadas

### 14.1 Opciones y posicionales intercalados

```python
args = parser.parse_intermixed_args()      # Permite: app.py a.txt --verbose b.txt
```

### 14.2 Pasar el resto de argumentos a otro programa

```python
parser.add_argument("comando", nargs=argparse.REMAINDER)
# app.py ejecutar -- ls -la  → comando = ['--', 'ls', '-la']
```

### 14.3 Convertir el `Namespace`

```python
datos = vars(args)                          # Diccionario
```

### 14.4 Configurar `logging` según la verbosidad

```python
import logging

NIVELES = [logging.WARNING, logging.INFO, logging.DEBUG]
logging.basicConfig(level=NIVELES[min(args.verbose, 2)], format="%(levelname)s %(message)s")
```

### 14.5 Autocompletado en la shell

argparse no incluye autocompletado, pero el paquete externo `argcomplete` lo añade:

```python
# PYTHON_ARGCOMPLETE_OK
import argcomplete
argcomplete.autocomplete(parser)
args = parser.parse_args()
```

---

## 15. Pruebas

Pase la lista de argumentos explícitamente a `parse_args`:

```python
import pytest
from miapp.cli import crear_parser, main

def test_valores_por_defecto():
    args = crear_parser().parse_args(["datos.csv"])
    assert args.formato == "csv"
    assert args.verbose == 0

def test_error_de_argumentos(capsys):
    with pytest.raises(SystemExit) as exc:
        crear_parser().parse_args(["--hilos", "-1"])
    assert exc.value.code == 2
    assert "mayor que 0" in capsys.readouterr().err

def test_main(tmp_path):
    archivo = tmp_path / "datos.csv"
    archivo.write_text("a,b\n1,2\n")
    assert main([str(archivo)]) == 0
```

---

## 16. Distribución

```toml
[project]
name = "miapp"
version = "1.0.0"

[project.scripts]
miapp = "miapp.cli:main"
```

`main` debe devolver el código de salida (o `None`). Ejecución como módulo con `src/miapp/__main__.py`:

```python
from miapp.cli import main
raise SystemExit(main())
```

Para scripts de un solo archivo con versión de Python declarada (PEP 723):

```python
# /// script
# requires-python = ">=3.11"
# ///
```

---

## 17. Ejemplo práctico completo

**Analizador de logs de acceso** (formato Apache/Nginx) con subcomandos, opciones compartidas mediante `parents`, tipos validados, grupos excluyentes, configuración por variable de entorno, salida en tabla, JSON o CSV, y gestión de errores con códigos de salida.

```python
"""
logstat.py — Estadísticas de logs de acceso (formato combinado Apache/Nginx).
Solo usa la librería estándar.
Uso: python logstat.py --help
"""
import argparse
import csv
import gzip
import json
import logging
import os
import re
import sys
from collections import Counter
from datetime import datetime
from pathlib import Path
from typing import Iterator

__version__ = "1.0.0"
log = logging.getLogger("logstat")

PATRON = re.compile(
    r'(?P<ip>\S+) \S+ \S+ \[(?P<fecha>[^\]]+)\] "(?P<metodo>\S+) (?P<ruta>\S+) [^"]*" '
    r'(?P<estado>\d{3}) (?P<bytes>\d+|-)(?: "[^"]*" "(?P<agente>[^"]*)")?'
)
FORMATO_FECHA = "%d/%b/%Y:%H:%M:%S %z"


# ------------------------------------------------------------------ Tipos de argumento
def archivo_log(texto: str) -> Path:
    ruta = Path(texto)
    if not ruta.is_file():
        raise argparse.ArgumentTypeError(f"no existe el archivo {texto!r}")
    return ruta


def entero_positivo(texto: str) -> int:
    if not texto.isdigit() or int(texto) == 0:
        raise argparse.ArgumentTypeError(f"se esperaba un entero positivo, recibido {texto!r}")
    return int(texto)


def fecha(texto: str) -> datetime:
    try:
        return datetime.fromisoformat(texto).astimezone()
    except ValueError:
        raise argparse.ArgumentTypeError(f"fecha no válida {texto!r} (use AAAA-MM-DD[THH:MM])")


def rango_estado(texto: str) -> range:
    """Acepta '404', '4xx' o '400-499'."""
    if re.fullmatch(r"[1-5]xx", texto):
        base = int(texto[0]) * 100
        return range(base, base + 100)
    if m := re.fullmatch(r"(\d{3})-(\d{3})", texto):
        return range(int(m[1]), int(m[2]) + 1)
    if re.fullmatch(r"\d{3}", texto):
        return range(int(texto), int(texto) + 1)
    raise argparse.ArgumentTypeError(f"estado no válido {texto!r} (ej.: 404, 5xx, 400-499)")


# ------------------------------------------------------------------ Lectura y filtrado
def leer(archivos: list[Path]) -> Iterator[dict]:
    for ruta in archivos:
        abrir = gzip.open if ruta.suffix == ".gz" else open
        with abrir(ruta, "rt", encoding="utf-8", errors="replace") as f:
            for n, linea in enumerate(f, 1):
                if m := PATRON.match(linea):
                    d = m.groupdict()
                    d["fecha"] = datetime.strptime(d["fecha"], FORMATO_FECHA)
                    d["estado"] = int(d["estado"])
                    d["bytes"] = 0 if d["bytes"] == "-" else int(d["bytes"])
                    yield d
                else:
                    log.debug("%s:%d línea no reconocida", ruta.name, n)


def filtrar(registros: Iterator[dict], args: argparse.Namespace) -> Iterator[dict]:
    for r in registros:
        if args.desde and r["fecha"] < args.desde:
            continue
        if args.hasta and r["fecha"] > args.hasta:
            continue
        if args.estado and r["estado"] not in args.estado:
            continue
        if args.metodo and r["metodo"] not in args.metodo:
            continue
        yield r


# ------------------------------------------------------------------ Salida
def emitir(filas: list[dict], formato: str, titulo: str = "") -> None:
    if not filas:
        print("Sin resultados.", file=sys.stderr)
        return
    if formato == "json":
        json.dump(filas, sys.stdout, ensure_ascii=False, indent=2, default=str)
        print()
    elif formato == "csv":
        w = csv.DictWriter(sys.stdout, fieldnames=list(filas[0]))
        w.writeheader()
        w.writerows(filas)
    else:
        columnas = list(filas[0])
        anchos = {c: max(len(c), *(len(str(f[c])) for f in filas)) for c in columnas}
        if titulo:
            print(f"\n{titulo}\n{'=' * len(titulo)}")
        print("  ".join(c.upper().ljust(anchos[c]) for c in columnas))
        print("  ".join("-" * anchos[c] for c in columnas))
        for f in filas:
            print("  ".join(str(f[c]).rjust(anchos[c]) if isinstance(f[c], (int, float))
                            else str(f[c]).ljust(anchos[c]) for c in columnas))


# ------------------------------------------------------------------ Comandos
def cmd_resumen(args) -> int:
    total = errores = bytes_ = 0
    ips, estados = set(), Counter()
    primera = ultima = None
    for r in filtrar(leer(args.archivos), args):
        total += 1
        bytes_ += r["bytes"]
        ips.add(r["ip"])
        estados[f"{r['estado'] // 100}xx"] += 1
        errores += r["estado"] >= 500
        primera = min(primera or r["fecha"], r["fecha"])
        ultima = max(ultima or r["fecha"], r["fecha"])
    if not total:
        print("No hay peticiones que cumplan los filtros.", file=sys.stderr)
        return 1
    filas = [
        {"métrica": "Peticiones", "valor": total},
        {"métrica": "IPs únicas", "valor": len(ips)},
        {"métrica": "Datos enviados (MB)", "valor": round(bytes_ / 1_048_576, 2)},
        {"métrica": "Errores 5xx (%)", "valor": round(errores * 100 / total, 2)},
        *({"métrica": f"Estados {k}", "valor": v} for k, v in sorted(estados.items())),
        {"métrica": "Desde", "valor": primera.isoformat(sep=" ")},
        {"métrica": "Hasta", "valor": ultima.isoformat(sep=" ")},
    ]
    emitir(filas, args.formato, "Resumen")
    return 0


def cmd_top(args) -> int:
    clave = {"ip": "ip", "ruta": "ruta", "agente": "agente", "estado": "estado"}[args.por]
    contador = Counter(r[clave] or "-" for r in filtrar(leer(args.archivos), args))
    total = sum(contador.values())
    filas = [{args.por: k, "peticiones": n, "%": round(n * 100 / total, 1)}
             for k, n in contador.most_common(args.limite)]
    emitir(filas, args.formato, f"Top {args.limite} por {args.por}")
    return 0 if filas else 1


def cmd_horas(args) -> int:
    contador = Counter(r["fecha"].hour for r in filtrar(leer(args.archivos), args))
    if not contador:
        print("Sin datos.", file=sys.stderr)
        return 1
    maximo = max(contador.values())
    if args.formato != "tabla":
        emitir([{"hora": h, "peticiones": contador.get(h, 0)} for h in range(24)], args.formato)
        return 0
    print("\nPeticiones por hora\n===================")
    for h in range(24):
        n = contador.get(h, 0)
        print(f"{h:02d}h {'█' * round(n * args.ancho / maximo):<{args.ancho}} {n}")
    return 0


# ------------------------------------------------------------------ Parser
def crear_parser() -> argparse.ArgumentParser:
    comun = argparse.ArgumentParser(add_help=False)
    comun.add_argument("archivos", nargs="+", type=archivo_log, metavar="LOG",
                       help="archivos de log (admite .gz)")

    filtros = comun.add_argument_group("filtros")
    filtros.add_argument("--desde", type=fecha, metavar="FECHA", help="incluir desde esta fecha")
    filtros.add_argument("--hasta", type=fecha, metavar="FECHA", help="incluir hasta esta fecha")
    filtros.add_argument("-s", "--estado", type=rango_estado, metavar="ESTADO",
                         help="código o rango: 404, 5xx, 400-499")
    filtros.add_argument("-m", "--metodo", action="append", type=str.upper,
                         choices=["GET", "POST", "PUT", "PATCH", "DELETE", "HEAD", "OPTIONS"],
                         help="método HTTP (repetible)")

    salida = comun.add_argument_group("salida")
    salida.add_argument("-f", "--formato", choices=["tabla", "json", "csv"],
                        default=os.environ.get("LOGSTAT_FORMATO", "tabla"),
                        help="formato de salida (por defecto: %(default)s; variable LOGSTAT_FORMATO)")

    parser = argparse.ArgumentParser(
        prog="logstat",
        description="Estadísticas de logs de acceso en formato combinado (Apache/Nginx).",
        epilog="Ejemplos:\n"
               "  logstat resumen access.log\n"
               "  logstat top access.log* --por ruta -s 4xx -n 20\n"
               "  logstat horas access.log --desde 2026-11-01 -f csv > horas.csv",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        allow_abbrev=False,
    )
    parser.add_argument("--version", action="version", version=f"%(prog)s {__version__}")
    verbosidad = parser.add_mutually_exclusive_group()
    verbosidad.add_argument("-v", "--verbose", action="count", default=0, help="más detalle (-vv)")
    verbosidad.add_argument("-q", "--quiet", action="store_true", help="solo errores")

    sub = parser.add_subparsers(dest="comando", required=True, title="comandos", metavar="COMANDO")

    p = sub.add_parser("resumen", parents=[comun], help="métricas generales")
    p.set_defaults(func=cmd_resumen)

    p = sub.add_parser("top", parents=[comun], help="elementos más frecuentes")
    p.add_argument("--por", choices=["ip", "ruta", "agente", "estado"], default="ip",
                   help="campo por el que agrupar (por defecto: %(default)s)")
    p.add_argument("-n", "--limite", type=entero_positivo, default=10,
                   help="número de resultados (por defecto: %(default)s)")
    p.set_defaults(func=cmd_top)

    p = sub.add_parser("horas", parents=[comun], help="histograma de peticiones por hora")
    p.add_argument("--ancho", type=entero_positivo, default=40, help="ancho de las barras")
    p.set_defaults(func=cmd_horas)
    return parser


def main(argv: list[str] | None = None) -> int:
    parser = crear_parser()
    args = parser.parse_args(argv)

    if args.desde and args.hasta and args.desde > args.hasta:
        parser.error("--desde no puede ser posterior a --hasta")

    nivel = logging.ERROR if args.quiet else [logging.WARNING, logging.INFO, logging.DEBUG][min(args.verbose, 2)]
    logging.basicConfig(level=nivel, format="%(levelname)s: %(message)s")

    try:
        return args.func(args)
    except KeyboardInterrupt:
        print("\nInterrumpido.", file=sys.stderr)
        return 130
    except OSError as e:
        log.error("%s", e)
        return 1


if __name__ == "__main__":
    raise SystemExit(main())
```

Uso:

```bash
python logstat.py resumen access.log
python logstat.py top access.log access.log.1.gz --por ruta -s 4xx -n 5
python logstat.py top access.log --por ip -m post -f json
python logstat.py horas access.log --desde 2026-11-20 --hasta 2026-11-21
python logstat.py -vv resumen access.log
```

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `error: the following arguments are required: x` | Falta un posicional u opción `required`. | Páselo o use `nargs="?"` con `default`. |
| `error: unrecognized arguments: …` | Argumento no declarado o en el analizador equivocado. | Revise el orden: opciones globales antes del subcomando. |
| `--flag False` sigue siendo `True` | Se usó `type=bool`. | `action="store_true"` o `BooleanOptionalAction`. |
| `AttributeError: 'Namespace' object has no attribute 'func'` | No se indicó subcomando y no es obligatorio. | `add_subparsers(required=True)`. |
| `ValueError: badly formed help string` | `%` sin escapar en `help`. | Use `%%`. |
| El valor por defecto de `choices` no se valida | argparse no valida el `default`. | Use un valor incluido en `choices`. |
| `append` acumula valores de ejecuciones anteriores en pruebas | `default=[]` compartido y mutado. | Use `default=None` y trate `None` como lista vacía. |
| `--opcion -5` falla | El valor empieza por `-` y parece otra opción. | `--opcion=-5`. |
| Los saltos de línea de la descripción desaparecen | El formateador reajusta el texto. | `RawDescriptionHelpFormatter`. |
| Abreviaturas que chocan con nuevas opciones | `allow_abbrev=True` por defecto. | `allow_abbrev=False`. |

---

## 19. Buenas prácticas

- Encapsule el parser en una función `crear_parser()` y la ejecución en `main(argv=None)` que devuelva un código de salida.
- Use `type` con funciones de validación propias y `ArgumentTypeError` para mensajes claros.
- Use `set_defaults(func=...)` para asociar cada subcomando a su función.
- Reutilice opciones comunes con `parents=[...]`.
- Use `allow_abbrev=False` en CLIs que vayan a crecer.
- Documente cada argumento con `help` y añada ejemplos en `epilog`.
- Escriba los errores en `stderr` y la salida útil en `stdout` para poder redirigirla.
- Valide las relaciones entre argumentos después de `parse_args()` con `parser.error()`.
- Use `BooleanOptionalAction` para banderas con valor por defecto `True`.
- Pruebe pasando listas de argumentos a `main()`.

---

## 20. Referencia rápida

```python
parser = argparse.ArgumentParser(prog=, description=, epilog=, formatter_class=, allow_abbrev=False,
                                 fromfile_prefix_chars="@", exit_on_error=True)
parser.add_argument("pos" | "-c", "--largo", type=, default=, required=, choices=, nargs=, action=,
                    const=, dest=, metavar=, help=, deprecated=)
action: store store_true store_false store_const append append_const extend count version help
        argparse.BooleanOptionalAction · clase propia (argparse.Action)
nargs: N "?" "*" "+" argparse.REMAINDER
grupo = parser.add_argument_group("título") · parser.add_mutually_exclusive_group(required=True)
sub = parser.add_subparsers(dest="cmd", required=True); p = sub.add_parser("x", parents=[comun], aliases=[...])
p.set_defaults(func=f) → args.func(args)
args = parser.parse_args(argv) · parse_known_args() · parse_intermixed_args() · vars(args)
parser.error("msg") (código 2) · argparse.ArgumentTypeError · help=argparse.SUPPRESS · "%(default)s"
```

---

## 21. Ejercicios propuestos

1. Cree un script `renombrar.py` que renombre archivos de una carpeta con un patrón, con opción `--simular` y confirmación.
2. Implemente una acción personalizada que lea un argumento `--rango 10:20` y lo convierta en un objeto `range`.
3. Añada al analizador de logs un subcomando `lentas` que lea el tiempo de respuesta (si el formato lo incluye) y muestre las peticiones más lentas.
4. Implemente la carga de un archivo TOML de configuración con prioridad inferior a la línea de comandos.
5. Escriba pruebas con `pytest` que verifiquen los errores de validación y la salida JSON del ejemplo completo.

---

## 22. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/argparse.html
- Tutorial oficial: https://docs.python.org/es/3/howto/argparse.html
- argcomplete (autocompletado): https://github.com/kislyuk/argcomplete
