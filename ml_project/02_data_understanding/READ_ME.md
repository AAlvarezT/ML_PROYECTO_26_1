# 02_data_understanding

Esta carpeta contiene los notebooks dedicados a la comprensión inicial de la base de datos de la Encuesta Permanente de Empleo Nacional 2024 (EPEN). Su objetivo es revisar la estructura del dataset, entender el diccionario de variables, explorar patrones iniciales y definir formalmente la variable objetivo del proyecto.

Esta etapa no busca entrenar modelos ni realizar transformaciones definitivas de preprocesamiento. Su función principal es construir una base metodológica clara para las etapas posteriores del pipeline de Machine Learning.

---

## Objetivo de la carpeta

La carpeta `02_data_understanding` tiene como propósito responder las siguientes preguntas:

- ¿Qué contiene la base original de la EPEN 2024?
- ¿Cuántas filas y columnas tiene el dataset?
- ¿Qué variables son relevantes para el problema?
- ¿Qué variables deben usarse como filtros?
- ¿Cuál será la variable objetivo del proyecto?
- ¿Cuál es la distribución inicial del target?
- ¿Qué variables pueden generar data leakage?
- ¿Qué patrones iniciales se observan en variables demográficas, laborales, educativas e ingresos?

El proyecto busca predecir **subempleo por insuficiencia de horas** en personas ocupadas de Lima Metropolitana. Para ello, se utiliza como variable objetivo `P209H`, recodificada como `target_subempleo_horas`.

---

## Archivos de la carpeta

```text
02_data_understanding/
│
├── 00_original_dataset_overview.ipynb
├── 01_data_dictionary_review.ipynb
├── 02_exploratory_analysis.ipynb
└── 03_target_definition.ipynb
```

00_original_dataset_overview.ipynb
```text
Este notebook realiza una primera revisión de la base original sin modificarla. Su objetivo es cargar el archivo fuente de la EPEN 2024, revisar sus dimensiones generales y generar un snapshot inicial del dataset.

Contenido principal
Carga del archivo original 250806 EPEN2024.xlsx.
Revisión del número de filas y columnas.
Visualización inicial de las primeras observaciones.
Revisión preliminar de variables clave como REGION, C208 y P209H.
Conteo preliminar del universo analítico.
Generación de un snapshot del dataset raw.
Salida generada
data/raw/epen2024_raw_snapshot.csv

Este archivo se guarda como una copia del dataset original en formato CSV para facilitar su uso en los notebooks posteriores sin alterar el Excel original.

Rol dentro del proyecto

Este notebook sirve como punto de partida técnico del proyecto. Permite verificar que la base fue cargada correctamente y deja disponible una versión reproducible del dataset original.
```

01_data_dictionary_review.ipynb
```text
Este notebook documenta la revisión del diccionario de datos de la EPEN 2024. Su objetivo es identificar las variables relevantes para el problema de Machine Learning y clasificarlas según su posible uso dentro del modelo.

Contenido principal
Revisión de variables del diccionario asociadas a:
Diseño muestral.
Características demográficas.
Condición de actividad.
Empleo.
Horas trabajadas.
Ingresos.
Educación.
Salud.
Pensiones.
Discapacidad.
Etnicidad.
Identificación de variables predictoras candidatas.
Identificación de variables de filtro.
Identificación de variables con riesgo de data leakage.
Revisión de códigos especiales de valores faltantes.
Variables importantes revisadas
Variable	Rol en el proyecto
RESIDENT	Filtro para residentes habituales
C208	Edad; filtro y predictor
OCUP300	Filtro para personas ocupadas
P209H	Variable base para construir el target
C333	Variable excluida por leakage
C334	Variable excluida por leakage
C207	Sexo
C366	Nivel educativo
C310	Categoría ocupacional
C312	Registro del negocio en SUNAT
C318_T, whoraT	Horas trabajadas
INGTOT, INGTOTP, ingtrabw	Ingresos
SEGURO1, C364_*	Protección social
Rol dentro del proyecto

Este notebook establece las decisiones iniciales sobre qué variables pueden ser usadas y cuáles deben excluirse. Es especialmente importante para evitar errores metodológicos como usar variables que revelen directamente el target.
```

