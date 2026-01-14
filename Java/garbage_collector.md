
## What is Garbage Collection in Java?

**Garbage Collection (GC)** is Java’s automatic memory management process.

👉 Its job is to:

-   Find objects in **heap memory** that are **no longer used**
    
-   **Free that memory** automatically
    

So you **don’t manually free memory** like in C/C++. JVM handles it for you.

### Simple example
```java
Person p = new Person();  // object created
p = null;                 // object eligible for GC
```

## How does Garbage Collector decide?

GC checks **reachability**:

-   If an object is reachable from **GC Roots**, it stays
    
-   Otherwise, it’s garbage
    

### GC Roots include:

-   Local variables (method stack)
    
-   Static variables
    
-   Active threads
    
-   JNI references
 

## Types of Garbage Collectors in Java

Different GC algorithms exist for different use cases (low latency, high throughput, etc.).
### 1. Serial Garbage Collector

**Best for:** Small applications, single CPU

-   Uses **single thread**
    
-   **Stops the world** during GC
    
-   Simple and low overhead
    
Enable:

`-XX:+UseSerialGC` 

----------

### 2. Parallel Garbage Collector (Throughput GC)

**Best for:** High-throughput apps

-   Uses **multiple threads**
    
-   Faster GC than Serial
    
-   Still **stop-the-world**
    
Default GC in Java 8

`-XX:+UseParallelGC` 

----------

### 3. CMS (Concurrent Mark Sweep) – ⚠ Deprecated

**Best for:** Low-latency apps (older JVMs)

-   Runs GC **concurrently** with application
    
-   Reduces pause times
    
-   Causes **fragmentation**
    
Deprecated since Java 9

----------

### 4. G1 Garbage Collector (Most Popular Now)

**Best for:** Large heap, low latency + good throughput

-   Heap divided into **regions**
    
-   Collects garbage **incrementally**
    
-   Predictable pause times
    
Default GC from Java 9+

`-XX:+UseG1GC` 

----------

### 5. ZGC (Ultra-Low Latency)

**Best for:** Huge heaps, real-time systems

-   Pause times in **milliseconds**
    
-   Almost fully concurrent
    
-   Very scalable
    
 Java 11+

`-XX:+UseZGC` 



### 6. Shenandoah GC

**Best for:** Low pause time, Red Hat JVMs

-   Concurrent compaction
    
-   Very low latency
    

`-XX:+UseShenandoahGC`

## Generational Garbage Collection

Most collectors use **Generational GC** based on this idea:

> Most objects die young


# Heap Structure

| Generation | Purpose |
| --- | --- |
| Young Gen | New objects |
| Old Gen | Long-lived objects |
| Metaspace | Class metadata |
### Young Generation spaces:

-   **Eden**
    
-   **Survivor S0 / S1**
    
# Types of GC Events

| GC Type | Description |
|---------|--------------|
| Minor GC | Cleans Young Gen |
| Major GC | Cleans Old Gen |
| Full GC | Cleans entire heap (slow!) |



## Can we request GC manually?

Yes, but JVM may ignore it:
```java
System.gc();
```

Use **only for testing**, never in production.



## Real-World Tip (Very Important)

In production:

-   Prefer **G1GC** (default)
    
-   Monitor with:
    
    -   GC logs
        
    -   JConsole / VisualVM
        
-   Avoid:
    
    -   Creating too many short-lived objects
        
    -   Memory leaks via static references

# Comparison Table

| GC Type | Threads | Stop-the-World | Pause Time | Heap Size | Use Case | Status |
|---------|---------|----------------|------------|-----------|----------|--------|
| Serial GC | Single | Yes | High | Small | Small apps | Active |
| Parallel GC | Multi | Yes | Medium | Medium–Large | Batch jobs | Active |
| CMS GC | Multi | Partial | Low | Medium | Low latency (old) | ❌ Deprecated |
| G1 GC | Multi | Partial | Low & predictable | Large | Enterprise apps | ✅ Default |
| ZGC  | Multi  | Almost none    | Very Low (ms)    | Very Large    | Real-time systems    | Active |
| Shenandoah GC        | Multi        | Almost none                | Very Low                | Large                | Low-latency apps                | Active |
| Epsilon GC        | None        | N/A                | None                | Fixed                | Testing only                | Active |

#  GC Usage Instructions

## 1️⃣ Command Line (Most Basic)
If you run your app using Java:
```bash
java -XX:+UseShenandoahGC -jar myapp.jar
```
That’s it. JVM will start with Shenandoah GC.

## 2️⃣ Spring Boot Application
### From command line
```bash
gjava -XX:+UseShenandoahGC -Xms2g -Xmx2g -jar app.jar
```
### Using environment variable
- **Linux / Mac:**
  ```bash
  export JAVA_OPTS="-XX:+UseShenandoahGC"
  java $JAVA_OPTS -jar app.jar
  ```
- **Windows:**
  ```cmd
  set JAVA_OPTS=-XX:+UseShenandoahGC
  java %JAVA_OPTS% -jar app.jar
  ```

## 3️⃣ application.properties / application.yml
> ❌ You cannot put this directly in:
> `application.properties`
> ❌ This will NOT work:
> `-XX:+UseShenandoahGC`
> JVM options must be applied before Spring starts.

## 4️⃣ Docker (Very Common in Production)
### Dockerfile ENTRYPOINT example:
```dockerfile
ENTRYPOINT ["java", "-XX:+UseShenandoahGC", "-jar", "app.jar"]
```
### Or using environment variable:
```dockerfile
env JAVA_OPTS="-XX:+UseShenandoahGC"
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## 5️⃣ Kubernetes / Helm
environment variable:
```yaml
env:
  - name: JAVA_OPTS
    value: "-XX:+UseShenandoahGC"
```
or in container args:
defined as:
```yaml
aargs:
  - "-XX:+UseShenandoahGC"
```

## 6️⃣ IDE (IntelliJ / Eclipse)
### IntelliJ IDEA:
1. Run → Edit Configurations 
2. Select your app
3. VM Options
4. Add:
   ```
   -XX:+UseShenandoahGC
   ```

## Eclipse

1. Run Configurations → Arguments → VM arguments

# 7️⃣ Systemd / Server Startup Script
```bash
JAVA_OPTS="-XX:+UseShenandoahGC"
ExecStart=/usr/bin/java $JAVA_OPTS -jar app.jar
```