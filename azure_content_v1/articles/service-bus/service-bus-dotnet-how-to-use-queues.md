<properties
    pageTitle="How to use Service Bus queues (.NET) | Microsoft Azure"
    description="Learn how to use Service Bus queues in Azure. Code samples written in C# using the .NET API."
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="get-started-article"
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


This article describes how to use Service Bus queues. The samples are written in C# and use the .NET API. The scenarios covered include creating queues and sending and receiving messages. For more information about queues, see the [Next steps](#next-steps.md) section.

> [AZURE.NOTE]
> To complete this tutorial, you need an Azure account. You can [activate your MSDN subscriber benefits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) or [sign up for a free trial](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A85619ABF).



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



## Add the Service Bus NuGet package
The [Service Bus **NuGet** package](https://www.nuget.org/packages/WindowsAzure.ServiceBus) is the easiest way to get the Service Bus API and to configure your application with all of the Service Bus dependencies. The NuGet Visual Studio extension makes it easy to install and update libraries and tools in Visual Studio and Visual Studio Express. The Service Bus NuGet package is the easiest way
to get the Service Bus API and to configure your application with all of the Service Bus dependencies.

To install the NuGet package in your application, do the following:

1. In Solution Explorer, right-click **References**, then click **Manage NuGet Packages**.
2. Search for "Service Bus" and select the **Microsoft Azure Service Bus** item. Click **Install** to complete the installation, then close this dialog box.

   ![][7]


You are now ready to write code for Service Bus.

## Set up a Service Bus connection string
Service Bus uses a connection string to store endpoints and credentials. You can put your connection string in a configuration file, rather than hard-coding it:

* When using Azure Cloud Services, it is recommended that you store your connection string using the Azure service configuration system (.csdef and .cscfg files).
* When using Azure websites or Azure Virtual Machines, it is recommended that you store your connection string using the .NET configuration system (for example, the Web.config file).

In both cases, you can retrieve your connection string using the [CloudConfigurationManager.GetSetting](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager.getsetting.aspx) method, as shown later in this article.

### Configure your connection string when using Cloud Services
The service configuration mechanism is unique to Azure Cloud Services projects and enables you to dynamically change configuration settings from the [Azure classic portal](http://manage.windowsazure.com) without redeploying your application. For example, add a `Setting` label to your service definition (.csdef) file, as shown in the next example.

```
<ServiceDefinition name="Azure1">
...
    <WebRole name="MyRole" vmsize="Small">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString" />
        </ConfigurationSettings>
    </WebRole>
...
</ServiceDefinition>
```
You then specify values in the service configuration (.cscfg) file, as shown in the next example.

```
<ServiceConfiguration serviceName="Azure1">
...
    <Role name="MyRole">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString"
                     value="Endpoint=sb://yourServiceNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
        </ConfigurationSettings>
    </Role>
...
</ServiceConfiguration>
```

Use the Shared Access Signature (SAS) key name and key values retrieved from the Azure classic portal as described in the previous section.

### Configure your connection string when using websites or Azure Virtual Machines
When using websites or Virtual Machines, it is recommended that you use the .NET configuration system (for example, **Web.config**). You store the connection string using the `<appSettings>` element.

```
<configuration>
    <appSettings>
        <add key="Microsoft.ServiceBus.ConnectionString"
             value="Endpoint=sb://yourServiceNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
    </appSettings>
</configuration>
```

Use the SAS name and key values that you retrieved from the Azure classic portal, as described in the previous section.

## Create a queue
You can perform management operations for Service Bus queues using the [NamespaceManager](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx) class. This class provides methods to create, enumerate, and delete queues.

This example constructs a [NamespaceManager](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx) object using the Azure [CloudConfigurationManager](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager) class with a connection string consisting of the base address of a Service Bus service namespace and the appropriate SAS credentials with permissions to manage it. This connection string is of the form shown in the following example.

````
Endpoint=sb://yourServiceNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedSecretValue=yourKey
````

Use the following example, given the configuration settings in the previous section.

```
// Create the queue if it does not exist already.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.QueueExists("TestQueue"))
{
    namespaceManager.CreateQueue("TestQueue");
}
```

There are overloads of the [CreateQueue](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.createqueue.aspx) method that enable you to tune properties
of the queue (for example, to set the default time-to-live (TTL) value to be applied to messages sent to the queue). These settings are applied using the [QueueDescription](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queuedescription.aspx) class. The following example shows how to create a queue named `TestQueue` with a maximum size of 5 GB and a default message TTL of 1 minute.

```
// Configure queue settings.
QueueDescription qd = new QueueDescription("TestQueue");
qd.MaxSizeInMegabytes = 5120;
qd.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

// Create a new queue with custom settings.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.QueueExists("TestQueue"))
{
    namespaceManager.CreateQueue(qd);
}
```

> [!NOTE]
> You can use the [QueueExists](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.queueexists.aspx) method on [NamespaceManager](https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx) objects to check if a queue with a specified name already exists within a service namespace.
> 
> 
## Send messages to a queue
To send a message to a Service Bus queue, your application creates a
[QueueClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx) object using the connection string.

The following code demonstrates how to create a [QueueClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx) object for the `TestQueue` queue you just created using the [CreateFromConnectionString](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.createfromconnectionstring.aspx) API call.

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

QueueClient Client =
    QueueClient.CreateFromConnectionString(connectionString, "TestQueue");

Client.Send(new BrokeredMessage());
```

Messages sent to, and received from Service Bus queues are instances of the [BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) class. [BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) objects have a set of standard properties (such as [Label](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) and [TimeToLive](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx)), a dictionary
that is used to hold custom application specific properties, and a body of arbitrary application data. An application can set the body of the message by passing any serializable object into the constructor of the[BrokeredMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) object, and the appropriate **DataContractSerializer** is then used to serialize the object. Alternatively, you can provide a **System.IO.Stream** object.

The following example demonstrates how to send five test messages to the `TestQueue` [QueueClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx) object obtained in the previous code example.

```
for (int i=0; i<5; i++)
{
  // Create message, passing a string message for the body.
  BrokeredMessage message = new BrokeredMessage("Test message " + i);

  // Set some addtional custom app-specific properties.
  message.Properties["TestProperty"] = "TestValue";
  message.Properties["Message number"] = i;

  // Send message to the queue.
  Client.Send(message);
}
```

Service Bus queues support a [maximum message size of 256 Kb](service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas) (the header, which includes  the standard and custom application properties, can have a maximum size of 64 KB). There is no limit on the number of messages held in a queue but there is a cap on the total size of the messages held by a queue. This queue size is defined at creation time, with an upper limit of 5 GB. If partitioning is enabled, the upper limit is higher. For more information, see [Partitioned messaging entities](service-bus-partitioning.md).

## How to receive messages from a queue
The recommended way to receive messages from a queue is to use a [QueueClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx) object. [QueueClient](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx) objects can work in two different modes: [ReceiveAndDelete and PeekLock](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.receivemode.aspx).

When using the **ReceiveAndDelete** mode, the receive is a single-shot operation - that is, when Service Bus receives a read request for a message in a queue, it marks the message as consumed, and returns it to the application. **ReceiveAndDelete** is the simplest model and works best for scenarios in which an application can tolerate not processing a message in the event of a failure. To understand this, consider a scenario in which the consumer issues the receive request and then crashes before processing it. Because Service Bus will have marked
the message as consumed, when the application restarts and begins consuming messages again, it will have missed the message that was consumed prior to the crash.

In **PeekLock** mode (which is the default mode), the receive becomes a two-stage operation, which makes it possible to support applications that cannot tolerate missing messages. When Service Bus receives a request,
it finds the next message to be consumed, locks it to prevent other consumers receiving it, and then returns it to the application. After the application finishes processing the message (or stores it reliably for future processing), it completes the second stage of the receive process by calling [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) on the received message. When Service Bus sees the [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) call, it marks the message as consumed, and removes it from the queue.

The following example demonstrates how messages can be received and processed using the default **PeekLock** mode. To specify a different [ReceiveMode](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.receivemode.aspx) value, you can use another overload of
[CreateFromConnectionString](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.createfromconnectionstring.aspx). This example uses the [OnMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.onmessage.aspx) callback
to process messages as they arrive into `TestQueue`.

```
string connectionString =
  CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
QueueClient Client =
  QueueClient.CreateFromConnectionString(connectionString, "TestQueue");

// Configure the callback options.
OnMessageOptions options = new OnMessageOptions();
options.AutoComplete = false;
options.AutoRenewTimeout = TimeSpan.FromMinutes(1);

// Callback to handle received messages.
Client.OnMessage((message) =>
{
    try
    {
        // Process message from queue.
        Console.WriteLine("Body: " + message.GetBody<string>());
        Console.WriteLine("MessageID: " + message.MessageId);
        Console.WriteLine("Test Property: " +
        message.Properties["TestProperty"]);

        // Remove message from queue.
        message.Complete();
    }
    catch (Exception)
    {
        // Indicates a problem, unlock message in queue.
        message.Abandon();
    }
}, options);
```

This example configures the [OnMessage](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.onmessage.aspx) callback using an [OnMessageOptions](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.onmessageoptions.aspx) object. [AutoComplete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.onmessageoptions.autocomplete.aspx)
is set to **false** to enable manual control over when to call [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) on the received message. [AutoRenewTimeout](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.onmessageoptions.autorenewtimeout.aspx) is set to 1 minute, which causes the client to wait for up to one minute for a message before the call times out and the client makes a new call to check for messages. This property value reduces the number of times the client makes chargeable calls that do not retrieve messages.

## How to handle application crashes and unreadable messages
Service Bus provides functionality to help you gracefully recover from errors in your application or with difficulties processing a message. If a receiver application is unable to process the message for some reason, then it can call the [Abandon](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.abandon.aspx) method on the received message (instead of the [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) method). This causes Service Bus to unlock the message within the queue and make it available to be received again, either by the same consuming application or by another consuming application.

There is also a time-out associated with a message locked within the queue, and if the application fails to process the message before the lock time-out expires (for example, if the application crashes), then Service Bus unlocks the message automatically and makes it available to be received again.

In the event that the application crashes after processing the message but before the [Complete](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) request is issued, the message will be redelivered to the application when it restarts. This is often called **At Least Once Processing**; that is, each message is processed at least once but in certain situations the same message may be redelivered. If the scenario cannot tolerate duplicate processing, then application developers should add additional logic to their application to handle duplicate message delivery. This is often achieved using the [MessageId](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) property of the message, which remains constant across delivery attempts.

## Next steps
Now that you've learned the basics of Service Bus queues, follow these links to learn more.

* Read about Service Bus messaging entities in [Queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md).
* Build a working application that sends and receives messages to and from a Service Bus queue with the [Service Bus brokered messaging .NET tutorial](service-bus-brokered-tutorial-dotnet.md).
* Download Service Bus samples from [Azure samples](https://code.msdn.microsoft.com/site/search?query=service%20busf%5B0%5D.Value=service%20busf%5B0%5D.Type=SearchTextac=2) or see the [overview of Service Bus samples](service-bus-samples.md).

  [Azure classic portal]: http://manage.windowsazure.com
  [7]: ./media/service-bus-dotnet-how-to-use-queues/getting-started-multi-tier-13.png
  [Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
  [Service Bus brokered messaging .NET tutorial]: service-bus-brokered-tutorial-dotnet.md
  [Azure samples]: https://code.msdn.microsoft.com/site/search?query=service%20bus&f%5B0%5D.Value=service%20bus&f%5B0%5D.Type=SearchText&ac=2
  [overview of Service Bus samples]: service-bus-samples.md
  [GetSetting]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager.getsetting.aspx
  [CloudConfigurationManager]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager
  [NamespaceManager]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx
  [BrokeredMessage]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx
  [QueueClient]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx
  [Complete]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx