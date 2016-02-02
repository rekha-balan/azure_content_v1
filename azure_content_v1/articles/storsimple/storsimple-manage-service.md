<properties 
   pageTitle="Deploy the StorSimple Manager service | Microsoft Azure"
   description="Explains how to create and delete the StorSimple Manager service in the Azure classic portal, and describes how to manage the service registration key."
   services="storsimple"
   documentationCenter=""
   authors="SharS"
   manager="carolz"
   editor="" />

<tags 
   ms.service="storsimple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="12/01/2015"
   ms.author="v-sharos" />

# Deploy the StorSimple Manager service
## Overview
The StorSimple Manager service runs in Microsoft Azure and connects to multiple StorSimple devices. After you create the service, you can use it to manage the devices from the Microsoft Azure classic portal running in a browser. This allows you to monitor all the devices that are connected to the StorSimple Manager service from a single, central location, thereby minimizing administrative burden.

The StorSimple Manager landing page lists all the StorSimple Manager services that you can use to manage your StorSimple storage devices. For each StorSimple Manager service, the following information is presented on the StorSimple Manager page:

* **Name** – The name that was assigned to your StorSimple Manager service when it was created. The service name cannot be changed after the service is created.

* **Status** – The status of the service, which can be **Active**, **Creating**, or **Online**.

* **Location** – The geographical location in which the StorSimple device will be deployed.

* **Subscription** – The billing subscription that is associated with your service.


The common tasks that can be performed through the StorSimple Manager page are:

* Create a service
* Delete a service
* Get the service registration key
* Regenerate the service registration key

This tutorial describes how to perform each of these tasks.

## Create a service
Use the **Quick Create** option to create a StorSimple Manager service if you want to deploy your StorSimple device. To create a service, you need to have:

* A subscription with an Enterprise Agreement
* An active Microsoft Azure storage account
* The billing information that is used for access management

You can also choose to generate a default storage account when you create the service.

A single service can manage multiple devices. However, a device cannot span multiple services. A large enterprise can have multiple service instances to work with different subscriptions, organizations, or even deployment locations.

Perform the following steps to create a service.

<!--author=alkohli last changed:01/14/2016-->


#### To create a new service

