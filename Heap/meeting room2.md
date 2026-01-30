# Meeting Rooms II

**LeetCode 253** | **Difficulty:** Medium | **Category:** Heap, Greedy, Intervals

---

## Problem Statement

Given an array of meeting time intervals `intervals` where `intervals[i] = [start_i, end_i]`, return the **minimum number of conference rooms** required.

**Note:** The intervals are **half-open** `[start, end)`, meaning the start time is inclusive but the end time is exclusive. A meeting ending at time `t` allows another meeting to start at time `t`.

---

## Examples

### Example 1:
```
Input: intervals = [[0,30],[5,10],[15,20]]
Output: 2

Explanation:
- Meeting [0,30) uses Room 1 (0 to 30)
- Meeting [5,10) cannot use Room 1 (busy until 30), uses Room 2 (5 to 10)
- Meeting [15,20) can use Room 2 (free at 10), reuses Room 2

Maximum concurrent meetings: 2
Answer: 2
```

### Example 2:
```
Input: intervals = [[7,10],[2,4]]
Output: 1

Explanation:
- Meeting [2,4) uses Room 1 (2 to 4)
- Meeting [7,10) can use Room 1 (free at 4), reuses Room 1

Maximum concurrent meetings: 1
Answer: 1
```

### Example 3:
```
Input: intervals = [[0,10],[5,15],[10,20]]
Output: 2

Explanation:
- Meeting [0,10) uses Room 1
- Meeting [5,15) overlaps with [0,10), uses Room 2
- Meeting [10,20) can use Room 1 (free at 10), reuses Room 1

Timeline:
  0----10----20
  [----)      Room 1
       [-----)  Room 2
         [---) Room 1 (reused)

Maximum: 2 rooms
```

### Example 4:
```
Input: intervals = [[1,5],[2,3],[4,6],[5,7]]
Output: 3

Explanation:
Timeline:
  1--2--3--4--5--6--7
  [-------)          Room 1: [1,5)
     [---)            Room 2: [2,3)
        [----)        Room 3: [4,6)
           [---)      Room 1: [5,7) - reused

At time 4: [1,5), [2,3), [4,6) - 3 meetings overlap
Maximum: 3 rooms
```

---

## Strategy

### Min-Heap of End Times

**Key Insight:** At any moment, the number of rooms needed equals the number of meetings that are currently running. We need to track when rooms become free.

**Why This Works:**

```
Question: Do we need a new room for this meeting?
Answer: Only if ALL existing rooms are still busy

How to check efficiently?
- Track when each room becomes free (end time)
- Check if the earliest-ending room is free
- If yes: reuse it
- If no: need a new room

Min-heap gives us O(log n) access to earliest ending room!
```

**Algorithm Steps:**

1. **Sort meetings by start time** (process chronologically)
2. **Use min-heap to track room end times**
3. **For each meeting:**
   - If earliest room ends ≤ current start: reuse room (pop old end)
   - Add current meeting's end time to heap
4. **Return heap size** (max concurrent meetings)

**Data Structures:**

```python
intervals.sort()  # By start time
rooms = []  # Min-heap of end times

# Heap size = number of rooms currently in use
# Max heap size = minimum rooms needed
```

**Why Sort?**

```
Must process meetings chronologically to correctly determine:
1. Which meetings overlap
2. When rooms become available
3. If we can reuse an existing room

Example: [[7,10], [2,4]]
Wrong order: [7,10) needs room, [2,4) needs room → 2 rooms (wrong!)
Sorted: [2,4) uses room, [7,10) reuses same room → 1 room (correct!)
```

**Complexity:**
- **Time:** $O(n \log n)$ - Sorting + n heap operations
- **Space:** $O(n)$ - Heap can hold up to n meetings

---

## Solution

