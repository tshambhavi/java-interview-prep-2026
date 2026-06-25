# ExecutorService, ThreadPool, Callable, and Future in Java

## 1. Why Do We Need ExecutorService?

**Problem with creating threads manually:**

```java
for (int i = 0; i < 1000; i++) {
    new Thread(() -> doTask()).start();
}
```

- Creates 1000 actual OS threads!
- Thread creation is EXPENSIVE (memory + CPU)
- No control over how many run simultaneously
- Can crash the system (OutOfMemoryError)
- No easy way to get RESULTS back from threads

**Solution:** ExecutorService manages a POOL of reusable threads instead of creating new ones constantly.

---

## 2. Creating ExecutorService — Thread Pool Types

```java
import java.util.concurrent.*;

// 1. Fixed Thread Pool — fixed number of threads
ExecutorService fixedPool = Executors.newFixedThreadPool(4);

// 2. Single Thread Executor — only 1 thread, tasks run sequentially
ExecutorService singlePool = Executors.newSingleThreadExecutor();

// 3. Cached Thread Pool — creates new threads as needed, reuses idle ones
ExecutorService cachedPool = Executors.newCachedThreadPool();

// 4. Scheduled Thread Pool — for delayed/periodic tasks
ScheduledExecutorService scheduledPool = Executors.newScheduledThreadPool(2);
```

### Basic Usage

```java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 5; i++) {
            int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " running on "
                    + Thread.currentThread().getName());
            });
        }

        executor.shutdown();   // IMPORTANT! Always shutdown when done
    }
}

// Output (only 3 threads, 5 tasks):
// Task 1 running on pool-1-thread-1
// Task 2 running on pool-1-thread-2
// Task 3 running on pool-1-thread-3
// Task 4 running on pool-1-thread-1   <- thread REUSED!
// Task 5 running on pool-1-thread-2   <- thread REUSED!
```

**Key insight:** Only 3 threads created, but 5 tasks executed! Threads are REUSED once they finish a task — this is the whole point of pooling.

---

## 3. Why shutdown() is Critical

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
executor.submit(() -> System.out.println("Task"));

// WITHOUT shutdown():
// JVM keeps running FOREVER!
// Thread pool threads are NOT daemon threads by default
// Program never exits even after task completes

executor.shutdown();   // Always do this!
```

```java
// Two ways to shut down:

executor.shutdown();
// Allows currently running/submitted tasks to complete
// Rejects NEW tasks after this call
// Graceful shutdown

executor.shutdownNow();
// Attempts to STOP all actively running tasks immediately
// Returns list of tasks that were waiting but never started
// Forceful shutdown
```

### Proper Shutdown Pattern

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
try {
    executor.submit(() -> doTask());
} finally {
    executor.shutdown();
    try {
        // Wait up to 5 seconds for tasks to finish
        if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
            executor.shutdownNow();   // force if timeout exceeded
        }
    } catch (InterruptedException e) {
        executor.shutdownNow();
    }
}
```

---

## 4. submit() vs execute()

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

// execute() — takes Runnable, returns NOTHING (void)
executor.execute(() -> System.out.println("Task A"));

// submit() — takes Runnable OR Callable, returns Future!
Future<?> future1 = executor.submit(() -> System.out.println("Task B"));

Future<Integer> future2 = executor.submit(() -> {
    return 42;   // Callable — CAN return a value!
});
```

| Method | Accepts | Returns | Can Return Value? |
|---|---|---|---|
| `execute()` | Runnable only | void | No |
| `submit()` | Runnable or Callable | Future | Yes (with Callable) |

---

## 5. Runnable vs Callable — The Key Difference

```java
// Runnable — CANNOT return a value, CANNOT throw checked exception
Runnable task1 = () -> {
    System.out.println("Running");
    // return something;  NOT ALLOWED
};

// Callable — CAN return a value, CAN throw checked exception
Callable<Integer> task2 = () -> {
    System.out.println("Calculating");
    return 10 + 20;   // ALLOWED!
};

