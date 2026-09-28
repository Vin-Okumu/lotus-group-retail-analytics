<h1 align = "center"> 
Lotus Group Retail Analytics: Transformation Notes
</h1>

Following the Data profiling done in the data quality assessment markdown, here are the identified transformation candidates.

Priority    |Table  |Column(s)  |Observed issue |Candidate transformation   |Why
----|----|----|----|----|----|
High	|`dim_customers`  |birth_date |Stored as Text |Convert Text → Date |Enables age/date calculations and date validation
High    |`dim_customers`  |gender |Male, male, MALE, etc. |Standardize case/category values   |Prevents fragmented gender analysis
High    |`dim_customers`  |customer_id    |3,050 rows but only 3,000 distinct IDs |Investigate duplicates before deciding |Cannot safely remove records without understanding duplicates
High    |`dim_customers`  |phone  |Stored as Integer  |Convert Integer → Text |Preserves leading zeros and treats phone numbers as identifiers
Medium  |`dim_customers`  |email  |13% missing    |Decide whether to retain missing values    |Missing email is a completeness issue, not necessarily something to replace
High    |`dim_employees`  |store_id   |Integer vs expected Text   |Standardize FK data type   |Necessary for reliable relationship matching
Medium  |`dim_employees`  |monthly_salary_egp |Integer vs expected Decimal    |Consider Fixed Decimal |Appropriate representation for monetary values
Medium  |`dim_products`   |unit_price |Integer vs expected Fixed Decimal  |Convert to Fixed Decimal   |Monetary field; improves semantic consistency
Medium  |`dim_products`   |unit_cost  |Integer vs expected Fixed Decimal  |Convert to Fixed Decimal   |Same reason
Medium  |`dim_products`   |unit_price_text    |Text representation of price   |Parse/validate against unit_price  |Potentially useful source field for transformation/audit
Medium  |`fact_order_details` |unit_price |Integer    |Consider Fixed Decimal |Monetary field
Medium  |`fact_order_details` |unit_cost  |Integer    |Convert to Fixed Decimal   |Monetary field
Medium  |`fact_order_details` |line_total_cost    |Integer    |Convert to Fixed Decimal   |Monetary field
Medium  |`fact_orders_2022_2023`  |total_cost |Integer |Convert to Fixed Decimal   |Monetary field
Medium  |`fact_orders_2024`   |total_cost |Integer    |Convert to Fixed Decimal   |Monetary field
Medium  |`fact_orders_2022_2023` + `2024`   |Both tables    |Same business grain, separate years    |Append Queries  |Create consolidated order fact
High    |`dim_date`   |Derived columns    |Need validation against full_date  |Recalculate/correct only if validation fails   |Ensures date dimension is internally consistent

### `dim_customers` — the most important transformation area

   - birth_date → Date
    - This is a fairly clear transformation.

        - Currently:
            - birth_date = Text
        - Expected:
            - birth_date = Date

        - Candidate transformation:
            - Convert birth_date from Text to Date using the correct source format.
            - But before converting, inspect the actual text representation.
                - For example, determine whether values look like:

                    - 01/15/1990
                    - 15/01/1990
                    - 1990-01-15

The format matters because Power Query can interpret ambiguous dates incorrectly.

So this is a definite transformation candidate, but we'll have to verify the format before implementing it.

### phone → Text

This is another strong candidate.
- Current: Integer
- Target: Text

This isn't really a mathematical field but an identifier.

- For example:
    - 0712345678 should remain: "0712345678" rather than becoming: 712345678
    - So I'd classify this as:
        - High-priority type correction.

### gender standardization

We have:

- Male
- Female
- male
- female
- MALE
- FEMALE

This should eventually become something like:

- Male
- Female

So we'll use a transformation like:
- Trim → Clean → Proper Case or a more explicit mapping.
- We'll document the transformation as:

    - Standardize categorical values to a consistent case after confirming that the observed variants represent the same business categories.

        - This should demonstrate we're not blindly applying Text.Proper.

