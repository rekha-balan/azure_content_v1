<properties
    pageTitle="Add authentication to your Windows Runtime 8.1 universal app | Azure Mobile Apps"
    description="Learn how to use Azure App Service Mobile Apps to authenticate users of your Windows app using a variety of identity providers, including: AAD, Google, Facebook, Twitter, and Microsoft."
    services="app-service\mobile"
    documentationCenter="windows"
    authors="mattchenderson" 
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-windows"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="11/23/2015"
    ms.author="glenga"/>

# Add authentication to your Windows app
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


This topic shows you how to add cloud-based authentication to your mobile app. In this tutorial, you add authentication to the Mobile Apps quickstart project using an identity provider that is supported by Azure App Service. After being successfully authenticated and authorized by your Mobile App backend, the user ID value is displayed.

This tutorial is based on the Mobile Apps quickstart. You must first complete the tutorial [Get started with Mobile Apps](app-service-mobile-windows-store-dotnet-get-started.md). 

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



With one of the Windows app projects set as the start-up project, press the F5 key to run the app; verify that an unhandled exception with a status code of 401 (Unauthorized) is raised after the app starts. This happens because the app attempts to access your Mobile App Code as an unauthenticated user, but the *TodoItem* table now requires authentication.

Next, you will update the app to authenticate users before requesting resources from your App Service.

## <a name="add-authentication"></a>Add authentication to the app

1. Open the shared project file MainPage.cs and add the following code snippet to the MainPage class:
	
		// Define a member variable for storing the signed-in user. 
        private MobileServiceUser user;

        // Define a method that performs the authentication process
        // using a Facebook sign-in. 
        private async System.Threading.Tasks.Task<bool> AuthenticateAsync()
        {
            string message;
            bool success = false;
            try
            {
                // Change 'MobileService' to the name of your MobileServiceClient instance.
                // Sign-in using Facebook authentication.
                user = await App.MobileService
                    .LoginAsync(MobileServiceAuthenticationProvider.Facebook);
                message =
                    string.Format("You are now signed in - {0}", user.UserId);

                success = true;
            }
            catch (InvalidOperationException)
            {
                message = "You must log in. Login Required";
            }

            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
            return success;
        }

    This code authenticates the user with a Facebook login. If you are using an identity provider other than Facebook, change the value of **MobileServiceAuthenticationProvider** above to the value for your provider.

3. Comment-out or delete the call to the **RefreshTodoItems** method in the existing **OnNavigatedTo** method override.

	This prevents the data from being loaded before the user is authenticated. Next, you will add a **Sign in** button to the app that triggers authentication.

4. Add the following code snippet to the MainPage class:

        private async void ButtonLogin_Click(object sender, RoutedEventArgs e)
        {
            // Login the user and then load data from the mobile app.
            if (await AuthenticateAsync())
            {
                // Hide the login button and load items from the mobile app.
                ButtonLogin.Visibility = Windows.UI.Xaml.Visibility.Collapsed;
                //await InitLocalStoreAsync(); //offline sync support.
                await RefreshTodoItems();
            }
        }
		
5. In the Windows Store app project, open the MainPage.xaml project file and add the following **Button** element just before the element that defines the **Save** button:

		<Button Name="ButtonLogin" Click="ButtonLogin_Click" 
                        Visibility="Visible">Sign in</Button>

6. In the Windows Phone Store app project, add the following **Button** element in the **ContentPanel**, after the **TextBox** element:

        <Button Grid.Row ="1" Grid.Column="1" Name="ButtonLogin" Click="ButtonLogin_Click" 
        	Margin="10, 0, 0, 0" Visibility="Visible">Sign in</Button>

8. Open the shared App.xaml.cs project file and add the following code:

        protected override void OnActivated(IActivatedEventArgs args)
        {
			// Windows Phone 8.1 requires you to handle the respose from the WebAuthenticationBroker.
            #if WINDOWS_PHONE_APP
            if (args.Kind == ActivationKind.WebAuthenticationBrokerContinuation)
            {
				// Completes the sign-in process started by LoginAsync.
				// Change 'MobileService' to the name of your MobileServiceClient instance. 
                App.MobileService.LoginComplete(args as WebAuthenticationBrokerContinuationEventArgs);
            }
            #endif

            base.OnActivated(args);
        }

	If the **OnActivated** method already exists, just add the `#if...#endif` code block.

