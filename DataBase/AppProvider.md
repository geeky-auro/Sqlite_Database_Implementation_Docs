# AppProvider
Content Provider for the TaskTimer app.This is the only class that knows about the **[AppDatabase]**

## What are content providers?
Content Providers are a very important component that serves the purpose of a relational database to store the data of applications.<br>
The role of the content provider in the android system is like a central repository in which data of the applications are stored,<br>
and it facilitates other applications to securely access and modifies that data based on the user requirements.<br>

In order to share the data, content providers have certain permissions that are used to grant or restrict the rights to other applications to interfere with the data.

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20200914015720/StructureofContentProvider-660x403.png" alt="Content Provider"/>

## Content URI
Content URI(Uniform Resource Identifier) is the key concept of Content providers. To access the data from a content provider, URI is used as a query string.
> Structure of a Content URI: **content://authority/optionalPath/optionalID**

Details of different parts of Content URI:

content:// – Mandatory part of the URI as it represents that the given URI is a Content URI.<br>
1.**authority** – Signifies the name of the content provider like contacts, browser, etc. This part must be unique for every content provider.<br>
2.**optionalPath** – Specifies the type of data provided by the content provider. It is essential as this part helps content providers to support different types of data that are not related to each other like audio and video files.<br>
3.**optionalID** – It is a numeric value that is used when there is a need to access a particular record.<br>
If an ID is mentioned in a URI then it is an id-based URI otherwise a directory-based URI.<br>

## Step-1.0 : Declare the following in the AndroidManifest.xml File
Register up your Content Provider in AndroidManifest.xml!
Construct a ContentProvider instance. Content providers must be declared in the manifest, accessed with ContentResolver, and created automatically by the system, so applications usually do not create ContentProvider instances directly.
```
<provider
            android:authorities="com.aurosaswatraj.tasktimer.provider"
            android:name="com.aurosaswatraj.tasktimer.AppProvider"
            android:exported="false"
            ></provider>
```
Exported stands for can the content provider be accessed by other applications.,.!
> Declares a content provider component. A content provider is a subclass of ContentProvider that supplies structured access to data managed by the application. All content providers in your application must be defined in a <provider> element in the manifest file; otherwise, the system is unaware of them and doesn't run them.
You only declare content providers that are part of your application. Content providers in other applications that you use in your application should not be declared.

The Android system stores references to content providers according to an authority string, part of the provider's content URI. For example, suppose you want to access a content provider that stores information about health care professionals. To do this, you call the method ContentResolver.query(), which among other arguments takes a URI that identifies the provider: 
```
content://com.example.project.healthcareprovider/nurses/rn
```
The content: scheme identifies the URI as a content URI pointing to an Android content provider. The authority **com.example.project.healthcareprovider** identifies the provider itself; the Android system looks up the authority in its list of known providers and their authorities.<br> The substring **nurses/rn** is a path, which the content provider can use to identify subsets of the provider data.
For more Information refer Android Official Documentation for [**provider**](https://developer.android.com/guide/topics/manifest/provider-element) Guide.
## Step-1.1 : Declare Following Variables
The generic URI syntax consists of a hierarchical sequence of 
components referred to as the scheme, authority, path, query, and 
fragment. 
The following are two example URIs and their component parts:

         foo://example.com:8042/over/there?name=ferret#nose
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
          |   _____________________|__
         / \ /                        \
         urn:example:animal:ferret:nose

For More Information refer : https://datatracker.ietf.org/doc/html/rfc3986#section-3
Variable Declaration in AppProvider:
```
  const val CONTENT_AUTHORITY = "com.aurosaswatraj.tasktimer.provider"
  val CONTENT_AUTHORITY_URI: Uri = Uri.parse("content://$CONTENT_AUTHORITY")
```
Also Declare these variables for future use in URI matching 
```
private const val TASKS = 100
private const val TASKS_ID = 101

private const val TIMINGS = 200
private const val TIMINGS_ID = 201
```
## Step 2.0: Declare a Class
Construct a ContentProvider instance. Content providers must be declared in the manifest, accessed with ContentResolver, and created automatically by the system, so applications usually do not create ContentProvider instances directly.
At construction time, the object is uninitialized, and most fields and methods are unavailable. Subclasses should initialize themselves in onCreate, not the constructor.
Content providers are created on the application main thread at application launch time. The constructor must not perform lengthy operations, or application startup will be delayed.<br>
**Declare a class AppProvider and inherit the class ContentProvider()**
```
  class AppProvider : ContentProvider(){
<!--  Here Comes the Body part Refer Step-3 ..!  -->
  }
```
