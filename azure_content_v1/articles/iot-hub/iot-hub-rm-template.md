<properties
    pageTitle="Create an IoT Hub using a Resource Manager template | Microsoft Azure"
    description="Follow this tutorial to get started using Resource Manager templates to create an IoT Hub."
    services="iot-hub"
    documentationCenter=".net"
    authors="dominicbetts"
    manager="timlt"
    editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="dotnet"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="11/23/2015"
     ms.author="dobett"/>

# Tutorial: Create an IoT hub using a C# program
> [AZURE.SELECTOR]
- [C# with template](iot-hub-rm-template.md)
- [C# with REST](iot-hub-rm-rest.md)

## Introduction
You can use Azure Resource Manager to create and manage Azure IoT hubs programmatically. This tutorial shows you how to use a resource manager template to create an IoT hub from a C# program.

> [!NOTE]
> Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the Resource Manager deployment model.
> 
> 
In order to complete this tutorial you'll need the following:

* Microsoft Visual Studio 2015.
* An active Azure account. <br/>If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/).
* [Microsoft Azure PowerShell 1.0](https://azure.microsoft.com/en-us/blog/azps-1-0-pre/) or later.

## Prepare to authenticate Resource Manager requests

You must authenticate all the operations that you perform on resources using the [Azure Resource Manager][lnk-authenticate-arm] with Azure Active Directory (AD). The easiest way to configure this is to use PowerShell or Azure CLI.

If you have not installed [Azure PowerShell 1.0][lnk-powershell-install], you can do so with the following PowerShell commands. You will need to run PowerShell as an administrator.

```
# Install the Azure Resource Manager modules from PowerShell Gallery
Install-Module AzureRM
Install-AzureRM

# Install the Azure Service Management module from PowerShell Gallery
Install-Module Azure
```

The following steps show how to set up password authentication for an AD application using PowerShell. You can run these commands in a standard PowerShell session.

1. Log in to your Azure subscription using the following command:

    ```
    Login-AzureRmAccount
    ```

2. Make a note of your **TenantId** and **SubscriptionId**. You will need them later.

3. Create a new Azure Active Directory application using the following command, replacing the place holders:

    - **{Display name}:** a display name for your application such as **MySampleApp**
    - **{Home page URL}:** the URL of the home page of your app such as **http://mysampleapp/home**. This URL does not need to point to a real application.
    - **{Application identifier}:** A unique identifier such as **http://mysampleapp**. This URL does not need to point to a real application.
    - **{Password}:** A password that you will use to authenticate with your app.

    ```
    New-AzureRmADApplication -DisplayName {Display name} -HomePage {Home page URL} IdentifierUris {Application identifier} -Password {Password}
    ```
    
4. Make a note of the **ApplicationId** of the application you created. You will need this later.

5. Create a new service principal using the following command, replacing **{MyApplicationId}** with the **ApplicationId** from the previous step:

    ```
    New-AzureRmADServicePrincipal -ApplicationId {MyApplicationId}
    ```
    
6. Setup a role assignment using the following command, replacing **{MyApplicationId}** with your **ApplicationId**.

    ```
    New-AzureRmRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName {MyApplicationId}
    ```
    
You have now finished creating the Azure AD application that will enable you to authenticate from your custom C# application. You will need the following values later in this tutorial:

- TenantId
- SubscriptionId
- ApplicationId
- Password

[lnk-authenticate-arm]: https://msdn.microsoft.com/library/azure/dn790557.aspx
[lnk-powershell-install]: https://azure.microsoft.com/en-us/blog/azps-1-0-pre/


## Prepare your Visual Studio project
1. In Visual Studio, create a new Visual C# Windows project using the **Console Application** project template. Name the project **CreateIoTHub**.

2. In Solution Explorer, right-click on your project and then click **Manage NuGet Packages**.

3. In NuGet Package Manager, check **Include prerelease** and search for **Microsoft.Azure.Management.Resources**. Select version **2.18.11-preview**. Click **Install**, in **Review Changes** click **OK**, then click **I Accept** to accept the licenses.

4. In NuGet Package Manager, search for **Microsoft.IdentityModel.Clients.ActiveDirectory**. Select version **2.19.208020213**. Click **Install**, in **Review Changes** click **OK**, then click **I Accept** to accept the license.

5. In NuGet Package Manager, search for **Microsoft.Azure.Common**. Select version **2.1.0**. Click **Install**, in **Review Changes** click **OK**, then click **I Accept** to accept the licenses.

6. In Program.cs, replace the existing **using** statements with the following:

    ```
 using System;
 using System.IO;
 using System.Net;
 using System.Net.Http.Headers;
 using Microsoft.Azure;
 using Microsoft.Azure.Management.Resources;
 using Microsoft.Azure.Management.Resources.Models;
 using Microsoft.IdentityModel.Clients.ActiveDirectory;
 ```
7. In Program.cs, add the following static variables replacing the placeholder values. You made a note of **ApplicationId**, **SubscriptionId**, **TenantId**, and **Password** earlier in this tutorial. **Resource group name** is the name of the resource group you will use when you create the IoT Hub, it can be a pre-existing resource group or a new one. **IoT Hub name** is the name of the IoT Hub you will create, such as **MyIoTHub**. **Deployment name** is a name for the deployment, such as **Deployment_01**.

    ```
 static string applicationId = "{Your ApplicationId}";
 static string subscriptionId = "{Your SubscriptionId";
 static string tenantId = "{Your TenantId}";
 static string password = "{Your application Password}";

 static string rgName = "{Resource group name}";
 static string iotHubName = "{IoT Hub name}";
 static string deploymentName = "{Deployment name}";
 ```


## Obtain a Resource Manager token

Azure Active Directory must authenticate all the tasks that you perform on resources using the Azure Resource Manager. The example shown here uses password authentication, for other approaches see [Authenticating Azure Resource Manager requests][lnk-authenticate-arm].

1. Add the following code to the **Main** method in Program.cs to retrieve a token from Azure AD using the application id and password.

    ```
    var authContext = new AuthenticationContext(string.Format  
      ("https://login.windows.net/{0}", tenantId));
    var credential = new ClientCredential(applicationId, password);
    AuthenticationResult token = authContext.AcquireTokenAsync
      ("https://management.core.windows.net/", credential).Result;
    
    if (token == null)
    {
      Console.WriteLine("Failed to obtain the token");
      return;
    }
    ```

2. Create a **ResourceManagementClient** object that uses the token by adding the following code to the end of the **Main** method:

    ```
    var creds = new TokenCloudCredentials(subscriptionId, token.AccessToken);
    var client = new ResourceManagementClient(creds);
    ```

3. Create, or obtain a reference to, the resource group you are using:

    ```
    var rgResponse = client.ResourceGroups.CreateOrUpdateAsync(rgName,
        new ResourceGroup("East US")).Result;
    if (rgResponse.StatusCode != HttpStatusCode.Created
        && rgResponse.StatusCode != HttpStatusCode.OK)
    {
      Console.WriteLine("Problem creating resource group");
      return;
    }
    ```

[lnk-authenticate-arm]: https://msdn.microsoft.com/library/azure/dn790557.aspx

## Submit a template to create an IoT hub
Use a JSON template to create a new IoT hub in your resource group. You can also use a template to make changes to an existing IoT hub.

1. In Solution Explorer, right-click on your project and click **Add** then **New Item**. Add a new JSON file called **template.json** to your project.

2. In Solution Explorer, select **template.json**, and then in **Properties** set **Copy to Output Directory** to **Copy always**.

3. Replace the contents of **template.json** with the following resource definition to add a new standard IoT hub to the **East US** region:

    ```
 {
 "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
 "contentVersion": "1.0.0.0",

 "resources": [
   {
     "apiVersion": "2015-08-15-preview",
     "type": "Microsoft.Devices/Iothubs",
     "name": "[IotHubName]",
     "location": "East US",
     "sku": {
       "name": "S1",
       "tier": "Standard",
       "capacity": 1
     },
     "properties": {
       "location": "East US"
     }
   }
 ]
 }
 ```
4. Add the following method to Program.cs:

    ```
 static bool CreateIoTHub(ResourceManagementClient client)
 {

 }
 ```
5. Add the following code to the **CreateIoTHub** method to load the template file, add the name of the IoT hub, and submit the template to the Azure resource manager:

    ```
 string template = File.ReadAllText("template.json");
 template = template.Replace("[IotHubName]", iotHubName);
 var createResponse = client.Deployments.CreateOrUpdateAsync(
   rgName,
   deploymentName,
   new Deployment()
   {
     Properties = new DeploymentProperties
     {
       Mode = DeploymentMode.Incremental,
       Template = template
     }
   }).Result;
 ```
6. Add the following code to the **CreateIoTHub** method that waits until the deployment completes successfully:

    ```
 string state = createResponse.Deployment.Properties.ProvisioningState;
 while (state != "Succeeded" && state != "Failed")
 {
   var getResponse = client.Deployments.GetAsync(
     rgName,
     deploymentName).Result;

   state = getResponse.Deployment.Properties.ProvisioningState;
   Console.WriteLine("Deployment state: {0}", state);
 }

 if (state != "Succeeded")
 {
   Console.WriteLine("Failed to create iothub");
   return false;
 }
 return true;
 ```


## Retrieve the IoT Hub keys

Display the authentication keys for the new IoT Hub.

1. Add the following method to Program.cs:

    ```
    static void ShowIoTHubKeys(ResourceManagementClient client, string token)
    {
    
    }
    ```

2. Add the following code to the **ShowIoTHubKeys** method to print the authentication keys to the console:

    ```
    client.HttpClient.DefaultRequestHeaders.Authorization = 
        new AuthenticationHeaderValue("Bearer", token);
    var httpsRepsonse = client.HttpClient.PostAsync(
        string.Format("https://management.azure.com/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}/listKeys?api-version=2015-08-15-preview", 
        subscriptionId, rgName, iotHubName),
        null).Result;
    
    Console.WriteLine("Keys: {0}, 
        httpsRepsonse.Content.ReadAsStringAsync().Result);
    ```

## Complete and run the application
You can now complete the application by calling the **CreateIoTHub** and **ShowIoTHubKeys** methods before you build and run it.

1. Add the following code to the end of the **Main** method:

    ```
 if (CreateIoTHub(client))
     ShowIoTHubKeys(client, token.AccessToken);
 Console.ReadLine();
 ```
2. Click **Build** and then **Build Solution**. Correct any errors.

3. Click **Debug** and then **Start Debugging** to run the application. It may take several minutes for the deployment to run.

4. You can verify that your application added the new IoT hub by visiting the [portal](https://portal.azure.com/) and viewing your list of resources, or by using the **Get-AzureRmResource** PowerShell cmdlet.


> [!NOTE]
> This example application adds an S1 Standard IoT Hub for which you will be billed. You can delete the IoT hub through the [portal](https://portal.azure.com/) or by using the **Remove-AzureRmResource** PowerShell cmdlet when you are finished.
> 
> 
## Next steps
* Explore the capabilities of the [IoT Hub Resource Provider REST API](https://msdn.microsoft.com/library/mt589014.aspx).
* Read [Azure Resource Manager overview](https://azure.microsoft.com/documentation/articles/resource-group-overview/) to learn more about the capabilities of Azure Resource Manager.

<!-- Links -->

[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-powershell-install]: https://azure.microsoft.com/en-us/blog/azps-1-0-pre/
[lnk-rest-api]: https://msdn.microsoft.com/library/mt589014.aspx
[lnk-azure-rm-overview]: https://azure.microsoft.com/documentation/articles/resource-group-overview/
