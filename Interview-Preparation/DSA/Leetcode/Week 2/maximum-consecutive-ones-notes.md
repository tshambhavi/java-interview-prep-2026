# Maximum Consecutive Ones (LeetCode 485)

## The Problem, In Plain Terms

Given a binary array (contains only 0s and 1s), return the maximum
number of consecutive 1s in the array.

```
Input:  [1, 1, 0, 1, 1, 1]
Output: 3
Reason: The last three 1s form the longest consecutive run

Input:  [1, 0, 1, 1, 0, 1]
Output: 2
Reason: Two runs of consecutive 1s exist: [1] at index 0,
        [1,1] at index 2-3, [1] at index 5
        Longest = 2

Input:  [0, 0, 0]
Output: 0
Reason: No 1s at all

Input:  [1, 1, 1, 1]
Output: 4
Reason: All elements are 1 — entire array is one consecutive run
```

---

## Key Observations Before Coding

```
1. Binary array — only 0s and 1s
   → No need to compare values, just check if element is 1

2. Consecutive means no gaps — a 0 RESETS the current count
   → When we hit 0, current streak ends immediately

3. We need the MAXIMUM across all streaks
   → Track both current streak and best streak seen so far

4. Simple single pass is enough
   → No sorting, no extra data structures needed
```

---

## Approach — Single Pass with Two Variables

### The Idea

```
Keep two variables:
  currentCount → length of the current streak of 1s
  maxCount     → longest streak seen so far

For each element:
  If element == 1 → increment currentCount
                    update maxCount if currentCount > maxCount
  If element == 0 → reset currentCount to 0
                    (streak is broken)

Return maxCount at the end
```

### Code

```java
class Solution {
    public int findMaxConsecutiveOnes(int[] nums) {
        int currentCount = 0;
        int maxCount = 0;

        for (int num : nums) {
            if (num == 1) {
                currentCount++;
                maxCount = Math.max(maxCount, currentCount);
            } else {
                currentCount = 0;   // reset on 0
            }
        }

        return maxCount;
    }
}
```

### Walking Through Example 1

```
nums = [1, 1, 0, 1, 1, 1]

num=1: currentCount=1, maxCount=max(0,1)=1
num=1: currentCount=2, maxCount=max(1,2)=2
num=0: currentCount=0, maxCount=2 (unchanged)
num=1: currentCount=1, maxCount=max(2,1)=2
num=1: currentCount=2, maxCount=max(2,2)=2
num=1: currentCount=3, maxCount=max(2,3)=3

Return 3 ✅
```

### Walking Through Example 2

```
nums = [1, 0, 1, 1, 0, 1]

num=1: currentCount=1, maxCount=1
num=0: currentCount=0, maxCount=1
num=1: currentCount=1, maxCount=1
num=1: currentCount=2, maxCount=2
num=0: currentCount=0, maxCount=2
num=1: currentCount=1, maxCount=2

Return 2 ✅
```

### Walking Through Edge Case — All Zeros

```
nums = [0, 0, 0]

num=0: currentCount=0, maxCount=0
num=0: currentCount=0, maxCount=0
num=0: currentCount=0, maxCount=0

Return 0 ✅
```

### Walking Through Edge Case — All Ones

```
nums = [1, 1, 1, 1]

num=1: currentCount=1, maxCount=1
num=1: currentCount=2, maxCount=2
num=1: currentCount=3, maxCount=3
num=1: currentCount=4, maxCount=4

Return 4 ✅
```

---

## Can We Update maxCount Only at Reset Instead of Every Step?

```java
// Alternative — update maxCount only when streak ends or array ends

class Solution {
    public int findMaxConsecutiveOnes(int[] nums) {
        int currentCount = 0;
        int maxCount = 0;

        for (int num : nums) {
            if (num == 1) {
                currentCount++;
            } else {
                maxCount = Math.max(maxCount, currentCount);
                currentCount = 0;
            }
        }

        // IMPORTANT: final update after loop ends
        // handles case where array ends with a streak of 1s
        return Math.max(maxCount, currentCount);
    }
}
```

