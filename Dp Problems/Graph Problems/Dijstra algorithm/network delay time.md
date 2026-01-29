# Network Delay Time

**LeetCode Link:** [https://leetcode.com/problems/network-delay-time/](https://leetcode.com/problems/network-delay-time/)

---

## Problem Statement

You are given a network of `n` nodes, labeled from `1` to `n`. You are also given `times`, a list of travel times as directed edges `times[i] = (ui, vi, wi)`, where `ui` is the source node, `vi` is the target node, and `wi` is the time it takes for a signal to travel from source to target.

We will send a signal from a given node `k`. Return the **minimum time** it takes for all the `n` nodes to receive the signal. If it is impossible for all nodes to receive the signal, return `-1`.

**Example 1:**
```
Input: times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
Output: 2

Visualization:
    1 ←--1-- 2 --1-→ 3
                    |
                    1
                    ↓
                    4

Signal sent from node 2:
  - Node 2 receives at time 0 (source)
  - Node 1 receives at time 1 (2→1)
  - Node 3 receives at time 1 (2→3)
  - Node 4 receives at time 2 (2→3→4)
  
Maximum time = 2 (when last node receives signal)
```

**Example 2:**
```
Input: times = [[1,2,1]], n = 2, k = 1
Output: 1

Node 1 → Node 2 (time 1)
Maximum time = 1
```

**Example 3:**
```
Input: times = [[1,2,1]], n = 2, k = 2
Output: -1

Signal starts at node 2, but there's no path to node 1.
Impossible for all nodes to receive signal.
```

**Constraints:**
- $1 \leq k \leq n \leq 100$
- $1 \leq \text{times.length} \leq 6000$
- $\text{times[i].length} == 3$
- $1 \leq u_i, v_i \leq n$
- $u_i \neq v_i$
- $0 \leq w_i \leq 100$
- All the pairs $(u_i, v_i)$ are **unique** (no duplicate edges)

---

## Strategy: Standard Dijkstra's Algorithm

### The Key Insight: Signal Propagation

**Think of the signal like a wave spreading through the network:**
- Signal starts at node `k` at time 0
- Signal travels along edges, taking time to reach neighbors
- Each node receives the signal when the **fastest path** reaches it
- Answer = time when the **last node** receives the signal

**Why Dijkstra?**
- This is a classic **shortest path** problem with weighted edges
- We need the minimum time to reach each node from source `k`
- Dijkstra efficiently finds shortest paths in weighted graphs

### Visual Intuition

```
Network:
    1 ←--1-- 2 --1-→ 3
                    |
                    1
                    ↓
                    4
Signal from node 2:

Time 0: [2] receives signal
        Distances: {1:∞, 2:0, 3:∞, 4:∞}

Time 1: [1, 3] receive signal (from 2)
        Distances: {1:1, 2:0, 3:1, 4:∞}

Time 2: [4] receives signal (from 3)
        Distances: {1:1, 2:0, 3:1, 4:2}

All nodes received! Maximum time = 2
```

### Algorithm Overview

1. **Build adjacency list:** Store graph as `{node: [(neighbor, weight), ...]}`
2. **Initialize distances:** All nodes to `∞`, source node `k` to `0`
3. **Dijkstra's algorithm:**
   - Use min-heap to always process node with smallest time
   - For each node, relax all neighbors
   - Update neighbor's time if we found a faster path
4. **Find maximum time:** The last node to receive signal determines the answer
5. **Check reachability:** If any node has distance `∞`, return `-1`

---

## Solution: Dijkstra with Min-Heap

```python
import heapq
from collections import defaultdict
from typing import List

class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        # 1. Build adjacency list
        # adj[u] = [(v, weight)] means edge u → v with travel time 'weight'
        adj = defaultdict(list)
        for u, v, w in times:
            adj[u].append((v, w))
        
        # 2. Initialize distances to infinity
        # Nodes are labeled 1 to n (not 0-indexed!)
        dist = {i: float('inf') for i in range(1, n + 1)}
        dist[k] = 0  # Source node has time 0
        
        # 3. Min-Priority Queue: (time_to_reach, node)
        # Start with source node k at time 0
        pq = [(0, k)]
        
        while pq:
            current_time, u = heapq.heappop(pq)
            
            # OPTIMIZATION: Skip if we've already found a faster path to u
            # This prevents reprocessing stale heap entries
            if current_time > dist[u]:
                continue
            
            # 4. Relax all neighbors of u
            for v, weight in adj[u]:
                # Calculate time to reach v through u
                new_time = current_time + weight
                
                # If this path is faster than previously known path to v
                if new_time < dist[v]:
                    dist[v] = new_time
                    heapq.heappush(pq, (new_time, v))
        
        # 5. Find the maximum time (when last node receives signal)
        max_time = max(dist.values())
        
        # If any node is unreachable (still infinity), return -1
        return max_time if max_time != float('inf') else -1
```

---

## Detailed Walkthrough

### Example: 4-Node Network

**Input:** `times = [[2,1,1],[2,3,1],[3,4,1]]`, `n = 4`, `k = 2`

**Step 1: Build Adjacency List**
```python
adj = {
    2: [(1, 1), (3, 1)],  # 2→1 (time 1), 2→3 (time 1)
    3: [(4, 1)],          # 3→4 (time 1)
    1: [],                # No outgoing edges
    4: []                 # No outgoing edges
}
```

**Step 2: Initialize Distances**
```python
dist = {1: ∞, 2: 0, 3: ∞, 4: ∞}
pq = [(0, 2)]  # Start from node 2 with time 0
```

**Step 3: Dijkstra's Algorithm**

| Step | Pop (time, node) | Current dist | Explore Neighbors | Update dist | Heap After |
|------|-----------------|--------------|-------------------|-------------|------------|
| Init | - | {1:∞, 2:0, 3:∞, 4:∞} | - | - | [(0,2)] |
| 1 | (0, 2) | {1:∞, 2:0, 3:∞, 4:∞} | • Node 1: 0+1=1 < ∞<br>• Node 3: 0+1=1 < ∞ | dist[1]=1<br>dist[3]=1 | [(1,1), (1,3)] |
| 2 | (1, 1) | {1:1, 2:0, 3:1, 4:∞} | No neighbors | - | [(1,3)] |
| 3 | (1, 3) | {1:1, 2:0, 3:1, 4:∞} | • Node 4: 1+1=2 < ∞ | dist[4]=2 | [(2,4)] |
| 4 | (2, 4) | {1:1, 2:0, 3:1, 4:2} | No neighbors | - | [] |

**Step 4: Find Maximum Time**
```python
dist = {1: 1, 2: 0, 3: 1, 4: 2}
max_time = max(1, 0, 1, 2) = 2
```

**Result:** `2` (Node 4 is the last to receive signal at time 2)

---

## Why This Works: The Mathematics

### Dijkstra's Correctness

**Key Property:** When a node is popped from the min-heap, its shortest path is finalized.

**Proof:**
1. We always process the node with minimum current time
2. If there were a shorter path, it would have been explored earlier
3. This greedy approach works because all edge weights are non-negative

**Invariant:** `dist[u]` always represents the shortest known path to `u`.

### Why Take the Maximum?

**Question:** "Why is the answer the maximum distance?"

**Answer:**
```
Signal propagates to all nodes simultaneously (in parallel):
  - Node A receives signal at time 3
  - Node B receives signal at time 5
  - Node C receives signal at time 2

When do ALL nodes have the signal?
  At time 5! (when the last node receives it)

Maximum distance = time when all nodes have received signal
```

### Signal Propagation Analogy

**Think of it like sending emails:**
```
You send an email to your team at 9:00 AM
  - Person A checks email at 9:03 AM (delay = 3 min)
  - Person B checks email at 9:05 AM (delay = 5 min)
  - Person C checks email at 9:02 AM (delay = 2 min)

When has everyone read your email?
  At 9:05 AM (when person B reads it)

Same concept: maximum delay = answer
```

---

## Edge Cases

### 1. Unreachable Nodes
```python
Input: times = [[1,2,1]], n = 2, k = 2
Output: -1

Node 2 can't reach node 1 (no path)
dist = {1: ∞, 2: 0}
max(∞, 0) = ∞ → return -1
```

### 2. Single Node
```python
Input: times = [], n = 1, k = 1
Output: 0

Only one node (the source itself)
Already has signal at time 0
```

### 3. Disconnected Components
```python
Input: times = [[1,2,1],[3,4,1]], n = 4, k = 1
Output: -1

Two components: {1,2} and {3,4}
Node 1 can reach node 2, but not nodes 3 or 4
Some nodes unreachable → return -1
```

### 4. Multiple Paths (Choose Shortest)
```python
Input: times = [[1,2,1],[1,2,3],[2,3,1]], n = 3, k = 1
Output: 2

Paths to node 2:
  - Direct: 1→2 (time 1) ✓ Better!
  - Direct: 1→2 (time 3)
Dijkstra chooses time 1

Path to node 3: 1→2→3 (time 2)
```

### 5. Self-Loop (Not in Constraints)
```python
# Constraints guarantee ui ≠ vi, so no self-loops
# But if allowed:
times = [[1,1,5],[1,2,1]], n = 2, k = 1

Self-loop doesn't help (going back to same node)
Just ignore it, shortest path to 2 is still 1
```

---

## Common Mistakes

### ❌ Mistake 1: Using 0-Indexed Nodes
```python
# Wrong! Nodes are 1 to n, not 0 to n-1
dist = {i: float('inf') for i in range(n)}
```

**Fix:** Use `range(1, n+1)` since nodes are labeled 1 to n.

### ❌ Mistake 2: Returning Minimum Instead of Maximum
```python
# Wrong!
return min(dist.values())  # Finds when first node receives signal
```

**Fix:** Return `max(dist.values())` (when last node receives signal).

### ❌ Mistake 3: Not Checking for Infinity
```python
# Wrong! Returns infinity instead of -1
return max(dist.values())
```

**Fix:** Check if max is infinity: `return max_time if max_time != float('inf') else -1`

### ❌ Mistake 4: Using BFS Instead of Dijkstra
```python
# Wrong! BFS finds shortest path by number of hops, not by weight
queue = deque([k])
```

**Fix:** Use Dijkstra with min-heap for weighted graphs.

### ❌ Mistake 5: Building Undirected Graph
```python
# Wrong! Graph is directed
adj[u].append((v, w))
adj[v].append((u, w))  # ← Don't add reverse edge!
```

**Fix:** Only add `u → v`, not `v → u` (graph is directed).

---

## Complexity Analysis

**Time Complexity:** $O(E \log V)$
- Building adjacency list: $O(E)$ where $E$ = number of edges
- Dijkstra's algorithm:
  - Each edge relaxed at most once: $O(E)$
  - Each relaxation involves heap push/pop: $O(\log V)$
  - Total: $O(E \log V)$
- Finding maximum: $O(V)$
- Overall: $O(E \log V)$

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$
- Distance dictionary: $O(V)$
- Priority queue: $O(V)$ in worst case
- Overall: $O(V + E)$

**For this problem:**
- $V = n \leq 100$
- $E = \text{times.length} \leq 6000$
- Very efficient even at maximum constraints

---

## Interview Insights

### Q: Why Dijkstra and not BFS?

**A:** BFS finds shortest path by **number of hops**, not by **weight sum**.

**Example:**
```
Graph:
  k --1--> A --1--> B
  k --100---------> B

BFS finds: k → B (1 hop, time 100)
Dijkstra finds: k → A → B (2 hops, time 2) ✓ Better!
```

### Q: Can this problem have negative weights?

**A:** No, constraints guarantee $0 \leq w_i \leq 100$.

**If it did:** Dijkstra would fail! Need Bellman-Ford instead.

**Why Dijkstra fails with negative weights:**
```
Graph: A --(-5)--> B --1--> C
       A --10--------------> C

Dijkstra's greedy approach:
1. Pop A (time 0)
2. Update C: time 10 ← finalized (wrong!)
3. Update B: time -5
4. Pop B, update C: time -4
5. But C already finalized! Miss better path.

Bellman-Ford checks all edges V-1 times → correct!
```

### Q: What if we wanted to track the actual path?

**A:** Maintain a `parent` dictionary:

```python
parent = {i: None for i in range(1, n+1)}

# During relaxation:
if new_time < dist[v]:
    dist[v] = new_time
    parent[v] = u  # Track where we came from
    heapq.heappush(pq, (new_time, v))

# Reconstruct path:
def get_path(node):
    path = []
    while node is not None:
        path.append(node)
        node = parent[node]
    return path[::-1]
```

### Q: How do we know when all nodes received the signal?

**A:** Check if any node still has distance `∞`:
```python
if any(d == float('inf') for d in dist.values()):
    return -1  # Not all nodes reachable
```

Or equivalently:
```python
max_time = max(dist.values())
if max_time == float('inf'):
    return -1
```

---

## Variations & Related Problems

### Similar Problems

1. **743. Network Delay Time** (this problem)
2. **1631. Path With Minimum Effort** (min-max Dijkstra)
3. **787. Cheapest Flights Within K Stops** (limited Dijkstra)
4. **1514. Path with Maximum Probability** (max-product Dijkstra)
5. **882. Reachable Nodes In Subdivided Graph** (weighted graph)

### Variation: Find Number of Reachable Nodes

**Question:** Instead of time, find how many nodes can receive the signal.

**Solution:**
```python
reachable = sum(1 for d in dist.values() if d != float('inf'))
return reachable
```

### Variation: Find Nodes at Specific Time

**Question:** Which nodes receive the signal at exactly time T?

**Solution:**
```python
nodes_at_time_T = [node for node, time in dist.items() if time == T]
return nodes_at_time_T
```

---

## Key Takeaways

1. **Standard Dijkstra application** for weighted directed graph
2. **Answer = maximum distance** (when last node receives signal)
3. **Check for unreachable nodes** (distance = ∞ → return -1)
4. **Nodes labeled 1 to n**, not 0 to n-1 (watch for off-by-one errors)
5. **Must use Dijkstra** for weighted graphs (BFS only works for unweighted)
6. **Time: $O(E \log V)$, Space: $O(V + E)$** — very efficient
7. **Skip stale heap entries** with `if current_time > dist[u]: continue`

**Mental Model:** Think of a signal spreading like a wave through the network, traveling at different speeds along different edges. The answer is when the wave has reached every node.