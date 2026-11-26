# Manual de uso: Estructuras de datos en Python

> **Categoría:** Programación · **Utilidad:** Almacenar, organizar y transformar colecciones de datos
> **Tipo:** Tipos integrados y librería estándar (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Conceptos clave](#2-conceptos-clave)
3. [Listas](#3-listas)
4. [Tuplas](#4-tuplas)
5. [Diccionarios](#5-diccionarios)
6. [Conjuntos](#6-conjuntos)
7. [Comprensiones](#7-comprensiones)
8. [Desempaquetado](#8-desempaquetado)
9. [Ordenar, filtrar y agregar](#9-ordenar-filtrar-y-agregar)
10. [Módulo `collections`](#10-módulo-collections)
11. [Copias y mutabilidad](#11-copias-y-mutabilidad)
12. [Elegir la estructura adecuada](#12-elegir-la-estructura-adecuada)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

Python incluye cuatro colecciones integradas —**listas, tuplas, diccionarios y conjuntos**— que cubren la gran mayoría de necesidades. El módulo estándar `collections` añade estructuras especializadas (`Counter`, `defaultdict`, `deque`…).

| Estructura | Sintaxis | Ordenada | Mutable | Duplicados | Acceso |
|---|---|---|---|---|---|
| `list` | `[1, 2, 3]` | Sí | Sí | Sí | Por índice |
| `tuple` | `(1, 2, 3)` | Sí | No | Sí | Por índice |
| `dict` | `{"a": 1}` | Sí (orden de inserción) | Sí | Claves únicas | Por clave |
| `set` | `{1, 2, 3}` | No | Sí | No | Pertenencia |
| `frozenset` | `frozenset({1, 2})` | No | No | No | Pertenencia |

---

## 2. Conceptos clave

| Concepto | Descripción |
|---|---|
| Colección | Objeto que agrupa varios valores. |
| Iterable | Objeto que se puede recorrer con `for` (listas, cadenas, diccionarios, archivos…). |
| Secuencia | Colección ordenada con acceso por índice (`list`, `tuple`, `str`, `range`). |
| Mutable | Se puede modificar después de crearse (`list`, `dict`, `set`). |
| Inmutable | No se puede modificar (`tuple`, `str`, `frozenset`, números). |
| *Hashable* | Valor inmutable que puede ser clave de diccionario o elemento de un conjunto. |
| Comprensión | Sintaxis compacta para crear colecciones a partir de otras. |
| Referencia | Las variables apuntan a objetos; asignar no copia. |

---

## 3. Listas

### 3.1 Creación y acceso

```python
numeros = [10, 20, 30, 40, 50]
mixta = [1, "dos", 3.0, None, [5, 6]]
vacia = []
desde_rango = list(range(1, 6))          # [1, 2, 3, 4, 5]

numeros[0]        # 10
numeros[-1]       # 50
numeros[1:3]      # [20, 30]
numeros[::2]      # [10, 30, 50]
len(numeros)      # 5
30 in numeros     # True
```

### 3.2 Modificación

```python
frutas = ["manzana", "pera"]
frutas.append("uva")                # Añadir al final
frutas.insert(0, "kiwi")            # Insertar en una posición
frutas.extend(["melón", "fresa"])   # Añadir varios
frutas[1] = "naranja"               # Reemplazar

frutas.remove("pera")               # Elimina la primera coincidencia (ValueError si no existe)
ultimo = frutas.pop()               # Extrae el último
primero = frutas.pop(0)             # Extrae por índice
del frutas[0]                       # Elimina por índice
frutas.clear()                      # Vacía la lista
```

### 3.3 Búsqueda y orden

```python
valores = [3, 1, 4, 1, 5, 9, 2]
valores.index(4)          # 2  (posición; ValueError si no existe)
valores.count(1)          # 2
valores.sort()            # Ordena la propia lista (devuelve None)
valores.sort(reverse=True)
valores.reverse()

ordenada = sorted(valores)          # Devuelve una lista nueva
min(valores), max(valores), sum(valores)
```

> `lista.sort()` modifica la lista y devuelve `None`. `sorted(lista)` devuelve una lista nueva.

### 3.4 Listas anidadas (matrices)

```python
matriz = [[1, 2, 3],
          [4, 5, 6]]
matriz[1][2]                               # 6
traspuesta = [list(fila) for fila in zip(*matriz)]   # [[1, 4], [2, 5], [3, 6]]
```

> Para crear una matriz vacía **no** use `[[0] * 3] * 2` (las filas serían el mismo objeto). Use `[[0] * 3 for _ in range(2)]`.

---

## 4. Tuplas

Secuencias **inmutables**. Útiles para datos que no deben cambiar y como claves de diccionario.

```python
punto = (3, 4)
color = 255, 128, 0          # Los paréntesis son opcionales
unico = (5,)                 # Tupla de un elemento: la coma es obligatoria
vacia = ()

punto[0]                     # 3
x, y = punto                 # Desempaquetado
```

### 4.1 Tuplas con nombre

```python
from typing import NamedTuple

class Punto(NamedTuple):
    x: float
    y: float

p = Punto(3, 4)
p.x, p.y                     # 3, 4
p._replace(x=10)             # Punto(x=10, y=4) (crea una nueva)
```

---

## 5. Diccionarios

Asocian **claves únicas** con valores. Desde Python 3.7 conservan el orden de inserción.

### 5.1 Creación y acceso

```python
cliente = {"nombre": "Ana", "edad": 34, "ciudad": "Madrid"}
otro = dict(nombre="Luis", edad=41)
desde_pares = dict([("a", 1), ("b", 2)])

cliente["nombre"]                    # 'Ana'
cliente["telefono"]                  # KeyError
cliente.get("telefono")              # None
cliente.get("telefono", "N/D")       # 'N/D'
"edad" in cliente                    # True (comprueba claves)
```

### 5.2 Modificación

```python
cliente["email"] = "ana@ejemplo.com"           # Añadir o reemplazar
cliente.update({"edad": 35, "vip": True})      # Varios a la vez
cliente |= {"activo": True}                    # Fusión en el sitio

cliente.setdefault("pedidos", []).append(1001) # Crea la clave si no existe
eliminado = cliente.pop("vip")                 # Extrae (KeyError si no existe)
cliente.pop("inexistente", None)               # Sin error
del cliente["activo"]

combinado = {**config_base, **config_usuario}  # Fusión (el segundo prevalece)
combinado = config_base | config_usuario       # Equivalente
```

### 5.3 Recorrido

```python
for clave in cliente:                  # Claves
    ...
for valor in cliente.values():
    ...
for clave, valor in cliente.items():
    print(f"{clave}: {valor}")
```

### 5.4 Diccionarios anidados

```python
empresa = {
    "nombre": "Acme",
    "empleados": [
        {"nombre": "Ana", "departamento": "Ventas"},
        {"nombre": "Luis", "departamento": "IT"},
    ],
    "direccion": {"ciudad": "Madrid", "cp": "28001"},
}
empresa["direccion"]["ciudad"]               # 'Madrid'
empresa["empleados"][1]["nombre"]            # 'Luis'
empresa.get("direccion", {}).get("pais", "España")   # Acceso seguro
```

> Las claves deben ser **hashables**: `str`, `int`, `tuple`… Nunca `list` ni `dict`.

---

## 6. Conjuntos

Colecciones **sin orden y sin duplicados**. Muy eficientes para comprobar pertenencia.

```python
etiquetas = {"python", "datos", "web"}
vacio = set()                          # {} crea un diccionario, no un conjunto
unicos = set([1, 2, 2, 3, 3, 3])       # {1, 2, 3}

etiquetas.add("ia")
etiquetas.discard("web")               # Sin error si no existe
etiquetas.remove("datos")              # KeyError si no existe
"python" in etiquetas                  # True, en tiempo constante
```

### 6.1 Operaciones de conjuntos

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b      # Unión               {1, 2, 3, 4, 5, 6}
a & b      # Intersección        {3, 4}
a - b      # Diferencia          {1, 2}
a ^ b      # Diferencia simétrica {1, 2, 5, 6}
a <= b     # ¿Es subconjunto?
a.isdisjoint(b)                  # ¿Sin elementos comunes?
```

Ejemplo útil:

```python
nuevos = set(clientes_hoy) - set(clientes_ayer)
```

### 6.2 Eliminar duplicados conservando el orden

```python
sin_duplicados = list(dict.fromkeys(["b", "a", "b", "c", "a"]))   # ['b', 'a', 'c']
```

---

## 7. Comprensiones

Sintaxis compacta para construir colecciones.

```python
cuadrados = [n ** 2 for n in range(10)]
pares = [n for n in range(20) if n % 2 == 0]
etiquetas = ["par" if n % 2 == 0 else "impar" for n in range(5)]
planos = [x for fila in matriz for x in fila]           # Aplanar

precios_iva = {producto: precio * 1.21 for producto, precio in precios.items()}
caros = {p: v for p, v in precios.items() if v > 100}
invertido = {v: k for k, v in diccionario.items()}

dominios = {email.split("@")[1] for email in emails}    # Conjunto
```

### 7.1 Expresiones generadoras

Igual que una comprensión de lista, pero con paréntesis: **no crean la lista en memoria**.

```python
total = sum(linea.importe for linea in lineas)
hay_negativos = any(x < 0 for x in valores)
todos_validos = all(es_valido(r) for r in registros)
```

> Si la comprensión ocupa más de una línea o tiene varias condiciones, use un bucle `for` normal: la legibilidad es prioritaria.

---

## 8. Desempaquetado

```python
nombre, edad, ciudad = ("Ana", 34, "Madrid")

primero, *resto = [1, 2, 3, 4]          # 1, [2, 3, 4]
primero, *_, ultimo = [1, 2, 3, 4]      # 1, 4   (_ = valor que se ignora)

for nombre, (x, y) in [("A", (1, 2)), ("B", (3, 4))]:
    ...

def media(*valores): ...
media(*[4, 6, 8])                       # Pasa la lista como argumentos sueltos
crear(**{"nombre": "Ana", "edad": 34})  # Pasa el diccionario como argumentos con nombre

combinada = [*lista_a, *lista_b]
```

---

## 9. Ordenar, filtrar y agregar

### 9.1 Ordenar con `key`

```python
empleados = [
    {"nombre": "Ana", "salario": 42000, "dpto": "Ventas"},
    {"nombre": "Luis", "salario": 38000, "dpto": "IT"},
    {"nombre": "Eva", "salario": 51000, "dpto": "IT"},
]

sorted(empleados, key=lambda e: e["salario"], reverse=True)
sorted(empleados, key=lambda e: (e["dpto"], -e["salario"]))   # Varios criterios

from operator import itemgetter
sorted(empleados, key=itemgetter("dpto", "nombre"))

sorted(nombres, key=str.lower)                # Sin distinguir mayúsculas
max(empleados, key=lambda e: e["salario"])    # Empleado con mayor salario
```

### 9.2 Filtrar y transformar

```python
it = [e for e in empleados if e["dpto"] == "IT"]
nombres = [e["nombre"] for e in empleados]

# Alternativas funcionales (menos habituales que las comprensiones)
list(filter(lambda e: e["salario"] > 40000, empleados))
list(map(str.upper, nombres))
```

### 9.3 Agrupar

```python
from collections import defaultdict

por_dpto = defaultdict(list)
for e in empleados:
    por_dpto[e["dpto"]].append(e["nombre"])
# {'Ventas': ['Ana'], 'IT': ['Luis', 'Eva']}

total_por_dpto = defaultdict(int)
for e in empleados:
    total_por_dpto[e["dpto"]] += e["salario"]
```

> `itertools.groupby` también agrupa, pero exige que los datos estén **previamente ordenados** por la clave.

---

## 10. Módulo `collections`

| Clase | Uso |
|---|---|
| `Counter` | Contar apariciones. |
| `defaultdict` | Diccionario con valor por defecto para claves nuevas. |
| `deque` | Cola de doble extremo, eficiente al añadir/quitar por ambos lados. |
| `OrderedDict` | Diccionario con operaciones de reordenación (`move_to_end`). |
| `ChainMap` | Vista combinada de varios diccionarios (configuración por capas). |

### 10.1 `Counter`

```python
from collections import Counter

palabras = "el gato y el perro y el pez".split()
conteo = Counter(palabras)
conteo["el"]                 # 3
conteo.most_common(2)        # [('el', 3), ('y', 2)]
conteo.update(["gato"])
conteo.total()               # Suma de todos los recuentos
```

### 10.2 `deque`

```python
from collections import deque

cola = deque(maxlen=5)       # Conserva solo los 5 últimos
cola.append("a"); cola.appendleft("z")
cola.pop(); cola.popleft()
cola.rotate(1)
```

> Use `deque` en lugar de `list.pop(0)`: en listas largas, extraer por el principio es lento.

---

## 11. Copias y mutabilidad

La asignación **no copia**: ambas variables apuntan al mismo objeto.

```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)                     # [1, 2, 3, 4]  ← también cambia
```

| Operación | Resultado |
|---|---|
| `b = a` | Misma lista |
| `b = a.copy()` / `list(a)` / `a[:]` | Copia superficial |
| `b = copy.deepcopy(a)` | Copia profunda (incluye objetos anidados) |

```python
import copy

original = {"cliente": "Ana", "pedidos": [1, 2]}
superficial = original.copy()
superficial["pedidos"].append(3)       # ⚠ Modifica también el original
profunda = copy.deepcopy(original)     # Independiente por completo
```

---

## 12. Elegir la estructura adecuada

| Necesidad | Estructura |
|---|---|
| Colección ordenada que cambia | `list` |
| Registro fijo de pocos campos | `tuple` / `NamedTuple` / `dataclass` |
| Búsqueda por identificador | `dict` |
| Eliminar duplicados / pertenencia rápida | `set` |
| Contar frecuencias | `Counter` |
| Agrupar por categoría | `defaultdict(list)` |
| Cola FIFO o historial limitado | `deque` |
| Grandes volúmenes numéricos | NumPy / pandas |

**Coste de operaciones habituales:**

| Operación | `list` | `dict` / `set` |
|---|---|---|
| `x in coleccion` | Lento (recorre todo) | Rápido (constante) |
| Acceso por índice/clave | Rápido | Rápido |
| Añadir al final | Rápido | Rápido |
| Insertar/extraer al inicio | Lento | — |

---

## 13. Ejemplo práctico completo

**Análisis de ventas**: a partir de una lista de registros, calcula ventas por región y producto, ranking de vendedores, ticket medio y detecta clientes recurrentes, mostrando un informe por consola.

```python
"""
Análisis de ventas con estructuras de datos de la librería estándar.
"""
from collections import Counter, defaultdict
from statistics import mean
from typing import NamedTuple


class Venta(NamedTuple):
    id: int
    fecha: str
    cliente: str
    vendedor: str
    region: str
    producto: str
    unidades: int
    precio: float

    @property
    def importe(self) -> float:
        return self.unidades * self.precio


DATOS = [
    (1, "2026-01-05", "Acme", "Ana", "Norte", "Portátil", 2, 899.0),
    (2, "2026-01-07", "Beta", "Luis", "Sur", "Monitor", 5, 189.9),
    (3, "2026-01-12", "Acme", "Ana", "Norte", "Ratón", 20, 19.5),
    (4, "2026-01-15", "Gamma", "Eva", "Este", "Portátil", 1, 949.0),
    (5, "2026-02-02", "Beta", "Luis", "Sur", "Teclado", 10, 59.9),
    (6, "2026-02-10", "Delta", "Eva", "Este", "Monitor", 3, 199.0),
    (7, "2026-02-18", "Acme", "Luis", "Norte", "Monitor", 4, 189.9),
    (8, "2026-03-01", "Gamma", "Ana", "Este", "Ratón", 15, 18.0),
    (9, "2026-03-09", "Épsilon", "Eva", "Sur", "Portátil", 3, 879.0),
    (10, "2026-03-20", "Beta", "Ana", "Sur", "Teclado", 8, 62.0),
]


def cargar() -> list[Venta]:
    return [Venta(*fila) for fila in DATOS]


def agrupar_importes(ventas: list[Venta], campo: str) -> dict[str, float]:
    totales: defaultdict[str, float] = defaultdict(float)
    for v in ventas:
        totales[getattr(v, campo)] += v.importe
    return dict(sorted(totales.items(), key=lambda par: par[1], reverse=True))


def imprimir_tabla(titulo: str, datos: dict[str, float], total: float) -> None:
    print(f"\n{titulo}")
    print("-" * 44)
    for clave, valor in datos.items():
        barra = "█" * round(valor / total * 30)
        print(f"{clave:<12}{valor:>12,.2f} €  {barra}")


def main() -> None:
    ventas = cargar()
    total = sum(v.importe for v in ventas)

    print("=" * 44)
    print("INFORME DE VENTAS · 1.er trimestre 2026")
    print("=" * 44)
    print(f"Operaciones:   {len(ventas)}")
    print(f"Facturación:   {total:,.2f} €")
    print(f"Ticket medio:  {mean(v.importe for v in ventas):,.2f} €")

    imprimir_tabla("Ventas por región", agrupar_importes(ventas, "region"), total)
    imprimir_tabla("Ventas por producto", agrupar_importes(ventas, "producto"), total)
    imprimir_tabla("Ranking de vendedores", agrupar_importes(ventas, "vendedor"), total)

    por_mes: defaultdict[str, list[Venta]] = defaultdict(list)
    for v in ventas:
        por_mes[v.fecha[:7]].append(v)
    print("\nEvolución mensual")
    print("-" * 44)
    for mes, lista in sorted(por_mes.items()):
        print(f"{mes}   {len(lista):>2} operaciones   {sum(v.importe for v in lista):>10,.2f} €")

    compras = Counter(v.cliente for v in ventas)
    recurrentes = {c: n for c, n in compras.items() if n > 1}
    print("\nClientes recurrentes:", ", ".join(f"{c} ({n})" for c, n in compras.most_common() if c in recurrentes))

    unidades = Counter()
    for v in ventas:
        unidades[v.producto] += v.unidades
    producto, cantidad = unidades.most_common(1)[0]
    print(f"Producto con más unidades: {producto} ({cantidad} uds.)")

    mayor = max(ventas, key=lambda v: v.importe)
    print(f"Mayor operación: n.º {mayor.id} · {mayor.cliente} · {mayor.importe:,.2f} €")

    regiones_por_vendedor: defaultdict[str, set[str]] = defaultdict(set)
    for v in ventas:
        regiones_por_vendedor[v.vendedor].add(v.region)
    multirregion = sorted(n for n, r in regiones_por_vendedor.items() if len(r) > 1)
    print("Vendedores en varias regiones:", ", ".join(multirregion))


if __name__ == "__main__":
    main()
```

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `IndexError: list index out of range` | Índice fuera de la lista. | Compruebe `len()` o recorra con `for`. |
| `KeyError: 'clave'` | Clave inexistente en un diccionario. | Use `dict.get(clave, defecto)` o `in`. |
| `TypeError: unhashable type: 'list'` | Lista usada como clave o elemento de conjunto. | Convierta a `tuple`. |
| `TypeError: 'tuple' object does not support item assignment` | Intento de modificar una tupla. | Cree una nueva o use una lista. |
| `AttributeError: 'NoneType' object has no attribute...` | `lista = lista.sort()` deja `None`. | Use `lista.sort()` o `lista = sorted(lista)`. |
| Cambios inesperados en otra variable | Asignación sin copia (`b = a`). | Use `.copy()` o `copy.deepcopy()`. |
| Todas las filas de la matriz cambian a la vez | `[[0] * n] * m`. | `[[0] * n for _ in range(m)]`. |
| `{}` no es un conjunto vacío | `{}` crea un diccionario. | Use `set()`. |
| `RuntimeError: dictionary changed size during iteration` | Se modifica el diccionario mientras se recorre. | Recorra una copia: `for k in list(d):`. |
| Elementos omitidos al borrar dentro de un `for` | Se modifica la lista mientras se recorre. | Cree una nueva lista con una comprensión. |

---

## 15. Buenas prácticas

- Elija la estructura según el uso: búsquedas frecuentes → `dict`/`set`; orden y duplicados → `list`.
- Use comprensiones para transformaciones sencillas; bucles para lógica compleja.
- Use `dict.get()` y `setdefault()` en lugar de comprobar claves manualmente.
- Use `Counter` y `defaultdict` en vez de reimplementar contadores y agrupaciones.
- Prefiera expresiones generadoras dentro de `sum`, `any`, `all`, `max`, `min`.
- No modifique una colección mientras la recorre.
- Para registros con campos fijos, use `NamedTuple` o `dataclass` en vez de diccionarios sueltos.
- Recuerde que la asignación no copia: use copias explícitas cuando sea necesario.
- Anote los tipos de las colecciones: `list[str]`, `dict[str, int]`, `set[int]`.

---

## 16. Referencia rápida

```python
l = [1, 2]; l.append(x) .insert(i, x) .extend(it) .remove(x) .pop(i) .sort(key=f) .index(x) .count(x)
t = (1, 2); x, y = t; t = (5,)
d = {"k": v}; d["k"] d.get("k", def) d.items() d.keys() d.values() d.update(o) d.pop("k") d | o
s = {1, 2}; s.add(x) .discard(x)  a | b  a & b  a - b  a ^ b   set()  ← vacío
[f(x) for x in it if c]   {k: v for k, v in d.items()}   {x for x in it}   (x for x in it)
sorted(it, key=lambda x: ..., reverse=True) / min / max / sum / any / all / len / zip / enumerate
primero, *resto = lista   /   [*a, *b]   /   {**d1, **d2}
Counter(it).most_common(n) / defaultdict(list) / deque(maxlen=n) / copy.deepcopy(obj)
```

---

## 17. Ejercicios propuestos

1. Dada una lista de palabras, obtenga las 5 más frecuentes ignorando mayúsculas y signos de puntuación.
2. Cree una agenda de contactos con diccionarios que permita añadir, buscar, modificar y eliminar contactos.
3. Dadas dos listas de correos electrónicos, obtenga los que están en ambas, solo en la primera y en total sin duplicados.
4. Agrupe una lista de alumnos y notas por curso y calcule la media, la nota máxima y el alumno con mejor nota de cada curso.
5. Amplíe el ejemplo completo para calcular la variación porcentual de facturación de cada mes respecto al anterior.

---

## 18. Referencias

- Estructuras de datos (tutorial oficial): https://docs.python.org/es/3/tutorial/datastructures.html
- Tipos integrados: https://docs.python.org/es/3/library/stdtypes.html
- Módulo `collections`: https://docs.python.org/es/3/library/collections.html
- Complejidad de las operaciones: https://wiki.python.org/moin/TimeComplexity
