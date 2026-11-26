# 📊 Estructura Completa del Proyecto SAT-Mat

## 🎯 Visión General

Este documento proporciona una **guía detallada** de la estructura, contenido y flujo de trabajo del proyecto SAT-Mat (Sistema de Alerta Temprana de Rezago en Matemáticas).

---

## 📂 Árbol de Directorios

```
Anexo_I_SAT-Mat_Implementacion/
│
├─ 📄 README.md                             [Este archivo actualizado]
├─ 📄 ESTRUCTURA_PROYECTO.md                [Este documento]
├─ 📔 SAT-Mat_Implementacion.ipynb           [ARCHIVO PRINCIPAL - Notebook de 10 secciones]
│
├─ 📋 requirements.txt                      [Dependencias Python]
│
├─ 📁 data/                                 [ENTRADA - Crear localmente]
│  ├─ validation.csv                        [~1M registros - Conjunto de entrenamiento]
│  └─ test.csv                              [~1M registros - Conjunto de prueba]
│
├─ 📁 outputs/                              [SALIDA - Generado automáticamente]
│  │
│  ├─ 📌 NÓMINA Y REPORTES EJECUTIVOS
│  │  ├─ nomina_alertas.csv                 [★ Nómina de alertas para Power BI]
│  │  └─ resumen_semanal_alertas.csv        [Resumen agregado por curso]
│  │
│  ├─ 🤖 MODELOS Y CONFIGURACIÓN
│  │  ├─ modelo_satmat.joblib               [Modelo RL + umbrales (exportable)]
│  │  └─ diccionario_datos.csv              [Definición de variables]
│  │
│  ├─ 📊 EVALUACIÓN DEL MODELO
│  │  ├─ resultados_cv.csv                  [Métricas de cada fold (5-fold CV)]
│  │  ├─ evaluacion_test.csv                [Desempeño en test.csv (1M)]
│  │  └─ sensibilidad_variables.csv         [Impact de variables: con/sin]
│  │
│  ├─ ⚖️ EQUIDAD Y AUDITORIA
│  │  ├─ auditoria_equidad.csv              [Brecha FNR/FPR - Umbral Medio]
│  │  ├─ auditoria_equidad_alto.csv         [Brecha FNR/FPR - Umbral Alto]
│  │  └─ cumplimiento_requerimientos.csv    [Trazabilidad RF/RNF]
│  │
│  ├─ 🧪 VALIDACIÓN DE AUTOMATIZACIÓN
│  │  ├─ predicciones_perfiles.csv          [5 perfiles tipo predichos (Sección 7)]
│  │  ├─ pruebas_funcionales.csv            [Registro P1-P10 (Sección 8.1)]
│  │  └─ reentrenamiento_log.json           [Log de reentrenamientos]
│  │
│  ├─ 📄 RESULTS METADATA
│  │  └─ resultados.json                    [Resumen en JSON]
│  │
│  └─ [otros archivos generados]
│
├─ 📁 figs/                                 [VISUALIZACIONES - Generado automáticamente]
│  ├─ corr_variables.png
│  ├─ distribucion_rezago.png
│  ├─ roc_curves.png
│  ├─ confusion_matrices.png
│  ├─ shap_summary.png
│  └─ [otros gráficos del EDA y modelado]
│
└─ 📁 .git/                                 [Control de versiones Git]

```

---

## 🔄 Flujo de Ejecución del Notebook (10 Secciones CRISP-DM)

