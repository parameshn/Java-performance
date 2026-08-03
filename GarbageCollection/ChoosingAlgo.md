# Choosing a GC Algorithm

The choice of a garbage collector depends on three main factors:

1. **Available hardware (especially CPU cores)**
2. **Application characteristics**
3. **Performance goals**

The book recommends starting with **G1 GC** as the default choice in JDK 11, but emphasizes that there are exceptions. The main trade-off is between the CPU required by G1's background threads and the application's own CPU requirements.

---

# When to Use the Serial Collector

The **Serial GC** is a good choice when:

- The machine has a single CPU.
- A Docker container or VM is limited to one CPU.
- The application is CPU-bound.
- The heap is relatively small.

## Why?

Serial GC uses only one GC thread.

With only one CPU available, running multiple GC threads provides no benefit and instead adds overhead.

The book shows that for a CPU-intensive batch job on a single CPU, the Serial collector outperformed both Parallel GC and G1 GC because it spent less time performing garbage collection. G1 GC also lost performance because its background threads competed with the application for the only available CPU.

---

# Single Hyper-Threaded CPU

Even if a machine appears to have two logical CPUs because of hyper-threading, the Serial collector can still outperform the other collectors for CPU-bound workloads.

The JVM may default to:

- **Parallel GC** (JDK 8)
- **G1 GC** (JDK 11)

but the book notes that Serial GC can still be the better choice in this situation.

---

# When to Use the Parallel (Throughput) Collector

The **Parallel GC** is intended for maximizing overall throughput.

It performs best when:

- Multiple physical CPU cores are available.
- The application is CPU-bound.
- Few or no Full GCs occur.
- G1's background threads would otherwise consume valuable CPU time.

The book shows examples where the Parallel collector outperformed G1 when:

- Full GCs were rare.
- The Old Generation stayed mostly full, causing G1's concurrent threads to perform more work.
- CPU resources were already fully utilized, leaving little room for G1's background threads.

---

# When to Use G1 GC

G1 GC is generally the preferred collector for most applications.

It performs well when:

- Multiple CPU cores are available.
- The application is latency-sensitive.
- Avoiding long Full GC pauses is important.
- Background GC threads have sufficient CPU resources.

Because G1 performs most Old Generation work concurrently, it can greatly reduce long application pauses.

For interactive applications such as REST services, the book shows that G1 often provides much better response times by avoiding Full GCs.

---

# CPU Usage Patterns

## Parallel GC

Most of the time, only application threads are running.

When a garbage collection occurs:

- All application threads stop.
- GC uses nearly all available CPU.

```text
CPU

100%  ████      ████      ████
       GC        GC        GC

 50%  ─────────────────────────
      Application
```

This produces short spikes of high CPU usage.

---

## G1 GC

Application threads continue running while background GC threads execute concurrently.

```text
CPU

75%  █████████████████████
     App + Background GC

50%  ─────────────────────
     Application
```

Instead of short spikes, G1 produces longer periods of moderate CPU utilization because application and GC threads execute simultaneously.

---

# Rule of Thumb (Book)

| Situation                             | Recommended GC |
| ------------------------------------- | -------------- |
| Single CPU / small container          | Serial GC      |
| Multi-core CPU-bound batch jobs       | Parallel GC    |
| General-purpose applications          | G1 GC          |
| Interactive services (REST, Web APIs) | G1 GC          |
| Low-latency applications              | G1 GC          |

---

# Quick Summary

- **G1 GC** is generally the best choice for most applications.
- **Serial GC** is effective for CPU-bound workloads running on a single CPU, including hyper-threaded single-core systems.
- **Parallel GC** is a good choice for CPU-bound applications on multi-core machines.
- G1 GC is usually preferable when minimizing long pauses is more important than maximizing raw throughput.

---

# Modern Note

For **Java 17/21/24**, this guidance is still largely valid:

- **G1 GC** remains the default and is the best starting point for most applications.
- **Parallel GC** remains an excellent choice for throughput-oriented batch workloads.
- **Serial GC** is primarily used for very small heaps, embedded systems, or single-core/containerized environments.
- For applications requiring **ultra-low latency**, **ZGC** (and **Shenandoah**, where available) are now production-ready alternatives that were experimental when the book was written.