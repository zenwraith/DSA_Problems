# Insert Interval

**LeetCode 57** | **Difficulty:** Medium | **Category:** Array, Intervals

---

## Problem Statement

You are given an array of non-overlapping intervals `intervals` where `intervals[i] = [start_i, end_i]` represent the start and the end of the `i`th interval and `intervals` is sorted in ascending order by `start_i`.

You are also given an interval `newInterval = [start, end]` that represents the start and end of another interval.

Insert `newInterval` into `intervals` such that `intervals` is still sorted in ascending order by `start_i` and `intervals` still does not have any overlapping intervals (merge overlapping intervals if necessary).

Return `intervals` after the insertion.

**Note:** You don't need to modify `intervals` in-place. You can make a new array and return it.

---

## Examples

### Example 1:
```
Input: intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]

Explanation:
[2,5] overlaps with [1,3] → merge to [1,5]
[2,5] doesn't overlap with [6,9] → keep [6,9]
```

### Example 2:
```
Input: intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]

Explanation:
New interval [4,8] overlaps with [3,5], [6,7], and [8,10]
Merge all: [3,10]
```

### Example 3:
```
Input: intervals = [], newInterval = [5,7]
Output: [[5,7]]

Explanation:
Empty intervals, just return newInterval
```

### Example 4:
```
Input: intervals = [[1,5]], newInterval = [2,3]
Output: [[1,5]]

Explanation:
[2,3] is completely inside [1,5]
Merge to [1,5]
```

### Example 5:
```
Input: intervals = [[1,5]], newInterval = [6,8]
Output: [[1,5],[6,8]]

Explanation:
No overlap, insert after
```

---

## Strategy

### Three-Phase Linear Scan

**Key Insight:** Process intervals in three distinct phases based on their relationship with `newInterval`.

**Visual Understanding:**

```
intervals:  [1,3]  [5,7]  [9,11]  [13,15]  [17,19]
newInterval:            [8,14]

Phase 1: Before overlap (intervals[i].end < newInterval.start)
  [1,3]  [5,7]  → Add to result as-is

Phase 2: Overlapping (intervals[i].start <= newInterval.end)
  [9,11]  [13,15]  → Merge with newInterval
  newInterval becomes [8,15]

Phase 3: After overlap (remaining intervals)
  [17,19]  → Add to result as-is

Result: [[1,3], [5,7], [8,15], [17,19]]
```

**Overlap Conditions:**

```
Two intervals [a, b] and [c, d] overlap if:
  a <= d AND c <= b

For our case:
  intervals[i][0] <= newInterval[1]  (start of interval <= end of new)
  AND
  newInterval[0] <= intervals[i][1]  (start of new <= end of interval)

Simplified: If not (interval.end < newInterval.start)
           and not (interval.start > newInterval.end)
```

**Three Phases:**

```python
# Phase 1: Add all intervals that end before newInterval starts
while i < n and intervals[i][1] < newInterval[0]:
    res.append(intervals[i])
    i += 1

# Phase 2: Merge all overlapping intervals
while i < n and intervals[i][0] <= newInterval[1]:
    newInterval[0] = min(newInterval[0], intervals[i][0])
    newInterval[1] = max(newInterval[1], intervals[i][1])
    i += 1

res.append(newInterval)  # Add merged interval

# Phase 3: Add all remaining intervals
while i < n:
    res.append(intervals[i])
    i += 1
```

**Why This Works:**

1. **Phase 1 safety:** If `interval.end < newInterval.start`, they can't overlap
2. **Phase 2 merging:** Keep expanding newInterval to cover all overlaps
3. **Phase 3 safety:** After merging, remaining intervals can't overlap (they're sorted)

**Complexity:**
- **Time:** $O(n)$ - Single pass through intervals
- **Space:** $O(n)$ - Result array

---

## Solution

