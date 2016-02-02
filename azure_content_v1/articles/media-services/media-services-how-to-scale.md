<properties 
    pageTitle="How to Scale a media service" 
    description="Learn how to scale Media Services by specifying the number of On-Demand Streaming Reserved Units and Encoding Reserved Units that you would like your account to be provisioned with." 
    services="media-services" 
    documentationCenter="" 
    authors="juliako" 
    manager="dwrede" 
    editor=""/>

<tags 
    ms.service="media-services" 
    ms.workload="media" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="12/05/2015"   
    ms.author="juliako"/>


# How to Scale a Media Service
## Overview
You can scale **Media Services** by specifying the number of **Streaming Reserved Units** and **Encoding Reserved Units** that you would like your account to be provisioned with. 

You can also scale your Media Services account by adding storage accounts to it. Each storage account is limited to 500 TB. To expand your storage beyond the default limitations, you can choose to attach multiple storage accounts to a single Media Services account.

This topic links to relevant topics.

## <a id="streaming_endpoins"></a>Streaming Reserved Units
For more information, see [Scaling streaming units](media-services-manage-origins.md#scale_streaming_endpoints).

## <a id="encoding_reserved_units"></a>Encoding Reserved Units
For information about scaling encoding units, see the following **Portal** and **.NET** topics.

> [AZURE.SELECTOR] 
- [Portal](../articles/media-services/media-services-portal-encoding-units.md)
- [.NET](../articles/media-services/media-services-dotnet-encoding-units.md)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)


Note that the reserved units are the same for Encoding and Indexing tasks.

## <a id="storage"></a>Scale Storage
For more information, see [Managing Media Services Assets across Multiple Storage Accounts](https://msdn.microsoft.com/library/azure/dn271889.aspx) and [Working with Azure Storage](https://msdn.microsoft.com/library/azure/dn767951.aspx).

## Media Services learning paths
You can view AMS learning paths here:

- [AMS Live Streaming Workflow](https://azure.microsoft.com/documentation/learning-paths/media-services-streaming-live/)
- [AMS on Demand Streaming Workflow](https://azure.microsoft.com/documentation/learning-paths/media-services-streaming-on-demand/)

## Provide feedback
Use the [User Voice](http://go.microsoft.com/fwlink/?linkid=698785&clcid=0x409) forum to provide feedback and make suggestions on how to improve Azure Media Services. You can also go directly to one of the following categories: 

- [Azure Media Player](https://feedback.azure.com/forums/169396-media-services/category/109320-azure-media-player/)
- [Client SDK Libraries](https://feedback.azure.com/forums/169396-media-services/category/144435-client-sdks/)
- [Encoding and Processing](https://feedback.azure.com/forums/169396-media-services/category/144411-encoding-and-processing/)
- [Live Streaming](https://feedback.azure.com/forums/169396-media-services/category/144414-live-streaming/)
- [Azure Portal](https://feedback.azure.com/forums/169396-media-services/category/144432-portal/)
- [REST API and Platform](https://feedback.azure.com/forums/169396-media-services/category/144423-rest-api-and-platform/)
- [VoD Streaming](https://feedback.azure.com/forums/169396-media-services/category/144429-vod-streaming/)

