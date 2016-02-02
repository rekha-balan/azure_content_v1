<properties 
    pageTitle="How to use Queue storage from Python | Microsoft Azure" 
    description="Learn how to use the Azure Queue service from Python to create and delete queues, and insert, get, and delete messages." 
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

# How to use Queue storage from Python
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-queues.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-queues.md)
- [Java](../articles/storage/storage-java-how-to-use-queue-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-queues.md)
- [PHP](../articles/storage/storage-php-how-to-use-queues.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-queue-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-queue-storage.md)

## Overview
This guide shows you how to perform common scenarios using the Azure Queue storage service. The samples are written in Python and use the [Python Azure Storage package](https://pypi.python.org/pypi/azure-storage). The scenarios covered include **inserting**, **peeking**,
**getting**, and **deleting** queue messages, as well as **creating and
deleting queues**. For more information on queues, refer to the [Next Steps][]Next Steps][]] section.

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
 

> [!NOTE]
> If you need to install Python or the [Python Azure package](https://pypi.python.org/pypi/azure), please see the [Python Installation Guide](../python-how-to-install.md).
> 
> 
## How To: Create a Queue
The **QueueService** object lets you work with queues. The following code creates a **QueueService** object. Add the following near the top of any Python file in which you wish to programmatically access Azure Storage:

    from azure.storage.queue import QueueService

The following code creates a **QueueService** object using the storage account name and account key. Replace 'myaccount' and 'mykey' with the real account and key.

    queue_service = QueueService(account_name='myaccount', account_key='mykey')

    queue_service.create_queue('taskqueue')


## How To: Insert a Message into a Queue
To insert a message into a queue, use the **put\_message** method to
create a new message and add it to the queue.

    queue_service.put_message('taskqueue', 'Hello World')


## How To: Peek at the Next Message
You can peek at the message in the front of a queue without removing it
from the queue by calling the **peek\_messages** method. By default,
**peek\_messages** peeks at a single message.

    messages = queue_service.peek_messages('taskqueue')
    for message in messages:
        print(message.message_text)


## How To: Dequeue the Next Message
Your code removes a message from a queue in two steps. When you call
**get\_messages**, you get the next message in a queue by default. A
message returned from **get\_messages** becomes invisible to any other
code reading messages from this queue. By default, this message stays
invisible for 30 seconds. To finish removing the message from the queue,
you must also call **delete\_message**. This two-step process of removing
a message assures that when your code fails to process a message due to
hardware or software failure, another instance of your code can get the
same message and try again. Your code calls **delete\_message** right
after the message has been processed.

    messages = queue_service.get_messages('taskqueue')
    for message in messages:
        print(message.message_text)
        queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)


## How To: Change the Contents of a Queued Message
You can change the contents of a message in-place in the queue. If the
message represents a work task, you could use this feature to update the
status of the work task. The code below uses the **update\_message**
method to update a message.

    messages = queue_service.get_messages('taskqueue')
    for message in messages:
        queue_service.update_message('taskqueue', message.message_id, 'Hello World Again', message.pop_receipt, 0)

## How To: Additional Options for Dequeuing Messages
There are two ways you can customize message retrieval from a queue.
First, you can get a batch of messages (up to 32). Second, you can set a
longer or shorter invisibility timeout, allowing your code more or less
time to fully process each message. The following code example uses the
**get\_messages** method to get 16 messages in one call. Then it processes
each message using a for loop. It also sets the invisibility timeout to
five minutes for each message.

    messages = queue_service.get_messages('taskqueue', numofmessages=16, visibilitytimeout=5*60)
    for message in messages:
        print(message.message_text)
        queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)

## How To: Get the Queue Length
You can get an estimate of the number of messages in a queue. The
**get\_queue\_metadata** method asks the queue service to return metadata
about the queue, and the **x-ms-approximate-messages-count** should be used as the index into the returned dictionary to find the count.
The result is only approximate because messages can be added or removed after the
queue service responds to your request.

    queue_metadata = queue_service.get_queue_metadata('taskqueue')
    count = queue_metadata['x-ms-approximate-messages-count']

## How To: Delete a Queue
To delete a queue and all the messages contained in it, call the
**delete\_queue** method.

    queue_service.delete_queue('taskqueue')

## Next Steps
Now that you have learned the basics of queue storage, follow these links
to learn about more complex storage tasks.

* Visit the [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/)

For more information, see also the [Python Developer Center](/develop/python/).

[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure package]: https://pypi.python.org/pypi/azure
[Python Azure Storage package]: https://pypi.python.org/pypi/azure-storage 

