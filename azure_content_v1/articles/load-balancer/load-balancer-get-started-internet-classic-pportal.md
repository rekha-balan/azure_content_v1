<properties 
   pageTitle="Get started creating an Internet facing load balancer in classic deployment model using the Azure portal | Microsoft Azure"
   description="Learn how to create an Internet facing load balancer in classic deployment model using the Azure portal"
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
   ms.date="12/07/2015"
   ms.author="joaoma" />

# Get started creating an Internet facing load balancer (classic) in the Azure portal
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




## Get started creating a load balancer endpoint using Azure portal
To create an Internet facing load balancer (classic) deployment model from the Azure portal, follow the steps below.

1. From a browser, navigate to http://portal.azure.com and, if necessary, sign in with your Azure account.

2. Go to virtual machines (classic) blade > select a virtual machine.

3. In the virtual machines "essentials" blade >  select  "all settings"

4. Click in "load balanced sets".

5. To create a new load balancer, click  "join" icon on the top of the load balanced sets blade.

6. Select the "load balanced set type" public for Internet facing load balancer. 

7. Click in "configure required settings" to open "choose a load balanced set" and click on "create a load balanced set".

8. In "create a load balanced set" blade, create a name for the load balancer set. Fill out the name, public port, probe protocol, probe port.

9. Change probe interval and retries if needed.

10. (optional) if you want, you can configure access control rules from load balancer set creation blade.

11. Click ok to go back to "join load balanced set" blade.

12. click ok and wait for new load balancer resource to show in the "load balancer sets" blade.


## Next steps
[Get started configuring an internal load balancer](load-balancer-internal-getstarted.md)

[Configure a load balancer distribution mode](load-balancer-distribution-mode.md)

[Configure idle TCP timeout settings for your load balancer](load-balancer-tcp-idle-timeout.md)

