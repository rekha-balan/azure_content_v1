<properties 
    pageTitle="Delivering Live Streaming with Azure Media Services" 
    description="This topic gives an overview of main components involoved in live streaming." 
    services="media-services" 
    documentationCenter="" 
    authors="Juliako" 
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


# Delivering Live Streaming Events with Azure Media Services
## Overview
When working with Live Streaming the following components are commonly involved: 

* A camera that is used to broadcast an event.
* A live video encoder that converts signals from the camera to streams that are sent to a live streaming service. 

    Optionally, multiple live encoders. For certain critical live events that demand very high availability and quality of experience, it is recommended to employ active-active redundant encoders to achieve seamless failover with no data loss.

* A live streaming service that enables you to do the following:   * ingest live content using various live streaming protocols (for example RTMP or Smooth Streaming), 
* encode your stream into adaptive bitrate stream
* preview your live stream,
* store the ingested content in order to be streamed later (Video-on-Demand)
* deliver the content through common streaming protocols (for example, MPEG DASH, Smooth, HLS, HDS) directly to your customers, or to a Content Delivery Network (CDN) for further distribution. 



**Microsoft Azure Media Services** (AMS) provides the ability to ingest,  encode, preview, store, and deliver your live streaming content.

When delivering your content to customers your goal is to deliver a high quality video to various devices under different network conditions. To take care of quality and network conditions, use live encoders to encode your stream to multi-bitrate (adaptive bitrate) video stream.  To take care of streaming on different devices, use Media Services [dynamic packaging](media-services-dynamic-packaging-overview.md) to dynamically re-package your stream to different protocols. Media Services supports delivery of the following adaptive bitrate streaming technologies: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH, and HDS (for Adobe PrimeTime/Access licensees only).

In Azure Media Services, **Channels**, **Programs**, and **StreamingEndpoints** handle all the live streaming functionalities including ingest, formatting, DVR, security, scalability and redundancy. 

A **Channel** represents a pipeline for processing live streaming content. Currently, a Channel can receive a live input streams in the following way:

* An on-premises live encoder sends a single-bitrate stream to the Channel that is enabled to perform live encoding with Media Services in one of the following formats: RTP (MPEG-TS), RTMP, or Smooth Streaming (Fragmented MP4). The Channel then performs live encoding of the incoming single bitrate stream to a multi-bitrate (adaptive) video stream. When requested, Media Services delivers the stream to customers.

* An on-premises live encoder sends a multi-bitrate **RTMP** or **Smooth Streaming** (Fragmented MP4) to the Channel. You can use the following live encoders that output multi-bitrate Smooth Streaming: Elemental, Envivio, Cisco.  The following live encoders output RTMP: Adobe Flash Live, Telestream Wirecast, and Tricaster transcoders. The ingested streams pass through **Channel**s without any further processing. Your live encoder can also send a single bitrate stream to a channel that is not enabled for live encoding, but that is not recommended. When requested, Media Services delivers the stream to customers.


## Working with Channels that are enabled to perform live encoding with Azure Media Services
The following diagram shows the major parts of the AMS platform that are involved in Live Streaming workflow where a Channel is enabled to perform live encoding with Media Services.  

![Live workflow][live-overview1]

For more information, see [Working with Channels that are Enabled to Perform Live Encoding with Azure Media Services](media-services-manage-live-encoder-enabled-channels.md). 

## Working with Channels that receive multi-bitrate live stream from on-premises Encoders
The following diagram shows the major parts of the AMS platform that are involved in this Live Streaming workflow.

![Live workflow][live-overview2]

For more information, see [Working with Channels that Receive Multi-bitrate Live Stream from On-premises Encoders
](media-services-manage-channels-overview.md). 

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

## Related topics
[Media Services Concepts](media-services-concepts.md)

[Azure Media Services Fragmented MP4 Live Ingest Specification](media-services-fmp4-live-ingest-overview.md)

[live-overview1]: ./media/media-services-live-streaming-workflow/media-services-live-streaming-new.png

[live-overview2]: ./media/media-services-live-streaming-workflow/media-services-live-streaming-current.png