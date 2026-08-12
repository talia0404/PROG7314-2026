# 🏗️ Step 01 — Project and API Setup

## 📚 What You Are Building

In this first step, you will create the foundation of the **ParkSmart** system.

You are **not building the complete application yet**.

The purpose of this step is to make sure that:

- Your Android project is ready.
- Your backend project is ready.
- Your Node.js server can run.
- Your Express API can receive requests.
- Your API can return JSON responses.
- Your project has a clear structure that can grow as more features are added.
- Sensitive and generated files are not committed to GitHub.

By the end of this step, the basic structure will be:

```text
ParkSmart
│
├── Android Application
│
└── REST API
```

The API will not connect to a database yet.

MongoDB and persistent data storage will be introduced in **Step 02**.

---

# 🎯 Learning Objectives

After completing this step, you should be able to:

- Explain the purpose of a REST API in a mobile application.
- Explain the role of Node.js.
- Explain the purpose of Express.
- Create a Node.js project using npm.
- Install and manage Node.js packages.
- Create an Express application.
- Create a basic REST endpoint.
- Understand the difference between an HTTP request and response.
- Return JSON from an API.
- Test API endpoints independently from Android.
- Organise a backend project into logical folders.
- Use environment variables for configuration.
- Protect sensitive and generated files using `.gitignore`.

---

# 🧠 1. Understand the ParkSmart Architecture

ParkSmart will eventually contain three major components:

```text
📱 Android Application
          ↓
🌐 REST API
          ↓
🍃 MongoDB Database
```

However, we are **not implementing all three components in this step**.

For now, we are focusing on:

```text
📱 Android Application

        +

🌐 REST API
```

The database will be added later.

---

## 📱 What Is the Android Application Responsible For?

The Android application is the part of ParkSmart that the user interacts with.

It will eventually be responsible for:

- Displaying screens.
- Collecting user input.
- Signing users in.
- Displaying parking areas.
- Displaying available parking bays.
- Allowing drivers to create reservations.
- Displaying QR parking passes.
- Allowing managers to scan QR codes.

However, the Android application should **not contain all of the application's business logic**.

It also should not communicate directly with the database.

---

## 🌐 What Is the REST API Responsible For?

The REST API sits between the Android application and the application's data.

Later, the architecture will become:

```text
Android
   ↓
REST API
   ↓
MongoDB
```

The API will eventually be responsible for:

- Receiving requests from Android.
- Validating data.
- Authenticating users.
- Authorising actions.
- Applying ParkSmart business rules.
- Reading data from MongoDB.
- Saving data to MongoDB.
- Checking parking availability.
- Creating reservations.
- Validating QR parking passes.

For now, we only need to prove that the API can:

```text
Receive Request
      ↓
Process Request
      ↓
Return Response
```

---

# 🟢 2. Check That Node.js Is Installed

ParkSmart's backend will use **Node.js**.

Node.js allows JavaScript to run outside of a web browser.

This means JavaScript can be used to create applications such as:

- Web servers.
- REST APIs.
- Command-line applications.
- Backend services.

Check whether Node.js is already installed.

Open a terminal and run:

```powershell
node --version
```

You should receive a version number.

Then check npm:

```powershell
npm --version
```

You should also receive a version number.

---

## 📦 What Is npm?

**npm** is the package manager used by Node.js.

Instead of manually downloading libraries, npm allows us to install them using commands.

For example:

```powershell
npm install express
```

npm will also keep track of the packages required by the project.

---

## 📥 If Node.js Is Not Installed

Download an **LTS version** of Node.js from:

**Node.js — Download**

https://nodejs.org/en/download

After installing Node.js:

1. Close your terminal.
2. Open a new terminal.
3. Run:

```powershell
node --version
```

4. Run:

```powershell
npm --version
```

Both commands must work before continuing.

### 📚 Helpful Resource

**Node.js — Introduction to Node.js**

