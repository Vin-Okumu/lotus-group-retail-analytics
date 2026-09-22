<h1 align = "center"> 
Lotus Group Retail Analytics: Data Quality Assessment
</h1>

# Data Profiling

As already established, we have to audit the dataset first before embarking on transformations and later chart creation..

The principle here is we are not cleaning the dataset yet; we are simply trying to answer the question of what we have on our hands before establishing how we'll fix underlying issues. 

Our first deliverable will be the table below:

Our first deliverable is as follows:

Table	             | Type	     | Expected Role| Rows	| Columns | Primary Key |	Grain
---------------------|-----------|--------------|-------|---------|-------------|---------------------
dim_date	         | Dimension | Date	        |  1096	|      13 |date_id      | One row/date
dim_stores	         | Dimension | Store	    |    15	|       8 |store_id	    | One row/store
dim_customers	     | Dimension | Customer     |  3000	|      10 |customer_id  | One row/customer
dim_employees	     | Dimension | Employee	    |   216	|       8 |employee_id  | One row/employee
dim_products	     | Dimension | Product	    |   345	|      10 |product_id   | One row/product
fact_orders_2022_2023| Fact	     | Orders	    |  7942 |      10 |order_id     | One row/ order
fact_orders_2024	 | Fact	     | Orders	    |  4058	|      10 |order_id     | One row/ order
fact_order_details	 | Fact	     | Order lines	| 25099 |      10 |detail_id    | One row/ order-product line
fact_returns	     | Fact	     | Returns	    |  1056 |       7 |return_id    | One row/ return

## Table Profiling

### Table1: dim_customers
- Our workng assumption is that one row represents one customer

- Expected vs actual data types

Column	            |Expected type  |Actual type
--------------------|-------------- |------------
customer_id	        |   Text        |   Text
full_name	        |   Text        |   Text
gender	            |   Text        |   Text
birth_date	        |   Date        |   Text
phone	            |   Text        |   Integer
email	            |   Text        |   Text
city	            |   Text        |   Text
region	            |   Text        |   Text
loyalty_tier	    |   Text        |   Text
registration_date	|   Date        |   Date

#### Table size
Table: dim_customers
Rows: 3050
Columns: 10
Expected grain: customer_id

##### profile - customer_id
    customer_id: should not be null
    customer_id: should not be duplicated
    customer_id: should uniquely identify a customer

###### Duplicates
customer_id
Nulls: 0
Distinct values: 3000
Unique values: 2950
Duplicates: 50

#### Missing values

- Here we want to go column by column and confirm column quality.
- Below is our resultant profile for dim_customers table

Column	            |   Empty?|	Errors?| 
--------------------|---------|--------|
customer_id			|      0% |     0% |           
full_name			|      0% |     0% |           
gender			    |      0% |     0% |           
birth_date			|      0% |     0% |           
phone			    |      0% |     0% |           
email			    |     13% |     0% |           
city			    |      0% |     0% |           
region			    |      0% |     0% |           
loyalty_tier		|	   0% |     0% |           
registration_date   |      0% |     0% |           

#### Categorical columns Profile
##### gender
Column distribution contains:

    Male
    Female
    male
    female
    MALE
    FEMALE

##### city
- Column distribution shows no variation in city names across records

- Each city is captured consistently across respective records

##### region
- column distribution shows no variation in region names

- Each region name is captured consistently across respective records

##### loyalty_tier
- Column distribution shows no variation in loyalty_tier names across records

- Each loyalty_tier is captured consistently across respective records

#### Whitespace profile
- 

#### Date profile
##### birth_date profile

Minimum - 01/01/1984

Maximum - 28/12/1995

- No suspicious dates recorded
    - No future dates 
    - No extremely old dates
    - No invalid dates

#### registration_date
Minimum - 01/01/2020

Maximum - 12/28/2023

- No suspicous dates recorded
    - No future dates-
    - No extremely old dates

#### Phone profile
- Phone numbers stored as integer instead of text/string

- No missing phone numbers 
    - Total count - 3050
    - Unique count - 2950
    - distinct - 3000

### Table 2: dim_date































