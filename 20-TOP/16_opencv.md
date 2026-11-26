# Manual de uso: OpenCV

> **Categoría:** Visión artificial · **Utilidad:** Procesamiento de imágenes y vídeo
> **Tipo:** Librería de terceros (`pip install opencv-python`) · **Versión de referencia:** 4.8 o superior

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Imágenes como arrays de NumPy](#3-imágenes-como-arrays-de-numpy)
4. [Lectura, visualización y escritura](#4-lectura-visualización-y-escritura)
5. [Espacios de color](#5-espacios-de-color)
6. [Operaciones geométricas](#6-operaciones-geométricas)
7. [Dibujo y texto](#7-dibujo-y-texto)
8. [Filtrado y mejora de imagen](#8-filtrado-y-mejora-de-imagen)
9. [Umbralización y segmentación](#9-umbralización-y-segmentación)
10. [Morfología](#10-morfología)
11. [Bordes, líneas y círculos](#11-bordes-líneas-y-círculos)
12. [Contornos y medición de objetos](#12-contornos-y-medición-de-objetos)
13. [Histogramas](#13-histogramas)
14. [Corrección de perspectiva](#14-corrección-de-perspectiva)
15. [Búsqueda de plantillas y puntos característicos](#15-búsqueda-de-plantillas-y-puntos-característicos)
16. [Códigos QR y de barras](#16-códigos-qr-y-de-barras)
17. [Vídeo y cámaras](#17-vídeo-y-cámaras)
18. [Detección de movimiento y seguimiento](#18-detección-de-movimiento-y-seguimiento)
19. [Redes neuronales con el módulo `dnn`](#19-redes-neuronales-con-el-módulo-dnn)
20. [Ejemplo práctico completo](#20-ejemplo-práctico-completo)
21. [Errores frecuentes](#21-errores-frecuentes)
22. [Buenas prácticas](#22-buenas-prácticas)
23. [Referencia rápida](#23-referencia-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Referencias](#25-referencias)

---

## 1. Descripción

**OpenCV** (*Open Source Computer Vision Library*) es la librería de visión artificial más utilizada. Escrita en C++ y optimizada, ofrece más de 2 500 algoritmos: procesamiento de imágenes, análisis de vídeo, calibración de cámaras, detección y seguimiento de objetos, reconocimiento de códigos, fotografía computacional e inferencia de redes neuronales. En Python, las imágenes son *arrays* de NumPy, lo que la integra con todo el ecosistema científico.

**Ventajas:**
- Muy rápida (C++, SIMD, multihilo) y apta para tiempo real.
- Funcionalidad extensa en una sola librería.
- Integración directa con NumPy, Matplotlib, scikit-image, PyTorch, etc.
- Multiplataforma; licencia Apache 2.0 (uso comercial libre).

**Limitaciones:**
- API heredada de C++ con convenciones particulares (BGR, coordenadas `(x, y)` frente a `[fila, columna]`).
- Los errores suelen ser poco descriptivos (`error: (-215:Assertion failed)`).
- Para entrenar modelos de *deep learning* se usan PyTorch/TensorFlow; OpenCV solo ejecuta modelos ya entrenados.
- `imshow` no funciona en entornos sin pantalla ni con la versión *headless*.

| Si necesita… | Opción |
|---|---|
| Visión artificial, vídeo, tiempo real | **OpenCV** |
| Operaciones sencillas con imágenes (redimensionar, formatos, texto) | Pillow |
| Algoritmos científicos de imagen con API pythónica | scikit-image |
| Detección de objetos con *deep learning* lista para usar | Ultralytics YOLO, MediaPipe |
| Captura de pantalla y automatización de escritorio | PyAutoGUI / mss |

---

## 2. Instalación e importación

| Paquete | Contenido |
|---|---|
| `opencv-python` | Módulos principales + ventanas (`imshow`). |
| `opencv-contrib-python` | Lo anterior + módulos adicionales (`xfeatures2d`, `ximgproc`, seguimiento avanzado…). |
| `opencv-python-headless` | Sin interfaz gráfica: servidores, Docker, integración continua. |

```bash
pip install opencv-python            # Instale solo UNO de los paquetes anteriores
```

```python
import cv2
import numpy as np
print(cv2.__version__)
```

> Tener instalados a la vez varios paquetes de OpenCV provoca conflictos. Si ocurre, desinstálelos todos y deje solo uno.

---

## 3. Imágenes como arrays de NumPy

| Imagen | `shape` | `dtype` |
|---|---|---|
| Color | `(alto, ancho, 3)` en orden **BGR** | `uint8` (0–255) |
| Gris | `(alto, ancho)` | `uint8` |
| Con transparencia | `(alto, ancho, 4)` BGRA | `uint8` |
| Máscara binaria | `(alto, ancho)` con valores 0 y 255 | `uint8` |

```python
img = cv2.imread("pieza.jpg")
alto, ancho = img.shape[:2]
pixel = img[100, 200]                     # [fila, columna] = [y, x] → array([B, G, R])
recorte = img[50:250, 100:400]            # ROI (vista, no copia)
recorte = img[50:250, 100:400].copy()     # Copia independiente
img[0:10, :] = (0, 0, 255)                # Franja roja (BGR)
lienzo = np.zeros((480, 640, 3), np.uint8)   # Imagen negra
b, g, r = cv2.split(img) ; img = cv2.merge([b, g, r])
```

> Las funciones de OpenCV reciben puntos como **`(x, y)`** (columna, fila), mientras que la indexación de NumPy es **`[y, x]`**.

Aritmética: `cv2.add(a, b)` satura en 255, mientras que `a + b` en NumPy desborda (250 + 10 = 4). Mezcla: `cv2.addWeighted(a, 0.7, b, 0.3, 0)`.

---

## 4. Lectura, visualización y escritura

```python
img = cv2.imread("foto.jpg")                              # Devuelve None si no puede leerla (no lanza excepción)
gris = cv2.imread("foto.jpg", cv2.IMREAD_GRAYSCALE)
png = cv2.imread("logo.png", cv2.IMREAD_UNCHANGED)        # Conserva el canal alfa
if img is None:
    raise FileNotFoundError("No se pudo leer la imagen")

cv2.imwrite("salida.jpg", img, [cv2.IMWRITE_JPEG_QUALITY, 90])
cv2.imwrite("salida.png", img, [cv2.IMWRITE_PNG_COMPRESSION, 6])

# Rutas con tildes o caracteres no ASCII (Windows): leer/escribir mediante NumPy
img = cv2.imdecode(np.fromfile("informes/año_2026.jpg", np.uint8), cv2.IMREAD_COLOR)
ok, buffer = cv2.imencode(".png", img); buffer.tofile("informes/resultado_ñ.png")

cv2.imshow("Ventana", img)            # Requiere opencv-python (no headless)
cv2.waitKey(0)                        # Espera una tecla; imprescindible para que se dibuje
cv2.destroyAllWindows()
```

Con Matplotlib (convertir a RGB): `plt.imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))`.

---

## 5. Espacios de color

```python
gris = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)               # H: 0–179, S y V: 0–255
lab = cv2.cvtColor(img, cv2.COLOR_BGR2LAB)

# Segmentar por color en HSV (más robusto a cambios de iluminación que BGR)
mascara_verde = cv2.inRange(hsv, (35, 60, 40), (85, 255, 255))
# El rojo abarca ambos extremos del tono: dos rangos
rojo = cv2.inRange(hsv, (0, 80, 50), (10, 255, 255)) | cv2.inRange(hsv, (170, 80, 50), (179, 255, 255))
solo_verde = cv2.bitwise_and(img, img, mask=mascara_verde)
```

---

## 6. Operaciones geométricas

```python
pequena = cv2.resize(img, (640, 480), interpolation=cv2.INTER_AREA)          # Reducir
grande = cv2.resize(img, None, fx=2, fy=2, interpolation=cv2.INTER_CUBIC)     # Ampliar

espejo = cv2.flip(img, 1)                                  # 1 horizontal, 0 vertical, -1 ambos
girada = cv2.rotate(img, cv2.ROTATE_90_CLOCKWISE)

centro = (ancho / 2, alto / 2)
M = cv2.getRotationMatrix2D(centro, angle=15, scale=1.0)   # Rotación arbitraria
rotada = cv2.warpAffine(img, M, (ancho, alto), borderMode=cv2.BORDER_REPLICATE)

M = np.float32([[1, 0, 40], [0, 1, -20]])                  # Traslación
movida = cv2.warpAffine(img, M, (ancho, alto))
con_borde = cv2.copyMakeBorder(img, 10, 10, 10, 10, cv2.BORDER_CONSTANT, value=(255, 255, 255))
```

| Interpolación | Uso |
|---|---|
| `INTER_AREA` | Reducir tamaño (sin *aliasing*). |
| `INTER_LINEAR` (defecto) | Uso general. |
| `INTER_CUBIC`, `INTER_LANCZOS4` | Ampliar con calidad. |
| `INTER_NEAREST` | Máscaras, etiquetas, *pixel art* (no mezcla valores). |

---

## 7. Dibujo y texto

```python
cv2.line(img, (10, 10), (200, 10), (0, 255, 0), thickness=2, lineType=cv2.LINE_AA)
cv2.rectangle(img, (50, 50), (200, 150), (255, 0, 0), 2)          # -1 = relleno
cv2.circle(img, (300, 200), 40, (0, 0, 255), -1)
cv2.ellipse(img, (300, 300), (80, 40), 30, 0, 360, (0, 255, 255), 2)
cv2.polylines(img, [np.array([[10, 300], [60, 250], [110, 300]], np.int32)], isClosed=True, color=(255, 255, 0))
cv2.putText(img, "OK 98%", (50, 40), cv2.FONT_HERSHEY_SIMPLEX, 1.0, (255, 255, 255), 2, cv2.LINE_AA)
(ancho_txt, alto_txt), base = cv2.getTextSize("OK 98%", cv2.FONT_HERSHEY_SIMPLEX, 1.0, 2)
cv2.drawMarker(img, (400, 100), (0, 0, 255), cv2.MARKER_CROSS, 20, 2)
```

> `putText` solo admite caracteres ASCII (no tildes ni «ñ»). Para texto en español, dibuje con Pillow (`ImageDraw.text` con una fuente TrueType) y convierta de nuevo a *array*.

---

## 8. Filtrado y mejora de imagen

```python
cv2.blur(img, (5, 5))                                   # Media
cv2.GaussianBlur(img, (5, 5), 0)                        # Gaussiano (reduce ruido general)
cv2.medianBlur(img, 5)                                  # Mediana (ruido «sal y pimienta»)
cv2.bilateralFilter(img, 9, 75, 75)                     # Suaviza conservando bordes
cv2.fastNlMeansDenoisingColored(img, None, 10, 10, 7, 21)   # Eliminación de ruido avanzada (lenta)

nucleo = np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]])
nitida = cv2.filter2D(img, -1, nucleo)                  # Enfoque (filtro propio)

ecualizada = cv2.equalizeHist(gris)                     # Contraste global
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
mejorada = clahe.apply(gris)                            # Contraste local (mejor con iluminación irregular)

ajustada = cv2.convertScaleAbs(img, alpha=1.3, beta=20) # Contraste (alpha) y brillo (beta)
tabla = np.array([((i / 255) ** (1 / 1.5)) * 255 for i in range(256)], np.uint8)
gamma = cv2.LUT(img, tabla)                             # Corrección gamma
```

---

## 9. Umbralización y segmentación

```python
_, binaria = cv2.threshold(gris, 127, 255, cv2.THRESH_BINARY)
umbral, otsu = cv2.threshold(gris, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)   # Umbral automático
_, invertida = cv2.threshold(gris, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)  # Objetos oscuros sobre fondo claro
adaptativa = cv2.adaptiveThreshold(gris, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                   cv2.THRESH_BINARY, blockSize=31, C=10)          # Documentos, sombras

n, etiquetas, stats, centroides = cv2.connectedComponentsWithStats(binaria, connectivity=8)
for i in range(1, n):                                    # 0 es el fondo
    x, y, w, h, area = stats[i]
```

| Método | Cuándo usarlo |
|---|---|
| `THRESH_BINARY` con valor fijo | Iluminación controlada y conocida. |
| Otsu | Histograma bimodal (objeto y fondo bien diferenciados). |
| Adaptativa | Iluminación no uniforme (documentos, sombras). |
| `inRange` en HSV | Segmentación por color. |
| `watershed`, `grabCut` | Separar objetos que se tocan / extraer primer plano. |

---

## 10. Morfología

Operaciones sobre máscaras binarias para limpiar y separar regiones:

```python
k = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
cv2.erode(mascara, k, iterations=1)                     # Adelgaza; elimina puntos pequeños
cv2.dilate(mascara, k, iterations=2)                    # Engrosa; une regiones cercanas
cv2.morphologyEx(mascara, cv2.MORPH_OPEN, k)            # Apertura: elimina ruido exterior
cv2.morphologyEx(mascara, cv2.MORPH_CLOSE, k)           # Cierre: rellena huecos interiores
cv2.morphologyEx(mascara, cv2.MORPH_GRADIENT, k)        # Contorno de las regiones
cv2.distanceTransform(mascara, cv2.DIST_L2, 5)          # Distancia al fondo (separar objetos en contacto)
```

---

## 11. Bordes, líneas y círculos

```python
bordes = cv2.Canny(cv2.GaussianBlur(gris, (5, 5), 0), 50, 150)     # Umbrales bajo y alto

lineas = cv2.HoughLinesP(bordes, rho=1, theta=np.pi / 180, threshold=80, minLineLength=50, maxLineGap=10)
for x1, y1, x2, y2 in (lineas.reshape(-1, 4) if lineas is not None else []):
    cv2.line(img, (x1, y1), (x2, y2), (0, 0, 255), 2)

circulos = cv2.HoughCircles(cv2.medianBlur(gris, 5), cv2.HOUGH_GRADIENT, dp=1.2, minDist=30,
                            param1=120, param2=40, minRadius=10, maxRadius=80)
if circulos is not None:
    for x, y, r in np.round(circulos[0]).astype(int):
        cv2.circle(img, (x, y), r, (0, 255, 0), 2)

sobel_x = cv2.Sobel(gris, cv2.CV_64F, 1, 0, ksize=3)
nitidez = cv2.Laplacian(gris, cv2.CV_64F).var()          # Valor bajo → imagen desenfocada
```

---

## 12. Contornos y medición de objetos

```python
contornos, jerarquia = cv2.findContours(mascara, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
contornos = [c for c in contornos if cv2.contourArea(c) > 200]          # Filtrar ruido
cv2.drawContours(img, contornos, -1, (0, 255, 0), 2)

for c in contornos:
    area = cv2.contourArea(c)
    perimetro = cv2.arcLength(c, closed=True)
    circularidad = 4 * np.pi * area / perimetro ** 2                   # 1 = círculo perfecto
    x, y, w, h = cv2.boundingRect(c)                                   # Rectángulo alineado
    (cx, cy), (rw, rh), angulo = cv2.minAreaRect(c)                    # Rectángulo girado mínimo
    caja = cv2.boxPoints(cv2.minAreaRect(c)).astype(int)
    (xc, yc), radio = cv2.minEnclosingCircle(c)
    M = cv2.moments(c)
    centroide = (M["m10"] / M["m00"], M["m01"] / M["m00"])
    aprox = cv2.approxPolyDP(c, 0.02 * perimetro, True)                # Polígono simplificado
    vertices = len(aprox)                                              # 3 triángulo, 4 cuadrilátero…
    solidez = area / cv2.contourArea(cv2.convexHull(c))               # < 1 → concavidades, defectos
```

| Modo de recuperación | Devuelve |
|---|---|
| `RETR_EXTERNAL` | Solo contornos exteriores. |
| `RETR_LIST` | Todos, sin jerarquía. |
| `RETR_CCOMP` | Dos niveles: exteriores y agujeros. |
| `RETR_TREE` | Jerarquía completa. |

`jerarquia[0][i] = [siguiente, anterior, primer_hijo, padre]`: con `RETR_CCOMP`, un contorno con `primer_hijo != -1` tiene agujeros.

**Medidas reales**: incluya en la escena un objeto de referencia de tamaño conocido (o calibre la cámara) para obtener la relación píxeles/milímetro.

---

## 13. Histogramas

```python
hist = cv2.calcHist([gris], [0], None, [256], [0, 256])
hist_hsv = cv2.calcHist([hsv], [0, 1], None, [50, 60], [0, 180, 0, 256])        # 2D: tono × saturación
cv2.normalize(hist_hsv, hist_hsv, 0, 1, cv2.NORM_MINMAX)
similitud = cv2.compareHist(hist_a, hist_b, cv2.HISTCMP_CORREL)                   # 1 = idénticos
```

---

## 14. Corrección de perspectiva

```python
origen = np.float32([[120, 80], [530, 110], [560, 470], [90, 440]])     # Esquinas detectadas (sup-izq, sup-der, inf-der, inf-izq)
destino = np.float32([[0, 0], [420, 0], [420, 594], [0, 594]])          # Proporción A4
M = cv2.getPerspectiveTransform(origen, destino)
plano = cv2.warpPerspective(img, M, (420, 594))

def ordenar_esquinas(puntos: np.ndarray) -> np.ndarray:
    """Ordena 4 puntos como sup-izq, sup-der, inf-der, inf-izq."""
    p = puntos.reshape(4, 2).astype(np.float32)
    suma, resta = p.sum(axis=1), np.diff(p, axis=1).ravel()
    return np.array([p[suma.argmin()], p[resta.argmin()], p[suma.argmax()], p[resta.argmax()]], np.float32)
```

---

## 15. Búsqueda de plantillas y puntos característicos

### 15.1 Coincidencia de plantillas

```python
resultado = cv2.matchTemplate(gris, plantilla, cv2.TM_CCOEFF_NORMED)
_, maximo, _, (x, y) = cv2.minMaxLoc(resultado)
if maximo > 0.8:
    h, w = plantilla.shape[:2]
    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)

ys, xs = np.where(resultado >= 0.85)                      # Todas las apariciones
```

Funciona si el objeto aparece con el mismo tamaño y orientación que la plantilla.

### 15.2 Puntos característicos (invariantes a escala y rotación)

```python
orb = cv2.ORB_create(nfeatures=1000)                      # SIFT: cv2.SIFT_create() (más preciso, más lento)
kp1, des1 = orb.detectAndCompute(plantilla, None)
kp2, des2 = orb.detectAndCompute(escena, None)
emparejador = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)   # NORM_L2 para SIFT
coincidencias = sorted(emparejador.match(des1, des2), key=lambda m: m.distance)[:50]

src = np.float32([kp1[m.queryIdx].pt for m in coincidencias]).reshape(-1, 1, 2)
dst = np.float32([kp2[m.trainIdx].pt for m in coincidencias]).reshape(-1, 1, 2)
H, inliers = cv2.findHomography(src, dst, cv2.RANSAC, 5.0)          # Localiza la plantilla en la escena
```

---

## 16. Códigos QR y de barras

```python
detector = cv2.QRCodeDetector()
texto, puntos, _ = detector.detectAndDecode(img)                    # "" si no lo descifra
ok, textos, puntos, _ = detector.detectAndDecodeMulti(img)          # Varios códigos

codificador = cv2.QRCodeEncoder.create()
qr = codificador.encode("https://www.ejemplo.es")                   # Matriz pequeña: ampliar con INTER_NEAREST
qr = cv2.resize(qr, None, fx=10, fy=10, interpolation=cv2.INTER_NEAREST)

lector = cv2.barcode.BarcodeDetector()                              # EAN-13, UPC, Code 128… (OpenCV ≥ 4.8)
ok, textos, tipos, puntos = lector.detectAndDecodeWithType(img)
```

---

## 17. Vídeo y cámaras

```python
captura = cv2.VideoCapture(0)                     # Cámara 0; o ruta de archivo / URL rtsp://
captura.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
fps = captura.get(cv2.CAP_PROP_FPS)
total = int(captura.get(cv2.CAP_PROP_FRAME_COUNT))

fourcc = cv2.VideoWriter_fourcc(*"mp4v")          # "XVID" para .avi
escritor = cv2.VideoWriter("salida.mp4", fourcc, 25, (1280, 720))   # Tamaño (ancho, alto) = el de los fotogramas

try:
    while True:
        ok, fotograma = captura.read()
        if not ok:
            break                                  # Fin del vídeo o error de cámara
        procesado = cv2.Canny(fotograma, 80, 160)
        escritor.write(cv2.cvtColor(procesado, cv2.COLOR_GRAY2BGR))   # Escribir siempre 3 canales
        cv2.imshow("Vista", procesado)
        if cv2.waitKey(1) & 0xFF == ord("q"):
            break
finally:
    captura.release()
    escritor.release()
    cv2.destroyAllWindows()
```

> Para capturar con poca latencia desde cámaras IP, lea los fotogramas en un hilo separado y procese solo el más reciente.

---

## 18. Detección de movimiento y seguimiento

```python
fondo = cv2.createBackgroundSubtractorMOG2(history=300, varThreshold=25, detectShadows=True)
mascara = fondo.apply(fotograma)                            # 255 primer plano, 127 sombras
_, mascara = cv2.threshold(mascara, 200, 255, cv2.THRESH_BINARY)   # Descartar sombras

# Flujo óptico (seguimiento de puntos entre fotogramas)
puntos = cv2.goodFeaturesToTrack(gris_anterior, maxCorners=100, qualityLevel=0.3, minDistance=7)
nuevos, estado, _ = cv2.calcOpticalFlowPyrLK(gris_anterior, gris_actual, puntos, None)

# Seguidor de un objeto a partir de un rectángulo inicial
seguidor = cv2.TrackerMIL_create()                         # CSRT/KCF en opencv-contrib
seguidor.init(fotograma, (x, y, w, h))
ok, caja = seguidor.update(siguiente_fotograma)
```

---

## 19. Redes neuronales con el módulo `dnn`

OpenCV ejecuta modelos entrenados en ONNX, TensorFlow, Caffe o Darknet, sin necesidad de PyTorch:

```python
red = cv2.dnn.readNetFromONNX("modelo.onnx")
blob = cv2.dnn.blobFromImage(img, scalefactor=1 / 255, size=(640, 640), swapRB=True)
red.setInput(blob)
salida = red.forward()

# Detector facial incluido (requiere descargar el modelo YuNet en ONNX)
detector = cv2.FaceDetectorYN.create("face_detection_yunet.onnx", "", (ancho, alto))
_, caras = detector.detect(img)                           # x, y, w, h, puntos faciales, puntuación
```

Modelos preentrenados: *OpenCV Zoo* (https://github.com/opencv/opencv_zoo).

---

## 20. Ejemplo práctico completo

**Estación de inspección visual** que procesa cuatro escenarios habituales en industria y oficina. Para que sea reproducible sin cámara ni archivos externos, el propio script **genera las imágenes y el vídeo de prueba** con ruido, sombras y perspectiva:

1. **Escáner de documentos**: localiza una hoja fotografiada en ángulo, corrige la perspectiva y la binariza con umbral adaptativo.
2. **Inspección de piezas**: segmenta arandelas y tornillos, los clasifica por forma (circularidad, relación de aspecto), mide su tamaño en milímetros con una referencia de escala y detecta arandelas defectuosas sin agujero (jerarquía de contornos).
3. **Trazabilidad**: lee un código QR girado y con ruido pegado en la caja.
4. **Conteo en cinta transportadora**: sustracción de fondo sobre un vídeo, seguimiento de centroides y conteo de objetos que cruzan una línea; guarda un vídeo anotado.

```python
"""
estacion_inspeccion.py — Visión artificial con OpenCV: documentos, piezas, QR y vídeo.
Requiere: pip install opencv-python numpy
Uso: python estacion_inspeccion.py
"""
from __future__ import annotations

import json
from dataclasses import asdict, dataclass
from pathlib import Path

import cv2
import numpy as np

SALIDA = Path("inspeccion")
SALIDA.mkdir(exist_ok=True)
RNG = np.random.default_rng(11)
VERDE, ROJO, AZUL, AMARILLO = (60, 180, 75), (40, 40, 220), (200, 120, 30), (0, 200, 230)


def guardar(nombre: str, img: np.ndarray) -> None:
    cv2.imwrite(str(SALIDA / nombre), img)


def etiqueta(img, texto, punto, color=(255, 255, 255), escala=0.5):
    """Texto con fondo para que sea legible sobre cualquier imagen."""
    (w, h), base = cv2.getTextSize(texto, cv2.FONT_HERSHEY_SIMPLEX, escala, 1)
    x, y = int(punto[0]), int(punto[1])
    cv2.rectangle(img, (x, y - h - 4), (x + w + 4, y + base), (30, 30, 30), -1)
    cv2.putText(img, texto, (x + 2, y), cv2.FONT_HERSHEY_SIMPLEX, escala, color, 1, cv2.LINE_AA)


def ruido(img: np.ndarray, sigma: float = 8) -> np.ndarray:
    return np.clip(img.astype(np.int16) + RNG.normal(0, sigma, img.shape), 0, 255).astype(np.uint8)


# ============================================================ 1. Escáner de documentos
def generar_foto_documento() -> np.ndarray:
    hoja = np.full((594, 420, 3), 245, np.uint8)
    cv2.putText(hoja, "ALBARAN DE ENTREGA", (40, 70), cv2.FONT_HERSHEY_DUPLEX, 0.9, (20, 20, 20), 2)
    cv2.putText(hoja, "N. 2026-11-0457", (40, 110), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (40, 40, 40), 2)
    for i, (art, uds) in enumerate([("Arandela M8", 500), ("Tornillo M8x40", 500), ("Tuerca M8", 480)]):
        y = 180 + i * 45
        cv2.putText(hoja, f"{art:<18}{uds:>6} uds", (40, y), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (30, 30, 30), 1)
        cv2.line(hoja, (40, y + 12), (380, y + 12), (180, 180, 180), 1)
    cv2.rectangle(hoja, (40, 480), (200, 540), (60, 60, 60), 1)
    cv2.putText(hoja, "Firma", (50, 530), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (90, 90, 90), 1)

    mesa = np.zeros((600, 800, 3), np.uint8)
    mesa[:] = (95, 120, 140)                                                  # Mesa de madera (BGR)
    mesa = cv2.GaussianBlur(ruido(mesa, 18), (7, 7), 0)
    esquinas = np.float32([[0, 0], [420, 0], [420, 594], [0, 594]])
    destino = np.float32([[205, 60], [590, 105], [640, 560], [150, 520]])      # Foto en ángulo
    M = cv2.getPerspectiveTransform(esquinas, destino)
    deformada = cv2.warpPerspective(hoja, M, (800, 600))
    mascara = cv2.warpPerspective(np.full((594, 420), 255, np.uint8), M, (800, 600))
    foto = np.where(mascara[..., None] > 0, deformada, mesa)
    gradiente = np.linspace(1.0, 0.55, 800)[None, :, None]                    # Sombra lateral
    return ruido((foto * gradiente).astype(np.uint8), 5)


def ordenar_esquinas(puntos: np.ndarray) -> np.ndarray:
    p = puntos.reshape(4, 2).astype(np.float32)
    suma, resta = p.sum(axis=1), np.diff(p, axis=1).ravel()
    return np.array([p[suma.argmin()], p[resta.argmin()], p[suma.argmax()], p[resta.argmax()]], np.float32)


def escanear_documento(foto: np.ndarray) -> dict:
    gris = cv2.cvtColor(foto, cv2.COLOR_BGR2GRAY)
    bordes = cv2.Canny(cv2.GaussianBlur(gris, (5, 5), 0), 40, 120)
    bordes = cv2.dilate(bordes, cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5)))
    contornos, _ = cv2.findContours(bordes, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    hoja = None
    for c in sorted(contornos, key=cv2.contourArea, reverse=True)[:5]:
        aprox = cv2.approxPolyDP(c, 0.02 * cv2.arcLength(c, True), True)
        if len(aprox) == 4 and cv2.contourArea(aprox) > 0.15 * foto.shape[0] * foto.shape[1]:
            hoja = ordenar_esquinas(aprox)
            break
    if hoja is None:
        raise RuntimeError("No se encontró ningún documento")

    ancho = int(max(np.linalg.norm(hoja[0] - hoja[1]), np.linalg.norm(hoja[3] - hoja[2])))
    alto = int(ancho * 297 / 210)                                              # Proporción A4
    M = cv2.getPerspectiveTransform(hoja, np.float32([[0, 0], [ancho, 0], [ancho, alto], [0, alto]]))
    plano = cv2.warpPerspective(foto, M, (ancho, alto))
    escaneado = cv2.adaptiveThreshold(cv2.cvtColor(plano, cv2.COLOR_BGR2GRAY), 255,
                                      cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 31, 15)

    anotada = foto.copy()
    cv2.polylines(anotada, [hoja.astype(np.int32)], True, VERDE, 3)
    for i, (x, y) in enumerate(hoja):
        cv2.circle(anotada, (int(x), int(y)), 8, ROJO, -1)
        etiqueta(anotada, str(i + 1), (x + 10, y))
    guardar("1_documento_original.jpg", foto)
    guardar("1_documento_detectado.jpg", anotada)
    guardar("1_documento_escaneado.png", escaneado)
    return {"esquinas": hoja.round(1).tolist(), "tamano_salida": [ancho, alto],
            "nitidez": round(float(cv2.Laplacian(escaneado, cv2.CV_64F).var()), 1)}


# ============================================================ 2. Inspección de piezas
@dataclass
class Pieza:
    id: int
    tipo: str
    centro: tuple[int, int]
    medida_mm: float
    circularidad: float
    estado: str


def generar_bandeja() -> tuple[np.ndarray, float]:
    """Bandeja clara con arandelas, tornillos y una referencia cuadrada de 20 mm. Devuelve (imagen, px/mm reales)."""
    px_mm = 4.0
    img = np.full((520, 760, 3), (200, 205, 210), np.uint8)
    cv2.rectangle(img, (20, 20), (20 + int(20 * px_mm), 20 + int(20 * px_mm)), (30, 30, 30), -1)   # Referencia
    arandelas = [((200, 120), 8.0, True), ((330, 110), 8.0, True), ((470, 140), 10.0, True),
                 ((620, 120), 8.0, False), ((180, 330), 10.0, True), ((330, 400), 8.0, True)]
    for (x, y), radio_mm, con_agujero in arandelas:
        cv2.circle(img, (x, y), int(radio_mm * px_mm), (70, 70, 75), -1, cv2.LINE_AA)
        if con_agujero:
            cv2.circle(img, (x, y), int(radio_mm * 0.45 * px_mm), (200, 205, 210), -1, cv2.LINE_AA)
    for (x, y), largo_mm, angulo in [((540, 250), 40, 15), ((560, 420), 30, -10), ((680, 300), 40, 90)]:
        rect = ((x, y), (largo_mm * px_mm, 8 * px_mm), angulo)
        cv2.fillPoly(img, [cv2.boxPoints(rect).astype(np.int32)], (60, 65, 70), cv2.LINE_AA)
        cabeza = np.array(rect[0]) - 0.5 * largo_mm * px_mm * np.array([np.cos(np.radians(angulo)), np.sin(np.radians(angulo))])
        cv2.circle(img, tuple(cabeza.astype(int)), int(7 * px_mm), (60, 65, 70), -1, cv2.LINE_AA)
    sombra = np.tile(np.linspace(1.05, 0.8, 760), (520, 1))[..., None]          # Iluminación desigual
    return ruido(np.clip(img * sombra, 0, 255).astype(np.uint8), 6), px_mm


def inspeccionar_piezas(img: np.ndarray) -> list[Pieza]:
    gris = cv2.GaussianBlur(cv2.cvtColor(img, cv2.COLOR_BGR2GRAY), (5, 5), 0)
    _, mascara = cv2.threshold(gris, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
    mascara = cv2.morphologyEx(mascara, cv2.MORPH_OPEN, cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5)))
    contornos, jerarquia = cv2.findContours(mascara, cv2.RETR_CCOMP, cv2.CHAIN_APPROX_SIMPLE)
    jerarquia = jerarquia[0]

    exteriores = [i for i, h in enumerate(jerarquia) if h[3] == -1 and cv2.contourArea(contornos[i]) > 150]
    # Calibración: la referencia es el exterior con 4 vértices y aspecto ~1 más cercano al origen
    ref = min((i for i in exteriores if len(cv2.approxPolyDP(contornos[i], 0.04 * cv2.arcLength(contornos[i], True), True)) == 4),
              key=lambda i: cv2.boundingRect(contornos[i])[0] + cv2.boundingRect(contornos[i])[1])
    px_mm = np.sqrt(cv2.contourArea(contornos[ref])) / 20.0

    anotada = img.copy()
    piezas: list[Pieza] = []
    for i in exteriores:
        if i == ref:
            continue
        c = contornos[i]
        area, perimetro = cv2.contourArea(c), cv2.arcLength(c, True)
        circularidad = 4 * np.pi * area / perimetro ** 2
        M = cv2.moments(c)
        centro = (int(M["m10"] / M["m00"]), int(M["m01"] / M["m00"]))
        tiene_agujero = jerarquia[i][2] != -1 and cv2.contourArea(contornos[jerarquia[i][2]]) > 30
        if circularidad > 0.8:
            (_, _), radio = cv2.minEnclosingCircle(c)
            medida = 2 * radio / px_mm
            tipo, estado = "arandela", "OK" if tiene_agujero else "DEFECTO: sin agujero"
            texto = f"Arandela D{medida:.1f}mm"
        else:
            (_, _), (w, h), _ = cv2.minAreaRect(c)
            medida = max(w, h) / px_mm
            tipo, estado = "tornillo", "OK"
            texto = f"Tornillo {medida:.0f}mm"                   # Longitud total (vástago + cabeza)
        color = VERDE if estado == "OK" else ROJO
        cv2.drawContours(anotada, contornos, i, color, 2)
        if tiene_agujero:
            cv2.drawContours(anotada, contornos, jerarquia[i][2], AMARILLO, 1)
        etiqueta(anotada, f"#{len(piezas) + 1} {texto}" + ("" if estado == "OK" else " X"), (centro[0] - 60, centro[1] - 38),
                 (255, 255, 255) if estado == "OK" else (120, 120, 255))
        piezas.append(Pieza(len(piezas) + 1, tipo, centro, round(medida, 1), round(circularidad, 3), estado))

    cv2.drawContours(anotada, contornos, ref, AZUL, 2)
    etiqueta(anotada, f"Ref. 20 mm = {px_mm * 20:.0f} px", (20, 125))
    resumen = f"Piezas: {len(piezas)} | Defectos: {sum(p.estado != 'OK' for p in piezas)}"
    etiqueta(anotada, resumen, (440, 500), escala=0.65)
    guardar("2_piezas_mascara.png", mascara)
    guardar("2_piezas_inspeccion.jpg", anotada)
    return piezas


# ============================================================ 3. Código QR
def generar_caja_con_qr(texto: str) -> np.ndarray:
    qr = cv2.QRCodeEncoder.create().encode(texto)
    qr = cv2.resize(qr, None, fx=6, fy=6, interpolation=cv2.INTER_NEAREST)
    qr = cv2.copyMakeBorder(qr, 24, 24, 24, 24, cv2.BORDER_CONSTANT, value=255)
    caja = np.full((420, 560, 3), (90, 140, 190), np.uint8)                    # Cartón
    h, w = qr.shape
    M = cv2.getRotationMatrix2D((w / 2, h / 2), 23, 1.0)
    girado = cv2.warpAffine(cv2.cvtColor(qr, cv2.COLOR_GRAY2BGR), M, (w, h), borderValue=(90, 140, 190))
    caja[100:100 + h, 180:180 + w] = girado
    return ruido(cv2.GaussianBlur(caja, (3, 3), 0), 10)


def leer_qr(img: np.ndarray) -> dict:
    texto, puntos, _ = cv2.QRCodeDetector().detectAndDecode(img)
    anotada = img.copy()
    if puntos is not None and texto:
        cv2.polylines(anotada, [puntos.astype(np.int32)], True, VERDE, 3)
        etiqueta(anotada, texto, puntos[0][0] + np.array([0, -12]), escala=0.6)
    guardar("3_qr.jpg", anotada)
    return {"leido": bool(texto), "contenido": texto}


# ============================================================ 4. Conteo en vídeo
def generar_video_cinta(ruta: Path, n_fotogramas: int = 260) -> None:
    ancho, alto = 640, 240
    escritor = cv2.VideoWriter(str(ruta), cv2.VideoWriter_fourcc(*"mp4v"), 25, (ancho, alto))
    objetos = [{"x": -60 - 100 * i, "y": int(RNG.integers(70, 170)), "r": int(RNG.integers(18, 30)),
                "v": 5.5} for i in range(7)]
    for _ in range(n_fotogramas):
        f = np.full((alto, ancho, 3), (70, 75, 80), np.uint8)
        for x in range(0, ancho, 40):                                         # Rodillos de la cinta
            cv2.line(f, (x, 40), (x, 200), (60, 64, 68), 2)
        for o in objetos:
            o["x"] += o["v"]
            cv2.circle(f, (int(o["x"]), o["y"]), o["r"], (40, 160, 230), -1, cv2.LINE_AA)
        escritor.write(ruido(f, 4))
    escritor.release()


def contar_en_video(ruta: Path) -> dict:
    captura = cv2.VideoCapture(str(ruta))
    if not captura.isOpened():
        raise RuntimeError(f"No se pudo abrir {ruta}")
    ancho, alto = int(captura.get(cv2.CAP_PROP_FRAME_WIDTH)), int(captura.get(cv2.CAP_PROP_FRAME_HEIGHT))
    salida = cv2.VideoWriter(str(SALIDA / "4_cinta_anotada.mp4"), cv2.VideoWriter_fourcc(*"mp4v"), 25, (ancho, alto))
    fondo = cv2.createBackgroundSubtractorMOG2(history=100, varThreshold=40, detectShadows=False)
    linea_x, siguiente_id, contados = ancho // 2, 0, 0
    pistas: dict[int, tuple[int, int]] = {}                        # id → último centroide
    nucleo = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (7, 7))
    n = 0
    while True:
        ok, f = captura.read()
        if not ok:
            break
        n += 1
        mascara = fondo.apply(f, learningRate=0.002)               # Aprendizaje lento: los objetos no pasan a ser «fondo»
        mascara = cv2.morphologyEx(cv2.threshold(mascara, 200, 255, cv2.THRESH_BINARY)[1], cv2.MORPH_OPEN, nucleo)
        num, _, stats, centroides = cv2.connectedComponentsWithStats(mascara)
        actuales = [tuple(map(int, centroides[i])) for i in range(1, num) if stats[i, cv2.CC_STAT_AREA] > 400]

        nuevas: dict[int, tuple[int, int]] = {}
        for c in actuales:                                          # Asociación por vecino más cercano
            candidatos = [(np.hypot(c[0] - p[0], c[1] - p[1]), i) for i, p in pistas.items() if i not in nuevas]
            distancia, ident = min(candidatos, default=(1e9, None))
            if ident is None or distancia > 40:
                ident, siguiente_id = siguiente_id, siguiente_id + 1
            elif pistas[ident][0] < linea_x <= c[0]:                # Cruce de izquierda a derecha
                contados += 1
            nuevas[ident] = c
        pistas = nuevas

        cv2.line(f, (linea_x, 0), (linea_x, alto), ROJO, 2)
        for ident, (x, y) in pistas.items():
            cv2.circle(f, (x, y), 4, AMARILLO, -1)
            etiqueta(f, f"id {ident}", (x + 6, y - 6), escala=0.4)
        etiqueta(f, f"Contados: {contados}", (10, 25), escala=0.7)
        salida.write(f)
        if n == 120:
            guardar("4_cinta_fotograma.jpg", f)
    captura.release()
    salida.release()
    return {"fotogramas": n, "objetos_contados": contados}


def main() -> int:
    informe: dict = {}
    print("1. Escáner de documentos")
    informe["documento"] = escanear_documento(generar_foto_documento())
    print(f"   Hoja detectada y enderezada a {informe['documento']['tamano_salida']} px")

    print("2. Inspección de piezas")
    bandeja, px_mm_real = generar_bandeja()
    piezas = inspeccionar_piezas(bandeja)
    informe["piezas"] = [asdict(p) for p in piezas]
    for p in piezas:
        print(f"   #{p.id} {p.tipo:<9} medida {p.medida_mm:5.1f} mm  circularidad {p.circularidad:.2f}  {p.estado}")

    print("3. Trazabilidad por QR")
    informe["qr"] = leer_qr(generar_caja_con_qr("LOTE:2026-0457;ART:ARAND-M8;UDS:500"))
    print(f"   Contenido: {informe['qr']['contenido'] or 'no leído'}")

    print("4. Conteo en cinta transportadora")
    video = SALIDA / "cinta_entrada.mp4"
    generar_video_cinta(video)
    informe["cinta"] = contar_en_video(video)
    print(f"   {informe['cinta']['objetos_contados']} objetos contados en {informe['cinta']['fotogramas']} fotogramas")

    (SALIDA / "informe.json").write_text(json.dumps(informe, ensure_ascii=False, indent=2), encoding="utf-8")
    print(f"\n✔ Resultados en la carpeta «{SALIDA}»")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Técnicas aplicadas:

| Escenario | Técnicas |
|---|---|
| Documento | Canny, dilatación, contornos, `approxPolyDP`, ordenación de esquinas, `warpPerspective`, umbral adaptativo. |
| Piezas | Otsu, apertura morfológica, `RETR_CCOMP` (agujeros), circularidad, `minAreaRect`, calibración con objeto de referencia. |
| QR | `QRCodeEncoder`, `QRCodeDetector.detectAndDecode`. |
| Vídeo | `VideoWriter`/`VideoCapture`, MOG2, componentes conectados, seguimiento por vecino más cercano, conteo por cruce de línea. |

---

## 21. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `AttributeError: 'NoneType' object has no attribute 'shape'` | `imread` no encontró o no pudo leer el archivo. | Compruebe la ruta; en Windows con tildes use `imdecode(np.fromfile(...))`. |
| Colores intercambiados (azul por rojo) | OpenCV usa BGR. | `cv2.cvtColor(img, cv2.COLOR_BGR2RGB)` al pasar a Matplotlib/Pillow. |
| `error: (-215:Assertion failed) !_src.empty()` | Imagen vacía (lectura fallida o fotograma nulo). | Verifique `img is not None` y `ok` de `read()`. |
| `error: (-215) … in function 'cvtColor'` | Número de canales incorrecto (p. ej., convertir a gris una imagen ya gris). | Compruebe `img.ndim` / `shape`. |
| `The function is not implemented. Rebuild the library with GTK+…` | `imshow` con la versión *headless*. | Instale `opencv-python` o guarde con `imwrite`. |
| `imshow` muestra una ventana gris o no responde | Falta `waitKey`. | `cv2.waitKey(1)` en bucles, `waitKey(0)` para imágenes. |
| El vídeo escrito está vacío o no se abre | Tamaño de fotograma distinto al indicado, códec no disponible o 1 canal. | Mismo `(ancho, alto)`, 3 canales, códec `mp4v`/`XVID`; `release()` al final. |
| `TypeError: … integer argument expected` en dibujo | Coordenadas `float` o tipos NumPy. | Convierta con `int()` o `.astype(int)`. |
| `findContours` no encuentra objetos | Objetos negros sobre fondo blanco. | Los objetos deben ser blancos: `THRESH_BINARY_INV`. |
| Desbordamientos al sumar imágenes | Aritmética `uint8` de NumPy. | `cv2.add`, `cv2.addWeighted` o convertir a `int16`/`float32`. |
| Tildes como «??» en `putText` | Solo ASCII. | Dibuje el texto con Pillow. |
| Conflicto al importar `cv2` | Varios paquetes de OpenCV instalados. | Desinstale todos y deje uno. |

---

## 22. Buenas prácticas

- Compruebe siempre el resultado de `imread`, `VideoCapture.isOpened()` y `read()`.
- Trabaje en escala de grises cuando el color no aporte información: es más rápido y robusto.
- Reduzca la resolución antes de procesar si la precisión lo permite (`INTER_AREA`).
- Suavice antes de detectar bordes o umbralizar, y limpie las máscaras con morfología.
- Use umbrales relativos o automáticos (Otsu, adaptativo, percentiles) en lugar de valores fijos cuando la iluminación varíe.
- Separe la adquisición (cámara/vídeo) del procesamiento y guarde imágenes intermedias para depurar.
- Para medidas reales, calibre la cámara (`calibrateCamera`) o use objetos de referencia en la escena.
- Libere siempre los recursos (`release()`, `destroyAllWindows()`) en un bloque `finally`.
- Valide los algoritmos con un conjunto de imágenes representativo (distintas iluminaciones, posiciones y defectos).

---

## 23. Referencia rápida

```python
cv2.imread(ruta, flags) · cv2.imwrite(ruta, img) · cv2.imshow/waitKey/destroyAllWindows · cv2.imdecode/imencode
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY|BGR2HSV|BGR2RGB) · cv2.inRange(hsv, bajo, alto) · cv2.bitwise_and/or/not
cv2.resize(img, (w, h), interpolation=) · cv2.flip · cv2.rotate · getRotationMatrix2D + warpAffine · getPerspectiveTransform + warpPerspective
cv2.GaussianBlur · medianBlur · bilateralFilter · createCLAHE().apply · equalizeHist · filter2D · LUT
cv2.threshold(gris, 0, 255, THRESH_BINARY+THRESH_OTSU) · adaptiveThreshold · connectedComponentsWithStats
cv2.morphologyEx(m, MORPH_OPEN|CLOSE, getStructuringElement(MORPH_ELLIPSE, (k, k))) · erode · dilate
cv2.Canny · HoughLinesP · HoughCircles · findContours(m, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE) · drawContours
contourArea · arcLength · boundingRect · minAreaRect · minEnclosingCircle · moments · approxPolyDP · convexHull
cv2.matchTemplate + minMaxLoc · ORB_create · BFMatcher · findHomography · QRCodeDetector().detectAndDecode
cv2.VideoCapture(0|ruta).read() · VideoWriter(ruta, VideoWriter_fourcc(*"mp4v"), fps, (w, h)) · createBackgroundSubtractorMOG2
cv2.line/rectangle/circle/putText/polylines · dnn.readNetFromONNX · dnn.blobFromImage
```

---

## 24. Ejercicios propuestos

1. Amplíe el escáner para procesar todas las imágenes de una carpeta y generar un PDF con las páginas enderezadas (con Pillow).
2. Añada a la inspección de piezas la detección de tuercas hexagonales (6 vértices con `approxPolyDP`) y de arandelas con el agujero descentrado.
3. Cree un contador de objetos por color (rojo, verde, azul) en la cinta usando segmentación HSV.
4. Implemente un detector de desenfoque que descarte fotos con varianza del laplaciano por debajo de un umbral.
5. Use el detector facial YuNet (`FaceDetectorYN`) con la cámara del equipo para pixelar automáticamente las caras.

---

## 25. Referencias

- Documentación oficial: https://docs.opencv.org/
- Tutoriales de OpenCV-Python: https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html
- Repositorio de paquetes para Python: https://github.com/opencv/opencv-python
- Modelos preentrenados (OpenCV Zoo): https://github.com/opencv/opencv_zoo
