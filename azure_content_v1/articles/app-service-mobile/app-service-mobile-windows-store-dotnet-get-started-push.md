<properties 
    pageTitle="Add push notifications to your Windows Runtime 8.1 universal app | Azure Mobile Apps" 
    description="Learn how to use Azure App Service Mobile Apps and Azure Notification Hubs to send push notifications to your Windows app." 
    services="app-service\mobile,notification-hubs" 
    documentationCenter="windows" 
    authors="ggailey777" 
    manager="dwrede" 
    editor=""/>

<tags 
    ms.service="app-service-mobile" 
    ms.workload="mobile" 
    ms.tgt_pltfrm="mobile-windows" 
    ms.devlang="dotnet" 
    ms.topic="article" 
    ms.date="11/25/2015" 
    ms.author="glenga"/>

# Add push notifications to your Windows Runtime 8.1 universal app
> [AZURE.SELECTOR]
- [Android](../articles/app-service-mobile-android-get-started-push.md)
- [iOS](../articles/app-service-mobile-ios-get-started-push.md)
- [Windows](../articles/app-service-mobile-windows-store-dotnet-get-started-push.md)
- [Xamarin.Android](../articles/app-service-mobile-xamarin-android-get-started-push.md)
- [Xamarin.iOS](../articles/app-service-mobile-xamarin-ios-get-started-push.md)
- [Xamarin.Forms](../articles/app-service-mobile-xamarin-forms-get-started-push.md)



