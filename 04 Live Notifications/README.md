# 🔔 Real-Time Notifications in Android with Firebase Cloud Messaging

## Introduction

Firebase Cloud Messaging, commonly called **FCM**, is a service that allows applications to receive messages from a remote source.

A notification can be delivered while the application is:

* Open in the foreground
* Running in the background
* Removed from the recent-apps screen

For this demonstration, notifications will be sent from the **Firebase Console**. A custom backend is therefore not required yet.

The application will use the following flow:

```text
Firebase Console
        ->
Firebase Cloud Messaging
        ->
Android device receives the message
        ->
FirebaseMessagingService processes the message
        ->
Android displays a notification
        ->
The user taps the notification
        ->
MainActivity opens
```

FCM provides the infrastructure used to send messages, but Android is still responsible for displaying and managing the final notification on the device.

---

# 🧱 Final Project Structure

Your project should contain the following important files:

```text
livenotificationdemo
│
├── app
│   ├── google-services.json
│   │
│   ├── src
│   │   └── main
│   │       ├── java
│   │       │   └── com.example.livenotificationdemo
│   │       │       ├── MainActivity.kt
│   │       │       └── MyFirebaseMessagingService.kt
│   │       │
│   │       ├── res
│   │       │   ├── drawable
│   │       │   │   └── ic_notification.xml
│   │       │   └── values
│   │       │       └── strings.xml
│   │       │
│   │       └── AndroidManifest.xml
│   │
│   └── build.gradle.kts
│
├── gradle
│   └── libs.versions.toml
│
└── build.gradle.kts
```

---

# 🆕 1. Create the Android Project

In Android Studio, create a new project:

```text
New Project
-> Empty Activity
```

Use the following settings:

```text
Name: LiveNotificationDemo

Package name:
com.example.livenotificationdemo

Language:
Kotlin

User interface:
Jetpack Compose

Minimum SDK:
API 24 or later
```

The package name is important because it will later be used to connect the Android application to the correct Firebase application.

---

# 🔥 2. Create a Firebase Project

Open Firebase Console and create a new project.

Suggested project name:

```text
Live Notification Demo
```

Google Analytics is optional for the introductory demonstration.

After the Firebase project has been created:

1. Open the project overview.
2. Select **Add app**.
3. Select the Android icon.
4. Enter the Android package name:

```text
com.example.livenotificationdemo
```

5. Select **Register app**.

The Firebase package name must match the Android application's `applicationId` exactly.

Open:

```text
app/build.gradle.kts
```

Confirm that it contains:

```kotlin
defaultConfig {
    applicationId = "com.example.livenotificationdemo"
}
```

Firebase uses the package name to identify which Android application belongs to the Firebase project. A mismatched package name prevents the application from loading the correct Firebase configuration.

---

# 📄 3. Download `google-services.json`

After registering the Android app, Firebase provides a file named:

```text
google-services.json
```

Download this file and place it directly inside the app module:

```text
livenotificationdemo
└── app
    ├── google-services.json
    ├── build.gradle.kts
    └── src
```

The correct location is:

```text
app/google-services.json
```

Do not place it inside:

```text
app/src/main
```

Do not place it in the project root.

Ensure the filename remains exactly:

```text
google-services.json
```

It should not be named:

```text
google-services (1).json
google-services.json.txt
```

The Google Services Gradle plugin reads this file and generates the Firebase configuration required by the Android application.

---

# 🧩 4. Add the Google Services Gradle Plugin

The Google Services plugin processes `google-services.json`.

It identifies the matching Android application and generates resources used to initialise Firebase.

## Project-level Gradle file

Open the project-level file:

```text
build.gradle.kts
```

Add the Google Services plugin inside `plugins {}`:

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false

    /*
     * Declares the Google Services plugin for the project.
     *
     * apply false means the plugin is available to the project,
     * but it will only be activated inside the app module.
     */
    id("com.google.gms.google-services") version "4.5.0" apply false
}
```

## Why is this plugin required?

The plugin:

* Reads `google-services.json`
* Matches the Firebase application using the package name
* Generates Firebase configuration resources
* Makes the configuration available when the application starts

Without this plugin, simply placing `google-services.json` in the project is not enough.

---

# 📦 5. Apply the Plugin and Add Firebase Dependencies

Open the app-level file:

```text
app/build.gradle.kts
```

## Apply the Google Services plugin

Add the plugin inside the existing `plugins {}` section:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)

    /*
     * Activates Google Services processing for this app module.
     */
    id("com.google.gms.google-services")
}
```

