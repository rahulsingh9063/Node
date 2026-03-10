# Express Middleware & Form Handling Complete Guide

### Middleware, req.body/params/query, Forms, File Operations

---

## 📚 What You'll Learn

This guide covers **Express middleware and form handling**:

✅ What is Middleware?  
✅ Middleware execution flow  
✅ Types of Middleware (5 types)  
✅ Form handling in Express  
✅ req.body, req.params, req.query explained  
✅ express.urlencoded() middleware  
✅ File operations with fs module  
✅ Reading and writing JSON files  
✅ Complete registration form example  
✅ import.meta.dirname in ES modules

**Best for:** Express development, form handling, middleware understanding, file operations

---

## Table of Contents

1. [What is Middleware?](#1-what-is-middleware)
2. [Middleware Flow](#2-middleware-flow)
3. [Creating Middleware](#3-creating-middleware)
4. [next() Function](#4-next-function)
5. [Types of Middleware](#5-types-of-middleware)
6. [Built-in Middleware](#6-built-in-middleware)
7. [req.body, req.params, req.query](#7-reqbody-reqparams-reqquery)
8. [Form Handling](#8-form-handling)
9. [File Operations with fs](#9-file-operations-with-fs)
10. [Complete Registration Example](#10-complete-registration-example)
11. [import.meta.dirname](#11-importmetadirname)
12. [Common Patterns](#12-common-patterns)
13. [Summary](#13-summary)
14. [Revision Checklist](#14-revision-checklist)

---

## 1. What is Middleware?

### Definition

**Middleware is a function that comes BETWEEN the request and response. It has access to the request (req) object, response (res) object, and the next() function.**

---

### Simple Explanation

```
Request → Middleware → Response

Think of it as a "filter" or "checkpoint" that every request passes through.
```

---

### Your Definition

```javascript
//! middleware -> it is a function which comes in between req and res
//! It has access to req and res object
//! It has next() which calls the next middleware
//! If no next middleware, request goes to controller (route handler)
```

---

### Visual: Middleware Position

```
Client Request
      ↓
┌─────────────────────┐
│   Middleware 1      │  ← Can modify req/res
└─────────────────────┘
      ↓ next()
┌─────────────────────┐
│   Middleware 2      │  ← Can modify req/res
└─────────────────────┘
      ↓ next()
┌─────────────────────┐
│   Route Handler     │  ← Final destination
│   (Controller)      │
└─────────────────────┘
      ↓
Response to Client
```

---

### Middleware Function Structure

```javascript
function myMiddleware(req, res, next) {
  // req  = request object (from client)
  // res  = response object (to send back)
  // next = function to call next middleware

  // Do something
  console.log("Middleware executed");

  // Call next() to continue
  next();
}
```

---

### Three Parameters

```javascript
(req, res, next) => {
  // req    → Request object (data from client)
  // res    → Response object (send data to client)
  // next   → Function to call next middleware
};
```

---

## 2. Middleware Flow

### Sequential Execution

**Middleware executes in the ORDER it's defined.**

```javascript
app.use(middleware1); // Runs first
app.use(middleware2); // Runs second
app.use(middleware3); // Runs third

app.get("/", handler); // Runs last
```

---

### Your Example — Middleware Flow

```javascript
import express from "express";
let app = express();

// Middleware 1
app.use((req, res, next) => {
  console.log("m1");
  req.myProperty = "abc"; // Add property to req
  next(); // Call next middleware
});

// Middleware 2
app.use((req, res, next) => {
  console.log("m2");
  next(); // Call next middleware
});

// Middleware 3
app.use((req, res, next) => {
  console.log("m3");
  next(); // Call route handler
});

// Route handler
app.get("/", (req, res) => {
  console.log(req.myProperty); // "abc" (set by m1)
  res.send("home");
});

app.listen(9000);
```

---

### Execution Trace

```
User visits: http://localhost:9000/

Console output:
────────────────────────────────────────
m1        ← Middleware 1 executes
m2        ← Middleware 2 executes
m3        ← Middleware 3 executes
abc       ← Route handler executes (console.log(req.myProperty))

Browser receives:
home
```

---

### Visual: Execution Flow

```
Request to "/"
      ↓
app.use(m1)
  - console.log("m1")
  - req.myProperty = "abc"
  - next() ✅
      ↓
app.use(m2)
  - console.log("m2")
  - next() ✅
      ↓
app.use(m3)
  - console.log("m3")
  - next() ✅
      ↓
app.get("/", handler)
  - console.log(req.myProperty) → "abc"
  - res.send("home")
      ↓
Response: "home"
```

---

### What Happens Without next()?

```javascript
app.use((req, res, next) => {
  console.log("m1");
  // Missing next()! ❌
});

app.get("/", (req, res) => {
  res.send("home"); // NEVER REACHED
});

// Request hangs forever (no response sent)
```

**Critical Rule:** If you don't call `next()` or send a response, the request hangs!

---

## 3. Creating Middleware

### Method 1: app.use() — Global Middleware

```javascript
// Applies to ALL routes
app.use((req, res, next) => {
  console.log("This runs for every request");
  next();
});

app.get("/"); // Middleware runs
app.get("/about"); // Middleware runs
app.post("/users"); // Middleware runs
```

---

### Method 2: Route-Specific Middleware

```javascript
// Only for specific route
app.get("/admin", middleware, (req, res) => {
  res.send("Admin page");
});

// middleware runs ONLY for /admin route
// Other routes don't use this middleware
```

---

### Method 3: Multiple Middleware

```javascript
// Chain multiple middleware
app.get("/", middleware1, middleware2, middleware3, (req, res) => {
  res.send("Final handler");
});

// Execution: m1 → m2 → m3 → handler
```

---

### Example: Logger Middleware

```javascript
// Custom logger middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  console.log(`Time: ${new Date().toISOString()}`);
  next();
});

// Logs every request:
// GET /
// Time: 2024-03-10T10:30:00.000Z
```

---

### Example: Adding Properties to req

```javascript
// Add custom property
app.use((req, res, next) => {
  req.customData = { userId: 123, role: "admin" };
  next();
});

// Use in route handler
app.get("/dashboard", (req, res) => {
  console.log(req.customData);
  // { userId: 123, role: "admin" }
  res.send("Dashboard");
});
```

---

## 4. next() Function

### What is next()?

**next() is a function that passes control to the next middleware or route handler.**

---

### next() Behavior

```javascript
// Case 1: Call next() → Continue to next middleware
app.use((req, res, next) => {
  console.log("Middleware 1");
  next(); // ✅ Continue
});

// Case 2: Don't call next() → Stop (must send response)
app.use((req, res, next) => {
  console.log("Middleware 2");
  res.send("Stopping here"); // ✅ Send response
  // No next() → Chain stops
});

// Case 3: Neither next() nor response → Request hangs ❌
app.use((req, res, next) => {
  console.log("Middleware 3");
  // No next(), no response → HANGS
});
```

---

### next() vs res.send()

```javascript
// Option 1: Call next() → Continue to next middleware
app.use((req, res, next) => {
  console.log("Logging...");
  next(); // Continue
});

// Option 2: Send response → Stop here
app.use((req, res, next) => {
  console.log("Authentication failed");
  res.status(401).send("Unauthorized");
  // No next() → Chain stops
});
```

---

### next() with Error

```javascript
// Pass error to error handler
app.use((req, res, next) => {
  const error = new Error("Something went wrong");
  next(error); // Skip to error middleware
});

// Error middleware (4 parameters)
app.use((err, req, res, next) => {
  console.error(err.message);
  res.status(500).send("Server Error");
});
```

---

## 5. Types of Middleware

### The 5 Types

```
1. App-level Middleware       → app.use()
2. Router-level Middleware     → router.use()
3. Built-in Middleware         → express.json(), express.urlencoded()
4. Third-party Middleware      → cors, morgan, helmet
5. Error Middleware            → 4 parameters (err, req, res, next)
```

---

### 1. App-Level Middleware

**Bound to the app object using `app.use()`.**

```javascript
import express from "express";
let app = express();

// App-level middleware
app.use((req, res, next) => {
  console.log("App-level middleware");
  next();
});

app.use("/admin", (req, res, next) => {
  console.log("Only for /admin routes");
  next();
});

app.get("/", (req, res) => {
  res.send("Home");
});
```

---

### 2. Router-Level Middleware

**Bound to Router instance using `router.use()`.**

```javascript
import { Router } from "express";
let router = Router();

// Router-level middleware
router.use((req, res, next) => {
  console.log("Router middleware");
  next();
});

// Only affects routes in this router
router.get("/users", (req, res) => {
  res.send("Users");
});

// Mount router
app.use("/api", router);
```

---

### 3. Built-in Middleware

**Provided by Express (no installation needed).**

```javascript
// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies (forms)
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static("public"));
```

---

### 4. Third-Party Middleware

**Installed via npm.**

```javascript
import cors from "cors";
import morgan from "morgan";
import helmet from "helmet";

// CORS (Cross-Origin Resource Sharing)
app.use(cors());

// Morgan (HTTP request logger)
app.use(morgan("dev"));

// Helmet (Security headers)
app.use(helmet());
```

---

### 5. Error Middleware

**Has 4 parameters: `(err, req, res, next)`.**

```javascript
// Error middleware (MUST have 4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);

  res.status(500).json({
    success: false,
    message: err.message,
    error: err,
  });
});

// Usage: Pass error to next()
app.get("/error", (req, res, next) => {
  const error = new Error("Something broke");
  next(error); // Triggers error middleware
});
```

---

## 6. Built-in Middleware

### express.json()

**Parses incoming JSON payloads.**

```javascript
app.use(express.json());

// Now you can access JSON data
app.post("/api/users", (req, res) => {
  console.log(req.body);
  // { "name": "Alice", "age": 30 }

  res.json({ received: req.body });
});

// Request body (JSON):
// { "name": "Alice", "age": 30 }
```

---

### express.urlencoded()

**Parses URL-encoded form data (from HTML forms).**

```javascript
// Your example:
app.use(express.urlencoded({ extended: true }));

// Now you can access form data
app.post("/register", (req, res) => {
  console.log(req.body);
  // { userEmail: "abc@example.com", userPassword: "123" }

  res.send("User registered");
});

// HTML form submits:
// userEmail=abc@example.com&userPassword=123
```

---

### extended: true vs false

```javascript
// extended: true (recommended)
app.use(express.urlencoded({ extended: true }));
// Supports nested objects and arrays
// Uses qs library

// extended: false
app.use(express.urlencoded({ extended: false }));
// Simple key-value pairs only
// Uses querystring library
```

---

### express.static()

**Serves static files (HTML, CSS, images, JS).**

```javascript
// Serve files from "public" folder
app.use(express.static("public"));

// File structure:
// public/
//   ├── index.html
//   ├── style.css
//   └── script.js

// Now accessible at:
// http://localhost:3000/index.html
// http://localhost:3000/style.css
// http://localhost:3000/script.js
```

---

## 7. req.body, req.params, req.query

### The Three Request Properties

```javascript
req.body; // → Form data, JSON data (POST/PUT/PATCH)
req.params; // → URL parameters (/users/:id)
req.query; // → Query string (?key=value&key2=value2)
```

---

### req.body — Form/JSON Data

**Contains data sent in request body (POST, PUT, PATCH).**

```javascript
// Requires middleware
app.use(express.json()); // For JSON
app.use(express.urlencoded({ extended: true })); // For forms

// HTML form
<form action="/register" method="post">
  <input name="userEmail" value="abc@example.com" />
  <input name="userPassword" value="123" />
  <button>Submit</button>
</form>;

// Express route
app.post("/register", (req, res) => {
  console.log(req.body);
  // Output: { userEmail: "abc@example.com", userPassword: "123" }

  let { userEmail, userPassword } = req.body;
  res.send(`Email: ${userEmail}`);
});
```

---

### req.params — URL Parameters

**Contains dynamic values from URL path.**

```javascript
// Route with parameter
app.get("/users/:id", (req, res) => {
  console.log(req.params);
  // { id: "123" }

  let userId = req.params.id;
  res.send(`User ID: ${userId}`);
});

// Request: GET /users/123
// req.params = { id: "123" }

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req, res) => {
  console.log(req.params);
  // { userId: "123", postId: "456" }

  let { userId, postId } = req.params;
  res.send(`User ${userId}, Post ${postId}`);
});

// Request: GET /users/123/posts/456
// req.params = { userId: "123", postId: "456" }
```

---

### req.query — Query String

**Contains data from query string (after `?`).**

```javascript
app.get("/search", (req, res) => {
  console.log(req.query);
  // { q: "javascript", limit: "10", page: "2" }

  let { q, limit, page } = req.query;
  res.send(`Search: ${q}, Limit: ${limit}, Page: ${page}`);
});

// Request: GET /search?q=javascript&limit=10&page=2
// req.query = { q: "javascript", limit: "10", page: "2" }
```

---

### Your Example — Query String

```javascript
// Your comment:
//! req.query -> key=value&key2=value2 -> anything after ? is query

// http://localhost:9000/varun?age=30&area=noida

app.get("/varun", (req, res) => {
  console.log(req.query);
  // { age: "30", area: "noida" }

  res.send(`Age: ${req.query.age}, Area: ${req.query.area}`);
});
```

---

### Real Amazon URL Example

```javascript
// Your Amazon URL (simplified):
// https://www.amazon.in/product?
//   _encoding=UTF8&
//   content-id=amzn1.sym.58c90a12&
//   qid=1773116976&
//   s=kitchen&
//   sr=1-1

app.get("/product", (req, res) => {
  console.log(req.query);
  // {
  //   _encoding: "UTF8",
  //   "content-id": "amzn1.sym.58c90a12",
  //   qid: "1773116976",
  //   s: "kitchen",
  //   sr: "1-1"
  // }
});
```

---

### Comparison Table

| Property     | Location     | Example                                           | Use For                      |
| ------------ | ------------ | ------------------------------------------------- | ---------------------------- |
| `req.body`   | Request body | `{ name: "Alice" }`                               | POST/PUT data, forms, JSON   |
| `req.params` | URL path     | `/users/:id` → `{ id: "123" }`                    | Dynamic routes, resource IDs |
| `req.query`  | Query string | `?page=2&limit=10` → `{ page: "2", limit: "10" }` | Filters, pagination, search  |

---

## 8. Form Handling

### HTML Form Basics

**Your form example:**

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>User Registration</title>
  </head>
  <body>
    <h1>Fill this form</h1>

    <!-- action: where to send data -->
    <!-- method: HTTP method (post) -->
    <form action="/register" method="post">
      <!-- name attribute: becomes property in req.body -->
      email: <input type="text" name="userEmail" />
      <br />
      password: <input type="text" name="userPassword" />
      <br />
      <button>Submit</button>
    </form>
  </body>
</html>
```

---

### Form Attributes

```html
<form action="/register" method="post">
  <!-- action  → Where to send data (endpoint) -->
  <!-- method  → HTTP method (GET or POST) -->

  <!-- name attribute → Property name in req.body -->
  <input type="text" name="userEmail" />
  <input type="text" name="userPassword" />

  <button>Submit</button>
</form>
```

---

### Form Submission Flow

```
User fills form:
─────────────────────────────────────────
email: abc@example.com
password: 123

User clicks Submit
      ↓
Browser sends POST request:
─────────────────────────────────────────
POST /register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

userEmail=abc@example.com&userPassword=123
      ↓
Express receives request
      ↓
Middleware parses body:
express.urlencoded({ extended: true })
      ↓
req.body = {
    userEmail: "abc@example.com",
    userPassword: "123"
}
      ↓
Route handler executes:
app.post("/register", (req, res) => {
    console.log(req.body);
    // Save to database
    res.send("Registered!");
});
```

---

### Serving Form HTML

**Your example:**

```javascript
app.get("/form", (req, res) => {
  let readStream = fs.createReadStream(
    path.join(import.meta.dirname, "pages", "form.html"),
    "utf-8",
  );

  // Pipe stream to response
  readStream.pipe(res);
});
```

---

### Alternative: res.sendFile()

```javascript
app.get("/form", (req, res) => {
  res.sendFile(path.join(import.meta.dirname, "pages", "form.html"));
});

// Simpler than streaming for static files
```

---

## 9. File Operations with fs

### Reading Files

```javascript
import fs from "node:fs";

// Synchronous (blocks execution)
let data = fs.readFileSync("./file.txt", "utf-8");
console.log(data);

// Asynchronous (non-blocking)
fs.readFile("./file.txt", "utf-8", (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

---

### Writing Files

```javascript
// Synchronous
fs.writeFileSync("./file.txt", "Hello World");

// Asynchronous
fs.writeFile("./file.txt", "Hello World", (err) => {
  if (err) throw err;
  console.log("File written");
});
```

---

### Appending to Files

```javascript
// Synchronous
fs.appendFileSync("./file.txt", "\nNew line");

// Asynchronous
fs.appendFile("./file.txt", "\nNew line", (err) => {
  if (err) throw err;
  console.log("Data appended");
});
```

---

### Reading JSON Files

```javascript
// Read JSON file
let jsonData = fs.readFileSync("./data.json", "utf-8");

// Parse JSON string to object
let data = JSON.parse(jsonData);

console.log(data);
// [{ "name": "Alice" }, { "name": "Bob" }]
```

---

### Writing JSON Files

```javascript
// JavaScript object
let data = [
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
];

// Convert to JSON string
let jsonString = JSON.stringify(data, null, 2);
// null, 2 → Pretty print with 2-space indentation

// Write to file
fs.writeFileSync("./data.json", jsonString);
```

---

### Streaming Files

```javascript
// Create read stream
let readStream = fs.createReadStream("./large-file.txt", "utf-8");

// Pipe to response
readStream.pipe(res);

// Memory efficient for large files
```

---

## 10. Complete Registration Example

### File Structure

```
project/
├── db/
│   └── data.json
├── pages/
│   └── form.html
├── app.js
└── package.json
```

---

### data.json (Initial)

```json
[
  {
    "userEmail": "existing@example.com",
    "userPassword": "password123"
  }
]
```

---

### form.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>User Registration</title>
  </head>
  <body>
    <h1>Fill this form</h1>
    <form action="/register" method="post">
      Email: <input type="text" name="userEmail" required />
      <br />
      Password: <input type="password" name="userPassword" required />
      <br />
      <button>Submit</button>
    </form>
  </body>
</html>
```

---

### app.js (CORRECTED VERSION)

```javascript
//? 1) Import
import express from "express";
import fs from "node:fs";
import path from "node:path";

//? 2) Create app
let app = express();

//! CRITICAL: Parse form data
app.use(express.urlencoded({ extended: true }));

//? 4) Routes

//~ Home page
app.get("/", (req, res) => {
  res.send("<h1>This is home page</h1>");
});

//~ Serve form
app.get("/form", (req, res) => {
  res.sendFile(path.join(import.meta.dirname, "pages", "form.html"));
});

//~ Handle registration
app.post("/register", (req, res) => {
  // Get form data
  let { userEmail, userPassword } = req.body;

  // Read existing data
  let existingData = [];
  try {
    let fileContent = fs.readFileSync("./db/data.json", "utf-8");
    existingData = JSON.parse(fileContent);
  } catch (error) {
    // File doesn't exist or is empty
    existingData = [];
  }

  // Add new user
  existingData.push({ userEmail, userPassword });

  // Write back to file
  fs.writeFileSync("./db/data.json", JSON.stringify(existingData, null, 2));

  res.send("User registered successfully!");
});

//? 3) Start server
app.listen(9000, (err) => {
  if (err) console.log(err);
  console.log("Server running at port 9000");
});
```

---

### What Was Wrong in Original Code

```javascript
// ❌ WRONG:
app.post("/register", (req, res) => {
  let { userEmail, userPassword } = req.body;

  let readFile = fs.readFileSync("./db/data.json", "utf-8");
  // readFile is read but NOT used!

  fs.appendFileSync(
    "./db/data.json",
    JSON.stringify({ userEmail, userPassword }),
  );
  // This just appends raw JSON string, breaking JSON structure

  res.send("done");
});

// Result in data.json:
// [{"userEmail":"old","userPassword":"old"}]{"userEmail":"new","userPassword":"new"}
// ❌ INVALID JSON!
```

---

### Fixed Version Explanation

```javascript
// ✅ CORRECT:
app.post("/register", (req, res) => {
  let { userEmail, userPassword } = req.body;

  // Step 1: Read existing data
  let existingData = [];
  try {
    let fileContent = fs.readFileSync("./db/data.json", "utf-8");
    existingData = JSON.parse(fileContent); // Parse to array
  } catch (error) {
    existingData = []; // Start with empty array if file doesn't exist
  }

  // Step 2: Add new user to array
  existingData.push({ userEmail, userPassword });

  // Step 3: Write entire array back
  fs.writeFileSync(
    "./db/data.json",
    JSON.stringify(existingData, null, 2), // Pretty print
  );

  res.send("User registered successfully!");
});

// Result in data.json:
// [
//   {
//     "userEmail": "old@example.com",
//     "userPassword": "old123"
//   },
//   {
//     "userEmail": "new@example.com",
//     "userPassword": "new123"
//   }
// ]
// ✅ VALID JSON!
```

---

### Testing the Flow

```
Step 1: Start server
────────────────────────────────────────
$ node app.js
Server running at port 9000

Step 2: Visit form
────────────────────────────────────────
Browser: http://localhost:9000/form
Response: form.html displayed

Step 3: Fill and submit form
────────────────────────────────────────
Email: alice@example.com
Password: password123
Click Submit

Step 4: Request sent
────────────────────────────────────────
POST /register
Body: userEmail=alice@example.com&userPassword=password123

Step 5: Server processes
────────────────────────────────────────
1. Parse body with express.urlencoded()
2. Extract userEmail and userPassword
3. Read data.json
4. Parse JSON to array
5. Add new user to array
6. Write array back to data.json

Step 6: Response
────────────────────────────────────────
Browser: "User registered successfully!"

Step 7: Check data.json
────────────────────────────────────────
[
  {
    "userEmail": "alice@example.com",
    "userPassword": "password123"
  }
]
```

---

## 11. import.meta.dirname

### What is import.meta.dirname?

**In ES modules, `import.meta.dirname` gives the absolute path of the current directory.**

---

### Your Comment

```javascript
console.log(import.meta.dirname);
//& This will give the absolute path of the current folder
//& in which the file is present in ES module scope
//& (in commonJS use __dirname)
```

---

### ES Modules vs CommonJS

```javascript
// ES Modules (type: "module")
console.log(import.meta.dirname);
// Output: /Users/username/project

// CommonJS (default)
console.log(__dirname);
// Output: /Users/username/project
```

---

### Usage Example

```javascript
import path from "node:path";

// Get file path
let filePath = path.join(import.meta.dirname, "pages", "form.html");
// Result: /Users/username/project/pages/form.html

// Use in Express
app.get("/form", (req, res) => {
  res.sendFile(path.join(import.meta.dirname, "pages", "form.html"));
});
```

---

### Why Use It?

```javascript
// ❌ BAD: Relative path (breaks if working directory changes)
res.sendFile("./pages/form.html");

// ✅ GOOD: Absolute path (always works)
res.sendFile(path.join(import.meta.dirname, "pages", "form.html"));
```

---

## 12. Common Patterns

### Pattern 1: Logger Middleware

```javascript
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});

// Output:
// [2024-03-10T10:30:00.000Z] GET /
// [2024-03-10T10:30:05.000Z] POST /register
```

---

### Pattern 2: Authentication Middleware

```javascript
function isAuthenticated(req, res, next) {
  if (req.headers.authorization) {
    next(); // Authenticated, continue
  } else {
    res.status(401).send("Unauthorized");
  }
}

// Use on protected routes
app.get("/dashboard", isAuthenticated, (req, res) => {
  res.send("Dashboard");
});
```

---

### Pattern 3: Error Handler

```javascript
// Route
app.get("/error", (req, res, next) => {
  const error = new Error("Something broke");
  next(error); // Pass to error handler
});

// Error middleware (at the end)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    success: false,
    message: err.message,
  });
});
```

---

### Pattern 4: JSON Response Helper

```javascript
app.use((req, res, next) => {
  // Add custom response method
  res.success = (data) => {
    res.json({
      success: true,
      data: data,
    });
  };

  res.error = (message) => {
    res.status(400).json({
      success: false,
      message: message,
    });
  };

  next();
});

// Use in routes
app.get("/users", (req, res) => {
  res.success({ users: ["Alice", "Bob"] });
});

app.get("/invalid", (req, res) => {
  res.error("Invalid request");
});
```

---

## 13. Summary

### Middleware

```
Definition: Function between request and response
Structure: (req, res, next) => { }
Purpose: Process request, modify req/res, call next()
```

---

### Middleware Types

```
1. App-level:        app.use()
2. Router-level:     router.use()
3. Built-in:         express.json(), express.urlencoded()
4. Third-party:      cors, morgan, helmet
5. Error:            (err, req, res, next) => { }
```

---

### Request Properties

```
req.body      → Form/JSON data (requires middleware)
req.params    → URL parameters (/users/:id)
req.query     → Query string (?key=value)
```

---

### Built-in Middleware

```javascript
express.json(); // Parse JSON
express.urlencoded({ extended: true }); // Parse forms
express.static("public"); // Serve static files
```

---

### Form Handling

```
1. HTML form with action and method
2. Input fields with name attribute
3. express.urlencoded() middleware
4. Access data via req.body
5. Process and save data
```

---

### File Operations

```javascript
// Read
fs.readFileSync(path, "utf-8");

// Write (replaces)
fs.writeFileSync(path, data);

// Append
fs.appendFileSync(path, data);

// JSON
JSON.parse(string); // String to object
JSON.stringify(obj); // Object to string
```

---

## 14. Revision Checklist

### Middleware Basics

- [ ] Can you explain what middleware is?
- [ ] Do you know middleware parameters (req, res, next)?
- [ ] Can you explain next() function?
- [ ] Do you know execution order?
- [ ] Can you create custom middleware?

### Middleware Types

- [ ] Can you list 5 types of middleware?
- [ ] Can you create app-level middleware?
- [ ] Can you use built-in middleware?
- [ ] Do you know error middleware structure?

### Built-in Middleware

- [ ] Can you use express.json()?
- [ ] Can you use express.urlencoded()?
- [ ] Do you know extended: true vs false?
- [ ] Can you serve static files?

### Request Properties

- [ ] Can you access req.body?
- [ ] Can you access req.params?
- [ ] Can you access req.query?
- [ ] Do you know when to use each?

### Form Handling

- [ ] Can you create HTML form?
- [ ] Do you know form attributes (action, method)?
- [ ] Can you handle form submission?
- [ ] Can you access form data in Express?

### File Operations

- [ ] Can you read files with fs?
- [ ] Can you write files?
- [ ] Can you read JSON files?
- [ ] Can you write JSON files correctly?
- [ ] Do you know difference between write and append?

### Complete Flow

- [ ] Can you serve HTML form?
- [ ] Can you handle form POST request?
- [ ] Can you save data to JSON file?
- [ ] Can you read existing data first?
- [ ] Can you append new data correctly?

---

> **💡 Interview Tip — "Explain middleware in Express with an example"**
>
> **Answer like this:**
>
> _"Middleware in Express is a function that sits between the incoming request and the final route handler. It has access to the request object, response object, and a next() function._
>
> _The middleware function can perform tasks like logging, authentication, data parsing, or modifying the request/response objects. After completing its task, it calls next() to pass control to the next middleware or route handler._
>
> _Here's a practical example:_
>
> ```javascript
> // Logger middleware
> app.use((req, res, next) => {
>   console.log(`${req.method} ${req.url}`);
>   next(); // Pass to next middleware
> });
>
> // Authentication middleware
> app.use((req, res, next) => {
>   if (req.headers.authorization) {
>     next(); // Authenticated, continue
>   } else {
>     res.status(401).send("Unauthorized");
>     // No next(), stops here
>   }
> });
>
> // Route handler (final destination)
> app.get("/dashboard", (req, res) => {
>   res.send("Dashboard");
> });
> ```
>
> _In this example, every request first goes through the logger middleware which logs the request, then through authentication middleware which checks authorization. If authentication passes, next() is called and the request reaches the route handler._
>
> _There are five types of middleware in Express: app-level, router-level, built-in like express.json(), third-party like cors, and error-handling middleware which has four parameters including err._
>
> _The key point is that middleware executes in the order it's defined, and you must either call next() to continue the chain or send a response to end it. Otherwise, the request hangs."_
>
> **Why this works:** Clear definition, explains next() function, provides working code example, mentions middleware types, explains execution flow, and highlights common pitfall (missing next()).
