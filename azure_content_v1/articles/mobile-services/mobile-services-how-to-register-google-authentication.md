<properties
    pageTitle="Register for Google authentication | Microsoft Azure"
    description="Learn how to register your apps to use Google to authenticate with Azure Mobile Services."
    services="mobile-services"
    documentationCenter="android"
    authors="ggailey777"
    manager="dwrede"
    editor=""/>


<tags 
    ms.service="mobile-services" 
    ms.workload="mobile" 
    ms.tgt_pltfrm="mobile-android" 
    ms.devlang="multiple" 
    ms.topic="article" 
    ms.date="11/30/2015" 
    ms.author="glenga"/>

# Register your apps for Google login with Mobile Services
>[AZURE.NOTE] This is an **Azure Mobile Services** topic.  Microsoft Azure recommends Azure App Service Mobile Apps for all new mobile backend deployments.
To get started with Azure App Service Mobile Apps, see the [App Service Mobile Apps documentation center](/documentation/services/app-service/mobile).


&nbsp;

> [AZURE.SELECTOR]
- [Azure Active Directory](../articles/mobile-services/mobile-services-how-to-register-active-directory-authentication.md)
- [Facebook](../articles/mobile-services/mobile-services-how-to-register-facebook-authentication.md)
- [Google](../articles/mobile-services/mobile-services-how-to-register-google-authentication.md)
- [Microsoft account](../articles/mobile-services/mobile-services-how-to-register-microsoft-authentication.md)
- [Twitter](../articles/mobile-services/mobile-services-how-to-register-twitter-authentication.md)


This topic shows you how to register your apps to be able to use Google to authenticate with Azure Mobile Services.

> [!NOTE]
> This tutorial is about [Azure Mobile Services](https://azure.microsoft.com/services/mobile-services/), a solution to help you build scalable mobile applications for any platform. Mobile Services makes it easy to sync data, authenticate users, and send push notifications. This page supports the [Get Started with Authentication](mobile-services-ios-get-started-users.md) tutorial, which shows how to sign in users to your app.
> <br/>If this is your first experience with Mobile Services, please complete the tutorial [Get Started with Mobile Services](mobile-services-ios-get-started.md).
> 
> 
To complete the procedure in this topic, you must have a Google account that has a verified email address. To create a new Google account, go to <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>.

1. Navigate to the [Google apis](http://go.microsoft.com/fwlink/p/?LinkId=268303) website, sign-in with your Google account credentials, click **Create Project**, provide a **Project name**, then click **Create**.

2. In the **Products & services** drop down menu, click **API Manager**, then under **Social APIs** click **Google+ API** > **Enable API**.

3. Click **Credentials** > **OAuth consent screen**, then select your **Email address**,  enter a **Product Name**, and click **Save**.

4. In the **Credentials** tab, click **Add credentials** > **OAuth 2.0 client ID**, then select **Web application**.


1. Type your mobile service URL in **Authorized JavaScript Origins**, replace the generated URL in **Authorized Redirect URI** with one of the following URL formats, and then click **Create**:

    + **.NET backend**: `https://<mobile_service>.azure-mobile.net/signin-google`
    + **JavaScript backend**: `https://<mobile_service>.azure-mobile.net/login/google`

     >[AZURE.NOTE]Make sure that you use the correct redirect URL path format for your type of Mobile Services backend. When this is incorrect, authentication will not succeed.

1. On the next screen, make a note of the values of the client ID and client secret.

   > [!IMPORTANT]
> The client secret is an important security credential. Do not share this secret with anyone or distribute it within a client application.
> 
> 

You are now ready to configure your mobile service to use Google sign-in for authentication in your app.

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->

[Google apis]: http://go.microsoft.com/fwlink/p/?LinkId=268303
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-dotnet/
