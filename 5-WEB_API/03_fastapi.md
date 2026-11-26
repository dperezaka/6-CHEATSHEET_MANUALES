# Manual de uso: FastAPI

> **Categoría:** Desarrollo web · **Utilidad:** Crear APIs modernas y rápidas
> **Tipo:** Librería de terceros (`pip install "fastapi[standard]"`) · **Versión de referencia:** 0.115 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y ejecución](#2-instalación-y-ejecución)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera API](#4-primera-api)
5. [Parámetros de ruta y de consulta](#5-parámetros-de-ruta-y-de-consulta)
6. [Cuerpo de la petición con Pydantic](#6-cuerpo-de-la-petición-con-pydantic)
7. [Modelos de respuesta y códigos de estado](#7-modelos-de-respuesta-y-códigos-de-estado)
8. [Errores y validación](#8-errores-y-validación)
9. [Formularios, archivos, cabeceras y cookies](#9-formularios-archivos-cabeceras-y-cookies)
10. [Inyección de dependencias](#10-inyección-de-dependencias)
11. [Organización con `APIRouter`](#11-organización-con-apirouter)
12. [Base de datos con SQLModel](#12-base-de-datos-con-sqlmodel)
13. [Seguridad: OAuth2 y JWT](#13-seguridad-oauth2-y-jwt)
14. [Asincronía: `async def` o `def`](#14-asincronía-async-def-o-def)
15. [Tareas en segundo plano, *lifespan* y *middleware*](#15-tareas-en-segundo-plano-lifespan-y-middleware)
16. [CORS, archivos estáticos y plantillas](#16-cors-archivos-estáticos-y-plantillas)
17. [WebSockets y *streaming*](#17-websockets-y-streaming)
18. [Documentación OpenAPI](#18-documentación-openapi)
19. [Configuración con `pydantic-settings`](#19-configuración-con-pydantic-settings)
20. [Pruebas](#20-pruebas)
21. [Despliegue](#21-despliegue)
22. [Ejemplo práctico completo](#22-ejemplo-práctico-completo)
23. [Errores frecuentes](#23-errores-frecuentes)
24. [Buenas prácticas](#24-buenas-prácticas)
25. [Referencia rápida](#25-referencia-rápida)
26. [Ejercicios propuestos](#26-ejercicios-propuestos)
27. [Referencias](#27-referencias)

---

## 1. Descripción

**FastAPI** es un *framework* para construir **APIs web** con Python basándose en las **anotaciones de tipo**. A partir de la firma de cada función genera automáticamente la validación de datos (con **Pydantic**), la serialización de respuestas y la **documentación interactiva OpenAPI** (Swagger UI y ReDoc). Funciona sobre **Starlette** (ASGI), por lo que admite código asíncrono, WebSockets y un rendimiento muy alto.

**Ventajas:**
- Validación y conversión automáticas de parámetros, cuerpos y respuestas.
- Documentación interactiva generada sin esfuerzo (`/docs`, `/redoc`).
- Excelente integración con el editor: autocompletado y comprobación de tipos.
- Sistema de **inyección de dependencias** sencillo y potente.
- Soporte nativo de `async`/`await`, WebSockets y tareas en segundo plano.
- Rendimiento entre los más altos de los *frameworks* Python.

**Limitaciones:**
- Centrado en APIs: no incluye ORM, panel de administración ni sistema de usuarios.
- Hay que elegir y combinar piezas (SQLModel/SQLAlchemy, Alembic, autenticación).
- Mezclar código bloqueante en funciones `async` degrada el rendimiento.
- La versión aún no es 1.0: revise las notas de cada actualización.

| Si necesita… | Opción |
|---|---|
| API REST/JSON moderna con validación y documentación automáticas | **FastAPI** |
| Aplicación web completa con HTML, admin y usuarios | Django |
| Aplicación web ligera con plantillas | Flask |
| API sobre un proyecto Django existente | Django REST Framework / Django Ninja |

---

## 2. Instalación y ejecución

```bash
python -m venv .venv
source .venv/bin/activate                 # Windows: .venv\Scripts\activate
pip install "fastapi[standard]"           # Incluye uvicorn, fastapi-cli, httpx, python-multipart…
```

```bash
fastapi dev main.py                       # Desarrollo: recarga automática · http://127.0.0.1:8000/docs
fastapi run main.py                       # Producción (sin recarga)
uvicorn main:app --reload                 # Alternativa con Uvicorn directamente
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `FastAPI()` | Aplicación ASGI principal. |
| Operación de ruta | Función decorada con `@app.get`, `@app.post`… |
| Parámetro de ruta | Parte variable de la URL: `/items/{id}`. |
| Parámetro de consulta | Parámetros tras `?`: `/items?limite=10`. |
| Cuerpo (*body*) | Datos JSON de la petición, definidos con un modelo Pydantic. |
| Modelo Pydantic | Clase que declara la estructura y validación de los datos. |
| `response_model` | Modelo que filtra y documenta la respuesta. |
| Dependencia (`Depends`) | Función reutilizable que FastAPI ejecuta e inyecta (BD, usuario actual…). |
| `APIRouter` | Grupo de rutas que se incluye en la aplicación. |
| OpenAPI | Especificación generada automáticamente a partir del código. |
| ASGI | Interfaz asíncrona entre servidor (Uvicorn) y aplicación. |

---

## 4. Primera API

```python
# main.py
from fastapi import FastAPI

app = FastAPI(title="Mi API", version="1.0.0")

@app.get("/")
def raiz():
    return {"mensaje": "Hola, FastAPI"}

@app.get("/saludo/{nombre}")
def saludo(nombre: str, veces: int = 1):
    return {"saludo": " ".join([f"Hola, {nombre}"] * veces)}
```

```bash
fastapi dev main.py
curl "http://127.0.0.1:8000/saludo/Ana?veces=2"
# {"saludo":"Hola, Ana Hola, Ana"}
curl "http://127.0.0.1:8000/saludo/Ana?veces=dos"
# 422: {"detail":[{"type":"int_parsing","loc":["query","veces"],"msg":"Input should be a valid integer…"}]}
```

Abra `http://127.0.0.1:8000/docs` para probar la API desde el navegador.

---

## 5. Parámetros de ruta y de consulta

### 5.1 Reglas

- Si el nombre aparece en la ruta (`{id}`), es un **parámetro de ruta**.
- Si es un tipo simple (`int`, `str`, `bool`…) y no está en la ruta, es un **parámetro de consulta**.
- Si es un modelo Pydantic, es el **cuerpo** de la petición.

### 5.2 Validaciones con `Path` y `Query`

```python
from typing import Annotated, Literal
from fastapi import FastAPI, Path, Query

@app.get("/productos/{producto_id}")
def obtener(producto_id: Annotated[int, Path(ge=1, description="Identificador del producto")]):
    ...

@app.get("/productos")
def listar(
    q: Annotated[str | None, Query(min_length=2, max_length=50)] = None,
    categoria: Literal["perifericos", "monitores", "redes"] | None = None,
    precio_max: Annotated[float | None, Query(gt=0)] = None,
    disponible: bool = True,                                     # ?disponible=false / 0 / no
    etiquetas: Annotated[list[str], Query()] = [],               # ?etiquetas=a&etiquetas=b
    pagina: Annotated[int, Query(ge=1)] = 1,
    tamano: Annotated[int, Query(ge=1, le=100)] = 20,
):
    ...
```

| Validación | Tipos | Significado |
|---|---|---|
| `gt`, `ge`, `lt`, `le` | Numéricos | Mayor que, mayor o igual… |
| `min_length`, `max_length` | Texto / listas | Longitud. |
| `pattern` | Texto | Expresión regular. |
| `alias` | Todos | Nombre distinto en la URL (`alias="item-id"`). |
| `deprecated`, `include_in_schema` | Todos | Documentación. |

### 5.3 Valores cerrados con `Enum`

```python
from enum import Enum

class Orden(str, Enum):
    nombre = "nombre"
    precio = "precio"
    fecha = "fecha"

@app.get("/productos")
def listar(orden: Orden = Orden.nombre): ...
```

### 5.4 Parámetros de consulta agrupados en un modelo

```python
from pydantic import BaseModel, Field

class Filtros(BaseModel):
    model_config = {"extra": "forbid"}                  # Rechaza parámetros desconocidos
    q: str | None = None
    pagina: int = Field(1, ge=1)
    tamano: int = Field(20, ge=1, le=100)

@app.get("/productos")
def listar(filtros: Annotated[Filtros, Query()]): ...
```

> Declare las rutas fijas **antes** que las variables: `/productos/destacados` debe ir antes que `/productos/{producto_id}`.

---

## 6. Cuerpo de la petición con Pydantic

### 6.1 Modelos

```python
from datetime import date
from decimal import Decimal
from pydantic import BaseModel, EmailStr, Field, field_validator, model_validator

class Direccion(BaseModel):
    calle: str
    ciudad: str
    codigo_postal: str = Field(pattern=r"^\d{5}$")

class ClienteCrear(BaseModel):
    nombre: str = Field(min_length=2, max_length=100, examples=["Ana Pérez"])
    email: EmailStr                                     # Requiere email-validator (incluido en [standard])
    fecha_nacimiento: date | None = None
    limite_credito: Decimal = Field(default=Decimal("0"), ge=0, max_digits=10, decimal_places=2)
    etiquetas: list[str] = []
    direccion: Direccion | None = None

    @field_validator("nombre")
    @classmethod
    def normalizar_nombre(cls, v: str) -> str:
        return " ".join(v.split()).title()

    @model_validator(mode="after")
    def comprobar_edad(self):
        if self.fecha_nacimiento and self.fecha_nacimiento > date.today():
            raise ValueError("La fecha de nacimiento no puede ser futura")
        return self

@app.post("/clientes")
def crear_cliente(cliente: ClienteCrear):
    return cliente.model_dump()
```

### 6.2 Métodos útiles de Pydantic v2

| Método | Uso |
|---|---|
| `modelo.model_dump(exclude_unset=True)` | Diccionario (solo campos enviados: ideal para PATCH). |
| `modelo.model_dump_json()` | JSON. |
| `Modelo.model_validate(dict_u_objeto)` | Crear y validar a partir de datos. |
| `modelo.model_copy(update={...})` | Copia con cambios. |
| `model_config = {"from_attributes": True}` | Crear desde objetos ORM. |

### 6.3 Varios cuerpos y valores sueltos

```python
from fastapi import Body

@app.put("/pedidos/{id}")
def actualizar(id: int, pedido: PedidoActualizar, cliente: ClienteRef,
               prioridad: Annotated[int, Body(ge=1, le=5)] = 3):
    ...
# Cuerpo esperado: {"pedido": {...}, "cliente": {...}, "prioridad": 2}
```

---

## 7. Modelos de respuesta y códigos de estado

Separe los modelos de entrada y de salida (por ejemplo, para no devolver contraseñas):

```python
from fastapi import status

class UsuarioCrear(BaseModel):
    email: EmailStr
    clave: str = Field(min_length=8)

class UsuarioPublico(BaseModel):
    id: int
    email: EmailStr

@app.post("/usuarios", response_model=UsuarioPublico, status_code=status.HTTP_201_CREATED)
def crear_usuario(datos: UsuarioCrear):
    usuario = guardar(datos)                # Devuelve un objeto con 'clave_hash'
    return usuario                          # response_model filtra: solo id y email
```

También puede indicar el tipo de retorno (FastAPI lo usa como `response_model`):

```python
@app.get("/usuarios/{id}")
def obtener(id: int) -> UsuarioPublico:
    ...
```

Opciones:

```python
@app.get("/productos", response_model=list[ProductoPublico], response_model_exclude_none=True)
@app.delete("/productos/{id}", status_code=204)               # Sin cuerpo de respuesta
```

Respuestas especiales:

```python
from fastapi.responses import JSONResponse, HTMLResponse, RedirectResponse, FileResponse, PlainTextResponse

@app.get("/informe", response_class=FileResponse)
def informe():
    return "informes/ventas.pdf"

@app.get("/antiguo")
def antiguo():
    return RedirectResponse("/nuevo", status_code=301)
```

---

## 8. Errores y validación

### 8.1 `HTTPException`

```python
from fastapi import HTTPException, status

@app.get("/productos/{id}")
def obtener(id: int):
    producto = repo.get(id)
    if producto is None:
        raise HTTPException(status.HTTP_404_NOT_FOUND, detail=f"Producto {id} no encontrado")
    return producto
```

### 8.2 Excepciones propias con manejador

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class StockInsuficiente(Exception):
    def __init__(self, producto_id: int, disponible: int):
        self.producto_id, self.disponible = producto_id, disponible

@app.exception_handler(StockInsuficiente)
async def manejar_stock(request: Request, exc: StockInsuficiente):
    return JSONResponse(status_code=409, content={
        "error": "stock_insuficiente", "producto_id": exc.producto_id, "disponible": exc.disponible})
```

### 8.3 Errores de validación (422)

Cuando la entrada no cumple los tipos o restricciones, FastAPI responde **422** con el detalle:

```json
{"detail": [{"type": "string_too_short", "loc": ["body", "nombre"], "msg": "String should have at least 2 characters", "input": "A"}]}
```

Personalizar el formato:

```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validacion(request: Request, exc: RequestValidationError):
    errores = {".".join(map(str, e["loc"][1:])): e["msg"] for e in exc.errors()}
    return JSONResponse(status_code=422, content={"error": "datos_no_validos", "campos": errores})
```

---

## 9. Formularios, archivos, cabeceras y cookies

```python
from fastapi import Form, File, UploadFile, Header, Cookie

@app.post("/login-form")
def login_form(usuario: Annotated[str, Form()], clave: Annotated[str, Form()]):
    ...

@app.post("/subir")
async def subir(archivo: UploadFile, descripcion: Annotated[str, Form()] = ""):
    contenido = await archivo.read()                        # bytes
    if archivo.content_type not in ("image/png", "image/jpeg"):
        raise HTTPException(415, "Solo se admiten PNG o JPEG")
    if len(contenido) > 5 * 1024 * 1024:
        raise HTTPException(413, "Archivo demasiado grande")
    return {"nombre": archivo.filename, "bytes": len(contenido)}

@app.post("/subir-varios")
async def subir_varios(archivos: list[UploadFile]): ...

@app.get("/cabeceras")
def cabeceras(user_agent: Annotated[str | None, Header()] = None,     # User-Agent
              x_token: Annotated[str | None, Header()] = None,        # X-Token
              sesion: Annotated[str | None, Cookie()] = None):
    ...
```

> Formularios y archivos necesitan `python-multipart` (incluido en `fastapi[standard]`). No se pueden mezclar `Form`/`File` con un cuerpo JSON en la misma operación.

---

## 10. Inyección de dependencias

### 10.1 Dependencias como funciones

```python
from fastapi import Depends

def paginacion(pagina: int = Query(1, ge=1), tamano: int = Query(20, ge=1, le=100)) -> dict:
    return {"offset": (pagina - 1) * tamano, "limit": tamano}

Paginacion = Annotated[dict, Depends(paginacion)]          # Alias reutilizable

@app.get("/productos")
def listar(p: Paginacion):
    return repo.listar(**p)

@app.get("/clientes")
def listar_clientes(p: Paginacion):
    ...
```

### 10.2 Dependencias con `yield` (recursos con limpieza)

```python
def get_sesion():
    with Session(motor) as sesion:
        yield sesion                         # Se cierra al terminar la petición

SesionDep = Annotated[Session, Depends(get_sesion)]
```

### 10.3 Dependencias encadenadas y de clase

```python
def usuario_actual(token: Annotated[str, Depends(oauth2_scheme)], sesion: SesionDep) -> Usuario: ...

def admin_actual(usuario: Annotated[Usuario, Depends(usuario_actual)]) -> Usuario:
    if not usuario.es_admin:
        raise HTTPException(403, "Se requieren permisos de administrador")
    return usuario

class FiltroFechas:
    def __init__(self, desde: date | None = None, hasta: date | None = None):
        self.desde, self.hasta = desde, hasta

@app.get("/ventas")
def ventas(f: Annotated[FiltroFechas, Depends()]): ...
```

### 10.4 Dependencias globales o por grupo

```python
app = FastAPI(dependencies=[Depends(verificar_api_key)])
router = APIRouter(dependencies=[Depends(admin_actual)])
@app.get("/x", dependencies=[Depends(registrar_acceso)])
```

### 10.5 Sustituir dependencias en pruebas

```python
app.dependency_overrides[get_sesion] = get_sesion_pruebas
app.dependency_overrides[usuario_actual] = lambda: Usuario(id=1, email="test@x.com")
```

---

## 11. Organización con `APIRouter`

```
app/
├── main.py              # Crea FastAPI e incluye los routers
├── config.py            # Configuración
├── database.py          # Motor y dependencia de sesión
├── modelos.py           # Tablas / modelos Pydantic
├── seguridad.py         # Hash, JWT, usuario_actual
└── routers/
    ├── productos.py
    └── usuarios.py
```

```python
# app/routers/productos.py
from fastapi import APIRouter

router = APIRouter(prefix="/productos", tags=["productos"])

@router.get("/")
def listar(): ...

@router.get("/{id}")
def obtener(id: int): ...
```

```python
# app/main.py
from fastapi import FastAPI
from app.routers import productos, usuarios

app = FastAPI()
app.include_router(productos.router)
app.include_router(usuarios.router, prefix="/api/v1")
```

---

## 12. Base de datos con SQLModel

**SQLModel** (del mismo autor) combina SQLAlchemy y Pydantic: una misma clase sirve como tabla y como modelo de validación.

```bash
pip install sqlmodel
```

```python
from sqlmodel import Field, Session, SQLModel, create_engine, select

class ProductoBase(SQLModel):
    nombre: str = Field(min_length=2, max_length=100, index=True)
    precio: float = Field(gt=0)
    stock: int = Field(default=0, ge=0)

class Producto(ProductoBase, table=True):            # Tabla
    id: int | None = Field(default=None, primary_key=True)

class ProductoCrear(ProductoBase):                   # Entrada
    pass

class ProductoActualizar(SQLModel):                  # Entrada parcial (PATCH)
    nombre: str | None = None
    precio: float | None = Field(default=None, gt=0)
    stock: int | None = Field(default=None, ge=0)

motor = create_engine("sqlite:///tienda.db", connect_args={"check_same_thread": False})

def get_sesion():
    with Session(motor) as sesion:
        yield sesion

SesionDep = Annotated[Session, Depends(get_sesion)]

@app.post("/productos", response_model=Producto, status_code=201)
def crear(datos: ProductoCrear, sesion: SesionDep):
    producto = Producto.model_validate(datos)
    sesion.add(producto)
    sesion.commit()
    sesion.refresh(producto)
    return producto

@app.get("/productos", response_model=list[Producto])
def listar(sesion: SesionDep, q: str | None = None, offset: int = 0, limit: int = Query(20, le=100)):
    consulta = select(Producto)
    if q:
        consulta = consulta.where(Producto.nombre.contains(q))
    return sesion.exec(consulta.offset(offset).limit(limit)).all()

@app.patch("/productos/{id}", response_model=Producto)
def actualizar(id: int, datos: ProductoActualizar, sesion: SesionDep):
    producto = sesion.get(Producto, id)
    if not producto:
        raise HTTPException(404, "No encontrado")
    producto.sqlmodel_update(datos.model_dump(exclude_unset=True))
    sesion.add(producto)
    sesion.commit()
    sesion.refresh(producto)
    return producto
```

Alternativas: **SQLAlchemy 2.x** directamente (síncrono o `AsyncSession`), **Tortoise ORM**, o consultas SQL con `asyncpg`. Para migraciones: **Alembic**.

---

## 13. Seguridad: OAuth2 y JWT

### 13.1 Clave de API sencilla

```python
from fastapi.security import APIKeyHeader

api_key_header = APIKeyHeader(name="X-API-Key")

def verificar_api_key(clave: Annotated[str, Depends(api_key_header)]):
    if not secrets.compare_digest(clave, settings.api_key):
        raise HTTPException(401, "Clave de API no válida")
```

### 13.2 OAuth2 con contraseña y *token* JWT

```python
from datetime import datetime, timedelta, timezone
import jwt                                                    # pip install pyjwt
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

SECRET_KEY = os.environ["SECRET_KEY"]
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def crear_token(sub: str, minutos: int = 30) -> str:
    exp = datetime.now(timezone.utc) + timedelta(minutes=minutos)
    return jwt.encode({"sub": sub, "exp": exp}, SECRET_KEY, algorithm="HS256")

@app.post("/token")
def login(form: Annotated[OAuth2PasswordRequestForm, Depends()], sesion: SesionDep):
    usuario = autenticar(sesion, form.username, form.password)
    if not usuario:
        raise HTTPException(401, "Credenciales incorrectas", headers={"WWW-Authenticate": "Bearer"})
    return {"access_token": crear_token(usuario.email), "token_type": "bearer"}

def usuario_actual(token: Annotated[str, Depends(oauth2_scheme)], sesion: SesionDep) -> Usuario:
    credenciales = HTTPException(401, "Token no válido", headers={"WWW-Authenticate": "Bearer"})
    try:
        email = jwt.decode(token, SECRET_KEY, algorithms=["HS256"]).get("sub")
    except jwt.InvalidTokenError:
        raise credenciales
    usuario = sesion.exec(select(Usuario).where(Usuario.email == email)).first()
    if usuario is None:
        raise credenciales
    return usuario

@app.get("/yo")
def yo(usuario: Annotated[Usuario, Depends(usuario_actual)]):
    return {"email": usuario.email}
```

> Guarde las contraseñas con un algoritmo de hash adecuado (`pwdlib` con Argon2, `bcrypt`, o `hashlib.pbkdf2_hmac` de la librería estándar). En `/docs`, el botón **Authorize** permite probar las rutas protegidas.

---

## 14. Asincronía: `async def` o `def`

| Use… | Cuando… |
|---|---|
| `async def` | El código usa librerías asíncronas (`httpx.AsyncClient`, `asyncpg`, `aiofiles`) y `await`. |
| `def` | El código usa librerías bloqueantes (SQLAlchemy síncrono, `requests`, lectura de archivos). FastAPI lo ejecuta en un *pool* de hilos. |

```python
import httpx

@app.get("/tiempo/{ciudad}")
async def tiempo(ciudad: str):
    async with httpx.AsyncClient(timeout=10) as cliente:
        r = await cliente.get("https://api.ejemplo.com/tiempo", params={"q": ciudad})
    return r.json()
```

> **Nunca** use código bloqueante (`time.sleep`, `requests.get`, consultas síncronas) dentro de `async def`: bloquea todo el servidor. Si debe hacerlo, use `def` o `await run_in_threadpool(funcion)`.

---

## 15. Tareas en segundo plano, *lifespan* y *middleware*

### 15.1 `BackgroundTasks`

Se ejecutan **después** de enviar la respuesta:

```python
from fastapi import BackgroundTasks

def enviar_email(destino: str, asunto: str) -> None:
    ...

@app.post("/pedidos", status_code=201)
def crear_pedido(pedido: PedidoCrear, tareas: BackgroundTasks):
    nuevo = guardar(pedido)
    tareas.add_task(enviar_email, pedido.email, f"Pedido {nuevo.id} recibido")
    return nuevo
```

> Para tareas largas, programadas o con reintentos, use una cola (Celery, ARQ, RQ, Dramatiq).

### 15.2 *Lifespan* (inicio y cierre)

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    SQLModel.metadata.create_all(motor)          # Al arrancar
    app.state.http = httpx.AsyncClient()
    yield
    await app.state.http.aclose()                # Al detener

app = FastAPI(lifespan=lifespan)
```

### 15.3 *Middleware*

```python
import time

@app.middleware("http")
async def tiempo_respuesta(request: Request, call_next):
    inicio = time.perf_counter()
    respuesta = await call_next(request)
    respuesta.headers["X-Tiempo"] = f"{(time.perf_counter() - inicio) * 1000:.1f}ms"
    return respuesta

from fastapi.middleware.gzip import GZipMiddleware
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

---

## 16. CORS, archivos estáticos y plantillas

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.ejemplo.com", "http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

```python
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app.mount("/static", StaticFiles(directory="static"), name="static")
plantillas = Jinja2Templates(directory="templates")

@app.get("/panel", response_class=HTMLResponse)
def panel(request: Request):
    return plantillas.TemplateResponse(request, "panel.html", {"titulo": "Panel"})
```

---

## 17. WebSockets y *streaming*

```python
from fastapi import WebSocket, WebSocketDisconnect

clientes: set[WebSocket] = set()

@app.websocket("/ws/chat")
async def chat(ws: WebSocket):
    await ws.accept()
    clientes.add(ws)
    try:
        while True:
            mensaje = await ws.receive_text()
            for c in clientes:
                await c.send_text(mensaje)
    except WebSocketDisconnect:
        clientes.discard(ws)
```

Respuestas por partes (archivos grandes, eventos):

```python
from fastapi.responses import StreamingResponse

async def generar_csv():
    yield "id;nombre\n"
    async for fila in repo.iterar():
        yield f"{fila.id};{fila.nombre}\n"

@app.get("/exportar")
def exportar():
    return StreamingResponse(generar_csv(), media_type="text/csv",
                             headers={"Content-Disposition": "attachment; filename=datos.csv"})
```

---

## 18. Documentación OpenAPI

```python
app = FastAPI(
    title="API de inventario",
    description="Gestión de **productos** y **pedidos**.",
    version="2.1.0",
    contact={"name": "Equipo de plataforma", "email": "plataforma@ejemplo.com"},
    openapi_tags=[{"name": "productos", "description": "Catálogo"}],
    docs_url="/docs", redoc_url="/redoc",           # None para desactivarlas
)

@app.get("/productos/{id}", tags=["productos"], summary="Obtener un producto",
         responses={404: {"description": "Producto no encontrado"}})
def obtener(id: int):
    """
    Devuelve un producto por su identificador.

    - **id**: identificador numérico.
    """
```

| URL | Contenido |
|---|---|
| `/docs` | Swagger UI (interactivo). |
| `/redoc` | ReDoc (lectura). |
| `/openapi.json` | Especificación (para generar clientes, Postman, etc.). |

---

## 19. Configuración con `pydantic-settings`

```bash
pip install pydantic-settings          # Incluido en fastapi[standard]
```

```python
from functools import lru_cache
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_prefix="APP_")
    database_url: str = "sqlite:///app.db"
    secret_key: str
    token_minutos: int = 30
    debug: bool = False

@lru_cache
def get_settings() -> Settings:
    return Settings()                      # Lee APP_DATABASE_URL, APP_SECRET_KEY… del entorno o .env

@app.get("/info")
def info(settings: Annotated[Settings, Depends(get_settings)]):
    return {"debug": settings.debug}
```

---

## 20. Pruebas

```python
from fastapi.testclient import TestClient
from main import app

cliente = TestClient(app)

def test_raiz():
    r = cliente.get("/")
    assert r.status_code == 200
    assert r.json() == {"mensaje": "Hola, FastAPI"}

def test_validacion():
    r = cliente.get("/saludo/Ana", params={"veces": "dos"})
    assert r.status_code == 422
```

Con base de datos de prueba y dependencias sustituidas:

```python
import pytest
from sqlmodel import SQLModel, Session, create_engine
from sqlmodel.pool import StaticPool

@pytest.fixture
def cliente():
    motor = create_engine("sqlite://", connect_args={"check_same_thread": False}, poolclass=StaticPool)
    SQLModel.metadata.create_all(motor)

    def sesion_pruebas():
        with Session(motor) as s:
            yield s

    app.dependency_overrides[get_sesion] = sesion_pruebas
    with TestClient(app) as c:                 # 'with' ejecuta el lifespan
        yield c
    app.dependency_overrides.clear()
```

Pruebas asíncronas: `httpx.AsyncClient(transport=httpx.ASGITransport(app=app), base_url="http://test")` con `pytest-anyio` o `pytest-asyncio`.

---

## 21. Despliegue

```bash
fastapi run app/main.py --workers 4 --port 8000
# o
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4 --proxy-headers
# o (gestión de procesos con Gunicorn, Linux)
gunicorn app.main:app -k uvicorn.workers.UvicornWorker -w 4 -b 0.0.0.0:8000
```

`Dockerfile` mínimo:

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app ./app
CMD ["fastapi", "run", "app/main.py", "--port", "80", "--proxy-headers"]
```

Lista de comprobación: HTTPS en el proxy (Nginx, Traefik), secretos en variables de entorno, CORS restringido, límites de tamaño de petición, registro de errores, `/docs` desactivado o protegido si la API no es pública, migraciones con Alembic.

---

## 22. Ejemplo práctico completo

**API de gestión de tareas por proyectos** con SQLModel, registro e inicio de sesión con JWT, contraseñas con PBKDF2 (librería estándar), modelos de entrada/salida separados, CRUD con permisos por propietario, filtros y paginación con dependencias, PATCH parcial, estadísticas, tarea en segundo plano, *lifespan*, *middleware*, manejador de errores propio, CORS y pruebas con `TestClient`.

```python
"""
tareas_api.py — API de tareas con FastAPI + SQLModel + JWT.
Requiere: pip install "fastapi[standard]" sqlmodel pyjwt
Uso:      fastapi dev tareas_api.py     → http://127.0.0.1:8000/docs
"""
import hashlib
import hmac
import logging
import os
import secrets
import time
from contextlib import asynccontextmanager
from datetime import date, datetime, timedelta, timezone
from enum import Enum
from typing import Annotated

import jwt
from fastapi import BackgroundTasks, Depends, FastAPI, HTTPException, Query, Request, status
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel, EmailStr
from sqlmodel import Field, Relationship, Session, SQLModel, create_engine, func, select

log = logging.getLogger("tareas_api")
SECRET_KEY = os.environ.get("APP_SECRET_KEY", "dev-" + "x" * 32)       # ¡Defina APP_SECRET_KEY en producción!
TOKEN_MINUTOS = 60
motor = create_engine(os.environ.get("APP_DATABASE_URL", "sqlite:///tareas.db"),
                      connect_args={"check_same_thread": False})


# ------------------------------------------------------------------ Modelos
class Prioridad(str, Enum):
    baja = "baja"
    media = "media"
    alta = "alta"


class Usuario(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True)
    nombre: str
    clave_hash: str
    proyectos: list["Proyecto"] = Relationship(back_populates="propietario")


class Proyecto(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    nombre: str = Field(index=True)
    descripcion: str = ""
    propietario_id: int = Field(foreign_key="usuario.id", index=True)
    propietario: Usuario = Relationship(back_populates="proyectos")
    tareas: list["Tarea"] = Relationship(back_populates="proyecto", cascade_delete=True)


class TareaBase(SQLModel):
    titulo: str = Field(min_length=3, max_length=200)
    descripcion: str = Field(default="", max_length=2000)
    prioridad: Prioridad = Prioridad.media
    vence: date | None = None


class Tarea(TareaBase, table=True):
    id: int | None = Field(default=None, primary_key=True)
    hecha: bool = False
    creada: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
    proyecto_id: int = Field(foreign_key="proyecto.id", index=True, ondelete="CASCADE")
    proyecto: Proyecto = Relationship(back_populates="tareas")


# Esquemas de entrada / salida
class UsuarioCrear(BaseModel):
    email: EmailStr
    nombre: str = Field(min_length=2, max_length=80)
    clave: str = Field(min_length=8, max_length=128)


class UsuarioPublico(BaseModel):
    id: int
    email: EmailStr
    nombre: str


class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"


class ProyectoCrear(BaseModel):
    nombre: str = Field(min_length=2, max_length=100)
    descripcion: str = ""


class ProyectoPublico(ProyectoCrear):
    id: int
    total_tareas: int = 0
    pendientes: int = 0


class TareaCrear(TareaBase):
    pass


class TareaActualizar(SQLModel):
    titulo: str | None = Field(default=None, min_length=3, max_length=200)
    descripcion: str | None = None
    prioridad: Prioridad | None = None
    vence: date | None = None
    hecha: bool | None = None


class TareaPublica(TareaBase):
    id: int
    hecha: bool
    creada: datetime
    proyecto_id: int


class PaginaTareas(BaseModel):
    total: int
    pagina: int
    tamano: int
    resultados: list[TareaPublica]


# ------------------------------------------------------------------ Seguridad
def hash_clave(clave: str) -> str:
    sal = secrets.token_hex(16)
    digest = hashlib.pbkdf2_hmac("sha256", clave.encode(), bytes.fromhex(sal), 600_000).hex()
    return f"pbkdf2_sha256$600000${sal}${digest}"


def verificar_clave(clave: str, almacenado: str) -> bool:
    _, iteraciones, sal, digest = almacenado.split("$")
    calculado = hashlib.pbkdf2_hmac("sha256", clave.encode(), bytes.fromhex(sal), int(iteraciones)).hex()
    return hmac.compare_digest(calculado, digest)


def crear_token(usuario_id: int) -> str:
    exp = datetime.now(timezone.utc) + timedelta(minutes=TOKEN_MINUTOS)
    return jwt.encode({"sub": str(usuario_id), "exp": exp}, SECRET_KEY, algorithm="HS256")


oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/token")


def get_sesion():
    with Session(motor) as sesion:
        yield sesion


SesionDep = Annotated[Session, Depends(get_sesion)]


def usuario_actual(token: Annotated[str, Depends(oauth2_scheme)], sesion: SesionDep) -> Usuario:
    error = HTTPException(status.HTTP_401_UNAUTHORIZED, "Credenciales no válidas",
                          headers={"WWW-Authenticate": "Bearer"})
    try:
        usuario_id = int(jwt.decode(token, SECRET_KEY, algorithms=["HS256"])["sub"])
    except (jwt.InvalidTokenError, KeyError, ValueError):
        raise error
    usuario = sesion.get(Usuario, usuario_id)
    if usuario is None:
        raise error
    return usuario


UsuarioDep = Annotated[Usuario, Depends(usuario_actual)]


def proyecto_del_usuario(proyecto_id: int, usuario: UsuarioDep, sesion: SesionDep) -> Proyecto:
    proyecto = sesion.get(Proyecto, proyecto_id)
    if proyecto is None or proyecto.propietario_id != usuario.id:
        raise HTTPException(404, "Proyecto no encontrado")
    return proyecto


ProyectoDep = Annotated[Proyecto, Depends(proyecto_del_usuario)]


def tarea_del_usuario(tarea_id: int, usuario: UsuarioDep, sesion: SesionDep) -> Tarea:
    tarea = sesion.get(Tarea, tarea_id)
    if tarea is None or tarea.proyecto.propietario_id != usuario.id:
        raise HTTPException(404, "Tarea no encontrada")
    return tarea


TareaDep = Annotated[Tarea, Depends(tarea_del_usuario)]


class Paginacion(BaseModel):
    pagina: int = Field(1, ge=1)
    tamano: int = Field(20, ge=1, le=100)


# ------------------------------------------------------------------ Errores, lifespan, aplicación
class ConflictoError(Exception):
    def __init__(self, mensaje: str):
        self.mensaje = mensaje


@asynccontextmanager
async def lifespan(app: FastAPI):
    SQLModel.metadata.create_all(motor)
    log.info("Base de datos lista")
    yield


app = FastAPI(title="API de tareas", version="1.0.0", lifespan=lifespan,
              description="Gestión de proyectos y tareas con autenticación JWT.")
app.add_middleware(CORSMiddleware, allow_origins=["http://localhost:5173"], allow_credentials=True,
                   allow_methods=["*"], allow_headers=["*"])


@app.middleware("http")
async def tiempo_respuesta(request: Request, call_next):
    inicio = time.perf_counter()
    respuesta = await call_next(request)
    respuesta.headers["X-Tiempo-Ms"] = f"{(time.perf_counter() - inicio) * 1000:.1f}"
    return respuesta


@app.exception_handler(ConflictoError)
async def manejar_conflicto(request: Request, exc: ConflictoError):
    return JSONResponse(status_code=409, content={"detail": exc.mensaje})


# ------------------------------------------------------------------ Autenticación
@app.post("/auth/registro", response_model=UsuarioPublico, status_code=201, tags=["auth"])
def registro(datos: UsuarioCrear, sesion: SesionDep):
    if sesion.exec(select(Usuario).where(Usuario.email == datos.email.lower())).first():
        raise ConflictoError("Ya existe un usuario con ese email")
    usuario = Usuario(email=datos.email.lower(), nombre=datos.nombre, clave_hash=hash_clave(datos.clave))
    sesion.add(usuario)
    sesion.commit()
    sesion.refresh(usuario)
    return usuario


@app.post("/auth/token", response_model=Token, tags=["auth"])
def login(form: Annotated[OAuth2PasswordRequestForm, Depends()], sesion: SesionDep):
    usuario = sesion.exec(select(Usuario).where(Usuario.email == form.username.lower())).first()
    if not usuario or not verificar_clave(form.password, usuario.clave_hash):
        raise HTTPException(401, "Email o contraseña incorrectos", headers={"WWW-Authenticate": "Bearer"})
    return Token(access_token=crear_token(usuario.id))


@app.get("/auth/yo", response_model=UsuarioPublico, tags=["auth"])
def yo(usuario: UsuarioDep):
    return usuario


# ------------------------------------------------------------------ Proyectos
def a_proyecto_publico(p: Proyecto) -> ProyectoPublico:
    return ProyectoPublico(id=p.id, nombre=p.nombre, descripcion=p.descripcion,
                           total_tareas=len(p.tareas), pendientes=sum(not t.hecha for t in p.tareas))


@app.get("/proyectos", response_model=list[ProyectoPublico], tags=["proyectos"])
def listar_proyectos(usuario: UsuarioDep, sesion: SesionDep):
    proyectos = sesion.exec(select(Proyecto).where(Proyecto.propietario_id == usuario.id)
                            .order_by(Proyecto.nombre)).all()
    return [a_proyecto_publico(p) for p in proyectos]


@app.post("/proyectos", response_model=ProyectoPublico, status_code=201, tags=["proyectos"])
def crear_proyecto(datos: ProyectoCrear, usuario: UsuarioDep, sesion: SesionDep):
    existe = sesion.exec(select(Proyecto).where(Proyecto.propietario_id == usuario.id,
                                                Proyecto.nombre == datos.nombre)).first()
    if existe:
        raise ConflictoError(f"Ya tiene un proyecto llamado «{datos.nombre}»")
    proyecto = Proyecto(**datos.model_dump(), propietario_id=usuario.id)
    sesion.add(proyecto)
    sesion.commit()
    sesion.refresh(proyecto)
    return a_proyecto_publico(proyecto)


@app.delete("/proyectos/{proyecto_id}", status_code=204, tags=["proyectos"])
def borrar_proyecto(proyecto: ProyectoDep, sesion: SesionDep):
    sesion.delete(proyecto)
    sesion.commit()


# ------------------------------------------------------------------ Tareas
def registrar_actividad(usuario_email: str, accion: str) -> None:
    """Tarea en segundo plano (p. ej., auditoría o notificación)."""
    log.info("Actividad de %s: %s", usuario_email, accion)


@app.post("/proyectos/{proyecto_id}/tareas", response_model=TareaPublica, status_code=201, tags=["tareas"])
def crear_tarea(datos: TareaCrear, proyecto: ProyectoDep, usuario: UsuarioDep, sesion: SesionDep,
                tareas_fondo: BackgroundTasks):
    tarea = Tarea.model_validate(datos, update={"proyecto_id": proyecto.id})
    sesion.add(tarea)
    sesion.commit()
    sesion.refresh(tarea)
    tareas_fondo.add_task(registrar_actividad, usuario.email, f"crea la tarea {tarea.id}")
    return tarea


@app.get("/proyectos/{proyecto_id}/tareas", response_model=PaginaTareas, tags=["tareas"])
def listar_tareas(
    proyecto: ProyectoDep,
    sesion: SesionDep,
    pag: Annotated[Paginacion, Query()],
    hecha: bool | None = None,
    prioridad: Prioridad | None = None,
    q: Annotated[str | None, Query(min_length=2)] = None,
    vencidas: bool = False,
):
    consulta = select(Tarea).where(Tarea.proyecto_id == proyecto.id)
    if hecha is not None:
        consulta = consulta.where(Tarea.hecha == hecha)
    if prioridad:
        consulta = consulta.where(Tarea.prioridad == prioridad)
    if q:
        consulta = consulta.where(Tarea.titulo.contains(q))
    if vencidas:
        consulta = consulta.where(Tarea.vence < date.today(), Tarea.hecha == False)  # noqa: E712
    total = sesion.exec(select(func.count()).select_from(consulta.subquery())).one()
    resultados = sesion.exec(consulta.order_by(Tarea.hecha, Tarea.vence, Tarea.id)
                             .offset((pag.pagina - 1) * pag.tamano).limit(pag.tamano)).all()
    return PaginaTareas(total=total, pagina=pag.pagina, tamano=pag.tamano, resultados=resultados)


@app.get("/tareas/{tarea_id}", response_model=TareaPublica, tags=["tareas"])
def obtener_tarea(tarea: TareaDep):
    return tarea


@app.patch("/tareas/{tarea_id}", response_model=TareaPublica, tags=["tareas"])
def actualizar_tarea(datos: TareaActualizar, tarea: TareaDep, sesion: SesionDep):
    cambios = datos.model_dump(exclude_unset=True)
    if not cambios:
        raise HTTPException(400, "No se ha indicado ningún cambio")
    tarea.sqlmodel_update(cambios)
    sesion.add(tarea)
    sesion.commit()
    sesion.refresh(tarea)
    return tarea


@app.delete("/tareas/{tarea_id}", status_code=204, tags=["tareas"])
def borrar_tarea(tarea: TareaDep, sesion: SesionDep):
    sesion.delete(tarea)
    sesion.commit()


@app.get("/estadisticas", tags=["informes"])
def estadisticas(usuario: UsuarioDep, sesion: SesionDep):
    filas = sesion.exec(
        select(Tarea.prioridad, Tarea.hecha, func.count())
        .join(Proyecto).where(Proyecto.propietario_id == usuario.id)
        .group_by(Tarea.prioridad, Tarea.hecha)).all()
    resumen = {p.value: {"pendientes": 0, "hechas": 0} for p in Prioridad}
    for prioridad, hecha, n in filas:
        resumen[prioridad.value if isinstance(prioridad, Prioridad) else prioridad]["hechas" if hecha else "pendientes"] = n
    total = sum(v["pendientes"] + v["hechas"] for v in resumen.values())
    hechas = sum(v["hechas"] for v in resumen.values())
    return {"total": total, "completado": round(hechas * 100 / total, 1) if total else 0.0,
            "por_prioridad": resumen}
```

Pruebas (`test_tareas_api.py`):

```python
import pytest
from fastapi.testclient import TestClient
from sqlmodel import Session, SQLModel, create_engine
from sqlmodel.pool import StaticPool

from tareas_api import app, get_sesion


@pytest.fixture
def cliente():
    motor = create_engine("sqlite://", connect_args={"check_same_thread": False}, poolclass=StaticPool)
    SQLModel.metadata.create_all(motor)

    def sesion_pruebas():
        with Session(motor) as s:
            yield s

    app.dependency_overrides[get_sesion] = sesion_pruebas
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()


def autenticar(cliente, email="ana@ejemplo.com") -> dict:
    cliente.post("/auth/registro", json={"email": email, "nombre": "Ana", "clave": "clave-segura"})
    token = cliente.post("/auth/token", data={"username": email, "password": "clave-segura"}).json()["access_token"]
    return {"Authorization": f"Bearer {token}"}


def test_registro_y_login(cliente):
    r = cliente.post("/auth/registro", json={"email": "ana@ejemplo.com", "nombre": "Ana", "clave": "clave-segura"})
    assert r.status_code == 201 and "clave" not in r.json() and "clave_hash" not in r.json()
    assert cliente.post("/auth/registro", json={"email": "ana@ejemplo.com", "nombre": "Ana",
                                               "clave": "clave-segura"}).status_code == 409
    assert cliente.post("/auth/token", data={"username": "ana@ejemplo.com", "password": "mal"}).status_code == 401


def test_requiere_token(cliente):
    assert cliente.get("/proyectos").status_code == 401


def test_flujo_completo(cliente):
    h = autenticar(cliente)
    proyecto = cliente.post("/proyectos", json={"nombre": "Web"}, headers=h).json()
    pid = proyecto["id"]
    for titulo, prioridad in [("Diseñar portada", "alta"), ("Revisar textos", "baja"), ("Subir a producción", "alta")]:
        assert cliente.post(f"/proyectos/{pid}/tareas", json={"titulo": titulo, "prioridad": prioridad},
                            headers=h).status_code == 201

    pagina = cliente.get(f"/proyectos/{pid}/tareas", params={"prioridad": "alta", "tamano": 1}, headers=h).json()
    assert pagina["total"] == 2 and len(pagina["resultados"]) == 1

    tid = pagina["resultados"][0]["id"]
    r = cliente.patch(f"/tareas/{tid}", json={"hecha": True}, headers=h)
    assert r.json()["hecha"] is True and r.json()["titulo"]                     # PATCH parcial

    stats = cliente.get("/estadisticas", headers=h).json()
    assert stats["total"] == 3 and stats["por_prioridad"]["alta"] == {"pendientes": 1, "hechas": 1}
    assert cliente.get("/proyectos", headers=h).json()[0]["pendientes"] == 2


def test_validacion_y_aislamiento(cliente):
    h1, h2 = autenticar(cliente), autenticar(cliente, "luis@ejemplo.com")
    pid = cliente.post("/proyectos", json={"nombre": "Privado"}, headers=h1).json()["id"]
    assert cliente.post(f"/proyectos/{pid}/tareas", json={"titulo": "x"}, headers=h1).status_code == 422
    assert cliente.get(f"/proyectos/{pid}/tareas", headers=h2).status_code == 404
```

```bash
fastapi dev tareas_api.py            # Documentación interactiva en /docs (botón Authorize)
pytest -q test_tareas_api.py
```

---

## 23. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `422 Unprocessable Entity` | Los datos no cumplen el modelo (tipo, longitud, campo ausente). | Lea `detail[].loc` y `msg` de la respuesta. |
| El cuerpo llega como parámetro de consulta | Se declaró un tipo simple en lugar de un modelo. | Use un modelo Pydantic o `Body()`. |
| `/items/destacados` devuelve 422 | La ruta variable `/items/{id}` está declarada antes. | Declare las rutas fijas primero. |
| `Form data requires "python-multipart"` | Falta la dependencia. | `pip install python-multipart` o `fastapi[standard]`. |
| La API se bloquea con pocas peticiones | Código bloqueante dentro de `async def`. | Use `def` o librerías asíncronas. |
| `ResponseValidationError` | La respuesta no cumple el `response_model`. | Ajuste el modelo o los datos devueltos (`from_attributes`). |
| Se filtra la contraseña en la respuesta | Se devolvió el objeto de BD sin modelo de salida. | Use `response_model` con un modelo público. |
| CORS bloqueado en el navegador | Origen no permitido. | Configure `CORSMiddleware` con los orígenes exactos. |
| `sqlite3.ProgrammingError: … same thread` | SQLite usado desde varios hilos. | `connect_args={"check_same_thread": False}`. |
| El *lifespan* no se ejecuta en pruebas | `TestClient` usado sin `with`. | `with TestClient(app) as cliente:`. |
| `401` en `/docs` tras autorizar | `tokenUrl` no coincide con la ruta real del *login*. | Revise `OAuth2PasswordBearer(tokenUrl=...)`. |

---

## 24. Buenas prácticas

- Use `Annotated` para parámetros y dependencias; cree alias (`SesionDep`, `UsuarioDep`) reutilizables.
- Separe modelos de entrada, salida y tabla (`Crear`, `Actualizar`, `Publico`).
- Defina siempre `response_model` (o el tipo de retorno) para filtrar y documentar las respuestas.
- Centralice la comprobación de permisos en dependencias (`proyecto_del_usuario`).
- Use `def` para código bloqueante y `async def` solo con librerías asíncronas.
- Organice el código en *routers* por dominio y la configuración con `pydantic-settings`.
- Devuelva códigos HTTP correctos (201 al crear, 204 al borrar, 404, 409, 422).
- Use Alembic para migraciones en lugar de `create_all()` en producción.
- Pruebe con `TestClient` y `dependency_overrides` sobre una base de datos aislada.
- Proteja o desactive `/docs` en APIs no públicas y restrinja CORS.

---

## 25. Referencia rápida

```python
app = FastAPI(title=, version=, lifespan=) · @app.get/post/put/patch/delete("/ruta/{id}", response_model=, status_code=, tags=)
def f(id: int, q: str | None = None, cuerpo: Modelo): ...
Annotated[int, Path(ge=1)] · Annotated[str | None, Query(min_length=2)] · Body() · Form() · File() · UploadFile · Header() · Cookie()
class M(BaseModel): campo: str = Field(min_length=2) · @field_validator · @model_validator · model_dump(exclude_unset=True)
raise HTTPException(404, "…") · @app.exception_handler(Exc) · RequestValidationError
Depends(f) · yield en dependencias · app.dependency_overrides[f] = g · APIRouter(prefix=, tags=) · app.include_router(r)
OAuth2PasswordBearer(tokenUrl=) · OAuth2PasswordRequestForm · APIKeyHeader · jwt.encode/decode
BackgroundTasks.add_task(f, *args) · @asynccontextmanager lifespan · @app.middleware("http") · CORSMiddleware
StreamingResponse · FileResponse · RedirectResponse · @app.websocket("/ws")
SQLModel(table=True) · Session(motor) · select(M).where() · sesion.exec().all() · sqlmodel_update()
TestClient(app) · fastapi dev main.py · fastapi run main.py · /docs · /redoc · /openapi.json
```

---

## 26. Ejercicios propuestos

1. Cree una API de libros con búsqueda por autor y año, paginación y documentación completa en `/docs`.
2. Añada al ejemplo completo etiquetas en las tareas (relación muchos a muchos) y un filtro por etiqueta.
3. Implemente *refresh tokens* y la invalidación de *tokens* al cerrar sesión.
4. Cree un *endpoint* `/exportar` que devuelva las tareas en CSV con `StreamingResponse`.
5. Añada un WebSocket que notifique en tiempo real a los clientes conectados cuando se crea o completa una tarea.

---

## 27. Referencias

- Documentación oficial (en español): https://fastapi.tiangolo.com/es/
- Tutorial: https://fastapi.tiangolo.com/tutorial/
- Pydantic: https://docs.pydantic.dev/
- SQLModel: https://sqlmodel.tiangolo.com/
- Starlette: https://www.starlette.io/
- Uvicorn: https://www.uvicorn.org/
