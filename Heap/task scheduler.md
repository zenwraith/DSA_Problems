# Task Scheduler

**LeetCode 621** | **Difficulty:** Medium | **Category:** Heap, Greedy, Math

---

## Problem Statement

Given a characters array `tasks`, representing the tasks a CPU needs to do, where each letter represents a different task. Tasks could be done in any order. Each task is done in one unit of time. For each unit of time, the CPU could complete either one task or just be idle.

However, there is a non-negative integer `n` that represents the cooldown period between two **same tasks** (the same letter in the array), that is, there must be at least `n` units of time between any two same tasks.

Return the **least number of units of times** that the CPU will take to finish all the given tasks.

---

## Examples

### Example 1:
```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8

Explanation: 
A -> B -> idle -> A -> B -> idle -> A -> B
There is at least 2 units of time between any two same tasks.
```

### Example 2:
```
Input: tasks = ["A","A","A","B","B","B"], n = 0
Output: 6

Explanation: 
On this case any permutation of size 6 would work since n = 0.
["A","A","A","B","B","B"]
["A","B","A","B","A","B"]
["B","B","B","A","A","A"]
...
And so on.
```

### Example 3:
```
Input: tasks = ["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2
Output: 16

Explanation: 
One possible solution is:
A -> B -> C -> A -> D -> E -> A -> F -> G -> A -> idle -> idle -> A -> idle -> idle -> A
```

### Example 4:
```
Input: tasks = ["A","B","C","D","E","A","B","C","D","E"], n = 4
Output: 10

Explanation:
A -> B -> C -> D -> E -> A -> B -> C -> D -> E
Each task appears twice. With n=4, we need 4 tasks between same tasks.
Fortunately, we have enough different tasks to fill the gaps.
```

---

## Strategy

### Approach 1: Math Formula (Optimal)

**Key Insight:** The minimum time is determined by the most frequent task and how many idle slots we need to accommodate the cooldown.

**Why This Works:**

```
Observation: The task that appears most often constrains the schedule

Example: tasks = [A,A,A,B,B,B], n = 2

The task with max frequency (A appears 3 times) creates a skeleton:
A _ _ A _ _ A

Between each A, we need at least n=2 units of time.
This creates (f_max - 1) groups of size (n + 1):

Group 1: A _ _
Group 2: A _ _
Final:   A

Number of groups: f_max - 1 = 3 - 1 = 2
Size of each group: n + 1 = 2 + 1 = 3
Last occurrence: 1 (the final A)

Total slots: (f_max - 1) × (n + 1) + 1

But what if multiple tasks have max frequency?
If A and B both appear 3 times:
A B _ A B _ A B

Last group has 2 tasks (A and B), not just 1.
So: (f_max - 1) × (n + 1) + count_max

What if we have more tasks than slots?
tasks = [A,A,B,C,D,E,F], n = 2
Formula gives: (2-1) × (2+1) + 1 = 4
But we have 7 tasks! Answer is max(len(tasks), formula_result)
```

**Formula:**

```python
f_max = max frequency of any task
count_max = number of tasks with max frequency

formula_result = (f_max - 1) × (n + 1) + count_max
answer = max(len(tasks), formula_result)
```

**Why Take Maximum?**

```
Case 1: Not enough tasks to fill gaps
  tasks = [A,A,A,B,B,B], n = 2
  Formula: (3-1) × (2+1) + 2 = 8
  Tasks: 6
  Need idle time, answer = 8

Case 2: Enough tasks to fill gaps
  tasks = [A,A,B,C,D,E,F,G], n = 2
  Formula: (2-1) × (2+1) + 1 = 4
  Tasks: 8
  No idle needed, answer = 8

The formula gives minimum WITH idle.
But if we have enough tasks, no idle needed!
```

**Complexity:**
- **Time:** $O(n)$ - Count tasks
- **Space:** $O(1)$ - At most 26 letters

