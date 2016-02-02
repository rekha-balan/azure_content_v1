<properties
    pageTitle="How to use Azure Blob storage from Java | Microsoft Azure"
    description="Learn how to use Azure Blob storage to upload, download, list, and delete blob content. Samples written in Java."
    services="storage"
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor="jimbe"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="Java"
    ms.topic="article"
    ms.date="12/01/2015"
    ms.author="micurd"/>

# How to use Blob storage from Java
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
This article will show you how to perform common scenarios using the Microsoft Azure Blob storage. The samples are written in Java and use the [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java). The scenarios covered include **uploading**, **listing**, **downloading**, and **deleting** blobs. For more information on blobs, see the [Next Steps](#NextSteps.md) section.

> [!NOTE]
> An SDK is available for developers who are using Azure Storage on Android devices. For more information, see the [Azure Storage SDK for Android](https://github.com/azure/azure-storage-android).
> 
> 
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
 

## Create a Java application
In this article, you will use storage features which can be run within a Java application locally, or in code running within a web role or worker role in Azure.

To do so, you will need to install the Java Development Kit (JDK) and create an Azure Storage account in your Azure subscription. Once you have done so, you will need to verify that your development system meets the minimum requirements and dependencies which are listed in the [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java) repository on GitHub. If your system meets those requirements, you can follow the instructions for downloading and installing the Azure Storage Libraries for Java on your system from that repository. Once you have completed those tasks, you will be able to create a Java application which uses the examples in this article.

## Configure your application to access Blob storage
Add the following import statements to the top of the Java file where you want to use the Azure Storage APIs to access blobs.

    // Include the following imports to use blob APIs.
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;

## Set up an Azure Storage connection string
An Azure Storage client uses a storage connection string to store
endpoints and credentials for accessing data management services. When running in a client application, you must provide the storage connection string in the following format, using the name of your storage account and the Primary access key for the storage account listed in the [Azure Portal](https://portal.azure.com) for the *AccountName* and *AccountKey* values. The following example shows how you can declare a static field to hold the connection string.

    // Define the connection-string with your values
    public static final String storageConnectionString =
        "DefaultEndpointsProtocol=http;" +
        "AccountName=your_storage_account;" +
        "AccountKey=your_storage_account_key";

In an application running within a role in Microsoft Azure, this string can be stored in the service configuration file, *ServiceConfiguration.cscfg*, and can be accessed with a call to the **RoleEnvironment.getConfigurationSettings** method. The followng example gets the connection string from a **Setting** element named *StorageConnectionString* in the service configuration file.

    // Retrieve storage account from connection-string.
    String storageConnectionString =
        RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

The following samples assume that you have used one of these two methods to get the storage connection string.

## Create a container
A **CloudBlobClient** object lets you get reference objects for containers and blobs. The following code creates a **CloudBlobClient** object.

> [!NOTE]
> There are additional ways to create **CloudStorageAccount** objects; for more information, see **CloudStorageAccount** in the [Azure Storage Client SDK Reference](http://dl.windowsazure.com/storage/javadoc/).
> 
> 
Every blob in Azure storage must reside in a container. The container forms part of the blob name. For example, `mycontainer` is the name of the container in these sample blob URIs:

	https://storagesample.blob.core.windows.net/mycontainer/blob1.txt
	https://storagesample.blob.core.windows.net/mycontainer/photos/myphoto.jpg

A container name must be a valid DNS name, conforming to the following naming rules:

1. Container names must start with a letter or number, and can contain only letters, numbers, and the dash (-) character.
1. Every dash (-) character must be immediately preceded and followed by a letter or number; consecutive dashes are not permitted in container names.
1. All letters in a container name must be lowercase.
1. Container names must be from 3 through 63 characters long.

> [AZURE.IMPORTANT] Note that the name of a container must always be lowercase. If you include an upper-case letter in a container name, or otherwise violate the container naming rules, you may receive a 400 error (Bad Request). 

Use the **CloudBlobClient** object to get a reference to the container you want to use. You can create the container if it doesn't exist with the **createIfNotExists** method, which will otherwise return the existing container. By default, the new container is private, so you must specify your storage access key (as you did earlier) to download blobs from this container.

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

        // Create the blob client.
       CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

       // Get a reference to a container.
       // The container name must be lower case
       CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

       // Create the container if it does not exist.
        container.createIfNotExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

### Optional: Configure a container for public access
A container's permissions are configured for private access by default, but you can easily configure a container's permissions to allow public, read-only access for all users on the Internet:

    // Create a permissions object.
    BlobContainerPermissions containerPermissions = new BlobContainerPermissions();

    // Include public access in the permissions object.
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);

    // Set the permissions on the container.
    container.uploadPermissions(containerPermissions);

## Upload a blob into a container
To upload a file to a blob, get a container reference and use it to get a blob reference. Once you have a blob reference, you can upload any stream by calling upload on the blob reference. This operation will create the blob if it doesn't exist, or overwrite it if it does. The following code sample shows this, and assumes that the container has already been created.

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

        // Create the blob client.
        CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

       // Retrieve reference to a previously created container.
        CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

        // Define the path to a local file.
        final String filePath = "C:\\myimages\\myimage.jpg";

        // Create or overwrite the "myimage.jpg" blob with contents from a local file.
        CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");
        File source = new File(filePath);
        blob.upload(new FileInputStream(source), source.length());
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## List the blobs in a container
To list the blobs in a container, first get a container reference like you did to upload a blob. You can use the container's **listBlobs** method with a **for** loop. The following code outputs the Uri of each blob in a container to the console.

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

        // Create the blob client.
        CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

        // Retrieve reference to a previously created container.
        CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

        // Loop over blobs within the container and output the URI to each of them.
        for (ListBlobItem blobItem : container.listBlobs()) {
           System.out.println(blobItem.getUri());
       }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

Note that you can name blobs with path information in their names. This creates a virtual directory structure that you can organize and traverse as you would a traditional file system. Note that the directory structure is virtual only - the only resources available in Blob storage are containers and blobs. However, the client library offers a **CloudBlobDirectory** object to refer to a virtual directory and simplify the process of working with blobs that are organized in this way.

For example, you could have a container named "photos", in which you might upload blobs named "rootphoto1", "2010/photo1", "2010/photo2", and "2011/photo1". This would create the virtual directories "2010" and "2011" within the "photos" container. When you call **listBlobs** on the "photos" container, the collection returned will contain **CloudBlobDirectory** and **CloudBlob** objects representing the directories and blobs contained at the top level. In this case, directories "2010" and "2011", as well as photo "rootphoto1" would be returned. You can use the **instanceof** operator to distinguish these objects.

Optionally, you can pass in parameters to the **listBlobs** method with
the **useFlatBlobListing** parameter set to true. This will result in
every blob being returned, regardless of directory. For more
information, see **CloudBlobContainer.listBlobs** in the [Azure Storage Client SDK Reference](http://dl.windowsazure.com/storage/javadoc/).

## Download a blob
To download blobs, follow the same steps as you did for uploading a blob in order to get a blob reference. In the uploading example, you called upload on the blob object. In the following example, call download to transfer the blob contents to a stream object such as a **FileOutputStream** that you can use to persist the blob to a local file.

    try
    {
        // Retrieve storage account from connection-string.
       CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

       // Create the blob client.
       CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

       // Retrieve reference to a previously created container.
       CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

       // Loop through each blob item in the container.
       for (ListBlobItem blobItem : container.listBlobs()) {
           // If the item is a blob, not a virtual directory.
           if (blobItem instanceof CloudBlob) {
               // Download the item and save it to a file with the same name.
                CloudBlob blob = (CloudBlob) blobItem;
                blob.download(new FileOutputStream("C:\\mydownloads\\" + blob.getName()));
            }
        }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## Delete a blob
To delete a blob, get a blob reference, and call **deleteIfExists**.

    try
    {
       // Retrieve storage account from connection-string.
       CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

       // Create the blob client.
       CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

       // Retrieve reference to a previously created container.
       CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

       // Retrieve reference to a blob named "myimage.jpg".
       CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");

       // Delete the blob.
       blob.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## Delete a blob container
Finally, to delete a blob container, get a blob container reference, and
call **deleteIfExists**.

    try
    {
       // Retrieve storage account from connection-string.
       CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

       // Create the blob client.
       CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

       // Retrieve reference to a previously created container.
       CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

       // Delete the blob container.
       container.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## Next steps
Now that you've learned the basics of Blob storage, follow these links to learn about more complex storage tasks.

* [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)
* [Azure Storage Client SDK Reference](http://dl.windowsazure.com/storage/javadoc/)
* [Azure Storage REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)
* [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/)

For more information, see also the [Java Developer Center](/develop/java/).

[Azure SDK for Java]: http://go.microsoft.com/fwlink/?LinkID=525671
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Azure Storage Client SDK Reference]: http://dl.windowsazure.com/storage/javadoc/
[Azure Storage REST API]: https://msdn.microsoft.com/library/azure/dd179355.aspx
[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
