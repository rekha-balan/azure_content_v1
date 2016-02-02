<properties
    pageTitle="Azure Notification Hubs Secure Push"
    description="Learn how to send secure push notifications to an Android app from Azure. Code samples written in Java and C#."
    documentationCenter="android"
    authors="wesmc7777"
    manager="dwrede"
    editor=""
    services="notification-hubs"/>

<tags
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="10/05/2015" 
    ms.author="wesmc"/>

# Azure Notification Hubs Secure Push
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Windows Universal](notification-hubs-aspnet-backend-windows-dotnet-secure-push.md)
> * [iOS](notification-hubs-aspnet-backend-ios-secure-push.md)
> * [Android](notification-hubs-aspnet-backend-android-secure-push.md)
> 
> 
## Overview
Push notification support in Microsoft Azure enables you to access an easy-to-use, multiplatform, scaled-out push infrastructure, which greatly simplifies the implementation of push notifications for both consumer and enterprise applications for mobile platforms.

Due to regulatory or security constraints, sometimes an application might want to include something in the notification that cannot be transmitted through the standard push notification infrastructure. This tutorial describes how to achieve the same experience by sending sensitive information through a secure, authenticated connection between the client device and the app backend.

At a high level, the flow is as follows:

1. The app back-end:   * Stores secure payload in back-end database.
* Sends the ID of this notification to the device (no secure information is sent).


2. The app on the device, when receiving the notification:   * The device contacts the back-end requesting the secure payload.
* The app can show the payload as a notification on the device.



It is important to note that in the preceding flow (and in this tutorial), we assume that the device stores an authentication token in local storage, after the user logs in. This guarantees a completely seamless experience, as the device can retrieve the notification’s secure payload using this token. If your application does not store authentication tokens on the device, or if these tokens can be expired, the device app, upon receiving the notification should display a generic notification prompting the user to launch the app. The app then authenticates the user and shows the notification payload.

This Secure Push tutorial shows how to send a push notification securely. The tutorial builds on the [Notify Users](notification-hubs-aspnet-backend-android-notify-users.md) tutorial, so you should complete the steps in that tutorial first.

> [!NOTE]
> This tutorial assumes that you have created and configured your notification hub as described in [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md).
> 
> 
## WebAPI Project

1. In Visual Studio, open the **AppBackend** project that you created in the **Notify Users** tutorial.
2. In Notifications.cs, replace the whole **Notifications** class with the following code. Be sure to replace the placeholders with your connection string (with full access) for your notification hub, and the hub name. You can obtain these values from the [Azure Classic Portal](http://manage.windowsazure.com). This module now represents the different secure notifications that will be sent. In a complete implementation, the notifications will be stored in a database; for simplicity, in this case we store them in memory.

		public class Notification
	    {
	        public int Id { get; set; }
	        public string Payload { get; set; }
	        public bool Read { get; set; }
	    }
    
    
	    public class Notifications
	    {
	        public static Notifications Instance = new Notifications();
	        
	        private List<Notification> notifications = new List<Notification>();
	
	        public NotificationHubClient Hub { get; set; }
	
	        private Notifications() {
	            Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}", 	"{hub name}");
	        }

	        public Notification CreateNotification(string payload)
	        {
	            var notification = new Notification() {
                Id = notifications.Count,
                Payload = payload,
                Read = false
            	};

            	notifications.Add(notification);

            	return notification;
	        }

	        public Notification ReadNotification(int id)
	        {
	            return notifications.ElementAt(id);
	        }
	    }

20. In NotificationsController.cs, replace the code inside the **NotificationsController** class definition with the following code. This component implements a way for the device to retrieve the notification securely, and also provides a way (for the purposes of this tutorial) to trigger a secure push to your devices. Note that when sending the notification to the notification hub, we only send a raw notification with the ID of the notification (and no actual message):

		public NotificationsController()
        {
            Notifications.Instance.CreateNotification("This is a secure notification!");
        }

        // GET api/notifications/id
        public Notification Get(int id)
        {
            return Notifications.Instance.ReadNotification(id);
        }

        public async Task<HttpResponseMessage> Post()
        {
            var secureNotificationInTheBackend = Notifications.Instance.CreateNotification("Secure confirmation.");
            var usernameTag = "username:" + HttpContext.Current.User.Identity.Name;

            // windows
            var rawNotificationToBeSent = new Microsoft.Azure.NotificationHubs.WindowsNotification(secureNotificationInTheBackend.Id.ToString(),
                            new Dictionary<string, string> {
                                {"X-WNS-Type", "wns/raw"}
                            });
            await Notifications.Instance.Hub.SendNotificationAsync(rawNotificationToBeSent, usernameTag);

            // apns
            await Notifications.Instance.Hub.SendAppleNativeNotificationAsync("{\"aps\": {\"content-available\": 1}, \"secureId\": \"" + secureNotificationInTheBackend.Id.ToString() + "\"}", usernameTag);

            // gcm
            await Notifications.Instance.Hub.SendGcmNativeNotificationAsync("{\"data\": {\"secureId\": \"" + secureNotificationInTheBackend.Id.ToString() + "\"}}", usernameTag);


            return Request.CreateResponse(HttpStatusCode.OK);
        }


