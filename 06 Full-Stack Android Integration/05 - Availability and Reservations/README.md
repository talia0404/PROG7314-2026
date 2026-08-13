
# 📅 Step 05 — Availability and Reservations

## 📚 What You Are Building

In Step 04, you connected the Android application to the ParkSmart API and built the first complete full-stack feature.

You now have:

```text
Compose
-> ViewModel
-> Repository
-> Retrofit
-> Express API
-> MongoDB
```

ParkSmart can also:

```text
Load Parking Areas
-> Load Parking Bays
-> Display Parking Information
```

The next challenge is much more important.

A parking bay may be marked:

```text
available
```

but that does **not** mean it is available at every date and time.

For example:

```text
Bay A01

08:00 - 09:00
Available

10:00 - 12:00
Already Reserved

13:00 - 15:00
Available
```

ParkSmart therefore needs to understand **time-based availability**.

In this step, you will build the reservation system.

You will implement:

- Date selection.
- Start-time selection.
- End-time selection.
- Parking availability checks.
- Reservation conflict detection.
- Reservation creation.
- Reservation ownership.
- Reservation status.
- Viewing a driver's reservations.
- Cancelling eligible reservations.
- Handling reservation errors.

The full flow will eventually resemble:

```text
Driver selects Parking Area
-> Selects Date
-> Selects Start Time
-> Selects End Time
-> Android requests availability
-> API checks Parking Bays
-> API checks existing Reservations
-> Available bays returned
-> Driver selects Parking Bay
-> Android requests reservation
-> API checks availability again
-> Reservation created
```

The API must remain the final authority.

---

# 🎯 Learning Objectives

After completing this step, you should be able to:

- Explain the difference between general bay status and time-based availability.
- Explain why availability must be calculated.
- Work with dates and times in Android.
- Use date and time pickers in Jetpack Compose.
- Represent reservation start and end times.
- Send date/time values through a REST API.
- Create a Mongoose Reservation model.
- Explain the relationship between a User, ParkingArea, ParkingBay and Reservation.
- Detect overlapping reservation periods.
- Explain why Android must not be trusted to make the final availability decision.
- Create an availability endpoint.
- Create a reservation endpoint.
- Enforce reservation ownership.
- Prevent users from cancelling another user's reservation.
- Handle reservation statuses.
- Display upcoming, previous and cancelled reservations in Android.
- Handle `409 Conflict` responses appropriately.

---

# 🧠 1. Understand the Problem

At the moment, a ParkingBay contains a status such as:

```text
available

unavailable

maintenance
```

This tells ParkSmart whether the bay is generally usable.

For example:

```text
Bay A01
Status: available
```

means:

> This bay is currently enabled and may be considered for reservations.

It does **not** mean:

> This bay is free for every possible date and time.

---

# 📊 2. General Status vs Time-Based Availability

Consider:

```text
Bay A01
Status: available
```

An existing reservation may already use the bay:

```text
10:00 -> 12:00
```

Therefore:

```text
09:00 -> 10:00
Available
```

```text
10:00 -> 12:00
Unavailable
```

```text
12:00 -> 13:00
Available
```

The bay's stored status did not change.

What changed was:

```text
Requested reservation period
```

This means availability must be **calculated**.

---

# 🏗️ 3. The Availability Decision

To determine whether a bay is available, the API must consider more than one condition.

At minimum:

```text
Parking Area exists
-> Parking Area is active
-> Parking Bay exists
-> Parking Bay is available
-> Requested time is valid
-> Requested time is within operating hours
-> No conflicting reservation exists
```

Only after all of these checks succeed should the bay be considered available.

---

# 🚫 4. Do Not Store a Permanent Booking Availability Boolean

Avoid designing reservations around a field such as:

```text
isAvailable = true
```

inside the ParkingBay document.

That cannot accurately represent time-based availability.

For example:

```text
Bay A01

08:00 -> available
10:00 -> reserved
14:00 -> available
```

A single Boolean cannot represent all three states.

Use:

```text
ParkingBay status
+
Reservation data
```

to calculate availability.

---

# 📦 5. Create the Reservation Model

You deliberately did not create a Reservation model in Step 02.

Now the model has a purpose.

Create:

```text
src/models/Reservation.js
```

A reservation represents:

```text
One Driver
-> One Parking Area
-> One Parking Bay
-> One Start Date/Time
-> One End Date/Time
```

---

# 📋 6. Plan the Reservation Fields

Your Reservation model should contain at least:

```text
driver

parkingArea

parkingBay

startDateTime

endDateTime

status

reservedAt

checkedInAt

completedAt

qrToken

createdAt

updatedAt
```

Some of these fields will not be fully used until Step 06.

