<properties
    pageTitle="How to use Azure Table Storage from Ruby | Microsoft Azure"
    description="Learn how to use Azure Table Storage in Azure. Code samples are written using the Ruby API."
    services="storage"
    documentationCenter="ruby"
    authors="tfitzmac"
    manager="wpickett"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="tomfitz"/>


# How to use Azure Table Storage from Ruby
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-tables.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-table-storage.md)
- [Java](../articles/storage/storage-java-how-to-use-table-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-tables.md)
- [PHP](../articles/storage/storage-php-how-to-use-table-storage.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-table-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-table-storage.md)

## Overview
This guide shows you how to perform common scenarios using the Azure Table service. The samples are written using the Ruby API. The scenarios covered include **creating and deleting a table, inserting and querying entities in a table**.

## What is the Table Service

The Azure Table storage service stores large amounts of
structured data. The service is a NoSQL datastore which accepts
authenticated calls from inside and outside the Azure cloud. Azure
tables are ideal for storing structured, non-relational data. Common
uses of the Table service include:

-   Storing TBs of structured data capable of serving web scale
    applications
-   Storing datasets that don't require complex joins, foreign keys, or
    stored procedures and can be denormalized for fast access
-   Quickly querying data using a clustered index
-   Accessing data using the OData protocol and LINQ queries with WCF
    Data Service .NET Libraries

You can use the Table service to store and query huge sets of
structured, non-relational data, and your tables will scale as demand
increases.

## Table Service Concepts

The Table service contains the following components:

![Table1][Table1]

-   **URL format:** Code addresses tables in an account using this
    address format:   
    http://`<storage account>`.table.core.windows.net/`<table>`  
      
    You can address Azure tables directly using this address with the
    OData protocol. For more information, see [OData.org][]

-   **Storage Account:** All access to Azure Storage is done
    through a storage account. See [Azure Storage Scalability and Performance Targets](storage-scalability-targets.md) for details about storage account capacity.

-   **Table**: A table is a collection of entities. Tables don't enforce
    a schema on entities, which means a single table can contain
    entities that have different sets of properties. The number of tables that a 
	storage account can contain is limited only by the 
    storage account capacity limit.

-   **Entity**: An entity is a set of properties, similar to a database
    row. An entity can be up to 1MB in size.

-   **Properties**: A property is a name-value pair. Each entity can
    include up to 252 properties to store data. Each entity also has 3
    system properties that specify a partition key, a row key, and a
    timestamp. Entities with the same partition key can be queried more
    quickly, and inserted/updated in atomic operations. An entity's row
    key is its unique identifier within a partition.


  
  [Table1]: ./media/storage-table-concepts-include/table1.png
  [OData.org]: http://www.odata.org/


## Create an Azure storage account

