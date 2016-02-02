<properties 
   pageTitle="How to create NSGs in ARM mode using a template| Microsoft Azure"
   description="Learn how to create and deploy NSGs in ARM using a template"
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

# How to create NSGs using a template
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
 

## NSG resources in a template file
You can view and download the [sample template](https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/).

The section below shows the definition of the front end NSG, based on the scenario above.

      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "[parameters('frontEndNSGName')]",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayName": "NSG - Front End"
      },
      "properties": {
        "securityRules": [
          {
            "name": "rdp-rule",
            "properties": {
              "description": "Allow RDP",
              "protocol": "Tcp",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "sourceAddressPrefix": "Internet",
              "destinationAddressPrefix": "*",
              "access": "Allow",
              "priority": 100,
              "direction": "Inbound"
            }
          },
          {
            "name": "web-rule",
            "properties": {
              "description": "Allow WEB",
              "protocol": "Tcp",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "sourceAddressPrefix": "Internet",
              "destinationAddressPrefix": "*",
              "access": "Allow",
              "priority": 101,
              "direction": "Inbound"
            }
          }
        ]
      }

To associate the NSG to the front end subnet, you have to change the subnet definition in the template, and use the reference id for the NSG.

        "subnets": [
          {
            "name": "[parameters('frontEndSubnetName')]",
            "properties": {
              "addressPrefix": "[parameters('frontEndSubnetPrefix')]",
              "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('frontEndNSGName'))]"
              }
            }
          }, ...

Notice the same being done for the back end NSG and the back end subnet in the template.

## Deploy the ARM template by using click to deploy
The sample template available in the public repository uses a parameter file containing the default values used to generate the scenario described above. To deploy this template using click to deploy, follow [this link](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd-NSG), click **Deploy to Azure**, replace the default parameter values if necessary, and follow the instructions in the portal.

## Deploy the ARM template by using PowerShell
To deploy the ARM template you downloaded by using PowerShell, follow the steps below.

> [AZURE.NOTE] Azure PowerShell is currently available in two releases - 1.0 and 0.9.8. If you have existing scripts and do not want to change them right now, you can continue using the 0.9.8 release. When using the 1.0 release, you should carefully test your scripts in pre-production environments before using them in production to avoid unexpected impacts.
>
> 1.0 cmdlets follow the naming pattern {verb}-AzureRm{noun}; whereas, the 0.9.8 names do not include **Rm** (for example, New-AzureRmResourceGroup instead of New-AzureResourceGroup). When using Azure PowerShell 0.9.8, you must first enable the Resource Manager mode by running the **Switch-AzureMode AzureResourceManager** command. This command is not necessary in 1.0.
>
> New features will be added to only the 1.0 release. For information about the 1.0 release, including how to install and uninstall the release, see [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).


1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

2. Run the **New-AzureRmResourceGroup** cmdlet to create a resource group using the template.

        New-AzureRmResourceGroup -Name TestRG -Location uswest `
         -TemplateFile 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.json' `
         -TemplateParameterFile 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.parameters.json'    

    Expected output:

        ResourceGroupName : TestRG
     Location          : westus
     ProvisioningState : Succeeded
     Tags              : 
     Permissions       : 
                         Actions  NotActions
                         =======  ==========
                         *                  

     Resources         : 
                         Name                Type                                     Location
                         ==================  =======================================  ========
                         sqlAvSet            Microsoft.Compute/availabilitySets       westus  
                         webAvSet            Microsoft.Compute/availabilitySets       westus  
                         SQL1                Microsoft.Compute/virtualMachines        westus  
                         SQL2                Microsoft.Compute/virtualMachines        westus  
                         Web1                Microsoft.Compute/virtualMachines        westus  
                         Web2                Microsoft.Compute/virtualMachines        westus  
                         TestNICSQL1         Microsoft.Network/networkInterfaces      westus  
                         TestNICSQL2         Microsoft.Network/networkInterfaces      westus  
                         TestNICWeb1         Microsoft.Network/networkInterfaces      westus  
                         TestNICWeb2         Microsoft.Network/networkInterfaces      westus  
                         NSG-BackEnd         Microsoft.Network/networkSecurityGroups  westus  
                         NSG-FrontEnd        Microsoft.Network/networkSecurityGroups  westus  
                         TestPIPSQL1         Microsoft.Network/publicIPAddresses      westus  
                         TestPIPSQL2         Microsoft.Network/publicIPAddresses      westus  
                         TestPIPWeb1         Microsoft.Network/publicIPAddresses      westus  
                         TestPIPWeb2         Microsoft.Network/publicIPAddresses      westus  
                         TestVNet            Microsoft.Network/virtualNetworks        westus  
                         testvnetstorageprm  Microsoft.Storage/storageAccounts        westus  
                         testvnetstoragestd  Microsoft.Storage/storageAccounts        westus  

     ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG


## Deploy the ARM template by using the Azure CLI
To deploy the ARM template by using the Azure CLI, follow the steps below.

1. If you have never used Azure CLI, see [Install and Configure the Azure CLI](xplat-cli-install.md) and follow the instructions up to the point where you select your Azure account and subscription.
2. Run the **azure config mode** command to switch to Resource Manager mode, as shown below.

        azure config mode arm

    Here is the expected output for the command above:

        info:    New mode is arm
3. Run the **azure group deployment create** cmdlet to deploy the new VNet by using the template and parameter files you downloaded and modified above. The list shown after the output explains the parameters used.

        azure group create -n TestRG -l westus -f 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.json' -e 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.parameters.json'

    Expected output:

        info:    Executing command group create
     info:    Getting resource group TestRG
     info:    Creating resource group TestRG
     info:    Created resource group TestRG
     info:    Initializing template configurations and parameters
     info:    Creating a deployment
     info:    Created template deployment "azuredeploy"
     data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG
     data:    Name:                TestRG
     data:    Location:            westus
     data:    Provisioning State:  Succeeded
     data:    Tags: null
     data:    
     info:    group create command OK

   * **-n (or --name)**. Name of the resource group to be created.
* **-l (or --location)**. Azure region where the resource group will be created.
* **-f (or --template-file)**. Path to your ARM template file.
* **-e (or --parameters-file)**. Path to your ARM parameters file.


