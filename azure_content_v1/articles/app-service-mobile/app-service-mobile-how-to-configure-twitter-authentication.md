<properties
    pageTitle="How to configure Twitter authentication for your App Services application"
    description="Learn how to configure Twitter authentication for your App Services application."
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
    ms.date="01/13/2016"
    ms.author="mahender"/>

# How to configure your App Service application to use Twitter login
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


This topic shows you how to configure Azure App Service to use Twitter as an authentication provider.

To complete the procedure in this topic, you must have a Twitter account that has a verified email address. To create a new Twitter account, go to <a href="http://go.microsoft.com/fwlink/p/?LinkID=268287" target="_blank">twitter.com</a>.

> [!NOTE]
> This topic demonstrates use of the App Service Authentication / Authorization feature. This replaces the App Service gateway for most applications. Differences that apply to using the gateway are called out in notes throughout the topic.
> 
> 
## <a name="register"> </a>Register your application with Twitter
1. Log on to the [Azure portal](https://portal.azure.com/), and navigate to your application. Copy your **URL**. You will use this to configure your Twitter app.

2. Navigate to the [Twitter Developers](http://go.microsoft.com/fwlink/p/?LinkId=268300) website, sign-in with your Twitter account credentials, and click **Create New App**.

3. Type in the **Name** and a **Description** for your new app. Paste in your application's **URL** for the **Website** value. Then, for the **Callback URL**, paste the **Callback URL** you copied earlier. This is your Mobile App gateway appended with the path, */.auth/login/twitter/callback*. For example, `https://contoso.azurewebsites.net/.auth/login/twitter/callback`. Make sure that you are using the HTTPS scheme.

   > [!NOTE]
>  If you are using the App Service Gateway instead of the App Service Authentication / Authorization feature, your redirect URL instead uses the gateway URL with the _/signin-twitter_ path.
> 
4. At the bottom the page, read and accept the terms. Then click **Create your Twitter application**. This registers the app displays the application details.

5. Click the **Settings** tab, check **Allow this application to be used to sign in with Twitter**, then click **Update Settings**.

6. Select the **Keys and Access Tokens** tab. Make a note of the values of **Consumer Key (API Key)** and **Consumer secret (API Secret)**.

   > [!NOTE]
> The consumer secret is an important security credential. Do not share this secret with anyone or distribute it with your app.
> 
> 
> 

## <a name="secrets"> </a>Add Twitter information to your application
> [!NOTE]
> If using the App Service Gateway, ignore this section and instead navigate to your gateway in the portal. Select **Settings**, **Identity**, and then **Twitter**. Paste in the values you obtained earlier and click **Save**.
> 
> 
1. Back in the [Azure portal](https://portal.azure.com/), navigate to your application. Click **Settings**, and then **Authentication / Authorization**.

2. If the Authentication / Authorization feature is not enabled, turn the switch to **On**.

3. Click **Twitter**. Paste in the App ID and App Secret values which you obtained previously. Then click **OK**.

   ![][1]

   By default, App Service provides authentication but does not restrict authorized access to your site content and APIs. You must authorize users in your app code. 

4. (Optional) To restrict access to your site to only users authenticated by Twitter, set **Action to take when request is not authenticated** to **Twitter**. This requires that all requests be authenticated, and all unauthenticated requests are redirected to Twitter for authentication.

5. Click **Save**. 


You are now ready to use Twitter for authentication in your app.

## <a name="related-content"> </a>Related Content
Add authentication to your Mobile App: [iOS][ios-get-started-users], [Xamarin.iOS][xamarin-ios-get-started-users], [Xamarin.Android][xamarin-android-get-started-users], [Windows Universal][windows-get-started-users].


[windows-get-started-users]: ../article/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-users.md
[xamarin-ios-get-started-users]: ../article/app-service-mobile/app-service-mobile-xamarin-ios-get-started-users.md
[xamarin-android-get-started-users]: ../article/app-service-mobile/app-service-mobile-xamarin-android-get-started-users.md
[ios-get-started-users]: ../article/app-service-mobile/app-service-mobile-ios-get-started-users.md


<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-twitter-authentication/app-service-twitter-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-twitter-authentication/mobile-app-twitter-settings.png

<!-- URLs. -->

[Twitter Developers]: http://go.microsoft.com/fwlink/p/?LinkId=268300
[Azure portal]: https://portal.azure.com/
[xamarin]: ../app-services-mobile-app-xamarin-ios-get-started-users.md