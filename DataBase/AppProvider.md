# AppProvider
Content Provider for the TaskTimer app.This is the only class that knows about the **[AppDatabase]**

<img src="https://developer.android.com/guide/topics/providers/images/content-provider-overview.png" alt="content Provider" />

## What are Content Providers?
Content Providers are a very important component that serves the purpose of a relational database to store the data of applications.<br>
The role of the content provider in the android system is like a central repository in which data of the applications are stored,<br>
and it facilitates other applications to securely access and modifies that data based on the user requirements.<br>

In order to share the data, content providers have certain permissions that are used to grant or restrict the rights to other applications to interfere with the data.<br>

## Before you start building
**1)** Decide if you need a content provider. You need to build a content provider if you want to provide one or more of the following features:<br>
-You want to offer complex data or files to other applications.<br>
-You want to allow users to copy complex data from your app into other apps.<br>
-You want to provide custom search suggestions using the search framework.<br>
-You want to expose your application data to widgets.<br>
-You want to implement the **AbstractThreadedSyncAdapter**,**CursorAdapter**, or **CursorLoader** classes.<br>



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

For More Information refer : https://datatracker.ietf.org/doc/html/rfc3986#section-3 <br>
You need to define your content provider URI address which will be used to access the content <br>
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
<!--  Here Comes the Body part Refer Step-2 ..!  -->
  }
```
Next you will need to create your own database to keep the content.<br>
Usually, Android uses SQLite database and framework needs to override **onCreate()** method which will use SQLite Open Helper method to create or open the provider's database.<br> When your application is launched, the **onCreate()** handler of each of its Content Providers is called on the main application thread.<br>

Next you will have to **implement/override** Content Provider queries to perform different database specific operations.<br>
**onCreate()** This method is called when the provider is started.We cannot create our database instance as it can only be accessed by getInstance method<br>
**query()** This method receives a request from a client. The result is returned as a Cursor object.<br>
**insert()** This method inserts a new record into the content provider.<br>
**delete()** This method deletes an existing record from the content provider.<br>
**update()** This method updates an existing record from the content provider.<br>
**getType()** This method returns the MIME type of the data at the given URI.<br>
And that's what we are going to implement in Step2.1..
## Step 2.1: **Implementing/Overriding** all the useful Methods
Before Overriding all The Functions we build a URI Matcher funtion to arrange out URI and return the correct URI :-<br>
```
        private fun buildUriMatcher(): UriMatcher {

//      to match the content URI every time user access table under content provider
        val matcher = UriMatcher(UriMatcher.NO_MATCH)
            
//      to access whole table
//      e.g. content://com.aurosaswatraj.tasktimer.provider/Tasks
        matcher.addURI(CONTENT_AUTHORITY, TasksContract.TABLE_NAME, TASKS)

//      to access a particular row of the table
//      e.g. content://com.aurosaswatraj.tasktimer.provider/Tasks/8
        matcher.addURI(CONTENT_AUTHORITY, "${TasksContract.TABLE_NAME}/#", TASKS_ID)

        matcher.addURI(CONTENT_AUTHORITY, TimingsContract.TABLE_NAME, TIMINGS)
        matcher.addURI(CONTENT_AUTHORITY, "${TimingsContract.TABLE_NAME}/#", TIMINGS_ID)

        return matcher
    }
```
Moving towards the next step..! <br>
Then create a variable which saves/creates an instance<br>
> by lazy()-Sometimes we need to construct objects that have a cumbersome initialization process. Also, often we cannot be sure that the object, for which we paid the cost of initialization at the start of our program, will be used in our program at all.
The concept of ‘lazy initialization’ was designed to prevent unnecessary initialization of objects.
            
``` 
            private val uriMatcher by lazy { buildUriMatcher() }
