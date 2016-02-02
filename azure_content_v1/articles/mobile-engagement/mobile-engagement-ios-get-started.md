<properties
    pageTitle="Get Started with Azure Mobile Engagement for iOS in Objective C"
    description="Learn how to use Azure Mobile Engagement with analytics and push notifications for iOS apps."
    services="mobile-engagement"
    documentationCenter="ios"
    authors="piyushjo"
    manager="dwrede"
    editor="" />

<tags
    ms.service="mobile-engagement"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-ios"
    ms.devlang="objective-c"
    ms.topic="hero-article"
    ms.date="09/22/2015"
    ms.author="piyushjo" />

# Get Started with Azure Mobile Engagement for iOS apps in Objective C
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Universal Windows](mobile-engagement-windows-store-dotnet-get-started.md)
> * [Windows Phone Silverlight](mobile-engagement-windows-phone-get-started.md)
> * [iOS | Obj C](mobile-engagement-ios-get-started.md)
> * [iOS | Swift](mobile-engagement-ios-swift-get-started.md)
> * [Android](mobile-engagement-android-get-started.md)
> * [Cordova](mobile-engagement-cordova-get-started.md)
> 
> 
This topic shows you how to use Azure Mobile Engagement to understand your app usage and send push notifications to segmented users to an iOS application.
In this tutorial, you create a blank iOS app that collects basic data and receives push notifications using Apple Push Notification System (APNS).

This tutorial requires the following:

