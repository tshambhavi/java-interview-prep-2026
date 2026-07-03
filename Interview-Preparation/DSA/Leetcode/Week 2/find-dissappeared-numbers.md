# Find All Numbers Disappeared in an Array (LeetCode 448)

## The Problem, In Plain Terms

Given an array of `n` integers where each integer is in the range `[1, n]`,
some numbers may appear twice and others may be missing entirely.
Return a list of all numbers in the range `[1, n]` that do NOT appear in the array.

```
Input:  [4, 3, 2, 7, 8, 2, 3, 1]
Output: [5, 6]
Reason: n = 8, so numbers should be 1 through 8
        5 and 6 are missing — 2 and 3 appear twice instead

Input:  [1, 1]
Output: [2]
Reason: n = 2, so numbers should be 1 and 2
        1 appears twice, 2 is missing
```

---

## Key Observations Before Coding

```
1. Array length is n, values are in range [1, n]
   → This is a STRONG hint — there's a connection between
     the VALUE of each element and its potential INDEX

2. Some numbers appear TWICE (causing others to be missing)
   → Total count stays the same, just wrong distribution

3. Follow-up challenge: Can you do it in O(n) time
   and O(1) extra space (ignoring the output list)?
   → The index-marking trick exists for exactly this reason
```

---

## Approach 1 — Using a HashSet (Simple and Clear)

### The Idea

```
Step 1: Put all elements of nums into a HashSet
Step 2: Iterate from 1 to n
        If a number is NOT in the set → it's missing → add to result
Step 3: Return result
```

### Code

```java
import java.util.*;

class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        Set<Integer> seen = new HashSet<>();

        for (int num : nums) {
            seen.add(num);
        }

        List<Integer> result = new ArrayList<>();
        for (int i = 1; i <= nums.length; i++) {
            if (!seen.contains(i)) {
                result.add(i);
            }
        }

        return result;
    }
}
```

### Walking Through Example 1

```
nums = [4, 3, 2, 7, 8, 2, 3, 1],  n = 8

Step 1: seen = {4, 3, 2, 7, 8, 1}   (duplicate 2 and 3 ignored)

Step 2: Check 1 through 8:
  i=1 → seen? YES → skip
  i=2 → seen? YES → skip
  i=3 → seen? YES → skip
  i=4 → seen? YES → skip
  i=5 → seen? NO  → result = [5]
  i=6 → seen? NO  → result = [5, 6]
  i=7 → seen? YES → skip
  i=8 → seen? YES → skip

Return [5, 6] ✅
```

### Complexity

```
Time:  O(n) — two linear passes
Space: O(n) — HashSet stores up to n elements
```

---

## Approach 2 — Index Marking Trick (O(1) Extra Space)

### The Core Insight

```
Since values are in range [1, n] and indices are [0, n-1],
each value v maps to index (v - 1)

If we see a value v, we MARK index (v-1) as "visited"
by making nums[v-1] NEGATIVE

After processing the entire array:
→ Any index i that is still POSITIVE means (i+1)
  was NEVER seen as a value — it's a missing number!
```

### Why Negation Works as a Mark

```
Negating is reversible and doesn't lose the original value
Math.abs(nums[i]) gives back the original value at any time

So we can:
→ Use the SIGN as a "visited" flag
→ Use Math.abs() to read the actual value whenever needed
→ The original data is preserved (just with changed signs)
→ No extra data structure needed!
```

### Code

```java
import java.util.*;

class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {

        // Step 1: Mark visited indices by negating
        for (int i = 0; i < nums.length; i++) {
            int index = Math.abs(nums[i]) - 1;   // value v maps to index v-1
                                                    // Math.abs() because nums[i]
                                                    // might already be negative
                                                    // from a previous marking!
            if (nums[index] > 0) {
                nums[index] = -nums[index];        // mark as visited
            }
        }

        // Step 2: Find indices that are still positive
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0) {
                result.add(i + 1);   // index i → missing number (i+1)
            }
        }

        return result;
    }
}
```

### Walking Through Example 1 Step by Step

```
nums = [4, 3, 2, 7, 8, 2, 3, 1]
index:   0  1  2  3  4  5  6  7

--- STEP 1: Mark visited ---

i=0: nums[0]=4  → index = |4| - 1 = 3
     nums[3] = 7 > 0 → negate → nums[3] = -7
     array: [4, 3, 2, -7, 8, 2, 3, 1]

i=1: nums[1]=3  → index = |3| - 1 = 2
     nums[2] = 2 > 0 → negate → nums[2] = -2
     array: [4, 3, -2, -7, 8, 2, 3, 1]

i=2: nums[2]=-2 → index = |-2| - 1 = 1   ← Math.abs() used here!
     nums[1] = 3 > 0 → negate → nums[1] = -3
     array: [4, -3, -2, -7, 8, 2, 3, 1]

i=3: nums[3]=-7 → index = |-7| - 1 = 6
     nums[6] = 3 > 0 → negate → nums[6] = -3
     array: [4, -3, -2, -7, 8, 2, -3, 1]

i=4: nums[4]=8  → index = |8| - 1 = 7
     nums[7] = 1 > 0 → negate → nums[7] = -1
     array: [4, -3, -2, -7, 8, 2, -3, -1]

i=5: nums[5]=2  → index = |2| - 1 = 1
     nums[1] = -3 < 0 → already marked! → skip
     array: [4, -3, -2, -7, 8, 2, -3, -1]

i=6: nums[6]=-3 → index = |-3| - 1 = 2
     nums[2] = -2 < 0 → already marked! → skip
     array: [4, -3, -2, -7, 8, 2, -3, -1]

i=7: nums[7]=-1 → index = |-1| - 1 = 0
     nums[0] = 4 > 0 → negate → nums[0] = -4
     array: [-4, -3, -2, -7, 8, 2, -3, -1]

--- Final marked array: [-4, -3, -2, -7, 8, 2, -3, -1] ---

--- STEP 2: Find positive indices ---

i=0: nums[0]=-4 → NEGATIVE → was visited (number 1 exists)
i=1: nums[1]=-3 → NEGATIVE → was visited (number 2 exists)
i=2: nums[2]=-2 → NEGATIVE → was visited (number 3 exists)
i=3: nums[3]=-7 → NEGATIVE → was visited (number 4 exists)
i=4: nums[4]=8  → POSITIVE → NOT visited → missing: i+1 = 5 ✅
i=5: nums[5]=2  → POSITIVE → NOT visited → missing: i+1 = 6 ✅
i=6: nums[6]=-3 → NEGATIVE → was visited (number 7 exists)
i=7: nums[7]=-1 → NEGATIVE → was visited (number 8 exists)

Result: [5, 6] ✅
```

