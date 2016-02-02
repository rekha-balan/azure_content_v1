<properties
    pageTitle="Add authentication on Android with Mobile Apps| Azure App Service"
    description="Learn how to use Mobile Apps in Azure App Service to authenticate users of your Android app through a variety of identity providers, including Google, Facebook, Twitter, and Microsoft."
    services="app-service\mobile"
    documentationCenter="android"
    authors="ggailey777"
    manager="dwrede"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="12/06/2015"
    ms.author="glenga"/>

# Add authentication to your Android app
> [AZURE.SELECTOR]
- [Android](../articles/app-service-mobile-android-get-started-users.md)
- [iOS](../articles/app-service-mobile-ios-get-started-users.md)
- [Windows](../articles/app-service-mobile-windows-store-dotnet-get-started-users.md)
- [Xamarin.Android](../articles/app-service-mobile-xamarin-android-get-started-users.md)
- [Xamarin.Forms](../articles/app-service-mobile-xamarin-forms-get-started-users.md)
- [Xamarin.iOS](../articles/app-service-mobile-xamarin-ios-get-started-users.md)



&nbsp;  
>[AZURE.NOTE] This is an **Azure Mobile Apps** topic. For Mobile Services topics, see the [Mobile Services documentation center](/documentation/services/mobile-services/).
>
>App Service Mobile Apps is our newest mobile backend platform and [gives you additional advantages](app-service-mobile-value-prop-migration-from-mobile-services.md) over Mobile Services. [Migrating to App Service](app-service-mobile-migrating-from-mobile-services.md) is  recommended for customers using the .NET backend SDK. However, [Mobile Apps Node SDK](https://github.com/azure/azure-mobile-apps-node) is currently in Preview and is not yet recommended for production use. SDK and API contracts are subject to change within minor version releases.


## Summary
In this tutorial, you add authentication to the todolist quickstart project on Android using a supported identity provider. This tutorial is based on the [Get started with Mobile Apps](app-service-mobile-android-get-started.md) tutorial, which you must complete first.

## <a name="register"></a>Register your app for authentication and configure the App Service

First, you need to register your app at an identity provider's site, and then you will set the provider-generated credentials in the Mobile App backend.

1. Configure your preferred identity provider by following the provider-specific instructions: 
	
	+ [Azure Active Directory](../articles/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication.md)
	+ [Facebook](../articles/app-service-mobile/app-service-mobile-how-to-configure-facebook-authentication.md)
	+ [Google](../articles/app-service-mobile/app-service-mobile-how-to-configure-google-authentication.md)
	+ [Microsoft](../articles/app-service-mobile/app-service-mobile-how-to-configure-microsoft-authentication.md)
	+ [Twitter](../articles/app-service-mobile/app-service-mobile-how-to-configure-twitter-authentication.md)

2. Repeat the previous steps for each provider you want to support in your app.


<!-- URLs. -->
[Azure portal]: https://portal.azure.com/


## <a name="permissions"></a>Restrict permissions to authenticated users

By default, APIs in a Mobile App backend can be invoked anonymously. Next, you need to restrict access to only authenticated clients.  

+ **Node.js backend (via portal)** :  
	
	In your Mobile App's **Settings**, click **Easy Tables** and select your table. Click **Change permissions**, select **Authenticated access only** for all permissions, then click **Save**. 

+ **.NET backend (C#)**:  

	In the server project, navigate to **Controllers** > **TodoItemController.cs**. Add the `[Authorize]` attribute to the **TodoItemController** class, as follows. To restrict access only to specific methods, you can also apply this attribute just to those methods instead of the class. Republish the server project.


        [Authorize]
        public class TodoItemController : TableController<TodoItem>

+ **Node.js backend (via Node.js code)** :  
	
	To require authentication for table access, add the following line to the Node.js server script:


        table.access = 'authenticated';

	For more details, refer to [How to: Require authentication for access to tables](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#howto-tables-auth). To learn how to download the quickstart code project from your site, see [How to: Download the Node.js backend quickstart code project using Git](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart).



* In Android Studio, open the project that you created when you completed the tutorial [Get started with Mobile Apps](app-service-mobile-android-get-started.md), then from the **Run** menu click **Run app** and verify that an unhandled exception with a status code of 401 (Unauthorized) is raised after the app starts.

     This happens because the app attempts to access the backend as an unauthenticated user, but the *TodoItem* table now requires authentication.


Next, you will update the app to authenticate users before requesting resources from the Mobile App backend.

## Add authentication to the app

1. In **Project Explorer** in Android Studio, open the ToDoActivity.java file and add the following import statements.

		import java.util.concurrent.ExecutionException;
		import java.util.concurrent.atomic.AtomicBoolean;

		import android.content.Context;
		import android.content.SharedPreferences;
		import android.content.SharedPreferences.Editor;

		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceAuthenticationProvider;
		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceUser;

2. Add the following method to the **ToDoActivity** class: 
	
		private void authenticate() {
		    // Login using the Google provider.
		    
			ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.Google);
	
	    	Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
	    		@Override
	    		public void onFailure(Throwable exc) {
	    			createAndShowDialog((Exception) exc, "Error");
	    		}   		
	    		@Override
	    		public void onSuccess(MobileServiceUser user) {
	    			createAndShowDialog(String.format(
	                        "You are now logged in - %1$2s",
	                        user.getUserId()), "Success");
	    			createTable();	
	    		}
	    	});   	
		}


	This creates a new method to handle the authentication process. The user is authenticated by using a Google login. A dialog is displayed which displays the ID of the authenticated user. You cannot proceed without a positive authentication.

    > [AZURE.NOTE] If you are using an identity provider other than Google, change the value passed to the **login** method above to one of the following: _MicrosoftAccount_, _Facebook_, _Twitter_, or _windowsazureactivedirectory_.

