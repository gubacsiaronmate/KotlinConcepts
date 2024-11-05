# Advanced Android Development with Kotlin - Detailed

## 1. Android Architecture Components

### 1.1 ViewModel
The ViewModel class is designed to store and manage UI-related data in a lifecycle-conscious way. It allows data to survive configuration changes such as screen rotations.

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData

// ViewModel class to manage UI-related data in a lifecycle-conscious way
class MainViewModel : ViewModel() {
    // MutableLiveData is a LiveData subclass which is mutable i.e., we can change its value.
    private val _text = MutableLiveData<String>()
    
    // LiveData is an observable data holder class. It's lifecycle-aware, 
    // meaning it respects the lifecycle of other app components like activities, fragments, or services.
    val text: LiveData<String> get() = _text

    // Initializing the LiveData object with a default value
    init {
        _text.value = "Hello, World!"
    }

    // Function to update the value of LiveData
    fun updateText(newText: String) {
        _text.value = newText
    }
}
```

### 1.2 LiveData
LiveData is an observable data holder class that is lifecycle-aware. It ensures that the UI components only update when they are in an active lifecycle state like STARTED or RESUMED.

```kotlin
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

// LiveData class example to observe data changes
class TimerViewModel : ViewModel() {
    // MutableLiveData to hold a Long value representing time
    private val _time = MutableLiveData<Long>()
    val time: LiveData<Long> get() = _time

    // Function to update the time value
    fun updateTime(newTime: Long) {
        _time.value = newTime
    }
}

// In your Activity or Fragment
// Observe the LiveData and update the UI whenever the observed data changes
viewModel.time.observe(viewLifecycleOwner, { newTime ->
    // Update the UI
    textViewTime.text = newTime.toString()
})
```

### 1.3 DataBinding
DataBinding is a library in Android Jetpack that allows you to bind UI components in your layouts to data sources in your app using a declarative format rather than programmatically.

```xml
<!-- activity_main.xml -->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- The <data> element is used to declare variables for binding expressions -->
    <data>
        <variable
            name="viewModel"
            type="com.example.app.MainViewModel" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".MainActivity">

        <!-- Binding the text property of TextView to viewModel's text LiveData -->
        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{viewModel.text}" />
    </LinearLayout>
</layout>
```

```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.databinding.DataBindingUtil
import androidx.lifecycle.ViewModelProvider
import com.example.app.databinding.ActivityMainBinding

// Using data binding to bind layout UI components to ViewModel data
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Initializing the ViewModel using ViewModelProvider
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
        
        // DataBindingUtil.setContentView will bind the activity layout file to the Activity
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // Assigning the ViewModel to a variable that can be used in layout
        binding.viewModel = viewModel
        
        // Setting the lifecycle owner of the binding to this Activitiy
        binding.lifecycleOwner = this
    }
}
```

### 1.4 Navigation Component
The Navigation component helps you implement navigation, from simple button clicks to more complex patterns like app bars and navigation drawers.

```xml
<!-- navigation_graph.xml: Describes all your app's navigation paths -->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/homeFragment">

    <!-- Declare the homeFragment -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.app.HomeFragment"
        tools:layout="@layout/fragment_home">
        <!-- Define the navigation action -->
        <action
            android:id="@+id/action_homeFragment_to_detailsFragment"
            app:destination="@id/detailsFragment" />
    </fragment>
    
    <!-- Declare the detailsFragment -->
    <fragment
        android:id="@+id/detailsFragment"
        android:name="com.example.app.DetailsFragment"
        tools:layout="@layout/fragment_details" />
</navigation>
```

```kotlin
// In your HomeFragment.kt to handle navigation between fragments
class HomeFragment : Fragment(R.layout.fragment_home) {
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        _binding = FragmentHomeBinding.bind(view)
        
