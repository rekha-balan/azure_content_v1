<properties 
   pageTitle="How to create NSGs in classic mode using the Azure CLI| Microsoft Azure"
   description="Learn how to create and deploy NSGs in classic mode using the Azure CLI"
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

# How to create NSGs (classic) in the Azure CLI
> [AZURE.SELECTOR]
- [PowerShell](virtual-networks-create-nsg-classic-ps.md)
- [Azure CLI](virtual-networks-create-nsg-classic-cli.md)

You can use an NSG to control traffic to one or more virtual machines (VMs), role instances, network adapters (NICs), or subnets in your virtual network. An NSG contains access control rules that allow or deny traffic based on traffic direction, protocol, source address and port, and destination address and port. The rules of an NSG can be changed at any time, and changes are applied to all associated instances.

For more information about NSGs, visit [what is an NSG](virtual-networks-nsg.md). 

>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the classic deployment model. You can also [create NSGs in the Resource Manager deployment model](virtual-networks-create-nsg-arm-cli.md).

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
 

The sample Azure CLI commands below expect a simple environment already created based on the scenario above. If you want to run the commands as they are displayed in this document, first build the test environment by [creating a VNet](virtual-networks-create-vnet-classic-cli.md).

## How to create the NSG for the front end subnet
To create an NSG named named *NSG-FrontEnd* based on the scenario above, follow the steps below.

1. If you have never used Azure CLI, see [Install and Configure the Azure CLI](xplat-cli-install.md) and follow the instructions up to the point where you select your Azure account and subscription.

2. Run the **azure config mode** command to switch to classic mode, as shown below.

        azure config mode asm

    Expected output:

        info:    New mode is asm
3. Run the **azure network nsg create** command to create an NSG.

        azure network nsg create -l uswest -n NSG-FrontEnd

    Expected output:

        info:    Executing command network nsg create
     info:    Creating a network security group "NSG-FrontEnd"
     info:    Looking up the network security group "NSG-FrontEnd"
     data:    Name                            : NSG-FrontEnd
     data:    Location                        : West US
     data:    Security group rules:
     data:    Name                               Source IP           Source Port  Destination IP   Destination Port  Protocol  Type      Action  Prior
     ity  Default
     data:    ---------------------------------  ------------------  -----------  ---------------  ----------------  --------  --------  ------  -----
     ---  -------
     data:    ALLOW VNET OUTBOUND                VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Outbound  Allow   65000
          true   
     data:    ALLOW VNET INBOUND                 VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Inbound   Allow   65000
          true   
     data:    ALLOW AZURE LOAD BALANCER INBOUND  AZURE_LOADBALANCER  *            *                *                 *         Inbound   Allow   65001
          true   
     data:    ALLOW INTERNET OUTBOUND            *                   *            INTERNET         *                 *         Outbound  Allow   65001
          true   
     data:    DENY ALL OUTBOUND                  *                   *            *                *                 *         Outbound  Deny    65500
          true   
     data:    DENY ALL INBOUND                   *                   *            *                *                 *         Inbound   Deny    65500
          true   
     info:    network nsg create command OK

    Parameters:

   * **-l (or --location)**. Azure region where the new NSG will be created. For our scenario, *westus*.
* **-n (or --name)**. Name for the new NSG. For our scenario, *NSG-FrontEnd*.

4. Run the **azure network nsg rule create** command to create a rule that allows access to port 3389 (RDP) from the Internet.

        azure network nsg rule create -a NSG-FrontEnd -n rdp-rule -c Allow -p Tcp -r Inbound -y 100 -f Internet -o * -e * -u 3389

    Expected output:

        info:    Executing command network nsg rule create
     info:    Looking up the network security group "NSG-FrontEnd"
     info:    Creating a network security rule "rdp-rule"
     info:    Looking up the network security group "NSG-FrontEnd"
     data:    Name                            : rdp-rule
     data:    Source address prefix           : INTERNET
     data:    Source Port                     : *
     data:    Destination address prefix      : *
     data:    Destination Port                : 3389
     data:    Protocol                        : TCP
     data:    Type                            : Inbound
     data:    Action                          : Allow
     data:    Priority                        : 100
     info:    network nsg rule create command OK

    Parameters:

   * **-a (or --nsg-name)**. Name of the NSG in which the rule will be created. For our scenario, *NSG-FrontEnd*.
* **-n (or --name)**. Name for the new rule. For our scenario, *rdp-rule*.
* **-c (or --action)**. Access level for the rule (Deny or Allow).
* **-p (or --protocol)**. Protocol (Tcp, Udp, or *) for the rule.
* **-r (or --type)**. Direction of connection (Inbound or Outbound).
* **-y (or --priority)**. Priority for the rule.
* **-f (or --source-address-prefix)**. Source address prefix in CIDR or using default tags.
* **-o (or --source-port-range)**. Source port, or port range.
* **-e (or --destination-address-prefix)**. Destination address prefix in CIDR or using default tags.
* **-u (or --destination-port-range)**. Destination port, or port range.    

