# SAT-Mat · Implementación (Anexo I) — Sistema de Alerta Temprana de Rezago en Matemáticas

**Grupo 6, SCY1102** — Gestión de Proyectos de Datos  
*Autores: Arelis Tovar, Dayana Vega, Encar Portillo*

---

## 📋 Descripción General

**SAT-Mat** es un **Sistema de Alerta Temprana (Early Warning System, EWS)** basado en Machine Learning para detectar estudiantes en riesgo de rezago en matemáticas. El sistema implementa un pipeline completo de ciencia de datos siguiendo la metodología **CRISP-DM** (Cross-Industry Standard Process for Data Mining), desde la comprensión de datos hasta el despliegue en producción.

### Propósito
Identificar estudiantes de educación media que presentan riesgo de bajo desempeño en matemáticas para habilitar intervenciones tempranas (tutorías, seguimiento docente) basadas en factores modificables (asistencia, horas de estudio).

### Alcance Técnico
- **Modelos evaluados**: Regresión Logística, Random Forest, XGBoost
- **Modelo seleccionado**: Regresión Logística (AUC ≈ 0.94, interpretable, explicable con SHAP)
- **Volumen de datos**: ~1 millón de registros por archivo (validation.csv, test.csv)
- **Variables**: 20 predictores + 1 objetivo (rezago)
- **Estrategia de alertas**: Dos umbrales (Medio y Alto) según precision/recall
- **Equidad**: Auditoria de disparidad de tasas FNR/FPR por subgrupo demográfico

---

## 🏗️ Estructura del Proyecto

```
Anexo_I_SAT-Mat_Implementacion/
├── SAT-Mat_Implementacion.ipynb    # Notebook principal (10 secciones)
├── requirements.txt                 # Dependencias Python
├── README.md                        # Este archivo
├── data/                           # (crear localmente)
│   ├── validation.csv              # Conjunto de entrenamiento (~1M)
│   └── test.csv                    # Conjunto de prueba (~1M)
├── outputs/                        # Resultados generados
│   ├── nomina_alertas.csv          # Nómina de alertas para Power BI
│   ├── resumen_semanal_alertas.csv # Resumen por curso
│   ├── modelo_satmat.joblib        # Modelo y umbrales serializados
│   ├── diccionario_datos.csv       # Definición de columnas
│   ├── resultados_cv.csv           # Resultados validación cruzada
│   ├── sensibilidad_variables.csv  # Análisis de sensibilidad
│   ├── evaluacion_test.csv         # Métricas en conjunto de prueba
│   ├── auditoria_equidad.csv       # Auditoria (umbral Medio)
│   ├── auditoria_equidad_alto.csv  # Auditoria (umbral Alto)
│   ├── cumplimiento_requerimientos.csv # Trazabilidad de requisitos
│   ├── predicciones_perfiles.csv   # 5 predicciones tipo
│   ├── pruebas_funcionales.csv     # Registro P1-P10
│   ├── reentrenamiento_log.json    # Log de reentrenamientos
│   └── resultados.json             # Resultados en formato JSON
├── figs/                           # Visualizaciones del informe
│   ├── corr_variables.png
│   ├── distribucion_rezago.png
│   ├── roc_curves.png
│   ├── confusion_matrices.png
│   ├── shap_summary.png
│   └── [otros gráficos]
└── .git/                           # Control de versiones
```

---

## 🔄 Fases CRISP-DM Implementadas

| Sección | Fase CRISP-DM | Paquetes WBS | Entregable | Descripción |
|---------|---|---|---|---|
| 1 | **Comprensión de datos** | 1.3.1, 1.3.2 | E3 | Carga, perfilamiento, verificación de supuestos, EDA |
| 2 | **Preparación de datos** | 1.3.3–1.3.5 | E4, H2 | ETL, seudonimización, poda de variables, tabla de auditoría |
| 3 | **Modelado** | 1.4.1–1.4.3 | E5 | Validación cruzada (5-fold), entrenamiento de 3 modelos |
| 4 | **Evaluación** | 1.5.1, 1.5.2, 1.5.4 | E6, H3 | Desempeño en test, equidad, umbrales de decisión |
| 5 | **Explicabilidad** | 1.4.4 | E6 | SHAP values, factores relevantes por alumno |
| 6 | **Despliegue por lotes** | 1.6.2, 1.6.3 | E7 | Scoring batch, nómina para Power BI, medición RNF-06 |
| 7 | **Predicciones nuevas** | — | E7 | Inferencia con datos nuevos, perfiles tipo |
| 8 | **Automatización** | 1.6.3 | E7 | Alerta, detección de drift, reentrenamiento automático |
| 9 | **Registro técnico** | 1.1.4 | — | Control de cambios (CC-01, etc.), trazabilidad |
| 10 | **Cumplimiento** | — | — | Auditoria de requisitos funcionales y no-funcionales |

