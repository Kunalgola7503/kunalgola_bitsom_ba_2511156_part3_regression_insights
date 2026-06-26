# Final Recommendation — Drivers of Monthly Store Sales

## Which factors appear most strongly associated with monthly sales?

Based on the final regression model (R² = 0.843, `outputs/model_equations.md`), in order of statistical strength:

1. **Footfall** — by far the strongest and most precisely estimated driver (t = 33.9, p ≈ 3e-106). Each visitor is worth roughly ₹33 in monthly sales.
2. **Marketing spend** — a strong, highly significant, and directly actionable lever (p = 2.1e-20): roughly ₹1.21 in sales per ₹1 spent.
3. **Inventory availability** — every percentage point of availability is worth about ₹3,117/month (p = 1.1e-11).
4. **Store type** — Airport stores significantly outsell a comparable High Street store (+₹24,123/month); Residential stores significantly undersell one (−₹18,845/month).
5. **Customer rating** — significantly associated with sales (+₹12,309 per star) once store size is controlled for. Note this relationship is *invisible* in a simple correlation (≈ 0) and only emerges after controlling for footfall/scale — a classic case of a "hidden" effect masked by store size differences.
6. **Distance to nearest competitor** — significantly *negatively* associated with sales, almost certainly reflecting that stores near competitors are in busier commercial areas, not that competition itself boosts sales (see causation note below).

## Which variables should leadership focus on?

**Marketing spend, inventory availability, and customer rating** are the three management can most directly act on and that this analysis shows are genuinely, significantly associated with sales — even after controlling for store size and type. These are the most defensible levers for a business action:
- Increasing marketing spend in under-invested stores
- Tightening inventory/stockout management
- Investing in service quality / customer experience improvements that lift ratings

**Store type and location** are useful for portfolio decisions (e.g., prioritizing Airport-format expansion, being cautious about new Residential-format stores) even though store type itself isn't a lever you "turn" for an existing store.

## Which variables should not be over-interpreted?

- **`avg_discount_pct`** — not statistically significant in either the simple (p = 0.10) or final multiple model (p = 0.12). There is no reliable evidence in this dataset that discounting moves sales up or down. Leadership should not assume a discounting strategy will lift sales based on this data, nor that it hurts sales — the honest answer is "we don't know from this analysis."
- **`competitor_distance_km`** — statistically significant, but very likely **not causal**. Treating this as "move closer to competitors to boost sales" would be a mistake; it almost certainly reflects that proximity to competitors is a marker of being in a high-traffic commercial zone, not a cause of higher sales in itself.
- **`holiday_flag` and `store_type_Mall`** — both borderline (p ≈ 0.06), directionally sensible but not confidently distinguishable from no effect at conventional significance levels. Treat as "suggestive, not proven."
- **`staff_count`** — deliberately excluded from the final model because of its strong overlap with footfall (r = 0.92); its standalone effect cannot be cleanly separated from footfall's effect with this data.

## What business action would you recommend?

1. **Protect and reallocate marketing spend toward stores/regions with currently low spend and high footfall potential** — the relationship is strong, significant, and directly controllable.
2. **Prioritize fixing inventory availability gaps** — a clear, significant, controllable lever with a large estimated payoff per percentage point.
3. **Invest in service-quality initiatives that lift customer ratings**, especially in stores where ratings lag — the model suggests a real payoff once store-size effects are removed.
4. **Re-examine the Mall store format** — Mall stores show the most inconsistent performance of any store type (see `analysis/residual_analysis.md`); some Malls beat expectations and some badly miss them, suggesting execution/location issues within the format that a single dummy variable can't see. A store-by-store review of underperforming Malls (e.g., STR-1023, STR-1068, STR-1007) is a reasonable next step.
5. **Do not change discounting strategy based on this analysis alone** — the evidence is inconclusive either way.

## What risks or limitations should leadership keep in mind?

- **This is 4 months of data for 80 stores** — a relatively short window that may not capture seasonal effects beyond Jan–Apr, or longer-term trends.
- **Footfall and staff_count are entangled** — we cannot cleanly say how much of footfall's effect is really "footfall" versus a proxy for overall store scale/investment.
- **Two variables required median imputation** (`competitor_distance_km`: 6 rows; `customer_rating`: 8 rows) — a small share of the data, unlikely to materially change conclusions, but not zero risk.
- **Mall stores show the widest unexplained variation** — predictions for Mall stores should be treated with the most caution of the four store types.
- **`monthly_profit` was deliberately excluded as a predictor** — it is a downstream consequence of sales (not an independent driver of it), so including it would have created circular, misleading "predictive power."

## Why does regression show association but not automatically prove causation?

Regression coefficients describe **how variables move together in the data we observed** — they do not, by themselves, tell us what would happen if we *intervened* and changed one variable while holding the world fixed. Three reasons this matters here specifically:

1. **Reverse causality / simultaneity:** stores might increase marketing spend or staffing *because* they already see (or anticipate) strong footfall, not the other way around — the arrow of cause could run either direction or both.
2. **Omitted variables / confounding:** `competitor_distance_km`'s negative coefficient is best explained by an omitted variable — local commercial density — that drives both competitor proximity and sales. The regression cannot distinguish "being near a competitor causes higher sales" from "being in a busy area causes both higher sales and more nearby competitors."
3. **No randomization:** stores were not randomly assigned their marketing budgets, locations, or staffing levels — they were chosen by business decisions that likely already reflect expectations about that store's potential. This is fundamentally different from the randomized experiment in Part 2 of this project, where Control/Treatment assignment was random and therefore supports a causal claim. A regression on observational data like this one can support a recommendation ("these factors are worth testing or prioritizing") but cannot, on its own, prove that changing one variable will produce the predicted change in sales. The next step to move from association to causation would be a controlled test (e.g., raising marketing spend in a randomly chosen subset of stores and measuring the actual change).

**Bottom line:** this analysis is strong evidence for *where to focus attention and testing effort* (marketing spend, inventory, service quality), not proof that pulling any one lever will mechanically produce the exact ₹-per-unit effect estimated here.
