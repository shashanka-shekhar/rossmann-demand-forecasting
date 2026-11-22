# Rossmann Demand Forecasting (Simplified Pipeline)

Daily sales forecasting for a sample of Rossmann drugstores. The pipeline compares a gradient boosting model against two simple baselines, measures what promotions contribute, and estimates the inventory cost of each forecast.

The whole project runs from one notebook, top to bottom, using the raw Rossmann CSV files.

## Results at a glance

Average across 5 test stores and 3 rolling 28-day windows:

| Model | MAPE % | WAPE % |
| --- | --- | --- |
| Gradient boosting (full features) | 9.18 | 9.04 |
| Gradient boosting without promo/holiday features | 10.70 | 10.63 |
| Same-weekday mean (last 4 weeks) | 19.92 | 19.74 |
| ARIMA with weekly seasonality | 29.05 | 31.96 |

Gradient boosting beat the same-weekday mean in every store-window. Estimated inventory cost at a 15% margin was €90,481 for gradient boosting against €179,866 for the same-weekday mean, a saving of about 50%. Results hold across margins of 15%, 25% and 35%.

## Data

Place these two files in `Rossmann/` under the project root:

- `train.csv`: daily sales, customers, open flag, promotion, holidays. 1,017,209 rows from 2013-01-01 to 2015-07-31.
- `store.csv`: store-level attributes such as store type, assortment, competitor distance and Promo2 details.

The raw data is not included in this repository.

## Method

**Store selection.** A store is eligible if it has a row for at least 95% of calendar days and no gap of 14 or more consecutive missing days. 935 of 1,115 stores qualify. Five test stores are drawn at random from the eligible set with a fixed seed (`42`), giving stores 98, 480, 486, 735 and 865.

**Calendar and leakage rule.** Each eligible store gets a row for every calendar day, so lags count real days. Closed days stay as zero sales. Every lag is at least 28 days old, so a forecast only uses sales known on the last training day. Promotions, holidays and day of week are known in advance and are allowed. Customer counts are not known in advance and are excluded.

**Features.** Day of week, promotion, state and school holidays, month, day of month, ISO week, store type, assortment, competitor distance (with a missing-value flag), Promo2, lags at 28, 35, 42 and 49 days, the mean of those four same-weekday lags, and a 28-day rolling mean lagged by 28 days.

**Models.**

- *Same-weekday mean:* for each weekday, the average sales of that weekday in the last 28 days before the cutoff.
- *ARIMA:* SARIMAX on log(1 + sales) with order (1,0,1) and weekly seasonal order (1,0,1,7), fitted per store.
- *Gradient boosting:* one `HistGradientBoostingRegressor` for all eligible stores, trained on log(1 + sales) using open days up to each cutoff. Settings: 400 iterations, learning rate 0.05, minimum 100 samples per leaf, early stopping off, random state 42.

**Evaluation.** Three rolling windows, with cutoffs 0, 28 and 56 days before the latest cutoff (2015-07-03). Each window forecasts the next 28 days. Scoring uses open days with positive sales only. Metrics are MAPE and WAPE, averaged over stores and then over windows.

**Promotion ablation.** The gradient boosting model is retrained without `Promo`, `Promo2`, `SchoolHoliday` and `StateHoliday` on the same three windows.

**Inventory cost.** The data has no unit costs, so the notebook uses a stated assumption:

- Over-forecast costs (1 − margin) × the over-forecast amount, the purchase cost of unsold stock.
- Under-forecast costs margin × the shortfall, the lost margin on the missed sale.

Margins of 15%, 25% and 35% are tested because the real figure is unknown.

## Results in detail

Per-window MAPE from the full notebook run (the simplified notebook computes the same figures):

| Cutoff | Gradient boosting | Same-weekday mean | ARIMA | GB without promo/holiday |
| --- | --- | --- | --- | --- |
| 2015-05-08 | 8.92 | 18.11 | 29.55 | 9.72 |
| 2015-06-05 | 10.17 | 23.55 | 30.72 | 12.72 |
| 2015-07-03 | 8.43 | 18.11 | 26.87 | 9.65 |

Inventory cost in € across the five test stores and three windows:

| Margin | Gradient boosting | Same-weekday mean | Saving |
| --- | --- | --- | --- |
| 15% | 90,481 | 179,866 | 89,385 (50%) |
| 25% | 90,725 | 184,414 | 93,689 (51%) |
| 35% | 90,970 | 188,962 | 97,992 (52%) |

## Repository layout

The notebook expects this folder structure under the project root:

```
Rossmann/
├── Rossmann/
│   ├── train.csv
│   └── store.csv
└── rsmn/                       (outputs)
    ├── rossmann_clean.csv
    ├── baseline_results.csv
    ├── holdout_daily_predictions.csv
    ├── ablation_results.csv
    └── inventory_cost_results.csv
```

## Setup

Requirements: Python 3.9+, `pandas`, `numpy`, `scikit-learn`, `statsmodels`, and Jupyter.

```
pip install pandas numpy scikit-learn statsmodels jupyter
```

## Running

1. Edit `ROOT` in the first cell to point at your project folder. It is currently set to `C:\Users\reinh\Downloads\Projects\Rossmann`.
2. Make sure the `rsmn/` output folder exists.
3. Run all cells top to bottom.

The notebook takes a while to run because it fits one ARIMA model per store per window and trains three gradient boosting models plus three ablation models.

## Limitations

- **Small test set.** The results cover five stores over three 28-day windows, which is 15 store-windows. The inventory saving is therefore an estimate for this sample, not for the whole chain.
- **Single-day stock decision.** The cost model treats each day's stock as a separate decision and ignores holding stock across days.
- **Assumed margins.** No unit costs are in the data, so the cost results depend on the tested margin range.

## Notes

- Results are reproducible because the random seed for test-store selection and the model's `random_state` are both fixed.
- Closed days are kept as zero sales on the full calendar so that weekly lags line up correctly.