## Add Firebase dependencies

Inside `dependencies {}`, add:

```kotlin
dependencies {

    /*
     * The Firebase Bill of Materials, or BoM, controls the versions
     * of Firebase libraries in this project.
     *
     * This helps ensure that Firebase libraries use compatible versions.
     */
    implementation(
        platform("com.google.firebase:firebase-bom:34.16.0")
    )

    /*
     * Adds Firebase Cloud Messaging support.
     *
     * This dependency provides:
     *
     * - FirebaseMessaging
     * - FirebaseMessagingService
     * - RemoteMessage
     * - FCM registration token generation
     * - Remote message delivery
     */
    implementation(
        "com.google.firebase:firebase-messaging"
    )
}
```

Do not add a version directly to `firebase-messaging` when using the Firebase BoM.

Correct:

```kotlin
implementation(
    platform("com.google.firebase:firebase-bom:34.16.0")
)

implementation(
    "com.google.firebase:firebase-messaging"
)
```

Avoid:

```kotlin
implementation(
    "com.google.firebase:firebase-messaging:some-version"
)
```

The Firebase BoM keeps Firebase library versions compatible. The main `firebase-messaging` module should be used instead of the older separate `firebase-messaging-ktx` module.

After adding the plugin and dependencies, run:

```text
File
-> Sync Project with Gradle Files
```

Wait for the sync to finish before continuing.

---

# ⚠️ 6. Check AndroidX Version Compatibility

If your project uses:

```text
Android Gradle Plugin: 8.12.3
compileSdk: 36
```

ensure that your AndroidX dependencies do not require API 37.

A compatible `libs.versions.toml` setup is:

```toml
[versions]
agp = "8.12.3"
kotlin = "2.0.21"

coreKtx = "1.17.0"
lifecycleRuntimeKtx = "2.10.0"
activityCompose = "1.11.0"

junit = "4.13.2"
junitVersion = "1.3.0"
espressoCore = "3.7.0"
composeBom = "2024.09.00"
```

Do not use the following versions with AGP `8.12.3` and `compileSdk 36`:

```toml
coreKtx = "1.19.0"
lifecycleRuntimeKtx = "2.11.0"
activityCompose = "1.13.0"
```

Those versions require a newer Android build configuration.

---

# 📜 7. Update `AndroidManifest.xml`

Open:

```text
app/src/main/AndroidManifest.xml
```

Replace or update it so that it contains the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!--
        Firebase Cloud Messaging requires internet access.

        The application communicates with Firebase servers to:
        - register the app installation,
        - receive messages,
        - and refresh its FCM token.
    -->
    <uses-permission android:name="android.permission.INTERNET" />

    <!--
        Android 13 and newer require permission before an application
        may display notifications.

        Declaring the permission in the manifest is required, but the
        app must also request it while running.
    -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.livenotificationdemo">

        <!--
            Registers the custom Firebase messaging service.

            Android Studio and Firebase do not automatically add this
            service declaration.

            exported="false" means that other applications cannot
            directly start this service.
        -->
        <service
            android:name=".MyFirebaseMessagingService"
            android:exported="false">

            <!--
                This action tells Firebase that this service is
                responsible for Firebase Cloud Messaging events.
            -->
            <intent-filter>
                <action
                    android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>

        </service>

        <!--
            Defines the default notification channel used by Firebase
            when an incoming notification does not specify a channel.

            The value points to a string resource in strings.xml.
        -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_channel_id"
            android:value="@string/default_notification_channel_id" />

        <!--
            Defines the default small notification icon.

            Android uses this icon when an incoming Firebase message
            does not provide a custom icon.
        -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_icon"
            android:resource="@drawable/ic_notification" />

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category
                    android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>

    </application>

</manifest>
```

## Why must the service be registered manually?

The Firebase dependency provides:

```text
FirebaseMessagingService
```

However, your class:

```text
MyFirebaseMessagingService
```

is a custom class created in your own project.

Firebase cannot automatically know:

* What your service is called
* Which package contains it
* Whether you created a custom service
* Whether it handles token updates
* Whether it handles foreground messages

The manifest entry connects your custom service to Firebase's messaging event.

A custom `FirebaseMessagingService` is required when the application needs to handle foreground messages, data messages, token updates or customised notification behaviour.

---

# 🔐 8. Understand Notification Permission

Android versions before Android 13 do not use the `POST_NOTIFICATIONS` runtime permission.

Starting from Android 13, API level 33, the application must ask the user for permission before displaying normal notifications.

The permission process is:

```text
Permission declared in manifest
        ->