https://nodejs.org/en/learn/getting-started/introduction-to-nodejs

---

# 📁 3. Prepare Your ParkSmart Projects

ParkSmart contains two separate applications:

```text
ParkSmart
│
├── Android Application
│
└── Backend API
```

Even though these applications work together, they perform different jobs.

You will work on both throughout this activity.

---

# 📱 4. Create the Android Project

Open **Android Studio**.

Create a new project:

```text
New Project
→ Empty Activity
```

Configure the project appropriately.

Suggested configuration:

```text
Name:
ParkSmart

Language:
Kotlin

UI:
Jetpack Compose

Minimum SDK:
API 24 or later
```

Choose an appropriate unique package name.

For example:

```text
com.yourname.parksmart
```

Do not simply copy another student's package name.

---

## ▶️ Test the Android Project

Before doing anything else to the Android project:

1. Allow Gradle to finish syncing.
2. Select an emulator or physical Android device.
3. Run the application.
4. Confirm that the default application opens.

You are **not required to build the ParkSmart interface yet**.

The purpose of this check is simply to confirm that the Android project works before additional functionality is introduced.

### ✅ Android Checkpoint

Before continuing:

- [ ] The Android project exists.
- [ ] Gradle Sync completes successfully.
- [ ] The project builds.
- [ ] The application launches.
- [ ] There are no unresolved project configuration errors.

---

# 🌐 5. Create the Backend API Project

Now create a separate folder for the ParkSmart API.

For example:

```text
ParkSmartAPI
```

Open this folder in **Visual Studio Code**.

Open the integrated terminal:

```text
Terminal
→ New Terminal
```

Make sure the terminal is currently inside your API project folder.

You can check the current location using:

```powershell
pwd
```

or inspect the path displayed in the terminal.

---

# 📦 6. Initialise the Node.js Project

Inside the API folder, run:

```powershell
npm init -y
```

This creates:

```text
package.json
```

Your project should now contain:

```text
ParkSmartAPI/
│
└── package.json
```

---

# 📄 7. Understand `package.json`

`package.json` describes your Node.js project.

Open it and inspect its contents.

It contains information such as:

- Project name.
- Version.
- Scripts.
- Dependencies.
- Development dependencies.

As packages are installed, npm records them here.

This means another developer does not need you to send them every installed package manually.

They can clone the project and later run:

```powershell
npm install
```

npm reads `package.json` and installs the required dependencies.

### 📚 Helpful Resource

**npm — package.json**

https://docs.npmjs.com/cli/configuring-npm/package-json

---

# 🌐 8. Install Express

ParkSmart will use **Express** to create the REST API.

Run:

```powershell
npm install express
```

After installation, inspect your project.

You should notice:

```text
node_modules/
package.json
package-lock.json
```

---

## 🧠 What Is Express?

Node.js gives us the ability to run JavaScript on the server.

Express provides tools that make building web servers and APIs easier.

Express helps us handle things such as:

```text
GET requests
POST requests
PUT requests
PATCH requests
DELETE requests
Routes
Request bodies
Responses
Middleware
```

For example, ParkSmart will eventually have endpoints resembling:

```text
GET /api/parking-areas

GET /api/reservations

POST /api/reservations

PATCH /api/reservations/:id/cancel
```

You are **not creating these endpoints yet**.

### 📚 Helpful Resources

**Express — Getting Started**

https://expressjs.com/en/starter/installing.html

**Express — Basic Routing**

https://expressjs.com/en/starter/basic-routing.html

**Express — Routing Guide**

https://expressjs.com/en/guide/routing.html

---

# 🔄 9. Install Nodemon

During development, you will frequently modify your backend files.

Normally, after changing your code, you would need to:

```text
Change Code
    ↓
Stop Server
    ↓
Start Server Again
```

Nodemon can automatically restart the server when your files change.

Install it as a development dependency:

```powershell
npm install --save-dev nodemon
```

