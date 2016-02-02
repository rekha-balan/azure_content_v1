<properties 
   pageTitle="Deploy a VM with a static public IP using PowerShell in Resource Manager | Microsoft Azure"
   description="Learn how to deploy VMs with a static public IP using PowerShell in Resource Manager"
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

# Deploy a VM with a static public IP using PowerShell
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


## Prerequisite: Install the Azure PowerShell Module
To perform the steps in this article, you'll need to [to install and configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

> [AZURE.NOTE] If you don't have an Azure account, you'll need one. Go sign up for a [free trial here](sign-up-organization.md). 

## Step 1 - Start your script
You can download the full PowerShell script used [here](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/virtual-network-deploy-static-pip-arm-ps.ps1). Follow the steps below to change the script to work in your environment.

1. Change the values of the variables below based on the values you want to use for your deployment. The values below map to the scenario used in this document.

        # Set variables resource group
     $rgName                = "IaaSStory"
     $location              = "West US"

     # Set variables for VNet
     $vnetName              = "WTestVNet"
     $vnetPrefix            = "192.168.0.0/16"
     $subnetName            = "FrontEnd"
     $subnetPrefix          = "192.168.1.0/24"

     # Set variables for storage
     $stdStorageAccountName = "iaasstorystorage"

     # Set variables for VM
     $vmSize                = "Standard_A1"
     $diskSize              = 127
     $publisher             = "MicrosoftWindowsServer"
     $offer                 = "WindowsServer"
     $sku                   = "2012-R2-Datacenter"
     $version               = "latest"
     $vmName                = "WEB1"
     $osDiskName            = "osdisk"
     $nicName               = "NICWEB1"
     $privateIPAddress      = "192.168.1.101"
     $pipName               = "PIPWEB1"
     $dnsName               = "iaasstoryws1"


## Step 2 - Create the necessary resources for your VM
Before creating a VM, you need a resource group, VNet, public IP, and NIC to be used by the VM.

1. Create a new resource group.

        New-AzureRmResourceGroup -Name $rgName -Location $location
2. Create the VNet and subnet.

        $vnet = New-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName `
         -AddressPrefix $vnetPrefix -Location $location   

     Add-AzureRmVirtualNetworkSubnetConfig -Name $subnetName `
         -VirtualNetwork $vnet -AddressPrefix $subnetPrefix

     Set-AzureRmVirtualNetwork -VirtualNetwork $vnet 
3. Create the public IP resource. 

        $pip = New-AzureRmPublicIpAddress -Name $pipName -ResourceGroupName $rgName `
         -AllocationMethod Static -DomainNameLabel $dnsName -Location $location
4. Create the network interface (NIC) for the VM in the subnet created above, with the public IP. Notice the first cmdlet retrieving the VNet from Azure, this is necessary since a **Set-AzureRmVirtualNetwork** was executed to change the existing VNet.

        $vnet = Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
     $subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $subnetName
     $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
         -Subnet $subnet -Location $location -PrivateIpAddress $privateIPAddress `
         -PublicIpAddress $pip
5. Create a storage account to host the VM OS drive.

        $stdStorageAccount = New-AzureRmStorageAccount -Name $stdStorageAccountName `
         -ResourceGroupName $rgName -Type Standard_LRS -Location $location


## Step 3 - Create the VM
Now that all necessary resources are in place, you can create a new VM.

1. Create the configuration object for the VM.

        $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize 
2. Get credentials for the VM local administrator account.

        $cred = Get-Credential -Message "Type the name and password for the local administrator account."
3. Create a VM configuration object.

        $vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName `
         -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
4. Set the operating system image for the VM.

        $vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig -PublisherName $publisher `
         -Offer $offer -Skus $sku -Version $version
5. Configure the OS disk.

        $osVhdUri = $stdStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $osDiskName + ".vhd"
     $vmConfig = Set-AzureRmVMOSDisk -VM $vmConfig -Name $osDiskName -VhdUri $osVhdUri -CreateOption fromImage
6. Add the NIC to the VM.

        $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id -Primary
7. Create the VM.

        New-AzureRmVM -VM $vmConfig -ResourceGroupName $rgName -Location $location
8. Save the script file.


## Step 4 - Run the script
After making any necessary changes, and understanding the script show above, run the script. 

1. From a PowerShell console, or PowerShell ISE, run the script above.
2. The output below should be displayed after a few minutes.

        ResourceGroupName : IaaSStory
     Location          : westus
     ProvisioningState : Succeeded
     Tags              : 
     ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory

     AddressSpace      : Microsoft.Azure.Commands.Network.Models.PSAddressSpace
     DhcpOptions       : Microsoft.Azure.Commands.Network.Models.PSDhcpOptions
     Subnets           : {FrontEnd}
     ProvisioningState : Succeeded
     AddressSpaceText  : {
                           "AddressPrefixes": [
                             "192.168.0.0/16"
                           ]
                         }
     DhcpOptionsText   : {}
     SubnetsText       : [
                           {
                             "Name": "FrontEnd",
                             "AddressPrefix": "192.168.1.0/24"
                           }
                         ]
     ResourceGroupName : IaaSStory
     Location          : westus
     ResourceGuid      : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
     Tag               : {}
     TagsTable         : 
     Name              : WTestVNet
     Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
     Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet

     AddressSpace      : Microsoft.Azure.Commands.Network.Models.PSAddressSpace
     DhcpOptions       : Microsoft.Azure.Commands.Network.Models.PSDhcpOptions
     Subnets           : {FrontEnd}
     ProvisioningState : Succeeded
     AddressSpaceText  : {
                           "AddressPrefixes": [
                             "192.168.0.0/16"
                           ]
                         }
     DhcpOptionsText   : {
                           "DnsServers": []
                         }
     SubnetsText       : [
                           {
                             "Name": "FrontEnd",
                             "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet/subnets/FrontEnd",
                             "AddressPrefix": "192.168.1.0/24",
                             "IpConfigurations": [],
                             "ProvisioningState": "Succeeded"
                           }
                         ]
     ResourceGroupName : IaaSStory
     Location          : westus
     ResourceGuid      : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
     Tag               : {}
     TagsTable         : 
     Name              : WTestVNet
     Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
     Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet

     TrackingOperationId : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
     RequestId           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
     Status              : Succeeded
     StatusCode          : OK
     Output              : 
     StartTime           : 1/12/2016 12:57:56 PM -08:00
     EndTime             : 1/12/2016 12:59:13 PM -08:00
     Error               : 
     ErrorText           : 


