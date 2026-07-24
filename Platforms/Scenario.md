# JVM Ergonomics in Docker Containers

One of the most important concepts in modern Java deployments is understanding **how the JVM sees its environment**.

When a Java application starts, it automatically configures itself based on the hardware resources it detects. This automatic configuration is called **JVM ergonomics**.

Modern JVMs are **container-aware**, meaning they respect Docker CPU and memory limits. Older JVMs did not.

---

# Scenario

Suppose you have a server with:

| Resource | Value |
|----------|------:|
| CPU | 4 Cores |
| RAM | 16 GB |

You want to run two Java applications.

Instead of installing them directly on the server, you use Docker.

```text
Physical Server
│
├── Docker Container A
│      Java Application A
│
└── Docker Container B
       Java Application B
```

You split the resources equally.

| Container | CPU | Memory |
|-----------|----:|-------:|
| Container A | 2 Cores | 8 GB |
| Container B | 2 Cores | 8 GB |

Docker can enforce these limits.

```bash
docker run --cpus=2 --memory=8g my-java-app
```

---

# What the JVM Does at Startup

When the JVM starts, it asks questions such as:

- How much memory is available?
- How many processors are available?

Using this information, the JVM automatically configures:

- Heap size (`-Xms`, `-Xmx`)
- Garbage Collector threads
- JIT compiler threads
- ForkJoinPool size
- Internal JVM thread pools

This automatic configuration is known as **JVM ergonomics**.

---

# Java 11+ (Container-Aware)

Suppose the container has:

```
2 CPU Cores
8 GB RAM
```

The JVM detects exactly those resources.

```
Available Processors = 2
Available Memory     = 8 GB
```

It may configure itself like this:

| JVM Setting | Example |
|-------------|---------|
| Heap | ~2 GB |
| GC Threads | 2 |
| Compiler Threads | 2 |

Everything is balanced because the JVM respects the container limits.

---

# Older Java 8 (Before 8u192)

Older JVMs ignored Docker resource limits.

Although Docker restricted the container to:

```
2 CPU Cores
8 GB RAM
```

the JVM examined the **host machine** instead.

```
Available Processors = 4
Available Memory     = 16 GB
```

Possible configuration:

| JVM Setting | Example |
|-------------|---------|
| Heap | 4 GB+ |
| GC Threads | 4 |
| Compiler Threads | 4 |

The JVM assumed it had access to resources that Docker had actually restricted.

---

# Why This Is Dangerous

Suppose the JVM expands the heap.

The JVM believes:

```
Available Memory = 16 GB
```

Docker allows only:

```
8 GB
```

Eventually:

```text
Heap Grows
      │
      ▼
Container Memory Limit Exceeded
      │
      ▼
Linux OOM Killer
      │
      ▼
Java Process Terminated
```

The JVM did not crash because of Java—it was terminated by the operating system.

---

# Too Many Garbage Collector Threads

Consider another example.

Host:

```
16 CPU Cores
```

Container:

```
2 CPU Cores
```

Older JVM:

```
GC Threads = 16
```

Reality:

```
Only 2 CPUs are available.
```

Now sixteen GC threads compete for two CPUs.

Result:

- Excessive context switching
- Lower CPU efficiency
- Reduced throughput

---

# Correct vs Incorrect Behavior

### Modern JVM

```text
Host
4 CPUs
16 GB
      │
      ▼
Container
2 CPUs
8 GB
      │
      ▼
JVM
Uses:
2 CPUs
8 GB
```

Everything matches.

---

### Older JVM

```text
Host
4 CPUs
16 GB
      │
      ▼
Container
2 CPUs
8 GB
      │
      ▼
JVM
Uses:
4 CPUs
16 GB
```

Resource mismatch.

---

# What Is JVM Ergonomics?

JVM ergonomics refers to the JVM's ability to automatically choose sensible defaults based on the available hardware.

Examples include:

- Initial heap size (`-Xms`)
- Maximum heap size (`-Xmx`)
- Garbage collector thread count
- JIT compiler thread count
- ForkJoinPool parallelism

Modern JVMs also consider Docker resource limits.

---

# Can Older JVMs Be Configured Correctly?

Yes.

You can manually configure the JVM.

```bash
java \
-Xms4g \
-Xmx4g \
-XX:ActiveProcessorCount=2 \
-jar app.jar
```

However, upgrading to a modern JDK is strongly recommended because container support is built in.

---

# JVM Diagnostic Tools Inside Containers

Some JVM tools require additional configuration inside Docker.

Examples:

- `jcmd`
- `jstack`
- `jmap`
- `jstat`

You may need to:

- Install the JDK inside the container.
- Grant additional container permissions.
- Connect remotely using JMX or Java Flight Recorder (JFR).

For production systems, these tools are often combined with external monitoring solutions.

---

# Key Takeaways

- JVM ergonomics automatically configures the JVM based on available hardware.
- Java 11+ (including Java 17 and Java 21) is container-aware.
- Older Java 8 versions ignored Docker resource limits.
- Incorrect CPU and memory detection can reduce performance or cause container termination.
- Modern JVMs automatically size heaps and thread pools according to container limits.