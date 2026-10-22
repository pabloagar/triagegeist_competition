# IntraSight-ESI

**English** · [Leer en español](README.es.md)

**Can patients in the same triage stratum have meaningfully different risks?**

Retrospective machine learning research on public NHAMCS emergency department data, developed for the Triagegeist competition. The target is hospitalization, transfer or death in the emergency department—not imminent physiological deterioration.

[Explore the notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Methods](docs/writeup.md) · [Reproduce the analysis](docs/README-before-bilingual-overview.md#reproduce)

**Research prototype.** No external or prospective clinical validation; no demonstrated benefit to patient care.

## The result, in context

![Original notebook figure comparing cross-validation and 2022 holdout enrichment in three NHAMCS triage strata](assets/retrospective-validation.png)

*Unmodified notebook figure. “ESI 3/4/5” means NHAMCS IMMEDR survey strata, not confirmed ESI assignments. Error bars: 95% bootstrap intervals; stratum 5 is exploratory.*

In **ESI-like 4**, the flagged subgroup had **5.31×** the stratum's baseline outcome rate in the 2022 holdout (95% bootstrap interval **3.36–7.30**). This is outcome enrichment, not an improvement in clinical outcomes.

| Measure | Development cross-validation | 2022 holdout |
| --- | --- | --- |
| Alert burden | Top 5% by score | 6.2% with the frozen threshold |
| Outcome enrichment | 5.24× | 5.31× |
| Positive predictive value | 14.1% | Full metrics in notebook |
| Recall | 26.2% | Full metrics in notebook |

The numeric threshold was frozen before the holdout; the realized alert burden therefore differs from the development target.

## How it was evaluated

- Development: NHAMCS 2015–2019, with year-blocked cross-validation; 2022 held out.
- LightGBM compared with regularized logistic regression.
- Synthetic-data audit, triage-available feature restrictions and a deliberately leaky control.
- Subgroup checks, frozen thresholds and calibration checks. The alert policy ranks scores; it does not claim individually calibrated probabilities.

The [data guide](data/README.md), [technical writeup](docs/writeup.md), notebook and [reproduction instructions](docs/README-before-bilingual-overview.md#reproduce) document the workflow. Python, pandas, NumPy, scikit-learn, LightGBM, SciPy and Matplotlib are used. Competition synthetic data are not redistributed.

## Limits and next steps

NHAMCS is a US survey. Its IMMEDR categories are proxies for this research, and its composite disposition outcome combines distinct events. The later-year holdout does not establish performance in Chile, live care or an external hospital. Subgroup checks do not settle fairness questions.

External validation, prospective evaluation and assessment of a reassessment workflow remain future research. Do not use this prototype for care decisions.

## My contribution

Clinical question, outcome definition, variable exclusions, synthetic-data audit and interpretation, with AI-assisted implementation and methodological review. **Pablo Aguirre Araya, physician in Chile.**

[Notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Contact](mailto:pablo.aguirre.araya@gmail.com)
