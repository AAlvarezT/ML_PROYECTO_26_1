# 03_preprocessing

Esta carpeta contiene los notebooks dedicados al preprocesamiento del dataset de la Encuesta Permanente de Empleo Nacional 2024 (EPEN). Su objetivo es limpiar, transformar y preparar las variables del universo analítico para que puedan ser usadas de manera correcta y confiable en la etapa de modelamiento.

Esta etapa toma como punto de partida el archivo `epen_target_defined.csv`, generado en `02_data_understanding/03_target_definition.ipynb`, y produce un conjunto de datasets procesados listos para la ingeniería de características y el entrenamiento de modelos.

---

## Objetivo de la carpeta

La carpeta `03_preprocessing` tiene como propósito resolver los siguientes problemas antes del modelamiento:

- ¿Qué variables tienen valores faltantes y cómo deben tratarse?
- ¿Existen valores atípicos que puedan distorsionar los modelos?
- ¿Qué variables deben excluirse del modelo por razones metodológicas?
- ¿Cómo deben transformarse las variables categóricas para ser usadas por los algoritmos?
- ¿Las variables numéricas necesitan ser reescaladas para modelos sensibles a la escala?

El proyecto busca predecir **subempleo por insuficiencia de horas** en personas ocupadas. Por ello, el preprocesamiento mantiene como restricciones permanentes en todos los notebooks:

- El target `target_subempleo_horas` **no se modifica en ningún paso**.
- Las variables de leakage `P209H`, `C333` y `C334` **no se usan en ningún análisis predictivo**.
- No se realiza train/test split, balanceo de clases ni entrenamiento de modelos en ningún notebook de esta etapa.

---

## Archivos de la carpeta

```text
03_preprocessing/
│
├── 00_categorical_encoding.ipynb
├── 01_missing_values_handling.ipynb
├── 02_outliers_handling.ipynb
├── 03_variable_filtering.ipynb
└── reescaling_standarization_normalization.ipynb
```

---

### 00_categorical_encoding.ipynb

```text
Este notebook transforma las variables categóricas del dataset en representaciones numéricas
utilizables por modelos de Machine Learning, trabajando sobre epen_target_defined.csv.

Contenido principal
Revisión de columnas disponibles y del target.
Recodificación binaria de variables dicotómicas con valores 1/2 hacia 1/0.
Codificación ordinal para nivel educativo (C366) y tamaño de empresa.
One-Hot Encoding para variables nominales sin orden natural.
Validación de columnas generadas y guardado del dataset codificado.

Transformaciones aplicadas
  - Variables dicotómicas (sexo, protección social, discapacidad, etc.): 1/2 → 1/0
  - Nivel educativo (C366): escala ordinal creciente
  - Variables nominales (categoría ocupacional, tipo de contrato, etc.): dummies 0/1

Salida generada
data/processed/epen_encoded.csv
data/processed/encoded_columns.csv  (registro de columnas generadas)

Rol dentro del proyecto
Este notebook documenta la codificación de variables categóricas como paso previo al
filtrado y modelamiento. Su salida puede usarse de referencia para el pipeline final
de feature engineering cuando se combine con el dataset tratado por outliers.
```

---

### 01_missing_values_handling.ipynb

