<properties
    pageTitle="Role Based Access Control in Mobile Services using .NET and the Azure Active Directory (Windows Store) | Microsoft Azure"
    description="Learn how to control access based on Azure Active Directory roles in your Windows Store application using a Mobile Service with a .NET backend."
    documentationCenter="windows"
    authors="wesmc7777"
    manager="dwrede"
    editor=""
    services="mobile-services"/>

<tags
    ms.service="mobile-services"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="12/07/2015"
    ms.author="wesmc"/>

# Role Based Access Control in Mobile Services using JavaScript and the Azure Active Directory
>[AZURE.NOTE] This is an **Azure Mobile Services** topic.  Microsoft Azure recommends Azure App Service Mobile Apps for all new mobile backend deployments.
To get started with Azure App Service Mobile Apps, see the [App Service Mobile Apps documentation center](/documentation/services/app-service/mobile).


&nbsp;

> [AZURE.SELECTOR-LIST (Platform | Backend)]
- [(Windows 8.x Store C# | .NET)](../articles/mobile-services-dotnet-backend-windows-store-dotnet-aad-rbac.md)

## Overview
Roles-based access control (RBAC) is the practice of assigning permissions to roles that your users can hold. It nicely defines boundaries on what certain classes of users can and cannot do. This tutorial will walk you through how to add basic RBAC to Azure Mobile Services.

This tutorial will demonstrate role based access control, checking each user's membership to a Sales group defined in the Azure Active Directory (AAD). The access check will be done with .NET Mobile Service backend using the [Graph REST API](http://msdn.microsoft.com/library/azure/hh974478.aspx) for Azure Active Directory. Only users who belong to the Sales group will be allowed to query the data.

> [!NOTE]
> The intent of this tutorial is to extend your knowledge of authentication to include authorization practices. It is expected that you first complete the [Add Authentication to your app](mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users.md) tutorial using the Azure Active Directory authentication provider. This tutorial continues to update the TodoItem application used in the [Add Authentication to your app](mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users.md) tutorial.
> 
> 
## Prerequisites
This tutorial requires the following:

* Visual Studio 2013 running on Windows 8.1.
* Completion of the [Add Authentication to your app](mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users.md) tutorial using the Azure Active Directory authentication provider.

## Generate a key for the Integrated Application
During the [Add Authentication to your app](mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users.md) tutorial, you created a registration for the integrated application when you completed the [Register to use an Azure Active Directory Login](mobile-services-how-to-register-active-directory-authentication.md) step. In this section you generate a key to be used when reading directory information with that integrated application's client ID.

1. Click **Applications** tab on your directory page in the [Azure classic portal](https://manage.windowsazure.com/).
  
2. Click your integrated application registration.

3. Click **Configure** on the application page and scroll down the the **keys** section of the page. 
4. Click **1 year** duration for a new key. Then click **Save** and the portal will display your new key value.
5. Copy the **Client ID** and **Key** shown after you save. Note that the key value will only be shown to you a single time after you have saved. 

    ![](./media/mobile-services-generate-aad-app-registration-access-key/client-id-and-key.png)

6. Scroll down to the bottom of the integrated application configuration page and enable the **Read directory data** permission for the application and click **Save**.

    ![](./media/mobile-services-generate-aad-app-registration-access-key/app-perms.png)


7. In the [Azure classic portal](https://manage.windowsazure.com/), navigate back to your mobile service and click the **Configure** tab. Scroll down to the **app settings** section and add the following app settings and click **Save**. 

    <table border="1">
    <tr>
    <th>App Setting Name</th><th>Description</th>
    </tr>
    <tr>
    <td>AAD_CLIENT_ID</td><td>The client id you copied from your integrated app in the steps above.</td>
    </tr>
    <tr>
    <td>AAD_CLIENT_KEY</td><td>The app key you generated in your AAD integrated app in the steps above.</td>
    </tr>
    <tr>
    <td>AAD_TENANT_DOMAIN</td><td>Your AAD domain name. Should be similar to "mydomain.onmicrosoft.com"</td>
    </tr>
    </table><br/>

 
    ![](./media/mobile-services-generate-aad-app-registration-access-key/aad-app-settings.png)
  

## Create a Sales group with membership
In this section you add two new users to your directory along with the new Sales group. One of the users will be granted membership to the sales group. The other user will not be granted membership to the group. 

### Create the users


1. In the [Azure classic portal](https://manage.windowsazure.com) navigate to the directory that you previously configured for authentication when you completed the tutorial to add authentication to your app.
2. Click **Users** at the top of the page. Then click the **Add User** button at the bottom. 
3. Complete the new user dialogs creating to create a user named **Bob**. Note the temporary password for the user. 
4. Create another user named **Dave**. Note the temporary password for the user.
5. The new users should look similar to what is shown below.

    ![](./media/mobile-services-aad-rbac-create-sales-group/users.png)    


### Create the Sales group


1. On the directory page, click **Groups** at the top of the page. Then click the **Add Group** button at the bottom. 
2. Enter **Sales** for the name of the group and press the complete button on the dialog to create the group. 

    ![](./media/mobile-services-aad-rbac-create-sales-group/sales-group.png)

### Add user membership to the Sales group.


1. Click **Groups** at the top of the directory page. Then click the **Sales** group to go to the sales group page. 
2. On the Sales group page, click **Add Members**. Add the user named **Bob** to the sales group. The user named **Dave** should not be a member of the group.

    ![](./media/mobile-services-aad-rbac-create-sales-group/group-membership.png)

3. On the Sales group page, click **Properties**, then copy the **Object ID** for the sales group at the bottom of the page. 

   
    ![](./media/mobile-services-aad-rbac-create-sales-group/sales-group-id.png)

4. Navigate back to your mobile service configuration page and add the object id as an app setting named **AAD\_SALES\_GROUP\_ID**. This tutorial uses group's object id as an app setting instead of looking up the id based on the group name. This is because the group name may change where the id stays the same.

    ![](./media/mobile-services-aad-rbac-create-sales-group/sales-group-id-app-setting.png)

## Create a custom authorization attribute on the mobile service
In this section you will create a new custom authorization attribute that can be used to perform access checks on mobile service operations. The attribute will look up an Active Directory group based on the role name passed to it. It will then perform access checks based on that group's membership.

1. In Visual Studio, right click mobile service .NET backend project and click **Manage NuGet Packages**.

2. In the NuGet Package Manager dialog, enter **ADAL** in the search criteria to find and install the **Active Directory Authentication Library** for your mobile service. This tutorial was most recently tested with the 3.3.205061641-alpha (Prerelease) version of the ADAL package.

3. In Visual Studio, right click your mobile service project and click **Add** then **New Folder**. Name the new folder **Utilities**.

4. In Visual Studio, right click the new **Utilities** folder and add a new class file named **AuthorizeAadRole.cs**.

    ![][0]

5. In the AuthorizeAadRole.cs file, add the following `using` statements at the top of the file.

        using System.Net;
     using System.Net.Http;
     using System.Web.Http;
     using System.Web.Http.Controllers;
     using System.Web.Http.Filters;
     using Newtonsoft.Json;
     using Microsoft.WindowsAzure.Mobile.Service.Security;
     using Microsoft.WindowsAzure.Mobile.Service;
     using Microsoft.IdentityModel.Clients.ActiveDirectory;
     using System.Globalization;
     using System.IO;
6. In AuthorizeAadRole.cs, add the following enumerated type to the Utilities namespace. In this example we only deal with the **Sales** role. The others are just examples of groups you might use.

        public enum AadRoles
     {
         Sales,
         Management,
         Development
     }
7. In AuthorizeAadRole.cs, add the following `AuthorizeAadRole` class definition to the Utilities namespace.

        [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = false, Inherited = true)]
     public class AuthorizeAadRole : AuthorizationFilterAttribute
     {
         private bool isInitialized;
         private bool isHosted;
         private ApiServices services = null;

         // Constants used with ADAL and the Graph REST API for AAD
         private const string AadInstance = "https://login.windows.net/{0}";
         private const string GraphResourceId = "https://graph.windows.net/";
         private const string APIVersion = "?api-version=2013-04-05";

         // App settings pulled from the Mobile Service
         private string tenantdomain;
         private string clientid;
         private string clientkey;
         private Dictionary<int, string> groupIds = new Dictionary<int, string>();

         private string token = null;

         public AuthorizeAadRole(AadRoles role)
         {
             this.Role = role;
         }

         // private class used to serialize the Graph REST API web response
         private class MembershipResponse
         {
             public bool value;
         }

         public AadRoles Role { get; private set; }

         // Generate a local dictionary for the role group ids configured as
         // Mobile Service app settings
         private void InitGroupIds()
         {
         }

         // Use ADAL and the authentication app settings from the Mobile Service to
         // get an AAD access token
         private string GetAADToken()
         {
         }

         // Given an AAD user id, check membership against the group associated with the role.
         private bool CheckMembership(string memberId)
         {
         }

         // Called when the user is attempting authorization
         public override void OnAuthorization(HttpActionContext actionContext)
         {
         }
     }




1. In AuthorizeAadRole.cs, update the `InitGroupIds` method on the `AuthorizeAadRole` class as follows. This method creates a dictionary mapping of the group ids to each role.

        private void InitGroupIds()
     {
         string groupId;

         if (services == null)
             return;

         if (!groupIds.ContainsKey((int)AadRoles.Sales))
         {
             if (services.Settings.TryGetValue("AAD_SALES_GROUP_ID", out groupId))
             {
                 groupIds.Add((int)AadRoles.Sales, groupId);
             }
             else
                 services.Log.Error("AAD_SALES_GROUP_ID app setting not found.");
         }
     }



1. In AuthorizeAadRole.cs, update the `GetAADToken` method on the `AuthorizeAadRole` class. This method uses the app settings stored in the Mobile Service to get an access token to the AAD from ADAL.

   > [!NOTE]
> ADAL for .NET includes an in-memory token cache by default to help alleviate extra network traffic against your Active Directory. However, you can write your own cache implementation or disable caching entirely. For more information see [ADAL for .NET].
> 
> 
        // Use ADAL and the authentication app settings from the Mobile Service to get an AAD access token
     private async Task<string> GetAADToken()
     {
         // Try to get the required AAD authentication app settings from the mobile service.
         if (!(services.Settings.TryGetValue("AAD_CLIENT_ID", out clientid) &
               services.Settings.TryGetValue("AAD_CLIENT_KEY", out clientkey) &
               services.Settings.TryGetValue("AAD_TENANT_DOMAIN", out tenantdomain)))
         {
             services.Log.Error("GetAADToken() : Could not retrieve mobile service app settings.");
             return null;
         }

         ClientCredential clientCred = new ClientCredential(clientid, clientkey);
         string authority = String.Format(CultureInfo.InvariantCulture, AadInstance, tenantdomain);
         AuthenticationContext authContext = new AuthenticationContext(authority);

         AuthenticationResult result = await authContext.AcquireTokenAsync(GraphResourceId, clientCred);
         if (result != null)
             token = result.AccessToken;
         else
             services.Log.Error("GetAADToken() : Failed to return a token.");

         return token;
     }
2. In AuthorizeAadRole.cs, update the `CheckMembership` method on the `AuthorizeAadRole` class. This method receives a user's object id. It then uses the AAD Graph REST API to check that object id to see if it is a member id for the group associated to the role selected on the `AuthorizeAadRole` class

       private bool CheckMembership(string memberId)
    {
        bool membership = false;
        string url = GraphResourceId + tenantdomain + "/isMemberOf" + APIVersion;
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);

        // Use the Graph REST API to check group membership in the AAD
        try
        {
            request.Method = "POST";
            request.ContentType = "application/json";
            request.Headers.Add("Authorization", token);
            using (var sw = new StreamWriter(request.GetRequestStream()))
            {
                // Request body must have the group id and a member id to check for membership
                string body = String.Format("\"groupId\":\"{0}\",\"memberId\":\"{1}\"",
                    groupIds[(int)Role], memberId);
                sw.Write("{" + body + "}");
            }

            WebResponse response = request.GetResponse();
            StreamReader sr = new StreamReader(response.GetResponseStream());
            string json = sr.ReadToEnd();
            MembershipResponse membershipResponse = JsonConvert.DeserializeObject<MembershipResponse>(json);
            membership = membershipResponse.value;
        }
        catch (Exception e)
        {
            services.Log.Error("OnAuthorization() exception : " + e.Message);
        }

        return membership;
    }



1. In AuthorizeAadRole.cs, update the `OnAuthorization` method in the `AuthorizeAadRole` class with the following code. This code expects that the user calling into the Mobiile Service has authenticated with the AAD.  It then gets the user's AAD object id and checks membership with the Active Directory group that corresponds to the role.

   > [!NOTE]
> You could look up the Active Directory group by name. However, in many cases it's a better practice to store the group id as a mobile service app setting. This is because the group name is more likely to change but, the id stays the same.
> 
> 
       public override void OnAuthorization(HttpActionContext actionContext)
    {
        if (actionContext == null)
        {
            throw new ArgumentNullException("actionContext");
        }

        services = new ApiServices(actionContext.ControllerContext.Configuration);

        // Check whether we are running in a mode where local host access is allowed
        // through without authentication.
        if (!this.isInitialized)
        {
            HttpConfiguration config = actionContext.ControllerContext.Configuration;
            this.isHosted = config.GetIsHosted();
            this.isInitialized = true;
        }

        // No security when hosted locally
        if (!this.isHosted && actionContext.RequestContext.IsLocal)
        {
            services.Log.Warn("AuthorizeAadRole: Local Hosting.");
            return;
        }

        ApiController controller = actionContext.ControllerContext.Controller as ApiController;
        if (controller == null)
        {
            services.Log.Error("AuthorizeAadRole: No ApiController.");
        }

        bool isAuthorized = false;
        try
        {
            // Initialize a mapping for the group id to our enumerated type
            InitGroupIds();

            // Retrieve a AAD token from ADAL
            GetAADToken();
            if (token == null)
            {
                services.Log.Error("AuthorizeAadRole: Failed to get an AAD access token.");
            }
            else
            {
                // Check group membership to see if the user is part of the group that corresponds to the role
                if (!string.IsNullOrEmpty(groupIds[(int)Role]))
                {
                    ServiceUser serviceUser = controller.User as ServiceUser;
                    if (serviceUser != null && serviceUser.Level == AuthorizationLevel.User)
                    {
                        var idents = serviceUser.GetIdentitiesAsync().Result;
                        AzureActiveDirectoryCredentials clientAadCredentials =
                            idents.OfType<AzureActiveDirectoryCredentials>().FirstOrDefault();
                        if (clientAadCredentials != null)
                        {
                            isAuthorized = CheckMembership(clientAadCredentials.ObjectId);
                        }
                    }
                }
            }
        }
        catch (Exception e)
        {
            services.Log.Error(e.Message);
        }
        finally
        {
            if (isAuthorized == false)
            {
                services.Log.Error("Denying access");

                actionContext.Response = actionContext.Request
                    .CreateErrorResponse(HttpStatusCode.Forbidden,
                        "User is not logged in or not a member of the required group");
            }
        }
    }
2. Save your changes to AuthorizeAadRole.cs.


## Add role based access checking to the database operations
1. In Visual Studio, expand the **Controllers** folder under the mobile service project. Open TodoItemController.cs.

2. In TodoItemController.cs, add a `using` statement for your utilities namespace that contains the custom authorization attribute.

        using todolistService.Utilities;
3. In TodoItemController.cs, you can add the attribute to your controller class or individual methods depending on how you want access checked. If you want all controller operations to check access based on the same role, just add the attribute to the class. Add the attribute to the class as follows for testing this tutorial.

        [AuthorizeAadRole(AadGroups.Sales)]
     public class TodoItemController : TableController<TodoItem>

    If you only wanted to access check insert, update, and delete operations, you would set the attribute only on those methods as follows.

        // PATCH tables/TodoItem
     [AuthorizeAadRole(AadGroups.Sales)]
     public Task<TodoItem> PatchTodoItem(string id, Delta<TodoItem> patch)
     {
         return UpdateAsync(id, patch);
     }

     // POST tables/TodoItem
     [AuthorizeAadRole(AadGroups.Sales)]
     public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
     {
         TodoItem current = await InsertAsync(item);
         return CreatedAtRoute("Tables", new { id = current.Id }, current);
     }

     // DELETE tables/TodoItem
     [AuthorizeAadRole(AadGroups.Sales)]
     public Task DeleteTodoItem(string id)
     {
         return DeleteAsync(id);
     }



1. Save TodoItemController.cs and build the mobile service to verify no syntax errors.
2. Publish the mobile service to your Azure account.

## Test the client's access

The instructions and screenshots below apply to testing a Windows Store client but, you can test this on any of the other platforms supported by Azure Mobile Services. 

1. In Visual Studio,run the client app and attempt to authenticate with the user account we created named Dave. 

    ![](./media/mobile-services-aad-rbac-test-app/dave-login.png)

2. Dave doesn't have membership to the Sales group. So the role based access check will denied access to the table operations. Close the client app.

    ![](./media/mobile-services-aad-rbac-test-app/unauthorized.png)

3. In Visual Studio, run the client app again. This time authenticate with the user account we created named Bob.

    ![](./media/mobile-services-aad-rbac-test-app/bob-login.png)

4. Bob does have membership to the Sales group. So the role based access check will allow access to the table operations.

    ![](./media/mobile-services-aad-rbac-test-app/success.png)





<!-- Images -->

[0]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-aad-rbac/add-authorize-aad-role-class.png

<!-- URLs. -->

[Add Authentication to your app]: mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users.md
[How to Register with the Azure Active Directory]: mobile-services-how-to-register-active-directory-authentication.md
[Directory Sync Scenarios]: http://msdn.microsoft.com/library/azure/jj573653.aspx
[Store Server Scripts]: mobile-services-store-scripts-source-control.md
[Register to use an Azure Active Directory Login]: mobile-services-how-to-register-active-directory-authentication.md
[Graph REST API]: http://msdn.microsoft.com/library/azure/hh974478.aspx
[IsMemberOf]: http://msdn.microsoft.com/library/azure/dn151601.aspx
[ADAL for .NET]: https://msdn.microsoft.com/library/azure/jj573266.aspx