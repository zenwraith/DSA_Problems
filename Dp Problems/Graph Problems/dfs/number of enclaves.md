# Number of Enclaves

**LeetCode Link:** [https://leetcode.com/problems/number-of-enclaves/](https://leetcode.com/problems/number-of-enclaves/)

---

## Problem Statement

You are given an `m x n` binary matrix `grid`, where `0` represents a sea cell and `1` represents a land cell.

A **move** consists of walking from one land cell to another adjacent (4-directionally) land cell or walking off the boundary of the grid.

Return the **number of land cells** in `grid` for which we **cannot** walk off the boundary of the grid in any number of moves.

**Example 1:**
```
Input: grid = [[0,0,0,0],
               [1,0,1,0],
               [0,1,1,0],
               [0,0,0,0]]
Output: 3

Explanation:
There are three 1s that are enclosed by 0s, and one 1 that is not enclosed
because its on the boundary.
```

**Example 2:**
```
Input: grid = [[0,1,1,0],
               [0,0,1,0],
               [0,0,1,0],
               [0,0,0,0]]
Output: 0

Explanation:
All 1s are either on the boundary or can reach the boundary.
```

**Example 3:**
```
Input: grid = [[0,0,0,1,1,1,0,1,1,0],
               [1,1,1,1,0,1,0,1,1,0],
               [0,1,0,1,1,0,0,0,0,0],
               [1,0,1,1,1,1,1,0,1,0]]
Output: 7

Explanation:
There are 7 land cells that cannot reach the boundary.
```

**Constraints:**
- $m$ == `grid.length`
- $n$ == `grid[i].length`
- $1 \leq m, n \leq 500$
- `grid[i][j]` is either `0` or `1`

---

## Strategy: Boundary DFS Elimination

### The Key Insight

Instead of checking each land cell to see if it can reach a boundary (expensive!), we **reverse the problem**:

1. **Eliminate all land cells connected to boundaries** (these can walk off)
2. **Count remaining land cells** (these are enclaves)

**Why This Works:**
- Any land cell connected to a boundary cell can walk off
- "Connected" means reachable via adjacent land cells
- DFS from boundaries marks all escapable land
- What remains must be enclosed!

### Visual Intuition

```
Original Grid:          After Boundary DFS:         Count Enclaves:
[0, 0, 0, 0]           [0, 0, 0, 0]                [·, ·, ·, ·]
[1, 0, 1, 0]     →    [0, 0, 1, 0]      →        [·, ·, 1, ·]  = 3
[0, 1, 1, 0]           [0, 1, 1, 0]                [·, 1, 1, ·]
[0, 0, 0, 0]           [0, 0, 0, 0]                [·, ·, ·, ·]
   ↑                      ↑
Boundary land          Eliminated!
```

### Algorithm Steps

1. **Boundary Scan:**
   - Scan all four edges of the grid
   - For each land cell (`1`) on boundary, run DFS

2. **DFS Elimination:**
   - Mark current cell as sea (`0`) - "sink the land"
   - Recursively visit all 4 adjacent land cells
   - This eliminates entire connected component

3. **Count Remaining:**
   - After all boundary DFS completes
   - Count all remaining `1`s - these are enclaves

---

## Detailed Walkthrough

### Example:
```
grid = [[0, 0, 0, 0],
        [1, 0, 1, 0],
        [0, 1, 1, 0],
        [0, 0, 0, 0]]
```

#### Step 1: Scan Top and Bottom Boundaries

**Top row (row 0):**
- (0,0) = 0 ✗
- (0,1) = 0 ✗
- (0,2) = 0 ✗
- (0,3) = 0 ✗

**Bottom row (row 3):**
- (3,0) = 0 ✗
- (3,1) = 0 ✗
- (3,2) = 0 ✗
- (3,3) = 0 ✗

No land on top/bottom boundaries.

#### Step 2: Scan Left and Right Boundaries

**Left column (col 0):**
- (0,0) = 0 ✗
- (1,0) = 1 ✓ **Found boundary land! Start DFS**

**DFS from (1,0):**
```
Visit (1,0): grid[1][0] = 1 → sink to 0
  Check neighbors:
    (0,0) = 0 ✗
    (2,0) = 0 ✗
    (1,-1) = out of bounds
    (1,1) = 0 ✗
  No more land to sink.

Grid after DFS from (1,0):
[0, 0, 0, 0]
[0, 0, 1, 0]  ← (1,0) sunk
[0, 1, 1, 0]
[0, 0, 0, 0]
```

Continue left column:
- (2,0) = 0 ✗
- (3,0) = 0 ✗

**Right column (col 3):**
- (0,3) = 0 ✗
- (1,3) = 0 ✗
- (2,3) = 0 ✗
- (3,3) = 0 ✗

No more boundary land found.

#### Step 3: Count Remaining Land

```
Final grid:
[0, 0, 0, 0]
[0, 0, 1, 0]  ← 1 enclave
[0, 1, 1, 0]  ← 2 enclaves
[0, 0, 0, 0]

Count = 3 ✓
```

---

## More Complex Example

```
Initial:
[0, 1, 1, 0]
[0, 0, 1, 0]
[0, 0, 1, 0]
[0, 0, 0, 0]
```

#### Boundary DFS:

**Top row:** (0,1) = 1 → **DFS!**
```
DFS from (0,1):
  Sink (0,1)
  Visit (0,2): Sink → Visit (1,2): Sink → Visit (2,2): Sink
  
Grid after:
[0, 0, 0, 0]
[0, 0, 0, 0]
[0, 0, 0, 0]
[0, 0, 0, 0]

All land eliminated! Count = 0
```

---

## Key Insights

1. **Boundary Elimination:** Only need to check edges, not entire grid

2. **In-Place Modification:** Sinking land to `0` acts as both:
   - Visited marker (prevents revisiting)
   - Elimination (removes escapable land)

3. **DFS Choice:** DFS is perfect because:
   - We need to explore entire connected component
   - No need for level-order traversal
   - Recursive solution is clean

4. **Two-Phase Approach:**
   - Phase 1: Eliminate (DFS from boundaries)
   - Phase 2: Count (scan entire grid)

5. **Connected Components:** Each DFS call eliminates one connected component that touches the boundary

---

## Edge Cases

1. **All land touches boundary:** Return 0
2. **No land cells:** Return 0
3. **All land is enclosed:** Return total land count
4. **Single cell grid:**
   - `[[0]]` → 0
   - `[[1]]` → 0 (boundary cell)
5. **Checkerboard pattern:** Most land touches boundaries
6. **Large connected component:** DFS handles efficiently with stack

---

## Complexity Analysis

**Time Complexity:** $O(m \cdot n)$
- Boundary scan: $O(2m + 2n) = O(m + n)$
- DFS: Each cell visited at most once: $O(m \cdot n)$
- Final count: $O(m \cdot n)$
- Total: $O(m \cdot n)$

**Space Complexity:** $O(m \cdot n)$ 
- Recursion stack in worst case (all cells are land in single component)
- In-place modification means no extra space for visited set
- Total: $O(m \cdot n)$ for call stack

**Optimization:** Can reduce to $O(\min(m,n))$ stack space with iterative DFS using explicit stack.

---

## Implementation

```python

class Solution:
    def numEnclaves(self, grid: List[List[int]]) -> int:

        n,m = len(grid) , len(grid[0])

        def dfs(r,c):
            # Boundary check and checking if it's land
            if r < 0 or r >= n or c < 0 or c >=m or grid[r][c] == 0:
                return 
            # "Sink" the land so we don't visit it again (acts as visited set)
            grid[r][c] = 0
            # Visit all 4 neighbors
            dfs(r+1, c)
            dfs(r-1,c)
            dfs(r, c+1)
            dfs(r , c-1)

        # Step 1: Sink all land connected to the Left and Right boundaries
        for r in range(n):
            dfs(r, 0) # left edge
            dfs(r, m-1) # right edge
        # Step 2: Sink all land connected to the Top and Bottom boundaries
        for c in range(m):
            dfs(0, c) # top edge
            dfs(n-1 ,  c) # bottom edge
        
        count  = 0
        # Step 3: Count the remaining 1s
        for r in range(n):
            for c in range(m):
                if grid[r][c] == 1:
                    count +=1
        
        return count

```

### Code Explanation

**Lines 5-6:** Get grid dimensions

**Lines 8-16:** DFS helper function
- **Line 9:** Base cases:
  - Out of bounds
  - Already water (0) or already visited
- **Line 11:** Sink the land (mark as visited and eliminate)
- **Lines 13-16:** Recursively visit all 4 neighbors

**Lines 19-21:** **Phase 1 - Left and Right boundaries**
- For each row, DFS from leftmost and rightmost cells
- Eliminates all land connected to left/right edges

**Lines 23-25:** **Phase 1 - Top and Bottom boundaries**
- For each column, DFS from topmost and bottommost cells
- Eliminates all land connected to top/bottom edges

**Lines 27-32:** **Phase 2 - Count enclaves**
- Scan entire grid
- Count remaining land cells (enclaves)

---

## Visual Representation

```
Example: Boundary Elimination

Initial State:
[0, 0, 0, 0]
[1, 0, 1, 0]  ← Left boundary has land at (1,0)
[0, 1, 1, 0]
[0, 0, 0, 0]

After DFS from (1,0):
[0, 0, 0, 0]
[·, 0, 1, 0]  ← (1,0) eliminated
[0, 1, 1, 0]
[0, 0, 0, 0]

No more boundary land.

Final - Count remaining:
[·, ·, ·, ·]
[·, ·, 1, ·]  ← Enclave!
[·, 1, 1, ·]  ← Enclaves!
[·, ·, ·, ·]

Result: 3
```

**Complex Example:**
```
[0, 1, 1, 0, 0]
[0, 0, 1, 1, 0]
[0, 1, 0, 1, 0]
[0, 1, 1, 1, 0]
[0, 0, 0, 0, 0]

DFS from (0,1):
Sinks: (0,1) → (0,2) → (1,2) → (1,3) → (2,3) → (3,3) → (3,2) → (3,1) → (2,1)
All connected to boundary eliminated!

Final: [0, ·, ·, 0, 0]
       [0, ·, ·, ·, 0]
       [0, ·, 0, ·, 0]
       [0, ·, ·, ·, 0]
       [0, 0, 0, 0, 0]

No enclaves! Return 0
```

---

## Alternative Approaches

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| **Boundary DFS** | $O(mn)$ | $O(mn)$ | Intuitive, optimal | Modifies input |
| BFS from boundaries | $O(mn)$ | $O(mn)$ | Same performance | More code |
| Union-Find | $O(mn \cdot \alpha(mn))$ | $O(mn)$ | No recursion | More complex, slower |
| Check each cell | $O((mn)^2)$ | $O(mn)$ | - | Too slow |

---

## Common Pitfalls

1. **Checking from interior:** Starting DFS from random cells instead of boundaries
2. **Not sinking visited cells:** Causes infinite loops
3. **Boundary scan bugs:** Missing corners or edges
4. **Stack overflow:** For very large grids, consider iterative DFS
5. **Not restoring grid:** If you need original values, make a copy first

---

## Pattern Recognition

This is a **"Reverse Elimination"** pattern:
- Instead of finding what you want, eliminate what you don't want
- Common in:
  - Surrounded Regions (LC 130) - flip O's to X's except boundary-connected
  - Pacific Atlantic Water Flow (LC 417) - DFS from both oceans
  - Number of Closed Islands (LC 1254) - similar boundary elimination

---

## Related Problems

- **1020. Number of Enclaves** (this problem)
- **130. Surrounded Regions** (same technique, different goal)
- **200. Number of Islands** (basic DFS/BFS)
- **695. Max Area of Island** (DFS with counting)
- **1254. Number of Closed Islands** (boundary elimination)
- **417. Pacific Atlantic Water Flow** (multi-source DFS)

---