Note that the `Post` method now does not send a toast notification. It sends a raw notification that contains only the notification ID, and not any sensitive content. Also, make sure to comment the send operation for the platforms for which you do not have credentials configured on your notification hub, as they will result in errors.

21. Now we will re-deploy this app to an Azure Website in order to make it accessible from all devices. Right-click on the **AppBackend** project and select **Publish**.

24. Select Azure Website as your publish target. Log in with your Azure account and select an existing or new Website, and make a note of the **destination URL** property in the **Connection** tab. We will refer to this URL as your *backend endpoint* later in this tutorial. Click **Publish**.

## Modify the Android project
Now that you modified your app back-end to send just the *id* of a notification, you have to change your Android app to handle that notification and call back your back-end to retrieve the secure message to be displayed.
To achieve this goal, you have to make sure that your Android app knows how to authenticate itself with your back-end when it receives the push notifications.

We will now modify the *login* flow in order to save the authentication header value in the shared preferences of your app. Analogous mechanisms can be used to store any authentication token (e.g. OAuth tokens) that the app will have to use without requiring user credentials.

1. In your Android app project, add the following constants at the top of the **MainActivity** class:

        public static final String NOTIFY_USERS_PROPERTIES = "NotifyUsersProperties";
     public static final String AUTHORIZATION_HEADER_PROPERTY = "AuthorizationHeader";
2. Still in the **MainActivity** class, update the `getAuthorizationHeader()` method to contain the following code:

        private String getAuthorizationHeader() throws UnsupportedEncodingException {
         EditText username = (EditText) findViewById(R.id.usernameText);
         EditText password = (EditText) findViewById(R.id.passwordText);
         String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
         basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);

         SharedPreferences sp = getSharedPreferences(NOTIFY_USERS_PROPERTIES, Context.MODE_PRIVATE);
         sp.edit().putString(AUTHORIZATION_HEADER_PROPERTY, basicAuthHeader).commit();

         return basicAuthHeader;
     }
3. Add the following `import` statements at the top of the **MainActivity** file:

        import android.content.SharedPreferences;


Now we will change the handler that is called when the notification is received.

1. In the **MyHandler** class change the `OnReceive()` method to contain:

        public void onReceive(Context context, Bundle bundle) {
         ctx = context;
         String secureMessageId = bundle.getString("secureId");
         retrieveNotification(secureMessageId);
     }
2. Then add the `retrieveNotification()` method, replacing the placeholder `{back-end endpoint}` with the back-end endpoint obtained while deploying your back-end:

        private void retrieveNotification(final String secureMessageId) {
         SharedPreferences sp = ctx.getSharedPreferences(MainActivity.NOTIFY_USERS_PROPERTIES, Context.MODE_PRIVATE);
         final String authorizationHeader = sp.getString(MainActivity.AUTHORIZATION_HEADER_PROPERTY, null);

         new AsyncTask<Object, Object, Object>() {
             @Override
             protected Object doInBackground(Object... params) {
                 try {
                     HttpUriRequest request = new HttpGet("{back-end endpoint}/api/notifications/"+secureMessageId);
                     request.addHeader("Authorization", "Basic "+authorizationHeader);
                     HttpResponse response = new DefaultHttpClient().execute(request);
                     if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                         Log.e("MainActivity", "Error retrieving secure notification" + response.getStatusLine().getStatusCode());
                         throw new RuntimeException("Error retrieving secure notification");
                     }
                     String secureNotificationJSON = EntityUtils.toString(response.getEntity());
                     JSONObject secureNotification = new JSONObject(secureNotificationJSON);
                     sendNotification(secureNotification.getString("Payload"));
                 } catch (Exception e) {
                     Log.e("MainActivity", "Failed to retrieve secure notification - " + e.getMessage());
                     return e;
                 }
                 return null;
             }
         }.execute(null, null, null);
     }



This method calls your app back-end to retrieve the notification content using the credentials stored in the shared preferences and displays it as a normal notification. The notification looks to the app user exactly like any other push notification.

Note that it is preferable to handle the cases of missing authentication header property or rejection by the back-end. The specific handling of these cases depend mostly on your target user experience. One option is to display a notification with a generic prompt for the user to authenticate to retrieve the actual notification.

## Run the Application
To run the application, do the following:

1. Make sure **AppBackend** is deployed in Azure. If using Visual Studio, run the **AppBackend** Web API application. An ASP.NET web page is displayed.

2. In Eclipse, run the app on a physical Android device or the emulator.

3. In the Android app UI, enter a username and password. These can be any string, but they must be the same value.

4. In the Android app UI, click **Log in**. Then click **Send push**.


