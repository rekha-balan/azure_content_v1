<properties
    pageTitle="Get started with Azure IoT Hub for C# | Microsoft Azure"
    description="Follow this tutorial to get started using Azure IoT Hub with C#."
    services="iot-hub"
    documentationCenter=".net"
    authors="dominicbetts"
    manager="timlt"
    editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="dotnet"
     ms.topic="hero-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="12/14/2015"
     ms.author="dobett"/>

# Get started with Azure IoT Hub for .NET
> [AZURE.SELECTOR]
- [C#](../articles/iot-hub-csharp-csharp-getstarted.md)
- [Java](../articles/iot-hub-java-java-getstarted.md)
- [Node.js](../articles/iot-hub-node-node-getstarted.md)

## Introduction
Azure IoT Hub is a fully managed service that enables reliable and secure bi-directional communications between millions of IoT devices and a solution backend. One of the biggest challenges IoT projects face is how to reliably and securely connect devices to the solution backend. To address this challenge, IoT Hub:

* Offers reliable device-to-cloud and cloud-to-device hyper-scale messaging.
* Enables secure communications using per-device security credentials and access control.
* Includes device libraries for the most popular languages and platforms.

This tutorial shows you how to:

* Use the Azure portal to create an IoT hub.
* Create a device identity in your IoT hub.
* Create a simulated device that sends telemetry to your cloud backend, and receives commands from your cloud backend.

At the end of this tutorial you will have three Windows console applications:

* **CreateDeviceIdentity**, which creates a device identity and associated security key to connect your simulated device.
* **ReadDeviceToCloudMessages**, which displays the telemetry sent by your simulated device.
* **SimulatedDevice**, which connects to your IoT hub with the device identity created earlier, and sends a telemetry message every second.

> [!NOTE]
> The article [IoT Hub SDKs](iot-hub-sdks-summary.md) provides information about the various SDKs that you can use to build both applications to run on devices and your solution backend.
> 
> 
To complete this tutorial you'll need the following:

* Microsoft Visual Studio 2015.

* An active Azure account. <br/>If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](http://azure.microsoft.com/pricing/free-trial/).


## Create an IoT hub
You need to create an IoT Hub for you simulated device to connect to. The following steps show you how to complete this task using the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com/).

2. In the Jumpbar, click **New**, then click **Internet of Things**, and then click **Azure IoT Hub**.

    ![][1]

3. In the **IoT hub** blade, choose the configuration for your IoT hub.

    ![][2]

   * In the **Name** box, enter a name for your IoT hub. If the **Name** is valid and available, a green check mark appears in the **Name** box.
* Select a **Pricing and scale tier**. This tutorial does not require a specific tier.
* In **Resource group**, create a new resource group, or select an existing one. For more information, see [Using resource groups to manage your Azure resources](resource-group-portal.md).
* In **Location**, select the location to host your IoT hub.  

4. When you have chosen your IoT hub configuration options, click **Create**.  It can take a few minutes for Azure to create your IoT hub. To check the status, you can monitor the progress on the Startboard or in the Notifications panel.

    ![][3]

5. When the IoT hub has been created successfully, open the blade of the new IoT hub, make a note of the **Hostname**, and then click the **Keys** icon.

    ![][4]

6. Click the **iothubowner** policy, then copy and make note of the connection string in the **iothubowner** blade.

    ![][5]


You have now created your IoT hub and you have the hostname and connection string you need to complete the rest of this tutorial.

## Create a device identity

In this section, you'll create a Windows console app that creates a new device identity in the identity registry in your IoT hub. A device cannot connect to IoT hub unless it has an entry in the device identity registry. Refer to the **Device Identity Registry** section of the [IoT Hub Developer Guide][lnk-devguide-identity] for more information. When you run this console application, it generates a unique device ID and key that your device can identify itself with when it sends device-to-cloud messages to IoT Hub.

1. In Visual Studio, add a new Visual C# Windows Classic Desktop project to the current solution using the **Console  Application** project template. Name the project **CreateDeviceIdentity**.

	![][10]

2. In Solution Explorer, right-click the **CreateDeviceIdentity** project, and then click **Manage NuGet Packages**.

3. In the **NuGet Package Manager** window, make sure the **Include prerelease** option is checked. Then search for **Microsoft Azure Devices**, click **Install** to install the **Microsoft.Azure.Devices** package, and accept the terms of use.

	![][11]

4. This downloads, installs, and adds a reference to the [Microsoft Azure IoT Service SDK][lnk-nuget-service-sdk] NuGet package.

4. Add the following `using` statements at the top of the **Program.cs** file:

		using Microsoft.Azure.Devices;
        using Microsoft.Azure.Devices.Common.Exceptions;

5. Add the following fields to the **Program** class, replacing the placeholder value with the connection string for the IoT hub you created in the previous section:

		static RegistryManager registryManager;
        static string connectionString = "{iothub connection string}";

