<properties 
    pageTitle="DocumentDB Java SDK | Microsoft Azure" 
    description="Learn all about the Java SDK including release dates, retirement dates, and changes made between each version of the DocumentDB Java SDK." 
    services="documentdb" 
    documentationCenter="java" 
    authors="ryancrawcour" 
    manager="jhubbard" 
    editor="cgronlun"/>

<tags 
    ms.service="documentdb" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="java" 
    ms.topic="article" 
    ms.date="01/19/2016" 
    ms.author="ryancraw"/>

# DocumentDB SDK
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [.NET SDK](documentdb-sdk-dotnet.md)
> * [Node.js SDK](documentdb-sdk-node.md)
> * [Java SDK](documentdb-sdk-java.md)
> * [Python SDK](documentdb-sdk-python.md)
> 
> 
## DocumentDB Java SDK
<table>

<tr><td>**Download**</td><td>[Maven](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb)</td></tr>

<tr><td>**Contribute**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-java/)</td></tr>

<tr><td>**Documentation**</td><td>[Java SDK Reference Documentation](http://azure.github.io/azure-documentdb-java/)</td></tr>

<tr><td>**Get Started**</td><td>[Get started with the Java SDK](documentdb-java-application.md)</td></tr>

<tr><td>**Current Supported Runtime**</td><td>[JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)</td></tr>
</table></br>

## Release Notes
### <a name="1.5.1"/>[1.5.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.5.1)
* Fixed a bug in HashPartitionResolver to generate hash values in little-endian to be consistent with other SDKs.

### <a name="1.5.0"/>[1.5.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.5.0)
* Add Hash & Range partition resolvers to assist with sharding applications across multiple partitions.

### <a name="1.4.0"/>[1.4.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.4.0)
* Implement Upsert. New upsertXXX methods added to support Upsert feature.
* Implement ID Based Routing. No public API changes, all changes internal.

### <a name="1.3.0"/>1.3.0
* Release skipped to bring version number in alignment with other SDKs

### <a name="1.2.0"/>[1.2.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.2.0)
* Supports GeoSpatial Index
* Validates id property for all resources. Ids for resources cannot contain ?, /, #, \, characters or end with a space.
* Adds new header "index transformation progress" to ResourceResponse.

### <a name="1.1.0"/>[1.1.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.1.0)
* Implements V2 indexing policy

### <a name="1.0.0"/>[1.0.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.0.0)
* GA SDK

## Release & Retirement Dates
Microsoft will provide notification at least **12 months** in advance of retiring an SDK in order to smooth the transition to a newer/supported version.

New features and functionality and optimizations are only added to the current SDK, as such it is  recommend that you always upgrade to the latest SDK version as early as possible. 

Any request to DocumentDB using a retired SDK will be rejected by the service.

> [!WARNING]
> All versions of the Azure DocumentDB SDK for Java prior to version **1.0.0** will be retired on **February 29, 2016**. 
> 
> 
<br/>

| Version | Release Date | Retirement Date  |
| --- | --- | --- |
| [1.5.1](#1.5.1) |December 31, 2015 |---  |
| [1.5.0](#1.5.0) |December 04, 2015 |--- |
| [1.4.0](#1.4.0) |October 05, 2015 |--- |
| [1.3.0](#1.3.0) |October 05, 2015 |--- |
| [1.2.0](#1.2.0) |August 05, 2015 |--- |
| [1.1.0](#1.1.0) |July 09, 2015 |--- |
| [1.0.1](#1.0.1) |May 12, 2015 |--- |
| [1.0.0](#1.0.0) |April 07, 2015 |--- |
| 0.9.5-prelease |Mar 09, 2015 |February 29, 2016 |
| 0.9.4-prelease |February 17, 2015 |February 29, 2016 |
| 0.9.3-prelease |January 13, 2015 |February 29, 2016 |
| 0.9.2-prelease |December 19, 2014 |February 29, 2016 |
| 0.9.1-prelease |December 19, 2014 |February 29, 2016 |
| 0.9.0-prelease |December 10, 2014 |February 29, 2016 |

## FAQ
**1. How will customers be notified of the retiring SDK?**

Microsoft will provide 12 month advance notification to the end of support of the retiring SDK in order to facilitate a smooth transition to a supported SDK. Further, customers will be notified through various communication channels – Azure Management Portal, Developer Center, blog post, and direct communication to assigned service administrators.

**2. Can customers author applications using a "to-be" retired DocumentDB SDK during the 12 month period?** 

Yes, customers will have full access to author, deploy and modify applications using the "to-be" retired DocumentDB SDK during the 12 month grace period. During the 12 month grace period, customers are advised to migrate to a newer supported version of DocumentDB SDK as appropriate.

**3. Can customers author and modify applications using a retired DocumentDB SDK after the 12 month notification period?**

After the 12 month notification period, the SDK will be retired. Any access to DocumentDB by an applications using a retired SDK will not be permitted by the DocumentDB platform. Further, Microsoft will not provide customer support on the retired SDK.

**4. What happens to Customer’s running applications that are using unsupported DocumentDB SDK version?**

Any attempts made to connect to the DocumentDB service with a retired SDK version will be rejected. 

**5. Will new features and functionality be applied to all non-retired SDKs**

New features and functionality will only be added to new versions. If you are using an old, non-retired, version of the SDK your requests to DocumentDB will still function as previous but you will not have access to any new capabilities.  

**6. What should I do if I cannot update my application before a cut-off date**

We recommend that you upgrade to the latest SDK as early as possible. Once an SDK has been tagged for retirement you will have 12 months to update your application. If, for whatever reason, you cannot complete your application update within this timeframe then please contact the [DocumentDB Team](mailto:askdocdb@microsoft.com) and request their assistance before the cutoff date.


## See Also
To learn more about DocumentDB, see [Microsoft Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) service page. 

