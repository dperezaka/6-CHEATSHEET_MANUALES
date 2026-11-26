# Manual de uso: Type hints (anotaciones de tipo)

> **Categoría:** Lenguaje · **Utilidad:** Anotaciones de tipo que documentan el código y permiten detectar errores
> **Tipo:** Característica del lenguaje + módulo estándar `typing` (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Requisitos y versiones](#2-requisitos-y-versiones)
3. [Conceptos clave](#3-conceptos-clave)
4. [Anotaciones básicas](#4-anotaciones-básicas)
5. [Colecciones](#5-colecciones)
6. [Uniones, opcionales y valores literales](#6-uniones-opcionales-y-valores-literales)
7. [Funciones como valores: `Callable`](#7-funciones-como-valores-callable)
8. [Estructuras de datos tipadas](#8-estructuras-de-datos-tipadas)
9. [Clases: `Self`, `ClassVar`, `Final`, `@override`](#9-clases-self-classvar-final-override)
10. [Genéricos](#10-genéricos)
11. [Protocolos (tipado estructural)](#11-protocolos-tipado-estructural)
12. [Alias de tipo y `NewType`](#12-alias-de-tipo-y-newtype)
13. [Estrechamiento de tipos](#13-estrechamiento-de-tipos)
14. [Sobrecargas, `ParamSpec` y decoradores](#14-sobrecargas-paramspec-y-decoradores)
15. [`Any`, `object`, `Never` y `cast`](#15-any-object-never-y-cast)
16. [Anotaciones en tiempo de ejecución](#16-anotaciones-en-tiempo-de-ejecución)
17. [Importaciones circulares y `TYPE_CHECKING`](#17-importaciones-circulares-y-type_checking)
18. [Archivos de stubs y paquetes tipados](#18-archivos-de-stubs-y-paquetes-tipados)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

Las **anotaciones de tipo** (*type hints*) indican qué tipo de valor se espera en variables, parámetros y retornos:

```python
def precio_con_iva(base: float, tipo: float = 0.21) -> float:
    return round(base * (1 + tipo), 2)
```

Python **no las comprueba al ejecutar**: son información para las personas y para herramientas como **mypy**, **Pyright**, **ty**, los IDE y librerías como **Pydantic**, **FastAPI** o **dataclasses**.

**Ventajas:**
- Documentan el código de forma precisa y siempre actualizada.
- Permiten detectar errores antes de ejecutar (con un comprobador de tipos).
- Mejoran el autocompletado, la navegación y la refactorización en el IDE.
- Habilitan validación automática en tiempo de ejecución (Pydantic, FastAPI, Typer).

**Limitaciones:**
- No impiden ejecutar código con tipos incorrectos: hace falta un comprobador.
- Algunos patrones muy dinámicos son difíciles de tipar con precisión.
- Requieren disciplina para mantenerlas en toda la base de código.

---

## 2. Requisitos y versiones

Este manual usa la sintaxis de **Python 3.12+**. Hitos principales:

| Versión | Novedad |
|---|---|
| 3.9 | Genéricos en colecciones integradas: `list[int]`, `dict[str, int]`. |
| 3.10 | Uniones con `|`: `int | None`. `ParamSpec`, `TypeAlias`, `TypeGuard`. |
| 3.11 | `Self`, `Never`, `LiteralString`, `Required`/`NotRequired`, `assert_never`. |
| 3.12 | Sintaxis de genéricos `class Caja[T]`, `def f[T]`, sentencia `type`. `@override`, `Unpack` para `**kwargs`. |
| 3.13 | Valores por defecto en parámetros de tipo, `TypeIs`, `ReadOnly` en `TypedDict`, `@deprecated`. |
| 3.14 | Evaluación diferida de anotaciones (PEP 649/749) y módulo `annotationlib`. |

Para usar novedades en versiones anteriores, instale `typing_extensions`:

```python
from typing_extensions import TypeIs, override   # Funciona en versiones antiguas
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Anotación | Tipo indicado tras `:` (variables, parámetros) o `->` (retorno). |
| Comprobador de tipos | Herramienta que analiza el código sin ejecutarlo (mypy, Pyright, ty). |
| Tipado gradual | Se puede anotar poco a poco; lo no anotado se trata como `Any`. |
| Genérico | Tipo parametrizado por otro tipo: `list[int]`, `Caja[str]`. |
| Tipado nominal | Compatibilidad por herencia (clases). |
| Tipado estructural | Compatibilidad por forma: tener los métodos necesarios (`Protocol`). |
| Estrechamiento | El comprobador deduce un tipo más concreto tras un `if`, `isinstance`, etc. |
| Varianza | Cómo se relacionan `Caja[Perro]` y `Caja[Animal]` (covariante, contravariante, invariante). |
| Stub | Archivo `.pyi` con solo las firmas tipadas de un módulo. |

---

## 4. Anotaciones básicas

```python
nombre: str = "Ana"
edad: int = 34
saldo: float = 1250.75
activo: bool = True
datos: bytes = b"\x00\x01"
nada: None = None

# Declarar sin asignar
resultado: int


def saludar(nombre: str, veces: int = 1) -> str:
    return ", ".join([f"Hola {nombre}"] * veces)


def registrar(mensaje: str) -> None:        # No devuelve nada
    print(mensaje)
```

> `int` es compatible donde se espera `float`, y `float` donde se espera `complex` (regla de promoción numérica).

---

## 5. Colecciones

```python
numeros: list[int] = [1, 2, 3]
precios: dict[str, float] = {"teclado": 59.9}
etiquetas: set[str] = {"nuevo", "oferta"}
punto: tuple[float, float] = (40.4, -3.7)          # Longitud fija
valores: tuple[int, ...] = (1, 2, 3, 4)            # Longitud variable
matriz: list[list[float]] = [[1.0, 2.0], [3.0, 4.0]]
```

Para **parámetros**, prefiera los tipos abstractos de `collections.abc`: aceptan más tipos concretos.

```python
from collections.abc import Iterable, Iterator, Mapping, Sequence, MutableMapping


def total(importes: Iterable[float]) -> float:            # Acepta list, tuple, set, generador…
    return sum(importes)


def primero(elementos: Sequence[str]) -> str:              # Admite indexación y len()
    return elementos[0]


def mostrar(config: Mapping[str, str]) -> None:            # Solo lectura
    for clave, valor in config.items():
        print(clave, valor)


def contar_hasta(n: int) -> Iterator[int]:                 # Generador
    yield from range(n)
```

| Para… | Use en parámetros | Use en retornos |
|---|---|---|
| Recorrer | `Iterable[T]` | `list[T]`, `Iterator[T]` |
| Indexar / `len` | `Sequence[T]` | `list[T]`, `tuple[T, ...]` |
| Diccionario de solo lectura | `Mapping[K, V]` | `dict[K, V]` |
| Diccionario modificable | `MutableMapping[K, V]` | `dict[K, V]` |

> **Regla:** sea **flexible en lo que acepta** (tipos abstractos) y **concreto en lo que devuelve**.

---

## 6. Uniones, opcionales y valores literales

```python
def buscar_cliente(id_: int) -> dict[str, str] | None:     # Puede no encontrar nada
    ...


def formatear(valor: int | float | str) -> str:
    return str(valor)


identificador: int | str = "A-001"
```

`X | None` equivale a `Optional[X]`. Prefiera la sintaxis con `|`.

Valores concretos con `Literal`:

```python
from typing import Literal

Modo = Literal["lectura", "escritura", "anexar"]


def abrir(ruta: str, modo: Modo = "lectura") -> None:
    ...


abrir("datos.txt", "escritura")    # ✔
abrir("datos.txt", "borrar")       # ✖ Error del comprobador
```

Enumeraciones (cuando los valores tienen comportamiento o se reutilizan mucho):

```python
from enum import StrEnum


class Estado(StrEnum):
    PENDIENTE = "pendiente"
    PAGADA = "pagada"
    ANULADA = "anulada"


def cambiar_estado(nuevo: Estado) -> None:
    ...
```

---

## 7. Funciones como valores: `Callable`

```python
from collections.abc import Callable, Awaitable

Validador = Callable[[str], bool]                          # Recibe str, devuelve bool


def aplicar(valor: str, validadores: list[Validador]) -> bool:
    return all(v(valor) for v in validadores)


def con_reintentos(funcion: Callable[[], int], intentos: int = 3) -> int:
    ...


Manejador = Callable[[dict[str, str]], Awaitable[None]]    # Función asíncrona
CualquierFirma = Callable[..., object]                      # Cualquier argumento
```

Para firmas con argumentos con nombre, use un `Protocol` con `__call__`:

```python
from typing import Protocol


class Notificador(Protocol):
    def __call__(self, destinatario: str, *, asunto: str, urgente: bool = False) -> None: ...
```

---

## 8. Estructuras de datos tipadas

### 8.1 `dataclass`

```python
from dataclasses import dataclass, field
from datetime import date


@dataclass(frozen=True, slots=True)
class Linea:
    producto: str
    cantidad: int
    precio: float

    @property
    def subtotal(self) -> float:
        return self.cantidad * self.precio


@dataclass
class Factura:
    numero: str
    fecha: date
    lineas: list[Linea] = field(default_factory=list)

    def total(self) -> float:
        return sum(l.subtotal for l in self.lineas)
```

### 8.2 `TypedDict`: diccionarios con claves conocidas

Útil para JSON, respuestas de APIs y configuraciones:

```python
from typing import NotRequired, ReadOnly, Required, TypedDict


class Usuario(TypedDict):
    id: ReadOnly[int]                 # 3.13+: no se puede modificar
    nombre: str
    email: str
    telefono: NotRequired[str]        # Clave opcional


class Opciones(TypedDict, total=False):     # Todas opcionales…
    idioma: str
    tema: str
    usuario_id: Required[int]               # …salvo esta


u: Usuario = {"id": 1, "nombre": "Ana", "email": "ana@ejemplo.com"}
```

### 8.3 `NamedTuple`

```python
from typing import NamedTuple


class Coordenada(NamedTuple):
    lat: float
    lon: float
    altitud: float = 0.0


madrid = Coordenada(40.4168, -3.7038)
lat, lon, _ = madrid
```

| Estructura | Mutable | Acceso | Cuándo |
|---|---|---|---|
| `dataclass` | Sí (o `frozen`) | Atributos | Modelos del dominio con métodos. |
| `TypedDict` | Sí | Claves | Datos JSON que ya son diccionarios. |
| `NamedTuple` | No | Atributos e índice | Registros pequeños e inmutables. |
| Pydantic `BaseModel` | Sí | Atributos | Datos externos que hay que **validar**. |

---

## 9. Clases: `Self`, `ClassVar`, `Final`, `@override`

```python
from typing import ClassVar, Final, Self, override

TASA_IVA: Final = 0.21                     # Constante: no se puede reasignar


class Consulta:
    tabla_por_defecto: ClassVar[str] = "clientes"     # Atributo de clase, no de instancia

    def __init__(self) -> None:
        self.filtros: list[str] = []
        self.limite_: int | None = None

    def donde(self, condicion: str) -> Self:          # Devuelve el tipo de la (sub)clase
        self.filtros.append(condicion)
        return self

    def limite(self, n: int) -> Self:
        self.limite_ = n
        return self

    @classmethod
    def nueva(cls) -> Self:
        return cls()


class ConsultaAuditada(Consulta):
    @override                                          # Error si el método no existe en la base
    def donde(self, condicion: str) -> Self:
        print("Filtro:", condicion)
        return super().donde(condicion)


q = ConsultaAuditada.nueva().donde("activo = 1").limite(10)    # Tipo: ConsultaAuditada
```

Clases abstractas:

```python
from abc import ABC, abstractmethod


class Repositorio(ABC):
    @abstractmethod
    def obtener(self, id_: int) -> dict[str, object] | None: ...
```

---

## 10. Genéricos

### 10.1 Funciones genéricas (sintaxis 3.12+)

```python
def primero[T](elementos: list[T]) -> T:
    return elementos[0]


x = primero([1, 2, 3])          # T = int
y = primero(["a", "b"])         # T = str
```

### 10.2 Restricciones

```python
from collections.abc import Hashable


def maximo[T: (int, float)](a: T, b: T) -> T:          # T es int o float (restricción)
    return a if a > b else b


def agrupar[K: Hashable, V](pares: list[tuple[K, V]]) -> dict[K, list[V]]:   # K con cota superior
    resultado: dict[K, list[V]] = {}
    for k, v in pares:
        resultado.setdefault(k, []).append(v)
    return resultado
```

### 10.3 Clases genéricas

```python
from dataclasses import dataclass


@dataclass
class Pagina[T]:
    elementos: list[T]
    total: int
    pagina: int
    tamano: int

    @property
    def paginas(self) -> int:
        return -(-self.total // self.tamano)


class Pila[T]:
    def __init__(self) -> None:
        self._datos: list[T] = []

    def apilar(self, x: T) -> None:
        self._datos.append(x)

    def desapilar(self) -> T:
        return self._datos.pop()


p = Pila[int]()
p.apilar(1)
p.apilar("a")         # ✖ Error del comprobador
```

### 10.4 Parámetros de tipo con valor por defecto (3.13+)

```python
class Respuesta[T = dict[str, object]]:
    def __init__(self, datos: T) -> None:
        self.datos = datos
```

### 10.5 Sintaxis anterior (Python < 3.12)

```python
from typing import Generic, TypeVar

T = TypeVar("T")


class Pila(Generic[T]):
    ...
```

---

## 11. Protocolos (tipado estructural)

Un `Protocol` define una **interfaz por forma**: cualquier clase con los métodos indicados es compatible, sin heredar.

```python
from typing import Protocol, runtime_checkable


@runtime_checkable
class Exportable(Protocol):
    def exportar(self) -> bytes: ...


class InformeCSV:
    def exportar(self) -> bytes:
        return b"a;b\n1;2\n"


class InformePDF:
    def exportar(self) -> bytes:
        return b"%PDF-1.7..."


def guardar(doc: Exportable, ruta: str) -> None:
    with open(ruta, "wb") as f:
        f.write(doc.exportar())


guardar(InformeCSV(), "informe.csv")      # ✔ Sin herencia
isinstance(InformePDF(), Exportable)      # True (gracias a @runtime_checkable)
```

> Los protocolos son ideales para **inyección de dependencias** y pruebas: el código depende de la interfaz, no de una clase concreta.

Protocolos estándar útiles: `collections.abc.Iterable`, `Sized`, `Container`, `typing.SupportsInt`, `SupportsFloat`, `SupportsIndex`.

---

## 12. Alias de tipo y `NewType`

### 12.1 Alias (sentencia `type`, 3.12+)

```python
type Json = dict[str, Json] | list[Json] | str | int | float | bool | None    # Recursivo
type Matriz = list[list[float]]
type Resultado[T] = tuple[T, None] | tuple[None, str]                          # Genérico


def leer_config(texto: str) -> Json: ...
```

### 12.2 `NewType`: tipos distintos con la misma representación

Evita confundir valores del mismo tipo base:

```python
from typing import NewType

ClienteId = NewType("ClienteId", int)
FacturaId = NewType("FacturaId", int)


def obtener_factura(id_: FacturaId) -> None: ...


cliente = ClienteId(42)
obtener_factura(cliente)                 # ✖ Error: ClienteId no es FacturaId
obtener_factura(FacturaId(1001))         # ✔
```

### 12.3 `Annotated`: metadatos adicionales

```python
from typing import Annotated

from pydantic import Field

Edad = Annotated[int, Field(ge=0, le=130)]          # Pydantic lo usa para validar
Email = Annotated[str, "formato: email"]            # Los comprobadores lo tratan como str
```

---

## 13. Estrechamiento de tipos

Los comprobadores deducen tipos más concretos tras ciertas comprobaciones:

```python
def longitud(valor: str | list[int] | None) -> int:
    if valor is None:
        return 0                     # Aquí: None
    if isinstance(valor, str):
        return len(valor.strip())    # Aquí: str
    return len(valor)                # Aquí: list[int]
```

### 13.1 `match` y exhaustividad

```python
from typing import assert_never


def describir(estado: Estado) -> str:
    match estado:
        case Estado.PENDIENTE:
            return "Pendiente de pago"
        case Estado.PAGADA:
            return "Pagada"
        case Estado.ANULADA:
            return "Anulada"
        case _:
            assert_never(estado)     # Error del comprobador si falta un caso
```

### 13.2 Funciones de comprobación: `TypeIs` (3.13+) y `TypeGuard`

```python
from typing import TypeIs


def es_lista_de_str(valor: list[object]) -> TypeIs[list[str]]:
    return all(isinstance(x, str) for x in valor)


def procesar(datos: list[object]) -> None:
    if es_lista_de_str(datos):
        print(", ".join(datos))      # datos: list[str]
```

> `TypeIs` estrecha en ambas ramas (`if` y `else`); `TypeGuard` solo en la rama verdadera. Prefiera `TypeIs`.

---

## 14. Sobrecargas, `ParamSpec` y decoradores

### 14.1 `@overload`: el tipo de retorno depende de los argumentos

```python
from typing import Literal, overload


@overload
def leer(ruta: str, binario: Literal[True]) -> bytes: ...
@overload
def leer(ruta: str, binario: Literal[False] = ...) -> str: ...
def leer(ruta: str, binario: bool = False) -> str | bytes:
    with open(ruta, "rb" if binario else "r") as f:
        return f.read()


texto = leer("a.txt")              # str
datos = leer("a.bin", True)        # bytes
```

### 14.2 Decoradores que conservan la firma

```python
import functools
import time
from collections.abc import Callable


def cronometrar[**P, R](funcion: Callable[P, R]) -> Callable[P, R]:
    @functools.wraps(funcion)
    def envoltura(*args: P.args, **kwargs: P.kwargs) -> R:
        inicio = time.perf_counter()
        try:
            return funcion(*args, **kwargs)
        finally:
            print(f"{funcion.__name__}: {time.perf_counter() - inicio:.3f} s")
    return envoltura


@cronometrar
def sumar(a: int, b: int) -> int:
    return a + b


sumar(1, 2)          # ✔ El comprobador conoce la firma (a: int, b: int) -> int
sumar("1", 2)        # ✖ Error
```

### 14.3 `**kwargs` tipados con `Unpack`

```python
from typing import TypedDict, Unpack


class OpcionesConexion(TypedDict, total=False):
    timeout: float
    reintentos: int
    verificar_ssl: bool


def conectar(url: str, **opciones: Unpack[OpcionesConexion]) -> None: ...


conectar("https://api", timeout=5, reintentos=3)     # ✔
conectar("https://api", tiempo=5)                    # ✖ Argumento desconocido
```

### 14.4 Marcar código obsoleto (3.13+)

```python
from warnings import deprecated


@deprecated("Use calcular_total() en su lugar")
def total_antiguo(lineas: list[float]) -> float:
    return sum(lineas)
```

---

## 15. `Any`, `object`, `Never` y `cast`

| Tipo | Significado | Uso |
|---|---|---|
| `Any` | Desactiva la comprobación: compatible con todo. | Último recurso, código dinámico o sin tipar. |
| `object` | Cualquier valor, pero solo admite operaciones comunes. | Cuando acepta cualquier cosa y no la manipula. |
| `Never` / `NoReturn` | La función nunca devuelve (lanza o sale). | Funciones de error, `assert_never`. |
| `cast(T, x)` | Le dice al comprobador «confíe en mí». No hace nada al ejecutar. | Casos donde usted sabe más que el comprobador. |

```python
from typing import Any, Never, cast


def registrar(valor: object) -> None:      # Mejor que Any: impide usos no seguros
    print(repr(valor))


def fallar(mensaje: str) -> Never:
    raise RuntimeError(mensaje)


datos = cast(dict[str, int], json.loads(texto))    # Sin validación real: considere Pydantic
```

> Prefiera `object` a `Any` cuando sea posible. Cada `Any` es un agujero en la comprobación.

---

## 16. Anotaciones en tiempo de ejecución

```python
import typing
from dataclasses import dataclass


@dataclass
class Producto:
    nombre: str
    precio: float
    etiquetas: list[str]


print(typing.get_type_hints(Producto))
# {'nombre': <class 'str'>, 'precio': <class 'float'>, 'etiquetas': list[str]}

tipo = typing.get_type_hints(Producto)["etiquetas"]
print(typing.get_origin(tipo), typing.get_args(tipo))    # <class 'list'> (<class 'str'>,)
```

Desde **Python 3.14**, las anotaciones se evalúan **de forma diferida** (PEP 649): se pueden usar nombres definidos más adelante sin comillas, y el módulo `annotationlib` permite obtenerlas en distintos formatos:

```python
from annotationlib import Format, get_annotations


class Nodo:
    siguiente: Nodo | None        # Sin comillas en 3.14+

print(get_annotations(Nodo, format=Format.STRING))    # {'siguiente': 'Nodo | None'}
```

En versiones anteriores, use comillas (`"Nodo | None"`) o `from __future__ import annotations` al inicio del módulo.

---

## 17. Importaciones circulares y `TYPE_CHECKING`

Cuando un tipo solo se necesita para las anotaciones, impórtelo dentro de `if TYPE_CHECKING:`; así no se importa al ejecutar:

```python
from __future__ import annotations          # Innecesario en 3.14+

from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from .pedidos import Pedido             # Solo lo ve el comprobador


class Cliente:
    def __init__(self) -> None:
        self.pedidos: list[Pedido] = []
```

> Útil también para evitar importar módulos pesados (pandas, numpy) solo por las anotaciones.

---

## 18. Archivos de stubs y paquetes tipados

### 18.1 Librerías sin tipos

Muchas librerías tienen tipos en paquetes `types-*` (proyecto **typeshed**):

```bash
uv add --dev types-requests types-PyYAML types-python-dateutil
```

### 18.2 Escribir un stub (`.pyi`)

```python
# legado.pyi — firmas para un módulo sin anotaciones
def calcular(base: float, coeficiente: float = ...) -> float: ...

class Conexion:
    def __init__(self, url: str) -> None: ...
    def consultar(self, sql: str) -> list[tuple[object, ...]]: ...
```

Generar stubs automáticamente como punto de partida:

```bash
uvx --from mypy stubgen -m legado -o stubs/
```

### 18.3 Publicar un paquete tipado

Añada un archivo vacío **`py.typed`** dentro del paquete (PEP 561):

```
src/mi_paquete/
├── __init__.py
└── py.typed
```

Sin `py.typed`, los comprobadores ignoran las anotaciones del paquete instalado.

---

## 19. Ejemplo práctico completo

**Sistema de pedidos** tipado de principio a fin: modelos inmutables, identificadores con `NewType`, repositorio genérico basado en protocolos, resultados explícitos, exhaustividad con `match` y decorador que conserva firmas. Solo usa la librería estándar.

```python
"""
pedidos.py — Ejemplo de código tipado (Python 3.12+).
Compruébelo con: mypy --strict pedidos.py  /  pyright pedidos.py  /  ty check pedidos.py
"""
from __future__ import annotations

import functools
import logging
from collections.abc import Callable, Iterable, Iterator
from dataclasses import dataclass, field, replace
from datetime import datetime, UTC
from decimal import Decimal
from enum import StrEnum
from typing import Literal, NewType, Protocol, Self, assert_never

log = logging.getLogger(__name__)

# ------------------------------------------------------------------ Tipos del dominio
PedidoId = NewType("PedidoId", int)
ClienteId = NewType("ClienteId", int)
Moneda = Literal["EUR", "USD"]


class Estado(StrEnum):
    BORRADOR = "borrador"
    CONFIRMADO = "confirmado"
    ENVIADO = "enviado"
    CANCELADO = "cancelado"


@dataclass(frozen=True, slots=True)
class Linea:
    sku: str
    cantidad: int
    precio_unitario: Decimal

    @property
    def subtotal(self) -> Decimal:
        return self.precio_unitario * self.cantidad


@dataclass(frozen=True, slots=True)
class Pedido:
    id: PedidoId
    cliente: ClienteId
    moneda: Moneda = "EUR"
    estado: Estado = Estado.BORRADOR
    lineas: tuple[Linea, ...] = ()
    creado: datetime = field(default_factory=lambda: datetime.now(UTC))

    @property
    def total(self) -> Decimal:
        return sum((l.subtotal for l in self.lineas), start=Decimal("0"))

    def con_linea(self, linea: Linea) -> Self:
        return replace(self, lineas=(*self.lineas, linea))

    def con_estado(self, estado: Estado) -> Self:
        return replace(self, estado=estado)


# ------------------------------------------------------------------ Resultados explícitos
@dataclass(frozen=True, slots=True)
class Ok[T]:
    valor: T


@dataclass(frozen=True, slots=True)
class Error:
    motivo: str


type Resultado[T] = Ok[T] | Error


# ------------------------------------------------------------------ Repositorio genérico
class ConId(Protocol):
    @property
    def id(self) -> int: ...


class Repositorio[T: ConId](Protocol):
    def obtener(self, id_: int) -> T | None: ...
    def guardar(self, entidad: T) -> None: ...
    def todos(self) -> Iterator[T]: ...


class RepositorioMemoria[T: ConId]:
    def __init__(self, iniciales: Iterable[T] = ()) -> None:
        self._datos: dict[int, T] = {e.id: e for e in iniciales}

    def obtener(self, id_: int) -> T | None:
        return self._datos.get(id_)

    def guardar(self, entidad: T) -> None:
        self._datos[entidad.id] = entidad

    def todos(self) -> Iterator[T]:
        yield from self._datos.values()


# ------------------------------------------------------------------ Decorador tipado
def auditado[**P, R](funcion: Callable[P, R]) -> Callable[P, R]:
    @functools.wraps(funcion)
    def envoltura(*args: P.args, **kwargs: P.kwargs) -> R:
        resultado = funcion(*args, **kwargs)
        log.info("%s → %r", funcion.__name__, resultado)
        return resultado
    return envoltura


# ------------------------------------------------------------------ Servicio
TRANSICIONES: dict[Estado, frozenset[Estado]] = {
    Estado.BORRADOR: frozenset({Estado.CONFIRMADO, Estado.CANCELADO}),
    Estado.CONFIRMADO: frozenset({Estado.ENVIADO, Estado.CANCELADO}),
    Estado.ENVIADO: frozenset(),
    Estado.CANCELADO: frozenset(),
}


class ServicioPedidos:
    def __init__(self, repo: Repositorio[Pedido]) -> None:
        self._repo = repo

    @auditado
    def anadir_linea(self, id_: PedidoId, linea: Linea) -> Resultado[Pedido]:
        pedido = self._repo.obtener(id_)
        if pedido is None:
            return Error(f"Pedido {id_} no encontrado")
        if pedido.estado is not Estado.BORRADOR:
            return Error("Solo se pueden modificar pedidos en borrador")
        if linea.cantidad <= 0:
            return Error("La cantidad debe ser positiva")
        actualizado = pedido.con_linea(linea)
        self._repo.guardar(actualizado)
        return Ok(actualizado)

    @auditado
    def cambiar_estado(self, id_: PedidoId, nuevo: Estado) -> Resultado[Pedido]:
        pedido = self._repo.obtener(id_)
        if pedido is None:
            return Error(f"Pedido {id_} no encontrado")
        if nuevo not in TRANSICIONES[pedido.estado]:
            return Error(f"Transición no permitida: {pedido.estado} → {nuevo}")
        if nuevo is Estado.CONFIRMADO and not pedido.lineas:
            return Error("No se puede confirmar un pedido vacío")
        actualizado = pedido.con_estado(nuevo)
        self._repo.guardar(actualizado)
        return Ok(actualizado)

    def pendientes_de(self, cliente: ClienteId) -> list[Pedido]:
        return [p for p in self._repo.todos()
                if p.cliente == cliente and p.estado in (Estado.BORRADOR, Estado.CONFIRMADO)]


def describir(estado: Estado) -> str:
    match estado:
        case Estado.BORRADOR:
            return "En edición"
        case Estado.CONFIRMADO:
            return "Pendiente de envío"
        case Estado.ENVIADO:
            return "Enviado"
        case Estado.CANCELADO:
            return "Cancelado"
        case _:
            assert_never(estado)


def mostrar(resultado: Resultado[Pedido]) -> str:
    match resultado:
        case Ok(valor=pedido):
            return f"Pedido {pedido.id}: {describir(pedido.estado)} · {pedido.total} {pedido.moneda}"
        case Error(motivo=motivo):
            return f"Error: {motivo}"


# ------------------------------------------------------------------ Uso
if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")
    repo = RepositorioMemoria[Pedido]([Pedido(PedidoId(1), ClienteId(10))])
    servicio = ServicioPedidos(repo)

    print(mostrar(servicio.anadir_linea(PedidoId(1), Linea("TEC-01", 2, Decimal("59.90")))))
    print(mostrar(servicio.anadir_linea(PedidoId(1), Linea("RAT-02", 1, Decimal("24.50")))))
    print(mostrar(servicio.cambiar_estado(PedidoId(1), Estado.CONFIRMADO)))
    print(mostrar(servicio.cambiar_estado(PedidoId(1), Estado.BORRADOR)))
    print(mostrar(servicio.anadir_linea(PedidoId(99), Linea("X", 1, Decimal(1)))))
    print("Pendientes:", [p.id for p in servicio.pendientes_de(ClienteId(10))])

    # Errores que detectaría el comprobador de tipos (descomente para verlo):
    # servicio.anadir_linea(ClienteId(10), Linea("X", 1, Decimal(1)))   # ClienteId ≠ PedidoId
    # Pedido(PedidoId(2), ClienteId(1), moneda="GBP")                    # "GBP" no es Moneda
    # servicio.cambiar_estado(PedidoId(1), "enviado")                    # str ≠ Estado
```

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: 'type' object is not subscriptable` | `list[int]` en Python < 3.9. | Actualice Python o use `from __future__ import annotations`. |
| `NameError` en una anotación | Nombre definido después (Python < 3.14). | Use comillas o `from __future__ import annotations`. |
| `Incompatible default for argument` (`x: int = None`) | `None` no es `int`. | `x: int | None = None`. |
| `List[int]` / `Optional` marcados como obsoletos por el linter | Sintaxis antigua de `typing`. | Use `list[int]` y `int | None` (Ruff `UP` lo corrige). |
| Mutar una lista recibida como `Sequence` da error | `Sequence` es de solo lectura. | Use `list[T]` si necesita modificarla, o devuelva una copia. |
| `list[Perro]` no es aceptado como `list[Animal]` | `list` es invariante. | Use `Sequence[Animal]` (covariante) en el parámetro. |
| El comprobador ignora los tipos de su paquete instalado | Falta `py.typed`. | Añada el archivo vacío `py.typed`. |
| Los tipos de una librería aparecen como `Any` | La librería no está tipada. | Instale `types-<paquete>` o escriba un stub. |
| `cast()` no convierte el valor | `cast` solo afecta al comprobador. | Convierta o valide de verdad (`int(x)`, Pydantic). |
| Importación circular al añadir anotaciones | Dos módulos se importan mutuamente. | `if TYPE_CHECKING:` para las importaciones de tipos. |

---

## 21. Buenas prácticas

- Anote **todas las funciones públicas**: parámetros y retorno (incluido `-> None`).
- Use la sintaxis moderna: `list[int]`, `X | None`, `class C[T]`, `type Alias = ...`.
- Acepte tipos abstractos (`Iterable`, `Sequence`, `Mapping`) y devuelva tipos concretos.
- Prefiera `object` a `Any`, y `TypeIs` a `cast`.
- Modele los estados con `Literal` o `Enum` y compruebe la exhaustividad con `assert_never`.
- Use `NewType` para identificadores que no deben mezclarse.
- Use `Protocol` para desacoplar dependencias y facilitar las pruebas.
- Las anotaciones no validan: para datos externos (JSON, formularios, variables de entorno), use **Pydantic**.
- Ejecute un comprobador de tipos (**mypy**, **Pyright** o **ty**) en el editor y en la CI.
- Añada `py.typed` si publica un paquete.

---

## 22. Referencia rápida

```python
x: int · s: str | None · xs: list[int] · d: dict[str, float] · t: tuple[int, ...]
def f(a: int, b: str = "") -> bool: ...
from collections.abc import Iterable, Sequence, Mapping, Callable, Iterator
Callable[[int, str], bool] · Literal["a", "b"] · Final · ClassVar[int] · Self
def f[T](x: list[T]) -> T · class C[T: Base] · type Alias[T] = list[T]
class P(Protocol): def m(self) -> int: ...
NewType("Id", int) · Annotated[int, meta] · TypedDict / NotRequired / ReadOnly
TypeIs[T] · assert_never(x) · @overload · @override · @deprecated
def deco[**P, R](f: Callable[P, R]) -> Callable[P, R]
**kwargs: Unpack[TD] · cast(T, x) · Never · if TYPE_CHECKING:
typing.get_type_hints(obj) · py.typed
```

---

## 23. Ejercicios propuestos

1. Anote por completo un módulo propio sin tipos y corrija los errores que señale `mypy --strict`.
2. Cree una clase genérica `Cache[K, V]` con caducidad por tiempo y compruebe que el comprobador detecta tipos incorrectos.
3. Defina un `Protocol` `Almacenamiento` con implementaciones en memoria y en disco, y una función que acepte cualquiera de ellas.
4. Escriba un decorador `reintentar` con `ParamSpec` que conserve la firma de la función decorada.
5. Modele los estados de un ticket de soporte con `StrEnum` y una función con `match` exhaustiva; añada un estado nuevo y observe el error del comprobador.

---

## 24. Referencias

- Documentación de `typing`: https://docs.python.org/es/3/library/typing.html
- Especificación del sistema de tipos: https://typing.python.org/en/latest/spec/
- Guías de tipado: https://typing.python.org/en/latest/guides/
- PEP 484 (anotaciones de tipo): https://peps.python.org/pep-0484/
- PEP 695 (sintaxis de genéricos): https://peps.python.org/pep-0695/
- PEP 649 (evaluación diferida): https://peps.python.org/pep-0649/
- typeshed: https://github.com/python/typeshed