```text
Este notebook identifica, cuantifica y trata los valores faltantes en el dataset,
incluyendo tanto valores nulos reales (NaN) como códigos especiales de omisión
utilizados por la EPEN (por ejemplo: 99, 9999, 999999).

Contenido principal
Carga del dataset epen_target_defined.csv con fallback a epen_filtered.csv.
Validación inicial del target (existencia, ausencia de nulos, distribución).
Identificación de valores nulos reales por columna.
Identificación y reemplazo de códigos especiales de omisión de la EPEN.
Comparación del conteo de nulos antes y después de reemplazar códigos especiales.
Definición de la estrategia de imputación por tipo de variable.
Creación de flags de missingness (*_missing_flag) para variables de ingresos y horas.
Aplicación de imputación simple: mediana para numéricas, moda o −1 para categóricas.
Validación final de integridad del dataset.
Guardado del dataset tratado y del resumen de valores faltantes.

Estrategia de imputación
  - Variables numéricas (edad, horas, ingresos): mediana
  - Variables categóricas con nulos moderados: moda
  - Variables categóricas con > 60% de nulos: categoría −1 ("No informado")
  - Target y variables de leakage: no se imputan

Variables importantes revisadas
  Variable               Tratamiento
  INGTOT, INGTOTP        Mediana (missing = no respuesta)
  INGTRABW, I339_1, etc. Mediana (universo = trabajadores ocupados)
  C208                   Mediana
  C318_T, C328_T, whoraT Mediana
  C310, C311, C312, etc. Moda o −1 según porcentaje de nulos
  C376, C377 (etnicidad) Preferencia por −1 ("No informado")
  C375_* (discapacidad)  Moda

Salida generada
data/processed/epen_missing_handled.csv
data/processed/missing_values_summary.csv

Rol dentro del proyecto
Este notebook produce el dataset base para el tratamiento de outliers. Su resultado
epen_missing_handled.csv es el insumo directo de 02_outliers_handling.ipynb.
```

---

### 02_outliers_handling.ipynb

```text
Este notebook identifica y trata los valores atípicos en las variables numéricas
relevantes del dataset, con énfasis en edad, horas trabajadas e ingresos.

Principio general
No se elimina ninguna observación automáticamente por ser un outlier estadístico.
Cada decisión de tratamiento está justificada por el dominio de la encuesta laboral.
Los valores extremos pueden representar casos reales y válidos (jornadas extendidas,
ingresos muy altos), por lo que el tratamiento se justifica caso a caso.

Contenido principal
Carga del dataset epen_missing_handled.csv con fallback a epen_target_defined.csv.
Validación inicial del target y de variables de leakage.
Selección de variables numéricas candidatas: edad, horas trabajadas e ingresos.
Resumen estadístico con percentiles clave (p01, p05, p25, p50, p75, p95, p99).
Visualización de outliers mediante boxplots e histogramas por grupo de variables.
Identificación de outliers estadísticos con el método IQR.
Identificación de valores conceptualmente inválidos por reglas de dominio.
Creación de flags de outliers (*_outlier_flag) antes de modificar valores.
Aplicación del tratamiento por tipo de variable.
Comparación antes/después de las estadísticas descriptivas.
Validación final de integridad del dataset y del target.
Guardado del dataset tratado y de los resúmenes de outliers.

Estrategia de tratamiento
  Variable       Tratamiento
  C208 (edad)    Valores fuera de [14, 98] → NaN
  Horas (negativas)   → NaN
  Horas (> 112 h/semana)  → winsorización con clip(upper=112)
  Ingresos negativos  → NaN
  Ingresos extremos   → transformación log1p (columnas *_log)

Flags creados
  C208_outlier_flag, whoraT_outlier_flag,
  INGTOT_outlier_flag, INGTRABW_outlier_flag

Salida generada
data/processed/epen_outliers_handled.csv
data/processed/outliers_summary.csv
data/processed/outliers_before_after_summary.csv

Rol dentro del proyecto
Este notebook produce el dataset principal del pipeline de preprocesamiento.
epen_outliers_handled.csv es el insumo para el reescalamiento y el filtrado de variables.
```

---

### 03_variable_filtering.ipynb

