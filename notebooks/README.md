# Forecasting Consensus Expectations of Nonfarm Payrolls (NFP)

A clean, reproducible setup to preprocess Bloomberg NFP survey data, explore forecast behavior, and generate point & distributional forecasts.

---

## TL;DR (Quick Start)

1) **Install Python (3.10 or 3.11).**  
2) **Open a terminal** in the project folder.  
3) **Create & activate an environment** (pick one):

**Conda**
    
    conda create -n nfp python=3.11 -y
    conda activate nfp
    pip install -r requirements.txt

**venv (built-in)**
    
    python -m venv .venv
    # Windows
    .venv\Scripts\activate
    # macOS/Linux
    source .venv/bin/activate
    
    pip install --upgrade pip
    pip install -r requirements.txt

4) **Put data in the right place**
- Copy your Bloomberg historical workbook to: `raw/nfp_historical.xlsx`
- (Optional) add any “single-release” XLSX files to `raw/` too (not recommended unless necessary, loading a single historical workbook is easiest)

5) **Launch Jupyter**
    
    jupyter lab
    # or
    jupyter notebook

6) **Run notebooks in order**
- `notebooks/01_preprocess.ipynb` → writes `out/nfp_df.parquet` and `out/nfp_df_full.parquet`
- `notebooks/02_explore.ipynb`     → diagnostic plots/tables (optional, informs modeling)
- `notebooks/03_forecast.ipynb`    → forecasting pipeline & intervals

---

## What’s inside

    nfp-forecasting/
    ├─ notebooks/
    │  ├─ 01_preprocess.ipynb   # Clean Bloomberg data → long panel + parquet
    │  ├─ 02_explore.ipynb      # Exploratory checks (errors, tails, spreads)
    │  └─ 03_forecast.ipynb     # Point ensembles + dynamic MV + intervals
    ├─ raw/
    │  └─ nfp_historical.xlsx   # (you provide/update) Bloomberg NFP workbook
    ├─ out/
    |  └─data
    │    ├─ nfp_df.parquet        # COVID-filtered panel (exported by 01)
    │    └─ nfp_df_full.parquet   # Full panel (exported by 01)
    ├─ requirements.txt
    └─ README.md

### Notebook summaries

- **01_preprocess**  
  Loads Bloomberg’s historical workbook (plus optional single-release sheets), enforces strict format checks, reshapes to a one-row-per-economist-per-release panel, and computes:
  - `surprise = actual − median_forecast`
  - `error    = forecast − actual`  
  Exports:
  - `out/nfp_df_full.parquet` (2003-06 onward)  
  - `out/nfp_df.parquet` (excludes 2020-01 to 2022-12)

- **02_explore** *(optional but helpful. doesn't impact modeling.)*  
  Rolling RMSE of the crowd median, contiguity coverage, error distribution diagnostics, spread vs. error linkage, regressions, and interval coverage checks using Student-t fits.

- **03_forecast**  
  End-to-end forecasting:
  - **Point ensembles:** inverse-error, EWMA, Soft-BMA, and MWU (multiplicative weights)
  - **Dynamic majority-vote ensemble:** rolling best-combo (k = 3 or 5; T3/T6/T12 windows)
  - **Distributional forecasts:** rolling 24-month Student-t, GARCH(1,1)-t, and Gaussian mixture (with optional crisis spread widening)
  - Built-in **environment & data checks** at the top of the notebook

---

## Dependencies

Everything is pinned at reasonable minimums in `requirements.txt`:

    # Python 3.10–3.11 recommended
    numpy>=1.26
    pandas>=2.1
    scipy>=1.11
    statsmodels>=0.14
    scikit-learn>=1.3
    matplotlib>=3.8
    tqdm>=4.66
    pyarrow>=14
    arch>=6.3
    ipykernel>=6.29
    jupyterlab>=4.0
    openpyxl>=3.1
    seaborn>=0.13
    plotly>=5.20
    packaging>=23

Tip: Notebook 3 begins with an **Environment Check** cell that verifies versions and presence of the parquet files before running anything heavy.

---

## Data you need to provide

- **Bloomberg NFP workbook** (the survey export with “Economist/Firm”, “Median”, “Actual”, and per-forecast “As of” columns).  
  Place it at: `raw/nfp_historical.xlsx`

- **Optional single-release files** (recent months not yet in the historical workbook).  
  Put them in `raw/` and list them in the `SINGLE_RELEASES` variable in *01_preprocess* (there’s a commented example in the notebook).

The repository intentionally excludes Bloomberg data. Only the two generated parquet files are kept in `out/`.

---

## How to run (step-by-step)

1) **Preprocess**
   - Open `notebooks/01_preprocess.ipynb`.
   - Confirm `HIST_PATH = "../raw/nfp_historical.xlsx"`.
   - (Optional) add any items to `SINGLE_RELEASES` as shown in the notebook.
   - Run all cells. You should see:
     - `out/nfp_df.parquet`
     - `out/nfp_df_full.parquet`

