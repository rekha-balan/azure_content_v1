<properties
    pageTitle="Azure AD service limits and restrictions"
    description="Usage constraints and other service limits for the Azure Active Directory service."
    services="active-directory"
    documentationCenter=""
    authors="curtand"
    manager="stevenpo"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="01/05/2016"
    ms.author="curtand"/>

# Azure AD service limits and restrictions
If you’re looking for the full set of Microsoft Azure service limits, see [Azure Subscription and Service Limits, Quotas, and Constraints](azure-subscription-service-limits.md).

Here are the usage constraints and other service limits for the Azure Active Directory service.

| Category | Limits |
|---|---|
| Directories | A single user can only be associated with a maximum of 20 Azure Active Directory directories.<br />Examples of possible combinations: <ul> <li>A single user creates 20 directories.</li><li>A single user is added to 20 directories as a member.</li><li>A single user creates 10 directories and later is added by others to 10 different directories.</li></ul> |  
| Objects | <ul><li>A maximum of 500,000 objects can be used in a single directory by users of the Free edition of Azure Active Directory.</li><li>A non-admin user can create no more than 250 objects.</li></ul> |
| Schema extensions | <ul><li>String type extensions can have maximum of 256 characters. </li><li>Binary type extensions are limited to 256 bytes.</li><li>100 extension values (across ALL types and ALL applications) can be written to any single object.</li><li>Only “User”, “Group”, “TenantDetail”, “Device”, “Application” and “ServicePrincipal” entities can be extended with “String” type or “Binary” type single-valued attributes.</li><li>Schema extensions are available only in Graph API-version 1.21-preview. The application must be granted write access to register an extension.</li></ul> |
| Applications | A maximum of 10 users can be owners of a single application. |
| Groups | <ul><li>A maximum of 10 users can be owners of a single group.</li><li>Any number of objects can be members of a single group in Azure Active Directory.</li><li>The number of members in a group you can synchronize from your on-premises Active Directory to Azure Active Directory is limited to 15K members, using Azure Active Directory Directory Synchronization (DirSync).</li><li>The number of members in a group you can synchronize from your on-premises Active Directory to Azure Active Directory using Azure AD Connect is limited to 50K members.</li></ul> |
| Access Panel | <ul><li>There is no limit to the number of applications that can be seen in the Access Panel per end user, for users assigned licenses for Azure AD Premium or the Enterprise Mobility Suite.</li><li>A maximum of 10 app tiles (examples: Box, Salesforce, or Dropbox) can be seen in the Access Panel for each end user for users assigned licenses for Free or Azure AD Basic editions of Azure Active Directory. This limit does not apply to Administrator accounts.</li></ul> |
| Reports | A maximum of 1,000 rows can be viewed or downloaded in any report. Any additional data is truncated. |


## What's next
* [Sign up for Azure as an organization](sign-up-organization.md)
* [How Azure subscriptions are associated with Azure AD](active-directory-how-subscriptions-associated-directory.md)
* [Azure AD terminology](active-directory-terminology.md)

