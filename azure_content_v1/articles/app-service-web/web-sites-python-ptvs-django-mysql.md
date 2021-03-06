<properties 
    pageTitle="Django and MySQL on Azure with Python Tools 2.2 for Visual Studio" 
    description="Learn how to use the Python Tools for Visual Studio to create a Django web app that stores data in a MySQL database instance and deploy it to Azure App Service Web Apps." 
    services="app-service\web" 
    documentationCenter="python" 
    authors="huguesv" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="app-service-web" 
    ms.workload="web" 
    ms.tgt_pltfrm="na" 
    ms.devlang="python" 
    ms.topic="get-started-article" 
    ms.date="11/17/2015"
    ms.author="huvalo"/>

# Django and MySQL on Azure with Python Tools 2.2 for Visual Studio
> [AZURE.SELECTOR]AZURE.SELECTOR]
> 
> * [.Net](web-sites-dotnet-get-started.md)
> * [Node.js](web-sites-nodejs-develop-deploy-mac.md)
> * [Java](web-sites-java-get-started.md)
> * [PHP - Git](web-sites-php-mysql-deploy-use-git.md)
> * [PHP - FTP](web-sites-php-mysql-deploy-use-ftp.md)
> * [Python](web-sites-python-ptvs-django-mysql.md)
> 
> 
In this tutorial, we'll use [Python Tools for Visual Studio](http://aka.ms/ptvs) to create a simple polls web app using one of the PTVS sample templates. This tutorial is also available as a [video](https://www.youtube.com/watch?v=oKCApIrS0Lo).

We'll learn how to use a MySQL service hosted on Azure, how to configure the web app to use MySQL, and how to publish the web app to [Azure App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714).

See the [Python Developer Center](/develop/python/) for more articles that cover development of Azure App Service Web Apps with PTVS using Bottle, Flask and Django web frameworks, with MongoDB, Azure Table Storage, MySQL and SQL Database services. While this article focuses on App Service, the steps are similar when developing [Azure Cloud Services](../cloud-services-python-ptvs.md).

