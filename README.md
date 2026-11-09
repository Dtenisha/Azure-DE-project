# Azure-DE-project
# Fuel Retail Data Engineering Pipeline | Azure

## Project Overview

An end-to-end data engineering project focused on building automated data ingestion pipelines for the **Fuel Retail domain** using Microsoft Azure.

The project demonstrates how to extract data from Azure SQL Database, implement metadata-driven ingestion, manage incremental data using CDC watermark logic, and store raw data in Azure Data Lake Storage Gen2.

The architecture follows a Medallion approach, with Azure Data Factory handling ingestion and Databricks planned for Bronze, Silver, and Gold transformations.

## Technologies Used

- **Azure SQL Database** — Source database containing fuel retail data
- **Azure Data Factory (ADF)** — Pipeline orchestration and data ingestion
- **Azure Data Lake Storage Gen2** — Data lake storage
- **SQL (T-SQL)** — Table creation, data generation, and querying
- **Parquet with Snappy compression** — Efficient columnar data storage
- **JSON** — Pipeline configuration and CDC watermark storage
- **Azure Managed Identity** — Authentication for Azure services
- **Databricks / PySpark** — Planned transformation and analytics layer

## Architecture

```text
Azure SQL Database
        |
        v
Azure Data Factory
        |
        |-- Lookup Activity
        |-- ForEach Activity
        |-- Copy Data Activity
        |-- If Condition Activity
        |-- Set Variable Activity
        |-- Web Activity
        |
        v
Azure Data Lake Storage Gen2
        |
        v
Bronze Layer (Raw Data)
        |
        v
Silver Layer (Planned)
        |
        v
Gold Layer (Planned)
```

## Database Design

The source database contains seven tables representing fuel retail operations.

| Table | Description |
|---|---|
| DimCustomer | Customer information |
| DimStation | Fuel station details |
| DimFuelProduct | Fuel product information |
| DimPump | Fuel pump details |
| DimDate | Calendar dimension |
| FactFuelSale | Fuel sales transactions |
| FactInventoryReading | Fuel inventory measurements |

The database uses primary and foreign keys to maintain relationships between dimensions and facts.

## Data Ingestion

### Initial Data Load

Azure Data Factory extracts source records from Azure SQL Database and writes them into ADLS Gen2 using Parquet format.

Key features:

- Parameterized SQL datasets
- Dynamic table selection
- Lookup-driven table discovery
- ForEach-based table processing
- Dynamic folder paths
- Snappy compression

### Incremental Data Load

The incremental pipeline uses an `updated_at` timestamp column to identify new and modified records.

A CDC watermark is maintained in JSON format:

```json
{
  "cdc": "1900-01-01"
}
```

The pipeline compares the source timestamp with the stored watermark and retrieves eligible records.

Example SQL:

```sql
SELECT *
FROM dbo.DimCustomer
WHERE updated_at > '2025-09-01';
```

A historical-load start-date parameter is also available to support backfill scenarios.

**Note:** This project uses timestamp-based incremental extraction rather than SQL Server's native Change Data Capture feature.

## Metadata-Driven Pipeline

Instead of configuring separate pipelines for each table, the project uses a JSON array containing table-level configurations.

Example:

```json
[
  {
    "schema": "dbo",
    "table": "DimCustomer",
    "cdc_column": "updated_at",
    "history_load_start_date": ""
  },
  {
    "schema": "dbo",
    "table": "FactFuelSale",
    "cdc_column": "updated_at",
    "history_load_start_date": ""
  }
]
```

A ForEach activity iterates through the configurations and supplies table-specific parameters to the ingestion pipeline.

The `DimDate` table does not contain an `updated_at` column and therefore requires full-load processing.

## Azure Data Lake Structure

The target storage structure is:

```text
sourcebronze/
├── DimCustomer/
├── DimStation/
├── DimFuelProduct/
├── DimPump/
├── DimDate/
├── FactFuelSale/
└── FactInventoryReading/
```

Separate CDC watermark files may be maintained for tables participating in incremental loading.

## Implemented and Planned Work

### Ingestion and Orchestration

- [x] Created fuel-domain dimension and fact table definitions
- [x] Generated synthetic initial-load and incremental-load SQL scripts
- [x] Configured Azure SQL Database and Azure Data Factory connectivity
- [x] Implemented pipeline and dataset parameters
- [x] Configured Lookup, ForEach, and Copy Data activities
- [x] Configured Parquet datasets with Snappy compression
- [x] Created a folder-creation workflow using Web Activity
- [ ] Complete validation of automated ingestion for all seven tables
- [ ] Complete CDC watermark initialization and update handling
- [ ] Verify historical backfill and rerun behavior

### Transformations and Analytics

- [ ] Import Bronze data into Databricks
- [ ] Implement Silver-layer cleaning and deduplication
- [ ] Build Gold-layer analytics tables
- [ ] Validate joins and aggregations
- [ ] Implement Delta Lake incremental processing

## Learning Objectives

This project focuses on practical experience with:

- Building parameterized and reusable ADF pipelines
- Implementing metadata-driven orchestration
- Understanding full vs incremental data loads
- Designing CDC watermark workflows
- Working with Azure SQL and ADLS Gen2
- Handling conditional execution and pipeline failures
- Preparing data for PySpark and Delta Lake transformations

## Future Improvements

- Implement incremental Delta Lake MERGE operations
- Add data quality validation
- Add pipeline monitoring and error notifications
- Implement retry and recovery strategies
- Build reporting-ready Gold tables
- Introduce CI/CD for pipeline deployment

## Project Status

**In progress** — Azure ingestion and orchestration development, with Databricks transformations planned.
