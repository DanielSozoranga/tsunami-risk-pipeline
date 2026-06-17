# Planificado vs Real + Retrospectiva — SeismicPipeline

**Issue:** SEIS-28 — Documentación final: ejecución real vs planificación y lecciones aprendidas  
**Asignado:** Daniel Sozoranga (Scrum Master)  
**Sprint:** Sprint 4 — Dashboard and Security

---

## 1. Resumen ejecutivo del proyecto

SeismicPipeline se ejecutó en 4 sprints semanales (20 mayo – 16 junio 2026), bajo metodología Scrum con un equipo de 2 personas desempeñando los 3 roles del marco (Scrum Master, Product Owner, Development Team). El proyecto entregó un pipeline completo de predicción de riesgo tsunamigénico con XGBoost, desde la ingesta de datos hasta un dashboard interactivo y documentación de ciberseguridad.

---

## 2. Planificado vs Real por Sprint

### Sprint 1 — Setup e Ingesta (20-26 May)

| Métrica | Planificado | Real |
|---|---|---|
| Story Points | 21 | 21 |
| Issues | 5 | 5 |
| Estado | — | ✅ 100% completado |

**Nota:** Aunque la fecha original del sprint ya había pasado al momento de iniciar los commits (12 Jun), todo el trabajo técnico ya estaba desarrollado en el notebook. La ejecución real de Git/Jira se concentró en una sola sesión intensiva el 12-13 de junio, replicando el orden lógico de las stories.

### Sprint 2 — ETL y Benchmarking (27 May - 2 Jun)

| Métrica | Planificado | Real |
|---|---|---|
| Story Points | 28 | 28 |
| Issues | 6 | 6 |
| Estado | — | ✅ 100% completado |

### Sprint 3 — Modelado XGBoost (3-9 Jun)

| Métrica | Planificado | Real |
|---|---|---|
| Story Points | 35 | 35 |
| Issues | 6 | 6 |
| Estado | — | ✅ 100% completado |

### Sprint 4 — Dashboard and Security (10-16 Jun)

| Métrica | Planificado | Real |
|---|---|---|
| Story Points | 50 | 50 |
| Issues | 10 | 10 |
| Estado | — | ✅ 100% completado |

---

## 3. Desviaciones identificadas

| Desviación | Causa | Impacto | Resolución |
|---|---|---|---|
| Automatización Jira no se disparó en la primera branch creada | Smart Commits no configurado en el sistema | Subtasks no transicionaban a Done | Implementada Automation Rule nativa de Jira: Story Done → Subtasks Done |
| Conflictos de merge en `notebooks/SeismicPipeline_Integrador_v3.ipynb` | Múltiples issues (SEIS-9 y SEIS-13) modificaron el mismo archivo en paralelo | Bloqueo temporal de 2 PRs | Resueltos manualmente vía editor web de GitHub, conservando el contenido más reciente de cada feature |
| Orden de issues de Sprint 4 no coincidía con la secuencia numérica de keys | El orden real en Jira está definido por fecha de vencimiento, no por número de key | Replanificación del orden de ejecución de commits | Verificado directamente en el backlog de Jira antes de continuar |

---

## 4. Retrospectiva del equipo

### 4.1 Qué funcionó bien

- La automatización GitHub→Jira (branch→In Progress, PR→In Review, merge→Done) funcionó de forma consistente una vez configurada correctamente, eliminando actualizaciones manuales del tablero.
- La convención de nombres de branch (`feature/SEIS-XX-descripcion-kebab-case`) hizo que la trazabilidad entre código y Jira fuera inmediata.
- Tener el notebook técnico completo desde el inicio permitió que los commits incrementales fueran extracciones organizadas del trabajo ya validado, reduciendo el riesgo de introducir bugs nuevos en cada PR.

### 4.2 Qué se puede mejorar

- Configurar la Automation Rule de Jira **antes** del primer commit, no reactivamente después de detectar que las subtareas no transicionaban.
- Coordinar con anticipación qué issues paralelos tocan el mismo archivo (`notebooks/...ipynb`) para reducir conflictos de merge.
- Verificar el orden real de un sprint en el backlog de Jira antes de planificar la secuencia de trabajo, en lugar de asumir orden numérico de keys.

### 4.3 Lecciones aprendidas

1. La configuración de automatizaciones de Jira debe validarse con una prueba end-to-end (crear branch → PR → merge) **antes** de iniciar el trabajo real del sprint, no durante.
2. En equipos de 2 personas desempeñando 3 roles Scrum, la disciplina de PRs pequeños y atómicos por subtarea es lo que mantiene la trazabilidad, incluso sin la presión de coordinación que tendría un equipo más grande.
3. Mantener el código fuente de verdad (notebook completo) desde etapas tempranas permite que la "implementación incremental por Git" sea en realidad documentación retroactiva de trabajo ya validado, lo cual reduce el riesgo técnico del proyecto a cambio de un mayor esfuerzo de organización al inicio.

---

## 5. Cierre del proyecto

Los 4 sprints se completaron al 100% en story points e issues planificados. El pipeline SeismicPipeline cumple su objetivo académico: un sistema reproducible de clasificación binaria de riesgo tsunamigénico, validado con eventos históricos ecuatorianos e internacionales, documentado en ciberseguridad bajo el marco CIA, y entregado con trazabilidad completa Git-Jira para el curso de Gestión de Proyectos de SI.
