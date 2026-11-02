# DAX Measures

[Back to README](../README.md)

The final PBIX has **20 measures**. All formulas below are copied from the model. No measure was removed or changed during this review.

| Recommendation | Count | Meaning |
|---|---|---|
| KEEP | 15 | Used and validated |
| REVIEW | 5 | Values are correct, but the name or meaning needs care |
| REMOVE CANDIDATE | 0 | Every measure is used by a visual or by another measure |

**How measures were validated:** results were queried from the model and compared with figures rebuilt from the source workbook. A validated value holds for the filter context tested, not for every possible context.

**Not in the model:** a measure comparing promoted and non-promoted weeks, and a separate Achievement % measure.

## Summary

| Measure | Purpose | Status |
|---|---|---|
| [Total Sales](#total-sales) | Actual sales. | KEEP |
| [Total Budget](#total-budget) | Simulated 2011 target. | KEEP |
| [Variance](#variance) | Sales gap against the simulated budget. | KEEP |
| [Variance %](#variance-) | Sales gap as a share of budget. | KEEP |
| [Total Base 2010](#total-base-2010) | 2010 sales for the matching week. | REVIEW |
| [Variance YoY](#variance-yoy) | Change in sales against the matching 2010 weeks. | KEEP |
| [YoY %](#yoy-) | Separates a real sales decline from missing the target. | REVIEW |
| [System Variance %](#system-variance-) | Benchmark: the result for all 77 stores. | KEEP |
| [Gap vs System](#gap-vs-system) | Shows how far a store sits from the 77-store result. | REVIEW |
| [Sales 2010](#sales-2010) | 2010 sales for year-on-year comparison. | KEEP |
| [Sales 2011](#sales-2011) | 2011 sales for year-on-year comparison. | KEEP |
| [Sales Change](#sales-change) | Change in dollars at any grain, including manufacturer. | KEEP |
| [Sales Change %](#sales-change-) | Change in percent. | KEEP |
| [Total Units](#total-units) | Units sold. | KEEP |
| [Avg Price](#avg-price) | Average realised price. | REVIEW |
| [Units Change %](#units-change-) | Volume change, 2010 to 2011. | KEEP |
| [Price Change %](#price-change-) | Change in average realised price. | REVIEW |
| [Feature Rate](#feature-rate) | Share of sold rows in the in-store circular. | KEEP |
| [Display Rate](#display-rate) | Share of sold rows on promotional display. | KEEP |
| [Discount Rate](#discount-rate) | Share of sold rows priced below base price. | KEEP |

## Total Sales

```dax
Total Sales = sum(Fact_Transactions[SPEND])
```

- **Purpose:** Actual sales.
- **Logic and filter context:** Sums `SPEND` under the current filters.
- **Used for:** All pages or questions
- **Visuals:** Summary (1 visual), Overview (3 visuals), Product (1 visual), Region (1 visual), Store Lookup (3 visuals)
- **Depends on:** Columns only
- **Used by:** `Variance YoY`, `Variance`, `Sales 2010`, `Sales 2011`, `Avg Price`
- **Validation:** Validated against source: 2011 = 9,503,709.30; all rows = 27,927,722.58.
- **Recommendation:** KEEP

## Total Budget

```dax
Total Budget = sum(Budget[Budget])
```

- **Purpose:** Simulated 2011 target.
- **Logic and filter context:** Sums the `Budget` column. Data exists for 2011 only and is not filtered by `Dim_Product`.
- **Used for:** All pages or questions
- **Visuals:** Summary (1 visual), Overview (3 visuals), Store Lookup (3 visuals)
- **Depends on:** Columns only
- **Used by:** `Variance`, `Variance %`
- **Validation:** Rebuilt independently: 10,416,222.58.
- **Recommendation:** KEEP

## Variance

```dax
Variance = [Total Sales] - [Total Budget]
```

- **Purpose:** Sales gap against the simulated budget.
- **Logic and filter context:** `Total Sales` minus `Total Budget`. Needs a Year = 2011 filter, because sales cover all years and budget covers 2011.
- **Used for:** All pages or questions
- **Visuals:** Summary (4 visuals), Overview (3 visuals), Region (2 visuals), Store (2 visuals), Store Lookup (3 visuals)
- **Depends on:** `Total Budget`, `Total Sales`
- **Used by:** `Variance %`
- **Validation:** -912,513.28, and category, state and store roll-ups agree.
- **Recommendation:** KEEP

## Variance %

```dax
Variance % = DIVIDE([Variance],[Total Budget])
```

- **Purpose:** Sales gap as a share of budget.
- **Logic and filter context:** `Variance` / `Total Budget`. Achievement against budget equals 1 + this value (91.24%); the model has no separate Achievement measure.
- **Used for:** All pages or questions
- **Visuals:** Summary (4 visuals), Overview (2 visuals), Region (1 visual), Store (1 visual), Store Lookup (2 visuals)
- **Depends on:** `Total Budget`, `Variance`
- **Used by:** `System Variance %`, `Gap vs System`
- **Validation:** -8.76%.
- **Recommendation:** KEEP

## Total Base 2010

```dax
Total Base 2010 = sum(Budget[Base_2010])
```

- **Purpose:** 2010 sales for the matching week.
- **Logic and filter context:** Sums `Base_2010`, stored against the 2011 week 364 days later. Only meaningful under a 2011 filter.
- **Used for:** Dependency pages or questions
- **Visuals:** None directly; used by other measures
- **Depends on:** Columns only
- **Used by:** `YoY %`, `Variance YoY`
- **Validation:** 9,634,699.37, equal to 2010 sales.
- **Recommendation:** REVIEW. The value is from 2010 but sits on 2011 dates, which is easy to misread. Used as a dependency only.

## Variance YoY

```dax
Variance YoY = [Total Sales] - [Total Base 2010]
```

- **Purpose:** Change in sales against the matching 2010 weeks.
- **Logic and filter context:** `Total Sales` minus `Total Base 2010`.
- **Used for:** Dependency pages or questions
- **Visuals:** None directly; used by other measures
- **Depends on:** `Total Base 2010`, `Total Sales`
- **Used by:** `YoY %`
- **Validation:** -130,990.07.
- **Recommendation:** KEEP

## YoY %

```dax
YoY % = DIVIDE([Variance YoY],[Total Base 2010])
```

- **Purpose:** Separates a real sales decline from missing the target.
- **Logic and filter context:** `Variance YoY` / `Total Base 2010`.
- **Used for:** All pages or questions
- **Visuals:** Summary (3 visuals), Overview (2 visuals), Region (1 visual), Store (1 visual), Store Lookup (2 visuals)
- **Depends on:** `Total Base 2010`, `Variance YoY`
- **Used by:** No other measure
- **Validation:** -1.36%.
- **Recommendation:** REVIEW. Wrong at product level because `Budget` has no product grain (QUAKER returns -96.26%). No current visual uses it that way.

## System Variance %

```dax
System Variance % = CALCULATE([Variance %],all(Dim_Store))
```

- **Purpose:** Benchmark: the result for all 77 stores.
- **Logic and filter context:** `CALCULATE([Variance %], ALL(Dim_Store))` removes store filters and keeps time and category filters.
- **Used for:** Dependency pages or questions
- **Visuals:** None directly; used by other measures
- **Depends on:** `Variance %`
- **Used by:** `Gap vs System`
- **Validation:** Reproduces monthly totals.
- **Recommendation:** KEEP. "System" means the 77 stores in the sample.

## Gap vs System

```dax
Gap vs System = [Variance %] - [System Variance %]
```

- **Purpose:** Shows how far a store sits from the 77-store result.
- **Logic and filter context:** `Variance %` minus `System Variance %`.
- **Used for:** Stores pages or questions
- **Visuals:** Store (1 visual)
- **Depends on:** `System Variance %`, `Variance %`
- **Used by:** No other measure
- **Validation:** Recomputing the rule returns exactly the 18 labelled stores.
- **Recommendation:** REVIEW. The unit is percentage points, although it is formatted as %.

## Sales 2010

```dax
Sales 2010 = CALCULATE([Total Sales],Dim_Date[Year] = 2010)
```

- **Purpose:** 2010 sales for year-on-year comparison.
- **Logic and filter context:** `CALCULATE` replaces the Year filter with 2010; other filters stay.
- **Used for:** Products pages or questions
- **Visuals:** Product (1 visual)
- **Depends on:** `Total Sales`
- **Used by:** `Sales Change`, `Sales Change %`
- **Validation:** 9,634,699.37.
- **Recommendation:** KEEP

## Sales 2011

```dax
Sales 2011 = CALCULATE([Total Sales],Dim_Date[Year] = 2011)
```

- **Purpose:** 2011 sales for year-on-year comparison.
- **Logic and filter context:** `CALCULATE` replaces the Year filter with 2011.
- **Used for:** Products pages or questions
- **Visuals:** Product (1 visual)
- **Depends on:** `Total Sales`
- **Used by:** `Sales Change`
- **Validation:** 9,503,709.30.
- **Recommendation:** KEEP

## Sales Change

```dax
Sales Change = [Sales 2011] - [Sales 2010]
```

- **Purpose:** Change in dollars at any grain, including manufacturer.
- **Logic and filter context:** `Sales 2011` minus `Sales 2010`.
- **Used for:** Products pages or questions
- **Visuals:** Product (1 visual)
- **Depends on:** `Sales 2010`, `Sales 2011`
- **Used by:** `Sales Change %`
- **Validation:** -130,990.07; QUAKER -379,852.04.
- **Recommendation:** KEEP

## Sales Change %

```dax
Sales Change % = DIVIDE([Sales Change],[Sales 2010])
```

- **Purpose:** Change in percent.
- **Logic and filter context:** `Sales Change` / `Sales 2010`.
- **Used for:** Products pages or questions
- **Visuals:** Product (1 visual)
- **Depends on:** `Sales 2010`, `Sales Change`
- **Used by:** No other measure
- **Validation:** -1.36%; QUAKER -51.31%.
- **Recommendation:** KEEP

## Total Units

```dax
Total Units = Sum(Fact_Transactions[UNITS])
```

- **Purpose:** Units sold.
- **Logic and filter context:** Sums `UNITS`.
- **Used for:** Dependency pages or questions
- **Visuals:** None directly; used by other measures
- **Depends on:** Columns only
- **Used by:** `Avg Price`, `Units Change %`
- **Validation:** All rows = 10,293,354, equal to source.
- **Recommendation:** KEEP. Adding units across categories only gives a count of unlike items.

## Avg Price

```dax
Avg Price = DIVIDE([Total Sales],[Total Units])
```

- **Purpose:** Average realised price.
- **Logic and filter context:** `Total Sales` / `Total Units`.
- **Used for:** Dependency pages or questions
- **Visuals:** None directly; used by other measures
- **Depends on:** `Total Sales`, `Total Units`
- **Used by:** `Price Change %`
- **Validation:** Values match source.
- **Recommendation:** REVIEW. Affected by product mix, so it is not a shelf-price change.

## Units Change %

```dax
Units Change % = DIVIDE(
    CALCULATE([Total Units], Dim_Date[Year] = 2011),
    CALCULATE([Total Units], Dim_Date[Year] = 2010)
) - 1
```

- **Purpose:** Volume change, 2010 to 2011.
- **Logic and filter context:** Ratio of `Total Units` for 2011 and 2010, minus 1.
- **Used for:** Products pages or questions
- **Visuals:** Product (1 visual)
- **Depends on:** `Total Units`
- **Used by:** No other measure
- **Validation:** -7.09%; COLD CEREAL -9.39%.
- **Recommendation:** KEEP

## Price Change %

```dax
Price Change % = DIVIDE(
    CALCULATE([Avg Price], Dim_Date[Year] = 2011),
    CALCULATE([Avg Price], Dim_Date[Year] = 2010)
) - 1
```

- **Purpose:** Change in average realised price.
- **Logic and filter context:** Ratio of `Avg Price` for 2011 and 2010, minus 1.
- **Used for:** Products pages or questions
- **Visuals:** Product (1 visual)
- **Depends on:** `Avg Price`
- **Used by:** No other measure
- **Validation:** +6.17%; COLD CEREAL +6.29%.
- **Recommendation:** REVIEW. Includes mix: COLD CEREAL is +5.02% at a fixed 2011 mix.

## Feature Rate

```dax
Feature Rate = AVERAGE(Fact_Transactions[FEATURE])
```

- **Purpose:** Share of sold rows in the in-store circular.
- **Logic and filter context:** Average of the 0/1 `FEATURE` flag.
- **Used for:** Promotions pages or questions
- **Visuals:** Promotion (2 visuals)
- **Depends on:** Columns only
- **Used by:** No other measure
- **Validation:** Monthly values match source.
- **Recommendation:** KEEP. Share of rows with sales, not of revenue.

## Display Rate

```dax
Display Rate = AVERAGE(Fact_Transactions[DISPLAY])
```

- **Purpose:** Share of sold rows on promotional display.
- **Logic and filter context:** Average of the 0/1 `DISPLAY` flag.
- **Used for:** Promotions pages or questions
- **Visuals:** Promotion (2 visuals)
- **Depends on:** Columns only
- **Used by:** No other measure
- **Validation:** Monthly values match source.
- **Recommendation:** KEEP. Share of rows with sales, not of revenue.

## Discount Rate

```dax
Discount Rate = AVERAGE(Fact_Transactions[Is_Discount])
```

- **Purpose:** Share of sold rows priced below base price.
- **Logic and filter context:** Average of `Is_Discount`, a Power Query flag where `PRICE` < `BASE_PRICE`.
- **Used for:** Promotions pages or questions
- **Visuals:** Promotion (2 visuals)
- **Depends on:** Columns only
- **Used by:** No other measure
- **Validation:** Monthly values match source.
- **Recommendation:** KEEP. Includes discounts that also have feature or display, so it differs from `TPR_ONLY`.
