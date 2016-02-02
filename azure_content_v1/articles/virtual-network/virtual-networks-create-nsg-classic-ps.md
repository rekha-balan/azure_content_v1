<properties 
   pageTitle="How to create NSGs in classic mode using PowerShell| Microsoft Azure"
   description="Learn how to create and deploy NSGs in classic mode using PowerShell"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
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

# How to create NSGs (classic) in PowerShell
> [AZURE.SELECTOR]
- [PowerShell](virtual-networks-create-nsg-classic-ps.md)
- [Azure CLI](virtual-networks-create-nsg-classic-cli.md)

You can use an NSG to control traffic to one or more virtual machines (VMs), role instances, network adapters (NICs), or subnets in your virtual network. An NSG contains access control rules that allow or deny traffic based on traffic direction, protocol, source address and port, and destination address and port. The rules of an NSG can be changed at any time, and changes are applied to all associated instances.

For more information about NSGs, visit [what is an NSG](virtual-networks-nsg.md). 

>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the classic deployment model. You can also [create NSGs in the Resource Manager deployment model](virtual-networks-create-nsg-arm-ps.md).

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
 

The sample PowerShell commands below expect a simple environment already created based on the scenario above. If you want to run the commands as they are displayed in this document, first build the test environment by [creating a VNet](virtual-networks-create-vnet-classic-netcfg-ps.md).

## How to create the NSG for the front end subnet
To create an NSG named named *NSG-FrontEnd* based on the scenario above, follow the steps below:

1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

2. Create a network security group named **NSG-FrontEnd**.

        New-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" -Location uswest `
         -Label "Front end subnet NSG"

    Expected output:

        Name         Location   Label               
     ----         --------   -----               
     NSG-FrontEnd West US     Front end subnet NSG



1. Create a security rule allowing access from the Internet to port 3389. 

        Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
     | Set-AzureNetworkSecurityRule -Name rdp-rule `
         -Action Allow -Protocol TCP -Type Inbound -Priority 100 `
         -SourceAddressPrefix Internet  -SourcePortRange '*' `
         -DestinationAddressPrefix '*' -DestinationPortRange '3389' 

    Expected output:

        Name     : NSG-FrontEnd
     Location : Central US
     Label    : Front end subnet NSG
     Rules    : 

                   Type: Inbound

                Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                        Prefix          Range         Address Prefix   Port Range             
                ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                rdp-rule             100       Allow    INTERNET        *             *                3389           TCP     
                ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
                BALANCER INBOUND                        CER                                                                   
                DENY ALL INBOUND     65500     Deny     *               *             *                *              *       



                      Type: Outbound

                   Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                           Prefix          Range         Address Prefix   Port Range             
                   ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                   ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                   ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
                   OUTBOUND                                                                                                      
                   DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *

1. Create a security rule allowing access from the Internet to port 80. 

        Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
     | Set-AzureNetworkSecurityRule -Name web-rule `
         -Action Allow -Protocol TCP -Type Inbound -Priority 200 `
         -SourceAddressPrefix Internet  -SourcePortRange '*' `
         -DestinationAddressPrefix '*' -DestinationPortRange '80' 

    Expected output:


        Name     : NSG-FrontEnd
        Location : Central US
        Label    : Front end subnet NSG
        Rules    : 

                      Type: Inbound

                   Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                           Prefix          Range         Address Prefix   Port Range             
                   ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                   rdp-rule             100       Allow    INTERNET        *             *                3389           TCP     
                   web-rule             200       Allow    INTERNET        *             *                80             TCP     
                   ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                   ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
                   BALANCER INBOUND                        CER                                                                   
                   DENY ALL INBOUND     65500     Deny     *               *             *                *              *       


                      Type: Outbound

                   Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                           Prefix          Range         Address Prefix   Port Range             
                   ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                   ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                   ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
                   OUTBOUND                                                                                                      
                   DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *   

## How to create the NSG for the back end subnet
1. Create a network security group named **NSG-BackEnd**.

        New-AzureNetworkSecurityGroup -Name "NSG-BackEnd" -Location uswest `
         -Label "Back end subnet NSG"

    Expected output:

        Name        Location   Label              
     ----        --------   -----              
     NSG-BackEnd West US    Back end subnet NSG



1. Create a security rule allowing access from the front end subnet to port 1433 (default port used by SQL Server). 

        Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
     | Set-AzureNetworkSecurityRule -Name rdp-rule `
         -Action Allow -Protocol TCP -Type Inbound -Priority 100 `
         -SourceAddressPrefix 192.168.1.0/24  -SourcePortRange '*' `
         -DestinationAddressPrefix '*' -DestinationPortRange '1433' 

    Expected output:

        Name     : NSG-BackEnd
     Location : Central US
     Label    : Back end subnet NSG
     Rules    : 

                   Type: Inbound

                Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                        Prefix          Range         Address Prefix   Port Range             
                ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                fe-rule              100       Allow    192.168.1.0/24  *             *                1433           TCP     
                ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
                BALANCER INBOUND                        CER                                                                   
                DENY ALL INBOUND     65500     Deny     *               *             *                *              *       



                      Type: Outbound

                   Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                           Prefix          Range         Address Prefix   Port Range             
                   ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                   ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                   ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
                   OUTBOUND                                                                                                      
                   DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *      

1. Create a security rule blocking access from the subnet to the Internet. 

        Get-AzureNetworkSecurityGroup -Name "NSG-BackEnd" `
     | Set-AzureNetworkSecurityRule -Name block-internet `
         -Action Deny -Protocol '*' -Type Outbound -Priority 200 `
         -SourceAddressPrefix '*'  -SourcePortRange '*' `
         -DestinationAddressPrefix Internet -DestinationPortRange '*' 

    Expected output:

        Name     : NSG-BackEnd
     Location : Central US
     Label    : Back end subnet NSG
     Rules    : 

                   Type: Inbound

                Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                        Prefix          Range         Address Prefix   Port Range             
                ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                fe-rule              100       Allow    192.168.1.0/24  *             *                1433           TCP     
                ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
                BALANCER INBOUND                        CER                                                                   
                DENY ALL INBOUND     65500     Deny     *               *             *                *              *       



                      Type: Outbound

                   Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
                                                           Prefix          Range         Address Prefix   Port Range             
                   ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
                   block-internet       200       Deny     *               *             INTERNET         *              *       
                   ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
                   ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
                   OUTBOUND                                                                                                      
                   DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *   