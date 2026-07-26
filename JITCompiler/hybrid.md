## Where Does Java's Hybrid Execution Model Come From?

Java is often described as using a **hybrid execution model**, but this behavior comes from the **JVM at runtime**, **not** from the Java compiler (`javac`).

### Compile Time: `javac`

The Java compiler (`javac`) has a single responsibility: it compiles Java source code into **Java bytecode**.

```text
Java Source (.java)
        │
        ▼
      javac
        │
        ▼
Java Bytecode (.class)
```

At this stage:

- Java source code is compiled into bytecode.
- No native machine code is generated.
- No interpretation takes place.
- The output is platform-independent bytecode.

### Runtime: JVM

The hybrid execution model begins when the JVM executes the bytecode.

Initially, the JVM **interprets** the bytecode so the application can start quickly.

As the application runs, the JVM identifies methods that are executed frequently (known as **hot methods**). These methods are then compiled into **native machine code** by the **Just-in-Time (JIT) compiler**.

```text
Java Bytecode (.class)
        │
        ▼
 JVM Interpreter
        │
        ▼
Frequently Executed?
        │
      Yes
        │
        ▼
   JIT Compiler
        │
        ▼
Native Machine Code
        │
        ▼
       CPU
```

### Complete Execution Flow

```text
               Compile Time
-----------------------------------------
Java Source (.java)
        │
        ▼
      javac
        │
        ▼
Java Bytecode (.class)

               Runtime (JVM)
-----------------------------------------
        │
        ▼
 JVM Interpreter
        │
        ▼
Frequently Executed?
        │
      Yes
        │
        ▼
   JIT Compiler
        │
        ▼
Native Machine Code
        │
        ▼
       CPU
```

### Key Point

The **hybrid execution model** is a feature of the **JVM**, not the `javac` compiler.

- **`javac`** only compiles Java source code into platform-independent bytecode.
- The **JVM** first interprets the bytecode and later uses the **JIT compiler** to compile frequently executed code into optimized native machine code.

This combination of **interpretation** and **Just-in-Time compilation** is what makes Java a **hybrid execution model**.