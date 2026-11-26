# Manual de uso: Django

> **Categoría:** Desarrollo web · **Utilidad:** Framework web completo («con baterías incluidas»)
> **Tipo:** Librería de terceros (`pip install django`) · **Versión de referencia:** 5.2 LTS – 6.x

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación y creación del proyecto](#2-instalación-y-creación-del-proyecto)
3. [Conceptos clave](#3-conceptos-clave)
4. [Estructura de un proyecto](#4-estructura-de-un-proyecto)
5. [Configuración (`settings.py`)](#5-configuración-settingspy)
6. [Modelos y migraciones](#6-modelos-y-migraciones)
7. [El ORM: consultas](#7-el-orm-consultas)
8. [El panel de administración](#8-el-panel-de-administración)
9. [URLs y vistas](#9-urls-y-vistas)
10. [Vistas basadas en clases](#10-vistas-basadas-en-clases)
11. [Plantillas](#11-plantillas)
12. [Formularios](#12-formularios)
13. [Archivos estáticos y subidos](#13-archivos-estáticos-y-subidos)
14. [Autenticación y permisos](#14-autenticación-y-permisos)
15. [Middleware, señales, mensajes y caché](#15-middleware-señales-mensajes-y-caché)
16. [Comandos de gestión](#16-comandos-de-gestión)
17. [APIs JSON y Django REST Framework](#17-apis-json-y-django-rest-framework)
18. [Tareas en segundo plano](#18-tareas-en-segundo-plano)
19. [Pruebas](#19-pruebas)
20. [Seguridad y despliegue](#20-seguridad-y-despliegue)
21. [Ejemplo práctico completo](#21-ejemplo-práctico-completo)
22. [Errores frecuentes](#22-errores-frecuentes)
23. [Buenas prácticas](#23-buenas-prácticas)
24. [Referencia rápida](#24-referencia-rápida)
25. [Ejercicios propuestos](#25-ejercicios-propuestos)
26. [Referencias](#26-referencias)

---

## 1. Descripción

**Django** es un *framework* web de alto nivel para Python que incluye todo lo necesario para construir aplicaciones web completas: ORM y migraciones de base de datos, sistema de URLs, vistas, plantillas, formularios con validación, autenticación de usuarios, panel de administración automático, internacionalización, caché, protección frente a ataques comunes y un servidor de desarrollo. Su lema es «el framework para perfeccionistas con plazos».

**Ventajas:**
- «Baterías incluidas»: muy pocas decisiones de infraestructura.
- Panel de administración generado automáticamente a partir de los modelos.
- ORM potente con migraciones automáticas.
- Seguridad por defecto: CSRF, XSS, inyección SQL, *clickjacking*, contraseñas con hash.
- Documentación excelente y ecosistema enorme (Django REST Framework, Celery, Wagtail, django-allauth…).
- Escala bien: lo usan Instagram, Mozilla, Disqus o Pinterest.

**Limitaciones:**
- Más pesado y estructurado que Flask o FastAPI para proyectos muy pequeños.
- Curva de aprendizaje inicial (proyecto, aplicaciones, configuración).
- El soporte asíncrono existe, pero el ORM es mayoritariamente síncrono.
- Para APIs puras se suele añadir Django REST Framework o Django Ninja.

| Si necesita… | Opción |
|---|---|
| Aplicación web completa con usuarios, administración y base de datos | **Django** |
| Aplicación pequeña o microservicio con libertad total de componentes | Flask |
| API moderna, asíncrona, con documentación OpenAPI automática | FastAPI |
| API REST sobre un proyecto Django | Django REST Framework / Django Ninja |
| Gestor de contenidos (CMS) | Wagtail (basado en Django) |

---

## 2. Instalación y creación del proyecto

```bash
python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install django

django-admin startproject config .   # Proyecto en la carpeta actual (paquete 'config')
python manage.py startapp tienda     # Una aplicación del proyecto
python manage.py migrate             # Crea las tablas iniciales (usuarios, sesiones…)
python manage.py createsuperuser     # Usuario administrador
python manage.py runserver           # http://127.0.0.1:8000  ·  /admin
```

> Django 6.x requiere Python 3.12 o superior. Para proyectos que deban mantenerse años sin grandes cambios, use la última versión **LTS** (5.2).

---

## 3. Conceptos clave

Django sigue el patrón **MTV** (Modelo – Plantilla – Vista):

| Concepto | Descripción |
|---|---|
| Proyecto | Configuración global: ajustes, URLs raíz, punto de entrada WSGI/ASGI. |
| Aplicación (*app*) | Módulo reutilizable con una funcionalidad (modelos, vistas, plantillas). |
| Modelo | Clase Python que representa una tabla de la base de datos. |
| Migración | Archivo que describe cambios en el esquema de la base de datos. |
| ORM / QuerySet | API para consultar la base de datos sin escribir SQL. |
| Vista | Función o clase que recibe una petición (`HttpRequest`) y devuelve una respuesta. |
| URLconf | Lista de rutas que asocia URLs con vistas. |
| Plantilla | HTML con el lenguaje de plantillas de Django (`{{ variable }}`, `{% etiqueta %}`). |
| Formulario | Clase que genera, valida y limpia datos de entrada. |
| Middleware | Capa que procesa todas las peticiones y respuestas. |
| Admin | Interfaz de administración generada a partir de los modelos. |

Flujo de una petición:

```
Navegador → URLconf → Vista → (Modelo/ORM ↔ BD) → Plantilla → HttpResponse → Navegador
                      ↑ middleware antes y después ↑
```

---

## 4. Estructura de un proyecto

```
mi_proyecto/
├── manage.py                 # Utilidad de línea de comandos
├── config/                   # Paquete del proyecto
│   ├── settings.py           # Configuración
│   ├── urls.py               # URLs raíz
│   ├── asgi.py / wsgi.py     # Puntos de entrada para servidores
├── tienda/                   # Aplicación
│   ├── migrations/           # Migraciones
│   ├── admin.py              # Registro en el admin
│   ├── apps.py               # Configuración de la app
│   ├── models.py             # Modelos
│   ├── views.py              # Vistas
│   ├── urls.py               # URLs de la app (se crea a mano)
│   ├── forms.py              # Formularios (se crea a mano)
│   ├── templates/tienda/     # Plantillas de la app
│   ├── static/tienda/        # CSS, JS, imágenes de la app
│   └── tests.py              # Pruebas
└── templates/                # Plantillas globales (base.html)
```

> Las plantillas y los estáticos de cada app se guardan en una subcarpeta con el **nombre de la app** (`templates/tienda/`) para evitar colisiones entre aplicaciones.

---

## 5. Configuración (`settings.py`)

```python
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY", "solo-para-desarrollo")
DEBUG = os.environ.get("DJANGO_DEBUG", "1") == "1"
ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS", "localhost,127.0.0.1").split(",")

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "tienda",                                  # Su aplicación
]

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
# PostgreSQL:
# "ENGINE": "django.db.backends.postgresql", "NAME": "app", "USER": "app",
# "PASSWORD": os.environ["DB_PASSWORD"], "HOST": "localhost", "PORT": "5432"

TEMPLATES = [{
    "BACKEND": "django.template.backends.django.DjangoTemplates",
    "DIRS": [BASE_DIR / "templates"],
    "APP_DIRS": True,
    "OPTIONS": {"context_processors": [
        "django.template.context_processors.request",
        "django.contrib.auth.context_processors.auth",
        "django.contrib.messages.context_processors.messages",
    ]},
}]

LANGUAGE_CODE = "es-es"
TIME_ZONE = "Europe/Madrid"
USE_I18N = True
USE_TZ = True

STATIC_URL = "static/"
STATIC_ROOT = BASE_DIR / "staticfiles"         # Destino de collectstatic
MEDIA_URL = "media/"
MEDIA_ROOT = BASE_DIR / "media"                # Archivos subidos por usuarios

DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"
LOGIN_URL = "login"
LOGIN_REDIRECT_URL = "/"
```

> Nunca guarde `SECRET_KEY` ni contraseñas en el repositorio: léalas de variables de entorno (o use `django-environ`).

---

## 6. Modelos y migraciones

### 6.1 Definir modelos

```python
# tienda/models.py
from django.conf import settings
from django.core.validators import MinValueValidator
from django.db import models
from django.urls import reverse


class Categoria(models.Model):
    nombre = models.CharField(max_length=80, unique=True)
    slug = models.SlugField(unique=True)

    class Meta:
        ordering = ["nombre"]
        verbose_name_plural = "categorías"

    def __str__(self) -> str:
        return self.nombre


class Producto(models.Model):
    class Estado(models.TextChoices):
        BORRADOR = "borrador", "Borrador"
        PUBLICADO = "publicado", "Publicado"
        RETIRADO = "retirado", "Retirado"

    nombre = models.CharField("nombre", max_length=200)
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])
    stock = models.PositiveIntegerField(default=0)
    estado = models.CharField(max_length=10, choices=Estado, default=Estado.BORRADOR)
    categoria = models.ForeignKey(Categoria, on_delete=models.PROTECT, related_name="productos")
    etiquetas = models.ManyToManyField("Etiqueta", blank=True)
    imagen = models.ImageField(upload_to="productos/%Y/", blank=True)     # Requiere Pillow
    creado_por = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL, null=True)
    creado = models.DateTimeField(auto_now_add=True)
    actualizado = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["-creado"]
        indexes = [models.Index(fields=["estado", "categoria"])]
        constraints = [models.CheckConstraint(condition=models.Q(precio__gte=0), name="precio_no_negativo")]

    def __str__(self) -> str:
        return self.nombre

    def get_absolute_url(self) -> str:
        return reverse("tienda:detalle", args=[self.pk])

    @property
    def disponible(self) -> bool:
        return self.estado == self.Estado.PUBLICADO and self.stock > 0


class Etiqueta(models.Model):
    nombre = models.CharField(max_length=40, unique=True)

    def __str__(self) -> str:
        return self.nombre
```

### 6.2 Tipos de campo frecuentes

| Campo | Uso |
|---|---|
| `CharField(max_length=)` / `TextField` | Texto corto / largo. |
| `IntegerField`, `PositiveIntegerField`, `BigIntegerField` | Enteros. |
| `DecimalField(max_digits, decimal_places)` | Importes (nunca `FloatField` para dinero). |
| `BooleanField` | Verdadero/falso. |
| `DateField`, `DateTimeField(auto_now_add, auto_now)`, `DurationField` | Fechas. |
| `EmailField`, `URLField`, `SlugField`, `UUIDField` | Texto con validación. |
| `JSONField` | Datos JSON. |
| `FileField`, `ImageField` | Archivos subidos. |
| `ForeignKey(Modelo, on_delete=)` | Relación muchos-a-uno. |
| `ManyToManyField(Modelo)` | Relación muchos-a-muchos. |
| `OneToOneField(Modelo, on_delete=)` | Relación uno-a-uno (perfiles). |
| `GeneratedField(expression=, output_field=, db_persist=)` | Columna calculada por la BD. |

Opciones comunes: `null`, `blank`, `default`, `unique`, `choices`, `db_index`, `verbose_name`, `help_text`, `validators`, `editable`.

| `on_delete` | Efecto al borrar el objeto referenciado |
|---|---|
| `CASCADE` | Borra también los objetos relacionados. |
| `PROTECT` | Impide el borrado (`ProtectedError`). |
| `SET_NULL` | Pone `NULL` (requiere `null=True`). |
| `SET_DEFAULT` / `SET(valor)` | Valor por defecto / indicado. |
| `RESTRICT` | Similar a `PROTECT`, con matices en cascadas. |

### 6.3 Migraciones

```bash
python manage.py makemigrations          # Genera migraciones a partir de los cambios en models.py
python manage.py migrate                 # Aplica migraciones pendientes
python manage.py showmigrations          # Estado de las migraciones
python manage.py sqlmigrate tienda 0001  # SQL que ejecutará una migración
python manage.py migrate tienda 0003     # Volver a una migración anterior
```

> Las migraciones se **versionan** con el código. No edite la base de datos a mano en producción.

---

## 7. El ORM: consultas

### 7.1 Crear, leer, actualizar, borrar

```python
from tienda.models import Producto, Categoria

cat = Categoria.objects.create(nombre="Periféricos", slug="perifericos")
p = Producto.objects.create(nombre="Teclado", precio="59.90", stock=10, categoria=cat)

Producto.objects.all()
Producto.objects.get(pk=1)                               # DoesNotExist / MultipleObjectsReturned
Producto.objects.filter(estado="publicado", stock__gt=0)
Producto.objects.exclude(categoria__slug="monitores")
Producto.objects.order_by("-precio", "nombre")[:10]
Producto.objects.filter(nombre__icontains="tecl").first()

p.precio = "54.90"
p.save(update_fields=["precio"])
Producto.objects.filter(stock=0).update(estado="retirado")   # Actualización masiva
p.delete()

obj, creado = Categoria.objects.get_or_create(slug="redes", defaults={"nombre": "Redes"})
obj, creado = Producto.objects.update_or_create(nombre="Hub", defaults={"precio": 30, "categoria": cat})
Producto.objects.bulk_create([Producto(nombre=f"P{i}", precio=1, categoria=cat) for i in range(100)])
```

### 7.2 Búsquedas por campo (*lookups*)

| Lookup | Ejemplo |
|---|---|
| `exact`, `iexact` | `nombre__iexact="teclado"` |
| `contains`, `icontains` | `nombre__icontains="usb"` |
| `startswith`, `endswith` | `codigo__startswith="TEC"` |
| `gt`, `gte`, `lt`, `lte` | `precio__lte=100` |
| `in` | `estado__in=["borrador", "retirado"]` |
| `range` | `creado__range=(inicio, fin)` |
| `isnull` | `imagen__isnull=True` |
| `year`, `month`, `date` | `creado__year=2026` |
| Relaciones | `categoria__nombre="Redes"`, `etiquetas__nombre="oferta"` |

### 7.3 Expresiones `Q` y `F`

```python
from django.db.models import Q, F

Producto.objects.filter(Q(nombre__icontains="usb") | Q(descripcion__icontains="usb"))
Producto.objects.filter(~Q(estado="retirado"))
Producto.objects.filter(stock__lt=F("stock_minimo"))
Producto.objects.update(precio=F("precio") * 1.05)          # Subida del 5 % en la BD
```

### 7.4 Agregaciones y anotaciones

```python
from django.db.models import Count, Sum, Avg, Max

Producto.objects.aggregate(total=Sum("stock"), media=Avg("precio"))
Categoria.objects.annotate(num=Count("productos")).filter(num__gt=5).order_by("-num")
Producto.objects.values("categoria__nombre").annotate(valor=Sum(F("precio") * F("stock")))
```

### 7.5 Rendimiento: evitar el problema N+1

```python
# ✖ Una consulta por producto para obtener su categoría
for p in Producto.objects.all():
    print(p.categoria.nombre)

# ✔ JOIN en una sola consulta (ForeignKey / OneToOne)
for p in Producto.objects.select_related("categoria"):
    print(p.categoria.nombre)

# ✔ Una consulta adicional para todas las relaciones ManyToMany / inversas
Producto.objects.prefetch_related("etiquetas")

Producto.objects.only("id", "nombre")          # Cargar solo algunas columnas
Producto.objects.filter(...).exists()          # Más eficiente que len() o count() para comprobar
Producto.objects.values_list("id", flat=True)
```

### 7.6 Transacciones y SQL directo

```python
from django.db import transaction

with transaction.atomic():
    pedido = Pedido.objects.create(cliente=c)
    Producto.objects.select_for_update().filter(pk=p.pk).update(stock=F("stock") - 1)

Producto.objects.raw("SELECT * FROM tienda_producto WHERE precio > %s", [100])
```

### 7.7 Gestores (*managers*) personalizados

```python
class ProductoQuerySet(models.QuerySet):
    def publicados(self):
        return self.filter(estado=Producto.Estado.PUBLICADO)

    def con_stock(self):
        return self.filter(stock__gt=0)

class Producto(models.Model):
    ...
    objects = ProductoQuerySet.as_manager()

Producto.objects.publicados().con_stock()
```

### 7.8 Consola interactiva

```bash
python manage.py shell          # Importa automáticamente los modelos (Django 5.2+)
python manage.py dbshell        # Cliente de la base de datos
```

---

## 8. El panel de administración

```python
# tienda/admin.py
from django.contrib import admin
from .models import Categoria, Producto, Etiqueta


@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ["nombre", "slug"]
    prepopulated_fields = {"slug": ["nombre"]}


class EtiquetaInline(admin.TabularInline):
    model = Producto.etiquetas.through
    extra = 1


@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ["nombre", "categoria", "precio", "stock", "estado", "disponible"]
    list_filter = ["estado", "categoria", "creado"]
    search_fields = ["nombre", "descripcion"]
    list_editable = ["precio", "stock"]
    list_per_page = 50
    date_hierarchy = "creado"
    autocomplete_fields = ["categoria"]
    readonly_fields = ["creado", "actualizado"]
    inlines = [EtiquetaInline]
    actions = ["publicar"]
    fieldsets = [
        (None, {"fields": ["nombre", "categoria", "descripcion"]}),
        ("Comercial", {"fields": ["precio", "stock", "estado"]}),
        ("Auditoría", {"fields": ["creado", "actualizado"], "classes": ["collapse"]}),
    ]

    @admin.display(boolean=True, description="¿Disponible?")
    def disponible(self, obj):
        return obj.disponible

    @admin.action(description="Publicar los productos seleccionados")
    def publicar(self, request, queryset):
        n = queryset.update(estado=Producto.Estado.PUBLICADO)
        self.message_user(request, f"{n} productos publicados.")

    def save_model(self, request, obj, form, change):
        if not change:
            obj.creado_por = request.user
        super().save_model(request, obj, form, change)


admin.site.site_header = "Administración de la tienda"
admin.site.register(Etiqueta)
```

> `autocomplete_fields` exige que el `ModelAdmin` del modelo relacionado defina `search_fields`.

---

## 9. URLs y vistas

### 9.1 URLs

```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("tienda.urls")),
]
```

```python
# tienda/urls.py
from django.urls import path
from . import views

app_name = "tienda"                          # Espacio de nombres: "tienda:detalle"
urlpatterns = [
    path("", views.lista, name="lista"),
    path("productos/<int:pk>/", views.detalle, name="detalle"),
    path("categorias/<slug:slug>/", views.por_categoria, name="categoria"),
    path("buscar/", views.buscar, name="buscar"),
]
```

Convertidores: `<int:x>`, `<str:x>`, `<slug:x>`, `<uuid:x>`, `<path:x>`. Para expresiones regulares: `re_path`.

Construir URLs:

```python
from django.urls import reverse
reverse("tienda:detalle", args=[5])           # '/productos/5/'
```

```django
<a href="{% url 'tienda:detalle' producto.pk %}">{{ producto }}</a>
```

### 9.2 Vistas basadas en funciones

```python
# tienda/views.py
from django.http import HttpResponse, JsonResponse, Http404
from django.shortcuts import get_object_or_404, redirect, render
from django.core.paginator import Paginator
from django.views.decorators.http import require_POST
from django.contrib.auth.decorators import login_required

from .models import Producto


def lista(request):
    productos = Producto.objects.publicados().select_related("categoria")
    pagina = Paginator(productos, 20).get_page(request.GET.get("page"))
    return render(request, "tienda/lista.html", {"pagina": pagina})


def detalle(request, pk):
    producto = get_object_or_404(Producto, pk=pk)          # 404 si no existe
    return render(request, "tienda/detalle.html", {"producto": producto})


@login_required
@require_POST
def retirar(request, pk):
    producto = get_object_or_404(Producto, pk=pk)
    producto.estado = Producto.Estado.RETIRADO
    producto.save(update_fields=["estado"])
    return redirect("tienda:detalle", pk=pk)
```

### 9.3 El objeto `request` y las respuestas

| `request.` | Contenido |
|---|---|
| `method` | `"GET"`, `"POST"`… |
| `GET`, `POST` | Parámetros (`QueryDict`): `request.GET.get("q", "")`. |
| `FILES` | Archivos subidos. |
| `user` | Usuario (`is_authenticated`, `has_perm()`). |
| `session` | Sesión (diccionario). |
| `headers`, `COOKIES`, `META` | Cabeceras, cookies, metadatos. |
| `body` | Cuerpo en bytes (JSON: `json.loads(request.body)`). |

| Respuesta | Uso |
|---|---|
| `render(request, plantilla, contexto)` | HTML a partir de una plantilla. |
| `redirect("nombre", **kw)` | Redirección. |
| `HttpResponse(texto, status=, content_type=)` | Respuesta genérica. |
| `JsonResponse(dict)` | JSON. |
| `FileResponse(open(ruta, "rb"), as_attachment=True)` | Descarga de archivos. |
| `StreamingHttpResponse(generador)` | Respuestas grandes por partes. |
| `raise Http404("…")` | Página no encontrada. |

---

## 10. Vistas basadas en clases

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.urls import reverse_lazy
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView


class ProductoLista(ListView):
    model = Producto
    paginate_by = 20
    template_name = "tienda/lista.html"            # Por defecto: tienda/producto_list.html
    context_object_name = "productos"

    def get_queryset(self):
        qs = Producto.objects.publicados().select_related("categoria")
        if q := self.request.GET.get("q"):
            qs = qs.filter(nombre__icontains=q)
        return qs

    def get_context_data(self, **kwargs):
        contexto = super().get_context_data(**kwargs)
        contexto["q"] = self.request.GET.get("q", "")
        return contexto


class ProductoDetalle(DetailView):
    model = Producto


class ProductoCrear(LoginRequiredMixin, PermissionRequiredMixin, CreateView):
    model = Producto
    form_class = ProductoForm
    permission_required = "tienda.add_producto"

    def form_valid(self, form):
        form.instance.creado_por = self.request.user
        return super().form_valid(form)               # Redirige a get_absolute_url()


class ProductoEditar(LoginRequiredMixin, UpdateView):
    model = Producto
    form_class = ProductoForm


class ProductoBorrar(LoginRequiredMixin, DeleteView):
    model = Producto
    success_url = reverse_lazy("tienda:lista")
```

```python
path("", views.ProductoLista.as_view(), name="lista"),
path("productos/<int:pk>/", views.ProductoDetalle.as_view(), name="detalle"),
```

| Vista genérica | Plantilla por defecto |
|---|---|
| `ListView` | `<app>/<modelo>_list.html` |
| `DetailView` | `<app>/<modelo>_detail.html` |
| `CreateView` / `UpdateView` | `<app>/<modelo>_form.html` |
| `DeleteView` | `<app>/<modelo>_confirm_delete.html` |
| `TemplateView`, `RedirectView`, `FormView` | Las indicadas |

---

## 11. Plantillas

### 11.1 Herencia

```django
{# templates/base.html #}
{% load static %}
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>{% block titulo %}Tienda{% endblock %}</title>
  <link rel="stylesheet" href="{% static 'css/estilos.css' %}">
</head>
<body>
  <nav>
    <a href="{% url 'tienda:lista' %}">Inicio</a>
    {% if user.is_authenticated %}
      Hola, {{ user.get_full_name|default:user.username }}
      <form method="post" action="{% url 'logout' %}">{% csrf_token %}<button>Salir</button></form>
    {% else %}
      <a href="{% url 'login' %}">Entrar</a>
    {% endif %}
  </nav>
  {% for m in messages %}<div class="aviso {{ m.tags }}">{{ m }}</div>{% endfor %}
  <main>{% block contenido %}{% endblock %}</main>
</body>
</html>
```

```django
{# tienda/templates/tienda/lista.html #}
{% extends "base.html" %}
{% block titulo %}Productos{% endblock %}
{% block contenido %}
  <h1>Productos</h1>
  {% for p in productos %}
    <article>
      <h2><a href="{{ p.get_absolute_url }}">{{ p.nombre|title }}</a></h2>
      <p>{{ p.descripcion|truncatewords:20 }}</p>
      <strong>{{ p.precio|floatformat:2 }} €</strong>
      {% if not p.disponible %}<span>Agotado</span>{% endif %}
    </article>
  {% empty %}
    <p>No hay productos.</p>
  {% endfor %}
  {% include "tienda/_paginacion.html" %}
{% endblock %}
```

### 11.2 Sintaxis

| Elemento | Ejemplo |
|---|---|
| Variable | `{{ producto.nombre }}`, `{{ lista.0 }}`, `{{ diccionario.clave }}` |
| Filtro | `{{ texto\|lower }}`, `{{ fecha\|date:"d/m/Y H:i" }}`, `{{ valor\|default:"—" }}` |
| Etiqueta | `{% if %}`, `{% for %}`, `{% url %}`, `{% static %}`, `{% csrf_token %}`, `{% with %}` |
| Comentario | `{# … #}` o `{% comment %}…{% endcomment %}` |
| Inclusión | `{% include "parcial.html" with x=1 %}` |

Filtros útiles: `length`, `join:", "`, `linebreaks`, `truncatechars:50`, `pluralize`, `yesno:"sí,no"`, `intcomma` (con `humanize`), `timesince`, `safe` (¡con cuidado!), `escape`.

> Django **escapa automáticamente** las variables en HTML (protección XSS). Use `|safe` solo con contenido de confianza.

### 11.3 Fragmentos con nombre (Django 6.0+)

```django
{% partialdef tarjeta inline %}
  <div class="tarjeta">{{ producto.nombre }}</div>
{% endpartialdef %}

{# Desde una vista se puede renderizar solo el fragmento: "tienda/lista.html#tarjeta" #}
```

### 11.4 Etiquetas y filtros propios

```python
# tienda/templatetags/tienda_extras.py   (con __init__.py en la carpeta)
from django import template
register = template.Library()

@register.filter
def euros(valor):
    return f"{valor:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")
```

```django
{% load tienda_extras %}
{{ producto.precio|euros }}
```

---

## 12. Formularios

### 12.1 `ModelForm`

```python
# tienda/forms.py
from django import forms
from .models import Producto


class ProductoForm(forms.ModelForm):
    class Meta:
        model = Producto
        fields = ["nombre", "categoria", "descripcion", "precio", "stock", "estado", "etiquetas"]
        widgets = {
            "descripcion": forms.Textarea(attrs={"rows": 4}),
            "etiquetas": forms.CheckboxSelectMultiple,
        }
        labels = {"stock": "Unidades en stock"}
        help_texts = {"precio": "IVA incluido"}

    def clean_nombre(self):
        nombre = self.cleaned_data["nombre"].strip()
        if len(nombre) < 3:
            raise forms.ValidationError("El nombre debe tener al menos 3 caracteres.")
        return nombre

    def clean(self):
        datos = super().clean()
        if datos.get("estado") == Producto.Estado.PUBLICADO and not datos.get("precio"):
            self.add_error("precio", "Un producto publicado necesita precio.")
        return datos
```

### 12.2 Procesar un formulario en una vista

```python
def crear(request):
    if request.method == "POST":
        form = ProductoForm(request.POST, request.FILES)
        if form.is_valid():
            producto = form.save(commit=False)
            producto.creado_por = request.user
            producto.save()
            form.save_m2m()                       # Necesario con commit=False y ManyToMany
            messages.success(request, "Producto creado.")
            return redirect(producto)
    else:
        form = ProductoForm()
    return render(request, "tienda/producto_form.html", {"form": form})
```

### 12.3 Plantilla

```django
<form method="post" enctype="multipart/form-data" novalidate>
  {% csrf_token %}
  {{ form.non_field_errors }}
  {% for campo in form %}
    <div class="campo{% if campo.errors %} con-error{% endif %}">
      {{ campo.label_tag }} {{ campo }}
      {% if campo.help_text %}<small>{{ campo.help_text }}</small>{% endif %}
      {{ campo.errors }}
    </div>
  {% endfor %}
  <button type="submit">Guardar</button>
</form>
```

Formas rápidas: `{{ form }}`, `{{ form.as_div }}`, `{{ form.as_p }}`. Para estilos con Bootstrap o Tailwind: `django-crispy-forms`.

### 12.4 Formularios sin modelo y *formsets*

```python
class ContactoForm(forms.Form):
    nombre = forms.CharField(max_length=100)
    email = forms.EmailField()
    mensaje = forms.CharField(widget=forms.Textarea, min_length=10)
    acepta = forms.BooleanField(label="Acepto la política de privacidad")

LineaFormSet = forms.inlineformset_factory(Pedido, LineaPedido, fields=["producto", "cantidad"], extra=2)
```

---

## 13. Archivos estáticos y subidos

```python
# config/urls.py (solo en desarrollo)
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

```bash
python manage.py collectstatic        # Copia todos los estáticos a STATIC_ROOT para producción
```

En producción, los estáticos los sirve el servidor web (Nginx) o **WhiteNoise** (`pip install whitenoise`, middleware `whitenoise.middleware.WhiteNoiseMiddleware`). Los archivos subidos (`MEDIA`) suelen guardarse en almacenamiento externo (S3 con `django-storages`).

---

## 14. Autenticación y permisos

### 14.1 Vistas de inicio y cierre de sesión

```python
# config/urls.py
path("cuentas/", include("django.contrib.auth.urls")),
# Proporciona: login, logout, password_change, password_reset…
# Plantillas esperadas: registration/login.html, etc.
```

```django
{# templates/registration/login.html #}
{% extends "base.html" %}
{% block contenido %}
<form method="post">{% csrf_token %}{{ form.as_div }}<button>Entrar</button></form>
{% endblock %}
```

> Desde Django 5.0, el cierre de sesión debe hacerse con **POST** (como en `base.html` del apartado 11.1).

### 14.2 Proteger vistas

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin, UserPassesTestMixin

@login_required
def perfil(request): ...

@permission_required("tienda.change_producto", raise_exception=True)
def editar(request, pk): ...

class SoloAutor(UserPassesTestMixin, UpdateView):
    def test_func(self):
        return self.get_object().creado_por == self.request.user
```

Django 5.1+ incluye `LoginRequiredMiddleware`, que exige sesión en **todas** las vistas salvo las marcadas con `@login_not_required`.

### 14.3 Usuarios, grupos y permisos

```python
from django.contrib.auth.models import User, Group, Permission

u = User.objects.create_user("ana", "ana@ejemplo.com", "ClaveSegura123")
editores, _ = Group.objects.get_or_create(name="Editores")
editores.permissions.add(Permission.objects.get(codename="change_producto"))
u.groups.add(editores)
u.has_perm("tienda.change_producto")          # True
```

```django
{% if perms.tienda.change_producto %}<a href="…">Editar</a>{% endif %}
```

### 14.4 Modelo de usuario personalizado

Defínalo **al principio** del proyecto (cambiarlo después es costoso):

```python
# cuentas/models.py
from django.contrib.auth.models import AbstractUser

class Usuario(AbstractUser):
    telefono = models.CharField(max_length=20, blank=True)

# settings.py
AUTH_USER_MODEL = "cuentas.Usuario"
```

En el código, refiérase siempre a `settings.AUTH_USER_MODEL` o `get_user_model()`.

---

## 15. Middleware, señales, mensajes y caché

### 15.1 Middleware propio

```python
import time, logging
log = logging.getLogger(__name__)

class TiempoRespuesta:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        inicio = time.perf_counter()
        respuesta = self.get_response(request)
        respuesta["X-Tiempo-Respuesta"] = f"{(time.perf_counter() - inicio) * 1000:.1f}ms"
        return respuesta

# settings.MIDDLEWARE.append("config.middleware.TiempoRespuesta")
```

### 15.2 Señales

```python
# tienda/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Producto

@receiver(post_save, sender=Producto)
def avisar_stock_bajo(sender, instance, created, **kwargs):
    if instance.stock < 5:
        ...  # enviar aviso

# tienda/apps.py
class TiendaConfig(AppConfig):
    name = "tienda"
    def ready(self):
        from . import signals  # noqa: F401
```

> Use las señales con moderación: la lógica explícita (en servicios o en `save()`) es más fácil de seguir.

### 15.3 Mensajes

```python
from django.contrib import messages
messages.success(request, "Guardado correctamente.")
messages.error(request, "No se pudo guardar.")
```

### 15.4 Caché

```python
from django.views.decorators.cache import cache_page
from django.core.cache import cache

@cache_page(60 * 5)
def portada(request): ...

total = cache.get_or_set("total_productos", lambda: Producto.objects.count(), timeout=300)
```

```django
{% load cache %}{% cache 600 menu_categorias %}…{% endcache %}
```

### 15.5 Correo

```python
from django.core.mail import send_mail
send_mail("Asunto", "Texto", "no-responder@ejemplo.com", ["ana@ejemplo.com"])
# Desarrollo: EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"
```

---

## 16. Comandos de gestión

### 16.1 Comandos integrados

| Comando | Uso |
|---|---|
| `runserver [puerto]` | Servidor de desarrollo. |
| `makemigrations`, `migrate`, `showmigrations` | Migraciones. |
| `createsuperuser`, `changepassword` | Usuarios. |
| `shell`, `dbshell` | Consolas. |
| `test` | Ejecutar pruebas. |
| `collectstatic` | Reunir estáticos. |
| `check --deploy` | Revisión de seguridad para producción. |
| `dumpdata`, `loaddata` | Exportar/importar datos (*fixtures*). |
| `inspectdb` | Generar modelos a partir de una base de datos existente. |

### 16.2 Comando propio

```python
# tienda/management/commands/retirar_sin_stock.py   (con __init__.py en management/ y commands/)
from django.core.management.base import BaseCommand
from tienda.models import Producto

class Command(BaseCommand):
    help = "Retira los productos publicados sin stock"

    def add_arguments(self, parser):
        parser.add_argument("--simular", action="store_true")

    def handle(self, *args, **opciones):
        qs = Producto.objects.publicados().filter(stock=0)
        if opciones["simular"]:
            self.stdout.write(f"Se retirarían {qs.count()} productos")
            return
        n = qs.update(estado=Producto.Estado.RETIRADO)
        self.stdout.write(self.style.SUCCESS(f"{n} productos retirados"))
```

```bash
python manage.py retirar_sin_stock --simular
```

---

## 17. APIs JSON y Django REST Framework

### 17.1 JSON con Django puro

```python
from django.http import JsonResponse

def api_productos(request):
    datos = list(Producto.objects.publicados().values("id", "nombre", "precio", "stock"))
    return JsonResponse({"resultados": datos, "total": len(datos)})
```

### 17.2 Django REST Framework (resumen)

```bash
pip install djangorestframework
```

```python
from rest_framework import serializers, viewsets, permissions, routers

class ProductoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Producto
        fields = ["id", "nombre", "precio", "stock", "estado", "categoria"]

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.select_related("categoria")
    serializer_class = ProductoSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

router = routers.DefaultRouter()
router.register("productos", ProductoViewSet)
urlpatterns = [path("api/", include(router.urls))]
```

Alternativa con *type hints* al estilo FastAPI: **Django Ninja**.

---

## 18. Tareas en segundo plano

Django 6.0 incorpora una API estándar de tareas (`django.tasks`). Define **cómo** declarar y encolar tareas; la ejecución la hace un *backend* (de terceros para producción, como `django-tasks`).

```python
from django.tasks import task

@task
def enviar_resumen(usuario_id: int) -> None:
    ...

enviar_resumen.enqueue(usuario_id=5)
```

```python
TASKS = {"default": {"BACKEND": "django.tasks.backends.immediate.ImmediateBackend"}}   # Ejecuta al momento (desarrollo)
```

En versiones anteriores o para necesidades avanzadas (reintentos, programación periódica): **Celery**, **RQ** o **Dramatiq**.

---

## 19. Pruebas

```python
# tienda/tests.py
from django.contrib.auth.models import User
from django.test import TestCase
from django.urls import reverse
from .models import Categoria, Producto


class ProductoTests(TestCase):
    @classmethod
    def setUpTestData(cls):                          # Datos compartidos (una vez por clase)
        cls.cat = Categoria.objects.create(nombre="Redes", slug="redes")
        cls.usuario = User.objects.create_user("ana", password="clave-segura-1")

    def test_disponible(self):
        p = Producto.objects.create(nombre="Router", precio=50, stock=0, categoria=self.cat,
                                    estado=Producto.Estado.PUBLICADO)
        self.assertFalse(p.disponible)

    def test_lista_muestra_publicados(self):
        Producto.objects.create(nombre="Switch", precio=30, stock=3, categoria=self.cat,
                                estado=Producto.Estado.PUBLICADO)
        r = self.client.get(reverse("tienda:lista"))
        self.assertEqual(r.status_code, 200)
        self.assertContains(r, "Switch")

    def test_crear_requiere_login(self):
        r = self.client.get(reverse("tienda:crear"))
        self.assertEqual(r.status_code, 302)

    def test_crear_con_login(self):
        self.client.force_login(self.usuario)
        ...
```

```bash
python manage.py test                     # Crea una base de datos de pruebas temporal
python manage.py test tienda.tests.ProductoTests.test_disponible
```

Herramientas habituales: `pytest-django`, `factory_boy` (datos de prueba), `coverage`.

---

## 20. Seguridad y despliegue

### 20.1 Lista de comprobación

```bash
python manage.py check --deploy
```

```python
# settings de producción
DEBUG = False
ALLOWED_HOSTS = ["www.ejemplo.com"]
SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
CSRF_TRUSTED_ORIGINS = ["https://www.ejemplo.com"]
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")   # Detrás de un proxy HTTPS
```

Django 6.0+ incluye además soporte nativo de **Content Security Policy** (`django.middleware.csp.ContentSecurityPolicyMiddleware` y el ajuste `SECURE_CSP`).

### 20.2 Servidores

```bash
pip install gunicorn
gunicorn config.wsgi:application --workers 4 --bind 0.0.0.0:8000      # WSGI

pip install uvicorn
uvicorn config.asgi:application --workers 4                           # ASGI (vistas async, WebSockets con Channels)
```

Arquitectura típica: **Nginx** (HTTPS, estáticos) → **Gunicorn/Uvicorn** → Django → **PostgreSQL**; Redis para caché y tareas.

> `runserver` es **solo para desarrollo**: no es seguro ni eficiente en producción.

---

## 21. Ejemplo práctico completo

**Gestor de incidencias de soporte**: modelos con relaciones y opciones, administración personalizada, lista con búsqueda, filtros y paginación, detalle con comentarios, alta y edición protegidas por inicio de sesión, cambio de estado por POST, API JSON, comando para cargar datos de demostración y pruebas automatizadas.

### 21.1 Creación del proyecto

```bash
django-admin startproject config soporte
cd soporte
python manage.py startapp incidencias
```

En `config/settings.py`, añada `"incidencias"` a `INSTALLED_APPS`, configure el idioma y las plantillas globales:

```python
INSTALLED_APPS += ["incidencias"]
LANGUAGE_CODE = "es-es"
TIME_ZONE = "Europe/Madrid"
TEMPLATES[0]["DIRS"] = [BASE_DIR / "templates"]
LOGIN_URL = "login"
LOGIN_REDIRECT_URL = "incidencias:lista"
LOGOUT_REDIRECT_URL = "incidencias:lista"
```

### 21.2 `incidencias/models.py`

```python
from django.conf import settings
from django.db import models
from django.urls import reverse
from django.utils import timezone


class IncidenciaQuerySet(models.QuerySet):
    def abiertas(self):
        return self.exclude(estado=Incidencia.Estado.CERRADA)

    def buscar(self, texto: str):
        if not texto:
            return self
        return self.filter(models.Q(titulo__icontains=texto) | models.Q(descripcion__icontains=texto))


class Incidencia(models.Model):
    class Prioridad(models.IntegerChoices):
        BAJA = 1, "Baja"
        MEDIA = 2, "Media"
        ALTA = 3, "Alta"
        CRITICA = 4, "Crítica"

    class Estado(models.TextChoices):
        NUEVA = "nueva", "Nueva"
        EN_CURSO = "en_curso", "En curso"
        RESUELTA = "resuelta", "Resuelta"
        CERRADA = "cerrada", "Cerrada"

    TRANSICIONES = {
        "nueva": ["en_curso", "cerrada"],
        "en_curso": ["resuelta", "nueva"],
        "resuelta": ["cerrada", "en_curso"],
        "cerrada": ["en_curso"],
    }

    titulo = models.CharField(max_length=150)
    descripcion = models.TextField()
    prioridad = models.IntegerField(choices=Prioridad, default=Prioridad.MEDIA)
    estado = models.CharField(max_length=10, choices=Estado, default=Estado.NUEVA)
    creada_por = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT,
                                   related_name="incidencias_creadas")
    asignada_a = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL, null=True, blank=True,
                                   related_name="incidencias_asignadas")
    creada = models.DateTimeField(auto_now_add=True)
    actualizada = models.DateTimeField(auto_now=True)
    cerrada_en = models.DateTimeField(null=True, blank=True)

    objects = IncidenciaQuerySet.as_manager()

    class Meta:
        ordering = ["-prioridad", "-creada"]
        indexes = [models.Index(fields=["estado", "prioridad"])]

    def __str__(self) -> str:
        return f"#{self.pk} {self.titulo}"

    def get_absolute_url(self) -> str:
        return reverse("incidencias:detalle", args=[self.pk])

    def estados_siguientes(self) -> list[tuple[str, str]]:
        return [(e, self.Estado(e).label) for e in self.TRANSICIONES[self.estado]]

    def cambiar_estado(self, nuevo: str) -> None:
        if nuevo not in self.TRANSICIONES[self.estado]:
            raise ValueError(f"No se puede pasar de «{self.get_estado_display()}» a «{nuevo}».")
        self.estado = nuevo
        self.cerrada_en = timezone.now() if nuevo == self.Estado.CERRADA else None
        self.save(update_fields=["estado", "cerrada_en", "actualizada"])


class Comentario(models.Model):
    incidencia = models.ForeignKey(Incidencia, on_delete=models.CASCADE, related_name="comentarios")
    autor = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    texto = models.TextField()
    creado = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["creado"]

    def __str__(self) -> str:
        return f"Comentario de {self.autor} en {self.incidencia}"
```

### 21.3 `incidencias/admin.py`

```python
from django.contrib import admin
from .models import Comentario, Incidencia


class ComentarioInline(admin.TabularInline):
    model = Comentario
    extra = 0
    readonly_fields = ["creado"]


@admin.register(Incidencia)
class IncidenciaAdmin(admin.ModelAdmin):
    list_display = ["id", "titulo", "prioridad", "estado", "asignada_a", "creada"]
    list_filter = ["estado", "prioridad", "asignada_a"]
    search_fields = ["titulo", "descripcion"]
    list_editable = ["estado"]
    date_hierarchy = "creada"
    readonly_fields = ["creada", "actualizada", "cerrada_en"]
    inlines = [ComentarioInline]
    actions = ["cerrar"]

    @admin.action(description="Cerrar las incidencias seleccionadas")
    def cerrar(self, request, queryset):
        n = queryset.update(estado=Incidencia.Estado.CERRADA)
        self.message_user(request, f"{n} incidencias cerradas.")
```

### 21.4 `incidencias/forms.py`

```python
from django import forms
from .models import Comentario, Incidencia


class IncidenciaForm(forms.ModelForm):
    class Meta:
        model = Incidencia
        fields = ["titulo", "descripcion", "prioridad", "asignada_a"]
        widgets = {"descripcion": forms.Textarea(attrs={"rows": 5})}

    def clean_titulo(self):
        titulo = self.cleaned_data["titulo"].strip()
        if len(titulo) < 5:
            raise forms.ValidationError("Describa la incidencia con al menos 5 caracteres.")
        return titulo


class ComentarioForm(forms.ModelForm):
    class Meta:
        model = Comentario
        fields = ["texto"]
        widgets = {"texto": forms.Textarea(attrs={"rows": 3, "placeholder": "Añadir un comentario…"})}
        labels = {"texto": ""}
```

### 21.5 `incidencias/views.py`

```python
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from django.contrib.auth.mixins import LoginRequiredMixin
from django.db.models import Count
from django.http import JsonResponse
from django.shortcuts import get_object_or_404, redirect
from django.views.decorators.http import require_POST
from django.views.generic import CreateView, DetailView, ListView, UpdateView

from .forms import ComentarioForm, IncidenciaForm
from .models import Incidencia


class ListaIncidencias(ListView):
    model = Incidencia
    paginate_by = 10
    context_object_name = "incidencias"

    def get_queryset(self):
        qs = Incidencia.objects.select_related("asignada_a").annotate(num_comentarios=Count("comentarios"))
        qs = qs.buscar(self.request.GET.get("q", "").strip())
        if estado := self.request.GET.get("estado"):
            qs = qs.filter(estado=estado)
        elif not self.request.GET.get("todas"):
            qs = qs.abiertas()
        if prioridad := self.request.GET.get("prioridad"):
            qs = qs.filter(prioridad=prioridad)
        return qs.order_by("-prioridad", "-creada")      # Orden explícito: necesario para paginar

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx.update(
            q=self.request.GET.get("q", ""),
            estados=Incidencia.Estado.choices,
            prioridades=Incidencia.Prioridad.choices,
        )
        conteo = dict(Incidencia.objects.values_list("estado").annotate(n=Count("id")).order_by())
        ctx["resumen"] = [(texto, conteo.get(clave, 0)) for clave, texto in Incidencia.Estado.choices]
        params = self.request.GET.copy()
        params.pop("page", None)
        ctx["params"] = params.urlencode()
        return ctx


class DetalleIncidencia(DetailView):
    model = Incidencia

    def get_queryset(self):
        return Incidencia.objects.select_related("creada_por", "asignada_a").prefetch_related("comentarios__autor")

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx["form_comentario"] = ComentarioForm()
        return ctx


class CrearIncidencia(LoginRequiredMixin, CreateView):
    model = Incidencia
    form_class = IncidenciaForm

    def form_valid(self, form):
        form.instance.creada_por = self.request.user
        messages.success(self.request, "Incidencia registrada.")
        return super().form_valid(form)


class EditarIncidencia(LoginRequiredMixin, UpdateView):
    model = Incidencia
    form_class = IncidenciaForm

    def form_valid(self, form):
        messages.success(self.request, "Cambios guardados.")
        return super().form_valid(form)


@login_required
@require_POST
def comentar(request, pk):
    incidencia = get_object_or_404(Incidencia, pk=pk)
    form = ComentarioForm(request.POST)
    if form.is_valid():
        comentario = form.save(commit=False)
        comentario.incidencia, comentario.autor = incidencia, request.user
        comentario.save()
        messages.success(request, "Comentario añadido.")
    else:
        messages.error(request, "El comentario no puede estar vacío.")
    return redirect(incidencia)


@login_required
@require_POST
def cambiar_estado(request, pk):
    incidencia = get_object_or_404(Incidencia, pk=pk)
    try:
        incidencia.cambiar_estado(request.POST.get("estado", ""))
        messages.success(request, f"Estado cambiado a «{incidencia.get_estado_display()}».")
    except ValueError as e:
        messages.error(request, str(e))
    return redirect(incidencia)


def api_incidencias(request):
    qs = Incidencia.objects.abiertas().buscar(request.GET.get("q", "")).select_related("asignada_a")[:100]
    return JsonResponse({"resultados": [{
        "id": i.pk, "titulo": i.titulo, "estado": i.estado, "prioridad": i.get_prioridad_display(),
        "asignada_a": i.asignada_a.username if i.asignada_a else None,
        "url": request.build_absolute_uri(i.get_absolute_url()),
    } for i in qs]})
```

### 21.6 `incidencias/urls.py` y `config/urls.py`

```python
# incidencias/urls.py
from django.urls import path
from . import views

app_name = "incidencias"
urlpatterns = [
    path("", views.ListaIncidencias.as_view(), name="lista"),
    path("nueva/", views.CrearIncidencia.as_view(), name="crear"),
    path("<int:pk>/", views.DetalleIncidencia.as_view(), name="detalle"),
    path("<int:pk>/editar/", views.EditarIncidencia.as_view(), name="editar"),
    path("<int:pk>/comentar/", views.comentar, name="comentar"),
    path("<int:pk>/estado/", views.cambiar_estado, name="estado"),
    path("api/incidencias/", views.api_incidencias, name="api"),
]
```

```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("cuentas/", include("django.contrib.auth.urls")),
    path("", include("incidencias.urls")),
]
```

### 21.7 Plantillas

```django
{# templates/base.html #}
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block titulo %}Soporte{% endblock %}</title>
  <style>
    body { font-family: system-ui, sans-serif; max-width: 960px; margin: 0 auto; padding: 1rem; color: #1f2933; }
    nav { display: flex; gap: 1rem; align-items: center; border-bottom: 2px solid #1F4E78; padding-bottom: .5rem; }
    nav .derecha { margin-left: auto; }
    table { width: 100%; border-collapse: collapse; } td, th { padding: .4rem; border-bottom: 1px solid #ddd; text-align: left; }
    .p4 { color: #C00000; font-weight: bold; } .p3 { color: #E07B00; }
    .aviso { padding: .5rem; margin: .5rem 0; background: #E8F4EA; } .aviso.error { background: #FBE9E9; }
    .estado { padding: .1rem .4rem; border-radius: 4px; background: #e9eef3; font-size: .85rem; }
    form.inline { display: inline; }
  </style>
</head>
<body>
  <nav>
    <a href="{% url 'incidencias:lista' %}"><strong>Soporte</strong></a>
    <a href="{% url 'incidencias:crear' %}">Nueva incidencia</a>
    <span class="derecha">
      {% if user.is_authenticated %}
        {{ user.username }}
        <form class="inline" method="post" action="{% url 'logout' %}">{% csrf_token %}<button>Salir</button></form>
      {% else %}<a href="{% url 'login' %}">Entrar</a>{% endif %}
    </span>
  </nav>
  {% for m in messages %}<div class="aviso {{ m.tags }}">{{ m }}</div>{% endfor %}
  {% block contenido %}{% endblock %}
</body>
</html>
```

```django
{# incidencias/templates/incidencias/incidencia_list.html #}
{% extends "base.html" %}
{% block titulo %}Incidencias{% endblock %}
{% block contenido %}
<h1>Incidencias</h1>
<p>{% for texto, total in resumen %}<span class="estado">{{ texto }}: {{ total }}</span> {% endfor %}</p>
<form method="get">
  <input name="q" value="{{ q }}" placeholder="Buscar…">
  <select name="estado"><option value="">Abiertas</option>
    {% for clave, texto in estados %}<option value="{{ clave }}" {% if request.GET.estado == clave %}selected{% endif %}>{{ texto }}</option>{% endfor %}
  </select>
  <select name="prioridad"><option value="">Todas las prioridades</option>
    {% for clave, texto in prioridades %}<option value="{{ clave }}" {% if request.GET.prioridad == clave|stringformat:"s" %}selected{% endif %}>{{ texto }}</option>{% endfor %}
  </select>
  <button>Filtrar</button>
</form>
<table>
  <tr><th>#</th><th>Título</th><th>Prioridad</th><th>Estado</th><th>Asignada a</th><th>💬</th></tr>
  {% for i in incidencias %}
  <tr>
    <td>{{ i.pk }}</td>
    <td><a href="{{ i.get_absolute_url }}">{{ i.titulo }}</a></td>
    <td class="p{{ i.prioridad }}">{{ i.get_prioridad_display }}</td>
    <td><span class="estado">{{ i.get_estado_display }}</span></td>
    <td>{{ i.asignada_a|default:"—" }}</td>
    <td>{{ i.num_comentarios }}</td>
  </tr>
  {% empty %}<tr><td colspan="6">No hay incidencias.</td></tr>{% endfor %}
</table>
{% if is_paginated %}
<p>
  {% if page_obj.has_previous %}<a href="?{{ params }}&page={{ page_obj.previous_page_number }}">« Anterior</a>{% endif %}
  Página {{ page_obj.number }} de {{ paginator.num_pages }}
  {% if page_obj.has_next %}<a href="?{{ params }}&page={{ page_obj.next_page_number }}">Siguiente »</a>{% endif %}
</p>
{% endif %}
{% endblock %}
```

```django
{# incidencias/templates/incidencias/incidencia_detail.html #}
{% extends "base.html" %}
{% block titulo %}{{ incidencia }}{% endblock %}
{% block contenido %}
<h1>{{ incidencia }}</h1>
<p>
  <span class="estado">{{ incidencia.get_estado_display }}</span>
  · Prioridad <strong class="p{{ incidencia.prioridad }}">{{ incidencia.get_prioridad_display }}</strong>
  · Abierta por {{ incidencia.creada_por }} el {{ incidencia.creada|date:"d/m/Y H:i" }}
  · Asignada a {{ incidencia.asignada_a|default:"nadie" }}
</p>
<p>{{ incidencia.descripcion|linebreaksbr }}</p>
{% if user.is_authenticated %}
  <a href="{% url 'incidencias:editar' incidencia.pk %}">Editar</a>
  {% for clave, texto in incidencia.estados_siguientes %}
    <form class="inline" method="post" action="{% url 'incidencias:estado' incidencia.pk %}">
      {% csrf_token %}<button name="estado" value="{{ clave }}">→ {{ texto }}</button>
    </form>
  {% endfor %}
{% endif %}
<h2>Comentarios ({{ incidencia.comentarios.all|length }})</h2>
{% for c in incidencia.comentarios.all %}
  <p><strong>{{ c.autor }}</strong> · {{ c.creado|timesince }}<br>{{ c.texto|linebreaksbr }}</p>
{% empty %}<p>Sin comentarios.</p>{% endfor %}
{% if user.is_authenticated %}
<form method="post" action="{% url 'incidencias:comentar' incidencia.pk %}">
  {% csrf_token %}{{ form_comentario.as_div }}<button>Comentar</button>
</form>
{% endif %}
{% endblock %}
```

```django
{# incidencias/templates/incidencias/incidencia_form.html #}
{% extends "base.html" %}
{% block contenido %}
<h1>{% if object %}Editar {{ object }}{% else %}Nueva incidencia{% endif %}</h1>
<form method="post" novalidate>{% csrf_token %}{{ form.as_div }}<button>Guardar</button></form>
{% endblock %}
```

```django
{# templates/registration/login.html #}
{% extends "base.html" %}
{% block contenido %}
<h1>Iniciar sesión</h1>
<form method="post">{% csrf_token %}{{ form.as_div }}<input type="hidden" name="next" value="{{ next }}"><button>Entrar</button></form>
{% endblock %}
```

### 21.8 Comando de datos de demostración

```python
# incidencias/management/commands/cargar_demo.py  (cree __init__.py en management/ y commands/)
import random

from django.contrib.auth import get_user_model
from django.core.management.base import BaseCommand
from django.db import transaction

from incidencias.models import Comentario, Incidencia

TITULOS = ["No arranca el portátil", "Error al imprimir", "VPN desconectada", "Correo no sincroniza",
           "Pantalla parpadea", "Acceso denegado a carpeta", "Lentitud en el ERP", "Teclado no responde"]


class Command(BaseCommand):
    help = "Crea usuarios e incidencias de ejemplo"

    def add_arguments(self, parser):
        parser.add_argument("--cantidad", type=int, default=30)

    @transaction.atomic
    def handle(self, *args, cantidad, **opciones):
        Usuario = get_user_model()
        if not Usuario.objects.filter(username="admin").exists():
            Usuario.objects.create_superuser("admin", "admin@ejemplo.com", "admin1234")
        tecnicos = []
        for nombre in ["laura", "pablo", "irene"]:
            u, creado = Usuario.objects.get_or_create(username=nombre)
            if creado:
                u.set_password("demo1234")
                u.save()
            tecnicos.append(u)
        random.seed(1)
        for _ in range(cantidad):
            inc = Incidencia.objects.create(
                titulo=random.choice(TITULOS), descripcion="Descripción generada automáticamente.",
                prioridad=random.choice(Incidencia.Prioridad.values),
                estado=random.choice(Incidencia.Estado.values),
                creada_por=random.choice(tecnicos), asignada_a=random.choice([*tecnicos, None]))
            for _ in range(random.randint(0, 3)):
                Comentario.objects.create(incidencia=inc, autor=random.choice(tecnicos), texto="Revisando…")
        self.stdout.write(self.style.SUCCESS(f"{cantidad} incidencias creadas (usuario admin / admin1234)."))
```

### 21.9 Pruebas: `incidencias/tests.py`

```python
from django.contrib.auth import get_user_model
from django.test import TestCase
from django.urls import reverse

from .models import Incidencia


class IncidenciaTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.usuario = get_user_model().objects.create_user("ana", password="clave-segura-1")
        cls.inc = Incidencia.objects.create(titulo="Impresora atascada", descripcion="Planta 2",
                                            creada_por=cls.usuario, prioridad=Incidencia.Prioridad.ALTA)

    def test_transicion_valida_e_invalida(self):
        self.inc.cambiar_estado("en_curso")
        self.assertEqual(self.inc.estado, "en_curso")
        with self.assertRaises(ValueError):
            self.inc.cambiar_estado("cerrada")

    def test_lista_y_busqueda(self):
        r = self.client.get(reverse("incidencias:lista"), {"q": "impresora"})
        self.assertContains(r, "Impresora atascada")
        r = self.client.get(reverse("incidencias:lista"), {"q": "inexistente"})
        self.assertContains(r, "No hay incidencias")

    def test_crear_requiere_login(self):
        r = self.client.get(reverse("incidencias:crear"))
        self.assertRedirects(r, f"{reverse('login')}?next={reverse('incidencias:crear')}")

    def test_crear_valida_titulo(self):
        self.client.force_login(self.usuario)
        r = self.client.post(reverse("incidencias:crear"), {"titulo": "abc", "descripcion": "x", "prioridad": 2})
        self.assertContains(r, "al menos 5 caracteres")
        r = self.client.post(reverse("incidencias:crear"),
                             {"titulo": "Monitor sin señal", "descripcion": "Sala B", "prioridad": 3})
        nueva = Incidencia.objects.get(titulo="Monitor sin señal")
        self.assertRedirects(r, nueva.get_absolute_url())
        self.assertEqual(nueva.creada_por, self.usuario)

    def test_comentar_y_cambiar_estado(self):
        self.client.force_login(self.usuario)
        self.client.post(reverse("incidencias:comentar", args=[self.inc.pk]), {"texto": "Reiniciada"})
        self.assertEqual(self.inc.comentarios.count(), 1)
        self.client.post(reverse("incidencias:estado", args=[self.inc.pk]), {"estado": "cerrada"})
        self.inc.refresh_from_db()
        self.assertEqual(self.inc.estado, "cerrada")
        self.assertIsNotNone(self.inc.cerrada_en)

    def test_api(self):
        datos = self.client.get(reverse("incidencias:api")).json()
        self.assertEqual(datos["resultados"][0]["titulo"], "Impresora atascada")
```

### 21.10 Puesta en marcha

```bash
python manage.py makemigrations incidencias
python manage.py migrate
python manage.py cargar_demo --cantidad 40
python manage.py test incidencias
python manage.py runserver
# http://127.0.0.1:8000  ·  http://127.0.0.1:8000/admin  (admin / admin1234)
```

---

## 22. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `TemplateDoesNotExist` | Ruta o nombre de la plantilla incorrectos, o app no registrada. | Revise `INSTALLED_APPS`, `TEMPLATES["DIRS"]` y la subcarpeta con el nombre de la app. |
| `no such table: app_modelo` | Migraciones no creadas o no aplicadas. | `makemigrations` + `migrate`. |
| `NoReverseMatch` | Nombre de URL o argumentos incorrectos; falta el espacio de nombres. | `{% url 'app:nombre' arg %}` y `app_name` en `urls.py`. |
| `403 CSRF verification failed` | Falta `{% csrf_token %}` o el origen no es de confianza. | Añada la etiqueta; configure `CSRF_TRUSTED_ORIGINS` con HTTPS. |
| `DisallowedHost` | El dominio no está en `ALLOWED_HOSTS`. | Añádalo. |
| `RelatedObjectDoesNotExist` | Se accede a una relación `OneToOne` inexistente. | Compruebe con `hasattr()` o capture la excepción. |
| Páginas lentas con muchas consultas | Problema N+1. | `select_related` / `prefetch_related`; use django-debug-toolbar. |
| Estáticos sin cargar con `DEBUG=False` | Django no sirve estáticos en producción. | `collectstatic` + Nginx o WhiteNoise. |
| `ImproperlyConfigured: SECRET_KEY` | Variable de entorno no definida. | Defínala en el entorno del servidor. |
| `SynchronousOnlyOperation` | Consulta ORM síncrona en contexto `async`. | Use los métodos `a…` (`aget`, `afilter`…) o `sync_to_async`. |
| Cambiar `AUTH_USER_MODEL` rompe las migraciones | Se cambió con el proyecto avanzado. | Defina el usuario personalizado al iniciar el proyecto. |

---

## 23. Buenas prácticas

- Cree un modelo de usuario personalizado al empezar, aunque sea vacío.
- Mantenga las aplicaciones pequeñas y centradas en una responsabilidad.
- Ponga la lógica de dominio en los modelos o en módulos de servicios, no en las plantillas ni en vistas largas.
- Use `QuerySet` personalizados para consultas reutilizables (`.abiertas()`, `.publicados()`).
- Vigile el número de consultas (`select_related`, `prefetch_related`, django-debug-toolbar).
- Guarde secretos en variables de entorno y separe la configuración de desarrollo y producción.
- Use siempre `{% csrf_token %}` y métodos POST para operaciones que modifican datos.
- Escriba pruebas para modelos, formularios y vistas; ejecútelas en integración continua.
- Ejecute `check --deploy` antes de cada despliegue y use HTTPS.
- Use PostgreSQL en producción y el mismo motor en desarrollo cuando sea posible.

---

## 24. Referencia rápida

```bash
django-admin startproject config . · python manage.py startapp app
makemigrations · migrate · createsuperuser · runserver · shell · test · collectstatic · check --deploy
```

```python
class M(models.Model): campo = models.CharField(max_length=100); fk = models.ForeignKey(X, on_delete=models.CASCADE)
M.objects.create/get/filter/exclude/order_by/values/annotate/aggregate/select_related/prefetch_related/update/delete
Q(a=1) | Q(b=2) · F("campo") + 1 · Count Sum Avg · transaction.atomic()
@admin.register(M) class MAdmin(admin.ModelAdmin): list_display list_filter search_fields actions inlines
path("ruta/<int:pk>/", vista, name="n") · app_name = "app" · include("app.urls") · reverse("app:n", args=[1])
render(request, "app/x.html", ctx) · redirect(obj) · get_object_or_404(M, pk=pk) · JsonResponse({...})
ListView DetailView CreateView UpdateView DeleteView · LoginRequiredMixin · @login_required · @require_POST
{% extends %} {% block %} {% url %} {% static %} {% csrf_token %} {% for %}{% empty %} {{ x|filtro }}
class F(forms.ModelForm): class Meta: model, fields · clean_<campo>() · clean() · form.is_valid() · form.save()
messages.success(request, "…") · @cache_page(60) · BaseCommand.handle() · TestCase + self.client
```

---

## 25. Ejercicios propuestos

1. Cree un blog con modelos `Articulo` y `Etiqueta`, vistas de lista (paginada) y detalle, y administración con publicación programada.
2. Amplíe el ejemplo completo con un panel de indicadores: incidencias abiertas por técnico, tiempo medio de resolución y gráfico por prioridad.
3. Añada al gestor de incidencias adjuntos (`FileField`) a los comentarios, con validación de tamaño y tipo.
4. Exponga las incidencias con Django REST Framework (lista, detalle, creación) con autenticación por *token*.
5. Prepare el proyecto para producción: ajustes por variables de entorno, PostgreSQL, WhiteNoise, Gunicorn y `check --deploy` sin avisos.

---

## 26. Referencias

- Documentación oficial: https://docs.djangoproject.com/es/
- Tutorial oficial: https://docs.djangoproject.com/es/stable/intro/tutorial01/
- Lista de comprobación de despliegue: https://docs.djangoproject.com/en/stable/howto/deployment/checklist/
- Django REST Framework: https://www.django-rest-framework.org/
- Django Ninja: https://django-ninja.dev/
