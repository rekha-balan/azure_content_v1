<properties
   pageTitle="Azure Storage Scalability and Performance Targets | Microsoft Azure"
   description="Learn about the scalability and performance targets for Azure Storage, including capacity, request rate, and inbound and outbound bandwidth for both standard and premium storage accounts. Understand performance targets for partitions within each of the Azure Storage services."
   services="storage"
   documentationCenter="na"
   authors="robinsh"
   manager="carmonm"
   editor="tysonn" />

<tags
   ms.service="storage"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="storage"
   ms.date="12/04/2015"
   ms.author="robinsh" />

# Azure Storage Scalability and Performance Targets
## Overview
This topic describes the scalability and performance topics for Microsoft Azure Storage. For a summary of other Azure limits, see [Azure Subscription and Service Limits, Quotas, and Constraints](../azure-subscription-service-limits.md).

> [!NOTE]
> All storage accounts run on the new flat network topology and support the scalability and performance targets outlined below, regardless of when they were created. For more information on the Azure Storage flat network architecture and on scalability, see [Microsoft Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx).
> 
> 
<!-- -->

> [!IMPORTANT]
> The scalability and performance targets listed here are high-end targets, but are achievable. In all cases, the request rate and bandwidth achieved by your storage account depends upon the size of objects stored, the access patterns utilized, and the type of workload your application performs. Be sure to test your service to determine whether its performance meets your requirements. If possible, avoid sudden spikes in the rate of traffic and ensure that traffic is well-distributed across partitions.
> 
> When your application reaches the limit of what a partition can handle for your workload, Azure Storage will begin to return error code 503 (Server Busy) or error code 500 (Operation Timeout) responses. When this occurs, the application should use an exponential backoff policy for retries. The exponential backoff allows the load on the partition to decrease, and to ease out spikes in traffic to that partition.
> 
> 
If the needs of your application exceed the scalability targets of a single storage account, you can build your application to use multiple storage accounts, and partition your data objects across those storage accounts. See [Storage Pricing Details](https://azure.microsoft.com/pricing/details/storage/) for information on volume pricing.

## Scalability targets for blobs, queues, tables, and files
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



## Scalability targets for virtual machine disks
An Azure virtual machine supports attaching a number of data disks. For optimal performance, you will want to limit the number of highly utilized disks attached to the virtual machine to avoid possible throttling. If all disks are not being highly utilized at the same time, the storage account can support a larger number disks.

- **For standard storage accounts:** A standard storage account has a maximum total request rate of 20,000 IOPS. The total IOPS across all of your virtual machine disks in a standard storage account should not exceed this limit.

	You can roughly calculate the number of highly utilized disks supported by a single standard storage account based on the request rate limit. For example, for a Basic Tier VM, the maximum number of highly utilized disks is about 66 (20,000/300 IOPS per disk), and for a Standard Tier VM, it is about 40 (20,000/500 IOPS per disk), as shown in the table below. 
 
- **For premium storage accounts:** A premium storage account has a maximum total throughput rate of 50 Gbps. The total throughput across all of your VM disks should not exceed this limit.

See [Virtual machine sizes](../virtual-machines/virtual-machines-size-specs.md) for additional details.

### Standard storage accounts
**Virtual machine disks: per disk limits**

 VM Tier | Basic Tier VM | Standard Tier VM
---|---|---
Disk size | 1023 GB | 1023 GB
Max 8 KB IOPS per persistent disk | 300 | 500
Max number of highly utilized disks | 66 | 40



### Premium storage accounts
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




## Scalability targets for Azure resource manager
The following limits apply when using the Azure Resource Manager and Azure Resource Groups only.

Resource|Default Limit
---|---
Storage account management operations (read)|800 per 5 minutes
Storage account management operations (write)|200 per hour
Storage account management operations (list)|100 per 5 minutes


## Partitions in Azure Storage
Every object that holds data that is stored in Azure Storage (blobs, messages, entities, and files) belongs to a partition, and is identified by a partition key. The partition determines how Azure Storage load balances blobs, messages, entities, and files across servers to meet the traffic needs of those objects. The partition key is unique within the storage account and is used to locate a blob, message, or entity.

The table shown above in [Scalability Targets for Standard Storage Accounts](#scalability-targets-for-standard-storage-accounts.md) lists the performance targets for a single partition for each service.

Partitions affect load balancing and scalability for each of the storage services in the following ways:

* **Blobs**: The partition key for a blob is container name + blob name. This means that each blob has its own partition. Blobs can therefore be distributed across many servers in order to scale out access to them. While blobs can be logically grouped in blob containers, there are no partitioning implications from this grouping.

* **Files**: The partition key for a file is account name + file share name. This means all files in a file share are also in a single partition.

* **Messages**: The partition key for a message is the queue name, so all messages in a queue are grouped into a single partition and are served by a single server. Different queues may be processed by different servers to balance the load for however many queues a storage account may have.

* **Entities**: The partition key for an entity is table name + partition key, where the partition key is the value of the required user-defined **PartitionKey** property for the entity.  

    All entities with the same partition key value are grouped into the same partition and are stored on the same partition server. This is an important point to understand in designing your application. Your application should balance the scalability benefits of spreading entities across multiple partitions with the data access advantages of grouping entities in a single partition.

    A key advantage to grouping a set of entities in a table into a single partition is that it's possible to perform atomic batch operations across entities in the same partition, since a partition exists on a single server. Therefore if you wish to perform batch operations, consider grouping entities with the same partition key.

    On the other hand, entities that are in the same table but that belong to different partitions can be load balanced across different servers, making it possible to have a large table with greater scalability.


## See Also
* [Storage Pricing Details](https://azure.microsoft.com/pricing/details/storage/)
* [Azure Subscription and Service Limits, Quotas, and Constraints](../azure-subscription-service-limits.md)
* [Premium Storage: High-Performance Storage for Azure Virtual Machine Workloads](storage-premium-storage-preview-portal/.md)
* [Azure Storage Replication](storage-redundancy.md)
* [Microsoft Azure Storage Performance and Scalability Checklist](storage-performance-checklist.md)
* [Microsoft Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

