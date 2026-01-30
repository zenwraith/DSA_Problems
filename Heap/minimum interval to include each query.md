# Minimum Interval to Include Each Query

**LeetCode 1851** | **Difficulty:** Hard | **Category:** Heap, Sorting, Line Sweep

---

## Problem Statement

You are given a 2D integer array `intervals`, where `intervals[i] = [left_i, right_i]` describes the `i`th interval starting at `left_i` and ending at `right_i` **(inclusive)**.

The **size** of an interval is defined as the number of integers it contains, or more formally `right_i - left_i + 1`.

You are also given an integer array `queries`. The answer to the `j`th query is the **size of the smallest interval** `i` such that `left_i <= queries[j] <= right_i`. If no such interval exists, the answer is `-1`.

Return an array containing the answers to the queries.

---

## Examples

### Example 1:
```
Input: intervals = [[1,4],[2,4],[3,6],[4,4]], queries = [2,3,4,5]
Output: [3,3,1,4]

Explanation:
- Query 2: [1,4] contains 2, size = 4. [2,4] contains 2, size = 3. Min = 3
- Query 3: [1,4], [2,4], [3,6] contain 3. Min size = 3
- Query 4: All contain 4. [4,4] has size 1. Min = 1
- Query 5: Only [3,6] contains 5, size = 4
```

### Example 2:
```
Input: intervals = [[2,3],[2,5],[1,8],[20,25]], queries = [2,19,5,22]
Output: [2,2,4,6]

Explanation:
- Query 2: [2,3] (size 2), [2,5] (size 4), [1,8] (size 8). Min = 2
- Query 19: No interval contains 19. Output = -1
  Wait, let me recalculate... Actually output says 2?
  
Let me recalculate properly:
- Query 2: [2,3], [2,5], [1,8] all contain 2. Min size = 2 (from [2,3])
- Query 19: No interval contains 19. Output = -1
  But expected is 2... Let me check the problem again.

Actually looking at the expected output [2,-1,4,6]:
- Query 2: Min size = 2 ✓
- Query 19: No interval, -1 ✓
- Query 5: [2,5] (size 4), [1,8] (size 8). Min = 4 ✓
- Query 22: [20,25] contains 22, size = 6 ✓
```

### Example 3:
```
Input: intervals = [[1,3],[2,2],[4,6]], queries = [1,2,3,4,5]
Output: [3,1,3,3,3]

Explanation:
- Query 1: [1,3] contains 1, size = 3
- Query 2: [1,3] (size 3), [2,2] (size 1). Min = 1
- Query 3: [1,3] contains 3, size = 3
- Query 4: [4,6] contains 4, size = 3
- Query 5: [4,6] contains 5, size = 3
```

---

## Strategy

### Sorted Sweep with Min-Heap

**Key Insight:** Process queries in sorted order and use a min-heap to efficiently track the smallest interval containing each query.

**Why This Works:**

```
If we process queries in ascending order:
1. Once we add an interval to consideration (start <= query), 
   it remains valid for all larger queries
2. We can remove intervals when query > end (they'll never be valid again)
3. Min-heap keeps track of smallest interval among valid ones

Example: intervals = [[1,4],[2,4],[3,6]], queries = [2,3,4]

Process query 2:
  Add [1,4] (starts at 1 ≤ 2)
  Add [2,4] (starts at 2 ≤ 2)
  Don't add [3,6] (starts at 3 > 2)
  Heap: [(3, 4, [1,4]), (3, 4, [2,4])]  (size, end)
  Answer: 3

Process query 3:
  Add [3,6] (starts at 3 ≤ 3)
  Heap: [(3, 4, [1,4]), (3, 4, [2,4]), (4, 6, [3,6])]
  Answer: 3

Process query 4:
  No new intervals
  Heap top is [4,4] with size 1? Wait...
```

**Algorithm Steps:**

1. **Sort intervals by start time**
2. **Sort queries but track original indices**
3. **For each query (in sorted order):**
   - Add all intervals where `start <= query` to min-heap
   - Remove intervals where `end < query` from heap
   - Top of heap is minimum size interval containing query

