# Manual de uso: Pillow (PIL)

> **Categoría:** Utilidades generales · **Utilidad:** Editar y procesar imágenes
> **Tipo:** Librería externa

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Operaciones básicas](#4-operaciones-básicas)
5. [Transformaciones geométricas](#5-transformaciones-geométricas)
6. [Color, filtros y ajustes](#6-color-filtros-y-ajustes)
7. [Dibujo y texto](#7-dibujo-y-texto)
8. [Composición de imágenes](#8-composición-de-imágenes)
9. [Operaciones avanzadas](#9-operaciones-avanzadas)
10. [Ejemplo práctico completo](#10-ejemplo-práctico-completo)
11. [Errores frecuentes](#11-errores-frecuentes)
12. [Buenas prácticas](#12-buenas-prácticas)
13. [Referencia rápida](#13-referencia-rápida)
14. [Ejercicios propuestos](#14-ejercicios-propuestos)
15. [Referencias](#15-referencias)

---

## 1. Descripción

**Pillow** es la versión moderna y mantenida de PIL (*Python Imaging Library*). Permite abrir, crear, editar y guardar imágenes en múltiples formatos.

**Casos de uso habituales:**
- Redimensionar y optimizar imágenes para web.
- Generar miniaturas y recortes.
- Añadir marcas de agua, textos y logotipos.
- Convertir entre formatos (JPEG, PNG, WebP, GIF, TIFF, BMP, ICO…).
- Crear imágenes desde cero (banners, tarjetas, gráficos sencillos).
- Procesar imágenes por lotes.
- Leer metadatos EXIF de fotografías.

---

## 2. Instalación e importación

```bash
pip install pillow
```

El paquete se llama `pillow`, pero se importa como `PIL`:

```python
from PIL import (
    Image, ImageOps, ImageFilter, ImageEnhance,
    ImageDraw, ImageFont, ImageChops, ImageColor, ExifTags,
)
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| `Image` | Objeto que representa una imagen en memoria. |
| `size` | `(ancho, alto)` en píxeles. |
| `mode` | Modo de color (ver tabla). |
| `format` | Formato del archivo de origen (`JPEG`, `PNG`…). |
| Coordenadas | `(x, y)`; origen `(0, 0)` en la esquina superior izquierda. |
| Caja (*box*) | Rectángulo `(izquierda, arriba, derecha, abajo)`. |

### 3.1 Modos de color

| Modo | Descripción |
|---|---|
| `1` | Blanco y negro (1 bit). |
| `L` | Escala de grises (8 bits). |
| `P` | Paleta de colores (GIF, PNG indexado). |
| `RGB` | Color verdadero (3 × 8 bits). |
| `RGBA` | RGB con canal de transparencia. |
| `CMYK` | Impresión. |

---

## 4. Operaciones básicas

### 4.1 Abrir una imagen

```python
img = Image.open("foto.jpg")
print(img.size)      # (4032, 3024)
print(img.mode)      # RGB
print(img.format)    # JPEG
img.show()           # Abre el visor del sistema
```

Con gestor de contexto (libera el archivo al terminar):

```python
with Image.open("foto.jpg") as img:
    img.load()
    ancho, alto = img.size
```

### 4.2 Crear una imagen nueva

```python
lienzo = Image.new("RGB", (800, 600), color="white")
transparente = Image.new("RGBA", (400, 400), (0, 0, 0, 0))
color_hex = Image.new("RGB", (100, 100), "#1F4E78")
```

### 4.3 Guardar y convertir formato

```python
img.save("foto.png")                                   # Formato por extensión
img.save("foto.jpg", quality=85, optimize=True, progressive=True)
img.save("foto.webp", quality=80, method=6)
img.save("icono.ico", sizes=[(16, 16), (32, 32), (64, 64)])
img.save("salida", format="PNG")                       # Formato explícito
```

| Parámetro | Formatos | Efecto |
|---|---|---|
| `quality` (1–95) | JPEG, WebP | Calidad de compresión. 80–85 es un buen equilibrio. |
| `optimize=True` | JPEG, PNG | Reduce tamaño sin pérdida adicional. |
| `progressive=True` | JPEG | Carga progresiva en web. |
| `lossless=True` | WebP | Compresión sin pérdida. |

### 4.4 Copiar

```python
copia = img.copy()
```

---

## 5. Transformaciones geométricas

### 5.1 Redimensionar

```python
img.resize((800, 600))                                  # Tamaño exacto (puede deformar)
img.resize((800, 600), Image.Resampling.LANCZOS)        # Máxima calidad

img.reduce(2)                                           # Mitad de tamaño (rápido)
```

### 5.2 Mantener la proporción

```python
# Modifica la imagen en el sitio; nunca la amplía
miniatura = img.copy()
miniatura.thumbnail((400, 400))

# Calcular manualmente un nuevo ancho
nuevo_ancho = 1200
proporcion = nuevo_ancho / img.width
img.resize((nuevo_ancho, int(img.height * proporcion)), Image.Resampling.LANCZOS)
```

### 5.3 Ajustar a un tamaño exacto

```python
ImageOps.fit(img, (500, 500))                    # Recorta lo que sobra (centrado)
ImageOps.contain(img, (500, 500))                # Cabe dentro sin recortar
ImageOps.pad(img, (500, 500), color="white")     # Cabe dentro y rellena el resto
ImageOps.cover(img, (500, 500))                  # Cubre el área (puede sobresalir)
```

### 5.4 Recortar

```python
recorte = img.crop((100, 50, 600, 450))          # (izq, arriba, der, abajo)

# Recorte centrado
w, h = img.size
lado = min(w, h)
cuadrado = img.crop(((w - lado) // 2, (h - lado) // 2, (w + lado) // 2, (h + lado) // 2))
```

### 5.5 Rotar y voltear

```python
img.rotate(90)                                   # Antihorario, mantiene tamaño
img.rotate(30, expand=True, fillcolor="white")   # Amplía el lienzo para no cortar
img.transpose(Image.Transpose.ROTATE_270)        # Rotación exacta sin pérdida
img.transpose(Image.Transpose.FLIP_LEFT_RIGHT)   # Espejo horizontal
img.transpose(Image.Transpose.FLIP_TOP_BOTTOM)   # Espejo vertical

ImageOps.mirror(img)
ImageOps.flip(img)
```

### 5.6 Corregir la orientación de fotos de móvil

```python
img = ImageOps.exif_transpose(img)
```

> Los móviles guardan la orientación en los metadatos EXIF. Sin esta corrección, muchas fotos aparecen giradas.

---

## 6. Color, filtros y ajustes

### 6.1 Conversión de modo

```python
gris = img.convert("L")
rgb = img.convert("RGB")            # Necesario antes de guardar RGBA como JPEG
rgba = img.convert("RGBA")
bn = img.convert("1")               # Blanco y negro
paleta = img.convert("P", palette=Image.Palette.ADAPTIVE, colors=64)
```

### 6.2 Filtros predefinidos

```python
img.filter(ImageFilter.BLUR)
img.filter(ImageFilter.GaussianBlur(radius=4))
img.filter(ImageFilter.BoxBlur(3))
img.filter(ImageFilter.SHARPEN)
img.filter(ImageFilter.UnsharpMask(radius=2, percent=150, threshold=3))
img.filter(ImageFilter.DETAIL)
img.filter(ImageFilter.EDGE_ENHANCE)
img.filter(ImageFilter.FIND_EDGES)
img.filter(ImageFilter.CONTOUR)
img.filter(ImageFilter.EMBOSS)
img.filter(ImageFilter.SMOOTH)
img.filter(ImageFilter.MedianFilter(3))       # Reduce ruido
```

### 6.3 Ajustes de imagen

```python
ImageEnhance.Brightness(img).enhance(1.2)     # Brillo
ImageEnhance.Contrast(img).enhance(1.3)       # Contraste
ImageEnhance.Color(img).enhance(1.5)          # Saturación
ImageEnhance.Sharpness(img).enhance(2.0)      # Nitidez
```

> Factor `1.0` = sin cambios; `< 1` reduce; `> 1` aumenta.

### 6.4 Operaciones con `ImageOps`

```python
ImageOps.grayscale(img)
ImageOps.invert(img.convert("RGB"))
ImageOps.autocontrast(img, cutoff=2)
ImageOps.equalize(img)
ImageOps.posterize(img.convert("RGB"), 3)
ImageOps.solarize(img, threshold=128)
ImageOps.colorize(img.convert("L"), black="#2B1B0E", white="#F4E1C1")   # Efecto sepia
ImageOps.expand(img, border=20, fill="white")
```

### 6.5 Acceso a píxeles y canales

```python
img.getpixel((10, 20))                 # (R, G, B)
img.putpixel((10, 20), (255, 0, 0))

r, g, b = img.split()                  # Separar canales
Image.merge("RGB", (b, g, r))          # Intercambiar canales

img.point(lambda p: 255 if p > 128 else 0)     # Aplicar función a cada píxel

img.getcolors(maxcolors=256)
img.histogram()
img.getextrema()                       # Mínimos y máximos por canal
```

---

## 7. Dibujo y texto

### 7.1 Formas básicas

```python
img = Image.new("RGB", (600, 400), "white")
d = ImageDraw.Draw(img)

d.line([(20, 20), (580, 20)], fill="black", width=3)
d.rectangle([(40, 50), (240, 180)], outline="navy", width=4)
d.rounded_rectangle([(280, 50), (560, 180)], radius=20, fill="#DCE6F1", outline="#1F4E78", width=2)
d.ellipse([(40, 220), (200, 370)], fill="orange")
d.polygon([(300, 370), (400, 220), (500, 370)], fill="seagreen")
d.arc([(420, 220), (580, 370)], start=0, end=270, fill="red", width=5)
d.point([(10, 10), (12, 12)], fill="black")
```

### 7.2 Texto con fuentes

```python
try:
    fuente = ImageFont.truetype("arial.ttf", 36)        # Windows
except OSError:
    fuente = ImageFont.load_default(size=36)           # Pillow ≥ 10.1

d.text((50, 50), "Informe mensual", font=fuente, fill="black")
d.text((300, 200), "Centrado", font=fuente, fill="black", anchor="mm")   # Anclado al centro
d.multiline_text((50, 120), "Línea 1\nLínea 2", font=fuente, fill="gray", spacing=8)
```

Rutas de fuentes habituales:

| Sistema | Ruta de ejemplo |
|---|---|
| Windows | `C:/Windows/Fonts/arial.ttf` |
| macOS | `/System/Library/Fonts/Supplemental/Arial.ttf` |
| Linux | `/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf` |

### 7.3 Medir el texto

```python
izq, arriba, der, abajo = d.textbbox((0, 0), "Texto", font=fuente)
ancho_texto = der - izq
alto_texto = abajo - arriba

d.textlength("Texto", font=fuente)     # Solo el ancho
```

### 7.4 Texto con contorno

```python
d.text((50, 300), "Oferta", font=fuente, fill="white", stroke_width=3, stroke_fill="black")
```

---

## 8. Composición de imágenes

### 8.1 Pegar una imagen sobre otra

```python
fondo = Image.open("fondo.jpg").convert("RGBA")
logo = Image.open("logo.png").convert("RGBA")
logo.thumbnail((150, 150))

pos = (fondo.width - logo.width - 20, fondo.height - logo.height - 20)
fondo.paste(logo, pos, mask=logo)       # La máscara respeta la transparencia
```

### 8.2 Fusión con transparencia

```python
resultado = Image.alpha_composite(fondo, capa_rgba)        # Ambas RGBA y mismo tamaño
mezcla = Image.blend(img1, img2, alpha=0.5)                # 50 % de cada una
compuesta = Image.composite(img1, img2, mascara_L)
```

### 8.3 Ajustar opacidad de una capa

```python
logo = logo.copy()
alfa = logo.getchannel("A").point(lambda a: int(a * 0.5))   # 50 % de opacidad
logo.putalpha(alfa)
```

### 8.4 Collage / mosaico

```python
def mosaico(rutas, columnas=3, tam=(300, 300), margen=10, fondo="white"):
    imagenes = [ImageOps.fit(Image.open(r), tam) for r in rutas]
    filas = -(-len(imagenes) // columnas)      # División entera hacia arriba
    ancho = columnas * tam[0] + (columnas + 1) * margen
    alto = filas * tam[1] + (filas + 1) * margen
    lienzo = Image.new("RGB", (ancho, alto), fondo)
    for i, im in enumerate(imagenes):
        x = margen + (i % columnas) * (tam[0] + margen)
        y = margen + (i // columnas) * (tam[1] + margen)
        lienzo.paste(im, (x, y))
    return lienzo
```

### 8.5 Máscara circular (foto de perfil)

```python
def recorte_circular(img, diametro=300):
    img = ImageOps.fit(img.convert("RGBA"), (diametro, diametro))
    mascara = Image.new("L", (diametro, diametro), 0)
    ImageDraw.Draw(mascara).ellipse((0, 0, diametro, diametro), fill=255)
    img.putalpha(mascara)
    return img

recorte_circular(Image.open("retrato.jpg")).save("avatar.png")
```

---

## 9. Operaciones avanzadas

### 9.1 Metadatos EXIF

```python
from PIL import ExifTags

with Image.open("foto.jpg") as img:
    exif = img.getexif()
    for id_tag, valor in exif.items():
        nombre = ExifTags.TAGS.get(id_tag, id_tag)
        print(f"{nombre}: {valor}")
```

Eliminar metadatos (privacidad: ubicación GPS, cámara…):

```python
limpia = Image.new(img.mode, img.size)
limpia.putdata(list(img.getdata()))
limpia.save("sin_metadatos.jpg", quality=90)
```

### 9.2 GIF animados

```python
# Leer fotogramas
from PIL import ImageSequence
with Image.open("animacion.gif") as gif:
    fotogramas = [f.copy() for f in ImageSequence.Iterator(gif)]

# Crear un GIF
fotogramas[0].save("nuevo.gif", save_all=True, append_images=fotogramas[1:],
                   duration=100, loop=0)
```

### 9.3 Comparar imágenes

```python
diferencia = ImageChops.difference(img1.convert("RGB"), img2.convert("RGB"))
son_iguales = diferencia.getbbox() is None
```

### 9.4 Recortar bordes vacíos automáticamente

```python
def recortar_bordes(img, color_fondo=(255, 255, 255)):
    fondo = Image.new(img.mode, img.size, color_fondo)
    caja = ImageChops.difference(img, fondo).getbbox()
    return img.crop(caja) if caja else img
```

### 9.5 Integración con NumPy, Matplotlib y OpenCV

```python
import numpy as np

array = np.asarray(img)                  # Imagen → array (alto, ancho, canales)
img = Image.fromarray(array.astype("uint8"))

# OpenCV usa BGR
import cv2
img_pil = Image.fromarray(cv2.cvtColor(img_cv, cv2.COLOR_BGR2RGB))
img_cv = cv2.cvtColor(np.asarray(img_pil), cv2.COLOR_RGB2BGR)
```

### 9.6 Trabajar en memoria (APIs, bases de datos)

```python
import io

buffer = io.BytesIO()
img.save(buffer, format="PNG")
datos_bytes = buffer.getvalue()

img = Image.open(io.BytesIO(datos_bytes))
```

### 9.7 Imágenes de gran tamaño

```python
Image.MAX_IMAGE_PIXELS = None       # Desactiva el límite de seguridad (solo con archivos de confianza)

with Image.open("enorme.tif") as img:
    img.draft("RGB", (2000, 2000))  # Carga reducida (JPEG): mucho más rápido
```

---

## 10. Ejemplo práctico completo

Optimizador de imágenes para web por lotes: corrige orientación, redimensiona, añade marca de agua, elimina metadatos y genera versiones WebP y miniaturas.

```python
from pathlib import Path
from PIL import Image, ImageDraw, ImageFont, ImageOps

ORIGEN = Path("originales")
DESTINO = Path("web")
EXTENSIONES = {".jpg", ".jpeg", ".png", ".webp", ".bmp", ".tif", ".tiff"}

ANCHO_MAX = 1600
TAM_MINIATURA = (400, 400)
TEXTO_MARCA = "© Mi Empresa 2026"


def cargar_fuente(tamano: int) -> ImageFont.ImageFont:
    for ruta in ("arial.ttf", "DejaVuSans.ttf",
                 "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf"):
        try:
            return ImageFont.truetype(ruta, tamano)
        except OSError:
            continue
    return ImageFont.load_default()


def marca_de_agua(img: Image.Image, texto: str) -> Image.Image:
    base = img.convert("RGBA")
    capa = Image.new("RGBA", base.size, (0, 0, 0, 0))
    dibujo = ImageDraw.Draw(capa)
    fuente = cargar_fuente(max(16, base.width // 40))

    izq, arr, der, abj = dibujo.textbbox((0, 0), texto, font=fuente)
    margen = base.width // 50
    pos = (base.width - (der - izq) - margen, base.height - (abj - arr) - margen)

    dibujo.text(pos, texto, font=fuente, fill=(255, 255, 255, 150),
                stroke_width=1, stroke_fill=(0, 0, 0, 120))
    return Image.alpha_composite(base, capa).convert("RGB")


def procesar(ruta: Path) -> tuple[int, int]:
    with Image.open(ruta) as original:
        img = ImageOps.exif_transpose(original)
        if img.mode in ("RGBA", "LA", "P"):
            fondo = Image.new("RGB", img.size, "white")
            fondo.paste(img.convert("RGBA"), mask=img.convert("RGBA").getchannel("A"))
            img = fondo
        else:
            img = img.convert("RGB")

        if img.width > ANCHO_MAX:
            alto = round(img.height * ANCHO_MAX / img.width)
            img = img.resize((ANCHO_MAX, alto), Image.Resampling.LANCZOS)

        img = marca_de_agua(img, TEXTO_MARCA)

        destino = DESTINO / f"{ruta.stem}.webp"
        img.save(destino, "WEBP", quality=80, method=6)     # Sin EXIF: no se copia

        mini = ImageOps.fit(img, TAM_MINIATURA, Image.Resampling.LANCZOS)
        mini.save(DESTINO / "miniaturas" / f"{ruta.stem}.webp", "WEBP", quality=75)

    return ruta.stat().st_size, destino.stat().st_size


def main():
    (DESTINO / "miniaturas").mkdir(parents=True, exist_ok=True)
    total_antes = total_despues = 0

    for ruta in sorted(ORIGEN.iterdir()):
        if ruta.suffix.lower() not in EXTENSIONES:
            continue
        try:
            antes, despues = procesar(ruta)
        except (OSError, ValueError) as e:
            print(f"✗ {ruta.name}: {e}")
            continue
        total_antes += antes
        total_despues += despues
        print(f"✓ {ruta.name:<30} {antes/1024:>8.0f} KB → {despues/1024:>6.0f} KB")

    if total_antes:
        ahorro = 100 * (1 - total_despues / total_antes)
        print(f"\nAhorro total: {ahorro:.1f} %")


if __name__ == "__main__":
    main()
```

---

## 11. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `OSError: cannot write mode RGBA as JPEG` | JPEG no admite transparencia. | Convierta con `.convert("RGB")` o pegue sobre fondo blanco. |
| `ModuleNotFoundError: No module named 'PIL'` | Pillow no instalado. | `pip install pillow`. |
| `OSError: cannot open resource` | Fuente `.ttf` no encontrada. | Indique la ruta completa o use `load_default()`. |
| Foto girada tras procesarla | Orientación EXIF ignorada. | Use `ImageOps.exif_transpose()`. |
| `thumbnail()` devuelve `None` | Modifica la imagen en el sitio. | No asigne el resultado; use `img.thumbnail(...)`. |
| `DecompressionBombWarning` | Imagen extremadamente grande. | Verifique el origen; ajuste `Image.MAX_IMAGE_PIXELS`. |
| `AttributeError: module 'PIL.Image' has no attribute 'ANTIALIAS'` | Constante eliminada en Pillow 10. | Use `Image.Resampling.LANCZOS`. |
| `textsize` no existe | Método eliminado en Pillow 10. | Use `textbbox` o `textlength`. |
| Pérdida de calidad progresiva | Guardar JPEG repetidamente. | Trabaje desde el original y guarde una sola vez. |

---

## 12. Buenas prácticas

- Use `with Image.open(...)` para liberar archivos.
- Trabaje siempre a partir del original y guarde el resultado final una sola vez.
- Aplique `exif_transpose` a cualquier foto de cámara o móvil.
- Use `LANCZOS` para redimensionar con la mejor calidad.
- Para web, use WebP (calidad 75–85) o JPEG progresivo.
- Elimine los metadatos EXIF antes de publicar fotos (pueden contener ubicación GPS).
- Para visión artificial o análisis complejo, combine Pillow con OpenCV o NumPy.

---

## 13. Referencia rápida

```python
Image.open(ruta) / Image.new(modo, tam, color) / img.save(ruta, quality=85)
img.size / img.mode / img.format / img.copy() / img.show()
img.resize(tam, Image.Resampling.LANCZOS) / img.thumbnail(tam) / img.crop(caja)
img.rotate(grados, expand=True) / img.transpose(Image.Transpose.FLIP_LEFT_RIGHT)
img.convert("L" | "RGB" | "RGBA")
img.filter(ImageFilter.GaussianBlur(2)) / ImageEnhance.Contrast(img).enhance(1.2)
ImageOps.fit / contain / pad / exif_transpose / grayscale / expand
d = ImageDraw.Draw(img); d.text / rectangle / ellipse / line / textbbox
ImageFont.truetype(ruta, tam) / ImageFont.load_default()
img.paste(otra, pos, mask) / Image.alpha_composite(a, b) / Image.blend(a, b, 0.5)
img.getexif() / np.asarray(img) / Image.fromarray(arr)
```

---

## 14. Ejercicios propuestos

1. Cree un script que convierta todas las imágenes de una carpeta a WebP mostrando el ahorro de espacio.
2. Genere tarjetas de presentación (1050 × 600 px) a partir de un CSV con nombre, cargo y email.
3. Construya un collage de 3 × 3 imágenes con un título en la parte superior.
4. Cree un avatar circular con borde de color a partir de una foto.
5. Lea los datos EXIF de una carpeta de fotos y ordénelas por fecha de captura.

---

## 15. Referencias

- Documentación oficial: https://pillow.readthedocs.io/
- Formatos compatibles: https://pillow.readthedocs.io/en/stable/handbook/image-file-formats.html
