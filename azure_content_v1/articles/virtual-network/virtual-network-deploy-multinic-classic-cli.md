<properties 
   pageTitle="Deploy multi NIC VMs using the Azure CLI in the classic deployment model | Microsoft Azure"
   description="Learn how to deploy multi NIC VMs using the Azure CLI in the classic deployment model"
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

# Deploy multi NIC VMs (classic) using the Azure CLI
> [AZURE.SELECTOR]
[PowerShell](virtual-network-deploy-multinic-classic-ps.md)
[Azure CLI](virtual-network-deploy-multinic-classic-cli.md)


You can create virtual machines (VMs) in Azure and attach multiple network interfaces (NICs) to each of your VMs. Multi NIC is a requirement for many network virtual appliances, such as application delivery and WAN optimization solutions. Multi NIC also provides more network traffic management functionality, including isolation of traffic between a front end NIC and back end NIC(s), or separation of data plane traffic from management plane traffic.

Before you can implement multi NICs in VMs, it is necessary to understand when you can use multi NICs, and how they are used. Read the [multi NIC overview](virtual-networks-multiple-nics.md) to learn more about VMs with multiple NICs.


> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the [Resource Manager model](virtual-network-deploy-multinic-arm-cli.md).

## Scenario

This document will walk through a deployment that uses multiple NICs in VMs in a specific scenario. In this scenario, you have a two-tiered IaaS workload hosted in Azure. Each tier is deployed in its own subnet in a virtual network (VNet). The front end tier is composed of several web servers, grouped together in a load balancer set for high availability. The back end tier is composed of several database servers. These database servers will be deployed with two NICs each, one for database access, the other for management. The scenario also includes Network Security Groups (NSGs) to control what traffic is allowed to each subnet, and NIC in the deployment. The figure below shows the basic architecture of this scenario.  

![MultiNIC scenario](./media/virtual-network-deploy-multinic-scenario-include/Figure1.png)



Since at this point in time you cannot have VMs with a single NIC and VMs with multiple NICs in the same cloud service, you will implement the back end servers in a different cloud service than and all other components in the scenario. The steps below use a cloud service named *IaaSStory* for the main resources, and *IaaSStory-BackEnd* for the back end servers.

## Prerequisites
Before you can deploy the back end servers, you need to deploy the main cloud service with all the necessary resources for this scenario. At minimum, you need to create a virtual network with a subnet for the backend. Visit [Create a virtual network by using the Azure CLI](virtual-networks-create-vnet-classic-cli.md) to learn how to deploy a virtual network.

## Prerequisite: Install the Azure CLI
To perform the steps in this article, you'll need to [install the Azure Command-Line Interface for Mac, Linux, and Windows (Azure CLI)](xplat-install.md) and you'll need to [log on to Azure](xplat-connect.md). 

> [AZURE.NOTE] If you don't have an Azure account, you'll need one. Go sign up for a [free trial here](sign-up-organization.md). In addition, to follow along completely you'll need to have either [jq](https://stedolan.github.io/jq/) or some other JSON parsing tool or library installed.

## Deploy the back end VMs
The backend VMs depend on the creation of the resources listed below.

* **Storage account for data disks**. For better performance, the data disks on the database servers will use solid state drive (SSD) technology, which requires a premium storage account. Make sure the Azure location you deploy to support premium storage.
* **NICs**. Each VM will have two NICs, one for database access, and one for management.
* **Availability set**. All database servers will be added to a single availability set, to ensure at least one of the VMs is up and running during maintenance. 

### Step 1 - Start your script
You can download the full bash script used [here](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/multinic.sh). Follow the steps below to change the script to work in your environment.

