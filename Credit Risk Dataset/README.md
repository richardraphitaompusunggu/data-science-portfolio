# Credit Risk Portfolio — Calibration Review

**A 50,000-loan, $165bn synthetic credit portfolio books $1.92bn of expected loss against $11.91bn
actually lost. Every chart in its standard risk report looks healthy.** This project finds out why,
and what the provision should be instead.

> **Tier A deliverable** built to the McKinsey communication standard — answer-first, MECE, action
> titles, every exhibit carrying a written "So What".
>
> **[Read the final report →](reports/final_report.html)**
> *(GitHub renders `.html` as source — open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
> or download and open locally.)*

---

## The answer

> The portfolio under-reserves by **$10.0bn** because expected loss is booked on a **1-year** PD that
> is *also* mis-calibrated at investment grade — and because both defects distort the **level** of risk
> rather than its **ranking**, every chart in the standard MI pack looks healthy.

| | |
|---|---|
| Booked expected loss | **$1.92bn** — covers 16% of realized loss |
| Realized loss | **$11.91bn** |
| Recommended provision | **$12.46bn** (95% CI $11.85–13.11bn) |
| Severe-scenario stressed EL | **$3.80bn** as reported → **$27.39bn** corrected |

---

## Four findings

**1 · The $9.99bn shortfall is two defects of near-equal size, not one.**
Expected loss is booked as `ead × pd_annual × lgd` — a *one-year* number — against losses that
accumulate over a ~4.5-year average term. Re-basing the **same** PD model to a lifetime horizon closes
**51% ($5.14bn)** of the gap with no new model at all. The remaining **49% ($4.85bn)** is genuine
mis-calibration, **$2.64bn** of it in investment grade. A single-defect diagnosis would have fixed the
wrong half.

**2 · The PD is calibrated at CCC and wrong by 64× at AAA.**
All seven grades fail a binomial exact test (FDR q < 0.05), and the miss ratio is **perfectly monotone
in credit quality** (Spearman ρ = −1.00): the safer a grade claims to be, the more wrong it is. Random
error does not do that. The book took **2,810 more defaults** than its own model expected.

**3 · The published stress test is arithmetically broken.**
Its **zero-shock baseline** — GDP shock 0, rate shock 0, PD multiplier exactly 1.0 — reports expected
loss *falling 10.2%*. Root cause: `expected_loss_stress` reconciles to `ead × pd × lgd` for 100% of
rows while `expected_loss_base` reconciles for 0%. Correcting the base restates the baseline to
**+0.000%** and raises every scenario by **18.4pp**.

**4 · Fix the parameter, not the model.**
The obvious remedy — retrain the PD — was tested and rejected. The incumbent `pd_annual` posts the
**highest AUC of any model tried** (0.883 vs 0.864 for XGBoost; a rating-only logistic reaches 0.880).
Discrimination is saturated. Retrained models are also *worse* calibrated out-of-sample, over-predicting
2.1× because they learned a 15.6% base rate and were asked to score cohorts defaulting at 7.8%.

### Two findings that contradict the supplied data review

- **Collateral drives severity, not frequency.** The review warns collateral "barely moves the needle"
  and predicts a weak LGD fit. True for *default frequency* (flat within 0.2pp) — false for *LGD*,
  which runs 45% secured → 66% unsecured, monotone in seniority, with collateral explaining **40%** of
  LGD variance. **LGD is the half of the framework that works**, which is what keeps the recommendation
  narrow.
- **The vintage curves need no extrapolation.** The review says recent cohorts "haven't reached 60
  months yet" and should be extrapolated. In fact all 36 cohorts are already **simulated forward** to
  month 60 — 2023Q4's 60th month falls in 2028Q4. Worse, the curves are driven by **one calendar
  shock**: 19 of 21 pre-2020 cohorts peak in **2020Q2**, at anywhere from 6 to 60 months on books.

---

## Notebooks

