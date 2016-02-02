<properties
   pageTitle="Create a virtual network using the Azure portal | Microsoft Azure"
   description="Learn how to create a virtual network using the Azure portal."
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

# Create a virtual network (classic) by using the Azure portal
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-vnet-classic-pportal.md)
- [Azure portal](virtual-networks-create-vnet-classic-portal.md)
- [PowerShell](virtual-networks-create-vnet-classic-netcfg-ps.md)
- [Azure CLI](virtual-networks-create-vnet-classic-cli.md)

An Azure virtual network (VNet) is a representation of your own network in the cloud. You can control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing. You can also further segment your VNet into subnets and deploy Azure IaaS virtual machines (VMs) and PaaS role instances, in the same way you can deploy physical and virtual machines to your on-premises datacenter. In essence, you can expand your network to Azure, bringing your own IP address blocks. Read the [virtual network overview](virtual-networks-overview.md) if you are not familiar with VNets.



>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This document covers creating a VNet by using the classic deployment model. You can also [create a virtual network in the Resource Manager deployment model by using the Azure preview portal](virtual-networks-create-vnet-arm-pportal.md).

You will learn to create a VNet from the Azure portal by using the UI, and by deploying a netcfg file.

## Scenario

To better illustrate how to create a VNet and subnets, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-vnet-scenario-include/vnet-scenario.png)

In this scenario you will create a VNet named **TestVNet** with a reserved CIDR block of **192.168.0.0./16**. Your VNet will contain the following subnets: 

- **FrontEnd**, using **192.168.1.0/24** as its CIDR block.
- **BackEnd**, using **192.168.2.0/24** as its CIDR block.

 

## How to create a VNet in the Azure portal

To create a VNet based on the scenario above, follow the steps below.

1. From a browser, navigate to http://manage.windowsazure.com and, if necessary, sign in with your Azure account.
2. Click **NEW** > **NETWORK SERVICES** > **VIRTUAL NETWORK** > **CUSTOM CREATE** as shown in the figure below.

	![Create VNet in portal](./media/virtual-networks-create-vnet-classic-portal-include/vnet-create-portal-figure1.gif)

3. On the **Virtual Network Details** page, type the **NAME** of the VNet, select its **LOCATION**, and then click on the arrow on the bottom right hand corner of the page to advance to step 2. The figure below shows the settings for our scenario.

	![Virtual network details page](./media/virtual-networks-create-vnet-classic-portal-include/vnet-create-portal-figure2.png)

4. On the **DNS Servers and VPN Connectivity** page, specify the name and IP address for up to 9 DNS servers to use. If you do not specify a DNS server, your VNet will use the internal naming resolution resolution provided by Azure. For our scenario, we will not configure DNS servers.
5. If you want to provide point-to-site VPN access to your VNet, enable the **Configure a point-to-site VPN** checkbox. If you do not configure a point-to-site VPN, you can add it to your VNet at any time after creation. For our scenario, we will not configure a point-to-site VPN.
6. If you want to provide site-to-site VPN connectivity between your VNet and another VNet or your on-premises network, enable the **Configure a site-to-site VPN** checkbox and specify if you want to use **ExpressRoute** or note, and the name of the network to connect to. If you do not configure a site-to-site VPN, you can add it to your VNet at any time after creation. For our scenario, we will not configure a site-to-site VPN.
7. Click on the arrow on the bottom right hand corner of the page to advance to step 3.The figure below shows the settings for our scenario.

	![DNS Servers and VPN connectivity page](./media/virtual-networks-create-vnet-classic-portal-include/vnet-create-portal-figure3.png)

8. On the **Virtual Network Address Spaces** page, under **STARTING IP**, click on *10.0.0.0* to change the VNet address space, and then type the starting address space you want to use. For our scenario, type *192.168.0.0*. 
9. Under **CIDR (ADDRESS COUNT)** select the number of bits for the subnet mask. For our scenario, select *16 (65536)*.
10. Under **SUBNETS**, click *Subnet-1* and rename the subnet if necessary. For our scenario, rename it to *FrontEnd*.

	>[AZURE.NOTE] If you click outside the name textbox for a subnet you will not be able to edit the name if the subnet again. To fix that, you need to remove the subnet by clicking on the X button to its right, then add a new subnet as described in step 13 below.

11. Under **STARTING IP** for the first subnet, specify the starting IP address for the subnet. For our scenario, type *192.168.1.0*.
12. Under **CIDR (ADDRESS COUNT)** select the number of bits for the subnet mask for the first subnet. For our scenario, select *24 (256)*.
13. Click **add subnet** to add a new subnet, if necessary. For our scenario, add a subnet and repeat steps 10 to 12 to configure the VNet as shown in the figure below.

	![Virtual network address spaces page](./media/virtual-networks-create-vnet-classic-portal-include/vnet-create-portal-figure4.png)

14. Click on the check mark button on the bottom right hand corner of the page to create the VNet. After a few seconds your VNet will be shown in the list of available VNets, as shown in the figure below.

	![New virtual network](./media/virtual-networks-create-vnet-classic-portal-include/vnet-create-portal-figure5.png)

## How to create a VNet using a network config file in the Azure portal

Azure uses an xml file to define all VNets available to a subscription. You can download this file, and edit it to modify or delete existing VNets, and create new ones. In this document, you will learn how to download this file, referred to as network configuration (or netcgf) file, and edit it to create a new VNet. Check the [Azure virtual network configuration schema](https://msdn.microsoft.com/library/azure/jj157100.aspx) to learn more about the network configuration file.

To create a VNet using a netcfg file through the Azure portal, follow the steps below.

1. From a browser, navigate to http://manage.windowsazure.com and, if necessary, sign in with your Azure account.
2. Scroll down on the list of services, and click on **NETWORKS** as seen below.

	![Azure virtual networks](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure1.gif)

3. On the bottom of the page, click the **EXPORT** button, as shown below.

	![Export button](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure2.png)

4. On the **Export network configuration** page, select the subscription you want to export the virtual network configuration from, and then click the check mark button on the bottom left hand corner of the page.
5. Follow your browser instructions to save the **NetworkConfig.xml** file. Make sure you note where you are saving the file.
6. Open the file you saved in step 5 above using any XML or text editor application, and look for the **<VirtualNetworkSites>** element. If you have any networks already created, each network will be displayed as its own **<VirtualNetworkSite>** element.
7. To create the virtual network described in this scenario, add the following XML just under the **<VirtualNetworkSites>** element:

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

8.  Save the network configuration file.
9.  In the Azure portal, on the bottom left hand corner of the page, click **NEW**, then click **NETWORK SERVICES**, then click **VIRTUAL NETWORK**, and then click **IMPORT CONFIGURATION** as shown in the figure below.

	![Import configuration](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure3.gif)

10.  On the **Import the network configuration file** page, click **BROWSE FOR FILE...**, then navigate to the folder you saved your file in step 8 above, select the file, and then click **Open**. The web page should look similar to the figure below. On the bottom right hand corner of the page, click on the arrow button to move to the next step.

	![Import network configuration file page](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure4.png)

11.   On the **Building your network** page, notice the entry for your new VNet, as shown in the figure below.

	![Building your network page](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure5.png)

12.   Click on the check mark button on the bottom right hand corner of the page to create the VNet. After a few seconds your VNet will be shown in the list of available VNets, as shown in the figure below.

	![New virtual network](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure6.png)