Callable<String> task3 = () -> {
    if (someCondition) {
        throw new Exception("Something went wrong");  // ALLOWED!
    }
    return "Success";
};
```

| Feature | Runnable | Callable |
|---|---|---|
| Method name | `run()` | `call()` |
| Return value | void only | Returns `V` (generic type) |
| Checked exceptions | Cannot throw | Can throw `Exception` |
| Since version | Java 1.0 | Java 5 (with ExecutorService) |

---

## 6. Future — Getting Results Back

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Callable<Integer> task = () -> {
    Thread.sleep(2000);   // simulate long computation
    return 100;
};

Future<Integer> future = executor.submit(task);

System.out.println("Task submitted, doing other work...");

// get() BLOCKS until result is ready!
Integer result = future.get();

System.out.println("Result: " + result);

executor.shutdown();

// Output:
// "Task submitted, doing other work..." (prints immediately)
// (waits 2 seconds here, blocked on future.get())
// "Result: 100" (prints after 2 seconds)
```

### Important Future Methods

```java
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(3000);
    return 50;
});

future.isDone();          // false (still running)
future.isCancelled();     // false (not cancelled)

// get() — BLOCKS until done (waits forever by default!)
Integer result = future.get();

// get() with TIMEOUT — throws TimeoutException if not done in time
try {
    Integer result2 = future.get(1, TimeUnit.SECONDS);
} catch (TimeoutException e) {
    System.out.println("Task took too long!");
}

// cancel() — attempt to cancel the task
future.cancel(true);   // true = interrupt if already running
```

---

## 7. Real World Example — Running Multiple Tasks in Parallel

```java
import java.util.*;
import java.util.concurrent.*;

public class Test {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        List<Callable<Integer>> tasks = new ArrayList<>();
        tasks.add(() -> { Thread.sleep(1000); return 10; });
        tasks.add(() -> { Thread.sleep(1000); return 20; });
        tasks.add(() -> { Thread.sleep(1000); return 30; });

        // invokeAll() — runs ALL tasks, waits for ALL to complete
        List<Future<Integer>> results = executor.invokeAll(tasks);

        int sum = 0;
        for (Future<Integer> f : results) {
            sum += f.get();
        }

        System.out.println("Total: " + sum);   // 60
        // Takes ~1 second total (parallel!)
        // NOT 3 seconds (which sequential would take)

        executor.shutdown();
    }
}
```

### invokeAny() — Get First Completed Result

```java
List<Callable<String>> tasks = new ArrayList<>();
tasks.add(() -> { Thread.sleep(3000); return "Slow server"; });
tasks.add(() -> { Thread.sleep(1000); return "Fast server"; });
tasks.add(() -> { Thread.sleep(2000); return "Medium server"; });

// invokeAny() — returns result of FIRST task to complete
// Cancels the rest!
String result = executor.invokeAny(tasks);
System.out.println(result);   // "Fast server" (completes first)
```

---

## 8. ThreadPoolExecutor — What's Actually Happening Internally

```
Executors.newFixedThreadPool(3) internally creates:

new ThreadPoolExecutor(
    3,                          // corePoolSize
    3,                          // maximumPoolSize
    0L, TimeUnit.MILLISECONDS,  // keepAliveTime
    new LinkedBlockingQueue<Runnable>()   // task queue
);
```

### How Tasks Flow Through The Pool

```
Task submitted
     |
     v
Are core threads (3) all busy?
     |
   NO -> Create new thread, run task immediately
     |
   YES -> Add task to QUEUE (LinkedBlockingQueue)
              |
              v
        Wait for a thread to become FREE
              |
              v
        Free thread picks up next task from queue
```

