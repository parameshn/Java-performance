# Generational Garbage Collectors

Most garbage collectors divide the heap into **generations**. Although the implementation differs among collectors, the basic design is similar.

The heap is divided into:

- **Young Generation**
- **Old (Tenured) Generation**

The **Young Generation** is further divided into:

- **Eden**
- **Survivor Space 0 (S0)**
- **Survivor Space 1 (S1)**

> **Note:** The term **eden** is sometimes incorrectly used to refer to the entire young generation. In reality, eden is only one part of the young generation.

```text
                Java Heap

+-----------------------------------------+
|                                         |
|           Old Generation                |
|                                         |
+--------------------+--------------------+
|       Young Generation                  |
|                                          |
|  Eden  | Survivor 0 | Survivor 1        |
+------------------------------------------+
```

---

# Why Generations?

The rationale behind generations is that **most Java objects have very short lifetimes**.

The book illustrates this with a `BigDecimal` example:

```java
sum = new BigDecimal(0);

for (StockPrice sp : prices.values()) {
    BigDecimal diff =
        sp.getClosingPrice().subtract(averagePrice);

    diff = diff.multiply(diff);

    sum = sum.add(diff);
}
```

Since `BigDecimal` is **immutable**, every arithmetic operation creates a **new object**.

For approximately 250 stock prices:

- `subtract()` creates a new object.
- `multiply()` creates another.
- `add()` creates another.

This loop alone creates roughly **750 intermediate `BigDecimal` objects**, many of which become unreachable during the next iteration.

The JDK itself also creates additional temporary objects internally.

This behavior is common in Java applications, so the garbage collector is designed to efficiently reclaim short-lived objects.

---

# Object Allocation

New objects are first allocated in the **Young Generation**.

More specifically, they are allocated in **Eden**, which occupies the majority of the young generation.

```text
New Object
     │
     ▼
+-------------+
|    Eden     |
+-------------+
```

---

# Minor (Young) GC

When Eden becomes full:

1. The JVM pauses all application threads.
2. The Young Generation is collected.
3. Unreachable objects are discarded.
4. Live objects are moved.

This operation is called:

- **Minor GC**
- **Young GC**

Both terms refer to the same operation.

```text
Eden Full
     │
     ▼
Stop-the-World
     │
     ▼
Minor GC
     │
     ├── Garbage → Removed
     │
     └── Live Objects
             │
             ├── Survivor Space
             └── Old Generation
```

---

# Why Minor GC Is Fast

The book describes two performance advantages.

## 1. Smaller Heap to Scan

Only the **Young Generation** is processed.

Since it is much smaller than the entire heap:

- Less memory is scanned.
- Collections finish more quickly.
- Stop-the-World pauses are shorter.

The trade-off is that these collections occur more frequently.

The author notes that, in most cases, **shorter and more frequent pauses are preferable to infrequent but much longer pauses**.

---

## 2. Automatic Compaction

During a Minor GC:

- Every surviving object is moved.
- Garbage is discarded.

As a result:

- Eden becomes empty.
- One survivor space becomes empty.
- Remaining live objects are compacted into the other survivor space.

```text
Before Minor GC

Eden
■■■■■■■■

S0
□□□□

S1
□□□□


After Minor GC

Eden
□□□□□□□□

S0
■■

S1
□□□□□□□□
```

The Young Generation is therefore **automatically compacted** after every Minor GC.

---

# Promotion to the Old Generation

Objects that survive multiple Minor GCs eventually move into the **Old Generation**.

```text
New Object
      │
      ▼
Eden
      │
Minor GC
      │
      ▼
Survivor Space
      │
Survives Again
      │
      ▼
Old Generation
```

The book introduces promotion here; the promotion policy is discussed in more detail later.

---

# Full GC

Eventually, the Old Generation also fills up.

The JVM must then:

1. Find unreachable objects.
2. Reclaim their memory.
3. Compact the heap.

This operation is called a **Full GC**.

Simple garbage collectors perform this by:

- Stopping all application threads.
- Scanning the Old Generation.
- Removing garbage.
- Compacting memory.

Because the Old Generation is much larger than the Young Generation, **Full GC usually results in significantly longer Stop-the-World pauses**.

---

# Concurrent Collectors

Instead of stopping the application for the entire collection, some garbage collectors perform much of their work **while application threads continue executing**.

These are called:

- **Concurrent Collectors**
- **Low-Pause Collectors**

(Some people refer to them as *pauseless* collectors, but the book notes this is not strictly correct.)

The main advantage is that they greatly reduce long application pauses.

However, this comes with trade-offs:

- Higher overall CPU usage.
- More complex implementation.
- Potentially more tuning.

The book also notes that by **JDK 11**, tuning collectors such as **G1 GC** had become significantly easier than in earlier releases.

---

# Choosing a Garbage Collector

The appropriate garbage collector depends on the application's performance goals.

## REST Applications

If request latency is critical:

- Long Full GC pauses directly affect user response times.
- A concurrent collector may be the better choice.

If average throughput is more important than occasional long pauses:

- A nonconcurrent collector may provide better overall performance.

CPU availability must also be considered.

Concurrent collectors require additional CPU resources.

---

## Batch Applications

For batch workloads:

If sufficient CPU is available:

- Concurrent collectors may allow the batch job to complete sooner by avoiding long Full GC pauses.

If CPU resources are limited:

- The additional CPU consumed by concurrent collection may increase the total execution time.

---

# Modern Status (Java 17/21/24)

The generational design remains a fundamental part of modern garbage collection.

Current production collectors include:

- **G1 GC** (default)
- **Generational ZGC**
- **Shenandoah** (supported OpenJDK builds)

Modern collectors still use the concepts introduced here:

- Young Generation
- Old Generation
- Eden
- Survivor Spaces
- Minor GC
- Object Promotion

The primary improvements in modern JVMs are:

- Much shorter Stop-the-World pauses.
- More concurrent collection phases.
- Better scalability on large heaps.
- Improved automatic tuning.

---

# Key Takeaways

- Most garbage collectors divide the heap into **Young** and **Old** generations.
- The Young Generation consists of **Eden** and **Survivor Spaces**.
- Most newly created objects are allocated in **Eden**.
- When the Young Generation fills, the JVM performs a **Minor (Young) GC**.
- Minor GCs are usually fast because only a small portion of the heap is collected.
- Surviving objects are moved, automatically compacting the Young Generation.
- Objects that survive multiple collections are promoted to the **Old Generation**.
- When the Old Generation fills, the JVM performs a **Full GC**, which generally causes longer pauses.
- Concurrent collectors reduce pause times by performing much of the collection work while application threads continue executing.
- Choosing a garbage collector involves balancing **pause time**, **throughput**, and **CPU usage** according to the application's performance goals.