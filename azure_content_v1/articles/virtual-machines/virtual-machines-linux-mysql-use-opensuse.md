<properties
    pageTitle="Install MySQL on a OpenSUSE Linux VM in Microsoft Azure"
    description="Learn to install MySQL on a virtual machine in Azure."
    services="virtual-machines"
    documentationCenter=""
    authors="cynthn"
    manager="timlt"
    editor=""
    tags="azure-service-management"/>

<tags
    ms.service="virtual-machines"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/21/2016"
    ms.author="cynthn"/>

# Install MySQL on a virtual machine running OpenSUSE Linux in Azure
[MySQL](http://www.mysql.com) is a popular, open-source SQL database. This tutorial shows you how to create a virtual machine running OpenSUSE Linux, then install MySQL.

> [AZURE.IMPORTANT] Azure has two different deployment models for creating and working with resources:  [Resource Manager and classic](../resource-manager-deployment-model.md).  This article covers using the classic deployment model. Microsoft recommends that most new deployments use the Resource Manager model.

<br>

## Create a virtual machine running OpenSUSE Linux
<properties writer="kathydav" editor="tysonn" manager="timlt" />

1. Sign in to the [Azure classic portal](http://manage.windowsazure.com).  Check out the [Free Trial](https://azure.microsoft.com/pricing/free-trial/) offer if you don't have a subscription yet.

2. On the command bar at the bottom of the window, click **New**.

3. Under **Compute**, click **Virtual Machine**, and then click **From Gallery**.

	![Create a New Virtual Machine][Image1]

4. Under the **SUSE** group, select an OpenSUSE virtual machine image, and then click arrow to continue.

5. On the first **Virtual machine configuration** page:

	- Type a **Virtual Machine Name**, such as "testlinuxvm". The name must contain between 3 and 15 characters, can contain only letters, numbers, and hyphens, and must start with a letter and end with either a letter or number.

	- Verify the **Tier** and pick a **Size**. The tier determines the sizes you can choose from. The size affects the cost of using it, as well as configuration options such as how many data disks you can attach. For details, see [Sizes for virtual machines](../articles/virtual-machines-size-specs.md).
	- Type a **New User Name**, or accept the default, **azureuser**. This name is added to the Sudoers list file.
	- Decide which type of **Authentication** to use. For general password guidelines, see [Strong passwords](http://msdn.microsoft.com/library/ms161962.aspx).

6. On the next **Virtual machine configuration** page:

	- Use the default **Create a new cloud service**.
	- In the **DNS Name** box, type a unique DNS name to use as part of the address, such as "testlinuxvm".
	- In the **Region/Affinity Group/Virtual Network** box, select a region where this virtual image will be hosted.
	- Under **Endpoints**, keep the SSH endpoint. You can add others now, or add, change, or delete them after the virtual machine is created.

	>[AZURE.NOTE] If you want a virtual machine to use a virtual network, you **must** specify the virtual network when you create the virtual machine. You can't add a virtual machine to a virtual network after you create the virtual machine. For more information, see [Virtual Network Overview](virtual-networks-overview.md).

7.	On the last **Virtual machine configuration** page, keep the default settings and then click the check mark to finish.

The portal lists the new virtual machine under **Virtual Machines**. While the status is reported as **(Provisioning)**, the virtual machine is being set up. When the status is reported as **Running**, you can move on to the next step.

##Connect to the Virtual Machine

You'll use SSH or PuTTY to connect to the virtual machine, depending on the operating system on the computer you'll connect from:

- From a computer running Linux, use SSH. At the command prompt, type:

	`$ ssh newuser@testlinuxvm.cloudapp.net -o ServerAliveInterval=180`

	Type the user's password.

- From a computer running Windows, use PuTTY. If you don't have it installed, download it from the [PuTTY Download Page][PuTTYDownload].

	Save **putty.exe** to a directory on your computer. Open a command prompt, navigate to that folder, and run **putty.exe**.

	Type the host name, such as "testlinuxvm.cloudapp.net", and type "22" for the **Port**.

	![PuTTY Screen][Image6]  

##Update the Virtual Machine (optional)

1. After you're connected to the virtual machine, you can optionally install system updates and patches. To run the update, type:

	`$ sudo zypper update`

2. Select **Software**, then **Online Update** to list available updates. Select **Accept** to start the installation and apply all new available patches (except the optional ones).

3. After installation is done, select **Finish**.  Your system is now up to date.

[PuTTYDownload]: http://www.puttyssh.org/download.html

[Image1]: ./media/create-and-configure-opensuse-vm-in-portal/CreateVM.png

[Image6]: ./media/create-and-configure-opensuse-vm-in-portal/putty.png


## Install and run MySQL on the virtual machine

1. To escalate privileges, type:

		sudo -s

	Enter your password.

2. To install MySQL Community Server edition, type:

		zypper install mysql-community-server

	Wait while MySQL downloads and installs.

3. To set MySQL to start when the system boots, type:

		insserv mysql

4. Start the MySQL daemon (mysqld) manually with this command:

		rcmysql start

	To check the status of the MySQL daemon, type:

		rcmysql status

	To stop the MySQL daemon, type:

		rcmysql stop

	> [AZURE.IMPORTANT] After installation, the MySQL root password is empty by default. We recommended that you run **mysql\_secure\_installation**, a script that helps secure MySQL. The script prompts you to change the MySQL root password, remove anonymous user accounts, disable remote root logins, remove test databases, and reload the privileges table. We recommended that you answer yes to all of these options and change the root password.

5. Type this to run the script MySQL installation script:

		mysql_secure_installation

6. Log in to MySQL:

		mysql -u root -p

	Enter the MySQL root password (which you changed in the previous step) and you'll be presented with a prompt where you can issue SQL statements to interact with the database.

7. To create a new MySQL user, run the following at the **mysql>** prompt:

		CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	Note, the semi-colons (;) at the end of the lines are crucial for ending the commands.

8. To create a database and grant the `mysqluser` user permissions on it, issue the following commands:

		CREATE DATABASE testdatabase;
		GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	Note that database user names and passwords are only used by scripts connecting to the database.  Database user account names do not necessarily represent actual user accounts on the system.

9. To log in from another computer, type:

		GRANT ALL ON testdatabase.* TO 'mysqluser'@'<ip-address>' IDENTIFIED BY 'password';

	where `ip-address` is the IP address of the computer from which you will connect to MySQL.

10. To exit the MySQL database administration utility, type:

		quit
		
## Add an endpoint

1. After MySQL is installed, you'll need to configure an endpoint to access MySQL remotely. Log in to the [Azure  classic portal][AzurePortal]. Click **Virtual Machines**, click the name of your new virtual machine, and then click **Endpoints**.

2. Click **Add** at the bottom of the page.


3. Add an endpoint named "MySQL" with protocol **TCP**, and **Public** and **Private** ports set to "3306".

4. To remotely connect to the virtual machine from your computer, type:

		mysql -u mysqluser -p -h <yourservicename>.cloudapp.net

	For example, using the virual machine we created in this tutorial, type this command:

		mysql -u mysqluser -p -h testlinuxvm.cloudapp.net

[MySQLDocs]: http://dev.mysql.com/doc/
[AzurePortal]: http://manage.windowsazure.com

[Image9]: ./media/install-and-run-mysql-on-opensuse-vm/LinuxVmAddEndpointMySQL.png


## Next steps
For details about MySQL, see the [MySQL Documentation](http://dev.mysql.com/doc/index-topic.html).

[MySQLDocs]: http://dev.mysql.com/doc/index-topic.html
[MySQL]: http://www.mysql.com

