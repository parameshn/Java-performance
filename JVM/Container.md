# One JVM per Docker Container

A common misconception is that multiple Docker containers share a single JVM.

They do not.

Every running Java application starts **its own independent JVM process**.

---

# Docker Image vs Docker Container

Think of Docker like Java.

| Java   | Docker           |
| ------ | ---------------- |
| Class  | Docker Image     |
| Object | Docker Container |

A Docker image is only a template.

A JVM exists only when a container is running.

---

# Example

Suppose you have one image.

```text
springboot-app:1.0
```

You start two containers.

```bash
docker run --name app1 springboot-app:1.0
docker run --name app2 springboot-app:1.0
```

Internally:

```text
Physical Machine

├── Container 1
│      JVM #1
│      Spring Boot Application
│
└── Container 2
       JVM #2
       Spring Boot Application
```

Although both containers use the same image, each container has:

- Its own JVM
- Its own heap
- Its own garbage collector
- Its own JIT compiler
- Its own threads
- Its own class loader

Nothing inside the JVM is shared.

---

# Why?

A Docker container is simply an isolated operating system process.

Every time you run:

```bash
java -jar app.jar
```

the operating system creates a new JVM process.

Running it twice creates two JVMs—even without Docker.

Docker simply isolates those processes.

---

# Multiple Java Applications

Suppose you deploy three services.

```text
Container A
JVM
Inventory Service

Container B
JVM
Order Service

Container C
JVM
Payment Service
```

The system now contains:

- 3 JVMs
- 3 heaps
- 3 garbage collectors
- 3 JIT compilers

Each JVM manages only its own application.

---

# Can Multiple Applications Share One JVM?

Yes, but this is much less common today.

Traditional application servers such as:

- Apache Tomcat
- WildFly / JBoss
- Oracle WebLogic
- IBM WebSphere

could host multiple applications inside a single JVM.

```text
One JVM
│
├── Application A
├── Application B
└── Application C
```

If the JVM crashed, every application stopped.

---

# Modern Microservices

Today, the preferred architecture is:

```text
Container A
└── JVM
    └── User Service

Container B
└── JVM
    └── Order Service

Container C
└── JVM
    └── Payment Service
```

Benefits include:

- Independent deployments
- Separate memory management
- Failure isolation
- Independent scaling
- Separate JVM tuning

Each JVM sizes its heap, GC threads, and compiler threads based on the resources available to its own container.

---

# Key Takeaways

- Every running Java application has its own JVM.
- Docker containers do not share JVM internals.
- Each JVM has its own heap, GC, JIT compiler, threads, and class loaders.
- Modern microservices typically run one JVM per container.
- Each JVM performs ergonomics independently based on the resources assigned to its container.