3. In the **onCreate** method, add the following line of code after the code that instantiates the `MobileServiceClient` object.

		authenticate();

	This call starts the authentication process.

4. Move the remaining code after `authenticate();` in the **onCreate** method to a new **createTable** method, which looks like this:

		private void createTable() {
	
			// Get the table instance to use.
			mToDoTable = mClient.getTable(ToDoItem.class);
	
			mTextNewToDo = (EditText) findViewById(R.id.textNewToDo);
	
			// Create an adapter to bind the items with the view.
			mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);
			ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
			listViewToDo.setAdapter(mAdapter);
	
			// Load the items from Azure.
			refreshItemsFromTable();
		}

9. From the **Run** menu, then click **Run app** to start the app and sign in with your chosen identity provider. 

   	When you are successfully logged-in, the app should run without errors, and you should be able to query the backend service and make updates to data.

## <a name="cache-tokens"></a>Cache authentication tokens on the client

The previous example showed a standard sign-in, which requires the client to contact both the identity provider and the backend Azure service every time that the app starts. Not only is this method inefficient, you can run into usage-related issues should many customers try to start you app at the same time. A better approach is to cache the authorization token returned by the Azure service and try to use this first before using a provider-based sign-in. 

>[AZURE.NOTE]You can cache the token issued by the backend Azure service regardless of whether you are using client-managed or service-managed authentication. This tutorial uses service-managed authentication.


1. Open the ToDoActivity.java file and add the following import statements:

        import android.content.Context;
        import android.content.SharedPreferences;
        import android.content.SharedPreferences.Editor;

2. Add the the following members to the `ToDoActivity` class.

    	public static final String SHAREDPREFFILE = "temp";	
	    public static final String USERIDPREF = "uid";	
    	public static final String TOKENPREF = "tkn";	


3. In the ToDoActivity.java file, add the the following definition for the `cacheUserToken` method.
 
    	private void cacheUserToken(MobileServiceUser user)
	    {
    		SharedPreferences prefs = getSharedPreferences(SHAREDPREFFILE, Context.MODE_PRIVATE);
    	    Editor editor = prefs.edit();
	        editor.putString(USERIDPREF, user.getUserId());
    	    editor.putString(TOKENPREF, user.getAuthenticationToken());
	        editor.commit();
    	}	
  
    This method stores the user id and token in a preference file that is marked private. This should protect access to the cache so that other apps on the device do not have access to the token because the preference is sandboxed for the app. However, if someone gains access to the device, it is possible that they may gain access to the token cache through other means. 

    >[AZURE.NOTE]You can further protect the token with encryption if token access to your data is considered highly sensitive and someone may gain access to the device. However, a completely secure solution is beyond the scope of this tutorial and dependent on your security requirements.


4. In the ToDoActivity.java file, add the the following definition for the `loadUserTokenCache` method.

    	private boolean loadUserTokenCache(MobileServiceClient client)
	    {
	        SharedPreferences prefs = getSharedPreferences(SHAREDPREFFILE, Context.MODE_PRIVATE);
    	    String userId = prefs.getString(USERIDPREF, "undefined"); 
	        if (userId == "undefined")
	            return false;
    	    String token = prefs.getString(TOKENPREF, "undefined"); 
    	    if (token == "undefined")
    	        return false;
        	    
    	    MobileServiceUser user = new MobileServiceUser(userId);
    	    user.setAuthenticationToken(token);
    	    client.setCurrentUser(user);
        	    
    	    return true;
	    }



5. In the *ToDoActivity.java* file, replace the `authenticate` method with the following method which uses a token cache. Change the login provider if you want to use an account other than Google.

		private void authenticate() {
			// We first try to load a token cache if one exists.
		    if (loadUserTokenCache(mClient))
		    {
		        createTable();
		    }
		    // If we failed to load a token cache, login and create a token cache
		    else
		    {
			    // Login using the Google provider.    
				ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.Google);
		
		    	Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
		    		@Override
		    		public void onFailure(Throwable exc) {
		    			createAndShowDialog("You must log in. Login Required", "Error");
		    		}   		
		    		@Override
		    		public void onSuccess(MobileServiceUser user) {
		    			createAndShowDialog(String.format(
		                        "You are now logged in - %1$2s",
		                        user.getUserId()), "Success");
		    			cacheUserToken(mClient.getCurrentUser());
		    			createTable();	
		    		}
		    	});
		    }
		}

6. Build the app and test authentication using a valid account. Run it at least twice. During the first run, you should receive a prompt to login and create the token cache. After that, each run will attempt to load the token cache for authentication and you should not be required to login.





## Next steps
Now that you completed this basic authentication tutorial, consider continuing on to one of the following tutorials:

* [Add push notifications to your Android app](app-service-mobile-android-get-started-push.md)   
Learn how to add push notifications support to your app and configure your Mobile App backend to use Azure Notification Hubs to send push notifications.

* [Enable offline sync for your Android app](app-service-mobile-android-get-started-offline-data.md)  
Learn how to add offline support your app using an Mobile App backend. Offline sync allows end-users to interact with a mobile app&mdash;viewing, adding, or modifying data&mdash;even when there is no network connection. 


<!-- Anchors. -->

[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Store authentication tokens on the client]: #cache-tokens
[Refresh expired tokens]: #refresh-tokens
[Next Steps]:#next-steps


<!-- URLs. -->

[Get started with Mobile Apps]: app-service-mobile-android-get-started.md