---

### Approach 2: Heap + Queue Simulation (More Intuitive)

**Key Insight:** Use max-heap to greedily schedule most frequent tasks, and queue to track cooldown periods.

**Why This Works:**

```
Greedy Strategy: Always pick the task with highest remaining count
This minimizes future idle time

Data Structures:
1. Max-heap: Available tasks (by count)
2. Queue: Tasks in cooldown (count, available_time)

Process:
1. Pop most frequent task from heap
2. Schedule it (time++)
3. If it has remaining instances, add to cooldown queue
4. Check if any task finished cooldown, add back to heap
5. If heap empty but queue has tasks → idle time
```

**Algorithm Steps:**

1. **Count task frequencies**
2. **Add all counts to max-heap** (negated for Python)
3. **While heap or queue not empty:**
   - Increment time
   - If heap available: pop and schedule task
     - If task has more instances: add to cooldown queue
   - Check queue: if task cooldown finished, add back to heap
4. **Return total time**

**Complexity:**
- **Time:** $O(T \log 26) = O(T)$ - T time units, heap of size ≤ 26
- **Space:** $O(1)$ - At most 26 tasks

---

## Solutions

### Solution 1: Math Formula (Recommended)

```python
from collections import Counter

class Solution:
    def leastInterval(self, tasks: List[str], n: int) -> int:
        """
        Calculate minimum time using math formula.
        
        Strategy:
        1. Most frequent task creates skeleton structure
        2. Between each occurrence, need n units of time
        3. (f_max - 1) groups of size (n + 1) + final tasks
        4. If enough tasks to fill gaps, no idle needed
        
        Formula: (f_max - 1) × (n + 1) + count_max
        Answer: max(total_tasks, formula_result)
        
        Args:
            tasks: List of task characters
            n: Cooldown period between same tasks
            
        Returns:
            Minimum time units to complete all tasks
        """
        # Count frequency of each task
        counts = Counter(tasks)
        
        # Find maximum frequency
        f_max = max(counts.values())
        
        # Count how many tasks have max frequency
        count_max = sum(1 for count in counts.values() if count == f_max)
        
        # Calculate using formula
        formula_result = (f_max - 1) * (n + 1) + count_max
        
        # Return max of formula and total tasks
        # (If enough tasks to fill gaps, no idle needed)
        return max(len(tasks), formula_result)
```

### Solution 2: Heap + Queue Simulation

```python
import heapq
from collections import Counter, deque

class Solution:
    def leastInterval(self, tasks: List[str], n: int) -> int:
        """
        Simulate scheduling with heap and cooldown queue.
        
        Strategy: Greedy scheduling with cooldown tracking
        1. Always pick most frequent available task
        2. After scheduling, put in cooldown queue
        3. When cooldown ends, return to available heap
        4. If no tasks available but queue not empty → idle
        
        Data Structures:
        - max_heap: Available tasks by frequency (negated)
        - cooldown_q: Tasks in cooldown [(count, available_time)]
        
        Args:
            tasks: List of task characters
            n: Cooldown period between same tasks
            
        Returns:
            Minimum time units to complete all tasks
        """
        # Count frequency of each task
        counts = Counter(tasks)
        
        # Max-heap of task counts (negative for max-heap in Python)
        max_heap = [-cnt for cnt in counts.values()]
        heapq.heapify(max_heap)
        
        # Queue for tasks in cooldown: [remaining_count, available_time]
        cooldown_q = deque()
        
        time = 0
        
        # Continue while tasks remain (in heap or queue)
        while max_heap or cooldown_q:
            time += 1
            
            if max_heap:
                # Schedule most frequent task
                cnt = heapq.heappop(max_heap) + 1  # +1 because negative
                
                # If task has more instances, add to cooldown
                if cnt != 0:  # cnt is negative, 0 means done
                    cooldown_q.append([cnt, time + n])
            
            # Check if any task finished cooldown
            if cooldown_q and cooldown_q[0][1] == time:
                heapq.heappush(max_heap, cooldown_q.popleft()[0])
        
        return time
```

