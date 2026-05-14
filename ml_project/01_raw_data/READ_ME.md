# 01_raw_data

Esta carpeta contiene los archivos originales utilizados como insumo para el proyecto de Machine Learning basado en la Encuesta Permanente de Empleo Nacional 2024.

## Contenido de la carpeta

Los archivos incluidos en esta carpeta son:

```text
250806 EPEN2024.xlsx
Diccionario de datos EPEN 2024.pdf
```

## Descripción de los archivos
250806 EPEN2024.xlsx
```text
Archivo principal de datos de la Encuesta Permanente de Empleo Nacional 2024. Contiene información individual sobre características del hogar, condición laboral, empleo, ingresos, educación, salud, pensiones y otras variables sociodemográficas.

Esta base será utilizada como fuente inicial para el análisis exploratorio, la limpieza de datos, la construcción de variables y el entrenamiento de modelos de clasificación.
```

Diccionario de datos EPEN 2024.pdf
```text
Documento que describe el significado, formato, tamaño, rango y codificación de las variables disponibles en la base de datos. El diccionario indica que el archivo corresponde a características de los miembros del hogar, empleo e ingreso, e incluye variables como sexo (C207), edad (C208), condición de actividad laboral (OCUP300), nivel educativo (C366), ingresos (INGTOT, INGTOTP, INGTRABW) y condición de residente habitual (RESIDENT).

También incluye variables relevantes para el proyecto, como P209H, que identifica si la persona tuvo voluntad y disponibilidad para trabajar más horas, y OCUP300, que clasifica la condición de actividad en ocupado, desocupado abierto, desocupado oculto e inactivo pleno.
```