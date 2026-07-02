# Third Maximum Number (LeetCode 414)

## The Problem, In Plain Terms

Given an integer array, return the **third distinct maximum** number.
If a third distinct maximum does not exist, return the **largest** number instead.

```
Input:  [3, 2, 1]
Output: 1
Reason: Third distinct maximum is 1

Input:  [1, 2]
Output: 2
Reason: No third distinct maximum exists → return the largest (2)

Input:  [2, 2, 3, 1]
Output: 1
Reason: Distinct values are {1, 2, 3}
        Third distinct maximum = 1
        (duplicates don't count as separate maximums!)
```

---

## Key Observations Before Coding

```
1. The word DISTINCT is critical
   → [2, 2, 3, 1] has only THREE distinct values: 1, 2, 3
   → Third distinct max = 1, NOT 2 (which would be wrong
     if you counted the duplicate 2 as a separate value)

2. If fewer than 3 distinct values exist → return the MAX
   → [1, 2] → only 2 distinct values → return 2 (the max)

3. Negative numbers are valid input
   → [-1, -2, -3] → third max = -3 (fully valid!)
   → This rules out using 0 or Integer.MIN_VALUE as
     "not found" sentinels safely — need a different approach
```

---

## Approach 1 — Using TreeSet (Cleanest)

### The Idea

```
TreeSet stores elements in SORTED ORDER automatically
and handles duplicates (ignores them, like all Sets)

Step 1: Add all elements to a TreeSet
        → duplicates removed automatically
        → elements sorted in ascending order

Step 2: If TreeSet has fewer than 3 elements
        → return the LAST element (the maximum)

Step 3: Otherwise remove the two largest elements
        (pollLast() removes and returns the largest)
        → what remains at the top is the third maximum
```

### Code

```java
import java.util.*;

class Solution {
    public int thirdMax(int[] nums) {

        TreeSet<Integer> set = new TreeSet<>();

        for (int num : nums) {
            set.add(num);   // duplicates ignored automatically
        }

        if (set.size() < 3) {
            return set.last();   // return the maximum
        }

        set.pollLast();   // remove 1st maximum
        set.pollLast();   // remove 2nd maximum
        return set.last(); // 3rd maximum is now at the top
    }
}
```

### Walking Through Examples

```
Example 1: [3, 2, 1]
TreeSet = {1, 2, 3}   (sorted ascending)
size = 3 → proceed
pollLast() → removes 3 → set = {1, 2}
pollLast() → removes 2 → set = {1}
last()     → returns 1 ✅

Example 2: [1, 2]
TreeSet = {1, 2}
size = 2 < 3 → return last() = 2 ✅

Example 3: [2, 2, 3, 1]
TreeSet = {1, 2, 3}   (duplicate 2 ignored)
size = 3 → proceed
pollLast() → removes 3 → set = {1, 2}
pollLast() → removes 2 → set = {1}
last()     → returns 1 ✅

Example 4 (negative numbers): [-1, -2, -3]
TreeSet = {-3, -2, -1}
size = 3 → proceed
pollLast() → removes -1 → set = {-3, -2}
pollLast() → removes -2 → set = {-3}
last()     → returns -3 ✅
```

---

## Approach 2 — Three Variables (Optimal Space, O(n) Time)

### The Idea

```
Track only three variables:
first  = largest value seen so far
second = second largest distinct value
third  = third largest distinct value

All initialized to null (using Long or Integer wrapper
to handle the case where actual values could be
Integer.MIN_VALUE — a common edge case trap!)

Iterate through the array, updating the three variables
whenever a new distinct value is found that fits
```

### Code

```java
class Solution {
    public int thirdMax(int[] nums) {
        Integer first = null, second = null, third = null;

        for (int num : nums) {
            // skip duplicates — if num already equals any of the three, ignore
            if ((first != null && num == first) ||
                (second != null && num == second) ||
                (third != null && num == third)) {
                continue;
            }

            if (first == null || num > first) {
                // new largest — shift everything down
                third = second;
                second = first;
                first = num;
            } else if (second == null || num > second) {
                // new second largest — shift second and third down
                third = second;
                second = num;
            } else if (third == null || num > third) {
                // new third largest
                third = num;
            }
        }

        // if third is still null, fewer than 3 distinct values existed
        return third == null ? first : third;
    }
}
```

### Walking Through Example 3: [2, 2, 3, 1]

```
Start: first=null, second=null, third=null

num=2:
  No duplicates to skip
  first == null → first=2
  State: first=2, second=null, third=null

num=2:
  num == first (2 == 2) → SKIP (duplicate!)

num=3:
  No duplicates
  3 > first(2) → third=null, second=2, first=3
  State: first=3, second=2, third=null

num=1:
  No duplicates
  1 < first(3), 1 < second(2)
  third == null → third=1
  State: first=3, second=2, third=1

third != null → return third = 1 ✅
```

