# Manual de uso: Hypothesis

> **Categoría:** Pruebas · **Utilidad:** Pruebas basadas en propiedades que generan casos automáticamente
> **Tipo:** Paquete externo (requiere instalación) · Se integra con pytest y unittest

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera prueba basada en propiedades](#4-primera-prueba-basada-en-propiedades)
5. [Estrategias básicas](#5-estrategias-básicas)
6. [Transformar y combinar estrategias](#6-transformar-y-combinar-estrategias)
7. [Estrategias compuestas y recursivas](#7-estrategias-compuestas-y-recursivas)
8. [Estrategias a partir de tipos y modelos](#8-estrategias-a-partir-de-tipos-y-modelos)
9. [Qué propiedades probar](#9-qué-propiedades-probar)
10. [Reducción de ejemplos (*shrinking*)](#10-reducción-de-ejemplos-shrinking)
11. [Control de la generación: `assume`, `example`, `note`, `target`](#11-control-de-la-generación-assume-example-note-target)
12. [Configuración y perfiles](#12-configuración-y-perfiles)
13. [Base de datos de ejemplos y reproducibilidad](#13-base-de-datos-de-ejemplos-y-reproducibilidad)
14. [Pruebas con estado (*stateful*)](#14-pruebas-con-estado-stateful)
15. [Integración con pytest y otras librerías](#15-integración-con-pytest-y-otras-librerías)
16. [Generar pruebas automáticamente (*ghostwriter*)](#16-generar-pruebas-automáticamente-ghostwriter)
17. [Ejemplo práctico completo](#17-ejemplo-práctico-completo)
18. [Errores frecuentes](#18-errores-frecuentes)
19. [Buenas prácticas](#19-buenas-prácticas)
20. [Referencia rápida](#20-referencia-rápida)
21. [Ejercicios propuestos](#21-ejercicios-propuestos)
22. [Referencias](#22-referencias)

---

## 1. Descripción

**Hypothesis** es una librería de **pruebas basadas en propiedades** (*property-based testing*). En lugar de escribir ejemplos concretos, se describe:

1. **qué datos** son válidos (mediante *estrategias*), y
2. **qué propiedad** debe cumplirse siempre.

Hypothesis genera cientos de casos, incluidos los extremos que rara vez se nos ocurren (cadenas vacías, Unicode, `NaN`, números enormes, listas con duplicados), y cuando encuentra un fallo lo **reduce** al ejemplo más simple posible.

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_ordenar_es_idempotente(lista):
    assert sorted(sorted(lista)) == sorted(lista)
```

**Ventajas:**
- Encuentra errores en casos límite que las pruebas con ejemplos no cubren.
- Ejemplos mínimos y reproducibles cuando algo falla.
- Se integra con pytest sin cambiar la forma de ejecutar las pruebas.
- Estrategias para tipos estándar, *dataclasses*, Pydantic, NumPy, pandas y Django.
- Pruebas con estado para sistemas con operaciones secuenciales (APIs, cachés, bases de datos).

**Limitaciones:**
- Exige pensar en **propiedades generales**, lo que al principio cuesta más que escribir ejemplos.
- Las pruebas tardan más (ejecutan muchos casos); ajustable con perfiles.
- No sustituye a las pruebas con ejemplos concretos de negocio: las complementa.

---

## 2. Instalación

```bash
uv add --dev hypothesis
uv add --dev "hypothesis[cli]"            # Incluye el ghostwriter («hypothesis write»)
python -m pip install hypothesis
```

Extras disponibles:

| Extra | Habilita |
|---|---|
| `cli` | Comando `hypothesis` y el *ghostwriter*. |
| `numpy`, `pandas` | Estrategias para *arrays* y *DataFrames*. |
| `django` | Estrategias para modelos y formularios Django. |
| `dateutil`, `pytz` | Zonas horarias adicionales. |
| `lark`, `regex` | Generación a partir de gramáticas y expresiones regulares avanzadas. |

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Propiedad | Afirmación que debe cumplirse para **cualquier** entrada válida. |
| Estrategia | Descripción de cómo generar datos (`st.integers()`, `st.text()`…). |
| `@given` | Decorador que ejecuta la prueba con datos generados. |
| Ejemplo | Un conjunto concreto de datos generados. |
| Reducción (*shrinking*) | Simplificar un ejemplo que falla hasta el mínimo que sigue fallando. |
| Base de datos de ejemplos | Directorio `.hypothesis/` donde se guardan los fallos para repetirlos. |
| Perfil | Conjunto de ajustes (número de ejemplos, plazos) para un entorno (local, CI). |
| Máquina de estados | Prueba que genera secuencias de operaciones (*stateful testing*). |

---

## 4. Primera prueba basada en propiedades

Función con un error sutil:

```python
def normalizar_nombre(nombre: str) -> str:
    """Convierte «  ana   GARCÍA » en «Ana García»."""
    return " ".join(p.capitalize() for p in nombre.split(" ") if p)
```

Pruebas con ejemplos (pasan):

```python
def test_ejemplos():
    assert normalizar_nombre("  ana   GARCÍA ") == "Ana García"
    assert normalizar_nombre("luis") == "Luis"
```

Prueba con propiedades:

```python
from hypothesis import given, strategies as st


@given(st.text())
def test_es_idempotente(nombre):
    una_vez = normalizar_nombre(nombre)
    assert normalizar_nombre(una_vez) == una_vez


@given(st.text())
def test_sin_espacios_extremos_ni_dobles(nombre):
    resultado = normalizar_nombre(nombre)
    assert resultado == resultado.strip()
    assert "  " not in resultado
```

```text
$ pytest test_nombres.py
E       AssertionError: assert '\x1f' == ''
E       Failing test case: test_sin_espacios_extremos_ni_dobles(
E           nombre='\x1f',
E       )
FAILED test_nombres.py::test_sin_espacios_extremos_ni_dobles
```

Hypothesis encontró que la función solo separa por el carácter espacio, no por otros caracteres que Python considera espacio en blanco (aquí, el separador de unidad `\x1f`; en otras ejecuciones puede aparecer un tabulador o un salto de línea). Corrección: `nombre.split()` en lugar de `nombre.split(" ")`.

---

## 5. Estrategias básicas

```python
from hypothesis import strategies as st

st.integers()                              # Cualquier entero
st.integers(min_value=0, max_value=100)
st.floats()                                # Incluye NaN, inf, -0.0
st.floats(min_value=0, max_value=1e6, allow_nan=False, allow_infinity=False)
st.decimals(min_value=0, max_value=10_000, places=2, allow_nan=False)
st.booleans()
st.none()
st.text()                                  # Unicode completo, incluida la cadena vacía
st.text(min_size=1, max_size=50)
st.text(alphabet=st.characters(categories=["Lu", "Ll", "Nd"]), min_size=1)   # Letras y dígitos
st.text(alphabet="ACGT", min_size=4)
st.from_regex(r"[A-Z]{3}-\d{3}", fullmatch=True)    # Cadenas que cumplen una expresión regular
st.binary(max_size=1024)
st.emails()
st.uuids()
st.ip_addresses(v=4)
st.dates(min_value=date(2020, 1, 1), max_value=date(2030, 12, 31))
st.datetimes(timezones=st.timezones())
st.timedeltas(max_value=timedelta(days=30))
st.sampled_from(["EUR", "USD", "GBP"])     # Uno de una lista (o de un Enum)
st.just(42)                                # Siempre el mismo valor
```

Colecciones:

```python
st.lists(st.integers())
st.lists(st.integers(), min_size=1, max_size=20, unique=True)
st.sets(st.text(max_size=5))
st.tuples(st.integers(), st.text())
st.dictionaries(keys=st.text(min_size=1), values=st.integers())
st.fixed_dictionaries({"id": st.integers(min_value=1), "nombre": st.text(min_size=1)},
                      optional={"email": st.emails()})
st.permutations([1, 2, 3, 4])
```

Explorar qué genera una estrategia:

```python
>>> st.lists(st.integers(0, 9), max_size=4).example()
[3, 0, 9]
```

> `.example()` es solo para explorar en la consola; no lo use dentro de las pruebas.

---

## 6. Transformar y combinar estrategias

```python
# map: transformar el valor generado
pares = st.integers().map(lambda x: x * 2)
importes = st.integers(min_value=1, max_value=1_000_000).map(lambda c: Decimal(c) / 100)

# filter: descartar valores (úselo con moderación)
impares = st.integers().filter(lambda x: x % 2 == 1)

# one_of / |: varias alternativas
identificador = st.integers(min_value=1) | st.uuids()
valor_json = st.one_of(st.none(), st.booleans(), st.integers(), st.text())

# flatmap: una estrategia que depende de un valor generado
lista_e_indice = st.lists(st.integers(), min_size=1).flatmap(
    lambda xs: st.tuples(st.just(xs), st.integers(min_value=0, max_value=len(xs) - 1))
)
```

> `filter` descarta ejemplos: si descarta demasiados, Hypothesis avisa (`FailedHealthCheck`). Prefiera construir directamente datos válidos con `map` o con límites (`min_value`, `min_size`).

---

## 7. Estrategias compuestas y recursivas

### 7.1 `@st.composite`

Para datos con relaciones internas:

```python
from datetime import date, timedelta

from hypothesis import strategies as st


@st.composite
def reservas(draw):
    entrada = draw(st.dates(min_value=date(2026, 1, 1), max_value=date(2027, 12, 31)))
    noches = draw(st.integers(min_value=1, max_value=30))
    personas = draw(st.integers(min_value=1, max_value=6))
    return {"entrada": entrada, "salida": entrada + timedelta(days=noches), "personas": personas}


@given(reservas())
def test_salida_posterior(reserva):
    assert reserva["salida"] > reserva["entrada"]
```

### 7.2 `st.data()`: generar dentro de la prueba

```python
@given(st.lists(st.integers(), min_size=1), st.data())
def test_eliminar_elemento(lista, data):
    indice = data.draw(st.integers(min_value=0, max_value=len(lista) - 1), label="indice")
    copia = lista.copy()
    del copia[indice]
    assert len(copia) == len(lista) - 1
```

### 7.3 Estructuras recursivas

```python
json_valido = st.recursive(
    st.none() | st.booleans() | st.integers() | st.floats(allow_nan=False) | st.text(),
    lambda hijos: st.lists(hijos, max_size=5) | st.dictionaries(st.text(), hijos, max_size=5),
    max_leaves=20,
)


@given(json_valido)
def test_json_ida_y_vuelta(valor):
    assert json.loads(json.dumps(valor)) == valor
```

---

## 8. Estrategias a partir de tipos y modelos

```python
from dataclasses import dataclass
from enum import Enum

from hypothesis import given, strategies as st


class Moneda(Enum):
    EUR = "EUR"
    USD = "USD"


@dataclass
class Importe:
    cantidad: int
    moneda: Moneda


st.from_type(Importe)                         # Deduce la estrategia de las anotaciones
st.from_type(list[tuple[int, str]])
st.builds(Importe, cantidad=st.integers(min_value=0))   # Ajusta algunos argumentos


@given(st.builds(Importe, cantidad=st.integers(0, 10**9)))
def test_importe(importe):
    assert importe.cantidad >= 0
```

Registrar una estrategia para un tipo propio:

```python
st.register_type_strategy(Nif, st.integers(0, 99_999_999).map(nif_con_letra))
```

Con **Pydantic**, `st.builds(Modelo)` usa las anotaciones; para respetar restricciones (`Field(gt=0)`, patrones), indíquelas explícitamente:

```python
st.builds(Producto, sku=st.from_regex(r"[A-Z]{3}-\d{3}", fullmatch=True),
          precio=st.floats(min_value=0.01, max_value=1e5))
```

---

## 9. Qué propiedades probar

Encontrar propiedades es la habilidad central. Patrones habituales:

| Patrón | Descripción | Ejemplo |
|---|---|---|
| **Ida y vuelta** (*round-trip*) | Codificar y decodificar devuelve el original. | `loads(dumps(x)) == x`, `descifrar(cifrar(x)) == x` |
| **Invariante** | Algo que no cambia tras la operación. | `len(sorted(x)) == len(x)`; el total de un carrito no es negativo |
| **Idempotencia** | Aplicar dos veces = aplicar una vez. | `normalizar(normalizar(x)) == normalizar(x)` |
| **Oráculo** | Comparar con una implementación sencilla o de referencia. | `mi_ordenacion(x) == sorted(x)` |
| **Metamórfica** | Relación entre resultados de entradas relacionadas. | `buscar(q)` ⊇ `buscar(q + " filtro")`; duplicar cantidades duplica el subtotal |
| **No falla** | La función no lanza excepciones inesperadas con entradas válidas. | El analizador no se rompe con texto arbitrario |
| **Conmutatividad / asociatividad** | El orden no importa. | `fusionar(a, b) == fusionar(b, a)` |
| **Inducción** | El resultado se construye paso a paso. | `total(xs + [x]) == total(xs) + x.subtotal` |

```python
@given(st.lists(st.integers()))
def test_oraculo(xs):
    assert ordenar_por_insercion(xs) == sorted(xs)


@given(st.lists(lineas(), min_size=1), st.integers(min_value=2, max_value=5))
def test_metamorfica_multiplicar(lineas_pedido, factor):
    multiplicadas = [replace(l, cantidad=l.cantidad * factor) for l in lineas_pedido]
    assert subtotal(multiplicadas) == subtotal(lineas_pedido) * factor
```

---

## 10. Reducción de ejemplos (*shrinking*)

Cuando una prueba falla, Hypothesis busca el ejemplo **más simple** que sigue fallando:

```python
@given(st.lists(st.integers()))
def test_media_menor_que_maximo(xs):
    assert sum(xs) / len(xs) <= max(xs)
```

```text
E       ZeroDivisionError: division by zero
E       Failing test case: test_media_menor_que_maximo(
E           xs=[],
E       )
```

En lugar de una lista aleatoria de 37 números, informa del caso mínimo: la lista vacía. La reducción es automática y funciona también con estrategias compuestas, `map` y `flatmap`.

---

## 11. Control de la generación: `assume`, `example`, `note`, `target`

```python
from hypothesis import assume, example, given, note, target, strategies as st


@given(st.integers(), st.integers())
@example(0, 0)                         # Caso concreto que se prueba siempre
@example(-1, 1).via("error #42")       # Documenta de dónde sale el ejemplo
def test_division(a, b):
    assume(b != 0)                     # Descarta los ejemplos que no aplican
    note(f"cociente={a // b}, resto={a % b}")   # Se muestra solo si la prueba falla
    assert (a // b) * b + a % b == a


@given(st.lists(st.floats(0, 1000), min_size=1))
def test_tiempo_respuesta(cargas):
    tiempo = simular_cola(cargas)
    target(tiempo, label="tiempo")     # Guía la búsqueda hacia tiempos más altos
    assert tiempo < 60
```

| Función | Uso |
|---|---|
| `assume(cond)` | Descartar ejemplos que no cumplen una precondición. Úsela poco: mejor generar datos válidos. |
| `@example(...)` | Añadir casos concretos (regresiones, casos de negocio). |
| `note(texto)` | Información adicional en el informe de fallo. |
| `event(texto)` | Estadísticas de lo generado (`pytest --hypothesis-show-statistics`). |
| `target(valor)` | Búsqueda dirigida: intenta maximizar el valor. |

---

## 12. Configuración y perfiles

```python
from datetime import timedelta

from hypothesis import HealthCheck, settings


@settings(max_examples=500, deadline=timedelta(milliseconds=500))
@given(st.text())
def test_intensiva(texto): ...


@settings(deadline=None, suppress_health_check=[HealthCheck.too_slow])
@given(st.lists(st.integers()))
def test_lenta(xs): ...
```

| Ajuste | Por defecto | Uso |
|---|---|---|
| `max_examples` | 100 | Número de ejemplos que se generan. |
| `deadline` | 200 ms | Tiempo máximo por ejemplo (`None` para desactivar). |
| `derandomize` | `False` | Generación determinista (útil en CI estricta). |
| `database` | `.hypothesis/` | Dónde se guardan los ejemplos que fallaron. |
| `phases` | Todas | Fases: explícitos, reutilizados, generados, dirigidos, reducción. |
| `suppress_health_check` | `[]` | Silenciar comprobaciones de salud concretas. |
| `print_blob` | `False` | Imprime un código para reproducir el fallo. |

Perfiles por entorno (en `tests/conftest.py`):

```python
import os
from datetime import timedelta

from hypothesis import Verbosity, settings

settings.register_profile("dev", max_examples=50)
settings.register_profile("ci", max_examples=500, deadline=timedelta(seconds=1), print_blob=True)
settings.register_profile("nocturno", max_examples=10_000, deadline=None)
settings.register_profile("depurar", max_examples=10, verbosity=Verbosity.verbose)
settings.load_profile(os.getenv("HYPOTHESIS_PROFILE", "dev"))
```

```bash
HYPOTHESIS_PROFILE=ci pytest
pytest --hypothesis-profile=nocturno
pytest --hypothesis-show-statistics
pytest --hypothesis-seed=12345              # Semilla fija
```

---

## 13. Base de datos de ejemplos y reproducibilidad

- Hypothesis guarda en **`.hypothesis/`** los ejemplos que provocaron fallos y los **reintenta primero** en la siguiente ejecución. Así, un fallo no «desaparece» por azar.
- Añada `.hypothesis/` a `.gitignore`.
- En CI, la carpeta no persiste: use `print_blob=True` para obtener un código de reproducción:

```text
You can reproduce this example by temporarily adding @reproduce_failure('6.170.17', b'AXicY2BgYAQAAA4ABA==') as a decorator on your test case
```

```python
from hypothesis import reproduce_failure

@reproduce_failure("6.170.17", b"AXicY2BgYAQAAA4ABA==")
@given(...)
def test_x(...): ...
```

> Cuando encuentre un fallo, añádalo como `@example(...)` permanente: es una prueba de regresión documentada.

Compartir la base de datos entre CI y local (opcional):

```python
from hypothesis.database import DirectoryBasedExampleDatabase, GitHubArtifactDatabase, MultiplexedDatabase

local = DirectoryBasedExampleDatabase(".hypothesis/examples")
compartida = GitHubArtifactDatabase("mi-org", "mi-repo")
settings.register_profile("dev", database=MultiplexedDatabase(local, compartida))
```

---

## 14. Pruebas con estado (*stateful*)

Para sistemas con operaciones encadenadas, Hypothesis genera **secuencias de operaciones** y comprueba invariantes tras cada paso, comparando con un modelo sencillo:

```python
from hypothesis import strategies as st
from hypothesis.stateful import RuleBasedStateMachine, invariant, precondition, rule


class PruebaCache(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.cache = CacheLRU(capacidad=3)     # Sistema real
        self.modelo: dict[str, int] = {}       # Modelo de referencia (simplificado)

    @rule(clave=st.sampled_from("abcde"), valor=st.integers())
    def guardar(self, clave, valor):
        self.cache.guardar(clave, valor)
        self.modelo[clave] = valor

    @rule(clave=st.sampled_from("abcde"))
    def leer(self, clave):
        valor = self.cache.obtener(clave)
        if valor is not None:
            assert valor == self.modelo[clave]

    @precondition(lambda self: len(self.modelo) > 0)
    @rule()
    def vaciar(self):
        self.cache.vaciar()
        self.modelo.clear()

    @invariant()
    def no_supera_capacidad(self):
        assert len(self.cache) <= 3


TestCache = PruebaCache.TestCase          # pytest lo descubre como clase de pruebas
```

Si falla, Hypothesis informa de la **secuencia mínima** de operaciones que reproduce el error.

Elementos disponibles: `@rule`, `@precondition`, `@invariant`, `@initialize`, `Bundle` (para reutilizar valores creados en pasos anteriores) y `multiple()`.

---

## 15. Integración con pytest y otras librerías

- `@given` funciona con fixtures de pytest: los argumentos no generados se toman de las fixtures.

  ```python
  @given(texto=st.text())
  def test_con_fixture(tmp_path, texto):
      (tmp_path / "a.txt").write_text(texto, encoding="utf-8")
  ```

  > Las fixtures de ámbito `function` **no** se recrean entre ejemplos: use estado inmutable o límpielo dentro de la prueba.

- `@pytest.mark.parametrize` se puede combinar con `@given`.
- Pruebas asíncronas: compatible con `pytest-asyncio` y `anyio`.

Estrategias para otras librerías:

```python
from hypothesis.extra.numpy import arrays
from hypothesis.extra.pandas import column, data_frames

arrays(dtype=float, shape=(3, 3), elements=st.floats(-1, 1))
data_frames([column("precio", elements=st.floats(0, 100)), column("unidades", dtype=int)])
```

Otras integraciones: `hypothesis.extra.django`, `hypothesis-jsonschema` (datos que cumplen un JSON Schema) y **Schemathesis** (pruebas de APIs a partir de su especificación OpenAPI, construido sobre Hypothesis).

---

## 16. Generar pruebas automáticamente (*ghostwriter*)

```bash
hypothesis write json.dumps json.loads --roundtrip
hypothesis write mi_modulo.normalizar_nombre --idempotent
hypothesis write mi_modulo.funcion                 # Prueba «no falla» con estrategias deducidas
hypothesis write sorted --equivalent mi_modulo.ordenar
hypothesis write mi_modulo                         # Todo el módulo
```

Genera código de prueba como punto de partida, deduciendo estrategias de las anotaciones de tipo. Revise y ajuste siempre el resultado.

---

## 17. Ejemplo práctico completo

**Pruebas basadas en propiedades de un módulo de facturación**: cálculo de importes con `Decimal`, reparto de un importe en cuotas, serialización a texto y un carrito probado con una máquina de estados. Hypothesis encuentra un error real en el reparto de cuotas.

### 17.1 Código (`facturacion.py`)

```python
from __future__ import annotations

from dataclasses import dataclass
from decimal import ROUND_HALF_UP, Decimal

CENT = Decimal("0.01")


@dataclass(frozen=True)
class Linea:
    sku: str
    cantidad: int
    precio: Decimal

    @property
    def subtotal(self) -> Decimal:
        return (self.precio * self.cantidad).quantize(CENT, ROUND_HALF_UP)


def total(lineas: list[Linea], iva: Decimal = Decimal("0.21")) -> Decimal:
    base = sum((l.subtotal for l in lineas), start=Decimal("0"))
    return (base * (1 + iva)).quantize(CENT, ROUND_HALF_UP)


def repartir(importe: Decimal, cuotas: int) -> list[Decimal]:
    """Divide un importe en cuotas iguales (versión con error)."""
    cuota = (importe / cuotas).quantize(CENT, ROUND_HALF_UP)
    return [cuota] * cuotas


def repartir_corregido(importe: Decimal, cuotas: int) -> list[Decimal]:
    """Divide un importe en cuotas; la última absorbe la diferencia de redondeo."""
    cuota = (importe / cuotas).quantize(CENT, ROUND_HALF_UP)
    return [cuota] * (cuotas - 1) + [importe - cuota * (cuotas - 1)]


def serializar(linea: Linea) -> str:
    return f"{linea.sku}|{linea.cantidad}|{linea.precio}"


def deserializar(texto: str) -> Linea:
    sku, cantidad, precio = texto.split("|")
    return Linea(sku, int(cantidad), Decimal(precio))


class Carrito:
    def __init__(self) -> None:
        self._lineas: dict[str, Linea] = {}

    def anadir(self, sku: str, cantidad: int, precio: Decimal) -> None:
        if cantidad <= 0:
            raise ValueError("cantidad no válida")
        actual = self._lineas.get(sku)
        nueva = cantidad + (actual.cantidad if actual else 0)
        self._lineas[sku] = Linea(sku, nueva, precio)

    def quitar(self, sku: str, cantidad: int) -> None:
        actual = self._lineas[sku]
        if cantidad >= actual.cantidad:
            del self._lineas[sku]
        else:
            self._lineas[sku] = Linea(sku, actual.cantidad - cantidad, actual.precio)

    def unidades(self, sku: str) -> int:
        return self._lineas[sku].cantidad if sku in self._lineas else 0

    @property
    def lineas(self) -> list[Linea]:
        return list(self._lineas.values())
```

### 17.2 Pruebas (`test_facturacion.py`)

```python
from decimal import Decimal

import pytest
from hypothesis import example, given, settings, strategies as st
from hypothesis.stateful import RuleBasedStateMachine, invariant, precondition, rule

from facturacion import (Carrito, Linea, deserializar, repartir, repartir_corregido, serializar,
                         total)

# ------------------------------------------------------------------ Estrategias del dominio
skus = st.from_regex(r"[A-Z]{3}-\d{3}", fullmatch=True)
precios = st.decimals(min_value=Decimal("0.01"), max_value=Decimal("9999.99"), places=2)
importes = st.decimals(min_value=Decimal("0.01"), max_value=Decimal("1000000"), places=2)
cantidades = st.integers(min_value=1, max_value=1000)
lineas = st.builds(Linea, sku=skus, cantidad=cantidades, precio=precios)


# ------------------------------------------------------------------ Propiedades
@given(st.lists(lineas, max_size=20))
def test_total_no_negativo_y_con_dos_decimales(ls):
    t = total(ls)
    assert t >= 0
    assert t == t.quantize(Decimal("0.01"))


@given(st.lists(lineas, max_size=10), lineas)
def test_anadir_una_linea_no_reduce_el_total(ls, extra):              # Propiedad metamórfica
    assert total(ls + [extra]) >= total(ls)


@given(st.lists(lineas, max_size=10))
def test_total_no_depende_del_orden(ls):
    assert total(ls) == total(list(reversed(ls)))


@given(lineas)
def test_serializacion_ida_y_vuelta(linea):
    assert deserializar(serializar(linea)) == linea


@pytest.mark.xfail(reason="repartir() pierde céntimos por redondeo", strict=True)
@given(importes, st.integers(min_value=1, max_value=12))
def test_repartir_conserva_el_importe(importe, cuotas):
    assert sum(repartir(importe, cuotas)) == importe


@given(importes, st.integers(min_value=1, max_value=12))
@example(Decimal("100.00"), 3)                                         # Caso de regresión
def test_repartir_corregido(importe, cuotas):
    partes = repartir_corregido(importe, cuotas)
    assert len(partes) == cuotas
    assert sum(partes) == importe
    assert max(partes) - min(partes) <= Decimal("0.01") * cuotas


# ------------------------------------------------------------------ Máquina de estados
class MaquinaCarrito(RuleBasedStateMachine):
    def __init__(self) -> None:
        super().__init__()
        self.carrito = Carrito()
        self.modelo: dict[str, int] = {}

    @rule(sku=st.sampled_from(["TEC-001", "RAT-002", "MON-003"]), cantidad=st.integers(1, 5),
          precio=precios)
    def anadir(self, sku, cantidad, precio):
        self.carrito.anadir(sku, cantidad, precio)
        self.modelo[sku] = self.modelo.get(sku, 0) + cantidad

    @precondition(lambda self: self.modelo)
    @rule(data=st.data())
    def quitar(self, data):
        sku = data.draw(st.sampled_from(sorted(self.modelo)))
        cantidad = data.draw(st.integers(1, self.modelo[sku] + 2))
        self.carrito.quitar(sku, cantidad)
        self.modelo[sku] -= cantidad
        if self.modelo[sku] <= 0:
            del self.modelo[sku]

    @invariant()
    def unidades_coinciden_con_el_modelo(self):
        for sku in ("TEC-001", "RAT-002", "MON-003"):
            assert self.carrito.unidades(sku) == self.modelo.get(sku, 0)

    @invariant()
    def sin_lineas_vacias(self):
        assert all(l.cantidad > 0 for l in self.carrito.lineas)


MaquinaCarrito.TestCase.settings = settings(max_examples=200, stateful_step_count=30)
TestCarrito = MaquinaCarrito.TestCase
```

### 17.3 Resultado

```text
$ pytest test_facturacion.py -q
....x..                                                                  [100%]
6 passed, 1 xfailed in 2.83s
```

Sin la marca `xfail`, Hypothesis muestra el contraejemplo mínimo del error de reparto:

```text
E       AssertionError: assert Decimal('0.02') == Decimal('0.01')
E        +  where Decimal('0.02') = sum([Decimal('0.01'), Decimal('0.01')])
E        +    where [Decimal('0.01'), Decimal('0.01')] = repartir(Decimal('0.01'), 2)
E       Failing test case: test_repartir_conserva_el_importe(
E           importe=Decimal('0.01'),
E           cuotas=2,
E       )
```

Repartir un céntimo en dos cuotas produce dos cuotas de `0.01 €`, que suman `0.02 €`. Con ejemplos «redondos» (100 € en 4 cuotas) el error no aparece nunca.

---

## 18. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `FailedHealthCheck: ... filter_too_much` | `filter`/`assume` descartan casi todo. | Genere datos válidos directamente (límites, `map`, `composite`). |
| `FailedHealthCheck: too_slow` / `data_too_large` | Estrategias que generan datos enormes o lentos. | Limite tamaños (`max_size`), simplifique estrategias. |
| `DeadlineExceeded` | Un ejemplo tarda más de 200 ms. | Optimice o use `deadline=None`/mayor en esa prueba. |
| `Flaky` | La prueba falla de forma no determinista (hora, azar, estado compartido). | Elimine la dependencia del reloj/aleatoriedad o del estado entre ejemplos. |
| `function-scoped fixture` *health check* | Fixture de ámbito `function` usada con `@given`. | Asegúrese de que el estado no se comparte entre ejemplos o suprima ese aviso si es seguro. |
| La prueba pasa siempre con `floats` y falla en producción | No se consideraron `NaN`/`inf`. | Decida si deben admitirse y pruébelo explícitamente. |
| `InvalidArgument` al combinar `@given` y argumentos posicionales | Mezcla de fixtures y argumentos generados mal ordenados. | Use argumentos con nombre en `@given(x=...)`. |
| Pruebas muy lentas en CI | `max_examples` alto en todas. | Perfiles: pocos ejemplos en local, más en CI y muchos en una ejecución nocturna. |

---

## 19. Buenas prácticas

- Combine pruebas con **ejemplos concretos** (reglas de negocio) y **propiedades** (comportamientos generales).
- Empiece por las propiedades más fáciles: ida y vuelta, «no falla», invariantes simples.
- Defina **estrategias del dominio** reutilizables (`skus`, `precios`, `lineas`) en un módulo común.
- Genere datos válidos directamente; use `filter` y `assume` lo mínimo.
- Use `Decimal` y estrategias `decimals` para importes; con `floats`, decida qué hacer con `NaN` e infinitos.
- Convierta cada fallo encontrado en un `@example` de regresión.
- Configure **perfiles** (`dev`, `ci`, `nocturno`) y active `print_blob` en CI.
- Use máquinas de estado para cachés, carritos, colas, repositorios y APIs con estado.
- Añada `.hypothesis/` a `.gitignore`.

---

## 20. Referencia rápida

```python
from hypothesis import given, example, assume, note, settings, strategies as st

@given(st.integers(0, 10), texto=st.text(min_size=1))
@example(0, texto="a")
@settings(max_examples=500, deadline=None)
def test_x(n, texto): assume(...); assert ...

st.integers/floats/decimals/text/booleans/none/binary/emails/uuids/dates/datetimes
st.lists/sets/tuples/dictionaries/fixed_dictionaries(min_size=, max_size=, unique=)
st.sampled_from(seq) · st.just(x) · st.one_of(a, b) · a | b · st.from_regex(r, fullmatch=True)
.map(f) · .filter(f) · .flatmap(f) · st.builds(Clase, campo=estrategia) · st.from_type(T)
@st.composite def e(draw): x = draw(...)  ·  st.data() → data.draw(...)  ·  st.recursive(...)
RuleBasedStateMachine · @rule · @invariant · @precondition · Bundle · Maquina.TestCase
settings.register_profile("ci", ...) · settings.load_profile(...)
```

```bash
pytest --hypothesis-profile=ci --hypothesis-show-statistics --hypothesis-seed=1
hypothesis write modulo.funcion [--roundtrip | --idempotent | --equivalent otra]
```

---

## 21. Ejercicios propuestos

1. Escriba una propiedad de ida y vuelta para una función que codifica y decodifica CSV con campos que contienen `;`, comillas y saltos de línea.
2. Cree una estrategia `@st.composite` que genere NIF válidos y pruebe que su validador los acepta todos.
3. Use un oráculo para probar una implementación propia de búsqueda binaria contra `bisect`.
4. Escriba una máquina de estados para una cola con prioridad y compárela con un modelo basado en `sorted`.
5. Use `hypothesis write --roundtrip` sobre un par de funciones propias y mejore las estrategias generadas.

---

## 22. Referencias

- Documentación oficial: https://hypothesis.readthedocs.io/
- Estrategias: https://hypothesis.readthedocs.io/en/latest/reference/strategies.html
- Pruebas con estado: https://hypothesis.readthedocs.io/en/latest/stateful.html
- Ghostwriter: https://hypothesis.readthedocs.io/en/latest/reference/integrations.html#ghostwriter
- Artículos sobre propiedades: https://hypothesis.works/articles/
- Schemathesis (APIs): https://schemathesis.readthedocs.io/
