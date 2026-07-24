# Throughput vs Response Time (Latency)

One of the most important concepts in performance engineering is understanding the difference between **throughput** and **response time (latency)**.

Although they're related, they measure **different aspects of system performance**.

---

# Restaurant Analogy

Imagine two restaurants.

### Restaurant A

- Serves **500 customers per hour**
- Each customer waits **30 minutes**

### Restaurant B

- Serves **300 customers per hour**
- Each customer waits **10 minutes**

Which restaurant is better?

It depends on your perspective.

- As the **restaurant owner**, you might prefer **Restaurant A** because it serves more customers and generates more revenue.
- As a **customer**, you'd probably choose **Restaurant B** because you receive your food much faster.

The same trade-off exists in software systems.

---

# Throughput

**Throughput** answers the question:

> **How much work can the system complete per unit of time?**

Common units include:

- Requests Per Second (RPS)
- Transactions Per Second (TPS)
- Messages Per Second

Example:

```
500 Requests/Second
```

means the server successfully processes **500 requests every second**.

### Higher Throughput Means

- More users served
- Better scalability
- Higher overall system capacity

---

# Response Time (Latency)

**Response Time** (also called **Latency**) answers the question:

> **How long does one request take to complete?**

Example:

```
300 ms
```

means a request takes **0.3 seconds** from arrival until the response is returned.

### Lower Latency Means

- Faster responses
- Better user experience
- Less waiting for users

---

# Comparing Two Servers

| Server | Throughput | Response Time |
| ------ | ---------: | ------------: |
| **A**  |    500 RPS |        500 ms |
| **B**  |    300 RPS |        300 ms |

---

## Server A

```text
500 Users
     │
     ▼
Server Processes Requests
     │
     ▼
Each User Waits ~500 ms
```

### Advantages

- Handles more traffic
- Better resource utilization
- Suitable for high-scale services

### Disadvantage

- Higher latency for individual users

---

## Server B

```text
300 Users
     │
     ▼
Server Processes Requests
     │
     ▼
Each User Waits ~300 ms
```

### Advantages

- Faster responses
- Better user experience

### Disadvantage

- Lower overall capacity

---

# Which Does Netflix Prioritize?

Netflix serves **millions of concurrent users** streaming videos.

Its primary concern is **throughput**, since it must handle an enormous number of simultaneous requests.

It still keeps latency low enough that users don't notice delays, but scalability is the primary goal.

---

# Which Does Google Search Prioritize?

Suppose you search:

```
Spring Boot Tutorial
```

If the results took:

```
500 ms
```

the search would feel noticeably slower.

Google therefore prioritizes **very low latency**, even if that slightly reduces maximum throughput.

---

# Which Does a Bank Prioritize?

Imagine transferring:

```
₹50,000
```

Would you rather wait:

```
100 ms
```

or

```
600 ms
```

Most users prefer the faster confirmation.

Banks therefore optimize for **low and predictable response times**, while still supporting many concurrent users.

---

# Why You Can't Maximize Both

Suppose your server has:

```
8 CPU Cores
```

### Moderate Load

```
300 RPS
CPU Usage = 50%
```

Result:

- CPUs have spare capacity
- Requests finish quickly
- Low latency

---

### Heavy Load

```
500 RPS
CPU Usage = 95%
```

Result:

- More requests are processed
- Requests begin waiting in queues

As queue length increases:

- Response time increases
- Throughput improves only until the server reaches capacity

---

### Maximum Load

```
CPU Usage = 100%
```

At this point:

- Throughput reaches its practical limit
- New requests spend most of their time waiting
- Some requests may even be dropped

This is why latency often increases dramatically as CPU utilization approaches **100%**.

---

# Basic Formulas

### Throughput

```text
Throughput = Completed Requests / Second
```

Measures the **system's capacity**.

---

### Response Time

```text
Response Time = Finish Time − Arrival Time
```

Measures how long **an individual request** takes to complete.

---

# Spring Boot Example

Consider the following REST endpoint.

```java
@GetMapping("/users")
public List<User> getUsers() {
    return service.findAll();
}
```

A performance test using **JMeter** or **k6** reports:

```
Throughput : 300 RPS
Latency    : 120 ms
```

After optimizations such as:

- Better database indexing
- Reduced object allocations
- Improved thread pool configuration

the results become:

```
Throughput : 500 RPS
Latency    : 140 ms
```

### What Changed?

- Throughput increased by approximately **67%**
- Latency increased by **20 ms**

Whether this is an improvement depends on the application's requirements.

---

# When to Prioritize Each Metric

| Application         | Primary Goal                      |
| ------------------- | --------------------------------- |
| Search Engines      | Low Response Time                 |
| Video Streaming     | High Throughput                   |
| Payment Systems     | Low and Predictable Response Time |
| Batch Processing    | High Throughput                   |
| Real-Time Gaming    | Extremely Low Response Time       |
| Analytics Pipelines | High Throughput                   |

---

# Throughput vs Response Time

| Throughput                    | Response Time                       |
| ----------------------------- | ----------------------------------- |
| Measures total work completed | Measures how long one request takes |
| Unit: RPS, TPS, Messages/sec  | Unit: ms, μs, ns                    |
| Higher is generally better    | Lower is generally better           |
| Important for scalability     | Important for user experience       |

---

# Key Takeaways

- **Throughput** measures **how much work** a system can complete per unit of time.
- **Response Time (Latency)** measures **how quickly** each individual request is served.
- Increasing throughput often increases CPU utilization, which can also increase latency.
- As utilization approaches **100%**, response times usually rise sharply because requests spend more time waiting in queues.
- The best performance strategy depends on the application's requirements.
- A well-designed system balances throughput and latency rather than maximizing one at the expense of the other.