For example:

```text
qrToken

checkedInAt

completedAt
```

can initially remain unused or nullable.

Do not remove them simply because the current step does not use them yet.

---

# 🔗 7. Create the Reservation Relationships

A Reservation relates to several other documents.

Conceptually:

```text
Reservation
|
|-> driver -> User
|
|-> parkingArea -> ParkingArea
|
|-> parkingBay -> ParkingBay
```

Research how to use:

```text
Schema.Types.ObjectId

ref
```

with Mongoose.

You already used this concept when creating the relationship between:

```text
ParkingBay
-> ParkingArea
```

Now you are applying the same idea to a more complex model.

### 📚 Helpful Resources

**Mongoose — SchemaTypes:**  
https://mongoosejs.com/docs/schematypes.html

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

---

# 🚦 8. Define Reservation Statuses

Use statuses such as:

```text
reserved

cancelled

checked_in

completed

expired
```

For Step 05, the most important statuses are:

```text
reserved

cancelled
```

The remaining statuses will become more important during QR check-in.

---

## `reserved`

The reservation has been successfully created.

The driver has not yet checked in.

---

## `cancelled`

The driver or an authorised user cancelled the reservation.

The cancelled reservation should remain in the database for history/auditing rather than necessarily being permanently deleted.

---

## `checked_in`

The driver has arrived and their parking pass has been validated.

This will be introduced in Step 06.

---

## `completed`

The parking session has finished.

This will also be introduced in Step 06.

---

## `expired`

The reservation was not used within the permitted period.

You may introduce automatic expiry later depending on your final ParkSmart implementation.

---

# ✅ 9. Add Reservation Schema Validation

Use Mongoose validation where appropriate.

Consider:

```text
driver
-> required

parkingArea
-> required

parkingBay
-> required

startDateTime
-> required

endDateTime
-> required

status
-> restricted allowed values
```

Enable timestamps.

### 📚 Helpful Resources

**Mongoose — Validation:**  
https://mongoosejs.com/docs/validation.html

**Mongoose — Timestamps:**  
https://mongoosejs.com/docs/timestamps.html

Remember:

> Schema validation is not enough for all reservation rules.

For example:

```text
Does this reservation overlap another reservation?
```

requires application/business logic.

---

# 🕒 10. Use Real Date/Time Values

Do not store reservation dates and times as random formatted display strings if you need to compare them reliably.

For example, avoid building the entire backend around values such as:

```text
"10am Tuesday"
```

or:

```text
"1 September at two"
```

Your API/database needs date/time values that can be compared consistently.

MongoDB supports the BSON Date type, and Mongoose supports JavaScript `Date` values.

### 📚 Helpful Resources

**Mongoose — Dates:**  
https://mongoosejs.com/docs/tutorials/dates.html

**MongoDB — BSON Date:**  
https://www.mongodb.com/docs/manual/reference/bson-types/#date

---

# 🌍 11. Think About Time Zones

Dates and times are more complicated than simply:

```text
hour + minute
```

Your backend should use a consistent representation for stored instants.

Your Android interface can display dates/times in a format appropriate for the user.

For this classroom project, decide on one clear approach and document it.

Do not mix several time-zone assumptions throughout the app.

For example, if ParkSmart is intended for South African parking facilities, your UI may display local South African time while your backend stores a consistent timestamp representation.

The important requirement is consistency.

---

# 📱 12. Add Date Selection in Android

The driver needs to select the date of their parking reservation.

Jetpack Compose provides Material date picker components.

Your screen should allow the driver to select:

```text
Reservation Date
```

Avoid requiring the user to manually type a date into a plain text field unless you have a strong reason.

A date picker reduces formatting mistakes.

### 📚 Helpful Resource

**Android — Date Pickers in Compose:**  
https://developer.android.com/develop/ui/compose/components/datepickers

The current Compose documentation provides docked, modal and modal-input date picker approaches. :contentReference[oaicite:0]{index=0}

---

# ⏰ 13. Add Time Selection in Android

The driver also needs:

```text
Start Time

End Time
```

Use an appropriate Compose time picker or time input.

The interface should make it clear which value represents:

```text
Start
```

and which represents:

```text
End
```

### 📚 Helpful Resources

**Android — Time Pickers in Compose:**  
https://developer.android.com/develop/ui/compose/components/time-pickers

**Android — Time Picker Dialogs:**  
https://developer.android.com/develop/ui/compose/components/time-pickers-dialogs

Compose's current time picker APIs include `TimePicker` and `TimeInput`, both backed by `TimePickerState`. :contentReference[oaicite:1]{index=1}

---

# 🎨 14. Plan the Availability Screen

