<properties
    pageTitle="Prepare your environment to back up Windows Server or Windows Client | Microsoft Azure"
    description="Prepare your environment to backup windows by creating a backup vault, downloading credentials, and installing the backup agent."
    services="backup"
    documentationCenter=""
    authors="Jim-Parker"
    manager="jwhit"
    editor=""
    keywords="backup vault; backup agent; backup windows;"/>

<tags
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.date="01/22/2016"
    ms.author="trinadhk; jimpark; markgal"/>


# Prepare your environment to back up Windows machines
This article will help you prepare to back up a Windows Server or Windows Client to Azure. To do this, you need an Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](https://azure.microsoft.com/pricing/free-trial/).

If you've already done this, you can start [backing up your Windows machines](backup-azure-backup-windows-server.md). Otherwise, continue through the steps below to make sure your environment is ready.

> [!NOTE]
> Previously you needed to create or acquire a X.509 v3 certificate in order to register your backup server. Certificates are still supported, but now to make Azure vault registration with a server easier, you can generate a vault credential right from the Quick Start page.
> 
> 
## Before you start
To back up files and data from your Windows Server to Azure, you must first:

* **Create a Backup vault** — Create a vault in the [Azure Backup management portal](http://manage.windowsazure.com).
* **Download vault credentials** — From the Dashboard page in the Azure Backup vault, download the vault credentials that will be used to register the Windows machine to the backup vault.
* **Install the Azure Backup Agent and register the server** — From the Dashboard page, click the link to download the [Azure Backup agent](http://aka.ms/azurebackup_agent). Install the agent, and register the server to the backup vault using the vault credentials.

## Create a Backup Vault
To back up files and data from your Windows Server or System Center Data Protection Manager (SCDPM) to Azure or when backing up IaaS VMs in Azure, you must create a backup vault in the geographic region where you want to store the data.

### Video walkthrough

Here's a quick video of the process.

[AZURE.VIDEO azure-backup-vault-creation]

The following steps will walk you through the creation of the vault used to store backups.

### Creating a backup vault
1. Sign in to the [Management Portal](https://manage.windowsazure.com/)
2. Click **New** > **Data Services** > **Recovery Services** > **Backup Vault** and choose **Quick Create**.
3. For the **Name** parameter, enter a friendly name to identify the backup vault. This needs to be unique for each subscription.
4. For the **Region** parameter, select the geographic region for the backup vault. The choice determines the geographic region to which your backup data is sent. By choosing a geographic region close to your location, you can reduce the network latency when backing up to Azure.
5. Click on **Create Vault** to complete the workflow. It can take a while for the backup vault to be created. To check the status, you can monitor the notifications at the bottom of the portal.

    ![Creating Vault](./media/backup-create-vault-wgif/create-vault-wgif.gif)

6. After the backup vault has been created, a message tells you the vault has been successfully created. The vault is also listed in the resources for Recovery Services as **Active**.

### Azure Backup - Storage Redundancy Options

> [AZURE.IMPORTANT] The best time to identify your storage redundancy option is right after vault creation, and before any machines are registered to the vault. Once an item has been registered to the vault, the storage redundancy option is locked and cannot be modified.

Your business needs would determine the storage redundancy of the Azure Backup backend storage. If you are using Azure as a primary backup storage endpoint (e.g. you are backing up to Azure from a Windows Server), you should consider picking (the default) Geo-Redundant storage option. This is seen under the **Configure** option of your Backup vault.

![GRS](./media/backup-create-vault/grs.png)

#### Geo-Redundant Storage (GRS)
GRS maintains six copies of your data. With GRS, your data is replicated three times within the primary region, and is also replicated three times in a secondary region hundreds of miles away from the primary region, providing the highest level of durability. In the event of a failure at the primary region, by storing data in GRS, Azure Backup ensures that your data is durable in two separate regions.

#### Locally Redundant Storage (LRS)
Locally Redundant Storage (LRS) maintains three copies of your data. LRS is replicated three times within a single facility in a single region. LRS protects your data from normal hardware failures, but not from the failure of an entire Azure facility.

If you are using Azure as a tertiary backup storage endpoint (e.g. you are using SCDPM to have a local backup copy on-premises & using Azure for your long term retention needs), you should consider choosing Locally Redundant Storage from the **Configure** option of your Backup vault. This brings down the cost of storing data in Azure, while providing a lower level of durability for your data that might be acceptable for tertiary copies.

![LRS](./media/backup-create-vault/lrs.png)


## Using vault credentials to authenticate with the Azure Backup service

The on-premises server (Windows client or Windows Server or Data Protection Manager server) needs to be authenticated with a backup vault before it can back up data to Azure. The authentication is achieved using “vault credentials”. The concept of vault credentials is similar to the concept of a “publish settings” file which is used in Azure PowerShell.

### What is the vault credential file?

The vault credentials file is a certificate generated by the portal for each backup vault. The portal then uploads the public key to the Access Control Service (ACS). The private key of the certificate is made available to the user as part of the workflow which is given as an input in the machine registration workflow. This authenticates the machine to send backup data to an identified vault in the Azure Backup service.

The vault credential is used only during the registration workflow. It is the user’s responsibility to ensure that the vault credentials file is not compromised. If it falls in the hands of any rogue-user, the vault credentials file can be used to register other machines against the same vault. However, as the backup data is encrypted using a passphrase which belongs to the customer, existing backup data cannot be compromised. To mitigate this concern, vault credentials are set to expire in 48hrs. You can download the vault credentials of a backup vault any number of times – but only the latest vault credential file is applicable during the registration workflow.

### Download the vault credential file

The vault credential file is downloaded through a secure channel from the Azure portal. The Azure Backup service is unaware of the private key of the certificate and the private key is not persisted in the portal or the service. Use the following steps to download the vault credential file to a local machine.

1.  Sign in to the [Management Portal](https://manage.windowsazure.com/)
2.  Click on **Recovery Services** in the left navigation pane and select the backup vault which you have created. Click on the cloud icon to get to the Quick Start view of the backup vault.

    ![Quick view](./media/backup-download-credentials/quickview.png)

3.  On the Quick Start page, click **Download vault credentials**. The  portal generates the vault credential file, which is made available for download.

    ![Download](./media/backup-download-credentials/downloadvc.png)

4.  The portal will generate a vault credential using a combination of the vault name and the current date. Click **Save** to download the vault credentials to the local account's downloads folder, or select Save As from the Save menu to specify a location for the vault credentials.

### Note
- Ensure that the vault credentials is saved in a location which can be accessed from your machine. If it is stored in a file share/SMB, check for the access permissions.
- The vault credentials file is used only during the registration workflow.
- The vault credentials file expires after 48hrs and can be downloaded from the portal.
- Refer to the Azure Backup [FAQ](](../articles/backup/backup-azure-backup-faq.md) for any questions on the workflow.


## Download, install, and register the Azure Backup agent

After creating the Azure Backup vault, an agent should be installed on each of your Windows machines (Windows Server, Windows client, System Center Data Protection Manager server, or Azure Backup Server machine) that enables back up of data and applications to Azure.

1. Sign in to the [Management Portal](https://manage.windowsazure.com/)

2. Click **Recovery Services**, then select the backup vault that you want to register with a server. The Quick Start page for that backup vault appears.

    ![Quick start](./media/backup-install-agent/quickstart.png)

3. On the Quick Start page, click the **For Windows Server or System Center Data Protection Manager or Windows client** option under **Download Agent**. Click **Save** to copy it to the local machine.

    ![Save agent](./media/backup-install-agent/agent.png)

4. Once the agent is installed, double click MARSAgentInstaller.exe to launch the installation of the Azure Backup agent. Choose the installation folder and scratch folder required for the agent. The cache location specified must have free space which is at least 5% of the backup data.

5.	If you use a proxy server to connect to the internet, in the **Proxy configuration** screen, enter the proxy server details. If you use an authenticated proxy, enter the user name and password details in this screen.

6.	The Azure Backup agent installs .NET Framework 4.5 and Windows PowerShell (if it’s not available already) to complete the installation.

7.	Once the agent is installed, click the **Proceed to Registration** button to continue with the workflow.

    ![Register](./media/backup-install-agent/register.png)

8. In the vault credentials screen, browse to and select the vault credentials file which was previously downloaded.

    ![Vault credentials](./media/backup-install-agent/vc.png)

    The vault credentials file is valid only for 48 hrs (after it’s downloaded from the portal). If you encounter any error in this screen (e.g “Vault credentials file provided has expired”), login to the Azure portal and download the vault credentials file again.

    Ensure that the vault credentials file is available in a location which can be accessed by the setup application. If you encounter access related errors, copy the vault credentials file to a temporary location in this machine and retry the operation.

    If you encounter an invalid vault credential error (e.g “Invalid vault credentials provided") the file is either corrupted or does not have the latest credentials associated with the recovery service. Retry the operation after downloading a new vault credential file from the portal. This error is typically seen if the user clicks on the **Download vault credential** option in the Azure portal, in quick succession. In this case, only the second vault credential file is valid.

9. In the **Encryption setting** screen, you can either generate a passphrase or provide a passphrase (minimum of 16 characters). Remember to save the passphrase in a secure location.

    ![Encryption](./media/backup-install-agent/encryption.png)

    > [AZURE.WARNING] If the passphrase is lost or forgotten; Microsoft cannot help in recovering the backup data. The end user owns the encryption passphrase and Microsoft does not have visibility into the passphrase used by the end user. Please save the file in a secure location as it is required during a recovery operation.

10. Once you click the **Finish** button, the machine is registered successfully to the vault and you are now ready to start backing up to Microsoft Azure.

11. When using Microsoft Azure Backup standalone you can modify the settings specified during the registration workflow by clicking on the **Change Properties** option in the Azure Backup mmc snap in.

    ![Change Properties](./media/backup-install-agent/change.png)

    Alternatively, when using Data Protection Manager, you can modify the settings specified  during the registration workflow by clicking the **Configure** option by selecting **Online** under the **Management** Tab.

    ![Configure Azure Backup](./media/backup-install-agent/configure.png)


## Next steps
* [Backup Windows Server or Windows Client](backup-azure-backup-windows-server.md)
* [Manage Windows Server or Windows Client](backup-azure-manage-windows-server.md)
* [Restore Windows Server or Windows Client from Azure](backup-azure-restore-windows-server.md)
* [Azure Backup FAQ](backup-azure-backup-faq.md)
* [Azure Backup Forum](http://go.microsoft.com/fwlink/p/?LinkId=290933)

