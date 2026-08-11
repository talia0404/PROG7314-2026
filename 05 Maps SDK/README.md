# 🗺️ Google Maps, Current Location and Place Search in Android

## 📚 What You Will Build

You will create a Jetpack Compose application that can:

* Display a Google Map
* Request foreground location permission
* Retrieve the device’s current location
* Move the camera to the current location
* Search for an address, landmark or business
* Place a marker on the searched location
* Move the map camera to the search result
* Display the selected place’s name and address
* Return to the current location using a button

The expected application flow is:

```text
Application opens
        ->
Google Map loads
        ->
User grants location permission
        ->
App retrieves current location
        ->
Map moves to the current location
        ->
User enters a location search
        ->
Places SDK returns a matching place
        ->
Map moves to the searched location
        ->
A marker displays the result
```

---

# 🧱 1. Create the Android Studio Project

Create a new Android Studio project:

```text
New Project
-> Empty Activity
```

Use:

```text
Name:
GoogleMapsDemo

Package name:
com.talia.googlemapsdemo

Language:
Kotlin

UI:
Jetpack Compose

Minimum SDK:
API 24
```

Do not use the older **Google Maps Activity** template.

The application will use the newer Jetpack Compose approach, so an Empty Activity is the cleaner starting point.

---

# ☁️ 2. Create a Google Cloud Project

Google Maps Platform services are managed through Google Cloud Console.

Create or select a Google Cloud project.

You must then:

1. Connect a billing account.
2. Enable **Maps SDK for Android**.
3. Enable **Places API (New)**.
4. Create an API key.

## APIs required

Enable:

```text
Maps SDK for Android
Places API (New)
```

## Why do we need two APIs?

The two APIs have different responsibilities.

### Maps SDK for Android

The Maps SDK is responsible for displaying and controlling the map.

It provides:

* Map tiles
* Roads and buildings
* Zooming
* Panning
* Rotation
* Camera movement
* Markers
* Map gestures
* Current-location display

### Places API (New)

The Places API is responsible for finding locations.

It allows the application to search using text such as:

```text
Durban ICC
Gateway Theatre of Shopping
Table Mountain
221B Baker Street
coffee shops in Umhlanga
```

The Places API can return:

* Place name
* Address
* Latitude
* Longitude
* Place ID
* Other place information

Think of the relationship as:

```text
Places API
-> Finds the location

Maps SDK
-> Displays the location
```

---

# 🔑 3. Create and Restrict the API Key

In Google Cloud Console:

```text
APIs & Services
-> Credentials
-> Create credentials
-> API key
```

Copy the generated API key.

## Restrict the API key

Open the API key and set:

```text
Application restriction:
Android apps
```

Add:

* Your application package name
* Your SHA-1 certificate fingerprint

Package example:

```text
com.talia.googlemapsdemo
```

## Generate the SHA-1

Open the Android Studio terminal and run:

```powershell
.\gradlew signingReport
```

Look for the `debug` variant.

Copy the value labelled:

```text
SHA1:
```

Add that SHA-1 fingerprint to your Google Cloud Android application restriction.

## API restrictions

Restrict the key so that it can only access:

```text
Maps SDK for Android
Places API (New)
```

## Why restrict the key?

The API key identifies your application when communicating with Google Maps Platform.

Restrictions reduce the chance of someone copying your API key and using it in another application.

The key should only work when:

* The correct package name is used
* The correct signing certificate is used
* One of the approved APIs is requested

Do not publish unrestricted API keys.

---

# 🔐 4. Add the Secrets Gradle Plugin

Do not paste the API key directly into Kotlin code.

Instead, use the Secrets Gradle Plugin.

This allows the real key to remain inside `local.properties`, which should not be committed to GitHub.

## Project-level `build.gradle.kts`

Open:

```text
build.gradle.kts
```

Add:

```kotlin
/*
 * The Secrets Gradle Plugin reads private values from a local
 * properties file and makes them available to the application.
 */
buildscript {
    dependencies {
        classpath(
            "com.google.android.libraries.mapsplatform.secrets-gradle-plugin:secrets-gradle-plugin:2.0.1"
        )
    }
}

plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
}
```

---

# 📦 5. Configure `app/build.gradle.kts`

Open:

```text
app/build.gradle.kts
```

## Add the Secrets plugin

Inside `plugins {}`:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)

    /*
     * Reads MAPS_API_KEY from local.properties.
     */
    id("com.google.android.libraries.mapsplatform.secrets-gradle-plugin")
}
```

---

## Enable BuildConfig

Inside `android {}`:

```kotlin
android {

    /*
     * Keep the rest of your normal Android configuration here.
     */

    buildFeatures {
        compose = true

        /*
         * Generates the BuildConfig class.
         *
         * The application will later use BuildConfig.MAPS_API_KEY
         * when initialising the Places SDK.
         */
        buildConfig = true
    }
}
```

---

## Add the required dependencies

Inside `dependencies {}`:

```kotlin
dependencies {

    /*
     * Provides the Google Map composables.
     *
     * Important classes include:
     * GoogleMap
     * Marker
     * MarkerState
     * MapProperties
     * CameraPositionState
     */
    implementation(
        "com.google.maps.android:maps-compose:6.12.0"
    )

    /*
     * Provides the Fused Location Provider.
     *
     * This is used to retrieve the user's current device location.
     */
    implementation(
        "com.google.android.gms:play-services-location:21.3.0"
    )

    /*
     * Provides the Places SDK.
     *
     * This is used to search for addresses, businesses,
     * landmarks and other named locations.
     */
    implementation(
        "com.google.android.libraries.places:places:5.1.1"
    )
}
```

Run:

```text
File
-> Sync Project with Gradle Files
```

Do not continue until Gradle Sync completes successfully.

---

# 🗝️ 6. Store the API Key

Open:

```text
local.properties
```

Add:

```properties
MAPS_API_KEY=PASTE_YOUR_API_KEY_HERE
```

Example:

```properties
sdk.dir=C\:\\Users\\Student\\AppData\\Local\\Android\\Sdk
MAPS_API_KEY=AIzaSyExampleKey
```

Do not use quotation marks.

Incorrect:

```properties
MAPS_API_KEY="AIzaSyExampleKey"
```

Correct:

```properties
MAPS_API_KEY=AIzaSyExampleKey
```

---

## Create `local.defaults.properties`

Create this file in the project root:

```text
local.defaults.properties
```

Add:

```properties
MAPS_API_KEY=DEFAULT_API_KEY
```

This provides a safe fallback when another developer clones the project but has not configured their own API key.

---

# ⚙️ 7. Configure the Secrets Plugin

At the bottom of:

```text
app/build.gradle.kts
```

add:

```kotlin
/*
 * Tells the Secrets Gradle Plugin where to find
 * the real API key and the fallback value.
 */
secrets {

    /*
     * Contains the real API key.
     */
    propertiesFileName = "local.properties"

    /*
     * Contains the fallback key.
     */
    defaultPropertiesFileName =
        "local.defaults.properties"

    /*
     * Prevent normal Gradle properties from being treated as secrets.
     */
    ignoreList.add("keyToIgnore")
    ignoreList.add("sdk.*")
}
```

---

# 📜 8. Update `AndroidManifest.xml`

Open:

```text
app/src/main/AndroidManifest.xml
```

Use:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!--
        Required for loading map tiles and performing Places searches.
    -->
    <uses-permission android:name="android.permission.INTERNET" />

    <!--
        Allows approximate foreground location.
    -->
    <uses-permission
        android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <!--
        Allows precise foreground location when the user approves it.
    -->
    <uses-permission
        android:name="android.permission.ACCESS_FINE_LOCATION" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GoogleMapsDemo">

        <!--
            Supplies the API key to Maps SDK for Android.

            MAPS_API_KEY is read from local.properties
            through the Secrets Gradle Plugin.
        -->
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="${MAPS_API_KEY}" />

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

---

# 📍 Why do we request both location permissions?

Android supports two levels of foreground location access.

## `ACCESS_COARSE_LOCATION`

Provides approximate location.

The result may only identify the user's general area.

## `ACCESS_FINE_LOCATION`

Provides more precise location when the user grants it.

Even when an application requests precise location, modern Android versions allow the user to select:

```text
Precise
```

or:

```text
Approximate
```

Your application must therefore be able to work when only approximate permission is granted.

This application does not require background location because location is only needed while the user is actively using the map.

---

# 🏷️ 9. Update `strings.xml`

Open:

```text
app/src/main/res/values/strings.xml
```

Use:

```xml
<resources>

    <string name="app_name">
        Google Maps Demo
    </string>

    <string name="search_hint">
        Search for an address or place
    </string>

    <string name="search_button">
        Search
    </string>

    <string name="current_location_button">
        My Location
    </string>

