# Data Integration

## Purpose

This document describes the expected CSV-to-warehouse integration path and the transformations currently coded in the load procedures. It records current behavior separately from business rules that still need approval.

## Integration diagram/photo slot

> **Photo placeholder:** Upload an integration/pipeline screenshot later at `docs/images/data_integration.png`. Then replace this note with: `![Data integration process](images/data_integration.png)`.

## Current integration path

CRM/ERP CSV extracts → Bronze tables (`BULK INSERT`) → Bronze checks → Silver transformations → Gold views → Gold checks and analytics.

This is currently a manually operated full-refresh batch pattern. Procedures truncate and repopulate tables; they do not implement incremental/CDC loads or preserve historical snapshots.

## Source-to-layer mapping

| Source | Bronze | Silver | Intended Gold use |
|---|---|---|---|
| `cust_info.csv` | `BRONZE.crm_cust_info` | `SILVER.crm_cust_info` | Customer dimension |
| `prd_info.csv` | `BRONZE.crm_prd_info` | `SILVER.crm_prd_info` | Product dimension |
| `sales_details.csv` | `BRONZE.crm_sales_details` | `SILVER.crm_sales_details` | Sales fact |
| `CUST_AZ12.csv` | `BRONZE.erp_CUST_AZ12` | `SILVER.erp_CUST_AZ12` | Customer birthdate/gender enrichment |
| `loc_A101.csv` | `BRONZE.erp_loc_A101` | `SILVER.erp_loc_A101` | Customer country enrichment |
| `PX_CAT_G1V2.csv` | `BRONZE.erp_PX_CAT_G1V2` | `SILVER.erp_PX_CAT_G1V2` | Product classification enrichment |

Source files are not included in the repository. Obtain them through an authorized source-data channel. Confirm exact headers/order, delimiter, quote rules, encoding, line endings, date/numeric formats, null markers, embedded delimiters/newlines, snapshot-vs-delta semantics, delivery cadence, source owner, late/missing-file policy, and expected row-count ranges. The current bulk statements use comma field termination and `FIRSTROW = 2`; test actual files before production-like use.

## Stored procedure rules currently coded

### Bronze: `dbo.Load_bronze`

- Loads all six CSV files into their respective Bronze tables.
- Truncates each target before `BULK INSERT`; a failure may leave targets empty or partially loaded.
- Uses `FIRSTROW = 2`, comma `FIELDTERMINATOR`, `TABLOCK`, and hardcoded local Windows paths.
- Prints timing and error details. Its `CATCH` block prints errors but does not rethrow them; callers must not treat a completed procedure call alone as proof of success.
- The script contains `EXEC Load_bronze` after the procedure definition, so executing the entire script also starts a load. Review the file and environment before running it.

### Silver: `silver.load_silver`

The procedure truncates and repopulates each of the six Silver tables. The current coded transformation rules are:

| Subject | Current coded behavior | Decision/validation still needed |
|---|---|---|
| CRM customer | Drops null customer IDs; keeps `ROW_NUMBER() = 1` per ID ordered by latest creation date; trims names; maps `S`/`M`, `F`/`M`; other codes become `n/a`. | Resolve timestamp ties and decide whether unknown codes should be null, retained, or labeled. |
| CRM product | Derives `cat_id` and product key from `prd_key`; defaults null cost to zero; maps selected product-line codes; derives end date as one day before the next start date per source key. | Confirm category-key parsing, zero-cost semantics, duplicate start-date ordering, and product history policy. |
| CRM sales | Converts eight-character integer date values to dates and maps zero/wrong-length dates to null; recalculates sales in specified invalid/inconsistent cases; derives price when null/nonpositive. | Validate impossible dates and conversion failures, currency, precision, discounts/returns, and zero/negative quantity behavior. |
| ERP customer | Removes an initial `NAS` prefix; converts future birthdates to null; normalizes selected gender values; other values become `n/a`. | Confirm identifier normalization and privacy/unknown-value rules. |
| ERP location | Removes hyphens from customer ID; maps `DE`, `US`, and `USA`; blank/null country becomes `n/a`; other values are trimmed. | Define complete canonical country mapping and unknown-country handling. |
| ERP category | Copies category, subcategory, and maintenance fields. | Define key uniqueness and allowed values. |

The Silver procedure's `CATCH` block prints error details but does not rethrow. Because tables are truncated before inserts and the procedure does not provide complete transaction/recovery semantics, inspect errors, row counts, and all targets before proceeding. `NULL`, `n/a`, and zero have different meanings and must follow explicit rules.

## Gold joins and checks

Gold views intend to enrich customer/product dimensions and associate sales to those dimensions using left joins. Left joins can retain unmatched source rows with missing enrichment or keys. Check unmatched counts/rates and right-side key uniqueness to detect fan-out. Reconcile source, Silver, and Gold row counts and business totals where definitions allow.

## Current DDL/view mismatches to resolve

- Silver procedure inserts product `cat_id`, absent from checked-in Silver product DDL.
- Gold joins reference ERP columns `id`/`cid`, whereas checked-in ERP DDL declares `erp_ID`/`erp_CID`.
- Silver sales transformation emits date values, whereas checked-in Silver sales DDL declares integer date fields.

Do not claim successful end-to-end execution until these contracts agree and representative CSVs have been tested.

## Recommended run audit and recovery

For each run, record a run ID, source file name/delivery timestamp, optional checksum, start/end timestamps, status, input/output/rejected counts, and error details. On failure, identify truncated/loaded targets, verify file completeness and row counts, then restart from a known-good stage only after deciding a full reload is safe. Run quality checks before publishing or consuming Gold.

## Related documents

- [Architecture](architecture.md)
- [Data models](data_models.md)
- [Data flow](data_flow.md)
- [Gold data catalog](data_catalog.md)
- [Quality checks](quality_checks.md)
- [Setup and operations](setup_and_operations.md)
