SegNet sobre ISPRS Vaihingen en Google Colab
Pipeline reproducible con auto-parcheo basado en DeepNetsForEO

DESCRIPCION
Este proyecto aplica SegNet (encoder–decoder con max-unpooling) para segmentación semántica por píxel sobre ISPRS Vaihingen. Partimos del repositorio público DeepNetsForEO y lo adaptamos con un flujo automatizado en Google Colab que:

* Monta Google Drive y detecta automáticamente el dataset.
* Parchea el notebook original (rutas dinámicas, log\_softmax(dim=1), carga de pesos).
* Crea un launcher ligero (sin entrenamiento/visualizaciones frágiles).
* Ejecuta inferencia con ventanas deslizantes y padding seguro.
* Calcula métricas (OA, F1/IoU por clase, mIoU, Kappa).
* Exporta PNG (paleta ISPRS) y GeoTIFF georreferenciados listos para SIG.

ESTRUCTURA DEL PROYECTO (rutas típicas en Colab)


├content/
├─ DeepNetsForEO/  
│  ├─ SegNet\_PyTorch\_v2.ipynb  
│  ├─ SegNet\_PyTorch\_v2\_colab.ipynb  
│  ├─ SegNet\_PyTorch\_v2\_launcher.ipynb
│  ├─ weights/
│  │  └─ segnet\_final\_reference.pth  
│  └─ out/  
├─ data/ISPRS/
│  └─ ISPRS\_dataset/Vaihingen/
│     ├─ top/  
│     └─ gts\_for\_participants/  
├─ config\_deepnets\_eo.json  
└─ manifest\_isprs.csv



REQUISITOS

* Google Colab (GPU opcional).
* Archivo ISPRS\_dataset.rar (Vaihingen) en Google Drive.
* Internet para clonar el repo y descargar pesos con gdown.

GUIA RAPIDA (Colab)

1. Reset limpio (opcional): desmontar Drive y limpiar /content.
2. Montar Drive y crear rutas base:
   PROJECT\_ROOT=/content/DeepNetsForEO
   DATA\_ROOT=/content/data/ISPRS
   WEIGHTS\_DIR=/content/DeepNetsForEO/weights
3. Localizar y extraer dataset: buscar ISPRS\_dataset.rar en MyDrive; instalar unrar y extraer a DATA\_ROOT.
4. Clonar DeepNetsForEO:
   git clone --depth=1 https://github.com/nshaud/DeepNetsForEO.git
5. Instalar dependencias:

   * pip install -r requirements.txt (puede fallar en repos antiguos)
   * pip install opencv-python scikit-image rasterio tifffile albumentations==1.4.7 matplotlib tqdm

6. Guardar configuración: detectar images/labels y escribir /content/config\_deepnets\_eo.json.
7. Descargar pesos:
   pip install gdown
   gdown --id <GDRIVE\_ID> -O /content/DeepNetsForEO/weights/segnet\_final\_reference.pth
8. AUTO PATCH + LAUNCH: detección de Vaihingen, inyección de rutas, corrección log\_softmax(dim=1), celda de carga de pesos, creación y ejecución del launcher.
9. Boot del modelo: instanciar SegNet, mover a GPU si hay, cargar pesos (strict=False).
10. Inferencia robusta: ventanas 256x256, stride 64, padding de parches, acumulación de logits; guardar PNG coloreados en /content/DeepNetsForEO/out.
11. Visualización rápida: panel RGB/GT/Pred/Overlay.
12. Métricas: OA, F1/IoU por clase, mIoU, Kappa desde la matriz de confusión.
13. Exportación: ZIP con predicciones y GeoTIFF monobanda (uint8) heredando georreferencia del raster fuente.

RESULTADOS

* Split de prueba: 6 tiles (p. ej., áreas 21, 23, 26, 3, 30, 5).
* Rendimiento (reportado por la celda de métricas):
  OA: 91.1%
  mIoU: 0.733
  Kappa: 0.881
  F1 por clase: buildings 0.933, roads 0.925, trees 0.915, cars 0.854, low vegetation 0.807, clutter 0.473
  IoU por clase: buildings 0.886, roads 0.861, trees 0.841, cars 0.745, low vegetation 0.755, clutter 0.391
* Visualizaciones: panel RGB/GT/Pred/Overlay; máscaras PNG en DeepNetsForEO/out; GeoTIFF en DeepNetsForEO/out\_geotiff.

DECISIONES CLAVE (OPTIMIZACION DEL FLUJO)

* Parcheo de compatibilidad: log\_softmax(dim=1) y rutas dinámicas.
* Carga de pesos con strict=False para tolerar diferencias menores de claves.
* Launcher sin entrenamiento/visualizaciones frágiles; test rápido extremo a extremo.
* Inferencia con ventaneo solapado + padding y acumulación de logits para suavizar bordes.
* Configuración centralizada (JSON) y manifest para trazabilidad.

PALETA ISPRS (RGB ↔ clase)
ID 0: roads/impervious   -> (255,255,255)
ID 1: buildings          -> (0,0,255)
ID 2: low vegetation     -> (0,255,255)
ID 3: trees              -> (0,255,0)
ID 4: cars               -> (255,255,0)
ID 5: clutter/background -> (255,0,0)
ID 6\*: undefined (opcional) -> (0,0,0)

REPRODUCIBILIDAD

* config\_deepnets\_eo.json guarda rutas y parámetros.
* manifest\_isprs.csv registra emparejamientos imagen/label.
* SegNet\_PyTorch\_v2\_launcher.ipynb se ejecuta programáticamente con nbclient.
* Split robusto con semilla fija (2/3–1/3; casos especiales 1–2 tiles).

SOLUCION DE PROBLEMAS

* Falla requirements.txt: instalar paquetes modernos (ver Paso 5).
* No se encuentra el RAR: mover ISPRS\_dataset.rar a MyDrive o añadir rutas a CANDIDATE\_DIRS.
* OOM en inferencia: aumentar STRIDE (128/256) o reducir BATCH\_SIZE.
* Predicciones erróneas/desalineadas: revisar DATA\_FOLDER/LABEL\_FOLDER y paleta RGB↔ID.
* GeoTIFF sin proyección: confirmar lectura de metadatos del raster fuente (rasterio.open(src)).

HOJA DE RUTA (MEJORAS)

* Fine-tuning con algunos tiles locales.
* Entrada IRRG/multiespectral para separar mejor low veg vs trees.
* TTA y data augmentation.
* Arquitecturas recientes (DeepLabV3+, U-Net++).
* Incertidumbre (MC Dropout) para mapas de confianza.
* Posprocesado (CRF/morfológicos) para refinar bordes y cars.

LICENCIA
Respeta la licencia del repositorio original DeepNetsForEO y los términos de uso de ISPRS Vaihingen. Este README y los scripts añadidos pueden distribuirse bajo MIT (ajústalo si prefieres otra).

AGRADECIMIENTOS
A los autores de DeepNetsForEO y a ISPRS por el benchmark Vaihingen. Gracias a la comunidad de Colab por mantener paquetes actualizados.

CITA SUGERIDA

* Badrinarayanan et al., SegNet: A Deep Convolutional Encoder–Decoder Architecture for Image Segmentation, IEEE TPAMI, 2017.
* ISPRS 2D Semantic Labeling (Vaihingen).
* Repositorio DeepNetsForEO.
