<properties 
   pageTitle="Deploy multi NIC VMs using PowerShell in the classic deployment model | Microsoft Azure"
   description="Learn how to deploy multi NIC VMs using PowerShell in the classic deployment model"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-service-management"
/>

<tags  
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/12/2015"
   ms.author="telmos" />

# Deploy multi NIC VMs (classic) using PowerShell
> [AZURE.SELECTOR]
[PowerShell](virtual-network-deploy-multinic-classic-ps.md)
[Azure CLI](virtual-network-deploy-multinic-classic-cli.md)


You can create virtual machines (VMs) in Azure and attach multiple network interfaces (NICs) to each of your VMs. Multi NIC is a requirement for many network virtual appliances, such as application delivery and WAN optimization solutions. Multi NIC also provides more network traffic management functionality, including isolation of traffic between a front end NIC and back end NIC(s), or separation of data plane traffic from management plane traffic.

Before you can implement multi NICs in VMs, it is necessary to understand when you can use multi NICs, and how they are used. Read the [multi NIC overview](virtual-networks-multiple-nics.md) to learn more about VMs with multiple NICs.


> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the [Resource Manager model](virtual-network-deploy-multinic-arm-ps.md).

## Scenario

This document will walk through a deployment that uses multiple NICs in VMs in a specific scenario. In this scenario, you have a two-tiered IaaS workload hosted in Azure. Each tier is deployed in its own subnet in a virtual network (VNet). The front end tier is composed of several web servers, grouped together in a load balancer set for high availability. The back end tier is composed of several database servers. These database servers will be deployed with two NICs each, one for database access, the other for management. The scenario also includes Network Security Groups (NSGs) to control what traffic is allowed to each subnet, and NIC in the deployment. The figure below shows the basic architecture of this scenario.  

![MultiNIC scenario](./media/virtual-network-deploy-multinic-scenario-include/Figure1.png)



Since at this point in time you cannot have VMs with a single NIC and VMs with multiple NICs in the same cloud service, you will implement the back end servers in a different cloud service than and all other components in the scenario. The steps below use a cloud service named *IaaSStory* for the main resources, and *IaaSStory-BackEnd* for the back end servers.

## Prerequisites
Before you can deploy the back end servers, you need to deploy the main cloud service with all the necessary resources for this scenario. At minimum, you need to create a virtual network with a subnet for the backend. Visit [Create a virtual network by using PowerShell](virtual-networks-create-vnet-classic-ps.md) to learn how to deploy a virtual network.

## Prerequisite: Install the Azure PowerShell Module
To perform the steps in this article, you'll need to [to install and configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

> [AZURE.NOTE] If you don't have an Azure account, you'll need one. Go sign up for a [free trial here](sign-up-organization.md). 

## Deploy the back end VMs
The backend VMs depend on the creation of the resources listed below.

* **Backend subnet**. The database servers will be part of a separate subnet, to segregate traffic. The script below expects this subnet to exist in a vnet named *WTestVnet*.
* **Storage account for data disks**. For better performance, the data disks on the database servers will use solid state drive (SSD) technology, which requires a premium storage account. Make sure the Azure location you deploy to support premium storage.
* **Availability set**. All database servers will be added to a single availability set, to ensure at least one of the VMs is up and running during maintenance. 

### Step 1 - Start your script
You can download the full PowerShell script used [here](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/multinic.ps1). Follow the steps below to change the script to work in your environment.

