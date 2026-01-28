# Guía de Despliegue y Comandos

Esta guía detalla los pasos para iniciar, verificar y reiniciar la aplicación en un entorno local utilizando Docker.

## Prerrequisitos
*   Docker y Docker Compose instalados.
*   Windows (PowerShell o CMD) para ejecutar los scripts `.bat` (o Bash para Linux/Mac si se adaptan los scripts).

## 1. Iniciar la Aplicación (Despliegue)

Para levantar toda la infraestructura (Hadoop, Spark, Node.js), utilice el siguiente comando en la raíz del proyecto:

```powershell
docker-compose up -d --build
```
*   `up`: Crea e inicia los contenedores.
*   `-d`: Ejecuta en segundo plano (detached mode).
*   `--build`: Fuerza la reconstrucción de las imágenes (útil si se han hecho cambios en Dockerfiles o configuración).

## 2. Inicialización y Verificación Automática

Una vez que los contenedores están corriendo, es necesario inicializar el entorno (crear carpetas en HDFS) y procesar los datos iniciales. Hemos automatizado todo esto en un solo script:

```powershell
.\scripts\verify_env.bat
```

Este script realizará secuencialmente:
1.  Verificación de que los contenedores están activos.
2.  Creación de la estructura de directorios en HDFS.
3.  Carga de datos crudos a HDFS (Trabajo Spark: `load_to_hdfs.py`).
4.  Ejecución de trabajos de limpieza ( `clean_data.py`).
5.  Ejecución de análisis básicos y avanzados para API V2 (`analytics_advanced.py`).
5.  Verificación de la disponibilidad de la API.

**Nota**: La primera ejecución puede tardar varios minutos mientras Spark procesa los datos.

## 3. Verificar la API

Para confirmar que la API está sirviendo datos correctamente, ejecute:

```powershell
.\scripts\check_api.bat
```
Este script probará los endpoints predefinidos y mostrará los códigos de respuesta HTTP (esperado: 200 OK).

## 4. Detener la Aplicación

Para detener los servicios sin borrar los datos:

```powershell
docker-compose stop
```

Para detener y eliminar los contenedores (conservando volúmenes si están configurados como externos, aunque en este proyecto son mayormente efímeros fuera de las carpetas montadas):

```powershell
docker-compose down
```

## 5. Reinicio Total (Borrar Todo)

Si desea comenzar desde cero, eliminando todos los volúmenes de datos (base de datos de HDFS, logs, etc.) y reconstruir todo limpio, use:

```powershell
docker-compose down -v
docker-compose up -d --build
```
*   `-v`: Elimina los volúmenes asociados a los contenedores. Esto es crucial cuando hay errores de corrupción de HDFS o conflictos de configuración persistentes.

## Resumen de Comandos Frecuentes

| Acción | Comando |
|--------|---------|
| **Iniciar** | `docker-compose up -d` |
| **Ver Logs DataNode** | `docker logs -f hadoop-datanode` |
| **Entrar a contenedor** | `docker exec -it hadoop-namenode bash` |
| **Ejecutar Spark Job manual** | `docker exec spark-master /spark/bin/spark-submit ...` |

## 6. Cargar Nuevos Datos (Años Futuros)

Si desea agregar datos de nuevos años (ej. **2020**, 2026) o actualizar los existentes, el proceso está diseñado para ser **automático** en cuanto a código (no necesita tocar la API ni el Frontend), pero requiere **ejecutar manualmente** una secuencia de comandos para procesar la nueva información.

### Estrategia: "Borrón y Cuenta Nueva" (Raw)
Para evitar duplicados de datos y conflictos, la estrategia más segura es borrar la carpeta de "datos crudos" (raw) en HDFS y recargar todo el conjunto de datos (años viejos + años nuevos) desde su carpeta local. El sistema es lo suficientemente rápido para permitir esto.

### Paso a Paso Detallado:

#### Paso 1: Preparar Archivos Locales
Asegúrese de que en su carpeta local del proyecto (`d:\nyc-taxi-bigdata\data\raw\`) existan las carpetas correspondientes a cada año.
*   `data/raw/2024/*.parquet` (Datos existentes)
*   **`data/raw/2020/*.parquet`** (Nuevos datos a agregar)

#### Paso 2: Limpiar HDFS (Evitar Duplicados)
El script de carga agrega datos ("append"). Si no limpiamos antes, tendríamos datos duplicados. Ejecute este comando para borrar la carpeta raw en HDFS (sus archivos locales están seguros):

```powershell
docker exec hadoop-namenode hdfs dfs -rm -r -f /data/nyc/raw/taxi-trips
```

#### Paso 3: Cargar Todo a HDFS
Este script escaneará recursivamente todas las carpetas en su directorio `raw` local (2020, 2024, etc.) y las subirá a Hadoop.

```powershell
docker exec spark-master /spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark-apps/load_to_hdfs.py
```
*Al finalizar, verá un reporte en consola indicando los archivos cargados de todos los años.*

#### Paso 4: Limpieza y Estandarización
Procesa todos los datos crudos que acabamos de subir. Elimina viajes inválidos y estandariza el formato. Este paso sobreescribe los datos procesados anteriores, asegurando consistencia.

```powershell
docker exec spark-master /spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark-apps/clean_data.py
```

#### Paso 5: Generar Analíticas (Actualización de API)
**Este es el paso clave.** Spark recalcula todas las estadísticas agrupando dinámicamente por año. Detectará automáticamente el nuevo año (ej. 2020) y generará los archivos JSON que consume la API.

```powershell
docker exec spark-master /spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark-apps/analytics_advanced.py
```

---

### ¿Qué sucede con la API y el Dashboard? (Importante)

Una vez completado el **Paso 5**, la actualización es **inmediata y automática**:

1.  **NO necesita reiniciar la API**: La API no guarda datos en memoria ni en código. Cada vez que usted consulta un endpoint, la API lee el archivo JSON más reciente generado por Spark. Al haber ejecutado el Paso 5, ese archivo ya contiene la información del nuevo año.
2.  **NO necesita modificar el Frontend**: El Dashboard web está programado para ser dinámico. Lee la lista de años disponibles desde los datos de la API.
    *   Simplemente **acceda al endpoint** o **recargue la página web (F5)**.
    *   Verá que el año **2020** aparece automáticamente en los filtros desplegables ("Select Year").
    *   Toda la data del nuevo año estará disponible para visualizar.

### Resumen de Comandos para Copiar y Pegar

Para realizar todo el proceso de actualización de una sola vez, puede pegar este bloque en su terminal PowerShell:

```powershell
# 1. Borrar datos antiguos en HDFS
docker exec hadoop-namenode hdfs dfs -rm -r -f /data/nyc/raw/taxi-trips

# 2. Cargar nuevos datos (Local -> HDFS)
docker exec spark-master /spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark-apps/load_to_hdfs.py

# 3. Limpiar y Procesar
docker exec spark-master /spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark-apps/clean_data.py

# 4. Generar Analíticas (Actualiza API)
docker exec spark-master /spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark-apps/analytics_advanced.py
```
