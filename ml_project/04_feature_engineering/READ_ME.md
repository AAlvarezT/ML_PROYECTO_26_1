# 04 – Feature Engineering

## Objetivo

Esta carpeta contiene los notebooks de **construcción de variables derivadas** (*feature engineering*) a partir del dataset filtrado de la EPEN 2024. Se crean variables nuevas aplicando reglas conceptuales fijas sobre las variables originales.

El feature engineering conceptual se realiza **antes** del train/test split porque no utiliza información aprendida del conjunto de datos; solo aplica transformaciones deterministas (umbrales, indicadores, proporciones) que no constituyen *data leakage*.

Al finalizar esta etapa, el dataset `epen_features_final.csv` está listo para la división entrenamiento/prueba.

---

## Estructura del Directorio

```
04_feature_engineering/
├── 01_demographic_features.ipynb   ← Variables demográficas (sexo, edad, etnicidad)
├── 02_education_features.ipynb     ← Variables de nivel educativo
├── 03_employment_features.ipynb    ← Variables de condiciones laborales
├── 04_income_features.ipynb        ← Variables de ingresos
├── train_test_split.ipynb          ← División 80/20 estratificada
│
├── demographic_features.ipynb      (versión anterior — conservado como referencia)
├── education_features.ipynb        (versión anterior — conservado como referencia)
├── employment_features.ipynb       (versión anterior — conservado como referencia)
└── income_features.ipynb           (versión anterior — conservado como referencia)
```

---

## Flujo de Datos

```
03_preprocessing/
  └── epen_variable_filtered.csv
            │
            ▼
  01_demographic_features.ipynb
            │
            ▼ epen_fe_demographic.csv
  02_education_features.ipynb
            │
            ▼ epen_fe_education.csv
  03_employment_features.ipynb
            │
            ▼ epen_fe_employment.csv
  04_income_features.ipynb
            │
            ▼ epen_features_final.csv
  train_test_split.ipynb
            │
   ┌────────┴────────┐
   │                 │
X_train.csv      X_test.csv
y_train.csv      y_test.csv
```

---

## Notebooks

### 01_demographic_features.ipynb

**Entrada:** `data/processed/epen_variable_filtered.csv`  
**Salida:** `data/feature_engineering/epen_fe_demographic.csv` + `demographic_features_created.csv`

Construye variables derivadas de características personales del trabajador.

| Feature | Descripción | Variable base |
|:--------|:-----------|:--------------|
| `sexo_mujer` | 1 si el trabajador es mujer | C207 |
| `sexo_hombre` | 1 si el trabajador es hombre | C207 |
| `edad` | Edad en años cumplidos (numérica) | C208 |
| `grupo_edad` | Grupo etario: 14_24 / 25_34 / 35_44 / 45_54 / 55_64 / 65_mas | C208 |
| `joven` | 1 si tiene entre 14 y 29 años | C208 |
| `adulto_mayor` | 1 si tiene 65 años o más | C208 |
| `jefe_hogar` | 1 si es jefe/a del hogar | C203 |
| `conyuge` | 1 si es cónyuge / conviviente | C203 |
| `hijo_hogar` | 1 si es hijo/a del jefe del hogar | C203 |
| `tiene_discapacidad` | 1 si tiene al menos una limitación permanente | C375_1–C375_6 |
| `cantidad_limitaciones` | Número total de limitaciones permanentes (0–6) | C375_1–C375_6 |
| `lengua_materna_indigena` | 1 si la lengua materna es una lengua originaria | C376 |
| `autoidentificacion_indigena` | 1 si se autoidentifica como indígena | C377 |
| `autoidentificacion_afro` | 1 si se autoidentifica como afroperuano/a | C377 |
| `autoidentificacion_mestizo` | 1 si se autoidentifica como mestizo/a | C377 |

---

### 02_education_features.ipynb

**Entrada:** `data/feature_engineering/epen_fe_demographic.csv`  
**Salida:** `data/feature_engineering/epen_fe_education.csv` + `education_features_created.csv`

Construye indicadores del nivel educativo del trabajador.