**Data Structures:**

```python
intervals.sort()  # By start time
sorted_queries = [(query_value, original_index), ...]

min_heap = [(interval_size, interval_end), ...]
# Min-heap prioritizes by size (smallest first)
# Also stores end to know when to remove

result = [-1] * len(queries)  # Initialize with -1
```

**Complexity:**
- **Time:** $O((n + q) \log n)$ where n = intervals, q = queries
  - Sorting: O(n log n + q log q)
  - Processing queries: O(q × log n) heap operations
- **Space:** $O(n + q)$ - Heap and sorted queries

---

## Solution

```python
import heapq

class Solution:
    def minInterval(self, intervals: List[List[int]], queries: List[int]) -> List[int]:
        """
        Find minimum interval size containing each query.
        
        Strategy: Sort intervals and queries, sweep with min-heap
        1. Sort intervals by start time
        2. Sort queries (keeping original indices)
        3. For each query:
           - Add intervals that start at or before query
           - Remove intervals that end before query
           - Top of heap is minimum size
        
        Key insight: Processing queries in sorted order allows us to
        add intervals once and remove them when they become invalid.
        
        Args:
            intervals: List of [start, end] intervals (inclusive)
            queries: List of query points
            
        Returns:
            List of minimum interval sizes for each query
        """
        # Sort intervals by start time
        intervals.sort()
        
        # Sort queries but keep track of original indices
        sorted_queries = sorted([(q, i) for i, q in enumerate(queries)])
        
        # Result array (initialize with -1 for "no interval")
        res = [-1] * len(queries)
        
        # Min-heap: (interval_size, interval_end)
        min_heap = []
        i = 0  # Pointer for intervals
        
        # Process queries in sorted order
        for q_val, q_idx in sorted_queries:
            # Add all intervals that start at or before this query
            while i < len(intervals) and intervals[i][0] <= q_val:
                l, r = intervals[i]
                size = r - l + 1  # Size includes both endpoints
                heapq.heappush(min_heap, (size, r))
                i += 1
            
            # Remove intervals that end before this query
            while min_heap and min_heap[0][1] < q_val:
                heapq.heappop(min_heap)
            
            # Top of heap is minimum size interval containing query
            if min_heap:
                res[q_idx] = min_heap[0][0]
        
        return res
```

---

## Detailed Walkthrough

### Example 1: `intervals = [[1,4],[2,4],[3,6],[4,4]], queries = [2,3,4,5]`

**Step 1: Sort**

```
Intervals (already sorted): [[1,4], [2,4], [3,6], [4,4]]
Sorted queries: [(2,0), (3,1), (4,2), (5,3)]
                 ↑ value, index
```

**Step 2: Process queries**

**Query 2 (index 0):**

| Action | Details |
|--------|---------|
| Add intervals | [1,4]: 1 ≤ 2 ✓ → heap: [(3,4)]<br>[2,4]: 2 ≤ 2 ✓ → heap: [(3,4), (3,4)] |
| Not added | [3,6]: 3 > 2 ✗<br>[4,4]: 4 > 2 ✗ |
| Remove invalid | None (both end at 4, which is ≥ 2) |
| Min size | heap[0] = (3, 4) → size = 3 |

Result: `res[0] = 3`

**Query 3 (index 1):**

| Action | Details |
|--------|---------|
| Add intervals | [3,6]: 3 ≤ 3 ✓ → heap: [(3,4), (3,4), (4,6)] |
| Not added | [4,4]: 4 > 3 ✗ |
| Remove invalid | None |
| Min size | heap[0] = (3, 4) → size = 3 |

Result: `res[1] = 3`

**Query 4 (index 2):**

| Action | Details |
|--------|---------|
| Add intervals | [4,4]: 4 ≤ 4 ✓ → heap: [(1,4), (3,4), (3,4), (4,6)] |
| Remove invalid | None |
| Min size | heap[0] = (1, 4) → size = 1 |

Result: `res[2] = 1`

**Query 5 (index 3):**

