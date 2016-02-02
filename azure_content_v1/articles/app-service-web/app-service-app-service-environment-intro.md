<properties 
    pageTitle="Introduction to App Service Environment" 
    description="Learn about the App Service Environment feature that provides secure, VNet-joined, dedicated scale units for running all of your apps." 
    services="app-service" 
    documentationCenter="" 
    authors="ccompy" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="app-service" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="01/05/2016"
    ms.author="stefsch"/>

# Introduction to App Service Environment
## Overview
An App Service Environment is a [Premium][PremiumTier]Premium][PremiumTier]PremiumTier] service plan option of Azure App Service that provides a fully isolated and dedicated environment for securely running Azure App Service apps at high scale, including [Web Apps][WebApps]Web Apps][WebApps]WebApps], [Mobile Apps][MobileApps]Mobile Apps][MobileApps]MobileApps], and [API Apps][APIApps]API Apps][APIApps]APIApps].  

App Service Environments are ideal for application workloads requiring:

* Very high scale
* Isolation and secure network access

Customers can create multiple App Service Environments within a single Azure region, as well as across multiple Azure regions.  This makes App Service Environments ideal for horizontally scaling state-less application tiers in support of high RPS workloads.

App Service Environments are isolated to running only a single customer's applications, and are always deployed into a virtual network.  Customers have fine-grained control over both inbound and outbound application network traffic, and applications can establish high-speed secure connections over virtual networks to on-premises corporate resources.

For an overview of how App Service Environments enable high scale and secure network access, see the [AzureCon Deep Dive][AzureConDeepDive]AzureCon Deep Dive][AzureConDeepDive]AzureConDeepDive] on App Service Environments!

For a deep-dive on horizontally scaling using multiple App Service Environments see the article on how to setup a [geo-distributed app footprint][GeodistributedAppFootprint]geo-distributed app footprint][GeodistributedAppFootprint]GeodistributedAppFootprint].

To see how the security architecture shown in the AzureCon Deep Dive was configured, see the article on implementing a [layered security architecture](app-service-app-service-environment-layered-security.md) with App Service Environments.

Apps running on App Service Environments can have their access gated by upstream devices such as web application firewalls (WAF).  The article on [configuring a WAF for App Service Environments](app-service-app-service-environment-web-application-firewall.md) covers this scenario. 

> [AZURE.NOTE] Although this article refers to web apps, it also applies to API apps and mobile apps.


## Dedicated Compute Resources
All of the compute resources in an App Service Environment are dedicated exclusively to a single subscription, and an App Service Environment can be configured with up to fifty (50) compute resources for exclusive use by a single application.

An App Service Environment is composed of a front-end compute resource pool, as well as one to three worker compute resource pools. 

The front-end pool contains compute resources responsible for SSL termination as well automatic load balancing of app requests within an App Service Environment. 

Each worker pool contains compute resources allocated to [App Service Plans][AppServicePlan]App Service Plans][AppServicePlan]AppServicePlan], which in turn contain one or more Azure App Service apps.  Since there can be up to three different worker pools in an App Service Environment, you have the flexibility to choose different compute resources for each worker pool.  

For example this allows you to create one worker pool with less powerful compute resources for App Service Plans intended for development or test apps.  A second (or even third) worker pool could use more powerful compute resources intended for App Service Plans running production apps.

For more details on the quantity of compute resources available to the front-end and worker pools, see [How To Configure an App Service Environment][HowToConfigureanAppServiceEnvironment]How To Configure an App Service Environment][HowToConfigureanAppServiceEnvironment]HowToConfigureanAppServiceEnvironment].  

For details on the available compute resource sizes supported in an App Service Environment, consult the [App Service Pricing][AppServicePricing]App Service Pricing][AppServicePricing]AppServicePricing] page and review the available options for App Service Environments in the Premium pricing tier.

## Virtual Network Support
An App Service Environment can either be created in a pre-existing regional classic "v1" virtual network, or a new regional classic "v1" virtual network ([more info on virtual networks][MoreInfoOnVirtualNetworks]more info on virtual networks][MoreInfoOnVirtualNetworks]MoreInfoOnVirtualNetworks]).  Since an App Service Environment always exists in a regional virtual network, and more precisely within a subnet of a regional virtual network, you can leverage the security features of virtual networks to control both inbound and outbound network communications.  

You can use [network security groups][NetworkSecurityGroups]network security groups][NetworkSecurityGroups]NetworkSecurityGroups] to restrict inbound network communications to the subnet where an App Service Environment resides.  This allows you to run apps behind upstream devices and services such as web application firewalls, and network SaaS providers.  

