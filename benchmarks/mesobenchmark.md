# Macrobenchmark

This section explains **why you should benchmark the complete application instead of individual methods or modules**. This approach is called a **macrobenchmark**.

We'll use realistic Spring Boot backend examples to understand the concept.

---

# What is a Macrobenchmark?

A **macrobenchmark** measures the performance of the **entire system** as it runs in production.

For a typical Spring Boot application, this means benchmarking the complete request lifecycle:

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

Consider the following service:

```java
public Order placeOrder(OrderRequest request)
```

Internally, it performs the following steps:

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

Suppose you benchmark only:

```java
calculatePrice();
```

Execution time:

```
2 ms
```

After optimization:

```
1 ms
```

At first glance, you've made the method **2× faster**.

However, what matters is the performance of the **entire request**.

---

# Real Request Timing

| Step | Time |
|------|------:|
| Authentication | 20 ms |
| Database Read | 80 ms |
| Price Calculation | 2 ms |
| Database Write | 90 ms |
| JSON Response | 8 ms |
| **Total** | **200 ms** |

After optimizing the pricing logic:

| Step | Time |
|------|------:|
| Authentication | 20 ms |
| Database Read | 80 ms |
| Price Calculation | 1 ms |
| Database Write | 90 ms |
| JSON Response | 8 ms |
| **Total** | **199 ms** |

Although the pricing method became twice as fast, the overall request improved by only **1 ms**.

Since the database dominates the request time, users are unlikely to notice the improvement.

> **Lesson:** Optimizing one module doesn't necessarily improve the performance of the entire application.

---

# The Pipeline Analogy

Think of your application as a water pipeline.

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

| Module | Capacity |
|---------|---------:|
| Authentication | 500 RPS |
| Business Logic | 200 RPS |
| Database | 100 RPS |
| Response | 300 RPS |

> **RPS = Requests Per Second**

The bottleneck is the database.

```
Database
100 RPS
```

Therefore, the entire application can process at most:

```
100 Requests/Second
```

The slowest component determines the overall throughput.

---

# What Happens If We Optimize the Wrong Component?

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

The pipeline now looks like this:

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

The application's maximum throughput is still:

```
100 Requests/Second
```

The database remains the bottleneck.

> Improving a component that is **not** the bottleneck often has little or no impact on overall system performance.

---

# Why Mocking Can Be Misleading

During unit testing, you might replace the database with a fake implementation.

```text
Spring Boot
      │
      ▼
Fake Database
```

The benchmark may appear extremely fast.

In production, however, the request path is much more complex.

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
Disk
      │
      ▼
Locks
      │
      ▼
Buffers
```

A benchmark using mocks ignores important real-world costs such as:

- Network latency
- JDBC overhead
- Database buffers
- Connection pooling
- Lock contention
- Serialization and deserialization

For realistic macrobenchmarks, benchmark against real external services whenever possible.

---

# Multiple JVMs on One Machine

Suppose your server has:

```
8 CPU Cores
```

Initially, it runs one application.

```text
JVM A
Spring Boot API
```

Everything performs well.

Now you deploy additional services.

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

It attempts to use all CPU cores.

At the same time:

- JVM B is processing requests.
- JVM C is running analytics.

All three JVMs compete for CPU time.

Instead of receiving:

```
100% CPU
```

JVM A might receive only:

```
40% CPU
```

As a result:

- Garbage collection takes longer.
- Request latency increases.
- Performance differs from isolated benchmark results.

This is why benchmarking a single JVM on an otherwise idle machine often fails to represent production behavior.

---

# Interview Takeaway

**Question:** Why is macrobenchmarking preferred over microbenchmarking?

**Answer:**

> A macrobenchmark measures the performance of the complete application under realistic conditions, including databases, networks, authentication services, caches, and other external dependencies. It identifies the actual bottlenecks and measures overall throughput and latency. Microbenchmarks are useful for comparing small code fragments, but they may not accurately represent production performance because of JIT optimizations, garbage collection, and interactions with the rest of the application.

---

# Key Takeaways

- A **macrobenchmark** measures the performance of the **entire application**, not just individual methods.
- The **slowest component (the bottleneck)** determines the maximum throughput of the system.
- Optimizing a component that is **not** the bottleneck often has little impact on user-perceived performance.
- Avoid excessive mocking when evaluating performance; include real databases, networks, and external services whenever practical.
- Benchmark under **production-like conditions**, including multiple JVMs or containers if that reflects the actual deployment environment.