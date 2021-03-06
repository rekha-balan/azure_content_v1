<properties 
    pageTitle="Move data to and from Azure SQL Data Warehouse | Azure Data Factory" 
    description="Learn how to move data to/from Azure SQL Data Warehouse using Azure Data Factory" 
    services="data-factory" 
    documentationCenter="" 
    authors="spelluru" 
    manager="jhubbard" 
    editor="monicar"/>

<tags 
    ms.service="data-factory" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="10/29/2015" 
    ms.author="spelluru"/>

# Move data to and from Azure SQL Data Warehouse using Azure Data Factory
This article outlines how you can use data factory copy activity to move data to Azure SQL Data Warehouse from another data store and move data from another data store to Azure SQL. This article builds on the [data movement activities](data-factory-data-movement-activities.md) article which presents a general overview of data movement with copy activity and supported data store combinations.

The following sample(s) show how to copy data to and from Azure SQL Data Warehouse and Azure Blob Storage. However, data can be copied **directly** from any of sources to any of the sinks stated [here](data-factory-data-movement-activities.md#supported-data-stores) using the Copy Activity in Azure Data Factory.  

## Sample: Copy data from Azure SQL Data Warehouse to Azure Blob
The sample below shows:

1. A linked service of type [AzureSqlDW](#azure-sql-data-warehouse-linked-service-properties.md).
2. A linked service of type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties). 
3. An input [dataset](data-factory-create-datasets.md) of type [AzureSqlDWTable](#azure-sql-data-warehouse-dataset-type-properties.md). 
4. An output [dataset](data-factory-create-datasets.md) of type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
5. A [pipeline](data-factory-create-pipelines.md) with Copy Activity that uses [SqlDWSource](#azure-sql-data-warehouse-copy-activity-type-properties.md) and [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).

The sample copies data belonging to a time series from a table in Azure SQL Data Warehouse database to a blob every hour. The JSON properties used in these samples are described in sections following the samples.

**Azure SQL Data Warehouse linked service:**

    {
      "name": "AzureSqlDWLinkedService",
      "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
          "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
      }
    }

**Azure Blob storage linked service:**

    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

**Azure SQL Data Warehouse input dataset:**

The sample assumes you have created a table “MyTable” in Azure SQL Data Warehouse and it contains a column called “timestampcolumn” for time series data.

Setting “external”: ”true” and specifying externalData policy informs the Data Factory service that the table is external to the data factory and not produced by an activity in the data factory.

    {
      "name": "AzureSqlDWInput",
      "properties": {
        "type": "AzureSqlDWTable",
        "linkedServiceName": "AzureSqlDWLinkedService",
        "typeProperties": {
          "tableName": "MyTable"
        },
        "external": true,
        "availability": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "externalData": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
          }
        }
      }
    }

**Azure Blob output dataset:**

Data is written to a new blob every hour (frequency: hour, interval: 1). The folder path for the blob is dynamically evaluated based on the start time of the slice that is being processed. The folder path uses year, month, day, and hours parts of the start time.

    {
      "name": "AzureBlobOutput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
          "partitionedBy": [
            {
              "name": "Year",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "yyyy"
              }
            },
            {
              "name": "Month",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "%M"
              }
            },
            {
              "name": "Day",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "%d"
              }
            },
            {
              "name": "Hour",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "%H"
              }
            }
          ],
          "format": {
            "type": "TextFormat",
            "columnDelimiter": "\t",
            "rowDelimiter": "\n"
          }
        },
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }


**Pipeline with Copy activity:**

