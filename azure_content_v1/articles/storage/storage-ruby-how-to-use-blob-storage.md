<properties
    pageTitle="How to use Blob storage from Ruby | Microsoft Azure"
    description="Learn how to use Blob storage to upload, download, list, and delete blob content. Samples written in Ruby."
    services="storage"
    documentationCenter="ruby"
    authors="tfitzmac"
    manager="wpickett"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="tomfitz"/>


# How to use Blob storage from Ruby
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
This guide will show you how to perform common scenarios using Blob storage. The samples are written using the Ruby API. The scenarios covered include **uploading, listing, downloading,** and **deleting** blobs.

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
 

## Create a Ruby application
Create a Ruby application. For instructions,
see [Create a Ruby application on Azure](/develop/ruby/tutorials/web-app-with-linux-vm/).

## Configure your application to access Storage
To use Azure Storage, you need to download and use the Ruby azure package, which includes a set of convenience libraries that communicate with the storage REST services.

### Use RubyGems to obtain the package
1. Use a command-line interface such as **PowerShell** (Windows), **Terminal** (Mac), or **Bash** (Unix).

2. Type "gem install azure" in the command window to install the gem and dependencies.


### Import the package
Using your favorite text editor, add the following to the top of the Ruby file where you intend to use storage:

    require "azure"

## Setup an Azure Storage Connection
The azure module will read the environment variables **AZURE\_STORAGE\_ACCOUNT** and **AZURE\_STORAGE\_ACCESS_KEY**
for information required to connect to your Azure storage account. If these environment variables are not set, you must specify the account information before using **Azure::Blob::BlobService** with the following code:

    Azure.config.storage_account_name = "<your azure storage account>"
    Azure.config.storage_access_key = "<your azure storage access key>"


To obtain these values:

1. Log in to the [Azure Portal](https://portal.azure.com).
2. Navigate to the storage account you want to use.
3. Click **MANAGE KEYS** at the bottom of the navigation pane.
4. In the pop up dialog, you'll see the storage account name, primary access key and secondary access key. For access key, you can use either the primary one or the secondary one.

## Create a container
Every blob in Azure storage must reside in a container. The container forms part of the blob name. For example, `mycontainer` is the name of the container in these sample blob URIs:

	https://storagesample.blob.core.windows.net/mycontainer/blob1.txt
	https://storagesample.blob.core.windows.net/mycontainer/photos/myphoto.jpg

A container name must be a valid DNS name, conforming to the following naming rules:

1. Container names must start with a letter or number, and can contain only letters, numbers, and the dash (-) character.
1. Every dash (-) character must be immediately preceded and followed by a letter or number; consecutive dashes are not permitted in container names.
1. All letters in a container name must be lowercase.
1. Container names must be from 3 through 63 characters long.

> [AZURE.IMPORTANT] Note that the name of a container must always be lowercase. If you include an upper-case letter in a container name, or otherwise violate the container naming rules, you may receive a 400 error (Bad Request). 

The **Azure::Blob::BlobService** object lets you work with containers and blobs. To create a container, use the **create\_container()** method.

The following code example creates a container or print out the error if there is any.

    azure_blob_service = Azure::Blob::BlobService.new
    begin
      container = azure_blob_service.create_container("test-container")
    rescue
      puts $!
    end

If you want to make the files in the container public, you can set the container's permissions.

You can just modify the <strong>create\_container()</strong> call to pass the **:public\_access\_level** option:

    container = azure_blob_service.create_container("test-container",
      :public_access_level => "<public access level>")


Valid values for the **:public\_access\_level** option are:

* **blob:** Specifies full public read access for container and blob data. Clients can enumerate blobs within the container via anonymous request, but cannot enumerate containers within the storage account.

* **container:** Specifies public read access for blobs. Blob data within this container can be read via anonymous request, but container data is not available. Clients cannot enumerate blobs within the container via anonymous request.


Alternatively, you can modify the public access level of a container by using **set\_container\_acl()** method to specify the public access level.

The following code example changes the public access level to **container**:

    azure_blob_service.set_container_acl('test-container', "container")

## Upload a blob into a container
To upload content to a blob, use the **create\_block\_blob()** method to create the blob, use a file or string as the content of the blob.

The following code uploads the file **test.png** as a new blob named "image-blob" in the container.

    content = File.open("test.png", "rb") { |file| file.read }
    blob = azure_blob_service.create_block_blob(container.name,
      "image-blob", content)
    puts blob.name

## List the blobs in a container
To list the containers, use **list_containers()** method.
To list the blobs within a container, use **list\_blobs()** method.

This outputs the urls of all the blobs in all the containers for the account.

    containers = azure_blob_service.list_containers()
    containers.each do |container|
      blobs = azure_blob_service.list_blobs(container.name)
      blobs.each do |blob|
        puts blob.name
      end
    end

## Download blobs
To download blobs, use the **get\_blob()** method to retrieve the contents.

The following code example demonstrates using **get\_blob()** to download the contents of "image-blob" and write it to a local file.

    blob, content = azure_blob_service.get_blob(container.name,"image-blob")
    File.open("download.png","wb") {|f| f.write(content)}

## Delete a Blob
Finally, to delete a blob, use the **delete\_blob()** method. The following code example demonstrates how to delete a blob.

    azure_blob_service.delete_blob(container.name, "image-blob")

## Next steps
To learn about more complex storage tasks, follow these links:

* [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/)
* [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) repository on GitHub
* [Transfer data with the AzCopy command-line utility](storage-use-azcopy.md)

