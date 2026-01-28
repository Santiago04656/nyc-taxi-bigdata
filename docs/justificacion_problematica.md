# Justificación y Planteamiento del Problema

## 1. Introducción y Contexto

En la era actual de la información, el crecimiento exponencial en la generación de datos ha planteado desafíos significativos para los sistemas de procesamiento tradicionales. Este fenómeno, comúnmente enmarcado dentro del paradigma de Big Data, es particularmente evidente en el sector del transporte urbano y la movilidad inteligente. La ciudad de Nueva York, con su densa red de transporte y millones de viajes diarios, genera un volumen masivo de datos transaccionales y geoespaciales que, si se analizan correctamente, pueden ofrecer *insights* invaluables para la planificación urbana, la optimización de flotas y la comprensión de patrones económicos.

El presente proyecto se centra en el análisis del conjunto de datos "NYC Taxi Trip Record Data", una colección exhaustiva que abarca millones de registros de viajes.

## 2. Planteamiento del Problema

La problemática central que aborda este proyecto reside en las limitaciones inherentes de los sistemas de gestión de bases de datos relacionales (RDBMS) convencionales para manejar conjuntos de datos de gran volumen, velocidad y variedad.

Específicamente, se identifican los siguientes desafíos críticos:

*   **Volumen de Datos**: El almacenamiento y consulta eficiente de millones de registros históricos supera la capacidad de procesamiento vertical de servidores estándar, resultando en tiempos de respuesta inaceptables para análisis exploratorios o reportes operacionales.
*   **Complejidad en el Procesamiento**: La necesidad de realizar transformaciones complejas, limpiezas de datos (manejo de valores nulos, inconsistencias de tipos) y agregaciones temporales requiere un poder de cómputo distribuido que no ofrecen las soluciones monolíticas.
*   **Latencia en la Visualización**: La extracción de conocimiento útil para la toma de decisiones requiere tiempos de latencia mínimos. Los analistas y partes interesadas necesitan dashboards interactivos que respondan ágilmente, algo difícil de lograr si las consultas subyacentes deben escanear terabytes de datos crudos en tiempo de ejecución.

La ausencia de una arquitectura escalable impide a las organizaciones aprovechar estos datos para optimizar rutas, predecir la demanda o entender el impacto de eventos externos en el flujo vehicular.

## 3. Justificación del Desarrollo

El desarrollo de esta aplicación se justifica bajo la necesidad académica y técnica de implementar una arquitectura de referencia moderna para el procesamiento de Big Data. Se propone una solución que integra un ecosistema robusto basado en **Hadoop** para el almacenamiento distribuido (HDFS) y **Apache Spark** para el procesamiento en memoria.

Los motivos principales para la implementación de esta solución son:

1.  **Escalabilidad Horizontal**: Demostrar cómo el almacenamiento distribuido permite escalar el sistema añadiendo nodos al clúster, garantizando la viabilidad del proyecto a largo plazo a medida que el conjunto de datos crece históricamente.
2.  **Optimización del Procesamiento (ETL)**: Implementar pipelines de extracción, transformación y carga (ETL) eficientes que conviertan datos crudos (Parquet) en estructuras optimizadas para la lectura, reduciendo drásticamente los tiempos de consulta analítica.
3.  **Interoperabilidad y Visualización**: Cerrar la brecha entre el "Backend de Big Data" y el usuario final mediante una capa de API ligera y un Frontend reactivo. Esto demuestra la aplicabilidad práctica de las tecnologías de Big Data en productos de software orientados al usuario.

En conclusión, este proyecto no solo busca resolver el problema técnico del procesamiento de grandes volúmenes de datos de taxis de NYC, sino que también sirve como un ejercicio académico riguroso para validar arquitecturas de software distribuidas frente a desafíos de datos del mundo real.
