<properties
    pageTitle="Notification Hubs Breaking News Tutorial - Android"
    description="Learn how to use Azure Service Bus Notification Hubs to send breaking news notifications to Android devices."
    services="notification-hubs"
    documentationCenter="android"
    authors="wesmc7777"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="12/15/2015" 
    ms.author="wesmc"/>


# Use Notification Hubs to send breaking news
> [AZURE.SELECTOR]
- [Windows Runtime 8.1 universal](../articles/notification-hubs/notification-hubs-windows-store-dotnet-send-breaking-news.md)
- [Windows Phone Silverlight 8.x](../articles/notification-hubs/notification-hubs-windows-phone-send-breaking-news.md)
- [iOS](../articles/notification-hubs/notification-hubs-ios-send-breaking-news.md)
- [Android](../articles/notification-hubs/notification-hubs-aspnet-backend-android-breaking-news.md)




## Overview
This topic shows you how to use Azure Notification Hubs to broadcast breaking news notifications to an Android app. When complete, you will be able to register for breaking news categories you are interested in, and receive only push notifications for those categories. This scenario is a common pattern for many apps where notifications have to be sent to groups of users that have previously declared interest in them, e.g. RSS reader, apps for music fans, etc.

Broadcast scenarios are enabled by including one or more *tags* when creating a registration in the notification hub. When notifications are sent to a tag, all devices that have registered for the tag will receive the notification. Because tags are simply strings, they do not have to be provisioned in advance. For more information about tags, refer to [Notification Hubs Routing and Tag Expressions](notification-hubs-routing-tag-expressions.md).

## Prerequisites
This topic builds on the app you created in [Get started with Notification Hubs](notification-hubs-android-get-started.md). Before starting this tutorial, you must have already completed [Get started with Notification Hubs](notification-hubs-android-get-started.md).

## Add category selection to the app
The first step is to add the UI elements to your existing main activity that enable the user to select categories to register. The categories selected by a user are stored on the device. When the app starts, a device registration is created in your notification hub with the selected categories as tags.

1. Open your res/layout/activity_main.xml file, and substitute the content with the following:

        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:tools="http://schemas.android.com/tools"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         android:paddingBottom="@dimen/activity_vertical_margin"
         android:paddingLeft="@dimen/activity_horizontal_margin"
         android:paddingRight="@dimen/activity_horizontal_margin"
         android:paddingTop="@dimen/activity_vertical_margin"
         tools:context="com.example.breakingnews.MainActivity"
         android:orientation="vertical">

             <CheckBox
                 android:id="@+id/worldBox"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="@string/label_world" />
             <CheckBox
                 android:id="@+id/politicsBox"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="@string/label_politics" />
             <CheckBox
                 android:id="@+id/businessBox"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="@string/label_business" />
             <CheckBox
                 android:id="@+id/technologyBox"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="@string/label_technology" />
             <CheckBox
                 android:id="@+id/scienceBox"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="@string/label_science" />
             <CheckBox
                 android:id="@+id/sportsBox"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:text="@string/label_sports" />
             <Button
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:onClick="subscribe"
                 android:text="@string/button_subscribe" />
     </LinearLayout>
2. Open your res/values/strings.xml file and add the following lines:

        <string name="button_subscribe">Subscribe</string>
     <string name="label_world">World</string>
     <string name="label_politics">Politics</string>
     <string name="label_business">Business</string>
     <string name="label_technology">Technology</string>
     <string name="label_science">Science</string>
     <string name="label_sports">Sports</string>

    Your main_activity.xml graphical layout should now look like this:

    ![][A1]

