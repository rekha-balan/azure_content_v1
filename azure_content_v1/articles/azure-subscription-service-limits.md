<properties
    pageTitle="Microsoft Azure Subscription and Service Limits, Quotas, and Constraints"
    description="Provides a list of common Azure subscription and service limits, quotas, and constraints. This includes information on how to increase limits along with maximum values."
    services=""
    documentationCenter=""
    authors="rothja"
    manager="jeffreyg"
    editor="monicar"/>

<tags
    ms.service="multiple"
    ms.workload="multiple"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/03/2015"
    ms.author="jroth"/>

# Azure Subscription and Service Limits, Quotas, and Constraints
## Overview
This document specifies some of the most common Microsoft Azure limits. Note that this does not currently cover all Azure services. Over time, these limits will be expanded and updated to cover more of the platform.

> [!NOTE]
> If you want to raise the limit above the **Default Limit**, you can [open an online customer support request at no charge](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/). The limits cannot be raised above the **Maximum Limit** value in the tables below. If there is no **Maximum Limit** column, then the specified resource does not have adjustable limits.
> 
> 
## Limits and the Azure Resource Manager
It is now possible to combine multiple Azure resources in to a single Azure Resource Group. When using Resource Groups, limits that once were global become managed at a regional level with the Azure Resource Manager. For more information about Azure Resource Groups, see [Using resource groups to manage your Azure resources](resource-group-portal.md).

In the limits below, a new table has been added to reflect any differences in limits when using the Azure Resource Manager. For example, there is a **Subscription Limits** table and a **Subscription Limits - Azure Resource Manager** table. When a limit applies to both scenarios, it is only shown in the first table. Unless otherwise indicated, limits are global across all regions.

