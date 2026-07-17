# 🔐 Google SSO in Android Using Firebase Authentication

Google Single Sign-On allows users to access an Android application using an existing Google account.

Instead of creating and remembering another username and password, the user selects a Google account that is already available on their device. Google verifies the user’s identity, while Firebase Authentication creates and manages the authenticated session inside the application.

The application never receives or stores the user’s Google password.

---

# 🏗️ Main Technologies Involved

A Google SSO implementation with Firebase normally involves three main components:

### 📱 Android Application

The Android application displays the sign-in interface, starts the authentication process and responds to the authentication result.

### 🟢 Google Identity Services

Google verifies the user’s identity and returns a secure Google ID token.

### 🔥 Firebase Authentication

Firebase validates the Google ID token, creates or retrieves the Firebase user account and maintains the authenticated session.

The general flow is:

```text
Android Application
       ->
Google authenticates the user
       ->
Google returns an ID token
       ->
Firebase verifies the token
       ->
Firebase signs the user into the application
```

---

# 📝 Step 1: Create the Android Project

Create a new Android application in Android Studio.

During project creation, important application details are generated, including:

* Application name
* Package name
* Minimum Android version
* Project structure
* Gradle build configuration

The **package name** is particularly important because Firebase uses it to identify the Android application.

An example package name could be:

```text
com.example.studentportal
```

The package name entered in Firebase must match the package name in the Android project exactly. Even a small difference will cause Firebase configuration problems.

---

# 🔥 Step 2: Create a Firebase Project

Open the Firebase Console and create a Firebase project.

The Firebase project acts as the backend environment for the Android application. It can contain services such as:

* Firebase Authentication
* Cloud Firestore
* Firebase Realtime Database
* Firebase Storage
* Firebase Analytics
* Firebase Cloud Messaging

For Google SSO, the most important service is **Firebase Authentication**.

A single Firebase project may contain multiple registered applications, such as:

* An Android application
* An iOS application
* A web application

These applications can all use the same Firebase backend.

---

# 📱 Step 3: Register the Android App with Firebase

Inside the Firebase project, add a new Android application.

Firebase will request the Android application's package name. This must be copied from the Android project and entered exactly as it appears.

Firebase may also request:

* An application nickname
* SHA certificate fingerprints

The application nickname is mainly used to help identify the app inside the Firebase Console. It does not change the Android application’s displayed name.

Registering the application creates a connection between the Android project and the Firebase project.

---

# 🔑 Step 4: Add the SHA Certificate Fingerprints

Firebase may require the application’s certificate fingerprints, especially for Google authentication.

The main fingerprints are:

* SHA-1
* SHA-256

These fingerprints identify the certificate used to sign the Android application.

During development, the application is normally signed using a **debug certificate**. When the application is released, it is usually signed using a separate **release certificate**.

This means that the debug and release versions of an application may have different SHA fingerprints.

The fingerprints help Google confirm that the authentication request comes from a trusted version of the registered application.

If the correct fingerprint is not registered, Google Sign-In may fail even when the rest of the Firebase configuration appears correct.

---

# ⚙️ Step 5: Download the Firebase Configuration File

After registering the Android application, Firebase provides a configuration file named:

```text
google-services.json
```

This file contains configuration information that allows the Android application to locate and communicate with the correct Firebase project.

It includes identifiers such as:

* Firebase project ID
* Application ID
* API-related configuration
* OAuth client information

The file is added to the Android application module.

It should normally be placed inside:

```text
app/
```

It should not be placed randomly in the root project folder, because the Firebase Gradle plugin expects it in the application module.

---

# 🔌 Step 6: Connect Firebase to the Android Project

The Android project must include the Firebase configuration plugin and Firebase libraries.

These allow the project to:

* Read the `google-services.json` file
* Initialise Firebase
* Access Firebase Authentication
* Communicate with Firebase services

Firebase recommends using its Android Bill of Materials, commonly called the **Firebase BoM**, to manage compatible Firebase library versions.

The BoM helps ensure that the Firebase libraries used in the project are designed to work together.

At this stage, the Android application is connected to Firebase, but Google authentication still needs to be enabled.

## 🔌 Plugins and Dependencies

The project uses **Firebase Authentication**, **Android Credential Manager**, and **Google Identity Services** to support Google Single Sign-On.

### 🧩 Project-Level Plugin

In the **project-level** `build.gradle.kts` file, add the Google Services plugin:

```kotlin
plugins {
    // Existing project plugins

    id("com.google.gms.google-services") version "4.5.0" apply false
}
```

The Google Services plugin reads the Firebase configuration contained in the `google-services.json` file.

