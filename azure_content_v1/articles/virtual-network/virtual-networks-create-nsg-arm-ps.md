<properties 
   pageTitle="How to create NSGs in ARM mode using PowerShell| Microsoft Azure"
   description="Learn how to create and deploy NSGs in ARM using PowerShell"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
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

# How to create NSGs in PowerShell
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-nsg-arm-pportal.md)
- [PowerShell](virtual-networks-create-nsg-arm-ps.md)
- [Azure CLI](virtual-networks-create-nsg-arm-cli.md)
- [ARM template](virtual-networks-create-nsg-arm-template.md)

You can use an NSG to control traffic to one or more virtual machines (VMs), role instances, network adapters (NICs), or subnets in your virtual network. An NSG contains access control rules that allow or deny traffic based on traffic direction, protocol, source address and port, and destination address and port. The rules of an NSG can be changed at any time, and changes are applied to all associated instances.

For more information about NSGs, visit [what is an NSG](virtual-networks-nsg.md). 

>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the Resource Manager deployment model. You can also [create NSGs in the classic deployment model](virtual-networks-create-nsg-classic-ps.md).

## Scenario

To better illustrate how to create NSGs, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-nsg-scenario-include/figure1.png)

In this scenario you will create an NSG for each subnet in the **TestVNet** virtual network, as described below: 

- **NSG-FrontEnd**. The front end NSG will be applied to the *FrontEnd* subnet, and contain two rules:	
	- **rdp-rule**. This rule will allow RDP traffic to the *FrontEnd* subnet.
	- **web-rule**. This rule will allow HTTP traffic to the *FrontEnd* subnet.
- **NSG-BackEnd**. The back end NSG will be applied to the *BackEnd* subnet, and contain two rules:	
	- **sql-rule**. This rule allows SQL traffic only from the *FrontEnd* subnet.
	- **web-rule**. This rule denies all internet bound traffic from the *BackEnd* subnet.

The combination of these rules create a DMZ-like scenario, where the back end subnet can only receive incoming traffic for SQL traffic from the front end subnet, and has no access to the Internet, while the front end subnet can communicate with the Internet, and receive incoming HTTP requests only.
 

The sample PowerShell commands below expect a simple environment already created based on the scenario above. If you want to run the commands as they are displayed in this document, first build the test environment by deploying [this template](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd), click **Deploy to Azure**, replace the default parameter values if necessary, and follow the instructions in the portal.

## How to create the NSG for the front end subnet
To create an NSG named named *NSG-FrontEnd* based on the scenario above, follow the steps below.

> [AZURE.NOTE] Azure PowerShell is currently available in two releases - 1.0 and 0.9.8. If you have existing scripts and do not want to change them right now, you can continue using the 0.9.8 release. When using the 1.0 release, you should carefully test your scripts in pre-production environments before using them in production to avoid unexpected impacts.
>
> 1.0 cmdlets follow the naming pattern {verb}-AzureRm{noun}; whereas, the 0.9.8 names do not include **Rm** (for example, New-AzureRmResourceGroup instead of New-AzureResourceGroup). When using Azure PowerShell 0.9.8, you must first enable the Resource Manager mode by running the **Switch-AzureMode AzureResourceManager** command. This command is not necessary in 1.0.
>
> New features will be added to only the 1.0 release. For information about the 1.0 release, including how to install and uninstall the release, see [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).


1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

2. Create a security rule allowing access from the Internet to port 3389.

        $rule1 = New-AzureRmNetworkSecurityRuleConfig -Name rdp-rule -Description "Allow RDP" `
         -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
         -SourceAddressPrefix Internet -SourcePortRange * `
         -DestinationAddressPrefix * -DestinationPortRange 3389
3. Create a security rule allowing access from the Internet to port 80. 

        $rule2 = New-AzureRmNetworkSecurityRuleConfig -Name web-rule -Description "Allow HTTP" `
         -Access Allow -Protocol Tcp -Direction Inbound -Priority 101 `
         -SourceAddressPrefix Internet -SourcePortRange * `
         -DestinationAddressPrefix * -DestinationPortRange 80
4. Add the rules created above to a new NSG named **NSG-FrontEnd**.

        $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location westus -Name "NSG-FrontEnd" `
         -SecurityRules $rule1,$rule2