> [!NOTE]
> It is important to emphasize that quotas for resources in Azure Resource Groups are per-region accessible by your subscription, and are not per-subscription, as the service management quotas are. Let's use core quotas as an example. If you need to request a quota increase with support for cores, you need to decide how many cores you want to use in which regions, and then make a specific request for Azure Resource Group core quotas for the amounts and regions that you want. Therefore, if you need to use 30 cores in West Europe to run your application there; you should specifically request 30 cores in West Europe. But you will not have a core quota increase in any other region -- only West Europe will have the 30-core quota.
> <!-- -->
> As a result, you may find it useful to consider deciding what your Azure Resource Group quotas need to be for your workload in any one region, and request that amount in each region into which you are considering deployment. See [troubleshooting deployment issues](resource-group-deploy-debug.md##authentication-subscription-role-and-quota-issues) for more help discovering your current quotas for specific regions.
> 
> 
## Service-specific limits
* [Active Directory](#active-directory-limits.md)
* [API Management](#api-management-limits.md)
* [App Service](#app-service-limits.md)
* [Application Insights](#application-insights-limits.md)
* [Azure Redis Cache](#azure-redis-cache-limits.md)
* [Azure RemoteApp](#azure-remoteapp-limits.md)
* [Backup](#backup-limits.md)
* [Batch](#batch-limits.md)
* [BizTalk Services](#biztalk-services-limits.md)
* [CDN](#cdn-limits.md)
* [Cloud Services](#cloud-services-limits.md)
* [Data Factory](#data-factory-limits.md)
* [DNS](#dns-limits.md)
* [DocumentDB](#documentdb-limits.md)
* [IoT Hub](#iot-hub-limits.md)
* [Key Vault](#key-vault-limits.md)
* [Media Services](#media-services-limits.md)
* [Mobile Engagement](#mobile-engagement-limits.md)
* [Mobile Services](#mobile-services-limits.md)
* [Multi-Factor Authentication](#multi-factor-authentication.md)
* [Networking](#networking-limits.md)
* [Notification Hub Service](#notification-hub-service-limits.md)
* [Operational Insights](#operational-insights-limits.md)
* [Resource Group](#resource-group-limits.md)
* [Scheduler](#scheduler-limits.md)
* [Search](#search-limits.md)
* [Service Bus](#service-bus-limits.md)
* [Site Recovery](#site-recovery-limits.md)
* [SQL Database](#sql-database-limits.md)
* [Storage](#storage-limits.md)
* [StorSimple System](#storsimple-system-limits.md)
* [Stream Analytics](#stream-analytics-limits.md)
* [Subscription](#subscription-limits.md)
* [Traffic Manager](#traffic-manager-limits.md)
* [Virtual Machines](#virtual-machines-limits.md)

### Subscription Limits
#### Subscription Limits
Resource|Default Limit|Maximum Limit
---|---|---
Cores per [subscription](http://msdn.microsoft.com/library/azure/hh531793.aspx) <sup>1</sup>|20|10,000
[Co-administrators](http://msdn.microsoft.com/library/azure/gg456328.aspx) per subscription|200|200
[Storage accounts](storage-create-storage-account.md) per subscription|100|100
[Cloud services](cloud-services-what-is.md) per subscription|20|200
[Local networks](http://msdn.microsoft.com/library/jj157100.aspx) per subscription|10|500
SQL Database servers per subscription|6|150
DNS servers per subscription|9|100
Reserved IPs per subscription|20|100
ExpressRoute dedicated circuits per subscription|10|25
Hosted service certificates per subscription|400|400
[Affinity groups](../virtual-network/virtual-networks-migrate-to-regional-vnet.md) per subscription|256|256
[Batch](https://azure.microsoft.com/services/batch/) accounts per region per subscription|1|50
Alert rules per subscription|250|250

<sup>1</sup>Extra Small instances count as one core towards the core limit despite using a partial core. 


#### Subscription Limits - Azure Resource Manager
The following limits apply when using the Azure Resource Manager and Azure Resource Groups. Limits that have not changed with the Azure Resource Manager are not listed below. Please refer to the previous table for those limits.

Resource|Default Limit|Maximum Limit
---|---|---
VMs per [subscription](billing-buy-sign-up-azure-subscription.md)|20<sup>1</sup> per Region|10,000 per Region
[Co-administrators](billing-add-change-azure-subscription-administrator.md) per subscription|Unlimited|Unlimited
[Storage accounts](storage-create-storage-account.md) per subscription|100|100<sup>2</sup>
[Resource Groups](resource-group-overview.md) per subscription|800|800
[Availability Sets](../virtual-machines/virtual-machines-manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) per subscription|2000 per Region|2000 per Region
Resource Manager API Reads|15000 per hour|15000 per hour
Resource Manager API Writes|1200 per hour|1200 per hour
Resource Manager API request size|4194304 bytes|4194304 bytes
[Cloud services](cloud-services-what-is.md) per subscription|Deprecated<sup>3</sup>|Deprecated<sup>3</sup>
[Affinity groups](../virtual-network/virtual-networks-migrate-to-regional-vnet.md) per subscription|Deprecated<sup>3</sup>|Deprecated<sup>3</sup>

<sup>1</sup>Default limits vary by offer Category Type, such as Free Trial, Pay-As-You-Go,  etc.

<sup>2</sup>Limit can be increased by contacting support.

<sup>3</sup>These features are no longer required with Azure Resource Groups and the Azure Resource Manager. 


### Resource Group Limits
Resource|Default Limit|Maximum Limit
---|---|---
Resources per [resource group](resource-group-overview.md) (per resource type)|800|800
Deployments per resource group|800|800
Resources per deployment|800|800
Management Locks (per unique scope)|20|20
Number of Tags (per resource or resource group)|15|15
Tag key length|512|512
Tag value length|256|256


### Virtual Machines Limits
#### Virtual Machine Limits
Resource|Default Limit|Maximum Limit
---|---|---
[Virtual machines](../articles/virtual-machines/virtual-machines-about.md) per cloud service<sup>1</sup>|50|50
Input endpoints per cloud service<sup>2</sup>|150|150

<sup>1</sup>Virtual machines created in Service Management (instead of Resource Manager) are automatically stored in a cloud service. You can add more virtual machines to that cloud service for load balancing and availability. See  [How to Connect Virtual Machines with a Virtual Network or Cloud Service](../articles/virtual-machines/cloud-services-connect-virtual-machine.md).

<sup>2</sup>Input endpoints allow communications to a virtual machine from outside the virtual machine's cloud service. Virtual machines in the same cloud service or virtual network can automatically communicate with each other. See [How to Set Up Endpoints to a Virtual Machine](../articles/virtual-machines/virtual-machines-set-up-endpoints.md). 


#### Virtual Machines Limits - Azure Resource Manager
The following limits apply when using the Azure Resource Manager and Azure Resource Groups. Limits that have not changed with the Azure Resource Manager are not listed below. Please refer to the previous table for those limits.

Resource|Default Limit
---|---
Virtual machines per availability set | 100 
Certificates per subscription|Unlimited<sup>1</sup>

<sup>1</sup>With Azure Resource Manager, certificates are stored in the Azure Key Vault. Although the number of certificates is unlimited for a subscription, there is still a 1 MB limit of certificates per deployment (which consists of either a single VM or an availability set).


### Networking Limits
#### ExpressRoute Limits

The following limits apply to ExpressRoute resources per subscription.

| Resource | Default Limit |
|---|---|
| ExpressRoute circuits per subscription | 10 |
| ExpressRoute circuits per region per subscription for ARM | 10 |
| Maximum number of routes for Azure private peering with ExpressRoute standard | 4,000 |
| Maximum number of routes for Azure private peering with ExpressRoute premium add-on | 10,000 |
| Maximum number of routes for Azure public peering with ExpressRoute standard | 200 |
| Maximum number of routes for Azure public peering with ExpressRoute premium add-on | 200 |
| Maximum number of routes for Azure Microsoft peering with ExpressRoute standard | 200 |
| Maximum number of routes for Azure Microsoft peering with ExpressRoute premium add-on | 200 |
| Number of virtual network links allowed per ExpressRoute circuit | see table below |

#### Number of Virtual Networks per ExpressRoute circuit

| **Circuit Size** | **Number of VNet links for standard** | **Number of VNet Links with Premium add-on** |
|---|---|---|
| 10 Mbps | 10 | Not Supported |
| 50 Mbps | 10 | 20 |
| 100 Mbps | 10 | 25 |
| 200 Mbps | 10 | 25 |
| 500 Mbps | 10 | 40 |
| 1 Gbps | 10 | 50 |
| 2 Gbps | 10 | 60 |
| 5 Gbps | 10 | 75 |
| 10 Gbps | 10 | 100 |



#### Networking Limits
The following limits apply only for networking resources managed through the classic deployment model per subscription.

Resource| Default limit | Maximum limit
--- | --- | --- 
Virtual networks per subscription | 50 | 100
Local network sites per virtual network | 20 | contact support
DNS Servers per virtual network | 20 | 100
Virtual machines and role instances per virtual network | 2048 | 2048
Concurrent TCP connections for a virtual machine or role instance | 500K | 500K 
Network Security Groups (NSG) | 100 | 200
NSG rules per NSG | 200 | 400
User defined route tables | 100 | 200
User defined routes per route table | 100 | 500
Public IP addresses (dynamic) | 5 | contact support
Reserved public IP addresses | 20 | contact support
Public VIP per deployment | 5 | contact support
Private VIP (ILB) per deployment | 1 | 1
Endpoint Access Control Lists (ACLs) | 50 | 50


#### Networking Limits - Azure Resource Manager

The following limits apply only for networking resources managed through Azure Resource Manager per region per subscription.

Resource| Default limit | Maximum Limit
--- | --- | ---
Virtual networks per subscription | 50 | 500
DNS Servers per virtual network | 9 | 25
Virtual machines and role instances per virtual network | 2048 | 2048
Concurrent TCP connections for a virtual machine or role instance | 500K |500K
Network Interfaces (NIC) | 300 | 1000
Network Security Groups (NSG) | 100 | 400
NSG rules per NSG | 200 | 500
User defined route tables | 100 | 400
User defined routes per route table | 100 | 500
Public IP addresses (dynamic) | 60 | contact support
Reserved public IP addresses | 20 | contact support
Load balancers (internal and internet facing) | 100 | contact support
Load balancer rules per load balancer | 150 | 150
Public front end IP per load balancer | 5 | contact support
Private front end IP per load balancer | 1 | contact support
Application gateways | 50 | 50

Contact support in case you need to increase limits from default.


#### Traffic Manager Limits
Resource| Default limit
---|---
Profiles per subscription | 100 <sup>1</sup>
Endpoints per profile| 200

<sup>1</sup>Contact support in case you need to increase these limits.

#### DNS Limits

| Resource	| Default limit 
--- | ---
| Zones per subscription | 50 <sup>1</sup>
| Record sets per zone| 1000 <sup>1</sup>
| Records per record set| 20

<sup>1</sup> Contact support in case you need to increase these limits.
The Azure DNS service is currently in Preview.  These limits will be reviewed when the service reaches General Availability.

### Storage Limits
For additional details on storage account limits, see [Azure Storage Scalability and Performance Targets](../articles/storage/storage-scalability-targets.md).

#### Storage Service Limits
Resource|Default Limit
---|---
Max number of storage accounts per subscription|100<sup>1</sup>
TB per storage account|500 TB
Max number of blob containers, blobs, file shares, tables, queues, entities, or messages per storage account|Only limit is the 500 TB storage account capacity
Max size of a single blob container, table, or queue|500 TB
Max number of blocks in a block blob or append blob|50,000
Max size of a block in a block blob or append blob|4 MB
Max size of a block blob or append blob|50,000 X 4 MB (approx. 195 GB) 
Max size of a page blob |1 TB
Max size of a table entity|1 MB
Max number of properties in a table entity|252
Max size of a message in a queue|64 KB
Max size of a file share|5 TB
Max size of a file in a file share|1 TB
Max number of files in a file share|Only limit is the 5 TB total capacity of the file share
Max 8 KB IOPS per share|1000
Max number of files in a file share|Only limit is the 5 TB total capacity of the file share
Max number of blob containers, blobs, file shares, tables, queues, entities, or messages per storage account|Only limit is the 500 TB storage account capacity
Max number of stored access policies per container, file share, table, or queue|5
Total Request Rate (assuming 1KB object size) per storage account|Up to 20,000 IOPS, entities per second, or messages per second
Target throughput for single blob|Up to 60 MB per second, or up to 500 requests per second
Target throughput for single queue (1 KB messages)|Up to 2000 messages per second
Target throughput for single table partition (1 KB entities)|Up to 2000 entities per second
Target throughput for single file share|Up to 60 MB per second
Max ingress<sup>2</sup> per storage account (US Regions)|10 Gbps if GRS/ZRS<sup>3</sup> enabled, 20 Gbps for LRS
Max egress<sup>2</sup> per storage account (US Regions)|20 Gbps if RA-GRS/GRS/ZRS<sup>3</sup> enabled, 30 Gbps for LRS
Max ingress<sup>2</sup> per storage account (European and Asian Regions)|5 Gbps if GRS/ZRS<sup>3</sup> enabled, 10 Gbps for LRS
Max egress<sup>2</sup> per storage account (European and Asian Regions)|10 Gbps if RA-GRS/GRS/ZRS<sup>3</sup> enabled, 15 Gbps for LRS

<sup>1</sup>If you require more than 100 storage accounts, contact [Azure Support](https://azure.microsoft.com/support/faq/) for assistance.

<sup>2</sup>*Ingress* refers to all data (requests) being sent to a storage account. *Egress* refers to all data (responses) being received from a storage account.  

<sup>3</sup>Azure Storage replication options include:

- **RA-GRS**: Read-access geo-redundant storage. If RA-GRS is enabled, egress targets for the secondary location are identical to those for the primary location.
- **GRS**:  Geo-redundant storage. 
- **ZRS**: Zone-redundant storage. Available only for block blobs. 
- **LRS**: Locally redundant storage. 



#### Virtual Machine Disk Limits
An Azure virtual machine supports attaching a number of data disks. For optimal performance, you will want to limit the number of highly utilized disks attached to the virtual machine to avoid possible throttling. If all disks are not being highly utilized at the same time, the storage account can support a larger number disks.

- **For standard storage accounts:** A standard storage account has a maximum total request rate of 20,000 IOPS. The total IOPS across all of your virtual machine disks in a standard storage account should not exceed this limit.

	You can roughly calculate the number of highly utilized disks supported by a single standard storage account based on the request rate limit. For example, for a Basic Tier VM, the maximum number of highly utilized disks is about 66 (20,000/300 IOPS per disk), and for a Standard Tier VM, it is about 40 (20,000/500 IOPS per disk), as shown in the table below. 
 
- **For premium storage accounts:** A premium storage account has a maximum total throughput rate of 50 Gbps. The total throughput across all of your VM disks should not exceed this limit.

See [Virtual machine sizes](../articles/virtual-machines/virtual-machines-size-specs.md) for additional details.

**Standard storage accounts**

**Virtual machine disks: per disk limits**

 VM Tier | Basic Tier VM | Standard Tier VM
---|---|---
Disk size | 1023 GB | 1023 GB
Max 8 KB IOPS per persistent disk | 300 | 500
Max number of highly utilized disks | 66 | 40



**Premium storage accounts**

**Virtual machine disks: per account limits**

Resource|Default Limit
---|---
Total disk capacity per account|35 TB
Total snapshot capacity per account|10 TB
Max bandwidth per account (ingress + egress<sup>1</sup>)|<=50 Gbps

<sup>1</sup>*Ingress* refers to all data (requests) being sent to a storage account. *Egress* refers to all data (responses) being received from a storage account.

**Virtual machine disks: per disk limits**

Premium Storage Disk Type | P10 | P20 | P30
---|---|---|---
Disk size | 128 GiB | 512 GiB | 1024 GiB (1 TB)
Max IOPS per disk | 500 | 2300 | 5000
Max throughput per disk | 100 MB per second | 150 MB per second | 200 MB per second
Max number of highly utilized disks | 62 | 41 | 31




#### Storage Resource Provider Limits
The following limits apply when using the Azure Resource Manager and Azure Resource Groups only.

Resource|Default Limit
---|---
Storage account management operations (read)|800 per 5 minutes
Storage account management operations (write)|200 per hour
Storage account management operations (list)|100 per 5 minutes


### Cloud Services Limits
Resource|Default Limit|Maximum Limit
---|---|---
[Web/worker roles per deployment](cloud-services-what-is.md)<sup>1</sup>|25|25
[Instance Input Endpoints](http://msdn.microsoft.com/library/gg557552.aspx#InstanceInputEndpoint) per deployment|25|25
[Input Endpoints](http://msdn.microsoft.com/library/gg557552.aspx#InputEndpoint) per deployment|25|25
[Internal Endpoints](http://msdn.microsoft.com/library/gg557552.aspx#InternalEndpoint) per deployment|25|25

<sup>1</sup>Each Cloud Service with Web/Worker roles can have two deployments, one for production and one for staging. Also note that this limit refers to the number of distinct roles (configuration) and not the number of instances per role (scaling).


### App Service Limits
The following App Service limits include limits for Web Apps, Mobile Apps, API Apps, and Logic Apps.

Resource|Free|Shared (Preview)|Basic|Standard|Premium (Preview)</th>
---|---|---|---|---|---
[Web, mobile, or API apps](../services/app-service/) per [App Service plan](web-sites-web-hosting-plan-overview.md)<sup>1</sup>|10|100|Unlimited<sup>2</sup>|Unlimited<sup>2</sup>|Unlimited<sup>2</sup>
[Logic apps](../services/app-service/) per [App Service plan](web-sites-web-hosting-plan-overview.md)</a><sup>1</sup>|10|10|10|20 per core|20 per core 
[App Service plan](web-sites-web-hosting-plan-overview.md)|1 per region|10 per resource group|10 per resource group|10 per resource group|10 per resource group
Compute instance type|Shared|Shared|Dedicated<sup>3</sup>|Dedicated<sup>3</sup>|Dedicated<sup>3</sup></p>
[Scale-Out](web-sites-scale.md) (max instances)|1 shared|1 shared|3 dedicated<sup>3</sup>|10 dedicated<sup>3</sup>|50 dedicated<sup>3,4</sup>
Storage<sup>5</sup>|1 GB<sup>5</sup>|1 GB<sup>5</sup>|10 GB<sup>5</sup>|50 GB<sup>5</sup>|500 GB<sup>4,5</sup></p>
CPU time (day)<sup>6</sup>|60 minutes|240 minutes|Unlimited, pay at standard [rates](../pricing/details/app-service/)</a>|Unlimited, pay at standard rates|Unlimited, pay at standard rates
Memory (1 hour)|1024 MB per App Service plan|1024 MB per app|N/A|N/A|N/A
Bandwidth|165 MB|Unlimited, [data transfer rates](../pricing/details/data-transfers/) apply|Unlimited, data transfer rates apply|Unlimited, data transfer rates apply|Unlimited, data transfer rates apply
Application architecture|32-bit|32-bit|32-bit/64-bit|32-bit/64-bit|32-bit/64-bit
Web Sockets per instance<sup>7</sup>|5|35|350|Unlimited|Unlimited
Concurrent [debugger connections](web-sites-dotnet-troubleshoot-visual-studio.md) per application|1|1|1|5|5
[azurewebsites.net subdomain with FTP/S and SSL](web-sites-configure-ssl-certificate.md)|X|X|X|X|X
[Custom domain](web-sites-custom-domain-name.md) support||X|X|X|X
Custom domain [SSL support](web-sites-configure-ssl-certificate.md)|||Unlimited|Unlimited, 5 SNI SSL and 1 IP SSL connections included|Unlimited, 5 SNI SSL and 1 IP SSL connections included
Integrated Load Balancer||X|X|X|X
[Always On](web-sites-configure.md)|||X|X|X
[Scheduled Backups](web-sites-backup.md)||||Once per day|Once every 5 minutes<sup>8</sup>
[Auto Scale](web-sites-scale.md)|||X|X|X
[WebJobs](web-sites-create-web-jobs.md)<sup>9</sup>|X|X|X|X|X
[Azure Scheduler](../services/scheduler/) support||X|X|X|X
[Endpoint monitoring](web-sites-monitor.md)|||X|X|X
[Staging Slots (Preview)](web-sites-staged-publishing.md)||||5|20
Custom domains per app</a>||500|500|500|500
SLA||<p>|99.9%|99.95%<sup>10</sup>|99.95%<sup>10</sup>

<sup>1</sup>Apps and storage quotas are per App Service plan unless noted otherwise.  
<sup>2</sup>The actual number of apps that you can host on these machines depends on the activity of the apps, the size of the machine instances, and the corresponding resource utilization.  
<sup>3</sup>Dedicated instances can be of different sizes. See [App Service Pricing](/pricing/details/app-service/) for more details. Additional instances are available by opening a support request.  
<sup>4</sup>Premium tier allows up to 50 computes instances (subject to availability) and 500 GB of disk space when using App Service Environments, and 20 compute instances and 250 GB storage otherwise.  
<sup>5</sup>The storage limit is the total content size across all apps in the 
same App Service plan. Storage limits can be increased by opening a support request.  
<sup>6</sup>These resources are constrained by physical resources on the dedicated instances (the instance size and the number of instances).  
<sup>7</sup>If you scale an app in the Basic tier to two instances, you have 350 concurrent connections for each of the two instances.  
<sup>8</sup>Premium tier allows backup intervals down up to every 5 minutes when using App Service Environments, and 50 times per day otherwise.  
<sup>9</sup>Run custom executables and/or scripts on demand, on a schedule, or continuously as a background task within your App Service instance. Always On is required for continuous WebJobs execution. Azure Scheduler Free or Standard is required for scheduled WebJobs.  
<sup>10</sup>SLA of 99.95% provided for deployments that use multiple instances with Azure Traffic Manager configured for failover.  


### Scheduler Limits
The following table describes each of the major quotas, limits, defaults, and throttles in Azure Scheduler.

|Resource|Limit Description|
|---|---|
|**Job size**|The maximum job size is 16K. If a PUT or a PATCH results in a job larger than these limits, a 400 Bad Request status code is returned.|
|**Request URL size**|Maximum size of the request URL is 2048 chars.|
|**Aggregate header size**|Maximum aggregate header size is 4096 chars.|
|**Header count**|Maximum header count is 50 headers.|
|**Body size**|Maximum body size is 8192 chars.|
|**Recurrence span**|Maximum recurrence span is 18 months.|
|**Time to start time**|Maximum “time to start time” is 18 months.|
|**Job history**|Maximum response body stored in job history is 2048 bytes.|
|**Frequency**|The default max frequency quota is 1 hour in a free job collection and 1 minute in a standard job collection. The max frequency is configurable on a job collection to be lower than the maximum. All jobs in the job collection are limited the value set on the job collection. If you attempt to create a job with a higher frequency than the maximum frequency on the job collection then request will fail with a 409 Conflict status code.|
|**Jobs**|The default max jobs quota is 5 jobs in a free job collection and 50 jobs in a standard job collection. The maximum number of jobs is configurable on a job collection. All jobs in the job collection are limited the value set on the job collection. If you attempt to create more jobs than the maximum jobs quota, then the request fails with a 409 Conflict status code.|
|**Job history retention**|Job history is retained for up to 2 months or up to the last 1000 executions.|
|**Completed and faulted job retention**|Completed and faulted jobs are retailed for 60 days.|
|**Timeout**|There’s a static (not configurable) request timeout of 30 seconds for HTTP actions. For longer running operations, follow HTTP asynchronous protocols; for example, return a 202 immediately but continue working in the background.|


### Batch Limits
Resource|Default Limit|Maximum Limit
---|---|---
Cores per Batch account|20|N/A<sup>1</sup>
Jobs and job schedules<sup>2</sup> per Batch account|20|10,000
Pools per Batch account|20|5000

<sup>1</sup> The number of cores per Batch account can be increased, but the maximum number is unspecified. Contact customer support to discuss increase options.
<sup>2</sup> Includes run-once active jobs and active job schedules. Completed jobs and job schedules are not limited.


### BizTalk Services Limits
The following table shows the limits for Azure Biztalk Services.

|RESOURCE|FREE (PREVIEW)|DEVELOPER|BASIC|STANDARD|PREMIUM|
|---|---|---|---|---|---|
|Scale out|N/A|N/A|Yes, in increments of 1 Basic Unit |Yes, in increments of 1 Standard Unit |Yes, in increments of 1 Premium Unit |
|Scale Limit|N/A|N/A|Up to 8 units |Up to 8 units |Up to 8 units|
|EAI Bridges per Unit|N/A|25|25|125|500|
|EDI Agreements per Unit|N/A|10|50|250|1000|
|Hybrid Connections per Unit|5|5|10|50|100|
|Hybrid Connection Data Transfer (GBs) per Unit|5|5|50|250|500|
|Number of connections using BizTalk Adapter Service per Unit|N/A|1|2|5|25|
|Archiving|N/A|Available|N/A|N/A|Available|
|High Availability |N/A|N/A|Available|Available|Available|


### DocumentDB Limits
=======
Entity|Quota (Standard Offer)
---|---
Database Accounts*|5
Number of databases per database account|100
Number of users per database account – across all databases|500,000
Number of permissions per database account – across all databases|2,000,000
Attachment storage per database account (Preview Feature)|2 GB
Maximum Request Units / second per collection|2500
Number of stored procedures, triggers and UDFs per collection* |25 each
Maximum execution time for stored procedure and trigger|5 seconds
Provisioned document storage / collection|10 GB
Maximum collections per database account*|100
Maximum document storage per database (100 collections)* |1 TB
Maximum Length of the Id property|255 characters
Maximum items per page|No practical limit
Maximum request size of document and attachment |512KB
Maximum request size of stored procedure, trigger and UDF|512KB
Maximum response size|1MB
String|All strings must conform to the UTF-8 encoding. Since UTF-8 is a variable width encoding, string sizes are determined using the UTF-8 bytes.
Maximum length of property or value|No practical limit
Maximum number of UDFs per query* |2
Maximum number of JOINs per query* |5
Maximum number of AND clauses per query* |20
Maximum number of OR clauses per query* |20
Maximum number of values per IN expression* |200
Maximum number of points in a polygon argument in a ST_WITHIN query* |16
Maximum number of collection creates per minute* |5
Maximum number of scale operations per minute* |5

Quotas listed with an asterisk (*) [can be adjusted by contacting Azure support](../articles/documentdb/documentdb-increase-limits.md).


### Mobile Engagement Limits
Resource|Maximum Limit
---|---
App Collection Users|5 per App Collection
Average Data points|200 per Active User/Day
Average App-Info set|50 per Active User/Day
Average Messages pushed|20 per Active User/Day
Segments|100 per app
Criteria per segment|10
Active Push Campaigns|50 per app
Total Push Campaigns (includes Active & Completed)|1000 per app


### Search Limits
The pricing tier determines the capacity and limits of your search service.

#### Standard Tier
Object|Limit
---|---
Maximum number of indexes|50 per Search service
Maximum number of fields per index|1000
Maximum document count|15 million per partition
Maximum storage size|25 GB per partition
Maximum partitions|12 per Search service
Maximum replicas|12 per Search service
Maximum search units|36 per Search service
Maximum search services|12 per Azure subscription
Maximum number of indexers|50 per Search service
Maximum number of datasources|50 per Search service
Maximum number of documents that can be indexed in a single indexer invocation|Unlimited
Maximum number of scoring profiles per index|16
Maximum number of functions per profile|8


#### Shared Tier (part of a multi-tenant service, free to Azure subscribers)
Object|Limit
---|---
Maximum number of indexes|3 per Search service
Maximum number of fields per index|1000
Maximum document count|10,000
Maximum storage size|50 MB
Maximum partitions|N/A
Maximum replicas|N/A
Maximum search units|N/A
Maximum search services|N/A
Maximum number of indexers|3
Maximum number of datasources|3
Maximum number of documents that can be indexed in a single indexer invocation|10,000
Maximum indexer running time|3 minutes
Maximum number of scoring profiles per index|16
Maximum number of functions per profile|8


To learn more about limits on keys, replica-partition combinations, requests, responses, and how to achieve high availability for different workloads, see [Service limits in Azure Search](search/search-limits-quotas-capacity.md).

### Media Services Limits
Resource|Default Limit|Maximum Limit
---|---|---
Azure Media Services (AMS) accounts in a single subscription||25
Assets per AMS account||1,000,000<sup>1</sup>
Chained tasks per job||30
Assets per task||50
Assets per job||100
Jobs per AMS account ||50,000<sup>2</sup>
Unique locators associated with an asset at one time||5<sup>4</sup>
Live channels per AMS account </p></td>|5</p></td>|N/A<sup>1</sup>
Programs in stopped state per channel </p></td>|50</p></td>|N/A<sup>1</sup>
Programs in running state per channel </p></td>|3</p></td>|3
Streaming endpoints in running state per AMS account</p></td>|2</p></td>|N/A<sup>1</sup>
Streaming units per streaming endpoint </p></td>|10 </p></td>|N/A<sup>1</sup>
Encoding units per AMS account </p></td>|25</p></td>|N/A<sup>1</sup>
Storage accounts | |1,000<sup>5</sup>

<sup>1</sup> You can request to update the limits for this quota by opening a support ticket. Do not create more AMS accounts to increase limits, instead submit a support ticket.

<sup>2</sup> This number includes queued, finished, active, and canceled jobs. It does not include deleted jobs. You can delete the old jobs using **IJob.Delete** or the **DELETE** HTTP request.

<sup>3</sup> When making a request to list Job entities, a maximum of 1,000 will be returned per request. If you need to keep track of all submitted Jobs, you can use top/skip as described in [OData system query options](http://msdn.microsoft.com/library/gg309461.aspx).

<sup>4</sup> Locators are not designed for managing per-user access control. To give different access rights to individual users, use Digital Rights Management (DRM) solutions.

<sup>5</sup> The storage accounts must be from the same Azure subscription.


### CDN Limits

Resource | Soft limit
---------|-----------
CDN profiles | 4
CDN endpoints per profile | 10
Custom domains per endpoint | 10 

Request an update to your subscription's soft limits by opening a support ticket.


### Mobile Services Limits

| TIER: | FREE | BASIC | STANDARD |
|----|----|----|----|
| API Calls | 500 K | 1.5 M / unit | 15 M / unit |
| Active Devices | 500 | Unlimited | Unlimited |
| Scale | N/A | Up to 6 units | Unlimited units |
| Push Notifications | Notification Hubs Free Tier included, up to 1 M pushes | Notification Hubs Basic Tier included, up to 10 M pushes | Notification Hubs Standard Tier included, up to 10 M pushes |
| Real time messaging/<br/>Web Sockets | Limited | 350 / mobile service | Unlimited |
| Offline synchronizations | Limited | Included | Included |
| Scheduled jobs  | Limited | Included | Included |
| SQL Database (required) <br/>Standard rates apply for additional capacity | 20 MB included | 20 MB included | 20 MB included |
| CPU capacity | 60 minutes / day | Unlimited | Unlimited |
| Outbound data transfer | 165 MB per day (daily Rollover) | Included | Included |

For additional details on these limits and for information on pricing, see [Mobile Services Pricing](https://azure.microsoft.com/pricing/details/mobile-services/). 


### Notification Hub Service Limits

| TIER: | FREE | BASIC | STANDARD |
|----|----|----|----|
| Included Pushes | 1 Million | 10 Million | 10 Million |
| Active Devices | 500 | Unlimited | Unlimited |
| Broadcast Tag Size | 10K | 10K | Unlimited |
| # of tags (broadcast groups) | 3K | 3K unless broadcasted to less than 5 devices | Unlimited |

For additional details on these limits and for information on pricing, see [Notification Hubs Pricing](https://azure.microsoft.com/pricing/details/notification-hubs/). 


### Service Bus Limits
The following table lists quota information specific to Service Bus messaging. Event Hubs limits are included in this table, but for more specific information about Event Hubs, see [Event Hubs Pricing](https://azure.microsoft.com/pricing/details/event-hubs/). For information about pricing and other quotas for Service Bus, see the [Service Bus Pricing](https://azure.microsoft.com/pricing/details/service-bus/) overview.

|Quota Name|Scope|Type|Behavior when exceeded|Value|
|---|---|---|---|---|
| Maximum number of namespaces per Azure subscription|Namespace|Static|Subsequent requests for additional namespaces will be rejected by the portal.|100|
|Queue/topic size|Entity|Defined upon creation of the queue/topic.|Incoming messages will be rejected and an exception will be received by the calling code.|1,2,3,4 or 5 GB.<br /><br />If [partitioning](service-bus-partitioning.md) is enabled, the maximum queue/topic size is 80 GB.|
|Number of concurrent connections on a namespace|Namespace|Static|Subsequent requests for additional connections will be rejected and an exception will be received by the calling code. REST operations do not count towards concurrent TCP connections.|NetMessaging: 1,000<br /><br />AMQP: 5,000|
|Number of concurrent connections on a queue/topic/subscription entity|Entity|Static|Subsequent requests for additional connections will be rejected and an exception will be received by the calling code. REST operations do not count towards concurrent TCP connections.|Capped by the limit of concurrent connections per namespace.|
|Number of concurrent receive requests on a queue/topic/subscription entity|Entity|Static|Subsequent receive requests will be rejected and an exception will be received by the calling code. This quota applies to the combined number of concurrent receive operations across all subscriptions on a topic.|5,000|
|Number of concurrent listeners on a relay|Entity|Static|Subsequent requests for additional connections will be rejected and an exception will be received by the calling code.|25|
|Number of concurrent relay listeners|System-wide|Static|Subsequent requests for additional connections will be rejected and an exception will be received by the calling code.|2,000|
|Number of concurrent relay connections per all relay endpoints in a service namespace|System-wide|Static|-|5,000|
|Number of relay endpoints per service namespace|System-wide|Static|-|10,000|
|Number of topics/queues per service namespace|System-wide|Static|Subsequent requests for creation of a new topic or queue on the service namespace will be rejected. As a result, if configured through the [Azure classic portal][], an error message will be generated. If called from the management API, an exception will be received by the calling code.|10,000<br /><br />The total number of topics plus queues in a service namespace must be less than or equal to 10,000.|
|Number of partitioned topics/queues per service namespace|System-wide|Static|Subsequent requests for creation of a new partitioned topic or queue on the service namespace will be rejected. As a result, if configured through the [Azure classic portal][], an error message will be generated. If called from the management API, a **QuotaExceededException** exception will be received by the calling code.|100<br /><br />Each partitioned queue or topic counts towards the quota of 10,000 entities per namespace.|
|Maximum size of any messaging entity name: namespace, queue, topic, subscription, or event hub|Entity|Static|-|50 characters|
|Maximum size of Event Hubs event|System-wide|Static|-|256 KB|
|Message size for a queue/topic/subscription entity|System-wide|Static|Incoming messages that exceed these quotas will be rejected and an exception will be received by the calling code.|Maximum message size: 256KB. <br /><br />**Note** Due to system overhead, this limit is usually slightly less than 256KB.<br /><br />Maximum header size: 64KB<br /><br />Maximum number of header properties in property bag: **MaxValue**<br /><br />Maximum size of property in property bag: No explicit limit. Limited by maximum header size.|
|Message size for [NetOnewayRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.netonewayrelaybinding.aspx) and [NetEventRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.neteventrelaybinding.aspx) relays|System-wide|Static|Incoming messages that exceed these quotas will be rejected and an exception will be received by the calling code.|64KB
|Message size for [HttpRelayTransportBindingElement](https://msdn.microsoft.com/library/microsoft.servicebus.httprelaytransportbindingelement.aspx) and [NetTcpRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.nettcprelaybinding.aspx) relays|System-wide|Static|-|Unlimited|
|Message property size for a queue/topic/subscription entity|System-wide|Static|A **SerializationException** exception is generated.|Maximum message property size for each property is 32K. Cumulative size of all properties cannot exceed 64K. This applies to the entire header of the [BrokeredMessage](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.aspx), which has both user properties as well as system properties (such as [SequenceNumber](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.sequencenumber.aspx), [Label](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.label.aspx), [MessageId](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx), and so on).|
|Number of subscriptions per topic|System-wide|Static|Subsequent requests for creating additional subscriptions for the topic will be rejected. As a result, if configured through the portal, an error message will be shown. If called from the management API an exception will be received by the calling code.|2,000|
|Number of SQL filters per topic|System-wide|Static|Subsequent requests for creation of additional filters on the topic will be rejected and an exception will be received by the calling code.|2,000|
|Number of correlation filters per topic|System-wide|Static|Subsequent requests for creation of additional filters on the topic will be rejected and an exception will be received by the calling code.|100,000|
|Size of SQL filters/actions|System-wide|Static|Subsequent requests for creation of additional filters will be rejected and an exception will be received by the calling code.|Maximum length of filter condition string: 1024 (1K).<br /><br />Maximum length of rule action string: 1024 (1K).<br /><br />Maximum number of expressions per rule action: 32.|

[Azure classic portal]: http://manage.windowsazure.com

### IoT Hub Limits
The following table lists the limits associated with the the different service tiers (S1, S2, F1). For information about the cost of each *unit* in each tier, see [IoT Hub Pricing](https://azure.microsoft.com/pricing/details/iot-hub/).

| Resource | S1 Standard | S2 Standard | F1 Free |
| -------- | ----------- | ----------- | ------- |
| Messages/day | 400,000 | 6,000,000 | 8,000 |
| Maximum units | 200 | 200 | 1 |

> [AZURE.NOTE] If you anticipate using more than 200 units with an S1 or S2 tier hub, please contact Microsoft support.

The following table lists the limits that apply to IoT Hub resources:

| Resource | Limit |
| -------- | ----- |
| Maximum IoT hubs per Azure subscription | 10 |
| Maximum number of device identities<br/>  returned in a single call | 1000 |
| IoT Hub message maximum retention | 7 days |
| Maximum size of device-to-cloud message | 256 KB |
| Maximum size of device-to-cloud batch | 256 KB |
| Maximum messages in device-to-cloud batch | 500 |
| Maximum size of cloud-to-device message | 64 KB |
| Maximum TTL for cloud-to-device messages | 2 days |
| Maximum delivery count for cloud-to-device <br/> messages | 100 |
| Maximum delivery count for feedback messages <br/> in response to a cloud-to-device message | 100 |
| Maximum TTL for feedback messages in <br/> response to a cloud-to-device message | 2 days |

The IoT Hub service throttles requests when the following quotas are exceeded:

| Throttle | Per-hub value |
| -------- | ------------- |
| Identity registry operations <br/> (create, retrieve, list, update, delete), <br/> individual or bulk import/export | 100/min/unit, up to 5000/min |
| Device connections | 120/sec/unit (for S2), 12/sec/unit (for S1). Minimum of 100/sec. |
| Device-to-cloud sends | 120/sec/unit (for S2), 12/sec/unit (for S1) <br/> Minimum of 100/sec |
| Cloud-to-device sends | 100/min/unit |
| Cloud-to-device receives | 1000/min/unit |


### Data Factory Limits
Data factory is a multi-tenant service that has the following default limits in place to make sure customer subscriptions are protected from each others workloads. Many of the limits can be easily raised for your subscription up to the maximum limit by contacting support. 

**Resource** | **Default Limit** | **Maximum Limit**
-------- | ------------- | -------------
pipelines within a data factory | 100 | 2500
datasets within a data factory | 500 | 5000
concurrent slices per dataset | 10 | 10
bytes per object for pipeline objects <sup>1</sup> | 200 KB | 2000 KB
bytes per object for dataset and linkedservice objects <sup>1</sup> | 30 KB | 2000 KB
fields per object | 100 | [Contact support](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
bytes per field name or identifier | 2 KB | [Contact support](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
bytes per field | 30 KB | [Contact support](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
HDInsight on-demand cluster cores within a subscription <sup>2</sup> | 48 | [Contact support](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
Retry count for pipeline activity runs | 1000 | MaxInt (32 bit)

<sup>1</sup> Pipeline, dataset, and linked service objects represent a logical grouping of your workload. Limits for these objects do not relate to amount of data you can move and process with the Azure Data Factory service. Data factory is designed to scale to handle petabytes of data.

<sup>2</sup>On-demand HDInsight cores are allocated out of the subscription that contains the data factory. As a result, the above limit is the Data Factory enforced core limit for on-demand HDInsight cores and is different from the core limit associated with your Azure subscription.


**Resource** | **Default lower limit** | **Minimum limit**
-------- | ------------------- | -------------
Scheduling interval | 15 minutes | 15 minutes
Interval between retry attempts | 1 second | 1 second
Retry timeout value | 1 second | 1 second


### Web service call limits

Azure resource manager has limits for API calls. You can make API calls at a rate within the [Azure Resource Manager API limits](azure-subscription-service-limits/#resource-group-limits). 




### Stream Analytics Limits
<properties 
   pageTitle="Stream Analytics limits table"
   description="Describes system limits and recommended sizes for Stream Analytics components and connections."
   services="stream-analytics"
   documentationCenter="NA"
   authors="jeffstokes72"
   manager="paulettm"
   editor="cgronlun" />
<tags 
   ms.service="stream-analytics"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="big-data"
   ms.date="10/22/2015"
   ms.author="jeffstok" />

| Limit identifier | Limit       | Comments |
|----------------- | ------------|--------- |
| Maximum number of Streaming Units per subscription per region | 50 | A request to increase streaming units for your subscription beyond 50 can be made by contacting [Microsoft Support](https://support.microsoft.com/en-us). |
| Maximum throughput of a Streaming Unit | 1MB/s* | Maximum throughput per SU depends on the scenario. Actual throughput may be lower and depends upon query complexity and partitioning. Further details can be found in the [Scale Azure Stream Analytics jobs to increase throughput](../articles/stream-analytics/stream-analytics-scale-jobs.md) article. |
| SELECT statement query limitation | 5 outputs per query | This limit may be increased in the future. |



### Active Directory Limits
Here are the usage constraints and other service limits for the Azure Active Directory service.

| Category | Limits |
|---|---|
| Directories | A single user can only be associated with a maximum of 20 Azure Active Directory directories.<br />Examples of possible combinations: <ul> <li>A single user creates 20 directories.</li><li>A single user is added to 20 directories as a member.</li><li>A single user creates 10 directories and later is added by others to 10 different directories.</li></ul> |  
| Objects | <ul><li>A maximum of 500,000 objects can be used in a single directory by users of the Free edition of Azure Active Directory.</li><li>A non-admin user can create no more than 250 objects.</li></ul> |
| Schema extensions | <ul><li>String type extensions can have maximum of 256 characters. </li><li>Binary type extensions are limited to 256 bytes.</li><li>100 extension values (across ALL types and ALL applications) can be written to any single object.</li><li>Only “User”, “Group”, “TenantDetail”, “Device”, “Application” and “ServicePrincipal” entities can be extended with “String” type or “Binary” type single-valued attributes.</li><li>Schema extensions are available only in Graph API-version 1.21-preview. The application must be granted write access to register an extension.</li></ul> |
| Applications | A maximum of 10 users can be owners of a single application. |
| Groups | <ul><li>A maximum of 10 users can be owners of a single group.</li><li>Any number of objects can be members of a single group in Azure Active Directory.</li><li>The number of members in a group you can synchronize from your on-premises Active Directory to Azure Active Directory is limited to 15K members, using Azure Active Directory Directory Synchronization (DirSync).</li><li>The number of members in a group you can synchronize from your on-premises Active Directory to Azure Active Directory using Azure AD Connect is limited to 50K members.</li></ul> |
| Access Panel | <ul><li>There is no limit to the number of applications that can be seen in the Access Panel per end user, for users assigned licenses for Azure AD Premium or the Enterprise Mobility Suite.</li><li>A maximum of 10 app tiles (examples: Box, Salesforce, or Dropbox) can be seen in the Access Panel for each end user for users assigned licenses for Free or Azure AD Basic editions of Azure Active Directory. This limit does not apply to Administrator accounts.</li></ul> |
| Reports | A maximum of 1,000 rows can be viewed or downloaded in any report. Any additional data is truncated. |


### Azure RemoteApp Limits

|Resource | Default limit|
|--------------|--------|
|Collections per user| 1|
|Published apps per collection|	100|
|Trial collection duration| 30 days|
|Trial collections| 2 per subscription|
|Users per trial collection| 10|
|Trial template images|	25|
|Paid collections| 3 (you can request an increase)|
|Paid template images| 25|
|Users - basic tier*| 400 (default)/ 800 (maximum)|
|Users - standard tier*| 250 (default)/ 500 (maximum)|
|Users- premium tier| 100 default. You can request an increase.|
|Users - premium plus tier | 50 default. You can request an increase.|
|Concurrent connections across all collections in a subscription| 5000 (you can request an increase)|
|User data storage (UPD) per user per collection| 50 GB|
|Idle timeout| 4 hours|
|Disconnected timeout| 4 hours|

*User limits in basic and standard tiers cannot be increased beyond the maximum limit listed above. 

The number of users is determined by the number of VMs used for your collection:

- Basic = 16 users per VM
- Standard = 10 users per VM
- Premium = 4 users per VM
- Premium plus = 2 users per VM

To request an increase for number of paid collections or concurrent connections, send an email to [remoteappforum@microsoft.com](mailto:remoteappforum@microsoft.com) with details of what you need, including your subscription ID. 

### StorSimple System Limits
<!--author=alkohli last changed: 12/15/15-->

| Limit identifier | Limit | Comments |
|----------------- | ------|--------- |
| Maximum number of storage account credentials | 64 | |
| Maximum number of volume containers | 64 | |
| Maximum number of volumes | 255 | |
| Maximum number of schedules per bandwidth template | 168 | A schedule for every hour, every day of the week (24*7). |
| Maximum size of a tiered volume on physical devices | 64 TB for 8100 and 8600 | 8100 and 8600 are physical devices. |
| Maximum size of a tiered volume on virtual devices in Azure | 30 TB for 8010 <br></br> 64 TB for 8020 | 8010 and 8020 are virtual devices in Azure that use Standard Storage and Premium Storage respectively. |
| Maximum size of a locally pinned volume on physical devices | 9 TB for 8100 <br></br> 24 TB for 8600 | 8100 and 8600 are physical devices. |
| Maximum number of iSCSI connections | 512 | |
| Maximum number of iSCSI connections from initiators | 512 | |
| Maximum number of access control records per device | 64 | |
| Maximum number of volumes per backup policy | 24 | |
| Maximum number of backups retained per backup policy | 64 | |
| Maximum number of schedules per backup policy | 10 | |
| Maximum number of snapshots of any type that can be retained per volume | 256 | This includes local snapshots and cloud snapshots. |
| Maximum number of snapshots that can be present in any device | 10,000 | |
| Maximum number of volumes that can be processed in parallel for backup, restore, or clone | 16 |<ul><li>If there are more than 16 volumes, they will be processed sequentially as processing slots become available.</li><li>New backups of a cloned or a restored tiered volume cannot occur until the operation is finished. However, for a local volume, backups are allowed after the volume is online.</li></ul>|
| Restore and clone recover time for tiered volumes | < 2 minutes | <ul><li>The volume is made available within 2 minutes of restore or clone operation, regardless of the volume size.</li><li>The volume performance may initially be slower than normal as most of the data and metadata still resides in the cloud. Performance may increase as data flows from the cloud to the StorSimple device.</li><li>The total time to download metadata depends on the allocated volume size. Metadata is automatically brought into the device in the background at the rate of 5 minutes per TB of allocated volume data. This rate may be affected by Internet bandwidth to the cloud.</li><li>The restore or clone operation is complete when all the metadata is on the device.</li><li>Backup operations cannot be performed until the restore or clone operation is fully complete.|
| Restore recover time for locally pinned volumes | < 2 minutes | <ul><li>The volume is made available within 2 minutes of the restore operation, regardless of the volume size.</li><li>The volume performance may initially be slower than normal as most of the data and metadata still resides in the cloud. Performance may increase as data flows from the cloud to the StorSimple device.</li><li>The total time to download metadata depends on the allocated volume size. Metadata is automatically brought into the device in the background at the rate of 5 minutes per TB of allocated volume data. This rate may be affected by Internet bandwidth to the cloud.</li><li>Unlike tiered volumes, in the case of locally pinned volumes, the volume data is also downloaded locally on the device. The restore operation is complete when all the volume data has been brought to the device.</li><li>The restore operations may be long and the total time to complete the restore will depend on the size of the provisioned local volume, your Internet bandwidth and the existing data on the device. Backup operations on the locally pinned volume are allowed while the restore operation is in progress.|
| Thin-restore availability | Last failover | |
| Maximum client read/write throughput (when served from the SSD tier)* | 920/720 MB/s with a single 10GbE network interface | Up to 2x with MPIO and two network interfaces. |
| Maximum client read/write throughput (when served from the HDD tier)* | 120/250 MB/s |
| Maximum client read/write throughput (when served from the cloud tier)* | 11/41 MB/s | Read throughput depends on clients generating and maintaining sufficient I/O queue depth. |

&#42; Maximum throughput per I/O type was measured with 100 percent read and 100 percent write scenarios. Actual throughput may be lower and depends on I/O mix and network conditions.


### Operational Insights Limits
<properties
   pageTitle="Operational Insights limits table"
   description="Describes system limits for Operational Insights."
   services="operational-insights"
   documentationCenter="NA"
   authors="bandersmsft"
   manager="jwhit"
   editor="" />
<tags
   ms.service="operational-insights"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="07/01/2015"
   ms.author="banders" />


The following limits apply to Operational Insights subscriptions.


|   |FREE|STANDARD|PREMIUM|
|---|---|---|---|
|Daily data transfer limit|500 MB<sup>1</sup>|None|None|
|Data retention period|7 days|1 month|12 months|
|Data storage limit|500 MB * 7 days = 3.5 GB|unlimited|unlimited|


<sup>1</sup>When customers reach their 500MB daily data transfer limit, data analyzation stops and resumes at the start of the next day. A day is based on UTC.


### Backup Limits
<properties
   pageTitle="Azure Backup limits table"
   description="Describes system limits for Azure Backup."
   services="backup"
   documentationCenter="NA"
   authors="Jim-Parker"
   manager="jwhit"
   editor="" />
<tags
   ms.service="backup"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="10/05/2015"
   ms.author="trinadhk";"jimpark"; "aashishr" />


The following limits apply to Azure Backup.

| Limit Identifier | Default Limit |
|---|---|
|Number of servers/machines that can be registered against each vault|50 for Windows Server/Client/SCDPM <br/> 200 for IaaS VMs|
|Size of a data source for data stored in Azure vault storage|54400 GB max<sup>1</sup>|
|Number of backup vaults that can be created in each Azure subscription|25|
|Number of times backup can be scheduled per day|3 per day for Windows Server/Client <br/> 2 per day for SCDPM <br/> Once a day for IaaS VMs|
|Data disks attached to an Azure virtual machine for backup|16|

- <sup>1</sup>The 54400 GB limit does not apply to IaaS VM backup.



### Site Recovery Limits
<properties
   pageTitle="Site Recovery limits table"
   description="Describes system limits for Site Recovery."
   services="site recovery"
   documentationCenter="NA"
   authors="csilauraa"
   manager="jwhit"
   editor="" />
<tags
   ms.service="site recovery"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="07/06/2015"
   ms.author="lauraa" />


The following limits apply to Azure Site Recovery:


|LIMIT IDENTIFIER|DEFAULT LIMIT|
|---|---|---|---|
|Number of vaults per subscription|25|
|Number of servers per Azure vault|250|
|Number of protection groups per Azure vault|No limit|
|Number of recovery plans per Azure vault|No limit|
|Number of servers per protection group|No limit|
|Number of servers per recovery plan|50|


### Application Insights Limits
 Limits depend on the [pricing tier](https://azure.microsoft.com/pricing/details/application-insights/) that you choose.

**Resource** | **Default Limit** | **Maximum Limit**
-------- | ------------- | -------------
Session data points<sup>1</sup> per month | unlimited | unlimited
Other data points per month | 5 million | 50 million<sup>2</sup>
Trace or Log data rate | 200 dp/s | 500 dp/s
Exception data rate | 50 dp/s | 50 dp/s
Other telemetry data rate | 200 dp/s | 500 dp/s
Raw  data retention |7 days| 30 days
Aggregated data retention | 13 months | unlimited
Property name count across the app | 100 | 100
Property name length | 100 | 100
Property value length | 1000 | 1000
Trace and Exception message length | 10000 | 10000
Metric name length |  100 | 100

<sup>1</sup> A data point is an individual metric value or event, with attached properties and measurements.

<sup>2</sup> You can purchase additional capacity beyond 50 million.
 
[About pricing and quotas in Application Insights](app-insights-pricing.md)

### API Management Limits
| Resource                          | Limit                                    |
|-----------------------------------|------------------------------------------|
| API Calls (per unit of scale)     | 32 million per day<sup>1</sup>            |
| Data transfer (per unit of scale) | 161 GB per day<sup>1</sup> |
| Cache                             | 5 GB<sup>1</sup> |
| Units of scale                    | Unlimited<sup>1</sup> |
| Azure Active Directory Integration| Unlimited User Accounts<sup>1</sup> |

<sup>1</sup>API Management limits are different for each pricing tier. To see the pricing tiers and their associated limits and scaling options, see [API Management Pricing](https://azure.microsoft.com/pricing/details/api-management/).

### Azure Redis Cache Limits
| Resource                                    | Limit                                  |
|---------------------------------------------|----------------------------------------|
| Cache size                                  | 530 GB ([contact us](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase) for more)                                  |
| Databases                                   | 16                                     |
| Max connected clients                       | 40,000                                 |
| Redis Cache replicas (for high availability) | 1 |
| Shards in a premium cache with clustering    | 10 |

Azure Redis Cache limits and sizes are different for each pricing tier. To see the pricing tiers and their associated sizes, see [Azure Redis Cache Pricing](https://azure.microsoft.com/pricing/details/cache/).

For more information on Azure Redis Cache configuration limits, see [Default Redis server configuration](redis-cache/cache-configure.md#default-redis-server-configuration).

Because configuration and management of Azure Redis Cache instances is done by Microsoft, not all Redis commands are supported in Azure Redis Cache. For more information, see [Redis commands not supported in Azure Redis Cache]((redis-cache/cache-configure.md#redis-commands-not-supported-in-azure-redis-cache).

### Key Vault Limits

| Transactions Type	| Max transactions allowed in 10 seconds, per vault per region
--- | ---
| HSM- CREATE KEY | 5
| HSM- other transactions | 1000
| Soft-key CREATE KEY | 10
| Soft-key other transactions | 1500
| All secrets, vault related transactions | 2000
 
 


### Multi-Factor Authentication
Resource|Default Limit|Maximum Limit
---|---|---
Max number of Trusted IP addresses/ranges</a> per subscription<sup>1</sup>|0|12
Remember my devices - number of days|14|60
Max number of app passwords?|0|No Limit
Allow **X** attempts during MFA call|1|99
Two-way Text message Timeout Seconds|60|600
Default one-time bypass seconds|300|1800
Lock user account after **X** consecutive MFA denials|Not Set|99
Reset account lockout counter after **X** minutes|Not Set|9999
Unlock account after **X** minutes|Not Set|9999


<sup>1</sup>This is expected to increase in the future.


### SQL Database Limits
For SQL Database limits, see [SQL Database Resource Limits](sql-database/sql-database-resource-limits.md).

## See Also
[Understanding Azure Limits and Increases](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)

[Virtual Machine and Cloud Service Sizes for Azure](http://msdn.microsoft.com/library/azure/dn197896.aspx)

