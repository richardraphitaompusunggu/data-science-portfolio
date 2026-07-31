# WHO Life Expectancy — a dataset that can rank countries but not the levers a funder would pull

**Tier A study** — descriptive (rung 1) → diagnostic (rung 2) → predictive (rung 4), built to the
McKinsey communication standard: answer-first, MECE, action titles, a "So What" under every exhibit.

📊 **[Read the final report](reports/final_report.html)** — self-contained HTML, all charts embedded.
*(GitHub shows `.html` as source; open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
or download and open locally.)*

---

## The question

> Can this WHO/UN panel identify which factors a country could **move** to raise life expectancy —
> or does it only rank countries that already differ; and what is the most defensible thing it can
> still be used for?

Framed as a brief to the analytics lead at a global-health funder deciding whether to prioritise
education, income or health-spending investments using the most-downloaded health panel on Kaggle.

## The finding

**Once you stop comparing rich countries to poor ones, ten of twelve factors lose their effect
entirely.**

Pooled R² of **0.889** collapses to a within-country **0.488** — country and year fixed effects
absorb **45%** of the apparent explanatory power. Log GDP shrinks **97%**, health-spend share
**94%**, adult mortality **89%**, and all three immunisation columns 83–87%.

Two survive:

| Factor | Within-country effect | 95% CI | |
|---|---:|---|---|
| **HIV/AIDS burden** | **−1.58 yr** per SD | [−2.08, −1.09] | survives |
| **Schooling** | **+0.95 yr** per SD | [+0.43, +1.46] | survives |
| log GDP, health spend, immunisation, BMI, adult mortality… | ~0 | crosses zero | not distinguishable |

In raw units schooling is worth **+0.34 years of life expectancy per additional year of schooling**
[+0.15, +0.53]. That the *same protocol* identifies two factors decisively is what makes the ten
nulls a calibrated statement about the data rather than a failure of method.

### The brief's own policy question, answered

*"For developing countries, does schooling or income associate with life-expectancy gains more than
health expenditure?"* Within-country, across 1,813 country-years:

| Lever | years per SD | 95% CI | verdict |
|---|---:|---|---|
| **Schooling** | **+1.30** | [+0.32, +2.29] | **distinguishable from zero** |
| Health-spend share | +0.11 | [−0.08, +0.29] | not distinguishable |
| Government health expenditure | +0.07 | [−0.17, +0.32] | not distinguishable |
| log GDP | −0.03 | [−0.14, +0.09] | not distinguishable |

Stable across all three cleaning variants.

---

## Before any of that: the file is digit-corrupted

**The character `0` has been deleted from every value in ten of nineteen numeric columns — including
the outcome.**

| Column | values containing a `0` digit | expected |
|---|---:|---:|
| `GDP` | **0 of 2,490** | 57% |
| `Population` | **0 of 2,286** | 44% |
| `Adult Mortality` | **0 of 2,928** | 15% |
| `BMI`, `Total expenditure`, `Polio`, `Diphtheria`, `Hepatitis B`, both `thinness` | **0** | 9–18% |

An estimated **6,217 cells** are wrong. It passes every null check, dtype check and range check.

**The target is corrupted too.** `Life expectancy` should have a `0` in the units position about 10%
of the time; it does so **1.0%** of the time (29 values vs ~293 expected), and carries a matching
**excess** of `.0` decimals (589 vs ~293). χ² p ≈ 1e-59 and 1e-67. `80.9` was written as `89.0` —
which is why the file's maximum life expectancy is **Italy 2004 at 89.0 years**.

The corruption is reversible where the panel is informative. Barbados' adult mortality reads
`…111, 19, 16, 14, 12, 11, 1, 98` and reconstructs to `…111, 109, 106, 104, 102, 101, 100, 98`.
**1,128 cells were repaired**, each with a ledger row. Recovery is partial and reported as such —
`GDP` and `Population` are **not** repaired, because at 57% and 44% corrupted the robust anchor is
itself mostly damaged, and a prototype that ignored that inserted zeros in the wrong places.

### Two more instrument defects

