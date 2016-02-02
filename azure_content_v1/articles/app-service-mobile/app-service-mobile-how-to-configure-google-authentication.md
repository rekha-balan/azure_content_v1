<properties
    pageTitle="How to configure Google authentication for your App Services application"
    description="Learn how to configure Google authentication for your App Services application."
    services="app-service\mobile"
    documentationCenter=""
    authors="mattchenderson" 
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

# How to configure your App Service application to use Google login
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


This topic shows you how to configure Azure App Service to use Google as an authentication provider.

To complete the procedure in this topic, you must have a Google account that has a verified email address. To create a new Google account, go to [accounts.google.com](http://go.microsoft.com/fwlink/p/?LinkId=268302).

> [!NOTE]
> This topic demonstrates use of the App Service Authentication / Authorization feature. This replaces the App Service gateway for most applications. Differences that apply to using the gateway are called out in notes throughout the topic.
> 
> 
## <a name="register"> </a>Register your application with Google
1. Log on to the [Azure portal](https://portal.azure.com/), and navigate to your application. Copy your **URL**. You will use this to configure your Google app.

2. Navigate to the [Google apis](http://go.microsoft.com/fwlink/p/?LinkId=268303) website, sign-in with your Google account credentials, click **Create Project**, provide a **Project name**, then click **Create**.

3. In the left navigation bar, click **API & Auth**, then under **Social APIs** click **Google+ API** > **Enable API**.

4. Click **API & Auth** > **Credentials** > **OAuth consent screen**, then select your **Email address**,  enter a **Product Name**, and click **Save**.

5. In the **Credentials** tab, click **Add credentials** > **OAuth 2.0 client ID**, then select **Web application**.

6. Paste the App Service **URL** you copied earlier into **Authorized JavaScript Origins**, then paste the **Redirect URI** you copied earlier into **Authorized Redirect URI**. The redirect URI is the URL of your application appended with the path, */.auth/login/google/callback*. For example, `https://contoso.azurewebsites.net/.auth/login/google/callback`. Make sure that you are using the HTTPS scheme. Then click **Create**.


    > [AZURE.NOTE]
    If you are using the App Service Gateway instead of the App Service Authentication / Authorization feature, your redirect URL instead uses the gateway URL with the _/signin-google_ path.


1. On the next screen, make a note of the values of the client ID and client secret.

    > [AZURE.IMPORTANT]
    The client secret is an important security credential. Do not share this secret with anyone or distribute it within a client application.


## <a name="secrets"> </a>Add Google information to your application
> [!NOTE]
> If using the App Service Gateway, ignore this section and instead navigate to your gateway in the portal. Select **Settings**, **Identity**, and then **Google**. Paste in the values you obtained earlier and click **Save**.
> 
> 
1. Back in the [Azure portal](https://portal.azure.com/), navigate to your application. Click **Settings**, and then **Authentication / Authorization**.

2. If the Authentication / Authorization feature is not enabled, turn the switch to **On**.

3. Click **Google**. Paste in the App ID and App Secret values which you obtained previously, and optionally enable any scopes your application requires. Then click **OK**.

   ![][1]

   By default, App Service provides authentication but does not restrict authorized access to your site content and APIs. You must authorize users in your app code. 

4. (Optional) To restrict access to your site to only users authenticated by Google, set **Action to take when request is not authenticated** to **Google**. This requires that all requests be authenticated, and all unauthenticated requests are redirected to Google for authentication.

5. Click **Save**. 


You are now ready to use Google for authentication in your app.

## <a name="related-content"> </a>Related Content
Add authentication to your Mobile App: [iOS][ios-get-started-users], [Xamarin.iOS][xamarin-ios-get-started-users], [Xamarin.Android][xamarin-android-get-started-users], [Windows Universal][windows-get-started-users].


[windows-get-started-users]: ../article/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-users.md
[xamarin-ios-get-started-users]: ../article/app-service-mobile/app-service-mobile-xamarin-ios-get-started-users.md
[xamarin-android-get-started-users]: ../article/app-service-mobile/app-service-mobile-xamarin-android-get-started-users.md
[ios-get-started-users]: ../article/app-service-mobile/app-service-mobile-ios-get-started-users.md


<!-- Anchors. -->

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-settings.png

<!-- URLs. -->

[Google apis]: http://go.microsoft.com/fwlink/p/?LinkId=268303

[Azure portal]: https://portal.azure.com/

