# Minimum Obstacle Removal to Reach Corner

**LeetCode 2290** | **Difficulty:** Hard | **Category:** 0-1 BFS, Dijkstra

---

## Problem Statement

You are given a 0-indexed 2D integer array `grid` of size `m x n`. Each cell has one of two values:
- `0` represents an **empty** cell
- `1` represents an **obstacle** that may be removed

You can move up, down, left, or right from and to an empty cell.

Return the **minimum number of obstacles** to remove so you can move from the upper left corner `(0, 0)` to the lower right corner `(m - 1, n - 1)`.

---

## Examples

### Example 1:
```
Input: grid = [[0,1,1],[1,1,0],[1,1,0]]
Output: 2

Explanation: 
We can remove the obstacles at (0, 1) and (0, 2) to create a path from (0, 0) to (2, 2).
```

### Example 2:
```
Input: grid = [[0,1,0,0,0],[0,1,0,1,0],[0,0,0,1,0]]
Output: 0

Explanation: 
We can move from (0, 0) to (2, 4) without removing any obstacles, so we return 0.
```

---

## Strategy

### The Key Insight: 0-1 BFS

This problem is a **shortest path problem** with a special property:

**Graph Modeling:**
- Each cell is a node
- Moving to an **empty cell** (0) has weight 0
- Moving to an **obstacle** (1) has weight 1 (you must remove it)
- Goal: Find path with minimum total weight

**Why 0-1 BFS?**

While we could use Dijkstra's algorithm (which would be $O(N^2 \log N^2)$), there's a faster specialized approach called **0-1 BFS** when edge weights are only 0 or 1.

**0-1 BFS Core Principle:**
```
Use a deque (double-ended queue) instead of a priority queue:
• If you move to a neighbor with weight 0 → add to FRONT (appendleft)
• If you move to a neighbor with weight 1 → add to BACK (append)
```

**Why this works:**
- The deque maintains implicit sorting by distance
- All cells with distance `d` are processed before cells with distance `d+1`
- This gives us the same optimality guarantee as Dijkstra but in linear time!

**Complexity:**
- **Time:** $O(N \times M)$ where N = rows, M = cols
- **Space:** $O(N \times M)$ for the distance matrix

---

## Solution

```python
from collections import deque
from typing import List

class Solution:
    def minimumObstacles(self, grid: List[List[int]]) -> int:
        rows, cols = len(grid), len(grid[0])
        
        # Distance matrix: dist[r][c] = minimum obstacles to remove to reach (r, c)
        dist = [[float('inf')] * cols for _ in range(rows)]
        dist[0][0] = 0
        
        # Deque for 0-1 BFS: stores (row, col)
        dq = deque([(0, 0)])
        
        # 4 directions: right, left, down, up
        directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
        
        while dq:
            r, c = dq.popleft()
            
            # Early termination: reached bottom-right corner
            if r == rows - 1 and c == cols - 1:
                return dist[r][c]
            
            # Explore all 4 neighbors
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                
                # Check bounds
                if 0 <= nr < rows and 0 <= nc < cols:
                    # Weight = 0 for empty cell, 1 for obstacle
                    weight = grid[nr][nc]
                    
                    # Relaxation: update if we found a better path
                    if dist[r][c] + weight < dist[nr][nc]:
                        dist[nr][nc] = dist[r][c] + weight
                        
                        # 0-1 BFS MAGIC:
                        # Weight 0: higher priority → add to front
                        # Weight 1: lower priority → add to back
                        if weight == 0:
                            dq.appendleft((nr, nc))
                        else:
                            dq.append((nr, nc))
        
        return dist[rows - 1][cols - 1]
```

---

## Detailed Walkthrough

### Example: `grid = [[0,1,1],[1,1,0],[1,1,0]]`

**Visual Grid:**
```
(0,0) (0,1) (0,2)
  0     1     1
  
(1,0) (1,1) (1,2)
  1     1     0
  
(2,0) (2,1) (2,2)
  1     1     0
```

