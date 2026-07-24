# 🔐 Biometric Authentication in Android

## 📚 Introduction

Biometric authentication allows an Android application to verify the identity of the person currently using the device.

Depending on the device, biometric authentication may use:

* A fingerprint
* Facial recognition
* Iris recognition
* Another biometric method supported by Android
* The device PIN, pattern or password as a fallback

Android applications should not attempt to collect, read or store a user’s actual fingerprint or facial information.

Instead, Android displays a secure system-controlled authentication prompt. The operating system performs the biometric comparison and informs the application whether authentication:

* Succeeded
* Failed
* Was cancelled
* Could not be completed

The application receives only the result of the authentication attempt. It does not receive the user’s fingerprint image, face image or biometric template. Android’s `BiometricPrompt` is specifically designed to display this system-managed authentication interface.

---

# 📦 1. Add the Required Dependencies

The first step is to add the AndroidX Biometric library to the project.

Dependencies are external libraries that provide additional classes and functionality that are not included automatically in a basic Android Studio project.

For this demonstration, the AndroidX Biometric library provides the main classes required to:

* Check whether biometric authentication is available
* Determine whether biometrics have been enrolled
* Display the Android biometric prompt
* Receive authentication results
* Allow the device PIN, password or pattern as a fallback
* Support biometric authentication across multiple Android versions

The current stable AndroidX Biometric release is:

```kotlin
implementation("androidx.biometric:biometric:1.1.0")
```

AndroidX Biometric `1.1.0` remains the stable release, while later versions are currently available only through preview channels.

---

## ❓ Why is this dependency required?

Without this dependency, Android Studio will not recognise classes such as:

```text
BiometricManager
BiometricPrompt
BiometricPrompt.PromptInfo
BiometricPrompt.AuthenticationCallback
```

These classes are not created by the developer. They are supplied by the AndroidX Biometric library.

The dependency also improves compatibility between Android versions. Instead of creating separate biometric implementations for different Android versions and manufacturers, the application uses one consistent AndroidX API.

---

# 📄 2. Update the Android Manifest

After adding the dependency, the next step is to declare that the application uses biometric authentication.

Add the following permission inside `<manifest>`, but above `<application>`:

```xml
<uses-permission android:name="android.permission.USE_BIOMETRIC" />
```

---

## ❓ Why is the biometric permission required?

The manifest tells Android which device features and permissions the application intends to use.

The following permission:

```xml
android.permission.USE_BIOMETRIC
```

allows the application to request authentication through biometric methods supported by the device. Android documents this permission as allowing an application to use device-supported biometric modalities.

Without this permission, the application should not attempt to access Android’s biometric authentication services.

---

## 🔔 Is this a runtime permission?

No.

The user does not receive a permission popup such as:

```text
Allow this application to use fingerprints?
```

This differs from runtime permissions such as:

* Camera
* Microphone
* Location
* Contacts

Biometric authentication is handled through the secure system prompt. The application requests authentication only when it needs the user to verify their identity.

---

## 🚫 Do not use the old fingerprint permission

You may find older tutorials that use:

```xml
<uses-permission android:name="android.permission.USE_FINGERPRINT" />
```

This permission is deprecated.

The modern permission is:

```xml
<uses-permission android:name="android.permission.USE_BIOMETRIC" />
```

Android deprecated `USE_FINGERPRINT` in API level 28 and recommends using `USE_BIOMETRIC` instead.

---

## 🌐 Is Internet permission required?

No.

A standalone biometric demonstration does not need:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Biometric authentication occurs locally on the device.

Internet permission would only be required if the application also used features such as:

* Firebase
* Google Sign-In
* An online API
* Cloud storage
* A remote database

For this demonstration, biometric authentication is independent of online authentication.

---

# 🏗️ 3. Prepare the Main Activity

In a normal Jetpack Compose project, the activity will initially inherit from:

```text
ComponentActivity
```

For the biometric demonstration, it should inherit from:

```text
FragmentActivity
```

---

## ❓ Why use `FragmentActivity`?

The AndroidX version of `BiometricPrompt` integrates with the activity and fragment lifecycle.

Using `FragmentActivity` allows the biometric prompt to be attached correctly to the application’s lifecycle.

This helps Android manage the prompt during events such as:

* Screen rotation
* The app temporarily moving into the background
* Activity recreation
* System interruptions
* Prompt cancellation

The project can still use Jetpack Compose with `FragmentActivity`.

You may continue using:

```text
setContent
Scaffold
Composable functions
Compose state
Material 3
```

Changing the activity type does not mean the project must use XML layouts or traditional fragments.

---

## 📥 Important imports

The project will require imports for the main biometric classes.

These include:

```text
BiometricManager
BiometricPrompt
FragmentActivity
ContextCompat
```

Each import has a separate purpose.

### `BiometricManager`

Used to check whether biometric authentication is currently available.

It can help determine whether:

* The device has biometric hardware
* The hardware is temporarily unavailable
* A biometric has been enrolled
* The requested biometric strength is supported
* A secure device credential exists

### `BiometricPrompt`

Used to display the system authentication prompt.

The application does not design its own fingerprint dialog. Android displays the prompt using a secure and consistent system interface.

### `FragmentActivity`

Provides the lifecycle support required by the AndroidX biometric prompt.

### `ContextCompat`

Used to obtain an executor that allows authentication callback results to be handled on the main application thread.

UI state should normally be updated on the main thread.

---

# 🧠 4. Plan the Application State

Before creating the functions, you must understand that the application needs to remember its current authentication state.

The app will generally have two main states:

```text
Locked
Authenticated
```

When the app starts, it should be locked.

After successful biometric authentication, it should display protected content.

If the user locks the app again, it should return to the locked state.

---

## 🔒 Authentication state

The project needs a state value that answers:

```text
Has the user successfully authenticated?
```

This state controls which screen is displayed.

Conceptually:

```text
false
-> Show the biometric authentication screen

true
-> Show the protected screen
```

In Jetpack Compose, state changes cause the interface to recompose = the developer does not manually close one screen and open another. Instead, the UI observes the authentication state and displays the correct content.

---

## 💬 Message state

The project should also include a state value for messages.

These messages may tell the user:

* Biometrics are available
* No biometric has been enrolled
* The sensor is unavailable
* Authentication failed
* Authentication was cancelled
* Too many attempts were made

This is important because users should not be left without feedback when authentication cannot continue.

---

## ⏳ Loading or authentication state

The app may include a Boolean state that records whether authentication is currently taking place.

This can be used to:

* Disable the authentication button
* Display a progress indicator
* Prevent repeated button presses
* Inform the user that an authentication attempt is active

Although Android displays the actual biometric prompt, the Compose interface should still manage its own state clearly.

---

## ✅ Availability state

The application should record whether biometric authentication is available.

For example:

```text
Biometrics available
-> Enable the Authenticate button

Biometrics unavailable
-> Disable the button and show an explanation
```

This prevents the user from repeatedly pressing a button that cannot work on the current device.

---

# 🛠️ 5. Functions Required in the Activity

The biometric implementation should be divided into clearly named functions.

You should not place all biometric work directly inside `onCreate()`.

Separating functionality into functions makes the project:

* Easier to read
* Easier to test
* Easier to explain
* Easier to debug
* Easier to maintain
* Less repetitive

The main functions are described below.

---

## 🔍 Function 1: Set up biometric authentication

Name:

```text
setupBiometricAuthentication()
```

This function is responsible for preparing the biometric system.

It should be called when the activity is created.

Its responsibilities include:

* Creating a `BiometricManager`
* Declaring which authenticators the app accepts
* Checking whether authentication is available
* Preparing the biometric prompt
* Preparing the prompt information
* Defining authentication callbacks

---

## ❓ Why should setup happen first?

The app should not show an authentication button before it knows whether the device can authenticate.

For example, the device may:

* Have no biometric sensor
* Have a sensor that is temporarily unavailable
* Support fingerprints but have none enrolled
* Have no secure screen lock configured
* Not support the requested biometric strength

The setup function allows the app to check these conditions before the user attempts authentication.

---

## 🧭 `BiometricManager`

Inside the setup function, the application uses `BiometricManager`.

