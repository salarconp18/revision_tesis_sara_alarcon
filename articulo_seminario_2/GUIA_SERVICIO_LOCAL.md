# Guía para desplegar localmente el servicio

## 1. Propósito de esta guía
Este proyecto despliega un servicio local de JupyterLab mediante Docker para ejecutar notebooks en un ambiente reproducible. El servicio se levanta con Docker Compose, expone JupyterLab en el puerto `8888` y monta la carpeta local `work/` dentro del contenedor en `/home/jovyan/work`. El ambiente de ejecución se define mediante Conda en el archivo `environment.yml` y se registra en Jupyter como el kernel `Python (sara_semseg)`. De esta manera, cualquier usuario con Docker instalado puede levantar el servicio, abrir los notebooks desde el navegador y ejecutarlos sin instalar manualmente las dependencias en su sistema operativo. Principalmente esta guia pretende atender lo siguiente:

1. Construir el ambiente de trabajo.
2. Levantar el servicio JupyterLab localmente.
3. Acceder a JupyterLab desde el navegador.
4. Abrir los notebooks disponibles.
5. Ejecutarlos dentro de un ambiente reproducible.
6. Conservar archivos, datos y resultados en la carpeta local del proyecto.
---

## 2. Lógica general del servicio
El proyecto está pensado para ejecutarse localmente mediante un contenedor Docker.

La lógica es la siguiente:

```text
Usuario
  ↓
Navegador web
  ↓
http://localhost:8888
  ↓
Servicio JupyterLab en Docker
  ↓
Notebooks ubicados en /home/jovyan/work
  ↓
Carpeta local ./work del proyecto
```

Esto significa que la persona que quiera replicar el trabajo no necesita instalar manualmente todas las librerías en su computador.  
Solo debe tener Docker instalado, construir el contenedor y levantar el servicio.
---

## 3. Archivos principales del despliegue
El despliegue se apoya en tres archivos principales:

| Archivo | Función |
|---|---|
| `Dockerfile` | Define cómo se construye la imagen Docker del servicio JupyterLab. |
| `docker-compose.yml` | Define cómo se levanta el servicio, qué puerto usa y qué carpeta local se monta dentro del contenedor. |
| `environment.yml` | Define el ambiente Conda y las librerías necesarias para ejecutar los notebooks. |
---

## 4. Estructura mínima recomendada
La carpeta del proyecto debe tener una estructura similar a esta:

```text
proyecto/
│
├── Dockerfile
├── docker-compose.yml
├── environment.yml
│
└── work/
    ├── notebooks/

La carpeta más importante para el usuario es:

```text
work/
```

Esta carpeta se monta dentro del contenedor en:

```text
/home/jovyan/work
```
---

## 5. Descripción del servicio definido en Docker Compose
El archivo `docker-compose.yml` define el servicio llamado:

```text
jupyter_semseg
```

Este servicio se encarga de levantar JupyterLab dentro de un contenedor Docker.

La configuración realiza principalmente lo siguiente:

- Construye la imagen usando el `Dockerfile`.
- Asigna el nombre `jupyter_semseg` al contenedor.
- Expone el puerto local `8888`.
- Monta la carpeta local `./work` dentro del contenedor.
- Permite el uso de una GPU NVIDIA, si está disponible y configurada.
- Mantiene el servicio activo salvo que sea detenido manualmente.

Esto permite acceder al servicio desde el navegador usando:

```text
http://localhost:8888
```
La sección clave para conservar archivos es:

```yaml
volumes:
  - ./work:/home/jovyan/work
