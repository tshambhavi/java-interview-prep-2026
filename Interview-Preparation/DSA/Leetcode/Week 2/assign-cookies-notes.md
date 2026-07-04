# Assign Cookies (LeetCode 455)

## The Problem, In Plain Terms

You are a parent trying to give cookies to children.
Each child has a greed factor `g[i]` — the minimum cookie size
that will satisfy them. Each cookie has a size `s[j]`.
A child is content only if `s[j] >= g[i]`.
Each child gets AT MOST one cookie. Each cookie can only be used ONCE.
Return the maximum number of content children.

```
Input:  g = [1, 2, 3],  s = [1, 1]
Output: 1
Reason: You have 2 cookies of size 1
        Only the child with greed 1 can be satisfied
        Children with greed 2 and 3 cannot be satisfied
        Max content children = 1

Input:  g = [1, 2],  s = [1, 2, 3]
Output: 2
Reason: Child with greed 1 gets cookie of size 1
        Child with greed 2 gets cookie of size 2 (or 3)
        Both children satisfied → Max = 2
```

---

## Key Observations Before Coding

```
1. You want to MAXIMIZE the number of content children
   → Greedy approach: try to satisfy as many children as possible

2. Wasting a big cookie on a small-greed child is inefficient
   → A cookie of size 5 given to a child who only needs size 1
     could have satisfied a child needing size 5 instead

3. The optimal strategy:
   → Sort both arrays
   → Match the SMALLEST sufficient cookie to the LEAST greedy child
   → If the smallest available cookie can't satisfy the least greedy
     child, no remaining child can be satisfied with that cookie either
     (since all remaining children are MORE greedy)
```

---

## Why Greedy Works Here

```
Claim: Always assign the smallest cookie that satisfies
       the current least-greedy unsatisfied child.

Proof by intuition:
Suppose you skip a small sufficient cookie and use a bigger one.
The bigger cookie satisfies the child just as well — you gain nothing.
But the smaller cookie you skipped might have ALSO worked for this child,
and the bigger cookie could have been needed for a greedier child later.

So using the smallest sufficient cookie NEVER hurts you,
and potentially saves bigger cookies for harder-to-satisfy children.
This is the classic greedy exchange argument.
```

---

## Approach — Sort + Two Pointers (Greedy)

### The Idea

```
Step 1: Sort greed array g in ascending order
Step 2: Sort cookie array s in ascending order
Step 3: Use two pointers:
        i → current child (least greedy first)
        j → current cookie (smallest first)

Step 4: For each cookie j:
        If s[j] >= g[i] → child i is satisfied
                          move to next child (i++)
                          move to next cookie (j++)
        If s[j] < g[i]  → this cookie is too small for this child
                          AND too small for all remaining children
                          (since they're more greedy)
                          discard this cookie (j++ only)

Step 5: Return i (number of children satisfied)
```

### Code

```java
import java.util.Arrays;

class Solution {
    public int findContentChildren(int[] g, int[] s) {
        Arrays.sort(g);   // sort greed factors ascending
        Arrays.sort(s);   // sort cookie sizes ascending

        int i = 0;   // pointer for children
        int j = 0;   // pointer for cookies

        while (i < g.length && j < s.length) {
            if (s[j] >= g[i]) {
                // cookie j satisfies child i
                i++;   // move to next child
            }
            // whether satisfied or not, move to next cookie
            j++;
        }

        return i;   // number of children satisfied
    }
}
```

### Walking Through Example 1

```
g = [1, 2, 3],  s = [1, 1]
After sorting: g = [1, 2, 3],  s = [1, 1]

i=0, j=0: s[0]=1 >= g[0]=1 → satisfied! i=1, j=1
i=1, j=1: s[1]=1 >= g[1]=2? NO (1 < 2) → not satisfied, j=2
j=2 → out of bounds → stop

Return i = 1 ✅
```

### Walking Through Example 2