Create or adapt a screen that allows the driver to choose:

```text
Parking Area

Date

Start Time

End Time
```

Then provide an action such as:

```text
Check Availability
```

Do not immediately show every bay as available before the API has checked.

---

# ✅ 15. Apply Android-Side Input Validation

Before sending an availability request, Android should check obvious problems.

For example:

```text
Date not selected

Start time not selected

End time not selected

End time before start time
```

This improves the user experience.

However:

> The API must perform the same important validation again.

Android validation cannot be trusted as the only protection.

---

# 🌐 16. Create an Availability Endpoint

Create a backend endpoint for checking parking availability.

A suitable design is:

```text
GET /api/parking-areas/:areaId/availability
```

The requested period may be supplied using query parameters.

Conceptually:

```text
?start=<date-time>
&end=<date-time>
```

For example:

```text
GET /api/parking-areas/AREA_ID/availability?start=...&end=...
```

Your exact date/time format should be documented and used consistently.

---

# 🔎 17. Understand Query Parameters

Query parameters are values supplied after:

```text
?
```

in a URL.

Conceptually:

```text
/resource?name=value
```

Multiple parameters can be separated using:

```text
&
```

For ParkSmart:

```text
availability
-> needs start time
-> needs end time
```

Retrofit supports query parameters using:

```text
@Query
```

### 📚 Helpful Resource

**Retrofit — API Declarations:**  
https://square.github.io/retrofit/declarations/

---

# 🧠 18. Validate the Availability Request

The API must validate the requested period.

At minimum:

1. Parking Area exists.
2. Start value exists.
3. End value exists.
4. Both values can be interpreted as valid dates/times.
5. Start occurs before end.
6. Reservation is not for an invalid past period.
7. Requested period is compatible with the Parking Area operating hours.
8. Parking Area is active.

If validation fails:

```text
400 Bad Request
```

is generally appropriate for invalid input.

---

# 🏢 19. Check Parking Area Status

Before calculating individual bay availability, check the Parking Area.

A Parking Area with:

```text
status = closed
```

or:

```text
status = maintenance
```

should not offer new reservations.

Do not continue querying for available bays if the facility itself cannot accept reservations.

---

# 🕐 20. Validate Operating Hours

Assume a Parking Area operates:

```text
06:00 -> 22:00
```

A request for:

```text
08:00 -> 10:00
```

may be valid.

A request for:

```text
23:00 -> 01:00
```

should be rejected unless your design explicitly supports overnight parking.

Define your ParkSmart operating-hours rules clearly.

---

# 🚙 21. Retrieve Candidate Parking Bays

Once the Parking Area and requested period are valid, retrieve bays belonging to the selected Parking Area.

Only consider bays whose general status permits reservation.

For example:

```text
available
```

should be considered.

A bay marked:

```text
maintenance
```

or:

```text
unavailable
```

must not appear as a bookable result.

---

# ⚠️ 22. Understand Reservation Overlap

This is the most important rule in Step 05.

Consider an existing reservation:

```text
10:00 -> 12:00
```

A new request:

```text
11:00 -> 13:00
```

overlaps.

It must be rejected.

---

# 🧠 23. The General Overlap Rule

Two periods overlap when:

```text
requestedStart < existingEnd

AND

requestedEnd > existingStart
```

You need to understand this rule before trying to build the MongoDB query.

---

# 📊 24. Work Through Overlap Examples

## Example 1

Existing:

```text
10:00 -> 12:00
```

Requested:

```text
11:00 -> 13:00
```

Result:

```text
Conflict
```

---

## Example 2

Existing:

```text
10:00 -> 12:00
```

Requested:

```text
09:00 -> 11:00
```

Result:

```text
Conflict
```

---

## Example 3

Existing:

```text
10:00 -> 12:00
```

Requested:

```text
09:00 -> 13:00
```

Result:

```text
Conflict
```

---

## Example 4

Existing:

```text
10:00 -> 12:00
```

Requested:

```text
12:00 -> 13:00
```

Result:

```text
No conflict
```

assuming ParkSmart treats the first reservation as ending exactly when the next begins.

---

## Example 5

Existing:

```text
10:00 -> 12:00
```

Requested:

```text
08:00 -> 10:00
```

Result:

```text
No conflict
```

using the same boundary rule.

---

# 🔎 25. Translate the Overlap Rule into a MongoDB Query

You are expected to research how MongoDB comparison operators can be used to find conflicting reservations.

Useful operators include:

```text
$lt

$lte

$gt

$gte

$and

$or
```

You should determine the query needed to find reservations where:

```text
requestedStart < existingEnd

AND

requestedEnd > existingStart
```

Do not copy a random Stack Overflow query without understanding which interval is being compared.

