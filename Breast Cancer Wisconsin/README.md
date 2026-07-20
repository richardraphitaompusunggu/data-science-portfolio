# Breast Cancer (WDBC) — Can Nuclear Morphology Tell Malignant from Benign?

A Tier A diagnostic **and** predictive analysis of 569 fine-needle-aspirate (FNA) samples, built to the
McKinsey communication standard: Pyramid Principle, MECE structuring, SCR storylining, answer-first delivery.

**📄 [Read the final report →](reports/final_report.html)** — self-contained HTML, light & dark themes, no external dependencies.

---

## The question

> *Can nuclear-morphology features computed from an FNA image reliably distinguish malignant from benign
> breast masses — and which measurements carry the diagnostic signal, given that the 30 features are heavily
> redundant by construction?*

Framed as a brief to a **clinical-decision-support lead** deciding whether digitized nuclear morphometry is
dependable enough to run as a **second-reader / triage flag** alongside a pathologist — and, if so, which
features to prioritize. The costly error is a **false negative** (a malignancy called benign), so the
analysis is judged on **malignant sensitivity**, never plain accuracy.

## The answer

> **Nuclear morphology separates malignant from benign nearly perfectly — a regularized linear model
> reaches 0.996 cross-validated ROC-AUC — but the signal is concentrated in a few *size* and *irregularity*
> "worst-case" measurements that are heavily redundant; so the deployment decision is which operating point
> to run, and — with no demographic data to audit — the model is defensible only as a second reader.**

| # | Key Line | Evidence |
|---|---|---|
| 1 | **The signal is concentrated, not spread** | Top separators are all size/irregularity features (`concave points_worst` d=2.7, `perimeter_worst` d=2.6, `radius_worst` d=2.5); **one feature alone reaches AUC 0.976**, while texture/standard-error features sit near zero effect |
| 2 | **The 30 features are only ~3 real signals** | Collinearity is extreme — **23/30** features with VIF>10 (max 3806), **21** feature pairs at \|r\|>0.9; **63%** of variance in the first two principal components |
| 3 | **Every model clears ROC-AUC ≈ 0.98 — the choice is the threshold** | At a sensitivity-first operating point the winner catches **41 of 42** malignancies (97.6% sensitivity) for **4** false alarms (94.4% specificity) on held-out data |

**Recommendation:** pilot the L2-logistic classifier as a **second-reader flag** at threshold ≈ 0.15 (tuned
for ≥98% sensitivity), prioritize capturing the "worst" size/irregularity features, and — because the data
carries no demographics to audit — **commission prospective multi-site validation before any autonomous role.**

## Results

| Model | CV ROC-AUC | Test ROC-AUC | Test PR-AUC |
|---|---|---|---|
| **Logistic (L2)** ★ | **0.996** | **0.998** | **0.996** |
| SVM (RBF) | 0.996 | 0.996 | 0.994 |
| Logistic (L1) | 0.993 | 0.998 | 0.996 |
| Random Forest | 0.990 | 0.998 | 0.996 |
| Gradient Boosting | 0.990 | 0.997 | 0.995 |
| 1-feature logistic *(baseline)* | 0.976 | 0.987 | 0.978 |
| Majority class *(baseline)* | 0.500 | 0.500 | 0.368 |

★ = winner: the **simplest** model within noise of the best CV AUC. Selection on RepeatedStratifiedKFold
(10×3 = 30 fits); the 114-sample held-out test scored **once**. The winner is well calibrated (Brier 0.021).

At the **clinical operating point** (threshold ≈ 0.15, tuned on training-fold CV for ≥98% sensitivity) the
winner scores **41/42 malignancies caught (97.6% sensitivity), 4 false alarms (94.4% specificity)** on the
held-out test — the second-reader trade-off, quoted in raw counts rather than at the arbitrary 0.5 default.

> Regularization, kernels, and trees all land within a whisker of each other. On a nearly separable,
> heavily collinear problem the algorithm stops being the differentiator — so the report spends its energy
> on the **operating point**, which is a clinical cost judgement, not a modelling one.

## Three findings that shaped the analysis

**1. Accuracy would have lied.** With 63% benign, a "predict benign" model scores 63% accuracy while missing
every cancer. The entire evaluation leads with **malignant recall and PR-AUC**, and the threshold is tuned
for the cost of a false negative — not left at 0.5.

