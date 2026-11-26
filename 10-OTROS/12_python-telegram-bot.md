# Manual de uso: python-telegram-bot

> **Categoría:** Utilidades generales · **Utilidad:** Crear bots de Telegram
> **Tipo:** Librería externa · **Versión de referencia:** 21.x / 22.x (asíncrona)

---

## Índice

1. [Descripción](#1-descripción)
2. [Requisitos previos: crear el bot](#2-requisitos-previos-crear-el-bot)
3. [Instalación e importación](#3-instalación-e-importación)
4. [Conceptos clave](#4-conceptos-clave)
5. [Operaciones básicas](#5-operaciones-básicas)
6. [Enviar contenido](#6-enviar-contenido)
7. [Teclados y botones](#7-teclados-y-botones)
8. [Conversaciones guiadas](#8-conversaciones-guiadas)
9. [Operaciones avanzadas](#9-operaciones-avanzadas)
10. [Despliegue](#10-despliegue)
11. [Ejemplo práctico completo](#11-ejemplo-práctico-completo)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Buenas prácticas](#13-buenas-prácticas)
14. [Referencia rápida](#14-referencia-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. Descripción

**python-telegram-bot** (PTB) es la librería más completa para crear bots de Telegram en Python. Envuelve la API oficial de bots de Telegram y añade una arquitectura de manejadores, filtros, conversaciones, tareas programadas y persistencia.

**Casos de uso habituales:**
- Notificaciones automáticas (alertas de servidores, pedidos, precios).
- Asistentes y atención al cliente.
- Bots de consulta (stock, horarios, estado de pedidos).
- Recogida de datos mediante formularios conversacionales.
- Automatización de tareas internas de equipo.

> Desde la versión 20, la librería es **asíncrona** (`async`/`await`). Los ejemplos antiguos con `Updater` y `dispatcher` no funcionan en versiones actuales.

---

## 2. Requisitos previos: crear el bot

1. Abra Telegram y busque **@BotFather**.
2. Envíe `/newbot`.
3. Indique un nombre visible (por ejemplo, `Asistente Mi Empresa`).
4. Indique un nombre de usuario que termine en `bot` (por ejemplo, `miempresa_asistente_bot`).
5. BotFather le entregará un **token** con este formato: `123456789:AAH...xyz`.

Otros comandos útiles de BotFather:

| Comando | Función |
|---|---|
| `/setdescription` | Descripción del bot. |
| `/setabouttext` | Texto «Acerca de». |
| `/setuserpic` | Imagen de perfil. |
| `/setcommands` | Lista de comandos que aparece en el menú. |
| `/revoke` | Genera un token nuevo (si se ha filtrado). |

> **El token es una contraseña.** No lo publique ni lo incluya en el código: guárdelo en una variable de entorno o en un archivo `.env`.

---

## 3. Instalación e importación

```bash
pip install python-telegram-bot python-dotenv
```

Extras opcionales:

```bash
pip install "python-telegram-bot[job-queue]"      # Tareas programadas
pip install "python-telegram-bot[webhooks]"       # Despliegue con webhooks
pip install "python-telegram-bot[all]"            # Todo
```

```python
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, ReplyKeyboardMarkup
from telegram.constants import ParseMode, ChatAction
from telegram.ext import (
    Application, ApplicationBuilder, CommandHandler, MessageHandler,
    CallbackQueryHandler, ConversationHandler, ContextTypes, filters,
)
```

Archivo `.env`:

```
TELEGRAM_TOKEN=123456789:AAH...xyz
```

---

## 4. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Application` | Núcleo del bot: recibe actualizaciones y las reparte a los manejadores. |
| `Update` | Cada evento recibido (mensaje, pulsación de botón, etc.). |
| `Handler` | Asocia un tipo de actualización con una función. |
| `CallbackContext` (`context`) | Acceso al bot, datos de usuario/chat, argumentos y tareas. |
| `filters` | Condiciones para que un manejador se active (texto, foto, comando…). |
| *Polling* | El bot consulta periódicamente a Telegram (ideal para desarrollo). |
| *Webhook* | Telegram envía las actualizaciones a una URL (ideal para producción). |
| `chat_id` | Identificador del chat al que se envían los mensajes. |

**Flujo:** el usuario escribe → Telegram genera un `Update` → `Application` lo recibe → el primer `Handler` cuyo filtro coincide ejecuta su función → la función responde.

---

## 5. Operaciones básicas

### 5.1 Bot mínimo («eco»)

```python
import os
from dotenv import load_dotenv
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, ContextTypes, filters

load_dotenv()
TOKEN = os.environ["TELEGRAM_TOKEN"]


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    usuario = update.effective_user
    await update.message.reply_text(f"Hola, {usuario.first_name}. Soy tu asistente.")


async def ayuda(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("/start – Iniciar\n/ayuda – Ver esta ayuda")


async def eco(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text(f"Has escrito: {update.message.text}")


def main() -> None:
    app = Application.builder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("ayuda", ayuda))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, eco))

    print("Bot en marcha. Pulse Ctrl+C para detenerlo.")
    app.run_polling(allowed_updates=Update.ALL_TYPES)


if __name__ == "__main__":
    main()
```

### 5.2 Información de la actualización

```python
update.effective_user.id          # ID del usuario
update.effective_user.first_name
update.effective_user.username
update.effective_chat.id          # ID del chat (para enviar mensajes)
update.effective_chat.type        # "private", "group", "supergroup", "channel"
update.message.text               # Texto recibido
update.message.date
update.message.message_id
```

### 5.3 Comandos con argumentos

```python
async def sumar(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    # /sumar 5 7 12  →  context.args = ["5", "7", "12"]
    try:
        total = sum(float(x.replace(",", ".")) for x in context.args)
    except ValueError:
        await update.message.reply_text("Uso: /sumar 5 7 12")
        return
    await update.message.reply_text(f"Resultado: {total:g}")

app.add_handler(CommandHandler("sumar", sumar))
```

### 5.4 Filtros más utilizados

| Filtro | Coincide con |
|---|---|
| `filters.TEXT` | Mensajes de texto. |
| `filters.COMMAND` | Comandos (`/algo`). |
| `filters.PHOTO` | Fotos. |
| `filters.Document.ALL` | Documentos. |
| `filters.Document.PDF` | PDF. |
| `filters.VOICE` / `filters.AUDIO` | Notas de voz / audio. |
| `filters.LOCATION` | Ubicaciones. |
| `filters.CONTACT` | Contactos compartidos. |
| `filters.ChatType.PRIVATE` | Chats privados. |
| `filters.ChatType.GROUPS` | Grupos. |
| `filters.User(user_id=[...])` | Usuarios concretos. |
| `filters.Regex(r"patrón")` | Texto que cumple una expresión regular. |

Combinación de filtros: `&` (y), `|` (o), `~` (no).

```python
MessageHandler(filters.TEXT & filters.ChatType.PRIVATE & ~filters.COMMAND, funcion)
MessageHandler(filters.Regex(r"(?i)^(hola|buenas)"), saludar)
```

---

## 6. Enviar contenido

### 6.1 Texto con formato

```python
from telegram.constants import ParseMode

await update.message.reply_text(
    "<b>Pedido confirmado</b>\n"
    "Número: <code>2026-1045</code>\n"
    "Total: <i>59,80 €</i>\n"
    '<a href="https://miempresa.com/pedidos/1045">Ver pedido</a>',
    parse_mode=ParseMode.HTML,
)
```

> Prefiera `HTML` a `MarkdownV2`: este último obliga a escapar muchos caracteres (`.`, `-`, `(`, `!`…). Si el texto contiene datos del usuario, escápelo con `html.escape()`.

### 6.2 Enviar a un chat concreto (sin responder)

```python
await context.bot.send_message(chat_id=123456789, text="Aviso del sistema")
```

### 6.3 Archivos multimedia

```python
await update.message.reply_photo(photo=open("grafico.png", "rb"), caption="Ventas de noviembre")
await update.message.reply_photo(photo="https://ejemplo.com/imagen.jpg")
await update.message.reply_document(document=open("informe.pdf", "rb"), filename="Informe.pdf")
await update.message.reply_audio(audio=open("audio.mp3", "rb"))
await update.message.reply_video(video=open("video.mp4", "rb"))
await update.message.reply_location(latitude=40.4168, longitude=-3.7038)
```

Varias imágenes en un álbum:

```python
from telegram import InputMediaPhoto

album = [InputMediaPhoto(open(f"foto{i}.jpg", "rb")) for i in range(1, 4)]
await context.bot.send_media_group(chat_id=update.effective_chat.id, media=album)
```

### 6.4 Indicador «escribiendo…»

```python
from telegram.constants import ChatAction

await context.bot.send_chat_action(update.effective_chat.id, ChatAction.TYPING)
```

### 6.5 Recibir archivos del usuario

```python
async def recibir_documento(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    documento = update.message.document
    archivo = await documento.get_file()
    await archivo.download_to_drive(f"descargas/{documento.file_name}")
    await update.message.reply_text(f"Recibido: {documento.file_name}")


async def recibir_foto(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    foto = update.message.photo[-1]                  # La de mayor resolución
    archivo = await foto.get_file()
    await archivo.download_to_drive(f"descargas/{foto.file_unique_id}.jpg")


app.add_handler(MessageHandler(filters.Document.ALL, recibir_documento))
app.add_handler(MessageHandler(filters.PHOTO, recibir_foto))
```

### 6.6 Editar y eliminar mensajes

```python
mensaje = await update.message.reply_text("Procesando…")
await mensaje.edit_text("Proceso completado ✅")
await mensaje.delete()
```

---

## 7. Teclados y botones

### 7.1 Teclado de respuesta (sustituye al teclado del móvil)

```python
from telegram import ReplyKeyboardMarkup, ReplyKeyboardRemove

teclado = ReplyKeyboardMarkup(
    [["📦 Mis pedidos", "💶 Tarifas"], ["📞 Contacto"]],
    resize_keyboard=True,
    one_time_keyboard=False,
    input_field_placeholder="Elija una opción",
)
await update.message.reply_text("¿En qué puedo ayudarle?", reply_markup=teclado)

# Ocultarlo
await update.message.reply_text("Teclado ocultado", reply_markup=ReplyKeyboardRemove())
```

Al pulsar un botón se envía su texto como mensaje normal; se gestiona con un `MessageHandler` y `filters.Regex`.

### 7.2 Botones en línea (*inline*)

```python
from telegram import InlineKeyboardButton, InlineKeyboardMarkup

botones = InlineKeyboardMarkup([
    [InlineKeyboardButton("✅ Confirmar", callback_data="confirmar"),
     InlineKeyboardButton("❌ Cancelar", callback_data="cancelar")],
    [InlineKeyboardButton("🌐 Abrir web", url="https://miempresa.com")],
])
await update.message.reply_text("¿Confirma el pedido?", reply_markup=botones)
```

### 7.3 Gestionar la pulsación

```python
from telegram.ext import CallbackQueryHandler

async def boton(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    consulta = update.callback_query
    await consulta.answer()                         # Obligatorio: quita el indicador de carga
    if consulta.data == "confirmar":
        await consulta.edit_message_text("Pedido confirmado ✅")
    else:
        await consulta.edit_message_text("Pedido cancelado")

app.add_handler(CallbackQueryHandler(boton))
```

Filtrar por patrón:

```python
app.add_handler(CallbackQueryHandler(ver_producto, pattern=r"^producto:"))
# callback_data="producto:A01"  →  consulta.data.split(":")[1]
```

> `callback_data` admite un máximo de **64 bytes**.

### 7.4 Menú de comandos desde el código

```python
from telegram import BotCommand

async def configurar_menu(app: Application) -> None:
    await app.bot.set_my_commands([
        BotCommand("start", "Iniciar el bot"),
        BotCommand("pedido", "Consultar un pedido"),
        BotCommand("ayuda", "Mostrar ayuda"),
    ])

app = Application.builder().token(TOKEN).post_init(configurar_menu).build()
```

---

## 8. Conversaciones guiadas

`ConversationHandler` permite crear formularios paso a paso mediante **estados**.

```python
from telegram.ext import ConversationHandler

NOMBRE, EMAIL, CONFIRMAR = range(3)


async def alta_inicio(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    await update.message.reply_text("Vamos a registrarle. ¿Cuál es su nombre?\n(/cancelar para salir)")
    return NOMBRE


async def alta_nombre(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    context.user_data["nombre"] = update.message.text.strip()
    await update.message.reply_text("Gracias. Indique su correo electrónico:")
    return EMAIL


async def alta_email(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    email = update.message.text.strip()
    if "@" not in email:
        await update.message.reply_text("El correo no parece válido. Inténtelo de nuevo:")
        return EMAIL                                   # Repite el mismo estado
    context.user_data["email"] = email
    teclado = ReplyKeyboardMarkup([["Sí", "No"]], one_time_keyboard=True, resize_keyboard=True)
    await update.message.reply_text(
        f"Nombre: {context.user_data['nombre']}\nEmail: {email}\n¿Es correcto?",
        reply_markup=teclado,
    )
    return CONFIRMAR


async def alta_confirmar(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    if update.message.text == "Sí":
        # guardar_en_bd(context.user_data)
        await update.message.reply_text("Registro completado ✅", reply_markup=ReplyKeyboardRemove())
    else:
        await update.message.reply_text("Registro descartado.", reply_markup=ReplyKeyboardRemove())
    context.user_data.clear()
    return ConversationHandler.END


async def cancelar(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    context.user_data.clear()
    await update.message.reply_text("Operación cancelada.", reply_markup=ReplyKeyboardRemove())
    return ConversationHandler.END


conversacion = ConversationHandler(
    entry_points=[CommandHandler("alta", alta_inicio)],
    states={
        NOMBRE: [MessageHandler(filters.TEXT & ~filters.COMMAND, alta_nombre)],
        EMAIL: [MessageHandler(filters.TEXT & ~filters.COMMAND, alta_email)],
        CONFIRMAR: [MessageHandler(filters.Regex(r"^(Sí|No)$"), alta_confirmar)],
    },
    fallbacks=[CommandHandler("cancelar", cancelar)],
    conversation_timeout=300,               # Se cancela tras 5 minutos de inactividad
)
app.add_handler(conversacion)
```

### 8.1 Almacenamiento de datos en `context`

| Atributo | Ámbito |
|---|---|
| `context.user_data` | Datos de cada usuario. |
| `context.chat_data` | Datos de cada chat. |
| `context.bot_data` | Datos compartidos por todo el bot. |

---

## 9. Operaciones avanzadas

### 9.1 Tareas programadas (`JobQueue`)

Requiere `pip install "python-telegram-bot[job-queue]"`.

```python
import datetime as dt
from zoneinfo import ZoneInfo


async def recordatorio(context: ContextTypes.DEFAULT_TYPE) -> None:
    await context.bot.send_message(context.job.chat_id, text=f"⏰ {context.job.data}")


async def recordar(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    # /recordar 10 Revisar el correo
    try:
        minutos = int(context.args[0])
        texto = " ".join(context.args[1:]) or "Recordatorio"
    except (IndexError, ValueError):
        await update.message.reply_text("Uso: /recordar <minutos> <texto>")
        return
    context.job_queue.run_once(recordatorio, when=minutos * 60,
                               chat_id=update.effective_chat.id, data=texto)
    await update.message.reply_text(f"Le avisaré en {minutos} minutos.")


# Tarea diaria a las 9:00 (hora de Madrid)
app.job_queue.run_daily(
    resumen_diario,
    time=dt.time(9, 0, tzinfo=ZoneInfo("Europe/Madrid")),
    days=(0, 1, 2, 3, 4),                   # Lunes a viernes (0 = lunes)
    chat_id=ID_CHAT_EQUIPO,
)

# Tarea repetitiva cada 15 minutos
app.job_queue.run_repeating(comprobar_servidor, interval=900, first=10)
```

### 9.2 Restringir el acceso

```python
USUARIOS_AUTORIZADOS = {123456789, 987654321}

app.add_handler(CommandHandler(
    "admin", panel_admin, filters=filters.User(user_id=USUARIOS_AUTORIZADOS)
))
```

O mediante un decorador:

```python
from functools import wraps

def solo_autorizados(func):
    @wraps(func)
    async def envoltorio(update: Update, context: ContextTypes.DEFAULT_TYPE, *args, **kwargs):
        if update.effective_user.id not in USUARIOS_AUTORIZADOS:
            await update.effective_message.reply_text("⛔ No tiene permiso para esta acción.")
            return
        return await func(update, context, *args, **kwargs)
    return envoltorio
```

### 9.3 Gestión global de errores

```python
import logging

logging.basicConfig(format="%(asctime)s | %(levelname)s | %(name)s | %(message)s", level=logging.INFO)
logging.getLogger("httpx").setLevel(logging.WARNING)       # Evita registrar el token en las URLs
logger = logging.getLogger(__name__)


async def manejar_error(update: object, context: ContextTypes.DEFAULT_TYPE) -> None:
    logger.error("Error al procesar una actualización", exc_info=context.error)
    if isinstance(update, Update) and update.effective_message:
        await update.effective_message.reply_text("Se ha producido un error. Inténtelo más tarde.")

app.add_error_handler(manejar_error)
```

### 9.4 Persistencia de datos entre reinicios

```python
from telegram.ext import PicklePersistence

persistencia = PicklePersistence(filepath="datos_bot.pickle")
app = Application.builder().token(TOKEN).persistence(persistencia).build()

# Las conversaciones también pueden persistir:
ConversationHandler(..., name="alta", persistent=True)
```

### 9.5 Enviar mensajes desde otro script (notificaciones)

```python
import asyncio
from telegram import Bot

async def notificar(texto: str) -> None:
    async with Bot(TOKEN) as bot:
        await bot.send_message(chat_id=ID_CHAT, text=texto)

asyncio.run(notificar("✅ Copia de seguridad completada"))
```

> Para conocer su `chat_id`, envíe un mensaje al bot y muestre `update.effective_chat.id` en un manejador.

### 9.6 Grupos

- Por defecto, en grupos el bot solo recibe comandos y respuestas a sus mensajes (*modo privacidad*).
- Para leer todos los mensajes: BotFather → `/setprivacy` → *Disable*, o hacerlo administrador.
- En grupos, los comandos pueden llegar como `/comando@nombre_bot`; PTB los gestiona automáticamente.

---

## 10. Despliegue

### 10.1 Polling (simple)

Válido en cualquier equipo o servidor con salida a Internet:

```python
app.run_polling()
```

### 10.2 Webhook (producción)

Requiere un dominio con HTTPS:

```python
app.run_webhook(
    listen="0.0.0.0",
    port=8443,
    url_path="telegram",
    webhook_url="https://bot.miempresa.com/telegram",
    secret_token="cadena-secreta-aleatoria",
)
```

### 10.3 Ejecución como servicio (Linux, systemd)

```ini
# /etc/systemd/system/bot.service
[Unit]
Description=Bot de Telegram
After=network-online.target

[Service]
WorkingDirectory=/opt/bot
EnvironmentFile=/opt/bot/.env
ExecStart=/opt/bot/.venv/bin/python bot.py
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now bot
```

---

## 11. Ejemplo práctico completo

Bot de consulta de pedidos con menú, botones en línea, búsqueda, restricción para administradores, recordatorio diario y gestión de errores.

```python
import datetime as dt
import html
import logging
import os
from zoneinfo import ZoneInfo

from dotenv import load_dotenv
from telegram import (
    BotCommand, InlineKeyboardButton, InlineKeyboardMarkup,
    ReplyKeyboardMarkup, Update,
)
from telegram.constants import ParseMode
from telegram.ext import (
    Application, CallbackQueryHandler, CommandHandler,
    ContextTypes, MessageHandler, filters,
)

load_dotenv()
TOKEN = os.environ["TELEGRAM_TOKEN"]
ADMINS = {int(x) for x in os.getenv("TELEGRAM_ADMINS", "").split(",") if x.strip()}

logging.basicConfig(format="%(asctime)s | %(levelname)s | %(message)s", level=logging.INFO)
logging.getLogger("httpx").setLevel(logging.WARNING)
log = logging.getLogger("bot")

# Datos de ejemplo (en un caso real, una base de datos o una API)
PEDIDOS = {
    "1045": {"cliente": "Ana López", "estado": "Enviado", "total": 59.80, "entrega": "26/11/2026"},
    "1046": {"cliente": "Luis Gómez", "estado": "En preparación", "total": 189.00, "entrega": "28/11/2026"},
    "1047": {"cliente": "Marta Ruiz", "estado": "Entregado", "total": 24.50, "entrega": "22/11/2026"},
}
ICONOS = {"Enviado": "🚚", "En preparación": "📦", "Entregado": "✅"}

MENU = ReplyKeyboardMarkup(
    [["🔎 Consultar pedido", "📋 Últimos pedidos"], ["📞 Contacto", "❓ Ayuda"]],
    resize_keyboard=True,
)


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


def ficha(numero: str) -> str:
    p = PEDIDOS[numero]
    return (
        f"<b>Pedido {numero}</b>\n"
        f"Cliente: {html.escape(p['cliente'])}\n"
        f"Estado: {ICONOS.get(p['estado'], '')} {p['estado']}\n"
        f"Total: {euros(p['total'])}\n"
        f"Entrega prevista: {p['entrega']}"
    )


# ---------- Manejadores ----------
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    nombre = html.escape(update.effective_user.first_name)
    await update.message.reply_text(
        f"Hola, <b>{nombre}</b> 👋\nSoy el asistente de pedidos. Elija una opción:",
        parse_mode=ParseMode.HTML, reply_markup=MENU,
    )


async def ayuda(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text(
        "Comandos disponibles:\n"
        "/pedido <número> – Consultar un pedido\n"
        "/ultimos – Ver los últimos pedidos\n"
        "/contacto – Datos de contacto\n"
        "/estadisticas – Resumen (solo administradores)",
    )


async def pedido(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if not context.args:
        context.user_data["esperando_pedido"] = True
        await update.message.reply_text("Indique el número de pedido:")
        return
    await mostrar_pedido(update, context.args[0])


async def mostrar_pedido(update: Update, numero: str) -> None:
    numero = numero.strip().lstrip("#")
    if numero not in PEDIDOS:
        await update.effective_message.reply_text(f"No existe ningún pedido con el número {html.escape(numero)}.")
        return
    botones = InlineKeyboardMarkup([[
        InlineKeyboardButton("🔄 Actualizar", callback_data=f"ver:{numero}"),
        InlineKeyboardButton("📞 Incidencia", callback_data=f"incidencia:{numero}"),
    ]])
    await update.effective_message.reply_text(ficha(numero), parse_mode=ParseMode.HTML,
                                              reply_markup=botones)


async def ultimos(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    botones = InlineKeyboardMarkup([
        [InlineKeyboardButton(f"{ICONOS.get(p['estado'], '')} {n} · {p['cliente']}", callback_data=f"ver:{n}")]
        for n, p in sorted(PEDIDOS.items(), reverse=True)
    ])
    await update.effective_message.reply_text("Últimos pedidos:", reply_markup=botones)


async def contacto(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.effective_message.reply_text(
        "📞 900 123 456 (L-V, 9:00-18:00)\n✉️ soporte@miempresa.com",
    )


async def estadisticas(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    total = sum(p["total"] for p in PEDIDOS.values())
    por_estado: dict[str, int] = {}
    for p in PEDIDOS.values():
        por_estado[p["estado"]] = por_estado.get(p["estado"], 0) + 1
    lineas = "\n".join(f"{ICONOS.get(e, '')} {e}: {n}" for e, n in por_estado.items())
    await update.message.reply_text(
        f"<b>Resumen</b>\nPedidos: {len(PEDIDOS)}\nFacturación: {euros(total)}\n\n{lineas}",
        parse_mode=ParseMode.HTML,
    )


async def botones(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    consulta = update.callback_query
    await consulta.answer()
    accion, numero = consulta.data.split(":", 1)
    if accion == "ver":
        await mostrar_pedido(update, numero)
    elif accion == "incidencia":
        await consulta.message.reply_text(
            f"Hemos registrado una incidencia para el pedido {numero}. "
            "Nuestro equipo le contactará en breve."
        )
        for admin in ADMINS:
            await context.bot.send_message(
                admin, f"⚠️ Incidencia en el pedido {numero} "
                       f"(usuario {update.effective_user.id})")


async def texto_libre(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    texto = update.message.text
    if context.user_data.pop("esperando_pedido", False):
        await mostrar_pedido(update, texto)
    elif texto == "🔎 Consultar pedido":
        context.user_data["esperando_pedido"] = True
        await update.message.reply_text("Indique el número de pedido:")
    elif texto == "📋 Últimos pedidos":
        await ultimos(update, context)
    elif texto == "📞 Contacto":
        await contacto(update, context)
    elif texto == "❓ Ayuda":
        await ayuda(update, context)
    elif texto.lstrip("#").isdigit():
        await mostrar_pedido(update, texto)
    else:
        await update.message.reply_text("No he entendido el mensaje. Use el menú o /ayuda.")


async def resumen_diario(context: ContextTypes.DEFAULT_TYPE) -> None:
    pendientes = [n for n, p in PEDIDOS.items() if p["estado"] != "Entregado"]
    for admin in ADMINS:
        await context.bot.send_message(admin, f"☀️ Buenos días. Pedidos pendientes: {len(pendientes)}")


async def error(update: object, context: ContextTypes.DEFAULT_TYPE) -> None:
    log.error("Error no controlado", exc_info=context.error)
    if isinstance(update, Update) and update.effective_message:
        await update.effective_message.reply_text("Se ha producido un error. Inténtelo de nuevo más tarde.")


async def post_init(app: Application) -> None:
    await app.bot.set_my_commands([
        BotCommand("start", "Menú principal"),
        BotCommand("pedido", "Consultar un pedido"),
        BotCommand("ultimos", "Últimos pedidos"),
        BotCommand("contacto", "Datos de contacto"),
        BotCommand("ayuda", "Ayuda"),
    ])


def main() -> None:
    app = Application.builder().token(TOKEN).post_init(post_init).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("ayuda", ayuda))
    app.add_handler(CommandHandler("pedido", pedido))
    app.add_handler(CommandHandler("ultimos", ultimos))
    app.add_handler(CommandHandler("contacto", contacto))
    app.add_handler(CommandHandler("estadisticas", estadisticas, filters=filters.User(user_id=ADMINS)))
    app.add_handler(CallbackQueryHandler(botones, pattern=r"^(ver|incidencia):"))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, texto_libre))
    app.add_error_handler(error)

    if app.job_queue and ADMINS:
        app.job_queue.run_daily(resumen_diario, time=dt.time(9, 0, tzinfo=ZoneInfo("Europe/Madrid")),
                                days=(0, 1, 2, 3, 4))

    log.info("Bot iniciado")
    app.run_polling(allowed_updates=Update.ALL_TYPES)


if __name__ == "__main__":
    main()
```

Archivo `.env`:

```
TELEGRAM_TOKEN=123456789:AAH...xyz
TELEGRAM_ADMINS=111111111,222222222
```

---

## 12. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `ImportError: cannot import name 'Updater'` / código con `dispatcher` | Ejemplos de la versión 13 o anterior. | Use `Application.builder()` y funciones `async`. |
| `RuntimeWarning: coroutine ... was never awaited` | Falta `await`. | Anteponga `await` a todas las llamadas a la API. |
| `telegram.error.InvalidToken` | Token incorrecto o vacío. | Revise la variable de entorno. |
| `telegram.error.Conflict: terminated by other getUpdates request` | Dos instancias del bot en ejecución. | Detenga la otra instancia. |
| `BadRequest: Can't parse entities` | HTML/Markdown mal formado. | Escape los datos con `html.escape()`. |
| `BadRequest: Message is not modified` | Se edita un mensaje con el mismo contenido. | Compruebe si hay cambios antes de editar. |
| `Forbidden: bot was blocked by the user` | El usuario bloqueó el bot. | Capture la excepción y elimine al usuario de la lista de envíos. |
| `AttributeError: 'NoneType' object has no attribute 'run_once'` | `job_queue` no instalado. | `pip install "python-telegram-bot[job-queue]"`. |
| El bot no responde en grupos | Modo privacidad activo. | Desactívelo en BotFather (`/setprivacy`). |
| Botones con indicador de carga infinito | Falta `await query.answer()`. | Llame siempre a `answer()` en los callbacks. |

---

## 13. Buenas prácticas

- Guarde el token en variables de entorno y **nunca** lo suba a un repositorio.
- Reduzca el nivel de log de `httpx` para no registrar el token en las URLs.
- Valide y escape siempre los datos introducidos por el usuario.
- Registre un manejador global de errores.
- Responda siempre a los `callback_query` con `answer()`.
- Respete los límites de Telegram: aprox. **30 mensajes por segundo** en total y **1 por segundo** por chat; para envíos masivos, introduzca pausas.
- Use `ConversationHandler` para flujos de varios pasos en lugar de gestionar estados manualmente.
- Use *polling* en desarrollo y *webhooks* en producción con alto volumen.
- Restrinja las funciones administrativas por ID de usuario.

---

## 14. Referencia rápida

```python
app = Application.builder().token(TOKEN).post_init(func).build()
app.add_handler(CommandHandler("cmd", func))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, func))
app.add_handler(CallbackQueryHandler(func, pattern=r"^prefijo:"))
app.add_handler(ConversationHandler(entry_points=[...], states={...}, fallbacks=[...]))
app.add_error_handler(func)
app.run_polling() | app.run_webhook(...)

async def func(update: Update, context: ContextTypes.DEFAULT_TYPE): ...
await update.message.reply_text(txt, parse_mode=ParseMode.HTML, reply_markup=...)
await context.bot.send_message(chat_id, txt) / send_photo / send_document
context.args / context.user_data / context.chat_data / context.bot_data
context.job_queue.run_once / run_repeating / run_daily
InlineKeyboardMarkup([[InlineKeyboardButton(txt, callback_data=...)]])
ReplyKeyboardMarkup([["A", "B"]], resize_keyboard=True)
```

---

## 15. Ejercicios propuestos

1. Cree un bot que devuelva el tiempo de una ciudad consultando una API pública con `httpx`.
2. Implemente un bot de lista de tareas por usuario con comandos `/nueva`, `/lista` y `/hecha <n>`, con persistencia.
3. Desarrolle un formulario de solicitud de vacaciones con `ConversationHandler` que notifique al responsable con botones de aprobar/rechazar.
4. Escriba un script que envíe al chat del equipo un resumen diario a las 8:30 con los datos de un CSV.
5. Construya un bot que reciba facturas en PDF y las guarde en carpetas por mes.

---

## 16. Referencias

- Documentación oficial: https://docs.python-telegram-bot.org/
- Wiki y ejemplos: https://github.com/python-telegram-bot/python-telegram-bot/wiki
- API de bots de Telegram: https://core.telegram.org/bots/api
