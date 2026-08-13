# 🏗️ Step 01 — Project and API Setup

## 📚 What You Are Building

Welcome to the first development step of **ParkSmart**.

Before we can add authentication, parking data, reservations or QR codes, we need to create the foundation of the system.

In this step, you will create and configure:

- The ParkSmart Android application.
- The ParkSmart Node.js backend.
- An Express REST API.
- A structured backend project.
- Environment configuration.
- Git ignore rules.
- A simple API endpoint for testing.

At the end of this step, you should have:

```text
Android Application
        +
Node.js / Express API
        +
Working Test Endpoint
```

You are **not** connecting Android to the API yet.

You are also **not** adding a database yet.

Those features will be introduced in later steps.

---

# 🎯 Learning Objectives

After completing this step, you should be able to:

- Explain the purpose of a REST API.
- Explain the role of Node.js in ParkSmart.
- Explain the purpose of Express.
- Explain the difference between the Android application and the backend API.
- Create a Node.js project.
- Understand the purpose of `package.json`.
- Install and manage npm packages.
- Create an Express application.
- Create a simple API endpoint.
- Understand basic HTTP methods.
- Organise a backend project into logical folders.
- Use environment variables for application configuration.
- Use `.gitignore` to prevent unnecessary or private files from being committed.
- Run and test a local REST API.

---

# 🧠 1. Understand What You Are Creating

ParkSmart will eventually consist of several technologies working together.

The final architecture will resemble:

```text
Android Application
        ->
REST API
        ->
Database
```

For now, we are only concerned with:

```text
Android Application

and

REST API
```

These are **two separate applications**.

---

## 📱 Android Application

The Android application is the client.

It will eventually be responsible for:

- Displaying the ParkSmart interface.
- Collecting user input.
- Signing users in.
- Displaying parking areas.
- Displaying available parking bays.
- Sending reservation requests.
- Displaying QR parking passes.
- Scanning QR codes.

The Android application will be developed using:

```text
Kotlin
+
Jetpack Compose
```

---

## 🌐 REST API

The REST API is the backend of ParkSmart.

It will eventually be responsible for:

- Receiving requests from Android.
- Authenticating users.
- Authorising actions.
- Validating data.
- Retrieving data.
- Creating reservations.
- Checking parking availability.
- Preventing conflicting reservations.
- Validating QR parking passes.
- Communicating with the database.

For this project, the API will be developed using:

```text
Node.js
+
Express
```

---

# 🤔 2. Why Do We Need an API?

It might be tempting to place everything inside the Android application.

For example:

```text
Android
        ->
Database
```

However, this would give the Android client too much responsibility.

Instead, ParkSmart will use:

```text
Android
        ->
API
        ->
Database
```

The API acts as the controlled middle layer between the mobile application and the application's data.

Later, when a driver attempts to reserve a parking bay, Android might send a request similar to:

```text
POST /api/reservations
```

The API will then be responsible for deciding whether that request is actually valid.

For example, it may need to check:

- Is the user authenticated?
- Does the parking area exist?
- Does the parking bay exist?
- Is the parking bay active?
- Is the requested time valid?
- Is another reservation already using the bay?
- Is the user allowed to perform this action?

The Android application requests an action.

The API decides whether that action should be allowed.

---

# 🌐 3. Understand REST APIs

A REST API exposes **endpoints** that clients can send requests to.

An endpoint consists of:

```text
HTTP Method + Path
```

For example:

```text
GET /api/parking-areas
```

The HTTP method describes the type of action being requested.

Common HTTP methods include:

| Method | Typical Purpose |
|---|---|
| `GET` | Retrieve data |
| `POST` | Create new data |
| `PUT` | Replace or update data |
| `PATCH` | Partially update data |
| `DELETE` | Remove data |

ParkSmart will eventually contain endpoints such as:

```text
GET /api/parking-areas

GET /api/parking-areas/:id

POST /api/reservations

GET /api/reservations/my

PATCH /api/reservations/:id/cancel
```

You are **not creating these endpoints yet**.

In this step, you will create only a simple test endpoint to prove that the API is working.

---

# 🧰 4. Software You Will Need

Before starting, make sure you have the following installed.

## Android Development

You should already have:

- Android Studio.
- Android SDK.
- An Android emulator or physical Android device.
- Git.

## Backend Development

You will need:

- Node.js.
- npm.
- Visual Studio Code.
- A web browser.
- An API testing tool.

You may use an API testing tool such as:

- Postman.
- Bruno.
- Insomnia.
- Thunder Client for VS Code.

You only need **one** API testing tool.

---

# 🟢 5. Install Node.js

Node.js allows JavaScript to execute outside a web browser.

In ParkSmart, Node.js will run our backend application.

Download Node.js from the official website:

**Node.js Downloads:**  
https://nodejs.org/en/download

Use an **LTS release** unless instructed otherwise.

LTS stands for:

```text
Long-Term Support
```

These releases are generally preferred for projects where stability is important.

---

## Verify the Installation

After installing Node.js, close and reopen your terminal.

Open:

```text
PowerShell
```

or the terminal inside VS Code.

Run:

```powershell
node --version
```

You should receive a version number.

For example:

```text
vXX.XX.X
```

Then run:

```powershell
npm --version
```

You should also receive a version number.

If both commands work, Node.js and npm are available on your computer.

---

# 📦 6. Understand npm

Node.js includes **npm**, the Node Package Manager.

npm allows us to install libraries created by other developers.

Instead of writing an entire web server framework ourselves, we can install Express.

Later, we will install additional packages for features such as:

```text
Database communication

Authentication

Security

Testing
```

npm also records the packages required by the project so that another developer can install the same dependencies.

---

# 📂 7. Create the ParkSmart Project

Create a main folder for the project.

For example:

```text
ParkSmart/
```

Your project will eventually contain separate areas for the Android application and backend API.

A suitable structure is:

```text
ParkSmart/
|
|-- android/
|
|-- api/
|
|-- README.md
|
|-- .gitignore
```

The exact location of the repository on your computer is your choice.

---

# 📱 8. Create the Android Project

Open Android Studio.

Select:

```text
New Project
-> Empty Activity
```

Create your ParkSmart Android application.

Suggested configuration:

```text
Name:
ParkSmartAndroid

Language:
Kotlin

UI:
Jetpack Compose

Minimum SDK:
API 24 or later
```

Choose an appropriate package name for your project.

For example:

```text
com.yourname.parksmart
```

Do not use another student's package name.

---

## Run the Android Application

Allow Android Studio to complete the Gradle Sync.

Then run the application using:

- An Android emulator, or
- A physical Android device.

You do not need to build the ParkSmart interface yet.

At this stage, you are only confirming that:

```text
Project created
        ->
Gradle builds
        ->
Application launches
```

---

## ✅ Android Checkpoint

Before continuing, confirm:

- [ ] The Android project has been created.
- [ ] Gradle Sync completes successfully.
- [ ] The application builds.
- [ ] The application launches.
- [ ] There are no unresolved build errors.

Do not spend time designing the ParkSmart interface yet.

---

# 🌐 9. Create the Backend Project

Now create the backend application.

Navigate to:

```text
ParkSmart/api/
```

Open this folder in Visual Studio Code.

Your VS Code workspace should currently be almost empty.

Open the integrated terminal:

```text
Terminal
-> New Terminal
```

Make sure the terminal is inside the `api` folder.

You can check the current directory using:

```powershell
pwd
```

---

# 📦 10. Initialise the Node.js Project

Inside the `api` folder, run:

```powershell
npm init -y
```

This initialises a Node.js project.

The `-y` option accepts the default values automatically.

After running the command, a new file should appear:

```text
package.json
```

The official npm documentation describes `npm init` as the command used to create or initialise a package.  
https://docs.npmjs.com/cli/commands/npm-init

---

# 📄 11. Understand `package.json`

Open:

```text
package.json
```

Do not immediately start changing everything inside it.

First understand what the file represents.

`package.json` describes your Node.js project.

It can contain information such as:

```text
Project name
Project version
Entry point
Scripts
Dependencies
Development dependencies
```

As you install packages, npm will update this file.

This means another developer does not need you to send them every installed library manually.

They can clone your project and use the information in `package.json` to install the required packages.

---

# 📦 12. Install Express

ParkSmart will use **Express** as its web framework.

Express provides functionality for:

- Creating the web server.
- Defining routes.
- Receiving HTTP requests.
- Returning HTTP responses.
- Processing JSON.
- Using middleware.

Install Express by running:

```powershell
npm install express
```

The official Express installation guide uses npm to install Express into a Node project.

**Express Installation:**  
https://expressjs.com/en/starter/installing.html