</resources>
```

---

# 🧠 10. Plan `MainActivity`

Do not begin by placing everything inside `setContent`.

First identify the responsibilities of the activity.

`MainActivity` will be responsible for:

* Managing location permission
* Creating the location client
* Creating the Places client
* Keeping track of the current location
* Keeping track of the searched location
* Performing place searches
* Reporting errors
* Providing information to the Compose UI

The UI should display state.

The activity should perform application logic.

---

# 📥 11. Add the Required Imports

As you work through the sections below, use Android Studio's auto-import functionality.

You will need classes from the following areas:

### Android

* `Manifest`
* `PackageManager`
* `Bundle`
* `Log`

### Activity APIs

* `ComponentActivity`
* `setContent`
* `enableEdgeToEdge`
* `ActivityResultContracts`

### Compose

* Layout components
* Material 3 components
* Compose state
* `LaunchedEffect`
* `Modifier`

### Android permissions

* `ContextCompat`

### Location

* `FusedLocationProviderClient`
* `LocationServices`

### Google Maps

* `LatLng`
* `CameraPosition`
* `CameraUpdateFactory`

### Places

* `Places`
* `Place`
* `PlacesClient`
* `SearchByTextRequest`

### Maps Compose

* `GoogleMap`
* `MapProperties`
* `Marker`
* `MarkerState`
* `rememberCameraPositionState`

---

# 🏗️ 12. Create the Activity Properties

Inside `MainActivity`, create the properties needed by the application.

Do this before `onCreate()`.

---

## Fused Location Provider

Create a `lateinit` property of type:

```text
FusedLocationProviderClient
```

### Why?

This object is responsible for retrieving the device's current location.

It combines information from sources such as:

* GPS
* Wi-Fi
* Mobile networks
* Device sensors

You will initialise it later inside `onCreate()`.

---

## Places Client

Create another `lateinit` property of type:

```text
PlacesClient
```

### Why?

The Places client communicates with the Places API.

You will use it later when the user searches for something such as:

```text
Durban ICC
```

or:

```text
Gateway Theatre of Shopping
```

---

# 🧠 13. Create the Application State

Create Compose state variables inside the activity.

These values will control what the UI displays.

---

## Location permission state

Create a Boolean that records:

```text
Does this application currently have foreground location permission?
```

The initial value should be:

```text
false
```

It should become `true` when either:

* Fine location permission is granted
* Coarse location permission is granted

---

## Current location state

Create a nullable `LatLng`.

This represents the user's device location.

Initial value:

```text
null
```

Why nullable?

Because when the application starts, the location has not been retrieved yet.

---

## Searched location state

Create another nullable `LatLng`.

This will store the coordinates returned from the Places search.

Initial value:

```text
null
```

---

## Searched place name

Create a nullable `String`.

This will contain the place's readable name.

Example:

```text
Gateway Theatre of Shopping
```

---

## Searched place address

Create another nullable `String`.

Example:

```text
1 Palm Boulevard, Umhlanga Ridge, Umhlanga
```

---

## Error message

Create a nullable `String`.

Use this to show errors such as:

```text
Location permission denied.
```

```text
No matching place found.
```

```text
Unable to retrieve current location.
```

When there is no error, the value should be `null`.

---

## Searching state

Create a Boolean named appropriately to indicate:

```text
Is a Places search currently running?
```

Initial value:

```text
false
```

This will later control the loading indicator and Search button.

---

# 🔐 14. Create the Location Permission Launcher

Use:

```text
ActivityResultContracts.RequestMultiplePermissions
```

Why multiple permissions?

Because the application requests:

```text
ACCESS_FINE_LOCATION
ACCESS_COARSE_LOCATION
```

The launcher callback will receive the result for both permissions.

Inside the callback:

1. Check whether fine location was granted.
2. Check whether coarse location was granted.
3. Set your permission-state Boolean to `true` if either one was granted.
4. If permission was granted:

   * Clear any previous location error.
   * Request the current location.
5. If permission was denied:

   * Store a useful error message.
   * Do not crash.
   * Remember that place search should still work without device-location permission.

Important principle:

```text
Location permission is required for My Location.