- **25 countries have no GDP for any of the 16 years** — and they are
  `United Kingdom of Great Britain and Northern Ireland`, `United States of America`,
  `Republic of Korea`, `Czechia`, `Viet Nam`, `Côte d'Ivoire`. Sixteen-of-sixteen missingness is a
  **failed join key**, not a country that reports irregularly. `dropna()` therefore deletes 50 of
  183 countries and lowers the Developed share of the sample from 20.8% to 18.7%.
- **`Status` mislabels six high-income countries** — France, Canada, Israel, Greece, Finland and
  Republic of Korea are all labelled *"Developing"*, while Bulgaria and Romania are *"Developed"*.

### And the strongest predictor partly *is* the outcome

`Income composition of resources` tops every correlation table (ρ = +0.91) and is documented as an
*income* index. Within a country it correlates **+0.86 with schooling** and only **+0.26 with log
GDP** — the opposite of what an income index would do. Adding life expectancy to a regression of it
on GDP and schooling lifts R² from **0.875 to 0.937**. It is excluded from every driver
specification.

> A predictor that beats every other predictor deserves an audit, not a promotion.

---

## The predictive claim needs two corrections

**Splitting by row instead of by country makes the model look 2.1× more accurate than it is.**

| Split | R² | MAE |
|---|---:|---:|
| Random 5-fold (what most notebooks use) | 0.982 | **0.74 yr** |
| `GroupKFold` by country | 0.940 | **1.55 yr** |

**And on a temporal holdout the model loses to doing nothing.**

| Trained ≤2011, scored 2012–15 | MAE |
|---|---:|
| Gradient boosting, 28 features | 1.05 yr |
| **Baseline: carry last year's value forward** | **0.88 yr** |
| Baseline: status-group mean | 6.09 yr |

The model is **19% worse than persistence**. Life expectancy is a slow-moving stock; a covariate
model must beat persistence to justify itself, and this one does not.

**Where it does earn its place:** for a country with *no* reported value, it predicts to **1.54
years** against **6.31** for the only available alternative — a 76% error reduction. That is the one
deployable product, and it ships with a per-region error table, because accuracy is worst in the
African region where covariate data is thinnest.

---

## Repository

```
notebooks/
  01_ingestion.ipynb    schema, lineage, the digit-integrity audit
  02_cleaning.ipynb     the eight defects, the ledgered repair, the unit contract
  03_eda.ipynb          one section per issue-tree branch, on both variants
  04_diagnostic.ipynb   Path A — effect sizes, FDR, the between/within decomposition
  05_modeling.ipynb     Path B — split schemes, feature blocks, baselines, fairness audit
  06_reporting.ipynb    Stage 7 — builds final_report.html
reports/final_report.html
```

Notebooks run in order from the raw CSV, import **no local modules**, and are seeded
(`RANDOM_STATE = 42`). Every number in the report is read from `reports/_key_figures.json` and the
CSVs written by notebooks 03–05 — none is typed by hand.

**Stack:** pandas 3.0.3 · scikit-learn 1.9.0 · statsmodels 0.14.6 · linearmodels 7.0 ·
scipy 1.18.0 · matplotlib 3.11.0

---

## What this analysis does not claim

**No coefficient here is causal.** Country fixed effects remove time-*invariant* confounders only.
They do not remove time-varying confounding, they do not resolve reverse causation (life expectancy
raises schooling as readily as the reverse), and they amplify attenuation from the measurement error
above. Path C is out of scope for this dataset: there is no treatment date, no assignment mechanism,
no instrument and no untreated donor pool. The HIV/AIDS result in particular coincides with
antiretroviral rollout and donor financing that this file does not measure.

Every headline result was recomputed on three dataset variants — as delivered, repaired, and with
the suspect cells nulled. The verdicts are unchanged on all three.

## Recommendation

1. **Stop using this file to prioritise between education, income and health-spending investments**;
   reclassify existing driver claims built on it as descriptive rankings.
2. **Add the digit-integrity audit and the all-periods-missing check to the ingestion template** —
   both are ten lines, and both fired here.
3. **Require a within-country specification and a carry-forward baseline** at review.
4. **Use the model only as a surveillance gap-filler**, at a measured 1.54-year error.

---

*Data: WHO Global Health Observatory + UN, merged and published on Kaggle (2018). 2,928
country-years, 183 countries, 2000–2015. The delivered file has 2,938 rows — ten "countries" appear
once, in 2013, and all ten have a null life expectancy.*