---

## 📊 Qué se Analiza

### 1. **Comprensión de Datos (Sección 1)**
- **Calidad**: ausencia de nulos, duplicados, rangos válidos
- **Supuestos vs. Realidad**:
  - Volumen real: ~1M registros (10x mayor que plan)
  - Sin ID de estudiante → identificador sustituto + hash SHA-256
  - Sin variable de cohorte → validación temporal con test.csv independiente
- **Correlaciones**: asistencia (r=0.82) y horas de estudio (r=0.46) como predictores principales
- **Proxies socioeconómicos**: internet y apoyo parental descartados (RNF-03)
- **Representatividad**: tasa de rezago plana por género, etnia, SES (~6.7%)

### 2. **Preparación de Datos (Sección 2)**
- **Seudonimización**: SHA-256 con sal (`SATMAT_SALT`)
- **Separación de atributos protegidos**: tabla de auditoría aparte (RF-07)
- **Poda de variables**: se descartan redundantes (Age), sensibles, con riesgo de fuga
- **Definition of Done ética**: validación automática de exclusiones

### 3. **Modelado (Sección 3)**
- **Validación cruzada**: 5 pliegues estratificados por clase y curso
- **Modelos comparados**:
  - Regresión Logística (baseline, **seleccionado**)
  - Random Forest
  - XGBoost
- **Resultado**: AUC ≈ 0.94 en todos; RL elegida por velocidad, interpretabilidad, coeficientes exactos y explicabilidad SHAP exacta
- **Análisis de sensibilidad**: agregar predictores con riesgo de fuga solo sube precisión 1-2pp → conjunto A mantiene explicabilidad (N3)

### 4. **Evaluación (Sección 4)**
- **Umbrales duales** (porque prevalencia 6.7% impide alcanzar recall≥0.80 y precisión≥0.50 con 1 umbral):
  - **Alto**: precisión ≥ 0.50 → derivación prioritaria a tutoría
  - **Medio**: recall ≥ 0.80 → seguimiento preventivo del docente
  - **Bajo**: resto
- **Auditoria de equidad**:
  - Criterio: brecha FNR ≤ 5 puntos porcentuales (igualdad de oportunidades)
  - Se evalúan ambos umbrales por subgrupo (género, etnia, SES, educación parental)
  - Nota: no hay auditoria interseccional (mejora para piloto real)

### 5. **Explicabilidad SHAP (Sección 5)**
- Valores SHAP exactos para RL (`LinearExplainer`)
- Interpretación en log-odds de rezago
- Factores relevantes individuales (hasta 3 por estudiante)

### 6. **Despliegue por Lotes (Sección 6)**
- Función de scoring que replica AWS Lambda
- Validación de entrada, ETL de inferencia, scoring, generación de fichas
- Nómina de alertas para Power BI
- **RNF-06**: tiempo < 15 minutos para 1M registros (medido)

### 7. **Predicciones con Datos Nuevos (Sección 7)**
- 5 perfiles tipo (p. ej., asistencia perfecta, 10 h/semana estudio, etc.)
- Predicción y explicabilidad para cada perfil

### 8. **Automatización (Sección 8)**
- **P1–P10**: pruebas funcionales de la tubería automática
- **Reentrenamiento**: log de intentos (promovido vs. rechazado)
- **Detección de drift**: verificación de distribución de predictores

---

## 🔧 Dependencias

```
pandas >= 2.2        # Manipulación de datos
numpy                # Álgebra lineal
scipy                # Funciones científicas
scikit-learn >= 1.5  # Modelos ML (Regresión Logística, RF, XGBoost)
xgboost >= 2.0       # Gradient boosting
shap >= 0.45         # Explicabilidad
matplotlib           # Visualización
joblib               # Serialización de modelos
```

**Instalación**:
```bash
pip install -r requirements.txt
```

---

## 🚀 Cómo Ejecutar

### 📌 Bloque de Carga Automático (Colab + Local)