---

## 🧠 Why Is It a Development Dependency?

Nodemon helps us **develop** the application.

The finished API does not require Nodemon to perform its actual job.

This is why it is installed using:

```text
--save-dev
```

Open `package.json`.

You should now see Express and Nodemon recorded separately under the appropriate dependency sections.

### 📚 Helpful Resource

**Nodemon**

https://www.npmjs.com/package/nodemon

---

# 🔐 10. Install dotenv

Install:

```powershell
npm install dotenv
```

`dotenv` allows the application to load configuration values from a `.env` file.

Later, ParkSmart will contain configuration values such as:

```text
PORT
MongoDB connection information
Firebase configuration
```

These values should not be scattered throughout your source code.

We will configure the `.env` file later in this step.

### 📚 Helpful Resource

**dotenv**

https://www.npmjs.com/package/dotenv

---

# 📁 11. Create the Backend Folder Structure

Inside the API project, create:

```text
src/
```

Inside `src`, create the following folders:

```text
config/
controllers/
middleware/
models/
routes/
services/
utils/
```

Also create:

```text
app.js
server.js
```

Your structure should now resemble:

```text
ParkSmartAPI/
│
├── src/
│   │
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   ├── utils/
│   │
│   ├── app.js
│   └── server.js
│
├── node_modules/
├── package.json
└── package-lock.json
```

Some of these folders will remain empty during Step 01.

That is expected.

They are being created now so that the API has a clear structure as it grows.

---

# 🧠 12. Understand the Backend Structure

Do not create folders without understanding why they exist.

---

## ⚙️ `config/`

This folder will contain configuration code.

Later this may include:

```text
MongoDB connection
Firebase Admin configuration
```

---

## 🛣️ `routes/`

Routes define the endpoints that clients can access.

For example:

```text
GET /api/parking-areas
```

A route helps determine what should happen when a particular request reaches the API.

---

## 🎮 `controllers/`

Controllers receive requests and coordinate the response.

A simplified flow might be:

```text
Request
   ↓
Route
   ↓
Controller
   ↓
Response
```

As ParkSmart becomes more complicated, controllers will work with other parts of the application.

---

## 🧠 `services/`

Services are useful for separating business logic from HTTP request-handling code.

Later, ParkSmart may need to perform logic such as:

```text
Receive Reservation Request
          ↓
Validate Parking Area
          ↓
Validate Parking Bay
          ↓
Check Date and Time
          ↓
Check Availability
          ↓
Create Reservation
```

We do not want all of that logic sitting directly inside a route.

---

## 🛡️ `middleware/`

Middleware contains functions that can run while an HTTP request is being processed.

Later, middleware will help with:

```text
Authentication
Authorisation
Error handling
Request processing
```

---

## 📦 `models/`

Models will later represent the structure of the data stored in MongoDB.

Do not create any ParkSmart models yet.

MongoDB and models will be covered in **Step 02**.

---

## 🧰 `utils/`

This folder can contain reusable helper functionality that does not clearly belong to another layer.

---

## 🌐 `app.js`

`app.js` will configure the Express application.

It is responsible for things such as:

```text
Creating Express
Configuring middleware
Registering routes
```

---

## ▶️ `server.js`

`server.js` is responsible for starting the application.

Separating the Express configuration from server startup will also make the application easier to test later.

---

# 🌐 13. Create the Express Application

Open:

```text
src/app.js
```

Your first task is to create the Express application.

You need to research and implement the following:

1. Import Express.
2. Create an Express application.
3. Configure Express to understand JSON request bodies.
4. Export the Express application so that `server.js` can use it.

You will need to investigate:

```text
express()
express.json()
module.exports
```

or the equivalent syntax if you have configured your project to use ES modules.

> ⚠️ Choose one module system and use it consistently. Do not randomly mix `require()` and `import` syntax.

### 📚 Resources

**Express — Using Middleware**

