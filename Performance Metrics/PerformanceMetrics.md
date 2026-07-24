# Performance Metrics

The author explains that application performance can be measured in **three different ways**, depending on what you're trying to evaluate:

1. **Elapsed Time (Batch Time)**
2. **Throughput**
3. **Response Time (Latency)**

Each metric answers a different question and is useful in different scenarios.

---

# 1. Elapsed Time (Batch Time)

**Question:**

> **How long does one complete job take?**

Elapsed time measures the total duration of a single task from start to finish.

## Example

Generate payroll for **50,000 employees**.

```text
Start
  │
  ▼
Generate Payroll
  │
  ▼
Stop
```

**Result**

```
18 seconds
```

This is known as **batch performance**.

### Common Examples

- Payroll generation
- PDF generation
- Video encoding
- Data migration
- ETL (Extract, Transform, Load) jobs
- Backup and restore operations

---

# 2. Throughput

**Question:**

> **How much work can the system complete per unit of time?**

Throughput measures the overall capacity of a system.

## Example

```text
1000 Users
     │
     ▼
Server
     │
     ▼
900 Requests/Second
```

Common units include:

- Requests Per Second (RPS)
- Transactions Per Second (TPS)
- Operations Per Second (OPS)

### Example

Suppose each request takes:

```
200 ms
```

Many clients continuously send requests.

The server processes:

```
500 Requests/Second
```

This value represents the system's **throughput**.

---

## Zero Think Time

A maximum-throughput benchmark assumes that clients never pause between requests.

```text
Send Request
      │
      ▼
Receive Response
      │
      ▼
Immediately Send Next Request
```

There is **no waiting** between requests.

The server remains fully utilized, making this an effective way to measure **maximum throughput**.

---

# 3. Response Time (Latency)

**Question:**

> **How long does a single request take to complete?**

Response time measures how long one user waits for a response.

## Example

Imagine visiting an e-commerce website.

```text
Open Product Page
        │
        ▼
Waiting...
        │
        ▼
Page Appears
```

Measured response time:

```
350 ms
```

This is the application's **response time** (or **latency**).

---

## Think Time

Real users do not continuously send requests.

Instead, they spend time reading or interacting with the application.

Example:

```text
Open Product
      │
      ▼
Read Page
30 Seconds
      │
      ▼
Click Next Product
```

The waiting period between requests is called **think time**.

Performance tests that include think time better simulate **real user behavior**.

---

# Throughput vs Response Time

Consider two servers.

| Server | Throughput | Response Time |
| ------ | ---------: | ------------: |
| **A**  |    500 RPS |        500 ms |
| **B**  |    300 RPS |        300 ms |

### Server A

- Handles more requests
- Supports more concurrent users
- Higher system capacity

### Server B

- Responds more quickly
- Better user experience
- Lower latency

Neither server is universally better.

The correct choice depends on the application's goals.

---

# Average Response Time vs Percentiles

Suppose a system processes **20 requests**.

Response times:

```text
1s
1s
1s
1s
1s
...
1s
100s
```

The average response time becomes:

```
6 seconds
```

At first glance, this appears terrible.

However, the average hides important information.

```text
19 Users
     │
     ▼
Completed Within 1 Second

1 User
     │
     ▼
Waited 100 Seconds
```

Only one request was unusually slow.

---

# Percentiles

Instead of relying only on averages, performance engineers often examine **percentiles**.

### 90th Percentile (P90)

```
90% of requests
        │
        ▼
Completed Within
1 Second
```

The slowest **10%** of requests took longer.

Similarly:

- **P95** → 95% of requests completed within the reported time.
- **P99** → 99% of requests completed within the reported time.

Percentiles provide a much clearer picture of user experience than averages alone.

---

# Why Percentiles Matter

Imagine two applications.

### Application A

```text
Average Response Time = 300 ms
P99 = 320 ms
```

Almost every user experiences similar performance.

---

### Application B

```text
Average Response Time = 300 ms
P99 = 8 seconds
```

Most users are happy, but a small percentage experience severe delays.

Although the averages are identical, **Application A provides a much more consistent user experience**.

For this reason, companies commonly monitor:

- P90
- P95
- P99

instead of relying solely on average response time.

---

# Choosing the Right Metric

| Metric            | Measures                        | Common Use Cases                              |
| ----------------- | ------------------------------- | --------------------------------------------- |
| **Elapsed Time**  | Total time for one complete job | Batch jobs, ETL, report generation            |
| **Throughput**    | Work completed per unit time    | Web servers, APIs, streaming systems          |
| **Response Time** | Time taken by one request       | Interactive applications, REST APIs, websites |

---

# Summary

```text
Performance Metrics
│
├── Elapsed Time
│     └── How long does one complete job take?
│
├── Throughput
│     └── How much work can the system perform per second?
│
└── Response Time
      └── How long does one request take?
```

---

# Key Takeaways

- **Elapsed Time** measures how long a complete task takes from start to finish.
- **Throughput** measures how much work a system can perform over time.
- **Response Time (Latency)** measures how long an individual request takes.
- **Zero Think Time** is used to measure maximum throughput.
- **Think Time** simulates realistic user behavior during performance testing.
- Average response time can hide slow requests.
- **P90**, **P95**, and **P99** provide a more accurate picture of user experience than averages alone.