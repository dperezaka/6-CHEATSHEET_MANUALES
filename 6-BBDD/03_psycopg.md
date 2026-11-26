# Manual de uso: Psycopg

> **Categoría:** Bases de datos · **Utilidad:** Conectar con PostgreSQL
> **Tipo:** Librería externa · **Versión de referencia:** Psycopg 3 (paquete `psycopg`)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Conectar](#4-conectar)
5. [Ejecutar consultas](#5-ejecutar-consultas)
6. [Formato de las filas](#6-formato-de-las-filas)
7. [Insertar, actualizar y eliminar](#7-insertar-actualizar-y-eliminar)
8. [Transacciones](#8-transacciones)
9. [SQL dinámico seguro](#9-sql-dinámico-seguro)
10. [Tipos de datos de PostgreSQL](#10-tipos-de-datos-de-postgresql)
11. [Carga masiva con COPY](#11-carga-masiva-con-copy)
12. [Pool de conexiones](#12-pool-de-conexiones)
13. [Funcionalidades avanzadas](#13-funcionalidades-avanzadas)
14. [Psycopg 2 frente a Psycopg 3](#14-psycopg-2-frente-a-psycopg-3)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**Psycopg** es el adaptador más utilizado para conectar Python con **PostgreSQL**, una de las bases de datos relacionales de código abierto más avanzadas. Psycopg 3 (paquete `psycopg`) es la versión actual; sustituye a `psycopg2`.

**Características:**
- Implementa el estándar DB-API 2.0 de Python.
- Soporte completo de tipos de PostgreSQL: `JSONB`, arrays, `UUID`, fechas con zona horaria, rangos, `NUMERIC`…
- Carga masiva de datos con `COPY`.
- Pool de conexiones.
- API síncrona y **asíncrona** (`async`/`await`).
- Utilizado internamente por SQLAlchemy, Django y Pandas para PostgreSQL.

**Casos de uso habituales:** aplicaciones web y APIs, sistemas de gestión, almacenes de datos, procesos ETL, informes.

---

## 2. Instalación e importación

```bash
pip install "psycopg[binary]"        # Incluye la librería de cliente compilada (recomendado)
pip install "psycopg[binary,pool]"   # + pool de conexiones
```

| Opción | Uso |
|---|---|
| `psycopg[binary]` | Instalación sencilla en cualquier sistema. Recomendada para desarrollo y la mayoría de usos. |
| `psycopg[c]` | Compila la extensión en C (requiere `libpq` y compilador). Máximo rendimiento. |
| `psycopg` | Versión en Python puro; requiere `libpq` instalada en el sistema. |

```python
import psycopg
from psycopg import sql
from psycopg.rows import dict_row
```

Para practicar, puede iniciar PostgreSQL con Docker:

```bash
docker run --name pg -e POSTGRES_PASSWORD=secreto -p 5432:5432 -d postgres:16
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Connection` | Conexión con el servidor PostgreSQL. |
| `Cursor` | Ejecuta consultas y recorre resultados. |
| Marcadores `%s` / `%(nombre)s` | Parámetros de consulta (siempre `%s`, sea cual sea el tipo). |
| Transacción | Psycopg abre una transacción con la primera sentencia; se confirma con `commit()`. |
| `row_factory` | Formato de las filas (tuplas, diccionarios, clases). |
| `psycopg.sql` | Composición segura de SQL dinámico (nombres de tablas y columnas). |
| `COPY` | Mecanismo de PostgreSQL para cargar o exportar datos masivamente. |
| *Pool* | Conjunto de conexiones reutilizables. |

---

## 4. Conectar

### 4.1 Cadena de conexión

```python
conn = psycopg.connect("host=localhost port=5432 dbname=tienda user=app password=secreto")
conn = psycopg.connect("postgresql://app:secreto@localhost:5432/tienda")
```

### 4.2 Parámetros con nombre

```python
import os

conn = psycopg.connect(
    host=os.getenv("PGHOST", "localhost"),
    port=int(os.getenv("PGPORT", "5432")),
    dbname=os.environ["PGDATABASE"],
    user=os.environ["PGUSER"],
    password=os.environ["PGPASSWORD"],
    connect_timeout=10,
    application_name="informes_ventas",
)
```

> Si existen las variables de entorno estándar de PostgreSQL (`PGHOST`, `PGPORT`, `PGDATABASE`, `PGUSER`, `PGPASSWORD`), basta con `psycopg.connect()` sin argumentos.

### 4.3 Gestor de contexto (recomendado)

```python
with psycopg.connect(DSN) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT version()")
        print(cur.fetchone()[0])
# Al salir: commit si no hubo errores, rollback si los hubo, y cierre de la conexión
```

> En Psycopg 3, `with connect(...)` **sí cierra** la conexión al terminar (en psycopg2 no lo hacía).

### 4.4 Conexión segura (SSL)

```python
psycopg.connect(DSN, sslmode="require")          # verify-full para validar el certificado
```

---

## 5. Ejecutar consultas

### 5.1 Consulta con parámetros

```python
with conn.cursor() as cur:
    cur.execute("SELECT id, nombre FROM clientes WHERE ciudad = %s AND saldo > %s", ("Madrid", 100))
    for id_, nombre in cur:
        print(id_, nombre)
```

Parámetros con nombre:

```python
cur.execute(
    "SELECT * FROM clientes WHERE ciudad = %(ciudad)s AND activo = %(activo)s",
    {"ciudad": "Madrid", "activo": True},
)
```

> ⚠ El marcador es **siempre `%s`** (no `?`), aunque el valor sea numérico o fecha. Y **nunca** construya la consulta con f-strings o `%` de Python:
> ```python
> cur.execute(f"SELECT * FROM clientes WHERE nombre = '{nombre}'")   # ❌ Inyección SQL
> cur.execute("SELECT * FROM clientes WHERE nombre = %s", (nombre,))   # ✅
> ```

### 5.2 Leer resultados

```python
cur.fetchone()          # Una fila (o None)
cur.fetchmany(100)      # Hasta 100 filas
cur.fetchall()          # Todas
cur.rowcount            # Filas afectadas / devueltas
cur.description         # Metadatos de columnas
[c.name for c in cur.description]
```

### 5.3 Ejecutar directamente desde la conexión

```python
fila = conn.execute("SELECT count(*) FROM clientes").fetchone()
total = fila[0]
```

### 5.4 Cláusula IN con listas

PostgreSQL admite arrays, por lo que se usa `= ANY(%s)`:

```python
ciudades = ["Madrid", "Sevilla", "Bilbao"]
cur.execute("SELECT * FROM clientes WHERE ciudad = ANY(%s)", (ciudades,))
```

### 5.5 El carácter `%` en consultas

Si la consulta contiene `%` literal (por ejemplo, en `LIKE` escrito directamente), duplíquelo, o mejor, páselo como parámetro:

```python
cur.execute("SELECT * FROM clientes WHERE nombre ILIKE %s", (f"%{texto}%",))
```

### 5.6 Cursor en servidor (grandes resultados)

Para recorrer millones de filas sin cargarlas todas en memoria:

```python
with conn.cursor(name="cursor_grande") as cur:
    cur.itersize = 5000
    cur.execute("SELECT * FROM movimientos")
    for fila in cur:
        procesar(fila)
```

---

## 6. Formato de las filas

```python
from psycopg.rows import dict_row, namedtuple_row, class_row

# Diccionarios
with conn.cursor(row_factory=dict_row) as cur:
    cur.execute("SELECT id, nombre, email FROM clientes")
    for c in cur:
        print(c["nombre"], c["email"])

# Para toda la conexión
conn = psycopg.connect(DSN, row_factory=dict_row)

# Tuplas con nombre
with conn.cursor(row_factory=namedtuple_row) as cur:
    cur.execute("SELECT id, nombre FROM clientes")
    print(cur.fetchone().nombre)

# Clases (dataclass o Pydantic)
from dataclasses import dataclass

@dataclass
class Cliente:
    id: int
    nombre: str
    email: str

with conn.cursor(row_factory=class_row(Cliente)) as cur:
    cur.execute("SELECT id, nombre, email FROM clientes")
    clientes: list[Cliente] = cur.fetchall()
```

---

## 7. Insertar, actualizar y eliminar

### 7.1 Insertar y obtener el id (`RETURNING`)

```python
with conn.cursor() as cur:
    cur.execute(
        "INSERT INTO clientes (nombre, email, ciudad) VALUES (%s, %s, %s) RETURNING id, creado",
        ("Ana López", "ana@ejemplo.com", "Madrid"),
    )
    nuevo_id, creado = cur.fetchone()
conn.commit()
```

### 7.2 Insertar varias filas

```python
clientes = [
    ("Luis Gómez", "luis@ejemplo.com", "Sevilla"),
    ("Marta Ruiz", "marta@ejemplo.com", "Valencia"),
]
with conn.cursor() as cur:
    cur.executemany("INSERT INTO clientes (nombre, email, ciudad) VALUES (%s, %s, %s)", clientes)
conn.commit()
```

> En Psycopg 3, `executemany` usa el modo *pipeline* y es eficiente. Para decenas de miles de filas o más, use `COPY` (sección 11).

### 7.3 Insertar o actualizar (*upsert*)

```python
cur.execute("""
    INSERT INTO productos (codigo, nombre, precio, stock)
    VALUES (%s, %s, %s, %s)
    ON CONFLICT (codigo) DO UPDATE
    SET precio = EXCLUDED.precio,
        stock  = productos.stock + EXCLUDED.stock
    RETURNING id, (xmax = 0) AS insertado
""", ("A01", "Teclado", 31.50, 10))
```

> `(xmax = 0) AS insertado` indica si la fila se insertó (`True`) o se actualizó (`False`).

### 7.4 Actualizar y eliminar

```python
cur.execute("UPDATE clientes SET ciudad = %s WHERE id = %s", ("Barcelona", 1))
print(cur.rowcount, "filas actualizadas")

cur.execute("DELETE FROM clientes WHERE activo = false RETURNING id")
eliminados = [r[0] for r in cur.fetchall()]
conn.commit()
```

---

## 8. Transacciones

### 8.1 Comportamiento por defecto

- La primera sentencia inicia una transacción.
- Los cambios no son visibles para otras conexiones hasta `commit()`.
- Si una sentencia falla, la transacción queda **abortada**: hay que hacer `rollback()` antes de seguir.

```python
try:
    cur.execute("UPDATE cuentas SET saldo = saldo - %s WHERE id = %s", (100, 1))
    cur.execute("UPDATE cuentas SET saldo = saldo + %s WHERE id = %s", (100, 2))
    conn.commit()
except psycopg.Error:
    conn.rollback()
    raise
```

### 8.2 Bloques transaccionales con `conn.transaction()`

```python
with conn.transaction():
    conn.execute("UPDATE cuentas SET saldo = saldo - %s WHERE id = %s", (100, 1))
    conn.execute("UPDATE cuentas SET saldo = saldo + %s WHERE id = %s", (100, 2))
# commit al salir; rollback si hay excepción
```

Los bloques anidados crean **puntos de guardado** automáticamente:

```python
with conn.transaction():
    conn.execute("INSERT INTO pedidos ...")
    for linea in lineas:
        try:
            with conn.transaction():                 # SAVEPOINT
                conn.execute("INSERT INTO lineas ...", linea)
        except psycopg.errors.CheckViolation:
            registrar_error(linea)                   # Solo se deshace esta línea
```

### 8.3 Modo *autocommit*

```python
conn = psycopg.connect(DSN, autocommit=True)
conn.execute("CREATE DATABASE informes")       # Algunas órdenes no pueden ir en transacción
conn.execute("VACUUM ANALYZE clientes")
```

### 8.4 Nivel de aislamiento y bloqueo de filas

```python
from psycopg import IsolationLevel
conn.isolation_level = IsolationLevel.SERIALIZABLE

with conn.transaction():
    cur.execute("SELECT stock FROM productos WHERE id = %s FOR UPDATE", (5,))    # Bloquea la fila
    ...
```

---

## 9. SQL dinámico seguro

Los parámetros `%s` solo sirven para **valores**. Para nombres de tablas, columnas u órdenes dinámicos use el módulo `psycopg.sql`:

```python
from psycopg import sql

tabla = "clientes"
columnas = ["nombre", "email", "ciudad"]

consulta = sql.SQL("SELECT {campos} FROM {tabla} WHERE {filtro} = %s ORDER BY {orden}").format(
    campos=sql.SQL(", ").join(map(sql.Identifier, columnas)),
    tabla=sql.Identifier(tabla),
    filtro=sql.Identifier("ciudad"),
    orden=sql.Identifier("nombre"),
)
cur.execute(consulta, ("Madrid",))
print(consulta.as_string(conn))
# SELECT "nombre", "email", "ciudad" FROM "clientes" WHERE "ciudad" = %s ORDER BY "nombre"
```

Inserción genérica para cualquier tabla:

```python
def insertar(conn, tabla: str, datos: dict) -> None:
    consulta = sql.SQL("INSERT INTO {} ({}) VALUES ({})").format(
        sql.Identifier(tabla),
        sql.SQL(", ").join(map(sql.Identifier, datos)),
        sql.SQL(", ").join(sql.Placeholder() * len(datos)),
    )
    conn.execute(consulta, list(datos.values()))
```

| Elemento | Uso |
|---|---|
| `sql.SQL("...")` | Fragmento SQL fijo. |
| `sql.Identifier("nombre")` | Nombre de tabla/columna (entrecomillado de forma segura). |
| `sql.Identifier("esquema", "tabla")` | Nombre cualificado `"esquema"."tabla"`. |
| `sql.Literal(valor)` | Valor literal incrustado (prefiera parámetros). |
| `sql.Placeholder()` | Marcador `%s`. |

---

## 10. Tipos de datos de PostgreSQL

### 10.1 Correspondencia

| PostgreSQL | Python |
|---|---|
| `integer`, `bigint`, `smallint` | `int` |
| `numeric` / `decimal` | `Decimal` |
| `real`, `double precision` | `float` |
| `text`, `varchar` | `str` |
| `boolean` | `bool` |
| `date` | `datetime.date` |
| `timestamp` | `datetime.datetime` (sin zona) |
| `timestamptz` | `datetime.datetime` (con zona) |
| `interval` | `datetime.timedelta` |
| `uuid` | `uuid.UUID` |
| `json` / `jsonb` | `dict` / `list` |
| `bytea` | `bytes` |
| `type[]` (arrays) | `list` |
| `inet` | `ipaddress` |

### 10.2 JSONB

```python
from psycopg.types.json import Jsonb

conn.execute("CREATE TABLE IF NOT EXISTS eventos (id serial PRIMARY KEY, datos jsonb)")
conn.execute("INSERT INTO eventos (datos) VALUES (%s)",
             (Jsonb({"tipo": "compra", "usuario": "ana", "importe": 59.8, "tags": ["web", "promo"]}),))

cur.execute("""
    SELECT datos->>'usuario' AS usuario, (datos->>'importe')::numeric AS importe
    FROM eventos
    WHERE datos @> %s
""", (Jsonb({"tipo": "compra"}),))
```

> Para escribir JSON, envuelva el diccionario con `Jsonb(...)` (o `Json(...)`). Al leer, Psycopg devuelve directamente `dict`/`list`.

Operadores JSONB útiles: `->` (devuelve JSON), `->>` (devuelve texto), `@>` (contiene), `?` (tiene la clave), `jsonb_array_elements`, `jsonb_set`. Cree un índice `GIN` para búsquedas rápidas:

```sql
CREATE INDEX ix_eventos_datos ON eventos USING GIN (datos);
```

### 10.3 Arrays

```python
conn.execute("INSERT INTO productos (nombre, etiquetas) VALUES (%s, %s)", ("Teclado", ["oferta", "nuevo"]))
cur.execute("SELECT nombre FROM productos WHERE %s = ANY(etiquetas)", ("oferta",))
cur.execute("SELECT nombre FROM productos WHERE etiquetas && %s", (["oferta", "liquidacion"],))   # Intersección
```

### 10.4 Fechas y zonas horarias

```python
from datetime import datetime
from zoneinfo import ZoneInfo

conn.execute("SET TIME ZONE 'Europe/Madrid'")
ahora = datetime.now(ZoneInfo("Europe/Madrid"))
conn.execute("INSERT INTO accesos (usuario, momento) VALUES (%s, %s)", ("ana", ahora))
```

> Use `timestamptz` para registrar instantes: PostgreSQL los guarda en UTC y los convierte a la zona de la sesión.

### 10.5 UUID

```python
import uuid
conn.execute("INSERT INTO sesiones (id, usuario) VALUES (%s, %s)", (uuid.uuid4(), "ana"))
```

O generado por PostgreSQL: `id uuid PRIMARY KEY DEFAULT gen_random_uuid()`.

---

## 11. Carga masiva con COPY

`COPY` es la forma más rápida de cargar o exportar grandes volúmenes (decenas o cientos de veces más rápido que `INSERT`).

### 11.1 Cargar filas desde Python

```python
filas = [(f"Cliente {i}", f"c{i}@ejemplo.com", "Madrid") for i in range(100_000)]

with conn.cursor() as cur:
    with cur.copy("COPY clientes (nombre, email, ciudad) FROM STDIN") as copia:
        for fila in filas:
            copia.write_row(fila)
conn.commit()
```

### 11.2 Cargar un archivo CSV

```python
with conn.cursor() as cur, open("clientes.csv", "rb") as f:
    with cur.copy("COPY clientes (nombre, email, ciudad) FROM STDIN WITH (FORMAT csv, HEADER true, DELIMITER ';')") as copia:
        while datos := f.read(1024 * 1024):
            copia.write(datos)
conn.commit()
```

### 11.3 Exportar a CSV

```python
with conn.cursor() as cur, open("export.csv", "wb") as f:
    with cur.copy("COPY (SELECT * FROM clientes ORDER BY id) TO STDOUT WITH (FORMAT csv, HEADER true)") as copia:
        for bloque in copia:
            f.write(bloque)
```

### 11.4 Leer filas con COPY

```python
with cur.copy("COPY (SELECT id, nombre FROM clientes) TO STDOUT") as copia:
    copia.set_types(["int4", "text"])
    for fila in copia.rows():
        print(fila)
```

### 11.5 Patrón de carga con tabla temporal (*upsert* masivo)

```python
with conn.transaction():
    conn.execute("CREATE TEMP TABLE tmp_productos (LIKE productos INCLUDING DEFAULTS) ON COMMIT DROP")
    with conn.cursor().copy("COPY tmp_productos (codigo, nombre, precio, stock) FROM STDIN") as copia:
        for fila in datos:
            copia.write_row(fila)
    conn.execute("""
        INSERT INTO productos (codigo, nombre, precio, stock)
        SELECT codigo, nombre, precio, stock FROM tmp_productos
        ON CONFLICT (codigo) DO UPDATE SET precio = EXCLUDED.precio, stock = EXCLUDED.stock
    """)
```

---

## 12. Pool de conexiones

Abrir una conexión con PostgreSQL es costoso. En aplicaciones con muchas peticiones (APIs, servicios), reutilice conexiones con un *pool*:

```bash
pip install "psycopg[pool]"
```

```python
from psycopg_pool import ConnectionPool

pool = ConnectionPool(DSN, min_size=2, max_size=10, open=True, kwargs={"row_factory": dict_row})

def obtener_cliente(id_: int) -> dict | None:
    with pool.connection() as conn:              # Toma una conexión y la devuelve al salir
        return conn.execute("SELECT * FROM clientes WHERE id = %s", (id_,)).fetchone()

pool.close()                                     # Al terminar la aplicación
```

Uso con FastAPI:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from psycopg_pool import AsyncConnectionPool

pool = AsyncConnectionPool(DSN, open=False)

@asynccontextmanager
async def ciclo_vida(app):
    await pool.open()
    yield
    await pool.close()

app = FastAPI(lifespan=ciclo_vida)

@app.get("/clientes/{id}")
async def cliente(id: int):
    async with pool.connection() as conn:
        cur = await conn.execute("SELECT id, nombre FROM clientes WHERE id = %s", (id,))
        return await cur.fetchone()
```

---

## 13. Funcionalidades avanzadas

### 13.1 API asíncrona

```python
import asyncio

async def main():
    async with await psycopg.AsyncConnection.connect(DSN) as conn:
        async with conn.cursor() as cur:
            await cur.execute("SELECT nombre FROM clientes LIMIT 5")
            async for fila in cur:
                print(fila)

asyncio.run(main())
```

### 13.2 Modo *pipeline* (muchas sentencias pequeñas)

```python
with conn.pipeline():
    for item in elementos:
        conn.execute("UPDATE productos SET stock = %s WHERE codigo = %s", (item.stock, item.codigo))
```

Envía las sentencias sin esperar cada respuesta: muy útil con servidores remotos.

### 13.3 Notificaciones (LISTEN / NOTIFY)

```python
# Proceso que escucha
conn = psycopg.connect(DSN, autocommit=True)
conn.execute("LISTEN nuevos_pedidos")
for aviso in conn.notifies():
    print("Nuevo pedido:", aviso.payload)

# Otro proceso (o un disparador) notifica
conn.execute("SELECT pg_notify('nuevos_pedidos', %s)", ("1045",))
```

### 13.4 Llamar a funciones y procedimientos

```python
cur.execute("SELECT calcular_descuento(%s, %s)", (cliente_id, importe))
conn.execute("CALL cerrar_mes(%s)", ("2026-11",))
```

### 13.5 Gestión de errores específicos

```python
from psycopg import errors

try:
    conn.execute("INSERT INTO clientes (email) VALUES (%s)", ("ana@ejemplo.com",))
    conn.commit()
except errors.UniqueViolation:
    conn.rollback()
    print("El email ya existe")
except errors.ForeignKeyViolation as e:
    conn.rollback()
    print("Referencia no válida:", e.diag.message_detail)
except errors.NotNullViolation as e:
    conn.rollback()
    print("Falta el campo:", e.diag.column_name)
except psycopg.OperationalError:
    print("Problema de conexión con el servidor")
```

### 13.6 Integración con Pandas y SQLAlchemy

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://app:secreto@localhost/tienda")
df = pd.read_sql("SELECT * FROM clientes", engine)
df.to_sql("clientes_copia", engine, if_exists="replace", index=False, method="multi", chunksize=1000)
```

> Pandas recomienda pasar un `Engine` de SQLAlchemy en lugar de una conexión de Psycopg.

---

## 14. Psycopg 2 frente a Psycopg 3

| Aspecto | psycopg2 | psycopg (3) |
|---|---|---|
| Paquete | `pip install psycopg2-binary` | `pip install "psycopg[binary]"` |
| Importación | `import psycopg2` | `import psycopg` |
| `with connect()` | Gestiona la transacción pero **no cierra** | Gestiona la transacción **y cierra** |
| `conn.execute()` | No existe | Disponible |
| Filas como dict | `RealDictCursor` | `row_factory=dict_row` |
| Carga masiva | `copy_expert`, `execute_values` | `cursor.copy()` |
| Asíncrono | No (solo con librerías externas) | Nativo |
| Parámetros | Enlazados en el cliente | En el servidor (más seguro y eficiente) |
| URL SQLAlchemy | `postgresql+psycopg2://` | `postgresql+psycopg://` |

---

## 15. Ejemplo práctico completo

Proceso ETL de ventas: crea el esquema, carga datos de ejemplo con `COPY`, realiza un *upsert* masivo de productos con tabla temporal, registra pedidos en transacciones con control de stock y bloqueo de filas, genera un informe con funciones de ventana y exporta a CSV.

```python
"""
ETL e informe de ventas con Psycopg 3.
Requisitos: pip install "psycopg[binary]"
Variable de entorno: DATABASE_URL=postgresql://usuario:clave@localhost:5432/tienda
"""
import csv
import io
import os
import random
from datetime import date, timedelta
from decimal import Decimal

import psycopg
from psycopg import errors
from psycopg.rows import dict_row
from psycopg.types.json import Jsonb

DSN = os.getenv("DATABASE_URL", "postgresql://app:app@localhost:5432/tienda")

ESQUEMA = """
DROP TABLE IF EXISTS lineas_pedido, pedidos, productos, clientes CASCADE;

CREATE TABLE clientes (
    id      serial PRIMARY KEY,
    nombre  text NOT NULL,
    email   text NOT NULL UNIQUE,
    ciudad  text NOT NULL,
    alta    date NOT NULL DEFAULT current_date
);

CREATE TABLE productos (
    id        serial PRIMARY KEY,
    codigo    text NOT NULL UNIQUE,
    nombre    text NOT NULL,
    categoria text NOT NULL,
    precio    numeric(10,2) NOT NULL CHECK (precio >= 0),
    stock     integer NOT NULL DEFAULT 0 CHECK (stock >= 0),
    atributos jsonb NOT NULL DEFAULT '{}'
);

CREATE TABLE pedidos (
    id         serial PRIMARY KEY,
    cliente_id integer NOT NULL REFERENCES clientes(id),
    fecha      date NOT NULL DEFAULT current_date,
    canal      text NOT NULL CHECK (canal IN ('web', 'tienda', 'telefono'))
);

CREATE TABLE lineas_pedido (
    pedido_id   integer REFERENCES pedidos(id) ON DELETE CASCADE,
    producto_id integer REFERENCES productos(id),
    cantidad    integer NOT NULL CHECK (cantidad > 0),
    precio      numeric(10,2) NOT NULL,
    PRIMARY KEY (pedido_id, producto_id)
);

CREATE INDEX ix_pedidos_fecha ON pedidos(fecha);
CREATE INDEX ix_productos_atributos ON productos USING GIN (atributos);
"""

CATALOGO = [
    ("TEC-01", "Teclado mecánico", "Periféricos", Decimal("59.90"), 120, {"color": "negro", "inalambrico": False}),
    ("RAT-01", "Ratón inalámbrico", "Periféricos", Decimal("24.50"), 200, {"color": "gris", "inalambrico": True}),
    ("MON-27", "Monitor 27\"", "Pantallas", Decimal("219.00"), 35, {"pulgadas": 27, "hz": 144}),
    ("POR-14", "Portátil 14\"", "Equipos", Decimal("899.00"), 20, {"ram_gb": 16, "ssd_gb": 512}),
    ("CAB-HD", "Cable HDMI 2 m", "Accesorios", Decimal("9.90"), 600, {"longitud_m": 2}),
    ("AUR-BT", "Auriculares BT", "Audio", Decimal("79.00"), 60, {"inalambrico": True}),
]


class StockInsuficiente(Exception):
    pass


def euros(v) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# ------------------------------------------------------------------ Carga
def crear_esquema(conn: psycopg.Connection) -> None:
    with conn.transaction():
        conn.execute(ESQUEMA)


def cargar_clientes_copy(conn: psycopg.Connection, n: int = 500) -> None:
    ciudades = ["Madrid", "Barcelona", "Sevilla", "Valencia", "Bilbao", "Zaragoza"]
    random.seed(7)
    with conn.transaction(), conn.cursor() as cur:
        with cur.copy("COPY clientes (nombre, email, ciudad, alta) FROM STDIN") as copia:
            for i in range(1, n + 1):
                copia.write_row((f"Cliente {i:04d}", f"cliente{i}@ejemplo.com", random.choice(ciudades),
                                 date(2025, 1, 1) + timedelta(days=random.randint(0, 600))))


def upsert_productos(conn: psycopg.Connection, catalogo: list[tuple]) -> tuple[int, int]:
    with conn.transaction(), conn.cursor() as cur:
        cur.execute("CREATE TEMP TABLE tmp_productos (LIKE productos INCLUDING DEFAULTS) ON COMMIT DROP")
        with cur.copy("COPY tmp_productos (codigo, nombre, categoria, precio, stock, atributos) FROM STDIN") as copia:
            for codigo, nombre, cat, precio, stock, attrs in catalogo:
                copia.write_row((codigo, nombre, cat, precio, stock, Jsonb(attrs)))
        cur.execute("""
            INSERT INTO productos (codigo, nombre, categoria, precio, stock, atributos)
            SELECT codigo, nombre, categoria, precio, stock, atributos FROM tmp_productos
            ON CONFLICT (codigo) DO UPDATE
               SET nombre = EXCLUDED.nombre, precio = EXCLUDED.precio,
                   stock = EXCLUDED.stock, atributos = EXCLUDED.atributos
            RETURNING (xmax = 0) AS insertado
        """)
        resultados = [r[0] for r in cur.fetchall()]
    return resultados.count(True), resultados.count(False)


def registrar_pedido(conn: psycopg.Connection, cliente_id: int, articulos: dict[str, int],
                     canal: str, fecha: date) -> int:
    """Crea un pedido descontando stock de forma atómica."""
    with conn.transaction(), conn.cursor() as cur:
        cur.execute("INSERT INTO pedidos (cliente_id, canal, fecha) VALUES (%s, %s, %s) RETURNING id",
                    (cliente_id, canal, fecha))
        pedido_id = cur.fetchone()[0]
        for codigo, cantidad in articulos.items():
            cur.execute("SELECT id, precio, stock FROM productos WHERE codigo = %s FOR UPDATE", (codigo,))
            fila = cur.fetchone()
            if fila is None or fila[2] < cantidad:
                raise StockInsuficiente(f"{codigo}: solicitado {cantidad}, disponible {fila[2] if fila else 0}")
            producto_id, precio, _ = fila
            cur.execute("UPDATE productos SET stock = stock - %s WHERE id = %s", (cantidad, producto_id))
            cur.execute("INSERT INTO lineas_pedido VALUES (%s, %s, %s, %s)",
                        (pedido_id, producto_id, cantidad, precio))
        return pedido_id


def simular_pedidos(conn: psycopg.Connection, n: int = 400) -> tuple[int, int]:
    random.seed(11)
    codigos = [c[0] for c in CATALOGO]
    ok = rechazados = 0
    for _ in range(n):
        articulos = {random.choice(codigos): random.randint(1, 3) for _ in range(random.randint(1, 3))}
        try:
            registrar_pedido(conn, random.randint(1, 500), articulos,
                             random.choice(["web", "web", "tienda", "telefono"]),
                             date(2026, 11, 24) - timedelta(days=random.randint(0, 180)))
            ok += 1
        except StockInsuficiente:
            rechazados += 1
    return ok, rechazados


# ------------------------------------------------------------------ Informes
def informe_mensual(conn: psycopg.Connection) -> list[dict]:
    with conn.cursor(row_factory=dict_row) as cur:
        cur.execute("""
            WITH mensual AS (
                SELECT date_trunc('month', p.fecha)::date AS mes,
                       sum(l.cantidad * l.precio)         AS ventas,
                       count(DISTINCT p.id)               AS pedidos
                FROM pedidos p JOIN lineas_pedido l ON l.pedido_id = p.id
                GROUP BY 1
            )
            SELECT to_char(mes, 'YYYY-MM') AS mes, pedidos, ventas,
                   round(100.0 * (ventas / lag(ventas) OVER (ORDER BY mes) - 1), 1) AS variacion_pct,
                   sum(ventas) OVER (ORDER BY mes) AS acumulado
            FROM mensual ORDER BY mes
        """)
        return cur.fetchall()


def top_categoria_por_ciudad(conn: psycopg.Connection) -> list[dict]:
    with conn.cursor(row_factory=dict_row) as cur:
        cur.execute("""
            SELECT ciudad, categoria, ventas FROM (
                SELECT c.ciudad, pr.categoria, sum(l.cantidad * l.precio) AS ventas,
                       rank() OVER (PARTITION BY c.ciudad ORDER BY sum(l.cantidad * l.precio) DESC) AS puesto
                FROM pedidos p
                JOIN clientes c ON c.id = p.cliente_id
                JOIN lineas_pedido l ON l.pedido_id = p.id
                JOIN productos pr ON pr.id = l.producto_id
                GROUP BY c.ciudad, pr.categoria
            ) t WHERE puesto = 1 ORDER BY ventas DESC
        """)
        return cur.fetchall()


def productos_inalambricos(conn: psycopg.Connection) -> list[tuple]:
    return conn.execute("SELECT codigo, nombre, stock FROM productos WHERE atributos @> %s ORDER BY codigo",
                        (Jsonb({"inalambrico": True}),)).fetchall()


def exportar_csv(conn: psycopg.Connection, ruta: str) -> None:
    with conn.cursor() as cur, open(ruta, "wb") as f:
        with cur.copy("""
            COPY (SELECT p.id, p.fecha, c.nombre AS cliente, c.ciudad, p.canal,
                         sum(l.cantidad * l.precio) AS total
                  FROM pedidos p JOIN clientes c ON c.id = p.cliente_id
                  JOIN lineas_pedido l ON l.pedido_id = p.id
                  GROUP BY p.id, c.nombre, c.ciudad ORDER BY p.fecha)
            TO STDOUT WITH (FORMAT csv, HEADER true, DELIMITER ';')
        """) as copia:
            for bloque in copia:
                f.write(bloque)


# ------------------------------------------------------------------ Principal
def main() -> None:
    with psycopg.connect(DSN, application_name="etl_ventas") as conn:
        crear_esquema(conn)
        cargar_clientes_copy(conn)
        nuevos, actualizados = upsert_productos(conn, CATALOGO)
        print(f"Productos: {nuevos} nuevos, {actualizados} actualizados")

        CATALOGO[0] = ("TEC-01", "Teclado mecánico RGB", "Periféricos", Decimal("64.90"), 150, {"color": "negro"})
        nuevos, actualizados = upsert_productos(conn, CATALOGO[:1])
        print(f"Re-sincronización: {nuevos} nuevos, {actualizados} actualizados")

        ok, rechazados = simular_pedidos(conn)
        print(f"Pedidos: {ok} registrados, {rechazados} rechazados por falta de stock")

        try:
            with conn.transaction():
                conn.execute("INSERT INTO clientes (nombre, email, ciudad) VALUES (%s, %s, %s)",
                             ("Duplicado", "cliente1@ejemplo.com", "Madrid"))
        except errors.UniqueViolation:
            print("Control de duplicados: email ya registrado ✔")

        print(f"\n{'Mes':<8} {'Pedidos':>8} {'Ventas':>14} {'Var. %':>8} {'Acumulado':>14}")
        for r in informe_mensual(conn):
            var = f"{r['variacion_pct']:+.1f}" if r["variacion_pct"] is not None else "—"
            print(f"{r['mes']:<8} {r['pedidos']:>8} {euros(r['ventas']):>14} {var:>8} {euros(r['acumulado']):>14}")

        print("\nCategoría líder por ciudad:")
        for r in top_categoria_por_ciudad(conn):
            print(f"  {r['ciudad']:<10} {r['categoria']:<12} {euros(r['ventas']):>12}")

        print("\nProductos inalámbricos (búsqueda JSONB):")
        for codigo, nombre, stock in productos_inalambricos(conn):
            print(f"  {codigo:<7} {nombre:<20} stock {stock}")

        exportar_csv(conn, "pedidos_export.csv")
        print("\nExportado: pedidos_export.csv")


if __name__ == "__main__":
    main()
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `psycopg.OperationalError: connection failed ... Connection refused` | Servidor parado, host o puerto incorrectos, o cortafuegos. | Compruebe el servicio y los parámetros de conexión. |
| `password authentication failed for user` | Usuario o contraseña incorrectos. | Revise las credenciales y `pg_hba.conf`. |
| `ImportError: no pq wrapper available` | Falta la librería `libpq`. | Instale `psycopg[binary]`. |
| `InFailedSqlTransaction: current transaction is aborted` | Se siguió ejecutando tras un error. | Llame a `conn.rollback()` o use `conn.transaction()`. |
| `ProgrammingError: query has N placeholders but M parameters were passed` | Número de `%s` y valores distinto, o parámetro único sin tupla. | Use `(valor,)` con coma; revise los marcadores. |
| Error de sintaxis con `?` | Se usó el marcador de SQLite. | En Psycopg, siempre `%s`. |
| `UniqueViolation`, `ForeignKeyViolation`, `CheckViolation` | Restricciones del esquema. | Capture `psycopg.errors.*` y haga `rollback`. |
| `TypeError` al insertar un `dict` en `jsonb` | El diccionario no se adapta automáticamente. | Envuélvalo con `Jsonb(datos)`. |
| `CREATE DATABASE cannot run inside a transaction block` | Orden incompatible con transacciones. | Use `autocommit=True`. |
| Los cambios no aparecen en la BD | Falta `commit()`. | Use `with connect(...)`, `conn.transaction()` o `commit()`. |
| `too many clients already` | Demasiadas conexiones abiertas. | Use un pool y cierre las conexiones. |

---

## 17. Buenas prácticas

- Use `psycopg` (versión 3) en proyectos nuevos.
- Pase **siempre** los valores como parámetros `%s`; use `psycopg.sql` para identificadores dinámicos.
- Guarde las credenciales en variables de entorno (`PG*` o `DATABASE_URL`).
- Use gestores de contexto para conexiones, cursores y transacciones.
- Agrupe las operaciones relacionadas en `conn.transaction()`.
- Use `COPY` para cargas masivas y cursores en servidor para lecturas muy grandes.
- Use un **pool** en aplicaciones con muchas peticiones.
- Aproveche los tipos de PostgreSQL (`numeric`, `timestamptz`, `jsonb`, arrays) y las restricciones del esquema.
- Capture las excepciones específicas de `psycopg.errors`.
- Para modelos de datos complejos, combine Psycopg con **SQLAlchemy**.

---

## 18. Referencia rápida

```python
conn = psycopg.connect("postgresql://u:c@host:5432/bd", row_factory=dict_row, autocommit=False)
with psycopg.connect(DSN) as conn, conn.cursor() as cur: ...

cur.execute("... WHERE a = %s AND b = %(b)s", params)   # Siempre %s
cur.executemany(sql, filas) / cur.fetchone() / fetchmany(n) / fetchall() / cur.rowcount
conn.execute(sql, params).fetchall()
conn.commit() / conn.rollback() / with conn.transaction(): ...

from psycopg.rows import dict_row, namedtuple_row, class_row
from psycopg import sql; sql.SQL("... {}").format(sql.Identifier("tabla"))
from psycopg.types.json import Jsonb
with cur.copy("COPY t (a, b) FROM STDIN") as c: c.write_row((1, "x"))
with conn.cursor(name="servidor") as cur: ...            # Cursor en servidor
from psycopg_pool import ConnectionPool; pool.connection()
psycopg.errors.UniqueViolation / ForeignKeyViolation / CheckViolation / OperationalError
```

---

## 19. Ejercicios propuestos

1. Conecte con PostgreSQL usando variables de entorno y muestre la versión del servidor y las tablas del esquema `public`.
2. Cargue un CSV de 1 millón de filas con `COPY` y compare el tiempo con `executemany`.
3. Cree una tabla con una columna `jsonb` y escriba consultas que filtren por claves anidadas usando un índice GIN.
4. Implemente una transferencia entre cuentas con bloqueo de filas que nunca deje saldos negativos.
5. Cree un pequeño servicio que escuche notificaciones `LISTEN/NOTIFY` y muestre en consola los nuevos pedidos.

---

## 20. Referencias

- Documentación oficial (Psycopg 3): https://www.psycopg.org/psycopg3/docs/
- Pool de conexiones: https://www.psycopg.org/psycopg3/docs/advanced/pool.html
- Documentación de PostgreSQL: https://www.postgresql.org/docs/
