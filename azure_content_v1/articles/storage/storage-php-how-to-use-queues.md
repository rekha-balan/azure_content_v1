<properties
    pageTitle="How to use Queue storage from PHP | Microsoft Azure"
    description="Learn how to use the Azure Queue storage service to create and delete queues, and insert, get, and delete messages. Samples are written in PHP."
    documentationCenter="php"
    services="storage"
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

# How to use Queue storage from PHP
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-queues.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-queues.md)
- [Java](../articles/storage/storage-java-how-to-use-queue-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-queues.md)
- [PHP](../articles/storage/storage-php-how-to-use-queues.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-queue-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-queue-storage.md)

## Overview
This guide will show you how to perform common scenarios by using the Azure Queue storage service. The samples are written via classes from the Windows SDK for PHP. The covered scenarios include inserting, peeking, getting, and deleting queue messages, as well as creating and deleting queues.

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
 

## Create a PHP application
The only requirement for creating a PHP application that accesses Azure Queue storage is the referencing of classes from the Azure SDK for PHP from within your code. You can use any development tools to create your application, including Notepad.

In this guide, you will use Queue storage features that can be called within a PHP application locally, or in code running within an Azure web role, worker role, or website.

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


## Configure your application to access Queue storage
To use the APIs for Azure Queue storage, you need to:

1. Reference the autoloader file by using the [require_once](http://www.php.net/manual/en/function.require-once.php) statement.
2. Reference any classes that you might use.

The following example shows how to include the autoloader file and reference the **ServicesBuilder** class.

> [!NOTE]
> This example (and other examples in this article) assumes that you have installed the PHP Client Libraries for Azure via Composer. If you installed the libraries manually or as a PEAR package, you will need to reference the `WindowsAzure.php` autoloader file.
> 
> 
    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;


In the examples below, the `require_once` statement will be shown always, but only the classes that are necessary for the example to execute will be referenced.

## Set up an Azure storage connection
To instantiate an Azure Queue storage client, you must first have a valid connection string. The format for the queue service connection string is as follows.

For accessing a live service:

    DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

For accessing the emulator storage:

    UseDevelopmentStorage=true


To create any Azure service client, you need to use the **ServicesBuilder** class. You can use either of the following techniques:

* Pass the connection string directly to it.
* Use **CloudConfigurationManager (CCM)** to check multiple external sources for the connection string:  * By default, it comes with support for one external source—environmental variables.
* You can add new sources by extending the **ConnectionStringSource** class.



For the examples outlined here, the connection string will be passed directly.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);