* XCode 6 or XCode 7, which you can install from your MAC App Store
* the [Mobile Engagement iOS SDK](http://aka.ms/qk2rnj)

Completing this tutorial is a prerequisite for all other Mobile Engagement tutorials for iOS apps.

> [!IMPORTANT]
> Completing this tutorial is a prerequisite for all other Mobile Engagement tutorials for iOS apps, and to complete it, you must have an active Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure Free Trial</a>.
> 
> 
## <a id="setup-azme"></a>Setup Mobile Engagement for your iOS app
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



## <a id="connecting-app"></a>Connect your app to the Mobile Engagement backend
This tutorial presents a "basic integration", which is the minimal set required to collect data and send a push notification. The complete integration documentation can be found in the [Mobile Engagement iOS SDK integration](../mobile-engagement-ios-sdk-overview/.md)

We will create a basic app with XCode to demonstrate the integration.

### Create a new iOS project
1. Start **Xcode** and in the pop-up, select **Create a new Xcode project**.

	![](./media/mobile-engagement-create-new-ios-app/xcode-new-project.png)

2. Select **Single View Application**, and then click **Next**.

	![](./media/mobile-engagement-create-new-ios-app/xcode-simple-view.png)

3. Fill in the **Product Name**, **Organization Name**, and **Organization Identifier**. Select **Objective-C** or **Swift** in the **Language** selection based on your app.

	![](./media/mobile-engagement-create-new-ios-app/xcode-project-props.png)

> [AZURE.IMPORTANT] Make sure that the Bundle Identifier matches with what you have defined in the Apple Developer console for AppId and that you have a corresponding certificate for it. 

Xcode will create the demo app into which we integrate Mobile Engagement.



### Connect your app to the Mobile Engagement backend
1. Download the [Mobile Engagement iOS SDK](http://aka.ms/qk2rnj).
2. Extract the .tar.gz file to a folder in your computer.
3. Right-click the project, and then select **Add files to**.

    ![][1]

4. Navigate to the folder where you extracted the SDK, select the `EngagementSDK` folder, and then press **OK**.

    ![][2]

5. Open the **Build Phases** tab, and in the **Link Binary With Libraries** menu, add the frameworks as shown below:

    ![][3]

6. For **XCode 7** - add `libxml2.tbd` instead of `libxml2.dylib`.

7. Go back to the Azure portal in your app's **Connection Info** page and copy the connection string.

    ![][4]

8. Add the following line of code in your **AppDelegate.m** file.

        #import "EngagementAgent.h"
9. Now paste the connection string in the `didFinishLaunchingWithOptions` delegate.

        - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
     {
           [...]
         //[EngagementAgent setTestLogEnabled:YES];

           [EngagementAgent init:@"Endpoint={YOUR_APP_COLLECTION.DOMAIN};SdkKey={YOUR_SDK_KEY};AppId={YOUR_APPID}"];
           [...]
     }
10. `setTestLogEnabled` is an optional statement which enables SDK logs for you to identify issues. 


## <a id="monitor"></a>Enable real-time monitoring
In order to start sending data and ensuring that the users are active, you must send at least one screen (Activity) to the Mobile Engagement backend.

1. Open the **ViewController.h** file and import **EngagementViewController.h**:

    `# import "EngagementViewController.h"`

2. Now replace the super class of the **ViewController** interface by `EngagementViewController`:

    `@interface ViewController : EngagementViewController`


## <a id="monitor"></a>Connect app with real-time monitoring
This section shows you how to connect your app to the Mobile Engagement backend by using the Mobile Engagement's real-time monitoring feature. 

1. In your **Azure Mobile Engagement** account, make sure you select the app you wish to monitor and manage in the **Mobile Engagement** portal. Navigate to your Mobile Engagement portal by clicking the **Engage** button at the bottom. 

	 ![](./media/mobile-engagement-connect-app-with-monitor/engage-button.png)

2. You will land in the Mobile Engagement portal. If the Monitor tab is not selected, click on the **Monitor**.

3. The monitor is ready to show you any device in real time, which will start your app.
	 
4. Start your app either in the emulator/simulator or on a connected device. You should see one session in the monitor if your integration is correct which means that your app is now connected to the Mobile Engagement backend and is sending data to it.  
	
	 ![](./media/mobile-engagement-connect-app-with-monitor/monitor.png)



## <a id="integrate-push"></a>Enable push notifications and in-app messaging
Mobile Engagement allows you to interact with your users and REACH with push notifications and in-app messaging in the context of campaigns. This module is called REACH in the Mobile Engagement portal.
The following sections set up your app to receive them.

### Enable your app to receive Silent Push Notifications
> [AZURE.IMPORTANT] To receive Push Notifications from Mobile Engagement, you need to enable `Silent Remote Notifications` in your application. You need to add the remote-notification value to the UIBackgroundModes array in your Info.plist file.

1. Open `info.plist` file in the project
2. Right click on the top item in the list (`Information Property List`) and add a new row

	![](./media/mobile-engagement-ios-silent-push/xcode-plist-add-silent-push1.png)

3. In the new row enter `Required background modes`

	![](./media/mobile-engagement-ios-silent-push/xcode-plist-add-silent-push2.png)

4. Click on the left arrow to expand the row
5. Add the following value to the item 0 `App downloads content in response to push notifications`

	![](./media/mobile-engagement-ios-silent-push/xcode-plist-add-silent-push3.png)

6. Once you make the change, the info.plist XML should contain the following key and value:

	    <key>UIBackgroundModes</key>
	    <array>
	    <string>remote-notification</string>
	    </array>

7. If you are using **Xcode 7+** and **iOS 9+**:
	- Enable **Push Notifications** in Targets > Your Target Name > Capabilities.


### Add the Reach library to your project
1. Right-click your project.
2. Select **Add file to**.
3. Navigate to the folder where you extracted the SDK.
4. Select the `EngagementReach` folder.
5. Click **Add**.

### Modify your Application Delegate
1. Back in **AppDeletegate.m** file, import the Engagement Reach module.

        #import "AEReachModule.h"
2. Inside the `application:didFinishLaunchingWithOptions` method, create a Reach module and pass it to your existing Engagement initialization line:

        - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
         AEReachModule * reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
         [EngagementAgent init:@"Endpoint={YOUR_APP_COLLECTION.DOMAIN};SdkKey={YOUR_SDK_KEY};AppId={YOUR_APPID}" modules:reach, nil];
         [...]
         return YES;
     }


### Enable your app to receive APNS Push Notifications
1. Add the following line to the `application:didFinishLaunchingWithOptions` method:

        if ([application respondsToSelector:@selector(registerUserNotificationSettings:)]) {
         [application registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert) categories:nil]];
         [application registerForRemoteNotifications];
     }
     else {

         [application registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert)];
     }
