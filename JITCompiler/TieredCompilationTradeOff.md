# Tiered Compilation Trade-offs

**Tiered Compilation** is enabled by default because it provides the best balance between **fast startup** and **high peak performance**.

However, there are situations where disabling Tiered Compilation may be beneficial, particularly in **memory-constrained environments**. :contentReference[oaicite:0]{index=0}

---

# Why Would You Disable Tiered Compilation?

Most modern systems have plenty of memory, but some environments do not.

Examples include:

- Docker containers with small memory limits.
- Small cloud virtual machines.
- Systems running many JVMs simultaneously.

In these cases, reducing the JVM's memory footprint may be more important than achieving the fastest startup time. :contentReference[oaicite:1]{index=1}

---

# Effect on the Code Cache

Tiered Compilation uses **both the C1 and C2 compilers**.

Because C1 compiles many methods during application startup, it stores significantly more compiled machine code in the **Code Cache**.

The book compares starting NetBeans with Tiered Compilation enabled and disabled:

| Compiler Mode          | Classes Compiled | Code Cache | Startup Time |
| ---------------------- | ---------------: | ---------: | -----------: |
| **+TieredCompilation** |           22,733 |    46.5 MB |       50.1 s |
| **-TieredCompilation** |            5,609 |    10.7 MB |       68.5 s |

Observations:

- Tiered Compilation compiled about **4× more classes**.
- It required about **4× more Code Cache memory**.
- Startup was approximately **18 seconds faster**.

Although the Code Cache used much more memory, the startup time improved significantly. :contentReference[oaicite:2]{index=2}

---

# Trade-off

## Tiered Compilation Enabled

### Advantages

- Faster application startup.
- Earlier compilation of hot methods.
- Better responsiveness during warm-up.
- Slightly better peak performance.

### Disadvantages

- Larger Code Cache.
- Higher memory usage.

```text
Interpreter
      │
      ▼
C1 (Many Methods)
      │
      ▼
C2
      │
      ▼
Large Code Cache
Fast Startup
```

---

## Tiered Compilation Disabled

### Advantages

- Smaller Code Cache.
- Lower memory usage.

### Disadvantages

- Slower startup.
- Fewer methods compiled.
- Longer warm-up period.

```text
Interpreter
      │
      ▼
C2 Only
      │
      ▼
Small Code Cache
Slow Startup
```

---

# Long-Running Applications

The book also compares a server application after different warm-up periods.

| Warm-up | Tiered Disabled | Tiered Enabled |
| ------: | --------------: | -------------: |
|     0 s |       23.72 OPS |      24.23 OPS |
|    60 s |       23.73 OPS |      24.26 OPS |
|   300 s |       24.42 OPS |      24.43 OPS |

The results show:

- During startup, Tiered Compilation performs better because methods are compiled sooner.
- After a long warm-up period, **both configurations achieve almost the same throughput**.
- Tiered Compilation still maintains a **very small advantage** because some methods remain compiled by C1 that would never be compiled by C2 alone. :contentReference[oaicite:3]{index=3}

---

# Modern Recommendation

For **almost all applications**, keep Tiered Compilation **enabled**.

Disable it only when:

- Memory is extremely limited.
- Code Cache size is a more important concern than startup time.
- You are running many JVMs on the same machine and need to reduce memory usage.

Otherwise, the JVM's default configuration provides the best balance between startup performance, memory usage, and peak execution performance. :contentReference[oaicite:4]{index=4}

---

# The `javac` Compiler

The **JIT compiler** has the greatest impact on runtime performance.

The `javac` compiler simply translates Java source code into **bytecode** before the program runs.

In general:

- Using the **`-g`** option to include debugging information does **not** affect runtime performance.
- Using the **`final`** keyword does **not** make the generated code faster.
- Recompiling an application with a newer version of **`javac`** usually does **not** improve performance. :contentReference[oaicite:5]{index=5}

---

# Exception in JDK 11

One notable exception is **JDK 11**.

JDK 11 introduced a new implementation of **string concatenation**.

To benefit from this improvement, applications must be **recompiled** with the JDK 11 `javac` compiler.

Apart from this exception, newer versions of `javac` generally do not require applications to be recompiled to gain runtime performance improvements. :contentReference[oaicite:6]{index=6}

---

# Summary

- Tiered Compilation improves startup time by compiling many methods early with the C1 compiler.
- The trade-off is increased Code Cache memory usage.
- Disabling Tiered Compilation reduces memory usage but increases startup time.
- After sufficient warm-up, applications with and without Tiered Compilation achieve nearly the same peak throughput.
- For most production applications, **Tiered Compilation should remain enabled**.
- The `javac` compiler generally has little impact on runtime performance; the primary exception is the string concatenation improvements introduced in JDK 11. :contentReference[oaicite:7]{index=7}