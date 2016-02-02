<properties
    pageTitle="Get started with Azure Mobile Engagement for Windows Phone Silverlight apps"
    description="Learn how to use Azure Mobile Engagement with analytics and push notifications for Windows Phone Silverlight apps."
    services="mobile-engagement"
    documentationCenter="windows"
    authors="piyushjo"
    manager="dwrede"
    editor="" />

<tags
    ms.service="mobile-engagement"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows-phone"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="09/22/2015"
    ms.author="piyushjo" />

# Get started with Azure Mobile Engagement for Windows Phone Silverlight apps
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Windows Universal](mobile-engagement-windows-store-dotnet-get-started.md)
> * [Windows Phone Silverlight](mobile-engagement-windows-phone-get-started.md)
> * [iOS | Obj C](mobile-engagement-ios-get-started.md)
> * [iOS | Swift](mobile-engagement-ios-swift-get-started.md)
> * [Android](mobile-engagement-android-get-started.md)
> * [Cordova](mobile-engagement-cordova-get-started.md)
> 
> 
This topic shows you how to use Azure Mobile Engagement to understand your app usage and send push notifications to segmented users of a Windows Phone Silverlight application.
This tutorial demonstrates the simple broadcast scenario using Mobile Engagement. In it, you create a blank Windows Phone Silverlight app that collects basic data and receives push notifications using Microsoft Push Notification Service (MPNS).

> [!NOTE]
> If you are targeting Windows Phone 8.1 (non-Silverlight), refer to the [Windows Universal tutorial](mobile-engagement-windows-store-dotnet-get-started.md).
> 
> 
This tutorial requires the following:

