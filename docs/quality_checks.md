# Data Quality Checks

## Purpose

This document describes the data-quality checks currently available in the repository, when to run them, how to interpret their results, and what should be added as the warehouse evolves. The checks are SQL diagnostic queries; they do not currently form an automated test suite and do not stop a load when a rule is violated.

The intent is to detect issues early, make data behavior visible, and help protect the Gold reporting views from incomplete or inconsistent inputs. A clean run means only that the checks included in the scripts returned no exceptions—it is not a guarantee that every business requirement or data defect has been covered.

## Check locations and current scope

| Script | Data examined | When to run | Current role |
| --- | --- | --- | --- |
| `tests/silver-data-quality` | Bronze CRM and ERP tables | After Bronze load and before Silver transformation | Source/landing-data diagnostics. Despite the filename, these queries inspect Bronze tables; they are not Silver-layer checks. |
| `tests/gold_quality_checks` | Gold dimension and fact views | After Gold views have been created or refreshed | Reporting-model diagnostics for key completeness, duplicates, unmatched references, missing values, and date sequencing. |

There are currently no dedicated, clearly identified Silver-layer quality-check queries. The `tests/silver-data-quality` filename may be renamed in a future change to make its Bronze scope obvious; update any references when doing so.

## Recommended validation workflow

1. **Load Bronze.** Run the configured full-refresh Bronze load and confirm it completed successfully.
2. **Check Bronze.** Run `tests/silver-data-quality`. Investigate returned records and aggregate counts. Decide whether each issue should be corrected at source, handled by a documented transformation rule, or treated as an approved exception.
3. **Load Silver.** Run the Bronze-to-Silver transformation only after reviewing the source checks. Confirm the procedure completed and that expected target tables contain data.
4. **Validate Silver.** Until a dedicated Silver suite is added, perform the recommended Silver checks in the section below or run equivalent ad hoc queries. Do not treat Bronze checks as a substitute for validating transformed results.
5. **Create or refresh Gold.** Ensure the Gold views compile against the current Silver definitions.
6. **Check Gold.** Run `tests/gold_quality_checks`. Review all returned records and verify the Gold model is suitable for the intended reporting use.
7. **Record the outcome.** For each execution, capture the date/time, source files or batch identifier, layer, check script/version, result, exception owner, and disposition. The current repository does not persist this run history automatically.

For a production or business-critical release, define which checks are release-blocking. At minimum, failures involving invalid joins, missing fact keys, duplicate business keys, or an unexpected empty load should be investigated before publishing data.

## Current Bronze/source checks

The queries in `tests/silver-data-quality` inspect the raw Bronze tables before `silver.load_silver` runs. They currently cover:

### CRM customers (`BRONZE.crm_cust_info`)

- Counts missing customer IDs and repeated customer IDs.
- Finds missing or blank first and last names.
- Flags marital-status values outside the expected `S` and `M` codes and gender values outside `F` and `M`.
- Identifies customers whose most recent creation date is tied across multiple rows, since the Silver load selects a latest row by creation date.

### CRM products (`BRONZE.crm_prd_info`)

- Counts missing and repeated product IDs.
- Finds missing product keys or names, negative costs, missing start dates, and end dates earlier than start dates.
- Flags product-line values outside the expected `M`, `R`, `S`, and `T` codes.
- Finds repeated product keys with the same start date.

### CRM sales (`BRONZE.crm_sales_details`)

- Finds missing order, product, or customer identifiers and missing/non-positive quantity, sales, or price values.
- Checks that order, ship, and due date values are non-zero, eight digits, and convertible to dates using the `YYYYMMDD` format.
- Flags rows where sales do not equal quantity multiplied by the absolute value of price.

### ERP customer reference (`BRONZE.erp_cust_az12`)

- Counts missing and repeated customer IDs.
- Finds future birthdates and gender values outside the expected `F`, `FEMALE`, `M`, and `MALE` values.

### ERP location reference (`BRONZE.erp_loc_a101`)

- Counts missing and repeated customer IDs.
- Finds missing/blank country values and country codes other than the explicitly mapped `DE`, `US`, and `USA` values.

### ERP product category reference (`BRONZE.erp_px_cat_g1v2`)

- Counts missing and repeated category IDs.
- Finds missing or blank category, subcategory, and maintenance values.

## Current Gold checks

The queries in `tests/gold_quality_checks` inspect `GOLD.dim_products`, `GOLD.dim_customers`, and `GOLD.fact_sales`. The assumptions stated in that script include unique product numbers and customer IDs, sales records matching both dimensions, and ship/due dates not preceding the order date.

### Product dimension

- Missing product key or product number.
- Duplicate product surrogate keys or product numbers.
- Missing or blank product names.

### Customer dimension

- Missing customer key or customer ID.
- Duplicate customer surrogate keys or customer IDs.
- Missing or blank customer numbers or customer names.

### Sales fact

- Missing order number, product key, customer key, sales, quantity, or price.
- Product or customer keys that do not match a row in the corresponding Gold dimension.
- Order dates later than ship dates or due dates.

## How to interpret results

- **Detail query returns rows:** each row is a candidate exception requiring review. Confirm whether it is a source defect, an expected edge case, or a transformation/model defect.
- **Aggregate query returns a non-zero count:** investigate the count and, where useful, run a detail query to identify the records.
- **No rows are returned:** the specific predicate did not find an exception. It does not prove that the entire dataset is correct.
- **A query errors or an object is missing:** treat the validation as incomplete, not as a pass. Confirm that scripts ran in the correct database and that the required tables/views and columns exist.

