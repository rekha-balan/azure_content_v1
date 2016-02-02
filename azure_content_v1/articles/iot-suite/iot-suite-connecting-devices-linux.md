<properties
   pageTitle="Connect a device using C on Linux | Microsoft Azure"
   description="Describes how to connect a device to the Azure IoT Suite preconfigured remote monitoring solution using an application written in C running on Linux."
   services=""
   suite="iot-suite"
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>

<tags
   ms.service="iot-suite"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="11/10/2015"
   ms.author="dobett"/>


# Connect your device to the IoT Suite remote monitoring preconfigured solution
> [AZURE.SELECTOR]
- [C on Windows](../articles/iot-suite/iot-suite-connecting-devices.md)
- [C on Linux](../articles/iot-suite/iot-suite-connecting-devices-linux.md)
- [C on mbed](../articles/iot-suite/iot-suite-connecting-devices-mbed.md)
- [Node.js](../articles/iot-suite/iot-suite-connecting-devices-node.md)

## Scenario overview

In this scenario, you will create a device that sends the following telemetry to the remote monitoring [preconfigured solution][lnk-what-are-preconfig-solutions]:

- External temperature
- Internal temperature
- Humidity

For simplicity, the code on the device generates sample values, but we encourage you to extend the sample by connecting real sensors to your device and sending real telemetry.

## Before you start

Before you write any code for your device, you should provision your remote monitoring preconfigured solution and then provision a device in that solution.

### Provision your remote monitoring preconfigured solution

The device you create will send data to an instance of the [remote monitoring][lnk-remote-monitoring] preconfigured solution. Visit [Get started with Azure IoT Suite][lnk-getstarted] to create an Azure account and provision IoT Suite. Select **Remote monitoring** when you create your new solution.

When the remote monitoring solution has been provisioned, click **Launch** to open the solution dashboard.

![][img-dashboard]

### Provision your device in the remote monitoring solution

> [AZURE.NOTE] If you have already provisioned a device in your solution, you can skip this step. You will need to know the device credentials when you create the client application.

For a device to connect to the preconfigured solution, it must be able to identify itself using valid credentials. You can get the device credentials from the solution dashboard and then include them in your client application. 

To add a new device to your remote monitoring solution, complete the following steps in the solution dashboard:

1.  In the lower left-hand corner of the dashboard, click **Add a device**.

    ![][1]

2.  In the **Custom Device** panel, click on **Add new**.

    ![][2]

3.  Choose **Let me define my own Device ID**, enter a Device ID such as **mydevice**, click **Check ID** to verify that name isn't in use, and then click **Create** to provision the device.

    ![][3]

5. Make a note the device credentials (Device ID, IoT Hub Hostname, and Device Key), your client application will need them to connect your device to the remote monitoring solution. Then click **Done**.

    ![][4]

6. Make sure your device displays correctly in the devices section. The status is **Pending** until the device establishes a connection to the remote monitoring solution.

    ![][5]

[img-dashboard]: ./media/iot-suite-selector-connecting/dashboard.png
[1]: ./media/iot-suite-selector-connecting/suite0.png
[2]: ./media/iot-suite-selector-connecting/suite1.png
[3]: ./media/iot-suite-selector-connecting/suite2.png
[4]: ./media/iot-suite-selector-connecting/suite3.png
[5]: ./media/iot-suite-selector-connecting/suite5.png

[lnk-getstarted]: http://www.microsoft.com/server-cloud/internet-of-things/getting-started.aspx
[lnk-what-are-preconfig-solutions]: ../articles/iot-suite/iot-suite-what-are-preconfigured-solutions.md
[lnk-remote-monitoring]: ../articles/iot-suite/iot-suite-remote-monitoring-sample-walkthrough.md


