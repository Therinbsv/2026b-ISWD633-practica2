# Conocimientos logrados  
## Comparación de conocimientos 
### Antes de la práctica
Mis conocimientos en Docker estaban enfocados en la ejecución de contenedores con sus configuraciones básicas de red y persistencia (volúmenes). Sabía cómo levantar servicios y conectarlos entre sí, pero no tenía control sobre el hardware que consumían, ni sabía cómo hacer que Docker reaccionara de forma automática si un servicio se caía. Tampoco tenía experiencia creando mis propias imágenes desde cero, limitándome a usar las imágenes oficiales de Docker Hub.

### Después de la práctica
He comprendido cómo optimizar y asegurar el entorno de ejecución mediante la limitación de recursos (CPU, memoria y swap) y la configuración de monitoreo de estado (Healthchecks). Aprendí a construir imágenes personalizadas a través de archivos Dockerfile, entendiendo la importancia de las capas, el uso de imágenes base actualizadas y el mecanismo de caché para acelerar futuras construcciones. Además, apliqué políticas de reinicio automático para garantizar la alta disponibilidad de los servicios y aprendí a gestionar la limpieza de recursos huérfanos.

## Lo aprendido

### 1. Limitar recursos (Generalidades)
* **Def:** Consiste en establecer restricciones específicas de hardware a un contenedor para prevenir el abuso de recursos, garantizar la estabilidad del host, aislar los servicios y cumplir con políticas de eficiencia.
* **Importancia:** Evita que un solo contenedor (por ejemplo, con un bucle infinito o mucha carga) consuma toda la memoria o CPU del servidor, degradando el rendimiento de los demás.

### 2. Memoria (RAM y Swap)
* **Def:** Permite definir la cantidad máxima de memoria que un contenedor puede usar.
* **Recomendaciones y requisitos previos:**
  * El valor mínimo permitido para la memoria RAM es de `6m`.
  * Para configurar Swap, **siempre** se debe usar junto con `--memory` para que Docker tenga un punto de referencia.
* **Pasos y comandos:**
  1. Limitar la RAM (ej. 300 megabytes):
     ```bash
     docker run -d --name <nombre> --memory=300m <imagen>:<tag>
     ```
  2. Limitar RAM y establecer un límite total combinado (RAM + Swap, ej. 1 gigabyte):
     ```bash
     docker run -d --name <nombre> --memory=300m --memory-swap=1g <imagen>:<tag>
     ```
     *(Fórmula: Swap máxima = memory-swap - memory. Ej: 1024m - 300m = 724m de swap).*

### 3. Procesador (CPU)
* **Def:** Restringe la cantidad de núcleos o el porcentaje de tiempo de CPU que un contenedor puede utilizar.
* **Recomendaciones y requisitos previos:**
  * Es útil conocer cuántos procesadores lógicos tiene la máquina. En Docker se consulta con `docker info` (buscando la línea `CPUs`).
* **Pasos y comandos:**
  1. Limitar la cantidad de núcleos (ej. 1.5 núcleos):
     ```bash
     docker run -d --name <nombre> --cpus="1.5" <imagen>:<tag>
     ```
  2. Asignar núcleos específicos (ej. usar solo los núcleos 0, 1 y 2):
     ```bash
     docker run -d --name <nombre> --cpuset-cpus="0-2" <imagen>:<tag>
     ```

### 4. Healthcheck (Monitoreo de estado)
* **Def:** Comandos que Docker ejecuta periódicamente dentro del contenedor para verificar si el servicio está funcionando. Si el comando devuelve código 0, está `(healthy)`; si falla (código distinto de 0), pasa a `(unhealthy)`.
* **Recomendaciones y requisitos previos:**
  * Usar `|| exit 1` al final del comando para asegurar que cualquier fallo sea detectado por Docker.
  * Configurar un `--health-start-period` si la aplicación tarda en arrancar.
* **Pasos y comandos:**
  1. Ejecutar un contenedor con un Healthcheck completo:
     ```bash
     docker run -d --name <nombre> --health-cmd="curl -f http://localhost/ || exit 1" --health-interval=5s --health-retries=3 --health-timeout=10s --health-start-period=5s <imagen>:<tag>
     ```
  2. Ver el estado (columna STATUS en `docker ps`):
     ```bash
     docker ps
     ```

### 5. Dockerfile (Creación de imágenes)
* **Def:** Archivo de texto plano con instrucciones secuenciales (capas) para construir una imagen personalizada.
* **Recomendaciones y requisitos previos:**
  * Usar la bandera `-y` en gestores de paquetes (`yum`, `apt`) para evitar que la construcción se pause pidiendo confirmación.
  * **Nota sobre imágenes base descontinuadas:** Durante la práctica, se intentó usar `centos:7` y `centos:8`, pero sus repositorios fueron descontinuados (EOL). Por ello, se optó por usar **`almalinux:8`** como imagen base, que es su reemplazo directo, compatible con `yum` y con soporte activo.
* **Pasos y comandos:**
  1. Crear el archivo (ej. usando `FROM almalinux:8`, `RUN`, `COPY`, `EXPOSE`, `CMD`).
  2. Construir la imagen (el punto indica el directorio actual):
     ```bash
     docker build -t <nombre_imagen>:<tag> .
     ```
  3. *Mecanismo de caché:* Al modificar un archivo y construir una nueva versión (ej. `:2.0`), los pasos previos se cargan desde la caché (`CACHED`), reduciendo el tiempo de espera a segundos.

### 6. Políticas de reinicio
* **Def:** Reglas que indican a Docker qué hacer si el contenedor se detiene (por fallo o manualmente).
* **Recomendaciones y requisitos previos:**
  * Analizar la naturaleza del servicio. Para servicios críticos siempre activos, usar `always`; para tareas que terminan, usar `on-failure`.
* **Pasos y comandos:**
  1. **`no`**: No reinicia (por defecto).
     ```bash
     docker run -d --name <nombre> <imagen>:<tag>
     ```
  2. **`always`**: Reinicia siempre (incluso si se reinicia Docker).
     ```bash
     docker run -d --name <nombre> --restart always <imagen>:<tag>
     ```
  3. **`on-failure`**: Reinicia solo si el código de salida es un error.
     ```bash
     docker run -d --name <nombre> --restart on-failure <imagen>:<tag>
     ```
  4. **`unless-stopped`**: Como `always`, pero si se detiene manualmente, se queda apagado.
     ```bash
     docker run -d --name <nombre> --restart unless-stopped <imagen>:<tag>
     ```
  *(Bonus de la práctica: Las imágenes huérfanas generadas por construcciones fallidas se limpian con `docker image prune`).*
