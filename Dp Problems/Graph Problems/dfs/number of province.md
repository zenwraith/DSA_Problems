# Number of Provinces

**LeetCode Link:** [https://leetcode.com/problems/number-of-provinces/](https://leetcode.com/problems/number-of-provinces/)

---

## Problem Statement

There are `n` cities. Some of them are connected, while some are not. If city `a` is connected directly with city `b`, and city `b` is connected directly with city `c`, then city `a` is connected indirectly with city `c`.

A **province** is a group of directly or indirectly connected cities and no other cities outside of the group.

You are given an `n x n` matrix `isConnected` where `isConnected[i][j] = 1` if the `iᵗʰ` city and the `jᵗʰ` city are directly connected, and `isConnected[i][j] = 0` otherwise.

Return the **total number of provinces**.

**Example 1:**
```
Input: isConnected = [[1,1,0],
                      [1,1,0],
                      [0,0,1]]
Output: 2

Explanation:
Cities 0 and 1 are connected (province 1)
City 2 is alone (province 2)
```

**Example 2:**
```
Input: isConnected = [[1,0,0],
                      [0,1,0],
                      [0,0,1]]
Output: 3

Explanation:
Each city is its own province.
```

**Example 3:**
```
Input: isConnected = [[1,1,1],
                      [1,1,1],
                      [1,1,1]]
Output: 1

Explanation:
All cities are connected - single province.
```

**Constraints:**
- $1 \leq n \leq 200$
- $n$ == `isConnected.length`
- $n$ == `isConnected[i].length`
- `isConnected[i][j]` is `1` or `0`
- `isConnected[i][i]` == `1` (a city is always connected to itself)
- `isConnected[i][j]` == `isConnected[j][i]` (symmetric matrix)

---

## Problem Analysis

### What We're Finding

This is a **connected components** problem:
- Each province = one connected component
- Cities = nodes in a graph
- Direct connections = edges
- Need to count number of separate components

### Graph Representation

The adjacency matrix is given directly!
```
isConnected = [[1,1,0],
               [1,1,0],
               [0,0,1]]

Graph:  0 ←→ 1
        
        2 (isolated)
```

---

## Solution 1: Depth-First Search (DFS)

### Strategy

Visit each unvisited city and explore its entire province using DFS:
1. Iterate through all cities
2. If city is unvisited, increment province count and DFS
3. DFS marks all cities in same province as visited

### Algorithm Steps

1. **Initialize:** Visited set and province counter
2. **Iterate cities:** For each city `i`:
   - If not visited:
     - Found new province! Increment counter
     - DFS to mark all cities in this province
3. **DFS Function:**
   - Mark current city as visited
   - Recursively visit all connected unvisited cities

### Detailed Walkthrough

**Example:** `isConnected = [[1,1,0],[1,1,0],[0,0,1]]`

#### Initialization
```
visited = set()
provinces = 0
```

#### Process City 0
```
City 0 not visited → Found new province!
provinces = 1
visited = {0}

DFS(0):
  Check neighbors: [0, 1, 2]
  - isConnected[0][0] = 1, but 0 already visited
  - isConnected[0][1] = 1, and 1 not visited ✓
    visited = {0, 1}
    DFS(1):
      Check neighbors: [0, 1, 2]
      - isConnected[1][0] = 1, but 0 visited
      - isConnected[1][1] = 1, but 1 visited
      - isConnected[1][2] = 0 (not connected)
  - isConnected[0][2] = 0 (not connected)
```

#### Process City 1
```
City 1 already visited ✓ skip
```

#### Process City 2
```
City 2 not visited → Found new province!
provinces = 2
visited = {0, 1, 2}

DFS(2):
  Check neighbors: [0, 1, 2]
  - isConnected[2][0] = 0 (not connected)
  - isConnected[2][1] = 0 (not connected)
  - isConnected[2][2] = 1, but 2 visited
```

#### Result
```
provinces = 2 ✓
```

### Implementation

```python
class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:

        n = len(isConnected)
        visited = set()
        provinces = 0

        def dfs(city):
            for neighbor, connected in enumerate(isConnected[city]):
                if connected == 1 and neighbor not in visited:
                    visited.add(neighbor)
                    dfs(neighbor)
        
        for i in range(n):
            if i not in visited:

                provinces += 1
                visited.add(i)
                dfs(i)

        return provinces

```

### Code Explanation

**Lines 4-6:** Initialize visited set and province counter

**Lines 8-12:** DFS helper function
- Enumerate through adjacency list for current city
- If connected (1) and unvisited, add to visited and recurse

**Lines 14-19:** Main loop
- For each city, if unvisited:
  - Found new province, increment counter
  - Mark as visited and DFS to mark entire province

### Complexity Analysis

**Time:** $O(n^2)$
- Outer loop: $O(n)$
- DFS visits each city once: $O(n)$
- Each DFS checks all n neighbors: $O(n)$
- Total: $O(n^2)$

**Space:** $O(n)$
- Visited set: $O(n)$
- Recursion stack: $O(n)$ worst case
- Total: $O(n)$

---

## Solution 2: Union-Find (Disjoint Set Union)

### Strategy

Union-Find is **perfect** for connected components!
- Initially: Each city is its own province (parent = self)
- For each connection: Union the two cities
- Result: Count number of unique roots (provinces)

### Key Concepts

**Parent Array:** `parent[i]` = parent of city `i`
- Initially: `parent[i] = i` (each city is its own parent)
- After unions: Cities in same province share same root

**Find:** Find the root/representative of a city
**Union:** Merge two provinces into one

### Algorithm Steps

1. **Initialize:** Each city points to itself, `provinces = n`
2. **Process edges:** For each connection `isConnected[i][j] = 1`:
   - Find roots of both cities
   - If different roots: union them, decrement provinces
3. **Return:** Final province count

### Detailed Walkthrough

**Example:** `isConnected = [[1,1,0],[1,1,0],[0,0,1]]`

#### Initialization
```
parent = [0, 1, 2]  (each city is its own parent)
provinces = 3
```

#### Process Connections
```
Check (0,0): same city, skip
Check (0,1): isConnected[0][1] = 1
  find(0) = 0, find(1) = 1
  Different roots! Union:
    parent[0] = 1  (merge 0 into 1's province)
    provinces = 2
    
  parent = [1, 1, 2]

Check (0,2): isConnected[0][2] = 0, skip

Check (1,0): already processed (symmetric)
Check (1,1): same city, skip
Check (1,2): isConnected[1][2] = 0, skip

Check (2,0): isConnected[2][0] = 0, skip
Check (2,1): isConnected[2][1] = 0, skip
Check (2,2): same city, skip
```

#### Result
```
provinces = 2 ✓

Final parent structure:
    1 (root)
   / 
  0

  2 (root)
```

### Implementation

```python

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:

        n = len(isConnected)
        parent = [ i for i in range(n)]
        self.provinces = n

        def find(i):
            if parent[i] == i:
                return i

            return find(parent[i])

        def union(i,j):
            root_i = find(i)
            root_j = find(j)

            if root_i != root_j:
                parent[root_i] = root_j
                self.provinces -=1

        for i in range(n):
            for j in range(i+1,n):
                if isConnected[i][j] == 1:
                    union(i,j)

        return self.provinces 

```

### Code Explanation

**Lines 5-6:** Initialize parent array and province count

**Lines 8-12:** Find operation (without path compression)
- Base case: if node is its own parent, it's a root
- Recursive case: find parent's root

**Lines 14-20:** Union operation
- Find roots of both cities
- If different, merge by setting one's parent to other
- Decrement province count (two become one)

**Lines 22-25:** Process all edges
- Only check upper triangle (j > i) since matrix is symmetric
- Union connected cities

### Optimizations

**Path Compression:** Make tree flatter
```python
def find(i):
    if parent[i] != i:
        parent[i] = find(parent[i])  # Path compression
    return parent[i]
```

**Union by Rank:** Attach smaller tree to larger
```python
rank = [0] * n

def union(i, j):
    root_i, root_j = find(i), find(j)
    if root_i != root_j:
        if rank[root_i] < rank[root_j]:
            parent[root_i] = root_j
        elif rank[root_i] > rank[root_j]:
            parent[root_j] = root_i
        else:
            parent[root_j] = root_i
            rank[root_i] += 1
        self.provinces -= 1
```

### Complexity Analysis

**Time:** $O(n^2 \cdot \alpha(n))$
- Process $n^2$ edges
- Each union/find: $O(\alpha(n))$ amortized (nearly constant)
- With optimizations: nearly $O(n^2)$

**Space:** $O(n)$
- Parent array: $O(n)$
- Rank array (if used): $O(n)$

---

## Solution 3: Breadth-First Search (BFS)

### Strategy

Similar to DFS but uses queue instead of recursion:
1. For each unvisited city, start BFS
2. BFS explores entire province level by level
3. Count how many times we start a new BFS

### Algorithm Steps

1. **Initialize:** Visited set and province counter
2. **Iterate cities:** For each unvisited city:
   - Increment province count
   - BFS to mark entire province
3. **BFS:** Use queue to explore all connected cities

### Detailed Walkthrough

**Example:** `isConnected = [[1,1,0],[1,1,0],[0,0,1]]`

#### Process City 0
```
City 0 not visited → New province!
provinces = 1
queue = deque([0])
visited = {0}

BFS:
  Dequeue 0:
    Check neighbors: isConnected[0] = [1,1,0]
    - Neighbor 0: visited
    - Neighbor 1: connected (1) and unvisited ✓
      Add to queue, mark visited
      queue = deque([1]), visited = {0,1}
    - Neighbor 2: not connected (0)
  
  Dequeue 1:
    Check neighbors: isConnected[1] = [1,1,0]
    - Neighbor 0: visited
    - Neighbor 1: visited
    - Neighbor 2: not connected (0)
  
  Queue empty, province fully explored
```

#### Process City 1
```
City 1 already visited, skip
```

#### Process City 2
```
City 2 not visited → New province!
provinces = 2
queue = deque([2])
visited = {0,1,2}

BFS:
  Dequeue 2:
    Check neighbors: isConnected[2] = [0,0,1]
    - Neighbor 0: not connected
    - Neighbor 1: not connected
    - Neighbor 2: visited
  
  Queue empty
```

#### Result
```
provinces = 2 ✓
```

### Implementation

```python
from collections import deque

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:
        n = len(isConnected)
        visited = set()
        provinces = 0
        
        for i in range(n):
            if i not in visited:
                # We found a new province!
                provinces += 1
                # Start BFS to mark all cities in this province
                queue = deque([i])
                visited.add(i)
                
                while queue:
                    curr_city = queue.popleft()
                    # Check all possible neighbors for curr_city
                    for neighbor, connected in enumerate(isConnected[curr_city]):
                        if connected == 1 and neighbor not in visited:
                            visited.add(neighbor)
                            queue.append(neighbor)
                            
        return provinces
```

### Code Explanation

**Lines 5-7:** Initialize structures

**Lines 9-23:** Main loop
- For each unvisited city:
  - Increment province counter (found new province)
  - Initialize queue with current city
  - BFS to explore entire province:
    - Dequeue city
    - Check all neighbors
    - If connected and unvisited, add to queue and visited

### Complexity Analysis

**Time:** $O(n^2)$
- Outer loop: $O(n)$
- BFS visits each city once: $O(n)$
- Each city checks all neighbors: $O(n)$
- Total: $O(n^2)$

**Space:** $O(n)$
- Visited set: $O(n)$
- Queue: $O(n)$ worst case
- Total: $O(n)$

---

## Comprehensive Comparison

### Approach Comparison Table

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| **DFS** | $O(n^2)$ | $O(n)$ | Simple, intuitive, less code | Recursion stack (stack overflow risk) |
| **Union-Find** | $O(n^2 \cdot \alpha(n))$ | $O(n)$ | Great for dynamic connectivity | More code, slightly slower without opts |
| **BFS** | $O(n^2)$ | $O(n)$ | No recursion, level-order | More code than DFS, same complexity |

### When to Use Each

**Use DFS when:**
- Simple counting of components
- Code brevity important
- Graph not too large (no stack overflow)

**Use Union-Find when:**
- Need to handle dynamic edge additions
- Multiple queries about connectivity
- Want to avoid recursion

**Use BFS when:**
- Need level-by-level exploration
- Want to avoid recursion
- Finding shortest paths (though not needed here)

---

## Visual Representation

```
Example: [[1,1,0,0],
          [1,1,0,0],
          [0,0,1,1],
          [0,0,1,1]]

Graph Representation:
Province 1:  0 ←→ 1
Province 2:  2 ←→ 3

DFS Traversal:
Start 0 → Visit 1 → Done (province 1)
Start 2 → Visit 3 → Done (province 2)
Result: 2 provinces

Union-Find Evolution:
Initial:  [0,1,2,3]
Union(0,1): [1,1,2,3] provinces=3
Union(2,3): [1,1,3,3] provinces=2
Result: 2 provinces

BFS Traversal:
Start 0:
  Level 0: {0}
  Level 1: {1}
  Done
Start 2:
  Level 0: {2}
  Level 1: {3}
  Done
Result: 2 provinces
```

---

## Edge Cases

1. **Single city:** `[[1]]` → 1 province
2. **All connected:** `[[1,1],[1,1]]` → 1 province
3. **All isolated:** `[[1,0],[0,1]]` → 2 provinces
4. **Chain:** Cities connected in sequence → 1 province
5. **Star pattern:** One city connected to all → 1 province
6. **Empty connections (except diagonal):** $n$ provinces

---

## Common Pitfalls

1. **Not marking start city as visited:** Can cause double counting
2. **Processing full matrix:** Only need upper triangle (symmetric)
3. **Union-Find province counting:** Must decrement on successful union only
4. **Off-by-one in initialization:** Make sure `provinces = n` initially
5. **Forgetting self-connections:** Diagonal is always 1 (not meaningful)

---

## Pattern Recognition

This problem demonstrates the **Connected Components** pattern:
- Core technique appears in:
  - Network connectivity
  - Social network friend circles
  - Region analysis in grids
  - Graph component counting

### Related Problems

- **547. Number of Provinces** (this problem)
- **200. Number of Islands** (2D grid version)
- **323. Number of Connected Components in Undirected Graph** (premium)
- **684. Redundant Connection** (Union-Find to detect cycles)
- **261. Graph Valid Tree** (Union-Find for tree validation)
- **990. Satisfiability of Equality Equations** (Union-Find)

---

## Key Takeaways

1. **Three valid approaches** - choose based on requirements
2. **DFS is simplest** for one-time component counting
3. **Union-Find shines** for dynamic connectivity queries
4. **BFS alternative** when avoiding recursion
5. **Adjacency matrix** given directly - no need to build graph
6. **Symmetric matrix** - optimization opportunity

---
