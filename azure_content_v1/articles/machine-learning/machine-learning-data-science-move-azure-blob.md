<properties 
    pageTitle="Move Data to and from Azure Blob Storage | Microsoft Azure" 
    description="Move Data to and from Azure Blob Storage" 
    services="machine-learning,storage" 
    documentationCenter="" 
    authors="bradsev" 
    manager="paulettm" 
    editor="cgronlun" />

<tags 
    ms.service="machine-learning" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="10/12/2015" 
    ms.author="bradsev;sunliangms;sachouks;mohabib" />

# Move Data to and from Azure Blob Storage
> [AZURE.SELECTOR]
- [Ingest data overview](../articles/machine-learning/machine-learning-data-science-ingest-data.md)
- [Azure Blob Storage](../articles/machine-learning/machine-learning-data-science-move-azure-blob.md)
- [SQL Server on Azure VM](../articles/machine-learning/machine-learning-data-science-move-sql-server-virtual-machine.md)
- [Azure SQL database](../articles/machine-learning/machine-learning-data-science-move-sql-azure.md)
- [HDInsight(Hadoop on Azure)](../articles/machine-learning/machine-learning-data-science-move-hive-tables.md)


## Introduction
Guidance on technologies used to move data to and/or from Azure Blob storage are linked here:

> [AZURE.SELECTOR]
- [Overview](../articles/machine-learning/machine-learning-data-science-move-azure-blob.md)
- [Azure Storage-Explorer](../articles/machine-learning/machine-learning-data-science-move-data-to-azure-blob-using-azure-storage-explorer.md)
- [AzCopy](../articles/machine-learning/machine-learning-data-science-move-data-to-azure-blob-using-azcopy.md)
- [Python](../articles/machine-learning/machine-learning-data-science-move-data-to-azure-blob-using-python.md)
- [SSIS](../articles/machine-learning/machine-learning-data-science-move-data-to-azure-blob-using-ssis.md)

Which method is best for you will depend on your scenario. The [Scenarios for the Advanced Analytics Process and Technology (ADAPT) in Azure Machine Learning](../machine-learning-data-science-plan-sample-scenarios.md) article helps you determine the resources you need for a variety of data science workflows used in the advanced analytics process.

> [!NOTE]
> For a complete introduction to Azure blob storage, please refer to [Azure Blob Basics](../storage-dotnet-how-to-use-blobs.md) and  [Azure Blob Service](https://msdn.microsoft.com/library/azure/dd179376.aspx). 
> 
> [!TIP]
> As an alternative, you can use [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) to create and schedule a pipeline that will download data from Azure blob storage, pass it to a published Azure Machine Learning web service, receive the predictive analytics results, and upload the results to storage. For more information, see [Create predictive pipelines using Azure Data Factory and Azure Machine Learning](../data-factory/data-factory-create-predictive-pipelines.md).
> 
> 
## Prerequisites
This document assumes that you have an Azure subscription, a storage account and the corresponding storage key for that account. Before uploading/downloading data, you must know your Azure storage account name and account key. 

* To set up an Azure subscription, see [Free one-month trial](https://azure.microsoft.com/pricing/free-trial/).
* For instructions on creating a storage account and for getting account and key information, see [About Azure storage accounts](../storage-create-storage-account.md).
