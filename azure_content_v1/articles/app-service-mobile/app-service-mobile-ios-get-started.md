<properties
    pageTitle="Create an iOS app on Azure App Service Mobile Apps | Microsoft Azure"
    description="Follow this tutorial to get started with using Azure mobile app backends for iOS development in Objective-C or Swift"
    services="app-service\mobile"
    documentationCenter="ios"
    authors="krisragh"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="na"
    ms.tgt_pltfrm="mobile-ios"
    ms.devlang="objective-c"
    ms.topic="hero-article"
    ms.date="11/30/2015"
    ms.author="krisragh"/>

# Create an iOS app
> [AZURE.SELECTOR]
- [Android](../articles/app-service-mobile/app-service-mobile-android-get-started.md)
- [iOS](../articles/app-service-mobile/app-service-mobile-ios-get-started.md)
- [Windows](../articles/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started.md)
- [Xamarin.Android](../articles/app-service-mobile/app-service-mobile-xamarin-android-get-started.md)
- [Xamarin.iOS](../articles/app-service-mobile/app-service-mobile-xamarin-ios-get-started.md)
- [Xamarin.Forms](../articles/app-service-mobile/app-service-mobile-xamarin-forms-get-started.md)


&nbsp;  
>[AZURE.NOTE] This is an **Azure Mobile Apps** topic. For Mobile Services topics, see the [Mobile Services documentation center](/documentation/services/mobile-services/).
>
>App Service Mobile Apps is our newest mobile backend platform and [gives you additional advantages](app-service-mobile-value-prop-migration-from-mobile-services.md) over Mobile Services. [Migrating to App Service](app-service-mobile-migrating-from-mobile-services.md) is  recommended for customers using the .NET backend SDK. However, [Mobile Apps Node SDK](https://github.com/azure/azure-mobile-apps-node) is currently in Preview and is not yet recommended for production use. SDK and API contracts are subject to change within minor version releases.


## Overview
This tutorial shows you how to use a cloud-based backend service ([Azure Mobile Apps](app-service-mobile-value-prop.md)) to an iOS mobile app.  You will create  a new mobile backend and use a simple *Todo list* iOS app that stores app data in Azure.

## Prerequisites
To complete this tutorial, you need the following:

* An [active Azure account](https://azure.microsoft.com/pricing/free-trial/)

* A PC with [Visual Studio Community 2013](https://go.microsoft.com/fwLink/p/?LinkID=534203) or newer

* A Mac with Xcode v7.0 or newer

* [Azure Mobile iOS framework](https://go.microsoft.com/fwLink/?LinkID=529823), which is automatically included as part of the quickstart project you download


## Create a new Azure mobile app backend
Follow these steps to create a new Mobile App backend.

1. Log into the [Azure Portal].

2. In the top left of the window, click the **+NEW** button > **Web + Mobile** > **Mobile App**, then provide a name for your Mobile App backend.

3. In the **Resource Group** box, select an existing resource group. If you have no resource groups, enter the same name as your app. 
 
	At this point, the default App Service plan is selected, which is in the Free tier. The App Service plan settings determine the location, features, cost and compute resources associated with your app. You can either select another App Service plan or create a new one. For more about App Services plans and how to create a new plan, see [Azure App Service plans in-depth overview](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)

4. Use the default App Service plan, select a different plan or [create a new plan](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md#create-an-app-service-plan), then click **Create**. 
	
	This creates the Mobile App backend. Later you will deploy your server project to this backend. Provisioning a Mobile App backend can take several minutes; the **Settings** blade for the Mobile App backend is displayed when complete. Before you can use the Mobile App backend, you must also define a connection a data store.

    > [AZURE.NOTE] As part of this tutorial, you create a new SQL Database instance and server. You can reuse this new database and administer it as you would any other SQL Database instance. If you already have a database in the same location as the new mobile app backend, you can instead choose **Use an existing database** and then select that database. The use of a database in a different location is not recommended because of additional bandwidth costs and higher latencies. Other data storage options are available. 

6. In the **Settings** blade for the new Mobile App backend, click **Quick start** > your client app platform > **Connect a database**. 

	![](./media/app-service-mobile-dotnet-backend-create-new-service/dotnet-backend-create-data-connection.png)

7. In the **Add data connection** blade, click **SQL Database** > **Create a new database**, type the database **Name**, choose a pricing tier, then click **Server**.  
 
    ![](./media/app-service-mobile-dotnet-backend-create-new-service/dotnet-backend-create-db.png)

8. In the **New server** blade, type a unique server name in the **Server name** field, provide a secure **Server admin login** and **Password**, make sure that **Allow azure services to access server** is checked, then click **OK** twice. This creates the new database and server.

10. Back in the **Add data connection** blade, click **Connection string**, type the login and password values for your database, then click **OK** twice.

	Creation of the database can take a few minutes.  Use the **Notifications** area to monitor the progress of the deployment.  You cannot continue until the database has been deployed sucessfully.


<!-- URLs. -->
[Azure Portal]: https://portal.azure.com/


You have now provisioned an Azure Mobile App backend that can be used by your mobile client applications. Next, you will download a server project for a simple "todo list" backend and publish it to Azure.

## Download the server project
1. On your PC, visit the [Azure portal](https://portal.azure.com/). Click **Browse All** > **Mobile Apps**, and then click the mobile app backend that you just created.

2. In the Mobile App blade, click **Settings**, and then under **Mobile App**, click **Quickstart** > **iOS (Objective-C)**. If you prefer Swift, click **Quickstart** > **iOS (Swift)** instead.


## Configure the server project

1. Back in the Mobile App backend settings, click **Get started** > your client platform. 

2. Under **Create a table API**, select your **Backend language**, either **C#** or **Node.js**:

	+ **Node.js backend** (via portal):  
	Accept the acknowledgment and click **Create TodoItem table**. This creates a new *TodoItem* table in your database.
	 
		>[AZURE.IMPORTANT] Switching an existing app backend to Node.js will overwrite all site contents.

	+ **.NET backend (C#)**:  
	Click **Download**, extract the compressed project files to your local computer, open the solution in Visual Studio, build the project to restore the NuGet packages, then deploy the project to Azure. To learn how to deploy a .NET backend server project to Azure, see [How to: Publish the server project](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#publish-server-project) in the .NET backend SDK topic. 
	 
You Mobile App backend is now ready to use with your client app.


## Download and run the iOS app

1. On your Mac, visit the [Azure Portal]. Click **Browse All** > **Mobile Apps** > the backend that you just created. In the mobile app settings, click **Quickstart** > **iOS (Objective-C)**. If you prefer Swift, click **Quickstart** > **iOS (Swift)** instead. Under **Download and run your iOS project**, click **Download**. This downloads a complete Xcode project for an app pre-configured to connect to your backend. Open the project using Xcode.

2. Press the **Run** button to build the project and start the app in the iOS simulator.

3. In the app, type meaningful text, such as _Complete the tutorial_ and then click the plus (**+**) icon. This sends a POST request to the Azure backend you deployed earlier. The backend inserts data from the request is into the TodoItem SQL table, and returns information about the newly stored items back to the mobile app. The mobile app displays this data in the list. 

   	![](./media/mobile-services-ios-run-app/mobile-quickstart-startup-ios.png)

[Azure Portal]: https://portal.azure.com/


<!-- Images. -->

<!-- URLs -->

[Azure portal]: https://portal.azure.com/
[Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Visual Studio Community 2013]: https://go.microsoft.com/fwLink/p/?LinkID=534203
