<properties
    pageTitle="Use Notification Hubs to send breaking news (Windows Universal)"
    description="Use Azure Notification Hubs with tags in the registration to send breaking news to a universal Windows app."
    services="notification-hubs"
    documentationCenter="windows"
    authors="wesmc7777"
    manager="dwrede"
    editor=""/>


<tags
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="12/14/2015"
    ms.author="wesmc"/>

# Use Notification Hubs to send breaking news
> [AZURE.SELECTOR]
- [Windows Runtime 8.1 universal](../articles/notification-hubs/notification-hubs-windows-store-dotnet-send-breaking-news.md)
- [Windows Phone Silverlight 8.x](../articles/notification-hubs/notification-hubs-windows-phone-send-breaking-news.md)
- [iOS](../articles/notification-hubs/notification-hubs-ios-send-breaking-news.md)
- [Android](../articles/notification-hubs/notification-hubs-aspnet-backend-android-breaking-news.md)




## Overview
This topic shows you how to use Azure Notification Hubs to broadcast breaking news notifications to a Windows Store or Windows Phone 8.1 (non-Silverlight) app. If you are targeting Windows Phone 8.1 Silverlight, please refer to the [Windows Phone](notification-hubs-windows-phone-send-breaking-news.md) version. When complete, you will be able to register for breaking news categories you are interested in, and receive only push notifications for those categories. This scenario is a common pattern for many apps where notifications have to be sent to groups of users that have previously declared interest in them, e.g. RSS reader, apps for music fans, and so on. 

Broadcast scenarios are enabled by including one or more *tags* when creating a registration in the notification hub. When notifications are sent to a tag, all devices that have registered for the tag will receive the notification. Because tags are simply strings, they do not have to be provisioned in advance. For more information about tags, refer to [Notification Hubs Routing and Tag Expressions](notification-hubs-routing-tag-expressions.md).

## Prerequisites
This topic builds on the app you created in [Get started with Notification Hubs](/manage/services/notification-hubs/getting-started-windows-dotnet/). Before starting this tutorial, you must have already completed [Get started with Notification Hubs](/manage/services/notification-hubs/getting-started-windows-dotnet/).

## Add category selection to the app
The first step is to add the UI elements to your existing main page that enable the user to select categories to register. The categories selected by a user are stored on the device. When the app starts, a device registration is created in your notification hub with the selected categories as tags.

1. Open the MainPage.xaml project file, then copy the following code in the **Grid** element:

        <Grid>
         <Grid.RowDefinitions>
             <RowDefinition/>
             <RowDefinition/>
             <RowDefinition/>
             <RowDefinition/>
             <RowDefinition/>
         </Grid.RowDefinitions>
         <Grid.ColumnDefinitions>
             <ColumnDefinition/>
             <ColumnDefinition/>
         </Grid.ColumnDefinitions>
         <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top" HorizontalAlignment="Center"/>
         <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="1" Grid.Column="0" HorizontalAlignment="Center"/>
         <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="2" Grid.Column="0" HorizontalAlignment="Center"/>
         <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="3" Grid.Column="0" HorizontalAlignment="Center"/>
         <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="1" Grid.Column="1" HorizontalAlignment="Center"/>
         <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="2" Grid.Column="1" HorizontalAlignment="Center"/>
         <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="3" Grid.Column="1" HorizontalAlignment="Center"/>
         <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click"/>
     </Grid>



1. Right click the **Shared** project and add a new class named **Notifications**, add the **public** modifier to the class definition, then add the following **using** statements to the new code file:

        using Windows.Networking.PushNotifications;
     using Microsoft.WindowsAzure.Messaging;
     using Windows.Storage;
     using System.Threading.Tasks;
2. Copy the following code into the new **Notifications** class:

        private NotificationHub hub;

     public Notifications(string hubName, string listenConnectionString)
     {
         hub = new NotificationHub(hubName, listenConnectionString);
     }

     public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
     {
         ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
         return await SubscribeToCategories(categories);
     }

     public IEnumerable<string> RetrieveCategories()
     {
         var categories = (string) ApplicationData.Current.LocalSettings.Values["categories"];
         return categories != null ? categories.Split(','): new string[0];
     }

     public async Task<Registration> SubscribeToCategories(IEnumerable<string> categories = null)
     {
         var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

         if (categories == null)
         {
             categories = RetrieveCategories();
         }

         // Using a template registration to support notifications across platforms.
         // Any template notifications that contain messageParam and a corresponding tag expression
         // will be delivered for this registration.

         const string templateBodyWNS = "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(messageParam)</text></binding></visual></toast>";

         return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "simpleWNSTemplateExample",
                 categories);
     }

    This class uses the local storage to store the categories of news that this device has to receive. Note that instead of calling the *RegisterNativeAsync* method we call *RegisterTemplateAsync* to register for the categories using a template registration. 

    We also provide a name for the template ("simpleWNSTemplateExample"), because we might want to register more than one template (for instance one for toast notifications and one for tiles) and we need to name them in order to be able to update or delete them.

    Note that if a device registers multiple templates with the same tag, an incoming message targeting that tag will result in multiple notifications delivered to the device (one for each template). This behavior is useful when the same logical message has to result in multiple visual notifications, for instance showing both a badge and a toast in a Windows Store application.

    For more information on templates, see [Templates](notification-hubs-templates.md).


