## Common JIT Optimizations

The JIT compiler improves performance by applying various optimizations while converting bytecode into native machine code. The C1 compiler performs simpler optimizations to improve startup time, while the C2 compiler performs more aggressive optimizations to maximize long-term performance.

| Optimization                  | Description                                                                                                                   |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **Method Inlining**           | Replaces a method call with the method's body, eliminating method call overhead.                                              |
| **Constant Folding**          | Evaluates constant expressions during compilation instead of at runtime.                                                      |
| **Dead Code Elimination**     | Removes code whose results are never used.                                                                                    |
| **Escape Analysis**           | Determines whether an object escapes a method or thread. If it does not, the JVM can optimize its allocation.                 |
| **Scalar Replacement**        | Replaces an object with its individual fields, avoiding object allocation.                                                    |
| **Loop Optimizations**        | Improves loops through techniques such as loop unrolling, moving invariant code outside the loop, and reducing bounds checks. |
| **Lock Elimination**          | Removes unnecessary synchronization when an object is accessed by only one thread.                                            |
| **Lock Coarsening**           | Combines multiple synchronization operations into a single larger lock to reduce locking overhead.                            |
| **Register Allocation**       | Keeps frequently used variables in CPU registers instead of main memory.                                                      |
| **Branch Prediction**         | Optimizes frequently executed branches based on runtime behavior.                                                             |
| **Code Motion**               | Moves computations outside loops or frequently executed paths when safe.                                                      |
| **Vectorization (SuperWord)** | Uses SIMD instructions to process multiple data elements with a single CPU instruction.                                       |

## Optimization Examples

### Method Inlining

Instead of:

```java
int square(int x) {
    return x * x;
}

int y = square(5);
```

The JIT may generate code equivalent to:

```java
int y = 5 * 5;
```

This removes the overhead of the method call.

---

### Constant Folding

Instead of:

```java
int x = 10 * 20;
```

The JIT computes the result during compilation:

```java
int x = 200;
```

---

### Dead Code Elimination

```java
int x = 5;
x = 10;
System.out.println(x);
```

The first assignment is removed because its value is never used.

---

### Escape Analysis

```java
Point p = new Point(1, 2);
return p.x + p.y;
```

If the object never escapes the method, the JVM may avoid allocating it on the heap.

---

### Scalar Replacement

Instead of creating an object:

```java
Point p = new Point(1, 2);
```

The JVM may replace it with individual variables:

```text
x = 1
y = 2
```

No object allocation is required.

---

### Loop Optimizations

```java
for (int i = 0; i < arr.length; i++) {
    sum += arr[i];
}
```

The JVM may:

- Move invariant computations outside the loop.
- Unroll the loop.
- Reduce bounds checks.

---

### Lock Elimination

```java
StringBuilder sb = new StringBuilder();
```

If the object is accessed by only one thread, unnecessary synchronization can be removed.

---

### Register Allocation

Instead of repeatedly accessing memory, frequently used variables are stored in CPU registers for faster access.

---

### Branch Prediction

```java
if (x != null) {
    process(x);
}
```

If the condition is usually true, the compiled code is optimized for that execution path.

---

### Vectorization (SuperWord)

Instead of processing one element at a time:

```text
1 + 1
2 + 2
3 + 3
4 + 4
```

The CPU performs multiple operations simultaneously using SIMD instructions.

## C1 vs C2 Optimizations

| C1 Compiler                    | C2 Compiler                                                      |
| ------------------------------ | ---------------------------------------------------------------- |
| Fast compilation               | Slower compilation                                               |
| Basic optimizations            | Aggressive optimizations                                         |
| Improves startup time          | Maximizes peak performance                                       |
| Performs less runtime analysis | Uses extensive runtime profiling to apply advanced optimizations |