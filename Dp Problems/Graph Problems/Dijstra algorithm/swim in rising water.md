# Swim in Rising Water

**LeetCode Link:** [https://leetcode.com/problems/swim-in-rising-water/](https://leetcode.com/problems/swim-in-rising-water/)

---

## Problem Statement

You are given an `n x n` integer matrix `grid` where each value `grid[i][j]` represents the elevation at that point `(i, j)`.

The rain starts to fall. At time `t`, the depth of the water everywhere is `t`. You can swim from a square to another 4-directionally adjacent square if and only if the elevation of both squares individually is at most `t`. You can swim infinite distances in zero time. Of course, you must stay within the boundaries of the grid during your swim.

You start at the top left square `(0, 0)`. Return the **least time** until you can reach the bottom right square `(n - 1, n - 1)`.

**Example 1:**
```
Input: grid = [[0,2],[1,3]]
Output: 3

Explanation:
Grid:  0  2
       1  3

At t = 0: Can reach (0,0) only (elevation 0 ≤ 0)
At t = 1: Can reach (0,0) and (1,0) (elevations 0,1 ≤ 1)
At t = 2: Can reach (0,0), (1,0), (0,1) (elevations 0,1,2 ≤ 2)
At t = 3: Can reach all cells including (1,1) (all elevations ≤ 3) ✓

Minimum time = 3
```

**Example 2:**
```
Input: grid = [[0,1,2,3,4],[24,23,22,21,5],[12,13,14,15,16],[11,17,18,19,20],[10,9,8,7,6]]
Output: 16

Explanation:
Grid:  0   1   2   3   4
      24  23  22  21   5
      12  13  14  15  16
      11  17  18  19  20
      10   9   8   7   6

Optimal path goes around the high elevations in the middle
Minimum time = 16 (max elevation along the path)
```

**Constraints:**
- $n$ == `grid.length`
- $n$ == `grid[i].length`
- $1 \leq n \leq 50$
- $0 \leq \text{grid[i][j]} < n^2$
- Each value `grid[i][j]` is **unique**.

---

## Strategy: Min-Max Path Problem

### The Key Insight: Water Level = Time

**Think of it as water filling up:**
```
Time 0: Water at depth 0
  Can only swim on cells with elevation ≤ 0

Time 5: Water at depth 5
  Can swim on any cell with elevation ≤ 5

Time t: Water at depth t
  Can swim on any cell with elevation ≤ t
```

**Question:** What's the minimum time (water level) needed to reach the destination?

**Answer:** The minimum of the maximum elevations along all possible paths!

### Why This Is Min-Max

Just like **Path With Minimum Effort**, this is a **"Min-Max"** path problem:
- **Path cost** = maximum elevation encountered along the path
- **Goal** = find path where this maximum is minimized

**Example:**
```
Path A: elevations [0, 5, 10, 3] → max = 10
Path B: elevations [0, 2, 7, 8] → max = 8
Path C: elevations [0, 1, 4, 6] → max = 6 ✓ Best!

Choose Path C (lowest maximum)
```

### Visual Intuition

```
Grid with elevations:
    0   2   5
    1   4   3
    2   3   6

Water rises over time:

t=0: Only (0,0) accessible (elevation 0)
t=1: Can reach (0,0) and (1,0) (elevations ≤ 1)
t=2: Can reach (0,0), (0,1), (1,0), (2,0) (elevations ≤ 2)
t=3: Can reach (0,0), (0,1), (1,0), (1,1), (2,0), (2,1) (elevations ≤ 3)
t=4: Can reach more cells including (1,1)

Answer: Minimum t where we can swim from (0,0) to (n-1,n-1)
```

### Three Approaches

1. **Dijkstra with Min-Heap** (Optimal, O(N² log N))
2. **Binary Search + DFS/BFS** (O(N² log(N²)))
3. **Union-Find** (Advanced, O(N² log N))

---

## Solution 1: Dijkstra with Min-Heap (Recommended)

### The Logic

- Use a **Min-Priority Queue** to always explore the cell with the lowest elevation first
- Track the **max height** seen so far on the current path
- When you reach the destination, that max height is your answer

```python
import heapq
from typing import List

class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        n = len(grid)
        
        # Min-heap stores: (max_height_on_path, row, col)
        # Start at (0,0) with initial height grid[0][0]
        pq = [(grid[0][0], 0, 0)]
        
        # Keep track of visited cells to avoid cycles
        visited = [[False] * n for _ in range(n)]
        visited[0][0] = True
        
        # 4 directions: right, left, down, up
        directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
        
        while pq:
            max_h, r, c = heapq.heappop(pq)
            
            # If we reach the destination, this is the minimum possible time
            if r == n - 1 and c == n - 1:
                return max_h
            
            # Explore all 4 neighbors
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                
                # Check bounds and if not visited
                if 0 <= nr < n and 0 <= nc < n and not visited[nr][nc]:
                    visited[nr][nc] = True
                    
                    # The time required to reach the neighbor is the
                    # maximum of the current max and the neighbor's elevation
                    new_max = max(max_h, grid[nr][nc])
                    heapq.heappush(pq, (new_max, nr, nc))
        
        return -1  # Should never reach here
```

**Key Insights:**
- **Update rule:** `new_max = max(current_max, grid[nr][nc])`
- **Not addition!** We take the maximum, not sum
- **First time we reach destination** = optimal answer (greedy property)
- **Mark visited immediately** to prevent revisiting

---

## Solution 2: Binary Search + DFS

### The Logic

**Binary Search on the Answer:**
- Answer is between 0 and max elevation in grid
- For a given time `t`, check if we can reach destination
- Use DFS/BFS to check reachability at time `t`

```python
from typing import List

class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        n = len(grid)
        
        def can_reach(t):
            """Check if we can reach (n-1, n-1) at time t"""
            # Can't even start if starting cell > t
            if grid[0][0] > t:
                return False
            
            visited = set([(0, 0)])
            stack = [(0, 0)]
            
            while stack:
                r, c = stack.pop()
                
                # Reached destination!
                if r == n - 1 and c == n - 1:
                    return True
                
                # Explore neighbors
                for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                    nr, nc = r + dr, c + dc
                    
                    # Check bounds, not visited, and elevation ≤ t
                    if (0 <= nr < n and 0 <= nc < n and 
                        (nr, nc) not in visited and grid[nr][nc] <= t):
                        visited.add((nr, nc))
                        stack.append((nr, nc))
            
            return False
        
        # Binary search on the answer
        low, high = 0, n * n - 1
        ans = high
        
        while low <= high:
            mid = (low + high) // 2
            
            if can_reach(mid):
                ans = mid  # Found a valid answer, try to find smaller
                high = mid - 1
            else:
                low = mid + 1  # Need more time
        
        return ans
```

**Key Insights:**
- **Binary search range:** [0, n²-1] (possible elevation values)
- **Monotonicity:** If reachable at time t, also reachable at time t+1
- **DFS checks reachability:** Only visit cells with elevation ≤ t
- **Time:** O(N² log(N²)) - log(N²) binary search iterations, each with O(N²) DFS

---

## Solution 3: Binary Search + BFS (Alternative)

```python
from collections import deque
from typing import List

class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        n = len(grid)
        
        def can_reach(t):
            """Check if we can reach (n-1, n-1) at time t using BFS"""
            if grid[0][0] > t:
                return False
            
            visited = [[False] * n for _ in range(n)]
            visited[0][0] = True
            queue = deque([(0, 0)])
            
            while queue:
                r, c = queue.popleft()
                
                if r == n - 1 and c == n - 1:
                    return True
                
                for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                    nr, nc = r + dr, c + dc
                    
                    if (0 <= nr < n and 0 <= nc < n and 
                        not visited[nr][nc] and grid[nr][nc] <= t):
                        visited[nr][nc] = True
                        queue.append((nr, nc))
            
            return False
        
        # Binary search on answer
        low, high = grid[0][0], max(max(row) for row in grid)
        
        while low < high:
            mid = (low + high) // 2
            
            if can_reach(mid):
                high = mid
            else:
                low = mid + 1
        
        return low
```

---

## Detailed Walkthrough

### Example: 2x2 Grid

**Input:** `grid = [[0,2],[1,3]]`

```
Grid:  0  2
       1  3

Goal: Swim from (0,0) to (1,1)
```

#### Dijkstra Approach

**Initial State:**
```python
pq = [(0, 0, 0)]  # (max_height, row, col)
visited = [[True, False],
           [False, False]]
```

| Step | Pop (max_h, r, c) | At Destination? | Explore Neighbors | Heap After |
|------|------------------|-----------------|-------------------|------------|
| 1 | (0, 0, 0) | No | Right (0,1): max(0,2)=2<br>Down (1,0): max(0,1)=1 | [(1,1,0), (2,0,1)] |
| 2 | (1, 1, 0) | No | Up (0,0): visited<br>Right (1,1): max(1,3)=3 | [(2,0,1), (3,1,1)] |
| 3 | (2, 0, 1) | No | Left (0,0): visited<br>Down (1,1): max(2,3)=3 | [(3,1,1), (3,1,1)] |
| 4 | (3, 1, 1) | **YES!** ✓ | **Return 3** | - |

**Result:** `3`

#### Binary Search Approach

**Search Range:** [0, 3]

| Iteration | low | high | mid | can_reach(mid)? | Update |
|-----------|-----|------|-----|-----------------|--------|
| 1 | 0 | 3 | 1 | No (can't reach 2 or 3) | low = 2 |
| 2 | 2 | 3 | 2 | No (can reach 0,1,2 but not 3) | low = 3 |
| 3 | 3 | 3 | 3 | Yes (all cells ≤ 3) | high = 2 |
| End | - | - | - | - | ans = 3 |

**Result:** `3`

---

## Why This Works: The Mathematics

### Dijkstra Correctness

**Claim:** Dijkstra with max operation finds minimum maximum elevation path.

**Proof:**
1. **Greedy Choice:** Always explore path with current minimum max elevation
2. **Optimal Substructure:** If optimal path to C has max elevation E, any subpath also has minimum max elevation
3. **Min-Heap Guarantee:** First time we pop destination = optimal answer

**Invariant:** When cell is popped from heap, we've found minimum max elevation to reach it.

### Binary Search Correctness

**Claim:** Binary search finds minimum t where destination is reachable.

**Proof:**
1. **Monotonicity:** If reachable at time t, also reachable at any t' > t
2. **Binary Search:** Finds smallest t where can_reach(t) = true
3. **DFS/BFS:** Correctly checks reachability using only cells with elevation ≤ t

---

## Comparison of Approaches

| Aspect | Dijkstra | Binary Search + DFS | Binary Search + BFS |
|--------|----------|---------------------|---------------------|
| **Time Complexity** | $O(N^2 \log N)$ | $O(N^2 \log(N^2))$ | $O(N^2 \log(N^2))$ |
| **Space Complexity** | $O(N^2)$ | $O(N^2)$ | $O(N^2)$ |
| **Implementation** | Medium | Medium | Medium |
| **Intuition** | Min-max path | Search answer space | Search answer space |
| **When to Use** | **Most optimal** | When binary search is clearer | When BFS preferred over DFS |

**Recommendation:** Use **Dijkstra** for optimal performance. Use binary search if you're more comfortable with that approach.

---

## Edge Cases

### 1. Single Cell (1x1 Grid)
```python
Input: grid = [[5]]
Output: 5

Already at destination, need water level ≥ 5
```

### 2. All Zeros
```python
Input: grid = [[0,0],[0,0]]
Output: 0

All cells at elevation 0, can reach immediately
```

### 3. Increasing Path
```python
Input: grid = [[0,1],[2,3]]
Output: 3

Path: 0 → 1 → 3 or 0 → 2 → 3
Both require water level ≥ 3
```

### 4. High Walls Force Detour
```python
Input: grid = [[0,1,2],
               [100,100,3],
               [4,5,6]]
Output: 6

Direct path blocked by 100s
Must go around: 0→1→2→3→6
Max elevation along path = 6
```

### 5. Starting Cell Has High Elevation
```python
Input: grid = [[10,1],[1,1]]
Output: 10

Can't even start until water level ≥ 10
```

---

## Common Mistakes

### ❌ Mistake 1: Using Addition Instead of Max
```python
# Wrong!
new_time = current_time + grid[nr][nc]  # Should use max!
```

**Fix:** Use `max(current_time, grid[nr][nc])`.

### ❌ Mistake 2: Not Marking Visited in Dijkstra
```python
# Wrong! Might revisit cells and create cycles
if 0 <= nr < n and 0 <= nc < n:
    heapq.heappush(pq, (new_max, nr, nc))
```

**Fix:** Mark as visited before adding to heap.

### ❌ Mistake 3: Wrong Binary Search Range
```python
# Wrong! Should be max elevation, not n²
low, high = 0, n * n
```

**Fix:** Use `high = max(max(row) for row in grid)` or `n² - 1`.

### ❌ Mistake 4: Forgetting Starting Cell Elevation
```python
# Wrong! Starting with 0
pq = [(0, 0, 0)]
```

**Fix:** Start with `grid[0][0]`: `pq = [(grid[0][0], 0, 0)]`.

### ❌ Mistake 5: Checking Visited After Popping in Binary Search
```python
# Wrong in binary search! Should check before adding
if (nr, nc) not in visited:
    # Explore after checking
```

**Fix:** Mark visited before adding to stack/queue to prevent duplicates.

---

## Complexity Analysis

### Dijkstra Approach

**Time Complexity:** $O(N^2 \log N)$
- Each cell added to heap once: $O(N^2)$
- Each heap operation: $O(\log N)$
- Total: $O(N^2 \log N)$

**Space Complexity:** $O(N^2)$
- Visited array: $O(N^2)$
- Priority queue: $O(N^2)$ in worst case

### Binary Search Approach

**Time Complexity:** $O(N^2 \log(N^2))$
- Binary search iterations: $O(\log(N^2)) = O(2 \log N)$
- Each iteration does DFS/BFS: $O(N^2)$
- Total: $O(N^2 \log(N^2))$

**Space Complexity:** $O(N^2)$
- Visited set/array: $O(N^2)$
- Stack/queue: $O(N^2)$ in worst case

**Comparison:**
- Dijkstra: Faster by a constant factor
- Binary Search: Easier to understand for some

---

## Interview Insights

### Q: Why is this a min-max problem?

**A:** Path cost = maximum elevation along path (not sum).
```
Goal: Minimize this maximum
Classic min-max optimization
```

### Q: What if elevations could repeat?

**A:** Algorithm still works! Uniqueness constraint just simplifies problem statement.

### Q: Can we use standard BFS?

**A:** No! BFS finds shortest path by number of edges, not by maximum elevation.

**Example:**
```
Path A: 2 edges, max elevation 100
Path B: 5 edges, max elevation 10

BFS finds A, but B is better (lower max elevation)
```

### Q: Why mark visited in Dijkstra?

**A:** Prevents infinite loops and redundant processing.
```
Without visited:
  Cell might be added multiple times with different max heights
  Causes infinite loop if cells form cycle

With visited:
  Each cell processed once
  Guaranteed termination
```

### Q: Which approach for interviews?

**A:** **Dijkstra is preferred:**
- Faster time complexity
- Shows understanding of min-max Dijkstra pattern
- More elegant solution

Binary search is acceptable but slightly less optimal.

---

## Related Problems

1. **778. Swim in Rising Water** (this problem)
2. **1631. Path With Minimum Effort** (similar min-max concept)
3. **1102. Path With Maximum Minimum Value** (maximize minimum)
4. **1368. Minimum Cost to Make at Least One Valid Path** (0-1 BFS)
5. **2290. Minimum Obstacle Removal to Reach Corner** (0-1 BFS)

---

## Key Takeaways

1. **Min-Max Problem:** Minimize the maximum elevation along path
2. **Modified Dijkstra:** Use `max(current, neighbor)` instead of `sum`
3. **Water Level = Time:** Need water level ≥ max elevation on path
4. **Mark Visited:** Essential to prevent cycles in Dijkstra
5. **Alternative Approach:** Binary search on answer with DFS/BFS
6. **Dijkstra Optimal:** $O(N^2 \log N)$ vs binary search $O(N^2 \log(N^2))$
7. **Start with grid[0][0]:** Initial max height must include starting cell
8. **Early Termination:** First pop of destination = optimal answer

**Mental Model:** Water is rising over time. You need just enough water to cover the tallest obstacle along your path from start to destination. Find the path where this "tallest obstacle" is as small as possible.


---

## Solution Implementations

Below are all three complete implementations for reference:

### Implementation 1: Dijkstra with Visited Set

```python
import heapq
from typing import List

class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        n = len(grid)
        
        # Min-heap stores: (max_height_on_path, row, col)
        pq = [(grid[0][0], 0, 0)]
        
        # Keep track of visited cells to avoid cycles
        visited = [[False] * n for _ in range(n)]
        visited[0][0] = True
        
        directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
        
        while pq:
            max_h, r, c = heapq.heappop(pq)
            
            # If we reach the destination, this is the minimum possible time
            if r == n - 1 and c == n - 1:
                return max_h
            
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                
                if 0 <= nr < n and 0 <= nc < n and not visited[nr][nc]:
                    visited[nr][nc] = True
                    # The time required to reach the neighbor is the 
                    # maximum of the current time and the neighbor's elevation
                    new_max = max(max_h, grid[nr][nc])
                    heapq.heappush(pq, (new_max, nr, nc))
        
        return -1
```

### Implementation 2: Binary Search with DFS

```python
from typing import List

class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        n, m = len(grid), len(grid[0])
        
        l, r = 0, max(max(i) for i in grid)
        
        def dfs(row, col, height):
            if (row >= 0 and row < n and col >= 0 and col < m and 
                (row, col) not in visited and grid[row][col] <= height):
                
                visited.add((row, col))
                
                for i, j in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    dfs(row + i, col + j, height)
        
        while l <= r:
            mid = (l + r) // 2
            
            visited = set()
            dfs(0, 0, mid)
            
            if (n - 1, m - 1) in visited:
                r = mid - 1
            else:
                l = mid + 1
        
        return r + 1
```

### Implementation 3: Binary Search with BFS

```python
from typing import List

class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        n = len(grid)
        
        def can_reach(t):
            if grid[0][0] > t:
                return False
            
            visited = set([(0, 0)])
            stack = [(0, 0)]
            
            while stack:
                r, c = stack.pop()
                
                if r == n - 1 and c == n - 1:
                    return True
                
                for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                    nr, nc = r + dr, c + dc
                    if (0 <= nr < n and 0 <= nc < n and 
                        (nr, nc) not in visited and grid[nr][nc] <= t):
                        visited.add((nr, nc))
                        stack.append((nr, nc))
            return False
        
        low, high = 0, n * n - 1
        ans = high
        
        while low <= high:
            mid = (low + high) // 2
            if can_reach(mid):
                ans = mid
                high = mid - 1
            else:
                low = mid + 1
        
        return ans
```

---

## Why Binary Search Works

Binary search is extremely fast: $O(\log N)$ iterations. Even if the grid is $50 \times 50$ (2,500 cells), binary search only takes about 11 steps ($\log_2 2500 \approx 11.2$) to find the answer.

**In each step:**
- We just do a simple $O(N^2)$ DFS/BFS to check reachability

**Total Complexity:**
- **Time:** $O(N^2 \log(N^2))$ 
- **Space:** $O(N^2)$ for the visited set/stack

**Why it works:**
- **Monotonicity:** If we can reach the destination at time t, we can also reach it at any time t' > t
- **Binary search finds:** The minimum t where destination becomes reachable
- **Each check:** DFS/BFS verifies if we can swim from (0,0) to (n-1,n-1) using only cells with elevation ≤ t