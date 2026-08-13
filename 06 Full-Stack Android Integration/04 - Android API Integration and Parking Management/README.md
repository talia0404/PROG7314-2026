
# 📡 Step 04 — Android API Integration and Parking Management

## 📚 What You Are Building

In the previous steps, you created:

```text
Android Application

and

Express API
-> MongoDB
```

You also added:

```text
Google Sign-In
-> Firebase Authentication
-> Firebase ID Token
-> Firebase Admin Verification
-> ParkSmart User Profile
```

The next goal is to connect the Android application properly to the ParkSmart REST API.

You will also use that connection to build the first complete ParkSmart feature:

```text
Parking Areas
-> Parking Bays
```

By the end of this step, the Android application should be able to:

- Connect to the local ParkSmart API.
- Send HTTP requests using Retrofit.
- Convert JSON responses into Kotlin objects.
- Send authenticated requests.
- Use a Repository to separate networking from the UI.
- Use ViewModels to manage screen state.
- Display loading, success, empty and error states.
- Retrieve parking areas from the API.
- Retrieve parking bays for a selected parking area.
- Allow managers to create and update parking information through protected API endpoints.
- Prevent drivers from performing manager-only operations.

The main Android architecture introduced in this step is:

```text
Compose Screen
-> ViewModel
-> Repository
-> Retrofit Service
-> ParkSmart API
```

---

# 🎯 Learning Objectives

After completing this step, you should be able to:

- Explain why Android should not call the database directly.
- Explain the role of Retrofit.
- Explain the purpose of a Retrofit service interface.
- Explain the difference between a network DTO and UI state.
- Explain the purpose of a Repository.
- Explain the purpose of a ViewModel.
- Explain why network requests should not be placed directly inside composables.
- Configure Android networking permissions.
- Connect an Android emulator to an API running on the development computer.
- Send GET, POST, PUT and PATCH requests using Retrofit.
- Attach a Firebase ID token to protected requests.
- Handle successful and unsuccessful HTTP responses.
- Represent loading, success, empty and error states in the UI.
- Retrieve ParkingArea data from the ParkSmart API.
- Retrieve ParkingBay data belonging to a ParkingArea.
- Build manager-only parking management functionality.
- Explain why API authorisation is still required even when Android hides manager controls.

---

# 🧠 1. Understand the New Architecture

Your Android application should not become one large file containing:

```text
Compose UI
+
Firebase
+
Retrofit
+
API logic
+
Error handling
+
Business rules
```

Instead, responsibilities should be separated.

ParkSmart will use an architecture similar to:

```text
Compose Screen
-> ViewModel
-> Repository
-> Retrofit
-> Express API
```

Each layer has a different responsibility.

---

# 🎨 2. Compose Screen

The Compose screen is responsible for the user interface.

It should:

- Display information.
- Collect user input.
- Display loading indicators.
- Display error messages.
- Display lists.
- Respond to user actions.
- Send events to the ViewModel.

For example:

```text
User taps Refresh
-> Screen tells ViewModel
```

The screen should not contain all of the networking logic itself.

Avoid creating code where a button directly performs an entire HTTP request and processes the result inside the composable.

---

# 🧠 3. ViewModel

The ViewModel manages state required by the UI.

For ParkSmart, a ViewModel may eventually manage information such as:

```text
Parking areas

Parking bays

Loading state

Error message

Selected parking area
```

The ViewModel communicates with the Repository.

Conceptually:

```text
Compose
-> ViewModel
-> Repository
```

The ViewModel should not need to know detailed Retrofit configuration.

### 📚 Helpful Resources

**Android — ViewModel Overview:**  
https://developer.android.com/topic/libraries/architecture/viewmodel

**Android — UI Layer:**  
https://developer.android.com/topic/architecture/ui-layer

**Android — UI State Production:**  
https://developer.android.com/topic/architecture/ui-layer/state-production

---

# 📦 4. Repository

The Repository belongs to the application's data layer.

Its purpose is to provide data to the rest of the application without forcing the UI to understand exactly where the data came from.

For ParkSmart:

```text
ViewModel
-> Repository
-> Retrofit Service
```

A Repository may contain operations conceptually related to:

```text
Get Parking Areas

Get Parking Bays

Create Parking Area

Update Parking Area

Create Parking Bay

Update Parking Bay
```

Android's architecture guidance recommends using repositories as the entry point to the data layer.

### 📚 Helpful Resources

**Android — Guide to App Architecture:**  
https://developer.android.com/topic/architecture

