# Macrobenchmark

**Date:** 14 July 2026
**Time:** 23:50

---

# What is a Macrobenchmark?

A **macrobenchmark** measures the performance of the **entire application** under realistic production conditions.

Unlike a microbenchmark, which focuses on an individual method or algorithm, a macrobenchmark evaluates the complete request lifecycle.

For a typical Spring Boot application, this includes:

```text
User
   │
   ▼
Spring Boot
   │
   ▼
Authentication (LDAP/OAuth)
   │
   ▼
Business Logic
   │
   ▼
Database
   │
   ▼
Redis
   │
   ▼
Response
```

Every component participates in the benchmark.

---

# Why Not Benchmark Only the Business Logic?

Consider the following Spring Boot service:

```java
public Order placeOrder(OrderRequest request)
```

Internally, it performs these steps:

```text
Receive Request
      │
      ▼
Authenticate User
      │
      ▼
Validate Order
      │
      ▼
Read Product from Database
      │
      ▼
Calculate Price
      │
      ▼
Save Order
      │
      ▼
Return Response
```

Suppose you benchmark only the pricing logic:

```java
calculatePrice();
```

Initially, it takes:

```
2 ms
```

After optimization:

```
1 ms
```

It appears that the method is now **2× faster**.

However, what matters is the performance of the **entire request**.

---

# Real Request Timing

| Step               |       Time |
| ------------------ | ---------: |
| Authentication     |      20 ms |
| Database Read      |      80 ms |
| Price Calculation  |       2 ms |
| Database Write     |      90 ms |
| JSON Serialization |       8 ms |
| **Total**          | **200 ms** |

After optimizing `calculatePrice()`:

| Step               |       Time |
| ------------------ | ---------: |
| Authentication     |      20 ms |
| Database Read      |      80 ms |
| Price Calculation  |       1 ms |
| Database Write     |      90 ms |
| JSON Serialization |       8 ms |
| **Total**          | **199 ms** |

Although the pricing algorithm became twice as fast, the overall request improved by only **1 ms**.

Most users would never notice this improvement because the **database dominates the request time**.

**Key idea:** Optimizing one module does not necessarily improve overall application performance.

---

# The Pipeline Analogy

Think of your application as a pipeline.

```text
User
   │
   ▼
Authentication
   │
   ▼
Business Logic
   │
   ▼
Database
   │
   ▼
Response
```

Each stage has a maximum throughput.

| Module         | Capacity (RPS) |
| -------------- | -------------: |
| Authentication |            500 |
| Business Logic |            200 |
| Database       |            100 |
| Response       |            300 |

> **RPS = Requests Per Second**

The bottleneck is the **Database**, which can handle only **100 requests per second**.

Therefore, the entire application is limited to:

```
100 Requests/Second
```

---

# Optimizing the Wrong Component

Suppose you optimize the business logic.

Before:

```
Business Logic
200 RPS
```

After:

```
Business Logic
400 RPS
```

The pipeline becomes:

```text
Authentication
500
   │
   ▼
Business Logic
400
   │
   ▼
Database
100
   │
   ▼
Response
300
```

The application's throughput remains:

```
100 Requests/Second
```

The database is still the bottleneck.

**Lesson:** Improving a component that is not the bottleneck often produces little or no noticeable improvement.

---

# Why Mocking Can Be Misleading

During unit testing, the database is often replaced with a mock or fake implementation.

```text
Spring Boot
      │
      ▼
Fake Database
```

The benchmark may appear excellent.

In production, however, the request path looks more like this:

```text
Spring Boot
      │
      ▼
PostgreSQL
      │
      ▼
Network
      │
      ▼
Disk I/O
      │
      ▼
Locks
      │
      ▼
Buffers
```

The production system includes additional overhead such as:

* Network latency
* JDBC overhead
* Connection pooling
* Database buffers
* Lock contention
* Serialization and deserialization

A benchmark using mocks ignores these costs and may not represent real-world performance.

For accurate macrobenchmarks, include real external services whenever practical.

---

# Multiple JVMs on One Machine

Consider a server with:

```
8 CPU Cores
```

Initially, it runs one application:

```text
JVM A
Spring Boot API
```

Everything performs well.

Now you deploy additional services:

```text
Server (8 CPUs)

├── JVM A (Spring Boot API)
├── JVM B (Background Worker)
└── JVM C (Analytics Service)
```

Each JVM assumes it has access to all available CPU resources.

---

# Garbage Collection Contention

Suppose JVM A starts a major garbage collection.

It attempts to utilize all CPU cores.

At the same time:

* JVM B is processing background jobs.
* JVM C is running analytics.
* The operating system must share CPU resources.

Instead of using:

```
100% CPU
```

JVM A may receive only:

```
40% CPU
```

As a result:

* Garbage collection takes longer.
* Request latency increases.
* Performance differs from isolated benchmark results.

This is why benchmarking a single JVM on an otherwise idle machine often fails to represent production behavior.

---

# Interview Takeaway

**Question:** Why is macrobenchmarking preferred over microbenchmarking?

**Answer:**

> A macrobenchmark measures the performance of the complete application under realistic conditions, including databases, networks, authentication services, caches, and other external dependencies. It identifies the true bottlenecks and measures overall throughput and latency. Microbenchmarks are valuable for comparing small pieces of code, but they may not reflect real production performance because of JIT compilation, garbage collection, caching, and interactions with the rest of the system.

---

# Key Takeaways

* A **macrobenchmark** measures the performance of the entire application rather than individual methods.
* The **slowest component (the bottleneck)** determines the maximum throughput of the system.
* Optimizing a component that is **not** the bottleneck often has little impact on user-perceived performance.
* Excessive mocking can produce misleading benchmark results because it excludes real-world costs such as network latency, databases, and serialization.
* Benchmark under **production-like conditions**, including real infrastructure and multiple JVMs or containers if that matches the deployment environment.
