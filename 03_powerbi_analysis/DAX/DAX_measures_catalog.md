<h1 align = "center"> 
Lotus Group Retail Analytics: DAX Measures Log
</h1>

# Objective

The goal here is not to create dozens of measures, but to create a compact set that answers the main business questions and can be reused across all report pages.

# Create a dedicated Measures table

This keeps the model organized.

In Power BI → Modeling → New Table, enter:

    _Measures = DATATABLE(
        "Measure Group", STRING,
        {
            {"Sales"},
            {"Orders"},
            {"Customers"},
            {"Products"},
            {"Returns"},
            {"Performance"}
        }
    )

We won't necessarily create all measures at once. 
 
 - A good rule of thumb is to create a handful and validate before proceeding with the next handful

# Core Sales measures

These will be the most important measures in the project.

## Total Sales

We'll use line_total_revenue from fact_order_details as our primary sales measure.

    Total Sales =
    SUM(fact_order_details[line_total_revenue])

This is important because we established earlier that the order-detail table is the source of truth for product-level revenue.

## Total Cost
    Total Cost =
    SUM(fact_order_details[line_total_cost])

## Gross Profit
    Gross Profit =
    [Total Sales] - [Total Cost]

## Gross Margin %
    Gross Margin % =
    DIVIDE(
        [Gross Profit],
        [Total Sales]
    )

We'll format this as Percentage.

# Core Order measures

## Total Orders

Since `fact_orders` is at order grain:

    Total Orders =
    DISTINCTCOUNT(fact_orders[order_id])

This is preferable to COUNTROWS() because it explicitly communicates that we are counting unique orders.

> **Update:** `Total Orders` was later changed so it also responds to the Category slicer. See *Changes to original measures* below.

## Units Sold
    Units Sold =
    SUM(fact_order_details[quantity])

## Average Order Value
    Average Order Value =
    DIVIDE(
        [Total Sales],
        [Total Orders]
    )

We'll use this as one of the key executive KPIs.

## Units per Order
    Units per Order =
    DIVIDE(
        [Units Sold],
        [Total Orders]
    )

This should help distinguish between:

- getting more orders
- selling more products per order.

# Core Customer measures

## Total Customers
    Total Customers =
    DISTINCTCOUNT(fact_orders[customer_id])

This counts customers who actually appear in the order data.

## Average Sales per Customer
    Sales per Customer =
    DIVIDE(
        [Total Sales],
        [Total Customers]
    )

We shuldn't interpret this as formal customer lifetime value. 
 - It's simply average sales attributable to each customer within the selected filter context.

# Core Product measures

We can now leverage the relationship:

    dim_products → fact_order_details

## Products Sold
    Products Sold =
    DISTINCTCOUNT(fact_order_details[product_id])

This should tell us how many distinct products generated sales in the current context.
 - We can also call this Active Products Sold to make the interpretation clearer.

## Average Selling Price
    Average Selling Price =
    DIVIDE(
        [Total Sales],
        [Units Sold]
    )

This is more useful than simply averaging the selling_price column because it is weighted by units sold.

# Core Returns measures

Now we use `fact_returns`.

## Returned Amount
    Returned Amount =
    SUM(fact_returns[return_amount])

## Number of Returns
    Number of Returns =
    DISTINCTCOUNT(fact_returns[return_id])

## Return Rate

I'd recommend defining return rate based on returned orders relative to total orders, rather than dividing returned monetary value by sales.

    Return Rate % =
    DIVIDE(
        [Number of Returns],
        [Total Orders]
    )

We'll format as Percentage.

> **Update:** the report pages now use **`Return Rate (Value) %`** (returned value ÷ sales) instead, because it shows how much revenue returns cost. `Return Rate %` is kept in the model for reference. See the redesign section below.

This gives management an intuitive measure:
 - What proportion of orders resulted in a recorded return?

# Time-performance measures

Now we can take advantage of `dim_date`.

## Sales YTD
    Sales YTD =
    TOTALYTD(
        [Total Sales],
        dim_date[full_date]
    )

## Sales Previous Year and YoY Sales Growth (replaced)

These two measures were first written with `SAMEPERIODLASTYEAR`. That function needs a marked date table, and in this model (where `dim_date` joins on `date_id`) it returns blank once a Year slicer is applied. They were removed and replaced by **`Sales PY`** and **`Sales vs PY %`**, which shift the selected year back by one explicitly. See *Prior Year* and *Core* in the redesign section below.

# Ramadan analysis

Since the dataset specifically contains `is_ramadan`, let's create one simple measure.

## Ramadan Sales
    Ramadan Sales =
    CALCULATE(
        [Total Sales],
        dim_date[is_ramadan] = 1
    )

And:

    Non-Ramadan Sales =
    CALCULATE(
        [Total Sales],
        dim_date[is_ramadan] = 0
    )

We can later use these to investigate whether sales patterns differ during Ramadan.

# Weekend sales

Our date table also contains the weekend indicator. So we can create

    Weekend Sales =
    CALCULATE(
        [Total Sales],
        dim_date[is_weekend] = 1
    )

And:

    Weekday Sales =
    CALCULATE(
        [Total Sales],
        dim_date[is_weekend] = 0
    )

This gives us another straightforward behavioral analysis without creating unnecessary complexity.

# Employee attribution

With 632 orer records missing employee IDs, I'll create one measure specifically to quantify that issue.

    Orders with Employee ID =
    CALCULATE(
        [Total Orders],
        NOT ISBLANK(fact_orders[employee_id])
    )

And:

    Orders without Employee ID =
    CALCULATE(
        [Total Orders],
        ISBLANK(fact_orders[employee_id])
    )

Then employee attribution rate becomes:

    Employee Attribution Rate % =
    DIVIDE(
        [Orders with Employee ID],
        [Total Orders]
    )

This isn't necessarily a dashboard KPI for executives. But it's useful as a data-quality/business-process metric, particularly for the portfolio because it demonstrates that we didn't simply ignore the missing employee attribution.

---

# Dashboard redesign measures (November 2026)

The report was rebuilt around insight-led titles, KPI comparisons and a single accent colour. The measures below were added for that. The model changes made at the same time are described in [`data_model.md`](../../02_documentation/data_model.md):

- returns now link to orders, so they respond to every slicer;
- three helper tables were added: `CustomerDecile`, `Share Metric` and `RefreshInfo`;
- `Period Type`, `short_month` and the store-name clean-up were added to the dimensions;
- auto date/time was switched off.

**186 measures added**, grouped by display folder: Core (28), Prior Year (20), Labels (3), KPI Tiles (72), Colours (15), Titles (45), Other (3).

## Changes to original measures

These measures now turn on two-way filtering between order lines and orders inside the measure (`CROSSFILTER`), so order, customer and return counts respond to the Category slicer. The relationship itself stays single-direction.

### Total Orders

```DAX
Total Orders =
CALCULATE ( DISTINCTCOUNT ( fact_orders[order_id] ), CROSSFILTER ( fact_order_details[order_id], fact_orders[order_id], BOTH ) )
```

### Total Customers

```DAX
Total Customers =
CALCULATE ( DISTINCTCOUNT ( fact_orders[customer_id] ), CROSSFILTER ( fact_order_details[order_id], fact_orders[order_id], BOTH ) )
```

### Returned Amount

```DAX
Returned Amount =
CALCULATE ( SUM ( fact_returns[return_amount] ), CROSSFILTER ( fact_order_details[order_id], fact_orders[order_id], BOTH ) )
```

### Number of Returns

```DAX
Number of Returns =
CALCULATE ( DISTINCTCOUNT ( fact_returns[return_id] ), CROSSFILTER ( fact_order_details[order_id], fact_orders[order_id], BOTH ) )
```

### Total Orders Returned

```DAX
Total Orders Returned =
CALCULATE ( DISTINCTCOUNT ( fact_returns[order_id] ), CROSSFILTER ( fact_order_details[order_id], fact_orders[order_id], BOTH ) )
```

