<properties
    pageTitle="Automatically Scale Virtual Machine Scale Sets | Microsoft Azure"
    description="Get started creating and managing your first Azure Virtual Machine Scale Sets using Azure PowerShell"
    services="virtual-machines"
    documentationCenter=""
    authors="davidmu1"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2016"
    ms.author="davidmu"/>

# Automatically scale machines in a Virtual Machine Scale Set
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Azure CLI](virtual-machines-vmss-walkthrough-cli.md)
> * [Azure PowerShell](virtual-machines-vmss-walkthrough.md)
> 
> 
<br>

> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model, which Microsoft recommends for most new deployments instead of the classic deployment model.

Virtual Machine Scale Sets make it easy for you to deploy and manage identical virtual machines as a set. Scale sets provide a highly scalable and customizable compute layer for hyperscale applications, and they support Windows platform images, Linux platform images, custom images, and extensions. For more information about scale sets, see [Virtual Machine Scale Sets](virtual-machines-vmss-overview.md).

This tutorial shows you how to create a Virtual Machine Scale Set of Windows virtual machines and automatically scale the machines in the set. You do this by creating an Azure Resource Manager template and deploying it using Azure PowerShell. For more information about templates, see [Authoring Azure Resource Manager templates](../resource-group-authoring-templates.md).