| Feature | Descripción | Variable base |
|:--------|:-----------|:--------------|
| `nivel_educativo_ord` | Nivel educativo como variable ordinal (1–12) | C366 |
| `educacion_basica_o_menos` | 1 si nivel ≤ 5 (hasta secundaria incompleta) | C366 |
| `secundaria_completa` | 1 si nivel == 6 | C366 |
| `superior_incompleta` | 1 si nivel en {7, 9} | C366 |
| `superior_completa` | 1 si nivel en {8, 10, 11, 12} | C366 |
| `universitaria_completa_o_mas` | 1 si nivel en {10, 11, 12} | C366 |
| `educacion_superior` | 1 si nivel ≥ 7 | C366 |
| `brecha_educativa_baja` | 1 si nivel ≤ 4 (hasta primaria completa) | C366 |
| `grupo_educativo` | Categoría textual del nivel educativo | C366 |

---

### 03_employment_features.ipynb

**Entrada:** `data/feature_engineering/epen_fe_education.csv`  
**Salida:** `data/feature_engineering/epen_fe_employment.csv` + `employment_features_created.csv`

Construye variables de condiciones laborales. Estas son las variables **más directamente relacionadas** con el subempleo por insuficiencia de horas.

> ⚠️ `C333` y `C334` (horas habituales deseadas) **NO se usan** — son leakage directo del target.

**Categoría ocupacional (C310):**

| Feature | Descripción |
|:--------|:-----------|
| `trabajador_independiente` | 1 si es trabajador independiente |
| `trabajador_dependiente` | 1 si es empleado/obrero dependiente |
| `empleador` | 1 si es empleador/patrono |
| `trabajador_hogar` | 1 si es trabajador del hogar |
| `trabajador_familiar_no_remunerado` | 1 si es TFNR |

**Sector y formalidad:**

| Feature | Descripción |
|:--------|:-----------|
| `sector_publico` | 1 si trabaja en sector público |
| `sector_privado` | 1 si trabaja en empresa privada |
| `empresa_registrada_sunat` | 1 si empresa tiene RUC activo/suspendido |
| `empresa_no_registrada_sunat` | 1 si empresa no está registrada en SUNAT |
| `lleva_libros_contables` | 1 si la empresa lleva libros contables |
| `posible_informalidad` | 1 si no registrada en SUNAT y sin libros contables |
| `empresa_pequena` | 1 si empresa tiene 1–10 trabajadores |
| `empresa_mediana_grande` | 1 si empresa tiene 11+ trabajadores |
| `busca_otro_empleo` | 1 si el trabajador busca otro empleo |

**Horas trabajadas:**

| Feature | Descripción |
|:--------|:-----------|
| `horas_totales` | Total horas trabajadas a la semana |
| `horas_principal` | Horas en empleo principal |
| `horas_secundaria` | Horas en empleo secundario |
| `trabaja_menos_35h` | 1 si trabaja menos de 35 horas/semana |
| `trabaja_mas_48h` | 1 si trabaja más de 48 horas/semana |
| `sobrejornada` | 1 si trabaja más de 60 horas/semana |
| `brecha_horas_normal` | Diferencia entre horas deseadas y horas actuales |

**Protección social:**

| Feature | Descripción |
|:--------|:-----------|
| `tiene_seguro_salud` | 1 si tiene algún seguro de salud |
| `tiene_essalud` | 1 si tiene EsSalud |
| `tiene_sis` | 1 si tiene SIS |
| `tiene_pension` | 1 si tiene AFP o SNP/ONP |
| `tiene_afp` | 1 si tiene AFP |
| `tiene_snp` | 1 si tiene SNP/ONP |
| `proteccion_social_completa` | 1 si tiene seguro de salud Y pensión |

---

### 04_income_features.ipynb

**Entrada:** `data/feature_engineering/epen_fe_employment.csv`  
**Salida:** `data/feature_engineering/epen_features_final.csv` + `income_features_created.csv`

Construye variables de nivel de ingresos. Es el **último notebook de feature engineering**.