## Core

Business measures added for the redesigned pages.

### Sales Change vs PY

```DAX
Sales Change vs PY =
IF ( NOT ISBLANK ( [Sales PY] ), [Total Sales] - [Sales PY] )
```

### Sales vs PY %

```DAX
Sales vs PY % =
DIVIDE ( [Total Sales] - [Sales PY], [Sales PY] )
```

### Orders per Customer

```DAX
Orders per Customer =
DIVIDE ( [Total Orders], [Total Customers] )
```

### Return Rate (Value) %

```DAX
Return Rate (Value) % =
DIVIDE ( [Returned Amount], [Total Sales] )
```

### Preventable Returned Amount

```DAX
Preventable Returned Amount =
CALCULATE ( [Returned Amount], fact_returns[return_reason] IN { "Duplicate Order", "Wrong Item Delivered" } )
```

### Preventable Share %

```DAX
Preventable Share % =
DIVIDE ( [Preventable Returned Amount], [Returned Amount] )
```

### Active Employees

```DAX
Active Employees =
CALCULATE ( DISTINCTCOUNT ( fact_orders[employee_id] ), fact_orders[employee_id] <> "", CROSSFILTER ( fact_order_details[order_id], fact_orders[order_id], BOTH ) )
```

### Attributed Sales

```DAX
Attributed Sales =
CALCULATE ( [Total Sales], fact_orders[employee_id] <> "" )
```

### Sales per Employee

```DAX
Sales per Employee =
DIVIDE ( [Attributed Sales], [Active Employees] )
```

### Orders per Employee

```DAX
Orders per Employee =
DIVIDE ( [Orders with Employee ID], [Active Employees] )
```

### Store Floor Space

```DAX
Store Floor Space =
SUM ( dim_stores[size_sqm] )
```

### Sales per m²

```DAX
Sales per m² =
DIVIDE ( [Total Sales], [Store Floor Space] )
```

### Store Count

```DAX
Store Count =
COUNTROWS ( FILTER ( VALUES ( dim_stores[store_name] ), [Total Sales] > 0 ) )
```

### Stores Growing

```DAX
Stores Growing =
COUNTROWS ( FILTER ( VALUES ( dim_stores[store_name] ), [Sales vs PY %] > 0 ) )
```

### Stores Shrinking

```DAX
Stores Shrinking =
COUNTROWS ( FILTER ( VALUES ( dim_stores[store_name] ), [Sales vs PY %] < 0 ) )
```

### Sales Share %

```DAX
Sales Share % =
DIVIDE ( [Total Sales], CALCULATE ( [Total Sales], ALLSELECTED ( dim_products[category] ) ) )
```

### Gross Profit Share %

```DAX
Gross Profit Share % =
DIVIDE ( [Gross Profit], CALCULATE ( [Gross Profit], ALLSELECTED ( dim_products[category] ) ) )
```

### Avg Daily Sales

```DAX
Avg Daily Sales =
DIVIDE ( [Total Sales], COUNTROWS ( dim_date ) )
```

### Ramadan Daily Sales

```DAX
Ramadan Daily Sales =
CALCULATE ( [Avg Daily Sales], dim_date[is_ramadan] = 1 )
```

### Non-Ramadan Daily Sales

```DAX
Non-Ramadan Daily Sales =
CALCULATE ( [Avg Daily Sales], dim_date[is_ramadan] = 0 )
```

### Weekend Daily Sales

```DAX
Weekend Daily Sales =
CALCULATE ( [Avg Daily Sales], dim_date[is_weekend] = 1 )
```

### Weekday Daily Sales

```DAX
Weekday Daily Sales =
CALCULATE ( [Avg Daily Sales], dim_date[is_weekend] = 0 )
```

### Ramadan Days

```DAX
Ramadan Days =
CALCULATE ( COUNTROWS ( dim_date ), dim_date[is_ramadan] = 1 )
```

### Ramadan Seasons

```DAX
Ramadan Seasons =
CALCULATE ( DISTINCTCOUNT ( dim_date[year] ), dim_date[is_ramadan] = 1 )
```

### Avg Monthly Sales

```DAX
Avg Monthly Sales =
DIVIDE ( [Total Sales], DISTINCTCOUNT ( dim_date[year] ) )
```

### Cumulative Revenue Share

```DAX
Cumulative Revenue Share =
VAR d = SELECTEDVALUE ( CustomerDecile[Decile] )
VAR Cust =
    FILTER (
        ADDCOLUMNS ( VALUES ( fact_orders[customer_id] ), "@s", [Total Sales] ),
        [@s] > 0
    )
VAR n = COUNTROWS ( Cust )
VAR k = ROUNDUP ( n * d / 10, 0 )
RETURN
    IF ( NOT ISBLANK ( d ) && n > 0, DIVIDE ( SUMX ( TOPN ( k, Cust, [@s], DESC ), [@s] ), SUMX ( Cust, [@s] ) ) )
```

### Top 20% Revenue Share

```DAX
Top 20% Revenue Share =
VAR Cust =
    FILTER (
        ADDCOLUMNS ( VALUES ( fact_orders[customer_id] ), "@s", [Total Sales] ),
        [@s] > 0
    )
VAR k = INT ( COUNTROWS ( Cust ) * 0.2 )
RETURN
    IF ( k > 0, DIVIDE ( SUMX ( TOPN ( k, Cust, [@s], DESC ), [@s] ), SUMX ( Cust, [@s] ) ) )
```

### Employee Store

```DAX
Employee Store =
VAR s = SELECTEDVALUE ( dim_employees[store_id] )
RETURN
    SUBSTITUTE ( LOOKUPVALUE ( dim_stores[store_name], dim_stores[store_id], s ), "Lotus ", "" )
```

## Prior Year

The same measure for the year before the selected year: `CALCULATE ( [measure], dim_date[year] = SELECTEDVALUE ( dim_date[year] ) - 1 )`. Blank when no single year is selected, so comparisons switch off cleanly.

### Sales PY

```DAX
Sales PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Total Sales], dim_date[year] = y - 1 ) )
```

### Gross Profit PY

```DAX
Gross Profit PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Gross Profit], dim_date[year] = y - 1 ) )
```

### Gross Margin % PY

```DAX
Gross Margin % PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Gross Margin %], dim_date[year] = y - 1 ) )
```

### Orders PY

```DAX
Orders PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Total Orders], dim_date[year] = y - 1 ) )
```

### Units Sold PY

```DAX
Units Sold PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Units Sold], dim_date[year] = y - 1 ) )
```

### Average Order Value PY

```DAX
Average Order Value PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Average Order Value], dim_date[year] = y - 1 ) )
```

### Customers PY

```DAX
Customers PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Total Customers], dim_date[year] = y - 1 ) )
```

### Sales per Customer PY

```DAX
Sales per Customer PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Sales per Customer], dim_date[year] = y - 1 ) )
```

### Orders per Customer PY

```DAX
Orders per Customer PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Orders per Customer], dim_date[year] = y - 1 ) )
```

### Return Rate (Value) % PY

```DAX
Return Rate (Value) % PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Return Rate (Value) %], dim_date[year] = y - 1 ) )
```

### Returned Amount PY

```DAX
Returned Amount PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Returned Amount], dim_date[year] = y - 1 ) )
```

### Number of Returns PY

```DAX
Number of Returns PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Number of Returns], dim_date[year] = y - 1 ) )
```

### Preventable Share % PY

```DAX
Preventable Share % PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Preventable Share %], dim_date[year] = y - 1 ) )
```

### Attribution Rate % PY

```DAX
Attribution Rate % PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Employee Attribution Rate %], dim_date[year] = y - 1 ) )
```

### Orders without Employee ID PY

```DAX
Orders without Employee ID PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Orders without Employee ID], dim_date[year] = y - 1 ) )
```

### Sales per Employee PY

