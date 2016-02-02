<properties 
    pageTitle="Configuring Python with Azure App Service Web Apps" 
    description="This tutorial describes options for authoring and configuring a basic Web server Gateway Interface (WSGI) compliant Python application on Azure App Service Web Apps." 
    services="app-service" 
    documentationCenter="python" 
    tags="python"
    authors="huguesv" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="app-service" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="python" 
    ms.topic="article" 
    ms.date="12/16/2015" 
    ms.author="huvalo"/>




# Configuring Python with Azure App Service Web Apps
This tutorial describes options for authoring and configuring a basic Web Server Gateway Interface (WSGI) compliant Python application on [Azure App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714).

It describes additional features of Git deployment, such as virtual environment and package installation using requirements.txt.

## Bottle, Django or Flask?
The Azure Marketplace contains templates for the Bottle, Django and Flask frameworks. If you are developing your first web app in Azure App Service, or you are not familiar with Git, we recommend that you follow one of these tutorials, which include step-by-step instructions for building a working application from the gallery using Git deployment from Windows or Mac:

* [Creating web apps with Bottle](web-sites-python-create-deploy-bottle-app.md)
* [Creating web apps with Django](web-sites-python-create-deploy-django-app.md)
* [Creating web apps with Flask](web-sites-python-create-deploy-flask-app.md)

## Web app creation on Azure Portal
This tutorial assumes an existing Azure subscription and access to the Azure Portal.

