# HotSpot Compilation

The Java implementation discussed in this book is Oracle's **HotSpot JVM**. The name **HotSpot** comes from the approach it takes toward compiling code.

In a typical program, only a small subset of the code is executed frequently, and the performance of an application depends primarily on how fast those sections of code are executed. These critical sections are known as the **hot spots** of the application. The more frequently a section of code is executed, the "hotter" it becomes.

## Why Doesn't the JVM Compile Code Immediately?

When the JVM executes code, it does not begin compiling it immediately. There are two basic reasons for this.

### 1. Avoiding Unnecessary Compilation

If a piece of code is executed only once, compiling it is essentially a wasted effort. In that case, it is faster to interpret the Java bytecodes than to compile them and execute the compiled code only once.

However, if the code is part of a frequently called method or a loop that executes many iterations, compiling it becomes worthwhile. The time spent compiling the code is outweighed by the savings gained from executing the faster compiled code multiple times.

For this reason, the JVM initially interprets the code so it can determine which methods are executed frequently enough to justify compilation.

### 2. Runtime Optimizations

The more often the JVM executes a method or loop, the more information it gathers about that code.

This allows the JVM to perform optimizations when compiling the code.

### Example: `equals()`

Consider the following statement:

```java
b = obj1.equals(obj2);
```

When the interpreter encounters this statement, it must determine the runtime type (class) of `obj1` to know which implementation of `equals()` should be executed. This dynamic lookup can be time-consuming.

Suppose the JVM observes that every time this statement executes, `obj1` is of type `java.lang.String`.

The JVM can then generate compiled code that directly calls:

```java
String.equals()
```

The code is now faster because:

- It is compiled.
- It skips the runtime lookup of which `equals()` method to execute.

This optimization is based on the assumption that `obj1` continues to refer to a `String`.

If `obj1` later refers to a different type, the JVM must **deoptimize** the compiled code and then **reoptimize** it based on the new runtime behavior.

Even with this possibility, the compiled code is generally faster because it avoids the repeated method lookup.

This type of optimization is possible only after observing the application's runtime behavior, which is another reason the JIT compiler waits before compiling code.

---

# Registers and Main Memory

One of the most important optimizations a compiler can make involves deciding when to use values from **main memory** and when to store values in a **register**.

Consider the following code:

```java
public class RegisterTest {

    private int sum;

    public void calculateSum(int n) {
        for (int i = 0; i < n; i++) {
            sum += i;
        }
    }
}
```

At some point, the instance variable `sum` must reside in main memory. However, retrieving a value from main memory is an expensive operation that takes multiple CPU cycles.

If `sum` were retrieved from and stored back to main memory during every iteration of the loop, performance would be poor.

Instead, the compiler:

1. Loads the initial value of `sum` into a register.
2. Performs the loop using the value stored in the register.
3. Stores the final value back to main memory at a later point.

This optimization is very effective, but it also affects the semantics of thread synchronization.

A thread cannot see the value of a variable stored in another thread's register. Thread synchronization determines when the value stored in a register is written back to main memory and becomes visible to other threads.

Register usage is a general compiler optimization, and the JIT compiler aggressively uses registers. This topic is discussed further in **Escape Analysis**.

---

# Quick Summary

- Java is designed to combine the platform independence of interpreted languages with the native performance of compiled languages.
- Java source code is compiled into an intermediate language called **Java bytecode**.
- The JVM further compiles the bytecode into native assembly language.
- During this compilation, the JVM performs optimizations that significantly improve performance.