# Chapter 4: Working with the JIT Compiler

The **Just-in-Time (JIT) compiler** is one of the most important components of the Java Virtual Machine (JVM). It has a significant impact on application performance because it determines how Java bytecode is translated into machine code.

This chapter explains how the JIT compiler works, its advantages and disadvantages, and how it has evolved across different JDK versions. It also introduces compiler tuning techniques that can help diagnose performance issues when an application is running slowly without an obvious cause.

---

# Just-in-Time Compilers: An Overview

Before understanding the JIT compiler, it is helpful to understand how different programming languages execute programs.

A CPU can execute only **machine code**, which consists of processor-specific instructions. Every programming language must eventually be translated into machine code before it can run.

---

# Compiled Languages

Languages such as **C++** and **Fortran** are compiled languages.

The source code is compiled into machine code before execution. The generated binary is targeted for a specific processor architecture.

Compatible processors can execute the same binary. For example, Intel and AMD processors share a common instruction set, allowing the same executable to run on both. However, binaries that use newer processor instructions may not run on older processors.

---

# Interpreted Languages

Languages such as **PHP** and **Perl** are interpreted.

Instead of producing a binary beforehand, the interpreter translates each line of the program into machine code as it executes.

This approach makes interpreted languages highly portable because the same source code can run on any system that has the appropriate interpreter.

However, interpreted code is generally slower because the same statements may be translated repeatedly, especially inside loops.

---

# Compilers vs. Interpreters

A compiler has knowledge of the entire program before generating machine code. This allows it to perform various optimizations, such as arranging instructions so that the CPU can continue executing useful work while waiting for data to be loaded from memory.

An interpreter processes one statement at a time and therefore has much less information available for optimization.

For this reason, compiled code is generally faster than interpreted code.

---

# Java's Approach

Java takes a middle ground between compiled and interpreted languages.

Java source code is first compiled into an intermediate language called **Java bytecode**, rather than directly into machine code.

```text
Java Source Code
        │
        ▼
      javac
        │
        ▼
 Java Bytecode
        │
        ▼
      JVM
        │
        ▼
 JIT Compiler
        │
        ▼
Machine Code
```

Because Java bytecode is platform independent, the same `.class` files can run on any system that has a compatible JVM.

As the application executes, the JVM compiles frequently executed bytecode into native machine code. Since this compilation happens while the program is running, it is called **Just-in-Time (JIT) compilation**.

---

# Platform Dependency

Although Java bytecode is platform independent, the machine code produced by the JIT compiler is platform specific.

For example, JDK 8 cannot generate code that uses some of the newer instruction sets available on Intel Skylake processors, whereas JDK 11 can take advantage of those newer instructions.