5. Run the **azure network nsg rule create** command to create a rule that allows access to port 80 (HTTP) from the Internet.

        azure network nsg rule create -a NSG-FrontEnd -n web-rule -c Allow -p Tcp -r Inbound -y 200 -f Internet -o * -e * -u 80

    Expected putput:

        info:    Executing command network nsg rule create
     info:    Looking up the network security group "NSG-FrontEnd"
     info:    Creating a network security rule "web-rule"
     info:    Looking up the network security group "NSG-FrontEnd"
     data:    Name                            : web-rule
     data:    Source address prefix           : INTERNET
     data:    Source Port                     : *
     data:    Destination address prefix      : *
     data:    Destination Port                : 80
     data:    Protocol                        : TCP
     data:    Type                            : Inbound
     data:    Action                          : Allow
     data:    Priority                        : 200
     info:    network nsg rule create command OK
6. Run the **azure network nsg subnet add** command to link the NSG to the front end subnet.

        azure network nsg subnet add -a NSG-FrontEnd --vnet-name TestVNet --subnet-name FrontEnd 

    Expected output:

        info:    Executing command network nsg subnet add
     info:    Looking up the network security group "NSG-FrontEnd"
     info:    Looking up the subnet "FrontEnd"
     info:    Looking up network configuration
     info:    Creating a network security group "NSG-FrontEnd"
     info:    network nsg subnet add command OK


## How to create the NSG for the back end subnet
To create an NSG named named *NSG-BackEnd* based on the scenario above, follow the steps below.

1. Run the **azure network nsg create** command to create an NSG.

        azure network nsg create -l uswest -n NSG-BackEnd

    Expected output:

        info:    Executing command network nsg create
     info:    Creating a network security group "NSG-BackEnd"
     info:    Looking up the network security group "NSG-BackEnd"
     data:    Name                            : NSG-BackEnd
     data:    Location                        : West US
     data:    Security group rules:
     data:    Name                               Source IP           Source Port  Destination IP   Destination Port  Protocol  Type      Action  Prior
     ity  Default
     data:    ---------------------------------  ------------------  -----------  ---------------  ----------------  --------  --------  ------  -----
     ---  -------
     data:    ALLOW VNET OUTBOUND                VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Outbound  Allow   65000
          true   
     data:    ALLOW VNET INBOUND                 VIRTUAL_NETWORK     *            VIRTUAL_NETWORK  *                 *         Inbound   Allow   65000
          true   
     data:    ALLOW AZURE LOAD BALANCER INBOUND  AZURE_LOADBALANCER  *            *                *                 *         Inbound   Allow   65001
          true   
     data:    ALLOW INTERNET OUTBOUND            *                   *            INTERNET         *                 *         Outbound  Allow   65001
          true   
     data:    DENY ALL OUTBOUND                  *                   *            *                *                 *         Outbound  Deny    65500
          true   
     data:    DENY ALL INBOUND                   *                   *            *                *                 *         Inbound   Deny    65500
          true   
     info:    network nsg create command OK

    Parameters:

   * **-l (or --location)**. Azure region where the new NSG will be created. For our scenario, *westus*.
* **-n (or --name)**. Name for the new NSG. For our scenario, *NSG-FrontEnd*.

2. Run the **azure network nsg rule create** command to create a rule that allows access to port 1433 (SQL) from the front end subnet.

        azure network nsg rule create -a NSG-BackEnd -n sql-rule -c Allow -p Tcp -r Inbound -y 100 -f 192.168.1.0/24 -o * -e * -u 1433

    Expected output:

        info:    Executing command network nsg rule create
     info:    Looking up the network security group "NSG-BackEnd"
     info:    Creating a network security rule "sql-rule"
     info:    Looking up the network security group "NSG-BackEnd"
     data:    Name                            : sql-rule
     data:    Source address prefix           : 192.168.1.0/24
     data:    Source Port                     : *
     data:    Destination address prefix      : *
     data:    Destination Port                : 1433
     data:    Protocol                        : TCP
     data:    Type                            : Inbound
     data:    Action                          : Allow
     data:    Priority                        : 100
     info:    network nsg rule create command OK



1. Run the **azure network nsg rule create** command to create a rule that denies access to the Internet.

        azure network nsg rule create -a NSG-BackEnd -n web-rule -c Deny -p Tcp -r Outbound -y 200 -f * -o * -e Internet -u 80

    Expected putput:

        info:    Executing command network nsg rule create
     info:    Looking up the network security group "NSG-BackEnd"
     info:    Creating a network security rule "web-rule"
     info:    Looking up the network security group "NSG-BackEnd"
     data:    Name                            : web-rule
     data:    Source address prefix           : *
     data:    Source Port                     : *
     data:    Destination address prefix      : INTERNET
     data:    Destination Port                : 80
     data:    Protocol                        : TCP
     data:    Type                            : Outbound
     data:    Action                          : Deny
     data:    Priority                        : 200
     info:    network nsg rule create command OK
2. Run the **azure network nsg subnet add** command to link the NSG to the back end subnet.

        azure network nsg subnet add -a NSG-BackEnd --vnet-name TestVNet --subnet-name BackEnd 

    Expected output:

        info:    Executing command network nsg subnet add
     info:    Looking up the network security group "NSG-BackEndX"
     info:    Looking up the subnet "BackEnd"
     info:    Looking up network configuration
     info:    Creating a network security group "NSG-BackEndX"
     info:    network nsg subnet add command OK

