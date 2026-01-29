

# **Longest Increasing Path in a Matrix**

**LeetCode Link:** [https://leetcode.com/problems/longest-increasing-path-in-a-matrix/](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

---

## **Problem Statement**

Given an `m x n` integers matrix, return the length of the longest increasing path in the matrix.

From each cell, you can either move in four directions: left, right, up, or down. You **may not** move diagonally or move outside the boundary.

**Example:**
```
Input: matrix = [[9,9,4],
                 [6,6,8],
                 [2,1,1]]
Output: 4
Explanation: The longest increasing path is [1, 2, 6, 9].

Input: matrix = [[3,4,5],
                 [3,2,6],
                 [2,2,1]]
Output: 4
Explanation: The longest increasing path is [3, 4, 5, 6].
```

---

## **Strategy: DFS with Memoization**

**Key Insight:** The "strictly increasing" constraint creates a **Directed Acyclic Graph (DAG)**. We can never revisit a cell in the same path because we only move to cells with greater values.

### **Why No Visited Set?**

In typical grid DFS problems (like finding islands), you need a `visited` set to avoid infinite loops:
```python
visited.add((r, c))
dfs(neighbor)
visited.remove((r, c))  # Backtrack
```

**But here:** The strictly increasing rule acts as a natural "visited" check!
- From cell with value 5, you can only move to cells with value > 5
- You can **never** return to value 5 in the current path
- This creates a DAG (no cycles possible)

### **The Algorithm**

1. **For each cell**, start a DFS to find the longest path from that cell
2. **DFS logic**: Try all 4 directions, only move if neighbor value is **strictly greater**
3. **Memoization**: Cache results to avoid recomputing
4. **Return**: Maximum path length across all starting cells

---

## **Solution**

```python
class Solution:
    def longestIncreasingPath(self, matrix: List[List[int]]) -> int:
        if not matrix or not matrix[0]:
            return 0
        
        rows, cols = len(matrix), len(matrix[0])
        memo = {}
        
        def dfs(r, c):
            # Return cached result if already computed
            if (r, c) in memo:
                return memo[(r, c)]
            
            # Base case: current cell is a path of length 1
            longest = 1
            
            # Try all 4 directions
            for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                nr, nc = r + dr, c + dc
                
                # Check bounds and strictly increasing condition
                if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                    # Recursively find longest path from neighbor
                    longest = max(longest, 1 + dfs(nr, nc))
            
            # Cache and return result
            memo[(r, c)] = longest
            return longest
        
        # Try starting from every cell
        max_path = 0
        for r in range(rows):
            for c in range(cols):
                max_path = max(max_path, dfs(r, c))
        
        return max_path
```

---

## **Example Walkthrough**

**Input:**
```
matrix = [[9, 9, 4],
          [6, 6, 8],
          [2, 1, 1]]
```

### **DFS from (2, 1) - Value 1**

```
Start at (2,1): value = 1

Check neighbors:
  (2,0): value = 2 > 1 ✓ Can move!
    From (2,0): value = 2
    Check neighbors:
      (1,0): value = 6 > 2 ✓ Can move!
        From (1,0): value = 6
        Check neighbors:
          (0,0): value = 9 > 6 ✓ Can move!
            From (0,0): value = 9
            No neighbors with value > 9
            Return 1
          Path: 6 → 9 gives 1 + 1 = 2
        Return 2
      Path: 2 → 6 → 9 gives 1 + 2 = 3
    Return 3
  Path: 1 → 2 → 6 → 9 gives 1 + 3 = 4

Result: 4
```

**Memoization Effect:**
```
After computing dfs(0, 0), memo[(0,0)] = 1
After computing dfs(1, 0), memo[(1,0)] = 2
After computing dfs(2, 0), memo[(2,0)] = 3
After computing dfs(2, 1), memo[(2,1)] = 4

If we later try dfs(0, 0) again from a different path,
we immediately return 1 without recomputation!
```

---

## **Why It Doesn't Loop Forever**

You might worry about infinite recursion. However, because we only move to a strictly greater value, we can never return to a cell we've already visited in the current path. This effectively turns the matrix into a **Directed Acyclic Graph (DAG)**.

**Visual Proof:**
```
Path: 1 → 2 → 6 → ?

Can we go back to 1? No! (6 > 1, we need next > 6)
Can we go back to 2? No! (6 > 2, we need next > 6)
Can we go back to 6? No! (6 = 6, we need next > 6)

The path can only move "upward" in value!
```

---

## **Memoization vs No Memoization**

### **Without Memoization:**
```python
def dfs(r, c):
    longest = 1
    for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
        nr, nc = r + dr, c + dc
        if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
            longest = max(longest, 1 + dfs(nr, nc))
    return longest  # No caching!
```
**Time:** $O(2^{m \times n})$ in worst case - exponential!

### **With Memoization (Our Solution):**
```python
if (r, c) in memo:
    return memo[(r, c)]
# ... compute ...
memo[(r, c)] = longest
return longest
```
**Time:** $O(m \times n)$ - each cell computed once!

---

## **A Cool Alternative: Topological Sort (Kahn's Algorithm)**

Because this problem is technically finding the longest path in a DAG, you could actually solve it without DFS.

### **The Approach:**

1. **Calculate the out-degree** of every cell (how many neighbors are larger).
2. **Put all cells with out-degree == 0** (the "peaks") into a queue.
3. **Perform a BFS level-by-level**, "peeling" the matrix like an onion from the largest values down to the smallest.
4. **The number of levels** you peel is your answer.

### **Topological Sort Solution:**
```python
def longestIncreasingPath(self, matrix):
    if not matrix:
        return 0
    
    rows, cols = len(matrix), len(matrix[0])
    out_degree = [[0] * cols for _ in range(rows)]
    
    # Calculate out-degrees
    for r in range(rows):
        for c in range(cols):
            for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                    out_degree[r][c] += 1
    
    # Start with cells that have no outgoing edges (local peaks)
    queue = deque()
    for r in range(rows):
        for c in range(cols):
            if out_degree[r][c] == 0:
                queue.append((r, c))
    
    # BFS level by level
    levels = 0
    while queue:
        levels += 1
        for _ in range(len(queue)):
            r, c = queue.popleft()
            # Reduce out-degree of neighbors
            for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] < matrix[r][c]:
                    out_degree[nr][nc] -= 1
                    if out_degree[nr][nc] == 0:
                        queue.append((nr, nc))
    
    return levels
```

---

## **Complexity Analysis**

### **DFS + Memoization (Main Solution)**

**Time Complexity:** $O(m \times n)$  
- We visit each cell at most once (due to memoization)
- Each cell does $O(1)$ work (checking 4 neighbors)
- **Total:** $O(m \times n)$

**Space Complexity:** $O(m \times n)$  
- Memoization table: $O(m \times n)$
- Recursion stack: $O(m \times n)$ in worst case (entire matrix is one long path)

### **Topological Sort Alternative**

**Time Complexity:** $O(m \times n)$  
Same as DFS approach.

**Space Complexity:** $O(m \times n)$  
Out-degree matrix and queue.