Do not silently remove records simply to make a check return no rows. Record the reason for an accepted exception, its owner, and whether it is temporary or permanent.

## Recommended severity and disposition

The repository does not currently assign severity levels or automatically fail a load. The following is a suggested operating convention to adopt as the project matures:

| Suggested severity | Examples | Suggested response |
| --- | --- | --- |
| **Critical** | Target layer is empty unexpectedly; duplicate business key causes join multiplication; fact rows have invalid or missing required dimension references | Stop downstream publication, investigate, and rerun only after safe recovery is understood. |
| **High** | Invalid date conversion, missing required sales measure, or large/unexplained increase in rejected or unmatched rows | Block or hold the reporting refresh until the owner reviews the issue. |
| **Medium** | Missing optional descriptive attributes, unmapped but usable reference values, or isolated duplicate source records with a documented resolution | Assign an owner and document the treatment; monitor recurrence. |
| **Low / informational** | Expected source variation that is handled by an agreed transformation rule | Track the count and confirm the rule remains appropriate. |

Severity should be agreed with data owners and report consumers; these examples are recommendations, not current automated behavior.

## Additional Silver checks recommended

Add a dedicated Silver validation script and run it after `silver.load_silver`. Consider checking:

- Required keys are present and unique at the intended grain for each Silver table.
- Customer deduplication produces at most one row per customer ID and ties are resolved deterministically.
- Product identifiers and derived category/product keys are populated and join correctly to the ERP category reference.
- Product validity intervals are coherent: start date is present, end date is not before start date, and overlapping intervals are detected where the business model disallows them.
- Sales date columns have the intended SQL data type and valid values; invalid source dates are either intentionally null or handled according to a documented rule.
- Sales quantity, price, and sales amount satisfy agreed business rules, including zero/negative values, nulls, and arithmetic precision.
- ERP customer IDs and CRM customer keys use compatible formats and join at the expected coverage.
- Country, gender, marital-status, product-line, category, and maintenance values follow the accepted reference-value policy.
- Row counts and distinct business-key counts reconcile to Bronze, with explained differences for filtered or deduplicated records.
- No unintended duplicates are introduced by reference-data joins.

These checks should be aligned with the actual Silver schema and business rules before they are treated as pass/fail criteria.

## Additional checks recommended across layers

### Completeness and volume

Record row counts per source file and per target table. Compare counts and distinct business-key counts between runs. Define reasonable expectations or thresholds for empty files, unusually small/large loads, and abrupt volume changes. A full refresh that loads zero rows should be treated as suspicious unless explicitly expected.

### Referential integrity and join coverage

Measure how many CRM sales rows match a customer and product, and how many customer/product dimension rows match their ERP reference data. Report both the unmatched count and percentage. An unmatched row may be a valid late-arriving or incomplete reference, so define the expected handling rather than assuming every miss is automatically invalid.

### Freshness and timeliness

Where source delivery timestamps or file dates are available, validate that the files are current and that the latest expected business date has arrived. The current scripts do not record source timestamps or define a freshness SLA; add these only after agreeing on delivery expectations.

### Schema and file-contract validation

Before loading, verify required files exist, expected headers/column counts are present, and data types and formats are compatible with the target tables. CSV delimiter, encoding, quoting, and embedded-newline behavior should be explicit in the source contract.

### Reconciliation

Define business-level reconciliations, for example total source sales versus transformed sales, with documented handling for corrections, deduplication, invalid records, and rounding. Reconciliation tolerances must be agreed with business owners; do not assume exact equality if transformations intentionally alter values.

### Stability and change detection

Track distributions and distinct values for key attributes (such as product line, country, gender, and category) so new source values are noticed rather than silently mapped to a generic value. Monitor duplicate and null rates over time, not only whether a single run returns rows.

## Important limitations and implementation considerations

- The existing tests are hand-run SQL queries; they do not emit a single pass/fail status, fail a deployment, or prevent downstream loads.
- There is no automated test runner, persistent exception table, run audit, alerting, or documented threshold configuration in the repository.
- The Bronze-check filename `tests/silver-data-quality` is misleading because its queries target Bronze tables.
- Some aggregate duplicate-count patterns use `COUNT(column) - COUNT(DISTINCT column)`, which does not count `NULL` values as duplicates. Missing-value checks must remain separate.
- The Gold surrogate keys are generated with `ROW_NUMBER()` in views and are not stable identifiers across changing data. Tests should not imply that these values are durable warehouse keys.
- The Gold checks rely on the Gold views compiling and on their joins producing the intended grain. Validate table/view column names, data types, and join cardinality before relying on test outcomes.
- The Bronze and Silver load procedures truncate tables as part of full refreshes. A failed load can leave incomplete target data; check procedure output and target row counts before retrying or publishing results.

## Operating record template

For each validation run, record at least:

| Field | Example / description |
| --- | --- |
| Run identifier | Unique batch or execution ID, if available |
| Run date/time | Timestamp and timezone |
| Environment | Development, test, or production |
| Layer and script | Bronze / `tests/silver-data-quality`, Silver / future dedicated script, or Gold / `tests/gold_quality_checks` |
| Source files | Names and delivery dates or checksums where available |
| Result | Pass, fail, warning, or not run; include exception counts |
| Exceptions | Rule, affected count, and link/location of details |
| Disposition | Fixed, accepted, quarantined, or pending |
| Owner | Person/team responsible for follow-up |

## Ownership and maintenance

Update this document and the SQL test scripts together whenever a source field, transformation, business rule, model grain, or expected value changes. Each critical rule should have a named owner, an agreed action on failure, and a way to confirm that the issue is resolved.
