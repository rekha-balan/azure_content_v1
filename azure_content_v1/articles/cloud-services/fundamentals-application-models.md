<properties 
    pageTitle="Compute Hosting Options Provided by Azure" 
    description="Learn about Azure compute hosting options and how they work: Virtual Machines, Websites, Cloud Services, and others." 
    headerExpose="" 
    footerExpose="" 
    services="cloud-services,virtual-machines"
    authors="Thraka" 
    documentationCenter=""
    manager="timlt"/>

<tags 
    ms.service="multiple" 
    ms.workload="multiple" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="09/08/2015" 
    ms.author="adegeo;cephalin;kathydav"/>




# Compute Hosting Options Provided by Azure
Azure provides different hosting models for running applications. Each one provides a different set of services, and so which one you choose depends on exactly what you're trying to do. This article looks at the options, describing each technology and giving examples of when you'd use it.

| Compute Options | Audience |
| --- | --- |
| [App Service](#tellmeas.md) |Scalable Web Apps, Mobile Apps, API Apps, and Logic Apps for any device |
| [Cloud Services](#tellmecs.md) |Highly available, scalable n-tier cloud apps with more control of the OS |
| [Virtual Machines](#tellmevm.md) |Customized Windows and Linux VMs with complete control of the OS |

<a name="tellmeas"></a>
## Tell me about app service

Azure Virtual Machines can handle a wide range of cloud hosting tasks. But creating and managing a VM infrastructure requires specialized skills and substantial effort. If you don't need complete control over the VMs that run your web apps, mobile app backends, API apps, etc., there's an easier (and cheaper) solution: *Platform as a Service* (PaaS). With PaaS, Azure handles most of the management work for the VMs that run your applications. [Azure App Service](../article/app-service/app-service-value-prop-what-is.md) is a fully managed PaaS offering that allows you to build, deploy, and scale enterprise-grade apps in seconds.

App Service is the best choice for many kinds of application workloads. A corporation might want to build or migrate a commercial website that can handle millions of hits a week and is deployed in several data centers across the globe. The same corporation might also have a line-of-business app that tracks expense reports for authenticated users from the corporate Active Directory, and the app might have a mobile device component and connect to on-premise resources and business processes. The expense reports might require periodic background jobs to calculate and summarize large volumes of information. An IT consultant might adapt a popular open source application to set up a content management system for a small business. The figure below shows some of the kinds of web apps that can run in Azure App Service.

<a name="appservice_diagram"></a>
![app service diagram](media/app-service-choose-me-content/diagram.png)
 
**Figure: Azure App Service supports static web pages, popular web applications, and custom web applications built with various technologies. You can also run mobile backends, API app, and non-web compute workloads (using WebJobs).** 

With Azure App Service, you can also run any kind of compute workload using the [WebJobs](../article/app-service-web/websites-webjobs-resources.md) feature. 

Azure App Service gives you the option of running on shared VMs that contain multiple apps created by multiple users, or on VMs that are used only by you. VMs are a part of a pool of resources managed by Azure App Service and thus allow for high reliability and fault tolerance.

Getting started is easy. With Azure App Service, users can select from a range of applications, frameworks and template and create a web app in seconds. They can then use their favorite development tools (WebMatrix, Visual Studio, any other editor) and source control options to set up continuous integration and develop as a team. Applications that rely on a MySQL DB can consume a MySQL service provided for Azure by ClearDB, a Microsoft partner.

Developers can create large, scalable web applications with Azure App Service. The technology supports creating applications using ASP.NET, PHP, Node.js and Python. Applications can use sticky sessions, for example, and many existing web apps can be moved to this cloud platform with no changes. Web apps built on Azure App Service can use other aspects of Azure, such as Service Bus, SQL Database, and Blob Storage. You can also run multiple copies of an application in different VMs, with Azure App Service automatically load balancing requests across them. And because new web app instances are created in VMs that already exist, starting a new application instance happens very quickly; it's significantly faster than waiting for a new VM to be created.

As the [figure](#appservice_diagram) above shows, you can publish code and other web content into Azure App Service in several ways. You can use FTP, FTPS, or Microsoft's WebDeploy technology. Azure App Service also supports publishing code from source control systems, including Git, GitHub, CodePlex, BitBucket, Dropbox, Mercurial, Team Foundation Server, and the cloud-based Team Foundation Service.


<a name="tellmecs"></a>
## Tell me about cloud services

Cloud Services is an example of Platform-as-a-Service (PaaS). Like [App Services](app-service-web-overview.md), this technology is designed to support applications that are scalable, reliable, and cheap to operate. Just like an [App Services](app-service-web-overview.md) are hosted on VMs, so too are Cloud Services, however, you have more control over the VMs. You can install your own software on Cloud Service VMs and you can remote into them.

![cs_diagram](./media/cloud-services-choose-me-content/diagram.png) 

More control also means less ease of use; unless you need the  additional control options, it's typically quicker and easier to get a web application up and running in Web Apps in App Service compared to Cloud Services. 

The technology provides two slightly different VM options: instances of *web roles* run a variant of Windows Server with IIS, while instances of *worker roles* run the same Windows Server variant without IIS. A Cloud Services application relies on some combination of these two options. 

Any combination of these two slightly different VM hosting options are available in a cloud service:

* **Web role**  
  Runs Windows Server with your web app automatically deployed to IIS.
* **Worker role**  
  Runs Windows Server without IIS.

For example, a simple application might use just a web role, while a more complex application might use a web role to handle incoming requests from users, then pass the work those requests create to a worker role for processing. (This communication could use [Service Bus](../articles/service-bus/fundamentals-service-bus-hybrid-solutions.md) or [Azure Queues](../articles/storage/storage-introduction.md).)

As the figure above suggests, all of the VMs in a single application run in the same cloud service. Because of this, users access the application through a single public IP address, with requests automatically load balanced across the application's VMs. The platform will [scale and deploy](../articles/cloud-services/cloud-services-how-to-scale.md) the VMs in a Cloud Services application in a way that avoids a single point of hardware failure. 

Even though applications run in virtual machines, it's important to understand that Cloud Services provides PaaS, not IaaS. Here's one way to think about it: With IaaS, such as Azure Virtual Machines, you first create and configure the environment your application will run in, then deploy your application into this environment. You're responsible for managing much of this world, doing things such as deploying new patched versions of the operating system in each VM. In PaaS, by contrast, it's as if the environment already exists. All you have to do is deploy your application. Management of the platform it runs on, including deploying new versions of the operating system, is handled for you.

## Scaling and management
With Cloud Services, you don't create virtual machines. Instead, you provide a configuration file that tells Azure how many of each you'd like, such as **three web role instances** and **two worker role instances**, and the platform creates them for you.  You still choose [what size](../articles/cloud-services/cloud-services-sizes-specs.md) those backing VMs should be, but you don't explicitly create them yourself. If your application needs to handle a greater load, you can ask for more VMs, and Azure will create those instances. If the load decreases, you can shut those instances down and stop paying for them.

A Cloud Services application is typically made available to users via a two-step process. A developer first [uploads the application](../articles/cloud-services/cloud-services-how-to-create-deploy.md) to the platform's staging area. When the developer is ready to make the application live, they use the Azure Management Portal to request that it be put into production. This [switch between staging and production](../articles/cloud-services/cloud-services-nodejs-stage-application.md) can be done with no downtime, which lets a running application be upgraded to a new version without disturbing its users. 

## Monitoring
Cloud Services also provides monitoring. Like Azure Virtual Machines, it will detect a failed physical server and restart the VMs that were running on that server on a new machine. But Cloud Services also detects failed VMs and applications, not just hardware failures. Unlike Virtual Machines, it has an agent inside each web and worker role, and so it's able to start new VMs and application instances when failures occur.

The PaaS nature of Cloud Services has other implications, too. One of the most important is that applications built on this technology should be written to run correctly when any web or worker role instance fails. To achieve this, a Cloud Services application shouldn't maintain state in the file system of its own VMs. Unlike VMs created with Azure Virtual Machines, writes made to Cloud Services VMs aren't persistent; there's nothing like a Virtual Machines data disk. Instead, a Cloud Services application should explicitly write all state to SQL Database, blobs, tables, or some other external storage. Building applications this way makes them easier to scale and more resistant to failure, both important goals of Cloud Services.


| Compute Options    | Audience   |
| ------------------ | --------   |
| [App Service]      | Scalable Web Apps, Mobile Apps, API Apps, and Logic Apps for any device |
| [Cloud Services]   | Highly available, scalable n-tier cloud apps with more control of the OS |
| [Virtual Machines] | Customized Windows and Linux VMs with complete control of the OS |

<a name="tellmevm"></a>
## Tell me about virtual machines

Azure Virtual Machines lets you create and use virtual machines in the cloud. Providing what's known as *Infrastructure as a Service (IaaS)*, virtual machine technology can be used in variety of ways. Some examples are:

- **Virtual machines (VMs) for development and test.** Development groups commonly use VMs because they offer a quick, easy way to create a computer with specific configurations required to code and test an application. Azure Virtual Machines provides a straightforward and economical way to create these VMs, use them, then delete them when they're no longer needed.
- **Running applications in the cloud.** It makes economic sense to run some applications in the public cloud. One example is an application that has large spikes in demand. Although you could equip your own data center with enough hardware to handle peak demand, that hardware might be underutilized much of the time. Running this application on Azure lets you pay for extra VMs only when you need them and shut them down when you don't. Or, suppose you're a start-up that needs on-demand computing resources quickly and with no commitment. Once again, Azure can be the right choice.
- **Extending your own datacenter into the public cloud.** When you use Azure Virtual Network, your organization can create a virtual network (VNET) that's an extension of your own on-premises network and add VMs to that VNET. This allows running applications such as [SharePoint](virtual-machines-sharepoint-infrastructure-services.md), [SQL Server](virtual-machines-sql-server-infrastructure-services.md) and others on an Azure VM. This approach might be easier to deploy or less expensive than running them in VMs your own datacenter.   
- **Disaster recovery.** Rather than paying continuously for a backup datacenter that's rarely used, IaaS-based disaster recovery lets you pay for the computing resources you need only when you really need them.  For example, if your primary datacenter goes down, you can create VMs running on Azure to run essential applications, then shut them down when they're no longer needed.

Like other virtual machines, a VM in Azure has an operating system, storage and networking capabilities and can run a wide variety of applications. You can use an image provided by Azure or one of it's partners, or use your own. Examples include various versions, editions and configurations of:
 
-	Windows Server 
-	Linux servers such as Suse, Ubuntu and CentOS
-	SQL Server
-	BizTalk Server 
-	SharePoint Server

Virtual machines use virtual hard disks (VHDs) to store their operating system (OS) and data. VHDs are also used for the images you can choose from to install an OS. The following figure shows this, as well as two of the tools for creating and managing your VMs.

<a name="fig_createvms"></a>
![vm_diagram](./media/virtual-machines-choose-me-content/diagram.png)

**Figure: Azure Virtual Machines provides Infrastructure as a Service.**

VMs can be managed using a browser-based portal, command-line tools with support for scripting, or directly through the REST API. Microsoft partners such as RightScale and ScaleXtreme also provide management services that rely on the REST API. 

Along with the OS, other configuration choices you have with VMs include:

- The size, which determines factors such as how many disks you can attach and the processing power. Azure offers a wide variety of sizes to support many types of uses. For details, see [Sizes for Virtual Machines](virtual-machines-size-specs.md).  
- The Azure region where your new VM will be hosted, such as in the US, Europe, or Asia. 
- VM extensions, which give your virtual machine additional capabilities, such as running anti-virus or using the Desired State Configuration feature of Windows PowerShell.

Other benefits to consider for VMs include:

**Pay-as-you-go** -- Azure charges an hourly price based on the VM’s size and operating system. For partial hours, Azure charges only for the minutes of use. Storage is priced and charged separately. For details, see [Virtual Machines Pricing](https://azure.microsoft.com/pricing/details/virtual-machines/).

**Resiliency** -- Azure monitors the physical hardware that hosts each running VM. If a physical server running a VM fails, Azure notices this, moves the VM to new hardware and restarts the VM. This process is sometimes called service healing. Azure also protects a virtual machine's data, by keeping redundant copies of the VHDs in blob storage. 





## Other Options
Azure also offers other compute hosting models for more specialized purposes, such as the following:

* [Mobile Services](/services/mobile-services/)  
Optimized to provide a cloud back-end for apps that run on mobile devices.
* [Batch](/services/batch/)  
Optimized for processing large volumes of similar tasks, ideally workloads which lend themselves to running as parallel tasks on multiple computers.
* [HDInsight (Hadoop)](/services/hdinsight/)  
Optimized for running [MapReduce](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options/#hadoop) jobs on Hadoop clusters. 

## What should I use? Making a choice
All three of the general purpose Azure compute hosting models let you build scalable, reliable applications in the cloud. Given this essential similarity, which one should you use?

App Service is the best choice for most web apps. Deployment and management are integrated into the platform, sites can scale quickly to handle high traffic loads, and the built-in load balancing and traffic manager provide high availability. You can move existing sites to App Service easily with an [online migration tool](https://www.migratetoazure.net/), use an open-source app from the Web Application Gallery, or create a new site using the framework and tools of your choice. The [WebJobs](http://go.microsoft.com/fwlink/?linkid=390226) feature makes it easy to add background job processing to your app, or even run a compute workload that isn't a web app at all. 

If you need more control over the web server environment, such as the ability to remote into your server or configure server startup tasks, Azure Cloud Services is typically the best option.

If you have an existing application that would require substantial modifications to run in Azure Websites or Azure Cloud Services, you could choose Azure Virtual Machines in order to simplify migrating to the cloud. However, correctly configuring, securing, and maintaining VMs requires much more time and IT expertise compared to Azure Websites and Cloud Services. If you are considering Azure Virtual Machines, make sure you take into account the ongoing maintenance effort required to patch, update, and manage your VM environment.

Sometimes, no single option is right. In situations like this, it's perfectly legal to combine options. For example, suppose you're building an application where you'd like the management benefits of Cloud Services web roles, but you also need to use standard SQL Server hosted in a Virtual Machine for compatibility or performance reasons. 

<!-- In this case, the best option is to combine compute hosting options, as the figure below shows.--

<a name="fig4"></a>
![07_CombineTechnologies][07_CombineTechnologies] 

**Figure: A single application can use multiple hosting options.**

As the figure illustrates, the Cloud Services VMs run in a separate cloud service from the Virtual Machines VMs. Still, the two can communicate quite efficiently, so building an app this way is sometimes the best choice.
[07_CombineTechnologies]: ./media/fundamentals-application-models/ExecModels_07_CombineTechnologies.png
!-->

[App Service]: #tellmeas
[Virtual Machines]: #tellmevm
[Cloud Services]: #tellmecs

## Next steps
* [Compare](../choose-web-site-cloud-service-vm/.md) App Service, Cloud Services, and Virtual Machines
* Learn more about [App Service](../app-service-web-overview.md)
* Learn more about [Cloud Service](services/cloud-services/.md)
* Learn more about [Virtual Machines](https://msdn.microsoft.com/library/azure/jj156143.aspx) 

