# Boston Housing — What Explains the Gap Between Tract Home Values?

A Tier A diagnostic and predictive analysis of 506 Boston census tracts (1970), built to the
McKinsey communication standard: Pyramid Principle, MECE structuring, SCR storylining, answer-first
delivery.

**📄 [Read the final report →](reports/final_report.html)** — self-contained HTML, light and dark themes, no external dependencies.

---

## The question

> *Holding 1970 Boston conditions fixed, which structural, environmental, and socioeconomic tract
> attributes explain the variation in median home value — and how much of that variation can be
> explained at all?*

Framed as a brief to a **metropolitan planning council** that must direct a fixed
neighbourhood-investment budget across tracts whose median values span **$5k to $50k** around a
**$21.2k** median.

## The answer

> **Variation in Boston tract home values is driven overwhelmingly by two attributes — social
> composition and dwelling size — whose effects on value are strongly non-linear; environmental and
> fiscal factors track value only because they track those two, and are therefore diagnostic markers
> rather than independent levers.**

| # | Key Line | Evidence |
|---|---|---|
| 1 | **Two attributes explain most of the spread** | `LSTAT` alone accounts for **54%** of variance and `RM` for **48%**; with their derivatives they hold **86%** of model importance vs 14% for everything else |
| 2 | **Both relationships bend, so linear models misprice the tails** | Allowing non-linearity lifts test R² from **0.808 → 0.922** — **11.4 points** |
| 3 | **Environmental and fiscal factors are inherited, not independent** | After controlling for `LSTAT` and `RM`, `NOX` retains just **7%** of its raw association, `INDUS` 13% |

**Recommendation:** commission site assessments of the **18 tracts** valued more than $2k below
their predicted value — *not* allocation against pollution or tax-burden indicators.

## Results

| Model | CV R² | Test R² | Test RMSE | vs baseline |
|---|---|---|---|---|
| **Gradient Boosting** | **0.852** | **0.922** | **$2.49k** | **−72%** |
| Random Forest | 0.824 | 0.898 | $2.85k | −68% |
| OLS | 0.762 | 0.808 | $3.90k | −56% |
| Lasso | 0.761 | 0.806 | $3.93k | −56% |
| Ridge | 0.764 | 0.799 | $3.99k | −55% |
| Baseline (mean) | −0.030 | −0.007 | $8.93k | — |

Regularisation bought nothing here: Ridge and Lasso land within noise of plain OLS, because the
collinearity they exist to handle (`TAX`–`RAD`, r=0.91) was already resolved at the specification
stage by dropping `TAX` from the linear model. The gain came entirely from **non-linearity**, not
from penalisation.

Validation: repeated 10-fold CV (30 fits) for selection; 102 held-out tracts scored **once**.

> **Note:** test R² (0.922) exceeds CV R² (0.852). With 102 test tracts that is sampling variation,
> not a better model — **the CV figure is the trustworthy one**, and the report says so.

## Three findings that changed the analysis

**1. The target is censored.** 16 tracts sit at exactly `MEDV = 50.0` — a survey ceiling, not an
observed maximum. They are also the *most desirable* tracts in the sample (7.5 rooms vs 6.3;
4.4% low-status vs 12.9%), so dropping them would delete the top of the market. They were retained
and flagged, every high-end estimate is reported as an **attenuated lower bound**, and an n=490
sensitivity run confirms the conclusions hold.

**2. `RAD` is an index, not a distance.** It runs 1–8 then jumps to 24. Fitting a linear term asserts
that 23→24 means the same as 1→2. It is treated as an ordered categorical throughout. Relatedly,
`TAX` correlates with it at **0.91**, so no "tax burden" conclusion is identifiable from this data
and none is offered.

**3. Excluding the race-derived column is not enough.** Dropping `B` costs **0.003 R²** — which is
the reason to worry, not to relax. A model reconstructs `B` from ordinary tract attributes at
**R² ≈ 0.40**. Removing the protected attribute made the model *blind to race, not neutral about
it*, and the report carries that in the main narrative rather than an appendix.

## Pipeline

| Notebook | Stage | What it does |
|---|---|---|
| [`01_ingestion.ipynb`](notebooks/01_ingestion.ipynb) | 1 | Loads the headerless file with the UCI schema, SHA-256 lineage, 14/14 range checks, governance classification |
| [`02_cleaning.ipynb`](notebooks/02_cleaning.ipynb) | 2 | 0 rows dropped, 0 imputed — three structural decisions (censoring, `RAD` dtype, crime tail) with a full cleaning log |
| [`03_eda.ipynb`](notebooks/03_eda.ipynb) | 3 | Hypothesis-driven EDA against a MECE issue tree; curvature quantified, VIF, partial correlations |
| [`04_analysis.ipynb`](notebooks/04_analysis.ipynb) | 5a + 4/5b/6 | FDR-corrected tests with effect sizes, 7 engineered features, 6-model bake-off, fairness audit, residual targeting |
| [`05_reporting.ipynb`](notebooks/05_reporting.ipynb) | 7 | Assembles the report from a machine-written figures file; 12 automated deliverable checks |

**Analysis path:** Path A (diagnostic) → Path B (predictive). **Path C (causal) is deliberately not
run** — a single 1970 cross-section has no assignment mechanism, instrument, or pre/post structure,
so no intervention claim is supported. The recommendation is framed as *targeting*, never as
intervention sizing.

## Reproducing

Place the source `housing.csv` in the project root, then:

```bash
pip install pandas==3.0.3 numpy==2.5.1 scikit-learn==1.9.0 scipy==1.18.0 \
            statsmodels==0.14.6 matplotlib==3.11.0 seaborn==0.13.2 pyarrow==25.0.0
jupyter lab      # run notebooks 01 → 05 in order
```

Seed `42` throughout and relative paths only. **The notebooks import no local modules** — chart
styling and the dtype contract are defined inline in each one, so a clone runs with nothing but the
source CSV and the packages above. (Only notebooks, the report, and this README are committed; data,
figures, and helper scripts stay local by design.)

## Limitations

- **1970 data.** Not valid for present-day valuation; values carry no inflation adjustment.
- **Association, not causation.** The residual shortlist says *where to look*, never *what to change*.
- **Censored target.** All high-end effects are lower bounds.
- **Not fit for allocating between demographic groups** — see finding 3 above.
- **No peer review.** Recorded as an open gate item rather than silently ticked.

## Ethical note

This dataset contains `B`, a race-derived column, and is widely criticised for it — scikit-learn
deprecated its loader on those grounds. It is used here for methodological demonstration only, with
that column **excluded from all models** and audited explicitly (exclusion cost, proxy
reconstruction, error by subgroup). Findings describe 1970 conditions and carry no present-day
policy implication.

---

**Source:** Harrison, D. & Rubinfeld, D.L. (1978), "Hedonic prices and the demand for clean air",
*Journal of Environmental Economics and Management* 5, 81–102 · via the UCI Machine Learning
Repository · 506 census tracts, Boston SMSA, 1970.
