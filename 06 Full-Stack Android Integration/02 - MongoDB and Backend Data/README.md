# 🍃 Step 02 — MongoDB and Backend Data

## 📚 What You Are Building

In Step 01, you created the foundation of the ParkSmart system.

You currently have:

```text
Android Application

and

Node.js + Express REST API
```

Your API can receive requests and return responses, but there is an important problem:

> Where will ParkSmart permanently store its data?

If the API restarts, information stored only inside JavaScript variables disappears.

ParkSmart needs persistent storage for information such as:

```text
Parking Areas

Parking Bays

Users

Reservations
```

In this step, you will introduce:

```text
MongoDB
+
Mongoose
```

By the end of this step, your backend architecture will be:

```text
API Client
-> Express API
-> Mongoose
-> MongoDB
```

You will learn MongoDB from the beginning.

You are **not expected to already know MongoDB**.

---

# 🎯 Learning Objectives

After completing this step, you should be able to:

- Explain what MongoDB is.
- Explain the difference between relational and document databases.
- Explain what a MongoDB document is.
- Explain what a MongoDB collection is.
- Explain the purpose of `_id`.
- Explain the purpose of an `ObjectId`.
- Create a MongoDB database.
- Connect the ParkSmart API to MongoDB.
- Explain why ParkSmart uses Mongoose.
- Explain the difference between a Mongoose schema and model.
- Create basic Mongoose schemas.
- Apply basic validation.
- Use timestamps.
- Understand relationships between MongoDB documents.
- Understand why indexes are important.
- Read and inspect data using MongoDB Compass or Atlas.
- Confirm that your API can successfully communicate with MongoDB.

---

# 🧠 1. What Is a Database?

A database stores information that an application needs to keep.

For example, ParkSmart will eventually store information such as:

```text
Parking Area

Name:
North Parking

Address:
12 Campus Road

Status:
active
```

It may also store:

```text
Parking Bay

Bay Number:
A01

Type:
standard

Status:
available
```

Without a database, this information would need to be recreated whenever the application starts.

A database provides **persistent storage**.

This means that the information remains available even after:

- The API stops.
- The server restarts.
- Android closes.
- Another user opens the application.

---

# 🗄️ 2. Relational Databases vs MongoDB

You may already be familiar with relational databases.

A relational database often organises data using:

```text
Tables
Rows
Columns
Primary Keys
Foreign Keys
```

For example:

```text
PARKING_AREAS

AreaID | Name          | Status
---------------------------------
1      | North Parking | active
2      | South Parking | active
```

MongoDB works differently.

MongoDB is a **document database**.

Instead of storing data primarily as rows inside tables, MongoDB stores documents inside collections.

The equivalent structure is closer to:

```text
Database
   ->
Collection
   ->
Document
```

---

# 📦 3. Understand Documents

A MongoDB **document** represents one stored item.

A parking area could conceptually be represented as:

```json
{
  "_id": "...",
  "name": "North Parking",
  "address": "12 Campus Road",
  "status": "active"
}
```

This resembles JSON.

MongoDB internally stores data using BSON, which is a binary representation that supports additional data types.

You do not need to understand the internal BSON format in detail for this activity.

The important idea is:

```text
One document
=
One stored object/item
```

Examples:

```text
One Parking Area
=
One document

One Parking Bay
=
One document

One Reservation
=
One document
```

### 📚 Helpful Resource

**MongoDB — Documents:**  
https://www.mongodb.com/docs/manual/core/document/

---

# 📚 4. Understand Collections

A **collection** groups related documents.

For example:

```text
ParkSmart Database
|
|-- parkingareas
|
|-- parkingbays
|
|-- users
|
|-- reservations
```

The `parkingareas` collection may contain:

```text
Parking Area Document 1

Parking Area Document 2

Parking Area Document 3
```

This is similar to how a relational database table groups related records, although the underlying database model is different.

---

# 🏷️ 5. Understand `_id`

Every MongoDB document requires a unique identifier.

MongoDB uses:

```text
_id
```

for this purpose.

A document may resemble:

```json
{
  "_id": "68...",
  "name": "North Parking",
  "status": "active"
}
```

If you do not manually provide `_id`, MongoDB normally generates one when the document is created.

The generated value is commonly an:

```text
ObjectId
```

This allows MongoDB to uniquely identify individual documents.

Later, ParkSmart will use document IDs when relating information such as:

