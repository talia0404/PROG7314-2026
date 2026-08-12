# 🚗 ParkSmart — Full-Stack Android Integration

Welcome to **ParkSmart**!

In this activity, you will build a full-stack Android application that allows users to **find parking, check availability, reserve parking bays and use a QR code to check in**.

Unlike the previous demonstrations, this activity does not focus on one isolated Android feature.

Instead, you will bring together several technologies and concepts to build **one complete system**.

---

# 📚 What Are We Building?

ParkSmart is a parking reservation system.

A **driver** should be able to:

- Sign in using their Google account.
- View available parking areas.
- View parking bays.
- Select a date and time.
- Check whether parking is available.
- Reserve an available parking bay.
- View their upcoming and previous reservations.
- Cancel an eligible reservation.
- Receive a QR parking pass for a confirmed reservation.

A **manager** should be able to:

- Manage parking areas.
- Manage parking bays.
- Scan a driver's QR parking pass.
- Validate a reservation.
- Check a driver into a parking facility.
- Complete a parking session.

---

# 🎯 What Is the Goal of This Activity?

The goal is **not simply to build a parking application**.

ParkSmart is designed to help you understand how the different parts of a modern mobile application work together.

By the end of the activity, you should understand how to connect:

```text
Android Application
        ->
Firebase Authentication
        ->
Retrofit
        ->
Node.js + Express REST API
        ->
MongoDB
```

You will also work with:

- Google Sign-In
- Firebase ID tokens
- Firebase Admin
- REST APIs
- Retrofit
- Repository architecture
- ViewModels
- Jetpack Compose
- MongoDB
- Mongoose
- Authentication
- Authorisation
- Business-rule validation
- QR-code generation
- QR-code scanning
- Testing

---

# 🧠 This Is Not a Follow-Along Demo

The previous demonstrations introduced individual technologies and features.

For example:

```text
Google SSO
Biometrics
Notifications
Maps
REST APIs
```

ParkSmart is different.

You will be given:

- Requirements
- Architecture guidance
- Implementation instructions
- Business rules
- Security requirements
- Testing guidance
- Helpful resources

You will then need to **design and implement parts of the solution yourself**.

> 💡 There may be more than one correct way to implement a requirement. Your solution should be logical, secure, maintainable and meet the requirements provided.

---

# 🏗️ System Architecture

ParkSmart consists of three main parts.

## 📱 Android Application

The Android application is responsible for:

- Displaying the user interface.
- Collecting user input.
- Signing users in.
- Sending requests to the API.
- Displaying information returned by the API.
- Managing UI state.
- Displaying QR parking passes.
- Scanning QR parking passes.

The Android application will use **Kotlin and Jetpack Compose**.

---

## 🌐 REST API

The Node.js + Express API sits between the Android application and the database.

It is responsible for:

- Receiving requests from Android.
- Authenticating users.
- Authorising actions.
- Validating incoming data.
- Applying business rules.
- Reading and modifying database information.
- Preventing invalid reservations.
- Validating QR parking passes.
- Returning appropriate HTTP responses.

---

## 🍃 MongoDB

MongoDB stores the application's persistent data.

This includes information such as:

```text
Users
Parking Areas
Parking Bays
Reservations
```

Mongoose will be used to define schemas, validation rules, relationships and indexes.

---

# 🔄 How the Components Communicate

A typical ParkSmart request will follow this structure:

```text
User
  ->
Compose Screen
  ->
ViewModel
  ->
Repository
  ->
Retrofit
  ->
Express API
  ->
Mongoose
  ->
MongoDB
```

The response then travels back through the application:

```text
MongoDB
  ->
Express API
  ->
Retrofit
  ->
Repository
  ->
ViewModel
  ->
Compose UI
```

This separation is important.

Your Compose screens should **not directly communicate with MongoDB**.

---

# 🔐 Authentication Architecture

ParkSmart uses Firebase Authentication to establish the identity of the user.

The authentication flow will eventually look similar to:

```text
Google Sign-In
      ->
Firebase Authentication
      ->
Firebase ID Token
      ->
Android sends token to API
      ->
Firebase Admin verifies token
      ->
API identifies authenticated user
      ->
ParkSmart profile and role retrieved
```

The API must not simply trust a user ID sent by Android.

The server must establish the user's identity from a **verified Firebase ID token**.

---

# 👥 ParkSmart Roles

ParkSmart contains two main roles.

## 🚘 Driver

A driver uses the application to find and reserve parking.

