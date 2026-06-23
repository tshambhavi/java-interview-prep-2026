# Synchronization

* Synchronization in multithreading is a mechanism that controls the execution and access of multiple threads to shared resources.
* It ensures that only one thread can access a critical section of code or resource at a time, preventing data inconsistency, race conditions, and corrupted states.

## Why synchronization is needed?

* In multithreaded applications, threads execute simultaneously and asynchronously. If multiple threads attempt to read and modify the same variable or object concurrently, their operations may interleave.
Race Conditions: Occur when the final outcome of a program depends on the unpredictable sequence in which threads are executed.
Data Inconsistency: A thread might overwrite the changes of another thread because it was reading outdated data from its local cache rather than the main memory.

