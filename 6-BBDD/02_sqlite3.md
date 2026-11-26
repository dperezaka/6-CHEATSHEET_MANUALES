# Manual de uso: SQLite3

> **Categoría:** Bases de datos · **Utilidad:** Trabajar con bases de datos SQLite
> **Tipo:** Librería estándar de Python (no requiere instalación)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Conectar y crear tablas](#4-conectar-y-crear-tablas)
5. [Insertar datos](#5-insertar-datos)
6. [Consultar datos](#6-consultar-datos)
7. [Actualizar y eliminar](#7-actualizar-y-eliminar)
8. [Transacciones](#8-transacciones)
9. [Tipos de datos y conversiones](#9-tipos-de-datos-y-conversiones)
10. [Funcionalidades de SQLite](#10-funcionalidades-de-sqlite)
11. [Rendimiento y concurrencia](#11-rendimiento-y-concurrencia)
12. [Copias de seguridad, importación y exportación](#12-copias-de-seguridad-importación-y-exportación)
13. [Ejemplo práctico completo](#13-ejemplo-práctico-completo)
14. [Errores frecuentes](#14-errores-frecuentes)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Referencia rápida](#16-referencia-rápida)
17. [Ejercicios propuestos](#17-ejercicios-propuestos)
18. [Referencias](#18-referencias)

---

## 1. Descripción

**SQLite** es un motor de base de datos SQL completo que guarda toda la base de datos en **un único archivo** (`.db`, `.sqlite`). No necesita servidor, instalación ni configuración. Python lo incluye a través del módulo estándar `sqlite3`.

**Casos de uso habituales:**
- Aplicaciones de escritorio y scripts que necesitan guardar datos estructurados.
- Prototipos y desarrollo antes de pasar a PostgreSQL/MySQL.
- Almacenamiento local de configuraciones, cachés y registros.
- Análisis de datos de tamaño medio con SQL.
- Intercambio de datos (un solo archivo portátil).
- Pruebas automáticas (base de datos en memoria).

**Cuándo NO usar SQLite:**
- Muchos usuarios escribiendo simultáneamente (aplicaciones web con mucho tráfico).
- Bases de datos compartidas por red entre varios equipos.
- Volúmenes muy grandes con necesidades de alta concurrencia.

---

## 2. Instalación e importación

No requiere instalación.

```python
import sqlite3

print(sqlite3.sqlite_version)       # Versión del motor SQLite
```

Herramientas recomendadas para explorar archivos `.db`: **DB Browser for SQLite**, **DBeaver**, la extensión *SQLite Viewer* de VS Code, o la consola `sqlite3`.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Connection` | Conexión con el archivo de base de datos (`sqlite3.connect`). |
| `Cursor` | Objeto que ejecuta sentencias y recorre resultados. |
| `execute()` | Ejecuta una sentencia SQL. |
| `executemany()` | Ejecuta la misma sentencia con muchos conjuntos de datos. |
| `commit()` | Confirma los cambios. |
| `rollback()` | Deshace los cambios no confirmados. |
| Parámetros | Marcadores `?` o `:nombre` para insertar valores de forma segura. |
| `row_factory` | Define cómo se devuelven las filas (tuplas, diccionarios…). |

**Flujo típico:** conectar → ejecutar → (confirmar) → leer resultados → cerrar.

---

## 4. Conectar y crear tablas

### 4.1 Conectar

```python
conn = sqlite3.connect("empresa.db")        # Crea el archivo si no existe
conn = sqlite3.connect(":memory:")          # Base de datos temporal en memoria
```

Con ruta basada en la ubicación del script:

```python
from pathlib import Path
RUTA_BD = Path(__file__).resolve().parent / "datos" / "empresa.db"
RUTA_BD.parent.mkdir(exist_ok=True)
conn = sqlite3.connect(RUTA_BD)
```

### 4.2 Cerrar

```python
conn.close()
```

> El gestor de contexto `with conn:` **no cierra** la conexión: gestiona la transacción (confirma o deshace). Para cerrar automáticamente:

```python
from contextlib import closing

with closing(sqlite3.connect("empresa.db")) as conn:
    with conn:                    # Transacción
        conn.execute(...)
```

### 4.3 Crear tablas

```python
conn.execute("""
    CREATE TABLE IF NOT EXISTS clientes (
        id          INTEGER PRIMARY KEY AUTOINCREMENT,
        nombre      TEXT    NOT NULL,
        email       TEXT    NOT NULL UNIQUE,
        ciudad      TEXT,
        saldo       REAL    NOT NULL DEFAULT 0 CHECK (saldo >= 0),
        activo      INTEGER NOT NULL DEFAULT 1,
        creado      TEXT    NOT NULL DEFAULT (datetime('now', 'localtime'))
    )
""")
conn.commit()
```

### 4.4 Varias sentencias a la vez

```python
conn.executescript("""
    CREATE TABLE IF NOT EXISTS productos (
        id INTEGER PRIMARY KEY,
        codigo TEXT UNIQUE NOT NULL,
        nombre TEXT NOT NULL,
        precio REAL NOT NULL CHECK (precio >= 0),
        stock INTEGER NOT NULL DEFAULT 0
    );

    CREATE TABLE IF NOT EXISTS pedidos (
        id INTEGER PRIMARY KEY,
        cliente_id INTEGER NOT NULL REFERENCES clientes(id) ON DELETE CASCADE,
        fecha TEXT NOT NULL DEFAULT (date('now')),
        total REAL NOT NULL DEFAULT 0
    );

    CREATE INDEX IF NOT EXISTS ix_pedidos_cliente ON pedidos(cliente_id);
    CREATE INDEX IF NOT EXISTS ix_pedidos_fecha ON pedidos(fecha);
""")
```

> `executescript` confirma automáticamente cualquier transacción pendiente antes de ejecutarse y **no admite parámetros**. Úselo solo para esquemas fijos.

### 4.5 Activar claves foráneas

SQLite **no aplica** las claves foráneas por defecto. Actívelas en cada conexión:

```python
conn.execute("PRAGMA foreign_keys = ON")
```

### 4.6 Consultar el esquema

```python
tablas = conn.execute("SELECT name FROM sqlite_master WHERE type = 'table'").fetchall()
columnas = conn.execute("PRAGMA table_info(clientes)").fetchall()
indices = conn.execute("PRAGMA index_list(pedidos)").fetchall()
```

### 4.7 Modificar tablas

```python
conn.execute("ALTER TABLE clientes ADD COLUMN telefono TEXT")
conn.execute("ALTER TABLE clientes RENAME COLUMN ciudad TO localidad")
conn.execute("ALTER TABLE clientes DROP COLUMN telefono")      # SQLite 3.35+
conn.execute("DROP TABLE IF EXISTS temporal")
```

---

## 5. Insertar datos

### 5.1 Con parámetros posicionales (`?`)

```python
conn.execute(
    "INSERT INTO clientes (nombre, email, ciudad) VALUES (?, ?, ?)",
    ("Ana López", "ana@ejemplo.com", "Madrid"),
)
conn.commit()
```

### 5.2 Con parámetros con nombre (`:nombre`)

```python
cliente = {"nombre": "Luis Gómez", "email": "luis@ejemplo.com", "ciudad": "Sevilla"}
conn.execute("INSERT INTO clientes (nombre, email, ciudad) VALUES (:nombre, :email, :ciudad)", cliente)
conn.commit()
```

> ⚠ **Nunca** construya el SQL con f-strings o concatenación:
> ```python
> conn.execute(f"SELECT * FROM clientes WHERE nombre = '{nombre}'")   # ❌ Inyección SQL
> conn.execute("SELECT * FROM clientes WHERE nombre = ?", (nombre,))   # ✅
> ```
> Un parámetro único debe ir en una tupla: `(nombre,)` (con coma).

### 5.3 Obtener el id generado

```python
cur = conn.execute("INSERT INTO clientes (nombre, email) VALUES (?, ?)", ("Marta", "marta@x.com"))
nuevo_id = cur.lastrowid
```

Con `RETURNING` (SQLite 3.35+):

```python
fila = conn.execute(
    "INSERT INTO clientes (nombre, email) VALUES (?, ?) RETURNING id, creado",
    ("Pedro", "pedro@x.com"),
).fetchone()
```

### 5.4 Inserción masiva

```python
clientes = [
    ("Cliente 1", "c1@x.com", "Madrid"),
    ("Cliente 2", "c2@x.com", "Valencia"),
    ("Cliente 3", "c3@x.com", "Bilbao"),
]
conn.executemany("INSERT INTO clientes (nombre, email, ciudad) VALUES (?, ?, ?)", clientes)
conn.commit()
```

> `executemany` dentro de una sola transacción es **mucho** más rápido que insertar fila a fila con `commit()` en cada una.

### 5.5 Insertar ignorando duplicados o actualizando

```python
conn.execute("INSERT OR IGNORE INTO clientes (nombre, email) VALUES (?, ?)", ("Ana", "ana@ejemplo.com"))
conn.execute("INSERT OR REPLACE INTO productos (codigo, nombre, precio) VALUES (?, ?, ?)", ("A01", "Teclado", 29.9))

# UPSERT (SQLite 3.24+): actualiza solo los campos indicados
conn.execute("""
    INSERT INTO productos (codigo, nombre, precio, stock) VALUES (?, ?, ?, ?)
    ON CONFLICT(codigo) DO UPDATE SET precio = excluded.precio, stock = stock + excluded.stock
""", ("A01", "Teclado", 31.5, 10))
```

> `INSERT OR REPLACE` **elimina** la fila existente y crea otra (cambia el id y dispara borrados en cascada). Prefiera `ON CONFLICT ... DO UPDATE`.

---

## 6. Consultar datos

### 6.1 Leer resultados

```python
cur = conn.execute("SELECT id, nombre, ciudad FROM clientes WHERE ciudad = ?", ("Madrid",))

cur.fetchone()          # Una fila (o None)
cur.fetchmany(10)       # Las 10 siguientes
cur.fetchall()          # Todas las restantes

for id_, nombre, ciudad in conn.execute("SELECT id, nombre, ciudad FROM clientes"):
    print(id_, nombre, ciudad)
```

### 6.2 Filas como diccionarios (`sqlite3.Row`)

```python
conn.row_factory = sqlite3.Row

fila = conn.execute("SELECT * FROM clientes WHERE id = ?", (1,)).fetchone()
fila["nombre"]           # Por nombre de columna
fila[0]                  # Por posición
fila.keys()              # Nombres de columnas
dict(fila)               # Convertir en diccionario
```

Función propia para obtener `dict` directamente:

```python
def dict_factory(cursor, fila):
    return {col[0]: valor for col, valor in zip(cursor.description, fila)}

conn.row_factory = dict_factory
```

### 6.3 Valor único

```python
total = conn.execute("SELECT COUNT(*) FROM clientes").fetchone()[0]
existe = conn.execute("SELECT 1 FROM clientes WHERE email = ?", ("ana@ejemplo.com",)).fetchone() is not None
```

### 6.4 Consultas habituales

```python
# Filtros, orden y límite
conn.execute("""
    SELECT nombre, saldo FROM clientes
    WHERE activo = 1 AND saldo > ? AND nombre LIKE ?
    ORDER BY saldo DESC
    LIMIT ? OFFSET ?
""", (100, "A%", 10, 0))

# Cláusula IN con una lista variable
ciudades = ["Madrid", "Sevilla", "Bilbao"]
marcadores = ",".join("?" * len(ciudades))
conn.execute(f"SELECT * FROM clientes WHERE ciudad IN ({marcadores})", ciudades)

# Agregación
conn.execute("""
    SELECT ciudad, COUNT(*) AS clientes, ROUND(AVG(saldo), 2) AS saldo_medio
    FROM clientes
    GROUP BY ciudad
    HAVING COUNT(*) >= 2
    ORDER BY clientes DESC
""")

# JOIN
conn.execute("""
    SELECT c.nombre, COUNT(p.id) AS pedidos, COALESCE(SUM(p.total), 0) AS facturado
    FROM clientes c
    LEFT JOIN pedidos p ON p.cliente_id = c.id
    GROUP BY c.id
    ORDER BY facturado DESC
""")
```

> En la cláusula `IN` con lista variable solo se insertan **marcadores** `?` con f-string; los valores siguen pasando como parámetros, por lo que es seguro.

### 6.5 Nombres de columnas del resultado

```python
cur = conn.execute("SELECT * FROM clientes")
columnas = [d[0] for d in cur.description]
```

### 6.6 Búsqueda sin distinguir mayúsculas ni tildes

`LIKE` no distingue mayúsculas en caracteres ASCII, pero **sí** en letras con tilde. Una solución es registrar una función propia (sección 10.4).

---

## 7. Actualizar y eliminar

```python
cur = conn.execute("UPDATE clientes SET ciudad = ? WHERE id = ?", ("Barcelona", 1))
print(cur.rowcount, "filas actualizadas")

conn.execute("UPDATE productos SET precio = ROUND(precio * 1.05, 2) WHERE categoria = ?", ("Periféricos",))

cur = conn.execute("DELETE FROM clientes WHERE activo = 0")
print(cur.rowcount, "filas eliminadas")

conn.commit()
```

> Un `UPDATE` o `DELETE` sin `WHERE` afecta a **todas** las filas.

---

## 8. Transacciones

### 8.1 Comportamiento por defecto

El módulo abre una transacción automáticamente antes de `INSERT`, `UPDATE` y `DELETE`. Los cambios **no se guardan** hasta llamar a `commit()`.

### 8.2 Gestor de contexto (recomendado)

```python
try:
    with conn:                                  # commit si todo va bien, rollback si hay excepción
        conn.execute("UPDATE cuentas SET saldo = saldo - ? WHERE id = ?", (100, 1))
        conn.execute("UPDATE cuentas SET saldo = saldo + ? WHERE id = ?", (100, 2))
except sqlite3.Error as e:
    print("Transferencia cancelada:", e)
```

### 8.3 Control explícito

```python
try:
    conn.execute("BEGIN")
    ...
    conn.commit()
except Exception:
    conn.rollback()
    raise
```

### 8.4 Modo *autocommit*

```python
conn = sqlite3.connect("empresa.db", isolation_level=None)     # Cada sentencia se confirma sola
conn = sqlite3.connect("empresa.db", autocommit=True)          # Python 3.12+
```

### 8.5 Puntos de guardado

```python
with conn:
    conn.execute("SAVEPOINT antes_importar")
    try:
        importar_lote(conn)
    except Exception:
        conn.execute("ROLLBACK TO antes_importar")
    conn.execute("RELEASE antes_importar")
```

---

## 9. Tipos de datos y conversiones

### 9.1 Clases de almacenamiento

SQLite usa **tipado dinámico**: el tipo de una columna es una «afinidad», no una restricción estricta.

| SQLite | Python |
|---|---|
| `NULL` | `None` |
| `INTEGER` | `int` |
| `REAL` | `float` |
| `TEXT` | `str` |
| `BLOB` | `bytes` |

No existen tipos nativos `BOOLEAN`, `DATE` ni `DECIMAL`:

| Dato | Recomendación |
|---|---|
| Booleano | `INTEGER` con 0 / 1 |
| Fecha / fecha y hora | `TEXT` en formato ISO 8601 (`2026-11-24`, `2026-11-24 22:15:00`) |
| Importes exactos | `INTEGER` en céntimos, o `TEXT` con `Decimal` |
| JSON | `TEXT` + funciones JSON de SQLite |

### 9.2 Tablas estrictas (SQLite 3.37+)

```sql
CREATE TABLE medidas (
    id INTEGER PRIMARY KEY,
    valor REAL NOT NULL,
    fecha TEXT NOT NULL
) STRICT;
```

Con `STRICT`, SQLite rechaza valores del tipo incorrecto.

### 9.3 Fechas

```python
from datetime import date, datetime

conn.execute("INSERT INTO pedidos (cliente_id, fecha) VALUES (?, ?)", (1, date.today().isoformat()))
fila = conn.execute("SELECT fecha FROM pedidos WHERE id = ?", (1,)).fetchone()
fecha = date.fromisoformat(fila[0])
```

Funciones de fecha en SQL:

```sql
SELECT date('now');                           -- 2026-11-24 (UTC)
SELECT datetime('now', 'localtime');          -- Hora local
SELECT date('now', '-30 days');
SELECT date('now', 'start of month');
SELECT strftime('%Y-%m', fecha) AS mes FROM pedidos;
SELECT julianday('2026-12-31') - julianday('2026-11-24');   -- Días entre fechas
```

### 9.4 Adaptadores y conversores personalizados

```python
from decimal import Decimal

sqlite3.register_adapter(Decimal, str)                               # Python → SQLite
sqlite3.register_converter("DECIMAL", lambda b: Decimal(b.decode())) # SQLite → Python

conn = sqlite3.connect("empresa.db", detect_types=sqlite3.PARSE_DECLTYPES)
conn.execute("CREATE TABLE IF NOT EXISTS facturas (id INTEGER PRIMARY KEY, importe DECIMAL)")
conn.execute("INSERT INTO facturas (importe) VALUES (?)", (Decimal("199.95"),))
importe = conn.execute("SELECT importe FROM facturas").fetchone()[0]     # Decimal('199.95')
```

Fechas con adaptadores explícitos (los predeterminados están obsoletos desde Python 3.12):

```python
sqlite3.register_adapter(date, lambda d: d.isoformat())
sqlite3.register_adapter(datetime, lambda d: d.isoformat(" "))
sqlite3.register_converter("DATE", lambda b: date.fromisoformat(b.decode()))
sqlite3.register_converter("TIMESTAMP", lambda b: datetime.fromisoformat(b.decode()))
```

### 9.5 Archivos binarios (BLOB)

```python
datos = Path("logo.png").read_bytes()
conn.execute("INSERT INTO imagenes (nombre, contenido) VALUES (?, ?)", ("logo.png", datos))

contenido = conn.execute("SELECT contenido FROM imagenes WHERE nombre = ?", ("logo.png",)).fetchone()[0]
Path("copia.png").write_bytes(contenido)
```

---

## 10. Funcionalidades de SQLite

### 10.1 JSON

```python
import json

conn.execute("CREATE TABLE IF NOT EXISTS eventos (id INTEGER PRIMARY KEY, datos TEXT)")
conn.execute("INSERT INTO eventos (datos) VALUES (?)",
             (json.dumps({"tipo": "login", "usuario": "ana", "ip": "10.0.0.5"}),))

conn.execute("""
    SELECT json_extract(datos, '$.usuario') AS usuario, COUNT(*)
    FROM eventos
    WHERE json_extract(datos, '$.tipo') = 'login'
    GROUP BY usuario
""").fetchall()
```

En SQLite 3.38+ también puede usar `datos ->> '$.usuario'`.

### 10.2 Búsqueda de texto completo (FTS5)

```python
conn.executescript("""
    CREATE VIRTUAL TABLE IF NOT EXISTS documentos_fts USING fts5(titulo, contenido, tokenize='unicode61 remove_diacritics 2');
""")
conn.executemany("INSERT INTO documentos_fts (titulo, contenido) VALUES (?, ?)", [
    ("Política de devoluciones", "El cliente dispone de 30 días para devolver el producto..."),
    ("Envíos", "Los envíos a Canarias tienen un plazo de 5 días laborables..."),
])
resultados = conn.execute("""
    SELECT titulo, snippet(documentos_fts, 1, '[', ']', '…', 8)
    FROM documentos_fts WHERE documentos_fts MATCH ? ORDER BY rank
""", ("devolucion*",)).fetchall()
```

> `remove_diacritics 2` hace que las búsquedas ignoren las tildes («envio» encuentra «envíos»). El asterisco final busca por **prefijo**: `devolucion*` encuentra «devolución», «devoluciones», etc. Sin él, solo coinciden palabras completas.
>
> Sintaxis de búsqueda: `"frase exacta"`, `palabra1 AND palabra2`, `palabra1 OR palabra2`, `palabra1 NOT palabra2`, `titulo:envios` (solo en una columna).

### 10.3 Funciones de ventana y CTE

```sql
WITH ventas_mes AS (
    SELECT strftime('%Y-%m', fecha) AS mes, cliente_id, SUM(total) AS total
    FROM pedidos GROUP BY mes, cliente_id
)
SELECT mes, cliente_id, total,
       RANK() OVER (PARTITION BY mes ORDER BY total DESC) AS puesto,
       SUM(total) OVER (PARTITION BY cliente_id ORDER BY mes) AS acumulado
FROM ventas_mes;
```

### 10.4 Funciones definidas en Python

```python
import unicodedata

def normalizar(texto):
    if texto is None:
        return None
    return unicodedata.normalize("NFKD", texto).encode("ascii", "ignore").decode().lower()

conn.create_function("normalizar", 1, normalizar, deterministic=True)

conn.execute("SELECT nombre FROM clientes WHERE normalizar(nombre) LIKE ?", (f"%{normalizar('lópez')}%",))
```

Funciones de agregación propias:

```python
import statistics

class Mediana:
    def __init__(self): self.valores = []
    def step(self, v):
        if v is not None: self.valores.append(v)
    def finalize(self):
        return statistics.median(self.valores) if self.valores else None

conn.create_aggregate("mediana", 1, Mediana)
conn.execute("SELECT ciudad, mediana(saldo) FROM clientes GROUP BY ciudad").fetchall()
```

Ordenación propia (*collation*) para orden alfabético sin tildes:

```python
conn.create_collation("es", lambda a, b: (normalizar(a) > normalizar(b)) - (normalizar(a) < normalizar(b)))
conn.execute("SELECT nombre FROM clientes ORDER BY nombre COLLATE es")
```

### 10.5 Vistas y disparadores

```sql
CREATE VIEW IF NOT EXISTS v_resumen_clientes AS
SELECT c.id, c.nombre, COUNT(p.id) AS pedidos, COALESCE(SUM(p.total), 0) AS facturado
FROM clientes c LEFT JOIN pedidos p ON p.cliente_id = c.id
GROUP BY c.id;

CREATE TRIGGER IF NOT EXISTS tr_auditoria_saldo
AFTER UPDATE OF saldo ON clientes
BEGIN
    INSERT INTO auditoria (tabla, registro_id, campo, anterior, nuevo, fecha)
    VALUES ('clientes', NEW.id, 'saldo', OLD.saldo, NEW.saldo, datetime('now'));
END;
```

### 10.6 Adjuntar otra base de datos

```python
conn.execute("ATTACH DATABASE 'historico.db' AS hist")
conn.execute("INSERT INTO hist.pedidos SELECT * FROM main.pedidos WHERE fecha < '2025-01-01'")
conn.execute("DETACH DATABASE hist")
```

---

## 11. Rendimiento y concurrencia

### 11.1 Configuración recomendada (PRAGMA)

```python
def configurar(conn: sqlite3.Connection) -> None:
    conn.execute("PRAGMA foreign_keys = ON")
    conn.execute("PRAGMA journal_mode = WAL")        # Lecturas y escrituras simultáneas
    conn.execute("PRAGMA synchronous = NORMAL")      # Más rápido; seguro con WAL
    conn.execute("PRAGMA busy_timeout = 5000")       # Espera 5 s si la BD está bloqueada
    conn.execute("PRAGMA cache_size = -20000")       # ~20 MB de caché
```

| PRAGMA | Efecto |
|---|---|
| `journal_mode = WAL` | Permite que haya lectores mientras se escribe. Muy recomendado. |
| `busy_timeout` | Evita el error `database is locked` esperando en lugar de fallar inmediatamente. |
| `foreign_keys = ON` | Aplica las claves foráneas. |
| `synchronous = NORMAL` | Menos escrituras a disco; buen equilibrio con WAL. |

### 11.2 Inserciones rápidas

```python
import time

datos = [(f"Cliente {i}", f"c{i}@x.com") for i in range(100_000)]

inicio = time.perf_counter()
with conn:                                     # Una única transacción
    conn.executemany("INSERT INTO clientes (nombre, email) VALUES (?, ?)", datos)
print(f"{time.perf_counter() - inicio:.2f} s")
```

> Insertar 100 000 filas con un `commit()` por fila puede tardar minutos; en una sola transacción, menos de un segundo.

### 11.3 Índices y plan de ejecución

```python
conn.execute("CREATE INDEX IF NOT EXISTS ix_clientes_ciudad ON clientes(ciudad)")
for paso in conn.execute("EXPLAIN QUERY PLAN SELECT * FROM clientes WHERE ciudad = ?", ("Madrid",)):
    print(paso)            # Debe indicar "SEARCH ... USING INDEX"
conn.execute("ANALYZE")    # Actualiza las estadísticas del planificador
```

### 11.4 Mantenimiento

```python
conn.execute("PRAGMA integrity_check").fetchone()     # ('ok',) si la BD está sana
conn.execute("PRAGMA optimize")
conn.execute("VACUUM")                                # Compacta y recupera espacio
```

### 11.5 Hilos

- Por defecto, una conexión solo puede usarse en el hilo que la creó.
- Lo recomendable es **una conexión por hilo**.
- `check_same_thread=False` permite compartirla, pero entonces debe protegerla con un `Lock`.

```python
conn = sqlite3.connect("empresa.db", check_same_thread=False)
```

---

## 12. Copias de seguridad, importación y exportación

### 12.1 Copia de seguridad en caliente

```python
def copia_seguridad(origen: str, destino: str) -> None:
    with closing(sqlite3.connect(origen)) as src, closing(sqlite3.connect(destino)) as dst:
        src.backup(dst)                       # Segura aunque la BD esté en uso

copia_seguridad("empresa.db", f"backup_{date.today():%Y%m%d}.db")
```

> No copie el archivo `.db` con `shutil.copy` mientras la aplicación escribe: podría obtener una copia inconsistente (y en modo WAL faltarían los cambios del archivo `-wal`).

### 12.2 Volcado SQL

```python
with open("volcado.sql", "w", encoding="utf-8") as f:
    for linea in conn.iterdump():
        f.write(f"{linea}\n")

# Restaurar
nueva = sqlite3.connect("restaurada.db")
nueva.executescript(Path("volcado.sql").read_text(encoding="utf-8"))
```

### 12.3 Importar y exportar CSV

```python
import csv

with open("clientes.csv", encoding="utf-8-sig", newline="") as f:
    lector = csv.DictReader(f, delimiter=";")
    with conn:
        conn.executemany("INSERT INTO clientes (nombre, email, ciudad) VALUES (:nombre, :email, :ciudad)", lector)

cur = conn.execute("SELECT * FROM clientes")
with open("export.csv", "w", encoding="utf-8-sig", newline="") as f:
    w = csv.writer(f, delimiter=";")
    w.writerow([d[0] for d in cur.description])
    w.writerows(cur)
```

### 12.4 Pandas

```python
import pandas as pd

df = pd.read_sql_query("SELECT * FROM clientes WHERE ciudad = ?", conn, params=("Madrid",))
df.to_sql("clientes_import", conn, if_exists="append", index=False)
```

---

## 13. Ejemplo práctico completo

Gestor de gastos personales/empresariales: esquema con categorías y movimientos, configuración óptima, capa de acceso a datos con funciones reutilizables, importación de CSV, informes mensuales con CTE y funciones de ventana, presupuesto por categoría y copia de seguridad.

```python
"""
Gestor de gastos con sqlite3.
"""
import csv
import sqlite3
from contextlib import closing
from datetime import date
from pathlib import Path

RAIZ = Path(__file__).resolve().parent
RUTA_BD = RAIZ / "gastos.db"

ESQUEMA = """
CREATE TABLE IF NOT EXISTS categorias (
    id          INTEGER PRIMARY KEY,
    nombre      TEXT NOT NULL UNIQUE,
    presupuesto INTEGER NOT NULL DEFAULT 0          -- en céntimos / mes
);

CREATE TABLE IF NOT EXISTS movimientos (
    id           INTEGER PRIMARY KEY,
    fecha        TEXT    NOT NULL CHECK (fecha = date(fecha)),
    concepto     TEXT    NOT NULL,
    importe      INTEGER NOT NULL,                   -- en céntimos; negativo = gasto
    categoria_id INTEGER REFERENCES categorias(id) ON DELETE SET NULL,
    UNIQUE (fecha, concepto, importe)                -- evita importar duplicados
);

CREATE INDEX IF NOT EXISTS ix_mov_fecha ON movimientos(fecha);
CREATE INDEX IF NOT EXISTS ix_mov_categoria ON movimientos(categoria_id);

CREATE VIEW IF NOT EXISTS v_movimientos AS
SELECT m.id, m.fecha, strftime('%Y-%m', m.fecha) AS mes, m.concepto,
       m.importe / 100.0 AS importe, COALESCE(c.nombre, 'Sin categoría') AS categoria
FROM movimientos m LEFT JOIN categorias c ON c.id = m.categoria_id;
"""

REGLAS = {                        # Palabra clave en el concepto → categoría
    "mercadona": "Alimentación", "carrefour": "Alimentación", "lidl": "Alimentación",
    "repsol": "Transporte", "renfe": "Transporte", "cabify": "Transporte",
    "iberdrola": "Suministros", "endesa": "Suministros", "movistar": "Suministros",
    "amazon": "Compras", "netflix": "Ocio", "spotify": "Ocio", "nomina": "Ingresos",
}
PRESUPUESTOS = {"Alimentación": 400, "Transporte": 150, "Suministros": 200,
                "Compras": 150, "Ocio": 60, "Ingresos": 0}


def a_centimos(texto: str) -> int:
    limpio = texto.strip().replace("€", "").replace(" ", "").replace(".", "").replace(",", ".")
    return round(float(limpio) * 100)


def euros(centimos: float) -> str:
    return f"{centimos / 100:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def conectar() -> sqlite3.Connection:
    conn = sqlite3.connect(RUTA_BD)
    conn.row_factory = sqlite3.Row
    conn.execute("PRAGMA foreign_keys = ON")
    conn.execute("PRAGMA journal_mode = WAL")
    conn.execute("PRAGMA busy_timeout = 5000")
    return conn


def inicializar(conn: sqlite3.Connection) -> None:
    conn.executescript(ESQUEMA)
    with conn:
        conn.executemany(
            "INSERT INTO categorias (nombre, presupuesto) VALUES (?, ?) "
            "ON CONFLICT(nombre) DO UPDATE SET presupuesto = excluded.presupuesto",
            [(n, p * 100) for n, p in PRESUPUESTOS.items()],
        )


def categorizar(conn: sqlite3.Connection, concepto: str) -> int | None:
    texto = concepto.lower()
    for clave, categoria in REGLAS.items():
        if clave in texto:
            fila = conn.execute("SELECT id FROM categorias WHERE nombre = ?", (categoria,)).fetchone()
            return fila["id"] if fila else None
    return None


def importar_csv(conn: sqlite3.Connection, ruta: Path) -> tuple[int, int]:
    """CSV del banco: fecha (dd/mm/aaaa);concepto;importe (1.234,56)."""
    nuevos = duplicados = 0
    with ruta.open(encoding="utf-8-sig", newline="") as f, conn:
        for fila in csv.DictReader(f, delimiter=";"):
            d, m, a = fila["fecha"].split("/")
            datos = (f"{a}-{m}-{d}", fila["concepto"].strip(), a_centimos(fila["importe"]),
                     categorizar(conn, fila["concepto"]))
            cur = conn.execute(
                "INSERT OR IGNORE INTO movimientos (fecha, concepto, importe, categoria_id) VALUES (?, ?, ?, ?)",
                datos)
            nuevos += cur.rowcount
            duplicados += 1 - cur.rowcount
    return nuevos, duplicados


def resumen_mensual(conn: sqlite3.Connection) -> list[sqlite3.Row]:
    return conn.execute("""
        WITH mensual AS (
            SELECT strftime('%Y-%m', fecha) AS mes,
                   SUM(CASE WHEN importe > 0 THEN importe ELSE 0 END) AS ingresos,
                   SUM(CASE WHEN importe < 0 THEN -importe ELSE 0 END) AS gastos
            FROM movimientos GROUP BY mes
        )
        SELECT mes, ingresos, gastos, ingresos - gastos AS ahorro,
               SUM(ingresos - gastos) OVER (ORDER BY mes) AS ahorro_acumulado
        FROM mensual ORDER BY mes
    """).fetchall()


def control_presupuesto(conn: sqlite3.Connection, mes: str) -> list[sqlite3.Row]:
    return conn.execute("""
        SELECT c.nombre, c.presupuesto,
               COALESCE(SUM(-m.importe), 0) AS gastado,
               ROUND(100.0 * COALESCE(SUM(-m.importe), 0) / NULLIF(c.presupuesto, 0), 1) AS porcentaje
        FROM categorias c
        LEFT JOIN movimientos m
               ON m.categoria_id = c.id AND m.importe < 0 AND strftime('%Y-%m', m.fecha) = ?
        WHERE c.presupuesto > 0
        GROUP BY c.id
        ORDER BY porcentaje DESC
    """, (mes,)).fetchall()


def sin_categoria(conn: sqlite3.Connection) -> list[sqlite3.Row]:
    return conn.execute("SELECT fecha, concepto, importe FROM v_movimientos "
                        "WHERE categoria = 'Sin categoría' ORDER BY fecha").fetchall()


def copia_seguridad() -> Path:
    destino = RAIZ / f"gastos_backup_{date.today():%Y%m%d}.db"
    with closing(sqlite3.connect(RUTA_BD)) as src, closing(sqlite3.connect(destino)) as dst:
        src.backup(dst)
    return destino


def crear_csv_ejemplo(ruta: Path) -> None:
    filas = [
        ("01/10/2026", "NOMINA OCTUBRE", "2.150,00"), ("03/10/2026", "MERCADONA 1234", "-87,45"),
        ("05/10/2026", "REPSOL ESTACION", "-60,00"), ("08/10/2026", "IBERDROLA FACTURA", "-74,30"),
        ("12/10/2026", "AMAZON EU", "-39,99"), ("15/10/2026", "NETFLIX", "-13,99"),
        ("20/10/2026", "CARREFOUR EXPRESS", "-156,20"), ("25/10/2026", "FARMACIA CENTRO", "-22,50"),
        ("01/11/2026", "NOMINA NOVIEMBRE", "2.150,00"), ("02/11/2026", "LIDL", "-112,80"),
        ("06/11/2026", "RENFE BILLETE", "-45,60"), ("09/11/2026", "MOVISTAR", "-55,00"),
        ("14/11/2026", "MERCADONA 1234", "-198,35"), ("18/11/2026", "SPOTIFY", "-10,99"),
        ("21/11/2026", "CABIFY", "-18,40"), ("22/11/2026", "MERCADONA 1234", "-131,10"),
    ]
    with ruta.open("w", encoding="utf-8-sig", newline="") as f:
        w = csv.writer(f, delimiter=";")
        w.writerow(["fecha", "concepto", "importe"])
        w.writerows(filas)


def main() -> None:
    csv_banco = RAIZ / "movimientos_banco.csv"
    if not csv_banco.exists():
        crear_csv_ejemplo(csv_banco)

    with closing(conectar()) as conn:
        inicializar(conn)
        nuevos, dup = importar_csv(conn, csv_banco)
        print(f"Importación: {nuevos} nuevos, {dup} duplicados ignorados\n")

        print(f"{'Mes':<8} {'Ingresos':>12} {'Gastos':>12} {'Ahorro':>12} {'Acumulado':>12}")
        for r in resumen_mensual(conn):
            print(f"{r['mes']:<8} {euros(r['ingresos']):>12} {euros(r['gastos']):>12} "
                  f"{euros(r['ahorro']):>12} {euros(r['ahorro_acumulado']):>12}")

        mes = "2026-11"
        print(f"\nPresupuesto {mes}:")
        for r in control_presupuesto(conn, mes):
            pct = r["porcentaje"] or 0
            barra = "█" * min(20, int(pct / 5))
            aviso = "  ⚠ SUPERADO" if pct > 100 else ""
            print(f"  {r['nombre']:<13} {euros(r['gastado']):>10} / {euros(r['presupuesto']):>10} "
                  f"{pct:6.1f} % {barra}{aviso}")

        pendientes = sin_categoria(conn)
        if pendientes:
            print("\nMovimientos sin categoría:")
            for r in pendientes:
                print(f"  {r['fecha']}  {r['concepto']:<20} {r['importe']:>8.2f} €")

        print("\nIntegridad:", conn.execute("PRAGMA integrity_check").fetchone()[0])

    print("Copia de seguridad:", copia_seguridad().name)


if __name__ == "__main__":
    main()
```

---

## 14. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| Los datos no se guardan | Falta `commit()`. | Use `with conn:` o llame a `conn.commit()`. |
| `sqlite3.OperationalError: database is locked` | Otra conexión está escribiendo o una transacción quedó abierta. | `PRAGMA busy_timeout`, modo WAL, cierre conexiones y confirme transacciones. |
| `sqlite3.IntegrityError: UNIQUE constraint failed` | Valor duplicado. | `INSERT OR IGNORE`, `ON CONFLICT` o capture la excepción. |
| `sqlite3.IntegrityError: FOREIGN KEY constraint failed` | Referencia a un registro inexistente. | Compruebe los datos; recuerde activar `foreign_keys`. |
| Las claves foráneas no se respetan | `foreign_keys` desactivado por defecto. | `PRAGMA foreign_keys = ON` en cada conexión. |
| `ProgrammingError: Incorrect number of bindings supplied` | Parámetro único sin tupla: `("Ana")`. | Use `("Ana",)` con coma. |
| `OperationalError: no such table` | Ruta de archivo distinta (se creó otra BD vacía). | Use rutas absolutas basadas en `__file__`. |
| `OperationalError: near "...": syntax error` | Palabra reservada o error SQL. | Revise la sentencia; entrecomille identificadores con `"`. |
| `ProgrammingError: SQLite objects created in a thread can only be used in that same thread` | Conexión compartida entre hilos. | Una conexión por hilo o `check_same_thread=False` con bloqueo. |
| `DeprecationWarning: The default date adapter is deprecated` | Uso implícito de adaptadores de fecha (Python 3.12+). | Registre adaptadores explícitos o guarde `isoformat()`. |
| Sumas con decimales inexactos | `REAL` es coma flotante. | Guarde importes en céntimos (`INTEGER`). |

---

## 15. Buenas prácticas

- Use **siempre** parámetros (`?` o `:nombre`).
- Active `foreign_keys`, `journal_mode = WAL` y `busy_timeout` al conectar.
- Agrupe las escrituras en transacciones (`with conn:`) y use `executemany`.
- Use `sqlite3.Row` para acceder a las columnas por nombre.
- Guarde fechas en ISO 8601 e importes en céntimos o como `Decimal`.
- Defina restricciones (`NOT NULL`, `UNIQUE`, `CHECK`, claves foráneas) en el esquema.
- Cree índices en las columnas usadas para filtrar y unir; verifique con `EXPLAIN QUERY PLAN`.
- Haga copias con `conn.backup()`, no copiando el archivo en uso.
- Cierre las conexiones (`closing` o `conn.close()`).
- Para aplicaciones grandes, considere **SQLAlchemy** sobre SQLite; para muchos usuarios concurrentes, PostgreSQL.

---

## 16. Referencia rápida

```python
conn = sqlite3.connect("bd.db" | ":memory:", detect_types=sqlite3.PARSE_DECLTYPES)
conn.row_factory = sqlite3.Row
conn.execute("PRAGMA foreign_keys = ON"); conn.execute("PRAGMA journal_mode = WAL")

conn.execute(sql, (p1, p2)) / conn.execute(sql, {"n": v}) / conn.executemany(sql, filas) / conn.executescript(sql)
cur.fetchone() / fetchmany(n) / fetchall() / cur.lastrowid / cur.rowcount / cur.description

with conn: ...                 # commit / rollback automáticos
conn.commit() / conn.rollback() / conn.close()
with closing(sqlite3.connect(...)) as conn: ...

conn.create_function("f", n, func) / create_aggregate("g", n, Clase) / create_collation("c", func)
sqlite3.register_adapter(tipo, f) / register_converter("TIPO", f)
src.backup(dst) / conn.iterdump()
sqlite3.Error / IntegrityError / OperationalError / ProgrammingError
```

---

## 17. Ejercicios propuestos

1. Cree una agenda de contactos con búsqueda por nombre sin distinguir tildes ni mayúsculas.
2. Importe un CSV de 100 000 filas y compare el tiempo con y sin transacción única.
3. Diseñe una base de datos de inventario con un disparador que registre cada cambio de stock en una tabla de auditoría.
4. Implemente un buscador de documentos con FTS5 que muestre fragmentos resaltados.
5. Programe una copia de seguridad diaria que conserve los últimos 7 días y verifique la integridad de cada copia.

---

## 18. Referencias

- Documentación de `sqlite3`: https://docs.python.org/es/3/library/sqlite3.html
- Documentación de SQLite: https://www.sqlite.org/docs.html
- Funciones JSON: https://www.sqlite.org/json1.html
- FTS5: https://www.sqlite.org/fts5.html