```text
Este notebook define qué variables se conservarán para el modelamiento y cuáles se
excluirán por razones metodológicas. La selección no se basa en desempeño predictivo
estadístico, sino en criterios previos de calidad y pertinencia de las variables.

Criterios de exclusión aplicados
  - Data leakage: variables que revelan directamente el target
  - Identificadores administrativos: variables de encuesta, vivienda o diseño muestral
  - Factor de expansión: fa_son24 no es una característica individual predictiva
  - Variables duplicadas exactas: columnas con valores idénticos a otra columna
  - Alta cardinalidad: códigos de ocupación/actividad que generarían demasiadas dummies
  - Alto porcentaje de nulos: columnas con > 60% de faltantes sin relevancia conceptual
  - Columnas constantes: variables con un solo valor único

Contenido principal
Carga del dataset con orden de preferencia:
  epen_outliers_handled.csv → epen_missing_handled.csv → epen_target_defined.csv
Validación inicial del target y del universo analítico.
Eliminación de variables de data leakage (P209H, C333, C334).
Eliminación de identificadores administrativos (ANIO, MES, CONGLOMERADO, etc.).
Extracción del factor de expansión fa_son24 como variable auxiliar.
Análisis de redundancia y correlación entre variables de ingresos y horas.
Detección y eliminación de columnas con > 60% de nulos.
Detección y eliminación de columnas constantes; documentación de casi-constantes.
Exclusión de variables de alta cardinalidad (C308_COD, C309_COD).
Definición de variables candidatas por grupo temático:
  demográficas, educación, empleo, horas, ingresos, protección social,
  discapacidad, etnicidad, variables escaladas y flags de missing/outliers.
Construcción del dataset base df_model_base.
Validaciones finales con asserts.
Guardado del dataset filtrado y de los reportes de variables.

Variables de leakage excluidas
  P209H   → Variable original usada para construir el target
  C333    → Pregunta directamente si la persona quería trabajar más horas
  C334    → Pregunta directamente si la persona estuvo disponible para más horas

Salida generada
data/processed/epen_variable_filtered.csv
data/processed/variables_kept.csv
data/processed/variables_removed.csv
data/processed/high_missing_variables.csv
data/processed/constant_variables.csv

Rol dentro del proyecto
Este notebook produce el dataset base limpio para la etapa de feature engineering.
epen_variable_filtered.csv será el insumo para 04_feature_engineering/ y para el
train/test split posterior.
```

---

### reescaling_standarization_normalization.ipynb

```text
Este notebook aplica transformaciones de escala a las variables cuantitativas reales
del dataset, generando versiones estandarizadas y normalizadas para modelos sensibles
a la magnitud de las variables de entrada.

Contexto
Los modelos basados en árboles (Decision Tree, Random Forest) no requieren escalamiento.
Los modelos lineales (Regresión Logística), SVM, KNN y redes neuronales sí se ven
afectados por la escala de las variables. Este notebook prepara versiones escaladas
del dataset para esos modelos.

Contenido principal
Carga del dataset epen_outliers_handled.csv con fallback a epen_missing_handled.csv.
Identificación de variables numéricas candidatas para reescalar:
  edad (C208), horas trabajadas y variables de ingresos originales y logarítmicas.
Definición de columnas que NO se escalan: códigos categóricos, binarias, dummies,
  identificadores, flags de missing/outliers y el target.
Aplicación de tres métodos de escalamiento:
  - StandardScaler: media = 0, desviación estándar = 1 (columnas _std)
  - MinMaxScaler: rango [0, 1] (columnas _minmax)
  - RobustScaler: basado en IQR, resistente a outliers (columnas _robust)
Comparación de distribuciones antes y después del escalamiento.
Generación del dataset final epen_scaled.csv con RobustScaler como método principal,
  por su mayor robustez ante la asimetría de ingresos y horas en la EPEN.
Guardado de versiones separadas por método para comparación en modelamiento.

Método principal elegido
RobustScaler, por ser más resistente a la asimetría y valores extremos que caracterizan
a las variables de ingresos y horas en encuestas laborales.

Salida generada
data/processed/epen_scaled.csv          (versión principal con RobustScaler)
data/processed/epen_standard_scaled.csv
data/processed/epen_minmax_scaled.csv
data/processed/epen_robust_scaled.csv
data/processed/scaled_columns.csv       (registro de columnas escaladas)
data/processed/not_scaled_columns.csv   (registro de columnas no escaladas)
data/processed/scaling_summary.csv      (resumen comparativo de métodos)

Rol dentro del proyecto
Este notebook genera las versiones escaladas del dataset para ser usadas en modelos
lineales durante la etapa de modelamiento. Los modelos de árboles podrán usar las
variables originales sin escalar desde epen_outliers_handled.csv o
epen_variable_filtered.csv.
```

---

## Orden recomendado de ejecución

Los notebooks principales del pipeline deben ejecutarse en el siguiente orden:

```
01_missing_values_handling.ipynb
↓
02_outliers_handling.ipynb
↓
reescaling_standarization_normalization.ipynb
↓
03_variable_filtering.ipynb
```

