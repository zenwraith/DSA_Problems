# Path With Minimum Effort

**LeetCode Link:** [https://leetcode.com/problems/path-with-minimum-effort/](https://leetcode.com/problems/path-with-minimum-effort/)

---

## Problem Statement

You are a hiker preparing for an upcoming hike. You are given `heights`, a 2D array of size `rows x columns`, where `heights[row][col]` represents the height of cell `(row, col)`. You are situated in the top-left cell, `(0, 0)`, and you hope to travel to the bottom-right cell, `(rows-1, columns-1)` (i.e., **0-indexed**). You can move **up, down, left, or right**, and you wish to find a route that requires the **minimum effort**.

A route's **effort** is the **maximum absolute difference** in heights between two consecutive cells of the route.

Return the **minimum effort** required to travel from the top-left cell to the bottom-right cell.

**Example 1:**
```
Input: heights = [[1,2,2],[3,8,2],[5,3,5]]
Output: 2

Explanation:
Grid:  1 → 2 → 2
       ↓       ↓
       3   8   2
       ↓       ↓
       5 → 3 → 5

Path: (0,0) → (1,0) → (2,0) → (2,1) → (2,2)
Heights: 1 → 3 → 5 → 3 → 5
Differences: |1-3|=2, |3-5|=2, |5-3|=2, |3-5|=2
Maximum difference: 2
```

**Example 2:**
```
Input: heights = [[1,2,2],[3,8,2],[5,3,5]]
Output: 2

Explanation:
Another path: (0,0) → (0,1) → (0,2) → (1,2) → (2,2)
Heights: 1 → 2 → 2 → 2 → 5
Differences: |1-2|=1, |2-2|=0, |2-2|=0, |2-5|=3
Maximum difference: 3 (worse than first path)

The minimum effort across all paths is 2.
```

**Example 3:**
```
Input: heights = [[1,2,3],[3,8,4],[5,3,5]]
Output: 1

Explanation:
Path: (0,0) → (0,1) → (0,2) → (1,2) → (2,2)
Heights: 1 → 2 → 3 → 4 → 5
Differences: |1-2|=1, |2-3|=1, |3-4|=1, |4-5|=1
Maximum difference: 1
```

**Constraints:**
- $\text{rows}$ == `heights.length`
- $\text{columns}$ == `heights[i].length`
- $1 \leq \text{rows, columns} \leq 100$
- $1 \leq \text{heights[i][j]} \leq 10^6$

---

## Strategy: Modified Dijkstra's Algorithm for "Min-Max" Path

### The Key Insight: Not Sum, But Maximum!

**Traditional Dijkstra:** Find path with minimum **sum** of edge weights.
```
Path cost = edge1 + edge2 + edge3 + ...
```

**This Problem:** Find path with minimum **maximum** of edge weights.
```
Path effort = max(edge1, edge2, edge3, ...)
```

**Example:**
```
Path A: edges [5, 2, 3] → effort = max(5, 2, 3) = 5
Path B: edges [4, 4, 4] → effort = max(4, 4, 4) = 4
Choose Path B! (lower maximum)
```

### Why Dijkstra Still Works

**Dijkstra's Key Property:** Greedy choice of minimum cost path is optimal.

**Proof this works for min-max:**
- If we reach a cell with effort $E$, we've found the minimum effort path to that cell
- Any alternative path with lower max effort would have been explored first (min-heap guarantees this)
- The greedy property holds: local optimum → global optimum

### Algorithm Overview

1. **Distance Matrix:** `dist[r][c]` = minimum effort to reach cell `(r, c)`
   - Initialize all to `∞`, except `dist[0][0] = 0`

2. **Min-Heap:** Store `(effort, row, col)` tuples
   - Always process the cell with minimum effort first

3. **Update Rule:** For each neighbor:
   ```python
   new_effort = max(current_effort, abs(height_difference))
   if new_effort < dist[neighbor]:
       dist[neighbor] = new_effort
       push to heap
   ```

4. **Termination:** First time we pop destination cell `(rows-1, cols-1)`, return its effort

### Visual Intuition

```
Heights:  1   2   10
          3   4   5
          6   7   8

Think of effort as "difficulty":
- Going 1→2: effort |1-2|=1 (easy)
- Going 2→10: effort |2-10|=8 (hard!)
- Going 1→3→4→5→8: max effort along path

Goal: Find path where the "hardest step" is as easy as possible
```

---

## Solution: Dijkstra with Min-Heap

```python
import heapq
from typing import List

class Solution:
    def minimumEffortPath(self, heights: List[List[int]]) -> int:
        rows, cols = len(heights), len(heights[0])
        
        # dist[r][c] = minimum "max-absolute-difference" to reach (r,c)
        dist = [[float('inf')] * cols for _ in range(rows)]
        dist[0][0] = 0
        
        # Min-heap: (current_effort, row, col)
        min_heap = [(0, 0, 0)]
        
        # 4 directions: right, left, down, up
        directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
        
        while min_heap:
            current_effort, r, c = heapq.heappop(min_heap)
            
            # Early termination: reached destination!
            if r == rows - 1 and c == cols - 1:
                return current_effort
            
            # Optimization: Skip if we've already found a better path to this cell
            if current_effort > dist[r][c]:
                continue
            
            # Explore all 4 neighbors
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                
                # Check bounds
                if 0 <= nr < rows and 0 <= nc < cols:
                    # Calculate effort to reach neighbor
                    # = max of (current effort, effort to move to neighbor)
                    new_effort = max(current_effort, abs(heights[nr][nc] - heights[r][c]))
                    
                    # If this path is better than previously found path
                    if new_effort < dist[nr][nc]:
                        dist[nr][nc] = new_effort
                        heapq.heappush(min_heap, (new_effort, nr, nc))
        
        # Should never reach here (destination always reachable)
        return 0
```

---

## Detailed Walkthrough

### Example: 3x3 Grid with Varying Heights

**Input:** `heights = [[1,2,2],[3,8,2],[5,3,5]]`

**Grid Visualization:**
```
     0   1   2
   +---+---+---+
0  | 1 | 2 | 2 |
   +---+---+---+
1  | 3 | 8 | 2 |
   +---+---+---+
2  | 5 | 3 | 5 |
   +---+---+---+

Goal: (0,0) → (2,2) with minimum effort
```

#### Initial State

**Distance Matrix:**
```
dist = [[0,   ∞,   ∞],
        [∞,   ∞,   ∞],
        [∞,   ∞,   ∞]]
```

**Min-Heap:** `[(0, 0, 0)]`

#### Step-by-Step Execution

| Step | Pop (effort, r, c) | At Destination? | Explore Neighbors | Update dist | Heap After |
|------|-------------------|-----------------|-------------------|-------------|------------|
| 1 | (0, 0, 0) | No | Right (0,1): \|1-2\|=1, max(0,1)=1<br>Down (1,0): \|1-3\|=2, max(0,2)=2 | dist[0][1]=1<br>dist[1][0]=2 | [(1,0,1), (2,1,0)] |
| 2 | (1, 0, 1) | No | Left (0,0): skip (worse)<br>Right (0,2): \|2-2\|=0, max(1,0)=1<br>Down (1,1): \|2-8\|=6, max(1,6)=6 | dist[0][2]=1<br>dist[1][1]=6 | [(1,0,2), (2,1,0), (6,1,1)] |
| 3 | (1, 0, 2) | No | Left (0,1): skip<br>Down (1,2): \|2-2\|=0, max(1,0)=1 | dist[1][2]=1 | [(1,1,2), (2,1,0), (6,1,1)] |
| 4 | (1, 1, 2) | No | Up (0,2): skip<br>Down (2,2): \|2-5\|=3, max(1,3)=3 | dist[2][2]=3 | [(2,1,0), (3,2,2), (6,1,1)] |
| 5 | (2, 1, 0) | No | Up (0,0): skip<br>Down (2,0): \|3-5\|=2, max(2,2)=2<br>Right (1,1): skip (worse) | dist[2][0]=2 | [(2,2,0), (3,2,2), (6,1,1)] |
| 6 | (2, 2, 0) | No | Up (1,0): skip<br>Right (2,1): \|5-3\|=2, max(2,2)=2 | dist[2][1]=2 | [(2,2,1), (3,2,2), (6,1,1)] |
| 7 | (2, 2, 1) | No | Left (2,0): skip<br>Up (1,1): skip<br>Right (2,2): \|3-5\|=2, max(2,2)=2 | dist[2][2]=2 (better!) | [(2,2,2), (3,2,2), (6,1,1)] |
| 8 | **(2, 2, 2)** | **YES! ✓** | **Return 2** | - | - |

**Final Distance Matrix:**
```
dist = [[0,   1,   1],
        [2,   6,   1],
        [2,   2,   2]]
```

**Optimal Path:** `(0,0) → (1,0) → (2,0) → (2,1) → (2,2)`
- Heights: `1 → 3 → 5 → 3 → 5`
- Efforts: `|1-3|=2, |3-5|=2, |5-3|=2, |3-5|=2`
- Maximum: `2` ✓

---

## Why This Works: The Mathematics

### Proof of Correctness

**Claim:** Dijkstra with max operation finds the minimum maximum effort path.

**Proof:**
1. **Greedy Choice:** We always explore the path with current minimum effort
2. **Optimal Substructure:** If path P to cell C has minimum effort E, then any subpath of P also has minimum effort for its endpoint
3. **No Negative Edges:** All efforts are non-negative (absolute differences)
4. **Min-Heap Guarantees:** When we pop a cell, we've found the optimal effort to reach it

**Key Invariant:** `dist[r][c]` = minimum effort to reach `(r,c)` among all explored paths.

### Why Early Termination Works

**Question:** Why can we return immediately when we pop the destination?

**Answer:**
```
When destination is popped from min-heap:
  - It has the smallest effort among all remaining cells
  - All paths with smaller effort already explored
  - No future path can have smaller effort
  - This is the optimal answer!
```

---

## Comparison: Standard Dijkstra vs Min-Max Dijkstra

| Aspect | Standard Dijkstra | Min-Max Dijkstra (This Problem) |
|--------|------------------|--------------------------------|
| **Goal** | Minimum **sum** of weights | Minimum **maximum** of weights |
| **Update Rule** | `dist + weight` | `max(dist, weight)` |
| **Edge Weight** | Edge weight | \|height difference\| |
| **Use Case** | Shortest path (GPS) | Minimum difficulty (hiking) |
| **Example** | Path [3,2,1] → cost 6 | Path [3,2,1] → effort 3 |
| **Heap Structure** | (cost, node) | (effort, row, col) |
| **Early Exit** | Same (first pop of dest) | Same (first pop of dest) |

### Code Comparison

**Standard Dijkstra (Sum):**
```python
new_cost = dist[node] + edge_weight
if new_cost < dist[neighbor]:
    dist[neighbor] = new_cost
    heappush(heap, (new_cost, neighbor))
```

**Min-Max Dijkstra (This Problem):**
```python
new_effort = max(current_effort, abs(height_diff))
if new_effort < dist[neighbor]:
    dist[neighbor] = new_effort
    heappush(heap, (new_effort, neighbor))
```

**Only difference:** `dist + weight` → `max(dist, weight)`

---

## Alternative Approaches

### 1. Binary Search + BFS (More Complex)

**Idea:** Binary search on the answer, check if path exists with effort ≤ mid.

```python
def minimumEffortPath(self, heights):
    def canReach(max_effort):
        # BFS to check if destination reachable with effort ≤ max_effort
        rows, cols = len(heights), len(heights[0])
        visited = [[False] * cols for _ in range(rows)]
        queue = deque([(0, 0)])
        visited[0][0] = True
        
        while queue:
            r, c = queue.popleft()
            if r == rows - 1 and c == cols - 1:
                return True
            
            for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
                nr, nc = r + dr, c + dc
                if (0 <= nr < rows and 0 <= nc < cols and 
                    not visited[nr][nc] and 
                    abs(heights[nr][nc] - heights[r][c]) <= max_effort):
                    visited[nr][nc] = True
                    queue.append((nr, nc))
        
        return False
    
    # Binary search on answer
    left, right = 0, max(max(row) for row in heights)
    result = right
    
    while left <= right:
        mid = (left + right) // 2
        if canReach(mid):
            result = mid
            right = mid - 1
        else:
            left = mid + 1
    
    return result
```

**Complexity:** $O(\text{rows} \times \text{cols} \times \log(\text{max height}))$

**Why Dijkstra is better:** Simpler, same complexity in practice.

### 2. Union-Find (Advanced)

Sort all edges by weight, use Union-Find to connect cells until start and end are connected.

**Complexity:** $O(\text{edges} \times \log(\text{edges}))$

**Trade-off:** More complex, similar performance.

---

## Edge Cases

### 1. Single Cell (Already at Destination)
```python
Input: heights = [[5]]
Output: 0

Explanation: Already at destination, no movement needed
```

### 2. Flat Terrain (All Same Height)
```python
Input: heights = [[1,1,1],[1,1,1]]
Output: 0

Explanation: All height differences are 0
```

### 3. Extreme Height Difference
```python
Input: heights = [[1,10],[2,3]]
Output: 2

Path: (0,0) → (1,0) → (1,1)
Heights: 1 → 2 → 3
Efforts: |1-2|=1, |2-3|=1
Max: 1

Alternative path: (0,0) → (0,1) → (1,1)
Heights: 1 → 10 → 3
Efforts: |1-10|=9, |10-3|=7
Max: 9 (worse!)

Choose first path with effort 1
```

### 4. Multiple Optimal Paths
```python
Input: heights = [[1,2,3],[2,3,4],[3,4,5]]
Output: 1

Multiple paths exist with max effort 1:
- Diagonal path
- Right then down
- Down then right
All equally good!
```

### 5. Forced High Effort Path
```python
Input: heights = [[1,2,1,1,1],
                  [1,99,99,99,1],
                  [1,99,99,99,1],
                  [1,1,1,1,1]]
Output: 1

Must go around the obstacle (99s)
Path along edges has max effort 1
```

---

## Common Mistakes

### ❌ Mistake 1: Using Sum Instead of Max
```python
# Wrong!
new_effort = current_effort + abs(heights[nr][nc] - heights[r][c])
```

**Fix:** Use `max()` not `+`.

### ❌ Mistake 2: Not Using Min-Heap
```python
# Wrong! (BFS doesn't guarantee optimal for this problem)
queue = deque([(0, 0, 0)])  # Regular queue
```

**Fix:** Use `heapq` min-heap to always explore minimum effort first.

### ❌ Mistake 3: Forgetting to Skip Stale Entries
```python
# Wrong! Might process same cell multiple times
current_effort, r, c = heapq.heappop(min_heap)
# Missing: if current_effort > dist[r][c]: continue
```

**Fix:** Skip if we've found a better path already.

### ❌ Mistake 4: Not Handling 1x1 Grid
```python
# Wrong! Returns 0 from end of function (misleading)
if rows == 1 and cols == 1:
    return 0
```

**Fix:** Early termination handles this naturally (first pop is destination).

### ❌ Mistake 5: Wrong Update Condition
```python
# Wrong!
if new_effort > dist[nr][nc]:  # Should be <, not >!
    dist[nr][nc] = new_effort
```

**Fix:** Update only if new effort is **smaller**.

---

## Complexity Analysis

**Time Complexity:** $O(\text{rows} \times \text{cols} \times \log(\text{rows} \times \text{cols}))$
- Each cell can be added to heap at most once per update
- In worst case, each cell updated $O(\text{degree})$ times ≈ 4 times
- Heap operations (push/pop): $O(\log(\text{heap size}))$
- Heap size: $O(\text{rows} \times \text{cols})$
- Total: $O(\text{rows} \times \text{cols} \times \log(\text{rows} \times \text{cols}))$

**Space Complexity:** $O(\text{rows} \times \text{cols})$
- Distance matrix: $O(\text{rows} \times \text{cols})$
- Min-heap: $O(\text{rows} \times \text{cols})$ in worst case
- Overall: $O(\text{rows} \times \text{cols})$

**Practical Performance:**
- For 100×100 grid: ~10,000 cells, very fast
- Early termination often means we don't explore all cells
- Typical real runtime much better than worst case

---

## Optimization: Skip Stale Entries

**Why is this optimization important?**

```python
if current_effort > dist[r][c]:
    continue
```

**Without this:**
- Cell (2,2) might be added to heap with effort 5
- Later, better path found, dist[2][2] updated to 3
- Eventually, stale entry (5, 2, 2) is popped
- Waste time exploring neighbors with worse effort

**With this:**
- Stale entry popped, immediately skipped
- No wasted neighbor exploration
- Significant speedup in practice!

---

## Key Interview Insights

### Q: Why Dijkstra and not BFS?

**A:** BFS doesn't work because:
```
BFS explores by number of steps, not by effort.

Example:
  Path A: 2 steps, effort [100, 100] → max = 100
  Path B: 5 steps, effort [1, 1, 1, 1, 1] → max = 1

BFS would find Path A first (fewer steps)
But Path B is optimal (lower effort)!

Dijkstra explores by effort → finds optimal path
```

### Q: Why can we terminate early?

**A:** Min-heap guarantee:
```
When destination is popped:
  effort = E

All remaining cells in heap have effort ≥ E
No future path can have effort < E
First pop of destination = optimal answer!
```

### Q: What if we used DFS?

**A:** DFS explores all paths (exponential time):
```python
def dfs(r, c, max_effort):
    if r == rows-1 and c == cols-1:
        return max_effort
    
    min_effort = infinity
    for neighbor in neighbors:
        effort = max(max_effort, height_diff)
        min_effort = min(min_effort, dfs(neighbor, effort))
    
    return min_effort

# Time: O(4^(rows*cols)) - exponential!
```

**Dijkstra is much faster:** $O(\text{rows} \times \text{cols} \times \log(\text{rows} \times \text{cols}))$

---

## Related Problems

- **1631. Path With Minimum Effort** (this problem)
- **778. Swim in Rising Water** (similar min-max concept)
- **1514. Path with Maximum Probability** (modified Dijkstra for max product)
- **1368. Minimum Cost to Make at Least One Valid Path** (0-1 BFS variant)
- **743. Network Delay Time** (standard Dijkstra)

---

## Key Takeaways

1. **Min-Max Problem:** Find path where the maximum edge weight is minimized
2. **Modified Dijkstra:** Use `max(current_effort, edge_weight)` instead of `sum`
3. **Greedy Property:** Still optimal because we explore minimum effort first
4. **Early Termination:** First pop of destination = optimal answer
5. **Skip Stale Entries:** Optimization to avoid processing outdated heap entries
6. **Not BFS:** BFS doesn't work because it optimizes step count, not effort
7. **Min-Heap Essential:** Priority queue ensures we explore minimum effort paths first

**Mental Model:** Think of effort as the "bottleneck" of a path. We want to find the path where the tightest bottleneck is as wide as possible (i.e., the maximum step is as small as possible).
