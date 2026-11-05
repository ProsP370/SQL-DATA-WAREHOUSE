# Data Integration

## Purpose and scope

This document describes how source files are expected to enter the warehouse, how they map to Bronze and Silver, and how the Gold reporting views consume the prepared data. It records current SQL behavior and separates it from recommendations that are not yet implemented.

## Integration pattern

```text
CRM/ERP CSV extracts
        -> Bronze tables (BULK INSERT)
        -> Bronze source checks
        -> Silver transformations
        -> Gold reporting views
        -> Gold checks and analytics
```

This is a manually operated batch pipeline. The current load procedures truncate and repopulate tables; they are not incremental, CDC-based, or history-preserving loads.

## Source inventory and mapping

| Source area | File | Bronze table | Silver table | Gold use |
| --- | --- | --- | --- | --- |
| CRM | `cust_info.csv` | `BRONZE.crm_cust_info` | `SILVER.crm_cust_info` | Customer dimension |
| CRM | `prd_info.csv` | `BRONZE.crm_prd_info` | `SILVER.crm_prd_info` | Product dimension |
| CRM | `sales_details.csv` | `BRONZE.crm_sales_details` | `SILVER.crm_sales_details` | Sales fact |
| ERP | `CUST_AZ12.csv` | `BRONZE.erp_CUST_AZ12` | `SILVER.erp_CUST_AZ12` | Customer birthdate/gender enrichment |
| ERP | `loc_A101.csv` | `BRONZE.erp_loc_A101` | `SILVER.erp_loc_A101` | Customer country enrichment |
| ERP | `PX_CAT_G1V2.csv` | `BRONZE.erp_PX_CAT_G1V2` | `SILVER.erp_PX_CAT_G1V2` | Product classification enrichment |

The files are expected by the SQL scripts but are not included in the repository. Obtain them through an authorized source-data channel. The SQL Server service account must be able to read the configured path.

## File contract

The current `BULK INSERT` statements use comma field termination and skip the first row (`FIRSTROW = 2`). The actual source contracts must be confirmed. For every file, document:

- exact headers and column order;
- delimiter, quote/text qualifier, encoding, and line endings;
- date representation, numeric precision, null and empty-value representation;
- treatment of embedded commas, quotes, and newlines;
- whether the delivery is a complete snapshot or a delta;
- file naming convention, delivery cadence, source owner, late/missing-file policy;
- expected row-count ranges or other arrival checks.

Do not assume the CSV format is safe for bulk import until tested against representative files.

## Bronze ingestion

`script/bronze/proc-stored-procedure` defines `dbo.Load_bronze`. The procedure loads the six source files and truncates each target table before its load. It prints timing and error details to the SQL output, but does not persist a run record. The file paths currently point to a local Windows directory and must be adapted.

Because truncation precedes loading, a failure may leave a target empty or partially populated. Check the execution result and row counts for every target before proceeding. The current procedure does not provide durable file-level lineage, run IDs, checksums, restart position, or a full transactional recovery strategy.

## Transformation rules

The Bronze-to-Silver procedure in `script/silver/proc-silver-stored-procedure` contains these behaviors:

| Subject | Current coded behavior | Business decision to confirm |
| --- | --- | --- |
| CRM customers | Trims names; maps `S`/`M` and `F`/`M` to labels; selects latest row per customer ID using creation date. | Handling of unknown codes and ties in latest timestamp. |
| CRM products | Splits/derives category and product keys; maps product-line codes; defaults missing cost to zero; derives an end date from the next start date. | Category-key mapping, cost default semantics, and product history policy. |
| CRM sales | Converts valid eight-digit dates; invalid values may become `NULL`; recalculates sales in specified invalid/inconsistent cases; may derive price. | Currency, discounts, returns, zero/negative values, rounding and precedence between sales and price. |
| ERP customer | Removes an `NAS` prefix where present; future birthdate becomes `NULL`; normalizes gender. | Identifier normalization and unknown-value policy. |
| ERP location | Removes hyphens from customer ID; maps selected country codes to names and blanks to `n/a`. | Accepted country set and canonical country representation. |
| ERP category | Copies category, subcategory, and maintenance fields. | Reference-data key uniqueness and allowed values. |

Use `NULL`, `n/a`, and zero only according to explicit rules: these values have different meanings to downstream analysis.

## Gold integration and joins

The Gold definitions intend to:

- enrich CRM products with ERP category information;
- enrich CRM customers with ERP birthdate/gender and location/country values;
- associate sales records to customer and product dimensions.

The view definitions use left joins, so unmatched records can remain with missing enrichment or dimension keys. Quality checks should quantify both counts and rates of unmatched rows. Validate that reference keys are unique before joins; a duplicate reference key can multiply rows and inflate sales totals.

## Operational sequence

1. Verify the approved source files are present and readable on the SQL Server host.
2. Create Bronze tables and configure file paths.
3. Run Bronze load and check completion/row counts.
4. Execute `tests/silver-data-quality` (these queries inspect Bronze, notwithstanding the filename).
5. Review source exceptions and document dispositions.
6. Create Silver tables and execute `silver.load_silver`.
7. Validate Silver outputs; a dedicated Silver test file is recommended but not currently present.
8. Create Gold views and run `tests/gold_quality_checks`.
9. Reconcile row counts, distinct keys, join coverage, and business totals where the transformation rules permit.

## Recommended integration audit

For each batch, persist or otherwise record a run ID, source file name, source delivery timestamp, checksum (where useful), start/end time, status, input/output/rejected row counts, and error details. The current repository does not implement this audit.

## Failure and recovery

Do not blindly rerun after an error: tables may already have been truncated. Determine which objects were affected, verify input files and row counts, and decide whether a full reload from known-good inputs is safe. For dependable operations, implement an explicit load state, transaction/failure behavior, and recovery procedure.

## Known code-contract risks

The current scripts need reconciliation before this mapping can be treated as executable documentation. Product category and ERP join columns referenced by Gold do not appear to match Silver DDL names, the Silver product DDL does not declare a category field referenced by transformation logic, and sales date target types differ from the date conversion behavior. Resolve these issues and test end-to-end with representative CSVs.

## Related documents

- [Architecture](architecture.md)
- [Data models](data_models.md)
- [Data catalog](data_catalog.md)
- [Quality checks](quality_checks.md)
- [Setup and operations](setup_and_operations.md)
