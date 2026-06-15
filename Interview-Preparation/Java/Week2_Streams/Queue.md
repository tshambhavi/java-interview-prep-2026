## Queue

A queue in JAVA is a linear data structure operating on First In First Out, (FIFO) principle, meaning elements are added from the back(rear) and removed from the head(front).
Queue is an interface, so it cannot be directly used to instantiate a queue. We rather use LinkedList or ArrayDeque for this purpose.

# How LinkedList Implements Queue Internally?

Each element is a Node:

Node structure:
[ data | prev | next ]

After offer(10), offer(20), offer(30):

REAR                              FRONT
 ↓                                  ↓
[30|prev=null|next→] → [20|←prev|next→] → [10|←prev|next=null]

offer() — adds new node at REAR (tail)
poll()  — removes node from FRONT (head)

This is why LinkedList is a 
Doubly Linked List —
needs to traverse both directions!

# How ArrayDeque Implements Queue Internally? (Recommended)

Uses circular array internally:

Initial:
array:  [ ][ ][ ][ ][ ][ ][ ][ ]
head=0, tail=0

After offer(10), offer(20), offer(30):
array:  [10][20][30][ ][ ][ ][ ][ ]
         ↑           ↑
        head        tail
        (front)     (rear)

After poll() — removes from head:
array:  [10][20][30][ ][ ][ ][ ][ ]
             ↑       ↑
            head    tail
            (10 logically removed,
             head pointer moved right)

Circular behaviour:
When tail reaches end of array
it wraps around to beginning:

array:  [ ][ ][30][40][50][60][70][10][20]
                                    ↑
                                   tail wrapped around!

# --------------------------------------------------------------------------------------- #

