# Análisis de Seguridad — API USGS

**Issue:** SEIS-17 — Vulnerabilidades de autenticación y seguridad API USGS  
**Asignado:** Ricardo Álvarez  
**Sprint:** Sprint 3 — Modelado XGBoost

---

## 1. Descripción de la superficie de ataque

La API USGS FDSN es un servicio público sin autenticación. El pipeline SeismicPipeline realiza 35 requests HTTP GET por ejecución. Los vectores de ataque evaluados son:

| Vector | Descripción |
|---|---|
| Inyección de parámetros | Manipulación de query params (`minmagnitude`, fechas, bbox) |
| Rate limiting | Comportamiento del servidor ante requests masivos |
| Man-in-the-Middle | Ausencia de validación de certificado TLS |
| Denegación de servicio | Impacto de caída de la API en el pipeline |

---

## 2. Vulnerabilidades identificadas

### 2.1 Ausencia de autenticación (Informativo)

| Campo | Detalle |
|---|---|
| **Severidad** | Informativa |
| **Descripción** | La API no requiere API key ni token. Cualquier cliente puede consumirla sin restricción de identidad. |
| **Impacto en el proyecto** | Bajo. El pipeline solo lee datos públicos. No expone credenciales propias. |
| **Mitigación aplicada** | Ninguna requerida. Documentar que no se almacenan tokens de la API. |

### 2.2 Rate limiting permisivo (Bajo)

| Campo | Detalle |
|---|---|
| **Severidad** | Baja |
| **Descripción** | Prueba realizada: 10 requests consecutivos sin `time.sleep()`. La API respondió sin bloqueos ni HTTP 429. |
| **Impacto** | Riesgo de IP ban o throttling en ejecuciones agresivas sin pausa. |
| **Mitigación aplicada** | `time.sleep(0.5)` entre requests anuales. Retry con backoff exponencial ante errores 5XX. |

### 2.3 Sin validación del esquema de respuesta JSON (Medio)

| Campo | Detalle |
|---|---|
| **Severidad** | Media |
| **Descripción** | Si la API cambia su estructura GeoJSON, el pipeline falla silenciosamente con `KeyError`. |
| **Impacto** | Corrupción del dataset sin detección. |
| **Mitigación aplicada** | Validación de presencia de la clave `features` en `consultar_usgs()`. Lanza `RuntimeError` si falta. |

### 2.4 Parámetros no sanitizados (Bajo)

| Campo | Detalle |
|---|---|
| **Severidad** | Baja |
| **Descripción** | Los parámetros del query string (fechas, magnitud, bbox) se construyen desde constantes en el código, no desde input del usuario. No hay vector de inyección real. |
| **Impacto** | Negligible en el contexto actual. |
| **Mitigación aplicada** | Constantes definidas en código (`MIN_MAGNITUD`, `BBOX_RING_OF_FIRE`). No se acepta input externo. |

### 2.5 Transmisión HTTP sin HTTPS verificado (Bajo)

| Campo | Detalle |
|---|---|
| **Severidad** | Baja |
| **Descripción** | La librería `requests` valida el certificado TLS por defecto (`verify=True`). La URL usa HTTPS. |
| **Impacto** | Riesgo teórico ante ataques MitM en redes no confiables. |
| **Mitigación aplicada** | No se usa `verify=False` en ningún punto del pipeline. |

---

## 3. Clasificación de riesgos (CVSS simplificado)

| Vulnerabilidad | Probabilidad | Impacto | Nivel |
|---|---|---|---|
| Sin autenticación | Alta | Ninguno (datos públicos) | Informativa |
| Rate limiting permisivo | Media | Bajo | Baja |
| Sin validación JSON schema | Media | Alto | Media |
| Parámetros no sanitizados | Baja | Bajo | Baja |
| TLS sin verificación forzada | Baja | Medio | Baja |

---

## 4. Controles implementados en el pipeline

```python
# Control 1: Validacion de respuesta JSON
if 'features' not in data:
    raise RuntimeError(f"Respuesta inesperada: {str(data)[:200]}")

# Control 2: Rate limiting voluntario
time.sleep(0.5)  # Entre cada request anual

# Control 3: Retry con backoff exponencial
espera = 2 ** (intento + 1)  # 2, 4, 8, 16 segundos

# Control 4: Timeout explicito
resp = requests.get(BASE_URL, params=params, timeout=60)
```

---

## 5. Conclusión

La API USGS presenta una superficie de ataque reducida para el pipeline SeismicPipeline dado que es una fuente de datos de solo lectura y pública. El riesgo más relevante es la **ausencia de validación del esquema JSON** (Severidad: Media), mitigado con la verificación de la clave `features` antes de procesar la respuesta. Los demás controles están implementados y documentados en la Sección 3.4 del notebook.
