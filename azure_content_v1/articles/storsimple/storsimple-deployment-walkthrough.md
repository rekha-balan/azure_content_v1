<properties
   pageTitle="Deploy an on-premises StorSimple device | Microsoft Azure"
   description="Describes the steps and best practices for deploying the StorSimple device and service. (Applies to Microsoft Azure StorSimple version .3 and earlier.)"
   services="storsimple"
   documentationCenter="NA"
   authors="alkohli"
   manager="adinah"
   editor="" />

<tags
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="01/14/2016"
   ms.author="alkohli" />

# Deploy your on-premises StorSimple device
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Update 2](../articles/storsimple/storsimple-deployment-walkthrough-u2.md)
> * [Update 1](../articles/storsimple/storsimple-deployment-walkthrough-u1.md)
> * [GA Release](../articles/storsimple/storsimple-deployment-walkthrough.md)
> 
> 
## Overview
Welcome to Microsoft Azure StorSimple device deployment. These deployment tutorials apply to StorSimple 8000 Series Release Version, StorSimple 8000 Series Update 0.1, StorSimple 8000 Series Update 0.2, and StorSimple 8000 Series Update 0.3. This series of tutorials describes how to configure your StorSimple device, and includes a configuration checklist, configuration prerequisites, and detailed configuration steps.

The information in these tutorials assumes that you have reviewed the safety precautions, and unpacked, racked, and cabled your StorSimple device. If you still need to perform those tasks, start with reviewing the [safety precautions](storsimple-safety.md). Depending on your device model, you can then unpack, rack mount, and cable by following the instructions in:

* [Unpack, rack mount, and cable your 8100](storsimple-8100-hardware-installation.md)
* [Unpack, rack mount, and cable your 8600](storsimple-8600-hardware-installation.md)

You will need administrator privileges to complete the setup and configuration process. We recommend that you review the configuration checklist before you begin. The deployment and configuration process can take some time to complete.