```
<img src="https://img.shields.io/badge/-Override%20Member%20Methods-%23557C55" alt="Override Member" />

```
 override fun onCreate(): Boolean {
        /**We cannot create our database instance as it can only be accessed by getInstance method*/
        return true
    }
            override fun query(
        uri: Uri,
        projection: Array<out String>?,
        selection: String?,
        selectionArgs: Array<out String>?,
        sortOrder: String?
    ): Cursor? {
        Log.d(TAG, "query called with uri $uri")
        val match = uriMatcher.match(uri)
        //Matcher is used to decide what matcher has been passed.>!
        //Use a query builder to build the query that will be executed by the database
         val queryBuilder = SQLiteQueryBuilder()
//        Copy paste the code..!
        when (match) {
            TASKS -> queryBuilder.tables = TasksContract.TABLE_NAME // SELECT ______ FROM Tasks

            TASKS_ID -> {
                queryBuilder.tables = TasksContract.TABLE_NAME // SELECT _____ FROM Tasks
                val taskId = TasksContract.getId(uri) // Long of our taskId. E.g. 1
                queryBuilder.appendWhere("${TasksContract.Columns.ID} = ") // SELECT ______ FROM Tasks WHERE (_id =)      // <-- change method
                queryBuilder.appendWhereEscapeString("$taskId") // SELECT ______ FROM Tasks WHERE (id = 'taskId')
            }
            
            TIMINGS -> queryBuilder.tables = TimingsContract.TABLE_NAME

            TIMINGS_ID -> {
                queryBuilder.tables = TimingsContract.TABLE_NAME
                val timingId = TimingsContract.getId(uri)
                queryBuilder.appendWhere("${TimingsContract.Columns.ID} = ")   // <-- and here
                queryBuilder.appendWhereEscapeString("$timingId")
            }

//            TASKS_DURATIONS -> queryBuilder.tables = DurationsContract.TABLE_NAME
//
//            TASKS_DURATIONS_ID -> {
//                queryBuilder.tables = DurationsContract.TABLE_NAME
//                val durationId = DurationsContract.getId(uri)
//                queryBuilder.appendWhereEscapeString("${DurationsContract.Columns.ID} = ")   // <-- and here
//                queryBuilder.appendWhereEscapeString("$durationId")
//            }

            else -> throw IllegalArgumentException("Unknown URI: $uri")
        }

        val db =
            AppDatabase.getInstance(context!!).readableDatabase // Our database in "TaskTimer.db"
        val cursor =
            queryBuilder.query(db, projection, selection, selectionArgs, null, null, sortOrder)
        Log.d(TAG, "query: rows in returned cursor = ${cursor.count}") // TODO remove this line

        return cursor
    }
            override fun insert(uri: Uri, values: ContentValues?): Uri? {
        Log.d(TAG, "insert called with uri $uri")
//        The code for the matched node (added using addURI), or -1 if there is no matched node.
        val match = uriMatcher.match(uri)
        //        matcher is used to decide what matcher has been passed.>!
        Log.d(TAG, "insert match is $match")
        val recordId: Long
        val returnUri: Uri
        when (match) {
            TASKS -> {
                val db =
                    AppDatabase.getInstance(context!!).writableDatabase // Our database in "TaskTimer.db"
                recordId = db.insert(TasksContract.TABLE_NAME, null, values)
                if (recordId != -1L) {
                    returnUri = TasksContract.buildUriFromId(recordId)
                } else {
                    throw SQLException("Failed to insert,Uri was $uri")
                }

            }
            TIMINGS -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                recordId = db.insert(TimingsContract.TABLE_NAME, null, values)
                if (recordId != -1L) {
                    returnUri = TimingsContract.buildUriFromId(recordId)
                } else {
                    throw SQLException("Failed to insert,Uri was $uri")
                }
            }
            else -> {
                throw java.lang.IllegalArgumentException("Unknown uri:$uri")
            }
        }

        if (recordId > 0) {
            // something was inserted
            Log.d(TAG, "insert: Setting notifyChange with $uri")
            context?.contentResolver?.notifyChange(uri, null)
        }


        Log.d(TAG, "Existing insert,returning $returnUri")
        return returnUri
    }
            override fun delete(uri: Uri, selection: String?, selectionArgs: Array<out String>?): Int {
        Log.d(TAG, "delete called with uri $uri")
        val match = uriMatcher.match(uri)
        //        matcher is used to decide what matcher has been passed.>!
        Log.d(TAG, "delete match is $match")

//        Database to count how many rows were updated?
        var count: Int
//        Database to know the selection criteria
        var selectionCriteria: String
        when (match) {
//            performing delete against the whole table..!
            TASKS -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                count = db.delete(TasksContract.TABLE_NAME, selection, selectionArgs)
            }
//            Performing the update on a single row..!
            TASKS_ID -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                val id = TasksContract.getId(uri)
                selectionCriteria = "${TasksContract.Columns.ID} = $id"

                if (selection != null && selection.isNotEmpty()) {
                    selectionCriteria += " AND ($selection)"
                }

                count = db.delete(TasksContract.TABLE_NAME, selectionCriteria, selectionArgs)
            }

            TIMINGS -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                count = db.delete(TimingsContract.TABLE_NAME, selection, selectionArgs)
            }
