<properties 
    pageTitle="Download the Azure SDK for Java (Linux)" 
    description="Download the Azure SDK for Java. Code is provided if you are set up to use Maven for build. Install steps provided for the Azure Tookit for Eclipse." 
    services="" 
    documentationCenter="java" 
    authors="rmcmurray" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="multiple" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="Java" 
    ms.topic="article" 
    ms.date="01/09/2016" 
    ms.author="robmcm"/>

# Download the Azure SDK for Java
## Azure Client Libraries for Java - Manual Download

The Azure Libraries for Java are distributed under the [Apache License, Version 2.0][license]. Click [here][zip-download] for a ZIP file of the libraries and all dependencies.  This is made available by Microsoft Open Technologies, Inc.  See the license.txt and ThirdPartyNotices.txt file file inside the ZIP for license and other information.

## Azure Libraries for Java - Maven

If your project is already set up to use Maven for build, add the following dependency to your pom.xml file. Note: For information about creating Maven projects in Eclipse which use the Azure libraries for Java, see [Getting Started with Azure Management Libraries for Java][maven-getting-started].

	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-compute</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-network</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-sql</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-storage</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-websites</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-media</artifactId>
	    <version>0.6.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-servicebus</artifactId>
	    <version>0.7.0</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-serviceruntime</artifactId>
	    <version>0.6.0</version>
	</dependency>


Within the `<version>` element, replace the version numbers in this example with valid version numbers, which can be obtained from the [Azure Libraries Repository on Maven](http://go.microsoft.com/fwlink/?LinkID=286274).

[license]: http://www.apache.org/licenses/LICENSE-2.0.html
[zip-download]: http://go.microsoft.com/fwlink/?LinkId=690320
[maven-getting-started]: http://go.microsoft.com/fwlink/?LinkID=622998

##Azure Toolkit for Eclipse

Prerequisites:

1. Windows 7, Windows 8, Windows Server 2008, or Windows Server 2012.
2. Macintosh or Linux operting systems listed in the [What's New in the Azure Toolkit for Eclipse](http://go.microsoft.com/fwlink/?LinkId=690333) article.
2. Eclipse Indigo or later.

Installation steps:

1. In Eclipse, from the **Help** menu, select **Install New Software**.
2. Enter the site location <http://dl.msopentech.com/eclipse> and press **Enter**.
3. Select the items to be installed and click **Finish**.

This plugin uses the latest version of the Azure SDK. This can be downloaded using the Web Platform Installer (WebPI) at <http://go.microsoft.com/fwlink/?LinkID=252838>. However, if you don't have it installed, when you create your first Azure deployment project, the Azure Toolkit for Eclipse will automatically install the appropriate version of the Azure SDK.



## Next steps
For more information, see the [Java Developer Center](/develop/java/).

