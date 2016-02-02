<properties 
    pageTitle="Configure a Web API project as an API app" 
    description="Learn how to configure a Web API project as an API app, using Visual Studio 2013 " 
    services="app-service\api" 
    documentationCenter=".net" 
    authors="bradygaster" 
    manager="wpickett" 
    editor="jimbe"/>

<tags 
    ms.service="app-service-api" 
    ms.workload="web" 
    ms.tgt_pltfrm="dotnet" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="01/08/2016" 
    ms.author="tdykstra"/>

# Configure a Web API project as an API app
> [AZURE.WARNING] This article refers to the original preview release of App Service API Apps.  For information about the current preview release of API Apps, see [App Service API Apps - What's changed](../articles/app-service-api/app-service-api-whats-changed.md). For tutorials that show how to work with the current preview release of API Apps, see [Get started with API Apps](../articles/app-service-api/app-service-api-dotnet-get-started.md). 


## Overview
This tutorial shows how to take an existing Web API project and configure it for deployment as an [API app](app-service-api-apps-why-best-platform.md) in [Azure App Service](../app-service/app-service-value-prop-what-is.md). Subsequent tutorials in the series show how to [deploy](app-service-dotnet-deploy-api-app.md) and [debug](../app-service-dotnet-remotely-debug-api-app.md) the API app project that you create in this tutorial.

For information about API apps, see [What are API apps?](app-service-api-apps-why-best-platform.md).

## <a name="setupdevenv"></a>Set up the development environment

To start, set up your development environment by installing the [Azure SDK for Visual Studio 2015](http://go.microsoft.com/fwlink/?linkid=518003) or the [Azure SDK for Visual Studio 2013](http://go.microsoft.com/fwlink/?LinkID=324322).

If you don't have Visual Studio installed, use the link for Visual Studio 2015, and Visual Studio will be installed along with the SDK.

>[AZURE.NOTE] Depending on how many of the SDK dependencies you already have on your machine, installing the SDK could take a long time, from several minutes to a half hour or more.


This tutorial requires version 2.6 or later of the Azure SDK for .NET.

## Configure a Web API project
This section shows how to configure an existing Web API project as an API app. You'll begin by using the Web API project template to create a Web API project, and then you'll configure it as an API app.

1. Open Visual Studio 2015 or Visual Studio 2013.

2. Click **File > New Project**. 

3. Select the **ASP.NET Web Application** template.

4. Make sure that the **Add Application Insights to Project** check box is cleared. 

5. Name the project *ContactsList*

    ![](./media/app-service-dotnet-create-api-app-visual-studio/01-filenew-v3.png)

6. Click **OK**.

7. In the **New ASP.NET Project** dialog, select the **Empty** project template.

8. Click the **Web API** check box.

9. Clear the **Host in the cloud** option.

    ![](./media/app-service-dotnet-create-api-app-visual-studio/webapinewproj.png)

10. Click **OK** to generate the project.

     ![](./media/app-service-dotnet-create-api-app-visual-studio/sewebapi.png)

11. In **Solution Explorer**, right-click the project (not the solution), and then select **Add > Azure API App SDK**.

    ![](./media/app-service-dotnet-create-api-app-visual-studio/addapiappsdk.png)

12. In the **Choose API App Metadata source** dialog, click **Automatic Metadata Generation**. 

    ![](./media/app-service-dotnet-create-api-app-visual-studio/chooseswagger.png)

    This choice enables the dynamic Swagger UI, which you'll see later in the tutorial. If you choose to upload a Swagger metadata file, it is saved with the file name *apiDefinition.swagger.json*, as explained in the following section. 

13. Click **OK**. 

    At this point, Visual Studio installs API app NuGet packages and adds API app metadata to the Web API project.  


## Review API app metadata

The metadata that enables a Web API project to be deployed as an API app is contained in an *apiapp.json* file and a *Metadata* folder.

![](./media/app-service-api-review-metadata/metadatainse.png)

The default contents of the *apiapp.json* file resemble the following example:

		{
		    "$schema": "http://json-schema.org/schemas/2014-11-01/apiapp.json#",
		    "id": "ContactsList",
		    "namespace": "microsoft.com",
		    "gateway": "2015-01-14",
		    "version": "1.0.0",
		    "title": "ContactsList",
		    "summary": "",
		    "author": "",
		    "endpoints": {
		        "apiDefinition": "/swagger/docs/v1",
		        "status": null
		    }
		}

Notice the `apiDefinition` endpoint `/swagger/docs/v1`:  by default, API app projects use the [Swashbuckle](https://www.nuget.org/packages/Swashbuckle) NuGet package to provide automatic [Swagger](http://swagger.io/) metadata generation. 

For this tutorial you can accept the defaults. The [API app metadata](#api-app-metadata) section later in the tutorial explains how to customize this metadata.


## Add Web API code

In the following steps you add code for a simple HTTP Get method that returns a hard-coded list of contacts. 

1. In Solution Explorer, right-click the **Models** folder and select **Add > Class**. 

	![](./media/app-service-api-define-api-app/03-add-new-class-v3.png) 

2. Name the new file *Contact.cs*. 

	![](./media/app-service-api-define-api-app/0301-add-new-class-dialog-v3.png) 

3. Click **Add**.

4. Once the *Contact.cs* file has been created, replace the entire contents of the file with the following code. 

		namespace ContactsList.Models
		{
			public class Contact
			{
				public int Id { get; set; }
				public string Name { get; set; }
				public string EmailAddress { get; set; }
			}
		}

5. Right-click the **Controllers** folder, and select **Add > Controller**. 

	![](./media/app-service-api-define-api-app/05-new-controller-v3.png)

6. In the **Add Scaffold** dialog, select the **Web API 2 Controller - Empty** option, and click **Add**. 

	![](./media/app-service-api-define-api-app/06-new-controller-dialog-v3.png)

7. Name the controller **ContactsController**, and click **Add**. 

	![](./media/app-service-api-define-api-app/07-new-controller-name-v2.png)

8. Once the ContactsController.cs file has been created, replace the entire contents of the file with the following code. 

		using ContactsList.Models;
		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Net;
		using System.Net.Http;
		using System.Threading.Tasks;
		using System.Web.Http;
		
		namespace ContactsList.Controllers
		{
		    public class ContactsController : ApiController
		    {
		        [HttpGet]
		        public IEnumerable<Contact> Get()
		        {
		            return new Contact[]{
						new Contact { Id = 1, EmailAddress = "barney@contoso.com", Name = "Barney Poland"},
						new Contact { Id = 2, EmailAddress = "lacy@contoso.com", Name = "Lacy Barrera"},
	                	new Contact { Id = 3, EmailAddress = "lora@microsoft.com", Name = "Lora Riggs"}
		            };
		        }
		    }
		}

## Enable Swagger UI

By default, API App projects are enabled with automatic [Swagger](http://swagger.io/ "Official Swagger information") metadata generation, and when you use the **Add API App SDK** menu entry to convert a Web API project, an API test page is also enabled by default.  

However, the Azure API App new-project template disables the API test page. When you create your API app project by using the API app project template, do the following steps to enable the test page.

**Note:** If you deploy the API app as *public anonymous* and with the Swagger UI enabled, anyone will be able to use the Swagger UI to discover and call your APIs. 

1. Open the *App_Start/SwaggerConfig.cs* file, and search for **EnableSwaggerUI**:

	![](./media/app-service-api-define-api-app/12-enable-swagger-ui-with-box.png)

2. Uncomment the following lines of code:

	        })
	    .EnableSwaggerUi(c =>
	        {

3. When you're done, the file should look like this:

	![](./media/app-service-api-define-api-app/13-enable-swagger-ui-with-box.png)

## Test the Web API

To view the API test page, perform the following steps.

1. Run the app locally (CTRL+F5).

	The browser opens and displays an HTTP 403 error because the base URL is not a valid web page or API method URL for this project.
 
3.  Navigate to the Swagger page by adding `/swagger` to the end of the base URL. 

	![](./media/app-service-api-define-api-app/swaggerhome.png)

2. Click **Contacts > Get > Try it out**, and you see that the API is functioning and returns the expected result. 

	![](./media/app-service-api-define-api-app/swaggertry.png)

3. In Visual Studio click **Debug > Stop Debugging**.


## API app metadata

This section provides additional information about API app metadata that you can customize.

Most of the properties in the *apiapp.json* file, and the files in the *Metadata* folder, affect the way an API app package is presented in the Azure Marketplace. The following sections explain which properties and files affect API apps when you deploy your code to an API app in your Azure subscription. 

### API app ID 

The `id` property determines the name of the API app.  For example:

		"id": "ContactsList",

![](./media/app-service-api-direct-deploy-metadata/apiappname.png)

### Namespace

Set the `namespace` property to the domain of your Azure Active Directory tenant. To find your domain, open your browser to the [Azure classic portal](https://manage.windowsazure.com/), browse **Active Directory**, and select the **Domains** tab. For example:

		"namespace": "contoso.onmicrosoft.com",

### Dynamic Swagger API definition

If the API app can return a dynamic [Swagger](http://swagger.io/) API definition, store the relative URL for a GET request that returns the API definition JSON in the `endpoints.apiDefinition` property. For example:  

		"endpoints": {
		    "apiDefinition": "/swagger/docs/v1"
		}

> **Note:** If you are using Swashbuckle to generate a Swagger API definition, HTTP method overloads in your Web API controllers result in duplicate operation ids. For more information, see [Customize Swashbuckle-generated operation identifiers](../article/app-service-api/app-service-api-dotnet-swashbuckle-customize.md).
  
### Static Swagger API definition

To provide a static [Swagger](http://swagger.io/) 2.0 API definition file, store the file in the *Metadata* folder and name the file *apiDefinition.swagger.json*

![](./media/app-service-api-direct-deploy-metadata/apidefinmetadata.png)

Leave `endpoints.apiDefinition` out of the *apiapp.json* file or set its value to null. If you include both an `endpoints.apiDefinition` URL and an *apiDefinition.swagger.json* file, the URL will take precedence and the file will be ignored.


## Next steps
Your API app is now ready to be deployed, and you can follow the [Deploy an API app](app-service-dotnet-deploy-api-app.md) tutorial to do that.

