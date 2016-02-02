<properties 
   pageTitle="Deploy a VM with a static public IP using the Azure CLI in Resource Manager | Microsoft Azure"
   description="Learn how to deploy VMs with a static public IP using the Azure CLI in Resource Manager"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>

<tags  
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/15/2015"
   ms.author="telmos" />

# Deploy a VM with a static public IP using the Azure CLI
> [AZURE.SELECTOR]
[Azure Portal](virtual-network-deploy-static-pip-arm-portal.md)
[PowerShell](virtual-network-deploy-static-pip-arm-ps.md)
[Azure CLI](virtual-network-deploy-static-pip-arm-cli.md)
[Template](virtual-network-deploy-static-pip-arm-template.md)


You can create virtual machines (VMs) in Azure and expose them to the public Internet by using a public IP address. By default, Public IPs are dynamic and the address associated to them may change when the VM is deleted. To guarantee that the VM always uses the same public IP address, you need to create a static Public IP. 

Before you can implement static Public IPs in VMs, it is necessary to understand when you can use static Public IPs, and how they are used. Read the [IP addressing overview](virtual-network-ip-addresses-overview-arm.md) to learn more about IP addressing in Azure.



> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model, which Microsoft recommends for most new deployments instead of the classic deployment model.

## Scenario

This document will walk through a deployment that uses a static public IP address allocated to a virtual machine (VM). In this scenario, you have a single VM with its own static public IP address. The VM is part of a subnet named **FrontEnd** and also has a static private IP address (**192.168.1.101**) in that subnet.

You may need a static IP address for web servers that require SSL connections in which the SSL certificate is linked to an IP address. 

![IMAGE DESCRIPTION](./media/virtual-network-deploy-static-pip-scenario-include/figure1.png)

You can follow the steps below to deploy the environment shown in the figure above.


## Prerequisite: Install the Azure CLI
To perform the steps in this article, you'll need to [install the Azure Command-Line Interface for Mac, Linux, and Windows (Azure CLI)](xplat-install.md) and you'll need to [log on to Azure](xplat-connect.md). 

> [AZURE.NOTE] If you don't have an Azure account, you'll need one. Go sign up for a [free trial here](sign-up-organization.md). In addition, to follow along completely you'll need to have either [jq](https://stedolan.github.io/jq/) or some other JSON parsing tool or library installed.

## Step 1 - Start your script
You can download the full bash script used [here](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/virtual-network-deploy-static-pip-arm-cli.sh). Follow the steps below to change the script to work in your environment.

1. Change the values of the variables below based on the values you want to use for your deployment. The values below map to the scenario used in this document.

        # Set variables for the new resource group
     rgName="IaaSStory"
     location="westus"

     # Set variables for VNet
     vnetName="TestVNet"
     vnetPrefix="192.168.0.0/16"
     subnetName="FrontEnd"
     subnetPrefix="192.168.1.0/24"

     # Set variables for storage
     stdStorageAccountName="iaasstorystorage"

     # Set variables for VM
     vmSize="Standard_A1"
     diskSize=127
     publisher="Canonical"
     offer="UbuntuServer"
     sku="14.04.2-LTS"
     version="latest"
     vmName="WEB1"
     osDiskName="osdisk"
     nicName="NICWEB1"
     privateIPAddress="192.168.1.101"
     username='adminuser'
     password='adminP@ssw0rd'
     pipName="PIPWEB1"
     dnsName="iaasstoryws1"


## Step 2 - Create the necessary resources for your VM
Before creating a VM, you need a resource group, VNet, public IP, and NIC to be used by the VM.

1. Create a new resource group.

        azure group create $rgName $location
2. Create the VNet and subnet.

        azure network vnet create --resource-group $rgName \
         --name $vnetName \
         --address-prefixes $vnetPrefix \
         --location $location
     azure network vnet subnet create --resource-group $rgName \
         --vnet-name $vnetName \
         --name $subnetName \
         --address-prefix $subnetPrefix
3. Create the public IP resource. 

        azure network public-ip create --resource-group $rgName \
         --name $pipName \
         --location $location \
         --allocation-method Static \
         --domain-name-label $dnsName 
