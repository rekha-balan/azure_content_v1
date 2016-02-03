<properties
    pageTitle="Web App Cloning"
    description="Learn how to clone your Web Apps to new Web Apps."
    services="app-service\web"
    documentationCenter=""
    authors="ahmedelnably"
    manager="stefsch"
    editor=""/>

<tags
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/13/2016"
    ms.author="ahmedelnably"/>

# Azure App Service App Cloning
With the release of Microsoft Azure PowerShell version 1.1.0 a new option has been added to New-AzureRMWebApp that would give the user the ability to clone an existing Web App to a newly created app in a different region or in the same region. This will enable customers to easily and quickly deploy a number of apps across different regions quickly and easily.

App cloning is currently only supported for premium tier app service plans. The new feature uses the same limitations as Web Apps Backup feature, see [Back up a web app in Azure App Service](web-sites-backup.md).

> [AZURE.NOTE] Although this article refers to web apps, it also applies to API apps and mobile apps.


## Cloning an existing App
Scenario: An existing web app in South Central US region, the user would like to clone the contents to a new web app in North Central US region. This can be accomplished by using the ARM version of the PowerShell cmdlet to create a new web app with the -SourceWebApp option.

Knowing the resource group name that contains the source web app, we can use the following PowerShell command to get the source web app's information (in this case named source-webapp):

    $srcapp = Get-AzureRmWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp

To create a new App Service Plan, we can use New-AzureRmAppServicePlan command as in the following example

    New-AzureRmAppServicePlan -Location "South Central US" -ResourceGroupName DestinationAzureResourceGroup -Name NewAppServicePlan

Using the New-AzureRmWebApp command, we can create the new web app in the North Central US region, and tie it to an existing premium tier App Service Plan, moreover we can use the same resource group as the source web app, or define a new resource group, the following demonstrates that:

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp

To clone an existing web app including all associated deployment slots, the user will need to use the IncludeSourceWebAppSlots parameter, the following PowerShell command demonstrates the use of that parameter with the New-AzureRmWebApp command:

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -IncludeSourceWebAppSlots $true

To clone an existing web app within the same region, the user will need to create a new resource group and a new app service plan in the same region, and then using the following PowerShell command to clone the web app

    $destapp = New-AzureRmWebApp -ResourceGroupName NewAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan NewAppServicePlan -SourceWebApp $srcap

## Cloning an existing App to an App Service Environment
Scenario: An existing web app in South Central US region, the user would like to clone the contents to a new web app to an existing App Service Environment (ASE).

Knowing the resource group name that contains the source web app, we can use the following PowerShell command to get the source web app's information (in this case named source-webapp):

    $srcapp = Get-AzureRmWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp

Knowing the ASE's name, and the resource group name that the ASE belongs to, the user can use the New-AzureRmWebApp command to create the new web app in the existing ASE, the following demonstrates that:

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -ASEName DestinationASE -ASEResourceGroupName DestinationASEResourceGroupName -SourceWebApp $srcapp

The Location parameter is required due to legacy reason, but in the case of creating an app in an ASE it will be ignored. 

## Cloning an existing App Slot
Scenario: The user would like to clone an existing Web App Slot to either a new Web App or a new Web App slot. The new Web App can be in the same region as the original Web App slot or in a different region.

Knowing the resource group name that contains the source web app, we can use the following PowerShell command to get the source web app slot's information (in this case named source-webappslot) tied to Web App source-webapp:

    $srcappslot = Get-AzureRmWebAppSlot -ResourceGroupName SourceAzureResourceGroup -Name source-webapp -Slot source-webappslot

The following demonstrates creating a clone of the source web app to a new web app:

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcappslot

## Configuring Traffic Manager while cloning a App
Configuring Traffic manager with When creating a new Web App while cloning from an existing one, you have the option to connect both web apps to either a new traffic manager profile or an existing one - note that only ARM version of Traffic Manager is supported.

### Creating a new Traffic Manager profile while cloning a App
Scenario: The user would like to clone an web app to another region, while configuring an ARM traffic manager profile that include both web apps. The following demonstrates creating a clone of the source web app to a new web app while configuring a new Traffic Manager profile:

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileName newTrafficManagerProfile

### Adding new cloned Web App to an existing Traffic Manager profile
Scenario: The user already have an ARM traffic manager profile that he would like to add both web apps as endpoint s. To do so, we first need to assemble the existing traffic manager profile id, we will need the subscription id, resource group name and the existing traffic manager profile name.

    $TMProfileID = "/subscriptions/<Your subscription ID goes here>/resourceGroups/<Your resource group name goes here>/providers/Microsoft.TrafficManagerProfiles/ExistingTrafficManagerProfileName"

After having the traffic manger id, the following demonstrates creating a clone of the source web app to a new web app while adding them to an existing Traffic Manager profile:

    $destapp = New-AzureRmWebApp -ResourceGroupName <Resource group name> -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileId $TMProfileID

## Current Restrictions
This feature is currently in preview, we are working to add new capabilities over time, the following list are the known restriction on the current version of app cloning:

* Auto scale settings are not cloned
* Backup schedule settings are not cloned
* VNET settings are not cloned
* App Insights are not automatically set up on the destination web app
* Easy Auth settings are not cloned
* Kudu Extension are not cloned
* TiP rules are not cloned

### References
* [Back up a web app in Azure App Service](web-sites-backup.md)
* [Azure Resource Manager support for Azure Traffic Manager Preview](../../articles/traffic-manager/traffic-manager-powershell-arm.md)
* [Introduction to App Service Environment](app-service-app-service-environment-intro.md)
* [Using Azure PowerShell with Azure Resource Manager](../powershell-azure-resource-manager.md)
