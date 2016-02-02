<properties 
   pageTitle="Create an Internet facing load balancer in Resource Manager using a template | Microsoft Azure"
   description="Learn how to create an Internet facing load balancer in Resource Manager using an ARM template"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>

<tags  
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/18/2015"
   ms.author="joaoma" />

# Get started creating an Internet facing load balancer using an ARM template
> [AZURE.SELECTOR]
- [PowerShell](load-balancer-get-started-internet-arm-ps.md)
- [Azure CLI](load-balancer-get-started-internet-arm-cli.md)
- [Template](load-balancer-get-started-internet-arm-template.md)


You can use a load balancer to provide high availability for your workloads in Azure. An Azure load balancer is a Layer-4 (TCP, UDP) type load balancer that distributes incoming traffic among healthy service instances in cloud services or virtual machines defined in a load balancer set.
 
You can configure a load balancer to.

- Load balance incoming Internet traffic to virtual machines (VMs). We refer to a load balancer in this scenario as an [Internet facing load balancer](load-balancer-internet-overview.md).
- Load balance traffic between VMs in a virtual network (VNet), VMs in cloud services, or between on-premises computers and VMs in a cross-premises virtual network. We refer to a load balancer in this scenario as an [internal load balancer (ILB)](load-balancer-internal-overview.md).
- 	Forward external traffic to a specific VM instance.


>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the Resource Manager deployment model. You can also [Learn how to create an Internet facing load balancer using classic deployment model](load-balancer-get-started-internet-classic-portal.md)

The following tasks will be done in this scenario:

- Create a load balancer receiving network traffic on port 80 and send load balanced traffic to virtual machines "web1" and "web2"
- Create NAT rules for remote desktop access for virtual machines behind the load balancer
- Create health probes

![Load balancer scenario](./media/load-balancer-get-started-internet-scenario-include/scenario-classic.png)




## Deploy the ARM template by using click to deploy
The sample template available in the public repository uses a parameter file containing the default values used to generate the scenario described above. To deploy this template using click to deploy, follow [this link](http://go.microsoft.com/fwlink/?LinkId=544801), click **Deploy to Azure**, replace the default parameter values if necessary, and follow the instructions in the portal.

## Deploy the ARM template by using PowerShell
To deploy the ARM template you downloaded by using PowerShell, follow the steps below.

1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

2. Run the **New-AzureRmResourceGroup** cmdlet to create a resource group using the template.

        New-AzureRmResourceGroup -Name TestRG -Location uswest `
         -TemplateFile 'https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json' `
         -TemplateParameterFile 'https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.parameters.json'    


## Deploy the ARM template by using the Azure CLI
To deploy the ARM template by using the Azure CLI, follow the steps below.

1. If you have never used Azure CLI, see [Install and Configure the Azure CLI](xplat-cli.md) and follow the instructions up to the point where you select your Azure account and subscription.
2. Run the **azure config mode** command to switch to Resource Manager mode, as shown below.

        azure config mode arm

    Here is the expected output for the command above:

        info:    New mode is arm
3. From your browser, navigate to **https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.parameters.json**, copy the contents of the json file and paste into a new file in your computer. For this scenario, you would be copying the values below to a file named **c:\lb\azuredeploy.parameters.json**.

4. Run the **azure group deployment create** cmdlet to deploy the new load balancer by using the template and parameter files you downloaded and modified above. The list shown after the output explains the parameters used.

        azure group create -n TestRG -l westus -f 'https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json' -e 'c:\lb\azuredeploy.parameters.json'


## Next steps
[Get started configuring an internal load balancer](load-balancer-internal-getstarted.md)

[Configure a load balancer distribution mode](load-balancer-distribution-mode.md)

[Configure idle TCP timeout settings for your load balancer](load-balancer-tcp-idle-timeout.md)