**Android — Data Layer:**  
https://developer.android.com/topic/architecture/data-layer

**Android — Architecture Recommendations:**  
https://developer.android.com/topic/architecture/recommendations

---

# 🌐 5. Retrofit

Retrofit is the HTTP client abstraction you will use to represent the ParkSmart REST API inside Android.

Retrofit allows you to describe endpoints using an interface.

Instead of manually constructing every HTTP request, you describe the API using annotations such as:

```text
@GET

@POST

@PUT

@PATCH

@DELETE

@Path

@Query

@Body

@Header
```

Conceptually:

```text
Retrofit Interface
-> HTTP Request
-> ParkSmart API
```

### 📚 Helpful Resources

**Retrofit — Official Documentation:**  
https://square.github.io/retrofit/

**Retrofit — API Declarations:**  
https://square.github.io/retrofit/declarations/

**Retrofit GitHub:**  
https://github.com/square/retrofit

---

# 📦 6. Add the Required Android Dependencies

Open:

```text
app/build.gradle.kts
```

Add Retrofit and a JSON converter.

Using Retrofit 3.0.0:

```kotlin
implementation("com.squareup.retrofit2:retrofit:3.0.0")
implementation("com.squareup.retrofit2:converter-gson:3.0.0")
```

You will also need ViewModel support for Compose.

Use the current stable Android Lifecycle/ViewModel dependency recommended by the Android documentation for your project version.

For example, add the appropriate:

```text
lifecycle-viewmodel-compose
```

dependency from the official Lifecycle release page.

Do not blindly copy an old version from an unrelated tutorial.

### 📚 Helpful Resources

**Retrofit GitHub:**  
https://github.com/square/retrofit

**Android Lifecycle Releases:**  
https://developer.android.com/jetpack/androidx/releases/lifecycle

**Compose and Other Libraries:**  
https://developer.android.com/develop/ui/compose/libraries

After adding dependencies:

```text
File
-> Sync Project with Gradle Files
```

Do not continue until Gradle Sync succeeds.

---

# 🌐 7. Add Android Internet Permission

Open:

```text
app/src/main/AndroidManifest.xml
```

Add:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

You may also add:

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

if your application needs to inspect the current network state.

Your manifest permissions belong above the:

```xml
<application>
```

element.

### Why?

Android applications need permission to perform network operations.

### 📚 Helpful Resource

**Android — Connect to the Network:**  
https://developer.android.com/develop/connectivity/network-ops/connecting

---

# 🖥️ 8. Understand `localhost`

Your ParkSmart API currently runs on your development computer.

For example:

```text
http://localhost:3000
```

When you open that address in the browser on your computer:

```text
localhost
```

means:

```text
this computer
```

However, an Android emulator is a separate virtual device.

Inside the emulator:

```text
localhost
```

means:

```text
the emulator itself
```

It does **not** mean your development computer.

---

# 🔗 9. Use `10.0.2.2` from the Android Emulator

The Android emulator provides a special address that allows the emulator to communicate with the host development computer.

Use:

```text
10.0.2.2
```

instead of:

```text
localhost
```

Therefore, if the API runs on:

```text
http://localhost:3000/
```

on your computer, Android should use:

```text
http://10.0.2.2:3000/
```

as the local development base URL.

### 📚 Helpful Resource

**Android — Emulator Networking:**  
https://developer.android.com/studio/run/emulator-networking

---

# 📱 10. Physical Device Networking

If you use a physical Android phone instead of the emulator, the same:

```text
10.0.2.2
```

address will not work.

The physical device must be able to reach your development computer across the network.

You may need to use the development computer's local network IP address.

For example:

```text
http://192.168.x.x:3000/
```

The exact address depends on your network.

The phone and development computer will normally need to be able to communicate over the same local network.

You may also need to consider:

- Windows Firewall.
- Network profile settings.
- The address the Node server is listening on.

For classroom testing, the Android emulator is usually easier to configure consistently.

---

# ⚠️ 11. Local HTTP Development

Your local development API currently uses:

```text
http
```

rather than:

```text
https
```

Modern Android versions restrict cleartext HTTP traffic in many situations.

For development, you may need to configure a Network Security Configuration that permits cleartext communication with your local development host.

Do not simply enable insecure HTTP communication for every domain in a production application.

---

# 🔐 12. Create a Development Network Security Configuration

Inside:

```text
app/src/main/res/
```

create a folder if necessary:

```text
xml/
```

Then create:

```text
network_security_config.xml
```

