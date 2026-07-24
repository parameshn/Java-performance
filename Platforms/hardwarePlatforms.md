# Chapter 1 – Hardware Platforms

Hardware plays a fundamental role in Java performance.

Even if your code is well-written, the **CPU architecture**, **number of cores**, and **deployment environment** (Virtual Machine or Docker container) significantly influence how the JVM performs.

Understanding the hardware underneath the JVM helps you make better decisions about thread pools, garbage collection, and application scalability.

---

# 1. Multicore Processors

Modern CPUs contain multiple processing cores.

Typical examples:

| Device | Physical Cores |
|---------|---------------:|
| Older Desktop | 1 |
| Modern Laptop | 4–16 |
| Enterprise Server | 32–128+ |

Each core can execute instructions independently.

### Example

| CPU | Maximum Parallel CPU-Bound Tasks |
|-----|---------------------------------:|
| 1 Core | 1 |
| 2 Cores | 2 |
| 4 Cores | 4 |
| 8 Cores | 8 |

For CPU-intensive workloads, increasing the number of **physical cores** generally increases throughput.

---

# 2. Hyper-Threading (Simultaneous Multithreading)

Many Intel processors support **Hyper-Threading**, while AMD refers to the same concept as **Simultaneous Multithreading (SMT)**.

Each physical core appears as **two logical CPUs** to the operating system.

### Example

| Physical Cores | Logical CPUs |
|---------------:|-------------:|
| 4 | 8 |
| 6 | 12 |
| 8 | 16 |

From the JVM's perspective, these logical CPUs are available for scheduling threads.

---

# 3. Hyper-Threading Does Not Double Performance

A common misconception is that Hyper-Threading doubles CPU performance.

Suppose you have:

```
4 Physical Cores
```

Without Hyper-Threading:

```
4 Logical CPUs
```

With Hyper-Threading:

```
8 Logical CPUs
```

You might expect twice the performance, but that is **not** how Hyper-Threading works.

### Why?

A physical core can execute only one instruction stream at a time.

When one thread is stalled (for example, waiting for data from memory), the processor can switch to another ready thread instead of leaving the core idle.

```text
Thread A Waiting
        │
        ▼
Core Executes Thread B
```

Hyper-Threading improves **CPU utilization**, not the number of physical execution units.

---

# 4. Performance Scaling Example

Consider a machine with **4 physical CPU cores**.

### Running 4 CPU-Bound Tasks

```text
Task 1 → Core 1
Task 2 → Core 2
Task 3 → Core 3
Task 4 → Core 4
```

Each task receives its own dedicated core.

Performance approaches:

```
≈ 4×
```

compared to running on a single core.

---

### Running 5 CPU-Bound Tasks

Now every physical core is already occupied.

The fifth task runs only when another thread stalls.

Result:

```
≈ 4.3×
```

instead of:

```
5×
```

Adding more tasks continues to produce **diminishing returns**.

Typical scaling:

```
4 Physical Cores
        +
Hyper-Threading
        │
        ▼
≈ 5–6× Performance
```

not the **8×** many people expect from eight logical CPUs.

---

# 5. Why Java Developers Should Care

Many JVM components automatically use multiple threads.

Examples include:

- Garbage Collection (GC)
- Just-In-Time (JIT) Compilation
- Parallel Streams
- ForkJoinPool
- ExecutorService

The JVM sizes many internal thread pools based on the available CPU count.

Knowing the difference between **physical cores** and **logical CPUs** helps you:

- Configure thread pools appropriately.
- Understand CPU utilization.
- Interpret benchmark results.
- Avoid excessive thread creation.

---

# 6. Virtual Machines (VMs)

A Virtual Machine provides an isolated operating system with dedicated virtual hardware.

Suppose a physical server has:

- 128 CPU cores
- 512 GB RAM

Your virtual machine may receive only:

- 2 CPU cores
- 16 GB RAM

```text
Physical Server
├── 128 CPU Cores
├── 512 GB RAM
│
└── Virtual Machine
      ├── 2 CPU Cores
      └── 16 GB RAM
```

From the JVM's perspective, the VM behaves like an independent computer.

JVM tuning should therefore be based on the **resources assigned to the VM**, not the physical host.

---

# 7. Docker Containers

Docker differs from a Virtual Machine.

A Docker container:

- Shares the host operating system.
- Runs as an isolated process.
- Can have CPU and memory limits.

```text
Host Operating System
        │
        ├── Docker Container A
        ├── Docker Container B
        └── Docker Container C
```

Today, Java applications are commonly deployed in Docker, especially in cloud environments.

---

# 8. Java 8 vs Java 11 in Docker

Older Java 8 releases (before **Java 8u192**) were **not container-aware**.

Suppose:

| Host Machine | Docker Container |
|--------------|-----------------|
| 16 CPU Cores | 2 CPU Cores |
| 64 GB RAM | 8 GB RAM |

Older JVMs assumed they could use all:

- 16 CPU cores
- 64 GB RAM

This often resulted in:

- Too many GC threads
- Oversized heap allocation
- Container crashes caused by exceeding memory limits

---

## Modern JVM Behavior

Starting with:

- Java 8u192
- Java 11
- Java 17
- Java 21

the JVM became **container-aware**.

It automatically detects Docker resource limits and adjusts:

- Heap size
- Garbage collection threads
- JIT compiler threads
- Internal JVM settings

This allows Java applications to behave correctly inside containers without extensive manual tuning.

---

# Summary

```text
Hardware
│
├── Physical CPU Cores
│      └── Determines maximum parallel execution
│
├── Hyper-Threading (SMT)
│      └── Improves utilization, not double performance
│
├── Virtual Machines
│      └── JVM sees allocated VM resources
│
└── Docker Containers
       └── JVM should respect container CPU and memory limits
```

---

# Key Takeaways

- **Physical CPU cores** are the primary factor determining CPU performance.
- **Hyper-Threading (SMT)** improves CPU utilization but does **not** double throughput.
- The JVM automatically configures many internal thread pools using the available CPU count.
- A **Virtual Machine** behaves like an independent machine from the JVM's perspective.
- Docker containers require a **container-aware JVM** to correctly detect CPU and memory limits.
- Modern JDKs (Java 17 and Java 21) automatically adapt to containerized environments and should be preferred for production deployments.