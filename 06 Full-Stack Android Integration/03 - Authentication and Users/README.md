# 🔐 Step 03 — Authentication and Users

## 📚 What You Are Building

In Step 02, you connected the ParkSmart API to MongoDB and created the first backend data models.

Your backend currently supports:

```text
Express API
-> Mongoose
-> MongoDB
```

You also created:

```text
ParkingArea
ParkingBay
```

ParkSmart now has a backend and somewhere to store parking information.

The next problem is:

> Who is using ParkSmart, and how can the API trust that the user really is who they claim to be?

In this step, you will introduce:

```text
Google Sign-In
-> Firebase Authentication
-> Firebase ID Token
-> ParkSmart API
-> Firebase Admin
-> ParkSmart User Profile
```

You will also introduce two ParkSmart roles:

```text
driver
manager
```

By the end of this step:

- A user should be able to sign in using Google.
- Firebase should authenticate the user.
- Android should be able to obtain a Firebase ID token.
- The ParkSmart API should be able to verify that token.
- A ParkSmart user profile should be created in MongoDB.
- Returning users should not receive duplicate profiles.
- New users should receive the `driver` role by default.
- Protected API requests should reject users who are not authenticated.
- Manager-only operations should be distinguishable from normal driver operations.

---

# 🎯 Learning Objectives

After completing this step, you should be able to:

- Explain the difference between authentication and authorisation.
- Explain the role of Firebase Authentication.
- Explain why Google Sign-In alone is not enough to secure the ParkSmart API.
- Configure an Android application with Firebase.
- Enable Google as an authentication provider.
- Understand the role of Credential Manager.
- Explain what a Firebase ID token is.
- Retrieve a Firebase ID token after authentication.
- Explain why an ID token should be sent to the backend.
- Configure Firebase Admin in a Node.js API.
- Verify Firebase ID tokens server-side.
- Explain why the API should trust the verified token instead of user information supplied in a request body.
- Create a ParkSmart User model.
- Synchronise authenticated Firebase users with MongoDB.
- Explain the difference between a `driver` and `manager`.
- Protect routes based on authentication.
- Understand the basics of role-based authorisation.

---

# 🧠 1. Authentication vs Authorisation

These two concepts are related, but they are not the same.

## Authentication

Authentication answers:

> Who are you?

For ParkSmart, the user will authenticate using their Google account.

The flow begins:

```text
User
-> Google Sign-In
-> Firebase Authentication
```

Firebase then establishes the user's identity.

---

## Authorisation

Authorisation answers:

> What are you allowed to do?

For example:

```text
Driver
-> View parking areas
-> Make reservations
-> View own reservations
```

A manager may additionally:

```text
Manager
-> Create parking areas
-> Manage parking bays
-> Scan QR passes
-> Check drivers in
```

A user can therefore be:

```text
Authenticated
```

but still:

```text
Not authorised
```

to perform a particular action.

For example:

```text
Signed-in driver
-> attempts manager-only operation
-> API rejects request
```

---

# 🔥 2. What Is Firebase Authentication?

Firebase Authentication provides identity and sign-in services for applications.

ParkSmart will use:

```text
Google Sign-In
+
Firebase Authentication
```

Firebase will be responsible for establishing the identity of the signed-in user.

After successful authentication, Firebase provides information such as:

```text
Firebase UID
Email
Display name
Authentication state
```

The Firebase UID is especially important.

It is a unique identifier associated with the authenticated Firebase user.

### 📚 Helpful Resources

**Firebase Authentication:**  
https://firebase.google.com/docs/auth

**Firebase — Authenticate with Google on Android:**  
https://firebase.google.com/docs/auth/android/google-signin

Firebase's current Android Google sign-in guide uses Credential Manager for the sign-in flow. :contentReference[oaicite:0]{index=0}

---

# 🧩 3. Understand the Authentication Architecture

ParkSmart should **not** rely on Android simply telling the API who the user is.

This would be unsafe:

```text
Android sends:

userId = "123"
role = "manager"

API trusts it
```

A modified application or manually constructed request could send false information.

Instead, ParkSmart will use:

```text
Google Sign-In
-> Firebase Authentication
-> Firebase ID Token
-> Android sends token to API
-> Firebase Admin verifies token
-> API receives trusted UID
```

The API can then use the trusted UID to find the corresponding ParkSmart user in MongoDB.

---

# 🎫 4. What Is a Firebase ID Token?

After Firebase authenticates a user, the Android application can obtain a Firebase ID token.

The token is proof that Firebase has authenticated the user.

Conceptually:

```text
User signs in
-> Firebase authenticates user
-> Firebase issues ID token
```

Android can then include this token when calling protected ParkSmart API endpoints.

The token should be sent using the HTTP `Authorization` header.

The standard format is:

```text
Authorization: Bearer <firebase-id-token>
```

The word:

```text
Bearer
```

indicates that the request contains a bearer token.

Do not place the ID token inside every request body unless there is a specific reason to do so.

---

# 🔐 5. Why Must the API Verify the Token?

The ParkSmart API exists outside the Android application.

It cannot simply assume:

```text
The Android app says the user is signed in
-> therefore the user must be signed in
```

The API must independently verify the token.

The complete process becomes:

