<properties 
   pageTitle="Create HDInsight Hadoop clusters with Azure Data Lake Store using the portal | Azure" 
   description="Use Azure Portal to create and use HDInsight Hadoop clusters with Azure Data Lake Store" 
   services="data-lake-store" 
   documentationCenter="" 
   authors="nitinme" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
   ms.service="data-lake-store"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="01/20/2016"
   ms.author="nitinme"/>

# Create an HDInsight cluster with Data Lake Store using Azure Portal
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [Using Portal](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Using PowerShell](data-lake-store-hdinsight-hadoop-use-powershell.md)
> 
> 
Learn how to use Azure Portal to create an HDInsight cluster (Hadoop, HBase, or Storm) with access to Azure Data Lake Store. Some important considerations for this release:

* **For Hadoop clusters (Windows and Linux)**, the Data Lake Store can only be used as an additional storage account. The default storage account for the such clusters will still be Azure Storage Blobs (WASB).

* **For Storm clusters (Windows and Linux)**, the Data Lake Store can be used to write data from a Storm topology. Data Lake Store can also be used to store reference data that can then be read by a Storm topology.

* **For HBase clusters (Windows and Linux)**, the Data Lake Store can be used as a default storage or additional storage.


In this article, we provision a Hadoop cluster with Data Lake Store as additional storage. Configuring HDInsight to work with Data Lake Store using the Azure Portal involves the following steps:

* Create an HDInsight cluster with authentication to an Azure Active Directory Service Principal
* Configure Data Lake Store access using the same Service Principal
* Run a test job on the cluster

## Prerequisites
Before you begin this tutorial, you must have the following:

