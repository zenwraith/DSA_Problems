# 01 Matrix

**LeetCode Link:** [https://leetcode.com/problems/01-matrix/](https://leetcode.com/problems/01-matrix/)

---

## Problem Statement

Given an `m x n` binary matrix `mat`, return the distance of the nearest `0` for each cell.

The distance between two adjacent cells is `1`.

**Example 1:**
```
Input: mat = [[0,0,0],
              [0,1,0],
              [0,0,0]]
Output: [[0,0,0],
         [0,1,0],
         [0,0,0]]
```

**Example 2:**
```
Input: mat = [[0,0,0],
              [0,1,0],
              [1,1,1]]
Output: [[0,0,0],
         [0,1,0],
         [1,2,1]]
```

**Example 3:**
```
Input: mat = [[0,1,1],
              [1,1,1],
              [1,1,0]]
Output: [[0,1,2],
         [1,2,1],
         [2,1,0]]
```

**Constraints:**
- $m$ == `mat.length`
- $n$ == `mat[i].length`
- $1 \leq m, n \leq 10^4$
- $1 \leq m \cdot n \leq 10^4$
- `mat[i][j]` is either `0` or `1`
- There is at least one `0` in `mat`

---

## Strategy: Multi-Source BFS

### The Key Insight

Instead of running BFS from each `1` cell to find the nearest `0` (which would be $O(m \cdot n \cdot m \cdot n)$), we **reverse the problem**:

- Start BFS from **all `0` cells simultaneously** (multi-source BFS)
- Propagate distances outward level by level
- Each `1` cell is reached by the nearest `0` first

### Why Multi-Source BFS Works

Think of it like a "flood fill" from all zeros:
- **Level 0:** All `0` cells (distance = 0)
- **Level 1:** All cells adjacent to `0`s (distance = 1)
- **Level 2:** All cells 2 steps away from nearest `0` (distance = 2)
- And so on...

Since BFS explores in increasing distance order, the first time we reach a cell is guaranteed to be via the shortest path.

### Algorithm Steps

1. **Initialization Phase:**
   - Add all `0` cells to the queue (they already have distance 0)
   - Mark all `1` cells as `-1` (unvisited)

2. **BFS Phase:**
   - Process queue level by level
   - For each cell, explore 4 neighbors
   - Update neighbor's distance if it's unvisited (`-1`)
   - Add newly discovered cells to queue

3. **Result:**
   - The matrix now contains the shortest distances

---

## Detailed Walkthrough

### Example: 
```
Input:  [[0, 0, 0],
         [0, 1, 0],
         [1, 1, 1]]
```

#### Step 1: Initialization

**Mark all 1s as -1, add all 0s to queue:**
```
Matrix:         Queue:
[0,  0,  0]     [(0,0), (0,1), (0,2), (1,0), (1,2)]
[0, -1,  0]     
[-1,-1,-1]      
```

#### Step 2: BFS Level 0

Process cells with distance 0 (all zeros). Their neighbors will get distance 1.

**Process (0,0):** neighbors: (0,1)✗, (1,0)✗ (both already 0)

**Process (0,1):** neighbors: (0,0)✗, (0,2)✗, (1,1)✓ (found -1!)
```
Matrix:         Queue adds:
[0, 0, 0]       (1,1) with dist 1
[0, 1, 0]       
[-1,-1,-1]      
```

**Process (0,2):** neighbors: (0,1)✗, (1,2)✗

**Process (1,0):** neighbors: (0,0)✗, (1,1)✓ (already scheduled), (2,0)✓
```
Matrix:         Queue adds:
[0, 0, 0]       (2,0) with dist 1
[0, 1, 0]       
[1,-1,-1]      
```

**Process (1,2):** neighbors: (0,2)✗, (1,1)✓ (already scheduled), (2,2)✓
```
Matrix:         Queue adds:
[0, 0, 0]       (2,2) with dist 1
[0, 1, 0]       
[1,-1, 1]      
```

**After Level 0:**
```
Matrix:         Queue:
[0, 0, 0]       [(1,1), (2,0), (2,2)]
[0, 1, 0]       
[1,-1, 1]      
```

#### Step 3: BFS Level 1

Process cells with distance 1.

**Process (1,1):** neighbors: (1,0)✗, (1,2)✗, (0,1)✗, (2,1)✓
```
Matrix:         Queue adds:
[0, 0, 0]       (2,1) with dist 2
[0, 1, 0]       
[1, 2, 1]      
```

**Process (2,0):** neighbors: (1,0)✗, (2,1)✓ (already scheduled)

**Process (2,2):** neighbors: (1,2)✗, (2,1)✓ (already scheduled)

**After Level 1:**
```
Matrix:         Queue:
[0, 0, 0]       [(2,1)]
[0, 1, 0]       
[1, 2, 1]      
```

#### Step 4: BFS Level 2

**Process (2,1):** All neighbors already visited

**Final Result:**
```
[[0, 0, 0],
 [0, 1, 0],
 [1, 2, 1]]
```

---

## Key Insights

1. **Multi-Source BFS:** Starting from all zeros simultaneously is more efficient than individual BFS from each 1

2. **In-Place Modification:** Using `-1` to mark unvisited cells allows us to modify the matrix in place

3. **Level-Order Guarantee:** BFS guarantees that cells are reached in increasing distance order

4. **No Visited Set Needed:** The distance values themselves indicate visited status

5. **Optimal Substructure:** If cell A is distance `d` from nearest zero, its unvisited neighbors are distance `d+1`

---

## Edge Cases

1. **All zeros:** Queue processes all cells immediately, all distances remain 0
2. **Single cell matrix:** Returns `[[0]]` or `[[0]]` (depending on value)
3. **Single row/column:** BFS works in 1D effectively
4. **Checkerboard pattern:** Each cell has distance 1 from a zero
5. **Large sparse matrices:** Efficient since only explores necessary cells

---

## Complexity Analysis

**Time Complexity:** $O(m \cdot n)$
- Each cell is added to the queue at most once
- Each cell's 4 neighbors are checked once
- Total operations: $O(4 \cdot m \cdot n) = O(m \cdot n)$

**Space Complexity:** $O(m \cdot n)$
- Queue can contain up to $O(m \cdot n)$ cells in worst case
- No additional space needed (in-place modification)
- Total: $O(m \cdot n)$

---

## Implementation

```python

class Solution:
    def updateMatrix(self, mat: List[List[int]]) -> List[List[int]]:

        rows, cols = len(mat) , len(mat[0])

        queue = deque()


        for r in range(rows):
            for c in range(cols):
                if mat[r][c] == 0:
                    queue.append((r,c))
                else:
                    mat[r][c] = -1

        directions = [ (1,0) , (-1,0) , (0,1) , (0,-1)]

        while queue:
            r,c = queue.popleft()

            for dr, dc in directions:
                nr , nc = r + dr , c + dc

                if 0 <= nr < rows and 0 <= nc < cols and mat[nr][nc] == -1:

                    mat[nr][nc] = mat[r][c] + 1
                    queue.append((nr,nc))

        return mat


```

### Code Explanation

**Lines 5-6:** Get matrix dimensions

**Lines 8-14:** Initialization phase
- Add all `0` cells to queue (they're the starting points)
- Mark all `1` cells as `-1` (unvisited marker)

**Line 16:** Define 4-directional movement (down, up, right, left)

**Lines 18-28:** BFS phase
- Process queue until empty
- For each cell, check all 4 neighbors
- If neighbor is unvisited (`-1`), update its distance and add to queue
- Distance = current cell's distance + 1

**Line 30:** Return the updated matrix with all distances

---

## Visual Representation

```
Example: [[0, 0, 0],     Step-by-step propagation:
         [0, 1, 0],
         [1, 1, 1]]      Level 0 (all 0s):
                         [0, 0, 0]
                         [0, _, 0]
                         [_, _, _]
                         Queue: all zero positions
                         
                         Level 1 (distance 1):
                         [0, 0, 0]
                         [0, 1, 0]
                         [1, _, 1]
                         Queue: (1,1), (2,0), (2,2)
                         
                         Level 2 (distance 2):
                         [0, 0, 0]
                         [0, 1, 0]
                         [1, 2, 1]
                         Queue: (2,1)
                         
                         Done!
```

**Propagation Animation:**
```
0 = zero cell (source)
· = processed
X = processing

Step 1:   [0  0  0]       Step 2:   [0  0  0]       Step 3:   [0  0  0]
          [0  X  0]                 [0  1  0]                 [0  1  0]
          [X  _  X]                 [1  X  1]                 [1  2  1]
```

---

## Alternative Approaches Comparison

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| BFS from each 1 | $O((mn)^2)$ | $O(mn)$ | Simple to understand | Too slow for large grids |
| **Multi-source BFS** | $O(mn)$ | $O(mn)$ | Optimal time, single pass | - |
| Dynamic Programming | $O(mn)$ | $O(1)$ | In-place, no queue | Two passes needed, less intuitive |
| DFS from each 1 | $O((mn)^2)$ | $O(mn)$ | - | Doesn't guarantee shortest path |

---

## Common Pitfalls

1. **Running BFS from each 1:** Results in $O((mn)^2)$ time complexity
2. **Not marking cells as visited:** Can cause infinite loops
3. **Using boolean visited array:** Wastes space when we can use `-1` marker
4. **Processing neighbors before checking bounds:** Array index errors
5. **Forgetting to add newly discovered cells to queue:** Incomplete BFS

---

## Related Problems

- **542. 01 Matrix** (this problem)
- **994. Rotting Oranges** (similar multi-source BFS)
- **1091. Shortest Path in Binary Matrix** (single-source BFS)
- **286. Walls and Gates** (same technique, premium)

---
