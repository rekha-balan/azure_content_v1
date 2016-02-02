<properties 
   pageTitle="Install Update 2 on your StorSimple device | Microsoft Azure"
   description="Explains how to install StorSimple 8000 Series Update 2 on your StorSimple 8000 series device."
   services="storsimple"
   documentationCenter="NA"
   authors="alkohli"
   manager="carmonm"
   editor="" />

<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="01/27/2016"
   ms.author="alkohli" />

# Install Update 2 on your StorSimple device
## Overview
This tutorial explains how to install Update 2 on a StorSimple device running an earlier software version via the Azure classic portal. The tutorial also covers the steps required for the update when a gateway is configured on a network interface other than DATA 0 of the StorSimple device and you are trying to update from a pre-Update 1 software version. 

Update 2 includes device software updates, LSI driver updates, and disk firmware updates. The device software and LSI updates are non-disruptive updates and can be applied via the Azure classic portal. The disk firmware updates are disruptive updates and can only be applied via the Windows PowerShell interface of the device. 

> [!IMPORTANT]
> 
> * You may not see Update 2 immediately because we do a phased rollout of the updates. Scan for updates in a few days again as this Update will become available soon.
> * A set of manual and automatic pre-checks are done prior to the install to determine the device health in terms of hardware state and network connectivity. These pre-checks are performed only if you apply the updates from the Azure classic portal. 
> * We recommend that you install the software and driver updates via the Azure  classic portal. You should only go to the Windows PowerShell interface of the device (to install updates) if the pre-update gateway check fails in the portal. The updates may take 4-7 hours to install (including the Windows Updates). The maintenance mode updates must be installed via the Windows PowerShell interface of the device. As maintenance mode updates are disruptive updates, these will result in a down time for your device.
> * If running the optional StorSimple Snapshot Manager, ensure that you have upgraded your Snapshot Manager version to Update 2 prior to updating the device.
> 
> 
## Preparing for updates
You will need to perform the following steps before you scan and apply the update:

1. Take a cloud snapshot of the device data.

2. Ensure that your controller fixed IPs are routable and can connect to the Internet. These fixed IPs will be used to service updates to your device. You can test this by running the following cmdlet on each controller from the Windows PowerShell interface of the device:

     `Test-Connection -Source <Fixed IP of your device controller> -Destination <Any IP or computer name outside of datacenter network> `

    **Sample output for Test-Connection when fixed IPs can connect to the Internet**


        Controller0>Test-Connection -Source 10.126.173.91 -Destination bing.com

        Source      Destination     IPV4Address      IPV6Address
        ----------------- -----------  -----------
        HCSNODE0  bing.com        204.79.197.200
        HCSNODE0  bing.com        204.79.197.200
        HCSNODE0  bing.com        204.79.197.200
        HCSNODE0  bing.com        204.79.197.200

        Controller0>Test-Connection -Source 10.126.173.91 -Destination  204.79.197.200

        Source      Destination       IPV4Address    IPV6Address
        ----------------- -----------  -----------
        HCSNODE0  204.79.197.200  204.79.197.200
        HCSNODE0  204.79.197.200  204.79.197.200
        HCSNODE0  204.79.197.200  204.79.197.200
        HCSNODE0  204.79.197.200  204.79.197.200

After you have successfully completed these manual pre-checks, you can proceed to scan and install the updates.

## Install Update 2 via the Azure classic portal
This is the recommended procedure to update your device. Perform the following steps.

<!--author=alkohli last changed: 01/26/16-->

#### To install Update 2 from the Azure portal

1. On the StorSimple service page, select your device. Navigate to **Devices** > **Maintenance**.

2. At the bottom of the page, click **Scan Updates**. A job will be created to scan for available updates. You will be notified when the job has completed successfully.

3. In the **Software Updates** section on the same page, you will see that new software updates are available. We recommend that you review the release notes before you apply Update 2 on your device.

    ![Install software updates](./media/storsimple-install-update2-via-portal/scanupdate1.png)

4. At the bottom of the page, click **Install Updates**.

5. You will be prompted for confirmation. Click **OK**.

6. An **Install Updates** dialog box will be presented. Your device should satisfy the checks listed in this dialog box. These steps were completed prior to the update. Select **I understand the above requirement and am ready to update my device**. Click the check icon.

    ![Confirmation message](./media/storsimple-install-update2-via-portal/InstallUpdate12_2M.png)

