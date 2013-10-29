---
layout: doc
lang: java
title: Stormpath Java Product Guide
---

Stormpath is a User Management API that reduces development time with instant-on, scalable user infrastructure. Stormpath’s intuitive API and expert support make it easy for developers to authenticate, manage and secure users and roles in any application.

For help to quickly get started with Stormpath, refer to the [Java Quickstart Guide](/java/quickstart).

***

## What is Stormpath?

Stormpath is the first easy, secure user management and authentication service for developers. 

Fast and intuitive to use, Stormpath enables plug-and-play security and accelerates application development on any platform. 

Built for developers, it offers an easy API, open source SDKs, and an active community. The flexible cloud service can manage millions of users with a scalable pricing model that is ideal for projects of any size.

By offloading user management and authentication to Stormpath, developers can bring new applications to market faster, reduce development and operations costs, and protect their users with best-in-class security.

### Architectural Overview

<img src="http://www.stormpath.com/sites/default/files/docs/Architecture.png" alt="High-level Architecture" title="High-level Architecture" width="700" height="430">

### Stormpath Admin Console

The Stormpath Admin Console allows authorized administrators to:

* Configure applications to access Stormpath
* Create and manage accounts and adjust group membership
* Create and manage directories and the associated groups
* Map directories and groups to allow accounts to log in to integrated applications
* Configure workflow or account administration automation 