https://expressjs.com/en/guide/using-middleware.html

**Express API Reference**

https://expressjs.com/en/api.html

Look specifically at:

```text
express()
express.json()
```

---

# 🧪 14. Create Your First Endpoint

Before connecting Android, Firebase or a database, prove that your API works independently.

Create a simple endpoint:

```text
GET /api/test
```

When the endpoint receives a GET request, it should return JSON.

For example:

```json
{
  "message": "ParkSmart API is running"
}
```

---

## 🧠 What Is an Endpoint?

An endpoint is a specific location exposed by an API.

It normally combines:

```text
HTTP Method + Path
```

For example:

```text
GET + /api/test
```

Later:

```text
GET + /api/parking-areas

POST + /api/reservations
```

These are different endpoints because they represent different operations.

---

# 📥 15. Understand the Request

When you access:

```text
GET /api/test
```

you are sending an **HTTP request** to the API.

A request can contain information such as:

```text
HTTP method
URL
Headers
Parameters
Query values
Request body
```

Not every request uses all of these.

For your first endpoint, the important parts are:

```text
Method:
GET

Path:
/api/test
```

---

# 📤 16. Understand the Response

The API processes the request and sends an HTTP response.

A response can contain:

```text
Status code
Headers
Body
```

Your response body will contain JSON.

For example:

```json
{
  "message": "ParkSmart API is running"
}
```

---

# 🔢 17. Understand HTTP Status Codes

HTTP status codes communicate the outcome of a request.

You will use these throughout ParkSmart.

Some common examples are:

| Code | Meaning |
|---|---|
| `200` | Request succeeded |
| `201` | Resource created successfully |
| `400` | Invalid request |
| `401` | Authentication required or invalid |
| `403` | User is authenticated but not authorised |
| `404` | Resource not found |
| `409` | Conflict with existing data/state |
| `500` | Unexpected server error |

You do not need to implement all of these now.

For your test route, a successful request should return an appropriate success response.

### 📚 Helpful Resource

**MDN — HTTP Response Status Codes**

https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

---

# ▶️ 18. Create `server.js`

Open:

```text
src/server.js
```

This file must start the Express server.

Your implementation should:

1. Load the Express application from `app.js`.
2. Determine which port the application should use.
3. Tell Express to listen on that port.
4. Display a useful terminal message when the server starts.

For now, your server should eventually display something similar to:

```text
ParkSmart API running on port 3000
```

Research the Express `listen()` method.

### 📚 Helpful Resource

**Express — Hello World Example**

https://expressjs.com/en/starter/hello-world.html

Pay attention to:

```text
app.listen()
```

---

# 🔐 19. Create the Environment File

At the root of the API project, create:

```text
.env
```

For now, it only needs:

```env
PORT=3000
```

Later, additional configuration will be added.

---

## 🧠 Why Use Environment Variables?

Avoid spreading configuration throughout the source code.

For example, instead of permanently writing:

```text
Port = 3000
```

inside application logic, the value can come from the environment:

```text
.env
  ↓
PORT
  ↓
server.js
```

This means configuration can change without changing application logic.

---

# ⚙️ 20. Load the Environment Configuration

Configure the project so that the value from:

```text
.env
```

can be accessed by the Node application.

Research how `dotenv` loads environment variables.

Your application should be able to access:

```text
PORT
```

through Node's environment.

### 📚 Helpful Resources

**dotenv — Documentation**

https://www.npmjs.com/package/dotenv

**Node.js — Environment Variables**

https://nodejs.org/en/learn/command-line/how-to-read-environment-variables-from-nodejs

---

# 📄 21. Create `.env.example`

Create another file:

```text
.env.example
```

Add:

```env
PORT=
```

Do not place private values inside this file.

---

## `.env` vs `.env.example`

### `.env`

Contains the actual values used on your computer.

For example:

```env
PORT=3000
```

This file should **not** be committed.

### `.env.example`