---

# 🔍 13. Check What npm Created

After installing Express, examine the `api` folder.

You should now see:

```text
api/
|
|-- node_modules/
|
|-- package-lock.json
|
|-- package.json
```

Each item has a different purpose.

---

## `package.json`

Describes your application and lists its dependencies.

---

## `package-lock.json`

Records the exact dependency versions installed for the project.

This helps developers install consistent dependency versions.

You should normally commit:

```text
package-lock.json
```

to Git.

---

## `node_modules/`

Contains the actual installed packages.

This folder can become very large.

You should **not** commit:

```text
node_modules/
```

to GitHub.

Another developer can recreate it using:

```powershell
npm install
```

This is why `package.json` and `package-lock.json` are important.

---

# ⚡ 14. Install Nodemon

During development, you will frequently change backend files.

Normally, after changing the server code, you may need to stop and restart Node.

For example:

```text
Edit code
-> Save
-> Stop server
-> Start server again
```

Nodemon makes this easier.

It watches your project for changes and automatically restarts the Node application.

Install Nodemon as a **development dependency**:

```powershell
npm install --save-dev nodemon
```

The `--save-dev` option tells npm that Nodemon is a development tool rather than a package required by the production application itself.

After installation, examine `package.json`.

You should now see a distinction between:

```text
dependencies
```

and:

```text
devDependencies
```

---

# 📁 15. Create the Backend Folder Structure

Create a folder called:

```text
src
```

inside the `api` project.

Inside `src`, create the following folders:

```text
config
controllers
middleware
models
routes
services
utils
```

Also create:

```text
app.js
server.js
```

Your backend should now resemble:

```text
api/
|
|-- src/
|   |
|   |-- config/
|   |
|   |-- controllers/
|   |
|   |-- middleware/
|   |
|   |-- models/
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
|-- node_modules/
|
|-- package-lock.json
|
|-- package.json
```

Some folders will remain empty during Step 01.

That is completely fine.

They will be used as ParkSmart grows.

---

# 🧠 16. Understand the Backend Structure

Before adding anything else, understand why these folders exist.

The aim is to avoid building the entire backend inside one enormous file.

---

## ⚙️ `config/`

Contains configuration code.

Later, this may contain configuration for:

```text
MongoDB

Firebase Admin
```

---

## 🧭 `routes/`

Defines the API endpoints available to clients.

For example, later you may have routes relating to:

```text
Users

Parking Areas

Parking Bays

Reservations
```

A route determines which part of the application should handle a particular request.

---

## 🎮 `controllers/`

Controllers receive requests and coordinate the appropriate response.

A typical flow will eventually look like:

```text
Request
-> Route
-> Controller
-> Response
```

As ParkSmart becomes more complicated, controllers may also call services.

---

## 🧠 `services/`

Services contain more complex application and business logic.

For example, reservation creation will eventually require several checks.

Rather than putting all of this inside the route, the application can separate the responsibilities.

A later flow may resemble:

```text
Request
-> Route
-> Controller
-> Service
-> Database
```

---

## 🛡️ `middleware/`

Middleware runs during the request/response process.

Later, ParkSmart will use middleware for features such as:

```text
Authentication

Authorisation

Error handling
```

You do not need to implement these features yet.

---

## 📦 `models/`

Models will eventually describe the data stored by ParkSmart.

We are **not creating the models in Step 01**.

MongoDB and Mongoose will be introduced in Step 02.

Leave this folder empty for now.

---

## 🧰 `utils/`

Contains reusable utility/helper functionality that does not naturally belong to another layer.

Do not place random code here simply because you are unsure where it belongs.

---

## 🌐 `app.js`

This file will configure the Express application.

It will eventually be responsible for tasks such as:

```text
Create Express application

Configure middleware

Configure JSON processing

Register routes
```

---

## ▶️ `server.js`

This file is responsible for starting the application.

Separating application configuration from server startup also makes the project easier to test and maintain later.

---

# 🌐 17. Create the Express Application

You are now ready to create the first version of the ParkSmart API.

Open:

```text
src/app.js
```

Your task is to configure a basic Express application.

You need to research and implement the following:

1. Import the Express package.
2. Create an Express application.
3. Configure the application to accept JSON request bodies.
4. Create a basic test endpoint.
5. Export the Express application so that `server.js` can use it.

Your test endpoint should use:

```text
GET
```

and should be available at:

```text
/api/test
```

When the endpoint is requested, return a simple JSON response indicating that the ParkSmart API is running.

For example, the response could conceptually contain:

```text
message:
ParkSmart API is running
```

> ⚠️ You are expected to implement this yourself. Use the resources below to determine how Express applications, routes and responses work.

### 📚 Resources

**Express — Hello World:**  
https://expressjs.com/en/starter/hello-world.html

**Express — Basic Routing:**  
https://expressjs.com/en/starter/basic-routing.html

**Express — Routing Guide:**  
https://expressjs.com/en/guide/routing.html

Pay particular attention to:

```text
express()

express.json()

app.use()

app.get()

request

response
```

---

# 🧠 18. Understand the Test Endpoint

Your test endpoint has an important purpose.

At this stage, ParkSmart has:

```text
No database

No authentication

No parking data

No reservations
```

If something goes wrong later, we need a simple way of determining whether the API itself is still running.

The test endpoint gives us that.

A request to:

```text
GET /api/test
```

should result in:

```text
Client
-> Express
-> Test Route
-> JSON Response
```

If this works, you have confirmed that Express can receive and respond to an HTTP request.

---

# ▶️ 19. Configure `server.js`

Open:

```text
src/server.js
```

This file is responsible for starting the API.

Your task is to:

1. Import the Express application from `app.js`.
2. Define the port the API should listen on.
3. Start the server.
4. Display a useful message in the terminal when the server starts.

For development, ParkSmart can initially use:

```text
3000
```

as its port.

When successful, your terminal should display a message similar to:

```text
ParkSmart API is running on port 3000
```

### 📚 Resources

**Express — Hello World:**  
https://expressjs.com/en/starter/hello-world.html

Review how Express uses:

```text
listen()
```

to start accepting requests.

---

# ⚙️ 20. Add npm Scripts

At the moment, you could manually tell Node which JavaScript file to execute.

Instead, configure scripts inside:

```text
package.json
```

Add appropriate scripts so that the project can support:

```text
npm start
```

and:

```text
npm run dev
```

The normal start command should run the application using Node.

The development command should run the application using Nodemon.

Your scripts should point to:

```text
src/server.js
```

### Why?

This gives developers a consistent way to start the project.

Instead of remembering:

```text
Which file do I run?

Which tool should I use?

Where is the server?
```

the developer can simply use:

```powershell
npm run dev
```

during development.

---

# ▶️ 21. Run the ParkSmart API

Open the VS Code terminal.

Make sure you are inside:

```text
ParkSmart/api/
```

Run:

```powershell
npm run dev
```

If everything has been configured correctly, the server should start.

You should see your server-start message in the terminal.

Keep this terminal open.

Your API is now waiting for requests.

---

# 🌍 22. Understand `localhost`

During development, your API is running on your own computer.

The address:

```text
localhost
```

refers to the machine on which the application is currently running.

If the API uses port `3000`, its base address will be:

```text
http://localhost:3000
```

Your test endpoint will therefore be:

```text
http://localhost:3000/api/test
```

We will deal with Android emulator networking and `10.0.2.2` in a later step.

For now, test the API directly from your computer.

---

# 🧪 23. Test the Endpoint in Your Browser

Because your test endpoint uses `GET`, you can test it using a browser.

With the API running, visit:

```text
http://localhost:3000/api/test
```

You should receive the JSON response you created.

If this works:

```text
Browser
-> localhost:3000
-> Express
-> /api/test
-> JSON response
```

Your first ParkSmart endpoint is working.

---

# 🧪 24. Test the Endpoint Using an API Client

Although a browser works for simple `GET` requests, it is not enough for proper API development.

You should become comfortable using an API testing tool.

Choose one of the following.

## Postman

https://www.postman.com/downloads/

## Bruno

https://www.usebruno.com/downloads

## Insomnia

https://insomnia.rest/download

## Thunder Client

https://www.thunderclient.com/

Thunder Client is available as a VS Code extension.

---

## Create Your First API Request

Create a new request.

Set the method to:

```text
GET
```

Use:

```text
http://localhost:3000/api/test
```

Send the request.

Inspect:

- The response body.
- The HTTP status.
- The response headers.
- The response time.

You should receive a successful response from ParkSmart.

---

# 📊 25. Understand HTTP Status Codes

Every HTTP response contains a status code.