1. Change the values of the variables below based on your existing resource group deployed above in [Prerequisites](#Prerequisites.md).

        location="useast2"
     vnetName="WTestVNet"
     backendSubnetName="BackEnd"
2. Change the values of the variables below based on the values you want to use for your backend deployment.

        backendCSName="IaaSStory-Backend"
     prmStorageAccountName="iaasstoryprmstorage"
     image="0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1"
     avSetName="ASDB"
     vmSize="Standard_DS3"
     diskSize=127
     vmNamePrefix="DB"
     osDiskName="osdiskdb"
     dataDiskPrefix="db"
     dataDiskName="datadisk"
     ipAddressPrefix="192.168.2."
     username='adminuser'
     password='adminP@ssw0rd'
     numberOfVMs=2


### Step 2 - Create necessary resources for your VMs
1. Create a new cloud service for all backend VMs. Notice the use of the `$backendCSName` variable for the resource group name, and `$location` for the Azure region.

        azure service create --serviceName $backendCSName \
         --location $location
2. Create a premium storage account for the OS and data disks to be used by yours VMs.

        azure storage account create $prmStorageAccountName \
         --location $location \
         --type PLRS 


### Step 3 - Create VMs with multiple NICs
1. Start a loop to create multiple VMs, based on the `numberOfVMs` variables.

        for ((suffixNumber=1;suffixNumber<=numberOfVMs;suffixNumber++));
     do
2. For each VM, specify the name and IP address of each of the two NICs.

            nic1Name=$vmNamePrefix$suffixNumber-DA
         x=$((suffixNumber+3))
         ipAddress1=$ipAddressPrefix$x

         nic2Name=$vmNamePrefix$suffixNumber-RA
         x=$((suffixNumber+53))
         ipAddress2=$ipAddressPrefix$x
3. Create the VM. Notice the usage of the `--nic-config` parameter, containing a list of all NICs with name, subnet, and IP address.

            azure vm create $backendCSName $image $username $password \
             --connect $backendCSName \
             --vm-name $vmNamePrefix$suffixNumber \
             --vm-size $vmSize \
             --availability-set $avSetName \
             --blob-url $prmStorageAccountName.blob.core.windows.net/vhds/$osDiskName$suffixNumber.vhd \
             --virtual-network-name $vnetName \
             --subnet-names $backendSubnetName \
             --nic-config $nic1Name:$backendSubnetName:$ipAddress1::,$nic2Name:$backendSubnetName:$ipAddress2::
4. For each VM, create two data disks.

            azure vm disk attach-new $vmNamePrefix$suffixNumber \
             $diskSize \
             vhds/$dataDiskPrefix$suffixNumber$dataDiskName-1.vhd

         azure vm disk attach-new $vmNamePrefix$suffixNumber \
             $diskSize \
             vhds/$dataDiskPrefix$suffixNumber$dataDiskName-2.vhd
     done


### Step 4 - Run the script
Now that you downloaded and changed the script based on your needs, run the script to create the back end database VMs with multiple NICs.

1. Save your script and run it from your **Bash** terminal. You will see the initial output, as shown below.

        info:    Executing command service create
     info:    Creating cloud service
     data:    Cloud service name IaaSStory-Backend
     info:    service create command OK
     info:    Executing command storage account create
     info:    Creating storage account
     info:    storage account create command OK
     info:    Executing command vm create
     info:    Looking up image 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
     info:    Looking up virtual network
     info:    Looking up cloud service
     info:    Getting cloud service properties
     info:    Looking up deployment
     info:    Creating VM
2. After a few minutes, the execution will end and you will see the rest of the output as shown below.

        info:    OK
     info:    vm create command OK
     info:    Executing command vm disk attach-new
     info:    Getting virtual machines
     info:    Adding Data-Disk
     info:    vm disk attach-new command OK
     info:    Executing command vm disk attach-new
     info:    Getting virtual machines
     info:    Adding Data-Disk
     info:    vm disk attach-new command OK
     info:    Executing command vm create
     info:    Looking up image 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
     info:    Looking up virtual network
     info:    Looking up cloud service
     info:    Getting cloud service properties
     info:    Looking up deployment
     info:    Creating VM
     info:    OK
     info:    vm create command OK
     info:    Executing command vm disk attach-new
     info:    Getting virtual machines
     info:    Adding Data-Disk
     info:    vm disk attach-new command OK
     info:    Executing command vm disk attach-new
     info:    Getting virtual machines
     info:    Adding Data-Disk
     info:    vm disk attach-new command OK