```python
class Solution:
    def insert(self, intervals: List[List[int]], newInterval: List[int]) -> List[List[int]]:
        """
        Insert newInterval into sorted non-overlapping intervals.
        
        Strategy: Three-phase linear scan
        1. Add all intervals that end before newInterval starts (no overlap)
        2. Merge all intervals that overlap with newInterval
        3. Add all remaining intervals (come after merged interval)
        
        Args:
            intervals: Sorted array of non-overlapping intervals
            newInterval: New interval to insert
            
        Returns:
            Updated array of non-overlapping intervals
        """
        res = []
        i = 0
        n = len(intervals)
        
        # Phase 1: Add all intervals that come before newInterval (no overlap)
        # Condition: interval ends before newInterval starts
        while i < n and intervals[i][1] < newInterval[0]:
            res.append(intervals[i])
            i += 1
        
        # Phase 2: Merge all overlapping intervals
        # Condition: interval starts before or at newInterval ends
        while i < n and intervals[i][0] <= newInterval[1]:
            # Expand newInterval to cover this overlapping interval
            newInterval[0] = min(newInterval[0], intervals[i][0])
            newInterval[1] = max(newInterval[1], intervals[i][1])
            i += 1
        
        # Add the merged interval
        res.append(newInterval)
        
        # Phase 3: Add all remaining intervals (come after merged interval)
        while i < n:
            res.append(intervals[i])
            i += 1
        
        return res
```

---

## Detailed Walkthrough

### Example 1: `intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]`

**Initial State:**
- intervals: `[[1,2],[3,5],[6,7],[8,10],[12,16]]`
- newInterval: `[4,8]`
- res: `[]`, i: `0`

**Phase 1: Add intervals before overlap**

| Step | i | Current Interval | Check | Action |
|------|---|------------------|-------|--------|
| 1 | 0 | [1,2] | 2 < 4? ✓ | Add [1,2] to res, i=1 |
| 2 | 1 | [3,5] | 5 < 4? ✗ | Exit Phase 1 |

After Phase 1: `res = [[1,2]]`, `i = 1`, `newInterval = [4,8]`

**Phase 2: Merge overlapping intervals**

| Step | i | Current | Check | Overlap? | Merge |
|------|---|---------|-------|----------|-------|
| 1 | 1 | [3,5] | 3 ≤ 8? ✓ | Yes | newInterval = [min(4,3), max(8,5)] = [3,8] |
| 2 | 2 | [6,7] | 6 ≤ 8? ✓ | Yes | newInterval = [min(3,6), max(8,7)] = [3,8] |
| 3 | 3 | [8,10] | 8 ≤ 8? ✓ | Yes | newInterval = [min(3,8), max(8,10)] = [3,10] |
| 4 | 4 | [12,16] | 12 ≤ 10? ✗ | No | Exit Phase 2 |

After Phase 2: `res = [[1,2]]`, `i = 4`, `newInterval = [3,10]`

Add merged interval: `res = [[1,2], [3,10]]`

**Phase 3: Add remaining intervals**

| Step | i | Current | Action |
|------|---|---------|--------|
| 1 | 4 | [12,16] | Add [12,16] to res, i=5 |
| 2 | 5 | - | i >= n, exit |

**Final Result:** `[[1,2], [3,10], [12,16]]` ✓

**Visual Timeline:**

```
Original:
[1,2]  [3,5]  [6,7]  [8,10]  [12,16]
              [4,8]  ← newInterval

After merge:
[1,2]       [3,10]        [12,16]
            ↑
      Merged [4,8] with [3,5], [6,7], [8,10]
```

---

### Example 2: `intervals = [[1,3],[6,9]], newInterval = [2,5]`

**Phase 1: Before overlap**

| i | Interval | Check: end < 2? | Action |
|---|----------|-----------------|--------|
| 0 | [1,3] | 3 < 2? ✗ | Exit Phase 1 |

After: `res = []`, `i = 0`, `newInterval = [2,5]`

**Phase 2: Merge overlapping**

| i | Interval | Check: start ≤ 5? | Merge |
|---|----------|-------------------|-------|
| 0 | [1,3] | 1 ≤ 5? ✓ | newInterval = [min(2,1), max(5,3)] = [1,5] |
| 1 | [6,9] | 6 ≤ 5? ✗ | Exit Phase 2 |