7. A set of automatic pre-checks will now start. These include:

	- **Controller health checks** to verify that both the device controllers are healthy and online.
	
	- **Hardware component health checks** to verify that all the hardware components on your StorSimple device are healthy.
	
	- **DATA 0 checks** to verify that DATA 0 is enabled on your device. If this interface is not enabled, you will need to enable it and then retry.
	
	- **DATA 2 and DATA 3 checks** to verify that DATA 2 and DATA 3 network interfaces are not enabled. If these interfaces are enabled, then you will need to disable them and then try to update your device. This check is performed only if you are updating from a device running GA software. Devices running versions 0.1, 0.2, or 0.3 will not need this check.
	
	- **Gateway check** on any device running a version prior to Update 1. This check is performed on all the device running pre-update 1 software but fails on the devices that have a gateway configured for a network interface other than DATA 0.
 
	Update 2 will only be applied if all the above pre-update checks are successfully completed. You will be notified that pre-update checks are in progress.
  
    ![Pre-check notification](./media/storsimple-install-update2-via-portal/InstallUpdate12_3M.png)

    The following is an example in which the pre-upgrade check failed. You will need to verify that both the device controllers are healthy and online. You will also need to check the health of the hardware components. In this example, Controller 0 and Controller 1 components need attention. You may need to contact Microsoft Support if you cannot address these issues by yourself.

   	 ![Pre-check failed](./media/storsimple-install-update2-via-portal/HCS_PreUpgradeChecksFailed-include.png)

	
	> [AZURE.NOTE] If you are updating from a pre-Update 1 software, after you have applied Update 2 on your StorSimple device, DATA 2 and DATA 3 checks and the gateway check will no longer be necessary for the future updates. The other pre-checks will still be required. If you updated from Update 1 or later, the DATA 2, DATA 3, and gateway pre-checks are not performed.


8. After the pre-upgrade checks are successfully completed, an update job will be created. You will be notified when the update job is successfully created.
 
    ![Update job creation](./media/storsimple-install-update2-via-portal/InstallUpdate12_44M.png)

    The update will then be applied on your device.
 
9. To monitor the progress of the update job, click **View Job**. On the **Jobs** page, you can see the update progress. 
    
10. The update will take a few hours to complete. Select the update job and click **Details** to view the details of the job at any time.
  
11. After the job is complete, navigate to the **Maintenance** page and scroll down to **Software Updates**.

12. Verify that your device is running **StorSimple 8000 Series Update 2 (6.3.9600.17673)**. The **Last updated date** should also be modified.