| Action | Details |
|--------|---------|
| Add intervals | None left |
| Remove invalid | (1,4): 4 < 5 ✓ remove<br>(3,4): 4 < 5 ✓ remove<br>(3,4): 4 < 5 ✓ remove<br>heap: [(4,6)] |
| Min size | heap[0] = (4, 6) → size = 4 |

Result: `res[3] = 4`

**Final Answer:** `[3, 3, 1, 4]` ✓

---

### Example 2: `intervals = [[2,3],[2,5],[1,8],[20,25]], queries = [2,19,5,22]`

**Step 1: Sort**

```
Intervals: [[1,8], [2,3], [2,5], [20,25]]
Sorted queries: [(2,0), (5,2), (19,1), (22,3)]
```

**Query 2 (index 0):**

```
Add: [1,8] (size 8), [2,3] (size 2), [2,5] (size 4)
Heap: [(2,3), (4,5), (8,8)]
Min: 2
res[0] = 2
```

**Query 5 (index 2):**

```
Add: None (all already added up to [2,5])
Remove: (2,3) because 3 < 5
Heap: [(4,5), (8,8)]
Min: 4
res[2] = 4
```

**Query 19 (index 1):**

```
Add: [20,25] starts at 20 > 19, don't add yet
Remove: (4,5) because 5 < 19
        (8,8) because 8 < 19
Heap: []
No interval contains 19
res[1] = -1
```

**Query 22 (index 3):**

```
Add: [20,25] (size 6)
Heap: [(6,25)]
Min: 6
res[3] = 6
```

**Final Answer:** `[2, -1, 4, 6]` ✓

---

## Why This Algorithm Works

### The Sorted Sweep Principle

**Observation 1: Monotonicity**
```
If interval [a,b] contains query q, and q' > q:
- If b >= q', then [a,b] also contains q'
- If b < q', then [a,b] will never contain any future query

This means: once we process a query, we can permanently remove
intervals that end before it.
```

**Observation 2: Growing Valid Set**
```
As we process queries in ascending order:
- Intervals become valid (when query reaches their start)
- Intervals become invalid (when query passes their end)
- We never need to revisit past intervals

This enables single-pass processing with heap maintenance.
```

**Visual Example:**

```
Intervals:
[1,4]   ████████
[2,4]     ██████
[3,6]       ████████
[4,4]         ██

Queries in order: 2, 3, 4, 5

At query 2:
  Valid: [1,4], [2,4]
  Heap: [(3,4), (3,4)]

At query 3:
  Valid: [1,4], [2,4], [3,6]
  Heap: [(3,4), (3,4), (4,6)]

At query 4:
  Valid: [1,4], [2,4], [3,6], [4,4]
  Heap: [(1,4), (3,4), (3,4), (4,6)]

At query 5:
  Valid: [3,6] only (others ended before 5)
  Heap: [(4,6)]
```

---

## Edge Cases

### 1. No Interval Contains Query
```python
intervals = [[1,3],[5,7]], queries = [4]
# Output: [-1]
# Gap between intervals
```

### 2. Single Query, Multiple Intervals
```python
intervals = [[1,5],[2,4],[3,3]], queries = [3]
# Output: [1]
# [3,3] has size 1, smallest
```

### 3. Query at Interval Boundary
```python
intervals = [[1,3],[4,6]], queries = [3,4]
# Output: [3, 3]
# Inclusive boundaries
```

### 4. All Intervals Same Size
```python
intervals = [[1,3],[5,7],[9,11]], queries = [2,6,10]
# Output: [3, 3, 3]
# All have size 3
```

### 5. Query Outside All Intervals
```python
intervals = [[1,5]], queries = [0,6]
# Output: [-1, -1]
# Both outside range
```

### 6. Duplicate Queries
```python
intervals = [[1,5]], queries = [3,3,3]
# Output: [5, 5, 5]
# Same answer for duplicate queries
```

### 7. Nested Intervals
```python
intervals = [[1,10],[2,3],[4,5]], queries = [3]
# Output: [2]
# [2,3] is smallest containing 3
```

