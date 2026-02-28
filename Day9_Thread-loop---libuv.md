# Node.js Thread Pool — Understanding libuv & Async Operations

### How Node.js Handles Heavy Tasks (Crypto Example)

---

## 📚 What You'll Learn

This guide explains **how Node.js uses multiple threads** behind the scenes:

✅ What the libuv thread pool is  
✅ The 7 threads in every Node.js process  
✅ How pbkdf2 password hashing works  
✅ Blocking vs non-blocking operations  
✅ What happens when you run 6 tasks with only 4 worker threads  
✅ How to change the thread pool size  
✅ Complete timing analysis and predictions

**Best for:** Understanding Node.js internals, performance optimization, interview preparation

---

## Table of Contents

1. [The Big Picture — 7 Threads in Node.js](#1-the-big-picture--7-threads-in-nodejs)
2. [What is the Thread Pool?](#2-what-is-the-thread-pool)
3. [What is pbkdf2?](#3-what-is-pbkdf2)
4. [Synchronous pbkdf2 — Blocking (Bad!)](#4-synchronous-pbkdf2--blocking-bad)
5. [Asynchronous pbkdf2 — Non-Blocking (Good!)](#5-asynchronous-pbkdf2--non-blocking-good)
6. [The Experiment — 6 Tasks, 4 Threads](#6-the-experiment--6-tasks-4-threads)
7. [What Happens Step by Step](#7-what-happens-step-by-step)
8. [Timing Analysis](#8-timing-analysis)
9. [Changing Thread Pool Size](#9-changing-thread-pool-size)
10. [Complete Code Walkthrough](#10-complete-code-walkthrough)
11. [Real-World Use Cases](#11-real-world-use-cases)
12. [Summary](#12-summary)
13. [Revision Checklist](#13-revision-checklist)

---

## 1. The Big Picture — 7 Threads in Node.js

### Every Node.js Process Has 7 Threads

**When you run any Node.js program**, you automatically get **7 threads running simultaneously**:

```
1 Main Thread        → Executes your JavaScript code
4 libuv Threads      → Handle heavy I/O operations
2 Garbage Collection → Clean up memory automatically
─────────────────────
= 7 threads total
```

---

### Visual: The 7 Threads

```
┌──────────────────────────────────────────────────────┐
│           Your Node.js Application                   │
│                                                      │
│  ┌────────────────┐         ┌──────────────────┐   │
│  │  Main Thread   │         │  libuv Thread    │   │
│  │  (JavaScript)  │◄───────►│  Pool (C++)      │   │
│  │                │         │                  │   │
│  │  Runs your     │         │  Thread 1  ████  │   │
│  │  code line     │         │  Thread 2  ████  │   │
│  │  by line       │         │  Thread 3  ████  │   │
│  └────────────────┘         │  Thread 4  ████  │   │
│                              └──────────────────┘   │
│                                                      │
│  ┌──────────────────┐                               │
│  │  GC Threads      │                               │
│  │  Thread 1        │  (Clean up memory)            │
│  │  Thread 2        │                               │
│  └──────────────────┘                               │
└──────────────────────────────────────────────────────┘
```

---

### What Each Thread Does

| Thread Type              | Count | Purpose                                          |
| ------------------------ | ----- | ------------------------------------------------ |
| **Main Thread**          | 1     | Runs your JavaScript code (the code you write)   |
| **libuv Worker Threads** | 4     | Handle heavy tasks (file I/O, crypto, DNS, etc.) |
| **Garbage Collection**   | 2     | Automatically clean up unused memory             |

---

### Key Insight

**JavaScript is single-threaded** (only 1 main thread runs your code)  
**But Node.js is multi-threaded** (7 threads total working together)

This is why Node.js can handle multiple operations at once even though JavaScript itself is single-threaded!

---

## 2. What is the Thread Pool?

### Simple Definition

**The thread pool is a group of 4 worker threads** that sit idle, waiting for heavy tasks to do.

Think of it like:

- **A restaurant kitchen** with 4 chefs
- **A call center** with 4 operators
- **A help desk** with 4 support staff

When a task comes in, one of the 4 threads picks it up and works on it.

---

### What Tasks Use the Thread Pool?

Not everything uses the thread pool. Only **CPU-intensive operations** or operations that **can't be delegated to the OS**.

| Uses Thread Pool ✅             | Doesn't Use Thread Pool ❌       |
| ------------------------------- | -------------------------------- |
| File system operations (some)   | Network requests (handled by OS) |
| Crypto operations (like pbkdf2) | Timers (handled by event loop)   |
| DNS lookups                     | Most async operations            |
| Compression (zlib)              | Promise callbacks                |

---

### Why Only 4 Threads?

**Default is 4 because:**

- Most computers have 4+ CPU cores
- 4 is a safe balance between parallelism and overhead
- You can change it (we'll see how later)

---

### Visual: Thread Pool in Action

```
Main Thread (your code):
─────────────────────────────────────
console.log("Start");
crypto.pbkdf2(...)  →  [Delegate to thread pool]
crypto.pbkdf2(...)  →  [Delegate to thread pool]
crypto.pbkdf2(...)  →  [Delegate to thread pool]
crypto.pbkdf2(...)  →  [Delegate to thread pool]
console.log("Continue...");  // Main thread keeps running!

Thread Pool:
─────────────────────────────────────
Thread 1: [████████] Working on task 1
Thread 2: [████████] Working on task 2
Thread 3: [████████] Working on task 3
Thread 4: [████████] Working on task 4
```

---

## 3. What is pbkdf2?

### Simple Explanation

**pbkdf2 = Password-Based Key Derivation Function 2**

It's a **password hashing algorithm** that converts a plain-text password into a secure, encrypted hash.

Think of it like:

- **A one-way blender** — once you blend it, you can't "unblend" it
- **A password scrambler** — "myPassword123" becomes "a3f7c9d2e8..."
- **A security lock** — makes passwords unreadable to hackers

---

### Why Use pbkdf2?

**Problem:** Storing passwords in plain text is dangerous:

```javascript
// ❌ NEVER DO THIS
let password = "myPassword123"; // Anyone can see it!
```

**Solution:** Hash the password with pbkdf2:

```javascript
// ✅ SAFE
let hash = pbkdf2("myPassword123", ...);
// Output: "a3f7c9d2e8b4f1a6..." (looks random, can't be reversed)
```

**Benefits:**

1. **Can't be reversed** — even if someone steals the hash, they can't get the original password
2. **Slow on purpose** — takes time to compute (prevents brute-force attacks)
3. **Salted** — same password produces different hashes with different salts

---

### pbkdf2 Parameters Explained

```javascript
crypto.pbkdf2(
  "qwerty123", // 1. Password to hash
  "random-string", // 2. Salt (random data)
  10000000, // 3. Iterations (how many times to scramble)
  64, // 4. Output length (in bytes)
  "sha512", // 5. Hashing algorithm
  (err, data) => {}, // 6. Callback (when done)
);
```

**Parameter breakdown:**

| Parameter      | What It Does                                                     | Example Value           |
| -------------- | ---------------------------------------------------------------- | ----------------------- |
| **Password**   | The plain-text password to hash                                  | `"qwerty123"`           |
| **Salt**       | Random string added to password (prevents rainbow table attacks) | `"random-string"`       |
| **Iterations** | How many times to hash (more = slower = more secure)             | `10000000` (10 million) |
| **Length**     | How long the output hash should be                               | `64` bytes              |
| **Algorithm**  | Which hashing algorithm to use                                   | `"sha512"`              |
| **Callback**   | Function called when hashing is done                             | `(err, data) => {}`     |

---

### Why 10 Million Iterations?

**More iterations = harder to crack**

```
1,000 iterations    → Fast (bad for security)
100,000 iterations  → Normal (good for most cases)
10,000,000 iterations → Very slow (excellent security, but takes time)
```

In our example, we use **10 million iterations** to make the operation take a long time (so we can see the thread pool in action).

---

## 4. Synchronous pbkdf2 — Blocking (Bad!)

### What is Synchronous?

**Synchronous = Blocking = Waits Until Done**

The program **stops and waits** for the operation to complete. Nothing else can run during this time.

---

### Example: pbkdf2Sync (Blocking)

```javascript
console.log("Start");

// This BLOCKS for ~3 seconds
let password1 = crypto.pbkdf2Sync(
  "qwerty123",
  "random-string",
  10000000,
  64,
  "sha512",
);

console.log("Middle"); // Doesn't run until password1 is done

let password2 = crypto.pbkdf2Sync(
  "qwerty123",
  "random-string",
  10000000,
  64,
  "sha512",
);

console.log("End"); // Doesn't run until password2 is done
```

---

### What Happens

```
Time →
─────────────────────────────────────────────────────────
0s:    console.log("Start")
       ↓
0s:    pbkdf2Sync starts
       [████████████████] Blocking for 3 seconds
       Main thread is FROZEN
       ↓
3s:    pbkdf2Sync finishes
       console.log("Middle")
       ↓
3s:    pbkdf2Sync starts again
       [████████████████] Blocking for 3 seconds
       Main thread is FROZEN AGAIN
       ↓
6s:    pbkdf2Sync finishes
       console.log("End")
```

**Total time:** ~6 seconds (3 + 3)

---

### The Problem

```javascript
console.time("hash");

let password1 = crypto.pbkdf2Sync(...);  // 3 seconds
let password2 = crypto.pbkdf2Sync(...);  // 3 seconds
let password3 = crypto.pbkdf2Sync(...);  // 3 seconds

console.timeEnd("hash");
// Output: hash: 9000ms (9 seconds!)
```

**Problems:**

1. **Main thread is blocked** — nothing else can run
2. **Sequential execution** — must do one at a time
3. **Wasted CPU** — if you have 4 cores, only using 1
4. **Bad for servers** — can't handle requests while hashing

---

### Visual: Synchronous Blocking

```
Main Thread (BLOCKED):
──────────────────────────────────────────
[Start] → [Hash1: 3s] → [Hash2: 3s] → [Hash3: 3s] → [End]
            ↑ FROZEN      ↑ FROZEN      ↑ FROZEN
          Nothing       Nothing       Nothing
          can run       can run       can run

Total: 9 seconds
```

---

## 5. Asynchronous pbkdf2 — Non-Blocking (Good!)

### What is Asynchronous?

**Asynchronous = Non-Blocking = Doesn't Wait**

The program **delegates the task** to the thread pool and **continues immediately**. The result comes back later via a callback.

---

### Example: pbkdf2 (Non-Blocking)

```javascript
console.log("Start");

crypto.pbkdf2(
  "qwerty123",
  "random-string",
  10000000,
  64,
  "sha512",
  (err, data) => {
    console.log("Hash 1 done!");
  },
);

console.log("Middle"); // Runs IMMEDIATELY (doesn't wait)

crypto.pbkdf2(
  "qwerty123",
  "random-string",
  10000000,
  64,
  "sha512",
  (err, data) => {
    console.log("Hash 2 done!");
  },
);

console.log("End"); // Runs IMMEDIATELY
```

---

### What Happens

```
Time →
─────────────────────────────────────────────────────────
0ms:   console.log("Start")

0ms:   crypto.pbkdf2(...) → Delegated to Thread Pool
       Main thread CONTINUES (doesn't wait)

0ms:   console.log("Middle")

0ms:   crypto.pbkdf2(...) → Delegated to Thread Pool
       Main thread CONTINUES

0ms:   console.log("End")

       [Main thread is now free!]

3000ms: Thread 1 finishes → "Hash 1 done!"
3000ms: Thread 2 finishes → "Hash 2 done!"
```

**Output order:**

```
Start
Middle
End
Hash 1 done!  (after ~3 seconds)
Hash 2 done!  (after ~3 seconds)
```

---

### The Magic

**Both hashes run in parallel** (at the same time) because they use different threads!

```
Main Thread:
────────────────────────────────────────
[Start] → [Delegate] → [Middle] → [Delegate] → [End] → [Free!]
   0ms       0ms         0ms         0ms        0ms

Thread Pool:
────────────────────────────────────────
Thread 1: [████████████████] Hash 1 (3 seconds)
Thread 2: [████████████████] Hash 2 (3 seconds)
          ↑ Both run at the SAME TIME!

Total time: ~3 seconds (not 6!)
```

---

## 6. The Experiment — 6 Tasks, 4 Threads

### The Code

```javascript
function hp1() {
  console.time("hashAsync1");
  crypto.pbkdf2(
    "qwerty123",
    "random-string",
    10000000,
    64,
    "sha512",
    (err, data) => {
      console.timeEnd("hashAsync1");
    },
  );
}

function hp2() {
  /* same pattern */
}
function hp3() {
  /* same pattern */
}
function hp4() {
  /* same pattern */
}
function hp5() {
  /* same pattern */
}
function hp6() {
  /* same pattern */
}

// Call all 6 at once
hp1();
hp2();
hp3();
hp4();
hp5();
hp6();
```

---

### The Question

**What happens when you have:**

- **6 tasks** to do
- But only **4 worker threads**?

**Answer:** Some tasks have to wait!

---

### Visual: Queue Forms

```
Tasks Submitted (all at once):
─────────────────────────────────
Task 1  ─┐
Task 2  ─┤
Task 3  ─┤─► Thread Pool (4 threads available)
Task 4  ─┘
Task 5  ─┐
Task 6  ─┘─► Must wait in queue

Thread Pool:
─────────────────────────────────
Thread 1: [████████] Task 1 (3 seconds)
Thread 2: [████████] Task 2 (3 seconds)
Thread 3: [████████] Task 3 (3 seconds)
Thread 4: [████████] Task 4 (3 seconds)

Queue:
  Task 5 (waiting...)
  Task 6 (waiting...)
```

---

## 7. What Happens Step by Step

### Timeline Breakdown

**Time: 0ms (Start)**

```
Main Thread:
  Calls hp1() → Delegates to thread pool
  Calls hp2() → Delegates to thread pool
  Calls hp3() → Delegates to thread pool
  Calls hp4() → Delegates to thread pool
  Calls hp5() → Delegates to thread pool
  Calls hp6() → Delegates to thread pool
  [Main thread finishes, goes idle]

Thread Pool (4 threads):
  Thread 1: Starts Task 1
  Thread 2: Starts Task 2
  Thread 3: Starts Task 3
  Thread 4: Starts Task 4

Queue (2 tasks waiting):
  Task 5 (waiting)
  Task 6 (waiting)
```

---

**Time: ~3000ms (First Batch Finishes)**

```
Thread 1: ✓ Task 1 DONE! → Pick up Task 5
Thread 2: ✓ Task 2 DONE! → Pick up Task 6
Thread 3: ✓ Task 3 DONE! → Idle (no more work)
Thread 4: ✓ Task 4 DONE! → Idle (no more work)

Output:
  hashAsync1: 3000ms
  hashAsync2: 3000ms
  hashAsync3: 3000ms
  hashAsync4: 3000ms
```

---

**Time: ~6000ms (Second Batch Finishes)**

```
Thread 1: ✓ Task 5 DONE!
Thread 2: ✓ Task 6 DONE!

Output:
  hashAsync5: 6000ms
  hashAsync6: 6000ms
```

---

### Final Output (Predicted)

```
hashAsync1: ~3000ms
hashAsync2: ~3000ms
hashAsync3: ~3000ms
hashAsync4: ~3000ms
hashAsync5: ~6000ms  ← Had to wait for a thread to become free
hashAsync6: ~6000ms  ← Had to wait for a thread to become free
```

---

### Visual: Complete Timeline

```
Time:  0ms          3000ms         6000ms
       ↓             ↓              ↓
       ┌─────────────┬──────────────┐
T1:    [████████████]|              |  ← Task 1 (3s)
T2:    [████████████]|              |  ← Task 2 (3s)
T3:    [████████████]|              |  ← Task 3 (3s)
T4:    [████████████]|              |  ← Task 4 (3s)
       └─────────────┴──────────────┘
       ┌─────────────┬──────────────┐
T5:    Wait...       [████████████] |  ← Task 5 (starts at 3s, finishes at 6s)
T6:    Wait...       [████████████] |  ← Task 6 (starts at 3s, finishes at 6s)
       └─────────────┴──────────────┘
```

**Key insight:** Tasks 5 and 6 had to **wait 3 seconds** for a thread to become available!

---

## 8. Timing Analysis

### Understanding the Results

**Batch 1 (Tasks 1-4):**

```
Start immediately (0ms)
All 4 threads busy
Finish at ~3000ms
```

**Batch 2 (Tasks 5-6):**

```
Start after Batch 1 finishes (3000ms)
Use threads that became free
Finish at ~6000ms (3000ms start + 3000ms work)
```

---

### Why Not All the Same Time?

```
❌ Wrong Expectation:
  "All 6 tasks should finish at the same time (3000ms)"

  Why wrong? Only 4 threads available!

✅ Correct Understanding:
  "First 4 finish at 3000ms, next 2 finish at 6000ms"

  Why? 6 tasks > 4 threads = some must wait
```

---

### Real Timing (Varies Slightly)

```
Typical output:
hashAsync1: 3021ms  ← ~3 seconds
hashAsync2: 3034ms  ← ~3 seconds
hashAsync3: 3018ms  ← ~3 seconds
hashAsync4: 3027ms  ← ~3 seconds
hashAsync5: 6042ms  ← ~6 seconds (waited ~3s, then worked ~3s)
hashAsync6: 6051ms  ← ~6 seconds
```

**Why variations (3021ms vs 3034ms)?**

- CPU scheduling
- Other processes running
- Random small delays

**But the pattern is clear:** 4 at ~3s, 2 at ~6s

---

## 9. Changing Thread Pool Size

### The Problem

**Default 4 threads** might not be enough for your workload:

```
6 tasks with 4 threads:
  Batch 1: 4 tasks (3 seconds)
  Batch 2: 2 tasks (3 seconds)
  Total: 6 seconds

What if we had 6 threads instead?
  All 6 tasks at once (3 seconds)
  Total: 3 seconds (2x faster!)
```

---

### How to Change Thread Pool Size

**Using Environment Variable:**

```bash
# Linux / Mac
UV_THREADPOOL_SIZE=6 node app.js

# Windows (Command Prompt)
set UV_THREADPOOL_SIZE=6
node app.js

# Windows (PowerShell)
$env:UV_THREADPOOL_SIZE=6
node app.js
```

---

### Example: 6 Threads

```bash
$ UV_THREADPOOL_SIZE=6 node crypto-test.js

# Output:
hashAsync1: 3021ms  ← All 6 finish
hashAsync2: 3028ms  ← at the same
hashAsync3: 3019ms  ← time now!
hashAsync4: 3025ms  ← Because we have
hashAsync5: 3031ms  ← 6 threads
hashAsync6: 3027ms  ← available!
```

**Result:** All tasks finish in ~3 seconds (not 6 seconds)

---

### Comparison Table

| Thread Pool Size        | Tasks 1-4 Finish     | Tasks 5-6 Finish | Total Time |
| ----------------------- | -------------------- | ---------------- | ---------- |
| **2 threads**           | ~6s (2 batches of 2) | ~9s              | ~9s        |
| **4 threads** (default) | ~3s                  | ~6s              | ~6s        |
| **6 threads**           | ~3s                  | ~3s              | ~3s        |
| **8 threads**           | ~3s                  | ~3s              | ~3s        |

**Diminishing returns:** After 6 threads, no improvement (because we only have 6 tasks)

---

### When to Increase Thread Pool Size

| Increase When              | Don't Increase When         |
| -------------------------- | --------------------------- |
| ✅ Heavy crypto operations | ❌ Simple network requests  |
| ✅ Many file operations    | ❌ Just timers and promises |
| ✅ Lots of DNS lookups     | ❌ CPU-light operations     |
| ✅ Compression tasks       | ❌ You have 1-2 tasks only  |

**Rule of thumb:**

- 4 threads (default) is good for most apps
- Increase to # of CPU cores for CPU-intensive work
- Don't go crazy (16+ threads won't help much)

---

### Visual: Thread Pool Sizes

```
2 Threads (too few):
────────────────────────────────────────
T1: [████] Wait [████] Wait [████]
T2: [████] Wait [████] Wait [████]
    0-3s   3-6s   6-9s  Total: 9s

4 Threads (default):
────────────────────────────────────────
T1: [████████]
T2: [████████]
T3: [████████]
T4: [████████]
    0-3s      3-6s  Total: 6s

6 Threads (optimal for 6 tasks):
────────────────────────────────────────
T1: [████████]
T2: [████████]
T3: [████████]
T4: [████████]
T5: [████████]
T6: [████████]
    0-3s  Total: 3s
```

---

## 10. Complete Code Walkthrough

### The Full Code Explained

```javascript
import crypto from "node:crypto";

// Function 1: Hash password asynchronously
function hp1() {
  console.time("hashAsync1"); // Start timer

  crypto.pbkdf2(
    "qwerty123", // Password
    "random-string", // Salt
    10000000, // 10 million iterations (slow!)
    64, // 64 bytes output
    "sha512", // Algorithm
    (err, data) => {
      // Callback when done
      console.timeEnd("hashAsync1"); // Show elapsed time
    },
  );
}

// Functions 2-6: Same pattern
function hp2() {
  /* ... */
}
function hp3() {
  /* ... */
}
function hp4() {
  /* ... */
}
function hp5() {
  /* ... */
}
function hp6() {
  /* ... */
}

// Execute all 6 at once (main thread doesn't wait)
hp1(); // Delegates to thread pool
hp2(); // Delegates to thread pool
hp3(); // Delegates to thread pool
hp4(); // Delegates to thread pool
hp5(); // Delegates to thread pool (must wait in queue)
hp6(); // Delegates to thread pool (must wait in queue)

// Main thread finishes immediately!
// Thread pool works in background
```

---

### What Happens in Memory

```
Main Thread (JavaScript):
─────────────────────────────────────
1. Import crypto module
2. Define functions hp1-hp6
3. Call hp1() → Delegate to thread pool
4. Call hp2() → Delegate to thread pool
5. Call hp3() → Delegate to thread pool
6. Call hp4() → Delegate to thread pool
7. Call hp5() → Add to queue (no free thread)
8. Call hp6() → Add to queue (no free thread)
9. Main thread finishes
10. Wait for callbacks...

libuv Thread Pool (C++):
─────────────────────────────────────
Thread 1: [Processing hp1...]
Thread 2: [Processing hp2...]
Thread 3: [Processing hp3...]
Thread 4: [Processing hp4...]

Queue:
  hp5 (waiting)
  hp6 (waiting)

After 3 seconds:
─────────────────────────────────────
Thread 1: ✓ hp1 done! → Callback → Pick up hp5
Thread 2: ✓ hp2 done! → Callback → Pick up hp6
Thread 3: ✓ hp3 done! → Callback → Idle
Thread 4: ✓ hp4 done! → Callback → Idle

After 6 seconds:
─────────────────────────────────────
Thread 1: ✓ hp5 done! → Callback
Thread 2: ✓ hp6 done! → Callback
```

---

## 11. Real-World Use Cases

### When You'll See This in Production

#### 1. User Registration System

```javascript
// When user signs up, hash their password
app.post("/register", async (req, res) => {
  const { username, password } = req.body;

  // This uses the thread pool (doesn't block main thread)
  crypto.pbkdf2(password, salt, 100000, 64, "sha512", (err, hashedPassword) => {
    // Save user to database
    saveUser(username, hashedPassword);
    res.send("Account created!");
  });

  // Server can handle other requests while hashing!
});
```

---

#### 2. Batch Processing Images

```javascript
const images = ['img1.jpg', 'img2.jpg', ..., 'img100.jpg'];

// Process all images concurrently
images.forEach(img => {
    // File operations use thread pool
    fs.readFile(img, (err, data) => {
        // Resize, compress, save
        processImage(data);
    });
});

// With 4 threads: processes 4 at a time
// With UV_THREADPOOL_SIZE=10: processes 10 at a time
```

---

#### 3. CSV File Processing

```javascript
const files = [
  "sales-jan.csv",
  "sales-feb.csv",
  "sales-mar.csv",
  "sales-apr.csv",
  "sales-may.csv",
  "sales-jun.csv",
];

// Process all files
files.forEach((file) => {
  fs.readFile(file, "utf-8", (err, data) => {
    const rows = data.split("\n");
    const total = calculateTotal(rows);
    console.log(`${file}: $${total}`);
  });
});

// Default 4 threads:
//   First 4 files process at once
//   Last 2 wait for threads to free up
```

---

## 12. Summary — Key Takeaways

### 🎯 The Big Ideas

| Concept                  | Key Point                          |
| ------------------------ | ---------------------------------- |
| **7 threads total**      | 1 main + 4 libuv + 2 GC            |
| **Thread pool**          | 4 worker threads (by default)      |
| **pbkdf2**               | Password hashing (slow on purpose) |
| **Sync = blocking**      | Freezes main thread                |
| **Async = non-blocking** | Delegates to thread pool           |
| **6 tasks, 4 threads**   | Some must wait in queue            |
| **Change pool size**     | `UV_THREADPOOL_SIZE=6`             |

---

### 📊 Quick Facts

```
Default thread pool size: 4
Main thread count: 1
GC thread count: 2
Total threads: 7

Can increase pool? Yes (UV_THREADPOOL_SIZE)
Recommended max: # of CPU cores
```

---

### ⚡ Performance Rules

```
More threads = more parallel work
But: Don't exceed CPU core count
Why: Context switching overhead

Example:
  4 cores + 4 threads = Good ✅
  4 cores + 16 threads = Bad ❌ (too much overhead)
```

---

### 🔄 The Flow

```
1. You call crypto.pbkdf2() (async)
2. Main thread delegates to thread pool
3. Main thread continues (non-blocking)
4. Thread pool thread processes task
5. When done, callback runs
6. Result available in callback
```

---

## 13. Revision Checklist

### Core Concepts

- [ ] Can you name the 7 threads in Node.js?
- [ ] Can you explain what the thread pool is?
- [ ] Do you know the default thread pool size (4)?
- [ ] Can you explain what pbkdf2 does?

### Blocking vs Non-Blocking

- [ ] Can you explain synchronous (blocking)?
- [ ] Can you explain asynchronous (non-blocking)?
- [ ] Do you know why sync is bad for servers?
- [ ] Can you predict output order of async calls?

### The Experiment

- [ ] Can you explain what happens with 6 tasks and 4 threads?
- [ ] Do you know which tasks finish first (1-4)?
- [ ] Do you know which tasks must wait (5-6)?
- [ ] Can you explain why tasks 5-6 take 6 seconds?

### Thread Pool Size

- [ ] Do you know how to change thread pool size?
- [ ] Can you write the UV_THREADPOOL_SIZE command?
- [ ] Do you know when to increase the pool size?
- [ ] Do you know the recommended maximum?

### Timing

- [ ] Can you predict timing for 4 threads?
- [ ] Can you predict timing for 6 threads?
- [ ] Can you explain timing variations?
- [ ] Do you know why batching occurs?

---

> **🎤 Interview Tip — "How does Node.js handle multiple heavy operations concurrently if JavaScript is single-threaded?"**
>
> **Answer like this:**
>
> _"Great question! While JavaScript itself is single-threaded, Node.js uses a library called libuv written in C that provides a thread pool with 4 worker threads by default. When you perform CPU-intensive operations like crypto hashing or file I/O, Node.js delegates those tasks to the thread pool._
>
> _Here's how it works: The main thread receives the operation, immediately delegates it to an available thread in the pool, and continues executing without waiting. The worker thread processes the task in the background. When complete, the callback is queued to run on the main thread._
>
> _For example, if I call crypto.pbkdf2 six times asynchronously with the default 4-thread pool, the first four operations start immediately on the four threads. Operations five and six wait in a queue. When any of the first four complete after about 3 seconds, threads become free and pick up the waiting tasks. So operations 1-4 finish at ~3 seconds, and operations 5-6 finish at ~6 seconds._
>
> _If needed, I can increase the pool size using the UV_THREADPOOL_SIZE environment variable, but generally 4 threads is optimal for most applications since it matches typical CPU core counts."_
>
> **Why this works:** Shows understanding of both JavaScript's single-threaded nature AND Node's multi-threaded architecture, provides concrete example with timing, and demonstrates knowledge of configuration options.