```
g = [1, 2],  s = [1, 2, 3]
After sorting: g = [1, 2],  s = [1, 2, 3]

i=0, j=0: s[0]=1 >= g[0]=1 → satisfied! i=1, j=1
i=1, j=1: s[1]=2 >= g[1]=2 → satisfied! i=2, j=2
i=2 → out of bounds → stop

Return i = 2 ✅
```

### Walking Through a Tricky Example

```
g = [1, 2, 3],  s = [3]
After sorting: g = [1, 2, 3],  s = [3]

i=0, j=0: s[0]=3 >= g[0]=1 → satisfied! i=1, j=1
j=1 → out of bounds → stop

Return i = 1

Why not more? Only 1 cookie available — even though it's large
enough for all children, one cookie can only satisfy ONE child.
The greedy approach correctly gives it to child 0 (least greedy),
maximizing the count.

What if we gave it to child 2 (greed=3) instead?
→ Still only 1 satisfied — no benefit, same result.
→ But giving to child 0 is still optimal (never worse).
```

### Another Tricky Example — Not All Cookies Usable

```
g = [10, 9, 8, 7],  s = [5, 6, 7, 8]
After sorting: g = [7, 8, 9, 10],  s = [5, 6, 7, 8]

i=0, j=0: s[0]=5 >= g[0]=7? NO → j=1
i=0, j=1: s[1]=6 >= g[0]=7? NO → j=2
i=0, j=2: s[2]=7 >= g[0]=7? YES → i=1, j=3
i=1, j=3: s[3]=8 >= g[1]=8? YES → i=2, j=4
j=4 → out of bounds → stop

Return i = 2 ✅
(Children needing 9 and 10 couldn't be satisfied — no cookie big enough)
```

---

## Why j Always Increments (Not Just When Satisfied)

```java
while (i < g.length && j < s.length) {
    if (s[j] >= g[i]) {
        i++;   // child satisfied, move to next child
    }
    j++;       // ALWAYS move to next cookie
}
```

```
Whether the cookie satisfies the child or not,
that cookie is either:
→ Used up (given to the child)   → advance j
→ Too small for current child    → this cookie is also too small
                                    for ALL remaining children
                                    (since g is sorted ascending —
                                     remaining children are MORE greedy)
                                    → useless, discard it → advance j

Either way, cookie j is DONE. Always advance j.
```

---

## What Happens if We Don't Sort?

```
g = [3, 1, 2],  s = [2, 1, 3]   (unsorted)

Without sorting, matching naively:
Child 0 (greed=3), Cookie 0 (size=2) → 2 < 3 → not satisfied, j++
Child 0 (greed=3), Cookie 1 (size=1) → 1 < 3 → not satisfied, j++
Child 0 (greed=3), Cookie 2 (size=3) → 3 >= 3 → satisfied! i++, j++
i=1 (child greed=1), j=3 → out of bounds → stop
Result = 1 ← WRONG!

Correct answer is 3 (all children can be satisfied)
With sorting: g=[1,2,3], s=[1,2,3]
Every child matched perfectly → Result = 3 ✅

Sorting is essential — without it the two-pointer
approach doesn't work correctly.
```

---

## Complexity

```
Time:  O(n log n + m log m)
       n log n to sort g (n = number of children)
       m log m to sort s (m = number of cookies)
       O(n + m) for the two-pointer traversal
       Overall dominated by sorting: O(n log n + m log m)

Space: O(1) extra space
       (sorting is in-place, only two integer pointers used)
       Note: Java's Arrays.sort uses O(log n) stack space
             for its internal quicksort — sometimes
             considered O(log n) space technically
```

---

## Greedy Pattern Recognition

```
This problem is a textbook GREEDY problem. Key signals:

1. "Maximize the number of X"
2. Two groups being matched (children ↔ cookies)
3. A threshold condition (cookie size >= greed factor)
4. Each item used at most once

Greedy signal: Sort both, match smallest sufficient
               resource to smallest requirement first.

Same greedy pattern appears in:
→ Activity Selection Problem (earliest end time first)
→ Fractional Knapsack
→ Jump Game (LeetCode 55)
```

---

## Edge Cases to Consider

