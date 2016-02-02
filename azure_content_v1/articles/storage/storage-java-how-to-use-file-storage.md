<properties
    pageTitle="How to use File storage from Java | Microsoft Azure"
    description="Learn how to use the Azure file service to upload, download, list, and delete files. Samples written in Java."
    services="storage"
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor="jimbe" />

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="Java"
    ms.topic="article"
    ms.date="01/11/2016"
    ms.author="jutang"/>

# How to use File Storage from Java
> [AZURE.SELECTOR]
- [.NET](../articles/storage/storage-dotnet-how-to-use-files.md)
- [Java](../articles/storage/storage-java-how-to-use-file-storage.md)


## Overview
In this guide you'll learn how to perform basic operations on the  Microsoft Azure File storage service. Through samples written in Java you'll learn how to  create shares and directories, upload, list, and delete files. If you are new to Microsoft Azure's File Storage service, going through the concepts in the sections that follow will be very helpful in understanding the samples.

## What is Azure File storage?

File storage offers shared storage for applications using the standard SMB 2.1 or SMB 3.0 protocol. Microsoft Azure virtual machines and cloud services can share file data across application components via mounted shares, and on-premises applications can access file data in a share via the File storage API.

Applications running in Azure virtual machines or cloud services can mount a File storage share to access file data, just as a desktop application would mount a typical SMB share. Any number of Azure virtual machines or roles can mount and access the File storage share simultaneously.

Since a File storage share is a standard file share in Azure using the SMB protocol, applications running in Azure can access data in the share via file I/O APIs. Developers can therefore leverage their existing code and skills to migrate existing applications. IT Pros can use PowerShell cmdlets to create, mount, and manage File storage shares as part of the administration of Azure applications. This guide will show examples of both.

Common uses of File storage include:

- Migrating on-premises applications that rely on file shares to run on Azure virtual machines or cloud services, without expensive rewrites
- Storing shared application settings, for example in configuration files
- Storing diagnostic data such as logs, metrics, and crash dumps in a shared location 
- Storing tools and utilities needed for developing or administering Azure virtual machines or cloud services

## File storage concepts

File storage contains the following components:

![files-concepts][files-concepts]

