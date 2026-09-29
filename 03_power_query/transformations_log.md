
<h1 align = "center"> 
Lotus Group Retail Analytics: Transformations Log
</h1>

# Transformation Phase

We'll use a simple rule:

Fix confirmed structural/type issues, standardize obvious inconsistencies, preserve legitimate missing values, and avoid changing business figures unless we've established they're wrong.

## 1. dim_customers

Applied:

### Remove duplicates
Our table has:

    3,050 rows
    3,000 distinct customer_id
    50 duplicate records: duplicates confirmed as exact matches

Remove duplicates: In Power Query:

    Open dim_customers → Select all columns → Go to Home → Remove Rows → Remove Duplicates.

Should restore our intended grain as: One row = one customer


### Convert birth_date: Text → Date

In Power Query:

    Select birth_date → Transform → Data Type → Date

- If Power Query asks about locale and the values are formatted as something like: 12/25/1985

    - Choose the appropriate locale rather than allowing an incorrect interpretation.
    
### Convert phone: Integer → Text

    Transform → Data Type → Text

### gender: standardize 

male, MALE → Male; female, FEMALE → Female
    
    Transform → Format → Capitalize Each Word

### Email

Keep missing email as null
    
## 2. dim_products

Standardize monetary fields to fixed decimal

## 3. Append order tables

Create:

    Home → Append Queries → Append Queries as New

Select:

    fact_orders_2022_2023
    fact_orders_2024

Name of the resulting query: `fact_orders`

## Resulting model

                        dim_date
                           │
                           │
    dim_customers ───── fact_orders ───── dim_stores
                           │
                           │
                     dim_employees
                           │
                           │
                    fact_order_details
                           │
                           │
                      dim_products


    fact_orders ───── fact_returns

# Transformation register, final

    Table	            Transformation	                    Reason

    dim_customers	    Removed 50 exact duplicate rows	    Restore one-row-per-customer grain
    dim_customers	    birth_date Text → Date	            Correct semantic data type
    dim_customers	    phone Integer → Text	            Phone numbers are identifiers, not measures
    dim_customers	    Standardized gender	                Prevent category fragmentation
    dim_products	    Monetary fields → Fixed Decimal	    Consistent financial typing
    fact_orders	        Appended 2022–23 + 2024	            Create consolidated order fact
    fact_orders	        Revenue/cost → Fixed Decimal	    Consistent financial typing
    fact_order_details	Monetary fields → Fixed Decimal	    Consistent financial typing
    fact_returns	    return_amount → Fixed Decimal	    Consistent financial typing


# Star Schema Model
Now, having loaded the 8 tables, we can go ahead and build our Star Schema model as folows:

Let's build the model one relationship at a time, keeping it simple and avoiding unnecessary fact-to-fact complexity.

## The target model

Our model should ultimately look conceptually like this:

                             dim_date
                                │
                                │ 1 : *
                                ▼
    dim_customers ───────► fact_orders ◄────── dim_stores
           │                   │
           │                   │
           │                   ▼
           │            fact_order_details ◄──── dim_products
           │
           └─────────────── (via fact_orders)

                           ▲
                           │
                      dim_employees

    dim_date ─────────────► fact_returns

Althugh we have 8 tables, we only need 7 relationships for the basic model.

## We Create the dim_date → fact_orders relationship

In Model view:

Drag:

    dim_date[date_id]
            ↓
    fact_orders[date_id]

Power BI should show:

    Setting	                        Value
    Cardinality	                    One to many (1:*)
    Cross-filter                    direction	Single
    Make this relationship active	Yes

The 1 side is dim_date.

The * side is fact_orders.

Reason? dim_date contains one row per date, while many orders can occur on the same date.

    dim_date
    20240101 ───────────┐
                        ├── many orders
                        ├── many orders
                        └── many orders

## We create the dim_customers → fact_orders relationship

Create:

    dim_customers[customer_id]
            ↓
    fact_orders[customer_id]

Set:

    Setting	        Value
    Cardinality	    One to many (1:*)
    Cross-filter	Single
    Active	        Yes

Implication:

    Customer
       ↓
    Orders

So when we put Customer, City, Region, or Loyalty Tier into a visual, it can filter the order data.

Important: We removed the 50 exact duplicate customer records during Power Query transformation phase.

Therefore:

    dim_customers = 3,000 customers
    fact_orders   = 12,000 orders

and customer_id is now appropriately unique on the dimension side.

## We create the dim_stores → fact_orders relationship

Create:

    dim_stores[store_id]
            ↓
    fact_orders[store_id]

We use:

- *1: **
- Single
- Active

This allows things such as:

- Sales by store
- Sales by region
- Sales by city
- Sales by store type

to filter the orders.

## We create dim_employees → fact_orders relationship

Create:

    dim_employees[employee_id]
            ↓
    fact_orders[employee_id]

Again:

- *1: **
- Single
- Active

Remember we have 632 orders where the employee ID was not captured.

Therefore, conceptually:

    dim_employees
    216 employees
         │
         │
         ▼
    fact_orders
    12,000 orders
         │
         ├── orders with employee ID
         │
         └── 632 orders with missing employee ID

- Power BI may therefore show a (Blank) employee member when you analyze orders by employee.
    - So we don't need to create an "Unknown Employee" record.
    - The blank category actually communicates something useful:
        - Employee attribution was unavailable for these orders.

That's a data-quality finding we can potentially mention later in the documentation.

## We create dim_products → fact_order_details relationship

Now we move to the product-level fact.

Create:

    dim_products[product_id]
            ↓
    fact_order_details[product_id]

Set:

- *1: **
- Single
- Active

This relationship is particularly important for our product analysis.

It allows:

    Category → Subcategory → Brand → Product → Order details

So measures based on:

    fact_order_details[line_total_revenue]

can be broken down by product attributes.

## We create fact_orders → fact_order_details relationship

This is the one relationship that deserves a little explanation.

Create:

    fact_orders[order_id]
            ↓
    fact_order_details[order_id]

Set:

    Setting	        Value

    Cardinality	    One to many (1:*)
    Cross-filter	Single
    Active	        Yes

Reason: 
- Our grain is:
    fact_orders - One row = one order

    fact_order_details - One row = one product line within an order

Therefore:

    Order 1001
           │
           ├── Product A
           ├── Product B
           └── Product C

One order can have multiple detail records.

This relationship allows filters flowing into fact_orders to reach the order details.

For example:

    dim_date → fact_orders → fact_order_details

So a date selection can ultimately affect our detailed sales calculations.

One important modeling point

This is technically a fact-to-fact relationship, so it isn't as pure a star schema as the dimension-to-fact relationships.

However, given the structure of this dataset, it is a practical and relatively simple solution. 

- We don't need to introduce another bridge/order dimension just to make the diagram theoretically purer.

## dim_date → fact_returns relationship

Finally, we connect returns to the date dimension.

Unlike orders, fact_returns doesn't have date_id.
- It has:
    - return_date

So we create:

    dim_date[full_date]
            ↓
    fact_returns[return_date]

Set:

- *1: **
- Single
- Active

This lets us analyze:

- Returns by year
- Returns by month
- Returns by quarter
- Returns by day
- Ramadan returns
- Weekend returns

etc.

## Our final relationship list

We should now have exactly these 7 relationships:

    #	From	                    To	                            Cardinality
    1	dim_date[date_id]	        fact_orders[date_id]	        1:*
    2	dim_customers[customer_id]	fact_orders[customer_id]	    1:*
    3	dim_stores[store_id]	    fact_orders[store_id]	        1:*
    4	dim_employees[employee_id]	fact_orders[employee_id]	    1:*
    5	dim_products[product_id]	fact_order_details[product_id]	1:*
    6	fact_orders[order_id]	    fact_order_details[order_id]	1:*
    7	dim_date[full_date]	        fact_returns[return_date]	    1:*

All should be:

Single-direction + Active

## What the finished model should communicate

Think of the model in terms of business processes rather than simply tables:

**Sales process**
    Customers ──┐
    Stores ─────┤
    Employees ──┤
    Dates ──────┼──► Orders ──► Order Details ◄── Products
                │
                └───────────────────────────────
    Returns process
    Dates ─────────► Returns

This gives us a model where the dimensions describe who, what, where and when, while the fact tables contain the business events.
