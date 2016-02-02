<properties
    pageTitle="Azure Active Directory B2C preview: Application registration | Microsoft Azure"
    description="How to register your application with Azure Active Directory B2C"
    services="active-directory-b2c"
    documentationCenter=""
    authors="swkrish"
    manager="mbaldwin"
    editor="bryanla"/>

<tags
    ms.service="active-directory-b2c"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/22/2015"
    ms.author="swkrish"/>

# Azure Active Directory B2C preview: how to Register your Application
> [AZURE.NOTE]
	This information applies to the Azure AD B2C consumer identity service preview.  For information on Azure AD for employees and organizations, 
	please refer to the [Azure Active Directory Developer Guide](active-directory-developers-guide.md).

## Pre-requisite
To build an application that accepts consumer sign up & sign in, you'll first need to register it with an Azure Active Directory B2C tenant. Get your own tenant using the steps outlined [here](active-directory-b2c-get-started.md). If you followed all the steps in that article, you should have the B2C features blade pinned to your Startboard.

> [AZURE.IMPORTANT]
You cannot use applications registered in the **Applications** tab on the classic [Azure Management Portal](https://manage.windowsazure.com/) for this.

## Navigate to the B2C Features Blade
You can navigate to the B2C features blade from either the Azure Portal or the Azure Classic Portal.

### 1. Directly on the Azure Portal
If you have the B2C features blade pinned to your Startboard, you will see it as soon as you sign in to the [Azure Portal](https://portal.azure.com/) as the Global Administrator of the B2C tenant.

You can also access the blade by clicking on **Browse** and then **Azure AD B2C** in the left hand navigation on the [Azure Portal](https://portal.azure.com/).

You can also access it directly by navigating to [https://portal.azure.com/{tenant}.onmicrosoft.com/?#blade/Microsoft_AAD_B2CAdmin/TenantManagementBlade/id/](https://portal.azure.com/{tenant}.onmicrosoft.com/?#blade/Microsoft_AAD_B2CAdmin/TenantManagementBlade/id/.md) where **{tenant}** is to be replaced by the name used at tenant creation time (for example, contosob2c). You can bookmark this link for future use.

    > [AZURE.IMPORTANT]
    You need to be a Global Administrator of the B2C tenant to be able to access the B2C features blade. A Global Administrator from any other tenant or a User from any tenant cannot access it.

### 2. Access via the Azure Classic Portal
Sign in to the [Azure Classic Portal](https://manage.windowsazure.com/) as the Subscription Administrator (this is the same work or school account or the same Microsoft Account that you used to sign up for Azure). Navigate to the Active Directory extension on the left and click on your B2C tenant. On the **Quick Start** tab (the first tab that opens up), click on **Manage B2C settings** under **Administer**. This will open up the B2C features blade in a new browser window or tab.

You can also find the **Manage B2C settings** link (in the **B2C Administration** section) on the **Configure** tab.

## Register an Application
1. On the B2C features blade on the Azure Portal, click on **Applications**.
2. Click **+Add** at the top of the blade.
3. The **Name** of the application will describe your application to consumers. For example, enter "Contoso B2C app".
4. If you are writing a web-based application, toggle the **Include web app / web API** switch to **Yes**. The **Reply URLs** are endpoints where Azure AD B2C will return any tokens your application requests. For example, enter `https://localhost:44321/`. If your application includes a server-side component (API) that needs to be secured, you'll want to create (and copy) an **Application Secret** as well by clicking the **Generate Key** button.

   > [!NOTE]
>  **Application Secret** is an important security credential.
> 
5. If you are writing a mobile application, toggle the **Include native client** switch to **Yes**. Copy down the default **Redirect URI** automatically created for you.

6. Click **Create** to register your application.
7. Click on the application that you just created and copy down the globally unique **Application ID** that you'll use later in your code.

## Build a Quick Start Application
Now that you have an application registered with Azure AD B2C, you can complete one of our quick start tutorials to get up & running. Here are a few recommendations:

| Mobile & Native Apps | Web Apps & Web APIs | Integrate Directly with Protocols |
| ----------------------- | ------------------------------- | --------------------- |
| [Add Sign Up & Sign In to an iOS App](active-directory-b2c-devquickstarts-ios.md) | Add Sign Up & Sign In to an AngularJS SPA (Coming Soon) | [Register an Application](active-directory-b2c-app-registration.md) |
| [Add Sign Up & Sign In to an Android App](active-directory-b2c-devquickstarts-android.md) | [Add Sign Up & Sign In to a .NET MVC App](active-directory-b2c-devquickstarts-web-dotnet.md)  | [Mobile Apps with OAuth 2.0](active-directory-b2c-reference-oauth-code.md) |
| Add Sign Up & Sign In to a Windows Universal App (Coming Soon) | [Add Sign Up & Sign In to a Node JS Web App](active-directory-b2c-devquickstarts-web-node.md) | [Web Apps with OpenID Connect](active-directory-b2c-reference-oidc.md) |
| Add Sign Up & Sign In to a Windows Desktop App (Coming Soon) | [Secure a .NET Web API](active-directory-b2c-devquickstarts-api-dotnet.md) | Single Page Apps with OpenID Connect (Coming Soon)
|  | [Secure a NodeJS Web API](active-directory-b2c-devquickstarts-api-node.md) | Server Side Daemons (Coming Soon) |
|  | [Call a Web API from a .NET Web App](active-directory-b2c-devquickstarts-web-api-dotnet.md) | Server Side Daemons (Coming Soon) |