```
┌─────────────────────────────────────────────────────────────────┐
│          SAT-Mat_Implementacion.ipynb                           │
│          (Notebook Principal - 10 Secciones)                   │
└─────────────────────────────────────────────────────────────────┘

┌─ CELDA 0: DETECCIÓN DE ENTORNO (AUTO-LOAD)
│  ├─ Detecta: ¿Google Colab o Jupyter Local?
│  │
│  ├─ Si es Colab:
│  │  ├─ Instala SHAP
│  │  ├─ Autoriza acceso a Google Drive
│  │  ├─ Busca carpeta SAT-Mat/data en Drive
│  │  ├─ Descarga validation.csv y test.csv
│  │  └─ Define DATA_PATH = 'data' en /content/SAT-Mat
│  │
│  └─ Si es Local:
│     ├─ Verifica que carpeta 'data/' existe
│     ├─ Verifica que validation.csv y test.csv presentes
│     ├─ Si falta algo, da instrucciones y detiene
│     └─ Define DATA_PATH = 'data' (ruta local)
│
├─ CELDA 1: CARGA DE LIBRERÍAS Y CONFIGURACIÓN
│  └─ Importa pandas, numpy, sklearn, xgboost, shap, joblib
│
├─ CELDA 2: CARGA DE DATOS (USANDO DATA_PATH)
│  Entrada: data/validation.csv, data/test.csv (descargados en Celda 0)
│  └─ train_raw = pd.read_csv(f'{DATA_PATH}/validation.csv')
│     test_raw = pd.read_csv(f'{DATA_PATH}/test.csv')
│
┌─ SECCIÓN 1: COMPRENSIÓN DE DATOS (WBS 1.3.1-1.3.2)
│  Entrada: validation.csv, test.csv
│  ├─ Carga de datos y perfilamiento
│  ├─ Verificación de calidad (nulos, duplicados, rangos)
│  ├─ EDA: correlaciones, proxies, representatividad
│  ├─ Identificación de redundancias (Age vs. Grade)
│  └─ Salida: Hallazgos de calidad, descartes iniciales
│     └─→ outputs/diccionario_datos.csv (copia inicial)
│
├─ SECCIÓN 2: PREPARACIÓN DE DATOS (WBS 1.3.3-1.3.5)
│  Entrada: validation.csv (limpio)
│  ├─ Seudonimización SHA-256 con sal (SATMAT_SALT)
│  ├─ Separación de atributos protegidos → tabla auditoría
│  ├─ Poda de variables (redundantes, sensibles, proxies)
│  ├─ Normalización de predictores
│  └─ Salida: Conjuntos train/validation sin atributos protegidos
│     └─→ outputs/[ETL metadata]
│
├─ SECCIÓN 3: MODELADO (WBS 1.4.1-1.4.3)
│  Entrada: Datos preparados
│  ├─ Validación cruzada 5-fold (estratificada)
│  ├─ Entrena 3 modelos:
│  │  ├─ Regresión Logística (baseline, ★ SELECCIONADO)
│  │  ├─ Random Forest
│  │  └─ XGBoost
│  ├─ Comparación de AUC, Precision, Recall
│  └─ Salida: Modelo RL con coeficientes interpretables
│     └─→ outputs/resultados_cv.csv, modelo_satmat.joblib
│
├─ SECCIÓN 4: EVALUACIÓN (WBS 1.5.1-1.5.2)
│  Entrada: Modelo RL, test.csv
│  ├─ Desempeño en test set (1M registros)
│  ├─ Fijación de umbrales (Out-of-Fold, NO con test)
│  │  ├─ Umbral "Alto": precision ≥ 0.50
│  │  └─ Umbral "Medio": recall ≥ 0.80
│  ├─ Auditoria de equidad (brecha FNR ≤ 5 pp)
│  │  ├─ por género, etnia, SES, educación parental
│  │  └─ en ambos umbrales
│  └─ Salida: Umbrales de decisión, matriz de auditoría
│     └─→ outputs/evaluacion_test.csv, auditoria_equidad*.csv
│
├─ SECCIÓN 5: EXPLICABILIDAD SHAP (WBS 1.4.4)
│  Entrada: Modelo RL, test set
│  ├─ Cálculo de valores SHAP exactos (LinearExplainer)
│  ├─ Interpretación en log-odds de rezago
│  ├─ Top 3 factores por estudiante
│  └─ Salida: Fichas de explicabilidad (incluidas en nomina_alertas)
│     └─→ outputs/nomina_alertas.csv (columnas SHAP)
│
├─ SECCIÓN 6: DESPLIEGUE POR LOTES (WBS 1.6.2-1.6.3)
│  Entrada: test.csv (1M registros), modelo_satmat.joblib
│  ├─ Función de scoring batch (replica AWS Lambda)
│  ├─ ETL de inferencia (validación, contexto, predictores)
│  ├─ Scoring y generación de fichas
│  ├─ Medición de RNF-06: tiempo < 15 minutos
│  └─ Salida: Nómina de alertas para Power BI
│     └─→ outputs/nomina_alertas.csv ★ [ENTREGABLE PRINCIPAL]
│           └─→ [incluye: ID, riesgo, factores SHAP, scores]
│
├─ SECCIÓN 7: PREDICCIONES CON DATOS NUEVOS
│  Entrada: Modelo RL, perfil_tipo_1, ..., perfil_tipo_5
│  ├─ Generar 5 perfiles tipo (asistencia, estudio, etc.)
│  ├─ Scoring de cada perfil
│  ├─ Explicabilidad SHAP por perfil
│  └─ Salida: Tabla con predicciones e interpretación
│     └─→ outputs/predicciones_perfiles.csv
│
├─ SECCIÓN 8: AUTOMATIZACIÓN (WBS 1.6.3)
│  Entrada: Pipeline + modelo
│  ├─ Pruebas Funcionales P1-P10:
│  │  ├─ P1-P5: Validación de entrada y transformación
│  │  ├─ P6-P8: Scoring y derivación
│  │  ├─ P9: Detección de drift
│  │  └─ P10: Reentrenamiento (simulado)
│  ├─ Reentrenamiento automático (loop try/accept)
│  └─ Salida: Registros de pruebas, log de reentrenamiento
│     └─→ outputs/pruebas_funcionales.csv, reentrenamiento_log.json
│
├─ SECCIÓN 9: REGISTRO TÉCNICO (WBS 1.1.4)
│  ├─ Registro de cambios de control (CC-01, etc.)
│  ├─ Acciones correctivas (AC-01, AC-02, AC-03)
│  └─ Seguimiento de limitaciones y supuestos
│
└─ SECCIÓN 10: CUMPLIMIENTO DE REQUERIMIENTOS
   ├─ Trazabilidad de RF (1-7) → Secciones, Outputs
   ├─ Trazabilidad de RNF (1-6) → Métricas, Registros
   └─ Salida: Matriz de cumplimiento
      └─→ outputs/cumplimiento_requerimientos.csv
```