The pipeline contains a Copy Activity that is configured to use the above input and output datasets and is scheduled to run every hour. In the pipeline JSON definition, the **source** type is set to **SqlDWSource** and **sink** type is set to **BlobSink**. The SQL query specified for the **SqlReaderQuery** property selects the data in the past hour to copy.

    {  
        "name":"SamplePipeline",
        "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
          {
            "name": "AzureSQLDWtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [
              {
                "name": "AzureSqlDWInput"
              }
            ],
            "outputs": [
              {
                "name": "AzureBlobOutput"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "SqlDWSource",
                "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
              },
              "sink": {
                "type": "BlobSink"
              }
            },
           "scheduler": {
              "frequency": "Hour",
              "interval": 1
            },
            "policy": {
              "concurrency": 1,
              "executionPriorityOrder": "OldestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
         ]
       }
    }

> [!NOTE]
> In the above example, **sqlReaderQuery** is specified for the SqlDWSource. The Copy Activity runs this query against the Azure SQL Data Warehouse source to get the data.
> 
> Alternatively, you can specify a stored procedure by specifying the **sqlReaderStoredProcedureName** and **storedProcedureParameters** (if the stored procedure takes parameters).
> 
> If you do not specify either sqlReaderQuery or sqlReaderStoredProcedureName, the columns defined in the structure section of the dataset JSON are used to build a query (select column1, column2 from mytable) to run against the Azure SQL Data Warehouse. If the dataset definition does not have the structure, all columns are selected from the table.
> 
> 
## Sample: Copy data from Azure Blob to Azure SQL Data Warehouse
The sample below shows:

1. A linked service of type [AzureSqlDW](#azure-sql-data-warehouse-linked-service-properties.md).
2. A linked service of type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties).
3. An [dataset](data-factory-create-datasets.md) dataset of type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
4. An [dataset](data-factory-create-datasets.md) dataset of type [AzureSqlDWTable](#azure-sql-data-warehouse-dataset-type-properties.md).
5. A [pipeline](data-factory-create-pipelines.md) with Copy activity that uses [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) and [SqlDWSink](#azure-sql-data-warehouse-copy-activity-type-properties.md).

The sample copies data belonging to a time series from Azure blob to a table in Azure SQL Data Warehouse database every hour. The JSON properties used in these samples are described in sections following the samples. 

**Azure SQL Data Warehouse linked service:**

    {
      "name": "AzureSqlDWLinkedService",
      "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
          "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
      }
    }

**Azure Blob storage linked service:**

    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

**Azure Blob input dataset:**

Data is picked up from a new blob every hour (frequency: hour, interval: 1). The folder path and file name for the blob are dynamically evaluated based on the start time of the slice that is being processed. The folder path uses year, month, and day part of the start time and file name uses the hour part of the start time. “external”: “true” setting informs the Data Factory service that this table is external to the data factory and not produced by an activity in the data factory.

    {
      "name": "AzureBlobInput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
          "fileName": "{Hour}.csv",
          "partitionedBy": [
            {
              "name": "Year",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "yyyy"
              }
            },
            {
              "name": "Month",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "%M"
              }
            },
            {
              "name": "Day",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "%d"
              }
            },
            {
              "name": "Hour",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "%H"
              }
            }
          ],
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ",",
            "rowDelimiter": "\n"
          }
        },
        "external": true,
        "availability": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "externalData": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
          }
        }
      }
    }

**Azure SQL Data Warehouse output dataset:**

