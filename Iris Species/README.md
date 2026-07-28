# Fisher's Iris — what a 150-row benchmark can and cannot tell you about choosing a model

**Tier A study** — descriptive (rung 1) → diagnostic (rung 2) → predictive (rung 4), built to the
McKinsey communication standard: answer-first, MECE, action titles, a "So What" under every exhibit.

📊 **[Read the final report](reports/final_report.html)** — self-contained HTML, all charts embedded.
*(GitHub shows `.html` as source; open it via [htmlpreview.github.io](https://htmlpreview.github.io/)
or download and open locally.)*

---

## The question

> Using only sepal and petal measurements, what is the simplest rule that separates the three Iris
> species — and, given 150 specimens, is the accuracy gap between competing models large enough to
> be measurable at all?

Iris is solved. Any classifier scores above 93%, and that is exactly why the interesting question is
not *"which model wins"* but **whether the leaderboard everyone publishes on this file means
anything.**

## The finding

**Changing only the random seed moves accuracy 13.3 points. Changing the algorithm moves it 4.0.**

Same data, same model, same code — only the `random_state` of an 80/20 split changes. Across 100
splits the median model swings **13.3 points** (k-NN swings 16.7, from 83.3% to a perfect 100%).
The entire spread between **eight** model families under 100-fold repeated CV is **4.0 points**.

> The seed lottery is **3.3× wider than the thing every leaderboard claims to measure** — and every
> one of the six models tested reaches 100% on *some* seed. Any of them can be crowned best by a
> sufficiently lucky split, with no dishonesty required.

A held-out set makes it concrete: the winning model scored **100.0%** on 30 held-out specimens, with
a 95% Wilson interval of **[88.6%, 100%]** — an interval nearly **three times wider than the entire
model-family spread**. "LDA achieves 100% on held-out data" is true, publishable, and fully
compatible with its real accuracy being 89%.

### The honest version of "no winner"

The obvious conclusion — *no model is distinguishable* — is **too strong, and the study says so.**
Under a 100-fold paired protocol, **5 of 7 rivals cannot be told apart from the best**; the two
weakest (gradient boosting, decision tree) genuinely are separable. The detection floor at n=150 is
**3.2 points** and the observed spread is **4.0** — it clears, but only just, and only under a
protocol nobody uses.

This is also where the study's own Stage 0 hypothesis was wrong, corrected in the report rather than
smoothed over.

### A methodological trap, caught in the act

Comparing models by bootstrapping over CV folds produces intervals **5× too narrow**, because 100
repeated-CV folds are not 100 independent measurements — they re-shuffle the same 150 specimens.

| Comparison | Naive fold bootstrap | Corrected-resampled (Nadeau–Bengio) |
|---|---|---|
| LDA − linear SVM | +1.9 pts, CI **[1.4, 2.5]** — decisive | same +1.9 pts, CI **spans zero**, p = 0.20 |

Both intervals are plotted on the same exhibit. The naive one never touches zero and is confidently
wrong.

## What the data actually supports

| Finding | Evidence |
|---|---|
| **One measurement does nearly all the work** | `PetalWidthCm` alone scores **96.1%** vs **97.9%** for all four features and **79.7%** for sepals alone — a 16.1-point gap between the two feature branches |
| **A rule needing no software** | Petal width < 0.6 cm → setosa; < 1.6 cm → versicolor; else virginica. **94.1% honest out-of-fold** (96.0% in-sample — the 1.9-point gap is the priced cost of fitting cuts on the data you evaluate on). Costs 3.8 points against the best model |
| **The problem is one-dimensional** | Fisher's first discriminant carries **99.1%** of between-species variance; the two petal columns correlate at ρ = 0.94 |
| **Separability, proved rather than asserted** | A hard-margin linear SVM separates setosa with **zero** training errors on petals alone; versicolor/virginica strands 2 specimens even with all four features *and* full knowledge of the labels |
| **Engineered features don't help** | All seven derived features from the data notes move LDA **+0.37 pts** (p = 0.59) and logistic **+2.03** (p = 0.11) — neither distinguishable from zero. `PetalLengthWidthRatio` alone *collapses* to 71.2%: the division cancels the size axis that does the work |
| **"96% accuracy" means 19 flowers** | **131 of 150** specimens are never misclassified by any of six model families across 20 shuffles. Nineteen are ever wrong; **three (Ids 71, 84, 134) are wrong for every model.** They are not outliers and not the defective rows — just flowers on the wrong side of the border |

## Three defects in the file everyone trusts

All 16 schema-validation rules pass. Zero nulls. That is a statement about **completeness**, and it
is silent about **correctness**.

1. **`Id` predicts species at 98.7%.** The file is sorted by species (Ids 1–50 / 51–100 / 101–150),
   so a decision tree on the index alone beats every real measurement. The project's own data notes
   call `Id` *"zero information value"* — the exact belief that makes `df.drop(columns=['Species'])`
   feel safe.
2. **Two documented UCI transcription errors are present.** Rows 35 and 38 carry the erroneous
   values, and the damage **fabricates a triplicate** with row 10. Held by three converging lines of
   evidence: the UCI errata note, the collision dissolving exactly under correction, and the
   chance-excess vanishing with it (p = 0.018 as delivered → 0.43 corrected).
3. **A genuine duplicate pair** (Ids 102/143) straddles the train/test boundary in **66%** of random
   stratified splits — verified both combinatorially and over 20,000 simulated splits.

**And the honest closure:** deduplicating or applying the corrections shifts accuracy by at most
**0.14 points** — 23× below the detection floor. Two of the three defects would have cost a real
project nothing. The `Id` leak is the one that bites, because it doesn't degrade a result, it
**manufactures** one.

## Notebooks

| # | Notebook | Stage | What it establishes |
|---|---|---|---|
| 01 | [Ingestion](notebooks/01_ingestion.ipynb) | 1 | 16 schema rules across 4 families (incl. a measurement-granularity check); the `Id` leakage forensic; two claims in the data notes tested and refuted |
| 02 | [Cleaning & forensics](notebooks/02_cleaning.ipynb) | 2 | Duplicates **diagnosed into two distinct causes**; UCI errata tested against the file; split-straddle probability; measurement-scale contract persisted |
| 03 | [EDA](notebooks/03_eda.ipynb) | 3 | Overlap coefficients per feature per species pair; the 1.1 cm empty corridor; petal ρ = 0.94 as the mechanism behind parsimony |
| 04 | [Diagnostic](notebooks/04_diagnostic.ipynb) | 5a | Assumptions tested *before* the tests they license; bootstrap CIs on Cohen's d; linear separability proved by hard-margin SVM; Fisher's LDA run as Fisher ran it |
| 05 | [Modelling](notebooks/05_modeling.ipynb) | 4 · 5b · 6 | Feature-block and model bake-offs on identical folds, the seed lottery, the MDD simulation, the named error specimens, ablation and three sensitivity runs |
| 06 | [Reporting](notebooks/06_reporting.ipynb) | 7 | Builds `final_report.html` — hardcodes no numbers, reads them all from upstream JSON/CSV |

## Method notes

- **One fixed set of 100 folds** (5-fold × 20 repeats), shared by every model, so all comparisons are
  paired rather than two independently wobbling means.
- **Corrected-resampled t-test** (Nadeau–Bengio) for every model comparison, with the naive fold
  bootstrap reported alongside *only* to show it is 5× too narrow.
- **Nested CV** — k, tree depth and SVM C/γ tuned in an inner 3-fold loop, so tuning never leaks into
  a reported score. Scalers and threshold cuts fitted on training folds only.
- **A minimum-detectable-difference table** (`_mdd_table.csv`) simulated from per-species Gaussians
  fitted to Iris: 7.5 pts at n=50, 3.2 at n=150, 1.1 at n=1000. This is the reusable artefact.
- **Negative results shipped** — the seven derived features, and the two the data notes singled out
  as most promising, are reported as measured nulls rather than quietly dropped.
- **Path C (causal) explicitly out of scope** — no genetic, environmental or temporal variables, and
  petal width is not an intervention anyone can assign.

## Limitations

All 150 specimens come from **one collector, one region (the Gaspé Peninsula), one season, 1935** —
with two of the three species taken from the same pasture on the same day. That is a **sampling
frame, not a population**: nothing here licenses a claim about *Iris* as a genus, and the ruler rule
would need refitting on any other population. A fairness audit is recorded as **not applicable with
its reason** (the model scores flowers, not people) rather than silently skipped; the
representation-bias check that *does* apply was executed and binds. Nothing is deployable.

The MDD table is a simulation calibrated to Iris's covariance structure, not a universal constant — a
harder problem at the same *n* will have a higher floor.

## Reproducing

```bash
pip install -r requirements.txt          # pandas 3.0.3, scikit-learn 1.9.0, scipy 1.18.0, …
# place Iris.csv in the project root
# run notebooks 01 → 06 in order
```

Seed 42 throughout; no local module imports, so a fresh clone runs on the CSV and the pinned packages
alone. Data files, figures and intermediate artefacts are gitignored — only notebooks, the report and
this README are tracked.

**Source:** [Iris Species](https://www.kaggle.com/datasets/uciml/iris) (150 × 6). Collected by Edgar
Anderson (1935), published by R.A. Fisher (1936), digitised by the UCI ML Repository — **including
its two documented errata**, which this study tests for and finds.