---

## 📊 Mapeo de Salidas (outputs/) → Usos

| Archivo | Generado | Destinatario | Propósito |
|---------|----------|--------------|----------|
| **nomina_alertas.csv** | Sección 6 | **Power BI** | ★ Nómina de riesgo (Bajo/Medio/Alto) + factores SHAP |
| **resumen_semanal_alertas.csv** | Sección 6 | Email automático | Resumen por curso para correo semanal |
| **modelo_satmat.joblib** | Sección 3 | Producción (Lambda) | Modelo RL + umbrales exportables |
| **evaluacion_test.csv** | Sección 4 | Reporte técnico | Métricas en test set (1M) |
| **auditoria_equidad.csv** | Sección 4 | Compliance | Brecha FNR/FPR por subgrupo (Umbral Medio) |
| **auditoria_equidad_alto.csv** | Sección 4 | Compliance | Brecha FNR/FPR por subgrupo (Umbral Alto) |
| **predicciones_perfiles.csv** | Sección 7 | Validación modelo | Perfiles tipo predichos |
| **pruebas_funcionales.csv** | Sección 8 | QA/Testing | Registro P1-P10 |
| **reentrenamiento_log.json** | Sección 8 | MLOps | Log de reentrenamientos (aceptado/rechazado) |
| **cumplimiento_requerimientos.csv** | Sección 10 | Auditoría | Trazabilidad RF/RNF |
| **diccionario_datos.csv** | Sección 1 | Documentación | Definición de variables |
| **resultados_cv.csv** | Sección 3 | Análisis | Métricas de cada fold (5-fold CV) |
| **sensibilidad_variables.csv** | Sección 3 | Análisis | Importancia de variables |
| **resultados.json** | Varias | Automatización | Resumen ejecutivo en JSON |

---

## 🔐 Privacidad y Seguridad

```
FLUJO DE DATOS SENSIBLES
├─ Entrada: validation.csv (contiene género, etnia, SES, etc.)
│
├─ [SECCIÓN 2: ETL]
│  ├─ ✓ Seudonimización SHA-256 → ID anónimo
│  ├─ ✓ Separación de atributos protegidos
│  │  └─→ outputs/[tabla_auditoria - segregada]
│  ├─ ✓ Poda de proxies (internet, apoyo parental)
│  └─ ✓ Validación automática (Definition of Done ética)
│
└─ Salida: nomina_alertas.csv
   ├─ ✓ Sin ID original (hash anónimo)
   ├─ ✓ Sin género, etnia, SES
   └─ ✓ Factores SHAP (asistencia, horas de estudio - modificables)
```

**Requisitos de Seguridad (RF-01, RF-07, RNF-02, RNF-03)**:
- Seudonimización obligatoria (sal via `SATMAT_SALT`)
- Tabla de auditoría segregada (acceso restringido)
- Exclusión de proxies socioeconómicos
- Validación automática de poda

---

## 🎯 Decisiones Clave

### 1. **Modelo Seleccionado**
   - **Regresión Logística** vs. Random Forest vs. XGBoost
   - **Criterio**: AUC ≈ 0.94 (igual), velocidad, interpretabilidad, SHAP exacta
   - **Justificación**: Relación casi lineal entre predictores y rezago (Sección 3.1)

### 2. **Estrategia de Alertas**
   - **Uno vs. Dos Umbrales** (AC-03)
   - **Cambio**: Dos umbrales (Medio/Alto) por imposibilidad de alcanzar recall≥0.80 y precisión≥0.50 con un solo umbral (prevalencia 6.7%)
   - **Resultado**:
     - Umbral **Alto** (precisión≥0.50): derivación prioritaria
     - Umbral **Medio** (recall≥0.80): seguimiento preventivo