```text
Android
-> Firebase ID Token
-> Express API
-> Firebase Admin
-> Token verified
-> Trusted Firebase UID
```

Firebase Admin's token-verification functionality can verify and decode a valid Firebase ID token and expose the associated `uid`. :contentReference[oaicite:1]{index=1}

### 📚 Helpful Resource

**Firebase — Verify ID Tokens:**  
https://firebase.google.com/docs/auth/admin/verify-id-tokens

Read this resource carefully.

It is one of the most important resources in this step.

---

# 📱 6. Create a Firebase Project

Open:

https://console.firebase.google.com/

Create a new Firebase project for ParkSmart.

Use a meaningful project name.

For example:

```text
ParkSmart
```

or:

```text
ParkSmart-PROG7314
```

You do not need to enable every Firebase product.

For this step, the important feature is:

```text
Firebase Authentication
```

---

# 🤖 7. Register the Android Application with Firebase

Inside the Firebase project:

1. Add a new application.
2. Choose **Android**.
3. Enter the exact Android package name used by your ParkSmart Android project.
4. Complete the Android app registration.

The package name must match the Android application's actual package/application ID.

For example:

```text
com.yourname.parksmart
```

and:

```text
com.yourname.parksmartdemo
```

are different applications.

Do not guess the package name.

Check the Android project configuration.

---

# 📄 8. Download `google-services.json`

Firebase will provide:

```text
google-services.json
```

Download this file.

Place it inside the Android application's:

```text
app/
```

module.

The structure should resemble:

```text
ParkSmartAndroid/
|
|-- app/
|   |
|   |-- google-services.json
|   |
|   |-- build.gradle.kts
|   |
|   |-- src/
```

Do not place it in:

```text
src/main/
```

Do not rename the file.

---

# 🔐 9. Treat Firebase Configuration Carefully

`google-services.json` contains Firebase project configuration.

It is not the same thing as a Firebase Admin private key, but you should still follow the repository policy provided for your module or organisation.

Never commit:

```text
Firebase Admin service-account private keys
```

to a public repository.

Those credentials provide privileged server access and are much more sensitive.

---

# 🧰 10. Add Firebase to the Android Project

Follow Firebase's current Android setup instructions.

You will need to:

1. Add the Google Services Gradle plugin.
2. Apply the plugin to the Android application module.
3. Add Firebase Authentication.
4. Add the required Credential Manager / Google identity dependencies.
5. Sync the Gradle project.

Because Firebase and Android library versions change, use the official documentation to obtain the current dependency versions rather than copying old versions from an unrelated tutorial.

### 📚 Helpful Resources

**Firebase — Add Firebase to Android:**  
https://firebase.google.com/docs/android/setup

**Firebase — Authenticate with Google on Android:**  
https://firebase.google.com/docs/auth/android/google-signin

**Android — Implement Sign in with Google:**  
https://developer.android.com/identity/sign-in/credential-manager-siwg-implementation

The current Android Sign in with Google guide covers adding dependencies, creating Credential Manager, building sign-in flows, handling responses, errors and sign-out. :contentReference[oaicite:2]{index=2}

---

# 🪪 11. Understand Credential Manager

Credential Manager provides Android's modern credential interface.

ParkSmart will use it as part of the Google Sign-In flow.

Its job is to help Android interact with available credentials.

Conceptually:

```text
ParkSmart
-> Credential Manager
-> Google account selection
-> Google credential
-> Firebase Authentication
```

Credential Manager itself is not the ParkSmart user database.

It helps with sign-in.

### 📚 Helpful Resources

**Android — About Credential Manager:**  
https://developer.android.com/identity/credential-manager

**Android — Implement Sign in with Google:**  
https://developer.android.com/identity/sign-in/credential-manager-siwg-implementation

Android describes Credential Manager as its unified API for credential-based sign-in experiences. :contentReference[oaicite:3]{index=3}

---

# 🔑 12. Generate the Android SHA-1 Certificate Fingerprint

Google authentication configuration commonly requires your application's signing certificate information.

Open the Android Studio terminal in your ParkSmart Android project.

Run:

```powershell
.\gradlew signingReport
```

Look for the:

```text
debug
```

variant.

Find:

```text
SHA1
```

Copy the complete SHA-1 value.

Make sure you use:

```text
signingReport
```

not:

```text
singingReport
```

The Gradle task is about **signing**, not karaoke.

---

# 🔥 13. Add the SHA-1 to Firebase

Open:

```text
Firebase Console
-> Project Settings
-> Your Android App
```

Add the debug SHA-1 certificate fingerprint.

Save the configuration.

If Firebase instructs you to download an updated:

```text
google-services.json
```

replace the previous version inside:

```text
app/
```

with the new file.

---

# 🔓 14. Enable Google Sign-In

Inside Firebase Console, open:

```text
Authentication
-> Sign-in method
```

Enable:

```text
Google
```

as a sign-in provider.

Save the configuration.

Firebase's Android Google authentication guide requires enabling the Google provider before the app can authenticate users with Google. :contentReference[oaicite:4]{index=4}

---

# 🌐 15. Locate the Web Client ID

The Google authentication flow requires the appropriate OAuth web client ID.

Locate the web client ID associated with your Firebase/Google project.