Its main purpose is to answer:

```text
Can this device authenticate using the methods requested by the app?
```

The app should not assume that every Android device has:

* A fingerprint reader
* Facial recognition
* A strong biometric sensor
* Enrolled biometric information
* A device PIN or password

`BiometricManager.canAuthenticate()` checks the current device state before the prompt is displayed. The authenticators interface allows the application to declare which types of authentication it supports.

---

## 🛡️ Allowed authenticators

The application must decide which authentication methods it accepts.

Common options include:

```text
BIOMETRIC_STRONG
BIOMETRIC_WEAK
DEVICE_CREDENTIAL
```

### `BIOMETRIC_STRONG`

Requests biometric methods that Android classifies as strong.

Examples may include:

* Certain fingerprint sensors
* Certain secure facial recognition systems

Not every facial recognition system is considered strong.

A basic camera-based face unlock system may not meet the security requirements of `BIOMETRIC_STRONG`.

### `BIOMETRIC_WEAK`

Allows biometric methods classified as weak or strong.

This supports more devices but provides a lower required security classification.

### `DEVICE_CREDENTIAL`

Allows the user to authenticate using the device’s:

* PIN
* Pattern
* Password

This is useful as a fallback when:

* The fingerprint sensor cannot read the user
* The user is wearing gloves
* Face recognition cannot see the user clearly
* The biometric sensor is temporarily unavailable
* The user prefers to use the device credential

---

## 🔐 Recommended classroom configuration

For this demonstration, the app can allow:

```text
BIOMETRIC_STRONG
OR
DEVICE_CREDENTIAL
```

This means the user may authenticate with:

* A strong biometric, or
* Their secure device screen lock

This provides a useful demonstration of both biometric and device-credential authentication.

---

## ⚠️ Compatibility consideration

Authenticator combinations can behave differently on older Android versions.

For a modern classroom project, you should test on:

* A recent physical Android device, or
* A recent Android emulator

The AndroidX library helps provide compatibility, but the available authentication options still depend on:

* Android version
* Device manufacturer
* Hardware capabilities
* Device security configuration

---

## 📊 Availability results

The setup function should respond to the result returned by `canAuthenticate()`.

Important possible results include the following.

### `BIOMETRIC_SUCCESS`

The requested authentication method is available.

The application can:

* Enable the authentication button
* Prepare to display the biometric prompt
* Inform the user that authentication is available

### `BIOMETRIC_ERROR_NO_HARDWARE`

The device does not contain suitable biometric hardware.

The app should:

* Disable biometric authentication
* Display a clear message
* Avoid opening the prompt

### `BIOMETRIC_ERROR_HW_UNAVAILABLE`

The hardware exists but is temporarily unavailable.

This may occur when:

* The sensor is busy
* The sensor has temporarily stopped responding
* The operating system has disabled it temporarily

The user may be able to try again later.

### `BIOMETRIC_ERROR_NONE_ENROLLED`

The device supports biometric authentication, but the user has not enrolled a biometric.

For example:

* No fingerprint has been added
* No face has been registered
* No secure device credential has been configured

The app should explain that the user must configure device security in Android Settings.

### `BIOMETRIC_ERROR_SECURITY_UPDATE_REQUIRED`

The biometric system requires a security update before it can be used safely.

The application should not attempt to bypass this result.

### `BIOMETRIC_ERROR_UNSUPPORTED`

The device does not support the specific combination of authenticators requested by the app.

For example, a particular Android version may not support the selected biometric and credential combination.

### `BIOMETRIC_STATUS_UNKNOWN`

Android could not determine the current biometric status.

The application should treat authentication as unavailable until a valid status is known.

---

# 🪟 6. Prepare the Biometric Prompt

The setup function must prepare a `BiometricPrompt`.

`BiometricPrompt` is the AndroidX component that displays the secure authentication interface.

It is responsible for:

* Opening the biometric dialog
* Communicating with the device sensor
* Handling system security
* Reporting the authentication result
* Supporting device-credential fallback

The system provides the dialog rather than allowing every application to create its own biometric interface. This gives users a recognisable and trusted authentication experience.

---

