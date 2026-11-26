# Manual de uso: Funciones y módulos en Python

> **Categoría:** Programación · **Utilidad:** Reutilizar código y organizar proyectos
> **Tipo:** Lenguaje y librería estándar (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Conceptos clave](#2-conceptos-clave)
3. [Definir y llamar funciones](#3-definir-y-llamar-funciones)
4. [Parámetros y argumentos](#4-parámetros-y-argumentos)
5. [Valores de retorno](#5-valores-de-retorno)
6. [Ámbito de las variables](#6-ámbito-de-las-variables)
7. [Anotaciones de tipo y documentación](#7-anotaciones-de-tipo-y-documentación)
8. [Funciones como objetos y `lambda`](#8-funciones-como-objetos-y-lambda)
9. [Clausuras y decoradores](#9-clausuras-y-decoradores)
10. [Generadores](#10-generadores)
11. [Módulos e importaciones](#11-módulos-e-importaciones)
12. [Paquetes](#12-paquetes)
13. [Librería estándar esencial](#13-librería-estándar-esencial)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

Las **funciones** agrupan instrucciones bajo un nombre para reutilizarlas, probarlas y entenderlas por separado. Los **módulos** (archivos `.py`) y los **paquetes** (carpetas de módulos) permiten repartir un programa en piezas con responsabilidades claras.

**Beneficios:**
- Evitan duplicar código (principio DRY: *Don't Repeat Yourself*).
- Facilitan las pruebas unitarias.
- Hacen el programa más legible: cada función describe *qué* hace con su nombre.
- Permiten compartir código entre proyectos.

---

## 2. Conceptos clave

| Concepto | Descripción |
|---|---|
| Función | Bloque de código con nombre que recibe datos y puede devolver un resultado. |
| Parámetro | Nombre declarado en la definición (`def f(x)`). |
| Argumento | Valor concreto que se pasa en la llamada (`f(5)`). |
| Retorno | Valor que devuelve la función con `return` (por defecto, `None`). |
| Ámbito | Zona del programa donde una variable es visible (local, global…). |
| Función pura | Su resultado depende solo de sus argumentos y no tiene efectos secundarios. |
| Decorador | Función que envuelve a otra para añadirle comportamiento. |
| Generador | Función que produce valores uno a uno con `yield`. |
| Módulo | Archivo `.py` importable. |
| Paquete | Carpeta con módulos (normalmente con `__init__.py`). |

---

## 3. Definir y llamar funciones

```python
def saludar(nombre):
    print(f"Hola, {nombre}")

saludar("Ana")
```

```python
def calcular_iva(base: float, tipo: float = 0.21) -> float:
    """Devuelve la cuota de IVA de una base imponible."""
    return round(base * tipo, 2)

cuota = calcular_iva(100)          # 21.0
```

> Una función debe hacer **una sola cosa** y su nombre debe ser un verbo que lo describa: `calcular_total`, `validar_email`, `exportar_csv`.

---

## 4. Parámetros y argumentos

### 4.1 Posicionales y con nombre

```python
def crear_usuario(nombre, email, activo=True):
    ...

crear_usuario("Ana", "ana@ejemplo.com")                    # Posicionales
crear_usuario(email="ana@ejemplo.com", nombre="Ana")       # Con nombre (cualquier orden)
crear_usuario("Ana", "ana@ejemplo.com", activo=False)      # Combinados
```

### 4.2 Valores por defecto

```python
def conectar(host="localhost", puerto=5432, tiempo_espera=10):
    ...
```

> ⚠ **Nunca** use un objeto mutable como valor por defecto: se comparte entre llamadas.

```python
def anadir(elemento, lista=[]):        # ❌ Incorrecto
    lista.append(elemento)
    return lista

def anadir(elemento, lista=None):      # ✔ Correcto
    if lista is None:
        lista = []
    lista.append(elemento)
    return lista
```

### 4.3 Número variable de argumentos

```python
def sumar(*numeros):                   # numeros es una tupla
    return sum(numeros)

sumar(1, 2, 3, 4)                      # 10

def registrar(evento, **detalles):     # detalles es un diccionario
    print(evento, detalles)

registrar("login", usuario="ana", ip="10.0.0.5")
```

### 4.4 Parámetros solo posicionales (`/`) y solo con nombre (`*`)

```python
def dividir(a, b, /, *, redondeo=2):
    return round(a / b, redondeo)

dividir(10, 3, redondeo=4)     # ✔
dividir(a=10, b=3)             # ❌ TypeError: a y b son solo posicionales
dividir(10, 3, 4)              # ❌ TypeError: redondeo es solo con nombre
```

> Los parámetros **solo con nombre** hacen las llamadas más legibles: `exportar(datos, sobrescribir=True)` es más claro que `exportar(datos, True)`.

**Orden obligatorio en la definición:**

```python
def f(pos_solo, /, normal, *args, solo_nombre, **kwargs): ...
```

---

## 5. Valores de retorno

```python
def es_par(n: int) -> bool:
    return n % 2 == 0

def estadisticas(valores: list[float]) -> tuple[float, float, float]:
    return min(valores), max(valores), sum(valores) / len(valores)

minimo, maximo, media = estadisticas([4, 8, 15, 16, 23, 42])
```

### 5.1 Retorno anticipado (cláusulas de guarda)

```python
def precio_final(precio: float, cliente: dict | None) -> float:
    if precio <= 0:
        raise ValueError("El precio debe ser positivo")
    if cliente is None:
        return precio
    if cliente.get("vip"):
        return precio * 0.9
    return precio
```

> Las **cláusulas de guarda** evitan anidar `if` y hacen visible el caso principal.

### 5.2 Sin `return`

Una función sin `return` (o con `return` vacío) devuelve `None`.

---

## 6. Ámbito de las variables

Python busca los nombres en este orden (**regla LEGB**):

| Nivel | Descripción |
|---|---|
| **L**ocal | Dentro de la función actual. |
| **E**nclosing | En funciones que contienen a la actual. |
| **G**lobal | En el módulo. |
| **B**uilt-in | Nombres integrados (`print`, `len`…). |

```python
contador = 0

def incrementar():
    global contador          # Necesario para MODIFICAR una variable global
    contador += 1

def externa():
    total = 0
    def interna():
        nonlocal total       # Modifica la variable de la función contenedora
        total += 1
    interna()
    return total
```

> Evite `global`: pase los datos como argumentos y devuelva resultados. El código es más predecible y más fácil de probar.

---

## 7. Anotaciones de tipo y documentación

### 7.1 Anotaciones de tipo

Las anotaciones **no se comprueban en ejecución**, pero los editores y herramientas como `mypy` las usan para detectar errores.

```python
from collections.abc import Callable, Iterable

def promedio(valores: Iterable[float]) -> float: ...
def buscar(clientes: list[dict[str, str]], email: str) -> dict[str, str] | None: ...
def aplicar(funcion: Callable[[int], int], datos: list[int]) -> list[int]: ...

type Matriz = list[list[float]]          # Alias de tipo (Python 3.12+)
def trasponer(m: Matriz) -> Matriz: ...
```

| Anotación | Significado |
|---|---|
| `int`, `str`, `float`, `bool` | Tipos básicos |
| `list[int]`, `dict[str, float]`, `set[str]`, `tuple[int, str]` | Colecciones |
| `tuple[int, ...]` | Tupla de longitud variable |
| `X \| None` | Valor de tipo X o `None` |
| `Callable[[A, B], R]` | Función que recibe A y B y devuelve R |
| `Any` | Cualquier tipo (evítelo cuando sea posible) |

### 7.2 *Docstrings*

```python
def amortizacion(capital: float, interes_anual: float, meses: int) -> float:
    """Calcula la cuota mensual de un préstamo (sistema francés).

    Args:
        capital: Importe prestado en euros.
        interes_anual: Tipo de interés anual en tanto por uno (0.035 = 3,5 %).
        meses: Número de cuotas.

    Returns:
        Importe de la cuota mensual, redondeado a céntimos.

    Raises:
        ValueError: Si ``meses`` no es positivo.
    """
```

```python
help(amortizacion)                 # Muestra la documentación
amortizacion.__doc__
```

---

## 8. Funciones como objetos y `lambda`

En Python las funciones son objetos: se pueden asignar, pasar como argumento y devolver.

```python
def doble(x): return x * 2
def triple(x): return x * 3

operaciones = {"doble": doble, "triple": triple}
operaciones["triple"](5)           # 15

def aplicar(funcion, valores):
    return [funcion(v) for v in valores]

aplicar(doble, [1, 2, 3])          # [2, 4, 6]
```

### 8.1 Funciones anónimas (`lambda`)

```python
cuadrado = lambda x: x ** 2        # Válido, pero mejor usar def
sorted(productos, key=lambda p: p["precio"])
```

> Use `lambda` solo para funciones de **una expresión** que se pasan como argumento (`key=`, `command=`…). Si necesita un nombre, use `def`.

### 8.2 `functools`

```python
from functools import partial, lru_cache, reduce

iva_reducido = partial(calcular_iva, tipo=0.10)
iva_reducido(100)                  # 10.0

@lru_cache(maxsize=None)           # Memoriza resultados (memoización)
def fibonacci(n: int) -> int:
    return n if n < 2 else fibonacci(n - 1) + fibonacci(n - 2)

reduce(lambda a, b: a * b, [1, 2, 3, 4])    # 24
```

---

## 9. Clausuras y decoradores

### 9.1 Clausuras

Una función interna que «recuerda» variables de la función que la creó:

```python
def crear_multiplicador(factor):
    def multiplicar(x):
        return x * factor
    return multiplicar

por_tres = crear_multiplicador(3)
por_tres(10)                       # 30
```

### 9.2 Decoradores

Un decorador recibe una función y devuelve otra que la envuelve:

```python
import functools
import time

def cronometrar(funcion):
    @functools.wraps(funcion)              # Conserva nombre y docstring originales
    def envoltura(*args, **kwargs):
        inicio = time.perf_counter()
        resultado = funcion(*args, **kwargs)
        print(f"{funcion.__name__}: {time.perf_counter() - inicio:.3f} s")
        return resultado
    return envoltura

@cronometrar
def procesar(datos):
    ...
```

### 9.3 Decoradores con parámetros

```python
def reintentar(veces: int = 3, espera: float = 1.0):
    def decorador(funcion):
        @functools.wraps(funcion)
        def envoltura(*args, **kwargs):
            for intento in range(1, veces + 1):
                try:
                    return funcion(*args, **kwargs)
                except ConnectionError:
                    if intento == veces:
                        raise
                    time.sleep(espera)
        return envoltura
    return decorador

@reintentar(veces=5, espera=2)
def descargar(url): ...
```

---

## 10. Generadores

Producen valores **bajo demanda** con `yield`, sin cargar toda la secuencia en memoria. Ideales para archivos grandes y flujos de datos.

```python
def leer_lineas(ruta):
    with open(ruta, encoding="utf-8") as f:
        for linea in f:
            yield linea.rstrip("\n")

for linea in leer_lineas("registro.log"):      # Procesa un archivo de varios GB sin problemas
    if "ERROR" in linea:
        print(linea)
```

```python
def lotes(iterable, tamano):
    lote = []
    for elemento in iterable:
        lote.append(elemento)
        if len(lote) == tamano:
            yield lote
            lote = []
    if lote:
        yield lote
```

> Desde Python 3.12, `itertools.batched(iterable, n)` hace lo mismo.

### 10.1 Encadenar generadores (*pipelines*)

```python
lineas = leer_lineas("ventas.csv")
campos = (l.split(";") for l in lineas)
importes = (float(c[3]) for c in campos if c[3])
total = sum(importes)
```

### 10.2 `itertools`

```python
from itertools import chain, islice, groupby, product, combinations, count, accumulate

list(chain([1, 2], [3, 4]))              # [1, 2, 3, 4]
list(islice(generador, 10))              # Primeros 10 elementos
list(accumulate([1, 2, 3, 4]))           # [1, 3, 6, 10]  (suma acumulada)
list(combinations("ABC", 2))             # [('A','B'), ('A','C'), ('B','C')]
list(product([0, 1], repeat=2))          # [(0,0), (0,1), (1,0), (1,1)]
```

---

## 11. Módulos e importaciones

Cualquier archivo `.py` es un módulo.

```python
# utilidades.py
IVA = 0.21

def formatear_euros(valor: float) -> str:
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")
```

```python
# main.py
import utilidades
utilidades.formatear_euros(1234.5)        # '1.234,50 €'

from utilidades import formatear_euros, IVA
import utilidades as ut
from utilidades import formatear_euros as euros
```

> ⚠ Evite `from modulo import *`: oculta de dónde procede cada nombre y puede sobrescribir otros.

### 11.1 Orden de las importaciones (PEP 8)

```python
# 1. Librería estándar
import json
from pathlib import Path

# 2. Librerías de terceros
import requests

# 3. Módulos propios
from mi_app.servicios import clientes
```

### 11.2 Dónde busca Python los módulos

1. La carpeta del script que se ejecuta.
2. Las rutas de la variable de entorno `PYTHONPATH`.
3. La librería estándar y los paquetes instalados (`site-packages`).

```python
import sys
print(sys.path)
```

> ⚠ No llame a sus archivos igual que módulos estándar o de terceros (`random.py`, `email.py`, `requests.py`): Python importaría el suyo.

---

## 12. Paquetes

```
mi_app/
├── main.py
└── facturacion/
    ├── __init__.py          # Marca la carpeta como paquete (puede estar vacío)
    ├── calculos.py
    ├── exportar.py
    └── modelos/
        ├── __init__.py
        └── factura.py
```

```python
from facturacion.calculos import total_factura
from facturacion.modelos.factura import Factura
```

### 12.1 Importaciones relativas (dentro del paquete)

```python
# facturacion/exportar.py
from .calculos import total_factura          # Mismo paquete
from .modelos.factura import Factura
from .. import configuracion                 # Paquete superior
```

### 12.2 Exponer una interfaz en `__init__.py`

```python
# facturacion/__init__.py
from .calculos import total_factura
from .modelos.factura import Factura

__all__ = ["total_factura", "Factura"]
```

```python
from facturacion import Factura, total_factura
```

### 12.3 Ejecutar un módulo de un paquete

```bash
python -m facturacion.exportar
```

---

## 13. Librería estándar esencial

| Módulo | Uso |
|---|---|
| `pathlib` | Rutas y archivos de forma orientada a objetos. |
| `os`, `shutil` | Sistema operativo, copiar y mover archivos. |
| `sys` | Argumentos, versión, salida del programa. |
| `datetime`, `zoneinfo` | Fechas, horas y zonas horarias. |
| `json`, `csv` | Lectura y escritura de JSON y CSV. |
| `re` | Expresiones regulares. |
| `math`, `statistics`, `decimal`, `random` | Cálculo. |
| `collections`, `itertools`, `functools` | Utilidades de datos y funciones. |
| `logging` | Registro de eventos. |
| `argparse` | Argumentos de línea de comandos. |
| `subprocess` | Ejecutar otros programas. |
| `sqlite3` | Base de datos SQL integrada. |
| `unittest` | Pruebas automatizadas. |
| `concurrent.futures`, `threading`, `asyncio` | Concurrencia. |

```python
from datetime import datetime, timedelta
ahora = datetime.now()
vencimiento = ahora + timedelta(days=30)
vencimiento.strftime("%d/%m/%Y")

import random
random.choice(["A", "B", "C"]); random.randint(1, 6); random.sample(range(100), 5)
```

---

## 14. Ejemplo práctico completo

**Herramienta de línea de comandos** para calcular un préstamo: módulo con funciones puras y documentadas, decorador de validación, generador del cuadro de amortización y argumentos con `argparse`.

```python
"""
prestamo.py · Calculadora de préstamos (sistema francés).

Uso:
    python prestamo.py 150000 3.5 25
    python prestamo.py 20000 6.9 5 --meses-mostrar 12 --csv cuadro.csv
"""
import argparse
import csv
import functools
from collections.abc import Callable, Iterator
from typing import NamedTuple


class Cuota(NamedTuple):
    numero: int
    cuota: float
    intereses: float
    amortizacion: float
    pendiente: float


def positivos(*nombres: str) -> Callable:
    """Decorador: valida que los argumentos indicados sean mayores que cero."""
    def decorador(funcion: Callable) -> Callable:
        @functools.wraps(funcion)
        def envoltura(*args, **kwargs):
            valores = dict(zip(funcion.__code__.co_varnames, args)) | kwargs
            for nombre in nombres:
                if valores.get(nombre, 1) <= 0:
                    raise ValueError(f"'{nombre}' debe ser mayor que cero")
            return funcion(*args, **kwargs)
        return envoltura
    return decorador


@positivos("capital", "meses")
def cuota_mensual(capital: float, interes_anual: float, meses: int) -> float:
    """Cuota mensual constante. ``interes_anual`` en porcentaje (3.5 = 3,5 %)."""
    i = interes_anual / 100 / 12
    if i == 0:
        return capital / meses
    return capital * i / (1 - (1 + i) ** -meses)


def cuadro_amortizacion(capital: float, interes_anual: float, meses: int) -> Iterator[Cuota]:
    """Genera, una a una, las cuotas del préstamo."""
    cuota = cuota_mensual(capital, interes_anual, meses)
    i = interes_anual / 100 / 12
    pendiente = capital
    for n in range(1, meses + 1):
        intereses = pendiente * i
        amortizado = cuota - intereses
        pendiente = max(pendiente - amortizado, 0.0)
        yield Cuota(n, cuota, intereses, amortizado, pendiente)


def euros(valor: float) -> str:
    """Formatea un número con el estilo español: 1.234,56 €."""
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def resumen(capital: float, interes_anual: float, anos: int) -> dict[str, float]:
    meses = anos * 12
    cuota = cuota_mensual(capital, interes_anual, meses)
    total = cuota * meses
    return {"cuota": cuota, "total_pagado": total, "total_intereses": total - capital}


def exportar_csv(ruta: str, cuotas: Iterator[Cuota]) -> int:
    with open(ruta, "w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.writer(f, delimiter=";")
        escritor.writerow(Cuota._fields)
        n = 0
        for c in cuotas:
            escritor.writerow([c.numero] + [f"{v:.2f}".replace(".", ",") for v in c[1:]])
            n += 1
    return n


def argumentos() -> argparse.Namespace:
    p = argparse.ArgumentParser(description="Calcula la cuota y el cuadro de amortización de un préstamo.")
    p.add_argument("capital", type=float, help="Importe del préstamo en euros")
    p.add_argument("interes", type=float, help="Interés nominal anual en %% (ej.: 3.5)")
    p.add_argument("anos", type=int, help="Plazo en años")
    p.add_argument("--meses-mostrar", type=int, default=6, help="Cuotas a mostrar (por defecto: 6)")
    p.add_argument("--csv", metavar="RUTA", help="Exporta el cuadro completo a CSV")
    return p.parse_args()


def main() -> None:
    args = argumentos()
    try:
        datos = resumen(args.capital, args.interes, args.anos)
    except ValueError as e:
        raise SystemExit(f"Error: {e}")

    print(f"\nPréstamo de {euros(args.capital)} al {args.interes} % a {args.anos} años")
    print(f"  Cuota mensual:      {euros(datos['cuota'])}")
    print(f"  Total a pagar:      {euros(datos['total_pagado'])}")
    print(f"  Total de intereses: {euros(datos['total_intereses'])}\n")

    print(f"{'N.º':>4}{'Cuota':>14}{'Intereses':>14}{'Amortización':>15}{'Pendiente':>16}")
    cuadro = cuadro_amortizacion(args.capital, args.interes, args.anos * 12)
    for c in cuadro:
        if c.numero > args.meses_mostrar:
            break
        print(f"{c.numero:>4}{euros(c.cuota):>14}{euros(c.intereses):>14}"
              f"{euros(c.amortizacion):>15}{euros(c.pendiente):>16}")

    if args.csv:
        n = exportar_csv(args.csv, cuadro_amortizacion(args.capital, args.interes, args.anos * 12))
        print(f"\n✔ {n} cuotas exportadas a {args.csv}")


if __name__ == "__main__":
    main()
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: f() missing 1 required positional argument` | Falta un argumento obligatorio. | Páselo o asigne un valor por defecto. |
| `TypeError: f() takes 1 positional argument but 2 were given` | En un método, falta `self`. | Declare `def metodo(self, ...)`. |
| La función devuelve `None` inesperadamente | Falta `return` o hay una rama sin él. | Revise todas las ramas. |
| La lista por defecto acumula valores entre llamadas | Valor por defecto mutable (`lista=[]`). | Use `None` y créela dentro. |
| `UnboundLocalError: local variable referenced before assignment` | Se asigna una variable global dentro de la función sin `global`. | Pase el valor como argumento o use `global`/`nonlocal`. |
| `ModuleNotFoundError: No module named 'x'` | Módulo no instalado o ruta incorrecta. | `pip install x` o revise la estructura del paquete. |
| `ImportError: attempted relative import with no known parent package` | Se ejecuta directamente un módulo con imports relativos. | Ejecútelo con `python -m paquete.modulo`. |
| `ImportError: cannot import name 'X' (circular import)` | Dos módulos se importan entre sí. | Extraiga el código común a un tercer módulo. |
| Se importa un archivo propio en lugar de una librería | Archivo con el mismo nombre (`random.py`). | Renombre el archivo y borre `__pycache__`. |
| El decorador hace que la función pierda su nombre | Falta `functools.wraps`. | Añada `@functools.wraps(funcion)`. |
| Un generador no devuelve nada la segunda vez | Los generadores se agotan tras recorrerse. | Vuelva a llamar a la función o use una lista. |

---

## 16. Buenas prácticas

- Cada función debe tener **una única responsabilidad** y, a ser posible, menos de 30-40 líneas.
- Use nombres verbales y descriptivos.
- Prefiera **funciones puras**: más fáciles de probar y reutilizar.
- Añada anotaciones de tipo y *docstrings* a las funciones públicas.
- Use parámetros solo con nombre (`*`) para opciones booleanas o poco obvias.
- No use objetos mutables como valores por defecto.
- Evite `global`: pase datos como argumentos y devuelva resultados.
- Use generadores para procesar grandes volúmenes de datos.
- Agrupe en módulos el código relacionado y evite las importaciones circulares.
- Ordene las importaciones (estándar, terceros, propias) y no use `import *`.
- Antes de escribir una utilidad, compruebe si ya existe en la librería estándar.

---

## 17. Referencia rápida

```python
def f(a, b=0, *args, clave=None, **kwargs) -> tipo: """docstring"""; return valor
def f(pos, /, normal, *, solo_nombre): ...
lambda x: x * 2      /   sorted(it, key=lambda x: x[1])
global x / nonlocal x
def deco(fn): @functools.wraps(fn) def env(*a, **k): return fn(*a, **k); return env
@lru_cache  /  partial(f, arg=valor)
def gen(): yield valor     /   (x for x in it)   /   next(gen)
import modulo / from modulo import nombre / import modulo as alias / from .local import x
if __name__ == "__main__": main()
python -m paquete.modulo
```

---

## 18. Ejercicios propuestos

1. Escriba una función `validar_dni(dni: str) -> bool` que compruebe la letra de un DNI español.
2. Cree un decorador `@registrar` que escriba en un archivo de texto el nombre de la función, sus argumentos y la hora de cada llamada.
3. Escriba un generador que lea un archivo CSV grande y devuelva solo las filas que cumplan un filtro recibido como función.
4. Organice en un paquete `geometria` los módulos `figuras.py` y `conversiones.py`, con una interfaz pública en `__init__.py`.
5. Amplíe el ejemplo completo con la opción `--amortizacion-anticipada` que recalcule el cuadro tras un pago extraordinario en un mes dado.

---

## 19. Referencias

- Definición de funciones (tutorial): https://docs.python.org/es/3/tutorial/controlflow.html#defining-functions
- Módulos (tutorial): https://docs.python.org/es/3/tutorial/modules.html
- `typing`: https://docs.python.org/es/3/library/typing.html
- `functools`: https://docs.python.org/es/3/library/functools.html
- `itertools`: https://docs.python.org/es/3/library/itertools.html
- Índice de la librería estándar: https://docs.python.org/es/3/library/index.html