//            Performing the update on a single row..!
            TIMINGS_ID -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                val id = TimingsContract.getId(uri)
                selectionCriteria = "${TimingsContract.Columns.ID} = $id"

                if (selection != null && selection.isNotEmpty()) {
                    selectionCriteria += " AND ($selection)"
                }

                count = db.delete(TimingsContract.TABLE_NAME, selectionCriteria, selectionArgs)
            }


            else -> {
                throw IllegalArgumentException("Unknown URI:$uri")
            }

        }
            if (count > 0) {
            // something was deleted
            Log.d(TAG, "delete: Setting notifyChange with $uri")
            context?.contentResolver?.notifyChange(uri, null)
        }

        Log.d(TAG, "Exiting delete, returning $count")
        return count

    }
             override fun update(
        uri: Uri,
        values: ContentValues?,
        selection: String?,
        selectionArgs: Array<out String>?
    ): Int {
        Log.d(TAG, "update called with uri $uri")
        val match = uriMatcher.match(uri)
        //        matcher is used to decide what matcher has been passed.>!
        Log.d(TAG, "update match is $match")

//        Database to count how many rows were updated?
        var count: Int
//        Database to know the selection criteria
        var selectionCriteria: String
        when (match) {
//            performing Update against the whole table..!
            TASKS -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                count = db.update(TasksContract.TABLE_NAME, values, selection, selectionArgs)
            }
//            Performing the update on a single row..!
            TASKS_ID -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                val id = TasksContract.getId(uri)
                selectionCriteria = "${TasksContract.Columns.ID} = $id"

                if (selection != null && selection.isNotEmpty()) {
                    selectionCriteria += " AND ($selection)"
                }

                count =
                    db.update(TasksContract.TABLE_NAME, values, selectionCriteria, selectionArgs)
            }

            TIMINGS -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                count = db.update(TimingsContract.TABLE_NAME, values, selection, selectionArgs)
            }
//            Performing the update on a single row..!
            TIMINGS_ID -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                val id = TimingsContract.getId(uri)
                selectionCriteria = "${TimingsContract.Columns.ID} = $id"

                if (selection != null && selection.isNotEmpty()) {
                    selectionCriteria += " AND ($selection)"
                }

                count =
                    db.update(TimingsContract.TABLE_NAME, values, selectionCriteria, selectionArgs)
            }

            else -> {
                throw IllegalArgumentException("Unknown URI:$uri")
            }

        }

        if (count > 0) {
//            Something was Updated
            Log.d(TAG, "Update :Setting notifyChange with $uri ")
//            call the ContextResolver to notify the change,
            context?.contentResolver?.notifyChange(uri, null)
        }

        Log.d(TAG, "Exiting update function returning count...! :$count")
        return count
    }
            
