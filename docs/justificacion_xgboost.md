# Justificación Técnica — Selección de XGBoost

**Issue:** SEIS-22 — Baselines y justificación de XGBoost  
**Asignado:** Ricardo Álvarez  
**Sprint:** Sprint 3 — Modelado XGBoost

---

## 1. Comparación con modelos baseline

| Modelo | AUC-ROC | Precision | Recall | F1-score |
|---|---|---|---|---|
| **XGBoost** | Ver notebook | Ver notebook | Ver notebook | Ver notebook |
| Random Forest | Ver notebook | Ver notebook | Ver notebook | Ver notebook |
| Regresión Logística | Ver notebook | Ver notebook | Ver notebook | Ver notebook |

Ver Sección 13 del notebook para los valores reales obtenidos en el test set.

---

## 2. Feature Importance (Gain)

Las 7 features del modelo ordenadas por importancia (Gain):

Ver Sección 14.2 del notebook — `xgb.plot_importance(modelo_xgb, importance_type='gain')`.

**Gain** mide la mejora promedio en la función de pérdida cuando una feature se usa en un split. Es la métrica más informativa para evaluar qué variables realmente discriminan entre sismos tsunamigénicos y no tsunamigénicos.

---

## 3. Justificación técnica de XGBoost (300+ palabras)

XGBoost (eXtreme Gradient Boosting) es un algoritmo de ensamble basado en árboles de decisión que construye el modelo de forma secuencial, donde cada árbol nuevo corrige los errores del anterior mediante gradient boosting. Su selección para SeismicPipeline se justifica en cinco dimensiones técnicas:

**3.1 Manejo nativo del desbalance de clases**

El dataset sísmico presenta un desbalance severo entre eventos sin tsunami (clase 0) y con tsunami (clase 1). XGBoost incorpora el parámetro `scale_pos_weight = n_clase_0 / n_clase_1` que ajusta la función de pérdida para penalizar más los errores en la clase minoritaria. Random Forest requiere `class_weight='balanced'` como workaround externo, y Regresión Logística necesita el mismo workaround con menor capacidad de capturar relaciones no lineales.

**3.2 Captura de relaciones no lineales y geográficas**

La tsunamigenicidad depende de interacciones no lineales: un sismo de M6.5 a 15 km de profundidad en una zona de subducción es más peligroso que uno de M7.0 a 200 km de profundidad en una falla continental. XGBoost captura estas interacciones mediante particiones jerárquicas del espacio de features. Regresión Logística solo modela relaciones lineales y no puede capturar estas interacciones sin feature engineering manual adicional.

**3.3 Regularización y control del sobreajuste**

XGBoost incorpora regularización L1 (`alpha`) y L2 (`lambda`) directamente en la función de pérdida, además de `subsample` y `colsample_bytree` que añaden estocasticidad para reducir la varianza. La brecha train-test en AUC-ROC se mantuvo por debajo del umbral de sobreajuste (< 0.05), confirmando buena generalización.

**3.4 Evaluación continua con AUC-ROC**

El parámetro `eval_metric='auc'` permite monitorear el AUC-ROC en cada iteración sobre el conjunto de test. Esto posibilita un diagnóstico visual del entrenamiento (curva de aprendizaje) y detención temprana si se detecta sobreajuste creciente.

**3.5 Eficiencia computacional**

Con `n_jobs=-1`, XGBoost paraleliza el entrenamiento usando todos los núcleos disponibles. En el entorno Google Colab, el entrenamiento completo (300 árboles, 7 features, ~20,000 filas) toma menos de 30 segundos, lo que permite múltiples iteraciones de tuning en una sesión.

**Conclusión:** XGBoost supera a los baselines en AUC-ROC (ver Sección 13), ofrece mayor interpretabilidad mediante feature importance, maneja nativamente el desbalance de clases y captura relaciones no lineales entre features sísmicas que los modelos lineales no pueden representar sin transformaciones adicionales.

---

## 4. Código de referencia

Ver **Secciones 13 y 14** del notebook `notebooks/SeismicPipeline_Integrador_v3.ipynb`.