Shows another developer which configuration variables the project expects.

For example:

```env
PORT=
```

This file **can** be committed.

Later, if another developer clones ParkSmart, they can see:

> "I need to create a `PORT` environment variable."

without receiving your private configuration.

---

# 📜 22. Add Scripts to `package.json`

You should not need to type the full path to `server.js` every time you want to run the application.

Open:

```text
package.json
```

Find:

```json
"scripts"
```

Configure scripts for:

```text
start
dev
```

Your scripts should allow you to use:

```powershell
npm start
```

to start the application normally.

And:

```powershell
npm run dev
```

to start the application using Nodemon.

The commands associated with the scripts should run:

```text
src/server.js
```

---

# ▶️ 23. Run the API

In the VS Code terminal, run:

```powershell
npm run dev
```

If everything is configured correctly, you should see a message similar to:

```text
ParkSmart API running on port 3000
```

Do not continue if the server cannot start.

Read the terminal output carefully and fix the error first.

---

# 🧪 24. Test the API in a Browser

Because your first endpoint uses `GET`, you can perform a quick test using a browser.

Navigate to:

```text
http://localhost:3000/api/test
```

You should receive your JSON response.

For example:

```json
{
  "message": "ParkSmart API is running"
}
```

If this works, you have successfully created your first REST API endpoint.

---

# 🧰 25. Test the API Using an API Client

A browser is useful for simple `GET` requests, but it is not sufficient for properly testing a REST API.

As ParkSmart grows, you will need to send:

```text
GET
POST
PUT
PATCH
DELETE
```

requests.

You may use an API client such as:

- Postman
- Bruno
- Insomnia
- Thunder Client for VS Code

Your lecturer may specify which tool should be used.

---

## Example Test

Create a request using:

```text
Method:
GET

URL:
http://localhost:3000/api/test
```

Send the request.

Confirm:

- The request succeeds.
- The expected status code is returned.
- The response body contains JSON.

### 📚 Helpful Resources

**Postman — Send Your First API Request**

https://learning.postman.com/docs/getting-started/first-steps/sending-the-first-request/

**Bruno — Documentation**

https://docs.usebruno.com/

**Thunder Client**

https://www.thunderclient.com/

---

# 🚫 26. Create `.gitignore`

Your project contains files that should not be uploaded to GitHub.

Create:

```text
.gitignore
```

at the appropriate repository level.

At minimum, your backend should ignore:

```gitignore
# Node dependencies
node_modules/

# Environment variables
.env

# Logs
*.log
```

Your Android project should also avoid committing local/generated files such as:

```gitignore
.gradle/
local.properties
**/build/
```

Depending on how your repository is structured, Android Studio may already have generated suitable ignore rules.

Do not blindly replace an existing `.gitignore`.

Review it first and add anything that is missing.

---

# 🧠 27. Why Don't We Commit `node_modules`?

The `node_modules` folder can contain thousands of generated dependency files.

Those dependencies are already described by:

```text
package.json
package-lock.json
```

Therefore, another developer can clone the project and run:

```powershell
npm install
```

to restore the required dependencies.

The repository does not need to store the entire `node_modules` folder.

---

# 🔐 28. Why Don't We Commit `.env`?

The `.env` file may eventually contain sensitive configuration.

For example:

```text
Database connection details
Private credentials
Service configuration
```

Even though the file currently contains only a port, it is good practice to protect it from the beginning.

Later, you should not suddenly have to remember:

> "Oh. That file has secrets now. Maybe I shouldn't have been committing it for three weeks." 😭

---

# 🔎 29. Check Git Before Committing

Before committing your work, run:

```powershell
git status
```

Review the files Git intends to track.

Make sure you are **not** committing:

```text
node_modules/
.env
local.properties
build folders
```

Do not assume `.gitignore` works simply because the file exists.

Check.

---

# 🧪 30. Test an Invalid Endpoint

Your API works when you request:

```text
/api/test
```

Now deliberately request something that does not exist.

For example:

```text
/api/does-not-exist
```

Observe what happens.

Ask yourself:

- What status code was returned?
- Did Express return a response?
- Is that response appropriate for an API?
- Would an Android application know what went wrong?

You will improve API error handling as ParkSmart develops.

For now, the important goal is to start thinking about **successful and unsuccessful requests**.

---

# 🔄 31. Understand What You Have Built

At this point, your system looks like this:

```text
API Client
    │
    │ HTTP GET
    ↓
Express API
    │
    │ Processes /api/test
    ↓
JSON Response
```

For example:

```text
GET /api/test
       ↓
Express
       ↓
{
  "message": "ParkSmart API is running"
}
```

You have **not connected Android to the API yet**.

You have **not added MongoDB yet**.

You have **not added authentication yet**.

That is intentional.

Each part will be introduced separately.

---

# 📂 32. Check Your Backend Structure

By the end of Step 01, your API should resemble:

```text
ParkSmartAPI/
│
├── src/
│   │
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   ├── utils/
│   │
│   ├── app.js
│   └── server.js
│
├── .env
├── .env.example
├── .gitignore
├── package.json
└── package-lock.json
```

Remember:

```text
config/
controllers/
middleware/
models/
routes/
services/
utils/
```

may still be empty.

Do not add unnecessary code simply to fill the folders.

---

# 🧪 Testing Checklist

Before moving on, perform the following tests.

## 📱 Android Project

- [ ] The Android project opens successfully.
- [ ] Gradle Sync succeeds.
- [ ] The application builds.
- [ ] The application launches on an emulator or physical device.

---

## 🟢 Node.js

Run:

```powershell
node --version
```

- [ ] A Node.js version is displayed.

Run:

```powershell
npm --version
```

- [ ] An npm version is displayed.

---

## 🌐 API

Run:

```powershell
npm run dev
```

- [ ] The server starts.
- [ ] Nodemon runs successfully.
- [ ] The correct port is displayed.

Send:

```text
GET /api/test
```

- [ ] The request succeeds.
- [ ] An appropriate success status is returned.
- [ ] JSON is returned.
- [ ] The expected message is returned.

Send a request to an invalid endpoint.

- [ ] You observed how the API handles an unknown route.

---

## 🔐 Project Configuration

- [ ] `.env` exists.
- [ ] `.env.example` exists.
- [ ] `PORT` is read from the environment.
- [ ] `.gitignore` exists.
- [ ] `.env` is ignored.
- [ ] `node_modules/` is ignored.
- [ ] Android-generated files are appropriately ignored.

---

# 🐛 Common Problems

## `node` Is Not Recognised

If:

```powershell
node --version
```

fails, Node.js may not be installed correctly or may not be available through your system PATH.

After installing Node.js:

1. Close the terminal.
2. Open a new terminal.
3. Try again.

---

## `npm` Is Not Recognised

npm is normally installed with Node.js.

Confirm that Node.js installed successfully and reopen your terminal.

---

## `Cannot find module 'express'`

Make sure you installed Express:

```powershell
npm install express
```

Also make sure your terminal is inside the correct project folder.

---

## `Cannot find module 'dotenv'`

Install dotenv:

```powershell
npm install dotenv
```

---

## Nodemon Does Not Work

Make sure it was installed:

```powershell
npm install --save-dev nodemon
```

Then use:

```powershell
npm run dev
```

rather than assuming `nodemon` is globally installed.

---

## `PORT` Is Undefined

Check:

- The file is named exactly `.env`.
- `.env` is in the correct location.
- dotenv is loaded before the environment value is accessed.
- The variable is named exactly `PORT`.

Environment-variable names must match.

```text
PORT
```

is not the same as:

```text
Port
```

---

## `Cannot GET /api/test`

Check:

