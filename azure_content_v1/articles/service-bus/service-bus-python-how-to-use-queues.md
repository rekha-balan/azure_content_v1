<properties 
    pageTitle="How to use Service Bus queues with Python | Microsoft Azure" 
    description="Learn how to use Azure Service Bus queues from Python." 
    services="service-bus" 
    documentationCenter="python" 
    authors="sethmanheim" 
    manager="timlt" 
    editor=""/>

<tags 
    ms.service="service-bus" 
    ms.workload="tbd" 
    ms.tgt_pltfrm="na" 
    ms.devlang="python" 
    ms.topic="article" 
    ms.date="10/08/2015" 
    ms.author="sethm"/>


# How to use Service Bus queues
> [AZURE.SELECTOR]
- [NET](../articles/service-bus/service-bus-dotnet-how-to-use-queues.md)
- [Java](../articles/service-bus/service-bus-java-how-to-use-queues.md)
- [Node.js](../articles/service-bus/service-bus-nodejs-how-to-use-queues.md)
- [PHP](../articles/service-bus/service-bus-php-how-to-use-queues.md)
- [Python](../articles/service-bus/service-bus-python-how-to-use-queues.md)
- [Ruby](../articles/service-bus/service-bus-ruby-how-to-use-queues.md)


This article describes how to use Service Bus queues. The samples are written in Python and use the [Python Azure package](https://pypi.python.org/pypi/azure). The scenarios covered include **creating queues, sending and receiving messages**, and **deleting queues**.

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



> [!NOTE]
> To install Python or the [Python Azure package](https://pypi.python.org/pypi/azure), see the [Python Installation Guide](../python-how-to-install.md).
> 
> 
## Create a queue
The **ServiceBusService** object enables you to work with queues. Add the following code near the top of any Python file in which you wish to programmatically access Service Bus:

```
from azure.servicebus import ServiceBusService, Message, Queue
```

The following code creates a **ServiceBusService** object. Replace `mynamespace`, `sharedaccesskeyname`, and `sharedaccesskey` with your namespace, shared access signature (SAS) key name, and value.

```
bus_service = ServiceBusService(
    service_namespace='mynamespace',
    shared_access_key_name='sharedaccesskeyname',
    shared_access_key_value='sharedaccesskey')
```

The values for the SAS key name and value can be found in the [Azure classic portal](http://manage.windowsazure.com) connection information, or in the Visual Studio **Properties** pane when selecting the Service Bus namespace in Server Explorer (as shown in the previous section).

```
bus_service.create_queue('taskqueue')
```

**create_queue** also supports additional options, which enable you to override default queue settings such as message time to live (TTL) or maximum queue size. The following example sets the maximum queue size to 5GB, and the TTL value to 1 minute:

```
queue_options = Queue()
queue_options.max_size_in_megabytes = '5120'
queue_options.default_message_time_to_live = 'PT1M'

bus_service.create_queue('taskqueue', queue_options)
```

## Send messages to a queue
To send a message to a Service Bus queue, your application calls the **send\_queue\_message** method on the **ServiceBusService** object.

The following example demonstrates how to send a test message to the queue named *taskqueue using* **send\_queue\_message**:

```
msg = Message(b'Test Message')
bus_service.send_queue_message('taskqueue', msg)
```

Service Bus queues support a maximum message size of 256 KB (the header, which includes the standard and custom application properties, can have a maximum size of 64 KB). There is no limit on the number of messages held in a queue but there is a cap on the total size of the messages held by a queue. This queue size is defined at creation time, with an upper limit of 5 GB. For more information about quotas, see [Azure Queues and Service Bus queues](service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas).

## Receive messages from a queue
Messages are received from a queue using the **receive\_queue\_message** method on the **ServiceBusService** object:

```
msg = bus_service.receive_queue_message('taskqueue', peek_lock=False)
print(msg.body)
```

Messages are deleted from the queue as they are read when the parameter **peek\_lock** is set to **False**. You can read (peek) and lock the message without deleting it from the queue by setting the parameter **peek\_lock** to **True**.

The behavior of reading and deleting the message as part of the receive operation is the simplest model, and works best for scenarios in which an application can tolerate not processing a message in the event of a failure. To understand this, consider a scenario in which the consumer issues the receive request and then crashes before processing it. Because Service Bus will have marked the message as being consumed, then when the application restarts and begins consuming messages again, it will have missed the message that was consumed prior to the crash.

If the **peek\_lock** parameter is set to **True**, the receive becomes a two stage operation, which makes it possible to support applications that cannot tolerate missing messages. When Service Bus receives a request, it finds the next message to be consumed, locks it to prevent other consumers receiving it, and then returns it to the application. After the application finishes processing the message (or stores it reliably for future processing), it completes the second stage of the receive process by calling the **delete** method on the **Message** object. The **delete** method will mark the message as being consumed and remove it from the queue.

```
msg = bus_service.receive_queue_message('taskqueue', peek_lock=True)
print(msg.body)

msg.delete()
```

## How to handle application crashes and unreadable messages
Service Bus provides functionality to help you gracefully recover from errors in your application or difficulties processing a message. If a receiver application is unable to process the message for some reason, then it can call the **unlock** method on the **Message** object. This will cause Service Bus to unlock the message within the queue and make it available to be received again, either by the same consuming application or by another consuming application.

There is also a timeout associated with a message locked within the queue, and if the application fails to process the message before the lock timeout expires (e.g., if the application crashes), then Service Bus will unlock the message automatically and make it available to be received again.

In the event that the application crashes after processing the message but before the **delete** method is called, then the message will be redelivered to the application when it restarts. This is often called **At Least Once Processing**, that is, each message will be processed at least once but in certain situations the same message may be redelivered. If the scenario cannot tolerate duplicate processing, then application developers should add additional logic to their application to handle duplicate message delivery. This is often achieved using the **MessageId** property of the message, which will remain constant across delivery attempts.

## Next steps
Now that you have learned the basics of Service Bus queues, follow these links to learn more.

* See [Queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md).

[Azure classic portal]: http://manage.windowsazure.com
[Python Azure package]: https://pypi.python.org/pypi/azure  
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[Azure Queues and Service Bus queues]: service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas

