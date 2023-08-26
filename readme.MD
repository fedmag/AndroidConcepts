# Manifest

```
It is, in fact, a manifest that exposes capabilities and needs of the app. It is used by the OS to understand what an app can do, the components it consists, the permissions it needs of, etc.
```
# Activity

```
Unit of the application the user interact with. The activity contains context information like if it is currenctly active on the sceen, if it is in bg, serves also as entrypoint for the app (i.e. the activity is the component that gets launched when the application is started).
```

With Jetpack compose you usually have just one activity.

## Lifecycle
At some point the activity is created and at later point also deleted. In between there is an entire collection of state that an action can have:
- `create` state -> initialize all variables in the activity, set views and general initialization (nothing is shown yet on the screen)
- `start` state -> when it becomes visible to the user (but he cannot interact with it yet)
- `resume` state -> This means that the activity is RUNNING. The activity in the foreground, user can interact with it. The RUNNING state is is kept till another activity is launched or another UI piece takes the foreground.
- `pause` state -> the activity is not in the foreground (or there is a dialog being shown), all variables and instances are kept in memory. NOTE: important data should  be saved in this stage as the following 2 stages are not called in special cases.
- `stop` state -> the activity is SURELY not visible to the user, for example if the user navigates to another screen.
- `destroy` state -> the user intentionally closed the activity (presses back button) or the code/os kills the activity to free-up resources. NOTE: Configuration changes (screen rotation, language change, etc.) cause the activity to be destroyed and recreated.
![lifecycle](imgs/activityLifecycle.png)

# Tasks

```
Collection of multiple screens or activity that form a backstack together. The backstack is a stack that contains the views that have been open in the order in which they were open.
Usually each app has its own task and android links them together.
```

## Launch mode
- `standard` -> a new instance of that activity will be pushed on the backstack.
- `single top` -> if there is already an existing instance of that activity the OS will use that one, i.e. does not create a new one.
- `single task` -> creates a new task (a new stack) in which that activity will be launched. The new task is independent of the previous launched task.
- `single instance` -> creates a new task (a new stack) in which that activity will be launched. The difference here is that, within this new task, only one instance of activity can exist, i.e. this stack contains at most one element.

# MVVM

Model - View - ViewModel is the preferred architectural pattern in Android.
![mvvm](imgs/mvvm.png)


# Context

Just an instance of a class. 

```
Bridge between the android app and the OS. Provides the `context` within which the application operates w.r.t. the OS.

It is usually needed whenever the app must communicate with other system compoonents or other apps:
    - get access to resources
    - save files on the FS
    - launch a different activity
    - etc..
```

![context](imgs/context.png)
As shown in the above picture, both `Activity` and `Application` are subclasses of the `Context` class.

## Activity vs Application Context
Each context has a specific lifetime.  
The activity context is active and alive as long as the activity is alive.  
The same is true for the application context, but this also means that the <strong>application context has a lifetime that is longer than the activity one. </strong>

It is always adviced not to store an activity context reference outside of an activity because that could produce <strong> memory leaks </strong>. 

Why would we need the activity context then? There are use-cases where the activity context is needed. For example, when requesting permissions we would use something like:  
```
ActivityCompat.requestPermissions(<activityContext>, ....)
```  
this is needed because the OS must show a transparent overlay over our screen in order to ask for permissions. This requires the OS to know about the current activity because the overlay is shown on top of it. The application context, on the other hand, is not connected to the app ui but the app itself which might well be in the bg.

# Resources
```
All folders under res 
```

- `drawable`: everything visual (png, jpeg, vector graphics, svg). In order to use those resources one must be in an env. with access to a context. Then: 
  ```
  resources.getDrawable(R.drawable.<resource-name>, <theme>)
  ```
- `mipmap`: for the icons of the app
- `values`: colors, text -> collaction of values used to localize the app for example 
- `xml`: ignore

# Intents
```
An envelop to transfer your intention of your app (called intent) to a different android component.  
In short: just a way to communicate with a different android component.
```

There are two type of intents:
- `explicit`: linked to a specified app that has to be specified within the intent. Ex: launch a specific tab in our own app. Or launch youtube app -> android will launch specifically that app as it is explicitly defined in the intent. Ex:
    ```kotlin
    ...
    {
        Button(onClick = {
            Intent(Intent.ACTION_MAIN).also{
                it.package = "com.google.android.youtube"
                try {
                    startActivity(it)
                } catch (e: ActivityNotFoundException) {
                    println("The app was not found!")
                    e.printStackTrace()
                }
            }
        }) {
            Text(text = "Click me")
        }
    }
    ...

    ```
