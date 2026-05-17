# Azure Data Factory Data Migration On Premise SQL Server to Cloud ADLS GEN 2

Modern organizations generate large volumes of operational data in on-premise systems. To enable scalable analytics and reporting, this data must be efficiently migrated to the cloud.

This project demonstrates an end-to-end data engineering pipeline that extracts data from an On-Premise SQL Server, processes it using Azure Data Factory, and stores it in Azure Data Lake Storage Gen2. The pipeline implements a metadata-driven incremental loading strategy using a watermark table, ensuring that only new or updated records are processed.

# 📌 Table of Contents
- [Business Problem](#business-problem)
- [Tech Stack](#tech-stack)
- [Data Warehouse Model](#data-warehouse-model)
- [DataWarehouse Configurations](#datawarehouse-configurations)
- [ADF Pipeline Logic](#adf-pipeline-logic)
- [Master Pipeline execution ](#master-pipeline-execution)
- [Child Pipeline Execution](#child-pipeline-execution)
- [Project Structure](#project-structure)
- [Business Impact](#business-impact)
- [Author & Contact](#author-&-contact)

# Business Problem

Organizations relying on on-premises SQL Server environments often face challenges in building scalable and efficient data ingestion frameworks for analytics and reporting. Traditional ETL processes commonly result in:

Data silos across operational systems
Manual and inefficient data movement processes
High latency in reporting and analytics
Difficulty handling incremental data loads
Limited scalability for growing enterprise datasets
Lack of centralized and governed data storage

The objective of this project is to design and implement a scalable Azure-based data ingestion solution capable of:

- ✅ Securely connecting to on-premises SQL Server using Self-Hosted Integration Runtime (SHIR)
- ✅ Performing dynamic full and incremental data loads using watermark-based processing
- ✅ Automating parent-child pipeline orchestration in Azure Data Factory (ADF)
- ✅ Storing enterprise data efficiently in Azure Data Lake Storage Gen2 (ADLS Gen2) as Parquet files
- ✅ Enabling scalable downstream analytics and reporting workloads
- ✅ Reducing data movement overhead through metadata-driven pipeline execution

# Tech Stack 

| Technology | Purpose |
|---|---|
| Microsoft Azure | Cloud platform for scalable data integration and engineering infrastructure |
| Azure Data Factory (ADF) | Pipeline orchestration, workflow automation, and metadata-driven ingestion |
| On-Premises SQL Server | Enterprise source system for operational data extraction |
| Self-Hosted Integration Runtime (SHIR) | Secure connectivity between Azure and on-premises environments |
| Azure Data Lake Storage Gen2 (ADLS Gen2) | Scalable cloud storage for sink layer and Parquet file storage |
| Parquet Format | Optimized columnar storage format for scalable analytics workloads |
| SQL | Source querying, watermark logic, and stored procedure execution |
| Stored Procedures | Metadata retrieval and watermark update processing |
| Watermark Table Logic | Incremental load tracking and change data processing |
| Dynamic Pipelines | Reusable parent-child pipeline orchestration framework |
| Lookup Activity | Dynamic retrieval of metadata and watermark values |
| Copy Activity | Data extraction and movement from source to sink |
| If Condition Activity | Conditional execution for full and incremental data loads |
| ForEach Activity | Iterative execution of child pipelines using metadata-driven orchestration |
| JSON | Configuration format for ADF pipelines, datasets, linked services, and runtime definitions |
| GitHub | Version control and project documentation |


# Solution Architecture 

Architecture Overview with Pipeline Flowchart

![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/091828f3e0112aa9781fb4e4a0e5d06b91d6b919/ADF%20SQL%20Migration.png)
![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/5d3b1590ea59b875cff2fec5773ea4086d59fbed/Pipeline%20Flow%20chart.png)

This project demonstrates an end-to-end Azure Data Factory and SQL-based incremental data migration pipeline from an on-premises SQL Server environment to a cloud-based data lake architecture. The solution uses a watermark-driven incremental loading strategy, where ADF dynamically compares the source table watermark column against the stored LastLoadValue maintained in a control table.

During the initial execution, if the LastLoadValue is 0, the pipeline performs a full load and generates Parquet files in the latest ingestion folder. Upon successful completion, a stored procedure updates the watermark control table with the latest processed value.

For subsequent executions, the pipeline performs incremental extraction by identifying records where the source watermark column value is greater than the previously processed LastLoadValue. To prevent accidental duplicate ingestion, additional validation logic ensures that if the maximum watermark value equals the existing LastLoadValue, the pipeline skips the append operation.

This orchestration pattern improves scalability, minimizes redundant processing, and ensures efficient incremental fact and dimension data movement using Azure Data Factory, SQL Server stored procedures, and Parquet-based storage.

# Data Warehouse Model

### Star Schema used for analytics
Dimension Tables:
- DimCustomer
- DimProduct
- DimDate
- DimRegion
- DimStore etc.
  
Fact Tables:
- FactSales
- FactOrders
- FactPayments
- FactReturns etc.

# DataWarehouse Configurations

![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/0289f6070d8240eb93de80f27719ff5381d7fd3e/DataWarehouse%20Configuration.png)

# ADF Pipeline Logic

- Lookup activity retrieves table metadata
- ForEach loop processes tables dynamically
- Condition checks if new records exist
- Copy activity loads data to ADLS

# Master Pipeline execution 

![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/aa68766f66d3437aaff24afa122e573429bbfcff/masterpipelineexecution.png)

The Lookup activity retrieves metadata from the watermark table through a stored procedure hosted on the on-premises SQL Server. It extracts details such as LastLoadValue, TableName, and SchemaName. The pipeline then iterates through the retrieved records using a ForEach activity and dynamically passes the metadata to the Child Pipeline for incremental data ingestion and processing.

# Child Pipeline Execution

![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/8ad69fa0a26814d521491171b17bfb36b134fd90/Childpipeline2.png)

The pipeline begins with a Lookup activity that retrieves the maximum value of the watermark column from the source table. This value is then passed to an If Condition activity to determine whether a full load or incremental load should be executed.

The If Condition logic evaluates the following scenarios:

If the LastLoadValue in the watermark table is NULL, the pipeline performs a full load.
If the current maximum watermark value from the source table is greater than the LastLoadValue stored in the watermark table, the pipeline performs an incremental load.

Under the True condition, the Copy activity dynamically generates the source query based on the load type:

For a full load, all records are extracted from the source table.
For an incremental load, only records where the watermark column value is greater than the stored LastLoadValue are extracted.

The extracted data is then written to the sink layer in dynamically generated date-based folders using the format DD/MM/YYYY.

This entire pipeline leverages a Self-Hosted Integration Runtime (SHIR) to securely establish connectivity between Azure Data Factory and the on-premises SQL Server environment for data ingestion and orchestration.

# Project Structure
```plaintext
adf-onprem-sqlserver-incremental-ingestion/
│
├── README.md
│
├── architecture/
│   └── solution_architecture.png
│
├── datasets/
│   ├── DS_SQL_PARQUET1.json
│   ├── DS_SQL_PARQUET_FILES.json
│   ├── DS_SQL_PARQUET_FILES_2.json
│   ├── LKP_MAX_VALUE.json
│   └── OnPremSQLLKP.json
│
├── pipelines/
│   ├── parent_pipeline/
│   │   ├── Parent Pipeline Orchestration.json
│   │   └── Test Parent Pipeline.json
│   │
│   ├── child_pipeline_incremental/
│   │   ├── Child Pipeline Incremental.json
│   │   └── Test Child Pipeline Incremental.json
│   │
│   └── child_pipeline_stored_procedure/
│       ├── Child Pipeline Stored Procedure.json
│       └── Test Child Pipeline Stored Procedure.json
│
├── linked_services/
│   ├── LS_ADLSGEN2.json
│   └── SqlServer1.json
│
├── integration_runtime/
│   └── IRONPREM.json
│
├── sql/
│   ├── stored_procedures/
│   │   └── UpdateWatermarkTable.sql
│   │
│   └── fact_scripts/
│       └── dimfactscript.sql
│
├── factory/
│   └── adfmigration11.json
│
├── sink_storage/
│   └── parquet_files_dd_mm_yyyy/
│
├── workflows/
│   └── incremental_load_workflow.md
│
├── docs/
│   ├── implementation_notes/
│   ├── pipeline_execution_flow/
│   └── watermark_incremental_logic/
│
└── publish_config/
    └── publish_config.json
```

# Business Impact

This solution modernizes enterprise analytics by leveraging Microsoft Azure Data Factory (ADF) and on-premises SQL Server integration to securely ingest and process data from source systems into a scalable Lakehouse architecture. The platform transforms raw operational data into business-ready insights, enabling stakeholders to monitor sales performance, optimize logistics and delivery operations, improve customer experience, and drive data-driven decision-making through centralized and governed analytics reporting.

# Author & Contact

Ajay Suresh
Azure Data Engineer | Power BI Developer

💼 Focus Areas: Azure Data Engineering, ETL Pipelines, PySpark, Data Warehousing, Power BI
- 🔗 LinkedIn Portfolio
- 🔗 GitHub Projects
  


