# Demand Forecasting for Rossmann Stores

## Problem

A store that orders too much stock wastes money on unsold product. A store that orders too little loses sales. This project tests whether a gradient boosting model forecasts daily store sales more accurately than simple baselines, and what the difference is worth in inventory cost.

## Data

Rossmann Store Sales (Kaggle): daily sales and store attributes for 1,115 German drugstores, from 1 January 2013 to 31 July 2015 (942 calendar days).

- **Eligible stores:** 935 of 1,115, with full coverage and no gap of 14 or more days.
- **Test stores:** five stores chosen at random from the eligible set, with a fixed seed: 98, 480, 486, 735 and 865. All five are closed on Sundays, which is typical for the chain. A larger check adds 20 more stores drawn at random from the rest of the eligible set (seed 2026), giving 25 in total.

## Method

**Cleaning.** Closed days are kept as zero sales on a full daily calendar, so weekly lags are correct. A missing row (the store is absent from the file) is treated as a gap, not as a closure. Stores with long gaps are excluded from training.

**Features.** Every lag is at least 28 days old, so a forecast never uses information from the period it is predicting. Features are: lagged sales at 28, 35, 42 and 49 days; the same-weekday average over four weeks; a 28-day rolling mean; day of week; promotions; school and state holidays; store type; assortment; competition distance; and month and week of year. Promotions and holidays are known in advance, so they are allowed. Customer counts are not known in advance, so they are excluded.

**Models.**
- Gradient boosting (scikit-learn `HistGradientBoostingRegressor`), trained once per window across all 935 eligible stores, on log sales.
- Same-weekday mean over the last four weeks, per store.
- ARIMA with weekly seasonality, per store, run on all three windows. It is univariate, so it uses sales only, with no promotions or holidays.
- Prophet was planned but not run, because of a conflict in the local Python environment.

**Validation.** Three non-overlapping 28-day holdout windows for the five test stores, ending on 3 July, 5 June and 8 May 2015. Each model is scored on open days with positive sales. Training uses only data before the window starts.

The 8 May window has 21 or 22 open days per store, not 24. The missing days are public holidays. Ascension Day (14 May) and Whit Monday (25 May) are closed for all five stores and flagged as public holidays in the data. Corpus Christi (4 June) is closed for three stores and open for two, which fits a holiday that applies only in some regions. Closed days are not forecast or scored, so the comparison is fair across all three windows.

**Inventory cost.** The data has no unit costs, so the cost uses a simple newsvendor-style rule in euros of sales value:
- Over-forecasting costs the write-off of unsold stock, which is (1 − margin) of the sales value.
- Under-forecasting costs the margin on the lost sale.

The margin is tested at 15%, 25% and 35%, because the true figure is unknown.

## Results

**Latest window (average across five test stores)**

| Model | MAPE | WAPE |
|---|---|---|
| Gradient boosting | 8.4% | 8.2% |
| Same-weekday mean | 18.1% | 17.3% |
| ARIMA | 26.9% | 29.5% |

**All three windows**

- Gradient boosting averages **9.2% MAPE**, against **19.9%** for the same-weekday mean and **29.1%** for ARIMA.
- ARIMA is worse than the same-weekday mean in every window (29.6%, 30.7% and 26.9%, from the earliest window to the latest).
- For the original five stores, gradient boosting beats the same-weekday mean in all 15 store-windows.
- Its accuracy varies less between windows (standard deviation of window averages 0.9, against 3.1 for the same-weekday mean).

**Larger sample (25 stores).** The same three windows, scored on 25 stores. The 20 new stores give results close to the original five: gradient boosting averages 9.0% MAPE and the same-weekday mean 18.1%. Across all 25 stores, gradient boosting averages 9.0% and the same-weekday mean 18.5%. On its three-window average, gradient boosting is better for all 25 stores. Across all 75 store-windows, it wins in 74. The exception is store 733 in the 8 May window (7.6% against 7.3%). A 95% bootstrap interval over stores puts the average gap at 8.2 to 10.7 MAPE points. The window-to-window spread also stays narrow (standard deviation 0.7 for gradient boosting, against 2.6 for the same-weekday mean).

The five original stores show a larger gap than the 20 new ones (10.7 against 9.2 points). That is one reason to read the five-store cost table as indicative only.

**Promotion ablation.** Removing the promotion and holiday features raises gradient boosting's average MAPE from 9.2% to 10.7%. The model without those features still beats the same-weekday mean in every window, so most of the gain comes from the lag and store structure rather than promotions alone.

**Inventory cost (five stores, three windows, 120 store-days per full window)**

| Margin | Gradient boosting | Same-weekday mean | Saving |
|---|---|---|---|
| 15% | €90,481 (4.5% of sales) | €179,866 (9.0%) | €89,385 (50%) |
| 25% | €90,725 (4.6%) | €184,414 (9.3%) | €93,689 (51%) |
| 35% | €90,970 (4.6%) | €188,962 (9.5%) | €97,992 (52%) |

The saving barely moves with the margin assumption, because it comes mainly from the accuracy gap.

## Limitations

- **Limited windows and period.** Three 28-day windows, all between May and July 2015. The sample is now 25 stores, and the gap holds for the 20 new ones, but the results cover one season and one year. Inventory cost is still computed on the original five stores only.
- **Weak univariate baseline.** ARIMA uses sales only, with no promotions, holidays or competition. Its gap to gradient boosting partly reflects missing information, not only a weaker seasonal model.
- **Unequal information.** The same-weekday mean does not use promotions, holidays or competition. Part of the gap comes from extra information, not only from a better model.
- **Assumed costs.** The margins and write-off rule are assumptions. The cost also assumes one day's stock is the decision unit and ignores holding over several days.
- **Ablation scope.** The ablation removed four features together, so it does not isolate promotions alone.

## Next steps

1. Repeat the evaluation on more windows, including other seasons, and across all eligible stores (rolling-origin validation). The store sample is now 25 of 935.
2. Run Prophet in a clean Linux or conda environment and add it to the comparison.
3. Test promotions on their own, to separate their effect from the holiday features.

## Tools

Python, pandas, NumPy, scikit-learn, statsmodels, Jupyter. All free.

## Reproduce

Run `rossmann_full_pipeline.ipynb` top to bottom. It writes the results to `Rossmann\processed`: `baseline_results.csv`, `holdout_daily_predictions.csv`, `ablation_results.csv`, `inventory_cost_results.csv` and `arima_all_windows.csv`. Then run `rossmann_sample_extension.ipynb` to produce the 25-store results in `sample_extension_results.csv`.

## Summary for a CV

Built a demand forecasting pipeline on 935 Rossmann stores in Python. Gradient boosting with lag features cut MAPE from 19.9% to 9.2% against a same-weekday baseline, across three 28-day holdout windows, and estimated a 50% reduction in stock-related cost under stated margin assumptions.