## Prerequisites
* Visual Studio 2013 or 2015
* [Python Tools 2.2 for Visual Studio](http://go.microsoft.com/fwlink/?LinkID=624025)
* [Python Tools 2.2 for Visual Studio Samples VSIX](http://go.microsoft.com/fwlink/?LinkID=624025)
* [Azure SDK Tools for VS 2013](http://go.microsoft.com/fwlink/?LinkId=323510) or [Azure SDK Tools for VS 2015](http://go.microsoft.com/fwlink/?LinkId=518003)
* [Python 2.7 32-bit](http://go.microsoft.com/fwlink/?LinkId=517190)


> [AZURE.NOTE]
> To complete this tutorial, you need an Azure account. You can <a href="/pricing/member-offers/msdn-benefits-details/" target="_blank">activate your Visual Studio subscriber benefits</a> or <a href="/pricing/free-trial/" target="_blank">sign up for a free trial</a>.


> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](http://go.microsoft.com/fwlink/?LinkId=523751), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.
> 
> 
## Create the Project
In this section, we'll create a Visual Studio project using a sample template. We'll create a virtual environment and install required packages. We'll create a local database using sqlite. Then we'll run the application locally.

1. In Visual Studio, select **File**, **New Project**.

2. The project templates from the PTVS Samples VSIX are available under **Python**, **Samples**. Select **Polls Django Web Project** and click OK to create the project.

     ![New Project Dialog](./media/web-sites-python-ptvs-django-mysql/PollsDjangoNewProject.png)

3. You will be prompted to install external packages. Select **Install into a virtual environment**.

     ![External Packages Dialog](./media/web-sites-python-ptvs-django-mysql/PollsDjangoExternalPackages.png)

4. Select **Python 2.7** as the base interpreter.

     ![Add Virtual Environment Dialog](./media/web-sites-python-ptvs-django-mysql/PollsCommonAddVirtualEnv.png)

5. Right-click the project node and select **Python**, **Django Sync DB**.

     ![Django Sync DB Command](./media/web-sites-python-ptvs-django-mysql/PollsDjangoSyncDB.png)

6. This will open a Django Management Console. Follow the prompts to create a user.

   This will create a sqlite database in the project folder.

     ![Django Management Console Window](./media/web-sites-python-ptvs-django-mysql/PollsDjangoConsole.png)

7. Confirm that the application works by pressing <kbd>F5</kbd>.

8. Click **Log in** from the navigation bar at the top.

     ![Web Browser](./media/web-sites-python-ptvs-django-mysql/PollsDjangoCommonBrowserLocalMenu.png)

9. Enter the credentials for the user you created when you synchronized the database.

     ![Web Browser](./media/web-sites-python-ptvs-django-mysql/PollsDjangoCommonBrowserLocalLogin.png)

10. Click **Create Sample Polls**.

      ![Web Browser](./media/web-sites-python-ptvs-django-mysql/PollsDjangoCommonBrowserNoPolls.png)

11. Click on a poll and vote.

      ![Web Browser](./media/web-sites-python-ptvs-django-mysql/PollsDjangoSqliteBrowser.png)


## Create a MySQL Database
For the database, we'll create a ClearDB MySQL hosted database on Azure.

As an alternative, you can create your own Virtual Machine running in Azure, then install and administer MySQL yourself.

You can create a database with a free plan by following these steps.

1. Log into the [Azure Portal](https://portal.azure.com/).

2. At the Top of the navigation pane, click **NEW**. Then, click **Data + Storage** > **MySQL Database**. 


1. Type "**mysql**" in the search box, then click **MySQL Database**, and then click **Create**.  -->

     <!-- ![Choose Add-on Dialog](./media/web-sites-python-ptvs-django-mysql/PollsDjangoClearDBAddon1.png) -->
2. Configure the new MySQL database by creating a new resource group and select the appropriate location for it.

     <!-- ![Personalize Add-on Dialog](./media/web-sites-python-ptvs-django-mysql/PollsDjangoClearDBAddon2.png) -->
3. Once the MySQL database is created, click **Properties** in the database blade.

4. Use the copy button to put the value of **CONNECTION STRING** on the clipboard.

## Configure the Project
In this section, we'll configure our web app to use the MySQL database we just created. We'll also install additional Python packages required to use MySQL databases with Django. Then we'll run the web app locally.

1. In Visual Studio, open **settings.py**, from the *ProjectName* folder. Temporarily paste the connection string in the editor. The connection string is in this format:

       Database=<NAME>;Data Source=<HOST>;User Id=<USER>;Password=<PASSWORD>

   Change the default database **ENGINE** to use MySQL, and set the values for **NAME**, **USER**, **PASSWORD** and **HOST** from the **CONNECTIONSTRING**.

       DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': '<Database>',
            'USER': '<User Id>',
            'PASSWORD': '<Password>',
            'HOST': '<Data Source>',
            'PORT': '',
        }
    }



1. In Solution Explorer, under **Python Environments**, right-click on the virtual environment and select **Install Python Package**.

2. Install the package `mysql-python` using **easy_install**.

      ![Install Package Dialog](./media/web-sites-python-ptvs-django-mysql/PollsDjangoMySQLInstallPackage.png)

3. Right-click the project node and select **Python**, **Django Sync DB**. 

   This will create the tables for the MySQL database we created in the previous section. Follow the prompts to create a user, which doesn't have to match the user in the sqlite database created in the first section.

     ![Django Management Console Window](./media/web-sites-python-ptvs-django-mysql/PollsDjangoConsole.png)

4. Run the application with `F5`. Polls that are created with **Create Sample Polls** and the data submitted by voting will be serialized in the MySQL database.


## Publish the web app to Azure App Service
The Azure .NET SDK provides an easy way to deploy your web app to Azure App Service.

1. In **Solution Explorer**, right-click on the project node and select **Publish**.

     ![Publish Web Dialog](./media/web-sites-python-ptvs-django-mysql/PollsCommonPublishWebSiteDialog.png)

2. Click on **Microsoft Azure Web Apps**.

3. Click on **New** to create a new web app.

4. Fill in the following fields and click **Create**.

   * **Web App name**
* **App Service plan**
* **Resource group**
* **Region**
* Leave **Database server** set to **No database**

  <!-- ![Create Site on Microsoft Azure Dialog](./media/web-sites-python-ptvs-django-mysql/PollsCommonCreateWebSite.png) -->

5. Accept all other defaults and click **Publish**.

6. Your web browser will open automatically to the published web app. You should see the web app working as expected, using the **MySQL** database hosted on Azure.

   Congratulations!

     ![Web Browser](./media/web-sites-python-ptvs-django-mysql/PollsDjangoAzureBrowser.png)


## Next steps
Follow these links to learn more about Python Tools for Visual Studio, Django and MySQL.

* [Python Tools for Visual Studio Documentation](http://aka.ms/ptvsdocs)  * [Web Projects](http://go.microsoft.com/fwlink/?LinkId=624027)
* [Cloud Service Projects](http://go.microsoft.com/fwlink/?LinkId=624028)
* [Remote Debugging on Microsoft Azure](http://go.microsoft.com/fwlink/?LinkId=624026)


* [Django Documentation](https://www.djangoproject.com/)
* [MySQL](http://www.mysql.com/)

For more information, see the [Python Developer Center](/develop/python/).

## What's changed
* For a guide to the change from Websites to App Service see: [Azure App Service and Its Impact on Existing Azure Services](http://go.microsoft.com/fwlink/?LinkId=529714)

<!--Link references-->

[Python Developer Center]: /develop/python/
[Azure Cloud Services]: ../cloud-services-python-ptvs.md

<!--External Link references-->

[Azure Portal]: https://portal.azure.com
[Python Tools for Visual Studio]: http://aka.ms/ptvs
[Python Tools 2.2 for Visual Studio]: http://go.microsoft.com/fwlink/?LinkID=624025
[Python Tools 2.2 for Visual Studio Samples VSIX]: http://go.microsoft.com/fwlink/?LinkID=624025
[Azure SDK Tools for VS 2013]: http://go.microsoft.com/fwlink/?LinkId=323510
[Azure SDK Tools for VS 2015]: http://go.microsoft.com/fwlink/?LinkId=518003
[Python 2.7 32-bit]: http://go.microsoft.com/fwlink/?LinkId=517190 
[Python Tools for Visual Studio Documentation]: http://aka.ms/ptvsdocs
[Remote Debugging on Microsoft Azure]: http://go.microsoft.com/fwlink/?LinkId=624026
[Web Projects]: http://go.microsoft.com/fwlink/?LinkId=624027
[Cloud Service Projects]: http://go.microsoft.com/fwlink/?LinkId=624028
[Django Documentation]: https://www.djangoproject.com/
[MySQL]: http://www.mysql.com/

