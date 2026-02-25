# MongoDB with Node.js — Complete Integration Guide

### npm, package.json, MongoClient & Database Operations

---

## 📚 What You'll Learn

This guide covers **connecting MongoDB to Node.js applications**:

✅ npm init and package.json  
✅ Installing third-party packages  
✅ node_modules and dependencies  
✅ MongoClient connection setup  
✅ Creating databases and collections  
✅ All CRUD operations from Node.js  
✅ Cursors and toArray()  
✅ Complete working examples

**Best for:** Building full-stack apps, backend development, interview preparation

---

## Table of Contents

1. [package.json — The Project Manifest](#1-packagejson--the-project-manifest)
2. [npm init — Creating package.json](#2-npm-init--creating-packagejson)
3. [Installing Packages with npm](#3-installing-packages-with-npm)
4. [The Three Changes After Install](#4-the-three-changes-after-install)
5. [node_modules and package-lock.json](#5-node_modules-and-package-lockjson)
6. [MongoDB Driver for Node.js](#6-mongodb-driver-for-nodejs)
7. [The 4 Steps to Use MongoDB](#7-the-4-steps-to-use-mongodb)
8. [Step 1 — Creating a Connection](#8-step-1--creating-a-connection)
9. [Step 2 — Creating/Using a Database](#9-step-2--creatingusing-a-database)
10. [Step 3 — Creating/Using a Collection](#10-step-3--creatingusing-a-collection)
11. [Step 4 — CRUD Operations](#11-step-4--crud-operations)
12. [Cursors and toArray()](#12-cursors-and-toarray)
13. [Complete Working Example](#13-complete-working-example)
14. [Best Practices](#14-best-practices)
15. [Summary](#15-summary)
16. [Revision Checklist](#16-revision-checklist)

---

## 1. package.json — The Project Manifest

### What is package.json?

**package.json is THE most important file in any Node.js project.** It's like a **passport** for your project — contains all essential information.

Think of it like:

- **Recipe card** for your project
- **Instruction manual** for others
- **Dependency list** (what packages you need)

---

### What's Inside package.json?

```json
{
  "name": "my-project", // Project name
  "version": "1.0.0", // Version number
  "description": "MongoDB with Node", // What it does
  "main": "index.js", // Entry point
  "scripts": {
    // Commands
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "keywords": ["mongodb", "node"], // Search keywords
  "author": "Your Name", // Who made it
  "license": "MIT", // License type
  "dependencies": {
    // Required packages
    "mongodb": "^6.3.0"
  }
}
```

---

### Why Do We Need It?

**Without package.json:**

```
❌ Can't install packages
❌ Can't share project easily
❌ Others can't run your project
❌ No way to track dependencies
```

**With package.json:**

```
✅ npm install works
✅ Others can run: npm install → Start immediately
✅ All dependencies tracked
✅ Version control
✅ Professional project structure
```

---

### The Most Important Part — dependencies

```json
"dependencies": {
  "mongodb": "^6.3.0",
  "express": "^4.18.2",
  "dotenv": "^16.0.3"
}
```

**This tells Node.js:** "My project needs these packages to run!"

When someone downloads your project:

```bash
$ npm install
# Reads dependencies from package.json
# Downloads all packages automatically
# Project ready to run!
```

---

### ⚠️ Must Be Unique

**Each project folder needs ITS OWN package.json:**

```
Correct:
────────────────────────────────
project1/
  ├── package.json  ✅
  └── index.js

project2/
  ├── package.json  ✅
  └── server.js

Wrong:
────────────────────────────────
projects/
  ├── package.json  ❌ (only one for multiple projects)
  ├── project1/
  └── project2/
```

**One folder = One package.json**

---

## 2. npm init — Creating package.json

### What is npm init?

**npm init is the command that creates package.json file** by asking you questions.

---

### Method 1: Interactive (Answer Questions)

```bash
$ npm init

# It will ask you:
package name: (my-project)
version: (1.0.0)
description: MongoDB with Node.js
entry point: (index.js)
test command:
git repository:
keywords: mongodb, node
author: Your Name
license: (ISC) MIT

Is this OK? (yes)
```

**Result:** Creates `package.json` with your answers

---

### Method 2: Quick Setup (Default Values)

```bash
$ npm init -y

# Creates package.json instantly with defaults:
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

**The `-y` flag = "Yes to all questions"** (uses defaults)

---

### When to Use Which?

| Use Case               | Command       | Why                             |
| ---------------------- | ------------- | ------------------------------- |
| **Learning/Practice**  | `npm init -y` | Fast, don't need customization  |
| **Quick prototype**    | `npm init -y` | Speed over detail               |
| **Production project** | `npm init`    | Want control over all fields    |
| **Open source**        | `npm init`    | Need proper description, author |

**Most developers use:** `npm init -y` → Then edit manually

---

### Visual: Project Setup Flow

```mermaid
graph TD
    A[New Project Folder] -->|Create| B[Run npm init -y]
    B --> C[package.json Created]
    C --> D[Install Packages]
    D --> E[Write Code]
    E --> F[Ready to Run!]

    style C fill:#c8e6c9
```

---

## 3. Installing Packages with npm

### What is npm?

**npm = Node Package Manager**

Think of it like:

- **App Store** for Node.js packages
- **Library** with millions of code modules
- **Package delivery service**

---

### Installing a Single Package

```bash
# Syntax:
npm install packageName
npm i packageName        # Shorthand (same thing)

# Example:
npm install mongodb
npm i mongodb           # Same!
```

**What happens:**

1. Downloads `mongodb` package from npm registry
2. Saves it in `node_modules/` folder
3. Adds it to `package.json` dependencies
4. Creates/updates `package-lock.json`

---

### Installing Multiple Packages

```bash
# Install multiple at once:
npm install mongodb express dotenv

# Or:
npm i mongodb express dotenv
```

**Saves time!** One command instead of three.

---

### Install vs Import

**Two separate steps:**

```bash
# Step 1: INSTALL (download the package)
npm install mongodb
```

```javascript
// Step 2: IMPORT (use it in your code)
import { MongoClient } from "mongodb";
```

**Think of it like:**

- Install = Buying a book
- Import = Opening the book to read

---

### Types of Dependencies

```bash
# Regular dependency (needed to run)
npm install mongodb

# Dev dependency (only needed during development)
npm install --save-dev nodemon
npm install -D nodemon      # Shorthand
```

**package.json will show:**

```json
{
  "dependencies": {
    "mongodb": "^6.3.0" // Needed in production
  },
  "devDependencies": {
    "nodemon": "^3.0.1" // Only for development
  }
}
```

---

## 4. The Three Changes After Install

### What Happens When You Install?

**Three things change in your project:**

```
Before npm install mongodb:
────────────────────────────────
my-project/
  ├── package.json
  └── index.js

After npm install mongodb:
────────────────────────────────
my-project/
  ├── package.json       ← Updated! (added dependency)
  ├── package-lock.json  ← Created!
  ├── node_modules/      ← Created! (contains packages)
  └── index.js
```

---

### Change 1: package.json Updated

**Before:**

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "dependencies": {}     ← Empty
}
```

**After:**

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "dependencies": {
    "mongodb": "^6.3.0"  ← Added!
  }
}
```

---

### Change 2: node_modules Folder Created

**node_modules/ contains:**

- The actual source code of `mongodb` package
- All its dependencies (packages it needs)
- Can be 100+ MB in size!

```
node_modules/
  ├── mongodb/            ← The package you installed
  │   ├── lib/
  │   ├── index.js
  │   └── package.json
  ├── bson/               ← mongodb's dependency
  ├── saslprep/           ← mongodb's dependency
  └── ... (50+ folders)   ← All dependencies
```

**⚠️ NEVER edit files in node_modules!**

---

### Change 3: package-lock.json Created

**package-lock.json contains:**

- Exact versions of ALL packages
- Dependency tree (what depends on what)
- Download URLs
- Checksums for security

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "packages": {
    "node_modules/mongodb": {
      "version": "6.3.0",
      "resolved": "https://registry.npmjs.org/mongodb/-/mongodb-6.3.0.tgz",
      "dependencies": {
        "bson": "^6.2.0",
        "saslprep": "^1.0.3"
      }
    }
  }
}
```

**Why it matters:** Ensures everyone gets EXACTLY the same versions.

---

### Visual: The Three Changes

```
npm install mongodb
       ↓
┌──────────────────────────────────────┐
│  Change 1: package.json              │
│  "dependencies": {                   │
│    "mongodb": "^6.3.0"  ← Added!    │
│  }                                   │
└──────────────────────────────────────┘
       ↓
┌──────────────────────────────────────┐
│  Change 2: node_modules/             │
│  ├── mongodb/           ← Created!   │
│  ├── bson/                           │
│  └── ... (all dependencies)          │
└──────────────────────────────────────┘
       ↓
┌──────────────────────────────────────┐
│  Change 3: package-lock.json         │
│  Exact versions locked  ← Created!   │
└──────────────────────────────────────┘
```

---

## 5. node_modules and package-lock.json

### What is node_modules?

**The folder where ALL installed packages live.**

Think of it like:

- **Library bookshelf** — holds all your books (packages)
- **Warehouse** — stores inventory (code)
- **Dependencies folder** — everything your project needs

---

### Why So Big?

```bash
$ npm install mongodb

# node_modules/ size:
50 MB - 200 MB (just for one package!)

# Why?
mongodb needs: bson, saslprep, etc.
bson needs: more packages
saslprep needs: more packages
... (dependency chain continues)
```

**One package = Many dependencies = Large folder**

---

### ⚠️ Should You Commit node_modules?

**NO! NEVER commit node_modules to Git!**

```bash
# .gitignore file:
node_modules/    ← ALWAYS ignore this folder
```

**Why?**

- Too large (100+ MB)
- Unnecessary (others can install with `npm install`)
- Causes merge conflicts

**Instead:**

- Commit `package.json` ✅
- Commit `package-lock.json` ✅
- Others run `npm install` to get packages

---

### What is package-lock.json?

**A snapshot of your exact dependency tree.**

Think of it like:

- **Receipt** with exact prices (exact versions)
- **Fingerprint** of your dependencies
- **Time capsule** — preserves exact state

---

### package.json vs package-lock.json

```json
// package.json (flexible versions):
{
  "dependencies": {
    "mongodb": "^6.3.0"    // Can install 6.3.x or 6.4.x
  }
}

// package-lock.json (exact versions):
{
  "packages": {
    "node_modules/mongodb": {
      "version": "6.3.0"   // EXACTLY 6.3.0
    }
  }
}
```

**package.json:** "I need mongodb version 6.x"  
**package-lock.json:** "I need EXACTLY mongodb 6.3.0"

---

### Why Both Files?

| File                  | Purpose              | Commit to Git? |
| --------------------- | -------------------- | -------------- |
| **package.json**      | Lists what you need  | ✅ Yes         |
| **package-lock.json** | Lists exact versions | ✅ Yes         |
| **node_modules/**     | Contains actual code | ❌ No          |

---

### The Workflow

```
Developer A (creates project):
───────────────────────────────────
$ npm init -y
$ npm install mongodb
→ Creates package.json, package-lock.json, node_modules/
→ Commits package.json + package-lock.json to Git
→ Ignores node_modules/

Developer B (clones project):
───────────────────────────────────
$ git clone project
$ npm install
→ Reads package-lock.json
→ Downloads EXACT same versions
→ Creates node_modules/
→ Project ready to run!
```

---

## 6. MongoDB Driver for Node.js

### What is the MongoDB Driver?

**A package that lets Node.js talk to MongoDB.**

Think of it like:

- **Translator** between JavaScript and MongoDB
- **Bridge** connecting two systems
- **API client** for database operations

---

### Installing MongoDB Driver

```bash
$ npm install mongodb

# This installs the official MongoDB driver for Node.js
```

---

### Importing MongoClient

```javascript
// Method 1: Import entire package
import mongodb from "mongodb";
console.log(mongodb.MongoClient); // MongoClient class

// Method 2: Import just what you need (better)
import { MongoClient } from "mongodb";
console.log(MongoClient); // MongoClient class
```

**Method 2 is preferred** — cleaner and more efficient.

---

### What is MongoClient?

**MongoClient is a class that handles database connections.**

Think of it like:

- **Phone** — used to make calls (connections)
- **Key** — opens the door to database
- **Connection manager**

---

## 7. The 4 Steps to Use MongoDB

### The Complete Workflow

```
Step 1: Create Connection  → Connect to MongoDB server
         ↓
Step 2: Create/Use Database → Choose which database
         ↓
Step 3: Create/Use Collection → Choose which collection
         ↓
Step 4: Perform Operations → CRUD (insert, find, update, delete)
```

---

### Visual: The 4 Steps

```mermaid
graph TD
    A[1. MongoClient.connect] -->|Connection| B[MongoDB Server]
    B --> C[2. client.db]
    C -->|Select Database| D[Database: nodeDB]
    D --> E[3. db.collection]
    E -->|Select Collection| F[Collection: users]
    F --> G[4. CRUD Operations]
    G --> H[insertOne, find, etc.]

    style A fill:#e1f5fe
    style D fill:#fff9c4
    style F fill:#c8e6c9
    style H fill:#ffccbc
```

---

## 8. Step 1 — Creating a Connection

### Connecting to MongoDB

```javascript
import { MongoClient } from "mongodb";

async function connectDB() {
  // Step 1: Create connection
  let client = await MongoClient.connect("mongodb://localhost:27017");
  console.log("Database connected");
}

connectDB();
```

---

### Connection String Breakdown

```
mongodb://localhost:27017
└──┬───┘ └────┬────┘ └─┬─┘
Protocol   Host     Port

Protocol: mongodb:// (MongoDB protocol)
Host: localhost (your computer)
Port: 27017 (default MongoDB port)
```

---

### What Happens?

```
MongoClient.connect("mongodb://localhost:27017")
       ↓
[Tries to connect to MongoDB server]
       ↓
Success? → Returns client object
       ↓
client object → Used for all database operations
```

---

### Error Handling

```javascript
async function connectDB() {
  try {
    let client = await MongoClient.connect("mongodb://localhost:27017");
    console.log("✓ Database connected");
    return client;
  } catch (error) {
    console.error("✗ Connection failed:", error.message);
    // Common errors:
    // - MongoDB not running
    // - Wrong connection string
    // - Network issues
  }
}
```

---

### Remote Connection (MongoDB Atlas)

```javascript
// Local MongoDB:
"mongodb://localhost:27017";

// Remote MongoDB (MongoDB Atlas):
"mongodb+srv://username:password@cluster.mongodb.net/dbname";
```

---

## 9. Step 2 — Creating/Using a Database

### Creating/Accessing a Database

```javascript
async function connectDB() {
  // Step 1: Connection
  let client = await MongoClient.connect("mongodb://localhost:27017");

  // Step 2: Create/use database
  let database = client.db("nodeDB");
  console.log("Database created");
}
```

---

### How db() Works

```javascript
let database = client.db("nodeDB");

// What happens:
// If "nodeDB" exists → Use it
// If "nodeDB" doesn't exist → Create it (when first data is inserted)
```

**MongoDB creates databases lazily** — only when you insert data!

---

### Lazy Creation Example

```javascript
let database = client.db("nonExistentDB");
// Database doesn't exist yet

await database.collection("users").insertOne({ name: "Alice" });
// NOW the database is created!
```

---

## 10. Step 3 — Creating/Using a Collection

### Two Methods to Get Collection

**Method A: createCollection() (Explicit)**

```javascript
// Creates collection explicitly
let collection = await database.createCollection("nodeUsers");
console.log("Collection created");

// Returns: Promise<Collection>
// Creates the collection immediately
```

**Method B: collection() (Implicit - Preferred)**

```javascript
// Gets reference to collection (creates if doesn't exist)
let collection = database.collection("col2");
console.log("Collection reference obtained");

// Returns: Collection (not a Promise)
// Creates collection when first document inserted
```

---

### Comparison: createCollection vs collection

| Feature                 | createCollection()          | collection()              |
| ----------------------- | --------------------------- | ------------------------- |
| **Returns**             | Promise<Collection>         | Collection (sync)         |
| **Needs await**         | ✅ Yes                      | ❌ No                     |
| **Creates immediately** | ✅ Yes                      | ❌ Lazy (on first insert) |
| **Use when**            | Need to create with options | Most common cases         |

---

### Example with Both Methods

```javascript
// Method A: Explicit creation
let collection1 = await database.createCollection("users", {
  validator: {
    $jsonSchema: {
      required: ["name", "email"],
    },
  },
});

// Method B: Implicit (preferred)
let collection2 = database.collection("products");
// Doesn't create until first insert
```

---

### Which Should You Use?

**Use `collection()` (Method B) most of the time:**

- Simpler (no await needed)
- Faster (no extra database call)
- Standard practice

**Use `createCollection()` (Method A) only when:**

- Need to set validators
- Need to set collection options
- Want explicit creation

---

## 11. Step 4 — CRUD Operations

### insertOne — Insert Single Document

```javascript
let collection = database.collection("users");

let result = await collection.insertOne({
    name: "Alice",
    age: 30
});

console.log(result);
// Output:
{
    acknowledged: true,
    insertedId: ObjectId("507f1f77bcf86cd799439011")
}
```

---

### insertMany — Insert Multiple Documents

```javascript
let results = await collection.insertMany([
    { productName: "laptop", qty: 3 },
    { productName: "phone", qty: 2 }
]);

console.log(results);
// Output:
{
    acknowledged: true,
    insertedCount: 2,
    insertedIds: {
        '0': ObjectId("..."),
        '1': ObjectId("...")
    }
}
```

---

### findOne — Find Single Document

```javascript
// Find any one document
let result = await collection.findOne();
console.log(result);
// { _id: ObjectId("..."), name: "Alice", age: 30 }

// Find specific document
let user = await collection.findOne({ name: "Alice" });
console.log(user);
// { _id: ObjectId("..."), name: "Alice", age: 30 }

// If not found
let notFound = await collection.findOne({ name: "NonExistent" });
console.log(notFound); // null
```

---

### find — Find Multiple Documents

```javascript
// Find all documents (returns cursor)
let cursor = collection.find();
let allDocs = await cursor.toArray();
console.log(allDocs);
// [{ ... }, { ... }, { ... }]

// Find with filter
let results = await collection.find({ qty: { $gt: 2 } }).toArray();
console.log(results);
// [{ productName: "laptop", qty: 3 }]
```

**Important:** `find()` returns a **cursor**, not the actual data!

---

### updateOne — Update Single Document

```javascript
let result = await collection.updateOne(
  { name: "Alice" }, // Filter
  { $set: { age: 31 } }, // Update
);

console.log(result);
// {
//     acknowledged: true,
//     matchedCount: 1,
//     modifiedCount: 1
// }
```

---

### updateMany — Update Multiple Documents

```javascript
let result = await collection.updateMany(
  { qty: { $lt: 5 } }, // Filter
  { $inc: { qty: 10 } }, // Update
);

console.log(result);
// {
//     acknowledged: true,
//     matchedCount: 2,
//     modifiedCount: 2
// }
```

---

### deleteOne — Delete Single Document

```javascript
let result = await collection.deleteOne({ name: "Alice" });

console.log(result);
// {
//     acknowledged: true,
//     deletedCount: 1
// }
```

---

### deleteMany — Delete Multiple Documents

```javascript
let result = await collection.deleteMany({ qty: { $lt: 3 } });

console.log(result);
// {
//     acknowledged: true,
//     deletedCount: 2
// }
```

---

## 12. Cursors and toArray()

### What is a Cursor?

**A cursor is a pointer to the result set** — it doesn't load all data at once.

Think of it like:

- **Netflix queue** — doesn't load all episodes at once
- **YouTube autoplay** — loads videos as needed
- **Lazy loading** — efficient memory usage

---

### find() Returns a Cursor

```javascript
let cursor = collection.find();
console.log(cursor);
// FindCursor { ... } (NOT the actual documents!)

// To get documents:
let documents = await cursor.toArray();
console.log(documents);
// [{ ... }, { ... }, { ... }] (actual documents)
```

---

### Why Cursors?

**Problem without cursors:**

```javascript
// If collection has 1 million documents:
let allDocs = await collection.findWithoutCursor();
// Loads 1 million documents into memory → CRASH!
```

**Solution with cursors:**

```javascript
// Cursor loads documents in batches (lazy loading)
let cursor = collection.find();
// Memory usage: ~1 MB (just the cursor)

for await (let doc of cursor) {
  console.log(doc); // Process one at a time
}
// Memory usage stays low!
```

---

### Working with Cursors

**Method 1: toArray() (Simple)**

```javascript
let docs = await collection.find().toArray();
// Converts cursor to array
// Good for: Small result sets
```

**Method 2: Async Iteration (Efficient)**

```javascript
let cursor = collection.find();

for await (let doc of cursor) {
  console.log(doc); // Process one document at a time
}
// Good for: Large result sets
```

**Method 3: forEach() (Callback)**

```javascript
await collection.find().forEach((doc) => {
  console.log(doc);
});
// Alternative to for-await
```

---

### Cursor Methods

```javascript
let cursor = collection.find({ qty: { $gt: 2 } });

// Convert to array
let array = await cursor.toArray();

// Count documents
let count = await cursor.count();

// Sort
cursor.sort({ qty: -1 }); // Descending

// Limit
cursor.limit(10); // Only 10 documents

// Skip
cursor.skip(5); // Skip first 5

// Chain methods
let results = await collection
  .find({ qty: { $gt: 2 } })
  .sort({ qty: -1 })
  .limit(10)
  .toArray();
```

---

### Visual: Cursor vs toArray()

```
Collection (1000 documents):
[Doc1, Doc2, Doc3, ..., Doc1000]

find() → Returns Cursor:
────────────────────────────────
Cursor → [Pointer to results]
Memory: 1 MB
Loads: On-demand

toArray() → Returns Array:
────────────────────────────────
Array → [Doc1, Doc2, ..., Doc1000]
Memory: 100 MB (all documents loaded)
Loads: Immediately
```

---

## 13. Complete Working Example

### Full CRUD Application

```javascript
import { MongoClient } from "mongodb";

async function main() {
  let client;

  try {
    // Step 1: Connect
    console.log("Connecting to database...");
    client = await MongoClient.connect("mongodb://localhost:27017");
    console.log("✓ Connected");

    // Step 2: Database
    let db = client.db("nodeDB");
    console.log("✓ Using database: nodeDB");

    // Step 3: Collection
    let users = db.collection("users");
    console.log("✓ Using collection: users");

    // Step 4: CRUD Operations

    // CREATE
    console.log("\n--- INSERT ---");
    let insertResult = await users.insertOne({
      name: "Alice Johnson",
      email: "alice@example.com",
      age: 30,
      city: "New York",
    });
    console.log("Inserted:", insertResult.insertedId);

    // INSERT MANY
    let insertManyResult = await users.insertMany([
      { name: "Bob Smith", email: "bob@example.com", age: 25 },
      { name: "Charlie Brown", email: "charlie@example.com", age: 35 },
    ]);
    console.log("Inserted count:", insertManyResult.insertedCount);

    // READ
    console.log("\n--- FIND ---");
    let allUsers = await users.find().toArray();
    console.log("All users:", allUsers.length);

    let alice = await users.findOne({ name: "Alice Johnson" });
    console.log("Found:", alice.name);

    let youngUsers = await users.find({ age: { $lt: 30 } }).toArray();
    console.log("Young users:", youngUsers.length);

    // UPDATE
    console.log("\n--- UPDATE ---");
    let updateResult = await users.updateOne(
      { name: "Alice Johnson" },
      { $set: { age: 31, city: "Boston" } },
    );
    console.log("Modified:", updateResult.modifiedCount);

    let updateManyResult = await users.updateMany(
      { age: { $lt: 30 } },
      { $inc: { age: 1 } },
    );
    console.log("Modified many:", updateManyResult.modifiedCount);

    // DELETE
    console.log("\n--- DELETE ---");
    let deleteResult = await users.deleteOne({ name: "Charlie Brown" });
    console.log("Deleted:", deleteResult.deletedCount);

    // FINAL COUNT
    let finalCount = await users.countDocuments();
    console.log("\nFinal user count:", finalCount);
  } catch (error) {
    console.error("Error:", error.message);
  } finally {
    // Always close connection
    if (client) {
      await client.close();
      console.log("\n✓ Connection closed");
    }
  }
}

main();
```

---

### Expected Output

```
Connecting to database...
✓ Connected
✓ Using database: nodeDB
✓ Using collection: users

--- INSERT ---
Inserted: 507f1f77bcf86cd799439011
Inserted count: 2

--- FIND ---
All users: 3
Found: Alice Johnson
Young users: 1

--- UPDATE ---
Modified: 1
Modified many: 1

--- DELETE ---
Deleted: 1

Final user count: 2

✓ Connection closed
```

---

## 14. Best Practices

### Always Close Connections

```javascript
// ✅ GOOD — Close connection
async function query() {
  let client = await MongoClient.connect(url);
  try {
    // Do operations
  } finally {
    await client.close(); // Always close!
  }
}

// ❌ BAD — Connection leak
async function query() {
  let client = await MongoClient.connect(url);
  // Do operations
  // No close() → Memory leak!
}
```

---

### Use Environment Variables

```javascript
// ❌ BAD — Hardcoded
const url = "mongodb://localhost:27017";

// ✅ GOOD — Environment variable
import dotenv from "dotenv";
dotenv.config();

const url = process.env.MONGODB_URL;
```

**Create `.env` file:**

```
MONGODB_URL=mongodb://localhost:27017
```

---

### Error Handling

```javascript
// ✅ GOOD — Proper error handling
async function safeQuery() {
  let client;
  try {
    client = await MongoClient.connect(url);
    let db = client.db("mydb");
    let collection = db.collection("users");
    return await collection.find().toArray();
  } catch (error) {
    console.error("Database error:", error.message);
    throw error;
  } finally {
    if (client) await client.close();
  }
}
```

---

### Reuse Connection

```javascript
// ❌ BAD — Connect every time
async function getUser(id) {
  let client = await MongoClient.connect(url); // New connection
  let db = client.db("mydb");
  let user = await db.collection("users").findOne({ _id: id });
  await client.close();
  return user;
}

// ✅ GOOD — Reuse connection
let client;

async function connect() {
  if (!client) {
    client = await MongoClient.connect(url);
  }
  return client;
}

async function getUser(id) {
  let client = await connect(); // Reuse existing
  let db = client.db("mydb");
  return await db.collection("users").findOne({ _id: id });
}
```

---

## 15. Summary — Key Takeaways

### 🎯 Setup Workflow

```
1. npm init -y                  → Create package.json
2. npm install mongodb          → Install driver
3. Import MongoClient           → Ready to code
```

---

### 📦 The Three Files

```
package.json       → Lists dependencies (commit to Git)
package-lock.json  → Locks versions (commit to Git)
node_modules/      → Contains code (ignore in Git)
```

---

### 🔌 The 4 Connection Steps

```javascript
// 1. Connection
let client = await MongoClient.connect("mongodb://localhost:27017");

// 2. Database
let db = client.db("nodeDB");

// 3. Collection
let collection = db.collection("users");

// 4. Operations
await collection.insertOne({ name: "Alice" });
```

---

### 📊 CRUD Operations

```javascript
// Create
insertOne({});
insertMany([{}, {}]);

// Read
findOne({});
find({}).toArray();

// Update
updateOne({}, { $set: {} });
updateMany({}, { $set: {} });

// Delete
deleteOne({});
deleteMany({});
```

---

### 🔄 Cursor vs toArray()

```
find() → Returns Cursor (pointer)
toArray() → Converts to Array (loads all)

Use cursor for: Large datasets
Use toArray() for: Small datasets
```

---

## 16. Revision Checklist

### Setup

- [ ] Can you create package.json with npm init?
- [ ] Can you install packages with npm install?
- [ ] Do you know the three changes after install?
- [ ] Do you know to ignore node_modules in Git?

### Connection

- [ ] Can you import MongoClient?
- [ ] Can you connect to MongoDB?
- [ ] Do you know the connection string format?
- [ ] Can you create/use a database?
- [ ] Can you create/use a collection?

### CRUD

- [ ] Can you insert one document?
- [ ] Can you insert many documents?
- [ ] Can you find one document?
- [ ] Can you find many documents?
- [ ] Can you update documents?
- [ ] Can you delete documents?

### Cursors

- [ ] Do you know what a cursor is?
- [ ] Can you convert cursor to array?
- [ ] Do you know when to use toArray()?
- [ ] Can you use async iteration?

### Best Practices

- [ ] Do you always close connections?
- [ ] Do you use environment variables?
- [ ] Do you handle errors properly?
- [ ] Do you reuse connections?

---

> **🎤 Interview Tip — "How would you connect MongoDB to a Node.js application and perform a query?"**
>
> **Answer like this:**
>
> _"First, I'd set up the project with npm init -y to create package.json, then install the MongoDB driver with npm install mongodb._
>
> _To connect and query, I follow four steps:_
>
> ```javascript
> import { MongoClient } from "mongodb";
>
> async function query() {
>   let client;
>   try {
>     // Step 1: Connect
>     client = await MongoClient.connect("mongodb://localhost:27017");
>
>     // Step 2: Select database
>     let db = client.db("myDatabase");
>
>     // Step 3: Get collection
>     let collection = db.collection("users");
>
>     // Step 4: Query
>     let users = await collection.find({ age: { $gt: 25 } }).toArray();
>     return users;
>   } finally {
>     // Always close connection
>     if (client) await client.close();
>   }
> }
> ```
>
> _Key points: I use async/await for cleaner code, include proper error handling with try-finally, always close the connection to prevent leaks, and use toArray() to convert the cursor to an actual array of documents. In production, I'd also use environment variables for the connection string and implement connection pooling for better performance."_
>
> **Why this works:** Shows complete workflow, includes error handling, demonstrates understanding of cursors, mentions production considerations, and provides working code.
