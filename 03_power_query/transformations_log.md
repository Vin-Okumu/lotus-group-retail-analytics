
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


