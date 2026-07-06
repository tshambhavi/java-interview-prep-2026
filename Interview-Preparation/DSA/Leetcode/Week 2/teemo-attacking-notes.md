# Teemo Attacking (LeetCode 495)

## The Problem, In Plain Terms

Teemo attacks an enemy at specific time points stored in array `timeSeries`.
Each attack poisons the enemy for `duration` seconds.
If a new attack happens while the enemy is still poisoned,
the poison timer RESETS to `duration` from that new attack time.
Poison does NOT stack — it just resets.
Return the total number of seconds the enemy is poisoned.

```
Input:  timeSeries = [1, 4],  duration = 2
Output: 4
Reason: Attack at t=1 → poisoned during [1, 2]  (2 seconds)
        Attack at t=4 → poisoned during [4, 5]  (2 seconds)
        No overlap → total = 2 + 2 = 4

Input:  timeSeries = [1, 2],  duration = 2
Output: 3
Reason: Attack at t=1 → would poison [1, 2]
        Attack at t=2 → RESETS poison → now poisoned [2, 3]
        Overlap at t=2 → total unique seconds = [1, 2, 3] = 3
        NOT 4 (which would be wrong due to double counting t=2)
```

---

## Key Observations Before Coding

```
1. Poison does NOT stack — it resets
   → A new attack before current poison expires
     simply extends/resets the timer, it doesn't ADD
     a fresh full duration on top

2. Two cases for each consecutive pair of attacks:
   Case A: Gap between attacks >= duration
           → No overlap, full duration applies to first attack
   Case B: Gap between attacks < duration
           → Overlap! First attack only poisons until next attack

3. The last attack ALWAYS gets its full duration
   → No next attack to interrupt it

4. timeSeries is already sorted (guaranteed by problem)
   → We can compare consecutive pairs directly
```

---

## The Core Logic — Two Cases

```
For attacks at time[i] and time[i+1]:

gap = time[i+1] - time[i]

Case A — gap >= duration:
         [time[i]----poison ends----]
                        [time[i+1]----poison ends----]
         No overlap → contribute full 'duration' seconds

Case B — gap < duration:
         [time[i]--------poison would end here]
                  [time[i+1] resets before that]
         Overlap → only contribute 'gap' seconds
                   (from time[i] to time[i+1])

Last attack: always contributes full 'duration'
             (nothing after it to cut it short)
```

### Visual for Example 2

```
timeSeries = [1, 2], duration = 2

Timeline:
t=1  t=2  t=3  t=4
 |    |    |    |
 [===attack1===]    would be [1,3) if uninterrupted
      [===attack2===]         [2,4) if uninterrupted

Actual poison:
t=1 to t=2: from attack 1 (gap=1, less than duration=2)
t=2 to t=4: from attack 2 (last attack, full duration=2)

Total = 1 + 2 = 3 ✅
```

---

## Approach — Linear Scan of Consecutive Pairs

### Code

```java
class Solution {
    public int findPoisonedDuration(int[] timeSeries, int duration) {
        int total = 0;

        for (int i = 0; i < timeSeries.length - 1; i++) {
            int gap = timeSeries[i + 1] - timeSeries[i];
            total += Math.min(gap, duration);
        }

        // last attack always gets full duration
        total += duration;

        return total;
    }
}
```

### Why Math.min(gap, duration) Captures Both Cases

```
If gap >= duration:  Math.min(gap, duration) = duration   (full duration)
If gap < duration:   Math.min(gap, duration) = gap         (partial, cut short)

One expression handles BOTH cases cleanly —
no if-else needed!
```

### Walking Through Example 1

```
timeSeries = [1, 4], duration = 2

i=0: gap = 4 - 1 = 3
     Math.min(3, 2) = 2  → total = 2

Last attack: total += 2 → total = 4

Return 4 ✅
```

### Walking Through Example 2

```
timeSeries = [1, 2], duration = 2

i=0: gap = 2 - 1 = 1
     Math.min(1, 2) = 1  → total = 1

Last attack: total += 2 → total = 3

Return 3 ✅
```

### Walking Through a Longer Example

```
timeSeries = [1, 2, 3, 4, 5], duration = 5

i=0: gap = 2-1 = 1 → Math.min(1,5) = 1  → total = 1
i=1: gap = 3-2 = 1 → Math.min(1,5) = 1  → total = 2
i=2: gap = 4-3 = 1 → Math.min(1,5) = 1  → total = 3
i=3: gap = 5-4 = 1 → Math.min(1,5) = 1  → total = 4

Last attack: total += 5 → total = 9

Return 9 ✅

Why 9? Attacks every 1 second, poison lasts 5 seconds.
Enemy is poisoned from t=1 all the way through t=10
but each attack just resets the clock.
Total unique poisoned seconds = 9 (t=1 through t=9 from
the 4 gaps, plus final 5 seconds from last attack at t=5
means t=5 through t=9 — wait let's verify):

t=1→attack: poisoned [1,6)
t=2→attack: resets, poisoned [2,7) — overlap [2,6) not double counted
t=3→attack: resets, poisoned [3,8)
t=4→attack: resets, poisoned [4,9)
t=5→attack: resets, poisoned [5,10)

Unique poisoned seconds: t=1 through t=9 = wait, actually
[1,10) — but our formula gives 9... Let's recount:

gap contributions: 1+1+1+1 = 4 seconds (t=1,2,3,4)
last attack:       5 seconds (t=5,6,7,8,9)
Total = 9 seconds

The enemy is poisoned from t=1 to t=10 (exclusive)
= 9 total seconds ✅
```

