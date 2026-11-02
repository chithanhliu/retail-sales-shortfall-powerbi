# Project Methodology

[Back to README](../README.md)

This document follows the project from business question to recommendations. Check results are labelled **PASS** only where the check was actually run.

## A. Business understanding

**Scenario:** A regional manager reviews 2011 sales for a retail store group and needs to decide where to look first. The scenario is constructed for this portfolio project; there was no real client.

**Stakeholders:** Regional manager (primary), with store managers, buying and marketing as owners of follow-up actions.

**Official business question** (translated from the dashboard):

> In 2011, which stores and product categories created the sales shortfall against budget, and which actions should the regional manager prioritise?

**Decisions supported:**

- Which categories, states and stores to review first.
- Which issues to pass to buying, stores or marketing.
- Whether a wide promotion increase is justified yet.

## B. Raw data understanding

| Item | Detail |
|---|---|
| Dataset | [dunnhumby, Breakfast at the Frat](https://www.dunnhumby.com/source-files/) |
| File | `dunnhumby - Breakfast at the Frat.xlsx` |
| Sheets | `Glossary`, `dh Store Lookup`, `dh Products Lookup`, `dh Transaction Data` |
| Grain | 1 row = 1 product (UPC) × 1 store × 1 week, only where a sale exists |
| Rows | 524,950 transactions; 79 store rows; 58 product rows |
| Period | 156 weeks ending 14 January 2009 to 4 January 2012, ending on Wednesdays |
| Coverage | 77 stores in TX (41), OH (31), KY (4), IN (1). 4 categories. 55 of 58 products have sales |
| Totals | Spend $27,927,722.58; Units 10,293,354 |

**Key fields**

| Field | Meaning (glossary) | Note |
|---|---|---|
| `SPEND` | Dollar sales | Can be added across any dimension |
| `UNITS` | Units sold | Added across categories it is only a count of unlike items |
| `VISITS` | Baskets containing the product | **Not additive** across products or weeks. Not used in measures |
| `HHS` | Purchasing households | **Not additive** across products, weeks or stores. Not used in measures |
| `PRICE` | Amount charged at shelf | Not additive |
| `BASE_PRICE` | Base price | Not additive |
| `FEATURE` | In the in-store circular | 0/1 |
| `DISPLAY` | On in-store promotional display | 0/1 |
| `TPR_ONLY` | Temporary price reduction only | 0/1. Not used in a measure |

**Quality observations in the raw file**

| Observation | Count | Decision |
|---|---:|---|
| `BASE_PRICE` blank | 185 rows | Keep. Counted as not discounted |
| `PRICE` blank | 23 rows | Keep. 208 rows have a blank price field, with $7,948.81 of sales |
| `UNITS` ≤ 0 | 5 rows | Keep. Cause unknown |
| `SPEND` ≤ 0 | 24 rows | Keep. Cause unknown |
| `PRICE` ≤ 0 | 1 row | Keep. Cause unknown |
| `VISITS` > `UNITS` | 2,309 rows (0.44%) | Keep. Cause unknown; these fields are not used in measures |
| `PRICE` > `BASE_PRICE` | 6,047 rows | Keep. Counted as not discounted |
| `PARKING_SPACE_QTY` blank | 52 store rows | Remove the column, keep the stores |
| Duplicate store IDs | 4503, 17627 | Rows differ only in `SEG_VALUE_NAME` |
| Duplicate transaction keys | 0 | None |

## C. Data preparation

The data was prepared in Excel Power Query first, then imported into Power BI.

### Step 1: Excel Power Query (`Breakfast_at_the_Frat_Cleaned.xlsx`)

The cleaning, growth calculation, budget and dimension queries were built here:

- Promote headers from the second row, remove blank rows and empty columns, and set data types.
- **Stores:** Add `Phan_Khuc`, setting the two duplicate store IDs to `Chưa xác định` ("undetermined") because the file does not show which segment is right. Remove `PARKING_SPACE_QTY` and `SEG_VALUE_NAME`, remove duplicates on `STORE_ID`, and add `Store_Label` (city and ID), because 2 store names repeat.
- **`Sales_Weekly`:** Join transactions to category and group by week, category and store.
- **`Growth_Category`:** Exclude store 387 and calculate category growth.
- **`Budget`:** Calculate the simulated weekly budget.
- **`Dim_Category`, `Dim_Date`:** Build the dimension tables.

### Step 2: Import into Power BI (`Project3_Final_Dashboard.pbix`)

The workbook's queries were brought in with **File → Import → Power Query, Power Pivot, Power View**. This import copies the query steps, so they keep running inside the PBIX, and their source remains the raw workbook. The cleaned workbook does not need to be present to refresh the PBIX.

Query comparison between the two files:

| PBIX query | Excel query | Comparison |
|---|---|---|
| `Dim_Product` | `dh Products Lookup` | Same steps |
| `Sales_Weekly` | `Sales_Weekly` | Same steps |
| `Growth_Category` | `Growth_Category` | Same steps |
| `Budget` | `Budget` | Same steps |
| `Dim_Category` | `Dim_Category` | Same steps |
| `Fact_Transactions` | `dh Transaction Data` | Same steps, plus `Is_Discount` |
| `Dim_Store` | `Dim_Store` | Same steps, plus `Muc_Uu_Tien` |
| `Dim_Date` | `Dim_Date` | Same steps, plus `Thang_So_Sanh` |

### Step 3: Additions in Power BI

- Rename tables to `Fact_Transactions` and `Dim_Product`.
- Add `Is_Discount` (1 when `PRICE` < `BASE_PRICE`; 0 otherwise or when a price is blank).
- Add `Thang_So_Sanh`, which aligns 2010 weeks to their 2011 month using the 364-day shift.
- Add `Muc_Uu_Tien`, a hard-coded priority label.

### Reconciliation

| Check | Raw | Cleaned Excel | Power BI model | Status |
|---|---:|---:|---:|---|
| Transaction rows | 524,950 | 524,950 | 524,950 | PASS |
| Spend | 27,927,722.58 | 27,927,722.58 | 27,927,722.58 | PASS |
| Units | 10,293,354 | 10,293,354 | 10,293,354 | PASS |
| Stores | 77 unique | 77 | 77 | PASS |
| Products | 58 (55 sold) | 58 | 58 (55 sold) | PASS |
| Weeks | 156 | 156 | 156 | PASS |
| Transaction keys match raw | n/a | 524,950 of 524,950 | Not compared row by row | PASS for Excel; model checked by totals |
| Duplicate business keys | 0 | n/a | n/a | PASS |
| Transactions without store, product or date | 0 | n/a | 0 | PASS |
| Category after join | 0 blank | n/a | n/a | PASS |
| `Sales_Weekly` total | n/a | Not on a sheet | Not loaded | PASS when rebuilt: 47,947 rows, same spend |
| Refresh | n/a | n/a | Completed on a temporary copy | PASS |

`VISITS` and `HHS` matched in total between raw and model, as a technical check only. Their totals have no business meaning.

`Sales_Weekly` could have 48,048 rows (156 × 77 × 4). It has 47,947, because store 387 lacks 25 weeks of 2009 in all 4 categories, and store 8035 lacks 1 week of 2011 in ORAL HYGIENE PRODUCTS. The data does not explain why. Missing combinations are not filled or estimated; see [how they are treated](Budget_Methodology.md#how-missing-combinations-are-treated).

## D. Budget construction

See [Budget Methodology](Budget_Methodology.md).

In short: `Budget = 2010 sales for the same week and store × (1 + category growth)`. It is simulated, at week × store × category grain, and totals $10,416,222.58 for 2011.

## E. Data modeling

```text
Dim_Category ──< Dim_Product ──< Fact_Transactions >── Dim_Store
      │                                  │                │
      │                                  └──────┐         │
      │                                      Dim_Date     │
      │                                         │         │
      └──────────────────────────< Budget >─────┴─────────┘
```

| Table | Type | Grain | Rows |
|---|---|---|---:|
| `Fact_Transactions` | Fact | Week × store × UPC | 524,950 |
| `Budget` | Fact (simulated) | Week × store × category, 2011 | 16,016 |
| `Dim_Date` | Dimension | Week | 156 |
| `Dim_Store` | Dimension | Store | 77 |
| `Dim_Product` | Dimension | UPC | 58 |
| `Dim_Category` | Dimension | Category | 4 |

- All relationships are active, many-to-one, and filter one way from dimension to fact.
- The two fact tables have different grains, so they connect only through shared dimensions.
- `Dim_Date` holds weeks and is not marked as a date table. Auto date/time is on.
- `Dim_Product` does not filter `Budget`. Budget measures are not meaningful by manufacturer or product, so the Product page compares 2010 with 2011 instead.

## F. DAX and KPIs

See [DAX Measures](DAX_Measures.md) for all 20 measures.

| Measure | Definition | Question it answers |
|---|---|---|
| `Variance` | Sales − Budget | How large is the gap? |
| `Variance %` | Variance / Budget | How large relative to target? |
| `YoY %` | (Sales − 2010 base) / 2010 base | Did sales actually fall? |
| `Gap vs System` | Store `Variance %` − 77-store `Variance %` | Is the store weaker than the group? |
| `Sales Change` | 2011 − 2010 sales | Which manufacturers moved most? |
| `Units Change %`, `Price Change %` | 2011 / 2010 − 1 | Volume or average price? |
| `Feature Rate`, `Display Rate`, `Discount Rate` | Share of sold rows with the flag | Did promotion activity change? |

**Filter context to remember:**

- Budget measures need Year = 2011.
- `Gap vs System` uses `ALL(Dim_Store)`, so it keeps the 77-store benchmark even when a visual shows selected stores.
- The promotion rates are shares of rows with sales, not shares of revenue.

## G. Dashboard analysis

The dashboard text is in Vietnamese. English page names below are translations.

| # | PBIX page | English | Question | Filter |
|---|---|---|---|---|
| 1 | Tóm tắt | Summary | Main business question | Year = 2011 |
| 2 | Tổng Quan | Overview | Which period and category hold the shortfall? | Year = 2011 |
| 3 | Sản phẩm | Product | Which manufacturers moved, and was it volume? | Years 2010, 2011 |
| 4 | Khu Vực | Region | Which states to prioritise? | Year = 2011 |
| 5 | Cửa hàng | Store | Which stores to review first? | Year = 2011 |
| 6 | Khuyến mãi | Promotion | What changed in promotion activity? | Years 2010, 2011 |
| 7 | Phụ lục: Tra cứu | Store Lookup | Store file and action plan | Year = 2011 |

Each page follows Key Observations, Business Interpretation and Next Analytical Step; the last page ends with a conclusion and action plan.

### 1. Summary

- **Key visuals:** 5 KPI cards; gap by category and state; ranked priority stores.
- **Validated observations:** Sales are $912,513 below the simulated budget (−8.76%), but only 1.36% below 2010. COLD CEREAL (−$692,690) and TX (−$464,054) are the largest gaps.
- **Interpretation:** Focus review effort on COLD CEREAL. This prioritises an investigation; it does not promise to recover the gap.
- **Next step:** Separate real decline from missing the target.

### 2. Overview

- **Key visuals:** KPI cards; monthly sales and budget; gap by category; evidence table.
- **Validated observations:** COLD CEREAL and BAG SNACKS miss budget and fall against 2010. FROZEN PIZZA grows 4.34% but is 4.45% under budget. ORAL HYGIENE PRODUCTS is 12.85% over budget.
- **Interpretation:** Prioritise volume recovery checks for the first two. For FROZEN PIZZA, question the growth assumption first.
- **Next step:** Separate manufacturer and volume effects.

### 3. Product

- **Key visuals:** QUAKER weekly sales; volume and average price change; manufacturer change.
- **Validated observations:** QUAKER fell $379,852 (−51.31%), with weekly sales close to zero in late 2011. POST FOODS fell $151,002 and TONYS $121,732. COLD CEREAL volume fell about 9.4% while average price rose about 6.3%.
- **Interpretation:** Buying should first confirm whether QUAKER items are still sold, supply changed, or data is missing. The data does not show the cause.
- **Next step:** See where the gap falls by state.

### 4. Region

- **Key visuals:** Gap by state; priority mix; state × category matrix.
- **Validated observations:** TX has the largest gap ($464,054, −10.98%). KY has the highest rate (−17.11%) with only 4 stores. OH misses budget by $315,046 but grows 1.66%. COLD CEREAL is the largest negative in all 4 states.
- **Interpretation:** TX by size, KY for depth. For OH, review expectations too. KY and IN samples are small.
- **Next step:** Move to specific stores.

### 5. Store

- **Key visuals:** Monthly gap vs system for High stores; priority list; store × category.
- **Validated observations:** ERLANGER (−$53,178, −21.24%) and COVINGTON (−$40,512, −18.92%) lead, followed by CROWLEY (−$28,593) and DENTON (−$24,547). The 18 priority stores total −$361,577.
- **Interpretation:** Review these stores first for range, stock, delivery and shelf condition. The label is a project screen, not a staff assessment.
- **Next step:** Check promotion activity without using it to replace availability checks.

### 6. Promotion

- **Key visuals:** Feature, display and discount rates by month; evidence matrix.
- **Validated observations:** In January, feature fell from 12.70% to 6.80% and display from 15.68% to 11.78%. In October, feature fell from 9.73% to 4.99% and display from 10.77% to 6.92%. Other months rose, for example February.
- **Interpretation:** Marketing should check whether these drops were planned. The page does not show that promotions caused sales changes.
- **Next step:** Build store files. A promotion test comes later.

### 7. Store Lookup

- **Key visuals:** Store slicer; KPI cards; monthly chart; category chart and table.
- **Use:** With a store selected, negative `% budget` and negative `YoY` call for operational checks. Negative `% budget` with positive `YoY` calls for reviewing the target first.
- **Final recommendation:** Approve a targeted review, not a wide promotion increase. The week-by-week plan on the page is a proposal, not a result from 2011 data.

## H. Findings and recommendations

| Validated finding | Recommendation | Owner |
|---|---|---|
| QUAKER fell $379,852, with weekly sales close to zero in late 2011 | Confirm listing, supply and data completeness. Review the target only if a range change is confirmed | Buying |
| COLD CEREAL has the largest gap (−$692,690) and is the largest negative in all 4 states | Start reviews with this category | Regional manager |
| TX has the largest dollar gap; KY has the highest rate | Assign TX review by size and KY by depth | Regional manager |
| ERLANGER, COVINGTON, CROWLEY and DENTON have the largest priority-store gaps | Review range, stock, delivery and shelf condition first | Store managers |
| FROZEN PIZZA and OH grow against 2010 but miss budget | Review the growth expectation before planning recovery actions | Planning |
| Feature and display rates fell in January and October, with no cost or margin data | Check plan against execution. Do not approve wide discounts. Test later with a comparison group | Marketing |

These are proposed actions. They were not implemented, and no business result was measured.

## I. Assumptions and limitations

- **Historical data:** 2009 to early 2012. Conclusions do not describe current conditions.
- **Simulated budget:** A shortfall is against a project assumption, not a real target.
- **Sample:** The glossary describes a sampling of stores and selected products. "System" means these 77 stores, not a whole chain.
- **Scenario:** The stakeholder and business context are constructed.
- **Weekly data:** There is no daily or customer-level detail. Weeks are assigned to a year or month by their Wednesday end date.
- **Rows only where sold:** Out-of-stock and no-demand cannot be told apart.
- **No costs, margin, inventory, competitors or staffing:** Causes and profitability cannot be assessed.
- **Promotion analysis:** Rates are descriptive. Formats overlap with discounts, and no causal effect is claimed.
- **Average price:** Affected by product mix.
- **Priority labels:** Hard-coded, chosen after seeing the data, and sensitive to QUAKER.
- **Small samples:** KY has 4 stores and IN has 1.
- **Static text:** Commentary does not change with slicers; figures quoted there describe the unfiltered view.
