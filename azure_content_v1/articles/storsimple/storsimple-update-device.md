<properties
   pageTitle="Update your StorSimple device | Microsoft Azure"
   description="Explains how to use the StorSimple update feature to install regular and maintenance mode updates and hotfixes."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="adinah"
   editor="" />

<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="01/13/2016"
   ms.author="v-sharos" />

# Update your StorSimple 8000 Series device
## Overview
The StorSimple updates features allow you to easily keep your StorSimple device up-to-date. Depending on the update type, you can apply updates to the device via the Azure classic portal or via the Windows PowerShell interface. This tutorial describes the update types and how to install each of them.

You can apply two types of device updates: 

* Regular (or Normal mode) updates
* Maintenance mode updates

You can install regular updates via the Azure classic portal or Windows PowerShell; however, you must use Windows PowerShell to install Maintenance mode updates. 

Each update type is described separately, below.

### Regular updates
Regular updates are non-disruptive updates that can be installed when the device is in Normal mode. These updates are applied through the Microsoft Update website to each device controller. 

> [!IMPORTANT]
> A controller failover may occur during the update process. However, this will not affect system availability or operation.
> 
> 
* For details on how to install regular updates via the Azure classic portal, see [Install regular updates via the Azure classic portal(#install-regular-updates-via-the-azure-classic-portal).

* You can also install regular updates via Windows PowerShell for StorSimple. For details, see [Install regular updates via Windows PowerShell for StorSimple](#install-regular-updates-via-windows-powershell-for-storsimple.md).


### Maintenance mode updates
Maintenance Mode updates are disruptive updates such as disk firmware upgrades or USM firmware upgrades. These updates require the device to be put into Maintenance mode. For details, see [Step 2: Enter Maintenance mode](#step2.md). You cannot use the Azure classic portal to install Maintenance mode updates. Instead, you must use Windows PowerShell for StorSimple. 

For details on how to install Maintenance mode updates, see [Install Maintenance mode updates via Windows PowerShell for StorSimple](#install-maintenance-mode-updates-via-windows-powershell-for-storsimple.md).

> [!IMPORTANT]
> Maintenance mode updates must be applied separately to each controller. 
> 
> 
## Install regular updates via the Azure classic portal
You can use the Azure classic portal to apply updates to your StorSimple device.

<!--author=SharS last changed: 9/17/15-->

#### To install regular updates via the Azure classic portal

1. On the **Devices** page, select the device on which you want to install updates.

2. Navigate to **Devices** > **Maintenance** and scroll down to **Software Updates**.

3. To check for updates, click **Check Updates** at the bottom of the page.

4. You will see a message if software updates are available. Click **Install Updates** to begin updating the device.

    You will be notified when the update is successfully installed.

## Install regular updates via Windows PowerShell for StorSimple
Alternatively, you can use Windows PowerShell for StorSimple to apply regular (Normal mode) updates.

> [!IMPORTANT]
> Although you can install regular updates using Windows PowerShell for StorSimple, we strongly recommend that you install regular updates through the Azure classic portal. Beginning with Update 1, pre-checks will be performed prior to installing updates from the portal. These pre-checks will preempt failures and ensure a smoother experience. 
> 
> 
<!--author=SharS last changed: 9/17/15-->

#### To install regular updates via Windows PowerShell for StorSimple

1. Open the device serial console and select option 1, **Log in with full access**. Type the password. The default password is *Password1*. 

2. At the command prompt, type:

     `Get-HcsUpdateAvailability`
    
    You will be notified if updates are available and whether the updates are disruptive or non-disruptive.

3. At the command prompt, type:

     `Start-HcsUpdate`

    The update process will start.

> [AZURE.IMPORTANT]
>
> - This command applies only to regular updates. You run this command on only one controller, but both controllers will be updated. 
> - You may notice a controller failover during the update process; however, the failover will not affect system availability or operation.


## Install Maintenance mode updates via Windows PowerShell for StorSimple
You use Windows PowerShell for StorSimple to apply Maintenance mode updates to your StorSimple device. All I/O requests are paused in this mode. Services such as non-volatile random access memory (NVRAM) or the clustering service are also stopped. Both controllers are rebooted when you enter or exit this mode. When you exit this mode, all the services will resume and should be healthy. (This may take a few minutes.)

If you need to apply Maintenance mode updates, you will receive an alert through the Azure classic portal that you have updates that must be installed. This alert will include instructions for using Windows PowerShell for StorSimple to install the updates. After you update your device, use the same procedure to change the device to Regular mode. For step-by-step instructions, see [Step 4: Exit Maintenance mode](#step4.md).

> [!IMPORTANT]
> 
> * Before entering Maintenance mode, verify that both device controllers are healthy by checking the **Hardware Status** on the **Maintenance** page in the Azure classic portal. If the controller is not healthy, contact Microsoft Support for the next steps. For more information, go to Contact Microsoft Support. 
> * When you are in Maintenance mode, you need to apply the update first on one controller and then on the other controller.
> 
> 
### Step 1: Connect to the serial console <a name="step1">
First, use an application such as PuTTY to access the serial console. The following procedure explains how to use PuTTY to connect to the serial console.

<!--author=SharS last changed: 9/17/15-->

#### To connect through the serial console

1. Connect your serial cable to the device (directly or through a USB-serial adapter).

2. Open the **Control Panel**, and then open the **Device Manager**.

3. Identify the COM port as shown in the following illustration.

     ![Connecting through serial console](./media/storsimple-use-putty/HCS_ConnectingDeviceS-include.png)

4. Start PuTTY. 

5. In the right pane, change the **Connection type** to **Serial**.

6. In the right pane, type the appropriate COM port. Make sure that the serial configuration parameters are set as follows:
  - Speed: 115,200
  - Data bits: 8
  - Stop bits: 1
  - Parity: None
  - Flow control: None

    These settings are shown in the following illustration.

     ![PuTTY settings](./media/storsimple-use-putty/HCS_PuttyConfig-include.png) 

    > [AZURE.NOTE] If the default flow control setting does not work, try setting the flow control to XON/XOFF.

7. Click **Open** to start a serial session.
 

### Step 2: Enter Maintenance mode <a name="step2">
After you connect to the console, determine whether there are updates to install, and enter Maintenance mode to install them.

<!--author=SharS last changed: 12/01/15-->

#### To enter Maintenance mode

1. In the serial console menu, choose option 1, **Log in with full access**.

2. Type the password. The default password is **Password1**.

3. At the command prompt, type

     `Enter-HcsMaintenanceMode`

4. You will see a warning message telling you that Maintenance mode will disrupt all I/O requests and sever the connection to the Azure classic portal, and you will be prompted for confirmation. Type **Y** to enter Maintenance mode.

    Both controllers will restart. When the restart is complete, another message will appear indicating that the device is in Maintenance mode.


### Step 3: Install your updates <a name="step3">
Next, install your updates.

<!--author=SharS last changed: 9/17/15-->

#### To install Maintenance mode updates via Windows PowerShell for StorSimple

1. If you haven't done so already, access the device serial console and select option 1, **Log in with full access**. 

2. Type the password. The default password is **Password1**.

3. At the command prompt, type:

     `Get-HcsUpdateAvailability` 
    
4. You will be notified if updates are available and whether the updates are disruptive or non-disruptive. To apply disruptive updates, you need to put the device into Maintenance mode. See [Step 2: Enter Maintenance mode](storsimple-update-device.md#step2) for instructions.

5. When your device is in Maintenance mode, at the command prompt, type: `Start-HcsUpdate`

6. You will be prompted for confirmation. After you confirm the updates, they will be installed on the controller that you are currently accessing. After the updates are installed, the controller will restart. 

7. Monitor the status of updates. Type:

	`Get-HcsUpdateStatus`
	
	If the `RunInProgress` is `True`, the update is still in progress. If `RunInProgress` is `False`, it indicates that the update has completed.  

7. When the update is installed on the current controller and it has restarted, connect to the other controller and perform steps 1 through 6.

8. After both controllers are updated, exit Maintenance mode. See [Step 4: Exit Maintenance mode](storsimple-update-device.md#step4) for instructions.


### Step 4: Exit Maintenance mode <a name="step4">
Finally, exit Maintenance mode.

<!--author=SharS last changed: 9/17/15-->

#### To exit Maintenance mode

1. At the command prompt type:

     `Exit-HcsMaintenanceMode`

2. A warning message and a confirmation message will appear. Type **Y** to exit Maintenance mode.

    Both controllers will restart. When the restart is complete, another message will appear indicating that the device is in Normal mode.

## Install hotfixes via Windows PowerShell for StorSimple
Unlike updates for Microsoft Azure StorSimple, hotfixes are installed from a shared folder. As with updates, there are two types of hotfixes: 

* Regular hotfixes 
* Maintenance mode hotfixes  

The following procedures explain how to use Windows PowerShell for StorSimple to install regular and Maintenance mode hotfixes.

<!--author=SharS last changed: 9/17/15-->

#### To install regular hotfixes via Windows PowerShell for StorSimple

1. Connect to the device serial console. For more information, see [Step 1: Connect to the serial console](storsimple-update-device.md#step1).

2. In the serial console menu, select option 1, **Log in with full access**. Type the password. The default password is **Password1**.

3. At the command prompt, type:

    `Start-HcsHotfix`

       >[AZURE.IMPORTANT]
       >
       >- This command applies only to regular hotfixes. You run this command on only one controller, but both controllers will be updated.
       >- You may notice a controller failover during the update process; however, the failover will not affect system availability or operation.

4. When prompted, supply the path to the network shared folder that contains the hotfix files.

5. You will be prompted for confirmation. Type **Y** to proceed with the hotfix installation.


<!--author=SharS last changed: 9/17/15-->

#### To install Maintenance mode hotfixes via Windows PowerShell for StorSimple

> [AZURE.IMPORTANT] In Maintenance mode, you need to apply the hotfix first on one controller and then on the other controller.

1. Place the device into Maintenance mode. See [Step 2: Enter Maintenance mode](storsimple-update-device.md#step2) for instructions on how to enter Maintenance mode.

2. To apply the hotfix, type:

     `Start-HcsHotfix` 

3. When prompted, supply the path to the network shared folder that contains the hotfix files.

4. You will be prompted for confirmation. Type **Y** to proceed with the hotfix installation.

5. After you have applied the hotfix on one controller, log on to the other controller. Apply the hotfix as you did for the previous controller.

6. After the hotfixes are applied, exit Maintenance mode. See [Step 4: Exit Maintenance mode](storsimple-update-device.md#step4) for instructions.


## What happens to updates if you perform a factory reset of the device?
If a device is reset to factory settings, then all the updates are lost. After the factory-reset device is registered and configured, you will need to manually install updates through the Azure classic portal and/or Windows PowerShell for StorSimple. For more information about factory reset, see [Reset the device to factory default settings](storsimple-manage-device-controller.md#reset-the-device-to-factory-default-settings).

## Next steps
* Learn more about [using Windows PowerShell for StorSimple to administer your StorSimple device](storsimple-windows-powershell-administration.md).
* Learn more about [using the StorSimple Manager service to administer your StorSimple device](storsimple-manager-service-administration.md).

