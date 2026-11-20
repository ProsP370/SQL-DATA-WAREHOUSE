# Naming Conventions

## Purpose

This document records naming styles present in the project’s data dictionary and SQL definitions, then gives consistent rules for future work. Use this guide when creating or reviewing warehouse objects. Existing names are not completely uniform; the conventions below apply especially to new objects and planned renaming efforts.

## 1. Layer/schema naming

The project uses layer names as schemas:

| Schema | Meaning | Example |
| --- | --- | --- |
| `BRONZE` | Raw/source landing | `BRONZE.crm_cust_info` |
| `SILVER` | Cleaned/conformed | `SILVER.crm_prd_info` |
| `GOLD` | Reporting/analytics | `GOLD.dim_customers` |

The intended convention is uppercase layer schemas. Existing statements vary between `Gold`, `GOLD`, `bronze`, and `BRONZE`; use uppercase consistently in new or revised SQL. SQL Server’s case sensitivity may vary by database configuration, so naming should not depend on case-only differences.

## 2. Source-system prefixes

Bronze and Silver names use source prefixes to retain lineage:

- `crm_` for CRM objects: `crm_cust_info`, `crm_prd_info`, `crm_sales_details`.
- `erp_` for ERP reference objects: `erp_CUST_AZ12`, `erp_loc_A101`, `erp_PX_CAT_G1V2`.

Preserve source identifiers in raw landing data where fidelity is important. Use clearer business aliases in Gold when doing so does not obscure lineage.

## 3. Entity and object naming patterns

Observed table patterns include:

- `<source>_<entity>_<descriptor>` — `crm_sales_details`.
- `<source>_<entity>` — `crm_cust_info`, `crm_prd_info`.
- `<source>_<source-code>` — `erp_loc_A101`, `erp_PX_CAT_G1V2`.

Some codes come from the source system. Do not expand or reinterpret `AZ12`, `A101`, or `G1V2` without a source-owner definition.

Gold follows common dimensional prefixes:

- `dim_` for dimensions: `dim_customers`, `dim_products`.
- `fact_` for facts: `fact_sales`.

Use nouns for tables and views. Use action verbs for procedures.

## 4. Data dictionary column prefixes

The source dictionary prefixes columns with a subject/system abbreviation:

| Prefix | Subject | Examples |
| --- | --- | --- |
| `Cst_` / `cst_` | CRM customer | `Cst_id`, `Cst_key`, `Cst_firstname`, `Cst_create_date` |
| `Prd_` / `prd_` | CRM product | `Prd_id`, `Prd_key`, `Prd_nm`, `Prd_start_dt` |
| `sls_` | CRM sales | `sls_ord_num`, `sls_prd_key`, `sls_order_dt`, `sls_quantity` |
| `erp_` | ERP fields | `erp_CID`, `erp_BDATE`, `erp_CNTRY`, `erp_CAT` |

These are observed source-oriented styles. They help distinguish similar fields and preserve data lineage. Existing capitalization is mixed; normalize new warehouse-created identifiers to lowercase snake case while retaining original source mappings when documented.

## 5. Dictionary suffixes and abbreviations observed

| Token/suffix | Observed meaning | Examples | Guidance |
| --- | --- | --- | --- |
| `_id` | Identifier | `Cst_id`, `Prd_id`, `sls_cust_id` | Use for a documented entity identifier; does not automatically mean a primary key. |
| `_key` | Matching/business key in source fields | `Cst_key`, `Prd_key`, `sls_prd_key` | Define whether the key is business, natural, or surrogate; do not infer from name alone. |
| `_nm` | Name | `Prd_nm` | Prefer full `name` for new curated fields, unless preserving source name. |
| `_dt` | Date/date-like source value | `Prd_start_dt`, `sls_order_dt` | Use a date SQL type in curated layers and document source encoding. |
| `_gndr` / `GEN` | Gender | `Cst_gndr`, `erp_GEN` | Keep restricted personal data appropriately protected. |
| `_ord_num` | Order number | `sls_ord_num` | Document grain/uniqueness. |
| `_sales` | Sales amount/value | `sls_sales` | Define currency, included components, and sign behavior. |
| `_quantity` | Quantity measure | `sls_quantity` | Document unit of measure. |
| `_price` | Price measure | `sls_price` | Document unit/currency/precision. |
| `BDATE` | Birth date abbreviation | `erp_BDATE` | Prefer `birth_date` for new curated business-facing fields. |
| `CNTRY` | Country abbreviation | `erp_CNTRY` | Prefer `country` in curated output. |
| `CAT`, `SUBCAT` | Category/subcategory abbreviations | `erp_CAT`, `erp_SUBCAT` | Prefer full descriptive names in curated output. |

