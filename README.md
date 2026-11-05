# SQL Data Warehouse

A SQL Server data warehouse project that brings together customer, product, sales, and ERP reference data. It uses a three-layer architecture—Bronze, Silver, and Gold—to load raw CSV files, clean and standardize them, and publish business-friendly views for analytics.

> **Project status:** This repository contains SQL scripts and quality-check queries. The `dataset/` directory currently contains a placeholder, not the source CSV files. You will need to obtain the required data files and configure their paths before running the load process.

## Table of contents

- [Project goals](#project-goals)
- [Architecture and data flow](#architecture-and-data-flow)
- [Data sources](#data-sources)
- [Gold reporting views](#gold-reporting-views)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [Setup and execution](#setup-and-execution)
- [Data quality checks](#data-quality-checks)
- [Important notes and limitations](#important-notes-and-limitations)
- [Future improvements](#future-improvements)
- [About me](#about-me)

## Project goals

This project demonstrates how to build a SQL-based analytical warehouse that:

- loads CRM and ERP CSV data into a raw landing layer;
- cleans, deduplicates, and standardizes data in a transformation layer;
- combines related data into customer and product dimensions and a sales fact view;
- provides SQL quality checks to help identify missing, duplicate, inconsistent, or unmatched data.

## Architecture and data flow

```text
CRM and ERP CSV files
        |
        v
Bronze schema: raw imported source data
        |
        v
Silver schema: cleaned and standardized tables
        |
        v
Gold schema: reporting views for analytics
```

### Bronze layer

The Bronze layer stores source data with minimal transformation. Its tables cover CRM customers, products, and sales, plus ERP customer, location, and product-category reference data. The load procedure truncates and reloads the tables from CSV files using SQL Server `BULK INSERT`.

### Silver layer

The Silver layer contains data prepared for reliable joins and analysis. The load procedure includes transformations such as:

- trimming customer names and standardizing marital status and gender values;
- deriving category and product keys and mapping product-line codes to readable labels;
- validating sales dates and converting valid date values;
- recalculating inconsistent sales amounts and deriving invalid or missing prices where possible;
- removing known prefixes or separators from ERP identifiers;
- standardizing country values and handling future birthdates;
- selecting the latest customer record based on its creation date.

### Gold layer

The Gold layer consists of views intended for reporting and analytics. It brings together the Silver tables as two dimensions and a sales fact view. These views expose descriptive names and measures without creating a separate stored copy of the query results.

## Data sources

The SQL scripts expect six CSV source files:

| Source area | Expected file | Contents |
| --- | --- | --- |
| CRM | `cust_info.csv` | Customer attributes and creation dates |
| CRM | `prd_info.csv` | Product details and product validity dates |
| CRM | `sales_details.csv` | Orders, customers, products, dates, quantities, and sales values |
| ERP | `CUST_AZ12.csv` | Customer birthdate and gender reference data |
| ERP | `loc_A101.csv` | Customer location/country reference data |
| ERP | `PX_CAT_G1V2.csv` | Product category, subcategory, and maintenance classifications |

The repository does not currently include these CSVs. Place them in a location accessible to the SQL Server service and update the file paths in the Bronze load procedure.

## Gold reporting views

The Gold DDL script creates the following views:

### `Gold.dim_products`
A product dimension with product identifiers and names, category and subcategory, maintenance classification, cost, product line, and start date.

### `Gold.dim_customers`
A customer dimension with customer identifiers and numbers, names, country, marital status, gender, birthdate, and record creation date.

### `Gold.fact_sales`
A sales view with order number, product and customer keys, order/ship/due dates, sales amount, quantity, and price.

## Repository structure

```text
.
├── dataset/                         # Source data location (currently a placeholder)
├── docs/                            # Project documentation
├── script/
│   ├── Init-sql-database            # Database and schema initialization script
│   ├── bronze/                      # Bronze table DDL and CSV load procedure
│   ├── silver/                      # Silver table DDL and transformation procedure
│   └── Gold/                        # Gold reporting view definitions
└── tests/
    ├── silver-data-quality          # Checks for raw Bronze data before Silver load
    └── gold_quality_checks          # Checks for Gold views after creation
```

## Requirements

- Microsoft SQL Server with permission to create databases, schemas, tables, procedures, and views.
- SQL Server Management Studio (SSMS) or another SQL client that supports SQL Server scripts and `GO` batch separators.
- The six CSV files listed above.
- SQL Server service access to the folder containing the CSV files, and permission to run `BULK INSERT`.

## Setup and execution

Run the scripts in this order, reviewing and adapting them for your environment:

1. **Initialize the database and schemas.** Review `script/Init-sql-database` before running it. It is destructive: it drops and recreates the `DATAWAREHOUSE` database if it already exists.
2. **Create Bronze tables.** Run `script/bronze/ddl bronze script`.
3. **Configure source paths.** Edit the `BULK INSERT` file paths in `script/bronze/proc-stored-procedure` to point to your CSV files. The paths currently refer to a particular local Windows machine and will not work unchanged in other environments.
4. **Load Bronze.** Run the Bronze procedure/script to load the six CSV files into the Bronze tables.
5. **Check the raw data.** Run `tests/silver-data-quality` and investigate any returned rows before transforming the data.
6. **Create Silver tables.** Run `script/silver/ddl silver script`.
7. **Transform Bronze to Silver.** Run `script/silver/proc-silver-stored-procedure` to create/execute the Silver load procedure.
8. **Create Gold views.** Run `script/Gold/Gold_view_ddl_script`.
9. **Validate Gold.** Run `tests/gold_quality_checks` and review any returned rows.

Use a development or test database first. Do not run the initialization script against a database containing data you need to keep.

## Data quality checks

The tests are SQL queries that report records needing review; they are not an automated pass/fail test suite. A check generally indicates a potential issue when it returns rows.

- `tests/silver-data-quality` checks Bronze CRM and ERP data for missing or duplicate identifiers, blank values, unexpected codes, invalid dates, and inconsistent sales values. Run it before the Silver load.
- `tests/gold_quality_checks` checks Gold products, customers, and sales for missing or duplicate keys, missing names/measures, unmatched dimension references, and date-order issues. Run it after creating the Gold views.

## Important notes and limitations

- **Initialization is destructive.** The initialization script drops and recreates `DATAWAREHOUSE` if it already exists. Back up any data you need and inspect the script before running it.
- **CSV files are not included.** The `dataset/` folder is currently a placeholder, so the load procedure cannot run until the files are supplied.
- **File paths are machine-specific.** Update the hardcoded `BULK INSERT` paths. The SQL Server service account, not just the interactive user, must be able to read those files.
- **Loads are full refreshes.** The procedures truncate target tables before loading; they are not incremental loads and are not designed to preserve prior target contents.
- **Gold surrogate keys are generated with `ROW_NUMBER()` in views.** These keys are calculated when a view is queried and are not persisted identifiers; their values can change when source rows or ordering change. Avoid treating them as stable keys across refreshes without reviewing this design.
- **Validate scripts against your SQL Server environment.** Review database names, schema/object references, permissions, source formats, and SQL syntax before running the scripts.

## Future improvements

Potential next steps for the project include:

- making source paths configurable instead of hardcoded;
- adding ETL run logging, error handling, and load audit tables;
- adding repeatable automated test execution and explicit pass/fail results;
- implementing incremental loads where appropriate;
- persisting stable dimension keys if downstream consumers require them;
- documenting the source-to-target mapping and business definitions in more detail.

## About me

Hi, I'm **[Your name]**. I work on data and SQL projects, including this SQL data warehouse. **[Add a short bio, your interests, or what you are learning here.]**

Connect with me:

- GitHub: [@ProsP370](https://github.com/ProsP370)
- LinkedIn: [Add your LinkedIn profile URL]
- [Other social platform]: [Add your profile URL or handle]

> Replace the bracketed text with your preferred bio and social media links before publishing.
