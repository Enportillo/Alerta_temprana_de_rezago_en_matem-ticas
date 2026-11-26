# 🚀 Quick Start - SAT-Mat

**Guía rápida para ejecutar el Sistema de Alerta Temprana de Rezago en Matemáticas**

---

## 1️⃣ Requisitos Previos

✅ Python 3.8+ (para Jupyter Local)  
✅ `validation.csv` y `test.csv` disponibles  
✅ Carpeta `data/` ya existe en el proyecto

---

## 2️⃣ Setup (Depende del Entorno)

### El notebook DETECTA AUTOMÁTICAMENTE dónde corre

La **primera celda**:
```
┌─────────────────────────────────────────┐
│ ¿Estamos en Google Colab?               │
├─────────────────────────────────────────┤
│ SÍ → Busca SAT-Mat/data en Drive         │
│      Descarga validation.csv y test.csv │
│                                         │
│ NO → Verifica carpeta data/ local       │
│      Confirma CSV presentes             │
└─────────────────────────────────────────┘
```

---

### Opción A: Google Colab (5 minutos)

**1. Preparar Google Drive**
```
📁 SAT-Mat/
   └─ 📁 data/
      ├─ validation.csv
      └─ test.csv
```

**2. Ir a Google Colab**
- Link: https://colab.research.google.com/

**3. Subir notebook**
- Archivo › Subir notebook
- Selecciona: `SAT-Mat_Implementacion.ipynb`

**4. Ejecutar**
- Entorno de ejecución › Ejecutar todas
- Primera celda se encarga de autenticar y descargar datos
- ¡Listo!

✨ **Ventaja**: Sin instalar nada, RAM automática, muy rápido

---

### Opción B: Jupyter Local (5 minutos)

**1. Copiar archivos CSV**
```bash
# Estructura esperada:
# .../Anexo_I_SAT-Mat_Implementacion/
# ├── SAT-Mat_Implementacion.ipynb
# ├── data/
# │  ├── validation.csv    ← AQUÍ
# │  └── test.csv          ← AQUÍ
# └── requirements.txt

# Comando:
cp /ruta/validation.csv ./data/
cp /ruta/test.csv ./data/
```

**2. Instalar dependencias**
```bash
pip install -r requirements.txt
```

**3. Ejecutar notebook**
```bash
jupyter notebook SAT-Mat_Implementacion.ipynb
# Primera celda verifica carpeta data/
# Kernel › Reiniciar & Ejecutar Todo
```

✨ **Ventaja**: Control local, sin conexión internet, archivos permanentes

---

## 3️⃣ Qué Esperar

```
✓ Sección 1: Carga y perfilamiento (1 min)
✓ Sección 2: Preparación de datos (1 min)
✓ Sección 3: Entrenamiento de modelos (4 min) ⏳ [Más lento]
✓ Sección 4: Evaluación y equidad (1 min)
✓ Sección 5: Explicabilidad SHAP (0.5 min)
✓ Sección 6: Scoring batch (0.5 min)
✓ Secciones 7-10: Validación y reportes (1 min)

TOTAL: ~8 minutos (con Random Forest es más lento)
```

---

## 4️⃣ Archivos de Salida (outputs/)

### 🎯 Archivo Principal
- **`nomina_alertas.csv`** ← Use this for Power BI
  - ID anonimizado | Riesgo (Bajo/Medio/Alto) | Factores SHAP top 3

### 📊 Evalución
- `evaluacion_test.csv` - Desempeño del modelo
- `auditoria_equidad.csv` - Equidad por subgrupo
- `cumplimiento_requerimientos.csv` - Trazabilidad

### 🤖 Modelo
- `modelo_satmat.joblib` - Exportable a producción

### 📈 Otros
- `resultados_cv.csv`, `predicciones_perfiles.csv`, `pruebas_funcionales.csv`, etc.

---

## 5️⃣ Problemas Comunes

### ❌ Colab: "No encontré SAT-Mat/data"
**Solución**:
1. Crear en Google Drive: SAT-Mat › data/
2. Copiar archivos CSV en data/
3. Volver a ejecutar notebook

### ❌ Colab: "Falta validation.csv"
**Solución**: Verificar que `validation.csv` está en `SAT-Mat/data` (no en raíz)

### ❌ Local: "Carpeta 'data' no encontrada"
**Solución**: 
```bash
mkdir data/
cp /ruta/validation.csv data/
cp /ruta/test.csv data/
```

### ❌ Local: "ImportError: No module 'shap'"
**Solución**:
```bash
pip install -r requirements.txt
```

### ❌ Ambos: "SystemExit(1)" o comando exit
**Solución**: Verificar que archivos CSV están en lugar correcto y reintentar

---

## 6️⃣ Verificar Ejecución

```bash
# Después de ejecutar, verificar outputs
ls -la outputs/

# Debe contener:
# ✓ nomina_alertas.csv
# ✓ modelo_satmat.joblib
# ✓ evaluacion_test.csv
# ✓ auditoria_equidad.csv
# [+ 15 archivos más]

# Verificar tamaño del modelo
ls -lh outputs/modelo_satmat.joblib
# Esperado: ~100 KB
```

---

## 7️⃣ Próximos Pasos

✅ **Power BI**: Cargar `nomina_alertas.csv` para dashboard  
✅ **Producción**: Usar `modelo_satmat.joblib` en AWS Lambda / microservicio  
✅ **Auditoria**: Revisar `auditoria_equidad.csv` y `cumplimiento_requerimientos.csv`  
✅ **Mejoras**: Ver sección **Limitaciones** en [README.md](README.md)

---

## 📚 Documentación Completa

Para detalles arquitectónicos y análisis profundo:
- **[README.md](README.md)** - Descripción completa del proyecto
- **[ESTRUCTURA_PROYECTO.md](ESTRUCTURA_PROYECTO.md)** - Mapeo detallado de fases y salidas

---

## 💬 Soporte

**Equipo**: Arelis Tovar, Dayana Vega, Encar Portillo  
**Curso**: SCY1102 Gestión de Proyectos de Datos  

---

**¡Listo para ejecutar! 🎉**