- `implicit`: just specify something you want to do. Android will check which app are able to satisfy that action and the user will be prompted to choose which app to use. Ex:
    ```kotlin
    ...
    {
        Button(onClick = {
            val intent = Intent(Intent.ACTION_SEND).apply{
                type = "text/plain"
                putExtra(Intent.EXTRA_EMAIL, arrayOf("test@test.com"))
                putExtra(Intent.EXTRA_SUBJECT, "my subject")
                putExtra(Intent.EXTRA_TEXT, "Content of the email")
            }
            if 
        }) {
            Text(text = "Click me")
        }
    }
    ...

    ```
    NOTE: since Android 10 or 11, the intents that an app can launch must be specified in a `<queries>` block in the android manifest:
    ```xml
    ...
    <queries> 
        <intent> 
            <action android:name="android.intent.action.SEND">
            <data android:mimeType="text/plain"/>
        </intent>
    </queries>
    ...
    ```

    
To register our app as an app that can resolve some intent we use `intent-filter`. You can specify as many as you want. For example, in order to register our app as candidate to open images:
```xml
<intent-filter> 
    <action android:name="android.intent.action.SEND">
    <category android:name="android.intent.category.DEFAULT"/>
    <data android:mimeType="image/*"/>
</intent-filter>

```

By default the calling app will launc a new instance of our app, even if an instance was already runnig. This does not make much sense. Therefore, it makes sense to specify `android:launchMode="singleTop" ` in the android manifest. In this way, if an instance of the app is running, the OS will use that one. 

Now the calling app packs the image within the intent, but our app must be able to pick that up "parse" the intent for the image and show it. The OS will trigger the function `onNewIntent(intent: Intent?)`, i.e. override this function to do what the app must do with that intent.  Ex:
```kotlin
...
Column(
    modifier = ...
    ...
) {
    myViewModel.uri?.let{ 
        AsyncImage( // This comes from 'coil' library
            model = myViewModel.uri,
            contentDescription = null
        )
    }
}
...


override fun onNewIntent(intent: Intent?) {
    super.onNewIntent(intent)
    val uri = intent?.getParcelableExtra(Intent.EXTRA_STREAM, Uri::class.java)
    myViewModel.updateUri(uri) // this is ofc an example. The Image composable is updated thru the ViewModel
}
```

# Broadcast and Broadcast receiver
```
System wide events that the app consumes/sends. These events are sent by apps (including yours) or Android system. 
To receive these events that app must register itself as broadcast receiver.
```
## Receive Broadcast
Example: once the device is fully booted up, Android will send a broadcast message to all apps that have registered themselves to receive such a message.
There are two ways of registering/unregistering a receiver: dinamically (in code and as long as the app is running) and statically (thru the manifest):  
- `Dinamically`: if we neeed to be notified when the phone is put in airplane mode, we would have a class like:
    ```kotlin
    class AirplaneModeReceiver: BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            if (intent?.action == Intent.ACTION_AIRPLANE_MODE_CHANGED) {
                val isTurnedOn = Settings.Global.getInt(
                    context?.contentResolver,
                    Settings.Global.AIRPLANE_MODE_ON
                ) != 0
                println("Is airplane mode enabled? $isTurnedOn")
            }
        }
    }

    // we also need to REGISTER the receiver in the activity:
    private val airplaneModeReceiver = AirplaneModeReceiver()

    override fun onCreate(...) {
        super.onCreate(...)
        registerReceiver(
            airplaneModeReceiver, 
            IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGE)
        )
    }

    // do not forger to DEREGISTER on destroy
    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(airplaneModeReceiver)
    }
    ```
- `statically`: there are only very few exceptions that allow to register a static receiver as, in general, increases battery consumption.
  ```xml
  ...
  <receiver android:name=".AirplaneModeReceiver">
    <intent-filter>
        <action> ...
    </intent-filter>
  </receiver>
  ...
  ```

  ## Send Broadcast
  Within an activity:
  ```kotlin
  ...
  Button(onClick = {
    sendBroadcast(
        Intent("TEST_ACTION")
    )
  })
  ...

  ```

# Services
```
An android component that does not have a UI and runs in the bg.
```
NOTE: there is no guarantee the Android will not kill the service to free up resources.  
Nowadays mainly foreground services are used, for background tasks, one usually uses the `workManager`.

## Foreground services
```
A service of which the user is aware of. Ex: if a music player app is currently playing music.
```
In this case we are sure the the OS will not kill the service.

Example of creating a foreground service:
```kotlin
class RunningService: Service() {
    override fun onBind(intent: Intent?): IBinder? {
        return null  //this means that nothing can communicate with this service
    }

    // triggered whenever an Intent is sent to this running service
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int { 
        when(intent?.action) {
            Actions.START.toString() -> start()
            Actions.START.toString() -> stopSelf() // stop the service
        }
        return super.onStartCommand(intent, flags, startId)
    }

    private fun start() {
        // we need a persistent notification in order to have a foreground service
        val notification = NotificationCompat.Builder(this, "channelID") // channel must be created
            .setSmallIcon(R.drawable.ic_launcher_foreground)
            .setContentTitle("Run is active")
            .setContentText("Elapsed time: 00:50")
            .build()
        startForeground(1, notification)
    }

    enum class Actions {
        START, STOP
    }
}
```

