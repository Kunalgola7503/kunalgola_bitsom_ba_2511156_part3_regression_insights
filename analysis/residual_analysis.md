# Residual Analysis — Final Model (Model B)

Predicted sales and residuals for all 320 store-months are calculated in `analysis/regression_workbook.xlsx → Predictions_Residuals`, using the Model B coefficients (`predicted_sales = intercept + Σ coefficient × variable`, formula-driven, not hardcoded). `residual = monthly_sales − predicted_sales`. Visual evidence: `screenshots/residuals_preview.png`.

Residuals average to ~0 by construction (mean ≈ 1.3e-8), with a standard deviation of about ₹41,100 — i.e., a typical store-month prediction is off by roughly ±₹41K on monthly sales that average ₹681K (about ±6%).

## Top 5 Largest POSITIVE Residuals (model under-predicts)

| Rank | store_id | Month | Store Type | Region | Actual Sales | Predicted Sales | Residual |
|---|---|---|---|---|---|---|---|
| 1 | STR-1058 | Feb-2025 | High Street | East | 870,937 | 767,424 | +103,514 |
| 2 | STR-1069 | Apr-2025 | High Street | West | 686,738 | 589,093 | +97,645 |
| 3 | STR-1030 | Feb-2025 | Residential | West | 820,519 | 728,664 | +91,855 |
| 4 | STR-1028 | Apr-2025 | Mall | East | 713,611 | 623,402 | +90,209 |
| 5 | STR-1059 | Feb-2025 | Residential | West | 723,305 | 636,466 | +86,839 |

**What this means in business terms:** these stores significantly out-perform what their marketing spend, footfall, inventory, rating, and store type would predict. This points to something the model doesn't capture — strong local management, a one-off local event, a particularly effective in-store promotion, or a nearby demand shock (e.g. a local event drawing in customers) that isn't in this dataset. These are good candidates for a **best-practice review** — what is STR-1058 doing in February that the model doesn't know about?

## Top 5 Largest NEGATIVE Residuals (model over-predicts)

| Rank | store_id | Month | Store Type | Region | Actual Sales | Predicted Sales | Residual |
|---|---|---|---|---|---|---|---|
| 1 | STR-1023 | Feb-2025 | Mall | South | 627,172 | 765,255 | −138,083 |
| 2 | STR-1017 | Mar-2025 | High Street | West | 685,379 | 803,620 | −118,241 |
| 3 | STR-1068 | Jan-2025 | Mall | East | 660,634 | 761,574 | −100,940 |
| 4 | STR-1012 | Jan-2025 | Residential | West | 595,468 | 691,725 | −96,258 |
| 5 | STR-1007 | Feb-2025 | Mall | West | 800,452 | 895,528 | −95,077 |

**What this means in business terms:** these stores under-perform what the model expects given their inputs. **3 of the 5 worst over-predictions are Mall stores** — worth flagging on its own (see below). This could reflect local execution issues, an unusually competitive local environment not fully captured by `competitor_distance_km`, a stockout not reflected in the monthly-average inventory figure, or simply a weaker month that regression to a single average can't anticipate.

## Is the Model Systematically Under- or Over-Predicting Certain Store Types?

- **By construction, the average residual for every `store_type` is ~0** — the store_type dummy variables in the model fully absorb each category's average difference from the High Street reference. So there's no *overall* bias by store type.
- **However, Mall stores show the widest spread of residuals.** 3 of the 5 largest negative residuals are Mall stores, while Mall does not appear at all among the top 5 positive residuals. Combined with `store_type_Mall` being only borderline significant (p ≈ 0.056) in the final model, this suggests Mall performance is more heterogeneous than the other store types — a single "Mall" dummy may be averaging over quite different real-world situations (e.g. a flagship mall location vs. a struggling one) that the model cannot tell apart.
- **By region, the model is not perfectly neutral**, even though region was not included as a predictor: average residuals are East ≈ −12,600, North ≈ −2,400, South ≈ +7,000, West ≈ +11,400. East stores tend to be *over*-predicted and West stores *under*-predicted, on average. This is a reasonable signal that region carries some independent information about sales that the current model doesn't capture — a natural next step (see Recommendation) would be to test region dummies in a future iteration.

## Practical Takeaway

The residual pattern says: trust the model's average story (footfall, marketing spend, inventory, and store type genuinely move sales), but don't expect it to perfectly predict any single store-month — and treat Mall-store predictions in particular with a bit more caution than High Street, Residential, or Airport.
