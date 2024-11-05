# Advanced Android Development with Kotlin

## 1. Android Architecture Components

### 1.1 ViewModel

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData

// ViewModel class to manage UI-related data in a lifecycle-conscious way
class MainViewModel : ViewModel() {
    private val _text = MutableLiveData<String>()
    val text: LiveData<String> get() = _text

    init {
        _text.value = "Hello, World!"
    }

    fun updateText(newText: String) {
        _text.value = newText
    }
}
```

### 1.2 LiveData

```kotlin
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData

// LiveData class example to observe data changes
class TimerViewModel : ViewModel() {
    private val _time = MutableLiveData<Long>()
    val time: LiveData<Long> get() = _time

    fun updateTime(newTime: Long) {
        _time.value = newTime
    }
}

// In your Activity or Fragment
viewModel.time.observe(viewLifecycleOwner, Observer { newTime ->
    // Update the UI
    textViewTime.text = newTime.toString()
})
```

### 1.3 DataBinding

```xml
<!-- activity_main.xml -->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

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
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.viewModel = viewModel
        binding.lifecycleOwner = this
    }
}
```

### 1.4 Navigation Component

```xml
<!-- navigation_graph.xml -->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.app.HomeFragment"
        tools:layout="@layout/fragment_home">
        <action
            android:id="@+id/action_homeFragment_to_detailsFragment"
            app:destination="@id/detailsFragment" />
    </fragment>
    <fragment
        android:id="@+id/detailsFragment"
        android:name="com.example.app.DetailsFragment"
        tools:layout="@layout/fragment_details" />
</navigation>
```

```kotlin
// In your HomeFragment.kt
class HomeFragment : Fragment(R.layout.fragment_home) {
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        _binding = FragmentHomeBinding.bind(view)
        
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

```kotlin
import kotlinx.coroutines.*

fun main() {
    // Launching a coroutine
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    Thread.sleep(2000L) // Blocking main thread to keep JVM alive
}
```

### 2.2 Flow

```kotlin
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // simulating asynchronous work
        emit(i) // Emit next value
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value -> println(value) } // Collect the flow
}
```

## 3. Dependency Injection

### 3.1 Hilt

```kotlin
// AppModule.kt
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideRepository(): MyRepository {
        return MyRepositoryImpl()
    }
}

// MyActivity.kt
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

```kotlin
// AppComponent.kt
@Component(modules = [AppModule::class])
interface AppComponent {
    fun inject(activity: MyActivity)
}

// AppModule.kt
@Module
class AppModule {
    @Provides
    fun provideRepository(): MyRepository {
        return MyRepositoryImpl()
    }
}

// MyActivity.kt
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

```kotlin
@Entity(tableName = "user_table")
data class User(
    @PrimaryKey(autoGenerate = true) val id: Int,
    val name: String,
    val age: Int
)

@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun addUser(user: User)

    @Query("SELECT * FROM user_table ORDER BY id ASC")
    fun readAllData(): LiveData<List<User>>
}

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

### 4.3 WorkManager

```kotlin
class UploadWorker(appContext: Context, workerParams: WorkerParameters) :
    Worker(appContext, workerParams) {

    override fun doWork(): Result {
        // Do the work here--in this case, upload the image.
        uploadImage()

        // Indicate whether the work finished successfully with the Result
        return Result.success()
    }

    private fun uploadImage() {
        // Logic to upload image
    }
}

// Enqueuing a work request
val uploadWorkRequest: WorkRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .build()

WorkManager
    .getInstance(applicationContext)
    .enqueue(uploadWorkRequest)
```

## 5. UI/UX Fundamentals and Advanced View Concepts

### 5.1 Custom Views and ViewExtensions

```kotlin
class CircleView(context: Context, attrs: AttributeSet): View(context, attrs) {
    private val paint = Paint().apply {
        color = Color.RED
        style = Paint.Style.FILL
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val radius = min(width, height) / 2.0f
        canvas.drawCircle(width / 2.0f, height / 2.0f, radius, paint)
    }
}
```

### 5.2 ConstraintLayout

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

```xml
<!-- Add Material Components dependencies in your build.gradle -->
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

```kotlin
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<User>
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val apiService = retrofit.create(ApiService::class.java)

// CoroutineScope to get data using Retrofit API
CoroutineScope(Dispatchers.IO).launch {
    val users = apiService.getUsers()
    withContext(Dispatchers.Main) {
        // Update UI with the list of users
    }
}
```

### 6.2 OkHttp

```kotlin
val client = OkHttpClient()

val request = Request.Builder()
    .url("https://api.example.com/users")
    .build()

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

```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

...

// Storing a value in DataStore
val THEME_KEY = preferencesKey<Boolean>("THEME_KEY")
runBlocking {
    context.dataStore.edit { settings ->
        settings[THEME_KEY] = true
    }
}

// Retrieving a value from DataStore
val themeFlow: Flow<Boolean> = context.dataStore.data
    .map { preferences ->
        preferences[THEME_KEY] ?: false
    }
```

## 8. Testing

### 8.1 Unit Testing

```kotlin
@Test
fun addition_isCorrect() {
    assertEquals(4, 2 + 2)
}

// Testing a ViewModel
@Test
fun testViewModel() {
    val viewModel = MainViewModel()
    assertEquals("Hello, World!", viewModel.text.value)
}
```

### 8.2 UI Testing

```kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
    @Test
    fun useAppContext() {
        val appContext = InstrumentationRegistry.getInstrumentation().targetContext
        assertEquals("com.example.app", appContext.packageName)
    }
}

@Test
fun clickButton_changesText() {
    onView(withId(R.id.my_button)).perform(click())
    onView(withId(R.id.my_text_view)).check(matches(withText("Button clicked")))
}
```

### 8.3 Instrumented Tests

Refer to the example in section 8.2.

## 9. App Performance and Optimization

### 9.1 Profiling Tools

Use Android Studio’s Profiler tool:
- **CPU Profiler**: For CPU usage and method tracing.
- **Memory Profiler**: For memory allocation and garbage collection events.
- **Network Profiler**: For network activity and bandwidth usage.

### 9.2 Memory Management

- **Avoid Memory Leaks**: Ensure no activity or fragment references are leaked.
- **Use Weak References**: When holding onto a reference that might outlive the activity or fragment.

### 9.3 Battery Optimization

- **JobScheduler**: For scheduling tasks that can run when the device is charging and on Wi-Fi.
- **Minimize Wake Locks**: To avoid draining the battery.

## 10. Security

### 10.1 Proguard/R8

```gradle
// In your build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### 10.2 Encrypted SharedPreferences and Databases

```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val sharedPreferences = EncryptedSharedPreferences.create(
    context,
    "secret_shared_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

sharedPreferences.edit().putString("secret_key", "secret_value").apply()
val value = sharedPreferences.getString("secret_key", "default_value")
```

### 10.3 Security Best Practices

- **Use HTTPS**: For all network communications.
- **Validate Input**: To prevent SQL injection and other malicious inputs.
- **Encrypt Sensitive Data**: Both in transit and at rest.

---

By understanding these advanced concepts and practicing with these detailed examples, you'll be well-prepared for your mobile app development contest. Good luck!