<properties 
    pageTitle="Create a Machine Learning workspace | Microsoft Azure" 
    description="How to create a workspace for Azure Machine Learning Studio" 
    services="machine-learning" 
    documentationCenter="" 
    authors="garyericson" 
    manager="paulettm" 
    editor="cgronlun"/>

<tags 
    ms.service="machine-learning" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="10/13/2015" 
    ms.author="garye;bradsev"/>


# Create an Azure Machine Learning workspace
This menu links to topics that describe how to set up the various data science environments used by the Cortana Analytics Process (CAPS).

> [AZURE.SELECTOR]
- [Set up environment](../articles/machine-learning/machine-learning-data-science-environment-setup.md)
- [Azure storage-account](../articles/storage/storage-create-storage-account.md)
- [Data science VM on Azure](../articles/machine-learning/machine-learning-data-science-virtual-machines.md)
- [HDInsight (Hadoop) cluster](../articles/machine-learning/machine-learning-data-science-customize-hadoop-cluster.md)
- [Azure Machine Learning workspace](../articles/machine-learning/machine-learning-create-workspace.md)


To use Azure Machine Learning Studio, you need to have a Machine Learning workspace. This workspace contains the tools you need to create, manage, and publish experiments. 

>[AZURE.NOTE (Try Azure Machine Learning for free)]
>
>No credit card or Azure subscription needed. <a href="https://studio.azureml.net/?selectAccess=true&o=2" target="_blank">**Get started now >**</a>


## To create a workspace
1. Sign-in to your Microsoft Azure account.
2. In the Microsoft Azure services panel, click **MACHINE LEARNING**.

    ![Machine Learning service][1][Machine Learning service][1]Machine Learning service][1]1]

3. Click **+NEW** at the bottom of the window.

4. Click **DATA SERVICES**, then **MACHINE LEARNING**, then **QUICK CREATE**.

    ![Quick Create of new workspace][3][Quick Create of new workspace][3]Quick Create of new workspace][3]3]

5. Enter a **WORKSPACE NAME** for your workspace and specify the **WORKSPACE OWNER**. The workspace owner must be a valid Microsoft account (e.g., name@outlook.com).

   > [!NOTE]
> Later, you can share the experiments you're working on by inviting others to your workspace. You can do this in Machine Learning Studio on the **SETTINGS** page. You just need the Microsoft account or organizational account for each user.
> 
6. Specify the Azure **LOCATION**, then enter an existing Azure **STORAGE ACCOUNT** or select **Create a new storage account** to create a new one.

7. Click **CREATE AN ML WORKSPACE**.

After your Machine Learning workspace is created, you will see it listed on the **machine learning** page.

For information about managing your workspace, see [Manage an Azure Machine Learning workspace](machine-learning-manage-workspace.md).
If you encounter a problem creating your workspace, see [Troubleshooting guide: Create and connect to an Machine Learning workspace](machine-learning-troubleshooting-creating-ml-workspace.md).

[Manage an Azure Machine Learning workspace]: machine-learning-manage-workspace.md
[Troubleshooting guide: Create and connect to an Machine Learning workspace]: machine-learning-troubleshooting-creating-ml-workspace.md

<!-- ![List of Machine Learning workspaces][2] -->

<!--Anchors-->
[To create a workspace]To create a workspace]: #createworkspace

<!--Image references-->
[1]1]: media/machine-learning-create-workspace/cw1.png
[2]2]: media/machine-learning-create-workspace/cw2.png
[3]3]: media/machine-learning-create-workspace/cw3.png



<!--Link references-->

