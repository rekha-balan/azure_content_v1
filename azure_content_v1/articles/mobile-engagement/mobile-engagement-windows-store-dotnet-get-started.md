<properties
    pageTitle="Get started with Azure Mobile Engagement for Windows Universal Apps"
    description="Learn how to use Azure Mobile Engagement with analytics and push notifications for Windows Universal Apps."
    services="mobile-engagement"
    documentationCenter="windows"
    authors="piyushjo"
    manager="dwrede"
    editor="" />

<tags
    ms.service="mobile-engagement"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows-store"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="09/22/2015"
    ms.author="piyushjo" />

# Get started with Azure Mobile Engagement for Windows Universal Apps
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
This topic shows you how to use Azure Mobile Engagement to understand your app usage and send push notifications to segmented users of a Windows Universal application.
This tutorial demonstrates the simple broadcast scenario using Mobile Engagement. You will create a blank Windows Universal App that collects basic app usage data and receives push notifications using Windows Notification Service (WNS).

This tutorial requires the following:

* Visual Studio 2013
* [MicrosoftAzure.MobileEngagement](http://go.microsoft.com/?linkid=9864592) Nuget package

> [!IMPORTANT]
> Completing this tutorial is a prerequisite for all other Mobile Engagement tutorials for Windows Universal Apps. To complete it - you must have an active Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure Free Trial</a>.
> 
> 
## <a id="setup-azme"></a>Setup Mobile Engagement for your Windows Universal app
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
This tutorial presents a "basic integration," which is the minimal set required to collect data and send a push notification. The complete integration documentation can be found in the [Mobile Engagement Windows Universal SDK integration](../mobile-engagement-windows-store-sdk-overview/.md).

We will create a basic app with Visual Studio to demonstrate the integration.

### Create a new Windows Universal App project
The following steps assume the use of Visual Studio 2015 though the steps are similar in earlier versions of Visual Studio. 

1. Start Visual Studio, and in the **Home** screen, select **New Project**.

2. In the pop-up, select **Windows 8** -> **Universal** -> **Blank App (Universal Windows 8.1)**. Fill in the app **Name** and **Solution name**, and then click **OK**.

    ![][1]


You have now created a new Windows Universal App project into which we will integrate the Azure Mobile Engagement SDK.

### Connect your app to Mobile Engagement backend
1. Install the [MicrosoftAzure.MobileEngagement](http://go.microsoft.com/?linkid=9864592) nuget package in your project. If you are targeting both Windows and Windows Phone platforms, you need to do this for both the projects. For Windows 8.x and Windows Phone 8.1 the same Nuget package places the correct platform-specific binaries in each project.

2. Open **Package.appxmanifest** and make sure that the following capability is added there:

        Internet (Client)

    ![][2]

3. Now copy the connection string that you copied earlier for your Mobile Engagement App and paste it in the `Resources\EngagementConfiguration.xml` file, between the `<connectionString>` and `</connectionString>` tags:

    ![][3]

   > [!TIP]
> If your App is going to target both Windows and Windows Phone platforms, you should still create two Mobile Engagement Applications - one for each supported platforms. This is to ensure that you are able to create correct segmentation of the audience and are able to send appropriately targeted notifications for each platform.
> 
4. In the `App.xaml.cs` file:

    a. Add the `using` statement:

            using Microsoft.Azure.Engagement;

    b. Add a method dedicated to the Engagement initialization and setting:

           private void InitEngagement(IActivatedEventArgs e)
        {
          EngagementAgent.Instance.Init(e);

          //... rest of the code
        }

    c. Initialize the SDK in the **OnLaunched** method:

            protected override void OnLaunched(LaunchActivatedEventArgs e)
         {
           InitEngagement(e);

           //... rest of the code
         }

    c. Insert the following in the **OnActivated** method and add the method if it is not already present:

            protected override void OnActivated(IActivatedEventArgs e)
         {
           InitEngagement(e);

           //... rest of the code
         }


## <a id="monitor"></a>Enable real-time monitoring
In order to start sending data and ensuring that the users are active, you must send at least one screen (Activity) to the Mobile Engagement backend.

1. In the **MainPage.xaml.cs**, add the following `using` statement:

    using Microsoft.Azure.Engagement.Overlay;

2. Replace the base class of **MainPage** from **Page** to **EngagementPageOverlay**:

        class MainPage : EngagementPageOverlay
3. In the `MainPage.xaml` file:

    a. Add to your namespaces declarations:

        xmlns:engagement="using:Microsoft.Azure.Engagement.Overlay"

    b. Replace the **Page** in the XML tag name with **engagement:EngagementPageOverlay**


> [!IMPORTANT]
> If your page overrides the `OnNavigatedTo` method, make sure to call `base.OnNavigatedTo(e)`. Otherwise,  the activity will not be reported (the `EngagementPage` calls `StartActivity` inside its `OnNavigatedTo` method). This is especially important in a Windows Phone project where the default template has an `OnNavigatedTo` method. 
> 
> 
## <a id="monitor"></a>Connect app with real-time monitoring
This section shows you how to connect your app to the Mobile Engagement backend by using the Mobile Engagement's real-time monitoring feature. 

1. In your **Azure Mobile Engagement** account, make sure you select the app you wish to monitor and manage in the **Mobile Engagement** portal. Navigate to your Mobile Engagement portal by clicking the **Engage** button at the bottom. 

	 ![](./media/mobile-engagement-connect-app-with-monitor/engage-button.png)

2. You will land in the Mobile Engagement portal. If the Monitor tab is not selected, click on the **Monitor**.

3. The monitor is ready to show you any device in real time, which will start your app.
	 
4. Start your app either in the emulator/simulator or on a connected device. You should see one session in the monitor if your integration is correct which means that your app is now connected to the Mobile Engagement backend and is sending data to it.  
	
	 ![](./media/mobile-engagement-connect-app-with-monitor/monitor.png)



## <a id="integrate-push"></a>Enable push notifications and in-app messaging
Mobile Engagement allows you to interact and reach your users with push notifications and in-app messaging in the context of campaigns. This module is called REACH in the Mobile Engagement portal.
The following sections set up your app to receive them.

### Enable your app to receive WNS Push Notifications
1. In the `Package.appxmanifest` file, in the **Application** tab, under **Notifications**, set **Toast capable:** to **Yes**

    ![][5]


### Initialize the REACH SDK
In `App.xaml.cs`, call **EngagementReach.Instance.Init(e);** in the **InitEngagement** function right after the agent initialization:

        private void InitEngagement(IActivatedEventArgs e)
        {
           EngagementAgent.Instance.Init(e);
           EngagementReach.Instance.Init(e);
        }

You're all set for sending a toast. Now we will verify that you have correctly carried out this basic integration.

### Grant access to Mobile Engagement to send notifications
1. Open [Windows Store Dev Center](https://dev.windows.com) in your web browser, login and create an account if necessary.
2. Click **Dashboard** at the top right corner and then click **Create a new app** from the left panel menu. 

    ![][9]

3. Create your app by reserving its name. 

    ![][10]

4. Once the app has been created, navigate to **Services -> Push notifications** from the left menu.

    ![][11]

5. In the Push notifications section, click on **Live Services site** link. 

    ![][12]

6. You will be navigated to the Push credentials section. Make sure you are in the **App Settings** section and then copy your **Package SID** and **Client secret**

    ![][13]

7. Navigate to the **Settings** of your Mobile Engagement portal, and click the **Native Push** section on the left. Then, click the **Edit** button to enter your **Package security identifier (SID)** and your **Secret Key** as shown below:

    ![][6]

8. Finally make sure that you have associated your Visual Studio app with this created app in the App store. You need to click on **Associate App with Store** from Visual Studio to do this.

    ![][7]


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

 

If the app was running then you will see an in-app notification otherwise you will see a toast notification if the app was closed. 
If you are seeing an in-app notification but not a toast notification and you are running the app in debug mode in Visual Studio then you should try **Lifecycle events -> Suspend** in the toolbar to ensure that the app is actually suspended. If you just clicked the Home button while debugging the application in Visual Studio then it doesn't always get suspended and while you will see the notification as in-app, it wouldn't show up as toast notification.  

![][8]

<!-- URLs. -->

[Mobile Engagement Windows Universal SDK documentation]: ../mobile-engagement-windows-store-integrate-engagement/
[MicrosoftAzure.MobileEngagement]: http://go.microsoft.com/?linkid=9864592
[Windows Store Dev Center]: https://dev.windows.com
[Windows Universal Apps - Overlay integration]: ../mobile-engagement-windows-store-integrate-engagement-reach/#overlay-integration

<!-- Images. -->

[1]: ./media/mobile-engagement-windows-store-dotnet-get-started/universal-app-creation.png
[2]: ./media/mobile-engagement-windows-store-dotnet-get-started/manifest-capabilities.png
[3]: ./media/mobile-engagement-windows-store-dotnet-get-started/add-connection-info.png
[5]: ./media/mobile-engagement-windows-store-dotnet-get-started/manifest-toast.png
[6]: ./media/mobile-engagement-windows-store-dotnet-get-started/enter-credentials.png
[7]: ./media/mobile-engagement-windows-store-dotnet-get-started/associate-app-store.png
[8]: ./media/mobile-engagement-windows-store-dotnet-get-started/vs-suspend.png
[9]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_create_app.png
[10]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_app_name.png
[11]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_services_push.png
[12]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_services_push_1.png
[13]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_services_push_creds.png


