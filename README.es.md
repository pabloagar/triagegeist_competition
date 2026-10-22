# IntraSight-ESI

[Read in English](README.md) · **Español**

**¿Pueden tener riesgos distintos los pacientes de una misma categoría de triaje?**

Investigación retrospectiva con machine learning y datos públicos de urgencias del NHAMCS, desarrollada para Triagegeist. El desenlace es hospitalización, traslado o muerte en urgencias; no deterioro fisiológico inminente.

[Explorar el notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Metodología](docs/writeup.md) · [Reproducir el análisis](docs/README-before-bilingual-overview.md#reproduce)

**Prototipo de investigación.** Sin validación clínica externa ni prospectiva y sin beneficio asistencial demostrado.

## El resultado, en contexto

![Figura original del notebook que compara enriquecimiento en validación cruzada y reserva de 2022 para tres estratos del NHAMCS](assets/retrospective-validation.png)

*Figura sin modificaciones. “ESI 3/4/5” representa estratos IMMEDR del NHAMCS, no asignaciones ESI confirmadas. Barras: intervalos bootstrap del 95%; el estrato 5 es exploratorio.*

En **ESI-like 4**, el subgrupo señalado tuvo una tasa del desenlace **5,31 veces** la basal del estrato en la reserva de 2022 (intervalo bootstrap del 95%: **3,36–7,30**). Es enriquecimiento del desenlace, no una mejora de resultados clínicos.

| Medida | Validación cruzada de desarrollo | Reserva de 2022 |
| --- | --- | --- |
| Proporción señalada | 5% con mayor puntuación | 6,2% con el umbral congelado |
| Enriquecimiento del desenlace | 5,24× | 5,31× |
| Valor predictivo positivo | 14,1% | Métricas completas en notebook |
| Sensibilidad | 26,2% | Métricas completas en notebook |

El umbral numérico se congeló antes de evaluar la reserva; por eso la proporción señalada difiere del objetivo de desarrollo.

## Cómo se evaluó

- Desarrollo: NHAMCS 2015–2019, con validación cruzada por año; reserva de 2022.
- LightGBM frente a regresión logística regularizada.
- Auditoría de datos sintéticos, restricción a variables disponibles en triaje y un control con fuga deliberada de información.
- Revisión por subgrupos, umbrales congelados y comprobaciones de calibración. La política ordena puntuaciones; no afirma probabilidades individuales calibradas.

La [guía de datos](data/README.md), [metodología](docs/writeup.md), el notebook y las [instrucciones de reproducción](docs/README-before-bilingual-overview.md#reproduce) documentan el proceso. Usa Python, pandas, NumPy, scikit-learn, LightGBM, SciPy y Matplotlib. Los datos sintéticos del concurso no se redistribuyen.

## Límites y próximos pasos

NHAMCS es una encuesta estadounidense. IMMEDR se usa como aproximación para esta investigación y el desenlace compuesto reúne eventos distintos. La reserva temporal no establece rendimiento en Chile, atención real ni un hospital externo. Los análisis por subgrupos no resuelven la equidad.

Quedan pendientes la validación externa, la evaluación prospectiva y estudiar un flujo de reevaluación. El prototipo no debe orientar decisiones asistenciales.

## Mi aporte

Pregunta clínica, definición del desenlace, exclusión de variables, auditoría de datos sintéticos e interpretación, con implementación y revisión metodológica asistidas por IA. **Pablo Aguirre Araya, médico en Chile.**

[Notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Contacto](mailto:pablo.aguirre.araya@gmail.com)
