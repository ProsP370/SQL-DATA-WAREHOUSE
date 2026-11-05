# Setup and Operations

## Purpose

This guide describes prerequisites, safe setup, execution order, reruns, and troubleshooting for the SQL Server warehouse. The repository currently contains manually executed SQL scripts; it does not define an orchestrator or automated deployment process.

## Prerequisites

- Microsoft SQL Server and a T-SQL client such as SQL Server Management Studio that supports `GO`.
- Permissions required to create databases, schemas, tables, procedures, and views.
- The six source files: `cust_info.csv`, `prd_info.csv`, `sales_details.csv`, `CUST_AZ12.csv`, `loc_A101.csv`, `PX_CAT_G1V2.csv`.
- A source folder readable by the SQL Server service account.
- A development/test database or explicit approval and backup before destructive scripts.

The CSVs are not included in the repository. Confirm their header, column order, delimiter, encoding, date formats, and null representations before loading.

## Destructive-operation warning

`script/Init-sql-database` drops and recreates `DATAWAREHOUSE` if it already exists. Bronze and Silver DDL scripts drop and recreate tables; load procedures truncate targets. These operations can permanently remove data. Confirm the connected SQL Server and database, inspect the scripts, and back up required data. Do not run these scripts against valuable data without approval.

## Execution order

1. **Review environment and SQL.** Confirm database target, server permissions, source files, and code. Reconcile current DDL/procedure/view column and type mismatches before claiming the pipeline is executable.
2. **Initialize database.** Run `script/Init-sql-database` only in an approved environment; verify `DATAWAREHOUSE` and `BRONZE`, `SILVER`, `GOLD` schemas.
3. **Create Bronze tables.** Run `script/bronze/ddl bronze script`.
4. **Configure loader paths.** Update local paths in `script/bronze/proc-stored-procedure`. Confirm server-side access and source file contract.
5. **Create/run Bronze loader.** The script defines `dbo.Load_bronze` and includes an `EXEC Load_bronze` at the end, so executing the whole file also starts a load. Review before running.
6. **Validate Bronze.** Run `tests/silver-data-quality`. Despite the filename, it queries Bronze. Investigate results.
7. **Create Silver tables.** Run `script/silver/ddl silver script` after reviewing its drop behavior.
8. **Run Silver transform.** Create and execute `silver.load_silver` from `script/silver/proc-silver-stored-procedure`; verify output and target counts.
9. **Create Gold views.** Run `script/Gold/Gold_view_ddl_script` only after confirming referenced columns exist.
10. **Validate Gold.** Run `tests/gold_quality_checks`; review results before reporting use.

Test the complete sequence with representative files. Documentation alone does not prove that the scripts execute successfully.

## File access and configuration

The Bronze procedure contains hardcoded local Windows paths. Change them to paths accessible from the SQL Server host. SQL Server’s service account needs read permission; access from the interactive user running SSMS is insufficient. Avoid storing secrets or sensitive file contents in SQL files or Git.

## Full refresh and safe reruns

Bronze and Silver use truncate-and-load behavior. A rerun overwrites the current layer contents and does not preserve previous snapshots. After an error:

1. identify which target tables were truncated or loaded;
2. inspect SQL errors and per-table row counts;
3. verify the source files are complete and intended for this run;
4. decide whether a full reload is safe and restart from a known-good stage;
5. rerun validation before rebuilding/publishing Gold;
6. record the issue, resolution, and any accepted exceptions.

The current code does not provide persistent run IDs, full transaction/recovery semantics, or a durable audit trail. Do not assume a procedure’s printed output proves every table loaded correctly.

## Troubleshooting

### `BULK INSERT` cannot access a CSV

Check server-side path, file existence, service-account permissions, exact file name, encoding, delimiter, first-row setting, and source column order.

### Import/conversion failure

Compare CSV layout and values with Bronze DDL. Check invalid dates/numbers, quoted delimiters, embedded newlines, and unexpected null markers. Retain a safe sample for reproduction without exposing personal data.

### Silver transformation fails

Check database context, object creation order, column names/types, insert target lists, and permissions. Compare procedure expressions with Silver table definitions.

### Gold view fails

Verify that all referenced Silver tables and columns exist, join keys have compatible types, schema names are correct, and SQL batches are executed in the intended order. The current repository has apparent object/column mismatches, so reconcile the definitions first.

### Quality checks return rows

Review the rule and records. Determine whether the cause is source quality, transformation behavior, incorrect join cardinality, or an approved exception. Do not delete rows merely to make a test return empty.

### Gold totals are too high

Investigate duplicate keys on dimension/reference join sides and verify the fact grain. Join fan-out can multiply fact records and inflate measures.

## Security and privacy

Customer names, birthdates, gender, marital status, and country should be treated as sensitive personal data. Apply least-privilege access, use approved/masked data for demonstrations, do not commit real extracts, and follow organizational retention and privacy requirements. The repository does not define the applicable policy or retention duration; obtain those from the data owner.

## Operational improvements recommended

For repeated or production-like use, add configurable source locations, load audit tables, per-table counts, structured errors, explicit transaction/recovery behavior, automated quality gates, scheduling, monitoring/alerts, backup/restore guidance, and named operational ownership. These are recommendations, not current implemented capabilities.

## Related documents

- [Architecture](architecture.md)
- [Data integration](data_integration.md)
- [Data models](data_models.md)
- [Data catalog](data_catalog.md)
- [Quality checks](quality_checks.md)
