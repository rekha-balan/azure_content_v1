<properties 
    pageTitle="Diagnosing issues with dependencies in Application Insights" 
    description="Find failures and slow performance caused by dependencies" 
    services="application-insights" 
    documentationCenter=""
    authors="alancameronwills" 
    manager="douge"/>

<tags 
    ms.service="application-insights" 
    ms.workload="tbd" 
    ms.tgt_pltfrm="ibiza" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="09/17/2015" 
    ms.author="awills"/>

# Diagnosing issues with dependencies in Application Insights
A *dependency* is an external component that is called by your app. It's typically a service called using HTTP, or a database, or a file system. In Visual Studio Application Insights, you can easily see how long your application waits for dependencies and how often a dependency call fails.

## Where you can use it
Out of the box dependency monitoring is currently available for:

* ASP.NET web apps and services running on an IIS server or on Azure
* [Java web apps](app-insights-java-agent.md)

For other types, such as device apps, you can write your own monitor using the [TrackDependency API](app-insights-api-custom-events-metrics.md#track-dependency).

The out-of-the-box dependency monitor currently reports calls to these  types of dependencies:

* ASP.NET  * SQL databases
* ASP.NET web and WCF services that use HTTP-based bindings
* Local or remote HTTP calls
* Azure DocumentDb, table, blob storage, and queue


* Java  * Calls to a database through a [JDBC](http://docs.oracle.com/javase/7/docs/technotes/guides/jdbc/) driver, such as MySQL, SQL Server, PostgreSQL or SQLite.


* Web pages  * AJAX calls



Again, you could write your own SDK calls to monitor other dependencies.

## To set up dependency monitoring
Install the appropriate agent for the host server.

| Platform | Install |
| --- | --- |
| IIS Server |[Status Monitor](app-insights-monitor-performance-live-website-now.md) |
| Azure Web App |[Application Insights Extension](../azure-portal/insights-perf-analytics.md) |
| Java web server |[Java web apps](app-insights-java-agent.md) |

The Status Monitor for IIS Servers doesn't need you to rebuild your source project with the Application Insights SDK. 

## <a name="diagnosis"></a> Diagnosing dependency performance issues
To assess the performance of requests at your server:

![In the Overview page of your application in Application Insights, click the Performance tile](./media/app-insights-dependencies/01-performance.png)

Scroll down to look at the grid of requests:

![List of requests with averages and counts](./media/app-insights-dependencies/02-reqs.png)

The top one is taking very long. Let's see if we can find out where the time is spent.

Click that row to see individual request events:

![List of request occurrences](./media/app-insights-dependencies/03-instances.png)

Click any long-running instance to inspect it further.

> [!NOTE]
> Scroll down a bit to choose an instance. Latency in the pipeline might mean that the data for the top instances is incomplete.
> 
> 
Scroll down to the remote dependency calls related to this request:

![Find Calls to Remote Dependencies, identify unusual Duration](./media/app-insights-dependencies/04-dependencies.png)

It looks like most of the time servicing this request was spent in a call to a local service. 

Select that row to get more information:

![Click through that remote dependency to identify the culprit](./media/app-insights-dependencies/05-detail.png)

The detail includes sufficient information to diagnose the problem.

## Failures
If there are failed requests, click the chart.

![Click the failed requests chart](./media/app-insights-dependencies/06-fail.png)

Click through a request type and request instance, to find a failed call to a remote dependency.

![Click a request type, click the instance to get to a different view of the same instance, click it to get exception details.](./media/app-insights-dependencies/07-faildetail.png)

## Custom dependency tracking
The standard dependency-tracking module automatically discovers external dependencies such as databases and REST APIs. But you might want some additional components to be treated in the same way. 

You can write code that sends dependency information, using the same [TrackDependency API](app-insights-api-custom-events-metrics.md#track-dependency) that is used by the standard modules.

For example, if you build your code with an assembly that you didn't write yourself, you could time all the calls to it, to find out what contribution it makes to your response times. To have this data displayed in the dependency charts in Application Insights, send it using `TrackDependency`.

```C#

            var success = false;
            var startTime = DateTime.UtcNow;
            var timer = System.Diagnostics.Stopwatch.StartNew();
            try
            {
                success = dependency.Call();
            }
            finally
            {
                timer.Stop();
                telemetry.TrackDependency("myDependency", "myCall", startTime, timer.Elapsed, success);
            }
```

If you want to switch off the standard dependency tracking module, remove the reference to DependencyTrackingTelemetryModule in [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md).

<!--Link references-->