&nbsp;  
>[AZURE.NOTE] This is an **Azure Mobile Apps** topic. For Mobile Services topics, see the [Mobile Services documentation center](/documentation/services/mobile-services/).
>
>App Service Mobile Apps is our newest mobile backend platform and [gives you additional advantages](app-service-mobile-value-prop-migration-from-mobile-services.md) over Mobile Services. [Migrating to App Service](app-service-mobile-migrating-from-mobile-services.md) is  recommended for customers using the .NET backend SDK. However, [Mobile Apps Node SDK](https://github.com/azure/azure-mobile-apps-node) is currently in Preview and is not yet recommended for production use. SDK and API contracts are subject to change within minor version releases.


## Overview
This topic shows you how to send push notifications to a Windows Runtime 8.1 universal app by using Azure App Service Mobile Apps and Azure Notification Hubs. In this scenario, when a new item is added, your Mobile App backend sends a push notification to all Windows apps registered with the Windows Notification Service (WNS).

This tutorial is based on the App Service Mobile App quickstart. Before you start this tutorial, you must first complete the quickstart tutorial [Create a Windows app](../app-service-mobile-windows-store-dotnet-get-started.md). If you do not use the downloaded quick start server project, you must add the push notification extension package to your project. For more information about server extension packages, see [Work with the .NET backend server SDK for Azure Mobile Apps](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md). 

## Prerequisites
To complete this tutorial, you need the following:

* An active [Microsoft Store account](http://go.microsoft.com/fwlink/p/?LinkId=280045).
* [Visual Studio Community 2013](https://go.microsoft.com/fwLink/p/?LinkID=391934)
* Complete the [quickstart tutorial](../app-service-mobile-windows-store-dotnet-get-started.md).  

## <a name="create-hub"></a>Create a Notification Hub
Follow these steps to create a new notification hub to use for push notifications. If you already have a notification hub, you can also connect it to your Mobile App backend. 

1. In the [Azure Portal], click **Browse** > **App Services**, then click your Mobile App backend > **All settings**, then under **Mobile** click **Push** > **Notification Hub**.

2. Click **+Notification Hub**, type a new **Notification Hub** name, which can be the same as your Mobile App backend, type a new namespace name or use an existing one, then click **OK** and finally **Create**.

	![](./media/app-service-mobile-create-notification-hub/create-new-hub-flow.png)

	This creates a new notification hub and connects it to your mobile app. If you have an existing notification hub, you can choose to connect it to your Mobile App backend instead of creating a new one.

Now you have connected a notification hub to your Mobile App backend. Later you will configure this notification hub to connect to a platform notification service (PNS) that sends push notifications to the native device.

[Azure Portal]: https://portal.azure.com/

## Register your app for push notifications
Before you can send push notifications to your Windows apps from Azure, you must submit your app to the Windows Store. You can then configure your server project to integrate with WNS.


1. In Visual Studio Solution Explorer, right-click the Windows Store app project, click **Store** > **Associate App with the Store...**. 

    ![Associate app with Windows Store](./media/app-service-mobile-register-wns/notification-hub-associate-win8-app.png)
    
2. In the wizard, click **Next**, sign in with your Microsoft account, type a name for your app in **Reserve a new app name**, then click **Reserve**.

3. After the app registration is successfully created, select the new app name, click **Next**, and then click **Associate**. This adds the required Windows Store registration information to the application manifest. 

7. Repeat steps 1 and 3 for the Windows Phone Store app project using the same registration you previously created for the Windows Store app.  

7. Navigate to the [Windows Dev Center](https://dev.windows.com/en-us/overview), sign-in with your Microsoft account, click the new app registration in **My apps**, then expand **Services** > **Push notifications**. 

8. In the **Push notifications** page, click **Live Services site** under **Microsoft Azure Mobile Services**.

9. In the **App Settings** tab, make a note of the values of **Client secret** and **Package SID**. 

    ![App setting in the developer center](./media/app-service-mobile-register-wns/mobile-services-win8-app-push-auth.png)

    > [AZURE.IMPORTANT] The client secret and package SID are important security credentials. Do not share these values with anyone or distribute them with your app.

## Configure the backend to send push notifications

1. In the [Azure Portal]( https://azure.portal.com/), click **Browse** > **App Services**, then click your Mobile App backend > **All settings**, then under **Mobile** click **Push**.

2. In Push notification services, click **Windows (WNS)**, enter the **Security key** (client secret) and **Package SID** that you obtained from the Live Services site, then click **Save**.

    ![Set the GCM API key in the portal](./media/app-service-mobile-configure-wns/mobile-push-wns-credentials.png)

Your Mobile App backend is now configured to use WNS to send push notifications to your Windows app using its notification hub.


## <a id="update-service"></a>Update the server to send push notifications
Now that push notifications are enabled in the app, you must update your app backend to send push notifications. Use the procedure below that matches your backend project type&mdash;either [.NET backend](#dotnet.md) or [Node.js backend](#nodejs.md).

### <a name="dotnet"></a>.NET backend project
1. In Visual Studio, right-click the server project and click **Manage NuGet Packages**, search for Microsoft.Azure.NotificationHubs, then click **Install**. This installs the Notification Hubs client library.

2. Expand **Controllers**, open TodoItemController.cs, and add the following using statements:

        using System.Collections.Generic;
     using Microsoft.Azure.NotificationHubs;
     using Microsoft.Azure.Mobile.Server.Config;
3. In the **PostTodoItem** method, add the following code after the call to **InsertAsync**: 

        // Get the settings for the server project.
     HttpConfiguration config = this.Configuration;
     MobileAppSettingsDictionary settings =
         this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();

     // Get the Notification Hubs credentials for the Mobile App.
     string notificationHubName = settings.NotificationHubName;
     string notificationHubConnection = settings
         .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;

     // Create the notification hub client.
     NotificationHubClient hub = NotificationHubClient
         .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);

     // Define a WNS payload
     var windowsToastPayload = @"<toast><visual><binding template=""ToastText01""><text id=""1"">"
                             + item.Text + @"</text></binding></visual></toast>";
     try
     {
         // Send the push notification.
         var result = await hub.SendWindowsNativeNotificationAsync(windowsToastPayload);

         // Write the success result to the logs.
         config.Services.GetTraceWriter().Info(result.State.ToString());
     }
     catch (System.Exception ex)
     {
         // Write the failure result to the logs.
         config.Services.GetTraceWriter()
             .Error(ex.Message, null, "Push.SendAsync Error");
     }

    This code tells the notification hub to send a push notification after a new item is insertion.

4. Republish the server project.


### <a name="nodejs"></a>Node.js backend project
1. If you haven't already done so, [download the quickstart project](app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart) or else use the [online editor in the Azure portal](app-service-mobile-node-backend-how-to-use-server-sdk.md#online-editor).

2. Replace the existing code in the todoitem.js file with the following:

        var azureMobileApps = require('azure-mobile-apps'),
     promises = require('azure-mobile-apps/src/utilities/promises'),
     logger = require('azure-mobile-apps/src/logger');

     var table = azureMobileApps.table();

     table.insert(function (context) {
     // For more information about the Notification Hubs JavaScript SDK,  
     // see http://aka.ms/nodejshubs
     logger.info('Running TodoItem.insert');

     // Define the WNS payload that contains the new item Text.
     var payload = "<toast><visual><binding template=\ToastText01\><text id=\"1\">"
                                 + context.item.text + "</text></binding></visual></toast>";

     // Execute the insert.  The insert returns the results as a Promise,
     // Do the push as a post-execute action within the promise flow.
     return context.execute()
         .then(function (results) {
             // Only do the push if configured
             if (context.push) {
                 // Send a WNS native toast notification.
                 context.push.wns.sendToast(null, payload, function (error) {
                     if (error) {
                         logger.error('Error while sending push notification: ', error);
                     } else {
                         logger.info('Push notification sent successfully!');
                     }
                 });
             }
             // Don't forget to return the results from the context.execute()
             return results;
         })
         .catch(function (error) {
             logger.error('Error while running context.execute: ', error);
         });
     });

     module.exports = table;  

    This sends a WNS toast notification that contains the item.text when a new todo item is inserted.

3. When editing the file on your local computer, republish the server project. 


## <a id="update-service"></a>Add push notifications to your app
1. Open the shared **App.xaml.cs** project file and add the following `using` statements:

        using System.Threading.Tasks;  
     using Windows.Networking.PushNotifications;       
2. In the same file, add the following **InitNotificationsAsync** method definition to the **App** class:

        private async Task InitNotificationsAsync()
     {
         // Get a channel URI from WNS.
         var channel = await PushNotificationChannelManager
             .CreatePushNotificationChannelForApplicationAsync();

         // Register the channel URI with Notification Hubs.
         await App.MobileService.GetPush().RegisterAsync(channel.Uri);
     }

    This code retrieves the ChannelURI for the app from WNS, and then registers that ChannelURI with your App Service Mobile App.

3. At the top of the **OnLaunched** event handler in **App.xaml.cs**, add the **async** modifier to the method definition and add the following call to the new **InitNotificationsAsync** method, as in the following example:

        protected async override void OnLaunched(LaunchActivatedEventArgs e)
     {
         await InitNotificationsAsync();

         // ...
     }

    This guarantees that the short-lived ChannelURI is registered each time the application is launched. 

4. In Solution Explorer double-click **Package.appxmanifest** of the Windows Store app, in **Notifications**, set **Toast capable** to **Yes**.

    From the **File** menu, click **Save All**.

5. Repeat the previous step in the Windows Phone Store app project.


Your app is now ready to receive toast notifications.

## <a id="test"></a>Test push notifications in your app

1. Right-click the Windows Store project, click **Set as StartUp Project**, then press the F5 key to run the Windows Store app.
	
	After the app starts, the device is registered for push notifications.

2. Stop the Windows Store app and repeat the previous step for the Windows Phone Store app.

	At this point, both devices are registered to receive push notifications.

3. Run the Windows Store app again, and type text in **Insert a TodoItem**, and then click **Save**.

   	Note that after the insert completes, both the Windows Store and the Windows Phone apps receive a push notification from WNS. The notification is displayed on Windows Phone even when the app isn't running.

   	![](./media/app-service-mobile-windows-universal-test-push/mobile-quickstart-push5-wp8.png)



## <a id="more"></a>More
* Templates give you flexibility to send cross-platform pushes and localized pushes. [How to use the managed client for Azure Mobile Apps](app-service-mobile-dotnet-how-to-use-client-library.md#how-to-register-push-templates-to-send-cross-platform-notifications) shows you how to register templates.
* Tags allow you to target segmented customers with pushes. [Work with the .NET backend server SDK for Azure Mobile Apps](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#how-to-add-tags-to-a-device-installation-to-enable-push-to-tags) shows you how to add tags to a device installation.

<!-- Anchors. -->

<!-- URLs. -->
[Azure Portal]Azure Portal]: https://portal.azure.com/

<!-- Images. -->