A development configuration can allow cleartext traffic specifically for the host you are using.

Research the Android Network Security Configuration documentation and configure your development environment appropriately.

Then reference the configuration from the `<application>` element in:

```text
AndroidManifest.xml
```

using:

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

> ⚠️ Configure this only as broadly as necessary for local development.

### 📚 Helpful Resource

**Android — Network Security Configuration:**  
https://developer.android.com/privacy-and-security/security-config

---

# 📁 13. Create the Android Package Structure

Do not place every ParkSmart class inside:

```text
MainActivity.kt
```

Create a clear package structure.

A suitable structure is:

```text
com.yourpackage.parksmart
|
|-- data/
|   |
|   |-- model/
|   |
|   |-- remote/
|   |
|   |-- repository/
|
|-- ui/
|   |
|   |-- screens/
|   |
|   |-- components/
|   |
|   |-- theme/
|
|-- viewmodel/
|
|-- auth/
|
|-- navigation/
|
|-- MainActivity.kt
```

You may adapt this structure if you have a justified alternative.

The important requirement is that responsibilities remain separated.

---

# 📦 14. `data/model/`

This package will contain Kotlin classes representing information used by the application.

Examples may include:

```text
ParkingArea

ParkingBay

User
```

You may also choose to distinguish between:

```text
Network DTOs
```

and:

```text
Application/domain models
```

depending on how advanced you want your architecture to become.

---

# 🌐 15. `data/remote/`

This package should contain networking-related components.

Examples:

```text
ParkSmartApiService

RetrofitClient

Network configuration
```

This is where Retrofit should live.

---

# 📦 16. `data/repository/`

This package contains repositories.

For this step, you may create something similar to:

```text
ParkingRepository
```

The Repository becomes the main data-access interface used by the ViewModel.

---

# 🧠 17. `viewmodel/`

This package contains ViewModels.

A suitable ViewModel for this step might be responsible for:

```text
Parking Areas

Parking Bays

Loading state

Error state

Manager operations
```

---

# 🎨 18. `ui/screens/`

This package contains full Compose screens.

For this step, you will eventually require screens such as:

```text
ParkingAreasScreen

ParkingAreaDetailsScreen

ParkingBayManagementScreen
```

Your exact naming and screen structure may differ.

---

# 🧩 19. Plan the API Responses

Before writing Retrofit interfaces, inspect the JSON returned by your ParkSmart API.

Android data classes must match the structure of the data being returned.

For example, a Parking Area may contain information such as:

```text
_id

name

description

address

latitude

longitude

openingTime

closingTime

status

createdAt

updatedAt
```

A Parking Bay may contain:

```text
_id

parkingArea

bayNumber

floor

section

bayType

status

createdAt

updatedAt
```

Do not create Android fields randomly.

Use the API contract.

---

# 📋 20. Create Kotlin Data Classes

Inside:

```text
data/model/
```

create the Kotlin classes required to represent the Parking Area and Parking Bay data returned by the API.

You should decide:

- Which properties are required.
- Which properties may be nullable.
- Whether the MongoDB `_id` property should map directly to an Android property such as `id`.
- How nested/populated Parking Area data should be represented.
- Whether request objects should be separate from response objects.

You are expected to design these classes based on the API responses.

---

# 🧠 21. DTOs vs Application Models

A DTO is a:

```text
Data Transfer Object
```

Its purpose is to represent information being sent across a boundary.

For example:

```text
API JSON
-> Android DTO
```

A small project may use the same Kotlin class throughout the app.

A more structured application may separate:

```text
ParkingAreaDto
```

from:

```text
ParkingArea
```

Do not create unnecessary layers simply because they exist in larger enterprise applications.

Use enough structure to keep responsibilities clear.

---

# 🌐 22. Create the Retrofit Service Interface

Inside:

```text
data/remote/
```

create an interface for the ParkSmart API.

For example:

```text
ParkSmartApiService
```

Do not write HTTP requests manually inside each ViewModel.

The interface should represent the endpoints required by Android.

---

# 📥 23. Add Parking Area Endpoints

The service should represent API operations for retrieving parking information.

At minimum, Android should eventually be able to perform:

```text
GET /api/parking-areas

GET /api/parking-areas/:id

GET /api/parking-areas/:areaId/bays
```

You may also need manager operations such as:

```text
POST /api/parking-areas

PUT /api/parking-areas/:id

PATCH /api/parking-areas/:id/status

POST /api/parking-areas/:areaId/bays

PUT /api/bays/:bayId

PATCH /api/bays/:bayId/status
```

