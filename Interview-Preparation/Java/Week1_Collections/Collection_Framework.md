--Collection Heirarchy--
Iterable
└── Collection
    ├── List
    │   ├── ArrayList
    │   ├── Vector
    │   │   └── Stack
    │   └── LinkedList
    │
    ├── Queue
    │   ├── Deque
    │   │   ├── ArrayDeque
    │   │   └── LinkedList
    │   └── PriorityQueue
    │
    └── Set
        ├── HashSet
        │   └── LinkedHashSet
        │
        └── SortedSet
            └── NavigableSet
                └── TreeSet

## Most Important Collection Implementations for Interviews

List:
- ArrayList
- LinkedList

Set:
- HashSet
- LinkedHashSet
- TreeSet

Queue:
- PriorityQueue
- ArrayDeque

Map (Separate Hierarchy):
- HashMap
- LinkedHashMap
- TreeMap
- Hashtable
- ConcurrentHashMap

# ******************************************************************* #

Collection Framework provides a set of ready-made data structures for data manipulation and organization without writing code from scratch.

- Allows dynamic resizing
- Provides interfaces such as Collections, Set, Map, List
- Provides ready to use data structures (ArrayList, Hashset, Hashmap)
- Provides built-in algorithms (such as searching, sorting, iteration)
- Improves Code reusability by reducing boilerplate code.

We cannot create an object of an interface directly, we create an object of the class Arraylist implementing the interface and assign it to the interface reference.
Collection<String> collection  = new ArrayList<>();

## Ordered Collection ##
- Maintains the insertion order
- Accessed via numerical indexes
- Duplicates generally allowed

## Unordered Collection ##
- Unpreditcable or randomized order
- Accessed by key or value matching
- Duplicates not allowed

# List
- Ordered Collection
- Allows Duplicates
- Accessed via numerical indexes
- Implemented using ArrayList, Vector, Stack, LinkedList classes

Declaration:
public interface List<E> extends Collection<E>

# Set
- Unordered Collection
- Duplicates not allowed
- Implemented using HashSet, TreeSet, LinkedHashSet, EnumSet, CopyOnWriteArraySet

Declaration:
public interface Set<E> extends Collection<E>

# SortedSet
- Extends Set and maintains elements in sorted order
- Provides methods for range-based operations
- Implementing Class: TreeSet

Declaration:
public interface SortedSet<E> extends Set<E>

# NavigableSet
- Extends SortedSet
- Provides navigable methods such as lower(), floor(), ceiling(), and higher()
- Implementing Class: TreeSet

Declaration:
public interface NavigableSet<E> extends SortedSet<E>

# Queue
- Follows FIFO (First In First Out) Approach
- Implementing classes: PriorityQueue, DeQueue, LinkedList

Declaration:
public interface Queue<E> extends Collections<E>

# Deque
 - Extends Queue
 - Allows operations(addition/removal) from both ends of the queue
 - Implementing classes: ArrayDeque, LinkedList

Declaration:
public interface Deque<E> extends Queue<E>

## Map ##
* Map interface does not extend Collections interface because it has different structure element, that is, key-value pair and not a single value element so the normal in-built methods will not work on Map. 
* Requires a different set of operations than a standard single-element operations.

The Collection view methods allows Map to be viewed as a Collection.
1. keySet - The set of keys in the Map.
2. values - Collection of values contained in the Map. This collection is not a set because multiple keys can map to the same value.
3. entrySet - the set of key-value pairs contained in the Map. The Map interface provides a small nested interface called Map.Entry, the type of elements in this is set.

* The collection views provides the only means to iteratre over a Map. (For ex.: iterate using keySet)

# ******************************************************************** #


