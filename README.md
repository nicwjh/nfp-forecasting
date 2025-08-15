# Forecasting Consensus Expectations of Nonfarm Payrolls (NFP)

<p align="center">
  <img src="https://github.com/nicwjh/nfp-forecasting/blob/main/figures/project_logo.png.png" alt="NFP Forecasting Logo" width="300" />
</p>

[![License][license-shield]][license-url]

## Overview

A professional, reproducible framework for **real-time point and distributional forecasts** of U.S. Nonfarm Payrolls (NFP).  
The project ingests Bloomberg survey data, builds **ensemble point forecasts** and **calibrated predictive intervals**, and evaluates performance with a common rubric;signals are transparent, comparable, and ready for portfolio use.

If you’re looking for setup instructions, data placement, and exact run order, see **`MANUAL.md`** (kept separate from this README to keep the front page concise).

---

## What this repository provides

- **Unified pipeline** that is indicator-agnostic once data are harmonized.
- **Point & directional ensembles**
  - Inverse-error (static), EWMA (decayed), soft-BMA (t-likelihood softmax), MWU (online).
  - A **robust majority-vote** ensemble that selects small, diverse combinations optimized for hit-rate and stability.
- **Distributional engines** for prediction intervals
  - Student-t (rolling), GMM (mixtures), t-GARCH(1,1) (conditional vol), BMA over Normal/t.
  - Optional spread-based **crisis multiplier** to widen bands in high-disagreement months.
- **Evaluation toolkit**
  - RMSE & Diebold–Mariano for levels; Hit Rate, exact Binomial, Pesaran–Timmermann for direction; coverage & mean-absolute gap for intervals; **Accuracy × Consistency (AC)** summaries by regime.
- **Reproducibility**
  - Pinned dependencies in `requirements.txt`, explicit panel/contiguity rules, environment checks at the top of notebooks.

---

## When this is useful

- **Pre-print positioning**: directional calls relative to the survey median and calibrated bands for sizing.
- **Risk & scenario work**: tail probabilities and interval widths that respond to regime shifts.
- **Research transparency**: walk-forward backtests and regime-wise diagnostics.

---

## Notebooks at a glance

- **01_preprocess** — schema checks, long-panel construction, export of `nfp_df.parquet` (COVID-filtered) and `nfp_df_full.parquet` (full history).  
- **02_explore** — exploratory checks that inform modeling (rolling RMSE, spread↔risk, heavy tails, coverage sanity).  
- **03_forecast** — point ensembles, dynamic majority-vote, and distributional engines with walk-forward evaluation and live snapshots.

For exact parameters, toggles, and export helpers (e.g., weights, `oos_maps`), see section “Tuning knobs” and “Exploring results & saving artifacts” in **`MANUAL.md`**.

---

## Getting started

- **Python 3.10–3.11** recommended.  
- Create a clean environment and `pip install -r requirements.txt`.  
- Follow **`MANUAL.md`** to place data in `raw/` and run the notebooks in order.

---

## License

Distributed under the **MIT License** - `LICENSE`. 

[license-shield]: https://img.shields.io/badge/License-MIT-yellow.svg
[license-url]: https://github.com/nicwjh/Macro-research?tab=MIT-1-ov-file
