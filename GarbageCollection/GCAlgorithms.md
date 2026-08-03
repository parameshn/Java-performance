# GC Algorithms (Modern Java 17/21/24)

OpenJDK provides several garbage collectors, each optimized for different performance goals such as throughput, pause time, latency, and memory footprint.

The status of the major collectors in modern Java is:

| GC Algorithm             | Java 17                                    | Java 21                                | Java 24                  |
| ------------------------ | ------------------------------------------ | -------------------------------------- | ------------------------ |
| Serial GC                | Supported                                  | Supported                              | Supported                |
| Parallel (Throughput) GC | Supported                                  | Supported                              | Supported                |
| G1 GC                    | Supported (Default)                        | Supported (Default)                    | Supported (Default)      |
| CMS                      | Removed                                    | Removed                                | Removed                  |
| ZGC                      | Supported                                  | Supported (Generational ZGC available) | Supported                |
| Shenandoah               | Supported (OpenJDK builds that include it) | Supported                              | Supported                |
| Epsilon GC               | Supported (Experimental)                   | Supported (Experimental)               | Supported (Experimental) |

---

# 1. Serial GC

**Goal:** Simplicity and low overhead.

Suitable for:

- Small heaps
- Single CPU/core environments
- Small containers
- Embedded systems

Characteristics:

- Single GC thread
- Stop-the-World for Minor and Full GC
- Fully compacts the heap

JVM Flag:

```text
-XX:+UseSerialGC
```

---

# 2. Parallel (Throughput) GC

**Goal:** Maximum application throughput.

Characteristics:

- Multiple GC threads
- Parallel Young GC
- Parallel Full GC
- Stop-the-World during collections
- High throughput
- Longer pauses than concurrent collectors

JVM Flag:

```text
-XX:+UseParallelGC
```

Best suited for:

- Batch jobs
- Scientific computing
- CPU-intensive workloads

---

# 3. G1 GC (Garbage-First)

**Default collector** in Java 17, Java 21, and Java 24.

Goal:

- Predictable pause times
- Good throughput
- Large heap support

Characteristics:

- Heap divided into regions
- Young and Old generations
- Parallel Young GC
- Mostly concurrent Old GC
- Incremental compaction
- Reduced Full GC frequency

JVM Flag:

```text
-XX:+UseG1GC
```

Best suited for:

- Spring Boot
- Microservices
- Enterprise applications
- General-purpose servers

---

# 4. ZGC

**Goal:** Extremely low pause times.

Characteristics:

- Concurrent collection
- Concurrent compaction
- Very short Stop-the-World pauses (typically under a few milliseconds)
- Designed for very large heaps

JVM Flag:

```text
-XX:+UseZGC
```

Java 21 introduced **Generational ZGC**, improving throughput while maintaining low pause times.

Best suited for:

- Trading systems
- Real-time analytics
- Large in-memory services
- Latency-sensitive applications

---

# 5. Shenandoah

Goal:

- Low pause times
- Concurrent heap compaction

Characteristics:

- Concurrent marking
- Concurrent evacuation
- Concurrent compaction
- Small pause times regardless of heap size

JVM Flag:

```text
-XX:+UseShenandoahGC
```

Available in supported OpenJDK builds.

---

# 6. Epsilon GC

Unlike other collectors:

- Performs **no garbage collection**
- Only allocates memory

When the heap becomes full:

```text
OutOfMemoryError
```

Useful for:

- Performance testing
- Benchmarking
- Short-lived applications
- GC research

JVM Flag:

```text
-XX:+UseEpsilonGC
```

---

# CMS (Removed)

The Concurrent Mark-Sweep (CMS) collector:

- Deprecated in Java 9.
- Removed starting in Java 14.

Its major limitation was that it could not efficiently compact the heap concurrently, leading to fragmentation.

G1 GC became its recommended replacement.

---

# Explicit Garbage Collection

Applications can explicitly request garbage collection:

```java
System.gc();
```

This is generally discouraged because:

- It may trigger an expensive Full GC (or initiate a major collection cycle depending on the collector).
- It interrupts the JVM's own GC scheduling.
- It rarely improves overall performance.

Typical exceptions include:

- Benchmarking
- Heap dump analysis
- Performance diagnostics

To ignore explicit GC requests:

```text
-XX:+DisableExplicitGC
```

---

# Choosing a Collector

| Collector  | Best For                                   |
| ---------- | ------------------------------------------ |
| Serial     | Small applications, embedded systems       |
| Parallel   | Maximum throughput                         |
| G1         | General-purpose applications (default)     |
| ZGC        | Extremely low latency and very large heaps |
| Shenandoah | Low latency with concurrent compaction     |
| Epsilon    | Testing and benchmarking only              |

---

# Modern Defaults

- **Java 17/21/24 default:** **G1 GC**
- **CMS:** Removed
- **Generational ZGC:** Production-ready and available in modern JDKs
- **Serial**, **Parallel**, **G1**, and **ZGC** are all production-supported collectors.