After: `res = []`, `i = 1`, `newInterval = [1,5]`

Add merged: `res = [[1,5]]`

**Phase 3: Add remaining**

Add [6,9]: `res = [[1,5], [6,9]]`

**Final Result:** `[[1,5], [6,9]]` ✓

---

### Example 3: `intervals = [[1,5]], newInterval = [2,3]` (Contained Inside)

**Phase 1:** [1,5] end is 5, not < 2, skip

**Phase 2:**
- [1,5] start is 1 ≤ 3 ✓
- Merge: newInterval = [min(2,1), max(3,5)] = [1,5]

**Phase 3:** No remaining

**Result:** `[[1,5]]` ✓

---

### Example 4: `intervals = [[1,5]], newInterval = [6,8]` (No Overlap)

**Phase 1:** 
- [1,5] end is 5 < 6 ✓
- Add [1,5]

**Phase 2:** No intervals left, skip

Add newInterval [6,8]

**Phase 3:** No remaining

**Result:** `[[1,5], [6,8]]` ✓

---

## Understanding Overlap Conditions

### Why `intervals[i][0] <= newInterval[1]`?

**Two intervals overlap if one starts before the other ends:**

```
Case 1: Overlap
[a-------b]
    [c-------d]
a ≤ d AND c ≤ b

Case 2: No overlap (gap between)
[a---b]    [c---d]
         ↑
        gap
b < c (interval ends before newInterval starts)

Case 3: No overlap (newInterval before interval)
    [c---d]    [a---b]
         ↑
        gap
d < a (newInterval ends before interval starts)
```

**In Phase 2:**
- We already passed Phase 1, so we know `intervals[i][1] >= newInterval[0]`
- Now we check `intervals[i][0] <= newInterval[1]`
- Together: They overlap!

**Examples:**

```
newInterval = [4,8]

[3,5]: start=3 ≤ 8? ✓ Overlap!
[6,7]: start=6 ≤ 8? ✓ Overlap!
[8,10]: start=8 ≤ 8? ✓ Overlap! (touching is overlap)
[12,16]: start=12 ≤ 8? ✗ No overlap
```

---

## Edge Cases

### 1. Empty Intervals
```python
intervals = [], newInterval = [5,7]
# Output: [[5,7]]
# Just return the new interval
```

### 2. Insert at Beginning
```python
intervals = [[3,5],[6,9]], newInterval = [1,2]
# Output: [[1,2],[3,5],[6,9]]
# Phase 1 adds nothing
# Phase 2 adds newInterval
# Phase 3 adds all existing
```

### 3. Insert at End
```python
intervals = [[1,2],[3,5]], newInterval = [6,8]
# Output: [[1,2],[3,5],[6,8]]
# Phase 1 adds all existing
# Phase 2 finds no overlap
# Add newInterval at end
```

### 4. Completely Contained
```python
intervals = [[1,5]], newInterval = [2,3]
# Output: [[1,5]]
# newInterval absorbed by existing interval
```

### 5. Completely Covers All
```python
intervals = [[2,3],[4,5],[6,7]], newInterval = [1,8]
# Output: [[1,8]]
# newInterval covers everything
```

### 6. Touching Intervals (Not Overlapping in Problem)
```python
intervals = [[1,2],[3,5]], newInterval = [2,3]
# Output: [[1,5]]
# [1,2] and [2,3] touch at 2
# Problem considers this as overlap: 2 ≤ 2
```

### 7. Single Point Interval
```python
intervals = [[1,3],[5,7]], newInterval = [4,4]
# Output: [[1,3],[4,4],[5,7]]
# Single point doesn't overlap neighbors
```

### 8. Merge Multiple Intervals
```python
intervals = [[1,2],[3,5],[6,7],[8,10]], newInterval = [4,9]
# Output: [[1,2],[3,10]]
# Merges [3,5], [6,7], [8,10] into [3,10]
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Overlap Condition
```python
# Wrong! Should be <=, not <
while i < n and intervals[i][0] < newInterval[1]:
    # ...