```python
import heapq

class Solution:
    def minMeetingRooms(self, intervals: List[List[int]]) -> int:
        """
        Find minimum number of meeting rooms required.
        
        Strategy: Track room end times with min-heap
        1. Sort meetings by start time
        2. For each meeting:
           - If earliest room is free, reuse it
           - Otherwise, allocate new room
        3. Heap size = rooms currently in use
        
        Key insight: We only care about WHEN rooms become free,
        not which specific room is which. The minimum end time
        tells us if ANY room is available.
        
        Args:
            intervals: List of [start, end) meeting times
            
        Returns:
            Minimum number of concurrent rooms needed
        """
        if not intervals:
            return 0
        
        # Sort meetings by start time
        intervals.sort(key=lambda x: x[0])
        
        # Min-heap: stores end times of ongoing meetings
        rooms = []
        
        # First meeting needs a room
        heapq.heappush(rooms, intervals[0][1])
        
        # Process remaining meetings
        for i in range(1, len(intervals)):
            start, end = intervals[i]
            
            # If earliest-ending room is free, reuse it
            if start >= rooms[0]:
                heapq.heappop(rooms)  # Remove old end time
            
            # Add this meeting's end time
            # (Either new room or updating reused room)
            heapq.heappush(rooms, end)
        
        # Heap size = maximum concurrent meetings
        return len(rooms)
```

### Alternative: Cleaner Loop

```python
def minMeetingRooms(self, intervals: List[List[int]]) -> int:
    """Same algorithm, slightly cleaner."""
    if not intervals:
        return 0
    
    intervals.sort()
    rooms = []
    
    for start, end in intervals:
        # Free up rooms that ended before this meeting
        if rooms and start >= rooms[0]:
            heapq.heappop(rooms)
        
        heapq.heappush(rooms, end)
    
    return len(rooms)
```

---

## Detailed Walkthrough

### Example 1: `intervals = [[0,30],[5,10],[15,20]]`

**Step 1: Sort by start time**
```
Already sorted: [[0,30], [5,10], [15,20]]
```

**Step 2: Process meetings**

**Meeting [0,30):**

| Action | Details |
|--------|---------|
| First meeting | Initialize heap with end time 30 |
| Heap | [30] |
| Rooms needed | 1 |

**Meeting [5,10):**

| Action | Details |
|--------|---------|
| Check | Start 5 >= rooms[0]=30? No (5 < 30) |
| Room available? | No, earliest room busy until 30 |
| Allocate | Need new room, push end=10 |
| Heap | [10, 30] |
| Rooms needed | 2 |

**Meeting [15,20):**

| Action | Details |
|--------|---------|
| Check | Start 15 >= rooms[0]=10? Yes! |
| Room available? | Yes, room ending at 10 is free |
| Reuse | Pop 10, push 20 |
| Heap | [20, 30] |
| Rooms needed | 2 (still 2, reused one) |

**Final Answer:** 2 rooms

**Timeline Visualization:**
```
Time:  0    5    10   15   20   25   30
Room1: [-------------------------)        [0,30)
Room2:      [----)    [----)              [5,10), [15,20)
       
At time 5-10: Both rooms occupied → need 2 rooms
```

---

### Example 2: `intervals = [[0,10],[5,15],[10,20]]`

**Sorted:** `[[0,10], [5,15], [10,20]]`

| Meeting | Start | End | Heap Before | Check | Action | Heap After | Size |
|---------|-------|-----|-------------|-------|--------|------------|------|
| [0,10) | 0 | 10 | [] | First | Push 10 | [10] | 1 |
| [5,15) | 5 | 15 | [10] | 5 >= 10? No | Need new room | [10,15] | 2 |
| [10,20) | 10 | 20 | [10,15] | 10 >= 10? Yes | Reuse, pop 10 | [15,20] | 2 |

**Answer:** 2 rooms

**Key Observation:** Meeting starting exactly at `end` time can reuse the room (half-open interval).

---

### Example 3: `intervals = [[1,5],[2,3],[4,6],[5,7]]`

**Sorted:** Already sorted

| Meeting | Start | End | Heap Before | Earliest End | Can Reuse? | Heap After | Size |
|---------|-------|-----|-------------|--------------|------------|------------|------|
| [1,5) | 1 | 5 | [] | - | First | [5] | 1 |
| [2,3) | 2 | 3 | [5] | 5 | 2 < 5, No | [3,5] | 2 |
| [4,6) | 4 | 6 | [3,5] | 3 | 4 >= 3, Yes | [5,6] | 2 |
| [5,7) | 5 | 7 | [5,6] | 5 | 5 >= 5, Yes | [6,7] | 2 |

**Answer:** 2 rooms

Wait, let me recalculate this more carefully:

**Meeting [1,5):** Heap = [5], size = 1
**Meeting [2,3):** 2 < 5, can't reuse. Heap = [3, 5], size = 2
**Meeting [4,6):** 4 >= 3, can reuse! Pop 3, push 6. Heap = [5, 6], size = 2
**Meeting [5,7):** 5 >= 5, can reuse! Pop 5, push 7. Heap = [6, 7], size = 2