The sample copies data to a table named “MyTable” in Azure SQL Data Warehouse. You should create the table in Azure SQL Data Warehouse with the same number of columns as you expect the Blob CSV file to contain. New rows are added to the table every hour. 

    {
      "name": "AzureSqlDWOutput",
      "properties": {
        "type": "AzureSqlDWTable",
        "linkedServiceName": "AzureSqlDWLinkedService",
        "typeProperties": {
          "tableName": "MyOutputTable"
        },
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }

**Pipeline with Copy Activity**

The pipeline contains a Copy Activity that is configured to use the above input and output datasets and is scheduled to run every hour. In the pipeline JSON definition, the **source** type is set to **BlobSource** and **sink** type is set to **SqlDWSink**.

    {  
        "name":"SamplePipeline",
        "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[  
          {
            "name": "AzureBlobtoSQLDW",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [
              {
                "name": "AzureBlobInput"
              }
            ],
            "outputs": [
              {
                "name": "AzureSqlDWOutput"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource",
                "blobColumnSeparators": ","
              },
              "sink": {
                "type": "SqlDWSink"
              }
            },
           "scheduler": {
              "frequency": "Hour",
              "interval": 1
            },
            "policy": {
              "concurrency": 1,
              "executionPriorityOrder": "OldestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
          ]
       }
    }

See the [Load data with Azure Data Factory](../sql-data-warehouse/sql-data-warehouse-get-started-load-with-azure-data-factory.md) article in the Azure SQL Data Warehouse documentation for a walkthrough. 

## Azure SQL Data Warehouse Linked Service properties
The following table provides description for JSON elements specific to Azure SQL Data Warehouse linked service. 

| Property | Description | Required |
| --- | --- | --- |
| type |The type property must be set to: **AzureSqlDW** |Yes |
| **connectionString** |Specify information needed to connect to the Azure SQL Data Warehouse instance for the connectionString property. |Yes |

Note: You need to configure [Azure SQL Database Firewall](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure). You need to configure the database server to [allow Azure Services to access the server](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure). Additionally, if you are copying data to Azure SQL Data Warehouse from outside Azure including from on-premises data sources with data factory gateway you need to configure appropriate IP address range for the machine that is sending data to Azure SQL Data Warehouse. 

## Azure SQL Data Warehouse Dataset type properties
For a full list of sections & properties available for defining datasets, please refer to the [Creating datasets](data-factory-create-datasets.md) article. Sections like structure, availability, and policy of a dataset JSON are similar for all dataset types (Azure SQL, Azure blob, Azure table, etc...). 

The typeProperties section is different for each type of dataset and provides information about the location of the data in the data store. The **typeProperties** section for the dataset of type **AzureSqlDWTable** has the following properties.

| Property | Description | Required |
| --- | --- | --- |
| tableName |Name of the table in the Azure SQL Data Warehouse database that the linked service refers to. |Yes |

## Azure SQL Data Warehouse Copy Activity type properties
For a full list of sections & properties available for defining activities, please refer to the [Creating Pipelines](data-factory-create-pipelines.md) article. Properties like name, description, input and output tables, various policies etc are available for all types of activities.

**Note:** The Copy Activity takes only one input and produces only one output.

Properties available in the typeProperties section of the activity on the other hand vary with each activity type and in case of Copy activity they vary depending on the types of sources and sinks.

### SqlDWSource
In case of Copy activity when source is of type **SqlDWSource** the following properties are available in **typeProperties** section:

| Property | Description | Allowed values | Required |
| --- | --- | --- | --- |
| sqlReaderQuery |Use the custom query to read data. |SQL query string. For example: select * from MyTable. |No |
| sqlReaderStoredProcedureName |Name of the stored procedure that reads data from the source table. |Name of the stored procedure. |No |
| storedProcedureParameters |Parameters for the stored procedure. |Name/value pairs. Names and casing of parameters must match the names and casing of the stored procedure parameters. |No |

If the **sqlReaderQuery** is specified for the SqlDWSource, the Copy Activity runs this query against the Azure SQL Data Warehouse source to get the data. 

Alternatively, you can specify a stored procedure by specifying the **sqlReaderStoredProcedureName** and **storedProcedureParameters** (if the stored procedure takes parameters). 

If you do not specify either sqlReaderQuery or sqlReaderStoredProcedureName, the columns defined in the structure section of the dataset JSON are used to build a query (select column1, column2 from mytable) to run against the Azure SQL Data Warehouse. If the dataset definition does not have the structure, all columns are selected from the table.

#### SqlDWSource example
    "source": {
        "type": "SqlDWSource",
        "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
        "storedProcedureParameters": {
            "stringData": { "value": "str3" },
            "id": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
        }
    }

**The stored procedure definition:** 

    CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
    (
        @stringData varchar(20),
        @id int
    )
    AS
    SET NOCOUNT ON;
    BEGIN
         select *
         from dbo.UnitTestSrcTable
         where dbo.UnitTestSrcTable.stringData != stringData
        and dbo.UnitTestSrcTable.id != id
    END
    GO


### SqlDWSink
**SqlDWSink** supports the following properties:

| Property | Description | Allowed values | Required |
| --- | --- | --- | --- |
| writeBatchSize |Inserts data into the SQL table when the buffer size reaches writeBatchSize |Integer. (unit = Row Count) |No (Default = 10000) |
| writeBatchTimeout |Wait time for the batch insert operation to complete before it times out. |(Unit = timespan) Example: “00:30:00” (30 minutes). |No |
| sqlWriterCleanupScript |User specified query for Copy Activity to execute such that data of a specific slice will be cleaned up. See repeatability section below for more details. |A query statement. |No |
| sliceIdentifierColumnName |User specified column name for Copy Activity to fill with auto generated slice identifier, which will be used to clean up data of a specific slice when rerun. See repeatability section below for more details. |Column name of a column with data type of binary(32). |No |

#### SqlDWSink example
    "sink": {
        "type": "SqlDWSink",
        "writeBatchSize": 1000000,
        "writeBatchTimeout": "00:05:00",
    }


## Repeatability during Copy

When copying data to Azure SQL/SQL Server from other data stores one needs to keep repeatability in mind to avoid unintended outcomes. 

When copying data to Azure SQL/SQL Server Database, copy activity will by default APPEND the data set to the sink table by default. For example, when copying data from a CSV (comma separated values data) file source containing two records to Azure SQL/SQL Server Database, this is what the table looks like:
	
	ID	Product		Quantity	ModifiedDate
	...	...			...			...
	6	Flat Washer	3			2015-05-01 00:00:00
	7 	Down Tube	2			2015-05-01 00:00:00

Suppose you found errors in source file and updated the quantity of Down Tube from 2 to 4 in the source file. If you re-run the data slice for that period, you’ll find two new records appended to Azure SQL/SQL Server Database. The below assumes none of the columns in the table have the primary key constraint.
	
	ID	Product		Quantity	ModifiedDate
	...	...			...			...
	6	Flat Washer	3			2015-05-01 00:00:00
	7 	Down Tube	2			2015-05-01 00:00:00
	6	Flat Washer	3			2015-05-01 00:00:00
	7 	Down Tube	4			2015-05-01 00:00:00

To avoid this, you will need to specify UPSERT semantics by leveraging one of the below 2 mechanisms stated below.

> [AZURE.NOTE] A slice can be re-run automatically in Azure Data Factory as per the retry policy specified.

### Mechanism 1

You can leverage **sqlWriterCleanupScript** property to first perform cleanup action when a slice is run. 

	"sink":  
	{ 
	  "type": "SqlSink", 
	  "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM table WHERE ModifiedDate >= \\'{0:yyyy-MM-dd HH:mm}\\' AND ModifiedDate < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
	}

The cleanup script would be executed first during copy for a given slice which would delete the data from the SQL Table corresponding to that slice. The activity will subsequently insert the data into the SQL Table. 

If the slice is now re-run, then you will find the quantity is updated as desired.
	
	ID	Product		Quantity	ModifiedDate
	...	...			...			...
	6	Flat Washer	3			2015-05-01 00:00:00
	7 	Down Tube	4			2015-05-01 00:00:00

Suppose the Flat Washer record is removed from the original csv. Then re-running the slice would produce the following result: 
	
	ID	Product		Quantity	ModifiedDate
	...	...			...			...
	7 	Down Tube	4			2015-05-01 00:00:00

Nothing new had to be done. The copy activity ran the cleanup script to delete the corresponding data for that slice. Then it read the input from the csv (which then contained only 1 record) and inserted it into the Table. 

### Mechanism 2

Another mechanism to achieve repeatability is by having a dedicated column (**sliceIdentifierColumnName**) in the target Table. This column would be used by Azure Data Factory to ensure the source and destination stay synchronized. This approach works when there is flexibility in changing or defining the destination SQL Table schema. 

This column would be used by Azure Data Factory for repeatability purposes and in the process Azure Data Factory will not make any schema changes to the Table. Way to use this approach:

1.	Define a column of type binary (32) in the destination SQL Table. There should be no constraints on this column. Let's name this column as ‘ColumnForADFuseOnly’ for this example.
2.	Use it in the copy activity as follows:

		"sink":  
		{ 
		  "type": "SqlSink", 
		  "sliceIdentifierColumnName": "ColumnForADFuseOnly"
		}

Azure Data Factory will populate this column as per its need to ensure the source and destination stay synchronized. The values of this column should not be used outside of this context by the user. 

Similar to mechanism 1, Copy Activity will automatically first clean up the data for the given slice from the destination SQL Table and then run the copy activity normally to insert the data from source to destination for that slice. 


## Specifying structure definition for rectangular datasets
The structure section in the datasets JSON is an **optional** section for rectangular tables (with rows & columns) and contains a collection of columns for the table. You will use the structure section for either providing type information for type conversions or doing column mappings. The following sections describe these features in detail. 

Each column contains the following properties:

| Property | Description | Required |
| -------- | ----------- | -------- |
| name | Name of the column. | Yes |
| type | Data type of the column. See type conversions section below for more details regarding when should you specify type information | No |
| culture | .NET based culture to be used when type is specified and is .NET type Datetime or Datetimeoffset. Default is “en-us”.  | No |
| format | Format string to be used when type is specified and is .NET type Datetime or Datetimeoffset. | No |

The following sample shows the structure section JSON for a table that has three columns userid, name, and lastlogindate.

    "structure": 
    [
        { "name": "userid"},
        { "name": "name"},
        { "name": "lastlogindate"}
    ],

Please use the following guidelines for when to include “structure” information and what to include in the **structure** section.

1.	**For structured data sources** that store data schema and type information along with the data itself (sources like SQL Server, Oracle, Azure table etc.), you should specify the “structure” section only if you want do column mapping of specific source columns to specific columns in sink and their names are not the same (see details in column mapping section below). 

	As mentioned above, the type information is optional in “structure” section. For structured sources, type information is already available as part of dataset definition in the data store, so you should not include type information when you do include the “structure” section.
2. **For schema on read data sources (specifically Azure blob)**  you can chose to store data without storing any schema or type information with the data. For these types of data sources you should include “structure” in the following 2 cases:
	1. You want to do column mapping.
	2. When the dataset is a source in a Copy activity, you can provide type information in “structure” and data factory will use this type information for conversion to native types for the sink. See [Move data to and from Azure Blob](../articles/data-factory/data-factory-azure-blob-connector.md) article for more information.

### Supported .NET-based types 
Data factory supports the following CLS compliant .NET based type values for providing type information in “structure” for schema on read data sources like Azure blob.

- Int16
- Int32 
- Int64
- Single
- Double
- Decimal
- Byte[]
- Bool
- String 
- Guid
- Datetime
- Datetimeoffset
- Timespan 

For Datetime & Datetimeoffset you can also optionally specify “culture” & “format” string to facilitate parsing of your custom Datetime string. See sample for type conversion below.



### Type Mapping for Azure SQL Data Warehouse
As mentioned in the [data movement activities](data-factory-data-movement-activities.md) article Copy activity performs automatic type conversions from automatic type conversions from source types to sink types with the following 2 step approach:

1. Convert from native source types to .NET type
2. Convert from .NET type to native sink type

When moving data to & from Azure SQL, SQL server, Sybase the following mappings will be used from SQL type to .NET type and vice versa.

The mapping is same as the [SQL Server Data Type Mapping for ADO.NET](https://msdn.microsoft.com/library/cc716729.aspx).

| SQL Server Database Engine type | .NET Framework type |
| --- | --- |
| bigint |Int64 |
| binary |Byte[]] |
| bit |Boolean |
| char |String, Char[]] |
| date |DateTime |
| Datetime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[]] |
| Float |Double |
| image |Byte[]] |
| int |Int32 |
| money |Decimal |
| nchar |String, Char[]] |
| ntext |String, Char[]] |
| numeric |Decimal |
| nvarchar |String, Char[]] |
| real |Single |
| rowversion |Byte[]] |
| smalldatetime |DateTime |
| smallint |Int16 |
| smallmoney |Decimal |
| sql_variant |Object * |
| text |String, Char[]] |
| time |TimeSpan |
| timestamp |Byte[]] |
| tinyint |Byte |
| uniqueidentifier |Guid |
| varbinary |Byte[]] |
| varchar |String, Char[]] |
| xml |Xml |

