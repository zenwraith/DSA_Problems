# Shortest Path in Binary Matrix

**LeetCode Link:** [https://leetcode.com/problems/shortest-path-in-binary-matrix/](https://leetcode.com/problems/shortest-path-in-binary-matrix/)

---

## Problem Statement

Given an `n x n` binary matrix `grid`, return the length of the **shortest clear path** in the matrix. If there is no clear path, return `-1`.

A **clear path** in a binary matrix is a path from the **top-left cell** `(0, 0)` to the **bottom-right cell** `(n - 1, n - 1)` such that:
- All the visited cells of the path are `0`
- All the adjacent cells of the path are **8-directionally connected** (i.e., they share an edge or a corner)

The **length of a clear path** is the number of visited cells of this path.

**Example 1:**
```
Input: grid = [[0,1],[1,0]]
Output: 2

Explanation:
Grid:  0  1
       1  0

Path: (0,0) → (1,1)
Length: 2
```

**Example 2:**
```
Input: grid = [[0,0,0],[1,1,0],[1,1,0]]
Output: 4

Explanation:
Grid:  0  0  0
       1  1  0
       1  1  0

Path: (0,0) → (0,1) → (0,2) → (1,2) → (2,2)
Length: 4
```

**Example 3:**
```
Input: grid = [[1,0,0],[1,1,0],[1,1,0]]
Output: -1

Explanation:
Grid:  1  0  0    (Start cell is blocked!)
       1  1  0
       1  1  0

No path possible since grid[0][0] = 1
```

**Constraints:**
- $n$ == `grid.length`
- $n$ == `grid[i].length`
- $1 \leq n \leq 100$
- `grid[i][j]` is `0` or `1`

---

## Strategy: Multi-Source BFS with 8 Directions

### Key Insight: 8-Directional Movement

**Unlike most grid problems** that only allow moving **Up, Down, Left, and Right** (4 directions), this problem allows **8 directions** including diagonals:

```
Neighbors of cell (r, c):

  (-1,-1)  (-1,0)  (-1,1)
    ↖️      ⬆️      ↗️
  
  (0,-1)   (r,c)   (0,1)
    ⬅️      ●       ➡️
  
  (1,-1)   (1,0)   (1,1)
    ↙️      ⬇️      ↘️
```

**Movement array:**
```python
directions = [
    (-1,-1), (-1,0), (-1,1),  # Top-left, Top, Top-right
    (0,-1),          (0,1),    # Left,         Right
    (1,-1),  (1,0),  (1,1)     # Bottom-left, Bottom, Bottom-right
]
```

### Why BFS?

**Shortest path in an unweighted graph** → BFS is optimal!

**BFS Guarantee:** The first time you reach the destination, it's via the shortest path.

**Proof:**
- BFS explores nodes layer by layer by distance
- Distance 1, then distance 2, then distance 3, etc.
- When destination is reached, no shorter path exists (would have been found earlier)

### Algorithm Overview

1. **Early exit:** If start `(0,0)` or end `(n-1, n-1)` is blocked (`1`), return `-1`
2. **Initialize BFS:**
   - Queue: `[(0, 0, distance=1)]` (start position with distance 1)
   - Mark `grid[0][0] = 1` to prevent revisiting
3. **BFS Loop:**
   - Pop current cell `(r, c, dist)`
   - If reached destination `(n-1, n-1)`, return `dist`
   - Explore all 8 neighbors
   - For each valid unvisited neighbor (value `0`):
     - Mark as visited (set to `1`)
     - Add to queue with `dist + 1`
4. **If queue exhausted:** No path exists, return `-1`

### Critical Optimization: Mark When Discovered

**❌ Wrong (Slow):**
```python
# Mark visited when POPPING from queue
r, c, dist = queue.popleft()
visited[r][c] = True  # Too late!
```

**✅ Correct (Fast):**
```python
# Mark visited when ADDING to queue
if grid[nr][nc] == 0:
    grid[nr][nc] = 1  # Mark immediately!
    queue.append((nr, nc, dist+1))
```

**Why?**
- If we wait to mark cells until popping, the **same cell can be added multiple times** by different neighbors
- This creates duplicate entries in the queue → wasted space and time
- Marking at discovery ensures each cell is added **exactly once**

---

## Solution: BFS with In-Place Marking

```python
from collections import deque
from typing import List

class Solution:
    def shortestPathBinaryMatrix(self, grid: List[List[int]]) -> int:
        n = len(grid)
        
        # Edge case: Start or end is blocked
        if grid[0][0] == 1 or grid[n-1][n-1] == 1:
            return -1
        
        # BFS Queue: (row, col, current_distance)
        # Start at (0,0) with path length 1 (counts the cell itself)
        queue = deque([(0, 0, 1)])
        
        # Mark start as visited (use 1 since we can't revisit)
        grid[0][0] = 1
        
        # 8 directions: diagonals + orthogonal
        directions = [
            (-1, -1), (-1, 0), (-1, 1),  # Top-left, Top, Top-right
            (0, -1),           (0, 1),    # Left,          Right
            (1, -1),  (1, 0),  (1, 1)     # Bottom-left, Bottom, Bottom-right
        ]
        
        while queue:
            r, c, dist = queue.popleft()
            
            # Check if we reached the destination
            if r == n - 1 and c == n - 1:
                return dist
            
            # Explore all 8 neighbors
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                
                # Check bounds and if cell is unvisited (value 0)
                if 0 <= nr < n and 0 <= nc < n and grid[nr][nc] == 0:
                    # Mark as visited immediately (optimization!)
                    grid[nr][nc] = 1
                    queue.append((nr, nc, dist + 1))
        
        # No path found
        return -1
```

---

## Detailed Walkthrough

### Example: 3x3 Grid with Obstacles

**Input:** `grid = [[0,0,0],[1,1,0],[1,1,0]]`

**Grid Visualization:**
```
  0   1   2
0 S → 0 → 0
1 X   X   0
2 X   X   E

S = Start (0,0)
E = End (2,2)
X = Blocked (1)
0 = Clear path
```

#### Step-by-Step BFS

**Initial State:**
- Queue: `[(0,0,1)]`
- Grid after marking start:
```
  1   0   0
  1   1   0
  1   1   0
```

| Step | Pop (r,c,dist) | Check Destination? | Explore Neighbors | Queue After | Grid State |
|------|----------------|-------------------|-------------------|-------------|------------|
| 0 | - | - | - | `[(0,0,1)]` | `[[1,0,0],[1,1,0],[1,1,0]]` |
| 1 | (0,0,1) | No (not at 2,2) | Check 8 neighbors:<br>• (-1,-1): out of bounds<br>• (-1,0): out of bounds<br>• (-1,1): out of bounds<br>• (0,-1): out of bounds<br>• **(0,1): valid! Add (0,1,2)**<br>• (1,-1): out of bounds<br>• (1,0): blocked (1)<br>• (1,1): blocked (1) | `[(0,1,2)]` | `[[1,1,0],[1,1,0],[1,1,0]]` |
| 2 | (0,1,2) | No | Check 8 neighbors:<br>• (-1,-1): out of bounds<br>• (-1,0): out of bounds<br>• (-1,1): out of bounds<br>• (0,-1): visited (1)<br>• **(0,2): valid! Add (0,2,3)**<br>• (1,0): blocked (1)<br>• (1,1): blocked (1)<br>• **(1,2): valid! Add (1,2,3)** | `[(0,2,3),(1,2,3)]` | `[[1,1,1],[1,1,1],[1,1,0]]` |
| 3 | (0,2,3) | No | Check 8 neighbors:<br>• All except (1,2) are blocked/visited/out of bounds<br>• (1,2) already visited (marked in previous step) | `[(1,2,3)]` | `[[1,1,1],[1,1,1],[1,1,0]]` |
| 4 | (1,2,3) | No | Check 8 neighbors:<br>• Most are blocked/visited<br>• **(2,2): valid! Add (2,2,4)** | `[(2,2,4)]` | `[[1,1,1],[1,1,1],[1,1,1]]` |
| 5 | (2,2,4) | **YES!** ✓ | Reached destination! | - | - |

**Result:** `4`

**Path:** `(0,0) → (0,1) → (0,2) → (1,2) → (2,2)`

---

## Visual Representation

### Distance Propagation

```
Step 0:           Step 1:           Step 2:
  1   .   .         1   2   .         1   2   3
  X   X   .         X   X   .         X   X   3
  X   X   .         X   X   .         X   X   .

Step 3:           Step 4 (Done!):
  1   2   3         1   2   3
  X   X   3         X   X   3
  X   X   .         X   X   4 ← Found!
```

Numbers represent the distance from the start when each cell was reached.

---

## Why This Works: BFS Properties

### 1. Layer-by-Layer Exploration

BFS explores cells in **waves** of increasing distance:
```
Distance 1: Start cell
Distance 2: All cells 1 step away
Distance 3: All cells 2 steps away
...
```

### 2. First Reach = Shortest Path

**Theorem:** In an unweighted graph, BFS finds the shortest path.

**Proof sketch:**
- Suppose there's a shorter path of length $k$
- BFS would explore all cells at distance $< k$ before distance $k$
- Contradiction: we would have found the shorter path first

### 3. In-Place Marking = No Duplicates

By marking cells as visited **when added to queue**:
- Each cell is added **at most once**
- Queue size is bounded by $O(n^2)$
- No redundant processing

---

## Edge Cases

### 1. Start or End Blocked
```python
Input: grid = [[1,0],[0,0]]
Output: -1

Grid:  X  0
       0  0

Start is blocked, impossible!
```

### 2. Start = End (1x1 Grid)
```python
Input: grid = [[0]]
Output: 1

Grid:  S/E

Already at destination, path length 1
```

### 3. Direct Diagonal Path
```python
Input: grid = [[0,1],[1,0]]
Output: 2

Grid:  S  X
       X  E

Path: (0,0) → (1,1) diagonally
```

### 4. No Path Exists
```python
Input: grid = [[0,1],[1,0]]
Output: -1 (if diagonal blocked)

Grid:  0  X
       X  0

All paths blocked by obstacles
```

### 5. All Clear Path
```python
Input: grid = [[0,0],[0,0]]
Output: 2

Grid:  S  0
       0  E

Multiple paths, all have length 2
Path: (0,0) → (1,1) or (0,0) → (0,1) → (1,1)
First is shorter!
```

---

## Common Mistakes

### ❌ Mistake 1: Marking Visited Too Late
```python
# Wrong!
r, c, dist = queue.popleft()
visited[r][c] = True  # Same cell added multiple times before this
```

**Fix:** Mark when adding to queue, not when popping.

### ❌ Mistake 2: Forgetting to Check Start/End
```python
# Wrong! Might try to traverse through blocked cells
queue = deque([(0, 0, 1)])
```

**Fix:** Check `grid[0][0] == 1` and `grid[n-1][n-1] == 1` first.

### ❌ Mistake 3: Wrong Initial Distance
```python
queue = deque([(0, 0, 0)])  # Wrong! Path length should be 1
```

**Fix:** Start with distance `1` (the start cell counts as 1 cell in the path).

### ❌ Mistake 4: Using Only 4 Directions
```python
# Wrong! Problem allows diagonals
directions = [(-1,0), (1,0), (0,-1), (0,1)]  # Only 4 directions
```

**Fix:** Use all 8 directions including diagonals.

### ❌ Mistake 5: Not Handling n x n Grid
```python
# Wrong! Assumes rows = cols, but should use n
rows, cols = len(grid), len(grid[0])
```

**Fix:** For this problem, `n = len(grid)` (square grid guaranteed).

---

## Complexity Analysis

**Time Complexity:** $O(n^2)$
- Each cell is visited **at most once** (marked when discovered)
- For each cell, we check 8 neighbors → $O(8) = O(1)$
- Total: $O(n^2 \times 8) = O(n^2)$

**Space Complexity:** $O(n^2)$
- **Queue:** In the worst case (no obstacles), all cells could be in the queue → $O(n^2)$
- **In-place marking:** We modify the input grid, so no extra visited array needed
- **Overall:** $O(n^2)$

**Space optimization:** We could use a separate `visited` set if we need to preserve the input grid, but that doesn't change the asymptotic complexity.

---

## Key Interview Insights

### Why Overwrite `grid[nr][nc] = 1` Immediately?

**Question:** "Why mark as visited when adding to queue instead of when popping?"

**Answer:**
```
Without immediate marking:
  Cell (1,1) gets discovered by:
    - (0,0) → adds (1,1) to queue
    - (0,1) → adds (1,1) to queue again!
    - (1,0) → adds (1,1) yet again!
  
  Queue becomes huge with duplicates!

With immediate marking:
  Cell (1,1) gets discovered by (0,0):
    - Mark grid[1][1] = 1 immediately
    - When (0,1) checks (1,1), it's already marked → skip
    - When (1,0) checks (1,1), it's already marked → skip
  
  Each cell added exactly once!
```

### BFS Shortest Path Guarantee

**Question:** "How do we know BFS finds the shortest path?"

**Answer:**
```
BFS explores in layers of increasing distance:
  Layer 0: Start (distance 1)
  Layer 1: All neighbors of start (distance 2)
  Layer 2: All neighbors of Layer 1 (distance 3)
  ...

When we first reach the destination:
  - All shorter distances already explored
  - No shorter path possible
  - Guaranteed optimal!
```

### 8 Directions vs 4 Directions

**Question:** "What changes if we only allow 4 directions?"

**Answer:**
```
8 Directions (this problem):
  grid = [[0,1],
          [1,0]]
  Path: (0,0) → (1,1) diagonally
  Length: 2

4 Directions only:
  grid = [[0,1],
          [1,0]]
  No path! Both orthogonal paths blocked.
  Length: -1

Impact: Diagonal moves can significantly shorten paths!
```

---

## Alternative Approach: A* Search (Overkill)

For this problem, BFS is optimal. But if you want to optimize further (e.g., on very large grids), you could use **A\* search** with Manhattan distance heuristic:

```python
import heapq

def shortestPathBinaryMatrix(self, grid: List[List[int]]) -> int:
    n = len(grid)
    if grid[0][0] == 1 or grid[n-1][n-1] == 1:
        return -1
    
    # Priority queue: (estimated_total, distance, row, col)
    pq = [(1 + max(n-1, n-1), 1, 0, 0)]  # f = g + h
    grid[0][0] = 1
    
    directions = [(-1,-1), (-1,0), (-1,1), (0,-1), (0,1), (1,-1), (1,0), (1,1)]
    
    while pq:
        _, dist, r, c = heapq.heappop(pq)
        
        if r == n-1 and c == n-1:
            return dist
        
        for dr, dc in directions:
            nr, nc = r + dr, c + dc
            if 0 <= nr < n and 0 <= nc < n and grid[nr][nc] == 0:
                grid[nr][nc] = 1
                h = max(abs(n-1-nr), abs(n-1-nc))  # Chebyshev distance
                heapq.heappush(pq, (dist + 1 + h, dist + 1, nr, nc))
    
    return -1
```

**Why A\* might help:** Prioritizes cells closer to the destination.

**Why it's overkill here:** 
- BFS is already $O(n^2)$ which is optimal for this problem
- A\* adds complexity without improving worst-case complexity
- The grid is small ($n \leq 100$)

---

## Related Problems

- **1091. Shortest Path in Binary Matrix** (this problem)
- **542. 01 Matrix** (multi-source BFS from all 0s)
- **994. Rotting Oranges** (multi-source BFS with time tracking)
- **1197. Minimum Knight Moves** (BFS with 8 directions like chess knight)
- **365. Water and Jug Problem** (BFS on state space)

---

## Key Takeaways

1. **8-directional BFS** allows diagonal movement (not just up/down/left/right)
2. **Mark cells when adding to queue**, not when popping (prevents duplicates)
3. **BFS guarantees shortest path** in unweighted graphs
4. **First time reaching destination = optimal path** (no need to explore further)
5. **In-place marking** saves space by reusing the input grid
6. **Early exit** for blocked start/end saves time
7. **Path length starts at 1** (counts the start cell)

**Mental Model:** BFS explores the grid in **expanding ripples** from the start, like water spreading. The first ripple to reach the destination is the shortest path!