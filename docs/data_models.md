# Data Models

## Purpose and scope

This document describes the business entities, intended dimensional model, row grain, relationships, and key assumptions. It distinguishes the intended model from the current SQL implementation, which has schema/column mismatches that must be reconciled before execution is considered validated.

## Model diagram photo slot

> **Photo placeholder:** Upload the model/ERD image later at `docs/images/data_model.png`. Then replace this note with: `![Warehouse data model](images/data_model.png)`.

## Business entities

- **Customer:** CRM customer profile, optionally enriched with ERP birthdate, gender, and country.
- **Product:** CRM product record/version, optionally enriched with ERP category, subcategory, and maintenance attributes.
- **Sales:** CRM sales-detail records associated with customer and product records.

## Intended Gold dimensional model and grain

### `GOLD.dim_customers`

**Intended grain:** one row per resolved CRM customer. Intended attributes: customer key, customer ID and business number, first and last name, country, marital status, gender, birthdate, and record creation date. ERP fields are enrichment attributes; confirm whether enrichment is optional and how multiple ERP matches are handled.

### `GOLD.dim_products`

**Intended grain:** one row per product, or one row per product version when historical reporting is required. Intended attributes: product key, product ID/number/name, category ID/name, subcategory, maintenance classification, cost, product line, and start date. Define the product history and effective-date policy.

### `GOLD.fact_sales`

**Intended grain:** one row per CRM sales-detail record, subject to confirmation against the source. `sls_ord_num` may identify an order rather than a unique line, so it may not alone identify a fact row. Intended fields include order number, customer and product dimension keys, order/ship/due dates, sales amount, quantity, and price.

Document currency, units, gross/net basis, discount/tax inclusion, returns/cancellations, sign behavior, and rounding before interpreting measures.

## Relationships and join rules

- Sales customer reference → customer dimension business identifier.
- Sales product reference → product dimension product number.
- CRM customer business key → ERP customer and location references.
- CRM product category key → ERP product category reference.

Check uniqueness of every right-hand join key. Duplicate reference keys can fan out joins and inflate fact counts and measures. Measure unmatched keys and define whether they remain unassigned or map to an explicit unknown member.

## Keys and history

The current Gold view SQL generates customer/product keys with `ROW_NUMBER()`. These are query-time values, not stable keys for downstream persistence. If stable identifiers or point-in-time reporting are needed, materialize dimensions and define key assignment, updates, late-arriving records, and slowly changing dimension behavior.

## Assumptions to confirm

1. Exact sales fact natural key and row grain.
2. Whether customer/product source versions and duplicate reference rows are expected.
3. Product effective-date and history behavior.
4. Definitions, currency, and units for all measures.
5. Required versus optional attributes and ERP enrichment.
6. Handling of unknown, invalid, unmatched, and late-arriving records.
7. Retention and point-in-time reporting needs.

## Current SQL contract caveats

The checked-in Silver procedure writes `cat_id` to `SILVER.crm_prd_info`, but the Silver DDL shown in the repository does not define `cat_id`. Gold SQL joins `pc.id`, `ci.cid`, and `la.cid`, while corresponding ERP DDL declares `erp_ID` and `erp_CID`. Silver sales dates are transformed as dates, but the current Silver DDL declares those columns as integers. Align DDL, transformations, and Gold view references before treating this model as executable.

## Related documents

- [Architecture](architecture.md)
- [Data integration](data_integration.md)
- [Data flow](data_flow.md)
- [Gold data catalog](data_catalog.md)
- [Quality checks](quality_checks.md)
