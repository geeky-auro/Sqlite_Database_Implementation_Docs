# Contract

## Step-1:Import the following Packages

```
import android.content.ContentUris
import android.net.Uri
import android.provider.BaseColumns
```
## Step-2.1:To Set Table Name

We are going to use internal Access Modifier so that outside apps cannot access the tables
> Internal means that any client inside this module who sees the declaring class sees its internal members.

``` internal const val TABLE_NAME = "Tasks" ```

## Step-2.1:To Declare Table fields

> The URI to ACCESS THE TASK TABLE

``` 
const val CONTENT_TYPE = "vnd.android.cursor.dir/vnd.$CONTENT_AUTHORITY.$TABLE_NAME"
const val CONTENT_ITEM_TYPE = "vnd.android.cursor.item/vnd.$CONTENT_AUTHORITY.$TABLE_NAME"
val CONTENT_URI: Uri = Uri.withAppendedPath(CONTENT_AUTHORITY_URI, TABLE_NAME)
```
The MIME types returned by ContentProvider.getType have two distinct parts:
> type/subType

The type portion indicates the well known type that is returned for a given URI by the ContentProvider, as the query methods can only return Cursors the type should always be:
1. vnd.android.cursor.dir for when you expect the Cursor to contain 0 through infinity items
**Or**
2. vnd.android.cursor.item for when you expect the Cursor to contain 1 item

The subType portion can be either a well known subtype or something unique to your application.
So when using a ContentProvider you can customize the second subType portion of the MIME type, but not the first portion. e.g a valid MIME type for your apps ContentProvider could be:
> vnd.android.cursor.dir/vnd.myexample.whatever

The MIME type returned from a ContentProvider can be used by an Intent to determine which activity to launch to handle the data retrieved from a given URI.

## Step-2.2:To Declare Table fields

```
object Columns {
        const val ID = BaseColumns._ID
        const val TASK_NAME = "Name"
        const val TASK_DESCRIPTION = "Description"
        const val TASK_SORT_ORDER = "SortOrder"
    }
```
 WE can use hard coded values for "_id" but google provides **BaseColumns._ID** ... for that...!
 
 ## Step-3:To create additional two functions
 ```
 
    fun getId(uri: Uri): Long {
        return ContentUris.parseId(uri)
    }

    fun buildUriFromId(id: Long): Uri {
        return ContentUris.withAppendedId(CONTENT_URI, id)
    }
```
