# Manual de uso: Pydantic

> **Categoría:** Validación de datos · **Utilidad:** Valida y convierte datos en tiempo de ejecución (APIs, JSON, configuración)
> **Tipo:** Paquete externo (requiere instalación) · Este manual cubre **Pydantic v2**

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Modelos básicos](#4-modelos-básicos)
5. [Campos y restricciones](#5-campos-y-restricciones)
6. [Tipos especiales](#6-tipos-especiales)
7. [Validadores personalizados](#7-validadores-personalizados)
8. [Serialización](#8-serialización)
9. [Alias y nombres de campo](#9-alias-y-nombres-de-campo)
10. [Configuración del modelo](#10-configuración-del-modelo)
11. [Errores de validación](#11-errores-de-validación)
12. [Modelos anidados, genéricos y uniones discriminadas](#12-modelos-anidados-genéricos-y-uniones-discriminadas)
13. [`TypeAdapter`, `dataclasses` y `validate_call`](#13-typeadapter-dataclasses-y-validate_call)
14. [JSON Schema](#14-json-schema)
15. [Integración con otras herramientas](#15-integración-con-otras-herramientas)
16. [Rendimiento](#16-rendimiento)
17. [Migrar desde Pydantic v1](#17-migrar-desde-pydantic-v1)
18. [Ejemplo práctico completo](#18-ejemplo-práctico-completo)
19. [Errores frecuentes](#19-errores-frecuentes)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Referencia rápida](#21-referencia-rápida)
22. [Ejercicios propuestos](#22-ejercicios-propuestos)
23. [Referencias](#23-referencias)

---

## 1. Descripción

**Pydantic** valida datos usando las **anotaciones de tipo** de Python. Se define un modelo como una clase con campos tipados y Pydantic se encarga de:

- **validar** que los datos cumplen los tipos y restricciones;
- **convertir** valores compatibles (`"42"` → `42`, `"2026-11-26"` → `date`);
- **serializar** a diccionarios y JSON;
- **generar JSON Schema** para documentar APIs.

```python
from pydantic import BaseModel

class Cliente(BaseModel):
    id: int
    nombre: str
    activo: bool = True

Cliente(id="7", nombre="Ana")          # Cliente(id=7, nombre='Ana', activo=True)
Cliente(id="siete", nombre="Ana")      # ValidationError: id debe ser un entero
```

**Ventajas:**
- Núcleo escrito en Rust (`pydantic-core`): muy rápido.
- Usa anotaciones estándar: compatible con mypy, Pyright y el IDE.
- Mensajes de error estructurados y detallados.
- Base de FastAPI, Typer, LangChain, SQLModel, pydantic-settings y muchas otras librerías.

**Limitaciones:**
- La validación tiene un coste: no la use en bucles críticos con datos ya validados.
- La conversión automática (modo *lax*) puede sorprender; use el modo estricto donde importe.

| Si necesita… | Opción |
|---|---|
| Validar datos externos (API, JSON, formularios, archivos) | **Pydantic** |
| Estructura interna sin validación | `dataclasses` |
| Configuración desde variables de entorno | pydantic-settings |
| Validar `DataFrame` | pandera |
| Máxima velocidad de (de)serialización sin validación rica | msgspec |

---

## 2. Instalación

```bash
uv add pydantic
uv add "pydantic[email]"          # Incluye email-validator para EmailStr
python -m pip install pydantic
```

Comprobar:

```python
import pydantic
print(pydantic.VERSION)
```

Paquetes complementarios:

| Paquete | Uso |
|---|---|
| `pydantic-settings` | Configuración desde variables de entorno y `.env`. |
| `pydantic-extra-types` | Tipos adicionales: teléfono, país, IBAN, color, coordenadas… |
| `email-validator` | Necesario para `EmailStr` y `NameEmail`. |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Modelo | Clase que hereda de `BaseModel` y define campos tipados. |
| Campo | Atributo anotado; puede tener valor por defecto y restricciones (`Field`). |
| Validación | Comprobar y convertir los datos de entrada al crear el modelo. |
| Modo *lax* | Por defecto: convierte tipos compatibles (`"1"` → `1`). |
| Modo estricto | No convierte: el tipo debe coincidir exactamente. |
| Validador | Función propia que valida o transforma un campo o el modelo completo. |
| Serialización | Convertir el modelo a `dict` o JSON (`model_dump`, `model_dump_json`). |
| Alias | Nombre alternativo de un campo en la entrada o salida (p. ej. `camelCase`). |
| `ValidationError` | Excepción con la lista de todos los errores encontrados. |

---

## 4. Modelos básicos

```python
from datetime import date, datetime
from decimal import Decimal

from pydantic import BaseModel


class Direccion(BaseModel):
    calle: str
    ciudad: str
    cp: str


class Cliente(BaseModel):
    id: int
    nombre: str
    email: str
    alta: date
    saldo: Decimal = Decimal("0")
    etiquetas: list[str] = []                  # Pydantic copia los valores por defecto: es seguro
    direccion: Direccion | None = None
    ultima_compra: datetime | None = None


# Crear desde argumentos
c = Cliente(id=1, nombre="Ana", email="ana@ejemplo.com", alta="2026-01-15")

# Crear desde un diccionario
c = Cliente.model_validate({"id": "1", "nombre": "Ana", "email": "ana@ejemplo.com",
                            "alta": "2026-01-15", "direccion": {"calle": "Mayor 1", "ciudad": "Madrid", "cp": "28013"}})

# Crear desde JSON (más rápido que json.loads + model_validate)
c = Cliente.model_validate_json('{"id": 1, "nombre": "Ana", "email": "a@b.es", "alta": "2026-01-15"}')

print(c.alta, type(c.alta))           # 2026-01-15 <class 'datetime.date'>
print(c.direccion)                    # None

# Copiar con cambios
c2 = c.model_copy(update={"saldo": Decimal("150.00")})

# Introspección
print(Cliente.model_fields.keys())
print(c.model_fields_set)             # Campos que se proporcionaron explícitamente
```

> Los atributos se validan **al crear** el modelo. Las asignaciones posteriores (`c.id = "x"`) no se validan salvo que active `validate_assignment=True`.

---

## 5. Campos y restricciones

### 5.1 `Field`

```python
from typing import Annotated

from pydantic import BaseModel, Field


class Producto(BaseModel):
    sku: str = Field(pattern=r"^[A-Z]{3}-\d{3}$", examples=["TEC-001"])
    nombre: str = Field(min_length=2, max_length=100)
    precio: float = Field(gt=0, le=100_000, description="Precio sin IVA en euros")
    stock: int = Field(default=0, ge=0)
    descuento: float = Field(default=0, ge=0, lt=1)
    etiquetas: list[str] = Field(default_factory=list, max_length=10)
    interno: str = Field(default="", exclude=True)            # No se serializa
    creado_por: str = Field(default="sistema", frozen=True)   # No se puede modificar
```

| Restricción | Tipos | Significado |
|---|---|---|
| `gt`, `ge`, `lt`, `le` | Números | Mayor que, mayor o igual, menor que, menor o igual. |
| `multiple_of` | Números | Múltiplo de. |
| `max_digits`, `decimal_places` | `Decimal` | Precisión. |
| `min_length`, `max_length` | Cadenas, listas | Longitud. |
| `pattern` | Cadenas | Expresión regular. |
| `default`, `default_factory` | Todos | Valor por defecto (fijo o calculado). |
| `alias`, `validation_alias`, `serialization_alias` | Todos | Nombres alternativos. |
| `description`, `examples`, `title` | Todos | Documentación y JSON Schema. |
| `exclude`, `frozen`, `repr` | Todos | Serialización, inmutabilidad, representación. |
| `strict` | Todos | Sin conversión de tipos para este campo. |

### 5.2 Tipos reutilizables con `Annotated`

```python
from typing import Annotated

from pydantic import Field, StringConstraints

Sku = Annotated[str, StringConstraints(pattern=r"^[A-Z]{3}-\d{3}$", to_upper=True, strip_whitespace=True)]
Precio = Annotated[Decimal, Field(gt=0, max_digits=10, decimal_places=2)]
Porcentaje = Annotated[float, Field(ge=0, le=100)]


class Linea(BaseModel):
    sku: Sku
    precio: Precio
    iva: Porcentaje = 21
```

---

## 6. Tipos especiales

```python
from datetime import date, datetime, timedelta
from enum import StrEnum
from ipaddress import IPv4Address
from pathlib import Path
from typing import Literal
from uuid import UUID

from pydantic import (AnyUrl, BaseModel, EmailStr, FilePath, HttpUrl, NonNegativeInt, PastDate,
                      PositiveInt, SecretStr, AwareDatetime, Json, ByteSize)


class Estado(StrEnum):
    ACTIVO = "activo"
    BAJA = "baja"


class Ejemplo(BaseModel):
    email: EmailStr                       # Requiere pydantic[email]
    web: HttpUrl                          # Valida y normaliza URLs http/https
    api: AnyUrl
    id_: UUID
    edad: PositiveInt                     # > 0
    reintentos: NonNegativeInt            # >= 0
    nacimiento: PastDate                  # Fecha en el pasado
    evento: AwareDatetime                 # Fecha-hora con zona horaria obligatoria
    duracion: timedelta                   # Admite segundos o "PT1H30M"
    contrasena: SecretStr                 # Se oculta en repr y logs: '**********'
    estado: Estado
    modo: Literal["prueba", "produccion"]
    ip: IPv4Address
    config: FilePath                      # Ruta que debe existir y ser un archivo
    salida: Path
    tamano: ByteSize                      # Admite "10MB", "1.5GiB"
    metadatos: Json[dict[str, int]]       # Cadena JSON que se analiza y valida
```

```python
p = Ejemplo(..., contrasena="secreta")
print(p.contrasena)                        # **********
print(p.contrasena.get_secret_value())     # secreta
```

Tipos adicionales con `pydantic-extra-types`:

```python
from pydantic_extra_types.phone_numbers import PhoneNumber
from pydantic_extra_types.country import CountryAlpha2
```

---

## 7. Validadores personalizados

### 7.1 Validador de campo

```python
from pydantic import BaseModel, ValidationInfo, field_validator


class Usuario(BaseModel):
    usuario: str
    nif: str
    email: str

    @field_validator("usuario")
    @classmethod
    def sin_espacios(cls, v: str) -> str:
        if " " in v:
            raise ValueError("no puede contener espacios")
        return v.lower()

    @field_validator("nif", mode="before")           # Antes de la validación de tipo
    @classmethod
    def normalizar_nif(cls, v: object) -> object:
        return v.replace("-", "").upper() if isinstance(v, str) else v

    @field_validator("nif")
    @classmethod
    def letra_nif(cls, v: str) -> str:
        letras = "TRWAGMYFPDXBNJZSQVHLCKE"
        if len(v) != 9 or not v[:8].isdigit() or letras[int(v[:8]) % 23] != v[8]:
            raise ValueError("NIF no válido")
        return v

    @field_validator("email")
    @classmethod
    def dominio_corporativo(cls, v: str, info: ValidationInfo) -> str:
        dominio = (info.context or {}).get("dominio")      # Contexto opcional de validación
        if dominio and not v.endswith(f"@{dominio}"):
            raise ValueError(f"debe pertenecer al dominio {dominio}")
        return v


Usuario.model_validate({"usuario": "ana", "nif": "12345678-z", "email": "ana@empresa.com"},
                       context={"dominio": "empresa.com"})
```

| `mode` | Cuándo se ejecuta | Recibe |
|---|---|---|
| `"after"` (defecto) | Tras validar el tipo | Valor ya convertido |
| `"before"` | Antes de validar el tipo | Valor en bruto (cualquier tipo) |
| `"wrap"` | Envuelve la validación | Valor y la función de validación (`handler`) |
| `"plain"` | Sustituye la validación | Valor en bruto |

### 7.2 Validador de modelo (varios campos)

```python
from datetime import date
from typing import Self

from pydantic import BaseModel, model_validator


class Reserva(BaseModel):
    entrada: date
    salida: date
    personas: int

    @model_validator(mode="after")
    def fechas_coherentes(self) -> Self:
        if self.salida <= self.entrada:
            raise ValueError("la salida debe ser posterior a la entrada")
        return self
```

### 7.3 Validadores reutilizables con `Annotated`

```python
from typing import Annotated

from pydantic import AfterValidator, BeforeValidator


def a_mayusculas(v: str) -> str:
    return v.upper()


def coma_decimal(v: object) -> object:
    return v.replace(".", "").replace(",", ".") if isinstance(v, str) else v


CodigoPais = Annotated[str, AfterValidator(a_mayusculas)]
ImporteES = Annotated[float, BeforeValidator(coma_decimal)]


class Movimiento(BaseModel):
    pais: CodigoPais
    importe: ImporteES


Movimiento(pais="es", importe="1.234,50")      # pais='ES', importe=1234.5
```

### 7.4 Campos calculados

```python
from pydantic import BaseModel, computed_field


class Linea(BaseModel):
    cantidad: int
    precio: float

    @computed_field
    @property
    def subtotal(self) -> float:
        return round(self.cantidad * self.precio, 2)


Linea(cantidad=3, precio=9.99).model_dump()    # {'cantidad': 3, 'precio': 9.99, 'subtotal': 29.97}
```

---

## 8. Serialización

```python
c.model_dump()                                  # dict con tipos Python (date, Decimal…)
c.model_dump(mode="json")                       # dict con tipos JSON (str, float…)
c.model_dump_json(indent=2)                     # Cadena JSON
c.model_dump(include={"id", "nombre"})
c.model_dump(exclude={"saldo"})
c.model_dump(exclude={"direccion": {"cp"}})     # Excluir campos anidados
c.model_dump(exclude_none=True)                 # Omite los None
c.model_dump(exclude_unset=True)                # Solo lo que se proporcionó (útil en PATCH)
c.model_dump(exclude_defaults=True)
c.model_dump(by_alias=True)                     # Usa los alias como claves
```

Personalizar la serialización de un campo:

```python
from datetime import datetime
from decimal import Decimal

from pydantic import BaseModel, field_serializer, PlainSerializer
from typing import Annotated

EurosTexto = Annotated[Decimal, PlainSerializer(lambda v: f"{v:.2f} €", return_type=str, when_used="json")]


class Factura(BaseModel):
    numero: str
    emitida: datetime
    total: EurosTexto

    @field_serializer("emitida")
    def fecha_corta(self, v: datetime) -> str:
        return v.strftime("%d/%m/%Y")
```

---

## 9. Alias y nombres de campo

```python
from pydantic import AliasChoices, AliasPath, BaseModel, ConfigDict, Field
from pydantic.alias_generators import to_camel


class Pedido(BaseModel):
    model_config = ConfigDict(alias_generator=to_camel, populate_by_name=True)

    id_pedido: int                         # Alias automático: idPedido
    fecha_entrega: str                     # fechaEntrega
    importe_total: float = Field(validation_alias=AliasChoices("importeTotal", "total", "amount"))
    ciudad: str = Field(validation_alias=AliasPath("envio", "direccion", "ciudad"))


p = Pedido.model_validate({"idPedido": 1, "fechaEntrega": "2026-12-01", "total": 99.5,
                           "envio": {"direccion": {"ciudad": "Sevilla"}}})
p.model_dump(by_alias=True)      # {'idPedido': 1, 'fechaEntrega': '2026-12-01', 'importeTotal': 99.5, 'ciudad': 'Sevilla'}
```

| Opción | Efecto |
|---|---|
| `alias` | Nombre en entrada y salida (con `by_alias=True`). |
| `validation_alias` | Solo para la entrada; admite `AliasChoices` y `AliasPath`. |
| `serialization_alias` | Solo para la salida. |
| `alias_generator` | Genera alias para todos los campos (`to_camel`, `to_pascal`, `to_snake`). |
| `populate_by_name` / `validate_by_name` | Permite usar también el nombre Python del campo en la entrada. |

---

## 10. Configuración del modelo

```python
from pydantic import BaseModel, ConfigDict


class Estricto(BaseModel):
    model_config = ConfigDict(
        extra="forbid",                  # Error si llegan campos desconocidos ("ignore" por defecto, "allow")
        frozen=True,                     # Inmutable y con hash
        strict=True,                     # Sin conversiones de tipo
        str_strip_whitespace=True,       # Quita espacios en cadenas
        str_min_length=1,                # Ninguna cadena vacía
        validate_assignment=True,        # Valida también las asignaciones
        validate_default=True,           # Valida los valores por defecto
        from_attributes=True,            # Permite crear desde objetos (ORM): model_validate(obj)
        use_enum_values=True,            # Guarda el valor del Enum, no el miembro
        ser_json_timedelta="iso8601",
        json_schema_extra={"examples": [{"id": 1}]},
    )
```

Uso de `from_attributes` con un ORM:

```python
class ClienteSalida(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    nombre: str


salida = ClienteSalida.model_validate(cliente_orm)     # Lee atributos del objeto SQLAlchemy
```

Modelo base común para todo el proyecto:

```python
class ModeloBase(BaseModel):
    model_config = ConfigDict(extra="forbid", str_strip_whitespace=True, validate_assignment=True)


class Cliente(ModeloBase):
    nombre: str
```

---

## 11. Errores de validación

Pydantic recopila **todos** los errores, no solo el primero:

```python
from pydantic import ValidationError

try:
    Producto(sku="mal", nombre="X", precio=-5)
except ValidationError as e:
    print(e.error_count())     # 3
    print(e)                    # Texto legible
    for err in e.errors():
        print(err["loc"], err["type"], err["msg"])
```

```text
('sku',) string_pattern_mismatch String should match pattern '^[A-Z]{3}-\d{3}$'
('nombre',) string_too_short String should have at least 2 characters
('precio',) greater_than Input should be greater than 0
```

Estructura de cada error:

| Clave | Contenido |
|---|---|
| `type` | Código del error (`missing`, `int_parsing`, `greater_than`…). |
| `loc` | Ruta al campo: `("lineas", 2, "precio")`. |
| `msg` | Mensaje legible (en inglés). |
| `input` | Valor recibido. |
| `ctx` | Datos adicionales (p. ej. `{"gt": 0}`). |

Mensajes en español: traduzca a partir de `type` y `ctx`:

```python
MENSAJES = {
    "missing": "Campo obligatorio",
    "greater_than": "Debe ser mayor que {gt}",
    "string_too_short": "Debe tener al menos {min_length} caracteres",
    "string_pattern_mismatch": "Formato no válido",
    "int_parsing": "Debe ser un número entero",
}


def traducir(e: ValidationError) -> list[str]:
    salida = []
    for err in e.errors():
        plantilla = MENSAJES.get(err["type"], err["msg"])
        campo = ".".join(str(p) for p in err["loc"])
        salida.append(f"{campo}: {plantilla.format(**err.get('ctx', {}))}")
    return salida
```

Errores propios con código:

```python
from pydantic_core import PydanticCustomError

raise PydanticCustomError("nif_invalido", "El NIF {nif} no es válido", {"nif": v})
```

---

## 12. Modelos anidados, genéricos y uniones discriminadas

### 12.1 Anidados y autorreferenciados

```python
class Categoria(BaseModel):
    nombre: str
    subcategorias: list["Categoria"] = []


Categoria.model_validate({"nombre": "Informática", "subcategorias": [{"nombre": "Periféricos"}]})
```

### 12.2 Genéricos

```python
from pydantic import BaseModel


class Pagina[T](BaseModel):
    elementos: list[T]
    total: int
    pagina: int = 1


respuesta = Pagina[Cliente].model_validate_json(texto_json)
```

### 12.3 Uniones discriminadas

Cuando un campo indica el tipo de objeto, Pydantic elige el modelo correcto de forma rápida y con errores claros:

```python
from typing import Annotated, Literal

from pydantic import BaseModel, Field


class PagoTarjeta(BaseModel):
    metodo: Literal["tarjeta"]
    numero: str = Field(min_length=16, max_length=16)


class PagoTransferencia(BaseModel):
    metodo: Literal["transferencia"]
    iban: str


class PagoBizum(BaseModel):
    metodo: Literal["bizum"]
    telefono: str


Pago = Annotated[PagoTarjeta | PagoTransferencia | PagoBizum, Field(discriminator="metodo")]


class Cobro(BaseModel):
    importe: float
    pago: Pago


Cobro.model_validate({"importe": 50, "pago": {"metodo": "bizum", "telefono": "600000000"}})
```

---

## 13. `TypeAdapter`, `dataclasses` y `validate_call`

### 13.1 Validar tipos que no son modelos

```python
from pydantic import TypeAdapter

ListaClientes = TypeAdapter(list[Cliente])
clientes = ListaClientes.validate_json(texto_json)
ListaClientes.dump_json(clientes)

Numeros = TypeAdapter(dict[str, list[int]])
Numeros.validate_python({"a": ["1", "2"]})      # {'a': [1, 2]}
```

> Cree los `TypeAdapter` **una vez** (a nivel de módulo) y reutilícelos: construirlos tiene un coste.

### 13.2 Dataclasses con validación

```python
from pydantic.dataclasses import dataclass


@dataclass
class Punto:
    x: float
    y: float


Punto(x="1.5", y=2)       # Punto(x=1.5, y=2.0)
```

### 13.3 Validar argumentos de funciones

```python
from pydantic import validate_call, Field
from typing import Annotated


@validate_call
def transferir(origen: str, destino: str, importe: Annotated[float, Field(gt=0)]) -> str:
    return f"{importe:.2f} € de {origen} a {destino}"


transferir("ES01", "ES02", "100")     # ✔ Convierte "100" en 100.0
transferir("ES01", "ES02", -5)        # ✖ ValidationError
```

---

## 14. JSON Schema

```python
import json

print(json.dumps(Producto.model_json_schema(), indent=2, ensure_ascii=False))
print(Producto.model_json_schema(mode="serialization"))    # Esquema de la salida
TypeAdapter(list[Producto]).json_schema()
```

Fragmento generado:

```json
{
  "properties": {
    "sku": {"examples": ["TEC-001"], "pattern": "^[A-Z]{3}-\\d{3}$", "title": "Sku", "type": "string"},
    "precio": {"description": "Precio sin IVA en euros", "exclusiveMinimum": 0, "maximum": 100000, "title": "Precio", "type": "number"}
  },
  "required": ["sku", "nombre", "precio"],
  "title": "Producto",
  "type": "object"
}
```

Útil para documentar APIs (FastAPI lo usa para OpenAPI), validar en el *frontend* o generar formularios.

---

## 15. Integración con otras herramientas

| Herramienta | Integración |
|---|---|
| **FastAPI** | Los modelos definen el cuerpo de las peticiones y respuestas, con validación y documentación automáticas. |
| **pydantic-settings** | `BaseSettings` carga configuración desde entorno y `.env`. |
| **SQLModel** | Modelos que son a la vez tablas SQLAlchemy y modelos Pydantic. |
| **mypy** | *Plugin* `pydantic.mypy` para comprobar el constructor de los modelos. |
| **Pyright / ty** | Compatibles mediante `dataclass_transform` (sin *plugin*). |
| **Hypothesis** | `st.builds(Modelo)` genera instancias válidas para pruebas. |

Ejemplo con FastAPI:

```python
from fastapi import FastAPI

app = FastAPI()


@app.post("/productos", status_code=201)
def crear(producto: Producto) -> Producto:      # Valida la entrada y documenta la respuesta
    return producto
```

---

## 16. Rendimiento

- Use `model_validate_json(texto)` en lugar de `model_validate(json.loads(texto))`.
- Reutilice los `TypeAdapter`.
- Use uniones **discriminadas** en lugar de uniones simples de modelos.
- Para datos internos ya validados, `model_construct()` crea el modelo **sin validar** (úselo con cuidado).
- Evite validadores `mode="wrap"` en campos muy frecuentes si no los necesita.
- Prefiera tipos concretos (`list[int]`) a `Sequence[int]` o `Any` cuando no necesite flexibilidad.

---

## 17. Migrar desde Pydantic v1

| v1 | v2 |
|---|---|
| `class Config:` | `model_config = ConfigDict(...)` |
| `.dict()` | `.model_dump()` |
| `.json()` | `.model_dump_json()` |
| `.parse_obj()` | `.model_validate()` |
| `.parse_raw()` | `.model_validate_json()` |
| `.copy()` | `.model_copy()` |
| `.schema()` | `.model_json_schema()` |
| `@validator` | `@field_validator` |
| `@root_validator` | `@model_validator` |
| `orm_mode = True` | `from_attributes=True` |
| `allow_population_by_field_name` | `populate_by_name` / `validate_by_name` |
| `BaseSettings` (en pydantic) | Paquete `pydantic-settings` |
| `constr(...)`, `conint(...)` | `Annotated[str, StringConstraints(...)]`, `Annotated[int, Field(...)]` |

Herramienta de migración automática:

```bash
uvx bump-pydantic src/
```

---

## 18. Ejemplo práctico completo

**Importador de pedidos** desde un archivo JSON Lines (un pedido por línea) procedente de una tienda online: valida cada registro con modelos anidados, uniones discriminadas para el pago, validadores cruzados y alias en `camelCase`; separa los válidos de los erróneos, genera un informe de errores en español y exporta el resultado.

```python
"""
importar_pedidos.py — Valida pedidos en formato JSON Lines con Pydantic v2.

Uso:  python importar_pedidos.py pedidos.jsonl
"""
from __future__ import annotations

import json
import sys
from collections import Counter
from datetime import date
from decimal import Decimal
from pathlib import Path
from typing import Annotated, Literal, Self

from pydantic import (AfterValidator, BaseModel, ConfigDict, Field, ValidationError, computed_field,
                      field_validator, model_validator)
from pydantic.alias_generators import to_camel

# ------------------------------------------------------------------ Tipos reutilizables
LETRAS_NIF = "TRWAGMYFPDXBNJZSQVHLCKE"


def validar_nif(v: str) -> str:
    v = v.replace("-", "").replace(" ", "").upper()
    if len(v) != 9 or not v[:8].isdigit() or LETRAS_NIF[int(v[:8]) % 23] != v[8]:
        raise ValueError("NIF no válido")
    return v


Nif = Annotated[str, AfterValidator(validar_nif)]
Sku = Annotated[str, Field(pattern=r"^[A-Z]{3}-\d{3}$")]
Importe = Annotated[Decimal, Field(ge=0, max_digits=10, decimal_places=2)]
CodigoPostal = Annotated[str, Field(pattern=r"^(0[1-9]|[1-4]\d|5[0-2])\d{3}$")]


class Base(BaseModel):
    model_config = ConfigDict(alias_generator=to_camel, populate_by_name=True,
                              extra="forbid", str_strip_whitespace=True)


# ------------------------------------------------------------------ Modelos
class Direccion(Base):
    calle: str = Field(min_length=3)
    ciudad: str
    codigo_postal: CodigoPostal


class Cliente(Base):
    nif: Nif
    nombre: str = Field(min_length=2)
    email: str = Field(pattern=r"^[^@\s]+@[^@\s]+\.[a-z]{2,}$")


class Linea(Base):
    sku: Sku
    cantidad: int = Field(gt=0, le=100)
    precio_unitario: Importe

    @computed_field
    @property
    def subtotal(self) -> Decimal:
        return self.precio_unitario * self.cantidad


class PagoTarjeta(Base):
    metodo: Literal["tarjeta"]
    ultimos_digitos: str = Field(pattern=r"^\d{4}$")


class PagoTransferencia(Base):
    metodo: Literal["transferencia"]
    iban: str = Field(pattern=r"^ES\d{22}$")


class PagoContrareembolso(Base):
    metodo: Literal["contrareembolso"]
    recargo: Importe = Decimal("3.00")


Pago = Annotated[PagoTarjeta | PagoTransferencia | PagoContrareembolso, Field(discriminator="metodo")]


class Pedido(Base):
    id: int = Field(gt=0)
    fecha: date
    fecha_entrega: date | None = None
    cliente: Cliente
    envio: Direccion
    lineas: list[Linea] = Field(min_length=1)
    pago: Pago
    cupon: str | None = None

    @field_validator("cupon")
    @classmethod
    def cupon_mayusculas(cls, v: str | None) -> str | None:
        return v.upper() if v else None

    @model_validator(mode="after")
    def reglas_de_negocio(self) -> Self:
        if self.fecha_entrega and self.fecha_entrega < self.fecha:
            raise ValueError("la fecha de entrega es anterior a la del pedido")
        skus = [l.sku for l in self.lineas]
        if len(skus) != len(set(skus)):
            raise ValueError("hay productos repetidos; agrúpelos en una sola línea")
        if isinstance(self.pago, PagoContrareembolso) and self.total > 500:
            raise ValueError("el contrareembolso no admite pedidos de más de 500 €")
        return self

    @computed_field
    @property
    def total(self) -> Decimal:
        base = sum((l.subtotal for l in self.lineas), start=Decimal("0"))
        recargo = self.pago.recargo if isinstance(self.pago, PagoContrareembolso) else Decimal("0")
        return base + recargo


# ------------------------------------------------------------------ Mensajes en español
MENSAJES = {
    "missing": "campo obligatorio",
    "extra_forbidden": "campo no permitido",
    "greater_than": "debe ser mayor que {gt}",
    "less_than_equal": "debe ser como máximo {le}",
    "string_pattern_mismatch": "formato no válido",
    "string_too_short": "debe tener al menos {min_length} caracteres",
    "too_short": "debe tener al menos {min_length} elemento(s)",
    "int_parsing": "debe ser un número entero",
    "decimal_parsing": "debe ser un número",
    "date_from_datetime_parsing": "fecha no válida (use AAAA-MM-DD)",
    "date_parsing": "fecha no válida (use AAAA-MM-DD)",
    "union_tag_invalid": "método de pago desconocido: {tag}",
    "union_tag_not_found": "falta el método de pago",
    "value_error": "{error}",
}


def mensajes(e: ValidationError) -> list[str]:
    salida = []
    for err in e.errors(include_url=False):
        ruta = ".".join(str(p) for p in err["loc"]) or "pedido"
        plantilla = MENSAJES.get(err["type"], err["msg"])
        ctx = {k: str(v).removeprefix("Value error, ") for k, v in err.get("ctx", {}).items()}
        try:
            texto = plantilla.format(**ctx)
        except KeyError:
            texto = err["msg"]
        salida.append(f"{ruta}: {texto}")
    return salida


# ------------------------------------------------------------------ Proceso
def importar(ruta: Path) -> tuple[list[Pedido], list[tuple[int, list[str]]]]:
    validos: list[Pedido] = []
    errores: list[tuple[int, list[str]]] = []
    for n, linea in enumerate(ruta.read_text(encoding="utf-8").splitlines(), start=1):
        if not linea.strip():
            continue
        try:
            validos.append(Pedido.model_validate_json(linea))
        except ValidationError as e:
            errores.append((n, mensajes(e)))
    return validos, errores


def main() -> int:
    ruta = Path(sys.argv[1] if len(sys.argv) > 1 else "pedidos.jsonl")
    validos, errores = importar(ruta)

    print(f"Pedidos válidos: {len(validos)} · con errores: {len(errores)}\n")
    for n, lista in errores:
        print(f"Línea {n}:")
        for m in lista:
            print(f"   ✖ {m}")

    if validos:
        metodos = Counter(p.pago.metodo for p in validos)
        total = sum((p.total for p in validos), start=Decimal("0"))
        print(f"\nImporte total: {total} € · Métodos de pago: {dict(metodos)}")
        salida = ruta.with_suffix(".validos.json")
        salida.write_text(json.dumps([p.model_dump(mode="json", by_alias=True) for p in validos],
                                     indent=2, ensure_ascii=False), encoding="utf-8")
        print(f"Exportado a {salida}")
    return 1 if errores else 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Archivo de prueba `pedidos.jsonl`:

```json
{"id": 1, "fecha": "2026-11-20", "cliente": {"nif": "12345678Z", "nombre": "Ana García", "email": "ana@ejemplo.com"}, "envio": {"calle": "Calle Mayor 1", "ciudad": "Madrid", "codigoPostal": "28013"}, "lineas": [{"sku": "TEC-001", "cantidad": 2, "precioUnitario": "59.90"}], "pago": {"metodo": "tarjeta", "ultimosDigitos": "4242"}, "cupon": "navidad"}
{"id": 2, "fecha": "2026-11-21", "fechaEntrega": "2026-11-19", "cliente": {"nif": "12345678A", "nombre": "L", "email": "luis@"}, "envio": {"calle": "Av. Sol", "ciudad": "Sevilla", "codigoPostal": "99999"}, "lineas": [], "pago": {"metodo": "paypal"}}
{"id": 3, "fecha": "2026-11-22", "cliente": {"nif": "87654321X", "nombre": "Marta Ruiz", "email": "marta@ejemplo.es"}, "envio": {"calle": "Gran Vía 10", "ciudad": "Bilbao", "codigoPostal": "48001"}, "lineas": [{"sku": "MON-200", "cantidad": 3, "precioUnitario": "189.00"}], "pago": {"metodo": "contrareembolso"}}
```

Resultado:

```text
Pedidos válidos: 1 · con errores: 2

Línea 2:
   ✖ cliente.nif: NIF no válido
   ✖ cliente.nombre: debe tener al menos 2 caracteres
   ✖ cliente.email: formato no válido
   ✖ envio.codigoPostal: formato no válido
   ✖ lineas: debe tener al menos 1 elemento(s)
   ✖ pago: método de pago desconocido: paypal
Línea 3:
   ✖ pedido: el contrareembolso no admite pedidos de más de 500 €

Importe total: 119.80 € · Métodos de pago: {'tarjeta': 1}
Exportado a pedidos.validos.json
```

> Observe que el error de fechas del pedido 2 no aparece: los validadores de modelo (`mode="after"`) solo se ejecutan si todos los campos son válidos.

---

## 19. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ImportError: email-validator is not installed` | Uso de `EmailStr` sin la dependencia. | `pip install "pydantic[email]"`. |
| `PydanticUserError: A non-annotated attribute was detected` | Atributo de clase sin anotación. | Anótelo o use `ClassVar`. |
| `AttributeError: 'Modelo' object has no attribute 'dict'` | Código de Pydantic v1. | `model_dump()`; vea la sección 17. |
| `extra_forbidden` inesperado | `extra="forbid"` y la entrada trae campos adicionales. | Revise la entrada o use `extra="ignore"`. |
| Un `"1"` se acepta como `1` sin querer | Modo *lax* por defecto. | `strict=True` en el campo o el modelo. |
| Los cambios de atributo no se validan | `validate_assignment` desactivado. | Actívelo o use modelos inmutables (`frozen=True`). |
| Alias ignorados al serializar | Falta `by_alias=True`. | `model_dump(by_alias=True)` o `serialize_by_alias=True` en la configuración. |
| No se puede usar el nombre Python con alias definidos | Falta `populate_by_name`. | Actívelo en `model_config`. |
| El validador de modelo no se ejecuta | Algún campo falló antes. | Es el comportamiento esperado: corrija primero los campos. |
| `PydanticSchemaGenerationError` con una clase propia | Pydantic no sabe validar ese tipo. | `arbitrary_types_allowed=True` o defina `__get_pydantic_core_schema__`. |

---

## 20. Buenas prácticas

- Use Pydantic en los **bordes** del sistema (APIs, archivos, colas, configuración); en el núcleo, `dataclasses` o modelos ya validados.
- Defina un **modelo base** con la configuración común del proyecto (`extra="forbid"`, `str_strip_whitespace`).
- Cree **tipos reutilizables** con `Annotated` (NIF, SKU, importe) en lugar de repetir restricciones.
- Prefiera las restricciones declarativas (`Field`, `StringConstraints`) a los validadores manuales.
- Use **uniones discriminadas** para datos polimórficos.
- Use `Decimal` para importes, nunca `float`.
- Use `SecretStr` para contraseñas y tokens: no aparecerán en registros ni trazas.
- Valide JSON directamente con `model_validate_json`.
- Separe modelos de **entrada** y de **salida** en las APIs (no exponga campos internos).
- Active el *plugin* de mypy o use Pyright/ty para comprobar los constructores en desarrollo.

---

## 21. Referencia rápida

```python
class M(BaseModel):
    model_config = ConfigDict(extra="forbid", frozen=True, strict=True, alias_generator=to_camel)
    campo: int = Field(gt=0, alias="x", description="...")
    lista: list[str] = Field(default_factory=list)

    @field_validator("campo", mode="before") @classmethod def v(cls, v): ...
    @model_validator(mode="after") def m(self) -> Self: ...
    @computed_field @property def calc(self) -> int: ...

M(**datos) · M.model_validate(dict) · M.model_validate_json(str)
m.model_dump(mode="json", exclude_none=True, by_alias=True) · m.model_dump_json()
m.model_copy(update={...}) · M.model_json_schema()
TypeAdapter(list[M]).validate_json(s) · @validate_call
Annotated[T, Field(...) | AfterValidator(f) | BeforeValidator(f)]
Annotated[A | B, Field(discriminator="tipo")]
except ValidationError as e: e.errors()
```

---

## 22. Ejercicios propuestos

1. Modele la respuesta JSON de una API pública (por ejemplo, la de PyPI) y valídela con `model_validate_json`.
2. Cree un tipo `Iban` con `Annotated` y un validador que compruebe el dígito de control.
3. Amplíe el importador con un nuevo método de pago `bizum` y verifique que la unión discriminada lo reconoce.
4. Genere el JSON Schema de `Pedido` y úselo para validar los datos con la librería `jsonschema`.
5. Escriba pruebas con Hypothesis que generen instancias válidas de `Linea` con `st.builds` y comprueben que `subtotal` nunca es negativo.

---

## 23. Referencias

- Documentación oficial: https://docs.pydantic.dev/latest/
- Conceptos (modelos, campos, validadores): https://docs.pydantic.dev/latest/concepts/models/
- Tipos admitidos: https://docs.pydantic.dev/latest/api/standard_library_types/
- Errores de validación: https://docs.pydantic.dev/latest/errors/validation_errors/
- Guía de migración a v2: https://docs.pydantic.dev/latest/migration/
- pydantic-extra-types: https://github.com/pydantic/pydantic-extra-types
