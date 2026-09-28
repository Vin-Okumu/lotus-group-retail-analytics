<h1 align = "center"> 
Lotus Group Retail Analytics: Transformation Notes
</h1>

Following the Data profiling done in the data quality assessment markdown, here are the identified transformation candidates.

    Priority	Table	        Column(s)	        Observed issue	Candidate transformation	Why
    High	    dim_customers	birth_date	        Stored as Text	Convert Text → Date	Enables age/date calculations and date validation
    High	    dim_customers	gender	            Male, male, MALE, etc.	Standardize case/category values	Prevents fragmented gender analysis
    High	    dim_customers	customer_id	        3,050 rows but only 3,000 distinct IDs	Investigate duplicates before deciding	Cannot safely remove records without understanding duplicates
    High	    dim_customers	phone	            Stored as Integer	Convert Integer → Text	Preserves leading zeros and treats phone numbers as identifiers
    Medium	    dim_customers	email	            13% missing	Decide whether to retain missing values	Missing email is a completeness issue, not necessarily something to replace
    High	    dim_employees	store_id	        Integer vs expected Text	Standardize FK data type	Necessary for reliable relationship matching
    Medium	    dim_employees	monthly_salary_egp	Integer vs expected Decimal	Consider Fixed Decimal	Appropriate representation for monetary values
    Medium	    dim_products	unit_price	        Integer vs expected Fixed Decimal	Convert to Fixed Decimal	Monetary field; improves semantic consistency
    Medium	    dim_products	unit_cost	        Integer vs expected Fixed Decimal	Convert to Fixed Decimal	Same reason
    Medium	    dim_products	unit_price_text	    Text representation of price	Parse/validate against unit_price	Potentially useful source field for transformation/audit
    Medium	    fact_order_details	unit_price	    Integer	Consider Fixed Decimal	Monetary field
    Medium	    fact_order_details	unit_cost	    Integer	Convert to Fixed Decimal	Monetary field
    Medium	    fact_order_details	line_total_cost	Integer	Convert to Fixed Decimal	Monetary field
    Medium	    fact_orders_2022_2023	total_cost	Integer	Convert to Fixed Decimal	Monetary field
    Medium	    fact_orders_2024	total_cost	    Integer	Convert to Fixed Decimal	Monetary field
    Medium	    fact_orders_2022_2023 + 2024	    Both tables	Same business grain, separate years	Append Queries	Create consolidated order fact
    High	    dim_date	        Derived columns	Need validation against full_date	Recalculate/correct only if validation fails	Ensures date dimension is internally consistent















