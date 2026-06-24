# Synchronization

* Synchronization in multithreading is a mechanism that controls the execution and access of multiple threads to shared resources.
* It ensures that only one thread can access a critical section of code or resource at a time, preventing data inconsistency, race conditions, and corrupted states.

## Why synchronization is needed?

* In multithreaded applications, threads execute simultaneously and asynchronously. If multiple threads attempt to read and modify the same variable or object concurrently, their operations may interleave.
Race Conditions: Occur when the final outcome of a program depends on the unpredictable sequence in which threads are executed.
Data Inconsistency: A thread might overwrite the changes of another thread because it was reading outdated data from its local cache rather than the main memory.

# Synchronization, Volatile, and Deadlock in Java

## 1. Why Do We Need Synchronization?

**Problem:** Multiple threads accessing SHARED data can cause RACE CONDITIONS.

### Example Without Synchronization

```java
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;   // NOT atomic! Actually 3 steps:
                   // 1. Read count
                   // 2. Add 1
                   // 3. Write back count
    }
}

public class Test {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        
        Runnable task = () -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        };
        
        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        System.out.println(counter.getCount());
        // Expected: 2000
        // Actual: Often LESS than 2000! (e.g. 1847, 1923...)
        // RACE CONDITION occurred!
    }
}
```

### Why Does This Happen?

Thread 1                    Thread 2

─────────────────────────────────────────

Read count = 5

Read count = 5

Add 1 → 6

Add 1 → 6

Write count = 6

Write count = 6
BOTH threads incremented, but count only

went from 5 to 6 — ONE increment was LOST!
This is called a RACE CONDITION

---

## 2. synchronized Keyword — The Fix

### Synchronized Method

```java
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

Now only ONE thread can execute `increment()` at a time. Other threads must WAIT until the current thread finishes and releases the lock.

Result: `count = 2000` every time — correct!

### Synchronized Block (More Granular Control)

```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        // Only this critical section is locked,
        // not the entire method!
        synchronized (lock) {
            count++;
        }
    }
}
```

**Why use a block instead of whole method?**

If a method has other code that doesn't touch shared data, locking the ENTIRE method is wasteful.

`synchronized` block locks ONLY the critical section:
- Better performance
- Other threads can still execute non-critical code

---

## 3. How synchronized Works Internally

Every Java object has an INTRINSIC LOCK (also called MONITOR LOCK).

When a thread enters a synchronized method/block:
1. Thread tries to ACQUIRE the lock on the object
2. If lock is FREE → thread acquires it, proceeds
3. If lock is HELD by another thread → current thread goes to BLOCKED state
4. When the lock holder finishes (exits synchronized block), it RELEASES the lock
5. One of the waiting threads acquires lock, proceeds

```java
// Synchronized on 'this' (default for instance methods)
public synchronized void method1() {
    // locks on 'this' object
}

// Equivalent explicit version:
public void method1() {
    synchronized (this) {
        // same effect
    }
}

// Synchronized on the CLASS (for static methods)
public static synchronized void method2() {
    // locks on Counter.class object, 
    // NOT on any instance!
}
```

### Important Distinction — Instance Lock vs Class Lock

```java
class Counter {
    public synchronized void instanceMethod() {
        // locks on "this" — the specific OBJECT instance
    }
    
    public static synchronized void staticMethod() {
        // locks on Counter.class — the CLASS itself
    }
}

Counter c1 = new Counter();
Counter c2 = new Counter();

// Thread A calls c1.instanceMethod()
// Thread B calls c2.instanceMethod()
// NO BLOCKING! Different objects = different locks!

// Thread A calls Counter.staticMethod()
// Thread B calls Counter.staticMethod()  
// BLOCKING! Same class = same lock, 
// regardless of which instance calls it
```

---

## 4. volatile Keyword

### The Problem volatile Solves — Visibility

Each thread can have its OWN CPU CACHE. A thread might read a STALE (outdated) value from its own cache instead of main memory!

```java
class Flag {
    private boolean running = true;   // NOT volatile
    
    public void stop() {
        running = false;
    }
    
    public void run() {
        while (running) {
            // do something
        }
        System.out.println("Stopped!");
    }
}

