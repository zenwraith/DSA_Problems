# Is Graph Bipartite?

**LeetCode Link:** [https://leetcode.com/problems/is-graph-bipartite/](https://leetcode.com/problems/is-graph-bipartite/)

---

## Problem Statement

There is an **undirected** graph with `n` nodes, where each node is numbered between `0` and `n - 1`. You are given a 2D array `graph`, where `graph[u]` is an array of nodes that node `u` is adjacent to.

A graph is **bipartite** if the nodes can be partitioned into two independent sets `A` and `B` such that **every edge** in the graph connects a node in set `A` and a node in set `B`.

Return `true` if and only if the graph is bipartite.

**Example 1:**
```
Input: graph = [[1,2,3],[0,2],[0,1,3],[0,2]]
Output: false

Explanation:
Cannot partition the nodes into two independent sets such that 
every edge connects nodes from different sets.
```

**Example 2:**
```
Input: graph = [[1,3],[0,2],[1,3],[0,2]]
Output: true

Explanation:
We can partition the nodes into sets {0, 2} and {1, 3}.
Every edge connects a node from one set to a node in the other set.
```

**Example 3:**
```
Input: graph = [[1],[0,3],[3],[1,2]]
Output: true

Explanation:
Partition: {0, 3} and {1, 2}
```

**Constraints:**
- `graph.length` == `n`
- $1 \leq n \leq 100$
- $0 \leq \text{graph[u].length} < n$
- $0 \leq \text{graph[u][i]} \leq n - 1$
- `graph[u]` does not contain `u`
- All values in `graph[u]` are **unique**
- If `graph[u]` contains `v`, then `graph[v]` contains `u` (undirected graph)

---

## Strategy: Graph Coloring (2-Coloring)

### The Key Insight

A graph is bipartite **if and only if** it can be 2-colored (colored with 2 colors such that no adjacent nodes have the same color).

**Why?**
- The two colors represent the two sets `A` and `B`
- If two adjacent nodes have the same color → they're in the same set → violates bipartite definition
- If we can successfully 2-color → all edges connect different colors → bipartite!

### Bipartite Detection = Odd Cycle Detection

**Key Theorem:** A graph is bipartite if and only if it contains **no odd-length cycles**.

**Intuition:**
- **Even cycle:** Can alternate colors: `0 → 1 → 0 → 1 → ... → 0`
- **Odd cycle:** Cannot alternate: `0 → 1 → 0 → 1 → 0 → ?` (needs to be 1 to close, but neighbor is also 1!)

### Algorithm Overview

**Both BFS and DFS work:**

1. **Initialize:** Mark all nodes as uncolored (`-1`)
2. **For each component:** (graph may be disconnected)
   - Pick an uncolored node and assign it color `0`
   - Use BFS/DFS to color all reachable nodes
3. **Coloring Rules:**
   - Uncolored neighbor → assign opposite color
   - Already colored neighbor → check if color is different
   - Same color as current → **not bipartite!**
4. **Return:** True if all nodes successfully colored

---

## Solution 1: BFS Approach

```python
from collections import deque

class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        n = len(graph)
        # -1: uncolored, 0: color A, 1: color B
        colors = [-1] * n
        
        # Check each component (graph may be disconnected)
        for i in range(n):
            # If already colored, skip (already processed)
            if colors[i] != -1:
                continue
            
            # Start BFS for this component
            queue = deque([i])
            colors[i] = 0  # Start with color 0
            
            while queue:
                node = queue.popleft()
                
                for neighbor in graph[node]:
                    # If neighbor isn't colored, give it the opposite color
                    if colors[neighbor] == -1:
                        colors[neighbor] = 1 - colors[node]
                        queue.append(neighbor)
                    # If neighbor has the SAME color, it's not bipartite!
                    elif colors[neighbor] == colors[node]:
                        return False
        
        return True
```

---

## Solution 2: DFS Approach

```python
class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        n = len(graph)
        colors = [-1] * n  # -1: uncolored, 0: Color A, 1: Color B
        
        def dfs(node, color_to_paint):
            # 1. Paint the current node
            colors[node] = color_to_paint
            
            # 2. Check all neighbors
            for neighbor in graph[node]:
                # If neighbor is not colored, color it with the OPPOSITE (1 - color)
                if colors[neighbor] == -1:
                    if not dfs(neighbor, 1 - color_to_paint):
                        return False
                # If neighbor is already colored, it MUST be different from current
                elif colors[neighbor] == colors[node]:
                    return False
            
            return True
        
        # Check every component (in case the graph is disconnected)
        for i in range(n):
            if colors[i] == -1:
                if not dfs(i, 0):
                    return False
        
        return True
```

---

## Detailed Walkthrough

### Example: Not Bipartite

**Input:** `graph = [[1,2,3],[0,2],[0,1,3],[0,2]]`

```
Graph Structure:
    0 ―― 1
    |\ /|
    | X |
    |/ \|
    3 ―― 2
```

**BFS Execution:**

| Step | Node | Color | Neighbor | Action | Result |
|------|------|-------|----------|--------|--------|
| 1 | 0 | 0 | - | Start BFS | colors = [0,-1,-1,-1] |
| 2 | 0 | 0 | 1 | Color 1 with 1 | colors = [0,1,-1,-1] |
| 3 | 0 | 0 | 2 | Color 2 with 1 | colors = [0,1,1,-1] |
| 4 | 0 | 0 | 3 | Color 3 with 1 | colors = [0,1,1,1] |
| 5 | 1 | 1 | 0 | Already colored 0 ≠ 1 ✓ | Continue |
| 6 | 1 | 1 | 2 | Already colored 1 = 1 ✗ | **Return False** |

**Why it failed:** Nodes 1 and 2 are neighbors and both have color 1!

This forms an **odd cycle**: `0 → 1 → 2 → 0` (length 3)

---

### Example: Bipartite Graph

**Input:** `graph = [[1,3],[0,2],[1,3],[0,2]]`

```
Graph Structure:
    0 ―― 1
    |    |
    |    |
    3 ―― 2
```

**BFS Execution:**

| Step | Node | Color | Neighbor | Action | colors |
|------|------|-------|----------|--------|--------|
| 1 | 0 | 0 | - | Start | [0,-1,-1,-1] |
| 2 | 0 | 0 | 1 | Color 1 with 1 | [0,1,-1,-1] |
| 3 | 0 | 0 | 3 | Color 3 with 1 | [0,1,-1,1] |
| 4 | 1 | 1 | 0 | Check: 0 ≠ 1 ✓ | Continue |
| 5 | 1 | 1 | 2 | Color 2 with 0 | [0,1,0,1] |
| 6 | 3 | 1 | 0 | Check: 0 ≠ 1 ✓ | Continue |
| 7 | 3 | 1 | 2 | Check: 0 ≠ 1 ✓ | Continue |
| 8 | 2 | 0 | 1 | Check: 1 ≠ 0 ✓ | Continue |
| 9 | 2 | 0 | 3 | Check: 1 ≠ 0 ✓ | Continue |

**Result:** `True` - Successfully 2-colored!

**Partition:**
- Set A (color 0): {0, 2}
- Set B (color 1): {1, 3}

---

## Handling Disconnected Graphs

**Example:** `graph = [[1],[0],[3],[2]]`

```
Component 1:  0 ―― 1
Component 2:  2 ―― 3
```

**Code Handles This:**
```python
for i in range(n):
    if colors[i] == -1:  # New component
        # Start BFS/DFS from this node
```

Each component is colored independently!

---

## Why Check All Components?

**Graph may be disconnected:**

```python
graph = [[],[],[]]  # No edges
```

Each node is its own component. We must check each:
- Component 1: {0}
- Component 2: {1}
- Component 3: {2}

All are trivially bipartite.

---

## BFS vs DFS Comparison

| Aspect | BFS | DFS |
|--------|-----|-----|
| **Data Structure** | Queue (deque) | Recursion stack |
| **Space** | $O(n)$ queue | $O(n)$ recursion |
| **Intuition** | Level-by-level coloring | Depth-first coloring |
| **Code Length** | Slightly longer | More concise |
| **Performance** | Same $O(V + E)$ | Same $O(V + E)$ |
| **Preference** | Iterative preference | Recursive preference |

**Both are equally valid!** Choose based on preference.

---

## Edge Cases

### 1. Single Node
```
graph = [[]]
Output: true
Explanation: One node is trivially bipartite
```

### 2. Two Nodes, No Edge
```
graph = [[],[]]
Output: true
Explanation: Two disconnected components
```

### 3. Self-Loop (Invalid by constraints)
```
graph = [[0]]
Would be: false (odd cycle of length 1)
But constraints guarantee: graph[u] does not contain u
```

### 4. Triangle (Odd Cycle)
```
graph = [[1,2],[0,2],[0,1]]
Output: false
Explanation: Odd cycle of length 3
```

### 5. Square (Even Cycle)
```
graph = [[1,3],[0,2],[1,3],[0,2]]
Output: true
Explanation: Even cycle can be 2-colored
```

---

## Common Pitfalls

1. **Forgetting disconnected components:** Must loop through all nodes
2. **Not checking already colored neighbors:** Must verify color compatibility
3. **Using boolean instead of 3 states:** Need uncolored state (-1)
4. **Assuming graph is connected:** May have multiple components
5. **Off-by-one in color flipping:** Use `1 - color` not `color + 1`

---

## Visual Representation

### Bipartite Graph (Even Cycle)
```
    0 ―― 1
    |    |
    3 ―― 2

Coloring:
    [0] ―― [1]
     |      |
    [1] ―― [0]

Sets: {0,2} and {1,3} ✓
```

### Non-Bipartite Graph (Odd Cycle)
```
    0 ―― 1
     \  /
      2

Coloring Attempt:
    [0] ―― [1]
      \  /
      [?] ← Cannot color! 
           Needs to be 1 (for edge to 0)
           AND 0 (for edge to 1) ✗
```

---

## Complexity Analysis

### BFS Approach
**Time Complexity:** $O(V + E)$
- Visit each vertex once: $O(V)$
- Check each edge once: $O(E)$
- Total: $O(V + E)$ where $V = n$, $E = \sum \text{graph[i].length}$

**Space Complexity:** $O(V)$
- Colors array: $O(V)$
- BFS queue: $O(V)$ in worst case
- Total: $O(V)$

### DFS Approach
**Time Complexity:** $O(V + E)$
- Same as BFS

**Space Complexity:** $O(V)$
- Colors array: $O(V)$
- Recursion stack: $O(V)$ in worst case (linear graph)
- Total: $O(V)$

---

## Related Problems

- **785. Is Graph Bipartite?** (this problem)
- **886. Possible Bipartition** (same concept, different setup)
- **207. Course Schedule** (cycle detection in directed graph)
- **261. Graph Valid Tree** (cycle + connectivity check)
- **323. Number of Connected Components** (component counting)

---

## Key Takeaways

1. **Bipartite ↔ 2-Colorable ↔ No Odd Cycles** (all equivalent)
2. **Use graph coloring** with 2 colors (or 3 states: uncolored, color A, color B)
3. **Check all components** for disconnected graphs
4. **BFS and DFS both work** equally well
5. **Detect conflict early:** Return false as soon as same-colored neighbors found

---

## Problem Variations

Often, the interviewer won't ask "Is this bipartite?" directly. Instead, they might phrase it like:

**"You have a group of people and some pairs dislike each other. Can you split them into two teams such that no two people on the same team dislike each other?"**

This is **exactly the same problem!** 
- **Dislike** = Edge
- **Two Teams** = Bipartite partitions

**Other disguises:**
- "Can you divide into two groups with no internal conflicts?"
- "Is it possible to assign two colors such that no adjacent nodes share a color?"
- "Can you separate into two independent sets?"

**Recognition pattern:** Anytime you need to partition into **exactly 2 groups** with a conflict/adjacency constraint → think **bipartite!**