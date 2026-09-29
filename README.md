
<h1 align = "center"> 
Lotus Group Retail Analytics
<h3 align = "center">
An End-to-End Retail Business Intelligence Project Using Power BI
</h3>
</h1>

<p align = "center">
<img src = "Cover.png" width = "1000" height = "350">
</p>

# Repository Structure

    lotus-group-retail-bi/
    │
    ├── 01_data/
    │   ├── raw/
    │   └── README.md
    │
    ├── 02_documentation/
    │   ├── project_breakdown.md
    │   ├── business_requirements.md
    │   ├── data_dictionary.md
    │   ├── data_quality_assessment.md
    │   └── data_model.md
    │
    ├── 03_power_query/
    │   ├── transformation_notes.md
    │   ├── transformation_plan.md
    │   └── transformations_log.md
    │
    ├── 04_DAX/
    │   └── DAX_measures_catalog.md
    │
    ├── 05_powerbi/
    │   ├── lotus_group_retail.pbix
    │   └── screenshots/
    │
    ├── 06_analysis/
    │   └── business_insights.md
    │
    └── README.md

# Background
# Lotus Group Retail BI



## 1. Project Overview

**Lotus Group** is a fictional retail business operating across Egypt, with stores selling products across **Clothing and Electronics**.

This project develops an end-to-end Business Intelligence solution for Lotus Group using **Power BI**.

Rather than treating the project as a simple dashboard-building exercise, the objective is to demonstrate the complete analytical workflow involved in turning imperfect operational data into a structured reporting and decision-support solution.

The project covers:

* Data profiling and quality assessment
* Data cleaning and transformation using Power Query
* Fact and dimension table design
* Star-schema modeling
* Relationship design
* DAX measure development
* Interactive Power BI reporting
* Retail performance analysis
* Data-quality documentation
* Business insight generation

The underlying dataset intentionally contains realistic data-quality and modeling challenges. This provides an opportunity to demonstrate not only how to build a dashboard, but also how to **investigate, document, and make informed decisions about imperfect data**.

---

# 2. Business Objective

The primary objective is to develop an integrated retail BI solution that enables Lotus Group management to:

* Monitor overall sales and profitability
* Understand changes in sales performance over time
* Compare store and regional performance
* Identify important product and category drivers
* Understand customer purchasing patterns
* Examine employee-related sales activity
* Monitor returns
* Investigate seasonal purchasing patterns, including Ramadan
* Identify important data-quality limitations that may affect reporting

The project therefore combines **operational reporting, descriptive analytics, and data-quality assessment** within a single BI solution.

---

# 3. Key Business Questions

The analysis is designed around questions such as:

### Sales Performance

* How much revenue is Lotus Group generating?
* How are sales changing over time?
* How much gross profit is being generated?
* What is the gross margin?
* How large is the average order?

### Product Performance

* Which categories and subcategories generate the most sales?
* Which products contribute most to revenue?
* How many products are actively generating sales?
* How does selling price vary across products and categories?

### Store Performance

* Which stores generate the most revenue?
* How does performance differ across regions?
* Do different store types exhibit different sales patterns?

### Customer Performance

* How many customers are purchasing from Lotus Group?
* How much sales value is associated with the customer base?
* How does performance vary across customer segments such as loyalty tiers?

### Returns

* How many returns are recorded?
* What is the value of returned merchandise?
* What reasons are associated with returns?
* How do returns vary over time?

### Seasonality

* How do sales differ between Ramadan and non-Ramadan periods?
* How do weekend and weekday sales compare?
* Are there identifiable seasonal patterns in purchasing activity?

---

# 4. Dataset Structure

The final Power BI model contains **8 loaded tables**.

### Dimension tables

| Table           | Grain                     | Purpose                                 |
| --------------- | ------------------------- | --------------------------------------- |
| `dim_customers` | One row per customer      | Customer attributes                     |
| `dim_date`      | One row per calendar date | Time intelligence and calendar analysis |
| `dim_employees` | One row per employee      | Employee attributes                     |
| `dim_products`  | One row per product       | Product attributes                      |
| `dim_stores`    | One row per store         | Store and geographic attributes         |

### Fact tables

| Table                | Grain                          | Purpose                       |
| -------------------- | ------------------------------ | ----------------------------- |
| `fact_orders`        | One row per order              | Order-level sales activity    |
| `fact_order_details` | One row per order-product line | Product-level sales and cost  |
| `fact_returns`       | One row per return             | Returned merchandise activity |

