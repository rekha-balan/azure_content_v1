<properties
    pageTitle="About Azure Virtual Machines | Microsoft Azure"
    description="Learn about the basics of virtual machines in Azure using both deployment models."
    services="virtual-machines"
    documentationCenter=""
    authors="cynthn"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager,azure-service-management"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/23/2015"
    ms.author="cynthn"/>

# About Azure virtual machines
> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md). This article covers using both models, but Microsoft recommends that most new deployments use the Resource Manager model.

Azure Virtual Machines is one of several types of on-demand, scalable computing resources that Azure offers. Typically, you'll choose a virtual machine if you need more control over the computing environment than the other choices offer.

An Azure virtual machine gives you the flexibility of virtualization without having to buy and maintain the physical hardware that runs the virtual machine. However, you still need to maintain the virtual machine -- configuring, patching, and maintaining the operating system and any other software that runs on the virtual machine.

This table helps you compare the choices offered by Azure.

| Compute Options    | Audience   |
| ------------------ | --------   |
| [App Service]      | Scalable Web Apps, Mobile Apps, API Apps, and Logic Apps for any device |
| [Cloud Services]   | Highly available, scalable n-tier cloud apps with more control of the OS |
| [Virtual Machines] | Customized Windows and Linux VMs with complete control of the OS |

<a name="tellmevm"></a>
## Tell me about virtual machines

Azure Virtual Machines lets you create and use virtual machines in the cloud. Providing what's known as *Infrastructure as a Service (IaaS)*, virtual machine technology can be used in variety of ways. Some examples are:

- **Virtual machines (VMs) for development and test.** Development groups commonly use VMs because they offer a quick, easy way to create a computer with specific configurations required to code and test an application. Azure Virtual Machines provides a straightforward and economical way to create these VMs, use them, then delete them when they're no longer needed.
- **Running applications in the cloud.** It makes economic sense to run some applications in the public cloud. One example is an application that has large spikes in demand. Although you could equip your own data center with enough hardware to handle peak demand, that hardware might be underutilized much of the time. Running this application on Azure lets you pay for extra VMs only when you need them and shut them down when you don't. Or, suppose you're a start-up that needs on-demand computing resources quickly and with no commitment. Once again, Azure can be the right choice.
- **Extending your own datacenter into the public cloud.** When you use Azure Virtual Network, your organization can create a virtual network (VNET) that's an extension of your own on-premises network and add VMs to that VNET. This allows running applications such as [SharePoint](virtual-machines-sharepoint-infrastructure-services.md), [SQL Server](virtual-machines-sql-server-infrastructure-services.md) and others on an Azure VM. This approach might be easier to deploy or less expensive than running them in VMs your own datacenter.   
- **Disaster recovery.** Rather than paying continuously for a backup datacenter that's rarely used, IaaS-based disaster recovery lets you pay for the computing resources you need only when you really need them.  For example, if your primary datacenter goes down, you can create VMs running on Azure to run essential applications, then shut them down when they're no longer needed.

Like other virtual machines, a VM in Azure has an operating system, storage and networking capabilities and can run a wide variety of applications. You can use an image provided by Azure or one of it's partners, or use your own. Examples include various versions, editions and configurations of:
 
-	Windows Server 
-	Linux servers such as Suse, Ubuntu and CentOS
-	SQL Server
-	BizTalk Server 
-	SharePoint Server

Virtual machines use virtual hard disks (VHDs) to store their operating system (OS) and data. VHDs are also used for the images you can choose from to install an OS. The following figure shows this, as well as two of the tools for creating and managing your VMs.

<a name="fig_createvms"></a>
![vm_diagram](./media/virtual-machines-choose-me-content/diagram.png)

**Figure: Azure Virtual Machines provides Infrastructure as a Service.**

VMs can be managed using a browser-based portal, command-line tools with support for scripting, or directly through the REST API. Microsoft partners such as RightScale and ScaleXtreme also provide management services that rely on the REST API. 

Along with the OS, other configuration choices you have with VMs include:

- The size, which determines factors such as how many disks you can attach and the processing power. Azure offers a wide variety of sizes to support many types of uses. For details, see [Sizes for Virtual Machines](virtual-machines-size-specs.md).  
- The Azure region where your new VM will be hosted, such as in the US, Europe, or Asia. 
- VM extensions, which give your virtual machine additional capabilities, such as running anti-virus or using the Desired State Configuration feature of Windows PowerShell.

Other benefits to consider for VMs include:

**Pay-as-you-go** -- Azure charges an hourly price based on the VM’s size and operating system. For partial hours, Azure charges only for the minutes of use. Storage is priced and charged separately. For details, see [Virtual Machines Pricing](https://azure.microsoft.com/pricing/details/virtual-machines/).

**Resiliency** -- Azure monitors the physical hardware that hosts each running VM. If a physical server running a VM fails, Azure notices this, moves the VM to new hardware and restarts the VM. This process is sometimes called service healing. Azure also protects a virtual machine's data, by keeping redundant copies of the VHDs in blob storage. 





## Next steps
Try out a virtual machine by creating a simple virtual machine quickly. For instructions, see one of these tutorials:

[Create a virtual machine running Linux](virtual-machines-linux-tutorial.md)

[Create a virtual machine running Windows in the Azure portal](virtual-machines-windows-tutorial.md)

Because those tutorials help you create a virtual machine quickly, they show you only one way to do it instead of describing the various tools you can use to create virtual machines. For a summary of these tools and links to instructions, see:

[Different ways to create a Linux virtual machine](virtual-machines-linux-choices-create-vm.md)

[Different ways to create a Windows virtual machine](virtual-machines-windows-choices-create-vm.md)

If you're just starting to learn about Azure virtual machines, be sure to also check out the [Frequently asked questions](virtual-machines-questions.md).

<!--links-->

[App Service]: app-service-choose-me.md
[Virtual Machines]: #tellmevm
[Cloud Services]: cloud-services-choose-me.md
