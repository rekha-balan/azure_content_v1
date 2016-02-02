<properties
    pageTitle="Get started with Azure Table storage using .NET | Microsoft Azure"
    description="Store unstructured data in the cloud using Azure Table storage, Microsoft's NoSQL data store. Get started with simple Table storage operations, including creating and deleting tables and inserting, updating, deleting, and querying data."
    services="storage"
    documentationCenter=".net"
    authors="tamram"
    manager="carmonm"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="01/24/2016"
    ms.author="tamram"/>


# Get started with Azure Table storage using .NET
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-tables.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-table-storage.md)
- [Java](../articles/storage/storage-java-how-to-use-table-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-tables.md)
- [PHP](../articles/storage/storage-php-how-to-use-table-storage.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-table-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-table-storage.md)

## Overview
Azure Table storage is a service that stores unstructured NoSQL data in the cloud. Table storage is a key/attribute store with a schemaless design. Because Table storage is schemaless, it's easy to adapt your data as the needs of your application evolve. Access to data is fast and cost-effective for all kinds of applications. Table storage is typically significantly lower in cost than traditional SQL for similar volumes of data. 

You can use Table storage to store flexible datasets, such as user data for web applications, address books, device information, and any other type of metadata that your service requires. You can store any number of entities in a table, and a storage account may contain any number of tables, up to the capacity limit of the storage account.

This tutorial shows how to write .NET code for some common scenarios using Azure Table storage, including creating and deleting a table and inserting, updating, deleting, and querying table data.

