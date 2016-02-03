<properties 
    pageTitle="Add authentication to your universal Windows 8.1 app | Azure Mobile Services"
    description="Learn how to use Mobile Services to authenticate users of your Windows Store app through a variety of identity providers, including Google, Facebook, Twitter, and Microsoft." 
    services="mobile-services" 
    documentationCenter="windows" 
    authors="ggailey777" 
    manager="dwrede" 
    editor=""/>

<tags 
    ms.service="mobile-services" 
    ms.workload="mobile" 
    ms.tgt_pltfrm="mobile-windows-store" 
    ms.devlang="dotnet" 
    ms.topic="article" 
    ms.date="11/22/2015" 
    ms.author="glenga"/>

# Add authentication to your universal Windows 8.1 app
> [AZURE.SELECTOR-LIST (Platform | Backend )]
- [(iOS | .NET)](../articles/mobile-services-dotnet-backend-ios-get-started-users.md)
- [(iOS | JavaScript)](../articles/mobile-services-ios-get-started-users.md)
- [(Windows Runtime 8.1 universal C# | .NET)](../articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users.md)
- [(Windows Runtime 8.1 universal C# | Javascript)](../articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-users.md)
- [(Windows Phone Silverlight 8.x | Javascript)](../articles/mobile-services-windows-phone-get-started-users.md)
- [(Android | Javascript)](../articles/mobile-services-android-get-started-users.md)
- [(Xamarin.iOS | .NET)](../articles/mobile-services-dotnet-backend-xamarin-ios-get-started-users.md)
- [(Xamarin.iOS | Javascript)](../articles/partner-xamarin-mobile-services-ios-get-started-users.md)
- [(Xamarin.Android | .NET)](../articles/mobile-services-dotnet-backend-xamarin-android-get-started-users.md)
- [(Xamarin.Android | Javascript)](../articles/partner-xamarin-mobile-services-android-get-started-users.md)
- [(HTML | Javascript)](../articles/mobile-services-html-get-started-users.md)


This topic shows you how to authenticate users in Azure Mobile Services from your universal Windows 8.1 app. In this tutorial, you add authentication to the quickstart project using an identity provider that is supported by Mobile Services. After being successfully authenticated and authorized by Mobile Services, the user ID value is displayed.

This tutorial is based on the Mobile Services quickstart. You must also first complete the tutorial [Get started with Mobile Services](mobile-services-javascript-backend-windows-store-dotnet-get-started.md). 

> [!NOTE]
> This tutorial shows you how to authenticate users in Windows Store and Windows Phone Store 8.1 apps. For a Windows Phone 8.0 or Windows Phone Silverlight 8.1 app, see this version of [Get started with authentication in Mobile Services](mobile-services-windows-phone-get-started-users.md).
> 
> 
## <a name="register"></a> Register your app for authentication and configure Mobile Services

1. In the [Azure classic portal](https://manage.windowsazure.com/), click **Mobile Services** > your mobile service > **Dashboard**, and make a note of the **Mobile Service URL** value.

2. Register your app with one or more of the following authentication providers:
   * [Google](mobile-services-how-to-register-google-authentication.md)
   * [Facebook](mobile-services-how-to-register-facebook-authentication.md)
   * [Twitter](mobile-services-how-to-register-twitter-authentication.md)
   * [Microsoft](mobile-services-how-to-register-microsoft-authentication.md)
   * [Azure Active Directory](mobile-services-how-to-register-active-directory-authentication.md). 
   
    Make a note of the client identity and client secret values generated by the provider. Do not distribute or share the client secret.

3. Back in the [Azure classic portal](https://manage.windowsazure.com/), click **Mobile Services** > your mobile service > **Identity** > your identity provider settings, then enter the client ID and secret value from your provider. 
 
You've now configured both your app and your mobile service to work with your auth provider. You may optionally repeat all these steps for each additional identity provider you'd like to support.

> [AZURE.IMPORTANT] Verify that you've set the correct redirect URI on your identity provider's developer site. As described in the linked instructions for each provider above, the redirect URI may be different for a .NET backend service vs. for a JavaScript backend service. An incorrectly configured redirect URI may result in the login screen not being displayed properly and the app malfunctioning in unexpected ways.


## <a name="permissions"></a> Restrict permissions to authenticated users

1. In the Server Explorer in Visual Studio, expand the **Azure** node, **Mobile Services**, and your mobile service.

2. Right-click the **TodoItem** table, click **Edit Permissions**, set all permissions to **Only authenticated users**, and then click **Apply**. This ensures that all operations against the _TodoItem_ table require an authenticated user.

3. Right-click the client app project, click **Debug**, then **Start new instance**; verify that an unhandled exception with a status code of 401 (Unauthorized) is raised after the app starts.

	This happens because the app attempts to access Mobile Services as an unauthenticated user, but the *TodoItem* table now requires authentication.

Next, you will update the app to authenticate users before requesting resources from the mobile service.


> [!NOTE]
> When you use Visual Studio tools to connect your app to a Mobile Service, the tool generate two sets of **MobileServiceClient** definitions, one for each client platform. This is a good time to simplify the generated code by unifying the `#if...#endif` wrapped **MobileServiceClient** definitions into a single unwrapped definition used by both versions of the app. You won't need to do this if you downloaded the quickstart app from the [Azure classic portal](https://manage.windowsazure.com/).
> 
> 
## <a name="add-authentication"></a> Add authentication to the app

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

 

Now, any user authenticated by your trusted identity providers can access the *TodoItem* table. To better secure user-specific data, you must also implement authorization. To do this you get the user ID of a given user, which can then be used to determine what level of access that user should have for a given resource.

## <a name="tokens"></a>Store the authorization token on the client
The previous example showed a standard sign-in, which requires the client to contact both the identity provider and the mobile service every time that the app starts. Not only is this method inefficient, you can run into usage-related issues should many customers try to start your app at the same time. A better approach is to cache the authorization token returned by Mobile Services and try to use this first before using a provider-based sign-in. 

> [!NOTE]
> You can cache the token issued by Mobile Services regardless of whether you are using client-managed or service-managed authentication. This tutorial uses service-managed authentication.
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

## <a name="next-steps"> </a>Next steps
In the next tutorial, [Service-side authorization of Mobile Services users](mobile-services-javascript-backend-service-side-authorization.md), you will take the user ID value provided by Mobile Services based on an authenticated user and use it to filter the data returned by Mobile Services. 

## See also
* [Enhanced users feature](http://go.microsoft.com/fwlink/p/?LinkId=506605)<br/>
You can get additional user data maintained by the identity provider in your mobile service by by calling the **user.getIdentities()** function in server scripts. 

* [Mobile Services .NET How-to Conceptual Reference](mobile-services-windows-dotnet-how-to-use-client-library.md)<br/>Learn more about how to use Mobile Services with a .NET client.


<!-- Anchors. -->

[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Store authentication tokens on the client]: #tokens
[Next Steps]:#next-steps


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253

[Get started with Mobile Services]: mobile-services-javascript-backend-windows-store-dotnet-get-started.md
[Get started with authentication]: ../mobile-services-javascript-backend-windows-store-dotnet-get-started-users.md
[Get started with push notifications]: ../mobile-services-javascript-backend-windows-store-dotnet-get-started-push.md
[Authorize users with scripts]: ../mobile-services-windows-store-dotnet-authorize-users-in-scripts.md

[Azure classic portal]: https://manage.windowsazure.com/
[Mobile Services .NET How-to Conceptual Reference]: mobile-services-windows-dotnet-how-to-use-client-library.md
[Register your Windows Store app package for Microsoft authentication]: ../mobile-services-how-to-register-store-app-package-microsoft-authentication.md