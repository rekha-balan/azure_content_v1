<properties
    pageTitle="App Model v2.0 | Microsoft Azure"
    description="How to register an  app with Microsoft for enabling sign-in and integrating apps with app model v2.0."
    services="active-directory"
    documentationCenter=""
    authors="dstrockis"
    manager="mbaldwin"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/09/2015"
    ms.author="dastrock"/>

# App model v2.0 preview: How to register an app with Microsoft
To build an app that accepts both MSA & Azure AD sign-in, you'll first need to register an app with Microsoft.  You won't be able to use any existing app you may have with Azure AD or MSA - it's time to create a brand new one.

> [!NOTE]
>     This information applies to the v2.0 app model public preview.  For instructions on how to integrate with the generally available Azure AD service, please refer to the [Azure Active Directory Developer Guide](active-directory-developers-guide.md).
> 
> 
## Visit the Microsoft  App Registration Portal
First things first - navigate to [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com).  This is the new app registration portal where you can manage anything & everything about your Microsoft apps.

Sign in with either a personal or work or school Microsoft account.  If you don't have either, sign up for a new personal account. Go ahead, it won't take long - we'll wait here.

Done? You should now be looking at your list of Microsoft apps, which is probably empty.  Let's change that.

<!-- TODO: Verify strings here -->
Click **Add an app**, and give it a name.  The portal will assign your app a
globally unique  Application Id that you'll use later in your code.  If your app includes a server-side component that needs access tokens for calling APIs
(think: Office, Azure, or 3rd party), you'll want to create an ** Application
Secret** here as well.
<!-- TODO: Link for app secrets -->

Next, add the Platforms that your app will use.

* For web based apps, provide a **Redirect URI** where sign-in messages can be sent.
* For mobile apps, copy down the default redirect uri automatically created for you.

Optionally, you can customize the look and feel of your sign-in page in the Profile section.  Make sure to click **Save** before moving on.

## Build a Quick Start App
Now that you have a Microsoft app, you can complete one of our quick start
tutorials to get up & running with app model v2.0.  Here are a few
recommendations:

| Mobile & Native Apps | Web Apps & Web APIs | Integrate Directly with Protocols |
| ----------------------- | ------------------------------- | --------------------- |
| Add Sign-In to an iOS App (Coming Soon) | [Add Sign-In to an AngularJS SPA (NodeJS)](active-directory-v2-devquickstarts-angular-node.md) | [Register an Application](active-directory-v2-app-registration.md) |
| Add Sign-In to an Android App (Coming Soon) | [Add Sign-In to an AngularJS SPA (.NET)](active-directory-v2-devquickstarts-angular-dotnet.md) | [Mobile Apps with OAuth 2.0](active-directory-v2-protocols-oauth-code.md) |
| Add Sign-In to a Windows Universal App (Coming Soon) | [Add Sign-In to a .NET MVC App](active-directory-v2-devquickstarts-dotnet-web.md) | [Web Apps with OpenID Connect](active-directory-v2-protocols-oidc.md) |
| [Add Sign-In to a Windows Desktop App](active-directory-v2-devquickstarts-wpf.md)| [Add Sign-In to a Node JS Web App](active-directory-v2-devquickstarts-node-web.md) | [Single Page Apps with OpenID Connect](active-directory-v2-protocols-implicit.md) 
| [Call Office 365 Rest APIs from an app](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2) | [Secure a .NET Web API](active-directory-v2-devquickstarts-dotnet-api.md) | Server Side Daemons (Coming Soon) |
|  |  [Secure a NodeJS Web API](active-directory-v2-devquickstarts-node-api.md) |
|  | [Call Office 365 REST APIs from the web](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2) |


