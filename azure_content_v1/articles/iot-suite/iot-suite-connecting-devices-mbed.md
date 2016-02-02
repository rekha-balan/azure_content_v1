<properties
   pageTitle="Connect a device using C on mbed | Microsoft Azure"
   description="Describes how to connect a device to the Azure IoT Suite preconfigured remote monitoring solution using an application written in C running on an mbed device."
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


## Build and run the C sample solution on mbed
The following instructions describe the steps for connecting an [mbed-enabled Freescale FRDM-K64F](https://developer.mbed.org/platforms/FRDM-K64F/) device to the remote monitoring solution.

### Connect the device to your network and desktop machine
1. Connect the mbed device to your network using an Ethernet cable. This step is necessary because the sample application requires internet access.

2. See [Getting Started with mbed](https://developer.mbed.org/platforms/FRDM-K64F/#getting-started-with-mbed) to connect your mbed device to your desktop PC.

3. If your desktop PC is running Windows, see [PC Configuration](https://developer.mbed.org/platforms/FRDM-K64F/#pc-configuration) to configure serial port access to your mbed device.


### Create an mbed project and import the sample code
1. In your web browser, go to the mbed.org [developer site](https://developer.mbed.org/). If you haven't signed up, you will see an option to create a new account (it's free). Otherwise, log in with your account credentials. Then click **Compiler** in the upper right-hand corner of the page. This should bring you to the Workspace Management interface.

2. Make sure the hardware platform you're using appears in the upper right-hand corner of the window, or click the icon in the right-hand corner to select your hardware platform.

3. Click **Import** on the main menu. Then click the **Click here** to import from URL link next to the mbed globe logo.

    ![][6]

4. In the pop-up window, enter the link for the sample code https://developer.mbed.org/users/AzureIoTClient/code/remote_monitoring/

    ![][7]

5. You can see in the mbed compiler that importing this project imported various libraries. Some are provided and maintained by the Azure IoT team ([azureiot_common](https://developer.mbed.org/users/AzureIoTClient/code/azureiot_common/), [iothub_client](https://developer.mbed.org/users/AzureIoTClient/code/iothub_client/), [iothub_amqp_transport](https://developer.mbed.org/users/AzureIoTClient/code/iothub_amqp_transport/), [iothub_http_transport](https://developer.mbed.org/users/AzureIoTClient/code/iothub_http_transport/), [proton-c-mbed](https://developer.mbed.org/users/AzureIoTClient/code/proton-c-mbed/)), while others are third party libraries available in the mbed libraries catalog.

    ![][8]

6. Open remote_monitoring\remote_monitoring.c, locate the following code in the file:

    ```
 static const char* deviceId = "[Device Id]";
 static const char* deviceKey = "[Device Key]";
 static const char* hubName = "[IoTHub Name]";
 static const char* hubSuffix = "[IoTHub Suffix, i.e. azure-devices.net]";
 ```
7. Replace [Device Id]Device Id] and [Device Key]Device Key], with your device data.

8. Use the IoT Hub Hostname device data to fill in IoTHub name and IoTHub Suffix. For example, if your IoT Hub Hostname is Contoso.azure-devices.net, Contoso is the IoTHub name and everything after it is the Suffix:

    ```
 static const char* deviceId = "mydevice";
 static const char* deviceKey = "mykey";
 static const char* hubName = "Contoso";
 static const char* hubSuffix = "azure-devices.net";
 ```

    ![][9]


### Build and run the program
1. Click **Compile** to build the program. You can safely ignore any warnings, but if the build generates errors, fix them before proceeding.

2. If the build is successful, a .bin file with the name of your project is generated. Copy the .bin file to the device. Saving the .bin file to the device causes the current terminal session to the device to reset. When it reconnects, reset the terminal again manually, or start a new terminal. This enables the mbed device to reset and start executing the program.

3. Connect to the device using an SSH client application, such as PuTTY. You can determine which serial port your device uses by checking the Windows Device Manager.


1. In PuTTY, click the **Serial** connection type. The device most likely connects at 115200, so enter that value in the **Speed** box. Then click **Open**:

    ![][11]

2. The program starts executing. You may have to reset the board (press CTRL+Break or press on the board's reset button) if the program does not start automatically when you connect.


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


[6]: ./media/iot-suite-connecting-devices-mbed/mbed1.png
[7]: ./media/iot-suite-connecting-devices-mbed/mbed2a.png
[8]: ./media/iot-suite-connecting-devices-mbed/mbed3a.png
[9]: ./media/iot-suite-connecting-devices-mbed/suite6.png
[11]: ./media/iot-suite-connecting-devices-mbed/mbed6.png

[lnk-mbed-home]: https://developer.mbed.org/platforms/FRDM-K64F/
[lnk-mbed-getstarted]: https://developer.mbed.org/platforms/FRDM-K64F/#getting-started-with-mbed
[lnk-mbed-pcconnect]: https://developer.mbed.org/platforms/FRDM-K64F/#pc-configuration

