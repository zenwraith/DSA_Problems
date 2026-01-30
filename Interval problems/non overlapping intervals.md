# Non-overlapping Intervals

**LeetCode 435** | **Difficulty:** Medium | **Category:** Greedy, Intervals, Sorting

---

## Problem Statement

Given an array of intervals `intervals` where `intervals[i] = [start_i, end_i]`, return the **minimum number of intervals you need to remove** to make the rest of the intervals non-overlapping.

**Note:** Intervals that touch at a single point are considered non-overlapping. For example, `[1,2]` and `[2,3]` are non-overlapping.

---

## Examples

### Example 1:
```
Input: intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1

Explanation:
Remove [1,3] to make remaining intervals non-overlapping:
[[1,2], [2,3], [3,4]] are non-overlapping
```

### Example 2:
```
Input: intervals = [[1,2],[1,2],[1,2]]
Output: 2

Explanation:
Keep only one [1,2], remove the other two
```

### Example 3:
```
Input: intervals = [[1,2],[2,3]]
Output: 0

Explanation:
No overlap ([1,2] and [2,3] touch at point 2, but that's allowed)
```

### Example 4:
```
Input: intervals = [[1,100],[11,22],[1,11],[2,12]]
Output: 2

Explanation:
Sort by end time: [[1,11], [2,12], [11,22], [1,100]]
Keep [1,11], remove [2,12] (overlaps)
Keep [11,22], remove [1,100] (overlaps)
Removed: 2
```

---

## Strategy

### Greedy: Sort by End Time (Activity Selection)

**Key Insight:** To maximize the number of intervals we keep (thus minimize removals), always choose the interval that **ends earliest**.

**Why Sort by End Time?**

```
Consider: [[1,10], [2,3], [3,4]]

If sorted by start time:
  Pick [1,10] first (starts earliest)
  [1,10] overlaps with [2,3] and [3,4]
  Remove 2 intervals ✗

If sorted by end time:
  Pick [2,3] first (ends earliest)
  Pick [3,4] next (doesn't overlap)
  [1,10] overlaps with both, remove it
  Remove 1 interval ✓
```

**The Greedy Principle:**

```
"Choose the activity that finishes first, leaving maximum room for others"

Timeline:
[1,10]  ████████████████████████
[2,3]      ███
[3,4]         ███

If we pick [1,10], it blocks 1-10
If we pick [2,3], it only blocks 2-3, leaving room for [3,4]
```

**Algorithm:**

1. **Sort intervals by end time** (ascending)
2. **Track the end time of last kept interval**
3. **For each interval:**
   - If `start >= prev_end`: No overlap, keep it
   - If `start < prev_end`: Overlaps, must remove it

**Intuition:** By always choosing the interval that finishes earliest, we leave the most "space" for future intervals. This is the classic **Activity Selection Problem**.

**Complexity:**
- **Time:** $O(n \log n)$ - Sorting dominates
- **Space:** $O(1)$ or $O(n)$ depending on sort implementation

---

## Solution

```python
class Solution:
    def eraseOverlapIntervals(self, intervals: List[List[int]]) -> int:
        """
        Find minimum number of intervals to remove for non-overlapping set.
        
        Strategy: Greedy approach using Activity Selection
        1. Sort by end time (keep intervals that finish earliest)
        2. Greedily pick non-overlapping intervals
        3. Count how many we must remove
        
        Key insight: Interval ending earliest leaves most room for others.
        This maximizes intervals kept, minimizes intervals removed.
        
        Args:
            intervals: Array of [start, end] intervals
            
        Returns:
            Minimum number of intervals to remove
        """
        # Sort by end time (greedy: pick earliest finishing)
        intervals.sort(key=lambda x: x[1])
        
        count = 0  # Count of removed intervals
        prev_end = float('-inf')  # End time of last kept interval
        
        for start, end in intervals:
            if start >= prev_end:
                # No overlap: keep this interval
                prev_end = end
            else:
                # Overlaps: remove this interval
                count += 1
        
        return count
```

---

## Detailed Walkthrough

### Example 1: `intervals = [[1,2],[2,3],[3,4],[1,3]]`

**Step 1: Sort by end time**

```
Original: [[1,2], [2,3], [3,4], [1,3]]
Sorted:   [[1,2], [1,3], [2,3], [3,4]]
           end=2   end=3   end=3   end=4
```

**Step 2: Iterate and decide**

| Step | Interval | Start | prev_end | Check | Decision | count | Update prev_end |
|------|----------|-------|----------|-------|----------|-------|-----------------|
| 0 | [1,2] | 1 | -∞ | 1 ≥ -∞? ✓ | Keep | 0 | 2 |
| 1 | [1,3] | 1 | 2 | 1 ≥ 2? ✗ | Remove | 1 | 2 |
| 2 | [2,3] | 2 | 2 | 2 ≥ 2? ✓ | Keep | 1 | 3 |
| 3 | [3,4] | 3 | 3 | 3 ≥ 3? ✓ | Keep | 1 | 4 |

