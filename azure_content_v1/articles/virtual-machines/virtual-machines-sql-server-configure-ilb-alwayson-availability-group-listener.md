<properties 
    pageTitle="Configure an ILB Listener for AlwaysOn Availability Groups | Microsoft Azure"
    description="This tutorial uses resources created with  the classic deployment model, and creates an AlwaysOn Availability Group Listener in Azure using an Internal Load Balancer (ILB)."
    services="virtual-machines"
    documentationCenter="na"
    authors="rothja"
    manager="jeffreyg"
    editor="monicar" 
    tags="azure-service-management"/>

<tags 
    ms.service="virtual-machines"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="11/06/2015"
    ms.author="jroth" />

# Configure an ILB listener for AlwaysOn Availability Groups in Azure
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Internal Listener](virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener.md)
> * [External Listener](virtual-machines-sql-server-configure-public-alwayson-availability-group-listener.md)
> 
> 
## Overview
This topic shows you how to configure a listener for an AlwaysOn Availability Group by using an **Internal Load Balancer (ILB)**. 

> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the Resource Manager model.

Your Availability Group can contain replicas that are on-premises only, Azure only, or span both on-premises and Azure for hybrid configurations. Azure replicas can reside within the same region or across multiple regions using multiple virtual networks (VNets). The steps below assume you have already [configured an availability group](virtual-machines-sql-server-alwayson-availability-groups-gui.md) but have not configured a listener. 

## Guidelines and limitations for internal listeners
Note the following guidelines on the availability group listener in Azure using ILB:

* The availability group listener is supported on Windows Server 2008 R2, Windows Server 2012, and Windows Server 2012 R2.

* Only one internal availability group listener is supported per cloud service, because the listener is configured to the ILB, and there is only one ILB per cloud service. However, it is possible to create multiple external listeners. For more information, see [Configure an external listener for AlwaysOn Availability Groups in Azure](virtual-machines-sql-server-configure-public-alwayson-availability-group-listener.md).

* It is not supported to create an internal listener in the same cloud service where you also have an external listener using the cloud service's public VIP.


## Determine the accessibility of the listener
It is important to realize that there are two ways to configure an Availability Group listener in Azure. These methods differ in the type of Azure load balancer you use when you create the listener. The following table describes the differences:

| Load Balancer | Implementation | Use When: |
| ------------- | -------------- | ----------- |
| **External** | Uses the **public Virtual IP address** of the cloud service that hosts the Virtual Machines. | You need to access the listener from outside the virtual network, including from the internet. |
| **Internal** | Uses **Internal Load Balancing (ILB)** with a private address for the listener. | You only access the listener from within the same virtual network. This includes site-to-site VPN in hybrid scenarios. |

>[AZURE.IMPORTANT] For a listener using the cloud service's public VIP (external load balancer), any data returned through the listener is considered egress and charged at normal data transfer rates in Azure. This is true even if the client is located in the same virtual network and datacenter as the listener and databases. This is not the case with a listener using ILB.

ILB can only be configured on virtual networks with a regional scope. Existing virtual networks that have been configured for an affinity group cannot use ILB. For more information, see [Internal Load Balancer](../articles/load-balancer/load-balancer-internal-overview.md).


This article focuses on creating a listener that uses an **Internal Load Balancer (ILB)**. If you need an public/external listener, see the version of this article that provides steps for setting up an [external listener](virtual-machines-sql-server-configure-public-alwayson-availability-group-listener.md)

## Create load-balanced VM endpoints with direct server return
For ILB, you must first create the internal load balancer. This is done in the script below. 

You must create a load-balanced endpoint for each VM hosting an Azure replica. If you have replicas in multiple regions, each replica for that region must be in the same cloud service in the same VNet. Creating Availability Group replicas that span multiple Azure regions requires configuring multiple VNets. For more information on configuring cross VNet connectivity, see  [Configure VNet to VNet Connectivity](../articles/vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md).

1. In the Azure portal, navigate to each VM hosting a replica and view the details.

1. Click the **Endpoints** tab for each of the VMs.

