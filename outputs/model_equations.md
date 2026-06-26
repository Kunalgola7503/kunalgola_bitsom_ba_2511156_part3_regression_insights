# Model Equations

## Dummy Variable Approach

`store_type` (Mall, High Street, Residential, Airport) is converted into 3 dummy variables: `store_type_Mall`, `store_type_Residential`, `store_type_Airport`.

**Reference category: High Street** (the most common type in the data, n = 116 of 320). High Street is intentionally left out of the model — including a dummy for all 4 categories alongside an intercept would create perfect multicollinearity (the "dummy variable trap"), since the 4 dummy columns would always sum to 1 and become a linear combination of the intercept term. Each included dummy's coefficient is read as: *"the difference in monthly sales versus a comparable High Street store, holding every other variable constant."*

`region` was considered as a second candidate categorical variable but was not included in the final model — see `analysis/residual_analysis.md` for evidence that region still carries some independent signal (a natural next step for a future iteration), and `analysis/model_comparison.md` for the full reasoning on model selection.

## Simple Regression Equations

**Simple Model 1:**
`monthly_sales = 560,777.35 + 2.1296 × marketing_spend`
R² = 0.1672

**Simple Model 2:**
`monthly_sales = 446,410.58 + 35.6780 × footfall`
R² = 0.7363

**Simple Model 3:**
`monthly_sales = 697,835.63 − 138,730.45 × avg_discount_pct`
R² = 0.0083 (not statistically significant, p = 0.104)

## Multiple Regression Equation (FINAL MODEL — Model B)

```
monthly_sales =  88,595.77
              +      1.2075 × marketing_spend
              +     33.4945 × footfall
              − 53,647.68   × avg_discount_pct
              +  3,117.09   × inventory_availability_pct
              −  3,039.49   × competitor_distance_km
              + 12,000.53   × holiday_flag
              + 12,309.48   × customer_rating
              + 11,948.28   × store_type_Mall
              − 18,844.93   × store_type_Residential
              + 24,123.17   × store_type_Airport
```
R² = 0.8431 | Adjusted R² = 0.8380 | N = 320

## Explanation of Each Coefficient (Final Model)

| Variable | Coefficient | Meaning in plain business language |
|---|---|---|
| Intercept | 88,595.77 | The model's baseline starting point for a High Street store with every numeric input at zero. Not meaningful as a standalone number (no real store has zero footfall), but mathematically required to anchor the line. |
| marketing_spend | +1.2075 | Every extra ₹1 spent on marketing in a month is associated with about ₹1.21 more in that month's sales, holding everything else fixed. |
| footfall | +33.49 | Every extra visitor in a month is associated with about ₹33 more in sales — the single strongest lever in the model. |
| avg_discount_pct | −53,647.68 | Direction suggests heavier discounting is associated with lower sales, but this is **not statistically significant** (p = 0.12) — treat as inconclusive, not a real effect. |
| inventory_availability_pct | +3,117.09 | Each extra percentage point of product availability is associated with about ₹3,117 more in monthly sales. |
| competitor_distance_km | −3,039.49 | Stores farther from a competitor show *lower* sales in this data. This is almost certainly **not a causal "competition helps" effect** — it more likely reflects that stores near competitors tend to sit in busier, higher-traffic retail clusters in the first place. |
| holiday_flag | +12,000.53 | Months with a meaningful holiday effect are associated with about ₹12,000 more in sales — directionally sensible, though it falls just short of the conventional 5% significance threshold (p = 0.059). |
| customer_rating | +12,309.48 | Each extra star of average customer rating is associated with about ₹12,309 more in monthly sales, once store size/scale is accounted for. |
| store_type_Mall | +11,948.28 | A Mall store sells about ₹11,948 more per month than an equivalent High Street store (borderline significance, p = 0.056). |
| store_type_Residential | −18,844.93 | A Residential-area store sells about ₹18,845 *less* per month than an equivalent High Street store. |
| store_type_Airport | +24,123.17 | An Airport store sells about ₹24,123 *more* per month than an equivalent High Street store — the largest store-type effect in the model. |

## Final Model Selected and Why

**Model B (the equation above) is the final model.** It was chosen over the larger "Model A" (which additionally included `staff_count`) because `staff_count` and `footfall` are highly correlated (r = 0.92) — keeping both variables in the same regression makes their individual coefficients unstable and hard to trust, even though the model's overall fit barely changes. Dropping `staff_count` cost only 0.004 of R² (0.8470 → 0.8431) while making the footfall coefficient's standard error much smaller and more reliable (t-statistic rose from 11.6 to 33.9). Full reasoning in `analysis/model_comparison.md`.