- The server is running.
- The URL is correct.
- The route uses `GET`.
- The route path is correct.
- `app.js` is being used by `server.js`.
- You saved your files.
- Nodemon restarted successfully.

---

## Port Already in Use

Another application may already be using the configured port.

Read the terminal error.

You can:

- Stop the other process, or
- Configure ParkSmart to use another suitable port.

Do not randomly change several settings at once. Identify the actual cause first.

---

# 📚 Helpful Resources

## 🟢 Node.js

**Node.js — Download**

https://nodejs.org/en/download

**Node.js — Introduction**

https://nodejs.org/en/learn/getting-started/introduction-to-nodejs

---

## 📦 npm

**npm — package.json**

https://docs.npmjs.com/cli/configuring-npm/package-json

**npm — Installing Packages**

https://docs.npmjs.com/downloading-and-installing-packages-locally

---

## 🌐 Express

**Express — Installing**

https://expressjs.com/en/starter/installing.html

**Express — Hello World**

https://expressjs.com/en/starter/hello-world.html

**Express — Basic Routing**

https://expressjs.com/en/starter/basic-routing.html

**Express — Routing**

https://expressjs.com/en/guide/routing.html

**Express — Using Middleware**

https://expressjs.com/en/guide/using-middleware.html

---

## 🔐 Environment Variables

**Node.js — Environment Variables**

https://nodejs.org/en/learn/command-line/how-to-read-environment-variables-from-nodejs

**dotenv**

https://www.npmjs.com/package/dotenv

---

## 🌍 HTTP

**MDN — Overview of HTTP**

https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview

**MDN — HTTP Request Methods**

https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods

**MDN — HTTP Status Codes**

https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

---

## 🧪 API Testing

**Postman — Send Your First Request**

https://learning.postman.com/docs/getting-started/first-steps/sending-the-first-request/

**Bruno — Documentation**

https://docs.usebruno.com/

**Thunder Client**

https://www.thunderclient.com/

---

## 💾 Git

**GitHub — Ignoring Files**

https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files

**GitHub — About Git**

https://docs.github.com/en/get-started/using-git/about-git

---

# ✅ Before Moving On

Do **not** continue to Step 02 simply because you reached the bottom of this page.

Confirm that you can actually demonstrate each of the following:

- [ ] My Android project builds and runs.
- [ ] I can explain what the Android application is responsible for.
- [ ] I can explain what the REST API is responsible for.
- [ ] Node.js is installed.
- [ ] npm is installed.
- [ ] My Node.js project has been initialised.
- [ ] Express is installed.
- [ ] Nodemon is installed.
- [ ] dotenv is installed.
- [ ] I understand the purpose of `package.json`.
- [ ] I understand the purpose of `package-lock.json`.
- [ ] My backend folder structure has been created.
- [ ] I understand the purpose of the main backend folders.
- [ ] `app.js` configures my Express application.
- [ ] `server.js` starts my server.
- [ ] My port is read from an environment variable.
- [ ] `npm run dev` starts the API.
- [ ] `GET /api/test` works.
- [ ] My API returns JSON.
- [ ] I can test my API independently from Android.
- [ ] `.env` is not being tracked by Git.
- [ ] `node_modules/` is not being tracked by Git.
- [ ] No private configuration has been committed.

If all of these checks pass, your ParkSmart foundation is ready.

---

# ➡️ Next: Step 02 — MongoDB and Backend Data

In the next step, you will answer a problem that your current API has:

> **What happens when ParkSmart needs to remember something after the server stops?**

You will introduce:

```text
MongoDB
    ↓
Collections and Documents
    ↓
Mongoose
    ↓
Schemas
    ↓
Models
    ↓
Validation
    ↓
Relationships
```

Your architecture will then grow from:

```text
Client
   ↓
Express API
```

to:

```text
Client
   ↓
Express API
   ↓
Mongoose
   ↓
MongoDB
```

Do not begin Step 02 until your API is working correctly.
````
