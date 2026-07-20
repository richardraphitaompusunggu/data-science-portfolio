# Superstore — Retail Profitability Diagnostic

**Which products, regions, and segments should the Superstore grow, fix, or exit — and how much
profit is recoverable from each decision?**

A Tier A data science project on 9,994 US retail order lines (2014–2017), built to the McKinsey
communication standard: Pyramid Principle, MECE structuring, SCR storyline, answer-first delivery.

📄 **[Read the full report →](reports/final_report.html)** *(download and open locally — GitHub
does not render HTML in-repo)*

---

## The answer

> **The 1,393 order lines discounted beyond 20% — 14% of volume — destroy $135,376 of profit,
> equal to 47% of the company's entire net profit. Every discount level above 20% loses money;
> below it, almost none do.**

| | |
|---|---|
| **Revenue, 2014–2017** | $2.30M |
| **Net margin** | 12.5% — flat for four years despite 51% revenue growth |
| **Order lines sold at a loss** | 18.7% (1,871 of 9,994) |
| **Profit destroyed by discounts >20%** | $135,376 — 47% of net profit |

### Four findings

1. **Losses have a name and a size.** Three sub-categories lose money outright — Tables
   (−$17,725), Bookcases (−$3,473), Supplies (−$1,189). 301 of 1,862 products destroy $76.7K
   between them, while 18% of the catalogue earns 80% of gross profit.
2. **Discount depth is the cause, and it behaves like a switch, not a dial.** At 20% discount, 14%
   of lines lose money and the segment earns an 11.8% margin. At 30%, **92% lose money**. By 80%,
   margin is −180%. Cramér's V of 0.82 against loss dominates every other factor tested.
3. **Regional variation is a symptom of the same cause.** Central posts the worst margin (7.9%) on
   the deepest average discount (24.0%); West the best (14.9%) on the shallowest (10.9%). A policy
   variance, not a market difference — and far cheaper to fix.
4. **The leak is predictable before the sale ships.** A classifier using only order-entry
   information reaches PR-AUC 0.96 against a 0.19 base rate, flagging loss lines at 88% precision.

### Recommendation

Cap discount authority at **20% company-wide**, with a 90-day controlled pilot in Central first.
Sized at **$135,376 recovered** on the walk-away scenario — which assumes nothing whatsoever about
customer behaviour, because the affected lines lose money by construction.

---

## What this project does *not* claim

The recommendation is an intervention, so it owes an identification strategy. There is no
randomisation, no instrument, and no natural experiment in this data — so the causal claim is
bounded deliberately:

| Claim | Supported? |
|---|---|
| Lines discounted past 20% are far less profitable | ✅ Yes — large effect, tight bootstrap CI |
| Discount depth is the strongest observed correlate of loss | ✅ Yes — Cramér's V 0.82 |
| Survives adjustment for mix, geography, time, order size | ✅ Yes — stable across specifications, LOO, placebo |
| Capping discounts *would* recover $135K | ⚠️ Bounded — holds on the walk-away scenario only |
| Discounting *causes* margin loss (a true ATE) | ❌ **No** — open back-door via COGS, which this data lacks |

Hypotheses **tested and rejected** are recorded in the report appendix rather than quietly dropped:
premium shipping does *not* erode margin (all four modes within 1.8 points); the discount cliff does
*not* differ by category; Central is *not* a structurally weak market; the leak is *not* cyclical.

---

## Repository contents

```
notebooks/
├── 01_ingestion.ipynb    Provenance (SHA-256), schema validation, grain declaration, PII review
├── 02_cleaning.ipynb     Type discipline, business rules, derived columns, leakage assertions
├── 03_eda.ipynb          Seven hypothesis-driven exhibits, action titles, So-What annotations
├── 04_analysis.ipynb     Statistical testing · bounded causal read · classifier · policy sizing
└── 05_reporting.ipynb    Assembles the self-contained HTML deliverable
reports/
└── final_report.html     The deliverable — light/dark themes, all figures embedded
```

**Notebooks are committed with their outputs** so they read on GitHub without being re-run — see
*Reproducing this* below for why.

---

## Method notes

**Non-parametric by necessity.** Shapiro–Wilk and Levene both reject normality and equal variance
decisively, so every group comparison uses rank-based tests with rank-based effect sizes. The
17-way sub-category family carries a Benjamini–Hochberg FDR correction.

**Effect size beside every p-value.** On n = 9,994 a small p-value alone means very little. The
Kruskal–Wallis test across sub-categories is the case in point: overwhelmingly significant, but a
*small* effect. The dollar figures do the arguing, not the asterisks.

**Temporal split, never random.** The classifier trains on 2014–2016 and validates on 2017. A
random split would leak 2017 discount regimes backward into training.

**Target leakage blocked by assertion.** `is_loss` is a deterministic function of `Profit`, so
`Profit` and every Profit-derived column are excluded from the feature matrix by a runtime
assertion, not by convention. Customer-history features are computed as-of, using only orders
strictly prior to the current one — also assertion-checked.

**Outliers retained.** The −$6,600 order line is the finding, not contamination. No rows were
removed at any stage; every exclusion decision was a deliberate "no".

**Palette validated, not eyeballed.** The chart palette was re-checked with a CVD/contrast
validator rather than trusted. Two departures from the project design spec were forced by it: the
teal was nudged from `#00857C` to `#009182` to clear the chroma floor (below it, the colour reads
grey), and slate was dropped from the categorical order after failing normal-vision separation
against cyan (ΔE 12.9, against a floor of 15). The primary encoding throughout is a two-colour
polarity pair — blue for profitable, amber for loss-making — which passes every check outright and
states the finding directly.

---

## Limitations

1. **No cost field.** Profit is supplied directly, so we can locate *where* profit is destroyed but
   not decompose *which* cost component destroys it.
2. **No returns flag.** Some observed losses may be returns rather than bad pricing — a genuine
   alternative explanation that cannot be ruled out here.
3. **No randomisation.** Sensitivity analysis shows an unobserved confounder would need to explain
   more residual variation than the entire sub-category fixed-effect block to overturn the result —
   a demanding bar, but not proof.
4. **Pre-scrubbed teaching dataset.** Zero nulls and zero duplicates are not what real operational
   data looks like. The cleaning stage is consequently thinner than a real engagement would demand,
   and this pipeline has not been stress-tested against messy input.
5. **Demand response unmeasured.** The sizing floor sidesteps this by assuming no demand at all on
   affected lines; the true optimum needs the pilot.

---

## Reproducing this

The source CSV is **not committed** — only notebooks, the report, and this README ship to the
repository. To re-run:

1. Download `Superstore.csv` (the Tableau/Kaggle US Superstore sample, 9,994 rows × 21 columns).
2. Place it in the project root, beside `notebooks/`.
3. Install the dependencies:
   ```
   pip install pandas==3.0.3 numpy==2.5.1 matplotlib==3.11.0 seaborn==0.13.2 \
               scipy==1.18.0 scikit-learn==1.9.0 statsmodels==0.14.6 pyarrow==25.0.0
   ```
4. Run the notebooks in order, `01` → `05`.

Notebook 01 recreates `data/raw/`; notebook 02 writes `data/processed/`. Both directories are
generated, not tracked.

> **Note.** The notebooks import two local helper modules — `src/mck_style.py` (chart tokens) and
> `src/build_report.py` (report CSS and HTML components) — which are not published here, since only
> notebooks, the report, and this README ship to the repository. Notebooks 01, 02, and 04's
> statistical sections run standalone; the charting cells in 03 and the report build in 05 need
> those modules. The committed outputs show exactly what they produce.

**Environment:** Python 3.14, pandas 3.0.3, scikit-learn 1.9.0, statsmodels 0.14.6. Seed fixed at
42 throughout; all paths relative; restart-and-run-all verified from a clean `data/` directory.

`lightgbm`, `shap`, and `ydata-profiling` are optional — the notebooks fall back to
`HistGradientBoostingClassifier`, permutation importance, and a hand-rolled profile respectively
rather than failing.

---

*Data: Superstore retail transactions, 9,994 order lines, January 2014 – December 2017. Single
country (United States), 4 regions, 49 states, 793 customers, 1,862 products.*
