# Project-Break-II-ML
 Proyecto de Machine Learning para el entrenamiento de un modelo en base a un Dataset
## Equipo compuesto por Danilo Castillo, Ricardo Díaz Coca y Arrate Alducin  

#  Predicción de Accidentes en Federados de Montaña de Madrid
#  Accident Prediction for Madrid Mountain Federation Members

---

## Español

### Descripción del proyecto

Este proyecto aborda un problema de **clasificación binaria** con el objetivo de predecir si un federado de la **Federación de Montaña** sufrirá un accidente durante una temporada. Para ello, se construyó un dataset propio a partir de registros internos de la federación (años 2020–2025), combinando datos de federados con el historial de siniestros.

El dataset cuenta con **121.276 registros** y presenta un fuerte desequilibrio de clases (~4,7% de casos positivos), lo que representa un reto metodológico central en el proyecto.

 Los ficheros Excel originales contienen datos confidenciales y **no están disponibles** en este repositorio.

---

### Pipeline del proyecto

**1. Generación del dataset**
Los datos se construyeron a partir de tablas Excel internas de la federación, cruzando el listado de federados con el registro histórico de siniestros. Cada fila representa un federado en una temporada concreta. La variable objetivo (`target`) indica si ese federado sufrió un accidente en esa temporada.

**2. Análisis exploratorio (EDA)**
Se analizó la distribución del target, la evolución de siniestros por temporada (2020–2025), la relación entre edad y accidentabilidad, el impacto de la modalidad de seguro y el historial previo de accidentes. Se identificó y documentó la naturaleza del desequilibrio de clases.

**3. Detección y corrección de data leakage**
Durante el análisis de correlaciones se detectó que la variable `numeroaccidentesaño` tenía una correlación de 0.95 con el target, confirmando data leakage. Dicha variable recoge los accidentes del mismo año que se quiere predecir, por lo que fue eliminada del modelo.

**4. Feature engineering**
- Se creó la variable `grupo_persona` combinando edad y categoría (Niño/a, Juvenil, Adulto × Hombre/Mujer).
- La modalidad de licencia se codificó de forma ordinal (escala 1–7) según el nivel de riesgo asociado a cada tipo de actividad.
- Se aplicó One-Hot Encoding sobre `grupo_persona`.
- Se aplicó transformación logarítmica (`log1p`) sobre `accidenteshistoricos` para reducir el efecto de la larga cola.
- Las variables numéricas se escalaron con `RobustScaler` para mayor robustez ante outliers.

**5. División train/test**
Se utilizó `GroupShuffleSplit` agrupando por `nsocio` para garantizar que los datos de un mismo federado no se repartan entre entrenamiento y test, evitando así una fuente adicional de data leakage.

**6. Entrenamiento y evaluación de modelos**
Se entrenaron tres modelos con validación cruzada por grupos (`GroupKFold`) y optimización de hiperparámetros mediante `GridSearchCV`:

| Modelo | AUC-ROC |
|---|---|
| Regresión Logística (baseline) | **0.6808** |
| XGBoost (con tuning) | 0.6934 (CV) |
| Random Forest | 0.6011 |

La métrica principal es el **AUC-ROC**, complementada con el **PR-AUC** (0.117, más del doble del baseline aleatorio de 0.05) dada la fuerte asimetría de clases.

**7. Interpretación del modelo**
Se calcularon los odds ratios del modelo de regresión logística para identificar los factores de riesgo más relevantes:

| Variable | Odds Ratio | Interpretación |
|---|---|---|
| `accidenteshistoricos` | 5.64 | Cada accidente previo multiplica por ~5.6 el riesgo |
| `modalidad` | 1.57 | Las licencias de mayor cobertura implican más riesgo |
| `edad` | 0.75 | A mayor edad, menor probabilidad de accidente |
| Niño/a Hombre | 0.11 | Los niños presentan muy bajo riesgo |

---

### Resultados y conclusiones

Los tres modelos evaluados convergieron en un rendimiento similar (AUC-ROC ≈ 0.68–0.69), lo que sugiere que el límite de rendimiento no está en el algoritmo sino en la **información disponible en las variables**. Muchos federados renuevan la licencia por motivos administrativos sin realizar actividad real, lo que introduce ruido estructural en el dataset.