Location permission is NOT required to search for another place.
```

---

# ▶️ 15. Configure `onCreate()`

Inside `onCreate()` perform the following steps in this order.

---

## Step 1: Call the normal Activity setup

Make sure:

```text
super.onCreate(...)
```

and:

```text
enableEdgeToEdge()
```

are called.

---

## Step 2: Initialise the Fused Location Provider

Use `LocationServices` to create the location client.

Pass the current Activity as the context.

Store the result in the property you created earlier.

### Why initialise it here?

The activity must have a valid Android context before the location service can be created.

---

# 🔑 16. Retrieve the API Key

Read the key generated by the Secrets Gradle Plugin from:

```text
BuildConfig.MAPS_API_KEY
```

Store it in a local variable.

Before continuing, check whether:

* The key is blank
* The key still equals `DEFAULT_API_KEY`

If either condition is true:

* Write an error to Logcat.
* Stop the Activity.

### Why check this?

Otherwise the application may continue and fail later with confusing Maps or Places errors.

Failing early makes configuration problems easier to identify.

---

# 📍 17. Initialise Places SDK

Before creating the Places client, check whether Places has already been initialised.

If it has not:

* Initialise **Places API (New)**.
* Pass the application context.
* Pass the API key.

Important:

Use the New Places API initialisation method, not older legacy setup.

### Why?

The project uses:

```text
SearchByTextRequest
```

which belongs to the newer Places SDK functionality.

---

# 🔎 18. Create the Places Client

After Places is initialised:

* Create a `PlacesClient`.
* Store it in the activity property you created earlier.

You will use this object every time the user performs a place search.

---

# 🔐 19. Check Existing Location Permission

Before showing the interface, call a function that checks whether the app already has location permission.

Why?

A user may have granted permission the previous time the application was used.

You should not ask the user for permission every time the app opens.

---

# 🎨 20. Configure `setContent`

Inside `setContent`:

1. Apply the project's Compose theme.
2. Create a `Scaffold`.
3. Pass the Scaffold's `innerPadding` to your screen.
4. Call a custom composable named something similar to:

```text
MapsScreen
```

Pass the screen all the state it needs.

The screen should receive:

* Whether location permission exists
* Current location
* Searched location
* Searched place name
* Searched place address
* Searching state
* Error message

The screen should also receive event functions for:

* Requesting permission
* Requesting current location
* Searching for a place

### Why pass functions to the composable?

The Compose screen should be responsible for:

```text
What does the UI look like?
```

The Activity should be responsible for:

```text
What happens when an action occurs?
```

This helps separate UI from logic.

---

# 🔐 21. Create `updateLocationPermissionStatus()`

Create a private function that checks the current Android permission state.

The function should:

1. Check `ACCESS_FINE_LOCATION`.
2. Check `ACCESS_COARSE_LOCATION`.
3. Use `ContextCompat.checkSelfPermission()`.
4. Compare the result with `PackageManager.PERMISSION_GRANTED`.
5. Set `hasLocationPermission` to `true` if either permission exists.

If permission already exists:

* Request the current device location.

### Why check both?

The user may grant approximate permission while denying precise permission.

That is still a valid foreground location permission.

---

# 📣 22. Create `requestLocationPermission()`

Create a private function responsible only for starting the permission request.

Use your previously created permission launcher.

Launch an array containing:

```text
ACCESS_FINE_LOCATION
ACCESS_COARSE_LOCATION
```

### Why put this in a function?

The same permission request may be triggered from:

* Initial setup
* The My Location button
* Another part of the app later

Keeping it in one function prevents duplication.

---

# 📍 23. Create `getCurrentLocation()`

Create a private function responsible for retrieving the device's location.

The function should follow this order carefully.

---

## Step 1: Check permission state

If the app does not currently have permission:

* Call the permission-request function.
* Stop the current function.

Do not try to access device location first.

---

## Step 2: Recheck actual Android permission

Even if your Boolean says permission exists, check Android's permission state again.

Why?

The user may revoke location permission from Android Settings while your application is still running.

Check both:

```text
ACCESS_FINE_LOCATION
ACCESS_COARSE_LOCATION
```

If neither exists:

* Update your permission-state Boolean to false.
* Stop the function.

---

## Step 3: Request the location

Use the Fused Location Provider.

For this simple demo, request the last known location.

The location operation is asynchronous.

This means the app does not immediately receive the value.

You must add:

* A success listener
* A failure listener

---

## Step 4: Handle success

Inside the success listener, check whether the returned location is `null`.

If a location exists:

1. Read its latitude.
2. Read its longitude.
3. Create a Google Maps `LatLng`.
4. Save this object into `currentLocation`.
5. Clear any previous error.
6. Log the coordinates in Logcat.

When `currentLocation` changes, Compose will update the UI.

---

## Step 5: Handle a null location

A null result is possible.

This can happen when:

* Device location is switched off
* The emulator has no location
* No location has recently been recorded
* Google Play services restarted

If location is null:

* Do not crash.
* Set a useful error message.

---

## Step 6: Handle failure

Add a failure listener.

If location retrieval fails:

* Store a readable error message.
* Log the technical exception to Logcat.

Users should see a friendly error.

Developers should see the full exception.

---

## Step 7: Handle SecurityException

Wrap the location request appropriately so that a permission-related security error does not crash the application.

If a security error occurs:

* Set `hasLocationPermission` to false.
* Show a permission error.
* Log the exception.

---

# 🔎 24. Create `searchForPlace()`

Create a private function that accepts:

```text
query: String
```

This function will convert text entered by the user into an actual Google Place.

---

## Step 1: Validate the query

Before sending anything to Google:

* Check whether the query is blank.
* If it is blank:

  * Display an error message.
  * Stop the function.

Do not send unnecessary API requests.

---

## Step 2: Start the loading state

Before starting the search:

* Set `isSearching` to `true`.
* Clear any previous error message.

The UI can use this state to:

* Disable the Search button
* Display a loading indicator

---

# 📋 25. Choose the Place Fields

Create a list containing only the fields required by this application.

Request:

```text
Place ID
Display name
Formatted address
Location
```

### Why explicitly request fields?

Places does not automatically return every possible field.

This is intentional.

Requesting only what you need:

* Reduces unnecessary data
* Makes the response easier to work with
* May affect billing
* Makes the app more efficient

For this screen, there is no reason to request information such as:

* Opening hours
* Phone number
* Reviews
* Rating

---

# 🔍 26. Build the Text Search Request

Create a `SearchByTextRequest`.

It needs:

* The user's query
* The requested Place fields

Limit the result count to:

```text
1
```

### Why one result?

This introductory demo only displays the best matching location.

A more advanced application could later display multiple results in a list.

---

# ☁️ 27. Send the Search

Use your `PlacesClient` to perform the Text Search.

Again, the operation is asynchronous.

Add:

* A success listener
* A failure listener

---

# ✅ 28. Handle Search Success

Inside the success listener:

1. Set `isSearching` back to false.
2. Read the returned list of places.
3. Take the first result.
4. Check whether a result exists.

If no result exists:

* Display a message such as:

```text
No matching location was found.
```

Do not attempt to access properties on a missing place.

---

# 📍 29. Read the Search Result

From the returned `Place`, retrieve:

* Location coordinates
* Display name
* Formatted address

Check whether the location coordinates exist.

If coordinates are unavailable:

* Display an error.
* Stop processing.

If coordinates exist:

1. Save them in `searchedLocation`.
2. Save the display name.
3. Save the formatted address.
4. Clear previous errors.
5. Write the result to Logcat.

Once `searchedLocation` changes, the map screen should react automatically.

---

# ❌ 30. Handle Search Failure

Inside the failure listener:

* Set `isSearching` back to false.
* Store a useful error message.
* Log the actual exception.

Possible causes include:

* Invalid API key
* Places API not enabled
* Billing problems
* Network failure
* Incorrect API restrictions

---

# 🎨 31. Create `MapsScreen`

Create a composable below the `MainActivity` class.

The screen should receive all the state values and event handlers from the activity.

Required parameters should include:

```text
Modifier

