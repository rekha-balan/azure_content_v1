<properties
    pageTitle="Azure Notification Hubs Notify Users for iOS with .NET backend"
    description="Learn how to send push notifications to users in Azure. Code samples written in Objective-C and the .NET API for the backend."
    documentationCenter="ios"
    authors="wesmc7777"
    manager="dwrede"
    editor=""
    services="notification-hubs"/>

<tags
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="ios"
    ms.devlang="objective-c"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="wesmc"/>

# Azure Notification Hubs Notify Users for iOS with .NET backend
> [AZURE.SELECTOR]
- [Windows Runtime 8.1 universal](../articles/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-notify-users.md)
- [iOS](../articles/notification-hubs/notification-hubs-aspnet-backend-ios-notify-users.md)
- [Android](../articles/notification-hubs/notification-hubs-aspnet-backend-android-notify-users.md)



## Overview
Push notification support in Azure enables you to access an easy-to-use, multiplatform, and scaled-out push infrastructure, which greatly simplifies the implementation of push notifications for both consumer and enterprise applications for mobile platforms. This tutorial shows you how to use Azure Notification Hubs to send push notifications to a specific app user on a specific device. An ASP.NET WebAPI backend is used to authenticate clients and to generate notifications, as shown in the guidance topic [Registering from your app backend](notification-hubs-registration-management.md#registration-management-from-a-backend).

> [!NOTE]
> This tutorial assumes that you have created and configured your notification hub as described in [Getting Started with Notification Hubs (iOS)](notification-hubs-ios-get-started.md). This tutorial is also the prerequisite to the [Secure Push (iOS)](notification-hubs-aspnet-backend-ios-secure-push.md) tutorial.
> If you are using Mobile Services as your backend service, see the [Mobile Services version](../mobile-services-javascript-backend-ios-push-notifications-app-users.md) of this tutorial.
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


## Modify your iOS app
1. Open the Single Page view app you created in the [Getting Started with Notification Hubs (iOS)](notification-hubs-ios-get-started.md) tutorial.

   > [!NOTE]
> In this section we assume that your project is configured with an empty organization name. If not, you will need to prepend your organization name to all class names.
> 
2. In your Main.storyboard add the components shown in the screenshot below from the object library.

    ![][1]

   * **Username**: A UITextField with placeholder text, *Enter Username*, immediately beneath the send results label and constrained to the left and right margins and beneath the send results label.
* **Password**: A UITextField with placeholder text, *Enter Password*, immediately beneath the username text field and constrained to the left and right margins and beneath the username text field. Check the **Secure Text Entry** option in the Attribute Inspector, under *Return Key*.
* **Log in**: A UIButton labeled immediately beneath the password text field and uncheck the **Enabled** option in the Attributes Inspector, under *Control-Content*
* **WNS**: Label and switch to enable sending the notification Windows Notification Service if it has been setup on the hub. See the [Windows Getting Started](notification-hubs-windows-store-dotnet-get-started.md) tutorial.
* **GCM**: Label and switch to enable sending the notification to Google Cloud Messaging if it has been setup on the hub. See [Android Getting Started](notification-hubs-android-get-started.md) tutorial.
* **APNS**: Label and switch to enable sending the notification to the Apple Platform Notification Service.
* **Recipent Username**:A UITextField with placeholder text, *Recipient username tag*, immediately beneath the GCM label and constrained to the left and right margins and beneath the GCM label.


    Some components were added in the [Getting Started with Notification Hubs (iOS)](notification-hubs-ios-get-started.md) tutorial.

1. **Ctrl** drag from the components in the view to ViewController.h and add these new outlets.

        @property (weak, nonatomic) IBOutlet UITextField *UsernameField;
     @property (weak, nonatomic) IBOutlet UITextField *PasswordField;
     @property (weak, nonatomic) IBOutlet UITextField *RecipientField;
     @property (weak, nonatomic) IBOutlet UITextField *NotificationField;

     // Used to enable the buttons on the UI
     @property (weak, nonatomic) IBOutlet UIButton *LogInButton;
     @property (weak, nonatomic) IBOutlet UIButton *SendNotificationButton;

     // Used to enabled sending notifications across platforms
     @property (weak, nonatomic) IBOutlet UISwitch *WNSSwitch;
     @property (weak, nonatomic) IBOutlet UISwitch *GCMSwitch;
     @property (weak, nonatomic) IBOutlet UISwitch *APNSSwitch;

     - (IBAction)LogInAction:(id)sender;
2. In ViewController.h, add the following `#define` just below your import statements. Substitute the *<Enter Your Backend Endpoint\>* placeholder with the Destination URL you used to deploy your app backend in the previous section. For example, *http://you_backend.azurewebsites.net*.

        #define BACKEND_ENDPOINT @"<Enter Your Backend Endpoint>"
3. In your project, create a new **Cocoa Touch class** named **RegisterClient** to interface with the ASP.NET back-end you created. Create the class inheriting from `NSObject`. Then add the following code in the RegisterClient.h.

        @interface RegisterClient : NSObject

     @property (strong, nonatomic) NSString* authenticationHeader;

     -(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
         andCompletion:(void(^)(NSError*))completion;

     -(instancetype) initWithEndpoint:(NSString*)Endpoint;

     @end
4. In the RegisterClient.m update the `@interface` section:

        @interface RegisterClient ()

     @property (strong, nonatomic) NSURLSession* session;
     @property (strong, nonatomic) NSURLSession* endpoint;

     -(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry
                 andCompletion:(void(^)(NSError*))completion;
     -(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token
                 completion:(void(^)(NSString*, NSError*))completion;
     -(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSString*)token
                 tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion;

     @end
5. Replace the `@implementation` section in the RegisterClient.m with the following code.


        @implementation RegisterClient

        // Globals used by RegisterClient
        NSString *const RegistrationIdLocalStorageKey = @"RegistrationId";

        -(instancetype) initWithEndpoint:(NSString*)Endpoint
        {
            self = [super init];
            if (self) {
                NSURLSessionConfiguration* config = [NSURLSessionConfiguration defaultSessionConfiguration];
                _session = [NSURLSession sessionWithConfiguration:config delegate:nil delegateQueue:nil];
                _endpoint = Endpoint;
            }
            return self;
        }

        -(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
                    andCompletion:(void(^)(NSError*))completion
        {
            [self tryToRegisterWithDeviceToken:token tags:tags retry:YES andCompletion:completion];
        }

        -(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry
                    andCompletion:(void(^)(NSError*))completion
        {
            NSSet* tagsSet = tags?tags:[[NSSet alloc] init];

            NSString *deviceTokenString = [[token description]
                stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"<>"]];
            deviceTokenString = [[deviceTokenString stringByReplacingOccurrencesOfString:@" " withString:@""]
                                    uppercaseString];

            [self retrieveOrRequestRegistrationIdWithDeviceToken: deviceTokenString
                completion:^(NSString* registrationId, NSError *error) {
                NSLog(@"regId: %@", registrationId);
                if (error) {
                    completion(error);
                    return;
                }

                [self upsertRegistrationWithRegistrationId:registrationId deviceToken:deviceTokenString
                    tags:tagsSet andCompletion:^(NSURLResponse * response, NSError *error) {
                    if (error) {
                        completion(error);
                        return;
                    }

                    NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*)response;
                    if (httpResponse.statusCode == 200) {
                        completion(nil);
                    } else if (httpResponse.statusCode == 410 && retry) {
                        [self tryToRegisterWithDeviceToken:token tags:tags retry:NO andCompletion:completion];
                    } else {
                        NSLog(@"Registration error with response status: %ld", (long)httpResponse.statusCode);

                        completion([NSError errorWithDomain:@"Registration" code:httpResponse.statusCode
                                    userInfo:nil]);
                    }

                }];
            }];
        }

        -(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSData*)token
                    tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion
        {
            NSDictionary* deviceRegistration = @{@"Platform" : @"apns", @"Handle": token,
                                                    @"Tags": [tags allObjects]};
            NSData* jsonData = [NSJSONSerialization dataWithJSONObject:deviceRegistration
                                options:NSJSONWritingPrettyPrinted error:nil];

            NSLog(@"JSON registration: %@", [[NSString alloc] initWithData:jsonData
                                                encoding:NSUTF8StringEncoding]);

            NSString* endpoint = [NSString stringWithFormat:@"%@/api/register/%@", _endpoint,
                                    registrationId];
            NSURL* requestURL = [NSURL URLWithString:endpoint];
            NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
            [request setHTTPMethod:@"PUT"];
            [request setHTTPBody:jsonData];
            NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                                                    self.authenticationHeader];
            [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];
            [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];

            NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request
                completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
            {
                if (!error)
                {
                    completion(response, error);
                }
                else
                {
                    NSLog(@"Error request: %@", error);
                    completion(nil, error);
                }
            }];
            [dataTask resume];
        }

        -(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token
                    completion:(void(^)(NSString*, NSError*))completion
        {
            NSString* registrationId = [[NSUserDefaults standardUserDefaults]
                                        objectForKey:RegistrationIdLocalStorageKey];

            if (registrationId)
            {
                completion(registrationId, nil);
                return;
            }

            // request new one & save
            NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/api/register?handle=%@",
                                    _endpoint, token]];
            NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
            [request setHTTPMethod:@"POST"];
            NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                                                    self.authenticationHeader];
            [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

            NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request
                completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
            {
                NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                if (!error && httpResponse.statusCode == 200)
                {
                    NSString* registrationId = [[NSString alloc] initWithData:data
                        encoding:NSUTF8StringEncoding];

                    // remove quotes
                    registrationId = [registrationId substringWithRange:NSMakeRange(1,
                                        [registrationId length]-2)];

                    [[NSUserDefaults standardUserDefaults] setObject:registrationId
                        forKey:RegistrationIdLocalStorageKey];
                    [[NSUserDefaults standardUserDefaults] synchronize];

                    completion(registrationId, nil);
                }
                else
                {
                    NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
                    if (error)
                        completion(nil, error);
                    else {
                        completion(nil, [NSError errorWithDomain:@"Registration" code:httpResponse.statusCode
                                    userInfo:nil]);
                    }
                }
            }];
            [dataTask resume];
        }

        @end

    The code above implements the logic explained in the guidance article [Registering from your app backend](notification-hubs-registration-management.md#registration-management-from-a-backend) using NSURLSession to perform REST calls to your app backend, and NSUserDefaults to locally store the registrationId returned by the notification hub.

    Note that this class requires its property **authorizationHeader** to be set in order to work properly. This property is set by the **ViewController** class after the log in.

1. In ViewController.h, add a `#import` statement for RegisterClient.h. Then add a declaration for the device token and reference to a `RegisterClient` instance in the `@interface` section:

        #import "RegisterClient.h"

     @property (strong, nonatomic) NSData* deviceToken;
     @property (strong, nonatomic) RegisterClient* registerClient;
2. In ViewController.m, add a private method declaration in the `@interface` section:

        @interface ViewController () <UITextFieldDelegate, NSURLConnectionDataDelegate, NSXMLParserDelegate>

     // create the Authorization header to perform Basic authentication with your app back-end
     -(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username
                     AndPassword:(NSString*)password;

     @end


> [!NOTE]
> The following snippet is not a secure authentication scheme, you should substitute the implementation of the **createAndSetAuthenticationHeaderWithUsername:AndPassword:** with your specific authentication mechanism that generates an authentication token to be consumed by the register client class, e.g. OAuth, Active Directory.
> 
> 
1. Then in the `@implementation` section of ViewController.m add the following code which adds the implementation for setting the device token and authentication header.

        -(void) setDeviceToken: (NSData*) deviceToken
     {
         _deviceToken = deviceToken;
         self.LogInButton.enabled = YES;
     }

     -(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username
                     AndPassword:(NSString*)password;
     {
         NSString* headerValue = [NSString stringWithFormat:@"%@:%@", username, password];

         NSData* encodedData = [[headerValue dataUsingEncoding:NSUTF8StringEncoding] base64EncodedDataWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];

         self.registerClient.authenticationHeader = [[NSString alloc] initWithData:encodedData
                                                     encoding:NSUTF8StringEncoding];
     }

     -(BOOL)textFieldShouldReturn:(UITextField *)textField
     {
         [textField resignFirstResponder];
         return YES;
     }

    Note how setting the device token enables the log in button. This is becasue as a part of the login action, the view controller registers for push notifications with the app backend. Hence, we do not want Log In action to be accessible till the device token has been properly set up. You can decouple the log in from the push registration as long as the former happens before the latter.

2. In ViewController.m, use the following snippets to implement the action method for your **Log In** button and a method to send the notification message using the ASP.NET backend.

       - (IBAction)LogInAction:(id)sender {
        // create authentication header and set it in register client
        NSString* username = self.UsernameField.text;
        NSString* password = self.PasswordField.text;

        [self createAndSetAuthenticationHeaderWithUsername:username AndPassword:password];

        __weak ViewController* selfie = self;
        [self.registerClient registerWithDeviceToken:self.deviceToken tags:nil
            andCompletion:^(NSError* error) {
            if (!error) {
                dispatch_async(dispatch_get_main_queue(),
                ^{
                    selfie.SendNotificationButton.enabled = YES;
                    [self MessageBox:@"Success" message:@"Registered successfully!"];
                });
            }
        }];
    }



        - (void)SendNotificationASPNETBackend:(NSString*)pns UsernameTag:(NSString*)usernameTag
                    Message:(NSString*)message
        {
            NSURLSession* session = [NSURLSession
                sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:nil
                delegateQueue:nil];

            // Pass the pns and username tag as parameters with the REST URL to the ASP.NET backend
            NSURL* requestURL = [NSURL URLWithString:[NSString
                stringWithFormat:@"%@/api/notifications?pns=%@&to_tag=%@", BACKEND_ENDPOINT, pns,
                usernameTag]];

            NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
            [request setHTTPMethod:@"POST"];

            // Get the mock authenticationheader from the register client
            NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                self.registerClient.authenticationHeader];
            [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

            //Add the notification message body
            [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];
            [request setHTTPBody:[message dataUsingEncoding:NSUTF8StringEncoding]];

            // Execute the send notification REST API on the ASP.NET Backend
            NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
                completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
            {
                NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                if (error || httpResponse.statusCode != 200)
                {
                    NSString* status = [NSString stringWithFormat:@"Error Status for %@: %d\nError: %@\n",
                                        pns, httpResponse.statusCode, error];
                    dispatch_async(dispatch_get_main_queue(),
                    ^{
                        // Append text because all 3 PNS calls may also have information to view
                        [self.sendResults setText:[self.sendResults.text stringByAppendingString:status]];
                    });
                    NSLog(status);
                }

                if (data != NULL)
                {
                    xmlParser = [[NSXMLParser alloc] initWithData:data];
                    [xmlParser setDelegate:self];
                    [xmlParser parse];
                }
            }];
            [dataTask resume];
        }


1. Update the action for the **Send Notification** button to use the ASP.NET backend and send to any PNS enabled by a switch.

        - (IBAction)SendNotificationMessage:(id)sender
        {
            //[self SendNotificationRESTAPI];
            [self SendToEnabledPlatforms];
        }


        -(void)SendToEnabledPlatforms
        {
            NSString* json = [NSString stringWithFormat:@"\"%@\"",self.notificationMessage.text];

            [self.sendResults setText:@""];

            if ([self.WNSSwitch isOn])
                [self SendNotificationASPNETBackend:@"wns" UsernameTag:self.RecipientField.text Message:json];

            if ([self.GCMSwitch isOn])
                [self SendNotificationASPNETBackend:@"gcm" UsernameTag:self.RecipientField.text Message:json];

            if ([self.APNSSwitch isOn])
                [self SendNotificationASPNETBackend:@"apns" UsernameTag:self.RecipientField.text Message:json];
        }



1. In function **ViewDidLoad**, add the following to instantiate the RegisterClient instance and set the delegate for your text fields.

       self.UsernameField.delegate = self;
    self.PasswordField.delegate = self;
    self.RecipientField.delegate = self;
    self.registerClient = [[RegisterClient alloc] initWithEndpoint:BACKEND_ENDPOINT];
2. Now in **AppDelegate.m**, remove all the content of the method **application:didRegisterForPushNotificationWithDeviceToken:** and replace it with the following to make sure that the view controller contains the latest device token retrieved from APNs:

       // Add import to the top of the file
    #import "ViewController.h"

    - (void)application:(UIApplication *)application
                didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
    {
        ViewController* rvc = (ViewController*) self.window.rootViewController;
        rvc.deviceToken = deviceToken;
    }
3. Finally in **AppDelegate.m**, make sure you have the following method:

       - (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
        NSLog(@"%@", userInfo);
        [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
    }


## Test the Application
1. In XCode, run the app on a physical iOS device (push notifications will not work in the simulator).

2. In the iOS app UI, enter a username and password. These can be any string, but they must both be the same string value. Then click **Log In**.

    ![][2]


1. You should see a pop-up informing you of registration success. Click **OK**.

    ![][3]

2. In the **Recipient username tag* text field, enter the user name tag used with the registration from another device.

3. Enter a notification message and click **Send Notification**.  Only the devices that have a registration with the recipient user name tag receive the notification message.  It is only sent to those users.

    ![][4]


[1]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-interface.png
[2]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-user-pwd.png
[3]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-registered.png
[4]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-msg.png
