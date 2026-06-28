# Section A: Thread, Runnable, Lifecycle
Q1. What is the difference between a Process and a Thread?

A Process is an independent program with its own memory space. A Thread is a lightweight unit of execution within a process — multiple threads in the same process share that process's memory.
Q2. Why is implementing Runnable preferred over extending Thread?

Java doesn't support multiple inheritance. If your class extends Thread, it can't extend anything else. Implementing Runnable keeps that door open since it's just an interface.
Q3. What's the real difference between start() and run()?

start() creates an actual new thread and the JVM eventually calls run() on it. Calling run() directly is just a normal method call — it executes on whichever thread called it, no new thread involved.
Q4. Can you call start() twice on the same thread object?

No — throws IllegalThreadStateException. Once terminated, a thread can't be restarted; you'd need a fresh Thread object.
Q5. What are the six thread lifecycle states?

NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED.
Q6. What's the difference between WAITING and TIMED_WAITING?

WAITING is indefinite (e.g., join() with no timeout). TIMED_WAITING has a bound (sleep(ms), wait(ms), join(ms)).
Q7. What does join() actually do?

Makes the calling thread pause until the thread you called join() on finishes. Used to enforce ordering.
Q8. What's a daemon thread, and when must setDaemon() be called?

A background thread the JVM won't wait for before shutting down (Garbage Collector is one). setDaemon() must be called before start(), or it throws an exception.

# Section B: Synchronization, Volatile, Deadlock
Q1. What is a race condition?

Multiple threads reading/modifying shared data concurrently without synchronization, causing incorrect results — because something like count++ is actually three separate steps, not one atomic operation.
Q2. What does synchronized actually lock onto?

Every object has an intrinsic "monitor" lock. A synchronized instance method locks on this (the specific object). A synchronized static method locks on the Class object itself — shared across all instances.
Q3. What problem does volatile solve, and what doesn't it solve?

It solves visibility — guarantees every thread reads the latest value from main memory instead of a stale cached copy. It does NOT solve atomicity — count++ on a volatile variable is still not thread-safe, because read-modify-write is still three separate steps that can interleave.
Q4. When would you choose volatile over synchronized?

For simple flags (like a "running" boolean) that are just read or set, not read-modified-and-written based on their own previous value.
Q5. What is deadlock, and what are the four conditions required for it?

Two or more threads stuck forever, each waiting on a lock the other holds. Requires: mutual exclusion, hold-and-wait, no preemption, and circular wait — break any one, and deadlock can't happen.
Q6. How does lock ordering prevent deadlock?

If every thread always acquires multiple locks in the same fixed order (e.g., always the lower-ID resource first), a circular wait can never form.
Q7. How is deadlock different from a race condition in terms of symptoms?

Race condition: program finishes but gives a wrong result. Deadlock: program never finishes — it just hangs.

# Section C: ExecutorService, ThreadPool, Callable, Future
Q1. Why use ExecutorService instead of manually creating threads?

Manual thread creation for every task is expensive and uncontrolled. ExecutorService reuses a fixed pool of threads instead of constantly creating/destroying them.
Q2. execute() vs submit() — what's the real difference?

execute() only takes a Runnable and returns nothing. submit() takes a Runnable or Callable and returns a Future, so you can retrieve a result or exception later.
Q3. Runnable vs Callable — core difference?

Runnable's run() returns void and can't throw checked exceptions. Callable's call() can return a value and can throw checked exceptions.
Q4. What happens when you call future.get()?

It blocks the calling thread until the task finishes and a result is available. There's an overloaded version with a timeout that throws TimeoutException if it takes too long.
Q5. What happens if you forget to call shutdown()?

The JVM never exits — pool threads aren't daemon threads by default, so they keep the program alive indefinitely.
Q6. shutdown() vs shutdownNow()?

shutdown() lets already-submitted tasks finish, then refuses new ones — graceful. shutdownNow() tries to stop everything immediately and hands back the tasks that never got to start — forceful.
Q7. invokeAll() vs invokeAny()?

invokeAll() runs everything and waits for all of them. invokeAny() returns as soon as the first one finishes and cancels the rest.

# Section D: Java Memory Model — Heap, Stack, GC
Q1. Core difference between Stack and Heap?

Stack holds local variables, references, and method call frames — one per thread, very fast. Heap holds the actual objects — shared across the whole app, managed by GC.
Q2. What causes StackOverflowError vs OutOfMemoryError?

StackOverflow: usually runaway/infinite recursion exhausting stack frames. OutOfMemory: heap has too many live objects and GC can't free enough space.
Q3. When does an object actually become eligible for garbage collection?

When it's no longer reachable from any GC Root (active stack variables, static variables, running threads) — even if other unreachable objects still point to it.
Q4. Can Java's GC handle two objects that reference each other (a cycle)?

Yes — because GC works by reachability from roots, not by reference counting. If neither object in the cycle is reachable from a root, both get collected.
Q5. Young Generation vs Old Generation — why split the heap at all?

Most objects die young. So Java frequently does cheap cleanups of a small "new arrivals" area (Young Gen/Eden), and only objects that survive several rounds get promoted to Old Gen, which is cleaned less often since it's a costlier scan.
Q6. Can a Java program leak memory despite having a garbage collector?

Yes — if something stays reachable that shouldn't (a static list you keep adding to and never clear, for example). GC only frees what's truly unreachable; it can't read your intent.

# Section E: Exception Handling — Checked, Unchecked, Custom
Q1. Core distinction between checked and unchecked exceptions?

Checked exceptions (Exception, but not RuntimeException) are compiler-enforced — must catch or declare. Unchecked (RuntimeException and below) aren't enforced and usually represent actual bugs in the code, not external conditions.
Q2. throw vs throws?

throw actually throws a specific exception instance, used inside a method body. throws is just a declaration in the method signature warning that this method might throw something.
Q3. When does finally NOT run?

If System.exit() is called inside the try/catch, or if the thread itself is forcibly killed.
Q4. What's try-with-resources for, and what must the resource implement?

Automatically closes a resource after the try block finishes, even on exception. The resource must implement AutoCloseable.
Q5. Can an overridden method throw a broader checked exception than the parent declared?

No — it can throw the same, something narrower, or nothing, but never broader. (This rule doesn't apply to unchecked exceptions at all.)
Q6. What is exception chaining, and why bother with it?

Passing the original exception as the "cause" when wrapping it in a new one (super(message, cause)), so the root reason isn't lost — shows up as "Caused by:" in the stack trace.
Q7. Should a custom exception be checked or unchecked?

Checked if the caller can realistically recover (retry, fallback). Unchecked if it really represents a bug/invalid state — which is why most modern frameworks default to unchecked for custom exceptions.

----------