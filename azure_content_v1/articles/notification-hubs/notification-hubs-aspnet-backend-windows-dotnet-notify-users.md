<properties
    pageTitle="Azure Notification Hubs Notify Users with .NET backend"
    description="Learn how to send secure push notifications in Azure. Code samples written in C# using the .NET API."
    documentationCenter="windows"
    authors="wesmc7777"
    manager="dwrede"
    services="notification-hubs"
    editor=""/>

<tags
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="11/09/2015"
    ms.author="wesmc"/>

# Azure Notification Hubs Notify Users with .NET backend
> [AZURE.SELECTOR]
- [Windows Runtime 8.1 universal](../articles/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-notify-users.md)
- [iOS](../articles/notification-hubs/notification-hubs-aspnet-backend-ios-notify-users.md)
- [Android](../articles/notification-hubs/notification-hubs-aspnet-backend-android-notify-users.md)



## Overview
Push notification support in Azure enables you to access an easy-to-use, multiplatform, and scaled-out push infrastructure, which greatly simplifies the implementation of push notifications for both consumer and enterprise applications for mobile platforms. This tutorial shows you how to use Azure Notification Hubs to send push notifications to a specific app user on a specific device. An ASP.NET WebAPI backend is used to authenticate clients. Using the authenticated client user, and tag will be automatically added by the backend to notification registration. This tag will be used to send by the backend to generate notifications for a specific user. For more information on registering for notifications using an app backend, see the guidance topic [Registering from your app backend](http://msdn.microsoft.com/library/dn743807.aspx). This tutorial builds on the notification hub and project that you created in the [Get started with Notification Hubs](notification-hubs-windows-store-dotnet-get-started.md) tutorial.

This tutorial is also the prerequisite to the [Secure Push](notification-hubs-aspnet-backend-windows-dotnet-secure-push.md) tutorial. After you have completed the steps in this tutorial, you can proceed to the [Secure Push](notification-hubs-aspnet-backend-windows-dotnet-secure-push.md) tutorial, which shows how to modify the code in this tutorial to send a push notification securely.

## Before you begin
We do take your feedback seriously. If you have any difficulties completing this topic, or recommendations for improving this content, we would appreciate your feedback at the bottom of the page.

The completed code for this tutorial can be found on GitHub [here](https://github.com/Azure/azure-notificationhubs-samples/tree/master/dotnet/NotifyUsers). 

## Prerequisites
Before you start this tutorial, you must have already completed these Mobile Services tutorials:

* [Get started with Notification Hubs](notification-hubs-windows-store-dotnet-get-started.md)<br/>You create your notification hub and reserve the app name and register to receive notifications in this tutorial. This tutorial assumes you have already completed these steps. If not, please follow the steps in [Getting Started with Notification Hubs (Windows Store)](notification-hubs-windows-store-dotnet-get-started.md); specifically, the sections [Register your app for the Windows Store](notification-hubs-windows-store-dotnet-get-started.md#register-your-app-for-the-windows-store) and [Configure your Notification Hub](notification-hubs-windows-store-dotnet-get-started.md#configure-your-notification-hub). In particular, make sure that you have entered the **Package SID** and **Client Secret** values in the portal, in the **Configure** tab for your notification hub. This configuration procedure is described in the section [Configure your Notification Hub](notification-hubs-windows-store-dotnet-get-started.md#configure-your-notification-hub). This is an important step: if the credentials on the portal do not match those specified for the app name you choose, the push notification will not succeed.

> [!NOTE]
> If you are using Mobile Services as your backend service, see the [Mobile Services version](../mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users.md) of this tutorial.
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


## Update the code for the client project
In this section, you update the code in the project you completed for the [Get started with Notification Hubs](notification-hubs-windows-store-dotnet-get-started.md) tutorial. The should already be associated with the store and configured for your notification hub. In this section, you will add code to call the new WebAPI backend and use it for registering and sending notifications.

1. In Visual Studio, open the the solution you created for the [Get started with Notification Hubs](notification-hubs-windows-store-dotnet-get-started.md) tutorial.

2. In Solution Explorer, right-click the **(Windows 8.1)** project and then click **Manage NuGet Packages**.

3. On the left-hand side, click **Online**.

4. In the **Search** box, type **Http Client**.

5. In the results list, click **Microsoft HTTP Client Libraries**, and then click **Install**. Complete the installation.

6. Back in the NuGet **Search** box, type **Json.net**. Install the **Json.NET** package, and then close the NuGet Package Manager window.

7. Repeat the steps above for the **(Windows Phone 8.1)** project to install the **JSON.NET** NuGet package for the Windows Phone project.


1. In Solution Explorer, in the **(Windows 8.1)** project, double-click **MainPage.xaml** to open it in the Visual Studio editor.

2. In the **MainPage.xaml** XML code, replace the `<Grid>` section with the following code. This code adds a username and password textbox that the user will authenticate with. It also adds textboxes for the notification message and the username tag that should receive the notification:

        <Grid>
         <Grid.RowDefinitions>
             <RowDefinition Height="Auto"/>
             <RowDefinition Height="*"/>
         </Grid.RowDefinitions>

         <TextBlock Grid.Row="0" Text="Notify Users" HorizontalAlignment="Center" FontSize="48"/>

         <StackPanel Grid.Row="1" VerticalAlignment="Center">
             <Grid>
                 <Grid.RowDefinitions>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                     <RowDefinition Height="Auto"/>
                 </Grid.RowDefinitions>
                 <Grid.ColumnDefinitions>
                     <ColumnDefinition></ColumnDefinition>
                     <ColumnDefinition></ColumnDefinition>
                     <ColumnDefinition></ColumnDefinition>
                 </Grid.ColumnDefinitions>
                 <TextBlock Grid.Row="0" Grid.ColumnSpan="3" Text="Username" FontSize="24" Margin="20,0,20,0"/>
                 <TextBox Name="UsernameTextBox" Grid.Row="1" Grid.ColumnSpan="3" Margin="20,0,20,0"/>
                 <TextBlock Grid.Row="2" Grid.ColumnSpan="3" Text="Password" FontSize="24" Margin="20,0,20,0" />
                 <PasswordBox Name="PasswordTextBox" Grid.Row="3" Grid.ColumnSpan="3" Margin="20,0,20,0"/>

                 <Button Grid.Row="4" Grid.ColumnSpan="3" HorizontalAlignment="Center" VerticalAlignment="Center"
                             Content="1. Login and register" Click="LoginAndRegisterClick" Margin="0,0,0,20"/>

                 <ToggleButton Name="toggleWNS" Grid.Row="5" Grid.Column="0" HorizontalAlignment="Right" Content="WNS" IsChecked="True" />
                 <ToggleButton Name="toggleGCM" Grid.Row="5" Grid.Column="1" HorizontalAlignment="Center" Content="GCM" />
                 <ToggleButton Name="toggleAPNS" Grid.Row="5" Grid.Column="2" HorizontalAlignment="Left" Content="APNS" />

                 <TextBlock Grid.Row="6" Grid.ColumnSpan="3" Text="Username Tag To Send To" FontSize="24" Margin="20,0,20,0"/>
                 <TextBox Name="ToUserTagTextBox" Grid.Row="7" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                 <TextBlock Grid.Row="8" Grid.ColumnSpan="3" Text="Enter Notification Message" FontSize="24" Margin="20,0,20,0"/>
                 <TextBox Name="NotificationMessageTextBox" Grid.Row="9" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                 <Button Grid.Row="10" Grid.ColumnSpan="3" HorizontalAlignment="Center" Content="2. Send push" Click="PushClick" Name="SendPushButton" />
             </Grid>
         </StackPanel>
     </Grid>



1. In Solution Explorer, in the **(Windows Phone 8.1)** project, open **MainPage.xaml** and replace the Windows Phone 8.1 `<Grid>` section with that same code above. The interface should look similar to whats shown below.

   ![][13]

2. In Solution Explorer, open the **MainPage.xaml.cs** file for the **(Windows 8.1)** and **(Windows Phone 8.1)** projects. Add the following `using` statements at the top of both files:

       using System.Net.Http;
    using Windows.Storage;
    using System.Net.Http.Headers;
    using Windows.Networking.PushNotifications;
    using Windows.UI.Popups;
    using System.Threading.Tasks;
3. In **MainPage.xaml.cs** for the **(Windows 8.1)** and **(Windows Phone 8.1)** projects, add the following member to the `MainPage` class. Be sure to replace `<Enter Your Backend Endpoint>` with the your actual backend endpoint obtained previously. For example, `http://mybackend.azurewebsites.net`.

       private static string BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";




1. Add the code below to the MainPage class in **MainPage.xaml.cs** for the **(Windows 8.1)** and **(Windows Phone 8.1)** projects.

   The `PushClick` method is the click handler for the **Send Push** button. It calls the backend to trigger a notification to all devices with a username tag that matches the `to_tag` parameter. The notification message is sent as JSON content in the request body.

   The `LoginAndRegisterClick` method is the click handler for the **Log in and register** button. It stores the basic authentication token in local storage (note that this represents any token your authentication scheme uses), then uses `RegisterClient` to register for notifications using the backend.


        private async void PushClick(object sender, RoutedEventArgs e)
        {
            if (toggleWNS.IsChecked.Value)
            {
                await sendPush("wns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
            }
            if (toggleGCM.IsChecked.Value)
            {
                await sendPush("gcm", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
            }
            if (toggleAPNS.IsChecked.Value)
            {
                await sendPush("apns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);

            }
        }

        private async Task sendPush(string pns, string userTag, string message)
        {
            var POST_URL = BACKEND_ENDPOINT + "/api/notifications?pns=" +
                pns + "&to_tag=" + userTag;

            using (var httpClient = new HttpClient())
            {
                var settings = ApplicationData.Current.LocalSettings.Values;
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                try
                {
                    await httpClient.PostAsync(POST_URL, new StringContent("\"" + message + "\"",
                        System.Text.Encoding.UTF8, "application/json"));
                }
                catch (Exception ex)
                {
                    MessageDialog alert = new MessageDialog(ex.Message, "Failed to send " + pns + " message");
                    alert.ShowAsync();
                }
            }
        }

        private async void LoginAndRegisterClick(object sender, RoutedEventArgs e)
        {
            SetAuthenticationTokenInLocalStorage();

            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

            // The "username:<user name>" tag gets automatically added by the message handler in the backend.
            // The tag passed here can be whatever other tags you may want to use.
            try
            {
                // The device handle used will be different depending on the device and PNS. 
                // Windows devices use the channel uri as the PNS handle.
                await new RegisterClient(BACKEND_ENDPOINT).RegisterAsync(channel.Uri, new string[] { "myTag" });

                var dialog = new MessageDialog("Registered as: " + UsernameTextBox.Text);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
                SendPushButton.IsEnabled = true;
            }
            catch (Exception ex)
            {
                MessageDialog alert = new MessageDialog(ex.Message, "Failed to register with RegisterClient");
                alert.ShowAsync();
            }
        }

        private void SetAuthenticationTokenInLocalStorage()
        {
            string username = UsernameTextBox.Text;
            string password = PasswordTextBox.Password;

            var token = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password));
            ApplicationData.Current.LocalSettings.Values["AuthenticationToken"] = token;
        }



1. In Solution Explorer, under the **Shared** project, open the **App.xaml.cs** file. Find the call to `InitNotificationsAsync()` in the `OnLaunched()` event handler. Comment out or delete the call to `InitNotificationsAsync()`. The button handler added above will initialize notification registrations.

        protected override void OnLaunched(LaunchActivatedEventArgs e)
        {
            //InitNotificationsAsync();


1. In Solution Explorer, right-click the **Shared** project, then click **Add**, and then click **Class**. Name the class **RegisterClient.cs**, then click **OK** to generate the class.

   This class will wrap the REST calls required to contact the app backend, in order to register for push notifications. It also locally stores the *registrationIds* created by the Notification Hub as detailed in [Registering from your app backend](http://msdn.microsoft.com/library/dn743807.aspx). Note that it uses an authorization token stored in local storage when you click the **Log in and register** button.


1. Add the following `using` statements at the top of the RegisterClient.cs file:

       using Windows.Storage;
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Newtonsoft.Json;
    using System.Threading.Tasks;
    using System.Linq;
2. Add the following code inside the `RegisterClient` class definition.

       private string POST_URL;

    private class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    public RegisterClient(string backendEndpoint)
    {
        POST_URL = backendEndpoint + "/api/register";
    }

    public async Task RegisterAsync(string handle, IEnumerable<string> tags)
    {
        var regId = await RetrieveRegistrationIdOrRequestNewOneAsync();

        var deviceRegistration = new DeviceRegistration
        {
            Platform = "wns",
            Handle = handle,
            Tags = tags.ToArray<string>()
        };

        var statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);

        if (statusCode == HttpStatusCode.Gone)
        {
            // regId is expired, deleting from local storage & recreating
            var settings = ApplicationData.Current.LocalSettings.Values;
            settings.Remove("__NHRegistrationId");
            regId = await RetrieveRegistrationIdOrRequestNewOneAsync();
            statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);
        }

        if (statusCode != HttpStatusCode.Accepted)
        {
            // log or throw
            throw new System.Net.WebException(statusCode.ToString());
        }
    }

    private async Task<HttpStatusCode> UpdateRegistrationAsync(string regId, DeviceRegistration deviceRegistration)
    {
        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string) settings["AuthenticationToken"]);

            var putUri = POST_URL + "/" + regId;

            string json = JsonConvert.SerializeObject(deviceRegistration);
                            var response = await httpClient.PutAsync(putUri, new StringContent(json, Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    private async Task<string> RetrieveRegistrationIdOrRequestNewOneAsync()
    {
        var settings = ApplicationData.Current.LocalSettings.Values;
        if (!settings.ContainsKey("__NHRegistrationId"))
        {
            using (var httpClient = new HttpClient())
            {
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                var response = await httpClient.PostAsync(POST_URL, new StringContent(""));
                if (response.IsSuccessStatusCode)
                {
                    string regId = await response.Content.ReadAsStringAsync();
                    regId = regId.Substring(1, regId.Length - 2);
                    settings.Add("__NHRegistrationId", regId);
                }
                else
                {
                    throw new System.Net.WebException(response.StatusCode.ToString());
                }
            }
        }
        return (string)settings["__NHRegistrationId"];

    }
3. Save all your changes.


## Testing the Application
1. Launch the application on both Windows 8.1 and Windows Phone 8.1. For Windows Phone 8.1 you can run the instance in the emulator or an actual device.

2. In the Windows 8.1 instance of the app, enter a **Username** and **Password** as shown in the screen below. It should differ from the user name and password you enter on Windows Phone.


1. Click **Log in and register** and verify a dialog shows that you have logged in. This will also enable the **Send Push** button.

    ![][14]

2. On the Windows Phone 8.1 instance, enter a user name string in both the **Username** and **Password** fields then click **Login and register**.

3. Then in the **Recipient Username Tag** field, enter the user name registered on Windows 8.1. Enter a notification message and click **Send Push**.

    ![][16]

4. Only the devices that have registered with the matching username tag receive the notification message.

    ![][15]


## Next Steps
* If you want to segment your users by interest groups, see [Use Notification Hubs to send breaking news](notification-hubs-windows-store-dotnet-send-breaking-news.md).
* To learn more about how to use Notification Hubs, see [Notification Hubs Guidance](http://msdn.microsoft.com/library/jj927170.aspx).

[9]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push9.png
[10]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push10.png
[11]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push11.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-ui.png
[14]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-windows-instance-username.png
[15]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-notification-received.png
[16]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-send-message.png



<!-- URLs. -->

[Get started with Notification Hubs]: notification-hubs-windows-store-dotnet-get-started.md
[Secure Push]: notification-hubs-aspnet-backend-windows-dotnet-secure-push.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-store-dotnet-send-breaking-news.md
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
