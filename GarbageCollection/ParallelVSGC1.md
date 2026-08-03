# Parallel GC vs G1 GC

Both **Parallel GC** and **G1 GC** are production garbage collectors, but they are optimized for different performance goals.

| Feature                     | Parallel GC                      | G1 GC                                    |
| --------------------------- | -------------------------------- | ---------------------------------------- |
| **Primary Goal**            | Maximum throughput               | Predictable low pause times              |
| **Default (Java 17/21/24)** | No                               | Yes                                      |
| **Young GC**                | Parallel, Stop-the-World         | Parallel, Stop-the-World                 |
| **Old GC**                  | Parallel, Stop-the-World Full GC | Mostly concurrent                        |
| **Full GC**                 | More likely                      | Less likely                              |
| **Heap Layout**             | Young + Old generations          | Heap divided into regions                |
| **Heap Compaction**         | During Full GC                   | Incremental during normal operation      |
| **CPU Usage**               | Uses CPU mainly during GC pauses | Uses background CPU continuously         |
| **Pause Times**             | Longer                           | Shorter and more predictable             |
| **Throughput**              | Usually higher                   | Slightly lower (background GC overhead)  |
| **Latency**                 | Higher                           | Lower                                    |
| **Large Heaps**             | Good                             | Excellent                                |
| **Best For**                | Batch jobs, scientific computing | Servers, microservices, web applications |

---

# Parallel GC

## Goal

Maximize the amount of application work completed (throughput).

## How it Works

```text
Application Running
        │
        ▼
Heap Full
        │
        ▼
Stop All Threads
        │
        ▼
Parallel GC Threads
        │
        ▼
Resume Application
```

### Characteristics

- Uses multiple GC threads.
- Stops all application threads during both Minor GC and Full GC.
- Uses all available CPU during garbage collection.
- No background garbage collection.

### Advantages

- Highest throughput.
- Simpler algorithm.
- Lower background overhead.

### Disadvantages

- Long Stop-the-World pauses.
- Poorer latency.
- Full GC pauses can become significant.

---

# G1 GC

## Goal

Minimize application pause times while maintaining good throughput.

## How it Works

```text
Application Running
        │
        ├───────────────► Background GC
        │                     │
        │                     ▼
        │            Concurrent Marking
        │
        ▼
Small Young GC Pauses
        │
        ▼
Continue Application
```

### Characteristics

- Heap divided into regions.
- Parallel Young GC.
- Mostly concurrent Old Generation collection.
- Background GC threads run while the application continues executing.
- Incremental heap compaction reduces fragmentation.

### Advantages

- Much shorter pause times.
- Better latency.
- Less likely to experience long Full GCs.
- Default collector in modern Java.

### Disadvantages

- Background GC threads consume CPU.
- Slightly lower throughput than Parallel GC.
- More complex algorithm.

---

# CPU Usage

## Parallel GC

```text
CPU

100%  ████      ████      ████
       GC        GC        GC

 50%  ─────────────────────────
      Application
```

The application pauses during garbage collection, and GC temporarily uses most of the CPU.

---

## G1 GC

```text
CPU

75%  █████████████████████
     App + Background GC

50%  ─────────────────────
     Application
```

The application continues running while background GC threads consume additional CPU.

---

# When to Use Each

## Use Parallel GC if:

- Maximum throughput is the primary goal.
- Long GC pauses are acceptable.
- Running CPU-intensive batch jobs.
- Full GCs are infrequent.

---

## Use G1 GC if:

- Response time matters.
- Building REST APIs or microservices.
- Running enterprise applications.
- Predictable pause times are important.
- Running general-purpose Java applications.

---

# Rule of Thumb

```text
Batch Processing
        │
        ▼
Parallel GC

----------------------------

Web Servers
Spring Boot
Microservices
REST APIs
        │
        ▼
G1 GC

----------------------------

Ultra-Low Latency
Trading Systems
Real-time Analytics
        │
        ▼
ZGC
```

---

# Performance Trade-off

```text
Highest Throughput
Parallel GC
        │
        ▼
Slightly Lower Throughput
G1 GC
        │
        ▼
Lowest Pause Times
ZGC
```

---

# Quick Summary

- **Parallel GC** prioritizes maximum throughput and uses multiple GC threads during Stop-the-World collections.
- **G1 GC** prioritizes predictable pause times by performing much of the Old Generation collection concurrently.
- **Parallel GC** generally provides higher throughput but longer pauses.
- **G1 GC** generally provides lower latency at the cost of some additional CPU usage.
- **G1 GC** is the default garbage collector in Java 17, Java 21, and Java 24.