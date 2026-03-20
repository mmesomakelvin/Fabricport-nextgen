# End-to-End Data Engineering with Microsoft Fabric

This project demonstrates a complete data analytics platform built entirely within **Microsoft Fabric**, covering every stage of the modern data engineering lifecycle — from workspace provisioning and data ingestion to star schema modelling and interactive Power BI reporting.

## Video Walkthrough

[Watch the full build on YouTube](https://www.youtube.com/watch?v=tBkvwY1ShA0&t=4893s)

---

## Architecture Overview

The platform follows a six-stage data flow:

```
Data Source (CSV via GitHub API)
    → Ingest (Fabric Data Pipeline)
    → Store (Fabric Lakehouse - Delta Parquet on OneLake)
    → Prep & Transform (PySpark Notebook + Dataflow Gen2)
    → Serve (Fabric Data Warehouse - Star Schema)
    → Visualise (Power BI Reports & Dashboards)
```

### Key Components

| Stage | Fabric Component | Purpose |
|-------|-----------------|---------|
| Ingest | Data Pipeline (`NextGenAnalytics_Ingest`) | Extracts sales data from GitHub API into the lakehouse |
| Store | Lakehouse (`DataStagingLakehouse`) | Raw data landing zone using Delta Parquet format |
| Transform | PySpark Notebook + Dataflow Gen2 | Adds Year/Month columns; splits CustomerName into FirstName/LastName |
| Serve | Data Warehouse (`LT_Reporting_DWH`) | Star schema with fact and dimension tables |
| Visualise | Power BI (`NEXT GEN DATA BI REPORT`) | Interactive reports with product rankings, customer analysis, and trend charts |

---

## Star Schema Design

The data warehouse uses a classic star schema with the following tables under the `Sales` schema:

**Fact Table**
- `Sales.Fact_Sales` — Sales transactions with foreign keys to dimension tables, plus Quantity, UnitPrice, TaxAmount, Year, and Month

**Dimension Tables**
- `Sales.Dim_Customer` — CustomerID, CustomerName, FirstName, LastName, EmailAddress
- `Sales.Dim_Item` — ItemID, ItemName

---

## Repository Files

| File | Description |
|------|-------------|
| `sales.csv` | Source sales dataset (Adventure Works sample data) |
| `NextGen Notebook.ipynb` | PySpark notebook that reads raw CSV, adds Year and Month columns, and saves as a Delta table |
| `SchemaFact&DimTablesQueryFabricDWH.sql` | T-SQL script to create the Sales schema, Fact_Sales, Dim_Customer, and Dim_Item tables |
| `LoadDataFromStagingLakehouseinFact&DimTablesDWHFabric.sql` | Stored procedure that loads data from the staging lakehouse into the star schema tables by year |
| `stagingsalesview.sql` | Creates a cross-database view (`Sales.staging_salesdata`) that reads directly from the lakehouse without copying data |
| `Execsp.sql` | Executes the stored procedure to populate the warehouse (e.g., `EXEC Sales.LoadDataFromStagingLakehouse 2021`) |
| `NextGenDataAnalytics_Portfolio (9).docx` | Full portfolio documentation covering every step of the project |

---

## Data Pipeline Flow

The automated pipeline (`NextGenAnalytics_Ingest`) runs three activities in sequence:

1. **Delete** — Removes stale CSV files from the lakehouse staging folder to prevent duplicates
2. **Copy Data** — Pulls fresh sales data from the GitHub API into `Files/SourceFilesSales/`
3. **Notebook** — Executes the PySpark notebook to transform and save the data as a Delta table

The pipeline supports parameterisation — the notebook's `table_name` parameter can be overridden by the pipeline to create different output tables without code changes.

---

## Tools & Services

- **Microsoft Fabric** (Trial Workspace)
  - OneLake
  - Lakehouse
  - Data Warehouse
  - Data Pipeline
  - PySpark Notebooks
  - Dataflow Gen2
  - Power BI (Direct Lake on SQL storage mode)
- **Apache Spark** (via Fabric compute)
- **T-SQL** (warehouse schema and stored procedures)
- **Power Query** (Dataflow Gen2 transformations)

---

## Key Learnings

- **Destination-first design**: Define the warehouse schema before building ingestion pipelines to establish clear data contracts
- **Parameterisation**: Both the notebook (`table_name` parameter) and stored procedure (`@OrderYear` parameter) are designed for reuse
- **Schema mismatch debugging**: Splitting a column in Dataflow Gen2 without preserving the original causes null values — always duplicate before splitting
- **Cross-database querying**: Fabric allows the warehouse to query lakehouse tables directly via views, eliminating unnecessary data duplication
