<properties
 pageTitle="IoT Hub HA and DR | Microsoft Azure"
 description="Describes features that help to build highly available IoT solutions with disaster recovery capabilities."
 services="iot-hub"
 documentationCenter=""
 authors="fsautomata"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="01/20/2016"
 ms.author="elioda"/>

# IoT Hub high availability and disaster recovery
As an Azure service, IoT Hub provides high availability (HA) using redundancies at the Azure region level, without any additional work required by the solution. In addition, Azure offers a number of features that help to build solutions with disaster recovery (DR) capabilities or cross-region availability if required. You must design and prepare your solutions to take advantage of these DR features if you want to provide global, cross-region high availability for devices or users. The article [Azure Business Continuity Technical Guidance](https://msdn.microsoft.com/library/azure/hh873027.aspx) describes the built-in features in Azure for business continuity and DR. The [Disaster Recovery and High Availability for Azure Applications](https://msdn.microsoft.com/library/azure/dn251004.aspx) paper provides architecture guidance on strategies for Azure applications to achieve HA and DR.

## Regional failover with IoT Hub
A complete treatment of deployment topologies in IoT solutions is outside the scope of this article, but for the purpose of high availability and disaster recovery we will consider the *regional failover* deployment model.

In a regional failover model, the solution backend is running primarily in one datacenter location, but an additional IoT hub and backend are deployed in another datacenter location for failover purposes, in case the IoT hub in the primary datacenter suffers an outage or the network connectivity from the device to the primary datacenter is somehow interrupted. Devices use a secondary service endpoint whenever the primary gateway cannot be reached. With a cross-region failover capability, the solution availability can be improved beyond the high availability of a single region.

At a high level, to implement a regional failover model with IoT Hub, you will need the following.

* **A secondary IoT hub and device routing logic**: In the case of a service disruption in your primary region, devices must start connecting to your secondary region. Given the state-aware nature of most services involved, it is common for solution administrators to trigger the inter-region failover process. The best way to communicate the new endpoint to devices, while maintaining control of the process, is have them regularly check a *concierge* service for the current active endpoint. The concierge service can be a simple web application that is replicated and kept reachable using DNS-redirection techniques (for example, using [Azure Traffic Manager](https://azure.microsoft.com/documentation/services/traffic-manager/)).
* **Identity registry replication** - In order to be usable, the secondary IoT hub must contain all device identities that can connect to the solution. The solution should keep geo-replicated backups of device identities, and upload them to the secondary IoT hub before switching the active endpoint for the devices. The device identity export functionality of IoT Hub is very useful in this context. For more information, see [IoT Hub Developer Guide - identity registry](iot-hub-devguide.md#identityregistry).
* **Merging logic** - When the primary region becomes available again, all the state and data that have been created in the secondary site must be migrated back to the primary region. This mostly relates to device identities and application meta-data, which must be merged with the primary IoT hub and any other application-specific stores in the primary region. To simplify this step, it is usually recommended that you use idempotent operations. This minimizes side-effects not only from eventual consistent distribution of events, but also from duplicates or out-of-order delivery of events. In addition, the application logic should be designed to tolerate potential inconsistencies or "slightly" out of date-state. This is due to the additional time it takes for the system to "heal" based on recovery point objectives (RPO). The following article provides more guidance on this topic: [Failsafe: Guidance for Resilient Cloud Architectures](https://msdn.microsoft.com/library/azure/jj853352.aspx).

## Next steps
Follow these links to learn more about Azure IoT Hub:

* [Get started with IoT Hubs (Tutorial)](iot-hub-csharp-csharp-getstarted.md)
* [What is Azure IoT Hub?](iot-hub-what-is-iot-hub.md)

[Azure Business Continuity Technical Guidance]: https://msdn.microsoft.com/library/azure/hh873027.aspx
[Disaster Recovery and High Availability for Azure Applications]: https://msdn.microsoft.com/library/azure/dn251004.aspx
[Failsafe: Guidance for Resilient Cloud Architectures]: https://msdn.microsoft.com/library/azure/jj853352.aspx
[Azure Traffic Manager]: https://azure.microsoft.com/documentation/services/traffic-manager/
[IoT Hub Developer Guide - identity registry]: iot-hub-devguide.md#identityregistry

[lnk-get-started]: iot-hub-csharp-csharp-getstarted.md
[What is Azure IoT Hub?]: iot-hub-what-is-iot-hub.md
