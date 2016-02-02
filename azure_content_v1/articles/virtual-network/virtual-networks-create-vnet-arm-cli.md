<properties
   pageTitle="Create a virtual network using Azure CLI | Microsoft Azure"
   description="Learn how to create a virtual network using Azure CLI in ARM | Resource Manager."
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

# Create a virtual network by using the Azure CLI
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-vnet-arm-pportal.md)
- [PowerShell](virtual-networks-create-vnet-arm-ps.md)
- [Azure CLI](virtual-networks-create-vnet-arm-cli.md)
- [ARM template](virtual-networks-create-vnet-arm-template-click.md)

An Azure virtual network (VNet) is a representation of your own network in the cloud. You can control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing. You can also further segment your VNet into subnets and deploy Azure IaaS virtual machines (VMs) and PaaS role instances, in the same way you can deploy physical and virtual machines to your on-premises datacenter. In essence, you can expand your network to Azure, bringing your own IP address blocks. Read the [virtual network overview](virtual-networks-overview.md) if you are not familiar with VNets.



>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This document covers creating a VNet by using the Resource Manager deployment model. You can also [create a virtual network in the classic deployment model by using the Azure CLI](virtual-networks-create-vnet-classic-cli.md).

## Scenario

To better illustrate how to create a VNet and subnets, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-vnet-scenario-include/vnet-scenario.png)

In this scenario you will create a VNet named **TestVNet** with a reserved CIDR block of **192.168.0.0./16**. Your VNet will contain the following subnets: 

- **FrontEnd**, using **192.168.1.0/24** as its CIDR block.
- **BackEnd**, using **192.168.2.0/24** as its CIDR block.

 

## How to create a VNet using the Azure CLI

You can use the Azure CLI to manage your Azure resources from the command prompt from any computer running Windows, Linux, or OSX. To create a VNet by using the Azure CLI, follow the steps below.

1. If you have never used the Azure CLI, see [Install and Configure the Azure CLI](xplat-cli-install.md) and follow the instructions up to the point where you select your Azure account and subscription.
2. Run the **azure config mode** command to switch to Resource Manager mode, as shown below.

		azure config mode arm

	Here is the expected output for the command above:

		info:    New mode is arm

3. If necessary, run the **azure group create** to create a new resource group, as shown below. Notice the output of the command. The list shown after the output explains the parameters used. For more information about resource groups, visit [Azure Resource Manager Overview](resource-group-overview.md/#resource-groups).

		azure group create -n TestRG -l centralus

	Here is the expected output for the command above:

		info:    Executing command group create
		+ Getting resource group TestRG
		+ Creating resource group TestRG
		info:    Created resource group TestRG
		data:    Id:                  /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG
		data:    Name:                TestRG
		data:    Location:            centralus
		data:    Provisioning State:  Succeeded
		data:    Tags: null
		data:
		info:    group create command OK

	- **-n (or --name)**. Name for the new resource group. For our scenario, *TestRG*.
	- **-l (or --location)**. Azure region where the new resource group will be created. For our scenario, *centralus*.

4. Run the **azure network vnet create** command to create a VNet and a subnet, as shown below. 

		azure network vnet create -g TestRG -n TestVNet -a 192.168.0.0/16 -l centralus

	Here is the expected output for the command above:

		info:    Executing command network vnet create
		+ Looking up virtual network "TestVNet"
		+ Creating virtual network "TestVNet"
		+ Loading virtual network state
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet2
		data:    Name                            : TestVNet
		data:    Type                            : Microsoft.Network/virtualNetworks
		data:    Location                        : centralus
		data:    ProvisioningState               : Succeeded
		data:    Address prefixes:
		data:      192.168.0.0/16
		info:    network vnet create command OK

	- **-g (or --resource-group)**. Name of the resource group where the VNet will be created. For our scenario, *TestRG*.
	- **-n (or --name)**. Name of the VNet to be created. For our scenario, *TestVNet*
	- **-a (or --address-prefixes)**. List of CIDR blocks used for the VNet address space. For our scenario, *192.168.0.0/16*
	- **-l (or --location)**. Azure region where the VNet will be created. For our scenario, *centralus*.

5. Run the **azure network vnet subnet create** command to create a subnet as shown below. Notice the output of the command. The list shown after the output explains the parameters used.

		azure network vnet subnet create -g TestRG -e TestVNet -n FrontEnd -a 192.168.1.0/24

	Here is the expected output for the command above:

		info:    Executing command network vnet subnet create
		+ Looking up the subnet "FrontEnd"
		+ Creating subnet "FrontEnd"
		+ Looking up the subnet "FrontEnd"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:    Type                            : Microsoft.Network/virtualNetworks/subnets
		data:    ProvisioningState               : Succeeded
		data:    Name                            : FrontEnd
		data:    Address prefix                  : 192.168.1.0/24
		data:
		info:    network vnet subnet create command OK

	- **-e (or --vnet-name**. Name of the VNet where the subnet will be created. For our scenario, *TestVNet*.
	- **-n (or --name)**. Name of the new subnet. For our scenario, *FrontEnd*.
	- **-a (or --address-prefix)**. Subnet CIDR block. Four our scenario, *192.168.1.0/24*.

6. Repeat step 5 above to create other subnets, if necessary. For our scenario, run the command below to create the *BackEnd* subnet.

		azure network vnet subnet create -g TestRG -e TestVNet -n BackEnd -a 192.168.2.0/24

4. Run the **azure network vnet show** command to view the properties of the new vnet, as shown below.

		azure network vnet show -g TestRG -n TestVNet

	Here is the expected output for the command above:

		info:    Executing command network vnet show
		+ Looking up virtual network "TestVNet"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		data:    Name                            : TestVNet
		data:    Type                            : Microsoft.Network/virtualNetworks
		data:    Location                        : centralus
		data:    ProvisioningState               : Succeeded
		data:    Address prefixes:
		data:      192.168.0.0/16
		data:    Subnets:
		data:      Name                          : FrontEnd
		data:      Address prefix                : 192.168.1.0/24
		data:
		data:      Name                          : BackEnd
		data:      Address prefix                : 192.168.2.0/24
		data:
		info:    network vnet show command OK


