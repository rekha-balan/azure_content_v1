<properties
    pageTitle="Get Started with Event Hubs in C# with Apache Storm | Microsoft Azure"
    description="Follow this tutorial to get started using Azure Event Hubs; sending events in C# and receiving them in an Apache Storm cluster."
    services="event-hubs"
    documentationCenter=""
    authors="fsautomata"
    manager="timlt"
    editor=""/>

<tags
    ms.service="event-hubs"
    ms.workload="na"
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

For more information, please see the [Event Hubs overview](event-hubs-overview.md).

In this tutorial, you will learn how to ingest messages into an Event Hub using a console application in C#, and to retrieve them in parallel using Apache Storm.

In order to complete this tutorial you will need the following:

* Microsoft Visual Studio Express 2013 for Windows

* A Java development environment configured to run [Maven](http://maven.apache.org/). For this tutorial, we will assume [Eclipse](https://www.eclipse.org/)

* An active Azure account. <br/>If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure Free Trial</a>.


## Create an Event Hub
1. Log on to the [Azure classic portal](https://manage.windowsazure.com/), and click **NEW** at the bottom of the screen.

2. Click on **App Services**, then **Service Bus**, then **Event Hub**, then **Quick Create**.

    ![][1]

3. Type a name for your Event Hub, select your desired region, and then click **Create a new Event Hub**.

    ![][2]

4. Click the namespace you just created (usually ***event hub name*-ns**).

    ![][3]

5. Click the **Event Hubs** tab at the top of the page, and then click the Event Hub you just created.

    ![][4]

6. Click the **Configure** tab at the top, add a rule named **SendRule** with *Send* rights, add another rule called **ReceiveRule** with *Listen* rights, and then click **Save**.

    ![][5]

7. On the same page, take note of the generated keys for **ReceiveRule**.

    ![][6c]

8. Click the **Dashboard** tab at the top, and then click **Connection Information**. Take note of the two connection strings.

    ![][6]


Your Event Hub is now created, and you have the connection strings you need to send and receive events.

## Send messages to Event Hubs

In this section, you'll write a Windows console app that sends events to your Event Hub.

1. In Visual Studio, create a new Visual C# Desktop App project using the **Console  Application** project template. Name the project **Sender**.

   	![][7]

2. In Solution Explorer, right-click the solution, and then click **Manage NuGet Packages for Solution...**. 

	This displays the Manage NuGet Packages dialog box.

3. Search for `Microsoft Azure Service Bus`, click **Install**, and accept the terms of use. 

	![][8]

	This downloads, installs, and adds a reference to the <a href="https://www.nuget.org/packages/WindowsAzure.ServiceBus/">Azure Service Bus library NuGet package</a>.

4. Add the following `using` statement at the top of the **Program.cs** file:

		using System.Threading;
		using Microsoft.ServiceBus.Messaging;

5. Add the following fields to the **Program** class, substituting the placeholder values with the name of the Event Hub you created in the previous section, and the connection string with **Send** rights:

		static string eventHubName = "{event hub name}";
        static string connectionString = "{send connection string}";

6. Add the following method to the **Program** class:

		static void SendingRandomMessages()
        {
            var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
            while (true)
            {
                try
                {
                    var message = Guid.NewGuid().ToString();
                    Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, message);
                    eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes(message)));
                }
                catch (Exception exception)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine("{0} > Exception: {1}", DateTime.Now, exception.Message);
                    Console.ResetColor();
                }

                Thread.Sleep(200);
            }
        }

	This method continuously sends events to your Event Hub with a 200ms delay.

7. Finally, add the following lines to the **Main** method:

		Console.WriteLine("Press Ctrl-C to stop the sender process");
        Console.WriteLine("Press Enter to start now");
        Console.ReadLine();
        SendingRandomMessages();


<!-- Images -->
[7]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp1.png
[8]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp2.png

## Receive messages with Apache Storm