## ❓ Why should developers not design their own fingerprint dialog?

A custom dialog cannot directly perform secure biometric verification.

Creating a fake fingerprint screen would introduce several problems:

* Users may not know whether the screen is trustworthy
* The application could mislead users
* Device manufacturers use different biometric hardware
* The app would not have access to the protected biometric comparison
* The interface may not follow Android security requirements
* The biometric prompt may not survive lifecycle changes correctly

The correct approach is to ask Android to display its official system prompt.

---

# 📣 7. Authentication Callbacks

The biometric prompt needs an authentication callback.

A callback is a section of the app that Android contacts when something happens during authentication.

The main callback events are:

```text
Authentication succeeded
Authentication failed
Authentication error
```

These events are not the same.

---

## ✅ Authentication succeeded

This event occurs when Android successfully verifies:

* The user’s biometric, or
* The accepted device credential

The app should then:

* Mark the user as authenticated
* Remove previous error messages
* Stop any loading state
* Display the protected screen

This is the only callback that should grant access to protected content.

---

## ❌ Authentication failed

This occurs when a biometric is presented but is not recognised.

For example:

* The wrong finger is used
* The fingerprint is only partially detected
* The face does not match
* The sensor cannot obtain a clear reading

The prompt normally remains open so that the user can try again.

The application must not unlock the protected screen.

The UI may show a message such as:

```text
Biometric not recognised. Please try again.
```

---

## ⚠️ Authentication error

An error means authentication could not continue normally.

Examples include:

* The user cancelled
* The system cancelled
* The sensor became unavailable
* Too many failed attempts occurred
* The user was temporarily locked out
* No biometric was enrolled
* The prompt timed out

The app should:

* Keep the protected area locked
* Stop the loading state
* Display an appropriate message
* Allow another attempt when appropriate

---

## 🧵 Main executor

The biometric prompt requires an executor.

The executor determines where callback code runs.

For this Compose project, the application should use the main executor.

This is important because the callback will update:

* Compose state
* Authentication status
* Error messages
* Loading indicators
* Screen content

UI state should be updated from the main application thread.

---

# 📝 8. Prepare the Prompt Information

The biometric system also requires a `PromptInfo` object.

`PromptInfo` describes the text and options displayed in the Android authentication dialog.

It normally includes:

* A title
* A subtitle
* A description
* The allowed authenticators

---

## 🏷️ Prompt title

The title is the main heading shown to the user.

Example purpose:

```text
Verify your identity
```

The title should clearly describe what the user needs to do.

Avoid vague titles such as:

```text
Continue
Security
Login
```

A clear title improves usability and helps the user understand why Android is requesting authentication.

---

## 💬 Prompt subtitle

The subtitle provides a short explanation.

For example:

```text
Authenticate to access the protected area
```

It should explain the immediate reason for authentication.

---

## 📝 Prompt description

The description provides more detail.

For example, it may explain that the user can use:

* A fingerprint
* Facial recognition
* A device PIN
* A device pattern
* A device password

The description should be brief. The biometric prompt is not the correct place for a long paragraph.

---

## 🚫 Negative button and device credentials

When an application allows `DEVICE_CREDENTIAL`, it should not also configure a custom negative button.

This is because Android uses the prompt’s alternative action to provide the device-credential option.

The system must manage this fallback securely and consistently.

---

# ▶️ 9. Function 2: Show the Biometric Prompt

Suggested name:

```text
showBiometricPrompt()
```

This function is called when the user presses the authentication button.

Its responsibility is to start the authentication process.

Before displaying the prompt, it should consider whether:

* Authentication is available
* The prompt has been prepared
* Another authentication attempt is already active
* The app should clear an old error message
* The UI should enter a loading state

The function then asks the `BiometricPrompt` to authenticate using the prepared `PromptInfo`.

---

## ❓ Why use a separate function?

The authentication button should not contain all biometric setup code.

The button’s responsibility is simply:

```text
The user requested authentication.
```

The separate function then decides how to respond.

This improves separation of concerns.

The UI handles user interaction, while the activity handles biometric authentication.

---

# 🔒 10. Function 3: Lock the Application

