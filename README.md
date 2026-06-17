# Predicción de la actividad turística en el Parque Nacional Tierra del Fuego

Proyecto del parcial de **Aprendizaje Automático** del Politécnico Malvinas Argentinas.  
Objetivo: predecir el nivel de visitas al Parque Nacional Tierra del Fuego y el tipo de visitante predominante (residente / no residente) para el mes siguiente, usando datos públicos del IPIEC.

---

## Datasets utilizados

Los tres datasets se encuentran en `data/raw/` y provienen del **IPIEC** (Instituto Provincial de Análisis e Investigación, Estadística y Censos), organismo oficial de estadísticas de la provincia de Tierra del Fuego.

> **Nota sobre visualización:** Los archivos `.xlsx` contienen múltiples hojas (datos mensuales, datos anuales, índice y ficha técnica). GitHub puede previsualizar archivos de una sola hoja, pero no archivos con varias pestañas. Por esa razón se incluyen en el repositorio para descarga directa.  
> La descripción completa de columnas, instancias, tipos de datos y decisiones de preprocesamiento se encuentra en [`docs/descripcion_dataset_y_origen.md`](docs/descripcion_dataset_y_origen.md).

| Archivo | Contenido | Período | Fuente |
|---|---|---|---|
| `16_1_04_Visitas-al-Parque-Nacional-TDF.xlsx` | Visitas al PN TDF por mes: total, residentes y no residentes | 2015–2026 | IPIEC – Turismo |
| `16_1_01_Pernoctaciones_ingresos_estadia_promedio.xlsx` | Pernoctaciones, viajeros y estadía promedio en Ushuaia por mes | 2004–2026 | IPIEC – Turismo |
| `22_2_01_Meteorologia_Temperatura_Precipitaciones.xlsx` | Temperatura, lluvia y días con nieve en Ushuaia y Río Grande por mes | 2009–2025 | IPIEC – Medio Ambiente |

---

## Estructura del proyecto

```text
data/
  raw/        # Datasets originales (.xlsx) descargados de IPIEC
  processed/  # Dataset limpio con lag y variables objetivo (si se guarda)
notebooks/    # Notebooks de exploración, preprocesamiento y modelos
src/          # Código Python reutilizable (carga de datos, funciones de modelo)
reports/
  figures/    # Gráficos y tablas del EDA y del modelo
docs/         # Documentos en Markdown (descripción del dataset, etc.)
```

---

## Notebooks

En la carpeta `notebooks/` se encuentran los análisis desarrollados:

- `01_exploracion_datasets.ipynb`  
  Carga de los tres datasets del IPIEC, limpieza inicial, análisis exploratorio de datos (EDA) y detección de problemas de calidad (nulos, año 2020 atípico, falta de datos meteorológicos de Ushuaia en 2024).
- `02_modelo_clasificacion.ipynb`  
  Construcción del dataset combinado (visitas + clima + pernoctaciones), creación de las variables objetivo `nivel_temporada` y `perfil_visitante`, generación del lag de 1 mes, entrenamiento de modelos de clasificación (KNN y Árbol de Decisión) y evaluación con métricas.

---

## Figuras principales

Las figuras generadas en los notebooks se guardaron en `reports/figures/`:

- `estacionalidad_visitas.png`  
  Promedio de visitas al PN TDF por mes (2015–2025, sin 2020), mostrando la estacionalidad marcada entre verano e invierno.
- `visitas_por_anio.png`  
  Total anual de visitas al parque, con caída post-pandemia y recuperación hacia 2023–2024.
- `distribucion_targets.png`  
  Distribución de las clases de `nivel_temporada` (Alta, Media, Baja) y del `perfil_visitante` (Residente / No_Residente); muestra que el primer target está balanceado y el segundo está desbalanceado.
- `correlacion_features.png`  
  Matriz de correlación entre variables numéricas (visitas, clima, pernoctaciones, viajeros, estadía promedio).
- `cm_knn_nivel_temporada.png`  
  Matriz de confusión del mejor modelo (KNN) para la predicción de `nivel_temporada`.
- `cm_tree_nivel_temporada.png`  
  Matriz de confusión del Árbol de Decisión para comparación e interpretación.

Estas figuras se utilizan también en el video para ilustrar el EDA y el desempeño de los modelos.

---

## Decisiones de preprocesamiento

Principales decisiones tomadas antes de modelar:

- **Exclusión de 2020** en todos los datasets:  
  El Parque Nacional estuvo cerrado o con restricciones por la pandemia de COVID-19, generando valores nulos o atípicos que rompen la estacionalidad normal del turismo.