Location permission Boolean

Current LatLng

Searched LatLng

Searched place name

Searched address

Searching Boolean

Error message

Request permission callback

Current-location callback

Search callback
```

---

# ⌨️ 32. Create Search Text State

Inside `MapsScreen`, create local Compose state for the text entered into the search field.

Use a `String`.

Initial value:

```text
""
```

Use `remember` so that the text remains available across recompositions.

### Why is this state inside the screen?

The search text only matters to the UI until the user presses Search.

The Activity does not need to own every single UI value.

---

# 🎥 33. Create the Camera State

Create a camera state using the Maps Compose camera-state helper.

Set the starting camera position to Durban.

Use approximately:

```text
Latitude:
-29.8587

Longitude:
31.0218

Zoom:
10
```

### Why use a default location?

When the app first opens:

* Location permission may not exist
* Current location may not be known
* No search has been performed

The map still needs somewhere to display.

---

# 🎯 34. Decide the Camera Target

Create a value representing where the camera should currently move.

Priority should be:

```text
Searched location
        ->
Current location
```

This means:

* When a search occurs, show the searched location.
* Otherwise, if the current location is available, show that.

If neither exists, leave the camera at the default position.

---

# 🎬 35. Animate Camera Movement

Use `LaunchedEffect` to observe changes to the camera target.

When the target changes:

* Animate the map camera.
* Move to the target latitude and longitude.
* Use a closer zoom level, approximately `16`.

### Why use `LaunchedEffect`?

Camera animation is a suspend operation.

It should not be run directly while Compose is drawing the UI.

`LaunchedEffect` allows the operation to run safely when the state changes.

The relationship is:

```text
searchedLocation changes
        ->
