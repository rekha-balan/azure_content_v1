<properties 
   pageTitle="StorSimple virtual device Update 2| Microsoft Azure"
   description="Learn how to create, deploy, and manage a StorSimple virtual device in a Microsoft Azure virtual network. (Applies to StorSimple Update 2)."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carmonm"
   editor="" />

<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="01/22/2016"
   ms.author="alkohli" />

# Deploy and manage a StorSimple virtual device in Azure (Update 2)
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Update 2](../articles/storsimple/storsimple-virtual-device-u2.md)
> * [Update 1](../articles/storsimple/storsimple-virtual-device-u1.md)
> * [GA Release](../articles/storsimple/storsimple-virtual-device.md)
> 
> 
## Overview
The StorSimple virtual device is an additional capability that comes with your Microsoft Azure StorSimple solution. The StorSimple virtual device runs on a virtual machine in a Microsoft Azure virtual network, and you can use it to back up and clone data from your hosts. 

#### Virtual device model comparison
The StorSimple virtual device is available in two models, a standard 8010 and a premium 8020 (introduced in Update 2). A comparison of the two models is tabulated below.

| Device model | 8010<sup>1</sup> | 8020 |
| --- | --- | --- |
| **Maximum capacity** |30 TB |64 TB |
| **Azure VM** |Standard_A3 (4 cores, 7 GB memory) |Standard_DS3 (4 cores, 14 GB memory) |
| **Version compatibility** |Versions running pre-Update 2 or later |Versions running Update 2 or later |
| **Region availability** |All Azure regions |Azure regions that support Premium Storage<br></br>For a list of regions that currently support Premium Storage, see [Azure Services by Region](https://azure.microsoft.com/regions/#services) |
| **Storage type** |Uses Azure Standard Storage<br></br> Learn how to [create a Standard Storage account]() |Uses Azure Premium Storage<br></br>Learn how to [create a Premium Storage account](storage-premium-storage-preview-portal.md#create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk) |
| **Workload guidance** |Item level retrieval of files from backups |Cloud dev and test scenarios, Low latency, higher performance workloads <br></br>Secondary device for disaster recovery |

<sup>1</sup> *Formerly known as 1100*.

This article describes the step-by-step process of deploying a StorSimple virtual device in Azure. After reading this article, you will:

* Understand how the virtual device differs from the physical device.

* Be able to create and configure the virtual device.

* Connect to the virtual device.

* Learn how to work with the virtual device.


This tutorial applies to all the StorSimple virtual devices running Update 2 and higher. 

## How the virtual device differs from the physical device
The StorSimple virtual device is a software-only version of StorSimple that runs on a single node in a Microsoft Azure Virtual Machine. The virtual device supports disaster recovery scenarios in which your physical device is not available, and is appropriate for use in item-level retrieval from backups, on-premises disaster recovery, and cloud dev and test scenarios.

#### Differences from the physical device
The following table shows some key differences between the StorSimple virtual device and the  StorSimple physical device.

|  | Physical device | Virtual device |
| --- | --- | --- |
| **Location** |Resides in the datacenter. |Runs in Azure. |
| **Network interfaces** |Has six network interfaces: DATA 0 through DATA 5. |Has only one network interface: DATA 0. |
| **Registration** |Registered during the configuration step. |Registration is a separate task. |
| **Service data encryption key** |Regenerate on the physical device and then update the virtual device with the new key. |Cannot regenerate from the virtual device. |

## Prerequisites for the virtual device
The following sections explain the configuration prerequisites for your StorSimple virtual device. Prior to deploying a virtual device, review the [security considerations for using a virtual device](storsimple-security.md#storsimple-virtual-device-security).

#### Azure requirements
Before you provision the virtual device, you need to make the following preparations in your Azure environment:

* For the virtual device, [configure a virtual network on Azure](../virtual-network/virtual-networks-create-vnet-classic-portal.md). If using Premium Storage, you must create a virtual network in an Azure region that supports Premium Storage. More information on [regions that currently support Premium Storage](https://azure.microsoft.com/regions/#services).
* It is advisable to use the default DNS server provided by Azure instead of specifying your own DNS server name. If your DNS server name is not valid or if the DNS server is not able to resolve IP addresses correctly, the creation of the virtual device will fail.
* Point-to-site and site-to-site are optional, but not required. If you wish, you can configure these options for more advanced scenarios. 
* You can create [Azure Virtual Machines](../virtual-machines/virtual-machines-about.md) (host servers) in the virtual network that can use the volumes exposed by the virtual device. These servers must meet the following requirements:                             

  * Be Windows or Linux VMs with iSCSI Initiator software installed.
* Be running in the same virtual network as the virtual device.
* Be able to connect to the iSCSI target of the virtual device through the internal IP address of the virtual device.

* Make sure you have configured support for iSCSI and cloud traffic on the same virtual network.


#### StorSimple requirements
Make the following updates to your Azure StorSimple service before you create a virtual device:

* Add [access control records](storsimple-manage-acrs.md) for the VMs that are going to be host servers for your virtual device.

* Use a [storage account](storsimple-manage-storage-accounts.md#add-a-storage-account) in the same region as the virtual device. Storage accounts in different regions may result in poor performance. You can use a Standard or Premium Storage account with the virtual device. More information on how to create a [Standard Storage account]() or a [Premium Storage account](storage-premium-storage-preview-portal.md#create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk)

* Use a different storage account for virtual device creation from the one used for your data. Using the same storage account may result in poor performance.


Make sure that you have the following information before you begin:

* Your Azure classic portal account with access credentials.

* A copy of the service data encryption key from your physical device.


## Create and configure the virtual device
Before performing these procedures, make sure that you have met the [Prerequisites for the virtual device](#prerequisites-for-the-virtual-device.md). 

After you have created a virtual network, configured a StorSimple Manager service, and registered your physical StorSimple device with the service, you can use the following steps to create and configure a StorSimple virtual device.

### Step 1: Create a virtual device
Perform the following steps to create the StorSimple virtual device.

#### To create a virtual device

1.  In the Azure portal, go to the **StorSimple Manager** service.

2. Go to the **Devices** page. Click **Create virtual device** at the bottom of the **Devices** page.

3. In the **Create Virtual Device dialog box**, specify the following details.

     ![StorSimple create virtual device](./media/storsimple-create-virtual-device-u2/CreatePremiumsva1.png)

	1. **Name** – A unique name for your virtual device.


	2. **Model** - Choose the model of the virtual device. This field is presented only if you are running Update 2 or later. An 8010 device model offers 30 TB of Standard Storage whereas 8020 has 64 TB of Premium Storage. Specify 8010
	3.  to deploy item level retrieval  scenarios from backups. Select 8020 to deploy high performance, low latency workloads or used as a secondary device for disaster recovery.
	 
	4. **Version** - Choose the version of the virtual device. If an 8020 device model is selected, then the version field will not be presented to the user. This option is absent if all the physical devices registered with this service are running Update 1 (or later). This field is presented only if you have a mix of pre-Update 1 and Update 1 physical devices registered with the same service. Given the version of the virtual device will determine which physical device you can failover or clone from, it is important that you create an appropriate version of the virtual device. Select:

	   - Version Update 0.3 if you will fail over or DR from a physical device running Update 0.3 or earlier. 
	   - Version Update 1 if you will fail over or clone from a physical device running Update 1 (or later). 
	   
	
	5. **Virtual Network** – Specify a virtual network that you want to use with this virtual device. If using Premium Storage (Update 2 or later), you must select a virtual network that is supported with the Premium Storage account. The unsupported virtual networks will be grayed out in the dropdown list. You will be warned if you select an unsupported virtual network. 

	5. **Storage Account for Virtual Device Creation** – Select a storage account to hold the image of the virtual device during provisioning. This storage account should be in the same region as the virtual device and virtual network. It should not be used for data storage by either the physical or the virtual device. By default, a new storage account will be created for this purpose. However, if you know that you already have a storage account that is suitable for this use, you can select it from the list. If creating a premium virtual device, the dropdown list will only display Premium Storage accounts. 

    	>[AZURE.NOTE] The virtual device can only work with the Azure storage accounts. Other cloud service providers such as Amazon, HP, and OpenStack (that are supported for the physical device) are not supported for the StorSimple virtual device.
	
	1. Click the check mark to indicate that you understand that the data stored on the virtual device will be hosted in a Microsoft datacenter. When you use only a physical device, your encryption key is kept with your device; therefore, Microsoft cannot decrypt it. 
	 
		When you use a virtual device, both the encryption key and the decryption key are stored in Microsoft Azure. For more information, see [security considerations for using a virtual device](storsimple-security/#storsimple-virtual-device-security).
	2. Click the check icon to create the virtual device. The device may take around 30 minutes to be provisioned.

	![StorSimple virtual device creating stage](./media/storsimple-create-virtual-device-u2/StorSimple_VirtualDeviceCreating1M.png)

    


### Step 2: Configure and register the virtual device
Before starting this procedure, make sure that you have a copy of the service data encryption key. The service data encryption key was created when you configured your first StorSimple device and you were instructed to save it in a secure location. If you do not have a copy of the service data encryption key, you must contact Microsoft Support for assistance.

Perform the following steps to configure and register your StorSimple virtual device.
<!---author: alkohli, last updated: 11/05/2015 --->

#### To configure and register the virtual device

1. Select the StorSimple virtual device you just created in the **Devices** page. 

2. Click **complete device setup**. This starts the Configure device wizard.

    ![StorSimple complete device setup in Devices page](./media/storsimple-configure-register-virtual-device/StorSimple_CompleteDeviceSetupSVA1M.png)

3. Enter the **Service Data Encryption Key** in the space provided.

4. Enter the Snapshot Manager and Device Administrator passwords of the length and settings specified.

5. Click the check mark to finish the initial configuration and registration of the virtual device. 

    ![StorSimple virtual device settings](./media/storsimple-configure-register-virtual-device/StorSimple_VirtualDeviceSettings1.png)

After the configuration and registration is complete, the device will come online. (It may take several minutes for the device to come online.)

![StorSimple virtual device online stage](./media/storsimple-configure-register-virtual-device/StorSimple_VirtualDeviceOnline1M.png)

### Step 3: (Optional) Modify the device configuration settings
The following section describes the device configuration settings needed for the StorSimple virtual device if you want to use CHAP, StorSimple Snapshot Manager or change the Device Administrator password.

#### Configure the CHAP initiator
This parameter contains the credentials that your virtual device (target) expects from the initiators (servers) that are attempting to access the volumes. The initiators will provide a CHAP user name and a CHAP password to identify themselves to your device during this authentication. For detailed steps, go to [Configure CHAP for your device](storsimple-configure-chap.md#unidirectional-or-one-way-authentication).

#### Configure the CHAP target
This parameter contains the credentials that your virtual device uses when a CHAP-enabled initiator requests mutual or bi-directional authentication. Your virtual device will use a Reverse CHAP user name and Reverse CHAP password to identify itself to the initiator during this authentication process. Note that CHAP target settings are global settings. When these are applied, all the volumes connected to the storage virtual device will use CHAP authentication. For detailed steps, go to [Configure CHAP for your device](storsimple-configure-chap.md#bidirectional-or-mutual-authentication).

#### Configure the StorSimple Snapshot Manager password
StorSimple Snapshot Manager software resides on your Windows host and allows administrators to manage backups of your StorSimple device in the form of local and cloud snapshots.

> [!NOTE]
> For the virtual device, your Windows host is an Azure virtual machine.
> 
> 
When configuring a device in the StorSimple Snapshot Manager, you will be prompted to provide the StorSimple device IP address and password to authenticate your storage device. For detailed steps, go to [Configure StorSimple Snapshot Manager password](storsimple-change-passwords.md#change-the-storsimple-snapshot-manager-password).

#### Change the device administrator password
When you use the Windows PowerShell interface to access the virtual device, you will be required to enter a device administrator password. For the security of your data, you are required to change this password before the virtual device can be used. For detailed steps, go to [Configure device administrator password](storsimple-change-passwords.md#change-the-device-administrator-password).

## Connect remotely to the virtual device
Remote access to your virtual device via the Windows PowerShell interface is not enabled by default. You need to enable remote management on the virtual device first, and then enable it on the client that will be used to access your virtual device.

The two-step process to connect remotely is detailed below.

### Step 1: Configure remote management
Perform the following steps to configure remote management for your StorSimple virtual device.


####To configure remote management on the device

1. On your virtual device, go to **Devices > Configure**.

2. Scroll down to the **Remote Management** section.

3. Set **Enable Remote Management** to **Yes**.

4. You can now choose to connect using HTTP. The default is to connect over HTTPS. Connecting over HTTP is acceptable only on trusted networks.

5. Click **Download Remote Management Certificate** to download a remote management certificate. You will specify a location in which to save this file. This certificate then needs to be installed on the client or host machine that you will use to connect to the virtual device.

6. Click **Save** at the bottom of the page.

### Step 2: Remotely access the virtual device
After you have enabled remote management on the StorSimple device configuration page, you can use Windows PowerShell remoting to connect to the virtual device from another virtual machine inside the same virtual network; for example, you can connect from the host VM that you configured and used to connect iSCSI. In most deployments, you will have already opened a public endpoint to access your host VM that you can use for accessing the virtual device.

> [!WARNING]
> **For enhanced security, we strongly recommend that you use HTTPS when connecting to the endpoints and then delete the endpoints after you have completed your PowerShell remote session.**
> 
> 
You should follow the procedures in [Connecting remotely to your StorSimple device](storsimple-remote-connect.md) to set up remoting for your virtual device.

## Connect directly to the virtual device
You can also connect directly to the virtual device. If you want to connect directly to the virtual device from another computer outside the virtual network or outside the Microsoft Azure environment, you need to create additional endpoints as described in the following procedure. 

Perform the following steps to create a public endpoint on the virtual device.

#### To create public endpoints on the virtual device

1. Sign in to the Azure classic portal.

- Click **Virtual Machines**, and then select the virtual machine that is being used as your virtual device.

- Click **Endpoints**. The **Endpoints** page lists all endpoints for the virtual machine.

- Click **Add**. The **Add Endpoint** dialog box appears. Click the arrow to continue.

- For the **Name**, type the following name for the endpoint: **WinRMHttps**.

- For the **Protocol**, specify **TCP**.

- For the **Public Port**, type the port numbers that you want to use for the connection.

- For the **Private Port**, type **5986**.

- Click the check mark to create the endpoint.

After the endpoint is created, you can view its details to determine the Public Virtual IP (VIP) address. Record this address.

We recommend that you connect from another virtual machine inside the same virtual network because this practice minimizes the number of public endpoints on your virtual network. When you use this method, you simply connect to the virtual machine through a Remote Desktop session and then configure that virtual machine for use as you would any other Windows client on a local network. You do not need to append the public port number because the port will already be known.

## Work with the StorSimple virtual device
Now that you have created and configured the StorSimple virtual device, you are ready to start working with it. You can work with volume containers, volumes, and backup policies on a virtual device just as you would on a physical StorSimple device; the only difference is that you need to make sure that you select the virtual device from your device list. Refer to [use the StorSimple Manager service to manage a virtual device](storsimple-manager-service-administration.md) for step-by-step procedures of the various management tasks for the virtual device.

The following sections discuss some of the differences you will encounter when working with the virtual device.

### Maintain a StorSimple virtual device
Because it is a software-only device, maintenance for the virtual device is minimal when compared to maintenance for the physical device. You have the following options:

* **Software updates** – You can view the date that the software was last updated, together with any update status messages. You can use the **Scan updates** button at the bottom of the page to perform a manual scan if you want to check for new updates. If updates are available, click **Install Updates** to install. Because there is only a single interface on the virtual device, this means that there will be a slight service interruption when updates are applied. The virtual device will shut down and restart (if necessary) to apply any updates that have been released. For a step-by-step procedure, go to [update your device](storsimple-update-device.md#install-regular-updates-via-the-azure-classic-portal).
* **Support package** – You can create and upload a support package to help Microsoft Support troubleshoot issues with your virtual device. For a step-by-step procedure, go to [create and manage a support package](storsimple-create-manage-support-package.md).

### Storage accounts for a virtual device
Storage accounts are created for use by the StorSimple Manager service, by the virtual device, and by the physical device. When you create your storage accounts, we recommend that you use a region identifier in the friendly name to help ensure that the region is consistent throughout all of the system components. For a virtual device, it is important that all of the components be in the same region to prevent performance issues.

For a step-by-step procedure, go to [add a storage account](storsimple-manage-storage-accounts.md#add-a-storage-account).

### Deactivate a StorSimple virtual device
Deactivating a virtual device deletes the VM and the resources created when it was provisioned. After the virtual device is deactivated, it cannot be restored to its previous state. Before you deactivate the virtual device, make sure to stop or delete clients and hosts that depend on it.

Deactivating a virtual device results in the following actions:

* The virtual device is removed.

* The OS disk and data disks created for the virtual device are removed.

* The hosted service and virtual network created during provisioning are retained. If you are not using them, you should delete them manually.

* Cloud snapshots created for the virtual device are retained.


For a step-by-step procedure, go to [Deactivate and delete your StorSimple device](storsimple-deactivate-and-delete-device.md).

As soon as the virtual device is shown as deactivated on the StorSimple Manager service page, you can delete the virtual device from device list on the **Devices** page.

### Start, stop and restart a virtual device
Unlike the StorSimple physical device, there is no power on or power off button to push on a StorSimple virtual device. However, there may be occasions where you need to stop and restart the virtual device. For example, some updates might require that the VM be restarted to finish the update process. The easiest way for you to start, stop, and restart a virtual device is to use the Virtual Machines Management Console.

When you look at the Management Console, the virtual device status is **Running** because it is started by default after it is created. You can start, stop, and restart a virtual machine at any time.

#### To stop and start a virtual device
To stop a virtual device, click its name, and then click **Shutdown**. While the virtual device is shutting down, its status is **Stopping**. After the virtual device is stopped, its status is **Stopped**.

Use the following cmdlets to stop and start a virtual device.

`Stop-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`


`Start-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`
    
#### To restart a virtual device

When a virtual device is running and you want to restart it, click its name, and then click **Restart**. While the virtual device is restarting, its status is **Restarting**. When the virtual device is ready for you to use, its status is **Running**.

Use the following cmdlet to restart a virtual device.

`Restart-AzureVM -ServiceName "MyStorSimpleservice1" -Name "MyStorSimpleDevice"`






### Reset to factory defaults
If you decide that you just want to start over with your virtual device, simply deactivate and delete it and then create a new one. Just like when your physical device is reset, your new virtual device will not have any updates installed; therefore, make sure to check for updates before using it.

## Failover to the virtual device
Disaster recovery (DR) is one of the key scenarios that the StorSimple virtual device was designed for. In this scenario, the physical StorSimple device or entire datacenter might not be available. Fortunately, you can use a virtual device to restore operations in an alternate location. During DR, the volume containers from the source device change ownership and are transferred to the virtual device. The prerequisites for DR are that the virtual device has been created and configured, all the volumes within the volume container have been taken offline, and the volume container has an associated cloud snapshot.

> [!NOTE]
> 
> * When using a virtual device as the secondary device for DR, keep in mind that the 8010 has 30 TB of Standard Storage and 8020 has 64 TB of Premium Storage. The higher capacity 8020 virtual device may be more suited for a DR scenario.
> * You cannot failover or clone from a device running Update 2 to a device running pre-Update 1 software. You can however fail over a device running Update 2 to a device running Update 1 (1.1 or 1.2)
> 
> 
For a step-by-step procedure, go to [failover to a virtual device](storsimple-device-failover-disaster-recovery.md#fail-over-to-a-storsimple-virtual-device).

## Shut down or delete the virtual device
If you previously configured and used a StorSimple virtual device but now want to stop accruing compute charges for its use, you can shut down the virtual device. Shutting down the virtual device doesn’t delete its operating system or data disks in storage. It does stop charges accruing on your subscription, but storage charges for the OS and data disks will continue.

If you delete or shut down the virtual device, it will appear as **Offline** on the Devices page of the StorSimple Manager service. You can choose to deactivate or delete the device if you also wish to delete the backups created by the virtual device. For more information, see [Deactivate and delete a StorSimple device](storsimple-deactivate-and-delete-device.md).

#### To shut down a virtual device

1. Sign in to the Management Portal.

2. Click **Virtual Machines**, and then select the virtual device.

3. Click **Shutdown**.


#### To delete a virtual device

1. Sign in to the Azure classic portal.

- Click **Virtual Machines**, and then select the virtual device.

- Click **Delete** and choose to delete all the virtual machine disks.

## Next steps
* Learn how to [Use the StorSimple Manager service to manage a virtual device](storsimple-manager-service-administration.md).

* Understand how to [Restore a StorSimple volume from a backup set](storsimple-restore-from-backup-set.md). 


