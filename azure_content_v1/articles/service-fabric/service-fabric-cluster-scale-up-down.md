<properties
   pageTitle="Scaling a Service Fabric cluster up or down | Microsoft Azure"
   description="Scale a Service Fabric Cluster up or down to match demand by adding or removing virtual machine nodes"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/11/2016"
   ms.author="chackdan"/>

# Scaling a Service Fabric Cluster up or down by adding or removing Virtual Machines (VMs) from it.
You can scale Service Fabric clusters up or down to match demand by adding or removing virtual machines.

> [!NOTE]
> It is assumed that your subscription has enough cores to add the new VMs that will make up this cluster.
> 
> 
## Manually scaling a Service Fabric cluster
If your cluster has multiple Node types, you will have to add or remove VMs to/from specific node types. Ypu can add VMs to one node type and remove VMs from the other, in the same deployment.

### Upgrading a cluster you had deployed using the portal
Since this is an involved process, we have a powershell Module uploaded to a Git Repo, that does this for you. 

**Step 1**: Copy this folder down to your machine from this [Git repo](https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers).

**Step 2**: Make sure  Azure SDK 1.0+ installed on your machine.

**Step 3**: Open a Powershell window and import the ServiceFabricRPHelpers.psm

```
Remove-Module ServiceFabricRPHelpers
```

Import the poweshell module you just copied down.
you can just copy the following cmd and change the path to the .psm1 to be that of your machine. 

```
Import-Module "C:\Users\chackdan\Documents\GitHub\Service-Fabric\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1"
```

 **Step 4**: Log in to your Azure Account

```
Login-AzureRmAccount
```

Run the Invoke-ServiceFabricRPClusterScaleUpgrade command , make sure that you have the resource group name and the subscription correct.

```
Invoke-ServiceFabricRPClusterScaleUpgrade -ResourceGroupName <string> -SubscriptionId <string>
```

Here is a filled out example of the same PS command

```
Invoke-ServiceFabricRPClusterScaleUpgrade -ResourceGroupName chackod02 -SubscriptionId 18ad2z84-84fa-4798-ad71-e70c07af879f
```

  **Step 5**: The command will now retrieve the cluster information and walk you through all the node types, first telling you the current VM/Node count for that node type and then ask you to provide what the new VM/Node count should be.

 **To scaleup this node type**, specify a larger number than the current VM count.

**To Scale down this node type**, specify a smaller number than the current VM count. 

These prompt will now loop through for all the node types. If your cluster has only one nodetype, then you will get prompted only once.

  **Step 6**: If you are adding new VMs, you will now get a prompt to provide the remote desktop userid and password for the VMs you are adding.

**Step 7**: In the output window, you will now see messages, informing you of the progress of your deployment. Once the deployment is complete, your cluster should have the number of Nodes you specified in Step 5.

![ScaleupDownPSOut][ScaleupDownPSOut]

**Step 8**: If this was a scale-down request, then you have one more step of deleting the VMs. The script de-activates all the VMs that you requested removed, ie, on these nodes/VMs there are no application or system replicas. So it is now safe to delete those VMs. 

**NOTE** - Although the deactivated nodes are not used by your SF cluster any more, You must delete the deactivated VMs, so that you do not get charged for them. 

**Step 8.1**: Log on to the Azure Portal [http://aka.ms/servicefabricportal](http://aka.ms/servicefabricportal).

**Step 8.2**: Browse to the Cluster resource you were scaling down and click on "All Settings" on the Essentials web part.

**Step 8.3**: Click on Node Types, you will now see a list of Nodes that are Deactivated. In this picture, chackodnt15, chackodnt24, chackodnt25 and chackodnt26 are the VMs that I now need to delete.

![DeactivatedNodeList][DeactivatedNodeList]

**Step 8.4**: Delete those VMs via PS or Portal. Once they are deleted, you have now finished with the scale down of your cluster. 

```
Remove-AzureRmResource -ResourceName <Deactivated Node name without the understore> -ResourceType Microsoft.Compute/virtualMachines -ResourceGroupName <Resource Group name> -ApiVersion <Api version>
```
Here is a filled out example of that command

```
Remove-AzureRmResource -ResourceName chackodnt26 -ResourceType Microsoft.Compute/virtualMachines -ResourceGroupName chackod02 -ApiVersion 2015-05-01-preview
```

### Upgrading a cluster that you had deployed using ARM PowerShell/CLI
If you had deployed your cluster using an ARM template initially, then you need to modify the count of the VMs for a given node type and all the resources that support the VM. The minimum number of VMs allowed for the primary node type is 5 (for non-test clusters), for test clusters the minimum is 3.

Once you have the new template, you can deploy it via ARM using the same resource group as the cluster that you are upgrading. 

## Auto Scaling of the Service Fabric cluster
At this time, Service Fabric clusters do not support auto-scaling. In the near future, clusters will be built on top of virtual machine scale sets (VMSS), at which time auto-scaling will become possible and will behave similarly to the auto-scale behavior available in Cloud Services.

## Next steps
* [Learn about cluster upgrades](service-fabric-cluster-upgrade.md)
* [Learn about partitioning stateful services for maximum scale](service-fabric-concepts-partitioning.md)

<!--Image references-->

[ScaleupDownPSOut]: ./media/service-fabric-cluster-scale-up-down/ScaleupDownPSOut.png
[DeactivatedNodeList]: ./media/service-fabric-cluster-scale-up-down/DeactivatedNodeList.png