Typical driver actions include:

```text
View Parking Areas
        ->
Check Availability
        ->
Select Parking Bay
        ->
Create Reservation
        ->
View QR Parking Pass
```

## 🧑‍💼 Manager

A manager is responsible for managing parking facilities and processing parking sessions.

Typical manager actions include:

```text
Manage Parking Areas
Manage Parking Bays
Scan QR Pass
Validate Reservation
Check In Driver
Complete Parking Session
```

Authentication answers:

> **Who is this user?**

Authorisation answers:

> **What is this user allowed to do?**

You will work with both concepts throughout ParkSmart.

---

# 🗺️ How You Will Build ParkSmart

Do **not** try to build the entire application at once.

ParkSmart has been divided into **six development steps**.

Complete and test each step before moving to the next one.

---

## 🏗️ Step 01 — Project and Backend Setup

You will prepare the foundation of ParkSmart.

You will:

- Create the required projects.
- Set up the Node.js + Express API.
- Organise the backend structure.
- Configure environment variables.
- Configure `.gitignore`.
- Connect the API to MongoDB.
- Introduce Mongoose.
- Create the initial database models.
- Confirm that the API can communicate with MongoDB.

### ✅ End Goal

Your Android project exists, your API runs successfully and your API can communicate with MongoDB.

---

## 🔐 Step 02 — Authentication and Users

You will connect Google Sign-In, Firebase Authentication and the ParkSmart API.

You will:

- Configure Firebase.
- Implement Google Sign-In.
- Authenticate users with Firebase.
- Retrieve Firebase ID tokens.
- Configure Firebase Admin in the API.
- Verify Firebase tokens.
- Protect API routes.
- Create ParkSmart user profiles.
- Synchronise Firebase users with MongoDB.
- Introduce driver and manager roles.

### ✅ End Goal

A user can sign in and the ParkSmart API can securely determine **who the user is and what role they have**.

---

## 📡 Step 03 — Android API Integration

You will connect the Android application to your REST API.

You will work with:

```text
Compose
   ->
ViewModel
   ->
Repository
   ->
Retrofit
   ->
ParkSmart API
```

You will:

- Configure Retrofit.
- Create API service interfaces.
- Create the required data classes/DTOs.
- Introduce repositories.
- Introduce ViewModels.
- Manage UI state.
- Handle loading, success and error states.
- Send authenticated requests to protected API endpoints.

### ✅ End Goal

Your Android application can securely send requests to and receive responses from the ParkSmart API.

---

## 🅿️ Step 04 — Parking Management

You will build the main parking data used by ParkSmart.

You will work with:

```text
Parking Areas
      ->
Parking Bays
```

You will:

- Create parking areas.
- Create parking bays.
- Define relationships between them.
- Display parking information in Android.
- Implement manager-only operations.
- Apply validation rules.
- Prevent invalid or duplicate data.
- Work with MongoDB relationships and indexes.

### ✅ End Goal

ParkSmart contains parking facilities and bays that can be managed through the API and displayed in Android.

---

## 📅 Step 05 — Availability and Reservations

You will implement the main business logic of ParkSmart.

The basic flow will be:

```text
Choose Parking Area
        ->
Choose Date and Time
        ->
Check Availability
        ->
Select Available Bay
        ->
Create Reservation
```

You will:

- Search for available parking bays.
- Work with dates and times.
- Validate operating hours.
- Detect reservation conflicts.
- Prevent overlapping reservations.
- Create reservations.
- Retrieve a user's reservations.
- Display upcoming and previous reservations.
- Cancel eligible reservations.
- Enforce reservation ownership.

### ✅ End Goal

A driver can find an available parking bay, reserve it and manage their reservations.

---

## 🎟️ Step 06 — QR Passes and Check-In

You will complete the ParkSmart parking journey.

You will:

- Generate secure QR tokens.
- Associate QR tokens with reservations.
- Display QR parking passes in Android.
- Scan QR codes.
- Send scanned tokens to the API.
- Validate QR tokens.
- Validate reservation status.
- Prevent duplicate check-ins.
- Check drivers in.
- Complete parking sessions.

The final process will resemble:

```text
Confirmed Reservation
        ->
QR Parking Pass
        ->
Manager Scans Pass
        ->
API Validates Reservation
        ->
Driver Checks In
        ->
Parking Session
        ->
Session Completed
```

### ✅ End Goal

ParkSmart supports the complete journey from **sign-in -> reservation -> QR pass -> check-in -> completion**.

