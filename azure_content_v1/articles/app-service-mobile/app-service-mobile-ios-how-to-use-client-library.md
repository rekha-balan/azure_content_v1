<properties
    pageTitle="How to Use iOS SDK for Azure Mobile Apps"
    description="How to Use iOS SDK for Azure Mobile Apps"
    services="app-service\mobile"
    documentationCenter="ios"
    authors="krisragh"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-ios"
    ms.devlang="objective-c"
    ms.topic="article"
    ms.date="12/30/2015"
    ms.author="krisragh"/>

# How to Use iOS Client Library for Azure Mobile Apps
> [AZURE.SELECTOR]
- [Android](../articles/app-service-mobile-android-how-to-use-client-library.md)
- [iOS](../articles/app-service-mobile-ios-how-to-use-client-library.md)
- [Managed (Windows/Xamarin)](../articles/app-service-mobile-dotnet-how-to-use-client-library.md)


&nbsp; 

>[AZURE.NOTE] This is an **Azure Mobile Apps** topic. For Mobile Services topics, see the [Mobile Services documentation center](/documentation/services/mobile-services/).
>
>App Service Mobile Apps is our newest mobile backend platform and [gives you additional advantages](app-service-mobile-value-prop-migration-from-mobile-services.md) over Mobile Services. [Migrating to App Service](app-service-mobile-migrating-from-mobile-services.md) is  recommended for customers using the .NET backend SDK. However, [Mobile Apps Node SDK](https://github.com/azure/azure-mobile-apps-node) is currently in Preview and is not yet recommended for production use. SDK and API contracts are subject to change within minor version releases.


This guide teaches you to perform common scenarios using the latest [Azure Mobile Apps iOS SDK](https://go.microsoft.com/fwLink/?LinkID=266533clcid=0x409). If you are new to Azure Mobile Apps, first complete [Azure Mobile Apps Quick Start](app-service-mobile-ios-get-started.md) to create a backend, create a table, and download a pre-built iOS Xcode project. In this guide, we focus on the client-side iOS SDK. To learn more about the .NET server-side SDK for the backend, see [Work with .NET Backend](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)

## Reference documentation
The reference documentation for the iOS client SDK is located here: [Azure Mobile Apps iOS Client Reference](http://azure.github.io/azure-mobile-services/iOS/v3/).

## <a name="Setup"></a>Setup and Prerequisites
This guide assumes that you have created a backend with a table. This guide assumes that the table has the same schema as the tables in those tutorials. This guide also assumes that in your code, you reference `MicrosoftAzureMobile.framework` and import `MicrosoftAzureMobile/MicrosoftAzureMobile.h`.

## <a name="create-client"></a>How to: Create Client
To access an Azure Mobile Apps backend in your project, create an `MSClient`. Replace `AppUrl` with the app URL. You may leave `gatewayURLString` and `applicationKey` empty. If you set up a gateway for authentication, populate `gatewayURLString` with the gateway URL.

**Objective-C**:

```
MSClient *client = [MSClient clientWithApplicationURLString:@"AppUrl"];
```

**Swift**:

```
let client = MSClient(applicationURLString: "AppUrl")
```


## <a name="table-reference"></a>How to: Create Table Reference
To access or update data, create a reference to the backend table. Replace `TodoItem` with the name of your table

**Objective-C**:

```
MSTable *table = [client tableWithName:@"TodoItem"];
```

**Swift**:

```
let table = client.tableWithName("TodoItem")
```


## <a name="querying"></a>How to: Query Data
To create a database query, query the `MSTable` object. The following query gets all the items in `TodoItem` and logs the text of each item.

**Objective-C**:

```
[table readWithCompletion:^(MSQueryResult *result, NSError *error) {
        if(error) { // error is nil if no error occured
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) { // items is NSArray of records that match query
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift**:

```
table.readWithCompletion({(result, error) -> Void in
    if error != nil { // error is nil if no error occured
        NSLog("ERROR %@", error!)
    } else {
        for item in (result?.items)! {
            NSLog("Todo Item: %@", item["text"] as! String)
        }
    }
})
```

## <a name="filtering"></a>How to: Filter Returned Data
To filter results, there are many available options.

To filter using a predicate, use an `NSPredicate` and `readWithPredicate`. The following filters returned data to find only incomplete Todo items.

**Objective-C**:

```
// Create a predicate that finds items where complete is false
NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
// Query the TodoItem table 
[table readWithPredicate:predicate completion:^(MSQueryResult *result, NSError *error) {
        if(error) {
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) {
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift**:

```
// Create a predicate that finds items where complete is false
let predicate =  NSPredicate(format:"complete == NO")
// Query the TodoItem table 
table.readWithPredicate(predicate, completion: { (result, error) -> Void in
    if error != nil {
        NSLog("ERROR %@", error!)
    } else {
        for item in (result?.items)! {
            NSLog("Todo Item: %@", item["text"] as! String)
        }
    }
})
```

## <a name="query-object"></a>How to: Use MSQuery
To perform a complex query (including sorting and paging), create an `MSQuery` object, directly or by using a predicate:

**Objective-C**:

```
MSQuery *query = [table query];
MSQuery *query = [table queryWithPredicate: [NSPredicate predicateWithFormat:@"complete == NO"]];
```

**Swift**:

```
let query = table.query()
let query = table.queryWithPredicate(NSPredicate(format:"complete == NO"))
```

`MSQuery` lets you control several query behaviors, including the following. Execute an `MSQuery` query by calling `readWithCompletion` on it, as shown in the next example.

* Specify order of results
* Limit which fields to return
* Limit how many records to return
* Specify total count in response
* Specify custom query string parameters in request
* Apply additional functions

## <a name="sorting"></a>How to: Sort Data with MSQuery
To sort results, let's look at an example. To first ascendingly by field `text` and then descendingly by field `completion`, invoke `MSQuery` like so:

**Objective-C**:

```
[query orderByAscending:@"text"];
[query orderByDescending:@"complete"];
[query readWithCompletion:^(MSQueryResult *result, NSError *error) {
        if(error) {
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) {
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift**:

```        
query.orderByAscending("text")
query.orderByDescending("complete")
query.readWithCompletion { (result, error) -> Void in
    if error != nil {
        NSLog("ERROR %@", error!)
    } else {
        for item in (result?.items)! {
            NSLog("Todo Item: %@", item["text"] as! String)
        }
    }
}
```


## <a name="selecting"></a><a name="parameters"></a>How to: Limit Fields and Expand Query String Parameters with MSQuery
To limit fields to be returned in a query, specify the names of the fields in the **selectFields** property. This returns only the text and completed fields:

**Objective-C**:

```
query.selectFields = @[@"text", @"complete"];
```

**Swift**:

```
query.selectFields = ["text", "complete"]
```

To include additional query string parameters in the server request (for example, because a custom server-side script uses them), populate `query.parameters` like so:

**Objective-C**:

```
query.parameters = @{
    @"myKey1" : @"value1",
    @"myKey2" : @"value2",
};
```

**Swift**:

```
query.parameters = ["myKey1": "value1", "myKey2": "value2"]
```

## <a name="inserting"></a>How to: Insert Data
To insert a new table row, create a new `NSDictionary` and invoke `table insert`. Mobile Services automatically generates new columns based on the `NSDictionary` if [Dynamic Schema](http://go.microsoft.com/fwlink/p/?LinkId=296271) is not disabled.

If `id` is not provided, the backend automatically generates a new unique ID. Provide your own `id` to use email addresses, usernames, or your own custom values as ID. Providing your own ID may ease joins and business-oriented database logic.

The `result` contains the new item that was inserted; depending on your server logic, it may have additional or modified data compared to what was passed to the server.

**Objective-C**:

```
NSDictionary *newItem = @{@"id": @"custom-id", @"text": @"my new item", @"complete" : @NO};
[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift**:

```
let newItem = ["id": "custom-id", "text": "my new item", "complete": false]
table.insert(newItem) { (result, error) -> Void in
    if error != nil {
        NSLog("ERROR %@", error!)
    } else {
        NSLog("Todo Item: %@", result!["text"] as! String)
    }
}
```

## <a name="modifying"></a>How to: Modify Data
To update an existing row, modify an item and call `update`:

**Objective-C**:

```
NSMutableDictionary *newItem = [oldItem mutableCopy]; // oldItem is NSDictionary
[newItem setValue:@"Updated text" forKey:@"text"];
[table update:newItem completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift**:

```
let newItem = oldItem.mutableCopy() as! NSMutableDictionary // oldItem is NSDictionary
newerItem["text"] = "Updated text"
table.update(newerItem  as [NSObject : AnyObject]) { (result, error) -> Void in
    if error != nil {
        NSLog("ERROR %@", error!)
    } else {
        NSLog("Todo Item: %@", result!["text"] as! String)
    }
}
```

Alternatively, supply the row ID and the updated field:

**Objective-C**:

```
[table update:@{@"id":@"custom-id", @"text":"my EDITED item"} completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift**:

```
table.update(["id": "custom-id", "text": "my EDITED item"]) { (result, error) -> Void in
    if error != nil {
        NSLog("ERROR %@", error!)
    } else {
        NSLog("Todo Item: %@", result!["text"] as! String)
    }

}
```

At minimum, the `id` attribute must be set when making updates.

## <a name="deleting"></a>How to: Delete Data
To delete an item, invoke `delete` with the item:

**Objective-C**:

```
[table delete:item completion:^(id itemId, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item ID: %@", itemId);
    }
}];
```

**Swift**:

```
table.delete(item as [NSObject : AnyObject]) { (itemId, error) -> Void in
    if error != nil {
        NSLog("ERROR %@", error!)
    } else {
        NSLog("Todo Item ID: %@", itemId! as! String)
    }
}
```

Alternatively, delete by providing a row ID:

**Objective-C**:

```
[table deleteWithId:@"37BBF396-11F0-4B39-85C8-B319C729AF6D" completion:^(id itemId, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item ID: %@", itemId);
    }
}];   
```

**Swift**:

```
table.deleteWithId("37BBF396-11F0-4B39-85C8-B319C729AF6D") { (itemId, error) -> Void in
        if error != nil {
            NSLog("ERROR %@", error!)
        } else {
            NSLog("Todo Item ID: %@", itemId! as! String)
        }
}
```

At minimum, the `id` attribute must be set when making deletes.

## <a name="templates"></a>How to: Register push templates to send cross-platform notifications
To register templates, simply pass along templates with your **client.push registerDeviceToken** method in your client app.

**Objective-C**:

```
[client.push registerDeviceToken:deviceToken template:iOSTemplate completion:^(NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    }
}];
```

**Swift**:

```
client.push!.registerDeviceToken(deviceToken, template: iOSTemplate, completion: { (error) -> Void in
            if error != nil {
                NSLog("ERROR %@", error!)
            }
        })
```

Your templates will be of type NSDictionary and can contain multiple templates in the following format:

**Objective-C**:

```
NSDictionary *iOSTemplate = @{ @"templateName": @{ @"body": @{ @"aps": @{ @"alert": @"$(message)" } } } };
```

**Swift**:

```
let iOSTemplate: [NSObject : AnyObject] = ["templateName": ["body": ["aps": ["alert": "$(message)"]]]]
```

Note that all tags will be stripped away for security. To add tags to installations or templates within installations, see [Work with the .NET backend server SDK for Azure Mobile Apps](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#tags).

To send notifications utilizing these registered templates, work with [Notification Hubs APIs](https://msdn.microsoft.com/library/azure/dn495101.aspx)

## <a name="errors"></a>How to: Handle Errors
When you call a mobile service, the completion block contains an `NSError` parameter. When an error occurs, this parameter is non-nil. In your code, you should check this parameter and handle the error as needed, as demonstrated in the code snippets above.

The file [`<WindowsAzureMobileServices/MSError.h>`](https://github.com/Azure/azure-mobile-services/blob/master/sdk/iOS/src/MSError.h) defines the constants `MSErrorResponseKey`, `MSErrorRequestKey`, and `MSErrorServerItemKey` to get more data related to the error, obtainable as follows:

**Objective-C**:

```
NSDictionary *serverItem = [error.userInfo objectForKey:MSErrorServerItemKey];
```

**Swift**:

```
let serverItem = error?.userInfo[MSErrorServerItemKey]
```

In addition, the file defines constants for each error code, which may be used as shown below:

**Objective-C**:

```
if (error.code == MSErrorPreconditionFailed) {
```

**Swift**:

```
if (error?.code == MSErrorPreconditionFailed) {
```

## <a name="adal"></a>How to: Authenticate users with the Active Directory Authentication Library
You can use the Active Directory Authentication Library (ADAL) to sign users into your application using Azure Active Directory. This is often preferable to using the `loginAsync()` methods, as it provides a more native UX feel and allows for additional customization.

1. Configure your mobile app backend for AAD sign-in by followin the [How to configure App Service for Active Directory login](app-service-mobile-how-to-configure-active-directory-authentication.md) tutorial. Make sure to complete the optional step of registering a native client application. For iOS, it is recommended (but not required) that the redirect URI is of the form `<app-scheme>://<bundle-id>`. Please see the [ADAL iOS quickstart](active-directory-devquickstarts-ios.md#em1-determine-what-your-redirect-uri-will-be-for-iosem) for more details.

2. Install ADAL using Cocoapods. Edit your podfile to include the following, replacing **YOUR-PROJECT** with the name of your Xcode project:

    source 'https://github.com/CocoaPods/Specs.git'
 link_with ['YOUR-PROJECT']'YOUR-PROJECT']
 xcodeproj 'YOUR-PROJECT'

    pod 'ADALiOS'

3. Using the Terminal, run `pod install` from the directory containing your project, and then open the generated Xcode workspace (not the project).

4. Add the below code to your application, according to the language you are using. In each, make the following replacements: 

5. Replace **INSERT-AUTHORITY-HERE** ith the name of the tenant in which you provisioned your application. The format should be https://login.windows.net/contoso.onmicrosoft.com. This value can be copied out of the Domain tab in your Azure Active Directory in the [Azure classic portal]Azure classic portal].

6. Replace **INSERT-RESOURCE-ID-HERE** with the client ID for your mobile app backend. You can obtain this from the **Advanced** tab under **Azure Active Directory Settings** in the portal.

7. Replace **INSERT-CLIENT-ID-HERE** with the client ID you copied from the native client application.

8. Replace **INSERT-REDIRECT-URI-HERE** with your site's */.auth/login/done* endpoint, using the HTTPS scheme. This value should be similar to *https://contoso.azurewebsites.net/.auth/login/done*.


**Objective-C**:

    #import <ADALiOS/ADAuthenticationContext.h>
    #import <ADALiOS/ADAuthenticationSettings.h>
    // ...
    - (void) authenticate:(UIViewController*) parent
               completion:(void (^) (MSUser*, NSError*))completionBlock;
    {
        NSString *authority = @"INSERT-AUTHORITY-HERE";
        NSString *resourceId = @"INSERT-RESOURCE-ID-HERE";
        NSString *clientId = @"INSERT-CLIENT-ID-HERE";
        NSURL *redirectUri = [[NSURL alloc]initWithString:@"INSERT-REDIRECT-URI-HERE"];
        ADAuthenticationError *error;
        ADAuthenticationContext authContext = [ADAuthenticationContext authenticationContextWithAuthority:authority error:&error];
        authContext.parentController = parent;
        [ADAuthenticationSettings sharedInstance].enableFullScreen = YES;
        [authContext acquireTokenWithResource:resourceId
                                     clientId:clientId
                                  redirectUri:redirectUri
                              completionBlock:^(ADAuthenticationResult *result) {
                                  if (result.status != AD_SUCCEEDED)
                                  {
                                      completionBlock(nil, result.error);;
                                  }
                                  else
                                  {
                                      NSDictionary *payload = @{
                                                                @"access_token" : result.tokenCacheStoreItem.accessToken
                                                                };
                                      [client loginWithProvider:@"aad" token:payload completion:completionBlock];
                                  }
                              }];
    }


**Swift**:

    // add the following imports to your bridging header:
    //     #import <ADALiOS/ADAuthenticationContext.h>
    //     #import <ADALiOS/ADAuthenticationSettings.h>

    func authenticate(parent:UIViewController, completion: (MSUser?, NSError?) -> Void) {
        let authority = "INSERT-AUTHORITY-HERE"
        let resourceId = "INSERT-RESOURCE-ID-HERE"
        let clientId = "INSERT-CLIENT-ID-HERE"
        let redirectUri = NSURL(string: "INSERT-REDIRECT-URI-HERE")
        var error: AutoreleasingUnsafeMutablePointer<ADAuthenticationError?> = nil
        let authContext = ADAuthenticationContext(authority: authority, error: error)
        authContext.parentController = parent
        ADAuthenticationSettings.sharedInstance().enableFullScreen = true;
        authContext.acquireTokenWithResource(resourceId, clientId: clientId, redirectUri: redirectUri, completionBlock: { (result) -> Void in
            if result.status != AD_SUCCEEDED {
                completion(nil, result.error)
            }
            else {
                let payload:[String:String] = ["access_token":result.tokenCacheStoreItem.accessToken]
                client.loginWithProvider("aad", token: payload, completion: completion)
            }
        })
    }


<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[Setup and Prerequisites]: #Setup
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #table-reference
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Bind data to the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching-tokens
[How to: Upload images and large files]: #blobs
[How to: Handle errors]: #errors
[How to: Design unit tests]: #unit-testing
[How to: Customize the client]: #customizing
[Customize request headers]: #custom-headers
[Customize data type serialization]: #custom-serialization
[Next Steps]: #next-steps
[How to: Use MSQuery]: #query-object

<!-- Images. -->

<!-- URLs. -->

[Azure Mobile Apps Quick Start]: app-service-mobile-ios-get-started.md

[Add Mobile Services to Existing App]: /develop/mobile/tutorials/get-started-data
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-ios
[Validate and modify data in Mobile Services by using server scripts]: /develop/mobile/tutorials/validate-modify-and-augment-data-ios
[Mobile Services SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Authentication]: /develop/mobile/tutorials/get-started-with-users-ios
[iOS SDK]: https://developer.apple.com/xcode

[Handling Expired Tokens]: http://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: http://go.microsoft.com/fwlink/p/?LinkId=301960
[Permissions]: http://msdn.microsoft.com/library/windowsazure/jj193161.aspx
[Service-side Authorization]: mobile-services-javascript-backend-service-side-authorization.md
[Use scripts to authorize users]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[Dynamic Schema]: http://go.microsoft.com/fwlink/p/?LinkId=296271
[How to: access custom parameters]: /develop/mobile/how-to-guides/work-with-server-scripts#access-headers
[Create a table]: http://msdn.microsoft.com/library/windowsazure/jj193162.aspx
[NSDictionary object]: http://go.microsoft.com/fwlink/p/?LinkId=301965
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[CLI to manage Mobile Services tables]: ../virtual-machines-command-line-tools.md#Mobile_Tables
[Conflict-Handler]: mobile-services-ios-handling-conflicts-offline-data.md#add-conflict-handling