To access the Stormpath Admin Console, visit [https://api.stormpath.com/login](https://api.stormpath.com/login)

### REST API

The Stormpath API offers authorized developers and administrators programmatic access to:

* Securely authenticate accounts.
* Create and manage accounts and adjust group membership.
* Manage directories.
* Manage groups.
* Initiate and process account automations.

For more detailed documentation on the Stormpath API, visit the [REST Product Guide](/rest/product-guide).

### Java SDK

The Stormpath Java SDK allows any JVM-based application to easily use the Stormpath Identity and Access Management service for all authentication and access control needs. The Stormpath Java SDK allows any JVM-based application to easily use Stormpath for all user management and authentication needs. The Java SDK can be found on [**Github**](https://github.com/stormpath/stormpath-sdk-java).

When making SDK method calls, the calls are translated into HTTPS requests to the Stormpath REST+JSON API. The Stormpath Java SDK therefore provides a clean object-oriented paradigm natural to JVM developers and alleviates the need to know how to make REST+JSON requests. Any JVM-based programming language can use the Stormpath Java SDK. JVM languages include Java, Groovy, Scala, Clojure, Kotlin, Jython, and JRuby.

Stormpath also offers guides and SDKs for [Ruby](/ruby/product-guide), [PHP](/php/product-guide), and [Python](/python/product-guide).

If you are using a language that does not yet have an SDK, you can use the REST API directly and refer to the [REST API Product Guide](/rest/product-guide).

***

## SDK Concepts

Throughout this guide, code examples appear using the Stormpath Java SDK.

Although knowing how the SDK is designed and how it works is not required to use it, knowing some of the design details will help you use it more efficiently and effectively.

### Client

The root entry point for SDK functionality is the `Client` instance. Using the client instance, you can access all tenant data, such as applications, directories, groups, and accounts.

#### Preferred Configuration

There are different ways to create a client instance to interact with your resources. The preferred mechanism is by reading a secure `apiKey.properties` file, where the ClientBuilder implementation is being used:

	import com.stormpath.sdk.client.*;
	...

	String path = System.getProperty("user.home") + "/.stormpath/apiKey.properties";
	Client client = new ClientBuilder().setApiKeyFileLocation(path).build();


This is heavily recommended if you have access to the file system.

#### Single URL Configuration

The [Java Quickstart Guide](/java/quickstart) assumes you have easy access to a private `apiKey.properties` file. Some applications however might not have access to the file system (such as Heroku applications). In these cases, you can also create a client instance using a single URL, where often the URL is available as an environment variable, such as `System.getenv()`.

This technique requires embedding the API key ID and secret as components in a URL, which allows you to have a single URL that contains all necessary information required to construct a `Client`.

{% docs warning %}
Do not use this technique if you have access to a file system. Depending on the environment, environment variables are less secure than reading a permission-restricted file like `apiKey.properties`.
{% enddocs %}

If you do not have access to the file system to read an `apiKey.properties` file, you can use the `ClientApplicationBuilder` instead of the `ClientBuilder`. The `ClientApplicationBuilder` accepts a single URL of the application Stormpath HREF with the API key information embedded in the URL. For example:<br>

	String appHref = "https://apiKeyId:apiKeySecret@api.stormpath.com/v1/applications/YOUR_APP_UID_HERE";


Make sure that the `apiKeyId` and `apiKeySecret` components are URL-encoded.

Then you can acquire the [ClientApplication](https://github.com/stormpath/stormpath-sdk-java/blob/master/api/src/main/java/com/stormpath/sdk/client/ClientApplication.java) using the `ClientApplicationBuilder` to obtain a `Client` and an `Application`:</p>

	import com.stormpath.sdk.client.*;
	import com.stormpath.sdk.application.*;
	...
	...
	ClientApplication clientApplication = new   	ClientApplicationBuilder().setApplicationHref(appHref).build();

	Client client = clientApplication.getClient();
	Application application = clientApplication.getApplication();

The ClientApplicationBuilder is a powerful utility that can be used in different ways, including all of the [ClientBuilder](https://github.com/stormpath/stormpath-sdk-java/blob/master/api/src/main/java/com/stormpath/sdk/client/ClientBuilder.java) functionalities. To learn more about its usage you can read the [ClientApplicationBuilder](https://github.com/stormpath/stormpath-sdk-java/blob/master/api/src/main/java/com/stormpath/sdk/client/ClientApplicationBuilder.java) API documentation.</p>

#### API Key Configuration

Another way to create a client is by creating an `ApiKey` instance with the API credentials and passing this instance to create the client instance:</p>

	import com.stormpath.sdk.client.*;
	...
	...
	ApiKey apiKey = new ApiKey("myApiKeyId", "myApiKeySecret");
	Client client = new Client(apiKey);

{% docs warning %}
DO NOT specify your actual `apiKey.id` and `apiKey.secret` values in source code! They are secure values associated with a specific person. You should never expose these values to other people, not even other co-workers.

Only use this technique if the values are obtained at runtime using a configuration mechanism that is not hard-coded into source code or easily-visible configuration files.
{% enddocs %}

### High-level Overview

The Stormpath SDK and the associated components reside and execute within your application at runtime. When making method calls on the SDK objects - particularly objects that represent REST data resources such as applications and accounts - the method call automatically triggers an HTTPS request to the Stormpath API server if necessary.

The HTTPS request allows you to program your application code to use regular Java objects and alleviates you from worrying about the lower-level HTTP REST+JSON details and individual REST resource HTTP endpoints.

Here is how the communication works:

<img src="http://www.stormpath.com/sites/default/files/docs/SDKCommunicationFlow.png" alt="SDK Communication Flow" title="SDK Communication Flow" width="700">

In this example, the request is trying to determine the `directory` of the existing SDK `account` resource instance.

The request is broken down as follows:

1. The application attempts to acquire the account directory instance.
2. If the directory resource is not already in memory, the SDK creates a request to send to the Stormpath API server.
3. An HTTPS GET request is executed.
4. The Stormpath API server authenticates the request caller and looks up the directory resource.
5. The Stormpath API server responds with the JSON representation of the directory resource.
6. The Stormpath SDK transforms the JSON response into a directory object instance.
7. The directory instance is returned to the application.

### Detailed Design 
The Stormpath SDK is designed with two primary design principles:

* **Composition**<br> Although most SDK end users never need to customize the implementation behavior of an SDK, the SDK is pluggable meaning that the functionality can be customized by plugging in new implementations of relevant components. The SDK leans toward the <a href="http://en.wikipedia.org/wiki/Software_interface#Software_interfaces" title="software interfaces">programming to interfaces</a> and principles of <a href="http://en.wikipedia.org/wiki/Object_composition" title="object composition">object composition</a>to support pluggability. Even if SDK end users never leverage this design, the design helps the Stormpath development team provide support and bug fixes without disrupting existing SDK usages.
* **Proxying**<br> Java instances representing REST resources use the <a href="http://en.wikipedia.org/wiki/Proxy_pattern" title="proxy software design">Proxy software design pattern</a> to intercept property access allowing the SDK implementation to automatically load the resource data or other referenced resources if necessary.<br> For anyone familiar with Java proxying using reflection, the Stormpath SDK **does not** use reflection-based proxies. Proxies are implemented as standard Java implementations of interfaces for speed.

#### Architectural Components 

The core component concepts of the SDK are as follows:

<img src="http://www.stormpath.com/sites/default/files/docs/ComponentArchitecture.png" alt="Stormpath SDK Component Architecture" title="Stormpath SDK Component Architecture" width="670">

* **Client** is the root entry point for SDK functionality and accessing other SDK components, such as the `DataStore`. A client is constructed with a Stormpath API key which is required to communicate with the Stormpath API server. After it is constructed, the client delegates to an internal DataStore to do most of its work.
* **DataStore** is central to the Stormpath SDK. It is responsible for managing all Java `resource` objects that represent Stormpath REST data resources such as applications, directories, and accounts. The DataStore is also responsible for translating calls made on Java `resource`objects into REST requests to the Stormpath API server as necessary. It works between your application and the Stormpath API server.

	* **RequestExecutor** is an internal infrastructure component used by the `DataStore` to execute HTTP requests (`GET`, `PUT`, `POST`, `DELETE`) as necessary. When the DataStore needs to send a Java `Resource` instance state to or query the server for resources, it delegates to the RequestExecutor to perform the actual HTTP requests. The Stormpath SDK default `RequestExecutor` implementation is `HttpClientRequestExecutor` which uses the <a href="http://hc.apache.org/httpcomponents-client-ga/" title="Apache HttpClient">Apache HttpClient</a> to execute the raw requests and read the raw responses.
	* **MapMarshaller** is an internal infrastructure component used by the `DataStore` to convert JSON strings into Java `Map` instances or the reverse. The map instances are used by the `ResourceFactory` to construct standard Java objects representing REST resources. The Stormpath SDK default `MapMarshaller` implementation is `JacksonMapMarshaller` which uses the <a href="http://jackson.codehaus.org/" title="Jackson Java JSON processor">Jackson Java JSON processor</a>.
	* **ResourceFactory** is an internal infrastructure component used by the `DataStore` to convert REST resource map data into standard Java `resource` objects. The ResourceFactory uses Maps from `MapMarshaller` to construct the Java resource instances.
	* **Resources** are standard Java objects that have a 1-to-1 correlation with REST data resources in the Stormpath API server such as applications and directories. Applications directly use these `resource` objects for security needs, such as authenticating user accounts, creating groups and accounts, finding application accounts, assigning accounts to groups, and resetting passwords.
	

### Resources and Proxying 

When applications interact with a Stormpath SDK `resource` instance, they are really interacting with an intelligent data-aware proxy, not a simple object with some properties. Specifically, the `resource` instance is a proxy to the SDK client `DataStore` allowing resource instances to load data that might not yet be available.

For example, using the SDK Communication Flow diagram in the [high-level overview](#high-level-overview) section, assuming you have a reference to an `account` object - perhaps you have queried for it or you already have the account `href` and you want to load the `account` resource from the server:

	String href = "https://api.stormpath.com/v1/accounts/someAccountUidHere";
	Account account = dataStore.getResource(href, Account.class);

This retrieves the account at the specified `href` location using an HTTP `GET` request.

If you also want information about the `directory` owning that account, every account has a reference to the parent directory location in the JSON representation. For example:

	{
	  "givenName": "John",
	  "surname": "Smith",
	  ...
	  "directory": {
	      "href": "https://api.stormpath.com/v1/directories/someDirectoryIdHere"
	  }
	  ...
	}

Notice the JSON `directory` property is only a link (pointer) to the directory; it does not contain any of the directory properties. The JSON shows we should be able to reference the `directory` property, and then reference the `href` property, and do another lookup (pseudo code):

	//this code will not work, for demonstration purposes only:
	String directoryHref = account.getDirectoryLink().getHref();
	Directory directory = dataStore.getResource(directoryHref, Directory.class);

This technique is cumbersome, verbose, and requires a lot of boilerplate code in your project. As such, SDK resources **automatically** execute the lookups for unloaded references for you using simple property navigation!

The previous lookup becomes:

	Directory directory = account.getDirectory();

If the directory already exists in memory because the `DataStore` has previousy loaded it, the directory is immediately returned. However, if the directory is not present, the directory `href` is used to return the directory properties (the immediate data loaded) automatically for you. This technique is known as *lazy loading* which allows you to traverse entire object graphs automatically without requiring constant knowledge of `href` URLs.

### Error Handling 

Errors thrown from the server are translated to runtime [ResourceException](https://github.com/stormpath/stormpath-sdk-java/blob/master/api/src/main/java/com/stormpath/sdk/resource/ResourceException.java" title="ResourceException) errors. This applies to all requests to the Stormpath API endpoints.

For example, when getting the current tenant from the client you can catch any error that the request might produce as follows:

	import com.stormpath.sdk.client.*;
	import com.stormpath.sdk.resource.ResourceException;

	try {

	client.getCurrentTenant();

	} catch (ResourceException re) {
	    System.out.println("Message: " + re.getMessage());
	    System.out.println("HTTP Status: " + re.getStatus());
	    System.out.println("Stormpath Code: " + re.getCode()); 
	    System.out.println("Developer Message: " + re.getDeveloperMessage());
	    System.out.println("More Information: " + re.getMoreInfo());
	}

***

## Applications

An [application](#applications) in Stormpath represents a real world software application that communicates with Stormpath for its user management and authentication needs.

When defining an application in Stormpath, it is typically associated with one or more directories or groups. The associated directories and groups form the application *user base*. The accounts within the associated directories and groups are considered the application users and can login to the application.

For applications, the basic details include:

Attribute | Description
:--- | :---
Name | The name used to identify the application within Stormpath. This value must be unique.
Description | A short description of the application. 
Status | By default, this value is set to `enabled`. Change the value to `disabled` if you want to prevent accounts from logging in to the application.

{% docs note %} 
A URL for the application is often helpful. 
{% enddocs %}

For applications, you can:

* [Locate the application REST URL](#locate-the-application-rest-url)
* [List applications](#list-applications)
* [Retrieve an application](#retrieve-an-application)
* [Register an application](#register-an-application)
* [Edit the details of an application](#edit-an-application)
* [Manage application login sources](#manage-application-login-sources)
	* [Change default account and group locations](#change-default-account-and-group-locations)
	* [Add another login source](#add-another-login-source)
	* [Change the login source priority order](#change-login-source-priority-order).
	* [Remove login sources](#remove-login-sources)
* [Enable an application](#enable-an-application)
* [Disable an application](#disable-an-application)
* [Delete an application](#delete-an-application)

### Locate the Application REST URL

When communicating with the Stormpath REST API, you might need to reference an application using the REST URL or `href`. For example, you require the REST URL to list applications by issuing an API request. 

To obtain an application REST URL:

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. In the Applications table, click the application name.<br>
The REST URL appears on the Details tab.<br><img src="http://www.stormpath.com/sites/default/files/docs/AppResturl.png" alt="Application Resturl" title="Application Resturl">

### List Applications

The application browser enables you to view and search for integrated applications.

To retrieve a list of applications, you must do the following:

* Get the current tenant from a client instance.
* Get the applications from the current tenant.

**Code:**

	import com.stormpath.sdk.client.*;
	import com.stormpath.sdk.application.*;
	import com.stormpath.sdk.tenant.*;
	...
	Tenant tenant = client.getCurrentTenant();

	ApplicationList applications = tenant.getApplications();

	for (Application app : applications) {
	    System.out.println("Application " + app.getName());
	    System.out.println("Application href " + app.getHref());
	    System.out.println("Application Description " + app.getDescription());
	    System.out.println("Application Status " + app.getStatus());
	}

### Retrieve an Application 

You will typically retrieve an `Application` referenced from another resource, for example using the `ApplicationList` parameter.

You can also directly retrieve a specific application using the `href` (REST URL) value. For any application, you can [find the application href](#locate-the-application-rest-url) in the Stormpath console. This can be particularly useful when you need an application href for the `ClientApplicationBuilder`.

After you have the `href` it can be loaded directly as an object instance by retrieving it from the server, using the datastore:

	import com.stormpath.sdk.ds.*;
	import com.stormpath.sdk.application.*;
	...
	...
	DataStore dataStore = client.getDataStore(); 
	String href = "https://api.stormpath.com/v1/applications/APP_UID_HERE";
	Application application = dataStore.getResource(href, Application.class); 

### Register an Application 

To authenticate a user account in your [application](#applications), you must first register the application with Stormpath.

To register an application, you must:

* Retrieve your current tenant from the client instance.
* Instantiate an application object.
* Set the application properties.
* Create the application from your current tenant.

To create applications, use the `_setters_` of a new application instance to set the values and then create the application from the tenant, as follows:

	import com.stormpath.sdk.tenant.*;
	 import com.stormpath.sdk.application.*;
	 ...
	 ...
	 Tenant tenant = client.getCurrentTenant();

	 Application application = client.getDataStore().instantiate(Application.class);
	 application.setName("Application Name");
	 application.setDescription("Application Description");

	 application = tenant.createApplication(application);

### Edit an Application 

To edit applications, use the `_setters_` of an existing application instance to set the values and call the `save()` method:

	import com.stormpath.sdk.application.*;
	import com.stormpath.sdk.resource.Status;
	...
	...
	application.setStatus(Status.DISABLED);  // the Status enum class provides the valid 	status' constants
	application.setName("New Application Name");
	application.setDescription("New Application Description");

	application.save();

### Manage Application Login Sources

[Login sources](#login-sources) define the user base for a given application. Login sources determine which user account stores are used and the order in which they are accessed when a user account attempts to log in to your application.

In Stormpath, a directory or group can be a login source for an application. At least one login source must be associated with an application for accounts to log in to that application.

####How Login Attempts Work

**Example:**
Assume an application named Foo has been mapped to two login sources, the Customers and Employees directories, in that order.

Here is what happens when a user attempts to log in to an application named Foo:  

<img src="http://www.stormpath.com/sites/default/files/docs/LoginAttemptFlow.png" alt="Login Sources Diagram" title="Login Sources Diagram" width="650" height="500">

You can configure multiple login sources, but only one is required for logging in. Multiple login sources allows each application to view multiple directories as a single repository during a login attempt.

After an application has been registered, or created, within Stormpath, you can:

* [Change default account and group locations](#change-default-account-and-group-locations)
* [Add another login source](#add-another-login-source) (directories)
* [Change the login source priority order](#ChangeLoginSourcePriority)
* [Remove login sources](#RemoveLoginSource)

To manage application login sources, you must log in to the Stormpath Admin Console:

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. Click the application name.
4. Click the **Login Sources** tab.<br>
The login sources appear in order of priority.<br> 
	<img src="http://www.stormpath.com/sites/default/files/docs/LoginSources.png" alt="Login Sources" title="Login Sources" width="650" height="170">

<a name="change-default-account-and-group-locations"></a>
#### Change Default Account and Group Locations

On the Login Sources tab for applications, you can select the login sources (directory or group) to use as the default locations when creating new accounts and groups.

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. Click the application name.
4. Click the **Login Sources** tab.
	a. To specify the default creation location(directory) for new accounts created in the application, in the appropriate row, select **New Account Location**.
	b. To specify the default creation location(directory) for new groups created in the application, in the appropriate row, select **New Group Location**.
5. Click **Save**.

<a name="add-another-login-source"></a>
#### Add Another Login Source

Adding a login source to an application provisions a directory or group to that application.  By doing so, all login source accounts can log into the application.

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. Click the application name.
4. Click the **Login Sources** tab.
5. Click **Add Login Source**.
6. In the *login source* list, select the appropriate directory.<br>
	<img src="http://www.stormpath.com/sites/default/files/docs/LSDropdown1.png" alt="Login Sources" title="Login Sources"><br>
7.  If the directory contains groups, you can select all users or specific group for access.<br> 
	<img src="http://www.stormpath.com/sites/default/files/docs/LSDropdown2.png" alt="Login Sources" title="Login Sources"><br>
8. Click **Add Login Source**.<br>
The new login source is added to the bottom of the login sources list.    

<a name="change-login-source-priority-order"></a>
#### Change Login Source Priority Order

When you map multiple login sources to an application, you must also define the login source order.

The login source order is important during the login attempt for a user account because of cases where the same user account exists in multiple directories. When a user account attempts to log in to an application, Stormpath searches the listed login sources in the order specified, and uses the credentials (password) of the first occurrence of the user account to validate the login attempt.

To specify the login source order:

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. Click the application name.
4. Click the **Login Sources** tab.
5. Click the row of the directory to move.
6. Drag the row to the appropriate order.<br>
	For example, if you want to move the first login source to the second login source, click anywhere in the first row of the login source table and drop the row on the second row.<br>
	<img src="http://www.stormpath.com/sites/default/files/docs/LoginPriority.png" alt="Login Sources" title="Login Sources" width="650">
7. Click **Save Priorities**.

<a name="remove-login-sources"></a>
#### Remove Login Sources

Removing a login source from an application deprovisions that directory or group from the application. By doing so, all accounts from the login source are no longer able to log into the application.

To remove a login source from an application:

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. Click the application name.
4. Click the **Login Sources** tab.
5. On the Login Sources tab, locate the directory or group.
6. Under the Actions column, click **Remove**.

### Enable an Application 

Enabling an application allows any enabled directories, groups, and accounts associated with the application login sources in Stormpath to log in.

To enable an application, you must:

* Get the datastore instance from a client instance.
* Get the application instance from the datastore with the application `href`.
* Set the `ENABLED` status to the application instance.
* Call the `save` method on the application instance.

**Code:**

	import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.application.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/applications/APPLICATION_UID_HERE";
	 Application application = dataStore.getResource(href, Application.class); 
	 application.setStatus(Status.ENABLED);  // the Status enum class provides the valid status' constants

	 application.save();

### Disable an Application 

Disabling an application prevents the application from logging in any accounts in Stormpath, but retains all application configurations. If you must temporarily turn off logins, disable an application.

The Stormpath application cannot be disabled.

To disable an application, you must:

* Get the datastore instance from a client instance.
* Get the application instance from the datastore with the application `href`.
* Set the `DISABLED` status to the application instance.
* Call the `save` method on the application instance.

**Code:**

	import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.application.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/applications/APPLICATION_UID_HERE";
	 Application application = dataStore.getResource(href, Application.class); 
	 application.setStatus(Status.DISABLED);  // the Status enum class provides the valid status' constants

	 application.save();

### Delete an Application 

Deleting an application completely erases the application and its configurations from Stormpath.

We recommend that you disable an application rather than delete it, if you anticipate that you might use the application again.

The Stormpath application cannot be deleted.

To delete an application, you must use the Stormpath Admin Console.

1. Log in to the Stormpath Admin Console.
2. Click the **Applications** tab.
3. Under the Actions column, click **Delete**. <br> The application is erased from Stormpath and no longer appears in the application browser.

***

## Directories

[Directories](#directories) contain [authentication](#authenticate-accounts) and [authorization](#list-groups) information about users and groups. Stormpath supports an unlimited number of directories. Administrators can use different directories to create silos of users. For example, you might store your customers in one directory and your employees in another.

For directories, the basic details include:</p>

Attribute | Description
:----- | :-----
Name | The name used to identify the directory within Stormpath. This value must be unique.
Description | Details about this specific directory.
Status | By default, this value is set to Enabled. Change the value to Disabled if you want to prevent all user accounts in the directory from authenticating even where the directory is set as a login source to an application.

Within Stormpath, there are two types of directories you can implement:

* A <strong>Cloud</strong> directory, also known as Stormpath-managed directories, which are hosted by Stormpath and use the Stormpath data model to store user and group information. This is the default and most common type of directory in Stormpath.
* A <strong>Mirrored</strong> directory, which is a Stormpath-hosted directory populated with data pushed from your existing LDAP/AD directory using a Stormpath synchronization agent. All user management is done on your existing LDAP/AD directory, but the cloud mirror can be accessed through the Stormpath APIs on your modern applications.

	* Agent directories cannot be created using the API.
	* You can specify various LDAP/AD object and attribute settings of the specific LDAP/AD server for users and groups.
	* If the agent status is Online, but you are unable to see any data when browsing your LDAP/AD mapped directory, it is likely that your object and filters are configured incorrectly.

You can add as many directories of each type as you require. Changing group memberships, adding accounts, or deleting accounts in directories affects ALL applications to which the directories are mapped as [login sources](#manage-application-login-sources).

LDAP/AD accounts and groups are automatically deleted when:

* The backing object is deleted from the LDAP/AD directory service.
* The backing LDAP/AD object information no longer matches the account filter criteria configured for the agent.
* The LDAP/AD directory is deleted.

For directories, you can:

* [Locate the directory REST URL](#locate-the-directory-rest-url)
* [List directories](#list-directories)
* [Create a directory](#create-a-directory)
	* [Create a cloud directory](#create-a-cloud-directory)
	* [Create a mirrored (LDAP) directory](#create-a-mirrored-directory)
* [Map directories to applications](#map-directories-to-applications)
* [Edit directory details](#edit-directory-details)
* [Update agent configuration](#update-agent-configuration)
* [Enable a directory](#enable-a-directory)
* [Disable a directory](#disable-a-directory)
* [Delete a directory](#delete-a-directory)

### Locate the Directory REST URL 

When communicating with the Stormpath REST API, you might need to reference a directory using the REST URL or `href`. For example, you require the REST URL to create accounts in the directory using an SDK.

To obtain a directory REST URL:

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. In the Directories table, click the directory name.<br> The REST URL appears on the Details tab.<br> <img src="http://www.stormpath.com/sites/default/files/docs/Resturl.png" alt="Application Resturl" title="Application Resturl">

### List Directories 

To retrieve directories, you must:

1. Get the current tenant from a client instance.
2. Get the directories from the current tenant.

**Code:**

	import com.stormpath.sdk.client.*;
	import com.stormpath.sdk.directory.*;
	import com.stormpath.sdk.tenant.*;
	...
	...
	Tenant tenant = client.getCurrentTenant();

	DirectoryList directories = tenant.getDirectories();

	for (Directory dir : directories) {
	    System.out.println("Directory " + dir.getName());
	    System.out.println("Directory href " + dir.getHref());
	    System.out.println("Directory Description " + dir.getDescription());
	    System.out.println("Directory Status " + dir.getStatus());
	}

### Retrieve a Directory 

You will typically retrieve a `Directory` linked from another resource.

You can also retrieve it or as a direct reference, such as `account.getDirectory();`.

Finally, you can also directly retrieve a specific directory using the `href` (REST URL) value. For any directory, you can find [the directory href](#locate-the-directory-rest-url) in the Stormpath console.

After you have the `href` it can be loaded directly as an object instance by retrieving it from the server, using the datastore:

	import com.stormpath.sdk.client.*;
	import com.stormpath.sdk.directory.*;
	import com.stormpath.sdk.ds.*;
	...
	...
	DataStore dataStore = client.getDataStore(); 
	String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	Directory directory = dataStore.getResource(href, Directory.class); 

### Create a Directory 

To create a directory for application authentication, you must know which type of directory service to use.

You can create a:

* [Cloud Directory](#create-a-cloud-directory), which is hosted by Stormpath and uses the Stormpath data model to store user and group information. This is the most common type of directory in Stormpath.

**OR**

* [Mirrored (LDAP) agent directory](#create-a-mirrored-directory), which uses a synchronization agent for your existing LDAP/AD directory. All user account management is done on your existing LDAP/AD directory with the Stormpath agent mirroring the primary LDAP/AD server.

{% docs note %}
The ability to create a mirrored, or agent, directory is connected to your subscription. If the option is not available, click the question mark for more information.
{% enddocs %}


<a name="create-a-cloud-directory"></a>
#### Create a Cloud Directory 

1. Click the **Directories** tab.
2. Click **Create Directory**.
3. Click **Cloud**.
4. Complete the field values noted in the table that follows.
5. Click **Create**. 

<br> <img src="http://www.stormpath.com/sites/default/files/docs/CreateCloudDirectory.png" alt="Create Cloud Directory" title="Create Cloud Directory" width="650" height="460">

Attribute | Description
:----- | :-----
Name | The name used to identify the directory within Stormpath. This value must be unique.
Description | Details about this specific directory.
Status | By default, this value is set to Enabled. Change the value to Disabled if you want to prevent all user accounts in the directory from authenticating even where the directory is set as a login source to an application.
Min characters | The minimum number of acceptable characters for the account password.
Max characters | The maximum number of acceptable characters for the account password.
Mandatory characters | The required character patterns which new passwords will be validated against. For example, for an alphanumeric password of at least 8 characters with at least one lowercase and one uppercase character, select the abc, ABC, and 012 options. The more patterns selected, the more secure the passwords but the more complicated for a user.

<a name="create-a-mirrored-directory"></a>
#### Create a Mirrored Directory

Mirrored directories, after initial configuration, are accessible through the Agents tab of the directory. 

1. Click the **Directories** tab.
2. Click **Create Directory**.
3. Click **Mirror**. 

	<img src="http://www.stormpath.com/sites/default/files/docs/CreateLDAPDirectory.png" alt="Create LDAP Directory" title="Create Mirrored Directory" width="650">

4. On the 1. Directory Basics tab, complete the field values as follows:
	
	Attribute | Description
:----- | :-----
Directory Name | A short name for this directory, unique from your other Stormpath directories.
Directory Description | An optional description explaining the purpose for the directory.
Directory Status | Whether or not the directory is to be used to authenticate accounts for any assigned applications. By default, this value is set to Enabled. Change the value to Disabled if you want to prevent all user accounts in the directory from authenticating even where the directory is set as a login source to an application.

5. Click **Next**.

	<img src="http://www.stormpath.com/sites/default/files/docs/CreateLDAP2.png" alt="Agent Configuration" title="Agent Configuration" width="640">
	
6. On the 2. Agent Configuration tab, complete the field values as follows::
	
	Attribute | Description
	:----- | :-----
Directory Service | The type of directory service to be mirrored. For example, LDAP.
Directory Host | The IP address or Host name of the directory server to connect to. This domain must be accessible to the agent, for example, behind any firewall that might exist.
Directory Port | The directory server port to connect to. Example: `10389` for LDAP, `10689` for LDAPS, however your directory server maybe configured differently.
Use SSL | Should the agent socket connection to the directory use Secure Sockets Layer (SSL) encryption? The Directory Port must accept SSL.
Agent User DN | The username used to connect to the directory. For example: `cn=admin,cn=users,dc=ad,dc=acmecorp,dc=com`
Agent Password | The agent user DN password.
Base DN | The base Distinguished Name (DN) to use when querying the directory. For example, `o=mycompany,c=com`
Directory Services Poll Interval | How often (in minutes) to poll the directory to detect directory object changes.

7. Click **Next**.

	<img src="http://www.stormpath.com/sites/default/files/docs/CreateLDAP3.png" alt="Account Configuration" title="Account Configuration" width="640">

		
8. On the 3. Account Configuration tab, complete the field values as follows:
		
	Attribute | Description 
:----- | :-----
Account DN Suffix | Optional value appended to the Base DN when accessing accounts. For example, `ou=Users`. If left unspecified, account searches will stem from the Base DN.
Account Object Class | The object class to use when loading accounts. For example, `user`
Account Object Filter | The filter to use when searching user objects.
Account Email Attribute | The attribute field to use when loading the user email address. Example: `email`
Account First Name Attribute | The attribute field to use when loading the account first name.  Example: `givenname`
Account Middle Name Attribute | The attribute field to use when loading the account middle name. Example: `middlename`
Account Last Name Attribute | The attribute field to use when loading the account last name. Example: `sn`
Account Login Name Attribute |  The attribute field to use when logging in the account. Example: `uid`
Account Password Attribute | The attribute field to use when loading the account password. Example: `password`

9. Click **Next**.

	<img src="http://www.stormpath.com/sites/default/files/docs/CreateLDAP4.png" alt="Group Configuration" title="Group Configuration" width="640">

10. On the 4. Group Configuration tab, complete the field values as follows:
	
	Attribute | Description 
:----- | :-----
Group DN Suffix | This value is used in addition to the base DN when searching and loading roles. An example is `ou=Roles`. If no value is supplied, the subtree search will start from the base DN.
Group Object Class | This is the name of the class used for the LDAP group object. Example: `group`
Group Object Filter | The filter to use when searching for group objects.
Group Name Attribute | The attribute field to use when loading the group name. Example: `cn`
Group Description Attribute | The attribute field to use when loading the group description. Example: `desc`
Group Members Attribute | The attribute field to use when loading the group members. Example: `member`

11. Click **Next**.
12. On the 5. Confirm tab, review the information and click **Create Directory**.<br>The webpage refreshes with the populated directory information. 
13. Review the Download Agent tab and perform the steps as directed.
	
	<img src="http://www.stormpath.com/sites/default/files/docs/LastLDAPCreate.png" alt="Download Agent" title="Download Agent">

The `agent.id` and `agent.key` values will be specific to the agent of this directory.

After creating a backed directory in the Stormpath Admin Console and installing the Stormpath agent, the synchronization of the directory data starts when you start the agent.

After the agent is configured, associated with the agent is a status. This status is the **agent communication status** which reflects communication state as the agent communicates with Stormpath. As you make changes to the agent configuration, those changes are automatically pushed to the remote client and applied. Any further errors or conditions appear under the status column. The valid status codes are:

* **Online**: The agent is online and all things are working nominally.
* **Offline**: Stormpath detects that the agent is not communicating with it at all.
* **Error**: The agent is online, but there is a problem with communicating nominally with Stormpath or LDAP.

{% docs note %}
After the directory has been created, although the Workflows tab appears, workflows cannot be configured for this type of directory.
{% enddocs %}

### Map Directories to Applications 

Currently, you can only associate directories with application in the Stormpath Admin Console.

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. Click the directory name.
4. Click the **Applications** tab.<br> The applications table shows the application for which the directory is providing account authentication, or log in, credentials.
5. To change the login source, you must modify the application login source information.<br> If the directory is currently not specified as a login source for an application, the table contains the following message:<br> 

{% docs note %}
Currently, there are no applications associated with this directory. To create an association, click here, and select an application. From the login sources tab, you can create the association.
{% enddocs %}

### Edit Directory Details 

To edit directories, use the `_setters_` of an existing directory instance to set the values and call the `save()` method:

	import com.stormpath.sdk.directory.*;
	import com.stormpath.sdk.resource.Status;
	...
	...
	directory.setStatus(Status.DISABLED);  // the Status enum class provides the valid status' constants
	directory.setName("New Directory Name");
	directory.setDescription("New Directory Description");

	directory.save();

### Update Agent Configuration

You can modify an agent configuration going through the "Directories" or "Agent" tabs.

The Agents tab contains a table listing all known agents used by you. Each table entry shows the following:

* The **Stormpath directory name** as a link which you can click to modify any parameters.
* A **description** which is pulled from the directory Details tab.
* The agent communication **status** reflects communication state as the agent communicates with Stormpath. As you make changes to the agent configuration, those changes are automatically pushed to the remote client and applied. Any further errors or conditions appear under the status column. The valid status codes are:

	* **Online**: The agent is online and all things are working nominally.
	* **Offline**: Stormpath detects that the agent is not communicating with it at all.
	* **Error**: The agent is online, but there is a problem with communicating nominally with Stormpath or LDAP/AD.

Although the Workflows tab appears for a mirrored LDAP/AD directory, workflows cannot be configured for this type of directory.

#### Directories Tab
1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. Click the directory name.
4. Click the **Agent Configuration** tab.
	{% docs note %}If you do not see an Agent Configuration tab, you are looking at a Stormpath cloud directory.{% enddocs %}
5. Make the necessary changes and click **Update**. 

#### Agents Tab
1. Log in to the Stormpath Admin Console.
2. Click the **Agents** tab.
3. Click the directory name.
4. Make the necessary changes and click **Update**.


### Enable a Directory 

Enabling previously disabled directories allows the groups and accounts to log into any applications for which the directory is defined as a login source.

To enable a directory, you must:

1. Get the datastore instance from a client instance.
2. Get the directory instance from the datastore with the href of the directory.
3. Set the directory instance to enabled.
4. Call the save method on the directory instance.

**Code:**

	import com.stormpath.sdk.client.*;
	 import com.stormpath.sdk.directory.*;
	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	 Directory directory = dataStore.getResource(href, Directory.class); 

	 directory.setStatus(Status.ENABLED);  // the Status enum class provides the valid status' constants

	 directory.save();

### Disable a Directory 

Disabling directories prevents the groups and accounts from logging into any applications connected to Stormpath, but retains the directory, group, and account data. If you must shut off several accounts quickly and easily, disable a directory.

The Stormpath Administrators directory cannot be disabled.

To disable a directory, you must:

1. Get the datastore instance from a client instance.
2. Get the directory instance from the datastore with the href of the directory.
3. Set the directory instance to disabled.
4. Call the save method on the directory instance.

**Code:**

	 import com.stormpath.sdk.client.*;
	 import com.stormpath.sdk.directory.*;
	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	 Directory directory = dataStore.getResource(href, Directory.class); 

	 directory.setStatus(Status.DISABLED);  // the Status enum class provides the valid status' constants

	 directory.save();

### Delete a Directory 

Deleting a directory completely erases the directory and all group and account data from Stormpath.

We recommend that you disable a directory rather than delete it, in case an associated application contains historical data associated with accounts in the directory.

The Stormpath Administrators directory cannot be deleted.

To delete a directory, you must use the Stormpath Admin Console.

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. Locate the directory and, in the Actions column, click **Delete**.

***

## Accounts 

In Stormpath, users are referred to as user account objects or [accounts](#accounts). The username and email fields for accounts are unique within a directory and are used to log into applications. Within Stormpath, an unlimited number of accounts per directory is supported. 

You manage LDAP/AD accounts on your primary LDAP/AD installation. LDAP/AD accounts and groups are automatically deleted when:

* The backing object is deleted from the LDAP/AD directory service.
* The backing LDAP/AD object information no longer matches the account filter criteria configured for the agent.
* The LDAP/AD directory is deleted.

An account is a unique identity within a directory. An account can exist in only a single directory but can be a part of multiple groups owned by that directory.

For accounts, the the basic detail information includes:

Attribute | Description
:----- | :-----
Directory | The directory to which the account will be added.
Username | The login name of the account for applications using username instead of email. The value must be unique within its parent directory.
First Name | The account owner first name.
Middle Name | The account owner middle name.
Last Name | The account owner last name.
First Name | The account owner first name.
Email | The account owner email address. This is can be used by applications, such as the Stormpath Admin Console, that use an email address for logging in. The value must be unique within its parent directory.
Status | The status is set to Enabled by default. It is only set to Disabled if you want to deny access to the account to Stormpath-connected applications.
Password | The credentials used by an account during a login attempt. The specified value must adhere to the password policies set for the parent directory.

{% docs note %} 
The account cannot be moved to a different directory after it has been created. 
{% enddocs %}

For accounts, you can: 

* [Locate the account REST URL](#locate-the-account-rest-url)
* [Authenticate accounts](#authenticate-accounts)
* [List accounts](#list-accounts)
	* [List accounts by group](#list-accounts-by-group)
	* [List accounts by directory](#list-accounts-by-directory)
	* [List accounts by application](#list-accounts-by-application)
* [Create an account](#create-an-account)
* [Edit account details](#edit-account-details)
* [Change an account password](#change-an-account-password)
* [Assign accounts to groups](#assign-accounts-to-groups)
* [Remove accounts from groups](#remove-accounts-from-groups)
* [Enable an account](#enable-accounts)
* [Disable an account](#disable-accounts)
* [Delete an account](#delete-an-account)

### Locate the Account REST URL 

When communicating with the Stormpath REST API, you might need to reference an account using the REST URL or `href`. For example, you require the REST URL to create accounts in the directory using an SDK. 

To obtain an account REST URL:

1. Log in to the Stormpath Admin Console.
2. Click the **Accounts** tab.
3. In the Accounts table, click the account name.<br>
The REST URL appears on the Details tab.

### Authenticate Accounts 

To authenticate an account you must have the application the account authenticates against. With the application, the account is authenticated by providing the username and password as follows:

	import com.stormpath.sdk.client.*;
	import com.stormpath.sdk.ds.*;
	import com.stormpath.sdk.application.*;
	import com.stormpath.sdk.account.*;
	import com.stormpath.sdk.authc.*;
	...

	String userHome = System.getProperty("user.home");
	String path = userHome + "/.stormpath/apiKey.properties";
	Client client = new ClientBuilder().setApiKeyFileLocation(path).build();
	DataStore dataStore = client.getDataStore();

	String href = "https://api.stormpath.com/v1/applications/APP_UID_HERE";
	Application application = dataStore.getResource(href, Application.class);

	// when the account is authenticated, it produces an AuthenticationResult
	String un = "usernameOrEmail"; //get from form
	String pw = "password"; //get from form
	AuthenticationRequest request = new UsernamePasswordRequest(un,pw);
	AuthenticationResult result = application.authenticateAccount(request);

	// from the result instance we get the authenticated Account
	Account account = result.getAccount();

### List Accounts 

For accounts, you can view, or list them according to their [group membership](#list-group-accounts), [the directories to which they belong](#list-directory-groups), or [the applications to which they are associated](#map-directories-to-applications).

<a name="list-accounts-by-group"></a>
#### List Accounts in a Group 

To list user accounts that are members of a certain group, you must:

1. Get the datastore instance from the client.
2. Get the group from the datastore with the group `href`.
3. Get the group accounts.

**Code:**

	import com.stormpath.sdk.group.*;
	 import com.stormpath.sdk.client.*;
	 import com.stormpath.sdk.ds.*;
	 ...    
	 ...
	 DataStore dataStore = client.getDataStore(); 
	 String href = "https://api.stormpath.com/v1/groups/GROUP_UID_HERE";
	 Group group = dataStore.getResource(href, Group.class);
	 AccountList accounts = group.getAccounts();

	 for (Account acc : accounts) {
	     System.out.println("Given Name " + acc.getGivenName());
	 }

<a name="list-accounts-by-directory"></a>
#### List Accounts in a Directory 

To list user accounts contained in a directory, you must:

1. Get the datastore instance from the client.
2. Get the directory from the datastore with the directory `href`.
3. Get the directory accounts from the directory instance.

**Code:**

	 import com.stormpath.sdk.directory.*;
	 import com.stormpath.sdk.client.*;
	 import com.stormpath.sdk.ds.*;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 
	 String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	 Directory directory = dataStore.getResource(href, Directory.class);
	 AccountList accounts = directory.getAccounts();

	 for (Account acc : accounts) {
	     System.out.println("Given Name " + acc.getGivenName());
	 }

<a name="list-accounts-by-application"></a>
#### List Accounts by Application 

To list user accounts mapped to an application, you must:

1. Get the datastore instance from the client.
2. Get the application from the datastore with the application `href`.
3. Get the application accounts.

**Code:**

	 import com.stormpath.sdk.application.*;
	 import com.stormpath.sdk.client.*;
	 import com.stormpath.sdk.ds.*;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 
	 String href = "https://api.stormpath.com/v1/applications/APPLICATION_UID_HERE";
	 Application application = dataStore.getResource(href, Application.class);
	 AccountList accounts = application.getAccounts();

	 for (Account acc : accounts) {
	 System.out.println("Given Name " + acc.getGivenName());
	 }

### Retrieve an Account 

To retrieve a specific account you need the `href` which can be loaded as an object instance by retrieving it from the server, using the datastore:

	import com.stormpath.sdk.ds.*;
	import com.stormpath.sdk.account.*;
	...
	...
	DataStore dataStore = client.getDataStore(); 
	String href = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";
	Account account = client.getDataStore().getResource(href, Account.class); 

### Create an Account 

To create a user accounts, you must:

1. Get the datastore instance from the client.
2. Get the directory where you want to create the account from the datastore with the directory href.
3. Set the account properties.
4. Create the account from the directory.

To create accounts, use the `_setters_` of a new account instance to set the values and create the account in a directory as follows:

**Code:**

	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.directory.*;
	 ...
	 ...
	 String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	 Directory directory = client.getDataStore().getResource(href, Directory.class);

	 Account account = client.getDataStore().instantiate(Account.class);
	 account.setGivenName("Given Name");
	 account.setSurname("Surname");
	 account.setUsername("Username");
	 account.setEmail("Email"); 
	 account.setMiddleName("Middle Name");
	 account.setPassword("Password");

	 directory.createAccount(account);

If you want to override the registration workflow and have the account created with ENABLED status right away, you may pass false as second argument, for example:

	directory.createAccount(account, false);

If you want to associate the account to a group, add the following:

	account.addGroup(group);

### Edit Account Details 

You can edit accounts using the `_setters_` of an existing account instance to set the values and call the `save();` method:

	import com.stormpath.sdk.account.*;
	import com.stormpath.sdk.resource.Status;
	...
	...
	account.setStatus(Status.DISABLED);  // the Status enum class provides the valid status'    constants
	account.setGivenName("New Given Name");
	account.setSurname("New Surname");
	account.setUsername("New Username");
	account.setEmail("New Email");
	account.setMiddleName("New Middle Name");

	account.save();

If you want to add a group to an account, do the following:

	account.addGroup(group);

### Assign Accounts to Groups 

The association between a group and an account can be done from an account or group instance. If the account is part of a directory containing groups, you can associate the account with (or make the account a member of) a group. To add an account to a group, you must:

1. Get the datastore instance from a client instance.
2. Get the group and account instance from the datastore with the corresponding hrefs.
3. Add the group to the account instance OR add the account to the group instance.

**Code:**

	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.group.*;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore();

	 String href = "https://api.stormpath.com/v1/groups/GROUP_UID_HERE";
	 Group group = dataStore.getResource(href, Group.class);

	 href = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";
	 Account account = dataStore.getResource(href, Account.class);

	 // account.addGroup(group) OR group.addAccount(account)

### Remove Accounts from Groups 

The remove an account from, or delete the account as a member of, a group you must:

1. Get the datastore instance from a client instance.
2. Get the group membership instance.
	* The group membership can be retrieved directly from the datastore, if the `href` is known to the user.
	* Another way of retrieving the group membership is by searching for the group membership that represents the relationship between the group and the account that you want to delete.
3. Delete the group membership by calling the `delete` method.

**Code:**

	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.group.*;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore();

	 String href = "https://api.stormpath.com/v1/groupMemberships/GROUP_MEMBERSHIP_UID_HERE";
	 GroupMembership groupMembership = dataStore.getResource(accountHref, GroupMembership.class);

	 groupMembership.delete();

**OR**

	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.group.*;
	 ...
	 ...
	 String groupHref = "https://api.stormpath.com/v1/groups/GROUP_UID_HERE";

	 String accountHref = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";
	 Account account = client.getDataStore.getResource(accountHref, Account.class);

	 boolean groupLinked = false;
	 GroupMembership groupMembership = null;
	 // looping the group membership aggregate of the account
	 for (GroupMembership tmpGroupMembership : account.getGroupMemberships()) {

	 groupMembership = tmpGroupMembership;
	 Group tmpGroup = groupMembership.getGroup();

	 // here, we make sure this is the group we're looking for
	 if (tmpGroup != null &amp;&amp; tmpGroup.getHref().contains(groupHref)) {

	    groupLinked = true;
	    break;
	  } 
	 }

	 // if the group was found, we delete it
	 if (groupLinked) {

	 groupMembership.delete();
	 }

### Enable Accounts 
Enabling a previously disabled account allows the account to log in to any applications where the directory or group is defined as an application login source.

{% docs note %}
Enabling and disabling accounts for mirrored (LDAP) directories is not available in Stormpath. You manage mirrored (LDAP) accounts on the primary server installation.
{% enddocs %}

To enable an account, you must:

1. Get the datastore instance from a client instance.
2. Get the account instance from the datastore with the account href.
3. Set the account instance status to enabled.
4. Call the save method on the account instance.

**Code:**

	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";
	 Account account = client.getDataStore().getResource(href, Account.class); 

	 account.setStatus(Status.ENABLED);  // the Status enum class provides the valid status' constants

	 account.save();

### Disable Accounts 

Disabling an account prevents the account from logging into any applications in Stormpath, but retains all account information. You typically disable an account if you must temporarily remove access privileges.

If you disable an account within a directory or group, you are completely disabling the account from logging in to any applications to which it is associated.

{% docs note %}
Enabling and disabling accounts for mirrored (LDAP) directories is not available in Stormpath. You manage mirrored (LDAP) accounts on the primary server installation.
{% enddocs %}

To disable an account, you must:

1. Get the datastore instance from a client instance.
2. Get the account instance from the datastore with the account href.
3. Set the account instance status to disabled.
4. Call the save method on the account instance.

**Code:**

	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";
	 Account account = client.getDataStore().getResource(href, Account.class); 

	 account.setStatus(Status.DISABLED);  // the Status enum class provides the valid status' constants

	 account.save();

### Delete an Account 

Deleting an account completely erases the account from the directory and erases all account information from Stormpath.

To delete an account, use the account object's `delete` function to delete the object:

**Code:**

	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";
	 Account account = client.getDataStore().getResource(href, Account.class); 

	 account.delete();

{% docs warning %}
Once an account has been deleted, it cannot be recovered. All information associated with the account resource (e.g., username, email, encrypted password, and so on) is permanently deleted. 
{% enddocs %}

## Groups

[Groups](#Group) are collections of accounts within a directory that are often used for authorization and access control to the application. In Stormpath, the term group is synonymous with [role](#Role).

You manage LDAP/AD groups on your primary LDAP/AD installation. LDAP/AD accounts and groups are automatically deleted when:

* The backing object is deleted from the LDAP/AD directory service.
* The backing LDAP/AD object information no longer matches the account filter criteria configured for the agent.
* The LDAP/AD directory is deleted.

For groups, the basic detail information includes:

Attribute | Description
:--- | :---
Name | The name of the group. Within a given directory, this value must be unique.
Description | A short description of the group.
Status | This is set to Enabled by default. This is only set to Disabled to prevent all group accounts from logging into any application even when the group is set as a login source to an application. 

{% docs note %} 
If an account is also a member to another group that does have access to an application, then the account can login. 
{% enddocs %}

With groups, you can:

* [Locate the group REST URL](#locate-the-group-rest-url)
* [List groups](#list-groups)
	* [List group accounts](#list-group-accounts)
	* [List directory groups](#list-directory-groups)
* [Create groups](#create-groups)
* [Edit group details](#edit-group-details)
* [Enable a group](#enable-a-group)
* [Disable a group](#disable-a-group)
* [Delete a group](#delete-a-group)

### Locate the Group REST URL

When communicating with the Stormpath REST API, you might need to reference a group using the REST URL or `href`. For example, you require the REST URL to create accounts to associate with the group in the directory using an SDK. 

To obtain a group REST URL:

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. In the Directories table, click the directory name.
4. Click the **Groups** tab.
5. Click the group name.<br>
The REST URL appears on the Details tab.

### List Groups 

For groups, you can view, or list them by [account membership](#list-account-groups) or [the directory](#list-directory-groups).

<a name="list-group-accounts"></a>

#### List Accounts in a Group 

To list all groups associated with an account, you must:

1. Get the datastore instance from a client instance.
2. Get the account instance from the datastore with the account href.
3. Get the groups from the account.

To list all groups on a directory or an account, loop the groups aggregate from a directory or an account:

	import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.account.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore();  

	 String href = "https://api.stormpath.com/v1/accounts/ACCOUNT_UID_HERE";

	 Account account = client.getDataStore().getResource(href, Account.class); 
	
	 GroupList groups = account.getGroups();

	 for (Group grp : groups) {
	 System.out.println("Group " + grp.getName());
	 }

<a name="list-directory-groups"></a>
#### List Groups in a Directory 

To list all groups contained within a directory, you must:

1. Get the datastore instance from a client instance.
2. Get the directory instance from the datastore with the directory href.
3. Get the groups from the directory.

To list all groups on a directory or an account, loop the groups aggregate from a directory or an account:

	import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.directory.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore(); 

	 String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	 Directory directory = client.getDataStore().getResource(href, Directory.class); 

	 GroupList groups = directory.getGroups();
	
	 for (Group grp : groups) {
	 System.out.println("Group " + grp.getName());
	 }

### Retrieve a Group 

To retrieve a specific group you need the `href` which can be loaded as an object instance by retrieving it from the server, using the datastore:

	import com.stormpath.sdk.ds.*;
	import com.stormpath.sdk.group.*;
	...
	...
	DataStore dataStore = client.getDataStore();
	String href = "https://api.stormpath.com/v1/groups/GROUP_UID_HERE";
	Group group = dataStore.getResource(href, Group.class);

### Create Groups 

You can only create groups for cloud directories.
To create directory groups, you must:

1. Get the datastore instance from a client instance.
2. Get a directory instance from the datastore with the directory href.
3. Instantiate a group with the datastore.
4. Set the group properties.
5. Create the group from the directory instance.

**Code:**

	import com.stormpath.sdk.directory.*;
	 import com.stormpath.sdk.group.*;
	 ...
	 ...
	 String href = "https://api.stormpath.com/v1/directories/DIR_UID_HERE";
	 Directory directory = client.getDataStore().getResource(href, Directory.class);

	 Group group = client.getDataStore().instantiate(Group.class);
	 group.setName("New Group");
	 group.setDescription("New Group Description");

	 group = directory.createGroup(group);

### Edit Group Details 

To edit groups, use the `_setters_` of an existing group instance to set the values and call the save method:

	import com.stormpath.sdk.account.*;
	import com.stormpath.sdk.resource.Status;
	...
	...
	group.setStatus(Status.DISABLED);  // the Status enum class provides the valid status'  constants
	group.setName("New Group Name");
	group.setDescription("New Group Description");

	group.save();

### Enable a Group 

If the group is contained within an *enabled directory where the directory is defined as a login source*, then enabling or re-enabling the group allows all accounts contained within the group (membership list) to log in to any applications for which the directory is defined as a login source.

If the group is contained within a *disabled directory where the directory is defined as a login source*, the group status is irrelevant and the group members are not be able to log in to any applications for which the directory is defined as a login source.

If the group is defined as a login source, then enabling or re-enabling the group allows accounts contained within the group (membership list) to log in to any applications for which the group is defined as a login source.

To enable a group, you must:

1. Get the datastore instance from a client instance.
2. Get the group instance from the datastore with the group href.
3. Set the group instance status to enabled.
4. Call the save method on the group instance.

**Code:**

	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.group.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore();

	 String href = "https://api.stormpath.com/v1/groups/GROUP_UID_HERE";
	 Group group = dataStore.getResource(href, Group.class);

	 group.setStatus(Status.ENABLED);  // the Status enum class provides the valid status' constants

	 group.save();

### Disable a Group 

If a group is explicitly set as an application login source, then disabling that group prevents any of its user accounts from logging into that application but retains the group data and memberships. You would typically disable a group if you must shut off a group of user accounts quickly and easily.

To disable a group, you must:

1. Get the datastore instance from a client instance.
2. Get the group instance from the datastore with the group href.
3. Set the group instance status to disabled.
4. Call the save method on the group instance.

**Code:**

	 import com.stormpath.sdk.ds.*;
	 import com.stormpath.sdk.group.*;
	 import com.stormpath.sdk.resource.Status;
	 ...
	 ...
	 DataStore dataStore = client.getDataStore();

	 String href = "https://api.stormpath.com/v1/groups/GROUP_UID_HERE";
	 Group group = dataStore.getResource(href, Group.class);

	 group.setStatus(Status.DISABLED);  // the Status enum class provides the valid status' constants

	 group.save();

### Delete a Group 

Deleting a cloud directory group erases the group and all its membership relationships. User accounts that are members of the group will not be deleted.

We recommend that you disable an group rather than delete it, if you believe you might need to retain the user data or application connection.

To delete a cloud directory group, you must use the Stormpath Admin Console.

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. Click the directory name.
4. Click the **Groups** tab.
5. Under the Actions column, click **Delete**.

***

## Workflow Automations

Workflows are common user management operations that are automated for you by Stormpath. Account Registration and Verification workflow configurations manage how accounts are created in your directory. The Password Reset workflow enables you to configure how password reset works and the context of messages. For both workflows, messages can be formatted in plain text or HTML.

Workflows are only available on cloud directories and only configurable using the Stormpath Admin Console.The Stormpath Administrator directory has default workflow automations which cannot be altered.<br>

On the Workflows tab, you can automate [account registration and verification](#account-registration-and-verification) and [password resets]().

<img src="http://www.stormpath.com/sites/default/files/docs/ManageWorkflows.png" alt="Workflow Automation" title="Workflow Automation" width="670" height="250">


### Account Registration and Verification

For the Account Registration and Verification workflow, you must perform the following actions:

* [Configure account registration and verification](#configure-account-reg)
* [Initiate account registration and verification](#initiate-account-reg)
* [Verify the account](#verify-the-account)
{% docs note %}
The ability to modify workflows, depends on your subscription level. If an option is not available (grayed out), click the ? for more information.
{% enddocs %}

<a name="configure-account-reg"></a>
#### Configure Account Registration and Verification

To configure account registration and verification:

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. Click the directory name.
4. Click **Workflows** tab.
5. On the Workflows tab, next to Registration and Verification, click **show**.
	* By default, the Account Registration and Verification workflow automation is disabled. By leaving this workflow off, all accounts created in the directory are enabled, unless otherwise specified, and the user does not receive any registration or verification emails from Stormpath.
	* By only enabling <strong>Enable Registration and Verification Workflow</strong> and not also enabling <strong>Require newly registered accounts to verify their email address</strong>, new accounts are marked as enabled and the users receive a registration success email. <br>

		<img src="http://www.stormpath.com/sites/default/files/docs/RegistrationVerification.png" alt="Account Registration and Verification" title="Account Registration and Verification" width="650" height="430">


	* You configure the Registration Success Message with the following attributes:
	
		Attribute | Description
:----- | :-----
Message Format | The message format for the body of the Account Registration Success email. It can be Plain Text or HTML. Available formats depend on the tenant subscription level.
"From" Name | The value to display in the "From" field of the Account Registration Success message.
"From" Email Address | The email address from which the Account Registration Success message is sent.
Subject | The value for the subject field of the Account Registration Success message.
Body | The value for the body of the message. Variable substitution is supported for the account first name, last name, username, and email, as well as the name of the directory where the account is registered.

	* By also selecting **Require newly registered accounts to verify their email address**:
		* Newly created accounts are given an *unverified* status and a verification email is sent to the user. The verification email contains a token unique to the user account. When the user clicks the link, they are sent to the verification base URL where the token is submitted to Stormpath for verification. If verified, the account status changes to enabled and a verification success email is sent to the user.
		* An Account Verification Message section appears.<br>
		
			<img src="http://www.stormpath.com/sites/default/files/docs/AccountVerificationMessage.png" alt="Account Verification" title="Account Verification" width="700" height="420">

		* You configure the Account Verification Message with the following attributes: <br>

			Attribute | Description
:----- | :-----
Account Base URL | Your application URL which receives the token and completes the workflow. Stormpath offers a default base URL to help during development.
Message Format | The message format for the body of the Account Verification email. It can be Plain Text or HTML. Available formats depend on the tenant subscription level.
From" Name | The value to display in the "From" field of the Account Success message.
"From" Email Address | The email address from which the Account Verification message is sent.
Subject | The value for the subject field of the Account Verification message.
Body | The value for the body of the message. Variable substitution is supported for the account first name, last name, username, and email, as well as the name of the directory where the account is registered and the url (containing the token) that the user must click.
		* A Verification Success Message section appears.<br>

			<img src="http://www.stormpath.com/sites/default/files/docs/VerificationEmailParams.png" alt="Email Verification" title="Email Verification" width="700" height="420">

		* You configure the Verification Success Message with the following attributes: <br>

			Attribute | Description
:----- | :-----
Message Format | The message format for the body of the Account Verification Success email. It can be Plain Text or HTML. Available formats depend on the tenant subscription level.
"From" Name | The value to display in the "From" field of the Account Verification Success message.
"From" Email Address | The email address from which the Account Verification Success message is sent.
Subject | The value for the subject field of the Account Verification Success message.
Body | The value for the body of the message. Variable substitution is supported for the account first name, last name, username, and email, as well as the name of the directory where the account is registered.

		
6. When all the fields are complete, click **Update**.

<a name="initiate-account-reg"></a>
#### Initiate Account Registration and Verification

If the workflow is enabled, an account registration is automatically initiated during an account creation. 

<a name="verify-the-account"></a>
#### Verify the Account

If a directory has the the account verification workflow enabled:

1. A newly created account in the directory has an `UNVERIFIED` status until the email address has been verified.
2. When a new user is registered for the first time, Stormpath sends an email to the user with a secure verification link, which includes a secure verification token.
3. When the user clicks the link in the email, they are sent to the verification URL set up in the verification workflow. 
	* To verify the account email address (which sets the account status to `ENABLED`), the verification token in the account verification email must be obtained from the link account holders receive in the email. 
	* This is achieved by implementing the following logic:

			import com.stormpath.sdk.tenant.*;
			import com.stormpath.sdk.account.*;
			...
			...
			String verificationToken = // obtain it from query parameter, according to the workflow configuration of the link

			Tenant tenant = client.getCurrentTenant();

			// when the account is correctly verified it gets activated and that account is returned in this verification
			Account account = tenant.verifyAccountEmail(verificationToken);

### Password Reset

When you reset an account password using Stormpath, the user receives an email with a link and a secure reset token. The link sends the user to a password reset page where they submit a new password to Stormpath. When the password is successfully reset, the user receives a success email. You can configure, at the directory level, how password reset works, the URL of the reset page, and the content of the email messages.

Messages can be formatted in plain text or HTML.

#### Configure Password Reset

To configure the password reset workflow:

1. Log in to the Stormpath Admin Console.
2. Click the **Directories** tab.
3. Click the directory name.
4. Click **Workflows** tab.
5. On the Workflows tab, next to Password Reset, click **show**.

	<img src="http://www.stormpath.com/sites/default/files/docs/ResetPW1.png" alt="Password Reset" title="Password Reset" width="640" height="430">
6. Complete the values as follows:<br>
		
	Attribute | Description
:----- | :-----
<a id ="BaseURL"></a>Base URL | Your application URL which receives the token and completes the workflow. Stormpath offers a default base URL to help during development.
Expiration Window | The number of hours that the password reset token remains valid from the time it is sent.

7. Under Password Reset Message, complete the values as follows:<br>

	Attribute | Description
:----- | :-----
Message Format | The message format for the body of the Password Reset email. It can be Plain Text or HTML. Available formats depend on the tenant subscription level.
"From" Name | The value to display in the "From" field of the Password Reset message.
"From" Email Address | The email address from which the Password Reset message is sent.
Subject | The value for the subject field of the Password Reset message.
Body | The value for the body of the message. Variable substitution is supported for the account first name, last name, username, and email, as well as the name of the directory where the account is registered, the url the user must click to verify their account, and the number of hours for which the URL is valid.

8. Under Password Reset Success Message, complete the values as follows:<br>

	Attribute | Description
:----- | :-----
Message Format | The message format for the body of the Password Reset Success email. It can be Plain Text or HTML. Available formats depend on the tenant subscription level.
"From" Name | The value to display in the "From" field of the Password Reset Success message.
"From" Email Address | The email address from which the Password Reset Success message is sent.
Subject | The value for the subject field of the Password Reset Success message.
Body | The value for the body of the message. Variable substitution is supported for the account first name, last name, username, and email, as well as the name of the directory where the account is registered.

	<img src="http://www.stormpath.com/sites/default/files/docs/ResetPW2.png" alt="Password Reset Message" title="Password Reset Message" width="640" height="418">

9. When all the fields are complete, click **Update**.

#### Initiate Password Reset

To initiate the password reset workflow in your application, you must create a password reset token, which is sent from Stormpath in an email to the user. 
	
This is done from the application as follows:

	import com.stormpath.sdk.application.*;
	...
	...
	String href = "https://api.stormpath.com/v1/applications/APP_UID_HERE";
	Application application = client.getDataStore().getResource(href, Application.class);

	// creating the password reset token and sending the email
	application.sendPasswordResetEmail('username or email');

#### Complete Password Reset

After the password reset token is created and the workflow is initiated, Stormpath sends a reset email to the user. The email contains a web link that includes the [base URL](#BaseURL) and the reset token. 

`https://myAwesomeapp.com/passwordReset?sptoken=TOKEN`

Where `myAwesomeapp.com/passwordReset` is the base URL.

After the user clicks the link, the user is sent to the base URL. The password reset token can then be obtained from the query string.

To complete password reset, collect and submit the user's new password with the reset token to Stormpath.

{% docs note %}
To complete the password reset, you do not need any identifying information from the user. Only the password reset token and the new password are required.
{% enddocs %}

The password is changed as follows:

	import com.stormpath.sdk.application.*;
	import com.stormpath.sdk.account.*;
	...
	...  
	String href = "https://api.stormpath.com/v1/applications/APP_UID_HERE";
	Application application = client.getDataStore().getResource(href, Application.class);

	// getting the Account from the token and changing the password
	Account account = application.verifyPasswordResetToken("PASS_RESET_TOKEN");
	account.setPassword("New Password");
	account.save();

***

## Java Sample Code 

### Spring MVC Sample App 

This <a href="https://github.com/stormpath/stormpath-spring-samples" title="pring MVC Sample">application</a> is a Spring MVC-based Twitter clone that helps demonstrate the core functionality of Stormpath. The sample application shows Stormpath-based login, user registration, email verification, password reset, group assignments, <a href="#RBAC" title="RBAC">RBAC</a> checks, and user profile updates.

####Readme: 

**stormpath-spring-samples**: Stormpath example applications based on the Spring Framework.

**tooter** is a sample application that emulates some of the features of <a href="http://twitter.com" title="Twitter">Twitter</a> with the purpose of showing developers how to use the Stormpath Java SDK.

This project requires Maven and access to the Maven Central repository.

Get started:

	$ git clone https://github.com/stormpath/stormpath-spring-samples.git
	$ cd stormpath-spring-samples
	$ mvn install


Deploy the .war file to your web container/application server and launch/access it according to the container configuration.

A detailed documentation on tooter and how to integrate with the Stormpath Java SDK can be found on the wiki <a href="https://github.com/stormpath/stormpath-spring-samples/wiki/Tooter" title="tooter">https://github.com/stormpath/stormpath-spring-samples/wiki/Tooter</a>.

### Apache Shiro Sample Web App 

This is a <a href="https://github.com/stormpath/stormpath-shiro-web-sample" title="shiro web sample">simple web application</a> secured by the Apache Shiro security framework and Stormpath. The example is based of the standard Apache Shiro web sample application and it shows login and <a href="#RBAC" title="RBAC">RBAC</a> using an authorization cache.

***

## Administering Stormpath

For more information about administering Stormpath using the Admin Console, please refer to the [Admin Console Product Guide](http://stormpath.com/docs/console/product-guide).

***

## Appendix 
### How to get the Java SDK jar files if you are not using a Maven-compatible build tool 

If you are not using a Maven-compatible build tool (such as Maven, Ant+Ivy, Gradle or SBT) to acquire your project dependencies, then you will need to manually download the Stormpath Java SDK jars and its runtime dependencies.

All Stormpath open source SDKs and other Java packages, such as the Shiro Stormpath plugin, are available in Maven central:

[http://search.maven.org/#search%7Cga%7C1%7Ccom.stormpath](http://search.maven.org/#search%7Cga%7C1%7Ccom.stormpath)

You can download any of our jars directly from there.

The problem with a direct download is that you do not automatically get any transitive dependencies required to use the jar. This list can sometimes be long, and it might change enough from release to release such that Stormpath does not manually maintain a list of them separate from our build configuration.

As such, if you are unable to use a Maven-compatible build tool, such as Ant+Ivy, Gradle, or SBT, in your project, you must download the Stormpath jars you require from Maven Central using the [link](http://search.maven.org/#search%7Cga%7C1%7Ccom.stormpath).

* This will get you the jar files you need to compile your project.

However, for transitive dependencies that must be in the classpath at runtime, the fastest thing to do is probably the following:

1. Check out the project from GitHub:

		$ git clone https://github.com/stormpath/stormpath-sdk-java.git

2. Build the project:

		$ mvn clean install

3. Download all runtime dependencies:

		$ mvn dependency:copy-dependencies -DincludeScope=runtime

4. Enter the target/dependency directory of the module you will be using.<br>For example, with the Stormpath Java SDK, the default option is an Apache HTTP-client-based runtime:

		$ cd extensions/httpclient/target/dependency

Here you will find a list of all the runtime dependencies necessary for the module.

Note that Stormpath and Shiro use the SLF4J logging API instead of commons-logging. Commons-logging is however in the runtime dependency list because HTTPClient needs it. Because of this conflict, you will probably want to excludecommons-logging.jar and include jcl-over-slf4j.jar (downloaded from maven central) instead, which is compatible with SLF4J logging and satisfies the needs of HTTPClient.

***

## Glossary of Terms


Attribute | Description
:----- | :----- |
<a id="account"></a>Account | An **account** is a unique identity within a directory. Stormpath does not use the term *user* because it implies a person, while accounts can represent a person, 3rd-party software, or non-human clients. Accounts can be used to log in to applications.
<a id="agent"></a>Agent | An **agent** populates LDAP directories. An agent status reflects communication/error state because the agent is communicating with Stormpath.
<a id="apikey"></a>API Key | An **API key** is a unique ID paired with a secret value. API keys are used by software applications to communicate with Stormpath through the Stormpath REST API.
<a id="application"></a>Application | An **application** is a software application that communicates with Stormpath. It is typically a real world application that you are building, such as a web application, but it can also be infrastructural software, such as a Unix or Web server.
<a id="authentication"></a>Authentication | **Authentication** is the act of proving someone (or something) is actually who they say they are. When an account is authenticated, there is a high degree of certainty that the account identity is legitimate.
<a id="authorization"></a>Authorization | **Authorization**, also known as Access Control, is the process of managing and enforcing access to protected resources, functionality, or behavior.
<a id="directory"></a>Directory | A **directory** is a collection of accounts and groups. Administrators can use different directories to create silos of accounts. For example, customers and employees can be stored in different directories.
<a id="directory-agent"></a>Directory Agent | A **directory agent** is a Stormpath software application installed on your corporate network to securely synchronize an on-premise directory, such as LDAP or Active Directory, into a Stormpath cloud directory.
<a id="directory-mirroring"></a>Directory Mirroring | **Directory mirroring** securely replicates selected data from one (source) directory to another (target or mirrored) directory for authentication and access control. The source directory is the authoritative source for all data. Changes are propagated to the target/mirror directory for convenience and performance benefits.
<a id="group"></a>Group | A **group** is a collection of accounts within a directory. In Stormpath, for anyone familiar with Role-Based Access Control, the term group is used instead of role.
<a id="group-membership"></a>Group Membership | A **group membership** is a two-way mapping between an account and a group.
<a id="account-store"></a>Account Store | A **account store** is a directory or group associated with an application for account authentication. Accounts within account stores associated with an application can login to that application.
<a id="account-store-mapping"></a>Account Store Mapping | An **account store mapping** is a mapping between a group or directory and an application.
<a id="identity-management"></a>Identity Management | **Identity management** is the management, authentication, authorization, and permissions of identities to increase security and productivity, while decreasing cost, downtime, and repetitive tasks.
<a id="role"></a>Role |A **role** is a classification of accounts, such as administrators or employees. In Stormpath, roles are represented as groups.
<a id="rbac"></a>Role-Based Access Control | **Role-Based Access Control** (RBAC) is the act of controlling access to protected resources or behavior based on the groups assigned to a particular account. RBAC is done using Stormpath groups.
<a id="rest-api-def"></a>REST API | **REST API** is a software architectural style enabling data transfer and functionality using common web-based communication protocols. Stormpath provides a REST API for tenants so they can easily integrate Stormpath with their software applications.
<a id="tenant"></a>Tenant | A **tenant** is a private partition within Stormpath containing all data and settings—specifically your applications, directories, groups and accounts. When you sign up for Stormpath, a tenant is created for you. You can add other user accounts (for example, for your co-workers) to your tenant to help you manage your data. For convenience, many companies like to have one tenant where they can easily manage all application, directory, and account information across their organization.*

{% docs note %}
*You must know your tenant when logging in to the Admin Console website. There is a "Forgot Tenant" link on the login page if you do not know what your tenant is.
{% enddocs %}