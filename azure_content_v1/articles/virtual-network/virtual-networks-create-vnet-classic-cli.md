<properties
   pageTitle="Create a virtual network using Azure CLI | Microsoft Azure"
   description="Learn how to create a virtual network using Azure CLI in ARM | Resource Manager."
   services="virtual-network"
   documentationCenter=""
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/07/2015"
   ms.author="telmos"/>

# Create a virtual network (classic) by using the Azure CLI
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-vnet-classic-pportal.md)
- [Azure portal](virtual-networks-create-vnet-classic-portal.md)
- [PowerShell](virtual-networks-create-vnet-classic-netcfg-ps.md)
- [Azure CLI](virtual-networks-create-vnet-classic-cli.md)

An Azure virtual network (VNet) is a representation of your own network in the cloud. You can control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing. You can also further segment your VNet into subnets and deploy Azure IaaS virtual machines (VMs) and PaaS role instances, in the same way you can deploy physical and virtual machines to your on-premises datacenter. In essence, you can expand your network to Azure, bringing your own IP address blocks. Read the [virtual network overview](virtual-networks-overview.md) if you are not familiar with VNets.



>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This document covers creating a VNet by using the classic deployment model. You can also [create a virtual network in the Resource Manager deployment model by using the Azure CLI](virtual-networks-create-vnet-arm-cli.md).

## Scenario

To better illustrate how to create a VNet and subnets, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-vnet-scenario-include/vnet-scenario.png)

In this scenario you will create a VNet named **TestVNet** with a reserved CIDR block of **192.168.0.0./16**. Your VNet will contain the following subnets: 

- **FrontEnd**, using **192.168.1.0/24** as its CIDR block.
- **BackEnd**, using **192.168.2.0/24** as its CIDR block.

 

## How to create a classic VNet using Azure CLI

You can use the Azure CLI to manage your Azure resources from the command prompt from any computer running Windows, Linux, or OSX. To create a VNet by using the Azure CLI, follow the steps below.

1. If you have never used Azure CLI, see [Install and Configure the Azure CLI](xplat-cli-install.md) and follow the instructions up to the point where you select your Azure account and subscription.
2. Run the **azure network vnet create** command to create a VNet and a subnet, as shown below. The list shown after the output explains the parameters used.

			azure network vnet create --vnet TestVNet -e 192.168.0.0 -i 16 -n FrontEnd -p 192.168.1.0 -r 24 -l "Central US"
	
	Expected output:

			info:    Executing command network vnet create
			+ Looking up network configuration
			+ Looking up locations
			+ Setting network configuration
			info:    network vnet create command OK

	- **--vnet**. Name of the VNet to be created. For our scenario, *TestVNet*
	- **-e (or --address-space)**. VNet address space. For our scenario, *192.168.0.0*
	- **-i (or -cidr)**. Network mask in CIDR format. For our scenario, *16*.
	- **-n (or --subnet-name**). Name of the first subnet. For our scenario, *FrontEnd*.
	- **-p (or --subnet-start-ip)**. Starting IP address for subnet, or subnet address space. For our scenario, *192.168.1.0*.
	- **-r (or --subnet-cidr)**. Network mask in CIDR format for subnet. For our scenario, *24*.
	- **-l (or --location)**. Azure region where the VNet will be created. For our scenario, *Central US*.

3. Run the **azure network vnet subnet create** command to create a subnet as shown below. The list shown after the output explains the parameters used.

			azure network vnet subnet create -t TestVNet -n BackEnd -a 192.168.2.0/24
	
	Here is the expected output for the command above:

			info:    Executing command network vnet subnet create
			+ Looking up network configuration
			+ Creating subnet "BackEnd"
			+ Setting network configuration
			+ Looking up the subnet "BackEnd"
			+ Looking up network configuration
			data:    Name                            : BackEnd
			data:    Address prefix                  : 192.168.2.0/24
			info:    network vnet subnet create command OK

	- **-t (or --vnet-name**. Name of the VNet where the subnet will be created. For our scenario, *TestVNet*.
	- **-n (or --name)**. Name of the new subnet. For our scenario, *BackEnd*.
	- **-a (or --address-prefix)**. Subnet CIDR block. Four our scenario, *192.168.2.0/24*.

4. Run the **azure network vnet show** command to view the properties of the new vnet, as shown below.

			azure network vnet show

	Here is the expected output for the command above:

			info:    Executing command network vnet show
			Virtual network name: TestVNet
			+ Looking up the virtual network sites
			data:    Name                            : TestVNet
			data:    Location                        : Central US
			data:    State                           : Created
			data:    Address space                   : 192.168.0.0/16
			data:    Subnets:
			data:      Name                          : FrontEnd
			data:      Address prefix                : 192.168.1.0/24
			data:
			data:      Name                          : BackEnd
			data:      Address prefix                : 192.168.2.0/24
			data:
			info:    network vnet show command OK