The application also needs a way to return to the locked state.

Suggested function or event name:

```text
lockApplication()
```

Its responsibility is to:

* Change the authenticated state back to false
* Remove unnecessary old messages
* Return the user to the biometric authentication screen

This does not delete biometric information.

It also does not remove fingerprints from the device.

It only changes the current app session from:

```text
Authenticated
```

to:

```text
Locked
```

---

## ❓ Why include a lock function?

A lock function makes the biometric process easier to demonstrate.

Without it, you may need to:

* Close the app
* Force-stop the app
* Restart the emulator
* Relaunch the activity

A lock button lets the user repeat the authentication process immediately.

It also demonstrates the difference between:

```text
Locking an application
```

and:

```text
Signing out of an online account
```

This project does not contain an online account, so the user is simply locking and unlocking protected local content.

---

# 🧩 11. Configure `setContent`

After the biometric functions have been prepared, the next step is to configure the Compose interface inside `setContent`.

`setContent` is the entry point for the Jetpack Compose UI.

It tells Android which composable content should be displayed inside the activity.

The basic structure should contain:

```text
Application theme
Scaffold
Authentication-state decision
Screen
```

---

## 🎨 Application theme

The project theme should surround the application content.

For example:

```text
BiometricAuthDemoTheme
```

The theme provides:

* Colours
* Typography
* Material styling
* Light and dark mode behaviour
* Consistent component appearance

You should use the theme generated with the Android Studio project unless they are intentionally customising it.

---

## 🏗️ Scaffold

A `Scaffold` provides a standard Material layout structure.

It can support elements such as:

* Top app bars
* Bottom navigation
* Floating action buttons
* Snackbars
* Main content padding

Even if the first biometric demo is simple, using a `Scaffold` gives the application a clean structure for later development.

The scaffold provides `innerPadding`.

That padding should be passed to the current screen so that content does not overlap system bars or scaffold components.

---

## 🔀 Screen decision

Inside `setContent`, the app should inspect the authentication state.

Conceptually:

```text
If the user is authenticated
    Show the protected screen
Otherwise
    Show the authentication screen
```

This is known as conditional UI.

The application does not need a navigation library for this basic two-screen demonstration.

The authentication state itself determines which composable is visible.

---

## 🔄 Recomposition

When authentication succeeds, the app updates the authentication state.

Compose notices that the state changed and automatically runs the relevant UI section again.

The result is:

```text
Authentication state changes to true
        ->
Compose recomposes
        ->
Authentication screen is removed
        ->
Protected screen is displayed
```

When the user locks the application:

```text
Authentication state changes to false
        ->
Compose recomposes
        ->
Protected screen is removed
        ->
Authentication screen is displayed
```

---

# 📱 12. Create the Authentication Screen

The first composable screen represents the locked state of the application.

Suggested name:

```text
AuthenticationScreen
```

This screen should not contain the internal biometric logic.

It should receive information and events through parameters.

---

## 📥 Authentication screen parameters

The screen may receive:

```text
Whether authentication is available
Whether authentication is in progress
A biometric status or error message
An event for the Authenticate button
```

This allows the screen to remain focused on presentation.

The activity remains responsible for:

* Biometric checks
* Prompt creation
* Authentication callbacks
* Authentication state

---

## 🧱 Required authentication screen elements

The screen should contain the following.

### Application title

Example:

```text
Biometric Authentication
```

The title tells the user what security method the application uses.

### Explanation

The screen should briefly explain why authentication is required.

For example:

```text
Verify your identity to access the protected content.
```

The message should be clear without being overly technical.

### Authentication status

The screen should display messages returned by the activity.

Possible messages include:

```text
Biometric authentication is available.

No biometric information has been enrolled.

The biometric sensor is temporarily unavailable.

Authentication was cancelled.

Biometric not recognised.
```

### Authenticate button

The screen should include a clearly labelled button.

Example:

```text
Authenticate
```

The button should be disabled when authentication is unavailable.

This prevents the user from starting an action that cannot succeed.

### Progress indicator

A progress indicator may be displayed while authentication is active.

This provides visual feedback and prevents users from repeatedly pressing the button.

---

