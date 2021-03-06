<properties
    pageTitle="Get started with queue storage and Visual Studio connected services (cloud services) | Microsoft Azure"
    description="How to get started using Azure Queue storage in a cloud service project in Visual Studio after connecting to a storage account using Visual Studio connected services"
    services="storage"
    documentationCenter=""
    authors="TomArcher"
    manager="douge"
    editor=""/>

<tags
    ms.service="storage"
    ms.workload="web"
    ms.tgt_pltfrm="vs-getting-started" 
    ms.devlang="na"
    ms.topic="article"
  ms.date="12/16/2015"
    ms.author="tarcher"/>

# Getting started with Azure Queue storage and Visual Studio connected services (cloud services projects)
## Overview
This article describes how to get started using Azure Queue storage in Visual Studio after you have created or referenced an Azure storage account in a cloud services project by using the  Visual Studio **Add Connected Services** dialog.

We'll show you how to create a queue in code. We'll also show you how to perform basic queue operations, such as adding, modifying, reading and removing queue messages. The samples are written in C# code and use the [Azure Storage Client Library for .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx).

The **Add Connected Services** operation installs the appropriate NuGet packages to access Azure storage in your project and adds the connection string for the storage account to your project configuration files.

* See [How to use Queue Storage from .NET](storage-dotnet-how-to-use-queues.md) for more information on manipulating queues in code.
* See [Storage documentation](https://azure.microsoft.com/documentation/services/storage/) for general information about Azure Storage.
* See [Cloud Services documentation](https://azure.microsoft.com/documentation/services/cloud-services/) for general information about Azure cloud services.
* See [ASP.NET](http://www.asp.net) for more information about programming ASP.NET applications.

Azure Queue storage is a service for storing large numbers of messages that can be accessed from anywhere in the world via authenticated calls using HTTP or HTTPS. A single queue message can be up to 64 KB in size, and a queue can contain millions of messages, up to the total capacity limit of a storage account.

## Access queues in code
To access queues in Visual Studio Cloud Services projects, you need to include the following items to any C# source file that access Azure Queue storage.

1. Make sure the namespace declarations at the top of the C# file include these **using** statements.

        using Microsoft.Framework.Configuration;
     using Microsoft.WindowsAzure.Storage;
     using Microsoft.WindowsAzure.Storage.Queue;
2. Get a **CloudStorageAccount** object that represents your storage account information. Use the following code to get the your storage connection string and storage account information from the Azure service configuration.

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
3. Get a **CloudQueueClient** object to reference the queue objects in your storage account.  

        // Create the queue client.
     CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
4. Get a **CloudQueue** object to reference a specific queue.

        // Get a reference to a queue named "messageQueue"
     CloudQueue messageQueue = queueClient.GetQueueReference("messageQueue");



**NOTE:** Use all of the above code in front of the code in the following samples.

## Create a queue in code
To create the queue in code, just add a call to **CreateIfNotExists**.

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    // Create the CloudQueue if it does not exist
    messageQueue.CreateIfNotExists();

## Add a message to a queue
To insert a message into an existing queue, create a new **CloudQueueMessage** object, then call the **AddMessage** method.

A **CloudQueueMessage** object can be created from either a string (in UTF-8 format) or a byte array.

Here is an example which inserts the message 'Hello, World'.

    // Get a reference to a CloudQueue object with the variable name 'messageQueue' as described in
    // the "Access queues in code" section.

    // Create a message and add it to the queue.
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    messageQueue.AddMessage(message);

## Read a message in a queue
You can peek at the message in the front of a queue without removing it from the queue by calling the **PeekMessage** method.

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    // Peek at the next message
    CloudQueueMessage peekedMessage = messageQueue.PeekMessage();

## Read and remove a message in a queue
Your code can remove (de-queue) a message from a queue in two steps.

1. Call **GetMessage** to get the next message in a queue. A message returned from **GetMessage** becomes invisible to any other code reading messages from this queue. By default, this message stays invisible for 30 seconds.
2. To finish removing the message from the queue, call **DeleteMessage**.

This two-step process of removing a message assures that if your code fails to process a message due to hardware or software failure, another instance of your code can get the same message and try again. The following code calls **DeleteMessage** right after the message has been processed.

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    // Get the next message in the queue.
    CloudQueueMessage retrievedMessage = messageQueue.GetMessage();

    // Process the message in less than 30 seconds

      // Then delete the message.
    await messageQueue.DeleteMessage(retrievedMessage);


## Use additional options to process and remove queue messages
There are two ways you can customize message retrieval from a queue.

* You can get a batch of messages (up to 32).
* You can set a longer or shorter invisibility timeout, allowing your code more or less
time to fully process each message. The following code example uses the
**GetMessages** method to get 20 messages in one call. Then it processes
each message using a **foreach** loop. It also sets the invisibility
timeout to five minutes for each message. Note that the 5 minutes starts
for all messages at the same time, so after 5 minutes have passed since
the call to **GetMessages**, any messages which have not been deleted
will become visible again.

Here's an example:

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    foreach (CloudQueueMessage message in messageQueue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Process all messages in less than 5 minutes, deleting each message after processing.

        // Then delete the message after processing
        messageQueue.DeleteMessage(message);

    }

## Get the queue length
You can get an estimate of the number of messages in a queue. The
**FetchAttributes** method asks the Queue service to
retrieve the queue attributes, including the message count. The **ApproximateMethodCount**
property returns the last value retrieved by the
**FetchAttributes** method, without calling the Queue service.

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    // Fetch the queue attributes.
    messageQueue.FetchAttributes();

    // Retrieve the cached approximate message count.
    int? cachedMessageCount = messageQueue.ApproximateMessageCount;

    // Display number of messages.
    Console.WriteLine("Number of messages in queue: " + cachedMessageCount);

## Use the Async-Await Pattern with common Azure Queue APIs
This example shows how to use the Async-Await pattern with common Azure Queue APIs. The sample calls the async version of each of the given methods, this can be seen by the **Async** post-fix of each method. When an async method is used the async-await pattern suspends local execution until the call completes. This behavior allows the current thread to do other work which helps avoid performance bottlenecks and improves the overall responsiveness of your application. For more details on using the Async-Await pattern in .NET see [Async and Await (C# and Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx)

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    // Create a message to put in the queue
    CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

    // Add the message asynchronously
    await messageQueue.AddMessageAsync(cloudQueueMessage);
    Console.WriteLine("Message added");

    // Async dequeue the message
    CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();
    Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

    // Delete the message asynchronously
    await messageQueue.DeleteMessageAsync(retrievedMessage);
    Console.WriteLine("Deleted message");

## Delete a queue
To delete a queue and all the messages contained in it, call the **Delete** method on the queue object.

    // Get a reference to a CloudQueue object with the variable name 'messageQueue'
    // as described in the "Access queues in code" section.

    // Delete the queue.
    messageQueue.Delete();

## Next steps

Now that you've learned the basics of Azure queue storage, follow these links to learn about more complex storage tasks.

- View the Queue service reference documentation in the [Storage Client Library for .NET](http://go.microsoft.com/fwlink/?LinkID=390731) reference for complete details about available APIs.
- Learn more about using Queue storage at [How to use Queue storage from .NET](storage-dotnet-how-to-use-queues.md)
- Learn about more advanced tasks you can perform with Azure Storage at [Storing and Accessing Data in Azure](https://msdn.microsoft.com/library/azure/gg433040.aspx).    
- Learn how to simplify the code you write to work with Azure Storage by using the [Azure WebJobs SDK](../app-service/websites-dotnet-webjobs-sdk.md)
- View more feature guides to learn about additional options for storing data in Azure.
  - Use [Blob Storage](./storage-dotnet-how-to-use-blobs.md) to store structured data.
  - Use [Table Storage](./storage-dotnet-how-to-use-tables.md) to store structured data.
  - Use [SQL Database](../sql-database/sql-database-dotnet-how-to-use.md) to store relational data.



