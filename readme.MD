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
`ActivityCompat.requestPermissions(<activityContext>, ....)`  
this is needed because the OS must show a transparent overlay over our screen in order to ask for permissions. This requires the OS to know about the current activity because the overlay is shown on top of it. The application context, on the other hand, is not connected to the app ui but the app itself which might well be in the bg.
