# Java Memory Model — Heap, Stack, and GC Basics

## 1. Overview — Where Does Java Store Things?

When a Java program runs, the JVM divides memory into several areas. The two most important for interviews are **Heap** and **Stack**.

```
JVM Memory Areas:
├── Heap            (shared, stores objects)
├── Stack            (per-thread, stores method calls + local variables)
├── Method Area / Metaspace  (stores class metadata)
├── PC Registers     (per-thread, tracks current instruction)
└── Native Method Stack (for native/C code calls)
```

---

## 2. Stack Memory

**What it stores:**
- Local variables (primitives like `int`, `boolean`)
- References to objects (NOT the object itself!)
- Method call frames (one frame per method call)

**Key properties:**
- Each THREAD gets its OWN stack
- Memory is allocated/deallocated in LIFO order (stack frames pushed/popped)
- Very FAST access (no garbage collection needed here)
- Fixed/small size — can overflow! (`StackOverflowError`)

### Example

```java
public class Test {
    public static void main(String[] args) {
        int x = 10;              // x stored in main()'s stack frame
        Test obj = new Test();   // 'obj' reference stored in stack
                                  // actual Test object stored in HEAP
        calculate(x);
    }

    static void calculate(int num) {
        int result = num * 2;    // 'result' and 'num' in calculate()'s stack frame
    }
}
```

### Visual

```
STACK (grows downward as methods are called)
+--------------------------+
| calculate() frame         |  <- currently executing
|   num = 10                 |
|   result = 20              |
+--------------------------+
| main() frame               |
|   x = 10                   |
|   obj = (ref -> 0x1A2B)    | -----> points to HEAP
+--------------------------+
```

### StackOverflowError

```java
public class Test {
    static void recurse() {
        recurse();   // calls itself infinitely
    }

    public static void main(String[] args) {
        recurse();   // StackOverflowError!
        // Each call adds a NEW frame to the stack
        // Eventually stack runs out of space
    }
}
```

---

## 3. Heap Memory

**What it stores:**
- All OBJECTS (instances of classes)
- Instance variables (non-static fields)
- Arrays

**Key properties:**
- SHARED across all threads (one Heap per JVM process)
- Managed by the Garbage Collector
- Larger than stack, but slower to access
- Can throw `OutOfMemoryError` if it runs out of space

### Example

```java
class Person {
    String name;     // stored INSIDE the object on heap
    int age;          // stored INSIDE the object on heap
}

public class Test {
    public static void main(String[] args) {
        Person p1 = new Person();   // object created on HEAP
        p1.name = "Shambhavi";       // modifies heap object
        p1.age = 25;

        Person p2 = p1;              // p2 is a NEW reference,
                                       // pointing to the SAME heap object!
        p2.age = 26;

        System.out.println(p1.age);  // 26! (because p1 and p2 point to SAME object)
    }
}
```

### Visual

```
STACK                          HEAP
+-------------+
| main() frame |
|  p1 -------- | -------> +-------------------+
|  p2 -------- | -------> | Person object       |
+-------------+           |  name = "Shambhavi" |
   (both point to          |  age = 26            |
    the SAME object!)      +-------------------+
```

---

## 4. Stack vs Heap — Comparison Table

| Feature | Stack | Heap |
|---|---|---|
| Stores | Local variables, references, method frames | Objects, instance variables, arrays |
| Scope | Per-thread (each thread has its own) | Shared across entire application |
| Access Speed | Faster | Slower |
| Size | Smaller, fixed | Larger, can grow |
| Memory management | Automatic (LIFO, frame popped on method return) | Managed by Garbage Collector |
| Error on exhaustion | `StackOverflowError` | `OutOfMemoryError` |
| Lifetime | Exists only during method execution | Exists until no longer referenced + GC collects it |

---

## 5. Heap Generations — Young, Old, Metaspace

The Heap itself is further divided into regions to make Garbage Collection more efficient:

```
HEAP
├── Young Generation
│   ├── Eden Space        (new objects created here first)
│   ├── Survivor Space 0   (S0)
│   └── Survivor Space 1   (S1)
└── Old Generation (Tenured)
    └── Long-lived objects end up here

METASPACE (outside heap, since Java 8)
└── Class metadata, method info, static variables
    (replaced the old "PermGen" from Java 7 and earlier)
```

### Why Generations Exist

```
Observation: MOST objects are short-lived
(created, used briefly, then discarded)

So it's more efficient to:
1. Create new objects in a SMALL area (Eden)
2. Frequently clean up Eden (Minor GC) — FAST
3. Only objects that SURVIVE multiple GC cycles
   get promoted to Old Generation
4. Old Generation is cleaned less often (Major GC) — SLOWER
   since most objects there are expected to live long
```

### Object Lifecycle Through Generations

```
new Object() created
        |
        v
   Eden Space  -----> Minor GC runs (objects without
        |              references are removed)
        |
   Survives? --YES--> Moved to Survivor Space (S0)
        |
   More Minor GC cycles, keeps surviving
        |
        v
   Promoted to OLD GENERATION (after surviving
   a threshold number of GC cycles, e.g. 15 by default)
        |
        v
   Major/Full GC eventually cleans Old Generation
```

---

## 6. Garbage Collection (GC) Basics

**What is GC?**
Automatic process that reclaims memory occupied by objects that are no longer reachable/referenced by the program.

### When Does an Object Become Eligible for GC?

```java
public class Test {
    public static void main(String[] args) {
        Person p1 = new Person();   // object created, p1 references it

        p1 = null;                   // NOW the Person object has
                                       // NO references pointing to it
                                       // -> eligible for Garbage Collection!
    }
}
```

```java
// Another way — reassigning reference
Person p1 = new Person();   // Object A created
Person p2 = new Person();   // Object B created

p1 = p2;   // p1 now points to Object B
           // Object A has NO references left
           // -> Object A is eligible for GC!
```

### How GC Determines "Reachability"

```
GC Roots (always considered reachable):
- Local variables in active stack frames
- Static variables
- Active threads

An object is eligible for GC when it is
NO LONGER reachable from any GC Root,
even if other objects still reference it
in a cycle (GC can detect and clean up cycles!)
```

### Example — Even Circular References Get Collected

```java
class Node {
    Node next;
}

public class Test {
    public static void main(String[] args) {
        Node a = new Node();
        Node b = new Node();
        a.next = b;
        b.next = a;   // circular reference!

        a = null;
        b = null;
        // Even though 'a' and 'b' reference EACH OTHER,
        // neither is reachable from main()'s stack anymore
        // -> BOTH are eligible for GC!
        // (Unlike older non-tracing GC algorithms,
        //  Java's GC handles cycles correctly)
    }
}
```

---

## 7. Types of GC Algorithms (Just Awareness Level)

```
Minor GC   -> Cleans Young Generation (Eden + Survivor spaces)
              Fast, frequent

Major/Full GC -> Cleans Old Generation (and sometimes everything)
              Slower, less frequent
              Can cause noticeable application pauses

Common GC Algorithms in JVM:
- Serial GC      -> single-threaded, simple, small applications
- Parallel GC     -> multi-threaded, throughput focused
- CMS (Concurrent Mark Sweep) -> deprecated since Java 9
- G1 GC (Garbage First) -> default since Java 9, balances
                            throughput and pause time
- ZGC / Shenandoah -> very low pause time, for large heaps
```

---

## 8. finalize() — Legacy Concept (Know It, Don't Use It)

```java
class Person {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Object being garbage collected");
    }
}
```

```
- finalize() is called by GC before reclaiming an object
- NOT guaranteed WHEN (or even IF) it will be called
- Deprecated since Java 9, removed in later versions
- Modern alternative: try-with-resources + AutoCloseable
  for deterministic cleanup (e.g. closing file/DB connections)
```

---

## 9. Common Memory-Related Errors

| Error | Cause |
|---|---|
| `StackOverflowError` | Too many nested/recursive method calls, stack frames exceed limit |
| `OutOfMemoryError: Java heap space` | Too many objects created, heap exhausted, GC can't free enough |
| `OutOfMemoryError: Metaspace` | Too many classes loaded (common in apps that dynamically generate classes) |

### Memory Leak Example (Even With GC!)

```java
public class Cache {
    private static List<Object> cache = new ArrayList<>();

    public void addToCache(Object obj) {
        cache.add(obj);   // adds, but NEVER removes!
    }
}

// Even though GC exists, objects in 'cache' are
// ALWAYS reachable (static reference), so they
// NEVER become eligible for GC — classic memory leak
// even in a garbage-collected language!
```

