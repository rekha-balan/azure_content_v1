<properties 
 pageTitle="Scheduler Outbound Authentication" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>

<tags 
 ms.service="scheduler" 
 ms.workload="infrastructure-services" 
 ms.tgt_pltfrm="na" 
 ms.devlang="dotnet" 
 ms.topic="article" 
 ms.date="12/04/2015" 
 ms.author="krisragh"/>

# Scheduler Outbound Authentication
Scheduler jobs may need to call out to services that require authentication. This way, a called service can determine if the Scheduler job can access its resources. Some of these services include other Azure services, Salesforce.com, Facebook, and secure custom websites.

## Adding and Removing Authentication
Adding authentication to a Scheduler job is simple – add a JSON child element `authentication` to the `request` element when creating or updating a job. Secrets passed to the Scheduler service in a PUT, PATCH, or POST request – as part of the `authentication` object – are never returned in responses. In responses, secret information is set to null or may have a public token that represents the authenticated entity.

To remove authentication, PUT or PATCH the job explicitly, setting the `authentication` object to null. You will not see any authentication properties back in response.

Currently, the only supported authentication types are the `ClientCertificate` model (for using the SSL/TLS client certificates), the `Basic` model (for Basic authentication), and the `ActiveDirectoryOAuth` model (for Active Directory OAuth authentication.)

## Request Body for ClientCertificate Authentication
When adding authentication using the `ClientCertificate` model, specify the following additional elements in the request body.  

| Element | Description |
|:--- |:--- |
| *authentication (parent element)* |Authentication object for using an SSL client certificate. |
| *type* |Required. Type of authentication.For SSL client certificates, the value must be `ClientCertificate`. |
| *pfx* |Required. Base64-encoded contents of the PFX file. |
| *password* |Required. Password to access the PFX file. |

## Response Body for ClientCertificate Authentication
When a request is sent with authentication info, the response contains the following authentication-related elements.

| Element | Description |
|:--- |:--- |
| *authentication (parent element)* |Authentication object for using an SSL client certificate. |
| *type* |Type of authentication. For SSL client certificates, the value is `ClientCertificate`. |
| *certificateThumbprint* |The thumbprint of the certificate. |
| *certificateSubjectName* |The subject distinguished name of the certificate. |
| *certificateExpiration* |The expiration date of the certificate. |

## Sample Request and Response for ClientCertificate Authentication
The following example request makes a PUT request that incorporates `ClientCertificate` authentication. The request is as follows:

    PUT https://management.core.windows.net/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/cs-brazilsouth-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/testScheduler 
    x-ms-version: 2013-03-01
    User-Agent: Microsoft.WindowsAzure.Scheduler.SchedulerClient/3.0.0.0 AzurePowershell/v0.8.10
    Content-Type: application/json; charset=utf-8
    Host: management.core.windows.net
    Content-Length: 4013
    Expect: 100-continue

    {
      "action": {
        "type": "http",
        "request": {
          "uri": "https://management.core.windows.net/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
          "method": "GET",
          "headers": {
            "x-ms-version": "2013-03-01"
          },
          "authentication": {
            "type": "clientcertificate",
            "password": "test",
            "pfx": "long-pfx-key”
          }
        }
      },
      "recurrence": {
        "frequency": "minute",
        "interval": 1
      }
    }

Once this request is sent, the response is as follows:

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Pragma: no-cache
    Content-Length: 721
    Content-Type: application/json; charset=utf-8
    Expires: -1
    Server: 1.0.6198.153 (rd_rdfe_stable.141027-2149) Microsoft-HTTPAPI/2.0
    x-ms-servedbyregion: ussouth2
    X-AspNet-Version: 4.0.30319
    X-Powered-By: ASP.NET


    {
      "id": "testScheduler",
      "action": {
        "request": {
          "uri": "https:\/\/management.core.windows.net\/7e2dffb5-45b5-475a-91be-d3d9973c82d5\/cloudservices\/CS-NorthCentralUS-scheduler\/resources\/scheduler\/~\/JobCollections\/testScheduler\/jobs\/test",
          "method": "GET",
          "headers": {
            "x-ms-version": "2013-03-01"
          },
          "authentication": {
            "type": "ClientCertificate",
            "certificateThumbprint": "C1645E2AF6317D9FCF9C78FE23F9DE0DAFAD2AB5",
            "certificateExpiration": "2021-01-01T08:00:00Z",
            "certificateSubjectName": "CN=Scheduler Management"
          }
        },
        "type": "http"
      },
      "recurrence": {
        "frequency": "minute",
        "interval": 1
      },
      "state": "enabled",
      "status": {
        "nextExecutionTime": "2014-10-29T21:52:35.2108904Z",
        "executionCount": 0,
        "failureCount": 0,
        "faultedCount": 0
      }
    }
