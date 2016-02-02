<properties 
    pageTitle="How to use Service Bus queues with PHP | Microsoft Azure" 
    description="Learn how to use Service Bus queues in Azure. Code samples written in PHP." 
    services="service-bus" 
    documentationCenter="php" 
    authors="sethmanheim" 
    manager="timlt" 
    editor=""/>

<tags 
    ms.service="service-bus" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="PHP" 
    ms.topic="article" 
    ms.date="01/26/2016" 
    ms.author="sethm"/>

# How to use Service Bus queues
> [AZURE.SELECTOR]
- [NET](../articles/service-bus/service-bus-dotnet-how-to-use-queues.md)
- [Java](../articles/service-bus/service-bus-java-how-to-use-queues.md)
- [Node.js](../articles/service-bus/service-bus-nodejs-how-to-use-queues.md)
- [PHP](../articles/service-bus/service-bus-php-how-to-use-queues.md)
- [Python](../articles/service-bus/service-bus-python-how-to-use-queues.md)
- [Ruby](../articles/service-bus/service-bus-ruby-how-to-use-queues.md)


This guide shows you how to use Service Bus queues. The samples are written in PHP and use the [Azure SDK for PHP](../php-download-sdk.md). The scenarios covered include **creating queues**, **sending and receiving messages**, and **deleting queues**.

## What are Service Bus queues?

Service Bus queues support a **brokered messaging** communication model. When using queues, components of a distributed application do not communicate directly with each other; instead they exchange messages via a queue, which acts as an intermediary (broker). A message producer (sender) hands off a message to the queue and then continues its processing. Asynchronously, a message consumer (receiver) pulls the message from the queue and processes it. The producer does not have to wait for a reply from the consumer in order to continue to process and send further messages. Queues offer **First In, First Out (FIFO)** message delivery to one or more competing consumers. That is, messages are typically received and processed by the receivers in the order in which they were added to the queue, and each message is received and processed by only one message consumer.

![QueueConcepts](./media/howto-service-bus-queues/sb-queues-08.png)

Service Bus queues are a general-purpose technology that can be used for a wide variety of scenarios:

-   Communication between web and worker roles in a multi-tier Azure application.
-   Communication between on-premises apps and Azure-hosted apps in a hybrid solution.
-   Communication between components of a distributed application running on-premises in different organizations or departments of an organization.

Using queues enables you to scale your applications more easily, and enable more resiliency to your architecture.

## Create a service namespace

To begin using Service Bus queues in Azure, you must first create a service namespace. A namespace provides a scoping container for addressing Service Bus resources within your application.

To create a service namespace:

1.  Log on to the [Azure classic portal][].

2.  In the left navigation pane of the portal, click **Service Bus**.

3.  In the lower pane of the portal, click **Create**.
	![](./media/howto-service-bus-queues/sb-queues-03.png)

4.  In the **Add a new namespace** dialog, enter a namespace name. The system immediately checks to see if the name is available.   
	![](./media/howto-service-bus-queues/sb-queues-04.png)

5.  After making sure the namespace name is available, choose the country or region in which your namespace should be hosted (make sure you use the same country/region in which you are deploying your compute resources).

	 > [AZURE.IMPORTANT] Pick the **same region** that you intend to choose for deploying your application. This will give you the best performance.

6. 	Leave the other fields in the dialog with their default values (**Messaging** and **Standard Tier**), then click the OK check mark. The system now creates your namespace and enables it. You might have to wait several minutes as the system provisions resources for your account.

	![](./media/howto-service-bus-queues/getting-started-multi-tier-27.png)

The namespace you created takes a moment to activate, and will then appear in the portal. Wait until the namespace status is **Active** before continuing.

## Obtain the default management credentials for the namespace

In order to perform management operations, such as creating a queue on the new namespace, you must obtain the management credentials for the namespace. You can obtain these credentials from the [Azure classic portal][].

###To obtain management credentials from the portal

