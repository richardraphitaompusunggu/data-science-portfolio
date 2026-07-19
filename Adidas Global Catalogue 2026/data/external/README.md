# External reference tables

Hand-built lookups joined during Stage 2 cleaning. Not part of the raw dataset.

| File | Purpose | Key → value |
|---|---|---|
| `fx_rates.csv` | Convert `price_local` to USD | `currency` → `usd_per_unit` |
| `category_map.csv` | Normalize localized `category` | `category_raw` → `category_canonical` (English US/GB taxonomy, 17 labels) |
| `gender_map.csv` | Normalize localized `gender_segment` | `gender_raw` → `gender_canonical` ∈ {M, W, U, K} |

## ⚠️ FX rates are placeholders

`fx_rates.csv` holds **reference approximations** for early-2026 levels (`source = reference-approx`),
pinned to `as_of = 2026-03-09` (the later snapshot). **Replace them with an authoritative source**
(ECB reference rates or OANDA) for that exact date before publishing any absolute `price_usd` figure.

`price_usd = price_local × usd_per_unit`. Cross-market *relative* comparisons (coefficient of variation,
which market is dearer) are robust to small rate errors; absolute USD claims are not until rates are pinned.

## Coverage

Each table is regenerated with an assertion that it covers **every** raw value present in
`archive/Adidas_Global.csv` — 9/9 currencies, 9/9 `gender_segment` values, 53/53 `category` values.
Null `category` (517 rows, mostly CN) is handled in `src/features.py` as an explicit `'No Category'`
bucket, not in `category_map.csv`.