**Visual Timeline:**

```
Sorted order (by end time):
[1,2]  ███
[1,3]  █████  ← Overlaps with [1,2], remove
[2,3]    ███
[3,4]      ███

Kept: [1,2], [2,3], [3,4]
Removed: [1,3]
```

**Final Answer:** `1` ✓

---

### Example 2: `intervals = [[1,100],[11,22],[1,11],[2,12]]`

**Step 1: Sort by end time**

```
Original: [[1,100], [11,22], [1,11], [2,12]]
Sorted:   [[1,11], [2,12], [11,22], [1,100]]
           end=11   end=12   end=22   end=100
```

**Step 2: Iterate**

| Step | Interval | Start | prev_end | Check | Decision | count | Update |
|------|----------|-------|----------|-------|----------|-------|--------|
| 0 | [1,11] | 1 | -∞ | 1 ≥ -∞? ✓ | Keep | 0 | 11 |
| 1 | [2,12] | 2 | 11 | 2 ≥ 11? ✗ | Remove | 1 | 11 |
| 2 | [11,22] | 11 | 11 | 11 ≥ 11? ✓ | Keep | 1 | 22 |
| 3 | [1,100] | 1 | 22 | 1 ≥ 22? ✗ | Remove | 2 | 22 |

**Visual:**

```
[1,11]   ███████████
[2,12]     ███████████ ← Overlaps [1,11], remove
[11,22]            ███████████
[1,100]  ████████████████████████████████████████ ← Overlaps, remove

Kept: [1,11], [11,22]
Removed: [2,12], [1,100]
```

**Final Answer:** `2` ✓

---

### Example 3: `intervals = [[1,2],[1,2],[1,2]]` (All Identical)

**Step 1: Sort** (all have same end time)

```
Sorted: [[1,2], [1,2], [1,2]]
```

**Step 2: Iterate**

| Step | Interval | Start | prev_end | Check | Decision | count |
|------|----------|-------|----------|-------|----------|-------|
| 0 | [1,2] | 1 | -∞ | 1 ≥ -∞? ✓ | Keep | 0 |
| 1 | [1,2] | 1 | 2 | 1 ≥ 2? ✗ | Remove | 1 |
| 2 | [1,2] | 1 | 2 | 1 ≥ 2? ✗ | Remove | 2 |

**Final Answer:** `2` (keep 1, remove 2) ✓

---

### Example 4: `intervals = [[1,2],[2,3]]` (Touching, No Overlap)

**Sort:** Already sorted by end

| Step | Interval | Start | prev_end | Check | Decision | count |
|------|----------|-------|----------|-------|----------|-------|
| 0 | [1,2] | 1 | -∞ | 1 ≥ -∞? ✓ | Keep | 0 |
| 1 | [2,3] | 2 | 2 | 2 ≥ 2? ✓ | Keep | 0 |

**Note:** `start >= prev_end` means touching intervals (like [1,2] and [2,3]) are considered non-overlapping.

**Final Answer:** `0` ✓

---

## Why Sort by End Time? (Deep Dive)

### The Activity Selection Problem

This is the classic **Activity Selection** problem from greedy algorithms.

**Theorem:** Always selecting the activity that finishes first produces an optimal solution.

**Proof Sketch:**

```
Let O = optimal solution
Let G = greedy solution (sort by end time)

Suppose G ≠ O

Let first activity in G be g₁ (ends earliest overall)
Let first activity in O be o₁

If g₁ = o₁, continue to next activity
If g₁ ≠ o₁, then:
  - g₁ ends ≤ o₁ ends (g₁ was chosen as earliest)
  - Replace o₁ with g₁ in O
  - This still works (g₁ doesn't conflict with later activities)
  - Contradiction: O wasn't optimal

Therefore, greedy = optimal
```

### Visual Comparison

**Example: [[1,10], [2,3], [3,4], [4,5]]**

**Sort by Start Time (Wrong):**
```
[1,10]   ████████████████████
[2,3]      ███
[3,4]         ███
[4,5]            ███

Pick [1,10] → blocks everything → remove 3 intervals ✗
```

**Sort by End Time (Correct):**
```
[2,3]      ███
[3,4]         ███
[4,5]            ███
[1,10]   ████████████████████

Pick [2,3], [3,4], [4,5] → remove only [1,10] ✓
```

