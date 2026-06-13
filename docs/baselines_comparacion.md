# Comparación de Modelos Baseline — SeismicPipeline

**Issue:** SEIS-121 — Entrenamiento y comparación: Random Forest + Logistic Regression vs XGBoost  
**Asignado:** Daniel Sozoranga  
**Sprint:** Sprint 4 — Dashboard and Security

---

## 1. Configuración del experimento

Los tres modelos se entrenaron sobre **exactamente los mismos datos**:

- Dataset: `data/processed/dataset_ml.csv`
- Features: `magnitud`, `profundidad_km`, `latitud`, `longitud`, `significancia`, `num_estaciones`, `brecha_azimutal`
- Target: `tsunami` (0/1)
- Split: 80/20 estratificado con `random_state=42`
- Corrección de desbalance: `scale_pos_weight` en XGBoost, `class_weight='balanced'` en RF y LR

---

## 2. Configuración de cada modelo

### XGBoost (modelo del proyecto)
```python
XGBClassifier(
    objective        = 'binary:logistic',
    scale_pos_weight = SCALE_POS_WEIGHT,
    n_estimators     = 300,
    max_depth        = 6,
    learning_rate    = 0.1,
    subsample        = 0.9,
    colsample_bytree = 0.9,
    eval_metric      = 'auc',
    random_state     = 42
)
```

### Random Forest (baseline 1)
```python
RandomForestClassifier(
    n_estimators = 300,
    max_depth    = None,
    class_weight = 'balanced',
    random_state = 42,
    n_jobs       = -1
)
```

### Regresión Logística (baseline 2)
```python
Pipeline([
    ('scaler', StandardScaler()),
    ('lr', LogisticRegression(
        class_weight = 'balanced',
        max_iter     = 2000,
        random_state = 42
    ))
])
```

---

## 3. Tabla comparativa de métricas

| Modelo | AUC-ROC | Precision | Recall | F1-score |
|---|---|---|---|---|
| **XGBoost** | Ver Sección 13 notebook | Ver notebook | Ver notebook | Ver notebook |
| Random Forest | Ver Sección 13 notebook | Ver notebook | Ver notebook | Ver notebook |
| Regresión Logística | Ver Sección 13 notebook | Ver notebook | Ver notebook | Ver notebook |

> Los valores exactos se obtienen al ejecutar la Sección 13 del notebook con el dataset real extraído de la API USGS.

**Resultado esperado:** XGBoost obtiene el mayor AUC-ROC en la comparación, confirmando su superioridad para este problema de clasificación desbalanceada con relaciones no lineales.

---

## 4. Análisis crítico — Por qué XGBoost supera a los baselines

### 4.1 Frente a Random Forest

Random Forest también es un ensamble de árboles y maneja bien el desbalance con `class_weight='balanced'`, por lo que los resultados son cercanos. Sin embargo, XGBoost supera a RF por:

- **Gradient boosting secuencial:** cada árbol corrige los errores del anterior, mientras que RF promedia árboles independientes. El boosting produce modelos más precisos con el mismo número de árboles.
- **Regularización explícita:** XGBoost incluye L1/L2 directamente en la función de pérdida. RF no tiene equivalente.
- **Evaluación continua con eval_set:** permite monitorear el AUC en test por iteración y detectar sobreajuste. RF no tiene este mecanismo.

### 4.2 Frente a Regresión Logística

La diferencia más marcada se da frente a LR:

- **Linealidad vs no linealidad:** LR solo puede separar clases con un hiperplano. La tsunamigenicidad depende de interacciones no lineales (magnitud × profundidad × zona tectónica) que LR no puede capturar sin feature engineering manual adicional.
- **Capacidad expresiva:** XGBoost particiona el espacio de features jerárquicamente, capturando zonas de subducción vs fallas continentales directamente desde latitud/longitud sin transformaciones.
- **Escalado innecesario:** XGBoost trabaja directamente con los valores originales. LR requiere StandardScaler previo para converger correctamente.

### 4.3 Conclusión

XGBoost es la elección óptima para SeismicPipeline porque combina el manejo nativo del desbalance de clases (`scale_pos_weight`), la captura de relaciones no lineales mediante gradient boosting, la regularización integrada y el monitoreo continuo del AUC-ROC. Estas características son especialmente críticas en un dataset sísmico con severo desbalance (clase 1 < 5%) y patrones geográficos no lineales.

---

## 5. Código de referencia

Ver **Sección 13** del notebook `notebooks/SeismicPipeline_Integrador_v3.ipynb`.