## Build and run the C sample solution on Linux
1. To clone the *Microsoft Azure IoT SDKs* GitHub repository and install the *Microsoft Azure IoT device SDK for C* in your Linux desktop environment, follow the [Set up a Linux development environment](https://github.com/azure/azure-iot-sdks/blob/develop/c/doc/devbox_setup.md#linux) instructions.

2. Open the file **c/serializer/samples/remote_monitoring/remote_monitoring.c** in a text editor.

3. Locate the following code in the file:

    ```
 static const char* deviceId = "[Device Id]";
 static const char* deviceKey = "[Device Key]";
 static const char* hubName = "[IoTHub Name]";
 static const char* hubSuffix = "[IoTHub Suffix, i.e. azure-devices.net]";
 ```
4. Replace **[Device Id]Device Id]** and **[Device Key]Device Key]** with values for your device from the remote monitoring solution dashboard.

5. Use the **IoT Hub Hostname** from the dashboard to replace **[IoTHub Name]IoTHub Name]** and **[IoTHub Suffix, i.e. azure-devices.net]IoTHub Suffix, i.e. azure-devices.net]**. For example, if your **IoT Hub Hostname** is **contoso.azure-devices.net**, replace **[IoTHub Name]IoTHub Name]** with **contoso** and replace **[IoTHub Suffix, i.e. azure-devices.net]IoTHub Suffix, i.e. azure-devices.net]** with **azure-devices.net** as shown below:

    ```
 static const char* deviceId = "mydevice";
 static const char* deviceKey = "mykey";
 static const char* hubName = "Contoso";
 static const char* hubSuffix = "azure-devices.net";
 ```
6. Save your changes and build the samples. To build your sample you can run the build.sh script in the **c/build_all/linux** directory. The build script creates a **cmake** folder to store the compiled sample programs.

7. Run the **cmake/serializer/samples/remote_monitoring/remote_monitoring** sample application.


## View device telemetry in the dashboard

The dashboard in the remote monitoring solution enables you to view the telemetry that your devices send to IoT Hub.

1. In your browser, return to the remote monitoring solution dashboard, click **Devices** in the left-hand panel to navigate to the **Devices list**.

2. In the **Devices list**, you should see that the status of your device is now **Running**.

    ![][18]

3. In the dashboard, select your device in the **Device to View** drop-down to view its telemetry. The telemetry from the sample application is 50 units for internal temperature, 55 units for external temperature, and 50 units for humidity. Note that by default the dashboard displays only temperature and humidity values.

    ![][img-telemetry]

## Send a command to your device

The dashboard in the remote monitoring solution enables you to request IoT Hub to send commands to your devices. For example, in the remote monitoring solution you can send a command to set the internal temperature of a device.

1. In the remote monitoring solution dashboard, click **Devices** in the left-hand panel to navigate to the **Devices list**.

2. Click **Device ID** for your device in the **Devices list**.

3. In the **Device details** panel, click **Commands**.

    ![][13]

4. In the **Command** drop-down, select **SetTemperature**, and then in **Temperature** enter a new temperature value. Click **Send command** to send the command to the device.

    ![][14]

    > [AZURE.NOTE] The command history initially shows the command status as **Pending**. When the device acknowledges the command, the status changes to **Success**.

5. On the dashboard, verify that the device is now sending 75 as the new temperature value.

## Next steps

The article [Customizing preconfigured solutions][lnk-customize] describes some ways you can extend this sample. Possible extensions include using real sensors and implementing additional commands.

[13]: ./media/iot-suite-visualize-connecting/suite4.png
[14]: ./media/iot-suite-visualize-connecting/suite7-1.png
[18]: ./media/iot-suite-visualize-connecting/suite10.png
[img-telemetry]: ./media/iot-suite-visualize-connecting/telemetry.png
[lnk-customize]: ../articles/iot-suite/iot-suite-guidance-on-customizing-preconfigured-solutions.md
[lnk-dev-messaging]: ../articles/iot-hub/iot-hub-devguide.md#messaging


[lnk-setup-linux]: https://github.com/azure/azure-iot-sdks/blob/develop/c/doc/devbox_setup.md#linux

