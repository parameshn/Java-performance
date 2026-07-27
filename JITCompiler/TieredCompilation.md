## Tiered Compilation Levels

When Tiered Compilation is enabled, the compilation log displays the **tier level** at which each method is compiled.

Although the JVM has two JIT compilers (C1 and C2), there are **five compilation levels** because the C1 compiler operates in multiple stages.

| Level | Description              |
| ----: | ------------------------ |
| **0** | Interpreted code         |
| **1** | Simple C1 compiled code  |
| **2** | Limited C1 compiled code |
| **3** | Full C1 compiled code    |
| **4** | C2 compiled code         |

### Normal Compilation Path

A typical compilation log shows that most methods follow this path:

```text
Level 0 (Interpreted)
        │
        ▼
Level 3 (Full C1)
        │
        ▼
Level 4 (C2)
```

All methods begin at **Level 0**, though this is not shown in the compilation log.

The C1 compiler waits until it has gathered enough information about how a method is used before performing **Level 3** compilation. If the method continues to execute frequently, it is later compiled by the C2 compiler at **Level 4**. The previous Level 3 version is then **made not entrant**.

This is the most common compilation path.

### When the C2 Compiler Queue Is Full

If the **C2 compiler queue** becomes full, methods are removed from the queue and compiled at **Level 2** instead.

At Level 2, the C1 compiler uses **invocation** and **back-edge counters**, but it does not require profile feedback.

Later, the method is compiled again:

```text
Level 0
    │
    ▼
Level 2
    │
    ▼
Level 3
    │
    ▼
Level 4
```

Once the C2 compiler queue becomes less busy, the method is eventually compiled at **Level 4**.

### When the C1 Compiler Queue Is Full

If the **C1 compiler queue** is full, a method waiting for **Level 3** compilation may become eligible for **Level 4**.

In that case, the JVM compiles it quickly at **Level 2** before moving it directly to **Level 4**.

```text
Level 0
    │
    ▼
Level 2
    │
    ▼
Level 4
```

### Trivial Methods

Very small (trivial) methods may:

- Start at **Level 2** or **Level 3**.
- Eventually move to **Level 1** because of their trivial nature.

If the **C2 compiler** cannot compile a method for some reason, that method is also compiled at **Level 1**.

### Deoptimization

When compiled code is **deoptimized**, it returns to:

```text
Level 0 (Interpreted)
```

### Performance Considerations

Several JVM flags control tiered compilation behavior, but tuning at this level rarely produces significant performance improvements.

The preferred compilation path is:

```text
Level 0 → Level 3 → Level 4
```

If methods are frequently compiled at **Level 2**, and additional CPU resources are available, increasing the number of compiler threads may reduce the size of the **C2 compiler queue**.

If additional CPU resources are not available, the remaining option is to reduce the size or complexity of the application.