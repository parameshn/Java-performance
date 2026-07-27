# Compilation Threads

When a method or loop reaches its **compilation threshold**, the JVM does **not** compile it immediately.

Instead, the method is placed into a **compilation queue**, where one or more **compiler threads** process it in the background. This allows the application to continue running while compilation takes place. :contentReference[oaicite:0]{index=0}

---

# Compilation Queue

The compilation queue is **not** a simple First-In, First-Out (FIFO) queue.

Methods that are executed more frequently (hotter methods) are given **higher priority** than less frequently executed methods.

This ensures that the methods with the greatest impact on performance are compiled first.

```text
Compilation Queue

Hot Method A      (Highest Priority)
Hot Method B
Warm Method C
Cold Method D

        │
        ▼
Compiler Threads
```

Because of this priority scheduling, the **compilation IDs** shown by `-XX:+PrintCompilation` may appear out of order. :contentReference[oaicite:1]{index=1}

---

# Separate C1 and C2 Queues

With **Tiered Compilation**, the JVM maintains **two separate compilation queues**:

- **C1 compilation queue**
- **C2 compilation queue**

Each queue is processed by its own compiler threads.

```text
                Methods Become Hot
                       │
      ┌────────────────┴────────────────┐
      ▼                                 ▼
 C1 Compilation Queue             C2 Compilation Queue
      │                                 │
 C1 Compiler Threads             C2 Compiler Threads
```

This allows C1 and C2 compilation to occur independently. :contentReference[oaicite:2]{index=2}

---

# Number of Compiler Threads

The JVM automatically determines the number of compiler threads based on the number of available CPUs.

For example:

| CPUs | C1 Threads | C2 Threads |
| ---: | ---------: | ---------: |
|    1 |          1 |          1 |
|    2 |          1 |          1 |
|    4 |          1 |          2 |
|    8 |          1 |          2 |
|   16 |          2 |          6 |
|   32 |          3 |          7 |
|   64 |          4 |          8 |
|  128 |          4 |         10 |

The default values are chosen automatically and are appropriate for most applications. :contentReference[oaicite:3]{index=3}

---

# `-XX:CICompilerCount`

The total number of compiler threads can be changed using:

```text
-XX:CICompilerCount=N
```

When **Tiered Compilation** is enabled:

- Approximately **one-third** of the compiler threads are assigned to the **C1 compiler**.
- The remaining threads are assigned to the **C2 compiler**.
- Each compiler always has at least one thread.

When Tiered Compilation is disabled, all compiler threads are used by the C2 compiler. :contentReference[oaicite:4]{index=4}

---

# Background Compilation

By default, compilation occurs **asynchronously**.

The JVM continues executing the application while compiler threads generate machine code in the background.

```text
Application Thread
        │
Executes Method
        │
Method Becomes Hot
        │
        ├──────────────► Compilation Queue
        │                      │
        │               Compiler Thread
        │                      │
Continue Running         Compile Machine Code
```

This behavior is controlled by:

```text
-XX:+BackgroundCompilation
```

which is **enabled by default**. :contentReference[oaicite:5]{index=5}

---

# `-Xbatch`

If background compilation is disabled using:

```text
-Xbatch
```

or

```text
-XX:-BackgroundCompilation
```

the application thread **waits** until compilation finishes.

```text
Method Becomes Hot
        │
        ▼
Compile Now
        │
(Application Waits)
        │
        ▼
Continue Execution
```

This is mainly useful for testing and benchmarking, not for normal production workloads. :contentReference[oaicite:6]{index=6}

---

# When Should You Change the Number of Compiler Threads?

For most applications, **you should not**.

The JVM automatically selects an appropriate number based on the available CPUs.

According to the book, changing `CICompilerCount` may be useful only in special situations, such as:

- Older JDK 8 versions running inside Docker containers where CPU detection is incorrect.
- Single-CPU virtual machines where fewer compiler threads reduce CPU contention during startup.
- Systems running many JVMs simultaneously, where reducing compiler threads may improve overall throughput.

Increasing compiler threads rarely provides significant benefits beyond slightly reducing application warm-up time. :contentReference[oaicite:7]{index=7}

---

# Summary

- Methods that become hot are placed into a **compilation queue** instead of being compiled immediately.
- The queue is **priority-based**, not FIFO; hotter methods are compiled first.
- Tiered Compilation maintains **separate queues** for the **C1** and **C2** compilers.
- The JVM automatically determines the number of compiler threads based on available CPUs.
- The number of compiler threads can be changed using `-XX:CICompilerCount`, but manual tuning is rarely needed.
- Compilation normally occurs **asynchronously** in the background using `-XX:+BackgroundCompilation`.
- Using `-Xbatch` disables background compilation and forces application threads to wait until compilation completes. :contentReference[oaicite:8]{index=8}