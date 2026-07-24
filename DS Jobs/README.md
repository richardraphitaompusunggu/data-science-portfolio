# Glassdoor Data-Science Jobs — Data-Source Review

**The most widely used public dataset for benchmarking data-science pay cannot benchmark pay at all.**
Its 672 postings carry only **30 distinct salary values**, and each one is a perfectly contiguous block
of rows — the fingerprint of 30 separate Glassdoor searches, each with its own salary filter, stitched
together. The "salary" column is the scrape query, not the job. This project proves that four
independent ways, then salvages what the file genuinely supports: a **role-and-skill market map** and a
shippable **role-family tagger** (macro-F1 0.71).

Framed for a **Head of Talent Analytics** ahead of a compensation-review and job-architecture cycle.
The decision it feeds: *do we admit this source into comp benchmarking (no), and what do we use it for
instead (role and skill structure).*

> **Note:** the salary-prediction model — this dataset's canonical use, reproduced in hundreds of public
> notebooks — was **kept and built in full** (seven model families, tuned, both validation protocols),
> precisely so the rejection is the verdict of a properly built model, not a weak attempt.

## Four findings

**1 · The salary field is a scrape-batch label, not a property of the job.**
672 postings carry 30 distinct salary strings, and every one occupies an unbroken run of row indices
(density 1.000 for all 30). A permutation test rejects chance at **p < 0.0001** — a random assignment
would leave ≈650 blocks, not 30. Formally, **η²(salary ~ batch) = 1.000**: the batch *is* the target.

**2 · The same posting sells for up to $179K apart depending only on which search returned it.**
Of 151 repeated postings, **149 (98.7%) carry two or more different salary bands** — same company, same
title, same description text. Median disagreement **$28K** (95% CI $24–34K). A quantity that takes five
values for one unchanged job is not measuring that job.

**3 · Nothing predicts salary, and this is a *powered* null — not a failed search.**
Every job attribute falls below the effect its own sample size could detect at 80% power; seniority
carries a **$1.1K discount** (Cohen's d = −0.03, where d = 0.29 was detectable) and company rating
correlates **r = 0.001**. The only cut that survives is *estimate provenance* — Glassdoor vs Employer
est., which is scrape metadata, not a job attribute. Seven tuned models, both protocols, top out at
**R² ≈ −0.01** — worse than predicting the mean, and short of a pre-registered 0.20 acceptance bar.

**4 · The role-and-skill structure is real, and one model ships.**
The same file that carries no salary signal carries strong role signal. Tableau appears in **51%** of
analyst postings and **0%** of ML-engineer postings; a PhD is demanded by **57%** of ML-engineer
postings versus **6%** of analyst postings. Role family is recoverable from description text at
**macro-F1 0.71** (81% accuracy) versus a 0.16 majority baseline — useful auto-tagging for a title
field that is 50% the single string "Data Scientist".

### Findings that contradict the supplied data review

- **Company names need no identity resolution.** The review lists this as a gap. Stripping the rating
  appended to `Company Name` (present in 622/672 rows) resolves **432 → 432** unique names — *zero*
  collisions. No fuzzy matching is performed; it would manufacture false merges to fix a problem that
  does not exist.
- **Missingness is worse than a `-1` count shows.** The review names the `-1` sentinel. There is a
  second, independent one: `"Unknown / Non-Applicable"`. `Revenue` carries both, putting it at **35.7%
  missing** rather than the 4.0% a `-1` count reports. `pandas.isnull()` sees zero missing across the
  whole file.

---

## Notebooks

| # | Notebook | Stage | What it establishes |
|---|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | 1 | SHA-256 lineage, 11 schema rules, the sentinel-missingness audit, and the **batch-structure integrity gate** (η² = 1.000 asserted at load) |
| 02 | [Cleaning](notebooks/02_cleaning.ipynb) | 2 | Two sentinel systems converted and counted; salary/title/location/skill parsing; the **three-level duplicate treatment** — the middle level deliberately preserved as evidence |
| 03 | [EDA](notebooks/03_eda.ipynb) | 3 | Hypothesis-driven, one section per issue-tree branch; the within-batch control that rules out a simple sort |
| 04 | [Forensic diagnostic](notebooks/04_forensic.ipynb) | 5a (Path A) | Permutation test, effect sizes + minimum detectable effects, ε² bias correction, FDR, the replicate-variance ceiling, and the **pre-registered acceptance test** |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | 4 → 6 (Path B) | 7-family salary bake-off (both protocols, tuned), the role-family tagger, the rating model, permutation importance, fairness audit |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | 7 | Builds [`final_report.html`](reports/final_report.html) — every figure read from `_key_figures.json`, none hardcoded |

> **Viewing the report:** GitHub shows `.html` as source. Open it via
> [htmlpreview.github.io](https://htmlpreview.github.io/) or download and open locally.

---

## Method notes

**Leakage discipline.** `Salary Estimate`, its parsed aliases, `batch_id`, and `index` are banned as
salary-model features. `index` is the sharpest trap — via the block structure it is a *perfect*
predictor of the target, scoring ~1.0 while learning nothing. `Job Title` is banned from the tagger
because the label is derived from it.

**Grouped validation, not random.** 27% of rows are re-observations of a posting already in the file,
so a random 80/20 split leaks **43%** of test postings into training. Every model uses `GroupKFold` on
posting identity; the random-split score is reported beside it only as the *leakage delta* — which
here buys nothing, because a repeated posting carries a *different* salary.

**A ceiling, computed before modelling.** Because identical postings carry different bands,
within-replicate variance caps R² at **≈0.46** on the rows where the floor is observable — no feature
set can exceed it. The report quotes 0.46, not the optimistic 0.71.

**Pre-registration.** The salary acceptance test (grouped R² ≥ 0.20) was written to
`reports/_preregistration.json` **before any model ran**, so the reject verdict cannot be
reverse-engineered from the result.

**Statistical discipline.** Effect sizes and confidence intervals lead every claim; p-values live in
tables. ε² corrects η²'s upward bias with many groups; Benjamini–Hochberg FDR across the attribute
family; Wilson intervals on proportions. Every "no effect" ships with the effect that *was* detectable.

**Fairness.** The tagger affects access to opportunities, so a protected-subgroup audit is mandatory —
and **impossible**, because the file has no applicant or demographic attribute. That impossibility is a
main-narrative limitation. Runnable audits were run: tagger error concentrates in the smallest role
families, and a PhD gating 57% of ML-engineer postings is a documented access barrier.

**Path C (causal) is explicitly out of scope** — no intervention, no assignment mechanism, no time
dimension. Skill–role associations are labelled association, never effect.

**Provenance.** These are Glassdoor *estimates*, not observed pay (20 rows are employer-supplied on a
different basis); there is no date field, so nothing here is a trend and the ~2020 vintage is inferred.

---

## Reproduce

```bash
pip install -r requirements.txt      # pandas, scikit-learn, statsmodels, xgboost, ...
# place Uncleaned_DS_jobs.csv/ in the project root, then run in order:
jupyter nbconvert --to notebook --execute --inplace notebooks/0*.ipynb
```

Seed `42` throughout. Only notebooks, `final_report.html`, and this README are committed; raw data,
`data/`, `models/`, `DOCS/`, and figures stay local (`.gitignore`).

**Data:** Kaggle — *Uncleaned Data Science Jobs* (Glassdoor scrape, 672 postings × 15 columns).
