# Naming Conventions

## Purpose

This document defines how to name and describe schemas, objects, columns, procedures, and scripts in this warehouse. It distinguishes current legacy names from the recommended convention for new or revised objects. Use the [Gold data catalog](data_catalog.md) for the meaning of Gold output columns; document all other layer fields alongside their schema/DDL or in layer-specific documentation.

## Schema and layer names

| Schema | Purpose | Example |
|---|---|---|
| `BRONZE` | Source landing | `BRONZE.crm_cust_info` |
| `SILVER` | Prepared and standardized data | `SILVER.crm_prd_info` |
| `GOLD` | Reporting/analytics outputs | `GOLD.dim_customers` |

Use uppercase layer schemas consistently. Existing SQL varies in capitalization.

## Tables and views

- Preserve `crm_` and `erp_` prefixes on source-aligned Bronze/Silver objects to show lineage.
- Use lowercase `snake_case` for new warehouse-created object names and columns where changes are safe.
- Use descriptive nouns for tables and views.
- Prefix Gold dimensions with `dim_` and facts with `fact_` (for example, `dim_customers`, `fact_sales`).
- Retain source-system codes such as `AZ12`, `A101`, and `G1V2` unless the source owner provides their meanings; do not guess their expansions.
- Check downstream references before renaming a published object or column.

## Column naming and description rules

- Keep original source identifiers in Bronze when needed for faithful mapping and traceability.
- Prefer explicit curated names such as `customer_id`, `birth_date`, `order_date`, `sales_amount`, and `unit_price` for newly designed fields.
- Use `_id` for a documented identifier, `_key` only when the key type is specified (business/natural/surrogate), `_date` for date values, and explicit measure names with documented currency/unit semantics.
- An identifier name does not prove uniqueness or a key constraint. A date-like suffix does not make an integer SQL date.
- Avoid undocumented abbreviations. If a source abbreviation must be retained, document its expansion and meaning.

For every table/view and important field, document: business meaning, source/lineage, SQL type, grain, key/uniqueness expectation, null/unknown behavior, allowed values, units/currency when relevant, sensitivity class, and any transformation or exception. Do not describe a proposed convention as implemented behavior.

## Existing source abbreviations

| Existing token | Observed intent | Guidance |
|---|---|---|
| `Cst_` / `cst_` | CRM customer fields | Preserve in source-aligned layers; use full names for new curated aliases. |
| `Prd_` / `prd_` | CRM product fields | `Prd_nm` is product name; `_dt` fields are dates in some schemas. |
| `sls_` | CRM sales fields | Document row grain and measure definitions. |
| `erp_` | ERP source fields | Retain source lineage; explain abbreviations from authoritative source documentation. |
| `_id` / `_key` | Identifier/key-like field | Document uniqueness and whether natural, business, or surrogate. |
| `BDATE`, `CNTRY`, `CAT`, `SUBCAT` | Birth date, country, category, subcategory abbreviations | Prefer explicit names in new curated output. |

## Stored procedure naming and documentation rules

- Use action-oriented names in lowercase snake case, such as `bronze.load_bronze` and `silver.load_silver`, for new procedures; the existing Bronze procedure is `dbo.Load_bronze` and is a legacy exception.
- Each procedure header should state purpose, source and target objects, parameters, full-refresh/incremental behavior, transformation rules, null/invalid-value rules, transaction/recovery behavior, error behavior, and a safe usage example.
- Keep comments aligned with executable SQL. Explicitly document destructive operations such as `TRUNCATE`, implicit execution statements, hardcoded environment-specific paths, and whether errors are rethrown or only printed.
- List every transformation decision (deduplication ordering/ties, code mappings, default values, date conversion, measure recalculation) with business owner/approval status. Do not silently change a business rule in a procedure.
- Use explicit insert column lists and schema-qualified object names; validate that source expressions, target column names, data types, and Gold references agree.
- A procedure that catches and prints an error without rethrowing must not be documented as reliably signaling failure to its caller.

## SQL and documentation file names

Use lowercase `snake_case` for new scripts and Markdown docs, for example `load_silver.sql`, `data_integration.md`, and `naming_conventions.md`. Existing filenames with spaces or inconsistent capitalization are legacy names; do not rename them without updating execution instructions and links.

## Related documents

- [Gold data catalog](data_catalog.md)
- [Data models](data_models.md)
- [Data integration](data_integration.md)
- [Data flow](data_flow.md)
- [Architecture](architecture.md)
