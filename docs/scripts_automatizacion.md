# Documentación de Scripts de Automatización

Este proyecto incluye scripts `.bat` (Batch) diseñados para automatizar tareas repetitivas de verificación, configuración y pruebas. A continuación se explica en detalle qué hace cada uno.

## 1. `scripts\verify_env.bat`

Este es el script maestro de orquestación. Su objetivo es asegurar que el entorno esté 100% operativo y con datos cargados.

### Flujo de Ejecución:

1.  **Detección de Contenedores (`docker ps`)**:
    *   Comprueba si los contenedores esenciales (`hadoop-namenode`, `hadoop-datanode`, `spark-master`, `nyc-taxi-api`) están en ejecución.
    *   Si alguno falta, detiene el script y alerta al usuario.

2.  **Preparación de HDFS**:
    *   Ejecuta comandos dentro del contenedor `hadoop-namenode`.
    *   Crea la estructura de directorios necesaria: `/data/nyc/raw`, `/data/nyc/processed`, `/data/nyc/analytics`.
    *   Ajusta permisos (`chmod 777`) para evitar errores de escritura por parte de Spark.

3.  **Carga de Datos (Job Spark)**:
    *   **Verificación Inteligente**: El script primero revisa si ya existen datos en HDFS.
    *   **Si está vacío**: Envía el trabajo `load_to_hdfs.py` para cargar los datos iniciales.
    *   **Si ya tiene datos**: **SALTA** este paso para ahorrar tiempo.
    *   *Nota*: Si deseas cargar nuevos archivos, usa `load_new_data.bat`.

4.  **Procesamiento de Datos (Jobs Spark)**:
    *   **Limpieza**: Ejecuta `clean_data.py` que filtra datos corruptos (ej. ubicaciones nulas) y guarda el resultado limpio.
    *   **Analítica**: Ejecuta `analytics_basic.py` que lee los datos limpios y genera archivos JSON con las métricas finales.

5.  **Verificación de API**:
    *   Realiza una petición de prueba básica usando `curl` para confirmar que el puerto 3000 responde.

---

## 2. `scripts\check_api.bat`

Este script es una herramienta de prueba rápida para el usuario final. Verifica específicamente la disponibilidad y corrección de los endpoints de la API.

### Flujo de Ejecución:

1.  **Prueba Raíz (`/`)**:
    *   Envía una petición GET a `http://localhost:3000/`.
    *   **Comportamiento Esperado**: Error 404. Esto es correcto y el script lo informa al usuario ("Normal").

2.  **Prueba Endpoint `trips-per-hour`**:
    *   Envía GET a `http://localhost:3000/api/trips-per-hour`.
    *   Muestra el **Código de Estado HTTP** (debe ser 200).
    *   Muestra los primeros 100 caracteres de la respuesta JSON para verificar visualmente que hay datos.

3.  **Prueba Endpoint `average-fare`**:
    *   Similar al anterior, prueba el endpoint de tarifa promedio.

### ¿Cuándo usarlo?
Use este script después de `verify_env.bat` para confirmar que, tras todo el procesamiento de datos, la API es capaz de leer y servir esa información correctamente.

---

## 3. `scripts\load_new_data.bat`

Script dedicado exclusivamente a la **carga incremental** de archivos.

*   **Función**: Escanea el directorio `data/raw` en tu máquina anfitriona y carga TODOS los archivos `.parquet` encontrados a HDFS.
*   **Soporte de Estructura**: 
    *   Este script es **recursivo**. Si organizas tus archivos en subcarpetas (ej. `data/raw/data-2025/archivo.parquet`, `data/raw/data-2009/archivo.parquet`), el script **preservará esa estructura** en HDFS.
    *   Los archivos quedarán en HDFS bajo `/data/nyc/raw/taxi-trips/data-2025/`, etc.
*   **Comportamiento**: 
    *   Usa el modo `append` (anexar).
    *   Genera un reporte detallado en `data/outputs-data/` con la fecha y hora de la carga.

## 4. `scripts\list_hdfs_files.bat`

Herramienta de visualización rápida.

*   **Función**: Muestra un listado de los archivos almacenados actualmente en el directorio principal de datos (`/data/nyc/raw/taxi-trips`).
*   **Uso**: Ejecútalo para verificar qué archivos han sido cargados exitosamente y ver sus tamaños.

## 5. `scripts\delete_hdfs_data.bat`

Herramienta de limpieza y gestión de datos.

*   **Modo Interactivo**: Si se ejecuta sin argumentos, muestra un menú:
    1.  **Eliminar TODO**: Borra el directorio `taxi-trips` completo y lo recrea vacío. Útil para reiniciar cargas desde cero.
    2.  **Eliminar específico**: T e pide el nombre de un archivo o carpeta específica para borrar.
*   **Modo Avanzado (Argumentos)**:
    *   `delete_hdfs_data.bat all`: Borra todo automáticamente.
    *   `delete_hdfs_data.bat nombre_archivo`: Borra solo ese archivo.

---

## 6. `scripts\pipeline_update.bat`

**Script Maestro de Actualización Automática**.

Este script unifica todo el ciclo de vida de los datos en un solo comando. Es la herramienta principal para actualizar el Dashboard con nueva información.

### Flujo de Trabajo
1.  **Carga (Ingesta)**: Ejecuta internamente `load_new_data.bat` para subir cualquier archivo nuevo desde `data/nyc/raw` a HDFS.
2.  **Procesamiento (Spark)**: Ejecuta secuencialmente:
    *   `clean_data.py`: Limpia y estandariza los nuevos registros.
    *   `analytics_basic.py` y `analytics_advanced.py`: Recalcula todas las métricas y KPIs globales (Ingresos, Tarifas, Viajes por Año).
3.  **Actualización**: Al finalizar, los datos en HDFS quedan listos. La API Node.js leerá automáticamente estos nuevos resultados en la próxima petición.

### ¿Cómo usarlo?
1.  Coloque sus nuevos archivos `.parquet` (ej. `yellow_tripdata_2024-01.parquet`) en la carpeta `data/nyc/raw`.
2.  Ejecute `scripts\pipeline_update.bat`.
3.  Espere a que termine el proceso.
4.  Recargue su navegador para ver las métricas actualizadas.
