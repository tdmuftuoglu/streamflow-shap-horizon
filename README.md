# streamflow-shap-horizon

Code and outputs for the manuscript

> Müftüoğlu, T. D. *When Does a Streamflow Forecast Stop Being a Forecast? A SHAP-Based Framework for Detecting Predictability Horizon Collapse Across Contrasting Polish Catchments.* (manuscript under review)

Archived version: [https://doi.org/10.5281/zenodo.20579070](https://doi.org/10.5281/zenodo.20579070)

## What the code does

The script trains gradient-boosting models (LightGBM and XGBoost) that forecast daily discharge *h* days ahead (*h* = 1, 2, 3, 4, 5, 6, 7, 10 and 14 days) at five Polish gauging stations. It explains the models with SHAP (SHapley Additive exPlanations) and determines the **SHAP-based predictability horizon (SHAP-H)**.

**SHAP-H** is the shortest lead time at which any of the four day-of-year harmonics (`doy_sin`, `doy_cos`, `doy_sin2`, `doy_cos2`) is among the three predictors with the largest mean |SHAP| value. Beyond this lead time, the forecast relies mainly on the calendar rather than on the current state of the catchment. SHAP-H is compared with the conventional R²-based horizon: the longest lead time up to which the test R² stays ≥ 0.30.

Everything is in one file, so the analysis can be pasted into a single Google Colab cell.

## Repository contents

| File | Description |
|---|---|
| `streamflow_shapH.ipynb` | Analysis as a one-cell notebook (Google Colab) |
| `streamflow_shapH.py` | The same code as a plain Python script |
| `requirements.txt` | Package versions used for the published results |
| `shapH_outputs.zip` | Outputs of the published run: tables (CSV), figures (TIFF + PNG) and `RUN_LOG.txt` |
| `README.md` | This file |

The input data are **not** included (see [Data](#data)).

## Stations

| Station | River | GRDC No. | Catchment area (km²) | NASA POWER point (lat, lon) |
|---|---|---|---|---|
| Nowy Targ | Dunajec | 6458402 | 435 | 49.49, 20.03 |
| Nowy Sącz | Dunajec | 6458406 | 4,337 | 49.63, 20.69 |
| Strękowa Góra | Narew | 6458165 | 7,199 | 53.22, 22.30 |
| Sieradz | Warta | 6457840 | 8,156 | 51.59, 18.73 |
| Sandomierz | Vistula | 6458460 | 31,809 | 50.68, 21.75 |

## Data

### Discharge (GRDC)

Mean daily discharge is available from the [GRDC Data Portal](https://grdc.bafg.de/data/data_portal/) (Global Runoff Data Centre, 56068 Koblenz, Germany). Download the daily series for the five GRDC numbers above in the standard GRDC text format. Under the GRDC data policy, downloaded data may not be redistributed, so the files are not part of this repository.

### Meteorology (NASA POWER)

Daily point data are available from the [NASA POWER Data Access Viewer](https://power.larc.nasa.gov/data-access-viewer/). Request the data as follows:

- **Temporal average:** Daily
- **Location:** the coordinates in the table above
- **Period:** 1984-01-01 to at least 2025-12-31
- **Output format:** CSV
- **Parameters (exactly these 14):** `ALLSKY_SFC_SW_DWN`, `T2M`, `T2MDEW`, `TS`, `T2M_MAX`, `T2M_MIN`, `ALLSKY_SFC_LW_DWN`, `RH2M`, `PRECTOTCORR`, `PS`, `WS2M`, `GWETTOP`, `GWETROOT`, `GWETPROF`

`TS` is downloaded but not used. The loader skips the 22-line header that POWER writes for 14 parameters, so request exactly this set.

### File names

The script expects these ten file names:

```
nowy_targ__dunajec__grdc.txt      nowy_targ__dunajec__nasa.csv
nowy_sacz__dunajec__grdc.txt      nowy_sacz__dunajec__nasa.csv
strekowa_gora__narev__grdc.txt    strekowa_gora__narev__nasa.csv
sieradz__warta__grdc.txt          sieradz__warta__nasa.csv
sandomierz__vistula__grdc.txt     sandomierz__vistula__nasa.csv
```

## How to run

### Google Colab (recommended)

1. Open `streamflow_shapH.ipynb` in Google Colab.
2. Run the cell. A file picker (**Choose Files**) opens. Select the ten input files; they can be selected together.
3. A full run takes about two hours. At the end, `shapH_outputs.zip` is downloaded automatically.

Missing packages (`optuna`, `lightgbm`, `xgboost`, `shap`) are installed automatically.

### Local computer

```bash
pip install -r requirements.txt
python streamflow_shapH.py
```

The input files are searched on the Desktop and its sub-folders. To use another folder, set an environment variable:

```bash
SHAPH_DATA_DIR=/path/to/data python streamflow_shapH.py
```

The outputs are written to `shapH_outputs/` and `shapH_outputs.zip` in the same folder.

### Redrawing the figures only

Set `FIGURES_ONLY = True` at the top of the script and run it. The picker asks for an output zip from an earlier run, for example the `shapH_outputs.zip` in this repository. All figures are then redrawn from its CSV files in about a minute, without refitting any model. Figure 1 is redrawn only when `obs_pred_test.csv` is present (see [Data policy note](#data-policy-note)).

## Method summary

| Step | Setting |
|---|---|
| Daily index | NASA POWER calendar; GRDC discharge left-joined, so gaps in the discharge record do not shift lags or targets |
| Predictors (38) | 12 discharge-memory features (Q(t), lags 1–30 days, rolling means and standard deviation), 22 meteorological features (including soil wetness and rolling sums/means), 4 day-of-year harmonics |
| Effective start | 1988-01-01 (`ALLSKY_SFC_LW_DWN` is missing before 1988; only complete rows are used) |
| Sub-training / validation | 1988–2009 / 2010–2014 |
| Final fit / test | 1988–2014 / 2015 to the end of each record |
| LightGBM tuning | Optuna, TPE sampler, 40 trials, objective = validation R² at *h* = 1; one configuration per station |
| XGBoost | Fixed settings (500 trees, depth 7, learning rate 0.05, subsampling 0.8) |
| Scenarios | Memory, Climate, Memory+Climate |
| Benchmarks | Persistence and day-of-year climatology (31-day smoothing) |
| SHAP | Exact TreeSHAP on all test days |
| SHAP-H variants | top-*k* with *k* = 1–5, annual harmonics only, four harmonics grouped |
| Robustness | 5 extra random seeds, XGBoost SHAP, SHAP on training-period inputs, test-set tuning (comparison only), expanding-window cross-validation 2018–2023 |
| Seasonal analysis | Winter/spring (Dec–May) vs summer/autumn (Jun–Nov) issue dates |

## Outputs

### Figures

| File | Manuscript |
|---|---|
| `Fig_obs_vs_pred` | Figure 1 |
| `Fig_skill_decay` | Figure 2 |
| `Fig_area_skill_horizon` | Figure 3 |
| `Fig_shap_fractions` | Figure 4 |
| `Fig_shap_bars_h1` | Figure 5 |
| `Fig_shap_bars_h7` | Figure 6 |
| `Fig_harmonic_rank` | Figure 7 |
| `Fig_seasonal_r2` | Figure 8 |
| `Fig_seasonal_shap_h7` | Figure 9 |

### Main tables

| File | Content | Manuscript |
|---|---|---|
| `basin_memory_diagnostics.csv` | Discharge statistics, autocorrelation, e-folding time, seasonal variance share | Table 1 |
| `results_all_models.csv` | R², RMSE and MAE for all stations, lead times, scenarios, models and benchmarks | Tables 3 and 5 |
| `table_shapH_summary.csv` | SHAP-H (all variants) and R²-based horizons | Tables 4 and 7 |
| `shap_fractions_main.csv` | SHAP group shares and harmonic ranks per lead time | Table 6 |
| `robustness_seeds.csv` | Seed robustness | Table 7 |
| `legacy_vs_validation.csv` | Validation tuning vs test-set tuning (`legacy` = test-set tuning, comparison only) | Table 8 |
| `cv_expanding_window.csv`, `table_cv_shapH.csv` | Cross-validation folds | Table 9 |
| `seasonal_r2.csv`, `shap_fractions_seasonal.csv` | Seasonal skill and SHAP composition | Figures 8 and 9 |
| `hyperparameters.csv`, `data_periods.csv` | Selected hyperparameters and the periods used | — |
| `RUN_LOG.txt` | Everything printed during the run, including package versions | — |

## Reproducibility

The published results were produced in Google Colab with Python 3.13, lightgbm 4.6.0, xgboost 3.4.1, shap 0.54.1, optuna 5.0.0 and scikit-learn 1.6.1 (see `requirements.txt` and `RUN_LOG.txt`). All random seeds are fixed. Other package versions or hardware can still lead Optuna to different hyperparameters and so change the numbers slightly. To reproduce the published values exactly, use the listed versions.

## Data policy note

`obs_pred_test.csv`, written by full runs of the current script, contains observed GRDC discharge for the test period. Because of the GRDC data policy, do not publish this file. The archived `shapH_outputs.zip` does not contain it.

## Citation

If you use this code, please cite the archived version ([https://doi.org/10.5281/zenodo.20579070](https://doi.org/10.5281/zenodo.20579070)) and the associated article once it is published.

## Author

Tevfik Denizhan Müftüoğlu