1. Verify that the **Name** and **Public Port** of the listener endpoint you want to use is not already in use. In the example below, the name is “MyEndpoint” and the port is “1433”.

1. On your local client, download and install [the latest PowerShell module](https://azure.microsoft.com/downloads/).

1. Launch **Azure PowerShell**. A new PowerShell session is opened with the Azure administrative modules loaded.

1. Run **Get-AzurePublishSettingsFile**. This cmdlet directs you to a browser to download a publish settings file to a local directory. You may be prompted for your log-in credentials for your Azure subscription.

1. Run the **Import-AzurePublishSettingsFile** command with the path of the publish settings file that you downloaded:

		Import-AzurePublishSettingsFile -PublishSettingsFile <PublishSettingsFilePath>

	Once the publish settings file is imported, you can manage your Azure subscription in the PowerShell session.

1. For **ILB**, you should assign a static IP address. First, examine the current VNet configuration by running the following command:

        (Get-AzureVNetConfig).XMLConfiguration
2. Note the **Subnet** name for the subnet that contains the VMs that host the replicas. This will be used in the **$SubnetName** parameter in the script. 

3. Then note the **VirtualNetworkSite** name and the starting **AddressPrefix** for the subnet that contains the VMs that host the replicas. Look for an available IP Address by passing both values to the **Test-AzureStaticVNetIP** command and examining the **AvailableAddresses**. For example, if the VNet was named *MyVNet* and had a subnet address range that started at *172.16.0.128*, the following command would list available addresses:

        (Test-AzureStaticVNetIP -VNetName "MyVNet"-IPAddress 172.16.0.128).AvailableAddresses
4. Choose one of the available addresses and use it in the **$ILBStaticIP** parameter of the following script. 

5. Copy the PowerShell script below into a text editor and set the variable values to suit your environment (note that defaults have been provided for some parameters). Note that existing deployments that use affinity groups cannot add ILB. For more information on ILB requirements, see [Internal Load Balancer](../load-balancer/load-balancer-internal-overview.md). Also, if your availability group spans Azure regions, you must run the script once in each datacenter for the cloud service and nodes that reside in that datacenter.

        # Define variables
     $ServiceName = "<MyCloudService>" # the name of the cloud service that contains the availability group nodes
     $AGNodes = "<VM1>","<VM2>","<VM3>" # all availability group nodes containing replicas in the same cloud service, separated by commas
     $SubnetName = "<MySubnetName>" # subnet name that the replicas use in the VNet
     $ILBStaticIP = "<MyILBStaticIPAddress>" # static IP address for the ILB in the subnet
     $ILBName = "AGListenerLB" # customize the ILB name or use this default value

     # Create the ILB
     Add-AzureInternalLoadBalancer -InternalLoadBalancerName $ILBName -SubnetName $SubnetName -ServiceName $ServiceName -StaticVNetIPAddress $ILBStaticIP

     # Configure a load balanced endpoint for each node in $AGNodes using ILB
     ForEach ($node in $AGNodes)
     {
         Get-AzureVM -ServiceName $ServiceName -Name $node | Add-AzureEndpoint -Name "ListenerEndpoint" -LBSetName "ListenerEndpointLB" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ILBName -DirectServerReturn $true | Update-AzureVM 
     }
6. Once you have set the variables, copy the script from the text editor into your Azure PowerShell session to run it. If the prompt still shows >>, type ENTER again to make sure the script starts running.Note 


> [!NOTE]
> The Azure classic portal does not support the Internal Load Balancer at this time, so you will not see either the ILB or the endpoints in the Azure classic portal. However, **Get-AzureEndpoint** returns an internal IP address if the Load Balancer is running on it. Otherwise, it returns null.
> 
> 
## Verify that KB2854082 is installed if necessary
Next, if any servers on the cluster are running Windows Server 2008 R2 or Windows Server 20012, you must verify that the hotfix [KB2854082](http://support.microsoft.com/kb/2854082) is installed on each of the on-premises servers or Azure VMs that are part of the cluster. Any server or VM that is in the cluster, but not in the availability group, should also have this hotfix installed.

In the remote desktop session for each of the cluster nodes, download [KB2854082](http://support.microsoft.com/kb/2854082) to a local directory. Then, install the hotfix on each of the cluster nodes sequentially. If the cluster service is currently running on the cluster node, the server is restarted at the end of the hotfix installation.

>[AZURE.WARNING] Stopping the cluster service or restarting the server affects the quorum health of your cluster and the availability group, and may cause your cluster to go offline. To maintain the high availability of your cluster during installation, make sure that:
>
> - The cluster is in optimal quorum health, 
> - All cluster nodes are online before installing the hotfix on any node, and
> - Allow the hotfix installation to run to completion on one node, including fully restarting the server, before installing the hotfix on any other node in the cluster.


## Open the firewall ports in availability group nodes
In this step, you create a firewall rule to open the probe port for the load-balanced endpoint (59999 as specified earlier), and another rule to open the availability group listener port. Since you created the load-balanced endpoint on the Azure VMs that contain availability group replicas, you need to open the probe port and the listener port on the respective Azure VMs.

1. On VMs hosting replicas, launch **Windows Firewall with Advanced Security**.

1. Right-click **Inbound Rules** and click **New Rule**.

1. In the **Rule Type** page, select **Port**, then click **Next**.

1. In the **Protocol and Ports** page, select **TCP** and type **59999** in the **Specific local ports** box. Then, click **Next**.

1. In the **Action** page, keep **Allow the connection** selected and click **Next**.

1. In the **Profile** page, accept the default settings and click **Next**.

1. In the **Name** page, specify a rule name, such as **AlwaysOn Listener Probe Port** in the **Name** text box, and click **Finish**.

1. Repeat the above steps for the availability group listener port (as specified earlier in the $EndpointPort parameter of the script) and specify an appropriate rule name, such as **AlwaysOn Listener Port**.

## Create the availability group listener
In this step, you manually create the availability group listener in Failover Cluster Manager and SQL Server Management Studio (SSMS).

1. Open Failover Cluster Manager from the node hosting the primary replica.

1. Select the **Networks** node, and note the cluster network name. This name will be used in the $ClusterNetworkName variable in the PowerShell script.

1. Expand the cluster name, and then click **Roles**.

1. In the **Roles** pane, right-click the availability group name and then select **Add Resource** > **Client Access Point**.

	![Add Client Access Point for Availability Group](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678769.gif)

1. In the **Name** box, create a name for this new listener, then click **Next** twice, and then click **Finish**. Do not bring the listener or resource online at this point.

1. Click the **Resources** tab, then expand the Client Access Point you just created. You will see the **IP Address** resource for each of the cluster networks in your cluster. If this is an Azure-only solution, you will only see one IP address resource.

1. If you are configuring a hybrid solution, continue with this step. If you are configuring an Azure only solution, skip to the next step. 
	 - Right-click the IP Address resource that corresponds to your on-premises subnet, then select **Properties**. Note the IP Address Name and network name.
	 - Select **Static IP Address**, assign an unused IP address and then click **OK**.

1. Right-click the IP Address resource that corresponds to your Azure subnet and then select Properties.
	>[AZURE.NOTE] If the listener later fails to come online due to a conflicting IP address selected by DHCP, you can configure a valid static IP Address in this properties window.

1. In the same **IP Address** properties window, change the **IP Address Name**. This IP address name will be used in the **$IPResourceName** variable of the PowerShell script. Repeat this step for each IP resource if your solution spans multiple Azure VNets.

1. For ILB, you must use the IP address of the Internal Load Balancer (ILB) created earlier. Use the following script to obtain this IP Address in PowerShell.

        # Define variables
     $ServiceName="<MyServiceName>" # the name of the cloud service that contains the AG nodes
     (Get-AzureInternalLoadBalancer -ServiceName $ServiceName).IPAddress
2. On one of the VMs, copy the PowerShell script below into a text editor and set the variables to the values you noted earlier.

        # Define variables
     $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
     $IPResourceName = "<IPResourceName>" # the IP Address resource name 
     $ILBIP = “<X.X.X.X>” # the IP Address of the Internal Load Balancer (ILB)

     Import-Module FailoverClusters

     # If you are using Windows Server 2012 or higher, use the Get-Cluster Resource command. If you are using Windows Server 2008 R2, use the cluster res command. Both commands are commented out. Choose the one applicable to your environment and remove the # at the beginning of the line to convert the comment to an executable line of code. 

     # Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
     # cluster res $IPResourceName /priv enabledhcp=0 address=$ILBIP probeport=59999  subnetmask=255.255.255.255
3. Once you have set the variables, open an elevated Windows PowerShell window, then copy the script from the text editor and paste into your Azure PowerShell session to run it. If the prompt still shows >>, type ENTER again to make sure the script starts running. 

4. Repeat this on each VM. This script configures the IP Address resource with the IP address of the cloud service and sets other parameters like the probe port. When the IP Address resource is brought online, it can then respond to the polling on the probe port from the load-balanced endpoint created earlier in this tutorial.


## Bring the listener online
1. Navigate back to Failover Cluster Manager.  Expand **Roles** and then highlight your Availability Group.  On the **Resources** tab, right-click the listener name and click Properties.

1. Click the **Dependencies** tab. If there are multiple resources listed, verify that the IP addresses have OR, not AND, dependencies.  Click **OK**.

1. Right-click the listener name and click **Bring Online**.

1. Once the listener is online, from the **Resources** tab, right-click the availability group and click **Properties**.

	![Configure the Availability Group Resource](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678772.gif)

1. Create a dependency on the listener name resource (not the IP address resources name). Click **OK**.

	![Add Dependency on the Listener Name](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678773.gif)

1. Launch **SQL Server Management Studio** and connect to the primary replica.

1. Navigate to **AlwaysOn High Availability** | **Availability Groups** | **<AvailabilityGroupName>** | **Availability Group Listeners**. 

3. You should now see the listener name that you created in Failover Cluster Manager. Right-click the listener name and click **Properties**.

1. In the **Port** box, specify the port number for the availability group listener by using the $EndpointPort you used earlier (in this tutorial, 1433 was the default), then click **OK**.


## Follow-up items
After the availability group listener is created, it may be necessary to adjust the **RegisterAllProvidersIP** and **HostRecordTTL** cluster parameters for the listener resource.  These parameters may reduce reconnection time after a failover which may prevent connection timeouts. For more information on these parameters, as well as sample code, see [Create or Configure an Availability Group Listener](https://msdn.microsoft.com/library/hh213080.aspx#MultiSubnetFailover).


## Test the availability group listener (within the same VNet)
In this step, you test the availability group listener using a client application running on the same network.

For client connectivity, please note the following requirements:

- Client connections to the listener must come from machines that reside in a different cloud service than the one that hosts the AlwaysOn Availability replicas.

- If the AlwaysOn replicas are in different subnets, clients must specify "MultisubnetFailover=True" in the connection string. This results in parallel connection attempts to replicas in the different subnets. Note that this scenario includes a cross-region AlwaysOn Availability Group deployment.

One example would be to connect to the listener from one of the VMs in the same Azure VNet (but not one that hosts a replica). An easy way to complete this test is to try to connect SSMS to the availability group listener. Another simple method is to run [SQLCMD.exe](https://technet.microsoft.com/library/ms162773.aspx) as follows:

	sqlcmd -S "<ListenerName>,<EndpointPort>" -d "<DatabaseName>" -Q "select @@servername, db_name()" -l 15

> [AZURE.NOTE] If the EndpointPort value is 1433, it is not required to specify it in the call. The previous call also assumes that the client machine is joined to the same domain and that the caller has been granted permissions on the database using windows authentication.

When testing the listener, be sure to fail over the availability group to make sure that clients can connect to the listener across failovers.

## Next Steps
In addition to automatically connecting clients to the primary replica, a listener can also be used to redirect read-only workloads to the secondaries. This can improve the performance and scalability of your overall solution. For more information, see 
[Use ReadIntent Routing with Azure AlwaysOn Availability Group Listener](http://go.microsoft.com/fwlink/?LinkId=522515).

For other information about using SQL Server in Azure, see [SQL Server on Azure Virtual Machines](../articles/virtual-machines/virtual-machines-sql-server-infrastructure-services.md).

