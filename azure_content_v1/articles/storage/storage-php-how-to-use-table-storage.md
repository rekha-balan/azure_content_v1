<properties
    pageTitle="How to use table storage from PHP | Microsoft Azure"
    description="Learn how to use the Table service from PHP to create and delete a table, and insert, delete, and query the table."
    services="storage"
    documentationCenter="php"
    authors="tfitzmac"
    manager="carmonm"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="PHP"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="tomfitz"/>


# How to use table storage from PHP
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-tables.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-table-storage.md)
- [Java](../articles/storage/storage-java-how-to-use-table-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-tables.md)
- [PHP](../articles/storage/storage-php-how-to-use-table-storage.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-table-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-table-storage.md)

## Overview
This guide shows you how to perform common scenarios using the Azure Table service. The samples are written in PHP and use the [Azure SDK for PHP](http://go.microsoft.com/fwlink/?LinkID=252473). The scenarios covered include **creating and deleting a table, and inserting, deleting, and querying entities in a table**. For more information on the Azure Table service, see the [Next steps](#NextSteps.md) section.

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
 

## Create a PHP application
The only requirement for creating a PHP application that accesses the Azure Table service is the referencing of classes in the Azure SDK for PHP from within your code. You can use any development tools to create your application, including Notepad.

In this guide, you use Table service features which can be called from within a PHP application locally, or in code running within an Azure web role, worker role, or website.

## Get the Azure Client Libraries
### Install via Composer

1. [Install Git][install-git]. Note that on Windows, you must also add the Git executable to your PATH environment variable. 

2. Create a file named **composer.json** in the root of your project and add the following code to it:

	```
	{
	    "repositories": [
	        {
	            "type": "pear",
	            "url": "http://pear.php.net"
	        }
	    ],
	    "require": {
	        "pear-pear.php.net/mail_mime" : "*",
	        "pear-pear.php.net/http_request2" : "*",
	        "pear-pear.php.net/mail_mimedecode" : "*",
	        "microsoft/windowsazure": "*"
	    }
	}
	```

3. Download **[composer.phar][composer-phar]** in your project root.

4. Open a command prompt and execute the following command in your project root

	```
	php composer.phar install
	```

### Install manually

To download and install the PHP Client Libraries for Azure manually, follow these steps:

> [AZURE.NOTE] The PHP Client Libraries for Azure have a dependency on the [HTTP_Request2](http://pear.php.net/package/HTTP_Request2), [Mail_mime](http://pear.php.net/package/Mail_mime), and [Mail_mimeDecode](http://pear.php.net/package/Mail_mimeDecode) PEAR packages. The recommended way to resolve these dependencies is to install these packages using the [PEAR package manager](http://pear.php.net/manual/en/installation.php).
 
1. Download a .zip archive that contains the libraries from [GitHub][php-sdk-github]. Alternatively, fork the repository and clone it to your local machine. The latter option requires a GitHub account and having Git installed locally.
	
2. Copy the `WindowsAzure` directory of the downloaded archive to your application directory structure.

For more information about installing the PHP Client Libraries for Azure (including information about installing as a PEAR package), see [Download the Azure SDK for PHP][download-SDK-PHP].

[php-sdk-github]: http://go.microsoft.com/fwlink/?LinkId=252719
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[download-SDK-PHP]: ../articles/php-download-sdk.md
[composer-phar]: http://getcomposer.org/composer.phar


## Configure your application to access the Table service
To use the Azure Table service APIs, you need to:

1. Reference the autoloader file using the [require_once](http://php.net/require_once) statement, and
2. Reference any classes you might use.

The following example shows how to include the autoloader file and reference the **ServicesBuilder** class.

> [!NOTE]
> This example (and other examples in this article) assume you have installed the PHP Client Libraries for Azure via Composer. If you installed the libraries manually or as a PEAR package, you need to reference the <code>WindowsAzure.php</code> autoloader file.
> 
> 
    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;


In the examples below, the `require_once` statement is always shown, but only the classes necessary for the example to execute are referenced.

## Set up an Azure storage connection
To instantiate an Azure Table service client, you must first have a valid connection string. The format for the Table service connection string is:

For accessing a live service:

    DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

For accessing the emulator storage:

    UseDevelopmentStorage=true


To create any Azure service client, you need to use the **ServicesBuilder** class. You can:

* pass the connection string directly to it or
* use the **CloudConfigurationManager (CCM)** to check multiple external sources for the connection string:  * by default, it comes with support for one external source - environmental variables
* you can add new sources by extending the **ConnectionStringSource** class



For the examples outlined here, the connection string will be passed directly.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);


## Create a table
A **TableRestProxy** object lets you create a table with the **createTable** method. When creating a table, you can set the Table service timeout. (For more information about the Table service timeout, see [Setting timeouts for Table service operations](http://msdn.microsoft.com/library/azure/dd894042.aspx).)

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try    {
        // Create table.
        $tableRestProxy->createTable("mytable");
    }
    catch(ServiceException $e){
        $code = $e->getCode();
        $error_message = $e->getMessage();
        // Handle exception based on error codes and messages.
        // Error codes and messages can be found here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
    }

For information about restrictions on table names, see [Understanding the Table service data model](http://msdn.microsoft.com/library/azure/dd179338.aspx).

## Add an entity to a table
To add an entity to a table, create a new **Entity** object and pass it to **TableRestProxy->insertEntity**. Note that when you create an entity, you must specify a `PartitionKey` and `RowKey`. These are the unique identifiers for an entity and are values that can be queried much faster than other entity properties. The system uses `PartitionKey` to automatically distribute the table’s entities over many storage nodes. Entities with the same `PartitionKey` are stored on the same node. (Operations on multiple entities stored on the same node perform better than on entities stored across different nodes.) The `RowKey` is the unique ID of an entity within a partition.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $entity = new Entity();
    $entity->setPartitionKey("tasksSeattle");
    $entity->setRowKey("1");
    $entity->addProperty("Description", null, "Take out the trash.");
    $entity->addProperty("DueDate",
                         EdmType::DATETIME,
                         new DateTime("2012-11-05T08:15:00-08:00"));
    $entity->addProperty("Location", EdmType::STRING, "Home");

    try{
        $tableRestProxy->insertEntity("mytable", $entity);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
    }

For information about Table properties and types, see [Understanding the Table service data model](http://msdn.microsoft.com/library/azure/dd179338.aspx).

The **TableRestProxy** class offers two alternative methods for inserting entities: **insertOrMergeEntity** and **insertOrReplaceEntity**. To use these methods, create a new **Entity** and pass it as a parameter to either method. Each method will insert the entity if it does not exist. If the entity already exists, **insertOrMergeEntity** updates property values if the properties already exist and adds new properties if they do not exist, while **insertOrReplaceEntity** completely replaces an existing entity. The following example shows how to use **insertOrMergeEntity**. If the entity with `PartitionKey` "tasksSeattle" and `RowKey` "1" does not already exist, it will be inserted. However, if it has previously been inserted (as shown in the example above), the `DueDate` property will be updated, and the `Status` property will be added. The `Description` and `Location` properties are also updated, but with values that effectively leave them unchanged. If these latter two properties were not added as shown in the example, but existed on the target entity, their existing values would remain unchanged.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    //Create new entity.
    $entity = new Entity();

    // PartitionKey and RowKey are required.
    $entity->setPartitionKey("tasksSeattle");
    $entity->setRowKey("1");

    // If entity exists, existing properties are updated with new values and
    // new properties are added. Missing properties are unchanged.
    $entity->addProperty("Description", null, "Take out the trash.");
    $entity->addProperty("DueDate", EdmType::DATETIME, new DateTime()); // Modified the DueDate field.
    $entity->addProperty("Location", EdmType::STRING, "Home");
    $entity->addProperty("Status", EdmType::STRING, "Complete"); // Added Status field.

    try    {
        // Calling insertOrReplaceEntity, instead of insertOrMergeEntity as shown,
        // would simply replace the entity with PartitionKey "tasksSeattle" and RowKey "1".
        $tableRestProxy->insertOrMergeEntity("mytable", $entity);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }


## Retrieve a single entity
The **TableRestProxy->getEntity** method allows you to retrieve a single entity by querying for its `PartitionKey` and `RowKey`. In the example below, the partition key `tasksSeattle` and row key `1` are passed to the **getEntity** method.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try    {
        $result = $tableRestProxy->getEntity("mytable", "tasksSeattle", 1);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

    $entity = $result->getEntity();

    echo $entity->getPartitionKey().":".$entity->getRowKey();

## Retrieve all entities in a partition
Entity queries are constructed using filters (for more information, see [Querying Tables and Entities](http://msdn.microsoft.com/library/azure/dd894031.aspx)). To retrieve all entities in partition, use the filter "PartitionKey eq *partition_name*". The following example shows how to retrieve all entities in the `tasksSeattle` partition by passing a filter to the **queryEntities** method.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $filter = "PartitionKey eq 'tasksSeattle'";

    try    {
        $result = $tableRestProxy->queryEntities("mytable", $filter);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

    $entities = $result->getEntities();

    foreach($entities as $entity){
        echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
    }

## Retrieve a subset of entities in a partition
The same pattern used in the previous example can be used to retrieve any subset of entities in a partition. The subset of entities you retrieve are determined by the filter you use (for more information, see [Querying tables and entities](http://msdn.microsoft.com/library/azure/dd894031.aspx)).The following example shows how to use a filter to retrieve all entities with a specific `Location` and a `DueDate` less than a specified date.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $filter = "Location eq 'Office' and DueDate lt '2012-11-5'";

    try    {
        $result = $tableRestProxy->queryEntities("mytable", $filter);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

    $entities = $result->getEntities();

    foreach($entities as $entity){
        echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
    }

## Retrieve a subset of entity properties
A query can retrieve a subset of entity properties. This technique, called *projection*, reduces bandwidth and can improve query performance, especially for large entities. To specify a property to be retrieved, pass the name of the property to the **Query->addSelectField** method. You can call this method multiple times to add more properties. After executing **TableRestProxy->queryEntities**, the returned entities will only have the selected properties. (If you want to return a subset of Table entities, use a filter as shown in the queries above.)

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\QueryEntitiesOptions;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $options = new QueryEntitiesOptions();
    $options->addSelectField("Description");

    try    {
        $result = $tableRestProxy->queryEntities("mytable", $options);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

    // All entities in the table are returned, regardless of whether
    // they have the Description field.
    // To limit the results returned, use a filter.
    $entities = $result->getEntities();

    foreach($entities as $entity){
        $description = $entity->getProperty("Description")->getValue();
        echo $description."<br />";
    }

## Update an entity
An existing entity can be updated by using the **Entity->setProperty** and **Entity->addProperty** methods on the entity, and then calling **TableRestProxy->updateEntity**. The following example retrieves an entity, modifies one property, removes another property, and adds a new property. Note that you can remove a property by setting its value to **null**.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $result = $tableRestProxy->getEntity("mytable", "tasksSeattle", 1);

    $entity = $result->getEntity();

    $entity->setPropertyValue("DueDate", new DateTime()); //Modified DueDate.

    $entity->setPropertyValue("Location", null); //Removed Location.

    $entity->addProperty("Status", EdmType::STRING, "In progress"); //Added Status.

    try    {
        $tableRestProxy->updateEntity("mytable", $entity);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Delete an entity
To delete an entity, pass the table name, and the entity's `PartitionKey` and `RowKey` to the **TableRestProxy->deleteEntity** method.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try    {
        // Delete entity.
        $tableRestProxy->deleteEntity("mytable", "tasksSeattle", "2");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

Note that for concurrency checks, you can set the Etag for an entity to be deleted by using the **DeleteEntityOptions->setEtag** method and passing the **DeleteEntityOptions** object to **deleteEntity** as a fourth parameter.

## Batch table operations
The **TableRestProxy->batch** method allows you to execute multiple operations in a single request. The pattern here involves adding operations to **BatchRequest** object and then passing the **BatchRequest** object to the **TableRestProxy->batch** method. To add an operation to a **BatchRequest** object, you can call any of the following methods multiple times:

* **addInsertEntity** (adds an insertEntity operation)
* **addUpdateEntity** (adds an updateEntity operation)
* **addMergeEntity** (adds a mergeEntity operation)
* **addInsertOrReplaceEntity** (adds an insertOrReplaceEntity operation)
* **addInsertOrMergeEntity** (adds an insertOrMergeEntity operation)
* **addDeleteEntity** (adds a deleteEntity operation)

The following example shows how to execute **insertEntity** and **deleteEntity** operations in a single request:

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;
    use WindowsAzure\Table\Models\BatchOperations;

     // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    // Create list of batch operation.
    $operations = new BatchOperations();

    $entity1 = new Entity();
    $entity1->setPartitionKey("tasksSeattle");
    $entity1->setRowKey("2");
    $entity1->addProperty("Description", null, "Clean roof gutters.");
    $entity1->addProperty("DueDate",
                          EdmType::DATETIME,
                          new DateTime("2012-11-05T08:15:00-08:00"));
    $entity1->addProperty("Location", EdmType::STRING, "Home");

    // Add operation to list of batch operations.
    $operations->addInsertEntity("mytable", $entity1);

    // Add operation to list of batch operations.
    $operations->addDeleteEntity("mytable", "tasksSeattle", "1");

    try    {
        $tableRestProxy->batch($operations);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

For more information about batching Table operations, see [Performing entity group transactions](http://msdn.microsoft.com/library/azure/dd894038.aspx).

## Delete a table
Finally, to delete a table, pass the table name to the **TableRestProxy->deleteTable** method.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create table REST proxy.
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try    {
        // Delete table.
        $tableRestProxy->deleteTable("mytable");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179438.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Next steps
Now that you’ve learned the basics of the Azure Table service, follow these links to learn about more complex storage tasks.

* Visit the [Azure Storage team blog](http://blogs.msdn.com/b/windowsazurestorage/)

For more information, see also the [PHP Developer Center](/develop/php/).

[download]: http://go.microsoft.com/fwlink/?LinkID=252473
[require_once]: http://php.net/require_once
[table-service-timeouts]: http://msdn.microsoft.com/library/azure/dd894042.aspx

[table-data-model]: http://msdn.microsoft.com/library/azure/dd179338.aspx
[filters]: http://msdn.microsoft.com/library/azure/dd894031.aspx
[entity-group-transactions]: http://msdn.microsoft.com/library/azure/dd894038.aspx
