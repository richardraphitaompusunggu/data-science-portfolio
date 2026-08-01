# Loan default prediction — a dataset that scores 0.94 AUC and predicts nothing

**Tier A study** — descriptive (rung 1) → diagnostic (rung 2) → predictive (rung 4), built to the
McKinsey communication standard: answer-first, MECE, action titles, a "So What" under every exhibit.

📊 **[Read the final report](reports/final_report.html)** — self-contained HTML, all charts embedded.
*(GitHub shows `.html` as source; open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
or download and open locally.)*

---

## The question

> This dataset reliably produces a ~0.90-accuracy, ~0.94-AUC default classifier. **Is that a model a
> lender could deploy, or an artifact of how the file was built?**

Framed as a brief to the **model risk lead at a consumer lender** who has been handed a candidate
applicant-scoring model whose headline metrics clear every threshold in the model approval policy.

## The finding

**It is an artifact.** The same Random Forest, same features, same seed:

| Split scheme | AUC | Accuracy | Recall on defaulters |
|---|---:|---:|---:|
| Random row split — *the number everyone reports* | **0.9358** | 0.8978 | 0.4766 |
| Grouped by applicant — *honest* | **0.4913** | 0.8770 | 0.0010 |
| 36,007 unique applicants — *nothing can leak* | **0.4987** | — | — |

The only difference between the first two rows is whether copies of the same applicant are allowed to
sit on both sides of the split. On the 36,007 applicants who appear exactly once — where a plain
`StratifiedKFold` is beyond dispute, because there is nothing to leak and no grouping involved —
out-of-fold AUC is **0.4987, 95% CI [0.4917, 0.5059]**.

Three disjoint designs, one answer: **chance**.

Accuracy under the honest split falls to 0.8770, which is exactly the majority-class rate. The model
has stopped saying anything at all.

## Before any of that: the file is 42,007 applicants stretched to 252,000 rows

`df.duplicated()` returns **zero**, because `Id` is unique on every row. Drop `Id` and **85.71% of
the file is copies**:

- **36,007** applicant profiles appear once
- **6,000** appear **27–40 times each**, covering 215,993 rows
- **nothing** appears between 2 and 26 times — a bimodal group-size distribution with an empty
  middle is a generator loop, not a data-entry accident

The replicated copies do not even agree on the label. Of the 6,000 replicated profiles, 4,817 are
unanimously non-default, **zero** are unanimously default, and 1,183 are labelled both 0 and 1 across
their own byte-identical copies, at a within-profile positive rate averaging 0.56. The label was
drawn per *row*, not attached to the applicant.

### Three more defects, all verified

| Defect | Evidence |
|---|---|
| **`Id` is the strongest predictor in the file** | HistGB on `Id` alone: **AUC 0.6744**. All eleven real attributes together: 0.5184. The file is sorted into two blocks defaulting at 11.0% and 20.2% |
| **The two blocks are the same applicants** | Income indistinguishable (KS p = 0.17); Age, Experience and both tenure columns agree to within 1.5%. Only the label rate differs — by 83% |
| **Train and test speak different languages** | Train writes `Software_Developer`, test writes `Software Developer`. 23 of 51 professions and 8 of 29 states differ by a separator alone → **72.45% of test rows** carry a category no encoder has seen |
| **The "holdout" is 85.72% already in train** | 24,003 of 28,000 rows have an exact feature-vector twin in the training file — and the vocabulary mismatch above hides it, reporting 23.7% instead |
| **The submission template is random noise** | 12.83% positive, AUC 0.504 against every feature. It is a formatting stub, not a benchmark — so **no metric in this project is computed against `Test Data.csv`** |

## The methodological result: a significance test is only as valid as its unit of independence

The data assessment asks for default rate by profession, state and city. Run the usual way, all three
are overwhelming — χ² returns p = 5e-98 for `Profession` and p ≈ 0 for `CITY`. Run the permutation at
the **applicant** level instead of the row level, changing nothing else:

| Column | Observed spread | Row-level placebo | Applicant-level placebo | BH-adjusted |
|---|---:|---|---|---|
| `Profession` | 8.3 pp | p < 0.001, **z = +19.8** | **p = 0.501** (z = −0.1) | 0.501 |
| `STATE` | 17.0 pp | p < 0.001, **z = +14.9** | **p = 0.131** | 0.197 |
| `CITY` | 30.0 pp | p < 0.001, **z = +32.0** | **p = 0.090** | 0.197 |

