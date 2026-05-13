# Proyecto Final: Transformación y Modelado de Indicadores Socioeconómicos en Latinoamérica

**Curso:** CC3074 - Minería de Datos  
**Integrantes:** Cristian Túnchez (231359), Dulce Ambrosio (231143), Daniel Chet (231177), Javier Linares (231135)

## Semana 1 - Análisis Exploratorio de Datos

### Descripción

En esta primera etapa se llevó a cabo el Análisis Exploratorio de Datos (EDA) siguiendo las fases 1 y 2 del framework CRISP-DM (Comprensión del Negocio y Comprensión de los Datos). Se trabajó con el dataset publicado por la Comisión Económica para América Latina y el Caribe (CEPAL) a través del Observatorio de Desarrollo Digital (ODD), el cual registra el porcentaje de personas usuarias de Internet por grupo etario en 14 países de América Latina y el Caribe, con cobertura temporal de 2000 a 2022.

El trabajo de la semana incluyó:

- **Descripción general del dataset:** análisis de la estructura (870 observaciones x 8 variables), tipos de datos, metadatos y problemas de calidad de datos.
- **Exploración de variables numéricas:** estadísticas descriptivas, distribución global y segmentada por grupo etario, país y año.
- **Exploración de variables categóricas:** tablas de frecuencia, cobertura temporal por país y balanceo entre grupos.
- **Análisis de relaciones:** evolución temporal por grupo etario y país, brechas digitales intergeneracionales, análisis de varianza explicada (eta-cuadrado) y correlaciones.

### Highlights

#### Resumen del EDA

| Aspecto                    | Hallazgo                                                                 |
| -------------------------- | ------------------------------------------------------------------------ |
| **Estructura**             | Datos de panel: 14 países x 6 grupos etarios x 21 años (2000-2022)       |
| **Completitud**            | 49.3% del panel completo (870 de 1,764 combinaciones posibles)           |
| **Variable objetivo**      | Una única variable numérica (`value`): porcentaje de uso de Internet     |
| **Variables informativas** | 3 de 8 columnas aportan variabilidad; 4 son constantes/nulas             |
| **Distribución**           | Simétrica, platikurtica, sin outliers IQR                                |
| **Tendencia principal**    | Crecimiento sostenido en todos los segmentos                             |
| **Brecha principal**       | Intergeneracional: diferencia sustancial entre jóvenes y adultos mayores |

#### Problemas de Datos para Fases Posteriores

1. **Panel desbalanceado:** La cobertura temporal difiere entre países. Esto complica modelos que asuman series temporales completas.
2. **Comparabilidad limitada del grupo <=17:** La edad de inicio de medición varía entre 4 y 10 años según el país.
3. **Variables constantes:** Deben eliminarse en la fase de preparación de datos (`indicator`, `unit`, `source_id`, `notes_ids`).
4. **Variable temporal como entero:** `Años__ESTANDAR` podría convertirse a datetime para análisis de series temporales.
5. **Composición cambiante:** Los promedios regionales por año reflejan diferentes conjuntos de países.

#### Preguntas para Fases Posteriores

- ¿Se puede predecir el uso de Internet futuro a partir de las tendencias históricas?
- ¿Qué factores externos (PIB, educación, políticas públicas) correlacionan con la adopción digital?
- ¿Es posible agrupar países por patrones similares de adopción digital (clustering)?
- ¿La brecha digital intergeneracional se está cerrando o ampliando?

## Semana 2 - Transformación del Conjunto de Datos

### Descripción

En esta segunda etapa se ejecutó la **Fase 3 completa de CRISP-DM (Preparación de Datos)** sobre el dataset de CEPAL analizado en la Semana 1, dejando el conjunto de datos listo para la Fase de Modelado de la Semana 3. Partiendo de los 870 registros × 8 columnas en formato largo, se construyó un conjunto analítico modelable de **175 filas × 60 columnas** exportado como `data_transformado.csv`.

El trabajo de la semana incluyó:

- **Limpieza preliminar:** eliminación de 4 columnas constantes/nulas (`indicator`, `unit`, `source_id`, `notes_ids`) y renombrado de columnas a snake_case en español.
- **Pivoteo long → wide:** transformación de los 6 grupos etarios en 6 indicadores numéricos independientes (`pct_uso_*`).
- **Manejo de valores faltantes:** completado del grid cartesiano país × año e imputación por interpolación lineal intra-país (`limit_area='inside'`), con NaN residuales documentados.
- **Detección de outliers:** diagnóstico doble (IQR + Z-score) sin aplicar correcciones, justificado por la acotación física `[0, 100]` y el carácter informativo de los extremos.
- **Ingeniería de variables:** generación de 17 nuevas features (3 brechas intergeneracionales, 6 lags temporales, 6 deltas anuales, 2 variables temporales).
- **Discretización:** creación de la variable ordinal `categoria_adopcion` con 4 niveles (Inicial, Emergente, Avanzada, Madura).
- **Codificación:** one-hot encoding de `pais` con `drop_first=True` (13 dummies).
- **Estandarización:** aplicación de `StandardScaler` sobre 21 features numéricas (sufijo `_z`).
- **Análisis de redundancia:** identificación documentada (no destructiva) de pares con `|r| > 0.95`.
- **Exportación:** consolidación en un único CSV comprehensivo de 60 columnas.

### Highlights

#### Pasos aplicados y su justificación

| #   | Transformación                                              | Sección | Forma resultante     |
| --- | ----------------------------------------------------------- | ------- | -------------------- |
| 1   | Eliminación de 4 columnas constantes/nulas                  | 1       | 870 × 4              |
| 2   | Renombrado a snake_case en español                          | 1       | 870 × 4              |
| 3   | Pivote `long → wide`                                        | 2       | 145 × 8              |
| 4   | Renombrado de los 6 indicadores a `pct_uso_*`               | 3       | 145 × 8              |
| 5   | Reindex al grid cartesiano (`país × año`)                   | 4.2     | 322 × 8 (con NaN)    |
| 6   | Interpolación lineal intra-país                             | 4.4     | 322 × 8 (NaN reduc.) |
| 7   | Drop de filas con todos los indicadores NaN                 | 4.4     | 175 × 8              |
| 8   | Diagnóstico de outliers (IQR + Z-score, sin corrección)     | 6       | 175 × 8              |
| 9   | Brechas intergeneracionales (3 features)                    | 7.1     | 175 × 11             |
| 10  | Lags temporales (6 features)                                | 7.2     | 175 × 17             |
| 11  | Deltas anuales (6 features)                                 | 7.3     | 175 × 23             |
| 12  | Variables temporales (2 features)                           | 7.4     | 175 × 25             |
| 13  | Discretización en `categoria_adopcion`                      | 8       | 175 × 26             |
| 14  | One-hot encoding de país (13 dummies, almacenados aparte)   | 9       | dummies (175 × 13)   |
| 15  | StandardScaler sobre 21 features (sufijo `_z`)              | 10      | 175 × 47             |
| 16  | Análisis de redundancia (no destructivo)                    | 11      | 175 × 47             |
| 17  | Concatenación + exportación a CSV (`data_transformado.csv`) | 13      | **175 × 60**         |

#### Decisiones documentadas y aplicadas

Durante esta semana, la Fase 3 del framework CRISP-DM se cubre íntegramente. Las decisiones técnicas relevantes:

- **Outliers no corregidos:** dominio físico [0, 100] acota los valores; los "outliers" detectados son señal (heterogeneidad inter-país), no ruido.
- **NaN en lags/deltas conservados:** el primer año por país es genuinamente no-derivable; la fase de modelado decidirá según algoritmo.
- **Redundancia documentada, no eliminada:** distintos algoritmos toleran multicolinealidad de forma distinta; reportar en lugar de eliminar.
- **StandardScaler sobre MinMaxScaler:** distribuciones aproximadamente normales, preserva información de extremos.
- **One-hot con `drop_first=True`:** evita colinealidad perfecta para modelos lineales.
- **CSV único comprehensivo:** un solo artefacto con todas las representaciones (raw, engineered, encoded, scaled).

#### Único elemento postergado a la Semana 3

- **Partición train / validation / test:** depende del algoritmo y la estrategia de validación temporal (split aleatorio vs split por año vs validación cruzada por país). Es una decisión que pertenece propiamente a la fase de Modelado.

## Semana 3 - Modelado Inicial: Construcción y Comparación de Modelos Base

### Descripción

