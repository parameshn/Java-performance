# Deoptimization

**Deoptimization** is the process by which the JVM **undoes a previous JIT compilation**.

The JIT compiler makes optimization decisions based on assumptions gathered from runtime profiling. If those assumptions later become invalid, the JVM discards the optimized machine code and either:

- Falls back to interpreted execution, or
- Recompiles the code using updated runtime information.

During this process, performance may temporarily decrease until the code is compiled again. :contentReference[oaicite:0]{index=0}

---

## Why Does Deoptimization Happen?

According to the book, deoptimization occurs in two situations:

1. **Code is made not entrant**
2. **Code is made zombie** :contentReference[oaicite:1]{index=1}

---

# 1. Made Not Entrant

A compiled method is marked **not entrant** when it should **no longer be used for future method calls**. Threads that are already executing the compiled code are allowed to finish, but any new calls will use different code.

There are two common reasons for this.

---

## Case 1: Compiler Assumptions Become Invalid

The JIT compiler optimizes code based on what it observes during execution.

For example:

```java
StockPriceHistory sph;

if (log == null)
    sph = new StockPriceHistoryImpl();
else
    sph = new StockPriceHistoryLogger();

sph.getHighPrice();
```

Suppose thousands of requests execute without logging:

```text
StockPriceHistoryImpl
StockPriceHistoryImpl
StockPriceHistoryImpl
StockPriceHistoryImpl
```

The compiler concludes:

> "The runtime type is always `StockPriceHistoryImpl`."

Based on this observation, it may perform optimizations such as:

- Method inlining
- Direct method dispatch
- Removing unnecessary type checks

Later, a request arrives with:

```text
?log=true
```

Now the runtime object becomes:

```text
StockPriceHistoryLogger
```

The compiler's previous assumption is no longer valid.

The optimized machine code cannot be used anymore, so the JVM marks it as **not entrant**, temporarily falls back to interpretation (or another compiled version), and eventually recompiles the method using the new execution profile.

```text
Old Compiled Code
        │
        ▼
Made Not Entrant
        │
        ▼
Interpreter / Recompile
        │
        ▼
New Optimized Code
```

This is known as a **deoptimization trap**. :contentReference[oaicite:2]{index=2}

---

## Case 2: Tiered Compilation

This is the **most common reason** in modern JVMs.

Normal tiered compilation proceeds as follows:

```text
Level 0 (Interpreter)
        │
        ▼
Level 3 (C1)
        │
        ▼
Level 4 (C2)
```

Initially, a hot method is compiled by the **C1 compiler**.

Later, the **C2 compiler** generates a more optimized version of the same method.

When the C2 version becomes available, the JVM must replace the older C1 version.

Instead of deleting the C1 code immediately, it marks it as **made not entrant**.

```text
C1 Machine Code
        │
        ▼
C2 Machine Code Created
        │
        ▼
C1 Code → Made Not Entrant
        │
        ▼
Future Calls Use C2
```

Although the compilation log reports this as a deoptimization, it actually improves performance because the JVM is replacing slower C1 code with faster C2 code. :contentReference[oaicite:3]{index=3}

---

# 2. Made Zombie

A compiled method becomes **zombie** only after it has already been marked **not entrant**.

By this point:

- No thread is executing the old compiled code.
- The old machine code is no longer needed.

The JVM can safely remove it from the **Code Cache**.

```text
Compiled Code
      │
      ▼
Made Not Entrant
      │
(All executing threads finish)
      │
      ▼
Made Zombie
      │
      ▼
Removed from Code Cache
```

Removing zombie methods frees space in the Code Cache for future compilations. :contentReference[oaicite:4]{index=4}

---

## What Happens If the Method Is Needed Again?

If the method later becomes hot again:

```text
Zombie Code
      │
Method Called Again
      │
      ▼
Interpreter
      │
      ▼
JIT Compilation
      │
      ▼
New Machine Code
```

The JVM simply recompiles the method.

According to the book, these recompilations usually have **no measurable effect** on overall application performance. :contentReference[oaicite:5]{index=5}

---

# Lifecycle of Compiled Code

```text
Interpreter
      │
      ▼
C1 Compilation
      │
      ▼
C2 Compilation
      │
      ▼
Old C1 Code
Made Not Entrant
      │
      ▼
Running Threads Finish
      │
      ▼
Made Zombie
      │
      ▼
Removed from Code Cache
```

---

# Does Deoptimization Hurt Performance?

### When Compiler Assumptions Change

If the compiler's assumptions become invalid, the JVM must:

- Discard the old optimized code.
- Execute interpreted code or another compiled version.
- Gather new profiling information.
- Recompile the method.

This introduces a **temporary performance cost**.

---

### During Tiered Compilation

This is expected behavior.

The JVM replaces C1 code with a more optimized C2 version.

Although the compilation log reports **made not entrant**, performance generally **improves** because future calls execute the faster C2 machine code. :contentReference[oaicite:6]{index=6}

---

# Summary

- **Deoptimization** is the JVM's process of replacing previously compiled code when compiler assumptions change.
- **Made Not Entrant** means the compiled code is no longer used for future calls, although existing executions may continue.
- **Made Zombie** means the old compiled code is no longer executable and is removed from the Code Cache.
- Deoptimization occurs mainly because:
  1. Runtime behavior changes and invalidates compiler assumptions.
  2. Tiered Compilation replaces C1-generated code with newer C2-generated code.
- In modern JVMs, deoptimization during Tiered Compilation is normal and usually results in **better overall performance**, not worse. :contentReference[oaicite:7]{index=7}