1. Using your Microsoft account credentials, log on to the Azure classic portal at this URL: [https://manage.windowsazure.com/](https://manage.windowsazure.com/).

2. In the Azure classic portal , click **New** > **Data Services** > **StorSimple Manager** > **Quick Create**.

3. In the form that is displayed, do the following:
  1. Supply a unique **Name** for your service. This is a friendly name that can be used to identify the service. The name can have between 2 and 50 characters that can be letters, numbers, and hyphens. The name must start and end with a letter or a number.
  2. Supply a **Location** for your service. In general, choose a Location closest to the geographical region where you want to deploy your device. You may also want to factor in the following: 
	 
		- If you have existing workloads in Azure that you also intend to deploy with your StorSimple device, you should use that datacenter.
		- Your StorSimple Manager service and Azure storage can be in two separate locations. In such a case, you are required to create the StorSimple Manager and Azure storage account separately. To create an Azure storage account, go to the Azure Storage service in the Azure classic portal and follow the steps in [Create an Azure Storage account](storage-create-storage-account.md#create-a-storage-account). After you create this account, add it to the StorSimple Manager service by following the steps in [Configure a new storage account for the service](storsimple-deployment-walkthrough.md#configure-a-new-storage-account-for-the-service).
		 
  3. Choose a **Subscription** from the drop-down list. The subscription is linked to your billing account. This field is not present if you have only one subscription.
  4. Select **Create a new storage account** to automatically create a storage account with the service. This storage account will have a special name such as "storsimplebwv8c6dcnf." If you need your data in a different location, uncheck this box. 
  5. Click **Create StorSimple Manager** to create the service.

   ![Create StorSimple Manager](./media/storsimple-create-new-service/HCS_CreateAService-include.png)

  You will be directed to the **Service** landing page. The service creation will take a few minutes. After the service is successfully created, you will be notified appropriately and the status of the service will change to **Active**.
 
   ![Service creation](./media/storsimple-create-new-service/HCS_StorSimpleManagerServicePage-include.png)

![Video available](./media/storsimple-create-new-service/Video_icon.png) **Video available**

To watch a video that demonstrates how to create a new StorSimple Manager service, click [here](https://azure.microsoft.com/documentation/videos/create-a-storsimple-manager-service/).

## Delete a service
Before you delete a service, make sure that no connected devices are using it. If the service is in use, deactivate the connected devices. The deactivate operation will sever the connection between the device and the service, but preserve the device data in the cloud. 

[AZURE.IMPORTANT]AZURE.IMPORTANT] After a service is deleted, the operation cannot be reversed. Any device that was using the service will need to be factory reset before it can be used with another service. In this scenario, the local data on the device, as well as the configuration, will be lost.

Perform the following steps to delete a service.

### To delete a service
1. On the **StorSimple Manager service** page, select the service that you wish to delete.

2. Click **Delete** at the bottom of the page.

3. Click **Yes** in the confirmation notification. It may take a few minutes for the service to be deleted.


## Get the service registration key
After you have successfully created a service, you will need to register your StorSimple device with the service. To register your first StorSimple device, you will need the service registration key. To register additional devices with an existing StorSimple service, you will need both the registration key and the service data encryption key (which is generated on the first device during registration). For more information about the service data encryption key, see [StorSimple security](storsimple-security.md). You can get the registration key by accessing **Registration Key** on the **Services** page.

Perform the following steps to get the service registration key.

<!--author=alkohli last changed: 9/17/15-->

#### To get the StorSimple service registration key

1. On the **StorSimple Manager service** page, click the service that you created. This will take you to the **Quick Start** page. (You can click the quick start icon ![StorSimple Quick Start icon ](./media/storsimple-get-service-registration-key/HCS_QuickStartIcon-include.png) to access the **Quick Start** page at any time.)

     ![StorSimple Quick Start page](./media/storsimple-get-service-registration-key/HCS_ServiceQuickStart-include.png)

2. Click **Get service registration key**. You can also click **Registration Key** at the bottom of the page. You will have to wait for a few minutes while the key is retrieved. The **Service Registration Key** dialog box appears.

     ![Service Registration Key dialog box](./media/storsimple-get-service-registration-key/HCS_GetServiceRegistrationKey-include.png)

3. Locate the service registration key.

4. Click the copy icon ![StorSimple Copy icon](./media/storsimple-get-service-registration-key/HCS_CopyIcon-include.png) to copy the key and save it for later use.

5. Click the check icon ![StorSimple Check icon](./media/storsimple-get-service-registration-key/HCS_CheckIcon-include.png) to close this dialog box and return to the **Quick Start** page.

> [AZURE.NOTE] The service registration key is used to register all the devices that need to register with your StorSimple Manager service.

![Video available](./media/storsimple-get-service-registration-key/Video_icon.png) **Video available**

To watch a video that demonstrates how to get the service registration key, click [here](https://azure.microsoft.com/documentation/videos/get-the-service-registration-key/).

Keep the service registration key in a safe location. You will need this key, as well as the service data encryption key, to register additional devices with this service. After obtaining the service registration key, you will need to configure your device through the Windows PowerShell for StorSimple interface.

For details on how to use this registration key, see [Step 3: Configure and register the device through Windows PowerShell for StorSimple](storsimple-deployment-walkthrough.md#step-2-configure-and-register-the-device-through-windows-powershell-for-storsimple).

## Regenerate the service registration key
You will need to regenerate a service registration key if you are required to perform key rotation or if the list of service administrators has changed. When you regenerate the key, the new key is used only for registering subsequent devices. The devices that were already registered are unaffected by this process.

Perform the following steps to regenerate a service registration key.

### To regenerate the service registration key
1. On the **StorSimple Manager service** page, click **Registration Key**.

2. In the **Service Registration Key** dialog box, click **Regenerate**.

3. You will see a confirmation message. Click **OK** to continue with the regeneration.

4. A new service registration key will appear.

5. Copy this key and save it for registering any new devices with this service.

6. Click the check icon ![Check icon](./media/storsimple-manage-service/HCS_CheckIcon.png) to close this dialog box.


## Next steps
* Learn more about the [StorSimple deployment process](storsimple-deployment-walkthrough.md).

* Learn more about [managing your StorSimple storage account](storsimple-manage-storage-accounts.md).

* Learn more about how to [use the StorSimple Manager service to administer your StorSimple device](storsimple-manager-service-administration.md).