2) **Explore** *(optional)*
   - Open `notebooks/02_explore.ipynb`.
   - It reads the parquet files from `out/` and produces diagnostic plots and small tables.

3) **Forecast**
   - Open `notebooks/03_forecast.ipynb`.
   - Run the **Environment & data check** cell first (it will error early if something is missing).
   - Run the sections you care about:
     - **1.* Point Forecast Ensembles** for signals / weight snapshots
     - **1.7 Dynamic MV Ensemble** for rolling majority-vote benchmark
     - **2.* Distributional Forecasting** for calibrated bands

---

## Tuning knobs (where to change behavior/tune hyperparameters)

Each knob is declared at the top of the relevant section in *03_forecast*.

**Point ensembles**
- Inverse-error / EWMA: `CONT_WINDOWS = [3, 6, 12]`, `METHODS`, `DECAYS`, `RIDGE`
- Soft-BMA: `NU_GRID = [3, 5, 10, 25]`
- MWU: `ETA_GRID`, `ALPHA_NEW`, `WEIGHT_CAP`, `WEIGHT_FLOOR`, `MIN_EXPERTS`,
  `PROBATION_M`, `MAX_SLEEP`, `MAX_MISS_12`
- Candidate pool alternative: `RUN_TOP_HR_POOL`, `TOP_HR_N`
- Dynamic MV tie-breakers & averaging: `K_SET`, `WINDOWS`, `USE_AVG_DIR`, `LAMBDA_AC`

**Intervals**
- Rolling window & nominal levels: `ROLL_WIN = 24`, `MIN_TRAIN = 24`, `LEVELS`
- Crisis widening: `BETA_BASE`, `BETA_CRIS`, `PCTL_THRES`
- GARCH scaling: `SCALE = 100.0`
- GMM: `K_GRID = range(1,5)`, `N_SIMS = 100_000`

---

## Exploring results & saving artifacts

A few useful in-memory objects from *03_forecast*:

- **Live weight snapshots** (latest month, per model/spec/economist):  
  `LIVE_WEIGHT_SNAPSHOTS` → `pd.DataFrame(LIVE_WEIGHT_SNAPSHOTS)`
- **Full weight history** (by month, used weights only):  
  `WEIGHT_HISTORY` → `pd.DataFrame(WEIGHT_HISTORY)`
- **Backtests / live rows / OOS tracks**:  
  - `eval_tables[model]["Full"|"COVID"]` – realized evaluation per spec  
  - `live_tables[model]["Full"|"COVID"]` – live rows per spec  
  - `oos_maps[model]["Full"|"COVID"][spec_id]` – time series of `smart`, `median`, `actual`

To export any of these:

    import pandas as pd
    pd.DataFrame(LIVE_WEIGHT_SNAPSHOTS).to_csv("out/live_weight_snapshots.csv", index=False)
    pd.DataFrame(WEIGHT_HISTORY).to_csv("out/weight_history.csv", index=False)

Note: In *02_explore*, most outputs are visual/printed and are not saved by default.

---

## Notes on panels & contiguity

- Two panels are used throughout:
  - **Full**: full history from 2003-06 onward
  - **COVID-filtered**: excludes 2020-01 to 2022-12  
- For short look-backs that fall entirely in the COVID gap, some methods **fall back to the Full panel** to compute weights (explicit in the code, don't want 2023 forecasts to use 2019 data).
- Strict **contiguity**: for a given look-back window, economists must have forecasts in **every** month of that window to be included.

---

## Troubleshooting

- **Missing package error** → run `pip install -r requirements.txt` in your active environment.  
- **Cannot find `raw/nfp_historical.xlsx`** → confirm the file path and name match exactly.  
- **Parquet read/write error** → ensure `pyarrow` is installed (it’s in `requirements.txt`).  
- **Jupyter not found** → after activating your environment, run `pip install jupyterlab` (already in `requirements.txt`).  

---