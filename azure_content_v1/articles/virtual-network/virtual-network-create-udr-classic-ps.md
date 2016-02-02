<properties 
   pageTitle="Control routing and use virtual appliances using PowerShell in the classic deployment model | Microsoft Azure"
   description="Learn how to control routing in VNets using PowerShell in the classic deployment model"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-service-management"
/>

<tags  
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/11/2015"
   ms.author="telmos" />

# Control routing and use virtual appliances (classic) using PowerShell
> [AZURE.SELECTOR]
[PowerShell](virtual-network-create-udr-classic-ps.md)
[Azure CLI](virtual-network-create-udr-classic-cli.md)


Although the use of system routes facilitates traffic automatically for your deployment, there are cases in which you want to control the routing of packets through a virtual appliance. You can do so by creating user defined routes that specify the next hop for packets flowing to a specific subnet to go to your virtual appliance instead, and enabling IP forwarding for the VM running as the virtual appliance.

Some of the cases where virtual appliances can be used include:

- Monitoring traffic with an intrusion detection system (IDS)
- Controlling traffic with a firewall

For more information about UDR and IP forwarding, visit [User Defined Routes and IP Forwarding](./virtual-networks-udr-overview.md).


>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the classic deployment model. You can also [ENTER ACTION FOR ARM HERE](armToken.md).

## Scenario

To better illustrate how to create UDRs, this document will use the scenario below.

![IMAGE DESCRIPTION](./media/virtual-network-create-udr-scenario-include/figure1.png)

In this scenario you will create one UDR for the *Front end subnet* and another UDR for the *Back end subnet* , as described below: 

- **UDR-FrontEnd**. The front end UDR will be applied to the *FrontEnd* subnet, and contain one route:	
	- **RouteToBackend**. This route will send all traffic to the back end subnet to the **FW1** virtual machine.
- **UDR-BackEnd**. The back end UDR will be applied to the *BackEnd* subnet, and contain one route:	
	- **RouteToFrontend**. This route will send all traffic to the front end subnet to the **FW1** virtual machine.

The combination of these routes will ensure that all traffic destined from one subnet to another will be routed to the **FW1** virtual machine, which is being used as a virtual appliance. You also need to turn on IP forwarding for that VM, to ensure it can receive traffic destined to other VMs.


The sample Azure PowerShell commands below expect a simple environment already created based on the scenario above. If you want to run the commands as they are displayed in this document, create the environment shown in [create a VNet (classic) using PowerShell](virtual-networks-create-vnet-classic-ps.md).

## Prerequisite: Install the Azure PowerShell Module
To perform the steps in this article, you'll need to [to install and configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

> [AZURE.NOTE] If you don't have an Azure account, you'll need one. Go sign up for a [free trial here](sign-up-organization.md). 

## Create the UDR for the front end subnet
To create the route table and route needed for the front end subnet based on the scenario above, follow the steps below.

1. Run the **`New-AzureRouteTable`** cmdlet to create a route table for the front end subnet.

        ```powershell
     New-AzureRouteTable -Name UDR-FrontEnd `
         -Location uswest `
         -Label "Route table for front end subnet"
     ```

    Output:

        Name         Location   Label                          
     ----         --------   -----                          
     UDR-FrontEnd West US    Route table for front end subnet
2. Run the **`Set-AzureRoute`** cmdlet to create a route in the route table created above to send all traffic destined to the back end subnet (192.168.2.0/24) to the **FW1** VM (192.168.0.4).

        ```powershell
     Get-AzureRouteTable UDR-FrontEnd `
         |Set-AzureRoute -RouteName RouteToBackEnd -AddressPrefix 192.168.2.0/24 `
         -NextHopType VirtualAppliance `
         -NextHopIpAddress 192.168.0.4
     ```

    Output:

        Name     : UDR-FrontEnd
     Location : West US
     Label    : Route table for frontend subnet
     Routes   : 
                Name                 Address Prefix    Next hop type        Next hop IP address
                ----                 --------------    -------------        -------------------
                RouteToBackEnd       192.168.2.0/24    VirtualAppliance     192.168.0.4  
3. Run the **`Set-AzureSubnetRouteTable`** cmdlet to associate the route table created above with the **FrontEnd** subnet.

        ```powershell
     Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
         -SubnetName FrontEnd `
         -RouteTableName UDR-FrontEnd
     ```


## Create the UDR for the back end subnet
To create the route table and route needed for the back end subnet based on the scenario above, follow the steps below.

1. Run the **`New-AzureRouteTable`** cmdlet to create a route table for the back end subnet.

        ```powershell
     New-AzureRouteTable -Name UDR-BackEnd `
         -Location uswest `
         -Label "Route table for back end subnet"
     ```
2. Run the **`Set-AzureRoute`** cmdlet to create a route in the route table created above to send all traffic destined to the front end subnet (192.168.1.0/24) to the **FW1** VM (192.168.0.4).

        ```powershell
     Get-AzureRouteTable UDR-BackEnd `
         |Set-AzureRoute -RouteName RouteToFrontEnd -AddressPrefix 192.168.1.0/24 `
         -NextHopType VirtualAppliance `
         -NextHopIpAddress 192.168.0.4
     ```
3. Run the **`Set-AzureSubnetRouteTable`** cmdlet to associate the route table created above with the **BackEnd** subnet.

        ```powershell
     Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
         -SubnetName FrontEnd `
         -RouteTableName UDR-FrontEnd
     ```
   ## Enable IP forwrding on the FW1 VM
   To enable IP forwarding in the FW1 VM, follow the steps below.

4. Run the **`Get-AzureIPForwarding`** cmdlet to chec the status of IP forwarding

        Get-AzureVM -Name FW1 -ServiceName TestRGFW `
         | Get-AzureIPForwarding

    Output:

        Disabled
5. Run the **`Set-AzureIPForwarding`** command to enable IP forwarding for the *FW1* VM.

        Get-AzureVM -Name FW1 -ServiceName TestRGFW `
         | Set-AzureIPForwarding -Enable
