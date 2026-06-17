# Pipeline ETL - Arquitectura Medallion en Lakehouse Relacional

Este repositorio contiene la resolución de la prueba técnica para la posición de Data Engineer Trainee. El objetivo del proyecto es diseñar e implementar un pipeline robusto de procesamiento de datos enfocado en la consistencia, integridad relacional y analítica de negocio.

## 🚀 Arquitectura del Proyecto (Medallion)

El flujo de datos se estructuró siguiendo las mejores prácticas de la industria mediante una arquitectura de tres capas:

1. **Capa Bronze (RAW):** Ingesta fiel e inmutable de las fuentes crudas (`customers`, `products`, `orders`).
2. **Capa Silver (STAGE):** Limpieza pesada, estandarización de catálogos, tipificación estricta, manejo de nulos lógicos y validación cruzada.
3. **Capa Gold (ANALYTICS):** Modelado multidimensional (Esquema Estrella) enfocado en responder a las necesidades analíticas del negocio a través de la tabla central unificada `FACT_VENTAS`.

---

## 💡 Fundamentos y Decisiones de Ingeniería de Datos

### 1. ¿Por qué se utilizó `dtype=str` en la ingesta (Capa RAW)?
No fue una restricción del código, sino una **decisión de diseño para garantizar la robustez**. En entornos productivos, dejar que el motor infiera los tipos automáticamente sobre fuentes sucias puede corromper datos (como truncar ceros a la izquierda en identificadores) o lanzar excepciones críticas (`TypeError`). Forzar la lectura como texto puro aísla la ingesta, asegurando que el 100% de las filas entren al sistema para ser procesadas de manera controlada en la capa de transformación.

### 2. ¿Por qué se implementó SQLite como almacenamiento relacional?
Se optó por SQLite debido a criterios de **portabilidad, eficiencia y reproducibilidad**:
* **Serverless y Autocontenido:** Toda la base de datos relacional vive en un único archivo físico (`lakehouse.db`). Esto elimina la necesidad de configurar servidores locales o credenciales complejas, garantizando que el evaluador pueda replicar el pipeline inmediatamente en cualquier entorno.
* **Integración:** Permite una transición fluida y de alta velocidad entre estructuras en memoria de Pandas y tablas indexadas con SQL real.

### 3. Preservación de Métricas de Negocio Derivadas (`dias_envio`)
En la construcción de la tabla de hechos final `FACT_VENTAS`, se removieron las columnas de metadatos técnicas de auditoría de Stage (`updated_at`), pero se retuvo y calculó la columna derivada `dias_envio`. En la ingeniería de datos moderna, las tablas de hechos deben enriquecerse con métricas atómicas calculadas que aporten un valor analítico directo (KPIs de logística y cumplimiento de SLAs), optimizando el rendimiento de las consultas posteriores en herramientas de BI.

### 4. Seguridad de la Información (Tratamiento de PII)
Se aplicó un enfoque riguroso de seguridad mediante dos técnicas diferenciadas:
* **Hashing Criptográfico (SHA-256):** Aplicado sobre los correos electrónicos. Al ser una función unidireccional e irreversible, protege la identidad del cliente pero preserva la unicidad del dato, permitiendo análisis de comportamiento de usuarios sin exponer datos reales.
* **Enmascaramiento Dinámico (Masking):** Aplicado en números telefónicos y tarjetas, ocultando los datos sensibles expuestos y manteniendo únicamente los dígitos finales necesarios para validaciones operativas.
