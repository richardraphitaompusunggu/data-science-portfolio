# Pokémon stats by type: budget power by tier, template only the types with a signature, skip the type checker

**Tier A study**: descriptive (rung 1) → diagnostic (rung 2) → a predictive check (Path B) for one design decision.
Built to the McKinsey communication standard: answer first, MECE key lines, action titles, and a "So What" under every
exhibit.

📊 **[Read the final report](reports/final_report.html)**: one self-contained HTML file with all charts embedded.
*(GitHub shows `.html` as source. Open it via [htmlpreview.github.io](https://htmlpreview.github.io/) or download it and
open it locally.)*

---

## The question

> **When designing a new roster of creatures, how much should a creature's type constrain its stats (its overall power
> and its stat profile), and can a model trained on Generations 1–6 check new designs for "off-type" stats?**

Written for the **lead creature-balance designer** on a new monster-collector title who uses the main-series roster as
the reference corpus. Three decisions hang on it: whether to author stats from type templates, what power-budget rules
to adopt, and whether the tools team should build a stats → type classifier as a design check.

## The answer

Type is a tilt, not a template. A creature's power is set by a few fixed tier rules, not by its type, and type shapes the
stat profile of only **10 of 18** types. Validated honestly, stats name the type **21%** of the time, against **14.6%**
for always guessing Water. The design team should budget power by tier, template stat shape only for the signature
types, and not build a stats-to-type checker.

| Key line | Evidence |
|---|---|
| **The file.** 721 species, not 800 independent rows | 48 Megas and 31 alternate forms reuse a species' number and type. The `Legendary` flag misses 8 species the games class as legendary or mythical. A species' nearest stat-shape neighbour is within two dex numbers **26%** of the time (chance 0.6%): evolution families break independence |
| **Power.** Set by tier rules | **53 of 54** legendary species sit at Total ≥ 580; the 7 pseudo-legendaries sit at exactly 600; **47 of 48** Megas add exactly 100. Legendary status explains 26% of Total, type 7%, and type's link does not survive correction once legendary status is held fixed. No generational climb (ρ = 0.06) |
| **Shape.** Type tilts ten types | Type explains at most **28%** of any stat's share of Total (Sp. Atk). Tested on 360 family means with BH correction, 10 of 17 testable types keep a signature (Steel and Rock +8 Defense per 100, Fighting +5 Attack, Electric +5 Speed, Normal +5 HP …). Water, Grass, Bug, Ice, Poison, Dragon and Ghost have none |
| **Prediction.** A checker would flag noise | Random forest on stat shares: **21.2% ± 1.8%** family-grouped, **20.5%** on an unseen generation, against **34%** under a random split. No pair of stats beats 19.3% (Attack × Defense ranks 4th of 15). Only 7 types reach a one-vs-rest AUC of 0.75 |

## Hypotheses that did not survive

The Stage 0 plan carried expectations from the data review. The report states each correction rather than dropping it:

1. **"Steel, Rock, Ghost and Dragon have the clearest signatures."** Half right. Steel and Rock have the two strongest;
   Ghost and Dragon have none that survives a family-level test. Ghost is even *anti*-recognisable (AUC 0.38): its
   families are unlike each other.
2. **"A stratified split is enough."** It is not. Evolution siblings straddle it and inflate accuracy by 13 points.
3. **"Natural archetypes exist (tank, sweeper, glass cannon)."** Not as separate clusters: silhouette ≈ 0.2 at every k,
   a few hundredths above structureless data. The one sharp group is 21 HP specialists (Chansey, Blissey, Wobbuffet) at 35% HP.
4. **"Dual types carry a stat premium."** The median is higher (452 vs 405), but within families the dual-typed member
   is the later evolution in 83% of cases. It is a stage effect, not a design rule.

## Corrections to the data review

`DOCS/pokemon_data_assessment.md` was written from the dataset description alone and marks its unchecked claims
**[verify]**. Each was checked against the raw file:

| Review says | File says |
|---|---|
| ~721 rows × 11 columns | 800 × 13 |
| `Generation` and `Legendary` probably absent | Both present; `Generation` matches the dex ranges exactly |
| — | `Legendary` mixes two definitions: 9 mythicals flagged, Mew, Celebi, Cresselia, Phione, Manaphy, Keldeo, Meloetta and Genesect not. A corrected `legendary_canon` is added beside it |
| `#` may repeat for forms | It does: 79 extra rows, form label glued onto the name (`VenusaurMega Venusaur`) |
| Stats positively correlated → multicollinearity | True of raw stats: a size effect. Stat shares trade off (Attack × Sp. Def ρ = −0.56, Defense × Speed −0.51) |
| Use a stratified split | Split by evolution family or by generation |
| Majority-class accuracy ~13–14% | 14.0% of rows, 14.6% of species: confirmed |

## Notebooks

| # | Notebook | What it does |
|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | **Stage 0 framing** (question, issue tree, situation–complication–resolution draft, report outline), schema contract, lineage + SHA-256, re-derivation of the data review, form audit, legendary-flag audit, evolution-family non-independence test |
| 02 | [Cleaning](notebooks/02_cleaning.ipynb) | Split glued form names, analysis grain (721 species), `legendary_canon`, evolution-family proxy, review §7 derived columns (stat shares, bulk, ratios), outlier flags, cleaning log |
| 03 | [EDA](notebooks/03_eda.ipynb) | One section per issue-tree branch: the file, power, shape. Six exhibits with action titles |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | Power tiers, variance decomposition of Total, power creep (species and family level), type signatures tested on family means, dual-type premium; Benjamini–Hochberg across 119 tests |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | Baselines, four validation protocols × three models × five seeds, permutation null, 15 stat pairs, per-type AUC, archetype clustering against a no-structure null, model card |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | Builds `reports/final_report.html`. Every number is read from `_key_figures.json`; nothing is hardcoded |

## Reproducing

```bash
pip install -r requirements.txt          # pandas 3.0.3, scikit-learn 1.9.0, statsmodels 0.14.6, ... (pinned)
# place Pokemon.csv in archive/, then run notebooks 01 → 06 in order (05 takes ~10 minutes)
```

Seeds are fixed throughout (`RANDOM_STATE = 42`; five seeds for model comparisons). Restart-and-run-all is verified from
the raw CSV. The notebooks import **no local modules**: chart styling and every helper are defined inline, so a fresh
clone runs on the pinned packages alone.

**Palette note:** the three identity colours `#2251FF, #008A7E, #C1841C` pass the dataviz validator on all pairs (worst
colour-blind ΔE 12.5, all ≥ 3:1 contrast on white). DESIGN.md's teal `#00857C` fails the chroma floor and is nudged to
`#008A7E`. Grey carries context; the type-signature heatmap uses an amber ← grey → blue diverging scale.

## What this analysis does not claim

**Nothing about strength in play.** Base stats feed the damage formula; moves, abilities, levels and items decide
battles, and none is in the file. **No causation:** the signatures describe how Generations 1–6 were designed, offered as
defaults, not laws. **Gen 1–6 values as recorded:** later games changed some base stats. The family proxy (consecutive
species sharing a primary type) misses evolutions added in later generations, so the honest model score is, if
anything, slightly optimistic.

**Data:** [Pokemon with stats](https://www.kaggle.com/datasets/abcsds/pokemon) (Kaggle). The raw CSV is not committed.
Pokémon names and stats belong to Nintendo / Game Freak / The Pokémon Company.

**Governed by:** `DOCS/STRUCTURE.md` (pipeline and communication standard), `DOCS/DESIGN.md` (visual system) and
`DOCS/pokemon_data_assessment.md` (the data review re-derived in `01`).