### customer_id duplicates — we are NOT transforming yet

This is probably the most important distinction.

- We have:

    - 3,050 rows
    - 3,000 distinct customer IDs
    - 2,950 unique customer IDs

- A tempting transformation would be:

    - Remove duplicates based on customer_id.

        - But we're not doing that yet.

            - We will have to know what the duplicates represent.

            - We need to inspect them further first.

For example:

    Scenario 1
    C001 | John Smith | Male | ... | john@email.com
    C001 | John Smith | Male | ... | john@email.com

This would be an exact duplicate.

    Scenario 2
    C001 | John Smith | Male | ... | john@email.com
    C001 | John Smith | Male | ... | john2@email.com

That's a conflicting customer record.

    Scenario 3
    C001 | John Smith | Male | ... | 0712345678
    C001 | John Smith | Male | ... | 0723456789

Potentially a legitimate attribute change or data-quality problem.

Therefore:

 - Duplicate customer IDs are an investigation candidate, not yet a transformation candidate.
 - This should be one of our first transformation-stage investigations.

### email — don't replace missing values

We have:
- 13% missing email values.
- We would not do something like:

    - null → "Unknown"

- unless there's a specific reporting requirement.

Reason?

- Because null contains useful information:
    - We don't have an email address.
    - Replacing it with "Unknown" changes the semantic meaning and can interfere with missingness analysis.

    - So our transformation candidate is actually:
        - Retain missing values but document the completeness issue.
        - We could potentially create an analytical flag later:
            - Has Email = Yes/No
        - if customer-contact analysis requires it.

### dim_employees

- store_id
    - Current: Integer
    - Expected: Text

This is a transformation candidate, but we'll first need to check the corresponding `dim_stores`.`store_id` type.

Our current inventory says:

- `dim_employees`.`store_id` → Integer
- `dim_stores`.`store_id` → Integer

    - If both are actually Integer, then there's no relationship problem caused by type mismatch, even though our original expected type said Text.

        - A good example of why the expected-vs-actual comparison shouldn't automatically generate transformations.

So we're currently marking this:

   - Review, not automatically transform.

- monthly_salary_egp

    - Current: Integer
    - Expected: Decimal

I wouldn't call this a serious data-quality issue.

If salaries are all whole EGP amounts, Integer is mathematically adequate.

However, for semantic consistency with monetary measures, we would need to standardize monetary fields to Fixed Decimal.

we'll classify this as:

- Low/Medium-priority type standardization.

## dim_products

We have three areas worth investigating.

 - I. unit_price
    - Current: Integer

Since this is money, we may need to standardize it to:

- Fixed Decimal Number
 - Same for `unit_cost`

This isn't necessarily correcting bad data, it is semantic type standardization.

 - II. unit_price_text

This is particularly interesting.

We have:

- `unit_price_text`
- `unit_price`

We need to determine whether:

`unit_price_text` → numeric conversion = `unit_price`

for every product.

 - If yes, unit_price_text may simply be a raw/source representation that we don't need in the final analytical model.

 - If no, then we have a potentially important data-quality issue.

Therefore:

We'll have to validate first; decide whether to retain/drop/derive afterward.

- III. Price vs cost

We haven't tested:

`unit_price` >= `unit_cost`

- So this is currently a business-rule investigation, not a transformation.

- We would not automatically modify prices or costs based on the result.

### fact_order_details

This table has the greatest potential for calculated validation transformations.

We should eventually create temporary audit columns for:

#### Expected selling price

Depending on the representation of `discount_pct`:

    [unit_price] * (1 - [discount_pct] / 100)

#### Expected revenue

    [quantity] * [selling_price]

#### Expected cost

    [quantity] * [unit_cost]

Then compare:

    Expected revenue vs line_total_revenue
    Expected cost vs line_total_cost

