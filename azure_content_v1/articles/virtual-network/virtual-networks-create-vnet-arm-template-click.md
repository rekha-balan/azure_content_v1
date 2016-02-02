<properties
   pageTitle="Create a virtual network using an ARM template | Microsoft Azure"
   description="Learn how to create a virtual network using an ARM template | Resource Manager."
   services="virtual-network"
   documentationCenter=""
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/07/2015"
   ms.author="telmos"/>

# Create a virtual network by using an ARM template
> [AZURE.SELECTOR]
- [Preview portal](virtual-networks-create-vnet-arm-pportal.md)
- [PowerShell](virtual-networks-create-vnet-arm-ps.md)
- [Azure CLI](virtual-networks-create-vnet-arm-cli.md)
- [ARM template](virtual-networks-create-vnet-arm-template-click.md)

An Azure virtual network (VNet) is a representation of your own network in the cloud. You can control your Azure network settings and define DHCP address blocks, DNS settings, security policies, and routing. You can also further segment your VNet into subnets and deploy Azure IaaS virtual machines (VMs) and PaaS role instances, in the same way you can deploy physical and virtual machines to your on-premises datacenter. In essence, you can expand your network to Azure, bringing your own IP address blocks. Read the [virtual network overview](virtual-networks-overview.md) if you are not familiar with VNets.



>[AZURE.IMPORTANT]Before you work with Azure resources, it's important to understand that Azure currently has two deployment models: Resource Manager, and classic. Make sure you understand [deployment models and tools](azure-classic-rm.md) before working with any Azure resource. You can view the documentation for different tools by clicking the tabs at the top of this article. This document covers creating a VNet by using the Resource Manager deployment model. You can also [create a virtual network in the classic deployment model](virtual-networks-create-vnet-classic-pportal.md).

You will learn how to download and modify and existing ARM template from GitHub, and deploy the template from GitHub, PowerShell, and the Azure CLI.

