<properties 
    pageTitle="Azure App Service: Scaling App Service Applications" 
    description="Learn the ins and outs of scaling Application in App Service." 
    keywords="app service, azure app service, scale, scalable, app service plan, app service cost"
    services="app-service" 
    documentationCenter="" 
    authors="btardif" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="app-service" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="12/08/2015" 
    ms.author="byvinyal"/>

# Azure App Service: Scaling App Service Applications
Applications hosted in Azure App Service can [achieve massive scale](https://azure.microsoft.com/blog/canadian-broadcasting-corporation-radio-canada-leverage-azure-for-smooth-election-coverage/)
however scaling an application is a complex problem that does not have a "one 
size fits all" solution. To correctly scale your application there are 3 key 
areas that will contribute to your applications success:

1. Understanding your application architecture and its weaknesses.   * Is your Application Statefull? Stateless?
* What are all the components of your application?  * Where are the bottle necks in the application? 


* When load is applied to your app, what will break first?


2. Understanding the expected load and performance requirements.   * Does the application need to serve one thousand users? or one million?
* Will traffic come from a single geographic location or globally?
* Are there seasonal variations? traffic peaks? 
* How fast should the app respond? 1 second? 1 millisecond?


3. Understanding and correctly leverage the platform hosting your app.   * What features should I leverage to achieve my scale goals?



This section will help you understand all the factors and help you device a 
strategy that takes advantage of the necessary App Service features to achieve 
your scalability goals.

* [Scale Azure App Service](../app-service/app-service-scale.md)
	* Understanding Redundancy and Reliability
	* StateFULL vs StateLESS applications
		* [Configuring your Stateless app](/blog/disabling-arrs-instance-affinity-in-windows-azure-web-sites/)

* Load Testing apps in App Service   

* App Service scaling best practices
	* Port exhaustion and connection pooling
	* [Leveraging Azure Storage from your App Service app](../storage/storage-dotnet-how-to-use-blobs.md)
	* [Leveraging Azure CDN from your App Service app](../cdn/cdn-overview.md)
	* [Leveraging Azure Redis Cache from your App Service app](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md)
		
* Geo distribution
	* [Leveraging Traffic Manager to achive global scale](../traffic-manager/traffic-manager-overview.md)

