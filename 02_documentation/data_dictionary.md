<h1 align="center">Lotus Group Retail Analytics: Data Dictionary</h1>

# Purpose

This document describes every table and column in the Lotus Group dataset: what each field means, its data type in the final Power BI model, its valid values or range, and any data-quality notes.

- **Raw files:** [`01_data/raw/`](../01_data/raw/)
- **How tables relate:** [`data_model.md`](data_model.md)
- **Quality findings in detail:** [`data_quality_assessment.md`](data_quality_assessment.md)

**Conventions**
- **Key:** PK = primary key, FK = foreign key
- **Type:** the data type in the final Power BI model, after Power Query transformation
- **Currency:** all monetary values are in Egyptian pounds (EGP)

---

# Table overview

| Table | Type | Grain | Raw Rows | Model Rows |
|---|---|---|---|---|
| `dim_customers` | Dimension | One row per customer | 3,050 | 3,000 |
| `dim_date` | Dimension | One row per calendar day | 1,096 | 1,096 |
| `dim_employees` | Dimension | One row per employee | 216 | 216 |
| `dim_products` | Dimension | One row per product | 345 | 345 |
| `dim_stores` | Dimension | One row per store | 15 | 15 |
| `fact_orders` | Fact | One row per order | 7,942 + 4,058 | 12,000 |
| `fact_order_details` | Fact | One row per order line (order × product) | 25,099 | 25,099 |
| `fact_returns` | Fact | One row per return | 1,056 | 1,056 |

`fact_orders` is built by appending `fact_orders_2022_2023` and `fact_orders_2024`. The two source tables are kept as staging queries but aren't loaded into the model.

---

# Dimension tables

## dim_customers

Registered Lotus Group customers.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `customer_id` | PK | Text | Unique customer identifier | `CUS00001` – `CUS03000` | 50 exact duplicate rows removed |
| `full_name` | | Text | Customer's full name | 1,665 distinct names | Names repeat across customers; always use `customer_id` to identify a customer |
| `gender` | | Text | Customer gender | Male, Female | Raw data had 6 variants (e.g. `male`, `MALE`); standardised to 2 |
| `birth_date` | | Date | Date of birth | — | Stored as text in raw data, in two formats (`YYYY-MM-DD` and `DD/MM/YYYY`); converted to Date |
| `phone` | | Text | Mobile phone number | 11 digits, e.g. `01158132110` | Converted from Integer to Text to keep the leading zero; some raw values have surrounding spaces |
| `email` | | Text | Email address | — | 382 missing (12.5%); left as null on purpose |
| `city` | | Text | Customer's city | 13 cities | |
| `region` | | Text | Customer's region | 7 regions (see `dim_stores`) | |
| `loyalty_tier` | | Text | Loyalty programme tier | Bronze, Silver, Gold, Platinum | |
| `registration_date` | | Date | Date the customer registered | 2020-01-01 – 2023-12-28 | |

## dim_date

Calendar table covering every day in the analysis period. Used for all time intelligence.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `date_id` | PK | Whole Number | Date key in `YYYYMMDD` format | 20220101 – 20241231 | Joins to `fact_orders[date_id]` |
| `full_date` | | Date | Calendar date | 2022-01-01 – 2024-12-31 | Joins to `fact_returns[return_date]`; marked as the model's date table |
| `day` | | Whole Number | Day of the month | 1 – 31 | |
| `month` | | Whole Number | Month number | 1 – 12 | Sort column for `month_name` |
| `month_name` | | Text | Month name | January – December | |
| `quarter` | | Whole Number | Quarter number | 1 – 4 | |
| `quarter_name` | | Text | Quarter label | Q1 – Q4 | |
| `year` | | Whole Number | Calendar year | 2022, 2023, 2024 | |
| `day_of_week` | | Whole Number | Day number, Monday = 1 | 1 – 7 | Sort column for `day_name` |
| `day_name` | | Text | Day name | Monday – Sunday | |
| `is_weekend` | | Whole Number (flag) | 1 if the day falls on the Egyptian weekend | 0, 1 | Weekend = **Friday and Saturday**, following Egyptian convention |
| `week_of_year` | | Whole Number | ISO week number | 1 – 52 | Early-January days can belong to week 52 of the previous year (ISO convention) |
| `is_ramadan` | | Whole Number (flag) | 1 if the day falls in Ramadan | 0, 1 | 90 days: 2 Apr – 1 May 2022, 23 Mar – 21 Apr 2023, 11 Mar – 9 Apr 2024 |

