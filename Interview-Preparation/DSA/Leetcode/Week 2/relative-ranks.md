# Problem 506 - Relative Ranks

## Pattern:
---------
Sort + HashMap + Restore Original Order

## Key Idea:
---------
1. Keep original array.
2. Sort a copy.
3. Assign rank using sorted order.
4. Store:
   Score -> Rank
5. Traverse original array and fetch ranks using the map.

## Recognition Pattern:
--------------------
If a problem asks:
"Return answers in original order"

Think:
Keep original data unchanged
+
Sort a copy
+
Use HashMap to map value -> answer
+
Traverse original array to restore order

--------------------