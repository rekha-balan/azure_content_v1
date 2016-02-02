<properties 
    pageTitle="Connect to a SQL Server Virtual Machine (Classic) | Microsoft Azure"
    description="This topic uses resources created with the classic deployment model, and describes how to connect to SQL Server running on a Virtual Machine in Azure. The scenarios differ depending on the networking configuration and the location of the client."
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
    ms.date="12/18/2015"
    ms.author="jroth" />

# Connect to a SQL Server Virtual Machine on Azure (Classic Deployment)
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Resource Manager](virtual-machines-sql-server-connectivity-resource-manager.md)
> * [Classic](virtual-machines-sql-server-connectivity.md)
> 
> 
## Overview
Configuring connectivity to SQL Server running on an Azure Virtual Machine does not differ dramatically from the steps required for an on-premises SQL Server instance. You still have to work with configuration steps involving the firewall, authentication, and database logins.

But there are some SQL Server connectivity aspects that are specific to Azure VMs. This article covers some [general connectivity scenarios](#connection-scenarios.md) and then provides [detailed steps for configuring SQL Server connectivity in an Azure VM](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm.md).

This article focuses on connectivity. For a full walk-through of both provisioning and connectivity, see [Provisioning a SQL Server Virtual Machine on Azure](virtual-machines-provision-sql-server.md).

> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the Resource Manager model.

## Connection scenarios
The way a client connects to SQL Server running on a Virtual Machine differs depending on the location of the client and the machine/networking configuration. These scenarios include:

* [Connect to SQL Server in the same cloud service](#connect-to-sql-server-in-the-same-cloud-service.md)
* [Connect to SQL Server over the internet](#connect-to-sql-server-over-the-internet.md)
* [Connect to SQL Server in the same virtual network](#connect-to-sql-server-in-the-same-virtual-network.md)

### Connect to SQL Server in the same cloud service
Multiple virtual machines can be created in the same cloud service. To understand this virtual machines scenario, see [How to connect virtual machines with a virtual network or cloud service](cloud-services-connect-virtual-machine.md).

First, follow the [steps in this article to configure connectivity](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm.md). Note that you do not have to setup a public endpoint if you are going to be connecting between machines in the same cloud service. 

You can use the VM **hostname** in the client connection string. The hostname is the name that you gave your VM during creation. For example, if you SQL VM named **mysqlvm** with a cloud service DNS name of **mycloudservice.cloudapp.net**, a client VM in the same cloud service could use the following connection string to connect:

    "Server=mysqlvm;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

### Connect to SQL Server over the Internet
If you want to connect to your SQL Server database engine from the Internet, you must create a virtual machine endpoint for incoming TCP communication. This Azure configuration step, directs incoming TCP port traffic to a TCP port that is accessible to the virtual machine.

First, follow the [steps in this article to configure connectivity](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm.md). Any client with internet access could then connect to the SQL Server instance by specifying the cloud service DNS name (such as **mycloudservice.cloudapp.net**) and the VM endpoint (such as **57500**).

    "Server=mycloudservice.cloudapp.net,57500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

Although this enables connectivity for clients over the internet, this does not imply that anyone can connect to your SQL Server. Outside clients have to the correct username and password. For additional security, don't use the well-known port 1433 for the public virtual machine endpoint. And if possible, consider adding an ACL on your endpoint to restrict traffic only to the clients you permit. For instructions on using ACLs with endpoints, see [Manage the ACL on an endpoint](virtual-machines-set-up-endpoints.md#manage-the-acl-on-an-endpoint). 

> [!NOTE]
> It is important to note that when you use this technique to communicate with SQL Server, all data returned is considered outgoing traffic from the datacenter. It is subject to normal [pricing on outbound data transfers](https://azure.microsoft.com/pricing/details/data-transfers/). This is true even if you use this technique from another machine or cloud service within the same Azure datacenter, because traffic still goes through Azure's public load balancer.
> 
> 
### Connect to SQL Server in the same virtual network
[Virtual Network](..\virtual-network\virtual-networks-overview.md) enables additional scenarios. You can connect VMs in the same virtual network, even if those VMs exist in different cloud services. And with a [site-to-site VPN](../vpn-gateway/vpn-gateway-site-to-site-create.md), you can create a hybrid architecture that connects VMs with on-premises networks and machines.

Virtual networks also enables you to join your Azure VMs to a domain. This is the only way to use Windows Authentication to SQL Server. The other connection scenarios require SQL Authentication with user names and passwords.

First, follow the [steps in this article to configure connectivity](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm.md). If you are going to configure a domain environment and Windows Authentication, you do not need to use the steps in this article to configure SQL Authentication and logins. Also, a public endpoint is not required in this scenario.

Assuming that you have configured DNS, you can connect to your SQL Server instance by specifying the SQL Server VM hostname in the connection string. The following example assumes that Windows Authentication has also been configured and that the user has been granted access to the SQL Server instance.

    "Server=mysqlvm;Integrated Security=true" 

Note that in this scenario, you could also specify the IP address of the VM.

## Steps for configuring SQL Server connectivity in an Azure VM
The following steps demonstrate how to connect to the SQL Server instance over the internet using SQL Server Management Studio (SSMS). However, the same steps apply to making your SQL Server virtual machine accessible for your applications, running both on-premises and in Azure.

Before you can connect to the instance of SQL Server from another VM or the internet, you must complete the following tasks as described in the sections that follow:

* [Create a TCP endpoint for the virtual machine](#create-a-tcp-endpoint-for-the-virtual-machine.md)
* [Open TCP ports in the Windows firewall](#open-tcp-ports-in-the-windows-firewall-for-the-default-instance-of-the-database-engine.md)
* [Configure SQL Server to listen on the TCP protocol](#configure-sql-server-to-listen-on-the-tcp-protocol.md)
* [Configure SQL Server for mixed mode authentication](#configure-sql-server-for-mixed-mode-authentication.md)
* [Create SQL Server authentication logins](#create-sql-server-authentication-logins.md)
* [Determine the DNS name of the virtual machine](#determine-the-dns-name-of-the-virtual-machine.md)
* [Connect to the Database Engine from another computer](#connect-to-the-database-engine-from-another-computer.md)

The connection path is summarized by the following diagram:

![Connecting to a SQL Server virtual machine](../../includes/media/virtual-machines-sql-server-connection-steps/SQLServerinVMConnectionMap.png)

### Create a TCP endpoint for the virtual machine

In order to access SQL Server from the internet, the virtual machine must have an endpoint to listen for incoming TCP communication. This Azure configuration step, directs incoming TCP port traffic to a TCP port that is accessible to the virtual machine.

>[AZURE.NOTE] If you are connecting within the same cloud service or virtual network, you do not have to create a publically accessible endpoint. In that case, you could continue to the next step. For more information, see [Connection Scenarios](../articles/virtual-machines/virtual-machines-sql-server-connectivity.md#connection-scenarios).

1. On the Azure Management Portal, click on **VIRTUAL MACHINES**.
	
2. Click on your newly created virtual machine. Information about your virtual machine is presented.
	
3. Near the top of the page, select the **ENDPOINTS** page, and then at the bottom of the page, click **ADD**.
	
4. On the **Add an Endpoint to a Virtual Machine** page, click **Add a Stand-alone Endpoint**, and then click the Next arrow to continue.
	
5. On the **Specify the details of the endpoint** page, provide the following information.

	- In the **NAME** box, provide a name for the endpoint.
	- In the **PROTOCOL** box, select **TCP**. You may type **57500** in the **PUBLIC PORT** box. Similarly, you may type SQL Server's default listening port **1433** in the **Private Port** box. Note that many organizations select different port numbers to avoid malicious security attacks. 

6. Click the check mark to continue. The endpoint is created.


### Open TCP ports in the Windows firewall for the default instance of the Database Engine

1. Connect to the virtual machine via Windows Remote Desktop. Once logged in, at the Start screen, type **WF.msc**, and then hit ENTER. 

	![Start the Firewall Program](./media/virtual-machines-sql-server-connection-steps/12Open-WF.png)
2. In the **Windows Firewall with Advanced Security**, in the left pane, right-click **Inbound Rules**, and then click **New Rule** in the action pane.

	![New Rule](./media/virtual-machines-sql-server-connection-steps/13New-FW-Rule.png)

3. In the **New Inbount Rule Wizard** dialog box, under **Rule Type**, select **Port**, and then click **Next**.

4. In the **Protocol and Ports** dialog, use the default **TCP**. In the **Specific local ports** box, then type the port number of the instance of the Database Engine (**1433** for the default instance or your choice for the private port in the endpoint step). 

	![TCP Port 1433](./media/virtual-machines-sql-server-connection-steps/14Port-1433.png)

5. Click **Next**.

6. In the **Action** dialog box, select **Allow the connection**, and then click **Next**.

	**Security Note:** Selecting **Allow the connection if it is secure** can provide additional security. Select this option if you want to configure additional security options in your environment.

	![Allow Connections](./media/virtual-machines-sql-server-connection-steps/15Allow-Connection.png)

7. In the **Profile** dialog box, select **Public**, **Private**, and **Domain**. Then click **Next**. 

    **Security Note:**  Selecting **Public** allows access over the internet. Whenever possible, select a more restrictive profile.

	![Public Profile](./media/virtual-machines-sql-server-connection-steps/16Public-Private-Domain-Profile.png)

8. In the **Name** dialog box, type a name and description for this rule, and then click **Finish**.

	![Rule Name](./media/virtual-machines-sql-server-connection-steps/17Rule-Name.png)

Open additional ports for other components as needed. For more information, see [Configuring the Windows Firewall to Allow SQL Server Access](http://msdn.microsoft.com/library/cc646023.aspx).


### Configure SQL Server to listen on the TCP protocol

1. While connected to the virtual machine, on the Start page, type **SQL Server Configuration Manager** and hit ENTER.
	
	![Open SSCM](./media/virtual-machines-sql-server-connection-steps/9Click-SSCM.png)

2. In SQL Server Configuration Manager, in the console pane, expand **SQL Server Network Configuration**.

3. In the console pane, click **Protocols for MSSQLSERVER** (he default instance name.) In the details pane, right-click TCP, it should be Enabled for the gallery images by default. For your custom images, click **Enable** (if its status is Disabled.)

	![Enable TCP](./media/virtual-machines-sql-server-connection-steps/10Enable-TCP.png)

5. In the console pane, click **SQL Server Services**. In the details pane, right-click **SQL Server (_instance name_)** (the default instance is **SQL Server (MSSQLSERVER)**), and then click **Restart**, to stop and restart the instance of SQL Server. 

	![Restart Database Engine](./media/virtual-machines-sql-server-connection-steps/11Restart.png)

7. Close SQL Server Configuration Manager.

For more information about enabling protocols for the SQL Server Database Engine, see [Enable or Disable a Server Network Protocol](http://msdn.microsoft.com/library/ms191294.aspx).

### Configure SQL Server for mixed mode authentication

The SQL Server Database Engine cannot use Windows Authentication without domain environment. To connect to the Database Engine from another computer, configure SQL Server for mixed mode authentication. Mixed mode authentication allows both SQL Server Authentication and Windows Authentication.

>[AZURE.NOTE] Configuring mixed mode authentication might not be necessary if you have configured an Azure Virtual Network with a configured domain environment.

1. While connected to the virtual machine, on the Start page, type **SQL Server 2014 Management Studio** and click the selected icon.

	![Start SSMS](./media/virtual-machines-sql-server-connection-steps/18Start-SSMS.png)

	The first time you open Management Studio it must create the users Management Studio environment. This may take a few moments.

2. Management Studio presents the **Connect to Server** dialog box. In the **Server name** box, type the name of the virtual machine to connect to the Database Engine  with the Object Explorer. (Instead of the virtual machine name you can also use **(local)** or a single period as the **Server name**. Select **Windows Authentication**, and leave **_your_VM_name_\your_local_administrator** in the **User name** box. Click **Connect**.

	![Connect to Server](./media/virtual-machines-sql-server-connection-steps/19Connect-to-Server.png)

3. In SQL Server Management Studio Object Explorer, right-click the name of the instance of SQL Server (the virtual machine name), and then click **Properties**.

	![Server Properties](./media/virtual-machines-sql-server-connection-steps/20Server-Properties.png)

4. On the **Security** page, under **Server authentication**, select **SQL Server and Windows Authentication mode**, and then click **OK**.

	![Select Authentication Mode](./media/virtual-machines-sql-server-connection-steps/21Mixed-Mode.png)

5. In the SQL Server Management Studio dialog box, click **OK** to acknowledge the requirement to restart SQL Server.

6. In Object Explorer, right-click your server, and then click **Restart**. (If SQL Server Agent is running, it must also be restarted.)

	![Restart](./media/virtual-machines-sql-server-connection-steps/22Restart2.png)

7. In the SQL Server Management Studio dialog box, click **Yes** to agree that you want to restart SQL Server.

### Create SQL Server authentication logins

To connect to the Database Engine from another computer, you must create at least one SQL Server authentication login.

1. In SQL Server Management Studio Object Explorer, expand the folder of the server instance in which you want to create the new login.

2. Right-click the **Security** folder, point to **New**, and select **Login...**.

	![New Login](./media/virtual-machines-sql-server-connection-steps/23New-Login.png)

3. In the **Login - New** dialog box, on the **General** page, enter the name of the new user in the **Login name** box.

4. Select **SQL Server authentication**.

5. In the **Password** box, enter a password for the new user. Enter that password again into the **Confirm Password** box.

6. To enforce password policy options for complexity and enforcement, select **Enforce password policy** (recommended). This is a default option when SQL Server authentication is selected.

7. To enforce password policy options for expiration, select **Enforce password expiration** (recommended). Enforce password policy must be selected to enable this checkbox. This is a default option when SQL Server authentication is selected.

8. To force the user to create a new password after the first time the login is used, select **User must change password at next login** (Recommended if this login is for someone else to use. If the login is for your own use, do not select this option.) Enforce password expiration must be selected to enable this checkbox. This is a default option when SQL Server authentication is selected. 

9. From the **Default database** list, select a default database for the login. **master** is the default for this option. If you have not yet created a user database, leave this set to **master**.

10. In the **Default language** list, leave **default** as the value.
    
	![Login Properties](./media/virtual-machines-sql-server-connection-steps/24Test-Login.png)

11. If this is the first login you are creating, you may want to designate this login as a SQL Server administrator. If so, on the **Server Roles** page, check **sysadmin**. 

	**Security Note:** Members of the sysadmin fixed server role have complete control of the Database Engine. You should carefully restrict membership in this role.

	![sysadmin](./media/virtual-machines-sql-server-connection-steps/25sysadmin.png)

12. Click OK.

For more information about SQL Server logins, see [Create a Login](http://msdn.microsoft.com/library/aa337562.aspx).


### Determine the DNS name of the virtual machine

To connect to the SQL Server Database Engine from another computer, you must know the Domain Name System (DNS) name of the virtual machine. (This is the name the internet uses to identify the virtual machine. You can use the IP address, but the IP address might change when Azure moves resources for redundancy or maintenance. The DNS name will be stable because it can be redirected to a new IP address.)  

1. In the Azure Management Portal (or from the previous step), select **VIRTUAL MACHINES**. 

2. On the **VIRTUAL MACHINE INSTANCES** page, under the **Quick Glance** column, find and copy the DNS name for the virtual machine.

	![DNS name](./media/virtual-machines-sql-server-connection-steps/sql-vm-dns-name.png)
	

### Connect to the Database Engine from another computer
 
1. On a computer connected to the internet, open SQL Server Management Studio.
2. In the **Connect to Server** or **Connect to Database Engine** dialog box, in the**Server name** box, enter the DNS name of the virtual machine (determined in the previous task) and a public endpoint port number in the format of *DNSName,portnumber* such as **tutorialtestVM.cloudapp.net,57500**.
To get the port number, log in to the Azure Management Portal and find the Virtual Machine. On the Dashboard, click **ENDPOINTS** and use the **PUBLIC PORT** assigned to **MSSQL**.
	![Public Port](./media/virtual-machines-sql-server-connection-steps/sql-vm-port-number.png)
3. In the **Authentication** box, select **SQL Server Authentication**.
5. In the **Login** box, type the name of a login that you created in an earlier task.
6. In the **Password** box, type the password of the login that you create in an earlier task.
7. Click **Connect**.

	![Connect using SSMS](./media/virtual-machines-sql-server-connection-steps/33Connect-SSMS.png)


## Next Steps
To see provisioning instructions along with these connectivity steps, see [Provisioning a SQL Server Virtual Machine on Azure](virtual-machines-provision-sql-server.md).

If you are also planning to use AlwaysOn Availability Groups for high availability and disaster recovery, you should consider implementing a listener. Database clients connect to the listener rather than directly to one of the SQL Server instances. The listener routes clients to the primary replica in the availability group. For more information, see [Configure an ILB listener for AlwaysOn Availability Groups in Azure](virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener.md).

It is important to review all of the security best practices for SQL Server running on an Azure virtual machine. For more information, see [Security Considerations for SQL Server in Azure Virtual Machines](virtual-machines-sql-server-security-considerations.md).

For other topics related to running SQL Server in Azure VMs, see [SQL Server on Azure Virtual Machines](virtual-machines-sql-server-infrastructure-services.md). 

