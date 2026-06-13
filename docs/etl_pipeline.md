# ETL Pipeline — SeismicPipeline

**Issue:** SEIS-11 — Limpieza y preprocesamiento del dataset sísmico  
**Asignado:** Daniel Sozoranga  
**Sprint:** Sprint 2 — ETL y Benchmarking

---

## 1. Descripción

Pipeline de limpieza y normalización aplicado al dataset crudo extraído de la API USGS (`usgs_raw.csv`). El objetivo es producir un dataset limpio (`usgs_clean.csv`) libre de duplicados, nulos críticos y valores físicamente imposibles.

---

## 2. Pasos del Pipeline (Sección 6 del notebook)

### 2.1 Eliminación de duplicados

Dos criterios aplicados en orden:

- **Por ID único USGS:** `df.drop_duplicates(subset='id', keep='first')`
- **Por posición espacio-temporal:** duplicados exactos en `tiempo_ms + latitud + longitud + magnitud`

### 2.2 Tratamiento de valores nulos

| Columna | Estrategia | Justificación |
|---|---|---|
| `magnitud`, `profundidad_km`, `latitud`, `longitud`, `tsunami` | Eliminar fila | Sin estos datos el registro es inutilizable |
| `num_estaciones`, `brecha_azimutal`, `significancia` | Imputar por mediana | Features de calidad; mediana es robusta a outliers |
| `dist_min_estacion`, `error_rms`, `intensidad_cdi/mmi` | No se usan | Excluidas por alta tasa de nulos (>40%) o leakage |

### 2.3 Normalización de tipos y rangos físicos

- Conversión explícita a `float64` e `int8` (target)
- Derivación de columnas `fecha` y `anio` desde `tiempo_ms`
- Corrección de longitudes > 180° al rango estándar [-180, 180]
- Filtro de rangos físicos válidos:
  - Magnitud: [5.0, 10.0]
  - Profundidad: [0, 750] km
  - Latitud/longitud: rangos estándar

### 2.4 Exportación

```python
RUTA_CLEAN = f"{DIRS['processed']}/usgs_clean.csv"
df_etl.to_csv(RUTA_CLEAN, index=False)
```

---

## 3. Resumen ANTES vs DESPUÉS

| Métrica | ANTES | DESPUÉS |
|---|---|---|
| Filas | ~25,000+ | Ver output del notebook |
| Nulos en features críticas | Presentes | 0 |
| Tipos de datos | Mixed (object, float) | Normalizados (float64, int8) |
| Duplicados | Presentes | 0 |
| Dataset exportado | — | `data/processed/usgs_clean.csv` |

---

## 4. Código de referencia

Ver **Sección 6** del notebook `notebooks/SeismicPipeline_Integrador_v3.ipynb`.
