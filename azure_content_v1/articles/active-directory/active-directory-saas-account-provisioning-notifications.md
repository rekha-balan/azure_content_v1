<properties
    pageTitle="Account provisioning notifications | Microsoft Azure"
    description="Learn how to ensure that you are notified of issues related to user provisioning that require your attention by enabling account provisioning notifications."
    services="active-directory"
    documentationCenter=""
    authors="markusvi"
    manager="stevenpo"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2016"
    ms.author="markusvi"/>


# Account Provisioning Notifications
With user provisioning, you can automate the process of managing users in third party SaaS applications. <br>
While this is an automated process, your interaction with this process is from time to time required. <br>
This is, for example the case, when the password of the account you have configured to exchange data with a third party SaaS application has expired. 

By enabling account provisioning notifications, you can ensure that you are notified of issues related to user provisioning that require your attention.

You activate or deactivate account provisioning notifications as part of your user provisioning configuration for a third party SaaS application.

![User Provisioning][1] 

To activate account provisioning notifications, select the related checkbox on the **Confirmation** dialog page, and then type the email alias of the recipient.

![Account Provisioning Notifications][2]

You can enter a distribution list as recipient; however, it is important to note that the notification email contains links to reports that are only accessible by the Azure AD administrators.

If you have account provisioning notifications enabled, you will receive emails about critical issues that are related to user provisioning. 
 However, to avoid an email overload, you will only receive one notification email per day for each SaaS application the notification email is enabled for.

##Related Articles

This article is part of a series on how to manage SaaS applications with Azure Active Directory. Below are all of the articles in the series:

- [Introduction to Single Sign-On and Managing App Access with Azure Active Directory](active-directory-appssoaccess-whatis.md)
	- [Managing Certificates for Federated Single Sign-On in Azure Active Directory](active-directory-sso-certs.md)
	- [Introduction to the Access Panel](active-directory-saas-access-panel-introduction.md)
	- [How to Deploy the Access Panel Extension for Internet Explorer using Group Policy](active-directory-saas-ie-group-policy.md)
	- [Troubleshooting the Access Panel Extension for Internet Explorer](active-directory-saas-ie-troubleshooting.md)
- [Automate User Provisioning and Deprovisioning to SaaS Applications](active-directory-saas-app-provisioning.md)
	- [Customizing Attribute Mappings for User Provisioning](active-directory-saas-customizing-attribute-mappings.md)
	- [Writing Expressions for Attribute Mappings](active-directory-saas-writing-expressions-for-attribute-mappings.md)
	- [Scoping Filters for User Provisioning](active-directory-saas-scoping-filters.md)
	- [Account Provisioning Notifications](active-directory-saas-account-provisioning-notifications.md)
- [List of Tutorials on How to Integrate SaaS Apps](active-directory-saas-tutorial-list.md)
	- [How to integrate Salesforce](active-directory-saas-salesforce-tutorial.md)
	- [How to integrate Google Apps](active-directory-saas-google-apps-tutorial.md)
	- [How to integrate Box](active-directory-saas-box-tutorial.md)
	- [How to integrate ServiceNow](active-directory-saas-servicenow-tutorial.md)
	- [How to integrate Dropbox for Business](active-directory-saas-dropboxforbusiness-tutorial.md)
	- [How to integrate Workday](active-directory-saas-workday-tutorial.md) 
	- [More SaaS App Tutorials...](active-directory-saas-tutorial-list.md)

<!--Image references-->

[1]: ./media/active-directory-saas-account-provisioning-notifications/ic766307.png
[2]: ./media/active-directory-saas-account-provisioning-notifications/ic766308.png