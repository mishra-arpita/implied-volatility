# Implied Volatility Surface Prediction
### Finance Club, IIT Roorkee — Open Projects 2026

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-Competition-20BEFF?logo=kaggle&logoColor=white)
![Score](https://img.shields.io/badge/Public%20MSE-0.0000423-brightgreen)
![Rank](https://img.shields.io/badge/Leaderboard-Top%20100-orange)

---

## Overview

This project predicts **missing implied volatility (IV) values** in a Nifty50 options dataset across different strikes and timestamps. It was submitted as part of the Finance Club IIT Roorkee Open Projects 2026 Kaggle competition, evaluated on Mean Squared Error (MSE).

**Author:** Arpita Mishra (23112021)  
**Competition metric:** MSE — lower is better  
**Public leaderboard score:** `0.0000423397`

---

## Problem Statement

Implied volatility is the market's consensus estimate of future uncertainty, embedded in option prices. An **IV surface** describes how IV varies across strike prices and timestamps. In practice, some values are missing due to illiquid strikes, sparse trading, or data quality issues.

Given a CSV of Nifty50 options IV data (975 timestamps × 28 strike columns, ~20% values missing), the task is to fill in all missing IV values as accurately as possible.

---

## Dataset

| Property | Detail |
|---|---|
| File | `dataset.csv` |
| Rows | 975 (5-minute intervals, Jan 7 – Jan 27, 2026) |
| Columns | `datetime`, `underlying_price`, 14 CE (Call) strikes, 14 PE (Put) strikes |
| CE strikes | 25200 – 26500 (step 100) |
| PE strikes | 23800 – 25100 (step 100) |
| Missing values | 5,460 (~20% of all IV cells) |
| Expiry | Jan 27, 2026 (Nifty weekly expiry) |

---

## Approach

### Core Idea
At any given timestamp, IV values across different strikes lie close to a smooth curve — the **IV smile**. For a missing strike's IV, the best estimate comes from the observed IVs at *other strikes at the same timestamp*, not from past/future timestamps.

### Method: Cross-Sectional Linear Interpolation

For each missing cell at timestamp `t` and strike `K`:
1. Collect all **observed** IV values at the same timestamp `t`
2. Fit a **linear interpolant** over the (strike, IV) pairs
3. Evaluate it at the missing strike `K`

For boundary strikes that require **extrapolation** (e.g. deepest OTM calls/puts), the same linear function is extended outward.

**Fallback:** If a row has fewer than 2 observed values (rare), pandas linear time interpolation fills the gap.

### Why Linear and Not Cubic Spline?

Leave-one-out (LOO) cross-validation showed:

| Method | LOO MSE |
|---|---|
| **Strike-space linear** ← chosen | **0.00003138** |
| Log-moneyness linear | 0.00003210 |
| Strike-space cubic spline | 0.00004721 |
| Log-moneyness cubic spline | 0.00004718 |
| Time-series linear | 0.00024497 |

Cubic splines overfit local curvature and perform worse at boundary strikes requiring extrapolation. Simple linear interpolation is more stable and generalises better.

### No Lookahead Bias
The cross-sectional method uses **only data from the same timestamp** — never future timestamps. The time-series fallback affects less than 1% of predictions.

---

## Repository Structure

```
├── dataset.csv                      # Input data (from competition)
├── iv_surface_prediction.ipynb      # Main notebook — run this to reproduce
├── submission_linear_cross.csv      # Final Kaggle submission
└── README.md                        # This file
```

---

## How to Reproduce

### 1. Install dependencies
```bash
pip install pandas numpy scipy matplotlib seaborn jupyter
```

### 2. Run the notebook
```bash
jupyter notebook iv_surface_prediction.ipynb
```
Run all cells top to bottom. The notebook will generate `submission_linear_cross.csv`.

### 3. Verify
The output CSV will exactly match the submitted file (verified: max diff = 0.00e+00 across all 5,460 predictions).

---

## Notebook Walkthrough

| Section | What it does |
|---|---|
| 1. Imports | All library imports in one place |
| 2. Load Data | Parse datetime, sort by time, inspect shape |
| 3. EDA | Missingness heatmap, IV smile plots, time-series plots, gap analysis |
| 4. Validation | Leave-one-out test on observed values — confirms method quality |
| 5. Predictions | Cross-sectional linear interpolation + fallback + safety clip |
| 6. Inspect | Stats and scatter plot of observed vs predicted |
| 7. Generate CSV | Produce Kaggle submission in required format |
| 8. Summary | Method table, design decisions, key observations |

---

## Key Observations

- **63% of missing values** have immediate time-neighbors (gap = 2 rows), making cross-sectional prediction accurate
- **17.7% of missing values** require extrapolation (boundary strikes) — these are the hardest to predict
- **Expiry day (Jan 27)** shows IV values of 3–6 for deep OTM options — this is real market behaviour (gamma/vega spike), not a data error
- The IV smile across Nifty strikes is **locally linear** in strike space, which is why linear interpolation beats more complex methods

---

## Results

| Submission | Method | Public MSE |
|---|---|---|
| Baseline | Simple mean fill | ~0.0296 |
| v2 | Cubic spline (log-moneyness) | 0.0007237 |
| **Final** | **Linear cross-sectional** | **0.0000423** |

---

## Background — Implied Volatility Surface

In options markets, traders quote prices in terms of **implied volatility** rather than absolute price. The IV surface describes how this volatility estimate varies across:
- **Strike dimension** — the IV smile/skew (higher IV for OTM options)
- **Time dimension** — IV evolves as new information arrives

Accurately filling missing values matters because even small errors in IV translate to meaningful errors in option pricing, delta hedging, and risk management.

---

## License

This project was submitted for academic purposes as part of Finance Club IIT Roorkee Open Projects 2026. The dataset is provided by the competition organisers.