```
KEY TAKEAWAY: GC only collects UNREACHABLE objects.
If you accidentally keep a reference alive
(e.g. static collections, listeners not unregistered),
GC CANNOT help you — this causes memory leaks
even in Java!
```

---

## Interview Questions

**Q1. What is the difference between Stack and Heap memory?**
A: Stack stores local variables, references, and method call frames — it is per-thread and very fast. Heap stores actual objects and is shared across the entire application, managed by the Garbage Collector.

**Q2. Where are object references stored vs the actual object?**
A: The reference variable is stored on the Stack (inside the method's frame), while the actual object it points to is stored on the Heap.

**Q3. What causes a StackOverflowError?**
A: Excessive or infinite recursion, where each method call adds a new frame to the stack until the fixed stack size is exhausted.

**Q4. What causes an OutOfMemoryError?**
A: The Heap runs out of space because too many objects are being created and the Garbage Collector cannot free enough memory (often due to memory leaks or genuinely high memory demand).

**Q5. When does an object become eligible for Garbage Collection?**
A: When it is no longer reachable from any GC Root (active stack frames, static variables, or active threads) — even if it's still referenced by other unreachable objects (e.g. in a circular reference).

**Q6. Can Java's GC handle circular references?**
A: Yes — Java's GC uses reachability analysis from GC Roots, not simple reference counting, so it can correctly identify and collect objects involved in circular references as long as neither is reachable from a GC Root.

**Q7. What is the difference between Young Generation and Old Generation?**
A: Young Generation (Eden + Survivor spaces) holds newly created, typically short-lived objects and is cleaned frequently via Minor GC. Old Generation holds long-lived objects that have survived multiple GC cycles and is cleaned less frequently via Major/Full GC.

**Q8. What is Metaspace?**
A: A memory area (outside the Heap) that stores class metadata, method information, and static variables. It replaced PermGen starting from Java 8.

**Q9. What is a memory leak in Java, given that it has automatic Garbage Collection?**
A: A memory leak occurs when objects are no longer needed but remain reachable (e.g. through static collections, unclosed resources, or unregistered listeners), preventing GC from reclaiming them even though they're logically unused.

**Q10. What is the purpose of finalize() and why is it discouraged now?**
A: finalize() was meant to allow cleanup before GC reclaims an object, but its execution timing is not guaranteed and it can even be skipped entirely. It's deprecated since Java 9 in favor of deterministic cleanup mechanisms like try-with-resources and AutoCloseable.

**Q11. What is the default Garbage Collector in modern Java versions?**
A: G1 GC (Garbage First), default since Java 9, designed to balance throughput and pause times for general-purpose applications.

**Q12. Does setting a reference to null guarantee immediate garbage collection?**
A: No — it only makes the object ELIGIBLE for garbage collection. The actual collection happens at a time determined by the JVM/GC algorithm, not immediately.

---

## Summary

| Concept | One Line |
|---|---|
| Stack | Per-thread memory for local variables, references, and method frames |
| Heap | Shared memory for all objects, managed by GC |
| StackOverflowError | Stack memory exhausted (usually deep/infinite recursion) |
| OutOfMemoryError | Heap memory exhausted (too many live objects) |
| Young Generation | Eden + Survivor spaces, holds new/short-lived objects |
| Old Generation | Holds long-lived objects that survived multiple GC cycles |
| Metaspace | Stores class metadata, replaced PermGen since Java 8 |
| GC Roots | Stack variables, static variables, active threads — reachability starting points |
| Minor GC | Cleans Young Generation, fast and frequent |
| Major/Full GC | Cleans Old Generation, slower, less frequent |
| Memory Leak | Unused objects stay reachable, GC cannot free them |

--------------

## Why is stack thread-safe but heap not?

Each thread gets its own stack. When a thread is created, the JVM allocates a separate stack just for it. So thread A's local variables and thread B's local variables live in completely different memory regions — they can never physically collide.
The heap is shared by all threads. There's exactly one heap per JVM process, and every thread can read and write any object on it. So if thread A and thread B both hold a reference to the same object, they can both modify it at the same time.

---------------