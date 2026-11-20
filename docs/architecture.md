# Architecture

## Document purpose

This document explains the SQL Data Warehouse project’s system boundary, processing layers, major objects, data flow, and operational limitations. It describes the repository as it exists and labels proposed capabilities as recommendations rather than implemented features.

## Project overview

The project brings together CRM customer, product, and sales extracts with ERP customer, location, and product-category reference extracts in a SQL Server database named `DATAWAREHOUSE`. The intended outcome is a reporting layer with customer and product dimensions and a sales fact view.

The repository contains SQL Server DDL, stored procedures, views, and SQL diagnostic checks. It does not contain the input CSVs or an orchestration service.

## System context and flow

<img width="1550" height="1052" alt="Data architecture draw corrected 2" src="https://github.com/user-attachments/assets/134062d7-c635-4e20-b22b-193d49283531" />

The intended data lifecycle is file delivery → Bronze load → source validation → Silver transformations → Gold view creation → Gold validation → analytics use.

## Layer responsibilities

### Bronze

Bronze is the source-oriented landing layer. It contains six tables: `crm_cust_info`, `crm_prd_info`, `crm_sales_details`, `erp_CUST_AZ12`, `erp_loc_A101`, and `erp_PX_CAT_G1V2`. The `dbo.Load_bronze` procedure uses `BULK INSERT` to load corresponding CSV files. The load truncates each destination before inserting, so it is a full refresh rather than an incremental process.

Bronze is intended to make source data available for validation and downstream processing. Persistent source snapshots, file checksums, and load audit history are not currently implemented.

### Silver

Silver is the preparation and standardization layer. `silver.load_silver` currently contains transformations for trimming and mapping attributes, selecting a customer record by latest creation date, deriving product fields and validity dates, converting sales date values, adjusting inconsistent sales/price values, normalizing ERP identifiers, and standardizing selected country and gender values.

The transformation rules are code behavior, not necessarily approved business definitions. Confirm them with source and business owners before using outputs as authoritative measures.

### Gold

Gold contains three intended reporting views: `dim_customers`, `dim_products`, and `fact_sales`. The views combine Silver objects and provide more report-oriented aliases. They are SQL views and do not materialize separate query results. Customer and product keys are generated with `ROW_NUMBER()` at query time and are not stable persisted identifiers.

## Model overview

The Gold layer is intended as a star-like model: a sales fact relates to customer and product dimensions. The expected fact grain appears to be one sales-detail record, but this must be verified against source data. The repository does not establish a formal unique sales-line key or enforce physical foreign keys.

See [Data models](data_models.md) for grain, relationships, and key-design considerations.

## Validation components

- `tests/silver-data-quality` checks Bronze data despite its filename.
- `tests/gold_quality_checks` checks the Gold views.

The checks are manually executed SQL diagnostics. They do not automatically fail loads, persist history, or prevent reporting publication.

## Technologies and execution model

- Database engine: Microsoft SQL Server / T-SQL.
- Source format: CSV files loaded with `BULK INSERT`.
- Transformations: SQL stored procedures and views.
- Execution: currently manual; no scheduler or orchestration framework is defined in the repository.

## Security and data handling

Customer data includes names, birthdates, gender, marital status, and country. Access should follow least privilege. Do not commit real source files, credentials, or sensitive extracts. Use synthetic or approved masked data for public demonstrations. Retention and privacy requirements must be supplied by the data owner; they are not defined by the scripts.

## Known limitations and implementation validation

Before claiming the pipeline is executable end-to-end, reconcile the code. Current SQL appears to contain schema/column mismatches: for example, the Gold definitions reference ERP/product columns that do not match the declared Silver DDL, and Silver sales date columns are declared as integers although the transformation converts values to dates. The database initialization script is destructive and its existence check should be validated. Run all scripts in sequence against representative data after correcting these issues.

Other current limitations:

- local hardcoded source file paths;
- full refresh, with no incremental history;
- no persistent load audit, alerting, or retry framework;
- no automated quality gates;
- query-time, unstable dimension keys;
- source contracts and freshness expectations not formally defined.

## Future architecture improvements

Prioritize: reconcile and test schemas; externalize paths; add batch/run audit and row counts; make failure and recovery behavior explicit; automate quality gates; define stable persisted keys if required; and introduce incremental processing only when source volume and history requirements justify it.

## Related documents

- [Data integration](data_integration.md)
- [Data models](data_models.md)
- [Data catalog](data_catalog.md)
- [Naming conventions](naming_conventions.md)
- [Quality checks](quality_checks.md)
- [Setup and operations](setup_and_operations.md)
