# Manual de uso: OpenCV

> **Categoría:** Inteligencia artificial · **Utilidad:** Visión artificial y procesamiento de imágenes
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** Apache 2.0 · **Importación:** `cv2`

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Leer, mostrar y guardar imágenes](#4-leer-mostrar-y-guardar-imágenes)
5. [Espacios de color](#5-espacios-de-color)
6. [Transformaciones geométricas](#6-transformaciones-geométricas)
7. [Filtros y mejora de la imagen](#7-filtros-y-mejora-de-la-imagen)
8. [Umbralización y segmentación](#8-umbralización-y-segmentación)
9. [Operaciones morfológicas](#9-operaciones-morfológicas)
10. [Bordes, contornos y medición de formas](#10-bordes-contornos-y-medición-de-formas)
11. [Detección de formas, características y códigos](#11-detección-de-formas-características-y-códigos)
12. [Dibujo y anotación](#12-dibujo-y-anotación)
13. [Vídeo y cámaras](#13-vídeo-y-cámaras)
14. [Aprendizaje profundo con el módulo `dnn`](#14-aprendizaje-profundo-con-el-módulo-dnn)
15. [Calibración y medición en unidades reales](#15-calibración-y-medición-en-unidades-reales)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**OpenCV** (*Open Source Computer Vision Library*) es la librería de referencia para **visión artificial** y procesamiento de imágenes y vídeo. Escrita en C++ y optimizada para tiempo real, ofrece más de 2 500 algoritmos: filtrado, segmentación, detección de bordes y contornos, geometría, calibración de cámaras, seguimiento de objetos, detección de características y ejecución de redes neuronales.

**Aplicaciones habituales:**
- Inspección de calidad en la industria (medición de piezas, detección de defectos).
- Lectura de códigos QR, matrículas y documentos (preprocesado para OCR).
- Conteo y seguimiento de objetos o personas en vídeo.
- Robótica, realidad aumentada, imagen médica, agricultura de precisión.

**Ventajas:**
- Muy rápida (C++ optimizado, SIMD, multihilo) y apta para tiempo real.
- Las imágenes son **arrays de NumPy**: se integra con todo el ecosistema científico de Python.
- Funciona en Windows, macOS, Linux, Android, iOS y dispositivos embebidos.
- Módulo `dnn` para ejecutar modelos de aprendizaje profundo (ONNX) sin depender de PyTorch o TensorFlow.

**Limitaciones:**
- Los algoritmos clásicos requieren ajustar parámetros según la iluminación y la escena.
- Para reconocimiento complejo (clasificar miles de objetos, segmentación semántica) se necesitan redes neuronales.
- Convenciones heredadas que causan errores: orden de color **BGR**, coordenadas `(x, y)` frente a índices `[fila, columna]`.

| Si necesita… | Opción |
|---|---|
| Procesado de imagen y vídeo, visión clásica, tiempo real | **OpenCV** |
| Detección y segmentación con modelos modernos | Ultralytics YOLO, PyTorch + torchvision |
| Clasificación de imágenes con modelos preentrenados | Hugging Face Transformers, timm |
| Procesado científico de imágenes | scikit-image |
| Edición sencilla (redimensionar, formatos) | Pillow |

---

## 2. Instalación e importación

| Paquete | Contenido |
|---|---|
| `opencv-python` | Módulos principales con interfaz gráfica (`imshow`) |
| `opencv-python-headless` | Sin interfaz gráfica: servidores, Docker, integración continua |
| `opencv-contrib-python` | Principales + módulos adicionales (*contrib*) |
| `opencv-contrib-python-headless` | *Contrib* sin interfaz gráfica |

```bash
pip install opencv-python            # Instale SOLO uno de los cuatro paquetes
```

> Instalar varios paquetes de OpenCV a la vez provoca conflictos. Si ocurre, desinstale todos (`pip uninstall opencv-python opencv-contrib-python opencv-python-headless`) y reinstale uno.

```bash
python -c "import cv2; print(cv2.__version__)"
```

Importaciones habituales:

```python
import cv2
import numpy as np
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Imagen | Array de NumPy: `(alto, ancho)` en gris o `(alto, ancho, 3)` en color, normalmente `uint8` (0–255). |
| BGR | OpenCV guarda el color en orden **azul, verde, rojo** (no RGB). |
| Píxel | `imagen[fila, columna]` = `imagen[y, x]`. |
| Coordenadas | Las funciones de OpenCV reciben puntos como `(x, y)`; el origen está arriba a la izquierda. |
| ROI | Región de interés: `imagen[y1:y2, x1:x2]` (vista, no copia). |
| Canal | Cada componente de color. `cv2.split` / `cv2.merge`. |
| Máscara / imagen binaria | Imagen de un canal con valores 0 y 255. |
| Núcleo (*kernel*) | Matriz pequeña usada en filtros y morfología. |
| Contorno | Lista de puntos que forma la frontera de un objeto. |

> Las funciones de OpenCV **devuelven** la imagen resultante; muy pocas modifican la entrada (las de dibujo sí lo hacen).

---

## 4. Leer, mostrar y guardar imágenes

```python
imagen = cv2.imread("foto.jpg")                       # BGR, uint8
gris = cv2.imread("foto.jpg", cv2.IMREAD_GRAYSCALE)
con_alfa = cv2.imread("logo.png", cv2.IMREAD_UNCHANGED)   # Conserva el canal alfa

if imagen is None:                                    # ⚠ imread NO lanza excepción si falla
    raise FileNotFoundError("No se pudo leer la imagen")

alto, ancho, canales = imagen.shape
cv2.imwrite("salida.png", imagen)
cv2.imwrite("salida.jpg", imagen, [cv2.IMWRITE_JPEG_QUALITY, 90])
```

**Mostrar** (requiere el paquete con interfaz gráfica):

```python
cv2.imshow("Ventana", imagen)
cv2.waitKey(0)                 # Espera una tecla (necesario para que la ventana se dibuje)
cv2.destroyAllWindows()
```

En Jupyter, use Matplotlib convirtiendo a RGB:

```python
import matplotlib.pyplot as plt
plt.imshow(cv2.cvtColor(imagen, cv2.COLOR_BGR2RGB)); plt.axis("off")
```

> `cv2.imread` no admite rutas con caracteres no ASCII en Windows (tildes, «ñ»). Alternativa: `cv2.imdecode(np.fromfile(ruta, np.uint8), cv2.IMREAD_COLOR)`.

---

## 5. Espacios de color

```python
gris = cv2.cvtColor(imagen, cv2.COLOR_BGR2GRAY)
rgb = cv2.cvtColor(imagen, cv2.COLOR_BGR2RGB)
hsv = cv2.cvtColor(imagen, cv2.COLOR_BGR2HSV)
lab = cv2.cvtColor(imagen, cv2.COLOR_BGR2LAB)
azul, verde, rojo = cv2.split(imagen)
```

| Espacio | Uso |
|---|---|
| Gris | Detección de bordes, umbralización, características |
| HSV (tono, saturación, valor) | Segmentar por color con independencia de la iluminación |
| LAB | Diferencias de color perceptuales, mejora de contraste |

**Segmentar por color en HSV** (en OpenCV el tono va de 0 a 179):

```python
mascara = cv2.inRange(hsv, (35, 60, 60), (85, 255, 255))    # Tonos verdes
solo_verde = cv2.bitwise_and(imagen, imagen, mask=mascara)
```

---

## 6. Transformaciones geométricas

```python
pequena = cv2.resize(imagen, (640, 480))                                  # Tamaño (ancho, alto)
mitad = cv2.resize(imagen, None, fx=0.5, fy=0.5, interpolation=cv2.INTER_AREA)   # Reducir
recorte = imagen[100:300, 200:500]                                        # [y1:y2, x1:x2]
volteada = cv2.flip(imagen, 1)                                            # 1 horizontal, 0 vertical
girada = cv2.rotate(imagen, cv2.ROTATE_90_CLOCKWISE)

M = cv2.getRotationMatrix2D(centro=(ancho / 2, alto / 2), angle=30, scale=1.0)
rotada = cv2.warpAffine(imagen, M, (ancho, alto))
```

**Corrección de perspectiva** (enderezar un documento o una etiqueta):

```python
origen = np.float32([[120, 80], [510, 110], [530, 420], [100, 400]])     # Esquinas en la foto
destino = np.float32([[0, 0], [420, 0], [420, 297], [0, 297]])          # Rectángulo A4 a escala
M = cv2.getPerspectiveTransform(origen, destino)
documento = cv2.warpPerspective(imagen, M, (420, 297))
```

| Interpolación | Uso |
|---|---|
| `INTER_AREA` | Reducir tamaño |
| `INTER_LINEAR` (defecto) / `INTER_CUBIC` | Ampliar |
| `INTER_NEAREST` | Máscaras y etiquetas (no mezcla valores) |

---

## 7. Filtros y mejora de la imagen

| Filtro | Función | Uso |
|---|---|---|
| `cv2.GaussianBlur(img, (5, 5), 0)` | Suavizado gaussiano | Reducir ruido antes de umbralizar o detectar bordes |
| `cv2.medianBlur(img, 5)` | Mediana | Ruido «sal y pimienta» |
| `cv2.bilateralFilter(img, 9, 75, 75)` | Suaviza conservando bordes | Fotografías, piel |
| `cv2.filter2D(img, -1, nucleo)` | Convolución propia | Enfoque, detección personalizada |
| `cv2.fastNlMeansDenoisingColored(img)` | Eliminación de ruido avanzada | Imágenes con poca luz |

**Contraste:**

```python
ecualizada = cv2.equalizeHist(gris)                               # Global
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))       # Local adaptativo (recomendado)
mejorada = clahe.apply(gris)
ajustada = cv2.convertScaleAbs(imagen, alpha=1.3, beta=20)         # Contraste y brillo
```

**Corrección de iluminación desigual:** estimar el fondo con una apertura morfológica de gran tamaño y restarlo (véase el ejemplo completo).

---

## 8. Umbralización y segmentación

```python
_, binaria = cv2.threshold(gris, 127, 255, cv2.THRESH_BINARY)                     # Umbral fijo
_, binaria = cv2.threshold(gris, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)      # Umbral automático
binaria = cv2.adaptiveThreshold(gris, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                cv2.THRESH_BINARY, 31, 5)                           # Umbral local
```

| Método | Cuándo |
|---|---|
| Fijo | Iluminación controlada y conocida |
| Otsu | Histograma con dos grupos claros (objeto/fondo) |
| Adaptativo | Iluminación desigual, documentos |
| `THRESH_BINARY_INV` | Objetos oscuros sobre fondo claro |

**Componentes conectados** (contar y medir regiones):

```python
n, etiquetas, estadisticas, centroides = cv2.connectedComponentsWithStats(binaria)
# estadisticas[i] = [x, y, ancho, alto, área]; la etiqueta 0 es el fondo
```

Otras técnicas: `cv2.watershed` (separar objetos que se tocan), `cv2.grabCut` (recortar un objeto a partir de un rectángulo), `cv2.kmeans` (agrupar colores).

---

## 9. Operaciones morfológicas

Se aplican a imágenes binarias para limpiar y ajustar formas:

```python
nucleo = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))     # MORPH_RECT, MORPH_CROSS
erosionada = cv2.erode(binaria, nucleo, iterations=1)
dilatada = cv2.dilate(binaria, nucleo, iterations=1)
abierta = cv2.morphologyEx(binaria, cv2.MORPH_OPEN, nucleo)        # Elimina puntos pequeños
cerrada = cv2.morphologyEx(binaria, cv2.MORPH_CLOSE, nucleo)       # Rellena huecos pequeños
gradiente = cv2.morphologyEx(binaria, cv2.MORPH_GRADIENT, nucleo)  # Contorno de los objetos
```

| Operación | Efecto |
|---|---|
| Erosión | Adelgaza objetos, elimina ruido pequeño |
| Dilatación | Engrosa objetos, une fragmentos |
| Apertura (erosión + dilatación) | Elimina manchas pequeñas sin alterar el tamaño |
| Cierre (dilatación + erosión) | Rellena huecos y grietas pequeñas |

---

## 10. Bordes, contornos y medición de formas

### 10.1 Bordes

```python
bordes = cv2.Canny(cv2.GaussianBlur(gris, (5, 5), 0), 50, 150)     # Umbrales inferior y superior
sobel_x = cv2.Sobel(gris, cv2.CV_64F, 1, 0, ksize=3)               # Gradiente horizontal
```

### 10.2 Contornos

```python
contornos, jerarquia = cv2.findContours(binaria, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
```

| Modo | Devuelve |
|---|---|
| `RETR_EXTERNAL` | Solo contornos exteriores |
| `RETR_LIST` | Todos, sin jerarquía |
| `RETR_CCOMP` | Dos niveles: exteriores y sus agujeros |
| `RETR_TREE` | Jerarquía completa |

`jerarquia[0][i] = [siguiente, anterior, primer_hijo, padre]` permite saber qué contornos son agujeros de otros.

### 10.3 Medidas de cada contorno

```python
area = cv2.contourArea(c)
perimetro = cv2.arcLength(c, closed=True)
x, y, w, h = cv2.boundingRect(c)                       # Rectángulo alineado con los ejes
(cx, cy), (w, h), angulo = cv2.minAreaRect(c)          # Rectángulo girado mínimo
(cx, cy), radio = cv2.minEnclosingCircle(c)
casco = cv2.convexHull(c)
M = cv2.moments(c); centro = (M["m10"] / M["m00"], M["m01"] / M["m00"])
aproximado = cv2.approxPolyDP(c, 0.02 * perimetro, True)   # Polígono simplificado (nº de lados)
```

| Descriptor | Fórmula | Interpretación |
|---|---|---|
| Circularidad | 4π · área / perímetro² | 1 = círculo; menor = forma irregular |
| Solidez | área / área del casco convexo | < 1 indica mellas o entrantes |
| Relación de aspecto | ancho / alto | Formas alargadas |
| Extensión | área / área del rectángulo envolvente | Ocupación del rectángulo |

> En imágenes digitalizadas, el perímetro en píxeles se sobreestima: un círculo perfecto suele dar una circularidad de ≈ 0,90, no 1. Calibre los umbrales con piezas correctas.

---

## 11. Detección de formas, características y códigos

### 11.1 Líneas y círculos (transformada de Hough)

```python
lineas = cv2.HoughLinesP(bordes, 1, np.pi / 180, threshold=80, minLineLength=50, maxLineGap=10)
circulos = cv2.HoughCircles(cv2.GaussianBlur(gris, (9, 9), 2), cv2.HOUGH_GRADIENT, dp=1.2, minDist=30,
                            param1=100, param2=30, minRadius=10, maxRadius=100)
# circulos: array (1, N, 3) con (x, y, radio), o None
```

### 11.2 Búsqueda de plantillas

```python
resultado = cv2.matchTemplate(imagen, plantilla, cv2.TM_CCOEFF_NORMED)
_, maximo, _, posicion = cv2.minMaxLoc(resultado)       # posicion = esquina superior izquierda
```

### 11.3 Puntos característicos y emparejamiento

```python
orb = cv2.ORB_create(nfeatures=1000)                    # SIFT: cv2.SIFT_create()
puntos1, desc1 = orb.detectAndCompute(img1, None)
puntos2, desc2 = orb.detectAndCompute(img2, None)
emparejador = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
coincidencias = sorted(emparejador.match(desc1, desc2), key=lambda m: m.distance)
```

Con las coincidencias, `cv2.findHomography(..., cv2.RANSAC)` estima la transformación entre imágenes (panorámicas, localización de objetos, alineación de documentos).

### 11.4 Códigos QR y marcadores

```python
texto, puntos, _ = cv2.QRCodeDetector().detectAndDecode(imagen)
detector = cv2.aruco.ArucoDetector(cv2.aruco.getPredefinedDictionary(cv2.aruco.DICT_4X4_50))
esquinas, ids, _ = detector.detectMarkers(imagen)       # Marcadores ArUco (robótica, AR)
```

---

## 12. Dibujo y anotación

Las funciones de dibujo **modifican la imagen** recibida; trabaje sobre una copia si necesita el original.

```python
lienzo = imagen.copy()
cv2.line(lienzo, (0, 0), (200, 100), (255, 0, 0), 2)                      # Color BGR, grosor
cv2.rectangle(lienzo, (50, 50), (200, 150), (0, 255, 0), 3)
cv2.circle(lienzo, (300, 300), 40, (0, 0, 255), -1)                       # -1 = relleno
cv2.ellipse(lienzo, (400, 200), (80, 40), 30, 0, 360, (255, 255, 0), 2)
cv2.polylines(lienzo, [puntos.astype(np.int32)], isClosed=True, color=(0, 255, 255), thickness=2)
cv2.drawContours(lienzo, contornos, -1, (0, 255, 0), 2)                   # -1 = todos
cv2.putText(lienzo, "Pieza 1: OK", (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 1.0, (255, 255, 255), 2, cv2.LINE_AA)
```

> `putText` no admite tildes ni «ñ» con las fuentes Hershey. Para texto con caracteres especiales, dibuje con Pillow (`ImageDraw.text` con una fuente TrueType).

---

## 13. Vídeo y cámaras

```python
captura = cv2.VideoCapture(0)                          # Cámara 0, archivo "video.mp4" o URL RTSP
captura.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
fps = captura.get(cv2.CAP_PROP_FPS)

escritor = cv2.VideoWriter("salida.mp4", cv2.VideoWriter_fourcc(*"mp4v"), 25, (1280, 720))

while True:
    ok, fotograma = captura.read()
    if not ok:                                         # Fin del vídeo o error de cámara
        break
    procesado = procesar(fotograma)
    escritor.write(procesado)                          # Mismo tamaño que el indicado al crearlo
    cv2.imshow("Vídeo", procesado)
    if cv2.waitKey(1) & 0xFF == ord("q"):              # Salir con «q»
        break

captura.release(); escritor.release(); cv2.destroyAllWindows()
```

**Detección de movimiento** con sustracción de fondo:

```python
sustractor = cv2.createBackgroundSubtractorMOG2(history=500, varThreshold=16, detectShadows=True)
mascara_movimiento = sustractor.apply(fotograma)
```

Seguimiento de objetos: `cv2.TrackerCSRT_create()` (inicializar con `tracker.init(fotograma, rectangulo)` y actualizar con `tracker.update(fotograma)`) o el flujo óptico `cv2.calcOpticalFlowPyrLK`.

---

## 14. Aprendizaje profundo con el módulo `dnn`

OpenCV ejecuta redes neuronales entrenadas en otros marcos, principalmente en formato **ONNX**:

```python
red = cv2.dnn.readNetFromONNX("modelo.onnx")
blob = cv2.dnn.blobFromImage(imagen, scalefactor=1 / 255, size=(640, 640), swapRB=True)   # BGR → RGB
red.setInput(blob)
salida = red.forward()
red.setPreferableBackend(cv2.dnn.DNN_BACKEND_CUDA)        # Si OpenCV se compiló con CUDA
```

Detectores listos para usar:

```python
detector_caras = cv2.FaceDetectorYN.create("face_detection_yunet.onnx", "", (ancho, alto))
_, caras = detector_caras.detect(imagen)                  # Modelo YuNet del repositorio OpenCV Zoo
```

> En OpenCV 4, los clasificadores en cascada de Haar (`cv2.CascadeClassifier`) se distribuían en `cv2.data.haarcascades`. Los paquetes de OpenCV 5 no incluyen esos archivos XML: para detectar caras use `FaceDetectorYN`, y para objetos en general, modelos modernos (YOLO exportado a ONNX).

---

## 15. Calibración y medición en unidades reales

Para medir en milímetros hay que conocer la relación píxel–mm y corregir la distorsión de la lente:

```python
# 1. Calibración con fotografías de un tablero de ajedrez (patrón de 9 × 6 esquinas interiores)
encontrado, esquinas = cv2.findChessboardCorners(gris, (9, 6))
# ... acumular puntos 3D del patrón y 2D detectados en varias fotos ...
_, matriz_camara, distorsion, _, _ = cv2.calibrateCamera(puntos_3d, puntos_2d, gris.shape[::-1], None, None)

# 2. Corregir la distorsión de cada imagen
corregida = cv2.undistort(imagen, matriz_camara, distorsion)

# 3. Escala: fotografiar un objeto de tamaño conocido en el mismo plano
mm_por_pixel = 50.0 / ancho_en_pixeles_del_patron_de_50_mm
```

> En inspección industrial, la precisión depende tanto del software como del montaje: cámara perpendicular al plano, iluminación estable (retroiluminación para siluetas) y objetivo telecéntrico si se requiere precisión alta.

---

## 16. Ejemplo práctico completo

**Inspección visual de arandelas en una línea de producción**: genera una imagen sintética de una cinta transportadora con iluminación desigual y ruido, y doce arandelas de 40 mm, cinco de ellas con defectos (demasiado grande, sin agujero, con una mella, ovalada y con el agujero pequeño). El programa corrige la iluminación con una apertura morfológica, segmenta con Otsu y morfología, localiza cada pieza y su agujero con la **jerarquía de contornos**, mide diámetros en milímetros, **circularidad** y **solidez**, clasifica cada pieza según tolerancias, y guarda una imagen anotada y un informe CSV.

Requisitos: `pip install opencv-python numpy`

```python
"""
Inspección visual de arandelas en una línea de producción con OpenCV.

Genera una imagen sintética de una cinta transportadora (iluminación desigual
y ruido) con arandelas correctas y defectuosas, corrige la iluminación,
segmenta las piezas, mide cada una (diámetro exterior e interior en mm,
circularidad, solidez), las clasifica como correctas o defectuosas según las
tolerancias y guarda una imagen anotada y un informe CSV.
"""
import csv
import math

import cv2
import numpy as np

MM_POR_PIXEL = 0.25                    # Calibración: 1 píxel = 0,25 mm
DIAMETRO_EXT_MM = (39.0, 41.0)         # Tolerancias de la arandela M20
DIAMETRO_INT_MM = (19.0, 21.0)
CIRCULARIDAD_MIN = 0.88                # Un círculo digitalizado da ≈ 0,90 (perímetro en píxeles)
SOLIDEZ_MIN = 0.985


# ------------------------------------------------------------------ Imagen sintética
def generar_imagen(semilla: int = 5) -> np.ndarray:
    rng = np.random.default_rng(semilla)
    alto, ancho = 700, 1350
    y, x = np.mgrid[0:alto, 0:ancho]
    fondo = 70 + 50 * (x / ancho) + 25 * np.sin(y / 180)                  # Iluminación desigual
    imagen = np.dstack([fondo * 0.9, fondo, fondo * 1.05]).astype(np.uint8)

    piezas = [("ok", 80), ("ok", 80), ("ok", 80), ("ok", 80), ("ok", 80), ("ok", 80),
              ("grande", 89), ("sin_agujero", 80), ("mella", 80), ("ovalada", 80),
              ("agujero_pequeno", 80), ("ok", 80)]
    for i, (tipo, radio_ext) in enumerate(piezas):
        cx, cy = 140 + (i % 6) * 215 + rng.integers(-12, 13), 200 + (i // 6) * 300 + rng.integers(-12, 13)
        color = tuple(int(c) for c in rng.integers(185, 215, 3))
        if tipo == "ovalada":
            cv2.ellipse(imagen, (cx, cy), (radio_ext + 12, radio_ext - 14), 30, 0, 360, color, -1)
        else:
            cv2.circle(imagen, (cx, cy), radio_ext, color, -1)
        if tipo == "mella":                                                # Muesca en el borde
            cv2.circle(imagen, (cx + radio_ext, cy), 18, tuple(int(v) for v in imagen[cy, cx + radio_ext + 25]), -1)
        if tipo != "sin_agujero":
            radio_int = 28 if tipo == "agujero_pequeno" else 40
            cv2.circle(imagen, (cx, cy), radio_int, tuple(int(v) for v in imagen[cy, cx - radio_ext - 15]), -1)
    ruido = rng.normal(0, 8, imagen.shape)
    return np.clip(imagen + ruido, 0, 255).astype(np.uint8)


# ------------------------------------------------------------------ Procesado
def segmentar(imagen: np.ndarray) -> np.ndarray:
    gris = cv2.cvtColor(imagen, cv2.COLOR_BGR2GRAY)
    gris = cv2.GaussianBlur(gris, (5, 5), 0)                               # Reducir ruido
    fondo = cv2.morphologyEx(gris, cv2.MORPH_OPEN,                         # Estimar la iluminación
                             cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (251, 251)))
    corregida = cv2.subtract(gris, fondo)                                  # Iluminación uniforme
    _, binaria = cv2.threshold(corregida, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
    nucleo = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
    binaria = cv2.morphologyEx(binaria, cv2.MORPH_OPEN, nucleo, iterations=2)    # Quitar puntos sueltos
    return cv2.morphologyEx(binaria, cv2.MORPH_CLOSE, nucleo, iterations=2)      # Cerrar huecos pequeños


def medir_piezas(binaria: np.ndarray) -> list[dict]:
    contornos, jerarquia = cv2.findContours(binaria, cv2.RETR_CCOMP, cv2.CHAIN_APPROX_NONE)
    piezas = []
    for i, contorno in enumerate(contornos):
        if jerarquia[0][i][3] != -1 or cv2.contourArea(contorno) < 2_000:  # Solo contornos exteriores grandes
            continue
        area = cv2.contourArea(contorno)
        perimetro = cv2.arcLength(contorno, closed=True)
        (cx, cy), radio = cv2.minEnclosingCircle(contorno)
        casco = cv2.convexHull(contorno)
        hijo = jerarquia[0][i][2]                                          # Primer contorno interior (agujero)
        agujeros = []
        while hijo != -1:
            if cv2.contourArea(contornos[hijo]) > 200:
                agujeros.append(contornos[hijo])
            hijo = jerarquia[0][hijo][0]
        diametro_int = (2 * math.sqrt(cv2.contourArea(agujeros[0]) / math.pi) * MM_POR_PIXEL
                        if agujeros else 0.0)
        piezas.append({
            "x": int(cx), "y": int(cy), "contorno": contorno,
            "diametro_ext_mm": 2 * radio * MM_POR_PIXEL,
            "diametro_int_mm": diametro_int,
            "circularidad": 4 * math.pi * area / perimetro ** 2,           # 1 = círculo perfecto
            "solidez": area / cv2.contourArea(casco),                      # < 1 si hay mellas
        })
    return sorted(piezas, key=lambda p: (p["y"] // 150, p["x"]))           # Orden de lectura


def clasificar(pieza: dict) -> list[str]:
    defectos = []
    if not DIAMETRO_EXT_MM[0] <= pieza["diametro_ext_mm"] <= DIAMETRO_EXT_MM[1]:
        defectos.append("diámetro exterior")
    if pieza["diametro_int_mm"] == 0:
        defectos.append("sin agujero")
    elif not DIAMETRO_INT_MM[0] <= pieza["diametro_int_mm"] <= DIAMETRO_INT_MM[1]:
        defectos.append("diámetro interior")
    if pieza["circularidad"] < CIRCULARIDAD_MIN:
        defectos.append("deformada")
    if pieza["solidez"] < SOLIDEZ_MIN:
        defectos.append("mella")
    return defectos


def anotar(imagen: np.ndarray, piezas: list[dict]) -> np.ndarray:
    resultado = imagen.copy()
    for n, p in enumerate(piezas, 1):
        color = (0, 200, 0) if not p["defectos"] else (0, 0, 255)         # BGR: verde / rojo
        cv2.drawContours(resultado, [p["contorno"]], -1, color, 4)
        texto = f"{n}: {'OK' if not p['defectos'] else 'NOK'}"
        cv2.putText(resultado, texto, (p["x"] - 50, p["y"] - 100), cv2.FONT_HERSHEY_SIMPLEX, 1.0, color, 2,
                    cv2.LINE_AA)
    cv2.putText(resultado, f"{sum(not p['defectos'] for p in piezas)}/{len(piezas)} correctas", (20, 45),
                cv2.FONT_HERSHEY_SIMPLEX, 1.3, (255, 255, 255), 3, cv2.LINE_AA)
    return resultado


def main() -> None:
    print(f"OpenCV {cv2.__version__}")
    imagen = generar_imagen()
    cv2.imwrite("cinta_original.png", imagen)

    binaria = segmentar(imagen)
    piezas = medir_piezas(binaria)
    for p in piezas:
        p["defectos"] = clasificar(p)

    print(f"Piezas detectadas: {len(piezas)}\n")
    print(f"{'Nº':>3}{'Ø ext (mm)':>12}{'Ø int (mm)':>12}{'Circul.':>9}{'Solidez':>9}  Resultado")
    for n, p in enumerate(piezas, 1):
        print(f"{n:>3}{p['diametro_ext_mm']:>12.2f}{p['diametro_int_mm']:>12.2f}{p['circularidad']:>9.3f}"
              f"{p['solidez']:>9.3f}  {'OK' if not p['defectos'] else 'NOK: ' + ', '.join(p['defectos'])}")
    correctas = sum(not p["defectos"] for p in piezas)
    print(f"\nRendimiento del lote: {correctas}/{len(piezas)} ({correctas / len(piezas):.0%})")

    cv2.imwrite("cinta_binaria.png", binaria)
    cv2.imwrite("cinta_inspeccion.png", anotar(imagen, piezas))
    with open("inspeccion.csv", "w", newline="", encoding="utf-8") as f:
        w = csv.writer(f, delimiter=";")
        w.writerow(["pieza", "diametro_ext_mm", "diametro_int_mm", "circularidad", "solidez", "defectos"])
        for n, p in enumerate(piezas, 1):
            w.writerow([n, f"{p['diametro_ext_mm']:.2f}", f"{p['diametro_int_mm']:.2f}",
                        f"{p['circularidad']:.3f}", f"{p['solidez']:.3f}", " | ".join(p["defectos"])])
    print("Guardado: cinta_original.png, cinta_binaria.png, cinta_inspeccion.png, inspeccion.csv")


if __name__ == "__main__":
    main()
```

Resultado:

```
OpenCV 5.0.0
Piezas detectadas: 12

 Nº  Ø ext (mm)  Ø int (mm)  Circul.  Solidez  Resultado
  1       40.21       20.12    0.896    0.993  OK
  2       40.11       20.20    0.898    0.994  OK
  3       40.00       20.21    0.900    0.994  OK
  4       40.00       20.23    0.898    0.994  OK
  5       39.94       20.41    0.891    0.993  OK
  6       39.87       20.49    0.894    0.993  OK
  7       44.66       20.12    0.905    0.995  NOK: diámetro exterior
  8       40.08        0.00    0.899    0.994  NOK: sin agujero
  9       40.05       20.21    0.833    0.975  NOK: deformada, mella
 10       46.14       20.23    0.863    0.994  NOK: diámetro exterior, deformada
 11       40.00       14.24    0.892    0.994  NOK: diámetro interior
 12       39.88       20.37    0.895    0.993  OK

Rendimiento del lote: 7/12 (58%)
Guardado: cinta_original.png, cinta_binaria.png, cinta_inspeccion.png, inspeccion.csv
```

La imagen `cinta_inspeccion.png` muestra cada arandela con su contorno en verde (correcta) o en rojo (defectuosa) y su número. Cada uno de los cinco defectos se detecta por el criterio adecuado: el diámetro exterior para la pieza grande, la ausencia de contorno interior para la pieza sin agujero, la solidez para la mella, la circularidad para la ovalada y el diámetro interior para la del agujero pequeño.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `AttributeError: 'NoneType' object has no attribute 'shape'` | `imread` no encontró la imagen (ruta incorrecta, tildes en Windows). | Compruebe `if imagen is None`; use `imdecode(np.fromfile(...))`. |
| Colores invertidos al mostrar con Matplotlib | OpenCV usa BGR. | `cv2.cvtColor(img, cv2.COLOR_BGR2RGB)`. |
| `error: (-215:Assertion failed) !_src.empty()` | Imagen vacía pasada a una función. | Verifique la lectura y los recortes. |
| `error: ... (-215:Assertion failed) ... depth == CV_8U` | Tipo de datos no admitido (p. ej. `float64` o `int64`). | Convierta con `.astype(np.uint8)` o `cv2.convertScaleAbs`. |
| Recorte con coordenadas cambiadas | Confusión entre `[y, x]` (NumPy) y `(x, y)` (OpenCV). | Recorte con `img[y1:y2, x1:x2]`; puntos con `(x, y)`. |
| `findContours` no encuentra objetos o encuentra el fondo | Objetos oscuros sobre fondo claro. | Use `THRESH_BINARY_INV`; los objetos deben ser blancos. |
| `cv2.imshow` falla o no abre ventana | Paquete *headless* o servidor sin pantalla. | Use `opencv-python` en escritorio, o guarde con `imwrite` / muestre con Matplotlib. |
| La ventana no se dibuja o se congela | Falta `cv2.waitKey`. | Llame a `cv2.waitKey(1)` en bucles y `waitKey(0)` para imágenes. |
| El vídeo guardado está vacío o no se reproduce | Tamaño de fotograma distinto al del `VideoWriter` o códec no disponible. | Use el mismo tamaño y pruebe `mp4v` o `XVID` (`.avi`). |
| `module 'cv2' has no attribute 'xfeatures2d'` (u otras funciones avanzadas) | Funciones del módulo *contrib*. | Instale `opencv-contrib-python` (y desinstale los otros paquetes). |
| `CascadeClassifier` vacío (`empty()` devuelve `True`) | Archivo XML no encontrado (no incluido en los paquetes de OpenCV 5). | Descargue el XML o use `FaceDetectorYN`. |
| Tildes y «ñ» se ven como «??» en `putText` | Las fuentes Hershey solo tienen ASCII. | Dibuje el texto con Pillow. |

---

## 18. Buenas prácticas

- Compruebe siempre que las imágenes se han leído (`is None`) antes de procesarlas.
- Recuerde las dos convenciones: **BGR** y **`(x, y)` frente a `[fila, columna]`**.
- **Controle la iluminación** en la captura: ahorra más trabajo que cualquier algoritmo.
- Reduzca ruido antes de umbralizar o detectar bordes, y corrija la iluminación desigual.
- Prefiera umbrales **automáticos o adaptativos** a valores fijos, y calibre los criterios de aceptación con piezas correctas reales.
- Encapsule cada etapa (preprocesado, segmentación, medición, decisión) en funciones probadas por separado.
- Guarde imágenes intermedias (binaria, anotada) para depurar y para auditoría de calidad.
- Para tareas de reconocimiento complejas, combine OpenCV (captura y preprocesado) con modelos de aprendizaje profundo.
- Use el paquete *headless* en servidores y contenedores.

---

## 19. Referencia rápida

```python
import cv2, numpy as np
img = cv2.imread(ruta[, cv2.IMREAD_GRAYSCALE]) · cv2.imwrite(ruta, img) · cv2.imshow(n, img); cv2.waitKey(0)
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY | COLOR_BGR2RGB | COLOR_BGR2HSV) · cv2.inRange(hsv, bajo, alto)
cv2.resize(img, (w, h)) · img[y1:y2, x1:x2] · cv2.flip · cv2.rotate · cv2.warpAffine · cv2.warpPerspective
cv2.GaussianBlur(img, (5, 5), 0) · cv2.medianBlur · cv2.bilateralFilter · cv2.createCLAHE(2.0, (8, 8)).apply(gris)
cv2.threshold(gris, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU) · cv2.adaptiveThreshold · cv2.connectedComponentsWithStats
cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5)) · cv2.morphologyEx(b, cv2.MORPH_OPEN | MORPH_CLOSE, k)
cv2.Canny(gris, 50, 150) · cv2.findContours(b, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
cv2.contourArea · cv2.arcLength · cv2.boundingRect · cv2.minAreaRect · cv2.minEnclosingCircle · cv2.convexHull · cv2.moments
cv2.HoughCircles · cv2.HoughLinesP · cv2.matchTemplate · cv2.ORB_create · cv2.QRCodeDetector · cv2.aruco
cv2.line · rectangle · circle · putText · drawContours
cv2.VideoCapture(0 | ruta) · cap.read() · cv2.VideoWriter(ruta, cv2.VideoWriter_fourcc(*"mp4v"), fps, (w, h))
cv2.dnn.readNetFromONNX · cv2.dnn.blobFromImage · cv2.FaceDetectorYN.create
```

---

## 20. Ejercicios propuestos

1. Escriba un programa que cuente monedas en una fotografía con `HoughCircles` y estime el importe total según el diámetro de cada moneda.
2. Detecte un documento fotografiado en ángulo, localice sus cuatro esquinas con `approxPolyDP` y enderécelo con `warpPerspective`.
3. Cree un contador de vehículos para un vídeo de tráfico con `createBackgroundSubtractorMOG2` y una línea virtual de conteo.
4. Lea códigos QR de una carpeta de imágenes de albaranes y genere un CSV con el contenido de cada uno.
5. Amplíe el ejemplo completo para procesar un vídeo de la cinta fotograma a fotograma y contar las piezas correctas y defectuosas que pasan.

---

## 21. Referencias

- Documentación oficial: https://docs.opencv.org/
- Tutoriales de OpenCV en Python: https://docs.opencv.org/master/d6/d00/tutorial_py_root.html
- Paquetes para Python: https://github.com/opencv/opencv-python
- Modelos preentrenados (OpenCV Zoo): https://github.com/opencv/opencv_zoo
- Sitio oficial: https://opencv.org/
