<properties
    pageTitle="Upload files from devices using IoT Hub | Microsoft Azure"
    description="Follow this tutorial to learn how to upload files from devices using Azure IoT Hub with C#."
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

# Tutorial: How to upload files from devices to the cloud with IoT Hub
## Introduction
Azure IoT Hub is a fully managed service that enables reliable and secure bi-directional communications between millions of IoT devices and an application back end. Previous tutorials ([Get started with IoT Hub](iot-hub-csharp-csharp-getstarted.md) and [Send Cloud-to-Device messages with IoT Hub](iot-hub-csharp-csharp-c2d.md)) illustrate the basic device-to-cloud and cloud-to-device messaging functionality of IoT Hub, and how to access them from devices and cloud components. [Process Device-to-Cloud messages](iot-hub-csharp-csharp-process-d2c.md) described a way to reliably store device-to-cloud messages in Azure blob storage. There are cases, however, where the data coming from devices does not map easily to relatively small device-to-cloud messages. Some examples are large files containing images, videos, vibration data sample at high frequency, or containing some form of preprocessed data. These files are usually processed in a batch fashion using tools such as [Azure Data Factory](https://azure.microsoft.com/en-us/documentation/services/data-factory/) or the [Hadoop](https://azure.microsoft.com/en-us/documentation/services/hdinsight/) stack. When uploading a file from a device is preferred to sending events, it is still possible to use IoT Hub security and reliability functionality.

This tutorial builds on the code presented in [Send Cloud-to-Device messages with IoT Hub](iot-hub-csharp-csharp-c2d.md) to show how to use cloud-to-device messages to securely provide to the device an Azure blob URI to be used to upload the file, and how to use IoT Hub delivery acknowledgments to trigger the processing of the file from your app back end. The advantages of this approach is the reuse of IoT Hub device identity, and of the delivery acknowledgment of cloud-to-device messages to inform the app back end that the file has been uploaded successfully.

> [!NOTE]
> The same approach used here can be used to securely have devices download files from the cloud.
> 
> 
You can find more information on cloud-to-device messages and IoT Hub security in the [IoT Hub Developer Guide](iot-hub-devguide.md).

At the end of this tutorial you will run two Windows console applications:

* **SimulatedDevice**, a modified version of the app created in [Send Cloud-to-Device messages with IoT Hub](iot-hub-csharp-csharp-c2d.md), which connects to your IoT hub, receives cloud-to-device messages containing Azure blob URIs. For each cloud-to-device message received, it triggers a file upload to the specified blob URI.
* **SendCloudToDevice**, which builds an Azure blob URI (as explained in [Create and Use a SAS with the Blob Service](../storage/storage-dotnet-shared-access-signature-part-2.md), sends it in a cloud-to-device message to the simulated device through IoT Hub, and then receives its delivery aknowledgment.

> [!NOTE]
> IoT Hub has SDK support for many device platforms and languages (including C, Java, and Javascript) though Azure IoT device SDKs. Refer to the [Azure IoT Developer Center](http://www.azure.com/develop/iot) for step by step instructions on how to connect your device to this tutorial's code, and generally to Azure IoT Hub. Azure IoT service SDKs for Java and Node are coming soon.
> 
> 
In order to complete this tutorial you'll need the following:

* Microsoft Visual Studio 2015,

* An active Azure account. <br/>If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdevelop%2Fiot%2Ftutorials%2Ffile-upload%2F target=_blank).


## Provision an Azure Storage account
Since the simulated device will upload a file in an Azure Storage blob, you must have an Azure Storage account. You can use an existing one, or follow the instructions in [About Azure Storage] to create a new one. Take note of the storage account connection string.

## Send an Azure blob URI to the simulated device

In this section, you'll modify the **SendCloudtoDevice** console app you created in [Send Cloud-to-Device messages with IoT Hub] to include an Azure blob URI with a shared access signature. This allows the cloud back end to grant write access to the blob only to the recipient of the cloud-to-device message.

1. In Visual Studio, right-click the **SendCloudtoDevice** project, and then click **Manage NuGet Packages...**. 

    This displays the Manage NuGet Packages window.

2. Search for `WindowsAzure.Storage`, click **Install**, and accept the terms of use. 

    This downloads, installs, and adds a reference to the [Microsoft Azure Storage SDK](https://www.nuget.org/packages/WindowsAzure.Storage/).

3. In the **Program.cs** file, add the following statements at the top of the file:

        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;

4. In the **Program** class, add the following class fields, substituting the connection string for your storage account.

        static string storageConnectionString = "{storage connection string}";

    Then add the following method (you can substitute any blob container name, this tutorial uses **iothubfileuploadtutorial**):
   
        private static async Task<string> GenerateBlobUriAsync()
        {
            var storageAccount = CloudStorageAccount.Parse(storageConnectionString);
            var blobClient = storageAccount.CreateCloudBlobClient();
            var blobContainer = blobClient.GetContainerReference("iothubfileuploadtutorial");
            await blobContainer.CreateIfNotExistsAsync();

            var blobName = String.Format("deviceUpload_{0}", Guid.NewGuid().ToString());
            CloudBlockBlob blob = blobContainer.GetBlockBlobReference(blobName);

            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
            sasConstraints.SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5);
            sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
            sasConstraints.Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write;
            string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);

            return blob.Uri + sasBlobToken;
        }

    This method creates a new blob reference and generates a shared access signature URI as described in [Create and use a SAS with Blob storage](https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-shared-access-signature-part-2/). Note that the method above generates a signature URI that is valid for 24 hours. If the target device needs more time to upload the file (e.g. it connects infrequently, it has unreliable connectivity to upload a large file), you might consider longer expiration times for the signatures.

5. Modify the **SendCloudToDeviceMessageAsync** in the following way:

        private async static Task SendCloudToDeviceMessageAsync()
        {
            var commandMessage = new Message();
            commandMessage.Properties["command"] = "FileUpload";
            commandMessage.Properties["fileUri"] = await GenerateBlobUriAsync();
            commandMessage.Ack = DeliveryAcknowledgement.Full;

            await serviceClient.SendAsync("myFirstDevice", commandMessage);
        }

    This method sends a cloud-to-device message that contains two application properties: one identifying this message as a command to upload a file, the other holding the blob URI. It also requests full delivery acknowledgments. Note that the information in the two application properties can be serialized in a message body, but that would incur additional processing to serialize and deserialize the information.

<!-- Links -->

[About Azure Storage]: https://azure.microsoft.com/en-us/documentation/articles/storage-create-storage-account/#create-a-storage-account

[IoT Hub Developer Guide - C2D]: iot-hub-devguide.md#c2d
[Azure IoT - Service SDK NuGet package]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[Transient Fault Handling]: https://msdn.microsoft.com/en-us/library/hh680901(v=pandp.50).aspx
[Get started with IoT Hub]: iot-hub-csharp-csharp-getstarted.md

<!-- Images -->









## Upload a file from a simulated device

In this section, you'll modify the simulated device application you created in [Send Cloud-to-Device messages with IoT Hub] to receive cloud-to-device messages from the IoT hub.

1. In Visual Studio, right-click the **SimulatedDevice** project, and then click **Manage NuGet Packages...**. 

    This displays the Manage NuGet Packages window.

2. Search for `WindowsAzure.Storage`, click **Install**, and accept the terms of use. 

    This downloads, installs, and adds a reference to the [Microsoft Azure Storage SDK](https://www.nuget.org/packages/WindowsAzure.Storage/).

3. In the **Program.cs** file, add the following statements at the top of the file:

        using System.IO;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;

4. In the **Program** class, change the **ReceiveC2dAsync** method in the following way:
         
        private static async void ReceiveC2dAsync()
        {
            Console.WriteLine("\nReceiving cloud to device messages from service");
            while (true)
            {
                Message receivedMessage = await deviceClient.ReceiveAsync();
                if (receivedMessage == null) continue;

                if (receivedMessage.Properties.ContainsKey("command") && receivedMessage.Properties["command"] == "FileUpload")
                {
                    UploadFileToBlobAsync(receivedMessage);
                    continue;
                }

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received message: {0}", Encoding.ASCII.GetString(receivedMessage.GetBytes()));
                Console.ResetColor();
            }
        }

    This makes the **ReceiveC2dAsync** differentiate messages with the `command` property set to `FileUpload`, which will be handled by the **UploadFileToBlobAsync** method.

    Add the method below to handle the file upload commands.
   
        private static async Task UploadFileToBlobAsync(Message fileUploadCommand)
        {
            var fileUri = fileUploadCommand.Properties["fileUri"];
            var blob = new CloudBlockBlob(new Uri(fileUri));

            byte[] data = new byte[10 * 1024 * 1024];
            Random rng = new Random();
            rng.NextBytes(data);

            MemoryStream msWrite = new MemoryStream(data);
            msWrite.Position = 0;
            using (msWrite)
            {
                await blob.UploadFromStreamAsync(msWrite);
            }
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Uploaded file to: {0}", fileUri);
            Console.ResetColor();

            await deviceClient.CompleteAsync(fileUploadCommand);
        }

    This method uses the Azure Storage SDK to upload a randomly generated 10Mb blob to the specified URI. Refer to [Azure Storage - How to use blobs] for more information on how to upload blobs.

> [AZURE.NOTE] Note how this implementation of the simulated device completes the cloud-to-device message only after the blob has been uploaded. This approach simplifies the processing of the uploaded files in the back end because the delivery acknowledgment represents the availability of the uploaded file for processing. As explained in the [IoT Hub Developer Guide][IoT Hub Developer Guide - C2D], however, a message that is not completed before the *visibility timeout* (usually 1 minute) is put back in the device queue, and the **ReceiveAsync()** method will receive it again. For scenarios where the file upload can take longer, it might be preferable for the simulated device to keep a durable store of current upload jobs. This allows the simulated device to complete the cloud-to-device message before the file upload is complete, and then send a device-to-cloud message notifying the back end of completion.

<!-- Links -->
[IoT Hub Developer Guide - C2D]: iot-hub-devguide.md#c2d
[Azure Storage - How to use blobs]: https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-how-to-use-blobs/#upload-a-blob-into-a-container

<!-- Images -->







## Run the applications
Now you are ready to run the applications.

1. From within Visual Studio, right click your solution and select **Set StartUp projects...**. Select **Multiple startup projects**, then select the **Start** action for both **SimulatedDevice**, and **SendCloudToDevice** apps.

2. Press **F5**, and you should see all applications start. Select the **SendCloudToDevice** window and press a key. You will see the simulated device output a message when it has uploaded the file, and the **SendCloudToDevice** app show the successful feedback receipt. You can use the [Azure portal](https://portal.azure.com/) or Visual Studio Server Explorer to check the presence of the file in your storage account.

   ![][50]


## Next steps
In this tutorial, you learned how to leverage cloud-to-device messages to simplify file uploads from devices. You can continue explore IoT hub features and scenarios with the following tutorial:

* [Process Device-to-Cloud messages](iot-hub-csharp-csharp-process-d2c.md), shows how to reliably process telemetry and interactive messages coming from devices.

Additional information on IoT Hub:

* [IoT Hub Overview](iot-hub-what-is-iot-hub.md)
* [IoT Hub Developer Guide](iot-hub-devguide.md)
* [IoT Hub Guidance](iot-hub-guidance.md)
* [Supported device platforms and languages](https://github.com/Azure/azure-iot-sdks/blob/master/doc/tested_configurations.md)
* [Azure IoT Developer Center](http://www.azure.com/develop/iot)

<!-- Images. -->

[50]: ./media/iot-hub-csharp-csharp-file-upload/run-apps1.png

<!-- Links -->

[Send Cloud-to-Device messages with IoT Hub]: iot-hub-csharp-csharp-c2d.md

[Azure portal]: https://portal.azure.com/

[Azure Data Factory]: https://azure.microsoft.com/en-us/documentation/services/data-factory/
[Hadoop]: https://azure.microsoft.com/en-us/documentation/services/hdinsight/

[Get started with IoT Hub]: iot-hub-csharp-csharp-getstarted.md
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
