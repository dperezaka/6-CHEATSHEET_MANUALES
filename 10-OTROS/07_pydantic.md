# Manual de uso: Pydantic

> **Categoría:** Utilidades generales · **Utilidad:** Validar y estructurar datos
> **Tipo:** Librería externa · **Versión de referencia:** Pydantic 2.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Restricciones con Field](#5-restricciones-con-field)
6. [Tipos especiales](#6-tipos-especiales)
7. [Validadores personalizados](#7-validadores-personalizados)
8. [Serialización](#8-serialización)
9. [Operaciones avanzadas](#9-operaciones-avanzadas)
10. [Configuración con pydantic-settings](#10-configuración-con-pydantic-settings)
11. [Ejemplo práctico completo](#11-ejemplo-práctico-completo)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**Pydantic** valida y convierte datos usando las anotaciones de tipo de Python. Se define un modelo con los campos y sus tipos, y Pydantic se encarga de:

- **Validar** que los datos cumplen el esquema.
- **Convertir** tipos compatibles (por ejemplo, `"42"` → `42`).
- **Informar** de errores de forma clara y estructurada.
- **Serializar** a diccionarios y JSON.
- **Generar** esquemas JSON automáticamente.

Es la base de **FastAPI** y se usa ampliamente para validar respuestas de APIs, archivos de configuración, formularios y datos de entrada en general.

---

## 2. Instalación e importación

```bash
pip install pydantic
```

Extras útiles:

```bash
pip install "pydantic[email]"        # Validación de emails
pip install pydantic-settings        # Configuración desde variables de entorno
```

```python
from pydantic import BaseModel, Field, ValidationError, field_validator, model_validator
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `BaseModel` | Clase base para definir modelos de datos. |
| Campo | Atributo anotado con su tipo. |
| Validación | Ocurre automáticamente al crear el objeto. |
| Coerción | Conversión automática de tipos compatibles (modo *lax*). |
| `ValidationError` | Excepción con el detalle de todos los errores. |
| `model_dump()` | Convierte el modelo en diccionario. |
| `model_validate()` | Crea el modelo a partir de un diccionario. |

---

## 4. Operaciones básicas

### 4.1 Definir un modelo

```python
from datetime import date
from pydantic import BaseModel


class Usuario(BaseModel):
    id: int
    nombre: str
    email: str
    activo: bool = True                   # Valor por defecto
    fecha_alta: date | None = None        # Opcional
    roles: list[str] = []                 # Pydantic copia los valores mutables de forma segura
```

### 4.2 Crear instancias

```python
u = Usuario(id=1, nombre="Ana", email="ana@ejemplo.com")
print(u)
print(u.nombre)
```

### 4.3 Conversión automática de tipos

```python
u = Usuario(id="42", nombre="Luis", email="luis@ejemplo.com",
            activo="true", fecha_alta="2026-11-24")

u.id            # 42 (int)
u.activo        # True
u.fecha_alta    # datetime.date(2026, 11, 24)
```

> Valores aceptados como booleanos: `true/false`, `1/0`, `yes/no`, `on/off`, `t/f`, `y/n`. Un valor como `"sí"` **no** es válido y genera un error.

### 4.4 Gestión de errores

```python
from pydantic import ValidationError

try:
    Usuario(id="abc", nombre="Ana")
except ValidationError as e:
    print(e.error_count(), "errores")
    for error in e.errors():
        print(error["loc"], "→", error["msg"])
```

Salida:

```
2 errores
('id',) → Input should be a valid integer, unable to parse string as an integer
('email',) → Field required
```

### 4.5 Crear desde diccionario o JSON

```python
datos = {"id": 3, "nombre": "Marta", "email": "marta@ejemplo.com"}
u = Usuario.model_validate(datos)

json_texto = '{"id": 4, "nombre": "Pedro", "email": "pedro@ejemplo.com"}'
u = Usuario.model_validate_json(json_texto)
```

### 4.6 Modelos anidados

```python
class Direccion(BaseModel):
    calle: str
    ciudad: str
    cp: str


class Cliente(BaseModel):
    nombre: str
    direccion: Direccion
    direcciones_envio: list[Direccion] = []


c = Cliente.model_validate({
    "nombre": "Ana",
    "direccion": {"calle": "Mayor 1", "ciudad": "Madrid", "cp": "28013"},
})
c.direccion.ciudad
```

---

## 5. Restricciones con Field

```python
from pydantic import Field


class Producto(BaseModel):
    codigo: str = Field(pattern=r"^[A-Z]{2}\d{4}$", description="Código interno")
    nombre: str = Field(min_length=2, max_length=100)
    precio: float = Field(gt=0, le=10_000)
    stock: int = Field(default=0, ge=0)
    descuento: float = Field(default=0, ge=0, le=1)
    etiquetas: list[str] = Field(default_factory=list, max_length=10)
```

| Restricción | Significado |
|---|---|
| `gt`, `ge` | Mayor que / mayor o igual que |
| `lt`, `le` | Menor que / menor o igual que |
| `multiple_of` | Múltiplo de |
| `min_length`, `max_length` | Longitud de cadenas y listas |
| `pattern` | Expresión regular |
| `default` / `default_factory` | Valor por defecto / función que lo genera |
| `alias` | Nombre alternativo en los datos de entrada |
| `description`, `examples` | Documentación (aparece en el esquema JSON) |

### 5.1 Alias (nombres distintos en origen)

```python
class Pedido(BaseModel):
    id_pedido: int = Field(alias="orderId")
    importe_total: float = Field(alias="totalAmount")

Pedido.model_validate({"orderId": 10, "totalAmount": 99.5})
```

---

## 6. Tipos especiales

```python
from datetime import datetime
from decimal import Decimal
from uuid import UUID
from typing import Literal
from pydantic import (
    EmailStr, HttpUrl, SecretStr, PositiveInt, NonNegativeFloat,
    conint, constr, IPvAnyAddress,
)


class Cuenta(BaseModel):
    id: UUID
    email: EmailStr                         # Requiere pydantic[email]
    web: HttpUrl | None = None
    clave: SecretStr                        # Se oculta al imprimir
    saldo: Decimal
    intentos: PositiveInt = 1
    tipo: Literal["personal", "empresa"]
    creada: datetime
    ip: IPvAnyAddress | None = None


c = Cuenta(
    id="6f1c1a3e-6c9e-4c9e-9f3a-2a1b3c4d5e6f",
    email="ana@ejemplo.com",
    clave="secreto123",
    saldo="1500.50",
    tipo="personal",
    creada="2026-11-24T10:00:00",
)
print(c.clave)                       # **********
print(c.clave.get_secret_value())    # secreto123
```

### 6.1 Enumeraciones

```python
from enum import Enum

class Estado(str, Enum):
    PENDIENTE = "pendiente"
    ENVIADO = "enviado"
    ENTREGADO = "entregado"

class Envio(BaseModel):
    estado: Estado = Estado.PENDIENTE
```

### 6.2 Tipos con restricciones reutilizables

```python
from typing import Annotated

CodigoPostal = Annotated[str, Field(pattern=r"^\d{5}$")]
Porcentaje = Annotated[float, Field(ge=0, le=100)]

class Tienda(BaseModel):
    cp: CodigoPostal
    margen: Porcentaje
```

---

## 7. Validadores personalizados

### 7.1 Validador de campo

```python
from pydantic import field_validator


class Registro(BaseModel):
    nombre: str
    dni: str

    @field_validator("nombre")
    @classmethod
    def normalizar_nombre(cls, v: str) -> str:
        return " ".join(v.split()).title()

    @field_validator("dni")
    @classmethod
    def validar_dni(cls, v: str) -> str:
        v = v.upper().replace("-", "").strip()
        letras = "TRWAGMYFPDXBNJZSQVHLCKE"
        if len(v) != 9 or not v[:8].isdigit() or letras[int(v[:8]) % 23] != v[8]:
            raise ValueError("DNI no válido")
        return v
```

Modos del validador:
- `mode="after"` (por defecto): se ejecuta después de la validación de tipo.
- `mode="before"`: se ejecuta antes, sobre el dato bruto.

```python
    @field_validator("precio", mode="before")
    @classmethod
    def coma_decimal(cls, v):
        return v.replace(",", ".") if isinstance(v, str) else v
```

### 7.2 Validador de modelo (varios campos a la vez)

```python
from datetime import date
from pydantic import model_validator


class Reserva(BaseModel):
    entrada: date
    salida: date
    huespedes: int

    @model_validator(mode="after")
    def comprobar_fechas(self):
        if self.salida <= self.entrada:
            raise ValueError("La fecha de salida debe ser posterior a la de entrada")
        return self
```

### 7.3 Campos calculados

```python
from pydantic import computed_field


class LineaPedido(BaseModel):
    precio: float
    unidades: int

    @computed_field
    @property
    def subtotal(self) -> float:
        return round(self.precio * self.unidades, 2)

LineaPedido(precio=9.99, unidades=3).model_dump()
# {'precio': 9.99, 'unidades': 3, 'subtotal': 29.97}
```

---

## 8. Serialización

```python
u = Usuario(id=1, nombre="Ana", email="ana@ejemplo.com")

u.model_dump()                              # dict
u.model_dump(exclude={"email"})             # Excluir campos
u.model_dump(include={"id", "nombre"})      # Solo algunos campos
u.model_dump(exclude_none=True)             # Omitir valores None
u.model_dump(exclude_unset=True)            # Solo los campos asignados explícitamente
u.model_dump(by_alias=True)                 # Usar alias en las claves
u.model_dump(mode="json")                   # Tipos compatibles con JSON

u.model_dump_json(indent=2)                 # Texto JSON
```

### 8.1 Copiar y modificar

```python
u2 = u.model_copy(update={"nombre": "Ana María"})
```

### 8.2 Esquema JSON

```python
import json
print(json.dumps(Producto.model_json_schema(), indent=2, ensure_ascii=False))
```

---

## 9. Operaciones avanzadas

### 9.1 Configuración del modelo

```python
from pydantic import ConfigDict


class ModeloEstricto(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,    # Elimina espacios en cadenas
        extra="forbid",               # Error si llegan campos no definidos ("ignore" por defecto)
        frozen=True,                  # Objetos inmutables
        validate_assignment=True,     # Valida también al modificar atributos
        populate_by_name=True,        # Permite usar el nombre o el alias
    )

    nombre: str
```

### 9.2 Modo estricto (sin conversión automática)

```python
class Estricto(BaseModel):
    model_config = ConfigDict(strict=True)
    edad: int

Estricto(edad="30")     # ValidationError: no convierte "30" en 30
```

### 9.3 Validar tipos sueltos con `TypeAdapter`

```python
from pydantic import TypeAdapter

adaptador = TypeAdapter(list[Usuario])
usuarios = adaptador.validate_python([
    {"id": 1, "nombre": "Ana", "email": "a@x.com"},
    {"id": 2, "nombre": "Luis", "email": "l@x.com"},
])
```

### 9.4 Uniones discriminadas

```python
from typing import Literal, Union
from pydantic import Field


class PagoTarjeta(BaseModel):
    tipo: Literal["tarjeta"]
    numero: str


class PagoTransferencia(BaseModel):
    tipo: Literal["transferencia"]
    iban: str


class Pago(BaseModel):
    metodo: Union[PagoTarjeta, PagoTransferencia] = Field(discriminator="tipo")


Pago.model_validate({"metodo": {"tipo": "transferencia", "iban": "ES12..."}})
```

### 9.5 Serializadores personalizados

```python
from pydantic import field_serializer


class Factura(BaseModel):
    importe: Decimal
    fecha: date

    @field_serializer("fecha")
    def fecha_es(self, v: date) -> str:
        return v.strftime("%d/%m/%Y")
```

### 9.6 Validar argumentos de funciones

```python
from pydantic import validate_call

@validate_call
def calcular_iva(base: float, tipo: Literal[4, 10, 21] = 21) -> float:
    return round(base * tipo / 100, 2)

calcular_iva("100")          # 21.0
calcular_iva(100, tipo=15)   # ValidationError
```

---

## 10. Configuración con pydantic-settings

Carga y valida la configuración desde variables de entorno o un archivo `.env`:

```python
from pydantic import SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict


class Ajustes(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_prefix="APP_")

    nombre: str = "MiApp"
    debug: bool = False
    db_url: str
    api_key: SecretStr
    puerto: int = 8000


ajustes = Ajustes()          # Lee APP_DB_URL, APP_API_KEY, APP_DEBUG…
```

`.env`:

```
APP_DB_URL=postgresql://user:pass@localhost/app
APP_API_KEY=abc123
APP_DEBUG=true
```

---

## 11. Ejemplo práctico completo

Validación de un archivo de pedidos con informe de errores por registro:

```python
import json
from datetime import date
from decimal import Decimal
from typing import Literal

from pydantic import (
    BaseModel, ConfigDict, EmailStr, Field, ValidationError,
    computed_field, field_validator, model_validator,
)


class Linea(BaseModel):
    producto: str = Field(min_length=1)
    unidades: int = Field(gt=0)
    precio: Decimal = Field(gt=0, decimal_places=2)

    @computed_field
    @property
    def subtotal(self) -> Decimal:
        return self.precio * self.unidades


class Pedido(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, extra="forbid")

    id: int
    cliente: str
    email: EmailStr
    fecha: date
    estado: Literal["pendiente", "enviado", "entregado"] = "pendiente"
    lineas: list[Linea] = Field(min_length=1)
    descuento: Decimal = Field(default=Decimal("0"), ge=0, le=1)

    @field_validator("cliente")
    @classmethod
    def capitalizar(cls, v: str) -> str:
        return v.title()

    @model_validator(mode="after")
    def fecha_no_futura(self):
        if self.fecha > date.today():
            raise ValueError("La fecha del pedido no puede ser futura")
        return self

    @computed_field
    @property
    def total(self) -> Decimal:
        bruto = sum((l.subtotal for l in self.lineas), Decimal("0"))
        return (bruto * (1 - self.descuento)).quantize(Decimal("0.01"))


DATOS = """
[
  {"id": 1, "cliente": "ana lópez", "email": "ana@ejemplo.com", "fecha": "2026-11-20",
   "lineas": [{"producto": "Teclado", "unidades": 2, "precio": "29.90"}]},
  {"id": 2, "cliente": "luis", "email": "no-es-email", "fecha": "2026-11-21",
   "lineas": [{"producto": "Ratón", "unidades": 0, "precio": "14.50"}]},
  {"id": 3, "cliente": "marta", "email": "marta@ejemplo.com", "fecha": "2026-11-22",
   "estado": "enviado", "descuento": "0.1",
   "lineas": [{"producto": "Monitor", "unidades": 1, "precio": "189.00"},
              {"producto": "Cable", "unidades": 3, "precio": "4.50"}]}
]
"""

validos: list[Pedido] = []
for registro in json.loads(DATOS):
    try:
        validos.append(Pedido.model_validate(registro))
    except ValidationError as e:
        print(f"\n❌ Pedido {registro.get('id')} rechazado:")
        for err in e.errors():
            campo = ".".join(str(x) for x in err["loc"])
            print(f"   - {campo}: {err['msg']}")

print(f"\n✅ {len(validos)} pedidos válidos")
for p in validos:
    print(f"   #{p.id} {p.cliente:<12} {p.estado:<10} {p.total:>8} €")

with open("pedidos_validos.json", "w", encoding="utf-8") as f:
    json.dump([p.model_dump(mode="json") for p in validos], f, indent=2, ensure_ascii=False)
```

---

## 12. Errores frecuentes

| Problema | Causa | Solución |
|---|---|---|
| `AttributeError: 'Model' has no attribute 'dict'` | Código de Pydantic v1. | Use `model_dump()` (v2). |
| `parse_obj` / `parse_raw` no existen | Métodos de v1. | Use `model_validate()` / `model_validate_json()`. |
| `@validator` obsoleto | Sintaxis de v1. | Use `@field_validator` con `@classmethod`. |
| `ImportError: email-validator is not installed` | Uso de `EmailStr` sin el extra. | `pip install "pydantic[email]"`. |
| Se ignoran campos desconocidos sin aviso | `extra="ignore"` por defecto. | Configure `extra="forbid"`. |
| Asignar un valor incorrecto no da error | No se valida al asignar. | Active `validate_assignment=True`. |
| `BaseSettings` no encontrado | Se movió a otro paquete en v2. | `pip install pydantic-settings`. |

---

## 13. Buenas prácticas

- Valide todos los datos que provienen del exterior (APIs, archivos, formularios, variables de entorno).
- Defina modelos separados para entrada, almacenamiento y salida.
- Use `extra="forbid"` en entradas críticas para detectar errores de nombres de campo.
- Use `SecretStr` para contraseñas y claves.
- Use `Decimal` para importes monetarios en lugar de `float`.
- Reutilice restricciones con `Annotated` para mantener la coherencia.
- Muestre al usuario los mensajes de `e.errors()` de forma legible.

---

## 14. Referencia rápida

```python
class M(BaseModel):
    model_config = ConfigDict(extra="forbid", validate_assignment=True)
    campo: int = Field(gt=0, alias="campoExterno")

    @field_validator("campo") @classmethod def v(cls, x): ...
    @model_validator(mode="after") def m(self): ...; return self
    @computed_field @property def calc(self) -> int: ...

M(**datos) | M.model_validate(dict) | M.model_validate_json(txt)
m.model_dump(exclude=..., mode="json") | m.model_dump_json(indent=2)
m.model_copy(update={...}) | M.model_json_schema()
TypeAdapter(list[M]).validate_python(lista)
@validate_call
```

---

## 15. Ejercicios propuestos

1. Defina un modelo `Empleado` con validación de DNI, email, salario positivo y fecha de alta no futura.
2. Lea un CSV de clientes, valide cada fila con Pydantic y genere un informe de errores.
3. Cree un modelo de configuración con `pydantic-settings` que lea un archivo `.env`.
4. Modele la respuesta de una API pública (por ejemplo, `https://jsonplaceholder.typicode.com/users`) y valídela.

---

## 16. Referencias

- Documentación oficial: https://docs.pydantic.dev/latest/
- pydantic-settings: https://docs.pydantic.dev/latest/concepts/pydantic_settings/
- Guía de migración v1 → v2: https://docs.pydantic.dev/latest/migration/