```

**Fix:**
```python
while i < n and intervals[i][0] <= newInterval[1]:
    # Touching intervals should merge
```

**Why?** Intervals that touch (like [1,2] and [2,3]) should merge to [1,3].

### ❌ Mistake 2: Forgetting to Add Merged Interval
```python
# Wrong! Forgot to add newInterval
while i < n and intervals[i][0] <= newInterval[1]:
    newInterval[0] = min(newInterval[0], intervals[i][0])
    newInterval[1] = max(newInterval[1], intervals[i][1])
    i += 1

# Missing: res.append(newInterval)

while i < n:
    res.append(intervals[i])
    i += 1
```

**Fix:** Add merged interval before Phase 3
```python
res.append(newInterval)  # Don't forget this!
```

### ❌ Mistake 3: Using Wrong Comparison in Phase 1
```python
# Wrong! Should be <, not <=
while i < n and intervals[i][1] <= newInterval[0]:
    res.append(intervals[i])
    i += 1
```

**Fix:**
```python
while i < n and intervals[i][1] < newInterval[0]:
    # Only add if ends BEFORE newInterval starts
```

**Why?** `[1,2]` and `[2,3]` touch at 2, should merge, not separate.

### ❌ Mistake 4: Not Updating Both Bounds
```python
# Wrong! Only updating end
while i < n and intervals[i][0] <= newInterval[1]:
    newInterval[1] = max(newInterval[1], intervals[i][1])  # Missing start update!
    i += 1
```

**Fix:**
```python
while i < n and intervals[i][0] <= newInterval[1]:
    newInterval[0] = min(newInterval[0], intervals[i][0])  # Update start
    newInterval[1] = max(newInterval[1], intervals[i][1])  # Update end
    i += 1
```

### ❌ Mistake 5: Modifying Original Array
```python
# Wrong approach! Don't modify in-place
intervals.insert(index, newInterval)
# Then try to merge...
```

**Fix:** Build new result array from scratch (as in our solution)

---

## Interview Insights

**Question Recognition:**
- "Insert interval" + "merge overlapping" → Three-phase scan
- Already sorted + non-overlapping → Can use linear scan

**Key Points to Mention:**

1. **Three distinct phases:**
   - "We process intervals in three phases: before, overlapping, and after the new interval."

2. **Linear time solution:**
   - "Because intervals are sorted, we can solve this in O(n) with a single pass."

3. **Overlap condition:**
   - "Two intervals overlap if one starts before or when the other ends. We check `start <= other_end`."

4. **Merging strategy:**
   - "For overlapping intervals, we expand newInterval to cover them by taking min of starts and max of ends."

**Follow-up Questions:**

1. **Q:** "What if intervals weren't sorted?"
   - **A:** Sort first (O(n log n)), then apply same algorithm. Total: O(n log n).

2. **Q:** "Can you do this in-place with O(1) space?"
   - **A:** Not easily. Would need to shift elements, making it O(n²). Better to use O(n) space for O(n) time.

3. **Q:** "How would you handle multiple new intervals?"
   - **A:** Sort all new intervals, insert one by one using this algorithm. Or merge new intervals first, then insert.

4. **Q:** "What if touching intervals shouldn't merge?"
   - **A:** Change `<=` to `<` in overlap condition: `intervals[i][0] < newInterval[1]`.

**Complexity Analysis:**
- Time: O(n) - Single pass through intervals
- Space: O(n) - Result array

---

## Related Problems

1. **Merge Intervals (LeetCode 56)** - Merge all overlapping intervals
2. **Non-overlapping Intervals (LeetCode 435)** - Remove minimum to make non-overlapping
3. **Meeting Rooms (LeetCode 252)** - Check if person can attend all meetings
4. **Meeting Rooms II (LeetCode 253)** - Minimum meeting rooms needed
5. **Interval List Intersections (LeetCode 986)** - Find intersections of two interval lists
6. **Remove Covered Intervals (LeetCode 1288)** - Remove intervals covered by others

---

## Comparison: Insert vs Merge Intervals

| Aspect | Insert Interval | Merge Intervals |
|--------|-----------------|-----------------|
| **Input** | Sorted, non-overlapping + new | Unsorted, may overlap |
| **Strategy** | Three-phase scan | Sort + merge |
| **Time** | O(n) | O(n log n) |
| **Space** | O(n) | O(n) |
| **Key** | Linear scan possible | Must sort first |

---

## Variations

### Variation 1: Return Index Where Inserted
```python
def insertWithIndex(intervals, newInterval):
    """Return both result and index of inserted interval."""
    res = []
    i = 0
    n = len(intervals)
    
    # Phase 1
    while i < n and intervals[i][1] < newInterval[0]:
        res.append(intervals[i])
        i += 1
    
    inserted_index = len(res)  # Track where merged interval goes
    
    # Phase 2
    while i < n and intervals[i][0] <= newInterval[1]:
        newInterval[0] = min(newInterval[0], intervals[i][0])
        newInterval[1] = max(newInterval[1], intervals[i][1])
        i += 1
    
    res.append(newInterval)
    
    # Phase 3
    while i < n:
        res.append(intervals[i])
        i += 1
    
    return res, inserted_index
