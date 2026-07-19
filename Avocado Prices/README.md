# 🥑 Hass Avocado Price Analysis (2015–2018)

Drivers, forecasting, and the causal pricing lever for US Hass avocado retail prices —
built end-to-end against the McKinsey-style pipeline in [`DOCS/STRUCTURE.md`](DOCS/STRUCTURE.md).

> **Governing thought:** Avocado price is structured by four learnable drivers — **type, region,
> season, supply** — forecastable within **±$0.11** a week ahead. But the pricing lever is causal and
> **negative**: demand is **price-elastic (−1.43)**, so a 10% price increase cuts volume ~14% and
> revenue ~4%. Grow through **product mix, premium-market allocation, and event-timed promotions** —
> not headline price hikes.

---

## Business questions → analytical paths

| # | Question | Path | Answer |
|---|---|---|---|
| 1 | What are the **price patterns**? Does price **differ by region / type**? | **A · Diagnostic** | Organic **+42.8%** premium (d=1.56); region explains **17.5%** of variance (~2× city spread); strong winter-trough / autumn-peak seasonality; price↔volume ρ = **−0.61** |
| 2 | Can we **predict** the price? | **B · Predictive** | 1-week-ahead forecast within **±$0.11** (R²=0.78); SARIMAX 26-week organic MAPE ~2%, conventional ~17% |
| 3 | What happens if we **change** the price? | **C · Causal** | Demand is **price-elastic (IV −1.43)** → raising price **loses** revenue |

---

## Key findings

- **Type is the biggest driver.** Organic median ≈ $1.63 vs conventional ≈ $1.13 — a 42.8% premium,
  *large* effect (Cohen's d = 1.56), distributions barely overlap.
- **Geography adds a ~2× spread.** Houston (~$0.98) → Hartford–Springfield (~$1.80); ANOVA F=73, η²=0.175.
- **Season & supply are predictable.** Prices trough Jan–Feb, peak Sep–Oct; high-volume weeks are
  low-price weeks (Spearman ρ = −0.61). 2017 was an anomalous high-price year (supply shortfall).
- **Forecasting works — short horizon best.** A gradient-boosting model using only *past* data +
  known-ahead demand events forecasts next week within ±$0.11 (56% better than seasonal-naive).
  Multi-week SARIMAX nails the stable organic line (~2% MAPE) but struggles with the volatile
  conventional line.
- **The pricing lever is causal and negative.** Correcting for supply/demand simultaneity with an
  instrumental-variables design (first-stage F=350) gives an own-price elasticity of **−1.43** —
  demand is elastic, so price increases reduce total revenue.

Full answer-first write-up (exhibit-by-exhibit): **[`reports/final_report.html`](reports/final_report.html)**.

---

## Project structure

```
Avocado Prices/
├── README.md                     # you are here
├── requirements.txt              # pinned dependencies
├── DOCS/
│   └── STRUCTURE.md              # the pipeline / reporting standard this project follows
├── avocado.csv/avocado.csv       # original source file (as delivered)
├── data/
│   ├── raw/avocado_raw.csv       # immutable copy (never edited)
│   └── processed/avocado_clean.csv  # output of Stage 2 cleaning
├── notebooks/                    # ← reviewable walkthroughs (executed, with outputs)
│   ├── 01_get_know_clean_eda.ipynb    # Part 1: get → know → clean → EDA
│   └── 02_deep_dive_paths_abc.ipynb   # Part 2: Path A / B / C deep dive
├── src/                          # reproducible analysis modules
│   ├── analysis_paths_a.py       # Stages 1–3 + Path A statistical testing
│   ├── analysis_path_b.py        # Path B concurrent (diagnostic) price model
│   ├── forward_forecast.py       # Path B forward 1-week-ahead model + event/holiday features
│   ├── sarimax_forecast.py       # Path B classical multi-week SARIMAX forecast
│   ├── causal_study.py           # Path C price-elasticity IV / 2SLS study
│   ├── build_report.py           # assembles reports/final_report.html
│   ├── build_notebook.py         # generates + executes notebook 01
│   └── build_notebook_part2.py   # generates + executes notebook 02
└── reports/
    ├── final_report.html         # McKinsey-style stakeholder report (self-contained)
    ├── figures/                  # 15 exported exhibits (PNG)
    └── *_summary.txt             # machine-readable result summaries
```

---

## How to run

**1. Set up the environment** (Python 3.11+; developed on 3.14):

```bash
python -m venv .venv
source .venv/Scripts/activate        # Windows Git Bash;  ...\.venv\Scripts\Activate.ps1 in PowerShell
pip install -r requirements.txt
```

**2. Review the notebooks** (already executed — open and read, or re-run):

```bash
jupyter lab notebooks/01_get_know_clean_eda.ipynb      # start here
jupyter lab notebooks/02_deep_dive_paths_abc.ipynb     # then the deep dive
```

**3. Or reproduce everything from the command line** (run from the project root):

```bash
python src/analysis_paths_a.py      # Stages 1–3 + Path A  → cleans data, writes figures 01–06
python src/analysis_path_b.py       # Path B concurrent model → figures 07–09
python src/forward_forecast.py      # Path B forward forecast → figures 10–12
python src/sarimax_forecast.py      # Path B SARIMAX          → figure 13
python src/causal_study.py          # Path C elasticity (IV)  → figures 14–15
python src/build_report.py          # assemble reports/final_report.html
```

> Run `analysis_paths_a.py` **first** — it produces `data/processed/avocado_clean.csv` that the
> other scripts and notebooks depend on.

**Regenerate the notebooks** (optional — the `build_notebook*.py` scripts overwrite the `.ipynb`
files, so edit the builders, not the notebooks, if you want changes to persist):

```bash
python src/build_notebook.py
python src/build_notebook_part2.py
```

---

## Data notes & caveats

- **Grain:** one row per week × region × type. *Average Price* is **per single avocado** (even when
  sold in bags). PLU 4046 / 4225 / 4770 = small / large / xlarge Hass; greenskins excluded.
- **`region` mixes granularities** — cities (e.g. `Albany`) sit alongside aggregates (`TotalUS`,
  `West`, `California`). The `is_aggregate_region` flag separates them; city-level analyses exclude
  aggregates to avoid double-counting.
- **Date range** is **2015–2018**, despite the source description saying "2018".
- **Causal caveat:** the −1.43 elasticity rests on the IV **exclusion restriction** (other-markets'
  price affects local demand only through local price). It is a well-identified estimate under a
  stated assumption — not incontrovertible. The gold-standard next step is a **geo-based price
  experiment**.
- **No true promotion field** exists; promotions are proxied by lagged volume surges + known demand
  events (Super Bowl, Cinco de Mayo, etc.).

---

## Suggested next steps

1. **Heterogeneous elasticities** by type × region (causal ML / CATE) for segment-specific pricing.
2. **Geo-based price experiment** to validate the −1.43 elasticity causally.
3. Integrate a **real promotion calendar** to sharpen both forecast and elasticity estimates.
4. Operationalize the forward model as a weekly **type × region price-expectation table** (±$0.11 band).

---

*Built to the communication standard in [`DOCS/STRUCTURE.md`](DOCS/STRUCTURE.md): answer-first,
MECE, effect sizes with every p-value, temporal splits for time-ordered data, and no causal claim
without an identification strategy.*
