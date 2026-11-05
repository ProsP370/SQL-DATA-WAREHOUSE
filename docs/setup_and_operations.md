# Setup and Operations

## 1. Overview

This project is a SQL Server warehouse pattern that requires a working SQL Server environment, source CSV files, and a structured sequence of commands to create the database, load data, transform it, and validate the final output.

This document describes the operational workflow for running the project safely and consistently.

## 2. Prerequisites

You need:

- Microsoft SQL Server
- SQL Server Management Studio or another SQL client supporting T-SQL
- access to the six source CSV files
- permission to create databases, schemas, tables, views, and procedures
- access to the folder where the CSV files are stored

## 3. Required source files

The warehouse expects the following source CSVs:

- `cust_info.csv`
- `prd_info.csv`
- `sales_details.csv`
- `CUST_AZ12.csv`
- `loc_A101.csv`
- `PX_CAT_G1V2.csv`

These files are not included in the repository. They must be obtained and placed in a location that the SQL Server service account can access.

## 4. Database initialization

Use the script in `script/Init-sql-database` to create the `DATAWAREHOUSE` database and its schemas.

Important:

- the script is destructive;
- it drops and recreates the database if it already exists;
- back up any existing data before running it.

The script creates the following schemas:

- `BRONZE`
- `SILVER`
- `GOLD`

## 5. Execution order

Run the scripts in this order:

1. Initialize the database and schemas.
2. Create the Bronze tables.
3. Update the CSV paths in the Bronze load procedure.
4. Run the Bronze load procedure.
5. Run Bronze/source quality checks.
6. Create the Silver tables.
7. Execute the Silver transformation.
8. Create or refresh the Gold views.
9. Run Gold quality checks.
10. Use the Gold views for reporting.

## 6. Bronze load

The Bronze load procedure is in:

- `script/bronze/ddl bronze script`
- `script/bronze/proc-stored-procedure`

The procedure:

- truncates each Bronze table;
- loads the relevant CSV file using `BULK INSERT`;
- prints timings to the output window;
- uses a full-refresh pattern rather than incremental loading.

Current limitation: the file paths are specific to a machine and must be replaced for each new environment.

## 7. Silver transformation

The Silver transformation procedure is in:

- `script/silver/ddl silver script`
- `script/silver/proc-silver-stored-procedure`

This step applies the cleaning, normalization, deduplication, and mapping rules needed before the data becomes report-ready.

This is where the project standardizes values and prepares the data for joins and dimension logic.

## 8. Gold creation

The Gold reporting views are created by:

- `script/Gold/Gold_view_ddl_script`

This script creates:

- `GOLD.dim_products`
- `GOLD.dim_customers`
- `GOLD.fact_sales`

Before using these views in reporting, validate the data against the quality checks.

## 9. Operational best practices

### Use a test environment first

Always test the full flow in a dev or test environment before using the warehouse in a production-like environment.

### Keep source-path configuration outside code

Avoid hardcoding local workstation paths in scripts. A better pattern is to configure source folders in a controlled environment or document the file location as part of deployment instructions.

### Validate before trusting results

Do not assume a successful script run means the data is trustworthy. The project depends on input validity and transformation logic that must be checked.

### Log loads and exceptions

The current project prints output to the SQL console, but it does not yet store a formal run log. Recommended additions include:

- run ID;
- source file list;
- row counts before and after each load;
- start and end timestamps;
- error details for each failed step.

## 10. Troubleshooting common issues

### Problem: `BULK INSERT` cannot access the CSV file

Possible causes:

- wrong file path;
- wrong file name;
- SQL Server service account cannot access the folder;
- file encoding or delimiter mismatch.

### Problem: load succeeds but quality checks return rows

This usually means the source data has quality issues or the transformation logic did not fully correct them. Investigate the returned rows and decide whether to:

- correct the source data;
- update the transformation logic;
- document an allowed exception.

### Problem: Gold view fails to create

Check the following:

- the required Silver objects exist;
- the referenced columns exist in the target tables;
- the schema names and object names are spelled correctly;
- the data types and join conditions are compatible.

### Problem: a rerun unexpectedly removes data

This is expected with the current design because the scripts truncate tables before reloading. Confirm whether the target tables are intentionally being cleared before rerunning.

## 11. Recovery and rerun guidance

Because the project uses full-refresh patterns:

- always review the data before rerunning;
- check whether the previous load completed partially or failed after truncation;
- redeploy or re-run from a known good source state where possible;
- document what you changed before a full rerun.

## 12. Summary

This project is easy to run in a controlled environment and is a good example of a basic warehouse pipeline. It works best when the source files are valid, the scripts are reviewed before execution, and the data is validated after each major step. With that discipline, the project can produce a reliable Bronze–Silver–Gold data flow for reporting and learning.