13. You will now see that Maintenance mode updates are available. In some cases when you are running Update 1.2, your disk firmware may already be up-to-date. In these instances, the portal will automatically determine that and not prompt you for the maintenance mode updates. 

	The maintenance mode updates are disruptive updates that result in device downtime and can only be applied via the Windows PowerShell interface of your device. Follow the steps listed in [install and verify maintenance mode hotfix](#to-install-and-verify-maintenance-mode-hotfix) to install these Maintenance mode updates.

> [AZURE.NOTE] In certain instances, the message indicating maintenance mode updates are available may be displayed up to 24 hours after the maintenance mode updates are successfully applied on the device.  




## Install Update 2 as a hotfix
Use this procedure only if you fail the gateway check when trying to install the updates through the Azure classic portal. The check fails as you have a gateway assigned to a non-DATA 0 network interface and your device is running a software version prior to Update 1. 

The software versions that can be upgraded using the hotfix method are Update 0.1, Update 0.2, and Update 0.3, Update 1, Update 1.1, and Update 1.2. The hotfix method involves the following three steps:

* Download the hotfixes from the Microsoft Update Catalog.
* Install and verify the regular mode hotfixes.
* Install and verify the maintenance mode hotfix.

The hotfixes applied through this method are as tabulated below:

| Order | KB | Name | Update type |
| --- | --- | --- | --- |
| 1 |KB3121901 |Software update |Regular |
| 2 |KB3121900 |LSI driver |Regular |
| 3 |KB3080728 |Storport fix |Regular |
| 4 |KB3090322 |Spaceport fix |Regular |
| 5 |KB3121899 |Disk firmware |Maintenance |

> [!IMPORTANT]
> 
> * If your device is running Release (GA) version, please contact [Microsoft Support](storsimple-contact-microsoft-support.md) to assist you with the update.
> * This procedure needs to be performed only once to apply Update 2. You can use the Azure classic portal to apply subsequent updates.
> * Each hotfix installation can take about 20 minutes to complete. Total install time is close to 2 hours. 
> * Before using this procedure to apply the update, make sure that both device controllers are online and all the hardware components are healthy.
> 
> 
Perform the following steps to apply Update 2 as a hotfix. 

<!--author=alkohli last changed: 01/26/16-->

#### To download hotfixes

Perform the following steps to download the software update from the Microsoft Update Catalog.

1. Start Internet Explorer and navigate to [http://catalog.update.microsoft.com/v7/site/Home.aspx](http://catalog.update.microsoft.com/v7/site/Home.aspx).

2. If you are a first-time user, you will be prompted to install a Microsoft Update Catalog. Click **Install**.
    
   	![Install catalog](./media/storsimple-install-update2-hotfix/HCS_InstallCatalog-include.png)

3. You will see a catalog search screen. Enter **3121901** in the search box, and click **Search**.

    ![Search catalog](./media/storsimple-install-update2-hotfix/HCS_SearchCatalog1-include.png)

4. You will see the **Cumulative Software Bundle Update 2.0 for StorSimple 8000 Series**. Click **Add**. The update will be added to the basket. 

5. Click **View Basket**.
 
6. Click **Download**. Specify or **Browse** to a local location where you want the download to appear. The update will be downloaded in a folder (same name as the update) to the chosen location. The folder can also be copied to a network share that is reachable from the device. 
       
	> [AZURE.NOTE] 
	> 
	> - You will also need to download **LSI driver update** (SAS Controller Update 2.0 for StorSimple 8000 Series - KB3121900),  **Storport update** (Hotfix for Windows Server 2012 R2 x64 Edition - KB3080728), **Spaceport update** (Hotfix for Windows Server 2012 R2 x64 Edition - KB3090322), and **Disk firmware update** (Cumulative Disk Firmware Update 2.0 for StorSimple 8000 Series - KB3121899) and copy to the same shared folder.
	> - The hotfix must be accessible from both controllers to detect any potential error messages from the peer controller.

#### To install and  verify regular mode hotfixes

Perform the following steps to install and verify the regular hotfixes.

1. To install the hotfixes, access the Windows PowerShell interface on your StorSimple device serial console. Follow the detailed instructions in [Use PuTTy to connect to the serial console](storsimple-deployment-walkthrough.md#use-putty-to-connect-to-the-device-serial-console). At the command prompt, press **Enter**.

4. Select **Option 1** to log on to the device with full access.

5. To install the hotfix, at the command prompt, type:

    `Start-HcsHotfix -Path <path to update file> -Credential <credentials in domain\username format>`

    Use IP rather than DNS in share path in the above command. The credential parameter is used only if you are accessing an authenticated share. 

	We recommend that you use the credential parameter to access shares. Even shares that are open to “everyone” are typically not open to unauthenticated users.

    A sample output is shown below.

        ````
        Controller0>Start-HcsHotfix -Path \\10.100.100.100\share
        \hcsmdssoftwareupdate.exe -Credential contoso\John
      
        Confirm

        This operation starts the hotfix installation and could reboot one or
        both of the controllers. If the device is serving I/Os, these will not 
        be disrupted. Are you sure you want to continue?
        [Y] Yes [N] No [?] Help (default is "Y"): Y

        ````
 
6. Type **Y** when prompted to confirm the hotfix installation.

7. Monitor the update by using the `Get-HcsUpdateStatus` cmdlet.

    The following sample output shows the update in progress. The `RunInprogress` will be `True` when the update is in progress.

        ````
        Controller0>Get-HcsUpdateStatus
        RunInprogress       : True
        LastHotfixTimestamp : 12/21/2015 10:36:13 PM
        LastUpdateTimestamp : 12/21/2015 10:35:25 PM
        Controller0Events   :
        Controller1Events   : 
        ````
 
     The following sample output indicates that the update is finished. The `RunInProgress` will be `False` when the update has completed.

        ````
        Controller1>Get-HcsUpdateStatus

        RunInprogress       : False
        LastHotfixTimestamp : 12/21/2015 10:59:13 PM
        LastUpdateTimestamp : 12/21/2015 10:35:25 PM
        Controller0Events   :
        Controller1Events   :

        ````
		

	> [AZURE.NOTE] Occasionally, the cmdlet reports `False` when the update is still in progress. To ensure that the hotfix is complete, wait for a few minutes, rerun this command and verify that the `RunInProgress` is `False`. If it is, then the hotfix has completed. 
	
8. After the software update is complete, repeat steps 3-5 to install and monitor the SaaS agent and MDS agent . Ensure that `all-hcsmdssoftwareupdate_0b438ddf0d5b686aada2378b754fac8c7f2160e9.exe` is installed before `all-cismdsagentupdatebundle_f98e62f4d56c79e2a6644d027af7a2393a93827a.exe`. 

9. Verify the system software versions. Type:

    `Get-HcsSystem`

    You should see the following versions:

    - HcsSoftwareVersion: 6.3.9600.17673
    - CisAgentVersion: 1.0.9150.0
    - MdsAgentVersion: 30.0.4698.13 
    
	If the version numbers do not change after applying the update, it indicates that the hotfix has failed to apply. Should you see this, please contact [Microsoft Support](storsimple-contact-microsoft-support.md) for further assistance.
    
9. Repeat steps 3-5 to install and monitor the remaining regular hotfixes.

	- The LSI driver using KB3121900.
	- The Storport fix using KB3080728.
	- The Spaceport fix using KB3090322.

#### To install and verify maintenance mode hotfix

Use KB3121899 to install disk firmware updates. These are disruptive updates and take around 30 minutes to complete. You can choose to install these in a planned maintenance window by connecting to the device serial console. 

Note that if your disk firmware is already up-to-date, you will not need to install these updates. Run the `Get-HcsUpdateAvailability` cmdlet from the device serial console. You will be notified if updates are available and whether the updates are disruptive (maintenance mode updates) or non-disruptive (regular).
 
To install the disk firmware updates, follow the instructions below.

1. Place the device in the Maintenance mode. Note that you should not use Windows PowerShell remoting when connecting to a device in Maintenance mode. You will need to run this cmdlet on the device controller when connected through the device serial console. Type:
		
	`Enter-HcsMaintenanceMode`

	A sample output is shown below.

		Controller0>Enter-HcsMaintenanceMode
		Checking device state...

		In maintenance mode, your device will not service IOs and will be disconnected from the Microsoft Azure StorSimple Manager service. Entering maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to enter maintenance mode?
		[Y] Yes [N] No (Default is "Y"): Y

		-----------------------MAINTENANCE MODE------------------------
		Microsoft Azure StorSimple Appliance Model 8100
		Name: Update2-8100-SHG0997879L76YD
		Software Version: 6.3.9600.17664
		Copyright (C) 2014 Microsoft Corporation. All rights reserved.
		You are connected to Controller0 - Passive
		---------------------------------------------------------------

		Serial Console Menu
		[1] Log in with full access
		[2] Log into peer controller with full access
		[3] Connect with limited access
		[4] Change language
		Please enter your choice>

	Both the controllers will be rebooted. After the reboot is complete, both controllers will be in the Maintenance mode. 

3. To install the disk firmware update, type:

	`Start-HcsHotfix -Path <path to update file> -Credential <credentials in domain\username format>`

	A sample output is shown below.

        Controller1>Start-HcsHotfix -Path \\10.100.100.100\share\DiskFirmwarePackage.exe -Credential contoso\john
		Enter Password:
		WARNING: In maintenance mode, hotfixes should be installed on each controller sequentially. After the hotfix is installed on this controller, install it on the peer controller.
		Confirm
		This operation starts a hotfix installation and could reboot one or both of the controllers. By installing new updates you agree to, and accept any additional terms associated with, the new functionality listed in the release notes (https://go.microsoft.com/fwLink/?LinkID=613790). Are you sure you want to continue?
		[Y] Yes [N] No (Default is "Y"): Y
		WARNING: Installation is currently in progress. This operation can take several minutes to complete.
	

1.  Monitor the install progress using `Get-HcsUpdateStatus` command. The update is complete when the `RunInProgress` changes to `False`.
 
2.  After the installation is complete, the controller on which the maintenance mode hotfix was installed will be rebooted. Log in as option 1 with full access and verify the disk firmware version. Type:
	
	`Get-HcsFirmwareVersion`
  
	The expected disk firmware versions are: 

	`XMGG, XGEG, KZ50, F6C2, VR08`

	A sample output is shown below.


        -----------------------MAINTENANCE MODE------------------------
    	Microsoft Azure StorSimple Appliance Model 8100
    	Name: Update2-8100-SHG0997879L76YD
    	Software Version: 6.3.9600.17664
    	Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    	You are connected to Controller1
    	---------------------------------------------------------------
    	
    	Controller1>Get-HcsFirmwareVersion
    	
    	
    	Controller0 : TalladegaFirmware
    	  ActiveBIOS:0.45.0006
    	  BackupBIOS:0.45.0008
    	  MainCPLD:17.0.0005
    	  ActiveBMCRoot:2.0.000E
    	  BackupBMCRoot:2.0.000E
    	  BMCBoot:2.0.0001
    	  LsiFirmware:19.00.00.00
    	  LsiBios:07.37.00.00
    	  Battery1Firmware:06.29
    	  Battery2Firmware:06.29
    	  DomFirmware:X231600
    	  CanisterFirmware:3.5.0.32
    	  CanisterBootloader:5.03
    	  CanisterConfigCRC:0xD1B030A4
    	  CanisterVPDStructure:0x06
    	  CanisterGEMCPLD:0x17
    	  CanisterVPDCRC:0xEE3504B4
    	  MidplaneVPDStructure:0x0C
    	  MidplaneVPDCRC:0xA6BD4F64
    	  MidplaneCPLD:0x10
    	  PCM1Firmware:1.00|1.05
    	  PCM1VPDStructure:0x05
    	  PCM1VPDCRC:0x41BEF99C
    	  PCM2Firmware:1.00|1.05
    	  PCM2VPDStructure:0x05
    	  PCM2VPDCRC:0x41BEF99C
    	
    	  DisksFirmware
    	  SEAGATE:ST400FM0073:XGEG
    	  SEAGATE:ST400FM0073:XGEG
    	  SEAGATE:ST400FM0073:XGEG
    	  SEAGATE:ST400FM0073:XGEG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG
    	  SEAGATE:ST4000NM0023:XMGG

	 Run the `Get-HcsFirmwareVersion` command on the second controller to verify that the software version has been updated. You can then exit the maintenance mode. Type the following command for each device controller: 

    `Exit-HcsMaintenanceMode`
     
1. The controllers will be rebooted when you exit the Maintenance mode. After the disk firmware updates are successfully applied and the device has exited maintenance mode, return to the Azure classic portal. Maintenance mode updates are not updated on the portal until 24 hours have elapsed. 






 
 



## Troubleshooting update failures
**What if you see a notification that the pre-upgrade checks have failed?**

If a pre-check fails, make sure that you have looked at the detailed notification bar at the bottom of the page. This provides guidance as to which pre-check has failed. The following illustration shows an instance in which such a notification appears. In this case, the controller health check and hardware component health check have failed. Under the **Hardware Status** section, you can see that both **Controller 0** and **Controller 1** components need attention. 

  ![Pre-check failure](./media/storsimple-install-update-2/HCS_PreUpdateCheckFailed-include.png)

You will need to make sure that both controllers are healthy and online. You will also need to make sure that all the hardware components in the StorSimple device are shown to be healthy on the Maintenance page. You can then try to install updates. If you are not able to fix the hardware component issues, then you will need to contact Microsoft Support for next steps.

**What if you receive a "Could not install updates" error message, and the recommendation is to refer to the update troubleshooting guide to determine the cause of the failure?**

One likely cause for this could be that you do not have connectivity to the Microsoft Update servers. This is a manual check that needs to be performed. If you lose connectivity to the update server, your update job would fail. You can check the connectivity by running the following cmdlet from the Windows PowerShell interface of your StorSimple device:

 `Test-Connection -Source <Fixed IP of your device controller> -Destination <Any IP or computer name outside of datacenter>`

Run the cmdlet on both controllers.

If you have verified the connectivity exists, and you continue to see this issue, please contact Microsoft Support for next steps.

## Next steps
Learn more about the [Update 2 release](storsimple-update2-release-notes.md).