Compose notices state change
        ->
LaunchedEffect runs
        ->
Camera animates
```

---

# 🧱 36. Create the Main Screen Layout

Use a vertical layout.

A recommended structure is:

```text
Column

    Heading

    Search field

    Row
        Search button
        My Location button

    Selected place information

    Error message

    Map
```

The map should use the remaining available space.

---

# 🔎 37. Create the Search Field

Create an `OutlinedTextField`.

Requirements:

* Connected to your `searchText` state
* Updates `searchText` whenever the user types
* Fills the available width
* Uses a clear label
* Uses a useful placeholder
* Allows one line only

Example purpose:

```text
Address, landmark or business
```

Example placeholder:

```text
Gateway Theatre of Shopping
```

---

# 🔘 38. Create the Search Button

Create a button that calls your search callback.

Pass the current search text to the callback.

The button should only be enabled when:

* Search text is not blank
* A search is not currently running

When `isSearching` is true:

* Replace or supplement the text with a progress indicator.

This prevents the user from repeatedly sending the same request while the previous one is still processing.

---

# 📍 39. Create the My Location Button

Create a second button.

When clicked:

### If permission already exists

Call the current-location callback.

### If permission does not exist

Call the permission-request callback.

The UI therefore decides which action is appropriate based on:

```text
hasLocationPermission
```

---

# 🏷️ 40. Display Search Result Information

If a searched place exists, display:

* Place name
* Formatted address

Do not show blank labels before a search occurs.

Use conditional Compose UI.

Conceptually:

```text
If place information exists
    display it