**Sort by Duration (Also Wrong):**
```
Could pick shorter intervals, but:
[1,2], [2,3], [10,20]

[1,2]  ██
[2,3]    ██
[10,20]              ██████████

Picking shortest first works here, but not always:
[[1,5], [2,3], [4,6]]

Sort by duration: [2,3], [1,5], [4,6]
- Pick [2,3]
- [1,5] overlaps [2,3], remove
- [4,6] doesn't overlap, keep
Result: Remove 1

Sort by end: [2,3], [1,5], [4,6]
- Pick [2,3]
- [1,5] overlaps, remove
- [4,6] doesn't overlap, keep
Result: Remove 1

Both work here, but end time is provably optimal always
```

---

## Edge Cases

### 1. Empty or Single Interval
```python
intervals = []
# Output: 0

intervals = [[1,2]]
# Output: 0
# Nothing to remove
```

### 2. All Non-overlapping
```python
intervals = [[1,2],[2,3],[3,4]]
# Output: 0
# Already non-overlapping
```

### 3. All Overlapping
```python
intervals = [[1,5],[2,4],[3,6]]
# Sort by end: [[2,4], [1,5], [3,6]]
# Keep [2,4], remove [1,5] and [3,6]
# Output: 2
```

### 4. Nested Intervals
```python
intervals = [[1,10],[2,3],[4,5]]
# Sort by end: [[2,3], [4,5], [1,10]]
# Keep [2,3] and [4,5], remove [1,10]
# Output: 1
```

### 5. Same Start, Different Ends
```python
intervals = [[1,2],[1,3],[1,4]]
# Sort by end: [[1,2], [1,3], [1,4]]
# Keep [1,2], remove [1,3] and [1,4]
# Output: 2
```

### 6. Same End, Different Starts
```python
intervals = [[1,5],[2,5],[3,5]]
# All have end=5, order might vary
# Keep first one processed, remove others
# Output: 2
```

### 7. Touching at Single Point
```python
intervals = [[1,2],[2,3],[3,4]]
# Output: 0
# Touching at single point is non-overlapping
```

---

## Common Mistakes

### ❌ Mistake 1: Sorting by Start Time
```python
# Wrong! Greedy by start doesn't guarantee optimal
intervals.sort(key=lambda x: x[0])
```

**Counterexample:**
```
[[1,10], [2,3], [3,4]]
Sort by start → pick [1,10] → remove 2
Sort by end → pick [2,3], [3,4] → remove 1 ✓
```

### ❌ Mistake 2: Using > Instead of >=
```python
# Wrong! Touching intervals are non-overlapping
if start > prev_end:  # Should be >=
    prev_end = end
```

**Why?** `[1,2]` and `[2,3]` should be non-overlapping (touch at 2).

### ❌ Mistake 3: Returning Number Kept Instead of Removed
```python
# Wrong! Return removals, not what we kept
kept = 0
for start, end in intervals:
    if start >= prev_end:
        kept += 1
        prev_end = end

return kept  # Wrong! Should return len(intervals) - kept
```

**Fix:**
```python
return count  # Count of removed intervals
```

### ❌ Mistake 4: Not Initializing prev_end Correctly
```python
# Wrong! Should be -infinity
prev_end = 0  # What if intervals start at negative numbers?
```

**Fix:**
```python
prev_end = float('-inf')
```

### ❌ Mistake 5: Trying to Track Which to Remove
```python
# Overcomplicated! Just count
to_remove = []
for i, (start, end) in enumerate(intervals):
    if start < prev_end:
        to_remove.append(i)
# ...
```

**Fix:** Just count, no need to track indices

---

## Interview Insights

**Question Recognition:**
- "Minimum intervals to remove" + "non-overlapping" → Greedy by end time
- Classic Activity Selection problem

**Key Points to Mention:**

1. **Why greedy works:**
   - "This is the Activity Selection problem. By always choosing the interval that ends earliest, we leave maximum room for future intervals."

2. **Why sort by end, not start:**
   - "Sorting by start time can cause us to pick a long interval early that blocks many shorter ones. End time ensures we're greedy for future space."

3. **Touching is non-overlapping:**
   - "The problem states intervals touching at a single point are non-overlapping, so we use `>=` not `>`."

4. **Complexity:**
   - "O(n log n) for sorting, O(n) for iteration. Total: O(n log n)."

**Follow-up Questions:**

1. **Q:** "Can you solve without sorting?"
   - **A:** No, we need sorted order to apply greedy correctly. O(n log n) is optimal.

2. **Q:** "What if you want the maximum number of non-overlapping intervals instead?"
   - **A:** Same algorithm! Just return `len(intervals) - count` instead of `count`.

3. **Q:** "What if touching intervals ARE considered overlapping?"
   - **A:** Change `>=` to `>` in the condition: `if start > prev_end`.