```text
Parking Bay
-> belongs to
Parking Area
```

### 📚 Helpful Resource

**MongoDB — ObjectId:**  
https://www.mongodb.com/docs/manual/reference/bson-types/#objectid

---

# 🧩 6. What Is Mongoose?

Your Node.js API could communicate with MongoDB directly using MongoDB's Node.js driver.

For ParkSmart, however, we will use:

```text
Mongoose
```

Mongoose provides an additional modelling layer between the Node.js application and MongoDB.

It helps us define:

- The expected shape of our documents.
- Field types.
- Required fields.
- Allowed values.
- Relationships.
- Timestamps.
- Validation.
- Models used to work with collections.

The flow becomes:

```text
Express
-> Mongoose
-> MongoDB
```

### 📚 Helpful Resources

**Mongoose — Getting Started:**  
https://mongoosejs.com/docs/

**MongoDB — Mongoose Integration:**  
https://www.mongodb.com/docs/drivers/node/current/integrations/mongoose/

---

# 🧱 7. Understand Schemas and Models

These two terms are important.

## Schema

A **schema** defines how a type of document should be structured.

For example, you might decide that every Parking Area should contain:

```text
name
description
address
status
```

You may also decide:

```text
name
-> required

status
-> must contain an allowed value
```

The schema describes these rules.

---

## Model

A **model** is created from a schema.

The model is what your application uses when working with documents in a collection.

Conceptually:

```text
Schema
-> describes the document

Model
-> allows the application to work with the collection
```

Later, models will allow ParkSmart to perform operations such as:

```text
Create Parking Area

Find Parking Areas

Update Parking Area

Delete Parking Area
```

### 📚 Helpful Resources

**Mongoose — Schemas:**  
https://mongoosejs.com/docs/guide.html

**Mongoose — Models:**  
https://mongoosejs.com/docs/models.html

---

# 📦 8. Install Mongoose

Open the terminal inside:

```text
ParkSmart/api/
```

Install Mongoose:

```powershell
npm install mongoose
```

After installation, check:

```text
package.json
```

Mongoose should appear under:

```text
dependencies
```

You should **not** install MongoDB libraries inside the Android application.

MongoDB communication belongs to the backend API.

---

# 🗺️ 9. Choose Where MongoDB Will Run

You have two common options.

## Option A — MongoDB Atlas

MongoDB Atlas hosts the database online.

This means you do not need to run the MongoDB database server manually on your own computer.

## Option B — MongoDB Community Server

MongoDB Community Server runs locally on your computer.

Your lecturer will advise you which option to use.

For this activity, **MongoDB Atlas is recommended** because it provides a consistent database environment that can be accessed by your backend using a connection string.

---

# ☁️ 10. Create a MongoDB Atlas Account

Go to:

https://www.mongodb.com/atlas

Create an account or sign in.

Once signed in, create a project for your work.

Suggested project name:

```text
ParkSmart
```

The exact MongoDB Atlas interface may change over time, so use the official documentation if the wording or location of a button differs from the screenshots or instructions you see elsewhere.

### 📚 Helpful Resources

**MongoDB Atlas:**  
https://www.mongodb.com/atlas

**MongoDB — Get Started:**  
https://www.mongodb.com/docs/get-started/

**MongoDB — Deploy a Free Cluster:**  
https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/

---

# 🗄️ 11. Create the Database Deployment

Inside your Atlas project, create a database deployment.

Choose the **Free** option where available.

You do not need a large or production-ready database for this practical.

Give the deployment a meaningful name.

For example:

```text
ParkSmartCluster
```

Choose an appropriate region.

A nearby region is generally preferable because it reduces unnecessary network distance.

Do not create several clusters for the same practical unless you have a specific reason to do so.

---

# 👤 12. Create a Database User

Your Node.js API needs credentials to connect to MongoDB.

Create a database user inside Atlas.

This is **not necessarily the same account** that you use to log into the MongoDB website.

The database user is specifically used to authenticate a database connection.

Choose:

```text
Username:
An appropriate database username

Password:
A strong password
```

Store the password securely.

You will need it when creating your connection string.

> ⚠️ Do not use a password that you use for your email, student portal, Google account or other personal accounts.

> ⚠️ Do not paste your MongoDB password into your README.

---

# 🌐 13. Configure Network Access

MongoDB Atlas controls which network locations are allowed to connect to the database.

Configure network access so that your development computer can connect.