Depending on the current Firebase interface, you may find it through the Firebase project configuration or associated Google Cloud OAuth credentials.

Do not confuse:

```text
Android client ID
```

with:

```text
Web client ID
```

The Google ID token configuration used with Firebase sign-in needs the correct server/web client ID.

Follow the current Firebase Google Sign-In documentation.

### 📚 Resource

https://firebase.google.com/docs/auth/android/google-signin

---

# 📱 16. Plan the Android Authentication State

Before implementing Google Sign-In, decide what states the interface needs.

At minimum, ParkSmart should support:

```text
Signed Out
Signed In
Loading
Authentication Error
```

A signed-out screen could display:

```text
ParkSmart

Sign in to continue.

[ Sign in with Google ]
```

A signed-in screen could display:

```text
Welcome

Display Name
Email

[ Continue ]

[ Sign Out ]
```

Do not place all authentication logic directly inside your composable functions.

The UI should display state and send user actions to the appropriate logic layer.

---

# 🛠️ 17. Implement Google Sign-In in Android

Using the Firebase and Android resources, implement the Google Sign-In flow.

Your implementation needs to:

1. Create or access Credential Manager.
2. Configure the Google credential request.
3. Use the correct web client ID.
4. Launch the credential request.
5. Handle the selected Google credential.
6. Obtain the Google ID token.
7. Convert the Google credential into a Firebase credential.
8. Authenticate with Firebase.
9. Update the Android authentication state.
10. Handle sign-in cancellation.
11. Handle sign-in errors.
12. Support sign-out.

You are expected to research the required classes and methods.

Do not copy a complete `MainActivity` from an unrelated project.

### 📚 Resources

**Firebase — Authenticate with Google on Android:**  
https://firebase.google.com/docs/auth/android/google-signin

**Android — Implement Sign in with Google:**  
https://developer.android.com/identity/sign-in/credential-manager-siwg-implementation

**Credential Manager Troubleshooting:**  
https://developer.android.com/identity/sign-in/credential-manager-troubleshooting-guide

---

# 🧪 18. Test Google Sign-In Before Continuing

Before working on the backend token verification, confirm that Android authentication works independently.

Test:

### Successful Sign-In

Expected:

```text
Tap Sign In
-> Google account selector appears
-> Select account
-> Firebase authenticates user
-> App shows signed-in state
```

### Cancelled Sign-In

Cancel the account selection.

The application should not crash.

### Sign-Out

Sign in, then sign out.

Confirm that the signed-out UI appears again.

### App Restart

Sign in.

Close and reopen the application.

Observe Firebase's current authentication state.

You should understand whether:

```text
FirebaseAuth.currentUser
```

contains an authenticated user.

### 📚 Resource

**Firebase — Manage Users on Android:**  
https://firebase.google.com/docs/auth/android/manage-users

Firebase documents retrieving the currently signed-in Firebase user through its Android authentication APIs. :contentReference[oaicite:5]{index=5}

---

# 🎫 19. Retrieve a Firebase ID Token

Once Firebase Authentication works, your Android application needs to obtain a Firebase ID token for the signed-in user.

Your task is to research how the current Firebase user can request an ID token.

The operation is asynchronous.

Your implementation should:

1. Confirm that a Firebase user exists.
2. Request the user's ID token.
3. Handle a successful token result.
4. Handle failure.
5. Avoid assuming token retrieval always succeeds.

For development purposes, you may log a short confirmation that a token was obtained.

Do **not** routinely print complete authentication tokens into production logs.

### 📚 Helpful Resource

**Firebase — Verify ID Tokens:**  
https://firebase.google.com/docs/auth/admin/verify-id-tokens

This page explains both obtaining an ID token on the client and verifying it on the backend.

---

# 🧠 20. Understand Token Lifetime

Do not treat one Firebase ID token as a permanent password.

Tokens have a limited lifetime and may be refreshed.

The Android application should obtain a current token when it needs to make authenticated requests rather than permanently hardcoding or storing one token forever.

Later, your Retrofit/Repository architecture will be responsible for attaching valid authentication information to protected requests.

For now, you only need to prove that Android can retrieve an ID token.

---

# 🟢 21. Install Firebase Admin in the API

Now move to:

```text
ParkSmart/api/
```

Open the VS Code terminal.

Install the Firebase Admin SDK:

```powershell
npm install firebase-admin
```

Firebase Admin is intended for trusted server environments.

It must **not** be installed and used as an administrative credential inside your Android application.

### 📚 Helpful Resources

**Firebase Admin SDK:**  
https://firebase.google.com/docs/admin/setup

**Firebase Admin Authentication:**  
https://firebase.google.com/docs/auth/admin

**Firebase Admin Node.js Reference:**  
https://firebase.google.com/docs/reference/admin/node

The Firebase Admin SDK is specifically intended for privileged environments such as backend servers. :contentReference[oaicite:6]{index=6}

---

# 🔐 22. Configure Firebase Admin Credentials

The API must have permission to communicate with Firebase as a trusted server.

Follow Firebase's Admin SDK setup documentation to configure the server credentials.

Depending on your environment, Firebase may support different credential strategies.

For local development, your lecturer may instruct you to use a Firebase service account.

If you use a downloaded service-account credential file:

> ⚠️ Treat it as a private secret.