### Type conversion sample
The following sample is for copying data from a Blob to Azure SQL with type conversions.

Suppose the Blob dataset is in CSV format and contains 3 columns. One of them is a datetime column with a custom datetime format using abbreviated French names for day of the week.

You will define the Blob Source dataset as follows along with type definitions for the columns.

	{
	    "name": "AzureBlobTypeSystemInput",
	    "properties":
	    {
	         "structure": 
	          [
	                { "name": "userid", "type": "Int64"},
	                { "name": "name", "type": "String"},
	                { "name": "lastlogindate", "type": "Datetime", "culture": "fr-fr", "format": "ddd-MM-YYYY"}
	          ],
	        "type": "AzureBlob",
	        "linkedServiceName": "StorageLinkedService",
	        "typeProperties": {
	            "folderPath": "mycontainer/myfolder",
	            "fileName":"myfile.csv",
	            "format":
	            {
	                "type": "TextFormat",
	                "columnDelimiter": ","
	            }
	        },
	        "external": true,
	        "availability":
	        {
	            "frequency": "Hour",
	            "interval": 1
	        },
			"policy": {
	            "externalData": {
	                "retryInterval": "00:01:00",
	                "retryTimeout": "00:10:00",
	                "maximumRetry": 3
	            }
			}
	    }
	}

