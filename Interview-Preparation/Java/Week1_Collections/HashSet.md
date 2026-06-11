### HashSet

- Stores unique elements
- Internally backed by HashMap
- Offers exceptional time complexity but does not preserve the insertion order.
- Permits one null element
- O(1) time complexity for add, remove and contains
- Not Thread-safe, it is not synchronized, for simultaneous updates, handle externally.


## Iteration Strategies:
HashSet is not index-based so in order to iterate, there are other approaches to follow:
1. Enhanced For-Each Loop

for (String lang : set) {
    System.out.println(lang);
}


2. Iterator Interface:

java.util.Iterator<String> it = set.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

3. Java 8+ Stream / forEach:

set.forEach(System.out.println);


## Internal Implementation
1. When set.add(element) is called, HashSet puts that object as key in the backing hashmap.
2. The corresponding value of the added key has dummy object internally defined as PRESENT.
3. HashMap keys are unique and do not allow duplicates, enabling the HashSet to reject if a duplicate entry is tried.
4. The default capacity of the HashSet is 16 and the load factor is 0.75 before it automatically resizes.

## ----------------------------------------------------------- ##