1. Change the values of the variables below based on your existing resource group deployed above in [Prerequisites](#Prerequisites.md).

        $location              = "West US"
     $vnetName              = "WTestVNet"
     $backendSubnetName     = "BackEnd"
2. Change the values of the variables below based on the values you want to use for your backend deployment.

        $backendCSName         = "IaaSStory-Backend"
     $prmStorageAccountName = "iaasstoryprmstorage"
     $avSetName             = "ASDB"
     $vmSize                = "Standard_DS3"
     $diskSize              = 127
     $vmNamePrefix          = "DB"
     $dataDiskSuffix        = "datadisk"
     $ipAddressPrefix       = "192.168.2."
     $numberOfVMs           = 2


### Step 2 - Create necessary resources for your VMs
You need to create a new cloud service, and a storage account for the data disks for all VMs. You also need to specify an image, and a local administrator account for the VMs. To create these resources, execute the following steps.

1. Create a new cloud service.

        New-AzureService -ServiceName $backendCSName -Location $location
2. Create a new premium storage account.

        New-AzureStorageAccount -StorageAccountName $prmStorageAccountName `
         -Location $location `
         -Type Premium_LRS
3. Set the storage account created above as the current storage account for your subscription.

        $subscription = Get-AzureSubscription `
         | where {$_.IsCurrent -eq $true}  
     Set-AzureSubscription -SubscriptionName $subscription.SubscriptionName `
         -CurrentStorageAccountName $prmStorageAccountName
4. Select an image for the VM.

        $image = Get-AzureVMImage `
         | where{$_.ImageFamily -eq "SQL Server 2014 RTM Web on Windows Server 2012 R2"} `
         | sort PublishedDate -Descending `
         | select -ExpandProperty ImageName -First 1
5. Set the local administrator account credentials.

        $cred = Get-Credential -Message "Enter username and password for local admin account"


### Step 3 - Create VMs
You need to use a loop to create as many VMs as you want, and create the necessary NICs and VMs within the loop. To create the NICs and VMs, execute the following steps.

1. Start a `for` loop to repeat the commands to create a VM and two NICs as many times as necessary, based on the value of the `$numberOfVMs` variable.

        for ($suffixNumber = 1; $suffixNumber -le $numberOfVMs; $suffixNumber++){
2. Create a `VMConfig` object specifying the image, size, and availability set for the VM.

            $vmName = $vmNamePrefix + $suffixNumber
         $vmConfig = New-AzureVMConfig -Name $vmName `
                         -ImageName $image `
                         -InstanceSize $vmSize `
                         -AvailabilitySetName $avSetName  
3. Provision the VM as a Windows VM.

            Add-AzureProvisioningConfig -VM $vmConfig -Windows `
             -AdminUsername $cred.UserName `
             -Password $cred.Password
4. Set the default NIC and assign it a static IP address.

            Set-AzureSubnet -SubnetNames $backendSubnetName -VM $vmConfig
         Set-AzureStaticVNetIP -IPAddress ($ipAddressPrefix+$suffixNumber+3) -VM $vmConfig
5. Add a second NIC for each VM.

            Add-AzureNetworkInterfaceConfig -Name ("RemoteAccessNIC"+$suffixNumber) `
             -SubnetName $backendSubnetName `
             -StaticVNetIPAddress ($ipAddressPrefix+(53+$suffixNumber)) `
             -VM $vmConfig 
6. Create to data disks for each VM.

            $dataDisk1Name = $vmName + "-" + $dataDiskSuffix + "-1"    
         Add-AzureDataDisk -CreateNew -VM $vmConfig `
             -DiskSizeInGB $diskSize `
             -DiskLabel $dataDisk1Name `
             -LUN 0       

         $dataDisk2Name = $vmName + "-" + $dataDiskSuffix + "-2"   
         Add-AzureDataDisk -CreateNew -VM $vmConfig `
             -DiskSizeInGB $diskSize `
             -DiskLabel $dataDisk2Name `
             -LUN 1
7. Create each VM, and end the loop.

            New-AzureVM -VM $vmConfig `
             -ServiceName $backendCSName `
             -Location $location `
             -VNetName $vnetName
     }


### Step 4 - Run the script
Now that you downloaded and changed the script based on your needs, runt he script to create the back end database VMs with multiple NICs.

1. Save your script and run it from the **PowerShell** command prompt, or **PowerShell ISE**. You will see the initial output, as shown below.

        OperationDescription    OperationId                          OperationStatus
     --------------------    -----------                          ---------------
     New-AzureService        xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded      
     New-AzureStorageAccount xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded      

     WARNING: No deployment found in service: 'IaaSStory-Backend'.
2. Fill out the information needed in the credentials prompt and click **OK**. The output below will be displayed.

        New-AzureVM             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded
     New-AzureVM             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded 