---

## Common Mistakes

### ❌ Mistake 1: Not Tracking Original Query Indices
```python
# Wrong! Loses original query order
sorted_queries = sorted(queries)

for q in sorted_queries:
    # ... process
    res.append(answer)  # Wrong order!
```

**Fix:**
```python
sorted_queries = sorted([(q, i) for i, q in enumerate(queries)])
res = [-1] * len(queries)

for q_val, q_idx in sorted_queries:
    # ...
    res[q_idx] = answer  # Correct position
```

### ❌ Mistake 2: Wrong Interval Size Calculation
```python
# Wrong! Off by one
size = r - l  # Should be r - l + 1
```

**Fix:**
```python
size = r - l + 1  # Inclusive [l, r]
```

### ❌ Mistake 3: Wrong Removal Condition
```python
# Wrong! Should be <, not <=
while min_heap and min_heap[0][1] <= q_val:
    heapq.heappop(min_heap)
```

**Fix:**
```python
while min_heap and min_heap[0][1] < q_val:
    heapq.heappop(min_heap)
```

**Why?** Interval [1,4] contains query 4 (inclusive boundary).

### ❌ Mistake 4: Not Sorting Intervals
```python
# Wrong! Must sort first
# intervals.sort()  # Missing!

for q_val, q_idx in sorted_queries:
    while i < len(intervals) and intervals[i][0] <= q_val:
        # This won't work correctly without sorted intervals
```

### ❌ Mistake 5: Storing Wrong Data in Heap
```python
# Wrong! Can't determine validity later
heapq.heappush(min_heap, size)  # Missing end time!
```

**Fix:**
```python
heapq.heappush(min_heap, (size, r))  # Need end for removal
```

---

## Interview Insights

**Question Recognition:**
- "For each query" + "minimum interval" + intervals + queries → Line sweep with heap
- Hard problem requiring multiple techniques

**Key Points to Mention:**

1. **Why sort both:**
   - "Sorting queries allows us to process them left-to-right, adding and removing intervals as we go."

2. **Why min-heap:**
   - "We need the minimum size among all valid intervals. A min-heap gives us O(1) access to minimum and O(log n) updates."

3. **Why store end time:**
   - "We need the end time to know when an interval becomes invalid for future queries."

4. **Track original indices:**
   - "We must track original query indices because we need to return answers in the original query order."

**Follow-up Questions:**

1. **Q:** "Can you solve without sorting queries?"
   - **A:** Yes, but less efficient. For each query, scan all intervals: O(n × q). With sorting: O((n+q) log n).

2. **Q:** "What if intervals overlap heavily?"
   - **A:** No issue. Heap handles overlapping intervals naturally by comparing sizes.

3. **Q:** "Can you optimize space?"
   - **A:** Current O(n) heap space is necessary. Could process in smaller batches if memory constrained.

4. **Q:** "What if queries have duplicates?"
   - **A:** Works correctly. Each duplicate query will get same answer (processed independently).

**Complexity Analysis:**
- Time: O((n + q) log n) - Sorting + heap operations
- Space: O(n + q) - Heap + sorted queries

---

## Related Problems

1. **Meeting Rooms II (LeetCode 253)** - Min rooms needed (interval scheduling)
2. **Merge Intervals (LeetCode 56)** - Merge overlapping intervals
3. **Insert Interval (LeetCode 57)** - Insert and merge
4. **Interval List Intersections (LeetCode 986)** - Find intersections
5. **Car Pooling (LeetCode 1094)** - Capacity constraint with timeline
6. **The Skyline Problem (LeetCode 218)** - Line sweep with events

---

## Alternative Approaches

### Approach 1: Brute Force (TLE)
```python
def minInterval_bruteforce(intervals, queries):
    """O(n × q) - Too slow for large inputs."""
    res = []
    
    for q in queries:
        min_size = float('inf')
        
        for l, r in intervals:
            if l <= q <= r:
                min_size = min(min_size, r - l + 1)
        
        res.append(min_size if min_size != float('inf') else -1)
    
    return res
```