Given the SQL type to .NET type mapping table above you would define the Azure SQL table with the following schema.

| Column Name | SQL Type |
| ----------- | -------- |
| userid | bigint |
| name | text |
| lastlogindate | datetime |

Next you will define the Azure SQL dataset as follows. Note: You do not need to specify “structure” section with type information since the type information is already specified in the underlying data store.

	{
	    "name": "AzureSQLOutput",
	    "properties": {
	        "type": "AzureSqlTable",
	        "linkedServiceName": "AzureSqlLinkedService",
	        "typeProperties": {
	            "tableName": "MyTable"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        }
	    }
	}

In this case data factory will automatically do the type conversions including the Datetime field with the custom datetime format using the fr-fr culture when moving data from Blob to Azure SQL.




## Column mapping with translator rules
Column mapping can be used to specify how columns specified in the “structure” of source table map to columns specified in the “structure” of sink table. The **columnMapping** property is available in the **typeProperties** section of the Copy activity.

Column mapping supports the following scenarios:

1.	All columns in the source table “structure” are mapped to all columns in the sink table “structure”.
2.	A subset of the columns in the source table “structure” are mapped to all columns in the sink table “structure”.

The following are error conditions and will result in an exception:

1.	Either fewer columns or more columns in the “structure” of sink table than specified in the mapping.
2.	Duplicate mapping.
3.	SQL query result does not have a column name that is specified in the mapping.

