<properties 
    pageTitle="Get Started with authentication for Mobile Apps in Xamarin Android" 
    description="Learn how to use Mobile Apps to authenticate users of your Xamarin Android app through a variety of identity providers, including AAD, Google, Facebook, Twitter, and Microsoft." 
    services="app-service\mobile" 
    documentationCenter="xamarin" 
    authors="mattchenderson" 
    manager="dwrede" 
    editor=""/>

<tags 
    ms.service="app-service-mobile" 
    ms.workload="mobile" 
    ms.tgt_pltfrm="mobile-xamarin-android" 
    ms.devlang="dotnet" 
    ms.topic="article" 
    ms.date="12/07/2015" 
    ms.author="mahender"/>

# Add authentication to your Xamarin.Android app
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


This topic shows you how to authenticate users of a Mobile App from your client application. In this tutorial, you add authentication to the quickstart project using an identity provider that is supported by Azure Mobile Apps. After being successfully authenticated and authorized in the Mobile App, the user ID value is displayed.

This tutorial is based on the Mobile App quickstart. You must also first complete the tutorial [Create a Xamarin.Android app](app-service-mobile-xamarin-android-get-started.md). If you do not use the downloaded quick start server project, you must add the authentication extension package to your project. For more information about server extension packages, see [Work with the .NET backend server SDK for Azure Mobile Apps](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md). 

## <a name="register"></a>Register your app for authentication and configure App Services

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



In Visual Studio or Xamarin Studio, run the client project on a device or emulator. Verify that an unhandled exception with a status code of 401 (Unauthorized) is raised after the app starts. This happens because the app attempts to access your Mobile App backend as an unauthenticated user. The *TodoItem* table now requires authentication.

Next, you will update the client app to request resources from the Mobile App backend with an authenticated user.

## <a name="add-authentication"></a>Add authentication to the app
The app is updated to require users to tap the **Sign in** button and authenticate before data is displayed.

1. Add the following code to the **TodoActivity** class:

        // Define a authenticated user.
     private MobileServiceUser user;
     private async Task<bool> Authenticate()
     {
             var success = false;
             try
             {
                 // Sign in with Facebook login using a server-managed flow.
                 user = await client.LoginAsync(this,
                     MobileServiceAuthenticationProvider.Facebook);
                 CreateAndShowDialog(string.Format("you are now logged in - {0}",
                     user.UserId), "Logged in!");

                 success = true;
             }
             catch (Exception ex)
             {
                 CreateAndShowDialog(ex, "Authentication failed");
             }
             return success;
     }

     [Java.Interop.Export()]
     public async void LoginUser(View view)
     {
         // Load data only after authentication succeeds.
         if (await Authenticate())
         {
             //Hide the button after authentication succeeds. 
             FindViewById<Button>(Resource.Id.buttonLoginUser).Visibility = ViewStates.Gone;

             // Load the data.
             OnRefreshItemsSelected();
         }
     }

    This creates a new method to authenticate a user and a method handler for a new **Sign in** button. The user in the example code above is authenticated by using a Facebook login. A dialog is used to display the user ID once authenticated. 

   > [!NOTE]
> If you are using an identity provider other than Facebook, change the value passed to **LoginAsync** above to one of the following: _MicrosoftAccount_, _Twitter_, _Google_, or _WindowsAzureActiveDirectory_.
> 
2. In the **OnCreate** method, delete or comment-out the following line of code:

        OnRefreshItemsSelected ();
3. In the Activity_To_Do.axml file, add the following *LoginUser* button definition before the existing *AddItem* button:

          <Button
         android:id="@+id/buttonLoginUser"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:onClick="LoginUser"
         android:text="@string/login_button_text" />
4. Add the following element to the Strings.xml resources file:

        <string name="login_button_text">Sign in</string> 
5. In Visual Studio or Xamarin Studio, run the client project on a device or emulator and sign in with your chosen identity provider. 

       When you are successfully logged-in, the app will display your login ID and the list of todo items, and you can make updates to the data.



<!-- URLs. -->

[Create a Xamarin.Android app]: app-service-mobile-xamarin-android-get-started.md

