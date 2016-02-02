<properties
    pageTitle="Get started with Azure Queue storage using .NET | Microsoft Azure"
    description="Send and receive messages asynchronously between application components using Azure Queue storage. Get started with simple Queue storage operations, including creating and deleting queues and adding, reading, and deleting queue messages."
    services="storage"
    documentationCenter=".net"
    authors="robinsh"
    manager="carmonm"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="01/24/2016"
    ms.author="gusapost"/>

# Get started with Azure Queue storage using .NET
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-queues.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-queues.md)
- [Java](../articles/storage/storage-java-how-to-use-queue-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-queues.md)
- [PHP](../articles/storage/storage-php-how-to-use-queues.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-queue-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-queue-storage.md)

## Overview
Azure Queue storage is a service that provides messaging queues in the cloud. In designing applications for scale, application components are often decoupled, so that they can scale independently.  Queue storage provides a reliable messaging solution for asynchronous communication between application components, whether they are running in the cloud, on the desktop, on an on-premises server, or on a mobile device. Queue storage also supports managing asynchronous tasks and building process work flows.

This tutorial shows how to write .NET code for some common scenarios using Azure Queue storage. Scenarios covered include creating and deleting queues and adding, reading, and deleting queue messages.

>[AZURE.NOTE] We recommend that you use the latest version of the Azure Storage Client Library for .NET to complete this tutorial. The latest version of the library is 6.x, available for download on [Nuget](https://www.nuget.org/packages/WindowsAzure.Storage/). The source for the client library is available on [GitHub](https://github.com/Azure/azure-storage-net). 

## What is Queue Storage?

Azure Queue storage is a service for storing large numbers of
messages that can be accessed from anywhere in the world via
authenticated calls using HTTP or HTTPS. A single queue message can be
up to 64 KB in size, and a queue can contain millions of messages, up to the
total capacity limit of a storage account. A storage account can contain up to 500 TB of blob, queue, and table data. See [Azure Storage Scalability and Performance Targets](http://msdn.microsoft.com/library/azure/dn249410.aspx) for details about storage account capacity.

Common uses of Queue storage include:

-   Creating a backlog of work to process asynchronously
-   Passing messages from an Azure Web role to an Azure worker role

## Queue Service Concepts

The Queue service contains the following components:

![Queue1](./media/storage-queue-concepts-include/queue1.png)


- **URL format:** Queues are addressable using the following URL format:   
	http://`<storage account>`.queue.core.windows.net/`<queue>` 
      
	The following URL addresses a queue in the diagram:  
		
		http://myaccount.queue.core.windows.net/imagesToDownload

- **Storage Account:** All access to Azure Storage is done through a storage account. See [Azure Storage Scalability and Performance Targets](../articles/storage/storage-scalability-targets.md) for details about storage account capacity.

- **Queue:** A queue contains a set of messages. All messages must be in a queue.

- **Message:** A message, in any format, of up to 64KB.


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

## Programmatically access Queue storage
### Obtaining the assembly

You can use NuGet to obtain the `Microsoft.WindowsAzure.Storage.dll` assembly. Right-click your project in **Solution Explorer** and choose **Manage NuGet Packages**.  Search online for "WindowsAzure.Storage" and click **Install** to install the Azure Storage package and dependencies.

`Microsoft.WindowsAzure.Storage.dll` is also included in the Azure SDK for .NET, which can be downloaded from the <a href="http://azure.microsoft.com/develop/net/#">.NET Developer Center</a>. The assembly is installed to the `%Program Files%\Microsoft SDKs\Azure\.NET SDK\<sdk-version>\ref\` directory.


### Namespace declarations
Add the following code namespace declarations to the top of any C\# file
in which you wish to programmatically access Azure Storage:

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Queue;

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


## Create a queue
A **CloudQueueClient** object lets you get reference objects for queues.
The following code creates a **CloudQueueClient** object. All code in
this guide uses a storage connection string stored in the Azure
application's service configuration. There are also other ways to create
a **CloudStorageAccount** object. See [CloudStorageAccount](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudstorageaccount_methods.aspx)
documentation for details.

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

Use the **queueClient** object to get a reference to the queue you want
to use. You can create the queue if it doesn't exist.

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExists();

## Insert a message into a queue
To insert a message into an existing queue, first create a new
**CloudQueueMessage**. Next, call the **AddMessage** method. A
**CloudQueueMessage** can be created from either a string (in UTF-8
format) or a **byte** array. Here is code which creates a queue (if it
doesn't exist) and inserts the message 'Hello, World':

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist.
    queue.CreateIfNotExists();

    // Create a message and add it to the queue.
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.AddMessage(message);

## Peek at the next message
You can peek at the message in the front of a queue without removing it
from the queue by calling the **PeekMessage** method.

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Peek at the next message
    CloudQueueMessage peekedMessage = queue.PeekMessage();

    // Display message.
    Console.WriteLine(peekedMessage.AsString);

## Change the contents of a queued message
You can change the contents of a message in-place in the queue. If the
message represents a work task, you could use this feature to update the
status of the work task. The following code updates the queue message
with new contents, and sets the visibility timeout to extend another 60
seconds. This saves the state of work associated with the message, and
gives the client another minute to continue working on the message. You
could use this technique to track multi-step workflows on queue
messages, without having to start over from the beginning if a
processing step fails due to hardware or software failure. Typically,
you would keep a retry count as well, and if the message is retried more
than *n* times, you would delete it. This protects against a message
that triggers an application error each time it is processed.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Get the message from the queue and update the message contents.
    CloudQueueMessage message = queue.GetMessage();
    message.SetMessageContent("Updated contents.");
    queue.UpdateMessage(message,
        TimeSpan.FromSeconds(60.0),  // Make it visible for another 60 seconds.
        MessageUpdateFields.Content | MessageUpdateFields.Visibility);

## De-queue the next message
Your code de-queues a message from a queue in two steps. When you call
**GetMessage**, you get the next message in a queue. A message returned
from **GetMessage** becomes invisible to any other code reading messages
from this queue. By default, this message stays invisible for 30
seconds. To finish removing the message from the queue, you must also
call **DeleteMessage**. This two-step process of removing a message
assures that if your code fails to process a message due to hardware or
software failure, another instance of your code can get the same message
and try again. Your code calls **DeleteMessage** right after the message
has been processed.

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Get the next message
    CloudQueueMessage retrievedMessage = queue.GetMessage();

    //Process the message in less than 30 seconds, and then delete the message
    queue.DeleteMessage(retrievedMessage);

## Use Async-Await pattern with common Queue storage APIs
This example shows how to use the Async-Await pattern with common Queue storage APIs. The sample calls the asynchronous version of each of the given methods, as indicated by the *Async* suffix of each method. When an async method is used, the async-await pattern suspends local execution until the call completes. This behavior allows the current thread to do other work, which helps avoid performance bottlenecks and improves the overall responsiveness of your application. For more details on using the Async-Await pattern in .NET see [Async and Await (C# and Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx)

    // Create the queue if it doesn't already exist
    if(await queue.CreateIfNotExistsAsync())
    {
        Console.WriteLine("Queue '{0}' Created", queue.Name);
    }
    else
    {
        Console.WriteLine("Queue '{0}' Exists", queue.Name);
    }

    // Create a message to put in the queue
    CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

    // Async enqueue the message
    await queue.AddMessageAsync(cloudQueueMessage);
    Console.WriteLine("Message added");

    // Async dequeue the message
    CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();
    Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

    // Async delete the message
    await queue.DeleteMessageAsync(retrievedMessage);
    Console.WriteLine("Deleted message");

## Leverage additional options for de-queuing messages
There are two ways you can customize message retrieval from a queue.
First, you can get a batch of messages (up to 32). Second, you can set a
longer or shorter invisibility timeout, allowing your code more or less
time to fully process each message. The following code example uses the
**GetMessages** method to get 20 messages in one call. Then it processes
each message using a **foreach** loop. It also sets the invisibility
timeout to five minutes for each message. Note that the 5 minutes starts
for all messages at the same time, so after 5 minutes have passed since
the call to **GetMessages**, any messages which have not been deleted
will become visible again.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Process all messages in less than 5 minutes, deleting each message after processing.
        queue.DeleteMessage(message);
    }

## Get the queue length
You can get an estimate of the number of messages in a queue. The
**FetchAttributes** method asks the Queue service to
retrieve the queue attributes, including the message count. The **ApproximateMessageCount**
property returns the last value retrieved by the
**FetchAttributes** method, without calling the Queue service.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Fetch the queue attributes.
    queue.FetchAttributes();

    // Retrieve the cached approximate message count.
    int? cachedMessageCount = queue.ApproximateMessageCount;

    // Display number of messages.
    Console.WriteLine("Number of messages in queue: " + cachedMessageCount);

## Delete a queue
To delete a queue and all the messages contained in it, call the
**Delete** method on the queue object.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Delete the queue.
    queue.Delete();

## Next steps
Now that you've learned the basics of Queue storage, follow these links
to learn about more complex storage tasks.

* View the Queue service reference documentation for complete details about available APIs:  * [Storage Client Library for .NET reference](http://go.microsoft.com/fwlink/?LinkID=390731clcid=0x409)
* [REST API reference](http://msdn.microsoft.com/library/azure/dd179355)


* Learn how to simplify the code you write to work with Azure Storage by using the [Azure WebJobs SDK](../websites-dotnet-webjobs-sdk/.md).
* View more feature guides to learn about additional options for storing data in Azure.  * Use [Table Storage](storage-dotnet-how-to-use-tables.md) to store structured data.
* Use [Blob Storage](storage-dotnet-how-to-use-blobs.md) to store unstructured data.
* Use [SQL Database](sql-database-dotnet-how-to-use.md) to store relational data.



  [Download and install the Azure SDK for .NET]: /develop/net/
  [.NET client library reference]: http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
  [Creating a Azure Project in Visual Studio]: http://msdn.microsoft.com/library/azure/ee405487.aspx
  [CloudStorageAccount]: http://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudstorageaccount_methods.aspx
  [Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
  [Configuring Connection Strings]: http://msdn.microsoft.com/library/azure/ee758697.aspx
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
