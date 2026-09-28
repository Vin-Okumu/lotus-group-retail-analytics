
<h1 align = "center"> 
Lotus Group Retail Analytics: Transformation Plan
</h1>

# Transformation Candidate Register

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


At this point, we have already established the fields that need transformation, and those that still need investigation.

Before starting to clean, we are interested in establishing whether suspected issues are real, and if they are, how large they are, and whether they require transformation.

# Investigation Sequence

We'll investigate in an orderly manner, following the sequence suggested below

    - Validate dim_date derived columns
    - Validate fact_orders date/date_id relationship
    - Investigate foreign-key integrity
    - Investigate the 217th employee ID
    - Investigate duplicate customers
    - Validate product price/cost logic
    - Validate order-detail arithmetic
    - Reconcile order-detail totals against order totals
    - Check order IDs before appending 2022–23 and 2024
    - Validate returns against orders

We'll start with **Investigation 1: `dim_date`** because it is the foundation for all subsequent time-based analysis.

## Investigation 1 — Validate `dim_date`

Our profiling has already established:

    1,096 rows
    1,096 distinct date_id
    1,096 distinct full_date
    no nulls/errors
    dates cover 2022–2024
    derived fields such as day, month, year, day_name, etc. appear structurally valid

But this does not prove that the derived values actually correspond to full_date.

For instance, this record could pass Column Quality:

    full_date	day	month	year
    15/03/2024	15	3	2024

while this could also pass Column Quality:

    full_date	day	month	year
    15/03/2024	14	3	2024

There is nothing "invalid" from Power Query's data-type perspective. The value is simply semantically wrong.

So we're going to test the actual relationships.

### Step 1 — Create a profiling/reference query

We don't want to modify the original `dim_date`.

In Power Query:

Right-click `dim_date` → Reference

Rename the new query:

    profile_dim_date_validation

This is important for us to separate:

    Raw/cleaning query
            ↓
    Profiling/validation query

rather than embedding investigative columns into the production table.

### Step 2 — Validate `day`

Go to: **Add Column → Custom Column**

Power Query supports custom columns specifically for calculations and logical tests involving existing columns.

Name: `check_day`

Formula:

    if [day] = Date.Day([full_date])
    then "Valid"
    else "Invalid"

Now filter check_day to:

#### Results
    Invalid: 0%
    Valid: 100%

Implication:

    day correctly represents the day component of full_date.

### Step 3 — Validate `month`

Create: `check_month`

Formula:

    if [month] = Date.Month([full_date])
    then "Valid"
    else "Invalid"

Filter to Invalid.

#### Results
    Invalid: 0%
    Valid: 100%

Implication:

    month correctly represents the month component of full_date.

### Step 4 — Validate `year`

Create: `check_year`

Formula:

    if [year] = Date.Year([full_date])
    then "Valid"
    else "Invalid"

Filter to Invalid.

#### Results
    Invalid: 0%
    Valid: 100%

Implication:

    year correctly represents the year component of full_date.

### Step 5 — Validate month_name

Create: `check_month_name`

Formula:

    if [month_name] = Date.MonthName([full_date])
    then "Valid"
    else "Invalid"

#### Results
    Invalid: 0%
    Valid: 100%

### Step 6 — Validate `quarter`

Create: `check_quarter`

Formula:

    if [quarter] = Date.QuarterOfYear([full_date])
    then "Valid"
    else "Invalid"

#### Results
    Invalid: 0%
    Valid: 100%

### Step 7 — Validate `quarter_name`

Here we first have to know the dataset's naming convention.

For our dataset our values are:

    Q1
    Q2
    Q3
    Q4

Hence we'll use:

    if [quarter_name] = "Q" & Text.From(Date.QuarterOfYear([full_date]))
    then "Valid"
    else "Invalid"

#### Results
    Invalid: 0%
    Valid: 100%

### Step 8 — Validate `day_of_week`

Here, it's important to note that Power Query's:

    Date.DayOfWeek([full_date])

defaults to:

    Monday = 0
    Tuesday = 1
    Wednesday = 2
    ...
    Sunday = 6

