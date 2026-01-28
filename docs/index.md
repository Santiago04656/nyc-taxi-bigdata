# Documentación del Proyecto: NYC Taxi Analytics

Bienvenido al centro de documentación del proyecto. Esta guía está organizada por **roles** para facilitar a los nuevos integrantes encontrar la información relevante según su área de trabajo.

## 🌟 Visión General (Lectura Recomendada para Todos)

Antes de profundizar en su área, se recomienda entender el contexto global del proyecto.

*   **[Arquitectura del Sistema](arquitectura.md)**: Diagrama y explicación de como interactúan Docker, Hadoop, Spark, Node.js y React.
*   **[Resumen Ejecutivo - Parte 1](resumen_parte_1.md)**: Origen del proyecto y objetivos iniciales.
*   **[Resumen Ejecutivo - Parte 2](resumen_parte_2.md)**: Evolución y estado actual.
*   **[Justificación y Problemática](justificacion_problematica.md)**: Análisis formal del problema de Big Data, desafíos técnicos y la justificación de la arquitectura elegida.

---

## 📊 Diagramas y Arquitectura Visual

Para una comprensión rápida y visual de todo el sistema:

*   **[Galería de Diagramas del Sistema](diagramas_sistema.md)**: (Consolidado de diagramas Mermaid explicados)
    1.  Despliegue (Docker/Contenedores)
    2.  Flujo General de Datos
    3.  Flujos Detallados por Rol (Infra, API, Frontend)

---

## 🐳 Equipo de Infraestructura y Datos (DevOps / Data Engineers)

Si eres responsable de **Docker, Hadoop, Spark** o los **Pipelines de Datos**, esta es tu sección.

1.  **[Guía de Despliegue y Comandos](despliegue_comandos.md)** (🚨 **Crítico**):
    *   Cómo levantar el entorno con Docker Compose.
    *   Comandos para iniciar, detener y reiniciar servicios.
    *   **Cómo cargar nuevos datos anuales**.
2.  **[Scripts de Automatización](scripts_automatizacion.md)**:
    *   Explicación técnica de `init.sh`, `verify_env.bat` y `check_api.bat`.
3.  **[Explicación de Carga y ETL](explicacion_carga.md)**:
    *   Detalle profundo de los jobs de Spark (`load_to_hdfs`, `clean_data`, `analytics`).
    *   Cómo se procesan los archivos Parquet.

---

## ⚙️ Equipo Backend (API Developers)

Si trabajas en la **API REST (Node.js/Express)** que sirve los datos al frontend.

1.  **[Documentación de la API](api_documentacion.md)**:
    *   Endpoints disponibles (V1 y V2).
    *   Estructuras de respuesta JSON.
    *   Lógica del servicio HDFS (`hdfs.service.js`).
2.  **Referencias útiles**:
    *   Revisa `despliegue_comandos.md` para saber cómo correr la API localmente.
    *   Consulta `arquitectura.md` para ver cómo la API conecta con Hadoop WebHDFS.

---

## 🎨 Equipo Frontend (Web Dashboard)

Si trabajas en la interfaz de usuario con **React, Next.js y TailwindCSS**.

1.  **[Documentación del Frontend](frontend_documentacion.md)**:
    *   Estructura del proyecto Next.js (`/frontend`).
    *   Componentes principales y librerías usadas (Recharts, Shadcn/ui).
    *   Gestión de estado y consumo de API con `useSWR`.
2.  **[Guía de Interfaces (UI/UX)](interfaces_gui.md)**:
    *   Diseño visual, paleta de colores y experiencia de usuario.
3.  **Para empezar**:
    *   Necesitarás correr el backend según `despliegue_comandos.md` para tener datos reales en tu UI.

---

## 🚀 Inicio Rápido

¿Acabas de llegar? Ejecuta esto para tener todo corriendo en 5 minutos:

1.  `docker-compose up -d --build`
2.  `.\scripts\verify_env.bat` (Espera a que termine)
3.  Abre `http://localhost:3000`