| # | Notebook | Stage | What it establishes |
|---|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | 1 | 5 tables, SHA-256 lineage, **28 schema rules**, and the three-window default reconciliation |
| 02 | [Cleaning](notebooks/02_cleaning.ipynb) | 2 | Structural-missingness policy; the four derived columns that carry the findings |
| 03 | [EDA](notebooks/03_eda.ipynb) | 3 | Hypothesis-driven, one section per issue-tree branch; the gap decomposition |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | 5a (Path A) | Binomial calibration tests, stress root-cause, two-way vintage trend test, Markov cross-check |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | 4 → 6b (Path B) | Model bake-off, survival, LGD GLM, the restated provision, fairness audit |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | 7 | Builds [`final_report.html`](reports/final_report.html) — every figure read from `_key_figures.json`, none hardcoded |

---

## Method notes

**Leakage discipline.** 13 columns are banned as features. The most dangerous is `pd_annual` — the
incumbent model's *output*. Including it would produce a model that predicts the thing being disproved.
It appears only as **Model 0c, the benchmark**.

**Origination-time split, not random.** Train on 2015Q1–2021Q4 originations, test on 2022Q1–2023Q4.
The test cohorts default at **7.8%** against **15.6%** in training — a genuine regime shift caused by
the 2020Q2 shock, not sampling noise. Both splits are reported.

**Two targets, both modelled.** `defaulted` includes **662 defaults dated after the 2024-12 data cut**,
out to 2033-09 — this dataset is a forward simulation, not a historical extract. `defaulted_by_2024_12`
excludes them. Divergence reported, not resolved silently.

**Statistical discipline.** At n = 50,000 every p-value is astronomically small, so **effect sizes lead
every claim and p-values appear only in tables**. Benjamini–Hochberg FDR across grade families; Holm on
pairwise LGD contrasts; Wilson intervals on every rate. The binomial-independence assumption is
**known to fail** (defaults cluster in 2020Q2) and that failure is stated in the report rather than
omitted.

**Robustness, pre-committed.** The headline was to be withdrawn if it depended on the horizon
convention. Compound and linear lifetime conversions agree within **0.7% across AAA–BBB**, where the
finding lives.

**Path C (causal) is explicitly out of scope** — no intervention, no assignment mechanism, no pre/post
design. The macro work is labelled sensitivity, never effect.

---

## Honest limitations

- **The data is synthetic.** No figure here describes a real institution. The transferable artifacts
  are the *methods* — the calibration test, the reconciliation harness, the zero-shock control, and the
  two-way trend test.
- **No protected-subgroup fairness audit is possible.** The data carries no age, gender, geography,
  ownership or firm-size attribute. For a model that allocates capital and prices credit this is a
  **blocking gap for deployment**, and it is stated in the main report rather than the appendix.
- **The retrained model has an adverse fairness finding.** Calibration degrades monotonically with
  borrower quality — it over-predicts default **8.4×** for the highest credit-score quintile, where its
  AUC falls to 0.62. Recommendation is to **recalibrate the parameter, not deploy the model**.
- **No join key exists between the five tables**, so no multi-level model is possible and the Markov
  cross-check cannot be reconciled to the loan book.
- **Sector, loan type and collateral carry no PD signal by construction** (all span < 2pp of default
  rate), so no segment-level recommendation appears anywhere.
- **The monitoring plan is unvalidated** — a static file with no live scoring.

---

## Reproducing

```bash
pip install pandas==3.0.3 numpy==2.5.1 scikit-learn==1.9.0 scipy==1.18.0 \
            statsmodels==0.14.6 matplotlib==3.11.0 seaborn==0.13.2 \
            pyarrow==25.0.0 xgboost==3.3.0
jupyter lab     # run notebooks 01 → 06 in order
```

`random_state = 42` throughout; all six notebooks pass restart-and-run-all. Source CSVs
(`loan_portfolio`, `credit_ratings`, `vintage_analysis`, `macro_stress_scenarios`,
`portfolio_metrics`) are not committed — only notebooks, the HTML report and this README ship.

`lifelines` is deliberately **not** used — survival analysis runs on `statsmodels.duration`
(Kaplan–Meier, log-rank, Cox) so a fresh clone works against the pinned dependency set.
