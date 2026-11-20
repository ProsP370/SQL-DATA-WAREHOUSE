# 🏗️ SQL Data Warehouse Project

A practical SQL Server data warehousing project that demonstrates how raw CRM and ERP data is loaded, cleaned, transformed, modeled, and validated for reporting and analysis.

## 🎯 Project Objective

The project converts source CSV files into an organized warehouse using a three-layer approach:

- **🥉 Bronze** – Preserves raw source records.
- **🥈 Silver** – Cleans, standardizes, and validates data.
- **🥇 Gold** – Provides business-ready dimensional and fact views.

It demonstrates ETL workflows, dimensional modeling, SQL development, data quality testing, and warehouse operations.

## 🥉🥈🥇 Medallion Architecture

### 🥉 Bronze — Raw Data

- Loads source CRM and ERP CSV files.
- Preserves source identifiers and values.
- Acts as the landing area for raw data.
- Uses T-SQL bulk-loading procedures.

### 🥈 Silver — Cleansed Data

- Standardizes formats and column names.
- Applies transformations and validation.
- Identifies or removes invalid records.
- Creates conformed tables for further processing.

### 🥇 Gold — Business Data

- Builds customer and product dimensions.
- Builds a fact table for sales analysis.
- Provides reporting-ready views.
- Supports downstream dashboards and analysis.

## 🔄 Data Architecture

<img width="1550" height="1052" alt="Data architecture draw corrected 2" src="https://github.com/user-attachments/assets/35178b09-811c-43ff-a951-8dcc7e35fc53" />



## 🛠️ Technologies

- Microsoft SQL Server
- T-SQL
- SQL Server Management Studio
- CSV source files
- Stored procedures and views
- SQL quality checks

## 📂 Repository Structure

- `script/` — Database, table, loader, transform, and view scripts
- `datasets/` — Source data resources
- `tests/` — Data-quality and validation queries
- `docs/` — Project and architecture documentation
- `README.md` — Project overview

## ✅ Data Quality

The project checks for:

- Missing or duplicate identifiers
- Invalid dates
- Invalid sales values
- Unexpected or null values
- Broken joins
- Duplicate or inflated fact records

## 🚀 Key Learnings

This project demonstrates:

- Medallion architecture
- ETL and ELT-style processing
- Data warehouse design
- Dimensional modeling
- SQL stored procedures
- Data transformation and cleansing
- Quality assurance
- Reporting-layer development

## ⚠️ Setup Notes

- Review scripts before execution.
- Source CSV files must be accessible to SQL Server.
- Bronze and Silver loads may overwrite existing data.
- Protect customer information and credentials.
- Use an approved development database for testing.

## 📚 Documentation

- [Architecture](docs/architecture.md)
- [Data Integration](docs/data_integration.md)
- [Data Models](docs/data_models.md)
- [Naming Conventions](docs/naming_conventions.md)
- [Setup and Operations](docs/setup_and_operations.md)
- [Quality Checks](docs/quality_checks.md)

---

## 👨‍💻 About Me

I am **Prosper Nwobu**, a **Data Engineer, Python Programmer, and SQL Enthusiast** focused on building practical data pipelines, data warehouse solutions, and analytical projects.

🔗 **GitHub:** [github.com/ProsP370](https://github.com/ProsP370)
