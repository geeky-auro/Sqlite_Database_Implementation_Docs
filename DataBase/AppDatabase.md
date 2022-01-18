# About
Basic Database class for the application <br>
The only class that should use **AppProvider.kt** Class ..!

## Step-1.0:Import the following Packages
```
import android.content.Context
import android.database.sqlite.SQLiteDatabase
import android.database.sqlite.SQLiteOpenHelper
```


## Step-2.0:Declaration of Variables
Declare the **Database Name** and **Database Version** and set the Access Modifier as **private** 
```
private const val DATABASE_NAME = "TaskTimer.db"
private const val DATABASE_VERSION = 3
```

## Step-2.1:Declaration of Class
1) Create a class called AppDatabase
2) Use Constructor to get Context/Activity
3) Inherit/Extend it to SQLiteOpenHelper and make use of public constructor
### Syntax
> SQLiteOpenHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version)

It Creates a helper object to create, open, and/or manage a database. <br>
**Refer Step 2.1 for variables associated with public constructor)**
```
internal class AppDatabase 
private constructor(context: Context) : SQLiteOpenHelper(context, DATABASE_NAME, null,DATABASE_VERSION) 
```
## Step-2.2: Implement Base class Methods
To avoid errors like **does not implement abstract base class members** implement two methods **onCreate(SQLiteDatabase), onUpgrade(SQLiteDatabase, int, int)**
```
    override fun onCreate(db: SQLiteDatabase) {
//       CREATE TABLE Tasks(_id INTEGER PRIMARY KEY NOT NULL,NAME TEXT NOT NULL,Description TEXT,Sortorder INTEGER);
        val sSQL = """CREATE TABLE ${TasksContract.TABLE_NAME} (
            ${TasksContract.Columns.ID} INTEGER PRIMARY KEY NOT NULL,
            ${TasksContract.Columns.TASK_NAME} TEXT NOT NULL,
            ${TasksContract.Columns.TASK_DESCRIPTION} TEXT,
            ${TasksContract.Columns.TASK_SORT_ORDER} INTEGER);""".replaceIndent(" ")
        db.execSQL(sSQL)
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        when (oldVersion) {
            1 -> {
//            Upgrade logic from Version 1
            }
            else -> {
                throw IllegalStateException("onUpgrade() with unknown version:$newVersion")
            }
        }
    }
```
**onCreate** Method is called when the database is created for the first time.<br>This is where the creation of tables and the initial population of the tables should happen. 
The sSql variable creates/designs up the following Query : **CREATE TABLE Tasks(_id INTEGER PRIMARY KEY NOT NULL,NAME TEXT NOT NULL,Description TEXT,Sortorder INTEGER);**

**onUpgrade** Method is called when the database needs to be upgraded.<br>The implementation should use this method to drop tables, add tables, or do anything else it needs to upgrade to the new schema version.

## Step-3.0:Create a Singleton Class
Singleton class only allows a single instance to be create.<br>
To make a class singleton mark the constructor as private

> Use this class as a template for singleton classes..!

```
open class SingletonHolder<out T : Any, in A>(creator: (A) -> T) {
    private var creator: ((A) -> T)? = creator
    @Volatile
    private var instance: T? = null

    fun getInstance(arg: A): T {
        val i = instance
        if (i != null) {
            return i
        }

        return synchronized(this) {
            val i2 = instance
            if (i2 != null) {
                i2
            } else {
                val created = creator!!(arg)
                instance = created
                creator = null
                created
            }
        }
    }
}
```
Source : https://bladecoder.medium.com/kotlin-singletons-with-argument-194ef06edd9e
## Step-3.1:Create a Instance in the AppProvider Class
```
companion object : SingletonHolder<AppDatabase, Context>(::AppDatabase)
 ```
### Next Step,is to use a **content provider**!

