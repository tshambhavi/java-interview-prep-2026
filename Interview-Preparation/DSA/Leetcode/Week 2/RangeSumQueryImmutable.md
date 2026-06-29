# Problem 303: Range Sum Query - Immutable

Since the array NEVER changes, you can precompute
the running total ONCE in the constructor, and then
every query becomes a simple subtraction — O(1).

This is the prefix sum technique:
prefix[k] = sum of all elements from index 0 to k-1

So:
sumRange(i, j) = prefix[j+1] - prefix[i]

## Why This Formula Works

nums:    [-2,  0,  3, -5,  2, -1]
index:     0   1   2   3   4   5

prefix:  [0, -2, -2,  1, -4, -2, -3]
index:    0   1   2   3   4   5   6

prefix[k] = sum of nums[0..k-1]
prefix[0] = 0                      (empty sum)
prefix[1] = -2                     (just nums[0])
prefix[2] = -2 + 0 = -2            (nums[0] + nums[1])
prefix[3] = -2 + 0 + 3 = 1         (nums[0..2])
prefix[4] = 1 + (-5) = -4          (nums[0..3])
prefix[5] = -4 + 2 = -2            (nums[0..4])
prefix[6] = -2 + (-1) = -3         (nums[0..5])

Now, to get sumRange(2, 5) — sum of nums[2] through nums[5]:

prefix[6] = sum of nums[0..5]  = -3
prefix[2] = sum of nums[0..1]  = -2

sumRange(2,5) = prefix[6] - prefix[2] = -3 - (-2) = -1 ✅

Why does subtracting work?
prefix[6] already includes nums[0] + nums[1] + ... + nums[5]
prefix[2] includes only nums[0] + nums[1]

Subtracting prefix[2] REMOVES nums[0] and nums[1],
leaving exactly nums[2] + nums[3] + nums[4] + nums[5]
— which is precisely the range you wanted!

## Why prefix Has Length n + 1, Not n ?

This trips a lot of people up. The +1 extra slot
(prefix[0] = 0) exists specifically so that
sumRange(0, j) doesn't need a special case.

Without prefix[0] = 0:
sumRange(0, j) would need: "just prefix[j+1], no subtraction"
                            ← a special case to handle!

With prefix[0] = 0:
sumRange(0, j) = prefix[j+1] - prefix[0] = prefix[j+1] - 0
                            ← same formula as everything else!


```java
class NumArray {
    private int[] prefix;

    public NumArray(int[] nums) {
        prefix = new int[nums.length + 1];
        prefix[0] = 0;
        for (int i = 0; i < nums.length; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }

    public int sumRange(int i, int j) {
        return prefix[j + 1] - prefix[i];
    }
}
```


## Interview Questions
Q1. Why use a prefix sum array instead of computing the sum directly each time?

A: Because the array is immutable, you can precompute cumulative sums once in O(n), making every subsequent sumRange() call O(1) instead of recomputing a possibly large range every time.
Q2. Why is the prefix array of size n+1 instead of n?

A: The extra slot, prefix[0] = 0, lets you compute sumRange(0, j) using the same formula as every other query, without a special case for ranges starting at index 0.
Q3. What is the formula for sumRange(i, j) using a prefix array, and why does it work?

A: prefix[j+1] - prefix[i]. prefix[j+1] holds the sum of everything from index 0 to j; prefix[i] holds the sum of everything from 0 to i-1. Subtracting removes exactly the part you don't want, leaving the sum from i to j.
Q4. What would happen to this approach if the array were mutable (could be updated)?

A: The prefix sum approach breaks down — updating one element would require recomputing the prefix array from that index onward, making updates O(n). That's why the mutable version of this problem (LeetCode 307) typically uses a Segment Tree or Fenwick Tree instead, to get O(log n) updates and queries.
Q5. What's the time/space trade-off of this approach?

A: O(n) time and space upfront to build the prefix array, in exchange for O(1) time per query afterward — ideal when there will be many queries on a fixed array.

## One Line Summary
Precompute cumulative sums once (O(n)), then answer
any range sum query in O(1) via prefix[j+1] - prefix[i].
Only valid because the array never changes.

----------