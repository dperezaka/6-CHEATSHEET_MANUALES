# Manual de uso: Python Fire

> **Categoría:** Interfaces de línea de comandos (CLI) · **Utilidad:** Convertir funciones, clases u objetos en CLIs automáticamente
> **Tipo:** Librería de terceros (`pip install fire`) · **Versión de referencia:** 0.7

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primer programa](#4-primer-programa)
5. [Qué se puede exponer](#5-qué-se-puede-exponer)
6. [Argumentos posicionales y con nombre](#6-argumentos-posicionales-y-con-nombre)
7. [Cómo interpreta Fire los valores](#7-cómo-interpreta-fire-los-valores)
8. [Grupos de comandos](#8-grupos-de-comandos)
9. [Encadenar llamadas](#9-encadenar-llamadas)
10. [Salida de resultados](#10-salida-de-resultados)
11. [Ayuda y documentación](#11-ayuda-y-documentación)
12. [Banderas especiales de Fire](#12-banderas-especiales-de-fire)
13. [Control de la conversión con decoradores](#13-control-de-la-conversión-con-decoradores)
14. [Usar Fire sin modificar el código](#14-usar-fire-sin-modificar-el-código)
15. [Errores y códigos de salida](#15-errores-y-códigos-de-salida)
16. [Pruebas](#16-pruebas)
17. [Distribución](#17-distribución)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Python Fire** es una librería, desarrollada originalmente en Google, que genera una interfaz de línea de comandos a partir de **cualquier objeto de Python**: funciones, clases, módulos, diccionarios o instancias. Basta una línea, `fire.Fire()`, para que los parámetros de las funciones se conviertan en argumentos y los métodos de una clase en subcomandos.

**Ventajas:**
- Mínimo esfuerzo: no hay que declarar argumentos ni opciones.
- Ideal para exponer código existente, prototipos, scripts internos y tareas de mantenimiento.
- Subcomandos automáticos a partir de clases y diccionarios.
- Permite encadenar llamadas y explorar objetos desde la terminal.
- Modo interactivo (REPL) con el objeto ya cargado.

**Limitaciones:**
- Poco control sobre la interfaz: nombres, validación y ayuda dependen del código.
- La conversión automática de valores puede sorprender (`010` es texto, `1,2` es una tupla).
- La ayuda es más pobre que la de Typer o Click y no admite personalización fina.
- No es la opción adecuada para herramientas públicas con usuarios no técnicos.

| Si necesita… | Opción |
|---|---|
| Exponer funciones o clases existentes en minutos | **Fire** |
| CLI pública con ayuda y validación cuidadas | Typer / Click |
| Sin dependencias externas | argparse |
| Preguntas interactivas guiadas | questionary / InquirerPy |

---

## 2. Instalación e importación

```bash
pip install fire
```

```python
import fire
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `fire.Fire(componente)` | Crea la CLI a partir del componente indicado y la ejecuta. |
| Componente | Objeto expuesto: función, clase, instancia, módulo, diccionario, lista… |
| Comando | Cada palabra de la línea de comandos accede a un atributo, llama a una función o indexa un elemento. |
| Argumento | Parámetro de una función, por posición o con `--nombre`. |
| Separador `-` | Indica a Fire que termine la llamada actual y continúe con el resultado. |
| Banderas de Fire | Opciones propias de Fire que se escriben tras `--`: `-- --help`, `-- --interactive`. |
| Resultado | Fire imprime el valor devuelto por la última llamada. |

**Idea central:** Fire recorre el objeto como si se escribiera código: `app.py usuarios crear ana` equivale a `componente.usuarios.crear("ana")`.

---

## 4. Primer programa

```python
import fire

def saludar(nombre: str = "mundo", veces: int = 1):
    """Saluda a una persona."""
    return "\n".join(f"Hola, {nombre}" for _ in range(veces))

if __name__ == "__main__":
    fire.Fire(saludar)
```

```bash
python saludo.py                         # Hola, mundo
python saludo.py Ana                     # Hola, Ana
python saludo.py --nombre=Ana --veces=2
python saludo.py Ana 3
python saludo.py --help
```

> Fire **imprime el valor devuelto**. No es necesario usar `print` dentro de la función (aunque también puede hacerse).

---

## 5. Qué se puede exponer

### 5.1 Una función

```python
fire.Fire(funcion)
```

### 5.2 Varias funciones (todo el módulo)

```python
def sumar(a, b):
    return a + b

def restar(a, b):
    return a - b

if __name__ == "__main__":
    fire.Fire()                 # Expone todo el ámbito global del módulo
```

```bash
python calc.py sumar 2 3
python calc.py restar 10 4
```

> `fire.Fire()` sin argumentos expone **todo** lo definido en el módulo, incluidas las importaciones. Para controlar qué se muestra, use un diccionario.

### 5.3 Un diccionario (recomendado para varias funciones)

```python
fire.Fire({
    "sumar": sumar,
    "restar": restar,
})
```

### 5.4 Una clase

```python
class Calculadora:
    """Operaciones aritméticas."""

    def __init__(self, precision: int = 2):
        self.precision = precision

    def dividir(self, a: float, b: float):
        """Divide A entre B."""
        return round(a / b, self.precision)

    def potencia(self, base: float, exp: int = 2):
        return round(base ** exp, self.precision)

if __name__ == "__main__":
    fire.Fire(Calculadora)
```

```bash
python calc.py dividir 10 3                    # 3.33
python calc.py --precision=4 dividir 10 3      # 3.3333 (argumento del constructor)
python calc.py potencia 2 --exp=10
```

> Los argumentos del constructor (`__init__`) se pasan **con nombre** (`--precision=4`) y pueden ir antes o después del método.

### 5.5 Una instancia

```python
fire.Fire(Calculadora(precision=3))      # El constructor ya está resuelto
```

### 5.6 Un módulo o una librería existente

```bash
python -m fire math sqrt 16             # 4.0
python -m fire os.path join a b c       # a/b/c
python -m fire ./utilidades.py limpiar --dias=30
```

---

## 6. Argumentos posicionales y con nombre

| Forma | Ejemplo |
|---|---|
| Posicional | `app.py copiar origen.txt destino/` |
| Con nombre y `=` | `app.py copiar --origen=origen.txt` |
| Con nombre y espacio | `app.py copiar --origen origen.txt` |
| Nombre con guiones | `--max-filas 10` o `--max_filas 10` (ambos válidos) |
| Booleano verdadero | `--verbose` |
| Booleano falso | `--noverbose` o `--verbose=False` |
| `*args` | `app.py sumar 1 2 3 4` |
| `**kwargs` | `app.py configurar --host=x --puerto=5432` |

```python
def exportar(tabla, formato="csv", comprimir=False, *columnas, **filtros):
    ...
```

```bash
python app.py exportar clientes --formato=json --comprimir nombre email --ciudad=Madrid
```

> Los parámetros **sin** valor por defecto son obligatorios; Fire muestra la ayuda si faltan.

### 6.1 Parámetros solo con nombre

Para evitar errores de orden, declare parámetros *keyword-only* con `*`:

```python
def desplegar(entorno, *, version, forzar=False):
    ...
```

```bash
python app.py desplegar produccion --version=1.4.2 --forzar
```

---

## 7. Cómo interpreta Fire los valores

Fire intenta interpretar cada valor como un **literal de Python**; si no puede, lo trata como texto.

| Entrada | Valor recibido | Tipo |
|---|---|---|
| `10` | `10` | `int` |
| `3.5` | `3.5` | `float` |
| `True` / `False` | `True` / `False` | `bool` |
| `None` | `None` | `NoneType` |
| `hola` | `'hola'` | `str` |
| `010` | `'010'` | `str` |
| `2026-11-24` | `'2026-11-24'` | `str` |
| `'[1,2,3]'` | `[1, 2, 3]` | `list` |
| `'{a:1}'` | `{'a': 1}` | `dict` |
| `1,2` | `(1, 2)` | `tuple` |
| `'"10"'` | `'10'` | `str` (comillas dobles dentro de simples) |

**Forzar texto:** encierre el valor en comillas dobles **dentro** de comillas simples (la shell elimina las exteriores):

```bash
python app.py buscar '"12345"'           # Recibe el texto '12345', no el número
```

**Convertir dentro de la función** (lo más robusto):

```python
def buscar(codigo):
    codigo = str(codigo)                 # Independiente de cómo lo interprete Fire
    ...
```

> Las anotaciones de tipo (`codigo: str`) **no** fuerzan la conversión en Fire; solo documentan. Para controlar la conversión, use los decoradores de la sección 13.

---

## 8. Grupos de comandos

### 8.1 Clases anidadas como grupos

```python
class Usuarios:
    """Gestión de usuarios."""

    def crear(self, nombre, admin=False):
        return f"Usuario {nombre} creado (admin={admin})"

    def listar(self):
        return ["ana", "luis", "marta"]


class BaseDatos:
    """Operaciones de base de datos."""

    def migrar(self, version="latest"):
        return f"Migrando a {version}"


class Aplicacion:
    """Herramienta de administración."""

    def __init__(self):
        self.usuarios = Usuarios()
        self.bd = BaseDatos()

    def estado(self):
        return "OK"


if __name__ == "__main__":
    fire.Fire(Aplicacion)
```

```bash
python admin.py estado
python admin.py usuarios crear ana --admin
python admin.py usuarios listar
python admin.py bd migrar --version=42
```

### 8.2 Diccionarios anidados

```python
fire.Fire({
    "usuarios": {"crear": crear_usuario, "listar": listar_usuarios},
    "bd": {"migrar": migrar, "respaldar": respaldar},
})
```

### 8.3 Ocultar miembros

Los atributos y métodos que empiezan por `_` **no** se muestran en la ayuda (aunque siguen siendo accesibles). Use el prefijo `_` para los métodos internos.

---

## 9. Encadenar llamadas

Si un método devuelve un objeto, se puede seguir invocando sobre el resultado:

```python
class Consulta:
    def __init__(self):
        self.filtros = []

    def donde(self, campo, valor):
        self.filtros.append(f"{campo}={valor}")
        return self

    def ordenar(self, campo):
        self.filtros.append(f"orden={campo}")
        return self

    def __str__(self):
        return " & ".join(self.filtros)

fire.Fire(Consulta)
```

```bash
python consulta.py donde ciudad Madrid donde activo True ordenar nombre
# ciudad=Madrid & activo=True & orden=nombre
```

### 9.1 El separador `-`

Cuando una función tiene `*args`, Fire no sabe dónde termina la llamada. Use `-` para cerrarla:

```bash
python app.py filtrar a b c - upper        # Llama a filtrar('a','b','c') y luego .upper() sobre el resultado
python app.py datos lista - 0              # Indexa el primer elemento de la lista devuelta
```

El separador se puede cambiar: `-- --separator=X`.

---

## 10. Salida de resultados

| Valor devuelto | Salida de Fire |
|---|---|
| `str`, `int`, `float` | Se imprime tal cual. |
| `None` | No imprime nada. |
| `list`, `tuple` | Un elemento por línea (o JSON si los elementos son simples y hay varios tipos). |
| `dict` | Pares `clave: valor`, uno por línea. |
| Generador | Imprime cada elemento según se produce. |
| Objeto con `__str__` | Se imprime `str(objeto)`. |
| Objeto sin `__str__` | Muestra la ayuda del objeto (sus miembros). |

**Recomendación:** para salidas destinadas a otros programas, devuelva texto ya formateado (JSON con `json.dumps`, CSV…) o imprima directamente y devuelva `None`.

```python
import json

def informe(formato="tabla"):
    datos = obtener_datos()
    if formato == "json":
        return json.dumps(datos, ensure_ascii=False, indent=2)
    return "\n".join(f"{d['nombre']:<20}{d['total']:>10}" for d in datos)
```

---

## 11. Ayuda y documentación

Fire construye la ayuda a partir de:
- El **nombre** de la función, clase o método.
- Los **parámetros** y sus valores por defecto.
- El **docstring** (admite formato Google, NumPy y reStructuredText para documentar argumentos).

```python
def importar(archivo, separador=";", codificacion="utf-8", simular=False):
    """Importa clientes desde un archivo CSV.

    Args:
        archivo: Ruta del archivo CSV.
        separador: Carácter separador de columnas.
        codificacion: Codificación del archivo.
        simular: Si es True, no guarda los cambios.
    """
```

```bash
python app.py importar --help
python app.py --help
```

```
NAME
    app.py importar - Importa clientes desde un archivo CSV.

SYNOPSIS
    app.py importar ARCHIVO <flags>

POSITIONAL ARGUMENTS
    ARCHIVO
        Ruta del archivo CSV.

FLAGS
    -s, --separador=SEPARADOR
        Default: ';'
        Carácter separador de columnas.
    ...
```

> Fire crea automáticamente abreviaturas de una letra (`-s`) cuando no son ambiguas.

La ayuda se muestra con un paginador (`less`). Para desactivarlo: `PAGER=cat python app.py --help`.

---

## 12. Banderas especiales de Fire

Se escriben **después de `--`** para no confundirlas con argumentos de la función:

| Bandera | Efecto |
|---|---|
| `-- --help` | Ayuda del componente actual (también `--help` directamente). |
| `-- --interactive` / `-- -i` | Abre un intérprete de Python (IPython si está instalado) con el resultado cargado. |
| `-- --trace` | Muestra cómo Fire ha recorrido el objeto (útil para depurar). |
| `-- --verbose` / `-- -v` | Incluye los miembros privados en la ayuda. |
| `-- --separator=X` | Cambia el separador `-` por otro carácter. |
| `-- --completion` | Genera el script de autocompletado para Bash. |
| `-- --completion fish` | Script para Fish. |

```bash
python app.py usuarios listar -- --interactive
python app.py bd migrar 42 -- --trace
python app.py -- --completion > ~/.app-completion.sh
```

---

## 13. Control de la conversión con decoradores

Para evitar la interpretación automática (por ejemplo, códigos con ceros iniciales), use `fire.decorators`:

```python
from datetime import date
from fire import decorators

@decorators.SetParseFn(str)                      # Todos los argumentos como texto
def buscar_producto(codigo):
    return f"Buscando {codigo!r}"

@decorators.SetParseFns(desde=date.fromisoformat, hasta=date.fromisoformat, codigo=str)
def informe(codigo, desde, hasta):
    return f"{codigo}: {(hasta - desde).days} días"
```

```bash
python app.py buscar_producto 007          # '007'
python app.py informe 007 2026-01-01 2026-03-01
```

| Decorador | Uso |
|---|---|
| `SetParseFn(fn)` | Aplica `fn` a todos los argumentos. |
| `SetParseFn(fn, "a", "b")` | Aplica `fn` solo a los argumentos indicados. |
| `SetParseFns(a=fn1, b=fn2)` | Una función por argumento. |

> Los decoradores de Fire no afectan al comportamiento de la función cuando se llama desde Python.

---

## 14. Usar Fire sin modificar el código

```bash
python -m fire modulo funcion arg1 --opcion=valor
python -m fire ./script.py Clase metodo
python -m fire json dumps '{"a": 1}' --indent=2
```

Muy útil para:
- Probar funciones de un módulo desde la terminal durante el desarrollo.
- Explorar librerías: `python -m fire pathlib Path . -- --interactive`.
- Crear tareas de mantenimiento sin escribir una CLI.

---

## 15. Errores y códigos de salida

| Situación | Comportamiento | Código |
|---|---|---|
| Ejecución correcta | Imprime el resultado. | 0 |
| Argumentos incorrectos o miembro inexistente | Muestra «ERROR: …» y la ayuda breve. | 2 |
| Excepción dentro de la función | Traza completa de Python. | 1 |
| `fire.core.FireExit` | Salida controlada. | El indicado |

Gestionar los errores de negocio con mensajes claros:

```python
import sys

def eliminar(id_):
    try:
        servicio.eliminar(int(id_))
    except NoEncontrado:
        print(f"Error: no existe el registro {id_}", file=sys.stderr)
        sys.exit(1)
    return f"Registro {id_} eliminado"
```

O centralizarlo envolviendo la llamada principal:

```python
def main():
    try:
        fire.Fire(Aplicacion)
    except ErrorAplicacion as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)
```

---

## 16. Pruebas

`fire.Fire` acepta el parámetro `command` (lista o texto) y **devuelve** el resultado, lo que facilita las pruebas:

```python
import fire
import pytest
from app import Aplicacion

def test_crear_usuario():
    resultado = fire.Fire(Aplicacion, command=["usuarios", "crear", "ana", "--admin"])
    assert "ana" in resultado

def test_argumento_obligatorio():
    with pytest.raises(fire.core.FireExit) as exc:
        fire.Fire(Aplicacion, command=["usuarios", "crear"])
    assert exc.value.code == 2
```

> Lo más recomendable es probar directamente las clases y funciones (Python puro) y dejar pocas pruebas de la capa Fire.

---

## 17. Distribución

```toml
[project]
name = "admin-tools"
version = "1.0.0"
dependencies = ["fire>=0.7"]

[project.scripts]
admin = "admin_tools.cli:main"
```

```python
# src/admin_tools/cli.py
import fire
from admin_tools.aplicacion import Aplicacion

def main():
    fire.Fire(Aplicacion, name="admin")      # 'name' fija el nombre mostrado en la ayuda
```

```bash
pip install -e .
admin usuarios listar
```

---

## 18. Ejemplo práctico completo

**Utilidades de mantenimiento de una carpeta de datos**: grupos de comandos mediante clases, configuración en el constructor, salida en texto o JSON, decoradores para controlar la conversión, simulación (`--simular`), mensajes de error y nombre de programa personalizado.

```python
"""
mantenimiento.py — Utilidades de mantenimiento de carpetas de datos.
Requiere: pip install fire
Uso: python mantenimiento.py --help
"""
import hashlib
import json
import shutil
import sys
import time
from collections import defaultdict
from datetime import datetime, timedelta
from pathlib import Path

import fire
from fire import decorators


def _tamano(n: float) -> str:
    for unidad in ("B", "KB", "MB", "GB"):
        if n < 1024:
            return f"{n:.1f} {unidad}"
        n /= 1024
    return f"{n:.1f} TB"


def _salida(datos, formato: str):
    if formato == "json":
        return json.dumps(datos, ensure_ascii=False, indent=2, default=str)
    if isinstance(datos, list):
        if not datos:
            return "Sin resultados."
        anchos = {k: max(len(k), *(len(str(d[k])) for d in datos)) for k in datos[0]}
        cabecera = "  ".join(k.upper().ljust(anchos[k]) for k in anchos)
        filas = ["  ".join(str(d[k]).ljust(anchos[k]) for k in anchos) for d in datos]
        return "\n".join([cabecera, "-" * len(cabecera), *filas])
    return "\n".join(f"{k:<22}{v}" for k, v in datos.items())


def _error(mensaje: str, codigo: int = 1):
    print(f"Error: {mensaje}", file=sys.stderr)
    sys.exit(codigo)


class Analisis:
    """Analiza el contenido de la carpeta."""

    def __init__(self, raiz: Path):
        self._raiz = raiz

    def resumen(self, formato="texto"):
        """Número de archivos, tamaño total y distribución por extensión.

        Args:
            formato: 'texto' o 'json'.
        """
        por_ext = defaultdict(lambda: [0, 0])
        for f in self._raiz.rglob("*"):
            if f.is_file():
                ext = f.suffix.lower() or "(sin extensión)"
                por_ext[ext][0] += 1
                por_ext[ext][1] += f.stat().st_size
        total = sum(n for n, _ in por_ext.values())
        tamano = sum(t for _, t in por_ext.values())
        if formato == "json":
            return _salida({"archivos": total, "bytes": tamano,
                            "extensiones": {k: {"archivos": n, "bytes": t} for k, (n, t) in por_ext.items()}},
                           "json")
        filas = [{"extensión": k, "archivos": n, "tamaño": _tamano(t)}
                 for k, (n, t) in sorted(por_ext.items(), key=lambda x: -x[1][1])]
        return f"Carpeta: {self._raiz}\nArchivos: {total} · Tamaño: {_tamano(tamano)}\n\n" + _salida(filas, "texto")

    def grandes(self, limite=10, minimo_mb=1.0, formato="texto"):
        """Lista los archivos más grandes.

        Args:
            limite: Número máximo de archivos a mostrar.
            minimo_mb: Tamaño mínimo en MB.
            formato: 'texto' o 'json'.
        """
        archivos = [f for f in self._raiz.rglob("*") if f.is_file() and f.stat().st_size >= minimo_mb * 1_048_576]
        archivos.sort(key=lambda f: f.stat().st_size, reverse=True)
        return _salida([{"archivo": str(f.relative_to(self._raiz)), "tamaño": _tamano(f.stat().st_size)}
                        for f in archivos[:int(limite)]], formato)

    def duplicados(self, formato="texto"):
        """Detecta archivos duplicados comparando su contenido (SHA-256)."""
        por_tamano = defaultdict(list)
        for f in self._raiz.rglob("*"):
            if f.is_file() and f.stat().st_size > 0:
                por_tamano[f.stat().st_size].append(f)
        por_hash = defaultdict(list)
        for candidatos in (g for g in por_tamano.values() if len(g) > 1):
            for f in candidatos:
                por_hash[hashlib.sha256(f.read_bytes()).hexdigest()].append(f)
        grupos = [g for g in por_hash.values() if len(g) > 1]
        return _salida([{"grupo": i, "archivo": str(f.relative_to(self._raiz)), "tamaño": _tamano(f.stat().st_size)}
                        for i, g in enumerate(grupos, 1) for f in g], formato)


class Limpieza:
    """Elimina o archiva archivos antiguos y temporales."""

    PATRONES_TEMPORALES = ("*.tmp", "*.bak", "~*", "*.log.old", "Thumbs.db", ".DS_Store")

    def __init__(self, raiz: Path):
        self._raiz = raiz

    def temporales(self, simular=True):
        """Elimina archivos temporales conocidos.

        Args:
            simular: Si es True (por defecto), solo muestra lo que se eliminaría. Use --nosimular para borrar.
        """
        encontrados = sorted({f for p in self.PATRONES_TEMPORALES for f in self._raiz.rglob(p) if f.is_file()})
        if not encontrados:
            return "No hay archivos temporales."
        liberado = sum(f.stat().st_size for f in encontrados)
        for f in encontrados:
            print(("[simulación] " if simular else "eliminado: ") + str(f.relative_to(self._raiz)))
            if not simular:
                f.unlink()
        accion = "Se liberarían" if simular else "Liberados"
        return f"\n{accion} {_tamano(liberado)} en {len(encontrados)} archivo(s)."

    def archivar(self, dias=90, destino="_archivo", simular=True):
        """Mueve a una subcarpeta los archivos no modificados en los últimos DIAS días.

        Args:
            dias: Antigüedad mínima en días.
            destino: Subcarpeta de destino (relativa a la raíz).
            simular: Si es True (por defecto), no mueve nada.
        """
        if int(dias) <= 0:
            _error("--dias debe ser mayor que 0", 2)
        limite = time.time() - int(dias) * 86400
        carpeta = self._raiz / destino
        antiguos = [f for f in self._raiz.rglob("*")
                    if f.is_file() and carpeta not in f.parents and f.stat().st_mtime < limite]
        for f in antiguos:
            nuevo = carpeta / f.relative_to(self._raiz)
            print(("[simulación] " if simular else "") + f"{f.relative_to(self._raiz)} → {nuevo.relative_to(self._raiz)}")
            if not simular:
                nuevo.parent.mkdir(parents=True, exist_ok=True)
                shutil.move(f, nuevo)
        fecha = (datetime.now() - timedelta(days=int(dias))).date()
        return f"\n{len(antiguos)} archivo(s) anteriores al {fecha}" + (" (simulación)" if simular else " archivados")


class Mantenimiento:
    """Utilidades de mantenimiento de carpetas de datos.

    Ejemplos:
        mantenimiento --raiz=./datos analisis resumen
        mantenimiento analisis grandes --limite=5
        mantenimiento limpieza temporales --nosimular
    """

    def __init__(self, raiz="."):
        """
        Args:
            raiz: Carpeta sobre la que se trabaja.
        """
        self._raiz = Path(str(raiz)).expanduser().resolve()
        if not self._raiz.is_dir():
            _error(f"la carpeta {self._raiz} no existe")
        self.analisis = Analisis(self._raiz)
        self.limpieza = Limpieza(self._raiz)

    @decorators.SetParseFn(str)
    def buscar(self, patron, formato="texto"):
        """Busca archivos cuyo nombre coincida con PATRON (comodines admitidos, p. ej. '*2026*').

        Args:
            patron: Patrón de búsqueda (siempre se trata como texto).
            formato: 'texto' o 'json'.
        """
        resultados = [{"archivo": str(f.relative_to(self._raiz)),
                       "modificado": datetime.fromtimestamp(f.stat().st_mtime).strftime("%Y-%m-%d %H:%M")}
                      for f in sorted(self._raiz.rglob(patron)) if f.is_file()]
        return _salida(resultados, formato)


def main():
    fire.Fire(Mantenimiento, name="mantenimiento")


if __name__ == "__main__":
    main()
```

Uso:

```bash
python mantenimiento.py --help
python mantenimiento.py --raiz=~/Descargas analisis resumen
python mantenimiento.py analisis grandes --limite=5 --minimo_mb=10
python mantenimiento.py analisis duplicados --formato=json > duplicados.json
python mantenimiento.py buscar '*2026*'
python mantenimiento.py limpieza temporales                  # Simulación
python mantenimiento.py limpieza temporales --nosimular      # Elimina
python mantenimiento.py limpieza archivar --dias=180 --nosimular
python mantenimiento.py analisis -- --interactive            # Explorar en un REPL
```

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ERROR: Could not consume arg: x` | Argumento sobrante o nombre de comando incorrecto. | Revise `--help`; los nombres de métodos distinguen mayúsculas. |
| `ERROR: The function received no value for the required argument: x` | Falta un parámetro obligatorio. | Páselo o asigne un valor por defecto. |
| Un código `007` llega como `7` o `'007'` de forma inconsistente | Conversión automática de literales. | `SetParseFn(str)` o `str(valor)` en la función. |
| Una fecha `2026-11-24` llega como texto | Fire no interpreta fechas. | Conviértala con `SetParseFns(fecha=date.fromisoformat)`. |
| `--help` muestra los miembros de un objeto en lugar del resultado | La función devolvió un objeto sin `__str__`. | Devuelva texto o defina `__str__`. |
| Aparecen funciones importadas como comandos | Se usó `fire.Fire()` sin argumentos. | Pase un diccionario o una clase. |
| `--interactive` se interpreta como argumento de la función | Falta el separador `--`. | `app.py comando -- --interactive`. |
| La ayuda se queda en un paginador | Fire usa `less` por defecto. | `PAGER=cat` o pulse `q`. |
| Una lista con espacios se rompe | La shell separa los elementos. | Entrecomille: `'[1, 2, 3]'`. |
| `--verbose=false` no funciona como se espera | `false` (minúsculas) no es literal de Python; llega como texto. | `--noverbose` o `--verbose=False`. |

---

## 20. Buenas prácticas

- Exponga una **clase o diccionario** explícito, no `fire.Fire()` sin argumentos.
- Prefije con `_` los atributos y métodos internos para que no aparezcan en la ayuda.
- Escriba docstrings con sección `Args:`: es la única documentación que verá el usuario.
- Use parámetros con valores por defecto y nombres claros; Fire los convierte directamente en banderas.
- Controle la conversión de valores críticos (identificadores, códigos, fechas) con `SetParseFn(s)` o dentro de la función.
- Devuelva texto formateado (o JSON) en lugar de estructuras complejas cuando la salida vaya a otros programas.
- Haga que las acciones destructivas simulen por defecto (`simular=True`) y requieran `--nosimular`.
- Use Fire para herramientas internas y prototipos; migre a Typer o Click cuando la CLI se convierta en un producto.

---

## 21. Referencia rápida

```python
fire.Fire()                               # Todo el módulo
fire.Fire(funcion | Clase | instancia | {"nombre": f, "grupo": {...}})
fire.Fire(componente, command=["a", "b"], name="prog")   # Para pruebas / nombre del programa

app.py metodo pos1 pos2 --clave=valor --bandera --nobandera
app.py --arg_constructor=x metodo ...
app.py grupo subcomando ...
app.py f a b - otro_metodo                # Separador para encadenar
app.py comando -- --help | --interactive | --trace | --verbose | --separator=X | --completion

Valores: 10→int  3.5→float  True→bool  hola→str  '[1,2]'→list  '{a:1}'→dict  1,2→tuple  '"10"'→str
from fire import decorators
@decorators.SetParseFn(str) · @decorators.SetParseFns(arg=funcion)
python -m fire modulo|./archivo.py componente ...
```

---

## 22. Ejercicios propuestos

1. Exponga con Fire una clase `Conversor` con métodos para convertir temperatura, longitud y peso, con un parámetro `precision` en el constructor.
2. Cree una CLI con grupos `usuarios` y `pedidos` usando diccionarios anidados y compare la ayuda resultante con la versión basada en clases.
3. Añada al ejemplo completo un comando `analisis antiguos` que liste los archivos no modificados en N días, con salida JSON.
4. Use `python -m fire` para explorar el módulo `statistics` y calcular la media y la mediana de una lista desde la terminal.
5. Escriba pruebas con `fire.Fire(..., command=[...])` para los comandos `buscar` y `limpieza temporales` del ejemplo.

---

## 23. Referencias

- Repositorio y documentación: https://github.com/google/python-fire
- Guía de uso: https://github.com/google/python-fire/blob/master/docs/guide.md
- Referencia de banderas: https://github.com/google/python-fire/blob/master/docs/using-cli.md