The template that you build in this tutorial is similar to a template that can be found in the template gallery. To learn more, see [Deploy a simple VM Scale Set with Windows VMs and a Jumpbox](https://azure.microsoft.com/documentation/templates/201-vmss-windows-jumpbox/).

Azure PowerShell is currently available in two releases - 1.0 and 0.9.8. If you have existing scripts and do not want to change them right now, you can continue using the 0.9.8 release. When using the 1.0 release, you should carefully test your scripts in pre-production environments before using them in production to avoid unexpected impacts.

1.0 cmdlets follow the naming pattern {verb}-AzureRm{noun}; whereas, the 0.9.8 names do not include **Rm** (for example, New-AzureRmResourceGroup instead of New-AzureResourceGroup). When using Azure PowerShell 0.9.8, you must first enable the Resource Manager mode by running the **Switch-AzureMode AzureResourceManager** command. This command is not necessary in 1.0.

New features will be added to only the 1.0 release. For information about the 1.0 release, including how to install and uninstall the release, see [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).



The Virtual Machine Scale Sets API version is 2015-06-15 and is a preview release for evaluation and testing purposes. It is recommended that for the preview release, you register with the feature before using VM Scale Sets. You do this by running the following command:

    Register-AzureRmProviderFeature -FeatureName vmssApiPreviewV2Access -ProviderNamespace Microsoft.Compute


## Step 1: Create a resource group and a storage account
1. **Sign in to Microsoft Azure**. Open the Microsoft Azure PowerShell window and run **Login-AzureRmAccount**.

2. **Create a resource group** – All resources must be deployed to a resource group. For this tutorial, name the resource group **vmsstestrg1**. See [New-AzureRmResourceGroup](https://msdn.microsoft.com/library/mt603739.aspx).

3. **Deploy a storage account into the new resource group** – This tutorial uses several storage accounts to facilitate the virtual machine scale set. Use [New-AzureRmStorageAccount](https://msdn.microsoft.com/library/mt607148.aspx) to create a storage account named **vmsstestsa**. Keep the Azure PowerShell window open for steps later in this tutorial.


## Step 2: Create the template
An Azure Resource Manager template makes it possible for you to deploy and manage Azure resources together by using a JSON description of the resources and associated deployment parameters.

1. In your favorite text editor, create the file C:\VMSSTemplate.json and add the initial JSON structure to support the template.

    ```
 {
     "$schema":"http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
     "contentVersion": "1.0.0.0",
     "parameters": {
     }
     "variables": {
     }
     "resources": [
     ]
 }
 ```
2. Parameters are not always required, but they make template management easier. They provide a way to specify values for the template, describe the type of the value, the default value if needed, and possibly the allowed values of the parameter.

    Add these parameters under the parameters parent element that you added to the template:

   * A name for the jumpbox virtual machine that is used to access the machines in the scale set.
* A name for the storage account where the template is stored.
* The number of instances of virtual machines to initially create in the scale set.
* The name and password of the administrator account on the virtual machines.
* A prefix for the resources that are created in the resource group.


    ```
    "vmName": {
        "type": "string"
    },
    "vmSSName": {
        "type": "string"
    },
    "instanceCount": {
        "type": "string"
    },
    "adminUsername": {
        "type": "string"
    },
    "adminPassword": {
        "type": "securestring"
    },
    "resourcePrefix": {
        "type": "string"
    }
    ```

1. Variables can be used in a template to specify values that may change frequently or values that need to be created from a combination of parameter values.

    Add these variables under the variables parent element that you added to the template:

   * DNS names that are used by the network interfaces.
* The size of the virtual machines used in the scale set. For more information about virtual machine sizes see, [Sizes for virtual machines](virtual-machines-size-specs.md).
* The platform image information for defining the operating system that will run on the virtual machines in the scale set. For more information about selecting images, see [Navigate and select Azure virtual machine images with Windows PowerShell and the Azure CLI](resource-groups-vm-searching.md).
* The IP address names and prefixes for the virtual network and subnets.
* The names and identifiers of the virtual network, load balancer, and network interfaces.
* Storage account names for the accounts associated with the machines in the scale set.
* Settings for the Diagnostics extension that is installed on the virtual machines. For more information about the Diagnostics extension, see [Create a Windows Virtual machine with monitoring and diagnostics using Azure Resource Manager Template](virtual-machines-extensions-diagnostics-windows-template.md).

  ```
"apiVersion": "2015-06-15"
"dnsName1": "[concat(parameters('resourcePrefix'),'dn1')] ",
"dnsName2": "[concat(parameters('resourcePrefix'),'dn2')] ",
"vmSize": "Standard_A0",
"imagePublisher": "MicrosoftWindowsServer",
"imageOffer": "WindowsServer",
"imageVersion": "2012-R2-Datacenter",
"addressPrefix": "10.0.0.0/16",
"subnetName": "Subnet",
"subnetPrefix": "10.0.0.0/24",
"publicIP1": "[concat(parameters('resourcePrefix'),'ip1')]",
"publicIP2": "[concat(parameters('resourcePrefix'),'ip2')]",
"loadBalancerName": "[concat(parameters('resourcePrefix'),'lb1')]",
"virtualNetworkName": "[concat(parameters('resourcePrefix'),'vn1')]",
"nicName1": "[concat(parameters('resourcePrefix'),'nc1')]",
"nicName2": "[concat(parameters('resourcePrefix'),'nc2')]",
"vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
"publicIPAddressID1": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIP1'))]",
"publicIPAddressID2": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIP2'))]",
"lbID": "[resourceId('Microsoft.Network/loadBalancers',variables('loadBalancerName'))]",
"nicId": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName2'))]",
"frontEndIPConfigID": "[concat(variables('lbID'),'/frontendIPConfigurations/loadBalancerFrontEnd')]",
"storageAccountType": "Standard_LRS",
"storageAccountSuffix": [ "a", "g", "m", "s", "y" ],
"diagnosticsStorageAccountName": "[concat(parameters('resourcePrefix'), 'saa')]",
"accountid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/','Microsoft.Storage/storageAccounts/', variables('diagnosticsStorageAccountName'))]",
"wadlogs": "<WadCfg> <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" /></WindowsEventLog>",
"wadperfcounter": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\"><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Processor Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU utilization\" locale=\"en-us\"/></PerformanceCounterConfiguration>",
"wadcfgxstart": "[concat(variables('wadlogs'),variables('wadperfcounter'),'<Metrics resourceId=\"')]",
"wadmetricsresourceid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name ,'/providers/','Microsoft.Compute/virtualMachineScaleSets/',parameters('vmssName'))]",
"wadcfgxend": "[concat('\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/><MetricAggregation scheduledTransferPeriod=\"PT1M\"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>')]"
```

2. In this tutorial, you deploy the following resources and extensions:

   * Microsoft.Storage/storageAccounts
* Microsoft.Network/virtualNetworks
* Microsoft.Network/publicIPAddresses
* Microsoft.Network/loadBalancers
* Microsoft.Network/networkInterfaces
* Microsoft.Compute/virtualMachines
* Microsoft.Compute/virtualMachineScaleSets
* Microsoft.Insights.VMDiagnosticsSettings
* Microsoft.Insights/autoscaleSettings

  For more information about Resource Manager resources, see [Azure Compute, Network, and Storage Providers under the Azure Resource Manager](virtual-machines-azurerm-versus-azuresm.md).

  Add the storage account resource under the resources parent element that you added to the template. This template uses a loop to create the recommended 5 storage accounts where the operating system disks and diagnostic data are stored. This set of accounts can support up to 100 virtual machines in a scale set, which is the current maximum. Each storage account is named with a letter designator that was defined in the variables combined with the suffix that you provide in the parameters for the template.

  ```
{
   "type": "Microsoft.Storage/storageAccounts",
   "name": "[concat(variables('resourcePrefix'), parameters('storageAccountSuffix')[copyIndex()])]",
   "apiVersion": "2015-05-01-preview",
   "copy": {
       "name": "storageLoop",
       "count": 5
   },
   "location": "[resourceGroup().location]",
   "properties": {
       "accountType": "[variables('storageAccountType')]"
   }
},
```

3. Add the virtual network resource. For more information, see [Network Resource Provider](../virtual-network/resource-groups-networking.md).

    ```
 {
     "apiVersion": "[variables('apiVersion')]",
     "type": "Microsoft.Network/virtualNetworks",
     "name": "[variables('virtualNetworkName')]",
     "location": "[resourceGroup().location]",
     "properties": {
         "addressSpace": {
             "addressPrefixes": [
                 "[variables('addressPrefix')]"
             ]
         },
         "subnets": [
             {
                 "name": "[variables('subnetName')]",
                 "properties": {
                     "addressPrefix": "[variables('subnetPrefix')]"
                 }
             }
         ]
     }
 },
 ```
4. Add the public IP address resources that are used by the load balancer and network interface.

    ```
 {
     "apiVersion": "[variables('apiVersion')]",
     "type": "Microsoft.Network/publicIPAddresses",
     "name": "[variables('publicIP1')]",
     "location": "[resourceGroup().location]",
     "properties": {
         "publicIPAllocationMethod": "Dynamic",
         "dnsSettings": {
             "domainNameLabel": "[variables('dnsName1')]"
         }
     }
 },
 {
     "apiVersion": "[variables('apiVersion')]",
     "type": "Microsoft.Network/publicIPAddresses",
     "name": "[variables('publicIP2')]",
     "location": "[resourceGroup().location]",
     "properties": {
         "publicIPAllocationMethod": "Dynamic",
         "dnsSettings": {
             "domainNameLabel": "[variables('dnsName2')]"
         }
     }
 },
 ```
5. Add the load balancer resource that is used by the scale set. For more information, see [Azure Resource Manager Support for Load Balancer](../load-balancer/load-balancer-arm.md).

    ```
 {
     "apiVersion": "[variables('apiVersion')]",
     "name": "[variables('loadBalancerName')]",
     "type": "Microsoft.Network/loadBalancers",
     "location": "[resourceGroup().location]",
     "dependsOn": [
         "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIP1'))]"
     ],
     "properties": {
         "frontendIPConfigurations": [
             {
                 "name": "loadBalancerFrontEnd",
                 "properties": {
                     "publicIPAddress": {
                         "id": "[variables('publicIPAddressID1')]"
                     }
                 }
             }
         ],
         "backendAddressPools": [
             {
                 "name": "bepool1"
             }
         ],
         "inboundNatPools": [
             {
                 "name": "natpool1",
                 "properties": {
                     "frontendIPConfiguration": {
                         "id": "[variables('frontEndIPConfigID')]"
                     },
                     "protocol": "tcp",
                     "frontendPortRangeStart": 50000,
                     "frontendPortRangeEnd": 50500,
                     "backendPort": 3389
                 }
             }
         ]
     }
 },
 ```
6. Add the network interface resource that is used by the jumpbox virtual machine. Because machines in a virtual machine scale set are not directly accessible using a public IP address, a jumpbox virtual machine is created in the same virtual network as the scale set and is used to remotely access the machines in the set.


    ```
    {
        "apiVersion": "2015-05-01-preview",
        "type": "Microsoft.Network/networkInterfaces",
        "name": "[variables('nicName1')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIP2'))]",
            "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
        ],
        "properties": {
            "ipConfigurations": [
                {
                    "name": "ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod": "Dynamic",
                        "publicIPAddress": {
                            "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIP2'))]"
                        },
                        "subnet": {
                            "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/virtualNetworks/',variables('virtualNetworkName'),'/subnets/',variables('subnetName'))]"
                        }
                    }
                }
            ]
        }
    },
    ```


1. Add the virtual machine in the same network as the scale set.

    ```
 {
     "apiVersion": "2015-05-01-preview",
     "type": "Microsoft.Compute/virtualMachines",
     "name": "[parameters('vmName')]",
     "location": "[resourceGroup().location]",
   "dependsOn": [
         "[concat('Microsoft.Network/networkInterfaces/', variables('nicName1'))]"
     ],
     "properties": {
         "hardwareProfile": {
             "vmSize": "[variables('vmSize')]"
         },
         "osProfile": {
             "computername": "[parameters('vmName')]",
             "adminUsername": "[parameters('adminUsername')]",
             "adminPassword": "[parameters('adminPassword')]"
         },
         "storageProfile": {
             "imageReference": {
                 "publisher": "[variables('imagePublisher')]",
                 "offer": "[variables('imageOffer')]",
                 "sku": "[variables('imageVersion')]",
                 "version": "latest"
             },
             "osDisk": {
                 "name": "osdisk1",
                 "vhd": {
                     "uri":  "[concat('https://',parameters('resourcePrefix'),'saa.blob.core.windows.net/vhds/',parameters('resourcePrefix'),'osdisk1.vhd')]"
                 },
                 "caching": "ReadWrite",
                 "createOption": "FromImage"        
             }
         },
         "networkProfile": {
             "networkInterfaces": [
                 {
                     "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName1'))]"
                 }
             ]
         }
     }
 },
 ```
2. Add the virtual machine scale set and specify the Diagnostics extension that is installed on all virtual machines in the scale set. Many of the settings for this resource are similar with the virtual machine resource. Here are the main differences:

   * **capacity** - specifies how many virtual machines should be initialized in the scale set. You set this value by specifying a value for the instanceCount parameter.

* **upgradePolicy** – specifies how updates are made to virtual machines in the scale set. Manual specifies that only new virtual machines are affected by changes in a template when it is redeployed. Automatic specifies that all machines in the scale set are updated and restarted.


   The virtual machine scale set is not created until all of the storage accounts are created as specified with the dependsOn element.

   ```
{
 "type": "Microsoft.Compute/virtualMachineScaleSets",
 "apiVersion": "[variables('apiVersion')]",
 "name": "[parameters('vmSSName')]",
 "location": "[resourceGroup().location]",
 "dependsOn": [
     "storageLoop",
     "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]",
     "[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]"
 ],
 "sku": {
     "name": "[variables('vmSize')]",
     "tier": "Standard",
     "capacity": "[parameters('instanceCount')]"
 },
 "properties": {
     "upgradePolicy": {
         "mode": "Manual"
     },
     "virtualMachineProfile": {
         "storageProfile": {
             "osDisk": {
                 "vhdContainers": [
                     "[concat('https://', parameters('resourcePrefix'), 'saa.blob.core.windows.net/vmss')]",
                     "[concat('https://', parameters('resourcePrefix'), 'sag.blob.core.windows.net/vmss')]",
                     "[concat('https://', parameters('resourcePrefix'), 'sam.blob.core.windows.net/vmss')]",
                     "[concat('https://', parameters('resourcePrefix'), 'sas.blob.core.windows.net/vmss')]",
                     "[concat('https://', parameters('resourcePrefix'), 'say.blob.core.windows.net/vmss')]"
                 ],
                 "name": "vmssosdisk",
                 "caching": "ReadOnly",
                 "createOption": "FromImage"
             },
             "imageReference": {
                 "publisher": "[variables('imagePublisher')]",
                 "offer": "[variables('imageOffer')]",
                 "sku": "[variables('imageVersion')]",
                 "version": "latest"
             }
         },
         "osProfile": {
             "computerNamePrefix": "[parameters('vmSSName')]",
             "adminUsername": "[parameters('adminUsername')]",
             "adminPassword": "[parameters('adminPassword')]"
         },
         "networkProfile": {
             "networkInterfaceConfigurations": [
                 {
                     "name": "[variables('nicName2')]",
                     "properties": {
                         "primary": "true",
                         "ipConfigurations": [
                             {
                                 "name": "ip1",
                                 "properties": {
                                     "subnet": {
                                         "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/virtualNetworks/',variables('virtualNetworkName'),'/subnets/',variables('subnetName'))]"
                                     },
                                     "loadBalancerBackendAddressPools": [
                                         {
                                             "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/loadBalancers/',variables('loadBalancerName'),'/backendAddressPools/bepool1')]"
                                         }
                                     ],
                                     "loadBalancerInboundNatPools": [
                                         {
                                             "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/loadBalancers/',variables('loadBalancerName'),'/inboundNatPools/natpool1')]"
                                         }
                                     ]
                                 }
                             }
                         ]
                     }
                 }
             ]
         },
         "extensionProfile": {
             "extensions": [
                 {
                     "name": "Microsoft.Insights.VMDiagnosticsSettings",
                     "properties": {
                         "publisher": "Microsoft.Azure.Diagnostics",
                         "type": "IaaSDiagnostics",
                         "typeHandlerVersion": "1.5",
                         "autoUpgradeMinorVersion": true,
                         "settings": {
                             "xmlCfg": "[base64(concat(variables('wadcfgxstart'),variables('wadmetricsresourceid'),variables('wadcfgxend')))]",
                             "storageAccount": "[variables('diagnosticsStorageAccountName')]"
                         },
                         "protectedSettings": {
                             "storageAccountName": "[variables('diagnosticsStorageAccountName')]",
                             "storageAccountKey": "[listkeys(variables('accountid'), variables('apiVersion')).key1]",
                             "storageAccountEndPoint": "https://core.windows.net"
                         }
                     }
                 }
             ]
         }
     }
 }
},
```
3. Add the autoscaleSettings resource that defines how the scale set adjusts based on the processor usage on the machines in the set. For this tutorial, these are the important values:

   * **metricName** - This is the same as the performance counter that we defined in the wadperfcounter variable. Using that variable, the Diagnostics extension collects the  **Processor(_Total)\% Processor Time** counter.
* **metricResourceUri** - This is the resource identifier of the virtual machine scale set.
* **timeGrain** – This is the granularity of the metrics that are collected. In this template, it is set to 1 minute.
* **statistic** – This determines how the metrics are combined to accommodate the automatic scaling action. The possible values are: Average, Min, Max. In this template we are looking for the average total CPU usage among the virtual machines in the scale set.
* **timeWindow** – This if the range of time in which instance data is collected. It must be between 5 minutes and 12 hours.
* **timeAggregation** –This determines how the data that is collected should be combined over time. The default value is Average. The possible values are: Average, Minimum, Maximum, Last, Total, Count.
* **operator** – This is the operator that is used to compare the metric data and the threshold. The possible values are: Equals, NotEquals, GreaterThan, GreaterThanOrEqual, LessThan, LessThanOrEqual.
* **threshold** – This is the value that triggers the scale action. In this template, machines are added to the scale set when the average CPU usage among machines in the set is over 50%.
* **direction** – This determines the action that is taken when the threshold value is achieved. The possible values are Increase or Decrease. In this template, the number of virtual machines in the scale set is increased if the threshold is over 50% in the defined time window.
* **type** – This is the type of action that should occur, this must be set to ChangeCount.
* **value** – This is the number of virtual machines that are added or removed from the scale set. This value must be 1 or greater. The default value is 1. In this template, the number of machines in the scale set increases by 1 when the threshold is met.
* **cooldown** – This is the amount of time to wait since the last scaling action before the next action occurs. This must be between 1 minute and I week.

   ```
{
 "type": "Microsoft.Insights/autoscaleSettings",
 "apiVersion": "2015-04-01",
 "name": "[concat(parameters('resourcePrefix'),'as1')]",
 "location": "[resourceGroup().location]",
 "dependsOn": [
     "[concat('Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]"
 ],
 "properties": {
     "enabled": true,
     "name": "[concat(parameters('resourcePrefix'),'as1')]",
     "profiles": [
         {
             "name": "Profile1",
             "capacity": {
                 "minimum": "1",
                 "maximum": "10",
                 "default": "1"
             },
             "rules": [
                 {
                     "metricTrigger": {
                         "metricName": "\\Processor(_Total)\\% Processor Time",
                         "metricNamespace": "",
                         "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]",
                         "timeGrain": "PT1M",
                         "statistic": "Average",
                         "timeWindow": "PT5M",
                         "timeAggregation": "Average",
                         "operator": "GreaterThan",
                         "threshold": 50.0
                     },
                     "scaleAction": {
                         "direction": "Increase",
                         "type": "ChangeCount",
                         "value": "1",
                         "cooldown": "PT5M"
                     }
                 }
             ]
         }
     ],
     "targetResourceUri": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]"
 }
}
```
4. Save the template file.    


## Step 3: Upload the template to storage
The template can be uploaded from the Microsoft Azure PowerShell window as long as you know the account name and the primary key of the storage account that you created in step 1.

1. In the Microsoft Azure PowerShell window, set a variable that specifies the name of the storage account that you deployed in step 1.

     $StorageAccountName = "vmstestsa"

2. Set a variable that specifies the primary key of the storage account.

     $StorageAccountKey = "<primary-account-key>"

   You can get this key by clicking the key icon when viewing the storage account resource in the Azure portal.

3. Create the storage account context object that is used to validate operations with the storage account.

     $ctx = New-AzureStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

4. Create a new templates container where the template that you created can be stored.

     $ContainerName = "templates"
  New-AzureStorageContainer -Name $ContainerName -Context $ctx  -Permission Blob

5. Upload the template file to the new container.

     $BlobName = "VMSSTemplate.json"
  $fileName = "C:\" + $BlobName
  Set-AzureStorageBlobContent -File $fileName -Container $ContainerName -Blob  $BlobName -Context $ctx


## Step 4: Deploy the template
Now that you created the template, you can start deploying the resources. Use this command to start the process:

        New-AzureRmResourceGroupDeployment -Name "vmsstestdp1" -ResourceGroupName "vmsstestrg1" -TemplateUri "https://vmsstestsa.blob.core.windows.net/templates/VMSSTemplate.json"

When you press enter, you are prompted to provide values for the variables you assigned. Provide these values:

    vmName: vmsstestvm1
    vmSSName: vmsstest1
    instanceCount: 5
    adminUserName: vmadmin1
    adminPassword: VMpass1
    resourcePrefix: vmsstest

It should take about 15 minutes for all of the resources to successfully be deployed.

> [!NOTE]
> You can also make use of the portal’s ability to deploy the resources. To do this, use this link:
> https://portal.azure.com/#create/Microsoft.Template/uri/<link to VM Scale Set JSON template>
> 
> 
## Step 5: Monitor resources
You can get some information about virtual machine scale sets using these methods:

* The Azure portal - You can currently get a limited amount of information using the portal.
* The [Azure Resource Explorer](https://resources.azure.com/) - This is the best tool to explore the current state of your scale set. Follow this path and you should see the instance view of the scale set that you created:

       subscriptions > {your subscription} > resourceGroups > vmsstestrg1 > providers > Microsoft.Compute > virtualMachineScaleSets > vmsstest1 > virtualMachines
* Azure PowerShell - Use this command to get some information:

       Get-AzureRmResource -name vmsstest1 -ResourceGroupName vmsstestrg1 -ResourceType Microsoft.Compute/virtualMachineScaleSets -ApiVersion 2015-06-15
* Connect to the jumpbox virtual machine just like you would any other machine and then you can remotely access the virtual machines in the scale set to monitor individual processes.


> [!NOTE]
> A complete REST API for obtaining information about scale sets can be found in [Virtual Machine Scale Sets](https://msdn.microsoft.com/library/mt589023.aspx)
> 
> 
## Step 6: Remove the resources
Because you are charged for resources used in Azure, it is always a good practice to delete resources that are no longer needed. You don’t need to delete each resource separately from a resource group. You can delete the resource group and all of its resources will automatically be deleted.

    Remove-AzureRmResourceGroup -Name vmsstestrg1

If you want to keep your resource group, you can delete the scale set only.

    Remove-AzureRmResource -Name vmsstest1 -ResourceGroupName vmsstestrg1 -ApiVersion 2015-06-15 -ResourceType Microsoft.Compute/virtualMachineScaleSets
