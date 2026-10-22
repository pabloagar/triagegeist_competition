# IntraSight-ESI

[English](README.md) · **Español**

**Buscar riesgo oculto dentro de una misma categoría de triaje.**

Proyecto de investigación retrospectiva con aprendizaje automático para la competencia Triagegeist. Estudia si es posible ordenar a pacientes de urgencias de una misma categoría de triaje según el riesgo de hospitalización, traslado o fallecimiento en el servicio de urgencias.

[Ver el repositorio público](https://github.com/pabloagar/triagegeist_competition) · [Abrir el notebook en Kaggle](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Leer el informe técnico](https://github.com/pabloagar/triagegeist_competition/blob/main/docs/writeup.md)

**Estado:** prototipo de investigación evaluado retrospectivamente con datos públicos de una encuesta estadounidense. No tiene validación clínica externa ni prospectiva y no debe utilizarse para tomar decisiones asistenciales.

## La pregunta

Los pacientes de una misma categoría de triaje no tienen todos el mismo riesgo. En lugar de predecir la categoría, IntraSight-ESI investiga si los datos disponibles durante el triaje permiten identificar un subgrupo con mayor frecuencia de desenlaces de disposición graves. Priorizar una reevaluación más temprana es una posible aplicación futura; su beneficio no se ha probado.

**Qué significa “ESI-like”:** NHAMCS registra la categoría de inmediatez `IMMEDR`, de cinco niveles, no asignaciones confirmadas de Emergency Severity Index. Los resultados corresponden a esos estratos de la encuesta. El desenlace describe la disposición del paciente, no un deterioro fisiológico inminente.

## Cómo se aborda

- Auditar los datos sintéticos de la competencia antes de interpretar el rendimiento del modelo.
- Usar datos públicos NHAMCS de urgencias de 2015–2019 para desarrollo y validación cruzada, reservando 2022 para una evaluación posterior.
- Comparar LightGBM con regresión logística regularizada.
- Separar las variables disponibles durante el triaje de un control deliberadamente contaminado con información posterior.
- Evaluar dentro de cada estrato, con validación cruzada por años, análisis de subgrupos y umbrales congelados para la muestra de 2022.
- Examinar la calibración como comprobación metodológica; la política de alerta se basa en ordenar puntajes, sin afirmar probabilidades individuales calibradas.

## Resultado retrospectivo principal

El grupo principal es **ESI-like 4**. La evaluación documentada informa:

| Medida | Validación cruzada de desarrollo | Muestra reservada de 2022 |
| --- | --- | --- |
| Proporción de alertas | 5 % con mayor puntaje | 6,2 % con el umbral congelado |
| Enriquecimiento del desenlace respecto de la frecuencia basal del estrato | 5,24× | 5,31×, con intervalo bootstrap del 95 % de 3,36–7,30 |
| Valor predictivo positivo | 14,1 % | Métricas completas de la muestra reservada en el notebook |
| Sensibilidad | 26,2 % | Métricas completas de la muestra reservada en el notebook |

El enriquecimiento compara la frecuencia del desenlace entre los registros señalados con la frecuencia basal del mismo estrato. No estima una mejora en los resultados de los pacientes. La proporción real de alertas de 2022 difiere del objetivo de desarrollo porque se mantuvo fijo el umbral numérico.

![Figura original del notebook que compara el enriquecimiento de validación cruzada y de la muestra de 2022 en los tres estratos ESI-like](assets/retrospective-validation.png)

*Figura original conservada en el notebook. Sus rótulos “ESI 3/4/5” son una abreviación de los estratos IMMEDR de NHAMCS, no asignaciones ESI confirmadas. Las barras muestran intervalos bootstrap del 95 %; ESI-like 5 mantiene carácter exploratorio.*

## Mi aporte

**Dr. Pablo Aguirre Araya, médico general de zona en Arica, Chile.** Formulación de la pregunta clínica, definición de desenlaces y exclusión de variables, auditoría de los datos sintéticos e interpretación de resultados, con apoyo de IA para la implementación y la revisión metodológica.

El aporte reúne el planteamiento clínico y una evaluación auditable, además de la implementación del modelo.

## Reproducibilidad

El repositorio público incluye el notebook de análisis, instrucciones de obtención de datos, auditorías, informe técnico y archivo de citación. El flujo utiliza Python, pandas, NumPy, scikit-learn, LightGBM, SciPy y Matplotlib. El proyecto especifica versiones y semillas aleatorias.

Para inspeccionar o reproducir el análisis, comenzar por las [instrucciones originales de reproducción](https://github.com/pabloagar/triagegeist_competition/blob/main/docs/README-before-bilingual-overview.md#reproduce) y la [guía de datos](https://github.com/pabloagar/triagegeist_competition/blob/main/data/README.md). Los datos sintéticos de la competencia no se redistribuyen.

## Límites y siguientes pasos

NHAMCS es una encuesta estadounidense, no el registro operacional de un hospital local. El desenlace compuesto reúne disposiciones distintas, y la muestra posterior no establece rendimiento en Chile ni en atención real. Las auditorías de subgrupos no resuelven por sí solas la equidad del modelo.

Los siguientes pasos de investigación serían la validación externa, una evaluación prospectiva y comprobar si un flujo de reevaluación mejora la atención sin generar una carga excesiva de alertas. El proyecto aún no ha llegado a esas etapas.

[Repositorio](https://github.com/pabloagar/triagegeist_competition) · [Notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Contacto](mailto:pablo.aguirre.araya@gmail.com)