Never commit it to GitHub.

Never place it in:

```text
README.md
```

Never place it inside:

```text
android/
```

Never send it to another user.

The Android application does **not** need the Firebase Admin private credential.

### 📚 Resource

**Firebase — Add the Admin SDK to Your Server:**  
https://firebase.google.com/docs/admin/setup

---

# 📁 23. Create Firebase Admin Configuration

You already have:

```text
src/config/
```

Create a Firebase configuration file inside this folder.

For example:

```text
firebaseAdmin.js
```

This file should be responsible for:

1. Importing the Firebase Admin functionality you need.
2. Loading the trusted Firebase configuration.
3. Initialising Firebase Admin.
4. Exporting the authentication functionality required by the rest of the backend.
5. Preventing unnecessary repeated initialisation.

You are expected to research the current Firebase Admin Node.js API and use the approach recommended by Firebase.

Do not copy an outdated Firebase Admin initialisation pattern without checking the current documentation.

---

# 🚫 24. Protect Firebase Admin Credentials

Update:

```text
.gitignore
```

to make sure any local service-account credential file is ignored.

Do not use a generic filename if it increases the chance that you accidentally commit a secret.

Also check:

```powershell
git status
```

before committing.

If a private key file appears in Git status, stop and fix the problem before continuing.

---

# 🛡️ 25. Create Authentication Middleware

You already created:

```text
src/middleware/
```

during Step 01.

Create a file for Firebase authentication middleware.

For example:

```text
authenticateFirebaseToken.js
```

The middleware's responsibility is:

```text
Request
-> Read Authorization header
-> Extract Bearer token
-> Verify token with Firebase Admin
-> Obtain trusted UID
-> Attach trusted identity to request
-> Continue
```

If authentication fails:

```text
Request
-> Reject
```

---

# 📥 26. Read the `Authorization` Header

Protected API requests will use:

```text
Authorization: Bearer <firebase-id-token>
```

Your middleware should check:

- Does the header exist?
- Is it in the expected format?
- Does it contain a token?
- Is the token valid?

Do not accept a user identity merely because Android sends:

```json
{
  "firebaseUid": "something"
}
```

The trusted identity must come from the verified token.

---

# ✅ 27. Verify the Firebase ID Token

Use Firebase Admin to verify the extracted ID token.

When verification succeeds, Firebase returns decoded token information.

Your middleware should obtain the trusted Firebase UID from that result.

The middleware should then attach the trusted authenticated identity to the Express request using a clearly named custom property.

Later route handlers will use this trusted value.

Firebase Admin's token-verification method checks the format, signature and expiry of the ID token before returning decoded token information. :contentReference[oaicite:7]{index=7}

### 📚 Resource

**Firebase — Verify ID Tokens:**  
https://firebase.google.com/docs/auth/admin/verify-id-tokens

Focus on:

```text
Authorization header
Bearer token
verifyIdToken()
decoded token
uid
```

---

# 🚨 28. Handle Authentication Failures

Your middleware should respond appropriately when:

### No Token Is Supplied

For example:

```text
Authorization header missing
```

Use an appropriate:

```text
401 Unauthorized
```

response.

### Header Format Is Invalid

For example:

```text
Authorization: banana
```

Reject the request.

### Token Is Invalid

Reject the request.

### Token Is Expired or Cannot Be Verified

Reject the request.

Do not expose Firebase stack traces or private server details in the response sent to Android.

A user-friendly API response might contain a consistent message field.

---

# 🧪 29. Create a Protected Test Endpoint

Before introducing ParkSmart users, prove that the middleware works.

Create a temporary or development test route such as:

```text
GET /api/protected-test
```

The endpoint should:

1. Use the Firebase authentication middleware.
2. Only execute if the token is valid.
3. Return a simple confirmation.
4. Optionally include non-sensitive trusted identity information for development testing.

Do not put business logic into this route.

The purpose is only to prove:

```text
Android token
-> Express
-> Firebase Admin verification
-> Protected route
```

---

# 🧪 30. Test the Protected Endpoint Without a Token

Using Postman, Bruno, Insomnia or another API client:

Send:

```text
GET /api/protected-test
```

without an `Authorization` header.

Expected result:

```text
401 Unauthorized
```

If the endpoint succeeds without a token, your route is not properly protected.

---

# 🧪 31. Test with an Invalid Token

Add an Authorization header containing something deliberately invalid.

For example:

```text
Authorization: Bearer definitely-not-a-valid-token
```

Expected result:

```text
401 Unauthorized
```

The API should not crash.

---

# 🧪 32. Test with a Valid Firebase Token

Obtain a current Firebase ID token from the authenticated Android application for development testing.

Use your API client.

Add:

```text
Authorization
```

with the value:

```text
Bearer <your-current-token>
```

Send the request.

Expected flow:

```text
API Client
-> Authorization header
-> Express middleware
-> Firebase Admin
-> Token verified
-> Protected endpoint
-> Successful response
```

> ⚠️ Do not publish, commit or share the token. Treat it as temporary authentication information.

---

# 👤 33. Create the ParkSmart User Model

You can now create:

```text
src/models/User.js
```

The User model connects:

```text
Firebase identity
```

to:

```text
ParkSmart application data
```

