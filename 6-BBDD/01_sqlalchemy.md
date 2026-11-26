# Manual de uso: SQLAlchemy

> **Categoría:** Bases de datos · **Utilidad:** Conectar Python con bases de datos mediante ORM y SQL
> **Tipo:** Librería externa · **Versión de referencia:** SQLAlchemy 2.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Conexión: Engine y URL](#4-conexión-engine-y-url)
5. [SQL directo con Core](#5-sql-directo-con-core)
6. [Definir modelos (ORM)](#6-definir-modelos-orm)
7. [Sesiones y CRUD](#7-sesiones-y-crud)
8. [Consultas con select()](#8-consultas-con-select)
9. [Relaciones](#9-relaciones)
10. [Uniones, agregaciones y subconsultas](#10-uniones-agregaciones-y-subconsultas)
11. [Transacciones](#11-transacciones)
12. [Rendimiento](#12-rendimiento)
13. [Migraciones con Alembic](#13-migraciones-con-alembic)
14. [Integración con Pandas y FastAPI](#14-integración-con-pandas-y-fastapi)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**SQLAlchemy** es el kit de herramientas SQL y ORM (*Object Relational Mapper*) de referencia en Python. Permite trabajar con bases de datos relacionales de dos formas complementarias:

| Capa | Uso |
|---|---|
| **Core** | Construir y ejecutar SQL desde Python con independencia del motor. |
| **ORM** | Representar tablas como **clases** y filas como **objetos**. |

**Motores compatibles:** SQLite, PostgreSQL, MySQL/MariaDB, SQL Server, Oracle y otros mediante complementos (DuckDB, Snowflake, BigQuery…).

**Ventajas:**
- El mismo código funciona con distintos motores cambiando solo la URL de conexión.
- Protección frente a inyección SQL mediante parámetros.
- Gestión de conexiones (*pool*), transacciones y relaciones entre tablas.
- Integración con Pandas, FastAPI, Flask, Alembic (migraciones).

> Este manual usa la sintaxis moderna de **SQLAlchemy 2.0** (`select()`, `Mapped`, `mapped_column`). Muchos tutoriales antiguos usan `session.query()`, que sigue funcionando pero se considera la API heredada.

---

## 2. Instalación e importación

```bash
pip install sqlalchemy
```

Controlador según el motor:

| Motor | Paquete | URL de ejemplo |
|---|---|---|
| SQLite | Incluido en Python | `sqlite:///empresa.db` |
| PostgreSQL | `pip install "psycopg[binary]"` | `postgresql+psycopg://usuario:clave@host:5432/bd` |
| MySQL / MariaDB | `pip install pymysql` | `mysql+pymysql://usuario:clave@host:3306/bd?charset=utf8mb4` |
| SQL Server | `pip install pyodbc` | `mssql+pyodbc://usuario:clave@dsn` |
| Oracle | `pip install oracledb` | `oracle+oracledb://usuario:clave@host:1521/?service_name=XE` |

```python
from sqlalchemy import create_engine, select, insert, update, delete, text, func
from sqlalchemy import String, Integer, Numeric, DateTime, ForeignKey, Boolean
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session, sessionmaker
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Engine` | Punto de entrada a la base de datos; gestiona el *pool* de conexiones. Uno por aplicación. |
| `Connection` | Conexión concreta (Core). |
| `Session` | Unidad de trabajo del ORM: registra cambios y los confirma con `commit()`. |
| `DeclarativeBase` | Clase base de la que heredan los modelos. |
| Modelo | Clase Python asociada a una tabla. |
| `Mapped[tipo]` | Anotación del tipo de cada columna. |
| `mapped_column()` | Configuración de la columna (clave primaria, longitud, índices…). |
| `relationship()` | Relación entre modelos (1:N, N:M). |
| `select()` | Construye consultas de lectura. |
| Metadatos | Descripción del esquema (`Base.metadata`). |

---

## 4. Conexión: Engine y URL

### 4.1 Crear el motor

```python
engine = create_engine("sqlite:///empresa.db", echo=False)
```

| Parámetro | Efecto |
|---|---|
| `echo=True` | Muestra el SQL generado (útil en desarrollo). |
| `pool_size=5` | Conexiones mantenidas en el *pool*. |
| `max_overflow=10` | Conexiones adicionales permitidas en picos. |
| `pool_pre_ping=True` | Comprueba la conexión antes de usarla (evita errores por conexiones caducadas). |
| `pool_recycle=3600` | Renueva conexiones cada hora (recomendado en MySQL). |

### 4.2 Construir la URL de forma segura

Las contraseñas con caracteres especiales (`@`, `:`, `/`, `#`) rompen la URL. Use `URL.create`:

```python
import os
from sqlalchemy import URL

url = URL.create(
    "postgresql+psycopg",
    username=os.environ["DB_USUARIO"],
    password=os.environ["DB_CLAVE"],
    host=os.getenv("DB_HOST", "localhost"),
    port=int(os.getenv("DB_PUERTO", "5432")),
    database=os.environ["DB_NOMBRE"],
)
engine = create_engine(url, pool_pre_ping=True)
```

> No escriba credenciales en el código: use variables de entorno (por ejemplo, con **python-dotenv**).

### 4.3 SQLite: rutas y memoria

```python
create_engine("sqlite:///datos/empresa.db")          # Ruta relativa
create_engine("sqlite:////home/ana/empresa.db")      # Ruta absoluta (Linux: 4 barras)
create_engine(r"sqlite:///C:\datos\empresa.db")      # Ruta absoluta (Windows)
create_engine("sqlite://")                           # Base de datos en memoria
```

### 4.4 Comprobar la conexión

```python
with engine.connect() as conn:
    print(conn.execute(text("SELECT 1")).scalar())
```

---

## 5. SQL directo con Core

### 5.1 Consultas con `text()`

```python
with engine.connect() as conn:
    resultado = conn.execute(text("SELECT id, nombre FROM clientes WHERE ciudad = :ciudad"),
                             {"ciudad": "Madrid"})
    for fila in resultado:
        print(fila.id, fila.nombre)          # Acceso por nombre
```

> Use **siempre** parámetros (`:nombre`). Nunca construya SQL concatenando datos del usuario.

### 5.2 Formas de leer el resultado

```python
r = conn.execute(text("SELECT * FROM clientes"))

r.all()                 # Lista de filas
r.first()               # Primera fila o None
r.one()                 # Exactamente una fila (error si hay 0 o varias)
r.one_or_none()
r.scalar()              # Primera columna de la primera fila
r.scalars().all()       # Lista con la primera columna de cada fila
r.mappings().all()      # Lista de diccionarios
```

### 5.3 Escribir datos

```python
with engine.begin() as conn:                  # begin() confirma automáticamente al salir
    conn.execute(
        text("INSERT INTO clientes (nombre, ciudad) VALUES (:nombre, :ciudad)"),
        [{"nombre": "Ana", "ciudad": "Madrid"}, {"nombre": "Luis", "ciudad": "Sevilla"}],  # Inserción múltiple
    )
```

### 5.4 Tablas Core (sin ORM)

```python
from sqlalchemy import MetaData, Table, Column

metadata = MetaData()
productos = Table(
    "productos", metadata,
    Column("id", Integer, primary_key=True),
    Column("nombre", String(80), nullable=False),
    Column("precio", Numeric(10, 2)),
)
metadata.create_all(engine)

with engine.begin() as conn:
    conn.execute(insert(productos), [{"nombre": "Teclado", "precio": 29.9}])
    filas = conn.execute(select(productos).where(productos.c.precio > 20)).all()
```

### 5.5 Reflejar tablas existentes

```python
metadata = MetaData()
metadata.reflect(bind=engine)                     # Carga todas las tablas de la BD
clientes = metadata.tables["clientes"]

ventas = Table("ventas", MetaData(), autoload_with=engine)   # Una tabla concreta
print(ventas.columns.keys())
```

Inspeccionar el esquema:

```python
from sqlalchemy import inspect
ins = inspect(engine)
ins.get_table_names()
ins.get_columns("clientes")
ins.get_foreign_keys("pedidos")
```

---

## 6. Definir modelos (ORM)

### 6.1 Clase base y modelo

```python
from datetime import datetime
from decimal import Decimal


class Base(DeclarativeBase):
    pass


class Cliente(Base):
    __tablename__ = "clientes"

    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(120), unique=True, index=True)
    ciudad: Mapped[str | None] = mapped_column(String(60))          # Opcional (NULL)
    activo: Mapped[bool] = mapped_column(default=True)
    saldo: Mapped[Decimal] = mapped_column(Numeric(12, 2), default=0)
    creado: Mapped[datetime] = mapped_column(server_default=func.now())

    def __repr__(self) -> str:
        return f"<Cliente {self.id} {self.nombre}>"
```

| Tipo Python | Tipo SQL por defecto |
|---|---|
| `int` | INTEGER |
| `str` | VARCHAR (indique longitud con `String(n)`) |
| `float` | FLOAT |
| `Decimal` | NUMERIC |
| `bool` | BOOLEAN |
| `datetime` / `date` | DATETIME / DATE |
| `X \| None` | Columna que admite NULL |

### 6.2 Opciones de `mapped_column`

```python
mapped_column(
    String(50),
    primary_key=False,
    nullable=False,
    unique=True,
    index=True,
    default="valor",                 # Valor por defecto en Python
    server_default="valor",          # Valor por defecto en la BD
    onupdate=func.now(),             # Al actualizar
    comment="Descripción de la columna",
)
```

### 6.3 Restricciones e índices compuestos

```python
from sqlalchemy import UniqueConstraint, CheckConstraint, Index


class Producto(Base):
    __tablename__ = "productos"
    __table_args__ = (
        UniqueConstraint("codigo", "proveedor_id", name="uq_codigo_proveedor"),
        CheckConstraint("precio >= 0", name="ck_precio_positivo"),
        Index("ix_categoria_precio", "categoria", "precio"),
    )

    id: Mapped[int] = mapped_column(primary_key=True)
    codigo: Mapped[str] = mapped_column(String(20))
    proveedor_id: Mapped[int]
    categoria: Mapped[str] = mapped_column(String(40))
    precio: Mapped[Decimal] = mapped_column(Numeric(10, 2))
```

### 6.4 Crear y eliminar tablas

```python
Base.metadata.create_all(engine)       # Crea las tablas que no existan
Base.metadata.drop_all(engine)         # Elimina todas (¡cuidado!)
```

> `create_all` **no modifica** tablas existentes. Para cambios de esquema use **Alembic** (sección 13).

### 6.5 Columnas reutilizables (mixins)

```python
class Auditoria:
    creado: Mapped[datetime] = mapped_column(server_default=func.now())
    modificado: Mapped[datetime] = mapped_column(server_default=func.now(), onupdate=func.now())


class Pedido(Auditoria, Base):
    __tablename__ = "pedidos"
    id: Mapped[int] = mapped_column(primary_key=True)
```

---

## 7. Sesiones y CRUD

### 7.1 Fábrica de sesiones

```python
SessionLocal = sessionmaker(bind=engine, expire_on_commit=False)

with SessionLocal() as session:
    ...
```

### 7.2 Crear (Create)

```python
with Session(engine) as session:
    ana = Cliente(nombre="Ana López", email="ana@ejemplo.com", ciudad="Madrid")
    session.add(ana)
    session.add_all([
        Cliente(nombre="Luis Gómez", email="luis@ejemplo.com", ciudad="Sevilla"),
        Cliente(nombre="Marta Ruiz", email="marta@ejemplo.com"),
    ])
    session.commit()
    print(ana.id)                     # El id se asigna tras el commit (o flush)
```

### 7.3 Leer (Read)

```python
with Session(engine) as session:
    cliente = session.get(Cliente, 1)                                    # Por clave primaria
    todos = session.scalars(select(Cliente)).all()
    madrid = session.scalars(select(Cliente).where(Cliente.ciudad == "Madrid")).all()
    uno = session.scalars(select(Cliente).where(Cliente.email == "ana@ejemplo.com")).one_or_none()
```

### 7.4 Actualizar (Update)

```python
with Session(engine) as session:
    cliente = session.get(Cliente, 1)
    cliente.ciudad = "Barcelona"
    session.commit()                  # SQLAlchemy detecta el cambio y genera el UPDATE
```

Actualización masiva:

```python
with Session(engine) as session:
    session.execute(update(Cliente).where(Cliente.ciudad.is_(None)).values(ciudad="Sin asignar"))
    session.commit()
```

### 7.5 Eliminar (Delete)

```python
with Session(engine) as session:
    cliente = session.get(Cliente, 3)
    if cliente:
        session.delete(cliente)
        session.commit()

    session.execute(delete(Cliente).where(Cliente.activo.is_(False)))
    session.commit()
```

### 7.6 Insertar o actualizar (*upsert*)

```python
# PostgreSQL
from sqlalchemy.dialects.postgresql import insert as pg_insert

stmt = pg_insert(Cliente).values(email="ana@ejemplo.com", nombre="Ana L.")
stmt = stmt.on_conflict_do_update(index_elements=["email"], set_={"nombre": stmt.excluded.nombre})
session.execute(stmt)

# SQLite: from sqlalchemy.dialects.sqlite import insert  (misma sintaxis)
# MySQL:  from sqlalchemy.dialects.mysql import insert;  .on_duplicate_key_update(nombre=...)
```

`session.merge(obj)` es una alternativa genérica basada en la clave primaria.

---

## 8. Consultas con select()

### 8.1 Filtros

```python
select(Cliente).where(Cliente.ciudad == "Madrid")
select(Cliente).where(Cliente.ciudad != "Madrid")
select(Cliente).where(Cliente.saldo > 1000)
select(Cliente).where(Cliente.saldo.between(100, 500))
select(Cliente).where(Cliente.nombre.like("A%"))
select(Cliente).where(Cliente.nombre.ilike("%lópez%"))          # Sin distinguir mayúsculas
select(Cliente).where(Cliente.nombre.contains("Ana"))
select(Cliente).where(Cliente.nombre.startswith("A"))
select(Cliente).where(Cliente.ciudad.in_(["Madrid", "Sevilla"]))
select(Cliente).where(Cliente.ciudad.not_in(["Madrid"]))
select(Cliente).where(Cliente.ciudad.is_(None))
select(Cliente).where(Cliente.ciudad.is_not(None))
```

### 8.2 Combinar condiciones

```python
from sqlalchemy import and_, or_, not_

select(Cliente).where(Cliente.ciudad == "Madrid", Cliente.activo)               # AND implícito
select(Cliente).where(and_(Cliente.saldo > 100, Cliente.activo))
select(Cliente).where(or_(Cliente.ciudad == "Madrid", Cliente.ciudad == "Sevilla"))
select(Cliente).where(not_(Cliente.activo))
```

### 8.3 Ordenar, limitar y paginar

```python
select(Cliente).order_by(Cliente.nombre)
select(Cliente).order_by(Cliente.saldo.desc(), Cliente.nombre)
select(Cliente).limit(10).offset(20)                     # Página 3 de 10 en 10
select(Cliente).distinct()
```

### 8.4 Seleccionar columnas concretas

```python
filas = session.execute(select(Cliente.nombre, Cliente.email).where(Cliente.activo)).all()
for nombre, email in filas:
    ...

nombres = session.scalars(select(Cliente.nombre)).all()     # Lista de valores
```

### 8.5 Contar y comprobar existencia

```python
total = session.scalar(select(func.count()).select_from(Cliente))
hay_madrid = session.scalar(select(select(Cliente).where(Cliente.ciudad == "Madrid").exists()))
```

### 8.6 Consultas dinámicas (filtros opcionales)

```python
def buscar_clientes(session, ciudad=None, texto=None, saldo_min=None, pagina=1, tamano=20):
    consulta = select(Cliente)
    if ciudad:
        consulta = consulta.where(Cliente.ciudad == ciudad)
    if texto:
        consulta = consulta.where(Cliente.nombre.ilike(f"%{texto}%"))
    if saldo_min is not None:
        consulta = consulta.where(Cliente.saldo >= saldo_min)
    consulta = consulta.order_by(Cliente.nombre).limit(tamano).offset((pagina - 1) * tamano)
    return session.scalars(consulta).all()
```

### 8.7 Ver el SQL generado

```python
consulta = select(Cliente).where(Cliente.ciudad == "Madrid")
print(consulta)
print(consulta.compile(engine, compile_kwargs={"literal_binds": True}))   # Con los valores
```

---

## 9. Relaciones

### 9.1 Uno a muchos (1:N)

Un cliente tiene muchos pedidos; cada pedido pertenece a un cliente.

```python
class Cliente(Base):
    __tablename__ = "clientes"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(100))

    pedidos: Mapped[list["Pedido"]] = relationship(
        back_populates="cliente", cascade="all, delete-orphan", order_by="Pedido.fecha"
    )


class Pedido(Base):
    __tablename__ = "pedidos"
    id: Mapped[int] = mapped_column(primary_key=True)
    fecha: Mapped[datetime] = mapped_column(server_default=func.now())
    total: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    cliente_id: Mapped[int] = mapped_column(ForeignKey("clientes.id", ondelete="CASCADE"))

    cliente: Mapped["Cliente"] = relationship(back_populates="pedidos")
```

Uso:

```python
with Session(engine) as session:
    ana = Cliente(nombre="Ana")
    ana.pedidos.append(Pedido(total=Decimal("59.80")))
    ana.pedidos.append(Pedido(total=Decimal("120.00")))
    session.add(ana)                  # Los pedidos se guardan en cascada
    session.commit()

    for p in ana.pedidos:
        print(p.id, p.total, p.cliente.nombre)
```

### 9.2 Muchos a muchos (N:M)

Un producto puede tener varias etiquetas y una etiqueta varios productos:

```python
from sqlalchemy import Table, Column

producto_etiqueta = Table(
    "producto_etiqueta", Base.metadata,
    Column("producto_id", ForeignKey("productos.id"), primary_key=True),
    Column("etiqueta_id", ForeignKey("etiquetas.id"), primary_key=True),
)


class Producto(Base):
    __tablename__ = "productos"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(80))
    etiquetas: Mapped[list["Etiqueta"]] = relationship(secondary=producto_etiqueta, back_populates="productos")


class Etiqueta(Base):
    __tablename__ = "etiquetas"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(40), unique=True)
    productos: Mapped[list["Producto"]] = relationship(secondary=producto_etiqueta, back_populates="etiquetas")
```

Si la tabla intermedia necesita columnas propias (cantidad, precio), créela como modelo completo (**objeto de asociación**), por ejemplo `LineaPedido` entre `Pedido` y `Producto`.

### 9.3 Uno a uno (1:1)

```python
class Empleado(Base):
    __tablename__ = "empleados"
    id: Mapped[int] = mapped_column(primary_key=True)
    ficha: Mapped["FichaEmpleado"] = relationship(back_populates="empleado", uselist=False)


class FichaEmpleado(Base):
    __tablename__ = "fichas"
    id: Mapped[int] = mapped_column(primary_key=True)
    empleado_id: Mapped[int] = mapped_column(ForeignKey("empleados.id"), unique=True)
    empleado: Mapped["Empleado"] = relationship(back_populates="ficha")
```

### 9.4 Opciones de cascada

| Valor | Efecto |
|---|---|
| `save-update` | Guarda los objetos relacionados al añadir el padre (por defecto). |
| `delete` | Elimina los hijos al eliminar el padre. |
| `delete-orphan` | Elimina un hijo cuando se separa de su padre. |
| `all` | `save-update, merge, refresh-expire, expunge, delete`. |

---

## 10. Uniones, agregaciones y subconsultas

### 10.1 JOIN

```python
consulta = (
    select(Cliente.nombre, Pedido.id, Pedido.total)
    .join(Pedido, Cliente.pedidos)                 # Usa la relación definida
    .where(Pedido.total > 50)
)
for nombre, id_pedido, total in session.execute(consulta):
    ...

select(Cliente).outerjoin(Cliente.pedidos)         # LEFT OUTER JOIN
select(Pedido).join(Cliente, Pedido.cliente_id == Cliente.id)   # Condición explícita
```

### 10.2 Agregaciones

```python
consulta = (
    select(
        Cliente.nombre,
        func.count(Pedido.id).label("num_pedidos"),
        func.sum(Pedido.total).label("facturado"),
        func.avg(Pedido.total).label("ticket_medio"),
        func.max(Pedido.fecha).label("ultimo_pedido"),
    )
    .join(Cliente.pedidos)
    .group_by(Cliente.id, Cliente.nombre)
    .having(func.sum(Pedido.total) > 100)
    .order_by(func.sum(Pedido.total).desc())
)
for fila in session.execute(consulta):
    print(fila.nombre, fila.num_pedidos, fila.facturado)
```

Funciones habituales: `func.count`, `func.sum`, `func.avg`, `func.min`, `func.max`, `func.coalesce`, `func.lower`, `func.upper`, `func.length`, `func.now`, `func.date`.

### 10.3 Expresiones condicionales

```python
from sqlalchemy import case

segmento = case(
    (func.sum(Pedido.total) >= 1000, "VIP"),
    (func.sum(Pedido.total) >= 300, "Habitual"),
    else_="Ocasional",
).label("segmento")
```

### 10.4 Subconsultas

```python
sub = (
    select(Pedido.cliente_id, func.sum(Pedido.total).label("total"))
    .group_by(Pedido.cliente_id)
    .subquery()
)
consulta = select(Cliente.nombre, sub.c.total).join(sub, Cliente.id == sub.c.cliente_id)

# Subconsulta escalar
media = select(func.avg(Pedido.total)).scalar_subquery()
grandes = select(Pedido).where(Pedido.total > media)
```

### 10.5 CTE (consultas WITH)

```python
ventas_cliente = (
    select(Pedido.cliente_id, func.sum(Pedido.total).label("total"))
    .group_by(Pedido.cliente_id)
    .cte("ventas_cliente")
)
consulta = select(Cliente.nombre, ventas_cliente.c.total).join(
    ventas_cliente, Cliente.id == ventas_cliente.c.cliente_id
)
```

### 10.6 Funciones de ventana

```python
ranking = func.rank().over(order_by=func.sum(Pedido.total).desc()).label("puesto")
consulta = (select(Cliente.nombre, func.sum(Pedido.total).label("total"), ranking)
            .join(Cliente.pedidos).group_by(Cliente.id, Cliente.nombre))
```

---

## 11. Transacciones

### 11.1 Confirmar y deshacer

```python
with Session(engine) as session:
    try:
        origen = session.get(Cuenta, 1)
        destino = session.get(Cuenta, 2)
        origen.saldo -= 100
        destino.saldo += 100
        session.commit()
    except Exception:
        session.rollback()
        raise
```

### 11.2 Bloque transaccional automático

```python
with Session(engine) as session, session.begin():
    ...                       # commit al salir; rollback si hay excepción
```

### 11.3 Puntos de guardado (transacciones anidadas)

```python
with Session(engine) as session, session.begin():
    session.add(Cliente(nombre="A", email="a@x.com"))
    try:
        with session.begin_nested():                     # SAVEPOINT
            session.add(Cliente(nombre="B", email="a@x.com"))   # Email duplicado
    except IntegrityError:
        pass                                             # Solo se deshace B
```

### 11.4 Bloqueo de filas

```python
cuenta = session.scalars(select(Cuenta).where(Cuenta.id == 1).with_for_update()).one()
```

### 11.5 `flush` frente a `commit`

| Operación | Efecto |
|---|---|
| `flush()` | Envía los cambios pendientes a la BD **dentro** de la transacción (por ejemplo, para obtener un `id`). |
| `commit()` | Hace `flush` y confirma la transacción de forma permanente. |
| `rollback()` | Deshace todo lo no confirmado. |

---

## 12. Rendimiento

### 12.1 El problema N+1 y la carga anticipada

```python
# ❌ 1 consulta para clientes + 1 consulta por cada cliente al acceder a .pedidos
for c in session.scalars(select(Cliente)):
    print(c.nombre, len(c.pedidos))

# ✅ 2 consultas en total
from sqlalchemy.orm import selectinload, joinedload

for c in session.scalars(select(Cliente).options(selectinload(Cliente.pedidos))):
    print(c.nombre, len(c.pedidos))

# ✅ 1 consulta con JOIN (ideal para relaciones N:1)
pedidos = session.scalars(select(Pedido).options(joinedload(Pedido.cliente))).all()
```

| Estrategia | Uso |
|---|---|
| `selectinload` | Colecciones (1:N, N:M). Recomendada por defecto. |
| `joinedload` | Relaciones a un único objeto (N:1, 1:1). |
| `lazy="raise"` | Provoca error si se accede sin cargar (detecta N+1 en desarrollo). |

### 12.2 Inserciones masivas

```python
datos = [{"nombre": f"Cliente {i}", "email": f"c{i}@x.com"} for i in range(10_000)]
with Session(engine) as session:
    session.execute(insert(Cliente), datos)       # Inserción masiva eficiente
    session.commit()
```

### 12.3 Recorrer grandes resultados

```python
for cliente in session.scalars(select(Cliente).execution_options(yield_per=1000)):
    procesar(cliente)
```

### 12.4 Índices

Cree índices en las columnas usadas en `WHERE`, `JOIN` y `ORDER BY` (`index=True` o `Index(...)`).

---

## 13. Migraciones con Alembic

Alembic gestiona los cambios de esquema (añadir columnas, tablas, índices) de forma versionada.

```bash
pip install alembic
alembic init migraciones
```

En `migraciones/env.py`:

```python
from mi_app.modelos import Base
target_metadata = Base.metadata
```

En `alembic.ini`, indique la URL (`sqlalchemy.url = ...`) o léala de variables de entorno en `env.py`.

```bash
alembic revision --autogenerate -m "crear tablas iniciales"   # Genera la migración comparando modelos y BD
alembic upgrade head                                          # Aplica las migraciones
alembic downgrade -1                                          # Deshace la última
alembic history                                               # Historial
alembic current                                               # Versión actual
```

> Revise siempre el archivo generado antes de aplicarlo: la autogeneración no detecta todo (por ejemplo, cambios de nombre de columna).

---

## 14. Integración con Pandas y FastAPI

### 14.1 Pandas

```python
import pandas as pd

df = pd.read_sql("SELECT * FROM clientes", engine)
df = pd.read_sql(select(Cliente.nombre, Cliente.ciudad).where(Cliente.activo), engine)
df = pd.read_sql(text("SELECT * FROM pedidos WHERE fecha >= :d"), engine, params={"d": "2026-01-01"})

df.to_sql("clientes_import", engine, if_exists="append", index=False, chunksize=1000)
```

| `if_exists` | Efecto |
|---|---|
| `"fail"` | Error si la tabla existe. |
| `"replace"` | Elimina y recrea la tabla. |
| `"append"` | Añade filas. |

### 14.2 FastAPI

```python
from fastapi import Depends, FastAPI, HTTPException

SessionLocal = sessionmaker(bind=engine, expire_on_commit=False)
app = FastAPI()


def obtener_sesion():
    with SessionLocal() as session:
        yield session


@app.get("/clientes/{id}")
def leer_cliente(id: int, session: Session = Depends(obtener_sesion)):
    cliente = session.get(Cliente, id)
    if not cliente:
        raise HTTPException(404, "Cliente no encontrado")
    return {"id": cliente.id, "nombre": cliente.nombre, "email": cliente.email}
```

### 14.3 Asíncrono

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

engine = create_async_engine("postgresql+psycopg://usuario:clave@localhost/bd")
SessionAsync = async_sessionmaker(engine, expire_on_commit=False)

async def listar():
    async with SessionAsync() as session:
        return (await session.scalars(select(Cliente))).all()
```

---

## 15. Ejemplo práctico completo

Sistema de pedidos con clientes, productos, pedidos y líneas (objeto de asociación), datos de ejemplo, transacción con control de stock, consultas de negocio y exportación a Excel.

```python
"""
Sistema de pedidos con SQLAlchemy 2.0.
Requisitos: pip install sqlalchemy pandas openpyxl
"""
from __future__ import annotations

from datetime import datetime, timedelta
from decimal import Decimal
import random

import pandas as pd
from sqlalchemy import (CheckConstraint, ForeignKey, Numeric, String, case, create_engine,
                        func, select)
from sqlalchemy.exc import IntegrityError
from sqlalchemy.orm import (DeclarativeBase, Mapped, Session, mapped_column, relationship,
                            selectinload)


class Base(DeclarativeBase):
    pass


class Cliente(Base):
    __tablename__ = "clientes"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(120), unique=True)
    ciudad: Mapped[str] = mapped_column(String(60), index=True)
    pedidos: Mapped[list[Pedido]] = relationship(back_populates="cliente", cascade="all, delete-orphan")


class Producto(Base):
    __tablename__ = "productos"
    __table_args__ = (CheckConstraint("stock >= 0", name="ck_stock_no_negativo"),)
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(80), unique=True)
    categoria: Mapped[str] = mapped_column(String(40))
    precio: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    stock: Mapped[int] = mapped_column(default=0)


class Pedido(Base):
    __tablename__ = "pedidos"
    id: Mapped[int] = mapped_column(primary_key=True)
    fecha: Mapped[datetime] = mapped_column(default=datetime.now)
    cliente_id: Mapped[int] = mapped_column(ForeignKey("clientes.id"))
    cliente: Mapped[Cliente] = relationship(back_populates="pedidos")
    lineas: Mapped[list[LineaPedido]] = relationship(back_populates="pedido", cascade="all, delete-orphan")

    @property
    def total(self) -> Decimal:
        return sum((l.subtotal for l in self.lineas), Decimal("0"))


class LineaPedido(Base):
    __tablename__ = "lineas_pedido"
    pedido_id: Mapped[int] = mapped_column(ForeignKey("pedidos.id"), primary_key=True)
    producto_id: Mapped[int] = mapped_column(ForeignKey("productos.id"), primary_key=True)
    cantidad: Mapped[int]
    precio_unitario: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    pedido: Mapped[Pedido] = relationship(back_populates="lineas")
    producto: Mapped[Producto] = relationship()

    @property
    def subtotal(self) -> Decimal:
        return self.precio_unitario * self.cantidad


class StockInsuficiente(Exception):
    pass


# ------------------------------------------------------------ Operaciones de negocio
def crear_pedido(session: Session, cliente_id: int, articulos: dict[int, int]) -> Pedido:
    """Crea un pedido descontando stock en una única transacción."""
    with session.begin_nested():
        pedido = Pedido(cliente_id=cliente_id)
        for producto_id, cantidad in articulos.items():
            producto = session.get(Producto, producto_id, with_for_update=True)
            if producto is None or producto.stock < cantidad:
                disponible = producto.stock if producto else 0
                raise StockInsuficiente(f"Producto {producto_id}: pedido {cantidad}, disponible {disponible}")
            producto.stock -= cantidad
            pedido.lineas.append(LineaPedido(producto=producto, cantidad=cantidad,
                                             precio_unitario=producto.precio))
        session.add(pedido)
    return pedido


def cargar_datos(session: Session) -> None:
    random.seed(1)
    ciudades = ["Madrid", "Sevilla", "Valencia", "Bilbao"]
    session.add_all([
        Cliente(nombre=f"Cliente {i:02d}", email=f"cliente{i}@ejemplo.com", ciudad=random.choice(ciudades))
        for i in range(1, 21)
    ])
    session.add_all([
        Producto(nombre="Teclado", categoria="Periféricos", precio=Decimal("29.90"), stock=200),
        Producto(nombre="Ratón", categoria="Periféricos", precio=Decimal("14.50"), stock=300),
        Producto(nombre="Monitor 27\"", categoria="Pantallas", precio=Decimal("189.00"), stock=40),
        Producto(nombre="Portátil", categoria="Equipos", precio=Decimal("749.00"), stock=15),
        Producto(nombre="Cable HDMI", categoria="Accesorios", precio=Decimal("9.90"), stock=500),
    ])
    session.flush()

    for _ in range(80):
        articulos = {random.randint(1, 5): random.randint(1, 3) for _ in range(random.randint(1, 3))}
        try:
            p = crear_pedido(session, random.randint(1, 20), articulos)
            p.fecha = datetime(2026, 11, 24) - timedelta(days=random.randint(0, 90))
        except StockInsuficiente:
            pass
    session.commit()


# ------------------------------------------------------------ Consultas
def ventas_por_cliente(session: Session) -> pd.DataFrame:
    total = func.sum(LineaPedido.cantidad * LineaPedido.precio_unitario)
    consulta = (
        select(
            Cliente.nombre, Cliente.ciudad,
            func.count(func.distinct(Pedido.id)).label("pedidos"),
            total.label("facturado"),
            case((total >= 1500, "VIP"), (total >= 500, "Habitual"), else_="Ocasional").label("segmento"),
        )
        .join(Cliente.pedidos).join(Pedido.lineas)
        .group_by(Cliente.id, Cliente.nombre, Cliente.ciudad)
        .order_by(total.desc())
    )
    return pd.read_sql(consulta, session.connection())


def ventas_por_categoria_y_mes(session: Session) -> pd.DataFrame:
    mes = func.strftime("%Y-%m", Pedido.fecha)              # SQLite; en PostgreSQL: func.to_char(...)
    consulta = (
        select(mes.label("mes"), Producto.categoria,
               func.sum(LineaPedido.cantidad).label("unidades"),
               func.sum(LineaPedido.cantidad * LineaPedido.precio_unitario).label("importe"))
        .select_from(LineaPedido).join(LineaPedido.pedido).join(LineaPedido.producto)
        .group_by(mes, Producto.categoria).order_by(mes, Producto.categoria)
    )
    return pd.read_sql(consulta, session.connection())


def ultimos_pedidos(session: Session, n: int = 5) -> list[Pedido]:
    consulta = (select(Pedido)
                .options(selectinload(Pedido.lineas).selectinload(LineaPedido.producto),
                         selectinload(Pedido.cliente))
                .order_by(Pedido.fecha.desc()).limit(n))
    return session.scalars(consulta).all()


# ------------------------------------------------------------ Principal
def main() -> None:
    engine = create_engine("sqlite:///tienda.db")
    Base.metadata.drop_all(engine)
    Base.metadata.create_all(engine)

    with Session(engine) as session:
        cargar_datos(session)

        print("Pedidos registrados:", session.scalar(select(func.count(Pedido.id))))
        print("\nÚltimos pedidos:")
        for p in ultimos_pedidos(session):
            detalle = ", ".join(f"{l.cantidad}× {l.producto.nombre}" for l in p.lineas)
            print(f"  #{p.id:<4} {p.fecha:%d/%m/%Y} {p.cliente.nombre:<11} {p.total:>9} €  ({detalle})")

        try:
            crear_pedido(session, 1, {4: 999})
            session.commit()
        except StockInsuficiente as e:
            session.rollback()
            print("\nPedido rechazado:", e)

        df_clientes = ventas_por_cliente(session)
        df_mes = ventas_por_categoria_y_mes(session)
        stock = pd.read_sql(select(Producto.nombre, Producto.stock).order_by(Producto.stock), session.connection())

    print("\nTop 5 clientes:\n", df_clientes.head().to_string(index=False))
    with pd.ExcelWriter("informe_ventas.xlsx") as excel:
        df_clientes.to_excel(excel, sheet_name="Clientes", index=False)
        df_mes.to_excel(excel, sheet_name="Categoría por mes", index=False)
        stock.to_excel(excel, sheet_name="Stock", index=False)
    print("\nInforme exportado: informe_ventas.xlsx")


if __name__ == "__main__":
    main()
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'psycopg'` (o `pymysql`) | Falta el controlador del motor. | Instálelo (sección 2). |
| `OperationalError: no such table` | No se crearon las tablas. | `Base.metadata.create_all(engine)` o Alembic. |
| `IntegrityError: UNIQUE constraint failed` | Valor duplicado en columna única. | Compruebe antes, use *upsert* o capture la excepción y haga `rollback()`. |
| `PendingRollbackError` | Se siguió usando la sesión tras un error. | Llame a `session.rollback()` tras cualquier excepción. |
| `DetachedInstanceError` | Acceso a una relación con la sesión cerrada. | Cargue la relación antes (`selectinload`) o use `expire_on_commit=False`. |
| `MultipleResultsFound` / `NoResultFound` | `.one()` con 0 o varias filas. | Use `.first()` o `.one_or_none()`. |
| `ArgumentError: Mapper ... could not assemble any primary key` | El modelo no tiene clave primaria. | Añada `primary_key=True`. |
| Error de URL con contraseñas con `@` o `#` | Caracteres especiales en la URL. | Use `URL.create(...)`. |
| `OperationalError: server has gone away` / conexión cerrada | Conexión inactiva caducada. | `pool_pre_ping=True` y `pool_recycle`. |
| Consultas muy lentas con relaciones | Problema N+1. | `selectinload` / `joinedload`. |
| `create_all` no añade una columna nueva | No modifica tablas existentes. | Use Alembic. |
| Avisos `LegacyAPIWarning` con `session.query()` | API 1.x. | Migre a `select()` + `session.scalars()`. |

---

## 17. Buenas prácticas

- Cree **un único** `Engine` por aplicación y reutilícelo.
- Use sesiones de vida corta (`with Session(engine) as session:`), una por operación o petición.
- Use parámetros siempre; nunca concatene datos en SQL.
- Guarde las credenciales en variables de entorno y construya la URL con `URL.create`.
- Use `Decimal` / `Numeric` para importes.
- Defina restricciones (`unique`, `CheckConstraint`, claves foráneas) en la base de datos, no solo en Python.
- Evite el problema N+1 con opciones de carga.
- Gestione los cambios de esquema con **Alembic**.
- Active `echo=True` en desarrollo para revisar el SQL generado.
- Agrupe en una sola transacción las operaciones que deben ser atómicas.

---

## 18. Referencia rápida

```python
engine = create_engine(url, echo=False, pool_pre_ping=True)
class Base(DeclarativeBase): pass
class M(Base):
    __tablename__ = "t"
    id: Mapped[int] = mapped_column(primary_key=True)
    campo: Mapped[str | None] = mapped_column(String(50), index=True)
    hijos: Mapped[list["H"]] = relationship(back_populates="padre", cascade="all, delete-orphan")
Base.metadata.create_all(engine)

with Session(engine) as s:
    s.add(obj); s.add_all([...]); s.commit(); s.rollback(); s.flush()
    s.get(M, id); s.delete(obj)
    s.scalars(select(M).where(...).order_by(...).limit(n)).all() | .first() | .one_or_none()
    s.execute(select(M.a, func.count()).join(...).group_by(...)).all()
    s.execute(update(M).where(...).values(...)); s.execute(delete(M).where(...))
    s.execute(insert(M), [dict, ...])

with engine.connect() as c: c.execute(text("SELECT ... :p"), {"p": v}).mappings().all()
with engine.begin() as c: ...                     # Transacción automática
selectinload / joinedload / with_for_update() / begin_nested()
pd.read_sql(consulta, engine) / df.to_sql("t", engine, if_exists="append", index=False)
```

---

## 19. Ejercicios propuestos

1. Modele una biblioteca con autores, libros (N:M con autores) y préstamos a socios.
2. Escriba una función de búsqueda de productos con filtros opcionales y paginación.
3. Importe un CSV de 50 000 filas con inserción masiva y mida el tiempo frente a insertar una a una.
4. Cree una consulta con el ranking mensual de los 3 productos más vendidos usando funciones de ventana.
5. Configure Alembic en un proyecto, añada una columna nueva a un modelo y genere y aplique la migración.

---

## 20. Referencias

- Documentación oficial: https://docs.sqlalchemy.org/en/20/
- Tutorial unificado 2.0: https://docs.sqlalchemy.org/en/20/tutorial/
- Alembic: https://alembic.sqlalchemy.org/