4. **Q:** "Can you return which intervals to remove?"
   - **A:** Track indices during iteration and store when we decide to remove.

**Complexity Analysis:**
- Time: O(n log n) - Sorting dominates
- Space: O(1) - Only storing counters (or O(n) for sorting workspace)

---

## Related Problems

1. **Merge Intervals (LeetCode 56)** - Merge overlapping intervals
2. **Insert Interval (LeetCode 57)** - Insert and merge new interval
3. **Meeting Rooms (LeetCode 252)** - Check if person can attend all
4. **Meeting Rooms II (LeetCode 253)** - Minimum conference rooms needed
5. **Minimum Number of Arrows to Burst Balloons (LeetCode 452)** - Very similar greedy
6. **Interval List Intersections (LeetCode 986)** - Find intersections

---

## Alternative Approach: Maximum Non-overlapping Intervals

**Same algorithm, different perspective:**

```python
def maxNonOverlapping(intervals):
    """Count maximum non-overlapping intervals we can keep."""
    intervals.sort(key=lambda x: x[1])
    
    kept = 0
    prev_end = float('-inf')
    
    for start, end in intervals:
        if start >= prev_end:
            kept += 1
            prev_end = end
    
    # Minimum to remove = total - maximum kept
    return len(intervals) - kept
```

This is equivalent to our main solution but counts what we keep instead of what we remove.

---

## Comparison: Different Interval Problems

| Problem | Goal | Strategy | Time |
|---------|------|----------|------|
| **Non-overlapping** | Minimize removals | Greedy by end | O(n log n) |
| **Merge Intervals** | Merge overlaps | Sort by start, merge | O(n log n) |
| **Insert Interval** | Insert + merge | Three-phase scan | O(n) |
| **Meeting Rooms** | Check if possible | Sort, check gaps | O(n log n) |
| **Meeting Rooms II** | Min rooms needed | Sweep line | O(n log n) |

---

## Variations

### Variation 1: Return Indices of Intervals to Remove
```python
def eraseOverlapIntervalsWithIndices(intervals):
    """Return indices of intervals to remove."""
    # Add original indices
    indexed = [(intervals[i], i) for i in range(len(intervals))]
    indexed.sort(key=lambda x: x[0][1])  # Sort by end time
    
    to_remove = []
    prev_end = float('-inf')
    
    for (start, end), idx in indexed:
        if start >= prev_end:
            prev_end = end
        else:
            to_remove.append(idx)
    
    return to_remove
```

### Variation 2: Return Maximum Non-overlapping Intervals
```python
def maxNonOverlappingIntervals(intervals):
    """Return the actual non-overlapping intervals."""
    intervals.sort(key=lambda x: x[1])
    
    result = []
    prev_end = float('-inf')
    
    for interval in intervals:
        if interval[0] >= prev_end:
            result.append(interval)
            prev_end = interval[1]
    
    return result
```

### Variation 3: Count Overlapping Pairs
```python
def countOverlappingPairs(intervals):
    """Count how many pairs of intervals overlap."""
    count = 0
    n = len(intervals)
    
    for i in range(n):
        for j in range(i + 1, n):
            # Check if [i] and [j] overlap
            if intervals[i][0] < intervals[j][1] and intervals[j][0] < intervals[i][1]:
                count += 1
    
    return count
```

---

## Summary

**Key Takeaways:**

1. **Greedy by End Time:**
   - Always choose interval that finishes earliest
   - Leaves maximum room for future intervals
   - Provably optimal (Activity Selection)

2. **Why Not Start Time:**
   - Long interval starting early blocks many short ones
   - End time ensures we're greedy for future opportunities
   - Start time greedy is NOT optimal

3. **Touching is Non-overlapping:**
   - Use `start >= prev_end`, not `start > prev_end`
   - `[1,2]` and `[2,3]` are non-overlapping
   - They touch at point 2 but don't overlap

4. **Count Removals:**
   - Track how many overlap (must remove)
   - Equivalent: count kept, return `len - kept`
   - Both approaches give same result

5. **Classic Greedy Problem:**
   - One of the foundational greedy algorithms
   - Same as Activity Selection
   - Appears in many real-world scenarios

**Template:**
```python
# Sort by end time
intervals.sort(key=lambda x: x[1])

count = 0
prev_end = float('-inf')

for start, end in intervals:
    if start >= prev_end:
        # Non-overlapping: keep it
        prev_end = end
    else:
        # Overlaps: remove it
        count += 1

return count
```

**Mental Model:**
- Think of scheduling activities on a single resource
- You want to fit as many activities as possible
- Always pick the one that finishes soonest
- This leaves the most time for future activities
- Greedily maximizing kept = minimizing removed
- This is one of the most elegant applications of greedy algorithms!