If you are simply deploying the ARM template directly from GitHub, without any changes, skip to [deploy a template from github](#deploy-the-arm-template-by-using-click-to-deploy.md).

## Scenario

To better illustrate how to create a VNet and subnets, this document will use the scenario below.

![VNet scenario](./media/virtual-networks-create-vnet-scenario-include/vnet-scenario.png)

In this scenario you will create a VNet named **TestVNet** with a reserved CIDR block of **192.168.0.0./16**. Your VNet will contain the following subnets: 

- **FrontEnd**, using **192.168.1.0/24** as its CIDR block.
- **BackEnd**, using **192.168.2.0/24** as its CIDR block.

 

## Download and understand the ARM template

You can download the existing ARM template for creating a VNet and two subnets from github, make any changes you might want, and reuse it. To do so, follow the steps below.

1. Navigate to https://github.com/Azure/azure-quickstart-templates/tree/master/101-two-subnets.
2. Click **azuredeploy.json**, and then click **RAW**.
3. Save the file to a a local folder on your computer.
4. If you are familiar with ARM templates, skip to step 7.
5. Open the file you just saved and look at the contents under **parameters** in line 5. ARM template parameters provide a placeholder for values that can be filled out during deployment.

	| Parameter | Description |
	|---|---|
	| **location** | Azure region where the VNet will be created |
	| **vnetName** | Name for the new VNet |
	| **addressPrefix** | Address space for the VNet, in CIDR format |
	| **subnet1Name** | Name for the first VNet |
	| **subnet1Prefix** | CIDR block for the first subnet |
	| **subnet2Name** | Name for the second VNet |
	| **subnet2Prefix** | CIDR block for the second subnet |

	>[AZURE.IMPORTANT] ARM templates maintained in github can change over time. Make sure you check the template before using it.
	
6. Check the content under **resources** and notice the following:

	- **type**. Type of resource being created by the template. In this case, **Microsoft.Network/virtualNetworks**, which represent a VNet.
	- **name**. Name for the resource. Notice the use of **[parameters('vnetName')]**, which means the name will provided as input by the user or a parameter file during deployment.
	- **properties**. List of properties for the resource. This template uses the address space and subnet properties during VNet creation.

7. Navigate back to https://github.com/Azure/azure-quickstart-templates/tree/master/101-two-subnets.
8. Click **azuredeploy-paremeters.json**, and then click **RAW**.
9. Save the file to a a local folder on your computer.
10. Open the file you just saved and edit the values for the parameters. Use the values below to deploy the VNet described in our scenario.

		{
		  "location": {
		    "value": "Central US"
		  },
		  "vnetName": {
		      "value": "TestVNet"
		  },
		  "addressPrefix": {
		      "value": "192.168.0.0/16"
		  },
		  "subnet1Name": {
		      "value": "FrontEnd"
		  },
		  "subnet1Prefix": {
		    "value": "192.168.1.0/24"
		  },
		  "subnet2Name": {
		      "value": "BackEnd"
		  },
		  "subnet2Prefix": {
		      "value": "192.168.2.0/24"
		  }
		}

11. Save the file.
  

## Deploy the ARM template by using PowerShell

To deploy the ARM template you downloaded by using PowerShell, follow the steps below.

1. If you have never used Azure PowerShell, see [How to Install and Configure Azure PowerShell](powershell-install-configure.md) and follow the instructions all the way to the end to sign into Azure and select your subscription.

3. If necessary, run the **New-AzureRmResourceGroup** cmdlet to create a new resource group. The command below creates a resource group named *TestRG* in the *Central US* azure region. For more information about resource groups, visit [Azure Resource Manager Overview](resource-group-overview.md).

		New-AzureRmResourceGroup -Name TestRG -Location centralus
		
	Here is the expected output for the command above:

		ResourceGroupName : TestRG
		Location          : centralus
		ProvisioningState : Succeeded
		Tags              :
		Permissions       :
		                    Actions  NotActions
		                    =======  ==========
		                    *
		ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG

4. Run the **New-AzureRmResourceGroupDeployment** cmdlet to deploy the new VNet by using the template and parameter files you downloaded and modified above.

		New-AzureRmResourceGroupDeployment -Name TestVNetDeployment -ResourceGroupName TestRG `
			-TemplateFile C:\ARM\azuredeploy.json -TemplateParameterFile C:\ARM\azuredeploy-parameters.json
			
	Here is the expected output for the command above:
		
		DeploymentName    : TestVNetDeployment
		ResourceGroupName : TestRG
		ProvisioningState : Succeeded
		Timestamp         : 8/14/2015 9:40:00 PM
		Mode              : Incremental
		TemplateLink      :
		Parameters        :
		                    Name             Type                       Value
		                    ===============  =========================  ==========
		                    location         String                     Central US
		                    vnetName         String                     TestVNet
		                    addressPrefix    String                     192.168.0.0/16
		                    subnet1Prefix    String                     192.168.1.0/24
		                    subnet1Name      String                     FrontEnd
		                    subnet2Prefix    String                     192.168.2.0/24
		                    subnet2Name      String                     BackEnd
		
		Outputs           :

5. Run the **Get-AzureRmVirtualNetwork** cmdlet to view the properties of the new VNet, as shown below.


		Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
		
	Here is the expected output for the command above:
		
		Name              : TestVNet
		ResourceGroupName : TestRG
		Location          : centralus
		Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
		ProvisioningState : Succeeded
		Tags              :
		AddressSpace      : {
		                      "AddressPrefixes": [
		                        "192.168.0.0/16"
		                      ]
		                    }
		DhcpOptions       : {
		                      "DnsServers": null
		                    }
		NetworkInterfaces : null
		Subnets           : [
		                      {
		                        "Name": "FrontEnd",
		                        "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
		                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
		                        "AddressPrefix": "192.168.1.0/24",
		                        "IpConfigurations": [],
		                        "NetworkSecurityGroup": null,
		                        "RouteTable": null,
		                        "ProvisioningState": "Succeeded"
		                      },
		                      {
		                        "Name": "BackEnd",
		                        "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
		                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
		                        "AddressPrefix": "192.168.2.0/24",
		                        "IpConfigurations": [],
		                        "NetworkSecurityGroup": null,
		                        "RouteTable": null,
		                        "ProvisioningState": "Succeeded"
		                      }
		                    ]

## Deploy the ARM template by using the Azure CLI

To deploy the ARM template you downloaded by using Azure CLI, follow the steps below.

1. If you have never used Azure CLI, see [Install and Configure the Azure CLI](xplat-cli-install.md) and follow the instructions up to the point where you select your Azure account and subscription.
2. Run the **azure config mode** command to switch to Resource Manager mode, as shown below.

		azure config mode arm

	Here is the expected output for the command above:

		info:    New mode is arm

3. If necessary, run the **azure group create** to create a new resource group, as shown below. Notice the output of the command. The list shown after the output explains the parameters used. For more information about resource groups, visit [Azure Resource Manager Overview](resource-group-overview.md).

		azure group create -n TestRG -l centralus

	Here is the expected output for the command above:

		info:    Executing command group create
		+ Getting resource group TestRG
		+ Creating resource group TestRG
		info:    Created resource group TestRG
		data:    Id:                  /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG
		data:    Name:                TestRG
		data:    Location:            centralus
		data:    Provisioning State:  Succeeded
		data:    Tags: null
		data:
		info:    group create command OK

	- **-n (or --name)**. Name for the new resource group. For our scenario, *TestRG*.
	- **-l (or --location)**. Azure region where the new resource group will be created. For our scenario, *centralus*.

4. Run the **azure group deployment create** cmdlet to deploy the new VNet by using the template and parameter files you downloaded and modified above. The list shown after the output explains the parameters used.

		azure group deployment create -g TestRG -n TestVNetDeployment -f C:\ARM\azuredeploy.json -e C:\ARM\azuredeploy-parameters.json

	Here is the expected output for the command above:

		info:    Executing command group deployment create
		+ Initializing template configurations and parameters
		+ Creating a deployment
		info:    Created template deployment "TestVNetDeployment"
		+ Registering providers
		info:    Registering provider microsoft.network
		+ Waiting for deployment to complete
		data:    DeploymentName     : TestVNetDeployment
		data:    ResourceGroupName  : TestRG
		data:    ProvisioningState  : Succeeded
		data:    Timestamp          : 2015-08-14T21:56:11.152759Z
		data:    Mode               : Incremental
		data:    Name           Type    Value
		data:    -------------  ------  --------------
		data:    location       String  Central US
		data:    vnetName       String  TestVNet
		data:    addressPrefix  String  192.168.0.0/16
		data:    subnet1Prefix  String  192.168.1.0/24
		data:    subnet1Name    String  FrontEnd
		data:    subnet2Prefix  String  192.168.2.0/24
		data:    subnet2Name    String  BackEnd
		info:    group deployment create command OK

	- **-g (or --resource-group)**. Name of the resource group the new VNet will be created in.
	- **-f (or --template-file)**. Path to your ARM template file.
	- **-e (or --parameters-file)**. Path to your ARM parameters file.

5. Run the **azure network vnet show** command to view the properties of the new vnet, as shown below.

		azure network vnet show -g TestRG -n TestVNet

	Here is the expected output for the command above:

		info:    Executing command network vnet show
		+ Looking up virtual network "TestVNet"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		data:    Name                            : TestVNet
		data:    Type                            : Microsoft.Network/virtualNetworks
		data:    Location                        : centralus
		data:    ProvisioningState               : Succeeded
		data:    Address prefixes:
		data:      192.168.0.0/16
		data:    Subnets:
		data:      Name                          : FrontEnd
		data:      Address prefix                : 192.168.1.0/24
		data:
		data:      Name                          : BackEnd
		data:      Address prefix                : 192.168.2.0/24
		data:
		info:    network vnet show command OK


## Deploy the ARM template by using click to deploy

You can reuse pre-defined ARM templates upload to a github repository maintained by Microsoft and open to the community. THese templates can be deployed straight out of github, or downloaded and modified to fit your needs. To deploy a template that creates a VNet with two subnets, follow the steps below.

1. From a browser, navigate to [https://github.com/Azure/azure-quickstart-templates](https://github.com/Azure/azure-quickstart-templates).
2. Scroll down the list of templates, and click **101-two-subnets**. Check the **README.md** file, as shown below.

	![READEME.md file in github](./media/virtual-networks-create-vnet-arm-template-click-include/figure1.png)

3. Click **Deploy to Azure**. If necessary, enter your Azure login credentials. 
4. In the **Parameters** blade, enter the values you want to use to create your new VNet, and then click **OK**. The figure below shows the values for our scenario.

	![ARM template parameters](./media/virtual-networks-create-vnet-arm-template-click-include/figure2.png)

4. Click **Resource group** and select a resource group to add the VNet to, or click **Create new** to add the VNet to a new resource group. To learn more about resource groups, see [](). The figure below shows the resource group settings for a new resource group called **TestRG**.

	![Resource group](./media/virtual-networks-create-vnet-arm-template-click-include/figure3.png)

5. If necessary, change the **Subscription** and **Location** settings for your VNet.
6. If you do not want to see the VNet as a tile in the **Startboard**, disable **Pin to Startboard**.
5. Click **Leagl terms**, read the terms, and click **Buy** to agree. 
6. Click **Create** to create the VNet.

	![Submitting deployment tile in preview portal](./media/virtual-networks-create-vnet-arm-template-click-include/figure4.png)

7. Once the deployment is done, click **TestVNet** > **All settings** > **Subnets** to see the subnet properties, as shown below.

	![Create VNet in preview portal](./media/virtual-networks-create-vnet-arm-template-click-include/figure5.gif)