3. Now create a class **Notifications** in the same package as your **MainActivity** class.

        import java.util.HashSet;
     import java.util.Set;

     import android.content.Context;
     import android.content.SharedPreferences;
     import android.os.AsyncTask;
     import android.util.Log;
     import android.widget.Toast;

     import com.google.android.gms.gcm.GoogleCloudMessaging;
     import com.microsoft.windowsazure.messaging.NotificationHub;

     public class Notifications {
         private static final String PREFS_NAME = "BreakingNewsCategories";
         private GoogleCloudMessaging gcm;
         private NotificationHub hub;
         private Context context;
         private String senderId;

         public Notifications(Context context, String senderId, String hubName, 
                                 String listenConnectionString) {
             this.context = context;
             this.senderId = senderId;

             gcm = GoogleCloudMessaging.getInstance(context);
             hub = new NotificationHub(hubName, listenConnectionString, context);
         }

         public void storeCategoriesAndSubscribe(Set<String> categories)
         {
             SharedPreferences settings = context.getSharedPreferences(PREFS_NAME, 0);
             settings.edit().putStringSet("categories", categories).commit();
             subscribeToCategories(categories);
         }

         public Set<String> retrieveCategories() {
             SharedPreferences settings = context.getSharedPreferences(PREFS_NAME, 0);
             return settings.getStringSet("categories", new HashSet<String>());
         }

         public void subscribeToCategories(final Set<String> categories) {
             new AsyncTask<Object, Object, Object>() {
                 @Override
                 protected Object doInBackground(Object... params) {
                     try {
                         String regid = gcm.register(senderId);

                         String templateBodyGCM = "{\"data\":{\"message\":\"$(messageParam)\"}}";

                         hub.registerTemplate(regid,"simpleGCMTemplate", templateBodyGCM, 
                             categories.toArray(new String[categories.size()]));
                     } catch (Exception e) {
                         Log.e("MainActivity", "Failed to register - " + e.getMessage());
                         return e;
                     }
                     return null;
                 }

                 protected void onPostExecute(Object result) {
                     String message = "Subscribed for categories: "
                             + categories.toString();
                     Toast.makeText(context, message,
                             Toast.LENGTH_LONG).show();
                 }
             }.execute(null, null, null);
         }

     }

    This class uses the local storage to store the categories of news that this device has to receive. It also contains methods to register for these categories.


1. In your **MainActivity** class remove your private fields for **NotificationHub** and **GoogleCloudMessaging**, and add a field for **Notifications**:

        // private GoogleCloudMessaging gcm;
     // private NotificationHub hub;
     private Notifications notifications;
2. Then, in the **onCreate** method, remove the initialization of the **hub** field and the **registerWithNotificationHubs** method. Then add the following lines which initialize an instance of the **Notifications** class. 


        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            MyHandler.mainActivity = this;

            NotificationsManager.handleNotifications(this, SENDER_ID,
                    MyHandler.class);

            notifications = new Notifications(this, SENDER_ID, HubName, HubListenConnectionString);

            notifications.subscribeToCategories(notifications.retrieveCategories());
        }

    `HubName` and `HubListenConnectionString` should already be set with the `<hub name>` and `<connection string with listen access>` placeholders with your notification hub name and the connection string for *DefaultListenSharedAccessSignature* that you obtained earlier.

    > [AZURE.NOTE] Because credentials that are distributed with a client app are not generally secure, you should only distribute the key for listen access with your client app. Listen access enables your app to register for notifications, but existing registrations cannot be modified and notifications cannot be sent. The full access key is used in a secured backend service for sending notifications and changing existing registrations.


1. Then, add the following imports and `subscribe` method to handle the subscribe button click event:

        import android.widget.CheckBox;
     import java.util.HashSet;
     import java.util.Set;

     public void subscribe(View sender) {
         final Set<String> categories = new HashSet<String>();

         CheckBox world = (CheckBox) findViewById(R.id.worldBox);
         if (world.isChecked())
             categories.add("world");
         CheckBox politics = (CheckBox) findViewById(R.id.politicsBox);
         if (politics.isChecked())
             categories.add("politics");
         CheckBox business = (CheckBox) findViewById(R.id.businessBox);
         if (business.isChecked())
             categories.add("business");
         CheckBox technology = (CheckBox) findViewById(R.id.technologyBox);
         if (technology.isChecked())
             categories.add("technology");
         CheckBox science = (CheckBox) findViewById(R.id.scienceBox);
         if (science.isChecked())
             categories.add("science");
         CheckBox sports = (CheckBox) findViewById(R.id.sportsBox);
         if (sports.isChecked())
             categories.add("sports");

         notifications.storeCategoriesAndSubscribe(categories);
     }

    This method creates a list of categories and uses the **Notifications** class to store the list in the local storage and register the corresponding tags with your notification hub. When categories are changed, the registration is recreated with the new categories.


