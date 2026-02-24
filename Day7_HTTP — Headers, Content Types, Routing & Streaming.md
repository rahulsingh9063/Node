# Node.js HTTP — Headers, Content Types, Routing & Streaming

### Complete Guide to Building Web Servers

---

## 📚 What You'll Learn

This guide covers **practical HTTP server development**:

✅ Setting headers (2 methods)  
✅ All content types (HTML, CSS, JSON, video)  
✅ Auto-restart with `--watch` flag  
✅ Serving files (HTML, CSS, JSON)  
✅ SSR vs CSR rendering strategies  
✅ Routing — handling multiple endpoints  
✅ Streaming responses with pipe()  
✅ CommonJS vs ES Modules blocking behavior

**Best for:** Building real servers, serving static files, API development, interview prep

---

## Table of Contents

1. [Setting Headers — The Two Methods](#1-setting-headers--the-two-methods)
2. [Content-Type — Why It Matters](#2-content-type--why-it-matters)
3. [All Content-Type Values](#3-all-content-type-values)
4. [Auto-Restart with --watch](#4-auto-restart-with---watch)
5. [Serving HTML Files](#5-serving-html-files)
6. [Serving CSS Files](#6-serving-css-files)
7. [Serving JSON Data](#7-serving-json-data)
8. [SSR vs CSR — Rendering Strategies](#8-ssr-vs-csr--rendering-strategies)
9. [Routing — Multiple Endpoints](#9-routing--multiple-endpoints)
10. [Streaming with pipe()](#10-streaming-with-pipe)
11. [CommonJS vs ES Modules — Blocking](#11-commonjs-vs-es-modules--blocking)
12. [Complete Server Examples](#12-complete-server-examples)
13. [Best Practices](#13-best-practices)
14. [Summary](#14-summary)
15. [Revision Checklist](#15-revision-checklist)

---

## 1. Setting Headers — The Two Methods

### Why Set Headers?

**Headers contain metadata** about your response — they tell the browser:

- What type of data you're sending (HTML? JSON? CSS?)
- Status of the request (200 OK? 404 Not Found?)
- Caching rules, cookies, authentication info, etc.

Think of headers like:

- **Package labels** (fragile, contents: electronics)
- **Envelope markings** (priority mail, do not bend)
- **Instructions** for the browser on how to handle the data

---

### Method 1: Separate Calls (Verbose)

**Set status code and headers separately:**

```javascript
import { createServer } from "node:http";

createServer((req, res) => {
  // Set status code
  res.statusCode = 201;

  // Set headers one by one
  res.setHeader("Content-Type", "text/plain");
  res.setHeader("My-Name", "abc");
  res.setHeader("X-Custom-Header", "value");

  res.end("Response sent!");
}).listen(9000);
```

**When to use:**

- When you want to set headers conditionally
- When you're setting many headers
- When you need to modify headers after setting them

---

### Method 2: writeHead() (Clean, One Call)

**Set status code and all headers in one call:**

```javascript
import { createServer } from "node:http";

createServer((req, res) => {
  // Set everything at once
  res.writeHead(200, {
    "Content-Type": "text/plain",
    "My-Name": "abc",
    "X-Custom-Header": "value",
  });

  res.end("Response sent!");
}).listen(9000);
```

**When to use:**

- When you know all headers upfront
- For cleaner, more concise code
- Most common in production

---

### Side-by-Side Comparison

```javascript
// Method 1: Separate (4 lines)
res.statusCode = 200;
res.setHeader("Content-Type", "text/html");
res.setHeader("Cache-Control", "no-cache");
res.end("Response");

// Method 2: writeHead (2 lines)
res.writeHead(200, {
  "Content-Type": "text/html",
  "Cache-Control": "no-cache",
});
res.end("Response");
```

**Result:** Exactly the same! Choose based on preference.

---

### ⚠️ CRITICAL Rule

**ALWAYS set headers BEFORE sending response:**

```javascript
// ✅ CORRECT — Headers first
res.writeHead(200, { "Content-Type": "text/html" });
res.end("<h1>Hello</h1>");

// ❌ WRONG — Headers after response
res.end("<h1>Hello</h1>");
res.writeHead(200, { "Content-Type": "text/html" }); // ERROR!
```

**Why?** Once you send data with `res.end()`, the connection is closed — you can't modify headers anymore!

---

### Real-World Example: API Response

```javascript
createServer((req, res) => {
  // Successful response
  res.writeHead(200, {
    "Content-Type": "application/json",
    "X-API-Version": "1.0",
    "Cache-Control": "no-cache",
    "Access-Control-Allow-Origin": "*", // CORS
  });

  res.end(
    JSON.stringify({
      success: true,
      data: { message: "Hello World" },
    }),
  );
}).listen(3000);
```

---

## 2. Content-Type — Why It Matters

### What is Content-Type?

**Content-Type tells the browser what kind of data you're sending**, so it knows how to display it.

---

### Without Content-Type (Browser Confused)

```javascript
createServer((req, res) => {
  // Missing Content-Type!
  res.end("<h1>Hello World</h1>");
}).listen(3000);
```

**What happens:**

- Browser sees: `<h1>Hello World</h1>` (as plain text)
- Browser thinks: "Is this text? HTML? JSON? 🤔"
- Browser displays: `<h1>Hello World</h1>` (raw text, not rendered HTML!)

---

### With Content-Type (Browser Happy)

```javascript
createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/html" });
  res.end("<h1>Hello World</h1>");
}).listen(3000);
```

**What happens:**

- Browser sees: Content-Type = text/html
- Browser thinks: "Ah! This is HTML, I should render it!"
- Browser displays: **Hello World** (rendered as heading)

---

### Visual: Impact of Content-Type

```
Same response body: { "name": "John" }

Content-Type: text/plain
Browser shows: { "name": "John" }  (as plain text)

Content-Type: application/json
Browser shows: (Pretty JSON viewer with syntax highlighting)
```

---

## 3. All Content-Type Values

### Common Content Types

| Data Type         | Content-Type Value       | Use Case          |
| ----------------- | ------------------------ | ----------------- |
| **Plain text**    | `text/plain`             | Simple text, logs |
| **HTML**          | `text/html`              | Web pages         |
| **CSS**           | `text/css`               | Stylesheets       |
| **JavaScript**    | `application/javascript` | JS files          |
| **JSON**          | `application/json`       | API responses     |
| **XML**           | `application/xml`        | Data interchange  |
| **Images (JPEG)** | `image/jpeg`             | Photos            |
| **Images (PNG)**  | `image/png`              | Graphics          |
| **Images (GIF)**  | `image/gif`              | Animations        |
| **Videos (MP4)**  | `video/mp4`              | Video files       |
| **Audio (MP3)**   | `audio/mpeg`             | Audio files       |
| **PDF**           | `application/pdf`        | Documents         |
| **ZIP**           | `application/zip`        | Compressed files  |

---

### Text-Based Content Types

```javascript
// 1. Plain text
res.writeHead(200, { "Content-Type": "text/plain" });
res.end("This is plain text");

// 2. HTML
res.writeHead(200, { "Content-Type": "text/html" });
res.end("<h1>This is HTML</h1>");

// 3. CSS
res.writeHead(200, { "Content-Type": "text/css" });
res.end("body { color: red; }");

// 4. JavaScript
res.writeHead(200, { "Content-Type": "application/javascript" });
res.end("console.log('Hello');");
```

---

### Data Formats

```javascript
// 1. JSON
res.writeHead(200, { "Content-Type": "application/json" });
res.end(JSON.stringify({ name: "John", age: 30 }));

// 2. XML
res.writeHead(200, { "Content-Type": "application/xml" });
res.end("<user><name>John</name></user>");
```

---

### Media Files

```javascript
// 1. Images
res.writeHead(200, { "Content-Type": "image/jpeg" });
res.end(imageBuffer); // Binary data

// 2. Videos (Note: Your code had a typo!)
res.writeHead(200, { "Content-Type": "video/mp4" });
res.end(videoBuffer);

// 3. Audio
res.writeHead(200, { "Content-Type": "audio/mpeg" });
res.end(audioBuffer);
```

---

### Real-World Example: File Server

```javascript
import { createServer } from "node:http";
import fs from "node:fs";

createServer((req, res) => {
  if (req.url === "/logo.png") {
    res.writeHead(200, { "Content-Type": "image/png" });
    fs.createReadStream("./logo.png").pipe(res);
  } else if (req.url === "/video.mp4") {
    res.writeHead(200, { "Content-Type": "video/mp4" });
    fs.createReadStream("./video.mp4").pipe(res);
  } else if (req.url === "/data.json") {
    res.writeHead(200, { "Content-Type": "application/json" });
    fs.createReadStream("./data.json").pipe(res);
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("Not Found");
  }
}).listen(3000);
```

---

## 4. Auto-Restart with --watch

### The Problem

**Every time you change your server code, you must manually restart:**

```bash
$ node server.js
Server running...

# Make changes to server.js
# Changes NOT reflected!

# Must stop server (Ctrl+C)
^C

# Restart server
$ node server.js
Server running...
# Now changes are visible
```

**This is tedious!** Restart 100+ times during development.

---

### The Solution — --watch Flag

**Node.js 18.11+ has a built-in watch mode:**

```bash
$ node --watch server.js
Server running...

# Make changes to server.js
# File changed, restarting...
# Server automatically restarts!
# Changes are now visible!
```

**Benefits:**

- ✅ No manual restarts
- ✅ Fast development
- ✅ No external tools needed
- ✅ Built into Node.js

---

### How It Works

```bash
# Start with watch mode
$ node --watch server.js
(node:12345) ExperimentalWarning: Watch mode is an experimental feature
Server running at port 3000

# Edit server.js (change "Hello" to "Hi")
# Output:
Completed running 'server.js'
# Automatically restarts!
Server running at port 3000
```

**Every time you save the file, Node restarts the server automatically!**

---

### Comparison: Manual vs --watch

```
Manual Restart (100 changes):
  Change code → Ctrl+C → node server.js → Test
  Change code → Ctrl+C → node server.js → Test
  Change code → Ctrl+C → node server.js → Test
  ... (97 more times)

  Total: 300 actions! 😫

With --watch (100 changes):
  $ node --watch server.js
  Change code → Save → Test
  Change code → Save → Test
  Change code → Save → Test
  ... (97 more times)

  Total: 100 actions! 😊
```

**3x fewer actions = 3x faster development!**

---

### Alternative: nodemon

**Before Node 18.11, we used nodemon (still popular):**

```bash
# Install globally
$ npm install -g nodemon

# Run with nodemon
$ nodemon server.js
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] starting `node server.js`
Server running...
```

**Comparison:**

| Feature          | --watch         | nodemon                        |
| ---------------- | --------------- | ------------------------------ |
| **Installation** | None (built-in) | npm install                    |
| **Speed**        | Fast            | Fast                           |
| **Features**     | Basic           | Advanced (delay, ignore, etc.) |
| **Best for**     | Simple projects | Professional projects          |

---

## 5. Serving HTML Files

### Basic HTML Response

```javascript
import { createServer } from "node:http";
import fs from "node:fs";

createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/html" });

  // Read HTML file
  let htmlData = fs.readFileSync("./index.html", "utf-8");

  res.end(htmlData);
}).listen(3000);
```

**index.html:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Server</title>
  </head>
  <body>
    <h1>Welcome to My Website</h1>
    <p>This page is served from Node.js!</p>
  </body>
</html>
```

**Access:** `http://localhost:3000` → Renders the HTML page

---

### Async HTML Response (Better!)

```javascript
createServer((req, res) => {
  fs.readFile("./index.html", "utf-8", (err, data) => {
    if (err) {
      res.writeHead(500, { "Content-Type": "text/plain" });
      return res.end("Error reading file");
    }

    res.writeHead(200, { "Content-Type": "text/html" });
    res.end(data);
  });
}).listen(3000);
```

**Why better?** Non-blocking — server can handle other requests while reading file.

---

### Error Handling

```javascript
createServer((req, res) => {
  fs.readFile("./index.html", "utf-8", (err, data) => {
    if (err) {
      // File not found
      if (err.code === "ENOENT") {
        res.writeHead(404, { "Content-Type": "text/html" });
        return res.end("<h1>404 - Page Not Found</h1>");
      }

      // Other errors
      res.writeHead(500, { "Content-Type": "text/plain" });
      return res.end("Internal Server Error");
    }

    res.writeHead(200, { "Content-Type": "text/html" });
    res.end(data);
  });
}).listen(3000);
```

---

## 6. Serving CSS Files

### Basic CSS Response

```javascript
createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/css" });

  let cssData = fs.readFileSync("./style.css", "utf-8");

  res.end(cssData);
}).listen(3000);
```

**style.css:**

```css
body {
  font-family: Arial, sans-serif;
  background-color: #f0f0f0;
  color: #333;
}

h1 {
  color: #007bff;
  text-align: center;
}
```

---

### Linking CSS to HTML

**Problem:** HTML and CSS on different routes

**Solution:** Link them together

**HTML file (index.html):**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Site</title>
    <link rel="stylesheet" href="/styles.css" />
  </head>
  <body>
    <h1>Styled Page</h1>
  </body>
</html>
```

**Server with routing:**

```javascript
createServer((req, res) => {
  if (req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/html" });
    fs.createReadStream("./index.html").pipe(res);
  } else if (req.url === "/styles.css") {
    res.writeHead(200, { "Content-Type": "text/css" });
    fs.createReadStream("./style.css").pipe(res);
  } else {
    res.writeHead(404);
    res.end("Not Found");
  }
}).listen(3000);
```

**Flow:**

1. Browser requests `/` → Gets HTML
2. HTML says: "I need `/styles.css`"
3. Browser requests `/styles.css` → Gets CSS
4. Browser applies styles to HTML

---

## 7. Serving JSON Data

### Basic JSON Response

```javascript
createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });

  let jsonResponse = fs.readFileSync("./data.json", "utf-8");

  res.end(jsonResponse);
}).listen(3000);
```

**data.json:**

```json
{
  "users": [
    { "id": 1, "name": "Alice", "email": "alice@example.com" },
    { "id": 2, "name": "Bob", "email": "bob@example.com" },
    { "id": 3, "name": "Charlie", "email": "charlie@example.com" }
  ]
}
```

**Access:** `http://localhost:3000` → Returns JSON data

---

### Creating JSON from JavaScript

```javascript
createServer((req, res) => {
  // Simulate database query
  const users = [
    { id: 1, name: "Alice", email: "alice@example.com" },
    { id: 2, name: "Bob", email: "bob@example.com" },
  ];

  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify(users));
}).listen(3000);
```

---

## 8. SSR vs CSR — Rendering Strategies

### What is Rendering?

**Rendering = Converting code (HTML/CSS/JS) into a visual page**

Think of it like:

- **Recipe → Cooked Meal** (code → visual page)
- **Blueprint → Built House** (HTML/CSS → webpage)

---

### Server-Side Rendering (SSR) — Legacy

**Server does ALL the work:**

```
User clicks link
    ↓
Request sent to server
    ↓
Server:
  - Fetches data from database
  - Generates HTML
  - Applies CSS styles
  - Runs JavaScript
  - Renders complete page
    ↓
Server sends FULLY RENDERED HTML
    ↓
Browser displays it immediately
```

**Pros:**

- ✅ Fast initial page load
- ✅ Good for SEO (search engines see content)
- ✅ Works without JavaScript

**Cons:**

- ❌ Heavy server load
- ❌ Slow subsequent navigation (full page reload)
- ❌ Wasted bandwidth (sends everything every time)

**Example (Traditional PHP, Ruby on Rails):**

```php
// server.php
<?php
  $users = getUsers(); // Database call
  echo "<html><body>";
  foreach($users as $user) {
    echo "<p>" . $user['name'] . "</p>";
  }
  echo "</body></html>";
?>
```

---

### Client-Side Rendering (CSR) — Modern

**Browser does the work:**

```
User clicks link
    ↓
Request sent to server
    ↓
Server sends:
  - Minimal HTML (just a div)
  - CSS file
  - JavaScript bundle
    ↓
Browser:
  - Receives files
  - Executes JavaScript
  - Fetches data from API
  - Renders page dynamically
    ↓
Browser displays page
```

**Pros:**

- ✅ Light server load
- ✅ Fast subsequent navigation (no reload)
- ✅ Rich interactivity
- ✅ Better user experience (SPA)

**Cons:**

- ❌ Slow initial load (must download JS)
- ❌ Bad for SEO (search engines see empty page)
- ❌ Requires JavaScript enabled

**Example (React, Vue, Angular):**

**Server sends minimal HTML:**

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="root"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```

**JavaScript renders everything:**

```javascript
// bundle.js
fetch("/api/users")
  .then((res) => res.json())
  .then((users) => {
    const html = users.map((u) => `<p>${u.name}</p>`).join("");
    document.getElementById("root").innerHTML = html;
  });
```

---

### Visual Comparison

```
SSR (Server-Side Rendering):
──────────────────────────────────────────────
Server:  [████████████] Heavy work (render page)
Browser: [█] Light work (just display)
Result:  Fast initial load, slow navigation

CSR (Client-Side Rendering):
──────────────────────────────────────────────
Server:  [█] Light work (send files)
Browser: [████████████] Heavy work (render page)
Result:  Slow initial load, fast navigation
```

---

### Modern Solutions — Hybrid Approaches

**Next.js, Nuxt.js, SvelteKit:**

- ISR (Incremental Static Regeneration)
- Hydration
- SSG (Static Site Generation)

**Best of both worlds!**

---

## 9. Routing — Multiple Endpoints

### What is Routing?

**Routing = Handling different URLs/endpoints on your server**

Think of it like:

- **Restaurant menu** — different dishes for different orders
- **Building directory** — different rooms for different purposes
- **Phone menu** — "Press 1 for sales, 2 for support"

---

### URL Structure

```
https://nodejs.org/en/about
└──┬───┘ └───┬────┘ └┬┘ └──┬──┘
 Protocol   Domain   Path  Route/Endpoint

Default route: "/"
About page: "/about"
Download: "/download"
Blog: "/blog"
```

---

### Basic Routing

```javascript
import { createServer } from "node:http";

createServer((req, res) => {
  let endpoint = req.url;

  if (endpoint === "/") {
    res.end("Home Page");
  } else if (endpoint === "/about") {
    res.end("About Page");
  } else if (endpoint === "/contact") {
    res.end("Contact Page");
  } else {
    res.statusCode = 404;
    res.end("404 - Page Not Found");
  }
}).listen(3000);
```

**Try:**

```
http://localhost:3000/         → "Home Page"
http://localhost:3000/about    → "About Page"
http://localhost:3000/contact  → "Contact Page"
http://localhost:3000/random   → "404 - Page Not Found"
```

---

### Routing with Files

```javascript
import { createReadStream } from "node:fs";
import { createServer } from "node:http";

createServer((req, res) => {
  let endpoint = req.url;

  // Route 1: HTML page
  if (endpoint === "/html" || endpoint === "/") {
    res.writeHead(200, { "Content-Type": "text/html" });
    let data = createReadStream("./index.html", "utf-8");
    data.pipe(res);
  }

  // Route 2: CSS file
  else if (endpoint === "/css") {
    res.writeHead(200, { "Content-Type": "text/css" });
    let data = createReadStream("./style.css", "utf-8");
    data.pipe(res);
  }

  // Route 3: JSON data
  else if (endpoint === "/json") {
    res.writeHead(200, { "Content-Type": "application/json" });
    let data = createReadStream("./data.json", "utf-8");
    data.pipe(res);
  }

  // Route 4: 404 Not Found
  else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 - Not Found");
  }
}).listen(3000);
```

---

### Default Route

**The root `/` is the default route:**

```javascript
// These are equivalent:
http://localhost:3000
http://localhost:3000/

// Both match:
if (req.url === '/') { ... }
```

---

## 10. Streaming with pipe()

### Why Streams?

**Problem without streams:**

```javascript
// ❌ Loads entire file into memory first
let htmlData = fs.readFileSync("./large-file.html", "utf-8");
res.end(htmlData);

// What if file is 1GB?
// → 1GB in RAM!
// → Server crashes!
```

**Solution with streams:**

```javascript
// ✅ Streams in chunks (64KB at a time)
let data = fs.createReadStream("./large-file.html", "utf-8");
data.pipe(res);

// Only 64KB in RAM at any time!
// Can serve files bigger than RAM!
```

---

### What is pipe()?

**pipe() connects a readable stream to a writable stream:**

```
Readable Stream (file) → pipe() → Writable Stream (response)
       ↓                              ↓
  Source (input)              Destination (output)
```

**Syntax:**

```javascript
readableStream.pipe(writableStream);
```

---

### Basic pipe() Example

```javascript
import { createReadStream } from "node:fs";
import { createServer } from "node:http";

createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/html" });

  // Create readable stream (source)
  let data = createReadStream("./index.html", "utf-8");

  // Pipe to response (destination)
  data.pipe(res);

  // That's it! File streams directly to browser
}).listen(3000);
```

---

### Manual vs pipe()

**❌ Manual way (tedious):**

```javascript
let data = createReadStream("./index.html", "utf-8");
let body = "";

data.on("data", (chunk) => {
  body += chunk.toString();
});

data.on("end", () => {
  res.writeHead(200, { "Content-Type": "text/html" });
  res.end(body);
});

data.on("error", (err) => {
  res.writeHead(500);
  res.end("Error");
});
```

**✅ Using pipe() (clean!):**

```javascript
let data = createReadStream("./index.html", "utf-8");
data.pipe(res);

// Automatically handles:
// - Reading chunks
// - Writing to response
// - Backpressure
// - Closing streams
```

---

### Visual: How pipe() Works

```
File: index.html (1MB)
       ↓
createReadStream()
       ↓
[64KB chunk 1] → pipe → res → Browser receives chunk 1
       ↓
[64KB chunk 2] → pipe → res → Browser receives chunk 2
       ↓
[64KB chunk 3] → pipe → res → Browser receives chunk 3
       ↓
... (16 chunks total for 1MB)
       ↓
Stream ends → Close connection
```

**Browser starts displaying content before entire file is loaded!**

---

### Complete Server with Streaming

```javascript
import { createReadStream } from "node:fs";
import { createServer } from "node:http";

createServer((req, res) => {
  if (req.url === "/html" || req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/html" });
    createReadStream("./index.html", "utf-8").pipe(res);
  } else if (req.url === "/css") {
    res.writeHead(200, { "Content-Type": "text/css" });
    createReadStream("./style.css", "utf-8").pipe(res);
  } else if (req.url === "/json") {
    res.writeHead(200, { "Content-Type": "application/json" });
    createReadStream("./data.json", "utf-8").pipe(res);
  } else if (req.url === "/video") {
    res.writeHead(200, { "Content-Type": "video/mp4" });
    createReadStream("./video.mp4").pipe(res);
    // Streams video in chunks — Netflix-style!
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("Not Found");
  }
}).listen(3000);
```

---

## 11. CommonJS vs ES Modules — Blocking

### The Key Difference

**Your comment in the code:**

```javascript
//~ commonJS blocks the code while importing whereas ESM does not block the code
```

**Let's understand this deeply:**

---

### CommonJS (Synchronous/Blocking)

```javascript
// CommonJS (require)
console.log("1. Before import");

const fs = require("node:fs"); // BLOCKS here
const http = require("node:http"); // BLOCKS here

console.log("2. After import");

// Output:
// 1. Before import
// (waits for fs to load)
// (waits for http to load)
// 2. After import
```

**How it works:**

1. Runs line 1
2. Stops and loads `fs` (synchronous)
3. Stops and loads `http` (synchronous)
4. Continues to line 5

**Blocking = Waits at each require()**

---

### ES Modules (Asynchronous/Non-Blocking)

```javascript
// ES Modules (import)
console.log("1. Before import");

import fs from "node:fs"; // Doesn't block
import { createServer } from "node:http"; // Doesn't block

console.log("2. After import");

// Output:
// (Both imports happen first)
// 1. Before import
// 2. After import
```

**How it works:**

1. Scans file and loads ALL imports first (before running any code)
2. Then runs the code

**Non-blocking = All imports load in parallel before execution starts**

---

### Visual Comparison

```
CommonJS (Synchronous):
──────────────────────────────────────
Time →
[Line 1] → [require fs] WAIT → [require http] WAIT → [Line 5]
           ↑ Blocks               ↑ Blocks

ES Modules (Asynchronous):
──────────────────────────────────────
Time →
[Load fs & http] (parallel) → [Run all code]
 ↑ No waiting!
```

---

### Real Impact

**Small projects:** Negligible difference

**Large projects with many imports:**

```javascript
// CommonJS — SLOW
const express = require("express"); // Wait 50ms
const mongoose = require("mongoose"); // Wait 100ms
const jwt = require("jsonwebtoken"); // Wait 30ms
const bcrypt = require("bcrypt"); // Wait 40ms
// Total: 220ms

// ES Modules — FAST
import express from "express"; // Load in parallel
import mongoose from "mongoose"; // Load in parallel
import jwt from "jsonwebtoken"; // Load in parallel
import bcrypt from "bcrypt"; // Load in parallel
// Total: ~100ms (parallel loading)
```

**ES Modules can be 2x faster for large applications!**

---

## 12. Complete Server Examples

### Example 1: Simple Static Server

```javascript
import { createReadStream } from "node:fs";
import { createServer } from "node:http";

createServer((req, res) => {
  const { url } = req;

  if (url === "/" || url === "/index") {
    res.writeHead(200, { "Content-Type": "text/html" });
    createReadStream("./public/index.html").pipe(res);
  } else if (url === "/styles.css") {
    res.writeHead(200, { "Content-Type": "text/css" });
    createReadStream("./public/styles.css").pipe(res);
  } else if (url === "/script.js") {
    res.writeHead(200, { "Content-Type": "application/javascript" });
    createReadStream("./public/script.js").pipe(res);
  } else {
    res.writeHead(404, { "Content-Type": "text/html" });
    res.end("<h1>404 - Page Not Found</h1>");
  }
}).listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});
```

---

### Example 2: REST API Server

```javascript
import { createServer } from "node:http";
import fs from "node:fs";

createServer((req, res) => {
  const { url, method } = req;

  // GET /api/users
  if (url === "/api/users" && method === "GET") {
    res.writeHead(200, { "Content-Type": "application/json" });
    fs.createReadStream("./data/users.json").pipe(res);
  }

  // GET /api/products
  else if (url === "/api/products" && method === "GET") {
    res.writeHead(200, { "Content-Type": "application/json" });
    fs.createReadStream("./data/products.json").pipe(res);
  }

  // GET /api/status
  else if (url === "/api/status") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(
      JSON.stringify({
        status: "online",
        uptime: process.uptime(),
        memory: process.memoryUsage(),
      }),
    );
  }

  // 404
  else {
    res.writeHead(404, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ error: "Endpoint not found" }));
  }
}).listen(3000, () => {
  console.log("API running at http://localhost:3000");
});
```

---

## 13. Best Practices

### Always Set Content-Type

```javascript
// ✅ GOOD
res.writeHead(200, { "Content-Type": "application/json" });
res.end(JSON.stringify(data));

// ❌ BAD — Browser confused
res.end(JSON.stringify(data));
```

---

### Use Streams for Large Files

```javascript
// ✅ GOOD — Streams
createReadStream("./large-video.mp4").pipe(res);

// ❌ BAD — Loads entire file into RAM
let video = fs.readFileSync("./large-video.mp4");
res.end(video);
```

---

### Handle Errors Properly

```javascript
// ✅ GOOD — Error handling
const stream = createReadStream("./file.txt");

stream.on("error", (err) => {
  res.writeHead(404, { "Content-Type": "text/plain" });
  res.end("File not found");
});

stream.pipe(res);
```

---

### Use --watch During Development

```bash
# ✅ GOOD — Auto-restart
node --watch server.js

# ❌ BAD — Manual restart every time
node server.js
```

---

### Return After Sending Response

```javascript
// ✅ GOOD — Return prevents further execution
if (req.url === "/") {
  res.end("Home");
  return; // Stop here
}

// ❌ BAD — Code after might run
if (req.url === "/") {
  res.end("Home");
  // Oops! Code below still runs
}
res.end("Other page"); // ERROR: Can't send twice!
```

---

## 14. Summary — Key Takeaways

### 🎯 Headers

```javascript
// Method 1: Separate
res.statusCode = 200;
res.setHeader("Content-Type", "text/html");

// Method 2: writeHead (better)
res.writeHead(200, { "Content-Type": "text/html" });
```

---

### 📄 Content Types

```
text/plain       → Plain text
text/html        → Web pages
text/css         → Stylesheets
application/json → API responses
video/mp4        → Videos (NOT application/video!)
```

---

### 🔄 Auto-Restart

```bash
node --watch server.js  # Built-in (Node 18+)
nodemon server.js       # External tool
```

---

### 🗺️ Routing

```javascript
if (req.url === '/') { ... }          // Home
else if (req.url === '/about') { ... } // About
else { ... }                          // 404
```

---

### 💨 Streaming

```javascript
// Use pipe() for files
createReadStream("./file.html").pipe(res);

// Don't use readFileSync for large files
```

---

### 🏗️ SSR vs CSR

```
SSR → Server renders, fast initial load
CSR → Browser renders, fast navigation
```

---

## 15. Revision Checklist

### Headers

- [ ] Can you set headers with res.statusCode + res.setHeader()?
- [ ] Can you set headers with res.writeHead()?
- [ ] Do you know headers must come before res.end()?
- [ ] Can you set multiple headers?

### Content Types

- [ ] Do you know text/plain vs text/html?
- [ ] Can you list 5 content types?
- [ ] Do you know why Content-Type matters?
- [ ] Do you know the correct video content type?

### File Serving

- [ ] Can you serve HTML files?
- [ ] Can you serve CSS files?
- [ ] Can you serve JSON files?
- [ ] Can you handle file not found errors?

### Routing

- [ ] Can you route different URLs?
- [ ] Do you know "/" is the default route?
- [ ] Can you handle 404 errors?
- [ ] Can you route based on HTTP method?

### Streaming

- [ ] Can you use pipe()?
- [ ] Do you know why streams are better?
- [ ] Can you stream large files?
- [ ] Do you know the alternative manual approach?

### Development

- [ ] Do you know how to use --watch?
- [ ] Can you auto-restart your server?
- [ ] Do you know about nodemon?

### Concepts

- [ ] Can you explain SSR vs CSR?
- [ ] Do you know CommonJS blocks during import?
- [ ] Do you know ES Modules don't block?

---

> **🎤 Interview Tip — "How would you build a server that serves both HTML pages and a REST API?"**
>
> **Answer like this:**
>
> _"I'd create a single server with routing that handles both static files and API endpoints. The key is using different Content-Type headers and separating concerns by URL structure._
>
> _For static files, I'd serve HTML, CSS, and JavaScript with appropriate Content-Types. For large files, I'd use streams with pipe() to avoid loading everything into memory. For the API, I'd use /api prefix for all endpoints and return JSON with application/json Content-Type._
>
> _Here's the structure:_
>
> ```javascript
> createServer((req, res) => {
>   const { url, method } = req;
>
>   // Static files
>   if (url === "/") {
>     res.writeHead(200, { "Content-Type": "text/html" });
>     createReadStream("./index.html").pipe(res);
>   }
>
>   // API endpoints
>   else if (url === "/api/users" && method === "GET") {
>     res.writeHead(200, { "Content-Type": "application/json" });
>     res.end(JSON.stringify(users));
>   }
>
>   // 404
>   else {
>     res.writeHead(404);
>     res.end("Not Found");
>   }
> }).listen(3000);
> ```
>
> _I'd also use node --watch during development for auto-restart, and implement proper error handling for missing files. In production, I'd likely use Express for better routing and middleware, but the fundamentals remain the same."_
>
> **Why this works:** Shows understanding of routing, Content-Types, streaming, error handling, and demonstrates awareness of when to use raw http vs frameworks.
