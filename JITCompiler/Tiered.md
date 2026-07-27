## Tiered Compilation Today

In modern Java (JDK 17, JDK 21, and later), there is no separate **client JVM** or **server JVM**. Instead, the HotSpot JVM includes both the **C1** and **C2** compilers and uses **Tiered Compilation** by default.

### What Happened to the Old Compiler Flags?

In older JDK versions, the following flags were used to select the compiler:

- `-client` – Use the client compiler.
- `-server` – Use the server compiler.
- `-d64` – Alias for `-server`.

Today:

| Flag      | Status                                   |
| --------- | ---------------------------------------- |
| `-client` | Ignored (no effect)                      |
| `-server` | Ignored (server behavior is the default) |
| `-d64`    | Removed in JDK 11 (causes an error)      |

### Do C1 and C2 Still Exist?

Yes. Both compilers are still part of the HotSpot JVM.

- **C1 (Compiler 1)** focuses on fast compilation and improves application startup time.
- **C2 (Compiler 2)** performs more aggressive optimizations to improve long-running application performance.

The JVM automatically decides when to use each compiler.

### How Tiered Compilation Works

```text
Program Starts
      │
      ▼
 Interpreter
      │
      ▼
 Method becomes hot
      │
      ▼
 C1 Compiler
(Fast Compilation)
      │
      ▼
 Method becomes hotter
      │
      ▼
 C2 Compiler
(Aggressive Optimizations)
      │
      ▼
 Optimized Native Code
```

This approach provides:

- Faster application startup through **C1**.
- Better peak performance through **C2**.

### Disabling Tiered Compilation

Tiered Compilation can be disabled using:

```text
-XX:-TieredCompilation
```

However, this is rarely needed. It is mainly used for:

- Performance experiments
- Benchmarking
- JVM research
- Diagnosing compiler-related issues

For most production applications, Tiered Compilation should remain enabled because it provides the best balance between startup performance and long-term execution speed.