<properties
    pageTitle="Azure Notification Hubs Secure Push"
    description="Learn how to send secure push notifications to an iOS app from Azure. Code samples written in Objective-C and C#."
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

This Secure Push tutorial shows how to send a push notification securely. The tutorial builds on the [Notify Users](notification-hubs-aspnet-backend-ios-notify-users.md) tutorial, so you should complete the steps in that tutorial first.

> [!NOTE]
> This tutorial assumes that you have created and configured your notification hub as described in [Getting Started with Notification Hubs (iOS)](notification-hubs-ios-get-started.md).
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

## Modify the iOS project
Now that you modified your app back-end to send just the *id* of a notification, you have to change your iOS app to handle that notification and call back your back-end to retrieve the secure message to be displayed.

To achieve this goal, we have to write the logic to retrieve the secure content from the app back-end.

1. In **AppDelegate.m**, make sure the app registers for silent notifications so it processes the notification id sent from the backend. Add the **UIRemoteNotificationTypeNewsstandContentAvailability** option in didFinishLaunchingWithOptions:

        [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeNewsstandContentAvailability];
2. In your **AppDelegate.m** add an implementation section at the top with the following declaration:

        @interface AppDelegate ()
     - (void) retrieveSecurePayloadWithId:(int)payloadId completion: (void(^)(NSString*, NSError*)) completion;
     @end
3. Then add in the implementation section the following code, substituting the placeholder `{back-end endpoint}` with the endpoint for your back-end obtained previously:


```
        NSString *const GetNotificationEndpoint = @"{back-end endpoint}/api/notifications";

        - (void) retrieveSecurePayloadWithId:(int)payloadId completion: (void(^)(NSString*, NSError*)) completion;
        {
            // check if authenticated
            ANHViewController* rvc = (ANHViewController*) self.window.rootViewController;
            NSString* authenticationHeader = rvc.registerClient.authenticationHeader;
            if (!authenticationHeader) return;


            NSURLSession* session = [NSURLSession
                                     sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]
                                     delegate:nil
                                     delegateQueue:nil];


            NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/%d", GetNotificationEndpoint, payloadId]];
            NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
            [request setHTTPMethod:@"GET"];
            NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@", authenticationHeader];
            [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

            NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
                NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                if (!error && httpResponse.statusCode == 200)
                {
                    NSLog(@"Received secure payload: %@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);

                    NSMutableDictionary *json = [NSJSONSerialization JSONObjectWithData:data options: NSJSONReadingMutableContainers error: &error];

                    completion([json objectForKey:@"Payload"], nil);
                }
                else
                {
                    NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
                    if (error)
                        completion(nil, error);
                    else {
                        completion(nil, [NSError errorWithDomain:@"APICall" code:httpResponse.statusCode userInfo:nil]);
                    }
                }
            }];
            [dataTask resume];
        }
```

    This method calls your app back-end to retrieve the notification content using the credentials stored in the shared preferences.

1. Now we have to handle the incoming notification and use the method above to retrieve the content to display. First, we have to enable your iOS app to run in the background when receiving a push notification. In **XCode**, select your app project on the left panel, then click your main app target in the **Targets** section from the central pane.

2. Then click your **Capabilities** tab at the top of your central pane, and check the **Remote Notifications** checkbox.

    ![][IOS1]


1. In **AppDelegate.m** add the following method to handle push notifications:

        -(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
     {
         NSLog(@"%@", userInfo);

         [self retrieveSecurePayloadWithId:[[userInfo objectForKey:@"secureId"] intValue] completion:^(NSString * payload, NSError *error) {
             if (!error) {
                 // show local notification
                 UILocalNotification* localNotification = [[UILocalNotification alloc] init];
                 localNotification.fireDate = [NSDate dateWithTimeIntervalSinceNow:0];
                 localNotification.alertBody = payload;
                 localNotification.timeZone = [NSTimeZone defaultTimeZone];
                 [[UIApplication sharedApplication] scheduleLocalNotification:localNotification];

                 completionHandler(UIBackgroundFetchResultNewData);
             } else {
                 completionHandler(UIBackgroundFetchResultFailed);
             }
         }];

     }

    Note that it is preferable to handle the cases of missing authentication header property or rejection by the back-end. The specific handling of these cases depend mostly on your target user experience. One option is to display a notification with a generic prompt for the user to authenticate to retrieve the actual notification.


## Run the Application
To run the application, do the following:

1. In XCode, run the app on a physical iOS device (push notifications will not work in the simulator).

2. In the iOS app UI, enter a username and password. These can be any string, but they must be the same value.

3. In the iOS app UI, click **Log in**. Then click **Send push**. You should see the secure notification being displayed in your notification center.


[IOS1]: ./media/notification-hubs-aspnet-backend-ios-secure-push/secure-push-ios-1.png