```DAX
Sales per Employee PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Sales per Employee], dim_date[year] = y - 1 ) )
```

### Orders per Employee PY

```DAX
Orders per Employee PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Orders per Employee], dim_date[year] = y - 1 ) )
```

### Sales per m² PY

```DAX
Sales per m² PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Sales per m²], dim_date[year] = y - 1 ) )
```

### Top 20% Revenue Share PY

```DAX
Top 20% Revenue Share PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Top 20% Revenue Share], dim_date[year] = y - 1 ) )
```

### Avg Daily Sales PY

```DAX
Avg Daily Sales PY =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF ( NOT ISBLANK ( y ), CALCULATE ( [Avg Daily Sales], dim_date[year] = y - 1 ) )
```

## Labels

Text used in subtitles, data labels and the footer.

### PY Label

```DAX
PY Label =
"vs " & FORMAT ( SELECTEDVALUE ( dim_date[year] ) - 1, "0" )
```

### Context Label

```DAX
Context Label =
VAR y =
    IF (
        HASONEVALUE ( dim_date[year] ),
        FORMAT ( VALUES ( dim_date[year] ), "0" ),
        IF ( ISFILTERED ( dim_date[year] ), "Selected years", "2022–2024" )
    )
VAR r =
    IF (
        ISFILTERED ( dim_stores[region] ),
        CONCATENATEX ( VALUES ( dim_stores[region] ), dim_stores[region], ", " ),
        "All regions"
    )
VAR c =
    IF (
        ISFILTERED ( dim_products[category] ),
        CONCATENATEX ( VALUES ( dim_products[category] ), dim_products[category], ", " ),
        "All categories"
    )
RETURN
    y & " · " & r & " · " & c
```

### Last Refreshed Text

```DAX
Last Refreshed Text =
"Last refreshed " & FORMAT ( MAX ( RefreshInfo[Refreshed At] ), "d mmm yyyy" )
```

## KPI Tiles

For each KPI card: the formatted value (`KPI x`), the change text (`KPI x Δ`, ▲/▼ vs prior year or another baseline) and the good/bad colour (`KPI x Colour`).

### KPI Sales

```DAX
KPI Sales =
VAR v = [Total Sales]
RETURN
    IF ( ISBLANK ( v ), "–", IF ( ABS ( v ) >= 1000000, "EGP " & FORMAT ( v / 1000000, "#,0.00" ) & "M", IF ( ABS ( v ) >= 10000, "EGP " & FORMAT ( v / 1000, "#,0" ) & "K", "EGP " & FORMAT ( v, "#,0" ) ) ) )
```

### KPI Sales Δ

```DAX
KPI Sales Δ =
VAR cur = [Total Sales]
VAR b = [Sales PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Sales Colour

```DAX
KPI Sales Colour =
VAR cur = [Total Sales]
VAR b = [Sales PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Gross Profit

```DAX
KPI Gross Profit =
VAR v = [Gross Profit]
RETURN
    IF ( ISBLANK ( v ), "–", IF ( ABS ( v ) >= 1000000, "EGP " & FORMAT ( v / 1000000, "#,0.00" ) & "M", IF ( ABS ( v ) >= 10000, "EGP " & FORMAT ( v / 1000, "#,0" ) & "K", "EGP " & FORMAT ( v, "#,0" ) ) ) )
```

### KPI Gross Profit Δ

```DAX
KPI Gross Profit Δ =
VAR cur = [Gross Profit]
VAR b = [Gross Profit PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Gross Profit Colour

```DAX
KPI Gross Profit Colour =
VAR cur = [Gross Profit]
VAR b = [Gross Profit PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Gross Margin

```DAX
KPI Gross Margin =
VAR v = [Gross Margin %]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0.0%" ) )
```

### KPI Gross Margin Δ

```DAX
KPI Gross Margin Δ =
VAR cur = [Gross Margin %]
VAR b = [Gross Margin % PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( cur - b ) * 100, "0.0" ) & " pts" & " " & [PY Label]
    )
```

### KPI Gross Margin Colour

```DAX
KPI Gross Margin Colour =
VAR cur = [Gross Margin %]
VAR b = [Gross Margin % PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Orders

```DAX
KPI Orders =
VAR v = [Total Orders]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "#,0" ) )
```

### KPI Orders Δ

```DAX
KPI Orders Δ =
VAR cur = [Total Orders]
VAR b = [Orders PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Orders Colour

```DAX
KPI Orders Colour =
VAR cur = [Total Orders]
VAR b = [Orders PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Return Rate

```DAX
KPI Return Rate =
VAR v = [Return Rate (Value) %]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0.0%" ) )
```

### KPI Return Rate Δ

```DAX
KPI Return Rate Δ =
VAR cur = [Return Rate (Value) %]
VAR b = [Return Rate (Value) % PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( cur - b ) * 100, "0.0" ) & " pts" & " " & [PY Label]
    )
```

### KPI Return Rate Colour

```DAX
KPI Return Rate Colour =
VAR cur = [Return Rate (Value) %]
VAR b = [Return Rate (Value) % PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#B42318", "#1A7F4B" ) )
```

### KPI Units

```DAX
KPI Units =
VAR v = [Units Sold]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "#,0" ) )
```

### KPI Units Δ

```DAX
KPI Units Δ =
VAR cur = [Units Sold]
VAR b = [Units Sold PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Units Colour

```DAX
KPI Units Colour =
VAR cur = [Units Sold]
VAR b = [Units Sold PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI AOV

```DAX
KPI AOV =
VAR v = [Average Order Value]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v, "#,0" ) )
```

### KPI AOV Δ

```DAX
KPI AOV Δ =
VAR cur = [Average Order Value]
VAR b = [Average Order Value PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI AOV Colour

```DAX
KPI AOV Colour =
VAR cur = [Average Order Value]
VAR b = [Average Order Value PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Sales per m2

```DAX
KPI Sales per m2 =
VAR v = [Sales per m²]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v, "#,0" ) )
```

### KPI Sales per m2 Δ

```DAX
KPI Sales per m2 Δ =
VAR cur = [Sales per m²]
VAR b = [Sales per m² PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Sales per m2 Colour

```DAX
KPI Sales per m2 Colour =
VAR cur = [Sales per m²]
VAR b = [Sales per m² PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Stores Growing

```DAX
KPI Stores Growing =
[Stores Growing] & " of " & [Store Count]
```

### KPI Stores Growing Δ

```DAX
KPI Stores Growing Δ =
IF ( ISBLANK ( [Sales PY] ), "No prior-year comparison", [Stores Shrinking] & " stores shrank " & [PY Label] )
```

### KPI Stores Growing Colour

```DAX
KPI Stores Growing Colour =
IF ( ISBLANK ( [Sales PY] ) || [Stores Shrinking] = 0, "#5B6573", "#B42318" )
```

### KPI Customers

```DAX
KPI Customers =
VAR v = [Total Customers]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "#,0" ) )
```

### KPI Customers Δ

```DAX
KPI Customers Δ =
VAR cur = [Total Customers]
VAR b = [Customers PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Customers Colour

```DAX
KPI Customers Colour =
VAR cur = [Total Customers]
VAR b = [Customers PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Sales per Customer

```DAX
KPI Sales per Customer =
VAR v = [Sales per Customer]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v, "#,0" ) )
```

### KPI Sales per Customer Δ

```DAX
KPI Sales per Customer Δ =
VAR cur = [Sales per Customer]
VAR b = [Sales per Customer PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Sales per Customer Colour

```DAX
KPI Sales per Customer Colour =
VAR cur = [Sales per Customer]
VAR b = [Sales per Customer PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Top20 Share

```DAX
KPI Top20 Share =
VAR v = [Top 20% Revenue Share]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0%" ) )
```

### KPI Top20 Share Δ

```DAX
KPI Top20 Share Δ =
VAR cur = [Top 20% Revenue Share]
VAR b = [Top 20% Revenue Share PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( cur - b ) * 100, "0.0" ) & " pts" & " " & [PY Label]
    )
