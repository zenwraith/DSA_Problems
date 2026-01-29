# Course Schedule II

**LeetCode Link:** [https://leetcode.com/problems/course-schedule-ii/](https://leetcode.com/problems/course-schedule-ii/)

---

## Problem Statement

There are a total of `numCourses` courses you have to take, labeled from `0` to `numCourses - 1`. You are given an array `prerequisites` where `prerequisites[i] = [aᵢ, bᵢ]` indicates that you **must take course `bᵢ` first** if you want to take course `aᵢ`.

- For example, the pair `[0, 1]` indicates that to take course `0` you have to first take course `1`.

Return **the ordering of courses** you should take to finish all courses. If there are many valid answers, return **any** of them. If it is impossible to finish all courses, return **an empty array**.

**Example 1:**
```
Input: numCourses = 2, prerequisites = [[1,0]]
Output: [0,1]
Explanation: There are a total of 2 courses to take. 
To take course 1 you should have finished course 0. 
So the correct course order is [0,1].
```

**Example 2:**
```
Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: [0,2,1,3] or [0,1,2,3]
Explanation: There are a total of 4 courses to take.
To take course 3 you should have finished both courses 1 and 2.
Both courses 1 and 2 should be taken after course 0.
So one correct course order is [0,1,2,3]. Another correct ordering is [0,2,1,3].
```

**Example 3:**
```
Input: numCourses = 1, prerequisites = []
Output: [0]
Explanation: There is only one course, no prerequisites.
```

**Example 4:**
```
Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: []
Explanation: There's a cycle: 0 → 1 → 0, so it's impossible.
```

**Constraints:**
- $1 \leq \text{numCourses} \leq 2000$
- $0 \leq \text{prerequisites.length} \leq \text{numCourses} \times (\text{numCourses} - 1)$
- `prerequisites[i].length` == `2`
- $0 \leq a_i, b_i < \text{numCourses}$
- $a_i \neq b_i$
- All pairs `[aᵢ, bᵢ]` are **distinct**

---

## Strategy: Topological Sort (Kahn's Algorithm)

### The Key Insight

This is **Course Schedule I + Return the Order**:

- **Course Schedule I:** Can we finish all courses? (cycle detection)
- **Course Schedule II:** Return the actual order to take courses (topological sort)

**Topological Sort** produces a linear ordering of vertices such that for every directed edge `u → v`, `u` comes before `v` in the ordering.

**Why this works:**
- If no cycle exists → topological sort gives valid course order
- If cycle exists → impossible to complete, return `[]`

### What is Topological Sort?

**Definition:** A linear ordering of vertices in a DAG (Directed Acyclic Graph) where all dependencies are respected.

**Example:**
```
Graph:
  0 → 1 → 3
  ↓   ↗
  2 ──

Valid topological orders:
- [0, 1, 2, 3]
- [0, 2, 1, 3]

Invalid:
- [1, 0, 2, 3] ❌ (0 must come before 1)
- [0, 3, 1, 2] ❌ (1 must come before 3)
```

### Kahn's Algorithm Explained

**Kahn's Algorithm** is a BFS-based approach:

1. **Calculate in-degrees:** Count prerequisites for each course
2. **Start with zero in-degree nodes:** Courses with no prerequisites
3. **Process iteratively:**
   - Take a course with in-degree 0
   - Add it to the result order
   - Reduce in-degree of all dependent courses
   - When a course's in-degree becomes 0, add to queue
4. **Check completeness:**
   - If processed all courses → return order
   - Otherwise → cycle detected → return `[]`

---

## Solution: BFS - Kahn's Algorithm

```python
from collections import deque

class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        # 1. Build adjacency list and in-degree array
        adj = [[] for _ in range(numCourses)]
        in_degree = [0] * numCourses
        
        for course, pre in prerequisites:
            adj[pre].append(course)  # pre → course
            in_degree[course] += 1   # course has one more prerequisite
        
        # 2. Add all courses with NO prerequisites to the queue
        queue = deque([i for i in range(numCourses) if in_degree[i] == 0])
        
        order = []  # This stores the topological order
        
        while queue:
            # Take a course (it has no remaining prerequisites)
            pre = queue.popleft()
            order.append(pre)  # Add to result order
            
            # 3. For every course that depends on this prerequisite
            for course in adj[pre]:
                in_degree[course] -= 1  # One prerequisite satisfied
                # If all prerequisites are met, add to queue
                if in_degree[course] == 0:
                    queue.append(course)
        
        # 4. If 'order' length is numCourses, we found a valid path
        # Otherwise, there was a cycle
        return order if len(order) == numCourses else []
```

---

## Detailed Walkthrough

### Example 1: Valid Course Order

**Input:** `numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]`

**Graph:**
```
0 → 1 → 3
↓   ↗
2 ──
```

**Adjacency List:**
```
0: [1, 2]
1: [3]
2: [3]
3: []
```

**In-degrees:**
```
in_degree = [0, 1, 1, 2]
Course 0: 0 prerequisites
Course 1: 1 prerequisite (0)
Course 2: 1 prerequisite (0)
Course 3: 2 prerequisites (1, 2)
```

#### BFS Execution

| Step | Queue | order | In-degree | Action |
|------|-------|-------|-----------|--------|
| Init | [0] | [] | [0,1,1,2] | Start with course 0 (in-degree 0) |
| 1 | [1,2] | [0] | [0,0,0,2] | Take 0, update 1 and 2 |
| 2 | [2] | [0,1] | [0,0,0,1] | Take 1, update 3 (now has in-degree 1) |
| 3 | [3] | [0,1,2] | [0,0,0,0] | Take 2, update 3 (now has in-degree 0) |
| 4 | [] | [0,1,2,3] | [0,0,0,0] | Take 3, done! |

**Result:** `[0,1,2,3]`

**Verification:**
- Course 0: No prerequisites ✓
- Course 1: Needs 0, and 0 comes before 1 ✓
- Course 2: Needs 0, and 0 comes before 2 ✓
- Course 3: Needs 1 and 2, both come before 3 ✓

---

### Example 2: Alternative Valid Order

**Same Graph:** `numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]`

**Alternative BFS Execution:** (if we process queue differently)

| Step | Queue | order | In-degree | Action |
|------|-------|-------|-----------|--------|
| Init | [0] | [] | [0,1,1,2] | Start with course 0 |
| 1 | [1,2] | [0] | [0,0,0,2] | Take 0, update 1 and 2 |
| 2 | [3] | [0,2] | [0,0,0,1] | Take **2 first**, update 3 |
| 3 | [3] | [0,2,1] | [0,0,0,0] | Take 1, update 3 (in-degree 0) |
| 4 | [] | [0,2,1,3] | [0,0,0,0] | Take 3, done! |

**Result:** `[0,2,1,3]` (also valid!)

**Both orders are correct** because courses 1 and 2 are independent (can be taken in any order after 0).

---

### Example 3: Cycle Detected

**Input:** `numCourses = 2, prerequisites = [[1,0],[0,1]]`

**Graph:**
```
0 ⇄ 1  (cycle!)
```

**In-degrees:**
```
in_degree = [1, 1]
Both courses have prerequisites!
```

#### BFS Execution

| Step | Queue | order | In-degree | Action |
|------|-------|-------|-----------|--------|
| Init | [] | [] | [1,1] | **No course with in-degree 0!** |
| End | [] | [] | [1,1] | Stuck! Cannot proceed |

**Result:** `[]` (empty array)

**Why:** Both courses depend on each other, creating a cycle. No course can be taken first.

---

## Visual Walkthrough

### Valid Topological Sort

```
Graph:
    0
   / \
  1   2
   \ /
    3

Step-by-step:
1. Take 0 (no prereqs)     order: [0]
2. Take 1 (0 done)         order: [0,1]
3. Take 2 (0 done)         order: [0,1,2]
4. Take 3 (1,2 done)       order: [0,1,2,3] ✓

Or alternatively:
1. Take 0                  order: [0]
2. Take 2 (0 done)         order: [0,2]
3. Take 1 (0 done)         order: [0,2,1]
4. Take 3 (1,2 done)       order: [0,2,1,3] ✓
```

### Cycle Detection

```
Graph:
    0 ⇄ 1

Queue never gets started!
in_degree = [1,1]  (both have prerequisites)
No course with in_degree 0 → stuck → return []
```

---

## Key Differences from Course Schedule I

| Aspect | Course Schedule I | Course Schedule II |
|--------|-------------------|-------------------|
| **Return Type** | Boolean (true/false) | List (order or empty) |
| **What we track** | Number of courses taken | Actual order of courses |
| **Key line** | `taken_courses += 1` | `order.append(course)` |
| **Success check** | `taken_courses == numCourses` | `len(order) == numCourses` |
| **On cycle** | Return `false` | Return `[]` |
| **Algorithm** | Same (Kahn's) | Same (Kahn's) |

**The difference is ONE LINE:** Instead of just counting, we record the order!

---

## Why Multiple Valid Answers?

**Question:** Why can there be multiple correct orders?

**Answer:** When courses are **independent** (no direct dependency), they can be taken in any order.

**Example:**
```
Graph:
  0 → 1
  0 → 2
  
Courses 1 and 2 are independent!

Valid orders:
- [0, 1, 2] ✓
- [0, 2, 1] ✓

Invalid:
- [1, 0, 2] ✗ (0 must come before 1)
```

**In BFS:** The order depends on which element comes out of the queue first (implementation-dependent).

---

## Edge Cases

### 1. No Prerequisites
```
Input: numCourses = 3, prerequisites = []
Output: [0, 1, 2] (or any permutation)
Explanation: All courses independent
```

### 2. Linear Chain
```
Input: numCourses = 3, prerequisites = [[1,0],[2,1]]
Graph: 0 → 1 → 2
Output: [0, 1, 2] (only one valid order)
```

### 3. Single Course
```
Input: numCourses = 1, prerequisites = []
Output: [0]
```

### 4. All Depend on One
```
Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,0]]
Graph: 0 → 1
       0 → 2
       0 → 3
Output: [0, 1, 2, 3] or [0, 2, 1, 3] or any order with 0 first
```

### 5. Complex Cycle
```
Input: prerequisites = [[1,0],[2,1],[0,2]]
Graph: 0 → 1 → 2 → 0 (cycle)
Output: []
```

---

## Common Pitfalls

1. **Forgetting to check cycle:** Must return `[]` if `len(order) != numCourses`
2. **Reversing edge direction:** `[a, b]` means `b → a` (b is prerequisite of a)
3. **Modifying in-degree incorrectly:** Must decrement when removing edge
4. **Not initializing queue properly:** Must add ALL courses with in-degree 0
5. **Assuming unique answer:** Multiple valid orders may exist

---

## Alternative Approach: DFS-Based Topological Sort

While Kahn's algorithm (BFS) is more intuitive for this problem, we can also use DFS:

```python
class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        adj = [[] for _ in range(numCourses)]
        for course, pre in prerequisites:
            adj[pre].append(course)
        
        visited = set()
        path = set()
        order = []
        
        def dfs(course):
            if course in path:
                return False  # Cycle detected
            if course in visited:
                return True   # Already processed
            
            path.add(course)
            for neighbor in adj[course]:
                if not dfs(neighbor):
                    return False
            
            path.remove(course)
            visited.add(course)
            order.append(course)  # Add in post-order
            return True
        
        for i in range(numCourses):
            if not dfs(i):
                return []
        
        return order[::-1]  # Reverse post-order gives topological sort
```

**Key Difference:** DFS adds courses in **post-order** (after exploring all dependencies), so we must reverse the result.

---

## BFS vs DFS for Topological Sort

| Aspect | BFS (Kahn's) | DFS |
|--------|--------------|-----|
| **Intuition** | Remove nodes with no incoming edges | Post-order traversal |
| **Order** | Natural (forward) | Reversed (need to reverse) |
| **Cycle Detection** | Check if all nodes processed | Check for back edges |
| **Code Complexity** | Slightly simpler | Slightly more complex |
| **Memory** | Queue | Recursion stack |
| **Preference** | ✓ More intuitive for this problem | Valid alternative |

**For Course Schedule II:** BFS (Kahn's) is preferred because:
- Natural forward ordering (no reversal needed)
- Easier to understand (peel dependencies layer by layer)

---

## Complexity Analysis

**Time Complexity:** $O(V + E)$
- Build graph: $O(E)$ for edges
- BFS: Visit each vertex once $O(V)$, check each edge once $O(E)$
- Total: $O(V + E)$ where $V = \text{numCourses}$, $E = \text{len(prerequisites)}$

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$ to store all edges
- In-degree array: $O(V)$
- Queue: $O(V)$ in worst case (all courses in queue at once)
- Order array: $O(V)$
- Total: $O(V + E)$

---

## Real-World Applications

**Topological sort is used in:**

1. **Build Systems:** Compile dependencies (e.g., Makefile, Maven)
   - Files must be compiled in dependency order

2. **Package Managers:** Install packages (npm, pip, apt)
   - Install dependencies before dependents

3. **Task Scheduling:** Project management (PERT charts)
   - Complete prerequisite tasks first

4. **Spreadsheet Formulas:** Cell calculation order
   - Calculate cells referenced by others first

5. **Database Schema:** Foreign key constraints
   - Create referenced tables first

6. **University Course Planning:** Actual course scheduling!
   - Take prerequisites before advanced courses

---

## Relationship to Other Problems

| Problem | Relationship |
|---------|-------------|
| **Course Schedule I** | Detects if topological sort exists |
| **Course Schedule II** | Returns the topological sort |
| **Course Schedule III** | Weighted scheduling (different algorithm) |
| **Alien Dictionary** | Topological sort on characters |
| **Parallel Courses** | Topological sort with levels/time |

---

## Key Takeaways

1. **Course Schedule II = Topological Sort** of course dependency graph
2. **Kahn's Algorithm (BFS):** Process nodes with zero in-degree iteratively
3. **Return order if successful, empty array if cycle detected**
4. **Multiple valid answers possible** when courses are independent
5. **One line difference from Course Schedule I:** Track order instead of just count
6. **Time and space:** Both $O(V + E)$
7. **Real-world:** Very practical for dependency resolution systems