These codes help the client understand what happened.

Some common status codes include:

| Status | Meaning |
|---|---|
| `200` | Request succeeded |
| `201` | New resource successfully created |
| `400` | Client sent an invalid request |
| `401` | Authentication is required or invalid |
| `403` | User is authenticated but not allowed to perform the action |
| `404` | Requested resource could not be found |
| `409` | Request conflicts with existing data/state |
| `500` | Unexpected server error |

You will use these extensively later.

For now, your test endpoint should return a successful response.

### 📚 Resource

**MDN — HTTP Response Status Codes:**  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

Do not memorise every HTTP status code.

Learn how to choose an appropriate code based on what happened.

---

# 🔐 26. Introduce Environment Variables

Applications frequently need configuration that may change between environments.

Examples include:

```text
Port numbers

Database connection strings

Authentication configuration

Private credentials
```

These values should not all be hardcoded throughout your application.

ParkSmart will use environment variables.

Node.js provides access to environment variables through:

```text
process.env
```

### 📚 Resource

**Node.js — Environment Variables:**  
https://nodejs.org/api/environment_variables.html

---

# 📦 27. Install `dotenv`

Install the `dotenv` package:

```powershell
npm install dotenv
```

`dotenv` allows the application to load values from a local `.env` file into the application's environment.

### 📚 Resource

**dotenv — npm:**  
https://www.npmjs.com/package/dotenv

---

# 📄 28. Create `.env`

Inside the `api` folder, create:

```text
.env
```

For now, add:

```env
PORT=3000
```

Do not add quotation marks.

Use:

```env
PORT=3000
```

not:

```env
PORT="3000"
```

Your application should be updated so that the server obtains the port from the environment rather than relying only on a hardcoded value.

You should also provide an appropriate fallback value if the environment variable is unavailable.

Use the Node.js and dotenv documentation to determine how this should be implemented.

---

# 📄 29. Create `.env.example`

Create another file:

```text
.env.example
```

Add:

```env
PORT=
```

The difference is important.

### `.env`

Contains the values used on your machine.

For example:

```env
PORT=3000
```

This file should **not** be committed.

### `.env.example`

Documents which environment variables another developer needs.

For example:

```env
PORT=
```

This file **can** be committed.

Later, `.env` may contain sensitive configuration.

Never place real credentials inside `.env.example`.

---

# 🚫 30. Configure `.gitignore`

Git should not track every file generated by your development tools.

Create or update the repository's:

```text
.gitignore
```

At minimum, make sure you ignore unnecessary Node.js and Android-generated files.

For example:

```gitignore
# Node dependencies
node_modules/

# Environment variables
.env

# Android local configuration
local.properties

# Android / Gradle generated files
.gradle/
**/build/

# IDE-specific local files
.idea/

# Operating system files
.DS_Store
Thumbs.db
```

> ⚠️ Be careful when ignoring IDE files in collaborative Android projects. Some project configuration may be intentionally shared. Review what your team actually needs rather than blindly ignoring every file.

GitHub's documentation explains that `.gitignore` tells Git which files and directories should not be tracked. :contentReference[oaicite:0]{index=0}

### 📚 Resources

**GitHub — Ignoring Files:**  
https://docs.github.com/en/get-started/git-basics/ignoring-files

**GitHub — `.gitignore` Templates:**  
https://github.com/github/gitignore

---

# 🔍 31. Check That `.gitignore` Works

Before committing your project, run:

```powershell
git status
```

Inspect the files Git wants to track.

You should **not** see:

```text
node_modules/

.env
```

waiting to be committed.

You should see files such as:

```text
package.json

package-lock.json

.env.example

src/
```

If `.env` has already been committed before being added to `.gitignore`, simply adding it to `.gitignore` will not automatically stop Git from tracking the existing file. GitHub documents that an already tracked file must first be untracked. :contentReference[oaicite:1]{index=1}

---

# 🧪 32. Test the API Again

Restart the API:

```powershell
npm run dev
```

Confirm that the application reads the port from your environment configuration.

Test:

```text
GET http://localhost:3000/api/test
```

Confirm that the endpoint still works.

You have now tested:

```text
Environment configuration
-> Server startup
-> Express
-> Route
-> HTTP response
```

---

# 🧯 33. Test an Invalid Endpoint

Do not only test successful requests.

Try requesting something that does not exist.

For example:

```text
GET http://localhost:3000/api/does-not-exist
```

Observe what Express returns.

Ask yourself:

- What HTTP status was returned?
- What response body was returned?
- Is this response suitable for a REST API?
- Would an Android application be able to handle this response cleanly?

You do not need to build a complete centralised error-handling system yet.

For now, understand that successful and unsuccessful requests both need to be considered when designing an API.

---

# 📁 34. Review Your Project Structure

By the end of Step 01, your project should resemble:

```text
ParkSmart/
|
|-- android/
|   |
|   |-- ParkSmartAndroid/
|
|-- api/
|   |
|   |-- src/
|   |   |
|   |   |-- config/
|   |   |
|   |   |-- controllers/
|   |   |
|   |   |-- middleware/
|   |   |
|   |   |-- models/
|   |   |
|   |   |-- routes/
|   |   |
|   |   |-- services/
|   |   |
|   |   |-- utils/
|   |   |
|   |   |-- app.js
|   |   |
|   |   |-- server.js
|   |
|   |-- .env
|   |
|   |-- .env.example
|   |
|   |-- package.json
|   |
|   |-- package-lock.json
|
|-- README.md
|
|-- .gitignore
```

Remember:

```text
node_modules/
```

exists locally but should not be committed to the repository.

---

# 🧠 35. Make Sure You Understand the Flow

At the end of this step, your backend flow is still very simple:

```text
API Client
-> HTTP Request
-> Express
-> Route
-> HTTP Response
```

For example:

```text
GET /api/test
-> Express receives request
-> Matching route is found
-> Route handles request
-> JSON response is returned
```

Later, ParkSmart will expand this into something closer to:

```text
Android
-> Route
-> Authentication
-> Controller
-> Service
-> Database
-> Response
```

Do not rush ahead.

Each layer will be introduced as it becomes necessary.

---

# 🐛 Common Problems

## `node` is not recognised

Run:

```powershell
node --version
```

If Windows cannot find the command:

1. Confirm that Node.js was installed.
2. Close and reopen VS Code.
3. Open a new terminal.
4. Try the command again.
5. If necessary, check whether Node.js was added to your system PATH.

---

## `npm` is not recognised

Run:

```powershell
npm --version
```

npm is normally installed together with Node.js.

If Node works but npm does not, restart your terminal and verify the Node.js installation.

---

## `Cannot find module 'express'`

Make sure your terminal is inside:

```text
ParkSmart/api/
```

Then run:

```powershell
npm install
```

Check that:

```text
node_modules/
```

has been created.

---

## `nodemon` is not recognised

If Nodemon was installed locally as a development dependency, use:

```powershell
npm run dev
```

using the script configured inside `package.json`.

You do not need a global Nodemon installation.

---

## Port 3000 is already in use

Another application may already be using the port.

Stop the other server or change:

```env
PORT=3000
```

to another available port, for example:

```env
PORT=3001
```

Remember that your request URL must then use the same port.

---

## The browser says the page cannot be reached

Check:

1. Is the API currently running?
2. Is the terminal showing an error?
3. Are you using the correct port?
4. Is the endpoint correct?
5. Did you accidentally stop the server?

---

## `/api/test` returns 404

Check:

- The HTTP method.
- The route path.
- Whether the route was registered.
- Whether the correct application file is being executed.
- Whether you saved your changes.

Remember:

```text
/api/test
```

and:

```text
/api/tests
```

are different paths.

---

## Changes do not appear

If using Nodemon:

1. Save the file.
2. Check the terminal.
3. Confirm that Nodemon restarted the server.
4. Send the request again.

---

# 📚 Helpful Resources

Use these resources when you get stuck rather than copying an entire solution from somewhere else.

## 🟢 Node.js

**Node.js — Download**  
https://nodejs.org/en/download

Install Node.js from here.

**Node.js — Introduction**  
https://nodejs.org/en/learn/getting-started/introduction-to-nodejs

Useful for understanding what Node.js is and why JavaScript can be used on a server.

**Node.js — Environment Variables**  
https://nodejs.org/api/environment_variables.html

Useful when configuring `.env` and `process.env`.

---

## 📦 npm

**npm — `npm init`**  
https://docs.npmjs.com/cli/commands/npm-init

Explains how a Node project is initialised.

**npm — `package.json`**  
https://docs.npmjs.com/cli/configuring-npm/package-json

Useful for understanding dependencies, scripts and project metadata.

