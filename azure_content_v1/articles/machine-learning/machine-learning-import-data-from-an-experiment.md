<properties
    pageTitle="Import data into Machine Learning Studio ferom another experiment | Microsoft Azure"
    description="How to save training data in Azure Machine Learning Studio and use it in another experiment."
    keywords="import data,data,data sources,training data"
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
    ms.date="10/07/2015"
    ms.author="garye;bradsev" />


# Import your data into Azure Machine Learning Studio from another experiment
> [AZURE.SELECTOR]
- [Overview & data formats](../articles/machine-learning/machine-learning-data-science-import-data.md)
- [Import data from local files](../articles/machine-learning/machine-learning-import-data-from-local-file.md)
- [Import data from online sources](../articles/machine-learning/machine-learning-import-data-from-online-sources.md)
- [Import data from an experiment](../articles/machine-learning/machine-learning-import-data-from-an-experiment.md)


There will be times when you'll want to take an intermediate result from an experiment and use it as part of another experiment.  To do this, you save the module as a dataset:

1. Right-click the output of the module that you want to save as a dataset.

2. Click **Save as Dataset**.

3. When prompted, enter a name and a description that would allow you to identify the dataset easily.

4. Click the **OK** checkmark.


When the save finishes, the dataset will be available for use within any experiment in your workspace. You can find it in the **Saved Datasets** list in the module palette.

