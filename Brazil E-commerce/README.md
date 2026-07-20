# Olist Brazilian E-Commerce — Why Do Customers Leave 1-Star Reviews?

A Tier A diagnostic + predictive analysis of ~99,000 Olist marketplace orders (Sep 2016 – Oct 2018),
built to the McKinsey communication standard: Pyramid Principle, MECE structuring, SCR storylining,
answer-first delivery. Blends tabular data with a Portuguese review-text corpus.

**📄 [Read the final report →](reports/final_report.html)** — self-contained HTML, light and dark
themes, no external dependencies.

---

## The question

> *What drives customer dissatisfaction on the Olist marketplace — how much is delivery execution
> versus product, price, or seller mix — and which slice of orders should operations fix first?*

Framed as a brief to the **VP of Marketplace Operations**, who must allocate a fixed quality budget
across four candidate levers over the next four quarters.

## The answer

> **Customer dissatisfaction has two separable causes with two different owners: a severe but narrow
> delivery-execution failure — late orders are 6.7× more likely to draw a 1–2 star review, and
> lateness is concentrated in the worst 20% of sellers — and a larger, quieter product-experience
> failure that arrives perfectly on time and accounts for 68% of all detractors.**

| # | Key Line | Evidence |
|---|---|---|
| 1 | **Delivery is the sharpest lever, but a narrow one** | Late orders draw 1–2 stars at **62.4%** vs **9.3%** on-time — a **6.7×** risk ratio (95% CI 6.55–6.93); the effect scales monotonically with days late, saturating near 80%. Yet lateness reaches only **6.8%** of orders. |
| 2 | **The lateness that exists is concentrated** | The worst **20%** of sellers carry **69%** of all late deliveries (Gini 0.66); ~50 distance-adjusted worst sellers account for **11%** of all late orders. The fix is targeting, not a blanket carrier renegotiation. |
| 3 | **Most dissatisfaction arrives on time — and it is the product** | **68%** of detractors (8,289 orders) got their parcel on time; among them the complaint flips from delivery to the **product itself** (69% name product vs 60% delivery) — invisible to every operational metric. |

**Recommendation:** split the budget by owner. Operations runs a seller-quality programme against the
~50 worst sellers and recalibrates the (12-day-padded, still-missing) delivery promise; Product opens
a listing-accuracy/returns workstream on the on-time detractor population. Do **not** spend on
payment/checkout UX — the effect is statistically flat (a well-powered null).

## Why the review text matters

The tabular data could establish *what correlates* with dissatisfaction but not explain the 68% of
detractors who were delivered on time. The 40,410 Portuguese review comments are the instrument that
adjudicates: they confirm delivery as the top complaint theme (independent of the model) **and**
resolve the on-time mystery — those customers are complaining about the product. Two independent data
sources — timestamps and free text — agreeing is the strongest claim this observational dataset
allows.

## Exhibits

**Part I — Diagnostic:** rating cliff · late→detractor contrast · days-late dose-response · seller
Pareto · delivery-estimate calibration · on-time detractors by category.
**Part II — Text adjudication:** complaint-theme share · two-problems split by lateness · complaint
mix by region · comment selection bias.
**Part III — Predictive:** model comparison (pre- vs post-delivery) · feature importance.

## How it was built

| Stage | Notebook | What it does |
|---|---|---|
| 1 | [01_ingestion](notebooks/01_ingestion.ipynb) | Load 9 tables, **verify latin-1 encoding** (UTF-8 silently corrupts accents), lineage + schema validation |
| 2 | [02_cleaning](notebooks/02_cleaning.ipynb) | Dedup reviews, join 9 tables → one order-level base table (row count asserted at every step), geolocation → zip centroids, PII-scrubbed text corpus |
| 3 | [03_eda](notebooks/03_eda.ipynb) | 12 hypothesis-driven exhibits, one per issue-tree branch |
| 4 | [04_analysis](notebooks/04_analysis.ipynb) | Effect-size-first hypothesis tests (FDR-corrected), leakage-safe models on a temporal split |
| 5 | [05_text_analysis](notebooks/05_text_analysis.ipynb) | NMF + auditable aspect lexicon, construct-validated, adjudicates the issue tree |
| 6 | [06_reporting](notebooks/06_reporting.ipynb) | Assembles the self-contained HTML report — every number pulled from JSON, none hardcoded |

**Key methodological choices**

- **Effect size &gt; p-value.** At n≈96k every difference is "significant"; each test reports a risk
  ratio / Cliff's δ / ε² with a CI, and negligible results (payment friction) are labelled *not
  material*.
- **Leakage discipline.** Text features barred from the models (written simultaneously with the
  score); seller history computed expanding-window with a shrinkage prior; temporal train/test split.
- **Honest negative results.** The order-level late-delivery model is reported as too weak to deploy
  (PR-AUC 0.086); the payment-friction hypothesis is a documented null. Both are informative.
- **LDA rejected** for the ~6-token comments; the headline blame number comes from a shipped,
  auditable lexicon, not a black-box topic model.

## Reproduce

```bash
pip install -r requirements.txt
# place the 9 Olist CSVs in archive/ (see data source below), then run in order:
jupyter nbconvert --to notebook --execute notebooks/0*.ipynb
```

Seed `42` throughout; each notebook passes restart-and-run-all and ends with its STRUCTURE.md gate
checklist ticked against real evidence.

## Data source

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
(Kaggle). Real, anonymised commercial data — company names replaced with Game of Thrones houses. Raw
CSVs are not committed to this repo.

## Limitations

Observational (no causal claim); ~25 months of history (weak seasonality); no cost data (impact in
orders, not ROI); review non-response; a fluent-annotator validation of the text lexicon was not
performed and is the primary next step.

> **Viewing the report:** GitHub shows `.html` as source. Open `reports/final_report.html` locally,
> or via [htmlpreview.github.io](https://htmlpreview.github.io/).
