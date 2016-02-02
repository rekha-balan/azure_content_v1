<properties
    pageTitle="How to use Table storage from Python | Microsoft Azure"
    description="Learn how you can use the Table service from Python to create and delete a table, and to insert and query a table."
    services="storage"
    documentationCenter="python"
    authors="emgerner-msft"
    manager="wpickett"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="12/11/2015"
    ms.author="emgerner"/>


# How to use Table storage from Python
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-tables.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-table-storage.md)
- [Java](../articles/storage/storage-java-how-to-use-table-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-tables.md)
- [PHP](../articles/storage/storage-php-how-to-use-table-storage.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-table-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-table-storage.md)

## Overview
This guide shows you how to perform common scenarios by using the Azure Table storage service. The samples are written in Python and use the [Python Azure Storage package](https://pypi.python.org/pypi/azure-storage). The covered scenarios include creating and deleting a
table, in addition to inserting and querying entities in a table.

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
 

[AZURE.NOTE]AZURE.NOTE] If you need to install Python or the [Python Azure package](https://pypi.python.org/pypi/azure), see the [Python installation guide](../python-how-to-install.md).

## Create a table
The **TableService** object lets you work with table services. The
following code creates a **TableService** object. Add the code near
the top of any Python file in which you wish to programmatically access Azure Storage:

    from azure.storage.table import TableService, Entity

The following code creates a **TableService** object by using the storage account name and account key.  Replace 'myaccount' and 'mykey' with the real account and key.

    table_service = TableService(account_name='myaccount', account_key='mykey')

    table_service.create_table('tasktable')

## Add an entity to a table
To add an entity, first create a dictionary that defines your entity
property names and values. Note that for every entity, you must
specify **PartitionKey** and **RowKey**. These are the unique
identifiers of your entities. You can query these values much
faster than you can query your other properties. The system uses **PartitionKey** to
automatically distribute the table entities over many storage nodes.
Entities that have the same **PartitionKey** are stored on the same node. **RowKey** is the unique ID of the entity within the partition that it
belongs to.

To add an entity to your table, pass a dictionary object
to the **insert\_entity** method.

    task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the trash', 'priority' : 200}
    table_service.insert_entity('tasktable', task)

You can also pass an instance of the **Entity** class to the **insert\_entity** method.

    task = Entity()
    task.PartitionKey = 'tasksSeattle'
    task.RowKey = '2'
    task.description = 'Wash the car'
    task.priority = 100
    table_service.insert_entity('tasktable', task)

## Update an entity
This code shows how to replace the old version of an existing entity
with an updated version.

    task = {'description' : 'Take out the garbage', 'priority' : 250}
    table_service.update_entity('tasktable', 'tasksSeattle', '1', task)

If the entity that is being updated does not exist, then the update
operation will fail. If you want to store an entity
regardless of whether it existed before, use **insert\_or\_replace_entity**.
In the following example, the first call will replace the existing entity. The second call will insert a new entity, since no entity with the specified **PartitionKey** and **RowKey** exists in the table.

    task = {'description' : 'Take out the garbage again', 'priority' : 250}
    table_service.insert_or_replace_entity('tasktable', 'tasksSeattle', '1', task)

    task = {'description' : 'Buy detergent', 'priority' : 300}
    table_service.insert_or_replace_entity('tasktable', 'tasksSeattle', '3', task)

## Change a group of entities
Sometimes it makes sense to submit multiple operations together in a
batch to ensure atomic processing by the server. To accomplish that, you
use the **begin\_batch** method on **TableService** and then call the
series of operations as usual. When you do want to submit the
batch, you call **commit\_batch**. Note that all entities must be in the same partition in order to be changed as a batch. The example below adds two entities together in a batch.

    task10 = {'PartitionKey': 'tasksSeattle', 'RowKey': '10', 'description' : 'Go grocery shopping', 'priority' : 400}
    task11 = {'PartitionKey': 'tasksSeattle', 'RowKey': '11', 'description' : 'Clean the bathroom', 'priority' : 100}
    table_service.begin_batch()
    table_service.insert_entity('tasktable', task10)
    table_service.insert_entity('tasktable', task11)
    table_service.commit_batch()

## Query for an entity
To query an entity in a table, use the **get\_entity** method by
passing **PartitionKey** and **RowKey**.

    task = table_service.get_entity('tasktable', 'tasksSeattle', '1')
    print(task.description)
    print(task.priority)

## Query a set of entities
This example finds all tasks in Seattle based on **PartitionKey**.

    tasks = table_service.query_entities('tasktable', "PartitionKey eq 'tasksSeattle'")
    for task in tasks:
        print(task.description)
        print(task.priority)

## Query a subset of entity properties
A query to a table can retrieve just a few properties from an entity.
This technique, called *projection*, reduces bandwidth and can improve
query performance, especially for large entities. Use the **select**
parameter and pass the names of the properties that you want to bring over
to the client.

The query in the following code returns only the descriptions of
entities in the table.

[AZURE.NOTE]AZURE.NOTE] The following snippet works only against the cloud
storage service. This is not supported by the storage
emulator.

    tasks = table_service.query_entities('tasktable', "PartitionKey eq 'tasksSeattle'", 'description')
    for task in tasks:
        print(task.description)

## Delete an entity
You can delete an entity by using its partition and row key.

    table_service.delete_entity('tasktable', 'tasksSeattle', '1')

## Delete a table
The following code deletes a table from a storage account.

    table_service.delete_table('tasktable')

## Next steps
Now that you have learned the basics of Table storage, follow these links
to learn about more complex storage tasks:

* See the MSDN reference [Azure Storage][]Azure Storage][]].
* Visit the [Azure Storage Team blog](http://blogs.msdn.com/b/windowsazurestorage/).

For more information, see also the [Python Developer Center](/develop/python/).

[Azure Storage Team blog]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure package]: https://pypi.python.org/pypi/azure
[Python Azure Storage package]: https://pypi.python.org/pypi/azure-storage