Use the Retrofit documentation to determine which annotations are appropriate.

### 📚 Helpful Resource

**Retrofit — API Declarations:**  
https://square.github.io/retrofit/declarations/

Focus on:

```text
@GET

@POST

@PUT

@PATCH

@Path

@Body

@Header
```

---

# 🧪 24. Make Sure the API Endpoints Exist First

Do not try to make Android call endpoints that have not been created.

Before connecting Android, test the required routes using:

```text
Postman

Bruno

Insomnia

or

Thunder Client
```

The recommended process is:

```text
Build API endpoint
-> Test endpoint independently
-> Confirm JSON response
-> Then connect Android
```

This makes debugging significantly easier.

If the endpoint already fails in Postman, Android is not the problem.

---

# 🏢 25. Create the Parking Area Backend Routes

You created the ParkingArea model in Step 02.

You now need proper ParkSmart routes rather than temporary database-testing routes.

Create appropriate backend route files inside:

```text
src/routes/
```

Your Parking Area routes should support the functionality required by ParkSmart.

At minimum, consider:

```text
Retrieve all visible parking areas

Retrieve one parking area

Create parking area

Update parking area

Change parking area status
```

Use meaningful REST endpoints.

---

# 🎮 26. Create Parking Area Controllers

Create controller functionality inside:

```text
src/controllers/
```

The controller should coordinate the request and response.

Do not place the entire implementation directly inside the route file.

Conceptually:

```text
Route
-> Controller
-> Model / Service
-> Response
```

For simple Parking Area operations, a controller may interact directly with a model.

For more complex business logic later, services will become more useful.

---

# ✅ 27. Apply Parking Area Validation

The API must enforce valid Parking Area data.

Remember the fields introduced previously:

```text
name

description

address

latitude

longitude

openingTime

closingTime

status
```

At minimum, validate:

- Required values.
- Latitude range.
- Longitude range.
- Allowed status values.
- Valid opening and closing times where appropriate.

Do not rely only on Android validation.

Android validation improves the user experience.

API validation protects the system.

---

# 👑 28. Protect Manager Parking Area Operations

Drivers should be able to retrieve parking information.

Drivers should **not** be able to create or modify parking areas.

Apply your authentication and role middleware from Step 03.

Conceptually:

```text
GET parking areas
-> authenticated driver or manager

POST parking area
-> manager only

PUT parking area
-> manager only

PATCH parking area status
-> manager only
```

Your exact access rules may vary according to your final design.

---

# 🚙 29. Create Proper Parking Bay Routes

Replace temporary Parking Bay testing logic with proper routes.

Required functionality should include:

```text
Retrieve bays for parking area

Create bay

Update bay

Change bay status
```

The API should enforce:

- Parking Area exists.
- Bay belongs to a valid Parking Area.
- Bay type is valid.
- Bay status is valid.
- Duplicate bay number within the same Parking Area is rejected.
- Driver cannot perform manager-only operations.

---

# 🧪 30. Test the Parking API Before Android

Using your API client, test:

### Parking Area Retrieval

```text
GET parking areas
```

Expected:

```text
Successful JSON response
```

### Parking Bay Retrieval

Select a real Parking Area ID.

Retrieve its bays.

### Driver Creates Parking Area

Expected:

```text
403 Forbidden
```

### Manager Creates Parking Area

Expected:

```text
Successful creation
```

### Invalid Parking Area Data

Expected:

```text
400 Bad Request
```

### Duplicate Parking Bay

Expected:

```text
Conflict or validation failure
```

Do not continue until the API behaves correctly without Android.

---

# 🔧 31. Create the Retrofit Instance

Inside:

```text
data/remote/
```

create the component responsible for constructing Retrofit.

It should configure:

- ParkSmart base URL.
- JSON converter.
- HTTP client if required.
- ParkSmart API service.

Your emulator development base URL should be similar to:

```text
http://10.0.2.2:3000/
```

The Retrofit base URL must end with:

```text
/
```

Keep networking configuration in one logical location rather than recreating Retrofit for every request.

---

# 🔐 32. Add Firebase Authentication to API Requests

Protected ParkSmart API routes require:

```text
Authorization: Bearer <firebase-id-token>
```

The Android application therefore needs a way to obtain a current Firebase ID token before calling protected endpoints.

You already implemented token retrieval in Step 03.

Now integrate it into your networking architecture.

Possible approaches include:

```text
Repository obtains token before protected request
```

or:

```text
OkHttp interceptor attaches token
```

You should research the approach that best suits your architecture.

For this activity, the important requirement is:

> Protected requests must send a current Firebase ID token.

---

# 🧩 33. Understand OkHttp

Retrofit uses OkHttp underneath for HTTP communication.

OkHttp can support features such as:

- Headers.
- Interceptors.
- Logging.
- Authentication handling.
- Request/response processing.

If you use an interceptor, it can centralise the process of attaching the:

```text
Authorization
```

header.

Do not manually duplicate token-header logic throughout every screen.

### 📚 Helpful Resources

**OkHttp:**  
https://square.github.io/okhttp/

**OkHttp — Interceptors:**  
https://square.github.io/okhttp/features/interceptors/

---

# 🔐 34. Do Not Log Full Authentication Tokens

During development it may be tempting to print every request and header.

Be careful.

Authentication headers may contain Firebase ID tokens.

Do not expose full tokens in:

```text
Logcat

GitHub

Screenshots

README files
```

If you use a network logging interceptor, review what information it logs.

---

# 📦 35. Create the Parking Repository

Create a Repository inside:

```text
data/repository/
```

For example:

```text
ParkingRepository
```

Its purpose is to provide operations required by the rest of the app.

Examples include:

```text
Get Parking Areas

Get Parking Bays

Create Parking Area

Update Parking Area

Create Parking Bay

Update Parking Bay
```

The Repository should call the Retrofit service.

The Compose screen should call the ViewModel.

---

# 🧠 36. Create the Parking ViewModel

Create an appropriate ViewModel.

The ViewModel should manage the UI state required for parking functionality.

For example:

```text
Parking areas

Selected parking area

Parking bays

Loading status

Error message
```

Do not simply create one Boolean called:

```text
isLoading
```

and one String for every possible application state if that becomes difficult to maintain.

Consider representing your screen state clearly.

---

# 🔄 37. Plan the UI States

At minimum, the Parking Areas screen should support:

```text
Loading

Success

Empty

Error
```

For example:

### Loading

```text
Loading parking areas...
```

### Success

Display the parking areas.

### Empty

```text
No parking areas are currently available.
```

### Error

```text
Unable to load parking areas.
```

The app should not show an empty white screen while waiting for the API.

---

# 🎨 38. Build the Parking Areas Screen

Create:

```text
ParkingAreasScreen
```

The screen should display active/visible parking facilities returned by the API.

Each item should display useful information.

Consider:

```text
Parking Area Name

Address

Operating Hours

Status
```

You may also display:

```text
Description
```

if appropriate.

Do not display every database field simply because it exists.

---

# 🖱️ 39. Allow the User to Select a Parking Area

When the user selects a Parking Area:

```text
Parking Areas
-> Selected Parking Area
-> Parking Area Details
```

The details screen should be able to identify which Parking Area was selected.

You may pass:

```text
Parking Area ID
```

between destinations rather than attempting to send an entire database object through navigation.

---

# 🧭 40. Add Navigation

If your application does not yet have navigation, configure Compose navigation.

Your application may eventually contain destinations such as:

```text
Home

Parking Areas

Parking Area Details

Manager Parking Management
```

Use the current Android navigation guidance appropriate for your Compose project.

### 📚 Helpful Resources

**Android — Navigation:**  
https://developer.android.com/guide/navigation

**Android — Navigation with Compose:**  
https://developer.android.com/develop/ui/compose/navigation

---

# 🏢 41. Create the Parking Area Details Screen

The details screen should display:

- Name.
- Description.
- Address.
- Operating hours.
- Status.
- Related Parking Bays.

The screen should request or receive the relevant Parking Area data through the appropriate ViewModel/Repository flow.

---

# 🚙 42. Display Parking Bays

Retrieve:

```text
GET /api/parking-areas/:areaId/bays
```

for the selected Parking Area.

Display the bays in a useful format.

Each bay should show information such as:

```text
Bay Number

Floor

Section

Bay Type

Status
```

Possible examples:

```text
A01
Ground Floor
Standard
Available
```

```text
EV02
Level 1
Electric
Maintenance
```

---

# 🎨 43. Make Statuses Easy to Understand

Users should be able to quickly distinguish:

```text
available

unavailable

maintenance
```

Do not rely only on colour.

Use text labels or icons as well.

This improves accessibility and makes the screen easier to understand.

---

# 👤 44. Driver Parking Experience

A driver should be able to:

```text
Open Parking Areas

-> Select Parking Area

-> View Details

-> View Parking Bays
```