## dim_employees

Store staff.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `employee_id` | PK | Text | Unique employee identifier | `EMP0001` – `EMP0216` | |
| `first_name` | | Text | First name | — | |
| `last_name` | | Text | Last name | — | |
| `gender` | | Text | Employee gender | Male, Female | |
| `role` | | Text | Job role | Cashier (80), Sales Associate (48), Store Manager (37), Department Manager (30), Senior Sales Associate (21) | |
| `store_id` | | Whole Number | Store the employee works at | 1 – 15 | Descriptive only; the employee's link to sales goes through `fact_orders` |
| `hire_date` | | Date | Date hired | 2016-04-10 – 2024-12-25 | |
| `monthly_salary_egp` | | Fixed Decimal | Monthly salary | EGP | Sensitive field; not used in the report |

## dim_products

The product catalogue.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `product_id` | PK | Text | Unique product identifier | `CLT0001` – `CLT0308` (clothing), `ELC0309` – `ELC0345` (electronics) | Prefix shows the category |
| `product_name_raw` | | Text | Product name; clothing names include colour and size | e.g. `T-Shirt \| Black \| S`, `iPhone 14 128GB` | Clothing names follow the pattern *Item \| Colour \| Size* |
| `category` | | Text | Product category | Clothing, Electronics | |
| `subcategory` | | Text | Product subcategory | **Clothing:** Tops, Bottoms, Outerwear, Footwear, Accessories, Women Wear · **Electronics:** Mobile Phones, Laptops, TVs & Screens, Tablets, Gaming, Audio & Wearables, Accessories & Storage, Networking, Printers | 15 subcategories |
| `brand` | | Text | Brand | 27 brands | |
| `unit_price_text` | | Text | List price as displayed text | e.g. `EGP 90` | Duplicate of `unit_price`; not used in analysis |
| `unit_price` | | Fixed Decimal | List price per unit | EGP | |
| `unit_cost` | | Fixed Decimal | Cost per unit | EGP | |
| `stock_qty` | | Whole Number | Units currently in stock | — | Snapshot value; no stock history |
| `is_active` | | Whole Number (flag) | 1 if the product is currently sold | 0 (26 products), 1 (319) | Inactive products still appear in past sales |

## dim_stores

Lotus Group store locations.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `store_id` | PK | Whole Number | Unique store identifier | 1 – 15 | |
| `store_name` | | Text | Store name | e.g. Lotus Cairo Festival City | |
| `city` | | Text | City | 13 cities | Cairo and Alexandria each have 2 stores |
| `district` | | Text | District or neighbourhood | 15 districts | |
| `region` | | Text | Region | Greater Cairo, Alexandria, Nile Delta, Canal Zone, Upper Egypt, Red Sea, South Sinai | |
| `opening_year` | | Whole Number | Year the store opened | 2016 – 2022 | |
| `size_sqm` | | Whole Number | Floor area in square metres | 140 – 420 | |
| `store_type` | | Text | Store format | Flagship (3), Standard (5), Small (7) | |

---

# Fact tables

## fact_orders

