<properties
    pageTitle="Creating web apps with Django in Azure"
    description="A tutorial that introduces you to running a Python web app in Azure App Service Web Apps."
    services="app-service\web"
    documentationCenter="python"
    tags="python"
    authors="huguesv" 
    manager="wpickett" 
    editor=""/>

<tags
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="hero-article" 
    ms.date="11/16/2015"
    ms.author="huvalo"/>


# Creating web apps with Django in Azure
This tutorial describes how to get started running Python on [Azure App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714). Web Apps provides limited free hosting and rapid deployment, and you can use Python! As your app grows, you can switch to paid hosting, and you can also integrate with all of the other Azure services.

You will create an application using the Django web framework (see alternate versions of this tutorial for [Flask](web-sites-python-create-deploy-flask-app.md) and [Bottle](web-sites-python-create-deploy-bottle-app.md)). You will create the web app from the Azure Marketplace, set up Git deployment, and clone the repository locally. Then you will run the application locally, make changes, commit and push them to Azure. The tutorial shows how to do this from Windows or Mac/Linux.


> [AZURE.NOTE]
> To complete this tutorial, you need an Azure account. You can <a href="/pricing/member-offers/msdn-benefits-details/" target="_blank">activate your Visual Studio subscriber benefits</a> or <a href="/pricing/free-trial/" target="_blank">sign up for a free trial</a>.


> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](http://go.microsoft.com/fwlink/?LinkId=523751), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.
> 
> 
## Prerequisites
* Windows, Mac or Linux
* Python 2.7 or 3.4
* setuptools, pip, virtualenv (Python 2.7 only)
* Git
* [Python Tools for Visual Studio](http://aka.ms/ptvs) (PTVS) - Note: this is optional

**Note**: TFS publishing is currently not supported for Python projects.

### Windows
If you don't already have Python 2.7 or 3.4 installed (32-bit), we recommend installing [Azure SDK for Python 2.7](http://go.microsoft.com/fwlink/?linkid=254281) or [Azure SDK for Python 3.4](http://go.microsoft.com/fwlink/?linkid=516990) using Web Platform Installer. This installs the 32-bit version of Python, setuptools, pip, virtualenv, etc (32-bit Python is what's installed on the Azure host machines). Alternatively, you can get Python from [python.org](http://www.python.org/).

For Git, we recommend [Git for Windows](http://msysgit.github.io/) or [GitHub for Windows](https://windows.github.com/). If you use Visual Studio, you can use the integrated Git support.

We also recommend installing [Python Tools 2.2 for Visual Studio](http://go.microsoft.com/fwlink/?LinkID=624025). This is optional, but if you have [Visual Studio](http://www.visualstudio.com/), including the free Visual Studio Community 2013 or Visual Studio Express 2013 for Web, then this will give you a great Python IDE.

### Mac/Linux
You should have Python and Git already installed, but make sure you have either Python 2.7 or 3.4.

## Web App Creation on Portal
The first step in creating your app is to create the web app via the [Azure Portal](https://portal.azure.com).

1. Log into the Azure Portal and click the **NEW** button in the bottom left corner.
2. Click **Web + Mobile**.
3. In the search box, type "python".
4. In the search results, select **Django**, then click **Create**.
5. Configure the new Django app, such as creating a new App Service plan and a new resource group for it. Then, click **Create**.
6. Configure Git publishing for your newly created web app by following the instructions at [Continuous deployment using GIT in Azure App Service](web-sites-publish-source-control.md).

## Application Overview
### Git repository contents
Here's an overview of the files you'll find in the initial Git repository, which we'll clone in the next section.

    \app\__init__.py
    \app\forms.py
    \app\models.py
    \app\tests.py
    \app\views.py
    \app\static\content\
    \app\static\fonts\
    \app\static\scripts\
    \app\templates\about.html
    \app\templates\contact.html
    \app\templates\index.html
    \app\templates\layout.html
    \app\templates\login.html
    \app\templates\loginpartial.html
    \DjangoWebProject\__init__.py
    \DjangoWebProject\settings.py
    \DjangoWebProject\urls.py
    \DjangoWebProject\wsgi.py

Main sources for the application. Consists of 3 pages (index, about, contact) with a master layout. Static content and scripts include bootstrap, jquery, modernizr and respond.

    \manage.py

Local management and development server support. Use this to run the application locally, synchronize the database, etc.

    \db.sqlite3

Default database. Includes the necessary tables for the application to run, but does not contain any users (synchronize the database to create a user).

    \DjangoWebProject.pyproj
    \DjangoWebProject.sln

Project files for use with [Python Tools for Visual Studio](http://aka.ms/ptvs).

    \ptvs_virtualenv_proxy.py

IIS proxy for virtual environments and PTVS remote debugging support.

    \requirements.txt

External packages needed by this application. The deployment script will pip install the packages listed in this file.

    \web.2.7.config
    \web.3.4.config

IIS configuration files. The deployment script will use the appropriate web.x.y.config and copy it as web.config.

### Optional files - Customizing deployment
Azure will determine that your application uses Python **if both of these conditions are true**:

- requirements.txt file in the root folder
- any .py file in the root folder OR a runtime.txt that specifies python

When that's the case, it will use a Python specific deployment script, which performs the standard synchronization of files, as well as additional Python operations such as:

- Automatic management of virtual environment
- Installation of packages listed in requirements.txt using pip
- Creation of the appropriate web.config based on the selected Python version.
- Collect static files for Django applications

You can control certain aspects of the default deployment steps without having to customize the script.

If you want to skip all Python specific deployment steps, you can create this empty file:

    \.skipPythonDeployment

If you want to skip collection of static files for your Django application:

    \.skipDjango 

For more control over deployment, you can override the default deployment script by creating the following files:

    \.deployment
    \deploy.cmd

You can use the [Azure command-line interface][] to create the files.  Use this command from your project folder:

    azure site deploymentscript --python

When these files don't exist, Azure creates a temporary deployment script and runs it.  It is identical to the one you create with the command above.

[Azure command-line interface]: http://azure.microsoft.com/downloads/


### Optional files - Python runtime
Azure will determine the version of Python to use for its virtual environment with the following priority:

1. version specified in runtime.txt in the root folder
1. version specified by Python setting in the web app configuration (the **Settings** > **Application Settings** blade for your web app in the Azure Portal)
1. python-2.7 is the default if none of the above are specified

Valid values for the contents of 

    \runtime.txt

are:

- python-2.7
- python-3.4

If the micro version (third digit) is specified, it is ignored.


### Additional files on server
Some files exist on the server but are not added to the git repository. These are created by the deployment script.

    \web.config

IIS configuration file. Created from web.x.y.config on every deployment.

    \env\

Python virtual environment. Created during deployment if a compatible virtual environment doesn't already exist on the web app. Packages listed in requirements.txt are pip installed, but pip will skip installation if the packages are already installed.

The next 3 sections describe how to proceed with the web app development under 3 different environments:

* Windows, with Python Tools for Visual Studio
* Windows, with command line
* Mac/Linux, with command line

## Web app development - Windows - Python Tools for Visual Studio
### Clone the repository
First, clone the repository using the URL provided on the Azure Portal. For more information, see [Continuous deployment using GIT in Azure App Service](web-sites-publish-source-control.md).

Open the solution file (.sln) that is included in the root of the repository.

![](./media/web-sites-python-create-deploy-django-app/ptvs-solution-django.png)

### Create virtual environment
Now we'll create a virtual environment for local development. Right-click on **Python Environments** select **Add Virtual Environment...**.

* Make sure the name of the environment is `env`.

* Select the base interpreter. Make sure to use the same version of Python that is selected for your web app (in runtime.txt or the **Application Settings** blade of your web app in the Azure Portal).

* Make sure the option to download and install packages is checked.


![](./media/web-sites-python-create-deploy-django-app/ptvs-add-virtual-env-27.png)

Click **Create**. This will create the virtual environment, and install dependencies listed in requirements.txt.

### Create a superuser
The database included with the application does not have any superuser defined. In order to use the sign-in functionality in the application, or the Django admin interface (if you decide to enable it), you'll need to create a superuser.

Run this from the command-line from your project folder:

    env\scripts\python manage.py createsuperuser

Follow the prompts to set the user name, password, etc.

### Run using development server
Press F5 to start debugging, and your web browser will open automatically to the page running locally.

![](./media/web-sites-python-create-deploy-django-app/windows-browser-django.png)

You can set breakpoints in the sources, use the watch windows, etc. See the [Python Tools for Visual Studio Documentation](http://aka.ms/ptvsdocs) for more information on the various features.

### Make changes
Now you can experiment by making changes to the application sources and/or templates.

After you've tested your changes, commit them to the Git repository:

![](./media/web-sites-python-create-deploy-django-app/ptvs-commit-django.png)

### Install more packages
Your application may have dependencies beyond Python and Django.

You can install additional packages using pip. To install a package, right-click on the virtual environment and select **Install Python Package**.

For example, to install the Azure SDK for Python, which gives you access to Azure storage, service bus and other Azure services, enter `azure`:

![](./media/web-sites-python-create-deploy-django-app/ptvs-install-package-dialog.png)

Right-click on the virtual environment and select **Generate requirements.txt** to update requirements.txt.

Then, commit the changes to requirements.txt to the Git repository.

### Deploy to Azure
To trigger a deployment, click on **Sync** or **Push**. Sync does both a push and a pull.

![](./media/web-sites-python-create-deploy-django-app/ptvs-git-push.png)

The first deployment will take some time, as it will create a virtual environment, install packages, etc.

Visual Studio doesn't show the progress of the deployment. If you'd like to review the output, see the section on [Troubleshooting - Deployment](#troubleshooting-deployment.md).

Browse to the Azure URL to view your changes.

## Web app development - Windows - command line
### Clone the repository
First, clone the repository using the URL provided on the Azure Portal, and add the Azure repository as a remote. For more information, see [Continuous deployment using GIT in Azure App Service](web-sites-publish-source-control.md).

    git clone <repo-url>
    cd <repo-folder>
    git remote add azure <repo-url>

### Create virtual environment
We'll create a new virtual environment for development purposes (do not add it to the repository). Virtual environments in Python are not relocatable, so every developer working on the application will create their own locally.

Make sure to use the same version of Python that is selected for your web app (in runtime.txt or the Application Settings blade of your web app in the Azure Portal).

For Python 2.7:

    c:\python27\python.exe -m virtualenv env

For Python 3.4:

    c:\python34\python.exe -m venv env

Install any external packages required by your application. You can use the requirements.txt file at the root of the repository to install the packages in your virtual environment:

    env\scripts\pip install -r requirements.txt

### Create a superuser
The database included with the application does not have any superuser defined. In order to use the sign-in functionality in the application, or the Django admin interface (if you decide to enable it), you'll need to create a superuser.

Run this from the command-line from your project folder:

    env\scripts\python manage.py createsuperuser

Follow the prompts to set the user name, password, etc.

### Run using development server
You can launch the application under a development server with the following command:

    env\scripts\python manage.py runserver

The console will display the URL and port the server listens to:

![](./media/web-sites-python-create-deploy-django-app/windows-run-local-django.png)

Then, open your web browser to that URL.

![](./media/web-sites-python-create-deploy-django-app/windows-browser-django.png)

### Make changes
Now you can experiment by making changes to the application sources and/or templates.

After you've tested your changes, commit them to the Git repository:

    git add <modified-file>
    git commit -m "<commit-comment>"

### Install more packages
Your application may have dependencies beyond Python and Django.

You can install additional packages using pip. For example, to install the Azure SDK for Python, which gives you access to Azure storage, service bus and other Azure services, type:

    env\scripts\pip install azure

Make sure to update requirements.txt:

    env\scripts\pip freeze > requirements.txt

Commit the changes:

    git add requirements.txt
    git commit -m "Added azure package"

### Deploy to Azure
To trigger a deployment, push the changes to Azure:

    git push azure master

You will see the output of the deployment script, including virtual environment creation, installation of packages, creation of web.config.

Browse to the Azure URL to view your changes.

## Web app development - Mac/Linux - command line
### Clone the repository
First, clone the repository using the URL provided on the Azure Portal, and add the Azure repository as a remote. For more information, see [Continuous deployment using GIT in Azure App Service](web-sites-publish-source-control.md).

    git clone <repo-url>
    cd <repo-folder>
    git remote add azure <repo-url>

### Create virtual environment
We'll create a new virtual environment for development purposes (do not add it to the repository). Virtual environments in Python are not relocatable, so every developer working on the application will create their own locally.

Make sure to use the same version of Python that is selected for your web app (in runtime.txt or the Application Settings blade of your web app in the Azure Portal).

For Python 2.7:

    python -m virtualenv env

For Python 3.4:

    python -m venv env

or

    pyvenv env

Install any external packages required by your application. You can use the requirements.txt file at the root of the repository to install the packages in your virtual environment:

    env/bin/pip install -r requirements.txt

### Create a superuser
The database included with the application does not have any superuser defined. In order to use the sign-in functionality in the application, or the Django admin interface (if you decide to enable it), you'll need to create a superuser.

Run this from the command-line from your project folder:

    env/bin/python manage.py createsuperuser

Follow the prompts to set the user name, password, etc.

### Run using development server
You can launch the application under a development server with the following command:

    env/bin/python manage.py runserver

The console will display the URL and port the server listens to:

![](./media/web-sites-python-create-deploy-django-app/mac-run-local-django.png)

Then, open your web browser to that URL.

![](./media/web-sites-python-create-deploy-django-app/mac-browser-django.png)

### Make changes
Now you can experiment by making changes to the application sources and/or templates.

After you've tested your changes, commit them to the Git repository:

    git add <modified-file>
    git commit -m "<commit-comment>"

### Install more packages
Your application may have dependencies beyond Python and Django.

You can install additional packages using pip. For example, to install the Azure SDK for Python, which gives you access to Azure storage, service bus and other Azure services, type:

    env/bin/pip install azure

Make sure to update requirements.txt:

    env/bin/pip freeze > requirements.txt

Commit the changes:

    git add requirements.txt
    git commit -m "Added azure package"

### Deploy to Azure
To trigger a deployment, push the changes to Azure:

    git push azure master

You will see the output of the deployment script, including virtual environment creation, installation of packages, creation of web.config.

Browse to the Azure URL to view your changes.

## Troubleshooting - Package Installation
Some packages may not install using pip when run on Azure.  It may simply be that the package is not available on the Python Package Index.  It could be that a compiler is required (a compiler is not available on the machine running the web app in Azure App Service).

In this section, we'll look at ways to deal with this issue.

### Request wheels

If the package installation requires a compiler, you should try contacting the package owner to request that wheels be made available for the package.

With the recent availability of [Microsoft Visual C++ Compiler for Python 2.7][], it is now easier to build packages that have native code for Python 2.7.

### Build wheels (requires Windows)

Note: When using this option, make sure to compile the package using a Python environment that matches the platform/architecture/version that is used on the web app in Azure App Service (Windows/32-bit/2.7 or 3.4).

If the package doesn't install because it requires a compiler, you can install the compiler on your local machine and build a wheel for the package, which you will then include in your repository.

Mac/Linux Users: If you don't have access to a Windows machine, see [Create a Virtual Machine Running Windows][] for how to create a VM on Azure.  You can use it to build the wheels, add them to the repository, and discard the VM if you like. 

For Python 2.7, you can install [Microsoft Visual C++ Compiler for Python 2.7][].

For Python 3.4, you can install [Microsoft Visual C++ 2010 Express][].

To build wheels, you'll need the wheel package:

    env\scripts\pip install wheel

You'll use `pip wheel` to compile a dependency:

    env\scripts\pip wheel azure==0.8.4

This creates a .whl file in the \wheelhouse folder.  Add the \wheelhouse folder and wheel files to your repository.

Edit your requirements.txt to add the `--find-links` option at the top. This tells pip to look for an exact match in the local folder before going to the python package index.

    --find-links wheelhouse
    azure==0.8.4

If you want to include all your dependencies in the \wheelhouse folder and not use the python package index at all, you can force pip to ignore the package index by adding `--no-index` to the top of your requirements.txt.

    --no-index

### Customize installation

You can customize the deployment script to install a package in the virtual environment using an alternate installer, such as easy\_install.  See deploy.cmd for an example that is commented out.  Make sure that such packages aren't listed in requirements.txt, to prevent pip from installing them.

Add this to the deployment script:

    env\scripts\easy_install somepackage

You may also be able to use easy\_install to install from an exe installer (some are zip compatible, so easy\_install supports them).  Add the installer to your repository, and invoke easy\_install by passing the path to the executable.

Add this to the deployment script:

    env\scripts\easy_install "%DEPLOYMENT_SOURCE%\installers\somepackage.exe"

### Include the virtual environment in the repository (requires Windows)

Note: When using this option, make sure to use a virtual environment that matches the platform/architecture/version that is used on the web app in Azure App Service (Windows/32-bit/2.7 or 3.4).

If you include the virtual environment in the repository, you can prevent the deployment script from doing virtual environment management on Azure by creating an empty file:

    .skipPythonDeployment

We recommend that you delete the existing virtual environment on the app, to prevent leftover files from when the virtual environment was managed automatically.


[Create a Virtual Machine Running Windows]: http://azure.microsoft.com/documentation/articles/virtual-machines-windows-tutorial/
[Microsoft Visual C++ Compiler for Python 2.7]: http://aka.ms/vcpython27
[Microsoft Visual C++ 2010 Express]: http://go.microsoft.com/?linkid=9709949


## Troubleshooting - Virtual Environment
The deployment script will skip creation of the virtual environment on Azure if it detects that a compatible virtual environment already exists.  This can speed up deployment considerably.  Packages that are already installed will be skipped by pip.

In certain situations, you may want to force delete that virtual environment.  You'll want to do this if you decide to include a virtual environment as part of your repository.  You may also want to do this if you need to get rid of certain packages, or test changes to requirements.txt.

There are a few options to manage the existing virtual environment on Azure:

### Option 1: Use FTP

With an FTP client, connect to the server and you'll be able to delete the env folder.  Note that some FTP clients (such as web browsers) may be read-only and won't allow you to delete folders, so you'll want to make sure to use an FTP client with that capability.  The FTP host name and user are displayed in your web app's blade on the [Azure Portal](https://portal.azure.com).

### Option 2: Toggle runtime

Here's an alternative that takes advantage of the fact that the deployment script will delete the env folder when it doesn't match the desired version of Python.  This will effectively delete the existing environment, and create a new one.

1. Switch to a different version of Python (via runtime.txt or the **Application Settings** blade in the Azure Portal)
1. git push some changes (ignore any pip install errors if any)
1. Switch back to initial version of Python
1. git push some changes again

### Option 3: Customize deployment script

If you've customized the deployment script, you can change the code in deploy.cmd to force it to delete the env folder.


## Troubleshooting - Static Files
Django has the concept of collecting static files. This takes all the static files from their original location and copies them to a single folder. For this application, they are copied to `/static`.

This is done because static files may come from different Django 'apps'. For example, the static files from the Django admin interfaces are located in a Django library subfolder in the virtual environment. Static files defined by this application are located in `/app/static`. As you use more Django 'apps', you'll have static files located in multiple places.

When running the application in debug mode, the application serves the static files from their original location.

When running the application in release mode, the application does **not** serve the static files. It is the responsibility of the web server to serve the files. For this application, IIS will serve the static files from `/static`.

The collection of static files is done automatically as part of the deployment script, clearing previously collected files. This means the collection occurs on every deployment, slowing down deployment a bit, but it ensures that obsolete files won't be available, avoiding a potential security issue.

If you want to skip collection of static files for your Django application:

    \.skipDjango

Then you'll need to do the collection manually on your local machine:

    env\scripts\python manage.py collectstatic

Then remove the `\static` folder from `.gitignore` and add it to the Git repository.

## Troubleshooting - Settings
Various settings for the application can be changed in `DjangoWebProject/settings.py`.

For developer convenience, debug mode is enabled. One nice side effect of that is you'll be able to see images and other static content when running locally, without having to collect static files.

To disable debug mode:

    DEBUG = False

When debug is disabled, the value for `ALLOWED_HOSTS` needs to be updated to include the Azure host name. For example:

    ALLOWED_HOSTS = (
        'pythonapp.azurewebsites.net',
    )

or to enable any:

    ALLOWED_HOSTS = (
        '*',
    )

In practice, you may want to do something more complex to deal with switching between debug and release mode, and getting the host name.

You can set environment variables through the Azure portal **CONFIGURE** page, in the **app settings** section.  This can be useful for setting values that you may not want to appear in the sources (connection strings, passwords, etc), or that you want to set differently between Azure and your local machine. In `settings.py`, you can query the environment variables using `os.getenv`.

## Using a Database
The database that is included with the application is a sqlite database. This is a convenient and useful default database to use for development, as it requires almost no setup. The database is stored in the db.sqlite3 file in the project folder.

Azure provides database services which are easy to use from a Django application. Tutorials for using [SQL Database](web-sites-python-ptvs-django-sql.md) and [MySQL](web-sites-python-ptvs-django-mysql.md) from a Django application show the steps necessary to create the database service, change the database settings in `DjangoWebProject/settings.py`, and the libraries required to install.

Of course, if you prefer to manage your own database servers, you can do so using Windows or Linux virtual machines running on Azure.

## Django Admin Interface
Once you start building your models, you'll want to populate the database with some data. An easy way to do add and edit content interactively is to use the Django administration interface.

The code for the admin interface is commented out in the application sources, but it's clearly marked so you can easily enable it (search for 'admin').

After it's enabled, synchronize the database, run the application and navigate to `/admin`.

## Next Steps
Follow these links to learn more about Django and Python Tools for Visual Studio:

* [Django Documentation](https://www.djangoproject.com/)
* [Python Tools for Visual Studio Documentation](http://aka.ms/ptvsdocs)

For information on using SQL Database and MySQL:

* [Django and MySQL on Azure with Python Tools for Visual Studio](web-sites-python-ptvs-django-mysql.md)
* [Django and SQL Database on Azure with Python Tools for Visual Studio](web-sites-python-ptvs-django-sql.md)

For more information, see the [Python Developer Center](/develop/python/).

## What's changed
* For a guide to the change from Websites to App Service see: [Azure App Service and Its Impact on Existing Azure Services](http://go.microsoft.com/fwlink/?LinkId=529714)

<!--Link references-->

[Django and MySQL on Azure with Python Tools for Visual Studio]: web-sites-python-ptvs-django-mysql.md
[Django and SQL Database on Azure with Python Tools for Visual Studio]: web-sites-python-ptvs-django-sql.md
[SQL Database]: web-sites-python-ptvs-django-sql.md
[MySQL]: web-sites-python-ptvs-django-mysql.md

<!--External Link references-->

[Azure SDK for Python 2.7]: http://go.microsoft.com/fwlink/?linkid=254281
[Azure SDK for Python 3.4]: http://go.microsoft.com/fwlink/?linkid=516990
[python.org]: http://www.python.org/
[Git for Windows]: http://msysgit.github.io/
[GitHub for Windows]: https://windows.github.com/
[Python Tools for Visual Studio]: http://aka.ms/ptvs
[Python Tools 2.2 for Visual Studio]: http://go.microsoft.com/fwlink/?LinkID=624025
[Visual Studio]: http://www.visualstudio.com/
[Python Tools for Visual Studio Documentation]: http://aka.ms/ptvsdocs
[Django Documentation]: https://www.djangoproject.com/
