<properties 
    pageTitle="How to use Service Bus topics with Python | Microsoft Azure" 
    description="Learn how to use Azure Service Bus topics and subscriptions from Python." 
    services="service-bus" 
    documentationCenter="python" 
    authors="sethmanheim" 
    manager="timlt" 
    editor=""/>

<tags 
    ms.service="service-bus" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="python" 
    ms.topic="article" 
    ms.date="01/26/2016" 
    ms.author="sethm"/>

# How to use Service Bus topics and subscriptions
> [AZURE.SELECTOR]
- [NET](../articles/service-bus/service-bus-dotnet-how-to-use-topics-subscriptions.md)
- [Java](../articles/service-bus/service-bus-java-how-to-use-topics-subscriptions.md)
- [Node.js](../articles/service-bus/service-bus-nodejs-how-to-use-topics-subscriptions.md)
- [PHP](../articles/service-bus/service-bus-php-how-to-use-topics-subscriptions.md)
- [Python](../articles/service-bus/service-bus-python-how-to-use-topics-subscriptions.md)
- [Ruby](../articles/service-bus/service-bus-ruby-how-to-use-topics-subscriptions.md)


This article describes how to use Service Bus topics and subscriptions. The samples are written in Python and use the [Python Azure package](https://pypi.python.org/pypi/azure). The scenarios covered include **creating topics and subscriptions**, **creating subscription filters**, **sending messages to a topic**, **receiving messages from a subscription**, and **deleting topics and subscriptions**. For more information about topics and subscriptions, see the [Next Steps](#next-steps.md) section.

## What are Service Bus topics and subscriptions?

Service Bus topics and subscriptions support a *publish/subscribe* messaging communication model. When using topics and subscriptions, components of a distributed application do not communicate directly with each other; instead they exchange messages via a topic, which acts as an intermediary.

![TopicConcepts](./media/howto-service-bus-topics/sb-topics-01.png)

In contrast with Service Bus queues, in which each message is processed by a single consumer, topics and subscriptions provide a "one-to-many" form of communication, using a publish/subscribe pattern. It is possible to
register multiple subscriptions to a topic. When a message is sent to a topic, it is then made available to each subscription to handle/process independently.

A subscription to a topic resembles a virtual queue that receives copies of the messages that were sent to the topic. You can optionally register filter rules for a topic on a per-subscription basis, which allows you to filter/restrict which messages to a topic are received by which topic subscriptions.

Service Bus topics and subscriptions enable you to scale and process a very large number of messages across a very large number of users and applications.

## Create a service namespace

To begin using Service Bus topics and subscriptions in Azure, you must first create a service namespace. A namespace provides a scoping container for addressing Service Bus resources within your application.

To create a service namespace:

1.  Log on to the [Azure classic portal][].

2.  In the left navigation pane of the portal, click **Service Bus**.

3.  In the lower pane of the portal, click **Create**.   
    ![][0]

4.  In the **Add a new namespace** dialog, enter a namespace name. The system immediately checks to see if the name is available.   
    ![][2]

5.  After making sure the namespace name is available, choose the country or region in which your namespace should be hosted (make sure you use the same country/region in which you are deploying your compute resources).

	> [AZURE.IMPORTANT] Pick the **same region** that you intend to choose for deploying your application. This will give you the best performance.

6. 	Leave the other fields in the dialog with their default values (**Messaging** and **Standard Tier**), then click the check mark. The system now creates your service namespace and enables it. You might have to wait several minutes as the system provisions resources for your account.

	![][6]

## Obtain the default management credentials for the namespace

In order to perform management operations, such as creating a topic or subscription on the new namespace, you must obtain the management credentials for the namespace. You can obtain these credentials from the Azure portal.

### Obtain management credentials from the portal

1.  In the left navigation pane, click the **Service Bus** node to display the list of available namespaces:   
    ![][0]

2.  Select the namespace you just created from the list shown:   
    ![][3]

3.  Click **Connection Information**.   
    ![][4]

4.  In the **Access connection information** dialog, find the connection string that contains the SAS key and key name. Make a note of these values, as you will use this information later to perform operations with the namespace. 


  [Azure classic portal]: http://manage.windowsazure.com
  [0]: ./media/howto-service-bus-topics/sb-queues-13.png
  [2]: ./media/howto-service-bus-topics/sb-queues-04.png
  [3]: ./media/howto-service-bus-topics/sb-queues-09.png
  [4]: ./media/howto-service-bus-topics/sb-queues-06.png
  
  [6]: ./media/howto-service-bus-topics/getting-started-multi-tier-27.png



**Note:** If you need to install Python or the [Python Azure package](https://pypi.python.org/pypi/azure), please see the [Python Installation Guide](../python-how-to-install.md).

## Create a topic
The **ServiceBusService** object enables you to work with topics. Add the following near the top of any Python file in which you wish to programmatically access Service Bus:

```
from azure.servicebus import ServiceBusService, Message, Topic, Rule, DEFAULT_RULE_NAME
```

The following code creates a **ServiceBusService** object. Replace `mynamespace`, `sharedaccesskeyname`, and `sharedaccesskey` with your actual namespace, Shared Access Signature (SAS) key name, and key value.

```
bus_service = ServiceBusService(
    service_namespace='mynamespace',
    shared_access_key_name='sharedaccesskeyname',
    shared_access_key_value='sharedaccesskey')
```

You can obtain the values for the SAS key name and value from the [Azure classic portal](http://manage.windowsazure.com) **Connection Information** window.

```
bus_service.create_topic('mytopic')
```

**create\_topic** also supports additional options, which enable you to override default topic settings such as message time to live or maximum topic size. The following example sets the maximum topic size to 5 GB, and a time to live (TTL) value of 1 minute:

```
topic_options = Topic()
topic_options.max_size_in_megabytes = '5120'
topic_options.default_message_time_to_live = 'PT1M'

bus_service.create_topic('mytopic', topic_options)
```

## Create subscriptions
Subscriptions to topics are also created with the **ServiceBusService** object. Subscriptions are named and can have an optional filter that restricts the set of messages delivered to the subscription's virtual queue.

> [!NOTE]
> Subscriptions are persistent and will continue to exist until either they, or the topic to which they are subscribed, are deleted.
> 
> 
### Create a subscription with the default (MatchAll) filter
The **MatchAll** filter is the default filter that is used if no filter is specified when a new subscription is created. When the **MatchAll** filter is used, all messages published to the topic are placed in the subscription's virtual queue. The following example creates a subscription named 'AllMessages' and uses the default **MatchAll**
filter.

```
bus_service.create_subscription('mytopic', 'AllMessages')
```

### Create subscriptions with filters
You can also define filters that enable you to specify which messages sent to a topic should show up within a specific topic subscription.

The most flexible type of filter supported by subscriptions is a **SqlFilter**, which implements a subset of SQL92. SQL filters operate on the properties of the messages that are published to the topic. For more information about the expressions that can be used with a SQL filter, see the [SqlFilter.SqlExpression](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx) syntax.

You can add filters to a subscription by using the **create\_rule** method of the **ServiceBusService** object. This method allows you to add new filters to an existing subscription.

> [!NOTE]
> Because the default filter is applied automatically to all new subscriptions, you must first remove the default filter or the **MatchAll** will override any other filters you may specify. You can remove the default rule by using the **delete\_rule** method of the **ServiceBusService** object.
> 
> 
The following example creates a subscription named `HighMessages` with a **SqlFilter** that only selects messages that have a custom **messagenumber** property greater than 3:

```
bus_service.create_subscription('mytopic', 'HighMessages')

rule = Rule()
rule.filter_type = 'SqlFilter'
rule.filter_expression = 'messagenumber > 3'

bus_service.create_rule('mytopic', 'HighMessages', 'HighMessageFilter', rule)
bus_service.delete_rule('mytopic', 'HighMessages', DEFAULT_RULE_NAME)
```

Similarly, the following example creates a subscription named `LowMessages` with a **SqlFilter** that only selects messages that have a **messagenumber** property less than or equal to 3:

```
bus_service.create_subscription('mytopic', 'LowMessages')

rule = Rule()
rule.filter_type = 'SqlFilter'
rule.filter_expression = 'messagenumber <= 3'

bus_service.create_rule('mytopic', 'LowMessages', 'LowMessageFilter', rule)
bus_service.delete_rule('mytopic', 'LowMessages', DEFAULT_RULE_NAME)
```

Now, when a message is sent to `mytopic` it is always delivered to receivers subscribed to the **AllMessages** topic subscription, and selectively delivered to receivers subscribed to the **HighMessages** and **LowMessages** topic subscriptions (depending on the message content).

## Send messages to a topic
To send a message to a Service Bus topic, your application must use the **send\_topic\_message** method of the **ServiceBusService** object.

The following example demonstrates how to send five test messages to `mytopic`. Note that the **messagenumber** property value of each message varies on the iteration of the loop (this determines which subscriptions receive it):

```
for i in range(5):
    msg = Message('Msg {0}'.format(i).encode('utf-8'), custom_properties={'messagenumber':i})
    bus_service.send_topic_message('mytopic', msg)
```

Service Bus topics support a maximum message size of 256 MB (the header, which includes the standard and custom application properties, can have a maximum size of 64 MB). There is no limit on the number of messages held in a topic but there is a cap on the total size of the messages held by a topic. This topic size is defined at creation time, with an upper limit of 5 GB. For more information about quotas, see [Azure Queues and Service Bus queues](service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas).

## Receive messages from a subscription
Messages are received from a subscription using the **receive\_subscription\_message** method on the **ServiceBusService** object:

```
msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=False)
print(msg.body)
```

Messages are deleted from the subscription as they are read when the parameter **peek\_lock** is set to **False**. You can read (peek) and lock the message without deleting it from the queue by setting the parameter **peek\_lock** to **True**.

The behavior of reading and deleting the message as part of the receive operation is the simplest model, and works best for scenarios in which an application can tolerate not processing a message in the event of a failure. To understand this, consider a scenario in which the consumer issues the receive request and then crashes before processing it. Because Service Bus will have marked the message as being consumed, then when the application restarts and begins consuming messages again, it will have missed the message that was consumed prior to the crash.

If the **peek\_lock** parameter is set to **True**, the receive becomes a two stage operation, which makes it possible to support applications that cannot tolerate missing messages. When Service Bus receives a request, it finds the next message to be consumed, locks it to prevent other consumers receiving it, and then returns it to the application. After the application finishes processing the message (or stores it reliably for future processing), it completes the second stage of the receive process by calling **delete** method on the **Message** object. The **delete** method marks the message as being consumed and removes it from the subscription.

```
msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=True)
print(msg.body)

msg.delete()
```

## How to handle application crashes and unreadable messages
Service Bus provides functionality to help you gracefully recover from errors in your application or difficulties processing a message. If a receiver application is unable to process the message for some reason, then it can call the **unlock** method on the **Message** object. This will cause Service Bus to unlock the message within the subscription and make it available to be received again, either by the same consuming application or by another consuming application.

There is also a timeout associated with a message locked within the subscription, and if the application fails to process the message before the lock timeout expires (for example, if the application crashes), then Service Bus unlocks the message automatically and makes it available to be received again.

In the event that the application crashes after processing the message but before the **delete** method is called, then the message will be redelivered to the application when it restarts. This is often called **At Least Once Processing**, that is, each message will be processed at least once but in certain situations the same message may be redelivered. If the scenario cannot tolerate duplicate processing, then application developers should add additional logic to their application to handle duplicate message delivery. This is often achieved using the **MessageId** property of the message, which will remain constant across delivery attempts.

## Delete topics and subscriptions
Topics and subscriptions are persistent, and must be explicitly deleted either through the [Azure classic portal](http://manage.windowsazure.com) or programmatically. The following example shows how to delete the topic named `mytopic`:

```
bus_service.delete_topic('mytopic')
```

Deleting a topic also deletes any subscriptions that are registered with the topic. Subscriptions can also be deleted independently. The following code shows how to delete a subscription named `HighMessages` from the `mytopic` topic:

```
bus_service.delete_subscription('mytopic', 'HighMessages')
```

## Next steps
Now that you've learned the basics of Service Bus topics, follow these links to learn more.

* See [Queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md).
* Reference for [SqlFilter.SqlExpression](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx).

[Azure classic portal]: http://manage.windowsazure.com
[Python Azure package]: https://pypi.python.org/pypi/azure  
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[SqlFilter.SqlExpression]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx
[Azure Queues and Service Bus queues]: service-bus-azure-and-service-bus-queues-compared-contrasted.md#capacity-and-quotas 
