<properties 
    pageTitle="How to Create a Media Processor | Microsoft Azure" 
    description="Learn how to create a media processor component to encode, convert format, encrypt, or decrypt media content for Azure Media Services. Code samples are written in C# and use the Media Services SDK for .NET." 
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


# How to: Get a Media Processor Instance
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [.NET](media-services-get-media-processor.md)
> * [REST](media-services-rest-get-media-processor.md)
> 
> 
## Overview
In Media Services a media processor is a component that handles a specific processing task, such as encoding, format conversion, encrypting, or decrypting media content. You typically create a media processor when you are creating a task to encode, encrypt, or convert the format of media content.

The following table provides the name and description of each available media processor.

| Media Processor Name | Description | More Information |
| --- | --- | --- |
| Azure Media Encoder |Lets you run encoding tasks using the Azure Media Encoder. |[Azure Media Encoder](media-services-encode-asset.md#azure_media_encoder) |
| Media Encoder Standard |Lets you run encoding tasks using the Media Encoder Standard. |[Azure Media Encoder](media-services-encode-asset.md#media_encoder_standard) |
| Media Encoder Premium Workflow |Lets you run encoding tasks using Media Encoder Premium Workflow. |[Media Encoder Premium Workflow](media-services-encode-asset.md#media_encoder_premium_wokrflow) |
| Azure Media Indexer |Enables you to make media files and content searchable, as well as generate closed captioning tracks and keywords. |[Indexing Media Files with Azure Media Indexer](media-services-index-content.md). |
| Azure Media Hyperlapse (preview) |Enables you to smooth out the "bumps" in your video with video stabilization. Also allows you to speed up your content into a consumable clip. |[Azure Media Hyperlapse](https://azure.microsoft.com/blog/?p=286281preview=1_ppp=61e1a0b3db)</a> |
| Storage Decryption |Lets you decrypt media assets that were encrypted using storage encryption. |N/A |
| Windows  Azure Media Packager |Lets you convert media assets from .mp4 to smooth streaming format. Also, lets you convert media assets from smooth streaming to the Apple HTTP Live Streaming (HLS) format. |[Task Preset Strings for the Azure Media Packager](http://msdn.microsoft.com/library/hh973635.aspx) |
| Windows  Azure Media Encryptor |Lets you encrypt media assets using PlayReady Protection. |[Task Preset Strings for the Azure Media Packager](http://msdn.microsoft.com/library/hh973610.aspx) |

## Get MediaProcessor
The following method shows how to get a media processor instance. The code example assumes the use of a module-level variable named **_context** to reference the server context as described in the section [How to: Connect to Media Services Programmatically](../media-services-set-up-computer/.md).

    private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
    {
         var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
            ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

        if (processor == null)
            throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

        return processor;
    }


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

## Next Steps
Now that you know how to get a media processor instance, go to the [How to Encode an Asset](media-services-encode-asset.md) topic which will show you how to use the Azure Media Encoder to encode an asset.

[How to Encode an Asset]: media-services-encode-asset.md
[Task Preset Strings for the Azure Media Encoder]: http://msdn.microsoft.com/library/jj129582.aspx
[How to: Connect to Media Services Programmatically]: ../media-services-set-up-computer/ 