* **An Azure subscription**. See [Get Azure free trial](https://azure.microsoft.com/pricing/free-trial/).
* **Enable your Azure subscription** for Data Lake Store Public Preview. See [instructions](data-lake-store-get-started-portal.md#signup).

## Create an HDInsight cluster with authentication to Azure Active Directory Service Principal
In this section, you create an HDInsight Hadoop cluster that uses the Data Lake Store as an additional storage. In this release, for a Hadoop cluster, Data Lake Store can only be used as an additional storage for the cluster. The default storage will still be the Azure storage blobs (WASB). So, we'll first create the storage account and storage containers required for the cluster.

1. Sign on to the new [Azure Portal](https://portal.azure.com).

2. Follow the steps at [Create Hadoop clusters in HDInsight](../hdinsight/hdinsight-provision-clusters.md#create-using-the-preview-portal) to start provisioning an HDInsight cluster.

3. On the **Optional Configuration** blade, click **Data Source**. In the **Data Source** blade, specify the details for the storage account and storage container, specify **Location** as **East US 2**, and then click **Cluster AAD Identity**.

    ![Add service principal to HDInsight cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.1.png "Add service principal to HDInsight cluster")

4. On the **Cluster AAD Identity** blade, you can choose to select an existing Service Principal or create a new one. 

   * **Create a new Service Principal**. In the **Cluster AAD Identity** blade, click **Create new**. Click **Service Principal**, and then in the **Create a Service Principal** blade, provide values to create a new service principal. As part of that, a certificate and an Azure Active Directory application is also created. Click **Create**.

    ![Add service principal to HDInsight cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.4.png "Add service principal to HDInsight cluster")

    On the **Cluster AAD Identity** blade, click **Select** to proceed with the service principal that will be created.

    ![Add service principal to HDInsight cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.5.png "Add service principal to HDInsight cluster")



    * **Choose an existing Service Principal**. In the **Cluster AAD Identity** blade, click **Use existing**. Click **Service Principal**, and then in the **Select a Service Principal** blade, search for an existing service principal. Click a service principal name and then click **Select**.

        ![Add service principal to HDInsight cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.2.png "Add service principal to HDInsight cluster")

        On the **Cluster AAD Identity** blade, upload the certificate (.pfx) you created earlier and provide the password you used to create the certificate. Click **Select**. This completes the Azure Active Directory configuration for HDInsight cluster.

        ![Add service principal to HDInsight cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.3.png "Add service principal to HDInsight cluster")

1. Click **Select** on the **Data Source** blade and continue with cluster provisioning as described at [Create Hadoop clusters in HDInsight](../hdinsight/hdinsight-provision-clusters.md#create-using-the-preview-portal).

2. Once the cluster is provisioned, you can verify that the Service Principal is associated with the HDInsight cluster. To do so, from the cluster blade, click **Settings**, click **Cluster AAD Identity**, and you should see the associated Service Principal.

    ![Add service principal to HDInsight cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/hdi.adl.6.png "Add service principal to HDInsight cluster")


## <a name="acl"></a>Configure service principal to access Data Lake Store file system
1. Sign on to the new [Azure Portal](https://portal.azure.com).

2. If you do not have a Data Lake Store account, create one. Follow the instructions at [Get started with Azure Data Lake Store using the Azure Portal](data-lake-store-get-started-portal.md).

    If you already have a Data Lake Store account, from the left pane, click **Browse**, click **Data Lake Store**, and then click the account name to which you want to grant access.

    Perform the following tasks under your Data Lake Store account. 

   * [Create a folder in your Data Lake Store](data-lake-store-get-started-portal.md#createfolder).
* [Upload a file to you Data Lake Store](data-lake-store-get-started-portal.md#uploaddata). If you are looking for some sample data to upload, you can get the **Ambulance Data** folder from the [Azure Data Lake Git Repository](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData).

  You will use the uploaded files later when you test the Data Lake Store account with the HDInsight cluster.


3. In the Data Lake Store blade, click **Data Explorer**.

    ![Data Explorer](./media/data-lake-store-hdinsight-hadoop-use-portal/adl.start.data.explorer.png "Data Explorer")

4. In the **Data Explorer** blade, click the root of your account, and then in your account blade, click the **Access** icon.

    ![Set ACLs on Data Lake file system](./media/data-lake-store-hdinsight-hadoop-use-portal/adl.acl.1.png "Set ACLs on Data Lake file system")

5. The **Access** blade lists the standard access and custom access already assigned to the root. Click the **Add** icon to add custom-level ACLs and include the service principal you created earlier.

    ![List standard and custom access](./media/data-lake-store-hdinsight-hadoop-use-portal/adl.acl.2.png "List standard and custom access")

6. Click the **Add** icon to open the **Add Custom Access** blade. In this blade, click **Select User or Group**, and then in **Select User or Group** blade, search for the service principal you created earlier in Azure Active Directory. The name of service principal created earlier is **HDIADL**. Click the service principal name and then click **Select**.

    ![Add a group](./media/data-lake-store-hdinsight-hadoop-use-portal/adl.acl.3.png "Add a group")

7. Click **Select Permissions**, select the permissions you want to assign to the service principal, and then click **OK**.

    ![Assign permissions to group](./media/data-lake-store-hdinsight-hadoop-use-portal/adl.acl.4.png "Assign permissions to group")

8. In the **Add Custom Access** blade, click **OK**. The newly added group, with the associated permissions, will now be listed in the **Access** blade.

    ![Assign permissions to group](./media/data-lake-store-hdinsight-hadoop-use-portal/adl.acl.5.png "Assign permissions to group")

9. If required, you can also modify the access permissions after you have added the service principal. Clear or select the check box for each permission type (Read, Write, Execute) based on whether you want to remove or assign that permission. Click **Save** to save the changes, or **Discard** to undo the changes.


## Run test jobs on the HDInsight cluster to use the Azure Data Lake Store
After you have configured an HDInsight cluster, you can run test jobs on the cluster to test that the HDInsight cluster can access data in Azure Data Lake Store. To do so, we will run some hive queries that target the Data Lake Store.

### For a Linux cluster
1. Open the cluster blade for the cluster that you just provisioned and then click **Dashboard**. This opens Ambari for the Linux cluster. When accessing Ambari, you will be prompted to authenticate to the site. Enter the admin (default admin,) account name and password you used when creating the cluster.

    ![Launch cluster dashboard](./media/data-lake-store-hdinsight-hadoop-use-portal/hdiadlcluster1.png "Launch cluster dashboard")

    You can also navigate directly to Ambari by going to https://CLUSTERNAME.azurehdinsight.net in a web browser (where **CLUSTERNAME** is the name of your HDInsight cluster).

2. Open the Hive view. Select the set of squares from the page menu (next to the **Admin** link and button on the right of the page,) to list available views. Select the **Hive** view.

    ![Selecting ambari views](./media/data-lake-store-hdinsight-hadoop-use-portal/selecthiveview.png) 

3. You should see a page similar to the following:

    ![Image of the hive view page, containing a query editor section](./media/data-lake-store-hdinsight-hadoop-use-portal/hiveview.png)

4. In the **Query Editor** section of the page, paste the following HiveQL statement into the worksheet:

        CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://mydatalakestore.azuredatalakestore.net:443/mynewfolder'
5. Click the **Execute** button at the bottom of the **Query Editor** to start the query. A **Query Process Results** section should appear beneath the **Query Editor** and display information about the job.

6. Once the query has finished, the **Query Process Results** section will display the results of the operation. The **Results** tab should contain the following information:

7. Run the following query to verify that the table was created.

        SHOW TABLES;

    The **Results** tab should show the following:

        hivesampletable
     vehicles

    **vehicles** is the table you created earlier. **hivesampletable** is a sample table available in all HDInsight clusters by default.

8. You can also run a query to retrieve data from the **vehicles** table.

        SELECT * FROM vehicles LIMIT 5;


### For a Windows cluster
1. Open the cluster blade for the cluster that you just provisioned and then click **Dashboard**.

    ![Launch cluster dashboard](./media/data-lake-store-hdinsight-hadoop-use-portal/hdiadlcluster1.png "Launch cluster dashboard")

    When prompted, enter the administrator credentials for the cluster.

2. This opens the Microsoft Azure HDInsight Query Console. Click **Hive Editor**.

    ![Open Hive editor](./media/data-lake-store-hdinsight-hadoop-use-portal/hdiadlcluster2.png "Open Hive editor")

3. In the Hive Editor, enter the following query and then click **Submit**.

        CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://mydatalakestore.azuredatalakestore.net:443/mynewfolder'

    In this Hive query, we create a table from data stored in Data Lake Store at `adl://mydatalakestore.azuredatalakestore.net:443/mynewfolder`. This location has a sample data file that you should have uploaded earlier. 

    The **Job Session** table at the bottom shows the status of the job changing from **Initializing**, to **Running**, to **Completed**. You can also click **View Details** to see more information about the completed job.

    ![Create table](./media/data-lake-store-hdinsight-hadoop-use-portal/hdiadlcluster3.png "Create table")

4. Run the following query to verify that the table was created.

        SHOW TABLES;

    Click **View Details** corresponding to this query and the output should show the following:

        hivesampletable
     vehicles

    **vehicles** is the table you created earlier. **hivesampletable** is a sample table available in all HDInsight clusters by default.

5. You can also run a query to retrieve data from the **vehicles** table.

        SELECT * FROM vehicles LIMIT 5;



## Access Data Lake Store using HDFS commands
Once you have configured the HDInsight cluster to use Data Lake Store, you can use the HDFS shell commands to access the store.

### For a Linux cluster
In this section you will SSH into the cluster and run the HDFS commands. Windows does not provide a built-in SSH client. We recommend using **PuTTY**, which can be downloaded from [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

For more information on using PuTTY, see [Use SSH with Linux-based Hadoop on HDInsight from Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).

Once connected, use the following HDFS filesystem command to list the files in the Data Lake Store.

    hdfs dfs -ls adl://<Data Lake Store account name>.azuredatalakestore.net:443/

This should list the file that you uploaded earlier to the Data Lake Store.

    15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
    Found 1 items
    -rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestore.azuredatalakestore.net:443/mynewfolder

You can also use the `hdfs dfs -put` command to upload some files to the Data Lake Store, and then use `hdfs dfs -ls` to verify whether the files were successfully uploaded.

### For a Windows cluster
1. Sign on to the new [Azure Portal](https://portal.azure.com).

2. Click **Browse**, click **HDInsight clusters**, and then click the HDInsight cluster that you created.

3. In the cluster blade, click **Remote Desktop**, and then in the **Remote Desktop** blade, click **Connect**.

    ![Remote into HDI cluster](./media/data-lake-store-hdinsight-hadoop-use-portal/ADL.HDI.PS.Remote.Desktop.png "Create an Azure Resource Group")

    When prompted, enter the credentials you provided for the remote desktop user. 

4. In the remote session, start Windows PowerShell, and use the HDFS filesystem commands to list the files in the Azure Data Lake Store.

         hdfs dfs -ls adl://<Data Lake Store account name>.azuredatalakestore.net:443/

    This should list the file that you uploaded earlier to the Data Lake Store.

        15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
     Found 1 items
     -rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestore.azuredatalakestore.net:443/mynewfolder

    You can also use the `hdfs dfs -put` command to upload some files to the Data Lake Store, and then use `hdfs dfs -ls` to verify whether the files were successfully uploaded.


## Considerations for provisioning HBase cluster that use Data Lake Store as default storage
For HBase clusters, you can use Data Lake Store accounts as default storage. If you choose to do so, to provision a cluster successfully, the service principal associated with the cluster **must** have access to the Data Lake Store account. You can ensure that in two ways:

* **If you use an existing service principal**, you must ensure that the service principal is added to the ACL at the root-level of the Data Lake Store file system before you start provisioning the cluster.
* **If you opt to create a new service principal** as part of cluster provisioning, as soon as the cluster starts provisioning, you must add the newly created service principal to the root-level of the Data Lake Store file system. If you fail to do so, the cluster will be provisioned but the HBase services will fail to start. To workaround this issue, you must then add the service principal to the ACL of the Data Lake Store account and then restart the HBase services using Ambari Web UI.

For instructions on how to add a service principal to a Data Lake Store file system, see [Configure service principal to access Data Lake Store file system](#acl.md).

## Use Data Lake Store in a Storm topology
You can use the Data Lake Store to write data from a Storm topology. For instructions on how to achieve this scenario, see [Use Azure Data Lake Store with Apache Storm with HDInsight](../hdinsight/hdinsight-storm-write-data-lake-store.md).

## See also
* [PowerShell: Create an HDInsight cluster to use Data Lake Store](data-lake-store-hdinsight-hadoop-use-powershell.md)

[makecert]: https://msdn.microsoft.com/library/windows/desktop/ff548309(v=vs.85).aspx
[pvk2pfx]: https://msdn.microsoft.com/library/windows/desktop/ff550672(v=vs.85).aspx
