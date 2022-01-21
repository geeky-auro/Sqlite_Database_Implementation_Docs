# TaskTimerViewModel
The view model class that's going to provide it with data.
Instead of extending the ViewModel class we'll extend an Android view model.<br>
Both classes work the same way.<br>
AndroidViewModel is a subclass of ViewModel,but it also has an application property.<br>
That's going to be useful if we want to access the database,because we'll need an application to get a ContentResolver.<br>

## Step 1.0:Adding Dependencies
    ```
    def lifecycle_version='2.1.0-alpha02'
    // ViewModel and LiveData
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    ```
## Step 1.1:Import the following packages
```
import android.app.Application
import android.database.ContentObserver
import android.database.Cursor
import android.net.Uri
import android.os.Handler
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
```
## Step 2.0:Creating a Class
Create a Class called **TaskTimerViewModel.md**
Extend the class to **AndroidViewModel class**
**Note:** Application context aware ViewModel.
Subclasses must have a constructor which accepts Application as the only parameter.<br>
Also **Application class** is the Base class for maintaining global application state.<br>
Use the primary constructor for getting a reference of Application then use that in  like **AndroidViewModel(application)**<br>
```
class TaskTimerViewModel(application: Application) : AndroidViewModel(application){
<!-- Body goes here! -->
}
```
## Step 2.1:Declaration of Variables
We want it to make a cursor available via a LiveData object.
     * When the cursor changes, observers will be notified of that change.<br>
     * The observer will be MainActivityFragment.<br>
     * When it gets notified of a change,it'll swap the cursor in the RecyclerviewAdapter. <br>
```
    private val databaseCursor = MutableLiveData<Cursor>()
    // Defining our cursor as a LiveData cursor
    val cursor: LiveData<Cursor>
    // Return the database cursor in the get method.
    get() = databaseCursor
```
## Step 2.2:Create a function called loadTask()
Our ViewModel will load the data from the database then update the database cursor LiveData object.<br>
Code for fetching data from the ContentProvider.<br>
```
 private fun loadTask() {
        val projection = arrayOf(
            TasksContract.Columns.ID,
            TasksContract.Columns.TASK_NAME,
            TasksContract.Columns.TASK_DESCRIPTION,
            TasksContract.Columns.TASK_SORT_ORDER
        )

//      <order by>Tasks.sortOrder, Tasks.Name
        val sortOrder ="${TasksContract.Columns.TASK_SORT_ORDER}, ${TasksContract.Columns.TASK_NAME}"
//      Define our cursor using to projection array and the sort order.
        val cursor = getApplication<Application>().contentResolver.query(
            TasksContract.CONTENT_URI, projection, null, null,
            sortOrder
        )
//      setting the database cursor value.
        databaseCursor.postValue(cursor)
    }
```
## Step 2.3 Use a init block
```
init {
        Log.d(TAG, "TaskTimerViewModel: Created.")
//      Register the observer
        getApplication<Application>().contentResolver.registerContentObserver(
        TasksContract.CONTENT_URI,
        true, contentObserver
        )
        loadTask()
    }
```
## Step 2.4:Declare the instance of Content Observer
```
private val contentObserver = object : ContentObserver(Handler()) {
        override fun onChange(selfChange: Boolean, uri: Uri?) {
            Log.d(TAG, "contentObserver.onChange: called. uri is $uri")
            loadTask()
        }
    }
```
Receives call backs for changes to content. Must be implemented by objects which are added to a ContentObservable.<br>
When a change is notified, the onChange function will be called.<br>
We log the fact, then we call loadTasks to reload the data,and update the current cursor LiveData.<br>
**handler** – The handler to run onChange on, or null if none.
## Step 2.5:override the onCleared Function
That function gets called when the ViewModel's no longer used and can be destroyed.
Note we Refister ContentObserver in init block and we unregister the ContentObserver in **onCleared** Function
```
override fun onCleared() {
        getApplication<Application>().contentResolver.unregisterContentObserver(contentObserver)
    }
```
## Step 2.5: Creating a viewModel Instance
Create a ViewModel instance
Describe viewModel. Remember to subscribe to it in onCreate.
```
private val viewModel by lazy {
        // ViewModelProviders.of(activity!!).get(TaskTimerViewModel::class.java)
        ViewModelProvider(this).get(TaskTimerViewModel::class.java)
    }
```
### In the onCreate() Method of MainActivityFragment.kt
To provide the new cursor, when we observe that it has changed.<br>
When the cursor changes, we pass the new one to swapCursor causing the adapter to get the new data.
```
 override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.cursor.observe(this, { cursor -> mAdapter.swapCursoe(cursor)?.close() })
    }
```

## Step2.6.1: Implement the CursorRecyclerViewAdapter.onTaskClickListener Interface
Create an Interface in an CursorRecyclerViewAdapter<br>
Now that we've defined the interface, we can pass in a reference to something that implements that interface,
so that the adapter knows what to call.(Adding in the primary Constructor)
CursorRecyclerViewAdapter <a href="https://github.com/geeky-auro"><img  src="https://img.shields.io/badge/-Under%20Production-%09%23FF0000" alt="Under Construction"  /></a>
```
interface onTaskClickListener{
        fun onEditClick(task:Task)
        fun onDeleteClick(task:Task)
        fun onTaskLongClick(task:Task)
    }
```
## Step 2.6.2:Creating Delete Function in TaskTimerViewModel.kt class
We'll pass in the ID of the task to delete, and call the ContentResolver's delete function to perform the deletion.<br>
Our ContentProvider will take care of deleting an individual task, if its ID is provided in the URI.<br>
The buildURIFromId function that we added to the TasksContract class,will return a URI with the ID appended.<br>
Because we're providing the ID, we don't need to use the last two parameters;<br>
the where clause and selection args.<br>
We just build up the URI from the ID of the task that's been passed to the function,then call the delete function of the Content resolver.<br>
The onDeleteClick function in MainActivityFragment will then call the ViewModel's delete task function,to tell the ViewModel to delete the task.<br>
```
fun deleteTask(taskId:Long) {
//        In Kotlin thread{function}- a thread can be instantiated and executed simply
//        More Info: https://www.baeldung.com/kotlin/threads-coroutines#kotlin-extension
//     Usage of coroutines..in Step-2.6.2.1    
        GlobalScope.launch {
            getApplication<Application>().contentResolver?.delete(
                TasksContract.buildUriFromId(taskId), null, null)
        }
    }
```
## Step 2.6.2.1 : Usage of Couroutines
To run processes in background thread we use Kotlin coroutines.<br>
**Coroutines** allow us to create asynchronous programs in a fluent way.<br>
The Kotlin language gives us basic constructs but can get access to more useful coroutines with the **kotlinx-coroutines-core** library. <br>
To use coroutines in your Android project, add the following dependency to your app's build.gradle file:
```
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
}
```
We can use GlobalScope.launch which uses the same dispatcher to run the process in background thread:
```
GlobalScope.launch {
    // Your code goes here..!
    }
```
**Imports will be automatically added ..!**

## Step 2.6.3:Overriding onDeleteClick() in MainActivityFragment.kt class 
Also Extend the **CursorRecyclerViewAdapter** interface in MainActivityFragment.kt
The **onDeleteClick** function in **MainActivityFragment** will then call the ViewModel's delete task function,to tell the ViewModel to delete the task.
```
override fun onDeleteClick(task: Task) {
      viewModel.deleteTask(task.id)
    }
```