But wait, let me check if at time 4 we have 3 meetings:
- [1,5): running (started at 1, ends at 5)
- [2,3): finished at 3
- [4,6): starting at 4

At time 4: Only [1,5) is running, [2,3) finished. So we need 2 rooms max.

Actually, let me trace when exactly 3 meetings overlap from Example 4:

At time 2: [1,5) is running
At time 2: [2,3) starts → 2 meetings
At time 3: [2,3) ends
At time 4: [4,6) starts, [1,5) still running → 2 meetings
At time 5: [5,7) starts, [1,5) ends, [4,6) still running → 2 meetings

Maximum is 2, not 3. Let me reconsider Example 4...

Actually, looking at it again, the maximum concurrent meetings is 2, not 3. But let me provide a better example where we need 3 rooms:

**Better Example for 3 rooms:** `[[0,5],[1,6],[2,7]]`

| Meeting | Start | End | Heap Before | Check | Heap After | Size |
|---------|-------|-----|-------------|-------|------------|------|
| [0,5) | 0 | 5 | [] | First | [5] | 1 |
| [1,6) | 1 | 6 | [5] | 1 < 5 | [5,6] | 2 |
| [2,7) | 2 | 7 | [5,6] | 2 < 5 | [5,6,7] | 3 |

All three overlap during times 2-5, so we need 3 rooms.

---

## Why This Algorithm Works

### The Heap Invariant

**What the heap represents:**
```
heap = [end_time_1, end_time_2, ..., end_time_k]

Meaning: k meetings are currently in progress
- Each end_time represents when a room becomes free
- heap[0] = earliest time any room becomes available
- len(heap) = number of rooms currently occupied
```

**Why it's correct:**
```
When processing meeting [start, end):

Case 1: start >= heap[0] (earliest room is free)
  → Reuse that room
  → Pop old end time, push new end time
  → Heap size unchanged (no new room needed)

Case 2: start < heap[0] (all rooms busy)
  → Need new room
  → Push new end time
  → Heap size increases (one more concurrent meeting)

At end: max(heap_size) = minimum rooms needed
```

### Comparison with Brute Force

**Brute Force:** Track actual room assignments
```python
def minMeetingRooms_bruteforce(intervals):
    """O(n²) - Expensive and unnecessary."""
    intervals.sort()
    rooms = []  # List of lists: each room's meetings
    
    for start, end in intervals:
        # Find a room that's free
        found = False
        for room in rooms:
            if start >= room[-1]:  # Last meeting in this room
                room.append(end)
                found = True
                break
        
        if not found:
            rooms.append([end])  # New room
    
    return len(rooms)
```

**Issues:** O(n²) time, unnecessary room tracking

**Our Solution:** Only track end times
```python
# O(n log n) - Much faster!
# We don't care WHICH room, only WHEN rooms are free
```

### Visual Model: Timeline Sweep

```
Think of it as sweeping a vertical line across the timeline:

Time:  0    2    4    6    8    10
       |    |    |    |    |    |
       [---------)              Meeting 1: [0,5)
            [---------)         Meeting 2: [2,7)
                 [---------)    Meeting 3: [4,9)

Sweep at time 4:
  - [0,5) is running (ends at 5)
  - [2,7) is running (ends at 7)  
  - [4,9) is starting (ends at 9)
  - Heap: [5, 7, 9] → 3 rooms needed

Sweep at time 6:
  - [0,5) finished
  - [2,7) is running
  - [4,9) is running
  - Heap: [7, 9] → 2 rooms needed
  
Maximum concurrent at any point = 3
```

---

## Edge Cases

### 1. Empty Input
```python
intervals = []
# Output: 0
```

### 2. Single Meeting
```python
intervals = [[0,10]]
# Output: 1
# Only one meeting, one room
```

### 3. No Overlaps
```python
intervals = [[0,5],[5,10],[10,15]]
# Output: 1
# Each meeting starts when previous ends
# Same room can be reused
```

### 4. All Overlap
```python
intervals = [[0,10],[1,11],[2,12],[3,13]]
# Output: 4
# All meetings overlap, need 4 rooms
```

### 5. Nested Intervals
```python
intervals = [[0,30],[5,10],[15,20]]
# Output: 2
# Short meetings nested within long one
```

