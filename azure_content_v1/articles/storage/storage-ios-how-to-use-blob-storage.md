<properties
    pageTitle="How to use Azure Blob storage from iOS | Microsoft Azure"
    description="Learn how to use Azure Blob storage to upload, download, list, and delete blob content. Samples written in Objective-C."
    services="storage"
    documentationCenter="ios"
    authors="micurd"
    manager="jahogg"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="Objective-C"
    ms.topic="article"
    ms.date="01/05/2016"
    ms.author="micurd"/>

# How to use Blob storage from iOS
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
This article will show you how to perform common scenarios using Microsoft Azure Blob storage. The samples are written in Objective-C and use the [Azure Storage iOS Library](https://github.com/Azure/azure-storage-ios). The scenarios covered include **uploading**, **listing**, **downloading**, and **deleting** blobs. For more information on blobs, see the [Next Steps](#next-steps.md) section. You can also download the [sample app](https://github.com/Azure/azure-storage-ios/tree/master/BlobSample) to quickly see the use of Azure Storage in an iOS application.

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
 

## Build the framework file
In order to use the Azure Storage iOS library, you will first need to build the framework file.

1. First, download or clone the [azure-storage-ios repo](https://github.com/azure/azure-storage-ios).

2. Go into *azure-storage-ios* -> *Lib* -> *Azure Storage Client Library*, and open `Azure Storage Client Library.xcodeproj` in Xcode.

3. At the top-left of Xcode, change the active scheme from "Azure Storage Client Library" to "Framework".

4. Build the project (⌘+B). This will create a `Azure Storage Client Library.framework` file on your Desktop.


## Import the Azure Storage iOS library into your application
You can import the Azure Storage iOS library into your application by doing the following:

1. Create a new project or open up your existing project in Xcode.

2. Click on your project in the left-hand navigation and click the *General* tab at the top of the project editor.

3. Under the *Linked Frameworks and Libraries* section, click the Add button (+).

4. Click *Add Other...*. Navigate to and add the `Azure Storage Client Library.framework` file you just created.

5. Under the *Linked Frameworks and Libraries* section, click the Add button (+) again.

6. In the list of libraries already provided, search for `libxml2.2.dylib` and add it to your project.


## Import Statement
You will need to include the following import statement in the file where you want to invoke the Azure Storage API.

    // Include the following import statement to use blob APIs.
    #import <Azure Storage Client Library/Azure_Storage_Client_Library.h>

## Configure your application to access Blob storage
There are two ways to authenticate your application to access Storage services:

* Shared Key: Use Shared Key for testing purposes only
* Shared Access Signature (SAS): Use SAS for production applications

### Shared Key
Shared Key authentication means that your application will use your account name and account key to access Storage services. For the purposes of quickly showing how to use Blob Storage from iOS, we will be using Shared Key authentication in this getting started.

> [AZURE.WARNING (Only use Shared Key authentication for testing purposes!) ]AZURE.WARNING (Only use Shared Key authentication for testing purposes!) ] Your account name and account key, which give full read/write access to the associated Storage account, will be distributed to every person that downloads your app. This is **not** good practice as you risk having your key compromised by untrusted clients.
> 
> 
When using Shared Key authentication, you will create a connection string. The connection string is comprised of:  

* The **DefaultEndpointsProtocol** - you can choose HTTP or HTTPS. However, using HTTPS is highly recommended.
* The **Account Name** - the name of your storage account
* The **Account Key** - If you're using the [Azure Portal](https://portal.azure.com), navigate to your storage account and click the **Keys** icon to find this information. If using the [Azure Classic Portal](https://manage.windowsazure.com), navigate to your storage account in the portal and click **Manage Access Keys**. 

Here is how it will look in your application:

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=account_name;AccountKey=account_key"];

### Shared Access Signatures (SAS)
For an iOS application, the recommended method for authenticating a request by a client against Blob storage is by using a Shared Access Signature (SAS). SAS allows you to grant a client access to a resource for a specified period of time, with a specified set of permissions.
As the storage account owner, you'll need to generate a SAS for your iOS clients to consume. To generate the SAS, you'll probably want to write a separate service that generates the SAS to be distributed to your clients. For testing purposes, you can also use the Azure CLI to generate a SAS. Note that your shared key credentials are used to generate a SAS, but your clients can then consume the SAS using the authentication information that is encapsulated in the SAS URL.
When you create the SAS, you can specify the time interval over which the SAS is valid, and the permissions that the SAS grants to the client. For example, for a blob container, the SAS can grant read, write, or delete permissions for a blob in the container, and list permissions to list the blobs in the container..

The following example shows how to use Azure CLI to generate a SAS token that grants read and write permissions for the container,*sascontainer*, until 12:00AM (UTC) September 5, 2015.  

1. First, follow this [guide](../xplat-cli/#how-to-install-the-azure-cli.md)  to learn how to install Azure CLI and connect to your Azure subscription.

2. Next, type the following command in Azure CLI to get the connection string for your account:

        azure storage account connectionstring show youraccountname
3. Create an environment variable with the connection string that you just generated:

        export AZURE_STORAGE_CONNECTION_STRING=”your connection string”
4. Generate SAS URL:

        azure storage container sas create --container sascontainer --permissions rw --expiry 2015-09-05T00:00:00
5. The SAS URL should look similar to the following:

        https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt?sv=2012-02-12&st=2013-04-29T22%3A18%3A26Z&se=2013-04-30T02%3A23%3A26Z&sr=b&sp=rw&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
6. In your iOS application, you can now get a reference to your container by using the SAS URL in the following manner:

        // Get a reference to a container in your Storage account
     AZSCloudBlobContainer *blobContainer = [[AZSCloudBlobContainer alloc] initWithUrl:[NSURL URLWithString:@" your SAS URL"]];


As you can see, when using a SAS token, you’re not exposing your account name and account key in your iOS application. You can learn more about SAS by checking out the [Shared Access Signature tutorial](../storage-dotnet-shared-access-signature-part-1.md).

## Asynchronous Operations
> [!NOTE]
> All methods that perform a request against the service are asynchronous operations. In the code samples, you’ll find that these methods have a completion handler. Code inside the completion handler will run **after** the request is completed. Code after the completion handler will run **while** the request is being made.
> 
> 
## Create a container
Every blob in Azure Storage must reside in a container. The following example shows how to create a container, called *newcontainer*, in your Storage account if it doesn't already exist. When choosing a name for your container, be mindful of the naming rules mentioned above.

     -(void)createContainer{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"newcontainer"];

        // Create container in your Storage account if the container doesn't already exist
        [blobContainer createContainerIfNotExistsWithCompletionHandler:^(NSError *error, BOOL exists) {
            if (error){
                NSLog(@"Error in creating container.");
            }
        }];
    }

You can confirm that this works by looking at the [Azure Portal](https://portal.azure.com) or any [Storage explorer](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx) and verifying that *newcontainer* is in the list of containers for your Storage account.

## Set Container Permissions
A container's permissions are configured for **Private** access by default. However, containers provide a few different options for container access:

* **Private**: Container and blob data can be read by the account owner only.

* **Blob**: Blob data within this container can be read via anonymous request, but container data is not available. Clients cannot enumerate blobs within the container via anonymous request.

* **Container**: Container and blob data can be read via anonymous request. Clients can enumerate blobs within the container via anonymous request, but cannot enumerate containers within the storage account.


The following example shows you how to create a container with **Container** access permissions which will allow public, read-only access for all users on the Internet:

     -(void)createContainerWithPublicAccess{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create container in your Storage account if the container doesn't already exist
        [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists){
            if (error){
                NSLog(@"Error in creating container.");
            }
        }];
    }

## Upload a blob into a container
As mentioned in the [Blob service concepts](#blob-service-concepts.md) section, Blob Storage offers three different types of blobs: block blobs, append blobs, and page blobs. At this moment, the Azure Storage iOS library only supports block blobs. In the majority of cases, block blob is the recommended type to use.

The following example shows how to upload a block blob from an NSString. If a blob with the same name already exists in this container, the contents of this blob will be overwritten.

     -(void)uploadBlobToContainer{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists)
         {
             if (error){
                 NSLog(@"Error in creating container.");
             }
              else{
                 // Create a local blob object
                  AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

                  // Upload blob to Storage
                  [blockBlob uploadFromText:@"This text will be uploaded to Blob Storage." completionHandler:^(NSError *error) {
                     if (error){
                        NSLog(@"Error in creating blob.");
                     }             
                  }];
             }
         }];
     }

