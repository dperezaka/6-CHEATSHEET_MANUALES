# Manual de uso: Programación orientada a objetos en Python

> **Categoría:** Programación · **Utilidad:** Modelar entidades del dominio con clases y objetos
> **Tipo:** Característica del lenguaje (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Conceptos clave](#2-conceptos-clave)
3. [Clases y objetos](#3-clases-y-objetos)
4. [Atributos y métodos](#4-atributos-y-métodos)
5. [Encapsulación y propiedades](#5-encapsulación-y-propiedades)
6. [Métodos especiales](#6-métodos-especiales)
7. [Herencia](#7-herencia)
8. [Polimorfismo y clases abstractas](#8-polimorfismo-y-clases-abstractas)
9. [Composición](#9-composición)
10. [Dataclasses](#10-dataclasses)
11. [Enumeraciones](#11-enumeraciones)
12. [Protocolos (tipado estructural)](#12-protocolos-tipado-estructural)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

La **programación orientada a objetos (POO)** organiza el código en torno a **objetos** que combinan datos (atributos) y comportamiento (métodos). En Python todo es un objeto: números, cadenas, funciones y módulos.

**Cuándo usar clases:**
- Para representar entidades con estado y comportamiento (`Cliente`, `Factura`, `CuentaBancaria`).
- Cuando varias funciones comparten los mismos datos.
- Para definir interfaces comunes con distintas implementaciones (exportar a CSV, a Excel, a PDF).

**Cuándo no son necesarias:**
- Para un cálculo aislado, basta una función.
- Para agrupar datos sin comportamiento, una `dataclass` o un `NamedTuple` es suficiente.

---

## 2. Conceptos clave

| Concepto | Descripción |
|---|---|
| Clase | Plantilla que define atributos y métodos. |
| Objeto (instancia) | Ejemplar concreto creado a partir de una clase. |
| Atributo | Dato asociado a un objeto (`cliente.nombre`) o a la clase. |
| Método | Función definida dentro de una clase. |
| `self` | Referencia al propio objeto; primer parámetro de los métodos de instancia. |
| Constructor | `__init__`: inicializa el objeto al crearlo. |
| Encapsulación | Ocultar detalles internos y exponer una interfaz controlada. |
| Herencia | Una clase hija reutiliza y amplía otra clase padre. |
| Polimorfismo | Objetos de clases distintas responden al mismo método. |
| Composición | Un objeto contiene a otros objetos como atributos. |

---

## 3. Clases y objetos

```python
class Cliente:
    def __init__(self, nombre: str, email: str) -> None:
        self.nombre = nombre
        self.email = email

    def saludar(self) -> str:
        return f"Hola, {self.nombre}"


ana = Cliente("Ana", "ana@ejemplo.com")       # Crear un objeto
ana.nombre                                     # 'Ana'
ana.saludar()                                  # 'Hola, Ana'
isinstance(ana, Cliente)                       # True
```

> Los nombres de clase se escriben en `PascalCase`: `Cliente`, `LineaFactura`, `GestorPedidos`.

---

## 4. Atributos y métodos

### 4.1 Atributos de instancia y de clase

```python
class Producto:
    iva = 0.21                       # Atributo de clase (compartido)
    total_creados = 0

    def __init__(self, nombre: str, precio: float) -> None:
        self.nombre = nombre         # Atributos de instancia (propios de cada objeto)
        self.precio = precio
        Producto.total_creados += 1

    def precio_con_iva(self) -> float:
        return round(self.precio * (1 + self.iva), 2)
```

> ⚠ No use listas o diccionarios como atributos de clase para datos de cada objeto: se compartirían entre todas las instancias.

### 4.2 Tipos de métodos

```python
class Fecha:
    def __init__(self, dia: int, mes: int, ano: int) -> None:
        self.dia, self.mes, self.ano = dia, mes, ano

    def formato(self) -> str:                          # Método de instancia
        return f"{self.dia:02d}/{self.mes:02d}/{self.ano}"

    @classmethod
    def desde_texto(cls, texto: str) -> "Fecha":       # Constructor alternativo
        dia, mes, ano = map(int, texto.split("/"))
        return cls(dia, mes, ano)

    @staticmethod
    def es_bisiesto(ano: int) -> bool:                 # No usa ni la instancia ni la clase
        return ano % 4 == 0 and (ano % 100 != 0 or ano % 400 == 0)


f = Fecha.desde_texto("26/11/2026")
Fecha.es_bisiesto(2028)                                # True
```

| Tipo | Decorador | Primer parámetro | Uso |
|---|---|---|---|
| De instancia | — | `self` | Trabajar con los datos del objeto. |
| De clase | `@classmethod` | `cls` | Constructores alternativos, datos de la clase. |
| Estático | `@staticmethod` | — | Utilidades relacionadas con la clase. |

---

## 5. Encapsulación y propiedades

Python no tiene atributos privados reales; se usan **convenciones**:

| Nombre | Significado |
|---|---|
| `nombre` | Público. |
| `_nombre` | Interno: no debe usarse fuera de la clase (convención). |
| `__nombre` | Se renombra internamente (*name mangling*) para evitar conflictos en subclases. |

### 5.1 Propiedades (`@property`)

Permiten validar o calcular un atributo manteniendo la sintaxis de acceso normal:

```python
class CuentaBancaria:
    def __init__(self, titular: str, saldo: float = 0.0) -> None:
        self.titular = titular
        self._saldo = saldo

    @property
    def saldo(self) -> float:                  # Lectura: cuenta.saldo
        return self._saldo

    def ingresar(self, importe: float) -> None:
        if importe <= 0:
            raise ValueError("El importe debe ser positivo")
        self._saldo += importe

    def retirar(self, importe: float) -> None:
        if importe > self._saldo:
            raise ValueError("Saldo insuficiente")
        self._saldo -= importe


cuenta = CuentaBancaria("Ana", 100)
cuenta.ingresar(50)
cuenta.saldo            # 150
cuenta.saldo = 1000     # ❌ AttributeError: la propiedad es de solo lectura
```

### 5.2 Propiedad con validación al asignar

```python
class Empleado:
    def __init__(self, nombre: str, salario: float) -> None:
        self.nombre = nombre
        self.salario = salario             # Usa el setter, también al crear el objeto

    @property
    def salario(self) -> float:
        return self._salario

    @salario.setter
    def salario(self, valor: float) -> None:
        if valor < 0:
            raise ValueError("El salario no puede ser negativo")
        self._salario = valor
```

---

## 6. Métodos especiales

Los métodos con doble guion bajo (*dunder*) integran sus clases con la sintaxis de Python.

| Método | Se invoca con | Uso |
|---|---|---|
| `__init__` | `Clase(...)` | Inicializar. |
| `__repr__` | `repr(obj)`, consola | Representación técnica (para depurar). |
| `__str__` | `str(obj)`, `print(obj)` | Representación legible. |
| `__eq__` | `a == b` | Igualdad. |
| `__lt__`, `__le__`… | `a < b` | Ordenación. |
| `__hash__` | `hash(obj)` | Uso como clave de diccionario. |
| `__len__` | `len(obj)` | Longitud. |
| `__getitem__` | `obj[i]` | Acceso por índice o clave. |
| `__iter__` | `for x in obj` | Iteración. |
| `__contains__` | `x in obj` | Pertenencia. |
| `__add__`, `__mul__`… | `a + b`, `a * b` | Operadores aritméticos. |
| `__bool__` | `if obj:` | Valor de verdad. |
| `__enter__` / `__exit__` | `with obj:` | Gestor de contexto. |

```python
class Dinero:
    def __init__(self, cantidad: float, moneda: str = "EUR") -> None:
        self.cantidad = round(cantidad, 2)
        self.moneda = moneda

    def __repr__(self) -> str:
        return f"Dinero({self.cantidad!r}, {self.moneda!r})"

    def __str__(self) -> str:
        return f"{self.cantidad:.2f} {self.moneda}"

    def __add__(self, otro: "Dinero") -> "Dinero":
        if self.moneda != otro.moneda:
            raise ValueError("Monedas distintas")
        return Dinero(self.cantidad + otro.cantidad, self.moneda)

    def __eq__(self, otro: object) -> bool:
        return isinstance(otro, Dinero) and (self.cantidad, self.moneda) == (otro.cantidad, otro.moneda)


print(Dinero(10) + Dinero(5.5))       # 15.50 EUR
```

### 6.1 Clase iterable tipo colección

```python
class Carrito:
    def __init__(self) -> None:
        self._lineas: list[tuple[str, int]] = []

    def anadir(self, producto: str, cantidad: int = 1) -> None:
        self._lineas.append((producto, cantidad))

    def __len__(self) -> int:
        return len(self._lineas)

    def __iter__(self):
        return iter(self._lineas)

    def __bool__(self) -> bool:
        return bool(self._lineas)


carrito = Carrito()
carrito.anadir("Teclado", 2)
if carrito:
    for producto, cantidad in carrito:
        print(producto, cantidad)
```

---

## 7. Herencia

```python
class Empleado:
    def __init__(self, nombre: str, salario_base: float) -> None:
        self.nombre = nombre
        self.salario_base = salario_base

    def salario(self) -> float:
        return self.salario_base

    def __str__(self) -> str:
        return f"{self.nombre} ({type(self).__name__}): {self.salario():,.2f} €"


class Comercial(Empleado):
    def __init__(self, nombre: str, salario_base: float, ventas: float, comision: float = 0.05) -> None:
        super().__init__(nombre, salario_base)        # Llama al constructor del padre
        self.ventas = ventas
        self.comision = comision

    def salario(self) -> float:                       # Sobrescribe el método
        return super().salario() + self.ventas * self.comision


class Directivo(Empleado):
    def salario(self) -> float:
        return self.salario_base * 1.2


plantilla = [Empleado("Luis", 2000), Comercial("Ana", 1800, 30000), Directivo("Eva", 4000)]
for e in plantilla:
    print(e)
```

```python
issubclass(Comercial, Empleado)       # True
isinstance(Comercial(...), Empleado)  # True
Comercial.__mro__                     # Orden de resolución de métodos
```

> Use la herencia solo cuando exista una relación **«es un»** real (un `Comercial` *es un* `Empleado`). Evite jerarquías de más de dos o tres niveles.

### 7.1 Herencia múltiple y *mixins*

```python
class SerializableJSONMixin:
    def a_json(self) -> str:
        import json
        return json.dumps(self.__dict__, ensure_ascii=False, default=str)


class Cliente(SerializableJSONMixin):
    def __init__(self, nombre: str) -> None:
        self.nombre = nombre


Cliente("Ana").a_json()               # '{"nombre": "Ana"}'
```

> Un *mixin* aporta una capacidad concreta y no se instancia por sí solo.

---

## 8. Polimorfismo y clases abstractas

### 8.1 Polimorfismo

Objetos distintos se usan del mismo modo si ofrecen los mismos métodos:

```python
total_nominas = sum(e.salario() for e in plantilla)    # Cada clase calcula a su manera
```

### 8.2 Clases abstractas (`abc`)

Definen una interfaz obligatoria para las subclases:

```python
from abc import ABC, abstractmethod


class Exportador(ABC):
    @abstractmethod
    def exportar(self, datos: list[dict], ruta: str) -> None:
        """Escribe los datos en la ruta indicada."""

    def extension(self) -> str:                    # Método concreto compartido
        return ".dat"


class ExportadorCSV(Exportador):
    def exportar(self, datos, ruta):
        ...

    def extension(self) -> str:
        return ".csv"


Exportador()           # ❌ TypeError: no se puede instanciar una clase abstracta
ExportadorCSV()        # ✔
```

---

## 9. Composición

Construir objetos a partir de otros (**«tiene un»**) suele ser más flexible que heredar.

```python
class Motor:
    def __init__(self, potencia_cv: int) -> None:
        self.potencia_cv = potencia_cv

    def arrancar(self) -> str:
        return "Motor en marcha"


class Coche:
    def __init__(self, modelo: str, motor: Motor) -> None:
        self.modelo = modelo
        self.motor = motor                  # El coche TIENE un motor

    def arrancar(self) -> str:
        return f"{self.modelo}: {self.motor.arrancar()}"


Coche("Berlina", Motor(150)).arrancar()
```

> **Regla práctica:** prefiera la composición a la herencia. Permite cambiar componentes (inyectar un motor eléctrico, un repositorio de pruebas…) sin modificar la clase.

---

## 10. Dataclasses

Generan automáticamente `__init__`, `__repr__` y `__eq__` para clases centradas en datos.

```python
from dataclasses import dataclass, field, asdict
from datetime import date


@dataclass
class Producto:
    codigo: str
    nombre: str
    precio: float
    stock: int = 0
    etiquetas: list[str] = field(default_factory=list)    # Mutable: use default_factory
    alta: date = field(default_factory=date.today)

    def __post_init__(self) -> None:                       # Validación tras __init__
        if self.precio < 0:
            raise ValueError("Precio negativo")

    @property
    def valor_stock(self) -> float:
        return self.precio * self.stock


p = Producto("TEC-01", "Teclado", 59.9, 120)
print(p)            # Producto(codigo='TEC-01', nombre='Teclado', precio=59.9, ...)
asdict(p)           # Convierte a diccionario
```

### 10.1 Opciones útiles

```python
@dataclass(frozen=True)            # Inmutable y hashable (puede ser clave de dict)
class Coordenada:
    lat: float
    lon: float

@dataclass(order=True)             # Genera <, <=, >, >= comparando los campos en orden
class Version:
    mayor: int
    menor: int
    parche: int

@dataclass(slots=True, kw_only=True)   # Menos memoria · solo argumentos con nombre
class Configuracion:
    host: str
    puerto: int = 8080
```

| Opción | Efecto |
|---|---|
| `frozen=True` | Objetos inmutables. |
| `order=True` | Comparaciones de orden. |
| `slots=True` | Menor consumo de memoria y acceso más rápido. |
| `kw_only=True` | Obliga a usar argumentos con nombre. |

---

## 11. Enumeraciones

Para conjuntos cerrados de valores (estados, tipos, categorías):

```python
from enum import Enum, StrEnum, auto


class EstadoPedido(StrEnum):
    PENDIENTE = auto()             # 'pendiente'
    ENVIADO = auto()
    ENTREGADO = auto()
    CANCELADO = auto()


class Prioridad(Enum):
    BAJA = 1
    MEDIA = 2
    ALTA = 3


estado = EstadoPedido.ENVIADO
estado == "enviado"                # True (StrEnum se compara con str)
EstadoPedido("entregado")          # EstadoPedido.ENTREGADO
Prioridad.ALTA.value               # 3
[e.name for e in Prioridad]        # ['BAJA', 'MEDIA', 'ALTA']

match estado:
    case EstadoPedido.ENVIADO:
        ...
```

> Use enumeraciones en lugar de cadenas sueltas (`"enviado"`): el editor autocompleta y los errores tipográficos se detectan.

---

## 12. Protocolos (tipado estructural)

Un `Protocol` define una interfaz **sin herencia**: cualquier clase que tenga los métodos requeridos la cumple (*duck typing* verificable con `mypy`).

```python
from typing import Protocol


class Notificador(Protocol):
    def enviar(self, destinatario: str, mensaje: str) -> None: ...


class NotificadorEmail:
    def enviar(self, destinatario: str, mensaje: str) -> None:
        print(f"Email a {destinatario}: {mensaje}")


class NotificadorConsola:
    def enviar(self, destinatario: str, mensaje: str) -> None:
        print(f"[{destinatario}] {mensaje}")


def avisar_vencimiento(notificador: Notificador, cliente: str) -> None:
    notificador.enviar(cliente, "Su factura vence mañana")


avisar_vencimiento(NotificadorEmail(), "ana@ejemplo.com")
```

| Mecanismo | Cuándo usarlo |
|---|---|
| Herencia | Relación «es un» y código compartido. |
| `ABC` | Interfaz obligatoria comprobada al instanciar. |
| `Protocol` | Interfaz flexible, sin acoplar las clases entre sí. |

---

## 13. Ejemplo práctico completo

**Sistema de pedidos de una tienda**: modelos con `dataclass` y `Enum`, reglas de negocio encapsuladas, descuentos intercambiables mediante polimorfismo, máquina de estados del pedido y notificaciones mediante un `Protocol`.

```python
"""
Gestión de pedidos con programación orientada a objetos.
Solo usa la librería estándar.
"""
from __future__ import annotations

from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime
from decimal import Decimal, ROUND_HALF_UP
from enum import StrEnum, auto
from itertools import count
from typing import Protocol

CENTIMO = Decimal("0.01")


def redondear(valor: Decimal) -> Decimal:
    return valor.quantize(CENTIMO, rounding=ROUND_HALF_UP)


# ------------------------------------------------------------------ Modelos
@dataclass(frozen=True)
class Producto:
    codigo: str
    nombre: str
    precio: Decimal
    iva: Decimal = Decimal("0.21")


@dataclass
class Linea:
    producto: Producto
    cantidad: int

    def __post_init__(self) -> None:
        if self.cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")

    @property
    def base(self) -> Decimal:
        return self.producto.precio * self.cantidad


@dataclass(frozen=True)
class Cliente:
    nombre: str
    email: str
    vip: bool = False


class Estado(StrEnum):
    BORRADOR = auto()
    CONFIRMADO = auto()
    ENVIADO = auto()
    ENTREGADO = auto()
    CANCELADO = auto()


TRANSICIONES: dict[Estado, set[Estado]] = {
    Estado.BORRADOR: {Estado.CONFIRMADO, Estado.CANCELADO},
    Estado.CONFIRMADO: {Estado.ENVIADO, Estado.CANCELADO},
    Estado.ENVIADO: {Estado.ENTREGADO},
    Estado.ENTREGADO: set(),
    Estado.CANCELADO: set(),
}


# ------------------------------------------------------------------ Descuentos (polimorfismo)
class Descuento(ABC):
    @abstractmethod
    def calcular(self, pedido: Pedido) -> Decimal: ...

    @abstractmethod
    def descripcion(self) -> str: ...


class SinDescuento(Descuento):
    def calcular(self, pedido: Pedido) -> Decimal:
        return Decimal("0")

    def descripcion(self) -> str:
        return "Sin descuento"


@dataclass
class DescuentoPorcentaje(Descuento):
    porcentaje: Decimal

    def calcular(self, pedido: Pedido) -> Decimal:
        return pedido.base * self.porcentaje / 100

    def descripcion(self) -> str:
        return f"Descuento {self.porcentaje} %"


@dataclass
class DescuentoVolumen(Descuento):
    umbral: Decimal
    importe: Decimal

    def calcular(self, pedido: Pedido) -> Decimal:
        return self.importe if pedido.base >= self.umbral else Decimal("0")

    def descripcion(self) -> str:
        return f"{self.importe} € a partir de {self.umbral} €"


# ------------------------------------------------------------------ Notificaciones (Protocol)
class Notificador(Protocol):
    def notificar(self, cliente: Cliente, mensaje: str) -> None: ...


class NotificadorConsola:
    def notificar(self, cliente: Cliente, mensaje: str) -> None:
        print(f"  ✉ Para {cliente.email}: {mensaje}")


# ------------------------------------------------------------------ Pedido
class TransicionInvalida(Exception):
    pass


class Pedido:
    _secuencia = count(1)

    def __init__(self, cliente: Cliente, notificador: Notificador, descuento: Descuento | None = None) -> None:
        self.numero = f"P-{next(Pedido._secuencia):05d}"
        self.cliente = cliente
        self.descuento = descuento or (DescuentoPorcentaje(Decimal(10)) if cliente.vip else SinDescuento())
        self._notificador = notificador
        self._lineas: list[Linea] = []
        self._estado = Estado.BORRADOR
        self.historial: list[tuple[datetime, Estado]] = [(datetime.now(), self._estado)]

    # ---------- Consulta
    @property
    def estado(self) -> Estado:
        return self._estado

    @property
    def base(self) -> Decimal:
        return sum((l.base for l in self._lineas), Decimal("0"))

    @property
    def importe_descuento(self) -> Decimal:
        return redondear(min(self.descuento.calcular(self), self.base))

    @property
    def iva(self) -> Decimal:
        if not self.base:
            return Decimal("0")
        factor = (self.base - self.importe_descuento) / self.base
        return redondear(sum((l.base * l.producto.iva * factor for l in self._lineas), Decimal("0")))

    @property
    def total(self) -> Decimal:
        return redondear(self.base - self.importe_descuento + self.iva)

    def __len__(self) -> int:
        return len(self._lineas)

    def __repr__(self) -> str:
        return f"Pedido({self.numero!r}, cliente={self.cliente.nombre!r}, estado={self._estado}, total={self.total})"

    # ---------- Modificación
    def anadir(self, producto: Producto, cantidad: int = 1) -> None:
        if self._estado is not Estado.BORRADOR:
            raise TransicionInvalida("Solo se pueden añadir productos a un pedido en borrador")
        for linea in self._lineas:
            if linea.producto == producto:
                linea.cantidad += cantidad
                return
        self._lineas.append(Linea(producto, cantidad))

    def _cambiar_estado(self, nuevo: Estado) -> None:
        if nuevo not in TRANSICIONES[self._estado]:
            raise TransicionInvalida(f"No se puede pasar de {self._estado} a {nuevo}")
        self._estado = nuevo
        self.historial.append((datetime.now(), nuevo))
        self._notificador.notificar(self.cliente, f"Su pedido {self.numero} está {nuevo}.")

    def confirmar(self) -> None:
        if not self._lineas:
            raise TransicionInvalida("No se puede confirmar un pedido vacío")
        self._cambiar_estado(Estado.CONFIRMADO)

    def enviar(self) -> None:
        self._cambiar_estado(Estado.ENVIADO)

    def entregar(self) -> None:
        self._cambiar_estado(Estado.ENTREGADO)

    def cancelar(self) -> None:
        self._cambiar_estado(Estado.CANCELADO)

    # ---------- Presentación
    def ticket(self) -> str:
        filas = [f"Pedido {self.numero} · {self.cliente.nombre} · {self._estado.upper()}", "-" * 50]
        for l in self._lineas:
            filas.append(f"{l.producto.nombre:<24}{l.cantidad:>4} x {l.producto.precio:>8} = {redondear(l.base):>9}")
        filas += ["-" * 50,
                  f"{'Base imponible':<40}{redondear(self.base):>10}",
                  f"{self.descuento.descripcion():<40}{-self.importe_descuento:>10}",
                  f"{'IVA':<40}{self.iva:>10}",
                  f"{'TOTAL (€)':<40}{self.total:>10}"]
        return "\n".join(filas)


# ------------------------------------------------------------------ Uso
if __name__ == "__main__":
    teclado = Producto("TEC-01", "Teclado mecánico", Decimal("59.90"))
    monitor = Producto("MON-27", "Monitor 27\"", Decimal("249.00"))
    libro = Producto("LIB-PY", "Libro de Python", Decimal("35.00"), iva=Decimal("0.04"))

    notificador = NotificadorConsola()
    ana = Cliente("Ana Ruiz", "ana@ejemplo.com", vip=True)
    luis = Cliente("Luis Gil", "luis@ejemplo.com")

    p1 = Pedido(ana, notificador)
    p1.anadir(monitor, 2)
    p1.anadir(teclado)
    p1.anadir(libro)
    p1.anadir(teclado)                     # Se acumula en la línea existente
    p1.confirmar()
    p1.enviar()
    print(p1.ticket(), "\n")

    p2 = Pedido(luis, notificador, DescuentoVolumen(umbral=Decimal(100), importe=Decimal(15)))
    p2.anadir(teclado, 2)
    print(p2.ticket(), "\n")

    try:
        p1.cancelar()                       # Un pedido enviado no se puede cancelar
    except TransicionInvalida as e:
        print(f"⚠ {e}")

    print(repr(p1))
```

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TypeError: __init__() takes 1 positional argument but 3 were given` | Falta `self` en `__init__` o el nombre está mal escrito (`__int__`). | `def __init__(self, ...)`. |
| `NameError: name 'nombre' is not defined` dentro de un método | Se usa `nombre` en lugar de `self.nombre`. | Acceda a los atributos mediante `self`. |
| `AttributeError: 'X' object has no attribute 'y'` | Atributo no inicializado en `__init__` o mal escrito. | Inicialice todos los atributos en `__init__`. |
| Todos los objetos comparten la misma lista | Lista declarada como atributo de clase. | Créela en `__init__` (o `field(default_factory=list)`). |
| `ValueError: mutable default <class 'list'> for field` | Lista como valor por defecto en una `dataclass`. | `field(default_factory=list)`. |
| `RecursionError` en una propiedad | El *getter* o *setter* usa el mismo nombre (`self.saldo`). | Guarde el valor en `self._saldo`. |
| El constructor del padre no se ejecuta | Falta `super().__init__(...)` en la subclase. | Llámelo al principio de `__init__`. |
| `TypeError: Can't instantiate abstract class` | Falta implementar un `@abstractmethod`. | Implemente todos los métodos abstractos. |
| `TypeError: unhashable type` al usar un objeto como clave | Se definió `__eq__` sin `__hash__`. | Use `@dataclass(frozen=True)` o defina `__hash__`. |
| `print(obj)` muestra `<__main__.X object at 0x…>` | No se definió `__repr__` ni `__str__`. | Defínalos o use una `dataclass`. |

---

## 15. Buenas prácticas

- Diseñe clases pequeñas con **una única responsabilidad**.
- Use `dataclass` para clases centradas en datos; reserve las clases manuales para lógica más compleja.
- Inicialice **todos** los atributos en `__init__`.
- Proteja el estado interno con `_atributo` y exponga propiedades o métodos con validación.
- Implemente `__repr__` en todas las clases: facilita la depuración.
- Prefiera la **composición** a la herencia; herede solo si existe una relación «es un» clara.
- Use `Enum`/`StrEnum` para conjuntos cerrados de valores.
- Defina interfaces con `ABC` o `Protocol` para poder sustituir implementaciones (por ejemplo, en pruebas).
- Lance excepciones propias con nombres descriptivos para errores del dominio.
- No cree clases solo para agrupar funciones sin estado: use un módulo.

---

## 16. Referencia rápida

```python
class C(Padre):
    atributo_clase = 0
    def __init__(self, x): super().__init__(); self.x = x; self._interno = None
    def metodo(self): ...
    @classmethod
    def crear(cls, ...): return cls(...)
    @staticmethod
    def util(...): ...
    @property
    def valor(self): return self._valor
    @valor.setter
    def valor(self, v): self._valor = v
    def __repr__(self): ...  __str__  __eq__  __lt__  __len__  __iter__  __getitem__  __add__

@dataclass(frozen=True, order=True, slots=True)  class D: a: int; b: list = field(default_factory=list)
class E(StrEnum): A = auto()     /   class P(Protocol): def m(self) -> None: ...
class A(ABC): @abstractmethod def m(self): ...
isinstance(obj, C) / issubclass(C, Padre) / type(obj).__name__ / vars(obj)
```

---

## 17. Ejercicios propuestos

1. Cree una clase `CuentaBancaria` con historial de movimientos, ingresos, retiradas, transferencias entre cuentas y una excepción `SaldoInsuficiente`.
2. Modele una jerarquía de figuras geométricas (`Circulo`, `Rectangulo`, `Triangulo`) con una clase abstracta `Figura` que obligue a implementar `area()` y `perimetro()`.
3. Implemente una clase `Inventario` que admita `len()`, `in`, iteración y acceso por código (`inventario["TEC-01"]`).
4. Defina una `dataclass` inmutable `Intervalo(inicio, fin)` con métodos para comprobar solapamiento y calcular la intersección.
5. Amplíe el ejemplo completo con un descuento por cupón de un solo uso y un `NotificadorArchivo` que escriba las notificaciones en un archivo de registro.

---

## 18. Referencias

- Clases (tutorial oficial): https://docs.python.org/es/3/tutorial/classes.html
- `dataclasses`: https://docs.python.org/es/3/library/dataclasses.html
- `enum`: https://docs.python.org/es/3/library/enum.html
- `abc`: https://docs.python.org/es/3/library/abc.html
- Modelo de datos (métodos especiales): https://docs.python.org/es/3/reference/datamodel.html
- PEP 544 – Protocolos: https://peps.python.org/pep-0544/