1.  In the left navigation pane, click the **Service Bus** node, to display the list of available namespaces:   
	![](./media/howto-service-bus-queues/sb-queues-13.png)

2.  Select the namespace you just created from the list shown:   
	![](./media/howto-service-bus-queues/sb-queues-09.png)

3.  Click **Connection Information**.   
	![](./media/howto-service-bus-queues/sb-queues-06.png)

4.  In the **Access connection information** pane, find the connection string that contains the SAS key and key name.   

	![](./media/howto-service-bus-queues/multi-web-45.png)
    
5.  Make a note of the key, or copy it to the clipboard.

  [Azure classic portal]: http://manage.windowsazure.com



## Create a PHP application
The only requirement for creating a PHP application that accesses the Azure Blob service is the referencing of classes in the [Azure SDK for PHP](../php-download-sdk.md) from within your code. You can use any development tools to create your application, or Notepad.

> [!NOTE]
> Your PHP installation must also have the [OpenSSL extension](http://php.net/openssl) installed and enabled.
> 
> 
In this guide, you will use service features which can be called from within a PHP application locally, or in code running within an Azure web role, worker role, or website.

## Get the Azure client libraries
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


## Configure your application to use Service Bus
To use the Service Bus queue APIs, do the following:

1. Reference the autoloader file using the [require_once](http://php.net/require_once) statement.
2. Reference any classes you might use.

The following example shows how to include the autoloader file and reference the **ServicesBuilder** class.

> [!NOTE]
> This example (and other examples in this article) assumes you have installed the PHP Client Libraries for Azure via Composer. If you installed the libraries manually or as a PEAR package, you must reference the **WindowsAzure.php** autoloader file.
> 
> 
    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;

In the examples below, the `require_once` statement will always be shown, but only the classes necessary for the example to execute are referenced.

## Set up a Service Bus connection
To instantiate a Service Bus client, you must first have a valid connection string in this format:

    Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]

Where **Endpoint** is typically of the format `https://[yourNamespace].servicebus.windows.net`.

To create any Azure service client you must use the **ServicesBuilder** class. You can:

* Pass the connection string directly to it.
* Use the **CloudConfigurationManager (CCM)** to check multiple external sources for the connection string:  * By default it comes with support for one external source - environmental variables
* You can add new sources by extending the **ConnectionStringSource** class



For the examples outlined here, the connection string will be passed directly.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $connectionString = "Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]";

    $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);


## How to: create a queue
You can perform management operations for Service Bus queues via the **ServiceBusRestProxy** class. A **ServiceBusRestProxy** object is constructed via the **ServicesBuilder::createServiceBusService** factory method with an appropriate connection string that encapsulates the token permissions to manage it.