9. Press the F5 key to run the Windows Store app, click the **Sign in** button, and sign into the app with your chosen identity provider. 

   	When you are successfully logged-in, the app should run without errors, and you should be able to query your backend and make updates to data.

10. Right-click the Windows Phone Store app project, click **Set as StartUp Project**, then repeat the previous step to verify that the Windows Phone Store app also runs correctly.  

 

## <a name="tokens"></a>Store the authentication token on the client
The previous example showed a standard sign-in, which requires the client to contact both the identity provider and the App Service every time that the app starts. Not only is this method inefficient, you can run into usage-relates issues should many customers try to start you app at the same time. A better approach is to cache the authorization token returned by your App Service and try to use this first before using a provider-based sign-in. 

> [!NOTE]
> You can cache the token issued by App Services regardless of whether you are using client-managed or service-managed authentication. This tutorial uses service-managed authentication.
> 
> 

1. In the MainPage.xaml.cs project file, add the following **using** statements:

		using System.Linq;		
		using Windows.Security.Credentials;

2. Replace the **AuthenticateAsync** method with the following code:

        private async System.Threading.Tasks.Task<bool> AuthenticateAsync()
        {
            string message;
            bool success = false;

            // This sample uses the Facebook provider.
            var provider = "Facebook";

            // Use the PasswordVault to securely store and access credentials.
            PasswordVault vault = new PasswordVault();
            PasswordCredential credential = null;

            try
            {
                // Try to get an existing credential from the vault.
                credential = vault.FindAllByResource(provider).FirstOrDefault();
            }
            catch (Exception)
            {
                // When there is no matching resource an error occurs, which we ignore.
            }

            if (credential != null)
            {
                // Create a user from the stored credentials.
                user = new MobileServiceUser(credential.UserName);
                credential.RetrievePassword();
                user.MobileServiceAuthenticationToken = credential.Password;

                // Set the user from the stored credentials.
                App.MobileService.CurrentUser = user;

                // Consider adding a check to determine if the token is 
                // expired, as shown in this post: http://aka.ms/jww5vp.

                success = true;
			    message = string.Format("Cached credentials for user - {0}", user.UserId);
            }
            else
            {
                try
                {
                    // Login with the identity provider.
                    user = await App.MobileService
                        .LoginAsync(provider);

                    // Create and store the user credentials.
                    credential = new PasswordCredential(provider,
                        user.UserId, user.MobileServiceAuthenticationToken);
                    vault.Add(credential);

                    success = true;
                }
                catch (MobileServiceInvalidOperationException)
                {
                    message = "You must log in. Login Required";
                }
            }
            message = string.Format("You are now logged in - {0}", user.UserId);
            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();

            return success;
        }

	In this version of **AuthenticateAsync**, the app tries to use credentials stored in the **PasswordVault** to access the service. A regular sign-in is also performed when there is no stored credential.

	>[AZURE.NOTE]A cached token may be expired, and token expiration can also occur after authentication when the app is in use. To learn how to determine if a token is expired, see [Check for expired authentication tokens](http://aka.ms/jww5vp). For a solution to handling authorization errors related to expiring tokens, see the post [Caching and handling expired tokens in Azure Mobile Services managed SDK](http://blogs.msdn.com/b/carlosfigueira/archive/2014/03/13/caching-and-handling-expired-tokens-in-azure-mobile-services-managed-sdk.aspx). 

3. Restart the app twice.

	Notice that on the first start-up, sign-in with the provider is again required. However, on the second restart the cached credentials are used and sign-in is bypassed. 

## Next steps
Now that you completed this basic authentication tutorial, consider continuing on to one of the following tutorials:

* [Add push notifications to your Windows app](app-service-mobile-windows-store-dotnet-get-started-push.md)   
Learn how to add push notifications support to your app and configure your Mobile App backend to use Azure Notification Hubs to send push notifications.

* [Enable offline sync for your Windows app](app-service-mobile-windows-store-dotnet-get-started-offline-data.md)  
Learn how to add offline support your app using an Mobile App backend. Offline sync allows end-users to interact with a mobile app&mdash;viewing, adding, or modifying data&mdash;even when there is no network connection. 


<!-- URLs. -->

[Get started with your mobile app]: app-service-mobile-windows-store-dotnet-get-started.md

