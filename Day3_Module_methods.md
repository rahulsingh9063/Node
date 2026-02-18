# Node.js fs Module & Streams

## Table of Contents

1. [Importing Built-in Modules](#1-importing-built-in-modules)
2. [What is the fs Module?](#2-what-is-the-fs-module)
3. [Synchronous vs Asynchronous Execution](#3-synchronous-vs-asynchronous-execution)
4. [Buffers & Encoding](#4-buffers--encoding)
5. [Synchronous fs Methods (Blocking)](#5-synchronous-fs-methods-blocking)
6. [Asynchronous fs Methods (Non-Blocking)](#6-asynchronous-fs-methods-non-blocking)
7. [Error-First Callbacks](#7-error-first-callbacks)
8. [Streams — Introduction](#8-streams--introduction)
9. [Why Streams Matter](#9-why-streams-matter)
10. [Streams Architecture Diagram](#10-streams-architecture-diagram)
11. [Complete Method Reference Table](#11-complete-method-reference-table)
12. [Summary](#12-summary)
13. [Revision Checklist](#13-revision-checklist)

---

## 1. Importing Built-in Modules

### ESM (ES Modules) Syntax

```javascript
// Using ESM — with "type": "module" in package.json
import fs from "node:fs";
import http from "node:http";
import os from "node:os";
import path from "node:path";
```

> **Note:** The `node:` prefix is optional but recommended — it explicitly marks the module as a built-in Node.js module (vs a third-party package).

---

### CommonJS Syntax

```javascript
// Using CommonJS (default in Node)
const fs = require("node:fs");
const http = require("node:http");
const os = require("node:os");
const path = require("node:path");
```

---

### Common Built-in Module Names

All built-in module names are **lowercase** and **separated by hyphens** if multi-word:

| Module          | Purpose                                      |
| --------------- | -------------------------------------------- |
| `fs`            | File system operations (read/write files)    |
| `http`          | Create HTTP servers, make requests           |
| `crypto`        | Hashing, encryption, random values           |
| `os`            | Get OS info (CPU, memory, platform)          |
| `path`          | Work with file/directory paths               |
| `cluster`       | Spawn multiple Node processes                |
| `events`        | EventEmitter — custom event handling         |
| `child_process` | Spawn subprocesses                           |
| `stream`        | Handle streaming data                        |
| `util`          | Utility functions (promisify, inspect, etc.) |

---

## 2. What is the fs Module?

**Definition:** `fs` stands for **File System**. It is a built-in Node.js module that provides utilities to **operate on files and folders** present in the operating system.

You can:

- Create, read, update, and delete files
- Create and delete directories (folders)
- Copy, rename, and move files
- Check file existence, permissions, size, etc.

```javascript
import fs from "node:fs";

console.log(fs); // Logs all available fs methods and properties
```

> **Why fs exists:** Browsers cannot access the file system for security reasons (imagine a malicious website deleting your files). Node.js is a server environment, so it can — and fs is the bridge.

---

## 3. Synchronous vs Asynchronous Execution

In JavaScript/Node.js, we can execute code in **two ways**:

### Synchronous (Blocking)

- Code runs **top-to-bottom**, one line at a time
- Each line **blocks** the next until it completes
- **Simple** but can freeze the entire program if a task is slow

```javascript
console.log(1);
fs.writeFileSync("./demo.txt", "some data"); // BLOCKS here until file is written
console.log("file created");
console.log(2);
console.log(3);

// Output: 1 → "file created" → 2 → 3 (in order, always)
```

---

### Asynchronous (Non-Blocking)

- Code **starts a task, moves on**, and handles the result **later via a callback**
- Does NOT block — other code continues executing
- **More complex** but essential for high-performance servers

```javascript
console.log(1);
fs.writeFile("./demo.txt", "some data", (err) => {
  // NON-BLOCKING
  if (err) console.log(err);
  console.log("file created");
});
console.log(2);
console.log(3);

// Output: 1 → 2 → 3 → "file created" (file creation happens in the background)
```

---

### When to Use Which

| Use Case                                                      | Synchronous                    | Asynchronous |
| ------------------------------------------------------------- | ------------------------------ | ------------ |
| **One-time setup scripts** (reading config at startup)        | ✅ Good                        | Optional     |
| **High-traffic servers** (handling thousands of requests/sec) | ❌ Bad (blocks other requests) | ✅ Essential |
| **CLI tools** (simple, single-user)                           | ✅ Fine                        | Optional     |
| **Real-time apps** (chat, live dashboards)                    | ❌ Very bad                    | ✅ Required  |

> **Golden rule:** In production servers, **always use async** for I/O operations (file read/write, database queries, network calls).

---

## 4. Buffers & Encoding

### What is a Buffer?

**Definition:** A **Buffer** is a **temporary storage area in RAM** that holds raw binary data. It's an array-like structure that stores data in binary format (0s and 1s) and **displays it in hexadecimal** when logged.

```javascript
let content = fs.readFileSync("./demo.txt"); // No encoding specified
console.log(content);
// <Buffer 68 65 6c 6c 6f>
//         h  e  l  l  o   (hex representation of ASCII values)
```

---

### What is Encoding?

**Definition:** Encoding is a **rule/specification** that determines how many bits each letter, number, or symbol will take, and how to convert between binary and human-readable text.

**Common encodings:**

| Encoding   | Size per character                                     | Use case                                              |
| ---------- | ------------------------------------------------------ | ----------------------------------------------------- |
| **UTF-8**  | 1-4 bytes                                              | Most common — supports all languages, emojis, symbols |
| **ASCII**  | 1 byte                                                 | English letters, numbers, basic symbols only          |
| **UTF-16** | 2-4 bytes                                              | Used by JavaScript strings internally                 |
| **base64** | Encodes binary data as text (for sending images, etc.) |

---

### Converting Buffer to String

```javascript
let content = fs.readFileSync("./demo.txt"); // Returns Buffer
console.log(content); // <Buffer 68 65 6c 6c 6f>

// Convert to human-readable string
console.log(content.toString("utf-8")); // "hello"

// OR specify encoding directly in readFileSync
let text = fs.readFileSync("./demo.txt", "utf-8");
console.log(text); // "hello" (already a string)
```

> **Best practice:** Always specify encoding (`"utf-8"`) when reading text files. Without it, you get a Buffer — which is fine for binary files (images, PDFs), but annoying for text.

---

## 5. Synchronous fs Methods (Blocking)

All synchronous methods end with **`Sync`**.

---

### 5.1 Creating a File — `writeFileSync()`

**Syntax:** `fs.writeFileSync(path, data)`

**What it does:**

- Creates a new file at the given path with the specified data
- **Overwrites** the file if it already exists (destructive!)

```javascript
console.log(1);
fs.writeFileSync("./demo.txt", "Hello World");
console.log("file created");
console.log(2);
console.log(3);

// Output: 1 → "file created" → 2 → 3
// Result: demo.txt now contains "Hello World"
```

> **Warning:** If `demo.txt` already exists, its contents will be **completely replaced** with "Hello World".

---

### 5.2 Reading a File — `readFileSync()`

**Syntax:** `fs.readFileSync(path, encoding)`

**What it does:**

- Reads the entire file at once into memory
- Returns a **Buffer** (if no encoding) or a **string** (if encoding is specified)

```javascript
console.log(1);
let content = fs.readFileSync("./demo.txt", "utf-8");
console.log("content:", content);
console.log(2);

// Output:
// 1
// content: Hello World
// 2
```

**Without encoding:**

```javascript
let buffer = fs.readFileSync("./demo.txt"); // No encoding
console.log(buffer); // <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64>
console.log(buffer.toString("utf-8")); // "Hello World"
```

---

### 5.3 Updating a File (Append) — `appendFileSync()`

**Syntax:** `fs.appendFileSync(path, newData)`

**What it does:**

- **Adds** new data to the **end** of an existing file (non-destructive)
- **Creates** the file if it doesn't exist

```javascript
try {
  fs.appendFileSync("./demo.txt", " New line added");
  console.log("done");
} catch (error) {
  console.log(error);
}

// Before: "Hello World"
// After:  "Hello World New line added"
```

> **Key difference from writeFileSync:** `appendFileSync` **adds to** the file. `writeFileSync` **replaces** the file.

---

### 5.4 Copying a File — `copyFileSync()`

**Syntax:** `fs.copyFileSync(source, destination)`

**What it does:**

- Copies the file from source to destination
- Overwrites destination if it exists

```javascript
fs.copyFileSync("./fs.js", "../../Starter/fs.txt");
console.log("done");

// fs.js content is now duplicated at ../../Starter/fs.txt
```

**Manual copy-paste example:**

```javascript
function copyPaste() {
  let contents = fs.readFileSync("./fs.js", "utf-8"); // Read source
  fs.writeFileSync("./fs.txt", contents); // Write to dest
  console.log("done");
}
copyPaste();
```

---

### 5.5 Deleting a File — `unlinkSync()`

**Syntax:** `fs.unlinkSync(path)`

**What it does:**

- Permanently deletes the file at the given path
- Throws an error if the file doesn't exist

```javascript
fs.unlinkSync("./demo.txt");
console.log("file deleted");

// demo.txt is now permanently gone
```

> **Warning:** There's no "recycle bin" — the file is **immediately deleted** from disk.

---

### 5.6 Creating a Folder — `mkdirSync()`

**Syntax:** `fs.mkdirSync(path)`

**What it does:**

- Creates a new directory (folder) at the given path
- Throws an error if the folder already exists

```javascript
fs.mkdirSync("../Third Party");
console.log("Created");

// A folder named "Third Party" is created one level up
```

**Creating nested directories:**

```javascript
fs.mkdirSync("./a/b/c", { recursive: true }); // Creates all missing parent folders
```

---

### 5.7 Deleting a Folder — `rmdirSync()`

**Syntax:** `fs.rmdirSync(path)`

**What it does:**

- Deletes an empty directory
- Throws an error if the directory is not empty (in older Node versions)

```javascript
fs.rmdirSync("../Third Party");
console.log("deleted");
```

**Deleting non-empty folders (Node 14.14+):**

```javascript
fs.rmdirSync("./folder", { recursive: true }); // Deletes folder and all contents
```

---

### 5.8 Renaming / Moving a File or Folder — `renameSync()`

**Syntax:** `fs.renameSync(oldPath, newPath)`

**What it does:**

- Renames a file or folder
- Can also **move** a file to a different directory by changing the path

```javascript
// Rename a file
fs.renameSync("./new.java", "./old.java");

// Move a file to a different folder
fs.renameSync("./new.java", "../Local Modules/old.html");
```

---

## 6. Asynchronous fs Methods (Non-Blocking)

All async methods:

- Do NOT end with `Sync`
- Take a **callback function** as the last parameter
- Follow the **error-first callback** pattern

---

### 6.1 Creating a File — `writeFile()`

**Syntax:** `fs.writeFile(path, data, callback)`

```javascript
console.log(1);

fs.writeFile("./demo.py", "some data", (err) => {
  if (err) console.log(err);
  console.log("file created");
});

console.log(2);
console.log(3);

// Output: 1 → 2 → 3 → "file created" (async — doesn't block)
```

---

### 6.2 Reading a File — `readFile()`

**Syntax:** `fs.readFile(path, encoding, callback)`

```javascript
console.log(1);

fs.readFile("./fs.html", "utf-8", (err, payload) => {
  if (err) console.log(err);
  console.log("payload:", payload);
});

console.log(2);
console.log(3);

// Output: 1 → 2 → 3 → "payload: [file content]"
```

---

### Other Async Methods

All the sync methods have async equivalents — just remove `Sync` and add a callback:

| Sync                         | Async                              |
| ---------------------------- | ---------------------------------- |
| `appendFileSync(path, data)` | `appendFile(path, data, callback)` |
| `copyFileSync(src, dest)`    | `copyFile(src, dest, callback)`    |
| `unlinkSync(path)`           | `unlink(path, callback)`           |
| `mkdirSync(path)`            | `mkdir(path, callback)`            |
| `rmdirSync(path)`            | `rmdir(path, callback)`            |
| `renameSync(old, new)`       | `rename(old, new, callback)`       |

---

## 7. Error-First Callbacks

**Definition:** Error-first callbacks are a Node.js convention where the **first parameter** of a callback is **always reserved for an error** (if one occurred).

```javascript
fs.readFile("./demo.txt", "utf-8", (err, data) => {
  //                                   ↑      ↑
  //                              error   success data
  if (err) {
    console.log("Error:", err);
    return; // Stop execution on error
  }
  console.log("Data:", data); // Only runs if no error
});
```

**Pattern:**

```javascript
function callback(err, result) {
  if (err) {
    // Handle error
    return;
  }
  // Use result
}
```

> **Used heavily in:** Node.js built-in modules, Express.js middleware, older async patterns (before Promises).

---

## 8. Streams — Introduction

### What is a Stream?

**Definition:** A stream is a **continuous flow of data** that can be **processed piece by piece** (in chunks), rather than loading the entire file into memory at once.

Think of it like drinking water from a tap vs filling a bucket first:

- **Bucket method** (no streams): Wait for the entire 10GB file to load into RAM, then process it → **RAM explodes if file is huge**
- **Tap method** (streams): Process the file chunk by chunk (e.g., 2MB at a time) → **RAM usage stays low and constant**

---

### Types of Streams in Node.js

| Type          | Direction                   | Example                           |
| ------------- | --------------------------- | --------------------------------- |
| **Readable**  | Data flows OUT (source)     | Reading a file, HTTP request body |
| **Writable**  | Data flows IN (destination) | Writing to a file, HTTP response  |
| **Duplex**    | Both read and write         | TCP socket                        |
| **Transform** | Read, modify, write         | Compression (gzip), encryption    |

---

## 9. Why Streams Matter

### Problem Without Streams

```javascript
// ❌ BAD — loads entire 10GB file into RAM
let content = fs.readFileSync("./huge-video.mp4"); // 10GB loaded into RAM
console.log(content); // RAM usage: 10GB+
// If you have 8GB RAM → CRASH
```

---

### Solution With Streams

```javascript
// ✅ GOOD — processes 10GB file in small chunks (e.g., 2MB at a time)
const readStream = fs.createReadStream("./huge-video.mp4");

readStream.on("data", (chunk) => {
  console.log("Received chunk:", chunk.length, "bytes");
  // Process this 2MB chunk
  // RAM usage: ~2MB (constant, regardless of file size)
});

readStream.on("end", () => {
  console.log("File fully read");
});
```

**Key benefit:** Streams allow you to process files **larger than your available RAM** without crashing.

---

## 10. Streams Architecture Diagram

From the diagram in your image:

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌─────────────┐         ┌──────────────┐         ┌──────────────┐ │
│  │   C Drive   │         │  RAM (8GB)   │         │   Console    │ │
│  │             │         │              │         │              │ │
│  │  pdf→10GB   │────────►│   ┌──────┐   │────────►│    2MB       │ │
│  │             │ streams │   │ data │   │ streams │              │ │
│  │  ┌────────┐ │         │   └──────┘   │         │              │ │
│  │  │ line1  │ │         │              │         │              │ │
│  │  │ line2  │ │         │              │         │              │ │
│  │  │ ....   │ │         │              │         │              │ │
│  │  └────────┘ │         │              │         │              │ │
│  └─────────────┘         └──────────────┘         └──────────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### How it Works

```mermaid
graph LR
    A["C Drive 10GB PDF file (stored on disk)"] -->|Stream chunks 2MB at a time| B["RAM (8GB) Temporary buffer Holds current chunk"]
    B -->|Process & output chunk by chunk| C["Console Displays 2MB at a time"]
```

**Step-by-step flow:**

1. **C Drive (Disk)** — The 10GB PDF file is stored here (permanent storage).
2. **RAM** — Only a **small chunk** (e.g., 2MB) is loaded into RAM at a time via streams.
3. **Console / Output** — The chunk is processed and displayed (or sent over network, written to another file, etc.).
4. **Repeat** — The next 2MB chunk is loaded into RAM (the previous chunk is discarded/freed), and the cycle continues until the entire file is processed.

**Without streams:**

- Entire 10GB file loaded into 8GB RAM → **System crash or swap thrashing**

**With streams:**

- Only 2MB loaded at a time → **RAM usage stays constant** regardless of file size

---

### Real-World Use Cases

| Use Case                               | Why Streams                                                              |
| -------------------------------------- | ------------------------------------------------------------------------ |
| **Video streaming (Netflix, YouTube)** | Can't load entire 4K movie into RAM — stream it chunk by chunk           |
| **Large log file processing**          | Process 50GB server logs without loading all into memory                 |
| **File uploads**                       | User uploads 2GB video — process it as it arrives, not after full upload |
| **Database exports**                   | Export millions of rows — stream results instead of loading all at once  |

---

## 11. Complete Method Reference Table

### Synchronous (Blocking) Methods

| Method           | Syntax                            | Purpose                    |
| ---------------- | --------------------------------- | -------------------------- |
| `writeFileSync`  | `fs.writeFileSync(path, data)`    | Create/overwrite file      |
| `readFileSync`   | `fs.readFileSync(path, encoding)` | Read entire file           |
| `appendFileSync` | `fs.appendFileSync(path, data)`   | Add to end of file         |
| `copyFileSync`   | `fs.copyFileSync(src, dest)`      | Copy file                  |
| `unlinkSync`     | `fs.unlinkSync(path)`             | Delete file                |
| `mkdirSync`      | `fs.mkdirSync(path)`              | Create directory           |
| `rmdirSync`      | `fs.rmdirSync(path)`              | Delete directory           |
| `renameSync`     | `fs.renameSync(old, new)`         | Rename/move file or folder |

---

### Asynchronous (Non-Blocking) Methods

| Method       | Syntax                                  | Purpose                    |
| ------------ | --------------------------------------- | -------------------------- |
| `writeFile`  | `fs.writeFile(path, data, callback)`    | Create/overwrite file      |
| `readFile`   | `fs.readFile(path, encoding, callback)` | Read entire file           |
| `appendFile` | `fs.appendFile(path, data, callback)`   | Add to end of file         |
| `copyFile`   | `fs.copyFile(src, dest, callback)`      | Copy file                  |
| `unlink`     | `fs.unlink(path, callback)`             | Delete file                |
| `mkdir`      | `fs.mkdir(path, callback)`              | Create directory           |
| `rmdir`      | `fs.rmdir(path, callback)`              | Delete directory           |
| `rename`     | `fs.rename(old, new, callback)`         | Rename/move file or folder |

---

## 12. Summary

| Concept                   | Key Point                                                |
| ------------------------- | -------------------------------------------------------- |
| **fs module**             | Built-in Node module for file system operations          |
| **Sync vs Async**         | Sync blocks, Async doesn't — always use async in servers |
| **Buffer**                | Raw binary data stored in RAM, displayed as hex          |
| **Encoding**              | UTF-8 is standard — converts binary to human text        |
| **Error-first callbacks** | First param is always `err` — check before using `data`  |
| **writeFileSync**         | Creates/overwrites file (destructive)                    |
| **appendFileSync**        | Adds to file (non-destructive)                           |
| **readFileSync**          | Reads entire file into memory at once                    |
| **unlinkSync**            | Permanently deletes file                                 |
| **Streams**               | Process large files chunk-by-chunk — saves RAM           |
| **Stream benefit**        | Can process files larger than available RAM              |

---

## 13. Revision Checklist

### Importing & Basics

- [ ] Can you import a built-in module using both ESM and CommonJS syntax?
- [ ] Can you name 5 common built-in Node modules?
- [ ] Can you explain what the fs module does?

### Sync vs Async

- [ ] Can you explain the difference between sync and async with an example?
- [ ] Do you know when to use sync (scripts, CLI) vs async (servers)?
- [ ] Can you predict output order for async code (1 → 2 → 3 → "file created")?

### Buffers & Encoding

- [ ] Can you explain what a Buffer is?
- [ ] Can you explain what encoding is and why UTF-8 is common?
- [ ] Can you convert a Buffer to a string?

### Synchronous Methods

- [ ] Can you write `writeFileSync` to create a file?
- [ ] Can you write `readFileSync` with encoding to read a file?
- [ ] Can you write `appendFileSync` to add data without overwriting?
- [ ] Can you write `copyFileSync` to duplicate a file?
- [ ] Can you write `unlinkSync` to delete a file?
- [ ] Can you write `mkdirSync` to create a folder?
- [ ] Can you write `renameSync` to rename or move a file?

### Asynchronous Methods

- [ ] Can you write `readFile` with a callback?
- [ ] Can you write `writeFile` with a callback?
- [ ] Do you know all sync methods have async equivalents (remove Sync, add callback)?

### Error-First Callbacks

- [ ] Can you explain the error-first callback pattern?
- [ ] Can you write correct error handling (`if (err) return;`)?

### Streams

- [ ] Can you explain what a stream is in simple words?
- [ ] Can you explain the problem streams solve (RAM usage)?
- [ ] Can you trace the diagram: C Drive → RAM → Console?
- [ ] Can you name a real-world use case for streams (video streaming, log processing)?

---

> **Interview tip:** When asked "Why use async instead of sync for file operations?", say: _"Synchronous methods block the entire event loop — if reading a large file takes 2 seconds, no other request can be processed during that time. In a production server handling thousands of requests, this would cause massive delays. Asynchronous methods offload the I/O to libuv, allowing the event loop to keep processing other requests. The callback fires when the I/O completes. This is why Node.js can handle high concurrency despite being single-threaded."_
