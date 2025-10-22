# SegNet (DeepNetsForEO) — Entorno reproducible con Docker + GPU

Este documento explica cómo **construir y ejecutar** un entorno reproducible para el notebook
`SegNet_PyTorch_v2.ipynb` usando **Docker Desktop (WSL2)** con **GPU NVIDIA** en Windows 11.

> **Contexto probado**
>
> - Windows 11 + Docker Desktop (WSL2) con *Engine running*
> - Driver NVIDIA host: CUDA 12.x (`nvidia-smi` muestra *CUDA Version: 12.8*)
> - Imagen propia: `segnet-gpu:0.3.2`
> - PyTorch: wheels **cu121** (compatibles con drivers 12.x)
> - Paquetes “horneados” para evitar conflictos (NumPy **1.26.4**, OpenCV **4.8.1.78**, etc.)

---

## 0) Comprobaciones rápidas

En PowerShell:

```powershell
# El engine responde
docker info

# (Opcional) Probar GPU dentro de un contenedor
docker run --rm --gpus all nvidia/cuda:12.8.0-base-ubuntu22.04 nvidia-smi
```

Si ves la tabla de `nvidia-smi`, Docker puede usar tu GPU.

---

## 1) Estructura recomendada

```
C:\...\segnet-docker\
  Dockerfile                # v0.3.2 (ver abajo)
  segnet-requirements.txt   # pin de versiones moderno (ver abajo)
C:\...\DeepNetsForEO\       # repo clonado
```

---

## 2) Archivos

**Dockerfile (v0.3.2)**

```dockerfile
FROM nvidia/cuda:12.8.0-cudnn-runtime-ubuntu22.04
ENV DEBIAN_FRONTEND=noninteractive PIP_DISABLE_PIP_VERSION_CHECK=1 PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y --no-install-recommends python3 python3-pip git ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /workspace

# PyTorch + CUDA (cu121)
RUN python3 -m pip install --upgrade pip  && pip install torch==2.2.2+cu121 torchvision==0.17.2+cu121 --index-url https://download.pytorch.org/whl/cu121

# Dependencias modernas y compatibles (NumPy 1.x + OpenCV 4.8)
COPY segnet-requirements.txt /tmp/segnet-requirements.txt
RUN pip install --no-cache-dir -r /tmp/segnet-requirements.txt

EXPOSE 8888
CMD ["jupyter","lab","--ip=0.0.0.0","--no-browser","--NotebookApp.token=","--NotebookApp.password=","--allow-root"]
```

**segnet-requirements.txt**

```txt
# Núcleo científico (Python 3.10 + Torch cu121)
numpy==1.26.4
scipy>=1.11,<2
pillow>=10
scikit-image>=0.21
scikit-learn>=1.4
matplotlib>=3.8,<3.9
tqdm>=4.66
# OpenCV compatible con NumPy 1.x
opencv-python-headless==4.8.1.78

# Jupyter stack (evita conflictos con pines antiguos)
ipython>=8
ipykernel>=6
jupyter>=1
jupyterlab>=4
jsonschema>=4
bleach>=6
html5lib>=1.1
jinja2>=3.1.2
markupsafe>=2.1.1
```

> **Nota:** No usamos el `requirements.txt` original del repo (tiene pines muy antiguos como
> `matplotlib==2.1.1`, `Jinja2==2.10`, `MarkupSafe==1.0`) que fallan en Python moderno.

---

## 3) Construir la imagen

```powershell
cd C:\...\segnet-docker
docker build -t segnet-gpu:0.3.2 .
```

---

## 4) Clonar el código

```powershell
cd ..
git clone https://github.com/nshaud/DeepNetsForEO.git
cd DeepNetsForEO
```

---

## 5) Ejecutar el contenedor con GPU + montar el repo

```powershell
docker run --gpus all -p 8888:8888 -v "${PWD}:/workspace/DeepNetsForEO" -it segnet-gpu:0.3.2
```

Abrir **http://localhost:8888** → entrar a
`/workspace/DeepNetsForEO/SegNet_PyTorch_v2.ipynb`.

**Kernel** a elegir en Jupyter: **Python 3 (ipykernel)**.

---

## 6) Verificar GPU dentro del notebook

```python
import numpy as np, cv2, torch
print("NumPy:", np.__version__)          # esperado 1.26.4
print("OpenCV:", cv2.__version__)        # esperado 4.8.1.x
print(torch.cuda.is_available(), torch.cuda.get_device_name(0), torch.version.cuda)  # True, GPU, cu121
```

---

## 7) Entrenamiento e inferencia (notas útiles)

- Si el modelo usa `F.log_softmax(...)`, **añade `dim=1`** en el `forward` para evitar warnings:
  ```python
  x = F.log_softmax(x, dim=1)
  ```
- Con `log_softmax` en el `forward`, la loss correcta es **`nn.NLLLoss()`**  
  (si en cambio el modelo devuelve *logits*, usa `nn.CrossEntropyLoss()`).
- En PyTorch moderno, reemplaza `loss.data[0]` por **`loss.detach().item()`**.
- Mueve datos y modelo a GPU:
  ```python
  device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
  net = net.to(device)
  xb, yb = xb.to(device, non_blocking=True).float(), yb.to(device, non_blocking=True).long()
  ```
- Acelera el `DataLoader`:
  ```python
  DataLoader(..., num_workers=4, pin_memory=True, persistent_workers=True)
  ```

---

## 8) No instales el `requirements.txt` del repo

No ejecutes `pip install -r DeepNetsForEO/requirements.txt`, ya que forzará versiones incompatibles.
El entorno ya viene listo con `segnet-requirements.txt`.

---

## 9) Actualizar la imagen sin parar la actual (opcional)

Puedes construir con un **nuevo tag** y levantar en otro puerto:

```powershell
docker build -t segnet-gpu:0.3.3 .
docker run --gpus all -p 8889:8888 -v "${PWD}:/workspace/DeepNetsForEO" -it segnet-gpu:0.3.3
```

Luego, cuando confirmes, detén y borra el contenedor viejo:
```powershell
docker ps
docker stop <ID_viejo> && docker rm <ID_viejo>
```

---

## 10) Limpieza

```powershell
# Ver uso de disco
docker system df

# Borrar contenedores parados
docker container prune -f

# Borrar capas/imágenes colgantes
docker image prune -f

# Quitar caché de build
docker builder prune -f
```

---

## 11) Troubleshooting

- **Jupyter pide token**: nuestra imagen lo desactiva (ver `CMD` del Dockerfile). Si lo pide, estás usando otra imagen/contendor.
- **GPU no aparece en contenedor**:
  - Docker Desktop → Settings → *General*: “Use WSL 2 based engine”
  - Docker Desktop → Settings → *Resources → WSL integration*: habilita tu distro.
  - `docker run --rm --gpus all nvidia/cuda:12.8.0-base-ubuntu22.04 nvidia-smi`
- **Conflictos con NumPy**: mantenemos `numpy==1.26.4` y `opencv-python-headless==4.8.1.78`.
  Si instalas otros paquetes, respeta ese pin o ajusta el `requirements` y **reconstruye**.

---

## 12) Persistencia

- **Código/datos**: persisten en el **host** (por el volumen `-v`).
- **Paquetes**: viven en la **imagen**. Si instalas a mano dentro del contenedor, solo afectan a ese contenedor. Para hacerlo reproducible, añade el paquete a `segnet-requirements.txt` y **reconstruye**.