## Create a queue
A **QueueRestProxy** object lets you create a queue by using the **createQueue** method. When creating a queue, you can set options on the queue, but doing so is not required. (The example below shows how to set metadata on a queue.)

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\CreateQueueOptions;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // OPTIONAL: Set queue metadata.
    $createQueueOptions = new CreateQueueOptions();
    $createQueueOptions->addMetaData("key1", "value1");
    $createQueueOptions->addMetaData("key2", "value2");

    try    {
        // Create queue.
        $queueRestProxy->createQueue("myqueue", $createQueueOptions);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

> [!NOTE]
> You should not rely on case sensitivity for metadata keys. All keys are read from the service in lowercase.
> 
> 
## Add a message to a queue
To add a message to a queue, use **QueueRestProxy->createMessage**. The method takes the queue name, the message text, and message options (which are optional).

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\CreateMessageOptions;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    try    {
        // Create message.
        $builder = new ServicesBuilder();
        $queueRestProxy->createMessage("myqueue", "Hello World!");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Peek at the next message
You can peek at a message (or messages) at the front of a queue without removing it from the queue by calling **QueueRestProxy->peekMessages**. By default, the **peekMessage** method returns a single message, but you can change that value by using the **PeekMessagesOptions->setNumberOfMessages** method.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\PeekMessagesOptions;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // OPTIONAL: Set peek message options.
    $message_options = new PeekMessagesOptions();
    $message_options->setNumberOfMessages(1); // Default value is 1.

    try    {
        $peekMessagesResult = $queueRestProxy->peekMessages("myqueue", $message_options);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

    $messages = $peekMessagesResult->getQueueMessages();

    // View messages.
    $messageCount = count($messages);
    if($messageCount <= 0){
        echo "There are no messages.<br />";
    }
    else{
        foreach($messages as $message)    {
            echo "Peeked message:<br />";
            echo "Message Id: ".$message->getMessageId()."<br />";
            echo "Date: ".date_format($message->getInsertionDate(), 'Y-m-d')."<br />";
            echo "Message text: ".$message->getMessageText()."<br /><br />";
        }
    }

## De-queue the next message
Your code removes a message from a queue in two steps. First, you call **QueueRestProxy->listMessages**, which makes the message invisible to any other code that's reading from the queue. By default, this message will stay invisible for 30 seconds. (If the message is not deleted in this time period, it will become visible on the queue again.) To finish removing the message from the queue, you must call **QueueRestProxy->deleteMessage**. This two-step process of removing a message assures that when your code fails to process a message due to hardware or software failure, another instance of your code can get the same message and try again. Your code calls **deleteMessage** right after the message has been processed.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // Get message.
    $listMessagesResult = $queueRestProxy->listMessages("myqueue");
    $messages = $listMessagesResult->getQueueMessages();
    $message = $messages[0];

    /* ---------------------
        Process message.
       --------------------- */

    // Get message ID and pop receipt.
    $messageId = $message->getMessageId();
    $popReceipt = $message->getPopReceipt();

    try    {
        // Delete message.
        $queueRestProxy->deleteMessage("myqueue", $messageId, $popReceipt);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Change the contents of a queued message
You can change the contents of a message in-place in the queue by calling **QueueRestProxy->updateMessage**. If the message represents a work task, you could use this feature to update the status of the work task. The following code updates the queue message with new contents, and it sets the visibility timeout to extend another 60 seconds. This saves the state of work that's associated with the message, and it gives the client another minute to continue working on the message. You could use this technique to track multi-step workflows on queue messages, without having to start over from the beginning if a processing step fails due to hardware or software failure. Typically, you would keep a retry count as well, and if the message is retried more than *n* times, you would delete it. This protects against a message that triggers an application error each time it is processed.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // Get message.
    $listMessagesResult = $queueRestProxy->listMessages("myqueue");
    $messages = $listMessagesResult->getQueueMessages();
    $message = $messages[0];

    // Define new message properties.
    $new_message_text = "New message text.";
    $new_visibility_timeout = 5; // Measured in seconds.

    // Get message ID and pop receipt.
    $messageId = $message->getMessageId();
    $popReceipt = $message->getPopReceipt();

    try    {
        // Update message.
        $queueRestProxy->updateMessage("myqueue",
                                    $messageId,
                                    $popReceipt,
                                    $new_message_text,
                                    $new_visibility_timeout);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Additional options for de-queuing messages
There are two ways that you can customize message retrieval from a queue. First, you can get a batch of messages (up to 32). Second, you can set a longer or shorter visibility timeout, allowing your code more or less time to fully process each message. The following code example uses the **getMessages** method to get 16 messages in one call. Then it processes each message by using a **for** loop. It also sets the invisibility timeout to five minutes for each message.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\ListMessagesOptions;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // Set list message options.
    $message_options = new ListMessagesOptions();
    $message_options->setVisibilityTimeoutInSeconds(300);
    $message_options->setNumberOfMessages(16);

    // Get messages.
    try{
        $listMessagesResult = $queueRestProxy->listMessages("myqueue",
                                                         $message_options);
        $messages = $listMessagesResult->getQueueMessages();

        foreach($messages as $message){

            /* ---------------------
                Process message.
            --------------------- */

            // Get message Id and pop receipt.
            $messageId = $message->getMessageId();
            $popReceipt = $message->getPopReceipt();

            // Delete message.
            $queueRestProxy->deleteMessage("myqueue", $messageId, $popReceipt);
        }
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Get queue length
You can get an estimate of the number of messages in a queue. The **QueueRestProxy->getQueueMetadata** method asks the queue service to return metadata about the queue. Calling the **getApproximateMessageCount** method on the returned object provides a count of how many messages are in a queue. The count is only approximate because messages can be added or removed after the queue service responds to your request.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    try    {
        // Get queue metadata.
        $queue_metadata = $queueRestProxy->getQueueMetadata("myqueue");
        $approx_msg_count = $queue_metadata->getApproximateMessageCount();
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

    echo $approx_msg_count;

## Delete a queue
To delete a queue and all the messages in it, call the **QueueRestProxy->deleteQueue** method.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create queue REST proxy.
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    try    {
        // Delete queue.
        $queueRestProxy->deleteQueue("myqueue");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179446.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }


## Next steps
Now that you've learned the basics of Azure Queue storage, follow these links to learn about more complex storage tasks:

* Visit the [Azure Storage Team blog](http://blogs.msdn.com/b/windowsazurestorage/).

For more information, see also the [PHP Developer Center](/develop/php/).

[download]: http://go.microsoft.com/fwlink/?LinkID=252473
[require_once]: http://www.php.net/manual/en/function.require-once.php
[Azure Portal]: portal.azure.com

