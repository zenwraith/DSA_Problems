# Rotting Oranges

**LeetCode Link:** [https://leetcode.com/problems/rotting-oranges/](https://leetcode.com/problems/rotting-oranges/)

---

## Problem Statement

You are given an `m x n` grid where each cell can have one of three values:

- `0` representing an empty cell
- `1` representing a fresh orange
- `2` representing a rotten orange

Every minute, any fresh orange that is **4-directionally adjacent** to a rotten orange becomes rotten.

Return the **minimum number of minutes** that must elapse until no cell has a fresh orange. If this is impossible, return `-1`.

**Example 1:**
```
Input: grid = [[2,1,1],
               [1,1,0],
               [0,1,1]]
Output: 4

Explanation:
Minute 0: [2,1,1]   Minute 1: [2,2,1]   Minute 2: [2,2,2]
          [1,1,0]             [2,1,0]             [2,2,0]
          [0,1,1]             [0,1,1]             [0,1,1]

Minute 3: [2,2,2]   Minute 4: [2,2,2]
          [2,2,0]             [2,2,0]
          [0,2,1]             [0,2,2]
```

**Example 2:**
```
Input: grid = [[2,1,1],
               [0,1,1],
               [1,0,1]]
Output: -1
Explanation: The orange in the bottom left corner (row 2, column 0) 
is never rotten, because rotting only happens 4-directionally.
```

**Example 3:**
```
Input: grid = [[0,2]]
Output: 0
Explanation: Since there are no fresh oranges at minute 0, answer is 0.
```

**Constraints:**
- $m$ == `grid.length`
- $n$ == `grid[i].length`
- $1 \leq m, n \leq 10$
- `grid[i][j]` is `0`, `1`, or `2`

---

## Strategy: Level-by-Level Multi-Source BFS

### The Key Insight

This is a **time-based propagation problem** where:
- Multiple sources (all rotten oranges) spread simultaneously
- We need to track the **number of time steps** (levels)
- Fresh oranges become rotten in **waves** (level-by-level)

### Why Level-by-Level BFS?

1. **Multi-Source:** All initially rotten oranges start spreading at the same time
2. **Time Tracking:** Each BFS level represents one minute passing
3. **Simultaneous Spread:** All oranges that rot at minute `t` spread at minute `t+1`

### Algorithm Steps

1. **Initialization:**
   - Count all fresh oranges
   - Add all initially rotten oranges to queue
   - If no fresh oranges exist, return 0

2. **BFS by Levels:**
   - Process one complete level (all current rotten oranges)
   - Each level = one minute
   - Mark newly rotted oranges and decrement fresh count

3. **Termination:**
   - If fresh count reaches 0, return minutes elapsed
   - If queue empties with fresh oranges remaining, return -1

---

## Detailed Walkthrough

### Example:
```
Initial Grid:
[2, 1, 1]
[1, 1, 0]
[0, 1, 1]
```

#### Step 0: Initialization

```
Fresh Count: 6
Queue: [(0,0)]  ← only one rotten orange initially
Minutes: 0
```

#### Minute 1: Process Level 0

**Process (0,0) - the initial rotten orange:**
- Check 4 neighbors:
  - (0,1) = 1 ✓ Fresh! Make it rotten
  - (1,0) = 1 ✓ Fresh! Make it rotten
  - (-1,0) = out of bounds
  - (0,-1) = out of bounds

```
Grid after minute 1:
[2, 2, 1]      Queue: [(0,1), (1,0)]
[2, 1, 0]      Fresh: 4
[0, 1, 1]      Minutes: 1
```

#### Minute 2: Process Level 1

**Process (0,1):**
- Neighbors:
  - (0,2) = 1 ✓ Make rotten
  - (1,1) = 1 ✓ Make rotten

**Process (1,0):**
- Neighbors:
  - (1,1) = 1 (already scheduled to rot)
  - (2,0) = 0 ✗ Empty cell

```
Grid after minute 2:
[2, 2, 2]      Queue: [(0,2), (1,1)]
[2, 2, 0]      Fresh: 2
[0, 1, 1]      Minutes: 2
```

#### Minute 3: Process Level 2

**Process (0,2):**
- No fresh neighbors

**Process (1,1):**
- Neighbors:
  - (2,1) = 1 ✓ Make rotten

```
Grid after minute 3:
[2, 2, 2]      Queue: [(2,1)]
[2, 2, 0]      Fresh: 1
[0, 2, 1]      Minutes: 3
```

#### Minute 4: Process Level 3

**Process (2,1):**
- Neighbors:
  - (2,2) = 1 ✓ Make rotten (last fresh orange!)

```
Grid after minute 4:
[2, 2, 2]      Queue: [(2,2)]
[2, 2, 0]      Fresh: 0  ← All rotted!
[0, 2, 2]      Minutes: 4
```

**Queue still has (2,2), but fresh_count = 0, so we return 4!**

---

## Key Insights

1. **Level-Based Time:** Each BFS level represents exactly one minute
   - Use `for _ in range(len(queue))` to process entire level

2. **Early Termination:** 
   - If no fresh oranges initially, return 0
   - Check fresh count after each minute, not just at the end

3. **Multi-Source Start:** All initial rotten oranges are added to queue before BFS begins

4. **In-Place Modification:** Grid values are modified directly (1 → 2)

5. **Impossibility Check:** If queue empties but fresh oranges remain, they're unreachable

---

## Edge Cases

1. **No fresh oranges initially:** Return 0 immediately
2. **Isolated fresh oranges:** Separated by empty cells or boundaries → return -1
3. **Grid of all rotten oranges:** Fresh count = 0, return 0
4. **Single cell grids:**
   - `[[2]]` → 0
   - `[[1]]` → -1
   - `[[0]]` → 0
5. **All empty cells:** Return 0

---

## Complexity Analysis

**Time Complexity:** $O(m \cdot n)$
- Each cell is processed at most once
- Each cell's 4 neighbors are checked once
- Total: $O(m \cdot n)$

**Space Complexity:** $O(m \cdot n)$
- Queue can contain up to $O(m \cdot n)$ cells
- In worst case, all cells are rotten and added to queue
- Total: $O(m \cdot n)$

---

## Implementation

```python

class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:

        rows, cols = len(grid) , len(grid[0])
        queue = deque()
        freash_count = 0
        minutes = 0

        for r in range(rows):
            for c in range(cols):

                if grid[r][c] == 2:
                    queue.append((r,c))
                elif grid[r][c] == 1:
                    freash_count +=1
        
        if freash_count == 0:
            return 0
        
        directions = [ (0,1) , (0,-1) , (1,0) , (-1,0)]

        while queue and freash_count > 0:
            minutes +=1 

            for _ in range(len(queue)):

                r,c = queue.popleft()

                for dr , dc in directions:

                    nr , nc = r+dr , c+dc

                    if 0 <= nr < rows and 0<= nc < cols and grid[nr][nc] ==1:
                        grid[nr][nc] = 2
                        freash_count -=1
                        queue.append((nr,nc))

        return minutes if freash_count == 0 else -1

```

### Code Explanation

**Lines 5-7:** Initialize data structures:
- `queue`: Stores all currently rotten oranges
- `fresh_count`: Tracks remaining fresh oranges
- `minutes`: Counts elapsed time

**Lines 9-15:** Initialization phase
- Scan entire grid
- Add all rotten oranges (2) to queue
- Count all fresh oranges (1)

**Lines 17-18:** Early termination if no fresh oranges exist

**Line 20:** Define 4-directional movements

**Lines 22-36:** Level-by-level BFS
- **Line 22:** Continue while queue has oranges AND fresh oranges remain
- **Line 23:** Increment time after each level
- **Line 25:** Process entire current level with fixed range
- **Lines 27-36:** For each rotten orange, check all 4 neighbors:
  - If neighbor is fresh (1), make it rotten (2)
  - Decrement fresh count
  - Add to queue for next level

**Line 38:** Return minutes if all oranges rotted, else -1

---

## Visual Representation

```
Example Propagation:

Minute 0:         Minute 1:         Minute 2:
[2, 1, 1]         [2, 2, 1]         [2, 2, 2]
[1, 1, 0]   →    [2, 1, 0]   →    [2, 2, 0]
[0, 1, 1]         [0, 1, 1]         [0, 1, 1]

Minute 3:         Minute 4:
[2, 2, 2]         [2, 2, 2]
[2, 2, 0]   →    [2, 2, 0]
[0, 2, 1]         [0, 2, 2]  ✓ Done!

Queue Evolution:
Minute 0: [(0,0)]
Minute 1: [(0,1), (1,0)]
Minute 2: [(0,2), (1,1)]
Minute 3: [(2,1)]
Minute 4: [(2,2)]
```

**Impossible Case:**
```
[2, 1, 1]    Fresh orange at (2,0) is isolated
[0, 1, 1]    by the empty cell (1,0)
[1, 0, 1]    → Return -1
   ↑
 Can't reach!
```

---

## Alternative Approaches Comparison

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| **Level BFS** | $O(mn)$ | $O(mn)$ | Clear time tracking, optimal | - |
| Regular BFS | $O(mn)$ | $O(mn)$ | - | Hard to track time accurately |
| DFS | $O(mn \cdot mn)$ | $O(mn)$ | - | Doesn't handle simultaneous spread |
| Simulation | $O(k \cdot mn)$ | $O(mn)$ | Easy to understand | Slower, $k$ = number of rounds |

---

## Common Pitfalls

1. **Not processing by levels:** Missing the time-based nature of spread
2. **Forgetting early termination:** Checking `fresh_count == 0` after entire BFS instead of during
3. **Off-by-one errors:** Incrementing minutes at wrong point in loop
4. **Not handling "no fresh oranges" case:** Should return 0, not -1
5. **Checking fresh count in wrong place:** Should check before processing level, not after

---

## Why This Pattern Matters

This problem demonstrates:
- **Level-order BFS** for time-based simulations
- **Multi-source BFS** for simultaneous spreading
- **State tracking** during graph traversal
- **Early termination** optimizations

---

## Related Problems

- **994. Rotting Oranges** (this problem)
- **542. 01 Matrix** (multi-source BFS, distance-based)
- **1091. Shortest Path in Binary Matrix** (single-source BFS)
- **1263. Minimum Moves to Move a Box** (multi-level BFS state)
- **1129. Shortest Path with Alternating Colors** (state-based BFS)

---