Application requests permission
        ->
Android displays system dialog
        ->
User selects Allow or Don't allow
        ->
Application receives the result
```

The manifest declaration alone does not display the permission dialog.

---

# 🏷️ 9. Add Notification Strings

Open:

```text
app/src/main/res/values/strings.xml
```

Use:

```xml
<resources>

    <string name="app_name">
        Live Notification Demo Group 1
    </string>

    <!--
        Internal ID used to identify the notification channel.

        This value is not normally shown to the user.

        It must match the channel ID used in
        MyFirebaseMessagingService.kt.
    -->
    <string name="default_notification_channel_id">
        live_notifications
    </string>

    <!--
        Name displayed to the user in Android notification settings.
    -->
    <string name="notification_channel_name">
        Live Notifications
    </string>

    <!--
        Description displayed in Android notification settings.
    -->
    <string name="notification_channel_description">
        Notifications received from Firebase Cloud Messaging
    </string>

</resources>
```

## Why use strings.xml?

Using string resources:

* Keeps visible text outside Kotlin files
* Supports future translation
* Reduces duplicated text
* Makes values easier to maintain
* Allows the manifest to reference a shared value

The notification channel ID must stay consistent after the channel is created.

---

# 🖼️ 10. Create the Notification Icon

Android notifications require a small icon.

Create:

```text
app/src/main/res/drawable/ic_notification.xml
```

Use:

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">

    <!--
        Notification small icons should normally use a simple white
        shape on a transparent background.

        Android applies the appropriate colour treatment when the
        notification is displayed.
    -->
    <path
        android:fillColor="#FFFFFFFF"
        android:pathData="M12,22c1.1,0 2,-0.9 2,-2h-4c0,1.1 0.9,2 2,2zM18,16v-5c0,-3.07 -1.63,-5.64 -4.5,-6.32V4c0,-0.83 -0.67,-1.5 -1.5,-1.5S10.5,3.17 10.5,4v0.68C7.64,5.36 6,7.92 6,11v5l-2,2v1h16v-1l-2,-2z" />

</vector>
```

Do not use a detailed, full-colour launcher icon as the notification small icon.

Small notification icons should be simple and recognisable.

---

# 🛰️ 11. Create `MyFirebaseMessagingService.kt`

Create a new Kotlin file in the same package as `MainActivity`.

File:

```text
MyFirebaseMessagingService.kt
```

Use:

```kotlin
package com.example.livenotificationdemo

import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.content.Intent
import android.os.Build
import android.util.Log

import androidx.core.app.NotificationCompat
import androidx.core.app.NotificationManagerCompat

import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage

/*
Why use MyFirebaseMessagingService and MainActivity?

MyFirebaseMessagingService (message delivery focused) -
- getting messages
- getting refreshed FCM tokens
- reading message content
- creating notification channels
- building visible notifications
- handling messages when MainActivity is not visible

MainActivity (ui focused) -
- displaying the compose ui
- asking for notification permission
- showing permission status
- getting the current FCM token
- showing the token for testing
 */

class MyFirebaseMessagingService : FirebaseMessagingService() {

    companion object {

        //use this tag to filter service messages in logcat
        private const val TAG = "FCMService"

        /*
        unique ID for the notification channel

        this must match the value in strings.xml:
        default_notification_channel_id
         */
        private const val CHANNEL_ID = "live_notifications"
    }

    /*
    called when firebase creates or refreshes the FCM registration token

    the token identifies one installation of the app

    in production:
    - send the token to your backend
    - associate the token with the correct user/device

    in this demo:
    - print the token in logcat
    - paste it into firebase console for testing
     */
    override fun onNewToken(token: String) {

        super.onNewToken(token)

        //view the token in logcat for testing
        Log.d(
            TAG,
            "NEW FCM TOKEN: $token"
        )
    }

    /*
    called when the service receives an FCM message

    foreground:
    - this callback runs
    - the app creates the notification

    background notification message:
    - firebase/android may display the notification automatically

    data message:
    - custom data is delivered here when android permits it
     */
    override fun onMessageReceived(
        remoteMessage: RemoteMessage
    ) {

        super.onMessageReceived(remoteMessage)

        Log.d(
            TAG,
            "Message received from: ${remoteMessage.from}"
        )

        /*
        remoteMessage.notification contains the standard firebase
        notification payload

        common values:
        - title
        - body
        - image url
         */
        val notificationTitle =
            remoteMessage.notification?.title

        val notificationBody =
            remoteMessage.notification?.body

        /*
        remoteMessage.data contains custom key-value values

        example:
        title = "New practical"
        message = "Practical 3 has been uploaded"
        screen = "practicals"
         */
        val dataTitle =
            remoteMessage.data["title"]

        val dataMessage =
            remoteMessage.data["message"]

        /*
        use the notification payload first

        if it doesn't exist, use the data payload

        if neither exists, use a default value
         */
        val finalTitle =
            notificationTitle
                ?: dataTitle
                ?: "New Notification"

        val finalMessage =
            notificationBody
                ?: dataMessage
                ?: "You have received a new Firebase message."

        Log.d(
            TAG,
            "Notification title: $finalTitle"
        )

        Log.d(
            TAG,
            "Notification message: $finalMessage"
        )

        /*
        create the notification channel before displaying the notification

        calling this repeatedly is safe because android won't create
        duplicate channels with the same ID
         */
        createNotificationChannel()

        //build and display the actual android notification
        showNotification(
            title = finalTitle,
            message = finalMessage
        )
    }

    /*
    create the notification channel required on android 8.0+

    notification channels allow the user to control:
    - sound
    - vibration
    - importance
    - notification visibility
     */
    private fun createNotificationChannel() {

        /*
        notification channels were introduced in android 8.0

        older android versions don't use channels
         */
        if (
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.O
        ) {

            //visible channel name shown in android settings
            val channelName =
                getString(
                    R.string.notification_channel_name
                )

            //visible channel description shown in android settings
            val channelDescription =
                getString(
                    R.string.notification_channel_description
                )

            /*
            high importance allows a prominent notification

            the final behaviour still depends on:
            - user settings
            - device sound settings
            - do not disturb mode
             */
            val importance =
                NotificationManager.IMPORTANCE_HIGH

            val notificationChannel =
                NotificationChannel(
                    CHANNEL_ID,
                    channelName,
                    importance
                ).apply {

                    description =
                        channelDescription

                    //enable vibration if the user/device permits it
                    enableVibration(true)

                    //enable notification lights on supported devices
                    enableLights(true)
                }

            /*
            get android's system notification manager

            this service manages notification channels and notifications
             */
            val notificationManager =
                getSystemService(
                    NotificationManager::class.java
                )

            /*
            register the notification channel with android

            re-registering an existing channel with the same ID
            doesn't create a duplicate
             */
            notificationManager
                .createNotificationChannel(
                    notificationChannel
                )
        }
    }

    /*
    build and display an android notification

    title = heading received from firebase
    message = body received from firebase
     */
    private fun showNotification(
        title: String,
        message: String
    ) {

        /*
        intent describes what should happen when the notification
        is tapped

        this intent opens MainActivity
         */
        val mainActivityIntent =
            Intent(
                this,
                MainActivity::class.java
            ).apply {

                /*
                reuse an existing MainActivity when possible

                CLEAR_TOP:
                remove activities above an existing MainActivity

                SINGLE_TOP:
                reuse MainActivity if it's already at the top
                 */
                flags =
                    Intent.FLAG_ACTIVITY_CLEAR_TOP or
                        Intent.FLAG_ACTIVITY_SINGLE_TOP

                /*
                pass the notification contents to MainActivity

                a larger application could use these values to:
                - open a specific page
                - show a specific announcement
                - load a record from an API
                 */
                putExtra(
                    "notification_title",
                    title
                )

                putExtra(
                    "notification_message",
                    message
                )
            }

        /*
        PendingIntent allows android to perform the intent later

        the notification system can open MainActivity even if
        the app is not currently visible
         */
        val pendingIntent =
            PendingIntent.getActivity(
                this,

                //request code identifies this PendingIntent
                0,

                mainActivityIntent,

                /*
                UPDATE_CURRENT:
                update the existing PendingIntent with new values

                IMMUTABLE:
                prevent other applications from modifying the intent
                 */
                PendingIntent.FLAG_UPDATE_CURRENT or
                    PendingIntent.FLAG_IMMUTABLE
            )

        /*
        build the visible notification

        NotificationCompat supports notifications consistently
        across different android versions
         */
        val notification =
            NotificationCompat.Builder(
                this,
                CHANNEL_ID
            )

                //every notification requires a small icon
                .setSmallIcon(
                    R.drawable.ic_notification
                )

                //main notification heading
                .setContentTitle(
                    title
                )

                //short message shown when notification is collapsed
                .setContentText(
                    message
                )

                /*
                allow longer notification text to expand instead of
                being cut off
                 */
                .setStyle(
                    NotificationCompat.BigTextStyle()
                        .bigText(message)
                )

                /*
                priority mainly affects android versions before
                notification channels controlled importance
                 */
                .setPriority(
                    NotificationCompat.PRIORITY_HIGH
                )

                //open MainActivity when notification is tapped
                .setContentIntent(
                    pendingIntent
                )

                //remove notification after the user taps it
                .setAutoCancel(true)

                //use the normal device notification defaults
                .setDefaults(
                    NotificationCompat.DEFAULT_ALL
                )

                .build()

        /*
        every posted notification needs an integer ID

        using the current time produces a different ID for each
        notification so new messages don't replace previous messages
         */
        val notificationId =
            System.currentTimeMillis()
                .toInt()

        try {

            /*
            request android to display the notification

            this is the line that actually posts the notification
             */
            NotificationManagerCompat
                .from(this)
                .notify(
                    notificationId,
                    notification
                )

            Log.d(
                TAG,
                "Notification displayed successfully."
            )

        } catch (
            securityException: SecurityException
        ) {

            /*
            this may occur on android 13+ when notification
            permission hasn't been granted
             */
            Log.e(
                TAG,
                "Notification permission has not been granted.",
                securityException
            )
        }
    }
}
```

