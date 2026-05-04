# Proyecto Final: Transformación y Modelado de Indicadores Socioeconómicos en Latinoamérica

**Curso:** CC3074 - Minería de Datos  
**Integrantes:** Cristian Túnchez (231359), Dulce Ambrosio (231143), Daniel Chet (231177), Javier Linares (231135)

---

## Semana 1 - Análisis Exploratorio de Datos

### Descripción

En esta primera etapa se llevó a cabo el Análisis Exploratorio de Datos (EDA) siguiendo las fases 1 y 2 del framework CRISP-DM (Comprensión del Negocio y Comprensión de los Datos). Se trabajó con el dataset publicado por la Comisión Económica para América Latina y el Caribe (CEPAL) a través del Observatorio de Desarrollo Digital (ODD), el cual registra el porcentaje de personas usuarias de Internet por grupo etario en 14 países de América Latina y el Caribe, con cobertura temporal de 2000 a 2022.

El trabajo de la semana incluyó:

- **Descripción general del dataset:** análisis de la estructura (870 observaciones x 8 variables), tipos de datos, metadatos y problemas de calidad de datos.
- **Exploración de variables numéricas:** estadísticas descriptivas, distribución global y segmentada por grupo etario, país y año.
- **Exploración de variables categóricas:** tablas de frecuencia, cobertura temporal por país y balanceo entre grupos.
- **Análisis de relaciones:** evolución temporal por grupo etario y país, brechas digitales intergeneracionales, análisis de varianza explicada (eta-cuadrado) y correlaciones.

### Highlights

#### 5.1 Resumen del EDA

| Aspecto                    | Hallazgo                                                                 |
| -------------------------- | ------------------------------------------------------------------------ |
| **Estructura**             | Datos de panel: 14 países x 6 grupos etarios x 21 años (2000-2022)       |
| **Completitud**            | 49.3% del panel completo (870 de 1,764 combinaciones posibles)           |
| **Variable objetivo**      | Una única variable numérica (`value`): porcentaje de uso de Internet     |
| **Variables informativas** | 3 de 8 columnas aportan variabilidad; 4 son constantes/nulas             |
| **Distribución**           | Simétrica, platikurtica, sin outliers IQR                                |
| **Tendencia principal**    | Crecimiento sostenido en todos los segmentos                             |
| **Brecha principal**       | Intergeneracional: diferencia sustancial entre jóvenes y adultos mayores |

#### 5.2 Problemas de Datos para Fases Posteriores

1. **Panel desbalanceado:** La cobertura temporal difiere entre países. Esto complica modelos que asuman series temporales completas.
2. **Comparabilidad limitada del grupo <=17:** La edad de inicio de medición varía entre 4 y 10 años según el país.
3. **Variables constantes:** Deben eliminarse en la fase de preparación de datos (`indicator`, `unit`, `source_id`, `notes_ids`).
4. **Variable temporal como entero:** `Años__ESTANDAR` podría convertirse a datetime para análisis de series temporales.
5. **Composición cambiante:** Los promedios regionales por año reflejan diferentes conjuntos de países.

#### 5.3 Preguntas para Fases Posteriores

- ¿Se puede predecir el uso de Internet futuro a partir de las tendencias históricas?
- ¿Qué factores externos (PIB, educación, políticas públicas) correlacionan con la adopción digital?
- ¿Es posible agrupar países por patrones similares de adopción digital (clustering)?
- ¿La brecha digital intergeneracional se está cerrando o ampliando?
