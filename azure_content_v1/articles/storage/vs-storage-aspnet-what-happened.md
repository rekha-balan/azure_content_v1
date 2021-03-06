<properties
    pageTitle="What happened to my ASP.NET project? | Microsoft Azure | Visual Studio connected services"
    description="Describes what happens after adding Azure Storage to a ASP.NET project using Visual Studio connected services"
    services="storage"
    documentationCenter=""
    authors="TomArcher"
    manager="douge"
    editor=""/>

<tags
    ms.service="storage"
    ms.workload="web"
    ms.tgt_pltfrm="vs-what-happened"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="tarcher"/>

# What happened to my ASP.NET project (Visual Studio Azure Storage connected service)?
## References added
The Azure Storage NuGet package was added to your Visual Studio project.  
This package adds the following .NET references:

* **Microsoft.Data.Edm**
* **Microsoft.Data.OData**
* **Microsoft.Data.Services.Client**
* **Microsoft.WindowsAzure.Configuration**
* **Microsoft.WindowsAzure.Storage**
* **Newtonsoft.Json**
* **System.Data**
* **System.Spatial**

## Connection string for Azure Storage added
In the web.config file of your project, an element was created with the selected storage account's connection string and key.

For more information, see [ASP.NET](http://www.asp.net).