Firebase Authentication stores authentication identity.

ParkSmart's MongoDB User document stores application-specific information.

---

# 🧠 34. Why Do We Need a Separate ParkSmart User?

Firebase knows things such as:

```text
UID
Email
Display name
Authentication provider
```

But ParkSmart needs additional information such as:

```text
Role
ParkSmart profile creation date
Application-specific information
```

Do not try to turn Firebase Authentication into your ParkSmart application database.

The two systems have different responsibilities.

Conceptually:

```text
Firebase
-> Identity

MongoDB User
-> ParkSmart profile
```

---

# 📋 35. Plan the User Model

Create fields for at least:

```text
firebaseUid
email
displayName
role
createdAt
updatedAt
```

You should decide:

- Appropriate field types.
- Which values are required.
- Which value must be unique.
- The default role.
- Allowed roles.

---

# 👥 36. Define the ParkSmart Roles

ParkSmart will use:

```text
driver
manager
```

A newly created user must automatically receive:

```text
driver
```

unless a trusted administrative process assigns another role.

Android must not be allowed to create a new user and submit:

```text
role = manager
```

as though that should be trusted.

The role must be controlled by the backend/database.

---

# 🛡️ 37. Protect the Role Field

Use schema validation to restrict the allowed values.

Research:

```text
enum
```

validation in Mongoose.

The role field should not accept arbitrary values such as:

```text
superAdmin
CEO
owner
parkingGod
```

unless those values are formally part of your system.

### 📚 Resource

**Mongoose — Validation:**  
https://mongoosejs.com/docs/validation.html

---

# 🔑 38. Make `firebaseUid` Unique

Each Firebase identity should map to only one ParkSmart user.

Therefore:

```text
firebaseUid
```

must be unique.

Conceptually:

```text
Firebase UID ABC
-> one ParkSmart User
```

not:

```text
Firebase UID ABC
-> User 1
-> User 2
-> User 3
```

Use an appropriate unique database/index rule.

### 📚 Resources

**MongoDB — Unique Indexes:**  
https://www.mongodb.com/docs/manual/core/index-unique/

**Mongoose — Indexes:**  
https://mongoosejs.com/docs/guide.html#indexes

---

# 🕒 39. Add Timestamps

Enable Mongoose timestamps on the User schema.

ParkSmart should automatically have:

```text
createdAt
updatedAt
```

Do not ask Android to supply trusted creation timestamps.

### 📚 Resource

**Mongoose — Timestamps:**  
https://mongoosejs.com/docs/timestamps.html

---

# 🔄 40. Create User Synchronisation

ParkSmart needs a way to connect a successfully authenticated Firebase user to a MongoDB profile.

Create an authenticated user synchronisation endpoint.

A suitable route is:

```text
POST /api/users/sync
```

This endpoint must require Firebase authentication.

---

# 🧠 41. Understand User Synchronisation

When a user signs in successfully for the first time:

```text
Firebase user exists
-> ParkSmart User does not exist
-> Create ParkSmart User
```

When the same user signs in again:

```text
Firebase user exists
-> ParkSmart User already exists
-> Return existing ParkSmart User
```

The API must not create a new MongoDB profile every time the user opens the app.

---

# 🛠️ 42. Implement the Synchronisation Endpoint

Your endpoint should:

1. Require Firebase authentication middleware.
2. Read the trusted Firebase UID from the verified token.
3. Use trusted Firebase token information where appropriate.
4. Search MongoDB for the matching ParkSmart User.
5. If the user exists, return the existing profile.
6. If the user does not exist, create a new profile.
7. Assign `driver` as the default role.
8. Return the ParkSmart profile.

Do not trust:

```text
firebaseUid
role
```

from the Android request body.

---

# 🚫 43. Do Not Trust This

An Android request like:

```json
{
  "firebaseUid": "abc123",
  "email": "student@example.com",
  "role": "manager"
}
```

must not be treated as proof of identity or authority.

The API should derive the identity from:

```text
Verified Firebase ID Token
```

The API should derive the role from:

```text
MongoDB ParkSmart User
```

---

# 🧪 44. Test First-Time User Synchronisation

Sign in with a Google account.

Obtain a valid Firebase ID token.

Call:

```text
POST /api/users/sync
```

with the appropriate authentication header.

For a user who does not yet exist, confirm that:

- One MongoDB User document is created.
- `firebaseUid` matches the authenticated Firebase identity.
- The correct email/display information is stored.
- `role` is `driver`.
- Timestamps exist.

Inspect the User document using:

```text
MongoDB Compass
```

or:

```text
MongoDB Atlas
```

---

# 🔁 45. Test Returning User Synchronisation

Call the synchronisation endpoint again with the same authenticated Firebase user.

Confirm:

```text
Existing user found
-> Existing profile returned
```

Check the MongoDB collection.

There should still be:

```text
one
```

User document for that Firebase UID.

If you now have duplicates, fix the problem before continuing.

---

# ❌ 46. Test User Synchronisation Without Authentication

Call:

```text
POST /api/users/sync
```

without a Firebase token.

Expected:

```text
401 Unauthorized
```

The endpoint must not create a user.

---

# 👑 47. Introduce Manager Users

You now have two roles:

```text
driver
manager
```

New users are automatically:

```text
driver
```

For development/testing, your lecturer may instruct you to manually update a selected user's role in MongoDB Compass.

For example:

```text
driver
-> manager
```

This is acceptable as a development/testing technique.

The Android app must not include a:

```text
Make Me Manager
```

button.

---

# 🛡️ 48. Create Role Authorisation Middleware

Authentication answers:

```text
Who is this?
```

You now need another layer that can answer:

```text
Is this user allowed to perform this action?
```

Create reusable role-checking middleware.

It should eventually be able to protect manager-only endpoints.

Conceptually:

```text
Request
-> Verify Firebase Token
-> Find ParkSmart User
-> Check Role
-> Continue or Reject
```

---

# 🔎 49. Where Should the Role Come From?

The role must come from the trusted ParkSmart user profile stored in MongoDB.

Do not trust:

```text
role
```

inside an Android request.

This would be insecure:

```text
Android:
"I am a manager"

API:
"Seems legit"
```

Instead:

```text
Firebase token
-> trusted UID
-> MongoDB User
-> trusted role
```

---

# 🚫 50. Use the Correct HTTP Status

Consider:

### User Is Not Authenticated

Use:

```text
401 Unauthorized
```

### User Is Authenticated but Lacks Permission

Use:

```text
403 Forbidden
```

These represent different situations.

```text
401
-> Who are you?

403
-> I know who you are, but you may not do this.
```

---

# 🧪 51. Create a Temporary Manager Test Endpoint

Create a temporary/development endpoint for testing manager authorisation.

For example:

```text
GET /api/manager-test
```

It should require:

```text
Firebase Authentication
+
Manager Role
```

The endpoint exists only to prove that your role protection works.

You will replace this with actual manager functionality later.

---

# 🧪 52. Test as a Driver

Sign in as a user whose ParkSmart role is:

```text
driver
```

Send an authenticated request to the manager test route.

Expected:

```text
403 Forbidden
```

---

# 🧪 53. Test as a Manager

Update your authorised test user's role to:

```text
manager
```

Sign in again or refresh the ParkSmart profile as needed.

Call the manager endpoint.

Expected:

```text
Successful response
```

You have now demonstrated:

```text
Authentication
+
Authorisation
```

---

# 📱 54. Return the ParkSmart Profile to Android

After authentication and synchronisation, Android needs to know the user's ParkSmart profile.

The profile response should contain appropriate application information such as:

```text
Display name
Email
Role
```

Android can then decide which interface elements to display.

For example:

```text
driver
-> Driver Home

manager
-> Manager features available
```

However:

> Hiding manager UI is a usability feature, not the final security control.

The API must still enforce manager permissions.

---

# 🎨 55. Plan Role-Based Android UI

Do not build every ParkSmart screen yet.

For now, prove that Android can display the authenticated user's role.

For example:

```text
Welcome, Student Name

Role:
Driver
```

A manager account could show:

```text
Role:
Manager
```

You may conditionally display a simple manager indicator or temporary test button.

Do not build the parking-management feature yet.

That comes later.

---

# 🚪 56. Implement Sign-Out Properly

The Android application must support sign-out.

Signing out should clear the relevant Firebase authentication state and any local ParkSmart user state.

After sign-out:

```text
Signed-in UI
-> Sign out
-> Signed-out UI
```

Do not continue displaying the previous user's ParkSmart information after Firebase has signed them out.

### 📚 Resource

**Android — Sign in with Google Implementation:**  
https://developer.android.com/identity/sign-in/credential-manager-siwg-implementation

The current guide includes sign-out as part of the Sign in with Google implementation flow. :contentReference[oaicite:8]{index=8}

---

# 🔁 57. Consider Application Restart Behaviour

Test this sequence:

```text
Sign in
-> Close application
-> Reopen application
```

Determine:

- Is a Firebase user still available?
- Does your Android state correctly reflect this?
- Do you need to synchronise/reload the ParkSmart profile?
- Does the role display correctly?

Your application should not permanently assume that the previous screen state is still correct.

---

# 📁 58. Review Your Backend Structure

Your API should now contain something similar to:

```text
api/
|
|-- src/
|   |
|   |-- config/
|   |   |
|   |   |-- database.js
|   |   |
|   |   |-- firebaseAdmin.js
|   |
|   |-- controllers/
|   |
|   |-- middleware/
|   |   |
|   |   |-- authenticateFirebaseToken.js
|   |   |
|   |   |-- requireRole.js
|   |
|   |-- models/
|   |   |
|   |   |-- ParkingArea.js
|   |   |
|   |   |-- ParkingBay.js
|   |   |
|   |   |-- User.js
|   |
|   |-- routes/
|   |
|   |-- services/
|   |
|   |-- utils/
|   |
|   |-- app.js
|   |
|   |-- server.js
|
|-- .env
|
|-- .env.example
|
|-- package.json
```

Your exact filenames may differ.

The important point is that responsibilities remain separated.

---

# 🧠 59. Understand the Complete Authentication Flow

You should now be able to explain this process:

```text
User taps Google Sign-In
-> Credential Manager obtains Google credential
-> Firebase authenticates user
-> Android obtains Firebase ID token
-> Android sends token to ParkSmart API
-> API authentication middleware reads token
-> Firebase Admin verifies token
-> API obtains trusted Firebase UID
-> MongoDB User profile is found or created
-> ParkSmart role is returned
-> Android updates UI
```

