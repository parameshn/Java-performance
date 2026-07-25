# 2. JDK vs JRE

The **Java Development Kit (JDK)** and the **Java Runtime Environment (JRE)** serve different purposes.

- **JDK** is used to **develop** Java applications.
- **JRE** is used to **run** Java applications.

Since performance engineers frequently analyze and troubleshoot JVM behavior, they primarily work with the **JDK**, which includes diagnostic and monitoring tools.

---

# JDK (Java Development Kit)

The **JDK** is the complete toolkit for Java development.

It includes:

- Java Virtual Machine (**JVM**)
- Java compiler (`javac`)
- Debugging tools
- Profiling tools
- Monitoring tools
- Additional development utilities

### Common JDK Tools

| Tool       | Purpose                                    |
| ---------- | ------------------------------------------ |
| `javac`    | Compiles Java source code into bytecode    |
| `java`     | Launches Java applications                 |
| `jcmd`     | Sends diagnostic commands to a running JVM |
| `jstack`   | Captures thread dumps                      |
| `jmap`     | Analyzes heap memory                       |
| `jfr`      | Records Java Flight Recorder events        |
| `jconsole` | Monitors JVM performance                   |

### Primary Uses

- Developing Java applications
- Compiling source code
- Debugging applications
- Profiling performance
- Monitoring and troubleshooting JVM behavior

---

# JRE (Java Runtime Environment)

The **JRE** provides everything required to **run** Java applications.

It includes:

- Java Virtual Machine (**JVM**)
- Java Standard Library (Core APIs)

### Primary Uses

- Running Java applications
- Providing the runtime environment
- Access to Java's standard libraries

### Limitation

The JRE **cannot compile Java source code** because it does not include the Java compiler (`javac`).

For example:

```bash
javac HelloWorld.java
```

This command is available only in the **JDK**.

---

# JDK vs JRE

| Feature                         |  JDK  |  JRE  |
| ------------------------------- | :---: | :---: |
| Includes JVM                    |   ✅   |   ✅   |
| Includes Java Libraries         |   ✅   |   ✅   |
| Can Run Java Programs           |   ✅   |   ✅   |
| Can Compile Java Code (`javac`) |   ✅   |   ❌   |
| Debugging Tools                 |   ✅   |   ❌   |
| Profiling Tools                 |   ✅   |   ❌   |
| Monitoring Tools                |   ✅   |   ❌   |
| Used for Development            |   ✅   |   ❌   |
| Used for Performance Analysis   |   ✅   |   ❌   |

---

# Why Performance Engineers Prefer the JDK

Performance analysis requires access to JVM diagnostic tools such as:

- `jcmd`
- `jmap`
- `jstack`
- `jfr`

These tools help investigate:

- Memory leaks
- Thread dumps
- Garbage collection behavior
- CPU usage
- Application bottlenecks

Since these tools are part of the **JDK**, the book focuses on the JDK rather than the JRE.

---

# Key Takeaways

- The **JDK** contains everything needed to develop, run, and analyze Java applications.
- The **JRE** contains only the components required to run Java applications.
- The JRE does **not** include the Java compiler (`javac`) or JVM diagnostic tools.
- Performance engineers use the **JDK** because it provides powerful tools such as **`jcmd`**, **`jmap`**, **`jstack`**, and **`jfr`** for monitoring and troubleshooting JVM performance.