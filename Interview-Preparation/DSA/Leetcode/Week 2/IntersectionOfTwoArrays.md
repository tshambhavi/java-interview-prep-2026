# Intersection of Two Arrays (LeetCode 349)

## The Problem, In Plain Terms

Given two arrays, return an array containing only the elements
that appear in BOTH arrays. Each element in the result must be unique
(no duplicates in the output), regardless of how many times it appears
in either input array.

```
Input:  nums1 = [1, 2, 2, 1],  nums2 = [2, 2]
Output: [2]

Input:  nums1 = [4, 9, 5],  nums2 = [9, 4, 9, 8, 4]
Output: [4, 9]  (order doesn't matter)
```

---

## Key Observations Before Coding

```
1. Result must contain UNIQUE elements only
   → Think Set (automatically handles duplicates)

2. An element qualifies only if it appears in BOTH arrays
   → Think "lookup" — is this element in the other array?

3. Order of output doesn't matter
   → No need to sort the result
```

---

## Approach 1 — Using HashSet (Optimal)

### The Idea

```
Step 1: Put ALL elements of nums1 into a HashSet
        (duplicates automatically removed)

Step 2: Iterate through nums2
        For each element, check if it exists in the set

Step 3: If it exists → it's in BOTH arrays → add to result set
        (result is also a Set to avoid duplicate results)

Step 4: Convert result set to array and return
```

### Code

```java
import java.util.*;

class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {

        // Step 1: add all of nums1 into a set
        Set<Integer> set1 = new HashSet<>();
        for (int num : nums1) {
            set1.add(num);
        }

        // Step 2 + 3: check nums2 elements against set1
        Set<Integer> resultSet = new HashSet<>();
        for (int num : nums2) {
            if (set1.contains(num)) {
                resultSet.add(num);   // add to result (auto-deduped)
            }
        }

        // Step 4: convert result set to int array
        int[] result = new int[resultSet.size()];
        int index = 0;
        for (int num : resultSet) {
            result[index++] = num;
        }

        return result;
    }
}
```

### Walking Through Example 1

```
nums1 = [1, 2, 2, 1]
nums2 = [2, 2]

Step 1: set1 = {1, 2}   (duplicates removed automatically)

Step 2-3:
  num=2 → set1.contains(2) = true  → resultSet = {2}
  num=2 → set1.contains(2) = true  → resultSet = {2} (already there, no duplicate)

Step 4: result = [2]
```

### Walking Through Example 2

```
nums1 = [4, 9, 5]
nums2 = [9, 4, 9, 8, 4]

Step 1: set1 = {4, 9, 5}

Step 2-3:
  num=9 → set1.contains(9) = true  → resultSet = {9}
  num=4 → set1.contains(4) = true  → resultSet = {9, 4}
  num=9 → set1.contains(9) = true  → resultSet = {9, 4} (already there)
  num=8 → set1.contains(8) = false → skip
  num=4 → set1.contains(4) = true  → resultSet = {9, 4} (already there)

Step 4: result = [9, 4]  (or [4, 9] — order doesn't matter)
```

---

## Approach 2 — Using Sorting + Two Pointers

### The Idea

```
Step 1: Sort both arrays
Step 2: Use two pointers (one per array), both starting at index 0
Step 3: Compare elements at both pointers:
        - Equal   → add to result (if not already added), advance both
        - nums1[i] < nums2[j] → advance i (nums1 is behind)
        - nums1[i] > nums2[j] → advance j (nums2 is behind)
Step 4: Continue until either pointer goes out of bounds
```

### Code

```java
import java.util.*;

class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Arrays.sort(nums1);
        Arrays.sort(nums2);

        List<Integer> result = new ArrayList<>();
        int i = 0, j = 0;

        while (i < nums1.length && j < nums2.length) {
            if (nums1[i] == nums2[j]) {
                // avoid adding duplicates to result
                if (result.isEmpty() || result.get(result.size() - 1) != nums1[i]) {
                    result.add(nums1[i]);
                }
                i++;
                j++;
            } else if (nums1[i] < nums2[j]) {
                i++;
            } else {
                j++;
            }
        }

        return result.stream().mapToInt(x -> x).toArray();
    }
}
```

### Walking Through Example 2

