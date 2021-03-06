<properties
    pageTitle="Azure AD Privileged Identity Management"
    description="A topic that explains what Azure AD Privileged Identity management is and how to configure it."
    services="active-directory"
    documentationCenter=""
    authors="kgremban"
    manager="stevenpo"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/21/2016"
    ms.author="kgremban"/>

# Azure AD Privileged Identity Management
Azure Active Directory (AD) Privileged Identity Management lets you manage, control, and monitor your privileged identities and their access to resources in Azure AD as well as other Microsoft online services like Office 365 or Microsoft Intune.  

To enable users to carry out privileged operations, organizations often need to give many of their users permanent privileged access in Azure AD, Azure or Office 365 resources, or other SaaS apps. For many customers, this is a growing security risk for their cloud-hosted resources because they cannot sufficiently monitor what those users are doing with their admin privileges. In addition, a compromised user account that has privileged access could impact their overall cloud security. Azure AD Privileged Identity Management helps to resolve this risk.  

Azure AD Privileged Identity Management lets you:  

* Discover which users are the Azure AD admins
* Enable on-demand, "just in time" administrative access to directory resources
* Get reports about administrator access history and about changes in administrator assignments
* Get alerts about access to a privileged role

Azure AD Privileged Identity Management can manage the built-in Azure Active Directory organizational roles:  

* Global Administrator
* Billing Administrator
* Service Administrator  
* User Administrator
* Password Administrator

## Just in time administrator access
Historically, you could assign a user to an admin role through the Azure Management Portal or Windows PowerShell. As a result, that user becomes a **permanent admin**, always active in his or her assigned role. This preview adds support for a **temporary admin**, which is a user who needs to complete an activation process for the assigned role.  The activation process changes the assignment of the user to a role in Azure AD from inactive to active.

## Enabling Privileged Identity Management for your directory
You can start using Azure AD Privileged Identity Management by accessing the [Azure portal](https://portal.azure.com/). For now, Azure AD Privileged Identity Management only appears in the Azure portal.  It does not appear in the classic portal. You must be a global administrator to enable Azure AD Privileged Identity Management for a directory.

![Azure portal - search for privileged identities - screenshot][1]

After [initializing the extension](active-directory-privileged-identity-management-getting-started.md), you will automatically become the first **Security administrator** of the directory. Only a security administrator can access this extension to manage the access for other administrators.  

During initialization, a tile of Azure AD Privileged Identity Management will be added to the start board of the Azure portal.

## Privileged Identity Management dashboard
Azure AD Privileged Identity Manager provides a dashboard which gives you important information such as:

* The number of users who are assigned to each privileged role  
* The number of temporary and permanent admins
* The administrator's access history

![PIM dashboard - screenshot][2]

## Privileged role management
With Azure AD Privileged Identity Management, you can manage the administrators by adding or removing permanent or temporary administrators to each role.

![PIM add/remove administrators - screenshot][3]

## Configure the role activation settings
Using the role activation setting you can configure the temporary role activation properties including:

* The duration of the role activation period
* The role activation notification
* The information a user needs to provide during the role activation process  

![PIM settings - administrator activation - screenshot][4]

## Role activation
In order to activate a role, a temporary admin needs to request a time-bound "activation" for the role. The activation can be requested using the **Activate my role** option in Azure AD Privileged Identity Management.

An admin who wants to activate a role needs to initialize Azure AD Privileged Identity Management in the Azure portal.

Any type of admin can use Azure AD Privileged Identity Management to activate his or her role.

Role activation is time-bound. In the Role Activation settings, you can configure the length of the activation as well as the required information that the admin needs to provide in order to activate the role.

![PIM administrator request role activation - screenshot][5]

## Role activation history
Using Azure AD Privileged Identity Management, you can also track changes in privileged role assignments and role activation history. This can be done using the audit log options:

![PIM activation history - screenshot][6]

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


<!--Image references-->

[1]: ./media/active-directory-privileged-identity-management-configure/Search_PIM.png
[2]: ./media/active-directory-privileged-identity-management-configure/PIM_Dash.png
[3]: ./media/active-directory-privileged-identity-management-configure/PIM_AddRemove.png
[4]: ./media/active-directory-privileged-identity-management-configure/PIM_RoleActivationSettings.png
[5]: ./media/active-directory-privileged-identity-management-configure/PIM_RequestActivation.png
[6]: ./media/active-directory-privileged-identity-management-configure/PIM_ActivationHistory.png