2. Add the `application:didRegisterForRemoteNotificationsWithDeviceToken` method as follows:

        - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
     {
          [[EngagementAgent shared] registerDeviceToken:deviceToken];
         NSLog(@"Registered Token: %@", deviceToken);
     }
3. Add the `didFailToRegisterForRemoteNotificationsWithError` method as follows:

        - (void)application:(UIApplication*)application didFailToRegisterForRemoteNotificationsWithError:(NSError*)error
     {

        NSLog(@"Failed to get token, error: %@", error);
     }
4. Add the `didReceiveRemoteNotification:fetchCompletionHandler` method as follows:

        - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))handler
     {
         [[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:handler];
     }


###Grant access to your Push Certificate to Mobile Engagement

To allow Mobile Engagement to send Push Notifications on your behalf, you need to grant it access to your certificate. This is done by configuring and entering your certificate into the Mobile Engagement portal. Make sure you obtain your .p12 certificate as explained in [Apple's documentation](https://developer.apple.com/library/prerelease/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW6)

1. Navigate to your Mobile Engagement portal. Ensure you're in the correct and then click on the **Engage** button at the bottom:

	![](./media/mobile-engagement-ios-send-push/engage-button.png)

2. Click on the **Settings** page in your Engagement Portal. From there click on the **Native Push** section to upload your p12 certificate:

	![](./media/mobile-engagement-ios-send-push/engagement-portal.png)

3. Select your p12, upload it and type your password:

	![](./media/mobile-engagement-ios-send-push/native-push-settings.png)

##<a id="send"></a>Send a notification to your app

We will now create a simple Push Notification campaign that will send a push to our app:

1. Navigate to the **Reach** tab in your Mobile Engagement portal.

2. Click **New Announcement** to create your push campaign

	![](./media/mobile-engagement-ios-send-push/new-announcement.png)

3. Setup the first fields of your campaign:

	![](./media/mobile-engagement-ios-send-push/campaign-first-params.png)

	- 	Provide a **Name** for your campaign 
	- 	Select the **Delivery time** as **Out of app only**: this is the simple Apple push notification type that features some text.
	- 	In the notification text, type first the **Title** which will be the first line in the push.
	- 	Then type your **Message** which will be the second line

4. Scroll down, and in the content section select **Notification only**

	![](./media/mobile-engagement-ios-send-push/campaign-content.png)

5. You're done setting the most basic campaign. Now scroll down and click on **Create** button to save your push notification campaign. 

6. Finally - click on **Activate** to send push notification. 

	![](./media/mobile-engagement-ios-send-push/campaign-activate.png)

7. You will be able receive the notification on your iOS device in the notification center like the following:

	![](./media/mobile-engagement-ios-send-push/iphone-notification.png)

8. If you have an Apple Watch paired with this iOS device then you will see the notification on your Apple Watch:

	![](./media/mobile-engagement-ios-send-push/apple-watch.png)


 

 

<!-- URLs. -->

[Mobile Engagement iOS SDK]: http://aka.ms/qk2rnj

<!-- Images. -->

[1]: ./media/mobile-engagement-ios-get-started/xcode-add-files.png
[2]: ./media/mobile-engagement-ios-get-started/xcode-select-engagement-sdk.png
[3]: ./media/mobile-engagement-ios-get-started/xcode-build-phases.png
[4]: ./media/mobile-engagement-ios-get-started/app-connection-info-page.png

