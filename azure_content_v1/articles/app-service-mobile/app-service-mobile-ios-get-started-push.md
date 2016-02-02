<properties
    pageTitle="Add Push Notifications to iOS App with Azure  Mobile Apps"
    description="Learn how to use Azure Mobile Apps to send push notifications to your iOS app."
    services="app-service\mobile"
    documentationCenter="ios"
    manager="dwrede"
    editor=""
    authors="krisragh"/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-ios"
    ms.devlang="objective-c"
    ms.topic="article"
    ms.date="11/25/2015"
    ms.author="krisragh"/>


# Add Push Notifications to your iOS App
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
In this tutorial, you add push notifications to the [iOS quick start](app-service-mobile-ios-get-started.md) project so that every time a record is inserted, a push notification is sent. This tutorial is based on the [iOS quick start](app-service-mobile-ios-get-started.md) tutorial, which you must complete first. If you do not use the downloaded quick start server project, you must add the push notification extension package to your project. For more information about server extension packages, see [Work with the .NET backend server SDK for Azure Mobile Apps](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md). 

The [iOS simulator does not support push notifications](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/iOS_Simulator_Guide/TestingontheiOSSimulator.html), so for this tutorial, you need a physical iOS device and an [Apple Developer Program membership](https://developer.apple.com/programs/ios/). 

## <a name="create-hub"></a>Create a Notification Hub
Follow these steps to create a new notification hub to use for push notifications. If you already have a notification hub, you can also connect it to your Mobile App backend. 

1. In the [Azure Portal], click **Browse** > **App Services**, then click your Mobile App backend > **All settings**, then under **Mobile** click **Push** > **Notification Hub**.

2. Click **+Notification Hub**, type a new **Notification Hub** name, which can be the same as your Mobile App backend, type a new namespace name or use an existing one, then click **OK** and finally **Create**.

	![](./media/app-service-mobile-create-notification-hub/create-new-hub-flow.png)

	This creates a new notification hub and connects it to your mobile app. If you have an existing notification hub, you can choose to connect it to your Mobile App backend instead of creating a new one.

Now you have connected a notification hub to your Mobile App backend. Later you will configure this notification hub to connect to a platform notification service (PNS) that sends push notifications to the native device.

[Azure Portal]: https://portal.azure.com/

## <a id="register"></a>Register app for push notifications

* [Register an App ID for your app](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW991). Create an explicit App ID (not a wildcard App ID) and for **Bundle ID**, use the exact **Bundle ID** that is in your Xcode quickstart project. It is also crucial that you check the **Push Notifications** option. 

* Next, [configuring push notifications](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW6). You may create either a "Development" or "Distribution" SSL certificate (remember to select the corresponding option in the Azure portal later.)


## Configure Azure to send push notifications


1.  On your Mac, launch **Keychain Access**. Open **Category** > **My Certificates**. Find the SSL certificate to export (that you downloaded earlier) and disclose its contents. Select only the certificate without selecting the private key, and [export it](https://support.apple.com/kb/PH20122?locale=en_US).

2. In the Azure portal, click **Browse All** > **App Services** > your Mobile App backend > **Settings** > **Mobile** > **Push** > **Configure required settings** > **+ Notification Hub**, and provide a name and namespace for your notification hub, and then click the **OK** button.

  	![][1]

3. In the **Create Notification Hub** blade, click the **Create** button.
     
    Before you proceed to the next step, click **Notifications**, to ensure that your notification hub setup is complete. 

4. In the Azure portal, click **Browse All** > **App Services** > your Mobile App backend > **Settings** > **Mobile** > **Push** > **Apple Push Notification Services** > **Upload Certificate**. Upload the .p12 file, selecting the correct **Mode** (corresponding to whether the client SSL certificate you generated earlier was Development or Distribution.) Your service is now configured to work with push notifications on iOS!

[1]: ./media/app-service-mobile-apns-configure-push/mobile-push-notification-hub.png


## <a id="update-server"></a>Update server project to send push notifications

+ **.NET backend (C#)**:  	
	1. In Visual Studio, right-click the server project and click **Manage NuGet Packages**, search for `Microsoft.Azure.NotificationHubs`, then click **Install**. This installs the Notification Hubs library for sending notifications from your backend.

	2. In the backend's Visual Studio project, open **Controllers** > **TodoItemController.cs**. At the top of the file, add the following `using` statement:

	        using Microsoft.Azure.Mobile.Server.Config;
	        using Microsoft.Azure.NotificationHubs;


	3. Replace the `PostTodoItem` method with the following code:  
      
	        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
	        {
	            TodoItem current = await InsertAsync(item);
	            // Get the settings for the server project.
	            HttpConfiguration config = this.Configuration;
	
	            MobileAppSettingsDictionary settings = 
	                this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();
	
	            // Get the Notification Hubs credentials for the Mobile App.
	            string notificationHubName = settings.NotificationHubName;
	            string notificationHubConnection = settings
	                .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;
	
	            // Create a new Notification Hub client.
	            NotificationHubClient hub = NotificationHubClient
	            .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);
	
	            // iOS payload
	            var appleNotificationPayload = "{\"aps\":{\"alert\":\"" + item.Text + "\"}}";
	
	            try
	            {
	                // Send the push notification and log the results.
	                var result = await hub.SendAppleNativeNotificationAsync(appleNotificationPayload);
	
	                // Write the success result to the logs.
	                config.Services.GetTraceWriter().Info(result.State.ToString());
	            }
	            catch (System.Exception ex)
	            {
	                // Write the failure result to the logs.
	                config.Services.GetTraceWriter()
	                    .Error(ex.Message, null, "Push.SendAsync Error");
	            }
	            return CreatedAtRoute("Tables", new { id = current.Id }, current);
	        }

	4. Republish the server project.

+ **Node.js backend** : 
   
	1. If you haven't already done so, [download the quickstart project](app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart) or else use the [online editor in the Azure portal](app-service-mobile-node-backend-how-to-use-server-sdk.md#online-editor).	
	
	2. Replace the todoitem.js table script with the following code:


	        var azureMobileApps = require('azure-mobile-apps'),
	            promises = require('azure-mobile-apps/src/utilities/promises'),
	            logger = require('azure-mobile-apps/src/logger');
	        
	        var table = azureMobileApps.table();
	        
	        // When adding record, send a push notification via APNS
	        // For this to work, you must have an APNS Hub already configured
	        table.insert(function (context) {
	            // For details of the Notification Hubs JavaScript SDK, 
	            // see http://aka.ms/nodejshubs
	            logger.info('Running TodoItem.insert');
	            
				// Create a payload that contains the new item Text.
	            var payload = "{\"aps\":{\"alert\":\"" + context.item.text + "\"}}";
	            
	            // Execute the insert.  The insert returns the results as a Promise,
	            // Do the push as a post-execute action within the promise flow.
	            return context.execute()
	                .then(function (results) {
	                    // Only do the push if configured
	                    if (context.push) {
	                        context.push.apns.send(null, payload, function (error) {
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

	2. When editing the file on your local computer, republish the server project.


## <a id="add-push"></a>Add push notifications to app

**Objective-C**:

1. In **QSAppDelegate.m**, import the iOS SDK and **QSTodoService.h**:
        
        #import <MicrosoftAzureMobile/MicrosoftAzureMobile.h>
        #import "QSTodoService.h"

2. In `didFinishLaunchingWithOptions` in **QSAppDelegate.m**, insert the following lines right before `return YES;`:

        UIUserNotificationSettings* notificationSettings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil];
        [[UIApplication sharedApplication] registerUserNotificationSettings:notificationSettings];
        [[UIApplication sharedApplication] registerForRemoteNotifications];

3. In **QSAppDelegate.m**, add the following handler methods. Your app is now updated to support push notifications. 

        // Registration with APNs is successful
        - (void)application:(UIApplication *)application
        didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
        
            QSTodoService *todoService = [QSTodoService defaultService];
            MSClient *client = todoService.client;
        
            [client.push registerDeviceToken:deviceToken completion:^(NSError *error) {
                if (error != nil) {
                    NSLog(@"Error registering for notifications: %@", error);
                }
            }];
        }
        
        // Handle any failure to register
        - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:
        (NSError *)error {
            NSLog(@"Failed to register for remote notifications: %@", error);
        }
        
        // Use userInfo in the payload to display an alert.
        - (void)application:(UIApplication *)application
              didReceiveRemoteNotification:(NSDictionary *)userInfo {
            NSLog(@"%@", userInfo);
        
            NSDictionary *apsPayload = userInfo[@"aps"];
            NSString *alertString = apsPayload[@"alert"];
        
            // Create alert with notification content.
            UIAlertController *alertController = [UIAlertController
                                          alertControllerWithTitle:@"Notification"
                                          message:alertString
                                          preferredStyle:UIAlertControllerStyleAlert];
        
            UIAlertAction *cancelAction = [UIAlertAction
                                           actionWithTitle:NSLocalizedString(@"Cancel", @"Cancel")
                                           style:UIAlertActionStyleCancel
                                           handler:^(UIAlertAction *action)
                                           {
                                               NSLog(@"Cancel");
                                           }];
            
            UIAlertAction *okAction = [UIAlertAction
                                       actionWithTitle:NSLocalizedString(@"OK", @"OK")
                                       style:UIAlertActionStyleDefault
                                       handler:^(UIAlertAction *action)
                                       {
                                           NSLog(@"OK");
                                       }];
            
            [alertController addAction:cancelAction];
            [alertController addAction:okAction];
            
            // Get current view controller.
            UIViewController *currentViewController = [[[[UIApplication sharedApplication] delegate] window] rootViewController];
            while (currentViewController.presentedViewController)
            {
                currentViewController = currentViewController.presentedViewController;
            }
            
            // Display alert.
            [currentViewController presentViewController:alertController animated:YES completion:nil];
        
        }

**Swift**:

1. Add file **ClientManager.swift** with the following contents. Replace _%AppUrl%_ with the URL of the Azure Mobile App backend.
        
        class ClientManager {
            static let sharedClient = MSClient(applicationURLString: "%AppUrl%")
        }

2. In **ToDoTableViewController.swift**, replace the `let client` line that initializes an `MSClient` with this line:

        let client = ClientManager.sharedClient
 
3. In **AppDelegate.swift**, replace the body of `func application` as follows:

        func application(application: UIApplication,
           didFinishLaunchingWithOptions launchOptions: [NSObject : AnyObject]?) -> Bool {
           application.registerUserNotificationSettings(
               UIUserNotificationSettings(forTypes: [.Alert, .Badge, .Sound],
                   categories: nil))
           application.registerForRemoteNotifications()
           return true
        }

2. In **AppDelegate.swift**, add the following handler methods. Your app is now updated to support push notifications.
        

        func application(application: UIApplication,
        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
            ClientManager.sharedClient.push?.registerDeviceToken(deviceToken, completion: { (error) -> Void in
                NSLog("Error registering for notifications: %@", error!.description)
            })
        }

            
        func application(application: UIApplication,
        didFailToRegisterForRemoteNotificationsWithError error: NSError) {
            NSLog("Failed to register for remote notifications: \n%@", error.description)
        }

        func application(application: UIApplication,
        didReceiveRemoteNotification userInfo: [NSObject : AnyObject]) {
            
            NSLog("%@", userInfo)
            
            let apsNotification = userInfo["aps"] as! NSDictionary
            let apsString       = apsNotification["alert"] as! String
            
            
            let alert = UIAlertController(title: "Alert", message:apsString, preferredStyle: .Alert)
            let okAction = UIAlertAction(title: "OK", style: .Default) { _ in
                NSLog("OK")
            }
            let cancelAction = UIAlertAction(title: "Cancel", style: .Default) { _ in
                NSLog("Cancel")
            }
            
            alert.addAction(okAction)
            alert.addAction(cancelAction)
            
            var currentViewController = UIApplication.sharedApplication().delegate?.window??.rootViewController
            while currentViewController?.presentedViewController != nil {
                currentViewController = currentViewController?.presentedViewController
            }
            
            currentViewController?.presentViewController(alert, animated: true){}
            
        }
    


## <a id="test"></a>Test push notifications in app

* In Xcode, press **Run** and start the app on an iOS device (not the simulator.) Click **OK** to accept push notifications; this request occurs the first time the app runs.

* In the app, add a new item and click **+**.

* Verify that a notification is received, then click **OK** to dismiss the notification. You have now successfully completed this tutorial.

  	![](../articles/media/mobile-services-ios-get-started-push/mobile-quickstart-push3-ios.png)


## <a id="more"></a>More
* Templates give you flexibility to send cross-platform pushes and localized pushes. [How to Use iOS Client Library for Azure Mobile Apps](app-service-mobile-ios-how-to-use-client-library.md#templates) shows you how to register templates.
* Tags allow you to target segmented customers with pushes. [Work with the .NET backend server SDK for Azure Mobile Apps](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#tags) shows you how to add tags to a device installation.

<!-- Anchors.  -->
[Generate iOS certificate signing request]Generate iOS certificate signing request]: #certificates
[Register your app and enable push notifications]Register your app and enable push notifications]: #register
[Create a provisioning profile for the app]Create a provisioning profile for the app]: #profile
[Add push notifications to the app]Add push notifications to the app]: #add-push
[Configure your mobile backend to send push requests]Configure your mobile backend to send push requests]: #configure
[Update the server to send push notifications]Update the server to send push notifications]: #update-server
[Publish the mobile backend to Azure]Publish the mobile backend to Azure]: #publish-mobile-service
[Test your app]Test your app]: #test-the-service

<!-- Images. -->

<!-- URLs. -->

[iOS quick start]: app-service-mobile-ios-get-started.md
