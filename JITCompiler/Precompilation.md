# Precompilation

Until now, the chapter has focused on **Just-In-Time (JIT) compilation**, where the JVM compiles methods **while the application is running**.

The drawback of JIT compilation is that applications go through a **warm-up period** before they reach peak performance.

Precompilation attempts to reduce or eliminate this warm-up by compiling code **before the application starts**. :contentReference[oaicite:0]{index=0}

---

# Why Precompilation?

JIT works well for long-running applications, but it is less effective when:

- The application runs for only a short time.
- Startup time is very important.
- Memory is limited (for example, embedded systems).
- The application finishes before the JIT has time to optimize hot methods.

In these situations, compiling code before execution may provide better startup performance. :contentReference[oaicite:1]{index=1}

---

# Types of Precompilation

The book discusses two experimental approaches:

1. **Ahead-of-Time (AOT) Compilation** (Standard JDK 11)
2. **Native Image** (GraalVM)

This section focuses on **Ahead-of-Time (AOT) Compilation**. :contentReference[oaicite:2]{index=2}

---

# Ahead-of-Time (AOT) Compilation

Instead of waiting for the JVM to compile methods at runtime, **Ahead-of-Time Compilation** compiles selected classes or modules **before the application starts**.

The compiled machine code is stored in a **shared library**.

When the application starts, the JVM loads this shared library and executes the precompiled code immediately.

```text
Traditional JIT

Java Classes
      │
      ▼
JVM Starts
      │
      ▼
Interpreter
      │
      ▼
JIT Compilation
      │
      ▼
Machine Code
```

```text
Ahead-of-Time Compilation

Java Classes
      │
      ▼
jaotc
      │
      ▼
Shared Library (.so/.dll)
      │
      ▼
JVM Starts
      │
      ▼
Machine Code Immediately
```

The goal is to reduce the application's warm-up period. :contentReference[oaicite:3]{index=3}

---

# Does AOT Always Improve Startup?

**No.**

Loading the shared library itself takes time.

If the application is very small (for example, **Hello World**), the time required to load the library may actually make startup **slower**.

AOT is intended for applications with longer startup times, such as REST servers, where the cost of loading the shared library is offset by the reduced JIT work. :contentReference[oaicite:4]{index=4}

---

# Creating an AOT Library

The JDK provides the **`jaotc`** tool to create an AOT shared library.

Example:

```bash
jaotc \
  --compile-commands=/tmp/methods.txt \
  --output JavaBaseFilteredMethods.so \
  --compile-for-tiered \
  --module java.base
```

This command:

- Reads the methods to compile from `methods.txt`.
- Compiles selected methods from the `java.base` module.
- Produces a shared library (`JavaBaseFilteredMethods.so`).
- Keeps the methods eligible for later C2 compilation using `--compile-for-tiered`. :contentReference[oaicite:5]{index=5}

---

# Why Not Compile Everything?

Although AOT can compile an entire module, the book recommends compiling **only selected methods**.

Compiling everything:

- Creates a much larger shared library.
- Increases library loading time.
- May reduce startup benefits.

Instead, compile only the methods that the application frequently uses. :contentReference[oaicite:6]{index=6}

---

# Selecting Methods to Compile

The file `methods.txt` contains the methods that should be compiled.

Example:

```text
compileOnly java.net.URI.getHost()Ljava/lang/String;
```

Only the listed methods are included in the shared library.

This keeps the library smaller and faster to load. :contentReference[oaicite:7]{index=7}

---

# Finding Frequently Used Methods

To determine which methods the application actually uses, run the application with:

```bash
java \
-XX:+UnlockDiagnosticVMOptions \
-XX:+LogTouchedMethods \
-XX:+PrintTouchedMethodsAtExit
```

When the application exits, the JVM prints every method that was executed.

Example output:

```text
java/net/URI.getHost:()Ljava/lang/String;
```

These methods can then be converted into entries for `methods.txt`. :contentReference[oaicite:8]{index=8}

---

# `--compile-for-tiered`

AOT compilation generates code similar to **C1-compiled code**.

For long-running applications, this is not the highest optimization level.

Using:

```text
--compile-for-tiered
```

allows the precompiled methods to **remain eligible for later C2 compilation**.

```text
AOT (C1-like)
       │
       ▼
Application Starts
       │
       ▼
Later
       │
       ▼
C2 Compilation
```

If this option is omitted, long-running applications may never benefit from C2's advanced optimizations. :contentReference[oaicite:9]{index=9}

---

# Interaction with Tiered Compilation

When Tiered Compilation is enabled, AOT methods are treated as another compilation level.

Eventually, the JIT recompiles hot methods with the C2 compiler.

The older AOT version is then marked **made not entrant**, just like C1 code is replaced by C2 code. :contentReference[oaicite:10]{index=10}

---

# Using the AOT Library

Run the application with:

```bash
java \
-XX:AOTLibrary=/path/to/JavaBaseFilteredMethods.so
```

The JVM loads the shared library during startup and executes the precompiled methods. :contentReference[oaicite:11]{index=11}

---

# Viewing AOT Activity

To verify that the JVM is using the shared library:

```text
-XX:+PrintAOT
```

Example output:

```text
373 105 aot[1] java.util.HashSet.<init>(I)V
```

This output indicates:

- **373** → Milliseconds since JVM startup.
- **105** → Method ID.
- **aot[1]** → The method was loaded from AOT library number 1.
- **java.util.HashSet.<init>(I)V** → The precompiled method being executed. :contentReference[oaicite:12]{index=12}

---

# Modern Status

The book discusses **AOT Compilation as an experimental feature in JDK 11**.

Today:

- The original **`jaotc` AOT compiler is no longer part of modern OpenJDK releases (such as JDK 17 and JDK 21).**
- For applications that need native startup performance, **GraalVM Native Image** has become the primary solution.
- Standard HotSpot JVMs continue to rely mainly on **Tiered JIT Compilation**.

---

# Summary

- Precompilation attempts to reduce the JVM's warm-up period by compiling code before execution.
- **Ahead-of-Time (AOT) Compilation** creates a shared library containing precompiled machine code.
- The `jaotc` tool generates the shared library from selected modules or classes.
- AOT is most beneficial for applications with long startup times, such as servers.
- The `--compile-for-tiered` option allows precompiled methods to be further optimized by the C2 compiler.
- In modern Java (JDK 17/21+), the original `jaotc` feature has largely been replaced by **GraalVM Native Image**, while the standard HotSpot JVM continues to use Tiered JIT Compilation. :contentReference[oaicite:13]{index=13}