---

## Detailed Walkthrough

### Example 1: `tasks = ["A","A","A","B","B","B"], n = 2`

**Approach 1: Math Formula**

```
Count: {A: 3, B: 3}
f_max = 3
count_max = 2 (both A and B have count 3)

Formula: (3 - 1) × (2 + 1) + 2
       = 2 × 3 + 2
       = 6 + 2
       = 8

len(tasks) = 6
max(6, 8) = 8 ✓
```

**Visualization:**
```
Skeleton created by max-frequency tasks:
A B _ A B _ A B
│ │ │ │ │ │ │ │
1 2 3 4 5 6 7 8

(f_max - 1) = 2 groups of (n + 1) = 3 slots:
[A B _] [A B _] [A B]
 group1  group2  final

Total: 2 × 3 + 2 = 8
```

**Approach 2: Simulation**

| Time | Heap (counts) | Action | Cooldown Queue | Result |
|------|---------------|--------|----------------|--------|
| 0 | [-3, -3] | - | [] | - |
| 1 | [-3] | Pop A (3→2) | [(-2, 4)] | A |
| 2 | [] | Pop B (3→2) | [(-2, 4), (-2, 5)] | B |
| 3 | [] | Idle | [(-2, 4), (-2, 5)] | idle |
| 4 | [-2] | Pop A (2→1) | [(-2, 5), (-1, 7)] | A |
| 5 | [-2] | Pop B (2→1) | [(-1, 7), (-1, 8)] | B |
| 6 | [] | Idle | [(-1, 7), (-1, 8)] | idle |
| 7 | [-1] | Pop A (1→0) | [(-1, 8)] | A |
| 8 | [-1] | Pop B (1→0) | [] | B |

**Final:** 8 time units ✓

---

### Example 2: `tasks = ["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2`

**Math Formula Approach:**

```
Count: {A: 6, B: 1, C: 1, D: 1, E: 1, F: 1, G: 1}
f_max = 6
count_max = 1 (only A)

Formula: (6 - 1) × (2 + 1) + 1
       = 5 × 3 + 1
       = 15 + 1
       = 16

len(tasks) = 12
max(12, 16) = 16 ✓
```

**Visualization:**
```
A _ _ A _ _ A _ _ A _ _ A _ _ A
A B C A D E A F G A idle idle A idle idle A

Groups of 3 between each A:
[A B C] [A D E] [A F G] [A idle idle] [A idle idle] [A]
 group1  group2  group3    group4         group5      final

5 groups × 3 slots + 1 final = 16
```

---

### Example 3: `tasks = ["A","B","C","D","E","A","B","C","D","E"], n = 4`

**Math Formula:**

```
Count: {A: 2, B: 2, C: 2, D: 2, E: 2}
f_max = 2
count_max = 5 (all have frequency 2)

Formula: (2 - 1) × (4 + 1) + 5
       = 1 × 5 + 5
       = 10

len(tasks) = 10
max(10, 10) = 10 ✓
```

**Visualization:**
```
Skeleton: A _ _ _ _ A
Fill:     A B C D E A B C D E

With n=4, we need 4 slots between same tasks.
We have exactly 5 different tasks, perfect fit!
No idle needed.
```

---

## Why This Algorithm Works

### Formula Derivation

**Step 1: Understand the constraint**
```
Most frequent task (f_max occurrences) must be spaced out.
Between each occurrence: at least n other time units.

Example: A appears 4 times, n = 2
A _ _ A _ _ A _ _ A

This creates (f_max - 1) gaps of size (n + 1).
```

**Step 2: Calculate slots**
```
Number of gaps: f_max - 1
Size of each gap: n + 1
Total slots for gaps: (f_max - 1) × (n + 1)
```