`onMessageReceived()` has a limited execution window and should be used for short processing tasks. Longer work should be moved to an appropriate background-work solution such as WorkManager.

---

# 📢 12. Understand Notification Channels

Notification channels were introduced in Android 8.0.

A channel groups notifications of the same type.

Examples:

```text
Announcements
Messages
Promotions
Reminders
Security alerts
```

Users can independently control each channel's:

* Sound
* Vibration
* Importance
* Visibility
* Notification dots

Once a channel has been created, the user controls most of its behaviour through Android Settings. Changing the channel's importance in code may not update an existing installed channel.

During development, if you change a channel and do not see the new behaviour:

1. Uninstall the app.
2. Reinstall it.
3. Allow notifications again.

This removes the old channel and allows Android to create it again.

---

# 📱 13. Create `MainActivity.kt`

Replace `MainActivity.kt` with:

```kotlin
package com.talia.livenotificationdemogroup2

import android.Manifest
import android.content.pm.PackageManager
import android.os.Build
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.core.content.ContextCompat
import com.google.firebase.messaging.FirebaseMessaging

/*

MyFirebaseMessagingService:
- getting the fb messages
- getting refreshed fcm tokens
- reading messages
- creating notifications
- building visible notifications
- handling messages when the activity is not visible

MainActivity:
- display the compose interface
- ask for notification permission
- show the notification permission status
- get the FCM token
- show the token FOR TESTING
 */

class MainActivity : ComponentActivity() {

    companion object {
        private const val TAG = "NotificationDemo"
    }

    //display current permission Status
    private var permissionStatus by mutableStateOf(
        "Notification permission has not been checked."
    )

    //store the current fcm registration token
    private var fcmToken by mutableStateOf<String?>(null)

    //stores an error message
    private var tokenError by mutableStateOf<String?>(null)

    /*
    ActivityResult API launcher is used to req notif permission on android 13+

    We don't display the dialog to req, android does that

    Callback runs after user chooses either Allow or Don't Allow
     */
    private val notificationPermissionLauncher =
        registerForActivityResult(
            ActivityResultContracts.RequestPermission()
        ){
            isGranted ->
            if(isGranted){
                permissionStatus = "Notification permission granted."

                Log.d(TAG, "The user has given permission.")
            }else{
                permissionStatus = "Notification permission not granted."

                Log.w(TAG, "The user has denied permission.")
            }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()

        /*
         1: CHECK THE CURRENT NOTIFICATION PERMISSION

        Call the function that checks the current notification permission status.

        Why:
        - when the app opens, permissionStatus still contains the default message
        - the app must check whether notifications are already allowed
        - this is especially important if the user granted or denied permission
          during a previous session

        Function to call:
        updateNotificationPermissionStatus()
         */


        /*
         2: REQUEST THE CURRENT FCM TOKEN

        Call the function that retrieves the Firebase Cloud Messaging token.

        Why:
        - every installation of the app receives an FCM registration token
        - this token identifies this specific app installation
        - the token will be copied from Logcat and pasted into Firebase Console
          when sending a test notification
        - Firebase returns the token asynchronously, so the screen may appear
          before the token has been retrieved

        Function to call:
        retrieveFcmToken()
         */


        setContent {

            /*
             3: APPLY THE PROJECT THEME

            Wrap the Compose screen inside the theme that was generated when
            this Android Studio project was created.

            Find the correct theme composable inside:

            ui/theme/Theme.kt

            The name will probably be similar to:

            LiveNotificationDemoGroup2Theme

            Why:
            - applies the application's colours
            - applies Material 3 typography
            - supports light and dark mode
            - keeps the interface visually consistent
             */


            /*
             4: CREATE A SCAFFOLD

            Inside the theme, create a Scaffold.

            The Scaffold must:
            - fill the available screen
            - provide innerPadding to its screen content

            Why:
            - Scaffold provides a standard Material 3 screen structure
            - innerPadding helps prevent the content from overlapping
              the status bar, navigation bar or other system UI
             */


            /*
             5: CREATE THE MAIN COLUMN

            Inside the Scaffold content, create a Column.

            The Column should:
            - fill the available screen
            - apply innerPadding from Scaffold
            - apply additional screen padding
            - arrange components vertically
            - align the content neatly
            - allow vertical scrolling

            Why vertical scrolling is useful:
            - FCM tokens are very long
            - the token may not fit on a small phone screen
            - without scrolling, some buttons or messages may be cut off

            Compose components that may be required:
            - Column
            - Modifier.fillMaxSize()
            - Modifier.padding()
            - Modifier.verticalScroll()
            - rememberScrollState()
            - Arrangement
            - Alignment
             */


            /*
             6: DISPLAY A SCREEN HEADING

            Add a clear heading such as:

            Firebase Cloud Messaging

            Use an appropriate Material 3 typography style.

            Why:
            - communicates the purpose of the application
            - creates a clear visual hierarchy
             */


            /*
             7: DISPLAY A SHORT DESCRIPTION

            Add text explaining that this app receives real-time push
            notifications from Firebase.

            Example meaning:

            This application receives real-time push notifications
            from Firebase.

            Students may choose their own wording.
             */


            /*
             8: DISPLAY THE CURRENT PERMISSION STATUS

            Add a Text component that displays:

            permissionStatus

            Why:
            - permissionStatus is Compose state
            - whenever its value changes, Compose automatically updates the UI
            - users need to know whether notification permission is enabled
             */


            /*
             9: CREATE AN ENABLE NOTIFICATIONS BUTTON

            Add a button labelled something similar to:

            Enable Notifications

            When clicked, the button must call:

            askNotificationPermission()

            Why:
            - Android 13+ requires the user to approve notification permission
            - the application cannot display normal notifications until permission
              has been granted
            - Android 12 and lower do not require this runtime permission, but the
              existing function already handles those versions
             */


            /*
             10: DISPLAY AN FCM TOKEN HEADING

            Add a label similar to:

            FCM Registration Token:

            This heading should appear before the token value.
             */


            /*
             11: DISPLAY THE FCM TOKEN

            Add a Text component that displays:

            fcmToken

            If fcmToken is null, display a message such as:

            Retrieving token...

            Why:
            - Firebase token retrieval is asynchronous
            - the token may not be ready when the Compose screen first appears
            - using a fallback message gives the user feedback while waiting

            Hint:
            Use Kotlin's Elvis operator to provide the fallback value.
             */


            /*
             12: DISPLAY TOKEN ERRORS CONDITIONALLY

            Only display an error message if tokenError is not null.

            The error should:
            - use tokenError as its text
            - use the Material theme's error colour
            - have suitable spacing around it

            Why:
            - an error section should not take up space when no error exists
            - displaying the error helps diagnose Firebase configuration,
              internet or Google Play Services problems

            Hint:
            Use tokenError?.let { ... }
             */


            /*
             13: CREATE A REFRESH TOKEN BUTTON

            Add a button labelled something similar to:

            Refresh FCM Token

            When clicked, call:

            retrieveFcmToken()

            Why:
            - token retrieval may fail temporarily
            - internet access may be unavailable
            - Firebase configuration may have just been corrected
            - students need a simple way to request the token again
             */


            /*
             14: DISPLAY TESTING INSTRUCTIONS

            Add a short message explaining that the token should be copied
            from Logcat and pasted into Firebase Console when sending a test
            notification.

            Important:
            - the token is being displayed for classroom testing
            - production applications should normally send the token securely
              to a backend
            - the token should not be publicly shared
             */


            /*
             15: ADD APPROPRIATE SPACING

            Add Spacer components between:
            - the heading and description
            - the permission status and permission button
            - the permission section and token section
            - the token and error message
            - the token and refresh button
            - the refresh button and final instructions

            Why:
            - improves readability
            - prevents UI components from appearing cramped
            - creates a clear visual hierarchy
             */
        }
    }

    /*
    update permission status = message describing status
     */
    private fun updateNotificationPermissionStatus(){

        //android versions before 13 do not use the POST_NOTIFICATION runtime permission
        if(
            Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU
        ){
            permissionStatus = "Notification permission is automatically available on " +
                    "Android 12 and lower."

            return
        }

        //13+
        val permissionGranted = ContextCompat.checkSelfPermission(
            this,
            Manifest.permission.POST_NOTIFICATIONS

        ) == PackageManager.PERMISSION_GRANTED

        permissionStatus =
            if (permissionGranted)
            {
                "Notification permission granted."
            } else {
                "Notification permission not granted."
            }
    }

    //request permission on android 13+
    private fun askNotificationPermission(){

        //account for android 12 and lower
        if(
            Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU
        ){
            permissionStatus = "Notification permission is automatically available on " +
                    "Android 12 and lower."

            return
        }

        //check if permission is already granted
        val permissionGranted =
            ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.POST_NOTIFICATIONS

            ) == PackageManager.PERMISSION_GRANTED

        if (permissionGranted){

            permissionStatus = "Permission granted."

            return
        }

        //some devices may recommend explaining why permissions are required,
        // esp if the user denies
        if(
            shouldShowRequestPermissionRationale(
                Manifest.permission.POST_NOTIFICATIONS
            )
        ){
            permissionStatus = "Permission to display notifications are needed so the app can show " +
                    "incoming Firebase messages."
        }

        //display androids permission dialog
        notificationPermissionLauncher.launch(
            Manifest.permission.POST_NOTIFICATIONS
        )

    }

    private fun retrieveFcmToken(){

        tokenError = null

        /*
        Firebase is asynchronous

        addOnCompleteListener runs after fb either return the token/ report and error
         */

        FirebaseMessaging
            .getInstance()
            .token
            .addOnCompleteListener { task ->
                if(!task.isSuccessful)
                {
                    tokenError = "Unable to get token."

                    Log.e(TAG,"Fetching the FCM registration token failed.",
                        task.exception
                    )
                    return@addOnCompleteListener
                }

                //token ids the installation of this app
                val token = task.result

                fcmToken = token

                //print the full token into logcat so we can copy it into firebase for TESTING ONLY
                Log.d(TAG,"FCM TOKEN: $token")
            }
    }

    override fun onResume() {
        super.onResume()

        //if the user turns off permissions, check again
        updateNotificationPermissionStatus()

    }
}
```