One row per customer order (the order header). Built by appending `fact_orders_2022_2023` and `fact_orders_2024`.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `order_id` | PK | Text | Unique order identifier | `ORD0000001` – `ORD0012000` | IDs aren't in date order |
| `order_date` | | Date | Date the order was placed | 2022-01-01 – 2024-12-31 | |
| `date_id` | FK → `dim_date` | Whole Number | Order date key | `YYYYMMDD` | |
| `customer_id` | FK → `dim_customers` | Text | Customer who placed the order | | Every ID matches `dim_customers` |
| `store_id` | FK → `dim_stores` | Whole Number | Store where the order was placed | 1 – 15 | |
| `employee_id` | FK → `dim_employees` | Text | Employee who processed the order | | **632 nulls (5.3%)**: employee not captured. Orders are kept; see the Employee Attribution Rate % measure |
| `payment_method` | | Text | How the customer paid | Cash, Credit Card, Debit Card, Fawry, InstaPay, Vodafone Cash | About 18% of raw values have leading or trailing spaces, which creates 12 variants instead of 6. Apply **Trim** in Power Query before using this field in a slicer |
| `order_status` | | Text | Final order status | Completed (10,442), Returned (1,063), Cancelled (495) | Total Sales includes all statuses |
| `total_revenue` | | Fixed Decimal | Order revenue after discounts | EGP | Equals the sum of the order's `line_total_revenue` in every order |
| `total_cost` | | Fixed Decimal | Order cost | EGP | Equals the sum of the order's `line_total_cost` |

## fact_order_details

One row per product on an order. This is the **source of truth for sales, cost and units**.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `detail_id` | PK | Text | Unique line identifier | `DTL0000001` – `DTL0025099` | |
| `order_id` | FK → `fact_orders` | Text | Parent order | | About 2.1 lines per order on average |
| `product_id` | FK → `dim_products` | Text | Product sold | | Every ID matches `dim_products` |
| `quantity` | | Whole Number | Units sold on this line | 1 – 4 | |
| `unit_price` | | Fixed Decimal | List price at time of sale | EGP | |
| `discount_pct` | | Whole Number | Discount applied, in percent | 0, 5, 10, 15, 20, 25, 30 | Average 7.9% |
| `selling_price` | | Fixed Decimal | Price per unit after discount | EGP | `unit_price × (1 − discount_pct / 100)` |
| `unit_cost` | | Fixed Decimal | Cost per unit | EGP | |
| `line_total_revenue` | | Fixed Decimal | Line revenue | EGP | `selling_price × quantity`; used by **Total Sales** |
| `line_total_cost` | | Fixed Decimal | Line cost | EGP | `unit_cost × quantity`; used by **Total Cost** |

## fact_returns

One row per returned order.

| Column | Key | Type | Description | Values / Range | Notes |
|---|---|---|---|---|---|
| `return_id` | PK | Text | Unique return identifier | `RET00001` – `RET01056` | |
| `order_id` | | Text | Order that was returned | | One return per order; every ID matches `fact_orders`. Not used as a relationship in the current model |
| `return_date` | FK → `dim_date[full_date]` | Date | Date the return was logged | 2022-01-08 – 2024-12-31 | |
| `return_reason` | | Text | Customer's reason for returning | Duplicate Order, Changed Mind, Quality Issue, Wrong Item Delivered, Size Issue, Defective Product | |
| `return_amount` | | Fixed Decimal | Value returned | EGP | Used by **Returned Amount** |
| `refund_method` | | Text | How the refund was paid | Cash, Credit Card, Debit Card, Fawry, InstaPay, Vodafone Cash | |
| `return_status` | | Text | Outcome of the return | Refunded (77.6%), Pending (14.2%), Rejected (8.2%) | |

---

# Business glossary

| Term | Definition |
|---|---|
| **Total Sales** | Sum of `line_total_revenue`. Gross sales after line discounts, before returns |
| **Gross Profit** | Total Sales − Total Cost |
| **Gross Margin %** | Gross Profit ÷ Total Sales |
| **Average Order Value (AOV)** | Total Sales ÷ number of distinct orders |
| **Return Rate %** | Number of returns ÷ total orders |
| **Employee Attribution Rate %** | Orders with an `employee_id` ÷ total orders |
| **Ramadan Sales** | Sales on days where `is_ramadan = 1` |
| **Weekend** | Friday and Saturday (Egyptian working week) |
| **Fawry / InstaPay / Vodafone Cash** | Egyptian electronic payment services |