Drivers do not need parking-management controls.

Do not display:

```text
Create Parking Area

Delete Parking Area

Create Parking Bay

Change Bay Status
```

to normal drivers.

---

# 👑 45. Manager Parking Experience

Managers need additional controls.

Depending on your UI design, managers may have:

```text
Add Parking Area

Edit Parking Area

Change Parking Area Status

Add Parking Bay

Edit Parking Bay

Change Parking Bay Status
```

The exact layout is your decision.

You may use:

- Buttons.
- Floating action buttons.
- Dialogues.
- Separate management screens.

The functionality must remain clear and usable.

---

# ⚠️ 46. Hiding Buttons Is Not Authorisation

If the Android application checks:

```text
role == manager
```

and shows manager controls only when true, that improves the interface.

However, someone can still manually call your API.

Therefore:

```text
Android hides manager button
```

and:

```text
API checks manager role
```

are both required.

The API remains responsible for final authorisation.

---

# 📝 47. Create Parking Area Input Forms

Managers need a way to enter Parking Area information.

The form should collect the required data.

For example:

```text
Name

Description

Address

Latitude

Longitude

Opening Time

Closing Time

Status
```

Apply suitable Android-side validation.

For example:

```text
Name cannot be blank

Latitude must be numeric

Longitude must be numeric
```

Do not assume Android validation is sufficient.

The API must validate again.

---

# 🚙 48. Create Parking Bay Input Forms

Managers should also be able to enter:

```text
Bay Number

Floor

Section

Bay Type

Status
```

The Parking Area relationship should not be typed manually by the user as a random MongoDB ID.

The application already knows which Parking Area is currently being managed.

Use that context appropriately.

---

# 🔄 49. Refresh the UI After Changes

After a successful manager operation, the Android interface should display the new server state.

For example:

```text
Manager creates bay
-> API confirms success
-> Android reloads or updates bay list
```

Do not show:

```text
Bay created successfully
```

before the API confirms it.

---

# ❌ 50. Handle Unsuccessful Manager Operations

Consider responses such as:

```text
400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict

500 Internal Server Error
```

Android should display useful messages.

Examples:

```text
You are not authorised to create parking areas.
```

```text
A bay with this number already exists in the selected parking area.
```

```text
Unable to connect to ParkSmart. Please try again.
```

Do not show a raw Node.js stack trace.

---

# 📡 51. Handle Network Failure

Test what happens if the API is not running.

For example:

```text
Stop Node API
-> Open Parking Areas screen
```

The application should not crash.

Display a useful error state.

For example:

```text
Unable to connect to ParkSmart.
```

Include a way to retry where appropriate.

---

# 🧪 52. Test the Complete Driver Flow

Test:

```text
Sign in as Driver
-> Open Parking Areas
-> Parking Areas load
-> Select one
-> Parking Details load
-> Parking Bays load
```

Confirm:

- Loading state displays.
- Successful data displays.
- Empty data is handled.
- API errors are handled.
- Manager controls are hidden.

---

# 🧪 53. Test the Complete Manager Flow

Test:

```text
Sign in as Manager
-> Open Parking Management
-> Create Parking Area
-> API confirms creation
-> Parking Area appears
-> Add Parking Bay
-> API confirms creation
-> Bay appears
```

Also test:

```text
Update Parking Area

Change Parking Area Status

Update Parking Bay

Change Parking Bay Status
```

---

# 🚫 54. Test Unauthorised Requests

Do not only test the UI.

Use Postman or another API client.

Call a manager endpoint using a valid driver's Firebase token.

Expected:

```text
403 Forbidden
```

This confirms that security exists on the API, not only in Android.

---

# 🧪 55. Test Invalid Parking Data

Try sending:

```text
Blank name

Invalid latitude

Invalid longitude

Invalid parking status

Invalid bay type

Duplicate bay number
```

Confirm that the API rejects invalid data.

Then confirm Android displays the API response appropriately.

---

# 📁 56. Review the Android Structure

Your Android project should now resemble something similar to:

```text
app/
|
|-- src/main/java/.../
|   |
|   |-- auth/
|   |
|   |-- data/
|   |   |
|   |   |-- model/
|   |   |
|   |   |-- remote/
|   |   |
|   |   |-- repository/
|   |
|   |-- ui/
|   |   |
|   |   |-- screens/
|   |   |
|   |   |-- components/
|   |   |
|   |   |-- theme/
|   |
|   |-- viewmodel/
|   |
|   |-- navigation/
|   |
|   |-- MainActivity.kt
```

