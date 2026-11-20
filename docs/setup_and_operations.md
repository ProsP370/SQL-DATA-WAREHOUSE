# Setup and Operations

## Requirements

- Microsoft SQL Server
- SQL Server Management Studio or another T-SQL client
- Permissions to create databases, schemas, tables, procedures, and views
- The six source CSV files
- A source folder accessible by the SQL Server service account

The CSV files are not included in this repository.

## Before Running

- Back up or use an approved development database.
- Review all SQL scripts before execution.
- Update source paths in `script/bronze/proc-stored-procedure`.
- Confirm CSV names, headers, encoding, delimiters, dates, and null values.

Warning: database, DDL, and load scripts can drop or overwrite warehouse objects and data.

## Run in This Order

1. Run `script/Init-sql-database` to create the database and schemas.
2. Run `script/bronze/ddl bronze script` to create Bronze tables.
3. Run `script/bronze/proc-stored-procedure` to create and execute the Bronze loader.
4. Run `tests/silver-data-quality` to check Bronze results.
5. Run `script/silver/ddl silver script` to create Silver tables.
6. Run `script/silver/proc-silver-stored-procedure` to create and execute `silver.load_silver`.
7. Run `script/Gold/Gold_view_ddl_script` to create Gold views.
8. Run `tests/gold_quality_checks` to validate Gold output.

## Rerunning Loads

Bronze and Silver load procedures replace existing layer data. Reloading does not preserve previous snapshots.

After a failure:

1. Check the SQL error and affected target tables.
2. Verify source file names and content.
3. Confirm target row counts.
4. Reload only when a complete, safe rerun is possible.
5. Run validation before publishing Gold data.

## Troubleshooting

- **CSV access denied:** Check the SQL Server service account, server-side path, file name, and folder permissions.
- **CSV conversion error:** Compare values and column layout with the Bronze table definitions.
- **Silver load failure:** Check database context, table columns and types, procedure logic, and permissions.
- **Gold view failure:** Verify Silver tables and columns exist and join keys have compatible types.
- **Unexpected quality results:** Review source data, transformation rules, joins, and approved exceptions.
- **Gold totals are too high:** Check duplicate keys and join cardinality.

## Security

Use approved data only. Protect customer names, birth dates, gender, marital status, and country fields. Follow least-privilege access and do not commit sensitive source files or credentials.

## Related Documents

- [Architecture](architecture.md)
- [Data integration](data_integration.md)
- [Data models](data_models.md)
- [Data catalog](data_catalog.md)
- [Quality checks](quality_checks.md)
