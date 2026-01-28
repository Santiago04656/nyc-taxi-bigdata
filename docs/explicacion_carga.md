# Explicación del Proceso de Carga: Particionamiento en Spark

Este documento explica el comportamiento observado durante la carga de datos en HDFS, específicamente por qué la cantidad de archivos resultantes puede ser mayor que la cantidad de archivos originales.

## El Fenómeno Observado

Al cargar, por ejemplo, **2 archivos** Parquet originales (`yellow_tripdata_2025-01.parquet` y `...-02.parquet`), es común observar que en HDFS aparecen **4 archivos** o más con nombres como `part-00000-...`, `part-00001-...`.

Esto es un comportamiento **normal, esperado y deseable** en sistemas de procesamiento distribuido como Apache Spark.

## ¿Por qué ocurre esto?

El motivo principal es el **Procesamiento en Paralelo**. Spark no procesa ni guarda los datos como un único bloque monolítico (a diferencia de un editor de texto convencional). Para maximizar la velocidad y eficiencia, divide los datos en fragmentos más pequeños llamados **particiones**.

### La Mecánica Interna

1.  **Recursos del Cluster**: En nuestro archivo `docker-compose.yml`, hemos configurado los trabajadores de Spark (`spark-worker`) para utilizar **2 núcleos** de CPU (`SPARK_WORKER_CORES=2`).
2.  **División de Tareas**: Por defecto, Spark intenta paralelizar la lectura y escritura utilizando todos los núcleos disponibles.

### Ejemplo Práctico (El caso de 2 archivos vs 4 salidas)

El flujo matemático aproximado que sigue Spark es el siguiente:

1.  **Lectura del Archivo 1**: Spark lo lee y decide dividirlo en 2 partes para que ambos núcleos trabajen a la vez escribiendo en el disco.
    *   Núcleo 1 escribe -> `part-00000-UUID_A.parquet`
    *   Núcleo 2 escribe -> `part-00001-UUID_A.parquet`
2.  **Lectura del Archivo 2**: Repite el proceso.
    *   Núcleo 1 escribe -> `part-00000-UUID_B.parquet`
    *   Núcleo 2 escribe -> `part-00001-UUID_B.parquet`

**Resultado Final**: 2 Archivos de Entrada × 2 Particiones/Núcleos = **4 Archivos de Salida**.

## Beneficios de este Comportamiento

Lejos de ser un error, esta fragmentación ofrece ventajas críticas para Big Data:

*   **Lectura Paralela**: Cuando ejecutes análisis posteriores (como calcular viajes por hora), Spark podrá leer esos 4 archivos simultáneamente, reduciendo el tiempo de procesamiento a la mitad en comparación con leer un solo archivo gigante secuencialmente.
*   **Escalabilidad**: Si tu dataset crece a Terabytes, Spark simplemente creará más particiones (miles de archivos `part-XXXXX`), permitiendo que cientos de computadoras trabajen en ellos al mismo tiempo sin saturarse.

## Conclusión

La presencia de múltiples archivos `part-XXXXX` en HDFS confirma que el sistema está utilizando correctamente los recursos paralelos asignados en la configuración de Docker.
