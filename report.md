# Rossmann Demand Forecasting: Project Report

## Summary

This project forecasts daily sales for a sample of Rossmann drugstores over 28-day horizons. A gradient boosting model, trained across stores with lagged sales and calendar features, cut forecast error to roughly half that of a same-weekday baseline. Using the model would also have lowered estimated inventory cost by about half at every margin tested.

## Data

The project uses the Rossmann training file (1,017,209 rows, 2013-01-01 to 2015-07-31) and the store attribute file. Closed days make up 17% of rows. Closed days always have zero sales, and no negative sales or duplicate store-day rows were found.

## Store selection

A store qualifies if it has records for at least 95% of calendar days and no gap of 14 or more consecutive days. This leaves 935 of 1,115 stores. Five test stores (98, 480, 486, 735 and 865) were drawn at random from this set using a fixed seed, so the selection can be repeated.

## Approach

Every eligible store is placed on a full calendar so that lags count real days. Every lag is at least 28 days old, which means a forecast uses only information available on the last training day. Promotions, holidays and day of week are known in advance and are included. Customer counts are not known in advance and are excluded.

Three models were compared:

- **Same-weekday mean:** the average sales for each weekday over the previous four weeks.
- **ARIMA:** a seasonal model fitted separately to each store on log sales.
- **Gradient boosting:** one model trained on all eligible stores, using calendar features, store attributes, promotion flags and lagged sales.

Each model was tested on three rolling windows, with forecast origins 0, 28 and 56 days before the latest cutoff (2015-07-03). The errors were measured as MAPE (mean absolute percentage error) and WAPE (weighted absolute percentage error), scored on open days with positive sales.

## Results

Averaged across all five stores and three windows:

| Model | MAPE % | WAPE % |
|---|---|---|
| Gradient boosting | 9.18 | 9.04 |
| Gradient boosting, no promo or holiday features | 10.70 | 10.63 |
| Same-weekday mean | 19.92 | 19.74 |
| ARIMA | 29.05 | 31.96 |

Gradient boosting had lower error than the same-weekday mean in every store-window. Its error was also more stable across windows, with a standard deviation of window averages of 0.90 points against 3.14 for the same-weekday mean.

Removing the promotion and holiday features raised the error from 9.18% to 10.70%. Promotions and holidays add real information, but most of the accuracy comes from the lagged sales and store structure.

## Inventory cost

No unit costs are in the data, so the cost analysis uses a stated assumption. Over-forecasting costs the purchase value of unsold stock, and under-forecasting costs the margin lost on a missed sale. Three gross margins were tested.

| Margin | Gradient boosting (€) | Same-weekday mean (€) | Saving (€) |
|---|---|---|---|
| 15% | 90,481 | 179,866 | 89,385 (50%) |
| 25% | 90,725 | 184,414 | 93,689 (51%) |
| 35% | 90,970 | 188,962 | 97,992 (52%) |

The saving is stable across margins, so the conclusion does not depend on the margin assumption.

## Limitations

- The evaluation covers five stores and three 28-day windows, so it is a sample rather than a full-chain test.
- The cost model assumes each day's stock is a separate decision and ignores carrying stock across days.
- Prophet was not used. It conflicted with a system library on the machine where the work was run.

## Next steps

- Test the model on a larger set of stores before relying on the cost estimate.
- Add real unit costs or a holding-cost model to replace the margin assumption.
- Run the full store set through the same pipeline to confirm the results hold at chain scale.
