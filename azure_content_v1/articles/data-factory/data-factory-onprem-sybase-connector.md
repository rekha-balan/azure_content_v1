<properties 
    pageTitle="Move data from Sybase | Azure Data Factory" 
    description="Learn about how to move data from Sybase Database using Azure Data Factory." 
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
    ms.date="11/12/2015" 
    ms.author="spelluru"/>

# Move data from Sybase using Azure Data Factory
This article outlines how you can use the Copy Activity in an Azure data factory to move data from Sybase to another data store. This article builds on the [data movement activities](data-factory-data-movement-activities.md) article which presents a general overview of data movement with copy activity and supported data store combinations.

Data Factory service supports connecting to on-premises Sybase sources using the Data Management Gateway. See [moving data between on-premises locations and cloud](data-factory-move-data-between-onprem-and-cloud.md) article to learn about Data Management Gateway and step-by-step instructions on setting up the gateway. 

**Note:** You need to leverage the gateway to connect to Sybase even if it is hosted in Azure IaaS VMs. If you are trying to connect to an instance of Sybase hosted in cloud, you can also install the gateway instance in the IaaS VM.

Data factory currently supports only moving data from Sybase to other data stores, not from other data stores to Sybase.

## Installation
For Data Management Gateway to connect to the Sybase Database, you need to install the [data provider for Sybase](http://go.microsoft.com/fwlink/?linkid=324846) on the same system as the Data Management Gateway.

> [!NOTE]
> See [Gateway Troubleshooting](data-factory-move-data-between-onprem-and-cloud.md#gateway-troubleshooting) for tips on troubleshooting connection/gateway related issues. 
> 
> 
## Sample: Copy data from Sybase to Azure Blob
This sample shows how to copy data from a Sybase database to an Azure Blob Storage. However, data can be copied **directly** to any of the sinks stated [here](data-factory-data-movement-activities.md#supported-data-stores) using the Copy Activity in Azure Data Factory.  

The sample has the following data factory entities:

1. A linked service of type [OnPremisesSybase](data-factory-onprem-sybase-connector.md#sybase-linked-service-properties).
2. A liked service of type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties).
3. An input [dataset](data-factory-create-datasets.md) of type [RelationalTable](data-factory-onprem-sybase-connector.md#sybase-dataset-type-properties).
4. An output [dataset](data-factory-create-datasets.md) of type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
5. The [pipeline](data-factory-create-pipelines.md) with Copy Activity that uses [RelationalSource](data-factory-onprem-sybase-connector.md#sybase-copy-activity-type-properties) and [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).

The sample copies data from a query result in Sybase database to a blob every hour. The JSON properties used in these samples are described in sections following the samples. 

As a first step, please setup the data management gateway as per the instructions in the [moving data between on-premises locations and cloud](data-factory-move-data-between-onprem-and-cloud.md) article.

**Sybase linked service:**

    {
        "name": "OnPremSybaseLinkedService",
        "properties": {
            "type": "OnPremisesSybase",
            "typeProperties": {
                "server": "<server>",
                "database": "<database>",
                "schema": "<schema>",
                "authenticationType": "<authentication type>",
                "username": "<username>",
                "password": "<password>",
                "gatewayName": "<gatewayName>"
            }
        }
    }

**Azure Blob storage linked service:**

    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorageLinkedService",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<AccountName>;AccountKey=<AccountKey>"
            }
        }
    }


**Sybase input dataset:**

The sample assumes you have created a table “MyTable” in Sybase and it contains a column called “timestamp” for time series data.

Setting “external”: true and specifying externalData policy tells data factory that this is a table that is external to the data factory and not produced by an activity in the data factory. Note that the **type** of the linked service is set to: **RelationalTable**. 

    {
        "name": "SybaseDataSet",
        "properties": {
            "type": "RelationalTable",
            "linkedServiceName": "OnPremSybaseLinkedService",
            "typeProperties": {},
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


**Azure Blob output dataset:**

Data is written to a new blob every hour (frequency: hour, interval: 1). The folder path for the blob is dynamically evaluated based on the start time of the slice that is being processed. The folder path uses year, month, day, and hours parts of the start time.

    {
        "name": "AzureBlobSybaseDataSet",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "mycontainer/sybase/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
                "format": {
                    "type": "TextFormat",
                    "rowDelimiter": "\n",
                    "columnDelimiter": "\t"
                },
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
                ]
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }


**Pipeline with Copy activity:**

The pipeline contains a Copy Activity that is configured to use the above input and output datasets and is scheduled to run every hour. In the pipeline JSON definition, the **source** type is set to **RelationalSource** and **sink** type is set to **BlobSink**. The SQL query specified for the **query** property selects the data from the DBA.Orders table in the database.

    {
        "name": "CopySybaseToBlob",
        "properties": {
            "description": "pipeline for copy activity",
            "activities": [
                {
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "RelationalSource",
                            "query": "select * from DBA.Orders"
                        },
                        "sink": {
                            "type": "BlobSink"
                        }
                    },
                    "inputs": [
                        {
                            "name": "SybaseDataSet"
                        }
                    ],
                    "outputs": [
                        {
                            "name": "AzureBlobSybaseDataSet"
                        }
                    ],
                    "policy": {
                        "timeout": "01:00:00",
                        "concurrency": 1
                    },
                    "scheduler": {
                        "frequency": "Hour",
                        "interval": 1
                    },
                    "name": "SybaseToBlob"
                }
            ],
            "start": "2014-06-01T18:00:00Z",
            "end": "2014-06-01T19:00:00Z"
        }
    }


## Sybase Linked Service properties
The following table provides description for JSON elements specific to Sybase linked service.

| Property | Description | Required |
| --- | --- | --- |
| type |The type property must be set to: **OnPremisesSybase** |Yes |
| server |Name of the Sybase server. |Yes |
| database |Name of the Sybase database. |Yes  |
| schema |Name of the schema in the database. |No |
| authenticationType |Type of authentication used to connect to the Sybase database. Possible values are: Anonymous, Basic, and Windows. |Yes |
| username |Specify user name if you are using Basic or Windows authentication. |No |
| password |Specify password for the user account you specified for the username. |No |
| gatewayName |Name of the gateway that the Data Factory service should use to connect to the on-premises Sybase database. |Yes  |

See [Setting Credentials and Security](data-factory-move-data-between-onprem-and-cloud.md#setting-credentials-and-security) for details about setting credentials for an on-premises Sybase data source.

## Sybase Dataset type properties
For a full list of sections & properties available for defining datasets, see the [Creating datasets](data-factory-create-datasets.md) article. Sections like structure, availability, and policy of a dataset JSON are similar for all dataset types (Azure SQL, Azure blob, Azure table, etc...).

The typeProperties section is different for each type of dataset and provides information about the location of the data in the data store. The **typeProperties** section for dataset of type **RelationalTable** (which includes Sybase dataset) has the following properties.

| Property | Description | Required |
| --- | --- | --- |
| tableName |Name of the table in the Sybase Database instance that linked service refers to. |No (if **query** of **RelationalSource** is specified) |

## Sybase Copy Activity type properties
For a full list of sections & properties available for defining activities, please see  [Creating Pipelines](data-factory-create-pipelines.md) article. Properties like name, description, input and output tables, various policies etc. are available for all types of activities. 

Properties available in the typeProperties section of the activity on the other hand vary with each activity type and in case of Copy activity they vary depending on the types of sources and sinks.

In case of Copy Activity when source is of type **RelationalSource** (which includes Sybase) the following properties are available in **typeProperties** section:

| Property | Description | Allowed values | Required |
| --- | --- | --- | --- |
| query |Use the custom query to read data. |SQL query string. For example: select * from MyTable. |No (if **tableName** of **dataset** is specified) |

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



## Type mapping for Sybase
As mentioned in the data movement activities article Copy activity performs automatic type conversions from automatic type conversions from source types to sink types with the following 2 step approach:

1. Convert from native source types to .NET type
2. Convert from .NET type to native sink type

Sybase supports T-SQL and T-SQL types. For a mapping table from sql types to .NET type, see  [Azure SQL Connector](data-factory-azure-sql-connector.md) article.

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









## Repeatability during Copy

When copying data from and to relational stores, you need to keep repeatability in mind to avoid unintended outcomes. 

**Note:** A slice can be re-run automatically in Azure Data Factory as per the retry policy specified. It is recommended to set a retry policy to guard against transient failures. Hence repeatability is an important aspect to take care of during data movement. 

**As a source:**

In most cases when reading from relational stores, you would want to read only the data corresponding to that slice. A way to do so would be by using the WindowStart and WindowEnd variables available in Azure Data Factory. Read about the variables and functions in Azure Data Factory here in the [Scheduling and Execution](data-factory-scheduling-and-execution.md) article. Example: 
	
	  "source": {
	    "type": "SqlSource",
	    "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm\\'', WindowStart, WindowEnd)"
	  },

The above query will read data from ‘MyTable’ that falls in the slice duration range. Re-run of this slice would also always ensure this behavior. 

In other cases, you may wish to read the entire Table (suppose for one time move only) and may define the sqlReaderQuery as follows:

	
	"source": {
	            "type": "SqlSource",
	            "sqlReaderQuery": "select * from MyTable"
	          },
	


