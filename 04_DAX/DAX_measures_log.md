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

## Sales Previous Year
    Sales Previous Year =
    CALCULATE(
        [Total Sales],
        SAMEPERIODLASTYEAR(dim_date[full_date])
    )

## YoY Sales Growth
    YoY Sales Growth % =
    DIVIDE(
        [Total Sales] - [Sales Previous Year],
        [Sales Previous Year]
    )

Format as Percentage.

This will be particularly useful for your executive page.

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