The following example shows how to instantiate a **ServiceBusRestProxy** and call **ServiceBusRestProxy->createQueue** to create a queue named `myqueue` within a `MySBNamespace` service namespace:

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\ServiceBus\Models\QueueInfo;

    // Create Service Bus REST proxy.
        $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

    try    {
        $queueInfo = new QueueInfo("myqueue");

        // Create queue.
        $serviceBusRestProxy->createQueue($queueInfo);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/library/windowsazure/dd179357
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

> [!NOTE]
> You can use the `listQueues` method on `ServiceBusRestProxy` objects to check if a queue with a specified name already exists within a service namespace.
> 
> 
## How to: send messages to a queue
To send a message to a Service Bus queue, your application calls the **ServiceBusRestProxy->sendQueueMessage** method. The following code shows how to send a message to the `myqueue` queue previously created within the
`MySBNamespace` service namespace.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\ServiceBus\Models\BrokeredMessage;

    // Create Service Bus REST proxy.
    $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

    try    {
        // Create message.
        $message = new BrokeredMessage();
        $message->setBody("my message");

        // Send message.
        $serviceBusRestProxy->sendQueueMessage("myqueue", $message);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here: 
        // http://msdn.microsoft.com/library/windowsazure/hh780775
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

Messages sent to (and received from ) Service Bus queues are instances of the **BrokeredMessage** class. **BrokeredMessage** objects have a set of standard methods (such as **getLabel**, **getTimeToLive**, **setLabel**, and **setTimeToLive**) and properties that are used to hold custom application-specific properties, and a body of arbitrary application data.

Service Bus queues support a maximum message size of 256 KB (the header, which includes the standard and custom application properties, can have a maximum size of 64 KB). There is no limit on the number of messages held in a queue but there is a cap on the total size of the messages held by a queue. This upper limit on queue size is 5 GB.

## How to receive messages from a queue
The best way to receive messages from a queue is to use a **ServiceBusRestProxy->receiveQueueMessage** method. Messages can be received in two different modes: **ReceiveAndDelete** (the default) and **PeekLock**.

When using the **ReceiveAndDelete** mode, receive is a single-shot operation - that is, when Service Bus receives a read request for a message in a queue, it marks the message as being consumed and returns it to the application. **ReceiveAndDelete** mode is the simplest model and works best for scenarios in which an application can tolerate not processing a message in the event of a failure. To understand this, consider a scenario in which the consumer issues the receive request and then crashes before processing it. Because Service Bus will have marked the message as being consumed, then when the application restarts and begins consuming messages again, it will have missed the message that was consumed prior to the crash.

In **PeekLock** mode, receiving a message becomes a two stage operation, which makes it possible to support applications that cannot tolerate missing messages. When Service Bus receives a request, it finds the next message to be consumed, locks it to prevent other consumers from receiving it, and then returns it to the application. After the application finishes processing the message (or stores it reliably for future processing), it completes the second stage of the receive process by passing the received message to **ServiceBusRestProxy->deleteMessage**. When Service Bus sees the **deleteMessage** call, it will mark the message as being consumed and remove it from the queue.

The following example shows how a message can be received and processed using **PeekLock** mode (not the default mode).

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

    // Create Service Bus REST proxy.
    $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

    try    {
        // Set the receive mode to PeekLock (default is ReceiveAndDelete).
        $options = new ReceiveMessageOptions();
        $options->setPeekLock();

        // Receive message.
        $message = $serviceBusRestProxy->receiveQueueMessage("myqueue", $options);
        echo "Body: ".$message->getBody()."<br />";
        echo "MessageID: ".$message->getMessageId()."<br />";

        /*---------------------------
            Process message here.
        ----------------------------*/

        // Delete message. Not necessary if peek lock is not set.
        echo "Message deleted.<br />";
        $serviceBusRestProxy->deleteMessage($message);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/windowsazure/hh780735
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## How to: handle application crashes and unreadable messages
Service Bus provides functionality to help you gracefully recover from errors in your application or difficulties processing a message. If a receiver application is unable to process the message for some reason, then it can call the **unlockMessage** method on the received message (instead of the **deleteMessage** method). This will cause Service Bus to unlock the message within the queue and make it available to be received again, either by the same consuming application or by another consuming application.

There is also a timeout associated with a message locked within the queue, and if the application fails to process the message before the lock timeout expires (for example, if the application crashes), then Service Bus will unlock the message automatically and make it available to be received again.

In the event that the application crashes after processing the message but before the **deleteMessage** request is issued, then the message will be redelivered to the application when it restarts. This is often called **At Least Once Processing**; that is, each message is processed at least once but in certain situations the same message may be redelivered. If the scenario cannot tolerate duplicate processing, then adding additional logic to applications to handle duplicate message delivery is recommended. This is often achieved using the **getMessageId** method of the message, which remains constant across delivery attempts.

## Next steps
Now that you've learned the basics of Service Bus queues, see [Queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md) for more information.

For more information, also see the [PHP Developer Center](/develop/php/).

[Queues, Topics, and Subscriptions]: service-bus-queues-topics-subscriptions.md
[require_once]: http://php.net/require_once


