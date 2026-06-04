# Descripción de los datasets y origen de los datos

Este documento describe los datasets utilizados en el proyecto de Aprendizaje Automático
orientado a predecir la actividad turística del Parque Nacional Tierra del Fuego
(nivel de visitas y tipo de visitante predominante) a partir de información oficial del IPIEC.

---

## 1. Origen de los datos

Todos los datos provienen del **Instituto Provincial de Análisis e Investigación,
Estadística y Censos (IPIEC)** de la Provincia de Tierra del Fuego AeIAS.

El IPIEC tiene como misión recopilar, organizar, analizar, producir y difundir datos
estadísticos de alta calidad y relevancia para la sociedad, de forma eficaz, independiente
y profesional, para favorecer la toma de decisiones públicas y privadas.

Los archivos originales se descargaron desde:

- **Estadísticas económicas → Turismo**
- **Estadísticas del medio ambiente → Ambientales**

y se guardaron sin modificar en la carpeta `data/raw/` del repositorio.

---

## 2. Dataset 1 — Visitas al Parque Nacional TDF

**Archivo:** `data/raw/16_1_04_Visitas-al-Parque-Nacional-TDF.xlsx`  
**Fuente:** IPIEC, sobre la base de datos de la Dirección de la Administración de Parques Nacionales.  
**Cobertura temporal utilizada:** 2015–2026 (mensual).  
**Cobertura geográfica:** Provincia de Tierra del Fuego AeIAS.

El archivo contiene varias hojas (tabla de datos, ficha técnica).  
En este proyecto se utiliza únicamente la hoja principal de datos mensuales.

Después de la lectura con `pandas` y el filtrado de filas válidas, el dataset queda con:

- **135 registros mensuales** entre enero de 2015 y marzo de 2026.
- **Columnas:**
  - `año` (int): año calendario.
  - `mes` (str): nombre del mes.
  - `total_visitas` (float): total de visitas al parque en el mes.
  - `residentes` (float): visitas de residentes.
  - `no_residentes` (float): visitas de no residentes.

**Problemas de calidad detectados:**

- En **2020** hay meses sin datos (abril, mayo y junio con valores nulos) y varios meses
  sin información de `no_residentes`, debido al cierre del parque por la pandemia de COVID‑19.
- Existe un registro aislado con valores faltantes en 2022.

**Decisión tomada:**

- **Excluir el año 2020 completo** del análisis, ya que representa una anomalía estructural
  (cierre total o parcial del parque) y no un comportamiento turístico normal.
- El resto de los años se conserva, eliminando únicamente las filas con valores nulos
  que no corresponden a 2020.

Este dataset es la base para construir las dos variables objetivo del proyecto:
`nivel_temporada` y `perfil_visitante`.

---

## 3. Dataset 2 — Meteorología (Temperatura y Precipitaciones)

**Archivo:** `data/raw/22_2_01_Meteorologia_Temperatura_Precipitaciones.xlsx`  
**Fuente:** IPIEC, con datos meteorológicos de Ushuaia y Río Grande.  
**Cobertura temporal utilizada:** 2015–2025 (mensual).  
**Cobertura geográfica:** Ushuaia y Río Grande.

El archivo incluye varias hojas (índice, datos por mes, datos por año, ficha técnica).
Para este proyecto se usa la hoja **“por mes”**, que contiene:

- Temperatura máxima media, mínima media y media mensual.
- Precipitaciones acumuladas en mm.
- Días con nieve.

Para cada mes se registran valores separados para **Río Grande** y **Ushuaia**.  
Luego de la limpieza y selección de columnas, el dataset queda con:

- **132 registros mensuales** entre enero de 2015 y diciembre de 2025.
- **Columnas principales:**
  - `año`, `mes`
  - `rg_temp_max`, `rg_temp_min`, `rg_temp_media`, `rg_lluvia_mm`, `rg_dias_nieve`
  - `ush_temp_max`, `ush_temp_min`, `ush_temp_media`, `ush_lluvia_mm`, `ush_dias_nieve`

**Problemas de calidad detectados:**

- Las columnas de Ushuaia (`ush_temp_*`, `ush_lluvia_mm`, `ush_dias_nieve`) presentan
  **valores faltantes en los 12 meses de 2024**.
- Algunas columnas de días con nieve tienen muchos ceros o nulos, lo que refleja
  ausencia de eventos o falta de registro.

**Decisiones tomadas:**

- Mantener el año **2024** en el análisis por su relevancia temporal.
- Para 2024, **usar las variables climáticas de Río Grande como proxy de Ushuaia**,
  ya que ambas ciudades pertenecen a la misma provincia y muestran un comportamiento
  climático similar en la serie histórica.
- Documentar explícitamente este uso de proxy en el preprocesamiento.

Este dataset aporta las **features climáticas del mes actual** que se utilizarán para
predecir el comportamiento turístico del mes siguiente.

---

## 4. Dataset 3 — Pernoctaciones y Viajeros en Ushuaia

**Archivo:** `data/raw/16_1_01_Pernoctaciones_ingresos_estadia_promedio.xlsx`  
**Fuente:** IPIEC, a partir de la Encuesta de Ocupación Hotelera (EOH) del INDEC
y del Observatorio Estadístico del Municipio de Río Grande.  
**Cobertura temporal utilizada:** 2015–2025 (mensual).  
**Cobertura geográfica:** ciudades de Ushuaia y Río Grande.

El archivo tiene varias hojas de datos mensuales por período
(2004–2011, 2012–2023, 2024 en adelante) más ficha técnica.
En este proyecto se extraen únicamente las columnas correspondientes a **Ushuaia**:

- `año`, `mes`
- `ush_pern_total`  (pernoctaciones totales)
- `ush_viaj_total`  (viajeros totales)
- `ush_estadia_total` (estadía promedio en días)

Después de combinar las diferentes hojas, el dataset queda con:

- **129 registros mensuales** para Ushuaia entre 2015 y 2025.
- Nulos solo en **abril y mayo de 2020**, coherentes con las restricciones turísticas
  de la pandemia.

**Decisión tomada:**

- Excluir las observaciones de 2020 en coherencia con el tratamiento aplicado al
  dataset de visitas al Parque Nacional.
- Utilizar las variables de pernoctaciones, viajeros y estadía promedio como
  **features complementarias** que capturan el nivel general de actividad turística
  de Ushuaia, más allá del parque.

---

## 5. Período final de trabajo

Luego de considerar la calidad de los datos y las anomalías por COVID‑19, se define:

- **Período de análisis principal:** 2015–2025  
- **Años excluidos:** 2020 (en todos los datasets utilizados)

Además, al aplicar un **lag de 1 mes** para transformar el problema en predicción
del mes siguiente, se pierden las últimas filas sin target disponible.

Se espera trabajar finalmente con alrededor de **100 instancias mensuales**,
lo cual es suficiente para un ejercicio académico de clasificación con los
modelos vistos en la materia (KNN, Árbol de Decisión, SVM).

---
