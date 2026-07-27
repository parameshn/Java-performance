# Tiered Compilation Levels

When **Tiered Compilation** is enabled, the compilation log shows the **tier level** at which each method is compiled. Although the JVM has only two JIT compilers—**C1** and **C2**—there are **five compilation levels** because the C1 compiler supports three compilation stages. :contentReference[oaicite:0]{index=0}

## Compilation Levels

| Level | Compiler     | Description                                                                                                                                                 |
| ----: | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **0** | Interpreter  | Executes bytecode and collects runtime profiling information. This level does not appear in the compilation log.                                            |
| **1** | C1 (Simple)  | Performs fast compilation with basic optimizations. Used for simple or trivial methods, or when higher levels are unnecessary.                              |
| **2** | C1 (Limited) | Uses invocation counters and back-edge (loop) counters for limited optimizations without full profiling. Often used when compiler queues are busy.          |
| **3** | C1 (Full)    | Uses full profiling information and continues collecting runtime data for the C2 compiler. This is the most common first compilation level for hot methods. |
| **4** | C2           | Produces highly optimized native code using the complete runtime profile collected by earlier stages.                                                       |

---

# Normal Compilation Path

Every method begins execution in the **interpreter (Level 0)**.

If the method becomes hot, it is first compiled by the **C1 compiler at Level 3**. While executing at Level 3, the JVM continues collecting profiling information.

If the method remains hot, the **C2 compiler** recompiles it at **Level 4**, replacing the previous C1 version.

```text
Level 0 (Interpreter)
        │
        ▼
Level 3 (Full C1)
        │
        ▼
Level 4 (C2)
```

When the Level 4 version becomes available, the previous Level 3 compiled code is marked **made not entrant**, allowing future calls to execute the faster C2 version. :contentReference[oaicite:1]{index=1}

---

# When the C2 Compiler Queue Is Busy

If the **C2 compiler queue** is busy, the JVM does not wait for C2 compilation.

Instead, the method is first compiled at **Level 2**, which relies only on **invocation counters** and **back-edge counters**.

After more profiling information is collected, the method is promoted to **Level 3**, and finally to **Level 4** when the C2 compiler becomes available.

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

This allows the method to execute as compiled code sooner instead of waiting in the C2 queue.

---

# When the C1 Compiler Queue Is Busy

If the **C1 compiler queue** is busy, a method may become eligible for C2 compilation before a Level 3 compilation occurs.

In that situation, the JVM typically compiles the method at **Level 2** and then promotes it directly to **Level 4**.

```text
Level 0
    │
    ▼
Level 2
    │
    ▼
Level 4
```

This avoids delaying optimization because of a busy C1 compiler queue.

---

# Trivial Methods

Very small or **trivial methods** usually do not benefit from aggressive optimization.

Such methods may be compiled only at **Level 1**, where the C1 compiler performs simple optimizations.

Similarly, if the C2 compiler cannot compile a method for some reason, the JVM may leave it at **Level 1** because further optimization would provide little benefit.

---

# Deoptimization

If compiled code becomes invalid—for example, because compiler assumptions change—the JVM **deoptimizes** the method.

The compiled code is discarded, execution falls back to the **interpreter (Level 0)**, and the JVM begins collecting profiling information again.

If the method becomes hot once more, it follows the normal compilation process.

```text
Compiled Code
      │
      ▼
Deoptimization
      │
      ▼
Level 0 (Interpreter)
      │
      ▼
Level 3
      │
      ▼
Level 4
```

---

# Performance Considerations

Tiered Compilation is designed to balance **startup performance** and **peak performance**.

The preferred compilation path for most hot methods is:

```text
Level 0
    │
    ▼
Level 3
    │
    ▼
Level 4
```

Occasional use of **Level 2** is normal. Frequent Level 2 compilations may indicate that the **C2 compiler queue** is busy.

In most applications, the JVM automatically manages these compilation levels effectively, and manually tuning them provides little practical benefit. 

---

# Summary

- Tiered Compilation consists of **five compilation levels** built on the **C1** and **C2** compilers.
- Most hot methods follow the path **Level 0 → Level 3 → Level 4**.
- **Level 2** is primarily an intermediate stage used when compiler queues are busy.
- **Level 1** is mainly used for trivial methods or methods that do not require aggressive optimization.
- During deoptimization, execution temporarily returns to **Level 0**, after which the JVM recompiles the method if it becomes hot again.
- The JVM automatically chooses the appropriate compilation level, so manual tuning is rarely necessary.