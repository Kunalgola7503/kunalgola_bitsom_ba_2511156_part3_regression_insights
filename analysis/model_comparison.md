# Model Comparison

All models were run on the cleaned dataset (320 store-months, 80 stores × 4 months; competitor_distance_km and customer_rating missing values imputed with the column median — see `analysis/regression_workbook.xlsx → Cleaned_Data`). Live computations are in `analysis/regression_workbook.xlsx`; a clean summary export is in `outputs/regression_summary.xlsx`.

## Simple Regression Models

### Model Name: Simple Model 1
- **Variables used:** Dependent = `monthly_sales`; Independent = `marketing_spend`
- **R-squared:** 0.1672
- **Significant variables:** `marketing_spend` is significant (p = 2.48E-14), but the model explains only ~17% of the variation in sales on its own.
- **Business usefulness:** Confirms marketing spend matters, but is not enough on its own to guide a marketing-spend decision — too much of sales is left unexplained.
- **Limitations:** Ignores store size, location, and everything else that drives sales; a store with low spend but huge footfall will look like an outlier this model can't explain.

### Model Name: Simple Model 2
- **Variables used:** Dependent = `monthly_sales`; Independent = `footfall`
- **R-squared:** 0.7363
- **Significant variables:** `footfall` is highly significant (p = 4.75E-94) and explains ~74% of sales variation alone — the strongest single-variable result in this analysis.
- **Business usefulness:** Useful as a sanity check (sales clearly track foot traffic) but doesn't tell leadership anything they can directly act on — you cannot "set" footfall the way you can set a marketing budget.
- **Limitations:** Single-variable models can't separate footfall's effect from related factors like store size/staffing that move together with it.

### Model Name: Simple Model 3
- **Variables used:** Dependent = `monthly_sales`; Independent = `avg_discount_pct`
- **R-squared:** 0.0083
- **Significant variables:** None — `avg_discount_pct` is **not** statistically significant on its own (p = 0.1040).
- **Business usefulness:** Important negative finding — there is no evidence from this simple test that discounting drives sales up or down. Useful for tempering any assumption that "more discount = more sales."
- **Limitations:** A non-significant simple result doesn't fully settle the question — discount could still matter in combination with other factors (it remains in the multiple regression for that reason).

## Multiple Regression Models

### Model Name: Model A — Full Multiple Regression
- **Variables used:** Dependent = `monthly_sales`; Independent = `marketing_spend`, `footfall`, `avg_discount_pct`, `staff_count`, `inventory_availability_pct`, `competitor_distance_km`, `holiday_flag`, `customer_rating`, plus `store_type` dummies (Mall, Residential, Airport; reference = High Street)
- **R-squared:** 0.8470 | **Adjusted R-squared:** 0.8415
- **Significant variables (p < 0.05):** marketing_spend, footfall, staff_count, inventory_availability_pct, competitor_distance_km, holiday_flag, customer_rating, store_type_Residential, store_type_Airport
- **Not significant:** avg_discount_pct (p = 0.089), store_type_Mall (p = 0.060)
- **Business usefulness:** Explains ~85% of sales variation — strong enough to support resource-allocation discussions.
- **Limitations:** `footfall` and `staff_count` are highly correlated (r = 0.92). Including both makes each individual coefficient less stable/trustworthy, even though the model's overall fit is fine. This is a textbook multicollinearity problem.

### Model Name: Model B — FINAL Multiple Regression (selected)
- **Variables used:** Same as Model A, **minus `staff_count`**
- **R-squared:** 0.8431 | **Adjusted R-squared:** 0.8380
- **Significant variables (p < 0.05):** marketing_spend, footfall, inventory_availability_pct, competitor_distance_km, customer_rating, store_type_Residential, store_type_Airport
- **Borderline (0.05–0.10):** holiday_flag (p = 0.059), store_type_Mall (p = 0.056)
- **Not significant:** avg_discount_pct (p = 0.123)
- **Business usefulness:** Virtually the same explanatory power as Model A (R² drops by only 0.004) but with a much more stable, trustworthy footfall coefficient (t-stat jumps from 11.6 to 33.9) since the redundant, collinear `staff_count` variable was removed.
- **Limitations:** Still an associational model (see `outputs/final_recommendation.md` on causation); it is a cross-sectional/panel snapshot of 4 months and may not generalize to a different season or a structurally different store.

## Why Model B Was Selected as Final

1. **Multicollinearity:** `footfall` and `staff_count` are two measurements of essentially the same underlying thing (store size/scale) — keeping both in a regression inflates standard errors and makes coefficients unstable from sample to sample. We kept `footfall` because it is the more fundamental, "upstream" quantity — staffing levels are typically set in response to expected footfall, not the other way around — and because footfall is something leadership can also act on indirectly (e.g., through marketing or location choices), whereas staffing is more of an operational lever for cost, not sales generation, in this dataset.
2. **Negligible cost in fit:** R² fell from 0.8470 to 0.8431 — a difference of 0.4 percentage points — for a large gain in interpretability.
3. **More trustworthy coefficients:** every remaining coefficient's standard error either shrank or stayed about the same, and the footfall coefficient became dramatically more precise.

## Summary Table

| Model | Variables | R² | Adj. R² | Significant Variables | Business Usefulness |
|---|---|---|---|---|---|
| Simple 1 | marketing_spend | 0.1672 | n/a | marketing_spend | Confirms relationship, too narrow alone |
| Simple 2 | footfall | 0.7363 | n/a | footfall | Strong but not directly actionable |
| Simple 3 | avg_discount_pct | 0.0083 | n/a | none | Useful negative finding |
| Model A (Full) | 11 predictors + store_type | 0.8470 | 0.8415 | 9 of 11 | Strong fit, unstable due to collinearity |
| **Model B (Final)** | 10 predictors + store_type | 0.8431 | 0.8380 | 7 of 10 | **Best balance of fit and reliability — selected** |
