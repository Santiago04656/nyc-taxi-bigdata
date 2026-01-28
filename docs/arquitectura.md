# Arquitectura y Funcionamiento de la Aplicación

Este documento describe la arquitectura, los componentes y el flujo de datos de la aplicación "NYC Taxi Big Data Analytics".

## Descripción General

El sistema es una plataforma de análisis de "Big Data" diseñada para procesar grandes volúmenes de datos de viajes de taxis de Nueva York. Utiliza un ecosistema basado en **Hadoop** para el almacenamiento distribuido y **Spark** para el procesamiento paralelo, expuesto finalmente a través de una **API REST en Node.js**.

## Componentes del Sistema

El sistema está contenerizado utilizando Docker y consta de los siguientes servicios principales:

1.  **Hadoop NameNode (`hadoop-namenode`)**:
    *   **Función**: Es el maestro del sistema de archivos distribuido (HDFS). Gestiona los metadatos de los archivos (nombres, permisos, ubicación de bloques).
    *   **Puerto**: 9870 (Web UI), 8020 (IPC).

2.  **Hadoop DataNode (`hadoop-datanode`)**:
    *   **Función**: Almacena los datos reales en bloques. Se comunica con el NameNode para registrarse y reportar su estado.
    *   **Configuración Clave**: Se conecta al NameNode via `hdfs://hadoop-namenode:8020`.

3.  **Spark Master (`spark-master`)**:
    *   **Función**: Coordinador del clúster de Spark. Distribuye las tareas de procesamiento entre los trabajadores.
    *   **Puerto**: 8080 (Web UI), 7077 (Master URL).

4.  **Spark Worker (`spark-worker`)**:
    *   **Función**: Ejecuta las tareas de cómputo (Jobs) asignadas por el Master.
    *   **Recursos**: Configurado para utilizar CPU y memoria para procesar datos en paralelo.

5.  **Node.js API (`nyc-taxi-api`)**:
    *   **Función**: Servidor web que expone los resultados del análisis a clientes externos (como un frontend o usuarios vía cURL).
    *   **Librerías**: Utiliza `webhdfs` para leer directamente los archivos JSON de resultados almacenados en HDFS.
    *   **Puerto**: 3000.

## Flujo de Datos

El procesamiento de la información sigue una tubería (pipeline) de tres etapas:

### 1. Ingesta (Carga)
*   **Origen**: Archivos Parquet ubicados localmente en `data/raw/`.
*   **Proceso**: Un trabajo de Spark (`load_to_hdfs.py`) lee estos archivos locales y los escribe en el sistema de archivos HDFS en la ruta `/data/nyc/raw/taxi-trips`.
*   **Objetivo**: Centralizar los datos crudos en el sistema distribuido.

### 2. Procesamiento (Limpieza y Análisis)
*   **Limpieza (`clean_data.py`)**:
    *   Lee los datos crudos de HDFS.
    *   Filtra registros inválidos (ubicaciones nulas, tarifas negativas).
    *   Escribe los datos limpios en `/data/nyc/processed/cleaned-trips`.
*   **Analítica (`analytics_basic.py`)**:
    *   Lee los datos limpios.
    *   Realiza agregaciones:
        *   Viajes por hora (`trips-by-hour`).
        *   Tarifa promedio (`avg-fare`).
    *   Guarda los resultados en formato JSON en `/data/nyc/analytics/`.

### 3. Consumo (API)
*   La API Node.js recibe una petición HTTP (ej. `GET /api/trips-per-hour`).
*   El servicio interno consulta HDFS buscando los archivos JSON generados en la etapa de análisis.
*   Retorna los datos procesados al cliente en formato JSON.

## Tecnologías Utilizadas
*   **Docker & Docker Compose**: Orquestación de contenedores.
*   **Apache Hadoop (HDFS)**: Almacenamiento persistente y distribuido.
*   **Apache Spark (PySpark)**: Motor de procesamiento de datos en memoria.
*   **Node.js & Express**: Backend ligero para la capa de presentación de datos.
