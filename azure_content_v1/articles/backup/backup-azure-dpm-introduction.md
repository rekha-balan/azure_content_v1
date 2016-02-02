<properties
    pageTitle="Introduction to Azure DPM backup | Microsoft Azure"
    description="An introduction to backing up DPM servers using the Azure Backup service"
    services="backup"
    documentationCenter=""
    authors="giridharreddy"
    manager="jwhit"
    editor=""/>

<tags
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/10/2015"
    ms.author="giridham;jimpark"/>

# Preparing to back up workloads to Azure with DPM
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Azure Backup Server](backup-azure-microsoft-azure-backup.md)
> * [System Center DPM](backup-azure-dpm-introduction.md)
> 
> 
This article provides an introduction to using Microsoft Azure Backup to protect your System Center Data Protection Manager (DPM) servers and workloads. By reading it you’ll understand:

* How Azure DPM server backup works
* The prerequisites to achieve a smooth backup experience
* The typical errors encountered and how to deal with them
* Supported scenarios

System Center DPM backs up file and application data. Data backed up to DPM can be stored on tape, on disk, or backed up to Azure with Microsoft Azure Backup. DPM interacts with Azure Backup as follows:

* **DPM deployed as a physical server or on-premises virtual machine** — If DPM is deployed as a physical server or as an on-premises Hyper-V virtual machine you can back up data to an Azure Backup vault in addition to disk and tape backup.
* **DPM deployed as an Azure virtual machine** — From System Center 2012 R2 with Update 3, DPM can be deployed as an Azure virtual machine. If DPM is deployed as an Azure virtual machine you can back up data to Azure disks attached to the DPM Azure virtual machine, or you can offload the data storage by backing it up to an Azure Backup vault.

## Why backup your DPM servers?
The business benefits of using Azure Backup for backing up DPM servers include:

* For on-premises DPM deployment, you can use Azure backup as an alternative to long-term deployment to tape.
* For DPM deployments in Azure, Azure Backup allows you to offload storage from the Azure disk, allowing you to scale up by storing older data in Azure Backup and new data on disk.

## How does DPM server backup work?
To back up a virtual machine, first a point-in-time snapshot of the data is needed. The Azure Backup service initiates the backup job at the scheduled time, and triggers the backup extension to take a snapshot. The backup extension coordinates with the in-guest VSS service to achieve consistency, and invokes the blob snapshot API of the Azure Storage service once consistency has been reached. This is done to get a consistent snapshot of the disks of the virtual machine, without having to shut it down.

After the snapshot has been taken, the data is transferred by the Azure Backup service to the backup vault. The service takes care of identifying and transferring only the blocks that have changed from the last backup making the backups storage and network efficient. When the data transfer is completed, the snapshot is removed and a recovery point is created. This recovery point can be seen in the Azure management portal.

> [!NOTE]
> For Linux virtual machines, only file-consistent backup is possible.
> 
> 
## Prerequisites
Prepare Azure Backup to back up DPM data as follows:

1. **Create a Backup vault** — Create a vault in the Azure Backup console.
2. **Download vault credentials** — In Azure Backup, upload the management certificate you created to the vault.
3. **Install the Azure Backup Agent and register the server** — From Azure Backup, install the agent on each DPM server and register the DPM server in the backup vault.

## Create a Backup Vault
To back up files and data from Windows Server or Data Protection Manager (DPM) to Azure or when backing up IaaS VMs to Azure, you must create a backup vault in the geographic region where you want to store the data.

The following steps will walk you through the creation of the vault used to store backups.

