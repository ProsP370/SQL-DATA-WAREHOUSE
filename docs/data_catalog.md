# Gold Data Catalog

## Scope

This catalog documents the **Gold reporting views only**. It describes the output aliases and intended meaning from `script/Gold/Gold_view_ddl_script`; it is not a catalog of Bronze or Silver fields. The Gold objects are views, so their results are computed when queried rather than stored as separate table rows.

## Gold object summary

| Object | Intended grain | Description |
|---|---|---|
| `GOLD.dim_customers` | One row per resolved CRM customer, if enrichment keys are unique | Customer profile with country and ERP birthdate enrichment. |
| `GOLD.dim_products` | One row per product/version, depending on source product history | Product details with ERP category attributes. |
| `GOLD.fact_sales` | Intended one row per sales-detail record; confirm source grain | Sales dates/measures associated with customer and product keys. |

## `GOLD.dim_customers`

| Current view column | Meaning | Source/notes |
|---|---|---|
| `Customer_key` | Query-time generated customer row number | `ROW_NUMBER()` ordered by customer ID; not a stable key. |
| `customer_id` | CRM customer identifier | From Silver customer ID. |
| `customer_number` | CRM customer business/matching number | From Silver customer key. |
| `first_name` | Customer first name | Personal data. |
| `last_name` | Customer last name | Personal data. |
| `Country` | Customer country | Intended from ERP location enrichment. |
| `marital_status` | Normalized marital status | Current Silver procedure maps selected codes and uses `n/a` otherwise. |
| `gender` | Normalized gender | Current Gold view uses CRM gender; ERP gender is not selected by this view. Personal data. |
| `birthdate` | Customer date of birth | Intended from ERP customer enrichment. Sensitive personal data; restrict access. |
| `create_date` | CRM customer record creation date | From Silver customer record. |

## `GOLD.dim_products`

| Current view column | Meaning | Source/notes |
|---|---|---|
| `Product_key` | Query-time generated product row number | `ROW_NUMBER()` ordered by product start date and product key; not stable. |
| `Product_id` | CRM product identifier | From Silver product record. |
| `Product_number` | CRM product business/matching number | Used by the sales view to join the product. |
| `Product_name` | Product display name | From Silver product record. |
| `Category_id` | Derived product category identifier | Silver product field; not present in the checked-in Silver DDL. |
| `Category` | Category label | Intended from ERP category reference. |
| `Subcategory` | Subcategory label | Intended from ERP category reference. |
| `Maintenance` | Maintenance classification | Intended from ERP category reference. |
| `Cost` | Product cost | Currency/units are not documented. |
| `Product_line` | Product-line description | Current Silver procedure maps selected source codes. |
| `Start_date` | Product version/effective start date | From Silver product record. |

## `GOLD.fact_sales`

| Current view column | Meaning | Source/notes |
|---|---|---|
| `Order_number` | Source order identifier | May not uniquely identify a sales line; confirm fact grain. |
| `Product_key` | Product dimension key from the view join | Current query-time key; may change across query results. |
| `Customer_key` | Customer dimension key from the view join | Current query-time key; may change across query results. |
| `Order_date` | Sales order date | Silver transform intends to convert source YYYYMMDD integer. |
| `Ship_date` | Shipment date | Silver transform intends to convert source YYYYMMDD integer. |
| `Due_date` | Due date | Silver transform intends to convert source YYYYMMDD integer. |
| `Sales` | Sales amount | Silver may recalculate; currency and gross/net definition need business-owner confirmation. |
| `Quantity` | Sold quantity | Unit of measure is not documented. |
| `Price` | Price measure | Currency, precision, and unit basis are not documented. |

## Join, key, and usage caveats

Gold views use left joins so unmatched rows may remain with missing enrichment/dimension keys. Validate uniqueness of each dimension/reference join key to prevent fan-out. `ROW_NUMBER()` keys generated in views are not durable identifiers; do not persist them as long-lived foreign keys without changing the model.

The checked-in Gold SQL references `pc.id`, `ci.cid`, and `la.cid`, while the related DDL names the ERP fields `erp_ID` and `erp_CID`. The product `Category_id` also depends on Silver `cat_id`, which the checked-in Silver DDL does not define. Resolve these schema mismatches before treating this catalog's intended enrichment as verified runtime behavior. Also confirm fact grain, currency, units, measure semantics, data owner, classification, retention, and downstream consumers.

Customer names, gender, and birthdate are personal data. Apply least-privilege access and use masked/synthetic data for demonstrations.

## Related documents

- [Data models](data_models.md)
- [Data integration](data_integration.md)
- [Data flow](data_flow.md)
- [Naming conventions](naming_conventions.md)
