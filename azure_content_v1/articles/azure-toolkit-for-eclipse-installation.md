<properties
    pageTitle="Installing the Azure Toolkit for Eclipse"
    description="Learn hot to install the Azure Toolkit for Eclipse."
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.workload="na"
    ms.tgt_pltfrm="multiple"
    ms.devlang="Java"
    ms.topic="article"
    ms.date="01/09/2016" 
    ms.author="robmcm"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690946.aspx -->

# Installing the Azure Toolkit for Eclipse
The Azure Toolkit for Eclipse provides templates and functionality that allow you to easily create, develop, test, and deploy Azure applications using the Eclipse development environment. It is an Open Source project, whose source code is available under the Apache License 2.0 from the project's site on GitHub at the following URL:

[https://github.com/MSOpenTech/WindowsAzureToolkitForEclipseWithJava](https://github.com/MSOpenTech/WindowsAzureToolkitForEclipseWithJava).

The following steps show you how to install the Azure Toolkit for Eclipse.

### Prerequisites ###

The Azure Toolkit for Eclipse requires the following software components:

* Eclipse IDE for Java EE Developers, Indigo or later. This can be downloaded from <http://www.eclipse.org/downloads/>.
* A Java Developer Kit (JDK), v1.7 or later. 
* Operating Systems - the Azure Toolkit for Eclipse has been tested on the following operating systems:
    * Windows 10, Windows 8 and 8.1, Windows 7
    * Windows Server 2012 and 2012 R2, Windows Server 2008 and 2008 R2
    * [Mac OS X](http://www.apple.com/osx) Versions "Yosemite" and "El Captain"
    * [Ubuntu Linux](http://www.ubuntu.com) Versions 12, 14, and 15

>[AZURE.NOTE] More information can be found on the [Azure Toolkit for Eclipse](http://marketplace.eclipse.org/content/azure-toolkit-eclipse) page at the Eclipse Marketplace.

> [AZURE.IMPORTANT] If you are using the Azure Toolkit for Eclipse on Windows, the toolkit requires installing the Azure SDK 2.7 or later. You have two options for installing the Azure SDK:
> 
> * You can download and install the Azure SDK by using the [Web Platform Installer (WebPI)][].
> * If you do not have the Azure SDK installed when you create your first Azure deployment project, you will be prompted to automatically download install the requisite version of the Azure SDK.
> 
> Note that the Azure SDK is only required on Windows.


## To install the Azure Toolkit for Eclipse
1. Start Eclipse.
2. Within Eclipse, at the menu click <strong>Help</strong>, then click <strong>Install New Software</strong>, as shown in the following diagram.
 ![][ic590123]
3. In the <strong>Available Software</strong> dialog, within the <strong>Work with</strong> text box, type <strong>http://dl.msopentech.com/eclipse</strong> followed by the <strong>Enter</strong> key.
4. In the <strong>Name</strong> pane, check <strong>Azure Toolkit for Eclipse</strong>, and uncheck <strong>Contact all update sites during install to find required software</strong>. Your screen should appear similar to the following:
 ![][ic719482]
5. If you expand the <strong>Azure Toolkit for Eclipse</strong>, you will see the following items:   * **Azure Access Control Services Filter**: This component provides support for authenticating application users with Azure ACS.
* **Azure Common Plugin**: This component contains the shared functionality relied upon by the other components.
* **Azure Toolkit for Eclipse**: This component contains the project configuration logic, the publish-to-cloud wizard, and user interface.
* **Microsoft JDBC Driver 4.0 for SQL Server**: This component simplifies application development using SQL Database.
* **Package for Apache Qpid Client Libraries for JMS**: This component provides the JMS client library from the Apache Qpid project to enable your application to use Advanced Messaging Queuing Protocol (AMQP)-based messaging in Azure
* **Package for Azure Libraries for Java**: This component allows you to build Azure applications in Java that allow you to take advantage of Azure scalable cloud computing resources.
* **Application Insights Plugin for Java**: This component allows you to use Azure's telemetry logging and analysis services for your applications and server instances.


6. Click **Next**. (If you experience unusual delays when installing the toolkit, ensure that **Contact all update sites during install to find required software** is unchecked.)
7. In the **Install Details** dialog, click **Next**.
8. In the **Review Licenses** dialog, review the terms of the license agreements. If you accept the terms of the license agreements, click **I accept the terms of the license agreements** and then click **Finish**. (The remaining steps assume you do accept the terms of the license agreements. If you do not accept the terms of the license agreements, exit the installation process.)
9. If prompted to restart Eclipse to complete installation, click **Restart Now**.

## See Also
[Azure Toolkit for Eclipse](http://go.microsoft.com/fwlink/?LinkID=699529)

[Creating a Hello World Application for Azure in Eclipse](http://go.microsoft.com/fwlink/?LinkID=699533)

[What's New in the Azure Toolkit for Eclipse](http://go.microsoft.com/fwlink/?LinkID=699552)

For more information about using Azure with Java, see the [Azure Java Developer Center](http://go.microsoft.com/fwlink/?LinkID=699547).

<!-- URL List -->

[Azure Toolkit for Eclipse]: http://go.microsoft.com/fwlink/?LinkID=699529
[Azure Java Developer Center]: http://go.microsoft.com/fwlink/?LinkID=699547
[Creating a Hello World Application for Azure in Eclipse]: http://go.microsoft.com/fwlink/?LinkID=699533
[Installing the Azure Toolkit for Eclipse]: http://go.microsoft.com/fwlink/?LinkId=699546
[Web Platform Installer (WebPI)]: http://go.microsoft.com/fwlink/?LinkID=252838 
[What's New in the Azure Toolkit for Eclipse]: http://go.microsoft.com/fwlink/?LinkID=699552

<!-- IMG List -->

[ic590123]: ./media/azure-toolkit-for-eclipse-installation/ic590123.png
[ic719482]: ./media/azure-toolkit-for-eclipse-installation/ic719482.png