La **primera celda del notebook** detecta automáticamente el entorno:
- **En Google Colab**: Busca `SAT-Mat/data` en Drive y descarga CSV automáticamente
- **En Jupyter Local**: Verifica que `data/` existe junto al notebook

```python
# Primera celda del notebook:
# - Importa google.colab (si existe, es Colab; si no, es Local)
# - En Colab: Autoriza Drive → busca SAT-Mat/data → descarga CSV
# - En Local: Verifica carpeta data/ → confirma CSV presentes
# - Define variable DATA_PATH para resto del notebook
```

---

### Opción 1: Google Colab (recomendado)

**Paso 1: Preparar Google Drive**
```
📁 Mi unidad (o Compartido conmigo)
   └─ 📁 SAT-Mat/
      └─ 📁 data/
         ├─ validation.csv
         └─ test.csv
```

**Paso 2: Subir y ejecutar**
1. En Google Colab: *Archivo › Subir notebook › SAT-Mat_Implementacion.ipynb*
2. Ejecutar: *Entorno de ejecución › Ejecutar todas*
   - Primera celda detecta Colab automáticamente
   - Pide autorizar acceso a Drive
   - Busca y descarga CSV
   - Resto del notebook ejecuta con normalidad
3. **Importante**: Descargar `outputs/` y `figs/` antes de cerrar (se borran)

**Tiempo**: ~8 minutos | **Ventaja**: Sin instalar nada, RAM ilimitada

---

### Opción 2: Jupyter Local

