<properties 
    pageTitle="Azure App Service Environment" 
    description="Learn how App Service work" 
    keywords="app service environment, azure app service environment"
    services="app-service" 
    documentationCenter="" 
    authors="yochay" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="app-service" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="12/08/2015" 
    ms.author="yochay"/>

# Overview
An App Service Environment is a [Premium](http://azure.microsoft.com/pricing/details/app-service/) service plan option of Azure App Service that provides a fully isolated and dedicated environment for securely running Azure App Service apps at high scale, including [Web Apps](http://azure.microsoft.com/documentation/articles/app-service-web-overview/), [Mobile Apps](http://azure.microsoft.com/documentation/articles/app-service-mobile-value-prop-preview/), and [API Apps](http://azure.microsoft.com/documentation/articles/app-service-api-apps-why-best-platform/).  

App Service Environments are ideal for application workloads requiring:

* Very high scale
* Isolation and secure network access

Customers can create multiple App Service Environments within a single Azure region, as well as across multiple Azure regions.  This makes App Service Environments ideal for horizontally scaling state-less application tiers in support of high RPS workloads.

App Service Environments are isolated to running only a single customer's applications, and are always deployed into a virtual network.  Customers have fine-grained control over both inbound and outbound application network traffic using [network security groups](https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/).  Applications can also establish high-speed secure connections over virtual networks to on-premises corporate resources.

Apps frequently need to access corporate resources such as internal databases and web services.  Apps running on App Service Environments can access resources reachable via [Site-to-Site](https://azure.microsoft.com/documentation/articles/vpn-gateway-site-to-site-create/) VPN and [Azure ExpressRoute](http://azure.microsoft.com/services/expressroute/) connections.

* [What is an App Service Environment?](../app-service-web/app-service-app-service-environment-intro.md)
  * [Setting Up a Geo-Distributed App Footprint](../app-service-web/app-service-app-service-environment-geo-distributed-scale.md)
  * [Implementing a Layered Security Architecture](../app-service-web/app-service-app-service-environment-layered-security.md)
  * [Configuring a Web Application Firewall with an App Service Environment](../app-service-web/app-service-app-service-environment-web-application-firewall.md)
* [Creating an App Service Environment](../app-service-web/app-service-web-how-to-create-an-app-service-environment.md)
* [Creating Apps in an App Service Environment](../app-service-web/app-service-web-how-to-create-a-web-app-in-an-ase.md)
* [Configuring an App Service Environment](../app-service-web/app-service-web-configure-an-app-service-environment.md) 
* [Scaling Apps in an App Service Environment](../app-service-web/app-service-web-scale-a-web-app-in-an-app-service-environment.md)
  * [Using Auto-Scale with an App Service Environment](app-service-environment-auto-scale.md)
* [Network Security and Architecture](../app-service-web/app-service-app-service-environment-network-architecture-overview.md)
  * [Securing Inbound Traffic](../app-service-web/app-service-app-service-environment-control-inbound-traffic.md)
  * [Connecting to Backend Resources](../app-service-web/app-service-app-service-environment-securely-connecting-to-backend-resources.md)
  * [ExpressRoute and App Service Environments](../app-service-web/app-service-app-service-environment-network-configuration-expressroute.md)

## Videos

* [AzureCon 2015:  Deploying Highly Secure and Scalable Apps](/documentation/videos/azurecon-2015-deploying-highly-scalable-and-secure-web-and-mobile-apps/)

<!-- LINKS -->

[PremiumTier]: http://azure.microsoft.com/pricing/details/app-service/
[WebApps]: http://azure.microsoft.com/documentation/articles/app-service-web-overview/
[MobileApps]: http://azure.microsoft.com/documentation/articles/app-service-mobile-value-prop-preview/
[APIApps]: http://azure.microsoft.com/documentation/articles/app-service-api-apps-why-best-platform/
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[SiteToSite]: https://azure.microsoft.com/documentation/articles/vpn-gateway-site-to-site-create/
[ExpressRoute]: http://azure.microsoft.com/services/expressroute/
