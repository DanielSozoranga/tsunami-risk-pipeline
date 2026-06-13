# Gobernanza y Gestión de Infraestructura TI — SeismicPipeline

**Proyecto:** SeismicPipeline — Predicción de Riesgo Tsunamigénico con XGBoost  
**Equipo:** Daniel Sozoranga (Scrum Master) · Ricardo Álvarez (Product Owner)  
**Universidad:** Universidad Internacional del Ecuador (UIDE) — 2026

---

## 1. Herramientas TI del Proyecto

| Herramienta | Rol en el proyecto | Justificación |
|---|---|---|
| **Google Colab** | Entorno de ejecución del notebook | GPU/CPU gratuita, integración nativa con Drive |
| **Google Drive** | Almacenamiento persistente de datos y modelos | Acceso compartido entre integrantes, persistencia entre sesiones |
| **GitHub** | Control de versiones y colaboración | Historial de cambios, branching, integración con Jira |
| **Jira (SEIS)** | Gestión del proyecto Scrum | Tablero, sprints, automatización GitHub→Jira |
| **GitHub Codespaces** | Entorno de desarrollo en la nube | Consistencia de entorno, acceso desde cualquier equipo |

---

## 2. Diagrama de Arquitectura del Flujo de Datos

```
API USGS (FDSN)
      |
      | HTTP GET (35 requests anuales)
      v
extraccion masiva
(consultar_usgs + retry backoff)
      |
      v
/data/raw/usgs_raw.csv          <- Google Drive
      |
      v
Pipeline ETL (Seccion 6)
- Eliminar duplicados
- Imputar nulos por mediana
- Normalizar tipos y rangos
      |
      v
/data/processed/usgs_clean.csv  <- Google Drive
      |
      v
Feature Engineering (7 vars)
+ Split estratificado 80/20
+ scale_pos_weight
      |
      v
XGBoost BinaryClassifier
(objective=binary:logistic)
      |
      v
/models/xgboost_model.pkl       <- Google Drive
      |
      v
predict_proba -> score por provincia costera
      |
      v
Dashboard Plotly + Mapa Ecuador
/data/processed/dashboard_interactivo.html
```

---

## 3. Riesgos de Infraestructura y Plan de Mitigación

| # | Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|---|
| 1 | **Caída de la API USGS** | Media | Alto | Retry con backoff exponencial (2, 4, 8, 16 seg); dataset respaldado en Drive |
| 2 | **Pérdida de sesión en Colab** | Alta | Medio | Dataset persistido en Drive; carga condicional (`if os.path.exists`) |
| 3 | **Rate limiting de la API** | Media | Medio | Paginación anual + `time.sleep(0.5)` entre requests |
| 4 | **Corrupción del dataset** | Baja | Alto | Validaciones con `assert` antes del entrenamiento; versiones en Drive |
| 5 | **Pérdida del modelo serializado** | Baja | Alto | Modelo exportado con `joblib`; re-entrenamiento documentado y reproducible con `SEED=42` |
| 6 | **Conflictos de merge en GitHub** | Media | Bajo | Estrategia de branching por issue; PRs pequeños y atómicos |

---

## 4. Convenciones del Proyecto

- **Semilla global:** `SEED = 42` en todas las operaciones aleatorias
- **Rutas:** definidas en el diccionario `DIRS` al inicio del notebook
- **Commits:** formato `feat(SEIS-XX): descripcion` (Conventional Commits)
- **Branches:** `feature/SEIS-XX-descripcion-kebab-case`
- **Datos:** nunca se commitean al repo (`.gitignore`); solo se almacenan en Drive