But our dataset uses:

    Monday = 1
    Tuesday = 2
    ...
    Sunday = 7

These are different conventions.

Since the numbering is 1–7

we'll create `check_day_of_week`

then we'll use:

    if [day_of_week] = Date.DayOfWeek([full_date], Day.Monday) + 1
    then "Valid"
    else "Invalid"

### Step 9 — Validate `day_name`

Create: `check_day_name`

Our dataset uses English names so:

    if [day_name] = Date.DayOfWeekName([full_date])
    then "Valid"
    else "Invalid"

### Step 10 — Validate `is_weekend`

We need to first establish the dataset's definition.

Without assuming our dataset has:

    Friday = weekend
    Saturday = weekend


create: `check_is_weekend`

with:

    if [is_weekend] =
        (if Date.DayOfWeek([full_date], Day.Sunday) >= 5
        then 1
        else 0)
    then "Valid"
    else "Invalid"

This deliberately uses Sunday as day 0, making:

    Sunday = 0
    Monday  = 1
    Tuesday = 2
    Wednesday = 3
    Thursday = 4
    Friday = 5
    Saturday = 6


Therefore:

    >= 5 → Friday/Saturday

### Step 11 — Validate `week_of_year`

This one needs slightly more care because week numbering systems differ.

We won't immediately assume that the dataset should match:

    Date.WeekOfYear([full_date])

Instead, we'll first inspect:

    week_of_year

and establish its range.

- We already found:

    - 52 distinct values.

That's interesting because a year can contain week 53 depending on the week-numbering convention and calendar year.

For this investigation, we'll create: `calculated_week`

rather than immediately declaring anything invalid:

    Date.WeekOfYear([full_date])

Then compare:

    week_of_year
    calculated_week

#### Results

There are discrepancies occuring, so we'll need to investigate the convention rather than automatically replacing the dataset values.

### Step 12 — Validate `is_ramadan`

This is different:
- We cannot simply derive this from full_date using a standard Power Query date function.
    - is_ramadan is a business/calendar attribute.

So we'll treat it as a business-rule validation, not a generic date transformation.

For now, we'll:

- Inspect its distinct values.
- Confirm whether they are 0/1, True/False, etc.
- Look at the date ranges associated with each value.
- Check whether the Ramadan periods appear logically consistent.

Create a reference query:

    profile_ramadan_dates

Then sort:

    full_date ascending

and inspect transitions in:

    is_ramadan

We are looking for something like:

    2022-03-01   0
    2022-03-02   0
    ...
    2022-04-02   1
    2022-04-03   1
    ...
    2022-05-01   0

rather than isolated random values.

We won't change is_ramadan just yet.


##  Investigation 2: Validate facts_orders date

### Cross-table investigation framework

    Area	                            What we're checking
    1. Orders → Date	                Do order dates and date keys agree, and do keys exist?
    2. Orders → Dimensions	            Do customer, store and employee IDs exist?
    3. Order Details → Orders/Products	Do every order line and product have a valid parent?
    4. Order Details → Orders	        Do line-level financials reconcile to order-level totals?
    5. Returns → Orders	                Do returns refer to legitimate orders?

We'll use Merge Queries rather than creating lots of custom columns.

#### 1. First: We'll create the consolidated Orders table

Because `fact_orders_2022_2023` and `fact_orders_2024` have the same structure, we eventually want:

`fact_orders`

But before we actually create the production table, let's create a profiling version.

- In Power Query:

    Home → Append Queries → Append Queries as New

- Select:

    fact_orders_2022_2023
    fact_orders_2024

- We'll name it:

    profile_fact_orders

 - This gives us one 12,000-row order table for cross-table investigation.

 - Immediately we check order_id
    - We get:
        - 12,000 rows
        - 12,000 distinct order IDs

It means the two source tables do not contain duplicate order IDs across the 2022–2024 boundary.

#### 2. Referential-integrity audit

Now we'll use one repeatable technique.

- For each relationship:
    - We'll merge the fact table with the dimension table using the relevant key, then use a Left Anti join to isolate unmatched records.
        - Power Query's Left Anti join returns rows from the first table for which there is no matching row in the second table.

This is exactly what we need.