5. Check the rules created in the NSG.

        $nsg

    Output showing just the security rules:

        SecurityRules        : [
                              {
                                "Name": "rdp-rule",
                                "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                                "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/rdp-rule",
                                "Description": "Allow RDP",
                                "Protocol": "Tcp",
                                "SourcePortRange": "*",
                                "DestinationPortRange": "3389",
                                "SourceAddressPrefix": "Internet",
                                "DestinationAddressPrefix": "*",
                                "Access": "Allow",
                                "Priority": 100,
                                "Direction": "Inbound",
                                "ProvisioningState": "Succeeded"
                              },
                              {
                                "Name": "web-rule",
                                "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                                "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/web-rule",
                                "Description": "Allow HTTP",
                                "Protocol": "Tcp",
                                "SourcePortRange": "*",
                                "DestinationPortRange": "80",
                                "SourceAddressPrefix": "Internet",
                                "DestinationAddressPrefix": "*",
                                "Access": "Allow",
                                "Priority": 101,
                                "Direction": "Inbound",
                                "ProvisioningState": "Succeeded"
                              }
                            ]
6. Associate the NSG created above to the *FrontEnd* subnet.

        $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
     Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd `
         -AddressPrefix 192.168.1.0/24 -NetworkSecurityGroup $nsg

    Output showing only the *FrontEnd* subnet settings, notice the value for the **NetworkSecurityGroup** property:

        Subnets           : [
                           {
                             "Name": "FrontEnd",
                             "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                             "AddressPrefix": "192.168.1.0/24",
                             "IpConfigurations": [
                               {
                                 "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1"
                               },
                               {
                                 "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1"
                               }
                             ],
                             "NetworkSecurityGroup": {
                               "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
                             },
                             "RouteTable": null,
                             "ProvisioningState": "Succeeded"
                           }


> [!WARNING]
> The output for the command above shows the content for the virtual network configuration object, which only exists on the computer where you are running PowerShell. You need to run the **Set-AzureRmVirtualNetwork** cmdlet to save these settings to Azure.
> 
> 
1. Save the new VNet settings to Azure.

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    Output showing just the NSG portion:

        "NetworkSecurityGroup": {
       "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
     }


## How to create the NSG for the back end subnet
To create an NSG named named *NSG-BackEnd* based on the scenario above, follow the steps below.

1. Create a security rule allowing access from the front end subnet to port 1433 (default port used by SQL Server).

        $rule1 = New-AzureRmNetworkSecurityRuleConfig -Name frontend-rule -Description "Allow FE subnet" `
         -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
         -SourceAddressPrefix 192.168.1.0/24 -SourcePortRange * `
         -DestinationAddressPrefix * -DestinationPortRange 1433
2. Create a security rule blocking access to the Internet. 

        $rule2 = New-AzureRmNetworkSecurityRuleConfig -Name web-rule -Description "Block Internet" `
         -Access Deny -Protocol * -Direction Outbound -Priority 200 `
         -SourceAddressPrefix * -SourcePortRange * `
         -DestinationAddressPrefix Internet -DestinationPortRange *
3. Add the rules created above to a new NSG named **NSG-BackEnd**.

        $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location westus `-Name "NSG-BackEnd" `
         -SecurityRules $rule1,$rule2
4. Associate the NSG created above to the *BackEnd* subnet.

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name BackEnd `
         -AddressPrefix 192.168.2.0/24 -NetworkSecurityGroup $nsg

    Output showing only the *BackEnd* subnet settings, notice the value for the **NetworkSecurityGroup** property:

        Subnets           : [
                   {
                     "Name": "BackEnd",
                     "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                     "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
                     "AddressPrefix": "192.168.2.0/24",
                     "IpConfigurations": [...],
                     "NetworkSecurityGroup": {
                       "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd"
                     },
                     "RouteTable": null,
                     "ProvisioningState": "Succeeded"
                   }
5. Save the new VNet settings to Azure.

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