El modelo no permite predecir accidentes individuales con alta precisión, pero **sí identifica perfiles de riesgo poblacional**, lo que puede ser útil para orientar campañas de prevención.

---

### Tecnologías utilizadas

- Python 3.12
- Pandas · NumPy
- Scikit-learn (LogisticRegression, RandomForest, GridSearchCV, GroupShuffleSplit, GroupKFold)
- XGBoost
- Matplotlib · Seaborn
- SciPy

---

 Se incluye una muestra del dataset en este repositorio, dicho dataset se encuetra en el siguiente enlace:  
 https://drive.google.com/file/d/1df0fJ6bPi5W26PJqgeBep9jJulWwAamR/view?usp=drive_link

---

---

## English

### Project description

This project addresses a **binary classification problem** aimed at predicting whether a member of the **Mountain Federation** will have an accident during a given season. A custom dataset was built from internal federation records (2020–2025), combining member data with historical accident records.

The dataset contains **121,276 records** with a significant class imbalance (~4.7% positive cases), which represents the main methodological challenge of the project.

The original Excel files contain confidential data and are **not available** in this repository.

---

### Project pipeline

**1. Dataset generation**
Data was built from internal federation Excel tables, joining the member list with the historical accident log. Each row represents a member in a specific season. The target variable (`target`) indicates whether that member had an accident during that season.

**2. Exploratory Data Analysis (EDA)**
Analysis covered target distribution, accident trends by season (2020–2025), the relationship between age and accident rates, the impact of insurance modality, and prior accident history. Class imbalance was identified and documented.

**3. Data leakage detection and correction**
A correlation analysis revealed that the variable `numeroaccidentesaño` had a 0.95 correlation with the target, confirming data leakage. This variable captures accidents from the same year being predicted and was therefore removed from the model.

**4. Feature engineering**
- The variable `grupo_persona` was created by combining age and category (Child, Youth, Adult × Male/Female).
- License modality was ordinally encoded (scale 1–7) based on the risk level associated with each type of activity.
- One-Hot Encoding was applied to `grupo_persona`.
- A logarithmic transformation (`log1p`) was applied to `accidenteshistoricos` to reduce the effect of the long tail.
- Numerical variables were scaled using `RobustScaler` for robustness against outliers.

**5. Train/test split**
`GroupShuffleSplit` was used, grouping by `nsocio`, to ensure that records from the same member are not split across training and test sets — avoiding an additional source of data leakage.

**6. Model training and evaluation**
Three models were trained using group cross-validation (`GroupKFold`) and hyperparameter tuning via `GridSearchCV`:

| Model | AUC-ROC |
|---|---|
| Logistic Regression (baseline) | **0.6808** |
| XGBoost (with tuning) | 0.6934 (CV) |
| Random Forest | 0.6011 |

The primary metric is **AUC-ROC**, complemented by **PR-AUC** (0.117, more than double the random baseline of 0.05) given the strong class imbalance.

**7. Model interpretation**
Odds ratios were computed from the logistic regression model to identify the most relevant risk factors:

| Variable | Odds Ratio | Interpretation |
|---|---|---|
| `accidenteshistoricos` | 5.64 | Each prior accident multiplies risk by ~5.6 |
| `modalidad` | 1.57 | Higher-coverage licenses imply greater risk |
| `edad` | 0.75 | Older age is associated with lower accident probability |
| Male Child | 0.11 | Children show very low accident risk |

---

### Results and conclusions

All three evaluated models converged to similar performance (AUC-ROC ≈ 0.68–0.69), suggesting that the performance ceiling is driven by **the information available in the variables** rather than the choice of algorithm. Many members renew their license for administrative reasons without engaging in actual mountain activity, which introduces structural noise into the dataset.

The model cannot predict individual accidents with high precision, but it **does identify population-level risk profiles**, which may be useful for guiding prevention campaigns.

---

### Technologies used

- Python 3.12
- Pandas · NumPy
- Scikit-learn (LogisticRegression, RandomForest, GridSearchCV, GroupShuffleSplit, GroupKFold)
- XGBoost
- Matplotlib · Seaborn
- SciPy

---

A sample of the dataset is included in this repository. The full dataset is available at the following link:  
https://drive.google.com/file/d/1df0fJ6bPi5W26PJqgeBep9jJulWwAamR/view?usp=drive_link
