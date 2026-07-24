# Chapter 2: An Approach to Performance Testing

The main lesson of this chapter is:

> **Benchmark the real application whenever possible.**
>
> Microbenchmarks are useful, but they are easy to get wrong in Java because of the **JIT compiler**, **garbage collection**, and other JVM optimizations.

---

# 1. Test the Real Application

The author divides benchmarks into three categories.

| Benchmark Type | What It Tests | Example |
|---------------|---------------|---------|
| **Microbenchmark** | A single method or operation | Compare `HashMap` vs `TreeMap` |
| **Mesobenchmark** | A subsystem | Benchmark only the caching layer |
| **Macrobenchmark** | The entire application | Benchmark a complete Spring Boot REST API |

The closer your benchmark is to the **real production application**, the more reliable the results will be.

---

# 2. What is a Microbenchmark?

A microbenchmark measures one very small piece of code.

Examples include:

- `StringBuilder` vs `StringBuffer`
- `HashMap` vs `TreeMap`
- Recursive Fibonacci vs Iterative Fibonacci

Microbenchmarks are useful for comparing algorithms and implementations.

However, because Java performs runtime optimizations, they must be written carefully.

---

# 3. Problem 1: Dead Code Elimination (DCE)

Suppose you write:

```java
long start = System.nanoTime();

fib(50);

long end = System.nanoTime();
```

You expect Java to calculate the Fibonacci number.

However, the result is never used.

The JIT compiler notices this.

```text
Result Computed
      │
      ▼
Never Used
      │
      ▼
Remove Computation
```

This optimization is called **Dead Code Elimination (DCE)**.

Your benchmark may report almost **0 ns**, not because the algorithm is extremely fast, but because the JVM removed it entirely.

---

## How to Prevent Dead Code Elimination

Always use the computed result.

```java
volatile long result;

result = fib(50);
```

Because the value is observed later, the compiler cannot remove the computation.

---

# 4. Problem 2: JVM Warm-Up

Java does not immediately compile methods into machine code.

Execution typically follows this sequence.

```text
First Execution
      │
      ▼
Interpreter
      │
      ▼
Method Becomes Hot
      │
      ▼
JIT Compiler
      │
      ▼
Machine Code
      │
      ▼
Fast Execution
```

If you benchmark only the first few executions, you mostly measure:

- Interpretation
- JIT compilation

rather than the application's steady-state performance.

---

## Correct Benchmark Flow

```text
Run Method
      │
      ▼
Warm Up JVM
      │
      ▼
Start Timer
      │
      ▼
Run Benchmark
      │
      ▼
Stop Timer
```

Always allow the JVM to warm up before recording measurements.

---

# 5. Problem 3: Threaded Microbenchmarks

Suppose you benchmark the following method.

```java
synchronized void increment() {
    counter++;
}
```

You start **100 threads**.

You think you're measuring:

```text
increment()
```

In reality, you're measuring:

```text
Lock Contention
      │
      ▼
Context Switching
      │
      ▼
Thread Scheduling
```

Most of the execution time is spent waiting for the synchronized lock, not incrementing the counter.

Your benchmark is no longer measuring what you intended.

---

# 6. Problem 4: Test Different Inputs

Suppose you benchmark only:

```java
fib(50);
```

The JVM may aggressively optimize because it always sees the same input.

Instead, benchmark a variety of values.

```java
fib(20);
fib(35);
fib(50);
fib(70);
fib(100);
```

or generate a list of inputs before the benchmark begins.

Using varied inputs better reflects real-world behavior.

---

# 7. Don't Generate Random Numbers Inside the Benchmark

### Incorrect

```java
for (...) {
    fib(random.nextInt());
}
```

Now you're measuring:

```text
Random Number Generation
              +
Fibonacci Computation
```

instead of Fibonacci alone.

---

## Correct

Generate the inputs before starting the timer.

```java
int[] inputs = ...;

// Start timer

for (...) {
    fib(inputs[i]);
}
```

Now the measured time reflects only the algorithm being tested.

---

# 8. Benchmark Realistic Inputs

Suppose your production application normally executes:

```java
fib(10);
```

but your benchmark measures:

```java
fib(1000);
```

The benchmark no longer represents production behavior.

Always benchmark the **common case**, not unrealistic edge cases.

---

# 9. Code Behaves Differently in Production

Suppose you benchmark:

```java
HashMap.get();
```

inside a tiny standalone program.

The JVM sees:

- One class
- Small call stack
- Predictable execution

Now place the same code inside a Spring Boot application.

```text
Spring Boot
      │
      ▼
Spring Security
      │
      ▼
Hibernate
      │
      ▼
Jackson
      │
      ▼
Logging
      │
      ▼
AOP
```

The runtime environment changes dramatically.

The JIT compiler may optimize the exact same method differently.

This is why standalone microbenchmarks don't always predict production performance.

---

# 10. Garbage Collection Can Change Results

Suppose there are two implementations.

### Version A

- Very fast
- Allocates many temporary objects

### Version B

- Slightly slower
- Allocates very few objects

In a small benchmark:

```
Version A wins.
```

because young-generation garbage collections are inexpensive.

In a busy production server:

```text
Many Objects Created
        │
        ▼
Objects Survive
        │
        ▼
Promoted to Old Generation
        │
        ▼
Full Garbage Collection
        │
        ▼
Application Pause
```

Now Version B may outperform Version A because it generates much less garbage.

---

# 11. Tiny Improvements May Not Matter

Suppose you optimize an operation by:

```
5 nanoseconds
```

If it is executed:

```
1 billion times
```

the optimization is worthwhile.

If it is executed:

```
once per REST request
```

users will probably never notice the difference.

Always optimize where the application actually spends its time.

---

# 12. Use JMH

Writing reliable Java microbenchmarks is difficult.

To solve this problem, the OpenJDK team created the **Java Microbenchmark Harness (JMH)**.

JMH automatically handles many common benchmarking pitfalls.

### Features

- JVM warm-up iterations
- Multiple measurement iterations
- Multiple JVM forks
- Protection against Dead Code Elimination
- Reliable timing and statistical analysis

Whenever possible, prefer JMH over writing your own benchmarking framework.

---

# Summary

```text
Real Application
      │
      ▼
Macrobenchmark
      │
      ▼
Most Reliable Results

Microbenchmark
      │
      ▼
Useful for Small Comparisons
      │
      ▼
Must Handle:
    • JVM Warm-Up
    • JIT Compilation
    • Dead Code Elimination
    • Garbage Collection
    • Realistic Inputs
```

---

# Key Takeaways

- Benchmark the **real application** whenever possible.
- Always **warm up the JVM** before recording measurements.
- Ensure the computed result is used, or the **JIT compiler may eliminate the work**.
- Benchmark with **realistic and varied inputs**.
- Do not include unrelated work (such as random number generation) inside the timed section.
- Remember that production behavior may differ because of **JIT optimizations** and **garbage collection**.
- Use **JMH** instead of writing your own microbenchmark framework whenever possible.