| Feature | Descripción |
|:--------|:-----------|
| `ingreso_total` | Ingreso total del trabajador (INGTOT) |
| `ingreso_principal` | Ingreso del empleo principal (INGTOTP o I339_1) |
| `ingreso_laboral` | Ingreso laboral semanal (ingtrabw) |
| `ingreso_total_log` | Log del ingreso total (log1p) |
| `ingreso_principal_log` | Log del ingreso principal |
| `ingreso_laboral_log` | Log del ingreso laboral |
| `ingreso_por_hora` | Ingreso laboral / (horas_totales × 4.33) |
| `ingreso_por_hora_log` | Log del ingreso por hora |
| `ingreso_secundario_total` | Suma de ingresos no laborales |
| `tiene_ingreso_secundario` | 1 si tiene ingresos secundarios |
| `proporcion_ingreso_principal` | ingreso_principal / ingreso_total |
| `dependencia_ingreso_principal` | 1 si proporción ≥ 0.80 |
| `bajo_ingreso_relativo` | 1 si ingreso total < percentil 25 |
| `alto_ingreso_relativo` | 1 si ingreso total > percentil 75 |
| `ingreso_cero` | 1 si ingreso total == 0 |
| `ingreso_outlier` | 1 si es outlier en ingreso total o laboral |

---

### train_test_split.ipynb

**Entrada:** `data/feature_engineering/epen_features_final.csv`  
**Salida:** `data/split/X_train.csv`, `X_test.csv`, `y_train.csv`, `y_test.csv`

| Parámetro | Valor |
|:----------|:------|
| Test size | 20% |
| Estratificación | `target_subempleo_horas` |
| random_state | 42 |

---

## Archivos Generados

| Archivo | Directorio | Descripción |
|:--------|:-----------|:------------|
| `epen_fe_demographic.csv` | `data/feature_engineering/` | Dataset + features demográficas |
| `demographic_features_created.csv` | `data/feature_engineering/` | Reporte de features demográficas |
| `epen_fe_education.csv` | `data/feature_engineering/` | Dataset + features educativas |
| `education_features_created.csv` | `data/feature_engineering/` | Reporte de features educativas |
| `epen_fe_employment.csv` | `data/feature_engineering/` | Dataset + features laborales |
| `employment_features_created.csv` | `data/feature_engineering/` | Reporte de features laborales |
| `epen_features_final.csv` | `data/feature_engineering/` | Dataset final con TODAS las features |
| `income_features_created.csv` | `data/feature_engineering/` | Reporte de features de ingresos |
| `X_train.csv` | `data/split/` | Features de entrenamiento |
| `X_test.csv` | `data/split/` | Features de prueba |
| `y_train.csv` | `data/split/` | Target de entrenamiento |
| `y_test.csv` | `data/split/` | Target de prueba |

---

## Orden de Ejecución

```
1. 01_demographic_features.ipynb
2. 02_education_features.ipynb
3. 03_employment_features.ipynb
4. 04_income_features.ipynb
5. train_test_split.ipynb
```

---

## Decisiones Metodológicas

### ¿Por qué hacer feature engineering antes del split?

El feature engineering de este proyecto aplica **reglas deterministas** (ej. `edad >= 65 → adulto_mayor = 1`). Estas reglas no "aprenden" del conjunto de datos, por lo que aplicarlas antes del split **no introduce data leakage**. Si se hiciera después, habría que aplicar las mismas reglas por separado a train y test, lo que sería equivalente pero más complejo.

### ¿Por qué stratify en el split?

El target presenta desbalance de clases (~25% subempleados, ~75% no subempleados). La estratificación garantiza que ambos conjuntos (train y test) tengan la misma proporción del target, lo que evita que uno quede sesgado.

### ¿Por qué feature selection va DESPUÉS del split?

Los métodos estadísticos de selección de variables (correlación de Pearson, importancia de variables, tests de hipótesis) se calculan **sobre los datos**. Si se calculan antes del split, el modelo podría aprender patrones que existen en el conjunto de prueba → data leakage estadístico.

Por eso, la feature selection estadística se realiza únicamente sobre `X_train` en la carpeta `05_feature_selection/`.

### Variables de leakage excluidas

| Variable | Razón |
|:---------|:------|
| `P209H` | Identificador del hogar — no es predictor |
| `C333` | Horas habituales que desea trabajar — parte directa del cálculo del target |
| `C334` | Horas habituales que desea trabajar — parte directa del cálculo del target |
| `fa_son24` | Factor de expansión — no es una característica del trabajador |
