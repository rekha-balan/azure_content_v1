<properties
    pageTitle="Get started with Azure IoT Hub for Node.js | Microsoft Azure"
    description="Follow this tutorial to get started using Azure IoT Hub with Node.js."
    services="iot-hub"
    documentationCenter="nodejs"
    authors="dominicbetts"
    manager="timlt"
    editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="javascript"
     ms.topic="hero-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="01/19/2016"
     ms.author="dobett"/>

# Get started with Azure IoT Hub for Node.js
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
* Create a simulated device that sends telemetry to your cloud backend.

At the end of this tutorial you will have three Node.js console applications:

* **CreateDeviceIdentity.js**, which creates a device identity and associated security key to connect your simulated device.
* **ReadDEviceToCloudMessages.js**, which displays the telemetry sent by your simulated device.
* **SimulatedDevice.js**, which connects to your IoT hub with the device identity created earlier, and sends a telemetry message every second.

> [!NOTE]
> The article [IoT Hub SDKs](iot-hub-sdks-summary.md) provides information about the various SDKs that you can use to build both applications to run on devices and your solution backend.
> 
> 
To complete this tutorial you'll need the following:

* Node.js version 0.12.x or later. <br/> [Prepare your development environment](https://github.com/Azure/azure-iot-sdks/blob/master/node/device/doc/devbox_setup.md) describes how to install Node.js for this tutorial on either Windows or Linux.

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

7. Click **Settings** on the IoT Hub blade, then **Messaging** on the **Settings** blade. On the **Messaging** blade, make a note of the **Event Hub-compatible name** and the **Event Hub-compatible endpoint**. You will need these values when you create your **read-d2c-messages** application.

    ![][6]


You have now created your IoT hub and you have the IoT Hub hostname, IoT Hub connection string, Event Hub-compatible name, and Event-Hub compatible endpoint values that you need to complete the rest of this tutorial.

## Create a device identity

In this section, you'll create a Node.js console app that creates a new device identity in the identity registry in your IoT hub. A device cannot connect to IoT hub unless it has an entry in the device identity registry. Refer to the **Device Identity Registry** section of the [IoT Hub Developer Guide][lnk-devguide-identity] for more information. When you run this console application, it generates a unique device ID and key that your device can identify itself with when it sends device-to-cloud messages to IoT Hub.

1. Create a new empty folder called **createdeviceidentity**. In the **createdeviceidentity** folder, create a new package.json file using the following command at your command-prompt. Accept all the defaults:

    ```
    npm init
    ```

2. At your command-prompt in the **createdeviceidentity** folder, run the following command to install the **azure-iothub** package:

    ```
    npm install azure-iothub --save
    ```

3. Using a text editor, create a new **CreateDeviceIdentity.js** file in the **createdeviceidentity** folder.

4. Add the following `require` statement at the start of the **CreateDeviceIdentity.js** file:

    ```
    'use strict';
    
    var iothub = require('azure-iothub');
    ```

5. Add the following code to the **CreateDeviceIdentity.js** file, replacing the placeholder value with the connection string for the IoT hub you created in the previous section: 

    ```
    var connectionString = '{iothub connection string}';
    
    var registry = iothub.Registry.fromConnectionString(connectionString);
    ```

6. Add the following code to create a new device definition in the device identity registry in your IoT hub. This code creates a new device if the device id does not exist in the registry, otherwise it returns the key of the existing device:

    ```
    var device = new iothub.Device(null);
    device.deviceId = 'myFirstDevice';
    registry.create(device, function(err, deviceInfo, res) {
      if (err) {
        registry.get(device.deviceId, printDeviceInfo);
      }
      if (deviceInfo) {
        printDeviceInfo(err, deviceInfo, res)
      }
    });

    function printDeviceInfo(err, deviceInfo, res) {
      if (deviceInfo) {
        console.log('Device id: ' + deviceInfo.deviceId);
        console.log('Device key: ' + deviceInfo.authentication.SymmetricKey.primaryKey);
      }
    }
    ```

7. Save and close **CreateDeviceIdentity.js** file.

8. To run the **createdeviceidentity** application, execute the following command at the command-prompt in the createdeviceidentity folder:

    ```
    node CreateDeviceIdentity.js 
    ```

9. Make a note of the **Device id** and **Device key**. You will need these later when you create an application that connects to IoT Hub as a device.

> [AZURE.NOTE] The IoT Hub identity registry only stores device identities to enable secure access to the hub. It stores device IDs and keys to use as security credentials and an enabled/disabled flag that enables you to disable access for an individual device. If you application needs to store other device-specific metadata, it should use an application-specific store. Refer to [IoT Hub Developer Guide][lnk-devguide-identity] for more information.

## Receive device-to-cloud messages

In this section, you'll create a Node.js console app that reads device-to-cloud messages from IoT Hub. An IoT hub exposes an [Event Hubs][lnk-event-hubs-overview]-compatible endpoint to enable you to read device-to-cloud messages. To keep things simple, this tutorial creates a basic reader that is not suitable for a high throughput deployment. The [Process device-to-cloud messages][lnk-processd2c-tutorial] tutorial shows you how to process device-to-cloud messages at scale. The [Get Started with Event Hubs][lnk-eventhubs-tutorial] tutorial provides further information on how to process messages from Event Hubs and is applicable to the IoT Hub Event Hub-compatible endpoints.

1. Create a new empty folder called **readdevicetocloudmessages**. In the **readdevicetocloudmessages** folder, create a new package.json file using the following command at your command-prompt. Accept all the defaults:

    ```
    npm init
    ```

2. At your command-prompt in the **readdevicetocloudmessages** folder, run the following command to install the **amqp10** and **bluebird** packages:

    ```
    npm install amqp10 bluebird --save
    ```

3. Using a text editor, create a new **ReadDeviceToCloudMessages.js** file in the **readdevicetocloudmessages** folder.

4. Add the following `require` statements at the start of the **ReadDeviceToCloudMessages.js** file:

    ```
    'use strict';

    var AMQPClient = require('amqp10').Client;
    var Policy = require('amqp10').Policy;
    var translator = require('amqp10').translator;
    var Promise = require('bluebird');
    ```

5. Add the following variable declarations, replacing the placeholders with the values you noted previously. The value of the **{your event hub-compatible namespace}** placeholder comes from the **Event Hub-compatible endpoint** - it takes the form **xxxxnamespace.servicebus.windows.net**.

    ```
    var protocol = 'amqps';
    var eventHubHost = '{your event hub-compatible namespace}';
    var sasName = 'iothubowner';
    var sasKey = '{your iot hub key}';
    var eventHubName = '{your event hub-compatible name}';
    var numPartitions = 2;
    ```

    > [AZURE.NOTE] This code assumes you created your IoT hub in the F1 (free) tier. A free IoT hub has two partitions named "0" and "1". If you created your IoT hub using one of the other pricing tiers, you should adjust the code to create a **MessageReceiver** for each partition.

6. Add the following filter definition. This application uses a filter when it creates a receiver so that the receiver only reads messages sent to IoT Hub after the receiver starts running. This is useful in a test environment so you can see the current set of messages, but in a production environment your code should make sure that it processes all the messages - see the [How to process IoT Hub device-to-cloud messages][lnk-processd2c-tutorial] tutorial for more information.

    ```
    var filterOffset = new Date().getTime();
    var filterOption;
    if (filterOffset) {
      filterOption = {
      attach: { source: { filter: {
      'apache.org:selector-filter:string': translator(
        ['described', ['symbol', 'apache.org:selector-filter:string'], ['string', "amqp.annotation.x-opt-enqueuedtimeutc > " + filterOffset + ""]])
        } } }
      };
    }
    ```

7. Add the following code to create the receive address and an AMQP client:

    ```
    var uri = protocol + '://' + encodeURIComponent(sasName) + ':' + encodeURIComponent(sasKey) + '@' + eventHubHost;
    var recvAddr = eventHubName + '/ConsumerGroups/$default/Partitions/';
    
    var client = new AMQPClient(Policy.EventHub);
    ```

8. Add the following two functions that print output to the console:

    ```
    var messageHandler = function (partitionId, message) {
      console.log('Received(' + partitionId + '): ', message.body);
    };
    
    var errorHandler = function(partitionId, err) {
      console.warn('** Receive error: ', err);
    };
    ```

9. Add the following function that acts as a receiver for a given partition using the filter:

    ```
    var createPartitionReceiver = function(partitionId, receiveAddress, filterOption) {
      return client.createReceiver(receiveAddress, filterOption)
        .then(function (receiver) {
          console.log('Listening on partition: ' + partitionId);
          receiver.on('message', messageHandler.bind(null, partitionId));
          receiver.on('errorReceived', errorHandler.bind(null, partitionId));
        });
    };
    ```

10. Add the following code to connect to the Event Hub-compatible endpoint and start the receivers:

    ```
    client.connect(uri)
      .then(function () {
        var partitions = [];
        for (var i = 0; i < numPartitions; ++i) {
          partitions.push(createPartitionReceiver(i, recvAddr + i, filterOption));
        }
        return Promise.all(partitions);
    })
    .error(function (e) {
        console.warn('Connection error: ', e);
    });
    ```

11. Save and close the **ReadDeviceToCloudMessages.js** file.

<!-- Links -->

[lnk-eventhubs-tutorial]: event-hubs-csharp-ephcs-getstarted.md
[lnk-devguide-identity]: iot-hub-devguide.md#identityregistry
[lnk-event-hubs-overview]: event-hubs-overview.md
[lnk-processd2c-tutorial]: iot-hub-csharp-csharp-process-d2c.md



## Create a simulated device app

In this section, you'll create a Node.js console app that simulates a device that sends device-to-cloud messages to an IoT hub.

1. Create a new empty folder called **simulateddevice**. In the **simulateddevice** folder, create a new package.json file using the following command at your command-prompt. Accept all the defaults:

    ```
    npm init
    ```

2. At your command-prompt in the **simulateddevice** folder, run the following command to install the **azure-iot-device-amqp** package:

    ```
    npm install azure-iot-device-amqp --save
    ```

3. Using a text editor, create a new **SimulatedDevice.js** file in the **simulateddevice** folder.

4. Add the following `require` statements at the start of the **SimulatedDevice.js** file:

    ```
    'use strict';

    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    ```

5. Add a **connectionString** variable and use it to create a device client. Replace **{youriothubname}** with your IoT hub name, and **{yourdeviceid}** and **{yourdevicekey}** with the device values you generated in the *Create a device identity* section:

    ```
    var connectionString = 'HostName={youriothubname}.azure-devices.net;DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}';
    
    var client = clientFromConnectionString(connectionString);
    ```

6. Add the following function to display output from the application:

    ```
    function printResultFor(op) {
      return function printResult(err, res) {
        if (err) console.log(op + ' error: ' + err.toString());
        if (res) console.log(op + ': message sent');
      };
    }
    ```

7. Use the **setInterval** function to send a new message to your IoT hub every second:

    ```
    setInterval(function(){
      var windSpeed = 10 + (Math.random() * 4);
      var data = JSON.stringify({ deviceId: 'myFirstDevice', windSpeed: windSpeed });
      var message = new Message(data);
      console.log("Sending message: " + message.getData());
      client.sendEvent(message, printResultFor('send'));
    }, 1000);
    ```

8. Save and close the **SimulatedDevice.js** file.

> [AZURE.NOTE] To keep things simple, this tutorial does not implement any retry policy. In production code, you should implement retry policies (such as an exponential backoff), as suggested in the MSDN article [Transient Fault Handling][lnk-transient-faults].

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/en-us/library/hh680901(v=pandp.50).aspx


## Run the applications
You are now ready to run the applications.

1. At a command-prompt in the **readdevicetocloudmessages** folder, run the following command to begin monitoring your IoT hub:

    ```
 node ReadDeviceToCloudMessages.js 
 ```

    ![][7]

2. At a command-prompt in the **simulateddevice** folder, run the following command to begin sending telemetry data to your IoT hub:

    ```
 node SimulatedDevice.js
 ```

    ![][8]


## Next steps
In this tutorial, you configured a new IoT hub in the portal and then created a device identity in the hub's identity registry. You used this device identity to in a simulated device that sends device-to-cloud messages to the hub and created another app that displays the messages received by the hub. You can continue to explore IoT hub features and other IoT scenarios in the following tutorials:

* [Send Cloud-to-Device messages with IoT Hub](iot-hub-csharp-csharp-c2d.md) shows how to send messages to devices, and process the delivery feedback produced by IoT Hub.
* [Process Device-to-Cloud messages](iot-hub-csharp-csharp-process-d2c.md) shows how to reliably process telemetry and interactive messages coming from devices.
* [Uploading files from devices](iot-hub-csharp-csharp-file-upload.md) describes a pattern that makes use of cloud-to-device messages to facilitate file uploads from devices.

<!-- Images. -->

[1]: ./media/iot-hub-node-node-getstarted/create-iot-hub1.png
[2]: ./media/iot-hub-node-node-getstarted/create-iot-hub2.png
[3]: ./media/iot-hub-node-node-getstarted/create-iot-hub3.png
[4]: ./media/iot-hub-node-node-getstarted/create-iot-hub4.png
[5]: ./media/iot-hub-node-node-getstarted/create-iot-hub5.png
[6]: ./media/iot-hub-node-node-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-node-node-getstarted/runapp1.png
[8]: ./media/iot-hub-node-node-getstarted/runapp2.png

<!-- Links -->

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/node/device/doc/devbox_setup.md
[lnk-c2d-tutorial]: iot-hub-csharp-csharp-c2d.md
[lnk-process-d2c-tutorial]: iot-hub-csharp-csharp-process-d2c.md
[lnk-upload-tutorial]: iot-hub-csharp-csharp-file-upload.md

[lnk-hub-sdks]: iot-hub-sdks-summary.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-resource-groups]: resource-group-portal.md
[lnk-portal]: https://portal.azure.com/