An abbreviation is not a full business definition and does not guarantee a SQL data type. In particular, `_key` does not prove uniqueness and `_dt` does not make an integer column a date.

## 6. Gold alias style

Gold aliases aim to be business-friendly but currently mix capitalization and styles, e.g. `Product_key`, `customer_id`, `Order_date`, `Sales`. That is an observed inconsistency.

Recommended style for newly introduced Gold fields is lowercase `snake_case` and explicit meaning, e.g. `customer_key`, `product_key`, `order_date`, `sales_amount`, `quantity`, `unit_price`. If consumers already depend on established aliases, document and migrate them deliberately rather than renaming silently.

## 7. Procedure and SQL file styles

Observed procedure names include `dbo.Load_bronze` and `silver.load_silver`; the capitalization differs. A consistent convention for future procedures is `<schema>.load_<layer>` or an equally explicit verb-based name, all in lowercase.

Current script names include spaces, mixed case, and vague descriptions (`ddl bronze script`, `Gold_view_ddl_script`). These are existing filenames, not a recommended standard. New SQL files should use lowercase `snake_case.sql`, descriptive actions, and layer names, for example `load_bronze_crm.sql` or `create_gold_views.sql`.

Documentation files should use lowercase `snake_case.md`, e.g. `data_integration.md`, `naming_conventions.md`. Avoid spaces and spelling variations.

## 8. Recommended conventions for new objects

1. Use uppercase `BRONZE`, `SILVER`, and `GOLD` schema names consistently.
2. Use lowercase `snake_case` for new normalized object and column names.
3. Keep `crm_` and `erp_` lineage prefixes in source-aligned Bronze/Silver objects.
4. Use `dim_`/`fact_` for Gold dimensional objects.
5. Prefer full, meaningful names in curated layers; retain source abbreviations only when needed for lineage.
6. Reserve `_id` for identifiers, `_key` for documented business/surrogate keys, `_date` for date fields, and explicit names such as `sales_amount` and `unit_price` for measures.
7. Document abbreviations, aliases, exceptions, and business definitions alongside the relevant SQL and documentation.
8. Check downstream dependencies before renaming any published object or field.

## 9. Examples

| Existing name | Observed style | Recommended new style |
| --- | --- | --- |
| `BRONZE.crm_cust_info` | Uppercase schema + source/entity snake-like table | Retain layer/schema pattern. |
| `Cst_firstname` | Mixed-case prefix and compact attribute | Preserve in raw source mapping; use `cst_first_name` in a newly normalized table if appropriate. |
| `sls_order_dt` | Sales prefix + abbreviated date suffix | Use `sls_order_date` for new curated fields. |
| `GOLD.dim_customers` | Layer schema + dimension prefix | Retain `dim_` pattern. |
| `Product_key` | Mixed-case Gold alias | `product_key`. |
| `Order_date` | Capitalized alias with underscore | `order_date`. |

## Summary

The data dictionary uses source/subject prefixes (`Cst_`, `Prd_`, `sls_`, `erp_`), compact abbreviations (`nm`, `dt`, `BDATE`, `CNTRY`), and identifier/measure suffixes (`_id`, `_key`, `_sales`, `_quantity`, `_price`). Apply these consistently while preserving the provenience of source fields and moving toward clear, documented names in curated warehouse objects.
