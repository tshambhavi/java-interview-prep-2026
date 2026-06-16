## Problem 108: Convert Sorted Array to Binary Search Tree

* A height balanced binary tree is a tree where the height of the left and right subtree is atmost 1 for any node.

Balance factor: Height (Left Subtree) - Height (Right Subtree) 

A tree is a height balanced binary tree if the balance factor is -1, 0 or 1.

* Recursion is the key hint.

* Key Pattern: Sorted Data -> Pick Middle -> Recursively solve left and right half.
This is classic Divide and conquer pattern.

* If we pick the first element, naturally the array being in ascending order ends up making the all other elements go to the right half of the node (first element), which does not satisfy the height balanced binary tree criteria. Hence, we start with the middle element of array and subarrays.

# ------------------------------------------------------------- #

## Problem 118: Pascal's triangle && Problem 119: Pascal's triangle II

* Build current row using previous row.
* First and last element is always 1.
* Inner elements comes from:
prviousRow[j-1] + previousRow[j]
* Dynamic Programming thinking: Current state depends on previous state.
* Output itself is O(n^2), so time complexity O(n^2) is optimal.
* Note: Problem 119 uses the same idea, except it does not require storing all the previous rows except the one needed for current row generation.

# ------------------------------------------------------------- #

## Problem 219: Contains Duplicate II

* For all the unique numbers, we need to keep track of the indices of the previously encountered elements.
For example: 1,0,1,1 and k=1.

Now when we encounter the second "1", we need to keep track of previously encountered "1" (same element) which is at index 0. This will help determine the difference between indicest to compare with k. 
If the condition is not satisfied, going further away from previously encountered "1" (index 0) will increase the difference, eventually failing. So we need to update the this previously encountered index to current and find the next same element to check (if we do). This has to be followed for all unique elements until we find the right indices (if we do else false).

* HashMap is often the first choice for faster lookup of previously seen values.
* Store: Value: Last seen index
Value is the key and last seen index is the value in the map.

If a problem asks:
"Have I seen this before?"
Think:
HashSet

If a problem asks:
"Have I seen this before and where?"
Think:
HashMap<Value, Index>

# ------------------------------------------------------------- #

## Problem 228: Summary Ranges

* Key idea is to remember the starting point. (Store the start).
* Keep iterating until you find the end. 
 How do we find the end?
When we encounter a number num[i] such that num[i-1]+1!=num[i]. That is when we need to process the range "start->num[i-1]" (num[i-1] is end).
* Handle single element range. Keep check of the end element -> Means store the end temporarily. If start==end, then it's a single element range. Process "start or end" only, else "start->end".

# Similar Problems (Homework): 
56. Merge Intervals
57. Insert Interval
163. Missing Ranges
128. Longest Consecutive Sequence

# ------------------------------------------------------------- #

## Problem 268: Missing Number

* I remember this problem, because i missed that all the arrays were starting from 0. I sorted the array and iterated over it until I found the missing element. (nums[i]+1==nums[i]? If true continue the loop, if not then you got your answer honey).

* And don't forget to handle the corner case where probably the missing element is 0 and if your array doesn't have 0 then return 0.

* I could've simply calculated the sum of a range of numbers using the formula: 
(n(n+1))/2
AND THEN I COULD'VE SUBTRACTED ALL THE EXISING ELEMENTS TO FIND THE MISSING NUMBER. 
But whatever. Huh.

* So the key idea here is try not to miss anything and and and remember the formula. You don't know when it might come in use.

# ------------------------------------------------------------- #

## Problem 283: Move Zereos



