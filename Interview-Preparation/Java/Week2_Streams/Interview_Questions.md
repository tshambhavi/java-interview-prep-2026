# Section A: Stack, Queue, Deque - Internals + Use Cases
Q1. How is Java's Stack class implemented internally?
A:  Stack extends Vector, which uses a resizable 
    synchronized array internally. Adds push, pop, 
    peek, empty, search methods on top of Vector.

Q2. Why is ArrayDeque preferred over Stack in modern Java?
A:  Stack extends Vector which is synchronized, 
    making it slower. ArrayDeque is faster, not 
    thread safe, but suitable for single-threaded use.

Q3. What is the internal structure of ArrayDeque?
A:  A resizable circular array with head and tail 
    pointers. Initial capacity is always a power of 2 
    (default 8), doubles when full.
    Default Capacity is 16 but minimum threshold is 8.
    If no size is provided while initializing the size is 16, if a minimum of 2 or 4 or such number is provided then the size initialized is 8.

Q4. Why does ArrayDeque capacity have to be a power of 2?
A:  So it can use bitwise AND (&) instead of modulo (%) 
    for circular index calculation — significantly 
    faster than modulo operation.

Q5. What is the difference between Array implementation 
    and LinkedList implementation of a Stack?
A:  Array: fixed size, contiguous memory, cache friendly, 
    risk of Stack Overflow.
    LinkedList: dynamic size, scattered memory, extra 
    pointer overhead, no overflow risk.

Q6. What is the difference between Queue and Deque?
A:  Queue is single-ended (FIFO) — add rear, remove front.
    Deque is double-ended — add/remove from both ends.
    Deque extends Queue and can function as a Stack too.

Q7. How does LinkedList implement Queue internally?
A:  Uses a doubly linked list. offer() adds node at tail, 
    poll() removes node from head. Each node has data, 
    prev, and next pointers.

Q8. Why is ArrayDeque generally faster than LinkedList 
    for Queue/Stack operations?
A:  ArrayDeque uses contiguous array memory (cache 
    friendly). LinkedList nodes are scattered across 
    heap memory, causing cache misses.

Q9. Can ArrayDeque store null elements?
A:  No — throws NullPointerException. null is used 
    internally to signal an empty slot. LinkedList 
    allows null values.

Q10. When would you choose LinkedList over ArrayDeque 
     despite it being slower?
A:   When you need null elements stored, or when you 
     need List-specific operations (index-based access) 
     alongside Queue/Deque behavior.

# Section B: Iterator, ListIterator, For-Each — Internals

Q1. How is Iterator implemented internally in ArrayList?
A:  ArrayList has a private inner class "Itr" implementing 
    Iterator. It maintains cursor, lastRet, and 
    expectedModCount. hasNext() checks cursor != size.

Q2. What is fail-fast iterator behavior?
A:  Throws ConcurrentModificationException if the 
    collection is structurally modified during iteration 
    by any means other than the iterator's own remove(). 
    Detected via modCount vs expectedModCount mismatch.

Q3. What is modCount and when does it increase?
A:  A counter tracking structural modifications 
    (add, remove, clear). It does NOT increase on set() 
    since that's a value change, not structural.

Q4. Why should you use Iterator's remove() instead of 
    Collection's remove() during iteration?
A:  Iterator's remove() updates expectedModCount = 
    modCount internally, keeping them in sync. 
    Collection's remove() only updates modCount, leaving 
    expectedModCount stale, causing an exception.

Q5. What is the difference between Iterator and 
    ListIterator?
A:  Iterator: forward only, works on all collections.
    ListIterator: forward AND backward, only on List 
    implementations. Also has add(), set(), nextIndex(), 
    previousIndex().

Q6. Is ListIterator fail-fast too?
A:  Yes — same modCount mechanism. Its add() and set() 
    methods also update expectedModCount to stay in sync.

Q7. How does the for-each loop work internally for 
    Collections vs Arrays?
A:  For Collections: compiler converts it to use 
    Iterator (hasNext/next loop).
    For Arrays: compiler converts it to a traditional 
    index-based for loop, since arrays don't implement 
    Iterable.

Q8. How does ArrayList's forEach() differ internally 
    from using a for-each loop with Iterator?
A:  ArrayList overrides forEach() to use a direct 
    index-based loop over the internal array, bypassing 
    Iterator object creation entirely — more efficient.

Q9. What's the difference between fail-fast and 
    fail-safe iterators?
A:  Fail-fast: works on original collection, throws 
    exception if modified (ArrayList, HashMap).
    Fail-safe: works on a clone, no exception, may not 
    reflect latest changes (ConcurrentHashMap, 
    CopyOnWriteArrayList).

Q10. Is Iterator thread safe?
A:   No — Iterator is fail-fast, not thread safe. For 
     thread safety, use ConcurrentHashMap or 
     CopyOnWriteArrayList iterators (fail-safe).

