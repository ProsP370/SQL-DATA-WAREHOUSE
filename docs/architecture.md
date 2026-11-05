# Data Architecture

## Purpose

This document describes the SQL Server warehouse layers and how their responsibilities fit together. It reflects the current repository implementation and calls out areas that still need validation; it is not proof that the full pipeline currently runs successfully.

## Project overview

The warehouse is intended to combine CRM customer, product, and sales extracts with ERP customer, location, and product-category reference data in the `DATAWAREHOUSE` database. SQL Server DDL creates the layer schemas and tables, stored procedures load Bronze and transform to Silver, and Gold views expose reporting datasets. The repository does not currently define an orchestration service.

## Architecture photo slot

> **Photo placeholder:** Upload the architecture screenshot/diagram later and place it at `docs/images/data_architecture.png`. Then replace this note with: `![Data warehouse architecture](images/data_architecture.png)`.

## Layer responsibilities

### Bronze — source landing

Bronze contains six source-aligned tables: `crm_cust_info`, `crm_prd_info`, `crm_sales_details`, `erp_CUST_AZ12`, `erp_loc_A101`, and `erp_PX_CAT_G1V2`. `dbo.Load_bronze` bulk-loads the CSV files. The current procedure truncates targets before loading and uses locally hardcoded file paths. Bronze is intended to retain source-oriented values; the current DDL does not include file/run audit columns or persistent snapshots.

### Silver — prepared and standardized

Silver contains corresponding CRM and ERP subject tables. `silver.load_silver` currently trims and maps selected values, resolves customer duplicates using the latest creation date, derives product category/product keys and product validity end dates, converts sales dates, and normalizes selected ERP identifiers and attributes. These are coded transformations; confirm their business meaning and edge-case policy with the data owner.

### Gold — reporting datasets

Gold exposes `dim_customers`, `dim_products`, and `fact_sales` as SQL views. Customer and product dimensions enrich CRM records using ERP reference data. The sales fact associates sales details with the dimensions. Views calculate rows when queried; they are not persisted tables. Current dimension keys use `ROW_NUMBER()` and therefore are not durable surrogate keys.

## Intended architecture flow

CSV source files → Bronze landing → source validation → Silver transformations → Gold views → Gold validation → analytics/reporting consumers.

The visual data-flow documentation is maintained in [Data Flow](data_flow.md), where photo slots are reserved for later uploads.

## Runtime and validation

- Database engine: Microsoft SQL Server / T-SQL.
- Source format: CSV via `BULK INSERT`.
- Transformations: stored procedures and views.
- Execution: currently manual; no scheduler is defined in the repository.
- `tests/silver-data-quality` currently checks Bronze despite its filename; `tests/gold_quality_checks` checks Gold views.

## Known implementation risks

Reconcile the SQL contracts before relying on an end-to-end run. The Silver procedure inserts `cat_id` into `SILVER.crm_prd_info`, but the checked-in Silver DDL does not declare that column. The Gold views refer to ERP columns such as `id` and `cid`, while the ERP DDL declares names such as `erp_ID` and `erp_CID`. The Silver sales transform converts dates, while its declared DDL currently uses integer date columns. These are documented mismatches, not resolved by this architecture description.

Other limitations include destructive full refreshes, hardcoded source paths, no durable run audit or automated quality gate, and unstable query-time dimension keys. Customer names and birthdates are personal data; use least-privilege access and approved masked or synthetic data for demonstrations.

## Related documents

- [Data models](data_models.md)
- [Data integration](data_integration.md)
- [Data flow](data_flow.md)
- [Gold data catalog](data_catalog.md)
- [Naming conventions](naming_conventions.md)
- [Setup and operations](setup_and_operations.md)