Use the Atlas interface and official documentation to add an appropriate IP access-list entry.

### 📚 Resource

**MongoDB Atlas — Configure IP Access List:**  
https://www.mongodb.com/docs/atlas/security/ip-access-list/

> ⚠️ Avoid making database access unnecessarily broad in a production application. Use an appropriate configuration for your development environment.

---

# 🔗 14. Obtain the MongoDB Connection String

After the database deployment is ready:

1. Open the cluster/deployment.
2. Select the connection option.
3. Choose the option for connecting an application.
4. Select the Node.js driver where applicable.
5. Copy the connection string.

It will resemble:

```text
mongodb+srv://<username>:<password>@<cluster-address>/
```

Your exact value will be different.

Replace the placeholders with your own database username and password where required.

> ⚠️ Do not paste your real connection string into GitHub, your README or screenshots shared publicly.

### 📚 Resource

**MongoDB Atlas — Connect Your Application:**  
https://www.mongodb.com/docs/atlas/connect-your-application/

---

# 🔐 15. Add the Connection String to `.env`

Open:

```text
api/.env
```

You already created this file during Step 01.

It currently contains something similar to:

```env
PORT=3000
```

Add another environment variable:

```env
MONGODB_URI=PASTE_YOUR_CONNECTION_STRING_HERE
```

Your file should conceptually resemble:

```env
PORT=3000
MONGODB_URI=YOUR_PRIVATE_CONNECTION_STRING
```

Do **not** place quotation marks around the value unless your configuration specifically requires them.

---

# 📄 16. Update `.env.example`

Open:

```text
api/.env.example
```

Update it to:

```env
PORT=
MONGODB_URI=
```

Notice the difference.

`.env` contains:

```text
Real values
```

`.env.example` contains:

```text
Variable names only
```

This lets another developer know which configuration values they need without exposing your credentials.

---

# 🔒 17. Check `.gitignore`

Confirm that:

```text
.env
```

is ignored.

Run:

```powershell
git status
```

Your real `.env` file should not appear as a new file waiting to be committed.

Your:

```text
.env.example
```

should be safe to commit.

If `.env` was already committed previously, adding it to `.gitignore` does not automatically remove it from Git history or tracking.

Refer back to the Git resources from Step 01 if necessary.

---

# 🧭 18. Create the Database Configuration File

You already created:

```text
src/config/
```

during Step 01.

Inside this folder, create:

```text
database.js
```

This file will be responsible for establishing the MongoDB connection.

Your task is to create a reusable database-connection function.

It should:

1. Access `MONGODB_URI` from the environment.
2. Check that a connection string exists.
3. Ask Mongoose to connect to MongoDB.
4. Handle the asynchronous connection process.
5. Display a useful success message when the connection succeeds.
6. Display useful error information when the connection fails.
7. Prevent the API from pretending that everything is working when the required database connection has failed.

You are expected to research how:

```text
mongoose.connect()
```

works.

Do not copy an entire database file without understanding it.

### 📚 Helpful Resources

**Mongoose — Connections:**  
https://mongoosejs.com/docs/connections.html

**Mongoose — Getting Started:**  
https://mongoosejs.com/docs/

Focus on:

```text
mongoose.connect()

async / await

try / catch

process.env
```

---

# 🧠 19. Why Should Database Logic Be in `config/`?

You could technically place the connection directly inside:

```text
server.js
```

However, separating the database configuration gives each file a clearer responsibility.

Conceptually:

```text
database.js
-> knows how to connect to MongoDB

server.js
-> knows how to start ParkSmart
```

This will become more useful as the project grows.

---

# ▶️ 20. Update the Server Start-up Process

Your API should establish its required database connection during start-up.

The intended sequence should now be:

```text
Start ParkSmart
-> Load environment configuration
-> Connect to MongoDB
-> Confirm connection
-> Start Express server
```

If MongoDB is essential to the application and the connection fails, you should not silently act as though ParkSmart is functioning normally.

Update your server start-up structure accordingly.

You are expected to determine how to call your reusable database-connection function from:

```text
server.js
```

---

# 🧪 21. Test the Database Connection

Start the backend:

```powershell
npm run dev
```

Watch the VS Code terminal.

You should see meaningful output indicating:

```text
MongoDB connection successful
```

and:

```text
ParkSmart API is running
```

The exact wording is your choice.

---

## Test a Failed Connection

Do not only test the successful path.

