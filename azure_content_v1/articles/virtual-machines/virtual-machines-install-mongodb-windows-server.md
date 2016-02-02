<properties
    pageTitle="Install MongoDB on a Windows VM | Microsoft Azure"
    description="Learn how to install MongoDB on an Azure VM created with the classic deployment model running Windows Server."
    services="virtual-machines"
    documentationCenter=""
    authors="dsk-2015"
    manager="timlt"
    editor="tysonn"
    tags="azure-service-management"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/14/2015"
    ms.author="dkshir"/>

# Install MongoDB on a Windows VM
> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the Resource Manager model.

[MongoDB](http://www.mongodb.org/) is a popular open-source, high-performance NoSQL database.  Using the [Azure classic portal](http://manage.windowsazure.com), you can create a virtual machine running Windows Server from the Image Gallery, using the classic deployment model. You can then install and configure a MongoDB database on the virtual machine.

## Create a virtual machine running Windows Server
Follow these instructions to create a virtual machine.

1. Sign in to the [classic portal](http://manage.windowsazure.com). Check out the [Free Trial](https://azure.microsoft.com/pricing/free-trial/) offer if you don't have a subscription yet.

2. On the command bar at the bottom of the window, click **New**.

3. Under **Compute**, click **Virtual Machine**, and then click **From Gallery**.

	![Navigate to From Gallery in the Command Bar](./media/virtual-machines-create-WindowsVM/fromgallery.png)

4. The first screen after this lets you **Choose an Image** for your virtual machine from the list of available images. You can choose an image from the gallery or select from images and disks that you have uploaded. The available images may differ depending on the subscription you're using.

5. The second screen lets you pick a computer name, size, and administrative user name and password. Use the tier and size required to run your app or workload. Here are some tips:

	- **Virtual Machine Name** can only contain letters, numbers and hypens. It also must start with a letter and end with a letter or a number.
	- **New User Name** refers to the administrative account that you use to manage the server. The password must be at 8-123 characters long and have at least 3 of the following: 1 lower case character, 1 upper case character, 1 number, and 1 special character. **You'll need the user name and password to log on to the virtual machine**.
	- A virtual machine's size affects the cost of using it, as well as configuration options such as how many data disks you can attach. For details, see [Sizes for virtual machines](../articles/virtual-machines-size-specs.md).

6. The third screen lets you configure resources for networking, storage, and availability. Here are some tips:

	- The **Cloud Service DNS Name** is the global DNS name that becomes part of the URI that's used to contact the virtual machine. You'll need to come up with your own cloud service name because it must be unique in Azure. Cloud services are important for scenarios using [multiple virtual machines](../articles/cloud-services-connect-virtual-machine.md).

	- For **Region/Affinity Group/Virtual Network**, use a region that's appropriate to your location. You can also choose to specify a virtual network instead.

	>[AZURE.NOTE] If you want a virtual machine to use a virtual network, you **must** specify the virtual network when you create the virtual machine. You can't join the virtual machine to a virtual network after you create the VM. For more information, see [Azure Virtual Network Overview](virtual-networks-overview.md).
	>
	> For details about configuring endpoints, see [How to Set Up Endpoints to a Virtual Machine](../articles/virtual-machines-set-up-endpoints.md).

7. The fourth configuration screen lets you install the VM Agent and configure some of the available extensions.

	>[AZURE.NOTE] The VM agent provides the environment for you to install extensions that can help you interact with or manage the virtual machine. For details, see [About the VM agent and extensions](virtual-machines-extensions-agent-about.md).  

8. After the virtual machine is created, the classic portal lists the new virtual machine under **Virtual Machines**. The corresponding cloud service and storage account also are created and are listed in those sections. Both the virtual machine and cloud service are started automatically and their status is listed as **Running**.

	![Configure VM Agent and the endpoints of the virtual machine](./media/virtual-machines-create-WindowsVM/vmcreated.png)


> [!NOTE]
> You can add an endpoint for MongoDB while creating the virtual machine, and configure it as follows: name it as **Mongo**, use **TCP** as the protocol, and set both the public and private ports to **27017**.
> 
> 
## Attach a data disk
To provide storage for the virtual machine, attach a data disk and then initialize it so that Windows can use it. You can either attach an existing disk if you already have data you want to use, or attach an empty disk.


For more details about disks, see [About Disks and VHDs for Virtual Machines](../articles/virtual-machines-disks-vhds.md).

##<a id="attachempty"></a>How to: Attach an empty disk

Attaching an empty disk is the simpler way to add a data disk, because Azure creates the .vhd file for you and stores it in the storage account.

1. Click **Virtual Machines**, and then select the appropriate virtual machine.

2. On the command bar, click **Attach**, and then click **Attach Empty Disk**.


	![Attach an empty disk](./media/howto-attach-disk-window-linux/AttachEmptyDisk.png)

3.	The **Attach an Empty Disk** dialog box appears.


	![Attach a new empty disk](./media/howto-attach-disk-window-linux/AttachEmptyDetail.png)


	Do the following:

	- In **File Name**, accept the default name or type another one for the .vhd file, which is used for the disk. The data disk uses an automatically generated name, even if you type another name for the .vhd file.

	- Type the **Size (GB)** of the data disk.

	- Click the check mark to finish.

4.	After the data disk is created and attached, it's listed in the dashboard of the virtual machine.

	![Empty data disk successfully attached](./media/howto-attach-disk-window-linux/AttachEmptySuccess.png)
	
> [AZURE.NOTE]
> After you add a new data disk, you'll need to log on to the virtual machine and initialize the disk so the virtual machine can use the disk for storage. 

##<a id="attachexisting"></a>How to: Attach an existing disk

Attaching an existing disk requires that you have a .vhd available in a storage account. Use the [Add-AzureVhd](http://go.microsoft.com/FWLink/p/?LinkID=391684) cmdlet to upload the .vhd file to the storage account. After you've created and uploaded the .vhd file, you can attach it to a virtual machine.

1. Click **Virtual Machines**, and then select the appropriate virtual machine.

2. On the command bar, click **Attach**, and then select **Attach Disk**.


	![Attach data disk](./media/howto-attach-disk-window-linux/AttachExistingDisk.png)

	The **Attach Disk** dialog box appears.



	![Enter data disk details](./media/howto-attach-disk-window-linux/AttachExistingDetail.png)

3. Select the data disk that you want to attach to the virtual machine.

4. Click the check mark to attach the data disk to the virtual machine.

5.	After the data disk is attached, it's listed in the dashboard of the virtual machine.


	![Data disk successfully attached](./media/howto-attach-disk-window-linux/AttachExistingSuccess.png)




For instructions on initializing the disk, see "How to: Initialize a new data disk in Windows Server" in [How to attach a data disk to a Windows virtual machine](storage-windows-attach-disk.md).

## Install and run MongoDB on the virtual machine
Follow these steps to install and run MongoDB on a virtual machine running Windows Server.

> [AZURE.IMPORTANT] MongoDB security features, such as authentication and IP address binding, are not enabled by default. Security features should be enabled before deploying MongoDB to a production environment.  See [Security and Authentication](http://www.mongodb.org/display/DOCS/Security+and+Authentication) for more information.

1. After you've connected to the virtual machine using Remote Desktop, open Internet Explorer from the **Start** menu on the virtual machine.

2. Select the **Tools** button in the upper right corner.  In **Internet Options**, select the **Security** tab, and then select the **Trusted Sites** icon, and finally click the **Sites** button. Add _http://\*.mongodb.org_ to the list of trusted sites.

3. Go to [Downloads- MongoDB] [MongoDownloads].

4. Find the **Current Stable Release**, select the latest **64-bit** version in the Windows column, download and run the MSI installer.

5. MongoDB is typically installed on C:\Program Files\MongoDB. Search for Environment Variables on the desktop and add the MongoDB binaries path to the PATH variable. For example, you might find the binaries at C:\Program Files\MongoDB\Server\3.0\bin on your machine.

6. Create MongoDB data and log directories in the data disk (drive **F:**, for example) you created in the steps above. From **Start**, select **Command Prompt** to open a command prompt window.  Type:

		C:\> F:
		F:\> mkdir \MongoData
		F:\> mkdir \MongoLogs

7. To run the database, run:

		F:\> C:
		C:\> mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log

	All log messages will be directed to the *F:\MongoLogs\mongolog.log* file as mongod.exe server starts and preallocates journal files. It may take several minutes for MongoDB to preallocate the journal files and start listening for connections.

8. To start the MongoDB administrative shell, open another command window from **Start** and type the following:

		C:\> cd \my_mongo_dir\bin  
		C:\my_mongo_dir\bin> mongo  
		>db  
		test
		> db.foo.insert( { a : 1 } )  
		> db.foo.find()  
		{ _id : ..., a : 1 }  
		> show dbs  
		...  
		> show collections  
		...  
		> help  

	The database is created by the insert.

9. Alternatively, you can install mongod.exe as a service:

		C:\mongodb\bin>mongod --logpath F:\MongoLogs\mongolog.log --logappend --dbpath F:\MongoData\ --install

	This creates a service named MongoDB with a description of "Mongo DB". The **--logpath** option must be used to specify a log file, since the running service will not have a command window to display output.  The **--logappend** option specifies that a restart of the service will cause output to append to the existing log file.  The **--dbpath** option specifies the location of the data directory. For more service-related command line options, see [Service-related command line options] [MongoWindowsSvcOptions].

	To start the service, run this command:

		C:\mongodb\bin>net start MongoDB

10. Now that MongoDB is installed and running, you'll need to open a port in Windows Firewall so you can remotely connect to MongoDB.  From the **Start** menu, select **Administrator Tools** and then **Windows Firewall with Advanced Security**.

11. In the left pane, select **Inbound Rules**.  In the **Actions** pane on the right, select **New Rule...**.

	![Windows Firewall][Image1]

	In the **New Inbound Rule Wizard**, select **Port** and then click **Next**.

	![Windows Firewall][Image2]

	Select **TCP** and then **Specific local ports**.  Specify a port of "27017" (the default port MongoDB listens on) and click **Next**.

	![Windows Firewall][Image3]

	Select **Allow the connection** and click **Next**.

	![Windows Firewall][Image4]

	Click **Next** again.

	![Windows Firewall][Image5]

	Specify a name for the rule, such as "MongoPort", and click **Finish**.

	![Windows Firewall][Image6]

12. If you didn't configure an endpoint for MongoDB when you created the virtual machine, you can do it now. You need both the firewall rule and the endpoint to be able to connect to MongoDB remotely. In the Management Portal, click **Virtual Machines**, click the name of your new virtual machine, and then click **Endpoints**.

	![Endpoints][Image7]

13. Click **Add** at the bottom of the page. Select **Add a Stand-Alone Endpoint** and click **Next**.

	![Endpoints][Image8]

14. Add an endpoint with name "Mongo", protocol **TCP**, and both **Public** and **Private** ports set to "27017". This will allow MongoDB to be accessed remotely.

	![Endpoints][Image9]

> [AZURE.NOTE] The port 27017 is the default port used by MongoDB. You can change this by the _--port_ subcommand when starting the mongod.exe server. Make sure to give the same port number in the firewall as well as the "Mongo" endpoint in the above instructions.


[MongoDownloads]: http://www.mongodb.org/downloads

[MongoWindowsSvcOptions]: http://www.mongodb.org/display/DOCS/Windows+Service


[Image1]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall1.png
[Image2]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall2.png
[Image3]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall3.png
[Image4]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall4.png
[Image5]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall5.png
[Image6]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall6.png
[Image7]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint.png
[Image8]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint2.png
[Image9]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint3.png


## Summary
In this tutorial, you learned how to create a virtual machine running Windows Server, remotely connect to it, and attach a data disk.  You also learned how to install and configure MongoDB on the Windows-based virtual machine. You can now access MongoDB on the Windows-based virtual machine, by following the advanced topics in the [MongoDB documentation](http://docs.mongodb.org/manual/).

[MongoDocs]: http://docs.mongodb.org/manual/
[MongoDB]: http://www.mongodb.org/
[AzurePortal]: http://manage.windowsazure.com
