# Budget Methodology

[Back to README](../README.md)

**The 2011 budget in this project is simulated.** It was built from historical sales for this analysis. It is not an official budget from dunnhumby or any business.

## 1. Why a budget was needed

The business question asks where sales fell short of target, but the dataset has no target. Comparing with 2010 alone shows whether sales fell, not whether they met an expectation. The simulated budget gives a transparent reference point for this comparison.

## 2. Why 2010 is the baseline

- 2010 is the most recent complete period before 2011. It has all 52 weeks (6 January to 29 December 2010) and all 16,016 store × category × week combinations. "Complete" applies at these levels only: at product × store × week level there are 179,414 of 220,220 possible rows, because rows exist only where a sale exists. This does not affect the budget, which is built at store × category × week grain.
- Each 2010 week carries over seasonal patterns for that store and category.
- 2009 is incomplete in the file: its first week ends on 14 January, and store 387 is missing 25 weeks.

## 3. Calculation

### Formula

```text
g(category) = Sales 2010 comparison weeks / Sales 2009 comparison weeks − 1

Budget(week + 364 days, store, category)
    = Sales 2010(week, store, category) × (1 + g(category))
```

### Growth rate g

g compares 51 matching weeks: 14 January to 30 December 2009, and 13 January to 29 December 2010.

| Category | 2009 sales | 2010 sales | g (rounded) | g (exact value used) |
|---|---:|---:|---:|---:|
| BAG SNACKS | $1,485,403.36 | $1,570,710.19 | +5.74% | 0.05743007744374272 |
| COLD CEREAL | $4,587,540.81 | $5,024,382.63 | +9.52% | 0.09522352783167944 |
| FROZEN PIZZA | $1,899,003.43 | $2,073,618.27 | +9.20% | 0.09195077651860856 |
| ORAL HYGIENE PRODUCTS | $543,009.18 | $529,610.73 | −2.47% | −0.024674444730378886 |

Power Query uses the **unrounded** value. Rounded rates are for display only, and recalculating with them gives a slightly different budget.

The negative rate for ORAL HYGIENE PRODUCTS was kept. Setting a floor of zero would add a judgement the data does not support.

### Why 51 weeks for growth but 52 weeks for evaluation

These periods do different jobs:

| Step | Weeks | Period | Reason |
|---|---:|---|---|
| Estimate g | 51 | 2009 compared with 2010 | The file has only 51 weeks in 2009, starting with the week ending 14 January. The week ending 6 January 2010 has no 2009 match, so it is left out of this comparison |
| Build baseline | 52 | 6 January to 29 December 2010 | All 2010 weeks are available, including the week ending 6 January |
| Evaluate 2011 | 52 | 5 January to 28 December 2011 | Each 2011 week has a 2010 week 364 days earlier |

### How store 387 is treated

Store 387 is missing 25 consecutive weeks in 2009 (weeks ending 11 February to 29 July). Keeping it would understate 2009 sales and inflate growth, for example raising COLD CEREAL from 9.52% to 10.43%.

| Step | Store 387 |
|---|---|
| Estimate g | **Excluded** |
| 2010 baseline | Included; all 52 weeks are present |
| 2011 budget | Included, using its category's g |
| 2011 evaluation | Included; all 52 weeks are present |

### How missing combinations are treated

Across all 156 weeks, `Sales_Weekly` has 47,947 of 48,048 possible store × category × week combinations. **No missing value is filled or estimated.**

| Missing combinations | Year | Count | Treatment |
|---|---:|---:|---|
| Store 387, all 4 categories, 25 weeks | 2009 | 100 | Store excluded from g |
| Store 8035, ORAL HYGIENE PRODUCTS, week ending 26 October | 2011 | 1 | Budget of $12.57 remains. With no sales row, it counts as a $12.57 shortfall |

After excluding store 387, neither g comparison period has a missing combination, and the 2010 baseline has none. The data does not explain the missing combinations. The store 8035 week could mean no sales or missing data; the effect is negligible.

### Implementation (Power Query)

These queries were built in Excel Power Query and imported into Power BI with the same steps.

1. `Sales_Weekly` joins transactions to `CATEGORY`, then groups `SPEND` by week, category and store.
2. `Growth_Category` removes store 387, labels the two comparison periods, and calculates `g`.
3. `Budget` keeps the 2010 weeks, renames sales to `Base_2010`, moves each date forward 364 days, joins `g`, and calculates `[Base_2010] * (1 + [g])`.