You can confirm that this works by looking at the [Azure Portal](https://portal.azure.com) or any [Storage explorer](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx) and verifying that the container, *containerpublic*, contains the blob, *sampleblob*. In this sample, we used a public container so you can also verify that this worked by going to the blobs URI:

    https://nameofyourstorageaccount.blob.core.windows.net/containerpublic/sampleblob

In addition to uploading a block blob from an NSString, similar methods exist for NSData, NSInputStream or a local file.

## List the blobs in a container
The following example shows how to list all blobs in a container. When performing this operation, be mindful of the following parameters:     

* **continuationToken** - The continuation token represents where the listing operation should start. If no token is provided, it will list blobs from the beginning. Any number of blobs can be listed, from zero up to a set maximum. Even if this method returns zero results, if `results.continuationToken` is not nil, there may be more blobs on the service that have not been listed.
* **prefix** - You can specify the prefix to use for blob listing. Only blobs that begin with this prefix will be listed.
* **useFlatBlobListing** - As mentioned in the [Naming and referencing containers and blobs](#naming-and-referencing-containers-and-blobs.md) section, although the Blob service is a flat storage scheme, you can create a virtual hierarchy by naming blobs with path information. However, non-flat listing is currently not supported; this is coming soon. For now, this value should be `YES`
* **blobListingDetails** - You can specify which items to include when listing blobs  * `AZSBlobListingDetailsNone`: List only committed blobs, and do not return blob metadata.
* `AZSBlobListingDetailsSnapshots`: List committed blobs and blob snapshots.
* `AZSBlobListingDetailsMetadata`: Retrieve blob metadata for each blob returned in the listing.
* `AZSBlobListingDetailsUncommittedBlobs`: List committed and uncommitted blobs.
* `AZSBlobListingDetailsCopy`: Include copy properties in the listing.
* `AZSBlobListingDetailsAll`: List all available committed blobs, uncommitted blobs, and snapshots, and return all metadata and copy status for those blobs.


* **maxResults** - The maximum number of results to return for this operation. Use -1 to not set a limit.
* **completionHandler** - The block of code to execute with the results of the listing operation.

In this example, a helper method is used to recursively call the list blobs method every time a continuation token is returned.

    -(void)listBlobsInContainer{
      // Create a storage account object from a connection string.
      AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

      // Create a blob service client object.
      AZSCloudBlobClient *blobClient = [account getBlobClient];

      // Create a local container object.
      AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

      //List all blobs in container
      [self listBlobsInContainerHelper:blobContainer continuationToken:nil prefix:nil blobListingDetails:AZSBlobListingDetailsAll maxResults:-1 completionHandler:^(NSError *error) {
         if (error != nil){
             NSLog(@"Error in creating container.");
         }
      }];
    }

    //List blobs helper method
    -(void)listBlobsInContainerHelper:(AZSCloudBlobContainer *)container continuationToken:(AZSContinuationToken *)continuationToken prefix:(NSString *)prefix blobListingDetails:(AZSBlobListingDetails)blobListingDetails maxResults:(NSUInteger)maxResults completionHandler:(void (^)(NSError *))completionHandler
    {
      [container listBlobsSegmentedWithContinuationToken:continuationToken prefix:prefix useFlatBlobListing:YES blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:^(NSError *error, AZSBlobResultSegment *results) {
         if (error)
         {
             completionHandler(error);
         }
         else
         {
             for (int i = 0; i < results.blobs.count; i++) {
                 NSLog(@"%@",[(AZSCloudBlockBlob *)results.blobs[i] blobName]);
             }
             if (results.continuationToken)
             {
                 [self listBlobsInContainerHelper:container continuationToken:results.continuationToken prefix:prefix blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:completionHandler];
             }
             else
             {
                 completionHandler(nil);
             }
         }
      }];
    }

## Download a blob
The following example shows how to download a blob to a NSString object.

     -(void)downloadBlobToString{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create a local blob object
        AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

        // Download blob    
        [blockBlob downloadToTextWithCompletionHandler:^(NSError *error, NSString *text) {
            if (error) {
                NSLog(@"Error in downloading blob");
            }
            else{
                NSLog(@"%@",text);
            }
        }];
    }

## Delete a blob
The following example shows how to delete a blob.

     -(void)deleteBlob{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create a local blob object
        AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob1"];

        // Delete blob
        [blockBlob deleteWithCompletionHandler:^(NSError *error) {
          if (error) {
            NSLog(@"Error in deleting blob.");
          }
        }];
     }

## Delete a blob container
The following example shows how to delete a container.

     -(void)deleteContainer{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Delete container
        [blobContainer deleteContainerIfExistsWithCompletionHandler:^(NSError *error, BOOL success) {
            if(error){
                NSLog(@"Error in deleting container");
            }
        }];
    }

## Next steps
Now that you've learned the basics of Blob storage, follow these links to learn about more complex storage tasks.

* [Azure Storage iOS Library](https://github.com/azure/azure-storage-ios)
* [Azure Storage REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)
* [Transfer data with the AzCopy command-line utility](storage-use-azcopy.md)
* [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage)

If you have questions regarding this library feel free to post to our [MSDN Azure forum](http://social.msdn.microsoft.com/Forums/windowsazure/home?forum=windowsazuredata) or [Stack Overflow](http://stackoverflow.com/questions/tagged/windows-azure-storage+or+windows-azure-storage+or+azure-storage-blobs+or+azure-storage-tables+or+azure-table-storage+or+windows-azure-queues+or+azure-storage-queues+or+azure-storage-emulator+or+azure-storage-files).
If you have feature suggestions for Azure Storage please post to [Azure Storage Feedback](https://feedback.azure.com/forums/217298-storage/).

[Azure Storage iOS Library]: https://github.com/azure/azure-storage-ios
[Azure Storage REST API]: https://msdn.microsoft.com/library/azure/dd179355.aspx
[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage