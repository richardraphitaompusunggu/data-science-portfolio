# Mall customer segmentation: keep the Affluent Spenders, test the young Under-engaged Affluent, size nothing from this file

**Tier A study**: descriptive (rung 1) → diagnostic (rung 2) → unsupervised segmentation, with an
experiment design for the one intervention it recommends (Path C, prospective). Built to the McKinsey
communication standard: answer first, MECE key lines, action titles, and a "So What" under every exhibit.

📊 **[Read the final report](reports/final_report.html)**: one self-contained HTML file with all charts
embedded.
*(GitHub shows `.html` as source. Open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
or download it and open it locally.)*

---

## The question

> **Which natural segments exist among the mall's loyalty members on income and spending, which of
> them should marketing retain and which should it try to convert, and how far can this file be
> trusted to size those targets?**

Written for the **Head of CRM & Marketing** at the mall, who decides how next quarter's loyalty budget
is split across segments and which conversion pilot to run.

## The answer

The 200 members fall into five segments that are real, stable and reproducible with **two income and
two score thresholds**. But the file was generated in pairs, so its segment sizes mean nothing, and
**no member over 40 scores above 60**. Marketing should protect the 39 Affluent Spenders, test conversion
first on the 15 under-40 Under-engaged Affluent with a pilot run on the full member base, and size every
budget from the real counts the rule produces.

| Key line | Evidence |
|---|---|
| **The file.** Generated two members at a time | All 100 consecutive ID pairs share an income (random order: at most 9 of 100). At the income extremes the even-ID twin outspends the odd-ID twin in **61 of 63** pairs, and every corner segment is a single ID parity. Segment sizes come from the generator, not from a mall |
| **The structure.** Five real, stable segments | Silhouette **0.555** against at most 0.44 from 200 no-structure nulls. Gaussian mixture and Ward agree (ARI 0.96 / 0.94); bootstrap Jaccard ≥ 0.97 for every segment. Income 40k / 70k and score 40 / 60 reproduce the clustering for **97.5%** of members |
| **The drivers.** Income sets the spread, age sets a ceiling, gender does nothing | r(income, score) = 0.01 [−0.13, 0.15], yet scores vary **4.2×** [3.1, 5.9] as much at the income extremes as in the middle. **0 of 78** over-40s score above 60 (24 expected by chance). Gender: d = 0.12 [−0.19, 0.41] |
| **The target.** Retain 39, pilot 15, size the pilot on the real base | All Affluent Spenders are aged 27–40. Only 15 of 35 Under-engaged Affluent are 40 or under. A 50% lift on a 5% response needs **~1,460 members per arm**; this file's 15 could only detect 5% → 66% |

## Hypotheses that did not survive

The Stage 0 plan carried three expectations from the data review. The report states each correction
rather than dropping it:

1. **"The file is a clean snapshot of a member base."** Rejected. `CustomerID` is the income rank
   (Spearman 1.00), not "no information content", and the rows were built in income-sorted pairs split
   into opposite corners. The shape of the segments stands; their sizes do not.
2. **"Income does not drive spending."** Half right. It does not move the average score, but it
   controls how widely scores vary (distance correlation 0.25, p = 0.001). That tells marketing where
   targeting matters.
3. **"The Under-engaged Affluent are the biggest conversion upside."** Narrowed to the 15 aged 40 or under.
   They have high-spending peers of the same age and income; the over-40s have none anywhere in the file.

## Corrections to the data review

`DOCS/mall_customers_analysis.md` was re-derived from the raw file. Every number reproduces, including the
k = 5 silhouette of 0.555, the cluster table and the age-band means. These statements do not hold:

| Review says | File says |
|---|---|
| `CustomerID`: "unique, sequential, no information content" | It is the income rank, and its parity alone determines the four corner segments |
| Cluster sizes indicate where the opportunity is ("biggest untapped potential") | The corner sizes are near-equal **by construction** and cannot size any budget |
| Score by income band is "flat", so income does not drive score | Flat in level, not in spread: SD 7.6 in the 41–69k band against 30–33 at either end |
| Age has "a modest negative relationship" (r = −0.33) | It is a hard ceiling at 40, not a slope |

## Notebooks

| # | Notebook | What it does |
|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | **Stage 0 framing** (question, issue tree, situation–complication–resolution draft, report outline), schema contract, lineage + SHA-256, re-derivation of the data review, construction audit |
| 02 | [Cleaning](notebooks/02_cleaning.ipynb) | Column roles (ID barred from features), assertions, outlier flags, forensic pair columns, the review's derived columns, cleaning log |
| 03 | [EDA](notebooks/03_eda.ipynb) | One section per issue-tree branch: the file, income, age, gender. Five exhibits with action titles |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | Permutation test of the pair structure, distance correlation and Brown–Forsythe for income, Fisher exact for the age ceiling, gender effect sizes; Benjamini–Hochberg across 11 tests |
| 05 | [Segmentation](notebooks/05_modeling.ipynb) | k selection against no-structure nulls, k-means k = 5, robustness (seeds, GMM, Ward, bootstrap, outliers), the two-threshold rule, targeting split at age 40, fairness, pilot power analysis, segmentation card |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | Builds `reports/final_report.html`. Every number is read from `_key_figures.json`; nothing is hardcoded |

## Reproducing

```bash
pip install -r requirements.txt          # pandas 3.0.3, scikit-learn 1.9.0, statsmodels 0.14.6, ... (pinned)
# place Mall_Customers.csv in archive/, then run notebooks 01 → 06 in order
```

`RANDOM_STATE = 42` throughout. Restart-and-run-all is verified from the raw CSV. The notebooks
import **no local modules**: chart styling and every helper are defined inline, so a fresh clone runs
on the pinned packages alone.

**Palette note:** the four segment colours `#2251FF, #008A7E, #C1841C, #00A9F4` pass the dataviz
validator on all pairs (worst colour-blind ΔE 12.5). Cyan is below 3:1 contrast on white, so every
segment also has its own marker shape and a direct label. DESIGN.md's teal `#00857C` fails the chroma
floor and is nudged to `#008A7E`. Mainstream is drawn in neutral grey as context.

## What this analysis does not claim

**No response, no causation.** The file has no campaign exposure, response or transactions, so segments
describe how members score now, not how they would react to an offer. The pilot design is how that gets
answered. **A teaching dataset, not a mall:** 200 members, generated in pairs, with an undefined Spending
Score and an income field of undocumented meaning. P-values describe this file, not a population.

**Data:** [Mall Customer Segmentation Data](https://www.kaggle.com/datasets/vjchoudhary7/customer-segmentation-tutorial-in-python)
(Kaggle). The raw CSV is not committed.

**Governed by:** `DOCS/STRUCTURE.md` (pipeline and communication standard), `DOCS/DESIGN.md`
(visual system) and `DOCS/mall_customers_analysis.md` (the data review re-derived in `01`).
