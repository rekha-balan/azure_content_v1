<properties
    pageTitle="Create a VM running Windows in the classic portal | Microsoft Azure"
    description="Create a Windows virtual machine in the Azure classic portal."
    services="virtual-machines"
    documentationCenter=""
    authors="cynthn"
    manager="timlt"
    editor=""
    tags="azure-service-management"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/06/2016"
    ms.author="cynthn"/>

# Create a virtual machine running Windows in the Azure classic portal
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Azure portal](virtual-machines-windows-tutorial.md)
> * [Azure classic portal](virtual-machines-windows-tutorial-classic-portal.md)
> * [PowerShell: Resource Manager deployment](virtual-machines-deploy-rmtemplates-powershell.md)
> * [PowerShell: Classic deployment](virtual-machines-ps-create-preconfigure-windows-vms.md)
> 
> 
<!-- HHTML comment in to break between the selector and the note in the include below-->

> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the [Resource Manager deployment model](virtual-machines-windows-tutorial.md).

This tutorial shows you how easy it is to create an Azure virtual machine (VM) running Windows in the Azure classic portal. We'll use a Windows Server image as an example, but that's just one of the many images Azure offers. Note that your image choices depend on your subscription. For example, Windows desktop images may be available to MSDN subscribers.

You can also create VMs using [your own images](virtual-machines-create-upload-vhd-windows-server.md). To learn about this and other methods, see [Different ways to create a Windows virtual machine](virtual-machines-windows-choices-create-vm.md).

> [AZURE.NOTE] <a name="note"></a>You need an Azure account to complete this tutorial:
  >
  > + You can [open an Azure account for free](/pricing/free-trial/?WT.mc_id=A261C142F): You get credits you can use to try out paid Azure services, and even after they're used up you can keep the account and use free Azure services, such as Websites. Your credit card will never be charged, unless you explicitly change your settings and ask to be charged.
  >
  > + You can [activate MSDN subscriber benefits](/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F): Your MSDN subscription gives you credits every month that you can use for paid Azure services.


## Video walkthrough
Here's a walkthrough of this tutorial.

[AZURE.VIDEO creating-a-windows-vm-on-microsoft-azure-classic-portal]AZURE.VIDEO creating-a-windows-vm-on-microsoft-azure-classic-portal]

## <a id="createvirtualmachine"> </a>How to create the virtual machine
This section shows you how to use the **From Gallery** option in the Azure classic portal to create the virtual machine. This option provides more configuration choices than the **Quick Create** option. For example, if you want to join a virtual machine to a virtual network, you'll need to use the **From Gallery** option.

> [!NOTE]
> You can also try the richer, customizable Azure portal to create a virtual machine, use enhanced monitoring and diagnostics, use Premium storage, and more. The available options for configuring a virtual machine in the two portals overlap substantially but aren't identical. For example, use the Azure portal to configure a virtual machine with Premium storage.
> 
> 
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


## Next steps
* Log on to the virtual machine. For instructions, see [Log on to a virtual machine running Windows Server](virtual-machines-log-on-windows-server.md).

* Attach a disk to store data. You can attach both empty disks and disks that contain data. For instructions, see the [Attach a data disk to a Windows virtual machine created with the classic deployment model](storage-windows-attach-disk.md).


