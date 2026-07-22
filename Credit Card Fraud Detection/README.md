# Credit Card Fraud — Is It Worth Reviewing, and Where Do You Draw the Line?

A Tier A diagnostic **and** predictive analysis of 284,807 card transactions, built to the
McKinsey communication standard: Pyramid Principle, MECE structuring, SCR storylining, answer-first delivery.

**📄 [Read the final report →](reports/final_report.html)** — self-contained HTML, light & dark themes, no external dependencies.

---

## The question

> *Given only 28 anonymized PCA components plus transaction time and amount, can we rank transactions by
> fraud risk well enough that a fixed-capacity review team catches most fraud value — and at what
> threshold should that team operate?*

Framed as a brief to a **card-issuer fraud-operations lead** who owns a manual-review queue with a fixed
daily capacity. Their decision is threefold: whether to deploy a scorer at all, where to set the cutoff,
and which failure modes to watch. Fraud is **0.17% of transactions — 1 in
578** — so plain accuracy is disqualified from the first page onward: a model
that flags nothing scores **99.83%** and catches zero fraud.

## The answer

> **Fraud in this portfolio is highly detectable despite fully anonymized features — a class-weighted
> gradient-boosted ranker converts a 1-in-599 needle-in-a-haystack into a
> 400-alert daily queue that catches 86% of fraud cases
> and 81% of fraud value while touching
> 0.7% of transaction volume — so the binding constraint is
> not model quality but review capacity, and the model should be deployed as a queue ranker, never as an
> autonomous decline engine.**

| # | Key Line | Evidence |
|---|---|---|
| 1 | **Fraud is learnable, and it holds up out-of-time** | **17 of 28** components separate the classes by a large effect, led by `V17` at **8.1 SD**; the winner reaches **0.831 average precision** vs a 0.0017 random floor (**497×**), and still scores **0.790** trained on Day 0 and tested on Day 1 |
| 2 | **But fraud is a *low-value* crime, so "fraud caught" has two answers** | Median fraud **9.82** vs **22.00** legitimate (bootstrap CI excludes zero) — the *mean* says the opposite. At capacity the queue catches **86% of cases** but **81% of value** |
| 3 | **The decision is capacity and cost, not the algorithm** | At 1,000 reviews/day precision is **20.5%** (~5 reviews per fraud). Hyperparameter tuning changed AP by **−0.012** — nothing — while the cost-optimal alert volume swings **197 → 1,245/day** purely on an unmeasured review cost |
| 4 | **What the model cannot do bounds how it may be deployed** | Every strong feature is an unnamed principal component, so it ranks but cannot explain; **no demographic field exists**, making a protected-subgroup fairness audit *impossible* rather than skipped |

**Recommendation:** deploy as a **review-queue ranker** at threshold ≈ 0.0009
(top 400 per 56,746 transactions ≈ 1,000 reviews/day) and **not** as
an auto-decline. Then: **(1)** ship the zero-amount business rule immediately — it needs no model and
flags **8.3×**-elevated risk in 0.64% of
volume; **(2)** measure the true cost per manual review within 30 days, the single highest-value missing
input; **(3)** stand up PSI drift monitoring before go-live and run 60 days in shadow mode.

## Results

Held-out test set (n = 56,746), **average precision** as the primary metric — random-ranker
floor 0.0017.

| Model | Test AP | Test ROC-AUC | Temporal AP (D0→D1) |
|---|---|---|---|
| **XGBoost (default)** ★ | **0.831** | 0.979 | **0.790** |
| XGBoost (tuned) | 0.819 | 0.977 | 0.788 |
| Random Forest | 0.812 | 0.944 | 0.795 |
| Logistic Regression | 0.721 | 0.969 | 0.702 |
| *Baseline: single feature `V14`* | *0.541* | *0.914* | 0.618 |
| Logistic + undersampling | 0.510 | 0.971 | — |
| *Baseline: Isolation Forest (unsupervised)* | *0.132* | *0.943* | — |
| *Baseline: always legitimate* | *0.0017* | *0.500* | — |

One honest wrinkle: **Random Forest edges the winner on the temporal split**
(0.795 vs 0.790). The gap is 0.005 AP — roughly one fraud case out of
201 — so it is noise rather than a reversal, and it does not move the operating point.
It is recorded here rather than smoothed over, because a reader comparing the two columns
would spot it.

**What the capacity curve says** — the exhibit the decision is actually made from:

| Reviews/day | % of volume | Precision | Fraud caught (count) | Fraud caught (value) |
|---|---|---|---|---|
| 100 | 0.07% | 97.5% | 41% | 29% |
| 250 | 0.18% | 77.0% | 81% | 77% |
| 500 | 0.35% | 40.5% | 85% | 78% |
| **1,000** ★ | **0.70%** | **20.5%** | **86%** | **81%** |
| 2,000 | 1.41% | 10.4% | 87% | 86% |
| 5,000 | 3.52% | 4.2% | 89% | 86% |

