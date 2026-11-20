# Data Integration

## How Data Is Integrated

The warehouse integrates CRM and ERP CSV extracts through a manual batch process:

```text
CRM/ERP CSV files
        ↓
Bronze (raw data loaded with BULK INSERT)
        ↓
Silver (cleaned and standardized data)
        ↓
Gold (reporting views)
```

The source files are loaded into Bronze tables, cleaned and transformed into Silver tables, and combined into Gold views for reporting.

| Source | File | Bronze table | Silver table | Gold use |
| --- | --- | --- | --- | --- |
| CRM | `cust_info.csv` | `BRONZE.crm_cust_info` | `SILVER.crm_cust_info` | Customer dimension |
| CRM | `prd_info.csv` | `BRONZE.crm_prd_info` | `SILVER.crm_prd_info` | Product dimension |
| CRM | `sales_details.csv` | `BRONZE.crm_sales_details` | `SILVER.crm_sales_details` | Sales fact |
| ERP | `CUST_AZ12.csv` | `BRONZE.erp_CUST_AZ12` | `SILVER.erp_CUST_AZ12` | Customer details |
| ERP | `loc_A101.csv` | `BRONZE.erp_loc_A101` | `SILVER.erp_loc_A101` | Customer countries |
| ERP | `PX_CAT_G1V2.csv` | `BRONZE.erp_PX_CAT_G1V2` | `SILVER.erp_PX_CAT_G1V2` | Product categories |

## Integration Rules

- Bronze loads truncate each target table before inserting the source data.
- Silver cleans names, dates, identifiers, and codes from the Bronze tables.
- Gold combines CRM and ERP data using customer and product keys.
- Left joins preserve unmatched records and allow missing ERP enrichment to be identified.
- Data quality checks should run after Bronze, Silver, and Gold processing.

## Related Documents

- [Architecture](architecture.md)
- [Data Models](data_models.md)
- [Data Catalog](data_catalog.md)
- [Quality Checks](quality_checks.md)