**Issues:** O(n × q) time, no optimization

### Approach 2: Binary Search + Scan (Better but still slow)
```python
def minInterval_binsearch(intervals, queries):
    """Sort intervals, binary search for each query."""
    intervals.sort()
    res = []
    
    for q in queries:
        min_size = float('inf')
        
        # Binary search for intervals that might contain q
        for l, r in intervals:
            if l <= q <= r:
                min_size = min(min_size, r - l + 1)
            elif l > q:
                break  # No more candidates
        
        res.append(min_size if min_size != float('inf') else -1)
    
    return res
```

**Better:** Still O(n × q) worst case, but with early termination

---

## Variations

### Variation 1: Return Interval Itself, Not Size
```python
def minIntervalObject(intervals, queries):
    """Return the actual interval, not just size."""
    intervals.sort()
    sorted_queries = sorted([(q, i) for i, q in enumerate(queries)])
    
    res = [None] * len(queries)
    min_heap = []
    i = 0
    
    for q_val, q_idx in sorted_queries:
        while i < len(intervals) and intervals[i][0] <= q_val:
            l, r = intervals[i]
            size = r - l + 1
            heapq.heappush(min_heap, (size, r, l))  # Add start too
            i += 1
        
        while min_heap and min_heap[0][1] < q_val:
            heapq.heappop(min_heap)
        
        if min_heap:
            size, r, l = min_heap[0]
            res[q_idx] = [l, r]
        else:
            res[q_idx] = None
    
    return res
```

### Variation 2: Count Valid Intervals Per Query
```python
def countIntervalsPerQuery(intervals, queries):
    """Count how many intervals contain each query."""
    intervals.sort()
    sorted_queries = sorted([(q, i) for i, q in enumerate(queries)])
    
    res = [0] * len(queries)
    valid_intervals = []
    i = 0
    
    for q_val, q_idx in sorted_queries:
        # Add intervals
        while i < len(intervals) and intervals[i][0] <= q_val:
            valid_intervals.append(intervals[i])
            i += 1
        
        # Remove expired
        valid_intervals = [iv for iv in valid_intervals if iv[1] >= q_val]
        
        res[q_idx] = len(valid_intervals)
    
    return res
```

---

## Summary

**Key Takeaways:**

1. **Line Sweep Pattern:**
   - Process events (queries) in sorted order
   - Maintain active set (intervals) as we sweep
   - Add/remove elements dynamically

2. **Min-Heap for Minimum:**
   - Need minimum size among valid intervals
   - Heap provides O(log n) updates, O(1) access
   - Store (size, end) to enable removal

3. **Sort Both Inputs:**
   - Sort intervals by start: enables sequential addition
   - Sort queries: enables single-pass processing
   - Track original indices: return in correct order

4. **Validity Management:**
   - Add when `interval.start <= query`
   - Remove when `interval.end < query`
   - Heap top is always valid and minimum

5. **Complexity Trade-off:**
   - Brute force: O(n × q)
   - Optimized: O((n+q) log n)
   - Worth the complexity for large inputs

**Template:**
```python
intervals.sort()
sorted_queries = sorted([(q, i) for i, q in enumerate(queries)])

res = [-1] * len(queries)
min_heap = []
i = 0

for q_val, q_idx in sorted_queries:
    # Add valid intervals
    while i < len(intervals) and intervals[i][0] <= q_val:
        l, r = intervals[i]
        heapq.heappush(min_heap, (r - l + 1, r))
        i += 1
    
    # Remove invalid intervals
    while min_heap and min_heap[0][1] < q_val:
        heapq.heappop(min_heap)
    
    # Get minimum
    if min_heap:
        res[q_idx] = min_heap[0][0]

return res
```

**Mental Model:**
- Imagine a timeline with intervals as segments
- Walk through timeline with query points
- As you move right:
  - New intervals become valid (reach their start)
  - Old intervals become invalid (pass their end)
- Keep track of smallest valid interval at each query point
- Min-heap efficiently maintains "smallest" as set changes
- This is a sophisticated application of line sweep + priority queue!