> [!NOTE]
> The StorSimple deployment information published on the Microsoft Azure website applies to StorSimple 8000 series devices only. For complete information about the 7000 series devices, go to: [http://onlinehelp.storsimple.com/](http://onlinehelp.storsimple.com). For 7000 series deployment information, see the [StorSimple System Quick Start Guide](http://onlinehelp.storsimple.com/111_Appliance/).
> 
> 
## Deployment steps
Perform these required steps to configure your StorSimple device and connect it to your StorSimple Manager service. In addition to the required steps, there are optional steps and procedures you may need during the deployment. The step-by-step deployment instructions indicate when you should perform each of these optional steps.

| Step | Description |
| --- | --- |
| **PREREQUISITES** |These need to be completed in preparation for the upcoming deployment. |
| Deployment configuration checklist. |Use this checklist to gather and record information prior to and during the deployment. |
| Deployment prerequisites. |These  validate the environment is ready for deployment. |
|  | |
| **STEP-BY-STEP DEPLOYMENT** |These steps are required to deploy your StorSimple device in production. |
| Step 1: Create a new service. |Set up cloud management and storage for your   StorSimple device. Skip this step if you have an existing service for other StorSimple devices. |
| Step 2: Get the service registration key. |Use this key to register & connect your StorSimple device with the management service. |
| Step 3: Configure and register the device through Windows PowerShell for StorSimple. |Connect the device to your network and register it with Azure to complete   the setup using the management service. |
| Step 4: Complete minimum device setup</br>Optional: Update your StorSimple device. |Use the management service to complete the device setup and enable it to provide storage. |
| Step 5: Create a volume container. |Create a container to provision volumes. A volume container has storage   account, bandwidth, and encryption settings for all the volumes contained in it. |
| Step 6: Create a volume. |Provision storage volume(s) on the StorSimple device for your servers. |
| Step 7: Mount, initialize, and format a volume.</br>Optional: Configure MPIO. |Connect your servers to the iSCSI storage provided by the device. Optionally configure MPIO to ensure that your servers can tolerate link, network, and interface failure. |
| Step 8: Take a backup. |Set up your backup policy to protect your data |
|  | |
| **OTHER PROCEDURES** |You may need to refer to these procedures as you deploy your solution. |
| Configure a new storage account for the service. | |
| Use PuTTY to connect to the device serial console. | |
| Get the IQN of a Windows Server host. | |
| Create a manual backup. | |

## Deployment configuration checklist
The following deployment configuration checklist describes the information that you need to collect before and as you configure the software on your StorSimple device. Preparing some of this information ahead of time will help streamline the process of deploying the StorSimple device in your environment. Use this checklist to also note down the configuration details as you deploy your device.

| Stage | Parameter | Details | Values |
| --- | --- | --- | --- |
| **Cable your device** |Serial access |Initial device configuration |Yes/No |
|  | | | |
| **Configure and register device** |Data 0 network settings |Data 0 IP Address:</br>Subnet mask:</br>Gateway:</br>Primary DNS server:</br>Primary NTP server:</br>Web proxy server IP/FQDN (optional):</br>Web proxy port: | |
|  |Device administrator password |Password must be between 8 and 15 characters containing lowercase, uppercase, numeric and special characters. | |
|  |StorSimple Snapshot Manager password |Password must be 14 or 15 characters containing lowercase, uppercase, numeric and special characters. | |
|  |Service Registration Key |This key is generated from the Azure classic portal. | |
|  |Service Data Encryption Key |This key is created when the device is registered with the management service via the Windows PowerShell for StorSimple. Copy this key and save it in a safe location. | |
|  | | | |
| **Complete minimum device setup** |Friendly name for your device |This is a descriptive name for the device. | |
|  |Timezone |Your device will use this time zone for all scheduled operations. | |
|  |Secondary DNS server |This is a required configuration. | |
|  |Network interface: Data 0 controller fixed IPs |These IP’s should be routable to the Internet.</br>Controller 0 fixed IP address:</br>Controller 1 fixed IP address: | |
|  | | | |
| **Additional network interface settings** |Network interface: Data 1</br>If iSCSI enabled, do not configure the Gateway. |Purpose: Cloud/iSCSI/Not used</br>IP address:</br>Subnet mask:</br>Gateway: | |
|  |Network interface: Data 2</br>If iSCSI enabled, do not configure the Gateway. |Purpose: Cloud/iSCSI/Not used</br>IP address:</br>Subnet mask:</br>Gateway: | |
|  |Network interface: Data 3</br>If iSCSI enabled, do not configure the Gateway. |Purpose: Cloud/iSCSI/Not used</br>IP address:</br>Subnet mask:</br>Gateway: | |
|  |Network interface: Data 4</br>If iSCSI enabled, do not configure the Gateway. |Purpose: Cloud/iSCSI/Not used</br>IP address:</br>Subnet mask:</br>Gateway: | |
|  |Network interface: Data 5</br>If iSCSI enabled, do not configure the Gateway. |Purpose: Cloud/iSCSI/Not used</br>IP address:</br>Subnet mask:</br>Gateway: | |
|  | | | |
| **Create a volume container** |Volume container name: |Name for the container | |
|  |Azure storage account: |Storage account name & access key to associate with this volume container | |
|  |Cloud storage encryption key: |Encryption key for storage in each container | |
|  | | | |
| **Create a volume** |Details for each volume |Volume name: | |
|  | |Size: | |
|  | |Usage type: | |
|  | |ACR name: | |
|  | |Default backup policy: | |
|  | | | |
| **Mount, initialize, and format a volume** |Details for each host server connecting to the storage |Windows Server name: | |
|  | |Windows Server IQN: | |
|  | |Windows Server volume name: | |
|  | |NTFS mount point/Drive letter: | |

## Deployment prerequisites
The following sections explain the configuration prerequisites for your StorSimple Manager service, your StorSimple device, and the network in your datacenter.

### For the StorSimple Manager service
Before you begin, make sure that:

* You have your Microsoft account with access credentials.

* You have your Microsoft Azure storage account with access credentials.

* Your Microsoft Azure subscription is enabled for the StorSimple Manager service. Your subscription should be purchased through the [Enterprise Agreement](https://azure.microsoft.com/pricing/enterprise-agreement/).

* You have access to terminal emulation software such as PuTTY.


### For the device in the datacenter
Before configuring the device, make sure that:

* Your device is fully unpacked, mounted on a rack and fully cabled for power, network, and serial access as described in:

  * [Unpack, rack mount, and cable your 8100 device](storsimple-8100-hardware-installation.md)
* [Unpack, rack mount, and cable your 8600 device](storsimple-8600-hardware-installation.md)


### For the network in the datacenter
Before you begin, make sure that:

* The ports in your datacenter firewall are opened to allow for iSCSI and cloud traffic as described in [Networking requirements for your StorSimple device](storsimple-system-requirements.md#networking-requirements-for-your-storsimple-device).
* The device in your datacenter can connect to outside network. Run the following [Windows PowerShell 4.0](http://www.microsoft.com/download/details.aspx?id=40855) cmdlets (tabulated below) to validate the connectivity to the outside network. Perform this validation on a computer (in datacenter network) that has connectivity to Azure and where you will deploy your StorSimple device.  

| For this parameter… | To check the validity… | Run these commands/cmdlets. |
| --- | --- | --- |
| **IP**</br>**Subnet**</br>**Gateway** |Is this a valid IPv4 or IPv6 address?</br>Is this a valid subnet?</br>Is this a valid gateway?</br>Is this a duplicate IP on network? |`ping ip`</br>`arp -a`</br>The `ping` and `arp` commands should fail indicating that there is no device in the datacenter network that is using this IP. |
|  | | |
| **DNS** |Is this a valid DNS and can resolve Azure URLs? |`Resolve-DnsName -Name www.bing.com -Server <DNS server IP address>` </br>An alternative command that can be used is:</br>`nslookup --dns-ip=<DNS server IP address> www.bing.com` |
|  |Check if port 53 is open. This is applicable only if you are using an external DNS for your device. Internal DNS should automatically resolve the external URLs. |`Test-Port -comp dc1 -port 53 -udp -UDPtimeout 10000`  </br>[More information on this cmdlet](http://learn-powershell.net/2011/02/21/querying-udp-ports-with-powershell/) |
|  | | |
| **NTP** |We trigger a time sync as soon as NTP server is input. Check UDP port 123 is open when you input `time.windows.com` or public time servers). |[Download and use this script](https://gallery.technet.microsoft.com/scriptcenter/Get-Network-NTP-Time-with-07b216ca). |
|  | | |
| **Proxy (optional)** |Is this a valid proxy URI and port? </br> Is the authentication mode correct? |<code>wget http://bing.com &#124; % {$_.StatusCode}</code></br>This command should be run immediately after configuring web proxy. If a status code of 200 is returned, it indicates that the connection is successful. |
|  |Is traffic routable through proxy? |Run the DNS validation, NTP check or HTTP check once after configuring proxy on your device. This will give a clear picture if traffic is getting blocked at proxy or elsewhere. |
|  | | |
| **Registration** |Check if outbound TCP ports 443, 80, 9354 are open. |`Test-NetConnection -Port   443 -InformationLevel Detailed`</br>[More information for Test-NetConnection cmdlet](https://technet.microsoft.com/library/dn372891.aspx) |

## Step-by-step deployment
Use the following step-by-step instructions to deploy your StorSimple device in the datacenter.

## Step 1: Create a new service
A StorSimple Manager service can manage multiple StorSimple devices. For the deployment of your first StorSimple device, you will need to create a new StorSimple Manager service.

> [!IMPORTANT]
> Skip this step if you have an existing StorSimple Manager service and you intend to deploy your StorSimple device with that service.
> 
> 
Perform the following steps to create a new instance of the StorSimple Manager service.

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

> [!IMPORTANT]
> If you did not enable the automatic creation of a storage account with your service, you will need to create at least one storage account after you have successfully created a service. This storage account will be used when you create a volume container.
> 
> If you did not create a storage account automatically, go to [Configure a new storage account for the service](#configure-a-new-storage-account-for-the-service.md) for detailed instructions.
> If you enabled the automatic creation of a storage account, go to [Step 2: Get the service registration key](#step-2:-get-the-service-registration-key.md).
> 
> 
## Step 2: Get the service registration key
After the StorSimple Manager service is up and running, you will need to get the service registration key. This key is used to register and connect your StorSimple device with the service.

Perform the following steps in the Azure classic portal.

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

## Step 3: Configure and register the device through Windows PowerShell for StorSimple
> [!IMPORTANT]
> Prior to performing this configuration, unplug all the network interfaces other than DATA 0 on both (active and passive) the controllers.
> 
> 
Use Windows PowerShell for StorSimple to complete the initial setup of your StorSimple device as explained in the following procedure. You will need to use terminal emulation software to complete this step. For more information, see [Use PuTTY to connect to the device serial console](#use-putty-to-connect-to-the-device-serial-console.md).

<!--author=alkohli last changed: 12/01/15-->


#### To configure and register the device

1. Access the Windows PowerShell interface on your StorSimple device serial console. See [Use PuTTY to connect to the device serial console](#use-putty-to-connect-to-the-device-serial-console) for instructions. **Be sure to follow the procedure exactly or you will not be able to access the console.**

2. In the session that opens up, press Enter one time to get a command prompt. 

3. You will be prompted to choose the language that you would like to set for your device. Specify the language, and then press Enter. 

    ![StorSimple configure and register device 1](./media/storsimple-configure-and-register-device/HCS_RegisterYourDevice1-include.png)

4. In the serial console menu that is presented, choose option 1 to log on with full access. 

    ![StorSimple register device 2](./media/storsimple-configure-and-register-device/HCS_RegisterYourDevice2-include.png)
  
     Complete steps 5-12 to configure the minimum required network settings for your device. **These configuration steps need to be performed on the active controller of the device.** The serial console menu indicates the controller state in the banner message. If you are not connected to the active controller, disconnect and then connect to the active controller.

5. At the command prompt, type your password. The default device password is **Password1**.

6. Type the following command:

     `Invoke-HcsSetupWizard` 

7. A setup wizard will appear to help you configure the network settings for the device. Supply the the following information: 
   - IP address for the DATA 0 network interface
   - Subnet mask
   - Gateway
   - IP address for Primary DNS server
   - IP address for Primary NTP server
   
      > [AZURE.NOTE] You may have to wait for a few minutes for the subnet mask and the DNS settings to be applied. If you get a "The device is not ready." error message, check the physical network connection on the DATA 0 network interface of your active controller.

8. (Optional) configure your web proxy server. Although web proxy configuration is optional, **be aware that if you use a web proxy, you can only configure it here**. For more information, go to [Configure web proxy for your device](storsimple-configure-web-proxy.md). If you run into any issues during this step, refer to troubleshooting guidance for [Errors during web proxy configuration](storsimple-troubleshoot-deployment.md#errors-during-the-optional-web-proxy-settings).
 

      > [AZURE.NOTE] You can press Ctrl + C at any time to exit the setup wizard. Any settings that you applied before you issued this command will be retained.

9. For security reasons, the device administrator password expires after the first session, and you will need to change it for subsequent sessions. When prompted, provide a device administrator password. A valid device administrator password must be between 8 and 15 characters. The password must contain a combination of lowercase characters, uppercase characters, numbers, and special characters.

10. The StorSimple Snapshot Manager password is also set here. You use this password when you authenticate a device with your Windows host running StorSimple Snapshot Manager. When prompted, provide a 14 to 15 character password. The password must contain a combination of three of the following: lowercase, uppercase, numeric, and special characters. 

    ![StorSimple register device 4](./media/storsimple-configure-and-register-device/HCS_RegisterYourDevice4-include.png)

    You can reset the StorSimple Snapshot Manager password from the StorSimple Manager service interface. For detailed steps, go to [Change the StorSimple passwords using the StorSimple Manager serivce](storsimple-change-passwords.md).

	To troubleshoot any issues during this step, refer to troubleshooting guidance for [Errors related to passwords](storsimple-troubleshoot-deployment.md#errors-related-to-device-administrator-and-storsimple-snapshot-manager-passwords).

11. The final step in the setup wizard registers your device with the StorSimple Manager service. For this, you will need the service registration key that you obtained in step 2. After you supply the registration key, you may need to wait for 2-3 minutes before the device is registered.

	To troubleshoot any possible device registration failures, refer to [Errors during device registration](storsimple-troubleshoot-deployment.md#errors-during-device-registration). For detailed troubleshooting, you can also refer to [Step-by-step troubleshooting example](storsimple-troubleshoot-deployment.md#step-by-step-storsimple-troubleshooting-example).

12. After the device is registered, a Service Data Encryption key will appear. Copy this key and save it in a safe location.
	
	> [AZURE.WARNING] This key will be required with the service registration key to register additional devices with the StorSimple Manager service. Refer to [StorSimple security](../articles/storsimple/storsimple-security.md) for more information about this key.

     ![StorSimple register device 6](./media/storsimple-configure-and-register-device/HCS_RegisterYourDevice6-include.png)

     To copy the text from the serial console window, simply select the text. You should then be able to paste it in the clipboard or any text editor. DO NOT use Ctrl + C to copy the service data encryption key. Using Ctrl + C will cause you to exit the setup wizard. As a result, the device administrator password and the StorSimple Snapshot Manager password will not be changed and the device will revert to the default passwords.

13. Exit the serial console.

14. Return to the Azure classic portal, and complete the following steps:
  1. Double-click your StorSimple Manager service to access the **Quick Start** page.
  2. Click **View connected devices**.
  3. On the **Devices** page, verify that the device has successfully connected to the service by looking up the status. The device status should be **Online**. If the device status is **Offline**, wait for a couple of minutes for the device to come online.
   
    ![StorSimple Devices page](./media/storsimple-configure-and-register-device/HCS_DevicesPageM-include.png) 
  
      > [AZURE.IMPORTANT] After the device is online, plug in the network cables that you had unplugged in the beginning of this step.

After the device is successfully registered and doesn't come online, you can run the `Test-HcsmConnection -Verbose` to ensure that the network connectivity is healthy. For the detailed usage of this cmdlet, go to [cmdlet reference for Test-HcsmConnection](https://technet.microsoft.com/library/dn715782.aspx).

![Video available](./media/storsimple-configure-and-register-device/Video_icon.png) **Video available**

To watch a video that demonstrates how to configure and register your device through Windows PowerShell for StorSimple, click [here](https://azure.microsoft.com/documentation/videos/initialize-the-storsimple-appliance/).

## Step 4: Complete minimum device setup
For the minimum device configuration of your StorSimple device, you are required to:

* Set up the secondary DNS server.
* Enable iSCSI on at least one network interface.
* Assign fixed IP addresses to both the controllers.

Perform the following steps in the Azure classic portal to complete the minimum device setup.

<!--author=alkohli last changed: 9/17/15-->

#### To complete the minimum StorSimple device setup

1. In the **Devices** page, select the device, click the arrow against the device name to go to the specific device page. 

	![Devices page with device online](./media/storsimple-complete-minimum-device-setup/HCS_DevicesPageM-include.png) 

2. Click quick start icon ![Quick Start Icon](./media/storsimple-complete-minimum-device-setup/HCS_QuickStartIcon-include.png) to access the device quick start page. Click **Complete device setup** to start the **Configure device** wizard.

	![Device quick start page](./media/storsimple-complete-minimum-device-setup/Device_Quick_Start_page_1M.png)

2. On the **Basic Settings** page, do the following:
  1. Supply a **friendly name** for your device. The default device name reflects information such as the device model and serial number. You can assign a friendly name of up to 64 characters to manage your device.
  2. Set the **time zone** based on the geographic location in which the device is being deployed. Your device will use this time zone for all scheduled operations.
  3. Under **DNS Settings**, provide an address for your **Secondary DNS Server**. If you are using IPv6, the field will be populated based on the IPv6 prefix provided in the Windows PowerShell interface. 
  If the secondary DNS server is not configured, you will not be allowed to save your device configuration.
  4. Under iSCSI enabled interfaces, enable at least one network for iSCSI. At least one network interface needs to be cloud-enabled and one interface needs to be iSCSI-enabled. DATA 0 is automatically cloud-enabled.
 
      ![StorSimple minimum device setup basic settings](./media/storsimple-complete-minimum-device-setup/HCS_MinDeviceSetupBasicSettings1-include.png)

3. Click the arrow icon. ![StorSimple arrow icon](./media/storsimple-complete-minimum-device-setup/HCS_ArrowIcon-include.png)

4. On the **Network Interfaces** page, provide the fixed IP addresses for Controller 0 and Controller 1. If the DATA 0 interface was configured for IPv4, the fixed IP addresses need to be provided in the IPv4 format. If you provided a prefix for IPv6 configuration, the fixed IP addresses will be populated automatically in these fields.


	> [AZURE.NOTE] 
 	> 
 	> - The controller fixed IP addresses need to be free IPs within the subnet accessible by the device IP address.
 	> - The fixed IP addresses for the controller are used for servicing the updates to the device, and therefore the fixed IPs must be routable and able to connect to the Internet.

    ![StorSimple minimum device setup network interfaces](./media/storsimple-complete-minimum-device-setup/HCS_MinDeviceSetupNetworkInterfaces2-include.png)

5. Click the check icon ![StorSimple check icon](./media/storsimple-complete-minimum-device-setup/HCS_CheckIcon-include.png).
  You will return to the device **Quick Start** page.

 > [AZURE.NOTE] You can modify all the other device settings at any time by accessing the **Configure** page.

![Video available](./media/storsimple-complete-minimum-device-setup/Video_icon.png) **Video available**

To watch a video that demonstrates how to complete the minimum device setup, click [here](https://azure.microsoft.com/documentation/videos/minimum-storsimple-device-setup/).

After the device configuration is complete, you must scan for updates and if available, install updates. The updates may take several hours to complete. Follow the instructions in [Scan for and apply updates](#scan-for-and-apply-updates.md).

## Step 5: Create a volume container
A volume container has storage account, bandwidth, and encryption settings for all the volumes contained in it. You will need to create a volume container before you can start provisioning volumes on your StorSimple device.

Perform the following steps in the Azure classic portal to create a volume container.

<!--author=SharS last changed: 9/17/15-->

#### To create a volume container

1. In the device **Quick Start** page, click **Add a volume container**. The **Create Volume Container** dialog box appears.

    ![Create Volume Container](./media/storsimple-create-volume-container/HCS_CreateVolumeContainerM-include.png)

2. In the **Create Volume container** dialog box:
  1. Supply a **Name** for your volume container. The name must be 3 to 32 characters long.
  2. Select a **Storage Account** to associate with this volume container. You can choose the default account that is generated at the time of service creation. You can also use the **Add new** option to specify a storage account that is not linked to this service subscription.
  3. Select **Enable Cloud Storage Encryption** to enable encryption of the data sent from the device to the cloud.
  4. Provide and confirm a **Cloud Storage Encryption Key** that is 8 to 32 characters long. This key is used by the device to access encrypted data.
  5. Select **Unlimited** in the **Specify bandwidth** drop-down list if you wish to consume all the available bandwidth. You can also set this option to **Custom** to employ bandwidth controls, and specify a value between 1 and 1,000 Mbps. 
  If you have your bandwidth usage information available, you may be able to allocate bandwidth based on a schedule by specifying **Select a bandwidth template**. For a step-by-step procedure, go to [Add a bandwidth template](storsimple-manage-bandwidth-templates.md#add-a-bandwidth-template).
  6. Click the check icon ![check-icon](./media/storsimple-create-volume-container/HCS_CheckIcon-include.png) to save this volume container and exit the wizard. 

  The newly created volume container will be listed on the **Volume containers** page.

![Video available](./media/storsimple-create-volume-container/Video_icon.png) **Video available**

To watch a video that demonstrates how to create a volume container in your StorSimple solution, click [here](https://azure.microsoft.com/documentation/videos/create-a-volume-container-in-your-storsimple-solution/).

## Step 6: Create a volume
After you create a volume container, you can provision a storage volume on the StorSimple device for your servers. Perform the following steps in the Azure classic portal to create a volume.

> [!IMPORTANT]
> StorSimple Manager can create only thinly provisioned volumes.  You cannot create fully or partially provisioned volumes.
> 
> 
<!--author=SharS last changed: 11/16/15-->

#### To create a volume

1. On the device **Quick Start** page, click **Add a volume**. This starts the Add a volume wizard.

2. In the Add a volume wizard, under **Basic Settings**, do the following:
   1. Supply a **Name** for your volume.
   2. Specify the **Provisioned Capacity** for your volume in GB or TB. The volume capacity must be between 1 GB and 64 TB for a physical device.
   3. On the drop-down list, select the **Usage Type** for your volume. 
   4. If you are using this volume for archival data, select the **Use this volume for less frequently accessed archival data** check box. For all other use cases, simply select **Tiered Volume**. (Tiered volumes were formerly called primary volumes).
   4. Click the arrow icon ![arrow-icon](./media/storsimple-create-volume/HCS_ArrowIcon-include.png) to go to the next page.

        ![Add volume](./media/storsimple-create-volume/AddVolume1-include.png)

3. In the **Additional Settings** dialog box, add a new access control record (ACR):
   1. Supply a **Name** for your ACR.
   2. Under **iSCSI Initiator Name**, provide the iSCSI Qualified Name (IQN) of your Windows host. If you don't have the IQN, go to [Get the IQN of a Windows Server host](#get-the-iqn-of-a-windows-server-host).
   3. We recommend that you enable a default backup by selecting the **Enable a default backup for this volume** check box. The default backup will create a policy that executes at 22:30 each day (device time) and creates a cloud snapshot of this volume.

        > [AZURE.NOTE] After the backup is enabled here, it cannot be reverted. You will need to edit the volume to modify this setting.

        ![Add volume](./media/storsimple-create-volume/AddVolume2-include.png)

4. Click the check icon ![check icon](./media/storsimple-create-volume/HCS_CheckIcon-include.png). A volume will be created with the specified settings.

![Video available](./media/storsimple-create-volume/Video_icon.png) **Video available**

To watch a video that demonstrates how to create a StorSimple volume, click [here](https://azure.microsoft.com/documentation/videos/create-a-storsimple-volume/).



## Step 7: Mount, initialize, and format a volume
> [!IMPORTANT]
> 
> * For the high availability of your StorSimple solution, we recommend that you configure MPIO on your Windows Server host (optional) prior to configuring iSCSI on your Windows Server host. MPIO configuration on host servers will ensure that the servers can tolerate a link, network, or interface failure.
> 
> * For MPIO and iSCSI installation and configuration instructions, go to [Configure MPIO for your StorSimple device](storsimple-configure-mpio-windows-server.md). These will also include the steps to mount, initialize and format StorSimple volumes.
> 
> 
> 
If you decide not to configure MPIO, perform the following steps to mount, initialize, and format your StorSimple volumes.

<!--author=SharS last changed: 9/17/15-->

#### To mount, initialize, and format a volume

1. Start the Microsoft iSCSI initiator.

2. In the **iSCSI Initiator Properties** window, on the **Discovery** tab, click **Discover Portal**.

3. In the **Discover Target Portal** dialog box, supply the IP address of your iSCSI-enabled network interface, and then click **OK**. 

4. In the **iSCSI Initiator Properties** window, on the **Targets** tab, locate the **Discovered targets**. The device status should appear as **Inactive**.

5. Select the target device and then click **Connect**. After the device is connected, the status should change to **Connected**. (For more information about using the Microsoft iSCSI initiator, see [Installing and Configuring Microsoft iSCSI Initiator][1]).

6. On your Windows host, press the Windows Logo key + X, and then click **Run**. 

7. In the **Run** dialog box, type **Diskmgmt.msc**. Click **OK**, and the **Disk Management** dialog box will appear. The right pane will show the volumes on your host.

8. In the **Disk Management** window, the mounted volumes will appear as shown in the following illustration. Right-click the discovered volume (click the disk name), and then click **Online**.

     ![Initialize format volume](./media/storsimple-mount-initialize-format-volume/HCS_InitializeFormatVolume-include.png) 

9. Right-click the volume (click the disk name) again, and then click **Initialize**.

10. To format a simple volume, perform the following steps:
  1. Select the volume, right-click it (click the right area), and click **New Simple Volume**.
  2. In the New Simple Volume wizard, specify the volume size and drive letter and configure the volume as an NTFS file system.
  3. Specify a 64 KB allocation unit size. This allocation unit size works well with the deduplication algorithms used in the StorSimple solution.
  4. Perform a quick format.

![Video available](./media/storsimple-mount-initialize-format-volume/Video_icon.png) **Video available**

To watch a video that demonstrates how to mount, initialize, and format a StorSimple volume, click [here](https://azure.microsoft.com/documentation/videos/mount-initialize-and-format-a-storsimple-volume/).

<!--Link references-->
[1]: https://technet.microsoft.com/library/ee338480(WS.10).aspx


## Step 8: Take a backup
Backups provide point-in-time protection of volumes and improve recoverability while minimizing restore times. You can take two types of backup on your StorSimple device: local snapshots and cloud snapshots. Each of these backup types can be **Scheduled** or **Manual**.

Perform the following steps in the Azure classic portal to create a scheduled backup.

<!--author=alkohli last changed: 9/17/15-->

### To take a backup

1. On the device **Quick Start** page, click **Add a backup policy**. This will start the Add Backup Policy wizard. 

2. On the **Define your backup policy** page:
  1. Supply a name that contains between 3 and 150 characters for your backup policy.
  2. Select the volumes to be backed up. If you select more than one volume, these volumes will be grouped together to create a crash-consistent backup.
  3. Click the arrow icon ![arrow-icon](./media/storsimple-take-backup/HCS_ArrowIcon-include.png). 
  
    ![Add-backup-policy](./media/storsimple-take-backup/HCS_AddBackupPolicyWizard1M-include.png)

3. On the **Define a schedule** page:
  1. Select the type of backup from the drop-down list. For faster restores, select **Local Snapshot**. For data resiliency, select **Cloud Snapshot**.
  2. Specify the backup frequency in minutes, hours, days, or weeks.
  3. Select a retention time. The retention choices depend on the backup frequency. For example, for a daily policy, the retention can be specified in weeks, whereas retention for a monthly policy is in months.
  4. Select the starting time and date for the backup policy.
  5. Select the **Enable** check box to enable the backup policy. 
  6. Click the check icon ![check-icon](./media/storsimple-take-backup/HCS_CheckIcon-include.png) to save the policy.

    ![Add-backup-policy](./media/storsimple-take-backup/HCS_AddBackupPolicyWizard2M-include.png)
 
     You now have a backup policy that will create scheduled backups of your volume data.

You have completed the device configuration. 

![Video available](./media/storsimple-take-backup/Video_icon.png) **Video available**

To watch a video that demonstrates how to take a StorSimple backup, click [here](https://azure.microsoft.com/documentation/videos/take-a-storsimple-backup/).

You can take a manual backup at any time. For procedures, go to [Create a manual backup](#Create-a-manual-backup.md).

## Configure a new storage account for the service
This is an optional step that you need to perform only if you did not enable the automatic creation of a storage account with your service. A Microsoft Azure storage account is required to create a StorSimple volume container.

If you need to create an Azure storage account in a different region, see [About Azure Storage Accounts](../storage/storage-create-storage-account.md) for step-by-step instructions.

Perform the following steps in the Azure classic portal, on the **StorSimple Manager service** page.

<!--author=SharS last changed: 9/17/15-->

#### To add a new storage account in StorSimple Release version (GA)

1. On the StorSimple Manager service landing page, select your service and double-click it. This will take you to the **Quick Start** page. Select the **Configure** page.

2. Click **Add/edit storage account**.

3. In the **Add/Edit Storage Account** dialog box, do the following:

  1. Click **Add new**.
  2. Provide a name for your storage account.
  3. Supply the primary **Access Key** for your Microsoft Azure storage account.
  4. Select **Enable SSL Mode** to create a secure channel for network communication between your device and the cloud. Clear the **Enable SSL Mode** check box only if you are operating within a private cloud.
  5. Click the check icon ![check icon](./media/storsimple-configure-new-storage-account/HCS_CheckIcon-include.png). You will be notified after the storage account is successfully created.

    ![Add storage account](./media/storsimple-configure-new-storage-account/HCS_AddStorageAccount-include.png)

4. The newly created storage account will be displayed on the **Configure** page under **Storage accounts**. Click **Save** to save the newly created storage account. Click **OK** when prompted for confirmation.


## Use PuTTY to connect to the device serial console
To connect to Windows PowerShell for StorSimple, you need to use terminal emulation software such as PuTTY. You can use PuTTY when you access the device directly through the serial console or by opening a telnet session from a remote computer.

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
 

## Scan for and apply updates
Updating your device can take anywhere from 1-4 hours. Perform the following steps to scan for and apply updates on your device.

> [!NOTE]
> If you have a gateway configured on a network interface other than Data 0, you will need to disable Data 2 and Data 3 network interfaces before installing the update. Go to **Devices > Configure** and disable Data 2 and Data 3 interfaces. You should re-enable these interfaces after the device is updated.
> 
> 
#### To update your device
1. On the device **Quick Start** page, click **Devices**. Select the physical device, click **Maintenance** and then click **Scan Updates**.  
2. A job to scan for available updates is created. If updates are available, the **Scan Updates** changes to **Install Updates**. Click **Install Updates**. You may be requested to disable Data 2 and Data 3 prior to installing the updates. You must disable these network interfaces or the updates may fail.
3. An update job will be created. Monitor the status of your update by navigating to **Jobs**.

   > [!NOTE]
> When the update job starts, it immediately displays the status as 50 percent. The status then changes to 100 percent only after the update job is complete. There is no real time status for the updates process.
> 
4. After the device is successfully updated, enable Data 2 and Data 3 network interfaces if these were disabled.


## Get the IQN of a Windows Server host
Perform the following steps to get the iSCSI Qualified Name (IQN) of a Windows host that is running Windows Server 2012.

<!--author=SharS last changed: 9/17/15-->

#### To get the IQN of a Windows host

1. Start the Microsoft iSCSI initiator on your Windows host.

2. In the **iSCSI Initiator Properties** window, on the **Configuration** tab, select and copy the string from the **Initiator Name** field.
 
    ![iSCSI initiator properties](./media/storsimple-get-iqn/HCS_iSCSIInitiatorPropertiesFigureIQN-include.png)

3. Save this string.


## Create a manual backup
Perform the following steps in the Azure classic portal to create an on-demand manual backup for a single volume on your StorSimple device.


<!--author=SharS last changed: 9/15/15-->


#### To create a manual backup

1. On the **Devices** page, go to the **Backup Policies** tab. This tab lists all the backup policies in a tabular format, including the policy for the volume that you want to back up.

2. Select the policy by clicking anywhere in the corresponding row except for the first column. At the bottom of the page, click **Take backup**. The button will expand to show the backup options: local snapshot and cloud snapshot. 

3. When you choose either of these options, you will be prompted for confirmation. Click **Yes**. 

    ![Create manual backup](./media/storsimple-create-manual-backup/HCS_CreateManualBackup1-include.png)
 
    This will start a job to create a snapshot. You will see a notification at the bottom of the page after the job is successfully created.

4. To monitor the job, click **View Job** in the notification area (at the bottom of the page). 

    ![Monitor the manual backup](./media/storsimple-create-manual-backup/HCS_CreateManualBackup2-include.png)

5. After the backup job is finished, go to the **Backup catalog** tab.

6. Set the filter selections to the appropriate device, backup policy, and time range. Click the check icon ![check icon](./media/storsimple-create-manual-backup/HCS_CheckIcon-include.png) after setting the filters.

  The backup should appear in the list of backup sets that is displayed in the catalog.


## Next steps
* Configure a [virtual device](storsimple-virtual-device.md).

* Use the [StorSimple Manager service](https://msdn.microsoft.com/library/azure/dn772396.aspx) to manage your StorSimple device.