---

# 🔑 14. Understand the FCM Registration Token

Firebase creates a registration token for each installation of the application.

The token identifies:

```text
This Firebase project
+
This Android application
+
This application installation
+
This device
```

The token is not the user's password.

For this demonstration, it is used to send a test message to one device.

The token may change when:

* The app is reinstalled
* App data is cleared
* Firebase refreshes the token
* The application is restored to another device

A production application should send the latest token to its backend rather than permanently hardcoding it.

---

# 🧪 15. Run the Application

Use either:

* A physical Android device with Google Play services
* An emulator created using a Google Play system image

A plain AOSP emulator may not support FCM correctly.

Run the app.

On Android 13 or newer, press:

```text
Enable Notifications
```

Select:

```text
Allow
```

The screen should update to:

```text
Notification permission granted.
```

---

# 🔎 16. Find the FCM Token

Open Logcat:

```text
View
-> Tool Windows
-> Logcat
```

Search for:

```text
FCM TOKEN
```

You should see:

```text
FCM TOKEN: a-very-long-token-value
```

Copy everything after:

```text
FCM TOKEN:
```

Do not copy:

* Quotation marks
* Extra spaces
* The Logcat date
* The tag
* Only part of the token

---

# 📤 17. Send a Test Message from Firebase Console

In Firebase Console, open:

```text
DevOps & Engagement
-> Messaging
```

Depending on the Firebase interface, select:

```text
Create campaign
-> Firebase Notification messages
```

Enter:

```text
Notification title:
PROG7314 Test

Notification text:
Your Firebase Cloud Messaging notification is working!
```

Select:

```text
Send test message
```

Paste the FCM registration token.

Select:

```text
Test
```

---

# 📴 18. Test Background Notification Behaviour

For the first test:

1. Open the application.
2. Grant notification permission.
3. Press the Home button.
4. Do not force-stop the application.
5. Send the Firebase test message.
6. Open the notification panel.
7. Tap the notification.

Expected result:

```text
The notification appears
        ->
The user taps it
        ->
MainActivity opens
```

When a notification payload arrives while the app is in the background, Firebase and Android can display it automatically.

---

# 🟢 19. Test Foreground Notification Behaviour

For the second test:

1. Keep the application open and visible.
2. Open Logcat.
3. Search for:

```text
FCMService
```

4. Send another test message from Firebase Console.

Expected result:

```text
onMessageReceived() runs
        ->
Message title and body are read
        ->
Notification channel is created
        ->
Notification is built
        ->
NotificationManagerCompat.notify() runs
        ->
Notification appears
```

Foreground messages require the application to handle the message and create a visible notification.

