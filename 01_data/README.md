# 01_data: Source Data

This folder holds the raw source data for the Lotus Group Retail Analytics project, exactly as it was received. **None of these files have been edited.** All cleaning happens in Power Query inside the `.pbix` file, so every change can be traced and repeated (see [`03_power_query/transformations_log.md`](../03_power_query/transformations_log.md)).

## Contents

```
01_data/
└── raw/
    ├── dim_customers.csv
    ├── dim_date.csv
    ├── dim_employees.csv
    ├── dim_products.csv
    ├── dim_stores.csv
    ├── fact_orders_2022_2023.csv
    ├── fact_orders_2024.csv
    ├── fact_order_details.csv
    └── fact_returns.csv
```

| File | Description | Rows | Columns |
|---|---|---|---|
| `dim_customers.csv` | Registered customers, with demographics, location and loyalty tier | 3,050 | 10 |
| `dim_date.csv` | Calendar from 2022 to 2024, with weekend and Ramadan flags | 1,096 | 13 |
| `dim_employees.csv` | Store staff, with role, store and hire date | 216 | 8 |
| `dim_products.csv` | Product catalogue, with category, brand, price and cost | 345 | 10 |
| `dim_stores.csv` | 15 stores, with city, region, size and format | 15 | 8 |
| `fact_orders_2022_2023.csv` | Order headers for 2022–2023 | 7,942 | 10 |
| `fact_orders_2024.csv` | Order headers for 2024 | 4,058 | 10 |
| `fact_order_details.csv` | Order lines: product, quantity, price, discount and cost | 25,099 | 10 |
| `fact_returns.csv` | Returns, with reason, amount and status | 1,056 | 7 |
| **Total** | | **42,877** | |

## Key facts

- **Period:** 1 January 2022 – 31 December 2024
- **Currency:** Egyptian pounds (EGP)
- **Format:** comma-separated UTF-8 CSV with a header row
- **Dates:** `YYYY-MM-DD`, except some `birth_date` values in `dim_customers`, which use `DD/MM/YYYY`
- **Keys:** text IDs with a prefix (`CUS`, `EMP`, `CLT`/`ELC`, `ORD`, `DTL`, `RET`); `store_id` is a whole number; `date_id` is `YYYYMMDD`

## Known data-quality issues

These issues are left in the raw files on purpose and handled in Power Query:

| File | Issue | How it's handled |
|---|---|---|
| `dim_customers.csv` | 50 exact duplicate rows (3,050 rows, 3,000 customers) | Duplicates removed |
| `dim_customers.csv` | `gender` spelled 6 different ways (`male`, `MALE`, …) | Standardised to Male / Female |
| `dim_customers.csv` | `birth_date` stored as text in two formats | Converted to Date |
| `dim_customers.csv` | `phone` read as a number, and some values have extra spaces | Converted to Text |
| `dim_customers.csv` | 382 missing emails | Left as null |
| `fact_orders_*.csv` | 632 orders with no `employee_id` | Kept, and tracked with the attribution-rate measure |
| `fact_orders_*.csv` | `payment_method` values with leading or trailing spaces | Trim before analysis |
| `fact_orders_*.csv` | Orders split across two files | Appended into `fact_orders` |

The full investigation is in [`02_documentation/data_quality_assessment.md`](../02_documentation/data_quality_assessment.md).

## Related documentation

- [Data dictionary](../02_documentation/data_dictionary.md): every column explained
- [Data model](../02_documentation/data_model.md): how the tables relate

## Licence and disclaimer

This is a **fictional, educational dataset** created for analytics practice. The company, people and transactions are not real. Any resemblance to real people or businesses is coincidental.
