# Course Schedule

**LeetCode Link:** [https://leetcode.com/problems/course-schedule/](https://leetcode.com/problems/course-schedule/)

---

## Problem Statement

There are a total of `numCourses` courses you have to take, labeled from `0` to `numCourses - 1`. You are given an array `prerequisites` where `prerequisites[i] = [aᵢ, bᵢ]` indicates that you **must take course `bᵢ` first** if you want to take course `aᵢ`.

- For example, the pair `[0, 1]` indicates that to take course `0` you have to first take course `1`.

Return `true` if you can finish all courses. Otherwise, return `false`.

**Example 1:**
```
Input: numCourses = 2, prerequisites = [[1,0]]
Output: true
Explanation: There are a total of 2 courses to take. 
To take course 1 you should have finished course 0. So it is possible.
```

**Example 2:**
```
Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false
Explanation: There are a total of 2 courses to take. 
To take course 1 you should have finished course 0, and to take course 0 
you should also have finished course 1. So it is impossible.
```

**Example 3:**
```
Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: true
Explanation: 
Course 0 → no prerequisites
Course 1 → needs 0
Course 2 → needs 0
Course 3 → needs 1 and 2
Possible order: [0,1,2,3] or [0,2,1,3]
```

**Constraints:**
- $1 \leq \text{numCourses} \leq 2000$
- $0 \leq \text{prerequisites.length} \leq 5000$
- `prerequisites[i].length` == `2`
- $0 \leq a_i, b_i < \text{numCourses}$
- All pairs `prerequisites[i]` are **unique**

---

## Strategy: Cycle Detection in Directed Graph

### The Key Insight

This is a **cycle detection problem** in a directed graph (DAG validation):

- **Courses** = Nodes
- **Prerequisites** = Directed edges
- **Can finish all courses** ↔ No cycles exist (DAG = Directed Acyclic Graph)

**Why?** If a cycle exists like `A → B → C → A`, then:
- To take A, you need B
- To take B, you need C
- To take C, you need A ← **Impossible!**

### Two Approaches

| Approach | Algorithm | Key Idea | Time | Space |
|----------|-----------|----------|------|-------|
| **Kahn's Algorithm** | BFS (Topological Sort) | Process nodes with no dependencies | $O(V+E)$ | $O(V+E)$ |
| **DFS Cycle Detection** | DFS with path tracking | Detect back edges | $O(V+E)$ | $O(V+E)$ |

Both are valid and have the same complexity. Choose based on preference.

---

## Solution 1: BFS - Kahn's Algorithm (Topological Sort)

### Algorithm Explanation

**Kahn's Algorithm** processes nodes in topological order by:
1. **Start with nodes that have no dependencies** (in-degree = 0)
2. **Remove each node and its edges** from the graph
3. **Repeat** until all nodes processed or stuck (cycle detected)

**Key Concept - In-Degree:**
- **In-degree** = number of incoming edges (prerequisites)
- **In-degree 0** = no prerequisites, can take now!
- After taking a course, reduce in-degree of dependent courses

### Step-by-Step Process

1. **Build graph:**
   - Adjacency list: `adj[pre] = [courses that need pre]`
   - In-degree array: `in_degree[course] = # of prerequisites`

2. **Initialize queue:**
   - Add all courses with `in_degree == 0` (no prerequisites)

3. **BFS loop:**
   - Take a course from queue
   - Reduce in-degree of all dependent courses
   - If a course's in-degree becomes 0, add to queue

4. **Check completion:**
   - If processed all courses → True
   - Otherwise → cycle exists → False

```python
from collections import deque

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        # 1. Build Adjacency List and In-degree array
        adj = [[] for _ in range(numCourses)]
        in_degree = [0] * numCourses
        
        for course, pre in prerequisites:
            adj[pre].append(course)  # pre → course
            in_degree[course] += 1    # course needs one more prerequisite
        
        # 2. Add all courses with NO prerequisites to the queue
        queue = deque([i for i in range(numCourses) if in_degree[i] == 0])
        
        taken_courses = 0
        while queue:
            # Take a course (remove from graph)
            pre = queue.popleft()
            taken_courses += 1
            
            # 3. For every course that depends on this prerequisite
            for course in adj[pre]:
                in_degree[course] -= 1  # One prerequisite satisfied
                # If all prerequisites are met, add to queue
                if in_degree[course] == 0:
                    queue.append(course)
        
        # If we were able to take all courses, return True
        return taken_courses == numCourses
```

---

## Solution 2: DFS - Cycle Detection

### Algorithm Explanation

**DFS Cycle Detection** uses two sets:
- **`visited`**: Nodes completely processed (and their subtrees are safe)
- **`path`**: Nodes in the current DFS path (recursion stack)

**Key Idea:**
- If we encounter a node **already in current path** → **cycle detected!**
- If we encounter a node **in visited** → safe, already verified

### Three States

| State | Meaning | Action |
|-------|---------|--------|
| Not visited | Not explored yet | Start DFS |
| In path | Currently exploring | **Cycle!** (back edge) |
| Visited | Fully explored | Skip (safe) |

### Step-by-Step Process

1. **Build adjacency list:** `adj[pre] = [courses that need pre]`
2. **For each course:** Run DFS to check for cycles
3. **DFS logic:**
   - If in current path → cycle found
   - If visited → safe, return false
   - Add to path, explore neighbors
   - Remove from path (backtrack), mark visited

```python
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        # Build adjacency list
        adj = [[] for i in range(numCourses)]
        
        for course, pre in prerequisites:
            adj[pre].append(course)  # pre → course
        
        visited = set()  # Completely processed nodes
        path = set()     # Current DFS path (recursion stack)
        
        def has_cycle(course):
            if course in path:
                return True  # Cycle detected! (back edge)
            if course in visited:
                return False  # Already verified, safe
            
            # Add to current path
            path.add(course)
            
            # Check all neighbors (dependent courses)
            for neighbor in adj[course]:
                if has_cycle(neighbor):
                    return True
            
            # Backtrack: remove from path, mark as visited
            path.remove(course)
            visited.add(course)
            return False
        
        # Check each course (graph may be disconnected)
        for i in range(numCourses):
            if has_cycle(i):
                return False
        
        return True
```

---

## Detailed Walkthrough

### Example 1: Valid Course Schedule

**Input:** `numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]`

**Graph:**
```
0 → 1 → 3
↓   ↗
2 ──
```

#### BFS Approach (Kahn's Algorithm)

| Step | Queue | Taken | In-degree | Action |
|------|-------|-------|-----------|--------|
| Init | [0] | 0 | [0,1,1,2] | Course 0 has no prereqs |
| 1 | [1,2] | 1 | [0,0,0,2] | Take 0, update 1 and 2 |
| 2 | [2] | 2 | [0,0,0,1] | Take 1, update 3 |
| 3 | [3] | 3 | [0,0,0,0] | Take 2, update 3 |
| 4 | [] | 4 | [0,0,0,0] | Take 3, done! |

**Result:** Taken 4 courses = numCourses → **True**

#### DFS Approach

```
Start from 0:
  path = {0}
  Check 1:
    path = {0,1}
    Check 3:
      path = {0,1,3}
      No neighbors
      visited = {3}, path = {0,1}
    visited = {1,3}, path = {0}
  Check 2:
    path = {0,2}
    Check 3:
      Already in visited ✓
    visited = {1,2,3}, path = {0}
  visited = {0,1,2,3}, path = {}

No cycles found → True
```

---

### Example 2: Invalid Course Schedule (Cycle)

**Input:** `numCourses = 2, prerequisites = [[1,0],[0,1]]`

**Graph:**
```
0 ⇄ 1  (cycle!)
```

#### BFS Approach

| Step | Queue | Taken | In-degree | Action |
|------|-------|-------|-----------|--------|
| Init | [] | 0 | [1,1] | No course with in-degree 0! |

**Result:** Taken 0 courses ≠ 2 → **False**

**Why:** Both courses depend on each other, so neither can be taken first.

#### DFS Approach

```
Start from 0:
  path = {0}
  Check 1:
    path = {0,1}
    Check 0:
      0 is in path! ← CYCLE DETECTED
      Return True
```

**Result:** Cycle found → **False**

---

## Visual Comparison: BFS vs DFS

### BFS (Kahn's Algorithm)

**Think:** "Peel the onion layer by layer"

```
Level 0: [courses with no prereqs] → Take them
Level 1: [courses now available]  → Take them
Level 2: [courses now available]  → Take them
...

If stuck with courses remaining → cycle!
```

### DFS (Cycle Detection)

**Think:** "Follow the chain until you find a loop"

```
Follow path: A → B → C → D
            ↑_____________↓  ← Loop! (A in path)
            
Or complete without revisiting → safe
```

---

## Why Two Sets in DFS?

### Without `visited` (only `path`)

❌ **Problem:** Revisit safe nodes repeatedly

```
Graph:
  0 → 1 → 2
  ↓
  3 → 2

Without visited:
  DFS(0) → DFS(1) → DFS(2) ✓
        → DFS(3) → DFS(2) ← Explore 2 AGAIN!
```

**Inefficiency:** Reprocess node 2 and its subtree

### With `visited`

✅ **Optimization:** Skip already-verified nodes

```
DFS(0) → DFS(1) → DFS(2) ✓ [mark 2 visited]
      → DFS(3) → DFS(2) ← Check: in visited? Yes! Skip.
```

**Result:** Each node processed once → $O(V+E)$ instead of exponential

---

## Edge Cases

### 1. No Prerequisites
```
Input: numCourses = 3, prerequisites = []
Output: true
Explanation: All courses independent
```

### 2. Linear Chain
```
Input: numCourses = 3, prerequisites = [[1,0],[2,1]]
Output: true
Graph: 0 → 1 → 2 (DAG, valid)
```

### 3. Self-Loop (Invalid by constraints)
```
Input: prerequisites = [[0,0]]
Would create: 0 → 0 (cycle)
But constraints ensure: aᵢ ≠ bᵢ
```

### 4. Multiple Components
```
Input: numCourses = 4, prerequisites = [[1,0],[3,2]]
Graph: 0 → 1    2 → 3 (two separate chains)
Output: true
```

### 5. Complex Cycle
```
Input: prerequisites = [[1,0],[2,1],[0,2]]
Graph: 0 → 1 → 2 → 0 (cycle)
Output: false
```

---

## Common Pitfalls

1. **Reversing edge direction:** `[a, b]` means `b → a` (b is prerequisite of a)
2. **Forgetting disconnected components:** Must check all nodes in DFS
3. **Not maintaining `path` in DFS:** Won't detect cycles correctly
4. **Using only `visited` in DFS:** Can't distinguish between "in current path" and "already processed"
5. **Incorrect in-degree calculation:** Must count incoming edges, not outgoing

---

## BFS vs DFS: When to Use?

| Scenario | Prefer BFS | Prefer DFS |
|----------|------------|------------|
| Need actual order | ✓ (gives topological sort) | ✗ (only detects cycle) |
| Iterative preferred | ✓ | ✗ |
| Recursive preferred | ✗ | ✓ |
| Stack overflow risk | ✓ (uses queue) | ⚠ (deep recursion) |
| Code simplicity | Medium | Slightly more complex |

**For Course Schedule:** BFS is slightly preferred if you need the actual course order (Course Schedule II). For just detecting feasibility, both are equal.

---

## Complexity Analysis

### BFS (Kahn's Algorithm)

**Time Complexity:** $O(V + E)$
- Build graph: $O(E)$ for edges
- BFS: Visit each vertex once $O(V)$, check each edge once $O(E)$
- Total: $O(V + E)$

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$
- In-degree array: $O(V)$
- Queue: $O(V)$
- Total: $O(V + E)$

### DFS (Cycle Detection)

**Time Complexity:** $O(V + E)$
- Build graph: $O(E)$
- DFS: Visit each vertex once $O(V)$, check each edge once $O(E)$
- Total: $O(V + E)$

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$
- Visited set: $O(V)$
- Path set: $O(V)$
- Recursion stack: $O(V)$ in worst case
- Total: $O(V + E)$

---

## Related Problems

- **207. Course Schedule** (this problem)
- **210. Course Schedule II** (return the order, not just true/false)
- **785. Is Graph Bipartite?** (different graph property check)
- **802. Find Eventual Safe States** (cycle detection variant)
- **269. Alien Dictionary** (topological sort application)
- **310. Minimum Height Trees** (tree/graph problem)

---

## Key Takeaways

1. **Course Schedule = Cycle Detection** in directed graph
2. **Two valid approaches:** BFS (Kahn's) or DFS (path tracking)
3. **BFS:** Uses in-degree and queue, gives topological order
4. **DFS:** Uses two sets (visited + path) to detect back edges
5. **Both $O(V+E)$ time and space**
6. **Real-world applications:** Task scheduling, build systems, dependency resolution
