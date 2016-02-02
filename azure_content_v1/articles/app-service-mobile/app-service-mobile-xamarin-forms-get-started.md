<properties
    pageTitle="Get Started with Mobile Apps by using Xamarin.Forms"
    description="Follow this tutorial to get started using Azure Mobile Apps for Xamarin.Forms development"
    services="app-service\mobile"
    documentationCenter="xamarin"
    authors="wesmc7777"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-xamarin"
    ms.devlang="dotnet"
    ms.topic="get-started-article"
    ms.date="11/23/2015"
    ms.author="normesta"/>

# Create a Xamarin.Forms app
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
This tutorial shows you how to add a cloud-based backend service to a Xamarin.Forms mobile app using an Azure Mobile App backend.  You will create both a new Mobile App backend and a simple *Todo list* Xamarin.Forms app that stores app data in Azure.

Completing this tutorial is a prerequisite for all other Mobile Apps tutorials for Xamarin.Forms.

## Prerequisites
To complete this tutorial, you need the following:

* An active Azure account. If you don't have an account, you can sign up for an Azure trial and get up to 10 free Mobile Apps that you can keep using even after your trial ends. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/).

* [Visual Studio Community 2013]Visual Studio Community 2013] or later.  If you install Visual Studio Community 2013, install [Xamarin](http://xamarin.com/download) separately.  You can install the Xamarin tools when you install Visual Studio 2015.

* A Mac with [Xcode](https://go.microsoft.com/fwLink/?LinkID=266532clcid=0x409) v7.0 or later and [Xamarin Studio](http://xamarin.com/download) installed. If you plan to build your app on a Windows computer by using Visual Studio, you'll still need access to a networked Mac to do it.


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
Follow the steps below to configure the server project to use either the Node.js or .NET backend.


1. Back in the Mobile App backend settings, click **Get started** > your client platform. 

2. Under **Create a table API**, select your **Backend language**, either **C#** or **Node.js**:

	+ **Node.js backend** (via portal):  
	Accept the acknowledgment and click **Create TodoItem table**. This creates a new *TodoItem* table in your database.
	 
		>[AZURE.IMPORTANT] Switching an existing app backend to Node.js will overwrite all site contents.

	+ **.NET backend (C#)**:  
	Click **Download**, extract the compressed project files to your local computer, open the solution in Visual Studio, build the project to restore the NuGet packages, then deploy the project to Azure. To learn how to deploy a .NET backend server project to Azure, see [How to: Publish the server project](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#publish-server-project) in the .NET backend SDK topic. 
	 
You Mobile App backend is now ready to use with your client app.


## (Optional) Test your backend project locally
If you chose a .NET backend configuration above, you can optionally test the backend locally.



The mobile app project lets you to run your new mobile app backend locally. This makes it easy to debug your service code before you even publish it to Azure.

1. On your Windows PC, extract project you downloaded earlier, and then open it in Visual Studio.

2. Select the bottom project which should be your Mobile App name with Service at the end of it. Press **CTRL-F5** to download the nuget packages, build the project and start the mobile app backend locally. When you run your mobile app client, pointed at localhost, it will talk to your local backend. 


## Download and run the Xamarin.Forms solution
Here you have a couple of choices. You can download the solution to a Mac and open it in Xamarin Studio, or you can download the solution to a Windows computer and open it in Visual Studio. You can also use both environments and switch back and forth between them. Consider these things:

* It's easier to run the iOS project of your solution on a Mac. You can use Visual Studio on your Windows computer if you want, but it's a bit more complicated because you have to connect to a networked Mac. If you're interested in doing that, see [Installing Xamarin.iOS on Windows](http://developer.xamarin.com/guides/ios/getting_started/installation/windows/).
* You can run the Android project on either your Mac or your Windows computer.
* You can run the Windows project(s) only by using Visual Studio on a Windows computer.

With all of this in mind, let's proceed.

1. On your Mac or on your Windows computer, open the [Azure Portal](https://portal.azure.com/) in a browser window.
2. On the settings blade for your Mobile App, click **Get Started** (under Mobile) > **Xamarin.Forms**. Under step 3, click  **Create a new app** if it's not already selected.  Next click the **Download** button.

   This downloads a project that contains a client application that is connected to your mobile app. Save the compressed project file to your local computer, and make a note of where you save it.

3. Extract the project that you downloaded, and then open it in Xamarin Studio or Visual Studio.

   ![][9]

   ![][8]


## (Optional) Run the iOS project
This section is for running the Xamarin iOS project for iOS devices. You can skip this section if you are not working with iOS devices.

#### In Xamarin Studio
1. Right-click the iOS project, and then click **Set As Startup Project**.
2. On the **Run** menu, click **Start Debugging** to build the project and start the app in the iPhone emulator.

#### In Visual Studio
1. Right-click the iOS project, and then click **Set as StartUp Project**.
2. On the **Build** menu, click **Configuration Manager**. 
3. In the **Configuration Manager** dialog box, select the **Build** and **Deploy** checkboxes of the iOS project.
4. Press the **F5** key to build the project and start the app in the iPhone emulator.

In the app, type meaningful text, such as *Learn Xamarin* and then click the **+** button.

![][10]

This sends a POST request to the new mobile app backend hosted in Azure. Data from the request is inserted into the TodoItem table. Items stored in the table are returned by the mobile app backend, and the data is displayed in the list.

> [!NOTE]
> You'll find the code that accesses your mobile app backend in the TodoItemManager.cs C# file of the portable class library project of your solution.
> 
> 
## (Optional) Run the Android project
This section is for running the Xamarin droid project for Android. You can skip this section if you are not working with Android devices.

#### In Xamarin Studio
1. Right-click the Android project, and then click **Set As Startup Project**.
2. On the **Run** menu, click **Start Debugging** to build the project and start the app in an Android emulator.

#### In Visual Studio
1. Right-click the Android project, and then click **Set as StartUp Project**.
2. On the **Build** menu, click **Configuration Manager**. 
3. In the **Configuration Manager** dialog box, select the **Build** and **Deploy** checkboxes of the Android project.
4. Press the **F5** key to build the project and start the app in an Android emulator.

In the app, type meaningful text, such as *Learn Xamarin* and then click the **+** button.

![][11]

This sends a POST request to the new mobile app backend hosted in Azure. Data from the request is inserted into the TodoItem table. Items stored in the table are returned by the mobile app backend, and the data is displayed in the list.

> [!NOTE]
> You'll find the code that accesses your mobile app backend in the TodoItemManager.cs C# file of the portable class library project of your solution.
> 
> 
## (Optional) Run the Windows project
This section is for running the Xamarin WinApp project for Windows devices. You can skip this section if you are not working with Windows devices.

#### In Visual Studio
1. Right-click any of the Windows projects, and then click **Set as StartUp Project**.
2. On the **Build** menu, click **Configuration Manager**. 
3. In the **Configuration Manager** dialog box, select the **Build** and **Deploy** checkboxes of the Windows project that you chose.
4. Press the **F5** key to build the project and start the app in a Windows emulator.

In the app, type meaningful text, such as *Learn Xamarin* and then click the **+** button.

This sends a POST request to the new mobile app backend hosted in Azure. Data from the request is inserted into the TodoItem table. Items stored in the table are returned by the mobile app backend, and the data is displayed in the list.

![][12]

> [!NOTE]
> You'll find the code that accesses your mobile app backend in the TodoItemManager.cs C# file of the portable class library project of your solution.
> 
> 
<!-- Anchors. -->

[Getting started with mobile app backends]:#getting-started
[Create a new mobile app backend]:#create-new-service
[Next Steps]:#next-steps


<!-- Images. -->

[6]: ./media/app-service-mobile-xamarin-forms-get-started/xamarin-forms-quickstart.png
[8]: ./media/app-service-mobile-xamarin-forms-get-started/xamarin-forms-quickstart-vs.png
[9]: ./media/app-service-mobile-xamarin-forms-get-started/xamarin-forms-quickstart-xs.png
[10]: ./media/app-service-mobile-xamarin-forms-get-started/mobile-quickstart-startup-ios.png
[11]: ./media/app-service-mobile-xamarin-forms-get-started/mobile-quickstart-startup-android.png
[12]: ./media/app-service-mobile-xamarin-forms-get-started/mobile-quickstart-startup-windows.png


<!-- URLs. -->

[Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[Mobile app SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[Azure Portal]: https://portal.azure.com/


[Xamarin Studio]: http://xamarin.com/download
[Xamarin]: http://xamarin.com/download
[Xcode]: https://go.microsoft.com/fwLink/?LinkID=266532&clcid=0x409
[Xamarin for Windows]: https://go.microsoft.com/fwLink/?LinkID=330242&clcid=0x409
[Installing Xamarin.iOS on Windows]: http://developer.xamarin.com/guides/ios/getting_started/installation/windows/
