<properties 
   pageTitle="Control routing and use virtual appliances in Resource Manager using PowerShell | Microsoft Azure"
   description="Learn how to control routing and use virtual appliances in Azure PowerShell"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>

<tags  
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/20/2015"
   ms.author="telmos" />

# Create User Defined Routes (UDR) in PowerShell
> [AZURE.SELECTOR]
[PowerShell](virtual-network-create-udr-arm-ps.md)
[Azure CLI](virtual-network-create-udr-arm-cli.md)
[Template](virtual-network-create-udr-arm-template.md)


Although the use of system routes facilitates traffic automatically for your deployment, there are cases in which you want to control the routing of packets through a virtual appliance. You can do so by creating user defined routes that specify the next hop for packets flowing to a specific subnet to go to your virtual appliance instead, and enabling IP forwarding for the VM running as the virtual appliance.

Some of the cases where virtual appliances can be used include:

- Monitoring traffic with an intrusion detection system (IDS)
- Controlling traffic with a firewall

For more information about UDR and IP forwarding, visit [User Defined Routes and IP Forwarding](./virtual-networks-udr-overview.md).


>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the Resource Manager deployment model. You can also [create UDRs in the classic deployment model](virtual-network-create-udr-classic-ps.md).

## Scenario

To better illustrate how to create UDRs, this document will use the scenario below.

![IMAGE DESCRIPTION](./media/virtual-network-create-udr-scenario-include/figure1.png)

In this scenario you will create one UDR for the *Front end subnet* and another UDR for the *Back end subnet* , as described below: 

- **UDR-FrontEnd**. The front end UDR will be applied to the *FrontEnd* subnet, and contain one route:	
	- **RouteToBackend**. This route will send all traffic to the back end subnet to the **FW1** virtual machine.
- **UDR-BackEnd**. The back end UDR will be applied to the *BackEnd* subnet, and contain one route:	
	- **RouteToFrontend**. This route will send all traffic to the front end subnet to the **FW1** virtual machine.

The combination of these routes will ensure that all traffic destined from one subnet to another will be routed to the **FW1** virtual machine, which is being used as a virtual appliance. You also need to turn on IP forwarding for that VM, to ensure it can receive traffic destined to other VMs.


