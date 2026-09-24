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

    Column                  Expected        Actual      What we're testing

    employee_id             Text            Text        Employee identifier
    first_name              Text            Text        Employee name
    last_name               Text            Text        Employee name
    gender                  Text            Text        Employee category
    role                    Text            Text        Employee role
    store_id                Text            Integer     Store foreign key
    hire_date               Date            Date        Employment start date
    monthly_salary_egp      Decimal         Integer     Salary amount

#### Table size
    Table: dim_employees
    Rows: 216
    Columns: 8
    Expected grain: one row per employee

#### Profile employee_id
employee_id is the candidate primary key
- basic requirements are:
    - emplpoyee_id should not be null
    - employee_id should not contain errors
    - employee_id should be unique

Column quality

    empty - 0%
    error - 0%

Column Distribution

    distinct values - 216
    unique values - 216

#### Profile completeness

    Column              Empty  Errors  Nulls    Distinct    Unique

    employee_id           0%      0%      0%         216       216
    first_name            0%      0%      0%          65        10
    last_name             0%      0%      0%          34         2
    gender                0%      0%      0%           2         0
    role                  0%      0%      0%           5         0
    store_id              0%      0%      0%          15         0
    hire_date             0%      0%      0%         207       199
    monthly_salary_egp    0%      0%      0%         211       206

#### Categorical columns Profile
##### gender
Column distribution contains:

    Male
    Female

No inconsistency that can fragment analysis in gender

##### role
Column distribution contains:

    Cashier
    Sales Associate
    Store Manager
    Department Manager
    Senior Sales Associate
    
No inconsistency that can fragment analysis in role

#### Profile monthly_salary_egp
    Minimum: 3516
    Maximum: 15490

### Table 4: dim_products
- working assumption: one row represents one product

#### Data types Expected vs Actual

    Column              Expected        Actual      What we're testing

    product_id          Text            Text        product identifier
    product_name_raw    Text            Text        product name
    category            Text            Text        product category
    subcategory         Text            Text        product subcategory
    brand               Text            Text        product brand
    unit_price_text     Text            Text        product price category
    unit_price          Fixed decimal   Integer     product unit price
    unit_cost           Fixed decimal   Integer     
    stock_qty           Integer         Integer     product quantity
    is_active           Integer         Integer     product availability

#### Table size
    Table: dim_product
    Rows: 345
    Columns: 10
    Expected grain: one row per product

#### Profile product_id
product_id is the candidate primary key
- basic requirements are:
    - product_id should not be null
    - product_id should not contain errors
    - product_id should be unique

Column quality

    empty - 0%
    error - 0%

Column Distribution

    distinct values - 345
    unique values - 345

#### Profile completeness

    Column              Empty  Errors  Nulls    Distinct    Unique

    product_id             0%      0%     0%         345       345
    product_name_raw       0%      0%     0%         345       345
    category               0%      0%     0%           2         0
    subcategory            0%      0%     0%          15         0
    brand                  0%      0%     0%          27        17
    unit_price_text        0%      0%     0%          93        39
    unit_price             0%      0%     0%          93        39
    unit_cost              0%      0%     0%          66        28        
    stock_qty              0%      0%     0%         243       164    
    is active              0%      0%     0%           2         0

#### Categorical columns Profile
##### category
Column distribution contains:

    Clothing
    Electronics

No inconsistency that can fragment analysis by category

##### subcategory
Column distribution contains:

    Women wear
    Tops
    Bottoms
    Outwear
    Footwear
    Accessories
    Audio & wearables
    Mobile Phones
    TV & Screens
    Laptops
    Accessories & Storage
    Tablets
    Gaming
    Networking
    Printers
    
No inconsistency that can fragment analysis by subcategory

##### brand
- Column distribution shows unique identification of brand categories.

- No inconsistency that can fragment analysis by brand

### Table 5: dim_stores
- working assumption: one row represents one store