        // Set an OnClickListener to navigate to detailsFragment
        binding.buttonNavigate.setOnClickListener {
            findNavController().navigate(R.id.action_homeFragment_to_detailsFragment)
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

## 2. Reactive Programming

### 2.1 Coroutines
A coroutine is a concurrency design pattern that you can use on Android to simplify code that executes asynchronously.

```kotlin
import kotlinx.coroutines.*

fun main() {
    // Launching a coroutine in GlobalScope - long-running application-wide background processing.
    GlobalScope.launch {
        // Delay coroutine for 1 second
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    // Blocking the main thread for 2 seconds to keep the JVM alive
    Thread.sleep(2000L) 
}
```

### 2.2 Flow
Flow is a coroutine builder that helps create a cold stream of data produced asynchronously.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

// Function returning a flow, emitting values from 1 to 3
fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        // Simulating asynchronous work
        delay(100) 
        emit(i) // Emits next value
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value -> println(value) } // Collect the flow
}
```

## 3. Dependency Injection

### 3.1 Hilt
Hilt is a dependency injection library for Android that reduces the boilerplate of doing manual dependency injection in your project.

```kotlin
// AppModule.kt - Provides app-wide dependencies
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideRepository(): MyRepository {
        return MyRepositoryImpl()
    }
}

// MyActivity.kt - An Android activity with injected dependencies
@AndroidEntryPoint
class MyActivity : AppCompatActivity() {
    @Inject lateinit var repository: MyRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Use the injected repository
        repository.doSomething()
    }
}
```

### 3.2 Dagger
Dagger is a fully static, compile-time dependency injection framework for Java, Kotlin and Android.

```kotlin
// AppComponent.kt - Interface that lists all your app-level dependencies.
@Component(modules = [AppModule::class])
interface AppComponent {
    fun inject(activity: MyActivity)
}

// AppModule.kt - Provides app-wide dependencies
@Module
class AppModule {
    @Provides
    fun provideRepository(): MyRepository {
        return MyRepositoryImpl()
    }
}

// MyActivity.kt - An Android activity with injected dependencies
class MyActivity : AppCompatActivity() {
    @Inject
    lateinit var repository: MyRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Initialize Dagger component and inject dependencies
        DaggerAppComponent.create().inject(this)
        
        // Use the injected repository
        repository.doSomething()
    }
}
```

## 4. Jetpack Libraries

### 4.1 Room
Room provides an abstraction layer over SQLite to allow fluent database access while harnessing the full power of SQLite.

```kotlin
// User.kt - The entity represents a table within the database.
@Entity(tableName = "user_table")
data class User(
    @PrimaryKey(autoGenerate = true) val id: Int,
    val name: String,
    val age: Int
)

// UserDao.kt - DAO is used to access the database.
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun addUser(user: User)

    @Query("SELECT * FROM user_table ORDER BY id ASC")
    fun readAllData(): LiveData<List<User>>
}

// UserDatabase.kt - The database class.
@Database(entities = [User::class], version = 1, exportSchema = false)
abstract class UserDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao

    companion object {
        @Volatile
        private var INSTANCE: UserDatabase? = null

        fun getDatabase(context: Context): UserDatabase {
            val tempInstance = INSTANCE
            if (tempInstance != null) {
                return tempInstance
            }
            synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    UserDatabase::class.java,
                    "user_database"
                ).build()
                INSTANCE = instance
                return instance
            }
        }
    }
}
```

### 4.2 Paging

```kotlin
class ItemDataSource(private val apiService: ApiService) : PagingSource<Int, Item>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Item> {
        val position = params.key ?: 1
        return try {
            val response = apiService.getItems(position, params.loadSize)
            LoadResult.Page(
                data = response.items,
                prevKey = if (position == 1) null else position - 1,
                nextKey = if (response.items.isEmpty()) null else position + 1
            )
        } catch (exception: Exception) {
            LoadResult.Error(exception)
        }
    }
}
```

## 5. UI/UX Fundamentals and Advanced View Concepts

### 5.1 Custom Views and ViewExtensions
Custom views allow you to extend or create your own UI components.

```kotlin
class CircleView(context: Context, attrs: AttributeSet): View(context, attrs) {
    private val paint = Paint().apply {
        color = Color.RED
        style = Paint.Style.FILL
    }

