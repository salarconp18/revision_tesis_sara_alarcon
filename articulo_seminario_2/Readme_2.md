# Segmentación semántica multiclase de coberturas urbanas

## Comparación entre un enfoque basado en superpíxeles SLIC y una arquitectura U-Net

Este repositorio contiene los insumos, notebooks, resultados y configuración computacional necesarios para reproducir los experimentos desarrollados en el marco del artículo:

**Segmentación semántica multiclase de coberturas urbanas: comparación entre un enfoque basado en superpíxeles SLIC y una arquitectura U-Net**

El propósito principal de este repositorio es servir como una guía reproducible para comparar dos enfoques de segmentación semántica multiclase aplicados a coberturas urbanas:

1. Un enfoque clásico basado en superpíxeles SLIC y clasificación supervisada.
2. Un enfoque de aprendizaje profundo basado en una arquitectura U-Net.

La comparación se plantea bajo un entorno reproducible, utilizando datos, notebooks, resultados y configuración computacional documentados.

---

## 1. Propósito del repositorio

Este repositorio tiene como finalidad:

- Documentar la estructura general del proyecto.
- Explicar el contenido de las carpetas principales.
- Indicar cómo descargar los datos pesados desde Zenodo.
- Especificar dónde deben ubicarse los datos para ejecutar los notebooks.
- Permitir el despliegue local de un ambiente JupyterLab mediante Docker.
- Facilitar la reproducción de los experimentos asociados a los métodos SLIC y U-Net.
- Conservar los resultados generados por cada enfoque metodológico.
- Servir como soporte técnico y reproducible del artículo científico.

Los datos pesados no se almacenan directamente en GitHub debido a su tamaño. Estos deben descargarse desde Zenodo y ubicarse en la ruta indicada en este documento.

---

## 2. Estructura general del proyecto

La estructura principal del proyecto es la siguiente:

```text
articulo_seminario_2/
│
├── articulo/
├── notebooks/
├── resultados_DL/
├── resultados_SLIC/
├── Dockerfile
├── docker-compose.yml
├── environment.yml
└── README.md
```

---

## 3. Descripción de carpetas y archivos principales

| Elemento | Descripción |
|---|---|
| `articulo/` | Carpeta destinada a el archivo relacionado con la escritura del artículo científico. |
| `notebooks/` | Carpeta que contiene los notebooks utilizados para el procesamiento, entrenamiento, inferencia, evaluación y comparación de los métodos SLIC y U-Net. |
| `resultados_DL/` | Carpeta destinada a almacenar resultados asociados al enfoque de Deep Learning, principalmente los experimentos con arquitectura U-Net. |
| `resultados_SLIC/` | Carpeta destinada a almacenar resultados asociados al enfoque clásico basado en superpíxeles SLIC y clasificación supervisada. |
| `Dockerfile` | Archivo que define la imagen Docker del ambiente de ejecución. |
| `docker-compose.yml` | Archivo que permite construir y levantar el servicio local de JupyterLab mediante Docker Compose. |
| `environment.yml` | Archivo que define el ambiente Conda con las librerías necesarias para ejecutar los notebooks. |
| `README.md` | Documento principal de orientación, reproducibilidad y ejecución del proyecto. |

---

## 4. Datos del proyecto

Los datos requeridos para reproducir los experimentos no se almacenan directamente en este repositorio debido a su tamaño.

Los datos estarán disponibles en Zenodo:

```text
https://doi.org/10.5281/zenodo.20043709
```

## 5. Contenido esperado del dataset en Zenodo

El depósito de Zenodo debe contener los insumos necesarios para reproducir los experimentos. De manera general, el dataset puede incluir:

| Tipo de dato | Descripción |
|---|---|
| Ortoimagen aérea | Imagen base utilizada para la segmentación semántica de coberturas urbanas. |
| Derivados LiDAR | Productos derivados como el nDSM. |
| Máscaras de referencia | Datos de verdad terreno o máscaras temáticas utilizadas para entrenamiento, validación y evaluación. |
| Archivos auxiliares | Insumos complementarios requeridos por los notebooks para ejecutar los experimentos. |

---

## 6. Descarga y ubicación de los datos