#### Data types Expecte vs Actual

    Columns         Expected        Actual      What we're measuring
    
    store_id        Integer         Integer     Store identifier
    store_name      Text            Text        Store name
    city            Text            Text        Store city location
    district        Text            Text        Store dictrict location
    region          Text            Text        Store regional location
    opening_year    Integer         Integer     Year store was openned
    size_sqm        Integer         Integer     Store size
    store_type      Text            Text        Store type

#### Table size
    Table: dim_stores
    Rows: 15
    Columns: 8
    Expected grain: one row per store

#### Profile product_id
store_id is the candidate primary key
- basic requirements are:
    - store_id should not be null
    - store_id should not contain errors
    - store_id should be unique

Column quality

    empty - 0%
    error - 0%

Column Distribution

    distinct values - 15
    unique values - 15

#### Profile completeness

    Column              Empty  Errors  Nulls    Distinct    Unique

    store_id               0%      0%     0%          15        15
    store_name             0%      0%     0%          15        15
    city                   0%      0%     0%          13        11
    district               0%      0%     0%          15        15
    region                 0%      0%     0%           7         2
    opening_year           0%      0%     0%           7         1
    size_sqm               0%      0%     0%          15        15
    store_type             0%      0%     0%           3         0        


#### Categorical columns Profile
##### city
- Column distribution shows unique identification of cities.

- No inconsistency that can fragment analysis by city

##### district
- Column distribution shows unique identification of districts.

- No inconsistency that can fragment analysis by district

#### region
- Column distribution shows unique identification of regins.

- No inconsistency that can fragment analysis by region

#### store_type
- Column distribution shows unique identification of store types.

- No inconsistency that can fragment analysis by store type
#### profile opening_year
    Minimum: 2016
    Maximum: 2022

### Table 6: fact_order_details
- Working assumption: one row represents one order-product line

#### Data types Ecpected vs Actual

    Columns             Expected        Actual          What we're measuring

    detail_id           Text            Text            Order detail identifier
    order_id            Text            Text            Order identifier
    product_id          Text            Text            Product identifier
    quantity            Integer         Integer         Order quantity
    unit_price          Integer         Integer         Order price
    discount_pct        Integer         Integer         Discount per order
    selling_price       Fixed decimal   Fixed decimal   Order selling price
    unit_cost           Fixed decimal   Integer         Unit cost per order detail
    line_total_revenue  Fixed decimal   Fixed decimal   Revenue per order
    line_total_cost     Fixed decimal   Integer         Order cost

#### Table size
    Table: fact_order_details
    Rows: 25099
    Columns: 10
    Expected grain: one row per order detail

#### Profile product_id
detail_id is the candidate primary key
- basic requirements are:
    - detail_id should not be null
    - detail_id should not contain errors
    - detail_id should be unique

Column quality

    empty - 0%
    error - 0%

Column Distribution

    distinct values - 25099
    unique values - 25099

#### Profile completeness

    Column              Empty  Errors  Nulls    Distinct    Unique

    detail_id              0%      0%     0%       25099     25099
    order_id               0%      0%     0%       12000      4843
    product_id             0%      0%     0%         345         0
    quantity               0%      0%     0%           4         0
    unit_price             0%      0%     0%          93         0
    discount_pct           0%      0%     0%           7         0
    selling_price          0%      0%     0%         537        17
    unit_cost              0%      0%     0%          66         0    
    line_total_revenue     0%      0%     0%        1239       206
    line_total_cost        0%      0%     0%         187         2



### Table 7: fact_orders_2022_2023
- Working assumption: one row represents one order

#### Data types Expected vs Actual

    Columns         Expected        Actual          What we're measuring

    order_id        Text            Text            Order identifier
    order_date      Date            Date            Date of order placement
    date_id         Integer         Integer         Date identifier
    customer_id     Text            Text            Customer identifier
    store_id        Integer         Integer         Store identifier
    employee_id     Text            Text            Employee identifier
    payment_method  Text            Text            Mode of customer payment
    order_status    Text            Text            Order status
    total_revenue   Fixed decimal   Fixed decimal   Revenue per order
    total_cost      Fixed decimal   Integer         Cost per order

#### Table size



### Table 8: fact_orders_2024


### Table 9: fact_returns