## Column mapping samples
> [AZURE.NOTE] The samples below are for Azure SQL and Azure Blob but are applicable in the same way for any data store that supports rectangular tables. You will have to adjust dataset and linked service definitions in examples below to point to data in the relevant data source.

### Sample 1 – column mapping from Azure SQL to Azure blob
In this sample, the input table has a structure and it points to a SQL table in an Azure SQL database.

	{
	    "name": "AzureSQLInput",
	    "properties": {
	        "structure": 
	         [
	           { "name": "userid"},
	           { "name": "name"},
	           { "name": "group"}
	         ],
	        "type": "AzureSqlTable",
	        "linkedServiceName": "AzureSqlLinkedService",
	        "typeProperties": {
	            "tableName": "MyTable"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
			"external": true,
			"policy": {
	            "externalData": {
	                "retryInterval": "00:01:00",
	                "retryTimeout": "00:10:00",
	                "maximumRetry": 3
	            }
			}
	    }
	}

In this sample, the output table has a structure and it points to a blob in an Azure blob storage.

	{
	    "name": "AzureBlobOutput",
	    "properties":
	    {
	         "structure": 
	          [
	                { "name": "myuserid"},
	                { "name": "myname" },
	                { "name": "mygroup"}
	          ],
	        "type": "AzureBlob",
	        "linkedServiceName": "StorageLinkedService",
	        "typeProperties": {
	            "folderPath": "mycontainer/myfolder",
	            "fileName":"myfile.csv",
	            "format":
	            {
	                "type": "TextFormat",
	                "columnDelimiter": ","
	            }
	        },
	        "availability":
	        {
	            "frequency": "Hour",
	            "interval": 1
	        }
	    }
	}

The JSON for the activity is shown below. The columns from source mapped to columns in sink (**columnMappings**) by using **Translator** property.

	{
	    "name": "CopyActivity",
	    "description": "description", 
	    "type": "Copy",
	    "inputs":  [ { "name": "AzureSQLInput"  } ],
	    "outputs":  [ { "name": "AzureBlobOutput" } ],
	    "typeProperties":    {
	        "source":
	        {
	            "type": "SqlSource"
	        },
	        "sink":
	        {
	            "type": "BlobSink"
	        },
	        "translator": 
	        {
	            "type": "TabularTranslator",
	            "ColumnMappings": "UserId: MyUserId, Group: MyGroup, Name: MyName"
	        }
	    },
	   "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        }
	}

**Column mapping flow:**

![Column mapping flow](./media/data-factory-data-stores-with-rectangular-tables/column-mapping-flow.png)

### Sample 2 – column mapping with SQL query from Azure SQL to Azure blob
In this sample, a SQL query is used to extract data from Azure SQL instead of simply specifying the table name and the column names in “structure” section. 

	{
	    "name": "CopyActivity",
	    "description": "description", 
	    "type": "CopyActivity",
	    "inputs":  [ { "name": " AzureSQLInput"  } ],
	    "outputs":  [ { "name": " AzureBlobOutput" } ],
	    "typeProperties":
	    {
	        "source":
	        {
	            "type": "SqlSource",
	            "SqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartDateTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
	        },
	        "sink":
	        {
	            "type": "BlobSink"
	        },
	        "Translator": 
	        {
	            "type": "TabularTranslator",
	            "ColumnMappings": "UserId: MyUserId, Group: MyGroup,Name: MyName"
	        }
	    },
	    "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        }
	}

In this case, the query results are first mapped to columns specified in “structure” of source. Next, the columns from source “structure” are mapped to columns in sink “structure” with rules specified in columnMappings.  Suppose the query returns 5 columns, two additional columns then those specified in the “structure” of source.

**Column mapping flow**

![Column mapping flow-2](./media/data-factory-data-stores-with-rectangular-tables/column-mapping-flow-2.png)