    // Override the onDraw method to specify how the view is drawn
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val radius = min(width, height) / 2.0f
        canvas.drawCircle(width / 2.0f, height / 2.0f, radius, paint)
    }
}
```

### 5.2 ConstraintLayout
ConstraintLayout is a flexible layout, and it allows you to create large and complex layouts with a flat view hierarchy.

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Hello, World!"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintWidth_percent="0.8" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### 5.3 MotionLayout
MotionLayout helps manage motion and widget animation in your app.

```xml
<androidx.constraintlayout.motion.widget.MotionLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutDescription="@xml/scene">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:src="@drawable/ic_launcher_foreground"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.motion.widget.MotionLayout>
```

`scene.xml`:
```xml
<MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:motion="http://schemas.android.com/apk/res-auto">

    <Transition
        motion:constraintSetStart="@+id/start"
        motion:constraintSetEnd="@+id/end"
        motion:duration="1000">
        <OnSwipe
            motion:touchAnchorId="@id/imageView"
            motion:touchAnchorSide="top"
            motion:dragDirection="dragDown" />
    </Transition>

    <ConstraintSet android:id="@+id/start">
        <Constraint
            android:id="@id/imageView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            motion:layout_constraintTop_toTopOf="parent" />
    </ConstraintSet>

    <ConstraintSet android:id="@+id/end">
        <Constraint
            android:id="@id/imageView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            motion:layout_constraintTop_toBottomOf="parent" />
    </ConstraintSet>
</MotionScene>
```

### 5.4 Material Design
Material Design is a design language developed by Google, which includes grid-based layouts, responsive animations and transitions, padding, and depth effects like lighting and shadows.

```xml
<!-- dependencies in build.gradle -->
dependencies {
    implementation 'com.google.android.material:material:1.4.0'
}

<!-- Usage in layout -->
<com.google.android.material.button.MaterialButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="BUTTON"
    app:cornerRadius="12dp"
    app:icon="@drawable/ic_android"
    app:iconPadding="8dp" />
```

## 6. Networking

### 6.1 Retrofit
Retrofit is a type-safe HTTP client for Android and Java. This library makes it easy to consume JSON or XML data which is parsed into Plain Old Java Objects (POJOs).

```kotlin
// API service interface
interface ApiService {
    // Function to retrieve user list from the API
    @GET("users")
    suspend fun getUsers(): List<User>
}

// Retrofit instance creation
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/") // Base URL for API
    .addConverterFactory(GsonConverterFactory.create()) // Gson for JSON parsing
    .build()

// Creating a service instance
val apiService = retrofit.create(ApiService::class.java)

// Using a CoroutineScope to get data from Retrofit API
CoroutineScope(Dispatchers.IO).launch {
    val users = apiService.getUsers() // suspend function call
    withContext(Dispatchers.Main) {
        // Update UI with the list of users
    }
}
```

### 6.2 OkHttp
OkHttp is a powerful HTTP client for Java and Android applications.

```kotlin
// Creating an OkHttpClient instance
val client = OkHttpClient()

// Building a request
val request = Request.Builder()
    .url("https://api.example.com/users")
    .build()

// Making an asynchronous call
client.newCall(request).enqueue(object : Callback {
    override fun onFailure(call: Call, e: IOException) {
        // Handle failure
    }

    override fun onResponse(call: Call, response: Response) {
        if (response.isSuccessful) {
            val responseData = response.body?.string()
            // Handle successful response
        }
    }
})
```

### 6.3 Gson/Moshi
Gson and Moshi are popular libraries for JSON parsing in Android.

```kotlin
// Using Gson to parse JSON
val json = """{ "name": "John", "age": 30 }"""
val user = Gson().fromJson(json, User::class.java)

