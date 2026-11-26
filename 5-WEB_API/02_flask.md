# Manual de uso: Flask

> **Categoría:** Desarrollo web · **Utilidad:** Framework web ligero
> **Tipo:** Librería de terceros (`pip install flask`) · **Versión de referencia:** 3.1

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Primera aplicación](#4-primera-aplicación)
5. [Rutas](#5-rutas)
6. [La petición (`request`)](#6-la-petición-request)
7. [Respuestas, redirecciones y errores](#7-respuestas-redirecciones-y-errores)
8. [Plantillas Jinja2](#8-plantillas-jinja2)
9. [Formularios y validación](#9-formularios-y-validación)
10. [Sesiones, cookies y mensajes *flash*](#10-sesiones-cookies-y-mensajes-flash)
11. [Archivos estáticos y subida de archivos](#11-archivos-estáticos-y-subida-de-archivos)
12. [Estructura de una aplicación: *factory* y *blueprints*](#12-estructura-de-una-aplicación-factory-y-blueprints)
13. [Configuración](#13-configuración)
14. [Base de datos con Flask-SQLAlchemy](#14-base-de-datos-con-flask-sqlalchemy)
15. [Autenticación con Flask-Login](#15-autenticación-con-flask-login)
16. [APIs JSON](#16-apis-json)
17. [Contextos, `g` y *hooks* de petición](#17-contextos-g-y-hooks-de-petición)
18. [Comandos de línea (`flask` CLI)](#18-comandos-de-línea-flask-cli)
19. [Pruebas](#19-pruebas)
20. [Despliegue](#20-despliegue)
21. [Ejemplo práctico completo](#21-ejemplo-práctico-completo)
22. [Errores frecuentes](#22-errores-frecuentes)
23. [Buenas prácticas](#23-buenas-prácticas)
24. [Referencia rápida](#24-referencia-rápida)
25. [Ejercicios propuestos](#25-ejercicios-propuestos)
26. [Referencias](#26-referencias)

---

## 1. Descripción

**Flask** es un *microframework* web para Python. Proporciona lo esencial (enrutado, petición y respuesta, plantillas con **Jinja2**, sesiones y un servidor de desarrollo basado en **Werkzeug**) y deja el resto (base de datos, formularios, autenticación) a **extensiones** que el desarrollador elige. Es ideal para aplicaciones pequeñas y medianas, prototipos, microservicios y APIs.

**Ventajas:**
- Muy sencillo de aprender: una aplicación funcional cabe en pocas líneas.
- Flexible: no impone estructura, ORM ni librería de formularios.
- Ecosistema maduro de extensiones (Flask-SQLAlchemy, Flask-Login, Flask-WTF, Flask-Migrate…).
- Documentación clara y gran comunidad.
- Buen soporte de pruebas con su cliente de test.

**Limitaciones:**
- Hay que elegir, integrar y mantener las piezas que Django ya trae.
- Sin panel de administración ni ORM propios.
- El soporte asíncrono es limitado (vistas `async` posibles, pero el modelo es WSGI).
- Para APIs con validación y documentación automáticas, FastAPI ofrece más de serie.

| Si necesita… | Opción |
|---|---|
| Aplicación web pequeña/mediana o microservicio con libertad de componentes | **Flask** |
| Aplicación completa con usuarios, admin y ORM integrados | Django |
| API moderna con validación y OpenAPI automáticas | FastAPI |
| Flask con soporte asíncrono nativo | Quart (API compatible) |

---

## 2. Instalación e importación

```bash
python -m venv .venv
source .venv/bin/activate              # Windows: .venv\Scripts\activate
pip install flask
pip install flask-sqlalchemy flask-login flask-wtf    # Extensiones habituales (opcionales)
```

```python
from flask import Flask, request, render_template, redirect, url_for, flash, jsonify, abort, session, g
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Aplicación (`Flask`) | Objeto central que registra rutas, configuración y extensiones. |
| Ruta | Asociación entre una URL y una función (vista). |
| Vista | Función que devuelve una respuesta (texto, HTML, JSON, redirección). |
| `request` | Objeto global (por contexto) con los datos de la petición actual. |
| Plantilla | HTML con sintaxis Jinja2 (`{{ }}`, `{% %}`). |
| *Blueprint* | Conjunto de rutas y recursos reutilizable que se registra en la aplicación. |
| *Application factory* | Función `create_app()` que construye y configura la aplicación. |
| Contexto | Ámbito de la aplicación y de la petición (`current_app`, `request`, `g`, `session`). |
| Extensión | Paquete que integra una funcionalidad (BD, login, formularios…). |

---

## 4. Primera aplicación

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def inicio():
    return "<h1>Hola, Flask</h1>"

@app.route("/saludo/<nombre>")
def saludo(nombre):
    return f"Hola, {nombre}"
```

```bash
flask --app app run --debug           # http://127.0.0.1:5000 (recarga automática y depurador)
```

> El modo `--debug` muestra un depurador interactivo en el navegador: **nunca** lo active en producción.

---

## 5. Rutas

### 5.1 Variables y convertidores

```python
@app.route("/productos/<int:id>")
def producto(id): ...

@app.route("/archivos/<path:ruta>")
def archivo(ruta): ...
```

Convertidores: `string` (defecto), `int`, `float`, `path`, `uuid`.

### 5.2 Métodos HTTP

```python
@app.route("/contacto", methods=["GET", "POST"])
def contacto():
    if request.method == "POST":
        ...
    return render_template("contacto.html")

# Atajos (Flask 2+)
@app.get("/tareas")
def listar(): ...

@app.post("/tareas")
def crear(): ...

@app.delete("/tareas/<int:id>")
def borrar(id): ...
```

### 5.3 Construir URLs

```python
from flask import url_for

url_for("producto", id=5)                    # '/productos/5'
url_for("static", filename="css/app.css")    # '/static/css/app.css'
url_for("listar", pagina=2)                  # '/tareas?pagina=2' (parámetros extra → query string)
url_for("producto", id=5, _external=True)    # URL absoluta
```

> Use siempre `url_for` en lugar de escribir rutas a mano: si cambia la URL, no hay que tocar el resto del código.

---

## 6. La petición (`request`)

```python
from flask import request

request.method                          # "GET", "POST"…
request.args.get("q", "")               # Parámetros de la URL (?q=…)
request.args.get("pagina", 1, type=int)
request.form["email"]                   # Datos de formulario (KeyError → 400 si falta)
request.form.get("email")
request.form.getlist("etiquetas")       # Valores múltiples
request.get_json()                      # Cuerpo JSON (415 si no es JSON; silent=True devuelve None)
request.files["archivo"]                # Archivos subidos
request.headers.get("Authorization")
request.cookies.get("tema")
request.remote_addr
request.path, request.url, request.host
```

---

## 7. Respuestas, redirecciones y errores

### 7.1 Tipos de respuesta

```python
from flask import make_response, jsonify, redirect, url_for, send_file, abort

@app.get("/ping")
def ping():
    return "pong"                                   # 200, text/html

@app.get("/estado")
def estado():
    return {"estado": "ok", "version": "1.0"}       # Diccionario → JSON automáticamente

@app.get("/lista")
def lista():
    return jsonify([1, 2, 3])                       # Listas → jsonify

@app.post("/recursos")
def crear():
    return {"id": 10}, 201, {"Location": "/recursos/10"}   # (cuerpo, estado, cabeceras)

@app.get("/viejo")
def viejo():
    return redirect(url_for("inicio"), code=301)

@app.get("/descargar")
def descargar():
    return send_file("informes/ventas.pdf", as_attachment=True, download_name="ventas.pdf")

@app.get("/cookie")
def cookie():
    resp = make_response("Preferencia guardada")
    resp.set_cookie("tema", "oscuro", max_age=60 * 60 * 24 * 30, httponly=True, samesite="Lax")
    return resp
```

### 7.2 Errores

```python
@app.get("/tareas/<int:id>")
def tarea(id):
    t = TAREAS.get(id)
    if t is None:
        abort(404)                                  # También: abort(403), abort(400, "Mensaje")
    return t

@app.errorhandler(404)
def no_encontrado(error):
    if request.path.startswith("/api/"):
        return {"error": "No encontrado"}, 404
    return render_template("404.html"), 404

@app.errorhandler(500)
def error_interno(error):
    return render_template("500.html"), 500
```

---

## 8. Plantillas Jinja2

Flask busca las plantillas en la carpeta `templates/`.

```python
@app.get("/tareas")
def tareas():
    return render_template("tareas.html", tareas=obtener_tareas(), titulo="Mis tareas")
```

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>{% block titulo %}Mi app{% endblock %}</title>
  <link rel="stylesheet" href="{{ url_for('static', filename='css/app.css') }}">
</head>
<body>
  {% with mensajes = get_flashed_messages(with_categories=true) %}
    {% for categoria, mensaje in mensajes %}<div class="aviso {{ categoria }}">{{ mensaje }}</div>{% endfor %}
  {% endwith %}
  {% block contenido %}{% endblock %}
</body>
</html>
```

```jinja
{# templates/tareas.html #}
{% extends "base.html" %}
{% block titulo %}{{ titulo }}{% endblock %}
{% block contenido %}
<h1>{{ titulo }}</h1>
<ul>
  {% for t in tareas %}
    <li class="{{ 'hecha' if t.hecha }}">
      {{ loop.index }}. {{ t.titulo|capitalize }}
      <small>{{ t.creada.strftime('%d/%m/%Y') }}</small>
    </li>
  {% else %}
    <li>No hay tareas.</li>
  {% endfor %}
</ul>
<p>Total: {{ tareas|length }}</p>
{% endblock %}
```

| Elemento | Ejemplo |
|---|---|
| Variable | `{{ usuario.nombre }}`, `{{ datos['clave'] }}` |
| Filtro | `{{ texto\|upper }}`, `{{ lista\|join(', ') }}`, `{{ valor\|default('—') }}`, `{{ n\|round(2) }}` |
| Condicional | `{% if x %}…{% elif y %}…{% else %}…{% endif %}` |
| Bucle | `{% for x in lista %}…{% else %}…{% endfor %}` (`loop.index`, `loop.first`, `loop.last`) |
| Herencia | `{% extends %}`, `{% block %}`, `{{ super() }}` |
| Inclusión | `{% include "parcial.html" %}` |
| Macro | `{% macro campo(nombre) %}…{% endmacro %}` + `{% from "macros.html" import campo %}` |
| Expresión en línea | `{{ 'activo' if seleccionado else '' }}` |

Filtros y variables globales propios:

```python
@app.template_filter("euros")
def euros(valor):
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")

@app.context_processor
def variables_globales():
    return {"anio_actual": datetime.now().year, "nombre_app": "Gestor"}
```

> Jinja2 escapa el HTML automáticamente en archivos `.html`. Use `|safe` solo con contenido de confianza.

---

## 9. Formularios y validación

### 9.1 Validación manual

```python
@app.route("/registro", methods=["GET", "POST"])
def registro():
    errores, datos = {}, {}
    if request.method == "POST":
        datos = {"nombre": request.form.get("nombre", "").strip(),
                 "email": request.form.get("email", "").strip().lower()}
        if len(datos["nombre"]) < 2:
            errores["nombre"] = "Indique su nombre."
        if "@" not in datos["email"]:
            errores["email"] = "Email no válido."
        if not errores:
            guardar(datos)
            flash("Registro completado", "ok")
            return redirect(url_for("inicio"))          # Patrón POST/Redirect/GET
    return render_template("registro.html", errores=errores, datos=datos)
```

### 9.2 Flask-WTF (con protección CSRF)

```python
from flask_wtf import FlaskForm
from wtforms import StringField, EmailField, PasswordField, SelectField, TextAreaField, BooleanField
from wtforms.validators import DataRequired, Email, Length, EqualTo, ValidationError

class RegistroForm(FlaskForm):
    nombre = StringField("Nombre", validators=[DataRequired(), Length(min=2, max=80)])
    email = EmailField("Email", validators=[DataRequired(), Email()])     # Email() requiere email-validator
    clave = PasswordField("Contraseña", validators=[DataRequired(), Length(min=8)])
    repetir = PasswordField("Repetir", validators=[EqualTo("clave", "Las contraseñas no coinciden")])
    acepta = BooleanField("Acepto las condiciones", validators=[DataRequired()])

    def validate_email(self, campo):
        if Usuario.query.filter_by(email=campo.data.lower()).first():
            raise ValidationError("Ese email ya está registrado.")

@app.route("/registro", methods=["GET", "POST"])
def registro():
    form = RegistroForm()
    if form.validate_on_submit():                # POST + válido + token CSRF correcto
        crear_usuario(form.data)
        return redirect(url_for("inicio"))
    return render_template("registro.html", form=form)
```

```jinja
<form method="post" novalidate>
  {{ form.hidden_tag() }}                         {# Incluye el token CSRF #}
  {% for campo in [form.nombre, form.email, form.clave, form.repetir] %}
    <p>{{ campo.label }} {{ campo() }}
      {% for e in campo.errors %}<span class="error">{{ e }}</span>{% endfor %}</p>
  {% endfor %}
  <p>{{ form.acepta() }} {{ form.acepta.label }}</p>
  <button>Registrarse</button>
</form>
```

> Flask-WTF requiere `SECRET_KEY`. Para proteger formularios HTML sin WTForms, use `CSRFProtect(app)` y `<input type="hidden" name="csrf_token" value="{{ csrf_token() }}">`.

---

## 10. Sesiones, cookies y mensajes *flash*

```python
app.config["SECRET_KEY"] = "cambie-esto"          # Obligatorio para session y flash

@app.post("/carrito/<int:id>")
def anadir(id):
    carrito = session.setdefault("carrito", [])
    carrito.append(id)
    session.modified = True                       # Necesario al modificar objetos mutables
    flash("Producto añadido al carrito", "ok")
    return redirect(url_for("carrito"))

session.pop("carrito", None)
session.clear()
session.permanent = True                          # Dura PERMANENT_SESSION_LIFETIME (31 días por defecto)
```

> La sesión por defecto se guarda **firmada (no cifrada)** en una cookie: el usuario puede leer su contenido, aunque no modificarlo. No guarde datos sensibles. Para sesiones en servidor: **Flask-Session** (Redis, base de datos).

---

## 11. Archivos estáticos y subida de archivos

Los archivos de `static/` se sirven en `/static/…`:

```jinja
<img src="{{ url_for('static', filename='img/logo.png') }}">
```

Subida segura:

```python
from pathlib import Path
from werkzeug.utils import secure_filename

app.config["MAX_CONTENT_LENGTH"] = 5 * 1024 * 1024        # 5 MB (413 si se supera)
CARPETA = Path(app.instance_path) / "subidas"
PERMITIDAS = {".png", ".jpg", ".jpeg", ".pdf"}

@app.post("/subir")
def subir():
    archivo = request.files.get("archivo")
    if not archivo or archivo.filename == "":
        abort(400, "No se ha enviado ningún archivo")
    nombre = secure_filename(archivo.filename)              # Elimina rutas y caracteres peligrosos
    if Path(nombre).suffix.lower() not in PERMITIDAS:
        abort(400, "Tipo de archivo no permitido")
    CARPETA.mkdir(parents=True, exist_ok=True)
    archivo.save(CARPETA / nombre)
    return {"guardado": nombre}, 201

@app.get("/subidas/<path:nombre>")
def ver_subida(nombre):
    return send_from_directory(CARPETA, nombre)             # Evita salir de la carpeta
```

---

## 12. Estructura de una aplicación: *factory* y *blueprints*

### 12.1 Estructura recomendada

```
mi_app/
├── pyproject.toml
├── instance/                 # Configuración local y datos (no se versiona)
├── tests/
└── src/mi_app/
    ├── __init__.py           # create_app()
    ├── extensiones.py        # db, login_manager…
    ├── modelos.py
    ├── web/                  # Blueprint de páginas HTML
    │   ├── __init__.py
    │   └── rutas.py
    ├── api/                  # Blueprint de la API JSON
    │   └── __init__.py
    ├── templates/
    └── static/
```

### 12.2 *Blueprints*

```python
# src/mi_app/api/__init__.py
from flask import Blueprint, jsonify

bp = Blueprint("api", __name__, url_prefix="/api")

@bp.get("/tareas")
def listar():
    return jsonify([...])
```

```python
url_for("api.listar")         # Nombre de endpoint: "<blueprint>.<función>"
```

### 12.3 *Application factory*

```python
# src/mi_app/__init__.py
from flask import Flask

def create_app(config: dict | None = None) -> Flask:
    app = Flask(__name__, instance_relative_config=True)
    app.config.from_mapping(SECRET_KEY="dev", SQLALCHEMY_DATABASE_URI="sqlite:///app.db")
    app.config.from_prefixed_env()                 # Variables FLASK_* del entorno
    if config:
        app.config.update(config)

    from .extensiones import db, login_manager
    db.init_app(app)
    login_manager.init_app(app)

    from . import api, web
    app.register_blueprint(web.bp)
    app.register_blueprint(api.bp)
    return app
```

```bash
flask --app mi_app run --debug          # Flask localiza create_app() automáticamente
```

> La *factory* evita importaciones circulares, permite crear varias instancias (por ejemplo, para pruebas con otra configuración) y es el patrón recomendado.

---

## 13. Configuración

```python
app.config["DEBUG"]                              # No lo active por código en producción
app.config.from_object("config.Produccion")      # Desde una clase
app.config.from_pyfile("config.py", silent=True) # Desde instance/config.py
app.config.from_file("config.toml", load=tomllib.load, text=False)
app.config.from_prefixed_env()                   # FLASK_SECRET_KEY → SECRET_KEY
```

```python
class Config:
    SECRET_KEY = os.environ.get("SECRET_KEY", "dev")
    SQLALCHEMY_DATABASE_URI = os.environ.get("DATABASE_URL", "sqlite:///app.db")

class Pruebas(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"
    WTF_CSRF_ENABLED = False
```

Claves importantes: `SECRET_KEY`, `TESTING`, `MAX_CONTENT_LENGTH`, `SESSION_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY`, `SESSION_COOKIE_SAMESITE`, `PERMANENT_SESSION_LIFETIME`, `JSON_SORT_KEYS` (vía `app.json.sort_keys`), `SERVER_NAME`.

---

## 14. Base de datos con Flask-SQLAlchemy

### 14.1 Modelos (SQLAlchemy 2.x)

```python
# extensiones.py
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass

db = SQLAlchemy(model_class=Base)
```

```python
# modelos.py
from datetime import datetime
from sqlalchemy import ForeignKey, String
from sqlalchemy.orm import Mapped, mapped_column, relationship
from .extensiones import db

class Proyecto(db.Model):
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(100), unique=True)
    tareas: Mapped[list["Tarea"]] = relationship(back_populates="proyecto", cascade="all, delete-orphan")

class Tarea(db.Model):
    id: Mapped[int] = mapped_column(primary_key=True)
    titulo: Mapped[str] = mapped_column(String(200))
    hecha: Mapped[bool] = mapped_column(default=False)
    creada: Mapped[datetime] = mapped_column(default=datetime.now)
    proyecto_id: Mapped[int | None] = mapped_column(ForeignKey("proyecto.id"))
    proyecto: Mapped["Proyecto | None"] = relationship(back_populates="tareas")
```

### 14.2 Consultas y operaciones

```python
from sqlalchemy import select

with app.app_context():
    db.create_all()                                    # Crea las tablas (sin migraciones)

    t = Tarea(titulo="Preparar informe")
    db.session.add(t)
    db.session.commit()

    tarea = db.session.get(Tarea, 1)                   # Por clave primaria (None si no existe)
    tarea = db.get_or_404(Tarea, 1)                    # 404 automático en una vista
    pendientes = db.session.scalars(select(Tarea).where(Tarea.hecha == False).order_by(Tarea.creada)).all()
    una = db.first_or_404(select(Tarea).filter_by(titulo="Preparar informe"))
    pagina = db.paginate(select(Tarea).order_by(Tarea.id), page=2, per_page=20)   # .items, .pages, .has_next

    tarea.hecha = True
    db.session.commit()

    db.session.delete(tarea)
    db.session.commit()
```

> La API antigua `Tarea.query.filter_by(...)` sigue funcionando, pero se considera heredada; la recomendada es `db.session.execute(select(...))`.

### 14.3 Migraciones con Flask-Migrate (Alembic)

```bash
pip install flask-migrate
flask --app mi_app db init
flask --app mi_app db migrate -m "crear tablas"
flask --app mi_app db upgrade
```

```python
from flask_migrate import Migrate
migrate = Migrate(app, db)
```

---

## 15. Autenticación con Flask-Login

```python
from flask_login import LoginManager, UserMixin, login_user, logout_user, login_required, current_user
from werkzeug.security import generate_password_hash, check_password_hash

login_manager = LoginManager()
login_manager.login_view = "web.login"                    # Redirección si no hay sesión
login_manager.login_message = "Inicie sesión para continuar."

class Usuario(UserMixin, db.Model):
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(120), unique=True)
    clave_hash: Mapped[str] = mapped_column(String(255))

    def fijar_clave(self, clave: str) -> None:
        self.clave_hash = generate_password_hash(clave)

    def comprobar_clave(self, clave: str) -> bool:
        return check_password_hash(self.clave_hash, clave)

@login_manager.user_loader
def cargar_usuario(id_: str):
    return db.session.get(Usuario, int(id_))

@bp.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        u = db.session.scalar(select(Usuario).filter_by(email=request.form["email"].lower()))
        if u and u.comprobar_clave(request.form["clave"]):
            login_user(u, remember=bool(request.form.get("recordar")))
            return redirect(request.args.get("next") or url_for("web.inicio"))
        flash("Credenciales incorrectas", "error")
    return render_template("login.html")

@bp.post("/logout")
@login_required
def logout():
    logout_user()
    return redirect(url_for("web.inicio"))

@bp.get("/perfil")
@login_required
def perfil():
    return render_template("perfil.html", usuario=current_user)
```

> Valide el parámetro `next` antes de redirigir (que sea una ruta interna) para evitar redirecciones abiertas.

---

## 16. APIs JSON

```python
from flask import Blueprint, request, abort

bp = Blueprint("api", __name__, url_prefix="/api")

@bp.get("/tareas")
def listar():
    pagina = request.args.get("pagina", 1, type=int)
    resultado = db.paginate(select(Tarea).order_by(Tarea.id), page=pagina, per_page=20, error_out=False)
    return {"tareas": [t.a_dict() for t in resultado.items], "pagina": pagina, "total": resultado.total}

@bp.post("/tareas")
def crear():
    datos = request.get_json(silent=True) or {}
    titulo = str(datos.get("titulo", "")).strip()
    if not titulo:
        return {"error": "El campo 'titulo' es obligatorio"}, 422
    t = Tarea(titulo=titulo)
    db.session.add(t)
    db.session.commit()
    return t.a_dict(), 201

@bp.errorhandler(404)
def api_404(e):
    return {"error": "Recurso no encontrado"}, 404
```

Para validación declarativa y documentación OpenAPI en Flask existen extensiones como **flask-smorest**, **APIFlask** o **Flask-Pydantic**. Para habilitar CORS: **Flask-Cors**.

---

## 17. Contextos, `g` y *hooks* de petición

| Objeto | Ámbito | Uso |
|---|---|---|
| `current_app` | Aplicación activa | Configuración y extensiones desde cualquier módulo. |
| `request` | Petición | Datos de la petición actual. |
| `session` | Petición (persistente en cookie) | Datos del usuario entre peticiones. |
| `g` | Petición | Almacén temporal (conexiones, usuario cargado). |

```python
import time
from flask import g

@app.before_request
def iniciar_cronometro():
    g.inicio = time.perf_counter()

@app.after_request
def cabecera_tiempo(respuesta):
    respuesta.headers["X-Tiempo"] = f"{(time.perf_counter() - g.inicio) * 1000:.1f}ms"
    return respuesta

@app.teardown_appcontext
def cerrar_conexion(excepcion):
    conexion = g.pop("bd", None)
    if conexion is not None:
        conexion.close()
```

Fuera de una petición (scripts, consola), cree el contexto manualmente:

```python
with app.app_context():
    db.create_all()

with app.test_request_context("/tareas?q=x"):
    print(request.args["q"], url_for("api.listar"))
```

---

## 18. Comandos de línea (`flask` CLI)

```bash
flask --app mi_app run --debug --port 8000
flask --app mi_app routes                 # Lista todas las rutas
flask --app mi_app shell                  # Consola con la app cargada
```

Comandos propios (usan Click):

```python
import click

@app.cli.command("init-db")
@click.option("--demo", is_flag=True, help="Carga datos de ejemplo")
def init_db(demo):
    """Crea las tablas de la base de datos."""
    db.drop_all()
    db.create_all()
    if demo:
        cargar_demo()
    click.echo("Base de datos inicializada.")
```

```bash
flask --app mi_app init-db --demo
```

> Para no escribir `--app` cada vez: variable `FLASK_APP=mi_app` o un archivo `.flaskenv` (requiere `python-dotenv`).

---

## 19. Pruebas

```python
# tests/conftest.py
import pytest
from mi_app import create_app
from mi_app.extensiones import db

@pytest.fixture
def app():
    app = create_app({"TESTING": True, "SQLALCHEMY_DATABASE_URI": "sqlite:///:memory:", "WTF_CSRF_ENABLED": False})
    with app.app_context():
        db.create_all()
        yield app

@pytest.fixture
def cliente(app):
    return app.test_client()

# tests/test_api.py
def test_crear_tarea(cliente):
    r = cliente.post("/api/tareas", json={"titulo": "Probar Flask"})
    assert r.status_code == 201
    assert r.get_json()["titulo"] == "Probar Flask"

def test_validacion(cliente):
    assert cliente.post("/api/tareas", json={}).status_code == 422

def test_pagina(cliente):
    r = cliente.get("/")
    assert "Tareas" in r.get_data(as_text=True)

def test_formulario_con_redireccion(cliente):
    r = cliente.post("/login", data={"email": "x@x.com", "clave": "mal"}, follow_redirects=True)
    assert r.status_code == 200
```

| Herramienta | Uso |
|---|---|
| `app.test_client()` | Simula peticiones: `get`, `post(json=…, data=…)`, `follow_redirects`. |
| `app.test_cli_runner()` | Ejecuta comandos `flask` en pruebas. |
| `with cliente.session_transaction() as s:` | Manipular la sesión. |

---

## 20. Despliegue

```bash
pip install gunicorn
gunicorn "mi_app:create_app()" --workers 4 --bind 0.0.0.0:8000      # Linux/macOS

pip install waitress
waitress-serve --port=8000 --call mi_app:create_app                   # Multiplataforma (Windows)
```

Lista de comprobación:
- `SECRET_KEY` larga y aleatoria, desde el entorno (`python -c "import secrets; print(secrets.token_hex(32))"`).
- Sin `--debug`.
- Detrás de un proxy (Nginx) con HTTPS: `ProxyFix` de Werkzeug para cabeceras `X-Forwarded-*`.
- `SESSION_COOKIE_SECURE=True`, `SESSION_COOKIE_HTTPONLY=True`, `SESSION_COOKIE_SAMESITE="Lax"`.
- Base de datos de producción (PostgreSQL) y migraciones con Flask-Migrate.
- Registro de errores (logging, Sentry).

```python
from werkzeug.middleware.proxy_fix import ProxyFix
app.wsgi_app = ProxyFix(app.wsgi_app, x_for=1, x_proto=1)
```

---

## 21. Ejemplo práctico completo

**Gestor de marcadores (enlaces guardados)** con *application factory*, *blueprints* web y API, Flask-SQLAlchemy, Flask-Login, registro e inicio de sesión, CRUD con validación, etiquetas, búsqueda y paginación, mensajes *flash*, páginas de error, comando CLI de datos de prueba, API JSON y pruebas con pytest. Toda la aplicación cabe en un archivo para facilitar su estudio; en un proyecto real se dividiría en módulos como en el apartado 12.

```python
"""
marcadores.py — Gestor de marcadores con Flask.
Requiere: pip install flask flask-sqlalchemy flask-login
Uso:
    flask --app marcadores init-db --demo
    flask --app marcadores run --debug
"""
import os
from datetime import datetime
from urllib.parse import urlparse

import click
from flask import Blueprint, Flask, abort, flash, redirect, render_template, request, url_for
from flask_login import (LoginManager, UserMixin, current_user, login_required, login_user, logout_user)
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import Column, ForeignKey, String, Table, func, or_, select
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from werkzeug.security import check_password_hash, generate_password_hash


# ------------------------------------------------------------------ Extensiones y modelos
class Base(DeclarativeBase):
    pass


db = SQLAlchemy(model_class=Base)
login_manager = LoginManager()
login_manager.login_view = "web.login"
login_manager.login_message = "Inicie sesión para continuar."
login_manager.login_message_category = "error"

marcador_etiqueta = Table(
    "marcador_etiqueta", Base.metadata,
    Column("marcador_id", ForeignKey("marcador.id", ondelete="CASCADE"), primary_key=True),
    Column("etiqueta_id", ForeignKey("etiqueta.id", ondelete="CASCADE"), primary_key=True),
)


class Usuario(UserMixin, db.Model):
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(120), unique=True)
    clave_hash: Mapped[str] = mapped_column(String(255))
    marcadores: Mapped[list["Marcador"]] = relationship(back_populates="usuario", cascade="all, delete-orphan")

    def fijar_clave(self, clave: str) -> None:
        self.clave_hash = generate_password_hash(clave)

    def comprobar_clave(self, clave: str) -> bool:
        return check_password_hash(self.clave_hash, clave)


class Etiqueta(db.Model):
    id: Mapped[int] = mapped_column(primary_key=True)
    nombre: Mapped[str] = mapped_column(String(40), unique=True)


class Marcador(db.Model):
    id: Mapped[int] = mapped_column(primary_key=True)
    url: Mapped[str] = mapped_column(String(500))
    titulo: Mapped[str] = mapped_column(String(200))
    notas: Mapped[str] = mapped_column(String(1000), default="")
    favorito: Mapped[bool] = mapped_column(default=False)
    creado: Mapped[datetime] = mapped_column(default=datetime.now)
    usuario_id: Mapped[int] = mapped_column(ForeignKey("usuario.id"))
    usuario: Mapped[Usuario] = relationship(back_populates="marcadores")
    etiquetas: Mapped[list[Etiqueta]] = relationship(secondary=marcador_etiqueta, lazy="selectin")

    @property
    def dominio(self) -> str:
        return urlparse(self.url).netloc

    def a_dict(self) -> dict:
        return {"id": self.id, "url": self.url, "titulo": self.titulo, "notas": self.notas,
                "favorito": self.favorito, "creado": self.creado.isoformat(timespec="seconds"),
                "etiquetas": [e.nombre for e in self.etiquetas]}


@login_manager.user_loader
def cargar_usuario(id_: str):
    return db.session.get(Usuario, int(id_))


# ------------------------------------------------------------------ Servicios
def validar_marcador(datos: dict) -> dict[str, str]:
    errores = {}
    url = datos.get("url", "").strip()
    partes = urlparse(url)
    if partes.scheme not in ("http", "https") or not partes.netloc:
        errores["url"] = "Indique una URL completa que empiece por http:// o https://"
    if not datos.get("titulo", "").strip():
        errores["titulo"] = "El título es obligatorio."
    if len(datos.get("notas", "")) > 1000:
        errores["notas"] = "Las notas no pueden superar 1000 caracteres."
    return errores


def obtener_etiquetas(texto: str) -> list[Etiqueta]:
    nombres = sorted({n.strip().lower() for n in texto.split(",") if n.strip()})
    etiquetas = []
    with db.session.no_autoflush:          # Evita guardar objetos a medio construir durante la consulta
        for nombre in nombres:
            etiqueta = db.session.scalar(select(Etiqueta).filter_by(nombre=nombre))
            if etiqueta is None:
                etiqueta = Etiqueta(nombre=nombre)
                db.session.add(etiqueta)
            etiquetas.append(etiqueta)
    return etiquetas


def aplicar_datos(m: Marcador, datos: dict) -> None:
    m.url = datos["url"].strip()
    m.titulo = datos["titulo"].strip()
    m.notas = datos.get("notas", "").strip()
    m.favorito = bool(datos.get("favorito"))
    m.etiquetas = obtener_etiquetas(datos.get("etiquetas", ""))


def consulta_marcadores(usuario: Usuario, texto: str = "", etiqueta: str = "", favoritos: bool = False):
    consulta = select(Marcador).where(Marcador.usuario_id == usuario.id)
    if texto:
        patron = f"%{texto}%"
        consulta = consulta.where(or_(Marcador.titulo.ilike(patron), Marcador.url.ilike(patron),
                                      Marcador.notas.ilike(patron)))
    if etiqueta:
        consulta = consulta.where(Marcador.etiquetas.any(Etiqueta.nombre == etiqueta))
    if favoritos:
        consulta = consulta.where(Marcador.favorito.is_(True))
    return consulta.order_by(Marcador.favorito.desc(), Marcador.creado.desc())


def marcador_del_usuario(id_: int) -> Marcador:
    m = db.get_or_404(Marcador, id_)
    if m.usuario_id != current_user.id:
        abort(404)                        # No revelar que existe
    return m


def ruta_interna(destino: str | None) -> bool:
    return bool(destino) and urlparse(destino).netloc == "" and destino.startswith("/")


# ------------------------------------------------------------------ Plantillas
BASE = """<!DOCTYPE html><html lang="es"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1"><title>{% block titulo %}Marcadores{% endblock %}</title>
<style>
 body{font-family:system-ui,sans-serif;max-width:860px;margin:auto;padding:1rem;color:#1f2933}
 nav{display:flex;gap:1rem;align-items:center;border-bottom:2px solid #1F4E78;padding-bottom:.5rem}
 nav .der{margin-left:auto} .aviso{padding:.5rem;margin:.5rem 0;background:#E8F4EA} .aviso.error{background:#FBE9E9}
 .marcador{border-bottom:1px solid #ddd;padding:.6rem 0} .dominio,.fecha{color:#6b7280;font-size:.85rem}
 .etiqueta{background:#e9eef3;border-radius:4px;padding:0 .35rem;margin-right:.25rem;font-size:.8rem;text-decoration:none}
 .error{color:#C00000;font-size:.85rem} label{display:block;margin-top:.6rem} input[type=text],input[type=url],input[type=email],input[type=password],textarea{width:100%}
 form.inline{display:inline}
</style></head><body>
<nav><a href="{{ url_for('web.inicio') }}"><strong>🔖 Marcadores</strong></a>
{% if current_user.is_authenticated %}<a href="{{ url_for('web.nuevo') }}">Nuevo</a>{% endif %}
<span class="der">{% if current_user.is_authenticated %}{{ current_user.email }}
<form class="inline" method="post" action="{{ url_for('web.logout') }}"><button>Salir</button></form>
{% else %}<a href="{{ url_for('web.login') }}">Entrar</a> · <a href="{{ url_for('web.registro') }}">Registrarse</a>{% endif %}</span></nav>
{% for cat, msg in get_flashed_messages(with_categories=true) %}<div class="aviso {{ cat }}">{{ msg }}</div>{% endfor %}
{% block contenido %}{% endblock %}</body></html>"""

LISTA = """{% extends "base" %}{% block contenido %}
<h1>Mis marcadores</h1>
<form method="get"><input type="text" name="q" value="{{ q }}" placeholder="Buscar…" style="width:50%">
<label style="display:inline"><input type="checkbox" name="favoritos" value="1" {{ 'checked' if favoritos }}> Solo favoritos</label>
{% if etiqueta %}<input type="hidden" name="etiqueta" value="{{ etiqueta }}"> Etiqueta: <strong>{{ etiqueta }}</strong>
<a href="{{ url_for('web.inicio', q=q) }}">✕</a>{% endif %} <button>Buscar</button></form>
<p class="fecha">{{ pagina.total }} marcador(es)</p>
{% for m in pagina.items %}<div class="marcador">
 {{ '⭐' if m.favorito }} <a href="{{ m.url }}" target="_blank" rel="noopener"><strong>{{ m.titulo }}</strong></a>
 <span class="dominio">{{ m.dominio }}</span><br>
 {% if m.notas %}<small>{{ m.notas }}</small><br>{% endif %}
 {% for e in m.etiquetas %}<a class="etiqueta" href="{{ url_for('web.inicio', etiqueta=e.nombre) }}">#{{ e.nombre }}</a>{% endfor %}
 <span class="fecha">{{ m.creado.strftime('%d/%m/%Y') }}</span> ·
 <a href="{{ url_for('web.editar', id=m.id) }}">editar</a>
 <form class="inline" method="post" action="{{ url_for('web.borrar', id=m.id) }}" onsubmit="return confirm('¿Eliminar?')"><button>borrar</button></form>
</div>{% else %}<p>No hay marcadores. <a href="{{ url_for('web.nuevo') }}">Añada el primero</a>.</p>{% endfor %}
{% if pagina.pages > 1 %}<p>{% if pagina.has_prev %}<a href="{{ url_for('web.inicio', page=pagina.prev_num, q=q, etiqueta=etiqueta) }}">« Anterior</a>{% endif %}
 Página {{ pagina.page }} de {{ pagina.pages }}
 {% if pagina.has_next %}<a href="{{ url_for('web.inicio', page=pagina.next_num, q=q, etiqueta=etiqueta) }}">Siguiente »</a>{% endif %}</p>{% endif %}
{% endblock %}"""

FORMULARIO = """{% extends "base" %}{% block contenido %}
<h1>{{ 'Editar' if marcador else 'Nuevo' }} marcador</h1>
<form method="post" novalidate>
{% for campo, etiqueta, tipo in [('url','URL','url'),('titulo','Título','text'),('etiquetas','Etiquetas (separadas por comas)','text')] %}
 <label>{{ etiqueta }}<input type="{{ tipo }}" name="{{ campo }}" value="{{ datos.get(campo, '') }}"></label>
 {% if errores[campo] %}<div class="error">{{ errores[campo] }}</div>{% endif %}
{% endfor %}
<label>Notas<textarea name="notas" rows="3">{{ datos.get('notas', '') }}</textarea></label>
{% if errores.notas %}<div class="error">{{ errores.notas }}</div>{% endif %}
<label><input type="checkbox" name="favorito" value="1" {{ 'checked' if datos.get('favorito') }}> Favorito</label>
<p><button>Guardar</button> <a href="{{ url_for('web.inicio') }}">Cancelar</a></p></form>{% endblock %}"""

ACCESO = """{% extends "base" %}{% block contenido %}<h1>{{ titulo }}</h1>
<form method="post"><label>Email<input type="email" name="email" value="{{ email }}"></label>
<label>Contraseña<input type="password" name="clave"></label>
{% if registro %}<label>Repetir contraseña<input type="password" name="clave2"></label>{% endif %}
<p><button>{{ titulo }}</button></p></form>{% endblock %}"""

ERROR = """{% extends "base" %}{% block contenido %}<h1>{{ codigo }}</h1><p>{{ mensaje }}</p>
<p><a href="{{ url_for('web.inicio') }}">Volver al inicio</a></p>{% endblock %}"""

PLANTILLAS = {"base": BASE, "lista": LISTA, "formulario": FORMULARIO, "acceso": ACCESO, "error": ERROR}


def pagina_html(nombre: str, **contexto) -> str:
    """Las plantillas se cargan desde PLANTILLAS (DictLoader); en un proyecto real estarían en templates/."""
    return render_template(nombre, **contexto)


# ------------------------------------------------------------------ Blueprint web
web = Blueprint("web", __name__)


@web.get("/")
@login_required
def inicio():
    q = request.args.get("q", "").strip()
    etiqueta = request.args.get("etiqueta", "").strip()
    favoritos = bool(request.args.get("favoritos"))
    pagina = db.paginate(consulta_marcadores(current_user, q, etiqueta, favoritos),
                         page=request.args.get("page", 1, type=int), per_page=10, error_out=False)
    return pagina_html("lista", pagina=pagina, q=q, etiqueta=etiqueta, favoritos=favoritos)


@web.route("/nuevo", methods=["GET", "POST"])
@login_required
def nuevo():
    datos, errores = {}, {}
    if request.method == "POST":
        datos = request.form.to_dict()
        errores = validar_marcador(datos)
        if not errores:
            m = Marcador(usuario=current_user)
            aplicar_datos(m, datos)
            db.session.add(m)
            db.session.commit()
            flash(f"Marcador «{m.titulo}» guardado.", "ok")
            return redirect(url_for("web.inicio"))
    return pagina_html("formulario", datos=datos, errores=errores, marcador=None)


@web.route("/<int:id>/editar", methods=["GET", "POST"])
@login_required
def editar(id):
    m = marcador_del_usuario(id)
    datos = {**m.a_dict(), "etiquetas": ", ".join(e.nombre for e in m.etiquetas)}
    errores = {}
    if request.method == "POST":
        datos = request.form.to_dict()
        errores = validar_marcador(datos)
        if not errores:
            aplicar_datos(m, datos)
            db.session.commit()
            flash("Cambios guardados.", "ok")
            return redirect(url_for("web.inicio"))
    return pagina_html("formulario", datos=datos, errores=errores, marcador=m)


@web.post("/<int:id>/borrar")
@login_required
def borrar(id):
    m = marcador_del_usuario(id)
    db.session.delete(m)
    db.session.commit()
    flash("Marcador eliminado.", "ok")
    return redirect(url_for("web.inicio"))


@web.route("/registro", methods=["GET", "POST"])
def registro():
    email = ""
    if request.method == "POST":
        email = request.form.get("email", "").strip().lower()
        clave, clave2 = request.form.get("clave", ""), request.form.get("clave2", "")
        if "@" not in email:
            flash("Email no válido.", "error")
        elif len(clave) < 8:
            flash("La contraseña debe tener al menos 8 caracteres.", "error")
        elif clave != clave2:
            flash("Las contraseñas no coinciden.", "error")
        elif db.session.scalar(select(Usuario).filter_by(email=email)):
            flash("Ese email ya está registrado.", "error")
        else:
            u = Usuario(email=email)
            u.fijar_clave(clave)
            db.session.add(u)
            db.session.commit()
            login_user(u)
            flash("Cuenta creada. ¡Bienvenido!", "ok")
            return redirect(url_for("web.inicio"))
    return pagina_html("acceso", titulo="Crear cuenta", registro=True, email=email)


@web.route("/login", methods=["GET", "POST"])
def login():
    email = ""
    if request.method == "POST":
        email = request.form.get("email", "").strip().lower()
        u = db.session.scalar(select(Usuario).filter_by(email=email))
        if u and u.comprobar_clave(request.form.get("clave", "")):
            login_user(u)
            destino = request.args.get("next")
            return redirect(destino if ruta_interna(destino) else url_for("web.inicio"))
        flash("Email o contraseña incorrectos.", "error")
    return pagina_html("acceso", titulo="Iniciar sesión", registro=False, email=email)


@web.post("/logout")
@login_required
def logout():
    logout_user()
    flash("Sesión cerrada.", "ok")
    return redirect(url_for("web.login"))


# ------------------------------------------------------------------ Blueprint API
api = Blueprint("api", __name__, url_prefix="/api")


@api.before_request
def exigir_sesion():
    if not current_user.is_authenticated:
        return {"error": "Autenticación requerida"}, 401


@api.get("/marcadores")
def api_listar():
    consulta = consulta_marcadores(current_user, request.args.get("q", ""), request.args.get("etiqueta", ""))
    return {"marcadores": [m.a_dict() for m in db.session.scalars(consulta)]}


@api.post("/marcadores")
def api_crear():
    datos = request.get_json(silent=True) or {}
    if isinstance(datos.get("etiquetas"), list):
        datos["etiquetas"] = ",".join(datos["etiquetas"])
    if errores := validar_marcador(datos):
        return {"errores": errores}, 422
    m = Marcador(usuario=current_user)
    aplicar_datos(m, datos)
    db.session.add(m)
    db.session.commit()
    return m.a_dict(), 201, {"Location": url_for("api.api_detalle", id=m.id)}


@api.get("/marcadores/<int:id>")
def api_detalle(id):
    return marcador_del_usuario(id).a_dict()


@api.delete("/marcadores/<int:id>")
def api_borrar(id):
    db.session.delete(marcador_del_usuario(id))
    db.session.commit()
    return "", 204


@api.get("/etiquetas")
def api_etiquetas():
    filas = db.session.execute(
        select(Etiqueta.nombre, func.count(Marcador.id))
        .join(Marcador.etiquetas).where(Marcador.usuario_id == current_user.id)
        .group_by(Etiqueta.nombre).order_by(func.count(Marcador.id).desc()))
    return {"etiquetas": [{"nombre": n, "marcadores": c} for n, c in filas]}


# ------------------------------------------------------------------ Factory
def create_app(config: dict | None = None) -> Flask:
    app = Flask(__name__, instance_relative_config=True)
    app.config.from_mapping(
        SECRET_KEY=os.environ.get("SECRET_KEY", "dev-cambie-esto"),
        SQLALCHEMY_DATABASE_URI=os.environ.get("DATABASE_URL", "sqlite:///marcadores.db"),
        SESSION_COOKIE_HTTPONLY=True,
        SESSION_COOKIE_SAMESITE="Lax",
    )
    if config:
        app.config.update(config)

    db.init_app(app)
    login_manager.init_app(app)
    app.register_blueprint(web)
    app.register_blueprint(api)

    from jinja2 import DictLoader
    app.jinja_loader = DictLoader(PLANTILLAS)

    @app.errorhandler(404)
    def no_encontrado(e):
        if request.path.startswith("/api/"):
            return {"error": "No encontrado"}, 404
        return pagina_html("error", codigo=404, mensaje="La página solicitada no existe."), 404

    @app.cli.command("init-db")
    @click.option("--demo", is_flag=True, help="Crea un usuario y marcadores de ejemplo.")
    def init_db(demo):
        """Crea (o recrea) las tablas."""
        db.drop_all()
        db.create_all()
        if demo:
            u = Usuario(email="demo@ejemplo.com")
            u.fijar_clave("demo1234")
            db.session.add(u)
            ejemplos = [
                ("https://docs.python.org/es/3/", "Documentación de Python", "python, docs", True),
                ("https://flask.palletsprojects.com/", "Documentación de Flask", "python, flask, web", True),
                ("https://developer.mozilla.org/es/", "MDN Web Docs", "web, html, css", False),
                ("https://www.sqlalchemy.org/", "SQLAlchemy", "python, bd", False),
            ]
            for url, titulo, etiquetas, fav in ejemplos:
                m = Marcador(usuario=u)
                aplicar_datos(m, {"url": url, "titulo": titulo, "etiquetas": etiquetas, "favorito": fav})
                db.session.add(m)
            db.session.commit()
            click.echo("Usuario de prueba: demo@ejemplo.com / demo1234")
        click.echo("Base de datos inicializada.")

    with app.app_context():
        db.create_all()
    return app
```

Pruebas (`test_marcadores.py`):

```python
import pytest
from marcadores import create_app, db


@pytest.fixture
def cliente():
    app = create_app({"TESTING": True, "SQLALCHEMY_DATABASE_URI": "sqlite:///:memory:"})
    with app.test_client() as c:
        yield c


def registrar(cliente, email="ana@ejemplo.com"):
    return cliente.post("/registro", data={"email": email, "clave": "clave-larga", "clave2": "clave-larga"},
                        follow_redirects=True)


def test_requiere_login(cliente):
    r = cliente.get("/")
    assert r.status_code == 302 and "/login" in r.headers["Location"]
    assert cliente.get("/api/marcadores").status_code == 401


def test_registro_y_crear_marcador(cliente):
    assert "Bienvenido" in registrar(cliente).get_data(as_text=True)
    r = cliente.post("/nuevo", data={"url": "https://flask.palletsprojects.com", "titulo": "Flask",
                                     "etiquetas": "Python, web"}, follow_redirects=True)
    html = r.get_data(as_text=True)
    assert "Flask" in html and "#python" in html


def test_validacion_formulario(cliente):
    registrar(cliente)
    r = cliente.post("/nuevo", data={"url": "flask", "titulo": ""})
    html = r.get_data(as_text=True)
    assert "URL completa" in html and "obligatorio" in html


def test_api_crud_y_aislamiento(cliente):
    registrar(cliente)
    r = cliente.post("/api/marcadores", json={"url": "https://example.com", "titulo": "Ejemplo",
                                               "etiquetas": ["demo"]})
    assert r.status_code == 201
    id_ = r.get_json()["id"]
    assert cliente.get("/api/marcadores?etiqueta=demo").get_json()["marcadores"][0]["id"] == id_
    assert cliente.post("/api/marcadores", json={"url": "x"}).status_code == 422

    cliente.post("/logout")
    registrar(cliente, "otro@ejemplo.com")
    assert cliente.get(f"/api/marcadores/{id_}").status_code == 404      # No puede ver marcadores ajenos
```

```bash
flask --app marcadores init-db --demo
flask --app marcadores run --debug       # demo@ejemplo.com / demo1234
pytest -q test_marcadores.py
```

> Las plantillas se incluyen como texto (`DictLoader`) para que el ejemplo sea un único archivo. En un proyecto real, colóquelas en `templates/` y use `render_template()`.

---

## 22. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `RuntimeError: Working outside of application context` | Uso de `current_app`, `db` o `url_for` fuera de una petición. | `with app.app_context():`. |
| `RuntimeError: The session is unavailable because no secret key was set` | Falta `SECRET_KEY`. | Configúrela (desde el entorno en producción). |
| `400 Bad Request: KeyError` | `request.form["campo"]` sin ese campo. | Use `request.form.get("campo")`. |
| `415 Unsupported Media Type` | `request.get_json()` sin cabecera JSON. | Envíe `Content-Type: application/json` o use `silent=True`. |
| `werkzeug.routing.BuildError` | Nombre de endpoint incorrecto en `url_for` (falta el prefijo del *blueprint*). | `url_for("blueprint.funcion")`. |
| `AssertionError: View function mapping is overwriting…` | Dos vistas con el mismo nombre de función. | Renombre o use `endpoint=`. |
| `TemplateNotFound` | Plantilla fuera de `templates/` o nombre incorrecto. | Revise la carpeta y la ruta relativa. |
| Importación circular | Módulos que se importan mutuamente con `app`. | *Application factory* y extensiones en un módulo propio. |
| `400 The CSRF token is missing` | Formulario sin `csrf_token` con Flask-WTF. | `{{ form.hidden_tag() }}` o desactive en pruebas. |
| `413 Request Entity Too Large` | Archivo mayor que `MAX_CONTENT_LENGTH`. | Ajuste el límite o informe al usuario. |
| Cambios en la sesión no se guardan | Se modificó un objeto mutable dentro de `session`. | `session.modified = True`. |

---

## 23. Buenas prácticas

- Use el patrón *application factory* y organice las rutas en *blueprints*.
- Mantenga la lógica de negocio fuera de las vistas (módulos de servicios o modelos).
- Use `url_for` para todas las URLs y `redirect` tras un POST correcto (patrón PRG).
- Configure con variables de entorno; nunca suba `SECRET_KEY` ni credenciales al repositorio.
- Proteja los formularios con CSRF (Flask-WTF o `CSRFProtect`) y las vistas privadas con `login_required`.
- Valide siempre la entrada (formularios, JSON, archivos) y use `secure_filename`.
- Use Flask-Migrate para los cambios de esquema en lugar de `create_all()` en producción.
- Escriba pruebas con `test_client` y una base de datos aislada.
- Despliegue con Gunicorn/Waitress detrás de un proxy HTTPS, nunca con el servidor de desarrollo.

---

## 24. Referencia rápida

```python
app = Flask(__name__) · @app.route("/x/<int:id>", methods=["GET", "POST"]) · @app.get / .post / .put / .delete
request.args.get("q") · request.form.get("c") · request.get_json(silent=True) · request.files["f"]
return "texto" | {"json": 1} | (cuerpo, 201, cabeceras) · jsonify(lista) · redirect(url_for("vista", id=1))
render_template("x.html", **ctx) · abort(404) · @app.errorhandler(404) · send_file / send_from_directory
session["k"] = v · flash("msg", "ok") · get_flashed_messages(with_categories=true) · make_response().set_cookie()
bp = Blueprint("api", __name__, url_prefix="/api") · app.register_blueprint(bp) · url_for("api.vista")
def create_app(config=None): app = Flask(__name__); ...; return app
@app.before_request · @app.after_request · @app.teardown_appcontext · g · current_app · app.app_context()
db = SQLAlchemy(model_class=Base) · db.session.add/commit/get/scalars(select(M)) · db.paginate · db.get_or_404
login_user(u) · logout_user() · @login_required · current_user · @login_manager.user_loader
@app.cli.command("x") · flask --app paquete run --debug · flask routes · app.test_client()
```

---

## 25. Ejercicios propuestos

1. Cree una API de notas con Flask (CRUD en memoria) y pruebas con `test_client`.
2. Convierta el ejemplo completo en un paquete con módulos separados (`modelos.py`, `web/`, `api/`, `templates/`) y añada Flask-Migrate.
3. Añada a los marcadores la importación de un archivo HTML exportado desde el navegador (subida de archivo y análisis con BeautifulSoup).
4. Implemente autenticación por *token* en la API (cabecera `Authorization: Bearer …`) además de la sesión.
5. Despliegue la aplicación con Gunicorn detrás de Nginx (o Waitress en Windows) con `SECRET_KEY` y base de datos desde variables de entorno.

---

## 26. Referencias

- Documentación oficial: https://flask.palletsprojects.com/
- Tutorial oficial (Flaskr): https://flask.palletsprojects.com/tutorial/
- Flask-SQLAlchemy: https://flask-sqlalchemy.readthedocs.io/
- Flask-Login: https://flask-login.readthedocs.io/
- Flask-WTF: https://flask-wtf.readthedocs.io/
- Jinja2: https://jinja.palletsprojects.com/