6. Add the following method to the **Program** class:

		private async static Task AddDeviceAsync()
        {
            string deviceId = "myFirstDevice";
            Device device;
            try
            {
                device = await registryManager.AddDeviceAsync(new Device(deviceId));
            }
            catch (DeviceAlreadyExistsException)
            {
                device = await registryManager.GetDeviceAsync(deviceId);
            }
            Console.WriteLine("Generated device key: {0}", device.Authentication.SymmetricKey.PrimaryKey);
        }

	This method creates a new device identity with ID **myFirstDevice** (if that device ID already exists in the registry, the code simply retrieves the existing device information). The app then displays the primary key for that identity. You will use this key in the simulated device to connect to your IoT hub.

7. Finally, add the following lines to the **Main** method:

		registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        AddDeviceAsync().Wait();
        Console.ReadLine();

8. Run this application, and make a note of the device key.

    ![][12]

> [AZURE.NOTE] The IoT Hub identity registry only stores device identities to enable secure access to the hub. It stores device IDs and keys to use as security credentials and an enabled/disabled flag that enables you to disable access for an individual device. If you application needs to store other device-specific metadata, it should use an application-specific store. Refer to [IoT Hub Developer Guide][lnk-devguide-identity] for more information.

## Receive device-to-cloud messages

In this section, you'll create a Windows console app that reads device-to-cloud messages from IoT Hub. An IoT hub exposes an [Event Hubs][lnk-event-hubs-overview]-compatible endpoint to enable you to read device-to-cloud messages. To keep things simple, this tutorial creates a basic reader that is not suitable for a high throughput deployment. The [Process device-to-cloud messages][lnk-processd2c-tutorial] tutorial shows you how to process device-to-cloud messages at scale. The [Get Started with Event Hubs][lnk-eventhubs-tutorial] tutorial provides further information on how to process messages from Event Hubs and is applicable to the IoT Hub Event Hub-compatible endpoints.

1. In Visual Studio, add a new Visual C# Windows Classic Desktop project to the current solution using the **Console  Application** project template. Name the project **ReadDeviceToCloudMessages**.

    ![][10]

2. In Solution Explorer, right-click the **ReadDeviceToCloudMessages** project, and then click **Manage NuGet Packages**.

3. In the **NuGet Package Manager** window, make sure the **Include prerelease** option is checked. Then search for **WindowsAzure.ServiceBus**, click **Install**, and accept the terms of use.

    This downloads, installs, and adds a reference to the [Azure Service Bus][lnk-servicebus-nuget], with all its dependencies.

4. Add the following `using` statement at the top of the **Program.cs** file:

        using Microsoft.ServiceBus.Messaging;

5. Add the following fields to the **Program** class, replacing the placeholder value with the connection string for the IoT hub you created in the *Create an IoT hub* section:

        static string connectionString = "{iothub connection string}";
        static string iotHubD2cEndpoint = "messages/events";
        static EventHubClient eventHubClient;

6. Add the following method to the **Program** class:

        private async static Task ReceiveMessagesFromDeviceAsync(string partition)
        {
            var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.Now);
            while (true)
            {
                EventData eventData = await eventHubReceiver.ReceiveAsync();
                if (eventData == null) continue;

                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine(string.Format("Message received. Partition: {0} Data: '{1}'", partition, data));
            }
        }

    This method uses an **EventHubReceiver** instance to receive messages from all the IoT hub device-to-cloud receive partitions. Notice how you pass a `DateTime.Now` parameter when you create the **EventHubReceiver** object so that it only receives messages sent after it starts. This is useful in a test environment so you can see the current set of messages, but in a production environment your code should make sure that it processes all the messages - see the [How to process IoT Hub device-to-cloud messages][lnk-processd2c-tutorial] tutorial for more information.

7. Finally, add the following lines to the **Main** method:

        Console.WriteLine("Receive messages\n");
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, iotHubD2cEndpoint);

        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;

        foreach (string partition in d2cPartitions)
        {
           ReceiveMessagesFromDeviceAsync(partition);
        }  
        Console.ReadLine();


<!-- Links -->

[lnk-eventhubs-tutorial]: event-hubs-csharp-ephcs-getstarted.md
[lnk-devguide-identity]: iot-hub-devguide.md#identityregistry
[lnk-servicebus-nuget]: https://www.nuget.org/packages/WindowsAzure.ServiceBus
[lnk-event-hubs-overview]: event-hubs-overview.md

[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-processd2c-tutorial]: iot-hub-csharp-csharp-process-d2c.md

<!-- Images -->
[10]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp1.png
[11]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp2.png
[12]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp3.png


## Create a simulated device app

In this section, you'll create a Windows console app that simulates a device that sends device-to-cloud messages to an IoT hub.

1. In Visual Studio, add a new Visual C# Windows Classic Desktop project to the current solution using the **Console  Application** project template.  Name the project **SimulatedDevice**.

   	![][30]

