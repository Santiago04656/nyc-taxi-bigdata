# Interfaces Gráficas (GUI) y URLs del Sistema

El sistema proporciona varias interfaces web para monitorear el estado de los componentes de Big Data y acceder a los datos.

## 1. Hadoop HDFS (NameNode UI)

Esta interfaz permite explorar el sistema de archivos distribuido, ver el estado de los DataNodes y monitorear el espacio en disco.

*   **URL**: [http://localhost:9870](http://localhost:9870)
*   **Uso Principal**:
    *   Ir a la pestaña **Utilities** -> **Browse the file system** para ver los archivos cargados.
    *   Navegar a `/data/nyc/` para verificar que las carpetas `raw`, `processed` y `analytics` existen y contienen datos.

## 2. Spark Master UI

Dashboard principal del clúster de Spark. Muestra los recursos disponibles (Workers, Cores, Memoria) y el historial de las aplicaciones ejecutadas.

*   **URL**: [http://localhost:8080](http://localhost:8080)
*   **Uso Principal**:
    *   Verificar que hay **Workers** registrados y en estado `ALIVE`.
    *   Monitorear **Running Applications** y **Completed Applications** para ver si nuestros scripts (`clean_data`, `analytics`) se ejecutaron exitosamente o fallaron.

## 3. Node.js API (Backend)

Es el servidor que expone los datos procesados. Aunque no tiene una interfaz gráfica de usuario (GUI) compleja, responde a peticiones HTTP.

*   **URL Base**: [http://localhost:3000](http://localhost:3000)
*   **Endpoints Disponibles**:
    *   `GET /api/trips-per-hour`: Retorna estadísticas de viajes agrupados por hora.
    *   `GET /api/average-fare`: Retorna el cálculo de la tarifa promedio.
    *   `GET /`: No definido (Retorna 404).

## Tabla Resumen de Puertos

| Servicio | Puerto Interno | Puerto Expuesto (Host) | Descripción |
|----------|----------------|------------------------|-------------|
| **HDFS Web UI** | 9870 | 9870 | Panel de control de Hadoop |
| **HDFS IPC** | 8020 | 8020 | Comunicación entre nodos (no HTTP) |
| **Spark Master UI** | 8080 | 8080 | Panel de control de Spark |
| **Spark Master** | 7077 | 7077 | Envío de trabajos (spark-submit) |
| **API Node.js** | 3000 | 3000 | Acceso a datos procesados |
