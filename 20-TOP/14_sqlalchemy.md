# Manual de uso: SQLAlchemy

> **Categoría:** Bases de datos · **Utilidad:** ORM y kit de herramientas SQL para Python
> **Tipo:** Librería de terceros (`pip install sqlalchemy`) · **Versión de referencia:** 2.0 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y controladores](#2-instalación-y-controladores)
3. [Arquitectura: Core y ORM](#3-arquitectura-core-y-orm)
4. [Motor y conexión](#4-motor-y-conexión)
5. [Definición de modelos (ORM declarativo)](#5-definición-de-modelos-orm-declarativo)
6. [Relaciones](#6-relaciones)
7. [Sesiones y transacciones](#7-sesiones-y-transacciones)
8. [Crear, leer, actualizar y eliminar](#8-crear-leer-actualizar-y-eliminar)
9. [Consultas con `select()`](#9-consultas-con-select)
10. [Uniones, agregados y subconsultas](#10-uniones-agregados-y-subconsultas)
11. [Funciones de ventana y CTE](#11-funciones-de-ventana-y-cte)
12. [Carga de relaciones y problema N+1](#12-carga-de-relaciones-y-problema-n1)
13. [Operaciones masivas e *upsert*](#13-operaciones-masivas-e-upsert)
14. [Validación, propiedades híbridas y eventos](#14-validación-propiedades-híbridas-y-eventos)
15. [SQL Core y SQL textual](#15-sql-core-y-sql-textual)
16. [Reflexión de bases de datos existentes](#16-reflexión-de-bases-de-datos-existentes)
17. [Migraciones con Alembic](#17-migraciones-con-alembic)
18. [SQLAlchemy asíncrono](#18-sqlalchemy-asíncrono)
19. [Integración con pandas y frameworks web](#19-integración-con-pandas-y-frameworks-web)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**SQLAlchemy** es la librería de referencia para trabajar con bases de datos relacionales en Python. Ofrece dos niveles:

- **Core**: construcción de SQL con objetos Python, independiente del motor, con gestión de conexiones y transacciones.
- **ORM** (*Object Relational Mapper*): asigna tablas a clases y filas a objetos, con relaciones, sesiones y unidad de trabajo.

Funciona con SQLite, PostgreSQL, MySQL/MariaDB, Oracle y SQL Server, entre otros, y es la base de herramientas como Alembic (migraciones), Flask-SQLAlchemy, SQLModel o pandas (`read_sql`).

**Ventajas:**
- SQL expresivo y seguro (parámetros enlazados, sin concatenar cadenas).
- Código portable entre motores de base de datos.
- ORM potente sin ocultar el SQL: se puede ver, controlar y optimizar.
- Tipado estático completo en la versión 2 (`Mapped[...]`).
- *Pool* de conexiones, transacciones y soporte asíncrono.

**Limitaciones:**
- Curva de aprendizaje significativa (sesiones, estados de objetos, estrategias de carga).
- Para cargas analíticas masivas, SQL directo, pandas/Polars o DuckDB suelen ser más eficientes.
- No incluye migraciones (se usa Alembic).

| Si necesita… | Opción |
|---|---|
| ORM completo y SQL portable | **SQLAlchemy** |
| Modelos compartidos con FastAPI (Pydantic + ORM) | SQLModel (basado en SQLAlchemy) |
| ORM integrado en un *framework* web completo | ORM de Django |
| Consultas SQL sencillas sin ORM | `sqlite3`, `psycopg` |
| Análisis de datos sobre archivos | DuckDB, pandas |

---

## 2. Instalación y controladores

```bash
pip install sqlalchemy
pip install "psycopg[binary]"     # PostgreSQL
pip install pymysql               # MySQL / MariaDB
pip install pyodbc                # SQL Server
pip install aiosqlite asyncpg     # Controladores asíncronos
pip install alembic               # Migraciones
```

| Motor | URL de conexión |
|---|---|
| SQLite (archivo) | `sqlite:///ruta/datos.db` |
| SQLite (memoria) | `sqlite://` |
| PostgreSQL | `postgresql+psycopg://usuario:clave@servidor:5432/bd` |
| MySQL | `mysql+pymysql://usuario:clave@servidor:3306/bd?charset=utf8mb4` |
| SQL Server | `mssql+pyodbc://usuario:clave@dsn` |
| PostgreSQL asíncrono | `postgresql+asyncpg://…` |

> No escriba credenciales en el código: léalas de variables de entorno. Si la contraseña contiene caracteres especiales, construya la URL con `sqlalchemy.URL.create(...)`.

---

## 3. Arquitectura: Core y ORM

| Componente | Función |
|---|---|
| `Engine` | Punto de acceso a la base de datos; gestiona el *pool* de conexiones. |
| `Connection` | Conexión individual (Core). |
| `MetaData`, `Table`, `Column` | Esquema en Core. |
| `DeclarativeBase`, `Mapped`, `mapped_column` | Modelos ORM. |
| `Session` | Unidad de trabajo del ORM: rastrea objetos y los sincroniza con la base de datos. |
| `select`, `insert`, `update`, `delete` | Constructores de sentencias (comunes a Core y ORM). |

---

## 4. Motor y conexión

```python
from sqlalchemy import create_engine, text

motor = create_engine("sqlite:///tienda.db", echo=False)       # echo=True muestra el SQL generado
motor_pg = create_engine(url, pool_size=10, max_overflow=20, pool_pre_ping=True, pool_recycle=1800)

with motor.connect() as conexion:
    resultado = conexion.execute(text("SELECT sqlite_version()"))
    print(resultado.scalar_one())

with motor.begin() as conexion:                                  # Transacción con COMMIT automático
    conexion.execute(text("UPDATE cuentas SET saldo = saldo - :importe WHERE id = :id"),
                     {"importe": 50, "id": 1})
```

- `pool_pre_ping=True` comprueba las conexiones antes de usarlas (evita errores tras desconexiones del servidor).
- Cree **un único `Engine` por base de datos** y proceso, y reutilícelo.

---

## 5. Definición de modelos (ORM declarativo)

```python
from datetime import datetime
from decimal import Decimal
from sqlalchemy import String, Numeric, ForeignKey, CheckConstraint, UniqueConstraint, Index, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class Producto(Base):
    __tablename__ = "productos"
    __table_args__ = (
        CheckConstraint("precio >= 0", name="ck_precio_positivo"),
        Index("ix_productos_categoria_precio", "categoria", "precio"),
    )

    id: Mapped[int] = mapped_column(primary_key=True)
    sku: Mapped[str] = mapped_column(String(20), unique=True)
    nombre: Mapped[str] = mapped_column(String(120))
    categoria: Mapped[str] = mapped_column(String(40), index=True)
    precio: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    stock: Mapped[int] = mapped_column(default=0)
    descripcion: Mapped[str | None]                              # Opcional → NULL permitido
    activo: Mapped[bool] = mapped_column(default=True)
    creado: Mapped[datetime] = mapped_column(server_default=func.now())

    def __repr__(self) -> str:
        return f"Producto({self.sku!r}, {self.nombre!r})"


Base.metadata.create_all(motor)          # Crea las tablas que no existan (en producción: Alembic)
```

| Anotación | Tipo SQL por defecto |
|---|---|
| `Mapped[int]` | `INTEGER` |
| `Mapped[str]` | `VARCHAR` (indique longitud con `String(n)` para MySQL/SQL Server) |
| `Mapped[Decimal]` | `NUMERIC` |
| `Mapped[float]` | `FLOAT` |
| `Mapped[bool]` | `BOOLEAN` |
| `Mapped[datetime]`, `Mapped[date]` | `DATETIME`, `DATE` |
| `Mapped[X \| None]` | Columna que admite `NULL` |
| `Mapped[dict]` con `JSON` | `JSON` |
| `Mapped[Enum]` (de Python) | `ENUM` / `VARCHAR` |

Parámetros habituales de `mapped_column`: `primary_key`, `unique`, `index`, `nullable`, `default` (Python), `server_default` (SQL), `onupdate`, `comment`.

> Use `Decimal` + `Numeric` para importes monetarios; `float` introduce errores de redondeo.

---

## 6. Relaciones

```python
from sqlalchemy import Table, Column
from sqlalchemy.orm import relationship


class Cliente(Base):
    __tablename__ = "clientes"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(100))
    pedidos: Mapped[list["Pedido"]] = relationship(back_populates="cliente", cascade="all, delete-orphan")


class Pedido(Base):
    __tablename__ = "pedidos"
    id: Mapped[int] = mapped_column(primary_key=True)
    cliente_id: Mapped[int] = mapped_column(ForeignKey("clientes.id", ondelete="CASCADE"))
    cliente: Mapped["Cliente"] = relationship(back_populates="pedidos")         # Muchos a uno
    lineas: Mapped[list["Linea"]] = relationship(back_populates="pedido", cascade="all, delete-orphan")


# Muchos a muchos mediante tabla de asociación
producto_etiqueta = Table(
    "producto_etiqueta", Base.metadata,
    Column("producto_id", ForeignKey("productos.id"), primary_key=True),
    Column("etiqueta_id", ForeignKey("etiquetas.id"), primary_key=True),
)

class Etiqueta(Base):
    __tablename__ = "etiquetas"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(30), unique=True)
    productos: Mapped[list["Producto"]] = relationship(secondary=producto_etiqueta, back_populates="etiquetas")
```

| Tipo | Definición |
|---|---|
| Uno a muchos | `Mapped[list["Hijo"]] = relationship(back_populates=...)` en el padre + `ForeignKey` en el hijo. |
| Muchos a uno | `Mapped["Padre"] = relationship(...)` en el hijo. |
| Uno a uno | Como muchos a uno, con `Mapped["Otro"]` en ambos lados (y `unique=True` en la clave foránea). |
| Muchos a muchos | `relationship(secondary=tabla_asociacion)`. |
| Asociación con datos | Clase intermedia propia (p. ej., `Linea` con cantidad y precio). |

`cascade="all, delete-orphan"`: las operaciones sobre el padre se propagan a los hijos y un hijo retirado de la colección se elimina.

---

## 7. Sesiones y transacciones

```python
from sqlalchemy.orm import Session, sessionmaker

FabricaSesiones = sessionmaker(motor, expire_on_commit=False)

with FabricaSesiones() as sesion:            # Cierra la sesión al salir
    with sesion.begin():                      # COMMIT al salir; ROLLBACK si hay excepción
        sesion.add(Cliente(nombre="Ana"))

# Forma abreviada
with FabricaSesiones.begin() as sesion:
    sesion.add(Cliente(nombre="Luis"))
```

| Método | Efecto |
|---|---|
| `add(obj)`, `add_all([...])` | Marca objetos para insertar/actualizar. |
| `flush()` | Envía los cambios pendientes a la BD **dentro** de la transacción (obtiene claves generadas). |
| `commit()` / `rollback()` | Confirma / deshace la transacción. |
| `get(Modelo, id)` | Obtiene por clave primaria (usa la caché de la sesión). |
| `delete(obj)` | Marca para eliminar. |
| `refresh(obj)` | Recarga desde la BD. |
| `expunge(obj)` | Separa el objeto de la sesión. |
| `begin_nested()` | Punto de guardado (`SAVEPOINT`). |

> `expire_on_commit=False` mantiene los atributos cargados tras el `commit` (útil para devolver objetos después de cerrar la sesión). Con el valor por defecto (`True`), acceder a un atributo tras el `commit` vuelve a consultar la base de datos.

**Ciclo de vida de un objeto**: *transient* (nuevo) → *pending* (añadido) → *persistent* (en BD y en sesión) → *detached* (sesión cerrada) / *deleted*.

---

## 8. Crear, leer, actualizar y eliminar

```python
from sqlalchemy import select, update, delete

with FabricaSesiones.begin() as s:
    ana = Cliente(nombre="Ana")
    ana.pedidos.append(Pedido())              # Se inserta en cascada
    s.add(ana)
    s.flush()                                 # ana.id ya está disponible
    print(ana.id)

with FabricaSesiones() as s:
    cliente = s.get(Cliente, 1)                                       # Por clave primaria
    ana = s.scalars(select(Cliente).where(Cliente.nombre == "Ana")).first()
    unico = s.scalars(select(Producto).where(Producto.sku == "A-1")).one()        # Error si 0 o >1
    quizas = s.scalars(select(Producto).where(Producto.sku == "Z-9")).one_or_none()
    todos = s.scalars(select(Producto).order_by(Producto.nombre)).all()

with FabricaSesiones.begin() as s:
    p = s.get(Producto, 1)
    p.precio = Decimal("19.90")                                       # Actualización por la unidad de trabajo
    s.execute(update(Producto).where(Producto.categoria == "Oficina")
              .values(precio=Producto.precio * Decimal("1.05")))     # Actualización masiva
    s.delete(s.get(Cliente, 3))
    s.execute(delete(Producto).where(Producto.activo.is_(False)))
```

| Obtención del resultado | Devuelve |
|---|---|
| `s.scalars(stmt).all()` | Lista de objetos (primera columna de cada fila). |
| `s.scalars(stmt).first()` | Primer objeto o `None`. |
| `.one()` / `.one_or_none()` | Exactamente uno (o ninguno). |
| `s.scalar(stmt)` | Primer valor de la primera fila (agregados). |
| `s.execute(stmt).all()` | Lista de filas (`Row`, como tuplas con nombre). |
| `s.execute(stmt).mappings().all()` | Lista de diccionarios. |

---

## 9. Consultas con `select()`

```python
from sqlalchemy import select, and_, or_, not_, func, desc

stmt = (
    select(Producto)
    .where(Producto.categoria == "Oficina", Producto.precio.between(5, 50))   # Varias condiciones = AND
    .where(or_(Producto.stock > 0, Producto.nombre.ilike("%reserva%")))
    .order_by(Producto.precio.desc(), Producto.nombre)
    .limit(20).offset(40)                                                      # Paginación
)

select(Producto.nombre, Producto.precio)                     # Solo algunas columnas → filas
select(Producto).where(Producto.categoria.in_(["A", "B"]))
select(Producto).where(Producto.descripcion.is_(None))
select(Producto).where(Producto.nombre.contains("silla"))    # LIKE '%silla%'
select(Producto).where(Producto.nombre.startswith("Mesa"))
select(Producto.categoria).distinct()
select(func.count()).select_from(Producto)                  # Recuento

print(stmt)                                                 # Ver el SQL generado
print(stmt.compile(motor, compile_kwargs={"literal_binds": True}))   # Con valores (solo para depurar)
```

| Operador | Expresión |
|---|---|
| `==`, `!=`, `<`, `>=` | Comparaciones. |
| `.in_()`, `.not_in()` | Pertenencia. |
| `.is_(None)`, `.is_not(None)` | Nulos. |
| `.like()`, `.ilike()`, `.contains()`, `.startswith()` | Texto. |
| `.between(a, b)` | Rango. |
| `and_()`, `or_()`, `not_()` / `&`, `\|`, `~` | Lógicos. |

---

## 10. Uniones, agregados y subconsultas

```python
# JOIN siguiendo la relación definida
stmt = (select(Cliente.nombre, func.count(Pedido.id).label("pedidos"),
               func.coalesce(func.sum(Linea.cantidad * Linea.precio_unitario), 0).label("total"))
        .join(Cliente.pedidos)                     # INNER JOIN
        .join(Pedido.lineas)
        .group_by(Cliente.id)
        .having(func.count(Pedido.id) >= 2)
        .order_by(desc("total")))
for fila in s.execute(stmt):
    print(fila.nombre, fila.pedidos, fila.total)

select(Cliente).outerjoin(Cliente.pedidos).where(Pedido.id.is_(None))    # Clientes sin pedidos (LEFT JOIN)

# Subconsulta escalar correlacionada
ultimo = (select(func.max(Pedido.fecha)).where(Pedido.cliente_id == Cliente.id)
          .correlate(Cliente).scalar_subquery())
select(Cliente.nombre, ultimo.label("ultimo_pedido"))

# Subconsulta como tabla derivada
ventas = (select(Linea.producto_id, func.sum(Linea.cantidad).label("unidades"))
          .group_by(Linea.producto_id).subquery())
select(Producto.nombre, ventas.c.unidades).join(ventas, Producto.id == ventas.c.producto_id)

# EXISTS
select(Producto).where(Producto.lineas.any())                            # Productos vendidos alguna vez
select(Cliente).where(~Cliente.pedidos.any(Pedido.fecha >= desde))       # Sin pedidos recientes

# CASE
from sqlalchemy import case
tramo = case((Producto.precio < 10, "económico"), (Producto.precio < 100, "medio"), else_="premium")
select(tramo.label("tramo"), func.count()).group_by(tramo)
```

Funciones SQL con `func.<nombre>`: `count`, `sum`, `avg`, `min`, `max`, `coalesce`, `lower`, `round`, `date`, `strftime` (SQLite), `date_trunc` (PostgreSQL)…

---

## 11. Funciones de ventana y CTE

```python
# Ranking de productos por ventas dentro de cada categoría
ranking = func.rank().over(partition_by=Producto.categoria, order_by=func.sum(Linea.cantidad).desc())
stmt = (select(Producto.categoria, Producto.nombre, func.sum(Linea.cantidad).label("uds"), ranking.label("puesto"))
        .join(Producto.lineas).group_by(Producto.id))

# Acumulado
func.sum(Pedido.total).over(order_by=Pedido.fecha)

# CTE (WITH): ventas mensuales y variación respecto al mes anterior
mensual = (select(func.strftime("%Y-%m", Pedido.fecha).label("mes"), func.sum(Pedido.total).label("ventas"))
           .group_by("mes").cte("mensual"))
stmt = select(mensual.c.mes, mensual.c.ventas,
              (mensual.c.ventas - func.lag(mensual.c.ventas).over(order_by=mensual.c.mes)).label("variacion"))
```

Las CTE recursivas se crean con `.cte(recursive=True)` y `union_all` (jerarquías, árboles de categorías).

---

## 12. Carga de relaciones y problema N+1

Por defecto, las relaciones se cargan de forma **perezosa** (*lazy*): al acceder a `cliente.pedidos` se lanza una consulta. En un bucle de N clientes, esto genera N+1 consultas.

```python
from sqlalchemy.orm import selectinload, joinedload, raiseload

# 2 consultas en total: clientes + todos sus pedidos (IN (...))
clientes = s.scalars(select(Cliente).options(selectinload(Cliente.pedidos).selectinload(Pedido.lineas))).all()

# 1 consulta con JOIN: adecuado para relaciones muchos a uno
pedidos = s.scalars(select(Pedido).options(joinedload(Pedido.cliente))).all()

# Prohibir cargas perezosas no previstas (detecta N+1 durante el desarrollo)
s.scalars(select(Cliente).options(raiseload("*")))
```

| Estrategia | Uso recomendado |
|---|---|
| `selectinload` | Colecciones (uno a muchos, muchos a muchos). |
| `joinedload` | Referencias a un único objeto (muchos a uno). |
| `lazy="raise"` / `raiseload` | Forzar carga explícita; ideal con sesiones asíncronas o APIs. |
| `contains_eager` | Rellenar la relación con un `JOIN` que ya hace la consulta. |

---

## 13. Operaciones masivas e *upsert*

```python
from sqlalchemy import insert

# Inserción masiva eficiente (executemany) con el ORM
s.execute(insert(Producto), [
    {"sku": "A-1", "nombre": "Grapadora", "categoria": "Oficina", "precio": Decimal("8.50")},
    {"sku": "A-2", "nombre": "Tijeras", "categoria": "Oficina", "precio": Decimal("4.20")},
])

# Inserción devolviendo valores generados (PostgreSQL, SQLite ≥ 3.35, MariaDB, SQL Server)
ids = s.scalars(insert(Producto).returning(Producto.id), filas).all()

# Actualización masiva por clave primaria
s.execute(update(Producto), [{"id": 1, "stock": 10}, {"id": 2, "stock": 0}])

# UPSERT (INSERT … ON CONFLICT), específico de cada motor
from sqlalchemy.dialects.sqlite import insert as sqlite_insert        # postgresql.insert en PostgreSQL
stmt = sqlite_insert(Producto).values(filas)
stmt = stmt.on_conflict_do_update(index_elements=["sku"],
                                  set_={"precio": stmt.excluded.precio, "stock": stmt.excluded.stock})
s.execute(stmt)
```

En MySQL: `from sqlalchemy.dialects.mysql import insert` y `.on_duplicate_key_update(...)`.

---

## 14. Validación, propiedades híbridas y eventos

```python
from sqlalchemy.orm import validates
from sqlalchemy.ext.hybrid import hybrid_property

class Linea(Base):
    __tablename__ = "lineas"
    id: Mapped[int] = mapped_column(primary_key=True)
    cantidad: Mapped[int]
    precio_unitario: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    descuento: Mapped[Decimal] = mapped_column(Numeric(4, 2), default=Decimal("0"))

    @validates("cantidad")
    def _validar_cantidad(self, clave, valor):
        if valor <= 0:
            raise ValueError("La cantidad debe ser positiva")
        return valor

    @hybrid_property
    def importe(self) -> Decimal:                         # Funciona en Python y en consultas SQL
        return self.cantidad * self.precio_unitario * (1 - self.descuento)

s.scalars(select(Linea).where(Linea.importe > 100))      # Se traduce a SQL
```

Eventos:

```python
from sqlalchemy import event

@event.listens_for(Producto, "before_update")
def _registrar(mapper, conexion, producto):
    producto.modificado = datetime.now()

@event.listens_for(motor, "connect")
def _sqlite_fk(conexion_dbapi, _):
    conexion_dbapi.execute("PRAGMA foreign_keys = ON")    # SQLite no aplica claves foráneas por defecto
```

---

## 15. SQL Core y SQL textual

```python
from sqlalchemy import MetaData, Table, Column, Integer, String, text, bindparam

metadatos = MetaData()
registro = Table("registro", metadatos,
                 Column("id", Integer, primary_key=True),
                 Column("evento", String(50)), Column("usuario", String(50)))
metadatos.create_all(motor)

with motor.begin() as c:
    c.execute(registro.insert(), [{"evento": "login", "usuario": "ana"}])
    filas = c.execute(select(registro).where(registro.c.usuario == "ana")).all()

# SQL escrito a mano, siempre con parámetros enlazados
consulta = text("SELECT nombre, precio FROM productos WHERE categoria = :cat AND precio < :max")
with motor.connect() as c:
    for fila in c.execute(consulta, {"cat": "Oficina", "max": 20}):
        print(fila.nombre, fila.precio)

# Lista de valores con IN
c.execute(text("SELECT * FROM productos WHERE id IN :ids").bindparams(bindparam("ids", expanding=True)),
          {"ids": [1, 2, 3]})
```

> **Nunca** construya SQL con f-strings ni concatenación con datos de usuario: use parámetros (`:nombre`). Evita la inyección SQL.

---

## 16. Reflexión de bases de datos existentes

```python
from sqlalchemy import inspect, MetaData
from sqlalchemy.ext.automap import automap_base

inspector = inspect(motor)
inspector.get_table_names()
inspector.get_columns("productos") ; inspector.get_foreign_keys("pedidos")

metadatos = MetaData(); metadatos.reflect(motor)          # Tablas como objetos Table
clientes = metadatos.tables["clientes"]

AutoBase = automap_base(); AutoBase.prepare(autoload_with=motor)   # Clases ORM generadas automáticamente
Cliente = AutoBase.classes.clientes
```

---

## 17. Migraciones con Alembic

`create_all` solo crea tablas nuevas; no modifica las existentes. Para evolucionar el esquema en producción use **Alembic**:

```bash
pip install alembic
alembic init migraciones
# En migraciones/env.py: from mi_app.modelos import Base; target_metadata = Base.metadata
# En alembic.ini: sqlalchemy.url = sqlite:///tienda.db
alembic revision --autogenerate -m "Añadir columna descuento"
alembic upgrade head            # Aplicar
alembic downgrade -1            # Deshacer la última
alembic history ; alembic current
```

Revise siempre el script autogenerado antes de aplicarlo (renombrados, tipos y datos no se detectan perfectamente).

---

## 18. SQLAlchemy asíncrono

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

motor = create_async_engine("sqlite+aiosqlite:///tienda.db")
FabricaAsync = async_sessionmaker(motor, expire_on_commit=False)

async def productos_baratos() -> list[Producto]:
    async with FabricaAsync() as s:
        resultado = await s.scalars(select(Producto).where(Producto.precio < 10)
                                    .options(selectinload(Producto.etiquetas)))
        return list(resultado)

async def crear_tablas():
    async with motor.begin() as c:
        await c.run_sync(Base.metadata.create_all)
```

En modo asíncrono, la carga perezosa implícita **no** está permitida: cargue las relaciones con `selectinload`/`joinedload`, o use `AsyncAttrs` y `await objeto.awaitable_attrs.relacion`.

---

## 19. Integración con pandas y frameworks web

```python
import pandas as pd
df = pd.read_sql(select(Producto.categoria, Producto.precio), motor)       # Acepta select() o text()
df.to_sql("resumen", motor, if_exists="replace", index=False)
```

Patrón de sesión por petición en FastAPI:

```python
def obtener_sesion():
    with FabricaSesiones() as sesion:
        yield sesion

@app.get("/productos/{id}")
def leer(id: int, sesion: Session = Depends(obtener_sesion)):
    return sesion.get(Producto, id) or HTTPException(404)
```

---

## 20. Ejemplo práctico completo

**Sistema de pedidos de una tienda** con modelo de dominio completo y capa de servicios:

- Modelos: `Cliente`, `Producto`, `Etiqueta` (muchos a muchos), `Pedido` y `Linea` (con propiedad híbrida `importe`), restricciones `CHECK`/`UNIQUE` y validación con `@validates`.
- Servicio de pedidos **transaccional**: comprueba y descuenta stock; si falta stock de cualquier línea, se revierte todo el pedido.
- Carga de datos masiva, **upsert** de una tarifa de precios y contador de consultas SQL para demostrar el problema **N+1** y su solución con `selectinload`.
- Informes: ventas por cliente (JOIN + GROUP BY + HAVING), ranking por categoría (función de ventana), evolución mensual con CTE y `lag`, productos nunca vendidos (`~any()`), y exportación a pandas.

```python
"""
tienda_sqlalchemy.py — Modelo de pedidos con SQLAlchemy 2: ORM, transacciones, consultas e informes.
Requiere: pip install sqlalchemy pandas
Uso: python tienda_sqlalchemy.py
"""
from __future__ import annotations

import random
from contextlib import contextmanager
from datetime import date, datetime, timedelta
from decimal import Decimal
from pathlib import Path

import pandas as pd
from sqlalchemy import (CheckConstraint, Column, ForeignKey, Numeric, String, Table, create_engine, desc, event,
                        func, insert, select)
from sqlalchemy.dialects.sqlite import insert as sqlite_insert
from sqlalchemy.ext.hybrid import hybrid_property
from sqlalchemy.orm import DeclarativeBase, Mapped, Session, mapped_column, relationship, selectinload, sessionmaker, validates

RUTA_BD = Path("tienda.db")


# ============================================================ Modelos
class Base(DeclarativeBase):
    pass


producto_etiqueta = Table(
    "producto_etiqueta", Base.metadata,
    Column("producto_id", ForeignKey("productos.id", ondelete="CASCADE"), primary_key=True),
    Column("etiqueta_id", ForeignKey("etiquetas.id", ondelete="CASCADE"), primary_key=True),
)


class Cliente(Base):
    __tablename__ = "clientes"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(120), unique=True)
    ciudad: Mapped[str] = mapped_column(String(60))
    alta: Mapped[date] = mapped_column(default=date.today)
    pedidos: Mapped[list[Pedido]] = relationship(back_populates="cliente", cascade="all, delete-orphan")

    @validates("email")
    def _email(self, _, valor: str) -> str:
        if "@" not in valor:
            raise ValueError(f"Correo no válido: {valor}")
        return valor.lower().strip()


class Etiqueta(Base):
    __tablename__ = "etiquetas"
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(30), unique=True)
    productos: Mapped[list[Producto]] = relationship(secondary=producto_etiqueta, back_populates="etiquetas")


class Producto(Base):
    __tablename__ = "productos"
    __table_args__ = (CheckConstraint("precio >= 0", name="ck_precio"), CheckConstraint("stock >= 0", name="ck_stock"))
    id: Mapped[int] = mapped_column(primary_key=True)
    sku: Mapped[str] = mapped_column(String(20), unique=True)
    nombre: Mapped[str] = mapped_column(String(120))
    categoria: Mapped[str] = mapped_column(String(40), index=True)
    precio: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    stock: Mapped[int] = mapped_column(default=0)
    etiquetas: Mapped[list[Etiqueta]] = relationship(secondary=producto_etiqueta, back_populates="productos")
    lineas: Mapped[list[Linea]] = relationship(back_populates="producto")

    def __repr__(self) -> str:
        return f"<{self.sku} {self.nombre}>"


class Pedido(Base):
    __tablename__ = "pedidos"
    id: Mapped[int] = mapped_column(primary_key=True)
    cliente_id: Mapped[int] = mapped_column(ForeignKey("clientes.id", ondelete="CASCADE"), index=True)
    fecha: Mapped[datetime] = mapped_column(default=datetime.now, index=True)
    estado: Mapped[str] = mapped_column(String(20), default="confirmado")
    cliente: Mapped[Cliente] = relationship(back_populates="pedidos")
    lineas: Mapped[list[Linea]] = relationship(back_populates="pedido", cascade="all, delete-orphan")

    @property
    def total(self) -> Decimal:
        return sum((l.importe for l in self.lineas), Decimal("0"))


class Linea(Base):
    __tablename__ = "lineas"
    __table_args__ = (CheckConstraint("cantidad > 0", name="ck_cantidad"),)
    id: Mapped[int] = mapped_column(primary_key=True)
    pedido_id: Mapped[int] = mapped_column(ForeignKey("pedidos.id", ondelete="CASCADE"), index=True)
    producto_id: Mapped[int] = mapped_column(ForeignKey("productos.id"), index=True)
    cantidad: Mapped[int]
    precio_unitario: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    descuento: Mapped[Decimal] = mapped_column(Numeric(4, 2), default=Decimal("0"))
    pedido: Mapped[Pedido] = relationship(back_populates="lineas")
    producto: Mapped[Producto] = relationship(back_populates="lineas")

    @hybrid_property
    def importe(self):
        return self.cantidad * self.precio_unitario * (1 - self.descuento)


def eur(valor) -> str:
    """Formato español: 1.234,56 €."""
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# ============================================================ Infraestructura
class ContadorSQL:
    """Cuenta las sentencias enviadas a la base de datos (para detectar N+1)."""

    def __init__(self, motor):
        self.n = 0
        event.listen(motor, "before_cursor_execute", self._contar)

    def _contar(self, *_):
        self.n += 1

    @contextmanager
    def medir(self, etiqueta: str):
        inicio = self.n
        yield
        print(f"   {etiqueta}: {self.n - inicio} consultas SQL")


def crear_motor(ruta: Path):
    motor = create_engine(f"sqlite:///{ruta}")

    @event.listens_for(motor, "connect")
    def _pragma(conexion_dbapi, _):
        conexion_dbapi.execute("PRAGMA foreign_keys = ON")

    return motor


# ============================================================ Servicios
class StockInsuficiente(Exception):
    pass


def crear_pedido(sesion: Session, cliente_id: int, articulos: dict[str, int],
                 descuento: Decimal = Decimal("0"), fecha: datetime | None = None) -> Pedido:
    """Crea un pedido descontando stock. Debe ejecutarse dentro de una transacción."""
    productos = {p.sku: p for p in sesion.scalars(
        select(Producto).where(Producto.sku.in_(articulos)).with_for_update())}   # Bloqueo de filas (PostgreSQL/MySQL)
    faltan = set(articulos) - set(productos)
    if faltan:
        raise LookupError(f"Productos inexistentes: {', '.join(sorted(faltan))}")
    pedido = Pedido(cliente_id=cliente_id, fecha=fecha or datetime.now())
    for sku, cantidad in articulos.items():
        producto = productos[sku]
        if producto.stock < cantidad:
            raise StockInsuficiente(f"{sku}: solicitadas {cantidad}, disponibles {producto.stock}")
        producto.stock -= cantidad
        pedido.lineas.append(Linea(producto=producto, cantidad=cantidad,
                                   precio_unitario=producto.precio, descuento=descuento))
    sesion.add(pedido)
    sesion.flush()                                         # Asigna el id y valida restricciones ya
    return pedido


def cargar_catalogo(sesion: Session) -> None:
    catalogo = [
        ("ELE-01", "Auriculares inalámbricos", "Electrónica", "59.90", 40, ["audio", "novedad"]),
        ("ELE-02", "Cargador USB-C 65 W", "Electrónica", "34.50", 60, ["accesorio"]),
        ("ELE-03", "Teclado mecánico", "Electrónica", "89.00", 25, ["oficina"]),
        ("OFI-01", "Cuaderno A5", "Oficina", "4.90", 300, ["papelería"]),
        ("OFI-02", "Bolígrafo gel (pack 10)", "Oficina", "7.80", 200, ["papelería"]),
        ("OFI-03", "Lámpara LED de escritorio", "Oficina", "32.00", 35, ["oficina", "novedad"]),
        ("HOG-01", "Taza térmica", "Hogar", "14.90", 120, ["regalo"]),
        ("HOG-02", "Manta de lana", "Hogar", "45.00", 30, ["regalo"]),
        ("HOG-03", "Difusor de aromas", "Hogar", "24.90", 0, ["regalo"]),
    ]
    etiquetas = {n: Etiqueta(nombre=n) for n in {e for *_, es in catalogo for e in es}}
    for sku, nombre, cat, precio, stock, es in catalogo:
        sesion.add(Producto(sku=sku, nombre=nombre, categoria=cat, precio=Decimal(precio), stock=stock,
                            etiquetas=[etiquetas[e] for e in es]))

    ciudades = ["Madrid", "Barcelona", "Valencia", "Sevilla", "Bilbao"]
    nombres = ["Ana Ruiz", "Luis Gómez", "Marta Sanz", "Jorge Pérez", "Elena Vidal", "Pablo Mora",
               "Lucía Navarro", "Diego Ortiz", "Sara Molina", "Raúl Castro", "Nuria Gil", "Iván León"]
    sesion.execute(insert(Cliente), [                          # Inserción masiva (executemany)
        {"nombre": n, "email": f"{n.split()[0].lower()}{i}@correo.es", "ciudad": ciudades[i % 5],
         "alta": date(2025, 1, 1) + timedelta(days=30 * i)} for i, n in enumerate(nombres)])


def simular_pedidos(fabrica: sessionmaker, n: int = 80) -> tuple[int, int]:
    rng = random.Random(7)
    with fabrica() as s:
        clientes = s.scalars(select(Cliente.id)).all()
        skus = s.scalars(select(Producto.sku).where(Producto.categoria != "Hogar")).all() + ["HOG-01", "HOG-02"]
    ok = rechazados = 0
    for i in range(n):
        articulos = {sku: rng.choice([1, 1, 2, 3]) for sku in rng.sample(skus, rng.randint(1, 3))}
        if i % 25 == 0:
            articulos["ELE-03"] = 40                          # Pedido imposible: fuerza un rollback
        fecha = datetime(2026, 1, 1) + timedelta(days=rng.randint(0, 300), hours=rng.randint(8, 21))
        try:
            with fabrica.begin() as s:                        # Una transacción por pedido
                crear_pedido(s, rng.choice(clientes), articulos,
                             descuento=Decimal("0.10") if rng.random() < 0.2 else Decimal("0"), fecha=fecha)
            ok += 1
        except StockInsuficiente:
            rechazados += 1
    return ok, rechazados


# ============================================================ Informes
def informes(fabrica: sessionmaker, contador: ContadorSQL) -> None:
    with fabrica() as s:
        print("\n1. Problema N+1 al recorrer pedidos y sus líneas")
        with contador.medir("Carga perezosa (lazy)"):
            total_lazy = sum(p.total for p in s.scalars(select(Pedido)).all())
        s.expunge_all()
        with contador.medir("Con selectinload"):
            pedidos = s.scalars(select(Pedido).options(selectinload(Pedido.lineas))).all()
            total_eager = sum(p.total for p in pedidos)
        assert total_lazy == total_eager
        print(f"   Facturación total: {eur(total_eager)}")

        print("\n2. Mejores clientes (JOIN + GROUP BY + HAVING)")
        importe = func.sum(Linea.importe)
        stmt = (select(Cliente.nombre, Cliente.ciudad, func.count(func.distinct(Pedido.id)).label("pedidos"),
                       func.round(importe, 2).label("total"))
                .join(Cliente.pedidos).join(Pedido.lineas)
                .group_by(Cliente.id).having(func.count(func.distinct(Pedido.id)) >= 5)
                .order_by(desc("total")).limit(5))
        for f in s.execute(stmt):
            print(f"   {f.nombre:<14} {f.ciudad:<10} {f.pedidos:>2} pedidos  {eur(f.total):>12}")

        print("\n3. Ranking de productos por categoría (función de ventana)")
        unidades = func.sum(Linea.cantidad)
        stmt = (select(Producto.categoria, Producto.nombre, unidades.label("uds"),
                       func.rank().over(partition_by=Producto.categoria, order_by=unidades.desc()).label("puesto"))
                .join(Producto.lineas).group_by(Producto.id).order_by(Producto.categoria, "puesto"))
        for f in s.execute(stmt):
            if f.puesto == 1:
                print(f"   {f.categoria:<12} n.º 1: {f.nombre} ({f.uds} uds)")

        print("\n4. Evolución mensual (CTE + lag)")
        mes = func.strftime("%Y-%m", Pedido.fecha)
        mensual = (select(mes.label("mes"), func.round(func.sum(Linea.importe), 2).label("ventas"))
                   .join(Pedido.lineas).group_by(mes).cte("mensual"))
        anterior = func.lag(mensual.c.ventas).over(order_by=mensual.c.mes)
        stmt = select(mensual.c.mes, mensual.c.ventas,
                      func.round((mensual.c.ventas - anterior) * 100.0 / anterior, 1).label("var"))
        for f in s.execute(stmt):
            variacion = "" if f.var is None else f"{f.var:+.1f} %".replace(".", ",")
            print(f"   {f.mes}  {eur(f.ventas):>12}  {variacion}")

        print("\n5. Productos nunca vendidos y etiquetas (EXISTS + muchos a muchos)")
        for p in s.scalars(select(Producto).where(~Producto.lineas.any())
                           .options(selectinload(Producto.etiquetas))):
            print(f"   {p.sku} {p.nombre} · stock {p.stock} · etiquetas: {', '.join(e.nombre for e in p.etiquetas)}")

        print("\n6. Productos bajo umbral de reposición")
        for sku, nombre, stock in s.execute(select(Producto.sku, Producto.nombre, Producto.stock)
                                            .where(Producto.stock < 20).order_by(Producto.stock)):
            print(f"   {sku} {nombre:<28} quedan {stock}")

    print("\n7. Exportación a pandas")
    df = pd.read_sql(select(Producto.categoria, Linea.cantidad, Linea.importe.label("importe"))
                     .join(Linea.producto), crear_motor(RUTA_BD))
    resumen = df.groupby("categoria").agg(unidades=("cantidad", "sum"), ventas=("importe", "sum")).round(2)
    print(resumen.to_string())


def actualizar_tarifa(fabrica: sessionmaker) -> None:
    print("\n8. Upsert de tarifa (actualiza existentes e inserta nuevos)")
    tarifa = [{"sku": "OFI-01", "nombre": "Cuaderno A5", "categoria": "Oficina", "precio": Decimal("5.20"), "stock": 280},
              {"sku": "HOG-04", "nombre": "Cojín decorativo", "categoria": "Hogar", "precio": Decimal("19.90"), "stock": 50}]
    stmt = sqlite_insert(Producto).values(tarifa)
    stmt = stmt.on_conflict_do_update(index_elements=[Producto.sku], set_={"precio": stmt.excluded.precio})
    with fabrica.begin() as s:
        s.execute(stmt)
        for p in s.scalars(select(Producto).where(Producto.sku.in_(["OFI-01", "HOG-04"]))):
            print(f"   {p.sku} {p.nombre}: {eur(p.precio)} (stock {p.stock})")


def main() -> int:
    RUTA_BD.unlink(missing_ok=True)
    motor = crear_motor(RUTA_BD)
    Base.metadata.create_all(motor)
    contador = ContadorSQL(motor)
    fabrica = sessionmaker(motor, expire_on_commit=False)

    with fabrica.begin() as s:
        cargar_catalogo(s)
    print("Catálogo cargado:", end=" ")
    with fabrica() as s:
        print(f"{s.scalar(select(func.count()).select_from(Producto))} productos, "
              f"{s.scalar(select(func.count()).select_from(Cliente))} clientes")

    ok, rechazados = simular_pedidos(fabrica)
    print(f"Pedidos confirmados: {ok} · rechazados por stock (revertidos): {rechazados}")
    with fabrica() as s:
        stock = s.scalar(select(Producto.stock).where(Producto.sku == "ELE-03"))
        vendidas = s.scalar(select(func.sum(Linea.cantidad)).join(Linea.producto).where(Producto.sku == "ELE-03"))
        print(f"Teclado (ELE-03): vendidas {vendidas} + stock {stock} = {vendidas + stock} "
              f"(stock inicial 25: los pedidos rechazados no descontaron nada)")
        try:
            s.add(Cliente(nombre="X", email="sin-arroba", ciudad="Madrid"))
        except ValueError as e:
            print(f"Validación: {e}")

    informes(fabrica, contador)
    actualizar_tarifa(fabrica)
    motor.dispose()
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `DetachedInstanceError: … is not bound to a Session` | Se accede a una relación perezosa tras cerrar la sesión. | Cargue con `selectinload`/`joinedload` antes de cerrar o mantenga la sesión abierta. |
| Los atributos se recargan (o fallan) tras `commit` | `expire_on_commit=True` (defecto). | `sessionmaker(..., expire_on_commit=False)` cuando lo necesite. |
| `IntegrityError: UNIQUE constraint failed` | Valor duplicado. | Compruebe antes, use *upsert* o capture la excepción y haga `rollback`. |
| `PendingRollbackError` | Se sigue usando la sesión tras un error. | `sesion.rollback()` o use `with sesion.begin():`. |
| `MultipleResultsFound` / `NoResultFound` | `.one()` con 0 o más de 1 resultado. | `.first()`, `.one_or_none()` o corrija el filtro. |
| Rendimiento pobre con muchas consultas pequeñas | Problema N+1. | `selectinload`, `joinedload`; detecte con `echo=True` o `raiseload`. |
| `ArgumentError: … expected a Mapped[] annotation` | Anotación sin `Mapped[...]`. | `campo: Mapped[tipo] = mapped_column(...)`. |
| `NoForeignKeysError` al definir una relación | Falta `ForeignKey` o es ambigua. | Añádala o indique `foreign_keys=[...]`. |
| Las claves foráneas no se aplican en SQLite | SQLite las desactiva por defecto. | Evento `connect` con `PRAGMA foreign_keys = ON`. |
| `create_all` no añade una columna nueva | Solo crea tablas inexistentes. | Migración con Alembic. |
| `MissingGreenlet` (asíncrono) | Carga perezosa implícita en `AsyncSession`. | Carga anticipada o `awaitable_attrs`. |
| Importes con decimales extraños | Uso de `float`. | `Numeric` + `Decimal`. |
| `ObjectNotExecutableError` con una cadena SQL | `execute("SELECT …")` sin `text()`. | `execute(text("SELECT …"))`. |

---

## 22. Buenas prácticas

- Un `Engine` por base de datos y proceso; sesiones cortas (por petición o por unidad de trabajo).
- Use bloques `with fabrica.begin()`: *commit* automático y *rollback* ante errores.
- Defina restricciones en la base de datos (`UNIQUE`, `CHECK`, `FOREIGN KEY`) además de validar en Python.
- Controle la carga de relaciones (`selectinload`, `raiseload`) y revise el SQL con `echo=True` durante el desarrollo.
- Use `Decimal`/`Numeric` para dinero y `DateTime(timezone=True)` cuando trabaje con zonas horarias.
- Mantenga la lógica de negocio en funciones de servicio que reciben la sesión: facilita las pruebas (SQLite en memoria).
- Gestione el esquema con Alembic y no con `create_all` en producción.
- Nunca interpole datos en SQL; use siempre parámetros.

---

## 23. Referencia rápida

```python
motor = create_engine(url, echo=False, pool_pre_ping=True) · Base.metadata.create_all(motor)
class M(Base): __tablename__ = "m"; id: Mapped[int] = mapped_column(primary_key=True); x: Mapped[str | None]
relationship(back_populates=, cascade="all, delete-orphan", secondary=) · ForeignKey("tabla.id", ondelete="CASCADE")
Fabrica = sessionmaker(motor, expire_on_commit=False) · with Fabrica.begin() as s: s.add(obj); s.flush()
s.get(M, id) · s.scalars(select(M).where(...)).all()/first()/one() · s.scalar(select(func.count())...) · s.execute(stmt).all()
select(M).where(c1, c2).order_by(M.x.desc()).limit(n).offset(k) · .join(M.rel) · .outerjoin · .group_by · .having
func.count/sum/avg/max · func.rank().over(partition_by=, order_by=) · .cte() · .subquery() · .scalar_subquery() · M.rel.any()
update(M).where(...).values(...) · delete(M).where(...) · s.delete(obj) · insert(M) + lista de dicts · dialects.sqlite.insert(...).on_conflict_do_update
options(selectinload(M.rel), joinedload(M.rel), raiseload("*")) · text("... :p") + {"p": v} · @validates · @hybrid_property
```

---

## 24. Ejercicios propuestos

1. Añada un modelo `Devolucion` relacionado con `Linea` y un servicio que repone stock y genera un abono, todo en una transacción.
2. Escriba un informe de retención: clientes con pedidos en el primer trimestre que vuelven a comprar en el segundo (subconsultas o CTE).
3. Implemente paginación por cursor (`WHERE id > :ultimo ORDER BY id LIMIT n`) y compárela con `OFFSET` en una tabla de 100 000 filas.
4. Configure Alembic sobre el ejemplo y cree una migración que añada la columna `coste` a `Producto`.
5. Convierta el servicio `crear_pedido` a versión asíncrona con `AsyncSession` y `aiosqlite`.

---

## 25. Referencias

- Documentación oficial: https://docs.sqlalchemy.org/en/20/
- Tutorial unificado (Core + ORM): https://docs.sqlalchemy.org/en/20/tutorial/index.html
- Guía de consultas ORM: https://docs.sqlalchemy.org/en/20/orm/queryguide/index.html
- Técnicas de carga de relaciones: https://docs.sqlalchemy.org/en/20/orm/queryguide/relationships.html
- Alembic: https://alembic.sqlalchemy.org/
