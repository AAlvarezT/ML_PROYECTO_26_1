Objetivo del notebook

El notebook divide el dataset final de variables construidas en dos subconjuntos:

Train set: usado para selección de variables, entrenamiento de modelos y balanceo de clases.
Test set: reservado exclusivamente para evaluar el desempeño final de los modelos.

La división se realiza con una proporción 80/20 y con estratificación sobre el target, para mantener la misma distribución de clases en entrenamiento y prueba.

Dataset de entrada

El notebook utiliza como entrada:

../data/feature_engineering/epen_features_final.csv

Este archivo debe haber sido generado previamente por la etapa de feature engineering, luego de construir variables demográficas, educativas, laborales, de ingresos, protección social, horas trabajadas y otros indicadores relevantes.

Variable objetivo

La variable objetivo es:

target_subempleo_horas

Su interpretación es:

Valor	Significado
1	Trabajador con voluntad y disponibilidad para trabajar más horas
0	Trabajador sin esa condición
Validaciones realizadas

Antes de dividir los datos, el notebook valida que:

target_subempleo_horas exista en el dataset.
El target no tenga valores nulos.
No existan variables de leakage como P209H, C333 o C334.
Las variables predictoras y el target puedan separarse correctamente.

Estas validaciones son importantes porque P209H, C333 y C334 están directamente relacionadas con la construcción del target y no deben entrar como predictores.

Proceso realizado

El notebook sigue estos pasos:

Carga el dataset final de feature engineering.
Verifica la existencia y consistencia del target.
Separa las variables predictoras (X) y la variable objetivo (y).
Realiza la división con train_test_split.
Usa test_size=0.2, stratify=y y random_state=42.
Verifica que la distribución del target se mantenga en train y test.
Guarda los archivos resultantes en la carpeta data/split.
Parámetros usados
Parámetro	Valor	Justificación
test_size	0.2	Reserva 20% de los datos para evaluación
stratify	y	Mantiene la proporción del target en train y test
random_state	42	Permite reproducibilidad
Target	target_subempleo_horas	Variable objetivo del proyecto
Archivos generados

El notebook genera los siguientes archivos:

../data/split/X_train.csv
../data/split/X_test.csv
../data/split/y_train.csv
../data/split/y_test.csv

Descripción:

Archivo	Descripción
X_train.csv	Variables predictoras para entrenamiento
X_test.csv	Variables predictoras para evaluación
y_train.csv	Target correspondiente al conjunto de entrenamiento
y_test.csv	Target correspondiente al conjunto de prueba
Importancia metodológica

La división train/test debe realizarse antes de la selección estadística de variables, el balanceo de clases y el entrenamiento de modelos.

Esto evita que información del conjunto de prueba influya en decisiones previas al modelamiento. Por ejemplo, si se seleccionaran variables usando todo el dataset antes del split, el modelo estaría indirectamente usando información del test set, lo que generaría data leakage.

Flujo correcto después de este notebook

Después de ejecutar este notebook, el pipeline debe continuar así:

Feature engineering conceptual
↓
Train/test split
↓
Feature selection usando solo X_train e y_train
↓
Modelamiento
↓
Balanceo de clases solo sobre entrenamiento, si aplica
↓
Evaluación final sobre X_test
Relación con las siguientes etapas

Los archivos generados por este notebook serán usados en:

05_feature_selection/selected_variables.ipynb

En esa etapa, la selección de variables debe hacerse únicamente con:

X_train.csv
y_train.csv

Luego, la lista de variables seleccionadas se aplica también a X_test.csv, pero sin usar X_test para decidir qué variables seleccionar.

Consideraciones importantes
No se realiza balanceo de clases en este notebook.
No se entrena ningún modelo.
No se hace selección de variables.
No se calculan métricas de desempeño.
El conjunto de prueba debe permanecer intacto hasta la etapa de evaluación.
La estratificación permite mantener la proporción original entre clases del target.