Temporarily introduce an invalid MongoDB connection value.

Restart the API.

Observe:

- What error appears?
- Does your application clearly indicate that MongoDB could not be reached?
- Does the error help you determine where the problem is?

Restore the correct connection string afterwards.

> 🧪 Good testing includes intentionally checking failure scenarios.

---

# 🧭 22. Install MongoDB Compass

MongoDB Compass is a graphical application for viewing and working with MongoDB data.

It makes it easier to inspect:

- Databases.
- Collections.
- Documents.
- Fields.
- Data types.
- Indexes.

Download Compass from:

https://www.mongodb.com/try/download/compass

### 📚 Resources

**MongoDB Compass:**  
https://www.mongodb.com/docs/compass/

**Install Compass:**  
https://www.mongodb.com/docs/compass/install/

---

# 🔗 23. Connect Compass to Your Database

Open MongoDB Compass.

Use your MongoDB connection information to connect to the same database deployment used by your API.

Once connected, familiarise yourself with the interface.

Locate where Compass displays:

```text
Databases
Collections
Documents
Indexes
```

Do not worry if ParkSmart collections do not exist yet.

MongoDB collections are generally created when data is first stored.

---

# 🏢 24. Start With One ParkSmart Model

Do **not** create every ParkSmart model at once.

You are learning MongoDB for the first time.

Start with:

```text
ParkingArea
```

This is a good first model because it is relatively simple and does not initially require several complicated relationships.

Create:

```text
src/models/ParkingArea.js
```

---

# 🧠 25. Plan the ParkingArea Data

Before writing the schema, decide what a Parking Area represents.

A parking area represents a physical parking facility.

Examples:

```text
North Campus Parking

Library Parking

Main Building Basement
```

For now, your ParkingArea should contain information such as:

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

Do **not** add manager ownership yet.

Users and roles will be introduced during the authentication step.

---

# 📋 26. Choose the Correct Data Types

Use the Mongoose SchemaTypes documentation to determine suitable data types.

Consider:

| Field | Think About |
|---|---|
| `name` | Text |
| `description` | Text |
| `address` | Text |
| `latitude` | Number |
| `longitude` | Number |
| `openingTime` | How will you represent this? |
| `closingTime` | How will you represent this? |
| `status` | Text with restricted values |

You must decide which fields:

- Are required.
- May be optional.
- Need default values.
- Need validation.

### 📚 Helpful Resource

**Mongoose — SchemaTypes:**  
https://mongoosejs.com/docs/schematypes.html

---

# 🚦 27. Define Parking Area Statuses

For ParkSmart, use these status values:

```text
active

closed

maintenance
```

The meaning of these statuses is:

### `active`

The parking area is operating normally.

### `closed`

The parking facility is currently closed and should not accept new reservations.

### `maintenance`

The facility is temporarily unavailable because maintenance is taking place.

The schema should prevent random values such as:

```text
sort-of-open

maybe

broken-ish
```

from being saved as valid ParkSmart statuses.

Research how Mongoose:

```text
enum
```

validation can restrict allowed String values.

---

# ✅ 28. Add Basic Validation

Your first schema should include sensible validation.

Consider requirements such as:

```text
name
-> required

address
-> required

latitude
-> required number

longitude
-> required number

status
-> allowed ParkSmart status
```

You should also consider valid geographical ranges:

```text
Latitude:
-90 to 90

Longitude:
-180 to 180
```

Research Mongoose's built-in validation options.

### 📚 Helpful Resource

**Mongoose — Validation:**  
https://mongoosejs.com/docs/validation.html

Focus on:

```text
required

enum

min

max

default
```

---

# 🕒 29. Add Timestamps

ParkSmart should know when important records were created and last changed.

Mongoose can manage:

```text
createdAt

updatedAt
```

automatically.

Enable timestamps for the ParkingArea schema.

Do not manually ask Android to provide these values.

### 📚 Resource

**Mongoose — Timestamps:**  
https://mongoosejs.com/docs/timestamps.html

---

# 📦 30. Create the ParkingArea Model

Once your schema has been designed:

1. Create the Mongoose schema.
2. Add your fields.
3. Add the required validation.
4. Enable timestamps.
5. Create a model from the schema.
6. Export the model so other backend files can use it.

You are expected to research the correct syntax from the Mongoose documentation.

### 📚 Resources

**Mongoose — Schemas:**  
https://mongoosejs.com/docs/guide.html

