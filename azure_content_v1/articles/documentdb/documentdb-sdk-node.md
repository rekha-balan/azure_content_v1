<properties 
    pageTitle="DocumentDB Node.js SDK | Microsoft Azure" 
    description="Learn all about the Node.js SDK including release dates, retirement dates, and changes made between each version of the DocumentDB Node.js SDK." 
    services="documentdb" 
    documentationCenter="nodejs" 
    authors="ryancrawcour" 
    manager="jhubbard" 
    editor="cgronlun"/>

<tags 
    ms.service="documentdb" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="nodejs" 
    ms.topic="article" 
    ms.date="01/25/2016" 
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
## DocumentDB Node.js SDK
<table>

<tr><td>**Download**</td><td>[NPM](https://www.npmjs.com/package/documentdb)</td></tr>

<tr><td>**Contribute**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-node/tree/master/source)</td></tr>

<tr><td>**Documentation**</td><td>[Node.js SDK Reference Documentation](http://azure.github.io/azure-documentdb-node/)</td></tr>

<tr><td>**Samples**</td><td>[Node.js Code Samples](https://github.com/Azure/azure-documentdb-node/tree/master/samples)</td></tr>

<tr><td>**Get Started**</td><td>[Get started with the Node.js SDK](documentdb-nodejs-get-started.md)</td></tr>

<tr><td>**Current Supported Platform**</td><td>[Node.js v0.10](https://nodejs.org/en/blog/release/v0.10.0/)<br/>[Node.js v0.12](https://nodejs.org/en/blog/release/v0.12.0/)<br/>[Node.js v4.2.0](https://nodejs.org/en/blog/release/v4.2.0/)</td></tr>
</table></br>

## Release notes
### <a name="1.5.2"/>1.5.2</a>
* Fixes issue [#95](https://github.com/Azure/azure-documentdb-node/issues/95) - EventEmitter listener leak warning

### <a name="1.5.1"/>1.5.1</a>
* Fixes issue [#92](https://github.com/Azure/azure-documentdb-node/issues/90) - rename folder Hash to hash for case sensitive systems

### <a name="1.5.0"/>1.5.0</a>
* Implement sharding support by adding hash & range partition resolvers

### <a name="1.4.0"/>1.4.0</a>
* Implement Upsert. New upsertXXX methods on documentClient. 

### <a name="1.3.0"/>1.3.0</a>
* Skipped to bring version numbers in alignment with other SDKs

### <a name="1.2.2"/>1.2.2</a>
* Split Q promises wrapper to new repository
* Update to package file for npm registry

### <a name="1.2.1"/>1.2.1</a>
* Implements ID Based Routing
* Fixes Issue [#49](https://github.com/Azure/azure-documentdb-node/issues/49) - current property conflicts with method current()

### <a name="1.2.0"/>1.2.0</a>
* Added support for GeoSpatial index.
* Validates id property for all resources. Ids for resources cannot contain ?, /, #, &#47;&#47;, characters or end with a space. 
* Adds new header "index transformation progress" to ResourceResponse.

### <a name="1.1.0"/>1.1.0</a>
* Implements V2 indexing policy

### <a name="1.0.3"/>1.0.3</a>
* Issue [#40](https://github.com/Azure/azure-documentdb-node/issues/40) - Implemented eslint and grunt configurations in the core and promise SDK

### <a name="1.0.2"/>1.0.2</a>
* Issue [#45](https://github.com/Azure/azure-documentdb-node/issues/45) - Promises wrapper does not include header with error.

### <a name="1.0.1"/>1.0.1</a>
* Implemented ability to query for conflicts by adding readConflicts, readConflictAsync, and queryConflicts
* Updated API documentation
* Issue [#41](https://github.com/Azure/azure-documentdb-node/issues/41) - client.createDocumentAsync error  

### <a name="1.0.0"/>1.0.0</a>
* GA SDK

## Release & Retirement Dates
Microsoft will provide notification at least **12 months** in advance of retiring an SDK in order to smooth the transition to a newer/supported version.

New features and functionality and optimizations are only added to the current SDK, as such it is  recommend that you always upgrade to the latest SDK version as early as possible. 

Any request to DocumentDB using a retired SDK will be rejected by the service.

> [!WARNING]
> All versions of the Azure DocumentDB SDK for Node.js prior to version **1.0.0** will be retired on **February 29, 2016**. 
> 
> 
<br/>

| Version | Release Date | Retirement Date  |
| --- | --- | --- |
| [1.5.2](#1.5.2) |January 22, 2016 |--- |
| [1.5.1](#1.5.1) |January 4, 2016 |--- |
| [1.5.0](#1.5.0) |December 31, 2015 |--- |
| [1.4.0](#1.4.0) |October 06, 2015 |--- |
| [1.3.0](#1.3.0) |October 06, 2015 |--- |
| [1.2.2](#1.2.2) |September 10, 2015 |--- |
| [1.2.1](#1.2.1) |August 15, 2015 |--- |
| [1.2.0](#1.2.0) |August 05, 2015 |--- |
| [1.1.0](#1.1.0) |July 09, 2015 |--- |
| [1.0.3](#1.0.3) |June 04, 2015 |--- |
| [1.0.2](#1.0.2) |May 23, 2015 |--- |
| [1.0.1](#1.0.1) |May 15, 2015 |--- |
| [1.0.0](#1.0.0) |April 08, 2015 |--- |
| 0.9.4-prelease |April 06, 2015 |February 29, 2016 |
| 0.9.3-prelease |January 14, 2015 |February 29, 2016 |
| 0.9.2-prelease |December 18, 2014 |February 29, 2016 |
| 0.9.1-prelease |August 22, 2014 |February 29, 2016 |
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