then also the manifest must be informed (and relevant permission) about such a service:
```xml 
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.POST_NOTIFICATION"/>
<application
    android:name=".RunningApp"  
    ...
    <activity>...</activity>
    <service android:name=".RunningService" android:foregroundServiceType="dataSync"/> 
    // there are many more types
    ...
/>
```
lastly, to activate it we need to launch an intent:
```kotlin

...
    Button(onClick = {
        Intent(applicationContext, RunningService::class.java).also {
            it.action = RunningService.Actions.START.toString()
            startService(it)
        }
    }) {
        Text(text = "Start Run")
    }

    Button(onClick = {
        Intent(applicationContext, RunningService::class.java).also {
            it.action = RunningService.Actions.STOP.toString()
            startService(it)
        }
    }) {
        Text(text = "Stop Run")
    }
...
```

Important note: as long as the service is running all the other activities of the app are working. So keep the service code simple and only use it to communicate with other pieces.

# WorkManager

```
Used to accomplish bg tasks in a reliable manner. 
The user is not necessarily interested in knowing that this processes are running (in contrast to foreground services).
```

Ex: sync data in the bg.

It makes sense to declare the activity as a single top in the maninfest with `android:launchMode="singleTop".`

```kotlin
class MainActivity : ComponentActivity() {

    private lateinit var workManager: WorkManager
    private val photoViewModel = PhotoViewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        workManager = WorkManager.getInstance(applicationContext)


        setContent {
            MinimalWealthTrackerTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    val workerResult = photoViewModel.workId?.let { id ->
                        workManager.getWorkInfoByIdLiveData(id).observeAsState().value
                    }
                    LaunchedEffect(key1 = workerResult?.outputData) {
                        if (workerResult?.outputData != null) {
                            val filePath = workerResult.outputData.getString(PhotoCompressionWorker.KEY_RESULT_PATH)
                            filePath?.let {
                                val bitmap = BitmapFactory.decodeFile(it)
                                photoViewModel.updateCompressedBitMap((bitmap))
                            }
                        }

                    }
                    Column {
                        photoViewModel.uncompressedUri?.let {
                            Text(text = "Uncompressesd")
                            AsyncImage(model = it, contenteDescription = null)
                        }
                        photoViewModel.uncompressedUri?.let {
                            Text(text = "Compressed")
                            Image(model = it.asImageBitmap(), contenteDescription = null)
                        }

                    }
                }
            }
        }
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        val uri = if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            intent?.getParcelableExtra(Intent.EXTRA_STREAM, Uri::class.java)
        } else {
            intent?.getParcelableExtra(Intent.EXTRA_STREAM)
        } ?: return
        photoViewModel.updateUncompressedUri(uri) // update the image to be shown

//        val request = PeriodicWorkRequestBuilder<PhotoCompressionWorker>() // for periodic requests
        val request = OneTimeWorkRequestBuilder<PhotoCompressionWorker>()
            .setInputData(
                workDataOf(
                    PhotoCompressionWorker.KEY_CONTENT_URI to uri,
                    PhotoCompressionWorker.KEY_COMPRESSION_THRESHOLD to 1024 * 20L
                )
            )
            .setConstraints(Constraints(
                requiresStorageNotLow = true
            ))
            .build()
        photoViewModel.updateWorkId(request.id)
        workManager.enqueue(request)
    }
}

class PhotoCompressionWorker(
    private val appContext: Context,
    private val params: WorkerParameters
): CoroutineWorker(appContext, params) {
    override suspend fun doWork(): Result {
//        setForeground() // only if we want to show something to the user
        return withContext(Dispatchers.IO) { // run this code on an optimized thread for IO operations
            val stringUri = params.inputData.getString(KEY_CONTENT_URI)
            val compressionThresholdInBytes = params.inputData.getLong(
                KEY_COMPRESSION_THRESHOLD, 0L
            )
            val uri = Uri.parse(stringUri)
            val bytes = appContext.contentResolver.openInputStream(uri)?.use {
                // use takes care of closing the stream (like with in PY)
                it.readBytes()
            } ?: return@withContext Result.failure()

            val bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.size)

            var outputBytes: ByteArray
            var quality = 100
            do {
                val outputStream = ByteArrayOutputStream()
                outputStream.use {
                    bitmap.compress(Bitmap.CompressFormat.JPEG, quality, it)
                    outputBytes = outputStream.toByteArray()
                    quality -= (quality * 0.1).roundToInt()
                }
            } while (outputBytes.size > compressionThresholdInBytes && quality > 5)

            val file = File(appContext.cacheDir, "${params.id}.jpg")
            file.writeBytes(outputBytes)

            return@withContext Result.success(workDataOf(
                KEY_RESULT_PATH to file.absoluteFile
            ))
        }
    }

    companion object {
        const val KEY_CONTENT_URI = "KEY_CONTENT_URI"
        const val KEY_COMPRESSION_THRESHOLD = "KEY_COMPRESSION_THRESHOLD"
        const val KEY_RESULT_PATH = "KEY_RESULT_PATH"
    }
}

```