```

### Variation 2: Count Merged Intervals
```python
def insertAndCount(intervals, newInterval):
    """Return result and count of merged intervals."""
    res = []
    i = 0
    n = len(intervals)
    
    # Phase 1
    while i < n and intervals[i][1] < newInterval[0]:
        res.append(intervals[i])
        i += 1
    
    # Phase 2
    merged_count = 0
    while i < n and intervals[i][0] <= newInterval[1]:
        newInterval[0] = min(newInterval[0], intervals[i][0])
        newInterval[1] = max(newInterval[1], intervals[i][1])
        merged_count += 1
        i += 1
    
    res.append(newInterval)
    
    # Phase 3
    while i < n:
        res.append(intervals[i])
        i += 1
    
    return res, merged_count
```

### Variation 3: Insert Multiple New Intervals
```python
def insertMultiple(intervals, newIntervals):
    """Insert multiple new intervals."""
    # Sort new intervals
    newIntervals.sort()
    
    result = intervals[:]
    for new in newIntervals:
        result = insert(result, new)
    
    return result
```

---

## Summary

**Key Takeaways:**

1. **Three-Phase Approach:**
   - Phase 1: Add intervals ending before newInterval
   - Phase 2: Merge all overlapping intervals
   - Phase 3: Add all remaining intervals

2. **Overlap Condition:**
   - Two intervals overlap if: `start1 <= end2 AND start2 <= end1`
   - Simplified for sorted: `intervals[i][0] <= newInterval[1]`
   - Use `<=` to handle touching intervals

3. **Merging Strategy:**
   - Take minimum of starts: `min(start1, start2)`
   - Take maximum of ends: `max(end1, end2)`
   - Keep expanding newInterval to cover all overlaps

4. **Linear Time:**
   - Single pass through intervals: O(n)
   - No sorting needed (already sorted)
   - Each interval processed exactly once

5. **Edge Cases:**
   - Empty intervals: return [newInterval]
   - No overlap: insert in correct position
   - Complete containment: absorbed or covering
   - Touching intervals: considered overlapping

**Template:**
```python
res = []
i = 0

# Phase 1: Before overlap
while i < n and intervals[i][1] < newInterval[0]:
    res.append(intervals[i])
    i += 1

# Phase 2: Merge overlaps
while i < n and intervals[i][0] <= newInterval[1]:
    newInterval[0] = min(newInterval[0], intervals[i][0])
    newInterval[1] = max(newInterval[1], intervals[i][1])
    i += 1

res.append(newInterval)

# Phase 3: After overlap
while i < n:
    res.append(intervals[i])
    i += 1

return res
```

**Mental Model:**
- Think of a timeline with existing appointments
- You want to add a new appointment
- Walk through the timeline from left to right:
  1. Keep all appointments that end before new one starts
  2. Merge all appointments that conflict with the new one
  3. Keep all appointments that start after merged one ends
- Simple, elegant, and efficient linear scan!