**Step 3: Handle multiple max-frequency tasks**
```
If multiple tasks have f_max, they all end together.
Last position needs count_max slots, not just 1.

Example: A and B both appear 3 times
A B _ A B _ A B
        ↑↑↑
    final group has 2

Formula: (f_max - 1) × (n + 1) + count_max
```

**Step 4: Handle sufficient task diversity**
```
What if we have so many different tasks that gaps are filled?

Example: [A,A,B,C,D,E,F,G], n = 2
Formula: (2-1) × (2+1) + 1 = 4
But we have 8 tasks!

Solution: A B C A D E F G (no idle needed)
Answer: max(total_tasks, formula_result)
```

### Heap Simulation Rationale

**Greedy Correctness:**
```
Always schedule most frequent task (when available)

Why? Minimize risk of future forced idle time.
If we delay frequent tasks, we might run out of other tasks
and be forced to idle more later.

Example: [A,A,A,B,C], n = 2
Good: A B C A _ A (5 units)
Bad:  B C A A idle A (6 units)
      ↑ Wasted B and C early, forced idle later
```

**Data Structure Rationale:**
```
Max-Heap: O(log 26) to get most frequent available task
Queue: O(1) to track cooldowns in chronological order

Why not track cooldown in heap?
- Need to check cooldown every time unit
- Queue with timestamp is more efficient
- Pop from queue when time matches
```

---

## Edge Cases

### 1. No Cooldown (n = 0)
```python
tasks = ["A","A","A"], n = 0
# Output: 3
# No spacing needed, just execute in order
```

### 2. All Different Tasks
```python
tasks = ["A","B","C","D","E"], n = 5
# Output: 5
# Each task appears once, no cooldown needed
```

### 3. Single Task Type
```python
tasks = ["A","A","A","A"], n = 2
# Output: 10
# A _ _ A _ _ A _ _ A
# Formula: (4-1) × (2+1) + 1 = 10
```

### 4. High Cooldown
```python
tasks = ["A","A","B"], n = 10
# Output: 22
# A _ _ _ _ _ _ _ _ _ _ A B
# Formula: (2-1) × (10+1) + 1 = 12
# But we have 3 tasks, so... wait
# Actually: (2-1) × (10+1) + 2 = 13 (A and B both have freq 1? No)
# A: 2, B: 1
# Formula: (2-1) × (10+1) + 1 = 12
# max(3, 12) = 12
```

### 5. Many Tasks, Small Cooldown
```python
tasks = ["A","A","A","B","B","B","C","C","C"], n = 1
# Output: 9
# A B A C A B C B C (no idle needed)
# Formula: (3-1) × (1+1) + 3 = 7
# max(9, 7) = 9
```

### 6. All Same Frequency
```python
tasks = ["A","A","B","B","C","C"], n = 2
# Output: 6
# A B C A B C
# Formula: (2-1) × (2+1) + 3 = 6
# max(6, 6) = 6
```

---

## Common Mistakes

### ❌ Mistake 1: Not Taking Maximum with Total Tasks
```python
# Wrong! Formula might be less than total tasks
def leastInterval(self, tasks, n):
    f_max = max(Counter(tasks).values())
    count_max = sum(1 for c in Counter(tasks).values() if c == f_max)
    return (f_max - 1) * (n + 1) + count_max  # WRONG!
```

**Fix:**
```python
formula_result = (f_max - 1) * (n + 1) + count_max
return max(len(tasks), formula_result)  # Correct
```

### ❌ Mistake 2: Wrong Count of Max-Frequency Tasks
```python
# Wrong! Counts frequency, not number of tasks with that frequency
count_max = f_max  # WRONG!
```

**Fix:**
```python
count_max = sum(1 for count in counts.values() if count == f_max)
```

