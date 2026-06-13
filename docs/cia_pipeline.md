# Propuesta Técnica CIA — SeismicPipeline

**Issue:** SEIS-25 — Confidencialidad, Integridad y Disponibilidad para consumo de APIs  
**Asignado:** Ricardo Álvarez  
**Sprint:** Sprint 4 — Dashboard and Security

---

## 1. Introducción

La tríada CIA (Confidencialidad, Integridad, Disponibilidad) es el marco fundamental de la ciberseguridad. SeismicPipeline consume datos de la API pública USGS y persiste resultados en Google Drive. Esta propuesta documenta las medidas implementadas y recomendadas para cada componente de la tríada en el contexto específico del pipeline.

---

## 2. Confidencialidad

*Garantizar que los datos y el código del pipeline solo sean accesibles por los miembros autorizados del equipo.*

### Medida C1 — Transmisión cifrada HTTPS

| Campo | Detalle |
|---|---|
| **Descripción** | Toda comunicación con la API USGS se realiza sobre HTTPS, cifrando los datos en tránsito. |
| **Implementación** | `requests.get(BASE_URL, ...)` donde `BASE_URL = 'https://earthquake.usgs.gov/...'`. La librería `requests` valida el certificado TLS por defecto (`verify=True`). |
| **Justificación** | Previene ataques Man-in-the-Middle que podrían interceptar o modificar los datos sísmicos en tránsito. |

### Medida C2 — Ausencia de credenciales hardcodeadas

| Campo | Detalle |
|---|---|
| **Descripción** | El pipeline no requiere API keys (la API USGS es pública), pero las rutas de Drive se gestionan mediante variables de código, no strings hardcodeados con tokens. |
| **Implementación** | `BASE = '/content/drive/MyDrive/SeismicPipeline'` — ruta relativa sin tokens. Para proyectos con autenticación: `token = os.environ.get('API_TOKEN')`. |
| **Justificación** | Elimina el riesgo de exposición de credenciales en el historial de commits de GitHub. |

### Medida C3 — Control de acceso al repositorio

| Campo | Detalle |
|---|---|
| **Descripción** | El repositorio `tsunami-risk-pipeline` es público (datos sísmicos son abiertos), pero las ramas `testing` y `main` requieren PR aprobado para merge. |
| **Implementación** | Branch protection rules en GitHub: `Require a pull request before merging` + `Require approving reviews: 1`. |
| **Justificación** | Garantiza que ningún cambio no revisado llegue a producción. |

---

## 3. Integridad

*Garantizar que los datos no sean alterados de forma no autorizada durante su ciclo de vida.*

### Medida I1 — Validación del esquema JSON de la API

| Campo | Detalle |
|---|---|
| **Descripción** | Cada respuesta de la API USGS es validada antes de procesarse para detectar cambios en la estructura del servicio. |
| **Implementación** | `if 'features' not in data: raise RuntimeError(...)` en `consultar_usgs()`. |
| **Justificación** | Protege contra respuestas malformadas o cambios en la API que podrían corromper silenciosamente el dataset. |

### Medida I2 — Validaciones con assert antes del entrenamiento

| Campo | Detalle |
|---|---|
| **Descripción** | El dataset de ML es validado programáticamente antes de entrenar el modelo para garantizar la integridad del target. |
| **Implementación** | `assert df_ml[TARGET].isna().sum() == 0` y `assert set(df_ml[TARGET].unique()) <= {0, 1}`. |
| **Justificación** | Detecta corrupción o manipulación del dataset antes de que afecte al modelo entrenado. |

### Medida I3 — Reproducibilidad con semilla global

| Campo | Detalle |
|---|---|
| **Descripción** | `SEED = 42` garantiza que todos los procesos aleatorios (splits, modelos) sean reproducibles y verificables. |
| **Implementación** | `random_state=SEED` en `train_test_split`, `XGBClassifier`, `RandomForestClassifier`. |
| **Justificación** | Permite detectar si los resultados han sido alterados al comparar con una ejecución de referencia. |

---

## 4. Disponibilidad

*Garantizar que el pipeline pueda ejecutarse cuando sea necesario, incluso ante fallos parciales.*

### Medida D1 — Retry con backoff exponencial

| Campo | Detalle |
|---|---|
| **Descripción** | Si la API USGS falla temporalmente, el pipeline reintenta automáticamente con esperas crecientes. |
| **Implementación** | `consultar_con_retry()`: esperas de 2, 4, 8 y 16 segundos entre reintentos (`espera = 2 ** (intento + 1)`). |
| **Justificación** | Evita que una falla transitoria de la API detenga la extracción de datos, mejorando la resiliencia del pipeline. |

### Medida D2 — Caché local del dataset en Drive

| Campo | Detalle |
|---|---|
| **Descripción** | El dataset crudo se persiste en Drive en la primera ejecución. Las siguientes cargan el CSV local sin volver a consumir la API. |
| **Implementación** | `if os.path.exists(RUTA_RAW): df_raw = pd.read_csv(RUTA_RAW)` — carga condicional. |
| **Justificación** | Si la API USGS está caída, el pipeline puede continuar con el dataset ya descargado. |

### Medida D3 — Timeout explícito en requests

| Campo | Detalle |
|---|---|
| **Descripción** | Cada request a la API tiene un timeout de 60 segundos para evitar que el pipeline quede bloqueado indefinidamente. |
| **Implementación** | `requests.get(BASE_URL, params=params, timeout=60)`. |
| **Justificación** | Garantiza que el pipeline siempre avance o falle de forma controlada, nunca se bloquee infinitamente. |

---

## 5. Resumen de controles por componente CIA

| Componente | Medida | Implementada |
|---|---|---|
| **Confidencialidad** | HTTPS con TLS verificado | ✅ |
| **Confidencialidad** | Sin credenciales hardcodeadas | ✅ |
| **Confidencialidad** | Branch protection en GitHub | ✅ |
| **Integridad** | Validación schema JSON respuesta | ✅ |
| **Integridad** | Assert target antes de entrenar | ✅ |
| **Integridad** | Semilla global SEED=42 | ✅ |
| **Disponibilidad** | Retry con backoff exponencial | ✅ |
| **Disponibilidad** | Caché local en Drive | ✅ |
| **Disponibilidad** | Timeout de 60s por request | ✅ |

---

## 6. Conclusión

SeismicPipeline implementa 9 controles de seguridad distribuidos en los tres componentes de la tríada CIA. El componente más robusto es la **Disponibilidad**, con tres mecanismos complementarios que garantizan la continuidad del pipeline ante fallos de la API. La **Integridad** está protegida por validaciones programáticas en puntos críticos del flujo. La **Confidencialidad** se garantiza mediante HTTPS y la ausencia de credenciales en el código fuente.
