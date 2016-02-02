<properties
    pageTitle="Send cloud-to-device messages with IoT Hub | Microsoft Azure"
    description="Follow this tutorial to learn how to send cloud-to-device messages using Azure IoT Hub with C#."
    services="iot-hub"
    documentationCenter=".net"
    authors="fsautomata"
    manager="timlt"
    editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="dotnet"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="09/29/2015"
     ms.author="elioda"/>

# Tutorial: How to send cloud-to-device messages with IoT Hub
## Introduction
Azure IoT Hub is a fully managed service that enables reliable and secure bi-directional communications between millions of IoT devices and an application back end. The [Get started with IoT Hub](iot-hub-csharp-csharp-getstarted.md) tutorial shows how to create an IoT hub, provision a device identity in it, and code a simulated device that sends device-to-cloud messages.

This tutorial builds on [Get started with IoT Hub](iot-hub-csharp-csharp-getstarted.md) and shows how to send cloud-to-device messages to a single device, how to request delivery acknowledgement (*feedback*) from IoT Hub, and receive it from your application cloud back end.

You can find more information on cloud-to-device messages in the [IoT Hub Developer Guide](iot-hub-devguide.md#c2d).

At the end of this tutorial you will run two Windows console applications:

* **SimulatedDevice**, a modified version of the app created in [Get started with IoT Hub](iot-hub-csharp-csharp-getstarted.md), which connects to your IoT hub and receives cloud-to-device messages.
* **SendCloudToDevice**, which sends a cloud-to-device message to the simulated device through IoT Hub, and then receives its delivery aknowledgment.

> [!NOTE]
> IoT Hub has SDK support for many device platforms and languages (including C, Java, and Javascript) though Azure IoT device SDKs. Refer to the [Azure IoT Developer Center](http://www.azure.com/develop/iot) for step by step instructions on how to connect your device to this tutorial's code, and generally to Azure IoT Hub. Azure IoT service SDKs for Java and Node are coming soon.
> 
> 
In order to complete this tutorial you'll need the following:

* Microsoft Visual Studio 2015,

* An active Azure account. <br/>If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdevelop%2Fiot%2Ftutorials%2Fc2d%2F target=_blank).


## Receiving messages on the simulated device

In this section, you'll modify the simulated device application you created in [Get started with IoT Hub] to receive cloud-to-device messages from the IoT hub.

1. In Visual Studio, in the **SimulatedDevice** project, add the following method to the **Program** class.

        private static async void ReceiveC2dAsync()
        {
            Console.WriteLine("\nReceiving cloud to device messages from service");
            while (true)
            {
                Message receivedMessage = await deviceClient.ReceiveAsync();
                if (receivedMessage == null) continue;

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received message: {0}", Encoding.ASCII.GetString(receivedMessage.GetBytes()));
                Console.ResetColor();

                await deviceClient.CompleteAsync(receivedMessage);
            }
        }

    The `ReceiveAsync` method asynchronously returns the received message at the time that it is received by the device. It returns *null* after a specifiable timeout period (in this case the default of 1 minute is used). When this happens, we want the code to continue waiting for new messages. This is the reason for the `if (receivedMessage == null) continue` line.

    The call to `CompleteAsync()` notifies IoT Hub that the message has been successfully processed and that it can be safely removed from the device queue. If something happened that prevented the device app from completing the processing of the message, IoT Hub will deliver it again; it is then important that message processing logic in the device app be *idempotent*, i.e. receiving multiple times the same message should produce the same result. An application can also temporarily `Abandon` a message, which will have IoT hub retain the message in the queue for future consumption; or `Reject` a message, which will permanently remove the message from the queue. Refer to [IoT Hub Developer Guide][IoT Hub Developer Guide - C2D] for more information on the cloud-to-device message lifecycle.

> [AZURE.NOTE] When using HTTP/1 instead of AMQP as a transport, the `ReceiveAsync` will return immediately. The supported pattern for cloud-to-device messages with HTTP/1 is intermittently connected devices that check for messages infrequently (i.e. less than every 25 mins). Issuing more HTTP/1 receives will result in IoT Hub throttling the requests. Refer to [IoT Hub Developer Guide][IoT Hub Developer Guide - C2D] for more information on the differences between AMQP and HTTP/1 support, and IoT Hub throttling.

2. Add the following method in the **Main** method right before the `Console.ReadLine()` line:

        ReceiveC2dAsync();

> [AZURE.NOTE] For simplicity's sake, this tutorial does not implement any retry policy. In production code, it is reccommended to implement retry policies (such as exponential backoff), as suggested in the MSDN article [Transient Fault Handling].

<!-- Links -->
[IoT Hub Developer Guide - C2D]: iot-hub-devguide.md#c2d

<!-- Images -->


## Send a cloud-to-device message from the app back end

In this section, you'll write a Windows console app that sends cloud-to-device messages to the simulated device app.

1. In the current Visual Studio solution, create a new Visual C# Desktop App project using the **Console  Application** project template. Name the project **SendCloudToDevice**.

   	![][20]

2. In Solution Explorer, right-click the solution, and then click **Manage NuGet Packages for Solution...**. 

	This displays the Manage NuGet Packages window.

3. Search for `Microsoft Azure Devices`, click **Install**, and accept the terms of use. 

	This downloads, installs, and adds a reference to the [Azure IoT - Service SDK NuGet package].

4. Add the following `using` statement at the top of the **Program.cs** file:

		using Microsoft.Azure.Devices;

5. Add the following fields to the **Program** class, substituting the placeholder value with the IoT hub connection string from [Get started with IoT Hub]:

		static ServiceClient serviceClient;
        static string connectionString = "{iot hub connection string}";

6. Add the following method to the **Program** class:

		private async static Task SendCloudToDeviceMessageAsync()
        {
            var commandMessage = new Message(Encoding.ASCII.GetBytes("Cloud to device message."));
            await serviceClient.SendAsync("myFirstDevice", commandMessage);
        }

	This method will send a new cloud-to-device message to the device with id `myFirstDevice`. Change this parameter accordingly, in case you modified from the one used in [Get started with IoT Hub].

7. Finally, add the following lines to the **Main** method:

        Console.WriteLine("Send Cloud-to-Device message\n");
        serviceClient = ServiceClient.CreateFromConnectionString(connectionString);

        Console.WriteLine("Press any key to send a C2D message.");
        Console.ReadLine();
        SendCloudToDeviceMessageAsync().Wait();
        Console.ReadLine();

8. From within Visual Studio, right click your solution and select **Set StartUp projects...**. Select **Multiple startup projects**, then select the **Start** action for **ProcessDeviceToCloudMessages**, **SimulatedDevice**, and **SendCloudToDevice**.

9.  Press **F5**, and you should see all three application start. Select the **SendCloudToDevice** windows and press **Enter**: you should see the message being received by the simulated app.

    ![][21]

## Receiving delivery feedback
It is possible to request delivery (or expiration) ackownledgments from IoT Hub for each cloud-to-device message. This enables the cloud back end to easily inform retry or compensation logic. Refer to the [IoT Hub Developer Guide][IoT Hub Developer Guide - C2D] for more information on cloud-to-device feedback.

In this section, you will modify the **SendCloudToDevice** app to request feedback and receive them from IoT Hub.

1. In Visual Studio, in the **SendCloudToDevice** project, add the following method to the **Program** class.
   
        private async static void ReceiveFeedbackAsync()
        {
            var feedbackReceiver = serviceClient.GetFeedbackReceiver();

            Console.WriteLine("\nReceiving c2d feedback from service");
            while (true)
            {
                var feedbackBatch = await feedbackReceiver.ReceiveAsync();
                if (feedbackBatch == null) continue;

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received feedback: {0}", string.Join(", ", feedbackBatch.Records.Select(f => f.StatusCode)));
                Console.ResetColor();

                await feedbackReceiver.CompleteAsync(feedbackBatch);
            }
        }

    Note that the receive pattern is the same one used to receive cloud-to-device messages from the device app.

2. Add the following method in the **Main** method right after the `serviceClient = ServiceClient.CreateFromConnectionString(connectionString)` line:

        ReceiveFeedbackAsync();

3. In order to request feedback for the delivery of your cloud-to-device message, you have to specify a property in the **SendCloudToDeviceMessageAsync** method. Add the following line, right after the `var commandMessage = new Message(...);` line:

        commandMessage.Ack = DeliveryAcknowledgement.Full;

4.  Run the apps by pressing **F5**, and you should see all three application start. Select the **SendCloudToDevice** windows and press **Enter**: you should see the message being received by the simulated app, and after a few seconds, the feedback message being received by your **SendCloudToDevice** app.

    ![][22]

> [AZURE.NOTE] For simplicity's sake, this tutorial does not implement any retry policy. In production code, it is reccommended to implement retry policies (such as exponential backoff), as suggested in the MSDN article [Transient Fault Handling].

<!-- Links -->

[IoT Hub Developer Guide - C2D]: iot-hub-devguide.md#c2d
[Azure IoT - Service SDK NuGet package]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[Transient Fault Handling]: https://msdn.microsoft.com/en-us/library/hh680901(v=pandp.50).aspx
[Get started with IoT Hub]: iot-hub-csharp-csharp-getstarted.md

<!-- Images -->
[20]: ./media/iot-hub-c2d-cloud-csharp/create-identity-csharp1.png
[21]: ./media/iot-hub-c2d-cloud-csharp/sendc2d1.png
[22]: ./media/iot-hub-c2d-cloud-csharp/sendc2d2.png








## Next steps
In this tutorial, you learned how to send and receive cloud-to-device messages. You can continue explore IoT hub features and scenario with the following tutorials:

* [Process Device-to-Cloud messages](iot-hub-csharp-csharp-process-d2c.md), shows how to reliably process telemetry and interactive messages coming from devices.
* [Uploading files from devices](iot-hub-csharp-csharp-file-upload.md), describes a pattern that makes use of cloud-to-device messages to facilitate file uploads from devices.

Additional information on IoT Hub:

* [IoT Hub Overview](iot-hub-what-is-iot-hub.md)
* [IoT Hub Developer Guide](iot-hub-devguide.md)
* [IoT Hub Guidance](iot-hub-guidance.md)
* [Supported device platforms and languages](https://github.com/Azure/azure-iot-sdks/blob/master/doc/tested_configurations.md)
* [Azure IoT Developer Center](http://www.azure.com/develop/iot)

<!-- Images. -->

<!-- Links -->

[Get started with IoT Hub]: iot-hub-csharp-csharp-getstarted.md

[IoT Hub Developer Guide - C2D]: iot-hub-devguide.md#c2d

[Azure portal]: https://portal.azure.com/

[Send Cloud-to-Device messages with IoT Hub]: iot-hub-csharp-csharp-c2d.md
[Process Device-to-Cloud messages]: iot-hub-csharp-csharp-process-d2c.md
[Uploading files from devices]: iot-hub-csharp-csharp-file-upload.md

[IoT Hub Overview]: iot-hub-what-is-iot-hub.md
[IoT Hub Guidance]: iot-hub-guidance.md
[IoT Hub Developer Guide]: iot-hub-devguide.md
[IoT Hub Supported Devices]: iot-hub-supported-devices.md
[Get started with IoT Hub]: iot-hub-csharp-csharp-getstarted.md
[Supported devices]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/tested_configurations.md
[Azure IoT Developer Center]: http://www.azure.com/develop/iot