### 📚 Helpful Resources

**MongoDB — Comparison Query Operators:**  
https://www.mongodb.com/docs/manual/reference/operator/query-comparison/

**MongoDB — Logical Query Operators:**  
https://www.mongodb.com/docs/manual/reference/operator/query-logical/

**Mongoose — Queries:**  
https://mongoosejs.com/docs/queries.html

---

# 🚫 26. Ignore Reservations That No Longer Block a Bay

Not every Reservation document should necessarily prevent a new booking.

For example:

```text
cancelled
```

reservations should not normally block a bay.

Your availability logic needs to consider which reservation statuses count as conflicts.

For Step 05, a typical blocking status may include:

```text
reserved
```

Later:

```text
checked_in
```

may also block the bay.

Define this carefully in your backend logic.

---

# 📤 27. Return Available Bays

The availability endpoint should return bays that:

```text
Belong to Parking Area
-> General status permits booking
-> Have no conflicting reservation
```

Android should receive enough information to display the available choices.

For example:

```text
Bay Number

Bay Type

Floor

Section
```

Do not return unnecessary private/internal data.

---

# 📱 28. Add Availability to Retrofit

Update your Retrofit service interface.

Represent:

```text
GET /api/parking-areas/:areaId/availability
```

Use the appropriate Retrofit annotations for:

```text
@Path

@Query
```

Your Android code should send:

```text
Parking Area ID

Start Date/Time

End Date/Time
```

to the API.

---

# 🧠 29. Add Availability State to the ViewModel

Your ViewModel should manage states such as:

```text
No search performed

Loading availability

Available bays returned

No bays available

Availability error
```

Do not use:

```text
empty list
```

to mean every possible state.

An empty list may mean:

```text
No bays are available
```

which is different from:

```text
Request has not been made yet
```

---

# 🎨 30. Display Available Bays

After the API returns available bays:

```text
Availability Screen
-> List of available bays
```

Each bay should display useful information such as:

```text
Bay A01

Type:
Standard

Floor:
Ground
```

Allow the driver to select one bay.

---

# 📭 31. Handle No Availability

If no bays are available for the requested period, display a clear message.

For example:

```text
No parking bays are available for the selected time.
```

Do not present this as a technical error.

The API worked correctly.

The result simply contains no available bays.

---

# 📅 32. Create the Reservation Endpoint

Once the driver selects a bay, Android needs to request a reservation.

Create a protected endpoint such as:

```text
POST /api/reservations
```

This route must require Firebase authentication.

The request should include the reservation details needed by the API.

For example:

```text
parkingArea

parkingBay

startDateTime

endDateTime
```

Do **not** ask Android to submit:

```text
driver
```

as the trusted reservation owner.

---

# 👤 33. Determine the Driver from Authentication

The reservation owner must come from:

```text
Firebase ID Token
-> Trusted UID
-> MongoDB User
```

Not:

```text
Android request body
-> driver ID
```

This prevents one user from creating reservations pretending to be another user.

---

# 🔁 34. Recheck Availability During Reservation Creation

This is extremely important.

Android may already have called:

```text
Check Availability
```

and shown:

```text
Bay A01 available
```

But some time may pass before the user presses:

```text
Reserve
```

During that time, another driver may reserve A01.

Therefore:

```text
Availability check
```

and:

```text
Reservation creation
```

are separate events.

The reservation endpoint must check availability **again**.

---

# ⚠️ 35. Understand the Race Condition

Consider:

```text
Driver A checks availability
-> A01 available

Driver B checks availability
-> A01 available
```

Then:

```text
Driver A reserves A01
-> success
```

A few seconds later:

```text
Driver B reserves A01
```

Driver B's Android screen may still show:

```text
A01 available
```

but that information is now outdated.

The API must reject Driver B's reservation.

This is why:

> Android displays availability. The API decides availability.

---

# 📡 36. Use `409 Conflict` for Reservation Conflicts

If a reservation cannot be created because another active reservation now conflicts with it:

```text
409 Conflict
```

is a useful HTTP status.

A predictable response might contain:

```json
{
  "message": "This parking bay is no longer available for the selected period."
}
```

Android should display the returned message appropriately.

---

# ✅ 37. Validate Reservation Creation

Before creating the Reservation, the API should confirm:

- User is authenticated.
- ParkSmart User exists.
- Parking Area exists.
- Parking Area is active.
- Parking Bay exists.
- Parking Bay belongs to the specified Parking Area.
- Parking Bay status permits reservations.
- Start and end values are valid.
- Start occurs before end.
- Period is within operating hours.
- Reservation is not invalidly in the past.
- No conflicting reservation exists.

