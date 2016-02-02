<properties
    pageTitle="Get Started with Event Hubs with C and Apache Storm | Microsoft Azure"
    description="Follow this tutorial to get started using Azure Event Hubs; sending events in C and receiving them in an Apache Storm cluster."
    services="event-hubs"
    documentationCenter=""
    authors="fsautomata"
    manager="timlt"
    editor=""/>

<tags
    ms.service="event-hubs"
    ms.workload="na"
    ms.tgt_pltfrm="c"
    ms.devlang="java"
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

For more information, please see [Event Hubs overview](event-hubs-overview.md).

In this tutorial, you will learn how to ingest messages into an Event Hub using a console application in C, and to retrieve them in parallel using Apache Storm.

In order to complete this tutorial you will need the following:

* A C development environment. For this tutorial, we will assume the gcc stack on an [Azure Linux VM](../virtual-machines/virtual-machines-linux-tutorial.md) with Ubuntu 14.04. Instructions for other environments will be provided in external links.

* A Java development environment configured to run [Maven](http://maven.apache.org/). For this tutorial, we will assume [Eclipse](https://www.eclipse.org/).

* An active Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/).


## Create an Event Hub
1. Log on to the [Azure classic portal](https://manage.windowsazure.com/), and click **NEW** at the bottom of the screen.

2. Click **App Services**, then **Service Bus**, then **Event Hub**, and then **Quick Create**.

    ![][1]

3. Type a name for your Event Hub, select your desired region, and then click **Create a new Event Hub**.

    ![][2]

4. Click the namespace you just created (usually ***event hub name*-ns**).

    ![][3]

5. Click the **Event Hubs** tab at the top of the page, and then click the Event Hub you just created.

    ![][4]

6. Click the **Configure** tab at the top of the page, add a rule called **SendRule** with *Send* rights, add another rule called **ReceiveRule** with *Listen* rights, and then click **Save**.

    ![][5]

7. On the same page, take note of the generated keys for **SendRule** and **ReceiveRule**.

    ![][6c]


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

2. Run the **sender** program, and see the events appear in the receiver window.

   ![][23]


> [!NOTE]
> In this tutorial only, use Storm in local mode for development purposes. Refer to the [HDInsight Storm overview](../hdinsight/hdinsight-storm-overview.md/.md) and the official [Apache Storm](https://storm.incubator.apache.org) documentation for more information of Storm deployments and patterns.
> 
> 
## Next steps
The following resources are available for developing applications integrating Event Hubs and Storm.

* [Analyzing sensor data with Storm and HDInsight](../hdinsight/hdinsight-storm-sensor-data-analysis.md) is a full scenario tutorial using Event Hubs, Storm, and HBase to ingest sensor data in an Hadoop cluster.
* [Develop streaming data processing applications with SCP.NET and C# on Storm and HDInsight](../hdinsight/hdinsight-storm-develop-csharp-visual-studio-topology.md) is a tutorial on how to write Storm pipelines using C#.

<!-- Images. -->

[1]: ./media/event-hubs-c-storm-getstarted/create-event-hub1.png
[2]: ./media/event-hubs-c-storm-getstarted/create-event-hub2.png
[3]: ./media/event-hubs-c-storm-getstarted/create-event-hub3.png
[4]: ./media/event-hubs-c-storm-getstarted/create-event-hub4.png
[5]: ./media/event-hubs-c-storm-getstarted/create-event-hub5.png
[6]: ./media/event-hubs-getstarted/create-event-hub6.png
[6c]: ./media/event-hubs-c-storm-getstarted/create-event-hub6c.png

[23]: ./media/event-hubs-c-storm-getstarted/receive-storm3.png

<!-- Links -->

[Azure classic portal]: https://manage.windowsazure.com/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[Event Hubs overview]: event-hubs-overview.md

[Apache Storm]: https://storm.incubator.apache.org
[HDInsight Storm overview]: ../hdinsight/hdinsight-storm-overview.md/
[Analyzing sensor data with Storm and HDInsight]: ../hdinsight/hdinsight-storm-sensor-data-analysis.md
[Develop streaming data processing applications with SCP.NET and C# on Storm and HDInsight]: ../hdinsight/hdinsight-storm-develop-csharp-visual-studio-topology.md