---

# 📨 20. Notification Messages and Data Messages

## Notification message

A notification message normally contains:

```text
Title
Body
Image
```

Firebase Console primarily sends this type of message.

Background behaviour:

```text
Firebase/Android displays the notification
```

Foreground behaviour:

```text
onMessageReceived() runs
-> app displays the notification
```

## Data message

A data message contains custom key-value values.

Example:

```text
title = New practical
message = Practical 4 has been uploaded
screen = practicals
practicalId = 4
```

The app can use this information to:

* Open a particular screen
* Load a specific database record
* Update local data
* Display a custom notification
* Trigger application-specific behaviour

The introductory Firebase Console test focuses mainly on notification messages.

---

# ⚠️ 21. Common Problems

## Notification permission was denied

Open:

```text
Settings
-> Apps
-> Live Notification Demo Group 1
-> Notifications
```

Enable notifications manually.

## No token appears

Check:

* `google-services.json` is inside `app/`
* The package name matches
* The Google Services plugin is applied
* Firebase Messaging dependency was added
* Gradle Sync completed
* The device has internet access
* The emulator includes Google Play

## Background notification works, but foreground does not

Check that `MyFirebaseMessagingService` exists.

Confirm the manifest contains:

```xml
<service
    android:name=".MyFirebaseMessagingService"
    android:exported="false">

    <intent-filter>
        <action
            android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>

</service>
```

## `onMessageReceived()` does not run for background console messages

This may be expected.

When a standard notification payload arrives while the app is in the background, Firebase can display it automatically without calling your foreground processing code in the same way.

Keep the app open when demonstrating `onMessageReceived()`.

## Notification has no sound

The notification channel may already exist with different settings.

Uninstall and reinstall the application, or open the application's notification settings and update the channel manually.

## The service is unresolved in the manifest

Confirm:

* The service file is named `MyFirebaseMessagingService.kt`
* The class is named `MyFirebaseMessagingService`
* It uses the same package as `MainActivity`
* It extends `FirebaseMessagingService`

## The token is rejected in Firebase Console

Confirm that:

* The full token was copied
* The token is current
* The token came from this app installation
* The Firebase Console project matches `google-services.json`
* The application was not reinstalled after copying the token

## Messages stop after force-stop

Force-stopping an app prevents it from receiving normal Firebase messages until the user manually opens the application again.

Removing an app from the recent-apps screen is not the same as force-stopping it.

---

# 🔐 22. Security Considerations

Do not:

* Hardcode FCM tokens permanently
* Publish user tokens publicly
* Send messages directly from an Android app using private server credentials
* Place Firebase service-account keys inside an Android project
* Trust message data without validating it
* Use push notifications as proof that a user is authenticated

In production, notifications should normally be sent from:

* A trusted backend
* Firebase Admin SDK
* FCM HTTP v1 API
* A secure serverless function

The Android application is the message recipient, not the trusted message sender.

---

# 🧭 Final Application Flow

```text
Application starts
        ->
Firebase initialises
        ->
Notification permission is checked
        ->
FCM registration token is requested
        ->
Token appears in Logcat
        ->
Token is pasted into Firebase Console
        ->
Firebase sends test message
        ->
MyFirebaseMessagingService receives foreground message
        ->
Notification channel is created
        ->
Android notification is built
        ->
Notification is displayed
        ->
User taps notification
        ->
MainActivity opens
```
