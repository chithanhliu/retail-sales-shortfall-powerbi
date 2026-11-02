# Power Query M Code

[Back to README](../README.md)

All 8 queries, copied from the final PBIX. They were built in Excel Power Query and imported into Power BI, where 3 columns were added. See [Project Methodology](Project_Methodology.md#c-data-preparation).

**File path:** The queries point to `D:\Small Real Projects\`. Change this to the location of `data/raw/` in your copy of this repository.

| Query | Loaded to model |
|---|---|
| `Dim_Product` | Yes |
| `Dim_Store` | Yes |
| `Fact_Transactions` | Yes |
| `Sales_Weekly` | No, staging only |
| `Growth_Category` | No, staging only |
| `Budget` | Yes |
| `Dim_Category` | Yes |
| `Dim_Date` | Yes |

## Dim_Product

```m
let
    Source = Excel.Workbook(File.Contents("D:\Small Real Projects\dunnhumby - Breakfast at the Frat.xlsx"), null, true),
    #"dh Products Lookup_Sheet" = Source{[Item="dh Products Lookup",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(#"dh Products Lookup_Sheet", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Column1", type any}, {"Column2", type text}, {"Column3", type text}, {"Breakfast at the Frat: A Time Series Analysis", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type any}, {"Column8", type any}, {"Column9", type any}, {"Column10", type any}, {"Column11", type any}, {"Column12", type any}, {"Column13", type any}, {"Column14", type any}, {"Column15", type any}}),
    #"Promoted Headers1" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers1",{{"UPC", Int64.Type}, {"DESCRIPTION", type text}, {"MANUFACTURER", type text}, {"CATEGORY", type text}, {"SUB_CATEGORY", type text}, {"PRODUCT_SIZE", type text}, {"Column7", type any}, {"Column8", type any}, {"Column9", type any}, {"Column10", type any}, {"Column11", type any}, {"Column12", type any}, {"Column13", type any}, {"Column14", type any}, {"Column15", type any}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type1",{"Column7", "Column8", "Column9", "Column10", "Column11", "Column12", "Column13", "Column14", "Column15"}),
    #"Removed Blank Rows" = Table.SelectRows(#"Removed Columns", each not List.IsEmpty(List.RemoveMatchingItems(Record.FieldValues(_), {"", null}))),
    #"Sorted Rows" = Table.Sort(#"Removed Blank Rows",{{"DESCRIPTION", Order.Ascending}, {"PRODUCT_SIZE", Order.Ascending}, {"SUB_CATEGORY", Order.Descending}}),
    #"Filtered Rows" = Table.SelectRows(#"Sorted Rows", each true)
in
    #"Filtered Rows"
```

## Dim_Store

```m
let
    Source = Excel.Workbook(File.Contents("D:\Small Real Projects\dunnhumby - Breakfast at the Frat.xlsx"), null, true),
    #"dh Store Lookup_Sheet" = Source{[Item="dh Store Lookup",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(#"dh Store Lookup_Sheet", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Column1", type any}, {"Column2", type text}, {"Column3", type text}, {"Breakfast at the Frat: A Time Series Analysis", type text}, {"Column5", type any}, {"Column6", type text}, {"Column7", type any}, {"Column8", type any}, {"Column9", type any}, {"Column10", type any}, {"Column11", type any}, {"Column12", type any}, {"Column13", type any}, {"Column14", type any}, {"Column15", type any}}),
    #"Promoted Headers1" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers1",{{"STORE_ID", Int64.Type}, {"STORE_NAME", type text}, {"ADDRESS_CITY_NAME", type text}, {"ADDRESS_STATE_PROV_CODE", type text}, {"MSA_CODE", Int64.Type}, {"SEG_VALUE_NAME", type text}, {"PARKING_SPACE_QTY", Int64.Type}, {"SALES_AREA_SIZE_NUM", Int64.Type}, {"AVG_WEEKLY_BASKETS", type number}, {"Column10", type any}, {"Column11", type any}, {"Column12", type any}, {"Column13", type any}, {"Column14", type any}, {"Column15", type any}}),
    #"Removed Blank Rows" = Table.SelectRows(#"Changed Type1", each not List.IsEmpty(List.RemoveMatchingItems(Record.FieldValues(_), {"", null}))),
    #"Removed Columns" = Table.RemoveColumns(#"Removed Blank Rows",{"Column10", "Column11", "Column12", "Column13", "Column14", "Column15"}),
    #"Sorted Rows" = Table.Sort(#"Removed Columns",{{"STORE_NAME", Order.Ascending}}),
    #"Duplicated Column" = Table.DuplicateColumn(#"Sorted Rows", "STORE_ID", "STORE_ID - Copy"),
    #"Removed Columns1" = Table.RemoveColumns(#"Duplicated Column",{"STORE_ID - Copy"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Removed Columns1",{{"STORE_ID", Int64.Type}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type2", "Phan_Khuc", each if [STORE_ID] = 4503 then "Chưa xác định" else if [STORE_ID] = 17627 then "Chưa xác định" else [SEG_VALUE_NAME]),
    #"Removed Columns2" = Table.RemoveColumns(#"Added Custom",{"PARKING_SPACE_QTY", "SEG_VALUE_NAME"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Columns2", {"STORE_ID"}),
    #"Changed Type3" = Table.TransformColumnTypes(#"Removed Duplicates",{{"STORE_ID", Int64.Type}}),
    #"Inserted Merged Column" = Table.AddColumn(#"Changed Type3", "Merged", each Text.Combine({[ADDRESS_CITY_NAME], Text.From([STORE_ID], "en-AU")}, " - "), type text),
    #"Renamed Columns" = Table.RenameColumns(#"Inserted Merged Column",{{"Merged", "Store_Label"}}),
    #"Added Custom1" = Table.AddColumn(#"Renamed Columns", "Muc_Uu_Tien", each if List.Contains({389, 13837, 25261, 8035, 19265, 4521, 19533, 21479}, [STORE_ID]) then "Cao"
else if List.Contains({2495, 11993, 17627, 2513, 23327, 25233, 6379, 17599, 4489, 8263}, [STORE_ID]) then "Theo dõi"
else "Không"),
    #"Changed Type4" = Table.TransformColumnTypes(#"Added Custom1",{{"Muc_Uu_Tien", type text}})
