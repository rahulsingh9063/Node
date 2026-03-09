# Express.js Complete Guide — Routing, Controllers & Middleware

### From Basic Setup to Modular Architecture

---

## 📚 What You'll Learn

This guide covers **Express.js fundamentals and best practices**:

✅ Express setup and initialization  
✅ HTTP methods (GET, POST, PUT, PATCH, DELETE)  
✅ Response methods (res.send(), res.end(), res.json())  
✅ Routing basics  
✅ Controllers pattern (separating logic)  
✅ Express Router for modular routing  
✅ Middleware concept with app.use()  
✅ File structure best practices  
✅ Nodemon for development

**Best for:** Express beginners, building REST APIs, interview preparation

---

## Table of Contents

1. [Express Setup — 4 Essential Steps](#1-express-setup--4-essential-steps)
2. [HTTP Methods](#2-http-methods)
3. [Response Methods Comparison](#3-response-methods-comparison)
4. [Basic Routing](#4-basic-routing)
5. [Controllers — Separating Logic](#5-controllers--separating-logic)
6. [Express Router — Modular Routing](#6-express-router--modular-routing)
7. [Middleware with app.use()](#7-middleware-with-appuse)
8. [Complete File Structure](#8-complete-file-structure)
9. [Nodemon for Auto-Restart](#9-nodemon-for-auto-restart)
10. [Best Practices](#10-best-practices)
11. [Summary](#11-summary)
12. [Revision Checklist](#12-revision-checklist)

---

## 1. Express Setup — 4 Essential Steps

### The 4 Steps to Create Express Server

**Every Express application follows these 4 steps:**

```javascript
//! STEP 1: Import express
import express from "express";

//! STEP 2: Create app instance by calling express()
let app = express();

//! STEP 3: Define routes
app.get("/", (req, res) => {
  res.send("Hello World");
});

//! STEP 4: Start server on a port
app.listen(3000, (err) => {
  if (err) throw err;
  console.log("Server running on port 3000");
});
```

---

### Step-by-Step Explanation

#### Step 1: Import Express

```javascript
import express from "express";
```

**What it does:**

- Imports the Express library
- `express` is a function you'll call to create your app

---

#### Step 2: Create App Instance

```javascript
let app = express();
```

**What it does:**

- Calls `express()` function
- Returns an `app` object
- This `app` object is your entire application
- You'll use it to define routes, middleware, etc.

**Type:**

```javascript
console.log(typeof express); // "function"
console.log(typeof app); // "object"
```

---

#### Step 3: Define Routes

```javascript
app.get("/", (req, res) => {
  res.send("Hello World");
});
```

**What it does:**

- Defines what happens when someone visits a URL
- Format: `app.METHOD(PATH, HANDLER)`
  - `METHOD` = HTTP method (get, post, put, etc.)
  - `PATH` = URL endpoint
  - `HANDLER` = function to execute

---

#### Step 4: Start Server

```javascript
app.listen(3000, (err) => {
  if (err) throw err;
  console.log("Server running on port 3000");
});
```

**What it does:**

- Starts HTTP server
- Listens on port 3000
- Callback runs when server starts successfully

**Parameters:**

- `3000` = port number
- `callback` = function to run after server starts

---

### Visual: Express App Flow

```
┌──────────────────────────────────────────────────────┐
│  1. Import express                                   │
│     import express from "express";                   │
└────────────────┬─────────────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────────────┐
│  2. Create app                                       │
│     let app = express();                             │
└────────────────┬─────────────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────────────┐
│  3. Define routes                                    │
│     app.get("/", handler);                           │
│     app.post("/users", handler);                     │
└────────────────┬─────────────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────────────┐
│  4. Start server                                     │
│     app.listen(3000);                                │
└──────────────────────────────────────────────────────┘
                 │
                 ↓
         Server Running ✅
```

---

## 2. HTTP Methods

### What are HTTP Methods?

**HTTP methods define the TYPE of operation:**

```
GET     → Retrieve data (Read)
POST    → Create new data
PUT     → Update entire resource
PATCH   → Update partial resource
DELETE  → Delete data
```

---

### Express Syntax for HTTP Methods

```javascript
// Format: app.METHOD(path, handler)

app.get("/users", handler); // GET request
app.post("/users", handler); // POST request
app.put("/users/:id", handler); // PUT request
app.patch("/users/:id", handler); // PATCH request
app.delete("/users/:id", handler); // DELETE request
```

---

### HTTP Methods with Examples

#### GET — Retrieve Data

```javascript
app.get("/users", (req, res) => {
  res.json({ users: ["Alice", "Bob", "Charlie"] });
});

// Browser request: GET http://localhost:3000/users
// Response: { "users": ["Alice", "Bob", "Charlie"] }
```

**Use for:**

- Fetching list of items
- Getting single item details
- Reading data (no modifications)

---

#### POST — Create Data

```javascript
app.post("/users", (req, res) => {
  // req.body contains data sent by client
  const newUser = req.body;

  res.status(201).json({
    success: true,
    message: "User created",
    data: newUser,
  });
});

// Request: POST http://localhost:3000/users
// Body: { "name": "Alice", "age": 25 }
```

**Use for:**

- Creating new records
- Submitting forms
- Adding new data

---

#### PUT — Update Entire Resource

```javascript
app.put("/users/:id", (req, res) => {
  const userId = req.params.id;
  const updatedUser = req.body;

  res.json({
    success: true,
    message: "User updated",
    data: updatedUser,
  });
});

// Request: PUT http://localhost:3000/users/123
// Body: { "name": "Alice", "age": 26, "email": "alice@example.com" }
// Replaces ENTIRE user object
```

**Use for:**

- Replacing entire resource
- Complete update

---

#### PATCH — Update Partial Resource

```javascript
app.patch("/users/:id", (req, res) => {
  const userId = req.params.id;
  const updates = req.body;

  res.json({
    success: true,
    message: "User updated",
    data: updates,
  });
});

// Request: PATCH http://localhost:3000/users/123
// Body: { "age": 26 }
// Updates ONLY age, keeps other fields
```

**Use for:**

- Partial updates
- Modifying specific fields

---

#### DELETE — Remove Data

```javascript
app.delete("/users/:id", (req, res) => {
  const userId = req.params.id;

  res.json({
    success: true,
    message: "User deleted",
  });
});

// Request: DELETE http://localhost:3000/users/123
```

**Use for:**

- Deleting records
- Removing data

---

### PUT vs PATCH

```javascript
// Original user:
{ id: 1, name: "Alice", age: 25, email: "alice@example.com" }

// PUT (Replace everything):
PUT /users/1
Body: { name: "Alice", age: 26 }
Result: { id: 1, name: "Alice", age: 26 }
        ↑ email is GONE (not provided)

// PATCH (Update specific fields):
PATCH /users/1
Body: { age: 26 }
Result: { id: 1, name: "Alice", age: 26, email: "alice@example.com" }
        ↑ email is KEPT (only age changed)
```

---

## 3. Response Methods Comparison

### The Three Main Response Methods

```javascript
res.end(); // Send response and end
res.send(); // Smart send (auto content-type)
res.json(); // Send JSON data
```

---

### res.end()

**Sends raw data and ends response.**

```javascript
app.get("/", (req, res) => {
  res.end("Hello World");
});

// Must manually set headers
app.get("/json", (req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ name: "Varun" }));
});
```

**Characteristics:**

- Basic response method
- No automatic content-type detection
- Must manually set headers for JSON
- From Node.js http module

---

### res.send()

**Smart response method that auto-detects content type.**

```javascript
app.get("/", (req, res) => {
  res.send("Hello World");
  // Content-Type: text/html
});

app.get("/data", (req, res) => {
  res.send({ name: "Varun" });
  // Content-Type: application/json (auto-detected!)
});

app.get("/buffer", (req, res) => {
  res.send(Buffer.from("Hello"));
  // Content-Type: application/octet-stream
});
```

**Characteristics:**

- Express method (not in raw Node.js)
- Automatically sets Content-Type
- Can send strings, objects, buffers, arrays
- Converts objects to JSON automatically

---

### res.json()

**Explicitly sends JSON response.**

```javascript
app.get("/json", (req, res) => {
  res.json({
    success: true,
    message: "Data retrieved",
    data: { name: "Varun" },
  });
  // Content-Type: application/json
});

// With status code
app.get("/users", (req, res) => {
  res.status(200).json({
    success: true,
    users: ["Alice", "Bob"],
  });
});
```

**Characteristics:**

- Express method
- Always sends JSON
- Sets Content-Type to application/json
- Can chain with .status()

---

### Comparison Table

| Method       | Auto Content-Type | Use For           | From    |
| ------------ | ----------------- | ----------------- | ------- |
| `res.end()`  | ❌ No             | Raw responses     | Node.js |
| `res.send()` | ✅ Yes            | Any response type | Express |
| `res.json()` | ✅ Yes (JSON)     | JSON data only    | Express |

---

### Which to Use?

```javascript
// ✅ RECOMMENDED for JSON APIs:
res.json({ data: "value" });

// ✅ RECOMMENDED for text/HTML:
res.send("Hello World");

// ❌ AVOID unless you need manual control:
res.end("data");
```

---

### Status Codes with Responses

```javascript
// Method 1: Using .status()
app.get("/users", (req, res) => {
  res.status(200).json({ users: [] });
});

// Method 2: Using .status() separately
app.get("/error", (req, res) => {
  res.status(404);
  res.json({ error: "Not found" });
});

// Common status codes:
res.status(200); // OK
res.status(201); // Created
res.status(400); // Bad Request
res.status(404); // Not Found
res.status(500); // Server Error
```

---

## 4. Basic Routing

### What is Routing?

**Routing determines how the application responds to client requests at specific endpoints (URLs).**

---

### Basic Route Structure

```javascript
app.METHOD(PATH, HANDLER);

// METHOD  = HTTP method (get, post, etc.)
// PATH    = URL endpoint ("/", "/about", "/users")
// HANDLER = Function to execute (req, res) => {}
```

---

### Simple Routes

```javascript
import express from "express";
let app = express();

// Route 1: Home page
app.get("/", (req, res) => {
  res.send("Home Page");
});

// Route 2: About page
app.get("/about", (req, res) => {
  res.end("About Page");
});

// Route 3: JSON response
app.get("/json", (req, res) => {
  res.status(200).json({
    success: true,
    message: "Data retrieved",
    data: {},
    error: {},
  });
});

app.listen(3000, (err) => {
  if (err) throw err;
  console.log("Server running on port 3000");
});
```

---

### Route Parameters

```javascript
// Dynamic route with parameter
app.get("/users/:id", (req, res) => {
  const userId = req.params.id;

  res.json({
    message: `User ID is ${userId}`,
  });
});

// Request: GET /users/123
// Response: { "message": "User ID is 123" }

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req, res) => {
  const { userId, postId } = req.params;

  res.json({
    userId: userId,
    postId: postId,
  });
});

// Request: GET /users/123/posts/456
// Response: { "userId": "123", "postId": "456" }
```

---

### Query Parameters

```javascript
app.get("/search", (req, res) => {
  const { q, limit } = req.query;

  res.json({
    query: q,
    limit: limit,
  });
});

// Request: GET /search?q=javascript&limit=10
// Response: { "query": "javascript", "limit": "10" }
```

---

## 5. Controllers — Separating Logic

### What are Controllers?

**Controllers contain the logic for handling requests. They separate route definitions from business logic.**

---

### Why Use Controllers?

```
✅ Separation of Concerns
✅ Reusable code
✅ Easier testing
✅ Better organization
✅ Follows MVC pattern
```

---

### Without Controllers (Bad Practice)

```javascript
// app.js - Everything in one file ❌

app.get("/", (req, res) => {
  // Logic here
  res.send("Home Page");
});

app.get("/about", (req, res) => {
  // Logic here
  res.end("About Page");
});

app.get("/json", (req, res) => {
  // Logic here
  res.status(200).json({ success: true });
});

// Problem: Logic mixed with routing
// Hard to test, hard to reuse
```

---

### With Controllers (Best Practice)

**File: controllers/controller.js**

```javascript
// Separate file for logic ✅

export let displayHomePage = (req, res) => {
  res.send("Home Page");
};

export let displayAboutPage = (req, res) => {
  res.end("About Page");
};

export let sendJSON = (req, res) => {
  res.status(202).json({
    success: true,
    message: "",
    data: {},
    error: {},
  });
};
```

---

**File: app.js**

```javascript
import express from "express";
import {
  displayHomePage,
  displayAboutPage,
  sendJSON,
} from "./controllers/controller.js";

let app = express();

// Clean routing - logic is separated
app.get("/", displayHomePage);
app.get("/about", displayAboutPage);
app.get("/json", sendJSON);

app.listen(3000, (err) => {
  if (err) throw err;
  console.log("Server running...");
});
```

---

### Benefits of Controllers

```
BEFORE (without controllers):
────────────────────────────────────────
app.js (200 lines)
  - Route definitions
  - Business logic
  - Database queries
  - Everything mixed together


AFTER (with controllers):
────────────────────────────────────────
app.js (20 lines)
  - Only routing

controllers/controller.js (180 lines)
  - All business logic
  - Easy to test
  - Reusable functions
```

---

### Controller Pattern

```javascript
// controllers/userController.js

export const getUsers = (req, res) => {
  // Logic to get all users
  const users = ["Alice", "Bob", "Charlie"];

  res.status(200).json({
    success: true,
    data: users,
  });
};

export const getUserById = (req, res) => {
  const { id } = req.params;

  // Logic to get user by ID
  res.status(200).json({
    success: true,
    data: { id, name: "Alice" },
  });
};

export const createUser = (req, res) => {
  const userData = req.body;

  // Logic to create user
  res.status(201).json({
    success: true,
    message: "User created",
    data: userData,
  });
};
```

---

## 6. Express Router — Modular Routing

### What is Express Router?

**Router is a mini Express application capable of handling middleware and routes. It allows you to organize routes into separate files.**

---

### Why Use Router?

```
✅ Modular code structure
✅ Separate files for different resources
✅ Better organization for large apps
✅ Each module is independent
✅ Easier to maintain
```

---

### The 3 Steps to Use Router

**Every route file follows these 3 steps:**

```javascript
//! STEP 1: Import Router from express
import { Router } from "express";

//! STEP 2: Create router instance
let router = Router();

//! STEP 3: Export router
export default router;
```

---

### Complete Router Example

**File: routes/route.js**

```javascript
//! STEP 1: Import Router
import { Router } from "express";

// Import controllers
import {
  displayAboutPage,
  displayHomePage,
  sendJSON,
} from "../controllers/controller.js";

//! STEP 2: Create router instance
let router = Router();

// Define routes on router (not app)
router.get("/", displayHomePage);
router.get("/about", displayAboutPage);
router.get("/json", sendJSON);

//! STEP 3: Export router
export default router;
```

---

**File: app.js**

```javascript
import express from "express";
let app = express();

//! Import route file
import myRoutes from "./routes/route.js";

//! Use router as middleware
app.use(myRoutes);

app.listen(3000, (err) => {
  if (err) throw err;
  console.log("Server running...");
});
```

---

### Multiple Route Files

**File: routes/userRoutes.js**

```javascript
import { Router } from "express";
import { getUsers, createUser } from "../controllers/userController.js";

let router = Router();

router.get("/users", getUsers);
router.post("/users", createUser);

export default router;
```

---

**File: routes/productRoutes.js**

```javascript
import { Router } from "express";
import {
  getProducts,
  createProduct,
} from "../controllers/productController.js";

let router = Router();

router.get("/products", getProducts);
router.post("/products", createProduct);

export default router;
```

---

**File: app.js**

```javascript
import express from "express";
let app = express();

// Import multiple route files
import userRoutes from "./routes/userRoutes.js";
import productRoutes from "./routes/productRoutes.js";

// Use all routes
app.use(userRoutes);
app.use(productRoutes);

app.listen(3000);
```

---

### Router with Base Path

```javascript
// app.js

import userRoutes from "./routes/userRoutes.js";
import productRoutes from "./routes/productRoutes.js";

// Add base path to all routes in the router
app.use("/api", userRoutes);
app.use("/api", productRoutes);

// Now routes become:
// /api/users
// /api/products
```

---

**routes/userRoutes.js**

```javascript
import { Router } from "express";
let router = Router();

// These routes will be prefixed with /api
router.get("/users", handler); // Becomes /api/users
router.post("/users", handler); // Becomes /api/users
router.get("/users/:id", handler); // Becomes /api/users/:id

export default router;
```

---

## 7. Middleware with app.use()

### What is Middleware?

**Middleware are functions that have access to the request (req), response (res), and next middleware function. They execute in sequence.**

---

### Middleware Flow

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

---

### app.use() Syntax

```javascript
// Use router as middleware
app.use(myRoutes);

// Use built-in middleware
app.use(express.json());

// Use custom middleware
app.use((req, res, next) => {
  console.log("Middleware executed");
  next(); // Pass to next middleware/route
});
```

---

### Router as Middleware

```javascript
import express from "express";
let app = express();

import myRoutes from "./routes/route.js";

// Router is used as middleware
app.use(myRoutes);

// Equivalent to:
// app.use("/", myRoutes);
```

---

### Built-in Middleware

```javascript
import express from "express";
let app = express();

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static("public"));

// Now you can access req.body in routes
app.post("/users", (req, res) => {
  console.log(req.body); // JSON data from client
  res.json({ received: req.body });
});
```

---

### Custom Middleware

```javascript
// Logger middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // MUST call next() to continue
});

// Authentication middleware
app.use((req, res, next) => {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ error: "No token" });
  }

  // Token is valid, continue
  next();
});

// Routes come after middleware
app.get("/users", (req, res) => {
  res.json({ users: [] });
});
```

---

### Middleware Execution Order

```javascript
import express from "express";
let app = express();

// Middleware 1
app.use((req, res, next) => {
  console.log("1. First middleware");
  next();
});

// Middleware 2
app.use(express.json());

// Middleware 3
app.use((req, res, next) => {
  console.log("3. Third middleware");
  next();
});

// Route handler
app.get("/", (req, res) => {
  console.log("4. Route handler");
  res.send("Done");
});

// Execution order: 1 → 2 → 3 → 4
```

---

## 8. Complete File Structure

### Organized Project Structure

```
my-express-app/
│
├── controllers/
│   ├── userController.js
│   ├── productController.js
│   └── authController.js
│
├── routes/
│   ├── userRoutes.js
│   ├── productRoutes.js
│   └── authRoutes.js
│
├── app.js
├── package.json
└── node_modules/
```

---

### File: controllers/userController.js

```javascript
// All user-related logic

export const getUsers = (req, res) => {
  res.status(200).json({
    success: true,
    data: ["Alice", "Bob", "Charlie"],
  });
};

export const getUserById = (req, res) => {
  const { id } = req.params;

  res.status(200).json({
    success: true,
    data: { id, name: "Alice" },
  });
};

export const createUser = (req, res) => {
  const userData = req.body;

  res.status(201).json({
    success: true,
    message: "User created",
    data: userData,
  });
};

export const updateUser = (req, res) => {
  const { id } = req.params;
  const updates = req.body;

  res.status(200).json({
    success: true,
    message: "User updated",
    data: { id, ...updates },
  });
};

export const deleteUser = (req, res) => {
  const { id } = req.params;

  res.status(200).json({
    success: true,
    message: "User deleted",
  });
};
```

---

### File: routes/userRoutes.js

```javascript
//! STEP 1: Import Router
import { Router } from "express";

// Import controllers
import {
  getUsers,
  getUserById,
  createUser,
  updateUser,
  deleteUser,
} from "../controllers/userController.js";

//! STEP 2: Create router
let router = Router();

// Define routes
router.get("/users", getUsers);
router.get("/users/:id", getUserById);
router.post("/users", createUser);
router.put("/users/:id", updateUser);
router.delete("/users/:id", deleteUser);

//! STEP 3: Export router
export default router;
```

---

### File: app.js

```javascript
//! a) Import express
import express from "express";

//! b) Create app
let app = express();

//! Middleware
app.use(express.json()); // Parse JSON bodies

//! Import routes
import userRoutes from "./routes/userRoutes.js";
import productRoutes from "./routes/productRoutes.js";

//! Use routes
app.use("/api", userRoutes);
app.use("/api", productRoutes);

//! c) Start server
app.listen(3000, (err) => {
  if (err) throw err;
  console.log("Server running on port 3000");
});
```

---

### API Endpoints Available

```
GET    /api/users           → Get all users
GET    /api/users/:id       → Get user by ID
POST   /api/users           → Create new user
PUT    /api/users/:id       → Update user
DELETE /api/users/:id       → Delete user

GET    /api/products        → Get all products
GET    /api/products/:id    → Get product by ID
POST   /api/products        → Create new product
PUT    /api/products/:id    → Update product
DELETE /api/products/:id    → Delete product
```

---

## 9. Nodemon for Auto-Restart

### What is Nodemon?

**Nodemon automatically restarts your Node.js server when file changes are detected.**

---

### Without Nodemon

```
1. Edit code
2. Stop server (Ctrl + C)
3. Restart server (node app.js)
4. Test
5. Repeat for every change ❌
```

---

### With Nodemon

```
1. Edit code
2. Nodemon auto-restarts ✅
3. Test immediately
```

---

### Installation

```bash
# Install globally (recommended)
npm install -g nodemon

# Or install locally in project
npm install nodemon --save-dev
```

---

### Usage

```bash
# Instead of:
node app.js

# Use:
nodemon app.js
```

---

### Output

```
[nodemon] starting `node app.js`
Server running on port 3000

# Edit a file and save

[nodemon] restarting due to changes...
[nodemon] starting `node app.js`
Server running on port 3000
```

---

### package.json Script

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
}
```

**Run with:**

```bash
npm run dev
```

---

## 10. Best Practices

### 1. Use Controllers

```javascript
// ❌ BAD: Logic in routes
app.get("/users", (req, res) => {
  // 50 lines of logic here
});

// ✅ GOOD: Logic in controllers
app.get("/users", getUsersController);
```

---

### 2. Use Router for Modular Code

```javascript
// ❌ BAD: Everything in app.js
app.get("/users", handler);
app.post("/users", handler);
app.get("/products", handler);
// 100 routes in one file

// ✅ GOOD: Separate route files
import userRoutes from "./routes/userRoutes.js";
import productRoutes from "./routes/productRoutes.js";
app.use("/api", userRoutes);
app.use("/api", productRoutes);
```

---

### 3. Use Middleware for Common Tasks

```javascript
// ✅ GOOD: Middleware for parsing JSON
app.use(express.json());

// ✅ GOOD: Middleware for logging
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

---

### 4. Consistent Response Format

```javascript
// ✅ GOOD: Standard response structure
res.json({
  success: true,
  message: "Operation successful",
  data: {},
  error: null,
});

// On error:
res.status(400).json({
  success: false,
  message: "Validation failed",
  data: null,
  error: { field: "email", reason: "Invalid format" },
});
```

---

### 5. Use Status Codes Correctly

```javascript
// ✅ GOOD: Appropriate status codes
res.status(200).json({ data }); // OK
res.status(201).json({ data }); // Created
res.status(400).json({ error }); // Bad Request
res.status(404).json({ error }); // Not Found
res.status(500).json({ error }); // Server Error
```

---

### 6. Error Handling

```javascript
// ✅ GOOD: Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);

  res.status(500).json({
    success: false,
    message: "Internal server error",
    error: err.message,
  });
});
```

---

## 11. Summary

### Express Setup — 4 Steps

```javascript
1. import express from "express";
2. let app = express();
3. app.get("/", handler);
4. app.listen(3000);
```

---

### HTTP Methods

```
GET    → Read data
POST   → Create data
PUT    → Update entire resource
PATCH  → Update partial resource
DELETE → Delete data
```

---

### Response Methods

```
res.end()   → Basic response (manual headers)
res.send()  → Smart response (auto content-type)
res.json()  → JSON response (recommended for APIs)
```

---

### Controllers Pattern

```
routes/      → Routing logic
controllers/ → Business logic
app.js       → App setup and middleware
```

---

### Router — 3 Steps

```javascript
1. import { Router } from "express";
2. let router = Router();
3. export default router;
```

---

### Middleware

```javascript
app.use(express.json()); // Parse JSON
app.use("/api", routes); // Mount router
app.use((req, res, next) => {}); // Custom middleware
```

---

## 12. Revision Checklist

### Basic Setup

- [ ] Can you import express?
- [ ] Can you create app with express()?
- [ ] Can you define routes?
- [ ] Can you start server with app.listen()?

### HTTP Methods

- [ ] Do you know GET, POST, PUT, PATCH, DELETE?
- [ ] Can you use app.get(), app.post(), etc.?
- [ ] Do you know difference between PUT and PATCH?

### Responses

- [ ] Can you use res.send()?
- [ ] Can you use res.json()?
- [ ] Can you use res.status()?
- [ ] Do you know when to use each method?

### Controllers

- [ ] Can you separate logic into controllers?
- [ ] Can you export controller functions?
- [ ] Can you import and use controllers in routes?

### Router

- [ ] Can you import Router from express?
- [ ] Can you create router instance?
- [ ] Can you define routes on router?
- [ ] Can you export router?
- [ ] Can you use router with app.use()?

### Middleware

- [ ] Do you understand what middleware is?
- [ ] Can you use app.use() for middleware?
- [ ] Can you use express.json()?
- [ ] Do you know middleware execution order?

### File Structure

- [ ] Can you organize code into folders?
- [ ] Do you know where to put controllers?
- [ ] Do you know where to put routes?
- [ ] Can you structure a complete Express app?

### Nodemon

- [ ] Can you install nodemon?
- [ ] Can you run app with nodemon?
- [ ] Do you know the benefit of nodemon?

---

> **💡 Interview Tip — "Explain the MVC pattern in Express"**
>
> **Answer like this:**
>
> _"In Express, we use the MVC (Model-View-Controller) pattern to organize code:_
>
> _The Controller layer contains business logic and is where we handle requests. I separate this into controller files with exported functions._
>
> _The Route layer defines URL endpoints and maps them to controllers. I use Express Router to modularize routes into separate files._
>
> _For example, I'd create userController.js with functions like getUsers, createUser, etc. Then in userRoutes.js, I import Router, define routes like router.get('/users', getUsers), and export the router. Finally, in app.js, I import the router and use app.use('/api', userRoutes)._
>
> _This separation gives us:_
>
> - _Clean code organization_
> - _Reusable controllers_
> - _Easy testing_
> - _Better maintainability_
>
> _The key is separating routing from logic using controllers and organizing routes with Express Router."_