Otherwise
    display nothing
```

---

# ⚠️ 41. Display Error Messages

Only display the error section when:

```text
errorMessage != null
```

Use the Material theme's error colour.

Possible messages include:

```text
Location permission denied.
```

```text
No matching place found.
```

```text
Search failed.
```

```text
Current location unavailable.
```

Do not display raw exception stack traces to users.

---

# 🗺️ 42. Create the Google Map

Give the map the remaining available height on the screen.

Create a `MapProperties` object.

Set its current-location property based on:

```text
hasLocationPermission
```

The location layer must only be enabled when permission exists.

### Why?

Trying to enable location features without permission can result in a security exception.

---

# 📍 43. Display the Current-Location Marker

Inside the `GoogleMap` content block:

Check whether:

```text
currentLocation
```

is not null.

If it exists:

* Create a `Marker`.
* Set its position to the current location.
* Give it a useful title.
* Optionally provide a short description.

The map may also display Google's built-in blue location indicator when the location layer is enabled.

---

# 🔎 44. Display the Search Marker

Check whether:

```text
searchedLocation
```

is not null.

If it exists:

* Create another marker.
* Set its coordinates to the searched location.
* Use the searched place name as the marker title.
* Use the formatted address as the marker description.

This allows the user to visually distinguish:

```text
My device location
```

from:

```text
The place I searched for
```

---

# 🧠 45. Understand the Main Components

## `GoogleMap`

Responsible for displaying the actual interactive map.

It provides:

* Zoom
* Pan
* Rotation
* Map tiles
* Gestures
* Markers

---

## `LatLng`

Represents one geographic coordinate.

It contains:

```text
Latitude
Longitude
```

Example:

```text
Durban
-29.8587, 31.0218
```

---

## `CameraPositionState`

Stores the current map-camera position.

The camera determines:

* Which location is centred
* Zoom
* Tilt
* Bearing

---

## `Marker`

Represents a point displayed on the map.

This project requires at least:

```text
Current-location marker
Search-result marker
```

---

## `MapProperties`

Controls map behaviour and optional map features.

In this demo it is used to control the current-location layer.

---

## `FusedLocationProviderClient`

Gets the Android device's location.

It combines multiple location sources rather than requiring the developer to manually choose GPS or network location.

---

## `PlacesClient`

Communicates with Places API.

Its job is:

```text
Text entered by user
        ->
Places API
        ->
Matching Place
```

---

## `SearchByTextRequest`

Represents the actual text-search request sent to Places.

It includes:

* Query
* Requested fields
* Result options

---

## `LaunchedEffect`

Runs asynchronous Compose work in response to state changes.

In this project it is useful for camera animation.

---

# ▶️ 46. Run the Application

Use:

* A physical Android device with Google Play services

or:

* An Android emulator using a Google Play-enabled image

The device needs internet access.

---

# 📍 47. Test Current Location

On a physical device:

1. Enable device location.
2. Launch the app.
3. Tap **My Location**.
4. Approve location access.
5. Select approximate or precise location.
6. Confirm that the camera moves.
7. Confirm that a current-location indicator or marker appears.

---

# 🖥️ 48. Test Current Location on an Emulator

Start the emulator.

Open:

```text
Extended Controls
-> Location
```

Enter:

```text
Latitude:
-29.8587