public class Test {
    public static void main(String[] args) throws InterruptedException {
        Flag flag = new Flag();
        
        Thread t1 = new Thread(flag::run);
        t1.start();
        
        Thread.sleep(100);
        flag.stop();   // sets running = false
        
        // PROBLEM: t1 might NEVER see this change!
        // t1 might loop FOREVER because it's reading 
        // a CACHED copy of 'running' that's still true
    }
}
```

### Fix With volatile

```java
class Flag {
    private volatile boolean running = true;   // volatile!
    
    public void stop() {
        running = false;
    }
    
    public void run() {
        while (running) {
            // do something
        }
        System.out.println("Stopped!");
    }
}
```

**What volatile guarantees:**

1. **VISIBILITY:** Every read of a volatile variable goes directly to MAIN MEMORY, not CPU cache. Every write immediately updates MAIN MEMORY.
2. Other threads ALWAYS see the LATEST value. No stale cached copies!

**What volatile does NOT guarantee:**

volatile does NOT make operations ATOMIC!

### volatile vs synchronized — Critical Difference!

```java
private volatile int count = 0;

public void increment() {
    count++;   // STILL NOT THREAD SAFE!
}

// WHY? count++ is THREE operations:
// 1. Read count        ← volatile ensures latest value
// 2. Add 1              
// 3. Write count        ← volatile ensures visibility

// But between step 1 and step 3, 
// ANOTHER thread could read the OLD value!
// volatile only fixes VISIBILITY, not ATOMICITY
```

| Feature | volatile | synchronized |
|---|---|---|
| Visibility (no stale cache reads) | Yes | Yes |
| Atomicity (no race conditions) | No | Yes |
| Mutual exclusion (one thread at a time) | No | Yes |
| Performance | Faster | Slower (locking overhead) |
| Use case | Simple flags, status variables | Complex operations, critical sections |

**Rule of thumb:**
- Use volatile for simple flag variables (like a boolean "running" or "shutdown" flag)
- Use synchronized when multiple threads need to READ-MODIFY-WRITE shared data

---

## 5. Deadlock

### What is Deadlock?

Deadlock occurs when TWO or MORE threads are BLOCKED FOREVER, each waiting for a lock that the OTHER thread holds.

Classic analogy: Two people trying to pass each other in a narrow hallway, each waiting for the other to move first — NEITHER moves, FOREVER!

### Code Example — Classic Deadlock

```java
class Account {
    private String name;
    private double balance;
    
    public Account(String name, double balance) {
        this.name = name;
        this.balance = balance;
    }
    
    public synchronized void transfer(Account to, double amount) {
        System.out.println(Thread.currentThread().getName() + 
            " attempting to lock " + this.name);
        synchronized (to) {
            System.out.println(Thread.currentThread().getName() + 
                " attempting to lock " + to.name);
            this.balance -= amount;
            to.balance += amount;
        }
    }
}

public class DeadlockDemo {
    public static void main(String[] args) {
        Account accountA = new Account("A", 1000);
        Account accountB = new Account("B", 1000);
        
        // Thread 1: transfers from A to B
        Thread t1 = new Thread(() -> {
            accountA.transfer(accountB, 100);
        });
        
        // Thread 2: transfers from B to A
        Thread t2 = new Thread(() -> {
            accountB.transfer(accountA, 50);
        });
        
        t1.start();
        t2.start();
        
        // DEADLOCK! Program hangs forever!
    }
}
```

### Why This Deadlocks — Step by Step

Thread 1 (t1)                    Thread 2 (t2)

─────────────────────────────────────────────────────

Locks accountA

Locks accountB

Tries to lock accountB

WAITING...                    Tries to lock accountA

WAITING...
t1 holds A, wants B

t2 holds B, wants A
NEITHER can proceed — STUCK FOREVER!

### 4 Conditions Required for Deadlock (All Must Be True)

1. **MUTUAL EXCLUSION** — Resources can't be shared (locks are exclusive)
2. **HOLD AND WAIT** — A thread holds one resource while waiting for another
3. **NO PREEMPTION** — A resource can't be forcibly taken from a thread
4. **CIRCULAR WAIT** — Thread A waits for Thread B's resource, Thread B waits for Thread A's resource (forms a CYCLE)

If you break even ONE condition, deadlock CANNOT occur!

---

## 6. How to Prevent Deadlock

### Solution 1 — Lock Ordering (Most Common Fix)

```java
class Account {
    private String name;
    private double balance;
    private final int id;   // unique identifier for ordering
    
