<h1 align = "center"> 
Lotus Group Retail Analytics: Data Quality Assessment
</h1>

# Data Profiling

As already established, we have to audit the dataset first before embarking on transformations and later chart creation..

The principle here is we are not cleaning the dataset yet; we are simply trying to answer the question of what we have on our hands before establishing how we'll fix underlying issues. 

Our first deliverable will be the table below:

Our first deliverable is as follows:

    Table	              Type       Expected Role      Rows        Columns    Primary Key 	  Grain
   
    dim_date	          Dimension  Date	            1096	         13    date_id          One row/date
    dim_stores	          Dimension  Store	              15	          8    store_id	        One row/store
    dim_customers	      Dimension  Customer           3000	         10    customer_id      One row/customer
    dim_employees	      Dimension  Employee	         216	          8    employee_id      One row/employee
    dim_products	      Dimension  Product	         345	         10    product_id       One row/product
    fact_orders_2022_2023 Fact	     Orders	            7942             10    order_id         One row/ order
    fact_orders_2024	  Fact	     Orders	            4058	         10    order_id         One row/ order
    fact_order_details	  Fact	     Order lines	   25099             10    detail_id        One row/ order-product line
    fact_returns	      Fact	     Returns	        1056              7    return_id        One row/ return

## Table Profiling Quality Log

### Table1: dim_customers
- Workng assumption: one row represents one customer

#### Data types: Expected vs Actual

    Column	            Expected type  Actual type  Analytical impact
    
    customer_id	           Text           Text     
    full_name	           Text           Text     
    gender	               Text           Text     
    birth_date	           Date           Text      Prevents reliable date calculations
    phone	               Text           Integer   potential loss of leading zeros
    email	               Text           Text     
    city	               Text           Text     
    region	               Text           Text     
    loyalty_tier	       Text           Text     
    registration_date	   Date           Date     

#### Table size
    Table: dim_customers
    Rows: 3050
    Columns: 10
    Expected grain: one row per customer

##### profile - customer_id
    customer_id: should not be null
    customer_id: should not be duplicated
    customer_id: should uniquely identify a customer

###### Duplicates
    customer_id
    Nulls: 0
    Distinct values: 3000
    Unique values: 2950

Violates expected dimension grain

#### Missing values
- Here we are going column by column and confirming column quality.

Below is our resultant profile for dim_customers table

    Column	               Empty?	Errors? 
    
    customer_id			      0%      0%            
    full_name			      0%      0%            
    gender			          0%      0%            
    birth_date			      0%      0%            
    phone			          0%      0%            
    email			         13%      0%            
    city			          0%      0%            
    region			          0%      0%            
    loyalty_tier			  0%      0%            
    registration_date         0%      0%            

#### Categorical columns Profile
##### gender
Column distribution contains:

    Male
    Female
    male
    female
    MALE
    FEMALE

Case inconsistency can fragment gender analysis 

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
- No whitespace realized in categorical or other text fields

#### Date profile
##### birth_date profile
- Stored as text

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

#### Transformation candidates
- Convert `birth_date` to Date after format validation
- Convert `phone` to Text
- Standardize case in `gender`
- Investigate duplicate `customer_id` records 
- Investigate repeated phone numbers
- Determine treatment of missing email values

### Table 2: dim_date
- Working assumption: One row represents a unique calendar date

#### Data Types: Expected vs Actual
 
    Column          Expected type   Actual    What we're testing
    
    date_id         Integer         Integer   Date key format
    full_date       Date            Date      Actual calendar date
    day             Integer         Integer   Day of month
    month           Integer         Integer   Month number
    month_name      Text            Text      Month label
    quarter         Integer         Integer   Quarter number
    quarter_name    Text            Text      Quarter label
    year            Integer         Integer   Calendar year
    day_of_week     Integer         Integer   Day-of-week number
    day_name        Text            Text      Day label
    is_weekend      Integer         Integer   Weekend indicator
    week_of_year    Integer         Integer   Week number
    is_ramadan      Integer         Integer   Ramadan indicator

#### Table size
    Table: dim_date
    Rows: 1096
    Columns: 13
    Expected grain: one row per calendar date

#### Profile date_id
date_id is the candidate primary key
- basic requirements are:
    - date_id should not be null
    - date_id should not contain errors
    - date_id should be unique

Column quality

    empty - 0%
    error - 0%

Column Distribution

    distinct values - 1096
    unique values - 1096

##### Findings
    date_id
    Nulls: 0
    Distinct: 1096
    Unique: 1096

#### Profile full_date
    full_date
    Nulls: 0
    Distinct: 1096
    Unique: 1096

#### Validate columns
    
    Column            Nulls    Error    Empty    Valid    Invalid  Distinct     Unique
    
    date_id              0%       0%       0%     100%         0%      1096       1096
    full_date            0%       0%       0%     100%         0%      1096       1096
    day                  0%       0%       0%     100%         0%        31          0
    month                0%       0%       0%     100%         0%        12          0
    year                 0%       0%       0%     100%         0%         3          0
    month_name           0%       0%       0%     100%         0%        12          0
    quarter              0%       0%       0%     100%         0%         4          0
    quarter_name         0%       0%       0%     100%         0%         4          0
    day_name             0%       0%       0%     100%         0%         7          0
    week_of_year         0%       0%       0%     100%         0%        52          0
    is_weekend           0%       0%       0%     100%         0%         2          0

### Table 3: dim_employees
- Working assumption: one row represents one employee

#### Data types Expected vs Actual

    Column                  Expected        Actual

    employee_id             Text            Text
    first_name              Text            Text
    last_name               Text            Text
    gender                  Text            Text
    role                    Text            Text
    store_id                Text            Integer
    hire_date               Date            Date
    monthly_salary_egp      Decimal         Integer

























