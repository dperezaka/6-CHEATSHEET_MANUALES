# Manual de uso: PyTorch

> **Categoría:** Inteligencia artificial · **Utilidad:** Deep Learning e investigación en IA
> **Tipo:** Librería de terceros (requiere instalación) · **Licencia:** BSD · **Gobernanza:** PyTorch Foundation (Linux Foundation)

---

## Índice

1. [Descripción](#1-descripción)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Conceptos clave](#3-conceptos-clave)
4. [Tensores](#4-tensores)
5. [Diferenciación automática (autograd)](#5-diferenciación-automática-autograd)
6. [Definir redes con `nn.Module`](#6-definir-redes-con-nnmodule)
7. [Datos: `Dataset` y `DataLoader`](#7-datos-dataset-y-dataloader)
8. [Funciones de pérdida y optimizadores](#8-funciones-de-pérdida-y-optimizadores)
9. [El bucle de entrenamiento](#9-el-bucle-de-entrenamiento)
10. [GPU y dispositivos](#10-gpu-y-dispositivos)
11. [Rendimiento: precisión mixta y `torch.compile`](#11-rendimiento-precisión-mixta-y-torchcompile)
12. [Aprendizaje por transferencia](#12-aprendizaje-por-transferencia)
13. [Guardar, cargar y exportar](#13-guardar-cargar-y-exportar)
14. [Entrenamiento distribuido y escalado](#14-entrenamiento-distribuido-y-escalado)
15. [Ecosistema](#15-ecosistema)
16. [Ejemplo práctico completo](#16-ejemplo-práctico-completo)
17. [Errores frecuentes](#17-errores-frecuentes)
18. [Buenas prácticas](#18-buenas-prácticas)
19. [Referencia rápida](#19-referencia-rápida)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Referencias](#21-referencias)

---

## 1. Descripción

**PyTorch** es una librería de aprendizaje profundo que combina tensores acelerados por GPU con diferenciación automática y un estilo de programación **imperativo y «pythónico»**: los modelos son clases de Python y el código se ejecuta línea a línea, lo que facilita la experimentación y la depuración. Es el marco dominante en investigación y la base de la mayoría de los modelos de lenguaje e IA generativa actuales.

**Ventajas:**
- Código natural en Python: se depura con `print`, `pdb` o el depurador del IDE.
- Flexibilidad total para arquitecturas nuevas (bucles, condiciones y estructuras dinámicas dentro del modelo).
- Ecosistema enorme: Hugging Face Transformers, torchvision, torchaudio, Lightning, timm…
- Soporte de GPU NVIDIA (CUDA), AMD (ROCm), Apple Silicon (MPS) e Intel.
- `torch.compile` acelera los modelos sin cambiar el código.

**Limitaciones:**
- Hay que escribir el bucle de entrenamiento (o usar librerías como Lightning o Accelerate).
- El despliegue en móviles y navegadores es menos directo que con TensorFlow.
- No incluye herramientas de alto nivel para datos tabulares clásicos.

| Si necesita… | Opción |
|---|---|
| Investigación, modelos propios, IA generativa, modelos de lenguaje | **PyTorch** |
| Modelos preentrenados de texto, visión y audio | Hugging Face Transformers (sobre PyTorch) |
| Entrenamiento con menos código repetitivo | PyTorch Lightning / Keras 3 con backend PyTorch |
| Despliegue en móviles y navegadores | TensorFlow / LiteRT, ONNX Runtime |
| Datos tabulares | scikit-learn, XGBoost, LightGBM, CatBoost |

---

## 2. Instalación e importación

La orden exacta depende del sistema y de la GPU; genérela en el selector de https://pytorch.org/get-started/locally/.

```bash
pip install torch torchvision                                        # Por defecto
pip install torch --index-url https://download.pytorch.org/whl/cpu   # Solo CPU (más ligero)
```

Comprobar la instalación y la aceleración disponible:

```python
import torch
print(torch.__version__)
print(torch.cuda.is_available())            # GPU NVIDIA / AMD (ROCm)
print(torch.backends.mps.is_available())    # Apple Silicon
```

Importaciones habituales:

```python
import torch
from torch import nn
import torch.nn.functional as F
from torch.utils.data import Dataset, DataLoader
```

---

## 3. Conceptos clave

| Concepto | Descripción |
|---|---|
| Tensor | Array multidimensional (similar a NumPy) que puede residir en CPU o GPU. |
| `requires_grad` | Indica que se deben calcular gradientes respecto a ese tensor. |
| Autograd | Motor que registra las operaciones y calcula gradientes con `backward()`. |
| `nn.Module` | Clase base de capas y modelos; contiene parámetros y el método `forward`. |
| Parámetro | Tensor entrenable de un módulo (`nn.Parameter`). |
| `Dataset` | Acceso a las muestras por índice. |
| `DataLoader` | Agrupa muestras en lotes, baraja y carga en paralelo. |
| Optimizador | Actualiza los parámetros con los gradientes (SGD, Adam, AdamW). |
| Modo `train()` / `eval()` | Cambia el comportamiento de `Dropout` y `BatchNorm`. |
| `state_dict` | Diccionario con los parámetros y estados; base para guardar y cargar. |
| Logits | Salida sin normalizar de la red (antes de *softmax*). |

**Ciclo de cada paso de entrenamiento:**

```
lote ─► modelo(x) ─► pérdida(pred, y) ─► optimizador.zero_grad() ─► pérdida.backward() ─► optimizador.step()
```

---

## 4. Tensores

### 4.1 Crear tensores

```python
a = torch.tensor([[1.0, 2.0], [3.0, 4.0]])
torch.zeros(2, 3); torch.ones(2, 3); torch.eye(3); torch.arange(10); torch.linspace(0, 1, 5)
torch.rand(3, 3); torch.randn(3, 3); torch.randint(0, 10, (5,))
torch.zeros_like(a); torch.empty(2, 2)
torch.from_numpy(array_numpy)       # Comparte memoria con NumPy
a.numpy()                           # Conversión a NumPy (en CPU)
a.item()                            # Escalar de Python (tensores de un elemento)
```

### 4.2 Atributos

```python
a.shape        # torch.Size([2, 2])
a.dtype        # torch.float32
a.device       # device(type='cpu')
```

> `torch.from_numpy` mantiene el tipo de NumPy (`float64` por defecto), mientras que las redes usan `float32`. Convierta con `.float()` o `dtype=torch.float32`.

### 4.3 Operaciones

```python
a + b; a * b; a @ b; torch.matmul(a, b)
a.sum(); a.mean(dim=0); a.max(dim=1); a.argmax(dim=1); a.std()
a.T; a.reshape(4, 1); a.view(-1); a.unsqueeze(0); a.squeeze(); a.permute(0, 2, 1)
torch.cat([a, b], dim=0); torch.stack([a, b])
torch.where(a > 2, a, 0.0); a.clamp(0, 1)
a[a > 2]; a[:, 0]
```

> `view` requiere memoria contigua; `reshape` funciona siempre (copia si es necesario). Los métodos terminados en guion bajo modifican el tensor en el sitio: `a.add_(1)`.

---

## 5. Diferenciación automática (autograd)

```python
x = torch.tensor(2.0, requires_grad=True)
y = x ** 3
y.backward()             # Calcula dy/dx
x.grad                   # tensor(12.)
```

| Herramienta | Uso |
|---|---|
| `tensor.backward()` | Calcula gradientes de un escalar respecto a las hojas con `requires_grad=True` |
| `tensor.grad` | Gradiente acumulado (se **suma** en cada `backward`) |
| `optimizador.zero_grad()` | Pone a cero los gradientes antes de cada paso |
| `with torch.no_grad():` | Desactiva el registro de operaciones (evaluación, ahorro de memoria) |
| `with torch.inference_mode():` | Como `no_grad`, más rápido; solo para inferencia |
| `tensor.detach()` | Copia sin historial de gradientes |
| `torch.autograd.grad(y, x)` | Gradiente sin acumular en `.grad` |

---

## 6. Definir redes con `nn.Module`

### 6.1 Modelo como clase

```python
class Clasificador(nn.Module):
    def __init__(self, n_entradas: int, n_clases: int):
        super().__init__()
        self.oculta = nn.Linear(n_entradas, 64)
        self.dropout = nn.Dropout(0.2)
        self.salida = nn.Linear(64, n_clases)

    def forward(self, x):
        x = F.relu(self.oculta(x))
        x = self.dropout(x)
        return self.salida(x)                # Logits

modelo = Clasificador(20, 3)
logits = modelo(torch.randn(8, 20))         # Llamar al modelo, no a forward()
```

### 6.2 Modelo secuencial

```python
modelo = nn.Sequential(
    nn.Linear(4, 8), nn.ReLU(),
    nn.Linear(8, 1),
)
```

### 6.3 Capas principales

| Capa | Uso |
|---|---|
| `nn.Linear(entrada, salida)` | Totalmente conectada |
| `nn.Conv1d / Conv2d / Conv3d` | Convoluciones (series, imágenes, vídeo) |
| `nn.MaxPool2d`, `nn.AdaptiveAvgPool2d` | Reducción espacial |
| `nn.BatchNorm1d/2d`, `nn.LayerNorm` | Normalización |
| `nn.Dropout` | Regularización |
| `nn.Embedding(n, dim)` | Vectores para índices (palabras, categorías) |
| `nn.LSTM`, `nn.GRU` | Redes recurrentes |
| `nn.TransformerEncoder`, `nn.MultiheadAttention` | *Transformers* y atención |
| `nn.ReLU`, `nn.GELU`, `nn.SiLU`, `nn.Sigmoid`, `nn.Softmax` | Activaciones |
| `nn.Flatten` | Aplanar a vector |

### 6.4 Inspeccionar el modelo

```python
print(modelo)
sum(p.numel() for p in modelo.parameters() if p.requires_grad)    # Parámetros entrenables
for nombre, parametro in modelo.named_parameters(): ...
modelo.state_dict().keys()
```

---

## 7. Datos: `Dataset` y `DataLoader`

### 7.1 `Dataset` propio

```python
class DatosClientes(Dataset):
    def __init__(self, X, y):
        self.X = torch.tensor(X, dtype=torch.float32)
        self.y = torch.tensor(y, dtype=torch.long)

    def __len__(self):
        return len(self.y)

    def __getitem__(self, i):
        return self.X[i], self.y[i]
```

Para tensores ya en memoria: `TensorDataset(X, y)`.

### 7.2 `DataLoader`

```python
cargador = DataLoader(
    datos,
    batch_size=64,
    shuffle=True,           # Barajar en cada época (solo entrenamiento)
    num_workers=4,          # Procesos de carga en paralelo
    pin_memory=True,        # Transferencias a GPU más rápidas
    drop_last=False,
)
for x, y in cargador:
    ...
```

> En Windows y macOS, con `num_workers > 0` el código principal debe estar dentro de `if __name__ == "__main__":`.

### 7.3 Utilidades

```python
from torch.utils.data import random_split, Subset, ConcatDataset, WeightedRandomSampler

entrenamiento, validacion = random_split(datos, [0.8, 0.2], generator=torch.Generator().manual_seed(42))
```

Para imágenes, `torchvision` aporta conjuntos de datos (`ImageFolder`) y transformaciones (`torchvision.transforms.v2`).

---

## 8. Funciones de pérdida y optimizadores

### 8.1 Pérdidas

| Problema | Pérdida | Salida del modelo |
|---|---|---|
| Clasificación multiclase | `nn.CrossEntropyLoss()` | Logits (sin *softmax*) y etiquetas enteras |
| Clasificación binaria / multietiqueta | `nn.BCEWithLogitsLoss()` | Logits (sin *sigmoid*) y etiquetas 0/1 `float` |
| Regresión | `nn.MSELoss()`, `nn.L1Loss()`, `nn.HuberLoss()` | Valores |

> `CrossEntropyLoss` y `BCEWithLogitsLoss` aplican internamente *softmax*/*sigmoid*: **no** añada esas activaciones al final del modelo.

Opciones útiles: `weight=` (pesos de clase), `label_smoothing=0.1`, `pos_weight=` (BCE desbalanceada).

### 8.2 Optimizadores y programación de la tasa de aprendizaje

```python
optimizador = torch.optim.AdamW(modelo.parameters(), lr=1e-3, weight_decay=1e-2)
# Otros: SGD(lr, momentum=0.9), Adam, RMSprop

programador = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizador, factor=0.5, patience=3)
programador = torch.optim.lr_scheduler.CosineAnnealingLR(optimizador, T_max=epocas)
programador = torch.optim.lr_scheduler.OneCycleLR(optimizador, max_lr=1e-2, total_steps=pasos)
```

---

## 9. El bucle de entrenamiento

```python
def entrenar_epoca(modelo, cargador, perdida_fn, optimizador, dispositivo):
    modelo.train()                                  # Activa Dropout y BatchNorm en modo entrenamiento
    for x, y in cargador:
        x, y = x.to(dispositivo), y.to(dispositivo)
        optimizador.zero_grad()                     # 1. Limpiar gradientes
        perdida = perdida_fn(modelo(x), y)          # 2. Propagación hacia delante
        perdida.backward()                          # 3. Retropropagación
        torch.nn.utils.clip_grad_norm_(modelo.parameters(), 1.0)   # (opcional) recorte
        optimizador.step()                          # 4. Actualizar pesos

@torch.no_grad()
def evaluar(modelo, cargador, dispositivo):
    modelo.eval()                                   # Desactiva Dropout; BatchNorm usa estadísticas fijas
    aciertos = 0
    for x, y in cargador:
        pred = modelo(x.to(dispositivo)).argmax(dim=1)
        aciertos += (pred.cpu() == y).sum().item()
    return aciertos / len(cargador.dataset)
```

> Olvidar `modelo.eval()` en la evaluación o `zero_grad()` en el entrenamiento son los dos errores más comunes de PyTorch.

**Parada temprana** (manual): guarde `copy.deepcopy(modelo.state_dict())` cuando mejore la pérdida de validación y deténgase tras N épocas sin mejora (véase el ejemplo completo).

---

## 10. GPU y dispositivos

```python
dispositivo = ("cuda" if torch.cuda.is_available()
               else "mps" if torch.backends.mps.is_available() else "cpu")

modelo = modelo.to(dispositivo)
x = x.to(dispositivo)
resultado = modelo(x).cpu()          # Volver a CPU para NumPy o para mostrar

torch.cuda.device_count(); torch.cuda.get_device_name(0)
torch.cuda.memory_allocated(); torch.cuda.empty_cache()
```

> El modelo y los datos deben estar **en el mismo dispositivo**. Mueva los lotes dentro del bucle, no el conjunto completo.

---

## 11. Rendimiento: precisión mixta y `torch.compile`

### 11.1 Precisión mixta automática (AMP)

```python
escalador = torch.amp.GradScaler("cuda")          # Solo necesario con float16

for x, y in cargador:
    optimizador.zero_grad()
    with torch.autocast(device_type="cuda", dtype=torch.float16):   # o torch.bfloat16
        perdida = perdida_fn(modelo(x), y)
    escalador.scale(perdida).backward()
    escalador.step(optimizador)
    escalador.update()
```

Con `bfloat16` (GPU recientes y CPU) no se necesita `GradScaler`.

### 11.2 `torch.compile`

```python
modelo_compilado = torch.compile(modelo)          # Compila al primer uso; acelera el entrenamiento
```

La primera llamada tarda más (compilación). La mejora es mayor en GPU y con modelos grandes.

### 11.3 Otros consejos

- Aumente `batch_size` hasta aprovechar la memoria de la GPU.
- Use `num_workers` y `pin_memory=True` para que la carga de datos no limite.
- `torch.backends.cudnn.benchmark = True` cuando el tamaño de las entradas es fijo.

---

## 12. Aprendizaje por transferencia

Reutilizar una red preentrenada y adaptarla a un problema propio con pocos datos:

```python
from torchvision.models import resnet18, ResNet18_Weights

pesos = ResNet18_Weights.DEFAULT
modelo = resnet18(weights=pesos)                 # Descarga los pesos la primera vez
transformacion = pesos.transforms()              # Preprocesado que espera el modelo

for parametro in modelo.parameters():
    parametro.requires_grad = False              # Congelar la red base
modelo.fc = nn.Linear(modelo.fc.in_features, 5)  # Nueva capa de salida (5 clases), entrenable

optimizador = torch.optim.AdamW(modelo.fc.parameters(), lr=1e-3)
```

Tras unas épocas, puede descongelar las últimas capas y continuar con una tasa de aprendizaje menor (*fine-tuning*). Para texto, use **Hugging Face Transformers**.

---

## 13. Guardar, cargar y exportar

### 13.1 Pesos (recomendado)

```python
torch.save(modelo.state_dict(), "modelo.pt")

modelo = Clasificador(20, 3)                                  # Recrear la arquitectura
modelo.load_state_dict(torch.load("modelo.pt", weights_only=True))
modelo.eval()
```

### 13.2 Puntos de control para reanudar el entrenamiento

```python
torch.save({"epoca": epoca, "modelo": modelo.state_dict(),
            "optimizador": optimizador.state_dict()}, "checkpoint.pt")
ckpt = torch.load("checkpoint.pt", weights_only=True)
```

> Desde PyTorch 2.6, `torch.load` usa `weights_only=True` por defecto: solo carga tensores y tipos básicos. **No** desactive esta protección con archivos de origen desconocido: podrían ejecutar código.

### 13.3 Exportar para producción

| Formato | Cómo | Uso |
|---|---|---|
| `torch.export` (`.pt2`) | `torch.export.export(modelo, (ejemplo,))` + `torch.export.save` | Grafo completo; base de los despliegues modernos |
| ONNX | `torch.onnx.export(modelo, (ejemplo,), "modelo.onnx", dynamo=True)` (requiere `onnx` y `onnxscript`) | ONNX Runtime, otros lenguajes y plataformas |
| ExecuTorch | A partir de `torch.export` | Móviles y dispositivos embebidos |

> TorchScript (`torch.jit.script`/`trace`) sigue funcionando pero ya no se desarrolla activamente; para nuevos proyectos se recomienda `torch.export`.

---

## 14. Entrenamiento distribuido y escalado

| Herramienta | Uso |
|---|---|
| `DistributedDataParallel` (DDP) | Varias GPU o máquinas, una copia del modelo por GPU |
| FSDP | Modelos que no caben en una GPU (reparte parámetros y estados) |
| `torchrun` | Lanzador de procesos distribuidos |
| Hugging Face **Accelerate** | Mismo código para CPU, una GPU, varias GPU o TPU |
| **PyTorch Lightning** | Estructura el código y gestiona distribución, AMP y registros |

```bash
torchrun --nproc_per_node=4 entrenar.py
```

---

## 15. Ecosistema

| Librería | Uso |
|---|---|
| **torchvision** | Modelos, conjuntos de datos y transformaciones de imagen |
| **torchaudio** | Audio y voz |
| **Hugging Face Transformers** | Modelos de lenguaje y multimodales preentrenados |
| **timm** | Cientos de modelos de visión preentrenados |
| **PyTorch Lightning** | Entrenamiento estructurado con menos código |
| **TorchMetrics** | Métricas compatibles con GPU y distribución |
| **PyTorch Geometric** | Redes neuronales de grafos |
| **TensorBoard / Weights & Biases / MLflow** | Seguimiento de experimentos |
| **ONNX Runtime / TorchServe / ExecuTorch** | Despliegue |

---

## 16. Ejemplo práctico completo

**Clasificación de dígitos manuscritos con una red convolucional**: usa el conjunto *digits* incluido en scikit-learn (no requiere descargas), un `Dataset` propio con **aumento de datos** (desplazamientos y ruido), una CNN con `BatchNorm` y `Dropout`, entrenamiento con AdamW, suavizado de etiquetas, reducción de la tasa de aprendizaje y **parada temprana**, evaluación con informe por clase y matriz de confusión, punto de control y exportación con `torch.export`.

Requisitos: `pip install torch scikit-learn`

```python
"""
Clasificación de dígitos manuscritos con una red convolucional en PyTorch.

Usa el conjunto «digits» incluido en scikit-learn (1 797 imágenes de 8×8),
Dataset y DataLoader propios con aumento de datos, una CNN con BatchNorm y
Dropout, bucle de entrenamiento con AdamW, programación de la tasa de
aprendizaje y parada temprana, evaluación detallada, guardado de puntos de
control y exportación con torch.export.
"""
import copy
import time

import numpy as np
import torch
from sklearn.datasets import load_digits
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.model_selection import train_test_split
from torch import nn
from torch.utils.data import DataLoader, Dataset

torch.manual_seed(42)
DISPOSITIVO = ("cuda" if torch.cuda.is_available()
               else "mps" if torch.backends.mps.is_available() else "cpu")
LOTE, EPOCAS_MAX, PACIENCIA = 64, 60, 8


# ------------------------------------------------------------------ Datos
class Digitos(Dataset):
    """Imágenes 8×8 en escala de grises (0-16) con aumento de datos opcional."""

    def __init__(self, imagenes: np.ndarray, etiquetas: np.ndarray, aumentar: bool = False):
        self.x = torch.tensor(imagenes / 16.0, dtype=torch.float32).unsqueeze(1)   # (N, 1, 8, 8)
        self.y = torch.tensor(etiquetas, dtype=torch.long)
        self.aumentar = aumentar

    def __len__(self) -> int:
        return len(self.y)

    def __getitem__(self, i: int):
        x = self.x[i]
        if self.aumentar:
            dx, dy = torch.randint(-1, 2, (2,)).tolist()                 # Desplazamiento de ±1 píxel
            x = torch.roll(x, shifts=(dy, dx), dims=(1, 2))
            x = (x + 0.05 * torch.randn_like(x)).clamp(0, 1)             # Ruido
        return x, self.y[i]


# ------------------------------------------------------------------ Modelo
class RedDigitos(nn.Module):
    def __init__(self, n_clases: int = 10):
        super().__init__()
        self.caracteristicas = nn.Sequential(
            nn.Conv2d(1, 32, kernel_size=3, padding=1), nn.BatchNorm2d(32), nn.ReLU(),
            nn.Conv2d(32, 64, kernel_size=3, padding=1), nn.BatchNorm2d(64), nn.ReLU(),
            nn.MaxPool2d(2),                                              # 8×8 → 4×4
            nn.Conv2d(64, 128, kernel_size=3, padding=1), nn.BatchNorm2d(128), nn.ReLU(),
            nn.AdaptiveAvgPool2d(1),                                      # → 128×1×1
        )
        self.clasificador = nn.Sequential(nn.Flatten(), nn.Dropout(0.3), nn.Linear(128, n_clases))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.clasificador(self.caracteristicas(x))                # Devuelve logits


# ------------------------------------------------------------------ Entrenamiento
def entrenar_epoca(modelo, cargador, perdida_fn, optimizador) -> float:
    modelo.train()
    total = 0.0
    for x, y in cargador:
        x, y = x.to(DISPOSITIVO), y.to(DISPOSITIVO)
        optimizador.zero_grad()
        perdida = perdida_fn(modelo(x), y)
        perdida.backward()
        optimizador.step()
        total += perdida.item() * len(y)
    return total / len(cargador.dataset)


@torch.no_grad()
def evaluar(modelo, cargador, perdida_fn) -> tuple[float, float, np.ndarray]:
    modelo.eval()
    total, predicciones = 0.0, []
    for x, y in cargador:
        x, y = x.to(DISPOSITIVO), y.to(DISPOSITIVO)
        logits = modelo(x)
        total += perdida_fn(logits, y).item() * len(y)
        predicciones.append(logits.argmax(dim=1).cpu())
    pred = torch.cat(predicciones).numpy()
    exactitud = (pred == cargador.dataset.y.numpy()).mean()
    return total / len(cargador.dataset), exactitud, pred


def main() -> None:
    print(f"PyTorch {torch.__version__} · dispositivo: {DISPOSITIVO}")
    digitos = load_digits()
    imagenes = digitos.images.astype(np.float32)
    x_ent, x_test, y_ent, y_test = train_test_split(imagenes, digitos.target, test_size=0.2,
                                                    stratify=digitos.target, random_state=42)
    x_ent, x_val, y_ent, y_val = train_test_split(x_ent, y_ent, test_size=0.15,
                                                  stratify=y_ent, random_state=42)
    cargadores = {
        "ent": DataLoader(Digitos(x_ent, y_ent, aumentar=True), batch_size=LOTE, shuffle=True),
        "val": DataLoader(Digitos(x_val, y_val), batch_size=256),
        "test": DataLoader(Digitos(x_test, y_test), batch_size=256),
    }
    print(f"Imágenes · entrenamiento {len(y_ent)} · validación {len(y_val)} · prueba {len(y_test)}")

    modelo = RedDigitos().to(DISPOSITIVO)
    print(f"Parámetros entrenables: {sum(p.numel() for p in modelo.parameters() if p.requires_grad):,}\n")
    perdida_fn = nn.CrossEntropyLoss(label_smoothing=0.05)
    optimizador = torch.optim.AdamW(modelo.parameters(), lr=3e-3, weight_decay=1e-4)
    programador = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizador, mode="min", factor=0.5, patience=3)

    mejor_perdida, sin_mejora, mejor_estado = float("inf"), 0, None
    inicio = time.perf_counter()
    for epoca in range(1, EPOCAS_MAX + 1):
        perdida_ent = entrenar_epoca(modelo, cargadores["ent"], perdida_fn, optimizador)
        perdida_val, exactitud_val, _ = evaluar(modelo, cargadores["val"], perdida_fn)
        programador.step(perdida_val)
        if epoca % 5 == 0 or epoca == 1:
            print(f"Época {epoca:2d} · pérdida ent {perdida_ent:.4f} · val {perdida_val:.4f} · "
                  f"exactitud val {exactitud_val:.1%} · lr {optimizador.param_groups[0]['lr']:.1e}")
        if perdida_val < mejor_perdida - 1e-4:
            mejor_perdida, sin_mejora = perdida_val, 0
            mejor_estado = copy.deepcopy(modelo.state_dict())
        else:
            sin_mejora += 1
            if sin_mejora >= PACIENCIA:
                print(f"Parada temprana en la época {epoca}")
                break
    modelo.load_state_dict(mejor_estado)
    print(f"Entrenamiento: {time.perf_counter() - inicio:.1f} s\n")

    # Evaluación final
    _, exactitud, pred = evaluar(modelo, cargadores["test"], perdida_fn)
    print(f"Exactitud en prueba: {exactitud:.2%}\n")
    print(classification_report(y_test, pred, digits=3))
    matriz = confusion_matrix(y_test, pred)
    errores = [(i, j, matriz[i, j]) for i in range(10) for j in range(10) if i != j and matriz[i, j]]
    print("Confusiones:", ", ".join(f"{i}→{j} ({n})" for i, j, n in sorted(errores, key=lambda e: -e[2])))

    # Guardar punto de control y exportar el modelo
    torch.save({"modelo": modelo.state_dict(), "optimizador": optimizador.state_dict(),
                "exactitud": exactitud}, "digitos_ckpt.pt")
    modelo.eval().cpu()
    ejemplo = torch.randn(1, 1, 8, 8)
    exportado = torch.export.export(modelo, (ejemplo,))
    torch.export.save(exportado, "digitos.pt2")
    recargado = torch.export.load("digitos.pt2").module()
    print(f"\nModelo exportado y recargado · diferencia máxima "
          f"{(recargado(ejemplo) - modelo(ejemplo)).abs().max():.1e}")

    # Inferencia con probabilidades sobre una imagen nueva
    with torch.inference_mode():
        x = torch.tensor(x_test[:1] / 16.0, dtype=torch.float32).unsqueeze(1)
        prob = torch.softmax(modelo(x), dim=1)[0]
    top = torch.topk(prob, 3)
    print(f"Imagen de prueba (real: {y_test[0]}) · predicción: " +
          ", ".join(f"{c} ({p:.1%})" for p, c in zip(top.values.tolist(), top.indices.tolist())))


if __name__ == "__main__":
    main()
```

Resultado (CPU):

```
PyTorch 2.15.0+cu132 · dispositivo: cpu
Imágenes · entrenamiento 1221 · validación 216 · prueba 360
Parámetros entrenables: 94,410

Época  1 · pérdida ent 1.6018 · val 2.4173 · exactitud val 9.7% · lr 3.0e-03
Época  5 · pérdida ent 0.4722 · val 0.5945 · exactitud val 87.5% · lr 3.0e-03
Época 10 · pérdida ent 0.4593 · val 0.4804 · exactitud val 94.9% · lr 3.0e-03
Época 15 · pérdida ent 0.4226 · val 0.4392 · exactitud val 98.6% · lr 3.0e-03
Época 20 · pérdida ent 0.3728 · val 0.3755 · exactitud val 98.1% · lr 1.5e-03
Época 25 · pérdida ent 0.3596 · val 0.3587 · exactitud val 99.1% · lr 7.5e-04
Época 30 · pérdida ent 0.3562 · val 0.3547 · exactitud val 99.1% · lr 7.5e-04
Época 35 · pérdida ent 0.3483 · val 0.3606 · exactitud val 99.5% · lr 7.5e-04
Parada temprana en la época 39
Entrenamiento: 9.2 s

Exactitud en prueba: 98.89%

              precision    recall  f1-score   support

           0      1.000     1.000     1.000        36
           1      0.947     1.000     0.973        36
           2      1.000     1.000     1.000        35
           3      1.000     1.000     1.000        37
           4      1.000     1.000     1.000        36
           5      0.974     1.000     0.987        37
           6      1.000     0.972     0.986        36
           7      1.000     1.000     1.000        36
           8      1.000     0.914     0.955        35
           9      0.973     1.000     0.986        36

    accuracy                          0.989       360
   macro avg      0.989     0.989     0.989       360
weighted avg      0.989     0.989     0.989       360

Confusiones: 8→1 (2), 6→5 (1), 8→9 (1)

Modelo exportado y recargado · diferencia máxima 0.0e+00
Imagen de prueba (real: 5) · predicción: 5 (99.2%), 9 (0.4%), 0 (0.2%)
```

> La pérdida de validación inicial elevada (2,42 con un 9,7 % de exactitud) se debe a que `BatchNorm` aún no ha estimado sus estadísticas: se estabiliza en pocas épocas. La pérdida no baja de ~0,35 por el suavizado de etiquetas, que impide predicciones con probabilidad del 100 %.

---

## 17. Errores frecuentes

| Error | Causa | Solución |
|---|---|---|
| `RuntimeError: mat1 and mat2 shapes cannot be multiplied (2x4 and 3x2)` | El tamaño de entrada de una capa `Linear` no coincide con los datos. | Ajuste `in_features`; imprima `x.shape` antes de la capa. |
| `RuntimeError: Expected all tensors to be on the same device` | Modelo y datos en dispositivos distintos. | `.to(dispositivo)` en ambos. |
| `RuntimeError: expected scalar type Float but found Double` | Datos `float64` (NumPy). | `torch.tensor(x, dtype=torch.float32)` o `.float()`. |
| `RuntimeError: Expected target ... to be of type Long` | Etiquetas no enteras en `CrossEntropyLoss`. | `y.long()`. |
| La pérdida no baja | Falta `zero_grad()`, tasa de aprendizaje inadecuada, *softmax* antes de `CrossEntropyLoss`. | Revise el bucle; pruebe a sobreajustar un lote pequeño. |
| Resultados de evaluación inestables o peores | Olvido de `modelo.eval()`. | Llame a `eval()` antes de evaluar y a `train()` antes de entrenar. |
| `CUDA out of memory` | Lote demasiado grande o tensores retenidos con su grafo. | Reduzca el lote, use AMP, `torch.no_grad()` en evaluación, acumule con `.item()`. |
| La memoria crece en cada iteración | Se acumula `perdida` (tensor con grafo) en lugar de `perdida.item()`. | Use `.item()` o `.detach()`. |
| `Can't call numpy() on Tensor that requires grad` | Tensor con historial de gradientes. | `t.detach().cpu().numpy()`. |
| `UnpicklingError: Weights only load failed` | `torch.load` con objetos no permitidos (PyTorch ≥ 2.6). | Guarde `state_dict` en lugar del modelo completo; use `weights_only=False` solo con archivos de confianza. |
| `DataLoader` se bloquea en Windows/macOS | `num_workers > 0` sin protección `__main__`. | Use `if __name__ == "__main__":` o `num_workers=0`. |
| Resultados no reproducibles | Semillas no fijadas. | `torch.manual_seed`, generadores en `DataLoader` y `torch.use_deterministic_algorithms(True)`. |

---

## 18. Buenas prácticas

- Escriba código **independiente del dispositivo** (variable `dispositivo`) para ejecutar igual en CPU, GPU o MPS.
- Antes de entrenar a fondo, compruebe que el modelo **sobreajusta un lote pequeño**: si no lo hace, hay un error en el código.
- Use `modelo.train()` / `modelo.eval()` y `torch.no_grad()` / `torch.inference_mode()` correctamente.
- Normalice las entradas y verifique las formas de los tensores en cada etapa.
- Guarde **`state_dict`** y puntos de control periódicos; registre hiperparámetros y métricas.
- Use validación, parada temprana y programación de la tasa de aprendizaje.
- Aproveche la precisión mixta y `torch.compile` en GPU.
- Use modelos preentrenados siempre que existan para su tipo de datos.
- Fije semillas para la reproducibilidad durante el desarrollo.
- Para proyectos grandes, valore Lightning o Accelerate para evitar código repetitivo.

---

## 19. Referencia rápida

```python
import torch; from torch import nn; import torch.nn.functional as F
torch.tensor(x, dtype=torch.float32) · torch.from_numpy(a) · t.numpy() · t.item() · t.to(dispositivo) · t.shape
t.reshape · t.view · t.unsqueeze · t.squeeze · t.permute · torch.cat · torch.stack · t.argmax(dim=1)
x.requires_grad_() · perdida.backward() · x.grad · torch.no_grad() · torch.inference_mode() · t.detach()

class M(nn.Module): __init__ (capas) + forward(x) · nn.Sequential(...) · nn.Linear · nn.Conv2d · nn.LSTM · nn.Embedding
nn.CrossEntropyLoss() · nn.BCEWithLogitsLoss() · nn.MSELoss() · torch.optim.AdamW(m.parameters(), lr=1e-3)
torch.optim.lr_scheduler.ReduceLROnPlateau / CosineAnnealingLR / OneCycleLR
Dataset (__len__, __getitem__) · TensorDataset · DataLoader(ds, batch_size, shuffle, num_workers) · random_split

modelo.train(); opt.zero_grad(); perdida = fn(modelo(x), y); perdida.backward(); opt.step()
modelo.eval(); with torch.no_grad(): pred = modelo(x).argmax(1)

torch.save(m.state_dict(), "m.pt") · m.load_state_dict(torch.load("m.pt", weights_only=True))
torch.export.export(m, (ej,)) · torch.export.save / load · torch.compile(m) · torch.autocast(device_type, dtype)
torch.manual_seed(42) · torch.cuda.is_available() · torch.backends.mps.is_available()
```

---

## 20. Ejercicios propuestos

1. Implemente una regresión lineal con tensores y autograd (sin `nn.Module`) y después con `nn.Linear` y un optimizador.
2. Entrene una red totalmente conectada para el ejemplo de abandono de clientes (manual de scikit-learn) y compare su AUC con la regresión logística.
3. Sustituya la CNN del ejemplo por una red sin convoluciones (`nn.Flatten` + `nn.Linear`) y compare exactitud y número de parámetros.
4. Aplique aprendizaje por transferencia con `resnet18` sobre un conjunto de imágenes propio organizado en carpetas (`torchvision.datasets.ImageFolder`).
5. Añada al ejemplo completo precisión mixta y `torch.compile`, y mida el efecto en el tiempo por época en GPU.

---

## 21. Referencias

- Documentación oficial: https://docs.pytorch.org/docs/stable/
- Tutoriales: https://docs.pytorch.org/tutorials/
- Instalación: https://pytorch.org/get-started/locally/
- `torch.export`: https://docs.pytorch.org/docs/stable/export.html
- Guía de rendimiento: https://docs.pytorch.org/tutorials/recipes/recipes/tuning_guide.html