---

## Why the Last Attack Is Handled Separately

```
The loop only processes pairs (i, i+1).
The last element has no "next" element to compare with.
It's never the first element of a pair in the loop.

Without the final total += duration:
→ The last attack's contribution would be completely ignored!

With total += duration after the loop:
→ Last attack always gets its full uninterrupted duration ✅

This is the most common mistake in this problem.
```

---

## Single Attack Edge Case

```
timeSeries = [5], duration = 3

Loop runs 0 times (length - 1 = 0 iterations)
total = 0

Last attack: total += 3 → total = 3

Return 3 ✅

The loop correctly skips when there's only one attack,
and the final += duration still handles it correctly.
```

---

## What If Attacks Happen at the Same Time?

```
timeSeries = [1, 1, 2], duration = 2

i=0: gap = 1-1 = 0 → Math.min(0,2) = 0 → total = 0
i=1: gap = 2-1 = 1 → Math.min(1,2) = 1 → total = 1

Last attack: total += 2 → total = 3

Return 3

When gap = 0, the second attack adds 0 seconds
(same time, no new ground covered) — handled naturally
by Math.min(0, duration) = 0 ✅
```

---

## Complexity

```
Time:  O(n) — single pass through the array
Space: O(1) — only one integer variable (total)
```

---

## Edge Cases to Consider

```
1. Single attack:
   [5], duration=3 → return 3

2. All attacks at same time:
   [1,1,1], duration=2 → return 2
   (all gaps are 0, only last attack contributes)

3. Attacks far apart (no overlap):
   [1,10,20], duration=3 → return 9 (3+3+3)

4. Attacks very close (lots of overlap):
   [1,2,3], duration=10 → return 12 (1+1+10)

5. duration = 1:
   [1,2,3,4], duration=1 → return 4
   (each attack poisons exactly 1 second, no overlap possible
    since gap is at least 1)
```

---

## Common Mistakes

```
Mistake 1: Forgetting the final total += duration
           → Last attack's contribution is completely ignored
           → Most common mistake on this problem

Mistake 2: Adding full duration for every attack
           → Counts overlapping seconds multiple times
           → [1,2] duration=2 would wrongly return 4 instead of 3

Mistake 3: Using gap >= duration as a condition without Math.min()
           → More verbose, same result, but easier to make an error

Mistake 4: Not handling single element array
           → Loop runs 0 times, but final += duration still works
           → No special case needed if code is structured correctly
```

---

## Why This Is a Greedy / Interval Problem

```
You can think of each attack as creating an interval:
[timeSeries[i], timeSeries[i] + duration]

The question becomes: what is the total length covered
by the UNION of these intervals?

With overlapping intervals, you don't double count —
Math.min(gap, duration) is exactly computing the
contribution of each interval that is NOT covered
by the next one.

This is a simplified version of the classic
"merge intervals" pattern — easier because
timeSeries is already sorted and we just need
total coverage, not the actual merged intervals.
```

---

## Complexity Summary

| Metric | Value |
|---|---|
| Time Complexity | O(n) |
| Space Complexity | O(1) |
| Passes needed | 1 |
| Sorting needed? | No (input guaranteed sorted) |

---

## Interview Questions

**Q1. What are the two cases when considering consecutive attacks?**
A: Case 1 — gap between attacks >= duration: no overlap, first attack contributes full duration. Case 2 — gap < duration: overlap exists, first attack only contributes `gap` seconds (until the next attack resets the timer).

**Q2. Why does Math.min(gap, duration) elegantly handle both cases?**
A: When gap >= duration, min returns duration (full contribution). When gap < duration, min returns gap (partial contribution). Both cases collapse into one expression without needing an if-else.

**Q3. Why must the last attack be handled separately after the loop?**
A: The loop only processes pairs (i, i+1). The last element is never the "first" element of a pair, so its contribution is never counted in the loop. Without `total += duration` after the loop, the last attack's poison duration is entirely ignored.

**Q4. What happens when two attacks occur at exactly the same time (gap = 0)?**
A: Math.min(0, duration) = 0, so the duplicate attack contributes 0 additional seconds — correct, since no new ground is covered. The pattern handles this edge case naturally.

**Q5. What is the time and space complexity?**
A: O(n) time — single linear pass. O(1) space — only one accumulator variable.

**Q6. How is this related to the merge intervals problem?**
A: Each attack creates an interval [t, t+duration]. This problem asks for the total length of the union of all these intervals. Math.min(gap, duration) computes the non-overlapping contribution of each interval with the next, which is equivalent to merging overlapping intervals and summing their lengths.

---

## One Line Summary

```
For each consecutive pair add Math.min(gap, duration),
then add full duration for the last attack — O(n), O(1).
```

---