Only then:

```text
Create Reservation
```

---

# 🕒 38. Set Reservation Values on the Server

The backend should control trusted fields such as:

```text
driver

status

reservedAt
```

For a new valid reservation:

```text
status
-> reserved
```

The client should not be trusted to decide:

```text
status = checked_in
```

during reservation creation.

---

# 📱 39. Confirm Reservation Success in Android

Do not display:

```text
Reservation successful
```

immediately after the driver presses the button.

Use:

```text
User taps Reserve
-> Loading state
-> API validates request
-> API creates Reservation
-> API returns success
-> Android displays confirmation
```

If the API rejects the request:

```text
Android displays error
```

---

# 🎉 40. Create a Reservation Confirmation State

After successful reservation creation, display useful information.

For example:

```text
Reservation Confirmed

Parking Area:
North Parking

Bay:
A01

Date:
1 September 2026

Time:
10:00 -> 12:00
```

You do not need to display a QR pass yet.

That comes in Step 06.

---

# 📋 41. Create the Driver's Reservation Endpoint

Drivers need to retrieve their own reservations.

Create a protected endpoint such as:

```text
GET /api/reservations/me
```

The API should:

1. Authenticate the user.
2. Find their ParkSmart User profile.
3. Retrieve reservations belonging to that user.
4. Return relevant Parking Area and Parking Bay information.

Do not accept a random:

```text
userId
```

query parameter as the trusted owner.

---

# 🔍 42. Populate Useful Reservation Information

A Reservation may store references to:

```text
ParkingArea

ParkingBay
```

When displaying reservations, Android will likely need readable information such as:

```text
Parking Area name

Parking Bay number
```

Research how:

```text
populate()
```

can help return useful related information.

### 📚 Helpful Resource

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

---

# 📱 43. Add a My Reservations Screen

Create a Compose screen for the signed-in driver's reservations.

For example:

```text
MyReservationsScreen
```

The screen should load:

```text
GET /api/reservations/me
```

using:

```text
ViewModel
-> Repository
-> Retrofit
```

---

# 🗂️ 44. Organise Reservation Information

You may choose to group reservations into sections such as:

```text
Upcoming

Previous

Cancelled
```

or display one list with clear status labels.

The exact UI design is your decision.

The user should be able to understand:

```text
Where?
When?
Which bay?
What status?
```

without opening MongoDB IDs or raw JSON.

---

# 🕒 45. Format Dates for the User

Do not display raw backend timestamp values if they are difficult for users to read.

For example, a raw API timestamp may resemble an ISO date/time representation.

Android can parse the value and display something more readable.

For example:

```text
01 September 2026
10:00
```

Use a consistent format throughout ParkSmart.

### 📚 Helpful Resource

**Android — DateTimeFormatter:**  
https://developer.android.com/reference/kotlin/java/time/format/DateTimeFormatter

`DateTimeFormatter` supports predefined and custom formats for parsing and displaying date/time values. :contentReference[oaicite:2]{index=2}

---

# ❌ 46. Create Reservation Cancellation

Drivers should be able to cancel an eligible reservation.

A suitable route might be:

```text
PATCH /api/reservations/:id/cancel
```

or another REST design that clearly represents the operation.

Do not simply delete every cancelled reservation from MongoDB.

Keeping the document allows ParkSmart to preserve reservation history.

---

# 👤 47. Enforce Reservation Ownership

A driver must only be able to cancel:

```text
their own reservation
```

This is not enough:

```text
Android hides cancellation button for someone else's reservation
```

The API must confirm:

```text
Reservation.driver
```

matches:

```text
Authenticated ParkSmart User
```

before changing the reservation.

---

# 🚫 48. Do Not Trust Reservation IDs Alone

Suppose Driver A owns:

```text
Reservation ID 123
```

Driver B manually calls:

```text
PATCH /api/reservations/123/cancel
```

If the API only checks:

```text
Does Reservation 123 exist?
```

Driver B could cancel Driver A's reservation.

The API must check:

```text
Does Reservation exist?
-> Does authenticated user own it?
-> Is cancellation allowed?
```

---

# 🚦 49. Enforce Valid Cancellation States

A reservation with:

```text
status = reserved
```

may be cancellable.

A reservation already marked:

```text
cancelled
```

should not be cancelled again.

Later:

```text
checked_in
```

reservations should not normally be cancellable through the driver's normal cancellation process.

Define and enforce your allowed status transitions.

---

# ⏰ 50. Consider a Cancellation Cut-Off

You may optionally introduce a rule such as:

```text
Reservations cannot be cancelled after the start time.
```

or:

```text
Reservations cannot be cancelled less than 15 minutes before the start.
```

