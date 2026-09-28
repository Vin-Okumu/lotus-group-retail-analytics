
<h1 align = "center"> 
Lotus Group Retail Analytics: Transformations Log
</h1>

# Transformation Phase

We'll use a simple rule:

Fix confirmed structural/type issues, standardize obvious inconsistencies, preserve legitimate missing values, and avoid changing business figures unless we've established they're wrong.

## 1. dim_customers

Applied:

    birth_date: Text → Date
    phone: Integer → Text
    gender: standardize male, MALE → Male; female, FEMALE → Female
    Keep missing email as null
    Do not delete the 50 duplicate customer IDs yet.

For the duplicate customers, we'll document:

Duplicate customer IDs were identified during profiling. Because the records were not fully investigated to determine whether they represent exact duplicates, updated customer records, or conflicting records, they were retained to avoid unsupported deletion.

This is much safer than arbitrarily removing them.