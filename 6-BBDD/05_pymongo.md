# Manual de uso: PyMongo

> **Categoría:** Bases de datos · **Utilidad:** Conectar con MongoDB
> **Tipo:** Librería externa (controlador oficial de MongoDB) · **Versión de referencia:** PyMongo 4.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Conectar](#4-conectar)
5. [Insertar documentos](#5-insertar-documentos)
6. [Consultar documentos](#6-consultar-documentos)
7. [Operadores de consulta](#7-operadores-de-consulta)
8. [Actualizar documentos](#8-actualizar-documentos)
9. [Eliminar documentos](#9-eliminar-documentos)
10. [Índices](#10-índices)
11. [Agregaciones](#11-agregaciones)
12. [Modelado de datos](#12-modelado-de-datos)
13. [Operaciones masivas y transacciones](#13-operaciones-masivas-y-transacciones)
14. [Funcionalidades avanzadas](#14-funcionalidades-avanzadas)
15. [Ejemplo práctico completo](#15-ejemplo-práctico-completo)
16. [Errores frecuentes](#16-errores-frecuentes)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Referencia rápida](#18-referencia-rápida)
19. [Ejercicios propuestos](#19-ejercicios-propuestos)
20. [Referencias](#20-referencias)

---

## 1. Descripción

**MongoDB** es una base de datos **NoSQL orientada a documentos**: en lugar de tablas y filas, almacena **documentos** con formato similar a JSON (BSON) dentro de **colecciones**. Cada documento puede tener una estructura distinta y contener listas y subdocumentos.

**PyMongo** es el controlador oficial para trabajar con MongoDB desde Python.

**Casos de uso habituales:**
- Catálogos de productos con atributos variables.
- Perfiles de usuario, configuraciones y preferencias.
- Registros de eventos, logs y datos de sensores.
- Contenidos (artículos, comentarios) y aplicaciones web/móviles.
- Prototipos donde el esquema cambia con frecuencia.

**Comparación con bases de datos relacionales:**

| SQL | MongoDB |
|---|---|
| Base de datos | Base de datos |
| Tabla | Colección |
| Fila | Documento |
| Columna | Campo |
| Clave primaria | `_id` |
| JOIN | Documentos embebidos o `$lookup` |
| `GROUP BY` | Canal de agregación (`aggregate`) |
| Esquema fijo | Esquema flexible (validación opcional) |

---

## 2. Instalación e importación

```bash
pip install pymongo
pip install "pymongo[srv]"        # Necesario para cadenas mongodb+srv:// (MongoDB Atlas) en versiones antiguas
```

```python
from pymongo import MongoClient, ASCENDING, DESCENDING, ReturnDocument
from pymongo.errors import DuplicateKeyError, ConnectionFailure, OperationFailure
from bson import ObjectId
```

Para practicar:

```bash
docker run --name mongo -p 27017:27017 -d mongo:7
```

O cree un clúster gratuito en **MongoDB Atlas**. Herramientas visuales: **MongoDB Compass**, extensión de MongoDB para VS Code.

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `MongoClient` | Cliente de conexión (gestiona un pool). Uno por aplicación. |
| Base de datos | Contenedor de colecciones (`cliente["tienda"]`). |
| Colección | Conjunto de documentos (`db["productos"]`). |
| Documento | Diccionario de Python almacenado como BSON. |
| `_id` | Identificador único de cada documento (`ObjectId` por defecto). |
| Filtro | Diccionario que describe qué documentos seleccionar. |
| Proyección | Qué campos devolver. |
| Cursor | Resultado iterable de `find()`. |
| Canal de agregación | Lista de etapas (`$match`, `$group`…) que transforman documentos. |

> Las bases de datos y colecciones se crean automáticamente al insertar el primer documento.

---

## 4. Conectar

### 4.1 Conexión local

```python
cliente = MongoClient("mongodb://localhost:27017/")
db = cliente["tienda"]                # o cliente.tienda
productos = db["productos"]           # o db.productos
```

### 4.2 Con autenticación y opciones

```python
import os

cliente = MongoClient(
    os.environ["MONGO_URI"],          # mongodb://usuario:clave@host:27017/?authSource=admin
    serverSelectionTimeoutMS=5000,    # Tiempo máximo para encontrar el servidor
    appname="gestor_pedidos",
    tz_aware=True,                    # Fechas con zona horaria
)
```

### 4.3 MongoDB Atlas

```python
cliente = MongoClient("mongodb+srv://usuario:clave@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority")
```

> Si la contraseña contiene caracteres especiales (`@`, `:`, `/`), codifíquela con `urllib.parse.quote_plus(clave)`.

### 4.4 Comprobar la conexión

```python
try:
    cliente.admin.command("ping")
    print("Conectado a MongoDB", cliente.server_info()["version"])
except ConnectionFailure as e:
    print("No se pudo conectar:", e)
```

### 4.5 Explorar

```python
cliente.list_database_names()
db.list_collection_names()
```

### 4.6 Cerrar

```python
cliente.close()

with MongoClient(uri) as cliente:          # Se cierra al salir
    ...
```

---

## 5. Insertar documentos

### 5.1 Un documento

```python
from datetime import datetime, timezone

producto = {
    "codigo": "TEC-01",
    "nombre": "Teclado mecánico",
    "categoria": "Periféricos",
    "precio": 59.90,
    "stock": 120,
    "etiquetas": ["oferta", "gaming"],
    "especificaciones": {"switches": "rojos", "inalambrico": False, "idioma": "ES"},
    "creado": datetime.now(timezone.utc),
}
resultado = productos.insert_one(producto)
print(resultado.inserted_id)          # ObjectId('6743...')
```

> `insert_one` **añade** la clave `_id` al diccionario original.

### 5.2 Varios documentos

```python
resultado = productos.insert_many([
    {"codigo": "RAT-01", "nombre": "Ratón inalámbrico", "categoria": "Periféricos", "precio": 24.50, "stock": 200},
    {"codigo": "MON-27", "nombre": "Monitor 27\"", "categoria": "Pantallas", "precio": 219.0, "stock": 35,
     "especificaciones": {"pulgadas": 27, "hz": 144}},
])
print(len(resultado.inserted_ids))
```

`ordered=False` continúa insertando aunque algún documento falle (por ejemplo, por duplicado):

```python
productos.insert_many(documentos, ordered=False)
```

### 5.3 `_id` propio

```python
db.paises.insert_one({"_id": "ES", "nombre": "España", "moneda": "EUR"})
```

### 5.4 Tipos de datos BSON

| Python | BSON |
|---|---|
| `dict` | Documento |
| `list` | Array |
| `str` | String |
| `int` | Int32 / Int64 |
| `float` | Double |
| `bool` | Boolean |
| `None` | Null |
| `datetime.datetime` | Date (UTC, precisión de milisegundos) |
| `bson.ObjectId` | ObjectId |
| `bson.Decimal128` | Decimal128 (importes exactos) |
| `bytes` | Binary |

> `datetime.date` (sin hora) **no** es compatible: conviértalo a `datetime`. Para importes exactos use `Decimal128`.

```python
from decimal import Decimal
from bson.decimal128 import Decimal128

db.facturas.insert_one({"importe": Decimal128(Decimal("199.95"))})
importe = db.facturas.find_one()["importe"].to_decimal()
```

---

## 6. Consultar documentos

### 6.1 Un documento

```python
doc = productos.find_one({"codigo": "TEC-01"})
doc = productos.find_one({"_id": ObjectId("6743a1b2c3d4e5f6a7b8c9d0")})
```

> El `_id` es un `ObjectId`, no una cadena: convierta con `ObjectId(texto)` al buscar por id recibido de una URL o formulario.

### 6.2 Varios documentos

```python
for p in productos.find({"categoria": "Periféricos"}):
    print(p["nombre"], p["precio"])

lista = list(productos.find({"stock": {"$gt": 0}}))
```

### 6.3 Proyección (campos a devolver)

```python
productos.find({}, {"nombre": 1, "precio": 1, "_id": 0})       # Solo nombre y precio
productos.find({}, {"especificaciones": 0})                    # Todo excepto especificaciones
```

### 6.4 Ordenar, limitar y paginar

```python
productos.find().sort("precio", DESCENDING).limit(10)
productos.find().sort([("categoria", ASCENDING), ("precio", DESCENDING)])

pagina, tamano = 3, 20
productos.find().sort("_id", ASCENDING).skip((pagina - 1) * tamano).limit(tamano)
```

> Para paginar colecciones muy grandes, es más eficiente filtrar por el último valor visto (`{"_id": {"$gt": ultimo_id}}`) que usar `skip`.

### 6.5 Contar y valores distintos

```python
productos.count_documents({"categoria": "Periféricos"})
productos.estimated_document_count()                         # Rápido, aproximado, sin filtro
productos.distinct("categoria")
productos.distinct("etiquetas", {"stock": {"$gt": 0}})
```

### 6.6 Campos anidados y listas

```python
productos.find({"especificaciones.inalambrico": True})       # Notación de punto
productos.find({"etiquetas": "oferta"})                      # La lista contiene "oferta"
productos.find({"especificaciones.pulgadas": {"$gte": 27}})
```

### 6.7 Convertir resultados a JSON o DataFrame

```python
from bson import json_util
import pandas as pd

texto = json_util.dumps(list(productos.find()), ensure_ascii=False, indent=2)    # Maneja ObjectId y fechas
df = pd.DataFrame(list(productos.find({}, {"_id": 0})))
df = pd.json_normalize(list(productos.find({}, {"_id": 0})))                      # Aplana subdocumentos
```

---

## 7. Operadores de consulta

### 7.1 Comparación

| Operador | Significado | Ejemplo |
|---|---|---|
| `$eq` | Igual | `{"precio": {"$eq": 10}}` (equivale a `{"precio": 10}`) |
| `$ne` | Distinto | `{"categoria": {"$ne": "Audio"}}` |
| `$gt` / `$gte` | Mayor / mayor o igual | `{"stock": {"$gt": 0}}` |
| `$lt` / `$lte` | Menor / menor o igual | `{"precio": {"$lte": 50}}` |
| `$in` | En la lista | `{"categoria": {"$in": ["Audio", "Pantallas"]}}` |
| `$nin` | No en la lista | `{"codigo": {"$nin": ["A", "B"]}}` |

```python
productos.find({"precio": {"$gte": 20, "$lte": 100}})        # Rango
```

### 7.2 Lógicos

```python
productos.find({"categoria": "Periféricos", "stock": {"$gt": 0}})               # AND implícito
productos.find({"$or": [{"stock": 0}, {"precio": {"$gt": 500}}]})
productos.find({"$and": [{"precio": {"$gt": 10}}, {"precio": {"$lt": 100}}]})
productos.find({"precio": {"$not": {"$gt": 100}}})
productos.find({"$nor": [{"categoria": "Audio"}, {"stock": 0}]})
```

### 7.3 Existencia y tipo

```python
productos.find({"especificaciones": {"$exists": True}})
productos.find({"descuento": {"$exists": False}})
productos.find({"precio": {"$type": "string"}})              # Detectar datos mal cargados
```

### 7.4 Texto con expresiones regulares

```python
productos.find({"nombre": {"$regex": "^tecl", "$options": "i"}})     # Empieza por "tecl", sin distinguir mayúsculas

import re
productos.find({"nombre": re.compile("inalámbrico", re.IGNORECASE)})
```

> Escape el texto del usuario con `re.escape(texto)`. Las expresiones que no empiezan por `^` no aprovechan índices.

### 7.5 Arrays

```python
productos.find({"etiquetas": {"$all": ["oferta", "gaming"]}})        # Contiene todas
productos.find({"etiquetas": {"$size": 2}})                          # Exactamente 2 elementos
productos.find({"valoraciones": {"$elemMatch": {"puntos": {"$gte": 4}, "verificada": True}}})
```

---

## 8. Actualizar documentos

### 8.1 Actualizar uno o varios

```python
productos.update_one({"codigo": "TEC-01"}, {"$set": {"precio": 54.90, "etiquetas": ["oferta"]}})

r = productos.update_many({"categoria": "Periféricos"}, {"$mul": {"precio": 1.05}})
print(r.matched_count, r.modified_count)
```

> Sin un operador (`$set`, `$inc`…), `update_one` produce un error. Para sustituir el documento completo use `replace_one`.

### 8.2 Operadores de actualización

| Operador | Efecto | Ejemplo |
|---|---|---|
| `$set` | Asigna valores | `{"$set": {"stock": 50}}` |
| `$unset` | Elimina campos | `{"$unset": {"descuento": ""}}` |
| `$inc` | Suma (o resta) | `{"$inc": {"stock": -2, "vendidos": 2}}` |
| `$mul` | Multiplica | `{"$mul": {"precio": 1.1}}` |
| `$min` / `$max` | Asigna si el valor es menor / mayor | `{"$max": {"precio_max": 60}}` |
| `$rename` | Renombra un campo | `{"$rename": {"nom": "nombre"}}` |
| `$currentDate` | Fecha actual | `{"$currentDate": {"modificado": True}}` |
| `$push` | Añade a una lista | `{"$push": {"etiquetas": "nuevo"}}` |
| `$addToSet` | Añade si no existe | `{"$addToSet": {"etiquetas": "oferta"}}` |
| `$pull` | Quita elementos | `{"$pull": {"etiquetas": "oferta"}}` |
| `$pop` | Quita el primero (-1) o el último (1) | `{"$pop": {"historial": -1}}` |

```python
# Añadir varios elementos y mantener solo los 10 últimos
productos.update_one(
    {"codigo": "TEC-01"},
    {"$push": {"historial_precios": {"$each": [{"precio": 54.9, "fecha": datetime.now(timezone.utc)}],
                                     "$slice": -10}}},
)

# Actualizar un elemento concreto de una lista ($)
pedidos.update_one({"_id": pid, "lineas.codigo": "RAT-01"}, {"$set": {"lineas.$.cantidad": 3}})
```

### 8.3 Insertar si no existe (*upsert*)

```python
productos.update_one(
    {"codigo": "AUR-BT"},
    {"$set": {"nombre": "Auriculares BT", "precio": 79.0},
     "$setOnInsert": {"creado": datetime.now(timezone.utc), "stock": 0}},
    upsert=True,
)
```

`$setOnInsert` solo se aplica cuando se crea un documento nuevo.

### 8.4 Buscar y modificar en una sola operación atómica

```python
doc = productos.find_one_and_update(
    {"codigo": "TEC-01", "stock": {"$gte": 2}},          # Solo si hay stock suficiente
    {"$inc": {"stock": -2}},
    return_document=ReturnDocument.AFTER,                # Devuelve el documento ya modificado
)
if doc is None:
    print("Stock insuficiente")
```

> Este patrón evita condiciones de carrera: la comprobación y la modificación ocurren de forma atómica.

### 8.5 Contadores secuenciales

```python
def siguiente_numero(db, nombre: str) -> int:
    doc = db.contadores.find_one_and_update(
        {"_id": nombre}, {"$inc": {"valor": 1}}, upsert=True, return_document=ReturnDocument.AFTER
    )
    return doc["valor"]

numero_factura = siguiente_numero(db, "facturas_2026")
```

### 8.6 Actualizaciones con canal de agregación

```python
productos.update_many({}, [{"$set": {"precio_iva": {"$round": [{"$multiply": ["$precio", 1.21]}, 2]}}}])
```

---

## 9. Eliminar documentos

```python
productos.delete_one({"codigo": "TMP-01"})
r = productos.delete_many({"stock": 0, "descatalogado": True})
print(r.deleted_count)

productos.find_one_and_delete({"codigo": "X"})        # Elimina y devuelve el documento

productos.drop()                                      # Elimina la colección completa
cliente.drop_database("pruebas")
```

> `delete_many({})` elimina **todos** los documentos de la colección.

---

## 10. Índices

Sin índices, MongoDB recorre toda la colección en cada consulta.

```python
productos.create_index("codigo", unique=True)
productos.create_index([("categoria", ASCENDING), ("precio", DESCENDING)])     # Compuesto
productos.create_index("especificaciones.inalambrico")                       # Campo anidado
productos.create_index("etiquetas")                                          # Multiclave (listas)

# Texto completo (uno por colección)
productos.create_index([("nombre", "text"), ("descripcion", "text")], default_language="spanish")
productos.find({"$text": {"$search": "teclado inalámbrico"}},
               {"score": {"$meta": "textScore"}}).sort([("score", {"$meta": "textScore"})])

# TTL: eliminar automáticamente documentos antiguos
db.sesiones.create_index("creado", expireAfterSeconds=3600)          # Se borran a la hora

# Índice parcial
productos.create_index("codigo_promocion", unique=True,
                       partialFilterExpression={"codigo_promocion": {"$exists": True}})

productos.index_information()
productos.drop_index("categoria_1_precio_-1")
```

Analizar si una consulta usa índices:

```python
plan = productos.find({"categoria": "Audio"}).explain()
print(plan["queryPlanner"]["winningPlan"])          # IXSCAN = usa índice; COLLSCAN = recorre todo
```

---

## 11. Agregaciones

El canal de agregación procesa los documentos en **etapas** sucesivas (equivalente a `WHERE`, `GROUP BY`, `JOIN`, `ORDER BY` en SQL).

### 11.1 Etapas principales

| Etapa | Función | Equivalente SQL |
|---|---|---|
| `$match` | Filtra documentos | `WHERE` |
| `$group` | Agrupa y calcula | `GROUP BY` |
| `$project` / `$set` | Selecciona o calcula campos | `SELECT` |
| `$sort` | Ordena | `ORDER BY` |
| `$limit` / `$skip` | Limita / salta | `LIMIT` / `OFFSET` |
| `$unwind` | Descompone una lista en documentos | — |
| `$lookup` | Une con otra colección | `JOIN` |
| `$count` | Cuenta | `COUNT(*)` |
| `$facet` | Varias agregaciones en paralelo | — |
| `$bucket` | Agrupa por rangos | — |
| `$out` / `$merge` | Guarda el resultado en una colección | `INSERT INTO ... SELECT` |

### 11.2 Ejemplo: ventas por categoría

```python
canal = [
    {"$match": {"fecha": {"$gte": datetime(2026, 1, 1, tzinfo=timezone.utc)}, "estado": {"$ne": "cancelado"}}},
    {"$unwind": "$lineas"},
    {"$group": {
        "_id": "$lineas.categoria",
        "unidades": {"$sum": "$lineas.cantidad"},
        "ventas": {"$sum": {"$multiply": ["$lineas.cantidad", "$lineas.precio"]}},
        "pedidos": {"$addToSet": "$_id"},
    }},
    {"$project": {"_id": 0, "categoria": "$_id", "unidades": 1,
                  "ventas": {"$round": ["$ventas", 2]}, "num_pedidos": {"$size": "$pedidos"}}},
    {"$sort": {"ventas": -1}},
]
for fila in db.pedidos.aggregate(canal):
    print(fila)
```

### 11.3 Acumuladores de `$group`

`$sum`, `$avg`, `$min`, `$max`, `$first`, `$last`, `$push` (lista), `$addToSet` (lista sin duplicados), `$count`.

### 11.4 Agrupar por fecha

```python
{"$group": {
    "_id": {"$dateToString": {"format": "%Y-%m", "date": "$fecha", "timezone": "Europe/Madrid"}},
    "ventas": {"$sum": "$total"},
}}
```

### 11.5 Unir colecciones (`$lookup`)

```python
canal = [
    {"$lookup": {"from": "clientes", "localField": "cliente_id", "foreignField": "_id", "as": "cliente"}},
    {"$unwind": "$cliente"},
    {"$project": {"total": 1, "cliente.nombre": 1, "cliente.ciudad": 1}},
]
```

### 11.6 Varios resultados a la vez (`$facet`)

```python
canal = [
    {"$facet": {
        "por_categoria": [{"$group": {"_id": "$categoria", "n": {"$sum": 1}}}],
        "rango_precios": [{"$bucket": {"groupBy": "$precio", "boundaries": [0, 25, 100, 500, 10_000],
                                       "default": "otros", "output": {"n": {"$sum": 1}}}}],
        "sin_stock": [{"$match": {"stock": 0}}, {"$count": "total"}],
    }}
]
resultado = next(productos.aggregate(canal))
```

---

## 12. Modelado de datos

### 12.1 Embeber frente a referenciar

| Embeber (subdocumentos) | Referenciar (ids) |
|---|---|
| Los datos se leen siempre juntos. | Los datos se consultan por separado. |
| Relación 1:1 o 1:pocos. | Relación 1:muchos o N:M con listas grandes. |
| Los datos embebidos no cambian a menudo. | Los datos se actualizan con frecuencia de forma independiente. |
| Ejemplo: líneas de un pedido, dirección. | Ejemplo: clientes y sus miles de pedidos. |

```python
# Embebido: el pedido incluye sus líneas y una copia de los datos del cliente en ese momento
pedido = {
    "numero": 1045,
    "cliente": {"id": ObjectId("..."), "nombre": "Ana López", "ciudad": "Madrid"},
    "lineas": [
        {"codigo": "TEC-01", "nombre": "Teclado", "cantidad": 1, "precio": 59.90},
        {"codigo": "RAT-01", "nombre": "Ratón", "cantidad": 2, "precio": 24.50},
    ],
    "total": 108.90,
}
```

> Límite: un documento no puede superar **16 MB**. Evite listas que crezcan sin límite dentro de un documento.

### 12.2 Validación de esquema

MongoDB permite exigir una estructura mínima:

```python
db.create_collection("clientes", validator={
    "$jsonSchema": {
        "bsonType": "object",
        "required": ["nombre", "email", "ciudad"],
        "properties": {
            "nombre": {"bsonType": "string", "minLength": 2},
            "email": {"bsonType": "string", "pattern": r"^[^@\s]+@[^@\s]+\.[^@\s]+$"},
            "ciudad": {"bsonType": "string"},
            "saldo": {"bsonType": ["double", "int", "decimal"], "minimum": 0},
        },
    }
})
```

> En Python, también puede validar los datos antes de insertarlos con **Pydantic**.

---

## 13. Operaciones masivas y transacciones

### 13.1 Operaciones masivas (`bulk_write`)

```python
from pymongo import InsertOne, UpdateOne, DeleteOne, ReplaceOne

operaciones = [
    UpdateOne({"codigo": c["codigo"]}, {"$set": c}, upsert=True)
    for c in catalogo_actualizado
] + [DeleteOne({"codigo": "OBSOLETO-1"})]

r = productos.bulk_write(operaciones, ordered=False)
print(r.upserted_count, r.modified_count, r.deleted_count)
```

### 13.2 Transacciones multidocumento

Requieren un **replica set** o un clúster (MongoDB Atlas lo es; una instancia local independiente no).

```python
with cliente.start_session() as sesion:
    with sesion.start_transaction():
        db.cuentas.update_one({"_id": 1}, {"$inc": {"saldo": -100}}, session=sesion)
        db.cuentas.update_one({"_id": 2}, {"$inc": {"saldo": 100}}, session=sesion)
        # Se confirma al salir del bloque; se deshace si hay una excepción
```

Con reintentos automáticos ante errores transitorios:

```python
def transferir(sesion):
    db.cuentas.update_one({"_id": 1}, {"$inc": {"saldo": -100}}, session=sesion)
    db.cuentas.update_one({"_id": 2}, {"$inc": {"saldo": 100}}, session=sesion)

with cliente.start_session() as sesion:
    sesion.with_transaction(transferir)
```

> Las operaciones sobre **un solo documento** siempre son atómicas: un buen modelado (embeber lo que cambia junto) reduce la necesidad de transacciones.

---

## 14. Funcionalidades avanzadas

### 14.1 Flujos de cambios (*change streams*)

Reciba en tiempo real los cambios de una colección (requiere replica set):

```python
with db.pedidos.watch([{"$match": {"operationType": "insert"}}]) as flujo:
    for cambio in flujo:
        print("Nuevo pedido:", cambio["fullDocument"]["numero"])
```

### 14.2 Consultas geoespaciales

```python
db.tiendas.create_index([("ubicacion", "2dsphere")])
db.tiendas.insert_one({"nombre": "Tienda Centro", "ubicacion": {"type": "Point", "coordinates": [-3.7038, 40.4168]}})

cerca = db.tiendas.find({"ubicacion": {"$near": {
    "$geometry": {"type": "Point", "coordinates": [-3.69, 40.42]},
    "$maxDistance": 2000,                                     # metros
}}})
```

> Las coordenadas GeoJSON van en orden **[longitud, latitud]**.

### 14.3 Archivos grandes (GridFS)

```python
import gridfs

fs = gridfs.GridFS(db)
archivo_id = fs.put(open("informe.pdf", "rb"), filename="informe.pdf", metadata={"cliente": "ana"})
contenido = fs.get(archivo_id).read()
```

### 14.4 Asíncrono

PyMongo 4.9+ incluye una API asíncrona (`AsyncMongoClient`); también existe la librería **Motor**:

```python
from pymongo import AsyncMongoClient

async def listar():
    cliente = AsyncMongoClient("mongodb://localhost:27017")
    async for p in cliente.tienda.productos.find({"stock": {"$gt": 0}}):
        print(p["nombre"])
```

### 14.5 Pruebas sin servidor

Para pruebas unitarias puede usar **mongomock**, que imita la API de PyMongo en memoria (con soporte parcial de agregaciones):

```python
import mongomock
cliente = mongomock.MongoClient()
```

> mongomock no cubre todo: por ejemplo, `bulk_write` con versiones recientes de PyMongo puede fallar (`unexpected keyword argument 'sort'`), y las transacciones y *change streams* no están disponibles. Para pruebas de integración, use un MongoDB real en Docker.

---

## 15. Ejemplo práctico completo

Catálogo y pedidos de una tienda online: índices, validación con un modelo propio, carga con `bulk_write`, búsqueda con filtros dinámicos, pedidos con descuento atómico de stock y compensación si falla, informes con agregaciones y exportación.

```python
"""
Tienda online con PyMongo.
Requisitos: pip install pymongo
Variable de entorno: MONGO_URI (por defecto mongodb://localhost:27017)
"""
import os
import random
import re
from datetime import datetime, timedelta, timezone

from bson import json_util
from pymongo import ASCENDING, DESCENDING, MongoClient, ReturnDocument, UpdateOne
from pymongo.errors import ConnectionFailure, DuplicateKeyError

URI = os.getenv("MONGO_URI", "mongodb://localhost:27017")
UTC = timezone.utc

CATALOGO = [
    {"codigo": "TEC-01", "nombre": "Teclado mecánico", "categoria": "Periféricos", "precio": 59.90,
     "stock": 80, "etiquetas": ["gaming"], "atributos": {"inalambrico": False, "color": "negro"}},
    {"codigo": "RAT-01", "nombre": "Ratón inalámbrico", "categoria": "Periféricos", "precio": 24.50,
     "stock": 150, "etiquetas": ["oferta"], "atributos": {"inalambrico": True, "dpi": 1600}},
    {"codigo": "MON-27", "nombre": "Monitor 27 pulgadas", "categoria": "Pantallas", "precio": 219.00,
     "stock": 25, "etiquetas": ["novedad"], "atributos": {"pulgadas": 27, "hz": 144}},
    {"codigo": "POR-14", "nombre": "Portátil 14 pulgadas", "categoria": "Equipos", "precio": 899.00,
     "stock": 10, "etiquetas": [], "atributos": {"ram_gb": 16, "ssd_gb": 512}},
    {"codigo": "AUR-BT", "nombre": "Auriculares Bluetooth", "categoria": "Audio", "precio": 79.00,
     "stock": 40, "etiquetas": ["oferta", "novedad"], "atributos": {"inalambrico": True}},
    {"codigo": "CAB-HD", "nombre": "Cable HDMI 2 m", "categoria": "Accesorios", "precio": 9.90,
     "stock": 500, "etiquetas": [], "atributos": {"longitud_m": 2}},
]
CIUDADES = ["Madrid", "Barcelona", "Sevilla", "Valencia", "Bilbao"]


class StockInsuficiente(Exception):
    pass


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# --------------------------------------------------------------- Preparación
def preparar(db) -> None:
    for nombre in ("productos", "clientes", "pedidos", "contadores"):
        db.drop_collection(nombre)
    db.productos.create_index("codigo", unique=True)
    db.productos.create_index([("categoria", ASCENDING), ("precio", ASCENDING)])
    db.productos.create_index("etiquetas")
    db.clientes.create_index("email", unique=True)
    db.pedidos.create_index([("fecha", DESCENDING)])
    db.pedidos.create_index("cliente.id")


def validar_producto(p: dict) -> dict:
    errores = []
    if not re.fullmatch(r"[A-Z]{3}-[A-Z0-9]{2}", p.get("codigo", "")):
        errores.append("código con formato incorrecto")
    if not isinstance(p.get("precio"), (int, float)) or p["precio"] < 0:
        errores.append("precio no válido")
    if not isinstance(p.get("stock"), int) or p["stock"] < 0:
        errores.append("stock no válido")
    if errores:
        raise ValueError(f"{p.get('codigo')}: {', '.join(errores)}")
    return p


def sincronizar_catalogo(db, catalogo: list[dict]) -> None:
    ahora = datetime.now(UTC)
    operaciones = [
        UpdateOne({"codigo": p["codigo"]},
                  {"$set": {**validar_producto(p), "actualizado": ahora},
                   "$setOnInsert": {"creado": ahora, "vendidos": 0}},
                  upsert=True)
        for p in catalogo
    ]
    r = db.productos.bulk_write(operaciones, ordered=False)
    print(f"Catálogo: {r.upserted_count} nuevos, {r.modified_count} actualizados")


def crear_clientes(db, n: int = 60) -> None:
    random.seed(4)
    db.clientes.insert_many([
        {"nombre": f"Cliente {i:03d}", "email": f"cliente{i}@ejemplo.com", "ciudad": random.choice(CIUDADES),
         "alta": datetime(2025, 1, 1, tzinfo=UTC) + timedelta(days=random.randint(0, 600))}
        for i in range(1, n + 1)
    ])


# --------------------------------------------------------------- Negocio
def siguiente_numero(db, serie: str) -> int:
    return db.contadores.find_one_and_update({"_id": serie}, {"$inc": {"valor": 1}},
                                             upsert=True, return_document=ReturnDocument.AFTER)["valor"]


def crear_pedido(db, cliente: dict, articulos: dict[str, int], fecha: datetime) -> dict:
    """Descuenta stock de forma atómica por producto; si alguno falla, revierte lo ya descontado."""
    lineas, descontados = [], []
    try:
        for codigo, cantidad in articulos.items():
            prod = db.productos.find_one_and_update(
                {"codigo": codigo, "stock": {"$gte": cantidad}},
                {"$inc": {"stock": -cantidad, "vendidos": cantidad}},
                projection={"codigo": 1, "nombre": 1, "categoria": 1, "precio": 1},
                return_document=ReturnDocument.AFTER,
            )
            if prod is None:
                raise StockInsuficiente(codigo)
            descontados.append((codigo, cantidad))
            lineas.append({"codigo": codigo, "nombre": prod["nombre"], "categoria": prod["categoria"],
                           "cantidad": cantidad, "precio": prod["precio"]})
    except StockInsuficiente:
        for codigo, cantidad in descontados:              # Compensación
            db.productos.update_one({"codigo": codigo}, {"$inc": {"stock": cantidad, "vendidos": -cantidad}})
        raise

    pedido = {
        "numero": siguiente_numero(db, "pedidos"),
        "fecha": fecha,
        "cliente": {"id": cliente["_id"], "nombre": cliente["nombre"], "ciudad": cliente["ciudad"]},
        "lineas": lineas,
        "total": round(sum(l["cantidad"] * l["precio"] for l in lineas), 2),
        "estado": "pendiente",
    }
    db.pedidos.insert_one(pedido)
    return pedido


def buscar_productos(db, texto: str | None = None, categoria: str | None = None,
                     precio_max: float | None = None, etiqueta: str | None = None,
                     solo_disponibles: bool = True, orden: str = "precio") -> list[dict]:
    filtro: dict = {}
    if texto:
        filtro["nombre"] = {"$regex": re.escape(texto), "$options": "i"}
    if categoria:
        filtro["categoria"] = categoria
    if precio_max is not None:
        filtro["precio"] = {"$lte": precio_max}
    if etiqueta:
        filtro["etiquetas"] = etiqueta
    if solo_disponibles:
        filtro["stock"] = {"$gt": 0}
    return list(db.productos.find(filtro, {"_id": 0, "codigo": 1, "nombre": 1, "precio": 1, "stock": 1})
                .sort(orden, ASCENDING))


# --------------------------------------------------------------- Informes
def ventas_por_mes(db) -> list[dict]:
    return list(db.pedidos.aggregate([
        {"$match": {"estado": {"$ne": "cancelado"}}},
        {"$group": {"_id": {"$dateToString": {"format": "%Y-%m", "date": "$fecha"}},
                    "pedidos": {"$sum": 1}, "ventas": {"$sum": "$total"}, "ticket_medio": {"$avg": "$total"}}},
        {"$sort": {"_id": 1}},
    ]))


def ventas_por_categoria(db) -> list[dict]:
    return list(db.pedidos.aggregate([
        {"$unwind": "$lineas"},
        {"$group": {"_id": "$lineas.categoria",
                    "unidades": {"$sum": "$lineas.cantidad"},
                    "ventas": {"$sum": {"$multiply": ["$lineas.cantidad", "$lineas.precio"]}}}},
        {"$sort": {"ventas": -1}},
    ]))


def mejores_clientes(db, n: int = 5) -> list[dict]:
    return list(db.pedidos.aggregate([
        {"$group": {"_id": "$cliente.id", "nombre": {"$first": "$cliente.nombre"},
                    "ciudad": {"$first": "$cliente.ciudad"}, "pedidos": {"$sum": 1}, "gastado": {"$sum": "$total"}}},
        {"$sort": {"gastado": -1}},
        {"$limit": n},
    ]))


# --------------------------------------------------------------- Principal
def main() -> None:
    with MongoClient(URI, serverSelectionTimeoutMS=5000) as cliente:
        try:
            cliente.admin.command("ping")
        except ConnectionFailure as e:
            raise SystemExit(f"No se puede conectar con MongoDB: {e}")

        db = cliente["tienda_demo"]
        preparar(db)
        sincronizar_catalogo(db, CATALOGO)
        crear_clientes(db)

        try:
            db.clientes.insert_one({"nombre": "Duplicado", "email": "cliente1@ejemplo.com", "ciudad": "Madrid"})
        except DuplicateKeyError:
            print("Control de duplicados por email ✔")

        random.seed(9)
        clientes = list(db.clientes.find())
        codigos = [p["codigo"] for p in CATALOGO]
        ok = rechazados = 0
        for _ in range(250):
            articulos = {random.choice(codigos): random.randint(1, 3) for _ in range(random.randint(1, 3))}
            fecha = datetime(2026, 11, 24, tzinfo=UTC) - timedelta(days=random.randint(0, 120))
            try:
                crear_pedido(db, random.choice(clientes), articulos, fecha)
                ok += 1
            except StockInsuficiente:
                rechazados += 1
        print(f"Pedidos: {ok} creados, {rechazados} rechazados por stock\n")

        print("Productos con etiqueta «oferta» por debajo de 100 €:")
        for p in buscar_productos(db, etiqueta="oferta", precio_max=100):
            print(f"  {p['codigo']}  {p['nombre']:<24} {euros(p['precio']):>10}  stock {p['stock']}")

        print("\nVentas por mes:")
        for m in ventas_por_mes(db):
            print(f"  {m['_id']}  {m['pedidos']:>3} pedidos  {euros(m['ventas']):>12}  ticket {euros(m['ticket_medio'])}")

        print("\nVentas por categoría:")
        for c in ventas_por_categoria(db):
            print(f"  {c['_id']:<12} {c['unidades']:>4} uds  {euros(c['ventas']):>12}")

        print("\nMejores clientes:")
        for c in mejores_clientes(db):
            print(f"  {c['nombre']:<12} {c['ciudad']:<10} {c['pedidos']:>2} pedidos  {euros(c['gastado']):>11}")

        agotados = db.productos.count_documents({"stock": 0})
        print(f"\nProductos agotados: {agotados}")

        with open("pedidos.json", "w", encoding="utf-8") as f:
            f.write(json_util.dumps(list(db.pedidos.find().sort("numero", 1).limit(20)),
                                    ensure_ascii=False, indent=2))
        print("Exportados los 20 primeros pedidos a pedidos.json")


if __name__ == "__main__":
    main()
```

---

## 16. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ServerSelectionTimeoutError` | Servidor no disponible, URI incorrecta o IP no autorizada (Atlas). | Compruebe el servicio, la URI y la lista de IPs permitidas. |
| `OperationFailure: Authentication failed` | Credenciales incorrectas o `authSource` equivocado. | Revise usuario, contraseña y `?authSource=admin`. |
| `InvalidURI` con contraseña | Caracteres especiales sin codificar. | `urllib.parse.quote_plus(clave)`. |
| `DuplicateKeyError` | Valor repetido en índice único. | Capture la excepción o use *upsert*. |
| `find_one({"_id": "6743..."})` devuelve `None` | El `_id` es `ObjectId`, no texto. | Use `ObjectId(texto)`. |
| `TypeError: Object of type ObjectId is not JSON serializable` | `json.dumps` no conoce `ObjectId` ni fechas BSON. | Use `bson.json_util.dumps` o convierta con `str()`. |
| `ValueError: update only works with $ operators` | `update_one` sin `$set` u otro operador. | Use operadores o `replace_one`. |
| `bson.errors.InvalidDocument: cannot encode object: datetime.date` | `date` no es compatible con BSON. | Use `datetime`. |
| `DocumentTooLarge` | Documento > 16 MB. | Divida los datos o use GridFS. |
| `OperationFailure: Transaction numbers are only allowed on a replica set` | Transacciones en servidor independiente. | Use un replica set o Atlas. |
| Consultas lentas | Sin índice adecuado (`COLLSCAN`). | Cree índices y revise con `explain()`. |
| Se crea una colección con nombre mal escrito | MongoDB crea colecciones automáticamente. | Revise los nombres; use constantes. |

---

## 17. Buenas prácticas

- Cree **un único** `MongoClient` por aplicación y reutilícelo (es seguro entre hilos).
- Guarde la URI de conexión en variables de entorno.
- Diseñe los documentos según **cómo se leen**: embeba lo que se consulta junto.
- Cree índices para los campos usados en filtros y ordenaciones; verifique con `explain()`.
- Use operadores atómicos (`$inc`, `find_one_and_update`) para evitar condiciones de carrera.
- Valide los datos antes de insertarlos (Pydantic o validación de esquema de MongoDB).
- Use `bulk_write` para cargas y sincronizaciones masivas.
- Use `datetime` con zona horaria UTC y `Decimal128` para importes exactos.
- Use proyecciones para devolver solo los campos necesarios.
- Escape con `re.escape` el texto del usuario en expresiones regulares.

---

## 18. Referencia rápida

```python
cli = MongoClient(uri, serverSelectionTimeoutMS=5000); db = cli["bd"]; col = db["coleccion"]
cli.admin.command("ping")

col.insert_one(doc).inserted_id / col.insert_many(docs, ordered=False)
col.find_one(filtro) / col.find(filtro, proyeccion).sort(campo, -1).skip(n).limit(m)
col.count_documents(filtro) / col.distinct(campo)
col.update_one(f, {"$set": {...}}, upsert=True) / update_many / replace_one
col.find_one_and_update(f, {"$inc": {...}}, return_document=ReturnDocument.AFTER)
col.delete_one(f) / delete_many(f) / col.drop()
col.create_index([("a", 1), ("b", -1)], unique=True) / col.index_information()
col.aggregate([{"$match": ...}, {"$group": {"_id": "$campo", "n": {"$sum": 1}}}, {"$sort": {"n": -1}}])
col.bulk_write([InsertOne(...), UpdateOne(...), DeleteOne(...)])
with cli.start_session() as s, s.start_transaction(): ...

Operadores: $eq $ne $gt $gte $lt $lte $in $nin $and $or $not $exists $regex $all $elemMatch $size
Actualización: $set $unset $inc $mul $push $addToSet $pull $setOnInsert $currentDate
ObjectId(texto) / bson.json_util.dumps(docs)
```

---

## 19. Ejercicios propuestos

1. Modele un blog con artículos, comentarios embebidos y autores referenciados; obtenga los 5 artículos más comentados.
2. Importe un CSV de productos con `bulk_write` usando *upsert* por código.
3. Cree una colección de sesiones con índice TTL que elimine los documentos a los 30 minutos.
4. Escriba una agregación con las ventas mensuales por ciudad y la variación respecto al mes anterior.
5. Implemente una búsqueda de tiendas cercanas a unas coordenadas usando un índice `2dsphere`.

---

## 20. Referencias

- Documentación de PyMongo: https://pymongo.readthedocs.io/
- Manual de MongoDB: https://www.mongodb.com/docs/manual/
- Canal de agregación: https://www.mongodb.com/docs/manual/aggregation/
- MongoDB Atlas (nivel gratuito): https://www.mongodb.com/atlas