The sample PowerShell commands below expect a simple environment already created based on the scenario above. If you want to run the commands as they are displayed in this document, first build the test environment by deploying [this template](http://github.com/telmosampaio/azure-templates/tree/master/IaaS-NSG-UDR-Before), click **Deploy to Azure**, replace the default parameter values if necessary, and follow the instructions in the portal.

## Prerequisite: Install the Azure PowerShell Module
To perform the steps in this article, you'll need to [to install and configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

> [AZURE.NOTE] If you don't have an Azure account, you'll need one. Go sign up for a [free trial here](sign-up-organization.md). 

## Create the UDR for the front end subnet
To create the route table and route needed for the front end subnet based on the scenario above, follow the steps below.

> [AZURE.NOTE] Azure PowerShell is currently available in two releases - 1.0 and 0.9.8. If you have existing scripts and do not want to change them right now, you can continue using the 0.9.8 release. When using the 1.0 release, you should carefully test your scripts in pre-production environments before using them in production to avoid unexpected impacts.
>
> 1.0 cmdlets follow the naming pattern {verb}-AzureRm{noun}; whereas, the 0.9.8 names do not include **Rm** (for example, New-AzureRmResourceGroup instead of New-AzureResourceGroup). When using Azure PowerShell 0.9.8, you must first enable the Resource Manager mode by running the **Switch-AzureMode AzureResourceManager** command. This command is not necessary in 1.0.
>
> New features will be added to only the 1.0 release. For information about the 1.0 release, including how to install and uninstall the release, see [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).


1. Create a route used to send all traffic destined to the back end subnet (192.168.2.0/24) to be routed to the **FW1** virtual appliance (192.168.0.4).

        $route = New-AzureRmRouteConfig -Name RouteToBackEnd `
         -AddressPrefix 192.168.2.0/24 -NextHopType VirtualAppliance `
         -NextHopIpAddress 192.168.0.4
2. Create a route table named **UDR-FrontEnd** in the **westus** region that contains the route created above.

        $routeTable = New-AzureRmRouteTable -ResourceGroupName TestRG -Location westus `
         -Name UDR-FrontEnd -Route $route
3. Create a variable that contains the VNet where the subnet is. In our scenario, the VNet is named **TestVNet**.

        $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
4. Associate the route table created above to the **FrontEnd** subnet.

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd `
         -AddressPrefix 192.168.1.0/24 -RouteTable $routeTable


> [!WARNING]
> The output for the command above shows the content for the virtual network configuration object, which only exists on the computer where you are running PowerShell. You need to run the **Set-AzureVirtualNetwork** cmdlet to save these settings to Azure.
> 
> 
1. Save the new subnet configuration in Azure.

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    Expected output:

        Name              : TestVNet
     ResourceGroupName : TestRG
     Location          : westus
     Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
     Etag              : W/"7df26c0e-652f-4754-bc4e-733fef7d5b2b"
     ProvisioningState : Succeeded
     Tags              : 
                         Name         Value
                         ===========  =====
                         displayName  VNet 

     AddressSpace      : {
                           "AddressPrefixes": [
                             "192.168.0.0/16"
                           ]
                         }
     DhcpOptions       : {
                           "DnsServers": null
                         }
     NetworkInterfaces : null
     Subnets           : [
                             ...,
                           {
                             "Name": "FrontEnd",
                             "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                             "AddressPrefix": "192.168.1.0/24",
                             "IpConfigurations": [
                               {
                                 "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB2/ipConfigurations/ipconfig1"
                               },
                               {
                                 "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB1/ipConfigurations/ipconfig1"
                               }
                             ],
                             "NetworkSecurityGroup": {
                               "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
                             },
                             "RouteTable": {
                               "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-FrontEnd"
                             },
                             "ProvisioningState": "Succeeded"
                           },
                             ...
                         ]    


## Create the UDR for the back end subnet
To create the route table and route needed for the back end subnet based on the scenario above, follow the steps below.

1. Create a route used to send all traffic destined to the front end subnet (192.168.1.0/24) to be routed to the **FW1** virtual appliance (192.168.0.4).

        $route = New-AzureRmRouteConfig -Name RouteToFrontEnd `
         -AddressPrefix 192.168.1.0/24 -NextHopType VirtualAppliance `
         -NextHopIpAddress 192.168.0.4
2. Create a route table named **UDR-BackEnd** in the **uswest** region that contains the route created above.

        $routeTable = New-AzureRmRouteTable -ResourceGroupName TestRG -Location westus `
         -Name UDR-BackEnd -Route $route
3. Associate the route table created above to the **BackEnd** subnet.

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name BackEnd `
         -AddressPrefix 192.168.2.0/24 -RouteTable $routeTable
4. Save the new subnet configuration in Azure.

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    Expected output:

        Name              : TestVNet
     ResourceGroupName : TestRG
     Location          : westus
     Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
     Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
     ProvisioningState : Succeeded
     Tags              : 
                         Name         Value
                         ===========  =====
                         displayName  VNet 

     AddressSpace      : {
                           "AddressPrefixes": [
                             "192.168.0.0/16"
                           ]
                         }
     DhcpOptions       : {
                           "DnsServers": null
                         }
     NetworkInterfaces : null
     Subnets           : [
                           ...,
                           {
                             "Name": "BackEnd",
                             "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
                             "AddressPrefix": "192.168.2.0/24",
                             "IpConfigurations": [
                               {
                                 "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICSQL2/ipConfigurations/ipconfig1"
                               },
                               {
                                 "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICSQL1/ipConfigurations/ipconfig1"
                               }
                             ],
                             "NetworkSecurityGroup": {
                               "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BacEnd"
                             },
                             "RouteTable": {
                               "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-BackEnd"
                             },
                             "ProvisioningState": "Succeeded"
                           }
                         ]


## Enable IP forwarding on FW1
To enable IP forwarding in the NIC used by **FW1**, follow the steps below.

1. Create a variable that contains the settings for the NIC used by FW1. In our scenario, the NIC is named **NICFW1**.

        $nicfw1 = Get-AzureRmNetworkInterface -ResourceGroupName TestRG -Name NICFW1
2. Enable IP forwarding, and save the NIC settings.

        $nicfw1.EnableIPForwarding = 1
     Set-AzureRmNetworkInterface -NetworkInterface $nicfw1

    Expected output:

        Name                 : NICFW1
     ResourceGroupName    : TestRG
     Location             : westus
     Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICFW1
     Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
     ProvisioningState    : Succeeded
     Tags                 : 
                            Name         Value                  
                            ===========  =======================
                            displayName  NetworkInterfaces - DMZ

     VirtualMachine       : {
                              "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/FW1"
                            }
     IpConfigurations     : [
                              {
                                "Name": "ipconfig1",
                                "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                                "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICFW1/ipConfigurations/ipconfig1",
                                "PrivateIpAddress": "192.168.0.4",
                                "PrivateIpAllocationMethod": "Static",
                                "Subnet": {
                                  "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/DMZ"
                                },
                                "PublicIpAddress": {
                                  "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPFW1"
                                },
                                "LoadBalancerBackendAddressPools": [],
                                "LoadBalancerInboundNatRules": [],
                                "ProvisioningState": "Succeeded"
                              }
                            ]
     DnsSettings          : {
                              "DnsServers": [],
                              "AppliedDnsServers": [],
                              "InternalDnsNameLabel": null,
                              "InternalFqdn": null
                            }
     EnableIPForwarding   : True
     NetworkSecurityGroup : null
     Primary              : True


