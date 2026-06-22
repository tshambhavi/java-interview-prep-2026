# Multithreading

* Multithreading in Java is a built-in feature that allows the concurrent execution of two or more parts of a program to maximize CPU utilization. A thread is a lightweight sub-process, the smallest unit of execution within an application, which shares a common memory space with other threads under the same process.

* Two ways to create threads:
1. Extending the Thread Class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running via extending Thread class.");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start(); // Allocates call stack and invokes run() asynchronously
    }
}


2. Implementing the Runnable Interface
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread running via Runnable interface.");
    }
}

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start(); 
    }
}
* Tip: Modern Java versions allow shortcut setups using concise Lambda expressions: Thread thread = new Thread(() -> System.out.println("Running")); thread.start();

## Why Runnable is Preferred Over Extending Thread?

🎯 KEY REASON: Java doesn't support multiple inheritance!

If your class extends Thread:
class MyTask extends Thread { }
→ It CANNOT extend any other class
→ Limits flexibility in design

If your class implements Runnable:
class MyTask implements Runnable { }
→ It CAN still extend another class if needed
→ More flexible, follows "favor composition over inheritance"

Example showing the limitation:
class MyTask extends Thread, extends SomeOtherClass { }
❌ COMPILE ERROR — Java doesn't allow this!

class MyTask extends SomeOtherClass implements Runnable { }
✅ This works perfectly fine!

## start() vs run() — Critical Interview Trap!

Thread t1 = new Thread(() -> System.out.println("Running"));

t1.start();   // ✅ Creates NEW thread, then calls run() on it
t1.run();     // ❌ Just calls run() like a NORMAL method call
              //    NO new thread is created!

 KEY DIFFERENCE:

* start():
→ Creates a new call stack (new thread)
→ JVM allocates thread resources
→ Eventually calls run() internally
→ Multiple calls to start() throw IllegalThreadStateException

* run():
→ Just a normal method call
→ Executes on the SAME thread that called it
→ No new thread created
→ Can be called multiple times like any method

## Thread Lifecycle — The 6 States

start()
        NEW  ──────────────────►  RUNNABLE
                                       │
                                       │ thread scheduler
                                       │ picks this thread
                                       ▼
                                   RUNNING
                                   /    |    \
                          wait()  /     |     \  sleep()/
                          sleep() /      |      \  block on I/O
                                 ▼      |       ▼
                            WAITING/   |    TIMED_WAITING
                            BLOCKED    |
                                 \     |     /
                                  \    |    /
                              notify()/  run() completes
                              available|
                                       ▼
                                  TERMINATED

Thread.State states:

1. NEW
   Thread object created but start() not called yet
   
   Thread t = new Thread();  // currently in NEW state

2. RUNNABLE
   start() has been called
   Thread is ready to run OR actually running
   (Java doesn't distinguish "ready" vs "running" as separate states!)
   
   t.start();  // now in RUNNABLE state

3. BLOCKED
   Thread is waiting to acquire a lock 
   (e.g., trying to enter synchronized block held by another thread)

4. WAITING
   Thread is waiting indefinitely for another thread 
   to perform a particular action
   Caused by: wait(), join() without timeout, 
   LockSupport.park()

5. TIMED_WAITING
   Thread is waiting for a specified time period
   Caused by: sleep(ms), wait(ms), join(ms)

6. TERMINATED
   Thread has completed execution
   (either finished normally or due to exception)


## Important Thread Methods

Thread t = new Thread(() -> { /* task */ });

t.start();           // starts the thread
t.join();            // current thread waits for t to finish
t.sleep(1000);        // static method, pauses current thread for 1000ms
t.getName();          // returns thread name
t.setName("Worker");  // sets thread name
t.getId();            // returns unique thread ID
t.isAlive();          // true if thread has started but not terminated
t.interrupt();        // interrupts thread (sends interrupt signal)
t.setPriority(5);      // sets priority (1 to 10, default 5)
t.getPriority();       // returns current priority
t.isDaemon();          // checks if thread is daemon
t.setDaemon(true);     // marks thread as daemon (background thread)

## join() - Very common interview question!

public class Test {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                System.out.println("Thread: " + i);
            }
        });
        
        t1.start();
        t1.join();    // main thread WAITS here until t1 finishes
        
        System.out.println("Main thread continues after t1 finishes");
    }
}

// WITHOUT join():
// Main thread might print "Main thread continues" 
// BEFORE t1 finishes printing — unpredictable order!

// WITH join():
// Guaranteed: all of t1's output prints FIRST,
// then main thread continues

## Daemon Thread

* A daemon thread in Java is a low-priority background thread that provides supporting services to user (non-daemon) threads. Its most defining characteristic is that the JVM automatically terminates and exits when only daemon threads remain active, meaning it will not wait for daemon threads to finish their execution.

* Example: Garbage Collector is a daemon thread!

* If ALL non-daemon (user) threads finish, JVM exits immediately — even if daemon threads are still running.

Thread t = new Thread(() -> {
    while (true) {
        System.out.println("Daemon running...");
    }
});

t.setDaemon(true);   // MUST be called BEFORE start()!
t.start();

// Main thread finishes quickly
// JVM exits, daemon thread is KILLED mid-execution
// even though its while(true) loop never naturally ends

⚠️ Important: setDaemon() MUST be called 
BEFORE start(), otherwise throws 
IllegalThreadStateException

## Thread Priority

Thread t1 = new Thread(() -> { /* task */ });
t1.setPriority(Thread.MAX_PRIORITY);   // 10
t1.setPriority(Thread.MIN_PRIORITY);   // 1
t1.setPriority(Thread.NORM_PRIORITY);  // 5 (default)

⚠️ Important: Priority is just a HINT to the 
thread scheduler, NOT a guarantee!

Actual scheduling depends on OS and JVM implementation
Higher priority does NOT guarantee execution order

## Multiple start() calls

Thread t = new Thread(() -> System.out.println("Running"));

t.start();   // ✅ Works fine
t.start();   // ❌ throws IllegalThreadStateException!

// WHY?
// Once a thread completes (TERMINATED state),
// it CANNOT be restarted
// A NEW Thread object must be created instead

# --------------------------------------------------------------- #