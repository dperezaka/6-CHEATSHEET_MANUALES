# Manual de uso: Pillow

> **Categoría:** Imágenes · **Utilidad:** Abrir, editar, crear y convertir imágenes
> **Tipo:** Librería de terceros (`pip install pillow`) · **Versión de referencia:** 10.0 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Abrir, inspeccionar y guardar](#3-abrir-inspeccionar-y-guardar)
4. [Modos de color y conversión](#4-modos-de-color-y-conversión)
5. [Redimensionar, recortar y girar](#5-redimensionar-recortar-y-girar)
6. [`ImageOps`: operaciones de alto nivel](#6-imageops-operaciones-de-alto-nivel)
7. [Ajustes de color con `ImageEnhance`](#7-ajustes-de-color-con-imageenhance)
8. [Filtros con `ImageFilter`](#8-filtros-con-imagefilter)
9. [Dibujo con `ImageDraw`](#9-dibujo-con-imagedraw)
10. [Texto y fuentes](#10-texto-y-fuentes)
11. [Transparencia, máscaras y composición](#11-transparencia-máscaras-y-composición)
12. [Operaciones por píxel y con NumPy](#12-operaciones-por-píxel-y-con-numpy)
13. [Metadatos EXIF](#13-metadatos-exif)
14. [Formatos: JPEG, PNG, WebP, AVIF, GIF, TIFF y PDF](#14-formatos-jpeg-png-webp-avif-gif-tiff-y-pdf)
15. [Imágenes animadas y multipágina](#15-imágenes-animadas-y-multipágina)
16. [Procesamiento por lotes](#16-procesamiento-por-lotes)
17. [Integración con otras librerías](#17-integración-con-otras-librerías)
18. [Seguridad](#18-seguridad)
19. [Ejemplo práctico completo](#19-ejemplo-práctico-completo)
20. [Errores frecuentes](#20-errores-frecuentes)
21. [Buenas prácticas](#21-buenas-prácticas)
22. [Referencia rápida](#22-referencia-rápida)
23. [Ejercicios propuestos](#23-ejercicios-propuestos)
24. [Referencias](#24-referencias)

---

## 1. Descripción

**Pillow** es la librería estándar de facto para trabajar con imágenes en Python (sucesora de PIL, *Python Imaging Library*). Permite abrir y guardar decenas de formatos, redimensionar, recortar, ajustar color, aplicar filtros, dibujar, escribir texto con fuentes TrueType, componer imágenes con transparencia y leer metadatos.

**Ventajas:**
- API sencilla y pythónica.
- Amplio soporte de formatos (JPEG, PNG, WebP, AVIF, GIF, TIFF, BMP, ICO, PDF…).
- Ligera y estable; dependencia de muchas otras librerías (Matplotlib, ReportLab, Django, scikit-image…).
- Texto con fuentes TrueType/OpenType y caracteres Unicode completos.

**Limitaciones:**
- No es una librería de visión artificial (detección, vídeo, geometría avanzada): para eso, OpenCV.
- Operaciones por píxel en Python puro son lentas: use NumPy o las funciones integradas.
- No edita formatos vectoriales (SVG) ni RAW de cámara (use `rawpy`).

| Si necesita… | Opción |
|---|---|
| Redimensionar, convertir, marcas de agua, miniaturas, texto | **Pillow** |
| Visión artificial y vídeo | OpenCV |
| Algoritmos científicos de imagen | scikit-image |
| Procesamiento de imágenes muy grandes y rápido | pyvips |
| Generar gráficos de datos | Matplotlib |

---

## 2. Instalación e importación

```bash
pip install pillow
```

El paquete se instala como `pillow`, pero se importa como **`PIL`**:

```python
from PIL import Image, ImageOps, ImageEnhance, ImageFilter, ImageDraw, ImageFont, ExifTags
from PIL import features
features.check("webp"), features.check("avif"), features.check("raqm")    # Soporte disponible
```

> No instale a la vez `PIL` (antiguo, abandonado) y `pillow`.

---

## 3. Abrir, inspeccionar y guardar

```python
from PIL import Image

with Image.open("foto.jpg") as img:          # Carga perezosa: lee la cabecera; los píxeles al usarlos
    print(img.format, img.size, img.mode)    # 'JPEG', (ancho, alto), 'RGB'
    ancho, alto = img.size
    img.info                                 # Metadatos del formato (dpi, icc_profile, exif…)
    img.load()                               # Forzar lectura completa si se cierra el archivo
    img.save("foto.png")                     # Formato deducido de la extensión
    img.save("foto.webp", quality=80)

img = Image.new("RGB", (800, 600), color="#1F4E78")      # Imagen nueva
img = Image.new("RGBA", (400, 400), (0, 0, 0, 0))        # Transparente
img.show()                                               # Abre el visor del sistema (para pruebas)
copia = img.copy()

Image.open(io.BytesIO(datos_bytes))                       # Desde memoria (p. ej., una descarga)
buffer = io.BytesIO(); img.save(buffer, format="PNG"); datos = buffer.getvalue()
```

> Use `with Image.open(...)` o llame a `img.close()`: evita archivos bloqueados (sobre todo en Windows) al procesar muchas imágenes.

---

## 4. Modos de color y conversión

| Modo | Descripción |
|---|---|
| `1` | Blanco y negro (1 bit). |
| `L` | Escala de grises (8 bits). |
| `LA` | Gris con transparencia. |
| `P` | Paleta (hasta 256 colores; GIF, PNG indexado). |
| `RGB` | Color verdadero. |
| `RGBA` | Color con canal alfa (transparencia). |
| `CMYK` | Impresión. |
| `I;16`, `I`, `F` | Enteros de 16/32 bits y coma flotante (imagen científica). |

```python
gris = img.convert("L")
rgb = img.convert("RGB")                        # Necesario antes de guardar RGBA/P como JPEG
bn = gris.point(lambda v: 255 if v > 128 else 0, mode="1")      # Umbral
paleta = img.convert("P", palette=Image.Palette.ADAPTIVE, colors=64)

r, g, b = img.split()                            # Canales
img = Image.merge("RGB", (b, g, r))

def a_rgb(img: Image.Image, fondo=(255, 255, 255)) -> Image.Image:
    """Aplana la transparencia sobre un fondo de color (para JPEG)."""
    if img.mode in ("RGBA", "LA", "P"):
        img = img.convert("RGBA")
        base = Image.new("RGB", img.size, fondo)
        base.paste(img, mask=img.getchannel("A"))
        return base
    return img.convert("RGB")
```

---

## 5. Redimensionar, recortar y girar

```python
from PIL import Image

pequena = img.resize((400, 300), Image.Resampling.LANCZOS)      # Tamaño exacto (puede deformar)
img.thumbnail((400, 400))                                       # En el sitio, mantiene proporción, solo reduce
mitad = img.reduce(2)                                           # Reducción rápida por factor entero

recorte = img.crop((izquierda, arriba, derecha, abajo))         # Caja (x0, y0, x1, y1)
girada = img.rotate(30, expand=True, fillcolor="white", resample=Image.Resampling.BICUBIC)
volteada = img.transpose(Image.Transpose.FLIP_LEFT_RIGHT)       # También ROTATE_90, ROTATE_270…
```

| Filtro (`Image.Resampling`) | Uso |
|---|---|
| `NEAREST` | *Pixel art*, máscaras, máxima velocidad. |
| `BILINEAR`, `BICUBIC` | Uso general. |
| `LANCZOS` | Máxima calidad al reducir (miniaturas, web). |
| `BOX`, `HAMMING` | Reducciones rápidas con buena calidad. |

> `Image.ANTIALIAS` se eliminó en Pillow 10: use `Image.Resampling.LANCZOS`.

---

## 6. `ImageOps`: operaciones de alto nivel

```python
from PIL import ImageOps

ImageOps.exif_transpose(img)                             # Aplica la orientación EXIF (fotos de móvil)
ImageOps.contain(img, (800, 800))                        # Cabe dentro, mantiene proporción
ImageOps.cover(img, (800, 800))                          # Cubre el área (puede sobresalir)
ImageOps.fit(img, (600, 600), centering=(0.5, 0.4))      # Recorta para rellenar exactamente (miniatura cuadrada)
ImageOps.pad(img, (1200, 630), color="white")            # Encaja con bandas de relleno
ImageOps.expand(img, border=20, fill="black")            # Marco
ImageOps.grayscale(img) ; ImageOps.invert(img.convert("RGB")) ; ImageOps.mirror(img) ; ImageOps.flip(img)
ImageOps.autocontrast(img, cutoff=1)                     # Estira el histograma por canal (descarta el 1 % extremo)
ImageOps.autocontrast(img, cutoff=1, preserve_tone=True) # Sobre la luminancia: no altera los tonos (fotos de producto)
ImageOps.equalize(img) ; ImageOps.posterize(img, 3) ; ImageOps.solarize(img, 128)
ImageOps.colorize(img.convert("L"), black="#0B2545", white="#EEF4ED")   # Monocromo a dos colores
```

| Función | Resultado para 1000×500 → (600, 600) |
|---|---|
| `contain` | 600×300 (cabe completa). |
| `cover` | 1200×600 (cubre el área). |
| `fit` | 600×600 recortada. |
| `pad` | 600×600 con bandas. |

---

## 7. Ajustes de color con `ImageEnhance`

```python
from PIL import ImageEnhance

img = ImageEnhance.Brightness(img).enhance(1.15)   # 1.0 = sin cambio; <1 oscurece
img = ImageEnhance.Contrast(img).enhance(1.2)
img = ImageEnhance.Color(img).enhance(0.0)         # Saturación: 0 = blanco y negro
img = ImageEnhance.Sharpness(img).enhance(2.0)     # Nitidez
```

---

## 8. Filtros con `ImageFilter`

```python
from PIL import ImageFilter

img.filter(ImageFilter.GaussianBlur(radius=4))
img.filter(ImageFilter.BoxBlur(3))
img.filter(ImageFilter.UnsharpMask(radius=2, percent=150, threshold=3))   # Enfoque profesional
img.filter(ImageFilter.MedianFilter(3))                                    # Elimina ruido impulsivo
img.filter(ImageFilter.FIND_EDGES) ; img.filter(ImageFilter.CONTOUR) ; img.filter(ImageFilter.EMBOSS)
img.filter(ImageFilter.Kernel((3, 3), [0, -1, 0, -1, 5, -1, 0, -1, 0]))    # Núcleo propio
img.filter(ImageFilter.MaxFilter(5))                                       # Dilatación (máscaras)
```

---

## 9. Dibujo con `ImageDraw`

```python
from PIL import ImageDraw

lienzo = Image.new("RGB", (600, 400), "white")
d = ImageDraw.Draw(lienzo)                         # Para semitransparencias: Draw(img, "RGBA")
d.line([(20, 20), (580, 20)], fill="#1F4E78", width=3)
d.rectangle((20, 40, 200, 140), outline="black", width=2, fill="#DDEBF7")
d.rounded_rectangle((220, 40, 400, 140), radius=16, fill="#2E75B6")
d.ellipse((420, 40, 520, 140), fill="#F4B183", outline="#C55A11", width=3)
d.polygon([(60, 300), (120, 180), (180, 300)], fill="#A9D18E")
d.regular_polygon((300, 250, 50), n_sides=6, rotation=30, fill="#FFD966")
d.arc((380, 180, 500, 300), start=0, end=270, fill="#7030A0", width=8)
d.pieslice((500, 200, 590, 290), start=-90, end=45, fill="#C00000")
d.point([(10, 390), (12, 390)], fill="red")
```

Las coordenadas son `(x, y)` desde la esquina superior izquierda; las cajas, `(x0, y0, x1, y1)`.

---

## 10. Texto y fuentes

```python
from PIL import ImageFont

fuente = ImageFont.truetype("DejaVuSans-Bold.ttf", size=36)       # Busca también en las carpetas del sistema
fuente_def = ImageFont.load_default(size=24)                       # Fuente integrada escalable (Pillow ≥ 10.1)

d.text((20, 20), "Descuento del 20 % en artículos de oficina", font=fuente, fill="black")
d.text((300, 200), "Centrado", font=fuente, fill="white", anchor="mm")    # Anclaje: centro-centro
d.text((20, 80), "Con contorno", font=fuente, fill="white", stroke_width=3, stroke_fill="black")
d.multiline_text((20, 140), "Línea 1\nLínea 2", font=fuente, spacing=8, align="center")

caja = d.textbbox((0, 0), "Texto", font=fuente)      # (x0, y0, x1, y1) → ancho = x1 - x0
ancho = d.textlength("Texto", font=fuente)
```

Ajustar un texto largo a un ancho máximo:

```python
def partir_texto(draw, texto: str, fuente, ancho_max: int) -> list[str]:
    lineas, actual = [], ""
    for palabra in texto.split():
        prueba = f"{actual} {palabra}".strip()
        if draw.textlength(prueba, font=fuente) <= ancho_max:
            actual = prueba
        else:
            lineas.append(actual)
            actual = palabra
    return lineas + [actual] if actual else lineas
```

| Anclaje | Significado |
|---|---|
| `la` (defecto) | Izquierda, ascendente (parte superior del texto). |
| `mm` | Centro horizontal y vertical. |
| `rs` | Derecha, línea base. |
| `ms` | Centro, línea base. |

> `draw.textsize()` y `font.getsize()` se eliminaron en Pillow 10: use `textbbox`, `textlength` o `font.getbbox`.

---

## 11. Transparencia, máscaras y composición

```python
fondo = Image.open("fondo.jpg").convert("RGBA")
logo = Image.open("logo.png").convert("RGBA")

fondo.paste(logo, (20, 20), mask=logo)              # Pegar respetando la transparencia del logo
fondo.alpha_composite(logo, dest=(20, 20))          # Composición alfa correcta (ambas RGBA)
mezcla = Image.blend(img_a, img_b, alpha=0.3)       # Mismo tamaño y modo
compuesta = Image.composite(img_a, img_b, mascara)  # Máscara L: 255 → img_a, 0 → img_b

# Esquinas redondeadas
mascara = Image.new("L", img.size, 0)
ImageDraw.Draw(mascara).rounded_rectangle((0, 0, *img.size), radius=40, fill=255)
redondeada = img.convert("RGBA"); redondeada.putalpha(mascara)

# Marca de agua semitransparente
capa = Image.new("RGBA", img.size, (0, 0, 0, 0))
ImageDraw.Draw(capa).text((img.width - 20, img.height - 20), "© MiEmpresa", font=fuente,
                          fill=(255, 255, 255, 110), anchor="rs")
con_marca = Image.alpha_composite(img.convert("RGBA"), capa)

# Sombra
sombra = Image.new("RGBA", (w + 40, h + 40), (0, 0, 0, 0))
sombra.paste((0, 0, 0, 120), (20, 25, w + 20, h + 25))
sombra = sombra.filter(ImageFilter.GaussianBlur(10))
```

---

## 12. Operaciones por píxel y con NumPy

```python
img.getpixel((10, 20)) ; img.putpixel((10, 20), (255, 0, 0))     # Lentos: solo para pocos píxeles
img.point(lambda v: min(255, int(v * 1.2)))                       # Tabla de transformación (rápido)
img.getextrema() ; img.histogram() ; img.getbbox()                # getbbox: caja del contenido no nulo
img.getcolors(maxcolors=256)

from PIL import ImageChops, ImageStat
diferencia = ImageChops.difference(img_a, img_b)                  # Comparar imágenes
sin_bordes = img.crop(ImageChops.difference(img, Image.new(img.mode, img.size, "white")).getbbox())   # Recortar márgenes
ImageStat.Stat(img).mean                                          # Media por canal

import numpy as np
arr = np.asarray(img)                     # (alto, ancho, canales), uint8, solo lectura
arr = np.array(img)                       # Copia modificable
arr[..., 0] = 0                           # Quitar el canal rojo
img2 = Image.fromarray(arr)
```

---

## 13. Metadatos EXIF

```python
from PIL import ExifTags

exif = img.getexif()
for etiqueta_id, valor in exif.items():
    print(ExifTags.TAGS.get(etiqueta_id, etiqueta_id), valor)

exif.get(ExifTags.Base.Orientation)                     # 1 normal; 3, 6, 8 = girada
exif.get(ExifTags.Base.Model) ; exif.get(ExifTags.Base.DateTime)
datos_foto = exif.get_ifd(ExifTags.IFD.Exif)            # Exposición, ISO, fecha original…
gps = exif.get_ifd(ExifTags.IFD.GPSInfo)                # Coordenadas (si existen)

exif[ExifTags.Base.Copyright] = "© 2026 MiEmpresa"
img.save("con_exif.jpg", exif=exif)
img.save("sin_metadatos.jpg")                           # Sin pasar exif= → se eliminan (privacidad: GPS)
```

> Las fotos de móvil suelen guardarse «tumbadas» con una etiqueta de orientación. Aplique `ImageOps.exif_transpose(img)` **antes** de redimensionar o recortar.

---

## 14. Formatos: JPEG, PNG, WebP, AVIF, GIF, TIFF y PDF

```python
img.save("foto.jpg", quality=85, optimize=True, progressive=True, subsampling="4:2:0")
img.save("grafico.png", optimize=True)                         # Sin pérdida
img.save("web.webp", quality=80, method=6)                     # method 0–6: más lento = más pequeño
img.save("sin_perdida.webp", lossless=True)
img.save("moderno.avif", quality=60)                           # Pillow ≥ 11.2 (con soporte AVIF)
img.save("escaneo.tif", compression="tiff_lzw", dpi=(300, 300))
img.save("icono.ico", sizes=[(16, 16), (32, 32), (48, 48), (256, 256)])
img.convert("RGB").save("documento.pdf", resolution=150)
```

| Formato | Transparencia | Compresión | Uso |
|---|---|---|---|
| JPEG | No | Con pérdida | Fotografías. |
| PNG | Sí | Sin pérdida | Gráficos, capturas, logotipos. |
| WebP | Sí | Ambas | Web (≈ 25–35 % menor que JPEG). |
| AVIF | Sí | Ambas | Web moderna (aún menor que WebP). |
| GIF | Sí (1 color) | Paleta 256 colores | Animaciones sencillas. |
| TIFF | Sí | Varias | Archivo, impresión, escaneo. |

---

## 15. Imágenes animadas y multipágina

```python
from PIL import ImageSequence

with Image.open("animacion.gif") as gif:
    print(gif.n_frames, gif.info.get("duration"))
    fotogramas = [f.convert("RGBA").copy() for f in ImageSequence.Iterator(gif)]

fotogramas[0].save("salida.gif", save_all=True, append_images=fotogramas[1:],
                   duration=80, loop=0, optimize=True, disposal=2)      # loop=0: infinito
fotogramas[0].save("salida.webp", save_all=True, append_images=fotogramas[1:], duration=80, loop=0)

paginas[0].save("informe.pdf", save_all=True, append_images=paginas[1:])     # PDF multipágina
paginas[0].save("archivo.tif", save_all=True, append_images=paginas[1:], compression="tiff_lzw")
```

---

## 16. Procesamiento por lotes

```python
from concurrent.futures import ProcessPoolExecutor
from pathlib import Path

def procesar(ruta: Path) -> str:
    with Image.open(ruta) as img:
        img = ImageOps.exif_transpose(img)
        img.thumbnail((1600, 1600), Image.Resampling.LANCZOS)
        destino = Path("web") / f"{ruta.stem}.webp"
        img.save(destino, quality=82)
    return destino.name

if __name__ == "__main__":
    Path("web").mkdir(exist_ok=True)
    rutas = [p for p in Path("originales").iterdir() if p.suffix.lower() in {".jpg", ".jpeg", ".png"}]
    with ProcessPoolExecutor() as ejecutor:                       # Aprovecha todos los núcleos
        for nombre in ejecutor.map(procesar, rutas):
            print("✔", nombre)
```

Para imágenes JPEG muy grandes, `img.draft("RGB", (2000, 2000))` antes de cargar decodifica a resolución reducida, mucho más rápido.

---

## 17. Integración con otras librerías

```python
import cv2, numpy as np
bgr = cv2.cvtColor(np.array(img.convert("RGB")), cv2.COLOR_RGB2BGR)      # Pillow → OpenCV
img = Image.fromarray(cv2.cvtColor(bgr, cv2.COLOR_BGR2RGB))             # OpenCV → Pillow

import matplotlib.pyplot as plt ; plt.imshow(img)                        # Matplotlib acepta imágenes Pillow

from PIL import ImageGrab
captura = ImageGrab.grab(bbox=(0, 0, 800, 600))                          # Captura de pantalla (Windows/macOS/Linux X11)
portapapeles = ImageGrab.grabclipboard()

from PIL import ImageTk                                                  # Mostrar en Tkinter
foto_tk = ImageTk.PhotoImage(img)
```

---

## 18. Seguridad

- Pillow protege contra **bombas de descompresión** (imágenes diminutas en disco con dimensiones enormes): emite `DecompressionBombWarning` a partir de ~89 megapíxeles y lanza `DecompressionBombError` al doble. Ajuste `Image.MAX_IMAGE_PIXELS` solo si confía en el origen.
- Con imágenes subidas por usuarios: verifique el formato real (`img.format`), limite tamaño de archivo y dimensiones, use `img.verify()` para detectar archivos corruptos (y vuelva a abrir después) y vuelva a guardar la imagen (elimina contenido embebido y metadatos).
- Mantenga Pillow actualizado: muchas versiones corrigen vulnerabilidades de los decodificadores.

---

## 19. Ejemplo práctico completo

**Procesador de imágenes de catálogo de comercio electrónico**. A partir de una carpeta de fotografías de producto (el script genera unas de muestra, incluida una con orientación EXIF girada y otra PNG con transparencia), produce:

- Imagen **principal** normalizada 1200×1200 (orientación corregida, fondo blanco, autocontraste suave, enfoque y marca de agua).
- **Miniaturas** cuadradas recortadas de forma inteligente, en WebP y JPEG.
- **Tarjeta para redes sociales** 1200×630 con degradado, esquinas redondeadas, sombra, precio y descripción ajustada al ancho.
- **Hoja de contactos** con todas las miniaturas y sus nombres, un **GIF animado** de presentación y un **PDF** multipágina.
- Un informe CSV con dimensiones originales, orientación EXIF, tamaños de archivo y ahorro de peso, procesando las imágenes en paralelo.

```python
"""
catalogo_pillow.py — Normaliza fotografías de producto y genera recursos para web, redes y PDF.
Requiere: pip install pillow
Uso: python catalogo_pillow.py [carpeta_originales]
"""
from __future__ import annotations

import csv
import sys
from concurrent.futures import ProcessPoolExecutor
from dataclasses import asdict, dataclass
from pathlib import Path

from PIL import ExifTags, Image, ImageDraw, ImageEnhance, ImageFilter, ImageFont, ImageOps

SALIDA = Path("catalogo_web")
CARPETAS = {n: SALIDA / n for n in ("principal", "miniaturas", "redes")}
AZUL, AZUL_CLARO, NARANJA = (31, 78, 120), (46, 117, 182), (230, 126, 34)
EXTENSIONES = {".jpg", ".jpeg", ".png", ".webp", ".tif", ".tiff"}

PRODUCTOS = {   # Datos que normalmente vendrían de una base de datos o un CSV
    "auriculares": ("Auriculares inalámbricos Pro", 59.90, "Cancelación activa de ruido y 30 horas de autonomía con estuche de carga rápida."),
    "lampara": ("Lámpara LED de escritorio", 32.00, "Brazo articulado, tres temperaturas de color y regulación táctil de intensidad."),
    "taza": ("Taza térmica de acero", 14.90, "Mantiene el café caliente durante seis horas. Tapa hermética apta para lavavajillas."),
    "mochila": ("Mochila urbana impermeable", 49.00, "Compartimento acolchado para portátil de 15,6 pulgadas y puerto USB integrado."),
}


def fuente(tamano: int, negrita: bool = False) -> ImageFont.FreeTypeFont:
    candidatas = ["DejaVuSans-Bold.ttf", "arialbd.ttf", "Arial Bold.ttf"] if negrita else ["DejaVuSans.ttf", "arial.ttf", "Arial.ttf"]
    for nombre in candidatas:
        try:
            return ImageFont.truetype(nombre, tamano)
        except OSError:
            continue
    return ImageFont.load_default(size=tamano)


def euros(v: float) -> str:
    return f"{v:,.2f} €".replace(",", "X").replace(".", ",").replace("X", ".")


# ============================================================ Imágenes de muestra
def generar_muestras(carpeta: Path) -> None:
    carpeta.mkdir(parents=True, exist_ok=True)
    colores = {"auriculares": (52, 73, 94), "lampara": (241, 196, 15), "taza": (192, 57, 43), "mochila": (39, 174, 96)}
    for i, (nombre, color) in enumerate(colores.items()):
        ancho, alto = (2400, 1600) if i % 2 == 0 else (1500, 2100)
        img = Image.new("RGB", (ancho, alto), (236, 232, 226))
        d = ImageDraw.Draw(img)
        for y in range(alto):                                            # Fondo de estudio con degradado
            v = int(236 - 40 * y / alto)
            d.line([(0, y), (ancho, y)], fill=(v, v - 3, v - 8))
        cx, cy, r = ancho // 2 + 150, alto // 2, min(ancho, alto) // 4
        d.ellipse((cx - r, cy + r - 40, cx + r, cy + r + 60), fill=(170, 165, 160))            # Sombra
        d.rounded_rectangle((cx - r, cy - r, cx + r, cy + r), radius=r // 3, fill=color)        # «Producto»
        d.ellipse((cx - r // 2, cy - r // 2, cx + r // 3, cy + r // 4), fill=tuple(min(255, c + 60) for c in color))
        img = ImageEnhance.Contrast(img).enhance(0.75)                   # Foto «apagada»
        exif = Image.Exif()
        exif[ExifTags.Base.Model] = "Camara de estudio X100"
        exif[ExifTags.Base.DateTime] = f"2026:11:0{i + 1} 10:30:00"
        if nombre == "lampara":                                          # Simula foto de móvil girada
            img = img.transpose(Image.Transpose.ROTATE_90)
            exif[ExifTags.Base.Orientation] = 6
        if nombre == "taza":                                             # PNG con fondo transparente
            mascara = Image.new("L", img.size, 0)
            ImageDraw.Draw(mascara).rounded_rectangle((cx - r, cy - r, cx + r, cy + r), radius=r // 3, fill=255)
            img = img.convert("RGBA")
            img.putalpha(mascara)
            img.save(carpeta / f"{nombre}.png")
        else:
            img.save(carpeta / f"{nombre}.jpg", quality=95, exif=exif)


# ============================================================ Procesamiento de una imagen
@dataclass
class Resultado:
    archivo: str
    formato: str
    tamano_original: str
    orientacion_exif: int
    kb_original: float
    kb_principal: float
    kb_miniatura_webp: float
    ahorro: str


def aplanar(img: Image.Image, fondo=(255, 255, 255)) -> Image.Image:
    if img.mode in ("RGBA", "LA", "P"):
        img = img.convert("RGBA")
        base = Image.new("RGB", img.size, fondo)
        base.paste(img, mask=img.getchannel("A"))
        return base
    return img.convert("RGB")


def recortar_margenes(img: Image.Image, tolerancia: int = 25) -> Image.Image:
    """Recorta el fondo uniforme alrededor del producto (comparando con el color de la esquina)."""
    fondo = Image.new("RGB", img.size, img.getpixel((0, 0)))
    from PIL import ImageChops
    diff = ImageChops.difference(img, fondo).convert("L").point(lambda v: 255 if v > tolerancia else 0)
    caja = diff.getbbox()
    if not caja:
        return img
    margen = int(0.06 * max(img.size))
    x0, y0, x1, y1 = caja
    return img.crop((max(0, x0 - margen), max(0, y0 - margen), min(img.width, x1 + margen), min(img.height, y1 + margen)))


def marca_agua(img: Image.Image, texto: str) -> Image.Image:
    capa = Image.new("RGBA", img.size, (0, 0, 0, 0))
    ImageDraw.Draw(capa).text((img.width - 30, img.height - 25), texto, font=fuente(28, True),
                              fill=(255, 255, 255, 150), anchor="rs", stroke_width=2, stroke_fill=(0, 0, 0, 90))
    return Image.alpha_composite(img.convert("RGBA"), capa).convert("RGB")


def procesar(ruta: Path) -> Resultado:
    with Image.open(ruta) as original:
        formato, tamano = original.format, original.size
        orientacion = original.getexif().get(ExifTags.Base.Orientation, 1)
        img = ImageOps.exif_transpose(original)                          # 1. Orientación correcta
        img = aplanar(img)                                               # 2. Transparencia → fondo blanco

    img = recortar_margenes(img)                                         # 3. Encuadre centrado en el producto
    img = ImageOps.autocontrast(img, cutoff=0.5, preserve_tone=True)    # 4. Corrección tonal sin alterar el color
    img = ImageEnhance.Color(img).enhance(1.1)
    principal = ImageOps.pad(img, (1200, 1200), method=Image.Resampling.LANCZOS, color="white")
    principal = principal.filter(ImageFilter.UnsharpMask(radius=1.5, percent=80, threshold=2))
    principal = marca_agua(principal, "© TiendaDemo")
    destino = CARPETAS["principal"] / f"{ruta.stem}.jpg"
    principal.save(destino, quality=85, optimize=True, progressive=True)   # Sin exif=: metadatos eliminados

    mini = ImageOps.fit(img, (400, 400), method=Image.Resampling.LANCZOS, centering=(0.5, 0.45))
    mini.save(CARPETAS["miniaturas"] / f"{ruta.stem}.webp", quality=80, method=6)
    mini.save(CARPETAS["miniaturas"] / f"{ruta.stem}.jpg", quality=82, optimize=True)

    kb = lambda p: round(p.stat().st_size / 1024, 1)
    kb_orig, kb_princ = kb(ruta), kb(destino)
    return Resultado(ruta.name, formato, f"{tamano[0]}x{tamano[1]}", orientacion, kb_orig, kb_princ,
                     kb(CARPETAS["miniaturas"] / f"{ruta.stem}.webp"), f"{1 - kb_princ / kb_orig:.0%}")


# ============================================================ Recursos derivados
def partir_texto(draw: ImageDraw.ImageDraw, texto: str, f, ancho_max: int) -> list[str]:
    lineas, actual = [], ""
    for palabra in texto.split():
        prueba = f"{actual} {palabra}".strip()
        if draw.textlength(prueba, font=f) <= ancho_max:
            actual = prueba
        else:
            lineas.append(actual)
            actual = palabra
    return lineas + [actual] if actual else lineas


def tarjeta_redes(clave: str) -> Path:
    nombre, precio, descripcion = PRODUCTOS[clave]
    lienzo = Image.new("RGB", (1200, 630))
    d = ImageDraw.Draw(lienzo)
    for x in range(1200):                                               # Degradado horizontal
        t = x / 1199
        d.line([(x, 0), (x, 630)], fill=tuple(int(a + (b - a) * t) for a, b in zip(AZUL, AZUL_CLARO)))

    foto = Image.open(CARPETAS["principal"] / f"{clave}.jpg").resize((480, 480), Image.Resampling.LANCZOS)
    mascara = Image.new("L", foto.size, 0)
    ImageDraw.Draw(mascara).rounded_rectangle((0, 0, *foto.size), radius=36, fill=255)
    sombra = Image.new("RGBA", (560, 560), (0, 0, 0, 0))
    ImageDraw.Draw(sombra).rounded_rectangle((40, 50, 520, 530), radius=36, fill=(0, 0, 0, 110))
    sombra = sombra.filter(ImageFilter.GaussianBlur(16))
    lienzo.paste(sombra, (30, 35), sombra)
    lienzo.paste(foto, (70, 75), mascara)

    x = 610
    titulo = fuente(46, True)
    y = 95
    for linea in partir_texto(d, nombre, titulo, 540):
        d.text((x, y), linea, font=titulo, fill="white")
        y += 58
    y += 10
    for linea in partir_texto(d, descripcion, fuente(26), 540):
        d.text((x, y), linea, font=fuente(26), fill=(220, 232, 245))
        y += 36
    etiqueta_precio = euros(precio)
    f_precio = fuente(54, True)
    ancho_precio = d.textlength(etiqueta_precio, font=f_precio)
    d.rounded_rectangle((x, 455, x + ancho_precio + 50, 535), radius=40, fill=NARANJA)
    d.text((x + 25 + ancho_precio / 2, 495), etiqueta_precio, font=f_precio, fill="white", anchor="mm")
    d.text((1170, 600), "tiendademo.es", font=fuente(22), fill=(200, 215, 235), anchor="rs")
    destino = CARPETAS["redes"] / f"{clave}_1200x630.png"
    lienzo.save(destino, optimize=True)
    return destino


def hoja_contactos(columnas: int = 4, lado: int = 260) -> Path:
    miniaturas = sorted(CARPETAS["miniaturas"].glob("*.jpg"))
    filas = -(-len(miniaturas) // columnas)
    margen, pie = 24, 40
    hoja = Image.new("RGB", (columnas * (lado + margen) + margen, filas * (lado + pie + margen) + margen + 60), "white")
    d = ImageDraw.Draw(hoja)
    d.text((margen, 20), f"Hoja de contactos · {len(miniaturas)} productos", font=fuente(26, True), fill=AZUL)
    for i, ruta in enumerate(miniaturas):
        x = margen + (i % columnas) * (lado + margen)
        y = 70 + (i // columnas) * (lado + pie + margen)
        with Image.open(ruta) as m:
            hoja.paste(ImageOps.expand(m.resize((lado - 2, lado - 2)), border=1, fill=(200, 200, 200)), (x, y))
        d.text((x + lado / 2, y + lado + 22), PRODUCTOS.get(ruta.stem, (ruta.stem,))[0][:28],
               font=fuente(16), fill="black", anchor="mm")
    destino = SALIDA / "hoja_contactos.png"
    hoja.save(destino)
    return destino


def gif_y_pdf() -> tuple[Path, Path]:
    tarjetas = [Image.open(p).convert("RGB") for p in sorted(CARPETAS["redes"].glob("*.png"))]
    fotogramas = []
    for actual, siguiente in zip(tarjetas, tarjetas[1:] + tarjetas[:1]):
        pequena, sig = actual.resize((600, 315)), siguiente.resize((600, 315))
        fotogramas += [pequena] * 6 + [Image.blend(pequena, sig, a) for a in (0.25, 0.5, 0.75)]   # Fundido
    gif = SALIDA / "presentacion.gif"
    paletizados = [f.convert("P", palette=Image.Palette.ADAPTIVE, colors=128) for f in fotogramas]
    paletizados[0].save(gif, save_all=True, append_images=paletizados[1:], duration=120, loop=0, optimize=True)

    paginas = [Image.open(p).convert("RGB") for p in sorted(CARPETAS["principal"].glob("*.jpg"))]
    pdf = SALIDA / "catalogo.pdf"
    paginas[0].save(pdf, save_all=True, append_images=paginas[1:], resolution=150)
    return gif, pdf


def main() -> int:
    origen = Path(sys.argv[1]) if len(sys.argv) > 1 else Path("originales")
    if not origen.exists():
        print(f"Generando imágenes de muestra en «{origen}»…")
        generar_muestras(origen)
    for carpeta in CARPETAS.values():
        carpeta.mkdir(parents=True, exist_ok=True)

    rutas = sorted(p for p in origen.iterdir() if p.suffix.lower() in EXTENSIONES)
    with ProcessPoolExecutor() as ejecutor:                               # En paralelo (un proceso por núcleo)
        resultados = list(ejecutor.map(procesar, rutas))

    print(f"\n{'Archivo':<16}{'Formato':<8}{'Original':>12}{'EXIF':>6}{'KB orig.':>10}{'KB web':>9}{'Ahorro':>8}")
    for r in resultados:
        print(f"{r.archivo:<16}{r.formato:<8}{r.tamano_original:>12}{r.orientacion_exif:>6}"
              f"{r.kb_original:>10.1f}{r.kb_principal:>9.1f}{r.ahorro:>8}")
    with open(SALIDA / "informe.csv", "w", newline="", encoding="utf-8-sig") as f:
        escritor = csv.DictWriter(f, fieldnames=list(asdict(resultados[0])), delimiter=";")
        escritor.writeheader()
        escritor.writerows(asdict(r) for r in resultados)

    for clave in (p.stem for p in rutas if p.stem in PRODUCTOS):
        tarjeta_redes(clave)
    contactos = hoja_contactos()
    gif, pdf = gif_y_pdf()
    print(f"\n✔ Tarjetas para redes: {len(list(CARPETAS['redes'].glob('*.png')))}")
    print(f"✔ {contactos.name}, {gif.name}, {pdf.name} e informe.csv en «{SALIDA}»")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Notas sobre el resultado:

- La foto con orientación EXIF 6 se guarda «tumbada» (2100×1500) y sale correctamente en vertical gracias a `exif_transpose`.
- Un ahorro negativo es normal cuando el original es un PNG muy simple (pocos colores): el formato sin pérdida comprime mejor que JPEG ese tipo de imagen. Para gráficos y logotipos conserve PNG o use WebP sin pérdida.
- `autocontrast(..., preserve_tone=True)` corrige el contraste sin desplazar los colores del producto, algo crítico en comercio electrónico.

---

## 20. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `OSError: cannot write mode RGBA as JPEG` | JPEG no admite transparencia. | `img.convert("RGB")` o aplanar sobre un fondo. |
| `AttributeError: module 'PIL.Image' has no attribute 'ANTIALIAS'` | Eliminado en Pillow 10. | `Image.Resampling.LANCZOS`. |
| `AttributeError: 'ImageDraw' object has no attribute 'textsize'` | Eliminado en Pillow 10. | `textbbox()` o `textlength()`. |
| `OSError: cannot open resource` | No se encuentra la fuente TrueType. | Ruta completa a la fuente o una fuente instalada; alternativa `load_default(size=...)`. |
| `PIL.UnidentifiedImageError: cannot identify image file` | Archivo corrupto, formato no soportado o no es una imagen. | Compruebe el archivo y el soporte (`features.check`). |
| Fotos de móvil giradas tras procesarlas | Se ignoró la orientación EXIF. | `ImageOps.exif_transpose(img)` al abrir. |
| `ValueError: images do not match` | `blend`/`composite`/`paste` con tamaños o modos distintos. | Igualar tamaño y modo antes. |
| `ValueError: bad transparency mask` | Máscara de modo o tamaño incorrecto. | Máscara en modo `L` o `1`, o RGBA, del mismo tamaño. |
| `DecompressionBombWarning/Error` | Imagen con demasiados píxeles. | Reduzca en origen o ajuste `Image.MAX_IMAGE_PIXELS` si la fuente es fiable. |
| Archivo bloqueado o «Too many open files» | No se cierran las imágenes. | `with Image.open(...)`. |
| GIF con colores pobres o bandas | Paleta limitada a 256 colores. | `convert("P", palette=ADAPTIVE)`, menos degradados, o WebP animado. |
| `ModuleNotFoundError: No module named 'PIL'` | No se instaló `pillow`. | `pip install pillow`. |

---

## 21. Buenas prácticas

- Abra las imágenes con `with` y aplique `exif_transpose` antes de cualquier transformación geométrica.
- Mantenga los originales intactos y genere derivados en otra carpeta.
- Reduzca con `LANCZOS` y aplique un enfoque suave (`UnsharpMask`) tras reducir mucho.
- Elija el formato según el uso: JPEG/WebP/AVIF para fotos, PNG para gráficos y transparencias.
- Elimine metadatos (en especial GPS) en las imágenes que publique.
- Use `ImageOps.fit/pad/contain` en lugar de calcular proporciones a mano.
- Para lotes grandes, paralelice con `ProcessPoolExecutor` y considere `draft()` con JPEG.
- Incluya fuentes TrueType con su aplicación si genera texto (no dependa de las del sistema).

---

## 22. Referencia rápida

```python
Image.open(ruta) · Image.new(modo, (w, h), color) · img.save(ruta, quality=, optimize=) · img.size/.mode/.format
img.convert("RGB"|"L"|"RGBA"|"P") · img.resize((w, h), Image.Resampling.LANCZOS) · img.thumbnail((w, h)) · img.crop((x0, y0, x1, y1))
img.rotate(grados, expand=True) · img.transpose(Image.Transpose.FLIP_LEFT_RIGHT) · img.paste(otra, (x, y), mask)
ImageOps.exif_transpose · fit · pad · contain · cover · expand · autocontrast · grayscale · colorize
ImageEnhance.Brightness/Contrast/Color/Sharpness(img).enhance(f) · img.filter(ImageFilter.GaussianBlur(r)|UnsharpMask())
d = ImageDraw.Draw(img) · d.rectangle/rounded_rectangle/ellipse/line/polygon · d.text((x, y), t, font=, fill=, anchor="mm")
ImageFont.truetype(ruta, tamaño) · ImageFont.load_default(size=) · d.textbbox · d.textlength
Image.alpha_composite(a, b) · Image.blend(a, b, alpha) · Image.composite(a, b, mascara) · img.putalpha(mascara)
img.getexif() · ExifTags.Base.Orientation · np.asarray(img) · Image.fromarray(arr)
img.save(ruta, save_all=True, append_images=[...], duration=, loop=0)   # GIF/WebP animado, PDF/TIFF multipágina
```

---

## 23. Ejercicios propuestos

1. Añada al procesador la generación de versiones responsivas (400, 800 y 1600 px) en WebP y AVIF, y produzca el fragmento HTML `<picture>` correspondiente.
2. Cree un generador de etiquetas de precio imprimibles (A4 con 3×8 etiquetas) que lea los productos de un CSV.
3. Implemente un detector de imágenes duplicadas mediante *hash* perceptivo (reducir a 8×8 en gris y comparar con la media).
4. Construya un collage automático para Instagram (1080×1080) con 4 fotos y un título superpuesto con fondo semitransparente.
5. Escriba una función que anonimice fotos: elimine todos los metadatos y pixele una región indicada.

---

## 24. Referencias

- Documentación oficial: https://pillow.readthedocs.io/
- Tutorial: https://pillow.readthedocs.io/en/stable/handbook/tutorial.html
- Formatos de archivo soportados: https://pillow.readthedocs.io/en/stable/handbook/image-file-formats.html
- Notas de versión (cambios y eliminaciones): https://pillow.readthedocs.io/en/stable/releasenotes/index.html