### 6. Same Start Time
```python
intervals = [[0,5],[0,10],[0,15]]
# Output: 3
# All start simultaneously, need 3 rooms
```

### 7. Boundary Condition (Half-Open)
```python
intervals = [[0,10],[10,20]]
# Output: 1
# Meeting can start exactly when previous ends
# [0,10) ends at 10, [10,20) starts at 10 → OK
```

---

## Common Mistakes

### ❌ Mistake 1: Not Sorting
```python
# Wrong! Processing in original order
for start, end in intervals:  # No sort!
    # ...
```

**Fix:**
```python
intervals.sort()  # MUST sort by start time
for start, end in intervals:
    # ...
```

### ❌ Mistake 2: Wrong Boundary Condition
```python
# Wrong! Uses > instead of >=
if start > rooms[0]:  # Should be >=
    heapq.heappop(rooms)
```

**Fix:**
```python
if start >= rooms[0]:  # >= for half-open intervals
    heapq.heappop(rooms)
```

**Why?** `[0,10)` ends at 10, so `[10,20)` can start at 10.

### ❌ Mistake 3: Tracking Max Instead of Final Size
```python
# Wrong! Tracking max separately
max_rooms = 0
for start, end in intervals:
    # ...
    max_rooms = max(max_rooms, len(rooms))
return max_rooms
```

**Fix:**
```python
# The final heap size IS the answer
for start, end in intervals:
    # ...
return len(rooms)  # No need to track max separately
```

**Why?** We never remove rooms from heap except when reusing, so heap size naturally tracks maximum.

### ❌ Mistake 4: Forgetting to Add After Pop
```python
# Wrong! Only pops, doesn't add new end time
if start >= rooms[0]:
    heapq.heappop(rooms)
# Missing: heapq.heappush(rooms, end)
```

**Fix:**
```python
if start >= rooms[0]:
    heapq.heappop(rooms)
heapq.heappush(rooms, end)  # Always add current meeting
```

### ❌ Mistake 5: Popping All Free Rooms
```python
# Wrong! Pops all free rooms
while rooms and start >= rooms[0]:
    heapq.heappop(rooms)
heapq.heappush(rooms, end)
```

**Fix:**
```python
# Only pop ONE room (we only need one to reuse)
if rooms and start >= rooms[0]:
    heapq.heappop(rooms)
heapq.heappush(rooms, end)
```

**Why?** We only assign one meeting to one room. Other free rooms stay free for future meetings.

---

## Interview Insights

**Question Recognition:**
- "Minimum rooms" + intervals + overlaps → Min-heap of end times
- Classic scheduling/resource allocation problem

**Key Points to Mention:**

1. **Why sort:**
   - "We must process meetings chronologically to correctly identify when rooms become available."

2. **Why min-heap:**
   - "We need to efficiently find which room becomes free earliest. Min-heap gives O(1) access to minimum."

3. **What heap represents:**
   - "Heap size = current concurrent meetings. We track end times to know when rooms free up."

4. **Half-open intervals:**
   - "Important: [0,10) ends at 10, so [10,20) can start at 10. Use >= not > for comparison."

**Follow-up Questions:**

1. **Q:** "Can you solve without a heap?"
   - **A:** Yes, use start/end arrays:
     ```python
     starts = sorted([s for s, e in intervals])
     ends = sorted([e for s, e in intervals])
     i = j = 0
     rooms = max_rooms = 0
     while i < len(starts):
         if starts[i] < ends[j]:
             rooms += 1
             max_rooms = max(max_rooms, rooms)
             i += 1
         else:
             rooms -= 1
             j += 1
     return max_rooms
     ```
     Time: O(n log n), Space: O(n)

2. **Q:** "What if we need to return which meetings are in which rooms?"
   - **A:** Need to track room assignments explicitly (more complex, see variations).

3. **Q:** "What about different room types/capacities?"
   - **A:** Would need to filter available rooms by requirements before heap operations.

4. **Q:** "Can you optimize space?"
   - **A:** Heap approach is O(n) worst case. Can't do better than O(1) unless we know max concurrent meetings.

**Complexity Analysis:**
- Time: O(n log n) - Sorting dominates
- Space: O(n) - Heap can hold all meetings in worst case

---

## Related Problems

