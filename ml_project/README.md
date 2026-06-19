# ML Project – Análisis y predicción del subempleo por insuficiencia de horas en el Perú

**Universidad del Pacífico – Facultad de Ingeniería | Curso: Machine Learning**  
Jordán, Guillermo · Munayco, Alessandra · Alvarez de la Torre, Arturo · Rojas, Priscila

> Aplicamos modelos de clasificación supervisada para predecir el **subempleo por insuficiencia de horas** en trabajadores ocupados del Perú, usando datos de la EPEN 2024 (INEI).

---

## Estructura del proyecto

```
ml_project/
│
├── 01_raw_data/
│   └── original_dataset.ipynb          ← Carga y documentación del dataset original
│
├── 02_data_understanding/
│   ├── data_dictionary_review.ipynb    ← Revisión del diccionario de variables
│   ├── target_definition.ipynb         ← Definición y análisis de la variable objetivo
│   └── exploratory_analysis.ipynb      ← Análisis exploratorio de datos (EDA)
│
├── 03_preprocessing/
│   ├── missing_values_handling.ipynb   ← Tratamiento de valores faltantes
│   ├── variable_filtering.ipynb        ← Eliminación de variables no informativas
│   ├── categorical_encoding.ipynb      ← Codificación de variables categóricas
│   └── train_test_split.ipynb          ← División entrenamiento / prueba (80/20)
│
├── 04_class_balancing/
│   └── oversampling_or_undersampling.ipynb  ← SMOTE y Random Undersampling
│
├── 05_feature_engineering/
│   ├── income_features.ipynb           ← Variables derivadas del ingreso
│   ├── employment_features.ipynb       ← Variables derivadas del empleo
│   ├── education_features.ipynb        ← Variables derivadas de educación
│   └── demographic_features.ipynb      ← Variables demográficas (edad, sexo, etc.)
│
├── 06_feature_selection/
│   └── selected_variables.ipynb        ← Selección por MI, RF Importance y SelectFromModel
│
├── 07_modelling/
│   ├── baseline_model.ipynb            ← Regresión Logística y Dummy Classifier
│   ├── decision_tree_or_random_forest.ipynb  ← Árbol de Decisión y Random Forest + GridSearchCV
│   └── model_comparison.ipynb          ← Comparación de todos los modelos
│
├── 08_evaluation/
│   ├── metrics.ipynb                   ← Métricas completas + curvas ROC y PR
│   ├── confusion_matrix.ipynb          ← Análisis de la matriz de confusión
│   └── feature_importance.ipynb        ← MDI, Permutación e (opcional) SHAP
│
├── 09_results/
│   ├── final_model.ipynb               ← Pipeline final serializado + validación
│   ├── conclusions.ipynb               ← Hallazgos y recomendaciones de política
│   └── limitations.ipynb               ← Limitaciones metodológicas y éticas
│
└── README.md                           ← Este archivo
```

---

## Orden de ejecución

Los notebooks deben ejecutarse **en el orden numérico** de las carpetas, ya que cada etapa genera archivos que son consumidos por la siguiente:

```
01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09
```

### Archivos intermedios generados

| Directorio | Archivos |
|---|---|
| `data/raw/` | `epen_snapshot.csv` |
| `data/processed/` | `epen_missing_handled.csv`, `epen_filtered.csv`, `epen_encoded.csv` |
| `data/split/` | `X_train.csv`, `X_test.csv`, `y_train.csv`, `y_test.csv` |
| `data/balanced/` | `X_train_balanced.csv`, `y_train_balanced.csv` |
| `data/feature_engineering/` | `epen_fe_income.csv`, `epen_fe_employment.csv`, `epen_fe_education.csv`, `epen_fe_demographic.csv` |
| `data/selected/` | `epen_selected.csv`, `selected_features.csv`, `feature_importances.csv` |
| `data/results/` | `model_comparison.csv`, `final_metrics.csv`, `feature_importance_final.csv` |
| `models/` | `baseline_lr.pkl`, `decision_tree.pkl`, `random_forest.pkl`, `final_model_pipeline.pkl`, `model_metadata.json` |

---

## Requerimientos

```
pandas>=1.5
numpy>=1.23
scikit-learn>=1.2
matplotlib>=3.6
seaborn>=0.12
imbalanced-learn>=0.10   # Para SMOTE
joblib>=1.2
shap>=0.42               # Opcional – para explicabilidad avanzada
```

Instalar con:
```bash
pip install pandas numpy scikit-learn matplotlib seaborn imbalanced-learn joblib
```

---

## Variable objetivo

| Variable | Descripción |
|---|---|
| `target_desocupado` | `1` = Persona desocupada (busca empleo activamente) |
| | `0` = Persona ocupada o fuera de la PEA |

---

## Modelo final

- **Algoritmo:** Random Forest Classifier
- **Preprocesamiento:** StandardScaler dentro de un Pipeline de scikit-learn
- **Optimización:** GridSearchCV con 5-fold cross-validation
- **Métrica principal:** ROC-AUC
