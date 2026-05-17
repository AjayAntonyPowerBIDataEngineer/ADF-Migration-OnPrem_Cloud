# Azure Data Factory Data Migration On Premise SQL Server to Cloud ADLS GEN 2

Modern organizations generate large volumes of operational data in on-premise systems. To enable scalable analytics and reporting, this data must be efficiently migrated to the cloud.

This project demonstrates an end-to-end data engineering pipeline that extracts data from an On-Premise SQL Server, processes it using Azure Data Factory, and stores it in Azure Data Lake Storage Gen2. The pipeline implements a metadata-driven incremental loading strategy using a watermark table, ensuring that only new or updated records are processed.


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

![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/15caa70080790b7bc8031f457fa5d2413a54c737/Child%20Pipeline%20execution.png)


# Key Takeaway

This solution implements a scalable and efficient incremental data loading framework using watermark logic. It ensures optimized data movement, reduces processing time, and maintains data consistency through dynamic pipeline orchestration and automated watermark updates.

- Built an end-to-end data engineering pipeline
- Implemented metadata-driven ingestion
- Used incremental loading for efficiency
- Designed scalable cloud architecture











