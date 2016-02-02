<properties 
    pageTitle="Configure a custom domain name for a web app in Azure App Service that uses Traffic Manager"wpickett"Use a custom domain name for an a web app in Azure App Service that includes Traffic Manager for load balancing." 
    description="Use a custom domain name for an a web app in Azure App Service that includes Traffic Manager for load balancing." 
    services="app-service\web" 
    documentationCenter="" 
    authors="rmcmurray" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="app-service-web" 
    ms.workload="web" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="01/09/2016" 
    ms.author="robmcm"/>

# Configuring a custom domain name for a web app in Azure App Service using Traffic Manager
> [AZURE.SELECTOR]
- [Buy Domain for Web Apps](../article/app-service-web/custom-dns-web-site-buydomains-web-app.md)
- [Web Apps with External Domains](../article/app-service-web/web-sites-custom-domain-name.md)
- [Web Apps with Traffic Manager](../article/app-service-web/web-sites-traffic-manager-custom-domain-name.md)
- [GoDaddy](../article/app-service-web/web-sites-godaddy-custom-domain-name.md)

When you use a Microsoft Azure Traffic Manager to load balance traffic to your Azure Website, that website can then be accessed using the **\*.trafficmanager.net** domain name assigned by Azure. You can also associate a custom domain name, such as www.contoso.com, with your website in order to provide a more recognizable domain name for your users.

This article provides generic instructions for using a custom domain name with Azure App Service that use Traffic Manager for load balancing.

If you do not already have a Traffic Manager profile, use the information in [Create a Traffic Manager profile using Quick Create](../traffic-manager/traffic-manager-manage-profiles.md) to create one. Note the **.trafficmanager.net** domain name associated with your Traffic Manager profile, as this will be used later by later steps in this document.


This article is for App Service (Web Apps, API Apps, Mobile Apps, Logic Apps); for Cloud Services, see <a href="/develop/net/common-tasks/custom-dns/">Configuring a Custom Domain Name in Azure</a>.

> [AZURE.NOTE]  **For instructions on using Traffic Manager to load balance traffic to a web app**, use the selector at the top of this article to select the Traffic Manager specific steps.
>
> **Custom domain names cannot be used with Free web app**. You must configure your web app for **Shared**, **Basic**, or **Standard** mode, which may change how much you are billed for your subscription. See <a href=/pricing/details/web-sites/">Web Apps Pricing Details</a> for more information.

<a name="understanding-records"></a>

## Understanding DNS records
The Domain Name System (DNS) is used to locate things on the internet. For example, when you enter an address in your browser, or click a link on a web page, it uses DNS to translate the domain into an IP address. The IP address is sort of like a street address, but it's not very human friendly. For example, it is much easier to remember a DNS name like **contoso.com** than it is to remember an IP address such as 192.168.1.88 or 2001:0:4137:1f67:24a2:3888:9cce:fea3.

The DNS system is based on *records*. Records associate a specific *name*, such as **contoso.com**, with either an IP address or another DNS name. When an application, such as a web browser, looks up a name in DNS, it finds the record, and uses whatever it points to as the address. If the value it points to is an IP address, the browser will use that value. If it points to another DNS name, then the application has to do resolution again. Ultimately, all name resolution will end in an IP address.

When you create an Azure Website, a DNS name is automatically assigned to the site. This name takes the form of **&lt;yoursitename&gt;.azurewebsites.net**. When you add your website as an Azure Traffic Manager endpoint, your website is then accessible through the **&lt;yourtrafficmanagerprofile&gt;.trafficmanager.net** domain.

> [AZURE.NOTE] When your website is configured as a Traffic Manager endpoint, you will use the **.trafficmanager.net** address when creating DNS records.

> You can only use CNAME records with Traffic Manager

There are also multiple types of records, each with their own functions and limitations, but for websites configured to as Traffic Manager endpoints, we only care about one; *CNAME* records.

###CNAME or Alias record

A CNAME record maps a *specific* DNS name, such as **mail.contoso.com** or **www.contoso.com**, to another (canonical) domain name. In the case of Azure Websites using Traffic Manager, the canonical domain name is the **&lt;myapp>.trafficmanager.net** domain name of your Traffic Manager profile. Once created, the CNAME creates an alias for the **&lt;myapp>.trafficmanager.net** domain name. The CNAME entry will resolve to the IP address of your **&lt;myapp>.trafficmanager.net** domain name automatically, so if the IP address of the website changes, you do not have to take any action.

Once traffic arrives at Traffic Manager, it then routes the traffic to your website, using the load balancing method it is configured for. This is completely transparent to visitors to your website. They will only see the custom domain name in their browser.

