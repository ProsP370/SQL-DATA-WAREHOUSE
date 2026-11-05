# Data Flow

## Purpose

This page is reserved for the visual data-flow documentation. The flow should show the six CRM/ERP CSV inputs, the six Bronze landing tables, the six Silver prepared tables, the three Gold reporting views, and the validation/analytics steps. The visual will be added when the project photos/screenshots are uploaded.

## Data-flow photo slots

> **Photo placeholder 1 — end-to-end flow:** Upload the overall flow image at `docs/images/data_flow_overview.png`, then replace this note with: `![End-to-end warehouse data flow](images/data_flow_overview.png)`.

> **Photo placeholder 2 — layer details (optional):** Upload a detailed layer/pipeline image at `docs/images/data_flow_layers.png`, then replace this note with: `![Warehouse layer flow](images/data_flow_layers.png)`.

## Flow labels to include in the future visual

1. **CRM inputs:** `cust_info.csv`, `prd_info.csv`, `sales_details.csv`.
2. **ERP inputs:** `CUST_AZ12.csv`, `loc_A101.csv`, `PX_CAT_G1V2.csv`.
3. **Bronze:** source-oriented landing tables loaded by `dbo.Load_bronze`.
4. **Validation:** current Bronze checks are in `tests/silver-data-quality` (the filename is misleading).
5. **Silver:** standardized tables populated by `silver.load_silver`.
6. **Gold:** `GOLD.dim_customers`, `GOLD.dim_products`, `GOLD.fact_sales` views.
7. **Gold validation and consumers:** `tests/gold_quality_checks`, then SQL/BI analytics.

Show the current process as a manually run, full-refresh pipeline. Mark that the load procedures truncate targets, audit/history is not persisted, and schema/column mismatches must be reconciled before the diagram is presented as a verified executable flow.

## Related documents

- [Architecture](architecture.md)
- [Data integration](data_integration.md)
- [Data models](data_models.md)
