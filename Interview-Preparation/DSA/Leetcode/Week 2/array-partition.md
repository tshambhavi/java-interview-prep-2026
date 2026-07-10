# Problem 561 - Array Partition

## Recognition Pattern:
--------------------
Maximize the sum of the smaller element in each pair.

## Key Observation:
----------------
Sort the array first.

## Greedy Strategy:
----------------
Pair adjacent elements.

## Reason:
-------
Sorting prevents large numbers from being "wasted"
by pairing them with very small numbers.

## Implementation:
---------------
Sort
↓
Take every alternate element
(index 0, 2, 4, ...)
↓
Return their sum

## Complexity:
-----------
Time  : O(n log n)
Space : O(1)

-----------

This is a classic greedy problem. The trick is recognizing that once the array is sorted, the answer is simply the sum of every even-indexed element.

-----------