Después de descargar los datos desde Zenodo, el usuario debe descomprimirlos dentro de la siguiente ruta local:

```text
articulo_seminario_2/work/data/
```

Por ejemplo, la estructura esperada puede ser:

```text
articulo_seminario_2/
│
├── articulo/
├── notebooks/
├── resultados_DL/
├── resultados_SLIC/
├── Dockerfile
├── docker-compose.yml
├── environment.yml
├── README.md
│
└── work/
    └── data/
        └── tesis_sara_datos_v1/
            ├── ortoimagen/
            ├── lidar/
            ├── mascaras/
            ├── particiones/
            └── auxiliares/
```

Dentro del contenedor Docker, esta ruta local estará disponible como:

```text
/home/jovyan/work/data/
```

Por tanto, los notebooks deben leer los datos desde rutas relativas a:

```text
data/tesis_sara_datos_v1/
```

o desde la ruta absoluta dentro del contenedor:

```text
/home/jovyan/work/data/tesis_sara_datos_v1/
```

---

## 7. Relación entre la carpeta local y JupyterLab

El servicio Docker monta la carpeta local:

```text
./work
```

dentro del contenedor como:

```text
/home/jovyan/work
```

Esto significa que cualquier archivo guardado dentro de JupyterLab en:

```text
/home/jovyan/work
```

quedará almacenado en la carpeta local:

```text
articulo_seminario_2/work/
```

Esta lógica permite conservar datos, notebooks editados, resultados intermedios y salidas finales aunque el contenedor se apague.

---

## 8. Ambiente computacional reproducible

El proyecto incluye tres archivos principales para construir el ambiente de ejecución:

| Archivo | Función |
|---|---|
| `Dockerfile` | Construye la imagen Docker del servicio JupyterLab. |
| `docker-compose.yml` | Define el servicio, el puerto de acceso, el volumen local y la configuración general del contenedor. |
| `environment.yml` | Define el ambiente Conda con Python y las librerías requeridas. |

El ambiente se crea con el nombre:

```text
sara_semseg
```

y se registra en JupyterLab como el kernel:

```text
Python (sara_semseg)
```

Este kernel debe seleccionarse al abrir y ejecutar los notebooks.

---

## 9. Requisitos previos

Para ejecutar el proyecto localmente se requiere:

1. Docker instalado.
2. Docker Compose disponible.
3. Un navegador web.
4. Espacio suficiente en disco para almacenar los datos descargados desde Zenodo.
5. GPU NVIDIA configurada para Docker, si se desea ejecutar procesamiento acelerado.

---

## 10. Construcción del ambiente Docker

Desde una terminal ubicada en la carpeta:

```text
articulo_seminario_2/
```

ejecutar:

```bash
docker compose build
```

Este comando construye la imagen Docker del proyecto e instala el ambiente definido en `environment.yml`.

También se puede construir y levantar el servicio en un solo paso:

```bash
docker compose up --build
```

---

## 11. Levantar el servicio JupyterLab

Para iniciar el servicio, ejecutar:

```bash
docker compose up
```

Cuando el servicio inicie correctamente, la terminal mostrará una URL similar a:

```text
http://127.0.0.1:8888/lab?token=xxxxxxxx
```

Copiar esa URL y abrirla en el navegador.

También se puede intentar acceder desde:

```text
http://localhost:8888
```

Si JupyterLab solicita un token, se debe usar el token mostrado en la terminal.

---

## 12. Acceso a los notebooks

Una vez abierto JupyterLab, navegar a:

```text
/home/jovyan/work
```

Los notebooks del proyecto se encuentran en la carpeta:

```text
notebooks/
```

Si se desea trabajar directamente dentro del ambiente montado por Docker, se recomienda copiar o ubicar los notebooks en:

```text
articulo_seminario_2/work/notebooks/
```

Dentro de JupyterLab, esta ruta corresponde a:

```text
/home/jovyan/work/notebooks/
```

---

## 13. Selección del kernel

Al abrir cada notebook, verificar que el kernel seleccionado sea:

```text
Python (sara_semseg)
```

Si aparece otro kernel, cambiarlo manualmente desde el menú:

```text
Kernel → Change Kernel → Python (sara_semseg)
```

---

## 14. Flujo general de reproducibilidad

El flujo recomendado para reproducir el proyecto es:

```text
1. Clonar o descargar este repositorio.
2. Descargar los datos desde Zenodo.
3. Descomprimir los datos en articulo_seminario_2/work/data/.
4. Construir el ambiente Docker.
5. Levantar el servicio JupyterLab.
6. Abrir los notebooks.
7. Seleccionar el kernel Python (sara_semseg).
8. Ejecutar los notebooks asociados al método SLIC.
9. Ejecutar los notebooks asociados al método U-Net.
10. Revisar las salidas en resultados_SLIC/ y resultados_DL/.
11. Comparar los resultados temáticos, geométricos y computacionales.
```

---

## 15. Métodos implementados

### 15.1 Enfoque basado en superpíxeles SLIC

El enfoque SLIC corresponde a un método clásico de segmentación basado en objetos o regiones. En este flujo, la imagen se divide en superpíxeles y posteriormente se calculan atributos por objeto para realizar una clasificación supervisada.

De manera general, el flujo metodológico es:

```text
Datos de entrada
↓
Generación de superpíxeles SLIC
↓
Extracción de atributos por superpíxel
↓
Asignación de etiqueta por mayoría
↓
Entrenamiento del clasificador
↓
Inferencia
↓
Evaluación temática y geométrica
```

Los resultados asociados a este método se almacenan en:

```text
resultados_SLIC/
```

---

### 15.2 Enfoque basado en U-Net

El enfoque de Deep Learning utiliza una arquitectura tipo U-Net para realizar segmentación semántica multiclase a nivel de píxel.

De manera general, el flujo metodológico es:

```text
Datos de entrada
↓
Preparación de teselas o parches
↓
Definición del conjunto de entrenamiento, validación y prueba
↓
Entrenamiento de la arquitectura U-Net
↓
Inferencia sobre el conjunto de prueba
↓
Evaluación temática y geométrica
↓
Análisis de desempeño computacional
```

Los resultados asociados a este método se almacenan en:

```text
resultados_DL/
```

---

## 16. Resultados

Los resultados del proyecto se organizan en dos carpetas principales:

| Carpeta | Contenido esperado |
|---|---|
| `resultados_SLIC/` | Métricas, mapas, matrices de confusión, salidas intermedias y resultados finales del enfoque SLIC. |
| `resultados_DL/` | Métricas, mapas, curvas de entrenamiento, matrices de confusión, predicciones y resultados finales del enfoque U-Net. |

Estas carpetas permiten separar claramente los productos obtenidos con cada método.

---

## 17. Criterios de comparación

La comparación entre los métodos puede considerar los siguientes criterios:

| Criterio | Descripción |
|---|---|
| Exactitud temática | Evalúa la correspondencia entre las clases predichas y las clases de referencia. |
| Exactitud geométrica | Evalúa la calidad espacial de los límites, bordes o formas segmentadas. |
| Desempeño computacional | Evalúa tiempos de ejecución, consumo energético y emisiones estimadas, si se usa CodeCarbon. |
| Reproducibilidad | Evalúa si los resultados pueden ser generados nuevamente bajo el mismo ambiente computacional y con los mismos datos. |

---

## 18. Clases temáticas consideradas

La segmentación semántica multiclase considera las siguientes clases temáticas:

| Código | Clase |
|---|---|
| `0` | Fondo |
| `1` | Construcción |
| `2` | Pavimento |
| `3` | Vegetación |
| `4` | Agua |
| `255` | NoData / clase ignorada |

La clase `255` se utiliza como valor ignorado en los procesos de evaluación, cuando aplica.

---

## 19. Persistencia de archivos y resultados

Los archivos generados dentro de JupyterLab deben guardarse dentro de:

```text
/home/jovyan/work
```

para que queden almacenados localmente en:

```text
articulo_seminario_2/work/
```

Ejemplos:

| Ruta dentro del contenedor | Ruta local |
|---|---|
| `/home/jovyan/work/data` | `articulo_seminario_2/work/data` |
| `/home/jovyan/work/notebooks` | `articulo_seminario_2/work/notebooks` |
| `/home/jovyan/work/outputs` | `articulo_seminario_2/work/outputs` |