**Mongoose — Models:**  
https://mongoosejs.com/docs/models.html

Focus on the relationship:

```text
Schema
-> Model
-> MongoDB Collection
```

---

# 🧪 31. Create a Temporary Database Test

Before building proper Parking Area API endpoints, prove that the model works.

Create a temporary test method or development-only test route.

Its purpose is to:

```text
Receive temporary test request
-> Create a ParkingArea document
-> Save it to MongoDB
-> Return the stored document
```

You are responsible for researching the appropriate Mongoose model operation.

Useful Mongoose operations to investigate include:

```text
create()

save()
```

### 📚 Resource

**Mongoose — Models:**  
https://mongoosejs.com/docs/models.html

> ⚠️ This is temporary development code. You will build the proper ParkSmart Parking Area routes later.

---

# 📤 32. Test Creating a Parking Area

Use your API testing tool.

Send a request containing valid Parking Area data.

An example request body may resemble:

```json
{
  "name": "North Parking",
  "description": "Parking area near the north entrance",
  "address": "12 Campus Road",
  "latitude": -29.8500,
  "longitude": 31.0200,
  "openingTime": "06:00",
  "closingTime": "22:00",
  "status": "active"
}
```

This JSON is **test data**, not the final implementation logic.

If successful, inspect the returned document.

Look for fields such as:

```text
_id

name

status

createdAt

updatedAt
```

---

# 🔍 33. Inspect the Document in MongoDB

Open MongoDB Compass or Atlas.

Find the collection created by Mongoose.

Open the document you created.

Compare what you sent:

```json
{
  "name": "North Parking",
  "status": "active"
}
```

with what MongoDB stored.

You should notice additional information such as:

```text
_id

createdAt

updatedAt
```

Ask yourself:

- Where did `_id` come from?
- Where did the timestamps come from?
- Did MongoDB preserve the data types correctly?

---

# ❌ 34. Test Validation

Now deliberately send invalid data.

Examples:

### Missing name

Remove the `name` value.

### Invalid status

Use:

```json
{
  "status": "banana"
}
```

### Invalid latitude

Try:

```json
{
  "latitude": 500
}
```

Observe what happens.

Your model should reject data that breaks the schema validation rules you configured.

> 🧪 Never test only valid input.

---

# 🔎 35. Test Reading Data

Create another temporary development-only endpoint or test method that retrieves Parking Area documents.

Research a suitable Mongoose model method for retrieving documents.

Useful methods to investigate include:

```text
find()

findById()
```

Test retrieving the Parking Area you created.

The intended flow is now:

```text
API Request
-> Express
-> Mongoose Model
-> MongoDB
-> Documents
-> API Response
```

### 📚 Resource

**Mongoose — Queries:**  
https://mongoosejs.com/docs/queries.html

---

# 🚙 36. Introduce a Second Model: ParkingBay

Once you understand the ParkingArea model, create:

```text
src/models/ParkingBay.js
```

A ParkingBay represents an individual bay inside a ParkingArea.

Examples:

```text
A01

A02

B01

EV03
```

---

# 📋 37. Plan the ParkingBay Data

The ParkingBay model should contain information such as:

```text
parkingArea

bayNumber

floor

section

bayType

status
```

Suggested bay types:

```text
standard

accessible

motorcycle

electric
```

Suggested statuses:

```text
available

unavailable

maintenance
```

Remember:

```text
status = available
```

means that the bay is generally enabled.

It does **not** mean the bay is available at every date and time.

Time-based reservation availability will be calculated later.

---

# 🔗 38. Create the Relationship Between Bays and Areas

A ParkingBay belongs to a ParkingArea.

Instead of storing the entire Parking Area document inside every Parking Bay, ParkSmart can store a reference to the related ParkingArea document.

Conceptually:

```text
ParkingBay
-> parkingArea
-> ParkingArea document ID
```

Research:

```text
Schema.Types.ObjectId

ref
```

### 📚 Helpful Resources

**Mongoose — SchemaTypes:**  
https://mongoosejs.com/docs/schematypes.html

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

**MongoDB — Referenced One-to-Many Relationships:**  
https://www.mongodb.com/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/

---

# 🧠 39. Understand the Relationship

Consider:

```text
North Parking
```

It might contain:

```text
A01
A02
A03
A04
A05
```

This is a:

```text
one-to-many
```

relationship.

One:

```text
ParkingArea
```

can relate to many:

```text
ParkingBay
```

documents.

The ParkingBay stores the reference to its parent ParkingArea.

This allows the API to determine:

```text
Which parking area does this bay belong to?
```

---

# 📦 40. Create the ParkingBay Schema and Model

Design the ParkingBay schema.

You need to:

1. Create its fields.
2. Choose appropriate data types.
3. Make important fields required.
4. Restrict bay types.
5. Restrict status values.
6. Reference the ParkingArea model.
7. Enable timestamps.
8. Create and export the model.

Do not add reservation logic yet.

---

# 🧪 41. Test Creating a Parking Bay

First make sure a ParkingArea exists.

Copy its MongoDB:

```text
_id
```

Create a temporary ParkingBay document referencing that ParkingArea.

Example test data might resemble:

```json
{
  "parkingArea": "PASTE_A_REAL_PARKING_AREA_ID_HERE",
  "bayNumber": "A01",
  "floor": "Ground",
  "section": "A",
  "bayType": "standard",
  "status": "available"
}
```

Your actual ID will be different.

Confirm that the document is saved.

Then inspect it in MongoDB Compass.

---

# 🔍 42. Retrieve Related Data

Research Mongoose:

```text
populate()
```

Use it to understand how a ParkingBay reference can be replaced with selected information from the related ParkingArea when reading the data.

The relationship concept is:

```text
ParkingBay document

parkingArea:
68...

        -> populate

ParkingArea information
```

Do not automatically populate every relationship in every query.

Use it when the related information is actually needed.

### 📚 Resource

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

---

# 🧱 43. Prevent Duplicate Bay Numbers

Consider the following:

```text
North Parking
-> A01

North Parking
-> A01
```

This should not be allowed.

However:

```text
North Parking
-> A01

South Parking
-> A01
```

may be valid because the bays belong to different parking areas.

This means:

```text
bayNumber
```

cannot simply be globally unique.

The unique combination should be:

```text
parkingArea + bayNumber
```

Research **compound unique indexes**.

Do not simply add:

```text
unique
```

to `bayNumber` by itself.

### 📚 Helpful Resources

**MongoDB — Unique Indexes:**  
https://www.mongodb.com/docs/manual/core/index-unique/

**MongoDB — Compound Indexes:**  
https://www.mongodb.com/docs/manual/core/indexes/index-types/index-compound/

**Mongoose — Indexes:**  
https://mongoosejs.com/docs/guide.html#indexes

---

# 🧪 44. Test the Duplicate Rule

After implementing your compound uniqueness rule:

Create:

```text
North Parking
-> A01
```

Then attempt to create:

```text
North Parking
-> A01
```

again.

The duplicate should be rejected.

Now create:

```text
South Parking
-> A01
```

This should be allowed.

This test proves that you understand the difference between:

```text
Globally unique bay number
```

and:

```text
Unique bay number within a parking area
```

---

# 🧠 45. Understand What We Are NOT Creating Yet

At this point, you may be tempted to create:

```text
User

Reservation

QRPass
```

Do not.

These models depend on concepts we will introduce later.

### User

Will be introduced with:

```text
Firebase Authentication
+
Roles
```

### Reservation

Will be introduced when we work with:

```text
Dates
Times
Availability
Ownership
Conflict Detection
```

The goal is to learn each concept when it becomes relevant.

---

# 🧹 46. Remove Temporary Test Logic

Once you have proved that:

```text
ParkingArea can be created

ParkingArea can be retrieved

ParkingBay can be created

ParkingBay references ParkingArea
```

remove unnecessary temporary testing endpoints or scripts.

Do not leave routes such as:

```text
/api/create-random-test-stuff
```

inside the finished application.

Proper feature routes will be introduced later.

---

# 📁 47. Review Your Backend Structure

Your API should now resemble:

```text
api/
|
|-- src/
|   |
|   |-- config/
|   |   |
|   |   |-- database.js
|   |
|   |-- controllers/
|   |
|   |-- middleware/
|   |
|   |-- models/
|   |   |
|   |   |-- ParkingArea.js
|   |   |
|   |   |-- ParkingBay.js
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
|
|-- package-lock.json
```

You should **not** yet have:

```text
User.js

Reservation.js
```

unless instructed otherwise.

---

# 🧠 48. Make Sure You Understand the New Flow

At the end of Step 01, you had:

```text
API Client
-> Express
-> Response
```

You now have:

```text
API Client
-> Express
-> Mongoose Model
-> MongoDB
-> Mongoose
-> Express
-> API Response
```

This is the first major expansion of the ParkSmart backend.

---

# 🆚 49. MongoDB Terminology Review

Make sure you can explain these terms without simply reading their definitions.

| MongoDB Term | Meaning |
|---|---|
| Database | Contains collections for an application |
| Collection | Group of related documents |
| Document | One stored item/record |
| Field | Individual value within a document |
| `_id` | Unique identifier for a document |
| ObjectId | Common MongoDB identifier type |
| Schema | Mongoose definition of document structure |
| Model | Interface used to work with a collection |
| Validation | Rules controlling acceptable data |
| Reference | Link from one document to another |
| Index | Database structure used to support queries and constraints |

---

# 🐛 Common Problems

## MongoDB connection times out

Check:

- Your internet connection.
- Atlas network access.
- Your connection string.
- Your database deployment status.

---

## Authentication failed

Check:

- Database username.
- Database password.
- Connection string.

Remember that your MongoDB **database user** and your MongoDB website account are different concepts.

---

## Password contains special characters

Some characters inside connection strings require URL encoding.

Refer to MongoDB's connection-string documentation instead of randomly changing your password or connection URI.

**Connection String Documentation:**  
https://www.mongodb.com/docs/manual/reference/connection-string/

---

## `MONGODB_URI` is undefined

Check:

- `.env` exists.
- The variable is called exactly `MONGODB_URI`.
- `dotenv` is loaded.
- The application is started from the expected location.
- There are no spelling differences.

These are different:

```text
MONGODB_URI
```

```text
MONGO_URI
```

---

## `mongoose` cannot be found

Make sure your terminal is inside:

```text
ParkSmart/api/
```

Run:

```powershell
npm install
```

Check:

```text
package.json
```

for Mongoose.

---

## MongoDB collection does not appear

A collection may not appear until data has actually been stored.

Confirm that your create operation succeeded.

---

## Validation does not work

Check:

- The validator is attached to the correct field.
- The field uses the expected data type.
- You are creating data through the Mongoose model rather than bypassing it.

---

## `CastError` for ObjectId

This commonly means a value expected to be a MongoDB ObjectId is not in the expected format.

Check the ID that was supplied.

Do not invent an ID.

Use the `_id` of an actual stored document when testing relationships.

---

## `populate()` does not return Parking Area data

Check:

- The field contains a real referenced ObjectId.
- `ref` matches the correct Mongoose model name.
- The related document actually exists.

---

## Duplicate bay is still being created

Check:

- Your compound index definition.
- Whether the index exists in MongoDB.
- Whether you accidentally made only `bayNumber` unique.
- Whether both values in the combination are correct.

Inspect indexes using MongoDB Compass where helpful.

---

# 📚 Helpful Resources

## 🍃 MongoDB Basics

**MongoDB — Get Started:**  
https://www.mongodb.com/docs/get-started/

Start here if MongoDB is completely new to you.

**MongoDB — Documents:**  
https://www.mongodb.com/docs/manual/core/document/

Useful for understanding documents and fields.

**MongoDB — BSON Types:**  
https://www.mongodb.com/docs/manual/reference/bson-types/

Useful for understanding MongoDB data types and ObjectIds.

---

## ☁️ MongoDB Atlas

**MongoDB Atlas:**  
https://www.mongodb.com/atlas

**Deploy a Free Cluster:**  
https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/

**Connect Your Application:**  
https://www.mongodb.com/docs/atlas/connect-your-application/

**Configure IP Access:**  
https://www.mongodb.com/docs/atlas/security/ip-access-list/

---

## 🧭 MongoDB Compass

**MongoDB Compass:**  
https://www.mongodb.com/docs/compass/

**Install Compass:**  
https://www.mongodb.com/docs/compass/install/

**Connect Compass to MongoDB:**  
https://www.mongodb.com/docs/compass/connect/

Use Compass to visually inspect your ParkSmart database, collections and documents.

---

## 📦 Mongoose

**Mongoose — Getting Started:**  
https://mongoosejs.com/docs/

**Mongoose — Connections:**  
https://mongoosejs.com/docs/connections.html

**Mongoose — Schemas:**  
https://mongoosejs.com/docs/guide.html

**Mongoose — Models:**  
https://mongoosejs.com/docs/models.html

