<properties
    pageTitle="Get started with Azure Notification Hubs for Android apps | Microsoft Azure"
    description="In this tutorial, you learn how to use Azure Notification Hubs to push notifications to Android devices."
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
    ms.topic="hero-article"
    ms.date="12/15/2015"
    ms.author="wesmc"/>

# Get started with Notification Hubs for Android apps
> [AZURE.SELECTOR]
- [Windows Runtime 8.1 universal](../articles/notification-hubs/notification-hubs-windows-store-dotnet-get-started.md)
- [Windows Phone Silverlight 8.x](../articles/notification-hubs/notification-hubs-windows-phone-get-started.md)
- [iOS](../articles/notification-hubs/notification-hubs-ios-get-started.md)
- [Android](../articles/notification-hubs/notification-hubs-android-get-started.md)
- [Kindle](../articles/notification-hubs/notification-hubs-kindle-get-started.md)
- [Baidu](../articles/notification-hubs/notification-hubs-baidu-get-started.md)
- [Xamarin.iOS](../articles/notification-hubs/partner-xamarin-notification-hubs-ios-get-started.md)
- [Xamarin.Android](../articles/notification-hubs/partner-xamarin-notification-hubs-android-get-started.md)
- [Chrome](../articles/notification-hubs/notification-hubs-chrome-get-started.md)


## Overview
This tutorial shows you how to use Azure Notification Hubs to send push notifications to an Android application.
You'll create a blank Android app that receives push notifications by using Google Cloud Messaging (GCM). When you're finished, you'll be able to use your notification hub to broadcast push notifications to all the devices running your app.

This tutorial demonstrates the simple broadcast scenario in using Notification Hubs. Be sure to follow along with the next tutorial to see how to use Notification Hubs to address specific users and groups of devices.

## Before you begin

The goal of this topic is to help you get started using Notification Hubs quickly as possible. This topic presents a very simple broadcast scenario example in order to concentrate on the basic concepts for Notification Hubs.

If you are already familiar with Notification Hubs, you may want to select another topic from the left-navigation or see the relevant links in [Next steps](#next-steps).

We do take your feedback seriously. If you have any difficulties completing this topic, or recommendations for improving this content, we would appreciate your feedback at the bottom of the page.


The completed code for this tutorial can be found on GitHub [here](https://github.com/Azure/azure-notificationhubs-samples/tree/master/Android/GetStarted). 

## Prerequisites
This tutorial requires the following:

* Android Studio, which you can download from <a href="http://go.microsoft.com/fwlink/?LinkId=389797">the Android site</a>.
* An active Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdocumentation%2Farticles%2Fnotification-hubs-android-get-started%2F).

Completing this tutorial is a prerequisite for all other Notification Hubs tutorials for Android apps.

## Create a project that supports Google Cloud Messaging

1. Navigate to the [Google Cloud Console](https://console.developers.google.com/project), sign in with your Google account credentials. 
 
2. Click **Create Project**, type a project name, then click **Create**. If requested, carry out the SMS Verification, and click **Create** again.

   	![](./media/mobile-services-enable-google-cloud-messaging/mobile-services-google-new-project.png)   

	 Type in your new **Project name** and click **Create project**.

3. Make a note of the project number in the **Projects** section. You will need to set this value as the *PROJECT_ID* variable in the client.

4. In the project dashboard, click **Use Google APIs** > **Cloud Messaging for Android**, then on the next page click **Enable API**. 

5. In the API Manager, Click **Credentials** > **Add Credential** > **API Key**. 

   	![](./media/mobile-services-enable-google-cloud-messaging/mobile-services-google-create-server-key.png)

6. In **Create a new key**, click **Server key**, type a name for your key, then click **Create**.

7. Make a note of the **API KEY** value.

	You will use this API key value to enable Azure to authenticate with GCM and send push notifications on behalf of your app.



## Configure a new notification hub


1. Log on to the [Azure Classic Portal](https://manage.windowsazure.com/), and then click **+NEW** at the bottom of the screen.

2. Click on **App Services**, then **Service Bus**, then **Notification Hub**, then **Quick Create**.

   	![](./media/notification-hubs-portal-create-new-hub/notification-hubs-create-from-portal.png)


3. Enter a **Notification Hub Name**. Select your desired **region** and **subscription**. 
 
	If you already have a service bus namespace that you want create the hub in, select your **Namespace Name**.  Otherwise, you can use the default **Namespace Name** which will be created based on the hub name as long as the namespace name is available. 

	Click **Create a new Notification Hub**.

   	![Set notification hub properties](./media/notification-hubs-portal-create-new-hub/notification-hubs-create-from-portal2.png)

4. Once the namespace and notification hub are created, your namespaces in service bus will be displayed. Click the namespace that you just created your hub in (usually ***notification hub name*-ns**). 

5. On your namespace page, click the **Notification Hubs** tab at the top, and then click on the notification hub you just created. This will open the dashboard for your new notification hub.

6. On the dashboard for your new hub click **View Connection String**. Take note of the two connection strings. You will use these later.

   	![](./media/notification-hubs-portal-create-new-hub/notification-hubs-view-connection-strings.png)

	![](./media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings.png)



&emsp;&emsp;7.   Click the **Configure** tab at the top, enter the **API Key** value you obtained in the previous section, and then click **Save**.

&emsp;&emsp;![](./media/notification-hubs-android-get-started/notification-hub-configure-android.png)

Your notification hub is now configured to work with GCM, and you have the connection strings to both register your app to receive notifications and to send push notifications.

## <a id="connecting-app"></a>Connect your app to the notification hub
### Create a new Android project
1. In Android Studio, start a new Android Studio project.

       ![][13]
2. Choose the **Phone and Tablet** form factor and the **Minimum SDK** that you want to support. Then click **Next**.

       ![][14]
3. Choose **Blank Activity** for the main activity, click **Next**, and then click **Finish**.


### Add Google Play services to the project
1. Open the Android SDK Manager by clicking the icon on the toolbar of Android Studio or by clicking **Tools** -> **Android** -> **SDK Manager** on the menu. Locate the target version of the Android SDK that is used in your project , open it by clicking **Show Package Details**, and choose **Google APIs**, if it is not already installed.

2. Click the **SDK Tools** tab. If you haven't already installed Google Play Service, click **Google Play Services** as shown below. Then click **Apply** to install. 
 
	Note the SDK path, for use in a later step. 

   	![](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)


3. Open the **build.gradle** file in the app directory.

	![](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-add-google-play-dependency.png)

4. Add this line under *dependencies*: 

   		compile 'com.google.android.gms:play-services-base:6.5.87'

5. Under *defaultConfig*, change *minSdkVersion* to 9.
 
6. Click the **Sync Project with Gradle Files** icon in the tool bar.

7. Open **AndroidManifest.xml** and add this tag to the *application* tag.

        <meta-data android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
 






### Add code
1. Download the notification-hubs-0.4.jar file from the **Files** tab of the [Notification-Hubs-Android-SDK on Bintray](https://bintray.com/microsoftazuremobile/SDK/Notification-Hubs-Android-SDK/0.4). Drag the file directly into the **libs** folder in the Project View window of Android Studio. Then right click the file and click **Add as Library**.

2. In the Build.Gradle file for the **app**, add the following line in the **dependencies** section.

        compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'

    Add the following repository after the **dependencies** section.

        repositories {
         maven {
             url "http://dl.bintray.com/microsoftazuremobile/SDK"
         }
     }
3. Set up the application to obtain a registration ID from GCM, and use it to register the app instance to the notification hub.

    In the AndroidManifest.xml file, add the following permissions below the  `</application>` tag. Make sure to replace `<your package>` with the package name shown at the top of the AndroidManifest.xml file (`com.example.testnotificationhubs` in this example).

        <uses-permission android:name="android.permission.INTERNET"/>
     <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
     <uses-permission android:name="android.permission.WAKE_LOCK"/>
     <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

     <permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
     <uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>
4. In the **MainActivity** class, add the following `import` statements above the class declaration.

        import android.app.AlertDialog;
     import android.content.DialogInterface;
     import android.os.AsyncTask;
     import com.google.android.gms.gcm.*;
     import com.microsoft.windowsazure.messaging.*;
     import com.microsoft.windowsazure.notifications.NotificationsManager;
     import android.widget.Toast;




1. Add the following private members at the top of the class.

        private String SENDER_ID = "<your project number>";
     private GoogleCloudMessaging gcm;
     private NotificationHub hub;
     private String HubName = "<Enter Your Hub Name>";
     private String HubListenConnectionString = "<Your default listen connection string>";
     private static Boolean isVisible = false;



    Make sure to update the three placeholders:
    * **SENDER_ID**: Set `SENDER_ID` to the project number that you obtained earlier from the project that you created in  the [Google Cloud Console](http://cloud.google.com/console).
    * **HubListenConnectionString**: Set `HubListenConnectionString` to the **DefaultListenAccessSignature** connection string for your hub. You can copy that connection string by clicking **View Connection String** on the **Dashboard** tab of your hub on the [Azure Classic Portal].
    * **HubName**: Use the name of your notification hub that appears at the top of the page in Azure for your hub (**not** the full URL). For example, use `"myhub"`.



1. In the **OnCreate** method of the **MainActivity** class, add the following code to register with the notification hub when the activity is created.

        MyHandler.mainActivity = this;
     NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);
     gcm = GoogleCloudMessaging.getInstance(this);
     hub = new NotificationHub(HubName, HubListenConnectionString, this);
     registerWithNotificationHubs();
2. In **MainActivity.java**, add the code below for the **registerWithNotificationHubs()** method. This method reports success after registering with Google Cloud Messaging and the notification hub.

        @SuppressWarnings("unchecked")
     private void registerWithNotificationHubs() {
         new AsyncTask() {
             @Override
             protected Object doInBackground(Object... params) {
                 try {
                     String regid = gcm.register(SENDER_ID);
                 ToastNotify("Registered Successfully - RegId : " +
                     hub.register(regid).getRegistrationId());
                 } catch (Exception e) {
                     ToastNotify("Registration Exception Message - " + e.getMessage());
                     return e;
                 }
                 return null;
             }
         }.execute(null, null, null);
     }



1. Add the `ToastNotify` method to the activity to display the notification when the app is running and visible. Also override `onStart`, `onPause`, `onResume` and `onStop` to determine whether the activity is visible to display the toast.

        @Override
     protected void onStart() {
         super.onStart();
         isVisible = true;
     }

     @Override
     protected void onPause() {
         super.onPause();
         isVisible = false;
     }

     @Override
     protected void onResume() {
         super.onResume();
         isVisible = true;
     }

     @Override
     protected void onStop() {
         super.onStop();
         isVisible = false;
     }

     public void ToastNotify(final String notificationMessage)
     {
         if (isVisible == true)
             runOnUiThread(new Runnable() {
                 @Override
                 public void run() {
                     Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
                 }
             });
     }
2. Because Android does not display notifications, you must write your own receiver. In **AndroidManifest.xml**, add the following element inside the `<application>` element.

   > [!NOTE]
> Replace the placeholder with your package name.
> 
> 
        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
         android:permission="com.google.android.c2dm.permission.SEND">
         <intent-filter>
             <action android:name="com.google.android.c2dm.intent.RECEIVE" />
             <category android:name="<your package name>" />
         </intent-filter>
     </receiver>



1. In the Project View, expand **app** > **src** > **main** > **java**. Right-click your package folder under **java**, click **New**, and then click **Java Class**.

    ![][6]

2. In the **Name** field for the new class, type **MyHandler**, and then click **OK**.


1. Add the following import statements at the top of **MyHandler.java**:

       import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;
    import android.os.Bundle;
    import android.support.v4.app.NotificationCompat;
    import com.microsoft.windowsazure.notifications.NotificationsHandler;



1. Update the class declaration as follows to make `MyHandler` a subclass of `com.microsoft.windowsazure.notifications.NotificationsHandler` as shown below.

       public class MyHandler extends NotificationsHandler {



1. Add the following code for the `MyHandler` class.

   This code overrides the `OnReceive` method, so the handler will pop up a Toast to show notifications that are received. The handler also sends the notification to the Android notification manager by using the `sendNotification()` method.

       public static final int NOTIFICATION_ID = 1;
    private NotificationManager mNotificationManager;
    NotificationCompat.Builder builder;
    Context ctx;

    static public MainActivity mainActivity;

    @Override
    public void onReceive(Context context, Bundle bundle) {
        ctx = context;
        String nhMessage = bundle.getString("message");

        sendNotification(nhMessage);
        mainActivity.ToastNotify(nhMessage);
    }

    private void sendNotification(String msg) {
        mNotificationManager = (NotificationManager)
            ctx.getSystemService(Context.NOTIFICATION_SERVICE);

        PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
            new Intent(ctx, MainActivity.class), 0);

        NotificationCompat.Builder mBuilder =
            new NotificationCompat.Builder(ctx)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Notification Hub Demo")
                .setStyle(new NotificationCompat.BigTextStyle()
                .bigText(msg))
                .setContentText(msg);

        mBuilder.setContentIntent(contentIntent);
        mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
    }
2. In Android Studio on the menu bar, click **Build** > **Rebuild Project** to make sure that no errors are detected.


## Send notifications
You can test receiving notifications in your app by sending notifications in the [Azure Classic Portal](https://manage.windowsazure.com/) via the debug tab on the notification hub, as shown in the screen below.

![][30]



Push notifications are normally sent in a back-end service like Mobile Services or ASP.NET using a compatible library. You can also use the REST API directly to send notification messages if a library is not available for your back-end. 

Here is a list of some other tutorials you may want to review for sending notifications:

- Azure Mobile Services : For an example of how to send notifications from an Azure Mobile Services backend integrated with Notification Hubs, see [Get started with push notifications in Mobile Services].  
- ASP.NET : [Use Notification Hubs to push notifications to users].
- Azure Notification Hub Java SDK: See [How to use Notification Hubs from Java](../articles/notification-hubs/notification-hubs-java-backend-how-to.md) for sending notifications from Java. This has been tested in Eclipse for Android Development.
- PHP: [How to use Notification Hubs from PHP](../articles/notification-hubs/notification-hubs-php-backend-how-to.md).


In the next section of the tutorial, you will learn how to use the [Notification Hub REST interface](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx) to send the notification message directly in your app. All registered devices receive the notification sent by any device.  




## (Optional) Send notifications from the app
1. In Android Studio Project View, expand **App** > **src** > **main** > **res** > **layout**. Open the **activity_main.xml** layout file and click the **Text** tab to update the text contents of the file. Update it with the code below, which adds new `Button` and `EditText` controls for sending notification messages to the notification hub. Add this code at the bottom, just before `</RelativeLayout>`.

        <Button
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"
     android:text="@string/send_button"
     android:id="@+id/sendbutton"
     android:layout_centerVertical="true"
     android:layout_centerHorizontal="true"
     android:onClick="sendNotificationButtonOnClick" />

     <EditText
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:id="@+id/editTextNotificationMessage"
     android:layout_above="@+id/sendbutton"
     android:layout_centerHorizontal="true"
     android:layout_marginBottom="42dp"
     android:hint="@string/notification_message_hint" />
2. Add this line to the **build.gradle** file under `android`

        useLibrary 'org.apache.http.legacy'
3. In Android Studio Project View, expand **App** > **src** > **main** > **res** > **values**. Open the **strings.xml** file and add the string values that are referenced by the new `Button` and `EditText` controls. Add these at the bottom of the file, just before `</resources>`.

        <string name="send_button">Send Notification</string>
     <string name="notification_message_hint">Enter notification message text</string>



1. In your **MainActivity.java** file, add the following `import` statements above the `MainActivity` class.

        import java.net.URLEncoder;
     import javax.crypto.Mac;
     import javax.crypto.spec.SecretKeySpec;

     import android.util.Base64;
     import android.view.View;
     import android.widget.EditText;

     import org.apache.http.HttpResponse;
     import org.apache.http.client.HttpClient;
     import org.apache.http.client.methods.HttpPost;
     import org.apache.http.entity.StringEntity;
     import org.apache.http.impl.client.DefaultHttpClient;



1. In your **MainActivity.java** file, add the following members at the top of the `MainActivity` class.

    Update `HubFullAccess` with the **DefaultFullSharedAccessSignature** connection string for your hub. This connection string can be copied from the [Azure Classic Portal](https://manage.windowsazure.com/) by clicking **View Connection String** on the **Dashboard** tab for your notification hub.

        private String HubEndpoint = null;
     private String HubSasKeyName = null;
     private String HubSasKeyValue = null;
     private String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";
2. Your activity holds the hub name and the full shared access connection string for the hub. You must create a Software Access Signature (SaS) token to authenticate a POST request to send messages to your notification hub. This is done by parsing the key data from the connection string and then creating the SaS token, as mentioned in the [Common Concepts](http://msdn.microsoft.com/library/azure/dn495627.aspx) REST API reference.

    In **MainActivity.java**, add the following method to the `MainActivity` class to parse your connection string.

        /**
      * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx
      * to parse the connection string so a SaS authentication token can be
      * constructed.
      *
      * @param connectionString This must be the DefaultFullSharedAccess connection
      *                         string for this example.
      */
     private void ParseConnectionString(String connectionString)
     {
         String[] parts = connectionString.split(";");
         if (parts.length != 3)
             throw new RuntimeException("Error parsing connection string: "
                     + connectionString);

         for (int i = 0; i < parts.length; i++) {
             if (parts[i].startsWith("Endpoint")) {
                 this.HubEndpoint = "https" + parts[i].substring(11);
             } else if (parts[i].startsWith("SharedAccessKeyName")) {
                 this.HubSasKeyName = parts[i].substring(20);
             } else if (parts[i].startsWith("SharedAccessKey")) {
                 this.HubSasKeyValue = parts[i].substring(16);
             }
         }
     }
3. In **MainActivity.java**, add the following method to the `MainActivity` class to create a SaS authentication token.

        /**
      * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx to
      * construct a SaS token from the access key to authenticate a request.
      *
      * @param uri The unencoded resource URI string for this operation. The resource
      *            URI is the full URI of the Service Bus resource to which access is
      *            claimed. For example,
      *            "http://<namespace>.servicebus.windows.net/<hubName>"
      */
     private String generateSasToken(String uri) {

         String targetUri;
         try {
             targetUri = URLEncoder
                     .encode(uri.toString().toLowerCase(), "UTF-8")
                     .toLowerCase();

             long expiresOnDate = System.currentTimeMillis();
             int expiresInMins = 60; // 1 hour
             expiresOnDate += expiresInMins * 60 * 1000;
             long expires = expiresOnDate / 1000;
             String toSign = targetUri + "\n" + expires;

             // Get an hmac_sha1 key from the raw key bytes
             byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
             SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");

             // Get an hmac_sha1 Mac instance and initialize with the signing key
             Mac mac = Mac.getInstance("HmacSHA256");
             mac.init(signingKey);

             // Compute the hmac on input data bytes
             byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));

             // Using android.util.Base64 for Android Studio instead of
             // Apache commons codec
             String signature = URLEncoder.encode(
                     Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");

             // Construct authorization string
             String token = "SharedAccessSignature sr=" + targetUri + "&sig="
                     + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
             return token;
         } catch (Exception e) {
             DialogNotify("Exception Generating SaS",e.getMessage().toString());
         }

         return null;
     }



1. In **MainActivity.java**, add the following method to the `MainActivity` class to handle the **Send Notification** button click and send the notification message to the hub by using the REST API.

        /**
      * Send Notification button click handler. This method parses the
      * DefaultFullSharedAccess connection string and generates a SaS token. The
      * token is added to the Authorization header on the POST request to the
      * notification hub. The text in the editTextNotificationMessage control
      * is added as the JSON body for the request to add a GCM message to the hub.
      *
      * @param v
      */
     public void sendNotificationButtonOnClick(View v) {
         EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
         final String json = "{\"data\":{\"message\":\"" + notificationText.getText().toString() + "\"}}";

         new Thread()
         {
             public void run()
             {
                 try
                 {
                     HttpClient client = new DefaultHttpClient();

                     // Based on reference documentation...
                     // http://msdn.microsoft.com/library/azure/dn223273.aspx
                     ParseConnectionString(HubFullAccess);
                     String url = HubEndpoint + HubName + "/messages/?api-version=2015-01";
                     HttpPost post = new HttpPost(url);

                     // Authenticate the POST request with the SaS token
                     post.setHeader("Authorization", generateSasToken(url));

                     // JSON content for GCM
                     post.setHeader("Content-Type", "application/json;charset=utf-8");

                     // Notification format should be GCM
                     post.setHeader("ServiceBusNotification-Format", "gcm");
                     post.setEntity(new StringEntity(json));

                     HttpResponse response = client.execute(post);
                 }
                 catch(Exception e)
                 {
                     DialogNotify("Exception",e.getMessage().toString());
                 }
             }
         }.start();
     }




## Test your app
#### Emulator testing
If you want to test on an emulator, make sure that your emulator image supports the Google API level that you choose for your app. If your image doesn't support the Google APIs, you will end up with the **SERVICE\_NOT\_AVAILABLE** exception.

Also make sure that you have added your Google account to your running emulator under **Settings** > **Accounts**. Otherwise, your attempts to register with GCM may result in the **AUTHENTICATION\_FAILED** exception.

#### Testing the app
1. Run the app and notice that the registration ID is reported for a successful registration.

       ![][18]
2. Enter a notification message to be sent to all Android devices that have registered with the hub.

       ![][19]
3. Press **Send Notification**. Any devices that have the app running will show an `AlertDialog` with the notification message. Devices that don't have the app running but were previously registered for the notifications will receive a notification added to the notification manager. Notifications can be viewed by swiping down from the upper-left corner.

       ![][21]


## Next steps
In this simple example, you sent broadcast notifications to all your Windows devices using the portal or a console app. We recommend the [Use Notification Hubs to push notifications to users](notification-hubs-aspnet-backend-android-notify-users.md) tutorial as the next step. It will show you how to send notifications from an ASP.NET backend using tags to target specific users.

If you want to segment your users by interest groups, see [Use Notification Hubs to send breaking news](notification-hubs-aspnet-backend-android-breaking-news.md). 

To learn more general information about Notification Hubs, see [Notification Hubs Guidance](http://msdn.microsoft.com/library/jj927170.aspx).

<!-- Images. -->

[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png

[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png

[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png

[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png

[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png

[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png

[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png


<!-- URLs. -->

[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md  
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: http://go.microsoft.com/fwlink/?LinkId=389800
[Azure Classic Portal]: https://manage.windowsazure.com/
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-android-notify-users.md
[Use Notification Hubs to send breaking news]: notification-hubs-aspnet-backend-android-breaking-news.md