### Why We Check `if (nums[index] > 0)` Before Negating

```
Because the same index might be visited TWICE
(when a value appears twice in the array)

If we negate blindly both times:
→ First visit:  positive  → negated → becomes negative (marked)
→ Second visit: negative  → negated → becomes positive again (UNMARKED!)

That's WRONG — we'd accidentally unmark a visited index!

Fix: only negate if currently POSITIVE
→ First visit:  positive → negate → negative ✅
→ Second visit: already negative → skip ✅
```

---

## Approach Comparison

| Feature | HashSet Approach | Index Marking Approach |
|---|---|---|
| Time Complexity | O(n) | O(n) |
| Space Complexity | O(n) | O(1) extra (modifies input) |
| Modifies input array? | No | Yes |
| Code simplicity | Simpler | More nuanced |
| Handles duplicates | Automatically via Set | Manual "already marked" check |
| Best when | Clarity preferred or input can't be modified | Follow-up: O(1) space required |

---

## Why Math.abs() Is Critical in the Index Marking Approach

```java
int index = Math.abs(nums[i]) - 1;

// nums[i] might already be NEGATIVE from a previous marking
// If we don't use Math.abs(), we'd compute a NEGATIVE index
// which would cause ArrayIndexOutOfBoundsException!

// Example:
// nums[2] was originally 2, but got negated to -2
// when we visit i=2 later, nums[2] = -2
// Without Math.abs(): index = -2 - 1 = -3  ← INVALID INDEX!
// With Math.abs():    index = 2 - 1  =  1  ← CORRECT!
```

---

## Edge Cases to Consider

```
1. All numbers present (no missing):
   [1, 2, 3, 4] → return []

2. All same number (maximum missing):
   [1, 1, 1, 1] → return [2, 3, 4]

3. Single element, present:
   [1] → return []

4. Single element, missing:
   [1, 1] → return [2]

5. All numbers at maximum (n appears n times):
   [2, 2] → return [1]

6. Already sorted, no missing:
   [1, 2, 3] → return []
```

---

## Common Mistakes

```
Mistake 1: Forgetting Math.abs() when computing the index
           → ArrayIndexOutOfBoundsException when nums[i] is negative

Mistake 2: Negating unconditionally (without checking > 0 first)
           → Double negation unmarks already-visited indices

Mistake 3: Using i instead of i+1 when collecting results
           → Off-by-one error (index 4 means number 5, not 4)

Mistake 4: Using nums[i] - 1 instead of Math.abs(nums[i]) - 1
           → Same as Mistake 1
```

---

## Complexity Summary

| Approach | Time | Space |
|---|---|---|
| HashSet | O(n) | O(n) |
| Index Marking | O(n) | O(1) extra |

---

## Interview Questions

**Q1. What is the key constraint that makes the index marking trick possible?**
A: Values are guaranteed to be in the range [1, n] where n is the array length. This means every value v maps to a valid index (v - 1), allowing us to use the array itself as a visited marker.

**Q2. Why do we use negation as the marking mechanism?**
A: Negation is reversible — Math.abs() always recovers the original value regardless of sign. It lets us encode a "visited" flag in the sign bit without needing extra space or losing the original data.

**Q3. Why must we use Math.abs(nums[i]) when computing the target index?**
A: Because nums[i] might already be negative from a previous marking step. Without Math.abs(), we'd compute a negative index, causing ArrayIndexOutOfBoundsException.

**Q4. Why do we check `if (nums[index] > 0)` before negating?**
A: To avoid double negation. If a value appears twice, both would try to mark the same index. Negating twice would undo the mark, making a visited index look unvisited. Checking `> 0` ensures we only negate once, keeping the mark permanent.

**Q5. What does a positive value at index i mean after the marking phase?**
A: That the number (i + 1) was never seen as a value in the input array — it's one of the missing numbers.

**Q6. What is the time and space complexity of each approach?**
A: HashSet: O(n) time, O(n) space. Index marking: O(n) time, O(1) extra space (not counting the output list).

**Q7. What is the trade-off of the index marking approach?**
A: It modifies the input array in place. In situations where the caller expects the input to remain unchanged, this would be unacceptable, and the HashSet approach (or restoring the array afterward) would be preferred.

**Q8. Why does off-by-one appear here (value v maps to index v-1)?**
A: Values are 1-indexed (range [1, n]) but array indices are 0-indexed (range [0, n-1]). To map value v to its corresponding index, subtract 1: index = v - 1.

---

## One Line Summary

```
Values in [1,n] map to indices [0,n-1] — negate nums[v-1]
to mark value v as seen, then collect all indices still
positive as missing numbers. O(n) time, O(1) extra space.
```

---