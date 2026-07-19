# HR Analytics at 2M Rows — A Technique Benchmark

An end-to-end, McKinsey-framework data-science study on a **synthetic, rule-generated** HR dataset of
**2,000,000 employee records** across 7 European markets. The point of the project is *honesty at scale*:
building a credible production-scale workflow (cleaning → EDA → statistics → predictive benchmarks → report)
while correctly separating genuine description from artifacts the synthetic cleaning baked in.

> **Governing thought:** treat this data as a **technique benchmark, not a source of HR insight** — reproduce
> the documented cleaning, report honest descriptive metrics at scale, and demonstrate which predictive tasks
> are legitimate (attrition) versus artifacts (salary, performance).

## 📊 The report

**[`reports/final_report.html`](reports/final_report.html)** — a self-contained, answer-first McKinsey-style
report (SCR storyline, action titles, "So What" annotations, methodology appendix).
GitHub shows `.html` as source; view it via **[htmlpreview](https://htmlpreview.github.io/?https://github.com/richardraphitaompusunggu/data-science-portfolio/blob/main/HR%20Dataset%20Clean%20%26%20Raw/reports/final_report.html)** or download and open locally.

## 📓 Notebooks

| # | Notebook | Stage | What it does |
|---|---|---|---|
| 01 | [`notebooks/01_ingestion.ipynb`](notebooks/01_ingestion.ipynb) | Ingestion | Load 2M×16, schema-validate, log metadata (MD5/counts), reconcile raw↔clean |
| 02 | [`notebooks/02_cleaning.ipynb`](notebooks/02_cleaning.ipynb) | Cleaning | Reproduce the raw→clean fixes, validate business rules, derive 8 features |
| 03 | [`notebooks/03_eda.ipynb`](notebooks/03_eda.ipynb) | EDA | Five descriptive cuts (composition, pay, attrition, hiring, performance) |
| 04 | [`notebooks/04_analysis.ipynb`](notebooks/04_analysis.ipynb) | Stats + ML | Effect sizes + attrition/salary/performance model benchmarks |
| 05 | [`notebooks/05_reporting.ipynb`](notebooks/05_reporting.ipynb) | Reporting | Assemble the final HTML report |

## 🔑 Key findings

1. **Reproducible cleaning.** All three documented raw-defect classes — **3,333** null ratings,
   **3,333** non-positive salaries (min −€99,932), **164,794** age–experience violations — resolve to zero with
   the author's stated techniques (stratified fill, Level×Dept median fill, age-floor alignment). The clean file
   passes every business rule.
2. **Honest description at scale.** Workforce is bottom-heavy (**78%** Junior/Mid); pay is a clean **~4.5×**
   Director-over-Junior ladder in every department; attrition is flat across departments (10.3% ±0.3pp) but
   **triples at the Junior level** (16.6% vs ~5% senior); hiring compounds **~18%/yr** (2009–2025).
3. **Predictive honesty.** An attrition classifier extracts **modest, seniority-driven signal** (ROC-AUC ≈ 0.66);
   a salary model's near-perfect R² (≈0.95) is exposed as a **median-fill artifact**, not a win; a performance
   model correctly finds **no signal** (ties the majority baseline).

## ⚠️ Data-reality limits (stated, not worked around)

- **`Year == Hire_Year` for every row** → no snapshot date → current tenure & time-to-attrition are
  uncomputable (`Tenure_Years` is deliberately **not** derived).
- **No `Gender` column** → pay-equity / DEI analysis is out of scope.
- **Salary is a near-deterministic function of Job_Level × Department** (median-fill artifact).
- **`Status` is independent of department** (though it does vary with seniority/tenure).
- Currency unspecified (European cities suggest EUR).

## 🗂 Structure

```
notebooks/   01–05 pipeline (outputs baked in)
reports/     final_report.html  (+ figures/, local only)
src/         mck_style.py — McKinsey chart style
DOCS/        STRUCTURE.md · DESIGN.md · hr_clean_dataset_analysis.md  (local only)
archive/     hr_clean.csv · hr_raw.csv  (2M rows each, local only — not committed)
```

## ▶️ Reproduce

```bash
pip install -r requirements.txt
# place hr_clean.csv / hr_raw.csv in archive/
jupyter nbconvert --to notebook --execute --inplace notebooks/0*.ipynb
```

All notebooks set `random_state=42`; models use a 200k analytic sample. Only `.ipynb`, the HTML report, and
this README are version-controlled (raw data, figures, and `src/` are git-ignored to keep the repo light).
