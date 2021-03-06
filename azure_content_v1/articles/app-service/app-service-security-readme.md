<properties 
    pageTitle="Azure App Service Security" 
    description="Learn how to secure Web, Mobile, API and Logic apps in Azure App Service." 
    services="app-service" 
    documentationCenter="" 
    authors="naziml" 
    manager="yochayk" 
    editor="wpickett"/>

<tags 
    ms.service="app-service" 
    ms.workload="web" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="12/10/2015" 
    ms.author="naziml"/>

# Azure App Service Security
Azure App Services complies with the key industry standards for security and reliability to provide our customers a secure platform. We have several easy to use features to help secure both inbound and outbound traffic for your app. We also help customers securre their application code by providing externally provided functionality to scan your web application for vulnerabilities.

* [Secure your web app using various means of authentication and authorization](../app-service-web/web-sites-authentication-authorization.md)
  * [Setup Azure Active Directory authentication for your app](https://azure.microsoft.com/blog/azure-websites-authentication-authorization/)
* [Secure traffic to your app by enabling Transport Layer Security (TLS/SSL) - HTTPS](../app-service-web/web-sites-configure-ssl-certificate.md)
  * [Force all incoming traffic over HTTPS connection](http://microsoftazurewebsitescheatsheet.info/#force-https)
  * [Enable Strict Transport Security (HSTS)](http://microsoftazurewebsitescheatsheet.info/#enable-http-strict-transport-security-hsts)
* [Restrict access to your app by client's IP address](http://microsoftazurewebsitescheatsheet.info/#filtering-traffic-by-ip)
* [Restrict access to your app by client's behavior - request frequency and concurrency](http://microsoftazurewebsitescheatsheet.info/#dynamic-ip-restrictions)
* [Scan your web app code for vulnerabilities using Tinfoil Security Scanning](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/)
* [Configure TLS mutual authentication to require client certificates to connect to your web app](../app-service-web/app-service-web-configure-tls-mutual-auth.md)
* [Configure a client certificate for use from your app to securely connect to external resources](https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/)
* [Remove standard server headers to avoid tools from fingerprinting your app](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/)
* [Securely connect your app with resources in a private network using Point-To-Site VPN](../app-service-web/web-sites-integrate-with-vnet.md)
* [Securely connect your app with resources in a private network using Hybrid Connections](../app-service-web/web-sites-hybrid-connection-get-started.md)
* [Achieve security isolation for your apps using App Service Environments (ASE)](../app-service-web/app-service-app-service-environment-intro.md)
  * [Configure a Web Application Firewall (WAF) in front of your ASE ](../app-service-web/app-service-app-service-environment-web-application-firewall.md)
  * [Configure access control for inbound network traffic for your ASE](../app-service-web/app-service-app-service-environment-control-inbound-traffic.md)
  * [Securely connect to backend resources from your ASE](../app-service-web/app-service-app-service-environment-securely-connecting-to-backend-resources.md)