in
    #"Changed Type4"
```

## Fact_Transactions

```m
let
    Source = Excel.Workbook(File.Contents("D:\Small Real Projects\dunnhumby - Breakfast at the Frat.xlsx"), null, true),
    #"dh Transaction Data_Sheet" = Source{[Item="dh Transaction Data",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(#"dh Transaction Data_Sheet", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Column1", type any}, {"Column2", type any}, {"Column3", type any}, {"Breakfast at the Frat: A Time Series Analysis", type any}, {"Column5", type any}, {"Column6", type any}, {"Column7", type any}, {"Column8", type any}, {"Column9", type any}, {"Column10", type any}, {"Column11", type any}, {"Column12", type any}, {"Column13", type any}, {"Column14", type any}, {"Column15", type any}}),
    #"Promoted Headers1" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Removed Blank Rows" = Table.SelectRows(#"Promoted Headers1", each not List.IsEmpty(List.RemoveMatchingItems(Record.FieldValues(_), {"", null}))),
    #"Changed Type1" = Table.TransformColumnTypes(#"Removed Blank Rows",{{"WEEK_END_DATE", type date}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type1",{"Column13", "Column14", "Column15"}),
    #"Filtered Rows" = Table.SelectRows(#"Removed Columns", each true),
    #"Changed Type2" = Table.TransformColumnTypes(#"Filtered Rows",{{"STORE_NUM", Int64.Type}, {"UPC", Int64.Type}, {"UNITS", Int64.Type}, {"VISITS", Int64.Type}, {"HHS", Int64.Type}, {"SPEND", type number}, {"PRICE", type number}, {"BASE_PRICE", type number}, {"FEATURE", Int64.Type}, {"DISPLAY", Int64.Type}, {"TPR_ONLY", Int64.Type}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type2", "Is_Discount", each if [PRICE] = null or [BASE_PRICE] = null then 0
else if [PRICE] < [BASE_PRICE] then 1 else 0),
    #"Changed Type3" = Table.TransformColumnTypes(#"Added Custom",{{"Is_Discount", Int64.Type}})
in
    #"Changed Type3"
```

## Sales_Weekly

```m
let
    Source = Fact_Transactions,
    #"Merged Queries" = Table.NestedJoin(Source, {"UPC"}, Dim_Product, {"UPC"}, "dh Products Lookup", JoinKind.LeftOuter),
    #"Expanded dh Products Lookup" = Table.ExpandTableColumn(#"Merged Queries", "dh Products Lookup", {"CATEGORY"}, {"CATEGORY"}),
    #"Reordered Columns" = Table.ReorderColumns(#"Expanded dh Products Lookup",{"WEEK_END_DATE", "STORE_NUM", "UPC", "CATEGORY", "UNITS", "VISITS", "HHS", "SPEND", "PRICE", "BASE_PRICE", "FEATURE", "DISPLAY", "TPR_ONLY"}),
    #"Grouped Rows" = Table.Group(#"Reordered Columns", {"CATEGORY", "WEEK_END_DATE", "STORE_NUM"}, {{"Sales", each List.Sum([SPEND]), type number}}),
    #"Reordered Columns1" = Table.ReorderColumns(#"Grouped Rows",{"WEEK_END_DATE", "CATEGORY", "STORE_NUM", "Sales"})
