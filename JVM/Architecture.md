# JDK, JRE, and JVM Architecture

Understanding the relationship between the **JDK**, **JRE**, and **JVM** is fundamental to Java development.

At a high level:

```text
Java Source Code
        │
        ▼
     javac
        │
        ▼
Bytecode (.class)
        │
        ▼
       JVM
        │
        ▼
Native Machine Code
        │
        ▼
       CPU
```

The **JDK** provides tools for developing Java applications, the **JRE** provides everything needed to run them, and the **JVM** executes Java bytecode.

---

# JDK (Java Development Kit)

The **JDK** is the complete Java development environment.

It contains:

- JRE
- Java compiler (`javac`)
- Documentation tool (`javadoc`)
- Debugger (`jdb`)
- Packaging tools (`jar`, `jlink`, `jpackage`)
- Other development utilities

```text
+--------------------------------------+
| JDK                     |
| ----------------------- |
| JRE                     |
| javac                   |
| javadoc                 |
| jdb                     |
| jar                     |
| jlink                   |
| Other Development Tools |
+--------------------------------------+
```

The JDK is used when **developing** Java applications.

---

# JRE (Java Runtime Environment)

The **JRE** contains everything required to **run** a Java application.

It consists of:

- JVM
- Java Standard Library (Core APIs)

```text
+--------------------------------------+
| JRE                       |
| ------------------------- |
| JVM                       |
| Java Core Libraries (API) |
+--------------------------------------+
```

Examples of core libraries include:

- `java.lang`
- `java.util`
- `java.io`
- `java.net`
- `java.time`

The JRE does **not** include development tools like `javac`.

> **Note:** Since Java 11, Oracle no longer distributes the JRE separately. Modern JDK distributions include everything needed for both development and execution.

---

# JVM (Java Virtual Machine)

The **JVM** is responsible for executing Java bytecode.

It performs tasks such as:

- Loading classes
- Verifying bytecode
- Managing memory
- Executing bytecode
- Compiling hot code with the JIT compiler
- Garbage collection

```text
                 JVM
+--------------------------------------------+
| Class Loader                              |
| Bytecode Verifier                         |
| Runtime Data Areas                        |
|   • Method Area                           |
|   • Heap                                  |
|   • Java Stack                            |
|   • PC Register                           |
|   • Native Method Stack                   |
|                                           |
| Execution Engine                          |
|   • Interpreter                           |
|   • JIT Compiler                          |
|   • Garbage Collector                     |
+--------------------------------------------+
```

---

# JVM Components

## 1. Class Loader

The **Class Loader** loads compiled `.class` files into memory.

Responsibilities include:

- Loading classes on demand
- Linking classes
- Initializing classes

```text
.class File
     │
     ▼
Class Loader
```

---

## 2. Bytecode Verifier

Before execution, the JVM verifies that the bytecode is safe.

It checks for:

- Illegal memory access
- Invalid instructions
- Stack consistency
- Type safety

```text
.class
   │
   ▼
Bytecode Verifier
   │
   ▼
Safe to Execute
```

This helps prevent many runtime security issues.

---

## 3. Runtime Data Areas

The JVM allocates several memory regions while executing a program.

### Heap

- Stores objects
- Shared by all threads
- Managed by the Garbage Collector

---

### Method Area

Stores:

- Class metadata
- Static variables
- Runtime constant pool
- Method bytecode

---

### Java Stack

Each thread owns its own stack.

It stores:

- Local variables
- Method parameters
- Return addresses

---

### PC Register

Each thread maintains a **Program Counter (PC)** register.

It keeps track of the next bytecode instruction to execute.

---

### Native Method Stack

Used when Java invokes native methods through **JNI (Java Native Interface)**.

---

# 4. Execution Engine

The Execution Engine runs Java bytecode.

It consists of:

- Interpreter
- JIT Compiler
- Garbage Collector

---

## Interpreter

The interpreter executes bytecode **one instruction at a time**.

```text
Bytecode
    │
    ▼
Interpreter
    │
    ▼
Execution
```

Advantages:

- Fast startup
- No compilation delay

Disadvantages:

- Slower execution for frequently executed code

---

## JIT (Just-In-Time) Compiler

The JIT compiler identifies **hot methods** and compiles them into native machine code.

```text
Frequently Executed Bytecode
             │
             ▼
       JIT Compiler
             │
             ▼
      Native Machine Code
```

Advantages:

- Significantly faster execution
- CPU executes optimized native instructions

---

## Garbage Collector

The Garbage Collector automatically reclaims memory occupied by objects that are no longer reachable.

```text
Unused Objects
       │
       ▼
Garbage Collector
       │
       ▼
Memory Reclaimed
```

Benefits include:

- Reduced memory leaks
- Automatic memory management
- Simplified application development

---

# Java Execution Flow

```text
You Write
Java Source Code
        │
        ▼
      javac
        │
        ▼
Java Bytecode (.class)
        │
        ▼
        JVM
        │
 ┌──────┴────────┐
 │               │
Interpreter   JIT Compiler
 │               │
 └──────┬────────┘
        ▼
Native Machine Code
        │
        ▼
       CPU
        │
        ▼
Execute Instructions
```

---

# Overall Architecture

```text
+---------------------------------------------+
| JDK (Java Development Kit)       |
| -------------------------------- |
| JRE + Development Tools          |
| (javac, javadoc, jar, jdb, etc.) |
+---------------------------------------------+
                     │
                     ▼
+---------------------------------------------+
| JRE (Java Runtime Environment)  |
| ------------------------------- |
| JVM + Java Core Libraries (API) |
+---------------------------------------------+
                     │
                     ▼
+=============================================+
| JVM                   |
| --------------------- |
| Class Loader          |
| Bytecode Verifier     |
| Runtime Data Areas    |
| • Method Area         |
| • Heap                |
| • Java Stack          |
| • PC Register         |
| • Native Method Stack |
|                       |
| Execution Engine      |
| • Interpreter         |
| • JIT Compiler        |
| • Garbage Collector   |
+=============================================+
                     │
                     ▼
          Native Machine Code
                     │
                     ▼
                    CPU
```

---

# Key Takeaways

- **JDK** is used to develop Java applications.
- **JRE** provides the runtime environment needed to execute Java programs.
- **JVM** executes Java bytecode and manages memory, compilation, and garbage collection.
- The **Class Loader** loads classes into memory.
- The **Bytecode Verifier** ensures bytecode is safe before execution.
- The **Interpreter** executes bytecode immediately, while the **JIT Compiler** optimizes frequently executed code into native machine instructions.
- The **Garbage Collector** automatically reclaims unused memory.
- Java achieves platform independence because the JVM translates platform-neutral bytecode into native instructions for the underlying operating system and CPU.