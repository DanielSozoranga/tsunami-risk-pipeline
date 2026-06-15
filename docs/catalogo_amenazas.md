# Catálogo de Amenazas y Mitigaciones — SeismicPipeline

**Issue:** SEIS-26 — Amenazas identificadas y estrategias de mitigación  
**Asignado:** Ricardo Álvarez  
**Sprint:** Sprint 4 — Dashboard and Security

---

## 1. Introducción

Este catálogo consolida las amenazas identificadas en los análisis de seguridad de la API USGS (SEIS-17) y de la infraestructura Google Colab (SEIS-24), priorizadas por nivel de riesgo compuesto (Probabilidad × Impacto) para facilitar la toma de decisiones de mitigación.

---

## 2. Catálogo de amenazas

### AMENAZA 1 — Exposición de credenciales en el notebook

| Campo | Detalle |
|---|---|
| **Origen** | Google Colab |
| **Descripción** | Hardcodeo de tokens, rutas con credenciales o API keys en celdas del notebook que luego se commiten a GitHub con outputs incluidos. |
| **Probabilidad** | Media |
| **Impacto** | Alto |
| **Nivel de riesgo** | 🔴 ALTO |
| **Mitigación** | Usar `os.environ.get()`. Limpiar outputs antes de commit. Agregar regla en `.gitignore` para outputs. |

### AMENAZA 2 — Session hijacking en Colab

| Campo | Detalle |
|---|---|
| **Origen** | Google Colab |
| **Descripción** | Extracción del token de autenticación de Google desde `~/.config/gcloud/` durante una sesión activa para suplantar la identidad del usuario. |
| **Probabilidad** | Baja |
| **Impacto** | Alto |
| **Nivel de riesgo** | 🔴 ALTO |
| **Mitigación** | Revocar tokens al finalizar sesión. Usar cuentas de servicio con permisos mínimos. No compartir sesiones activas. |

### AMENAZA 3 — Corrupción silenciosa del dataset por cambio de API

| Campo | Detalle |
|---|---|
| **Origen** | API USGS |
| **Descripción** | La API USGS cambia su estructura GeoJSON sin previo aviso. El pipeline extrae datos con campos incorrectos o ausentes sin detectar el error, produciendo un dataset corrupto que llega al modelo. |
| **Probabilidad** | Baja |
| **Impacto** | Alto |
| **Nivel de riesgo** | 🔴 ALTO |
| **Mitigación** | Validar presencia de `features` en cada respuesta. Validar columnas esperadas después de la extracción. Logs de extracción por año para detectar anomalías. |

### AMENAZA 4 — Acceso no autorizado a Google Drive montado

| Campo | Detalle |
|---|---|
| **Origen** | Google Colab |
| **Descripción** | Un notebook malicioso compartido puede leer, modificar o exfiltrar archivos del Drive montado (`/content/drive/`) sin que el usuario lo detecte. |
| **Probabilidad** | Media |
| **Impacto** | Medio |
| **Nivel de riesgo** | 🟠 MEDIO |
| **Mitigación** | Solo abrir notebooks de fuentes confiables con Drive montado. Usar carpetas compartidas con permisos limitados. |

### AMENAZA 5 — Denegación de servicio por caída de la API USGS

| Campo | Detalle |
|---|---|
| **Origen** | API USGS |
| **Descripción** | La API USGS no está disponible durante la extracción. Sin mecanismo de resiliencia, el pipeline falla completamente y se pierde la sesión de Colab. |
| **Probabilidad** | Media |
| **Impacto** | Medio |
| **Nivel de riesgo** | 🟠 MEDIO |
| **Mitigación** | Retry con backoff exponencial. Caché local del dataset en Drive. Estrategia de carga condicional `if os.path.exists(RUTA_RAW)`. |

### AMENAZA 6 — Ejecución de código oculto en notebooks compartidos

| Campo | Detalle |
|---|---|
| **Origen** | Google Colab |
| **Descripción** | Celdas con `cellView: form` o código base64 ofuscado que se ejecuta sin ser visible en la interfaz de Colab. Puede instalar backdoors o exfiltrar datos. |
| **Probabilidad** | Baja |
| **Impacto** | Medio |
| **Nivel de riesgo** | 🟠 MEDIO |
| **Mitigación** | Inspeccionar el JSON del `.ipynb` antes de ejecutar notebooks de terceros. Ejecutar en cuentas sandbox. |

### AMENAZA 7 — Rate limiting y bloqueo de IP por la API USGS

| Campo | Detalle |
|---|---|
| **Origen** | API USGS |
| **Descripción** | Requests masivos sin pausa pueden resultar en bloqueo temporal de la IP de salida de Colab, interrumpiendo la extracción. |
| **Probabilidad** | Media |
| **Impacto** | Bajo |
| **Nivel de riesgo** | 🟡 BAJO |
| **Mitigación** | `time.sleep(0.5)` entre requests anuales. Paginación anual (35 requests) en lugar de un único request masivo. |

### AMENAZA 8 — Contaminación del modelo por data leakage

| Campo | Detalle |
|---|---|
| **Origen** | Pipeline ML |
| **Descripción** | Inclusión accidental de variables post-evento (`felt`, `cdi`, `mmi`) como features del modelo, produciendo métricas infladas que no se replican en producción real. |
| **Probabilidad** | Media |
| **Impacto** | Alto |
| **Nivel de riesgo** | 🔴 ALTO |
| **Mitigación** | Lista explícita de features en `FEATURES`. Documentación de variables excluidas con justificación de leakage. Validación en Sección 7.5 del notebook. |

---

## 3. Tabla resumen — Priorizada por nivel de riesgo

| # | Amenaza | Origen | Prob. | Impacto | Riesgo |
|---|---|---|---|---|---|
| 1 | Exposición credenciales notebook | Colab | Media | Alto | 🔴 ALTO |
| 2 | Session hijacking | Colab | Baja | Alto | 🔴 ALTO |
| 3 | Corrupción dataset por cambio API | USGS | Baja | Alto | 🔴 ALTO |
| 4 | Data leakage en features | Pipeline ML | Media | Alto | 🔴 ALTO |
| 5 | Acceso Drive montado | Colab | Media | Medio | 🟠 MEDIO |
| 6 | Caída API USGS (DoS) | USGS | Media | Medio | 🟠 MEDIO |
| 7 | Código oculto en notebooks | Colab | Baja | Medio | 🟠 MEDIO |
| 8 | Rate limiting y bloqueo IP | USGS | Media | Bajo | 🟡 BAJO |

---

## 4. Conclusión

El catálogo identifica 8 amenazas, de las cuales 4 son de nivel **Alto** y requieren controles inmediatos. Las amenazas críticas se concentran en la gestión de credenciales (Colab) y la integridad del pipeline de datos (API USGS + leakage). Los controles implementados en SeismicPipeline cubren el 100% de las amenazas identificadas con al menos una medida de mitigación activa. Las amenazas de nivel Medio y Bajo requieren controles de proceso adicionales documentados en el plan de gobernanza (SEIS-10).