El notebook `00_categorical_encoding.ipynb` puede ejecutarse de manera independiente a partir de `epen_target_defined.csv`, ya que su objetivo es documentar la codificación de variables categóricas como referencia metodológica.

---

## Archivos generados por esta etapa

| Notebook | Archivo generado | Uso posterior |
|---|---|---|
| `00_categorical_encoding.ipynb` | `epen_encoded.csv` | Referencia de codificación categórica |
| `00_categorical_encoding.ipynb` | `encoded_columns.csv` | Registro de columnas codificadas |
| `01_missing_values_handling.ipynb` | `epen_missing_handled.csv` | Insumo para tratamiento de outliers |
| `01_missing_values_handling.ipynb` | `missing_values_summary.csv` | Resumen de nulos tratados |
| `02_outliers_handling.ipynb` | `epen_outliers_handled.csv` | Insumo para escalamiento y filtrado |
| `02_outliers_handling.ipynb` | `outliers_summary.csv` | Resumen IQR por variable |
| `02_outliers_handling.ipynb` | `outliers_before_after_summary.csv` | Comparación antes/después |
| `03_variable_filtering.ipynb` | `epen_variable_filtered.csv` | Dataset base para feature engineering |
| `03_variable_filtering.ipynb` | `variables_kept.csv` | Registro de variables conservadas |
| `03_variable_filtering.ipynb` | `variables_removed.csv` | Registro de variables eliminadas |
| `03_variable_filtering.ipynb` | `high_missing_variables.csv` | Variables con alto % de nulos |
| `03_variable_filtering.ipynb` | `constant_variables.csv` | Variables constantes o casi constantes |
| `reescaling_standarization_normalization.ipynb` | `epen_scaled.csv` | Dataset principal escalado (RobustScaler) |
| `reescaling_standarization_normalization.ipynb` | `epen_standard_scaled.csv` | Versión con StandardScaler |
| `reescaling_standarization_normalization.ipynb` | `epen_minmax_scaled.csv` | Versión con MinMaxScaler |
| `reescaling_standarization_normalization.ipynb` | `epen_robust_scaled.csv` | Versión con RobustScaler |
| `reescaling_standarization_normalization.ipynb` | `scaled_columns.csv` | Registro de columnas escaladas |
| `reescaling_standarization_normalization.ipynb` | `not_scaled_columns.csv` | Registro de columnas no escaladas |
| `reescaling_standarization_normalization.ipynb` | `scaling_summary.csv` | Comparación de métodos de escalamiento |

---

## Decisiones metodológicas tomadas

En esta etapa se definieron las siguientes decisiones:

- El target `target_subempleo_horas` no se modifica en ningún notebook de preprocesamiento.
- Las variables `P209H`, `C333` y `C334` se tratan como variables de leakage y no se usan como predictores ni en imputaciones.
- Los valores fuera del rango etario válido `[14, 98]` se convierten a `NaN` en la variable de edad `C208`.
- Las horas trabajadas superiores a 112 h/semana (límite físico: 16 h × 7 días) se winsorizan con `clip(upper=112)`.
- Los ingresos negativos se convierten a `NaN` y los ingresos en general se transforman con `log1p` para reducir asimetría.
- Los flags de missingness (`*_missing_flag`) y de outliers (`*_outlier_flag`) se crean antes de modificar los valores originales, preservando la información sobre qué registros fueron afectados.
- Las variables `C308_COD` y `C309_COD` (códigos de ocupación y actividad económica) se excluyen del modelo base por su alta cardinalidad; podrían recuperarse en versiones avanzadas con agrupación por gran grupo CIUO o sección ISIC.
- `fa_son24` (factor de expansión muestral) no se usa como predictor; puede conservarse en archivos auxiliares para análisis descriptivo ponderado.
- Se elige **RobustScaler** como método principal de escalamiento por su mayor resistencia a la asimetría y valores extremos de las variables de ingresos y horas en la EPEN.

## Relación con las siguientes carpetas

La carpeta `03_preprocessing` alimenta directamente a la etapa de ingeniería de características ubicada en:

```
04_feature_engineering/
```

En particular, el archivo:

```
data/processed/epen_variable_filtered.csv
```

será usado como punto de partida para la creación de nuevas variables, el train/test split y la selección de características.