1. In the App.xaml.cs project file, add the following property to the **App** class:

        public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");

    This property is used to create and access a **Notifications** instance.

    In the above code, replace the `<hub name>` and `<connection string with listen access>` placeholders with your notification hub name and the connection string for *DefaultListenSharedAccessSignature* that you obtained earlier.

   > [!NOTE]
> Because credentials that are distributed with a client app are not generally secure, you should only distribute the key for listen access with your client app. Listen access enables your app to register for notifications, but existing registrations cannot be modified and notifications cannot be sent. The full access key is used in a secured backend service for sending notifications and changing existing registrations.
> 
2. In your MainPage.xaml.cs, add the following line:

        using Windows.UI.Popups;
3. In the MainPage.xaml.cs project file, add the following method:

        private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
     {
         var categories = new HashSet<string>();
         if (WorldToggle.IsOn) categories.Add("World");
         if (PoliticsToggle.IsOn) categories.Add("Politics");
         if (BusinessToggle.IsOn) categories.Add("Business");
         if (TechnologyToggle.IsOn) categories.Add("Technology");
         if (ScienceToggle.IsOn) categories.Add("Science");
         if (SportsToggle.IsOn) categories.Add("Sports");

         var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

         var dialog = new MessageDialog("Subscribed to: " + string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
         dialog.Commands.Add(new UICommand("OK"));
         await dialog.ShowAsync();
     }

    This method creates a list of categories and uses the **Notifications** class to store the list in the local storage and register the corresponding tags with your notification hub. When categories are changed, the registration is recreated with the new categories.


Your app is now able to store a set of categories in local storage on the device and register with the notification hub whenever the user changes the selection of categories.

## Register for notifications
These steps register with the notification hub on startup using the categories that have been stored in local storage.

> [!NOTE]
> Because the channel URI assigned by the Windows Notification Service (WNS) can change at any time, you should register for notifications frequently to avoid notification failures. This example registers for notification every time that the app starts. For apps that are run frequently, more than once a day, you can probably skip registration to preserve bandwidth if less than a day has passed since the previous registration.
> 
> 
1. Open the App.xaml.cs file and update the **InitNotificationsAsync** method to use the `notifications` class to subscribe based on categories.

        // *** Remove or comment out these lines *** 
     //var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
     //var hub = new NotificationHub("your hub name", "your listen connection string");
     //var result = await hub.RegisterNativeAsync(channel.Uri);

     var result = await notifications.SubscribeToCategories();

    This makes sure that every time the app starts it retrieves the categories from local storage and requests a registeration for these categories. The **InitNotificationsAsync** method was created as part of the [Get started with Notification Hubs](/manage/services/notification-hubs/getting-started-windows-dotnet/) tutorial.

2. In the MainPage.xaml.cs project file, add the following code to the *OnNavigatedTo* method:

        protected override void OnNavigatedTo(NavigationEventArgs e)
     {
         var categories = ((App)Application.Current).notifications.RetrieveCategories();

         if (categories.Contains("World")) WorldToggle.IsOn = true;
         if (categories.Contains("Politics")) PoliticsToggle.IsOn = true;
         if (categories.Contains("Business")) BusinessToggle.IsOn = true;
         if (categories.Contains("Technology")) TechnologyToggle.IsOn = true;
         if (categories.Contains("Science")) ScienceToggle.IsOn = true;
         if (categories.Contains("Sports")) SportsToggle.IsOn = true;
     }

    This updates the main page based on the status of previously saved categories.


The app is now complete and can store a set of categories in the device local storage used to register with the notification hub whenever the user changes the selection of categories. Next, we will define a backend that can send category notifications to this app.

## Sending tagged notifications

This section shows how to send breaking news as tagged template notifications from a .NET console app.

If you are using Mobile Services please refer to the [Get Started with Push](mobile-services-dotnet-backend-windows-store-dotnet-get-started-push.md) tutorials. 

If you want to use Java or PHP refer to [How to use Notification Hubs from Java/PHP](../articles/notification-hubs/notification-hubs-java-backend-how-to.md). You can send notifications from any backend using the [Notification Hub REST interface](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx).

Skip steps 1-3 if you created the console app for sending notifications when you completed [Get started with Notification Hubs][get-started].

1. In Visual Studio create a new Visual C# console application: 

   	![][13]

2. In the Visual Studio main menu, click **Tools**, **Library Package Manager**, and **Package Manager Console**, then in the console window type the following and press **Enter**:

        Install-Package Microsoft.Azure.NotificationHubs
 	
	This adds a reference to the Azure Notification Hubs SDK using the <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet package</a>. 

3. Open the file Program.cs and add the following `using` statement:

        using Microsoft.Azure.NotificationHubs;

4. In the `Program` class, add the following method, or replace it if it already exists:

        private static async void SendTemplateNotificationAsync()
        {
			// Define the notification hub.
		    NotificationHubClient hub = 
				NotificationHubClient.CreateClientFromConnectionString(
					"<connection string with full access>", "<hub name>");

            // Create an array of breaking news categories.
            var categories = new string[] { "World", "Politics", "Business", 
											"Technology", "Science", "Sports"};

            // Sending the notification as a template notification. All template registrations that contain 
			// "messageParam" and the proper tags will receive the notifications. 
			// This includes APNS, GCM, WNS, and MPNS template registrations.

            Dictionary<string, string> templateParams = new Dictionary<string, string>();

            foreach (var category in categories)
            {
                templateParams["messageParam"] = "Breaking " + category + " News!";            
                await hub.SendTemplateNotificationAsync(templateParams, category);
            }
		 }

	This code sends a template notification for each of the six tags in the string array. The use of tags makes sure that devices receive notifications only for the registered categories.	

6. In the above code, replace the `<hub name>` and `<connection string with full access>` placeholders with your notification hub name and the connection string for *DefaultFullSharedAccessSignature* from the dashboard of your notification hub.

7. Add the following lines in the **Main** method:

         SendTemplateNotificationAsync();
		 Console.ReadLine();

8. Build the console app.

<!-- Anchors -->
[From a console app]: #console
[From Mobile Services]: #mobile-services
[Run the app and generate notifications]: #test-app

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

[15]: ./media/notification-hubs-back-end/notification-hub-scheduler1.png
[16]: ./media/notification-hubs-back-end/notification-hub-scheduler2.png

<!-- URLs. -->
[get-started]: ../articles/notification-hubs/notification-hubs-windows-store-dotnet-get-started.md
[Use Notification Hubs to send notifications to users]: ../articles/tutorial-notify-users-mobileservices.md
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Store]: http://msdn.microsoft.com/library/jj927172.aspx
[Notification Hubs REST interface]: http://msdn.microsoft.com/library/windowsazure/dn223264.aspx


