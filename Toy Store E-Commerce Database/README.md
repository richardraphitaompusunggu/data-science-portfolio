# Maven Fuzzy Factory — Online Toy Store Growth Diagnostic

A Tier A, decision-grade data analysis of **Maven Fuzzy Factory**, a (fictional) online toy/plush
retailer, covering its **first three years of trading (2012-03-19 → 2015-03-19)**. The project traces
the full acquisition-to-purchase journey — website traffic, on-site behaviour, orders, and refunds — to
answer one question:

> **Where in our funnel and product portfolio are the highest-value, actionable opportunities to grow
> revenue, and which levers should we pull first?**

The work is produced to the McKinsey communication standard (answer-first, MECE, action titles) defined
in [`DOCS/STRUCTURE.md`](DOCS/STRUCTURE.md), styled per [`DOCS/DESIGN.md`](DOCS/DESIGN.md), and planned in
[`DOCS/IMPLEMENTATION_PLAN.md`](DOCS/IMPLEMENTATION_PLAN.md).

---

## The data

Six related tables (event-log clickstream + tabular transactions). Full field definitions live in
[`DOCS/maven_fuzzy_factory_data_dictionary.csv`](DOCS/maven_fuzzy_factory_data_dictionary.csv).

| Table | Rows | Grain | What it holds |
|---|---:|---|---|
| `website_sessions` | 472,871 | one visit | traffic + UTM attribution, device, new-vs-repeat |
| `website_pageviews` | 1,188,124 | one page view | timestamped clickstream (`pageview_url`) |
| `orders` | 32,313 | one order | revenue (`price_usd`) and cost (`cogs_usd`) → margin |
| `order_items` | 40,025 | one product in an order | product-level revenue + `is_primary_item` (cross-sell) |
| `order_item_refunds` | 1,731 | one refund | refund amount + timing |
| `products` | 4 | catalog | product names + staggered launch dates |

**How they connect:** `website_sessions` → `website_pageviews` (session id) reconstructs browsing; a
session may convert into an `orders` row; each order fans out into `order_items`, which may each have an
`order_item_refunds` row; `products` is the lookup.

**Known limits (bound our claims):** no marketing *spend* (so no true CAC/ROAS), no geography or customer
demographics, A/B tests are **unlabelled** (inferred from URL × time → quasi-experimental), no refund
reason codes, and timezone-naive timestamps. See the plan's §1 and §8 for the full list.

---

## Approach

- **Path A — Descriptive / Diagnostic** is primary (what happened across the funnel and portfolio, and
  what's associated with conversion and revenue).
- **Path C — Causal (quasi-experimental)** is optional, only for the landing-page split test if the data
  supports a clean read.
- **Not** a machine-learning project — the decision is insight-driven and past-facing.

The analysis is organised around a MECE issue tree: **Acquisition → Conversion → Monetization → Leakage**.

---

## Repository structure

```
.
├── DOCS/
│   ├── STRUCTURE.md               # pipeline + McKinsey reporting standard (what/why)
│   ├── DESIGN.md                  # visual system for report + charts (look)
│   ├── IMPLEMENTATION_PLAN.md     # this project's plan (Stage 0 framing → deliverables)
│   └── maven_fuzzy_factory_data_dictionary.csv
├── data/
│   ├── raw/                       # immutable source CSVs (never edited after ingestion)
│   └── processed/                 # cleaned outputs (Stage 2 — not yet built)
├── src/
│   ├── ingestion.py               # Stage 1 — load, hash, validate schema  ✅ built
│   └── __init__.py
├── notebooks/                     # 01_ingestion … 05_reporting (per stage)
├── reports/figures/               # exported exhibits (PNG/SVG)
├── logs/
│   └── ingestion_log.json         # provenance: counts, SHA-256 hashes, schema checks
├── tests/
├── requirements.txt
└── README.md
```

---

## Getting started

Requires **Python 3.11+** (developed on 3.14) and the packages in `requirements.txt`.

```bash
pip install -r requirements.txt

# Stage 1 — ingest the raw data, validate schema, write the provenance log
python src/ingestion.py
```

Expected output: a summary table showing all six tables **PASS** schema validation, and a written
`logs/ingestion_log.json`.

---

## Progress

| Stage | Status |
|---|---|
| 0 — Problem framing | ✅ Done (`DOCS/IMPLEMENTATION_PLAN.md`) |
| 1 — Data ingestion | ✅ Done (`src/ingestion.py`, `logs/ingestion_log.json`) |
| 2 — Cleaning & wrangling | ✅ Done (`src/cleaning.py`, `src/validation.py`, `data/processed/`) |
| 3 — EDA | ✅ Done (`src/mck_style.py`, `src/visualization.py`, `notebooks/03_eda.ipynb`, 10 exhibits in `reports/figures/`) |
| 5a — Statistical testing | ✅ Done (`src/analysis.py`, `notebooks/04_analysis.ipynb`) — confirmed device gap; **ruled out landing-page finding as confounding** |
| 5d — Causal (optional) | ⬜ Not pursued — landers were device-targeted, not randomized; a *prospective* A/B test is the right path |
| 7 — Reporting | ✅ Done (`src/build_report.py`, `reports/final_report.html`, `reports/executive_summary.md`) |

**Pipeline complete (Stages 0 → 7).** Final deliverable: [`reports/final_report.html`](reports/final_report.html) — a self-contained, theme-aware Tier A report — with the one-page [`reports/executive_summary.md`](reports/executive_summary.md).

## Key findings

**Governing thought:** *The two growth levers worth funding are the mobile conversion gap and cross-sell of higher-margin newer products; the landing-page differences are a confounding artifact, and channel mix and refunds are secondary.*

1. **Mobile is the biggest conversion lever** — desktop converts 8.5% vs mobile 3.1% (2.75×); mobile is 31% of traffic.
2. **Cross-sell grows profit** — multi-item orders went 0% → 34% after 2013; AOV $50 → $64; newer products carry 68%+ margin vs the flagship's 61%.
3. **The landing-page "winner" is a confounding artifact** — the landers were run in different eras and targeted at different device segments; matched like-for-like, none reliably beats `/home`. The naive read would have wrongly retired `/lander-3` and over-invested in `/lander-5`. Optimising landing pages needs a *prospective randomised A/B test*.

---

## What Stage 1 (ingestion) guarantees

- **Immutability** — raw CSVs are copied into `data/raw/` and never edited; every later stage rebuilds
  from them.
- **Provenance** — each load records row/column counts and a **SHA-256 hash** so silent file changes are
  detectable and every run is reproducible.
- **Schema validation** — column set, column order, and primary-key uniqueness/non-nullity are checked
  against the data dictionary. Discrepancies are *reported*, never silently fixed (that is Stage 2's job).