1. Sign in to the [Management Portal](https://manage.windowsazure.com/)
2. Click **New** > **Data Services** > **Recovery Services** > **Backup Vault** and choose **Quick Create**.

    ![Create vault](./media/backup-create-vault/createvault1.png)

3. For the **Name** parameter, enter a friendly name to identify the backup vault. This needs to be unique for each subscription.

4. For the **Region** parameter, select the geographic region for the backup vault. The choice determines the geographic region to which your backup data is sent. By choosing a geographic region close to your location, you can reduce the network latency when backing up to Azure.

5. Click on **Create Vault** to complete the workflow. It can take a while for the backup vault to be created. To check the status, you can monitor the notifications at the bottom of the portal.

    ![Creatinging Vault](./media/backup-create-vault/creatingvault1.png)

6. After the backup vault has been created, a message tells you the vault has been successfully created. The vault is also listed in the resources for Recovery Services as **Active**.

    ![Creating Vault status](./media/backup-create-vault/backupvaultstatus1.png)


### Azure Backup - Storage Redundancy Options

> AZURE.IMPORTANT The best time to identify your storage redundancy option is right after vault creation, and before any machines are registered to the vault. Once an item has been registered to the vault, the storage redundancy option is locked and cannot be modified.

Your business needs would determine the storage redundancy of the Azure Backup backend storage. If you are using Azure as a primary backup storage endpoint (e.g. you are backing up to Azure from a Windows Server), you should consider picking (the default) Geo-Redundant storage option. This is seen under the **Configure** option of your Backup vault.

![GRS](./media/backup-create-vault/grs.png)

#### Geo-Redundant Storage (GRS)
GRS maintains six copies of your data. With GRS, your data is replicated three times within the primary region, and is also replicated three times in a secondary region hundreds of miles away from the primary region, providing the highest level of durability. In the event of a failure at the primary region, by storing data in GRS, Azure Backup ensures that your data is durable in two separate regions.

#### Locally Redundant Storage (LRS)
Loclly redundant storage (LRS) maintains three copies of your data. LRS is replicated three times within a single facility in a single region. LRS protects your data from normal hardware failures, but not from the failure of an entire Azure facility.

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


## Requirements (and limitations)
* DPM can be running as a physical server or a Hyper-V virtual machine installed on System Center 2012 SP1 or System Center 2012 R2. It can also be running as an Azure virtual machine running on System Center 2012 R2 with at least DPM 2012 R2 Update Rollup 3 or a Windows virtual machine in VMWare running on System Center 2012 R2 with at least Update Rollup 5.
* If you’re running DPM with System Center 2012 SP1 you should install Update Roll up 2 for System Center Data Protection Manager SP1. This is required before you can install the Azure Backup Agent.
* The DPM server should have Windows PowerShell and .Net Framework 4.5 installed.
* DPM can back up most workloads to Azure Backup. For a full list of what’s supported see the Azure Backup support items below.
* Data stored in Azure Backup can’t be recovered with the “copy to tape” option.
* You’ll need an Azure account with the Azure Backup feature enabled. If you don't have an account, you can create a free trial account in just a couple of minutes. Read about [Azure Backup pricing](https://azure.microsoft.com/pricing/details/backup/).
* Using Azure Backup requires the Azure Backup Agent to be installed on the servers you want to back up. Each server must have at least 2.5 GB of local free storage space for cache location, although 15 GB of free local storage space to be used for the cache location is recommended.
* Data will be stored in the Azure vault storage. There’s no limit to the amount of data you can back up to an Azure Backup vault but the size of a data source (for example a virtual machine or database) shouldn’t exceed 54400 GB.

These file types are supported for back up to Azure:

* Encrypted (Full backups only)
* Compressed (Incremental backups supported)
* Sparse (Incremental backups supported)
* Compressed and sparse (Treated as Sparse)

And these are unsupported:

* Servers on case-sensitive file systems aren’t supported.
* Hard links (Skipped)
* Reparse points (Skipped)
* Encrypted and compressed (Skipped)
* Encrypted and sparse (Skipped)
* Compressed stream
* Sparse stream

> [!NOTE]
> From in System Center 2012 DPM with SP1 onwards you can backup up workloads protected by DPM to Azure using Microsoft Azure Backup.
> 
> 
