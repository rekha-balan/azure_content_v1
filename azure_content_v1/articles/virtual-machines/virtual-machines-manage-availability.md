<properties
    pageTitle="Manage the availability of VMs | Microsoft Azure"
    description="Learn how to use multiple virtual machines to ensure high availability for your Azure application."
    services="virtual-machines"
    documentationCenter=""
    authors="kenazk"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager,azure-service-management"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="07/23/2015"
    ms.author="kenazk"/>

# Manage the availability of virtual machines
> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md). This article covers using both models, but Microsoft recommends that most new deployments use the Resource Manager model.

## Understand planned vs. unplanned maintenance
There are two types of Microsoft Azure platform events that can affect the availability of your virtual machines: planned maintenance and unplanned maintenance.

* **Planned maintenance events** are periodic updates made by Microsoft to the underlying Azure platform to improve overall reliability, performance, and security of the platform infrastructure that your virtual machines run on. The majority of these updates are performed without any impact upon your virtual machines or cloud services. However, there are instances where these updates require a reboot of your virtual machine to apply the required updates to the platform infrastructure.

* **Unplanned maintenance events** occur when the hardware or physical infrastructure underlying your virtual machine has faulted in some way. This may include local network failures, local disk failures, or other rack level failures. When such a failure is detected, the Azure platform will automatically migrate your virtual machine from the unhealthy physical machine hosting your virtual machine to a healthy physical machine. Such events are rare, but may also cause your virtual machine to reboot.


## Follow best practices when you design your application for high availability
To reduce the impact of downtime due to one or more of these events, we recommend the following high availability best practices for your virtual machines:

* [Configure multiple virtual machines in an Availability Set for redundancy](#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy.md)
* [Configure each application tier into separate Availability Sets](#configure-each-application-tier-into-separate-availability-sets.md)
* [Combine the Load Balancer with Availability Sets](#combine-the-load-balancer-with-availability-sets.md)
* [Avoid single instance virtual machines in Availability Sets](#avoid-single-instance-virtual-machines-in-availability-sets.md)

### Configure multiple virtual machines in an Availability Set for redundancy
To provide redundancy to your application, we recommend that you group two or more virtual machines in an Availability Set. This configuration ensures that during either a planned or unplanned maintenance event, at least one virtual machine will be available and meet the 99.95% Azure SLA. For more information about service level agreements, see the “Cloud Services, virtual machines, and Virtual Network” section in [Service Level Agreements](../../../support/legal/sla/.md).

Each virtual machine in your Availability Set is assigned an Update Domain (UD) and a Fault Domain (FD) by the underlying Azure platform. For a given Availability Set, five non-user-configurable UDs are assigned to indicate groups of virtual machines and underlying physical hardware that can be rebooted at the same time. When more than five virtual machines are configured within a single Availability Set, the sixth virtual machine will be placed into the same UD as the first virtual machine, the seventh in the same UD as the second virtual machine, and so on. The order of UDs being rebooted may not proceed sequentially during planned maintenance, but only one UD will be rebooted at a time.

FDs define the group of virtual machines that share a common power source and network switch. By default, the virtual machines configured within your Availability Set are separated across two FDs. While placing your virtual machines into an Availability Set does not protect your application from operating system or application-specific failures, it does limit the impact of potential physical hardware failures, network outages, or power interruptions.

<!--Image reference-->
   ![UD FD configuration](./media/virtual-machines-manage-availability/ud-fd-configuration.png)

> [!NOTE]
> For instructions, see [How to Configure an Availability Set for virtual machines](virtual-machines-how-to-configure-availability.md).
> 
> 
### Configure each application tier into separate Availability Sets
If the virtual machines in your Availability Set are all nearly identical and serve the same purpose for your application, we recommend that you configure an Availability Set for each tier of your application.  If you place two different tiers in the same Availability Set, all virtual machines in the same application tier can be rebooted at once. By configuring at least two virtual machines in an Availability Set for each tier, you guarantee that at least one virtual machine in each tier will be available.

For example, you could put all the virtual machines in the front-end of your application running IIS, Apache, Nginx, etc., in a single Availability Set. Make sure that only front-end virtual machines are placed in the same Availability Set. Similarly, make sure that only data-tier virtual machines are placed in their own Availability Set, like your replicated SQL Server virtual machines or your MySQL virtual machines.

<!--Image reference-->
   ![Application tiers](./media/virtual-machines-manage-availability/application-tiers.png)

### Combine the Load Balancer with Availability Sets
Combine the Azure Load Balancer with an Availability Set to get the most application resiliency. The Azure Load Balancer distributes traffic between multiple virtual machines. For our Standard tier virtual machines, the Azure Load Balancer is included. Note that not all virtual machine tiers include the Azure Load Balancer. For more information about load balancing your virtual machines, read [Load Balancing virtual machines](../load-balance-virtual-machines.md).

If the load balancer is not configured to balance traffic across multiple virtual machines, then any planned maintenance event will affect the only traffic-serving virtual machine, causing an outage to your application tier. Placing multiple virtual machines of the same tier under the same load balancer and Availability Set enables traffic to be continuously served by at least one instance.

### Avoid single instance virtual machines in Availability Sets
Avoid leaving a single instance virtual machine in an Availability Set by itself. Virtual machines in this configuration do not qualify for a SLA guarantee and will face downtime during Azure planned maintenance events. Please note, single virtual machine instance within an Availability Set, will also receive advanced email notification in multi-instance virtual machines planned maintenance notification. 

<!-- Link references -->

[Configure multiple virtual machines in an Availability Set for redundancy]: #configure-multiple-virtual-machines-in-an-availability-set-for-redundancy
[Configure each application tier into separate Availability Sets]: #configure-each-application-tier-into-separate-availability-sets
[Combine the Load Balancer with Availability Sets]: #combine-the-load-balancer-with-availability-sets
[Avoid single instance virtual machines in Availability Sets]: #avoid-single-instance-virtual-machines-in-availability-sets
[How to Configure An Availability Set for virtual machines]: virtual-machines-how-to-configure-availability.md
