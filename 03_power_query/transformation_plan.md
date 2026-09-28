
<h1 align = "center"> 
Lotus Group Retail Analytics: Transformation Plan
</h1>

### Transformation Candidate Register

Here we are creating a transformation register


ID	|Table	|Issue	|Evidence	|Candidate action	|Status
----|----|----|----|----|----|
T01	|dim_customers	|`birth_date` stored as Text	|Actual type = Text	|Convert to Date	|Confirmed
T02	|dim_customers	|`phone` stored as Integer	|Actual type = Integer	|Convert to Text	|Confirmed
T03	|dim_customers	|Gender case inconsistency	|6 representations observed	|Standardize categories	|Confirmed
T04	|dim_customers	|Duplicate customer IDs	|3,050 rows vs 3,000 distinct IDs	|Investigate duplicate records	|Investigate
T05	|dim_customers	|Missing email	|13% empty	|Retain nulls / assess reporting need	|Investigate
T06	|dim_date	|Derived fields	|Need comparison with full_date	|Validate/recalculate if necessary	|Investigate
T07	|dim_products	|Numeric price fields	|Integer monetary fields	|Standardize numeric type	|Candidate
T08	|dim_products	|`unit_price_text`	|Text + numeric price fields	|Reconcile representations	|Investigate
T09	|dim_products	|Price/cost relationship	|Not yet tested	|Validate unit_price >= unit_cost	|Investigate
T10	|fact_order_details	|Revenue arithmetic	|Not yet tested	|Validate line revenue calculation	|Investigate
T11	|fact_order_details	|Cost arithmetic	|Not yet tested	|Validate line cost calculation	|Investigate
T12	|fact_orders	|Two source tables	|Same grain/process	|Append into consolidated fact	|Confirmed
T13	|fact_orders	|Employee IDs	|217 fact IDs vs 216 employees	|Identify orphan employee ID	|Investigate
T14	|fact_orders	|Order/date consistency	|Not yet tested	|Validate date_id against order_date	|Investigate
T15	|fact_returns	|Order relationship	|Not yet tested	|Validate return order_id against orders	|Investigate