If you cannot explain why each step exists, review the section before continuing.

---

# 🔒 60. Security Rules to Remember

## Android Must Not Be Trusted for Identity

Do not trust:

```text
firebaseUid
```

sent in a normal request body.

Use:

```text
Verified Firebase token
```

---

## Android Must Not Be Trusted for Roles

Do not trust:

```text
role = manager
```

from Android.

Use:

```text
MongoDB User role
```

---

## Firebase Admin Credentials Stay on the Server

Never place Firebase Admin credentials in Android.

Never commit them to GitHub.

---

## ID Tokens Are Temporary Credentials

Do not publish them.

Do not hardcode them.

Do not permanently store one development token and reuse it forever.

---

## UI Restrictions Are Not Security

This:

```text
Hide manager button from driver
```

is useful.

But this is required:

```text
API rejects driver calling manager endpoint
```

---

# 🐛 Common Problems

## Google account picker does not appear

Check:

- Credential Manager dependencies.
- Google ID dependency.
- Firebase configuration.
- Google authentication provider.
- Emulator/device Google Play support.
- Sign-in request configuration.

Use:

https://developer.android.com/identity/sign-in/credential-manager-troubleshooting-guide

---

## Google Sign-In succeeds but Firebase authentication fails

Check:

- Firebase project.
- Android package name.
- `google-services.json`.
- SHA-1.
- Web client ID.
- Google provider enabled in Firebase.
- Google token passed correctly to Firebase Authentication.

---

## Web client ID cannot be found

Check the Firebase project and associated Google Cloud OAuth configuration.

Make sure you are looking for the appropriate:

```text
Web client ID
```

and not only an Android OAuth client.

Refer to:

https://firebase.google.com/docs/auth/android/google-signin

---

## `google-services.json` appears wrong or outdated

If you changed Firebase Android configuration, such as adding signing certificates, download a fresh copy and replace the old file in:

```text
app/
```

---

## `FirebaseAuth.currentUser` is null

Possible reasons:

- User is not signed in.
- Authentication failed.
- Sign-out occurred.
- You are checking authentication state before the sign-in operation finishes.

Do not assume a user always exists.

---

## ID token retrieval fails

Check:

- Firebase user exists.
- Firebase authentication completed.
- Device has network access.
- Error callback/log details.

---

## API always returns `401`

Check:

- Header exists.
- Header is spelled `Authorization`.
- Value begins with `Bearer `.
- A space exists after `Bearer`.
- Token is current.
- Firebase Admin uses the correct Firebase project.
- The token was not truncated when copied for testing.

---

## Firebase Admin cannot verify the token

Check:

- Firebase Admin configuration.
- Project identity.
- Service credentials.
- Token belongs to the expected Firebase project.
- Server time is correct.
- Token is complete.

---

## Firebase Admin service-account file appears in Git

Stop before committing.

Add the file to:

```text
.gitignore
```

Then check:

```powershell
git status
```

If it was previously tracked, you must untrack it appropriately.

Do not simply rename the file and commit it under a different name.

---

## Duplicate MongoDB users appear

Check:

- `firebaseUid` uniqueness.
- Your synchronisation lookup.
- Whether you create before checking.
- Whether the same Firebase UID is being used consistently.

One Firebase identity should map to one ParkSmart profile.

---

## Driver can access manager endpoint

Check:

- Manager endpoint uses authentication middleware.
- Role middleware is actually applied.
- User is loaded from MongoDB.
- The role check uses the stored role.
- You did not trust a role sent by Android.

---

# 📚 Helpful Resources

## 🔥 Firebase Authentication

**Firebase Authentication Overview:**  
https://firebase.google.com/docs/auth

Use this for background on Firebase Authentication and supported identity providers. :contentReference[oaicite:9]{index=9}

**Firebase — Authenticate with Google on Android:**  
https://firebase.google.com/docs/auth/android/google-signin

This is the main resource for connecting Google Sign-In, Credential Manager and Firebase Authentication. :contentReference[oaicite:10]{index=10}

**Firebase — Manage Users on Android:**  
https://firebase.google.com/docs/auth/android/manage-users

Useful for current-user state and authenticated user information. :contentReference[oaicite:11]{index=11}

---

## 🔑 Credential Manager

**Android — About Credential Manager:**  
https://developer.android.com/identity/credential-manager

Use this to understand the role of Credential Manager. :contentReference[oaicite:12]{index=12}

**Android — About Sign in with Google:**  
https://developer.android.com/identity/sign-in/credential-manager-siwg

Provides an overview of Sign in with Google through Credential Manager. :contentReference[oaicite:13]{index=13}

**Android — Implement Sign in with Google:**  
https://developer.android.com/identity/sign-in/credential-manager-siwg-implementation

Use this while implementing the Android sign-in flow. :contentReference[oaicite:14]{index=14}

**Credential Manager Troubleshooting:**  
https://developer.android.com/identity/sign-in/credential-manager-troubleshooting-guide

Useful when Google account selection or credential retrieval fails. :contentReference[oaicite:15]{index=15}

---

## 🎫 Firebase ID Tokens