Longitude:
31.0218
```

Select:

```text
Set Location
```

Return to the app.

Tap:

```text
My Location
```

If the last known location is still null:

1. Open Google Maps on the emulator.
2. Grant location permission.
3. Set the emulator location again.
4. Return to your app.
5. Try again.

---

# 🔎 49. Test Place Search

Try searching for:

```text
Gateway Theatre of Shopping
```

Expected flow:

```text
User enters query
        ->
Search callback runs
        ->
Places Text Search runs
        ->
Place is returned
        ->
Coordinates are stored
        ->
Compose recomposes
        ->
Camera moves
        ->
Marker appears
```

Also test:

```text
Durban ICC
```

```text
Table Mountain
```

```text
Nelson Mandela Square
```

```text
Eiffel Tower
```

```text
coffee shops in Umhlanga
```

---

# 🐛 50. Troubleshooting

## Map is blank or grey

Check:

* Internet connection
* Maps SDK for Android is enabled
* API key exists
* Billing is connected
* Package restriction is correct
* SHA-1 restriction is correct
* Manifest metadata exists

Search Logcat for:

```text
Google Maps Android API
```

---

## App immediately closes

Check:

```properties
MAPS_API_KEY=YOUR_REAL_API_KEY
```

Ensure it is not:

```properties
MAPS_API_KEY=DEFAULT_API_KEY
```

---

## Place search fails

Check:

* Places API (New) is enabled
* Places dependency synced
* Places SDK was initialised
* Correct API key was supplied
* API restrictions permit Places API
* Internet connection works

---

## Current location is null

Possible causes:

* Device location is off
* Emulator location was not configured
* No location has recently been stored
* Permission was denied
* Google Play services restarted

A null location is a valid possibility and must be handled.

---

## My Location does nothing

Check:

* Manifest contains both location permissions
* Runtime permission request was implemented
* Permission callback updates the state
* Current-location function is called after permission succeeds

---

## Blue dot does not appear

Check whether the map location layer is enabled only after:

```text
hasLocationPermission == true
```

---

## API key works on one computer but not another

Different computers may use different debug signing certificates.

Run:

```powershell
.\gradlew signingReport
```

on the other machine and add its debug SHA-1 to the API-key restrictions.

---

# 🔐 51. Security and Privacy

Do not:

* Hardcode the API key in Kotlin
* Commit the real API key to GitHub
* Request background location for this demo
* Track users continuously
* Assume precise permission was granted
* Enable location features without permission
* Request unnecessary Places fields
* Store location history unnecessarily

The app should continue allowing place search even if the user refuses device-location permission.

---

# ✅ 52. Completion Checklist

Before considering the project complete, confirm that:

* Google Cloud project exists
* Billing is connected
* Maps SDK for Android is enabled
* Places API (New) is enabled
* API key was created
* API key is Android-restricted
* Package name is correct
* SHA-1 was added
* Secrets Gradle Plugin is configured
* API key exists in `local.properties`
* Maps Compose dependency is added
* Location dependency is added
* Places dependency is added
* Internet permission exists
* Coarse location permission exists
* Fine location permission exists
* Maps API key metadata exists
* Fused Location Provider was initialised
* Places SDK was initialised
* Places client was created
* Existing permission is checked
* Runtime permission request works
* Current location is retrieved
* Google Map displays
* Search field works
* Search request works
* Search result coordinates are stored
* Search-result name and address display
* Current-location marker displays
* Search-result marker displays
* Camera moves to current location
* Camera moves to searched location
* Errors are handled
* Loading state is handled

---

# 🧭 Final Application Architecture

```text
MainActivity
│
├── Location permission
├── Fused Location Provider
├── Places Client
├── Current-location state
├── Search-result state
├── Search logic
└── Error/loading state
        │
        ▼
MapsScreen
│
├── Search field
├── Search button
├── My Location button
├── Place information
├── Error messages
└── GoogleMap
        │
        ├── Current-location marker
        └── Search-result marker
```

The important design principle is:

```text
MainActivity
-> handles logic and data

MapsScreen
-> displays state and sends user actions back to MainActivity
```

You should be able to complete the application by implementing each section in order and testing after every major step.
