# Rossmann Demand Forecasting

Forecasts daily store sales for Rossmann drugstores and links forecast error to inventory cost. A gradient boosting model with lag features is benchmarked against a same-weekday mean baseline and ARIMA.

## Business question

Over- and under-forecasting both cost money: too much stock ties up capital and too little loses sales. This project asks how much a better forecast reduces that cost compared with simple baselines.

## Results

Error is measured with MAPE (mean absolute percentage error). Lower is better.

| Model | MAPE (latest validation window) |
|---|---|
| Gradient boosting (lag, promo and holiday features) | **8.4%** |
| Same-weekday mean (baseline) | 18.1% |
| ARIMA | 26.9% |

- **Across 3 rolling windows:** gradient boosting averages 9.2% MAPE versus 19.9% for the same-weekday baseline.
- **Feature check:** removing the promotion and holiday features raises MAPE from 8.4% to 10.7%, so those features contribute to the gain.
- **Inventory cost:** forecast error is converted to an estimated inventory cost, and gradient boosting cuts that cost by about 50% compared with the baseline. This holds across assumed margins of 15-35%. The margin and holding-cost figures are assumptions, not values from the data.

## Approach

1. **Load and clean.** Combine `train.csv` and `store.csv`, remove closed days and zero-sales rows, and run data checks.
2. **Baselines.** Same-weekday mean and ARIMA.
3. **Model.** Gradient boosting with lag features, promotion flags and holiday features.
4. **Validation.** Time-based rolling windows, with training data always earlier than the validation period, so no future information leaks into the model. No random shuffling.
5. **Business link.** Translate forecast error into inventory cost using stated margin and holding-cost assumptions.

## Repository contents

| File | Purpose |
|---|---|
| `rossmann_full_pipeline.ipynb` | Full pipeline: cleaning, baselines, model, validation, cost analysis |
| `rossmann_write_up.md` | Written summary of the method and findings |
| `requirements.txt` | Python dependencies |
| `data/README.md` | Where to download the data |

## Data

Rossmann Store Sales dataset from Kaggle: https://www.kaggle.com/c/rossmann-store-sales

The data files are not included in this repository. Download `train.csv` and `store.csv` and place them in the `data/` folder.

## How to run

```bash
git clone https://github.com/<your-username>/rossmann-demand-forecasting.git
cd rossmann-demand-forecasting
pip install -r requirements.txt
jupyter notebook rossmann_full_pipeline.ipynb
```

Run the notebook from top to bottom. Make sure the data paths in the first cells point to the `data/` folder.

## Limitations

- **Prophet was not run.** A local environment conflict prevented it, so ARIMA is the statistical baseline.
- **Cost estimates rely on assumptions.** The 15-35% margin range and the holding costs are stated assumptions. Real figures from Rossmann would change the saving.
- **Single dataset.** Results are for Rossmann stores in this period and may not transfer to other retailers.
- **Multiple model comparisons.** Models were compared on the same validation windows, so the best-looking result is partly subject to chance.

## Tools

Python, pandas, NumPy, scikit-learn, statsmodels, Jupyter.

## Author

Shashanka Shekhar | [LinkedIn](https://www.linkedin.com/in/shashanka-shekhar2/)