**Paso 1: Preparar carpeta data/**
```bash
# La carpeta ya existe. Copiar archivos:
cp /ruta/validation.csv Anexo_I_SAT-Mat_Implementacion/data/
cp /ruta/test.csv Anexo_I_SAT-Mat_Implementacion/data/

# Estructura esperada:
# Anexo_I_SAT-Mat_Implementacion/
# ├── SAT-Mat_Implementacion.ipynb
# ├── data/                        ← CREAR/CONFIRMAR
# │  ├── validation.csv            ← COPIAR AQUÍ
# │  └── test.csv                  ← COPIAR AQUÍ
# ├── outputs/
# ├── figs/
# └── requirements.txt
```

**Paso 2: Instalar y ejecutar**
```bash
# 1. Instalar dependencias
pip install -r requirements.txt

# 2. Ejecutar notebook
jupyter notebook SAT-Mat_Implementacion.ipynb

# 3. Primera celda verificará:
#    - Carpeta data/ existe
#    - Archivos CSV presentes
#    - Si falta algo, da instrucciones y detiene
#
# 4. Luego: Kernel › Reiniciar & Ejecutar Todo
```

**Tiempo**: ~8 minutos | **Ventaja**: Control local, sin conexión internet

---

### 🔧 Troubleshooting de Carga

| Problema | Solución |
|----------|----------|
| **Colab**: "No encontré SAT-Mat/data" | Crear en Drive: SAT-Mat › data, copiar CSV ahí |
| **Colab**: "Falta validation.csv" | Verificar CSV en SAT-Mat/data en Drive |
| **Colab**: "AssertionError" al descargar | Verificar que Drive se autorizó (hizo click en link) |
| **Local**: "Carpeta 'data' no encontrada" | Crear carpeta `data/` en directorio del notebook |
| **Local**: "Falta validation.csv" | Copiar CSV a carpeta `data/` |
| **Local**: "ImportError: shap" | `pip install -r requirements.txt` |
| **Local**: SystemExit(1) | Verificar que data/ y CSV están presentes |

---

## 📁 Archivos de Salida (outputs/)

### Nómina y Reportes
- **`nomina_alertas.csv`**: Nómina priorizada con nivel de riesgo (Bajo/Medio/Alto), hasta 3 factores SHAP top y columnas de validación de orientación. *Insumo directo para Power BI.*
- **`resumen_semanal_alertas.csv`**: Resumen agregado por curso para correo semanal

### Modelos y Configuración
- **`modelo_satmat.joblib`**: Modelo RL entrenado + umbrales de decisión (exportable a producción)
- **`diccionario_datos.csv`**: Definición de tipos, rangos y transformaciones de cada variable

### Evaluación del Modelo
- **`resultados_cv.csv`**: Métricas de cada fold de validación cruzada (precision, recall, AUC, etc.)
- **`evaluacion_test.csv`**: Desempeño en conjunto de prueba (1M registros)
- **`sensibilidad_variables.csv`**: Impact de cada variable en precisión/recall (con/sin variable)
- **`resultados.json`**: Resultados resumidos en JSON (automatizable)

### Equidad y Auditoria
- **`auditoria_equidad.csv`**: Brecha FNR/FPR por subgrupo en umbral Medio
- **`auditoria_equidad_alto.csv`**: Brecha FNR/FPR por subgrupo en umbral Alto
- **`cumplimiento_requerimientos.csv`**: Trazabilidad de requisitos (RF-01 a RF-07, RNF-01 a RNF-06)

### Validación de Automatización
- **`predicciones_perfiles.csv`**: 5 predicciones sobre perfiles tipo (asistencia, estudio, etc.)
- **`pruebas_funcionales.csv`**: Registro de pruebas P1–P10 (entrada, salida esperada, resultado)
- **`reentrenamiento_log.json`**: Log de intentos de reentrenamiento (promovido/rechazado)

### Visualizaciones (figs/)
- Correlaciones y EDA
- Curvas ROC
- Matrices de confusión
- SHAP summary plots
- Distribuciones de predicciones y rezago

---

## ⚙️ Variables de Entorno

- **`SATMAT_SALT`** (opcional): sal para seudonimización SHA-256. Valor por defecto (`demo_salt_123`) es solo para demostración. En producción, usar valor secreto único.

```bash
export SATMAT_SALT="tu_sal_secreta"
```

---

## 🎯 Decisiones Clave y Cambios de Control

### Supuestos del Plan vs. Datos Reales
| Supuesto (Plan) | Realidad | Cambio | Impacto |
|---|---|---|---|
| 107,350 registros | ~1M por archivo | CC-01 | RNF-06 (lote < 15 min) requiere medición |
| Incluir train.csv | Sobrepasa límite de RAM | CC-01 | Se entrena con validation.csv; test.csv reservado |
| ID de estudiante | No existe | — | Hash posicional; en producción usar StudentID |
| Dos cohortes (años) | Sin variable de cohorte | AC-02 | Validación temporal reemplazada por test.csv independiente |

### Acciones Correctivas
- **AC-03**: Umbrales duales (Medio/Alto) en lugar de uno solo → soluciona trade-off precision/recall
- **AC-01**: Exclusión de proxies socioeconómicos → protege equidad (C4, N3)
- **AC-02**: Test.csv como validación temporal → garantiza generalización

---

## 🔐 Privacidad y Seguridad

- **Seudonimización SHA-256**: identificador sustituto se hashea de forma irreversible
- **Separación de atributos protegidos**: género, etnia, SES, educación parental en tabla aparte
- **Poda de variables**: se descartan internet, apoyo parental (proxies y potencialmente intrusivas)
- **Auditoria de equidad**: brecha FNR ≤ 5 pp entre subgrupos

---

## ⚠️ Limitaciones Declaradas

1. **Datos sintéticos**: rezago es uniforme (6.7%) entre subgrupos → no refleja correlaciones socioeconómicas reales. **En producción, re-auditar equidad con datos del establecimiento.**

2. **Identificador sustituto**: sin ID de estudiante, el hash se genera sobre posición de fila. En producción, `hash_id()` debe usar `StudentID` (RUN o ID institucional) si está disponible.

3. **Sin auditoria interseccional**: se evalúan subgrupos independientes (género, etnia, SES) pero no combinaciones (p. ej., mujer + bajo SES). Mejora recomendada para piloto real.

4. **Duplicación de 12.º grado**: este curso agrupa edades 17 y 18, resultando en el doble de registros y alertas en resumen por curso. Requiere normalización si se usa para decisiones agregadas.

5. **Auditoria de edad**: `Age` se descarta por redundancia con `Grade` (determinación casi perfecta). Validar que grados correspondan a rangos de edad esperados.

---

## 📖 Referencias

- Hosmer, D. W., Lemeshow, S., & Sturdivant, R. X. (2013). *Applied Logistic Regression* (3rd ed.). Wiley.
- Hardt, M., Price, E., & Srebro, N. (2016). Equality of opportunity in supervised learning. *NIPS*.
- SHAP documentation: https://shap.readthedocs.io/

---

## 📞 Contacto

Para preguntas técnicas o ajustes del modelo, contactar al equipo (Arelis Tovar, Dayana Vega, Encar Portillo).

---

**Última actualización**: 2024  
**Estado**: Implementación completa (Anexo I, SCY1102 Grupo 6)