### Why Use Integer (wrapper) Instead of int (primitive)?

```
If we used int and initialized to Integer.MIN_VALUE
as a "not found" sentinel:

What if the array contains Integer.MIN_VALUE itself?
[-2147483648, 1, 2]

first  = Integer.MIN_VALUE (initialized, not "not found")
→ We'd WRONGLY think we already have a third maximum
  when we actually haven't seen a third value yet!

Using Integer (nullable wrapper) lets null mean
"not found yet" cleanly, with NO sentinel value needed
This correctly handles Integer.MIN_VALUE as an actual
array element
```

---

## Approach Comparison

| Feature | TreeSet Approach | Three Variables Approach |
|---|---|---|
| Time Complexity | O(n log n) — TreeSet insertions | O(n) — single pass |
| Space Complexity | O(n) — stores all distinct elements | O(1) — just 3 variables |
| Code simplicity | Simpler, cleaner to read | More logic, but optimal |
| Handles duplicates | Automatically via Set | Manual duplicate check |
| Handles negatives | Yes | Yes (using Integer wrapper) |
| Best when | Readability preferred | Optimal space/time needed |

---

## The Integer.MIN_VALUE Trap — Common Interview Mistake

```java
// WRONG APPROACH — fails for arrays containing Integer.MIN_VALUE
int first  = Integer.MIN_VALUE;
int second = Integer.MIN_VALUE;
int third  = Integer.MIN_VALUE;

// Input: [1, 2, Integer.MIN_VALUE]
// After processing:
// first = 2, second = 1, third = Integer.MIN_VALUE

// Is third a "real" third maximum, or just uninitialized?
// We CAN'T TELL! The sentinel value collides with real data.

// This returns Integer.MIN_VALUE (correct by accident here,
// but WRONG logic — would fail for [1, 2] where third
// should stay "not found" but appears to be Integer.MIN_VALUE)
```

```java
// CORRECT APPROACH — use nullable Integer
Integer third = null;
// null unambiguously means "not found yet"
// Integer.MIN_VALUE as an actual value is handled correctly
```

---

## Edge Cases to Consider

```
1. All duplicates: [5, 5, 5]
   → Only 1 distinct value → return 5 (the max)

2. Exactly 3 distinct values: [3, 2, 1]
   → Third max = 1

3. More than 3 distinct values: [5, 4, 3, 2, 1]
   → Third max = 3

4. Negative numbers: [-1, -2, -3]
   → Third max = -3

5. Mixed positive and negative: [-1, 1, 0]
   → Third max = -1

6. Contains Integer.MIN_VALUE: [1, 2, Integer.MIN_VALUE]
   → Third distinct max = Integer.MIN_VALUE ← must work correctly!

7. Two elements: [1, 2]
   → No third max → return 2 (the largest)

8. One element: [1]
   → No third max → return 1
```

---

## Complexity Summary

| Approach | Time | Space |
|---|---|---|
| TreeSet | O(n log n) | O(n) |
| Three Variables | O(n) | O(1) |

---

## Interview Questions

**Q1. What does "distinct" mean in this problem, and why does it matter?**
A: Distinct means unique values only — duplicates are not counted separately. So [2, 2, 3, 1] has only three distinct values {1, 2, 3}. Without understanding this, you might mistakenly count the second 2 as the third maximum.

**Q2. Why is using Integer.MIN_VALUE as a sentinel value dangerous here?**
A: Because Integer.MIN_VALUE can legitimately appear as an element in the array. If you initialize `third = Integer.MIN_VALUE` to mean "not found," you can't tell if it was genuinely set from actual data or just never updated — causing incorrect results.

**Q3. Why use Integer (wrapper class) instead of int (primitive) for the three variables approach?**
A: Integer can be null, which unambiguously means "not found yet." This avoids the sentinel value problem entirely and correctly handles Integer.MIN_VALUE as a real array element.

**Q4. Why does TreeSet work well for this problem?**
A: TreeSet maintains elements in sorted order and ignores duplicates automatically. This makes finding the third maximum as simple as removing the two largest elements and returning what remains at the top.

**Q5. What is the time complexity of each approach?**
A: TreeSet: O(n log n) due to sorted insertions. Three variables: O(n) since it's a single linear pass with O(1) work per element.

**Q6. What should be returned if fewer than 3 distinct values exist?**
A: The maximum value in the array — not the third slot, not an error, just the single largest value.

**Q7. If the interviewer asks for O(1) space, which approach do you choose?**
A: The three variables approach — it tracks only first, second, and third maximum in three variables, using no extra data structures proportional to input size.

---

## One Line Summary

```
Track three distinct maximums using either a TreeSet
(simpler, O(n log n)) or three nullable Integer variables
(optimal, O(n)). Use null — not Integer.MIN_VALUE — as
the "not found" sentinel to safely handle all edge cases.
```

---
