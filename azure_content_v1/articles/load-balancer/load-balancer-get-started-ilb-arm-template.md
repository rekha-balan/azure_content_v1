<properties 
   pageTitle="Create an internal load balancer using a template in Resource Manager | Microsoft Azure"
   description="Learn how to create an internal load balancer using a template in Resource Manager"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"
/>

<tags  
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/09/2015"
   ms.author="joaoma" />

# Get stated creating an internal load balancer using a template
> [AZURE.SELECTOR]
[PowerShell](load-balancer-get-started-ilb-arm-ps.md)
[Azure CLI](load-balancer-get-started-ilb-arm-cli.md)
[Template](load-balancer-get-started-ilb-arm-template.md)


<BR>

Azure Internal Load Balancing (ILB) provides network load balancing between virtual machines that reside inside a cloud service, a virtual network with a regional scope. An internal load balancer acts as a security boundary not allowing direct access to the virtual machines behind this type of load balancer from Internet.

For information about the use and configuration of virtual networks with a regional scope, see [Regional Virtual Networks](virtual-networks-migrate-to-regional-vnet.md). Existing virtual networks that have been configured for an affinity group cannot use ILB.





> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model, which Microsoft recommends for most new deployments instead of the [classic deployment model](load-balancer-get-started-ilb-classic-ps.md).

## Scenario

You are creating an internal load balancer according to the following scenario

![IMAGE DESCRIPTION](./media/load-balancer-get-started-ilb-scenario-include/figure1.png)

An internal load balancer is configured in a virtual network  
- 2 virtual machines called DB1 and DB2<BR>
- Endpoints <BR>
- Internal load balancer<BR>


## Deploy the template by using click to deploy
The sample template available in the public repository uses a parameter file containing the default values used to generate the scenario described above. To deploy this template using click to deploy, follow [this link]((https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer.md), click **Deploy to Azure**, replace the default parameter values if necessary, and follow the instructions in the portal.

## Deploy the template by using PowerShell
To deploy the template you downloaded by using PowerShell, follow the steps below.

1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.
2. Run the **Switch-AzureMode** cmdlet to switch to Resource Manager mode, as shown below.

        Switch-AzureMode AzureResourceManager

    Here is the expected output for the command above:

        WARNING: The Switch-AzureMode cmdlet is deprecated and will be removed in a future release.


> [!WARNING]
> The Switch-AzureMode cmdlet will be deprecated soon. When that happens, all Resource Manager cmdlets will be renamed.
> 
> 
3.Download the [parameters](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.parameters.json) file to your local disk.<BR>

1. Edit the file and save it.<BR>
2. Run the **New-AzureResourceGroup** cmdlet to create a resource group using the template. 

        New-AzureResourceGroup -Name TestRG -Location westus `
            -TemplateFile 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json' `
            -TemplateParameterFile 'C:\temp\azuredeploy.parameters.json'



## Deploy the template by using the Azure CLI
To deploy the template by using the Azure CLI, follow the steps below.

1. If you have never used Azure CLI, see [Install and Configure the Azure CLI](xplat-cli.md) and follow the instructions up to the point where you select your Azure account and subscription.
2. Run the **azure config mode** command to switch to Resource Manager mode, as shown below.

        azure config mode arm

    Here is the expected output for the command above:

        info:    New mode is arm
3. Open the [parameter file](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.parameters.json), select its contents, and save it to a file in your computer. For this example, we saved the parameters file to *parameters.json*.

4. Run the **azure group deployment create** cmdlet to deploy the new VNet by using the template and parameter files you downloaded and modified above. The list shown after the output explains the parameters used.

        azure group create -n TestRG -l westus --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json -e parameters.json




## Next steps
[Configure a load balancer distribution mode using source IP affinity](load-balancer-distribution-mode.md)

[Configure idle TCP timeout settings for your load balancer](load-balancer-tcp-idle-timeout.md)

