<properties
    pageTitle="Dynamic Packaging Overview"
    description="The topic gives and overview of dynamic packaging."
    authors="Juliako"
    manager="dwrede"
    editor=""
    services="media-services"
    documentationCenter=""/>

<tags
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/28/2016"
    ms.author="juliako"/>


# Dynamic Packaging
## Overview
Microsoft Azure Media Services can be used to deliver many media source file formats, media streaming formats, and content protection formats to a variety of client technologies (for example, iOS, XBOX, Silverlight, Windows 8). These clients understand different protocols, for example iOS requires an HTTP Live Streaming (HLS) V4 format and Silverlight and Xbox require Smooth Streaming. If you have a set of adaptive bitrate (multi-bitrate) MP4 (ISO Base Media 14496-12) files or a set of adaptive bitrate Smooth Streaming files that you want to serve to clients that understand MPEG DASH, HLS or Smooth Streaming, you should take advantage of Media Services dynamic packaging.

With dynamic packaging all you need is to create an asset that contains a set of adaptive bitrate MP4 files or adaptive bitrate Smooth Streaming files. Then, based on the specified format in the manifest or fragment request, the On-Demand Streaming server will ensure that you receive the stream in the protocol you have chosen. As a result, you only need to store and pay for the files in single storage format and Media Services service will build and serve the appropriate response based on requests from a client.

The following diagram shows the traditional encoding and static packaging workflow.

![Static Encoding](./media/media-services-dynamic-packaging-overview/media-services-static-packaging.png)

The following diagram shows the dynamic packaging workflow.

![Dynamic Encoding](./media/media-services-dynamic-packaging-overview/media-services-dynamic-packaging.png)

> [!NOTE]
> To take advantage of dynamic packaging, you must first get at least one On-demand streaming unit for the streaming endpoint from which you plan to delivery your content. For more information, see [How to Scale Media Services](media-services-manage-origins.md#scale_streaming_endpoints).
> 
> 
## Common scenario
1. Upload an input file (called a mezzanine file). For example, H.264, MP4, or WMV (for the list of supported formats see [Formats Supported by the Media Encoder Standard](media-services-media-encoder-standard-formats.md).

2. Encode your mezzanine file to H.264 MP4 adaptive bitrate sets.

3. Publish the asset that contains the adaptive bitrate MP4 set by creating the On-Demand Locator.

4. Build the streaming URLs to access and stream your content.


## Preparing assets for dynamic streaming
To prepare your asset for dynamic streaming you have two options:

1. [Upload a master file](media-services-dotnet-upload-files.md).
2. [Use the Media Encoder Standard encoder to produce H.264 MP4 adaptive bitrate sets](media-services-dotnet-encode-with-media-encoder-standard.md).
3. [Stream your content](media-services-deliver-content-overview.md).

-OR-

1. Upload pre-enocded MP4 files. 

   > [!NOTE]
> This opiton is not recommended.
> 
2. [Validate your pre-encoded files](media-services-static-packaging.md#validating-adaptive-bitrate-mp4s-encoded-with-external-encoders).

3. [Stream your content](media-services-deliver-content-overview.md).

## <a id="unsupported_formats"></a>Formats that are not supported by dynamic packaging
The following source file formats are not supported by dynamic packaging.

* Dolby digital mp4 files.
* Dolby digital smooth files.

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

