# Cleveland Heart Disease — What This Benchmark Actually Shows

**Tier A diagnostic + predictive analysis of the UCI Cleveland heart-disease file**, following the
McKinsey communication standard (Pyramid Principle, MECE, SCR storyline, answer-first delivery).

> **Governing thought.** This file's celebrated accuracy is an artefact of duplication and
> label-adjacent features; the real, decision-relevant signal is worth 0.87 AUC, it comes almost
> entirely from the exercise stress test and symptom presentation, and it must not be deployed.

📄 **[Read the full report →](reports/final_report.html)**
*(GitHub renders `.html` as source — open via [htmlpreview](https://htmlpreview.github.io/) or download and open locally.)*

---

## The business question

> Which clinical and exercise-test findings distinguish patients with angiographic heart disease in
> this referral cohort — and, once the file's 70% duplication and angiography-derived features are
> removed, how well can a classifier actually separate the classes?

Framed as a brief to a **clinical data-science lead** deciding two things: whether the 95–100%
accuracy figures routinely published on this file are credible, and which signals would justify a
prospective triage study. "Deploy a patient risk score" is **not** answerable from this data, and
the report says so on page one rather than in a footnote.

---

## Headline findings

| | |
|---|---|
| **302** | actual patients in a file of **1,025 rows** — 723 rows (70.5%) are exact copies |
| **98.5%** | of a standard test set has an **exact twin in its own training set** |
| **+18 pts** | accuracy the standard protocol invents (100% → 81.6%, same model, same features) |
| **0.873** | honest pre-test AUC, 95% CI [0.833, 0.910] |
| **11 of 13** | features survive FDR correction — cholesterol and fasting blood sugar do not |

### 1. The file is 302 patients wearing 1,025 name tags

723 of 1,025 rows are exact duplicates, a mean replication of 3.4×. Deduplication also shifts the
label balance (51.3% → 54.3% positive), so the copying was not uniform.

**The leakage experiment.** Same model, same 13 features, only the split changes:

| Protocol | Data | Accuracy | ROC-AUC |
|---|---|---|---|
| P1 — contaminated holdout | 1,025 rows | **1.000** | **1.000** |
| P2 — contaminated 10-fold CV | 1,025 rows | **1.000** | **1.000** |
| P3 — honest, deduplicated | 302 patients | 0.816 | 0.904 |

A random forest scores **perfectly** under the protocol most published benchmarks use — because
98.5% of its test set is memorised. **Ten-fold CV does not rescue it**; the folds share the same
patients. Cross-validation protects against an unlucky split, not against duplicated rows.

The inflation is model-dependent in a diagnostic way: contamination is worth **+0.096 AUC** to the
random forest but only **+0.027** to penalised logistic regression. When a flexible model beats a
linear one by a wide margin on small tabular clinical data, duplication is the first thing to
suspect, not the last.

### 2. Two "valid" codes were missing data in disguise

Schema validation at ingestion caught `thal = 0` and `ca = 4` — codes the data dictionary does not
permit, the original UCI `?` markers coerced into numbers. Six patients post-deduplication.
Modelling them as real categories would have invented a thallium result and a fifth coronary vessel.
Recoded to explicit `NaN`, with imputation deferred into the CV folds; a sensitivity check confirms
the conclusion is unchanged (**+0.003 AUC** when those patients are dropped instead).

### 3. The stress test carries the signal; resting lipids carry none

11 of 13 features survive Benjamini–Hochberg correction, each reported with a bootstrap CI rather
than a bare p-value. Strongest: ST depression (*d* = −0.95), peak heart rate (*d* = +0.93),
chest-pain type (*V* = 0.52), exercise angina (*V* = 0.43).

**The null result is the useful part.** Cholesterol differs between groups by 8.4 mg/dl with a CI
spanning zero ([−20.3, +3.2]) and does not survive correction; neither does fasting blood sugar. In
a cohort already referred for angiography, resting lipids add nothing on top of the stress test — so
a triage score built on them would be measuring the wrong thing. This is a statement about
*discrimination within a referral population*, not about whether cholesterol causes heart disease.

Under mutual adjustment (pseudo-R² = 0.426, max VIF = 1.71) the univariate ranking survives — the
signal is structural, not an artefact of one test's assumptions.

### 4. Honest performance is triage-grade, and model choice is irrelevant

**Scope 1 (pre-test, 11 features): 0.873 AUC.** **Scope 2 (adds angiography-derived `ca` and
`thal`): 0.911** — the +0.038 gap is the price of circularity, not skill, since both features come
from the same procedure that defines the label.

Every classifier — L1/L2 logistic, random forest, gradient boosting, SVM, k-NN — lands within
0.048 AUC of every other, **smaller than the ±0.066 fold-to-fold noise of any one of them**. The
flexible models finish *lower*. A single feature (`oldpeak`) already reaches 0.739.

Six clinically motivated derived features were built and **ablation-tested, then rejected**
(+0.001 / +0.003 AUC — inside the noise). Reported as a negative result, not silently dropped.

At a threshold set for 90% sensitivity — because a missed diagnosis is the costly error — the model
catches **148 of 164** disease cases, misses **16**, and sends **50 of 138** healthy patients for
further workup. Calibration is sound (Brier 0.144, 42% better than the base rate).

### 5. The fairness failure is one of threshold equity, not discrimination

Women are 96 of 302 patients (32%). The model ranks them *at least as well* as men
(AUC 0.896 [0.803, 0.967] vs 0.836 [0.780, 0.889]) — but one shared threshold treats them very
differently:

| | Sensitivity | Specificity | Flagged for workup |
|---|---|---|---|
| **Women** (75% prevalence) | 98.6% | 45.8% | **87.5%** |
| **Men** (45% prevalence) | 83.7% | 67.5% | 55.3% |

Nearly nine in ten women are flagged, and more than half the healthy ones. The driver is referral
prevalence — 75% vs 45% — a **selection artefact, not a fact about women**: historically women were
sent for angiography only when suspicion was already high. The model inherits that selection as a
lower effective bar.

The proxy check came back **negative**: sex is reconstructible from the other features at only
AUC 0.599. Unlike most datasets, dropping the column here genuinely would remove most of its direct
influence — a live design option, though it would not touch the prevalence-driven threshold effect.

---

## Recommendation

1. **Mandate dedup-before-split and a declared feature scope** for any tabular medical benchmark the
   group publishes or reviews. Treat existing results on this file as void until re-run.
2. **Prioritise the exercise-stress variables** — ST depression, exercise angina, ST slope,
   age-adjusted peak heart rate — in any prospective triage study. Do not build one around resting
   cholesterol or fasting blood sugar.
3. **Do not deploy.** A 1988 single-cohort model with n = 302, no external validation, and a
   threshold that flags 88% of women versus 55% of men is a methodological artefact, not a clinical
   instrument.

---

## Method

| Stage | Notebook | What it establishes |
|---|---|---|
| 1 · Ingestion | [`01_ingestion.ipynb`](notebooks/01_ingestion.ipynb) | Lineage + SHA-256, 14 schema rules with **2 documented failures**, governance review flagging `sex`/`age` as protected |
| 2 · Cleaning | [`02_cleaning.ipynb`](notebooks/02_cleaning.ipynb) | Duplication **forensics** (not just `drop_duplicates`), placeholder recoding, outliers retained by decision |
| 3 · EDA | [`03_eda.ipynb`](notebooks/03_eda.ipynb) | Distributions, cross-tabs, correlation done correctly for nominal codes |
| 5a · Diagnostic | [`04_diagnostic.ipynb`](notebooks/04_diagnostic.ipynb) | Assumption checks → test selection, effect sizes + bootstrap CIs, FDR correction, odds ratios |
| 4 + 5b + 6 · Modelling | [`05_modeling.ipynb`](notebooks/05_modeling.ipynb) | Leakage experiment, feature ablation, two scopes, nested CV, operating point, calibration, fairness audit |
| 7 · Reporting | [`06_reporting.ipynb`](notebooks/06_reporting.ipynb) | Builds `final_report.html` — reads every number from artefacts, hardcodes none |

**Discipline applied throughout.** Deduplication before any split. Two feature scopes reported
separately so label-adjacent measurements can never be mistaken for skill. Preprocessing fitted
inside CV folds. Hyperparameters tuned in a **nested** inner loop — tuning on the reported folds
would have printed an optimistic +0.005 AUC, a smaller cousin of the same error. Repeated stratified
10-fold × 5 as the headline, because the one-touch 61-patient holdout returns a CI 0.21 AUC points
wide — far too wide to rank models. `RANDOM_STATE = 42` everywhere.

---

## Reproducing

```bash
pip install pandas==3.0.3 numpy==2.5.1 scikit-learn==1.9.0 scipy==1.18.0 \
            statsmodels==0.14.6 matplotlib==3.11.0 seaborn==0.13.2 pyarrow==25.0.0
jupyter lab                              # run notebooks/01 → 06 in order
```

Requires only `heart.csv` in the project root. The notebooks import **no local modules** — palette,
styling and the dtype contract are defined inline in each — so a fresh clone runs on the pinned
packages alone.

---

## Limitations

- **1988 single-cohort referral population.** These are associations among patients *already
  selected for angiography*, not population prevalence. The inverted age effect — disease patients
  average 52.6 years against 56.6 for those without — is a signature of that selection.
- **n = 302.** Every confidence interval here is wide, and they are reported rather than hidden.
- **No external validation cohort, no temporal data**, so no drift analysis is possible.
- **Causal inference is out of scope by design.** Every feature is an observed marker or test
  result, not an assignable intervention.
- **Not a clinical instrument.** Analysis is capped at methodological benchmarking and signal
  prioritisation.

---

*Data: [UCI Machine Learning Repository — Heart Disease (Cleveland)](https://archive.ics.uci.edu/dataset/45/heart+disease), 1988. De-identified public research data.*
