# Manual de uso: Typing

> **Categoría:** Utilidades generales · **Utilidad:** Añadir tipado al código
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tipos básicos](#4-tipos-básicos)
5. [Colecciones](#5-colecciones)
6. [Tipos opcionales y uniones](#6-tipos-opcionales-y-uniones)
7. [Tipos intermedios](#7-tipos-intermedios)
8. [Tipos avanzados](#8-tipos-avanzados)
9. [Verificación con mypy](#9-verificación-con-mypy)
10. [Ejemplo práctico completo](#10-ejemplo-práctico-completo)
11. [Errores frecuentes](#11-errores-frecuentes)
12. [Buenas prácticas](#12-buenas-prácticas)
13. [Referencia rápida](#13-referencia-rápida)
14. [Ejercicios propuestos](#14-ejercicios-propuestos)
15. [Referencias](#15-referencias)

---

## 1. Descripción

Las **anotaciones de tipo** (*type hints*) permiten indicar qué tipo de datos esperan y devuelven las variables y funciones. El módulo `typing` proporciona las herramientas para expresar tipos complejos.

**Ventajas:**
- Detecta errores antes de ejecutar el código (con herramientas como **mypy** o **Pyright**).
- Mejora el autocompletado y la navegación en el editor (VS Code, PyCharm).
- Sirve como documentación viva del código.
- Es la base del funcionamiento de librerías como **Pydantic** y **FastAPI**.

> **Importante:** Python **no** comprueba los tipos en tiempo de ejecución. Las anotaciones son información para herramientas y personas.

---

## 2. Instalación e importación

No requiere instalación.

```python
from typing import Any, Literal, Callable, TypedDict, Protocol, TypeVar, Final
```

Para verificar los tipos se recomienda instalar un comprobador:

```bash
pip install mypy
```

> Este manual usa la sintaxis moderna (Python 3.10+): `list[int]`, `int | None`. En versiones anteriores se usaba `List[int]`, `Optional[int]`, importados de `typing`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Anotación de variable | `edad: int = 30` |
| Anotación de función | `def f(x: int) -> str:` |
| Comprobador estático | Herramienta que analiza el código sin ejecutarlo (mypy, Pyright). |
| Genérico | Tipo parametrizado: `list[str]`, `dict[str, int]`. |
| Alias de tipo | Nombre para un tipo complejo reutilizable. |

---

## 4. Tipos básicos

### 4.1 Variables

```python
nombre: str = "Ana"
edad: int = 28
altura: float = 1.68
activo: bool = True
datos: bytes = b"\x00\x01"
nada: None = None
```

### 4.2 Funciones

```python
def saludar(nombre: str, veces: int = 1) -> str:
    return ("Hola, " + nombre + ". ") * veces


def registrar(mensaje: str) -> None:      # No devuelve nada
    print(mensaje)
```

### 4.3 `Any`: cualquier tipo

```python
from typing import Any

def mostrar(valor: Any) -> None:
    print(valor)
```

> Úselo con moderación: desactiva la comprobación de tipos.

---

## 5. Colecciones

```python
numeros: list[int] = [1, 2, 3]
precios: dict[str, float] = {"teclado": 29.9, "raton": 14.5}
etiquetas: set[str] = {"nuevo", "oferta"}

coordenada: tuple[float, float] = (40.41, -3.70)      # Tamaño fijo
valores: tuple[int, ...] = (1, 2, 3, 4)               # Tamaño variable

matriz: list[list[float]] = [[1.0, 2.0], [3.0, 4.0]]
pedidos: dict[int, list[str]] = {1: ["A", "B"], 2: ["C"]}
```

### 5.1 Tipos abstractos para parámetros

Para aceptar cualquier secuencia o iterable (no solo listas):

```python
from collections.abc import Iterable, Sequence, Mapping

def total(importes: Iterable[float]) -> float:
    return sum(importes)

total([1.0, 2.5])          # Lista
total((1.0, 2.5))          # Tupla
total(x for x in [1.0])    # Generador
```

| Tipo | Admite |
|---|---|
| `Iterable[T]` | Cualquier cosa recorrible con `for`. |
| `Sequence[T]` | Listas, tuplas, cadenas (con índice y longitud). |
| `Mapping[K, V]` | Diccionarios y similares (solo lectura). |
| `MutableMapping[K, V]` | Diccionarios modificables. |

---

## 6. Tipos opcionales y uniones

### 6.1 Uniones

```python
def formatear(valor: int | float) -> str:
    return f"{valor:.2f}"

identificador: int | str = "A-102"
```

### 6.2 Valores opcionales

```python
def buscar_usuario(id_usuario: int) -> dict | None:
    """Devuelve el usuario o None si no existe."""
    ...

usuario = buscar_usuario(5)
if usuario is not None:              # El comprobador exige verificar None
    print(usuario["nombre"])
```

### 6.3 Parámetros opcionales con valor por defecto `None`

```python
def conectar(host: str, puerto: int | None = None) -> None:
    puerto = puerto or 5432
```

---

## 7. Tipos intermedios

### 7.1 `Literal`: valores concretos permitidos

```python
from typing import Literal

Nivel = Literal["DEBUG", "INFO", "WARNING", "ERROR"]

def configurar(nivel: Nivel) -> None: ...

configurar("INFO")        # Correcto
configurar("TRACE")       # mypy: error
```

### 7.2 Alias de tipo

```python
Coordenada = tuple[float, float]
Ruta = list[Coordenada]

def distancia_total(ruta: Ruta) -> float: ...
```

En Python 3.12+:

```python
type Coordenada = tuple[float, float]
```

### 7.3 `Callable`: funciones como parámetros

```python
from collections.abc import Callable

def aplicar(valores: list[float], funcion: Callable[[float], float]) -> list[float]:
    return [funcion(v) for v in valores]

aplicar([1, 2, 3], lambda x: x * 1.21)
```

`Callable[[int, str], bool]` → función que recibe `int` y `str` y devuelve `bool`.

### 7.4 `TypedDict`: diccionarios con estructura

```python
from typing import TypedDict, NotRequired

class Cliente(TypedDict):
    id: int
    nombre: str
    email: str
    telefono: NotRequired[str]        # Clave opcional

def enviar_bienvenida(cliente: Cliente) -> None:
    print(f"Enviando email a {cliente['email']}")

enviar_bienvenida({"id": 1, "nombre": "Ana", "email": "ana@ejemplo.com"})
```

### 7.5 Clases de datos tipadas

```python
from dataclasses import dataclass, field

@dataclass
class Producto:
    codigo: str
    nombre: str
    precio: float
    etiquetas: list[str] = field(default_factory=list)
```

### 7.6 `NamedTuple`

```python
from typing import NamedTuple

class Punto(NamedTuple):
    x: float
    y: float

p = Punto(1.5, 2.0)
p.x
```

### 7.7 `Final` y `ClassVar`

```python
from typing import Final, ClassVar

IVA: Final = 0.21          # No debe reasignarse

class Config:
    instancias: ClassVar[int] = 0    # Atributo de clase, no de instancia
```

---

## 8. Tipos avanzados

### 8.1 Genéricos con `TypeVar`

```python
from typing import TypeVar

T = TypeVar("T")

def primero(elementos: list[T]) -> T:
    return elementos[0]

primero([1, 2, 3])          # El comprobador infiere int
primero(["a", "b"])         # Infiere str
```

Sintaxis de Python 3.12+:

```python
def primero[T](elementos: list[T]) -> T:
    return elementos[0]
```

### 8.2 Clases genéricas

```python
from typing import Generic, TypeVar

T = TypeVar("T")

class Repositorio(Generic[T]):
    def __init__(self) -> None:
        self._datos: dict[int, T] = {}

    def guardar(self, id_: int, elemento: T) -> None:
        self._datos[id_] = elemento

    def obtener(self, id_: int) -> T | None:
        return self._datos.get(id_)

repo_productos: Repositorio[Producto] = Repositorio()
```

### 8.3 `Protocol`: tipado estructural (*duck typing*)

```python
from typing import Protocol

class Exportable(Protocol):
    def exportar(self) -> str: ...

class Factura:
    def exportar(self) -> str:
        return "FACTURA"

class Informe:
    def exportar(self) -> str:
        return "INFORME"

def guardar(doc: Exportable) -> None:
    print(doc.exportar())

guardar(Factura())   # Válido: tiene el método exportar, aunque no herede de Exportable
```

### 8.4 `Self` (Python 3.11+)

```python
from typing import Self

class Consulta:
    def filtrar(self, campo: str) -> Self:
        ...
        return self
```

### 8.5 `overload`: varias firmas para una función

```python
from typing import overload

@overload
def convertir(valor: str) -> int: ...
@overload
def convertir(valor: int) -> str: ...

def convertir(valor: str | int) -> int | str:
    return int(valor) if isinstance(valor, str) else str(valor)
```

### 8.6 `TYPE_CHECKING`: evitar importaciones circulares

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from modelos import Pedido      # Solo se importa al comprobar tipos

def procesar(pedido: "Pedido") -> None: ...
```

### 8.7 `cast` y `Annotated`

```python
from typing import cast, Annotated

valor = cast(int, obtener_valor())          # Indica al comprobador el tipo real

Edad = Annotated[int, "entre 0 y 120"]      # Metadatos (usados por Pydantic/FastAPI)
```

### 8.8 Consultar anotaciones en tiempo de ejecución

```python
from typing import get_type_hints
get_type_hints(saludar)     # {'nombre': str, 'veces': int, 'return': str}
```

---

## 9. Verificación con mypy

### 9.1 Ejecutar

```bash
mypy mi_script.py
mypy src/ --strict
```

### 9.2 Ejemplo de detección

```python
def doble(x: int) -> int:
    return x * 2

doble("3")
```

```
error: Argument 1 to "doble" has incompatible type "str"; expected "int"
```

### 9.3 Configuración en `pyproject.toml`

```toml
[tool.mypy]
python_version = "3.12"
strict = true
ignore_missing_imports = true
```

### 9.4 Ignorar una línea concreta

```python
resultado = libreria_sin_tipos.funcion()  # type: ignore[no-untyped-call]
```

> En VS Code, la extensión **Pylance** (Pyright) muestra los errores de tipo mientras escribe. Active `"python.analysis.typeCheckingMode": "basic"` o `"strict"`.

---

## 10. Ejemplo práctico completo

Sistema de inventario totalmente tipado:

```python
from __future__ import annotations

from collections.abc import Iterable
from dataclasses import dataclass, field
from typing import Literal, Protocol, TypedDict

Categoria = Literal["hardware", "software", "servicio"]


@dataclass
class Producto:
    codigo: str
    nombre: str
    categoria: Categoria
    precio: float
    stock: int = 0
    etiquetas: list[str] = field(default_factory=list)

    def valor_inventario(self) -> float:
        return self.precio * self.stock


class Resumen(TypedDict):
    total_productos: int
    valor_total: float
    sin_stock: list[str]


class Notificador(Protocol):
    def enviar(self, mensaje: str) -> None: ...


class NotificadorConsola:
    def enviar(self, mensaje: str) -> None:
        print(f"[AVISO] {mensaje}")


class Inventario:
    def __init__(self, notificador: Notificador) -> None:
        self._productos: dict[str, Producto] = {}
        self._notificador = notificador

    def agregar(self, productos: Iterable[Producto]) -> None:
        for p in productos:
            self._productos[p.codigo] = p

    def buscar(self, codigo: str) -> Producto | None:
        return self._productos.get(codigo)

    def por_categoria(self, categoria: Categoria) -> list[Producto]:
        return [p for p in self._productos.values() if p.categoria == categoria]

    def retirar(self, codigo: str, unidades: int) -> bool:
        producto = self.buscar(codigo)
        if producto is None or producto.stock < unidades:
            return False
        producto.stock -= unidades
        if producto.stock == 0:
            self._notificador.enviar(f"{producto.nombre} se ha quedado sin stock")
        return True

    def resumen(self) -> Resumen:
        productos = self._productos.values()
        return {
            "total_productos": len(self._productos),
            "valor_total": round(sum(p.valor_inventario() for p in productos), 2),
            "sin_stock": [p.nombre for p in productos if p.stock == 0],
        }


if __name__ == "__main__":
    inv = Inventario(NotificadorConsola())
    inv.agregar([
        Producto("H01", "Teclado", "hardware", 29.9, 10),
        Producto("H02", "Monitor", "hardware", 189.0, 1),
        Producto("S01", "Antivirus", "software", 39.0, 50),
    ])
    inv.retirar("H02", 1)
    print(inv.resumen())
```

Compruebe el código con:

```bash
mypy --strict inventario.py
```

---

## 11. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `TypeError: 'type' object is not subscriptable` | Uso de `list[int]` en Python < 3.9. | Actualice Python o use `from __future__ import annotations`. |
| `TypeError: unsupported operand type(s) for \|` | Uso de `int \| None` en Python < 3.10. | Use `Optional[int]` o `from __future__ import annotations`. |
| El código se ejecuta aunque los tipos sean incorrectos | Python no valida tipos en ejecución. | Use mypy/Pyright, o Pydantic para validar datos. |
| `Item "None" of "X \| None" has no attribute` | No se comprueba `None`. | Añada `if valor is not None:`. |
| Argumento mutable por defecto (`list = []`) | Lista compartida entre llamadas. | Use `None` o `field(default_factory=list)`. |
| Referencia a una clase aún no definida | Orden de declaración. | Use comillas (`"Clase"`) o `from __future__ import annotations`. |

---

## 12. Buenas prácticas

- Anote siempre los parámetros y el valor de retorno de las funciones públicas.
- Use tipos abstractos (`Iterable`, `Sequence`, `Mapping`) en parámetros y tipos concretos en valores de retorno.
- Evite `Any` salvo que sea realmente necesario.
- Use `Literal` en lugar de cadenas libres para opciones fijas.
- Integre mypy o Pyright en el editor y en la integración continua.
- Introduzca el tipado de forma gradual en proyectos existentes.
- Para validar datos externos en ejecución (APIs, formularios), combine tipado con **Pydantic**.

---

## 13. Referencia rápida

```python
x: int                         def f(a: str, b: int = 0) -> bool:
list[int]  dict[str, float]    set[str]  tuple[int, str]  tuple[int, ...]
int | None                     Literal["a", "b"]
Callable[[int], str]           Iterable[T]  Sequence[T]  Mapping[K, V]
TypedDict  NamedTuple          @dataclass
TypeVar("T")  Generic[T]       Protocol
Final  ClassVar  Self          cast()  Annotated[T, meta]
TYPE_CHECKING                  get_type_hints()
```

---

## 14. Ejercicios propuestos

1. Añada anotaciones de tipo completas a un script propio y corrija todos los errores que indique `mypy --strict`.
2. Defina un `TypedDict` para la respuesta de una API y escriba una función que la procese.
3. Cree una clase genérica `Cola[T]` con métodos `encolar`, `desencolar` y `vacia`.
4. Defina un `Protocol` llamado `Pagable` y dos clases que lo cumplan sin heredar de él.

---

## 15. Referencias

- Documentación oficial: https://docs.python.org/es/3/library/typing.html
- mypy: https://mypy.readthedocs.io/
- Guía de tipado de Python: https://typing.python.org/