---

# 📂 Activity Structure

Each step has its own folder.

```text
ParkSmart/
|
|-- README.md
|
|-- Step 01 - Project and Backend Setup/
|   |-- README.md
|
|-- Step 02 - Authentication and Users/
|   |-- README.md
|
|-- Step 03 - Android API Integration/
|   |-- README.md
|
|-- Step 04 - Parking Management/
|   |-- README.md
|
|-- Step 05 - Availability and Reservations/
|   |-- README.md
|
|-- Step 06 - QR Passes and Check-In/
|   |-- README.md
```

Start with **Step 01** and work through the folders in order.

> ⚠️ **Do not skip ahead.** Later steps rely on functionality created during earlier steps.

---

# 📖 What You Will Find in Each Step

The exact content will differ depending on the topic, but each step will generally contain:

### 📚 What You Are Building

An explanation of the feature and how it fits into ParkSmart.

### 🎯 Learning Objectives

What you should understand after completing the step.

### 🧠 How It Works

An explanation of the architecture, concepts and application flow.

### 🛠️ Implementation Instructions

Detailed guidance explaining what you need to create and configure.

### 🔒 Security and Business Rules

Rules that must be enforced by your application and API.

### 🧪 Testing

How to test whether your implementation works correctly.

### 🐛 Common Problems

Common errors and areas to check when something is not working.

### 📚 Helpful Resources

Official documentation and other useful resources relating specifically to that step.

### ✅ Before Moving On

A checklist that must be completed before starting the next step.

---

# 📌 How to Read the Requirements

Throughout the activity, different terms indicate how important a requirement is.

| Term | Meaning |
|---|---|
| **Must** | Required. Your implementation needs to meet this requirement. |
| **Should** | Strongly recommended unless you have a justified alternative. |
| **May** | Optional. |
| **Suggested** | An example approach. Another suitable implementation may be used. |

When an exact implementation is not provided, you are expected to use the requirements and resources to determine an appropriate solution.

---

# 🔒 An Important Security Rule

Throughout ParkSmart, remember:

> **The Android application is not the final security boundary.**

Android can:

- Validate input for usability.
- Hide manager functionality from drivers.
- Check whether fields are empty.
- Display available parking bays.
- Prevent obviously invalid actions.

However, a client application can be modified or bypassed.

Therefore, important rules must also be enforced by the **API**.

For example:

```text
Android:
"Only managers can see this button."

                    NOT ENOUGH

API:
"Is the authenticated user actually a manager?"

                    REQUIRED
```

The API must enforce rules relating to:

- Authentication
- Authorisation
- Ownership
- Reservation availability
- Reservation conflicts
- QR validity
- Check-in status
- Valid state transitions

---

# 🧪 Test as You Build

Do not wait until the entire application is finished before testing it.

The recommended development cycle is:

```text
Implement
   ->
Run
   ->
Test
   ->
Fix
   ->
Test Again
   ->
Commit
   ->
Continue
```

Each step includes a **Before Moving On** checklist.

Do not continue simply because the application compiles.

Make sure the feature actually behaves correctly.

---

# 💾 Commit Your Progress

Commit your work regularly.

Good commits describe a meaningful change.

For example:

```text
Add Firebase authentication middleware

Create parking area model

Connect Android to parking API

Add reservation conflict validation

Implement QR check-in
```

Avoid relying on one large commit after completing the entire application.

---

# ⚠️ Before You Start

Keep the following in mind throughout the activity:

- Do not hardcode passwords or private credentials.
- Do not commit `.env` files or private keys.
- Do not trust user IDs supplied by the Android client.
- Do not place all application logic inside Compose screens.
- Do not allow Android to be the only place where important business rules are checked.
- Do not ignore unsuccessful API responses.
- Do not assume that a request is valid simply because it came from your Android application.
- Test both successful and unsuccessful scenarios.

---

# 🚀 Ready to Start?

You are not expected to know how to implement every ParkSmart feature immediately.

That is the purpose of the activity.

Work through the application **one step at a time**, use the provided resources when you need them and make sure you understand why each component is required.

Your development journey is:

```text
01 Project + Backend
        ->
02 Authentication + Users
        ->
03 Android + API
        ->
04 Parking Management
        ->
05 Availability + Reservations
        ->
06 QR Passes + Check-In
```

Once **Step 01** is working, move on to the next step.

## ➡️ Start with: `Step 01 - Project and Backend Setup`
