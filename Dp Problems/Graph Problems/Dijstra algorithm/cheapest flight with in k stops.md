# Cheapest Flights Within K Stops

**LeetCode Link:** [https://leetcode.com/problems/cheapest-flights-within-k-stops/](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

---

## Problem Statement

There are `n` cities connected by some number of flights. You are given an array `flights` where `flights[i] = [fromi, toi, pricei]` indicates that there is a flight from city `fromi` to city `toi` with cost `pricei`.

You are also given three integers `src`, `dst`, and `k`, return **the cheapest price** from `src` to `dst` with at most `k` stops. If there is no such route, return `-1`.

**Example 1:**
```
Input: n = 4, flights = [[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]], src = 0, dst = 3, k = 1
Output: 700

Explanation:
Graph:
    0 --100--> 1 --600--> 3
    ↑          |
  100        100
    |          ↓
    +-------- 2 --200--> 3

Path with at most 1 stop (2 edges):
  Route 1: 0 → 1 → 3 (cost: 100 + 600 = 700) ✓
  Route 2: 0 → 1 → 2 → 3 (2 stops, exceeds k=1) ✗

Cheapest within k=1 stops: 700
```

**Example 2:**
```
Input: n = 3, flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 1
Output: 200

Explanation:
Direct path: 0 → 2 (cost 500)
Path with 1 stop: 0 → 1 → 2 (cost 100 + 100 = 200) ✓ Cheaper!
```

**Example 3:**
```
Input: n = 3, flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 0
Output: 500

Explanation:
k = 0 means NO stops (direct flight only)
Only option: 0 → 2 (cost 500)
```

**Constraints:**
- $1 \leq n \leq 100$
- $0 \leq \text{flights.length} \leq (n \times (n - 1) / 2)$
- $\text{flights[i].length} == 3$
- $0 \leq \text{from}_i, \text{to}_i < n$
- $\text{from}_i \neq \text{to}_i$
- $1 \leq \text{price}_i \leq 10^4$
- There will not be any multiple flights between two cities.
- $0 \leq \text{src, dst, k} < n$
- $\text{src} \neq \text{dst}$

---

## Strategy: BFS with Level Tracking

### The Key Constraint: Limited Stops

**Standard Dijkstra:** Find shortest path with ANY number of edges.

**This Problem:** Find shortest path with **at most K stops** (K+1 edges).

**Why Different?**
```
Standard Dijkstra might find:
  Path A: 3 edges, cost 100 ← Optimal by cost
  
But with k=1 (at most 1 stop):
  Path A has 2 stops (3 edges - 1 = 2 stops) ✗ Exceeds limit!
  Path B: 2 edges, cost 150 ✓ Valid (1 stop)
  
Choose Path B even though it costs more!
```

### Understanding Stops vs Edges

**Important Distinction:**
```
Stops = intermediate cities (not counting source/destination)
Edges = flights taken

Example path: A → B → C → D
  Edges: 3 (A→B, B→C, C→D)
  Stops: 2 (only B and C are stops)
  
If k=2: We can take at most 3 edges
```

**In code:**
- `k stops` = `k+1 edges` maximum
- We iterate `k+1` times to allow `k` intermediate stops

### Why BFS Instead of Dijkstra?

**BFS Approach:**
- Process level by level (each level = one edge away)
- After `k+1` levels, we've explored all paths with at most `k` stops
- Natural way to track and limit the number of edges

**Dijkstra Approach:**
- Would need to track `(node, stops_used)` as state
- More complex state space
- BFS is simpler for this constraint

### Algorithm Overview (BFS Approach)

1. **Build adjacency list** from flights
2. **Initialize distances:** Source = 0, others = ∞
3. **BFS by levels:**
   - Each level represents one more flight/edge
   - Process all nodes at current level before moving to next
   - Stop after `k+1` levels (allowing `k` stops)
4. **Update distances:** If we find cheaper path within allowed stops
5. **Return:** Distance to destination (or -1 if unreachable)

---

## Solution 1: BFS with Level-by-Level Processing

```python
from collections import deque, defaultdict
from typing import List

class Solution:
    def findCheapestPrice(self, n: int, flights: List[List[int]], src: int, dst: int, k: int) -> int:
        # 1. Build adjacency list
        adj = defaultdict(list)
        for u, v, w in flights:
            adj[u].append((v, w))
        
        # 2. Initialize distances to infinity
        dist = [float('inf')] * n
        dist[src] = 0
        
        # 3. BFS queue: (node, current_cost)
        queue = deque([(src, 0)])
        stops = 0  # Track number of stops used
        
        # 4. Process level by level (each level = one more flight)
        while queue and stops <= k:
            size = len(queue)  # Process all nodes at current level
            
            # Process all nodes at this level before incrementing stops
            for i in range(size):
                u, current_cost = queue.popleft()
                
                # Explore all neighbors
                for v, weight in adj[u]:
                    new_cost = current_cost + weight
                    
                    # Update if we found a cheaper way to reach v
                    if new_cost < dist[v]:
                        dist[v] = new_cost
                        queue.append((v, dist[v]))
            
            stops += 1  # One more level processed
        
        # 5. Return result
        return dist[dst] if dist[dst] != float('inf') else -1
```

**Key Insights:**
- Process nodes **level by level** (not immediately as in Dijkstra)
- `size = len(queue)` ensures we process entire level before moving to next
- Track `stops` to ensure we don't exceed `k` stops
- Update distance even if node was visited before (might find cheaper path)

---

## Solution 2: Bellman-Ford (K+1 Iterations)

```python
from typing import List

class Solution:
    def findCheapestPrice(self, n: int, flights: List[List[int]], src: int, dst: int, k: int) -> int:
        # 1. Initialize prices to infinity
        prices = [float('inf')] * n
        prices[src] = 0
        
        # 2. Relax edges k+1 times (allowing k stops)
        for i in range(k + 1):
            # Create temporary array to avoid using updated values in same iteration
            temp_prices = prices[:]
            
            # Try to relax each edge
            for u, v, w in flights:
                # Skip if source node is unreachable
                if prices[u] == float('inf'):
                    continue
                
                # Relax edge u → v
                if prices[u] + w < temp_prices[v]:
                    temp_prices[v] = prices[u] + w
            
            # Update prices for next iteration
            prices = temp_prices
        
        # 3. Return result
        return prices[dst] if prices[dst] != float('inf') else -1
```

**Key Insights:**
- Bellman-Ford relaxes all edges multiple times
- After `i` iterations, we have shortest paths using at most `i` edges
- `k+1` iterations → paths with at most `k` stops (k+1 edges)
- `temp_prices` ensures we don't use updated values in same iteration

---

## Detailed Walkthrough

### Example: 4-City Network with k=1

**Input:** `n = 4`, `flights = [[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]]`, `src = 0`, `dst = 3`, `k = 1`

**Graph:**
```
    0 --100--> 1 --600--> 3
    ↑          |
  100        100
    |          ↓
    +-------- 2 --200--> 3
```

#### BFS Approach Walkthrough

**Initial State:**
```python
dist = [0, ∞, ∞, ∞]  # Source 0 has cost 0
queue = [(0, 0)]      # (node, cost)
stops = 0
```

**Level 0 (stops = 0):**
```
Process: (0, 0)
  Neighbors of 0:
    - Node 1: cost 0 + 100 = 100 < ∞ → Update dist[1] = 100
  
dist = [0, 100, ∞, ∞]
queue = [(1, 100)]
stops = 1
```

**Level 1 (stops = 1):**
```
Process: (1, 100)
  Neighbors of 1:
    - Node 2: cost 100 + 100 = 200 < ∞ → Update dist[2] = 200
    - Node 3: cost 100 + 600 = 700 < ∞ → Update dist[3] = 700
  
dist = [0, 100, 200, 700]
queue = [(2, 200), (3, 700)]
stops = 2  ← Exceeds k=1, stop here!
```

**Result:** `dist[3] = 700`

**Path:** `0 → 1 → 3` (cost 700, 1 stop)

#### Bellman-Ford Approach Walkthrough

**Initial State:**
```python
prices = [0, ∞, ∞, ∞]
```

**Iteration 1 (i=0):**
```
Process all edges:
  0→1 (100): prices[0]=0 + 100 = 100 < ∞ → temp[1] = 100
  1→2 (100): prices[1]=∞, skip
  2→0 (100): prices[2]=∞, skip
  1→3 (600): prices[1]=∞, skip
  2→3 (200): prices[2]=∞, skip

prices = [0, 100, ∞, ∞]
```

**Iteration 2 (i=1):**
```
Process all edges:
  0→1 (100): 0 + 100 = 100, no improvement
  1→2 (100): prices[1]=100 + 100 = 200 < ∞ → temp[2] = 200
  2→0 (100): prices[2]=∞, skip
  1→3 (600): prices[1]=100 + 600 = 700 < ∞ → temp[3] = 700
  2→3 (200): prices[2]=∞, skip

prices = [0, 100, 200, 700]
```

**Result:** `prices[3] = 700`

---

## Why This Works: The Mathematics

### BFS Correctness

**Claim:** Processing level by level ensures we find cheapest path within k stops.

**Proof:**
1. Each level represents paths with one more edge
2. After level L, we've explored all paths with ≤ L edges
3. After k+1 levels, we've explored all paths with ≤ k stops
4. We update distances if we find cheaper paths
5. Result is minimum cost among all valid paths

**Key Property:** BFS explores paths in order of number of edges (not cost).

### Bellman-Ford Correctness

**Claim:** k+1 iterations of edge relaxation find shortest paths with ≤ k stops.

**Proof:**
1. After i iterations, we have shortest paths using ≤ i edges
2. Each iteration considers all edges
3. After k+1 iterations, shortest paths with ≤ k+1 edges found
4. This equals paths with ≤ k stops

### Why temp_prices Is Crucial

**Without temp_prices (WRONG):**
```python
for i in range(k + 1):
    for u, v, w in flights:
        if prices[u] + w < prices[v]:
            prices[v] = prices[u] + w  # Using updated value!

# Problem: Within same iteration, updated values are used
# This allows using more edges than intended!
```

**With temp_prices (CORRECT):**
```python
for i in range(k + 1):
    temp_prices = prices[:]  # Snapshot
    for u, v, w in flights:
        if prices[u] + w < temp_prices[v]:
            temp_prices[v] = prices[u] + w
    prices = temp_prices  # Update after all edges processed

# All edges use OLD values, then update together
# Ensures exactly i edges used after i iterations
```

---

## Comparison: BFS vs Bellman-Ford

| Aspect | BFS Approach | Bellman-Ford Approach |
|--------|--------------|----------------------|
| **Time Complexity** | $O(k \times E)$ | $O(k \times E)$ |
| **Space Complexity** | $O(V + E)$ (queue + adj list) | $O(V)$ (prices array) |
| **Implementation** | More code, queue management | Simpler, nested loops |
| **Intuition** | Level-by-level exploration | Iterative edge relaxation |
| **Early Termination** | Possible (if queue empty) | No (processes all k+1 iterations) |
| **When to Use** | When graph is sparse | When flights array is compact |

**Both approaches are correct!** Choose based on preference:
- **BFS:** More intuitive for "levels" concept
- **Bellman-Ford:** Simpler implementation, less code

---

## Edge Cases

### 1. No Stops Allowed (k=0)
```python
Input: flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 0
Output: 500

k=0 means direct flight only (no intermediate stops)
Only path: 0 → 2 (cost 500)
Path 0 → 1 → 2 has 1 stop, exceeds k=0
```

### 2. No Path Within K Stops
```python
Input: flights = [[0,1,100],[1,2,100]], src = 0, dst = 2, k = 0
Output: -1

Only path: 0 → 1 → 2 (1 stop)
But k=0 requires direct flight
No valid path → return -1
```

### 3. Multiple Paths, Choose Cheapest Within K
```python
Input: flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 1
Output: 200

Path A: 0 → 2 (0 stops, cost 500)
Path B: 0 → 1 → 2 (1 stop, cost 200) ✓ Cheaper!

Both valid, choose Path B
```

### 4. Cheaper Path Requires More Stops
```python
Input: flights = [[0,1,10],[1,2,10],[0,2,100]], src = 0, dst = 2, k = 0
Output: 100

Cheapest path overall: 0 → 1 → 2 (cost 20, 1 stop)
But k=0 forbids it!
Must take direct: 0 → 2 (cost 100)
```

### 5. Disconnected Graph
```python
Input: flights = [[0,1,100]], src = 0, dst = 2, k = 10
Output: -1

Node 2 is not reachable from 0
No path exists regardless of k
```

---

## Common Mistakes

### ❌ Mistake 1: Confusing Stops and Edges
```python
# Wrong! k stops = k edges
for i in range(k):  # Should be k+1!
```

**Fix:** Use `range(k + 1)` because k stops means k+1 edges.

### ❌ Mistake 2: Not Using Temporary Array in Bellman-Ford
```python
# Wrong! Updates affect same iteration
for i in range(k + 1):
    for u, v, w in flights:
        if prices[u] + w < prices[v]:
            prices[v] = prices[u] + w  # Cascading updates!
```

**Fix:** Use `temp_prices` to snapshot values before updates.

### ❌ Mistake 3: Not Processing Level by Level in BFS
```python
# Wrong! Processes nodes as soon as added
while queue and stops <= k:
    u, cost = queue.popleft()
    # Process immediately
    stops += 1  # Wrong tracking!
```

**Fix:** Use `size = len(queue)` to process entire level before incrementing stops.

### ❌ Mistake 4: Stopping BFS Too Early
```python
# Wrong! Might find cheaper path later
if u == dst:
    return current_cost
```

**Fix:** Don't return early; cheaper paths might exist within k stops.

### ❌ Mistake 5: Using Dijkstra Without State Modification
```python
# Wrong! Standard Dijkstra doesn't track stops
pq = [(0, src)]
while pq:
    cost, u = heapq.heappop(pq)
    # Can't limit stops!
```

**Fix:** Either use BFS/Bellman-Ford, or modify Dijkstra to track `(cost, node, stops)`.

---

## Complexity Analysis

### BFS Approach

**Time Complexity:** $O(k \times E)$
- Process at most k+1 levels
- Each level might add all edges to queue
- Total: $O(k \times E)$

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$
- Distance array: $O(V)$
- Queue: $O(V)$ in worst case

### Bellman-Ford Approach

**Time Complexity:** $O(k \times E)$
- k+1 iterations
- Each iteration processes all E edges
- Total: $O(k \times E)$

**Space Complexity:** $O(V)$
- Prices array: $O(V)$
- Temp prices: $O(V)$
- No adjacency list needed

**For this problem:**
- $V = n \leq 100$
- $E \leq n(n-1)/2 \leq 4950$
- $k < n$
- Both approaches are efficient

---

## Interview Insights

### Q: Why not standard Dijkstra?

**A:** Standard Dijkstra finds shortest path by cost, but doesn't track/limit edges used.

**Solution:** Modify Dijkstra to track state as `(cost, node, stops)`:
```python
pq = [(0, src, 0)]  # (cost, node, stops_used)
visited = {}  # Track (node, stops) → cost

while pq:
    cost, u, stops = heapq.heappop(pq)
    
    if u == dst:
        return cost
    
    if stops > k:
        continue
    
    if (u, stops) in visited and visited[(u, stops)] <= cost:
        continue
    visited[(u, stops)] = cost
    
    for v, price in adj[u]:
        heapq.heappush(pq, (cost + price, v, stops + 1))
```

**Trade-off:** More complex state, but might be faster for sparse graphs.

### Q: When to use BFS vs Bellman-Ford?

**A:** 
- **BFS:** More intuitive, better for sparse graphs
- **Bellman-Ford:** Simpler code, no need to build adjacency list

### Q: Can we use DFS?

**A:** Yes, with memoization:
```python
def dfs(node, stops, cost):
    if node == dst:
        return cost
    if stops > k:
        return float('inf')
    
    min_cost = float('inf')
    for neighbor, price in adj[node]:
        min_cost = min(min_cost, dfs(neighbor, stops + 1, cost + price))
    
    return min_cost

# But this is exponential without memoization!
# BFS/Bellman-Ford are better choices
```

---

## Related Problems

1. **787. Cheapest Flights Within K Stops** (this problem)
2. **743. Network Delay Time** (standard Dijkstra, no stop limit)
3. **1514. Path with Maximum Probability** (maximize product)
4. **1334. Find the City With the Smallest Number of Neighbors at a Threshold Distance** (Floyd-Warshall)
5. **1928. Minimum Cost to Reach Destination in Time** (time constraint)

---

## Key Takeaways

1. **Limited Dijkstra:** Standard shortest path with constraint on number of edges
2. **Two approaches:** BFS (level-by-level) or Bellman-Ford (k+1 iterations)
3. **k stops = k+1 edges** (important distinction!)
4. **Temporary array crucial** in Bellman-Ford to prevent cascading updates
5. **Process level by level** in BFS to track stops correctly
6. **Don't return early** in BFS—cheaper path might exist within limit
7. **Time: $O(k \times E)$**, efficient for small k
8. **Both approaches equally valid**, choose based on preference

**Mental Model:** Think of each iteration/level as taking one more flight. After k+1 flights, you've explored all paths with at most k intermediate stops (layovers).