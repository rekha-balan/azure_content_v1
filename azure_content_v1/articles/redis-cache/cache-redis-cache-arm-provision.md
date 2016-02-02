<properties 
    pageTitle="Provision a Redis Cache" 
    description="Use Azure Resource Manager template to deploy an Azure Redis Cache." 
    services="app-service" 
    documentationCenter="" 
    authors="tfitzmac" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="cache" 
    ms.workload="web" 
    ms.tgt_pltfrm="cache-redis" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="12/16/2015" 
    ms.author="tomfitz"/>

# Create a Redis Cache using a template
In this topic, you will learn how to create an Azure Resource Manager template that deploys an Azure Redis Cache. The cache can be used with an existing storage account to keep diagnostic data. You will learn how to define which resources are deployed and 
how to define parameters that are specified when the deployment is executed. You can use this template for your own deployments, or customize it to meet your requirements.

Currently, diagnostic settings are shared for all caches in the same region for a subscription. Updating one cache in the region will affect all other caches in the region.

For more information about creating templates, see [Authoring Azure Resource Manager Templates](../resource-group-authoring-templates.md).

For the complete template, see [Redis Cache template](https://github.com/Azure/azure-quickstart-templates/blob/master/101-redis-cache/azuredeploy.json).

> [!NOTE]
> ARM templates for the new [Premium tier](cache-premium-tier-intro.md) are available. 
> 
> * [Create a Premium Redis Cache with clustering](https://azure.microsoft.com/documentation/templates/201-redis-premium-cluster-diagnostics/)
> * [Create Premium Redis Cache with data persistence](https://azure.microsoft.com/documentation/templates/201-redis-premium-persistence/)
> * [Create Premium Redis Cache with VNet and optional clustering](https://azure.microsoft.com/documentation/templates/201-redis-premium-vnet-cluster-diagnostics/)
> 
> To check for the latest templates, see [Azure Quickstart Templates](https://azure.microsoft.com/documentation/templates/) and search for `Redis Cache`.
> 
> 
## What you will deploy
In this template, you will deploy an Azure Redis Cache that uses an existing storage account for diagnostic data.

To run the deployment automatically, click the following button:

[![Deploy to Azure](./media/cache-redis-cache-arm-provision/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-redis-cache%2Fazuredeploy.json)

## Parameters
With Azure Resource Manager, you define parameters for values you want to specify when the template is deployed. The template includes a section called Parameters that contains all of the parameter values.
You should define a parameter for those values that will vary based on the project you are deploying or based on the 
environment you are deploying to. Do not define parameters for values that will always stay the same. Each parameter value is used in the template to define the resources that are deploy. 

We will describe each parameter in the template.


### redisCacheName

The name of the Azure Redis Cache to create.

    "redisCacheName": {
      "type": "string"
    }

### redisCacheSKU

The pricing tier of the new Azure Redis Cache.

    "redisCacheSKU": {
      "type": "string",
      "allowedValues": [ "Basic", "Standard" ],
      "defaultValue": "Standard"
    }

The template defines the values that are permitted for this parameter (Basic or Standard), and assigns a default value (Standard) if no value is specified. Basic provides a single node with multiple sizes available up to 53 GB.
Standard provides two-node Primary/Replica with multiple sizes available up to 53 GB and 99.9% SLA.

### redisCacheFamily

The family for the sku.

    "redisCacheFamily": {
      "type": "string",
      "defaultValue": "C"
    }

### redisCacheCapacity

The size of the new Azure Redis Cache instance. 

    "redisCacheCapacity": {
      "type": "int",
      "allowedValues": [ 0, 1, 2, 3, 4, 5, 6 ],
      "defaultValue": 1
    }

The template defines the values that are permitted for this parameter (0, 1, 2, 3, 4, 5 or 6), and assigns a default value (1) if no value is specified. Those numbers correspond to following cache sizes: 
0 = 250 MB, 1 = 1 GB, 2 = 2.5 GB, 3 = 6 GB, 4 = 13 GB, 5 = 26 GB, 6 = 53 GB

### redisCacheVersion

The Redis server version of the new cache.

    "redisCacheVersion": {
      "type": "string",
      "defaultValue": "2.8"
    }


### redisCacheLocation
The location of the Redics Cache. For best perfomance, use the same location as the app to be used with the cache.

    "redisCacheLocation": {
      "type": "string"
    }

### diagnosticsStorageAccountName
The name of the existing storage account to use for diagnostics. 

    "diagnosticsStorageAccountName": {
      "type": "string"
    }

### enableNonSslPort
A boolean value that indicates whether to allow access via non-SSL ports.

    "enableNonSslPort": {
      "type": "bool"
    }

### diagnosticsStatus
A value that indicates whether diagnostices is enabled. Use ON or OFF.

    "diagnosticsStatus": {
      "type": "string",
      "defaultValue": "ON",
      "allowedValues": [
            "ON",
            "OFF"
        ]
    }

## Resources to deploy
### Redis Cache
Creates the Azure Redis Cache.

    {
      "apiVersion": "2015-08-01",
      "name": "[parameters('redisCacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[parameters('redisCacheLocation')]",
      "properties": {
        "enableNonSslPort": "[parameters('enableNonSslPort')]",
        "redisVersion": "[parameters('redisCacheVersion')]",
        "sku": {
          "capacity": "[parameters('redisCacheCapacity')]",
          "family": "[parameters('redisCacheFamily')]",
          "name": "[parameters('redisCacheSKU')]"
        }
      },
        "resources": [
          {
            "apiVersion": "2014-04-01",
            "type": "diagnosticSettings",
            "name": "service", 
            "location": "[parameters('redisCacheLocation')]",
            "dependsOn": [
              "[concat('Microsoft.Cache/Redis/', parameters('redisCacheName'))]"
            ],
            "properties": {
              "status": "[parameters('diagnosticsStatus')]",
              "storageAccountName": "[parameters('diagnosticsStorageAccountName')]",
              "retention": "30"
            }
          }
        ]
    }

## Commands to run deployment

To deploy the resources to Azure, you must be logged in to your Azure account and you must use the Azure Resource Manager module. To learn about using Azure Resource Manager with either Azure PowerShell or Azure CLI, 
see:

- [Using Azure PowerShell with Azure Resource Manager](../articles/powershell-azure-resource-manager.md)
- [Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management](../articles/xplat-cli-azure-resource-manager.md).

The examples below assume you already have a resource group in your account with the specified name. 


### PowerShell
    New-AzureRmResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup -redisCacheName ExampleCache -redisCacheLocation "West US"

### Azure CLI
    azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-redis-cache/azuredeploy.json -g ExampleDeployGroup