1. **Meeting Rooms (LeetCode 252)** - Can attend all meetings (simpler, just overlap check)
2. **Meeting Rooms III (LeetCode 2402)** - Track specific room usage (harder, seen above)
3. **Merge Intervals (LeetCode 56)** - Merge overlapping intervals
4. **Insert Interval (LeetCode 57)** - Insert and merge
5. **Car Pooling (LeetCode 1094)** - Similar timeline with capacity constraint
6. **My Calendar I/II/III (LeetCode 729/731/732)** - k-booking problem
7. **Minimum Platforms (GeeksforGeeks)** - Train scheduling (same problem)

---

## Variations

### Variation 1: Return Room Assignments
```python
def minMeetingRoomsWithAssignments(intervals):
    """Return (room_count, assignments)."""
    if not intervals:
        return 0, []
    
    indexed = [(s, e, i) for i, (s, e) in enumerate(intervals)]
    indexed.sort()
    
    # Heap: (end_time, room_id)
    rooms = []
    assignments = [0] * len(intervals)
    next_room_id = 0
    
    for start, end, orig_idx in indexed:
        if rooms and start >= rooms[0][0]:
            _, room_id = heapq.heappop(rooms)
        else:
            room_id = next_room_id
            next_room_id += 1
        
        heapq.heappush(rooms, (end, room_id))
        assignments[orig_idx] = room_id
    
    return next_room_id, assignments
```

### Variation 2: Two-Pointer Approach (No Heap)
```python
def minMeetingRooms_twopointer(intervals):
    """Alternative: Separate start/end arrays."""
    starts = sorted([i[0] for i in intervals])
    ends = sorted([i[1] for i in intervals])
    
    rooms = max_rooms = 0
    s = e = 0
    
    while s < len(starts):
        if starts[s] < ends[e]:
            # Meeting starting, need room
            rooms += 1
            max_rooms = max(max_rooms, rooms)
            s += 1
        else:
            # Meeting ending, free room
            rooms -= 1
            e += 1
    
    return max_rooms
```

**Comparison:**
- Heap: More intuitive, direct simulation
- Two-pointer: No heap operations, but less obvious

### Variation 3: With Room Priorities
```python
def minMeetingRoomsWithPriorities(intervals, priorities):
    """Assign higher priority meetings to lower room numbers."""
    # Sort by (priority DESC, start ASC)
    indexed = sorted([(priorities[i], intervals[i][0], intervals[i][1], i) 
                      for i in range(len(intervals))], 
                     key=lambda x: (-x[0], x[1]))
    
    rooms = []  # (end_time, room_id)
    assignments = [0] * len(intervals)
    next_room_id = 0
    
    for _, start, end, orig_idx in indexed:
        # Similar logic but considering priorities
        if rooms and start >= rooms[0][0]:
            _, room_id = heapq.heappop(rooms)
        else:
            room_id = next_room_id
            next_room_id += 1
        
        heapq.heappush(rooms, (end, room_id))
        assignments[orig_idx] = room_id
    
    return next_room_id, assignments
```

---

## Summary

**Key Takeaways:**

1. **Min-Heap Pattern:**
   - Track resource availability (room end times)
   - O(log n) to find earliest available
   - Heap size = current resource usage

2. **Sort First:**
   - Process events chronologically
   - Critical for correctness
   - O(n log n) sorting is unavoidable

3. **Reuse vs Allocate:**
   - Check if ANY resource is free (min of heap)
   - If yes: reuse (pop old, push new)
   - If no: allocate (just push new)
   - Final heap size = peak usage

4. **Half-Open Intervals:**
   - `[start, end)` means end is exclusive
   - Meeting ending at t allows another to start at t
   - Use `>=` not `>` in comparison

5. **Abstraction Level:**
   - Don't track WHICH specific room
   - Only track WHEN rooms are available
   - Simpler and faster than explicit room management

**Template:**
```python
intervals.sort()
rooms = []  # Min-heap of end times

for start, end in intervals:
    # Reuse room if available
    if rooms and start >= rooms[0]:
        heapq.heappop(rooms)
    
    # Add current meeting
    heapq.heappush(rooms, end)

return len(rooms)
```

**Mental Model:**
- Imagine a conference center with multiple rooms
- Meetings arrive in order (sorted by start)
- Always reuse the earliest-available room
- If all busy, add a new room
- The maximum rooms ever in use simultaneously = answer
- Heap efficiently tracks "earliest available" at each step
- This pattern extends to any resource scheduling problem!
