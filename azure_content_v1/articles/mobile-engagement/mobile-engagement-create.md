<properties
    pageTitle="Create an Azure Mobile Engagement App | Microsoft Azure"
    description="Describes how to create a new Mobile Engagement App Collection in Azure and start managing your apps with the Mobile Engagement portal."
    services="mobile-engagement"
    documentationCenter=""
    authors="piyushjo"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="mobile-engagement"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows-store"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="11/22/2015"  
    ms.author="juliako"/>


# Create an Azure Mobile Engagement App
This article shows how to use the **Quick Create** method to create a new **Azure Mobile Engagement** App. The article also shows how to navigate to your **Mobile Engagement** portal in order to start monitoring and managing your apps. 

Note that you must add a minimum set of "basic integration" in order to be able to collect data for your app and send push notifications. The complete integration documentation can be found in the [Mobile Engagement integration](mobile-engagement-windows-store-integrate-engagement.md).

> [!IMPORTANT]
> To complete any Azure Mobile Engagement tutorial, you must have an active Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure Free Trial</a>.
> 
> 
## Setup Mobile Engagement for your mobile app in Azure
1. Log on to the [Azure Classic Portal](https://manage.windowsazure.com), and then click **+NEW** at the bottom of the screen.

2. Click on **App Services**, then **Mobile Engagement**, and then **Create**.

   	![](./media/mobile-engagement-create-app-in-portal/create-mobile-engagement-app.png)

3. In the popup that appears, enter the following information:

   	![](./media/mobile-engagement-create-app-in-portal/create-azme-popup.png)

	- **Application Name**: Name of your application. 
	- **Platform**: Target platform for the app. You must create one Mobile Engagement app per platform that you are targeting for your mobile application. 
	- **Application Resource Name**: Name by which this application will be accessible via APIs and URLs. 
	- **Location**: Region/Data center where this app and app collection will be hosted.
	- **Collection**: Select a previously created Collection or select 'New Collection'.
	- **Collection Name**: Represents your group of applications. This will also ensure all your apps are in a group that will allow aggregated calculations of metrics. You should use your company name or department here if applicable.

4. Select the app you just created in the **Applications** tab.

5. Click on **CONNECTION INFO** in order to display the connection settings to put into your SDK integration in your mobile app.

6. Copy the **CONNECTION STRING** - this is what you will need to identify this app in your Application code and connect with Mobile Engagement from your App.

   	![](./media/mobile-engagement-create-app-in-portal/app-connection-info-page.png)



## Navigate to your Mobile Engagement portal
To start monitoring and managing your application, navigate to your Mobile Engagement portal by clicking the **Engage** button at the bottom.

![](../../includes/media/mobile-engagement-connect-app-with-monitor/engage-button.png)

Once you are in the  Mobile Engagement portal, you can analiyze, create and manage segments, reach out to the users, etc.:    

* [Monitor real time data about your application](mobile-engagement-user-interface-monitor.md)
* [Analyze historical data about your application](mobile-engagement-user-interface-analytics.md)
* [Create and manage segments of users to identify usage patterns](mobile-engagement-user-interface-segments.md)
* [Reach out to the users of your application with push notifications](mobile-engagement-user-interface-reach.md)

## See Also
[Define your Mobile Engagement strategy](mobile-engagement-define-your-mobile-engagement-strategy.md)

[Get started with Azure Mobile Engagement](mobile-engagement-windows-store-dotnet-get-started.md) (you can select other mobile platforms at the top of the page).