---

### 📱 App-Level Plugins

In the **app-level** `build.gradle.kts` file, add the Google Services plugin:

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")

    // Allows the application to use Firebase configuration
    id("com.google.gms.google-services")
}
```

When using Jetpack Compose, the project may also include:

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")

    id("com.google.gms.google-services")
}
```

Keep the plugins already generated by Android Studio. Only add the Google Services plugin if it is not already present.

A simplified app-level `build.gradle.kts` configuration will contain the following firebase sso dependancies:

```kotlin

dependencies {

    // Firebase
    implementation(platform("com.google.firebase:firebase-bom:34.16.0"))
    implementation("com.google.firebase:firebase-auth")

    // Credential Manager
    implementation("androidx.credentials:credentials:1.3.0")
    implementation(
        "androidx.credentials:credentials-play-services-auth:1.3.0"
    )

    // Google Identity
    implementation(
        "com.google.android.libraries.identity.googleid:googleid:1.1.1"
    )

    // Keep the existing Android and Compose dependencies
    // generated by Android Studio.
}
```

---

# 🔐 Step 7: Enable Google as a Sign-In Provider

Open the **Authentication** section in the Firebase Console.

Before using authentication providers, Firebase may require the Authentication service to be activated.

In the list of sign-in providers, select **Google** and enable it.

The Google provider configuration normally requires:

* Google provider enabled
* Project support email selected
* Changes saved

Enabling the provider tells Firebase that users are allowed to authenticate using Google accounts.

Firebase Authentication supports Google and several other federated identity providers.

---

# 🌐 Step 8: Identify the Web Client ID

Google authentication uses an OAuth client ID to identify the application requesting authentication.

Even though the application is an Android application, Firebase Google authentication commonly uses the **Web client ID**, sometimes referred to as the:

```text
default_web_client_id
```

This ID represents the backend or server audience that should receive the Google ID token.

It is important not to confuse the Web client ID with the Android OAuth client ID.

The Android client identifies the installed Android application, while the Web client ID is used when requesting the Google ID token that Firebase will later verify.

Using the incorrect client ID may cause token errors or failed authentication.

---

# 📦 Step 9: Add the Authentication Libraries

The Android project requires libraries for the different parts of the authentication process.

These normally include libraries for:

* Firebase Authentication
* Android Credential Manager
* Google ID credentials

Firebase Authentication manages the Firebase user session.

Credential Manager manages the sign-in experience on the Android device.

Google ID libraries allow the application to request and interpret a Google identity credential.

Credential Manager provides a modern Android authentication interface and can work with passwords, passkeys and Google ID tokens.

---

# 🪪 Step 10: Use Android Credential Manager

Credential Manager is the modern Android API used to request user credentials.

It provides a more consistent authentication experience and can support different types of credentials through one interface.

For Google SSO, Credential Manager can:

* Search for Google accounts available to the user
* Present an account-selection interface
* Return the selected Google credential
* Return the Google ID token
* Report cancellations or authentication errors

Google’s current Android guidance uses Credential Manager for Sign in with Google.

---

# 🎨 Step 11: Create the Signed-Out Interface

The application should display a signed-out screen when no authenticated Firebase user exists.

This screen may include:

* Application logo
* Application name
* Welcome message
* Brief explanation of the sign-in process
* **Continue with Google** button
* Loading indicator
* Error message area

The sign-in button should clearly tell the user that Google will be used for authentication.

For example:

```text
Continue with Google
```

The button should not misleadingly suggest that the application itself is collecting the user’s Google password.

---

# 👆 Step 12: Start Authentication When the User Selects Sign-In

When the user taps the Google sign-in button, the application creates a credential request.

The request indicates that the application wants a Google ID credential.

The request includes the Web client ID so that Google knows which application or backend is requesting the token.

Credential Manager then attempts to find suitable Google accounts.

Depending on the configuration, the application may first search for accounts that have already been authorised for the app.

If no authorised account is found, the application may perform a broader request that allows the user to select another Google account.

---

# 👤 Step 13: Display the Google Account Selection Interface

Credential Manager displays the available Google account options.

The user may:

* Select an existing Google account
* Select an account previously used with the application
* Add another Google account
* Cancel the sign-in process

The exact interface may differ depending on:

* Android version
* Device manufacturer
* Google Play services
* Number of Google accounts on the device
* Whether the user has previously authorised the app

The application does not control or collect the Google password screen.

Google handles that part of the process.

---

# 🛡️ Step 14: Google Authenticates the User

Google verifies the user’s identity.

The user may be required to:

* Select an account
* Enter a password
* Confirm a device lock
* Complete multi-factor authentication
* Approve the application’s request

The authentication requirements depend on the user’s Google account and security settings.

Once authentication succeeds, Google creates a signed ID token.

---

# 🎫 Step 15: Receive the Google ID Token

Credential Manager returns a Google identity credential to the Android application.

The credential contains an **ID token**.

An ID token is a digitally signed token containing identity-related claims, such as:

* User identifier
* Email address
* Display name
* Profile image information
* Token issuer
* Intended audience
* Token expiry time

The token is proof that Google successfully authenticated the user.

It is not the user’s password, and it should not be treated as a password.

---

# 🔁 Step 16: Convert the Google Token into a Firebase Credential

Google has authenticated the user, but the user is not yet signed into Firebase.

The Google ID token must be exchanged for a Firebase-compatible authentication credential.

This credential connects the Google identity to Firebase Authentication.

Conceptually, the process is:

```text
Google ID Token
       ->
Firebase Authentication Credential
       ->
Firebase Sign-In Request
```

This step allows Firebase to verify the identity supplied by Google.

---

# 🔥 Step 17: Authenticate with Firebase

The Firebase credential is submitted to Firebase Authentication.

Firebase checks that:

* The Google token is valid
* The token has not expired
* Google issued the token
* The token was intended for the configured application
* The Google provider is enabled
* The application is correctly registered

If validation succeeds, Firebase creates an authenticated session.

Firebase Authentication then provides access to the current Firebase user.

---

# 👥 Step 18: Create or Retrieve the Firebase User

Firebase checks whether the Google account has previously signed into the Firebase project.

### New user

If the Google account has not been used before, Firebase creates a new Firebase Authentication user record.

### Returning user

If the account already exists, Firebase retrieves the existing user record and signs the user into it.

Each Firebase user is assigned a unique identifier called a:

```text
UID
```

The UID is unique within the Firebase project and is more reliable than using an email address as a database identifier.

An email address can change, but the Firebase UID normally remains associated with the same Firebase account.

---

# 📄 Step 19: Access the Authenticated User Details

After successful sign-in, the application can access information made available through the authenticated Firebase user.

This may include:

* Firebase UID
* Display name
* Email address
* Profile picture URL
* Authentication provider
* Email verification status

The amount of information available depends on what Google supplies and what the user has allowed.

The application should only use information that is required for its functionality.

---

# 💾 Step 20: Store Additional User Information

Firebase Authentication stores identity and authentication information, but it is not intended to function as the application’s full user profile database.

The application may create a separate user record in:

* Cloud Firestore
* Firebase Realtime Database

For example, an application could store:

```text
users
 └── firebase-user-uid
      ├── name
      ├── email
      ├── role
      ├── dateCreated
      └── preferences
```

The Firebase UID can be used as the document or record identifier.

This links the database profile to the authenticated Firebase account.

Sensitive information should not be stored unnecessarily.

---

# 🏠 Step 21: Display the Signed-In Interface

After authentication succeeds, the application should display its signed-in state.

This could be:

* Home screen
* Dashboard
* Profile screen
* Main navigation screen

The signed-in interface may display:

* User’s name
* User’s email address
* User’s profile picture
* Sign-out button

The application should not display the sign-in screen once a valid authenticated session exists.

---

# 🔄 Step 22: Maintain the User Session

Firebase Authentication maintains the user session on the device.

When the application is closed and opened again, the user will normally remain signed in.

When the application starts, it should check the current Firebase user.

Conceptually:

```text
Application starts
       ->
Check Firebase current user
        │
   ┌────┴────┐
   │         │
User exists  No user
   │         │
   >         >
Home screen  Sign-in screen
```

This prevents users from having to repeat Google authentication every time they open the application.

---

# 🚪 Step 23: Sign the User Out of Firebase

When the user selects **Sign Out**, the application must end the Firebase Authentication session.

Once Firebase sign-out is complete:

* The current Firebase user becomes unavailable
* Protected screens should no longer be accessible
* The application should return to the signed-out interface

The application’s UI should immediately reflect the authentication state.

---

# 🧹 Step 24: Clear the Credential State When Appropriate

Signing out of Firebase clears the Firebase session, but Credential Manager may still remember information about the credential previously selected.

The credential state may also need to be cleared so that the next authentication attempt can present the correct account-selection experience.

This is particularly useful when:

* Multiple people use the same device
* The user wants to choose a different Google account
* The application should not automatically reuse the previous account

Google’s Credential Manager implementation guidance includes handling sign-in responses, errors and sign-out behaviour.

---

