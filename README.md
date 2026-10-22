# IntraSight-ESI

**English** · [Leer en español](README.es.md)

**Looking for hidden risk within the same triage category.**

A retrospective machine learning research project for the Triagegeist competition. It asks whether emergency department patients assigned the same triage stratum can be ranked by their risk of hospitalization, transfer or death in the emergency department.

[View the public repository](https://github.com/pabloagar/triagegeist_competition) · [Open the Kaggle notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Read the technical writeup](https://github.com/pabloagar/triagegeist_competition/blob/main/docs/writeup.md)

**Status:** research prototype evaluated retrospectively on public US survey data. It has not undergone external or prospective clinical validation and must not be used to make care decisions.

## The question

Patients within a single triage category do not all have the same risk. Rather than predicting the category itself, IntraSight-ESI investigates whether information available at triage can identify a subgroup with a higher rate of serious disposition outcomes. Earlier reassessment is the proposed future use case; its benefit has not been tested.

**What “ESI-like” means here:** NHAMCS records the five-level `IMMEDR` immediacy category, not confirmed Emergency Severity Index assignments. Results refer to those survey strata. The outcome describes disposition, not imminent physiological deterioration.

## Approach

- Audit the competition's synthetic data before interpreting model performance.
- Use public NHAMCS emergency department data from 2015–2019 for development and cross-validation; hold out 2022 for a later evaluation.
- Compare LightGBM with regularized logistic regression.
- Separate triage-available features from a deliberately leaky control that includes later information.
- Assess performance within each stratum, with year-blocked cross-validation, subgroup checks and frozen thresholds for the 2022 holdout.
- Evaluate calibration as a methodological check; base the alert policy on score ranking rather than claiming individually calibrated probabilities.

## Main retrospective result

The primary group is **ESI-like 4**. The documented evaluation reports:

| Measure | Development cross-validation | 2022 forward holdout |
| --- | --- | --- |
| Alert burden | Top 5% by score | 6.2% under the frozen threshold |
| Outcome enrichment over the stratum's baseline rate | 5.24× | 5.31×, with a 95% bootstrap interval of 3.36–7.30 |
| Positive predictive value | 14.1% | See the notebook for the full holdout metrics |
| Recall | 26.2% | See the notebook for the full holdout metrics |

Enrichment compares the outcome rate among flagged records with the baseline rate in the same stratum. It is not an estimate of improved patient outcomes. The holdout's realized alert burden differs from the development target because the numeric threshold was kept fixed.

![Original notebook figure comparing cross-validation and 2022 holdout enrichment across the three ESI-like strata](assets/retrospective-validation.png)

*Original saved notebook figure. Its “ESI 3/4/5” labels are shorthand for NHAMCS IMMEDR strata, not confirmed ESI assignments. Error bars show 95% bootstrap intervals; ESI-like 5 remains exploratory.*

## My contribution

**Dr. Pablo Aguirre Araya, physician in Chile's MGZ public service program, Arica.** Framing the clinical question, defining outcomes and variable exclusions, auditing the synthetic data and interpreting the results, with AI-assisted development for implementation and methodological review.

The contribution is the clinical framing and auditable evaluation, alongside the model implementation.

## Reproducibility

The public repository includes the analysis notebook, data acquisition instructions, audits, a technical writeup and a citation file. The workflow is written in Python, using pandas, NumPy, scikit-learn, LightGBM, SciPy and Matplotlib. Package versions and random seeds are specified in the project.

To inspect or reproduce the analysis, start with the [original reproduction instructions](https://github.com/pabloagar/triagegeist_competition/blob/main/docs/README-before-bilingual-overview.md#reproduce) and [data guide](https://github.com/pabloagar/triagegeist_competition/blob/main/data/README.md). The competition's synthetic dataset is not redistributed.

## Limits and next steps

NHAMCS is a US survey rather than a local hospital's operational record. The composite outcome combines different dispositions, and the later holdout does not establish performance in Chile or in live care. Subgroup audits do not resolve fairness questions.

The next research steps are external validation, prospective evaluation and testing whether a reassessment workflow would improve care without creating excessive alert burden. This project has not reached those stages.

[Repository](https://github.com/pabloagar/triagegeist_competition) · [Notebook](https://www.kaggle.com/code/pabloaguirrearaya/intrasight-esi-v2) · [Contact](mailto:pablo.aguirre.araya@gmail.com)