# Section C: Comparable vs Comparator

Q1. What is the core difference between Comparable 
    and Comparator?
A:  Comparable defines natural ordering INSIDE the class 
    itself using compareTo(). Comparator defines custom 
    ordering OUTSIDE the class using compare().

Q2. Can a class have multiple sorting orders using 
    Comparable?
A:  No — a class can implement Comparable only once, 
    giving exactly ONE natural ordering. For multiple 
    orderings, use multiple Comparator implementations.

Q3. What does compareTo() return and what do the 
    return values mean?
A:  Returns an int. Negative = this object comes before, 
    zero = equal, positive = this object comes after 
    the compared object.

Q4. How do you sort a list by multiple fields using 
    Comparator?
A:  Use Comparator chaining:
    Comparator.comparing(Employee::getDept)
        .thenComparing(Employee::getSalary)

Q5. What happens if you don't override equals() when 
    implementing Comparable?
A:  compareTo() and equals() can become inconsistent — 
    sorted collections (TreeSet, TreeMap) use compareTo() 
    for equality checks, which can cause unexpected 
    duplicate handling.

Q6. Which collections internally rely on Comparable 
    or Comparator?
A:  TreeMap and TreeSet use compareTo() (Comparable) by 
    default, or a Comparator if explicitly provided in 
    their constructor.

Q7. Can you use a lambda expression for Comparator?
A:  Yes — since Comparator is a functional interface:
    Comparator<Employee> byName = (e1, e2) -> 
        e1.getName().compareTo(e2.getName());

# Section D: Streams, Lambda, Functional Interfaces

Q1. What is a Functional Interface?
A:  An interface with exactly ONE abstract method 
    (can have multiple default/static methods). 
    Enables lambda expression usage.

Q2. Why are Streams lazy?
A:  Intermediate operations (filter, map) don't execute 
    until a terminal operation (collect, forEach) is 
    called — allows efficient single-pass processing.

Q3. Can a Stream be reused after a terminal operation?
A:  No — once consumed, calling another operation throws 
    IllegalStateException. Streams are single-use.

Q4. What is the difference between map() and flatMap()?
A:  map() transforms each element 1-to-1. flatMap() 
    transforms AND flattens nested structures into a 
    single stream (e.g., List<List<Integer>> → 
    List<Integer>).

Q5. How does reduce() work internally?
A:  Takes an identity value and accumulator function, 
    combining elements one by one:
    reduce(0, (a,b) -> a+b) on [1,2,3] = ((0+1)+2)+3 = 6

Q6. What is the actual processing order in a Stream 
    pipeline — element by element, or operation by 
    operation?
A:  Element by element — each element flows through the 
    ENTIRE pipeline (filter→map→collect) before the next 
    element starts, due to laziness.

Q7. What is the difference between findFirst() and 
    findAny()?
A:  findFirst() always returns the first matching 
    element in order. findAny() may return any matching 
    element — can be faster in parallel streams.

Q8. Are Streams thread safe?
A:  Sequential streams aren't inherently thread safe but 
    are safe if the source isn't modified concurrently. 
    Parallel streams are designed for multi-threaded 
    processing via fork/join.

# Section E: Optional, Method References, forEach

Q1. Why was Optional introduced in Java 8?
A:  To make absence of value explicit in the type system, 
    forcing callers to handle the empty case instead of 
    risking NullPointerException.

Q2. What is the difference between Optional.of() and 
    Optional.ofNullable()?
A:  Optional.of(value) throws NPE immediately if value is 
    null. Optional.ofNullable(value) safely wraps null 
    into an empty Optional.

Q3. What is the difference between orElse() and 
    orElseGet()?
A:  orElse() takes a value, evaluated EAGERLY even if 
    Optional has a value. orElseGet() takes a Supplier, 
    evaluated LAZILY only when Optional is empty. 
    orElseGet() preferred for expensive default 
    computation.

Q4. Should Optional be used as a class field or method 
    parameter?
A:  No — considered an anti-pattern. Optional is designed 
    for return types only.

Q5. What are the four types of method references?
A:  1. Static: ClassName::staticMethod
    2. Bound instance: object::method
    3. Unbound instance: ClassName::method
    4. Constructor: ClassName::new

Q6. Can every lambda be converted to a method reference?
A:  No — only lambdas that do NOTHING but call a single 
    existing method. Lambdas with extra logic or multiple 
    statements cannot be method references.

Q7. How does ArrayList's forEach() differ from the 
    default Iterable.forEach()?
A:  Default Iterable.forEach() uses the standard for-each 
    (Iterator-based) loop. ArrayList overrides it to use 
    direct index-based array access for better 
    performance.

Q8. Is using isPresent() + get() on Optional considered 
    good practice?
A:  No — defeats the purpose of Optional, essentially a 
    disguised null check. Prefer ifPresent(), map(), or 
    orElse() for functional-style handling.

# ------------------------------------------------------- #