**2. No single feature is irreplaceable.** Permutation importance on the winner is *tiny and dispersed*
(largest single-feature AUC drop = 0.010): removing any one size/irregularity feature barely hurts because
its collinear siblings substitute. This is the model-side echo of the VIF result — the trustworthy driver
ranking is the univariate **Cohen's d**, not permutation importance, and the report says so rather than
over-claiming a "top feature."

**3. The fairness audit cannot be run — and that caps the recommendation.** WDBC carries **no age, race,
site, or any demographic attribute**, so subgroup performance is *un-auditable from this data*. That is not
a free pass: it is a hard limitation that holds the deployment claim at "second reader, pending prospective
multi-site validation." It lives in the main narrative, not a footnote.

## Pipeline

| Notebook | Stage | What it does |
|---|---|---|
| [`01_ingestion.ipynb`](notebooks/01_ingestion.ipynb) | 1 | Loads `data.csv`, drops the phantom all-null trailing column, SHA-256 lineage, 10 schema checks incl. the exact **357 B / 212 M** split, governance classification |
| [`02_cleaning.ipynb`](notebooks/02_cleaning.ipynb) | 2 | 0 nulls / 0 duplicates independently verified; drops `id`, encodes `M=1/B=0`, flags the 13 zero-concavity benign samples, retains biological tails — full cleaning log |
| [`03_eda.ipynb`](notebooks/03_eda.ipynb) | 3 | Hypothesis-driven EDA against a MECE issue tree: Cohen's d ranking, univariate AUC, collinearity heatmap, PCA, zero-inflation — every chart an action title + So What |
| [`04_analysis.ipynb`](notebooks/04_analysis.ipynb) | 5a + 4/5b/6 | Welch + Mann-Whitney with Cohen's d & bootstrap CI, BH-FDR, VIF, 5 engineered features in a leakage-safe Pipeline, 7-model bake-off, clinical operating point, calibration, fairness constraint — writes all 7 exhibits + `_key_figures.json` |
| [`05_reporting.ipynb`](notebooks/05_reporting.ipynb) | 7 | Assembles `final_report.html` from the machine-written figures file; 12 automated deliverable checks |

**Analysis path:** Path A (diagnostic) → Path B (predictive classification). **Path C (causal) is
deliberately not run** — nuclear morphology is an observed marker, not an intervention, so no counterfactual
is identifiable and no "change feature X" claim is offered.

## Reproducing

Place the source `data.csv` in the project root, then:

```bash
pip install pandas==3.0.3 numpy==2.5.1 scikit-learn==1.9.0 scipy==1.18.0 \
            statsmodels==0.14.6 matplotlib==3.11.0 seaborn==0.13.2 pyarrow==25.0.0
jupyter lab      # run notebooks 01 → 05 in order
```

Seed `42` throughout, relative paths only. **The notebooks import no local modules** — chart styling and the
dtype contract are inlined in each one, so a clone runs with nothing but the source CSV and the packages
above. Only the notebooks, the report, and this README are committed; data, figures, models, and helper
artifacts stay local by design.

## Limitations

- **Single-source historical sample** (n=569, Wisconsin); no external validation cohort.
- **No demographics** → fairness subgroup audit impossible (see finding 3).
- **Association, not causation** — decision support only, never autonomous diagnosis.
- **"Worst" features may be instrument/segmentation-specific** — portability is unproven.
- **Near-perfect separability is a known property of this benchmark** — real-world screens are harder.
- **No peer review** — recorded as an open gate item rather than silently ticked.

## Ethical note

This is a clinical dataset and the model affects people. It is used here for methodological demonstration:
the diagnosis is a decision-support signal, not a verdict; the recommendation is explicitly capped at
second-reader use; and the impossibility of a fairness audit is surfaced, not hidden. No present-day
clinical authority is claimed.

---

**Source:** Wisconsin Diagnostic Breast Cancer (WDBC) — Wolberg, W.H., Street, W.N. & Mangasarian, O.L. ·
via the UCI Machine Learning Repository · 569 FNA samples, 357 benign / 212 malignant, 30 nuclear-morphology
features (10 measurements × mean / SE / worst).
