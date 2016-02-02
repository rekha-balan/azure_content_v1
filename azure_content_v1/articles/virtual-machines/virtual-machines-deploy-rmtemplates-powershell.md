<properties
    pageTitle="Manage Azure Resource Manager VMs | Microsoft Azure"
    description="Manage virtual machines using Azure Resource Manager, templates, and PowerShell."
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
    ms.date="01/07/2016"
    ms.author="davidmu"/>

# Manage virtual machines using Azure Resource Manager and PowerShell
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [PowerShell](virtual-machines-deploy-rmtemplates-powershell.md)
> * [CLI](virtual-machines-deploy-rmtemplates-azure-cli.md)
> 
> 
<br>

Using Azure PowerShell and Resource Manager templates provides you with a lot of power and flexibility when managing resources in Microsoft Azure. You can use the tasks in this article to manage virtual machine resources.

> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model, which Microsoft recommends for most new deployments instead of the classic deployment model.

These tasks use only PowerShell:

* [Display information about a virtual machine](#displayvm.md)
* [Start a virtual machine](#start.md)
* [Stop a virtual machine](#stop.md)
* [Restart a virtual machine](#restart.md)
* [Delete a virtual machine](#delete.md)

Azure PowerShell is currently available in two releases - 1.0 and 0.9.8. If you have existing scripts and do not want to change them right now, you can continue using the 0.9.8 release. When using the 1.0 release, you should carefully test your scripts in pre-production environments before using them in production to avoid unexpected impacts.

1.0 cmdlets follow the naming pattern {verb}-AzureRm{noun}; whereas, the 0.9.8 names do not include **Rm** (for example, New-AzureRmResourceGroup instead of New-AzureResourceGroup). When using Azure PowerShell 0.9.8, you must first enable the Resource Manager mode by running the **Switch-AzureMode AzureResourceManager** command. This command is not necessary in 1.0.

New features will be added to only the 1.0 release. For information about the 1.0 release, including how to install and uninstall the release, see [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).



## <a id="displayvm"></a>Display information about a virtual machine
In the following command, replace *resource group name* with the name of the resource group that contains the virtual machine and *VM name* with the name of the machine, and then run it:

    Get-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

It returns something like this:

    ResourceGroupName        : rg1
    Id                       : /subscriptions/{subscription-id}/resourceGroups/
                                                            rg1/providers/Microsoft.Compute/virtualMachines/vm1
    Name                     : vm1
    Type                     : Microsoft.Azure.Management.Compute.Models.VirtualMachineGetResponse
    Location                 : westus
    Tags                     : {}
    AvailabilitySetReference : null
    Extensions               : []
    HardwareProfile          :  {
                                                                "VirtualMachineSize": "Standard_D1"
                                                            }
    InstanceView             : null
    Location                 : westus
    Name                     : vm1
    NetworkProfile           :  {
                                                                "NetworkInterfaces": [
                                                                    {
                                                                        "Primary": null,
                                                                        "ReferenceUri": "/subscriptions/{subscription-id}/resourceGroups/
                                                                        rg1/providers/Microsoft.Network/networkInterfaces/nc1"
                                                                    }
                                                                ]
                                                            }
    OSProfile                :  {
                                                                "AdminPassword": null,
                                                                "AdminUsername": "WinAdmin1",
                                                                "ComputerName": "vm1",
                                                                "CustomData": null,
                                                                "LinuxConfiguration": null,
                                                                "Secrets": [],
                                                                "WindowsConfiguration": {
                                                                    "AdditionalUnattendContents": [],
                                                                    "EnableAutomaticUpdates": true,
                                                                    "ProvisionVMAgent": true,
                                                                    "TimeZone": null,
                                                                    "WinRMConfiguration": null
                                                                }
                                                            }
    Plan                     : null
    ProvisioningState        : Succeeded
    StorageProfile           :     {
                                                                "DataDisks": [],
                                                                "ImageReference": {
                                                                    "Offer": "WindowsServer",
                                                                    "Publisher": "MicrosoftWindowsServer",
                                                                    "Sku": "2012-R2-Datacenter",
                                                                    "Version": "latest"
                                                                },
                                                                "OSDisk": {
                                                                    "OperatingSystemType": "Windows",
                                                                    "Caching": "ReadWrite",
                                                                    "CreateOption": "FromImage",
                                                                    "Name": "osdisk",
                                                                    "SourceImage": null,
                                                                    "VirtualHardDisk": {
                                                                        "Uri": "http://sa1.blob.core.windows.net/vhds/osdisk1.vhd"
                                                                    }
                                                                }
                                                            }
    DataDiskNames            :  {}
    NetworkInterfaceIDs      : { /subscriptions/{subscription-id}/resourceGroups/
                                                                rg1/providers/Microsoft.Network/networkInterfaces/nc1}

If you would like to see a video of this task being done, take a look at this:

[AZURE.VIDEO displaying-information-about-a-virtual-machine-in-microsoft-azure-with-powershell]AZURE.VIDEO displaying-information-about-a-virtual-machine-in-microsoft-azure-with-powershell]

## <a id="start"></a>Start a virtual machine
In the following command, replace *resource group name* with the name of the resource group that contains the virtual machine and *VM name* with the name of the machine, and then run it:

    Start-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

It returns something like this:

    Status              : Succeeded
    StatusCode          : OK
    RequestId           : 06935ddf-6e89-48d2-b46a-229493e3e9d1
    Output              :
    Error               :
    StartTime           : 4/28/2015 11:10:35 AM -07:00
    EndTime             : 4/28/2015 11:11:41 AM -07:00
    TrackingOperationId : c1aa0a70-4f4f-4d6c-a8ac-7ea35c004ce0

If you would like to see a video of this task being done, take a look at this:

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

## <a id="stop"></a>Stop a virtual machine
In the following command, replace *resource group name* with the name of the resource group that contains the virtual machine and *VM name* with the name of the machine, and then run it:

    Stop-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

You're asked for confirmation:

    Virtual machine stopping operation
    This cmdlet will stop the specified virtual machine. Do you want to continue?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

It returns something like this:

    Status              : Succeeded
    StatusCode          : OK
    RequestId           : aac41de1-b85d-4429-9a3d-040b922d2e6d
    Output              :
    Error               :
    StartTime           : 4/28/2015 11:10:35 AM -07:00
    EndTime             : 4/28/2015 11:11:41 AM -07:00
    TrackingOperationId : e1705973-d266-467e-8655-920016145347

If you would like to see a video of this task being done, take a look at this:

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

## <a id="restart"></a>Restart a virtual machine
In the following command, replace *resource group name* with the name of the resource group that contains the virtual machine and *VM name* with the name of the machine, and then run it:

    Restart-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

It returns something like this:

    Status              : Succeeded
    StatusCode          : OK
    RequestId           : 4b05891c-fdff-4c9a-89ca-e4f1d7691aed
    Output              :
    Error               :
    StartTime           : 1/5/2016 12:06:53 PM -08:00
    EndTime             : 1/5/2016 12:06:54 PM -08:00
    TrackingOperationId : 5aeeab89-45ab-41b9-84ef-9e9a7e732207


If you would like to see a video of this task being done, take a look at this:

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

## <a id="delete"></a>Delete a virtual machine
In the following command, replace *resource group name* with the name of the resource group that contains the virtual machine and *VM name* with the name of the machine, and then run it:  

    Remove-AzureRmVM -ResourceGroupName "resource group name" –Name "VM name"

> [!NOTE]
> You can use the **-Force** parameter to skip the confirmation prompt.
> 
> 
You're asked for confirmation if you didn't use the -Force parameter:

    Virtual machine removal operation
    This cmdlet will remove the specified virtual machine. Do you want to continue?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

It returns something like this:

    Status              : Succeeded
    StatusCode          : OK
    RequestId           : 2d723b40-ce1f-4b11-a603-dc659a13b6f0
    Output              :
    Error               :
    StartTime           : 1/5/2016 12:10:28 PM -08:00
    EndTime             : 1/5/2016 12:12:12 PM -08:00
    TrackingOperationId : d138ab29-83bf-4948-9d13-dab87db1a639

If you would like to see a video of this task being done, take a look at this:

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

