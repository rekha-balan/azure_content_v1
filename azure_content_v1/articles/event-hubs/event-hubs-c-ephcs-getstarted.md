<properties
    pageTitle="Get Started with Event Hubs in C and C# | Microsoft Azure"
    description="Follow this tutorial to get started using Azure Event Hubs; sending events in C and receiving hem in C# using the EventProcessorHost."
    services="event-hubs"
    documentationCenter=""
    authors="fsautomata"
    manager="timlt"
    editor=""/>

<tags
    ms.service="event-hubs"
    ms.workload="na"
    ms.tgt_pltfrm="c"
    ms.devlang="csharp"
    ms.topic="article"
    ms.date="12/09/2015"
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

For more information, please see [Event Hubs Overview](event-hubs-overview.md).

In this tutorial, you will learn how to ingest messages into an Event Hub using a console application in C, and to retrieve them in parallel using the C# [Event Processor Host](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost) library.

In order to complete this tutorial you will need the following:

* A C development environment. For this tutorial, we will assume the gcc stack on an [Azure Linux VM](../virtual-machines/virtual-machines-linux-tutorial.md) with Ubuntu 14.04. Instructions for other environments will be provided in external links.

* Microsoft Visual Studio Express 2013 for Windows

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

In this section we will write a C app to send events to your Event Hub. We will use the Proton AMQP library from the [Apache Qpid project](http://qpid.apache.org/). This is analogous to using Service Bus queues and topics with AMQP from C as shown [here](https://code.msdn.microsoft.com/Using-Apache-Qpid-Proton-C-afd76504). For more information, see [Qpid Proton documentation](http://qpid.apache.org/proton/index.html).

1. From the [Qpid AMQP Messenger page](http://qpid.apache.org/components/messenger/index.html), click the **Installing Qpid Proton** link and follow the instructions depending on your environment. We will assume a Linux environment; for example, an [Azure Linux VM](../articles/virtual-machines/virtual-machines-linux-tutorial.md) with Ubuntu 14.04.

2. To compile the Proton library, install the following packages:

	```
	sudo apt-get install build-essential cmake uuid-dev openssl libssl-dev
	```

3. Download the [Qpid Proton library](http://qpid.apache.org/proton/index.html) library, and extract it, e.g.:

	```
	wget http://apache.fastbull.org/qpid/proton/0.7/qpid-proton-0.7.tar.gz
	tar xvfz qpid-proton-0.7.tar.gz
	```

4. Create a build directory, compile and install:

	```
	cd qpid-proton-0.7
	mkdir build
	cd build
	cmake -DCMAKE_INSTALL_PREFIX=/usr ..
	sudo make install
	```

5. In your work directory, create a new file called **sender.c** with the following content. Remember to substitute the value for your Event Hub name and namespace name (the latter is usually `{event hub name}-ns`). You must also substitute a URL-encoded version of the key for the **SendRule** created earlier. You can URL-encode it [here](http://www.w3schools.com/tags/ref_urlencode.asp).

	```
	#include "proton/message.h"
	#include "proton/messenger.h"
	
	#include <getopt.h>
	#include <proton/util.h>
	#include <sys/time.h>
	#include <stddef.h>
	#include <stdio.h>
	#include <string.h>
	#include <unistd.h>
	#include <stdlib.h>
	
	#define check(messenger)                                                     \
	  {                                                                          \
	    if(pn_messenger_errno(messenger))                                        \
	    {                                                                        \
	      printf("check\n");													 \
	      die(__FILE__, __LINE__, pn_error_text(pn_messenger_error(messenger))); \
	    }                                                                        \
	  }  
	
	pn_timestamp_t time_now(void)
	{
	  struct timeval now;
	  if (gettimeofday(&now, NULL)) pn_fatal("gettimeofday failed\n");
	  return ((pn_timestamp_t)now.tv_sec) * 1000 + (now.tv_usec / 1000);
	}  
	
	void die(const char *file, int line, const char *message)
	{
	  printf("Dead\n");
	  fprintf(stderr, "%s:%i: %s\n", file, line, message);
	  exit(1);
	}
	
	int sendMessage(pn_messenger_t * messenger) {
		char * address = (char *) "amqps://SendRule:{Send Rule key}@{namespace name}.servicebus.windows.net/{event hub name}";
		char * msgtext = (char *) "Hello from C!";
	
		pn_message_t * message;
		pn_data_t * body;
		message = pn_message();
	
		pn_message_set_address(message, address);
		pn_message_set_content_type(message, (char*) "application/octect-stream");
		pn_message_set_inferred(message, true);
	
		body = pn_message_body(message);
		pn_data_put_binary(body, pn_bytes(strlen(msgtext), msgtext));
	
		pn_messenger_put(messenger, message);
		check(messenger);
		pn_messenger_send(messenger, 1);
		check(messenger);
	
		pn_message_free(message);
	}
	
	int main(int argc, char** argv) {
		printf("Press Ctrl-C to stop the sender process\n");
	
		pn_messenger_t *messenger = pn_messenger(NULL);
		pn_messenger_set_outgoing_window(messenger, 1);
		pn_messenger_start(messenger);
	
		while(true) {
			sendMessage(messenger);
			printf("Sent message\n");
			sleep(1);
		}
	
		// release messenger resources
		pn_messenger_stop(messenger);
		pn_messenger_free(messenger);
	
		return 0;
	}
	```

6. Compile the file, assuming **gcc**:

	```
	gcc sender.c -o sender -lqpid-proton
	```

> [AZURE.NOTE] In this code, we use an outgoing window of 1 to force the messages out as soon as possible. In general, your application should try to batch messages to increase throughput. See [Qpid AMQP Messenger page](http://qpid.apache.org/components/messenger/index.html) for more information about how to use the Qpid Proton library in this and other environments, and from platforms for which bindings are provided (currently Perl, PHP, Python, and Ruby).


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

1. Run the **Receiver** project from within Visual Studio, then wait for it to start the receivers for all the partitions.

    ![][21]

2. Run the **Sender** program, and see the events appear in the receiver window.

    ![][24]


## Next steps
Now that you've built a working application that creates an Event Hub and sends and receives data, you can move on to the following scenarios:

* A complete [sample application that uses Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097).
* The [Scale out Event Processing with Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3) sample.
* A [queued messaging solution](../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md) using Service Bus queues.
* [Event Hubs overview](event-hubs-overview.md)

<!-- Images. -->

[1]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub1.png
[2]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub2.png
[3]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub3.png
[4]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub4.png
[5]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub5.png
[6]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub6.png
[6b]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub6b.png


[21]: ./media/event-hubs-c-ephcs-getstarted/run-csharp-ephcs1.png
[24]: ./media/event-hubs-c-ephcs-getstarted/receive-eph-c.png

<!-- Links -->

[Azure classic portal]: https://manage.windowsazure.com/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[Event Hubs overview]: event-hubs-overview.md
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[queued messaging solution]: ../service-bus/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md
