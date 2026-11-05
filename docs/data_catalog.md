# Data Catalog

## Purpose and interpretation

This catalog is the project’s reference for warehouse layers, datasets, fields, intended grain, source lineage, and known caveats. Bronze names/types are taken from the declared Bronze DDL. Silver and Gold descriptions represent intended transformations and view outputs; reconcile names and types against executable SQL because current scripts contain inconsistencies.

## Layer catalog

| Layer | Schema | Purpose | Current behavior |
| --- | --- | --- | --- |
| Raw landing | `BRONZE` | Hold CSV-derived source records | `BULK INSERT`; tables are truncated before the current full refresh. |
| Prepared | `SILVER` | Standardize values and prepare joins | Stored procedure transforms Bronze; no historical snapshot is maintained. |
| Reporting | `GOLD` | Expose business-facing dimensions and facts | SQL views; query results are not materialized. |

## Object catalog

| Object | Intended grain | Description / source |
| --- | --- | --- |
| `BRONZE.crm_cust_info` | Source customer record | CRM customer extract, `cust_info.csv`. |
| `BRONZE.crm_prd_info` | Source product/version record | CRM product extract, `prd_info.csv`. |
| `BRONZE.crm_sales_details` | Source sales detail | CRM transaction extract, `sales_details.csv`. |
| `BRONZE.erp_CUST_AZ12` | ERP customer reference row | Customer birthdate/gender, `CUST_AZ12.csv`. |
| `BRONZE.erp_loc_A101` | ERP location reference row | Customer country, `loc_A101.csv`. |
| `BRONZE.erp_PX_CAT_G1V2` | ERP product category reference row | Category metadata, `PX_CAT_G1V2.csv`. |
| `SILVER.crm_cust_info` | Latest selected customer record per ID | CRM customer values after trimming/mapping/deduplication. |
| `SILVER.crm_prd_info` | Product record/version | CRM product values after key, line, cost, and validity transformations. |
| `SILVER.crm_sales_details` | Sales detail | CRM sales values with date and measure transformations. |
| `SILVER.erp_CUST_AZ12` | ERP customer reference | Normalized customer ID, birthdate, and gender. |
| `SILVER.erp_loc_A101` | ERP location reference | Normalized customer ID and country. |
| `SILVER.erp_PX_CAT_G1V2` | ERP category reference | Category, subcategory, maintenance attributes. |
| `GOLD.dim_customers` | Intended one row per customer | Customer profile enriched with ERP attributes. |
| `GOLD.dim_products` | Intended product or product-version row | Product details enriched with ERP category attributes. |
| `GOLD.fact_sales` | Intended one row per sales-detail record; confirm | Sales dates/measures joined to customer and product views. |

## CRM customer fields (`BRONZE.crm_cust_info`)

| Field | Declared type | Meaning and notes |
| --- | --- | --- |
| `Cst_id` | `int` | Customer identifier; candidate business key. |
| `Cst_key` | `nvarchar(50)` | Customer matching/business key used for cross-source joins. |
| `Cst_firstname` | `nvarchar(50)` | First name; trimmed in Silver. Personal data. |
| `Cst_lastname` | `nvarchar(50)` | Last name; trimmed in Silver. Personal data. |
| `Cst_marital_status` | `nvarchar(50)` | Source status code; Silver maps `S`/`M` and otherwise emits `n/a`. |
| `Cst_gndr` | `nvarchar(50)` | Source gender code; Silver maps `F`/`M` and otherwise emits `n/a`. Personal data. |
| `Cst_create_date` | `date` | Customer record creation date; used to select latest record. |

## CRM product fields (`BRONZE.crm_prd_info`)

| Field | Declared type | Meaning and notes |
| --- | --- | --- |
| `Prd_id` | `int` | Product identifier. |
| `Prd_key` | `nvarchar(50)` | Product matching/business key; parsed in Silver. |
| `Prd_nm` | `nvarchar(50)` | Product display name. |
| `Prd_cost` | `int` | Product cost; missing values currently become zero in Silver. Currency/units are unspecified. |
| `Prd_line` | `nvarchar(50)` | Source line code; mapped to descriptive label in Silver. |
| `Prd_start_dt` | `date` | Product validity start date. |
| `Prd_end_dt` | `date` | Product validity end; Silver derives end dates from subsequent starts in current logic. |

## CRM sales fields (`BRONZE.crm_sales_details`)

| Field | Declared type | Meaning and notes |
| --- | --- | --- |
| `sls_ord_num` | `nvarchar(50)` | Order identifier; does not by itself prove a unique sales-line grain. |
| `sls_prd_key` | `nvarchar(50)` | Product reference used to join product data. |
| `sls_cust_id` | `int` | Customer reference used to join customer data. |
| `sls_order_dt` | `int` | Source date encoded as integer, expected `YYYYMMDD`. |
| `sls_ship_dt` | `int` | Source ship date encoded as integer, expected `YYYYMMDD`. |
| `sls_due_dt` | `int` | Source due date encoded as integer, expected `YYYYMMDD`. |
| `sls_sales` | `int` | Sales value; Silver may recalculate. Currency and gross/net definition need owner confirmation. |
| `sls_quantity` | `int` | Quantity measure. Unit is not documented. |
| `sls_price` | `int` | Price measure; currency and precision need confirmation. |

## ERP fields

| Object.field | Declared type | Meaning and notes |
| --- | --- | --- |
| `erp_CUST_AZ12.erp_CID` | `nvarchar(50)` | ERP customer identifier; Silver removes `NAS` prefix where present. |
| `erp_CUST_AZ12.erp_BDATE` | `date` | Customer birthdate; future dates become `NULL`. Personal data. |
| `erp_CUST_AZ12.erp_GEN` | `nvarchar(50)` | ERP gender value; standardized in Silver. Personal data. |
| `erp_loc_A101.erp_CID` | `nvarchar(50)` | ERP customer reference identifier; hyphens removed in Silver. |
| `erp_loc_A101.erp_CNTRY` | `nvarchar(50)` | Country code/value; selected codes are mapped to country names. |
| `erp_PX_CAT_G1V2.erp_ID` | `nvarchar(50)` | ERP product/category reference identifier. |
| `erp_PX_CAT_G1V2.erp_CAT` | `nvarchar(50)` | Product category label. |
| `erp_PX_CAT_G1V2.erp_SUBCAT` | `nvarchar(50)` | Product subcategory label. |
| `erp_PX_CAT_G1V2.erp_MAINTENANCE` | `nvarchar(50)` | Maintenance classification. |

## Gold view field intent

- `GOLD.dim_customers`: generated customer key, customer ID/number, first/last name, country, marital status, gender, birthdate, and create date.
- `GOLD.dim_products`: generated product key, product ID/number/name, category fields, cost, product line, and start date.
- `GOLD.fact_sales`: order number, generated dimension keys, order/ship/due dates, sales, quantity, and price.

Gold surrogate keys are generated by `ROW_NUMBER()` in views and are not stable identifiers. Confirm exact aliases against the Gold SQL and repaired Silver schema before using them in downstream contracts.

## Business metadata to complete

For each field/table, add owner/steward, sensitivity classification, authoritative source, refresh expectation, retention period, nullable/required status, allowed values, and downstream consumers. Business owners must define sales currency/semantics, fact grain, and treatment of returns before the catalog can be considered a complete business glossary.

## Related documents

- [Data integration](data_integration.md)
- [Data models](data_models.md)
- [Naming conventions](naming_conventions.md)
- [Quality checks](quality_checks.md)
