# Node.js Event Loop, process.nextTick & Promises

### Complete Guide to Asynchronous Execution Order

---

## 📚 What You'll Learn

This guide covers **Node.js event loop and execution order**:

✅ Node.js Runtime Architecture (Call Stack, Heap, Event Loop)  
✅ Synchronous vs Asynchronous execution  
✅ process.nextTick() — Highest priority  
✅ Promises and microtask queue  
✅ Event Loop phases and queues  
✅ Execution order predictions  
✅ Step-by-step trace of all 8 examples  
✅ Browser vs Node.js differences

**Best for:** Understanding async code, debugging execution order, interview preparation

---

## Table of Contents

1. [Node.js Runtime Architecture](#1-nodejs-runtime-architecture)
2. [Synchronous vs Asynchronous Code](#2-synchronous-vs-asynchronous-code)
3. [The Event Loop — Complete Picture](#3-the-event-loop--complete-picture)
4. [Microtask vs Macrotask Queues](#4-microtask-vs-macrotask-queues)
5. [process.nextTick() — Highest Priority](#5-processnexttick--highest-priority)
6. [Promises and Microtask Queue](#6-promises-and-microtask-queue)
7. [Execution Order Rules](#7-execution-order-rules)
8. [Example 1-8 — Complete Traces](#8-example-1-8--complete-traces)
9. [Browser vs Node.js](#9-browser-vs-nodejs)
10. [Real-World Use Cases](#10-real-world-use-cases)
11. [Summary](#11-summary)
12. [Revision Checklist](#12-revision-checklist)

---

## 1. Node.js Runtime Architecture

### The Complete Picture

```
┌────────────────────────────────────────────────────────────────────┐
│                    NODE.JS RUNTIME ENVIRONMENT                      │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    JAVASCRIPT ENGINE (V8)                     │ │
│  │                                                                │ │
│  │  ┌─────────────────┐           ┌────────────────────────┐   │ │
│  │  │   CALL STACK    │           │        HEAP            │   │ │
│  │  │                 │           │                        │   │ │
│  │  │  [Function 3]   │           │   Objects              │   │ │
│  │  │  [Function 2]   │           │   Variables            │   │ │
│  │  │  [Function 1]   │           │   Arrays               │   │ │
│  │  │  [main()]       │           │   Memory               │   │ │
│  │  └─────────────────┘           └────────────────────────┘   │ │
│  │                                                                │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                       EVENT LOOP (libuv)                      │ │
│  │                                                                │ │
│  │  ┌────────────────────────────────────────────────────────┐ │ │
│  │  │              MICROTASK QUEUES (Priority)               │ │ │
│  │  │                                                          │ │ │
│  │  │  1. nextTick Queue         [cb1] [cb2] [cb3]          │ │ │
│  │  │     ↑ HIGHEST PRIORITY                                 │ │ │
│  │  │                                                          │ │ │
│  │  │  2. Promise Queue          [cb1] [cb2]                 │ │ │
│  │  │     (Microtask Queue)                                  │ │ │
│  │  └────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  │  ┌────────────────────────────────────────────────────────┐ │ │
│  │  │              MACROTASK QUEUES (Phases)                 │ │ │
│  │  │                                                          │ │ │
│  │  │  3. Timer Queue            [setTimeout] [setInterval]  │ │ │
│  │  │     Phase 1: Timers                                    │ │ │
│  │  │                                                          │ │ │
│  │  │  4. I/O Queue              [fs.readFile] [http]        │ │ │
│  │  │     Phase 2: Pending I/O                               │ │ │
│  │  │                                                          │ │ │
│  │  │  5. Check Queue            [setImmediate]              │ │ │
│  │  │     Phase 3: Check                                     │ │ │
│  │  │                                                          │ │ │
│  │  │  6. Close Queue            [socket.on('close')]        │ │ │
│  │  │     Phase 4: Close Callbacks                           │ │ │
│  │  └────────────────────────────────────────────────────────┘ │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    THREAD POOL (libuv)                        │ │
│  │                                                                │ │
│  │     [Thread 1] [Thread 2] [Thread 3] [Thread 4]             │ │
│  │     File I/O, DNS, Crypto, Compression                       │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    NATIVE OS APIS                             │ │
│  │                                                                │ │
│  │     Windows IOCP  |  macOS kqueue  |  Linux epoll            │ │
│  │     (Network I/O, File System Operations)                    │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

---

### Components Explained

#### 1. **Call Stack** (V8)

- **LIFO** (Last In, First Out) structure
- Executes **synchronous code** line by line
- One thing at a time (single-threaded JavaScript)

#### 2. **Heap** (V8)

- **Memory allocation** for objects, variables, arrays
- Garbage collected automatically

#### 3. **Event Loop** (libuv)

- **Coordinator** between Call Stack and Queues
- Monitors Call Stack and Queues
- Pushes callbacks to Call Stack when ready

#### 4. **Microtask Queues** (Highest Priority)

- **nextTick Queue** — `process.nextTick()`
- **Promise Queue** — Promises, `queueMicrotask()`

#### 5. **Macrotask Queues** (Phases)

- **Timer Queue** — `setTimeout()`, `setInterval()`
- **I/O Queue** — File operations, Network requests
- **Check Queue** — `setImmediate()`
- **Close Queue** — Close events (`socket.on('close')`)

#### 6. **Thread Pool** (libuv)

- 4 threads by default (C++ threads)
- Handles CPU-intensive operations

#### 7. **Native OS APIs**

- Operating system handles network I/O
- Different APIs per OS (IOCP, kqueue, epoll)

---

## 2. Synchronous vs Asynchronous Code

### Synchronous Code (Blocking)

**Executes line by line, in order:**

```javascript
console.log(1);
console.log(2);
console.log(3);
let data = fs.readFileSync("./file.txt"); // BLOCKS here
for (let i = 0; i < 1000000; i++) {} // BLOCKS here

// Output:
// 1
// 2
// 3
// (waits for file read)
// (waits for loop)
```

**Does Event Loop come into picture?** ❌ **NO**

**Why?** No asynchronous code = No callbacks = Event Loop has nothing to do

---

### Asynchronous Code (Non-Blocking)

**Delegates work, continues immediately:**

```javascript
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

console.log(3);

// Output:
// 1
// 3
// 2  ← Comes later (from Timer Queue)
```

**Does Event Loop come into picture?** ✅ **YES**

**Why?** `setTimeout` is async → Callback goes to Timer Queue → Event Loop processes it

---

### Visual: Sync vs Async Execution

```
SYNCHRONOUS:
────────────────────────────────────────
Call Stack:
[Line 1] → Execute → Done
[Line 2] → Execute → Done
[Line 3] → Execute → Done

No queues involved!


ASYNCHRONOUS:
────────────────────────────────────────
Call Stack:
[Line 1] → Execute → Done
[setTimeout] → Register callback → Continue
[Line 3] → Execute → Done
[Call Stack Empty!]
    ↓
Event Loop:
Check Timer Queue → [callback] → Push to Call Stack
    ↓
Call Stack:
[callback] → Execute → Done
```

---

## 3. The Event Loop — Complete Picture

### What is the Event Loop?

**A loop that monitors the Call Stack and Queues**, pushing callbacks from queues to the Call Stack when the stack is empty.

Think of it like:

- **Traffic controller** — directs traffic (callbacks)
- **Coordinator** — between Call Stack and Queues
- **Infinite loop** — never stops checking

---

### Event Loop Phases (Simplified)

```
┌─────────────────────────────────────────────────────────────────┐
│                    EVENT LOOP (One Iteration)                   │
└─────────────────────────────────────────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│  1. nextTick Queue (Microtask)       │  ← HIGHEST PRIORITY
│     Execute ALL callbacks            │
└──────────────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│  2. Promise Queue (Microtask)        │  ← HIGH PRIORITY
│     Execute ALL callbacks            │
└──────────────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│  3. Timer Queue (Macrotask)          │  ← setTimeout, setInterval
│     Execute expired timers           │
└──────────────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│  4. I/O Queue (Macrotask)            │  ← fs, http, network
│     Execute I/O callbacks            │
└──────────────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│  5. Check Queue (Macrotask)          │  ← setImmediate
│     Execute setImmediate callbacks   │
└──────────────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│  6. Close Queue (Macrotask)          │  ← close events
│     Execute close callbacks          │
└──────────────────────────────────────┘
         │
         ↓
    (Repeat if queues have callbacks)
```

---

### Key Rule: Microtasks Always First

**After EVERY phase**, Event Loop checks:

1. **nextTick Queue** — Empty it completely
2. **Promise Queue** — Empty it completely
3. Then move to next macrotask phase

---

## 4. Microtask vs Macrotask Queues

### The Two Categories

```
MICROTASKS (Higher Priority):
────────────────────────────────────────
1. nextTick Queue      process.nextTick(cb)
2. Promise Queue       Promises, queueMicrotask(cb)

MACROTASKS (Lower Priority):
────────────────────────────────────────
3. Timer Queue         setTimeout(cb), setInterval(cb)
4. I/O Queue           fs.readFile(cb), http requests
5. Check Queue         setImmediate(cb)
6. Close Queue         socket.on('close', cb)
```

---

### Execution Priority (Highest to Lowest)

```
1. Synchronous Code           ← Executes first
2. process.nextTick()         ← HIGHEST async priority
3. Promises / Microtasks      ← HIGH async priority
4. setTimeout / setInterval   ← MEDIUM async priority
5. setImmediate              ← LOW async priority
6. I/O callbacks             ← Variable timing
```

---

### Visual: Priority Hierarchy

```
┌────────────────────────────────────────────────┐
│         EXECUTION PRIORITY                      │
│                                                 │
│  ╔════════════════════════════════════════╗   │
│  ║  Synchronous Code                      ║   │
│  ║  (console.log, variables, functions)   ║   │
│  ╚════════════════════════════════════════╝   │
│                    ↓                            │
│  ╔════════════════════════════════════════╗   │
│  ║  process.nextTick()                    ║   │
│  ║  ← HIGHEST PRIORITY                    ║   │
│  ╚════════════════════════════════════════╝   │
│                    ↓                            │
│  ╔════════════════════════════════════════╗   │
│  ║  Promise Queue                         ║   │
│  ║  ← HIGH PRIORITY                       ║   │
│  ╚════════════════════════════════════════╝   │
│                    ↓                            │
│  ╔════════════════════════════════════════╗   │
│  ║  Timer Queue (setTimeout)              ║   │
│  ║  ← MEDIUM PRIORITY                     ║   │
│  ╚════════════════════════════════════════╝   │
│                    ↓                            │
│  ╔════════════════════════════════════════╗   │
│  ║  I/O Queue, Check, Close               ║   │
│  ║  ← LOWER PRIORITY                      ║   │
│  ╚════════════════════════════════════════╝   │
└────────────────────────────────────────────────┘
```

---

## 5. process.nextTick() — Highest Priority

### What is process.nextTick()?

**A Node.js-specific function that schedules a callback to execute IMMEDIATELY after the current operation completes**, before any other asynchronous code.

Think of it like:

- **"Do this RIGHT NEXT"**
- **Express lane** — skip the normal queue
- **VIP pass** — highest priority

---

### Syntax

```javascript
process.nextTick(callback);

// Example:
process.nextTick(() => {
  console.log("I run next!");
});
```

---

### Basic Example

```javascript
console.log("1");

process.nextTick(() => {
  console.log("2");
});

console.log("3");

// Output:
// 1
// 3
// 2

// Why?
// 1. Sync code runs first: 1, 3
// 2. Call Stack empty → Event Loop checks nextTick Queue
// 3. Executes callback: 2
```

---

### Why HIGHEST Priority?

**nextTick Queue is checked BEFORE Promise Queue:**

```javascript
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("Promise"));

// Output:
// nextTick  ← Runs first
// Promise   ← Runs second
```

---

### Visual: nextTick Execution

```
Code:
────────────────────────────────────────
console.log("1");
process.nextTick(() => console.log("2"));
console.log("3");

Step-by-Step:
────────────────────────────────────────
Call Stack:        nextTick Queue:
[console.log("1")] []
  ↓ Execute
"1" printed

Call Stack:        nextTick Queue:
[nextTick(...)]    []
  ↓ Register callback
                   [() => console.log("2")]

Call Stack:        nextTick Queue:
[console.log("3")] [() => console.log("2")]
  ↓ Execute
"3" printed

Call Stack:        nextTick Queue:
[]                 [() => console.log("2")]
  ↓ Event Loop activates

Call Stack:        nextTick Queue:
[() => console.log("2")] []
  ↓ Execute
"2" printed

Final Output: 1, 3, 2
```

---

## 6. Promises and Microtask Queue

### Promise Queue (Microtask Queue)

**Promises are added to the Microtask Queue** (also called Promise Queue).

---

### Basic Example

```javascript
console.log("1");

Promise.resolve().then(() => {
  console.log("2");
});

console.log("3");

// Output:
// 1
// 3
// 2

// Why?
// 1. Sync code: 1, 3
// 2. Event Loop checks Promise Queue
// 3. Executes: 2
```

---

### nextTick vs Promise Priority

```javascript
console.log("1");

process.nextTick(() => console.log("2"));

Promise.resolve().then(() => console.log("3"));

console.log("4");

// Output:
// 1
// 4
// 2  ← nextTick (HIGHEST priority)
// 3  ← Promise (HIGH priority)
```

**Rule:** nextTick Queue is COMPLETELY EMPTIED before Promise Queue

---

## 7. Execution Order Rules

### The Golden Rules

```
1. ALL synchronous code executes FIRST

2. AFTER sync code, Event Loop processes queues:
   a) nextTick Queue → Empty COMPLETELY
   b) Promise Queue → Empty COMPLETELY
   c) Timer Queue → Process
   d) I/O Queue → Process
   e) Check Queue → Process
   f) Close Queue → Process

3. If any microtask creates another microtask:
   - Add to respective queue
   - Process in SAME iteration

4. Event Loop processes queues in BATCHES:
   - nextTick: ALL callbacks before moving
   - Promise: ALL callbacks before moving
```

---

### Priority Summary

```
Highest Priority:
  1. Synchronous code
  2. process.nextTick()
  3. Promises / Microtasks

Lower Priority:
  4. setTimeout()
  5. setImmediate()
  6. I/O callbacks
```

---

## 8. Example 1-8 — Complete Traces

### Example 1: Basic nextTick

```javascript
console.log("1");
console.log("2");
console.log("3");

process.nextTick(() => {
  console.log("nt");
});
```

**Step-by-Step Trace:**

```
Call Stack:           nextTick Queue:
────────────────────────────────────────
1. [console.log("1")]  []
   Output: 1

2. [console.log("2")]  []
   Output: 2

3. [console.log("3")]  []
   Output: 3

4. [nextTick(...)]     []
   Register callback → [() => console.log("nt")]

5. []                  [() => console.log("nt")]
   Event Loop checks → Process nextTick Queue

6. [console.log("nt")] []
   Output: nt
```

**Final Output:** `1, 2, 3, nt`

---

### Example 2: Multiple nextTicks

```javascript
console.log(1);

process.nextTick(() => {
  console.log(4);
});

console.log(2);

process.nextTick(() => {
  console.log(3);
});
```

**Step-by-Step Trace:**

```
Call Stack:           nextTick Queue:
────────────────────────────────────────
1. [console.log(1)]    []
   Output: 1

2. [nextTick(...)]     []
   Register → [() => console.log(4)]

3. [console.log(2)]    [() => console.log(4)]
   Output: 2

4. [nextTick(...)]     [() => console.log(4)]
   Register → [() => console.log(4), () => console.log(3)]

5. []                  [() => console.log(4), () => console.log(3)]
   Event Loop processes in ORDER:

6. [console.log(4)]    [() => console.log(3)]
   Output: 4

7. [console.log(3)]    []
   Output: 3
```

**Final Output:** `1, 2, 4, 3`

**Key Point:** nextTick callbacks execute in the ORDER they were added.

---

### Example 3: Nested nextTick

```javascript
console.log(1);

process.nextTick(() => {
  console.log(4);
});

console.log(2);

process.nextTick(() => {
  console.log(3);

  process.nextTick(() => {
    console.log(5);
  });

  console.log(6);
});
```

**Step-by-Step Trace:**

```
Sync code: 1, 2

nextTick Queue after sync: [cb1, cb2]
  cb1 = () => console.log(4)
  cb2 = () => {
           console.log(3);
           process.nextTick(() => console.log(5));
           console.log(6);
        }

Processing nextTick Queue:
────────────────────────────────────────
1. Execute cb1:
   Output: 4

2. Execute cb2:
   - console.log(3) → Output: 3
   - nextTick() → Add to queue: [cb3]
   - console.log(6) → Output: 6

3. Queue now: [cb3]
   Execute cb3:
   Output: 5
```

**Final Output:** `1, 2, 4, 3, 6, 5`

**Key Point:** Nested nextTick is added to queue and processed in SAME iteration.

---

### Example 4: Multiple Nested nextTicks

```javascript
console.log(0);

process.nextTick(() => {
  console.log(1);
});

process.nextTick(() => {
  console.log(2);

  process.nextTick(() => {
    console.log(3);
  });

  console.log(4);
});

process.nextTick(() => {
  console.log(5);
});

console.log(6);
```

**Step-by-Step Trace:**

```
Sync code: 0, 6

nextTick Queue: [cb1, cb2, cb3]

Processing:
────────────────────────────────────────
1. cb1: Output: 1

2. cb2:
   - console.log(2) → Output: 2
   - nextTick() → Add cb4 to queue
   - console.log(4) → Output: 4
   Queue now: [cb3, cb4]

3. cb3: Output: 5

4. cb4: Output: 3
```

**Final Output:** `0, 6, 1, 2, 4, 5, 3`

---

### Example 5: nextTick + Promise

```javascript
console.log("1");

process.nextTick(() => {
  console.log(2);
});

process.nextTick(() => {
  console.log(3);
});

Promise.resolve().then(() => {
  console.log(4);
});

process.nextTick(() => {
  console.log(5);
});

console.log(6);
```

**Step-by-Step Trace:**

```
Sync code: "1", 6

Queues after sync:
────────────────────────────────────────
nextTick Queue: [cb1, cb2, cb3]
  cb1 = () => console.log(2)
  cb2 = () => console.log(3)
  cb3 = () => console.log(5)

Promise Queue: [cb4]
  cb4 = () => console.log(4)

Event Loop processing:
────────────────────────────────────────
1. Empty nextTick Queue FIRST:
   - cb1: Output: 2
   - cb2: Output: 3
   - cb3: Output: 5

2. THEN empty Promise Queue:
   - cb4: Output: 4
```

**Final Output:** `"1", 6, 2, 3, 5, 4`

**Key Rule:** nextTick Queue is COMPLETELY emptied before Promise Queue.

---

### Example 6: Complex nextTick + Promise

```javascript
console.log("1");

process.nextTick(() => {
  console.log(2);
});

process.nextTick(() => {
  console.log(3);

  process.nextTick(() => {
    console.log("3 nested");
  });
});

Promise.resolve().then(() => {
  console.log(4);
});

process.nextTick(() => {
  console.log(5);
});

console.log(6);

Promise.resolve().then(() => {
  console.log(7);

  Promise.resolve().then(() => {
    console.log(8);
  });
});
```

**Step-by-Step Trace:**

```
Sync code: "1", 6

Initial Queues:
────────────────────────────────────────
nextTick: [cb1, cb2, cb3]
Promise: [cb4, cb5]

Process nextTick Queue:
────────────────────────────────────────
1. cb1: Output: 2

2. cb2:
   - console.log(3) → Output: 3
   - Add to nextTick: [cb3, cbNested]

3. cb3: Output: 5

4. cbNested: Output: "3 nested"

Process Promise Queue:
────────────────────────────────────────
5. cb4: Output: 4

6. cb5:
   - console.log(7) → Output: 7
   - Add to Promise: [cbNested2]

7. cbNested2: Output: 8
```

**Final Output:** `"1", 6, 2, 3, 5, "3 nested", 4, 7, 8`

---

### Example 7: Promise inside nextTick

```javascript
console.log("1");

process.nextTick(() => {
  console.log(2);

  Promise.resolve().then(() => {
    console.log("nested promise inside nextTick");
  });
});

process.nextTick(() => {
  console.log(3);
});

Promise.resolve().then(() => {
  console.log(4);

  process.nextTick(() => {
    console.log("nested nextTick inside promise");
  });
});

Promise.resolve().then(() => {
  console.log(5);
});

console.log(6);
```

**Critical Rule:** Event Loop processes queues in BATCHES.

**Step-by-Step Trace:**

```
Sync code: "1", 6

Initial Queues:
────────────────────────────────────────
nextTick: [cb1, cb2]
Promise: [cb3, cb4]

Process nextTick Queue (BATCH):
────────────────────────────────────────
1. cb1:
   - console.log(2) → Output: 2
   - Promise.resolve() → Add to Promise Queue
   Promise Queue now: [cb3, cb4, cbFromNextTick]

2. cb2: Output: 3

nextTick Queue is now EMPTY → Move to Promise Queue

Process Promise Queue (BATCH):
────────────────────────────────────────
3. cb3:
   - console.log(4) → Output: 4
   - nextTick() → Add to nextTick Queue
   nextTick Queue now: [cbFromPromise]

4. cb4: Output: 5

5. cbFromNextTick:
   Output: "nested promise inside nextTick"

Promise Queue is now EMPTY → Check nextTick Queue again

Process nextTick Queue (BATCH):
────────────────────────────────────────
6. cbFromPromise:
   Output: "nested nextTick inside promise"
```

**Final Output:**

```
"1"
6
2
3
4
5
"nested promise inside nextTick"
"nested nextTick inside promise"
```

**Key Rule:** Event Loop processes ENTIRE queue before moving to next queue.

---

### Example 8: Complex Nesting

```javascript
console.log("1");

process.nextTick(() => {
  console.log(2);

  Promise.resolve().then(() => {
    console.log(3);

    Promise.resolve().then(() => {
      console.log(4);
    });
  });
});

process.nextTick(() => {
  console.log(5);
});

Promise.resolve().then(() => {
  console.log(6);

  process.nextTick(() => {
    console.log(7);

    process.nextTick(() => {
      console.log(8);
    });
  });
});

Promise.resolve().then(() => {
  console.log(9);
});

console.log(10);
```

**Step-by-Step Trace:**

```
Sync code: "1", 10

Initial Queues:
────────────────────────────────────────
nextTick: [cb1, cb2]
Promise: [cb3, cb4]

Batch 1: Process nextTick Queue
────────────────────────────────────────
1. cb1:
   - Output: 2
   - Add to Promise: [cb3, cb4, cbA]

2. cb2:
   - Output: 5

Batch 2: Process Promise Queue
────────────────────────────────────────
3. cb3:
   - Output: 6
   - Add to nextTick: [cbB]

4. cb4:
   - Output: 9

5. cbA:
   - Output: 3
   - Add to Promise: [cbC]

6. cbC:
   - Output: 4

Batch 3: Process nextTick Queue
────────────────────────────────────────
7. cbB:
   - Output: 7
   - Add to nextTick: [cbD]

8. cbD:
   - Output: 8
```

**Final Output:** `"1", 10, 2, 5, 6, 9, 3, 4, 7, 8`

---

### Final Example (Your Code)

```javascript
process.nextTick(() => {
  console.log("this is process.nextTick 1");
});

process.nextTick(() => {
  console.log("this is process.nextTick 2");

  process.nextTick(() =>
    console.log("this is the inner next tick inside next tick"),
  );
});

process.nextTick(() => {
  console.log("this is process.nextTick 3");
});

Promise.resolve().then(() => {
  console.log("this is Promise.resolve 1");
});

Promise.resolve().then(() => {
  console.log("this is Promise.resolve 2");

  process.nextTick(() =>
    console.log("this is the inner next tick inside Promise then block"),
  );
});

Promise.resolve().then(() => {
  console.log("this is Promise.resolve 3");
});
```

**Step-by-Step Trace:**

```
No sync code (except registrations)

Initial Queues:
────────────────────────────────────────
nextTick: [nt1, nt2, nt3]
Promise: [p1, p2, p3]

Batch 1: nextTick Queue
────────────────────────────────────────
1. nt1: Output: "this is process.nextTick 1"

2. nt2:
   - Output: "this is process.nextTick 2"
   - Add to nextTick: [nt3, ntNested]

3. nt3: Output: "this is process.nextTick 3"

4. ntNested:
   Output: "this is the inner next tick inside next tick"

Batch 2: Promise Queue
────────────────────────────────────────
5. p1: Output: "this is Promise.resolve 1"

6. p2:
   - Output: "this is Promise.resolve 2"
   - Add to nextTick: [ntFromPromise]

7. p3: Output: "this is Promise.resolve 3"

Batch 3: nextTick Queue (again)
────────────────────────────────────────
8. ntFromPromise:
   Output: "this is the inner next tick inside Promise then block"
```

**Final Output:**

```
this is process.nextTick 1
this is process.nextTick 2
this is process.nextTick 3
this is the inner next tick inside next tick
this is Promise.resolve 1
this is Promise.resolve 2
this is Promise.resolve 3
this is the inner next tick inside Promise then block
```

---

## 9. Browser vs Node.js

### Browser Queue System

**Browser has 2 queues:**

```
1. Microtask Queue:
   - Promises
   - queueMicrotask()
   - MutationObserver

2. Macrotask Queue (Task Queue):
   - setTimeout()
   - setInterval()
   - DOM events
   - UI rendering
```

---

### Node.js Queue System (Corrected)

**Node.js has MULTIPLE queues:**

```
MICROTASK QUEUES:
1. nextTick Queue       process.nextTick()
2. Promise Queue        Promises, queueMicrotask()

MACROTASK QUEUES (Event Loop Phases):
3. Timer Queue          setTimeout(), setInterval()
4. I/O Queue            fs.readFile(), http requests
5. Check Queue          setImmediate()
6. Close Queue          socket.on('close')
```

---

### Key Differences

| Feature              | Browser          | Node.js                         |
| -------------------- | ---------------- | ------------------------------- |
| **Microtask Queues** | 1 queue          | 2 queues (nextTick + Promise)   |
| **Macrotask Queues** | 1 queue          | 4+ queues (phases)              |
| **nextTick**         | ❌ Not available | ✅ Available (highest priority) |
| **setImmediate**     | ❌ Not available | ✅ Available                    |
| **DOM events**       | ✅ Available     | ❌ Not available                |

---

### Common APIs

**Browser AND Node.js:**

```javascript
setTimeout();
setInterval();
Promise;
queueMicrotask();
```

**Node.js ONLY:**

```javascript
process.nextTick();
setImmediate();
```

**Browser ONLY:**

```javascript
requestAnimationFrame()
DOM events
```

---

## 10. Real-World Use Cases

### Use Case 1: Ensuring Order

```javascript
// Problem: Need to ensure callback runs AFTER current operation
function processData(data) {
  // Process data synchronously
  console.log("Processing:", data);

  // Ensure cleanup runs after current operation
  process.nextTick(() => {
    console.log("Cleanup after processing");
  });
}

processData("user data");
console.log("Main code continues");

// Output:
// Processing: user data
// Main code continues
// Cleanup after processing
```

---

### Use Case 2: Error Handling

```javascript
// Ensure errors are always async (consistent behavior)
function fetchData(callback) {
  if (!callback) {
    // Make error async (not sync)
    return process.nextTick(() => {
      throw new Error("Callback required");
    });
  }

  // Simulate async operation
  setTimeout(() => {
    callback(null, "data");
  }, 100);
}
```

---

### Use Case 3: Deferring Heavy Work

```javascript
function processLargeArray(arr) {
  console.log("Start processing");

  arr.forEach((item, index) => {
    // Process each item
    console.log("Processing item", index);

    // After every 100 items, defer to event loop
    if (index % 100 === 0) {
      process.nextTick(() => {
        console.log("Yielding to event loop");
      });
    }
  });
}
```

---

## 11. Summary — Key Takeaways

### 🎯 Execution Priority

```
HIGHEST → LOWEST:

1. Synchronous code
2. process.nextTick()
3. Promises (Microtasks)
4. setTimeout() / setInterval()
5. setImmediate()
6. I/O callbacks
```

---

### 🔄 Queue Processing Rules

```
1. Sync code executes FIRST
2. nextTick Queue → Empty COMPLETELY
3. Promise Queue → Empty COMPLETELY
4. Timer Queue → Process phase
5. I/O Queue → Process phase
6. Check Queue → Process phase
7. Repeat if queues have callbacks
```

---

### 📊 Queue Types

```
MICROTASKS (Highest Priority):
- nextTick Queue
- Promise Queue

MACROTASKS (Lower Priority):
- Timer Queue
- I/O Queue
- Check Queue
- Close Queue
```

---

### ⚠️ Critical Rules

```
✅ Microtasks always process before macrotasks
✅ nextTick always processes before Promises
✅ Event Loop processes queues in BATCHES
✅ Nested callbacks added to respective queues
✅ Each queue is COMPLETELY emptied before moving on
```

---

## 12. Revision Checklist

### Concepts

- [ ] Can you explain the Node.js runtime architecture?
- [ ] Do you know all components (Call Stack, Heap, Event Loop)?
- [ ] Can you explain synchronous vs asynchronous?
- [ ] Do you know when Event Loop activates?

### Queues

- [ ] Can you list all microtask queues?
- [ ] Can you list all macrotask queues?
- [ ] Do you know queue priorities?
- [ ] Can you explain batch processing?

### process.nextTick

- [ ] Do you know what process.nextTick does?
- [ ] Can you explain why it has highest priority?
- [ ] Can you predict nextTick execution order?
- [ ] Do you know about nested nextTicks?

### Promises

- [ ] Do you know Promise queue priority?
- [ ] Can you explain nextTick vs Promise priority?
- [ ] Can you predict Promise execution order?
- [ ] Do you know about nested Promises?

### Execution Order

- [ ] Can you trace Example 1 output?
- [ ] Can you trace Example 2 output?
- [ ] Can you trace Example 5 output?
- [ ] Can you trace Example 7 output?
- [ ] Can you trace the final complex example?

### Browser vs Node.js

- [ ] Do you know the differences?
- [ ] Can you list Node.js-only APIs?
- [ ] Can you list Browser-only APIs?

---

> **🎤 Interview Tip — "Explain the execution order of process.nextTick, Promises, and setTimeout"**
>
> **Answer like this:**
>
> _"The execution order follows a strict priority hierarchy in Node.js:_
>
> _First, all synchronous code executes completely. Once the call stack is empty, the Event Loop begins processing asynchronous callbacks._
>
> _The Event Loop has two categories of queues: microtasks and macrotasks. Microtasks always have higher priority._
>
> _Among microtasks, process.nextTick has the HIGHEST priority. The Event Loop completely empties the nextTick queue before moving to anything else. After that, it empties the Promise queue (also called microtask queue)._
>
> _Only after ALL microtasks are processed does the Event Loop move to macrotasks like setTimeout, which are in the Timer queue._
>
> _Here's a quick example:_
>
> ```javascript
> console.log("1");
> process.nextTick(() => console.log("2"));
> Promise.resolve().then(() => console.log("3"));
> setTimeout(() => console.log("4"), 0);
> console.log("5");
>
> // Output: 1, 5, 2, 3, 4
> // Sync first (1, 5), then nextTick (2), then Promise (3), then setTimeout (4)
> ```
>
> _The key insight is that the Event Loop processes queues in batches — it completely empties one queue before moving to the next. This prevents starvation but means careful attention is needed to avoid blocking with too many microtasks."_
>
> **Why this works:** Shows deep understanding of Event Loop, explains the priority system clearly, provides concrete example with reasoning, mentions the batch processing mechanism, and touches on potential issues.
