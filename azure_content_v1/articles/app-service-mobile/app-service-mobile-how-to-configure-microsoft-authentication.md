<properties
    pageTitle="How to configure Microsoft Account authentication for your App Services application"
    description="Learn how to configure Microsoft Account authentication for your App Services application."
    authors="mattchenderson" 
    services="app-service\mobile"
    documentationCenter=""
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="na"
    ms.devlang="multiple"
    ms.topic="article"
    ms.date="11/20/2015"
    ms.author="mahender"/>

# How to configure your App Service application to use Microsoft Account login
> [AZURE.SELECTOR]
- [Azure Active Directory](../articles/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication.md)
- [Facebook](../articles/app-service-mobile/app-service-mobile-how-to-configure-facebook-authentication.md)
- [Google](../articles/app-service-mobile/app-service-mobile-how-to-configure-google-authentication.md)
- [Microsoft account](../articles/app-service-mobile/app-service-mobile-how-to-configure-microsoft-authentication.md)
- [Twitter](../articles/app-service-mobile/app-service-mobile-how-to-configure-twitter-authentication.md)


&nbsp;

>[AZURE.NOTE] This is an **Azure Mobile Apps** topic. For Mobile Services topics, see the [Mobile Services documentation center](/documentation/services/mobile-services/).
>
>App Service Mobile Apps is our newest mobile backend platform and [gives you additional advantages](app-service-mobile-value-prop-migration-from-mobile-services.md) over Mobile Services. [Migrating to App Service](app-service-mobile-migrating-from-mobile-services.md) is  recommended for customers using the .NET backend SDK. However, [Mobile Apps Node SDK](https://github.com/azure/azure-mobile-apps-node) is currently in Preview and is not yet recommended for production use. SDK and API contracts are subject to change within minor version releases.


This topic shows you how to configure Azure App Service to use Microsoft Account as an authentication provider.

> [!NOTE]
> This topic demonstrates use of the App Service Authentication / Authorization feature. This replaces the App Service gateway for most applications. Differences that apply to using the gateway are called out in notes throughout the topic.
> 
> 
## <a name="register"> </a>Register your application with Microsoft Account
1. Log on to the [Azure portal](https://portal.azure.com/), and navigate to your application. Copy your **URL**. You will use this to configure your Microsoft Account app.

2. Navigate to the [My Applications](http://go.microsoft.com/fwlink/p/?LinkId=262039) page in the Microsoft Account Developer Center, and log on with your Microsoft account, if required.

3. Click **Create application**, then type an **Application name** and click **I accept**.

4. Click **API Settings**. Select **Yes** for **Mobile or desktop client app**. In the **Redirect URL** field, enter your application's **Redirect URL** and click **Save**. Your redirect URI is the URL of your application appended with the path, */.auth/login/microsoftaccount/callback*. For example, `https://contoso.azurewebsites.net/.auth/login/microsoftaccount/callback`. Make sure that you are using the HTTPS scheme.

    ![][0]


    > [AZURE.NOTE]
    If you are using the App Service Gateway instead of the App Service Authentication / Authorization feature, your redirect URL instead uses the gateway URL with the _/signin-microsoft_ path.


1. Click **App Settings** and make a note of the values of the **Client ID** and **Client secret**.

    > [AZURE.NOTE] The client secret is an important security credential. Do not share the client secret with anyone or distribute it within a client application.


## <a name="secrets"> </a>Add Microsoft Account information to your application
> [!NOTE]
> If using the App Service Gateway, ignore this section and instead navigate to your gateway in the portal. Select **Settings**, **Identity**, and then **Microsoft Account**. Paste in the values you obtained earlier and click **Save**.
> 
> 
1. Back in the [Azure portal](https://portal.azure.com/), navigate to your application. Click **Settings**, and then **Authentication / Authorization**.

2. If the Authentication / Authorization feature is not enabled, turn the switch to **On**.

3. Click **Microsoft Account**. Paste in the App ID and App Secret values which you obtained previously, and optionally enable any scopes your application requires. Then click **OK**.

    ![][1]

    By default, App Service provides authentication but does not restrict authorized access to your site content and APIs. You must authorize users in your app code. 

4. (Optional) To restrict access to your site to only users authenticated by Microsoft account, set **Action to take when request is not authenticated** to **Microsoft Account**. This requires that all requests be authenticated, and all unauthenticated requests are redirected to Microsoft account for authentication.

5. Click **Save**. 


You are now ready to use Microsoft Account for authentication in your app.

## <a name="related-content"> </a>Related Content
Add authentication to your Mobile App: [iOS][ios-get-started-users], [Xamarin.iOS][xamarin-ios-get-started-users], [Xamarin.Android][xamarin-android-get-started-users], [Windows Universal][windows-get-started-users].


[windows-get-started-users]: ../article/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-users.md
[xamarin-ios-get-started-users]: ../article/app-service-mobile/app-service-mobile-xamarin-ios-get-started-users.md
[xamarin-android-get-started-users]: ../article/app-service-mobile/app-service-mobile-xamarin-android-get-started-users.md
[ios-get-started-users]: ../article/app-service-mobile/app-service-mobile-ios-get-started-users.md


<!-- Authenticate your app with Live Connect Single Sign-On: [Windows](windows-liveconnect) -->



<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/app-service-microsoftaccount-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/mobile-app-microsoftaccount-settings.png

<!-- URLs. -->

[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Azure portal]: https://portal.azure.com/