>[AZURE.NOTE] We recommend that you use the latest version of the Azure Storage Client Library for .NET to complete this tutorial. The latest version of the library is 6.x, available for download on [Nuget](https://www.nuget.org/packages/WindowsAzure.Storage/). The source for the client library is available on [GitHub](https://github.com/Azure/azure-storage-net). 

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
 

## Setup a storage connection string

The Azure Storage Client Library for .NET supports using a storage connection string to configure endpoints and credentials for accessing storage services. We recommend that you maintain your storage connection string in a configuration file, rather than hard-coding it into your application. You have two options for saving your connection string:

- If your application runs in an Azure cloud service, save your connection string using the Azure service configuration system (`*.csdef` and `*.cscfg` files). See [How to create and deploy a cloud service](../articles/cloud-services/cloud-services-how-to-create-deploy.md) for details about Azure cloud service configuration.
- If your application runs on Azure virtual machines, or if you are building .NET applications that will run outside of Azure, save your connection string using the .NET configuration system (e.g. `web.config` or `app.config` file).

Later on in this guide, we will show how to retrieve your connection string from your code.

### Configuring your connection string from an Azure cloud service

Azure Cloud Services has a unique service configuration mechanism that enables you to dynamically change configuration settings from the Azure Management Portal without redeploying your application.

To configure your connection string in the Azure service configuration:

1.  Within the Solution Explorer of Visual Studio, in the **Roles**
    folder of your Azure Deployment Project, right-click your
    web role or worker role and click **Properties**.  
    ![Select the properties on a Cloud Service role in Visual Studio][connection-string1]

2.  Click the **Settings** tab and press the **Add Setting** button.  
    ![Add a Cloud Service setting in visual Studio][connection-string2]

    A new **Setting1** entry will then show up in the settings grid.

3.  In the **Type** drop-down of the new **Setting1** entry, choose
    **Connection String**.  
    ![Set connection string type][connection-string3]

4.  Click the **...** button at the right end of the **Setting1** entry.
    The **Storage Account Connection String** dialog will open.

5.  Choose whether you want to target the storage emulator (Microsoft
    Azure storage simulated on your local machine) or a storage
    account in the cloud. The code in this guide works with either
    option. 

	> [AZURE.NOTE] You can target the storage emulator to avoid incurring any costs associated with Azure Storage. However, if you do choose to target an Azure storage account in the cloud, costs for performing this tutorial will be negligible.

	If you are targeting a storage account in the cloud, then enter the primary access key for that storage account. To learn how to copy your primary access key via the Azure Management Portal, see [View, copy, and regenerate storage access keys](storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys).

	> [AZURE.NOTE] Your storage account key is similar to the root password for your storage account. Be sure to protect your key. Avoid distributing it to other users or saving it in a plain-text file that is accessible to others. Regenerate your key using the Management Portal if you believe it may have been compromised.
	
    ![Select target environment][connection-string4]

6.  Change the entry **Name** from **Setting1** to a friendlier name
    like **StorageConnectionString**. You will reference this
    connection string later in the code in this guide.  
    ![Change connection string name][connection-string5]
	
### Configuring your connection string using .NET configuration

If you are writing an application that is not an Azure cloud service, (see previous section), it is recommended you use the .NET configuration system (e.g. `web.config` or `app.config`). This includes Azure Websites or Azure Virtual Machines, as well as applications designed to run outside of Azure. You store the connection string using the `<appSettings>` element as follows. Replace `account-name` with the name of your storage account, and `account-key` with your account access key:

	<configuration>
  		<appSettings>
    		<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key" />
  		</appSettings>
	</configuration>

For example, the configuration setting in your config file may be similar to:

	<configuration>
    	<appSettings>
      		<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=nYV0gln9fT7bvY+rxu2iWAEyzPNITGkhM88J8HUoyofpK7C8fHcZc2kIZp6cKgYRUM74lHI84L50Iau1+9hPjB==" />
    	</appSettings>
	</configuration>

You are now ready to perform the how-to tasks in this guide.

[connection-string1]: ./media/storage-configure-connection-string-include/connection-string1.png
[connection-string2]: ./media/storage-configure-connection-string-include/connection-string2.png
[connection-string3]: ./media/storage-configure-connection-string-include/connection-string3.png
[connection-string4]: ./media/storage-configure-connection-string-include/connection-string4.png
[connection-string5]: ./media/storage-configure-connection-string-include/connection-string5.png

[Configuring Connection Strings]: http://msdn.microsoft.com/library/azure/ee758697.aspx

## Programmatically access Table storage
### Obtaining the assembly

You can use NuGet to obtain the `Microsoft.WindowsAzure.Storage.dll` assembly. Right-click your project in **Solution Explorer** and choose **Manage NuGet Packages**.  Search online for "WindowsAzure.Storage" and click **Install** to install the Azure Storage package and dependencies.

`Microsoft.WindowsAzure.Storage.dll` is also included in the Azure SDK for .NET, which can be downloaded from the <a href="http://azure.microsoft.com/develop/net/#">.NET Developer Center</a>. The assembly is installed to the `%Program Files%\Microsoft SDKs\Azure\.NET SDK\<sdk-version>\ref\` directory.


### Namespace declarations
Add the following code namespace declarations to the top of any C\# file
in which you wish to programmatically access Azure Storage.

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Table;

Make sure you reference the `Microsoft.WindowsAzure.Storage.dll` assembly.

### Retrieving your connection string
You can use the **CloudStorageAccount** type to represent 
your Storage Account information. If you are using an 
Azure project template and/or have a reference to the
Microsoft.WindowsAzure.CloudConfigurationManager namespace, you 
can use the **CloudConfigurationManager** type
to retrieve your storage connection string and storage account
information from the Azure service configuration:

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

If you are creating an application with no reference to Microsoft.WindowsAzure.CloudConfigurationManager, and your connection string is located in the `web.config` or `app.config` as show above, then you can use **ConfigurationManager** to retrieve the connection string.  You will need to add a reference to System.Configuration.dll to your project and add another namespace declaration for it:

	using System.Configuration;
	...
	CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);


## Create a table
A **CloudTableClient** object lets you get reference objects for tables
and entities. The following code creates a **CloudTableClient** object
and uses it to create a new table. All code in this article assumes that
the application being built is an Azure Cloud Services project and
uses a storage connection string stored in the Azure application's service configuration.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the table if it doesn't exist.
    CloudTable table = tableClient.GetTableReference("people");
    table.CreateIfNotExists();

## Add an entity to a table
Entities map to C\# objects by using a custom class derived from
**TableEntity**. To add an entity to a table, create a
class that defines the properties of your entity. The following code
defines an entity class that uses the customer's first name as the row
key and last name as the partition key. Together, an entity's partition
and row key uniquely identify the entity in the table. Entities with the
same partition key can be queried faster than those with different
partition keys, but using diverse partition keys allows for greater scalability of parallel operations.  For any property that should be stored in the Table service,
the property must be a public property of a supported type that exposes both `get` and `set`.
Also, your entity type *must* expose a parameter-less constructor.

    public class CustomerEntity : TableEntity
    {
        public CustomerEntity(string lastName, string firstName)
        {
            this.PartitionKey = lastName;
            this.RowKey = firstName;
        }

        public CustomerEntity() { }

        public string Email { get; set; }

        public string PhoneNumber { get; set; }
    }

Table operations that involve entities are performed via the **CloudTable**
object that you created earlier in the "Create a table" section. The operation to be performed
is represented by a **TableOperation** object.  The following code example shows the creation of the **CloudTable** object and then a **CustomerEntity** object.  To prepare the operation, a **TableOperation** object is created to insert the customer entity into the table.  Finally, the operation is executed by calling **CloudTable.Execute**.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    customer1.PhoneNumber = "425-555-0101";

    // Create the TableOperation object that inserts the customer entity.
    TableOperation insertOperation = TableOperation.Insert(customer1);

    // Execute the insert operation.
    table.Execute(insertOperation);

## Insert a batch of entities
You can insert a batch of entities into a table in one write
operation. Some other notes on batch
operations:

* You can perform updates, deletes, and inserts in the same single batch operation.
* A single batch operation can include up to 100 entities.
* All entities in a single batch operation must have the same
 partition key.
* While it is possible to perform a query as a batch operation, it must be the only operation in the batch.

<!-- -->
The following code example creates two entity objects and adds each
to **TableBatchOperation** by using the **Insert** method. Then, **CloudTable.Execute** is called to execute the operation.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create the batch operation.
    TableBatchOperation batchOperation = new TableBatchOperation();

    // Create a customer entity and add it to the table.
    CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
    customer1.Email = "Jeff@contoso.com";
    customer1.PhoneNumber = "425-555-0104";

    // Create another customer entity and add it to the table.
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.Email = "Ben@contoso.com";
    customer2.PhoneNumber = "425-555-0102";

    // Add both customer entities to the batch insert operation.
    batchOperation.Insert(customer1);
    batchOperation.Insert(customer2);

    // Execute the batch operation.
    table.ExecuteBatch(batchOperation);

## Retrieve all entities in a partition
To query a table for all entities in a partition, use a **TableQuery** object.
The following code example specifies a filter for entities where 'Smith'
is the partition key. This example prints the fields of
each entity in the query results to the console.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Construct the query operation for all customer entities where PartitionKey="Smith".
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

    // Print the fields for each customer.
    foreach (CustomerEntity entity in table.ExecuteQuery(query))
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
            entity.Email, entity.PhoneNumber);
    }

## Retrieve a range of entities in a partition
If you don't want to query all the entities in a partition, you can
specify a range by combining the partition key filter with a row key filter. The following code example
uses two filters to get all entities in partition 'Smith' where the row
key (first name) starts with a letter earlier than 'E' in the alphabet and then
prints the query results.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create the table query.
    TableQuery<CustomerEntity> rangeQuery = new TableQuery<CustomerEntity>().Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"),
            TableOperators.And,
            TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "E")));

    // Loop through the results, displaying information about the entity.
    foreach (CustomerEntity entity in table.ExecuteQuery(rangeQuery))
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
            entity.Email, entity.PhoneNumber);
    }

## Retrieve a single entity
You can write a query to retrieve a single, specific entity. The
following code uses **TableOperation** to specify the customer 'Ben Smith'.
This method returns just one entity rather than a
collection, and the returned value in **TableResult.Result** is a **CustomerEntity** object.
Specifying both partition and row keys in a query is the fastest way to
retrieve a single entity from the Table service.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the retrieve operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Print the phone number of the result.
    if (retrievedResult.Result != null)
       Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
    else
       Console.WriteLine("The phone number could not be retrieved.");

## Replace an entity
To update an entity, retrieve it from the Table service, modify the
entity object, and then save the changes back to the Table service. The
following code changes an existing customer's phone number. Instead of
calling **Insert**, this code uses
**Replace**. This causes the entity to be fully replaced on the server,
unless the entity on the server has changed since it was retrieved, in
which case the operation will fail.  This failure is to prevent your application
from inadvertently overwriting a change made between the retrieval and
update by another component of your application.  The proper handling of this failure
is to retrieve the entity again, make your changes (if still valid), and then
perform another **Replace** operation.  The next section will
show you how to override this behavior.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity object.
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
    {
       // Change the phone number.
       updateEntity.PhoneNumber = "425-555-0105";

       // Create the InsertOrReplace TableOperation.
       TableOperation updateOperation = TableOperation.Replace(updateEntity);

       // Execute the operation.
       table.Execute(updateOperation);

       Console.WriteLine("Entity updated.");
    }

    else
       Console.WriteLine("Entity could not be retrieved.");

## Insert-or-replace an entity
**Replace** operations will fail if the entity has been changed since
it was retrieved from the server.  Furthermore, you must retrieve
the entity from the server first in order for the **Replace** operation to be successful.
Sometimes, however, you don't know if the entity exists on the server
and the current values stored in it are irrelevant. Your update should
overwrite them all.  To accomplish this, you would use an **InsertOrReplace**
operation.  This operation inserts the entity if it doesn't exist, or
replaces it if it does, regardless of when the last update was made.  In the
following code example, the customer entity for Ben Smith is still retrieved, but it is then saved back to the server via **InsertOrReplace**.  Any updates
made to the entity between the retrieval and update operations will be
overwritten.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity object.
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
    {
       // Change the phone number.
       updateEntity.PhoneNumber = "425-555-1234";

       // Create the InsertOrReplace TableOperation.
       TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(updateEntity);

       // Execute the operation.
       table.Execute(insertOrReplaceOperation);

       Console.WriteLine("Entity was updated.");
    }

    else
       Console.WriteLine("Entity could not be retrieved.");

## Query a subset of entity properties
A table query can retrieve just a few properties from an entity instead of all the entity properties. This technique, called projection, reduces bandwidth and can improve query performance, especially for large entities. The query in the
following code returns only the email addresses of entities in the
table. This is done by using a query of **DynamicTableEntity** and
also **EntityResolver**. You can learn more about projection on the [Introducing Upsert and Query Projection blog post](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx). Note that projection is not supported on the local storage emulator, so this code runs only when you're using an account on the Table service.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Define the query, and select only the Email property.
    TableQuery<DynamicTableEntity> projectionQuery = new TableQuery<DynamicTableEntity>().Select(new string[] { "Email" });

    // Define an entity resolver to work with the entity after retrieval.
    EntityResolver<string> resolver = (pk, rk, ts, props, etag) => props.ContainsKey("Email") ? props["Email"].StringValue : null;

    foreach (string projectedEmail in table.ExecuteQuery(projectionQuery, resolver, null, null))
    {
        Console.WriteLine(projectedEmail);
    }

## Delete an entity
You can easily delete an entity after you have retrieved it, by using the same pattern
shown for updating an entity.  The following code
retrieves and deletes a customer entity.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that expects a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity.
    CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

    // Create the Delete TableOperation.
    if (deleteEntity != null)
    {
       TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

       // Execute the operation.
       table.Execute(deleteOperation);

       Console.WriteLine("Entity deleted.");
    }

    else
       Console.WriteLine("Could not retrieve the entity.");

## Delete a table
Finally, the following code example deletes a table from a storage account. A
table that has been deleted will be unavailable to be re-created for a
period of time following the deletion.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Delete the table it if exists.
    table.DeleteIfExists();

## Retrieve entities in pages asynchronously
If you are reading a large number of entities, and you want to process/display entities as they are retrieved rather than waiting for them all to return, you can retrieve entities by using a segmented query. This example shows how to return results in pages by using the Async-Await pattern so that execution is not blocked while you're waiting for a large set of results to return. For more details on using the Async-Await pattern in .NET, see [Asynchronous programming with Async and Await (C# and Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx).

    // Initialize a default TableQuery to retrieve all the entities in the table.
    TableQuery<CustomerEntity> tableQuery = new TableQuery<CustomerEntity>();

    // Initialize the continuation token to null to start from the beginning of the table.
    TableContinuationToken continuationToken = null;

    do
    {
        // Retrieve a segment (up to 1,000 entities).
        TableQuerySegment<CustomerEntity> tableQueryResult =
            await table.ExecuteQuerySegmentedAsync(tableQuery, continuationToken);

        // Assign the new continuation token to tell the service where to
        // continue on the next iteration (or null if it has reached the end).
        continuationToken = tableQueryResult.ContinuationToken;

        // Print the number of rows retrieved.
        Console.WriteLine("Rows retrieved {0}", tableQueryResult.Results.Count);

    // Loop until a null continuation token is received, indicating the end of the table.
    } while(continuationToken != null);

## Next steps
Now that you've learned the basics of Table storage, follow these links
to learn about more complex storage tasks:

* View the Table service reference documentation for complete details about available APIs:  * [Storage Client Library for .NET reference](http://go.microsoft.com/fwlink/?LinkID=390731clcid=0x409)
* [REST API reference](http://msdn.microsoft.com/library/azure/dd179355)


* Learn how to simplify the code you write to work with Azure Storage by using the [Azure WebJobs SDK](../app-service-web/websites-dotnet-webjobs-sdk-get-started.md)
* View more feature guides to learn about additional options for storing data in Azure.  * Use [Blob Storage](storage-dotnet-how-to-use-blobs.md) to store unstructured data.
* Use [SQL Database](sql-database-dotnet-how-to-use.md) to store relational data.



  [Download and install the Azure SDK for .NET]: /develop/net/
  [Creating an Azure Project in Visual Studio]: http://msdn.microsoft.com/library/azure/ee405487.aspx

  [Blob5]: ./media/storage-dotnet-how-to-use-table-storage/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-table-storage/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-table-storage/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-table-storage/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-table-storage/blob9.png

  [Introducing Upsert and Query Projection blog post]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx
  [.NET Client Library reference]: http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
  [Azure Storage Team blog]: http://blogs.msdn.com/b/windowsazurestorage/
  [Configure Azure Storage connection strings]: http://msdn.microsoft.com/library/azure/ee758697.aspx
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
  [How to: Programmatically access Table storage]: #tablestorage