Two things this table settles. Precision is **98% at 100 reviews/day** — the very top
of the ranking is almost pure fraud — but that queue catches only 41% of cases.
And the curve is **flat beyond 1,000/day**: doubling the team to 2,000 buys 1 more point of
case recall while precision halves to 10.4%.

## Three findings that changed the recommendation

1. **The metric choice *is* the analysis.** ROC-AUC reads **0.979** for a model whose
   precision at operating capacity is 20.5%. It appears in this project only
   to keep showing that gap. A methodology-first report would have led with 0.98 and
   misled the reader.
2. **Removing 1,081 duplicate rows was a leakage decision, not housekeeping**
   — and it was tested rather than asserted. 19 of 492 fraud rows
   were exact copies (3.9% of the positive class). Retaining them
   inflates average precision by **+0.042**, which is exactly the artificial
   gain removal exists to prevent.
3. **A feature failed its test, and the negative result is reported.** `sec_since_prev_txn` looked like a
   velocity signal (Cohen's *d* = 0.50) but permutation importance on
   held-out data returned a *negative* value at rank 31 of 32. It was a faint
   echo of time-of-day; true per-card velocity is unbuildable without a cardholder key.

## Pipeline

| Notebook | Stage | What it does |
|---|---|---|
| [01_ingestion.ipynb](notebooks/01_ingestion.ipynb) | 1 | SHA-256 lineage, 14 schema rules, governance review — including the *absence* of demographics logged as a downstream blocker |
| [02_cleaning.ipynb](notebooks/02_cleaning.ipynb) | 2 | Verifies every documented claim independently; the duplicate decision argued on leakage grounds; 6 derived columns and 2 explicit refusals |
| [03_eda.ipynb](notebooks/03_eda.ipynb) | 3 | Hypothesis-driven, one section per issue-tree branch; ghost deck revised — 1 exhibit cut on negative evidence, 1 added |
| [04_diagnostic.ipynb](notebooks/04_diagnostic.ipynb) | 5a | Mann-Whitney U, Cohen's *d* + CIs, BH-FDR, Wilson intervals — and a demonstration of why p-values are demoted at n = 284k |
| [05_modeling.ipynb](notebooks/05_modeling.ipynb) | 4·5b·6·6b | Two baselines, six models, random **and** temporal splits, capacity curve, cost band, fairness audit, drift plan |
| [06_reporting.ipynb](notebooks/06_reporting.ipynb) | 7 | Builds the self-contained HTML report; 12 automated assembly checks |

## Method notes

- **Two splits, both reported.** A stratified random 80/20 *and* a temporal Day 0 → Day 1 holdout. The
  recommendation quotes the **temporal** figure, because production scoring is always out-of-time.
- **Leakage discipline.** Raw `Time` excluded (a row index in disguise); `amount_zscore` excluded (global
  = a monotone transform, per-class = target leakage); scaling fit inside the pipeline; undersampling
  applied **inside fold loops only**; test set scored once.
- **Effect sizes lead, p-values do not.** At n = 283,726 a component separating the classes
  by 0.105 SD — a gap no plot could show — still clears p < 0.05.
- **Fairness.** The protected-subgroup audit is **impossible** on this data, and that is reported in the
  main narrative rather than the appendix. The audits that *are* possible — amount band, hour-of-day —
  were run.
- **No SMOTE, no `imbalanced-learn`, no SHAP.** Class weighting beat in-fold undersampling by
  22%; permutation importance replaces SHAP.
  A fresh clone runs on the pinned list in `requirements.txt`.

## Limitations

48 hours of 2013 European data — no seasonality, no concept-drift evidence, and the
drift thresholds are **specified but unvalidated**. PCA features are uninterpretable and non-transferable
to another issuer's schema. No cardholder key, so the strongest real-world fraud features cannot be built.
Label provenance is unknown (confirmed vs. flagged fraud). **This is a proof of concept, not a
production-ready system.**

## Data

`creditcard.csv` (284,807 × 31, 144 MB) is **not committed**. Download from
[Kaggle — Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)
(ULB Machine Learning Group) and place it at `creditcard.csv/creditcard.csv`.

```bash
pip install -r requirements.txt
jupyter lab   # run notebooks 01 → 06 in order
```

Seed `RANDOM_STATE = 42` throughout. Every statistic in the report is read from
`reports/_key_figures.json`, written by the notebooks — so the report cannot drift out of sync with the
analysis. This README is generated from the same artifacts.
