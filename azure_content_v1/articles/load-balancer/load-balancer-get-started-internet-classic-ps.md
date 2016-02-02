<properties 
   pageTitle="Get started creating an Internet facing load balancer in classic mode using PowerShell | Microsoft Azure"
   description="Learn how to create an Internet facing load balancer in classic mode using PowerShell"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carolz"
   editor=""
   tags="azure-service-management"
/>

<tags  
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="01/21/2016"
   ms.author="joaoma" />

# Get started creating an Internet facing load balancer (classic) in PowerShell
>[AZURE.SELECTOR]
- [Azure Portal](load-balancer-get-started-internet-classic-pportal.md)
- [Azure classic Portal](load-balancer-get-started-internet-classic-portal.md)
- [PowerShell](load-balancer-get-started-internet-classic-ps.md)
- [Azure CLI](load-balancer-get-started-internet-classic-cli.md)
- [Azure Cloud Services](load-balancer-get-started-internet-classic-cloud.md)


You can use a load balancer to provide high availability for your workloads in Azure. An Azure load balancer is a Layer-4 (TCP, UDP) type load balancer that distributes incoming traffic among healthy service instances in cloud services or virtual machines defined in a load balancer set.
 
You can configure a load balancer to.

- Load balance incoming Internet traffic to virtual machines (VMs). We refer to a load balancer in this scenario as an [Internet facing load balancer](load-balancer-internet-overview.md).
- Load balance traffic between VMs in a virtual network (VNet), VMs in cloud services, or between on-premises computers and VMs in a cross-premises virtual network. We refer to a load balancer in this scenario as an [internal load balancer (ILB)](load-balancer-internal-overview.md).
- 	Forward external traffic to a specific VM instance.


>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This article covers the classic deployment model. You can also [Learn how to create an Internet facing load balancer using Azure Resource Manager](load-balancer-get-started-internet-arm-ps.md).

The following tasks will be done in this scenario:

- Create a load balancer receiving network traffic on port 80 and send load balanced traffic to virtual machines "web1" and "web2"
- Create NAT rules for remote desktop access for virtual machines behind the load balancer
- Create health probes

![Load balancer scenario](./media/load-balancer-get-started-internet-scenario-include/scenario-classic.png)




## Set up load balancer using PowerShell
To set up a load balancer using powershell, follow the steps below:

1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

1. After creating a virtual machine, you can use PowerShell cmdlets to add a load balancer to a virtual machine within the same cloud service.

In the following example you will add a load balancer set called "webfarm" to cloud service "mytestcloud" (or myctestcloud.cloudapp.net) , adding the endpoints for the load balancer to virtual machines named "web1" and "web2". The load balancer receives network traffic on port 80 and load balances between the virtual machines defined by the local endpoint (in this case port 80) using TCP.

### Step 1
Create a load balanced endpoint for the first VM "web1"

    Get-AzureVM -ServiceName "mytestcloud" -Name "web1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 80 -LBSetName "WebFarm" -ProbePort 80 -ProbeProtocol "http" -ProbePath '/' | Update-AzureVM

### Step 2
Create another endpoint for the second VM  "web2" using the same load balancer set name

    Get-AzureVM -ServiceName "mytestcloud" -Name "web2" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 80 -LBSetName "WebFarm" -ProbePort 80 -ProbeProtocol "http" -ProbePath '/' | Update-AzureVM

## Remove a virtual machine from a load balancer
You can use Remove-AzureEndpoint to remove a virtual machine endpoint from the load balancer 

    Get-azureVM -ServiceName mytestcloud  -Name web1 |Remove-AzureEndpoint -Name httpin| Update-AzureVM

## Next steps
You can also [get started creating an internal load balancer](load-balancer-get-started-ilb-classic-ps.md) and configure what type of [distribution mode](load-balancer-distribution-mode.md) for an especific load balancer network traffic behavior.

If your application needs to keep connections alive for servers behind a load balancer, you can understand more about [idle TCP timeout settings for a load balancer](load-balancer-tcp-idle-timeout.md). It will help to learn about idle connection behavior when you are using Azure Load Balancer. 

