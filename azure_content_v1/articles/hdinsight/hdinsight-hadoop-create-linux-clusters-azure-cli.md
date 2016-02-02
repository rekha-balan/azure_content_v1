<properties
       pageTitle="Create Hadoop, HBase, or Storm clusters on Linux in HDInsight using the cross-platform Azure CLI | Microsoft Azure"
       description="Learn how to create Linux-based HDInsight clusters using the cross-platform Azure CLI, Azure Resource Manager templates, and the Azure REST API. You can specify the cluster type (Hadoop, HBase, or Storm,) or use scripts to install custom components.."
       services="hdinsight"
       documentationCenter=""
       authors="Blackmist"
       manager="paulettm"
       editor="cgronlun"
    tags="azure-portal"/>

<tags
       ms.service="hdinsight"
       ms.devlang="na"
       ms.topic="article"
       ms.tgt_pltfrm="na"
       ms.workload="big-data"
       ms.date="01/22/2016"
       ms.author="larryfr"/>

# Create Linux-based clusters in HDInsight using the Azure CLI
> [AZURE.SELECTOR]
- [Windows-based](hdinsight-provision-clusters.md)
- [Overview](hdinsight-hadoop-provision-linux-clusters.md)
- [Azure portal](hdinsight-hadoop-create-linux-clusters-portal.md)
- [Azure CLI](hdinsight-hadoop-create-linux-clusters-azure-cli.md)
- [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
- [REST API (cURL)](hdinsight-hadoop-create-linux-clusters-curl-rest.md)
- [.NET SDK](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md)


The Azure CLI is a cross-platform command-line utility that allows you to manage Azure Services. It can be used, along with Azure Resource management templates, to create an HDInsight cluster, along with associated storage accounts and other services.

Azure Resource Management templates are JSON documents that describe a **resource group** and all resources in it (such as HDInsight.) This template based approach allows you to define all the resources that you need for HDInsight in one template, and to manage changes to the group as a whole through **deployments** that apply changes to the group.

The steps in this document walk through the process of creating a new HDInsight cluster using the Azure CLI and a template.

> [!IMPORTANT]
> The steps in this document use the default number of worker nodes (4) for an HDInsight cluster. If you plan on more than 32 worker nodes, either at cluster creation or by scaling the cluster after creation, then you must select a head node size with at least 8 cores and 14GB ram.
> 
> For more information on node sizes and associated costs, see [HDInsight pricing](https://azure.microsoft.com/pricing/details/hdinsight/).
> 
> 
## Prerequisites
* **An Azure subscription**. See [Get Azure free trial](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

* **Azure CLI**. For information on installing the CLI, see [Install the Azure CLI](../xplat-cli-install.md).


## Login to your Azure subscription
Follow the steps documented in [Connect to an Azure subscription from the Azure Command-Line Interface (Azure CLI)](../xplat-cli-connect.md) and connect to your subscription using the **login** method.

## Create a cluster
The following steps should be performed from a command-prompt, shell or terminal session after installing and configuring the Azure CLI.

1. Use the following command to authenticate to your Azure subscription:

        azure login

    You will be prompted to provide your name and password. If you have multiple Azure subscriptions, you can use `azure account set <subscriptionname>` to set the subscription that the Azure CLI commands will use.

2. Switch to Azure Resource Manager mode using the following command:

        azure config mode arm
3. Create a template for your HDInsight cluster. The following are some basic example templates:

   * [Linux-based cluster, using an SSH public key](https://github.com/Azure/azure-quickstart-templates/tree/master/hdinsight-linux-ssh-publickey)
* [Linux-based cluster, using a password for the SSH account](https://github.com/Azure/azure-quickstart-templates/tree/master/hdinsight-linux-ssh-password)

  Both of these templates also create the default Azure Storage Account used by HDInsight.

  The files you will need are the **azuredeploy.json** and **azuredeploy.parameters.json**. Copy these files locally before continuing.


4. Open the **azuredeploy.parameters.json** file in an editor, and provide values for the items in the `parameters` section:

   * **location**: The data center that the resources will be created in. You can view the `location` section in the **azuredeploy.json** file for a list of allowed locations.
* **clusterName**: The name of the HDInsight cluster. This name must be unique, or the deployment will fail.
* **clusterStorageAccountName**: The name of the Azure Storage Account that will be created for the HDInsight cluster. This name must be unique, or the deployment will fail.
* **clusterLoginPassword**: The password for the cluster admin user. This should be a secure password, as it is used to access web sites and REST services on the cluster.
* **sshUserName**: The name of the first SSH user to create for this cluster. SSH will be used to remotely access the cluster using this account.
* **sshPublicKey**: If you are using the template that requires an SSH public key, you must add your public key on this line. For more information on generating and working with public keys, see the following articles:

  * [Use SSH with Linux-based Hadoop on HDInsight from Linux, Unix, or OS X](hdinsight-hadoop-linux-use-ssh-unix.md)
* [Use SSH with Linux-based Hadoop on HDInsight from Windows](hdinsight-hadoop-linux-use-ssh-windows.md)

* **sshPassword**: If you are using the template that requires an SSH password, you must add a password on this line.

  Once you are done, save and close the file.


5. Use the following to create an empty resource group. Replace **RESOURCEGROUPNAME** with the name you wish to use for this group. Replace **LOCATION** with the data center that you want to create your HDInsight cluster in:

        azure group create RESOURCEGROUPNAME LOCATION

   > [!NOTE]
> If the location name contains spaces, put it in quotes. For example "South Central US".
> 
6. Use the following command to create the initial deployment for this resource group. Replace **PATHTOTEMPLATE** with the path to the **azuredeploy.json** template file. Replace **PATHTOPARAMETERSFILE** with the path to the **azuredeploy.parameters.json** file. Replace **RESOURCEGROUPNAME** with the name of the group you created in the previous step:

        azure group deployment create -f PATHTOTEMPLATE -e PATHTOPARAMETERSFILE -g RESOURCEGROUPNAME -n InitialDeployment

    Once the deployment has been accepted, you should see a message similar to `group deployment create command ok`.

7. It may take some time for the deployment to complete, around 15 minutes. you can view information about the deployment using the following command. Replace **RESOURCEGROUPNAME** with the name of the resource group used in the previous step:

        azure group log show -l RESOURCEGROUPNAME

    Once the deployment completes, the **Status** field will contain the value **Succeeded**.  If a failure occurs during deployment, you can get more information on the failure using the following command

        azure group log show -l -v RESOURCEGROUPNAME


## Next steps
Now that you have successfully created an HDInsight cluster, use the following to learn how to work with your cluster:

### Hadoop clusters
* [Use Hive with HDInsight](hdinsight-use-hive.md)
* [Use Pig with HDInsight](hdinsight-use-pig.md)
* [Use MapReduce with HDInsight](hdinsight-use-mapreduce.md)

### HBase clusters
* [Get started with HBase on HDInsight](hdinsight-hbase-tutorial-get-started-linux.md)
* [Develop Java applications for HBase on HDInsight](hdinsight-hbase-build-java-maven-linux.md)

### Storm clusters
* [Develop Java topologies for Storm on HDInsight](hdinsight-storm-develop-java-topology.md)
* [Use Python components in Storm on HDInsight](hdinsight-storm-develop-python-topology.md)
* [Deploy and monitor topologies with Storm on HDInsight](hdinsight-storm-deploy-monitor-topology-linux.md)

