<properties 
    pageTitle="Create NoSQL database account - Free Trial | Microsoft Azure" 
    description="Learn how to create database accounts using the online database creator for Azure DocumentDB, a managed NoSQL document database for JSON. Get a free trial today."
    keywords="Free trial, online database creator, create a database, create database, documentdb, azure, Microsoft azure"
    services="documentdb" 
    documentationCenter="" 
    authors="mimig1" 
    manager="jhubbard" 
    editor="monicar"/>

<tags 
    ms.service="documentdb" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="get-started-article" 
    ms.date="12/17/2015" 
    ms.author="mimig"/>

# Create a DocumentDB database account using the Azure portal
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Azure Portal](documentdb-create-account.md)
> * [Azure CLI and ARM](documentdb-automation-resource-manager-cli.md)
> 
> 
To use Microsoft Azure DocumentDB, you must create a DocumentDB database account using either the Azure portal, Azure Resource Manager templates, or Azure command-line interface (CLI). This article shows how to create a database account using the Azure portal. To create an account using Azure Resource Manager or Azure CLI, see [Automate DocumentDB database account creation](documentdb-automation-resource-manager-cli.md). 

Are you new to DocumentDB? Watch [this](https://azure.microsoft.com/documentation/videos/create-documentdb-on-azure/) four minute video by Scott Hanselman to see how to complete the most common tasks in the online portal.

1.	Sign in to the online [Microsoft Azure portal](https://portal.azure.com/).
2.	In the Jumpbar, click **New**, then click **Data + Storage**, and then click **Azure DocumentDB**. 
  
	![Screen shot of the Azure portal  to create a database, highlighting the New button, Data + storage in the Create blade, and Azure DocumentDB in the Data + Storage blade](media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png)  

3. In the **New DocumentDB account** blade, specify the desired configuration for the DocumentDB account. 
 
	![Screen shot of the New DocumentDB blade](media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-2.png) 


	- In the **ID** box, enter a name to identify the DocumentDB account.  When the **ID** is validated, a green check mark appears in the **ID** box. The **ID** value becomes the host name within the URI. The **ID** may contain only lowercase letters, numbers, and the '-' character, and must be between 3 and 50 characters. Note that *documents.azure.com* is appended to the endpoint name you choose, the result of which will become your DocumentDB account endpoint.
	

	- The **Account Tier** lens is locked because DocumentDB supports a single standard account tier. For more information, see [DocumentDB pricing](http://go.microsoft.com/fwlink/p/?LinkID=402317&clcid=0x409).
	
	- For **Subscription**, select the Azure subscription that you want to use for the DocumentDB account. If your account has only one subscription, that account is selected by default.

	- In **Resource Group**, select or create a resource group for your DocumentDB account.  By default, a new Resource group will be created.  You may, however, choose to select an existing resource group to which you would like to add your DocumentDB account. For more information, see [Using the Azure portal to manage your Azure resources](resource-group-portal.md).
 
	- Use **Location** to specify the geographic location in which to host your DocumentDB account.   

4.	Once the new DocumentDB account options are configured, click **Create**.  It can take a few minutes to create the DocumentDB account.  To check the status, you can monitor the progress on the Startboard.  
	![Screen shot of the Creating tile on the Startboard - Online database creator](media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-3.png)  
  
	Or, you can monitor your progress from the Notifications hub.  

	![Create databases quickly - Screen shot of the Notifications hub, showing that the DocumentDB account is being created](media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-4.png)  

	![Screen shot of the Notifications hub, showing that the DocumentDB account was created successfully and deployed to a resource group - Online database creator notification](media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-5.png)

5.	After the DocumentDB account is created, it is ready for use with the default settings in the online portal. Note that the default consistency of the DocumentDB account is set to **Session**.  You can adjust the default consistency setting by clicking the **Default Consistency** tile on the **DocumentDB account** blade.

    ![Screen shot of the Resource Group blade - begin application development](media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-6.png)  

[How to: Create a DocumentDB account]: #Howto
[Next steps]: #NextSteps
[documentdb-manage]:../articles/documentdb/documentdb-manage.md


## Next steps
Now that you have a DocumentDB account, the next step is to create a DocumentDB database. You can create a database by using one of the following:

* The C# .NET samples in the [DatabaseManagement](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples/DatabaseManagement) project of the [azure-documentdb-net](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples) repository on GitHub.
* The Azure portal, as described in [Create a DocumentDB database using the Azure portal](documentdb-create-database.md).
* The all-inclusive tutorials: [.NET](documentdb-get-started.md), [.NET MVC](documentdb-dotnet-application.md), [Java](documentdb-java-application.md), [Node.js](documentdb-nodejs-application.md), or [Python](documentdb-python-application.md).
* The [DocumentDB SDKs](documentdb-sdk-dotnet.md). DocumentDB has .NET, Java, Python, Node.js, and JavaScript API SDKs. 

After creating your database, you need to [add one or more collections](documentdb-create-collection.md) to the database, then [add documents](documentdb-view-json-document-explorer.md) to the collections. 

After you have documents in a collection, you can use [DocumentDB SQL](documentdb-sql-query.md) to [execute queries](documentdb-sql-query.md#executing-queries) against your documents by using the [Query Explorer](documentdb-query-collections-query-explorer.md) in the Portal, the [REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx), or one of the [SDKs](documentdb-sdk-dotnet.md).

To learn more about DocumentDB, explore these resources:

* [Learning path for DocumentDB](https://azure.microsoft.com/documentation/learning-paths/documentdb/)
* [DocumentDB resource model and concepts](documentdb-resources.md)