[**Apache Storm**](https://storm.incubator.apache.org) is a distributed real time computation system that simplifies reliable processing of unbounded streams of data. This section shows how to use an Event Hubs Storm spout to receive events from Event Hubs. Using Apache Storm, you can split events across multiple processes hosted in different nodes. The Event Hubs integration with Storm simplifies event consumption by transparently checkpointing its progress using Storm's Zookeeper installation, managing persistent checkpoints and parallel receives from Event Hubs.

For more information about Event Hubs receive patterns, see the [Event Hubs overview][].

This tutorial uses an [HDInsight Storm][] installation, which comes with the Event Hubs spout already available.

1. Follow the [HDInsight Storm - Get Started](../hdinsight/hdinsight-storm-overview.md) procedure to create a new HDInsight cluster, and connect to it via Remote Desktop.

2. Copy the `%STORM_HOME%\examples\eventhubspout\eventhubs-storm-spout-0.9-jar-with-dependencies.jar` file to your local development environment. This contains the events-storm-spout.

3. Use the following command to install the package into the local Maven store. This enables you to add it as a reference in the Storm project in a later step.

		mvn install:install-file -Dfile=target\eventhubs-storm-spout-0.9-jar-with-dependencies.jar -DgroupId=com.microsoft.eventhubs -DartifactId=eventhubs-storm-spout -Dversion=0.9 -Dpackaging=jar

4. In Eclipse, create a new Maven project (click **File**, then **New**, then **Project**).

   ![][12]

5. Select **Use default Workspace location**, then click **Next**

6. Select the **maven-archetype-quickstart** archetype, then click **Next**

7. Insert a **GroupId** and **ArtifactId**, then click **Finish**

8. In **pom.xml**, add the following dependencies in the `<dependency>` node.

		<dependency>
			<groupId>org.apache.storm</groupId>
			<artifactId>storm-core</artifactId>
			<version>0.9.2-incubating</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.microsoft.eventhubs</groupId>
			<artifactId>eventhubs-storm-spout</artifactId>
			<version>0.9</version>
		</dependency>
		<dependency>
			<groupId>com.netflix.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>1.3.3</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-log4j12</artifactId>
				</exclusion>
			</exclusions>
			<scope>provided</scope>
		</dependency>

9. In the **src** folder, create a file called **Config.properties** and copy the following content, substituting the following values:

		eventhubspout.username = ReceiveRule

		eventhubspout.password = {receive rule key}

		eventhubspout.namespace = ioteventhub-ns

		eventhubspout.entitypath = {event hub name}

		eventhubspout.partitions.count = 16

		# if not provided, will use storm's zookeeper settings
		# zookeeper.connectionstring=localhost:2181

		eventhubspout.checkpoint.interval = 10

		eventhub.receiver.credits = 10

	The value for **eventhub.receiver.credits** determines how many events are batched before releasing them to the Storm pipeline. For the sake of simplicity, this example sets this value to 10. In production, it should usually be set to higher values; for example, 1024.

10. Create a new class called **LoggerBolt** with the following code:

		import java.util.Map;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import backtype.storm.task.OutputCollector;
		import backtype.storm.task.TopologyContext;
		import backtype.storm.topology.OutputFieldsDeclarer;
		import backtype.storm.topology.base.BaseRichBolt;
		import backtype.storm.tuple.Tuple;

		public class LoggerBolt extends BaseRichBolt {
			private OutputCollector collector;
			private static final Logger logger = LoggerFactory
				      .getLogger(LoggerBolt.class);

			@Override
			public void execute(Tuple tuple) {
				String value = tuple.getString(0);
				logger.info("Tuple value: " + value);

				collector.ack(tuple);
			}

			@Override
			public void prepare(Map map, TopologyContext context, OutputCollector collector) {
				this.collector = collector;
				this.count = 0;
			}

			@Override
			public void declareOutputFields(OutputFieldsDeclarer declarer) {
				// no output fields
			}

		}

	This Storm bolt logs the content of the received events. This can easily be extended to store tuples in a storage service. The [HDInsight sensor analysis tutorial] uses this same approach to store data into HBase.

11. Create a class called **LogTopology** with the following code:

		import java.io.FileReader;
		import java.util.Properties;
		import backtype.storm.Config;
		import backtype.storm.LocalCluster;
		import backtype.storm.StormSubmitter;
		import backtype.storm.generated.StormTopology;
		import backtype.storm.topology.TopologyBuilder;
		import com.microsoft.eventhubs.samples.EventCount;
		import com.microsoft.eventhubs.spout.EventHubSpout;
		import com.microsoft.eventhubs.spout.EventHubSpoutConfig;

		public class LogTopology {
			protected EventHubSpoutConfig spoutConfig;
			protected int numWorkers;

			protected void readEHConfig(String[] args) throws Exception {
				Properties properties = new Properties();
				if (args.length > 1) {
					properties.load(new FileReader(args[1]));
				} else {
					properties.load(EventCount.class.getClassLoader()
							.getResourceAsStream("Config.properties"));
				}

				String username = properties.getProperty("eventhubspout.username");
				String password = properties.getProperty("eventhubspout.password");
				String namespaceName = properties
						.getProperty("eventhubspout.namespace");
				String entityPath = properties.getProperty("eventhubspout.entitypath");
				String zkEndpointAddress = properties
						.getProperty("zookeeper.connectionstring"); // opt
				int partitionCount = Integer.parseInt(properties
						.getProperty("eventhubspout.partitions.count"));
				int checkpointIntervalInSeconds = Integer.parseInt(properties
						.getProperty("eventhubspout.checkpoint.interval"));
				int receiverCredits = Integer.parseInt(properties
						.getProperty("eventhub.receiver.credits")); // prefetch count
																	// (opt)
				System.out.println("Eventhub spout config: ");
				System.out.println("  partition count: " + partitionCount);
				System.out.println("  checkpoint interval: "
						+ checkpointIntervalInSeconds);
				System.out.println("  receiver credits: " + receiverCredits);

				spoutConfig = new EventHubSpoutConfig(username, password,
						namespaceName, entityPath, partitionCount, zkEndpointAddress,
						checkpointIntervalInSeconds, receiverCredits);

				// set the number of workers to be the same as partition number.
				// the idea is to have a spout and a logger bolt co-exist in one
				// worker to avoid shuffling messages across workers in storm cluster.
				numWorkers = spoutConfig.getPartitionCount();

				if (args.length > 0) {
					// set topology name so that sample Trident topology can use it as
					// stream name.
					spoutConfig.setTopologyName(args[0]);
				}
			}

			protected StormTopology buildTopology() {
				TopologyBuilder topologyBuilder = new TopologyBuilder();

				EventHubSpout eventHubSpout = new EventHubSpout(spoutConfig);
				topologyBuilder.setSpout("EventHubsSpout", eventHubSpout,
						spoutConfig.getPartitionCount()).setNumTasks(
						spoutConfig.getPartitionCount());
				topologyBuilder
						.setBolt("LoggerBolt", new LoggerBolt(),
								spoutConfig.getPartitionCount())
						.localOrShuffleGrouping("EventHubsSpout")
						.setNumTasks(spoutConfig.getPartitionCount());
				return topologyBuilder.createTopology();
			}

			protected void runScenario(String[] args) throws Exception {
				boolean runLocal = true;
				readEHConfig(args);
				StormTopology topology = buildTopology();
				Config config = new Config();
				config.setDebug(false);

				if (runLocal) {
					config.setMaxTaskParallelism(2);
					LocalCluster localCluster = new LocalCluster();
					localCluster.submitTopology("test", config, topology);
					Thread.sleep(5000000);
					localCluster.shutdown();
				} else {
					config.setNumWorkers(numWorkers);
				    StormSubmitter.submitTopology(args[0], config, topology);
				}
			}

			public static void main(String[] args) throws Exception {
				LogTopology topology = new LogTopology();
				topology.runScenario(args);
			}
		}


	This class creates a new Event Hubs spout, using the properties in the configuration file to instatiate it. It is important to note that this example creates as many spouts tasks as the number of partitions in the Event Hub, in order to use the maximum parallelism allowed by that Event Hub.

<!-- Links -->
[Event Hubs overview]: event-hubs-overview.md
[HDInsight Storm]: ../hdinsight/hdinsight-storm-overview.md
[HDInsight sensor analysis tutorial]: ../hdinsight/hdinsight-storm-sensor-data-analysis.md

<!-- Images -->

[12]: ./media/service-bus-event-hubs-getstarted/create-storm1.png
[13]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp1.png
[14]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp1.png


## Run the applications
Now you are ready to run the applications.

1. Run the **LogTopology** class from Eclipse, then wait for it to start the receivers for all the partitions.

2. Run the **Sender** project, press **Enter** in the console window, and see the events appear in the receiver window.

   ![][22]


## Next steps
Now that you've built a working application that creates an Event Hub and sends and receives data, you can move on to the following scenarios:

* A complete [sample application that uses Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097).
* The [Scale out Event Processing with Event Hubs](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3) sample.
* A [queued messaging solution](../service-bus-dotnet-multi-tier-app-using-service-bus-queues.md) using Service Bus queues.

<!-- Images. -->

[1]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub1.png
[2]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub2.png
[3]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub3.png
[4]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub4.png
[5]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub5.png
[6]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub6.png
[6c]: ./media/event-hubs-csharp-storm-getstarted/create-event-hub6c.png

[22]: ./media/event-hubs-csharp-storm-getstarted/receive-storm1.png

<!-- Links -->

[Azure classic portal]: https://manage.windowsazure.com/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[Event Hubs overview]: event-hubs-overview.md
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[queued messaging solution]: ../service-bus-dotnet-multi-tier-app-using-service-bus-queues.md