## Request Body for Basic Authentication
When adding authentication using the `Basic` model, specify the following additional elements in the request body.

| Element | Description |
|:--- |:--- |
| *authentication (parent element)* |Authentication object for using Basic authentication. |
| *type* |Required. Type of authentication. For Basic authentication, the value must be `Basic`. |
| *username* |Required. Username to authenticate. |
| *password* |Required. Password to authenticate. |

## Response Body for Basic Authentication
When a request is sent with authentication info, the response contains the following authentication-related elements.

| Element | Description |
|:--- |:--- |
| *authentication (parent element)* |Authentication object for using Basic authentication. |
| *type* |Type of authentication. For Basic authentication, the value is `Basic`. |
| *username* |The authenticated username. |

## Sample Request and Response for Basic Authentication
The following example request makes a PUT request that incorporates `Basic` authentication. The request is as follows:

    PUT https://management.core.windows.net/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/cs-brazilsouth-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/testScheduler 
    x-ms-version: 2013-03-01
    User-Agent: Microsoft.WindowsAzure.Scheduler.SchedulerClient/3.0.0.0 AzurePowershell/v0.8.10
    Content-Type: application/json; charset=utf-8
    Host: management.core.windows.net
    Expect: 100-continue

    {
      "action": {
        "type": "http",
        "request": {
          "uri": "https://management.core.windows.net/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
          "method": "GET",
          "headers": {
            "x-ms-version": "2013-03-01"
          },
        "authentication":{  
          "username":"user1",
          "password":"password",
          "type":"basic"
          }           
        }
      },
      "recurrence": {
        "frequency": "minute",
        "interval": 1
      }
    }

Once this request is sent, the response is as follows:

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Pragma: no-cache
    Content-Length: 721
    Content-Type: application/json; charset=utf-8
    Expires: -1
    Server: 1.0.6198.153 (rd_rdfe_stable.141027-2149) Microsoft-HTTPAPI/2.0
    x-ms-servedbyregion: ussouth2
    X-AspNet-Version: 4.0.30319
    X-Powered-By: ASP.NET

    {
      "id": "testScheduler",
      "action": {
        "request": {
          "uri": "https:\/\/management.core.windows.net\/7e2dffb5-45b5-475a-91be-d3d9973c82d5\/cloudservices\/CS-NorthCentralUS-scheduler\/resources\/scheduler\/~\/JobCollections\/testScheduler\/jobs\/test",
          "method": "GET",
          "headers": {
            "x-ms-version": "2013-03-01"
          },
          "authentication":{  
            "username":"user1",
            "type":"Basic"
          }
        },
        "type": "http"
      },
      "recurrence": {
        "frequency": "minute",
        "interval": 1
      },
      "state": "enabled",
      "status": {
        "nextExecutionTime": "2014-10-29T21:52:35.2108904Z",
        "executionCount": 0,
        "failureCount": 0,
        "faultedCount": 0
      }
    }

## Request Body for ActiveDirectoryOAuth Authentication
When adding authentication using the `ActiveDirectoryOAuth` model, specify the following additional elements in the request body.

| Element | Description |
|:--- |:--- |
| *authentication (parent element)* |Authentication object for using ActiveDirectoryOAuth authentication. |
| *type* |Required. Type of authentication. For ActiveDirectoryOAuth authentication, the value must be `ActiveDirectoryOAuth`. |
| *tenant* |Required. The tenant identifier for the Azure AD tenant. |
| *audience* |Required. This is set to https://management.core.windows.net/. |
| *clientId* |Required. Provide the client identifier for the Azure AD application. |
| *secret* |Required. Secret of the client that is requesting the token. |