### Grain and variation

| Question | Answer |
|---|---|
| Grain | Week × store × category |
| Differs by category? | Yes, through g |
| Differs by store or week? | Yes, through the 2010 base. g itself is the same for every store and week in a category |
| Weekly and monthly allocation | There is no separate allocation. Budget is held by week and added up to months |
| Different numbers of weeks | Moving dates by 364 days compares 52 weeks with 52 weeks on the same weekday |

### Verified example

ERLANGER (store 389), COLD CEREAL, week ending 5 January 2011:

```text
Base (week ending 6 January 2010)  = $2,531.36
g (COLD CEREAL)                    = 9.5224%
Budget                             = 2,531.36 × 1.095224 = $2,772.41
Actual sales                       = $1,757.31
Variance                           = −$1,015.10 (−36.61%)
```

## 4. Validation

Budget was rebuilt separately from the source workbook and compared with the Power BI model.

| Check | Result | Status |
|---|---|---|
| Rows | 16,016 = 52 weeks × 77 stores × 4 categories | PASS |
| Duplicate week, store, category keys | 0 | PASS |
| Budget total | $10,416,222.58 in both | PASS |
| Base total | $9,634,699.37, equal to 2010 sales | PASS |
| g values | Match in both | PASS |
| Weeks covered | 52, from 5 January to 28 December 2011 | PASS |
| Budget against actual join | 16,015 matched; 1 budget row has no sale (store 8035, ORAL HYGIENE PRODUCTS, week ending 26 October 2011, $12.57) | PASS, noted |
| Roll-ups | Category, state and store totals all give −$912,513.28 | PASS |
| Double counting across fact tables | None. Both connect through dimensions | PASS |
| Refresh | Completed on a temporary copy; totals unchanged | PASS |

"Sales Gap" in this project is the `Variance` measure. The model has no Achievement % measure; achievement equals 1 + `Variance %`, or 91.24% for 2011.

## 5. Results depend on the assumption

| Assumption | Effect |
|---|---|
| Current method | Budget expects +8.11%. Gap is −$912,513 (−8.76%) |
| No growth (g = 0) | Gap is −$130,990 (−1.36%), the same as the change from 2010 |
| Keep store 387 in g | g would be 7.26%, 10.43%, 10.19% and −0.98%, which is higher |

## 6. Assumptions and limitations

- The target is **simulated**. Falling short is not evidence that a real target was missed.
- 2009 to 2010 growth may not hold in 2011.
- Each store receives the same category growth rate, regardless of its own trend.
- The budget inherits unusual 2010 weeks, including 2010 promotion timing. A monthly gap can reflect a calendar shift in promotions, not only store performance.
- There is no budget by manufacturer or product. The budget should not be analysed at that level.
- If a product was discontinued, the budget still includes its expected sales. This matters for QUAKER, whose sales fell close to zero in late 2011. The data does not show why.

## 7. Priority store screening

These rules are **project assumptions, not business rules**. They were set after looking at 2011 results, and are used to choose where to investigate first.

| Rule | Definition |
|---|---|
| Measure | `Gap vs System`: store `Variance %` minus the result for all 77 stores in the same month |
| Priority list | Below the 77-store result by more than 10 percentage points, for at least 2 consecutive months |
| High (`Cao`) | At least 6 months beyond the threshold (8 stores) |
| Watch (`Theo dõi`) | 2 to 5 months beyond the threshold (10 stores) |

Recalculating the rule from model results returns exactly the same 18 stores and 8 High stores. The labels are hard-coded in Power Query and do not update with new data.

| Threshold | Stores meeting the rule |
|---|---:|
| 5 points | 40 |
| **10 points (used)** | **18** |
| 15 points | 9 |
| 20 points | 1 |

An earlier rule, more than 5% below budget for 2 consecutive months, was rejected because 74 of 77 stores met it.

**Limitations:**

- 10 points and 6 months were chosen so the list is short enough to act on. They are not external standards.
- Stores were selected for poor results, so their poor results are not independent evidence of a cause.
- The labels rest on the simulated budget.
- Excluding QUAKER gives 19 stores: 16 stay, CINCINNATI - 8035 and FLOWER MOUND - 17627 leave, and 3 others join.