If you introduce such a rule:

- Document it.
- Enforce it on the API.
- Display it clearly to users.

Do not create arbitrary hidden rules.

---

# 📡 51. Add Reservation Endpoints to Retrofit

Update your Retrofit service interface.

Your Android application should now represent endpoints for:

```text
Check availability

Create reservation

Get my reservations

Cancel reservation
```

Use the appropriate Retrofit annotations.

Possible annotations include:

```text
@GET

@POST

@PATCH

@Path

@Query

@Body
```

---

# 📦 52. Extend the Repository

Update your data layer.

The Repository should now support operations conceptually relating to:

```text
Check availability

Create reservation

Load my reservations

Cancel reservation
```

Do not bypass the Repository from your Compose screens.

---

# 🧠 53. Create or Extend Reservation ViewModel State

Your reservation UI may require state such as:

```text
Selected date

Selected start time

Selected end time

Available bays

Selected bay

Availability loading

Reservation loading

Reservation success

Reservation error

My reservations
```

Keep the state understandable.

Do not create disconnected state variables without considering how the screen behaves as a whole.

---

# 🎨 54. Represent Loading Correctly

Checking availability requires a network request.

Creating a reservation requires another network request.

These should have appropriate loading states.

For example:

```text
Checking available bays...
```

and:

```text
Creating reservation...
```

Prevent repeated button presses while an important request is still processing where appropriate.

---

# 🧪 55. Test the Availability Endpoint Independently

Before testing through Android, use Postman, Bruno or another API client.

Test:

### Valid Period

Expected:

```text
Available bays returned
```

### Parking Area Does Not Exist

Expected:

```text
404 Not Found
```

### Invalid Date/Time

Expected:

```text
400 Bad Request
```

### End Before Start

Expected:

```text
400 Bad Request
```

### Closed Parking Area

Expected:

```text
Appropriate rejection
```

### No Available Bays

Expected:

```text
Successful request
+
Empty availability result
```

This is not necessarily a server error.

---

# 🧪 56. Test Reservation Creation Independently

Test:

### Valid Reservation

Expected:

```text
201 Created
```

### Invalid Parking Bay

Expected:

```text
404 or validation response
```

### Bay Under Maintenance

Expected:

```text
Rejected
```

### Time Outside Operating Hours

Expected:

```text
Rejected
```

### Overlapping Reservation

Expected:

```text
409 Conflict
```

### Missing Authentication

Expected:

```text
401 Unauthorized
```

---

# 🧪 57. Test Ownership

Create:

```text
Driver A

Driver B
```

Driver A creates a reservation.

Driver B attempts to cancel Driver A's reservation.

Expected:

```text
403 Forbidden
```

or another appropriate ownership rejection according to your API design.

The cancellation must not succeed.

---

# 🧪 58. Test Race-Like Behaviour Manually

You should test the most important ParkSmart scenario.

Use two users or two API requests.

1. Check availability for the same bay/time.
2. Confirm both initially see it as available.
3. Create the first reservation.
4. Attempt the second reservation.

Expected:

```text
First reservation
-> success

Second conflicting reservation
-> rejected
```

If both succeed, your reservation logic is incorrect.

---

# 🔁 59. Refresh Availability After Reservation

After a reservation succeeds:

```text
A previously available bay
```

should no longer appear as available for the conflicting period.

Request availability again and confirm the result.

---

# 📱 60. Test the Full Android Flow

Test:

```text
Sign In
-> Select Parking Area
-> Choose Date
-> Choose Start Time
-> Choose End Time
-> Check Availability
-> Select Bay
-> Reserve
-> API confirms
-> Confirmation shown
-> Open My Reservations
-> Reservation appears
```

Then:

```text
Cancel Reservation
-> API confirms cancellation
-> UI updates
```

---

# 📁 61. Review Your Backend Structure

Your backend will now contain something similar to:

```text
src/
|
|-- config/
|
|-- controllers/
|   |
|   |-- parkingAreaController.js
|   |-- parkingBayController.js
|   |-- reservationController.js
|
|-- middleware/
|
|-- models/
|   |
|   |-- User.js
|   |-- ParkingArea.js
|   |-- ParkingBay.js
|   |-- Reservation.js
|
|-- routes/
|   |
|   |-- parkingAreaRoutes.js
|   |-- parkingBayRoutes.js
|   |-- reservationRoutes.js
|
|-- services/
|   |
|   |-- reservationService.js
|
|-- app.js
|
|-- server.js
```

Your exact structure may differ.

Reservation conflict logic is a good candidate for:

```text
services/
```

because it represents business logic rather than simple request handling.

---

# 🧠 62. Understand the Full Reservation Flow

