# ADF-Migration-OnPrem_Cloud
ADF Migration On Prem SQL Server to Cloud



# Solution Architecture 

![image_alt](https://github.com/AjayAntonyPowerBIDataEngineer/ADF-Migration-OnPrem_Cloud/blob/091828f3e0112aa9781fb4e4a0e5d06b91d6b919/ADF%20SQL%20Migration.png)

This project demonstrates an end-to-end Azure Data Factory and SQL-based incremental data migration pipeline from an on-premises SQL Server environment to a cloud-based data lake architecture. The solution uses a watermark-driven incremental loading strategy, where ADF dynamically compares the source table watermark column against the stored LastLoadValue maintained in a control table.

During the initial execution, if the LastLoadValue is 0, the pipeline performs a full load and generates Parquet files in the latest ingestion folder. Upon successful completion, a stored procedure updates the watermark control table with the latest processed value.

For subsequent executions, the pipeline performs incremental extraction by identifying records where the source watermark column value is greater than the previously processed LastLoadValue. To prevent accidental duplicate ingestion, additional validation logic ensures that if the maximum watermark value equals the existing LastLoadValue, the pipeline skips the append operation.

This orchestration pattern improves scalability, minimizes redundant processing, and ensures efficient incremental fact and dimension data movement using Azure Data Factory, SQL Server stored procedures, and Parquet-based storage.

