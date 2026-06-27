# Regression-Based Business Insights & Model Interpretation

## 1. Business Problem Summary

A retail chain's leadership wants to understand what drives monthly sales performance across stores, to inform decisions on marketing spend, inventory investment, discounting strategy, staffing, and which store types/regions to prioritize. This analysis uses regression to identify which factors are most strongly associated with `monthly_sales`, and provides a data-backed recommendation while being explicit about what regression can and cannot prove.

## 2. Dataset Description

`data/business_regression_data.xlsx`:
- **store_performance** — 320 rows: 80 stores × 4 months (Jan–Apr 2025). Columns cover store identity, region, store type, marketing spend, footfall, discounting, staffing, inventory availability, competitor distance, holiday flag, customer rating, monthly sales, and monthly profit.
- **data_dictionary** — field definitions for every column.

### Dependent Variable
- `monthly_sales` — the target variable for every model in this repository.

### Potential Independent Variables
- **Numerical:** `marketing_spend`, `footfall`, `avg_discount_pct`, `staff_count`, `inventory_availability_pct`, `competitor_distance_km`, `holiday_flag` (0/1), `customer_rating`
- **Categorical:** `region` (East/North/South/West), `store_type` (Mall/High Street/Residential/Airport)

### Variables Needing Cleaning or Transformation
- `competitor_distance_km` — 6 missing values, imputed with the column median
- `customer_rating` — 8 missing values, imputed with the column median
- `store_type` — converted into 3 dummy variables (see below)

### Variables Not Useful for Regression (and why)
- `store_id` — a unique identifier (80 distinct values); not a meaningful numeric or low-cardinality categorical predictor
- `month` — only 4 distinct values in a 4-month window; too short a panel to reliably estimate seasonality, so it was not used as a predictor (flagged as a limitation)
- `monthly_profit` — a downstream consequence of sales, not an independent driver of it; including it as a predictor of `monthly_sales` would be circular (data leakage), so it is excluded from every regression

## 3. Regression Approach

Three simple regressions (`monthly_sales` on one variable at a time) and two multiple regressions were run, all computed live in Excel via the `LINEST()` array function in `analysis/regression_workbook.xlsx` (verified against an independent Python/statsmodels calculation — every coefficient, standard error, t-stat, and p-value matches). The workbook contains 7 sheets: Original_Data, Cleaned_Data, Dummy_Variables, Simple_Regression, Multiple_Regression, Predictions_Residuals, and Model_Comparison.

## 4. Dummy Variable Approach

`store_type` was converted into 3 dummy columns (`store_type_Mall`, `store_type_Residential`, `store_type_Airport`), with **High Street as the reference category** (the most common type, n=116) to avoid the dummy-variable trap. Each dummy's coefficient is read as the sales difference vs. a comparable High Street store. Full explanation in `outputs/model_equations.md`.

## 5. Model Comparison Summary

| Model | Variables | R² | Adj. R² |
|---|---|---|---|
| Simple Model 1 | marketing_spend | 0.1672 | n/a |
| Simple Model 2 | footfall | 0.7363 | n/a |
| Simple Model 3 | avg_discount_pct | 0.0083 (not significant) | n/a |
| Model A (Full) | 11 predictors incl. staff_count | 0.8470 | 0.8415 |
| **Model B (Final)** | 10 predictors, drops staff_count | **0.8431** | **0.8380** |

Model B was selected over Model A because `footfall` and `staff_count` are highly collinear (r = 0.92); dropping `staff_count` cost almost no explanatory power but made the remaining coefficients far more stable. Full detail in `analysis/model_comparison.md`.

## 6. Final Model Selected

**Model B:** `monthly_sales ~ marketing_spend + footfall + avg_discount_pct + inventory_availability_pct + competitor_distance_km + holiday_flag + customer_rating + store_type_Mall + store_type_Residential + store_type_Airport`

R² = 0.8431, Adj. R² = 0.8380, N = 320. Full equation and coefficient-by-coefficient explanation in `outputs/model_equations.md`. Residual analysis (predicted vs. actual, largest over/under-predictions) in `analysis/residual_analysis.md`.

## 7. Business Recommendation

Focus on **marketing spend, inventory availability, and customer rating** — all significantly and directly associated with sales, and all controllable levers. Treat **discounting** as inconclusive (not significant in any model) and **competitor distance** as an association to note, not a lever to pull (almost certainly reflects local commercial density, not a causal competitive effect). Review the **Mall store format** specifically — it shows the widest spread of unexplained performance of any store type. Full recommendation, including why regression shows association rather than proof of causation, is in `outputs/final_recommendation.md`.

## 8. Assumptions and Limitations

- Median imputation was used for 2 variables with a small number of missing values (6 and 8 rows respectively, out of 320).
- `month`/seasonality and `region` were not included as predictors in the final model — both are flagged as reasonable extensions for a future iteration (residual analysis shows region still carries some unexplained signal).
- 4 months of data for 80 stores is a fairly short panel; conclusions may not generalize beyond this window.
- `footfall` and `staff_count` are highly correlated; the model cannot fully separate their individual contributions, which is why `staff_count` was dropped from the final model rather than left in with an unstable coefficient.
- Regression coefficients reflect association in observational data, not proven causation — see `outputs/final_recommendation.md` for the full explanation.

## 9. Screenshots Included

| File | Shows |
|---|---|
| `screenshots/simple_regression_output.png` | Simple regression output (monthly_sales ~ footfall) |
| `screenshots/multiple_regression_output.png` | Final multiple regression output, full coefficient table |
| `screenshots/residuals_preview.png` | Top 5 positive and top 5 negative residuals |
| `screenshots/model_comparison_preview.png` | All 5 models compared side by side |