- **Meteorología 2024 – uso de Río Grande como proxy de Ushuaia:**  
  El dataset meteorológico no tiene datos de Ushuaia para 2024. Se usan las variables de Río Grande (misma provincia y comportamiento climático similar) como aproximación, manteniendo así los meses recientes en el análisis.
- **Período de trabajo:**  
  Se trabaja con datos mensuales de 2015 a 2025, excluyendo 2020. Tras el cruce de datasets y la creación del lag de 1 mes, quedan **115 observaciones** útiles.
- **Lag de 1 mes:**  
  Las variables del mes actual (clima, visitas, pernoctaciones) se usan para predecir el nivel de visitas del **mes siguiente**, transformando el problema en predicción a un paso adelante.

---

## Variables objetivo

A partir de las visitas del mes siguiente se construyeron dos targets:

- `nivel_temporada`  
  Clasificación en **Alta**, **Media** y **Baja** temporada usando terciles del número de visitas del mes siguiente. La distribución quedó casi equilibrada (Alta ≈ 40, Media ≈ 37, Baja ≈ 38).
- `perfil_visitante`  
  Indica si en el mes siguiente predominan los **Residentes** o los **No_Residentes`. Esta variable quedó desbalanceada (muchos meses con residentes, pocos con no residentes), por lo que se analizó como información complementaria pero no se utilizó como objetivo principal de modelado.

---

## Modelos de Aprendizaje Automático

El objetivo de modelado fue predecir el `nivel_temporada` del mes siguiente.

### Features utilizadas

- Visitas del mes actual: `total_visitas`, `residentes`, `no_residentes`.
- Variables climáticas: `ush_temp_media`, `ush_lluvia_mm`, `rg_temp_media`, `rg_lluvia_mm`.
- Actividad hotelera en Ushuaia: `ush_pern_total`, `ush_viaj_total`, `ush_estadia_total`.
- Mes del año como variable categórica.

### Preprocesamiento y pipeline

Se utilizó un `ColumnTransformer` con:

- **SimpleImputer (mediana)** + **StandardScaler** para variables numéricas.
- **OneHotEncoder** para el mes.

Todo se integró en un `Pipeline` de scikit-learn (preprocesamiento + modelo), evitando fugas de información entre train y test.

### Modelos probados

- **K-Nearest Neighbors (KNN, k = 5)**  
  - Accuracy en test ≈ **0.74**; F1 macro ≈ **0.74**.  
  - Accuracy promedio en validación cruzada (5 folds): **0.67 ± 0.08**.  
  - Buen rendimiento para distinguir meses de **Alta** y **Baja** temporada; mayor confusión en la clase **Media**.
- **Árbol de Decisión (profundidad máxima 4)**  
  - Accuracy en test ≈ **0.61**; accuracy medio en CV ≈ **0.52 ± 0.06**.  
  - Peor desempeño que KNN, pero aporta interpretabilidad sobre la importancia de las variables (visitas y viajeros del mes actual y el mes del año como factores clave).

---

## Conclusiones generales

El trabajo muestra que, combinando datos oficiales del IPIEC de visitas, clima y pernoctaciones, es posible construir un pipeline completo de Aprendizaje Automático para el Parque Nacional Tierra del Fuego: desde la selección y limpieza de datasets, pasando por el EDA, hasta el entrenamiento y evaluación de modelos de clasificación.

Con un dataset mensual relativamente pequeño (115 observaciones), el modelo KNN logra una capacidad de predicción **moderada** del `nivel_temporada` del mes siguiente, suficiente para identificar razonablemente los meses de alta y baja temporada, aunque con más dificultad en la categoría media. El Árbol de Decisión complementa el análisis aportando interpretabilidad.

Se exploró usar `perfil_visitante` (Residente / No_Residente) como segundo objetivo, pero el desbalance de clases y la cantidad limitada de datos hacen que las métricas para la clase minoritaria sean poco robustas. Por eso se documenta como variable de análisis complementaria y se propone trabajarla en el futuro con más datos y técnicas específicas para clases desbalanceadas.

---

## Video de presentación

El video explicativo del proyecto (objetivos, datos, EDA, modelos y resultados) está disponible en:

> **[Enlace al video](URL_DEL_VIDEO)**  ← reemplazar con la URL final (YouTube o archivo accesible).

En el video se recorre este repositorio, se muestran los notebooks y las figuras principales, y se discuten las decisiones de modelado y las conclusiones del proyecto.