Apps also frequently need to access corporate resources such as internal databases and web services.  A common approach is to make these endpoints available only to internal network traffic flowing within an Azure virtual network.  Once an App Service Environment is joined to the same virtual network as the internal services, apps running in the environment can access them, including endpoints reachable via [Site-to-Site][SiteToSite]Site-to-Site][SiteToSite]SiteToSite] and [Azure ExpressRoute][ExpressRoute]Azure ExpressRoute][ExpressRoute]ExpressRoute] connections.

For more details on how App Service Environments work with virtual networks and on-premises networks consult the following articles on [Network Architecture][NetworkArchitectureOverview]Network Architecture][NetworkArchitectureOverview]NetworkArchitectureOverview], [Controlling Inbound Traffic][ControllingInboundTraffic]Controlling Inbound Traffic][ControllingInboundTraffic]ControllingInboundTraffic], and [Securely Connecting to Backends][SecurelyConnectingToBackends]Securely Connecting to Backends][SecurelyConnectingToBackends]SecurelyConnectingToBackends]. 

**Note:**  An App Service Environment cannot be created in a "v2" virtual network.

## Getting started
To get started with App Service Environments, see [How To Create An App Service Environment][HowToCreateAnAppServiceEnvironment]How To Create An App Service Environment][HowToCreateAnAppServiceEnvironment]HowToCreateAnAppServiceEnvironment]

For more information about the Azure App Service platform, see [Azure App Service][AzureAppService]Azure App Service][AzureAppService]AzureAppService].

For an overview of the App Service Environment network architecture, see the [Network Architecture Overview][NetworkArchitectureOverview]Network Architecture Overview][NetworkArchitectureOverview]NetworkArchitectureOverview] article.

For details on using an App Service Environment with ExpressRoute, see the following article on [Express Route and App Service Environments][NetworkConfigDetailsForExpressRoute]Express Route and App Service Environments][NetworkConfigDetailsForExpressRoute]NetworkConfigDetailsForExpressRoute].

## What's changed
* For a guide to the change from Websites to App Service see: [Azure App Service and Its Impact on Existing Azure Services](http://go.microsoft.com/fwlink/?LinkId=529714)


>[AZURE.NOTE] If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](http://go.microsoft.com/fwlink/?LinkId=523751), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.


<!-- LINKS -->
[PremiumTier]PremiumTier]: http://azure.microsoft.com/pricing/details/app-service/
[MoreInfoOnVirtualNetworks]MoreInfoOnVirtualNetworks]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[AppServicePlan]AppServicePlan]: http://azure.microsoft.com/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/
[HowToCreateAnAppServiceEnvironment]HowToCreateAnAppServiceEnvironment]: http://azure.microsoft.com/documentation/articles/app-service-web-how-to-create-an-app-service-environment/
[AzureAppService]AzureAppService]: http://azure.microsoft.com/documentation/articles/app-service-value-prop-what-is/
[WebApps]WebApps]: http://azure.microsoft.com/documentation/articles/app-service-web-overview/
[MobileApps]MobileApps]: http://azure.microsoft.com/documentation/articles/app-service-mobile-value-prop-preview/
[APIApps]APIApps]: http://azure.microsoft.com/documentation/articles/app-service-api-apps-why-best-platform/
[LogicApps]LogicApps]: http://azure.microsoft.com/documentation/articles/app-service-logic-what-are-logic-apps/
[AzureConDeepDive]AzureConDeepDive]:  https://azure.microsoft.com/documentation/videos/azurecon-2015-deploying-highly-scalable-and-secure-web-and-mobile-apps/
[GeodistributedAppFootprint]GeodistributedAppFootprint]:  https://azure.microsoft.com/documentation/articles/app-service-app-service-environment-geo-distributed-scale/
[NetworkSecurityGroups]NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[SiteToSite]SiteToSite]: https://azure.microsoft.com/documentation/articles/vpn-gateway-site-to-site-create/
[ExpressRoute]ExpressRoute]: http://azure.microsoft.com/services/expressroute/
[HowToConfigureanAppServiceEnvironment]HowToConfigureanAppServiceEnvironment]:  http://azure.microsoft.com/documentation/articles/app-service-web-configure-an-app-service-environment/
[ControllingInboundTraffic]ControllingInboundTraffic]:  https://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/
[SecurelyConnectingToBackends]SecurelyConnectingToBackends]:  https://azure.microsoft.com/documentation/articles/app-service-app-service-environment-securely-connecting-to-backend-resources/
[NetworkArchitectureOverview]NetworkArchitectureOverview]:  https://azure.microsoft.com/documentation/articles/app-service-app-service-environment-network-architecture-overview/
[NetworkConfigDetailsForExpressRoute]NetworkConfigDetailsForExpressRoute]:  https://azure.microsoft.com/documentation/articles/app-service-app-service-environment-network-configuration-expressroute/
[AppServicePricing]AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/ 

<!-- IMAGES -->