```
### onCreate Method()
``` 
val match=uriMatcher.match(uri) 
``` 

Matcher is used to decide what matcher has been passed..!

``` 
val queryBuilder = SQLiteQueryBuilder() 
```
 Use a query builder to build the query that will be executed by the database

In the following code segment...
```
when (match) {
            TASKS -> queryBuilder.tables = TasksContract.TABLE_NAME 

            TASKS_ID -> {
                queryBuilder.tables = TasksContract.TABLE_NAME 
                val taskId = TasksContract.getId(uri) 
                queryBuilder.appendWhere("${TasksContract.Columns.ID} = ")      
                queryBuilder.appendWhereEscapeString("$taskId") 
            }

```

``` 
TASKS -> queryBuilder.tables = TasksContract.TABLE_NAME 
```
If the incoming URI was for all of table
> Ex : SELECT ______ FROM Tasks

In the next following Code :-
```
        TASKS_ID -> {
                queryBuilder.tables = TasksContract.TABLE_NAME 
                val taskId = TasksContract.getId(uri) 
                queryBuilder.appendWhere("${TasksContract.Columns.ID} =")    
                queryBuilder.appendWhereEscapeString("$taskId") 
            }
```

The incoming URI was for a single row

``` 
queryBuilder.tables = TasksContract.TABLE_NAME 
```
> SELECT _____ FROM Tasks

```  
val taskId = TasksContract.getId(uri) 
```
> Long of our taskId. E.g. 1

``` 
queryBuilder.appendWhere("${TasksContract.Columns.ID} = ") 
```
> SELECT ______ FROM Tasks WHERE (_id =) 

``` 
queryBuilder.appendWhereEscapeString("$taskId") 
```
> SELECT ______ FROM Tasks WHERE (id = 'taskId')

<img src="https://img.shields.io/badge/-Note-yellow" alt="note"/>

> **queryBuilder.appendWhereEscapeString()** is used to append values not append entire where clause.

And the rest follows the same ... **:)** 

### If the URI is not recognized You should do some error handling here.
We used the following code snippet for that.. :-)
```
else -> throw IllegalArgumentException("Unknown URI: $uri")
```
Finally after the end of querying the entire table, and querying for a single record:
###  call the code to actually do the query
```
val db =AppDatabase.getInstance(context!!).readableDatabase
```
> Our database in "TaskTimer.db"

```
val cursor =queryBuilder.query(db, projection, selection, selectionArgs, null, null, sortOrder)
```

``` 
return cursor 
```

Next function to be overrided is :-
### insert Method()

The **insert()** method adds a new row to the appropriate table, using the values in the ContentValues argument. If a column name is not in the ContentValues argument, you may want to provide a default value for it either in your provider code or in your database schema.

> The code for the matched node (added using addURI), or -1 if there is no matched node.

```
val match = uriMatcher.match(uri)
```

>  matcher is used to decide what matcher has been passed..!

Declare the following variables and initialize them lately
```
val recordId: Long
val returnUri: Uri
```

Perform the match and insert data using the matcher

```
 when (match) {
            TASKS -> {
                val db =
                    AppDatabase.getInstance(context!!).writableDatabase // Our database in "TaskTimer.db"
                recordId = db.insert(TasksContract.TABLE_NAME, null, values)
                if (recordId != -1L) {
                    returnUri = TasksContract.buildUriFromId(recordId)
                } else {
                    throw SQLException("Failed to insert,Uri was $uri")
                }

            }

        }
```
In the following code snippet..

```
val db = AppDatabase.getInstance(context!!).writableDatabase
```

> We referred to Our database in "TaskTimer.db"

```
recordId = db.insert(TasksContract.TABLE_NAME, null, values)
                if (recordId != -1L) {
                    returnUri = TasksContract.buildUriFromId(recordId)
                } else {
                    throw SQLException("Failed to insert,Uri was $uri")
                }
```

> We inserted our values and initialized our returnUri

> insert()-Convenience method for inserting a row into the database 
and returns the row ID of the newly inserted row, or -1 if an error occurred (long)

### Parameters for insert()
-<b><span style="color: green;">table</span></b>	String: the table to insert the row into <br>
-<b><span style="color: green;">nullColumnHack</span></b>	String: optional; may be null. SQL doesn't allow inserting a completely empty row without naming at least one column name. If your provided values is empty, no column names are known and an empty row can't be inserted. If not set to null, the nullColumnHack parameter provides the name of nullable column name to explicitly insert a NULL into in the case where your values is empty. <br>
-<b><span style="color: green;">values</span></b>	ContentValues: this map contains the initial column values for the row. The keys should be the column names and the values the column values 

### Similarly if there's some error 
```
else -> {
        throw java.lang.IllegalArgumentException("Unknown uri:$uri")
        }
```

Finally to end up **insert()** function <br>
Notify registered observers that several rows have been updated. <br>
something was inserted
```
if (recordId > 0) {
            Log.d(TAG, "insert: Setting notifyChange with $uri")
            context?.contentResolver?.notifyChange(uri, null)
        }
```
And
```
return returnUri
```

### delete Method()
**Convenience method for deleting rows in the database.** <br>
Return type: **Int**

Parameters used:- <br>
**table**	String: the table to delete from <br>
**whereClause**	String: the optional WHERE clause to apply when deleting. Passing null will delete all rows.<br>
**whereArgs**	String: You may include ?s in the where clause, which will be replaced by the values from whereArgs. The values will be bound as Strings.<br>

``` 
val match=uriMatcher.match(uri) 
``` 

Matcher is used to decide what matcher has been passed..!

 **Database to count how many rows were updated?** 
 ```
 var count: Int
 ```
 **Database to know the selection criteria** 
 ```
 var selectionCriteria: String
 ```
Following is the code snippet described with inline comments..
```
 when (match) {
//            performing delete against the whole table..!
            TASKS -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                count = db.delete(TasksContract.TABLE_NAME, selection, selectionArgs)
            }
//            Performing the update on a single row..!
            TASKS_ID -> {
                val db = AppDatabase.getInstance(context!!).writableDatabase
                val id = TasksContract.getId(uri)
                selectionCriteria = "${TasksContract.Columns.ID} = $id"

                if (selection != null && selection.isNotEmpty()) {
                    selectionCriteria += " AND ($selection)"
                }

                count = db.delete(TasksContract.TABLE_NAME, selectionCriteria, selectionArgs)
            }
```

### Check if something was deleted then
```
if (count > 0) {
            // something was deleted
            Log.d(TAG, "delete: Setting notifyChange with $uri")
            context?.contentResolver?.notifyChange(uri, null)
        }
```

count has been initialized inside **when block** <br>
### Finally return the count
```
return count
```






















































A summary of the queryBuilder.query constructor
<br>
**queryBuilder** >> "SELECT ______ FROM Tasks WHERE (id = 'taskId')" //taskId is a Long
<br>
**db** >> reference to our database to query from
<br>
**projection** >> "col, col, col, ..." An Array of columns that should be included for each row retrieved
<br>
**selection** >> "WHERE col = value" selection specifies the criteria for selecting rows
<br>
**selectionArgs** >> replaces placeholders in the selection clause
<br>
**groupBy** >> "GROUP BY col, col, ..." combines data for rows with identical values in all included columns
<br>
**having** >> "HAVING condition" eg: 'COUNT(_id) **<** 10'. Adds a condition to the GROUP BY results
<br>
**sortOrder** >> "ORDER BY col, col,..." Orders the rows by a their value in a column/columns
<br>
The output with constructor elements, and output in an example instance below from a match of TASKS_ID
> SELECT   projection    FROM Tasks WHERE (_id = 'taskId') ORDER BY sortOrder <br>
> SELECT Name, SortOrder FROM Tasks WHERE (_id = '1')      ORDER BY SortOrder <br>
       
### Cursor Constructor..!
In the case of our cursor constructor:
```
val cursor = queryBuilder.query(db, projection, selection, selectionArgs, null, null, sortOrder)
```
- db was created in our query(){...} code.
<br>
- (at this point) projection was coded into our cursor's contentResolver in MainActivity and passed in the query() constructor.
<br>
- selection is null and not used (instead added via our when(match) directly into queryBuilder).
<br>
- selectionArgs is null and not used.
<br>
- groupBy was hardcoded as null.
<br>
- having was hardcoded as null.
<br>
- (at this point) sortOrder was coded into our cursor's contentResolver in MainActivity and passed in the query() constructor.
<br>
This creates a new cursor, and is returned to create the cursor in MainActivity, and subsequently used.
