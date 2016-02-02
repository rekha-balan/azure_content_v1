<properties
   pageTitle="Create a virtual network using a network configuration file in the Azure portal | Microsoft Azure"
   description="Learn how to create a virtual network using a network configuration file in the Azure portal."
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

# Create a virtual network (classic) by using PowerShell
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-vnet-classic-pportal.md)
- [Azure portal](virtual-networks-create-vnet-classic-portal.md)
- [PowerShell](virtual-networks-create-vnet-classic-netcfg-ps.md)
- [Azure CLI](virtual-networks-create-vnet-classic-cli.md)

An Azure virtual network (VNet) is a representation of your own network in the cloud. You can control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing. You can also further segment your VNet into subnets and deploy Azure IaaS virtual machines (VMs) and PaaS role instances, in the same way you can deploy physical and virtual machines to your on-premises datacenter. In essence, you can expand your network to Azure, bringing your own IP address blocks. Read the [virtual network overview](virtual-networks-overview.md) if you are not familiar with VNets.



>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This document covers creating a VNet by using the classic deployment model. You can also [create a virtual network in the Resource Manager deployment model](virtual-networks-create-vnet-arm-ps.md).

## Scenario

To better illustrate how to create a VNet and subnets, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-vnet-scenario-include/vnet-scenario.png)

In this scenario you will create a VNet named **TestVNet** with a reserved CIDR block of **192.168.0.0./16**. Your VNet will contain the following subnets: 

- **FrontEnd**, using **192.168.1.0/24** as its CIDR block.
- **BackEnd**, using **192.168.2.0/24** as its CIDR block.

 

## How to create a VNet using a network config file from PowerShell

Azure uses an xml file to define all VNets available to a subscription. You can download this file, and edit it to modify or delete existing VNets, and create new ones. In this document, you will learn how to download this file, referred to as network configuration (or netcgf) file, and edit it to create a new VNet. Check the [Azure virtual network configuration schema](https://msdn.microsoft.com/library/azure/jj157100.aspx) to learn more about the network configuration file.

To create a VNet using a netcfg file using PowerShell, follow the steps below.

1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.
2. From the Azure PowerShell console, use the **Get-AzureVnetConfig** cmdlet to download the network configuration file by running the command below. 

		Get-AzureVNetConfig -ExportToFile c:\NetworkConfig.xml

	Expected output:

		XMLConfiguration                                                                                                     
		----------------                                                                                                     
		<?xml version="1.0" encoding="utf-8"?>...  

3. Open the file you saved in step 2 above using any XML or text editor application, and look for the **<VirtualNetworkSites>** element. If you have any networks already created, each network will be displayed as its own **<VirtualNetworkSite>** element.
4. To create the virtual network described in this scenario, add the following XML just under the **<VirtualNetworkSites>** element:

		<VirtualNetworkSite name="TestVNet" Location="Central US">
		  <AddressSpace>
		    <AddressPrefix>192.168.0.0/16</AddressPrefix>
		  </AddressSpace>
		  <Subnets>
		    <Subnet name="FrontEnd">
		      <AddressPrefix>192.168.1.0/24</AddressPrefix>
		    </Subnet>
		    <Subnet name="BackEnd">
		      <AddressPrefix>192.168.2.0/24</AddressPrefix>
		    </Subnet>
		  </Subnets>
		</VirtualNetworkSite>

9.  Save the network configuration file.
10. From the Azure PowerShell console, use the **Set-AzureVnetConfig** cmdlet to upload the network configuration file by running the command below. Notice the output under the command, you should see **Succeeded** under **OperationStatus**. If that is not the case, check the xml file for errors.

		Set-AzureVNetConfig -ConfigurationPath c:\NetworkConfig.xml

	Here is the expected output for the command above:

		OperationDescription OperationId                          OperationStatus
		-------------------- -----------                          ---------------
		Set-AzureVNetConfig  49579cb9-3f49-07c3-ada2-7abd0e28c4e4 Succeeded 
	
11. From the Azure PowerShell console, use the **Get-AzureVnetSite** cmdlet to verify that the new network was added by running the command below. 

		Get-AzureVNetSite -VNetName TestVNet

	Here is the expected output for the command above:

		AddressSpacePrefixes : {192.168.0.0/16}
		Location             : Central US
		AffinityGroup        : 
		DnsServers           : {}
		GatewayProfile       : 
		GatewaySites         : 
		Id                   : b953f47b-fad9-4075-8cfe-73ff9c98278f
		InUse                : False
		Label                : 
		Name                 : TestVNet
		State                : Created
		Subnets              : {FrontEnd, BackEnd}
		OperationDescription : Get-AzureVNetSite
		OperationId          : 3f35d533-1f38-09c0-b286-3d07cd0904d8
		OperationStatus      : Succeeded