```
1. No cookies:
   g = [1, 2], s = [] → return 0

2. No children:
   g = [], s = [1, 2] → return 0

3. All children satisfied:
   g = [1, 2, 3], s = [1, 2, 3] → return 3

4. No children satisfied (all cookies too small):
   g = [5, 6], s = [1, 2] → return 0

5. More cookies than children:
   g = [1, 2], s = [1, 2, 3, 4] → return 2
   (extra cookies unused)

6. More children than cookies:
   g = [1, 2, 3, 4], s = [2, 3] → return 2
   (some children go without)

7. One child, one cookie — satisfied:
   g = [2], s = [3] → return 1

8. One child, one cookie — not satisfied:
   g = [3], s = [2] → return 0
```

---

## Common Mistakes

```
Mistake 1: Forgetting to sort both arrays
           → Two pointer comparison is meaningless without sorting

Mistake 2: Only incrementing j when satisfied
           → Cookie too small for current child is ALSO too small
             for all remaining (more greedy) children — must discard

Mistake 3: Returning j instead of i
           → j counts cookies used/discarded, i counts satisfied children

Mistake 4: Trying to maximize greedily in reverse
           (giving biggest cookie to greediest child first)
           → Also works! But slightly less intuitive to implement.
           The smallest-to-smallest direction is cleaner.
```

---

## Alternative — Greedy From the Other Direction

```
Instead of smallest cookie to least greedy child,
you can go BIGGEST cookie to MOST greedy child:

Sort g descending, s descending
Try to satisfy the greediest child first with the biggest cookie
If the biggest cookie can't satisfy the greediest child,
no remaining cookie can either (all are smaller)
→ Move to next (less greedy) child

Both directions are valid and give the same result.
The smallest-to-smallest direction is more commonly seen
in interview answers.
```

---

## Complexity Summary

| Step | Time | Space |
|---|---|---|
| Sort g | O(n log n) | O(1) |
| Sort s | O(m log m) | O(1) |
| Two pointer pass | O(n + m) | O(1) |
| **Total** | **O(n log n + m log m)** | **O(1)** |

---

## Interview Questions

**Q1. Why does sorting both arrays help solve this problem?**
A: Sorting lets us use a two-pointer greedy approach — always matching the smallest sufficient cookie to the least greedy unsatisfied child. Without sorting, we can't efficiently find the optimal match and risk suboptimal assignments.

**Q2. Why do we always increment j regardless of whether the child was satisfied?**
A: If the cookie satisfies the child, it's used up. If it doesn't, it's too small for this child — and since g is sorted ascending, it's also too small for all remaining (more greedy) children. Either way, the cookie is done and j advances.

**Q3. Why do we return i and not j at the end?**
A: i counts how many children were satisfied (it increments only on a successful match). j counts how many cookies were processed (both used and discarded). We want the number of content children, which is i.

**Q4. What is the time complexity and what dominates it?**
A: O(n log n + m log m) overall. The sorting of both arrays dominates — the two-pointer traversal is only O(n + m).

**Q5. Could we solve this without sorting? What would the complexity be?**
A: Yes, using a nested loop checking every cookie for every child — O(n * m) time. Or using a frequency map — O(n + m) time but O(n + m) space. Sorting + two pointers gives the best balance of O(n log n + m log m) time and O(1) space.

**Q6. What greedy choice does this problem make, and why is it safe?**
A: Always assign the smallest sufficient cookie to the least greedy child. It's safe because using a larger cookie on a small-greed child wastes potential — the larger cookie might have been needed for a greedier child, while the smaller cookie would have worked just as well for the current child.

**Q7. What happens if there are more cookies than children?**
A: The loop exits when i reaches g.length (all children satisfied). Extra cookies are simply ignored. The result is still the total number of children (all satisfied).

**Q8. What happens if there are more children than cookies?**
A: The loop exits when j reaches s.length (all cookies used). Some children remain unsatisfied. The result is however many children were satisfied before cookies ran out.

---

## One Line Summary

```
Sort both arrays, then greedily assign the smallest
sufficient cookie to the least greedy child using two
pointers — O(n log n + m log m) time, O(1) extra space.
```

---
