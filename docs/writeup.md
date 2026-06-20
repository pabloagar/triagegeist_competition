# IntraSight-ESI: Finding the hidden high-risk patient within a triage level

*A within-ESI risk-stratification layer for earlier reassessment, built and validated end-to-end on real NHAMCS emergency-department data.*

## Clinical Problem Statement

As a physician in emergency care, I want to be precise about the real bedside problem. Much triage ML tries to predict the ESI level itself — useful as a benchmark, but not the hardest clinical question. Many ESI 1 and a large fraction of ESI 2 patients declare themselves quickly at the bedside: shock, respiratory failure, unresponsiveness, or major trauma rarely need a model to be noticed.

The harder safety problem is subtler. It lives inside the lower-acuity categories, especially ESI 3 and 4 — patients who often look stable enough to wait, yet a small subgroup will ultimately need hospital-level care. In a crowded department, this is exactly the group most vulnerable to delayed reassessment: not from carelessness, but because the patient does not look dangerous yet. The failure mode is concrete — a stable-looking patient whose deterioration is recognized later than it could have been.

IntraSight-ESI targets that gap. It does not re-predict the ESI label; it takes the triage category as given and asks: *within* a fixed ESI category, which patients carry unusually high risk of a serious disposition outcome? It is not a replacement for clinical judgment or a new triage scale, but a prioritization layer for earlier reassessment inside the existing pathway — a suggestion to look again sooner, made by a person, not the model.

## Approach and Methodology

The project began on the competition's synthetic dataset, and the key lesson came from watching it fail. A model reached a global AUC of 0.81, but a stratified permutation test broke the illusion: shuffling the outcome *within* each triage level left the AUC unchanged (0.814 shuffled vs 0.814 real, Z = −0.8) — the model had recovered the ESI level itself, not hidden within-category risk. A clinician-led audit explained why: the synthetic records contained patterns that cannot exist in living patients. BMI was hard-clamped to exactly [10, 65]; 2,105 records (2.6%) had diastolic pressure at or above systolic; and missingness fell entirely in low-acuity groups (≈20% of ESI 4, ≈24% of ESI 5, versus 0% in ESI 1–3) while no deceased patient had a missing vital — a structure no real ED produces. The lesson: an aggregate metric can look polished while the data beneath it would not survive a moment of clinical scrutiny.

I pivoted to real data: the National Hospital Ambulatory Medical Care Survey (NHAMCS), a public CDC/NCHS survey of US ED visits, which preserves the genuine joint physiology of real patients. Training and cross-validation use five consecutive pre-COVID years, 2015–2019 (59,571 ESI 3–5 visits); 2020–2021 are excluded as COVID-disrupted. NHAMCS 2022 (8,473 visits) is held out completely as an independent forward holdout — it never enters training, cross-validation, calibration, or threshold selection, and is scored once at the end. A holdout that sits chronologically *after* training is more defensible than one before it.

The outcome was a composite of hospitalization, transfer to another facility, or death in the ED (`ADMITHOS | TRANOTH | DIEDED`) — interpreted as a need for hospital-level care with operational consequences, not as a claim about imminent physiologic deterioration.

The analysis plan — leakage audit, primary hypotheses, subgroup definitions, validation strategy — was written before the model was trained or any result examined, following pre-registration principles. The core validation used Leave-One-Year-Out cross-validation: each training year is held out in turn, testing whether the signal is stable across calendar years rather than only within a random split. Early stopping uses the most recent training year, never the test year, to avoid a subtle temporal leak. Three feature sets were defined deliberately: **Set A**, strict triage-only (22 features: demographics, arrival mode, vitals, structured reason-for-visit codes); **Set B**, EHR-at-triage (35 features, adding comorbidity flags available if the nurse has record access); and **Set C**, a leaky positive control (36 features) that intentionally includes post-triage information to confirm the pipeline detects leakage when present. NUMMED (medications given during the visit) was excluded from the honest sets because it occurs *after* the risk moment the model supports — the same reason it is omitted in the landmark Harvard triage ML study (Raita et al., Critical Care 2019). RACERETH, PAYTYPER, and calendar year are excluded from every set.

LightGBM was the main model: gradient-boosted trees handle non-linear interactions in heterogeneous tabular clinical data without manual specification. An L2-regularized logistic regression was included as a transparent comparator. Predicted risks were calibrated with cross-validated isotonic regression so scores can be communicated as probabilities, not just rankings.

A defining feature of this submission: the notebook recomputes the entire pipeline live, from the raw NHAMCS fixed-width files through to the figures, with no precomputed artifact read back in. Every number below is produced by the preceding cell, and a built-in consistency check confirms each headline figure traces to a live variable.

## Results and Findings

On real data, the intra-ESI signal is robust. Permutation tests (200 shuffles per group, on fixed out-of-fold predictions) showed genuine within-category predictability, with Z-scores of **50.86, 21.38, and 7.97** for ESI 3, 4, and 5 respectively (all p < 0.005).

The main operational result is ESI 4. Alerting only the top 5% of ESI 4 patients by score produced a positive predictive value of 14.1%, an enrichment of **5.24×** over the ESI 4 base rate, and a recall of 26.2%. In practical terms, roughly one in seven alerted ESI 4 patients experienced the serious outcome, while the alert burden remains small enough for a busy ED to act on.

| Group | Alert burden | PPV | Enrichment (CV) | Recall | Enrichment (2022 holdout) | Interpretation |
|---|---|---|---|---|---|---|
| ESI 3 | Top 10% | 41.7% | 3.16× | 31.6% | 2.89× | Useful prioritization layer |
| ESI 4 | Top 5% | 14.1% | 5.24× | 26.2% | 5.31× | **Primary target group** |
| ESI 5 | Top 10% | 10.2% | 3.22× | 32.2% | 5.12× (wide CI) | Exploratory only |