```
This works too, but needs the extra Math.max() AFTER the loop.

Why? If the array ends with 1s (like [1,0,1,1,1]),
the last streak never hits a 0 to trigger the update.
Without the final Math.max(), you would return the wrong answer.

The first approach (updating inside the if block on every 1)
is safer and simpler — no easy-to-forget final step needed.
```

---

## Why Not Use a Counter Array or HashMap?

```
This problem only asks for the LENGTH of the longest run.
You do not need to track WHERE it is, or store past counts.

Two variables is genuinely all you need:
→ currentCount resets on 0, increments on 1
→ maxCount holds the best seen

Adding extra data structures would:
→ Increase space complexity unnecessarily to O(n)
→ Add complexity for no benefit
→ Signal to interviewer that you are overcomplicating it
```

---

## Complexity

```
Time:  O(n) — single pass through the array
Space: O(1) — only two integer variables, no extra storage
```

---

## Related Problems — Same Pattern, More Complex

```
This problem is the FOUNDATION for harder variants:

LeetCode 487 — Max Consecutive Ones II
→ Allowed to FLIP at most ONE 0 to 1
→ What is the longest possible run?
→ Uses sliding window approach

LeetCode 1004 — Max Consecutive Ones III
→ Allowed to FLIP at most K zeros to 1s
→ Sliding window with a budget counter

Understanding 485 clearly makes 487 and 1004
much easier to approach — they extend the exact
same "track a streak" thinking with extra constraints.
```

---

## Edge Cases to Consider

```
1. All zeros:      [0, 0, 0]       → return 0
2. All ones:       [1, 1, 1]       → return 3
3. Single one:     [1]             → return 1
4. Single zero:    [0]             → return 0
5. Ones at end:    [0, 1, 1, 1]   → return 3
6. Ones at start:  [1, 1, 0, 0]   → return 2
7. Alternating:    [1, 0, 1, 0, 1] → return 1
```

---

## Common Mistakes

```
Mistake 1: Forgetting to reset currentCount to 0 on encountering a 0
           → currentCount keeps growing even across gaps

Mistake 2: In the alternative approach, forgetting the final
           Math.max(maxCount, currentCount) after the loop
           → Arrays ending with a streak of 1s return wrong answer

Mistake 3: Returning currentCount instead of maxCount
           → Returns the LAST streak length, not the MAXIMUM

Mistake 4: Initializing maxCount to -1 or some arbitrary value
           → If all zeros, should return 0, not -1
           → Always initialize both variables to 0
```

---

## Complexity Summary

| Metric | Value |
|---|---|
| Time Complexity | O(n) |
| Space Complexity | O(1) |
| Passes needed | 1 |
| Extra data structures | None |

---

## Interview Questions

**Q1. What is the time and space complexity of your solution?**
A: O(n) time — single pass through the array. O(1) space — only two integer variables regardless of input size.

**Q2. Why do we reset currentCount to 0 when we see a 0?**
A: Because a 0 breaks the consecutive streak of 1s. Any further 1s after a 0 start a brand new streak, so the counter must restart from 0.

**Q3. Why initialize both currentCount and maxCount to 0?**
A: If the array contains no 1s at all (all zeros), the correct answer is 0. Initializing to 0 handles this edge case naturally without any special check.

**Q4. What would go wrong if you only updated maxCount when hitting a 0?**
A: If the array ends with a streak of 1s (like [1, 0, 1, 1]), that final streak never triggers a 0 to update maxCount, so the last and potentially longest streak would be missed. You would need an extra update after the loop to fix it.

**Q5. How does this problem relate to LeetCode 1004 — Max Consecutive Ones III?**
A: Problem 485 is the base case with no flips allowed. Problem 1004 extends it by allowing up to K zeros to be flipped to 1s, using a sliding window where the window can contain at most K zeros. Understanding 485 is the foundation for solving 1004.

**Q6. Could you solve this with a single variable instead of two?**
A: Not cleanly — you would lose track of the maximum when a streak resets. Two variables are the minimum: one for the running streak and one for the best streak seen so far.

---

## One Line Summary

```
Single pass: increment currentCount on 1, reset to 0 on 0,
track best with maxCount — O(n) time, O(1) space.
```

---