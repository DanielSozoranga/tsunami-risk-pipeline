# Análisis de Vulnerabilidades — Infraestructura Google Colab

**Issue:** SEIS-24 — Vulnerabilidades en infraestructura Google Colab del proyecto  
**Asignado:** Daniel Sozoranga  
**Sprint:** Sprint 4 — Dashboard and Security

---

## 1. Introducción

Google Colab es el entorno de ejecución del pipeline SeismicPipeline. Al ser una plataforma en la nube compartida y con sesiones temporales, presenta vectores de ataque específicos que deben identificarse, clasificarse y mitigarse para garantizar la seguridad del proyecto.

---

## 2. Vectores de ataque identificados

### 2.1 Exposición de tokens y credenciales en el notebook (Alto)

| Campo | Detalle |
|---|---|
| **Severidad** | Alta |
| **Descripción** | Si las credenciales de Google Drive o APIs se escriben directamente en celdas del notebook, quedan expuestas en el historial de versiones de GitHub y en el output guardado. |
| **Evidencia** | Cualquier `print()` de variables de entorno o hardcodeo de rutas con tokens en el código fuente. |
| **Impacto CIA** | Confidencialidad: comprometida. Integridad: atacante puede modificar datos en Drive. Disponibilidad: no afectada directamente. |
| **Mitigación** | Usar variables de entorno con `os.environ.get()`. Nunca commitear notebooks con outputs que contengan credenciales. Agregar `*.ipynb` outputs al `.gitignore`. |

### 2.2 Session hijacking via token de autenticación de Colab (Alto)

| Campo | Detalle |
|---|---|
| **Severidad** | Alta |
| **Descripción** | Google Colab genera un token de autenticación por sesión accesible desde el runtime. Un atacante con acceso físico o remoto al entorno puede extraer este token y suplantar la identidad del usuario. |
| **Evidencia** | El token está disponible en `~/.config/gcloud/` durante la sesión activa. Comando: `!cat ~/.config/gcloud/application_default_credentials.json` |
| **Impacto CIA** | Confidencialidad: acceso a Drive y datos del proyecto. Integridad: posibilidad de modificar archivos en Drive. Disponibilidad: posible eliminación de datasets. |
| **Mitigación** | Revocar tokens al finalizar cada sesión. No compartir notebooks en ejecución. Usar cuentas de servicio con permisos mínimos. |

### 2.3 Acceso no autorizado a Google Drive montado (Medio)

| Campo | Detalle |
|---|---|
| **Severidad** | Media |
| **Descripción** | Al montar Drive con `drive.mount('/content/drive')`, todos los archivos del Drive del usuario son accesibles desde el runtime de Colab. Un notebook malicioso compartido puede leer, modificar o exfiltrar archivos sin que el usuario lo note. |
| **Evidencia** | `!ls /content/drive/MyDrive/` lista todos los archivos del Drive montado. |
| **Impacto CIA** | Confidencialidad: datos sísmicos y modelo XGBoost expuestos. Integridad: dataset podría ser modificado. Disponibilidad: archivos podrían ser eliminados. |
| **Mitigación** | Solo montar Drive cuando sea necesario. No abrir notebooks de fuentes no confiables mientras Drive está montado. Usar carpetas compartidas específicas con permisos limitados en lugar de montar el Drive completo. |

### 2.4 Ejecución de código arbitrario via celdas ocultas (Medio)

| Campo | Detalle |
|---|---|
| **Severidad** | Media |
| **Descripción** | Los notebooks `.ipynb` son archivos JSON que pueden contener celdas ocultas o código ofuscado que se ejecuta sin que el usuario lo vea en la interfaz. Un notebook compartido malicioso puede exfiltrar datos o instalar backdoors. |
| **Evidencia** | Inspección del JSON del notebook: celdas con `"metadata": {"cellView": "form"}` o con código base64 en strings. |
| **Impacto CIA** | Confidencialidad: exfiltración de datos. Integridad: modificación del entorno de ejecución. Disponibilidad: corrupción del runtime. |
| **Mitigación** | Revisar el JSON del notebook antes de ejecutar. Usar `nbformat` para inspeccionar programáticamente. Ejecutar notebooks de terceros en cuentas sandbox aisladas. |

### 2.5 Persistencia de datos en caché del runtime (Bajo)

| Campo | Detalle |
|---|---|
| **Severidad** | Baja |
| **Descripción** | Variables en memoria, archivos temporales y outputs de celdas pueden persistir durante la sesión activa. Si el runtime es compartido (poco probable en Colab gratuito pero posible en entornos enterprise), otro usuario podría acceder. |
| **Impacto CIA** | Confidencialidad: mínimo en Colab estándar. |
| **Mitigación** | Limpiar outputs antes de compartir. Usar `Runtime > Factory reset runtime` al finalizar. |

---

## 3. Tabla de clasificación de riesgos

| Vulnerabilidad | Probabilidad | Impacto | Nivel | CIA afectado |
|---|---|---|---|---|
| Exposición de tokens en notebook | Media | Alto | **ALTO** | C, I |
| Session hijacking | Baja | Alto | **ALTO** | C, I, D |
| Acceso no autorizado a Drive montado | Media | Medio | **MEDIO** | C, I, D |
| Ejecución de código en celdas ocultas | Baja | Medio | **MEDIO** | C, I, D |
| Persistencia en caché del runtime | Baja | Bajo | **BAJO** | C |

---

## 4. Controles implementados en SeismicPipeline

```python
# Control 1: Sin hardcodeo de credenciales
BASE = os.path.abspath('./SeismicPipeline')  # Ruta relativa, sin tokens

# Control 2: Carga condicional de Drive
try:
    from google.colab import drive
    drive.mount('/content/drive')
except ModuleNotFoundError:
    pass  # Entorno local sin Drive

# Control 3: Ninguna celda con outputs de credenciales
# Todos los prints muestran solo rutas y conteos, nunca tokens
```

---

## 5. Conclusión

La infraestructura Google Colab presenta dos vulnerabilidades de nivel **Alto** relacionadas con la gestión de credenciales y tokens de sesión. Los controles implementados en SeismicPipeline mitigan los riesgos más críticos: no se hardcodean credenciales, no se imprimen tokens en outputs y el montaje de Drive se realiza únicamente cuando es necesario. Las vulnerabilidades residuales requieren controles de proceso (revisar notebooks antes de ejecutar, revocar tokens al finalizar) que están documentados en la política de gobernanza del proyecto.