02_exploratory_analysis.ipynb
```text
Este notebook desarrolla el análisis exploratorio de datos del universo analítico. Su objetivo es entender la distribución del target y observar relaciones iniciales entre variables relevantes y el subempleo por insuficiencia de horas.

Contenido principal
Distribución del target target_subempleo_horas.
Análisis de variables numéricas clave.
Análisis de variables categóricas.
Comparación de edad y horas trabajadas según el target.
Tasa de subempleo por nivel educativo.
Tasa de subempleo por categoría ocupacional.
Matriz de correlación entre variables numéricas.
Identificación de patrones iniciales relevantes para el modelamiento.
Hallazgos principales

El análisis exploratorio identifica un universo analítico de aproximadamente 24,054 trabajadores ocupados de Lima Metropolitana, considerando residentes habituales, personas de 14 años a más, ocupados y con respuesta válida en P209H.

La distribución del target muestra un desbalance moderado:

Clase	Significado	Proporción aproximada
0	No subempleado por horas	75.1%
1	Subempleado por horas	24.9%

Además, el análisis muestra que los trabajadores clasificados como subempleados por horas tienden a concentrarse en rangos menores de horas trabajadas. También se observan relaciones preliminares con variables como educación, categoría ocupacional, ingresos y protección social.

Rol dentro del proyecto

Este notebook permite entender el comportamiento inicial de la base antes del preprocesamiento y modelamiento. Sus resultados ayudan a justificar la necesidad de aplicar balanceo de clases y a priorizar variables para las etapas posteriores.
```
03_target_definition.ipynb
```text
Este notebook define formalmente la variable objetivo del proyecto. Aunque parte del análisis del target se explora en el EDA, este archivo deja cerrada metodológicamente la construcción del target.

Contenido principal
Justificación de la problemática de subempleo por insuficiencia de horas.
Definición del universo analítico.
Construcción de la variable target_subempleo_horas.
Revisión de la distribución del target.
Identificación de variables con riesgo de data leakage.
Guardado del dataset filtrado con el target definido.
Universo analítico

El target se construye únicamente sobre personas que cumplen las siguientes condiciones:

Criterio	Variable	Condición
Residente habitual	RESIDENT	== 1
Edad mínima	C208	>= 14
Ocupado	OCUP300	== 1
Respuesta válida	P209H	1 o 2
Definición del target

La variable P209H se recodifica de la siguiente manera:

Valor original de P209H	Significado	Target final
1	Tiene voluntad y disponibilidad para trabajar más horas	1
2	No tiene voluntad y disponibilidad para trabajar más horas	0

La variable final se denomina:

target_subempleo_horas
Variables excluidas por data leakage

Las siguientes variables no deben usarse como predictoras:

Variable	Motivo
P209H	Es la variable usada para construir el target
C333	Pregunta directamente si la persona quería trabajar más horas
C334	Pregunta directamente si la persona estaba disponible para trabajar más horas
Salida generada
data/processed/epen_target_defined.csv
```

Este archivo sirve como insumo para la etapa de preprocesamiento.

Orden recomendado de ejecución

Los notebooks deben ejecutarse en el siguiente orden:

00_original_dataset_overview.ipynb
↓
01_data_dictionary_review.ipynb
↓
02_exploratory_analysis.ipynb
↓
03_target_definition.ipynb

Aunque parte del análisis del target aparece en el EDA, el notebook 03_target_definition.ipynb es el que formaliza la variable objetivo y genera el archivo que será usado en las siguientes etapas.

Archivos generados por esta etapa
Notebook	Archivo generado	Uso posterior
00_original_dataset_overview.ipynb	data/raw/epen2024_raw_snapshot.csv	Base raw en CSV para análisis posteriores
03_target_definition.ipynb	data/processed/epen_target_defined.csv	Base filtrada con target definido para preprocesamiento
Decisiones metodológicas tomadas

En esta etapa se definieron las siguientes decisiones:

El problema a modelar será el subempleo por insuficiencia de horas, no la desocupación general.
El target será target_subempleo_horas, construido a partir de P209H.
El universo analítico estará compuesto por personas residentes habituales, de 14 años a más, ocupadas y con respuesta válida en P209H.
Las variables P209H, C333 y C334 serán excluidas de los predictores por data leakage.
La clase positiva representa aproximadamente una cuarta parte del universo analítico, por lo que se considera un desbalance moderado.
El factor de expansión fa_son24 no se utilizará directamente como predictor en el modelo base.
Las variables sensibles, como etnicidad o discapacidad, deberán tratarse con cuidado metodológico y ético si se incluyen en el modelo.
Relación con las siguientes carpetas

La carpeta 02_data_understanding alimenta directamente a la etapa de preprocesamiento ubicada en:

03_preprocessing/

En particular, el archivo:

data/processed/epen_target_defined.csv

será usado para:

Manejo de valores faltantes.
Tratamiento de outliers.
Filtrado de variables.
Codificación de variables categóricas.
División entre entrenamiento y prueba.