<properties
    pageTitle="Create a VM with a template | Microsoft Azure"
    description="Use a Resource Manager template and Azure PowerShell to create a new Windows virtual machine."
    services="virtual-machines"
    documentationCenter=""
    authors="davidmu1"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2016"
    ms.author="davidmu"/>

# Create a Windows virtual machine with a Resource Manager template and PowerShell
> [AZURE.NOTE] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model, which Microsoft recommends for most new deployments instead of the classic deployment model.

You can easily create a new Windows-based Azure virtual machine (VM) using a Resource Manager template with Azure PowerShell. This template creates a single virtual machine running Windows in a new virtual network with a single subnet in a new resource group.

![](./media/virtual-machines-create-windows-powershell-resource-manager-template-simple/windowsvm.png)

Azure PowerShell is currently available in two releases - 1.0 and 0.9.8. If you have existing scripts and do not want to change them right now, you can continue using the 0.9.8 release. When using the 1.0 release, you should carefully test your scripts in pre-production environments before using them in production to avoid unexpected impacts.

1.0 cmdlets follow the naming pattern {verb}-AzureRm{noun}; whereas, the 0.9.8 names do not include **Rm** (for example, New-AzureRmResourceGroup instead of New-AzureResourceGroup). When using Azure PowerShell 0.9.8, you must first enable the Resource Manager mode by running the **Switch-AzureMode AzureResourceManager** command. This command is not necessary in 1.0.

New features will be added to only the 1.0 release. For information about the 1.0 release, including how to install and uninstall the release, see [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).



## Create a Windows VM
Follow these steps to create a Windows VM using a Resource Manager template in the Github template repository with Azure PowerShell.

In the following commands, fill in a name that you want to use for the deployment, a name for the new resource group, and the location where the resource should be created.

        $deployName="<deployment name>"
        $RGName="<resource group name>"
        $locName="<Azure location, such as West US>"
        $templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-windows-vm/azuredeploy.json"
        New-AzureRmResourceGroup –Name $RGName –Location $locName
        New-AzureRmResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI


When you run the **New-AzureRmResourceGroupDeployment** command, you are prompted to supply the values of parameters in the "parameters" section of the JSON file. When you specify all the parameter values, the command creates the resource group and the virtual machine.

You should see something like this:

        cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
        Supply values for the following parameters:
        (Type !? for Help.)
        newStorageAccountName: newsaacct
        adminUsername: WinAdmin1
        adminPassword: *********
        dnsNameForPublicIP: contoso
        VERBOSE: 10:56:59 AM - Template is valid.
        VERBOSE: 10:56:59 AM - Create template deployment 'TestDeployment'.
        VERBOSE: 10:57:08 AM - Resource Microsoft.Network/virtualNetworks 'MyVNET' provisioning status is succeeded
        VERBOSE: 10:57:11 AM - Resource Microsoft.Network/publicIPAddresses 'myPublicIP' provisioning status is running
        VERBOSE: 10:57:11 AM - Resource Microsoft.Storage/storageAccounts 'newsaacct' provisioning status is running
        VERBOSE: 10:57:38 AM - Resource Microsoft.Storage/storageAccounts 'newsaacct' provisioning status is succeeded
        VERBOSE: 10:57:40 AM - Resource Microsoft.Network/publicIPAddresses 'myPublicIP' provisioning status is succeeded
        VERBOSE: 10:57:45 AM - Resource Microsoft.Compute/virtualMachines 'MyWindowsVM' provisioning status is running
        VERBOSE: 10:57:45 AM - Resource Microsoft.Network/networkInterfaces 'myVMNic' provisioning status is succeeded
        VERBOSE: 11:01:59 AM - Resource Microsoft.Compute/virtualMachines 'MyWindowsVM' provisioning status is succeeded

        DeploymentName    : TestDeployment
        ResourceGroupName : TestRG
        ProvisioningState : Succeeded
        Timestamp         : 4/28/2015 6:02:13 PM
        Mode              : Incremental
        TemplateLink      :
        Parameters        :
                                            Name                   Type                       Value
                                            ===============        =========================  ==========
                                            newStorageAccountName  String                     newsaacct
                                            adminUsername          String                     WinAdmin1
                                            adminPassword          SecureString
                                            dnsNameForPublicIP     String                     contoso
                                            windowsOSVersion       String                     2012-R2-Datacenter
        Outputs           :


## Next Steps
Learn how to manage the virtual machine that you just created by reviewing [Manage virtual machines using Azure Resource Manager and PowerShell](virtual-machines-deploy-rmtemplates-powershell.md).

