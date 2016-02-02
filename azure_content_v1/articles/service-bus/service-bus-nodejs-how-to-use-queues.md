<properties 
    pageTitle="How to use Service Bus queues in Node.js | Microsoft Azure" 
    description="Learn how to use Service Bus queues in Azure from a Node.js app." 
    services="service-bus" 
    documentationCenter="nodejs" 
    authors="sethmanheim" 
    manager="timlt" 
    editor=""/>

<tags 
    ms.service="service-bus" 
    ms.workload="tbd" 
    ms.tgt_pltfrm="na" 
    ms.devlang="nodejs" 
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


This article describes how to use Service Bus queues. The samples are written in JavaScript and use the Node.js Azure module. The scenarios covered include **creating queues**, **sending and receiving messages**, and **deleting queues**. For more information on queues, see the [Next steps][]Next steps][]] section.

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



## Create a Node.js application
Create a blank Node.js application. For instructions on how to create a Node.js application, see [Create and deploy a Node.js application to an Azure Website](../app-service-web/web-sites-nodejs-develop-deploy-mac.md), or [Node.js Cloud Service](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (using Windows PowerShell).

## Configure your application to use Service Bus
To use Azure Service Bus, download and use the Node.js Azure package. This package includes a set of libraries that communicate with the Service Bus REST services.

### Use Node Package Manager (NPM) to obtain the package
1. Use the **Windows PowerShell for Node.js** command window to navigate to the **c:\\node\\sbqueues\\WebRole1** folder in which you created your sample application.

2. Type **npm install azure** in the command window, which should result in output similar to the following:

    ```
 azure@0.7.5 node_modules\azure
     ├── dateformat@1.0.2-1.2.3
     ├── xmlbuilder@0.4.2
     ├── node-uuid@1.2.0
     ├── mime@1.2.9
     ├── underscore@1.4.4
     ├── validator@1.1.1
     ├── tunnel@0.0.2
     ├── wns@0.5.3
     ├── xml2js@0.2.7 (sax@0.5.2)
     └── request@2.21.0 (json-stringify-safe@4.0.0, forever-agent@0.5.0, aws-sign@0.3.0, tunnel-agent@0.3.0, oauth-sign@0.3.0, qs@0.6.5, cookie-jar@0.3.0, node-uuid@1.4.0, http-signature@0.9.11, form-data@0.0.8, hawk@0.13.1)
 ```
3. You can manually run the **ls** command to verify that a **node\_modules** folder was created. Inside that folder find the **azure** package, which contains the libraries you need to access Service Bus queues.


### Import the module
Using Notepad or another text editor, add the following to the top of the **server.js** file of the application:

```
var azure = require('azure');
```

### Set up an Azure Service Bus connection
The Azure module reads the environment variables AZURE\_SERVICEBUS\_NAMESPACE and AZURE\_SERVICEBUS\_ACCESS\_KEY to obtain information required to connect to Service Bus. If these environment variables are not set, you must specify the account information when calling **createServiceBusService**.

For an example of setting the environment variables in a configuration file for an Azure Cloud Service, see [Node.js Cloud Service with Storage](../cloud-services/storage-nodejs-use-table-storage-cloud-service-app.md).

For an example of setting the environment variables in the [Azure classic portal](http://manage.windowsazure.com) for an Azure Website, see [Node.js Web Application with Storage](../storage/storage-nodejs-how-to-use-table-storage.md).

## Create a queue
The **ServiceBusService** object enables you to work with Service Bus queues. The following code creates a **ServiceBusService** object. Add it near the top of the **server.js** file, after the statement to import the Azure module:

```
var serviceBusService = azure.createServiceBusService();
```

By calling **createQueueIfNotExists** on the **ServiceBusService** object, the specified queue is returned (if it exists), or a new queue with the specified name is created. The following code uses **createQueueIfNotExists** to create or connect to the queue named `myqueue`:

```
serviceBusService.createQueueIfNotExists('myqueue', function(error){
    if(!error){
        // Queue exists
    }
});
```

**createServiceBusService** also supports additional options, which enable you to override default queue settings such as message time to live or maximum queue size. The following example sets the maximum queue size to 5 GB, and a time to live (TTL) value of 1 minute:

```
var queueOptions = {
      MaxSizeInMegabytes: '5120',
      DefaultMessageTimeToLive: 'PT1M'
    };

serviceBusService.createQueueIfNotExists('myqueue', queueOptions, function(error){
    if(!error){
        // Queue exists
    }
});
```

### Filters
Optional filtering operations can be applied to operations performed using **ServiceBusService**. Filtering operations can include logging, automatically retrying, etc. Filters are objects that implement a method with the signature:

```
function handle (requestOptions, next)
```

After doing its pre-processing on the request options, the method must call `next`, passing a callback with the following signature:

```
function (returnObject, finalCallback, next)
```

In this callback, and after processing the **returnObject** (the response from the request to the server), the callback must either invoke `next` if it exists to continue processing other filters, or simply invoke `finalCallback`, which ends the service invocation.

Two filters that implement retry logic are included with the Azure SDK for Node.js, **ExponentialRetryPolicyFilter** and **LinearRetryPolicyFilter**. The following creates a **ServiceBusService** object that uses the **ExponentialRetryPolicyFilter**:

```
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var serviceBusService = azure.createServiceBusService().withFilter(retryOperations);
```

## Send messages to a queue
To send a message to a Service Bus queue, your application calls the **sendQueueMessage** method on the **ServiceBusService** object. Messages sent to (and received from) Service Bus queues are **BrokeredMessage** objects, and have a set of standard properties (such as **Label** and **TimeToLive**), a dictionary that is used to hold custom application specific properties, and a body of arbitrary application data. An application can set the body of the message by passing a string as the message. Any required standard properties are populated with default values.

The following example demonstrates how to send a test message to the queue named `myqueue` using **sendQueueMessage**:

```
var message = {
    body: 'Test message',
    customProperties: {
        testproperty: 'TestValue'
    }};
serviceBusService.sendQueueMessage('myqueue', message, function(error){
    if(!error){
        // message sent
    }
});
```

Service Bus queues support a maximum message size of 256 KB (the header, which includes the standard and custom application properties, can have a maximum size of 64 KB). There is no limit on the number of messages held in a queue but there is a cap on the total size of the messages held by a queue. This queue size is defined at creation time, with an upper limit of 5 GB. For more information about quotas, see [Azure Queues and Service Bus queues](service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas).

## Receive messages from a queue
Messages are received from a queue using the **receiveQueueMessage** method on the **ServiceBusService** object. By default, messages are deleted from the queue as they are read; however, you can read (peek) and lock the message without deleting it from the queue by setting the optional parameter **isPeekLock** to **true**.

The default behavior of reading and deleting the message as part of the receive operation is the simplest model, and works best for scenarios in which an application can tolerate not processing a message in the event of a failure. To understand this, consider a scenario in which the consumer issues the receive request and then crashes before processing it. Because Service Bus will have marked the message as being consumed, then when the application restarts and begins consuming messages again, it will have missed the message that was consumed prior to the crash.

If the **isPeekLock** parameter is set to **true**, the receive becomes a two stage operation, which makes it possible to support applications that cannot tolerate missing messages. When Service Bus receives a request, it finds the next message to be consumed, locks it to prevent other consumers receiving it, and then returns it to the application. After the application finishes processing the message (or stores it reliably for future processing), it completes the second stage of the receive process by calling **deleteMessage** method and providing the message to be deleted as a parameter. The **deleteMessage** method will mark the message as being consumed and remove it from the queue.

The following example demonstrates how to receive and process messages using **receiveQueueMessage**. The example first receives and deletes a message, and then receives a message using **isPeekLock** set to **true**, then deletes the message using **deleteMessage**:

```
serviceBusService.receiveQueueMessage('myqueue', function(error, receivedMessage){
    if(!error){
        // Message received and deleted
    }
});
serviceBusService.receiveQueueMessage('myqueue', { isPeekLock: true }, function(error, lockedMessage){
    if(!error){
        // Message received and locked
        serviceBusService.deleteMessage(lockedMessage, function (deleteError){
            if(!deleteError){
                // Message deleted
            }
        });
    }
});
```

## How to handle application crashes and unreadable messages
Service Bus provides functionality to help you gracefully recover from errors in your application or difficulties processing a message. If a receiver application is unable to process the message for some reason, then it can call the **unlockMessage** method on the **ServiceBusService** object. This will cause Service Bus to unlock the
message within the queue and make it available to be received again, either by the same consuming application or by another consuming application.

There is also a timeout associated with a message locked within the queue, and if the application fails to process the message before the lock timeout expires (e.g., if the application crashes), then Service Bus will unlock the message automatically and make it available to be received again.

In the event that the application crashes after processing the message but before the **deleteMessage** method is called, then the message will be redelivered to the application when it restarts. This is often called **At Least Once Processing**, that is, each message will be processed at least once but in certain situations the same message may be redelivered. If the scenario cannot tolerate duplicate processing, then application developers should add additional logic to their application to handle duplicate message delivery. This is often achieved using the **MessageId** property of the message, which will remain constant across delivery attempts.

## Next steps
To learn more, see the following resources.

* [Queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md)
* [Azure SDK for Node](https://github.com/Azure/azure-sdk-for-node) repository on GitHub
* [Node.js Developer Center](/develop/nodejs/)

  [Azure SDK for Node]: https://github.com/Azure/azure-sdk-for-node
  [Azure classic portal]: http://manage.windowsazure.com

  [Node.js Cloud Service]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md
  [Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
  [Create and deploy a Node.js application to an Azure Website]: ../app-service-web/web-sites-nodejs-develop-deploy-mac.md
  [Node.js Cloud Service with Storage]: ../cloud-services/storage-nodejs-use-table-storage-cloud-service-app.md
  [Node.js Web Application with Storage]: ../storage/storage-nodejs-how-to-use-table-storage.md
  [Azure Queues and Service Bus queues]: service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas

