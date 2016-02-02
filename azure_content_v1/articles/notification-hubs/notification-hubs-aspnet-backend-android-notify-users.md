<properties
    pageTitle="Azure Notification Hubs Notify Users for Android with .NET backend"
    description="Learn how to send push notifications to users in Azure. Code samples written in Java for Android"
    documentationCenter="android"
    services="notification-hubs"
    authors="wesmc7777"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="wesmc"/>

# Azure Notification Hubs Notify Users for Android with .NET backend
> [AZURE.SELECTOR]
- [Windows Runtime 8.1 universal](../articles/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-notify-users.md)
- [iOS](../articles/notification-hubs/notification-hubs-aspnet-backend-ios-notify-users.md)
- [Android](../articles/notification-hubs/notification-hubs-aspnet-backend-android-notify-users.md)



## Overview
Push notification support in Azure enables you to access an easy-to-use, multiplatform, and scaled-out push infrastructure, which greatly simplifies the implementation of push notifications for both consumer and enterprise applications for mobile platforms. This tutorial shows you how to use Azure Notification Hubs to send push notifications to a specific app user on a specific device. An ASP.NET WebAPI backend is used to authenticate clients and to generate notifications, as shown in the guidance topic [Registering from your app backend](notification-hubs-registration-management.md#registration-management-from-a-backend). This tutorial builds on the notification hub that you created in the [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md) tutorial.

> [!NOTE]
> This tutorial assumes that you have created and configured your notification hub as described in [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md).
> 
> 
## Create the WebAPI Project

A new ASP.NET WebAPI backend will be created in the sections that follow and it will have three main purposes:

1. **Authenticating Clients**: A message handler will be added later to authenticate client requests and associate the user with the request.
2. **Client Notification Registrations**: Later, you will add a controller to handle new registrations for a client device to receive notifications. The authenticated user name will automatically be added to the registration as a [tag](https://msdn.microsoft.com/library/azure/dn530749.aspx).
3. **Sending Notifications to Clients**: Later, you will also add a controller to provide a way for a user to trigger a secure push to devices and clients associated with the tag. 

The following steps show how to create the new ASP.NET WebAPI backend: 


> [AZURE.NOTE] **Important**: Before starting this tutorial, please ensure that you have installed the latest version of the NuGet Package Manager. To check, start Visual Studio. From the **Tools** menu, click **Extensions and Updates**. Search for **NuGet Package Manager for Visual Studio 2013**, and make sure you have version 2.8.50313.46 or later. If not, please uninstall, then reinstall the NuGet Package Manager.
> 
> ![][B4]

> [AZURE.NOTE] Make sure you have installed the Visual Studio [Azure SDK](https://azure.microsoft.com/downloads/) for website deployment.

1. Start Visual Studio or Visual Studio Express. Click **Server Explorer** and sign in to your Azure account. Visual Studio will need you signed in to create the web site resources on your account.
2. In Visual Studio, click **File**, then click **New**, then **Project**, expand **Templates**, **Visual C#**, then click **Web** and **ASP.NET Web Application**, type the name **AppBackend**, and then click **OK**. 
	
	![][B1]

3. In the **New ASP.NET Project** dialog, click **Web API**, then click **OK**.

	![][B2]

4. In the **Configure Microsoft Azure Web App** dialog, choose a subscription, and an **App Service plan** you have already created. You can also choose **Create a new app service plan** and create one from the dialog. You do not need a database for this tutorial. Once you have selected your app service plan, click **OK** to create the project.

	![][B5]



## Authenticating Clients to the WebAPI Backend

In this section, you will create a new message handler class named **AuthenticationTestHandler** for the new backend. This class is derived from [DelegatingHandler](https://msdn.microsoft.com/library/system.net.http.delegatinghandler.aspx) and added as a message handler so it can process all requests coming into the backend. 



1. In Solution Explorer, right-click the **AppBackend** project, click **Add**, then click **Class**. Name the new class **AuthenticationTestHandler.cs**, and click **Add** to generate the class. This class will be used to authenticate users using *Basic Authentication* for simplicity. Note that your app can use any authentication scheme.

2. In AuthenticationTestHandler.cs, add the following `using` statements:

        using System.Net.Http;
        using System.Threading;
        using System.Security.Principal;
        using System.Net;
        using System.Web;

3. In AuthenticationTestHandler.cs, replacing the `AuthenticationTestHandler` class definition with the following code. 

	This handler will authorize the request when the following three conditions are all true:
	* The request included an *Authorization* header. 
	* The request uses *basic* authentication. 
	* The user name string and the password string are the same string.

	Otherwise, the request will be rejected. This is not a true authentication and authorization approach. It is just a very simple example for this tutorial.

	If the request message is authenticated and authorized by the `AuthenticationTestHandler`, then the basic authentication user will be attached to the current request on the [HttpContext](https://msdn.microsoft.com/library/system.web.httpcontext.current.aspx). User information in the HttpContext will be used by another controller (RegisterController) later to add a [tag](https://msdn.microsoft.com/library/azure/dn530749.aspx) to the notification registration request.

		public class AuthenticationTestHandler : DelegatingHandler
	    {
	        protected override Task<HttpResponseMessage> SendAsync(
	        HttpRequestMessage request, CancellationToken cancellationToken)
	        {
	            var authorizationHeader = request.Headers.GetValues("Authorization").First();
	
	            if (authorizationHeader != null && authorizationHeader
	                .StartsWith("Basic ", StringComparison.InvariantCultureIgnoreCase))
	            {
	                string authorizationUserAndPwdBase64 =
	                    authorizationHeader.Substring("Basic ".Length);
	                string authorizationUserAndPwd = Encoding.Default
	                    .GetString(Convert.FromBase64String(authorizationUserAndPwdBase64));
	                string user = authorizationUserAndPwd.Split(':')[0];
	                string password = authorizationUserAndPwd.Split(':')[1];
	
	                if (verifyUserAndPwd(user, password))
	                {
	                    // Attach the new principal object to the current HttpContext object
	                    HttpContext.Current.User =
	                        new GenericPrincipal(new GenericIdentity(user), new string[0]);
	                    System.Threading.Thread.CurrentPrincipal =
	                        System.Web.HttpContext.Current.User;
	                }
	                else return Unauthorized();
	            }
	            else return Unauthorized();
	
	            return base.SendAsync(request, cancellationToken);
	        }
	
	        private bool verifyUserAndPwd(string user, string password)
	        {
	            // This is not a real authentication scheme.
	            return user == password;
	        }
	
	        private Task<HttpResponseMessage> Unauthorized()
	        {
	            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
	            var tsc = new TaskCompletionSource<HttpResponseMessage>();
	            tsc.SetResult(response);
	            return tsc.Task;
	        }
	    }

	> [AZURE.NOTE] **Security Note**: The `AuthenticationTestHandler` class does not provide true authentication. It is used only to mimic basic authentication and is not secure. You must implement a secure authentication mechanism in your production applications and services.				

4. Add the following code at the end of the `Register` method in the **App_Start/WebApiConfig.cs** class to register the message handler:

		config.MessageHandlers.Add(new AuthenticationTestHandler());

5. Save your changes.

## Registering for Notifications using the WebAPI Backend

In this section, we will add a new controller to the WebAPI backend to handle requests to register a user and device for notifications using the client library for notification hubs. The controller will add a user tag for the user that was authenticated and attached to the HttpContext by the `AuthenticationTestHandler`. The tag will have the string format, `"username:<actual username>"`.


 

1. In Solution Explorer, right-click the **AppBackend** project and then click **Manage NuGet Packages**.

2. On the left-hand side, click **Online**, and search for **Microsoft.Azure.NotificationHubs** in the **Search** box.

3. In the results list, click **Microsoft Azure Notification Hubs**, and then click **Install**. Complete the installation, then close the NuGet package manager window.

	This adds a reference to the Azure Notification Hubs SDK using the <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet package</a>.

4. We will now create a new class file that represents the different secure notifications that will be sent. In a complete implementation, the notifications are stored in a database. For simplicity, this tutorial stores them in memory. In the Solution Explorer, right-click the **Models** folder, click **Add**, then click **Class**. Name the new class **Notifications.cs**, then click **Add** to generate the class. 

	![][B6]

5. In Notifications.cs, add the following `using` statement at the top of the file:

        using Microsoft.Azure.NotificationHubs;

6. Replace the `Notifications` class definition with the following and make sure to replace the two placeholders with the connection string (with full access) for your notification hub, and the hub name (available at [Azure Classic Portal](http://manage.windowsazure.com)):

		public class Notifications
        {
            public static Notifications Instance = new Notifications();
        
            public NotificationHubClient Hub { get; set; }

            private Notifications() {
                Hub = NotificationHubClient.CreateClientFromConnectionString("<your hub's DefaultFullSharedAccessSignature>", 
																			 "<hub name>");
            }
        }



7. Next we will create a new controller named **RegisterController**. In Solution Explorer, right-click the **Controllers** folder, then click **Add**, then click **Controller**. Click the **Web API 2 Controller -- Empty** item, and then click **Add**. Name the new class **RegisterController**, and then click **Add** again to generate the controller.

	![][B7]

	![][B8]

8. In RegisterController.cs, add the following `using` statements:

        using Microsoft.Azure.NotificationHubs;
		using Microsoft.Azure.NotificationHubs.Messaging;
        using AppBackend.Models;
        using System.Threading.Tasks;
        using System.Web;

9. Add the following code inside the `RegisterController` class definition. Note that in this code, we add a user tag for the user this is attached to the HttpContext. The user was authenticated and attached to the HttpContext by the message filter we added, `AuthenticationTestHandler`. You can also add optional checks to verify that the user has rights to register for the requested tags.

		private NotificationHubClient hub;

        public RegisterController()
        {
            hub = Notifications.Instance.Hub;
        }

        public class DeviceRegistration
        {
            public string Platform { get; set; }
            public string Handle { get; set; }
            public string[] Tags { get; set; }
        }

        // POST api/register
        // This creates a registration id
        public async Task<string> Post(string handle = null)
        {
            string newRegistrationId = null;
            
            // make sure there are no existing registrations for this push handle (used for iOS and Android)
            if (handle != null)
            {
                var registrations = await hub.GetRegistrationsByChannelAsync(handle, 100);

                foreach (RegistrationDescription registration in registrations)
                {
                    if (newRegistrationId == null)
                    {
                        newRegistrationId = registration.RegistrationId;
                    }
                    else
                    {
                        await hub.DeleteRegistrationAsync(registration);
                    }
                }
            }

            if (newRegistrationId == null) 
				newRegistrationId = await hub.CreateRegistrationIdAsync();

            return newRegistrationId;
        }

        // PUT api/register/5
        // This creates or updates a registration (with provided channelURI) at the specified id
        public async Task<HttpResponseMessage> Put(string id, DeviceRegistration deviceUpdate)
        {
            RegistrationDescription registration = null;
            switch (deviceUpdate.Platform)
            {
                case "mpns":
                    registration = new MpnsRegistrationDescription(deviceUpdate.Handle);
                    break;
                case "wns":
                    registration = new WindowsRegistrationDescription(deviceUpdate.Handle);
                    break;
                case "apns":
                    registration = new AppleRegistrationDescription(deviceUpdate.Handle);
                    break;
                case "gcm":
                    registration = new GcmRegistrationDescription(deviceUpdate.Handle);
                    break;
                default:
                    throw new HttpResponseException(HttpStatusCode.BadRequest);
            }

            registration.RegistrationId = id;
            var username = HttpContext.Current.User.Identity.Name;

            // add check if user is allowed to add these tags
            registration.Tags = new HashSet<string>(deviceUpdate.Tags);
            registration.Tags.Add("username:" + username);

            try
            {
                await hub.CreateOrUpdateRegistrationAsync(registration);
            }
            catch (MessagingException e)
            {
                ReturnGoneIfHubResponseIsGone(e);
            }

            return Request.CreateResponse(HttpStatusCode.OK);
        }

        // DELETE api/register/5
        public async Task<HttpResponseMessage> Delete(string id)
        {
            await hub.DeleteRegistrationAsync(id);
            return Request.CreateResponse(HttpStatusCode.OK);
        }

        private static void ReturnGoneIfHubResponseIsGone(MessagingException e)
        {
            var webex = e.InnerException as WebException;
            if (webex.Status == WebExceptionStatus.ProtocolError)
            {
                var response = (HttpWebResponse)webex.Response;
                if (response.StatusCode == HttpStatusCode.Gone)
                    throw new HttpRequestException(HttpStatusCode.Gone.ToString());
            }
        }

10. Save your changes.

## Sending Notifications from the WebAPI Backend

In this section you add a new controller that exposes a way for client devices to send a notification based on the username tag using Azure Notification Hubs Service Management Library in the ASP.NET WebAPI backend.


1. Create another new controller named **NotificationsController**. Create it the same way you created the **RegisterController** in the previous section.

2. In NotificationsController.cs, add the following `using` statements:

        using AppBackend.Models;
        using System.Threading.Tasks;
        using System.Web;

3. Add the following method to the **NotificationsController** class.

	This code send a notification type based on the Platform Notification Service (PNS) `pns` parameter. The value of `to_tag` is used to set the *username* tag on the message. This tag must match a username tag of an active notification hub registration. The notification message is pulled from the body of the POST request and formatted for the target PNS. 

	Depending on the Platform Notification Service (PNS) that your supported devices use to receive notifications, different notifications are supported using different formats. For example on Windows devices, you could use a [toast notification with WNS](https://msdn.microsoft.com/library/windows/apps/br230849.aspx) that isn't directly supported by another PNS. So your backend would need to format the notification into a supported notification for the PNS of devices you plan to support. Then use the appropriate send API on the [NotificationHubClient class](https://msdn.microsoft.com/library/azure/microsoft.azure.notificationhubs.notificationhubclient_methods.aspx)

        public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag)
        {
            var user = HttpContext.Current.User.Identity.Name;
            string[] userTag = new string[2];
            userTag[0] = "username:" + to_tag;
            userTag[1] = "from:" + user;

            Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;
            HttpStatusCode ret = HttpStatusCode.InternalServerError;

            switch (pns.ToLower())
            {
                case "wns":
                    // Windows 8.1 / Windows Phone 8.1
                    var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" + 
                                "From " + user + ": " + message + "</text></binding></visual></toast>";
                    outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
                    break;
                case "apns":
                    // iOS
                    var alert = "{\"aps\":{\"alert\":\"" + "From " + user + ": " + message + "\"}}";
                    outcome = await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);
                    break;
                case "gcm":
                    // Android
                    var notif = "{ \"data\" : {\"message\":\"" + "From " + user + ": " + message + "\"}}";
                    outcome = await Notifications.Instance.Hub.SendGcmNativeNotificationAsync(notif, userTag);
                    break;
            }

            if (outcome != null)
            {
                if (!((outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Abandoned) ||
                    (outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Unknown)))
                {
                    ret = HttpStatusCode.OK;
                }
            }

            return Request.CreateResponse(ret);
        }


4. Press **F5** to run the application and to ensure the accuracy of your work so far. The app should launch a web browser and display the ASP.NET home page. 

##Publish the new WebAPI Backend

1. Now we will deploy this app to an Azure Website in order to make it accessible from all devices. Right-click on the **AppBackend** project and select **Publish**.

2. Select **Microsoft Azure Web Apps** as your publish target.

    ![][B15]

3. Log in with your Azure account and select an existing or new Web App.

    ![][B16]

4. Make a note of the **destination URL** property in the **Connection** tab. We will refer to this URL as your *backend endpoint* later in this tutorial. Click **Publish**.

    ![][B18]


[B1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[B2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[B3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[B4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[B5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[B6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[B7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[B8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[B14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png
[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users15.PNG
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG


## Create the Android Project
The next step is to create the Android application.

1. Follow the [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md) tutorial to create and configure your app to receive push notifications from GCM.

2. Open your **res/layout/activity_main.xml** file, replace the with the following content definitions.

    This adds new EditText controls for logging in as a user. Also a field is added for a username tag that will be part of notifications you send:

        <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
         android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
         android:paddingRight="@dimen/activity_horizontal_margin"
         android:paddingTop="@dimen/activity_vertical_margin"
         android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

     <EditText
         android:id="@+id/usernameText"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:ems="10"
         android:hint="@string/usernameHint"
         android:layout_above="@+id/passwordText"
         android:layout_alignParentEnd="true" />
     <EditText
         android:id="@+id/passwordText"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:ems="10"
         android:hint="@string/passwordHint"
         android:inputType="textPassword"
         android:layout_above="@+id/buttonLogin"
         android:layout_alignParentEnd="true" />
     <Button
         android:id="@+id/buttonLogin"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="@string/loginButton"
         android:onClick="login"
         android:layout_above="@+id/toggleButtonGCM"
         android:layout_centerHorizontal="true"
         android:layout_marginBottom="24dp" />
     <ToggleButton
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:textOn="WNS on"
         android:textOff="WNS off"
         android:id="@+id/toggleButtonWNS"
         android:layout_toLeftOf="@id/toggleButtonGCM"
         android:layout_centerVertical="true" />
     <ToggleButton
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:textOn="GCM on"
         android:textOff="GCM off"
         android:id="@+id/toggleButtonGCM"
         android:checked="true"
         android:layout_centerHorizontal="true"
         android:layout_centerVertical="true" />
     <ToggleButton
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:textOn="APNS on"
         android:textOff="APNS off"
         android:id="@+id/toggleButtonAPNS"
         android:layout_toRightOf="@id/toggleButtonGCM"
         android:layout_centerVertical="true" />
     <EditText
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:id="@+id/editTextNotificationMessageTag"
         android:layout_below="@id/toggleButtonGCM"
         android:layout_centerHorizontal="true"
         android:hint="@string/notification_message_tag_hint" />
     <EditText
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:id="@+id/editTextNotificationMessage"
         android:layout_below="@+id/editTextNotificationMessageTag"
         android:layout_centerHorizontal="true"
         android:hint="@string/notification_message_hint" />
     <Button
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="@string/send_button"
         android:id="@+id/sendbutton"
         android:onClick="sendNotificationButtonOnClick"
         android:layout_below="@+id/editTextNotificationMessage"
         android:layout_centerHorizontal="true" />
     </RelativeLayout>




1. Open your **res/values/strings.xml** file and replace the `send_button` definition with the following lines that redefine the string for the `send_button` and add strings for the other controls:

        <string name="usernameHint">Username</string>
     <string name="passwordHint">Password</string>
     <string name="loginButton">1. Log in</string>
     <string name="send_button">2. Send Notification</string>
     <string name="notification_message_tag_hint">
         Recipient username tag
     </string>

    Your main_activity.xml graphical layout should now look like this:

    ![][A1]

2. Create a new class named **RegisterClient** in the same package as your `MainActivity` class. Use the code below for the new class file.

        import java.io.IOException;
     import java.io.UnsupportedEncodingException;
     import java.util.Set;

     import org.apache.http.HttpResponse;
     import org.apache.http.HttpStatus;
     import org.apache.http.client.ClientProtocolException;
     import org.apache.http.client.HttpClient;
     import org.apache.http.client.methods.HttpPost;
     import org.apache.http.client.methods.HttpPut;
     import org.apache.http.client.methods.HttpUriRequest;
     import org.apache.http.entity.StringEntity;
     import org.apache.http.impl.client.DefaultHttpClient;
     import org.apache.http.util.EntityUtils;
     import org.json.JSONArray;
     import org.json.JSONException;
     import org.json.JSONObject;

     import android.content.Context;
     import android.content.SharedPreferences;
     import android.util.Log;

     public class RegisterClient {
         private static final String PREFS_NAME = "ANHSettings";
         private static final String REGID_SETTING_NAME = "ANHRegistrationId";
         private String Backend_Endpoint;
         SharedPreferences settings;
         protected HttpClient httpClient;
         private String authorizationHeader;

         public RegisterClient(Context context, String backendEnpoint) {
             super();
             this.settings = context.getSharedPreferences(PREFS_NAME, 0);
             httpClient =  new DefaultHttpClient();
             Backend_Endpoint = backendEnpoint + "/api/register";
         }

         public String getAuthorizationHeader() {
             return authorizationHeader;
         }

         public void setAuthorizationHeader(String authorizationHeader) {
             this.authorizationHeader = authorizationHeader;
         }

         public void register(String handle, Set<String> tags) throws ClientProtocolException, IOException, JSONException {
             String registrationId = retrieveRegistrationIdOrRequestNewOne(handle);

             JSONObject deviceInfo = new JSONObject();
             deviceInfo.put("Platform", "gcm");
             deviceInfo.put("Handle", handle);
             deviceInfo.put("Tags", new JSONArray(tags));

             int statusCode = upsertRegistration(registrationId, deviceInfo);

             if (statusCode == HttpStatus.SC_OK) {
                 return;
             } else if (statusCode == HttpStatus.SC_GONE){
                 settings.edit().remove(REGID_SETTING_NAME).commit();
                 registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
                 statusCode = upsertRegistration(registrationId, deviceInfo);
                 if (statusCode != HttpStatus.SC_OK) {
                     Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                     throw new RuntimeException("Error upserting registration");
                 }
             } else {
                 Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                 throw new RuntimeException("Error upserting registration");
             }
         }

         private int upsertRegistration(String registrationId, JSONObject deviceInfo)
                 throws UnsupportedEncodingException, IOException,
                 ClientProtocolException {
             HttpPut request = new HttpPut(Backend_Endpoint+"/"+registrationId);
             request.setEntity(new StringEntity(deviceInfo.toString()));
             request.addHeader("Authorization", "Basic "+authorizationHeader);
             request.addHeader("Content-Type", "application/json");
             HttpResponse response = httpClient.execute(request);
             int statusCode = response.getStatusLine().getStatusCode();
             return statusCode;
         }

         private String retrieveRegistrationIdOrRequestNewOne(String handle) throws ClientProtocolException, IOException {
             if (settings.contains(REGID_SETTING_NAME))
                 return settings.getString(REGID_SETTING_NAME, null);

             HttpUriRequest request = new HttpPost(Backend_Endpoint+"?handle="+handle);
             request.addHeader("Authorization", "Basic "+authorizationHeader);
             HttpResponse response = httpClient.execute(request);
             if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                 Log.e("RegisterClient", "Error creating registrationId: " + response.getStatusLine().getStatusCode());
                 throw new RuntimeException("Error creating Notification Hubs registrationId");
             }
             String registrationId = EntityUtils.toString(response.getEntity());
             registrationId = registrationId.substring(1, registrationId.length()-1);

             settings.edit().putString(REGID_SETTING_NAME, registrationId).commit();

             return registrationId;
         }
     }

    This component implements the REST calls required to contact the app backend, in order to register for push notifications. It also locally stores the *registrationIds* created by the Notification Hub as detailed in [Registering from your app backend](notification-hubs-registration-management.md#registration-management-from-a-backend). Note that it uses an authorization token stored in local storage when you click the **Log in** button.

3. In your `MainActivity` class remove or comment out your private field for `NotificationHub`, and add a field for the `RegisterClient` class and a string for your ASP.NET backend's endpoint. Be sure to replace `<Enter Your Backend Endpoint>` with the your actual backend endpoint obtained previously. For example, `http://mybackend.azurewebsites.net`.


        //private NotificationHub hub;
        private RegisterClient registerClient;
        private static final String BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";


1. In your `MainActivity` class, in the `onCreate` method, remove or comment out the initialization of the `hub` field and the call to the `registerWithNotificationHubs` method. Then add code to initialize an instance of the `RegisterClient` class. The method should contain the following lines:

        @Override
     protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);

         MyHandler.mainActivity = this;
         NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);
         gcm = GoogleCloudMessaging.getInstance(this);

         //hub = new NotificationHub(HubName, HubListenConnectionString, this);
         //registerWithNotificationHubs();

         registerClient = new RegisterClient(this, BACKEND_ENDPOINT);

         setContentView(R.layout.activity_main);
     }
2. In your `MainActivity` class, delete or comment out the entire `registerWithNotificationHubs` method. It will not be used in this tutorial.

3. Add the following `import` statements to your **MainActivity.java** file.

        import android.widget.Button;
     import java.io.UnsupportedEncodingException;
     import android.content.Context;
     import java.util.HashSet;
     import android.widget.Toast;
     import org.apache.http.client.ClientProtocolException;
     import java.io.IOException;
     import org.apache.http.HttpStatus;



1. Then, add the following methods to handle the **Log in** button click event and sending push notifications.

        @Override
     protected void onStart() {
         super.onStart();
         Button sendPush = (Button) findViewById(R.id.sendbutton);
         sendPush.setEnabled(false);
     }

     public void login(View view) throws UnsupportedEncodingException {
         this.registerClient.setAuthorizationHeader(getAuthorizationHeader());

         final Context context = this;
         new AsyncTask<Object, Object, Object>() {
             @Override
             protected Object doInBackground(Object... params) {
                 try {
                     String regid = gcm.register(SENDER_ID);
                     registerClient.register(regid, new HashSet<String>());
                 } catch (Exception e) {
                     DialogNotify("MainActivity - Failed to register", e.getMessage());
                     return e;
                 }
                 return null;
             }

             protected void onPostExecute(Object result) {
                 Button sendPush = (Button) findViewById(R.id.sendbutton);
                 sendPush.setEnabled(true);
                 Toast.makeText(context, "Logged in and registered.",
                         Toast.LENGTH_LONG).show();
             }
         }.execute(null, null, null);
     }

     private String getAuthorizationHeader() throws UnsupportedEncodingException {
         EditText username = (EditText) findViewById(R.id.usernameText);
         EditText password = (EditText) findViewById(R.id.passwordText);
         String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
         basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);
         return basicAuthHeader;
     }

     /**
      * This method calls the ASP.NET WebAPI backend to send the notification message
      * to the platform notification service based on the pns parameter.
      *
      * @param pns     The platform notification service to send the notification message to. Must
      *                be one of the following ("wns", "gcm", "apns").
      * @param userTag The tag for the user who will receive the notification message. This string
      *                must not contain spaces or special characters.
      * @param message The notification message string. This string must include the double quotes
      *                to be used as JSON content.
      */
     public void sendPush(final String pns, final String userTag, final String message)
             throws ClientProtocolException, IOException {
         new AsyncTask<Object, Object, Object>() {
             @Override
             protected Object doInBackground(Object... params) {
                 try {

                     String uri = BACKEND_ENDPOINT + "/api/notifications";
                     uri += "?pns=" + pns;
                     uri += "&to_tag=" + userTag;

                     HttpPost request = new HttpPost(uri);
                     request.addHeader("Authorization", "Basic "+ getAuthorizationHeader());
                     request.setEntity(new StringEntity(message));
                     request.addHeader("Content-Type", "application/json");

                     HttpResponse response = new DefaultHttpClient().execute(request);

                     if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                         DialogNotify("MainActivity - Error sending " + pns + " notification",
                             response.getStatusLine().toString());
                         throw new RuntimeException("Error sending notification");
                     }
                 } catch (Exception e) {
                     DialogNotify("MainActivity - Failed to send " + pns + " notification ", e.getMessage());
                     return e;
                 }

                 return null;
             }
         }.execute(null, null, null);
     }



    The `login` handler for the **Log in** button generates a basic authentication token using on the input username and password (note that this represents any token your authentication scheme uses), then it uses `RegisterClient` to call the backend for registration.

    The `sendPush` method calls the backend to trigger a secure notification to the user based on the user tag. The platform notification service that `sendPush` targets depends on the `pns` string passed in.

1. In your `MainActivity` class, update the `sendNotificationButtonOnClick` method to call the `sendPush` method with the user's selected platform notification services as follows.

       /**
     * Send Notification button click handler. This method sends the push notification
     * message to each platform selected.
     *
     * @param v The view
     */
    public void sendNotificationButtonOnClick(View v)
            throws ClientProtocolException, IOException {

        String nhMessageTag = ((EditText) findViewById(R.id.editTextNotificationMessageTag))
                .getText().toString();
        String nhMessage = ((EditText) findViewById(R.id.editTextNotificationMessage))
                .getText().toString();

        // JSON String
        nhMessage = "\"" + nhMessage + "\"";

        if (((ToggleButton)findViewById(R.id.toggleButtonWNS)).isChecked())
        {
            sendPush("wns", nhMessageTag, nhMessage);
        }
        if (((ToggleButton)findViewById(R.id.toggleButtonGCM)).isChecked())
        {
            sendPush("gcm", nhMessageTag, nhMessage);
        }
        if (((ToggleButton)findViewById(R.id.toggleButtonAPNS)).isChecked())
        {
            sendPush("apns", nhMessageTag, nhMessage);
        }
    }




## Run the Application
1. Run the application on a device or an emulator using Android Studio.

2. In the Android app, enter a username and password. They must both be the same string value and they must not contain spaces or special characters.

3. In the Android app, click **Log in**. Wait for a toast message that states **Logged in and registered**. This will enable the **Send Notification** button.

    ![][A2]

4. Click the toggle buttons to enable all platforms where you have ran the app and registered a user.

5. Enter the user's name that will receive the notification message. That user must be registered for notifications on the target devices.

6. Enter a message for the user to receive as a push notification message.

7. Click **Send Notification**.  Each device that has a registration with the matching username tag will receive the push notification.

[A1]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users.png
[A2]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users-enter-password.png
