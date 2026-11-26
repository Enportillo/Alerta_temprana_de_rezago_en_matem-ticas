# 📥 Guía de Carga de Datos - SAT-Mat

**Sistema inteligente de carga para Google Colab y Jupyter Local**

---

## ⚡ Cómo Funciona

```
┌─────────────────────────────────────────────────────────────┐
│          Primera Celda del Notebook                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  try:                                                       │
│      import google.colab                                    │
│      EN_COLAB = True                                        │
│  except:                                                    │
│      EN_COLAB = False                                       │
│                                                             │
│  if EN_COLAB:                                               │
│      # Colab: Autoriza Drive, busca, descarga              │
│      DATA_PATH = 'data'  (en /content/SAT-Mat)             │
│  else:                                                      │
│      # Local: Verifica carpeta data/                       │
│      DATA_PATH = 'data'  (junto al notebook)              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🌐 Google Colab

### Setup (3 pasos)

**1. Crear estructura en Google Drive**
```
📁 Compartido conmigo o Mi unidad
   └─ 📁 SAT-Mat/
      └─ 📁 data/
         ├─ validation.csv
         └─ test.csv
```

**2. Ir a Google Colab**
```
https://colab.research.google.com/
Archivo › Subir notebook › SAT-Mat_Implementacion.ipynb
```

**3. Ejecutar notebook**
```
Entorno de ejecución › Ejecutar todas

La primera celda:
  ✓ Detecta Colab
  ✓ Pide autorizar Drive
  ✓ Busca SAT-Mat/data
  ✓ Descarga CSV
  ✓ Define DATA_PATH
  ✓ Resto ejecuta normalmente
```

### Resultado
```
✓ Entorno: Google Colab
✓ Paso 1: Autorizando acceso a Google Drive...
✓ Paso 2: Buscando carpeta SAT-Mat en tu Drive...
  ✓ Carpeta SAT-Mat/data encontrada
✓ Paso 3: Descargando archivos de Drive...
  Descargando validation.csv... ✓ validation.csv: 300 MB
  Descargando test.csv... ✓ test.csv: 300 MB
✓ Listo. Directorio: /content/SAT-Mat
```

### Problemas Colab

| Problema | Solución |
|----------|----------|
| "No encontré SAT-Mat/data" | Crear en Drive: SAT-Mat › data/, copiar CSV |
| "AssertionError" | Click en link de autorización de Drive |
| "Falta validation.csv" | Copiar CSV en SAT-Mat/data (no raíz) |
| Lento | Normal, es 1M filas. Esperar ~5-10 min |

---

## 💻 Jupyter Local

### Setup (3 pasos)

**1. Preparar carpeta data/**
```bash
# La carpeta ya existe. Copiar archivos:
cp /ruta/validation.csv Anexo_I_SAT-Mat_Implementacion/data/
cp /ruta/test.csv Anexo_I_SAT-Mat_Implementacion/data/

# Verificar estructura:
# Anexo_I_SAT-Mat_Implementacion/
# ├── SAT-Mat_Implementacion.ipynb
# ├── data/
# │  ├── validation.csv    ← AQUÍ
# │  └── test.csv          ← AQUÍ
# ├── QUICK_START.md
# └── requirements.txt
```

**2. Instalar dependencias**
```bash
pip install -r requirements.txt
```

**3. Ejecutar notebook**
```bash
jupyter notebook SAT-Mat_Implementacion.ipynb

La primera celda:
  ✓ Detecta Local
  ✓ Verifica carpeta 'data/'
  ✓ Verifica CSV presentes
  ✓ Si falta algo, da instrucciones y detiene
  ✓ Define DATA_PATH
  ✓ Kernel › Reiniciar & Ejecutar Todo
```

### Resultado
```
Entorno: Jupyter Local
✓ validation.csv: 300 MB
✓ test.csv: 300 MB
✓ Listo. Directorio: /home/usuario/.../Anexo_I_SAT-Mat_Implementacion
```

### Problemas Local

| Problema | Solución |
|----------|----------|
| "Carpeta 'data' no encontrada" | Crear: `mkdir data/` |
| "Falta validation.csv" | Copiar CSV a `data/` |
| "ImportError: shap" | `pip install -r requirements.txt` |
| "FileNotFoundError" | Verificar que cwd es correcto (jupyter desde carpeta proyecto) |

---

## 🔄 Flujo Completo

### Colab
```
Notebook en Colab
    ↓
Primera celda: try import google.colab → ÉXITO
    ↓
EN_COLAB = True
    ↓
Instala SHAP
    ↓
Autoriza Google Drive
    ↓
Busca SAT-Mat/data en Drive
    ↓
Descarga validation.csv y test.csv
    ↓
Define DATA_PATH = 'data' en /content/SAT-Mat
    ↓
Resto del notebook: usar DATA_PATH
    ↓
Resultados en /content/SAT-Mat/outputs/ y /figs/
```

### Local
```
Notebook en Jupyter local
    ↓
Primera celda: try import google.colab → FALLA
    ↓
EN_COLAB = False
    ↓
Verifica os.path.exists('data')
    ↓
Verifica 'validation.csv' en data/
    ↓
Verifica 'test.csv' en data/
    ↓
Define DATA_PATH = 'data'
    ↓
Resto del notebook: usar DATA_PATH
    ↓
Resultados en ./outputs/ y ./figs/
```

---

## 📝 Notas Importantes

### Variable DATA_PATH
La primera celda define:
```python
DATA_PATH = 'data'  # Ambos entornos usan esto
```

Resto del notebook usa:
```python
train = pd.read_csv(f'{DATA_PATH}/validation.csv')
test = pd.read_csv(f'{DATA_PATH}/test.csv')
```

✓ Funciona en Colab: `/content/SAT-Mat/data/`  
✓ Funciona en Local: `./data/`

### Verificación Automática
Primera celda **detiene la ejecución** si:
- Falta carpeta data/ (Local)
- Faltan CSV (Local)
- Falta Drive auth (Colab)
- Falta SAT-Mat/data en Drive (Colab)

✓ Mejor que ejecutar con archivos faltantes

### Descarga de Colab
Archivos generados en Colab se **borran al cerrar**.

✓ Descargar outputs/ y figs/ antes de cerrar

---

## 🎯 Checklist Rápido

### Colab
- [ ] Google Drive preparado (SAT-Mat › data › CSV)
- [ ] Google Colab abierto (colab.research.google.com)
- [ ] Notebook subido
- [ ] "Ejecutar todas" iniciado
- [ ] Primera celda completada sin errores
- [ ] Outputs descargados antes de cerrar

### Local
- [ ] Carpeta data/ existe
- [ ] CSV copiados a data/
- [ ] pip install -r requirements.txt ejecutado
- [ ] jupyter notebook iniciado desde carpeta proyecto
- [ ] Notebook abierto
- [ ] Kernel › Reiniciar & Ejecutar Todo
- [ ] Primera celda completada sin errores

---

## 📞 Soporte

Si hay problemas:
1. Revisar tabla de problemas arriba
2. Consultar QUICK_START.md (troubleshooting)
3. Revisar README.md (sección ejecución)
4. Contactar al equipo

---

**¡Listo para ejecutar! 🚀**