2. In Solution Explorer, right-click the **SimulatedDevice** project, and then click **Manage NuGet Packages**.

3. In the **NuGet Package Manager** window, make sure the **Include prerelease** option is checked. Search for **Microsoft Azure Devices Client**, click **Install**, and accept the terms of use.

	This downloads, installs, and adds a reference to the [Azure IoT - Device SDK NuGet package][lnk-device-nuget].

4. Add the following `using` statement at the top of the **Program.cs** file:

		using Microsoft.Azure.Devices.Client;
        using Newtonsoft.Json;
        using System.Threading;

5. Add the following fields to the **Program** class, substituting the placeholder values with the IoT hub hostname you retrieved in the *Create an IoT hub* section and the device key retrieved in the *Create a device identity* section:

		static DeviceClient deviceClient;
        static string iotHubUri = "{iot hub hostname}";
        static string deviceKey = "{device key}";

6. Add the following method to the **Program** class:

		private static async void SendDeviceToCloudMessagesAsync()
        {
            double avgWindSpeed = 10; // m/s
            Random rand = new Random();

            while (true)
            {
                double currentWindSpeed = avgWindSpeed + rand.NextDouble() * 4 - 2;

                var telemetryDataPoint = new
                {
                    deviceId = "myFirstDevice",
                    windSpeed = currentWindSpeed
                };
                var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
                var message = new Message(Encoding.ASCII.GetBytes(messageString));

                await deviceClient.SendEventAsync(message);
                Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, messageString);

                Thread.Sleep(1000);
            }
        }

	This method sends a new device-to-cloud message every second. The message contains a JSON-serialized object with the deviceId and a randomly generated number to simulate a wind speed sensor.

7. Finally, add the following lines to the **Main** method:

        Console.WriteLine("Simulated device\n");
        deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey("myFirstDevice", deviceKey));

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();

  By default, the **Create** method creates a **DeviceClient** that uses the AMQP protocol to communicate with IoT Hub. To use the HTTPS protocol, use the override of the **Create** method that enables you to specify the protocol. If you choose to use the HTTPS protocol, you should also add the **Microsoft.AspNet.WebApi.Client** NuGet package to your project to include the **System.Net.Http.Formatting** namespace.


> [AZURE.NOTE] To keep things simple, this tutorial does not implement any retry policy. In production code, you should implement retry policies (such as an exponential backoff), as suggested in the MSDN article [Transient Fault Handling][lnk-transient-faults].

<!-- Links -->

[lnk-device-nuget]: https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/
[lnk-transient-faults]: https://msdn.microsoft.com/en-us/library/hh680901(v=pandp.50).aspx

<!-- Images -->
[30]: ./media/iot-hub-getstarted-device-csharp/create-identity-csharp1.png


## Run the applications
You are now ready to run the applications.

1. In Visual Studio, in Solution Explorer, right-click your solution and then click **Set StartUp projects**. Select **Multiple startup projects**, then select **Start** as the **Action** for both the **ReadDeviceToCloudMessages** and **SimulatedDevice** projects.

    ![][41]

2. Press **F5** to start both apps running. The console output from the **SimulatedDevice** app shows the messages your simulated device sends to your IoT hub, and the console output from the **ReadDeviceToCloudMessages** app shows the messages that your IoT hub receives.

    ![][42]


## Next steps
In this tutorial, you configured a new IoT hub in the portal and then created a device identity in the hub's identity registry. You used this device identity to in a simulated device that sends device-to-cloud messages to the hub and created another app that displays the messages received by the hub. You can continue to explore IoT hub features and other IoT scenarios in the following tutorials:

* [Send Cloud-to-Device messages with IoT Hub](iot-hub-csharp-csharp-c2d.md) shows how to send messages to devices, and process the delivery feedback produced by IoT Hub.
* [Process Device-to-Cloud messages](iot-hub-csharp-csharp-process-d2c.md) shows how to reliably process telemetry and interactive messages coming from devices.
* [Uploading files from devices](iot-hub-csharp-csharp-file-upload.md) describes a pattern that makes use of cloud-to-device messages to facilitate file uploads from devices.

<!-- Images. -->

[1]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub1.png
[2]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub2.png
[3]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub3.png
[4]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub4.png
[5]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub5.png
[41]: ./media/iot-hub-csharp-csharp-getstarted/run-apps1.png
[42]: ./media/iot-hub-csharp-csharp-getstarted/run-apps2.png

<!-- Links -->

[lnk-c2d-tutorial]: iot-hub-csharp-csharp-c2d.md
[lnk-process-d2c-tutorial]: iot-hub-csharp-csharp-process-d2c.md
[lnk-upload-tutorial]: iot-hub-csharp-csharp-file-upload.md

[lnk-hub-sdks]: iot-hub-sdks-summary.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-resource-groups]: resource-group-portal.md
[lnk-portal]: https://portal.azure.com/