# ⚠️ Step 25: Handle Authentication Errors

Not every sign-in attempt will succeed.

The application should be prepared to handle situations such as:

* User cancels the sign-in process
* No Google account is available
* Internet connection is unavailable
* Invalid Web client ID
* Incorrect package name
* Missing SHA fingerprint
* Google provider is disabled
* Firebase configuration file is missing
* Token validation fails
* Credential Manager cannot find a valid credential

Errors should be shown using clear, user-friendly messages.

For example:

```text
Google sign-in was cancelled.
```

```text
Unable to sign in. Please check your internet connection and try again.
```

Detailed technical errors can be written to Logcat for debugging, but users should not be shown confusing exception messages.

---

# ⏳ Step 26: Handle the Loading State

Authentication is not always immediate.

While the application is waiting for Credential Manager or Firebase, it should display a loading state.

During this time, the application may:

* Display a progress indicator
* Disable the sign-in button
* Prevent repeated sign-in requests
* Display a message such as “Signing in…”

Once the process finishes, the loading state should be cleared whether authentication succeeds or fails.

This prevents the user from tapping the sign-in button multiple times and starting several authentication requests.

---

# 🔒 Step 27: Protect Authenticated Content

Authentication should control access to protected screens and data.

A user who is not signed in should not be able to access areas such as:

* Personal profile
* User dashboard
* Saved records
* Private files
* Administrative features

Firebase Security Rules can also use the authenticated user’s UID to restrict database and storage access.

Authentication in the user interface alone is not sufficient security. The backend rules must also enforce access restrictions.

---

# 🧪 Step 28: Test the Complete Authentication Flow

The authentication process should be tested under several conditions.

Important test scenarios include:

* First-time sign-in
* Returning user sign-in
* Multiple Google accounts
* User cancels account selection
* User has no suitable Google account
* Internet connection is unavailable
* User signs out
* User selects a different account
* Application is closed and reopened
* Firebase session is restored
* Invalid configuration is handled correctly

Testing only the successful case is not enough. Authentication frequently fails because of configuration issues rather than interface problems.

---

# 📲 Step 29: Configure the Release Version

The debug version and the release version of an Android application may use different signing certificates.

Before publishing the application, the release SHA fingerprints must also be registered with Firebase.

When Google Play App Signing is used, the certificate used by Google Play may differ from the local upload certificate.

The correct fingerprints must therefore be added to Firebase for the published version of the application.

After changing Firebase configuration, an updated `google-services.json` file may need to be downloaded and added to the project.

Without this step, Google authentication may work during development but fail after the application is released.

---

# 📊 Complete Authentication Flow

```text
Create Android Project
       ->
Create Firebase Project
       ->
Register Android App
       ->
Add SHA-1 and SHA-256 Fingerprints
       ->
Download google-services.json
       ->
Connect Firebase to Android
       ->
Enable Google Authentication
       ->
Obtain the Web Client ID
       ->
Add Authentication Libraries
       ->
Create Signed-Out Interface
       ->
User taps "Continue with Google"
       ->
Credential Manager starts
       ->
User selects a Google account
       ->
Google verifies the user
       ->
Google returns an ID token
       ->
Create Firebase credential
       ->
Firebase validates the token
       ->
Firebase creates or retrieves user
       ->
Application displays signed-in state
       ->
Firebase maintains the session
       ->
User signs out
```

---

# 🧩 Responsibilities of Each Component

| Component                      | Main responsibility                              |
| ------------------------------ | ------------------------------------------------ |
| Android application            | Starts sign-in and updates the interface         |
| Credential Manager             | Requests and returns available credentials       |
| Google                         | Verifies the user’s Google identity              |
| Google ID token                | Proves that Google authenticated the user        |
| Firebase Authentication        | Verifies the token and manages the app session   |
| Firestore or Realtime Database | Stores additional application-specific user data |
| Firebase Security Rules        | Restrict access to authenticated data            |

---

# 💡 Key Points to Remember

* Google verifies the user’s identity.
* Credential Manager handles the credential-selection process on Android.
* Firebase validates the Google token and manages the application session.
* The Android application never receives the user’s Google password.
* The package name registered in Firebase must match the Android project exactly.
* SHA fingerprints help Google verify the identity of the application.
* The Web client ID is used when requesting a Google ID token.
* A Google login and a Firebase login are connected but are not the same step.
* Firebase Authentication stores authentication information, not the application’s complete user profile.
* The Firebase UID should be used to identify the user in Firebase databases.
* Authentication errors and cancellations must be handled properly.
* Release certificate fingerprints must be configured before publishing the application.
