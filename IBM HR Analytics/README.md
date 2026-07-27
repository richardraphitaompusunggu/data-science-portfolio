# IBM HR Employee Attrition — driver analysis & early-warning model

**Tier A study** — diagnostic (rung 2) → predictive (rung 4), built to the McKinsey communication
standard: answer-first, MECE, action titles, a "So What" under every exhibit.

📊 **[Read the final report](reports/final_report.html)** — self-contained HTML, all charts embedded.
*(GitHub shows `.html` as source; open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
or download and open locally.)*

---

## The question

> Among current employee profiles, which characteristics are genuinely associated with attrition —
> once features that are statistically indistinguishable from random noise are stripped out and pay
> is separated from seniority — and how well can the survivors score attrition risk *before* an
> engagement survey is run?

## The finding

**11 of this file's 27 candidate attrition drivers cannot be told apart from a column of random
numbers.**

The dataset ships with an accidental gift: `DailyRate`, `HourlyRate` and `MonthlyRate` are presented
as compensation fields but are verifiably uniform random draws — they correlate with actual pay at
|ρ| < 0.06, their implied hours-per-day and days-per-month ratios spread 12–24× between the 5th and
95th percentile, and uniformity cannot be rejected.

Rather than delete them at cleaning time, this study **keeps them as a calibrated placebo control**.
That turns "is this association real?" from a p-value argument into a direct comparison against a
known zero — and the result justifies the design:

> **`DailyRate` — proven random — scores 0.545 univariate AUC against attrition and survives
> Benjamini–Hochberg correction at q = 0.048.** A conventional significance procedure publishes it
> as a driver. The placebo is what stops that happening.

Measured against the ceiling those known-random columns actually reach, the casualties are not
obscure fields: **PerformanceRating** (0.501, exactly chance), **PercentSalaryHike**,
**TrainingTimesLastYear**, **WorkLifeBalance** and **YearsSinceLastPromotion** — all standing items
on attrition dashboards.

## What survives

| Finding | Evidence |
|---|---|
| **Overtime is the one dominant lever** | 30.5% attrition vs 10.4%, in all three departments; largest effect size in the file (Cramér's V = 0.24); OR ≈ 2.1 after adjustment; and 3.4× the next column's permutation importance in the fitted model |
| **Pay is not a lever** | The widely-quoted 38% median pay gap between leavers and stayers collapses to 10% within Job Level 1 — `MonthlyIncome` is 90.3% explained by `JobLevel` (R²). A within-level effect survives (OR 0.45, p = 0.006) but lives entirely in the top pay quartile |
| **Attrition is a first-year, entry-level problem** | 34.9% in year one vs 8.1% after a decade; 143 of 237 leavers sit in Job Level 1 |
| **Business travel and stock options survive adjustment** | Monotonic 8.0% → 15.0% → 24.9% by travel frequency; stock-option response is *non*-monotonic (24.4% / 9.4% / 7.6% / 17.6%), so it is encoded nominal — treating it as ordinal reports the wrong sign |

## The model, and what it costs

An **HRIS-only score** (no engagement survey) reaches **0.553 PR-AUC** against a 0.161 base-rate
floor — 0.510 on the held-out set, 95% CI [0.37, 0.65]. It is **well calibrated**
(Brier 0.102 vs 0.135 for a base-rate model), so risk tiers are supportable rather than misleading.

At the recommended operating point it **flags 344 of 1,176 employees to catch 133 of 190 leavers** —
2.6 conversations per leaver found. Pushing to 90% recall would mean calling nearly two-thirds of
the workforce to catch 38 more.

The five engagement-survey fields are **quarantined into a separate scope** because the data
assessment cannot rule out that they were captured at or near the point of leaving. They add
+0.102 PR-AUC — a lift with two indistinguishable explanations, genuine signal or post-decision
reporting, which a single snapshot cannot separate.

### A fairness finding that deletion does not fix

The model flags single employees **2.8× as often** as divorced ones. That is statistically correct
(their actual attrition rates are 25.5% vs 10.1%) and it is a protected-adjacent targeting
disparity. Dropping `MaritalStatus` moves the ratio only to 2.6× — because the remaining HRIS
columns reconstruct marital status at **0.94 AUC**, with `StockOptionLevel` carrying most of it.
This is the "we just didn't include the protected attribute" non-defence, demonstrated with a
number. Gender, by contrast, is neither predictive nor reconstructible (0.51 AUC).

---

## Notebooks

| # | Notebook | Stage | What it establishes |
|---|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | 1 | 32 schema rules across 3 families, 0 failures; sampling-frame survivorship check (null result, reported); protected attributes flagged |
| 02 | [Cleaning & forensics](notebooks/02_cleaning.ipynb) | 2 | Three columns convicted as noise on 4 independent tests, then **retained as a placebo control**; measurement-scale contract persisted to disk; collinearity register |
| 03 | [EDA](notebooks/03_eda.ipynb) | 3 | Where attrition concentrates; the pay confound made visible; Likert monotonicity checks that decide the encoding |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | 5a | Per-feature permutation nulls built **before** judging; FDR + noise-ceiling verdicts; pay decomposition across three specifications |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | 4 · 5b · 6 | Two-scope bake-off, ablation, operating point in headcount, calibration, fairness audit + proxy test |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | 7 | Builds `final_report.html` — hardcodes no numbers, reads them all from upstream JSON |

## Method notes

- **Noise floor built before any feature is judged** — per-feature permutation nulls (2,000 label
  shuffles; 400 for nominal features, scored **out-of-fold** so cardinality cannot buy separation),
  plus 3 real and 20 synthetic known-random columns to locate the ceiling.
- **Three-criterion verdicts** — a feature is a DRIVER only if it clears the noise ceiling *and*
  survives Benjamini–Hochberg FDR. Significance alone was demonstrably not enough.
- **Scope quarantine** — the primary model never sees an engagement field at any stage, including
  feature selection.
- **Negative results shipped** — the ten derived features proposed in the data assessment were
  ablation-tested as a block, changed PR-AUC by −0.006 against a fold-to-fold sd of 0.07, and were
  **dropped**. Class weighting was tested and not used.
- **Path C (causal) explicitly out of scope** — no hire dates, no exit dates, no assignment
  mechanism. Every finding is stated as an association; the report says so on page one.

## Limitations

This is IBM's **synthetic** demonstration file, not a real workforce — the coefficients describe a
generator, not an organisation. What transfers is the *method* (noise calibration, scope quarantine,
confound decomposition) and the instrumentation recommendation: **capture hire date, exit date, exit
reason and manager ID** before anyone attempts a causal claim. Nothing here is deployable.

## Reproducing

```bash
pip install -r requirements.txt          # pandas 3.0.3, scikit-learn 1.9.0, statsmodels 0.14.6, …
# place WA_Fn-UseC_-HR-Employee-Attrition.csv in the project root
# run notebooks 01 → 06 in order
```

Seed 42 throughout; no local module imports, so a fresh clone runs on the CSV and the pinned
packages alone. Data files, figures and intermediate artefacts are gitignored — only notebooks, the
report and this README are tracked.

**Source:** [IBM HR Analytics Employee Attrition & Performance](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset)
(1,470 × 35, fictional data created by IBM data scientists).
