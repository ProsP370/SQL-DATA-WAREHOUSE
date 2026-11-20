# Data Flow

This diagram shows how data moves from the CRM and ERP source systems through the warehouse to reporting-ready tables.

## Flow overview

1. **Sources:** CRM provides sales, customer, and product data; ERP provides customer, location, and product-category data.
2. **Bronze layer:** Raw source data is first loaded into the warehouse with minimal changes.
3. **Silver layer:** Data is cleaned and standardized so it can be joined consistently.
4. **Gold layer:** Curated tables support analysis: `fact_sales` stores sales activity, while `dim_customers` and `dim_products` provide descriptive context.

Arrows indicate the direction of data movement and how source tables contribute to the final tables.

## Data flow diagram

<img width="1538" height="962" alt="Data flow diagram" src="https://github.com/user-attachments/assets/c1fa7ebc-e1c8-4b0c-a231-d634a17eb4d6" />