The original order data was supplied in two separate tables:

* `fact_orders_2022_2023`
* `fact_orders_2024`

These were appended in Power Query to create the final `fact_orders` table.

The original source tables are retained as staging queries for transformation traceability but are not loaded into the final analytical model.

---

# 5. Data Quality Investigation

One of the main purposes of this project is to demonstrate that BI development begins **before the dashboard**.

The raw data was systematically profiled to investigate:

* Row and column counts
* Data types
* Missing values
* Duplicate records
* Unique values
* Date ranges
* Categorical consistency
* Key uniqueness
* Referential relationships between tables
* Derived date attributes

Several issues were identified and documented.

### Customer data

The raw `dim_customers` table contained **3,050 rows**, but only **3,000 distinct customers**.

Investigation showed that the duplicated records were exact matches. Therefore, the 50 duplicate rows were removed during Power Query transformation.

Other customer-level issues included:

* `birth_date` stored as text
* `phone` stored as an integer rather than text
* inconsistent capitalization in gender values
* missing email values

The missing email values were retained rather than artificially replaced because missing contact information is different from an unknown value.

### Employee attribution

The order data contains orders for which the employee ID was **not captured**.

This is treated as a **missing employee attribution issue**, rather than as evidence of invalid employee IDs.

The orders are retained in the analytical model so that missing employee information does not result in the loss of sales records.

### Date data

The date dimension was subjected to additional validation because it contains several derived calendar attributes.

Attributes including:

* day
* month
* year
* month name
* quarter
* quarter name
* day of week
* day name

were checked against the underlying date.

The dataset's calendar conventions were retained where they represented deliberate source conventions rather than clear errors.

The date table also contains business-specific attributes such as:

* weekend indicator
* Ramadan indicator

which support subsequent seasonal analysis.

---

# 6. Data Transformation

Power Query was used as the primary ETL layer.

The transformation process focused on making the data **analytically reliable without unnecessarily altering the source information**.

Key transformations included:

* Removing exact duplicate customer records
* Converting date fields to appropriate date types
* Converting identifier fields such as phone numbers to text
* Standardizing categorical values
* Converting monetary fields to Fixed Decimal Number
* Appending the 2022–2023 and 2024 order tables
* Standardizing data types across the final fact tables
* Retaining documented source attributes where no clear transformation was justified

An important principle throughout the transformation process was:

> **Do not "fix" data simply because it looks unusual. Investigate first, establish the business meaning, and transform only when there is sufficient justification.**

---

# 7. Data Model

The final model follows a dimensional modeling approach.

```text
                         dim_date
                            │
                            │
                            ▼
dim_customers ───────► fact_orders ◄────── dim_stores
                            │
                            │
                            ▼
                     fact_order_details
                            ▲
                            │
                      dim_products

                      dim_employees
                            │
                            ▼
                       fact_orders

dim_date ─────────────► fact_returns
```

The main relationships are:

* `dim_date` → `fact_orders`
* `dim_customers` → `fact_orders`
* `dim_stores` → `fact_orders`
* `dim_employees` → `fact_orders`
* `dim_products` → `fact_order_details`
* `fact_orders` → `fact_order_details`
* `dim_date` → `fact_returns`

The model uses **one-to-many relationships with single-direction filtering**.

The `fact_orders` → `fact_order_details` relationship reflects the difference in grain between order headers and product-level order lines.

---

# 8. Analytical Layer — DAX

The analytical layer is implemented using reusable DAX measures rather than relying heavily on calculated columns.

Core measures include:

### Sales & Profitability

* Total Sales
* Total Cost
* Gross Profit
* Gross Margin %

### Order Performance

* Total Orders
* Units Sold
* Average Order Value
* Units per Order

### Customer Performance

* Total Customers
* Sales per Customer

### Product Performance

* Products Sold
* Average Selling Price

### Returns

* Returned Amount
* Number of Returns
* Return Rate %

### Time Intelligence

* Sales YTD
* Sales Previous Year
* YoY Sales Growth %

### Seasonality

* Ramadan Sales
* Non-Ramadan Sales
* Weekend Sales
* Weekday Sales

### Data Quality

* Orders with Employee ID
* Orders without Employee ID
* Employee Attribution Rate %

The measure layer is designed to provide reusable calculations that can be applied across different dimensions and dashboard pages.

---

# 9. Planned Dashboard

The Power BI report is being developed around several analytical perspectives.

