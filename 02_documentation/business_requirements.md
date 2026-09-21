<h1 align = "center"> 
Lotus Group Retail Analytics: Business Requirements
</h1>

# 1. Business context

The Lotus Group is a fictional Egyptian retail organization operating across multiple stores and selling products in two major categories:

- Clothing
- Electronics

The organization has:

- multiple stores
- customers across Egypt
- employees
- clothing and electronic products
- sales orders
- order line items
- returns
- a calendar containing Egyptian seasonal information such as Ramadan.

The purpose of the BI solution driven from this analytics process is to turn these operational data sources into information that management can use to understand sales performance, customer behavior, product performance, store performance, employee performance, returns, and seasonality.

# 2. Primary business objective

My primary goal undertaking this project is:

Develop an integrated retail BI solution that enables Lotus Group management to:
- monitor sales performance, 
- understand the drivers of revenue,
- identify differences in store/product/customer performance, 
- evaluate returns, and 
- investigate temporal and seasonal purchasing patterns.

This objective should be broad enough to support the entire dataset but focused enough to prevent the project from becoming a collection of unrelated charts.

# 3. Stakeholders

With the dataset at hand, there is a potential to simulate several stakeholder groups, including:

Stakeholder	            | Information they need
------------------------|-----------------------------------------------
Executive Management	| Overall business performance
Sales Management	    | Sales trends and order performance
Store Managers	        | Store and employee performance
Product Management	    | Product/category performance
Customer Management	    | Customer purchasing behavior
Operations Management	| Orders and returns
BI/Analytics Team	    | Data quality, modeling and analytical metrics

- This should give us a useful principle:

    - Every dashboard element should ultimately answer a stakeholder question.

# 4. Business questions

Before beginning charting, I want to establish the questions first, organized into the following six analytical domains.

### I. Overall sales performance

Management should be able to answer:

- What is total sales revenue?
- How many orders were placed?
- How many units were sold?
- What is the average order value?
- How has sales changed over time?
- What is the year-over-year growth rate?
- Which months/quarters generate the most sales?
- What proportion of sales comes from each product category?

These I expect to be the foundation of the executive dashboard.

### II. Product performance

Management should be able to answer:

- Which product categories generate the most revenue?
- Which individual products generate the most revenue?
- Which products sell the greatest number of units?
- Are the highest-volume products also the highest-revenue products?
- Which products have relatively high return amounts?
- What proportion of total sales comes from the top products?

This I expect to allow us distinguish sales volume from sales value; an important analytical distinction.

- For instance, a product selling 1,000 units isn't necessarily more commercially important than one selling 300 units if their prices differ substantially.

### III. Store and geographic performance

Because dim_stores contains governorate, region and store type, I'm interested in investigating:

- Which stores generate the most sales?
- Which regions generate the most sales?
- Which governorates generate the most sales?
- How do Flagship, Standard and Small stores compare?
- Which stores process the most orders?
- Are high-sales stores also high-order-volume stores?
- How does employee productivity differ across stores?

I suspect this will give me a strong geographic/operational analysis page.

### IV. Customer performance

The customer dimension should allow us to investigate:

- How many customers have made purchases?
- How is sales distributed across customers?
- Which customers/customer tiers generate the highest sales?
- How many orders does the typical customer place?
- What is the average customer order value?
- Are sales highly concentrated among a small number of customers?
- Does customer purchasing vary geographically?

### V. Employee performance

Since we have employees linked to stores and sales orders, I'm interested in investigating:

- How many employees are associated with each store?
- How much sales revenue is associated with each employee?
- How many orders does each employee handle?
- What is average sales per employee?
- Are there significant differences between stores?

But there's an important caveat that we'll investigate during the profiling phase:

- What exactly does the employee relationship represent?

    - If an employee is simply the person recorded against an order, then we can analyze sales associated with that employee.

    - However, I want to refrain from automatically interpreting employee association as "employee productivity" until I've established the business meaning of this field.

That is precisely the type of issue I'll be uncovering in the data audit phase.

### VI. Returns

For this project, I want to perceive Returns as particularly valuable because they allow us to move beyond simple revenue reporting.

Questions of interest here include:

- What is the total returned amount?
- What percentage of sales value is returned?
- What are the major return reasons?
- Which products have the highest return amounts?
- Which stores have the highest return rates?
- Are particular product categories associated with more returns?
- Do returns exhibit seasonal patterns?

This should give the project an operational quality dimension, rather than simply measuring sales.

### VII. Time and seasonality

I understand the dim_date table should give this project another analytical dimension.

I'll want to investigate:

- Year
- Quarter
- Month
- Week
- Day
- Weekend/weekday
- Ramadan/non-Ramadan

Questions of interest include:

- What are the monthly sales patterns?
- Which days of the week generate the most sales?
- Are weekends materially different from weekdays?
- Does sales behavior differ during Ramadan?
- Which products/categories perform differently during Ramadan?
- Does return behavior change seasonally?

The Ramadan analysis could become one of the project's more distinctive analytical components.

# 5. The KPI framework

Before building the report, I want to first establish teh project's initial KPI framework.

- **Core KPIs**

    KPI	                   | Business purpose
    -----------------------|-------------------------------------
    Total Sales	           | Overall revenue performance
    Total Orders	       | Transaction volume
    Units Sold	           | Product volume
    Average Order Value	   | Value per transaction
    Total Customers	       | Customer reach
    Returned Amount	       | Value lost through returns
    Return Rate	           | Return intensity
    YoY Sales Growth	   | Performance relative to prior year

- **Potential secondary KPIs:**

    - Sales per Store
    - Sales per Employee
    - Units per Order
    - Category Contribution %
    - Customer Contribution %
    - Ramadan Sales
    - Weekend Sales

Unfortunately, I can only finalize these after examining the actual columns.

# 6. An important modeling question

According to the author of the dataset being used in this project, `fact_order_details` is the source of truth for revenue-related calculations.

But we need to establish the exact relationship between:

`fact_orders` and `fact_order_details` tables

- For instance, in a case where an order, say Order 1001 has multiple products, as illustrated below:

        Order 1001
            ├── Product A
            ├── Product B
            └── Product C

- My assumption is that if `fact_orders` contains one row per order while `fact_order_details` contains one row per product line, then calculating revenue from both tables without understanding the grain could lead to double counting.

- Before embarking on creating teh DAX, this kind of modeling issue should be established during audit and not assumed.