You should now be able to explain:

```text
Driver selects date/time
-> Android validates obvious input
-> Retrofit requests availability
-> API validates period
-> API retrieves eligible Parking Bays
-> API searches Reservations for conflicts
-> Available bays returned
-> Driver selects bay
-> Android requests reservation
-> API validates everything again
-> Conflict check runs again
-> Reservation stored
-> Success returned
-> Android updates UI
```

---

# 🔒 Security and Business Rules

## The API Owns Availability

Android may display availability.

The API decides whether the reservation is permitted.

---

## The API Owns Reservation Ownership

Android must not decide which user owns a Reservation.

Use:

```text
Verified Firebase UID
-> ParkSmart User
-> Reservation.driver
```

---

## The API Controls Status

Do not trust Android to create:

```text
status = checked_in
```

or:

```text
status = completed
```

---

## Validate Dates on Both Sides

Android validation:

```text
Better user experience
```

API validation:

```text
Required for correctness and security
```

---

## Cancelled Reservations Should Not Block New Bookings

Make sure your conflict query considers reservation status.

---

# 🐛 Common Problems

## Date picker returns an unexpected date

Check how the selected date value is converted and which time zone is being used.

Do not mix local and UTC assumptions without understanding the conversion.

---

## Time appears correct in Android but different in MongoDB

Check:

- Time zone conversion.
- Date serialisation.
- API parsing.
- MongoDB stored Date value.

Make sure your project has one documented date/time strategy.

---

## Availability always returns every bay

Check:

- Existing Reservation query.
- Parking Bay IDs.
- Requested start/end values.
- Blocking reservation statuses.
- Overlap conditions.

---

## Availability always returns no bays

Check:

- Date parsing.
- Parking Area status.
- Parking Bay status.
- Overlap comparison operators.
- Existing test data.

---

## A reservation ending at 12:00 blocks one starting at 12:00

Review your comparison operators.

If your intended boundary rule allows back-to-back bookings:

```text
10:00 -> 12:00

12:00 -> 13:00
```

your overlap query must reflect that decision.

---

## Duplicate/overlapping reservations are created

The final conflict check must occur inside the reservation creation flow.

Do not rely only on the earlier availability endpoint.

---

## Reservation has the wrong driver

Do not accept the owner from Android.

Check the verified Firebase UID and MongoDB User lookup.

---

## Driver can cancel someone else's reservation

Check reservation ownership before updating status.

---

## Android receives `409`

This may be a valid business response.

For example:

```text
The bay was reserved by another user before your request completed.
```

Handle it as a reservation conflict rather than a generic app crash.

---

## Raw ISO time looks ugly

Parse and format the API value for the UI.

Do not alter the stored backend value simply to make the UI prettier.

---

# 📚 Helpful Resources

## 📅 Compose Date Pickers

**Android — Date Pickers:**  
https://developer.android.com/develop/ui/compose/components/datepickers

Use this when implementing the reservation date selector. The current Compose documentation covers docked, modal and modal-input date pickers. :contentReference[oaicite:3]{index=3}

---

## ⏰ Compose Time Pickers

**Android — Time Pickers:**  
https://developer.android.com/develop/ui/compose/components/time-pickers

**Android — Time Picker Dialogs:**  
https://developer.android.com/develop/ui/compose/components/time-pickers-dialogs

Focus on:

```text
TimePicker

TimeInput

TimePickerState
```

:contentReference[oaicite:4]{index=4}

---

## 🕒 Date/Time Formatting

**Android — DateTimeFormatter:**  
https://developer.android.com/reference/kotlin/java/time/format/DateTimeFormatter

Use this to understand parsing and display formatting for dates and times. :contentReference[oaicite:5]{index=5}

---

## 🍃 Mongoose Dates

**Mongoose — Working with Dates:**  
https://mongoosejs.com/docs/tutorials/dates.html

Use this when designing:

```text
startDateTime

endDateTime

reservedAt
```

---

## 🔎 Mongoose Queries

**Mongoose — Queries:**  
https://mongoosejs.com/docs/queries.html

Use this while building availability and conflict searches.

---

## 🧮 MongoDB Comparison Operators

**MongoDB — Comparison Query Operators:**  
https://www.mongodb.com/docs/manual/reference/operator/query-comparison/

Focus on:

```text
$lt

$lte

$gt

$gte
```

---

## 🧠 MongoDB Logical Operators

**MongoDB — Logical Query Operators:**  
https://www.mongodb.com/docs/manual/reference/operator/query-logical/

Focus on:

```text
$and

$or
```

---

## 🔗 Mongoose Relationships

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

Useful for returning readable Parking Area and Parking Bay information with Reservation data.

---

