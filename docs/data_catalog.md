# Gold Data Catalog

## Purpose

This catalog describes the **Gold layer** only. It documents the business-ready views used for reporting and analysis.

The Gold layer combines cleaned data from Silver and presents it in an easy-to-use format:

- **Customer dimension** — customer information
- **Product dimension** — product and category information
- **Sales fact** — sales transactions and measures

> Bronze and Silver objects are not described in this catalog.

## Gold Objects

| Gold view | Type | Business purpose |
| --- | --- | --- |
| `GOLD.dim_customers` | Dimension | Contains one reporting row for each customer. |
| `GOLD.dim_products` | Dimension | Contains one reporting row for each product. |
| `GOLD.fact_sales` | Fact | Contains sales transactions and their customer and product references. |

## Customer Dimension

**View:** `GOLD.dim_customers`

This view combines CRM customer data with ERP birthdate and country information.

| Column | Meaning |
| --- | --- |
| `Customer_key` | Reporting key generated for the customer dimension. |
| `customer_id` | CRM customer identifier. |
| `customer_number` | Customer key used to match customer information across systems. |
| `first_name` | Customer's first name. |
| `last_name` | Customer's last name. |
| `Country` | Customer country from ERP location data. |
| `marital_status` | Customer marital status. |
| `gender` | Customer gender. |
| `birthdate` | Customer birthdate from ERP data. |
| `create_date` | Date the customer record was created. |

**Simple join logic:**

- CRM customer data is the starting point.
- ERP birthdate data is matched using `customer_number` and `erp_cid`.
- ERP country data is matched using `customer_number` and `erp_cid`.
- Unmatched CRM customers remain in the result.

## Product Dimension

**View:** `GOLD.dim_products`

This view combines CRM product data with ERP category information.

| Column | Meaning |
| --- | --- |
| `Product_key` | Reporting key generated for the product dimension. |
| `Product_id` | Internal CRM product identifier. |
| `Product_number` | Product number used for matching and reporting. |
| `Product_name` | Name of the product. |
| `Category_id` | Product category identifier. |
| `Category` | Product category name. |
| `Subcategory` | Product subcategory name. |
| `Maintenance` | Product maintenance classification. |
| `Cost` | Product cost. |
| `Product_line` | Product line or brand category. |
| `Start_date` | Date the product became available. |

**Simple join logic:**

- CRM product data is the starting point.
- ERP category data is matched using `Category_id` and `erp_id`.
- Unmatched CRM products remain in the result.

## Sales Fact

**View:** `GOLD.fact_sales`

This view contains the sales transactions available for reporting.

| Column | Meaning |
| --- | --- |
| `Order_number` | Sales order identifier. |
| `Product_key` | Reference to `GOLD.dim_products.Product_number`. |
| `Customer_key` | Reference to `GOLD.dim_customers.customer_id`. |
| `Order_date` | Date the order was placed. |
| `Ship_date` | Date the order was shipped. |
| `Due_date` | Date the order was due. |
| `Sales` | Sales amount for the transaction. |
| `Quantity` | Quantity sold. |
| `Price` | Price used for the transaction. |

**Simple join logic:**

- Sales records start from the Silver CRM sales table.
- Product keys match the product dimension by product number.
- Customer keys match the customer dimension by customer ID.

## Reporting Model

```text
GOLD.dim_customers     GOLD.dim_products
        │                      │
        └──────────┬───────────┘
                   │
             GOLD.fact_sales
```

- `GOLD.fact_sales` contains the business measures.
- `GOLD.dim_customers` provides customer details.
- `GOLD.dim_products` provides product details.

## Important Notes

- Gold objects are SQL views. They return results when queried but do not store a separate copy of the data.
- Dimension keys generated with `ROW_NUMBER()` are created when the view is read. They should not be treated as permanent database identifiers.
- `GOLD.fact_sales` does not currently enforce database-level foreign-key constraints.
- Prices, currencies, sales calculations, and return behavior should be confirmed with the business owner before financial reporting.
- The customer and product views use left joins so that base CRM records are retained when ERP enrichment is unavailable.

## Using the Gold Layer

Example: to view customer sales, join the fact and dimensions by their keys:

```sql
SELECT
    s.Order_number,
    c.first_name,
    c.last_name,
    p.Product_name,
    s.Sales,
    s.Quantity
FROM GOLD.fact_sales AS s
LEFT JOIN GOLD.dim_customers AS c
    ON s.Customer_key = c.customer_id
LEFT JOIN GOLD.dim_products AS p
    ON s.Product_key = p.Product_number;
```

## Related Documents

- [Architecture](architecture.md)
- [Data Integration](data_integration.md)
- [Data Models](data_models.md)
- [Quality Checks](quality_checks.md)
