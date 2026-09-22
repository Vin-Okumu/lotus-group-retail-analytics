<h1 align = "center"> 
Lotus Group Retail Analytics: Data Quality Assessment
</h1>

# Data Profiling

As already established, we have to audit the dataset first before embarking on transformations and later chart creation..

The principle here is we are not cleaning the dataset yet; we are simply trying to answer the question of what we have on our hands before establishing how we'll fix underlying issues. 

Our first deliverable will be the table below:

Our first deliverable is the following table:

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