4. Create the network interface (NIC) for the VM in the subnet created above, with the public IP. Notice the first set of commands are used to retrieve the **Id** of the subnet created above.

        subnetId="$(azure network vnet subnet show --resource-group $rgName \
                     --vnet-name $vnetName \
                     --name $subnetName|grep Id)"

     subnetId=${subnetId#*/}

     azure network nic create --name $nicName \
         --resource-group $rgName \
         --location $location \
         --private-ip-address $privateIPAddress \
         --subnet-id $subnetId \
         --public-ip-name $pipName


> [!TIP]
> The first command above uses [grep](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_04_02.html) and [string manipulation](http://tldp.org/LDP/abs/html/string-manipulation.html) (more specifically, substring removal). 
> 
> 
1. Create a storage account to host the VM OS drive.

        azure storage account create $stdStorageAccountName \
         --resource-group $rgName \
         --location $location --type LRS 


## Step 3 - Create the VM
Now that all necessary resources are in place, you can create a new VM.

1. Create the VM.

        azure vm create --resource-group $rgName \
         --name $vmName \
         --location $location \
         --vm-size $vmSize \
         --subnet-id $subnetId \
         --nic-names $nicName \
         --os-type linux \
         --image-urn $publisher:$offer:$sku:$version \
         --storage-account-name $stdStorageAccountName \
         --storage-account-container-name vhds \
         --os-disk-vhd $osDiskName.vhd \
         --admin-username $username \
         --admin-password $password
2. Save the script file.


## Step 4 - Run the script
After making any necessary changes, and understanding the script show above, run the script. 

1. From a bash console, run the script above.

        sh myscript.sh
2. The output below should be displayed after a few minutes.

        info:    Executing command group create
     info:    Getting resource group IaaSStory
     info:    Creating resource group IaaSStory
     info:    Created resource group IaaSStory
     data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory
     data:    Name:                IaaSStory
     data:    Location:            westus
     data:    Provisioning State:  Succeeded
     data:    Tags: null
     data:
     info:    group create command OK
     info:    Executing command network vnet create
     info:    Looking up virtual network "TestVNet"
     info:    Creating virtual network "TestVNet"
     info:    Loading virtual network state
     data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/TestVNet
     data:    Name                            : TestVNet
     data:    Type                            : Microsoft.Network/virtualNetworks
     data:    Location                        : westus
     data:    ProvisioningState               : Succeeded
     data:    Address prefixes:
     data:      192.168.0.0/16
     info:    network vnet create command OK
     info:    Executing command network vnet subnet create
     info:    Looking up the subnet "FrontEnd"
     info:    Creating subnet "FrontEnd"
     info:    Looking up the subnet "FrontEnd"
     data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
     data:    Type                            : Microsoft.Network/virtualNetworks/subnets
     data:    ProvisioningState               : Succeeded
     data:    Name                            : FrontEnd
     data:    Address prefix                  : 192.168.1.0/24
     data:
     info:    network vnet subnet create command OK
     info:    Executing command network public-ip create
     info:    Looking up the public ip "PIPWEB1"
     info:    Creating public ip address "PIPWEB1"
     info:    Looking up the public ip "PIPWEB1"
     data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/publicIPAddresses/PIPWEB1
     data:    Name                            : PIPWEB1
     data:    Type                            : Microsoft.Network/publicIPAddresses
     data:    Location                        : westus
     data:    Provisioning state              : Succeeded
     data:    Allocation method               : Static
     data:    Idle timeout                    : 4
     data:    IP Address                      : 40.78.63.253
     data:    Domain name label               : iaasstoryws1
     data:    FQDN                            : iaasstoryws1.westus.cloudapp.azure.com
     info:    network public-ip create command OK
     info:    Executing command network nic create
     info:    Looking up the network interface "NICWEB1"
     info:    Looking up the public ip "PIPWEB1"
     info:    Creating network interface "NICWEB1"
     info:    Looking up the network interface "NICWEB1"
     data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/networkInterfaces/NICWEB1
     data:    Name                            : NICWEB1
     data:    Type                            : Microsoft.Network/networkInterfaces
     data:    Location                        : westus
     data:    Provisioning state              : Succeeded
     data:    Enable IP forwarding            : false
     data:    IP configurations:
     data:      Name                          : NIC-config
     data:      Provisioning state            : Succeeded
     data:      Public IP address             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/publicIPAddresses/PIPWEB1
     data:      Private IP address            : 192.168.1.101
     data:      Private IP Allocation Method  : Static
     data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory2/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
     data:
     info:    network nic create command OK
     info:    Executing command storage account create
     info:    Creating storage account
     info:    storage account create command OK
     info:    Executing command vm create
     info:    Looking up the VM "WEB1"
     info:    Using the VM Size "Standard_A1"
     info:    The [OS, Data] Disk or image configuration requires storage account
     info:    Looking up the storage account iaasstorystorage
     info:    Looking up the NIC "NICWEB1"
     info:    Creating VM "WEB1"
     info:    vm create command OK