-   **Storage Account:** All access to Azure Storage is done
    through a storage account. See [Azure Storage Scalability and Performance Targets](http://msdn.microsoft.com/library/azure/dn249410.aspx) for details about storage account capacity.

-   **Share:** A File storage share is an SMB file share in Azure. 
    All directories and files must be created in a parent share. An account can contain an
    unlimited number of shares, and a share can store an unlimited
    number of files, up to the 5 TB total capacity of the file share.

-   **Directory:** An optional hierarchy of directories. 

-	**File:** A file in the share. A file may be up to 1 TB in size.

-   **URL format:** Files are addressable using the following URL
    format:   
    https://`<storage
    account>`.file.core.windows.net/`<share>`/`<directory/directories>`/`<file>`  
    
    The following example URL could be used to address one of the files in the
    diagram above:  
    `http://samples.file.core.windows.net/logs/CustomLogs/Log1.txt`

For details about how to name shares, directories, and files, see [Naming and Referencing Shares, Directories, Files, and Metadata](http://msdn.microsoft.com/library/azure/dn167011.aspx).

[files-concepts]: ./media/storage-file-concepts-include/files-concepts.png

## Create an Azure storage account

The easiest way to create your first Azure storage account is by using the [Azure Management Portal](https://manage.windowsazure.com). To learn more, see [Create a storage account](../articles/storage/storage-create-storage-account.md#create-a-storage-account).

You can create an Azure storage account by using [Azure PowerShell](../articles/storage/storage-powershell-guide-full.md) or [Azure CLI](../articles/storage/storage-azure-cli.md), or by using the [Azure Storage Resource Provider REST API](https://msdn.microsoft.com/library/azure/mt163683.aspx).
 

## Create a Java application
To build the samples, you will need the Java Development Kit (JDK) and the [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java). You should also have created an Azure storage account.

## Setup your application to use File storage
To use the Azure storage APIs, add the following statement to the top of the Java file where you intend to access the storage service from.

    // Include the following imports to use blob APIs.
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.file.*;

## Setup an Azure storage connection string
To use File storage, you need to connect to your Azure storage account. The first step would be to configure a connection string which we'll use to connect to your storage account. Let's define a static variable to do that.

    // Configure the connection-string with your values
    public static final String storageConnectionString =
        "DefaultEndpointsProtocol=http;" +
        "AccountName=your_storage_account_name;" +
        "AccountKey=your_storage_account_key";

> [!NOTE]
> Replace your_storage_account_name and your_storage_account_key with the actual values for your storage account.
> 
> 
## Connecting to an Azure storage account
To connect to your storage account, you need to use the **CloudStorageAccount** object, passing a connection string to its **parse** method.

    // Use the CloudStorageAccount object to connect to your storage account
    try {
        CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
    } catch (InvalidKeyException invalidKey) {
        // Handle the exception
    }

**CloudStorageAccount.parse** throws an InvalidKeyException so you'll need to put it inside a try/catch block.

## How to: Create a Share
All files and directories in File storage reside in a container called a **Share**. Your storage account can have as much shares as your account capacity allows. To obtain access to a share and its contents, you need to use a File storage client.

    // Create the file storage client.
    CloudFileClient fileClient = storageAccount.createCloudFileClient();

Using the File storage client, you can then obtain a reference to a share.

    // Get a reference to the file share
    CloudFileShare share = fileClient.getShareReference("sampleshare");

To actually create the share, use the **createIfNotExists** method of the CloudFileShare object.

    if (share.createIfNotExists()) {
        System.out.println("New share created");
    }

At this point, **share** holds a reference to a share named **sampleshare**.

## How to: Upload a file
An Azure File Storage Share contains at the very least, a root directory where files can reside. In this section, you'll learn how to upload a file from local storage onto the root directory of a share.

The first step in uploading a file is to obtain a reference to the directory where it should reside. You do this by calling the **getRootDirectoryReference** method of the share object.

    //Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.getRootDirectoryReference();

Now that you have a reference to the root directory of the share, you can upload a file onto it using the following code.

    // Define the path to a local file.
    final String filePath = "C:\\temp\\Readme.txt";

    CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
    cloudFile.uploadFromFile(filePath);

## How to: Create a Directory
You can also organize storage by putting files inside sub-directories instead of having all of them in the root directory. The Azure file storage service allows you to create as much directories as your account will allow. The code below will create a sub-directory named **sampledir** under the root directory.

    //Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.getRootDirectoryReference();

    //Get a reference to the sampledir directory
    CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

    if (sampleDir.createIfNotExists()) {
        System.out.println("sampledir created");
    } else {
        System.out.println("sampledir already exists");
    }

## How to: List files and directories in a share
Obtaining a list of files and directories within a share is easily done by calling **listFilesAndDirectories** on a CloudFileDirectory reference. The method returns a list of ListFileItem objects which you can iterate on. As an example, the following code will list files and directories inside the root directory.

    //Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.getRootDirectoryReference();

    for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
        System.out.println(fileItem.getUri());
    }


## How to: Download a file
One of the more frequent operations you will perform against file storage is to download files. In the following example, the code downloads SampleFile.txt and displays its contents.

    //Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.getRootDirectoryReference();

    //Get a reference to the directory that contains the file
    CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

    //Get a reference to the file you want to download
    CloudFile file = sampleDir.getFileReference("SampleFile.txt");

    //Write the contents of the file to the console.
    System.out.println(file.downloadText());

## How to: Delete a file
Another common file storage operation is file deletion. The following code deletes a file named SampleFile.txt stored inside a directory named **sampledir**.

    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.getRootDirectoryReference();

    // Get a reference to the directory where the file to be deleted is in
    CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

    String filename = "SampleFile.txt"
    CloudFile file;

    file = containerDir.getFileReference(filename)
    if ( file.deleteIfExists() ) {
        System.out.println(filename + " was deleted");
    }


## How to: Delete a directory
Deleting a directory is a fairly simple task, although it should be noted that you cannot delete a directory that still contains files or other directories.

    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.getRootDirectoryReference();

    // Get a reference to the directory you want to delete
    CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

    // Delete the directory
    if ( containerDir.deleteIfExists() ) {
        System.out.println("Directory deleted");
    }


## How to: Delete a Share
Deleting a share is done by calling the **deleteIfExists** method on a CloudFileShare object. Here's sample code that does that.

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

        // Create the file client.
       CloudFileClient fileClient = storageAccount.createCloudFileClient();

       // Get a reference to the file share
       CloudFileShare share = fileClient.getShareReference("sampleshare");

       if (share.deleteIfExists()) {
           System.out.println("sampleshare deleted");
       }
    } catch (Exception e) {
        e.printStackTrace();
    }

## Next steps
If you would like to learn more about other Azure storage APIs, follow these links.

* [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)
* [Azure Storage Client SDK Reference](http://dl.windowsazure.com/storage/javadoc/)
* [Azure Storage REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)
* [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/)
* [Transfer data with the AzCopy command-line utility](storage-use-azcopy.md)

[Azure SDK for Java]: http://azure.microsoft.com/develop/java/
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Azure Storage Client SDK Reference]: http://dl.windowsazure.com/storage/javadoc/
[Azure Storage REST API]: https://msdn.microsoft.com/library/azure/dd179355.aspx
[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