**Key insight:** With Fixed Thread Pool, if all threads are busy, NEW tasks just WAIT in an unbounded queue (they don't get rejected, just queued indefinitely).

---

## 9. Comparison of Thread Pool Types

| Pool Type | Thread Count | Behavior | Use Case |
|---|---|---|---|
| `newFixedThreadPool(n)` | Fixed at n | Extra tasks queue up | Known, steady workload |
| `newSingleThreadExecutor()` | Always 1 | Tasks run one by one, in order | Sequential task execution |
| `newCachedThreadPool()` | Unbounded (grows/shrinks) | Creates new thread if none idle, reuses idle ones, kills after 60s idle | Many short-lived async tasks |
| `newScheduledThreadPool(n)` | Fixed at n | Runs tasks after delay or periodically | Cron-like scheduled jobs |

### ScheduledExecutorService Example

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

// Run once, after 3 second delay
scheduler.schedule(() -> System.out.println("Ran once!"),
    3, TimeUnit.SECONDS);

// Run repeatedly every 2 seconds (starting immediately)
scheduler.scheduleAtFixedRate(() -> System.out.println("Tick!"),
    0, 2, TimeUnit.SECONDS);
```

---

## 10. Common Interview Trap — Forgetting shutdown()

```java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.submit(() -> System.out.println("Hello"));

        // Program does NOT exit even after this line!
        // Pool threads keep the JVM alive

        // FIX:
        executor.shutdown();   // NOW program can exit
    }
}
```

---

## Interview Questions

**Q1. Why use ExecutorService instead of creating threads manually?**
A: Creating threads manually for every task is expensive and uncontrolled. ExecutorService manages a reusable pool of threads, reducing overhead and giving control over concurrency level.

**Q2. What is the difference between execute() and submit()?**
A: execute() accepts only Runnable and returns void. submit() accepts Runnable or Callable and returns a Future, allowing you to retrieve results or exceptions.

**Q3. What is the difference between Runnable and Callable?**
A: Runnable's run() method returns void and cannot throw checked exceptions. Callable's call() method returns a value (generic type V) and can throw checked exceptions.

**Q4. What does Future.get() do if the task isn't done yet?**
A: It BLOCKS the calling thread until the task completes. An overloaded version accepts a timeout, throwing TimeoutException if the task doesn't finish in time.

**Q5. What happens if you forget to call shutdown()?**
A: The JVM keeps running indefinitely because thread pool threads are not daemon threads by default — the program never naturally exits.

**Q6. What is the difference between shutdown() and shutdownNow()?**
A: shutdown() allows currently running and queued tasks to complete, rejecting new submissions. shutdownNow() attempts to immediately stop all running tasks and returns the list of tasks that never started.

**Q7. What is the difference between invokeAll() and invokeAny()?**
A: invokeAll() runs all tasks and waits for ALL of them to complete, returning a list of Futures. invokeAny() returns the result of whichever task completes FIRST, cancelling the rest.

**Q8. What happens to extra tasks in a fixed thread pool when all threads are busy?**
A: They are added to an internal task queue (LinkedBlockingQueue by default) and wait until a thread becomes free, rather than being rejected.

**Q9. What is the difference between newFixedThreadPool and newCachedThreadPool?**
A: Fixed pool maintains a constant number of threads; excess tasks queue up. Cached pool creates new threads as needed and reuses idle ones, terminating threads idle for 60+ seconds — better for many short-lived bursty tasks.

**Q10. Can a Callable be passed to execute()?**
A: No — execute() only accepts Runnable. To run a Callable, you must use submit(), which returns a Future to retrieve the result.

---

## Summary

| Concept | One Line |
|---|---|
| ExecutorService | Manages a reusable pool of threads |
| Thread Pool | Group of pre-created threads that execute submitted tasks |
| Callable | Like Runnable but can return a value and throw exceptions |
| Future | Represents a result that will be available later |
| execute() | Fire-and-forget, Runnable only, no return |
| submit() | Returns Future, accepts Runnable or Callable |
| shutdown() | Graceful pool termination, finishes pending tasks |
| shutdownNow() | Forceful termination, stops running tasks immediately |
| invokeAll() | Run all tasks, wait for all to finish |
| invokeAny() | Run all tasks, return first one to finish |

---------------------