---

## 🌐 Express

**Express — Installing**  
https://expressjs.com/en/starter/installing.html

Use this when setting up Express. The current Express documentation requires Node.js 18 or later for Express 5. :contentReference[oaicite:2]{index=2}

**Express — Hello World**  
https://expressjs.com/en/starter/hello-world.html

Useful for understanding how an Express application starts.

**Express — Basic Routing**  
https://expressjs.com/en/starter/basic-routing.html

This is particularly important for Step 01.

Express defines routes using the general structure:

```text
app.METHOD(PATH, HANDLER)
```

where the method represents the HTTP method and the path represents the endpoint. :contentReference[oaicite:3]{index=3}

**Express — Routing Guide**  
https://expressjs.com/en/guide/routing.html

Use this to understand routes in more detail.

---

## 🌍 HTTP

**MDN — HTTP Methods**  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods

Useful for understanding:

```text
GET
POST
PUT
PATCH
DELETE
```

**MDN — HTTP Status Codes**  
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

Useful for understanding responses such as:

```text
200
201
400
401
403
404
409
500
```

---

## 🧪 API Testing

**Postman**  
https://www.postman.com/downloads/

**Bruno**  
https://www.usebruno.com/downloads

**Insomnia**  
https://insomnia.rest/download

**Thunder Client**  
https://www.thunderclient.com/

You only need one of these tools.

---

## 🔐 Git and `.gitignore`

**GitHub — Ignoring Files**  
https://docs.github.com/en/get-started/git-basics/ignoring-files

Explains how `.gitignore` works. :contentReference[oaicite:4]{index=4}

**GitHub — Official `.gitignore` Templates**  
https://github.com/github/gitignore

Useful when deciding which generated files should not be committed. :contentReference[oaicite:5]{index=5}

---

# 🔒 Security Checks

Before completing Step 01, check that:

- `.env` is not being committed.
- `node_modules/` is not being committed.
- No passwords or credentials have been placed inside JavaScript files.
- No private values have been placed inside the README.
- Environment-specific values are kept outside the application logic.
- `.env.example` contains variable names but no private values.

You do not have many secrets yet.

That will change significantly in later steps.

Start using good habits now.

---

# 🧪 Step 01 Testing Checklist

Test the following before moving on.

## 📱 Android

- [ ] The ParkSmart Android project exists.
- [ ] Gradle Sync succeeds.
- [ ] The Android application builds.
- [ ] The Android application launches.

## 🟢 Node.js

- [ ] `node --version` works.
- [ ] `npm --version` works.
- [ ] The Node project has been initialised.
- [ ] `package.json` exists.
- [ ] `package-lock.json` exists.

## 🌐 Express

- [ ] Express is installed.
- [ ] Nodemon is installed as a development dependency.
- [ ] `app.js` exists.
- [ ] `server.js` exists.
- [ ] `npm run dev` starts the API.
- [ ] The terminal displays a useful startup message.

## 🧪 API

- [ ] `GET /api/test` works.
- [ ] The endpoint returns JSON.
- [ ] The endpoint returns a successful HTTP status.
- [ ] You can test the endpoint using an API client.
- [ ] You tested at least one invalid endpoint.

## ⚙️ Configuration

- [ ] `dotenv` is installed.
- [ ] `.env` exists.
- [ ] `PORT` is stored in `.env`.
- [ ] `.env.example` exists.
- [ ] The API can read the port from the environment.

## 🔐 Git

- [ ] `.gitignore` exists.
- [ ] `.env` is ignored.
- [ ] `node_modules/` is ignored.
- [ ] Android-generated files are appropriately ignored.
- [ ] `git status` does not show private or unnecessary generated files waiting to be committed.

---

# ✅ Step 01 Complete

At this point, you should have two working applications:

```text
ParkSmart Android Application

and

ParkSmart REST API
```

They are **not communicating with each other yet**.

Your API also does **not have persistent data yet**.

That is intentional.

You have established the foundation that the rest of ParkSmart will use.

Your current backend flow is:

```text
API Client
-> Express
-> Test Endpoint
-> JSON Response
```

In the next step, we need to solve an important problem:

> **Where will ParkSmart store its users, parking areas, parking bays and reservations?**

That introduces the next part of the system:

```text
Express API
-> MongoDB
```

# ➡️ Continue to Step 02 — MongoDB and Backend Data
