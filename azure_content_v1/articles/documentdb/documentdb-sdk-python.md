<properties 
    pageTitle="DocumentDB Python SDK | Microsoft Azure" 
    description="Learn all about the Python SDK including release dates, retirement dates, and changes made between each version of the DocumentDB Python SDK." 
    services="documentdb" 
    documentationCenter="python" 
    authors="ryancrawcour" 
    manager="jhubbard" 
    editor="cgronlun"/>

<tags 
    ms.service="documentdb" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="python" 
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
## DocumentDB Python SDK
<table>

<tr><td>**Download**</td><td>[PyPI](https://pypi.python.org/pypi/pydocumentdb)</td></tr>

<tr><td>**Contribute**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-python)</td></tr>

<tr><td>**Documentation**</td><td>[Python SDK Reference Documentation](http://azure.github.io/azure-documentdb-python/)</td></tr>

<tr><td>**Get Started**</td><td>[Get started with the Python SDK](documentdb-python-application.md)</td></tr>

<tr><td>**Current Supported Platform**</td><td>[Python 2.7](https://www.python.org/download/releases/2.7/)</td></tr>
</table></br>

## Release notes
### <a name="1.5.0"/>[1.5.0](https://pypi.python.org/pypi/pydocumentdb/1.5.0)
* Add Hash & Range partition resolvers to assist with sharding applications across multiple partitions.

### <a name="1.4.2"/>[1.4.2](https://pypi.python.org/pypi/pydocumentdb/1.4.2)
* Implement Upsert. New UpsertXXX methods added to support Upsert feature.
* Implement ID Based Routing. No public API changes, all changes internal.

### <a name="1.2.0"/>[1.2.0](https://pypi.python.org/pypi/pydocumentdb/1.2.0)
* Supports GeoSpatial index.
* Validates id property for all resources. Ids for resources cannot contain ?, /, #, \, characters or end with a space.
* Adds new header "index transformation progress" to ResourceResponse.

### <a name="1.1.0"/>[1.1.0](https://pypi.python.org/pypi/pydocumentdb/1.1.0)
* Implements V2 indexing policy

### <a name="1.0.1"/>[1.0.1](https://pypi.python.org/pypi/pydocumentdb/1.0.1)
* Supports proxy connection

### <a name="1.0.0"/>[1.0.0](https://pypi.python.org/pypi/pydocumentdb/1.0.0)
* GA SDK

## Release & retirement dates
Microsoft will provide notification at least **12 months** in advance of retiring an SDK in order to smooth the transition to a newer/supported version.

New features and functionality and optimizations are only added to the current SDK, as such it is  recommend that you always upgrade to the latest SDK version as early as possible. 

Any request to DocumentDB using a retired SDK will be rejected by the service.

> [!WARNING]
> All versions of the Azure DocumentDB SDK for Python prior to version **1.0.0** will be retired on **February 29, 2016**. 
> 
> 
<br/>

| Version | Release Date | Retirement Date  |
| --- | --- | --- |
| [1.5.0](#1.5.0) |January 03, 2016 |--- |
| [1.4.2](#1.4.2) |October 06, 2015 |--- |
| [1.4.1](#1.4.1) |October 06, 2015 |--- |
| [1.2.0](#1.2.0) |August 06, 2015 |--- |
| [1.1.0](#1.1.0) |July 09, 2015 |--- |
| [1.0.1](#1.0.1) |May 25, 2015 |--- |
| [1.0.0](#1.0.0) |April 07, 2015 |--- |
| 0.9.4-prelease |January 14, 2015 |February 29, 2016 |
| 0.9.3-prelease |December 09, 2014 |February 29, 2016 |
| 0.9.2-prelease |November 25, 2014 |February 29, 2016 |
| 0.9.1-prelease |September 23, 2014 |February 29, 2016 |
| 0.9.0-prelease |August 21, 2014 |February 29, 2016 |

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


## See also
To learn more about DocumentDB, see [Microsoft Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) service page. 