**Firebase — Verify ID Tokens:**  
https://firebase.google.com/docs/auth/admin/verify-id-tokens

This is essential for the ParkSmart backend.

Focus on:

```text
Client obtains Firebase ID token

Authorization header

verifyIdToken()

Decoded token

uid
```

Firebase Admin can verify and decode a properly signed, unexpired Firebase ID token and return its trusted UID. :contentReference[oaicite:16]{index=16}

---

## 🛡️ Firebase Admin

**Firebase — Add the Admin SDK:**  
https://firebase.google.com/docs/admin/setup

Use this when configuring Firebase Admin inside the ParkSmart Node.js API.

**Firebase — Admin Authentication:**  
https://firebase.google.com/docs/auth/admin

Useful for understanding server-side authentication responsibilities. :contentReference[oaicite:17]{index=17}

**Firebase Admin Node.js API Reference:**  
https://firebase.google.com/docs/reference/admin/node

Use this as a technical reference for the Node.js Admin SDK. :contentReference[oaicite:18]{index=18}

> The Firebase Admin SDK should run in a trusted server environment, not inside the Android client. :contentReference[oaicite:19]{index=19}

---

## 📦 Mongoose User Model

**Mongoose — Schemas:**  
https://mongoosejs.com/docs/guide.html

**Mongoose — Validation:**  
https://mongoosejs.com/docs/validation.html

**Mongoose — Timestamps:**  
https://mongoosejs.com/docs/timestamps.html

**MongoDB — Unique Indexes:**  
https://www.mongodb.com/docs/manual/core/index-unique/

Use these when building the ParkSmart User model.

---

## 🌐 HTTP Authentication

**MDN — Authorization Header:**  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization

Useful for understanding:

```text
Authorization: Bearer <token>
```

**MDN — HTTP Status Codes:**  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

Focus particularly on:

```text
401 Unauthorized

403 Forbidden
```

---

# 🧪 Step 03 Testing Checklist

## 📱 Android Authentication

- [ ] Firebase project exists.
- [ ] Android application is registered with Firebase.
- [ ] Package name matches.
- [ ] `google-services.json` is in the correct location.
- [ ] SHA-1 has been configured.
- [ ] Google authentication provider is enabled.
- [ ] Credential Manager dependencies are configured.
- [ ] Firebase Authentication dependency is configured.
- [ ] Google Sign-In opens.
- [ ] A Google account can be selected.
- [ ] Firebase authenticates the user.
- [ ] Signed-in user information is available.
- [ ] Cancelled sign-in is handled.
- [ ] Authentication errors are handled.
- [ ] Sign-out works.
- [ ] Authentication state is handled after application restart.

---

## 🎫 Firebase ID Token

- [ ] Android can retrieve a Firebase ID token.
- [ ] Token-retrieval failure is handled.
- [ ] Tokens are not hardcoded.
- [ ] Tokens are not committed or published.

---

## 🌐 API Authentication

- [ ] `firebase-admin` is installed.
- [ ] Firebase Admin is configured.
- [ ] Private Admin credentials are ignored by Git.
- [ ] Authentication middleware exists.
- [ ] Missing token returns `401`.
- [ ] Invalid token returns `401`.
- [ ] Valid token is successfully verified.
- [ ] Trusted UID comes from the decoded token.

---

## 👤 ParkSmart User

- [ ] `User.js` exists.
- [ ] `firebaseUid` is stored.
- [ ] `firebaseUid` is unique.
- [ ] Email is stored appropriately.
- [ ] Display name is stored appropriately.
- [ ] Role is stored.
- [ ] Allowed roles are restricted.
- [ ] Default role is `driver`.
- [ ] Timestamps are enabled.

---

## 🔄 Synchronisation

- [ ] `/api/users/sync` is protected.
- [ ] First authentication creates one ParkSmart User.
- [ ] Returning authentication returns the existing user.
- [ ] Repeated synchronisation does not create duplicates.
- [ ] The API does not trust a UID sent by Android.
- [ ] The API does not trust a role sent by Android.
- [ ] User profile and role can be returned to Android.

---

## 🛡️ Authorisation

- [ ] Role-checking middleware exists.
- [ ] Driver can access normal authenticated endpoints.
- [ ] Driver cannot access manager-only test endpoint.
- [ ] Driver receives `403` for manager-only action.
- [ ] Manager can access the manager-only test endpoint.
- [ ] Role is obtained from MongoDB.

---

# ✅ Step 03 Complete

You started this step with:

```text
Android Application

and

Express API
-> MongoDB
```

You now have a secure identity flow:

```text
Google Sign-In
-> Firebase Authentication
-> Firebase ID Token
-> Express API
-> Firebase Admin Verification
-> Trusted Firebase UID
-> MongoDB User
-> ParkSmart Role
```

ParkSmart can now answer:

> Who is this user?

and:

> What role does this user have?

However, Android is still not properly structured to communicate with all of the ParkSmart API endpoints.

In the next step, you will introduce the Android networking and architecture layer:

```text
Compose
-> ViewModel
-> Repository
-> Retrofit
-> ParkSmart API
```

You will then use this structure to retrieve and manage the parking data you created in Step 02.

# ➡️ Continue to Step 04 — Android API Integration and Parking Management