---

## 20. Detener el servicio

Para detener el servicio, presionar en la terminal:

```bash
Ctrl + C
```

Luego ejecutar:

```bash
docker compose down
```

Este comando apaga el contenedor, pero no elimina los archivos guardados en:

```text
articulo_seminario_2/work/
```

---

## 21. Problemas frecuentes

| Problema | Posible causa | Solución |
|---|---|---|
| No abre `localhost:8888` | El servicio no está activo | Ejecutar `docker compose up`. |
| El puerto `8888` está ocupado | Otro servicio usa el mismo puerto | Cambiar el puerto externo en `docker-compose.yml`, por ejemplo `8889:8888`. |
| JupyterLab pide token | Es el comportamiento normal de seguridad | Copiar la URL completa mostrada en la terminal. |
| No aparecen los datos | Los datos no fueron descargados o están en otra ruta | Verificar que estén en `articulo_seminario_2/work/data/`. |
| No aparecen los notebooks | Los notebooks no están en la ruta esperada | Verificar la carpeta `notebooks/` o `work/notebooks/`. |
| No aparece el kernel `Python (sara_semseg)` | El ambiente no se creó correctamente | Reconstruir con `docker compose build --no-cache`. |
| Error con GPU NVIDIA | Docker no tiene acceso a la GPU | Revisar drivers, NVIDIA Container Toolkit y configuración de Docker. |
| Los resultados desaparecen | Se guardaron fuera de `/home/jovyan/work` | Guardar resultados dentro de `/home/jovyan/work`. |

---

## 22. Citación de los datos

Cuando el depósito de Zenodo esté publicado, citar los datos usando el DOI correspondiente:

```text
PENDIENTE: agregar cita del dataset en Zenodo.
```

Formato sugerido:

```text
Autora. (Año). Dataset para la segmentación semántica multiclase de coberturas urbanas [Data set]. Zenodo. https://doi.org/10.5281/zenodo.xxxxxxx
```

---

## 23. Citación del repositorio

Si el repositorio también se archiva mediante Zenodo a partir de una release de GitHub, citar la versión del repositorio usando el DOI generado para el código, notebooks y documentación:

```text
PENDIENTE: 10.5281/zenodo.20043709
```

Formato sugerido:

```text
Alarcon, Sara Geraldine (2026). Segmentación semántica multiclase de coberturas urbanas: comparación entre SLIC y U-Net [Software]. Zenodo. https://doi.org/10.5281/zenodo.xxxxxxx
```

---

## 24. Resumen para reproducir el proyecto

Para reproducir el proyecto en una máquina local:

```text
1. Descargar o clonar el repositorio.
2. Descargar los datos desde Zenodo.
3. Descomprimir los datos en articulo_seminario_2/work/data/.
4. Abrir una terminal en articulo_seminario_2/.
5. Ejecutar docker compose up --build.
6. Abrir la URL de JupyterLab mostrada en la terminal.
7. Seleccionar el kernel Python (sara_semseg).
8. Ejecutar los notebooks correspondientes.
9. Revisar los resultados generados.
10. Comparar los resultados de SLIC y U-Net.
```

---

## 25. Estado del proyecto

Este repositorio está diseñado como una guía reproducible para el desarrollo, documentación y comparación experimental de métodos de segmentación semántica multiclase de coberturas urbanas.

El contenido puede actualizarse conforme se consoliden:

- Los notebooks finales.
- El depósito de datos en Zenodo.
- Los resultados definitivos.
- Las tablas y figuras del artículo.
- La versión final del manuscrito.
- El DOI del dataset.
- El DOI del repositorio archivado.

---

## 26. Contacto

Para dudas relacionadas con la reproducción del proyecto, revisar primero:

1. La ubicación de los datos descargados desde Zenodo.
2. La correcta construcción del ambiente Docker.
3. La selección del kernel `Python (sara_semseg)`.
4. Las rutas utilizadas dentro de los notebooks.
5. La correspondencia entre las salidas generadas y las carpetas `resultados_SLIC/` y `resultados_DL/`.
