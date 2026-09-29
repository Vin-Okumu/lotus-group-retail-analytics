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