```
nums1 sorted = [4, 5, 9]
nums2 sorted = [4, 4, 8, 9, 9]

i=0, j=0 → nums1[0]=4, nums2[0]=4 → EQUAL → result=[4], i=1, j=1
i=1, j=1 → nums1[1]=5, nums2[1]=4 → 5 > 4  → advance j → j=2
i=1, j=2 → nums1[1]=5, nums2[2]=8 → 5 < 8  → advance i → i=2
i=2, j=2 → nums1[2]=9, nums2[2]=8 → 9 > 8  → advance j → j=3
i=2, j=3 → nums1[2]=9, nums2[3]=9 → EQUAL → result=[4,9], i=3, j=4
i=3 → out of bounds → stop

Result: [4, 9] ✅
```

---

## Approach Comparison

| Feature | HashSet Approach | Two Pointers Approach |
|---|---|---|
| Time Complexity | O(n + m) | O(n log n + m log m) — sorting cost |
| Space Complexity | O(n + m) — two sets | O(1) extra (ignoring output) |
| Requires sorting? | No | Yes |
| Easier to code? | Yes | Slightly more complex |
| Best when | Arrays are unsorted, speed matters | Arrays are already sorted |
| Duplicates handling | Automatic via Set | Manual check needed |

```
General recommendation:
Use HashSet approach by default — simpler, faster (O(n+m))
Use Two Pointers if interviewer asks "can you do it without
extra space?" or tells you arrays are already sorted
```

---

## Why HashSet.contains() is O(1)

```
HashSet internally uses a HashMap
Each element is hashed to a bucket index
Looking up if an element exists just recomputes
the hash and checks that bucket — O(1) average

This is why the overall solution is O(n + m):
O(n) to build set1 from nums1
O(m) to iterate through nums2 and check each element
Total: O(n + m) — linear!
```

---

## Common Mistakes

```
Mistake 1: Using nums1 directly instead of a Set
           → Would include duplicates in result

Mistake 2: Using a List for lookup (contains on List is O(n)!)
           → Making overall solution O(n * m) — very slow

Mistake 3: Not handling duplicates in result
           → In two pointers approach, forgetting to
              check if element was already added

Mistake 4: Confusing this with Intersection II (LeetCode 350)
           → 350 keeps duplicates in the result (if element
              appears twice in both, include it twice)
           → 349 (this problem) only needs unique elements
```

---

## Related Problem — Intersection II (LeetCode 350)

```
349 (this problem): each element in result appears ONCE
    [1,2,2,1] ∩ [2,2] = [2]

350 (follow-up):    each element appears min(count in nums1, count in nums2) times
    [1,2,2,1] ∩ [2,2] = [2, 2]

For 350: use HashMap to count frequencies instead of HashSet
```

---

## Complexity Summary

| Approach | Time | Space |
|---|---|---|
| HashSet | O(n + m) | O(n + m) |
| Two Pointers | O(n log n + m log m) | O(1) extra |

---

## Interview Questions

**Q1. Why use a HashSet instead of an ArrayList for the lookup?**
A: HashSet.contains() is O(1) average due to hashing. List.contains() is O(n) because it scans linearly. Using a List would make the overall solution O(n * m) instead of O(n + m).

**Q2. Why do we use a Set for the result too, not just for the lookup?**
A: To automatically deduplicate the result. Even if an element appears multiple times in nums2 and matches nums1, using a Set ensures it only appears once in the output.

**Q3. What is the time complexity of the HashSet approach?**
A: O(n + m) — O(n) to build set1 from nums1, O(m) to iterate nums2 and check each element against set1, each check being O(1).

**Q4. When would you prefer the two pointers approach over HashSet?**
A: If the arrays are already sorted (saves the sorting cost), or if the interviewer asks for O(1) extra space (the two pointer approach uses no extra data structures beyond the result).

**Q5. How is this different from LeetCode 350 — Intersection II?**
A: This problem (349) returns unique elements only — each element appears once in the result. Intersection II (350) considers duplicates — if an element appears twice in both arrays, it appears twice in the result.

**Q6. What happens if nums1 is very large and nums2 is small?**
A: It's better to build the HashSet from the SMALLER array and iterate through the larger one. This minimizes memory usage for the set while keeping lookup O(1) during iteration through the larger array.

---

## One Line Summary

```
Put nums1 elements into a HashSet, then iterate nums2 checking
each element against the set — O(n+m) time, O(n+m) space.
Use a result Set to automatically avoid duplicates in output.
```

---
