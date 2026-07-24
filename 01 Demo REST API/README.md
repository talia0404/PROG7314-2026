## 🌐 Building Your First REST API

### 🎯 Objective

In this practical, you will create your first ASP.NET Core Web API for this module.

The purpose of this activity is to revise the fundamental concepts of RESTful web services before we begin integrating Android applications with APIs later in the semester.

Your API will manage a small collection of books.

---

# 📖 Scenario

A local library would like a simple REST API that allows applications to retrieve and manage information about books.

📌 The API **does not** need a database yet.

For this activity, all data can be stored temporarily in memory.

---

# 📝 Requirements

## Step 1 – Create the Project

Create a new **ASP.NET Core Web API** project.

Your project should:

*  Use Controllers
*  Include Swagger/OpenAPI support
*  Enable HTTPS

Once created, remove the default **Weather Forecast** example so that you are left with a clean project.

---

## 📚 Step 2 – Create the Model

Create a model that represents a book.

Each book should contain the following information:

*  Book ID
*  Title
*  Author
*  Year Published

> 💡 Think carefully about the most appropriate data type for each property.

---

## 🎮 Step 3 – Create the Controller

Create a controller responsible for handling all requests related to books.

Your controller should expose the following endpoints:

| HTTP Method | Endpoint          | Description            |
| ----------- | ----------------- | ---------------------- |
| GET         | `/api/books`      | Retrieve all books     |
| GET         | `/api/books/{id}` | Retrieve a single book |
| POST        | `/api/books`      | Add a new book         |
| DELETE      | `/api/books/{id}` | Delete a book          |

> ⚠️ Do **not** use a database for this activity.

Instead, maintain your data using an **in-memory collection**.

---

## ➕ Step 4 – Add Sample Data

Before testing your API, populate your collection with at least **three books**.

📚 You may choose any books you like.

---

## 🧪 Step 5 – Test the API

Use **Swagger** to test every endpoint.

Ensure that:

*  All books can be retrieved.
*  A single book can be retrieved using its ID.
*  New books can be added.
*  Books can be deleted successfully.

---

## 🚨 Step 6 – Handle Invalid Requests

Your API should respond appropriately when a user requests a book that does not exist.

Think about:

* 🤔 Which HTTP status code should be returned?
* 💬 Should the user receive an error message?

---



# 💡 Remember

This practical is designed to help you **understand how a REST API works** before we begin connecting Android applications to APIs later in the semester.

Don't worry about creating the "perfect" API just yet—we'll continue improving and expanding these concepts throughout the module. 🚀
