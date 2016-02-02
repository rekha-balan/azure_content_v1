<properties
    pageTitle="How to use Azure Blob storage from Python | Microsoft Azure"
    description="Learn how to use the Azure Blob storage from Python to upload, list, download, and delete blobs."
    services="storage"
    documentationCenter="python"
    authors="emgerner-msft"
    manager="wpickett"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="12/11/2015"
    ms.author="emgerner"/>

# How to use Azure Blob storage from Python
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-blobs.md)
- [Node.js](../articles/storage/storage-nodejs-how-to-use-blob-storage.md)
- [Java](../articles/storage/storage-java-how-to-use-blob-storage.md)
- [C++](../articles/storage/storage-c-plus-plus-how-to-use-blobs.md)
- [PHP](../articles/storage/storage-php-how-to-use-blobs.md)
- [Ruby](../articles/storage/storage-ruby-how-to-use-blob-storage.md)
- [Python](../articles/storage/storage-python-how-to-use-blob-storage.md)
- [iOS](../articles/storage/storage-ios-how-to-use-blob-storage.md)
- [Xamarin](../articles/storage/storage-xamarin-blob-storage.md)

## Overview
This article will show you how to perform common scenarios using Blob storage. The samples are written in Python and use the [Python Azure Storage package](https://pypi.python.org/pypi/azure-storage). The scenarios covered include uploading, listing,
downloading, and deleting blobs.

## What is Blob Storage

Azure Blob storage is a service for storing large amounts of
unstructured data, such as text or binary data, that can be accessed from anywhere in the world via
HTTP or HTTPS. You can use Blob storage to expose data publicly to the world, or
to store application data privately.

Common uses of Blob storage include:

-   Serving images or documents directly to a browser
-   Storing files for distributed access
-   Streaming video and audio
-   Performing secure backup and disaster recovery
-   Storing data for analysis by an on-premises or Azure-hosted service

## Blob service concepts

The Blob service contains the following components:

![Blob1][Blob1]

-   **Storage Account:** All access to Azure Storage is done
    through a storage account. See [Azure Storage Scalability and Performance Targets](storage-scalability-targets.md) for details about storage account capacity.

-   **Container:** A container provides a grouping of a set of blobs.
    All blobs must be in a container. An account can contain an
    unlimited number of containers. A container can store an unlimited
    number of blobs.

-   **Blob:** A file of any type and size. Azure Storage offers three types of blobs: block blobs, page blobs, and append blobs.
    
	*Block blobs* are ideal for storing text or binary files, such as documents and media files. *Append blobs* are similar to block blobs in that they are made up of blocks, but they are optimized for append operations, so they are useful for logging scenarios. A single block blob or append blob can contain up to 50,000 blocks of up to 4 MB each, for a total size of slightly more than 195 GB (4 MB X 50,000).
    
	*Page blobs* can be up to 1 TB in size, and are more efficient for frequent read/write operations. Azure Virtual Machines use page blobs as OS and data disks.

	For more information about blobs, see [Understanding Block Blobs, Page Blobs, and Append Blobs](https://msdn.microsoft.com/library/azure/ee691964.aspx).

## Naming and referencing containers and blobs

You can address a blob in your storage account using the following URL format:
   
    http://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>  
      
For example, here is a URL that addresses one of the blobs in the diagram above:  

    http://sally.blob.core.windows.net/movies/MOV1.AVI

### Container naming rules

A container name must be a valid DNS name and conform to the following rules:

- A container name must be all lowercase.
- Container names must start with a letter or number, and can contain only letters, numbers, and the dash (-) character.
- Every dash (-) character must be immediately preceded and followed by a letter or number; consecutive dashes are not permitted in container names.
- Container names must be from 3 through 63 characters long.

### Blob naming rules

A blob name must conform to the following rules:

- A blob name can contain any combination of characters.
- A blob name must be at least one character long and cannot be more than 1,024 characters long.
- Blob names are case-sensitive.
- Reserved URL characters must be properly escaped.
- The number of path segments comprising the blob name cannot exceed 254. A path segment is the string between consecutive delimiter characters (*e.g.*, the forward slash '/') that corresponds to the name of a virtual directory.

The Blob service is based on a flat storage scheme. You can create a virtual hierarchy by specifying a character or string delimiter within the blob name to create a virtual hierarchy. For example, the following list shows some valid and unique blob names:

	/a
	/a.txt
	/a/b
	/a/b.txt

You can use the delimiter character to list blobs hierarchically.


[Blob1]: ./media/storage-blob-concepts-include/blob1.jpg



## Create an Azure storage account

The easiest way to create your first Azure storage account is by using the [Azure Management Portal](https://manage.windowsazure.com). To learn more, see [Create a storage account](../articles/storage/storage-create-storage-account.md#create-a-storage-account).

You can create an Azure storage account by using [Azure PowerShell](../articles/storage/storage-powershell-guide-full.md) or [Azure CLI](../articles/storage/storage-azure-cli.md), or by using the [Azure Storage Resource Provider REST API](https://msdn.microsoft.com/library/azure/mt163683.aspx).
 

## Create a container
> [!NOTE]
> If you need to install Python or the [Python Azure package](https://pypi.python.org/pypi/azure), see the [Python Installation Guide](../python-how-to-install.md).
> 
> 
The **BlobService** object lets you work with containers and blobs. The
following code creates a **BlobService** object. Add the following near
the top of any Python file in which you wish to programmatically access Azure Storage.

    from azure.storage.blob import BlobService

The following code creates a **BlobService** object using the storage account name and account key.  Replace 'myaccount' and 'mykey' with the real account and key.

    blob_service = BlobService(account_name='myaccount', account_key='mykey')

Every blob in Azure storage must reside in a container. The container forms part of the blob name. For example, `mycontainer` is the name of the container in these sample blob URIs:

	https://storagesample.blob.core.windows.net/mycontainer/blob1.txt
	https://storagesample.blob.core.windows.net/mycontainer/photos/myphoto.jpg

A container name must be a valid DNS name, conforming to the following naming rules:

1. Container names must start with a letter or number, and can contain only letters, numbers, and the dash (-) character.
1. Every dash (-) character must be immediately preceded and followed by a letter or number; consecutive dashes are not permitted in container names.
1. All letters in a container name must be lowercase.
1. Container names must be from 3 through 63 characters long.

> [AZURE.IMPORTANT] Note that the name of a container must always be lowercase. If you include an upper-case letter in a container name, or otherwise violate the container naming rules, you may receive a 400 error (Bad Request). 

In the following code example, you can use a **BlobService** object to create the container if it doesn't exist.

    blob_service.create_container('mycontainer')

By default, the new container is private, so you must specify your storage access key (as you did earlier) to download blobs from this container. If you want to make the files within the container available to everyone, you can create the container and pass the public access level using the following code.

    blob_service.create_container('mycontainer', x_ms_blob_public_access='container')

Alternatively, you can modify a container after you have created it using the following code.

    blob_service.set_container_acl('mycontainer', x_ms_blob_public_access='container')

After this change, anyone on the Internet can see blobs in a public
container, but only you can modify or delete them.

## Upload a blob into a container
To upload data to a blob, use the **put\_block\_blob\_from\_path**, **put\_block\_blob\_from\_file**, **put\_block\_blob\_from\_bytes** or **put\_block\_blob\_from\_text** methods. They are high-level methods that perform the necessary chunking when the size of the data exceeds 64 MB.

**put\_block\_blob\_from\_path** uploads the contents of a file from the specified path, and **put\_block\_blob\_from\_file** uploads the contents from an already opened file/stream. **put\_block\_blob\_from\_bytes** uploads an array of bytes, and **put\_block\_blob\_from\_text** uploads the specified text value using the specified encoding (defaults to UTF-8).

The following example uploads the contents of the **sunset.png** file into the **myblob** blob.

    blob_service.put_block_blob_from_path(
        'mycontainer',
        'myblob',
        'sunset.png',
        x_ms_blob_content_type='image/png'
    )

## List the blobs in a container
To list the blobs in a container, use the **list\_blobs** method. Each
call to **list\_blobs** will return a segment of results. To get all results,
check the **next\_marker** of the results and call **list\_blobs** again as
needed. The following code outputs the **name** of each blob in a container
to the console.

    blobs = []
    marker = None
    while True:
        batch = blob_service.list_blobs('mycontainer', marker=marker)
        blobs.extend(batch)
        if not batch.next_marker:
            break
        marker = batch.next_marker
    for blob in blobs:
        print(blob.name)

## Download blobs
Each segment of results can contain a variable number of blobs up to a maximum
of 5000. If **next\_marker** exists for a particular segment, there may be
more blobs in the container.

To download data from a blob, use **get\_blob\_to\_path**, **get\_blob\_to\_file**, **get\_blob\_to\_bytes**, or **get\_blob\_to\_text**. They are high-level methods that perform the necessary chunking when the size of the data exceeds 64 MB.

The following example demonstrates using **get\_blob\_to\_path** to download the contents of the **myblob** blob and store it to the **out-sunset.png** file.

    blob_service.get_blob_to_path('mycontainer', 'myblob', 'out-sunset.png')

## Delete a blob
Finally, to delete a blob, call **delete_blob**.

    blob_service.delete_blob('mycontainer', 'myblob')

## Next steps
Now that you have learned the basics of Blob storage, follow these links
to learn about more complex storage tasks.

* Visit the [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/)
* [Transfer data with the AzCopy command-line utility](storage-use-azcopy.md)

For more information, see also the [Python Developer Center](/develop/python/).

[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure package]: https://pypi.python.org/pypi/azure
[Python Azure Storage package]: https://pypi.python.org/pypi/azure-storage  