If you do not have an existing web app, you can create one from the [Azure Portal](https://portal.azure.com).  Click the NEW button in the top left corner, then click **Web + Mobile** > **Web app**.

## Git Publishing
Configure Git publishing for your newly created web app by following the instructions at [Continuous deployment using GIT in Azure App Service](web-sites-publish-source-control.md). This tutorial uses Git to create, manage, and publish our Python web app to Azure App Service.

Once Git publishing is set up, a Git repository will be created and associated with your web app. The repository's URL will be displayed and can henceforth be used to push data from the local development environment to the cloud. To publish applications via Git, make sure a Git client is also installed and use the instructions provided to push your web app content to Azure App Service.

## Application Overview
In the next sections, the following files are created. They should be placed in the root of the Git repository.

    app.py
    requirements.txt
    runtime.txt
    web.config
    ptvs_virtualenv_proxy.py


## WSGI Handler
WSGI is a Python standard described by [PEP 3333](http://www.python.org/dev/peps/pep-3333/) defining an interface between the web server and Python. It provides a standardized interface for writing various web applications and frameworks using Python. Popular Python web frameworks today use WSGI. Azure App Service Web Apps gives you support for any such frameworks; in addition, advanced users can even author their own as long as the custom handler follows the WSGI specification guidelines.

Here's an example of an `app.py` that defines a custom handler:

    def wsgi_app(environ, start_response):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        start_response(status, response_headers)
        response_body = 'Hello World'
        yield response_body.encode()

    if __name__ == '__main__':
        from wsgiref.simple_server import make_server

        httpd = make_server('localhost', 5555, wsgi_app)
        httpd.serve_forever()

You can run this application locally with `python app.py`, then browse to `http://localhost:5555` in your web browser.

## Virtual Environment
Although the example app above doesn't require any external packages, it is likely that your application will require some.

To help manage external package dependencies, Azure Git deployment supports the creation of virtual environments.

When Azure detects a requirements.txt in the root of the repository, it automatically creates a virtual environment named `env`. This only occurs on the first deployment, or during any deployment after the selected Python runtime has changed.

You will probably want to create a virtual environment locally for development, but don't include it in your Git repository.

## Package Management
Packages listed in requirements.txt will be installed automatically in the virtual environment using pip. This happens on every deployment, but pip will skip installation if a package is already installed.

Example `requirements.txt`:

    azure==0.8.4


## Python Version
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


Example `runtime.txt`:

    python-2.7


## Web.config
You'll need to create a web.config file to specify how the server should handle requests.

Note that if you have a web.x.y.config file in your repository, where x.y matches the selected Python runtime, then Azure will automatically copy the appropriate file as web.config.

The following web.config examples rely on a virtual environment proxy script, which is described in the next section.  They work with the WSGI handler used in the example `app.py` above.

Example `web.config` for Python 2.7:

    <?xml version="1.0"?>
    <configuration>
      <appSettings>
        <add key="WSGI_ALT_VIRTUALENV_HANDLER" value="app.wsgi_app" />
        <add key="WSGI_ALT_VIRTUALENV_ACTIVATE_THIS"
             value="D:\home\site\wwwroot\env\Scripts\activate_this.py" />
        <add key="WSGI_HANDLER"
             value="ptvs_virtualenv_proxy.get_virtualenv_handler()" />
        <add key="PYTHONPATH" value="D:\home\site\wwwroot" />
      </appSettings>
      <system.web>
        <compilation debug="true" targetFramework="4.0" />
      </system.web>
      <system.webServer>
        <modules runAllManagedModulesForAllRequests="true" />
        <handlers>
          <remove name="Python27_via_FastCGI" />
          <remove name="Python34_via_FastCGI" />
          <add name="Python FastCGI"
               path="handler.fcgi"
               verb="*"
               modules="FastCgiModule"
               scriptProcessor="D:\Python27\python.exe|D:\Python27\Scripts\wfastcgi.py"
               resourceType="Unspecified"
               requireAccess="Script" />
        </handlers>
        <rewrite>
          <rules>
            <rule name="Static Files" stopProcessing="true">
              <conditions>
                <add input="true" pattern="false" />
              </conditions>
            </rule>
            <rule name="Configure Python" stopProcessing="true">
              <match url="(.*)" ignoreCase="false" />
              <conditions>
                <add input="{REQUEST_URI}" pattern="^/static/.*" ignoreCase="true" negate="true" />
              </conditions>
              <action type="Rewrite"
                      url="handler.fcgi/{R:1}"
                      appendQueryString="true" />
            </rule>
          </rules>
        </rewrite>
      </system.webServer>
    </configuration>


Example `web.config` for Python 3.4:

    <?xml version="1.0"?>
    <configuration>
      <appSettings>
        <add key="WSGI_ALT_VIRTUALENV_HANDLER" value="app.wsgi_app" />
        <add key="WSGI_ALT_VIRTUALENV_ACTIVATE_THIS"
             value="D:\home\site\wwwroot\env\Scripts\python.exe" />
        <add key="WSGI_HANDLER"
             value="ptvs_virtualenv_proxy.get_venv_handler()" />
        <add key="PYTHONPATH" value="D:\home\site\wwwroot" />
      </appSettings>
      <system.web>
        <compilation debug="true" targetFramework="4.0" />
      </system.web>
      <system.webServer>
        <modules runAllManagedModulesForAllRequests="true" />
        <handlers>
          <remove name="Python27_via_FastCGI" />
          <remove name="Python34_via_FastCGI" />
          <add name="Python FastCGI"
               path="handler.fcgi"
               verb="*"
               modules="FastCgiModule"
               scriptProcessor="D:\Python34\python.exe|D:\Python34\Scripts\wfastcgi.py"
               resourceType="Unspecified"
               requireAccess="Script" />
        </handlers>
        <rewrite>
          <rules>
            <rule name="Static Files" stopProcessing="true">
              <conditions>
                <add input="true" pattern="false" />
              </conditions>
            </rule>
            <rule name="Configure Python" stopProcessing="true">
              <match url="(.*)" ignoreCase="false" />
              <conditions>
                <add input="{REQUEST_URI}" pattern="^/static/.*" ignoreCase="true" negate="true" />
              </conditions>
              <action type="Rewrite" url="handler.fcgi/{R:1}" appendQueryString="true" />
            </rule>
          </rules>
        </rewrite>
      </system.webServer>
    </configuration>


Static files will be handled by the web server directly, without going through Python code, for improved performance.

In the above examples, the location of the static files on disk should match the location in the URL. This means that a request for `http://pythonapp.azurewebsites.net/static/site.css` will serve the file on disk at `\static\site.css`.

`WSGI_ALT_VIRTUALENV_HANDLER` is where you specify the WSGI handler. In the above examples, it's `app.wsgi_app` because the handler is a function named `wsgi_app` in `app.py` in the root folder.

`PYTHONPATH` can be customized, but if you install all your dependencies in the virtual environment by specifying them in requirements.txt, you shouldn't need to change it.

## Virtual Environment Proxy
The following script is used to retrieve the WSGI handler, activate the virtual environment and log errors. It is designed to be generic and used without modifications.

Contents of `ptvs_virtualenv_proxy.py`:

     # ############################################################################
     #
     # Copyright (c) Microsoft Corporation. 
     #
     # This source code is subject to terms and conditions of the Apache License, Version 2.0. A 
     # copy of the license can be found in the License.html file at the root of this distribution. If 
     # you cannot locate the Apache License, Version 2.0, please send an email to 
     # vspython@microsoft.com. By using this source code in any fashion, you are agreeing to be bound 
     # by the terms of the Apache License, Version 2.0.
     #
     # You must not remove this notice, or any other, from this software.
     #
     # ###########################################################################

    import datetime
    import os
    import sys
    import traceback

    if sys.version_info[0] == 3:
        def to_str(value):
            return value.decode(sys.getfilesystemencoding())

        def execfile(path, global_dict):
            """Execute a file"""
            with open(path, 'r') as f:
                code = f.read()
            code = code.replace('\r\n', '\n') + '\n'
            exec(code, global_dict)
    else:
        def to_str(value):
            return value.encode(sys.getfilesystemencoding())

    def log(txt):
        """Logs fatal errors to a log file if WSGI_LOG env var is defined"""
        log_file = os.environ.get('WSGI_LOG')
        if log_file:
            f = open(log_file, 'a+')
            try:
                f.write('%s: %s' % (datetime.datetime.now(), txt))
            finally:
                f.close()

    ptvsd_secret = os.getenv('WSGI_PTVSD_SECRET')
    if ptvsd_secret:
        log('Enabling ptvsd ...\n')
        try:
            import ptvsd
            try:
                ptvsd.enable_attach(ptvsd_secret)
                log('ptvsd enabled.\n')
            except: 
                log('ptvsd.enable_attach failed\n')
        except ImportError:
            log('error importing ptvsd.\n');

    def get_wsgi_handler(handler_name):
        if not handler_name:
            raise Exception('WSGI_ALT_VIRTUALENV_HANDLER env var must be set')

        if not isinstance(handler_name, str):
            handler_name = to_str(handler_name)

        module_name, _, callable_name = handler_name.rpartition('.')
        should_call = callable_name.endswith('()')
        callable_name = callable_name[:-2] if should_call else callable_name
        name_list = [(callable_name, should_call)]
        handler = None
        last_tb = ''

        while module_name:
            try:
                handler = __import__(module_name, fromlist=[name_list[0][0]])
                last_tb = ''
                for name, should_call in name_list:
                    handler = getattr(handler, name)
                    if should_call:
                        handler = handler()
                break
            except ImportError:
                module_name, _, callable_name = module_name.rpartition('.')
                should_call = callable_name.endswith('()')
                callable_name = callable_name[:-2] if should_call else callable_name
                name_list.insert(0, (callable_name, should_call))
                handler = None
                last_tb = ': ' + traceback.format_exc()

        if handler is None:
            raise ValueError('"%s" could not be imported%s' % (handler_name, last_tb))

        return handler

    activate_this = os.getenv('WSGI_ALT_VIRTUALENV_ACTIVATE_THIS')
    if not activate_this:
        raise Exception('WSGI_ALT_VIRTUALENV_ACTIVATE_THIS is not set')

    def get_virtualenv_handler():
        log('Activating virtualenv with %s\n' % activate_this)
        execfile(activate_this, dict(__file__=activate_this))

        log('Getting handler %s\n' % os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        handler = get_wsgi_handler(os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        log('Got handler: %r\n' % handler)
        return handler

    def get_venv_handler():
        log('Activating venv with executable at %s\n' % activate_this)
        import site
        sys.executable = activate_this
        old_sys_path, sys.path = sys.path, []

        site.main()

        sys.path.insert(0, '')
        for item in old_sys_path:
            if item not in sys.path:
                sys.path.append(item)

        log('Getting handler %s\n' % os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        handler = get_wsgi_handler(os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        log('Got handler: %r\n' % handler)
        return handler


## Customize Git deployment
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


## Next steps
For more information, see the [Python Developer Center](/develop/python/).

> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](http://go.microsoft.com/fwlink/?LinkId=523751), where you can immediately create a short-lived starter web app in App Service. No credit cards required; no commitments.
> 
> 
## What's changed
* For a guide to the change from Websites to App Service see: [Azure App Service and Its Impact on Existing Azure Services](http://go.microsoft.com/fwlink/?LinkId=529714)

