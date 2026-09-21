<h1 align = "center"> 
Lotus Group Retail Analytics
</h1>

## 1. Intro & Background
The dataset used in this project,  **The Lotus Group Retail Datset** is an educational dataset designed to simulate a real-world retail business operating across Egypt.

Instead of providing perfectly clean data, this dataset intentionally contains common data quality issues found in real business environments, hence a perfect dataset to practice the complete Business Intelligence workflow—from data cleaning to dashboard development.

The dataset follows a Star Schema architecture and is suitable for hands-on practicing of:

  - Data Warehousing
  - Star Schema Modeling
  - Power BI
  - Power Query
  - SQL
  - DAX
  - ETL
  - Business Intelligence, and
  - Data Modeling

For this project, my interest is to structure it around the following fictional business questions:
   - How is Lotus Group's retail business performing across stores, products, customers, employees, and time, and 
   - What operational factors are driving sales and returns?

The overall aim is to get enough traction to demonstrate the process:

    Raw data → ETL → data quality → dimensional modeling → DAX → analysis → visualization → business recommendations

# 2. The Project Phases

For a fully documented portfolio project, the whole idea here is to work through the project in the following sequence.

### Phase 1 — Business understanding

- Before touching Power Query, I want to establish:

    - Who is Lotus Group?
    - What does the business sell?
    - What constitutes a sale?
    - What constitutes a return?
    - What questions should management be able to answer?
    - What KPIs matter?

- Deliverable: Business Requirements Document.

### Phase 2 — Raw-data audit

- The project involves nine data files that I'll need to load and systematically profile.

- Here, my intention is to examine:

    - row counts
    - column counts
    - data types
    - missing values
    - duplicate records
    - unique values
    - invalid values
    - inconsistent text
    - date ranges
    - numerical ranges
    - referential integrity
    - potential primary keys
    - potential foreign keys

- This is particularly important because the dataset intentionally contains quality problems.

- Deliverable: Data Quality Assessment.

### Phase 3 — Power Query / ETL

- Here, my interest is to demonstrate actual BI engineering rather than merely importing tables. For instance:

    - I'll need to append `fact_orders_2022_2023` to `fact_orders_2024` using `Append Queries` to come up with a complete table that I'll then call `fact_orders`

    - And potentially join `fact_returns` table to `customer/order information` using `Merge Queries` to come up with a table I can name `return analysis dataset`

- It'll be crucial to document every transformation. For instance:

Raw column → Trim whitespace → Standardize text → Replace invalid values → Correct data type → Handle missing values → Validate → Load

- Deliverable: documented Power Query transformation pipeline.

### Phase 4 — Star-schema modeling

- Itake this as one of the most important parts of the project.

- It'll be crucial to establish the grain of every table first. For example:

    - `fact_order_details`

        - Potential grain:

            - One row = one product line within one sales order.

                - This immediately tells us what the table can legitimately answer.

                - For instance:
                    - An order, say Order 1001 may have Product A, Product B and Product C

                        - That's three rows in the fact table but one order.

                - This distinction is particularly important for when I'll be writing DAX.

- We'll similarly establish the grain of:

    - `fact_orders`
    - `fact_returns`
    - `dim_products`
    - `dim_customers`
    - `dim_stores`
    - `dim_employees`
    - `dim_date`

- After which we'll identify:

    - PKs
    - FKs
    - cardinality
    - filter direction
    - active/inactive relationships
    - dimension-to-fact relationships

### Phase 5 — DAX layer

- Once we establish that the model is correct, we'll create a proper measure layer.

- Rather than creating dozens of random measures, I'm interested in organizing them into business categories.

    - Sales
    - Total Sales
    - Total Orders
    - Total Units
    - Average Order Value
    - Average Selling Price
    - Customers
    - Total Customers
    - Active Customers
    - Sales per Customer
    - Products
    - Product Sales
    - Product Units Sold
    - Product Contribution %
    - Returns
    - Returned Amount
    - Return Rate
    - Net Sales
    - Time
    - Sales LY
    - YoY Sales Growth
    - MTD Sales
    - YTD Sales
    - Operations
    - Sales per Employee
    - Sales per Store

- My aim with this is to demonstrate my understanding of why a measure is needed and what business question it answers.

### Phase 6 — The Ramadan dimension
##### A particularly interesting aspect of the entire project

- This is one feature I intend to deliberately exploit rather than treat as another column. And why is that?

    - The dataset represents Egypt, so I believe I can investigate something like:

        - Does retail behavior differ during Ramadan?

            - And report on the business' aspect, for example:

                - Ramadan Sales  vs Non-Ramadan Sales

                - And potentially examine:

                    - sales
                    - orders
                    - average order value
                    - product categories
                    - stores
                    - governorates
                    - return behavior

    - This should give the dashboard a genuine analytical story rather than just:

        - "Sales were $X and Cairo had the highest sales."

### Phase 7 — Business analysis

- I want to structure the analytical questions into the following themes:

    - Executive performance
    - How much revenue was generated?
    - How many orders were processed?
    - How many units were sold?
    - How is performance changing over time?
    - What is the YoY growth rate?
    - Store performance
    - Which stores generate the most sales?
    - How does performance differ by store type?
    - How does performance vary across governorates?
    - Are high-sales stores also high-volume stores?
    - Product performance
    - Clothing vs Electronics
    - Top products
    - Bottom products
    - Product contribution to revenue
    - Units vs revenue
    - Customer performance
    - Customer concentration
    - Repeat purchasing
    - Customer sales distribution
    - Geographic customer patterns
    - Employee performance
    - Sales per employee
    - Orders handled
    - Store-level employee performance
    - Returns
    - Return rate
    - Return amount
    - Return reasons
    - Products with high return rates
    - Stores with unusual return behavior
    - Seasonality
    - Monthly trends
    - Quarter trends
    - Weekend vs weekday
    - Ramadan vs non-Ramadan

- I want to believe this is enough analytical depth for my portfolio project without turning it into an unnecessarily complicated data science project.

### Phase 8 — Dashboard structure

- I intend to report the findings in 4–5 dashboard pages; it would not be aesthetically pleasing to put everything on one page. 

- I want to structure the pages as follows:

    - **Page 1 — Executive Overview**

        - A management-level dashboard with:

            - Possible KPIs:

                - Total Sales
                - Total Orders
                - Units Sold
                - Average Order Value
                - Return Rate
                - YoY Growth

            - Then:

                - monthly sales trend
                - sales by region
                - sales by category
                - top stores/products
    - ***Page 2 — Sales & Product Performance**

        - Focus to be on:

            - product category
            - product
            - units
            - revenue
            - AOV
            - contribution %
    - **Page 3 — Store & Geographic Performance**

        - Focus to be on:

            - governorate
            - region
            - store type
            - store
            - sales
            - orders
            - sales per employee

    - **Page 4 — Customer & Employee Analysis**

        - Here I want to explore:

            - customer purchasing
            - customer concentration
            - employee performance
            - store staffing
            - sales per employee
    - **Page 5 — Returns & Seasonality**

        - Here I'm interested in including:

            - return rate
            - return amount
            - return reasons
            - products generating returns
            - Ramadan vs non-Ramadan sales
            - monthly/seasonal patterns

