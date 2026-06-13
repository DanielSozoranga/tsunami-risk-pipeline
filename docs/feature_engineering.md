# Feature Engineering — SeismicPipeline

**Issue:** SEIS-12 — Feature engineering y selección de variables  
**Asignado:** Daniel Sozoranga  
**Sprint:** Sprint 2 — ETL y Benchmarking

---

## 1. Variables seleccionadas (7 features)

| Feature | Nombre original API | Justificación técnica |
|---|---|---|
| `magnitud` | `mag` | Principal predictor de tsunamigenicidad. Sismos M≥7.0 generan el 90% de tsunamis históricos |
| `profundidad_km` | `depth` | Crítica: sismos superficiales (<70 km) tienen mayor potencial tsunamigénico por deformación del fondo marino |
| `latitud` | `latitude` | Contexto tectónico geográfico — zonas de subducción específicas tienen mayor riesgo |
| `longitud` | `longitude` | Idem latitud — permite al modelo aprender patrones regionales del Ring of Fire |
| `significancia` | `sig` | Índice compuesto USGS que combina magnitud, intensidad y reportes. Correlación r=+0.42 con target |
| `num_estaciones` | `nst` | Calidad de localización — más estaciones = epicentro más preciso |
| `brecha_azimutal` | `gap` | Cobertura angular de estaciones — brechas grandes indican menor precisión de localización |

---

## 2. Variables excluidas y razón

| Variable | Razón de exclusión |
|---|---|
| `reportes_sentido` (felt) | **Leakage** — solo disponible después del evento |
| `intensidad_cdi` | **Leakage** — ídem |
| `intensidad_mmi` | **Leakage** — ídem |
| `dist_min_estacion` | >40% de valores nulos — ver Sección 7.3 |
| `error_rms` | Correlación baja con target (r<0.05) |
| `nivel_alerta_pager` | Categórica con >60% nulos |

---

## 3. Target

```python
TARGET = 'tsunami'   # Binario: 0 = no tsunami | 1 = tsunami (flag oficial USGS)
```

- **Clase 0 (no tsunami):** mayoría de eventos
- **Clase 1 (tsunami):** minoría — justifica `scale_pos_weight` en XGBoost
- Verificación: sin data leakage, sin nulos, valores únicos = {0, 1}

---

## 4. Dataset final de ML

```python
FEATURES = [
    'magnitud', 'profundidad_km', 'latitud', 'longitud',
    'significancia', 'num_estaciones', 'brecha_azimutal'
]
TARGET = 'tsunami'
```

Exportado a: `data/processed/dataset_ml.csv`

---

## 5. Código de referencia

Ver **Secciones 7.4, 7.5 y 9.1** del notebook `notebooks/SeismicPipeline_Integrador_v3.ipynb`.