in
    #"Reordered Columns1"
```

## Growth_Category

```m
let
    Source = Sales_Weekly,
    #"Filtered Rows" = Table.SelectRows(Source, each ([STORE_NUM] <> 387)),
    #"Added Custom" = Table.AddColumn(#"Filtered Rows", "Ky", each if [WEEK_END_DATE] >= #date(2009,1,14) and [WEEK_END_DATE] <= #date(2009,12,30) then "Sales_2009" else if [WEEK_END_DATE] >= #date(2010,1,13) and [WEEK_END_DATE] <= #date(2010,12,29) then "Sales_2010" else "Bo"),
    #"Filtered Rows1" = Table.SelectRows(#"Added Custom", each ([Ky] <> "Bo")),
    #"Grouped Rows" = Table.Group(#"Filtered Rows1", {"Ky", "CATEGORY"}, {{"Sales", each List.Sum([Sales]), type number}}),
    #"Pivoted Column" = Table.Pivot(#"Grouped Rows", List.Distinct(#"Grouped Rows"[Ky]), "Ky", "Sales", List.Sum),
    #"Added Custom1" = Table.AddColumn(#"Pivoted Column", "g", each ([Sales_2010] - [Sales_2009])/[Sales_2009]),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom1",{{"g", type number}})
in
    #"Changed Type"
```

## Budget

```m
let
    Source = Sales_Weekly,
    #"Filtered Rows" = Table.SelectRows(Source, each [WEEK_END_DATE] >= #date(2010, 1, 6) and [WEEK_END_DATE] <= #date(2010, 12, 29)),
    #"Renamed Columns" = Table.RenameColumns(#"Filtered Rows",{{"Sales", "Base_2010"}}),
    #"Added Custom" = Table.AddColumn(#"Renamed Columns", "Budget_Week", each Date.AddDays([WEEK_END_DATE],364)),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Budget_Week", type date}}),
    #"Merged Queries" = Table.NestedJoin(#"Changed Type", {"CATEGORY"}, Growth_Category, {"CATEGORY"}, "Growth_Category", JoinKind.LeftOuter),
    #"Expanded Growth_Category" = Table.ExpandTableColumn(#"Merged Queries", "Growth_Category", {"g"}, {"Growth_Category.g"}),
    #"Renamed Columns3" = Table.RenameColumns(#"Expanded Growth_Category",{{"Growth_Category.g", "g"}}),
    #"Added Custom1" = Table.AddColumn(#"Renamed Columns3", "Budget", each [Base_2010]* (1+[g])),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom1",{{"Budget", type number}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type1",{"WEEK_END_DATE"}),
    #"Renamed Columns1" = Table.RenameColumns(#"Removed Columns",{{"Budget_Week", "WEEK_END_DATE"}}),
    #"Reordered Columns" = Table.ReorderColumns(#"Renamed Columns1",{"WEEK_END_DATE", "CATEGORY", "STORE_NUM", "Base_2010", "g", "Budget"})
in
    #"Reordered Columns"
```

## Dim_Category

```m
let
    Source = Dim_Product,
    #"Removed Columns" = Table.RemoveColumns(Source,{"UPC", "DESCRIPTION", "MANUFACTURER", "SUB_CATEGORY", "PRODUCT_SIZE"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Columns")
in
    #"Removed Duplicates"
```

## Dim_Date

```m
let
    Source = Sales_Weekly,
    #"Removed Columns" = Table.RemoveColumns(Source,{"CATEGORY", "STORE_NUM", "Sales"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Columns"),
    #"Inserted Year" = Table.AddColumn(#"Removed Duplicates", "Year", each Date.Year([WEEK_END_DATE]), Int64.Type),
    #"Inserted Month" = Table.AddColumn(#"Inserted Year", "Month", each Date.Month([WEEK_END_DATE]), Int64.Type),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Month", "Month Name", each Date.MonthName([WEEK_END_DATE]), type text),
    #"Added Custom" = Table.AddColumn(#"Inserted Month Name", "Thang_So_Sanh", each if [Year] = 2010 then Date.Month(Date.AddDays([WEEK_END_DATE], 364)) else [Month]),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Thang_So_Sanh", Int64.Type}})
in
    #"Changed Type"
```