These are profiling/validation transformations, not final data transformations.

That's an important distinction.

We would have to create a column such as:

    Revenue Check

with:

    if Number.Abs([line_total_revenue] - ([quantity] * [selling_price])) < 0.01
    then "Valid"
    else "Investigate"

But we won't need to modify `line_total_revenue`.

### The two order tables should eventually be appended

This one is a structural transformation candidate.

We have:

    fact_orders_2022_2023
    7,942 rows

    fact_orders_2024
    4,058 rows

Both have:

 - same 10-column structure
 - same business process
 - same expected grain
 - consecutive time periods

Therefore the eventual transformation is:

    fact_orders_2022_2023
              +
    fact_orders_2024
              ↓
         fact_orders

But before doing the Append, we have to perform one final check:

 - Are `order_id` values unique across the two tables combined?
    - Both tables individually have unique order IDs. That does not prove that the combined population does.

### dim_date — don't recreate it automatically

Our date table is actually looking very healthy.

We won't start changing it just because we're doing transformations.

Instead, we'll need to validate:

    full_date
       ↓
    day
    month
    month_name
    quarter
    quarter_name
    year
    day_of_week
    day_name
    week_of_year
    is_weekend

If the calculated values are correct, leave them alone.

That's actually a valuable finding:

"Profiling confirmed the date dimension's derived attributes were internally consistent, so no transformation is needed."

That's much better than transforming everything just because we can.

### Things that should NOT currently be transformation candidates

Based on our evidence, we would not transform these:

`dim_stores`

- No obvious categorical problems.

Don't modify:

    city
    district
    region
    store_type

unless the cross-table audit reveals an issue.

`dim_employees`

Don't modify:

    gender
    role

Our profiling suggests these are already standardized.

`dim_products`

Don't modify:

    category
    subcategory
    brand

Your current profiling doesn't show a reason to do so.

`fact_orders`

Don't modify:

    payment_method
    order_status

No inconsistency has been identified.

`fact_returns`

Don't modify:

    return_reason
    refund_method
    return_status

Again, we've found no categorical inconsistency.

### We would now create a Transformation Candidate Register

This is actually worth adding as a separate document:

documentation/transformation_plan.md

I'd structure it like this:

ID	Table	Issue	Evidence	Candidate action	Status
T01	dim_customers	birth_date stored as Text	Actual type = Text	Convert to Date	Confirmed
T02	dim_customers	phone stored as Integer	Actual type = Integer	Convert to Text	Confirmed
T03	dim_customers	Gender case inconsistency	6 representations observed	Standardize categories	Confirmed
T04	dim_customers	Duplicate customer IDs	3,050 rows vs 3,000 distinct IDs	Investigate duplicate records	Investigate
T05	dim_customers	Missing email	13% empty	Retain nulls / assess reporting need	Investigate
T06	dim_date	Derived fields	Need comparison with full_date	Validate/recalculate if necessary	Investigate
T07	dim_products	Numeric price fields	Integer monetary fields	Standardize numeric type	Candidate
T08	dim_products	unit_price_text	Text + numeric price fields	Reconcile representations	Investigate
T09	dim_products	Price/cost relationship	Not yet tested	Validate unit_price >= unit_cost	Investigate
T10	fact_order_details	Revenue arithmetic	Not yet tested	Validate line revenue calculation	Investigate
T11	fact_order_details	Cost arithmetic	Not yet tested	Validate line cost calculation	Investigate
T12	fact_orders	Two source tables	Same grain/process	Append into consolidated fact	Confirmed
T13	fact_orders	Employee IDs	217 fact IDs vs 216 employees	Identify orphan employee ID	Investigate
T14	fact_orders	Order/date consistency	Not yet tested	Validate date_id against order_date	Investigate
T15	fact_returns	Order relationship	Not yet tested	Validate return order_id against orders	Investigate