### Determining your Tenant Identifier
You can find the tenant identifier for the Azure AD tenant by running `Get-AzureAccount` in Azure PowerShell.

## Response Body for ActiveDirectoryOAuth Authentication
When a request is sent with authentication info, the response contains the following authentication-related elements.

| Element | Description |
|:--- |:--- |
| *authentication (parent element)* |Authentication object for using ActiveDirectoryOAuth authentication. |
| *type* |Type of authentication. For ActiveDirectoryOAuth authentication, the value is `ActiveDirectoryOAuth`. |
| *tenant* |The tenant identifier for the Azure AD tenant. |
| *audience* |This is set to https://management.core.windows.net/. |
| *clientId* |The client identifier for the Azure AD application. |

## Sample Request and Response for ActiveDirectoryOAuth Authentication
The following example request makes a PUT request that incorporates `ActiveDirectoryOAuth` authentication. The request is as follows:

    PUT https://management.core.windows.net/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/cs-brazilsouth-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/testScheduler 
    x-ms-version: 2013-03-01
    User-Agent: Microsoft.WindowsAzure.Scheduler.SchedulerClient/3.0.0.0 AzurePowershell/v0.8.10
    Content-Type: application/json; charset=utf-8
    Host: management.core.windows.net
    Expect: 100-continue

    {
      "action": {
        "type": "http",
        "request": {
          "uri": "https://management.core.windows.net/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
          "method": "GET",
          "headers": {
            "x-ms-version": "2013-03-01"
          },
          "authentication":{  
            "tenant":"01234567-89ab-cdef-0123-456789abcdef",
            "audience":"https://management.core.windows.net/",
            "clientId":"8a14db88-4d1a-46c7-8429-20323727dfab",
            "secret": "&lt;secret-key&gt;",
            "type":"ActiveDirectoryOAuth"
          }                      
        }
      },
      "recurrence": {
        "frequency": "minute",
        "interval": 1
      }
    }

Once this request is sent, the response is as follows:

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Pragma: no-cache
    Content-Length: 721
    Content-Type: application/json; charset=utf-8
    Expires: -1
    Server: 1.0.6198.153 (rd_rdfe_stable.141027-2149) Microsoft-HTTPAPI/2.0
    x-ms-servedbyregion: ussouth2
    X-AspNet-Version: 4.0.30319
    X-Powered-By: ASP.NET


    {
      "id": "testScheduler",
      "action": {
        "request": {
          "uri": "https:\/\/management.core.windows.net\/7e2dffb5-45b5-475a-91be-d3d9973c82d5\/cloudservices\/CS-NorthCentralUS-scheduler\/resources\/scheduler\/~\/JobCollections\/testScheduler\/jobs\/test",
          "method": "GET",
          "headers": {
            "x-ms-version": "2013-03-01"
          },
          "authentication":{  
            "tenant":"01234567-89ab-cdef-0123-456789abcdef",
            "audience":"https://management.core.windows.net/",
            "clientId":"8a14db88-4d1a-46c7-8429-20323727dfab",
            "type":"ActiveDirectoryOAuth"
          }
        },
        "type": "http"
      },
      "recurrence": {
        "frequency": "minute",
        "interval": 1
      },
      "state": "enabled",
      "status": {
        "nextExecutionTime": "2014-10-29T21:52:35.2108904Z",
        "executionCount": 0,
        "failureCount": 0,
        "faultedCount": 0
      }
    }

## See Also
 [What is Scheduler?](scheduler-intro.md)

 [Azure Scheduler concepts, terminology, and entity hierarchy](scheduler-concepts-terms.md)

 [Get started using Scheduler in the Azure portal](scheduler-get-started-portal.md)

 [Plans and billing in Azure Scheduler](scheduler-plans-billing.md)

 [Azure Scheduler REST API reference](https://msdn.microsoft.com/library/mt629143)

 [Azure Scheduler PowerShell cmdlets reference](scheduler-powershell-reference.md)

 [Azure Scheduler high-availability and reliability](scheduler-high-availability-reliability.md)

 [Azure Scheduler limits, defaults, and error codes](scheduler-limits-defaults-errors.md)