- 2.1 Orders → Date

We'll start with:
- profile_fact_orders

Go to:
- Home → Merge Queries → Merge Queries as New
    - Configure:
        - First table: `profile_fact_orders`
        - Second table: `dim_date`

Select:
- profile_fact_orders[date_id]

and:
- dim_date[date_id]

Join kind:
- Left Anti (rows only in first)

Name:
- audit_orders_missing_dates

**Interpretation**

- The result has:

0 rows

meaning: Every order's date_id exists in dim_date.

#### Orders → Customers

We want to create another Merge query.

- First: `profile_fact_orders`
- Second: `dim_customers`
    - Join: customer_id → customer_id
    - Join kind: Left Anti
    - Name: audit_orders_missing_customers

This answers:

**Result**: The table is empty, meaning there are no orders referring to customer IDs that don't exist in the customer dimension?

Note: 

    We already know:

    dim_customers
    3,050 rows
    3,000 distinct customer IDs

So we're deliberately not treating duplicate customer records as missing customers.

A customer ID appearing multiple times in the dimension can still match an order.

The separate question of whether those duplicate customer records should be resolved comes later.

#### Orders → Stores

We'll merge: `profile_fact_orders` with  `dim_stores`

on: `store_id`

using: Left Anti and name it `audit_orders_missing_stores`

**Results**: The table is empty, meaning there are no orders referring to store IDs that don't exist in the store dimension.

**We'll do the same for `dim_employees`, `dim_products`, `fact_order_details`, `fact_returns`

**Results**
- audit_orders_missing_employees: Table has 632 records missing employee_id
- audit_details_missing_orders:  The table is empty.
- audit_returns_missing_orders: The table is empty 

#### Cross-table audit table

    Relationship	    Fact rows	Distinct FK	Unmatched rows	Result

    Orders → Date	    12,000	    1096	    0	            ?
    Orders → Customer	12,000	    3050	    0	            ?
    Orders → Store	    12,000	    15	        0	            ?
    Orders → Employee	12,000	    217	        632	            ?
    Details → Order	    25,099	    12,000	    0	            ?
    Returns → Order	    1,056	    1,056	    0	            ?

## Investigation 3: Financial Reconciliation

Here we're interested in determining whether the financial information at the order-line level agrees with the financial information at the order level.

Our structure is:

    fact_order_details
            ↓
        order_id
            ↓
    fact_orders

At the detail level we have:

    quantity
    unit_price
    discount_pct
    selling_price
    unit_cost
    line_total_revenue
    line_total_cost

At the order level:

    total_revenue
    total_cost

### Phase 1 — Validate each order line

Before aggregating anything, we want to test the arithmetic within fact_order_details.

There are three relationships worth checking:

#### A. Discount → selling price

Conceptually:
    selling_price = unit_price × (1 - discount)

#### B. Quantity → revenue
    line_total_revenue = quantity × selling_price

#### C. Quantity → cost
    line_total_cost = quantity × unit_cost

We don't want to modify the existing columns though, so:

- Create a Reference of:

    fact_order_details

and call it:

    profile_detail_financial_validation

##### First investigate discount_pct

The dataset has discoiunt_pct recorded as 10 to mean 10%

Knowing this, we create:

    `calculated_selling_price`

with:

    [unit_price] * (1 - [discount_pct] / 100)

Then we create:

    `check_selling_price`

with:

    if Number.Round([selling_price], 2) =
       Number.Round([calculated_selling_price], 2)
    then "Valid"
    else "Invalid"

The rounding is intentional because we're dealing with monetary calculations.
- What we're looking for:

Invalid = 0: Implying the discount/selling-price relationship is internally consistent.

##### Validate line revenue

Now we create:

    `calculated_line_revenue`

Formula:

[quantity] * [selling_price]

Then:

check_line_revenue

Formula:

if Number.Round([line_total_revenue], 2) =
   Number.Round([calculated_line_revenue], 2)
then "Valid"
else "Invalid"

Filter to:

Invalid

Record:

number of invalid rows
percentage of total detail rows

With 25,099 detail records, we'll know exactly how many financial records don't reconcile.

