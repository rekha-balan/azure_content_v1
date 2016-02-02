<properties
    pageTitle="How to use blob storage from PHP | Microsoft Azure"
    description="Learn how to use the Azure blob service to upload, list, download, and delete blobs. Code samples are written in PHP."
    documentationCenter="php"
    services="storage"
    authors="tfitzmac"
    manager="wpickett"
    editor="mollybos"/>

<tags
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="PHP"
    ms.topic="article"
    ms.date="12/16/2015"
    ms.author="tomfitz"/>

# How to use blob storage from PHP
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
This guide shows you how to perform common scenarios using the Azure blob service. The samples are written in PHP and use the [Azure SDK for PHP](http://go.microsoft.com/fwlink/?LinkID=252473). The scenarios covered include **uploading**, **listing**, **downloading**, and **deleting** blobs. For more information on blobs, see the [Next steps](#next-steps.md) section.

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
 

## Create a PHP application
The only requirement for creating a PHP application that accesses the Azure blob service is the referencing of classes in the Azure SDK for PHP from within your code. You can use any development tools to create your application, including Notepad.

In this guide, you use service features, which can be called within a PHP application locally or in code running within an Azure web role, worker role, or website.

## Get the Azure Client Libraries
### Install via Composer

1. [Install Git][install-git]. Note that on Windows, you must also add the Git executable to your PATH environment variable. 

2. Create a file named **composer.json** in the root of your project and add the following code to it:

	```
	{
	    "repositories": [
	        {
	            "type": "pear",
	            "url": "http://pear.php.net"
	        }
	    ],
	    "require": {
	        "pear-pear.php.net/mail_mime" : "*",
	        "pear-pear.php.net/http_request2" : "*",
	        "pear-pear.php.net/mail_mimedecode" : "*",
	        "microsoft/windowsazure": "*"
	    }
	}
	```

3. Download **[composer.phar][composer-phar]** in your project root.

4. Open a command prompt and execute the following command in your project root

	```
	php composer.phar install
	```

### Install manually

To download and install the PHP Client Libraries for Azure manually, follow these steps:

> [AZURE.NOTE] The PHP Client Libraries for Azure have a dependency on the [HTTP_Request2](http://pear.php.net/package/HTTP_Request2), [Mail_mime](http://pear.php.net/package/Mail_mime), and [Mail_mimeDecode](http://pear.php.net/package/Mail_mimeDecode) PEAR packages. The recommended way to resolve these dependencies is to install these packages using the [PEAR package manager](http://pear.php.net/manual/en/installation.php).
 
1. Download a .zip archive that contains the libraries from [GitHub][php-sdk-github]. Alternatively, fork the repository and clone it to your local machine. The latter option requires a GitHub account and having Git installed locally.
	
2. Copy the `WindowsAzure` directory of the downloaded archive to your application directory structure.

For more information about installing the PHP Client Libraries for Azure (including information about installing as a PEAR package), see [Download the Azure SDK for PHP][download-SDK-PHP].

[php-sdk-github]: http://go.microsoft.com/fwlink/?LinkId=252719
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[download-SDK-PHP]: ../articles/php-download-sdk.md
[composer-phar]: http://getcomposer.org/composer.phar


## Configure your application to access the blob service
To use the Azure blob service APIs, you need to:

1. Reference the autoloader file using the [require_once](http://php.net/require_once) statement, and
2. Reference any classes you might use.

The following example shows how to include the autoloader file and reference the **ServicesBuilder** class.

> [!NOTE]
> This example (and other examples in this article) assume you have installed the PHP Client Libraries for Azure via Composer. If you installed the libraries manually or as a PEAR package, you need to reference the `WindowsAzure.php` autoloader file.
> 
> 
    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;


In the examples below, the `require_once` statement will be shown always, but only the classes necessary for the example to execute are referenced.

## Set up an Azure storage connection
To instantiate an Azure blob service client, you must first have a valid connection string. The format for the blob service connection string is:

For accessing a live service:

    DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

For accessing the storage emulator:

    UseDevelopmentStorage=true


To create any Azure service client, you need to use the **ServicesBuilder** class. You can:

* Pass the connection string directly to it or
* Use the **CloudConfigurationManager (CCM)** to check multiple external sources for the connection string:  * By default, it comes with support for one external source - environmental variables.
* You can add new sources by extending the **ConnectionStringSource** class.



For the examples outlined here, the connection string will be passed directly.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);

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

A **BlobRestProxy** object lets you create a blob container with the **createContainer** method. When creating a container, you can set options on the container, but doing so is not required. (The example below shows how to set the container access control list (ACL) and container metadata.)

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Blob\Models\CreateContainerOptions;
    use WindowsAzure\Blob\Models\PublicAccessType;
    use WindowsAzure\Common\ServiceException;

    // Create blob REST proxy.
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


    // OPTIONAL: Set public access policy and metadata.
    // Create container options object.
    $createContainerOptions = new CreateContainerOptions();

    // Set public access policy. Possible values are
    // PublicAccessType::CONTAINER_AND_BLOBS and PublicAccessType::BLOBS_ONLY.
    // CONTAINER_AND_BLOBS:
    // Specifies full public read access for container and blob data.
    // proxys can enumerate blobs within the container via anonymous
    // request, but cannot enumerate containers within the storage account.
    //
    // BLOBS_ONLY:
    // Specifies public read access for blobs. Blob data within this
    // container can be read via anonymous request, but container data is not
    // available. proxys cannot enumerate blobs within the container via
    // anonymous request.
    // If this value is not specified in the request, container data is
    // private to the account owner.
    $createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);

    // Set container metadata.
    $createContainerOptions->addMetaData("key1", "value1");
    $createContainerOptions->addMetaData("key2", "value2");

    try    {
        // Create container.
        $blobRestProxy->createContainer("mycontainer", $createContainerOptions);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179439.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

Calling **setPublicAccess(PublicAccessType::CONTAINER\_AND\_BLOBS)** makes the container and blob data accessible via anonymous requests. Calling **setPublicAccess(PublicAccessType::BLOBS_ONLY)** makes only blob data accessible via anonymous requests. For more information about container ACLs, see [Set container ACL (REST API)](http://msdn.microsoft.com/library/azure/dd179391.aspx).

For more information about Blob service error codes, see [Blob Service Error Codes](http://msdn.microsoft.com/library/azure/dd179439.aspx).

## Upload a blob into a container
To upload a file as a blob, use the **BlobRestProxy->createBlockBlob** method. This operation creates the blob if it doesn't exist, or overwrites it if it does. The code example below assumes that the container has already been created and uses [fopen](http://www.php.net/fopen) to open the file as a stream.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create blob REST proxy.
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


    $content = fopen("c:\myfile.txt", "r");
    $blob_name = "myblob";

    try    {
        //Upload blob
        $blobRestProxy->createBlockBlob("mycontainer", $blob_name, $content);
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179439.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

Note that the previous sample uploads a blob as a stream. However, a blob can also be uploaded as a string using, for example, the [file\_get\_contents](http://php.net/file_get_contents) function. To do this using the previous sample, change `$content = fopen("c:\myfile.txt", "r");` to `$content = file_get_contents("c:\myfile.txt");`.

## List the blobs in a container
To list the blobs in a container, use the **BlobRestProxy->listBlobs** method with a **foreach** loop to loop through the result. The following code displays the name of each blob as output in a container and displays its URI to the browser.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create blob REST proxy.
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


    try    {
        // List blobs.
        $blob_list = $blobRestProxy->listBlobs("mycontainer");
        $blobs = $blob_list->getBlobs();

        foreach($blobs as $blob)
        {
            echo $blob->getName().": ".$blob->getUrl()."<br />";
        }
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179439.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }


## Download a blob
To download a blob, call the **BlobRestProxy->getBlob** method, then call the **getContentStream** method on the resulting **GetBlobResult** object.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create blob REST proxy.
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


    try    {
        // Get blob.
        $blob = $blobRestProxy->getBlob("mycontainer", "myblob");
        fpassthru($blob->getContentStream());
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179439.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

Note that the example above gets a blob as a stream resource (the default behavior). However, you can use the [stream\_get\_contents](http://www.php.net/stream_get_contents) function to convert the returned stream to a string.

## Delete a blob
To delete a blob, pass the container name and blob name to **BlobRestProxy->deleteBlob**.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create blob REST proxy.
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


    try    {
        // Delete container.
        $blobRestProxy->deleteBlob("mycontainer", "myblob");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179439.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Delete a blob container
Finally, to delete a blob container, pass the container name to **BlobRestProxy->deleteContainer**.

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // Create blob REST proxy.
    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


    try    {
        // Delete container.
        $blobRestProxy->deleteContainer("mycontainer");
    }
    catch(ServiceException $e){
        // Handle exception based on error codes and messages.
        // Error codes and messages are here:
        // http://msdn.microsoft.com/library/azure/dd179439.aspx
        $code = $e->getCode();
        $error_message = $e->getMessage();
        echo $code.": ".$error_message."<br />";
    }

## Next steps
Now that you've learned the basics of the Azure blob service, follow these links to learn about more complex storage tasks.

* Visit the [Azure Storage team blog](http://blogs.msdn.com/b/windowsazurestorage/)
* See the PHP block blob example at [https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/BlockBlobExample.php](https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/BlockBlobExample.php).
* See the PHP page blob example at [https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/PageBlobExample.php](https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/PageBlobExample.php)
* [Transfer data with the AzCopy command-line utility](storage-use-azcopy.md)

For more information, see also the [PHP Developer Center](/develop/php/).

[download]: http://go.microsoft.com/fwlink/?LinkID=252473
[container-acl]: http://msdn.microsoft.com/library/azure/dd179391.aspx
[error-codes]: http://msdn.microsoft.com/library/azure/dd179439.aspx
[file_get_contents]: http://php.net/file_get_contents
[require_once]: http://php.net/require_once
[fopen]: http://www.php.net/fopen
[stream-get-contents]: http://www.php.net/stream_get_contents
