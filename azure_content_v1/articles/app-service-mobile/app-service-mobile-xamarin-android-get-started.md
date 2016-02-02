<properties
    pageTitle="Get Started with Azure Mobile Apps for Xamarin.Android apps"
    description="Follow this tutorial to get started using Azure Mobile Apps for Xamarin Android development"
    services="app-service\mobile"
    documentationCenter="xamarin"
    authors="ggailey777"
    manager="dwrede"
    editor="" />

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-xamarin-android"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="11/17/2015"
    ms.author="glenga" />

# Create a Xamarin.Android App
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
This tutorial shows you how to add a cloud-based backend service to a Xamarin.Android app. For more information, see [What are Mobile Apps](app-service-mobile-value-prop.md). 

A screenshot from the completed app is below:

![][0]

Completing this tutorial is a prerequisite for all other Mobile Apps tutorials for Xamarin.Android apps.

## Prerequisites
To complete this tutorial, you need the following:

* An active Azure account. If you don't have an account, you can sign up for an Azure trial and get up to 10 free Mobile Apps that you can keep using even after your trial ends. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/).

* [Visual Studio Community 2013](https://go.microsoft.com/fwLink/p/?LinkID=534203) or later.  If you install Visual Studio Community 2013, install [Xamarin](http://xamarin.com/download) separately.  You can install the Xamarin tools when you install Visual Studio 2015.


> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](https://tryappservice.azure.com/?appServiceName=mobile), where you can immediately create a short-lived starter Mobile App in App Service. No credit cards required; no commitments.
> 
> 
## Create a new Azure Mobile App backend
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

## Configure the server project

1. Back in the Mobile App backend settings, click **Get started** > your client platform. 

2. Under **Create a table API**, select your **Backend language**, either **C#** or **Node.js**:

	+ **Node.js backend** (via portal):  
	Accept the acknowledgment and click **Create TodoItem table**. This creates a new *TodoItem* table in your database.
	 
		>[AZURE.IMPORTANT] Switching an existing app backend to Node.js will overwrite all site contents.

	+ **.NET backend (C#)**:  
	Click **Download**, extract the compressed project files to your local computer, open the solution in Visual Studio, build the project to restore the NuGet packages, then deploy the project to Azure. To learn how to deploy a .NET backend server project to Azure, see [How to: Publish the server project](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#publish-server-project) in the .NET backend SDK topic. 
	 
You Mobile App backend is now ready to use with your client app.


## Download and run the Xamarin.Android app
1. Under **Download and run your Xamarin.Android project**, click the **Download** button.

      This downloads a project that contains a client application that is connected to your mobile app. Save the compressed project file to your local computer, and make a note of where you save it.

2. Press the **F5** key to build the project and start the app. 

3. In the app, type meaningful text, such as *Complete the tutorial* and then click the **Add** button.

    ![][10]

    This sends a POST request to the new mobile app backend hosted in Azure. Data from the request is inserted into the TodoItem table. Items stored in the table are returned by the mobile app backend, and the data appears in the list.

   > [!NOTE]
> You can review the code that accesses your mobile app backend to query and insert data, which is found in the ToDoActivity.cs C# file.
> 
> 

## Next steps
* [Add authentication to your app ](app-service-mobile-xamarin-android-get-started-users.md)
<br/>Learn how to authenticate users of your app with an identity provider.

<!-- Images. -->

[0]: ./media/app-service-mobile-xamarin-android-get-started/mobile-quickstart-completed-android.png
[6]: ./media/app-service-mobile-xamarin-android-get-started/mobile-portal-quickstart-xamarin.png
[8]: ./media/app-service-mobile-xamarin-android-get-started/mobile-xamarin-project-android-vs.png
[9]: ./media/app-service-mobile-xamarin-android-get-started/mobile-xamarin-project-android-xs.png
[10]: ./media/app-service-mobile-xamarin-android-get-started/mobile-quickstart-startup-android.png

<!-- URLs. -->

[Azure Portal]: https://azure.portal.com/
[Xamarin]: http://xamarin.com/download
[Xcode]: https://go.microsoft.com/fwLink/?LinkID=266532&clcid=0x409
[Xamarin for Windows]: https://go.microsoft.com/fwLink/?LinkID=330242&clcid=0x409

[Visual Studio Community 2013]: https://go.microsoft.com/fwLink/p/?LinkID=534203