Your exact names may differ.

Avoid structures such as:

```text
MainActivity.kt
-> authentication
-> Retrofit
-> every screen
-> every API call
-> every business rule
```

---

# 🧠 57. Understand the Complete Data Flow

When the Parking Areas screen opens:

```text
Compose Screen
-> ViewModel
-> Repository
-> Retrofit
-> Express API
-> MongoDB
```

The data returns:

```text
MongoDB
-> Express API
-> JSON
-> Retrofit
-> Repository
-> ViewModel
-> Compose Screen
```

This is the first complete ParkSmart full-stack feature.

---

# 🔒 Security Rules to Remember

## Android Does Not Connect Directly to MongoDB

Use:

```text
Android
-> API
-> MongoDB
```

---

## Protected Requests Require Authentication

Use:

```text
Authorization: Bearer <current-firebase-id-token>
```

---

## Manager Operations Require Server-Side Authorisation

Do not rely only on:

```text
role == manager
```

inside Compose.

---

## Do Not Log Full Tokens

Avoid exposing authentication information in development logs.

---

## Validate on Android and API

Android validation:

```text
Improves user experience
```

API validation:

```text
Protects application data
```

Both are useful.

---

# 🐛 Common Problems

## Android cannot connect to `localhost`

Do not use:

```text
http://localhost:3000/
```

inside the Android emulator.

Use:

```text
http://10.0.2.2:3000/
```

---

## `CLEARTEXT communication not permitted`

Your local API uses HTTP.

Configure an appropriate Android Network Security Configuration for local development.

See:

https://developer.android.com/privacy-and-security/security-config

---

## Retrofit says the base URL is invalid

Check that the base URL ends with:

```text
/
```

For example:

```text
http://10.0.2.2:3000/
```

---

## Android receives 404

Check:

- API route.
- HTTP method.
- Path parameter.
- Base URL.
- API server port.

Remember:

```text
/api/parking-areas
```

and:

```text
/api/parking-area
```

are different routes.

---

## Android receives 401

Check:

- User is signed in.
- Firebase token was retrieved.
- Authorization header exists.
- Header uses `Bearer`.
- Token is current.

---

## Android receives 403

The user is authenticated but may not have the required role.

Check:

```text
MongoDB User role
```

---

## JSON conversion fails

Compare:

```text
Actual API JSON
```

with:

```text
Android data class
```

Check:

- Field names.
- Nested objects.
- Nullability.
- Data types.

---

## Parking areas load in Postman but not Android

Check:

```text
Base URL

10.0.2.2

Internet permission

Network security configuration

Retrofit path

Android error logs
```

---

## `NetworkOnMainThreadException`

Do not perform blocking network operations directly on the main UI thread.

Use the asynchronous/coroutine-friendly networking approach appropriate for Retrofit and your ViewModel architecture.

---

## Duplicate parking bays are still created

This is a backend/database rule.

Check the compound uniqueness rule created previously for:

```text
parkingArea + bayNumber
```

Android cannot guarantee uniqueness.

---

# 📚 Helpful Resources

## 🏗️ Android Architecture

**Android — Guide to App Architecture:**  
https://developer.android.com/topic/architecture

**Android — Architecture Recommendations:**  
https://developer.android.com/topic/architecture/recommendations

**Android — Data Layer:**  
https://developer.android.com/topic/architecture/data-layer

**Android — UI Layer:**  
https://developer.android.com/topic/architecture/ui-layer

Focus on:

```text
UI layer

ViewModel

Data layer

Repository

Unidirectional data flow
```

---

## 🧠 ViewModel

**Android — ViewModel Overview:**  
https://developer.android.com/topic/libraries/architecture/viewmodel

Use this when deciding what state belongs outside your composables.

---

## 📡 Networking

**Android — Connect to the Network:**  
https://developer.android.com/develop/connectivity/network-ops/connecting

**Android — Network Security Configuration:**  
https://developer.android.com/privacy-and-security/security-config

Use these resources for:

```text
INTERNET permission

Secure networking

Local HTTP development
```

---

## 🖥️ Android Emulator Networking

**Android — Emulator Networking:**  
https://developer.android.com/studio/run/emulator-networking

Use this to understand why:

```text
10.0.2.2
```

is used instead of:

```text
localhost
```

from the Android emulator.

---

## 📡 Retrofit

**Retrofit:**  
https://square.github.io/retrofit/

**Retrofit — API Declarations:**  
https://square.github.io/retrofit/declarations/

