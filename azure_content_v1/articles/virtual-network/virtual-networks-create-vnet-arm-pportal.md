<properties
   pageTitle="Create a virtual network using the Azure preview portal | Microsoft Azure"
   description="Learn how to create a virtual network using the Azure preview portal in ARM | Resource Manager."
   services="virtual-network"
   documentationCenter=""
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/07/2015"
   ms.author="telmos"/>

# Create a virtual network using the Azure preview portal
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-vnet-arm-pportal.md)
- [PowerShell](virtual-networks-create-vnet-arm-ps.md)
- [Azure CLI](virtual-networks-create-vnet-arm-cli.md)
- [ARM template](virtual-networks-create-vnet-arm-template-click.md)

An Azure virtual network (VNet) is a representation of your own network in the cloud. You can control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing. You can also further segment your VNet into subnets and deploy Azure IaaS virtual machines (VMs) and PaaS role instances, in the same way you can deploy physical and virtual machines to your on-premises datacenter. In essence, you can expand your network to Azure, bringing your own IP address blocks. Read the [virtual network overview](virtual-networks-overview.md) if you are not familiar with VNets.



>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This document covers creating a VNet by using the Resource Manager deployment model. You can also [create a virtual network in the classic deployment model by using the Azure preview portal](virtual-networks-create-vnet-classic-pportal.md).

## Scenario

To better illustrate how to create a VNet and subnets, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-vnet-scenario-include/vnet-scenario.png)

In this scenario you will create a VNet named **TestVNet** with a reserved CIDR block of **192.168.0.0./16**. Your VNet will contain the following subnets: 

- **FrontEnd**, using **192.168.1.0/24** as its CIDR block.
- **BackEnd**, using **192.168.2.0/24** as its CIDR block.

 

## How to create a VNet in the Azure preview portal

To create a VNet based on the scenario above by using the Azure preview portal, follow the steps below.

1. From a browser, navigate to http://portal.azure.com and, if necessary, sign in with your Azure account.
2. Click **NEW** > **Networking** > **Virtual network**, then click **Resource Manager** from the **Select a deployment model** list, and then click **Create**, as seen in the figure below.

	![Create VNet in preview portal](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure1.gif)

3. On the **Create virtual network** blade, configure the VNet settings as shown in the figure below.

	![Create virtual network blade](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure2.png)

4. Click **Resource group** and select a resource group to add the VNet to, or click **Create new** to add the VNet to a new resource group. The figure below shows the resource group settings for a new resource group called **TestRG**. For more information about resource groups, visit [Azure Resource Manager Overview](resource-group-overview.md/#resource-groups).

	![Resource group](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure3.png)

5. If necessary, change the **Subscription** and **Location** settings for your VNet. 

6. If you do not want to see the VNet as a tile in the **Startboard**, disable **Pin to Startboard**. 

7. Click **Create** and notice the tile named **Creating Virtual network** as shown in the figure below.

	![Creating virtual network tile](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure4.png)

8. Wait for the VNet to be created, then in the **Virtual network** blade, click **All settings** > **Subnets** > **Add** as seen below.

	![Adding subnet in the preview portal](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure5.gif)

9. Specify the subnet settings for the *BackEnd* subnet, as shown below, and then click **OK**. 

	![Subnet settings](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure6.png)

10. Notice the list of subnets, as shown in the figure below.

	![List of subnets in VNet](./media/virtual-networks-create-vnet-arm-pportal-include/vnet-create-arm-pportal-figure7.png)