### 3. **Conjunto de Validación Temporal**
   - **Plan vs. Realidad** (CC-01, AC-02)
   - **Cambio**: test.csv como validación temporal (plan esperaba dos cohortes)
   - **Garantía**: Generalización a datos nunca vistos

### 4. **Exclusión de Proxies**
   - **InternetAccess, ParentSupport descartados** (AC-01)
   - **Justificación**: proxies socioeconómicos, intrusivas, impiden explicabilidad (N3)
   - **Resultado**: Factores modificables (asistencia, estudio)

---

## 🧮 Métricas Clave

| Métrica | Valor | Fuente |
|---------|-------|--------|
| Volumen entrenamiento | ~1M (validation.csv) | Sección 1 |
| Volumen prueba | ~1M (test.csv) | Sección 1 |
| Variables entrada | 20 predictores + 1 objetivo | Sección 1 |
| Variables descartadas | 2 (Age, InternetAccess, ParentSupport) | Sección 2 |
| AUC modelo RL | 0.94 | Sección 3 |
| Folds validación cruzada | 5 (estratificados) | Sección 3 |
| Prevalencia rezago | 6.7% | Sección 1 |
| Umbral Alto (precisión) | ≥0.50 | Sección 4 |
| Umbral Medio (recall) | ≥0.80 | Sección 4 |
| Brecha FNR máxima (Equidad) | ≤5 pp | Sección 4 |
| Tiempo lote (1M registros) | <15 min | Sección 6 (RNF-06) |
| Factores SHAP por alumno | Hasta 3 top | Sección 5 |

---

## ⚠️ Limitaciones y Supuestos

| Limitación | Impacto | Recomendación |
|-----------|--------|----------------|
| **Datos sintéticos** | Rezago plano entre subgrupos (6.7%) | Re-auditar equidad con datos reales |
| **Sin ID estudiante** | Hash posicional (vulnerable) | En producción, usar StudentID |
| **Sin auditoria interseccional** | Sesgo no detectado en combinaciones | Implementar para piloto real |
| **Datos de 12.º grado duplicados** | El doble de alertas en ese curso | Normalizar en resumen agregado |
| **Validez externa limitada** | Correlaciones socioeconómicas no reflejan realidad | Reentrenar con datos del establecimiento |

---

## 🚀 Cómo Usar Este Proyecto

### Fase 1: Setup Inicial
```bash
# Clonar repositorio
git clone [url] Anexo_I_SAT-Mat_Implementacion
cd Anexo_I_SAT-Mat_Implementacion

# Instalar dependencias
pip install -r requirements.txt

# Preparar datos (crear data/ y copiar CSV)
mkdir data
cp /ruta/validation.csv data/
cp /ruta/test.csv data/
```

### Fase 2: Ejecutar Notebook
```bash
# Opción 1: Google Colab (recomendado)
# - Subir notebook a Colab
# - Ejecutar › Ejecutar todas

# Opción 2: Jupyter local
jupyter notebook SAT-Mat_Implementacion.ipynb
# Kernel › Reiniciar & Ejecutar Todo
```

### Fase 3: Validar Outputs
```bash
# Verificar archivos generados
ls -la outputs/
ls -la figs/

# Archivo principal para Power BI
cat outputs/nomina_alertas.csv | head -20

# Auditoria de equidad
cat outputs/auditoria_equidad.csv

# Cumplimiento de requerimientos
cat outputs/cumplimiento_requerimientos.csv
```

### Fase 4: Despliegue en Producción
```bash
# Cargar modelo en AWS Lambda / microservicio
import joblib
modelo = joblib.load('outputs/modelo_satmat.joblib')
prediccion = modelo.predict_proba(X_new)
```

---

## 📚 Referencias Técnicas

- **CRISP-DM**: https://en.wikipedia.org/wiki/Cross-industry_standard_process_for_data_mining
- **SHAP**: Lundberg & Lee (2017). *A Unified Approach to Interpreting Model Predictions*
- **Equidad**: Hardt et al. (2016). *Equality of Opportunity in Supervised Learning*
- **Regresión Logística**: Hosmer et al. (2013). *Applied Logistic Regression* (3ª ed.)

---

## 👥 Equipo y Contacto

**Autores**: Arelis Tovar, Dayana Vega, Encar Portillo  
**Curso**: SCY1102 Gestión de Proyectos de Datos  
**Institución**: Grupo 6

Para preguntas, abrir un issue o contactar directamente.

---

**Última actualización**: 2024  
**Versión**: 1.0 (Implementación Completa)