## Run the app and generate notifications
1. In Visual Studio, press F5 to compile and start the app.

    ![][1]

    Note that the app UI provides a set of toggles that lets you choose the categories to subscribe to.

2. Enable one or more categories toggles, then click **Subscribe**.

    The app converts the selected categories into tags and requests a new device registration for the selected tags from the notification hub. The registered categories are returned and displayed in a dialog.

    ![][19]

3. Send a new notification from the backend in one of the following ways:

   * **Console app:** start the console app.

* **Java/PHP:** run your app/script.

  Notifications for the selected categories appear as toast notifications.

  ![][14]



## Next steps
In this tutorial we learned how to broadcast breaking news by category. Consider completing one of the following tutorials that highlight other advanced Notification Hubs scenarios:

* [Use Notification Hubs to broadcast localized breaking news](/manage/services/notification-hubs/breaking-news-localized-dotnet/)

    Learn how to expand the breaking news app to enable sending localized notifications.


<!-- Anchors. -->

[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

[1]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-breakingnews-win1.png

[14]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-toast-2.png


[19]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-reg-2.png

<!-- URLs.-->

[get-started]: /manage/services/notification-hubs/getting-started-windows-dotnet/
[Use Notification Hubs to broadcast localized breaking news]: /manage/services/notification-hubs/breaking-news-localized-dotnet/
[Notify users with Notification Hubs]: /manage/services/notification-hubs/notify-users
[Mobile Service]: /develop/mobile/tutorials/get-started/
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Store]: http://msdn.microsoft.com/library/jj927172.aspx
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253

[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
