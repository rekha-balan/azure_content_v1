<properties 
    pageTitle="Move data to and from Azure Blob | Azure Data Factory" 
    description="Learn how to move data to/from Azure Blob Storage using Azure Data Factory" 
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
    ms.date="01/26/2016" 
    ms.author="spelluru"/>

# Move data to and from Azure Blob using Azure Data Factory
This article outlines how you can use the Copy Activity in an Azure data factory to move data to Azure Blob from another data store and move data from another data store to Azure Blob. This article builds on the [data movement activities](data-factory-data-movement-activities.md) article which presents a general overview of data movement with the copy activity and the supported data store combinations.

The following sample(s) show how to copy data to and from Azure Blob Storage and Azure SQL Database. However, data can be copied **directly** from any of sources to any of the sinks stated [here](data-factory-data-movement-activities.md#supported-data-stores) using the Copy Activity in Azure Data Factory.  

## Sample: Copy data from Azure Blob to Azure SQL
The sample below shows:

1. A linked service of type [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties).
2. A linked service of type [AzureStorage](#azure-storage-linked-service-properties.md).
3. An input [dataset](data-factory-create-datasets.md) of type [AzureBlob](#azure-blob-dataset-type-properties.md).
4. An output [dataset](data-factory-create-datasets.md) of type [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties).
5. A [pipeline](data-factory-create-pipelines.md) with a Copy activity that uses [BlobSource](#azure-blob-copy-activity-type-properties.md) and [SqlSink](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties).

The sample copies data belonging to a time series from an Azure blob to a table in an Azure SQL database every hour. The JSON properties used in these samples are described in sections following the samples. 

**Azure SQL linked service:**

    {
      "name": "AzureSqlLinkedService",
      "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
          "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
      }
    }

**Azure Storage linked service:**

    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

Azure Data Factory supports two types of Azure Storage linked services: **AzureStorage** and **AzureStorageSas**. For the first one, you specify the connection string that includes the account key and for the later one, you specify the Shared Access Signature (SAS) Uri.   See [Linked Services](#linked-services.md) section for details.  

**Azure Blob input dataset:**

Data is picked up from a new blob every hour (frequency: hour, interval: 1). The folder path and file name for the blob are dynamically evaluated based on the start time of the slice that is being processed. The folder path uses year, month, and day part of the start time and file name uses the hour part of the start time. “external”: “true” setting informs the Data Factory service that this table is external to the data factory and not produced by an activity in the data factory.

    {
      "name": "AzureBlobInput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
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

**Azure SQL output dataset:**

The sample copies data to a table named “MyTable” in an Azure SQL database. You should create the table in your Azure SQL database with the same number of columns as you expect the Blob CSV file to contain. New rows are added to the table every hour.

    {
      "name": "AzureSqlOutput",
      "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
          "tableName": "MyOutputTable"
        },
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }

**Pipeline with a Copy activity:**

The pipeline contains a Copy Activity that is configured to use the above input and output datasets and is scheduled to run every hour. In the pipeline JSON definition, the **source** type is set to **BlobSource** and **sink** type is set to **SqlSink**. 

    {  
        "name":"SamplePipeline",
        "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[  
          {
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [
              {
                "name": "AzureBlobInput"
              }
            ],
            "outputs": [
              {
                "name": "AzureSqlOutput"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource",
                "blobColumnSeparators": ","
              },
              "sink": {
                "type": "SqlSink"
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

## Sample: Copy data from Azure SQL to Azure Blob
The sample below shows:

1. A linked service of type [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties).
2. A linked service of type [AzureStorage](#azure-storage-linked-service-properties.md).
3. An input [dataset](data-factory-create-datasets.md) of type [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties).
4. An output [dataset](data-factory-create-datasets.md) of type [AzureBlob](#azure-blob-dataset-type-properties.md).
5. A [pipeline](data-factory-create-pipelines.md) with Copy activity that uses [SqlSource](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties) and [BlobSink](#azure-blob-copy-activity-type-properties.md).

The sample copies data belonging to a time series from a table in Azure SQL database to a blob every hour. The JSON properties used in these samples are described in sections following the samples. 

**Azure SQL linked service:**

    {
      "name": "AzureSqlLinkedService",
      "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
          "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
      }
    }

**Azure Storage linked service:**

    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

Azure Data Factory supports two types of Azure Storage linked services: **AzureStorage** and **AzureStorageSas**. For the first one, you specify the connection string that includes the account key and for the later one, you specify the Shared Access Signature (SAS) Uri.   See [Linked Services](#linked-services.md) section for details.  

**Azure SQL input dataset:**

The sample assumes you have created a table “MyTable” in Azure SQL and it contains a column called “timestampcolumn” for time series data. 

Setting “external”: ”true” and specifying externalData policy informs the Azure Data Factory service that this is a table that is external to the data factory and not produced by an activity in the data factory.

    {
      "name": "AzureSqlInput",
      "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
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
          "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
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

**Pipeline with the Copy activity:**

The pipeline contains a Copy Activity that is configured to use the above input and output datasets and is scheduled to run every hour. In the pipeline JSON definition, the **source** type is set to **SqlSource** and **sink** type is set to **BlobSink**. The SQL query specified for the **SqlReaderQuery** property selects the data in the past hour to copy.

    {  
        "name":"SamplePipeline",
        "properties":{  
            "start":"2014-06-01T18:00:00",
            "end":"2014-06-01T19:00:00",
            "description":"pipeline for copy activity",
            "activities":[  
                  {
                    "name": "AzureSQLtoBlob",
                    "description": "copy activity",
                    "type": "Copy",
                    "inputs": [
                      {
                        "name": "AzureSQLInput"
                      }
                    ],
                    "outputs": [
                      {
                        "name": "AzureBlobOutput"
                      }
                    ],
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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

## Linked Services
There are two types of linked services you can use to link an Azure blob storage to an Azure data factory. They are: **AzureStorage** linked service and **AzureStorageSas** linked service. The Azure Storage linked service provides the data factory with global access to the Azure Storage. Whereas, The Azure Storage SAS (Shared Access Signature) linked service provides the data factory with restricted/time-bound access to the Azure Storage. There are no other differences between these two linked services. Choose the linked service that suits your needs. The following sections provide more details on these two linked services.

## Azure Storage Linked Service

The **Azure Storage linked service** allows you to link an Azure storage account to an Azure data factory by using the **account key**. This provides the data factory with global access to the Azure Storage. The following table provides description for JSON elements specific to Azure Storage linked service.

| Property | Description | Required |
| :-------- | :----------- | :-------- |
| type | The type property must be set to: **AzureStorage** | Yes |
| connectionString | Specify information needed to connect to Azure storage for the connectionString property. | Yes |

See the following article for steps to view/copy the account key for an Azure Storage: [View, copy, and regenerate storage access keys](../storage/storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys).

**Example:**  
  
	{  
		"name": "StorageLinkedService",  
		"properties": {  
			"type": "AzureStorage",  
			"typeProperties": {  
				"connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"  
			}  
		}  
	}  


## Azure Storage Sas Linked Service  
A shared access signature (SAS) provides delegated access to resources in your storage account. This means that you can grant a client limited permissions to objects in your storage account for a specified period of time and with a specified set of permissions, without having to share your account access keys. The SAS is a URI that encompasses in its query parameters all of the information necessary for authenticated access to a storage resource. To access storage resources with the SAS, the client only needs to pass in the SAS to the appropriate constructor or method. For detailed information about SAS, see [Shared Access Signatures: Understanding the SAS Model](../storage/storage-dotnet-shared-access-signature-part-1.md)
  
The Azure Storage SAS linked service allows you to link an Azure Storage Account to an Azure data factory by using a Shared Access Signature (SAS). This provides the data factory with restricted/time-bound access to all/specific resources (blob/container) in the storage. The following table provides description for JSON elements specific to Azure Storage SAS linked service. 

| Property | Description | Required |
| :-------- | :----------- | :-------- |
| type | The type property must be set to: **AzureStorageSas**  | Yes |
| sasUri | Specify Shared Access Signature URI to the Azure Storage resources such as blob, container, or table. See the notes below for details. | Yes | 


**Example:**
  
	{  
		"name": "StorageSasLinkedService",  
		"properties": {  
			"type": "AzureStorageSas",  
			"typeProperties": {  
				"sasUri": "<storageUri>?<sasToken>"   
			}  
		}  
	}  

When creating an **SAS URI**, considering the following:  

- Azure Data Factory supports only **Service SAS**, not Account SAS. See [Types of Shared Access Signatures](../storage/storage-dotnet-shared-access-signature-part-1.md#types-of-shared-access-signatures) for details about these two types.
- Appropriate read/write **permissions** need to be set on objects based on how the linked service (read, write, read/write) will be used in your data factory.
- **Expiry time** needs to be set appropriately. Make sure that the access to Azure Storage objects does not expire within the active period of the pipeline.
- Uri should be created at the right container/blob or Table level based on the need. A SAS Uri to an Azure blob allows the Data Factory service to access that particular blob. A SAS Uri to an Azure blob container allows the Data Factory service to iterate through blobs in that container. If you need to provide access more/fewer objects later, or update the SAS URI, remember to update the linked service with the new URI.   


## Azure Blob Dataset type properties
For a full list of JSON sections & properties available for defining datasets, see the [Creating datasets](data-factory-create-datasets.md) article. Sections like structure, availability, and policy of a dataset JSON are similar for all dataset types (Azure SQL, Azure blob, Azure table, etc...).

The **typeProperties** section is different for each type of dataset and provides information about the location, format etc. of the data in the data store. The typeProperties section for dataset of type **AzureBlob** dataset has the following properties.

| Property | Description | Required |
| --- | --- | --- |
| folderPath |Path to the container and folder in the blob storage. Example: myblobcontainer\myblobfolder\ |Yes |
| fileName |<p>Name of the blob. fileName is optional. </p><p>If you specify a filename, the activity (including Copy) works on the specific Blob.</p><p>When fileName is not specified Copy will include all Blobs in the folderPath for input dataset.</p><p>When fileName is not specified for an output dataset, the name of the generated file would be in the following this format: Data.<Guid>.txt (for example: : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt</p> |No |
| partitionedBy |partitionedBy is an optional property. You can use it to specify a dynamic folderPath and filename for time series data. For example, folderPath can be parameterized for every hour of data. See the [Leveraging partitionedBy property section](#Leveraging-partitionedBy-property.md) below for details and examples. |No |
| format |Two formats types are supported: **TextFormat**, **AvroFormat**. You need to set the type property under format to either of these values. When the format is TextFormat you can specify additional optional properties for format. See the [Specifying TextFormat](#specifying-textformat.md) section below for more details. |No |
| compression |Specify the type and level of compression for the data. Supported types are: GZip, Deflate, and BZip2 and supported levels are: Optimal and Fastest. See [Compression support](#compression-support.md) section for more details. |No |

### Leveraging partitionedBy property
As mentioned above, you can specify a dynamic folderPath and filename for time series data with the **partitionedBy** section, Data Factory macros and the system variables: SliceStart and SliceEnd, which indicate start and end times for a given data slice.

See [Data Factory System Variables](data-factory-scheduling-and-execution.md#data-factory-system-variables) and [Data Factory Functions Reference](data-factory-scheduling-and-execution.md#data-factory-functions-reference) to learn about Data Factory system variables and functions that you can use in the partitionedBy section.   

See [Creating Datasets](data-factory-create-datasets.md) and [Scheduling & Execution](data-factory-scheduling-and-execution.md) articles for more details on time series datasets, scheduling and slices.

#### Sample 1
    "folderPath": "wikidatagateway/wikisampledataout/{Slice}",
    "partitionedBy": 
    [
        { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
    ],

In the above example {Slice} is replaced with the value of Data Factory system variable SliceStart in the format (YYYYMMDDHH) specified. The SliceStart refers to start time of the slice. The folderPath is different for each slice. For example: wikidatagateway/wikisampledataout/2014100103 or wikidatagateway/wikisampledataout/2014100104

#### Sample 2
    "folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
    "fileName": "{Hour}.csv",
    "partitionedBy": 
     [
        { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } }, 
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }, 
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } } 
    ],

In the above example, year, month, day, and time of SliceStart are extracted into separate variables that are used by folderPath and fileName properties.

### Specifying TextFormat
If the format is set to **TextFormat**, you can specify the following **optional** properties in the **Format** section.

| Property | Description | Required |
| --- | --- | --- |
| columnDelimiter |The character used as a column separator in a file. Only one character is allowed at this time. This tag is optional. The default value is comma (,). |No |
| rowDelimiter |The character used as a raw separator in file. Only one character is allowed at this time. This tag is optional. The default value is any of the following: [“\r\n”, “\r”,” \n”]“\r\n”, “\r”,” \n”]. |No |
| escapeChar |<p>The special character used to escape column delimiter shown in content. This tag is optional. No default value. You must specify no more than one character for this property.</p><p>For example, if you have comma (,) as the column delimiter but you want have comma character in the text (example: “Hello, world”), you can define ‘$’ as the escape character and use string “Hello$, world” in the source.</p><p>Note that you cannot specify both escapeChar and quoteChar for a table.</p> |No |
| quoteChar |<p>The special character is used to quote the string value. The column and row delimiters inside of the quote characters would be treated as part of the string value. This tag is optional. No default value. You must specify no more than one character for this property.</p><p>For example, if you have comma (,) as the column delimiter but you want have comma character in the text (example: <Hello, world>), you can define ‘"’ as the quote character and use string <"Hello, world"> in the source. This property is applicable to both input and output tables.</p><p>Note that you cannot specify both escapeChar and quoteChar for a table.</p> |No |
| nullValue |<p>The character(s) used to represent null value in blob file content. This tag is optional. The default value is “\N”.</p><p>For example, based on above sample, “NaN” in blob will be translated as null value while copied into e.g. SQL Server.</p> |No |
| encodingName |Specify the encoding name. For the list of valid encoding names, see: [Encoding.EncodingName Property](https://msdn.microsoft.com/library/system.text.encoding.aspx). For example: windows-1250 or shift_jis. The default value is: UTF-8. |No |

#### Samples
The following sample shows some of the format properties for TextFormat.

    "typeProperties":
    {
        "folderPath": "mycontainer/myfolder",
        "fileName": "myblobname"
        "format":
        {
            "type": "TextFormat",
            "columnDelimiter": ",",
            "rowDelimiter": ";",
            "quoteChar": "\"",
            "NullValue": "NaN"
        }
    },

To use an escapeChar instead of quoteChar, replace the line with quoteChar with the following:

    "escapeChar": "$",

### Specifying AvroFormat
If the format is set to AvroFormat, you do not need to specify any properties in the Format section within the typeProperties section. Example:

    "format":
    {
        "type": "AvroFormat",
    }

To use Avro format in a Hive table, you can refer to [Apache Hive’s tutorial](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe).

### Compression support  
Processing large data sets can cause I/O and network bottlenecks. Therefore, compressed data in stores can not only speed up data transfer across the network and save disk space, but also bring significant performance improvements in processing big data. At this time, compression is supported for file-based data stores such as Azure Blob or On-premises File System.  

To specify compression for a dataset, use the **compression** property in the dataset JSON as in the following example:   

	{  
		"name": "AzureBlobDataSet",  
	  	"properties": {  
	    	"availability": {  
	    		"frequency": "Day",  
	    	  	"interval": 1  
	    	},  
	    	"type": "AzureBlob",  
	    	"linkedServiceName": "StorageLinkedService",  
	    	"typeProperties": {  
	    		"fileName": "pagecounts.csv.gz",  
	    	  	"folderPath": "compression/file/",  
	    	  	"compression": {  
	    	    	"type": "GZip",  
	    	    	"level": "Optimal"  
	    	  	}  
    		}  
	  	}  
	}  
 
Note that the **compression** section has two properties:  
  
- **Type:** the compression codec, which can be **GZIP**, **Deflate** or **BZIP2**.  
- **Level:** the compression ratio, which can be **Optimal** or **Fastest**. 
	- **Fastest:** The compression operation should complete as quickly as possible, even if the resulting file is not optimally compressed. 
	- **Optimal**: The compression operation should be optimally compressed, even if the operation takes a longer time to complete. 
	
	See [Compression Level](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) topic for more information. 

Suppose the above sample dataset is used as the output of a copy activity, the copy activity will compresses the output data with GZIP codec using optimal ratio and then write the compressed data into a file named pagecounts.csv.gz in the Azure Blob Storage.   

When you specify compression property in an input dataset JSON, the pipeline can read compressed data from the source and when you specify the property in an output dataset JSON, the copy activity can write compressed data to the destination. Here are a few sample scenarios: 

- Read GZIP compressed data from an Azure blob, decompress it, and write result data to an Azure SQL database. You define the input Azure Blob dataset with the compression JSON property in this case. 
- Read data from a plain-text file from on-premises File System, compress it using GZip format, and write the compressed data to an Azure blob. You define an output Azure Blob dataset with the compression JSON property in this case.  
- Read a GZIP-compressed data from an Azure blob, decompress it, compress it using BZIP2, and write result data to an Azure blob. You define the input Azure Blob dataset with compression type set to GZIP and the output dataset with compression type set to BZIP2 in this case.   


## Azure Blob Copy Activity type properties
For a full list of sections & properties available for defining activities, see the [Creating Pipelines](data-factory-create-pipelines.md) article. Properties like name, description, input and output tables, various policies etc are available for all types of activities.

Properties available in the typeProperties section of the activity on the other hand vary with each activity type and in case of Copy activity they vary depending on the types of sources and sinks

**BlobSource** supports the following properties in the **typeProperties** section:

| Property | Description | Allowed values | Required |
| --- | --- | --- | --- |
| treatEmptyAsNull |Specifies whether to treat null or empty string as null value. |TRUE<br/>FALSE |No |
| skipHeaderLineCount |Indicates how many lines need be skipped. It is applicable only when input dataset is using **TextFormat**. |Integer from 0 to Max. |No |
| recursive |Indicates whether the data is read recursively from the sub folders or only from the specified folder. |True (default value), False |No |

**BlobSink** supports the following properties **typeProperties** section:

| Property | Description | Allowed values | Required |
| --- | --- | --- | --- |
| blobWriterAddHeader |Specifies whether to add header of column definitions. |TRUE<br/>FALSE (default) |No |
| copyBehavior |Defines the copy behavior when the source is BlobSource or FileSystem. |<p>There are three possible values for the copyBehavior property. </p><ul><li>**PreserveHierarchy:** preserves the file hierarchy in the target folder, i.e., the relative path of source file to source folder is identical to the relative path of target file to target folder.</li><li>**FlattenHierarchy:** all files from the source folder will be in the first level of target folder. The target files will have auto generated name. </li><li>**MergeFiles: (default)** merges all files from the source folder to one file. If the File/Blob Name is specified, the merged file name would be the specified name; otherwise, would be auto-generated file name.</li></ul> |No |

### recursive and copyBehavior examples
This section describes the resulting behavior of the Copy operation for different combinations of recursive and copyBehavior values. 

| recursive | copyBehavior | Resulting behavior |
| --- | --- | --- |
| true |preserveHierarchy |<p>For a source folder Folder1 with the following structure:</p> <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>the target folder Folder1 will have the same structure as the source<p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>.   |
| true |flattenHierarchy |<p>For a source folder Folder1 with the following structure:</p> <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>the target Folder1 will have the following structure: <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File5</p> |
| true |mergeFiles |<p>For a source folder Folder1 with the following structure:</p> <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>the target Folder1 will have the following structure: <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File 5 contents will be merged into one file with auto-generated file name</p> |
| false |preserveHierarchy |<p>For a source folder Folder1 with the following structure:</p> <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>the target folder Folder1 will have the following structure<p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/></p><p>Subfolder1 with File3, File4, and File5 are not picked up.</p>. |
| false |flattenHierarchy |<p>For a source folder Folder1 with the following structure:</p> <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>the target folder Folder1 will have the following structure<p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;auto-generated name for File2<br/></p><p>Subfolder1 with File3, File4, and File5 are not picked up.</p>. |
| false |mergeFiles |<p>For a source folder Folder1 with the following structure:</p> <p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5</p>the target folder Folder1 will have the following structure<p>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 contents will be merged into one file with auto-generated file name. auto-generated name for File1</p><p>Subfolder1 with File3, File4, and File5 are not picked up.</p>. |

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