```

Esto permite que los archivos guardados desde JupyterLab dentro de `/home/jovyan/work` queden almacenados en la carpeta local `./work`. Esta carpeta se crea automáticamente en caso de que no exista en el explorador de archivos.
---

## 6. Descripción del Dockerfile
El `Dockerfile` construye la imagen del servicio a partir de una imagen base de Jupyter:

```dockerfile
quay.io/jupyter/scipy-notebook:2025-12-31
```

Luego crea un ambiente Conda llamado:

```text
sara_semseg
```

Este ambiente se construye a partir del archivo:

```text
environment.yml
```

Después, el ambiente se registra como kernel de Jupyter con el nombre:

```text
Python (sara_semseg)
```

Ese será el kernel que debe seleccionarse al abrir los notebooks.

Además, el `Dockerfile` instala PyTorch con soporte CUDA mediante `pip`, lo cual permite aprovechar GPU NVIDIA si el equipo y Docker están correctamente configurados.
---

## 7. Descripción del ambiente Conda
El archivo `environment.yml` define el ambiente llamado:

```text
sara_semseg
```

Este ambiente incluye Python 3.11 y librerías para:

| Grupo | Ejemplos de librerías |
|---|---|
| Análisis científico | `numpy`, `scipy`, `pandas`, `matplotlib` |
| Machine Learning | `scikit-learn`, `scikit-image`, `joblib` |
| Procesamiento geoespacial | `rasterio`, `gdal`, `pyproj`, `shapely`, `fiona`, `tifffile` |
| Jupyter | `ipykernel`, `jupyterlab`, `notebook` |
| Visión por computador y deep learning | `opencv`, `albumentations`, `segmentation-models-pytorch`, `pillow` |

---

## 8. Requisitos para ejecutar el servicio
Antes de levantar el servicio, la persona debe tener instalado:

1. Docker.
2. Docker Compose.
3. Un navegador web.

Si se desea usar GPU, adicionalmente se requiere:

1. Una GPU NVIDIA compatible.
2. Drivers NVIDIA instalados.
3. NVIDIA Container Toolkit configurado.
4. Docker con acceso a la GPU.
---

## 9. Preparación antes de ejecutar
Antes de construir el contenedor, verificar que los archivos estén ubicados en la raiz de la carpeta:

```text
Dockerfile
docker-compose.yml
environment.yml
```

En Windows, es importante revisar si el explorador de archivos está ocultando las extensiones.
---

## 10. Construir la imagen Docker
Desde una terminal ubicada en la carpeta raíz del proyecto, ejecutar:

```bash
docker compose build
```

Este comando construye la imagen del servicio JupyterLab e instala el ambiente Conda definido en `environment.yml`.

---

## 11. Levantar el servicio JupyterLab

Después de construir la imagen, ejecutar:

```bash
docker compose up
```

También se puede construir y levantar el servicio en un solo comando:

```bash
docker compose up --build
```

Cuando el servicio inicie correctamente, la terminal mostrará una URL similar a:

```text
http://127.0.0.1:8888/lab?token=xxxxxxxx
```

Esa URL debe copiarse y abrirse en el navegador.

También se puede intentar ingresar desde:

```text
http://localhost:8888
```

En caso de que Jupyter solicite un token, se debe usar el token mostrado en la terminal.

---

## 12. Abrir los notebooks
Una vez dentro de JupyterLab, se debe navegar a la carpeta:

```text
/home/jovyan/work
```

Esa ruta corresponde a la carpeta local:

```text
./work
```

Por ejemplo, si los notebooks están en:

```text
work/notebooks/
```

en JupyterLab aparecerán en:

```text
/home/jovyan/work/notebooks/
```

Desde allí, la persona puede abrir el notebook que desee ejecutar.
---

## 13. Seleccionar el kernel correcto
Al abrir un notebook, verificar que el kernel seleccionado sea:

```text
Python (sara_semseg)
```

Este kernel corresponde al ambiente Conda creado dentro del contenedor.

Si el notebook se abre con otro kernel, se debe cambiar manualmente desde el menú de JupyterLab:

```text
Kernel → Change Kernel → Python (sara_semseg)
```
---

## 14. Ejecución de los notebooks
Los notebooks pueden ejecutarse directamente desde JupyterLab.

Esta guía no define un orden obligatorio de ejecución, porque el propósito principal es desplegar el servicio y permitir el acceso al ambiente.  
El usuario responsable del proyecto puede indicar, dentro de cada notebook o en una guía metodológica separada, qué notebook debe abrirse según el análisis que se quiera replicar.
---

## 15. Persistencia de archivos
Todo lo que se guarde dentro de:

```text
/home/jovyan/work
```

quedará almacenado en la carpeta local:

```text
./work
```

Ejemplos:

| Ruta dentro del contenedor | Ruta en la máquina local |
|---|---|
| `/home/jovyan/work/notebooks` | `./work/notebooks` |
| `/home/jovyan/work/data` | `./work/data` |
| `/home/jovyan/work/outputs` | `./work/outputs` |

Por esta razón, los resultados de los notebooks deben guardarse dentro de `/home/jovyan/work` para que no se pierdan al apagar el contenedor.
---

## 16. Detener el servicio
Para detener el servicio, presionar en la terminal:

```bash
Ctrl + C
```

Luego ejecutar:

```bash
docker compose down
```

Este comando apaga el contenedor, pero no elimina los archivos guardados en `./work`.
---

## 18. Problemas frecuentes
| Problema | Posible causa | Solución |
|---|---|---|
| No abre `localhost:8888` | El servicio no está activo | Ejecutar `docker compose up`. |
| El puerto `8888` está ocupado | Otro Jupyter usa el mismo puerto | Cambiar el puerto externo, por ejemplo `8889:8888`. |
| Jupyter pide token | Es el comportamiento normal de seguridad | Copiar la URL completa que aparece en la terminal. |
| No aparecen los notebooks | Los archivos no están dentro de `work/` | Ubicar los notebooks dentro de `./work`. |
| Los resultados desaparecen | Se guardaron fuera de `/home/jovyan/work` | Guardar resultados en `/home/jovyan/work/outputs`. |
| No aparece el kernel `Python (sara_semseg)` | El ambiente no se creó correctamente | Reconstruir la imagen con `docker compose build --no-cache`. |
| Error con GPU NVIDIA | Docker no tiene acceso a la GPU | Revisar drivers, NVIDIA Container Toolkit y configuración de Docker. |
| Docker no encuentra el Dockerfile | El archivo tiene extensión `.txt` | Renombrar `Dockerfile.txt` a `Dockerfile`. |
---

## 20. Resumen para replicar el servicio
Para replicar el servicio en otra máquina:

1. Instalar Docker.
2. Copiar la carpeta del proyecto.
3. Verificar que existan `Dockerfile`, `docker-compose.yml`, `environment.yml` y `work/`.
4. Abrir una terminal en la carpeta raíz del proyecto.
5. Ejecutar:

```bash
docker compose up --build
```

6. Copiar la URL de JupyterLab mostrada en la terminal.
7. Abrir la URL en el navegador.
8. Ingresar a `/home/jovyan/work`.
9. Abrir los notebooks disponibles.
10. Ejecutarlos usando el kernel `Python (sara_semseg)`.
---
