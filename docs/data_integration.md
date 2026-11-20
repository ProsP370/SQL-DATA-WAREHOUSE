# Data Integration

## How Data Is Integrated

CRM and ERP data are integrated to create the warehouse dimensions and fact data. The CRM system supplies sales transactions and core customer and product records, while the ERP system supplies additional customer and product attributes.

<img width="1570" height="1022" alt="Relatonal tables" src="https://github.com/user-attachments/assets/75eb1f46-4d9d-4d16-94bf-408955f1147f" />


## Source and Warehouse Mapping

| Source | Source file | Bronze table | Silver table | Gold use |
| --- | --- | --- | --- | --- |
| CRM | `cust_info.csv` | `BRONZE.crm_cust_info` | `SILVER.crm_cust_info` | Customer dimension |
| CRM | `prd_info.csv` | `BRONZE.crm_prd_info` | `SILVER.crm_prd_info` | Product dimension |
| CRM | `sales_details.csv` | `BRONZE.crm_sales_details` | `SILVER.crm_sales_details` | Sales fact |
| ERP | `CUST_AZ12.csv` | `BRONZE.erp_CUST_AZ12` | `SILVER.erp_CUST_AZ12` | Customer details |
| ERP | `loc_A101.csv` | `BRONZE.erp_loc_A101` | `SILVER.erp_loc_A101` | Customer countries |
| ERP | `PX_CAT_G1V2.csv` | `BRONZE.erp_PX_CAT_G1V2` | `SILVER.erp_PX_CAT_G1V2` | Product categories |

## Integration Relationships

- `crm_sales_details.Prd_key` links sales transactions to `crm_prd_info.Prd_Key` for product analysis.
- `crm_sales_details.cst_id` links sales transactions to `crm_cust_info.cst_id` for customer analysis.
- `crm_cust_info.cst_key` is linked to `erp_cust_az12.erp_cid` to add extra customer information, including the birth date.
- `crm_cust_info.cst_key` is also linked to `erp_loc_a101.erp_cid` to add the customer's country.
- `crm_prd_info.Prd_Key` is linked to `erp_px_cat_g1v2.id` to add product category information.

## Integration Rules

- Bronze loads truncate each target table before inserting the source data.
- Silver cleans names, dates, identifiers, codes, and text fields from the Bronze tables.
- Gold left joins CRM records with ERP enrichment so unmatched source records are preserved.
- Customer and product keys must be standardized before joins are applied.
- CRM values provide the primary dimensions and facts; ERP values enrich those records.
- Data quality checks should run after Bronze, Silver, and Gold processing.

## Related Documents

- [Architecture](architecture.md)
- [Data Models](data_models.md)
- [Data Catalog](data_catalog.md)
- [Quality Checks](quality_checks.md)
