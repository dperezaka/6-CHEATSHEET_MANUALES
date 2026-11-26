# Manual de uso: PyMySQL

> **Categoría:** Bases de datos · **Utilidad:** Conectar con MySQL
> **Tipo:** Librería externa (Python puro) · Compatible con **MySQL** y **MariaDB**

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Conectar](#4-conectar)
5. [Crear tablas](#5-crear-tablas)
6. [Consultar datos](#6-consultar-datos)
7. [Insertar, actualizar y eliminar](#7-insertar-actualizar-y-eliminar)
8. [Transacciones](#8-transacciones)
9. [Tipos de datos](#9-tipos-de-datos)
10. [Carga masiva y grandes resultados](#10-carga-masiva-y-grandes-resultados)
11. [Gestión de conexiones](#11-gestión-de-conexiones)
12. [Funcionalidades avanzadas](#12-funcionalidades-avanzadas)
13. [Integración con Pandas y SQLAlchemy](#13-integración-con-pandas-y-sqlalchemy)
14. [Ejemplo práctico completo](#14-ejemplo-práctico-completo)
15. [Errores frecuentes](#15-errores-frecuentes)
16. [Buenas prácticas](#16-buenas-prácticas)
17. [Referencia rápida](#17-referencia-rápida)
18. [Ejercicios propuestos](#18-ejercicios-propuestos)
19. [Referencias](#19-referencias)

---

## 1. Descripción

**PyMySQL** es un cliente de **MySQL** y **MariaDB** escrito íntegramente en Python. Implementa el estándar DB-API 2.0, por lo que su uso es muy similar a `sqlite3` o `psycopg`.

**Ventajas:**
- No necesita compilación ni librerías del sistema: se instala en cualquier plataforma con `pip`.
- Compatible con MySQL 5.7+, MySQL 8.x y MariaDB.
- Muy usado junto a SQLAlchemy, Pandas, Flask y Django.

**Alternativas:**

| Librería | Características |
|---|---|
| **PyMySQL** | Python puro; instalación sencilla. |
| `mysqlclient` (MySQLdb) | Extensión en C; más rápida; requiere compilación/librerías. Recomendada por Django. |
| `mysql-connector-python` | Oficial de Oracle. |
| `aiomysql` / `asyncmy` | Asíncronas. |

---

## 2. Instalación e importación

```bash
pip install pymysql
pip install "pymysql[rsa]"         # Necesario para la autenticación caching_sha2_password de MySQL 8 sin SSL
```

```python
import pymysql
import pymysql.cursors
```

Para practicar con Docker:

```bash
docker run --name mysql -e MYSQL_ROOT_PASSWORD=secreto -e MYSQL_DATABASE=tienda \
           -e MYSQL_USER=app -e MYSQL_PASSWORD=app -p 3306:3306 -d mysql:8.4
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Connection` | Conexión con el servidor (`pymysql.connect`). |
| `Cursor` | Ejecuta consultas y devuelve resultados. |
| `DictCursor` | Cursor que devuelve las filas como diccionarios. |
| `SSCursor` / `SSDictCursor` | Cursores sin búfer para resultados muy grandes. |
| Marcadores `%s` / `%(nombre)s` | Parámetros de consulta. |
| `autocommit` | Desactivado por defecto: los cambios requieren `commit()`. |
| `charset="utf8mb4"` | Codificación completa de Unicode (incluye emojis). |
| Motor InnoDB | Motor de almacenamiento con transacciones y claves foráneas (por defecto). |

---

## 4. Conectar

### 4.1 Conexión básica

```python
import os

conexion = pymysql.connect(
    host=os.getenv("MYSQL_HOST", "localhost"),
    port=int(os.getenv("MYSQL_PORT", "3306")),
    user=os.environ["MYSQL_USER"],
    password=os.environ["MYSQL_PASSWORD"],
    database=os.environ["MYSQL_DATABASE"],
    charset="utf8mb4",
    cursorclass=pymysql.cursors.DictCursor,
    connect_timeout=10,
    autocommit=False,
)
```

| Parámetro | Descripción |
|---|---|
| `host`, `port` | Servidor y puerto (3306 por defecto). |
| `user`, `password` | Credenciales. |
| `database` | Base de datos inicial (también `db`). |
| `charset` | Use siempre `utf8mb4`. |
| `cursorclass` | Tipo de cursor por defecto. |
| `autocommit` | Confirmación automática de cada sentencia. |
| `connect_timeout`, `read_timeout`, `write_timeout` | Tiempos máximos (segundos). |
| `ssl` | Configuración SSL (`{"ca": "/ruta/ca.pem"}`). |
| `init_command` | Sentencia ejecutada al conectar (p. ej. `SET time_zone='+00:00'`). |
| `local_infile` | Permite `LOAD DATA LOCAL INFILE`. |

### 4.2 Gestores de contexto

```python
with pymysql.connect(**config) as conexion:
    with conexion.cursor() as cursor:
        cursor.execute("SELECT VERSION() AS version")
        print(cursor.fetchone())
    conexion.commit()
# La conexión se cierra al salir del bloque
```

> El bloque `with conexion` **cierra** la conexión, pero **no** confirma automáticamente: llame a `commit()` cuando corresponda.

### 4.3 Comprobar y recuperar la conexión

```python
conexion.ping()                         # Error si la conexión se ha perdido (ver sección 11.1)
conexion.open                           # True si está abierta
conexion.get_server_info()              # '8.4.0' / '11.4.2-MariaDB'
```

---

## 5. Crear tablas

```python
with conexion.cursor() as cur:
    cur.execute("""
        CREATE TABLE IF NOT EXISTS clientes (
            id         INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            nombre     VARCHAR(100) NOT NULL,
            email      VARCHAR(150) NOT NULL UNIQUE,
            ciudad     VARCHAR(60),
            saldo      DECIMAL(12,2) NOT NULL DEFAULT 0,
            activo     BOOLEAN NOT NULL DEFAULT TRUE,
            creado     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
            modificado DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            INDEX ix_ciudad (ciudad)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
    """)

    cur.execute("""
        CREATE TABLE IF NOT EXISTS pedidos (
            id         INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            cliente_id INT UNSIGNED NOT NULL,
            fecha      DATE NOT NULL,
            total      DECIMAL(12,2) NOT NULL DEFAULT 0,
            estado     ENUM('pendiente','enviado','entregado','cancelado') NOT NULL DEFAULT 'pendiente',
            CONSTRAINT fk_pedidos_cliente FOREIGN KEY (cliente_id)
                REFERENCES clientes(id) ON DELETE CASCADE,
            INDEX ix_fecha (fecha)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
    """)
conexion.commit()
```

> **Intercalación (*collation*):** `utf8mb4_0900_ai_ci` (MySQL 8) compara sin distinguir mayúsculas ni tildes (`ai` = *accent insensitive*, `ci` = *case insensitive*): «Lopez» y «López» se consideran iguales en búsquedas y en índices `UNIQUE`. En MariaDB use `utf8mb4_unicode_ci` o `utf8mb4_uca1400_ai_ci`.

Consultar el esquema:

```python
cur.execute("SHOW TABLES")
cur.execute("DESCRIBE clientes")
cur.execute("SHOW CREATE TABLE clientes")
cur.execute("SHOW INDEX FROM pedidos")
```

---

## 6. Consultar datos

### 6.1 Consultas con parámetros

```python
with conexion.cursor() as cur:
    cur.execute("SELECT id, nombre, saldo FROM clientes WHERE ciudad = %s AND saldo > %s",
                ("Madrid", 100))
    for fila in cur.fetchall():
        print(fila["nombre"], fila["saldo"])       # Con DictCursor
```

Parámetros con nombre:

```python
cur.execute("SELECT * FROM clientes WHERE ciudad = %(ciudad)s AND activo = %(activo)s",
            {"ciudad": "Sevilla", "activo": True})
```

> ⚠ Marcador **siempre `%s`**, para cualquier tipo. Nunca use f-strings con datos externos:
> ```python
> cur.execute(f"SELECT * FROM clientes WHERE email = '{email}'")   # ❌ Inyección SQL
> cur.execute("SELECT * FROM clientes WHERE email = %s", (email,))  # ✅
> ```

### 6.2 Leer resultados

```python
cur.fetchone()          # Una fila (None si no hay más)
cur.fetchmany(50)
cur.fetchall()
cur.rowcount            # Filas devueltas o afectadas
cur.description         # Metadatos de columnas
```

### 6.3 Tipos de cursor

```python
from pymysql.cursors import Cursor, DictCursor, SSCursor, SSDictCursor

with conexion.cursor(DictCursor) as cur:       # Cursor concreto para este bloque
    ...
```

| Cursor | Filas | Memoria |
|---|---|---|
| `Cursor` | Tuplas | Carga todo el resultado |
| `DictCursor` | Diccionarios | Carga todo el resultado |
| `SSCursor` | Tuplas | Lee del servidor a medida que avanza |
| `SSDictCursor` | Diccionarios | Lee del servidor a medida que avanza |

### 6.4 Cláusula IN con listas

PyMySQL convierte listas y tuplas en una lista SQL entre paréntesis:

```python
ciudades = ("Madrid", "Sevilla", "Bilbao")
cur.execute("SELECT * FROM clientes WHERE ciudad IN %s", (ciudades,))
# → ... WHERE ciudad IN ('Madrid','Sevilla','Bilbao')
```

> No ponga paréntesis alrededor de `%s`: PyMySQL los añade. Con una lista vacía se produce un error de sintaxis; compruébelo antes.

### 6.5 LIKE y el carácter `%`

```python
cur.execute("SELECT * FROM clientes WHERE nombre LIKE %s", (f"%{texto}%",))
```

Si escribe `%` literal en la consulta y además usa parámetros, duplíquelo (`%%`).

### 6.6 Ver la consulta final

```python
print(cur.mogrify("SELECT * FROM clientes WHERE ciudad = %s", ("Madrid",)))
# SELECT * FROM clientes WHERE ciudad = 'Madrid'
```

### 6.7 Consultas habituales

```python
# Agregación con JOIN
cur.execute("""
    SELECT c.ciudad, COUNT(DISTINCT c.id) AS clientes, COUNT(p.id) AS pedidos,
           COALESCE(SUM(p.total), 0) AS facturado
    FROM clientes c
    LEFT JOIN pedidos p ON p.cliente_id = c.id AND p.estado <> 'cancelado'
    GROUP BY c.ciudad
    ORDER BY facturado DESC
""")

# Paginación
cur.execute("SELECT * FROM clientes ORDER BY id LIMIT %s OFFSET %s", (20, 40))

# Ventas por mes
cur.execute("""
    SELECT DATE_FORMAT(fecha, '%%Y-%%m') AS mes, SUM(total) AS ventas
    FROM pedidos WHERE fecha >= %s GROUP BY mes ORDER BY mes
""", ("2026-01-01",))
```

> En el último ejemplo, `%Y-%m` se escribe `%%Y-%%m` porque la consulta usa parámetros.

---

## 7. Insertar, actualizar y eliminar

### 7.1 Insertar y obtener el id

```python
with conexion.cursor() as cur:
    cur.execute("INSERT INTO clientes (nombre, email, ciudad) VALUES (%s, %s, %s)",
                ("Ana López", "ana@ejemplo.com", "Madrid"))
    nuevo_id = cur.lastrowid
conexion.commit()
```

### 7.2 Inserción múltiple

```python
clientes = [
    ("Luis Gómez", "luis@ejemplo.com", "Sevilla"),
    ("Marta Ruiz", "marta@ejemplo.com", "Valencia"),
    ("Pedro Sanz", "pedro@ejemplo.com", "Bilbao"),
]
with conexion.cursor() as cur:
    filas = cur.executemany("INSERT INTO clientes (nombre, email, ciudad) VALUES (%s, %s, %s)", clientes)
conexion.commit()
print(filas, "filas insertadas")
```

> Con sentencias `INSERT ... VALUES`, `executemany` las agrupa en una sola sentencia con varios `VALUES`, lo que es muy eficiente.

### 7.3 Insertar o actualizar

```python
# Actualizar si la clave única ya existe
cur.execute("""
    INSERT INTO productos (codigo, nombre, precio, stock)
    VALUES (%s, %s, %s, %s) AS nuevo
    ON DUPLICATE KEY UPDATE precio = nuevo.precio, stock = productos.stock + nuevo.stock
""", ("A01", "Teclado", 31.50, 10))
```

> La sintaxis `AS nuevo` está disponible desde MySQL 8.0.19. En versiones anteriores y en MariaDB use `VALUES(precio)`: `ON DUPLICATE KEY UPDATE precio = VALUES(precio)`.

```python
cur.execute("INSERT IGNORE INTO clientes (nombre, email) VALUES (%s, %s)", ("Ana", "ana@ejemplo.com"))
# Ignora la fila si viola una clave única (cur.rowcount == 0)
```

`REPLACE INTO` elimina y vuelve a insertar la fila: evítelo si hay claves foráneas o columnas que no se indican.

### 7.4 Actualizar y eliminar

```python
with conexion.cursor() as cur:
    cur.execute("UPDATE clientes SET ciudad = %s WHERE id = %s", ("Barcelona", 1))
    print(cur.rowcount, "filas modificadas")

    cur.execute("DELETE FROM pedidos WHERE estado = %s AND fecha < %s", ("cancelado", "2025-01-01"))
conexion.commit()
```

> En MySQL, `rowcount` en un `UPDATE` cuenta las filas **realmente cambiadas** (no las que coinciden pero ya tenían ese valor), salvo que se conecte con `client_flag=pymysql.constants.CLIENT.FOUND_ROWS`.

---

## 8. Transacciones

```python
try:
    with conexion.cursor() as cur:
        cur.execute("UPDATE cuentas SET saldo = saldo - %s WHERE id = %s", (100, 1))
        cur.execute("UPDATE cuentas SET saldo = saldo + %s WHERE id = %s", (100, 2))
    conexion.commit()
except pymysql.MySQLError:
    conexion.rollback()
    raise
```

### 8.1 Gestor de transacción propio

```python
from contextlib import contextmanager


@contextmanager
def transaccion(conexion):
    try:
        conexion.begin()
        with conexion.cursor() as cur:
            yield cur
        conexion.commit()
    except Exception:
        conexion.rollback()
        raise


with transaccion(conexion) as cur:
    cur.execute("INSERT INTO pedidos (cliente_id, fecha, total) VALUES (%s, CURDATE(), %s)", (1, 59.8))
    cur.execute("UPDATE clientes SET saldo = saldo - %s WHERE id = %s", (59.8, 1))
```

### 8.2 Bloqueo de filas

```python
with transaccion(conexion) as cur:
    cur.execute("SELECT stock FROM productos WHERE id = %s FOR UPDATE", (5,))
    stock = cur.fetchone()["stock"]
    if stock >= 2:
        cur.execute("UPDATE productos SET stock = stock - 2 WHERE id = %s", (5,))
```

### 8.3 Puntos de guardado

```python
cur.execute("SAVEPOINT antes_linea")
try:
    cur.execute("INSERT INTO lineas ...")
except pymysql.IntegrityError:
    cur.execute("ROLLBACK TO SAVEPOINT antes_linea")
```

> Las órdenes DDL (`CREATE`, `ALTER`, `DROP`) provocan un **commit implícito** en MySQL: no pueden deshacerse con `rollback()`.

---

## 9. Tipos de datos

| MySQL | Python |
|---|---|
| `TINYINT`, `INT`, `BIGINT` | `int` |
| `BOOLEAN` (= `TINYINT(1)`) | `int` (0/1) |
| `DECIMAL(p,s)` | `Decimal` |
| `FLOAT`, `DOUBLE` | `float` |
| `VARCHAR`, `TEXT`, `ENUM` | `str` |
| `DATE` | `datetime.date` |
| `DATETIME`, `TIMESTAMP` | `datetime.datetime` |
| `TIME` | `datetime.timedelta` |
| `JSON` | `str` (convertir con `json.loads`) |
| `BLOB`, `BINARY` | `bytes` |
| `NULL` | `None` |

### 9.1 Booleanos

```python
cur.execute("SELECT activo FROM clientes WHERE id = %s", (1,))
activo = bool(cur.fetchone()["activo"])        # MySQL devuelve 0 / 1
```

### 9.2 Importes

```python
from decimal import Decimal
cur.execute("INSERT INTO pedidos (cliente_id, fecha, total) VALUES (%s, %s, %s)", (1, "2026-11-24", Decimal("59.80")))
```

### 9.3 JSON

```python
import json

cur.execute("CREATE TABLE IF NOT EXISTS eventos (id INT AUTO_INCREMENT PRIMARY KEY, datos JSON)")
cur.execute("INSERT INTO eventos (datos) VALUES (%s)", (json.dumps({"tipo": "login", "usuario": "ana"}),))

cur.execute("""
    SELECT datos->>'$.usuario' AS usuario, COUNT(*) AS n
    FROM eventos WHERE JSON_EXTRACT(datos, '$.tipo') = 'login'
    GROUP BY usuario
""")
```

### 9.4 Fechas y zona horaria

```python
from datetime import datetime
cur.execute("INSERT INTO accesos (usuario, momento) VALUES (%s, %s)", ("ana", datetime.now()))
```

- `DATETIME` guarda la fecha tal cual, sin zona horaria.
- `TIMESTAMP` se guarda en UTC y se convierte según la zona de la sesión (rango limitado hasta 2038).
- Recomendación: guarde en UTC y fije la zona de la sesión con `init_command="SET time_zone = '+00:00'"`.

---

## 10. Carga masiva y grandes resultados

### 10.1 Inserción por lotes

```python
def insertar_por_lotes(conexion, sql, filas, tamano=5000):
    total = 0
    with conexion.cursor() as cur:
        for i in range(0, len(filas), tamano):
            total += cur.executemany(sql, filas[i:i + tamano])
            conexion.commit()
    return total
```

> MySQL limita el tamaño de cada paquete (`max_allowed_packet`, 64 MB por defecto en MySQL 8). Los lotes evitan superarlo.

### 10.2 `LOAD DATA LOCAL INFILE` (la opción más rápida)

```python
conexion = pymysql.connect(**config, local_infile=True)
with conexion.cursor() as cur:
    cur.execute("""
        LOAD DATA LOCAL INFILE %s
        INTO TABLE clientes
        CHARACTER SET utf8mb4
        FIELDS TERMINATED BY ';' OPTIONALLY ENCLOSED BY '"'
        LINES TERMINATED BY '\\n'
        IGNORE 1 LINES
        (nombre, email, ciudad)
    """, ("clientes.csv",))
conexion.commit()
```

> Requiere que el servidor tenga activada la opción `local_infile=1`.

### 10.3 Leer millones de filas sin agotar la memoria

```python
with conexion.cursor(pymysql.cursors.SSDictCursor) as cur:
    cur.execute("SELECT * FROM movimientos")
    for fila in cur:                 # Se lee del servidor fila a fila
        procesar(fila)
```

> Con un cursor sin búfer debe leer **todo** el resultado (o cerrar el cursor) antes de ejecutar otra consulta en la misma conexión.

---

## 11. Gestión de conexiones

### 11.1 Conexiones caducadas

MySQL cierra las conexiones inactivas tras `wait_timeout` (8 horas por defecto). En procesos de larga duración:

```python
def conexion_viva(conexion, config: dict):
    """Devuelve la conexión si sigue activa; si no, crea una nueva."""
    try:
        conexion.ping()                      # Lanza una excepción si la conexión se ha perdido
        return conexion
    except (pymysql.err.Error, AttributeError):
        return pymysql.connect(**config)
```

> En PyMySQL 1.1+ el parámetro `ping(reconnect=True)` está **obsoleto** (genera `DeprecationWarning`). La recomendación es crear una conexión nueva cuando la actual se ha perdido, o usar un pool (SQLAlchemy con `pool_pre_ping=True`).

### 11.2 Clase de acceso a datos reutilizable

```python
class BaseDatos:
    def __init__(self, **config):
        self.config = {"charset": "utf8mb4", "cursorclass": pymysql.cursors.DictCursor,
                       "autocommit": False, **config}
        self._conexion = None

    @property
    def conexion(self):
        try:
            if self._conexion is None or not self._conexion.open:
                raise pymysql.err.InterfaceError("sin conexión")
            self._conexion.ping()
        except pymysql.err.Error:
            self._conexion = pymysql.connect(**self.config)
        return self._conexion

    def consultar(self, sql, params=None):
        with self.conexion.cursor() as cur:
            cur.execute(sql, params)
            return cur.fetchall()

    def ejecutar(self, sql, params=None) -> int:
        with self.conexion.cursor() as cur:
            filas = cur.execute(sql, params)
        self.conexion.commit()
        return filas

    def cerrar(self):
        if self._conexion and self._conexion.open:
            self._conexion.close()
```

### 11.3 Pool de conexiones

PyMySQL no incluye un pool. Opciones:
- Usar **SQLAlchemy** con la URL `mysql+pymysql://...` (incluye pool).
- Librerías como `DBUtils` (`PooledDB`).

```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://app:app@localhost/tienda?charset=utf8mb4",
                       pool_size=5, pool_recycle=3600, pool_pre_ping=True)
```

---

## 12. Funcionalidades avanzadas

### 12.1 Procedimientos almacenados

```python
cur.callproc("calcular_comisiones", (2026, 11))
resultado = cur.fetchall()
```

### 12.2 Varias sentencias en una llamada

```python
from pymysql.constants import CLIENT

conexion = pymysql.connect(**config, client_flag=CLIENT.MULTI_STATEMENTS)
with conexion.cursor() as cur:
    cur.execute("SELECT COUNT(*) AS n FROM clientes; SELECT COUNT(*) AS n FROM pedidos")
    print(cur.fetchall())
    cur.nextset()
    print(cur.fetchall())
```

> Active `MULTI_STATEMENTS` solo cuando sea necesario: aumenta el riesgo en caso de inyección SQL.

### 12.3 Escapar identificadores dinámicos

Los parámetros solo sirven para **valores**. Para nombres de tablas o columnas, valide contra una lista permitida:

```python
COLUMNAS_ORDEN = {"nombre", "ciudad", "saldo", "creado"}

def listar(cur, orden: str = "nombre", desc: bool = False):
    if orden not in COLUMNAS_ORDEN:
        raise ValueError(f"Columna no permitida: {orden}")
    cur.execute(f"SELECT * FROM clientes ORDER BY `{orden}` {'DESC' if desc else 'ASC'}")
    return cur.fetchall()
```

### 12.4 Gestión de errores

```python
try:
    cur.execute("INSERT INTO clientes (nombre, email) VALUES (%s, %s)", ("Ana", "ana@ejemplo.com"))
    conexion.commit()
except pymysql.err.IntegrityError as e:
    conexion.rollback()
    codigo, mensaje = e.args
    if codigo == 1062:
        print("Registro duplicado:", mensaje)
    elif codigo == 1452:
        print("Clave foránea no válida")
except pymysql.err.OperationalError as e:
    print("Error de conexión o del servidor:", e.args)
except pymysql.err.ProgrammingError as e:
    print("Error de SQL:", e.args)
```

| Código | Significado |
|---|---|
| 1045 | Acceso denegado (usuario/contraseña). |
| 1049 | Base de datos desconocida. |
| 1054 | Columna desconocida. |
| 1062 | Entrada duplicada (clave única). |
| 1146 | La tabla no existe. |
| 1213 | Interbloqueo (*deadlock*): reintente la transacción. |
| 1451 / 1452 | Restricción de clave foránea. |
| 2003 | No se puede conectar con el servidor. |
| 2006 / 2013 | Se perdió la conexión con el servidor. |

### 12.5 Reintentar ante interbloqueos

```python
import time

def con_reintentos(funcion, intentos=3):
    for n in range(1, intentos + 1):
        try:
            return funcion()
        except pymysql.err.OperationalError as e:
            if e.args[0] in (1213, 1205) and n < intentos:       # Deadlock / lock wait timeout
                conexion.rollback()
                time.sleep(0.2 * n)
                continue
            raise
```

---

## 13. Integración con Pandas y SQLAlchemy

```python
import pandas as pd
from sqlalchemy import create_engine, text

engine = create_engine("mysql+pymysql://app:app@localhost:3306/tienda?charset=utf8mb4")

df = pd.read_sql(text("SELECT * FROM pedidos WHERE fecha >= :desde"), engine, params={"desde": "2026-01-01"})
df.to_sql("pedidos_copia", engine, if_exists="replace", index=False, chunksize=2000, method="multi")
```

> Si la contraseña contiene caracteres especiales, construya la URL con `sqlalchemy.URL.create(...)`.

---

## 14. Ejemplo práctico completo

Sistema de gestión de reservas de salas: esquema con restricciones, inserción masiva de datos, comprobación de solapamientos con bloqueo de filas dentro de una transacción, gestión de errores por código, informe de ocupación y exportación a CSV.

```python
"""
Reservas de salas con PyMySQL.
Requisitos: pip install pymysql
Variables de entorno: MYSQL_HOST, MYSQL_USER, MYSQL_PASSWORD, MYSQL_DATABASE
"""
import csv
import os
import random
from contextlib import contextmanager
from datetime import date, datetime, time, timedelta

import pymysql
from pymysql.cursors import DictCursor

CONFIG = {
    "host": os.getenv("MYSQL_HOST", "localhost"),
    "port": int(os.getenv("MYSQL_PORT", "3306")),
    "user": os.getenv("MYSQL_USER", "app"),
    "password": os.getenv("MYSQL_PASSWORD", "app"),
    "database": os.getenv("MYSQL_DATABASE", "tienda"),
    "charset": "utf8mb4",
    "cursorclass": DictCursor,
    "autocommit": False,
    "init_command": "SET time_zone = '+01:00'",
}

ESQUEMA = [
    "DROP TABLE IF EXISTS reservas",
    "DROP TABLE IF EXISTS salas",
    "DROP TABLE IF EXISTS empleados",
    """CREATE TABLE salas (
        id        SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        nombre    VARCHAR(50) NOT NULL UNIQUE,
        planta    TINYINT NOT NULL,
        capacidad SMALLINT UNSIGNED NOT NULL,
        equipos   JSON
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4""",
    """CREATE TABLE empleados (
        id           INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        nombre       VARCHAR(100) NOT NULL,
        email        VARCHAR(150) NOT NULL UNIQUE,
        departamento VARCHAR(50) NOT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4""",
    """CREATE TABLE reservas (
        id          INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        sala_id     SMALLINT UNSIGNED NOT NULL,
        empleado_id INT UNSIGNED NOT NULL,
        inicio      DATETIME NOT NULL,
        fin         DATETIME NOT NULL,
        asistentes  SMALLINT UNSIGNED NOT NULL,
        motivo      VARCHAR(200),
        CONSTRAINT ck_horario CHECK (fin > inicio),
        CONSTRAINT fk_res_sala FOREIGN KEY (sala_id) REFERENCES salas(id),
        CONSTRAINT fk_res_empleado FOREIGN KEY (empleado_id) REFERENCES empleados(id),
        INDEX ix_sala_inicio (sala_id, inicio, fin)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4""",
]

SALAS = [
    ("Madrid", 1, 6, '{"pantalla": true, "videoconferencia": true}'),
    ("Sevilla", 1, 4, '{"pantalla": true}'),
    ("Valencia", 2, 12, '{"pantalla": true, "videoconferencia": true, "pizarra": true}'),
    ("Bilbao", 2, 20, '{"proyector": true, "videoconferencia": true}'),
]
DEPARTAMENTOS = ["Ventas", "Marketing", "Finanzas", "Tecnología", "RR. HH."]


class ReservaNoDisponible(Exception):
    pass


@contextmanager
def transaccion(conexion):
    conexion.begin()
    try:
        with conexion.cursor() as cur:
            yield cur
        conexion.commit()
    except Exception:
        conexion.rollback()
        raise


def crear_esquema(conexion) -> None:
    with conexion.cursor() as cur:
        for sentencia in ESQUEMA:
            cur.execute(sentencia)
        cur.executemany("INSERT INTO salas (nombre, planta, capacidad, equipos) VALUES (%s, %s, %s, %s)", SALAS)
        random.seed(3)
        empleados = [(f"Empleado {i:03d}", f"empleado{i}@empresa.com", random.choice(DEPARTAMENTOS))
                     for i in range(1, 81)]
        cur.executemany("INSERT INTO empleados (nombre, email, departamento) VALUES (%s, %s, %s)", empleados)
    conexion.commit()


def reservar(conexion, sala: str, empleado_id: int, inicio: datetime, minutos: int,
             asistentes: int, motivo: str = "") -> int:
    fin = inicio + timedelta(minutes=minutos)
    if inicio.time() < time(8) or fin.time() > time(20) or fin.date() != inicio.date():
        raise ReservaNoDisponible("Fuera del horario de 08:00 a 20:00")

    with transaccion(conexion) as cur:
        cur.execute("SELECT id, capacidad FROM salas WHERE nombre = %s FOR UPDATE", (sala,))
        datos_sala = cur.fetchone()
        if datos_sala is None:
            raise ReservaNoDisponible(f"La sala {sala} no existe")
        if asistentes > datos_sala["capacidad"]:
            raise ReservaNoDisponible(f"Capacidad máxima de {sala}: {datos_sala['capacidad']} personas")

        cur.execute("""
            SELECT r.inicio, r.fin, e.nombre
            FROM reservas r JOIN empleados e ON e.id = r.empleado_id
            WHERE r.sala_id = %s AND r.inicio < %s AND r.fin > %s
            LIMIT 1
        """, (datos_sala["id"], fin, inicio))
        conflicto = cur.fetchone()
        if conflicto:
            raise ReservaNoDisponible(
                f"{sala} ocupada de {conflicto['inicio']:%H:%M} a {conflicto['fin']:%H:%M} ({conflicto['nombre']})")

        cur.execute("""
            INSERT INTO reservas (sala_id, empleado_id, inicio, fin, asistentes, motivo)
            VALUES (%s, %s, %s, %s, %s, %s)
        """, (datos_sala["id"], empleado_id, inicio, fin, asistentes, motivo))
        return cur.lastrowid


def salas_libres(conexion, inicio: datetime, minutos: int, asistentes: int,
                 requiere_video: bool = False) -> list[dict]:
    fin = inicio + timedelta(minutes=minutos)
    filtro_video = "AND JSON_EXTRACT(s.equipos, '$.videoconferencia') = true" if requiere_video else ""
    with conexion.cursor() as cur:
        cur.execute(f"""
            SELECT s.nombre, s.planta, s.capacidad
            FROM salas s
            WHERE s.capacidad >= %s {filtro_video}
              AND NOT EXISTS (SELECT 1 FROM reservas r
                              WHERE r.sala_id = s.id AND r.inicio < %s AND r.fin > %s)
            ORDER BY s.capacidad
        """, (asistentes, fin, inicio))
        return cur.fetchall()


def ocupacion_semanal(conexion, lunes: date) -> list[dict]:
    with conexion.cursor() as cur:
        cur.execute("""
            SELECT s.nombre AS sala,
                   COUNT(r.id) AS reservas,
                   ROUND(COALESCE(SUM(TIMESTAMPDIFF(MINUTE, r.inicio, r.fin)), 0) / 60, 1) AS horas,
                   ROUND(100 * COALESCE(SUM(TIMESTAMPDIFF(MINUTE, r.inicio, r.fin)), 0) / (5 * 12 * 60), 1) AS ocupacion
            FROM salas s
            LEFT JOIN reservas r ON r.sala_id = s.id AND r.inicio >= %s AND r.inicio < %s
            GROUP BY s.id, s.nombre
            ORDER BY ocupacion DESC
        """, (lunes, lunes + timedelta(days=5)))
        return cur.fetchall()


def reservas_por_departamento(conexion) -> list[dict]:
    with conexion.cursor() as cur:
        cur.execute("""
            SELECT e.departamento, COUNT(*) AS reservas, ROUND(AVG(r.asistentes), 1) AS media_asistentes
            FROM reservas r JOIN empleados e ON e.id = r.empleado_id
            GROUP BY e.departamento ORDER BY reservas DESC
        """)
        return cur.fetchall()


def exportar(conexion, ruta: str) -> int:
    with conexion.cursor(pymysql.cursors.SSCursor) as cur, open(ruta, "w", encoding="utf-8-sig", newline="") as f:
        cur.execute("""
            SELECT r.id, s.nombre, e.nombre, e.departamento,
                   DATE_FORMAT(r.inicio, '%%d/%%m/%%Y %%H:%%i'), DATE_FORMAT(r.fin, '%%H:%%i'),
                   r.asistentes, r.motivo
            FROM reservas r JOIN salas s ON s.id = r.sala_id JOIN empleados e ON e.id = r.empleado_id
            ORDER BY r.inicio
        """, ())          # Con parámetros (aunque sea una tupla vacía), %% se convierte en %
        w = csv.writer(f, delimiter=";")
        w.writerow(["id", "sala", "empleado", "departamento", "inicio", "fin", "asistentes", "motivo"])
        n = 0
        for fila in cur:
            w.writerow(fila)
            n += 1
    return n


def main() -> None:
    with pymysql.connect(**CONFIG) as conexion:
        print("Servidor:", conexion.get_server_info())
        crear_esquema(conexion)

        lunes = date(2026, 11, 30)
        random.seed(5)
        ok = rechazadas = 0
        for _ in range(150):
            dia = lunes + timedelta(days=random.randint(0, 4))
            inicio = datetime.combine(dia, time(random.randint(8, 18), random.choice([0, 30])))
            try:
                reservar(conexion, random.choice(SALAS)[0], random.randint(1, 80), inicio,
                         random.choice([30, 60, 90, 120]), random.randint(2, 14), "Reunión")
                ok += 1
            except ReservaNoDisponible:
                rechazadas += 1
        print(f"Reservas: {ok} confirmadas, {rechazadas} rechazadas\n")

        try:
            reservar(conexion, "Sevilla", 1, datetime.combine(lunes, time(10)), 60, 10)
        except ReservaNoDisponible as e:
            print("Ejemplo de rechazo:", e)

        hueco = datetime.combine(lunes, time(9))
        print(f"\nSalas libres el {hueco:%d/%m %H:%M} (1 h, 5 personas, con videoconferencia):")
        for s in salas_libres(conexion, hueco, 60, 5, requiere_video=True):
            print(f"  {s['nombre']:<9} planta {s['planta']}  capacidad {s['capacidad']}")

        print(f"\nOcupación semana del {lunes:%d/%m/%Y}:")
        for r in ocupacion_semanal(conexion, lunes):
            barra = "█" * int(r["ocupacion"] / 5)
            print(f"  {r['sala']:<9} {r['reservas']:>3} reservas {r['horas']:>5} h {r['ocupacion']:>5} % {barra}")

        print("\nReservas por departamento:")
        for r in reservas_por_departamento(conexion):
            print(f"  {r['departamento']:<11} {r['reservas']:>3}  (media {r['media_asistentes']} asistentes)")

        print(f"\nExportadas {exportar(conexion, 'reservas.csv')} reservas a reservas.csv")


if __name__ == "__main__":
    main()
```

---

## 15. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `OperationalError: (2003, "Can't connect to MySQL server")` | Servidor parado, host/puerto incorrectos o cortafuegos. | Compruebe el servicio y la red. |
| `OperationalError: (1045, "Access denied for user")` | Credenciales o permisos incorrectos. | Revise usuario, contraseña y `GRANT`. |
| `RuntimeError: 'cryptography' package is required for sha256_password or caching_sha2_password` | Autenticación de MySQL 8. | `pip install "pymysql[rsa]"`. |
| Los cambios no se guardan | `autocommit=False` y falta `commit()`. | Llame a `conexion.commit()`. |
| `IntegrityError: (1062, "Duplicate entry")` | Valor repetido en clave única. | `INSERT IGNORE`, `ON DUPLICATE KEY UPDATE` o capture el error. |
| `OperationalError: (2006, 'MySQL server has gone away')` | Conexión caducada o paquete demasiado grande. | Compruebe con `ping()` y reconecte; inserte por lotes. |
| `DeprecationWarning: The 'reconnect' argument is deprecated` | Uso de `ping(reconnect=True)` en PyMySQL 1.1+. | Cree una conexión nueva si `ping()` falla (sección 11.1). |
| `ValueError: unsupported format character` | `%` literal en una consulta con parámetros. | Escriba `%%`. |
| `TypeError: not all arguments converted during string formatting` | Parámetro único sin tupla. | Use `(valor,)`. |
| Caracteres o emojis incorrectos (`???`) | Codificación `utf8` (3 bytes) en tabla o conexión. | Use `utf8mb4` en conexión, base de datos y tablas. |
| `ProgrammingError: (2014, "Commands out of sync")` | Se ejecutó otra consulta sin leer el resultado de un cursor sin búfer. | Lea todo el resultado o cierre el cursor antes. |
| `OperationalError: (1213, 'Deadlock found')` | Interbloqueo entre transacciones. | Reintente la transacción. |
| Búsquedas que no distinguen tildes cuando sí debería | Intercalación `_ai_ci`. | Use una intercalación `_as_cs` o `utf8mb4_bin` en esa columna. |

---

## 16. Buenas prácticas

- Use siempre `charset="utf8mb4"` y tablas `utf8mb4` con motor InnoDB.
- Pase los valores como parámetros `%s`; valide con listas permitidas los nombres de columnas o tablas dinámicos.
- Guarde las credenciales en variables de entorno.
- Agrupe las operaciones relacionadas en transacciones y haga `rollback()` ante errores.
- Use `DictCursor` para mayor legibilidad y `SSCursor` para grandes resultados.
- Use `executemany` por lotes o `LOAD DATA` para cargas masivas.
- En procesos largos, compruebe la conexión con `ping()` y cree una nueva si se ha perdido (o use un pool).
- Use `DECIMAL` para importes y defina claves foráneas y restricciones en el esquema.
- Para aplicaciones con muchas peticiones, use un pool (SQLAlchemy).
- Recuerde que DDL hace *commit* implícito: no mezcle cambios de esquema y datos en la misma transacción.

---

## 17. Referencia rápida

```python
con = pymysql.connect(host=, port=3306, user=, password=, database=, charset="utf8mb4",
                      cursorclass=pymysql.cursors.DictCursor, autocommit=False)
with con:                                   # Cierra al salir
    with con.cursor() as cur:
        cur.execute("... WHERE a = %s AND b IN %s", (v, (1, 2, 3)))
        cur.executemany("INSERT ... VALUES (%s, %s)", filas)
        cur.fetchone() / fetchmany(n) / fetchall() / cur.lastrowid / cur.rowcount
        cur.mogrify(sql, params) / cur.callproc("proc", args)
    con.commit() / con.rollback() / con.begin() / con.ping()

pymysql.cursors.Cursor | DictCursor | SSCursor | SSDictCursor
pymysql.err.IntegrityError | OperationalError | ProgrammingError | DataError   (e.args[0] = código)
"mysql+pymysql://u:c@host:3306/bd?charset=utf8mb4"      # SQLAlchemy / Pandas
```

---

## 18. Ejercicios propuestos

1. Cree una base de datos de biblioteca con libros, socios y préstamos, y una función que impida prestar un libro ya prestado.
2. Importe un CSV de 200 000 filas comparando `executemany` por lotes con `LOAD DATA LOCAL INFILE`.
3. Escriba una función de búsqueda de clientes con filtros opcionales y ordenación segura por columna.
4. Almacene configuraciones de usuario en una columna `JSON` y consulte las de quienes tengan el modo oscuro activado.
5. Implemente una clase de acceso a datos con reconexión automática y reintentos ante interbloqueos.

---

## 19. Referencias

- Documentación oficial: https://pymysql.readthedocs.io/
- Repositorio: https://github.com/PyMySQL/PyMySQL
- Manual de MySQL 8: https://dev.mysql.com/doc/refman/8.4/en/
- Documentación de MariaDB: https://mariadb.com/kb/en/documentation/
