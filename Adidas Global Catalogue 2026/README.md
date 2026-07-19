# Adidas Global Catalogue 2026 — Cross-Market Diagnostic

A Tier A, McKinsey-framework data-science project analysing an adidas product-catalogue snapshot across
10 markets (BR, CN, DE, FR, GB, IN, JP, KR, MX, US) at two dates (2026-02-28, 2026-03-09). Grain is
**size-level**: one row = product × size × market × snapshot. 44,888 rows.

## Key finding

> **adidas holds one price ladder across markets — the divergence that matters is in stock and range,
> not price or promotion.** Availability is the sharpest, most predictable cross-market gap; assortment
> blends a global core with local tilts; price is broadly harmonized (a tail exception); discounting is
> effectively absent (0.2% of SKUs).

Full deliverable: **[`reports/final_report.html`](reports/final_report.html)** (self-contained; open in a browser).

## Pipeline (governed by [`DOCS/STRUCTURE.md`](DOCS/STRUCTURE.md))

| Stage | Notebook | Output |
|---|---|---|
| 0 · Framing | [`notebooks/00_problem_framing.md`](notebooks/00_problem_framing.md), [`00_ghost_deck.md`](notebooks/00_ghost_deck.md) | question, issue tree, SCR, ghost deck |
| 1 · Ingestion | [`notebooks/01_ingestion.ipynb`](notebooks/01_ingestion.ipynb) | schema validation, reconciliation |
| 2 · Cleaning | [`notebooks/02_cleaning.ipynb`](notebooks/02_cleaning.ipynb) | `data/processed/adidas_clean.parquet` (44,888×46) |
| 3 · EDA | [`notebooks/03_eda.ipynb`](notebooks/03_eda.ipynb) | 5 descriptive cuts + stats (`reports/figures/`) |
| 4/6 · Predictive | [`notebooks/04_analysis.ipynb`](notebooks/04_analysis.ipynb) | availability classifier + price model (`models/`) |
| 7 · Reporting | `src/build_report.py` | `reports/final_report.html` |

Maps to the data assessment ([`DOCS/Adidas_Dataset_Analysis.md`](DOCS/Adidas_Dataset_Analysis.md)): §3 cleaning,
§4 framing, §5 descriptive, §6 predictive, §7 derived columns. Design system: [`DOCS/DESIGN.md`](DOCS/DESIGN.md).

## Reproduce

```bash
# deps: pandas, numpy, pyarrow, scipy, scikit-learn, matplotlib, seaborn, jupyter, joblib
python src/ingestion.py            # Stage 1 checks
python src/cleaning.py             # → data/processed/adidas_clean.parquet
python src/validation.py           # post-clean contract (PASS)
jupyter nbconvert --to notebook --execute --inplace notebooks/03_eda.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/04_analysis.ipynb
python src/build_report.py         # → reports/final_report.html
```

External reference tables (hand-built, [`data/external/`](data/external/)): `fx_rates.csv` (USD),
`category_map.csv` (53→17 canonical), `gender_map.csv`. **FX rates are placeholders** — replace with an
authoritative source before publishing absolute USD figures (see [`data/external/README.md`](data/external/README.md)).

## Scope & caveats

Catalogue panel only — **no sales, revenue, cost, or demand** data. No sales/margin/elasticity analysis,
no causal claims, no real time-series (two dates). All findings are **associational**. Iteration log:
[`CHANGELOG.md`](CHANGELOG.md).
