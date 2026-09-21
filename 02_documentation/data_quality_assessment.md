<h1 align = "center"> 
Lotus Group Retail Analytics: Data Quality Assessment
</h1>

# Data Profiling

As already established, we have to audit the dataset first before embarking on transformations and later chart creation..

The principle here is we are not cleaning the dataset yet; we are simply trying to answer the question of what we have on our hands before establishing how we'll fix underlying issues. 

Our first deliverable will be the table below:

Our first deliverable is a table as follows:

Table	                | Type	    | Expected      | Role	| Rows	| Columns	| Primary Key	Grain
------------------------|-----------|---------------|-------|-------|-----------|----------------
dim_date	            | Dimension	| Date	        |  ?	|    ?	|    ?	    | One row/date
dim_stores	            | Dimension	| Store	        |  ?	|    ?	|    ?	    | One row/store
dim_customers	        | Dimension	| Customer      |  ?	|    ?	|    ?	    | One row/customer
dim_employees	        | Dimension	| Employee	    |  ?	|    ?	|    ?	    | One row/employee
dim_products	        | Dimension	| Product	    |  ?	|    ?	|    ?	    | One row/product
fact_orders_2022_2023	| Fact	    | Orders	    |  ?	|    ?	|    ?	    | ?
fact_orders_2024	    | Fact	    | Orders	    |  ?	|    ?	|    ?	    | ?
fact_order_details	    | Fact	    | Order lines	|  ?	|    ?	|    ?	    | ?
fact_returns	        | Fact	    | Returns	    |  ?	|    ?	|    ?	    | ?


