# Triagegeist — Intra-ESI Prioritization Alert System

An intra-ESI risk stratification system that flags higher-risk patients *within*
the same Emergency Severity Index (ESI) triage category, using real NHAMCS
emergency department data (2016–2022). Built for the Triagegeist Kaggle competition.

## What it does

Standard ESI triage assigns each patient a level (1–5). This project asks a
narrower question: *within* a given ESI level (3, 4, 5), can we identify the
patients who are most likely to experience a serious disposition outcome
(hospitalization, transfer, or death in the ED)? The system produces a
prioritization alert to suggest earlier reassessment for those patients.

## Data

This project uses the **National Hospital Ambulatory Medical Care Survey
(NHAMCS)**, a public dataset from the U.S. CDC. The raw data is **not** included
in this repository. Download it from the CDC:
[https://www.cdc.gov/nchs/ahcd/datasets_documentation_related.htm](https://ftp.cdc.gov/pub/Health_Statistics/NCHS/dataset_documentation/NHAMCS/)

A 2015 survey year is held out as an independent historical validation set.

## Pipeline

Run the notebooks in this order (each depends on outputs of the previous ones):

0. `00_protocol_and_leakage_audit.ipynb` — pre-registered analysis protocol, hypotheses, and leakage audit (defined before looking at results)
1. `01_eda_nhamcs.ipynb` — data loading, cleaning, exploratory analysis
2. `02_model_validation.ipynb` — LightGBM model, cross-validation, SHAP
3. `02b_logistic_baseline.ipynb` — L2 logistic regression baseline
4. `03_alert_policy.ipynb` — alert policy and comparison vs. clinical rules
5. `04_validation_subgroups_fairness.ipynb` — calibration, 2015 holdout, subgroup and fairness audit
6. `05_kaggle_submission.ipynb` — final submission notebook (loads precomputed artifacts)

## Methodology highlights

- **Pre-registered protocol:** hypotheses and analysis plan fixed up front (NB00) before results were examined
- **Validation:** Leave-One-Year-Out (LOYO) cross-validation
- **Feature sets:** triage-only (strict), EHR-at-triage, and a leaky positive control
- **Leakage control:** NUMMED excluded; a deliberate leaky feature set used as positive control
- **Calibration:** isotonic calibration, evaluated with Brier skill score
- **Independent holdout:** 2015 survey year
- **Baseline:** L2-regularized logistic regression
- **Fairness:** alert-rate audit by race/ethnicity, sex, and payer

## Reproducibility

Built and tested with **Python 3.11.9**. Install dependencies with
`pip install -r requirements.txt`, download NHAMCS as described above, then run
the notebooks in order. The submission notebook (NB05) reads precomputed
artifacts from `reports/`.

## License

Code released under the MIT License (see LICENSE).
