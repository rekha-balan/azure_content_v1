<properties
    pageTitle="Redis Cluster Resource Manager Template"
    description="Learn to easily deploy a new Redis cluster on Ubuntu VMs using Azure PowerShell or the Azure CLI and a Resource Manager template"
    services="virtual-machines"
    documentationCenter=""
    authors="timwieman"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"/>

<tags
    ms.service="virtual-machines"
    ms.workload="multiple"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/04/2015"
    ms.author="twieman"/>

# Redis cluster with a Resource Manager template
> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model, which Microsoft recommends for most new deployments instead of the classic deployment model.

Redis is an open-source key-value cache and store, where keys can contain data structures such as strings, hashes, lists, sets and sorted sets. Redis supports a set of atomic operations on these data types.  With the release of Redis version 3.0, Redis Cluster is now available in the latest stable version of Redis.  Redis Cluster is a distributed implementation of Redis where data is automatically sharded across multiple Redis nodes, with the ability to continue operations when a subset of nodes is experiencing failures.

Microsoft Azure Redis Cache is a dedicated Redis cache service, managed by Microsoft, but not all Microsoft Azure customers want to use Azure Redis Cache.  Some want to keep their Redis cache behind a subnet within their own Azure deployments, while others are more comfortable hosting their own Redis servers on Linux virtual machines in order to fully take advantage of all Redis features.

This tutorial will walk through using a sample Azure Resource Manager  template to deploy a Redis cluster on Ubuntu VMs within a subnet in a resource group in Microsoft Azure.  In addition to Redis 3.0 Cluster, this template also supports deploying Redis 2.8 with Redis Sentinel.  Note that this tutorial will focus on the Redis 3.0 Cluster implementation.

The Redis cluster is created behind a subnet, so there is no public IP access to the Redis cluster.  As part of the deployment, an optional “jump box” can be deployed.  This “jump box” is an Ubuntu VM deployed in the subnet as well, but it *does* expose a public IP address with an open SSH port that you can SSH to.  Then from the “jump box”, you can SSH to all the Redis VMs in the subnet.

This template utilizes a “t-shirt size” concept in order to specify a “Small”, “Medium”, or “Large” Redis Cluster setup.  When the Azure Resource Manager template language supports more dynamic template sizing, this could be changed to specify the number of Redis Cluster master nodes, slave nodes, VM size, etc.  For now, you can see the VM size and the number of masters and slaves defined in the file azuredeploy.json in the variables `tshirtSizeSmall`, `tshirtSizeMedium`, and `tshirtSizeLarge`.

The Redis Cluster template for the “Medium” t-shirt size creates this configuration:

![cluster-architecture](media/virtual-machines-redis-template/cluster-architecture.png)

Before diving into more details related to Azure Resource Manager and the template we will use for this deployment, make sure you have Azure PowerShell or the Azure CLI configured correctly.

## Setting up PowerShell for Resource Manager templates

Before you can use Azure PowerShell with Resource Manager, you will need to have the right Windows PowerShell and Azure PowerShell versions.

### Verify PowerShell versions

Verify you have Windows PowerShell version 3.0 or 4.0. To find the version of Windows PowerShell, type this command at a Windows PowerShell command prompt.

	$PSVersionTable

You will receive the following type of information:

	Name                           Value
	----                           -----
	PSVersion                      3.0
	WSManStackVersion              3.0
	SerializationVersion           1.1.0.1
	CLRVersion                     4.0.30319.18444
	BuildVersion                   6.2.9200.16481
	PSCompatibleVersions           {1.0, 2.0, 3.0}
	PSRemotingProtocolVersion      2.2


Verify that the value of **PSVersion** is 3.0 or 4.0. If not, see [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) or [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855).

### Set your Azure account and subscription

If you don't already have an Azure subscription, you can activate your [MSDN subscriber benefits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) or sign up for a [free trial](https://azure.microsoft.com/pricing/free-trial/).

Open an Azure PowerShell command prompt and log on to Azure with this command.

	Login-AzureRmAccount

If you have multiple Azure subscriptions, you can list your Azure subscriptions with this command.

	Get-AzureRmSubscription

