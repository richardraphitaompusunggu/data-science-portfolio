# Video game sales — size the slate for the power law, localise by genre, screen with the score

**Tier A study**: descriptive (rung 1) → diagnostic (rung 2) → predictive (rung 4), built to the
McKinsey communication standard: answer first, MECE key lines, action titles, and a "So What" under
every exhibit.

📊 **[Read the final report](reports/final_report.html)**: one self-contained HTML file with all charts
embedded.
*(GitHub shows `.html` as source. Open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
or download it and open it locally.)*

---

## The question

> **Where in the physical console market does sales value concentrate (across titles, publishers,
> platforms, genres and regions), and how much of a release's outcome can a greenlight committee
> foresee from what it knows at launch?**

Written for the **Head of Portfolio Strategy at a mid-size third-party publisher**, which has no
platform of its own. Each year they decide how many titles to fund, which genre and platform to put
them on, which regions to localise for, and how much weight a pre-launch hit score should carry.

## The answer

Physical game sales follow a power law that has been the same for 35 years. What a release sells
depends far more on **who publishes it, and on which platform**, than on its genre. A third-party slate
should be sized for a ~12% hit rate, localised by genre, and screened with a launch-time score. That
score finds hits **4.0×** more often than picking at random and should be used to screen projects
out, not to choose them.

| Key line | Evidence |
|---|---|
| **The record.** Most of the post-2010 "collapse" is two platforms retiring | DS + Wii are **85%** of the releases lost between 2008–10 and 2012–14. Hit units fell 37% against 56% for the long tail, so the hit rate per release **rose** from 9.9% to 14.4% [+2.5, +6.2 pp]. 2015–16 sales are incomplete |
| **Where value concentrates.** A constant power law, with platform holders at the top | The top 10% of releases earn **59%** of units. Above a common 100k floor that share sits near 50% **in every era**: the apparent rise after 2005 was the database listing more small titles. Platform holders' own games have **4.1×** the odds of a hit [3.6, 4.6] on the same platform and in the same genre (2.3× without Nintendo) |
| **Where taste differs.** Japan's taste is genre-specific and durable | Japan buys **38%** of role-playing units and **4%** of shooter units. Its ranking of genres correlates at ρ = 0.70–0.94 across every pair of eras (chance ≈ 0.50). Europe's share of units tripled to 34%, and its genre ranking is *not* stable |
| **What a committee can foresee.** A useful screen, not a way to pick hits | Publisher explains **5.8×** the variance genre does (held-out R² 17.8% vs 3.1%). On 2012–14 releases it never saw, a launch-time score's top tenth hits **57%** [47, 66] (third-party: 46% against an 11.7% base rate), and its bottom half hits **2.6%** |

## Two hypotheses that did not survive, and why that matters

The Stage 0 plan expected two findings the evidence then rejected. The report states both corrections
rather than dropping them:

1. **"The market became more hit-driven after 2005."** Across all releases the top tenth's share does
   rise, by 5.0 pp [+1.9, +8.2]. Over the same years, the share of each era's releases selling under
   100k grew from 13% to 38%: vgchartz was listing more small titles. Above a common 100k floor the
   rise is 2.6 pp [−1.1, +6.4] and not significant. The power law is a **constant**, which is a more
   useful planning fact than a trend.
2. **"Publisher track record carries the hit score."** The ablation says otherwise. Every feature family
   alone reaches at most 0.31 PR-AUC, against 0.51 together. Platform & genre is the weakest family on
   its own (0.21) but the costliest to remove (−0.12), because it gives the track record its context.

## Corrections to the data review

`DOCS/vgsales_data_review.md` was re-derived from the raw file. Its headline figures reproduce, except
for these:

| Review says | File says |
|---|---|
| Global ≠ sum of regions in only 10 rows | **4,511 rows (27%)** differ by ±0.01 and 10 by ±0.02. It is rounding, so checks need a 0.02 tolerance |
| 0 full-row duplicates | 1 exact duplicate once `Rank` (a sort index) is ignored, plus a scraper stray and a record **split in two** that has to be merged, not dropped |
| "> 100,000 copies" threshold | **5,781 rows (35%)** are below it. The floor is wherever vgchartz stopped listing titles |
| Year issues: DS 1985, one 2020 row | 4 rows predate their platform and 4 postdate the snapshot. Lifetime sales for 2015–16 are **right-censored**, a problem the review did not flag |

## Notebooks

| # | Notebook | What it does |
|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | **Stage 0 framing** (question, issue tree, situation–complication–resolution draft, report outline), schema check, lineage + SHA-256, re-derivation of the data review |
| 02 | [Cleaning](notebooks/02_cleaning.ipynb) | Four repeated keys handled four ways, impossible years, conservative year and publisher recovery from the same title on other platforms (flagged), derived columns, title table, rule checks |
| 03 | [EDA](notebooks/03_eda.ipynb) | One section per issue-tree branch: the record, titles, owners, regions, products. Six exhibits with action titles |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | Floor-robust concentration (bootstrap), first-party premium within platform and genre (Mantel–Haenszel, fixed-effects logit), variance comparison on held-out titles, permutation-tested taste stability, Benjamini–Hochberg across 20 tests |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | Time-split, title-disjoint hit classifier using only launch-time features. Publisher history uses earlier years only, and that is asserted in code. Three baselines, ablation, third-party and per-platform checks, calibration, slate-size arithmetic, model card |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | Builds `reports/final_report.html`. Every number is read from `_key_figures.json`; nothing is hardcoded |

## Reproducing

```bash
pip install -r requirements.txt          # pandas 3.0.3, scikit-learn 1.9.0, statsmodels 0.14.6, ... (pinned)
# place vgsales.csv in archive/, then run notebooks 01 → 06 in order
```

`RANDOM_STATE = 42` throughout. Restart-and-run-all is verified from the raw CSV. The notebooks
import **no local modules**: chart styling and every helper are defined inline, so a fresh clone runs
on the pinned packages alone.

**Palette note:** DESIGN.md's teal `#00857C` fails the dataviz validator's chroma floor (C = 0.097 < 0.10;
it reads as grey). It is nudged to `#008A7E`, the nearest passing step on the same hue. The
validated set `#2251FF, #008A7E, #C1841C` passes every check (worst colour-blind ΔE 12.5).

## What this analysis does not claim

Units, not money. Physical retail only, with no digital or mobile. vgchartz **estimates**, including
console bundles. The data **ends in 2016**: the structural patterns (the power law, the first-party
premium, the shape of regional taste) are hypotheses to re-test on current data, and the levels are
ten years old. **Nothing here is causal.**

**Data:** [Video Game Sales](https://www.kaggle.com/datasets/gregorut/videogamesales) (Kaggle,
scraped from vgchartz.com). The raw CSV is not committed.

**Governed by:** `DOCS/STRUCTURE.md` (pipeline and communication standard), `DOCS/DESIGN.md`
(visual system) and `DOCS/vgsales_data_review.md` (what the data is and isn't).