Your app is now able to store a set of categories in local storage on the device and register with the notification hub whenever the user changes the selection of categories.

## Register for notifications
These steps register with the notification hub on startup using the categories that have been stored in local storage.

> [!NOTE]
> Because the registrationId assigned by Google Cloud Messaging (GCM) can change at any time, you should register for notifications frequently to avoid notification failures. This example registers for notification every time that the app starts. For apps that are run frequently, more than once a day, you can probably skip registration to preserve bandwidth if less than a day has passed since the previous registration.
> 
> 
1. Add the following code at the end of the **onCreate** method in the **MainActivity** class:

        notifications.subscribeToCategories(notifications.retrieveCategories());

    This makes sure that every time the app starts it retrieves the categories from local storage and requests a registeration for these categories. 

2. Then update the `onStart()` method of the `MainActivity` class as follows:

    @Override
 protected void onStart() {

        super.onStart();
     isVisible = true;

     Set<String> categories = notifications.retrieveCategories();

     CheckBox world = (CheckBox) findViewById(R.id.worldBox);
     world.setChecked(categories.contains("world"));
     CheckBox politics = (CheckBox) findViewById(R.id.politicsBox);
     politics.setChecked(categories.contains("politics"));
     CheckBox business = (CheckBox) findViewById(R.id.businessBox);
     business.setChecked(categories.contains("business"));
     CheckBox technology = (CheckBox) findViewById(R.id.technologyBox);
     technology.setChecked(categories.contains("technology"));
     CheckBox science = (CheckBox) findViewById(R.id.scienceBox);
     science.setChecked(categories.contains("science"));
     CheckBox sports = (CheckBox) findViewById(R.id.sportsBox);
     sports.setChecked(categories.contains("sports"));
    }

    This updates the main activity based on the status of previously saved categories.


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
1. In Android Studio, build the app and start it on a device or emulator.

    Note that the app UI provides a set of toggles that lets you choose the categories to subscribe to.

2. Enable one or more categories toggles, then click **Subscribe**.

    The app converts the selected categories into tags and requests a new device registration for the selected tags from the notification hub. The registered categories are returned and displayed in a toast notification.

3. Send a new notification by running the .NET Console app.  Alternatively, you can send tagged template notifications using the debug tab of your notification hub in the [Azure Classic Portal](https://manage.windowsazure.com).

    Notifications for the selected categories appear as toast notifications.


## Next steps
In this tutorial we learned how to broadcast breaking news by category. Consider completing one of the following tutorials that highlight other advanced Notification Hubs scenarios:

* [Use Notification Hubs to broadcast localized breaking news](/manage/services/notification-hubs/breaking-news-localized-dotnet/)

    Learn how to expand the breaking news app to enable sending localized notifications.


<!-- Images. -->

[A1]: ./media/notification-hubs-aspnet-backend-android-breaking-news/android-breaking-news1.PNG

<!-- URLs.-->

[get-started]: notification-hubs-android-get-started.md
[Use Notification Hubs to broadcast localized breaking news]: /manage/services/notification-hubs/breaking-news-localized-dotnet/
[Notify users with Notification Hubs]: /manage/services/notification-hubs/notify-users
[Mobile Service]: /develop/mobile/tutorials/get-started/
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Store]: http://msdn.microsoft.com/library/jj927172.aspx
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Azure Classic Portal]: https://manage.windowsazure.com
[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