> [AZURE.NOTE] Some domain registrars only allow you to map subdomains when using a CNAME record, such as **www.contoso.com**, and not root names, such as **contoso.com**. For more information on CNAME records, see the documentation provided by your registrar, <a href="http://en.wikipedia.org/wiki/CNAME_record">the Wikipedia entry on CNAME record</a>, or the <a href="http://tools.ietf.org/html/rfc1035">IETF Domain Names - Implementation and Specification</a> document.


<a name="bkmk_configsharedmode"></a>

## Configure your web apps for standard mode
Setting a custom domain name on a web app in Azure App Service that is load balanced by Traffic Manager is only available for **Standard** mode websites. Before switching a web app from the Free App Service plan mode to the Shared, Basic or Standard mode, you must first remove spending caps in place for your App Service subscription. 

For more information on the App Service plan modes, including how to change the mode of your site, see [How to scale web sites](../article/app-service-web/web-sites-scale.md).

<a name="bkmk_configurecname"></a>

## Add a DNS record for your custom domain
> [!NOTE]
> If you have purchased domain through Azure App Service Web Apps then skip following steps and refer to the final step of [Buy Domain for Web Apps](custom-dns-web-site-buydomains-web-app.md) article. 
> 
> 
To associate your custom domain with a web app in Azure App Service, you must add a new entry in the DNS table for your custom domain by using tools provided by the domain registrar that you purchased your domain name from. Use the following steps to locate and use the DNS tools.

1. Log on to your account at your domain registrar, and look for a page for managing DNS records. Look for links or areas of the site labeled as **Domain Name**, **DNS**, or **Name Server Management**. Often a link to this page can be found be viewing your account information, and then looking for a link such as **My domains**.

2. Once you have found the management page for your domain name, look for a link that allows you to edit the DNS records. This might be listed as a **Zone file**, **DNS Records**, or as an **Advanced** configuration link.

   * The page will most likely have a few records already created, such as an entry associating '**@**' or '\*' with a 'domain parking' page. It may also contain records for common sub-domains such as **www**.
* The page will mention **CNAME records**, or provide a drop-down to select a record type. It may also mention other records such as **A records** and **MX records**. In some cases, CNAME records will be called by other names such as an **Alias Record**.
* The page will also have fields that allow you to **map** from a **Host name** or **Domain name** to another domain name.

3. While the specifics of each registrar vary, in general you map *from* your custom domain name (such as **contoso.com**,) *to* the Traffic Manager domain name (**contoso.trafficmanager.net**) that is used for your web app.

4. Once you have finished adding or modifying DNS records at your registrar, save the changes.


<a name="enabledomain"></a>

## Enable Traffic Manager
After the records for your domain name have propagated, you should be able to use your browser to verify that your custom domain name can be used to access your web app in Azure App Service.

> [AZURE.NOTE] It can take some time for your CNAME to propagate through the DNS system. You can use a service such as <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> to verify that the CNAME is available.

If you have not already added your web app as a Traffic Manager endpoint, you must do this before name resolution will work, as the custom domain name routes to Traffic Manager. Traffic Manager then routes to your web app. Use the information in [Add or Delete Endpoints](../traffic-manager/traffic-manager-endpoints.md) to add your web app as an endpoint in your Traffic Manager profile.

> [AZURE.NOTE] If your web app is not listed when adding an endpoint, verify that it is configured for **Standard** App Service plan mode. You must use **Standard** mode for your web app in order to work with Traffic Manager.

1. In your browser, open the [Azure Portal](https://portal.azure.com).

2. In the **Web Apps** tab, click the name of your web app, select **Settings**, and then select **Custom domains and SSL**

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

3. In the **Custom domains and SSL** blade, click **Bring external domains**.

	![](./media/custom-dns-web-site/dncmntask-cname-7.png)

4. Use the **DOMAIN NAMES** text boxes to enter the Traffic Manager domain name (contoso.trafficmanager.net) to associate with this web app.

	![](./media/custom-dns-web-site/dncmntask-cname-8.png)

5. Click **Save** to save the domain name configuration.

	Once configuration has completed, the custom domain name will be listed in the **domain names** section of your web app.

At this point, you should be able to enter the Traffic Manager domain name (contoso.trafficmanager.net) name in your browser and see that it successfully takes you to your web app.


## What's changed
* For a guide to the change from Websites to App Service see: [Azure App Service and Its Impact on Existing Azure Services](http://go.microsoft.com/fwlink/?LinkId=529714)


>[AZURE.NOTE] If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](http://go.microsoft.com/fwlink/?LinkId=523751), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.