**Step-by-Step Execution:**

| Step | Deque | Current | Neighbor | Weight | New Dist | Action | Deque After |
|------|-------|---------|----------|--------|----------|--------|-------------|
| 0 | `[(0,0)]` | - | - | - | dist[0][0]=0 | Initialize | `[(0,0)]` |
| 1 | `[(0,0)]` | (0,0) | (0,1) | 1 | 0+1=1 | Append back | `[(1,0), (0,1)]` |
| 1 | `[(0,0)]` | (0,0) | (1,0) | 1 | 0+1=1 | Append back | `[(1,0), (0,1)]` |
| 2 | `[(1,0), (0,1)]` | (1,0) | (2,0) | 1 | 1+1=2 | Append back | `[(0,1), (1,1), (2,0)]` |
| 2 | `[(1,0), (0,1)]` | (1,0) | (1,1) | 1 | 1+1=2 | Append back | `[(0,1), (1,1), (2,0)]` |
| 3 | `[(0,1), (1,1), (2,0)]` | (0,1) | (0,2) | 1 | 1+1=2 | Append back | `[(1,1), (2,0), (0,2)]` |
| 3 | `[(0,1), (1,1), (2,0)]` | (0,1) | (1,1) | 1 | 1+1=2 | Already 2, skip | `[(1,1), (2,0), (0,2)]` |
| 4 | `[(1,1), (2,0), (0,2)]` | (1,1) | (1,2) | 0 | 2+0=2 | **Append front** | `[(1,2), (2,0), (0,2), (2,1)]` |
| 5 | `[(1,2), (2,0), (0,2), (2,1)]` | (1,2) | (2,2) | 0 | 2+0=2 | **Append front** | `[(2,2), ...]` |
| 6 | `[(2,2), ...]` | (2,2) | Target! | - | **Return 2** | Done | - |

**Final Distance Matrix:**
```
dist = [
  [0, 1, 2],
  [1, 2, 2],
  [2, 3, 2]
]
```

**Optimal Path:** (0,0) → (1,0) → (1,1) → (1,2) → (2,2)
- Obstacles removed: positions (1,0) and (1,1) = **2 obstacles**

---

## Why Not Standard BFS?

**Standard BFS Limitation:**
A standard BFS (using a normal queue) **only works when every edge has the same weight**.

**Problem with Standard BFS Here:**
```python
# Standard BFS would do this:
queue = deque([(0, 0)])
# Process all neighbors equally
for neighbor in neighbors:
    queue.append(neighbor)  # All go to back
```

**Issue:** You might reach a cell via an obstacle-heavy path first, even though a "free" path (all 0s) exists.

**Example:**
```
Grid: [[0, 1],
       [0, 0]]

Standard BFS might explore:
  (0,0) → (0,1) [obstacle!] → (1,1)
  
But optimal is:
  (0,0) → (1,0) → (1,1) [no obstacles!]
```

**0-1 BFS Guarantee:** By adding weight-0 edges to the front, we always process the "cheapest" paths first, ensuring optimality.

---

## Alternative: Dijkstra's Algorithm

You can also solve this with standard Dijkstra:

```python
import heapq

def minimumObstacles(grid):
    rows, cols = len(grid), len(grid[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    dist[0][0] = 0
    
    # Min-heap: (obstacles_removed, row, col)
    pq = [(0, 0, 0)]
    
    while pq:
        d, r, c = heapq.heappop(pq)
        
        if r == rows - 1 and c == cols - 1:
            return d
        
        if d > dist[r][c]:
            continue
        
        for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols:
                new_dist = d + grid[nr][nc]
                if new_dist < dist[nr][nc]:
                    dist[nr][nc] = new_dist
                    heapq.heappush(pq, (new_dist, nr, nc))
    
    return dist[rows-1][cols-1]
```

**Complexity:**
- **Time:** $O(N \times M \times \log(N \times M))$
- **Space:** $O(N \times M)$