**Retrofit GitHub:**  
https://github.com/square/retrofit

Focus on:

```text
Retrofit.Builder

baseUrl

ConverterFactory

@GET

@POST

@PUT

@PATCH

@Path

@Query

@Body

@Header
```

---

## 🌐 OkHttp

**OkHttp:**  
https://square.github.io/okhttp/

**OkHttp — Interceptors:**  
https://square.github.io/okhttp/features/interceptors/

Useful when researching how to apply headers consistently to API requests.

---

## 🧭 Navigation

**Android — Navigation:**  
https://developer.android.com/guide/navigation

**Android — Navigation with Compose:**  
https://developer.android.com/develop/ui/compose/navigation

Use this when creating navigation between:

```text
Parking Areas

Parking Details

Parking Management
```

---

## 🌐 Express

**Express — Routing:**  
https://expressjs.com/en/guide/routing.html

**Express — Using Middleware:**  
https://expressjs.com/en/guide/using-middleware.html

Use these when converting temporary parking test routes into proper ParkSmart endpoints.

---

## 📦 Mongoose

**Mongoose — Queries:**  
https://mongoosejs.com/docs/queries.html

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

**Mongoose — Validation:**  
https://mongoosejs.com/docs/validation.html

Use these resources when implementing Parking Area and Parking Bay API functionality.

---

# 🧪 Step 04 Testing Checklist

## 📡 Android Networking

- [ ] Retrofit is installed.
- [ ] JSON converter is installed.
- [ ] Internet permission is configured.
- [ ] Local development networking works.
- [ ] Android emulator uses `10.0.2.2`.
- [ ] Network Security Configuration is appropriate for local development.
- [ ] Retrofit has one clear base configuration.

---

## 🏗️ Android Architecture

- [ ] Networking is not implemented directly inside composables.
- [ ] Retrofit service interface exists.
- [ ] Repository exists.
- [ ] ViewModel exists.
- [ ] Parking UI state is clearly represented.
- [ ] Loading state works.
- [ ] Empty state works.
- [ ] Error state works.
- [ ] Successful state works.

---

## 🔐 Authentication

- [ ] Protected Android requests include a Firebase ID token.
- [ ] Missing authentication is handled.
- [ ] `401` responses are handled.
- [ ] `403` responses are handled.
- [ ] Full authentication tokens are not exposed in logs.

---

## 🏢 Parking Areas

- [ ] Proper Parking Area API routes exist.
- [ ] Temporary test routes have been removed.
- [ ] Android retrieves Parking Areas.
- [ ] Parking Areas display correctly.
- [ ] A selected Parking Area can be opened.
- [ ] Parking Area details display correctly.
- [ ] Invalid Parking Area data is rejected.
- [ ] Driver cannot create or modify Parking Areas.
- [ ] Manager can perform authorised Parking Area operations.

---

## 🚙 Parking Bays

- [ ] Proper Parking Bay API routes exist.
- [ ] Bays can be retrieved for a Parking Area.
- [ ] Parking Bays display in Android.
- [ ] Bay type displays correctly.
- [ ] Bay status displays correctly.
- [ ] Duplicate bay rules still work.
- [ ] Driver cannot create or modify bays.
- [ ] Manager can perform authorised bay operations.

---

## 🧪 Failure Testing

- [ ] API offline state has been tested.
- [ ] Invalid endpoint has been tested.
- [ ] Invalid authentication has been tested.
- [ ] Driver calling manager route has been tested.
- [ ] Invalid Parking Area data has been tested.
- [ ] Invalid Parking Bay data has been tested.
- [ ] Duplicate Parking Bay has been tested.

---

# ✅ Step 04 Complete

You started this step with:

```text
Android Application

and

Express API
-> MongoDB
```

You now have a complete Android-to-backend data flow:

```text
Compose
-> ViewModel
-> Repository
-> Retrofit
-> Express API
-> MongoDB
```

ParkSmart can now:

```text
Authenticate User
-> Load ParkSmart Profile
-> Retrieve Parking Areas
-> Retrieve Parking Bays
```

Managers can also manage the parking information through protected API operations.

However, a bay being marked:

```text
available
```

does not mean it is available at every date and time.

ParkSmart still needs to answer:

> Is this parking bay free during the driver's requested period?

That introduces the most important business logic in the application:

```text
Date and Time Selection
-> Availability
-> Reservation Conflict Detection
-> Reservation Creation
```

# ➡️ Continue to Step 05 — Availability and Reservations