The easiest way to create your first Azure storage account is by using the [Azure Management Portal](https://manage.windowsazure.com). To learn more, see [Create a storage account](../articles/storage/storage-create-storage-account.md#create-a-storage-account).

You can create an Azure storage account by using [Azure PowerShell](../articles/storage/storage-powershell-guide-full.md) or [Azure CLI](../articles/storage/storage-azure-cli.md), or by using the [Azure Storage Resource Provider REST API](https://msdn.microsoft.com/library/azure/mt163683.aspx).
 

## Create a Ruby application
For instructions how to create a Ruby application,
see [Create a Ruby application in Azure](/develop/ruby/tutorials/web-app-with-linux-vm/).

## Configure your application to access Storage
To use Azure Storage, you need to download and use the Ruby azure package which includes a set of convenience libraries that communicate with the Storage REST services.

### Use RubyGems to obtain the package
1. Use a command-line interface such as **PowerShell** (Windows), **Terminal** (Mac), or **Bash** (Unix).

2. Type **gem install azure** in the command window to install the gem and dependencies.


### Import the package
Use your favorite text editor, add the following to the top of the Ruby file where you intend to use Storage:

    require "azure"

## Set up an Azure Storage connection
The azure module will read the environment variables **AZURE\_STORAGE\_ACCOUNT** and **AZURE\_STORAGE\_ACCESS\_KEY** for information required to connect to your Azure Storage account. If these environment variables are not set, you must specify the account information before using **Azure::TableService** with the following code:

    Azure.config.storage_account_name = "<your azure storage account>"
    Azure.config.storage_access_key = "<your azure storage access key>"

To obtain these values:

1. Log in to the [Azure Portal](https://portal.azure.com).

2. Navigate to your storage account.

3. On the **Settings** blade, select **Keys**.

4. Copy the desired access key value.


## Create a table
The **Azure::TableService** object lets you work with tables and entities. To create a table, use the **create\_table()** method. The following example creates a table or print out the error if there is any.

    azure_table_service = Azure::TableService.new
    begin
      azure_table_service.create_table("testtable")
    rescue
      puts $!
    end

## Add an entity to a table
To add an entity, first create a hash object that defines your entity properties. Note that for every entity you must specify a **PartitionKey** and **RowKey**. These are the unique identifiers of your entities, and are values that can be queried much faster than your other properties. Azure Storage uses **PartitionKey** to automatically distribute the table's entities over many storage nodes. Entities with the same **PartitionKey** are stored on the same node. The **RowKey** is the unique ID of the entity within the partition it belongs to.

    entity = { "content" => "test entity",
      :PartitionKey => "test-partition-key", :RowKey => "1" }
    azure_table_service.insert_entity("testtable", entity)

## Update an entity
There are multiple methods available to update an existing entity:

* **update\_entity():** Update an existing entity by replacing it.
* **merge\_entity():** Updates an existing entity by merging new property values into the existing entity.
* **insert\_or\_merge\_entity():** Updates an existing entity by replacing it. If no entity exists, a new one will be inserted:
* **insert\_or\_replace\_entity():** Updates an existing entity by merging new property values into the existing entity. If no entity exists, a new one will be inserted.

The following example demonstrates updating an entity using **update\_entity()**:

    entity = { "content" => "test entity with updated content",
      :PartitionKey => "test-partition-key", :RowKey => "1" }
    azure_table_service.update_entity("testtable", entity)

With **update\_entity()** and **merge\_entity()**, if the entity that you are updating doesn't exist then the update operation will fail. Therefore if you wish to store an entity regardless of whether it already exists, you should instead use **insert\_or\_replace\_entity()** or **insert\_or\_merge\_entity()**.

## Work with groups of entities
Sometimes it makes sense to submit multiple operations together in a batch to ensure atomic processing by the server. To accomplish that, you first create a **Batch** object and then use the **execute\_batch()** method on **TableService**. The following example demonstrates submitting two entities with RowKey 2 and 3 in a batch. Notice that it only works for entities with the same PartitionKey.

    azure_table_service = Azure::TableService.new
    batch = Azure::Storage::Table::Batch.new("testtable",
      "test-partition-key") do
      insert "2", { "content" => "new content 2" }
      insert "3", { "content" => "new content 3" }
    end
    results = azure_table_service.execute_batch(batch)

## Query for an entity
To query an entity in a table, use the **get\_entity()** method, by passing the table name, **PartitionKey** and **RowKey**.

    result = azure_table_service.get_entity("testtable", "test-partition-key",
      "1")

## Query a set of entities
To query a set of entities in a table, create a query hash object and use the **query\_entities()** method. The following example demonstrates getting all the entities with the same **PartitionKey**:

    query = { :filter => "PartitionKey eq 'test-partition-key'" }
    result, token = azure_table_service.query_entities("testtable", query)

> [!NOTE]
> If the result set is too large for a single query to return, a continuation token will be returned which you can use to retrieve subsequent pages.
> 
> 
## Query a subset of entity properties
A query to a table can retrieve just a few properties from an entity. This technique, called "projection", reduces bandwidth and can improve query performance, especially for large entities. Use the select clause and pass the names of the properties you would like to bring over to the client.

    query = { :filter => "PartitionKey eq 'test-partition-key'",
      :select => ["content"] }
    result, token = azure_table_service.query_entities("testtable", query)

## Delete an entity
To delete an entity, use the **delete\_entity()** method. You need to pass in the name of the table which contains the entity, the PartitionKey and RowKey of the entity.

        azure_table_service.delete_entity("testtable", "test-partition-key", "1")

## Delete a table
To delete a table, use the **delete\_table()** method and pass in the name of the table you want to delete.

        azure_table_service.delete_table("testtable")

## Next steps
To learn about more complex storage tasks, follow these links:

* [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/)
* [Azure SDK for Ruby](http://github.com/WindowsAzure/azure-sdk-for-ruby) repository on GitHub
