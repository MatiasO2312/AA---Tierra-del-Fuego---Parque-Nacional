# Predicción de la actividad turística en el Parque Nacional Tierra del Fuego

Proyecto del parcial de **Aprendizaje Automático** del Politécnico Malvinas Argentinas.  
Objetivo: predecir el nivel de visitas al Parque Nacional Tierra del Fuego y el tipo de visitante predominante (residente / no residente) para el mes siguiente, usando datos públicos del IPIEC.

## Datasets utilizados

Los tres datasets se encuentran en `data/raw/` y provienen del **IPIEC** (Instituto Provincial de Análisis e Investigación, Estadística y Censos), organismo oficial de estadísticas de la provincia de Tierra del Fuego.

> **Nota sobre visualización:** Los archivos `.xlsx` contienen múltiples hojas (datos mensuales, datos anuales, índice y ficha técnica). GitHub puede previsualizar archivos de una sola hoja, pero no archivos con varias pestañas. Por esa razón se incluyen en el repositorio para descarga directa. La descripción completa de columnas, instancias y tipos de datos se encuentra en [`docs/descripcion_dataset_y_origen.md`](docs/descripcion_dataset_y_origen.md).

| Archivo | Contenido | Período | Fuente |
|---|---|---|---|
| `16_1_04_Visitas-al-Parque-Nacional-TDF.xlsx` | Visitas al PN TDF por mes: total, residentes y no residentes | 2015–2026 | IPIEC – Turismo |
| `16_1_01_Pernoctaciones_ingresos_estadia_promedio.xlsx` | Pernoctaciones, viajeros y estadía promedio en Ushuaia por mes | 2004–2026 | IPIEC – Turismo |
| `22_2_01_Meteorologia_Temperatura_Precipitaciones.xlsx` | Temperatura, lluvia y días con nieve en Ushuaia y Río Grande por mes | 2009–2025 | IPIEC – Medio Ambiente |

## Estructura del proyecto

```text
data/
  raw/        # Datasets originales (.xlsx) descargados de IPIEC
  processed/  # Dataset limpio con lag y variables objetivo
notebooks/    # Notebooks de exploración, preprocesamiento y modelos
src/          # Código Python reutilizable (carga de datos, funciones de modelo)
reports/
  figures/    # Gráficos y tablas para el informe final
docs/         # Documentos en Markdown (descripción del dataset, análisis, etc.)
```

## Origen de los datos

Los tres datasets se descargaron de la página oficial del IPIEC: 

- Estadísticas económicas – sección Turismo  
- Estadísticas del medio ambiente – sección Ambientales

El IPIEC tiene como misión **producir y difundir datos estadísticos oficiales de calidad** sobre la realidad económica, social y ambiental de la provincia, para apoyar la toma de decisiones públicas y privadas.

## Próximos pasos

- Describir en detalle las variables de cada dataset.
- Unir visitas, pernoctaciones y meteorología por año y mes.
- Crear las variables objetivo `nivel_temporada` y `perfil_visitante`.
- Entrenar y evaluar modelos de clasificación (KNN, Árbol de Decisión, SVM) usando scikit-learn.
