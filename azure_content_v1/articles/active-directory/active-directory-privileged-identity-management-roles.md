<properties
   pageTitle="Azure Privileged Identity Management: Roles"
   description="Learn what roles are used for privileged identities with the Azure Privileged Identity Management extension."
   services="active-directory"
   documentationCenter=""
   authors="kgremban"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="01/21/2016"
   ms.author="kgremban"/>

# Azure Privileged Identity Management: Roles
<!-- **PLACEHOLDER: Need description of how this works. Azure PIM uses roles from MSODS objects.**-->

## Roles from Azure Active Directory, Office 365, and other sources
Azure Privileged Identity Management (PIM) uses the following roles as default administrator roles:

* Global Administrator
* Billing Administrator
* Service Administrator
* User Administrator
* Password Administrator

For more information about roles from Office 365, Exchange Online, SharePoint Online and Skype for Business go to [Assigning admin roles in Office 365](https://support.office.com/en-us/article/Assigning-admin-roles-in-Office-365-eac4d046-1afd-4f1a-85fc-8219c79e1504?ui=en-USrs=en-USad=US).

<!--**PLACEHOLDER: The above article may not be the one we want since PIM gets roles from places other that Office 365**-->


<!-- ## The PIM Security Administrator Role **PLACEHOLDER: Need description of the Security Administrator role.**-->

## User Roles and Logging In
> [!NOTE]
> In order for a user to be able to sign in to Azure PIM, they must have a license for Azure.
> 
> 
## Assigning a License to a User in Azure AD
> [!NOTE]
> The license option will only show up if licenses actually exit for this subscription.
> 
> 
1. With a global administrator account or a co-administrator account, sign in to [http://manage.windowsazure.com](http://manage.windowsazure.com).
2. Click on **All Items** in the main menu.
3. Select the directory you want to work with and that has licenses associated with it.
4. Click on **Licenses**. The list of available licenses will appear.
5. Click the license plan which contains the licenses that you want to distribute.
6. Click **Assign Users**.
7. Select the user that you want to assign a license to.
8. Click the **Assign** button.  The user can now log in to Azure.

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## Next steps
- [Getting Started with Azure Privileged Identity Management](active-directory-privileged-identity-management-getting-started.md)
- [Roles in Azure PIM](active-directory-privileged-identity-management-roles.md)
- [The Security Wizard](active-directory-privileged-identity-management-security-wizard.md)
- [How to Add or Remove a User Role](active-directory-privileged-identity-management-how-to-add-role-to-user.md)
- [How to Activate or Deactivate a Role](active-directory-privileged-identity-management-how-to-activate-role.md)
- [How to Change or View the Default Activation Settings for a Role](active-directory-privileged-identity-management-how-to-change-default-settings.md)
- [How to Configure Security Alerts](active-directory-privileged-identity-management-how-to-configure-security-alerts.md)
- [How to Start a Security Review](active-directory-privileged-identity-management-how-to-start-security-review.md)
- [How to Perform a Security Review](active-directory-privileged-identity-management-how-to-perform-security-review.md)
- [How to Require MFA](active-directory-privileged-identity-management-how-to-require-mfa.md)
- [How to Use the Audit Log](active-directory-privileged-identity-management-how-to-use-audit-log.md)