En esta tercera etapa se ejecutó la **porción inicial de la Fase 4 (Modelado) del marco CRISP-DM** sobre el conjunto `data_transformado.csv` (175 filas × 60 columnas) construido en la Semana 2. Se planteó el problema como una **regresión supervisada** sobre la variable objetivo `pct_uso_total`, bajo un **enfoque predictivo (lag-based)** que solo admite información del año anterior y atributos invariantes, evitando leakage estructural. El dataset modelable quedó en 161 filas × 22 predictoras tras eliminar las 14 filas correspondientes al primer año observado por país.

El trabajo de la semana incluyó:

- **Formulación del problema:** definición como regresión supervisada con `pct_uso_total` como target y justificación del enfoque lag-based.
- **Selección de features:** identificación explícita de columnas prohibidas por leakage (29 descartadas) y construcción del conjunto admisible de 22 predictoras.
- **División train/test:** split estratificado por cuartiles del target (`pd.qcut(y, q=4)`), con verificación de balance entre train y test.
- **Definición de modelos:** entrenamiento de 5 modelos de tres familias distintas (lineal, basada en distancia, basada en árboles), con escalado encapsulado en `Pipeline` para evitar leakage.
- **Entrenamiento y evaluación:** cálculo de MAE, RMSE, R² y MAPE en train y test para los cinco modelos.
- **Comparación y diagnóstico de overfit:** análisis de gaps train-test.
- **Diagnóstico visual:** predicho vs real, residuales vs predicho, histograma y Q-Q plot del mejor modelo.
- **Importancia de features:** lectura conjunta de coeficientes lineales y `feature_importances_` de modelos tree-based.
- **Análisis de errores:** descomposición por país, por década y observaciones con mayor error absoluto.

### Highlights

#### Hallazgos centrales del modelado inicial

- **El problema es altamente predecible bajo enfoque lag-based**: cuatro de los cinco modelos superan R² = 0.98 en test. El componente autorregresivo (el valor del año anterior) carga con la mayor parte de la señal.
- **Ranking del experimento**: Ridge (R² = 0.9919) > Linear Regression (R² = 0.9916) > Random Forest (R² = 0.9889) > Decision Tree (R² = 0.9821) > KNN (R² = 0.8882). Ridge y Linear son prácticamente indistinguibles entre sí; Random Forest queda muy cerca con una arquitectura completamente distinta, lo cual da robustez al hallazgo.
- **Linealidad confirmada**: la cercanía entre Linear y Ridge, junto con el R² casi máximo de ambos, sugiere que la relación entre los lags y el target es esencialmente lineal. La regularización L2 aporta una mejora marginal al estabilizar coeficientes inflados por multicolinearidad.
- **Overfit claro en Decision Tree**: R² train = 1.0000 con R² test = 0.9821 (gap 0.018). El árbol único sin restricciones memoriza el train. Random Forest, al promediar 100 árboles, reduce drásticamente este efecto (gap 0.009).
- **KNN como modelo no apto** para este problema: la distancia euclídea en un espacio mixto (5 lags continuos + 13 dummies binarias + 4 features adicionales) produce vecindarios poco informativos. El gap R² (0.048) y el MAE en test (6.05) lo descartan claramente frente a los demás.
- **Importancia concentrada en dos predictores**: `pct_uso_total_lag1` (62-75 % en árboles) y `pct_uso_26_50_lag1` (14-24 %) concentran casi toda la información. Las dummies de país y las variables temporales aportan poco, indicando que el lag del propio target ya absorbe la heterogeneidad geográfica y temporal.
- **Sesgo de sobreestimación moderado**: residual medio en test = −0.36 (sobreestima ~0.4 puntos porcentuales en promedio). El sesgo aparece en países con saltos abruptos en su trayectoria reciente (Bolivia 2014, Ecuador 2019, Colombia 2022).

#### Limitaciones del experimento

1. **Un único split** introduce varianza en las métricas: con otra semilla el orden Ridge/Linear/RF podría intercambiarse dado lo cercano de sus valores.
2. **Hiperparámetros por defecto**: ningún modelo se entrenó con búsqueda de parámetros; los valores reportados son una cota inferior del rendimiento alcanzable.
3. **Dependencia temporal ignorada en el split**: el split aleatorio mezcla años recientes y antiguos entre train y test. Una validación temporal complementaría el análisis.
4. **Tamaño muestral pequeño** (161 filas, 22 predictoras): los modelos de alta capacidad (DT, KNN) están en desventaja estructural frente a los lineales regularizados.