### ❌ Mistake 3: Simulation Without Checking Queue
```python
# Wrong! Doesn't check if cooldown finished
while max_heap:
    time += 1
    cnt = heapq.heappop(max_heap) + 1
    if cnt != 0:
        cooldown_q.append([cnt, time + n])
    # Missing: check cooldown_q!
```

**Fix:**
```python
while max_heap or cooldown_q:  # Check both conditions
    time += 1
    # ... schedule task ...
    # Check if cooldown finished
    if cooldown_q and cooldown_q[0][1] == time:
        heapq.heappush(max_heap, cooldown_q.popleft()[0])
```

### ❌ Mistake 4: Adding Zero Counts to Queue
```python
# Wrong! Adds completed tasks to cooldown
if cnt:  # Wrong check
    cooldown_q.append([cnt, time + n])
```

**Fix:**
```python
if cnt != 0:  # cnt is negative, 0 means task complete
    cooldown_q.append([cnt, time + n])
```

### ❌ Mistake 5: Forgetting to Negate for Max-Heap
```python
# Wrong! Creates min-heap, not max-heap
max_heap = [cnt for cnt in counts.values()]
```

**Fix:**
```python
max_heap = [-cnt for cnt in counts.values()]  # Negate for max-heap
```

---

## Interview Insights

**Question Recognition:**
- "Schedule tasks" + "cooldown" + "minimize time" → Greedy or math
- Classic scheduling problem with constraints

**Key Points to Mention:**

1. **Two approaches:**
   - "There's an elegant O(n) math formula, but I can also simulate with heap if you want the actual schedule."

2. **Formula intuition:**
   - "The most frequent task creates a skeleton. We need (f_max - 1) groups of size (n + 1), plus the final tasks."

3. **Why take maximum:**
   - "If we have enough task diversity, no idle time needed. Formula gives minimum with idle, but we might not need any."

4. **Simulation advantage:**
   - "Simulation can produce the actual schedule, not just the time. Useful if we need to print A -> B -> idle -> A..."

**Follow-up Questions:**

1. **Q:** "Can you print the actual schedule?"
   - **A:** Yes, modify simulation to track scheduled tasks in order.

2. **Q:** "What if tasks have different execution times?"
   - **A:** Need different approach. This assumes all tasks take 1 unit.

3. **Q:** "What if cooldown is different for each task type?"
   - **A:** Formula won't work. Need simulation with per-task cooldown tracking.

4. **Q:** "What if we have dependencies (task B must run after task A)?"
   - **A:** Topological sort first, then schedule respecting both dependencies and cooldown.

5. **Q:** "What about multiple CPUs?"
   - **A:** Much harder. Need to track state per CPU and schedule across them.

**Complexity Analysis:**
- Formula: O(n) time, O(1) space
- Simulation: O(n log k) time, O(k) space (k ≤ 26)

---

## Related Problems

1. **Rearrange String k Distance Apart (LeetCode 358)** - Similar cooldown constraint
2. **Reorganize String (LeetCode 767)** - Special case with n = 1
3. **CPU Scheduling** - Classic OS problem
4. **Maximum Frequency Stack (LeetCode 895)** - Frequency-based decisions
5. **Top K Frequent Elements (LeetCode 347)** - Frequency counting with heap
6. **Task Scheduler II (LeetCode 2365)** - Similar but simpler version

---

## Variations

### Variation 1: Return Actual Schedule
```python
def leastIntervalWithSchedule(self, tasks, n):
    """Return (time, schedule) tuple."""
    counts = Counter(tasks)
    max_heap = [(-cnt, task) for task, cnt in counts.items()]
    heapq.heapify(max_heap)
    
    time = 0
    cooldown_q = deque()
    schedule = []
    
    while max_heap or cooldown_q:
        time += 1
        
        if max_heap:
            cnt, task = heapq.heappop(max_heap)
            schedule.append(task)
            
            if cnt + 1 != 0:
                cooldown_q.append([cnt + 1, task, time + n])
        else:
            schedule.append("idle")
        
        if cooldown_q and cooldown_q[0][2] == time:
            cnt, task, _ = cooldown_q.popleft()
            heapq.heappush(max_heap, (cnt, task))
    
    return time, schedule
```