**Comparison:**
| Approach | Time Complexity | When to Use |
|----------|----------------|-------------|
| **0-1 BFS** | $O(N \times M)$ | Weights are only 0 and 1 |
| **Dijkstra** | $O(N \times M \times \log(N \times M))$ | General weighted graphs |

**Verdict:** 0-1 BFS is **faster and simpler** for this specific problem!

---

## Edge Cases

1. **Already clear path (no obstacles):**
   ```python
   grid = [[0, 0], [0, 0]]
   # Output: 0
   ```

2. **Start or end is an obstacle:**
   ```python
   grid = [[1, 0], [0, 0]]
   # Must remove start obstacle
   # Output: 1
   ```

3. **Single cell:**
   ```python
   grid = [[0]]
   # Already at destination
   # Output: 0
   ```

4. **All obstacles:**
   ```python
   grid = [[1, 1], [1, 1]]
   # Must remove 3 obstacles for shortest path
   # Output: 3
   ```

---

## Common Mistakes

### ❌ Mistake 1: Using Standard BFS
```python
# Wrong! Doesn't guarantee shortest path
queue = deque([(0, 0)])
while queue:
    r, c = queue.popleft()
    for neighbor in neighbors:
        queue.append(neighbor)  # Treats all equally
```

**Fix:** Use 0-1 BFS with deque front/back strategy.

### ❌ Mistake 2: Not Checking Distance Before Adding
```python
# Wrong! Can add duplicate states
if new_dist < dist[nr][nc]:
    dist[nr][nc] = new_dist
    # Missing check before adding to deque
```

**Fix:** Always update distance and then add to deque.

### ❌ Mistake 3: Using Weight from Current Cell Instead of Next Cell
```python
# Wrong!
weight = grid[r][c]  # Current cell's value
```

**Fix:**
```python
# Correct!
weight = grid[nr][nc]  # Next cell's value (the one we're moving TO)
```

---

## Interview Insights

**Question Recognition:**
- Grid + shortest path + **binary weights (0/1)** → Think 0-1 BFS
- Keywords: "minimum obstacles", "empty/blocked cells"

**Optimization Discussion:**
- Interviewer: "Can you optimize further?"
  - You: "0-1 BFS is already optimal at $O(N \times M)$ for this binary weight case!"

**Follow-up Questions:**
1. **Q:** "What if obstacles have different removal costs?"
   - **A:** Use Dijkstra with priority queue.

2. **Q:** "What if we can only remove K obstacles?"
   - **A:** State becomes (r, c, obstacles_removed), use BFS/Dijkstra.

3. **Q:** "Can you solve without extra space?"
   - **A:** Can modify grid in-place, but generally not recommended in interviews.

**Complexity Trade-offs:**
- 0-1 BFS: Best for binary weights
- Dijkstra: More general, but slightly slower
- Standard BFS: Incorrect for weighted graphs!

---

## Related Problems

1. **Shortest Path in Binary Matrix (LeetCode 1091)** - 0-1 BFS on grid
2. **Path With Minimum Effort (LeetCode 1631)** - Min-max Dijkstra
3. **Swim in Rising Water (LeetCode 778)** - Min-max path finding
4. **Cheapest Flights Within K Stops (LeetCode 787)** - Constrained shortest path

---

## Summary

**Key Takeaways:**

1. **Problem Type:** Shortest path with binary weights (0 or 1)

2. **Algorithm:** 0-1 BFS with deque
   - Weight 0 → front (higher priority)
   - Weight 1 → back (lower priority)

3. **Why It Works:** Deque maintains implicit distance ordering

4. **Complexity:** 
   - Time: $O(N \times M)$ - Linear!
   - Space: $O(N \times M)$ - Distance matrix

5. **Mental Model:** Think of removing obstacles as having a cost. We want the path with minimum total cost (obstacles removed).

**When to Use 0-1 BFS:**
- Shortest path problem ✓
- Edge weights are only 0 or 1 ✓
- Want better than Dijkstra's $O(E \log V)$ ✓