None survives. Replication multiplies *n* sixfold without adding one bit of information, and a χ²
statistic scales with *n*. `CITY`'s median category holds 791 rows and **132 applicants** — and it is
the smaller number that governs the standard error.

The same reversal hits the low-cardinality columns. Home ownership looks worth −3.58 pp
[−5.07, −1.95] pooled; on the clean 36,007-applicant sub-sample it is **+0.82 pp [−1.05, +2.70]**.
Car ownership does worse than vanish — it **changes sign significantly in both directions** (−1.72 pp
pooled, **+1.00 pp [+0.07, +1.89]** clean). Not one attribute in the file survives an honest
univariate test.

## The fairness finding

This is a model that would score people for credit, so the audit is mandatory — and the result is
stranger than bias. **The model declines the groups that default least.** Nine of seventeen subgroups
are declined more often than they default, including the four lowest-risk groups in the file:

| Group | Actual default rate | Share the model declines |
|---|---:|---:|
| owns their home | 8.98% | **13.40%** |
| neither rents nor owns | 9.95% | **18.17%** |
| married | 10.25% | **13.80%** |
| owns a car | 11.10% | **12.78%** |

Across all subgroups the correlation between how often a group defaults and how often it is declined
is **+0.15** — essentially unrelated. That is disparate impact with no accuracy defence available:
the usual justification for declining a group more is that it defaults more, and here the opposite
holds. Sixteen of seventeen subgroups rank at chance.

## The transferable output: a four-check pre-modelling audit

Each check is under ten lines. All four fail on this file.

| # | Check | Why it exists |
|---|---|---|
| 1 | `duplicated()` **excluding identifier columns** | a surrogate key defeats the standard duplicate test |
| 2 | Score the row identifier as a feature | catches sorted files and index leakage |
| 3 | Compare category vocabulary across train and test *before* fitting any encoder | catches silent scoring failures |
| 4 | Check holdout membership in train | catches a holdout that is not held out |

Plus the rule the placebo section establishes: **run every group-difference test at the unit of
independence, calibrated by a permutation of that unit** — not of rows.

## What this analysis does not claim

**No null result here transfers to real credit portfolios.** Income and employment stability are
well-established credit predictors in real lending books. This file is synthetic: `Income`, `Age` and
`Experience` are mutually uncorrelated (|r| ≤ 0.007) and near-uniform, and 32,189 rows (12.77%) imply
a career starting before age 18 — the youngest at 1. Every claim is bounded to *this instrument*.

Causal inference is out of scope and stays out: no treatment, no assignment mechanism, no timestamps,
no instrument, no discontinuity.

## Notebooks

| # | Notebook | What it does |
|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | Schema contract, lineage + SHA-256, the **four-check audit** (all fail), submission-template diagnostic |
| 02 | [Cleaning](notebooks/02_cleaning.ipynb) | Replication mechanism proved four ways, vocabulary reconciliation, the **profile index**, three dataset variants |
| 03 | [EDA](notebooks/03_eda.ipynb) | One section per issue-tree branch, every bivariate view at **row and applicant level side by side** |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | Placebo calibration at both units, effect sizes with bootstrap CIs, BH correction, verdict per branch |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | Four model families × three split schemes, four baselines, ablation, **fairness audit**, submission |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | Builds `reports/final_report.html` — every number read from `_key_figures.json`, nothing hardcoded |

## Reproducing

```bash
pip install -r requirements.txt          # pandas 3.0.3, scikit-learn 1.9.0, ... (pinned)
# place the three Kaggle CSVs in archive/, then run notebooks 01 → 06 in order
```

`RANDOM_STATE = 42` throughout; restart-and-run-all verified from the raw CSVs. The notebooks import
**no local modules** — chart styling, the audit checks and the placebo harness are defined inline in
each notebook, so a fresh clone runs on the pinned packages alone.

**Data:** [Loan Prediction Based on Customer Behavior](https://www.kaggle.com/datasets/subhamjain/loan-prediction-based-on-customer-behavior)
(Kaggle, Univ.AI hackathon — synthetic). Raw CSVs are not committed.

**Governed by:** [`DOCS/STRUCTURE.md`](DOCS/STRUCTURE.md) (pipeline and communication standard),
[`DOCS/DESIGN.md`](DOCS/DESIGN.md) (visual system), and the project's
[`IMPLEMENTATION_PLAN.md`](IMPLEMENTATION_PLAN.md).