## 📡 Retrofit

**Retrofit — API Declarations:**  
https://square.github.io/retrofit/declarations/

Focus on:

```text
@GET

@POST

@PATCH

@Path

@Query

@Body
```

---

## 🧠 Android UI State

**Android — UI Layer:**  
https://developer.android.com/topic/architecture/ui-layer

**Android — State Production:**  
https://developer.android.com/topic/architecture/ui-layer/state-production

Use these while managing:

```text
Availability loading

Available bays

Reservation loading

Reservation success

Reservation errors
```

---

## 🌐 HTTP Status Codes

**MDN — HTTP Status Codes:**  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

Pay particular attention to:

```text
200 OK

201 Created

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict
```

---

# 🧪 Step 05 Testing Checklist

## 📦 Reservation Model

- [ ] `Reservation.js` exists.
- [ ] Driver relationship exists.
- [ ] Parking Area relationship exists.
- [ ] Parking Bay relationship exists.
- [ ] Start date/time exists.
- [ ] End date/time exists.
- [ ] Status is restricted.
- [ ] Timestamps are enabled.
- [ ] Trusted fields are controlled by the backend.

---

## 📅 Date and Time UI

- [ ] Driver can select a date.
- [ ] Driver can select a start time.
- [ ] Driver can select an end time.
- [ ] Missing values are handled.
- [ ] End-before-start is handled in Android.
- [ ] Date/time values are sent consistently to the API.

---

## 📊 Availability

- [ ] Availability endpoint exists.
- [ ] Parking Area ID is validated.
- [ ] Start/end values are validated.
- [ ] Closed Parking Area is rejected.
- [ ] Maintenance Parking Area is rejected.
- [ ] Unavailable bays are excluded.
- [ ] Maintenance bays are excluded.
- [ ] Existing reservations are checked.
- [ ] Cancelled reservations do not incorrectly block bays.
- [ ] Empty availability is handled correctly.
- [ ] Android displays available bays.

---

## ⚠️ Conflict Detection

- [ ] Overlap rule is implemented.
- [ ] Partial overlap from the start is rejected.
- [ ] Partial overlap from the end is rejected.
- [ ] Reservation surrounding an existing booking is rejected.
- [ ] Reservation inside an existing booking is rejected.
- [ ] Back-to-back bookings behave according to the documented rule.
- [ ] Conflict logic is rechecked during reservation creation.

---

## 📅 Reservation Creation

- [ ] Reservation endpoint is protected.
- [ ] Driver comes from authenticated user.
- [ ] Parking Area exists.
- [ ] Parking Bay exists.
- [ ] Bay belongs to Parking Area.
- [ ] Bay status is valid.
- [ ] Time range is valid.
- [ ] Operating hours are enforced.
- [ ] Conflicting reservation returns an appropriate response.
- [ ] Successful creation returns `201`.
- [ ] Android displays success only after API confirmation.

---

## 📋 Reservation Management

- [ ] Driver can retrieve their own reservations.
- [ ] Reservation information includes readable Parking Area details.
- [ ] Reservation information includes readable Parking Bay details.
- [ ] Upcoming reservations display.
- [ ] Previous reservations display.
- [ ] Reservation status displays.
- [ ] Eligible reservation can be cancelled.
- [ ] Cancelled reservation remains in history.
- [ ] Driver cannot cancel another driver's reservation.
- [ ] Invalid cancellation state is rejected.

---

## 🧪 Failure Testing

- [ ] Missing authentication tested.
- [ ] Invalid Parking Area tested.
- [ ] Invalid Parking Bay tested.
- [ ] Invalid dates tested.
- [ ] Closed Parking Area tested.
- [ ] Maintenance bay tested.
- [ ] No-availability case tested.
- [ ] Overlap tested.
- [ ] Reservation race scenario tested manually.
- [ ] Ownership violation tested.
- [ ] API-offline state tested in Android.

---

# ✅ Step 05 Complete

You started this step with parking data:

```text
Parking Areas
-> Parking Bays
```

You now have a working reservation process:

```text
Select Parking Area
-> Select Date and Time
-> Check Availability
-> Select Parking Bay
-> Create Reservation
-> View My Reservations
-> Cancel Eligible Reservation
```

Most importantly, ParkSmart now enforces its central business rule:

```text
One Parking Bay
-> Cannot have overlapping active Reservations
```

However, a confirmed reservation still needs a secure way to be used when the driver arrives.

In the final step, you will add:

```text
Reservation
-> Secure QR Token
-> QR Parking Pass
-> Manager Scanner
-> API Validation
-> Check-In
-> Session Completion
```

# ➡️ Continue to Step 06 — QR Passes and Check-In
