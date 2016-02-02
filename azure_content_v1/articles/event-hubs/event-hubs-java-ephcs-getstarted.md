<properties
    pageTitle="Get Started with Event Hubs in Java | Microsoft Azure"
    description="Follow this tutorial to get started using Azure Event Hubs; sending events with Java and receiving them in C# using the EventProcessorHost."
    services="event-hubs"
    documentationCenter=""
    authors="fsautomata"
    manager="timlt"
    editor=""/>

<tags
    ms.service="event-hubs"
    ms.workload="core"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/05/2015"
    ms.author="sethm"/>

# Get started with Event Hubs
> [AZURE.SELECTOR-LIST (Language | Backend )]
- [(C# | EventProcessorHost C#)](../articles/service-bus-event-hubs-csharp-ephcs-getstarted.md)
- [(C# | Apache Storm)](../articles/service-bus-event-hubs-csharp-storm-getstarted.md)
- [(Java | EventProcessorHost C#)](../articles/service-bus-event-hubs-java-ephcs-getstarted.md)
- [(Java | Apache Storm)](../articles/service-bus-event-hubs-java-storm-getstarted.md)
- [(C | EventProcessorHost C#)](../articles/service-bus-event-hubs-c-ephcs-getstarted.md)
- [(C | Apache Storm)](../articles/service-bus-event-hubs-c-storm-getstarted.md)

## Introduction
Event Hubs is a highly scalable ingestion system that can intake millions of events per second, enabling an application to process and analyze the massive amounts of data produced by your connected devices and applications. Once collected into Event Hubs, you can transform and store data using any real-time analytics provider or storage cluster.

For more information, see the [Event Hubs overview](event-hubs-overview.md).

This tutorial shows how to ingest messages into an Event Hub using a console application in Java, and to retrieve them in parallel using the C# [Event Processor Host](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost) library.

In order to complete this tutorial, you will need the following:

* A Java development environment. For this tutorial, we will assume [Eclipse](https://www.eclipse.org/).

* Microsoft Visual Studio Express for Windows

* An active Azure account. <br/>If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure Free Trial</a>.


## Create an Event Hub
1. Log on to the [Azure classic portal](https://manage.windowsazure.com/), and click **NEW** at the bottom of the screen.

2. Click **App Services**, then **Service Bus**, then **Event Hub**, then **Quick Create**.

    ![][1]

3. Type a name for your Event Hub, select your desired region, and then click **Create a new Event Hub**.

    ![][2]

4. Click the namespace you just created (usually ***event hub name*-ns**).

    ![][3]

5. Click the **Event Hubs** tab at the top of the page, and then click the Event Hub you just created.

    ![][4]

6. Click the **Configure** tab at the top of the page, add a rule named **SendRule** with *Send* rights, add another rule called **ReceiveRule** with *Manage, Send, Listen* rights, and then click **Save**.

    ![][5]

7. On the same page, take note of the generated keys for **SendRule**.

    ![][6b]

8. Click the **Dashboard** tab at the top of the page, and then click **Connection Information**. Take note of the two connection strings.

    ![][6]


Your Event Hub is now created, and you have the connection strings you need to send and receive events.

## Send messages to Event Hubs

In this section, we write a Java console app to send events to your Event Hub. We make use of the JMS AMQP provider from the [Apache Qpid project](http://qpid.apache.org/). This is analogous to using Service Bus queues and topics with AMQP through Java, as shown [here](../service-bus/service-bus-java-how-to-use-jms-api-amqp.md). For more information, see the [Qpid JMS documentation](http://qpid.apache.org/releases/qpid-0.30/programming/book/QpidJMS.html) and [Java Messaging Service](http://www.oracle.com/technetwork/java/jms/index.html).

1. In Eclipse, install the [Azure Toolkit for Eclipse](https://msdn.microsoft.com/library/azure/hh690946.aspx). This includes the Qpid JMS AMQP client libraries.

2. In Eclipse, create a new Java project named **Sender**.

3. In Eclipse Package Explorer, right-click the **Sender** project and select **Properties**. In the left pane of the dialog, click **Java Build Path**, then click the **Libraries** tab, and then the **Add Library** button. Select **Package for Apache Qpid Client Libraries for JMS (by MS Open Tech)**, click **Next**, and then click **Finish**.

	![][8]

4. Create a file named **servicebus.properties** in the root of the **Sender** project, with the following content. Remember to substitute the values of your:
	- Event Hub name.
	- Namespace name (the latter is usually `{event hub name}-ns`).
	- URL-encoded **SendRule** key (you made a note of this key when you created your Event Hub). You can URL-encode it [here](http://www.w3schools.com/tags/ref_urlencode.asp).

			# servicebus.properties - sample JNDI configuration

			# Register a ConnectionFactory in JNDI using the form:
			# connectionfactory.[jndi_name] = [ConnectionURL]
			connectionfactory.SBCF = amqps://SendRule:{Send Rule key}@{namespace name}.servicebus.windows.net/?sync-publish=false

			# Register some queues in JNDI using the form
			# queue.[jndi_name] = [physical_name]
			# topic.[jndi_name] = [physical_name]
			queue.EventHub = {event hub name}

5. Create a new class named **Sender**. Add the following `import` statements:

		import java.io.BufferedReader;
		import java.io.IOException;
		import java.io.InputStreamReader;
		import java.io.UnsupportedEncodingException;
		import java.util.Hashtable;

		import javax.jms.BytesMessage;
		import javax.jms.Connection;
		import javax.jms.ConnectionFactory;
		import javax.jms.Destination;
		import javax.jms.JMSException;
		import javax.jms.MessageProducer;
		import javax.jms.Session;
		import javax.naming.Context;
		import javax.naming.InitialContext;
		import javax.naming.NamingException;

6. Then, add the following code to it:

		public static void main(String[] args) throws NamingException,
				JMSException, IOException, InterruptedException {
			// Configure JNDI environment
			Hashtable<String, String> env = new Hashtable<String, String>();
			env.put(Context.INITIAL_CONTEXT_FACTORY,
					"org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory");
			env.put(Context.PROVIDER_URL, "servicebus.properties");
			Context context = new InitialContext(env);

			ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");

			Destination queue = (Destination) context.lookup("EventHub");

			// Create Connection
			Connection connection = cf.createConnection();

			// Create sender-side Session and MessageProducer
			Session sendSession = connection.createSession(false,
					Session.AUTO_ACKNOWLEDGE);
			MessageProducer sender = sendSession.createProducer(queue);

			System.out.println("Press Ctrl-C to stop the sender process");
			System.out.println("Press Enter to start now");
			BufferedReader commandLine = new java.io.BufferedReader(
					new InputStreamReader(System.in));
			commandLine.readLine();

			while (true) {
				sendBytesMessage(sendSession, sender);
				Thread.sleep(200);
			}
		}

		private static void sendBytesMessage(Session sendSession, MessageProducer sender) throws JMSException, UnsupportedEncodingException {
	        BytesMessage message = sendSession.createBytesMessage();
	        message.writeBytes("Test AMQP message from JMS".getBytes("UTF-8"));
	        sender.send(message);
	        System.out.println("Sent message");
	    }



<!-- Images -->
[8]: ./media/service-bus-event-hubs-getstarted/create-sender-java1.png


## Receive messages with EventProcessorHost

[EventProcessorHost][] is a .NET class that simplifies receiving events from Event Hubs by managing persistent checkpoints and parallel receives from those Event Hubs. Using [EventProcessorHost][], you can split events across multiple receivers, even when hosted in different nodes. This example shows how to use [EventProcessorHost][] for a single receiver. The [Scaled out event processing][] sample shows how to use [EventProcessorHost][] with multiple receivers.

In order to use [EventProcessorHost][], you must have an [Azure Storage account][]:

1. Log on to the [Azure classic portal][], and click **NEW** at the bottom of the screen.

2. Click **Data Services**, then **Storage**, then **Quick Create**, and then type a name for your storage account. Select your desired region, and then click **Create Storage Account**.

    ![][11]

3. Click the newly created storage account, and then click **Manage Access Keys**:

    ![][12]

    Copy the access key to use later in this tutorial.

4. In Visual Studio, create a new Visual C# Desktop App project using the **Console  Application** project template. Name the project **Receiver**.

    ![][14]

5. In Solution Explorer, right-click the solution, and then click **Manage NuGet Packages**.

	The **Manage NuGet Packages** dialog box appears.

6. Search for `Microsoft Azure Service Bus Event Hub - EventProcessorHost`, click **Install**, and accept the terms of use.

    ![][13]

	This downloads, installs, and adds a reference to the [Azure Service Bus Event Hub - EventProcessorHost NuGet package](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost), with all its dependencies.

7. Right-click the **Receiver** project, click **Add**, and then click **Class**. Name the new class **SimpleEventProcessor**, and then click **OK** to create the class.

8. Add the following statements at the top of the SimpleEventProcessor.cs file:

	```
	using Microsoft.ServiceBus.Messaging;
	using System.Diagnostics;
	using System.Threading.Tasks;
	```

	Then, substitute the following code for the body of the class:

	```
    class SimpleEventProcessor : IEventProcessor
	{
	    Stopwatch checkpointStopWatch;

	    async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
	    {
	        Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
	        if (reason == CloseReason.Shutdown)
	        {
	            await context.CheckpointAsync();
	        }
	    }

	    Task IEventProcessor.OpenAsync(PartitionContext context)
	    {
	        Console.WriteLine("SimpleEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
	        this.checkpointStopWatch = new Stopwatch();
	        this.checkpointStopWatch.Start();
	        return Task.FromResult<object>(null);
	    }

	    async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
	    {
	        foreach (EventData eventData in messages)
	        {
	            string data = Encoding.UTF8.GetString(eventData.GetBytes());

	            Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
	                context.Lease.PartitionId, data));
	        }

	        //Call checkpoint every 5 minutes, so that worker can resume processing from 5 minutes back if it restarts.
	        if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
            {
                await context.CheckpointAsync();
                this.checkpointStopWatch.Restart();
            }
	    }
	}
    ````

	This class will be called by the **EventProcessorHost** to process events received from the Event Hub. Note that the `SimpleEventProcessor` class uses a stopwatch to periodically call the checkpoint method on the **EventProcessorHost** context. This ensures that, if the receiver is restarted, it will lose no more than five minutes of processing work.

9. In the **Program** class, add the following `using` statements at the top of the file:

	```
	using Microsoft.ServiceBus.Messaging;
	using Microsoft.Threading;
	using System.Threading.Tasks;
	```

	Then, modify the `Main` method in the `Program` class as follows, substituting the Event Hub name and connection string, and the storage account and key that you copied in the previous sections:

    ```
	static void Main(string[] args)
    {
      string eventHubConnectionString = "{event hub connection string}";
      string eventHubName = "{event hub name}";
      string storageAccountName = "{storage account name}";
      string storageAccountKey = "{storage account key}";
      string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", storageAccountName, storageAccountKey);

      string eventProcessorHostName = Guid.NewGuid().ToString();
      EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
      Console.WriteLine("Registering EventProcessor...");
      eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>().Wait();

      Console.WriteLine("Receiving. Press enter key to stop worker.");
      Console.ReadLine();
      eventProcessorHost.UnregisterEventProcessorAsync().Wait();
    }
	````

> [AZURE.NOTE] This tutorial uses a single instance of [EventProcessorHost][]. To increase throughput, it is recommended that you run multiple instances of [EventProcessorHost][], as shown in the [Scaled out event processing][] sample. In those cases, the various instances automatically coordinate with each other in order to load balance the received events. If you want multiple receivers to each process *all* the events, you must use the **ConsumerGroup** concept. When receiving events from different machines, it might be useful to specify names for [EventProcessorHost][] instances based on the machines (or roles) in which they are deployed. For more information about these topics, see the [Event Hubs Overview][] and [Event Hubs Programming Guide][] topics.

<!-- Links -->
[Event Hubs Overview]: event-hubs-overview.md
[Event Hubs Programming Guide]: event-hubs-programming-guide.md
[Scaled out event processing]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[Azure Storage account]: ../storage/storage-create-storage-account.md
[EventProcessorHost]: http://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[Azure classic portal]: http://manage.windowsazure.com

<!-- Images -->

[11]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp2.png
[12]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp3.png
[13]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp1.png
[14]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp1.png



## Run the applications
Now you are ready to run the applications.

1. Run the **Receiver** project from Visual Studio, then wait for it to start the receivers for all the partitions.

   ![][21]

2. Run the **Sender** project, press **Enter** in the console window, and see the events appear in the receiver window.

   ![][22]


## Next steps
Now that you've built a working application that creates an Event Hub and sends and receives data, you can move on to the following scenarios:

* A complete [sample application that uses Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097).
* The [Scale out Event Processing with Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3) sample.
* A [queued messaging solution](../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md) using Service Bus queues.

For more information, see the [Java Developer Center](/develop/java/).

<!-- Images. -->

[1]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub1.png
[2]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub2.png
[3]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub3.png
[4]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub4.png
[5]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub5.png
[6]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub6.png
[6b]: ./media/event-hubs-java-ephcs-getstarted/create-event-hub6b.png


[21]: ./media/event-hubs-java-ephcs-getstarted/run-csharp-ephcs1.png
[22]: ./media/event-hubs-java-ephcs-getstarted/run-csharp-ephcs2.png

<!-- Links -->

[Azure classic portal]: https://manage.windowsazure.com/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[Event Hubs overview]: event-hubs-overview.md
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[queued messaging solution]: ../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md