    public synchronized void transfer(Account to, double amount) {
        Account first = this.id < to.id ? this : to;
        Account second = this.id < to.id ? to : this;
        
        synchronized (first) {
            synchronized (second) {
                this.balance -= amount;
                to.balance += amount;
            }
        }
    }
}

// Now BOTH threads ALWAYS try to lock accounts 
// in the SAME order (lower id first)
// This breaks CIRCULAR WAIT — no deadlock possible!
```

### Solution 2 — tryLock() With Timeout

```java
import java.util.concurrent.locks.ReentrantLock;

class Account {
    private final ReentrantLock lock = new ReentrantLock();
    
    public void transfer(Account to, double amount) {
        try {
            if (lock.tryLock(1, TimeUnit.SECONDS)) {
                try {
                    if (to.lock.tryLock(1, TimeUnit.SECONDS)) {
                        try {
                            // transfer logic
                        } finally {
                            to.lock.unlock();
                        }
                    }
                } finally {
                    lock.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// If lock can't be acquired within timeout,
// thread gives up instead of waiting forever!
// Breaks the "wait forever" aspect of deadlock
```

### Solution 3 — Avoid Nested Locks

Simplest solution: Design your code to NEVER need to hold two locks at the same time.

If a thread acquires Lock A, make sure it releases Lock A BEFORE trying to acquire Lock B.

---

## 7. Deadlock vs Race Condition — Don't Confuse These!

| Feature | Race Condition | Deadlock |
|---|---|---|
| What happens | Wrong/inconsistent RESULT | Threads STUCK FOREVER, no progress |
| Cause | Missing synchronization | Circular waiting for locks |
| Symptom | Program runs but gives wrong output | Program HANGS, never completes |
| Fix | Add synchronized/locks | Lock ordering, tryLock, avoid nested locks |

---

## Interview Questions

**Q1. What is a race condition?**
A: When multiple threads access and modify shared data concurrently without synchronization, leading to unpredictable or incorrect results, because operations like `count++` aren't atomic.

**Q2. What does the synchronized keyword do internally?**
A: Uses the object's intrinsic/monitor lock. A thread must acquire this lock before entering the synchronized block/method. Other threads attempting to enter are blocked until the lock is released.

**Q3. What is the difference between synchronizing on an instance method vs a static method?**
A: Instance method synchronized locks on "this" (the specific object). Static method synchronized locks on the Class object itself — shared across ALL instances.

**Q4. What problem does volatile solve?**
A: Visibility problem — ensures every read/write of the variable goes directly to main memory instead of a thread's local CPU cache, so all threads see the latest value immediately.

**Q5. Does volatile make an operation thread-safe?**
A: No — volatile only guarantees visibility, NOT atomicity. Operations like `count++` (read-modify-write) are still NOT thread-safe even if count is volatile, because multiple steps can still interleave between threads.

**Q6. When should you use volatile vs synchronized?**
A: Use volatile for simple flag/status variables that are just read/written (not modified based on previous value). Use synchronized when multiple threads need to perform read-modify-write operations on shared data.

**Q7. What is deadlock?**
A: A situation where two or more threads are blocked forever, each waiting for a lock held by another thread, forming a circular dependency with no way to proceed.

**Q8. What are the four necessary conditions for deadlock?**
A: Mutual exclusion, hold and wait, no preemption, and circular wait. All four must hold simultaneously for deadlock to occur.

**Q9. How do you prevent deadlock using lock ordering?**
A: Ensure all threads acquire multiple locks in the SAME consistent order (e.g., always lock the lower-ID resource first). This breaks the circular wait condition, since no cycle can form.

**Q10. What is the difference between deadlock and race condition?**
A: Race condition produces incorrect results while the program still completes execution. Deadlock causes threads to be permanently stuck, halting program progress entirely.

**Q11. Can synchronized blocks cause deadlock?**
A: Yes — if two threads try to acquire the same two locks in DIFFERENT orders, deadlock can occur, even though each individual synchronized block is correctly written.

**Q12. What is a monitor lock in Java?**
A: Every Java object has an associated intrinsic lock (monitor). synchronized methods/blocks use this lock to ensure mutual exclusion — only one thread can hold the monitor at a time.

---