// Using Moshi to parse JSON
val moshi = Moshi.Builder().build()
val jsonAdapter = moshi.adapter(User::class.java)
val user = jsonAdapter.fromJson(json)
```

## 7. Data Storage and Management

### 7.1 SharedPreferences
SharedPreferences is an API that manages key-value pairs, providing a simple way to read and write preferences.

```kotlin
// Storing data in SharedPreferences
val sharedPref = getSharedPreferences("myPreferences", Context.MODE_PRIVATE)
with(sharedPref.edit()) {
    putString("username", "JohnDoe")
    putInt("userAge", 30)
    apply()
}

// Retrieving data from SharedPreferences
val username = sharedPref.getString("username", null)
val userAge = sharedPref.getInt("userAge", 0)
```

### 7.2 Room
Refer to the example in section 4.1.

### 7.3 DataStore
DataStore is a data storage solution that is part of Jetpack and uses Kotlin coroutines and Flow to store data asynchronously.

```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

// Preference Key
val THEME_KEY = preferencesKey<Boolean>("THEME_KEY")

// Storing a value in DataStore
runBlocking {
    // Asynchronous storage
    context.dataStore.edit { settings ->
        settings[THEME_KEY] = true
    }
}

// Retrieving a value from DataStore
val themeFlow: Flow<Boolean> = context.dataStore.data
        .map { preferences ->
            // Reading the stored value
            preferences[THEME_KEY] ?: false
        }

// Collecting the Flow in a CoroutineScope
lifecycleScope.launch {
    themeFlow.collect { isDarkModeEnabled ->
        // Update UI based on the retrieved value
        updateThemeUI(isDarkModeEnabled)
    }
}
```

## 8. WorkManager

WorkManager is the recommended solution for persistent work. Work is persistent in that it will resume even if the user exits the app or restarts the device.

### 8.1 Simple WorkManager Example
```kotlin
import androidx.work.Worker
import androidx.work.WorkerParameters
import android.content.Context
import androidx.work.OneTimeWorkRequest
import androidx.work.WorkManager

class MyWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    override fun doWork(): Result {
        // Do the work here
        return Result.success()
    }
}

// Scheduling the work
val myWorkRequest = OneTimeWorkRequest.Builder(MyWorker::class.java).build()
WorkManager.getInstance(context).enqueue(myWorkRequest)
```

### 8.2 WorkManager Chaining
```kotlin
val firstWorkRequest = OneTimeWorkRequest.Builder(FirstWorker::class.java).build()
val secondWorkRequest = OneTimeWorkRequest.Builder(SecondWorker::class.java).build()

// Chaining the work
WorkManager.getInstance(context)
    .beginWith(firstWorkRequest)
    .then(secondWorkRequest)
    .enqueue()
```

### 8.3 Periodic Work
Use PeriodicWorkRequest to execute tasks periodically.

```kotlin
val myPeriodicWorkRequest = PeriodicWorkRequest.Builder(
    MyWorker::class.java, 
    24, 
    TimeUnit.HOURS
).build()

WorkManager.getInstance(context).enqueue(myPeriodicWorkRequest)
```

## 9. Advanced Kotlin Features

### 9.1 Extension Functions
Extension functions allow you to add functionality to existing classes without modifying their source code.

```kotlin
// Adding an extension function to the String class
fun String.removeFirstAndLastChar(): String = this.substring(1, this.length - 1)

val originalString = "hello"
val modifiedString = originalString.removeFirstAndLastChar() // ello
```

### 9.2 Higher-Order Functions and Lambdas
Kotlin functions can accept other functions as parameters and return functions, which is useful for implementing functional interfaces like Runnable or custom operations.

```kotlin
// Define a higher-order function
fun performOperation(x: Int, y: Int, operation: (Int, Int) -> Int): Int {
    return operation(x, y)
}

// Lambda function to add two integers
val sum = { a: Int, b: Int -> a + b }

// Using the higher-order function
val result = performOperation(10, 20, sum) // 30
```

### 9.3 Coroutines
Refer to the example in section 2.1.

## Conclusion
With this extended lesson, you should have a good grasp of advanced Android development principles and best practices using Kotlin. These fundamental concepts and advanced techniques will empower you to build robust, scalable, and efficient Android applications.