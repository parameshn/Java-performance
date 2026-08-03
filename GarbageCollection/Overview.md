# Garbage Collection Overview

One of the most attractive features of programming in Java is that developers do not need to explicitly manage the lifecycle of objects. Objects are created when needed, and when an object is no longer in use, the JVM automatically reclaims the memory occupied by that object.

Although garbage collection introduces additional runtime work, the author argues that tuning a garbage collector is generally much easier than tracking down memory-management bugs such as null pointers and dangling pointers in languages that require manual memory management.

---

## Basic Goal of Garbage Collection

At a high level, garbage collection consists of three primary tasks:

1. Finding objects that are still in use (**live objects**).
2. Identifying objects that are no longer reachable (**garbage**).
3. Reclaiming the memory occupied by those unused objects.

---

## Why Reference Counting Is Not Enough

Garbage collection is sometimes described as finding objects that no longer have any references.

However, the book explains that simple **reference counting** is not sufficient.

Consider a linked list:

```text
Head
 │
 ▼
A → B → C
```

Each object references another object.

If nothing references **Head**, then the entire list is unreachable and can be collected, even though every object except the head still has a reference.

An even more obvious example is a circular list:

```text
A → B → C
↑       │
└───────┘
```

Every object has at least one reference, yet if nothing outside the list references it, the entire structure is garbage.

Because of situations like these, the JVM does **not** determine object liveness by counting references.

---

## Reachability Analysis

Instead, the JVM periodically searches the heap for unused objects using **reachability analysis**.

The search begins from a special set of objects called **GC Roots**.

According to the book, GC Roots primarily include:

- Thread stacks
- System classes

These objects are always considered reachable.

The garbage collector follows every reference starting from these roots.

```text
GC Root
   │
   ▼
Object A
   │
   ▼
Object B
   │
   ▼
Object C
```

Every object reachable from a GC Root is considered **live**.

Objects that cannot be reached from any GC Root are considered **garbage**, even if they still reference each other or even reference live objects.

---

## Reclaiming Memory

Once unreachable objects have been identified, the JVM frees the memory occupied by those objects.

The reclaimed memory can then be reused for future object allocations.

---

## Memory Fragmentation

Simply freeing memory is usually not enough.

Suppose a program repeatedly allocates:

- One array of **1,000 bytes**
- One array of **24 bytes**

Eventually, the heap becomes full.

If all of the 24-byte arrays become unreachable while the 1,000-byte arrays remain live, the heap now contains many small free regions.

```text
□□□□□□□□□□□□□□□□□□□□□□□□
■■■■□□□□■■■■□□□□■■■■□□□□
```

Although there is free memory, it is fragmented into small pieces.

A large allocation cannot fit into any single free region.

---

## Heap Compaction

To solve fragmentation, the JVM compacts the heap.

Live objects are moved together so that the remaining free memory becomes one contiguous region.

Before compaction:

```text
□□□□□□□□□□□□□□□□□□□□□□□□
■■■■□□□□■■■■□□□□■■■■□□□□
```

After compaction:

```text
□□□□□□□□□□□□□□□□□□□□□□□□
■■■■■■■■■■■■□□□□□□□□□□□□
```

Different garbage collectors perform compaction differently.

Some:

- Delay compaction until necessary.
- Compact only portions of the heap.
- Move only small groups of objects at a time.

These implementation differences contribute to the different performance characteristics of each garbage collector.

---

## Mutator Threads and GC Threads

Java applications are typically multithreaded.

The book distinguishes between two logical groups of threads:

### Mutator Threads

Application threads that execute business logic and continuously create and modify objects.

### GC Threads

Threads responsible for:

- Finding live objects.
- Identifying garbage.
- Reclaiming memory.
- Compacting the heap.

---

## Stop-the-World (STW) Pauses

While garbage collection is relocating objects or updating references, application threads cannot safely access those objects.

For this reason, the JVM temporarily pauses all application (mutator) threads while certain garbage collection work is performed.

These pauses are called **Stop-the-World (STW) pauses**.

```text
Application Threads
        │
        ▼
Running
        │
        ▼
Stop-the-World Pause
        │
        ▼
Garbage Collection
        │
        ▼
Application Resumes
```

According to the author, Stop-the-World pauses generally have the greatest impact on application performance.

One of the primary goals of modern garbage collectors is therefore to minimize the duration of these pauses.

---

## Modern Status (Java 17/21/24)

The concepts introduced in this overview remain fundamental in modern Java.

The JVM still:

- Uses **reachability analysis** instead of reference counting.
- Starts traversal from **GC Roots**.
- Identifies **live** and **unreachable** objects.
- Reclaims unused memory.
- Performs heap compaction when required.
- Uses separate **GC threads** and **mutator threads**.
- Performs **Stop-the-World (STW)** pauses when necessary.

Modern collectors such as **G1**, **ZGC**, **Shenandoah**, and **Generational ZGC** implement these same concepts using different algorithms.

The major improvement is that modern collectors perform much more work **concurrently** with application threads, significantly reducing Stop-the-World pause times compared to older collectors.

---

## Key Takeaways

- Java provides automatic memory management through Garbage Collection.
- The JVM determines object liveness using **reachability analysis**, not reference counting.
- The search begins from **GC Roots**.
- Reachable objects are **live**; unreachable objects are **garbage**.
- After reclaiming memory, the JVM may compact the heap to reduce fragmentation.
- Java applications contain **mutator threads** and **GC threads**.
- Some GC operations require **Stop-the-World (STW)** pauses.
- Modern garbage collectors retain these same principles while minimizing pause times through concurrent collection techniques.