### Variation 2: Minimum CPUs Needed (Fixed Time)
```python
def minCPUsNeeded(self, tasks, n, max_time):
    """
    Given max time, find minimum CPUs needed.
    Different problem variant.
    """
    counts = Counter(tasks)
    f_max = max(counts.values())
    
    # With one CPU, need at least (f_max - 1) * (n + 1) + count_max
    min_time_single_cpu = (f_max - 1) * (n + 1) + sum(1 for c in counts.values() if c == f_max)
    
    # Calculate minimum CPUs
    return math.ceil(min_time_single_cpu / max_time)
```

### Variation 3: Different Cooldowns Per Task
```python
def leastIntervalVarCooldown(self, tasks, cooldowns):
    """
    tasks: ['A', 'A', 'B', 'B']
    cooldowns: {'A': 2, 'B': 1}
    """
    counts = Counter(tasks)
    max_heap = [(-cnt, task) for task, cnt in counts.items()]
    heapq.heapify(max_heap)
    
    time = 0
    cooldown_q = deque()
    
    while max_heap or cooldown_q:
        time += 1
        
        if max_heap:
            cnt, task = heapq.heappop(max_heap)
            
            if cnt + 1 != 0:
                # Use task-specific cooldown
                available_time = time + cooldowns[task]
                cooldown_q.append([cnt + 1, task, available_time])
        
        # Check all tasks in queue (multiple might be ready)
        while cooldown_q and cooldown_q[0][2] == time:
            cnt, task, _ = cooldown_q.popleft()
            heapq.heappush(max_heap, (cnt, task))
    
    return time
```

---

## Summary

**Key Takeaways:**

1. **Formula Approach (Preferred):**
   - Most frequent task creates skeleton
   - Formula: (f_max - 1) × (n + 1) + count_max
   - Handle excess tasks: max(total_tasks, formula_result)
   - O(n) time, O(1) space

2. **Greedy Principle:**
   - Schedule most frequent tasks first
   - Minimizes future forced idle time
   - Max-heap efficiently tracks frequency

3. **Cooldown Management:**
   - Tasks become unavailable after execution
   - Queue tracks when tasks become available again
   - Check queue each time unit

4. **Two Solutions:**
   - Formula: Fast, only gives time
   - Simulation: Slower, can produce actual schedule
   - Choose based on requirements

5. **Edge Case Awareness:**
   - No cooldown (n = 0): just return len(tasks)
   - High diversity: no idle needed
   - Single task type: maximum idle time

**Formula Template:**
```python
def leastInterval(self, tasks, n):
    counts = Counter(tasks)
    f_max = max(counts.values())
    count_max = sum(1 for c in counts.values() if c == f_max)
    
    formula_result = (f_max - 1) * (n + 1) + count_max
    return max(len(tasks), formula_result)
```

**Simulation Template:**
```python
def leastInterval(self, tasks, n):
    counts = Counter(tasks)
    max_heap = [-cnt for cnt in counts.values()]
    heapq.heapify(max_heap)
    
    time = 0
    cooldown_q = deque()
    
    while max_heap or cooldown_q:
        time += 1
        
        if max_heap:
            cnt = heapq.heappop(max_heap) + 1
            if cnt != 0:
                cooldown_q.append([cnt, time + n])
        
        if cooldown_q and cooldown_q[0][1] == time:
            heapq.heappush(max_heap, cooldown_q.popleft()[0])
    
    return time
```

**Mental Model:**
- Most frequent task is the bottleneck
- Create groups separated by cooldown periods
- Fill groups with other tasks
- If enough variety, no idle needed
- This is a classic greedy scheduling problem!