* Visual Studio 2013
* [MicrosoftAzure.MobileEngagement](http://go.microsoft.com/?linkid=9874664) Nuget package

> [!IMPORTANT]
> Completing this tutorial is a prerequisite for all other Mobile Engagement tutorials for Windows Phone Silverlight apps, and to complete it, you must have an active Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure Free Trial</a>.
> 
> 
## <a id="setup-azme"></a>Setup Mobile Engagement for your Windows Phone app
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
This tutorial presents a "basic integration", which is the minimal set required to collect data and send a push notification. The complete integration documentation can be found in the [Mobile Engagement Windows Phone SDK integration](../mobile-engagement-windows-phone-sdk-overview/.md)

We will create a basic app with Visual Studio to demonstrate the integration.

### Create a new Windows Phone Silverlight project
The following steps assume the use of Visual Studio 2015 though the steps are similar in earlier versions of Visual Studio. 

1. Start Visual Studio, and in the **Home** screen, select **New Project**.

2. In the pop-up, select **Windows 8** -> **Windows Phone** -> **Blank App (Windows Phone Silverlight)**. Fill in the app **Name** and **Solution name**, and then click **OK**.

    ![][1]

3. You can choose to target either **Windows Phone 8.0** or **Windows Phone 8.1**.


You have now created a new Windows Phone Silverlight app into which we will integrate the Azure Mobile Engagement SDK.

### Connect your app to the Mobile Engagement backend
1. Install the [MicrosoftAzure.MobileEngagement](http://go.microsoft.com/?linkid=9874664) nuget package in your project.

2. Open `WMAppManifest.xml` (under the Properties folder) and make sure the following are declared (add them if they are not) in the `<Capabilities />` tag:

        <Capability Name="ID_CAP_NETWORKING" />
     <Capability Name="ID_CAP_IDENTITY_DEVICE" />

    ![][2]

3. Now paste the connection string that you copied earlier for your Mobile Engagement app and paste it in the `Resources\EngagementConfiguration.xml` file, between the `<connectionString>` and `</connectionString>` tags:

    ![][3]

4. In the `App.xaml.cs` file:

    a. Add the `using` statement:

            using Microsoft.Azure.Engagement;

    b. Initialize the SDK in the `Application_Launching` method:

            private void Application_Launching(object sender, LaunchingEventArgs e)
         {
           EngagementAgent.Instance.Init();
         }

    c. Insert the following in the `Application_Activated`:

            private void Application_Activated(object sender, ActivatedEventArgs e)
         {
            EngagementAgent.Instance.OnActivated(e);
         }


## <a id="monitor"></a>Enable real-time monitoring
In order to start sending data and ensuring that the users are active, you must send at least one screen (Activity) to the Mobile Engagement backend.

1. In the MainPage.xaml.cs, add the `using` statement:

        using Microsoft.Azure.Engagement;
2. Replace the base class of **MainPage**, which is before **PhoneApplicationPage**, with **EngagementPage**.

        class MainPage : EngagementPage 
3. In your `MainPage.xml` file:

    a. Add to your namespaces declarations:

            xmlns:engagement="clr-namespace:Microsoft.Azure.Engagement;assembly=Microsoft.Azure.Engagement.EngagementAgent.WP"

    b. Replace `phone:PhoneApplicationPage` in the XML tag name with `engagement:EngagementPage`.


## <a id="monitor"></a>Connect app with real-time monitoring
This section shows you how to connect your app to the Mobile Engagement backend by using the Mobile Engagement's real-time monitoring feature. 

1. In your **Azure Mobile Engagement** account, make sure you select the app you wish to monitor and manage in the **Mobile Engagement** portal. Navigate to your Mobile Engagement portal by clicking the **Engage** button at the bottom. 

	 ![](./media/mobile-engagement-connect-app-with-monitor/engage-button.png)

2. You will land in the Mobile Engagement portal. If the Monitor tab is not selected, click on the **Monitor**.

3. The monitor is ready to show you any device in real time, which will start your app.
	 
4. Start your app either in the emulator/simulator or on a connected device. You should see one session in the monitor if your integration is correct which means that your app is now connected to the Mobile Engagement backend and is sending data to it.  
	
	 ![](./media/mobile-engagement-connect-app-with-monitor/monitor.png)



## <a id="integrate-push"></a>Enable push notifications and in-app messaging
Mobile Engagement allows you to interact and reach your users with Push Notifications and in-app Messaging in the context of campaigns. This module is called REACH in the Mobile Engagement portal.
The following sections set up your app to receive them.

### Enable your app to receive MPNS Push Notifications
Add new Capabilities to your `WMAppManifest.xml` file:

        ID_CAP_PUSH_NOTIFICATION
        ID_CAP_WEBBROWSERCOMPONENT

   ![][5]

### Initialize the REACH SDK
1. In `App.xaml.cs`, call `EngagementReach.Instance.Init();` in the **Application_Launching** function, right after the agent initialization:

        private void Application_Launching(object sender, LaunchingEventArgs e)
     {
        EngagementAgent.Instance.Init();
        EngagementReach.Instance.Init();
     }
2. In `App.xaml.cs`, call `EngagementReach.Instance.OnActivated(e);` in the **Application_Activated** function, right after the agent initialization:

        private void Application_Activated(object sender, ActivatedEventArgs e)
     {
        EngagementAgent.Instance.OnActivated(e);
        EngagementReach.Instance.OnActivated(e);
     }


You're all set. Now we will verify that you have correctly cried out this basic integration.

## <a id="send"></a>Send a notification to your app
We will now create a simple push notification campaign that sends a push notification to our app.

1. Navigate to the **REACH** tab in your Mobile Engagement portal.

2. Click **New announcement** to create your push notification campaign.

	![](./media/mobile-engagement-windows-push-campaign/new-announcement.png)

3. Set up the first field of your campaign through the following steps:

	![](./media/mobile-engagement-windows-push-campaign/campaign-first-params.png)

	a. Provide a **Name** for your campaign.

	b. Select **Delivery time** as *Any time*.

	d. In the notification text - type the **Title** which will be in bold in the push.

	e. Then type your **Message**

4. Scroll down, and in the **Content** section, select **Notification only**.

	![](./media/mobile-engagement-windows-push-campaign/campaign-content.png)

5. You're done setting the most basic campaign possible. Now scroll down again and click the **Create** button to save your campaign.

6. Last step: Click **Activate** to activate your campaign and to send push notifications.

	![](./media/mobile-engagement-windows-push-campaign/campaign-activate.png)

 

You should now see a notification on your device which will show up as an in-app notification if the app is open otherwise as a toast notification like the following: 

![][6]

<!-- URLs. -->

[MicrosoftAzure.MobileEngagement]: http://go.microsoft.com/?linkid=9874664
[Mobile Engagement Windows Phone SDK documentation]: ../mobile-engagement-windows-phone-integrate-engagement/

<!-- Images. -->

[1]: ./media/mobile-engagement-windows-phone-get-started/project-properties.png
[2]: ./media/mobile-engagement-windows-phone-get-started/wmappmanifest-capabilities.png
[3]: ./media/mobile-engagement-windows-phone-get-started/add-connection-string.png
[5]: ./media/mobile-engagement-windows-phone-get-started/reach-capabilities.png
[6]: ./media/mobile-engagement-windows-phone-get-started/push-screenshot.png
