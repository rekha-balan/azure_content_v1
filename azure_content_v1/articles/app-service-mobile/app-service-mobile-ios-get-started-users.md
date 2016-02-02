<properties
    pageTitle="Add Authentication on iOS with Azure Mobile Apps"
    description="Learn how to use Azure Mobile Apps to authenticate users of your iOS app through a variety of identity providers, including AAD, Google, Facebook, Twitter, and Microsoft."
    services="app-service\mobile"
    documentationCenter="ios"
    authors="krisragh" 
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-ios"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="12/28/2015"
    ms.author="krisragh"/>

# Add authentication to your iOS app
> [AZURE.SELECTOR]
- [Android](../articles/app-service-mobile-android-get-started-users.md)
- [iOS](../articles/app-service-mobile-ios-get-started-users.md)
- [Windows](../articles/app-service-mobile-windows-store-dotnet-get-started-users.md)
- [Xamarin.Android](../articles/app-service-mobile-xamarin-android-get-started-users.md)
- [Xamarin.Forms](../articles/app-service-mobile-xamarin-forms-get-started-users.md)
- [Xamarin.iOS](../articles/app-service-mobile-xamarin-ios-get-started-users.md)



&nbsp;  
>[AZURE.NOTE] This is an **Azure Mobile Apps** topic. For Mobile Services topics, see the [Mobile Services documentation center](/documentation/services/mobile-services/).
>
>App Service Mobile Apps is our newest mobile backend platform and [gives you additional advantages](app-service-mobile-value-prop-migration-from-mobile-services.md) over Mobile Services. [Migrating to App Service](app-service-mobile-migrating-from-mobile-services.md) is  recommended for customers using the .NET backend SDK. However, [Mobile Apps Node SDK](https://github.com/azure/azure-mobile-apps-node) is currently in Preview and is not yet recommended for production use. SDK and API contracts are subject to change within minor version releases.


In this tutorial, you add authentication to the [iOS quick start](app-service-mobile-ios-get-started.md) project using a supported identity provider. This tutorial is based on the [iOS quick start](app-service-mobile-ios-get-started.md) tutorial, which you must complete first. 

## <a name="register"></a>Register your app for authentication and configure the App Service

First, you need to register your app at an identity provider's site, and then you will set the provider-generated credentials in the Mobile App backend.

1. Configure your preferred identity provider by following the provider-specific instructions: 
	
	+ [Azure Active Directory](../articles/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication.md)
	+ [Facebook](../articles/app-service-mobile/app-service-mobile-how-to-configure-facebook-authentication.md)
	+ [Google](../articles/app-service-mobile/app-service-mobile-how-to-configure-google-authentication.md)
	+ [Microsoft](../articles/app-service-mobile/app-service-mobile-how-to-configure-microsoft-authentication.md)
	+ [Twitter](../articles/app-service-mobile/app-service-mobile-how-to-configure-twitter-authentication.md)

2. Repeat the previous steps for each provider you want to support in your app.


<!-- URLs. -->
[Azure portal]: https://portal.azure.com/


## <a name="permissions"></a>Restrict permissions to authenticated users

By default, APIs in a Mobile App backend can be invoked anonymously. Next, you need to restrict access to only authenticated clients.  

+ **Node.js backend (via portal)** :  
	
	In your Mobile App's **Settings**, click **Easy Tables** and select your table. Click **Change permissions**, select **Authenticated access only** for all permissions, then click **Save**. 

+ **.NET backend (C#)**:  

	In the server project, navigate to **Controllers** > **TodoItemController.cs**. Add the `[Authorize]` attribute to the **TodoItemController** class, as follows. To restrict access only to specific methods, you can also apply this attribute just to those methods instead of the class. Republish the server project.


        [Authorize]
        public class TodoItemController : TableController<TodoItem>

+ **Node.js backend (via Node.js code)** :  
	
	To require authentication for table access, add the following line to the Node.js server script:


        table.access = 'authenticated';

	For more details, refer to [How to: Require authentication for access to tables](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#howto-tables-auth). To learn how to download the quickstart code project from your site, see [How to: Download the Node.js backend quickstart code project using Git](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart).



In Xcode, press **Run** to  start the app. An exception will be raised because the app attempts to access the backend as an unauthenticated user, but *TodoItem* table now requires authentication.

## <a name="add-authentication"></a>Add authentication to app
**Objective-C**: 

1. On your Mac, open _QSTodoListViewController.m_ in Xcode and add the following method. Change _google_ to _microsoftaccount_, _twitter_, _facebook_, or _windowsazureactivedirectory_ if you're not using Google as your identity provider. If you use Facebook, [you will need to whitelist Facebook domains in your app](https://developers.facebook.com/docs/ios/ios9#whitelist).

            - (void) loginAndGetData
            {
                MSClient *client = self.todoService.client;
                if (client.currentUser != nil) {
                    return;
                }
            
                [client loginWithProvider:@"google" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
                    [self refresh];
                }];
            }


2. Replace `[self refresh]` in `viewDidLoad` in _QSTodoListViewController.m_ with the following:

            [self loginAndGetData];

3. Press  _Run_ to start the app, and then log in. When you are logged in, you should be able to view the Todo list and make updates.

**Swift**:

1. On your Mac, open _ToDoTableViewController.swift_ in Xcode and add the following method. Change _google_ to _microsoftaccount_, _twitter_, _facebook_, or _windowsazureactivedirectory_ if you're not using Google as your identity provider. If you use Facebook, [you will need to whitelist Facebook domains in your app](https://developers.facebook.com/docs/ios/ios9#whitelist).
        
            func loginAndGetData()
            {
                let client = self.table!.client
                if client.currentUser != nil {
                    return
                }
                    
                client.loginWithProvider("google", controller: self, animated: true, completion: { (user, error) -> Void in
                    self.refreshControl?.beginRefreshing()
                    self.onRefresh(self.refreshControl)
                })
            }

2. Remove the lines `self.refreshControl?.beginRefreshing()` and `self.onRefresh(self.refreshControl)` at the end of `viewDidLoad()` in _ToDoTableViewController.swift_. Add a call to `loginAndGetData()` in their place:

            loginAndGetData()

3. Press  _Run_ to start the app, and then log in. When you are logged in, you should be able to view the Todo list and make updates.


<!-- URLs. -->

[iOS quick start]: app-service-mobile-ios-get-started.md

[Azure portal]: https://portal.azure.com