**Mongoose — SchemaTypes:**  
https://mongoosejs.com/docs/schematypes.html

---

## ✅ Validation

**Mongoose — Validation:**  
https://mongoosejs.com/docs/validation.html

Focus on:

```text
required

enum

min

max

default
```

---

## 🕒 Timestamps

**Mongoose — Timestamps:**  
https://mongoosejs.com/docs/timestamps.html

Use this to understand:

```text
createdAt

updatedAt
```

---

## 🔗 Relationships

**Mongoose — Populate:**  
https://mongoosejs.com/docs/populate.html

**MongoDB — Referenced One-to-Many Relationships:**  
https://www.mongodb.com/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/

Focus on:

```text
ObjectId

ref

populate()
```

---

## 🔎 Queries

**Mongoose — Queries:**  
https://mongoosejs.com/docs/queries.html

Useful for understanding how models retrieve MongoDB documents.

---

## 🧱 Indexes

**MongoDB — Unique Indexes:**  
https://www.mongodb.com/docs/manual/core/index-unique/

**MongoDB — Compound Indexes:**  
https://www.mongodb.com/docs/manual/core/indexes/index-types/index-compound/

Use these when implementing:

```text
parkingArea + bayNumber
-> unique combination
```

---

# 🔒 Security Checks

Before completing this step, confirm:

- Your MongoDB connection string is stored in `.env`.
- `.env` is not committed.
- Database credentials are not inside JavaScript files.
- Database credentials are not inside the README.
- `.env.example` does not contain private values.
- Your database password is not a password used for another personal account.
- MongoDB network access is configured appropriately.
- Android does not contain your MongoDB connection string.
- Android does not connect directly to MongoDB.

Remember:

```text
Android
-> API
-> MongoDB
```

Not:

```text
Android
-> MongoDB
```

---

# 🧪 Step 02 Testing Checklist

## 🍃 MongoDB

- [ ] A MongoDB database deployment is available.
- [ ] A database user has been created.
- [ ] Network access has been configured.
- [ ] You have obtained a connection string.
- [ ] The connection string is stored in `.env`.
- [ ] `.env.example` has been updated.

## 📦 Mongoose

- [ ] Mongoose is installed.
- [ ] `database.js` exists.
- [ ] The API connects successfully to MongoDB.
- [ ] A failed MongoDB connection produces useful information.

## 🏢 ParkingArea

- [ ] `ParkingArea.js` exists.
- [ ] The schema contains the required fields.
- [ ] Appropriate data types are used.
- [ ] Required fields are validated.
- [ ] Status values are restricted.
- [ ] Latitude is validated.
- [ ] Longitude is validated.
- [ ] Timestamps are enabled.
- [ ] A Parking Area can be saved.
- [ ] A Parking Area can be retrieved.
- [ ] Invalid Parking Area data is rejected.

## 🚙 ParkingBay

- [ ] `ParkingBay.js` exists.
- [ ] A bay references a ParkingArea.
- [ ] Bay types are restricted.
- [ ] Bay statuses are restricted.
- [ ] Timestamps are enabled.
- [ ] A Parking Bay can be saved.
- [ ] Related Parking Area information can be retrieved.
- [ ] Duplicate bay numbers within the same Parking Area are rejected.
- [ ] The same bay number can exist in different Parking Areas.

## 🔍 Database Inspection

- [ ] You can connect using MongoDB Compass or Atlas.
- [ ] You can locate the ParkSmart database.
- [ ] You can locate the Parking Area collection.
- [ ] You can locate the Parking Bay collection.
- [ ] You understand `_id`.
- [ ] You can identify the ParkingArea reference stored by a ParkingBay.

---

# ✅ Step 02 Complete

You started this step with:

```text
Express API
```

You now have:

```text
Express API
-> Mongoose
-> MongoDB
```

You also have your first ParkSmart data relationship:

```text
Parking Area
-> contains
Parking Bays
```

You have **not** yet created users.

That is intentional.

A ParkSmart user is connected to an authenticated Firebase identity, so we will introduce the User model at the same time as authentication.

In the next step, you will answer two important questions:

> **Who is using ParkSmart?**

and:

> **How can the API trust that the user really is who they claim to be?**

This introduces:

```text
Google Sign-In
-> Firebase Authentication
-> Firebase ID Token
-> Firebase Admin
-> ParkSmart User Profile
-> Roles
```

# ➡️ Continue to Step 03 — Authentication and Users
