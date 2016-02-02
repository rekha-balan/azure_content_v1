
<properties 
   pageTitle="Get started creating an Internet facing load balancer in classic deployment model using the Azure classic portal | Microsoft Azure"
   description="Learn how to create an Internet facing load balancer in classic deployment model using the Azure classic portal"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
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

# Get started creating an Internet facing load balancer (classic) in the Azure classic portal
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




## Set up an Internet-facing load balancer for virtual machines
In order to load balance network traffic from the Internet across the virtual machines of a cloud service, you must create a load-balanced set. This procedure assumes that you have already created the virtual machines and that they are all within the same cloud service.

**To configure a load-balanced set for virtual machines**

1. In the Azure classic portal, click **Virtual Machines**, and then click the name of a virtual machine in the load-balanced set.
2. Click **Endpoints**, and then click **Add**.

3. On the **Add an endpoint to a virtual machine** page, click the right arrow.

4. On the **Specify the details of the endpoint** page:

   * In **Name**, type a name for the endpoint or select the name from the list of predefined endpoints for common protocols.
* In **Protocol**, select the protocol required by the type of endpoint, either TCP or UDP, as needed.
* In **Public Port and Private Port**, type the port numbers that you want the virtual machine to use, as needed. You can use the private port and firewall rules on the virtual machine to redirect traffic in a way that is appropriate for your application. The private port can be the same as the public port. For example, for an endpoint for web (HTTP) traffic, you could assign port 80 to both the public and private port.

5. Select **Create a load-balanced set**, and then click the right arrow.

6. On the **Configure the load-balanced set** page, type a name for the load-balanced set, and then assign the values for probe behavior of the Azure Load Balancer.
The Load Balancer uses probes to determine if the virtual machines in the load-balanced set are available to receive incoming traffic.

7. Click the check mark to create the load-balanced endpoint. You will see **Yes** in the **Load-balanced set name** column of the **Endpoints** page for the virtual machine.

8. In the portal, click **Virtual Machines**, click the name of an additional virtual machine in the load-balanced set, click **Endpoints**, and then click **Add**.

9. On the **Add an endpoint to a virtual machine** page, click **Add endpoint to an existing load-balanced set**, select the name of the load-balanced set, and then click the right arrow.

10. On the **Specify the details of the endpoint** page, type a name for the endpoint, and then click the check mark.
For the additional virtual machines in the load-balanced set, repeat steps 8-10.


## Next steps
[Get started configuring an internal load balancer](load-balancer-internal-getstarted.md)

[Configure a load balancer distribution mode](load-balancer-distribution-mode.md)

[Configure idle TCP timeout settings for your load balancer](load-balancer-tcp-idle-timeout.md)