Four findings matter as much as the headline.

**First, the signal does not depend on model complexity.** In ESI 4, LightGBM and logistic regression were nearly equivalent in discrimination (ROC AUC 0.7571 vs 0.7486). The contribution is therefore not a black-box trick; the important part is the intra-category framing, the leakage control, and the alert policy.

**Second, the age-subgroup result refuted my own hypothesis — and that is the interesting part.** I had pre-declared that the model would help most in patients aged ≥65. It did not: enrichment was consistently *higher* in younger patients (ESI 4: 5.35× for <65 vs 2.04× for ≥65). Clinically this makes sense — older patients already carry visibly elevated baseline risk that ESI partly captures, so age itself is the cue. A stable-looking 45-year-old with an ESI 4 label is different; there the model surfaces risk that age alone would not flag.

**Third, the result is robust to the outcome definition and to survey sampling.** Adding observation-unit-then-hospitalization (OBSHOS) to the outcome did not change ESI 4 enrichment at all (6.27× either way, on the years where OBSHOS is reliable): zero patients were flagged as serious *only* because of OBSHOS, so the headline does not depend on that exclusion. Weighting by the NHAMCS patient visit weights (PATWT) — a sensitivity check, not a national estimate — left the numbers close (ESI 4: 5.80× weighted [4.83–6.73] vs 5.24× unweighted), evidence the signal is not a sampling artifact.

**Fourth, the policy held on the 2022 forward holdout** with thresholds frozen in advance: ESI 4 enrichment was 5.31× [3.36–7.30]. I read this as *no evidence of policy degradation* on independent, later data — not as improvement — and I state the central confound plainly: 2022 is the only post-COVID year here, so any difference mixes genuine temporal drift with post-COVID shifts in case mix and coding that this design cannot separate.

On equity, RACERETH and PAYTYPER were never predictors, but I audited alert rates and PPV by group. In ESI 4, alert rates broadly tracked group base rates (Hispanic patients had both the highest base rate, 3.1%, and the highest alert rate, 6.2%; non-Hispanic White and Black patients sat near 3.4–3.6%), and PPV when alerts fired was similar across groups (0.127–0.167). This admits more than one reading — real differences in measured risk versus differences in feature distributions that determine who crosses the threshold — and retrospective survey data cannot adjudicate between them. I report it transparently rather than claim the fairness question is solved.

## Limitations and Future Work

What the solution does *not* do: it does not predict imminent deterioration, does not replace clinical judgment, and is not a rule-out tool — non-alerted patients still receive standard reassessment for their ESI category.

Assumptions that may not hold in a live environment: NHAMCS is a survey, not a hospital log (partially mitigated by the PATWT check above); chief complaint is structured RFV codes, not the free-text notes a real ED produces; the composite outcome combines dispositions with different clinical meanings; and all validation is retrospective, including the 2022 holdout. ESI 5 remains exploratory — few outcomes, wide intervals, and a simple clinical rule (age ≥65 plus hypoxemia) outperforms the model there, so it is not recommended for deployment. Characterizing misses on 2022 showed false negatives skew younger and have more missing vitals than true positives — a concrete, addressable failure mode rather than random error.

What would move this toward deployment: external validation on a real ED's operational data; true prospective validation, measuring whether earlier reassessment of alerted patients actually changes outcomes; recurring fairness monitoring by demographic group; and workflow integration with clinician override and alert-burden controls.

## Reproducibility Notes

**Datasets.** NHAMCS ED Public Use Files, CDC/NCHS, survey years 2015–2019 and 2022 — public domain (CC0). To make the raw CDC files reproducible on Kaggle, they are mirrored in a companion public dataset (`nhamcs-ed-2015-2022-raw`); attach it to run the notebook. The competition's synthetic dataset is read only from the competition's own input mount (used solely for the Section 2 audit) and is never redistributed.

**The submitted notebook runs end-to-end without errors, set to public, with no precomputed artifacts.** It parses the raw fixed-width files and executes the full pipeline in a single pass on CPU (no GPU, a few minutes), regenerating every figure and number, followed by a provenance check confirming the run parsed fresh from raw files in the Kaggle environment.

**Environment.** Python 3.11; key packages lightgbm, shap, scikit-learn, pandas, numpy (all preinstalled on Kaggle). Random seeds are fixed (`RANDOM_STATE = 42`) across CV splits, isotonic calibration folds, and permutation tests, so reported metrics reproduce on re-execution.

Full pipeline, audits, and reproducibility materials: **github.com/pabloagar/triagegeist_competition**

## Development note

This project was built by a physician working in emergency care, with formal training in applied AI, using AI-assisted development for code generation and methodological review. Every clinical decision — outcome definition, variable exclusions, synthetic-data audit, interpretation of every result — was made and verified by the clinician. Domain expertise paired with modern tooling can produce a rigorous, auditable clinical analysis, and the clinical judgment is the part that cannot be delegated.

## What is new, and why it matters

The novelty is not the algorithm. It is the question. Triage AI usually asks "what acuity level is this patient?" — IntraSight-ESI asks instead: among patients *already assigned the same acuity level*, who is unusually risky? Waiting-room harm is not always caused by missing the obviously crashing patient; sometimes it is losing sight of the stable-looking one whose risk is hidden inside a low-acuity label. IntraSight-ESI shows that this hidden signal exists in real public ED data, persists across five training years, survives a deliberate leakage audit, replicates on an independent forward holdout, holds under survey weighting, and is robust to the observation-unit outcome choice — a small, clinically reviewable alert layer for earlier reassessment within the existing triage system.
