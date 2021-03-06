<properties
    pageTitle="Get Started with Mobile Services for Windows Universal apps | Microsoft Azure"
    description="Follow this tutorial to get started using Azure Mobile Services for universal Windows app development in C#."
    services="mobile-services"
    documentationCenter="windows"
    authors="ggailey777"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="mobile-services"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="11/06/2015"
    ms.author="glenga"/>


# <a name="getting-started"> </a>Get started with Mobile Services
>[AZURE.NOTE] This is an **Azure Mobile Services** topic.  Microsoft Azure recommends Azure App Service Mobile Apps for all new mobile backend deployments.
To get started with Azure App Service Mobile Apps, see the [App Service Mobile Apps documentation center](/documentation/services/app-service/mobile).


&nbsp;

> [AZURE.SELECTOR-LIST (Platform | Backend )]
- [(iOS | .NET)](../articles/mobile-services-dotnet-backend-ios-get-started.md)
- [(iOS | JavaScript)](../articles/mobile-services-ios-get-started.md)
- [(Windows Runtime 8.1 universal C# | .NET)](../articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started.md)
- [(Windows Runtime 8.1 universal C# | Javascript)](../articles/mobile-services-javascript-backend-windows-store-dotnet-get-started.md)
- [(Windows Runtime 8.1 universal JavaScript | Javascript)](../articles/mobile-services-javascript-backend-windows-store-javascript-get-started.md)
- [(Android | .NET)](../articles/mobile-services-dotnet-backend-android-get-started.md)
- [(Android | Javascript)](../articles/mobile-services-android-get-started.md)
- [(Xamarin.iOS | .NET)](../articles/mobile-services-dotnet-backend-xamarin-ios-get-started.md)
- [(Xamarin.iOS | Javascript)](../articles/partner-xamarin-mobile-services-ios-get-started.md)
- [(Xamarin.Android | .NET)](../articles/mobile-services-dotnet-backend-xamarin-android-get-started.md)
- [(Xamarin.Android | Javascript)](../articles/partner-xamarin-mobile-services-android-get-started.md)
- [(HTML | Javascript)](../articles/mobile-services-html-get-started.md)
- [(PhoneGap | Javascript)](../articles/mobile-services-javascript-backend-phonegap-get-started.md)
- [(Sencha | Javascript)](../articles/partner-sencha-mobile-services-get-started.md)


&nbsp;

> [!TIP]
> If you are new to mobile development using Microsoft Azure, [get started with Azure Mobile Apps](app-service-mobile-dotnet-backend-windows-store-dotnet-get-started-preview.md) instead of Azure Mobile Services; Mobile Apps gives you [additional advantages](app-service-mobile-value-prop-migration-from-mobile-services-preview.md).
> 
> 
This tutorial shows you how to add a cloud-based backend service to a universal Windows app using Azure Mobile Services. Universal Windows app solutions include projects for both Windows Store 8.1 and Windows Phone Store 8.1 apps and a common shared project. For more information, see [Build universal Windows apps that target Windows and Windows Phone](http://msdn.microsoft.com/library/windows/apps/xaml/dn609832.aspx).

In this tutorial, you will create both a new mobile service and a simple *To do list* app that stores app data in the new mobile service. The mobile service that you will create uses the supported .NET languages using Visual Studio for server-side business logic and to manage the mobile service. To create a mobile service that lets you write your server-side business logic in JavaScript, see the JavaScript backend version of this topic.

> [!NOTE]
> This topic shows you how to create a new mobile service project and universal Windows app by using the Azure classic portal. By using Visual Studio 2013 Update 3, you can also add a new mobile service project to an existing Visual Studio solution. For more information, see [Add Mobile Services to an existing app](mobile-services-dotnet-backend-windows-universal-dotnet-get-started-data.md).
> 
> To add a mobile service to an Windows Phone 8.0 or Windows Phone Silverlight 8.1 app project, see [Add Mobile Services to an existing Windows Phone app](mobile-services-dotnet-backend-windows-phone-get-started-data.md).
> 
> 

The following are screen captures from the completed app:

![](./media/mobile-services-windows-universal-get-started/mobile-quickstart-completed.png)
<br/>Windows Store app

![](./media/mobile-services-windows-universal-get-started/mobile-quickstart-completed-wp8.png)
<br/>Windows Phone Store app

Completing this tutorial is a prerequisite for all other Mobile Services tutorials for Windows Store and Windows Phone Store apps. 

To complete this tutorial, you need the following:

* An active Azure account. If you don't have an account, you can sign up for an Azure trial and get up to 10 free mobile services that you can keep using even after your trial ends. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-dotnet-get-started%2F).
* [Visual Studio 2013](https://go.microsoft.com/fwLink/p/?LinkID=257546).

## Create a new mobile service

Follow these steps to create a new mobile service.

1.	Log into the [Azure classic portal](https://manage.windowsazure.com/). At the bottom of the navigation pane, click **+NEW**. Expand **Compute** and **Mobile Service**, then click **Create**.
	
	![](./media/mobile-services-dotnet-backend-create-new-service/mobile-create.png)

	This displays the **Create a Mobile Service** dialog.

2.	In the **Create a Mobile Service** page, select **Create a free 20 MB SQL Database**, select **.NET** runtime, then type a subdomain name for the new mobile service in the **URL** textbox. Click the right arrow button to go to the next page.
	
	![](./media/mobile-services-dotnet-backend-create-new-service/mobile-create-page1.png)

	This displays the **Specify database settings** page.

	> [AZURE.NOTE] As part of this tutorial, you create a new SQL Database instance and server. You can reuse this new database and administer it as you would any other SQL Database instance. If you already have a database in the same region as the new mobile service, you can instead choose **Use existing Database** and then select that database. The use of a database in a different region is not recommended because of additional bandwidth costs and higher latencies.

3.	In **Name**, type the name of the new database, then type **Login name**, which is the administrator login name for the new SQL Database server, type and confirm the password, and click the check button to complete the process.
	![](./media/mobile-services-dotnet-backend-create-new-service/mobile-create-page2.png)

You have now created a new mobile service that can be used by your mobile apps.


## Create a new universal Windows app
Once you have created your mobile service, you can follow an easy quickstart in the Azure classic portal to either create a new app or modify an existing app to connect to your mobile service.

In this section you will create a new universal Windows app that is connected to your mobile service.

1. In the [Azure classic portal](https://manage.windowsazure.com/), click **Mobile Services**, and then click the mobile service that you just created.

2. In the quickstart tab, click **Windows** under **Choose platform** and expand **Create a new Windows Store app**.

       This displays the three easy steps to create a Windows Store app connected to your mobile service.

      ![Mobile Services quickstart steps](./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started/mobile-quickstart-steps.png)

3. If you haven't already done so, download and install [Visual Studio 2013](https://go.microsoft.com/fwLink/p/?LinkID=257546) on your local computer or virtual machine.

4. Under **Download and run your app and service locally**, select a language for your Windows Store app, then click **Download**.

      This downloads a solution contains projects for both the mobile service and for the sample *To do list* application that is connected to your mobile service. Save the compressed project file to your local computer, and make a note of where you save it.


## Test the app against the local mobile service

The mobile service project that you download lets you to run your new mobile service right on your local computer or virtual machine. This makes it easy to debug your service code before you even publish it to Azure.

In this section, you will test your new app against the mobile service running locally.

1. Browse to the location where you saved the compressed project files, expand the files on your computer, and open the solution file in Visual Studio.

2. In the Solution Explorer in Visual Studio, right-click your service project, click **Set as StartUp Project**, and then press the **F5** key to build the project and start the mobile service locally.

	![](./media/mobile-services-dotnet-backend-test-local-service-dotnet/mobile-service-startup.png)

	A web page is displayed after the mobile service starts successfully.

3. To test the store app, right-click your client app project, click **Set as StartUp Project**, and then press the **F5** key to rebuild the project and start the app.

	This starts the app, which connects to the local mobile service instance.	

4. In the app, type meaningful text, such as _Complete the tutorial_, in **Insert a TodoItem**, and then click **Save**.

	This sends a POST request to the local mobile service. Data from the request is inserted into the TodoItem table. Items stored in the table are returned by the mobile service, and the data is displayed in the second column in the app.

> [!NOTE]
> You can review the code that accesses your mobile service to query and insert data, which is found in the MainPage.xaml.cs file.
> 
> 
## Publish your mobile service

1. In Visual Studio, right-click the project, click **Publish** > **Microsoft Azure Mobile Services**. Instead of using Visual Studio, [you may also use Git](../articles/mobile-services/mobile-services-dotnet-backend-store-code-source-control.md).

2. Sign in with Azure credentials and select your service from **Existing Mobile Services**. Visual Studio downloads your publish settings directly from Azure. Finally, click **Publish**.


<ol start="4">
<li><p>In the Shared code project, open the App.xaml.cs file, locate the code that creates a <a href="http://msdn.microsoft.com/library/Windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx" target="_blank">MobileServiceClient</a> instance, comment-out the code that creates this client using <em>localhost</em> and uncomment the code that creates the client using the remote mobile service URL, which looks like the following:</p>

        <pre><code>public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://todolist.azure-mobile.net/",
            "XXXX-APPLICATION-KEY-XXXXX");</code></pre>

    <p>The client will now access the mobile service published to Azure.</p></li>
</ol>

## Test the app against the mobile service hosted in Azure
Now that the mobile service is published and the client is connected to the remote mobile service hosted in Azure, we can run the app using Azure for item storage.


1. Press the F5 key to rebuild the project and start the Windows Store app.

2. In the app, type meaningful text, such as *Complete the tutorial*, in **Insert a TodoItem**, and then click **Save**.

	![](./media/mobile-services-windows-universal-test-app/mobile-quickstart-startup.png)

	This sends a POST request to the new mobile service hosted in Azure.

3. Stop debugging and change the default start up project in the universal Windows solution to the Windows Phone Store app and press F5 again.

	![](./media/mobile-services-windows-universal-test-app/mobile-quickstart-completed-wp8.png)
	
	Notice that data saved from the previous step is loaded from the mobile service after the app starts.

## Next Steps
Now that you have completed the quickstart, learn how to perform additional important tasks in Mobile Services:

* [Add Mobile Services to an existing app](../mobile-services-dotnet-backend-windows-store-dotnet-get-started-data.md)
<br/>Learn more about storing and querying data using Mobile Services.

* [Get started with offline data sync](mobile-services-windows-store-dotnet-get-started-offline-data.md)
<br/>Learn how to use offline data sync to make your app responsive and robust.

* [Add authentication to your Mobile Services app ](../mobile-services-dotnet-backend-windows-store-dotnet-get-started-users.md)
<br/>Learn how to authenticate users of your app with an identity provider.

* [Add push notifications to your app](../mobile-services-dotnet-backend-windows-store-dotnet-get-started-push.md)
<br/>Learn how to send a very basic push notification to your app.

* [Troubleshoot a Mobile Services .NET backend](mobile-services-dotnet-backend-how-to-troubleshoot.md)
<br/> Learn how to diagnose and fix issues that can arise with a Mobile Services .NET backend.


For more information about universal Windows apps, see [Supporting multiple device platforms from a single mobile service](mobile-services-how-to-use-multiple-clients-single-service.md#shared-vs).


>[AZURE.TIP] **Looking for something else?**  
>If this topic didn't contain what you were expecting, is missing something, or in some other way didn't meet your needs, please provide us with you feedback using the Disqus thread below.

<!-- Anchors. -->

<!-- Images. -->



<!-- URLs. -->

[Visual Studio 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[Get started with data]: mobile-services-dotnet-backend-windows-universal-dotnet-get-started-data.md
[Get started with data]: ../mobile-services-dotnet-backend-windows-store-dotnet-get-started-data.md
[Get started with offline data sync]: mobile-services-windows-store-dotnet-get-started-offline-data.md
[Get started with authentication]: ../mobile-services-dotnet-backend-windows-store-dotnet-get-started-users.md
[Get started with push notifications]: ../mobile-services-dotnet-backend-windows-store-dotnet-get-started-push.md
[Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[Mobile Services SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[JavaScript and HTML]: mobile-services-win8-javascript/
[Azure classic portal]: https://manage.windowsazure.com/
[Troubleshoot a Mobile Services .NET backend]: mobile-services-dotnet-backend-how-to-troubleshoot.md