## 🎨 Recommended layout structure

A simple layout may use:

```text
Column
```

with:

```text
Vertical centring
Horizontal centring
Screen padding
Spacing between elements
```

Possible structure:

```text
Biometric Authentication

Verify your identity to access the protected content.

Biometric status or error message

Authenticate button
```

You may create their own visual design as long as the required information and functionality remain clear.

---

## ♿ Accessibility considerations

The authentication screen should:

* Use readable text sizes
* Provide sufficient spacing
* Avoid relying only on colour to communicate errors
* Use clear button labels
* Support device credentials where possible
* Display understandable feedback
* Avoid technical error codes in the main UI

For example, displaying:

```text
ERROR_LOCKOUT_PERMANENT
```

is less helpful than:

```text
Biometric authentication is locked. Unlock your device using its PIN, pattern or password.
```

---

# 🔓 13. Create the Protected Screen

The second composable represents the authenticated state.

Suggested name:

```text
ProtectedScreen
```

This screen should only be displayed after the authentication success callback has updated the app state.

---

## 🧱 Required protected screen elements

The screen should include:

### Success heading

Example:

```text
Authentication Successful
```

### Access confirmation

Example:

```text
You now have access to the protected content.
```

### Protected content

You should include content that is not visible on the locked screen.

Examples may include:

* A private message
* Account information
* Secure notes
* A protected settings section
* A confidential document title
* A demonstration dashboard

The content does not need to contain genuinely sensitive information for the classroom demo.

### Lock button

The screen must include a button that returns the application to the locked state.

Example:

```text
Lock Application
```

This button should update the authentication state rather than closing the app.

---

## 🔐 Why must protected content be conditionally displayed?

The protected content should not merely be hidden behind another visual element.

It should only be included in the Compose UI when the user is authenticated.

Conceptually:

```text
Not authenticated
-> Protected screen is not composed

Authenticated
-> Protected screen is composed
```

This creates a clearer separation between the locked and unlocked application states.

---

# 🧪 14. Test on a Physical Android Device

Before testing, the device must have security configured.

You should open the phone settings and configure:

```text
Settings
-> Security and privacy
-> Device unlock
```

The exact menu names differ between manufacturers.

The device should have:

* A PIN, pattern or password
* At least one enrolled fingerprint or face
* A supported biometric sensor

---

## ✅ Physical-device test cases

You should test the following situations.

### Test 1: Successful biometric

1. Open the application.
2. Press **Authenticate**.
3. Use the enrolled fingerprint or face.
4. Confirm that the protected screen appears.

### Test 2: Unrecognised biometric

1. Lock the application.
2. Press **Authenticate**.
3. Use an unregistered finger.
4. Confirm that access is not granted.
5. Confirm that the prompt allows another attempt.

### Test 3: User cancellation

1. Open the prompt.
2. Cancel or press Back.
3. Confirm that the app remains locked.
4. Confirm that a useful message appears.

### Test 4: Device credential

1. Open the prompt.
2. Select the PIN, pattern or password option.
3. Enter the correct device credential.
4. Confirm that access is granted.

### Test 5: Lock application

1. Authenticate successfully.
2. Press **Lock Application**.
3. Confirm that the authentication screen returns.
4. Confirm that protected content is no longer visible.

---

# 🖥️ 15. Test on Emulator

The emulator can simulate fingerprint authentication.

Create or start a virtual device through:

```text
Tools
-> Device Manager
```

Use a recent Android system image.

---

## 🔢 Configure a device PIN

Inside the emulator, open:

```text
Settings
-> Security
-> Screen lock
```

Set a PIN, pattern or password.

Android normally requires a secure screen lock before a fingerprint can be enrolled.

---

## 👆 Enrol a fingerprint

Inside the emulator settings, locate the fingerprint or biometric section.

Add a fingerprint when prompted.

The emulator does not scan a real finger. Android Studio simulates fingerprint input.

---

## 🎛️ Send a fingerprint event

When the biometric prompt appears, open the emulator’s extended controls.

Depending on the Android Studio version, this may appear under:

```text
Extended Controls
-> Fingerprint
```

or:

```text
Extended Controls
-> Virtual sensors
-> Fingerprint
```

Select a registered fingerprint and send the touch event.

The biometric prompt should report successful authentication.

---

# 🧭 16. Complete Project Flow

The finished application should follow this sequence:

```text
Application opens
        ->
Biometric setup function runs
        ->
App checks device capabilities
        ->
Authentication screen appears
        ->
User presses Authenticate
        ->
Android system prompt appears
        ->
Authentication succeeds
        ->
Authentication state changes
        ->
Compose recomposes
        ->
Protected screen appears
        ->
User presses Lock Application
        ->
Authentication state resets
        ->
Authentication screen returns
```

---

# 🔍 17. What Each Component Does

## AndroidX Biometric dependency

Provides the biometric classes and compatibility support.

## Manifest permission

Declares that the application uses device-supported biometric authentication.

## `FragmentActivity`

Provides the lifecycle support used by AndroidX `BiometricPrompt`.

## `BiometricManager`

Checks whether the requested authentication methods are available.

## Allowed authenticators

Define which methods the user may use, such as a strong biometric or device credential.

## `BiometricPrompt`

Displays the secure Android-controlled authentication interface.

## `PromptInfo`

Defines the title, subtitle, description and permitted authentication methods.

## Authentication callback

Receives success, failure and error events from Android.

## Authentication state

Controls whether the locked or protected screen is displayed.

## `setContent`

Displays the Jetpack Compose interface.

## Authentication screen

Allows the user to begin biometric authentication and displays feedback.

## Protected screen

Displays content only after authentication succeeds.

---

# ⚠️ 18. Important Security Notes

## Biometrics do not create an application account

Biometric authentication confirms that the current device user is authorised.

It does not automatically provide:

* A username
* An email address
* A cloud account
* A Firebase user
* A Google account
* A database record

Biometrics and online login solve different problems.

---

## The application does not store biometric data

The application must never attempt to save:

* Fingerprint images
* Face images
* Biometric templates
* Raw sensor information

Android stores and manages enrolled biometric information through the device’s protected security system.

The app only receives an authentication result.

---

## A Boolean is suitable only for this demonstration

In a classroom demonstration, the protected screen may be controlled using an in-memory Boolean authentication state.

This is useful for teaching:

* Biometric availability
* System prompts
* Authentication callbacks
* Compose state
* Conditional UI

However, a Boolean alone is not sufficient protection for highly sensitive production data.

For stronger security, biometrics may be combined with:

* Android Keystore
* Cryptographic keys
* Encrypted local storage
* Secure server authentication
* Token expiration
* Session timeout
* Reauthentication before sensitive actions

AndroidX Biometric can also be used with cryptographic operations for more advanced security implementations.

---

# ✅ 19. Your Implementation Checklist

Before submitting or demonstrating the project, confirm that:

* The AndroidX Biometric dependency was added
* Gradle Sync completed successfully
* `USE_BIOMETRIC` was added to the manifest
* The activity uses `FragmentActivity`
* `BiometricManager` checks device availability
* The app handles all major availability results
* The accepted authenticators are clearly defined
* The biometric prompt is prepared
* Prompt information includes a title and explanation
* Authentication success is handled
* Authentication failure is handled
* Authentication errors are handled
* The app starts in a locked state
* The authentication screen is displayed first
* The Authenticate button opens the system prompt
* Protected content appears only after success
* Authentication failure does not unlock the app
* The user can cancel the prompt
* The protected screen contains a Lock button
* Locking returns the user to the authentication screen
* The app was tested on an emulator or physical device
* The code contains meaningful comments
* The UI displays clear, user-friendly messages

---

# 🎯 After completing this demonstration, you should be able to:

* Explain what biometric authentication is
* Explain why Android controls the biometric prompt
* Add the AndroidX Biometric dependency
* Declare biometric use in the manifest
* Check whether biometric authentication is available
* Distinguish between biometric strength levels
* Allow device credentials as a fallback
* Explain authentication callbacks
* Manage locked and authenticated UI states
* Display protected content conditionally
* Test biometrics using a device or emulator
* Explain the security limitations of a basic biometric demonstration