```

### KPI Top20 Share Colour

```DAX
KPI Top20 Share Colour =
VAR cur = [Top 20% Revenue Share]
VAR b = [Top 20% Revenue Share PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#5B6573", "#5B6573" ) )
```

### KPI Orders per Customer

```DAX
KPI Orders per Customer =
VAR v = [Orders per Customer]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0.00" ) )
```

### KPI Orders per Customer Δ

```DAX
KPI Orders per Customer Δ =
VAR cur = [Orders per Customer]
VAR b = [Orders per Customer PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Orders per Customer Colour

```DAX
KPI Orders per Customer Colour =
VAR cur = [Orders per Customer]
VAR b = [Orders per Customer PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Attribution

```DAX
KPI Attribution =
VAR v = [Employee Attribution Rate %]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0.0%" ) )
```

### KPI Attribution Δ

```DAX
KPI Attribution Δ =
VAR cur = [Employee Attribution Rate %]
VAR b = [Attribution Rate % PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( cur - b ) * 100, "0.0" ) & " pts" & " " & [PY Label]
    )
```

### KPI Attribution Colour

```DAX
KPI Attribution Colour =
VAR cur = [Employee Attribution Rate %]
VAR b = [Attribution Rate % PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Unattributed

```DAX
KPI Unattributed =
VAR v = [Orders without Employee ID]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "#,0" ) )
```

### KPI Unattributed Δ

```DAX
KPI Unattributed Δ =
VAR cur = [Orders without Employee ID]
VAR b = [Orders without Employee ID PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( cur - b ), "#,0" ) & " orders" & " " & [PY Label]
    )
```

### KPI Unattributed Colour

```DAX
KPI Unattributed Colour =
VAR cur = [Orders without Employee ID]
VAR b = [Orders without Employee ID PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#B42318", "#1A7F4B" ) )
```

### KPI Sales per Employee

```DAX
KPI Sales per Employee =
VAR v = [Sales per Employee]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v, "#,0" ) )
```

### KPI Sales per Employee Δ

```DAX
KPI Sales per Employee Δ =
VAR cur = [Sales per Employee]
VAR b = [Sales per Employee PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Sales per Employee Colour

```DAX
KPI Sales per Employee Colour =
VAR cur = [Sales per Employee]
VAR b = [Sales per Employee PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Orders per Employee

```DAX
KPI Orders per Employee =
VAR v = [Orders per Employee]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0.0" ) )
```

### KPI Orders per Employee Δ

```DAX
KPI Orders per Employee Δ =
VAR cur = [Orders per Employee]
VAR b = [Orders per Employee PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Orders per Employee Colour

```DAX
KPI Orders per Employee Colour =
VAR cur = [Orders per Employee]
VAR b = [Orders per Employee PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Avg Daily Sales

```DAX
KPI Avg Daily Sales =
VAR v = [Avg Daily Sales]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v / 1000, "#,0.0" ) & "K" )
```

### KPI Avg Daily Sales Δ

```DAX
KPI Avg Daily Sales Δ =
VAR cur = [Avg Daily Sales]
VAR b = [Avg Daily Sales PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "Across " & FORMAT ( COUNTROWS ( dim_date ), "#,0" ) & " trading days",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Avg Daily Sales Colour

```DAX
KPI Avg Daily Sales Colour =
VAR cur = [Avg Daily Sales]
VAR b = [Avg Daily Sales PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Ramadan Daily

```DAX
KPI Ramadan Daily =
VAR v = [Ramadan Daily Sales]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v / 1000, "#,0.0" ) & "K" )
```

### KPI Ramadan Daily Δ

```DAX
KPI Ramadan Daily Δ =
VAR cur = [Ramadan Daily Sales]
VAR b = [Non-Ramadan Daily Sales]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No Ramadan days in this selection",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & "vs rest of year"
    )
```

### KPI Ramadan Daily Colour

```DAX
KPI Ramadan Daily Colour =
VAR cur = [Ramadan Daily Sales]
VAR b = [Non-Ramadan Daily Sales]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Weekend Daily

```DAX
KPI Weekend Daily =
VAR v = [Weekend Daily Sales]
RETURN
    IF ( ISBLANK ( v ), "–", "EGP " & FORMAT ( v / 1000, "#,0.0" ) & "K" )
```

### KPI Weekend Daily Δ

```DAX
KPI Weekend Daily Δ =
VAR cur = [Weekend Daily Sales]
VAR b = [Weekday Daily Sales]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & "vs weekdays"
    )
```

### KPI Weekend Daily Colour

```DAX
KPI Weekend Daily Colour =
VAR cur = [Weekend Daily Sales]
VAR b = [Weekday Daily Sales]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#1A7F4B", "#B42318" ) )
```

### KPI Ramadan Days

```DAX
KPI Ramadan Days =
VAR v = [Ramadan Days]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "#,0" ) )
```

### KPI Ramadan Days Δ

```DAX
KPI Ramadan Days Δ =
[Ramadan Seasons] & IF ( [Ramadan Seasons] = 1, " Ramadan season", " Ramadan seasons" )
```

### KPI Ramadan Days Colour

```DAX
KPI Ramadan Days Colour =
"#5B6573"
```

### KPI Returned Value

```DAX
KPI Returned Value =
VAR v = [Returned Amount]
RETURN
    IF ( ISBLANK ( v ), "–", IF ( ABS ( v ) >= 1000000, "EGP " & FORMAT ( v / 1000000, "#,0.00" ) & "M", IF ( ABS ( v ) >= 10000, "EGP " & FORMAT ( v / 1000, "#,0" ) & "K", "EGP " & FORMAT ( v, "#,0" ) ) ) )
```

### KPI Returned Value Δ

```DAX
KPI Returned Value Δ =
VAR cur = [Returned Amount]
VAR b = [Returned Amount PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Returned Value Colour

```DAX
KPI Returned Value Colour =
VAR cur = [Returned Amount]
VAR b = [Returned Amount PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#B42318", "#1A7F4B" ) )
```

### KPI Returns

```DAX
KPI Returns =
VAR v = [Number of Returns]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "#,0" ) )
```

### KPI Returns Δ

```DAX
KPI Returns Δ =
VAR cur = [Number of Returns]
VAR b = [Number of Returns PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( DIVIDE ( cur - b, b ) ), "0.0%" ) & " " & [PY Label]
    )
```

### KPI Returns Colour

```DAX
KPI Returns Colour =
VAR cur = [Number of Returns]
VAR b = [Number of Returns PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#B42318", "#1A7F4B" ) )
```

### KPI Preventable Share

```DAX
KPI Preventable Share =
VAR v = [Preventable Share %]
RETURN
    IF ( ISBLANK ( v ), "–", FORMAT ( v, "0%" ) )
```

### KPI Preventable Share Δ

```DAX
KPI Preventable Share Δ =
VAR cur = [Preventable Share %]
VAR b = [Preventable Share % PY]
RETURN
    IF (
        ISBLANK ( b ) || b = 0,
        "No prior-year comparison",
        IF ( cur >= b, "▲ ", "▼ " ) & FORMAT ( ABS ( cur - b ) * 100, "0.0" ) & " pts" & " " & [PY Label]
    )
```

### KPI Preventable Share Colour

```DAX
KPI Preventable Share Colour =
VAR cur = [Preventable Share %]
VAR b = [Preventable Share % PY]
RETURN
    IF ( ISBLANK ( b ) || b = 0 || cur = b, "#5B6573", IF ( cur > b, "#B42318", "#1A7F4B" ) )
```

## Colours

Hex colours applied through conditional formatting (Format → fx → Field value) to highlight one bar or dot and to colour good/bad values.

### Colour Clothing

```DAX
Colour Clothing =
IF ( SELECTEDVALUE ( dim_products[category] ) = "Clothing", "#1F4E79", "#B8C0CC" )
```

### Colour Weekend

```DAX
Colour Weekend =
IF ( SELECTEDVALUE ( dim_date[is_weekend] ) = 1, "#1F4E79", "#B8C0CC" )
```

### Colour Preventable Reason

```DAX
Colour Preventable Reason =
IF ( SELECTEDVALUE ( fact_returns[return_reason] ) IN { "Duplicate Order", "Wrong Item Delivered" }, "#1F4E79", "#B8C0CC" )
```

### Colour Region Gain

```DAX
Colour Region Gain =
VAR hasPY = NOT ISBLANK ( CALCULATE ( [Sales PY], ALLSELECTED ( dim_stores[region] ) ) )
VAR bestGain = MAXX ( TOPN ( 1, ALLSELECTED ( dim_stores[region] ), [Sales Change vs PY], DESC ), dim_stores[region] )
VAR bestSales = MAXX ( TOPN ( 1, ALLSELECTED ( dim_stores[region] ), [Total Sales], DESC ), dim_stores[region] )
VAR target = IF ( hasPY, bestGain, bestSales )
RETURN
    IF ( SELECTEDVALUE ( dim_stores[region] ) = target, "#1F4E79", "#B8C0CC" )
```

### Colour Top Sales per m2

```DAX
Colour Top Sales per m2 =
VAR t = TOPN ( 1, ALLSELECTED ( dim_stores[store_name] ), [Sales per m²], DESC )
RETURN
    IF ( SELECTEDVALUE ( dim_stores[store_name] ) IN SELECTCOLUMNS ( t, "@k", dim_stores[store_name] ), "#1F4E79", "#B8C0CC" )
```

### Colour Top Unattributed

```DAX
Colour Top Unattributed =
VAR t = TOPN ( 3, ALLSELECTED ( dim_stores[store_name] ), [Orders without Employee ID], DESC )
RETURN
    IF ( SELECTEDVALUE ( dim_stores[store_name] ) IN SELECTCOLUMNS ( t, "@k", dim_stores[store_name] ), "#1F4E79", "#B8C0CC" )
```

### Colour Top Return Rate

```DAX
Colour Top Return Rate =
VAR t = TOPN ( 1, ALLSELECTED ( dim_stores[store_name] ), [Return Rate (Value) %], DESC )
RETURN
    IF ( SELECTEDVALUE ( dim_stores[store_name] ) IN SELECTCOLUMNS ( t, "@k", dim_stores[store_name] ), "#1F4E79", "#B8C0CC" )
```

### Colour Sales vs PY

```DAX
Colour Sales vs PY =
IF (
    ISBLANK ( [Sales vs PY %] ),
    "#5B6573",
    IF ( [Sales vs PY %] >= 0, "#1A7F4B", "#B42318" )
)
```

### Colour Tier Flag

```DAX
Colour Tier Flag =
IF ( SELECTEDVALUE ( dim_customers[loyalty_tier] ) IN { "Bronze", "Silver" }, "#B42318", "#1D2633" )
```

### Colour Low Margin

```DAX
Colour Low Margin =
IF ( [Gross Margin %] < 0.12, "#B42318", "#1D2633" )
```

### Colour Margin Scale

```DAX
Colour Margin Scale =
VAR m = [Gross Margin %]
RETURN
    SWITCH ( TRUE (), ISBLANK ( m ), BLANK (), m >= 0.23, "#C9D9EA", m >= 0.21, "#E1EAF3", "#F4F7FA" )
```

### Colour Sales vs PY Fill

```DAX
Colour Sales vs PY Fill =
IF (
    ISBLANK ( [Sales vs PY %] ),
    BLANK (),
    IF ( [Sales vs PY %] >= 0, "#E8F4EE", "#FCEBEA" )
)
```

### Colour Platinum

```DAX
Colour Platinum =
IF (
    SELECTEDVALUE ( dim_customers[loyalty_tier] ) = "Platinum",
    "#1F4E79",
    "#B8C0CC"
)
```

### Colour Ramadan

```DAX
Colour Ramadan =
IF ( SELECTEDVALUE ( dim_date[Period Type] ) = "Ramadan", "#1F4E79", "#B8C0CC" )
```

### Colour Ramadan Change

```DAX
Colour Ramadan Change =
VAR _isRamadan = SELECTEDVALUE ( dim_date[Period Type] ) = "Ramadan"
VAR _ramadan =
    CALCULATE ( [Avg Daily Sales], REMOVEFILTERS ( dim_date[Period Type] ), dim_date[is_ramadan] = 1 )
VAR _rest =
    CALCULATE ( [Avg Daily Sales], REMOVEFILTERS ( dim_date[Period Type] ), dim_date[is_ramadan] = 0 )
RETURN
    IF (
        _isRamadan && _rest > 0,
        IF ( _ramadan >= _rest, "#1A7F4B", "#B42318" ),
        "#1D2633"
    )
```

## Titles

Dynamic visual titles and subtitles. Each title states the current finding and recalculates when slicers change, with a neutral fallback when the selection leaves too little data.

### Title Exec Monthly

```DAX
Title Exec Monthly =
VAR cur = [Total Sales]
VAR py = [Sales PY]
VAR Mo = ADDCOLUMNS ( VALUES ( dim_date[month_name] ), "@s", [Total Sales] )
VAR bm = MAXX ( TOPN ( 1, Mo, [@s], DESC ), dim_date[month_name] )
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( cur ), "No sales for this selection",
        ISBLANK ( py ) || py = 0, bm & " was the strongest month for sales",
        "Sales " & IF ( cur >= py, "grew ", "fell " ) & FORMAT ( ABS ( DIVIDE ( cur - py, py ) ), "0.0%" ) & " " & [PY Label] & "; " & bm & " was the strongest month"
    )
```

### Subtitle Exec Monthly

```DAX
Subtitle Exec Monthly =
VAR y = SELECTEDVALUE ( dim_date[year] )
VAR legendText =
    IF (
        ISBLANK ( y ),
        "Monthly sales (EGP), selected years combined",
        "Monthly sales (EGP) · Solid line: " & y & " · Dashed line: " & ( y - 1 )
    )
RETURN
    legendText & " · " & [Filter Label]
```

### Title Exec Category

```DAX
Title Exec Category =
VAR C =
    ADDCOLUMNS ( VALUES ( dim_products[category] ), "@ss", [Sales Share %], "@gs", [Gross Profit Share %] )
VAR t = TOPN ( 1, C, [@gs] - [@ss], DESC )
RETURN
    IF (
        COUNTROWS ( C ) <= 1,
        SELECTEDVALUE ( dim_products[category] ) & " only: gross margin " & FORMAT ( [Gross Margin %], "0.0%" ),
        MAXX ( t, dim_products[category] ) & " is " & FORMAT ( MAXX ( t, [@ss] ), "0%" ) & " of sales but "
            & FORMAT ( MAXX ( t, [@gs] ), "0%" ) & " of gross profit"
    )
```

### Subtitle Exec Category

```DAX
Subtitle Exec Category =
"Share of sales vs share of gross profit, by category · " & [Context Label]
```

### Title Exec Region

```DAX
Title Exec Region =
VAR R =
    ADDCOLUMNS ( VALUES ( dim_stores[region] ), "@d", [Sales Change vs PY], "@g", [Sales vs PY %], "@s", [Total Sales] )
VAR t = TOPN ( 1, R, [@d], DESC )
VAR big = TOPN ( 1, R, [@s], DESC )
VAR d = MAXX ( t, [@d] )
VAR ad = ABS ( d )
RETURN
    SWITCH (
        TRUE (),
        COUNTROWS ( R ) = 1, SELECTEDVALUE ( dim_stores[region] ) & " sales: " & IF ( ABS ( [Total Sales] ) >= 1000000, "EGP " & FORMAT ( [Total Sales] / 1000000, "#,0.00" ) & "M", IF ( ABS ( [Total Sales] ) >= 10000, "EGP " & FORMAT ( [Total Sales] / 1000, "#,0" ) & "K", "EGP " & FORMAT ( [Total Sales], "#,0" ) ) ),
        ISBLANK ( [Sales PY] ), MAXX ( big, dim_stores[region] ) & " is the largest region with " & FORMAT ( DIVIDE ( MAXX ( big, [@s] ), [Total Sales] ), "0%" ) & " of sales",
        d >= 0, MAXX ( t, dim_stores[region] ) & " added " & IF ( ABS ( ad ) >= 1000000, "EGP " & FORMAT ( ad / 1000000, "#,0.00" ) & "M", IF ( ABS ( ad ) >= 10000, "EGP " & FORMAT ( ad / 1000, "#,0" ) & "K", "EGP " & FORMAT ( ad, "#,0" ) ) ) & " (" & FORMAT ( MAXX ( t, [@g] ), "+0%;-0%" ) & ") " & [PY Label] & ", the biggest gain of any region",
        "Every region shrank " & [PY Label] & "; " & MAXX ( t, dim_stores[region] ) & " fell least (" & FORMAT ( MAXX ( t, [@g] ), "+0%;-0%" ) & ")"
    )
```

### Subtitle Exec Region

```DAX
Subtitle Exec Region =
"Sales by region (EGP); highlighted = biggest gain · " & [Context Label]
```

### Title Exec Stores

```DAX
Title Exec Stores =
VAR St = ADDCOLUMNS ( VALUES ( dim_stores[store_name] ), "@s", [Total Sales], "@g", [Sales vs PY %] )
VAR top3 = TOPN ( 3, St, [@s], DESC )
VAR share = DIVIDE ( SUMX ( top3, [@s] ), CALCULATE ( [Total Sales], REMOVEFILTERS ( dim_stores[store_name] ) ) )
VAR shrank = COUNTROWS ( FILTER ( top3, [@g] < 0 ) )
RETURN
    IF (
        COUNTROWS ( St ) < 4,
        "Store performance for this selection",
        "The top 3 stores deliver " & FORMAT ( share, "0%" ) & " of sales"
            & IF ( ISBLANK ( [Sales PY] ), "", IF ( shrank = 0, ", and all 3 grew", ", but " & shrank & " of them shrank" ) )
    )
```

### Subtitle Exec Stores

```DAX
Subtitle Exec Stores =
VAR y = SELECTEDVALUE ( dim_date[year] )
RETURN
    IF (
        ISBLANK ( y ),
        "Top 6 stores by sales · " & [Context Label],
        "Top 6 stores by sales, " & y & " · % change vs " & ( y - 1 ) & " · " & [Filter Label]
    )
```

### Title Sales Subcategory

```DAX
Title Sales Subcategory =
VAR _subcat = VALUES ( dim_products[subcategory] )
VAR _top3 = TOPN ( 3, _subcat, [Gross Profit], DESC )
VAR _clothingInTop3 =
    COUNTROWS (
        FILTER ( _top3, CALCULATE ( SELECTEDVALUE ( dim_products[category] ) ) = "Clothing" )
    ) + 0
VAR _best = TOPN ( 1, _subcat, [Gross Profit], DESC )
VAR _bestName = MAXX ( _best, dim_products[subcategory] )
VAR _bestMargin = CALCULATE ( [Gross Margin %], _best )
VAR _categoryFiltered = ISFILTERED ( dim_products[category] )
RETURN
    SWITCH (
        TRUE (),
        COUNTROWS ( _subcat ) >= 3 && _clothingInTop3 = 3 && NOT _categoryFiltered,
            "The 3 most profitable subcategories are all clothing",
        COUNTROWS ( _subcat ) >= 3 && _clothingInTop3 = 0 && NOT _categoryFiltered,
            "The 3 most profitable subcategories are all electronics",
        _bestName & " is the most profitable subcategory (" & FORMAT ( _bestMargin, "0%" ) & " margin)"
    )
```

### Subtitle Sales Subcategory

```DAX
Subtitle Sales Subcategory =
"Gross profit by subcategory (EGP); clothing highlighted · " & [Context Label]
```

### Title Sales Scatter

```DAX
Title Sales Scatter =
VAR Sub =
    FILTER (
        ADDCOLUMNS (
            VALUES ( dim_products[subcategory] ),
            "@c", CALCULATE ( SELECTEDVALUE ( dim_products[category] ) ),
            "@m", [Gross Margin %]
        ),
        NOT ISBLANK ( [@m] )
    )
VAR E = FILTER ( Sub, [@c] = "Electronics" )
VAR C = FILTER ( Sub, [@c] = "Clothing" )
VAR et = IF ( COUNTROWS ( E ) > 0, "Electronics earns " & FORMAT ( MINX ( E, [@m] ), "0%" ) & "–" & FORMAT ( MAXX ( E, [@m] ), "0%" ) & " margins" )
VAR ct = IF ( COUNTROWS ( C ) > 0, "clothing earns " & FORMAT ( MINX ( C, [@m] ), "0%" ) & "–" & FORMAT ( MAXX ( C, [@m] ), "0%" ) )
RETURN
    SWITCH (
        TRUE (),
        NOT ISBLANK ( et ) && NOT ISBLANK ( ct ), et & "; " & ct,
        NOT ISBLANK ( et ), et,
        NOT ISBLANK ( ct ), "Clothing earns " & FORMAT ( MINX ( C, [@m] ), "0%" ) & "–" & FORMAT ( MAXX ( C, [@m] ), "0%" ) & " margins",
        "Sales vs margin by subcategory"
    )
```

### Subtitle Sales Scatter

```DAX
Subtitle Sales Scatter =
"Each dot is a subcategory: sales (x) vs gross margin (y) · " & [Context Label]
```

### Title Sales Products

```DAX
Title Sales Products =
VAR _top = TOPN ( 1, VALUES ( dim_products[product_name_raw] ), [Gross Profit], DESC )
VAR gp = CALCULATE ( [Gross Profit], _top )
VAR gm = CALCULATE ( [Gross Margin %], _top )
RETURN
    IF (
        ISBLANK ( gp ),
        "No product sales for this selection",
        MAXX ( _top, dim_products[product_name_raw] ) & " is the top profit earner: " & IF ( ABS ( gp ) >= 1000000, "EGP " & FORMAT ( gp / 1000000, "#,0.00" ) & "M", IF ( ABS ( gp ) >= 10000, "EGP " & FORMAT ( gp / 1000, "#,0" ) & "K", "EGP " & FORMAT ( gp, "#,0" ) ) ) & " at " & FORMAT ( gm, "0.0%" ) & " margin"
    )
```

### Subtitle Sales Products

```DAX
Subtitle Sales Products =
"Products ranked by gross profit; margin under 12% in red · " & [Context Label]
```

### Title Stores Psm

```DAX
Title Stores Psm =
VAR St =
    ADDCOLUMNS ( FILTER ( VALUES ( dim_stores[store_name] ), [Total Sales] > 0 ), "@p", [Sales per m²] )
VAR n = COUNTROWS ( St )
VAR TopRow = TOPN ( 1, St, [@p], DESC )
VAR nm = SUBSTITUTE ( MAXX ( TopRow, dim_stores[store_name] ), "Lotus ", "" )
VAR v = MAXX ( TopRow, [@p] )
VAR others = AVERAGEX ( EXCEPT ( St, TopRow ), [@p] )
RETURN
    SWITCH (
        TRUE (),
        n = 0, "No store sales for this selection",
        n = 1, nm & " earns EGP " & FORMAT ( v, "#,0" ) & " per m²",
        nm & " earns the most per m², " & FORMAT ( DIVIDE ( v, others ) - 1, "0%" ) & " above the other " & ( n - 1 ) & " stores' average"
    )
```

### Subtitle Stores Psm

```DAX
Subtitle Stores Psm =
"Sales per m² of floor space (EGP) · " & [Context Label]
```

### Title Stores Scatter

```DAX
Title Stores Scatter =
VAR _stores =
    FILTER (
        ADDCOLUMNS (
            VALUES ( dim_stores[store_name] ),
            "@x", [Store Floor Space],
            "@y", [Total Sales],
            "@p", [Sales per m²]
        ),
        NOT ISBLANK ( [@y] )
    )
VAR _n = COUNTROWS ( _stores )
VAR _sx = SUMX ( _stores, [@x] )
VAR _sy = SUMX ( _stores, [@y] )
VAR _sxy = SUMX ( _stores, [@x] * [@y] )
VAR _sxx = SUMX ( _stores, [@x] * [@x] )
VAR _slope = DIVIDE ( _n * _sxy - _sx * _sy, _n * _sxx - _sx * _sx )
VAR _intercept = DIVIDE ( _sy - _slope * _sx, _n )
VAR _best = TOPN ( 1, _stores, [@p], DESC )
VAR _name = MAXX ( _best, dim_stores[store_name] )
VAR _size = MAXX ( _best, [@x] )
VAR _sales = MAXX ( _best, [@y] )
VAR _uplift = DIVIDE ( _sales, _intercept + _slope * _size ) - 1
VAR _beaten = FILTER ( _stores, [@x] > _size && [@y] < _sales )
VAR _nBeaten = COUNTROWS ( _beaten ) + 0
VAR _clause =
    SWITCH (
        TRUE (),
        _nBeaten = 1, " and outsold the bigger " & MAXX ( _beaten, dim_stores[store_name] ),
        _nBeaten > 1, " and outsold " & _nBeaten & " bigger stores",
        ""
    )
RETURN
    SWITCH (
        TRUE (),
        _n < 3, "Store size vs sales",
        _uplift > 0, _name & " sells " & FORMAT ( _uplift, "0%" ) & " more than its size predicts" & _clause,
        _name & " earns the most per m² of floor space"
    )
```

### Subtitle Stores Scatter

```DAX
Subtitle Stores Scatter =
"Each dot is a store: floor space in m² (x) vs sales in EGP (y); dashed line = trend; highlighted = most sales per m² · " & [Context Label]
```

### Title Stores Table

```DAX
Title Stores Table =
VAR St =
    FILTER ( ADDCOLUMNS ( VALUES ( dim_stores[store_name] ), "@g", [Sales vs PY %] ), NOT ISBLANK ( [@g] ) )
VAR n = COUNTROWS ( St )
VAR neg = COUNTROWS ( FILTER ( St, [@g] < 0 ) )
VAR w = TOPN ( 1, St, [@g], ASC )
RETURN
    SWITCH (
        TRUE (),
        n = 0, "Store league table (no prior-year comparison)",
        neg = 0, "All " & n & " stores grew " & [PY Label],
        neg & " of " & n & " stores shrank " & [PY Label] & ", led by "
            & SUBSTITUTE ( MAXX ( w, dim_stores[store_name] ), "Lotus ", "" ) & " (" & FORMAT ( MAXX ( w, [@g] ), "+0%;-0%" ) & ")"
    )
```

### Subtitle Stores Table

```DAX
Subtitle Stores Table =
"Stores sorted by change vs prior year, weakest first · " & [Context Label]
```

### Title Cust Pareto

```DAX
Title Cust Pareto =
VAR s = [Top 20% Revenue Share]
RETURN
    IF ( ISBLANK ( s ), "Customer revenue concentration", "The top 20% of customers drive " & FORMAT ( s, "0%" ) & " of revenue" )
```

### Subtitle Cust Pareto

```DAX
Subtitle Cust Pareto =
"Cumulative share of revenue by top X% of customers · " & [Context Label]
```

### Title Cust Tier

```DAX
Title Cust Tier =
VAR _tiers =
    FILTER (
        ADDCOLUMNS ( VALUES ( dim_customers[loyalty_tier] ), "@p", [Sales per Customer] ),
        NOT ISBLANK ( [@p] )
    )
VAR _platinum = MAXX ( FILTER ( _tiers, dim_customers[loyalty_tier] = "Platinum" ), [@p] )
VAR _best = TOPN ( 1, _tiers, [@p], DESC )
VAR _bestName = MAXX ( _best, dim_customers[loyalty_tier] )
VAR _bestSpend = MAXX ( _best, [@p] )
VAR _lowest = TOPN ( 1, _tiers, [@p], ASC )
VAR _lowName = MAXX ( _lowest, dim_customers[loyalty_tier] )
VAR _lowSpend = MAXX ( _lowest, [@p] )
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( _platinum ) || COUNTROWS ( _tiers ) < 2, "Spend per customer by loyalty tier",
        _bestName = "Platinum",
            "Platinum members spend the most: " & FORMAT ( DIVIDE ( _platinum, _lowSpend ) - 1, "0%" )
                & " more per head than " & _lowName & " members",
        "Platinum members spend " & FORMAT ( 1 - DIVIDE ( _platinum, _bestSpend ), "0%" )
            & " less per head than " & _bestName & " members: Tiers don't track value"
    )
```

### Subtitle Cust Tier

```DAX
Subtitle Cust Tier =
"Average spend per customer by loyalty tier (EGP); Platinum highlighted · " & [Context Label]
```

### Title Cust Top

```DAX
Title Cust Top =
VAR T10 = TOPN ( 10, FILTER ( VALUES ( dim_customers[customer_id] ), [Total Sales] > 0 ), [Total Sales], DESC )
VAR nLow = COUNTROWS ( FILTER ( T10, CALCULATE ( SELECTEDVALUE ( dim_customers[loyalty_tier] ) ) IN { "Bronze", "Silver" } ) )
RETURN
    IF ( COUNTROWS ( T10 ) = 0, "Top customers", nLow & " of the top " & COUNTROWS ( T10 ) & " spenders are Bronze or Silver members" )
```

### Subtitle Cust Top

```DAX
Subtitle Cust Top =
"Top 10 customers by spend; Bronze/Silver flagged in red · " & [Context Label]
```

### Title Emp Unattributed

```DAX
Title Emp Unattributed =
VAR St = ADDCOLUMNS ( VALUES ( dim_stores[store_name] ), "@u", [Orders without Employee ID] )
VAR tot = [Orders without Employee ID]
VAR t3 = TOPN ( 3, St, [@u], DESC )
RETURN
    IF (
        ISBLANK ( tot ) || tot = 0,
        "Every order has an employee recorded",
        "3 stores account for " & FORMAT ( DIVIDE ( SUMX ( t3, [@u] ), tot ), "0%" ) & " of orders with no employee, led by "
            & SUBSTITUTE ( MAXX ( TOPN ( 1, St, [@u], DESC ), dim_stores[store_name] ), "Lotus ", "" )
    )
```

### Subtitle Emp Unattributed

```DAX
Subtitle Emp Unattributed =
"Orders with no employee recorded, by store; top 3 highlighted · " & [Context Label]
```

### Title Emp Trend

```DAX
Title Emp Trend =
VAR Mo =
    FILTER ( ADDCOLUMNS ( VALUES ( dim_date[month_name] ), "@r", [Employee Attribution Rate %] ), NOT ISBLANK ( [@r] ) )
VAR lo = TOPN ( 1, Mo, [@r], ASC )
RETURN
    IF (
        COUNTROWS ( Mo ) = 0,
        "Attribution rate by month",
        "Attribution ranged " & FORMAT ( MINX ( Mo, [@r] ), "0.0%" ) & "–" & FORMAT ( MAXX ( Mo, [@r] ), "0.0%" ) & "; lowest in " & MAXX ( lo, dim_date[month_name] )
    )
```

### Subtitle Emp Trend

```DAX
Subtitle Emp Trend =
VAR y = SELECTEDVALUE ( dim_date[year] )
VAR legendText =
    IF (
        ISBLANK ( y ),
        "Share of orders credited to an employee, by month, selected years combined",
        "Share of orders credited to an employee · Solid line: " & y & " · Dashed line: " & ( y - 1 )
    )
RETURN
    legendText & " · " & [Filter Label]
```

### Title Emp Top

```DAX
Title Emp Top =
VAR _top = TOPN ( 1, FILTER ( VALUES ( dim_employees[employee_id] ), [Total Sales] > 0 ), [Total Sales], DESC )
VAR nm = CALCULATE ( SELECTEDVALUE ( dim_employees[Employee Name] ), _top )
VAR s = CALCULATE ( [Total Sales], _top )
VAR o = CALCULATE ( [Total Orders], _top )
VAR st = CALCULATE ( [Employee Store], _top )
RETURN
    IF (
        ISBLANK ( s ),
        "Top employees by attributed sales",
        nm & " (" & st & ") leads with " & IF ( ABS ( s ) >= 1000000, "EGP " & FORMAT ( s / 1000000, "#,0.00" ) & "M", IF ( ABS ( s ) >= 10000, "EGP " & FORMAT ( s / 1000, "#,0" ) & "K", "EGP " & FORMAT ( s, "#,0" ) ) ) & " from " & o & " orders"
    )
```

### Subtitle Emp Top

```DAX
Subtitle Emp Top =
"Employees ranked by attributed sales · " & [Context Label]
```

### Title Season Ramadan

```DAX
Title Season Ramadan =
VAR C =
    ADDCOLUMNS (
        VALUES ( dim_products[category] ),
        "@c",
            VAR r = [Ramadan Daily Sales]
            VAR n = [Non-Ramadan Daily Sales]
            RETURN
                IF ( r > 0 && n > 0, r / n - 1 )
    )
VAR txt =
    CONCATENATEX ( FILTER ( C, NOT ISBLANK ( [@c] ) ), dim_products[category] & " " & FORMAT ( [@c], "+0%;-0%" ), ", ", [@c], ASC )
RETURN
    IF ( [Ramadan Days] = 0 || ISBLANK ( [Ramadan Days] ), "No Ramadan days in this selection", "Ramadan changes daily sales: " & txt )
```

### Subtitle Season Ramadan

```DAX
Subtitle Season Ramadan =
"Average daily sales (EGP), Ramadan vs rest of year · " & [Context Label]
```

### Title Season Weekday

```DAX
Title Season Weekday =
VAR g = DIVIDE ( [Weekend Daily Sales], [Weekday Daily Sales] ) - 1
RETURN
    IF (
        ISBLANK ( [Weekend Daily Sales] ),
        "Average daily sales by day of week",
        "Weekends (Fri–Sat) average " & FORMAT ( ABS ( g ), "0%" ) & IF ( g < 0, " below ", " above " ) & "weekdays"
    )
```

### Subtitle Season Weekday

```DAX
Subtitle Season Weekday =
"Average daily sales by day of week (EGP); weekend highlighted · " & [Context Label]
```

### Title Season Monthly

```DAX
Title Season Monthly =
VAR Mo = FILTER ( ADDCOLUMNS ( VALUES ( dim_date[month_name] ), "@s", [Avg Monthly Sales] ), NOT ISBLANK ( [@s] ) )
VAR hi = MAXX ( TOPN ( 1, Mo, [@s], DESC ), dim_date[month_name] )
VAR lo = MAXX ( TOPN ( 1, Mo, [@s], ASC ), dim_date[month_name] )
RETURN
    IF ( COUNTROWS ( Mo ) < 2, "Average monthly sales", hi & " is the strongest month and " & lo & " the weakest" )
```

### Subtitle Season Monthly

```DAX
Subtitle Season Monthly =
"Average sales per month across selected years (EGP); Ramadan fell in Mar–May · " & [Context Label]
```

### Title Ret Reason

```DAX
Title Ret Reason =
IF (
    ISBLANK ( [Returned Amount] ),
    "No returns in this selection",
    "Preventable errors (duplicate orders, wrong items) are " & FORMAT ( [Preventable Share %], "0%" ) & " of returned value"
)
```

### Subtitle Ret Reason

```DAX
Subtitle Ret Reason =
"Returned value by reason (EGP); preventable reasons highlighted · " & [Context Label]
```

### Title Ret Monthly

```DAX
Title Ret Monthly =
VAR Mo =
    FILTER ( ADDCOLUMNS ( VALUES ( dim_date[month_name] ), "@r", [Return Rate (Value) %] ), NOT ISBLANK ( [@r] ) )
VAR pk = TOPN ( 1, Mo, [@r], DESC )
RETURN
    IF (
        COUNTROWS ( Mo ) = 0,
        "Return rate by month",
        "Returns peaked at " & FORMAT ( MAXX ( pk, [@r] ), "0.0%" ) & " of sales in " & MAXX ( pk, dim_date[month_name] )
            & ", vs " & FORMAT ( [Return Rate (Value) %], "0.0%" ) & " overall"
    )
```

### Subtitle Ret Monthly

```DAX
Subtitle Ret Monthly =
"Returned value as % of sales, by month of order · " & [Context Label]
```

### Title Ret Store

```DAX
Title Ret Store =
VAR St =
    FILTER ( ADDCOLUMNS ( VALUES ( dim_stores[store_name] ), "@r", [Return Rate (Value) %] ), NOT ISBLANK ( [@r] ) )
VAR t = TOPN ( 1, St, [@r], DESC )
VAR ov = CALCULATE ( [Return Rate (Value) %], REMOVEFILTERS ( dim_stores[store_name] ) )
RETURN
    IF (
        COUNTROWS ( St ) = 0,
        "Return rate by store",
        SUBSTITUTE ( MAXX ( t, dim_stores[store_name] ), "Lotus ", "" ) & " returns " & FORMAT ( MAXX ( t, [@r] ), "0.0%" )
            & " of its sales, " & FORMAT ( DIVIDE ( MAXX ( t, [@r] ), ov ), "0.0" ) & "× the overall rate"
    )
```

### Subtitle Ret Store

```DAX
Subtitle Ret Store =
"Returned value as % of sales, by store; highest highlighted · " & [Context Label]
```

### Subtitle Ret Trend

```DAX
Subtitle Ret Trend =
VAR y = SELECTEDVALUE ( dim_date[year] )
VAR legendText =
    IF (
        ISBLANK ( y ),
        "Returned value as % of sales, by month of order , selected years combined",
        "Returned value as % of sales, by month of order · Solid line: " & y & " · Dashed line: " & ( y - 1 )
    )
RETURN
    legendText & " · " & [Filter Label]
```

## Other

Helper measures used by individual visuals.

### Filter Label

```DAX
Filter Label =
VAR r =
    IF (
        ISFILTERED ( dim_stores[region] ),
        CONCATENATEX ( VALUES ( dim_stores[region] ), dim_stores[region], ", " ),
        "All regions"
    )
VAR c =
    IF (
        ISFILTERED ( dim_products[category] ),
        CONCATENATEX ( VALUES ( dim_products[category] ), dim_products[category], ", " ),
        "All categories"
    )
RETURN
    r & " · " & c
```

### Category Share

```DAX
Category Share =
SWITCH (
    SELECTEDVALUE ( 'Share Metric'[Order] ),
    1, [Sales Share %],
    2, [Gross Profit Share %]
)
```

### Ramadan Change Label

```DAX
Ramadan Change Label =
VAR _isRamadan = SELECTEDVALUE ( dim_date[Period Type] ) = "Ramadan"
VAR _ramadan =
    CALCULATE ( [Avg Daily Sales], REMOVEFILTERS ( dim_date[Period Type] ), dim_date[is_ramadan] = 1 )
VAR _rest =
    CALCULATE ( [Avg Daily Sales], REMOVEFILTERS ( dim_date[Period Type] ), dim_date[is_ramadan] = 0 )
RETURN
    IF ( _isRamadan && _rest > 0, FORMAT ( DIVIDE ( _ramadan, _rest ) - 1, "+0%;-0%" ) )
```

