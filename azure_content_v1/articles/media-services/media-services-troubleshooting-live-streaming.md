<properties 
    pageTitle="Troubleshooting live streaming guide" 
    description="This topic gives suggestions on how to troubleshoot live streaming problems." 
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

# Troubleshooting live streaming guide
This topic gives suggestions on how to troubleshoot some live streaming problems.

## Issues related to on-premises encoders
This section gives suggestions on how to troubleshoot problems related to on-premises encoders that are configured to send a single bitrate stream to AMS channels that are enabled for live encoding.

### Problem: There is no option for outputting a progressive stream
* **Potential issue**: The encoder being used doesn't automatically deinterlace. 

    **Troubleshooting steps**: Look for a de-interlacing option within the encoder interface. Once de-interlacing is enabled, check again for progressive output settings. 


### Problem: Tried several encoder output settings and still unable to connect.
* **Potential issue**: Azure encoding channel was not properly reset. 

    **Troubleshooting steps**: Make sure the encoder is no longer pushing to AMS, stop and reset the channel. Once running again, try connecting your encoder with the new settings. If this still does not correct the issue, try creating a new channel entirely, sometimes channels can become corrupt after several failed attempts.  

* **Potential issue**: The GOP size or key frame settings are not optimal. 

    **Troubleshooting steps**: Recommended GOP size or keyframe interval is 2 seconds. Some encoders calculate this setting in number of frames, while others use seconds. For example: When outputting 30fps, the GOP size would be 60 frames, which is equivalent to 2 seconds.  

* **Potential issue**: Closed ports are blocking the stream. 

    **Troubleshooting steps**: When streaming via RTMP, check firewall and/or proxy settings to confirm that outbound ports 1935 and 1936 are open. When using RTP streaming, confirm that outbound port 2010 is open. 


### Problem: When configuring the encoder to stream with the RTP protocol, there is no place to enter a host name.
* **Potential issue**: Many RTP encoders do not allow for host names, and an IP address will need to be acquired.  

    **Troubleshooting steps**: To find the IP address, open a command prompt on any computer. To do this in Windows, open the Run launcher (WIN + R) and type “cmd” to open.  

    Once the command prompt is open, type "Ping [AMS Host Name]AMS Host Name]". 

    The host name can be derived by omitting the port number from the Azure Ingest URL, as highlighted in the following example: 

    rtp://test2-amstest009.rtp.channel.mediaservices.windows.net:2010/ 

    ![fmle](./media/media-services-fmle-live-encoder/media-services-fmle10.png)


### Problem: Unable to playback the published stream.
* **Potential issue**: There is no Streaming Endpoint running, or there is no streaming units (scale units) allocated. 

    **Troubleshooting steps**: Navigate to the "Streaming Endpoint" tab in the AMSE tool, and confirm there is a Streaming Endpoint running with one streaming unit. 


> [!NOTE]
> If after following the troubleshooting steps you still cannot successfully stream, submit a support ticket using the Azure Classic Portal.
> 
> 
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