### Page 1 — Executive Overview

A management-level summary of:

* Revenue
* Orders
* Units
* Profitability
* Sales trends
* Store performance
* Product/category contribution
* Returns and seasonal indicators

### Page 2 — Sales & Product Performance

Focuses on:

* Category performance
* Subcategory performance
* Product contribution
* Units sold
* Selling price
* Revenue and profitability

### Page 3 — Store & Geographic Performance

Examines:

* Store sales
* Regional performance
* City performance
* Store type
* Store-level profitability

### Page 4 — Customer & Employee Analysis

Examines:

* Customer activity
* Loyalty tiers
* Customer contribution
* Employee-attributed orders
* Employee sales activity

### Page 5 — Returns & Seasonality

Examines:

* Return volume
* Return value
* Return reasons
* Ramadan vs non-Ramadan sales
* Weekend vs weekday patterns
* Trends over time

The final dashboard structure may evolve as the analysis reveals which questions are most useful.

---

# 10. Project Workflow

The project follows the following workflow:

```text
Raw Data
   │
   ▼
Data Profiling
   │
   ▼
Data Quality Assessment
   │
   ▼
Power Query Transformation
   │
   ▼
Data Model
   │
   ▼
Relationships
   │
   ▼
DAX Measures
   │
   ▼
Dashboard Development
   │
   ▼
Business Analysis
   │
   ▼
Documentation & Insights
```

This workflow is intentional.

The dashboard is the **final delivery layer**, rather than the starting point of the project.

---

# 11. Repository Structure

```text
lotus-group-retail-bi/
│
├── README.md
│
├── data/
│   ├── raw/
│   └── README.md
│
├── documentation/
│   ├── business_requirements.md
│   ├── data_dictionary.md
│   ├── data_quality_assessment.md
│   ├── data_model.md
│   └── dax_measure_catalog.md
│
├── power_query/
│   └── transformation_notes.md
│
├── sql/
│   └── analysis_queries.sql
│
├── powerbi/
│   ├── lotus_group_retail.pbix
│   └── screenshots/
│
└── analysis/
    └── business_insights.md
```

---

# 12. Tools & Technologies

| Tool             | Purpose                                      |
| ---------------- | -------------------------------------------- |
| **Power BI**     | Data modeling, DAX and dashboard development |
| **Power Query**  | Data profiling, transformation and ETL       |
| **DAX**          | Analytical measures and time intelligence    |
| **SQL**          | Supporting analytical queries                |
| **Git / GitHub** | Version control and portfolio documentation  |

---

# 13. What This Project Demonstrates

This project is intended to demonstrate practical capability across the BI lifecycle, including:

* Translating business questions into analytical requirements
* Working with imperfect operational data
* Conducting structured data-quality investigations
* Making defensible transformation decisions
* Building dimensional data models
* Designing relationships between fact and dimension tables
* Developing reusable DAX measures
* Applying time intelligence
* Designing management-oriented dashboards
* Communicating analytical findings
* Documenting assumptions and limitations

An important focus is **traceability**.

Where a transformation or modeling decision was made, the project aims to document:

1. What was observed
2. Why it mattered
3. What decision was made
4. How it was implemented
5. What effect it had on the analytical model

---

# 14. Project Status

**Current stage:** DAX measure development completed; dashboard development next.

### Completed

* [x] Business questions defined
* [x] Raw tables profiled
* [x] Data-quality issues investigated
* [x] Transformation decisions documented
* [x] Power Query transformation layer created
* [x] Final 8-table model loaded
* [x] Relationships established
* [x] Core DAX measure layer developed

### In Progress

* [ ] Executive dashboard
* [ ] Sales & product dashboard
* [ ] Store & geographic dashboard
* [ ] Customer & employee dashboard
* [ ] Returns & seasonality dashboard
* [ ] Business insights
* [ ] Final documentation
* [ ] Dashboard screenshots

---

# 15. Data Disclaimer

This project uses a **fictional/educational retail dataset** created for analytical and learning purposes.

The company, customers, employees, stores, transactions and associated business information should not be interpreted as representing actual Lotus Group operations or real individuals.

The dataset is intentionally structured to contain realistic data-quality challenges so that the project can demonstrate practical data preparation and BI development techniques.

---

## Author

**Vincent Okumu**

This repository documents an end-to-end BI project developed to demonstrate practical skills in data analysis, data preparation, dimensional modeling, Power BI, Power Query and DAX.