You will receive the following type of information:

	SubscriptionId            : fd22919d-eaca-4f2b-841a-e4ac6770g92e
	SubscriptionName          : Visual Studio Ultimate with MSDN
	Environment               : AzureCloud
	SupportedModes            : AzureServiceManagement,AzureResourceManager
	DefaultAccount            : johndoe@contoso.com
	Accounts                  : {johndoe@contoso.com}
	IsDefault                 : True
	IsCurrent                 : True
	CurrentStorageAccountName :
	TenantId                  : 32fa88b4-86f1-419f-93ab-2d7ce016dba7

You can set the current Azure subscription by running these commands at the Azure PowerShell command prompt. Replace everything within the quotes, including the < and > characters, with the correct name.

	$subscr="<SubscriptionName from the display of Get-AzureRmSubscription>"
	Select-AzureRmSubscription -SubscriptionName $subscr -Current

For more information about Azure subscriptions and accounts, see [How to: Connect to your subscription](powershell-install-configure.md#Connect).


<properties services="virtual-machines" title="Using Azure CLI with Azure Resource Manager" authors="squillace" solutions="" manager="timlt" editor="tysonn" />

<tags
   ms.service="virtual-machine"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="linux"
   ms.workload="infrastructure"
   ms.date="04/13/2015"
   ms.author="rasquill" />

## Using Azure CLI with Azure Resource Manager (ARM)

Before you can use the Azure CLI with Resource Manager commands and templates to deploy Azure resources and workloads using resource groups, you will need an account with Azure (of course). If you do not have an account, you can get a [free Azure trial here](https://azure.microsoft.com/pricing/free-trial/).

> [AZURE.NOTE] If you don't already have an Azure account but you do have a subscription to MSDN subscription, you can get free Azure credits by activating your [MSDN subscriber benefits here](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) -- or you can use the free account. Either will work for Azure access.

### Step 1: Verify the Azure CLI version

To use Azure CLI for imperative commands and ARM templates, you need to have at least version 0.8.17. To verify your version, type `azure --version`. You should see something like:

    $ azure --version
    0.8.17 (node: 0.10.25)

If you need to update your version of Azure CLI, see [Azure CLI](https://github.com/Azure/azure-xplat-cli).

### Step 2: Verify you are using a work or school identity with Azure

You can only use the ARM command mode if you are using an [Azure Active Directory tenant](https://msdn.microsoft.com/library/azure/jj573650.aspx#BKMK_WhatIsAnAzureADTenant) or a [Service Principal Name](https://msdn.microsoft.com/library/azure/dn132633.aspx). (These are also called *organizational ids*.)

To see if you have one, log in by typing `azure login` and using your work or school username and password when prompted. If you do have one, you should see something like the following:

    $ azure login
    info:    Executing command login
    warn:    Please note that currently you can login only via Microsoft organizational account or service principal. For instructions on how to set them up, please read http://aka.ms/Dhf67j.
    Username: ahmet@contoso.onmicrosoft.com
    Password: *********
    |info:    Added subscription Visual Studio Ultimate with MSDN
    info:    Setting subscription Visual Studio Ultimate with MSDN as default
    info:    Added subscription Azure Free Trial
    +
    info:    login command OK

If you do not see this, you must create a new tenant (or service principal) with your Microsoft account identity. (This is often the case with personal MSDN subscriptions or free trial subscriptions.) To create a work or school id from your Azure account created with a Microsoft id, see [Associate an Azure AD Directory with a new Azure Subscription](https://msdn.microsoft.com/library/azure/jj573650.aspx#BKMK_WhatIsAnAzureADTenant). If you think you should have an organizational id already, you may need to talk with the person who created the account for you.

### Step 3: Choose your Azure subscription

If you have only one subscription in your Azure account, Azure CLI associates itself with that subscription by default. If you have more than one subscription, you need to select the subscription you want to use by typing `azure account set <subscription id or name> true` where _subscription id or name_ is either the subscription id or the subscription name that you would like to work with in the current session.

You should see something like the following:

    $ azure account set "Azure Free Trial" true
    info:    Executing command account set
    info:    Setting subscription to "Azure Free Trial" with id "2lskd82-434-4730-9df9-akd83lsk92sa".
    info:    Changes saved
    info:    account set command OK

### Step 4: Place your Azure CLI in the ARM mode

To use the Azure Resource Management (ARM) mode with the Azure CLI, type `azure config mode arm`. You should see something like the following:

    $ azure config mode arm
    info:    New mode is arm

> [AZURE.NOTE] You can switch back to use Azure service management commands by typing `azure config mode asm`.


## Deploy a Redis cluster by using a Resource Manager template
Follow these steps to create a Redis cluster by using a Resource Manager template from the GitHub template repository. Each step will include directions for both Azure PowerShell and the Azure CLI.

### Step 1-a: Download the template files by using Azure PowerShell
Create a local folder for the JSON template and other associated files (for example, C:\Azure\Templates\RedisCluster).

Substitute in the folder name of your local folder and run these commands:

```powershell
$folderName="<folder name, such as C:\Azure\Templates\RedisCluster>"
$webclient = New-Object System.Net.WebClient
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy.json"
$filePath = $folderName + "\azuredeploy.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy-parameters.json"
$filePath = $folderName + "\azuredeploy-parameters.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/empty-resources.json"
$filePath = $folderName + "\empty-resources.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/jumpbox-resources.json"
$filePath = $folderName + "\jumpbox-resources.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/node-resources.json"
$filePath = $folderName + "\node-resources.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/redis-cluster-install.sh"
$filePath = $folderName + "\redis-cluster-install.sh"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/redis-cluster-setup.sh"
$filePath = $folderName + "\redis-cluster-setup.sh"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/redis-sentinel-startup.sh"
$filePath = $folderName + "\redis-sentinel-startup.sh"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/shared-resources.json"
$filePath = $folderName + "\shared-resources.json"
$webclient.DownloadFile($url,$filePath)
```

### Step 1-b: Download the template files by using the Azure CLI
Clone the entire template repository by using a Git client of your choice, for example:

```
git clone https://github.com/Azure/azure-quickstart-templates C:\Azure\Templates
```

When the cloning is completed, look for the **redis-high-availability** folder in your C:\Azure\Templates directory.

### Step 2: (optional) Understand the template parameters
When you create a Redis cluster by using a template, you must specify a set of configuration parameters. To see the parameters that you need to specify for the template in a local JSON file before running the command to create the Redis cluster, open the JSON file in a tool or text editor of your choice.

Look for the "parameters" section at the top of the file, which lists the set of parameters that are needed by the template to configure the Redis cluster. Here is the "parameters" section for the azuredeploy.json template:

```json
"parameters": {
    "adminUsername": {
        "type": "string",
        "metadata": {
            "Description": "Administrator user name used when provisioning virtual machines"
        }
    },
    "adminPassword": {
        "type": "securestring",
        "metadata": {
            "Description": "Administrator password used when provisioning virtual machines"
        }
    },
    "storageAccountName": {
        "type": "string",
        "defaultValue": "",
        "metadata": {
            "Description": "Unique namespace for the Storage account where the virtual machine's disks will be placed"
        }
    },
    "location": {
        "type": "string",
        "metadata": {
            "Description": "Location where resources will be provisioned"
        }
    },
    "virtualNetworkName": {
        "type": "string",
        "defaultValue": "redisVirtNet",
        "metadata": {
            "Description": "The arbitrary name of the virtual network provisioned for the Redis cluster"
        }
    },
    "addressPrefix": {
        "type": "string",
        "defaultValue": "10.0.0.0/16",
        "metadata": {
            "Description": "The network address space for the virtual network"
        }
    },
    "subnetName": {
        "type": "string",
        "defaultValue": "redisSubnet1",
        "metadata": {
            "Description": "Subnet name for the virtual network that resources will be provisioned into"
        }
    },
    "subnetPrefix": {
        "type": "string",
        "defaultValue": "10.0.0.0/24",
        "metadata": {
            "Description": "Address space for the virtual network subnet"
        }
    },
    "nodeAddressPrefix": {
        "type": "string",
        "defaultValue": "10.0.0.1",
        "metadata": {
            "Description": "The IP address prefix that will be used for constructing a static private IP address for each node in the cluster"
        }
    },
    "jumpbox": {
        "type": "string",
        "defaultValue": "Disabled",
        "allowedValues": [
            "Enabled",
            "Disabled"
        ],
        "metadata": {
            "Description": "The flag allowing to enable or disable provisioning of the jump-box VM that can be used to access the Redis nodes"
        }
    },
    "tshirtSize": {
        "type": "string",
        "defaultValue": "Small",
        "allowedValues": [
            "Small",
            "Medium",
            "Large"
        ],
        "metadata": {
            "Description": "T-shirt size of the Redis deployment"
        }
    },
    "redisVersion": {
        "type": "string",
        "defaultValue": "stable",
        "metadata": {
            "Description": "The version of the Redis package to be deployed on the cluster (or use 'stable' to pull in the latest and greatest)"
        }
    },
    "redisClusterName": {
        "type": "string",
        "defaultValue": "redis-cluster",
        "metadata": {
            "Description": "The arbitrary name of the Redis cluster"
        }
    }
},
```

Each parameter has details such as data type and allowed values. This allows for validation of parameters passed during template execution in an interactive mode (e.g., Azure PowerShell or Azure CLI), as well as a self-discovery UI that could be dynamically built by parsing the list of required parameters and their descriptions.

### Step 3-a: Deploy a Redis cluster by using a template via Azure PowerShell
Prepare a parameters file for your deployment by creating a JSON file containing runtime values for all parameters. This file will then be passed as a single entity to the deployment command. If you do not include a parameters file, Azure PowerShell will use any default values specified in the template, and then prompt you to fill in the remaining values.

Here is an example you can find in the azuredeploy-parameters.json file.  Note that you will need to provide valid values for the parameters `storageAccountName`, `adminUsername`, and `adminPassword`, plus any customizations to the other parameters:

```json
{
    "storageAccountName": {
        "value": "redisdeploy1"
    },
    "adminUsername": {
        "value": ""
    },
    "adminPassword": {
        "value": ""
    },
    "location": {
        "value": "West US"
    },
    "virtualNetworkName": {
        "value": "redisClustVnet"
    },
    "subnetName": {
        "value": "Subnet1"
    },
    "addressPrefix": {
        "value": "10.0.0.0/16"
    },
    "subnetPrefix": {
        "value": "10.0.0.0/24"
    },
    "nodeAddressPrefix": {
        "value": "10.0.0.1"
    },
    "redisVersion": {
        "value": "3.0.0"
    },
    "redisClusterName": {
        "value": "redis-arm-cluster"
    },
    "jumpbox": {
        "value": "Enabled"
    },
    "tshirtSize":  {
        "value": "Small"
    }
}
```

> [!NOTE]
> The parameter `storageAccountName` must be a non-existent, unique Storage account name that satisfies the naming requirements for a Microsoft Azure Storage account (lowercase letters and numbers only).  This Storage account will be created as part of the deployment process.
> 
> 
Fill in an Azure deployment name, resource group name, Azure location, and the folder of your saved JSON files. Then run these commands:

```powershell
$deployName="<deployment name>, such as TestDeployment"
$RGName="<resource group name>, such as TestRG"
$locName="<Azure location, such as West US>"
$folderName="<folder name, such as C:\Azure\Templates\RedisCluster>"
$templateFile= $folderName + "\azuredeploy.json"
$templateParameterFile= $folderName + "\azuredeploy-parameters.json"

New-AzureResourceGroup –Name $RGName –Location $locName

New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParameterFile $templateParameterFile -TemplateFile $templateFile
```

> [!NOTE]
> `$RGName` must be unique within your subscription.
> 
> 
When you run the **New-AzureResourceGroupDeployment** command, this will extract parameter values from the parameters JSON file (azuredeploy-parameters.json), and will start executing the template accordingly. Defining and using multiple parameter files with your different environments (test, production, etc.) will promote template reuse and simplify complex multi-environment solutions.

When deploying, please keep in mind that a new Azure Storage account needs to be created, so the name you provide as the Storage account parameter must be unique and meet all requirements for an Azure Storage account (lowercase letters and numbers only).

During the deployment, you will see something like this:

    PS C:\> New-AzureResourceGroup –Name $RGName –Location $locName

    ResourceGroupName : TestRG
    Location          : westus
    ProvisioningState : Succeeded
    Tags              :
    Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *

    ResourceId        : /subscriptions/1234abc1-abc1-1234-12a1-ab1ab12345ab/resourceGroups/TestRG

    PS C:\> New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParameterFile $templateParameterFile -TemplateFile $templateFile
    VERBOSE: 2:39:10 PM - Template is valid.
    VERBOSE: 2:39:14 PM - Create template deployment 'TestDeployment'.
    VERBOSE: 2:39:25 PM - Resource Microsoft.Resources/deployments 'shared-resources' provisioning status is running
    VERBOSE: 2:40:04 PM - Resource Microsoft.Resources/deployments 'node-resources0' provisioning status is running
    VERBOSE: 2:40:05 PM - Resource Microsoft.Resources/deployments 'jumpbox-resources' provisioning status is running
    VERBOSE: 2:40:05 PM - Resource Microsoft.Resources/deployments 'node-resources1' provisioning status is running
    VERBOSE: 2:40:05 PM - Resource Microsoft.Resources/deployments 'shared-resources' provisioning status is succeeded
    VERBOSE: 2:42:23 PM - Resource Microsoft.Resources/deployments 'jumpbox-resources' provisioning status is succeeded
    VERBOSE: 2:47:44 PM - Resource Microsoft.Resources/deployments 'node-resources1' provisioning status is succeeded
    VERBOSE: 2:47:57 PM - Resource Microsoft.Resources/deployments 'node-resources0' provisioning status is succeeded
    VERBOSE: 2:48:01 PM - Resource Microsoft.Resources/deployments 'lastnode-resources' provisioning status is running
    VERBOSE: 2:58:24 PM - Resource Microsoft.Resources/deployments 'lastnode-resources' provisioning status is succeeded

    DeploymentName    : TestDeployment
    ResourceGroupName : TestRG
    ProvisioningState : Succeeded
    Timestamp         : 4/28/2015 7:58:23 PM
    Mode              : Incremental
    TemplateLink      :
    Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    adminUsername    String                     myadmin
                    adminPassword    SecureString
                    storageAccountName  String                     myuniqstgacct87
                    location         String                     West US
                    virtualNetworkName  String                     redisClustVnet
                    addressPrefix    String                     10.0.0.0/16
                    subnetName       String                     Subnet1
                    subnetPrefix     String                     10.0.0.0/24
                    nodeAddressPrefix  String                     10.0.0.1
                    jumpbox          String                     Enabled
                    tshirtSize       String                     Small
                    redisVersion     String                     3.0.0
                    redisClusterName  String                     redis-arm-cluster

    Outputs           :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    installCommand   String                     bash redis-cluster-install.sh -n redis-arm-cluster -v
                    3.0.0 -c 3 -m 3 -s 0 -p 10.0.0.1
                    setupCommand     String                     bash redis-cluster-install.sh -n redis-arm-cluster -v
                    3.0.0 -c 3 -m 3 -s 0 -p 10.0.0.1 -l
    ...

During and after deployment, you can check all the requests that were made during provisioning, including any errors that occurred.

To do that, go to the [Azure portal](https://portal.azure.com), and do the following:

* In the left-hand navigation bar, click **Browse**, and then scroll down and click **Resource Groups**.
* Select the resource group that you just created, to bring up the “Resource Group” blade.
* In the **Monitoring** section, select the “Events” bar graph. This will display the events for your deployment.
* By clicking individual events, you can drill further down into the details of each operation made on behalf of the template.

If you need to remove this resource group and all of its resources (the Storage account, virtual machine, and virtual network) after testing, use this command:

```powershell
Remove-AzureResourceGroup –Name "<resource group name>"
```

### Step 3-b: Deploy a Redis cluster by using a template via the Azure CLI
To deploy a Redis cluster via the Azure CLI, first create a resource group by specifying a name and a location:

```powershell
azure group create TestRG "West US"
```

Pass this resource group name, the location of the JSON template file, and the location of the parameters file (see the above Azure PowerShell section for details) into the following command:

```powershell
azure group deployment create TestRG -f .\azuredeploy.json -e .\azuredeploy-parameters.json
```

You can check the status of individual resource deployments by using the following command:

```powershell
azure group deployment list TestRG
```

## A tour of the Redis cluster template structure and file organization
In order to create a robust and reusable approach to Resource Manager template design, additional thinking is required to organize the series of complex and interrelated tasks required during deployment of a complex solution like Redis Cluster. By leveraging Resource Manager template linking capabilities, resource looping, and script execution through related extensions, it’s possible to implement a modular approach that can be reused with virtually any complex template-based deployment.

This diagram describes the relationships between all files downloaded from GitHub for this deployment:

![redis-files](media/virtual-machines-redis-template/redis-files.png)

This section steps you through the structure of the azuredeploy.json template for the Redis cluster.

If you have not downloaded a copy of the template file, designate a local folder as the location for the file and create it (for example, C:\Azure\Templates\RedisCluster). Fill in the folder name and run these commands:

```powershell
$folderName="<folder name, such as C:\Azure\Templates\RedisCluster>"
$webclient = New-Object System.Net.WebClient
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy.json"
$filePath = $folderName + "\azuredeploy.json"
$webclient.DownloadFile($url,$filePath)
```

Open the azuredeploy.json template in a text editor or tool of your choice. The following information describes the structure of the template file and the purpose of each section. Alternately, you can see the contents of this template in your browser from [here](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy.json).

### "parameters" section
We mentioned already the role of azuredeploy-parameters.json, which will be used to pass a given set of parameter values during template execution. The "parameters" section of azuredeploy.json specifies parameters that are used to input data into this template.

Here is an example of a parameter for the “t-shirt size”:

```json
"tshirtSize": {
    "type": "string",
    "defaultValue": "Small",
    "allowedValues": [
        "Small",
        "Medium",
        "Large"
    ],
    "metadata": {
        "Description": "T-shirt size of the Redis deployment"
    }
},
```

> [!NOTE]
> Notice that `defaultValue` may be specified, as well as `allowedValues`.
> 
> 
### "variables" section
The "variables" section specifies variables that can be used throughout this template. Basically this contains a number of fields (JSON data types or fragments) that will be set to constants or calculated values at execution time.  Here are some examples that range from simple to more complex:

```json
"vmStorageAccountContainerName": "vhd-redis",
"vmStorageAccountDomain": ".blob.core.windows.net",
"vnetID": "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworkName'))]",
...
"machineSettings": {
    "adminUsername": "[parameters('adminUsername')]",
    "machineNamePrefix": "redisnode-",
    "osImageReference": {
        "publisher": "[variables('osFamilyUbuntu').imagePublisher]",
        "offer": "[variables('osFamilyUbuntu').imageOffer]",
        "sku": "[variables('osFamilyUbuntu').imageSKU]",
        "version": "latest"
    }
},
...
"vmScripts": {
    "scriptsToDownload": [
        "[concat(variables('scriptUrl'), 'redis-cluster-install.sh')]",
        "[concat(variables('scriptUrl'), 'redis-cluster-setup.sh')]",
        "[concat(variables('scriptUrl'), 'redis-sentinel-startup.sh')]"
    ],
    "installCommand": "[concat('bash ', variables('installCommand'))]",
    "setupCommand": "[concat('bash ', variables('installCommand'), ' -l')]"
}
```

The `vmStorageAccountContainerName` and `vmStorageAccountDomain` variables are examples of simple name/value variables. `vnetID` is an example of a variable that is calculated at run time using the functions `resourceId` and `parameters`. `machineSettings` builds on these concepts even further by nesting the JSON object `osImageReference` in the `machineSettings` variable. `vmScripts` contains a JSON array, `scriptsToDownload`, which is calculated at run time using the `concat` and `variables` functions.  

If you want to customize the size of the Redis Cluster deployment, then you can change the properties of the variables `tshirtSizeSmall`, `tshirtSizeMedium`, and `tshirtSizeLarge` in the azuredeploy.json template.  

```json
"tshirtSizeSmall": {
    "vmSizeMember": "Standard_A1",
    "numberOfMasters": 3,
    "numberOfSlaves": 0,
    "totalMemberCount": 3,
    "totalMemberCountExcludingLast": 2,
    "vmTemplate": "[concat(variables('templateBaseUrl'), 'node-resources.json')]"
},
"tshirtSizeMedium": {
    "vmSizeMember": "Standard_A2",
    "numberOfMasters": 3,
    "numberOfSlaves": 3,
    "totalMemberCount": 6,
    "totalMemberCountExcludingLast": 5,
    "vmTemplate": "[concat(variables('templateBaseUrl'), 'node-resources.json')]"
},
"tshirtSizeLarge": {
    "vmSizeMember": "Standard_A5",
    "numberOfMasters": 3,
    "numberOfSlaves": 6,
    "totalMemberCount": 9,
    "totalMemberCountExcludingLast": 8,
    "arbiter": "Enabled",
    "vmTemplate": "[concat(variables('templateBaseUrl'), 'node-resources.json')]"
},
```

Note:  The `totalMemberCountExcludingLast` and `totalMemberCount` properties are needed because the template language currently does not have “math” operations.

More information regarding the template language can be found in MSDN at [Azure Resource Manager Template Language](../resource-group-authoring-templates.md).

### "resources" section
The "resources" section is where most of the action happens. Looking carefully inside this section, you can immediately identify two different cases. The first is an element defined of type `Microsoft.Resources/deployments` that essentially invokes a nested deployment within the main one. The second is the `templateLink` property (and related `contentVersion` property) that makes it possible specify a linked template file that will be invoked, passing a set of parameters as input. These can be seen in this template fragment:

```json
{
    "name": "shared-resources",
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2015-01-01",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('sharedTemplateUrl')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "commonSettings": {
                "value": "[variables('commonSettings')]"
            },
            "storageSettings": {
                "value": "[variables('storageSettings')]"
            },
            "networkSettings": {
                "value": "[variables('networkSettings')]"
            }
        }
    }
},
```

From this first example it is clear how azuredeploy.json in this scenario has been organized as a sort of orchestration mechanism, invoking a number of other template files. Each file is responsible for part of the required deployment activities.

In particular, the following linked templates will be used for this deployment:

* **shared-resource.json**: contains the definition of all resources that will be shared across the deployment. Examples are Storage accounts used to store a VM’s OS disks, virtual networks, and availability sets.
* **jumpbox-resources.json**: deploys the “jump box” VM and all related resources, such as network interface, public IP address, and the input endpoint used to SSH into the environment.
* **node-resources.json**: deploys all Redis Cluster node VMs and connected resources (network adapters, private IPs, etc.). This template also deploys VM extensions (custom scripts for Linux) and invokes a bash script to physically install and set up Redis on each node.  The script to invoke is passed to this template in the `machineSettings` parameter of the `commandToExecute` property.  All but one of the Redis Cluster nodes can be deployed and scripted in parallel.  One node needs to be saved until the end because the Redis Cluster setup can be run on only one node, and it must be done after all of the nodes are running the Redis server.  This is why the script to execute is passed to this template; the last node needs to run a slightly different script that will not only install the Redis server, but also set up the Redis cluster.

Let’s drill down into *how* this last template, node-resources.json, is used, as it is one of the most interesting from a template development perspective. One important concept to highlight is how a single template file can deploy multiple copies of a single resource type, and for each instance, it can set unique values for required settings. This concept is known as **resource looping**.

When node-resources.json is invoked from within the main azuredeploy.json file, it is invoked from inside a resource that uses the `copy` element to create a loop of sorts. A resource that uses the `copy` element will “copy” itself for the number of times specified in the `count` parameter of the `copy` element. For all settings where it is necessary to specify unique values between different instances of the deployed resource, the **copyindex()** function can be used to obtain a numeric value indicating the current index in that particular resource loop creation. In the following fragment from azuredeploy.json, you can see this concept applied to multiple VMs being created for the Redis Cluster nodes:

```json
{
    "type": "Microsoft.Resources/deployments",
    "name": "[concat('node-resources', copyindex())]",
    "apiVersion": "2015-01-01",
    "dependsOn": [
        "[concat('Microsoft.Resources/deployments/', 'shared-resources')]"
    ],
    "copy": {
        "name": "memberNodesLoop",
        "count": "[variables('clusterSpec').totalMemberCountExcludingLast]"
    },
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('clusterSpec').vmTemplate]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "commonSettings": {
                "value": "[variables('commonSettings')]"
            },
            "storageSettings": {
                "value": "[variables('storageSettings')]"
            },
            "networkSettings": {
                "value": "[variables('networkSettings')]"
            },
            "machineSettings": {
                "value": {
                    "adminUsername": "[variables('machineSettings').adminUsername]",
                    "machineNamePrefix": "[variables('machineSettings').machineNamePrefix]",
                    "osImageReference": "[variables('machineSettings').osImageReference]",
                    "vmSize": "[variables('clusterSpec').vmSizeMember]",
                    "machineIndex": "[copyindex()]",
                    "vmScripts": "[variables('vmScripts').scriptsToDownload]",
                    "commandToExecute": "[concat(variables('vmScripts').installCommand, ' -i ', copyindex())]"
                }
            },
            "adminPassword": {
                "value": "[parameters('adminPassword')]"
            }
        }
    }
},
```

Another important concept in resource creation is the ability to specify dependencies and precedencies between resources, as you can see in the `dependsOn` JSON array. In this particular template, you can see that the Redis Cluster nodes are dependent on the shared resources being created first.

As was previously mentioned, the last node needs to wait for provisioning until all other nodes in the Redis cluster have been provisioned with the Redis server running on them. This is accomplished in azuredeploy.json by having a resource named `lastnode-resources` that depends on the `copy` loop named `memberNodesLoop` from the template snippet above. After provisioning of `memberNodesLoop` is completed, `lastnode-resources` can be provisioned:

```json
{
    "name": "lastnode-resources",
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2015-01-01",
    "dependsOn": [
        "memberNodesLoop"
    ],
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('clusterSpec').vmTemplate]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "commonSettings": {
                "value": "[variables('commonSettings')]"
            },
            "storageSettings": {
                "value": "[variables('storageSettings')]"
            },
            "networkSettings": {
                "value": "[variables('networkSettings')]"
            },
            "machineSettings": {
                "value": {
                    "adminUsername": "[variables('machineSettings').adminUsername]",
                    "machineNamePrefix": "[variables('machineSettings').machineNamePrefix]",
                    "osImageReference": "[variables('machineSettings').osImageReference]",
                    "vmSize": "[variables('clusterSpec').vmSizeMember]",
                    "machineIndex": "[variables('clusterSpec').totalMemberCountExcludingLast]",
                    "vmScripts": "[variables('vmScripts').scriptsToDownload]",
                    "commandToExecute": "[concat(variables('vmScripts').setupCommand, ' -i ', variables('clusterSpec').totalMemberCountExcludingLast)]"
                }
            },
            "adminPassword": {
                "value": "[parameters('adminPassword')]"
            }
        }
    }
}
```

Notice how the `lastnode-resources` resource passes a slightly different `machineSettings.commandToExecute` to the linked template. This is because for the last node, in addition to the installed Redis server, it needs to call a script to set up the Redis cluster (which must be done only once after all the Redis servers are up and running).

Another interesting fragment to explore is the one related to the `CustomScriptForLinux` VM extensions. These are installed as a separate type of resource, with a dependency on each cluster node. In this case, this is used to install and set up Redis on each VM node. Let’s look at a snippet from the node-resources.json template that uses these:

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat('vmMember', parameters('machineSettings').machineIndex, '/installscript')]",
    "apiVersion": "2015-05-01-preview",
    "location": "[parameters('commonSettings').location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', 'vmMember', parameters('machineSettings').machineIndex)]"
    ],
    "properties": {
        "publisher": "Microsoft.OSTCExtensions",
        "type": "CustomScriptForLinux",
        "typeHandlerVersion": "1.2",
        "settings": {
            "fileUris": "[parameters('machineSettings').vmScripts]",
            "commandToExecute": "[parameters('machineSettings').commandToExecute]"
        }
    }
}
```

You can see that this resource depends on the resource VM already being deployed (`Microsoft.Compute/virtualMachines/vmMember<X>`, where `<X>` is the parameter `machineSettings.machineIndex`, which is the index of the VM that was passed to this script using the **copyindex()** function).

By familiarizing yourself with the other files included in this deployment, you will be able to understand all the details and best practices required to organize and orchestrate complex deployment strategies for multi-node solutions, based on any technology, leveraging Azure Resource Manager templates. While not mandatory, a recommended approach is to structure your template files as highlighted by the following diagram:

![redis-template-structure](media/virtual-machines-redis-template/redis-template-structure.png)

In essence, this approach suggests to:

* Define your core template file as a central orchestration point for all specific deployment activities, leveraging template linking to invoke sub-template executions.
* Create a specific template file that will deploy all resources shared across all other specific deployment tasks (Storage accounts, virtual network configuration, etc.). This can be heavily reused between deployments that have similar requirements in terms of common infrastructure.
* Include optional resource templates for spot requirements specific to a given resource.
* For identical members of a group of resources (nodes in a cluster, etc.), create specific templates that leverage resource looping in order to deploy multiple instances with unique properties.
* For all post-deployment tasks (product installation, configurations, etc.), leverage script deployment extensions and create scripts specific to each technology.

For more information, see [Azure Resource Manager Template Language](../resource-group-authoring-templates.md).

