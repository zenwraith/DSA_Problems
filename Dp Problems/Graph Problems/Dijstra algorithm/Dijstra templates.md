# Dijkstra's Algorithm Templates & Patterns

A comprehensive guide to Dijkstra's algorithm templates and common problem patterns.

---

## Table of Contents
1. [Core Templates](#core-templates)
2. [When to Use Dijkstra](#when-to-use-dijkstra)
3. [Key Concepts](#key-concepts)
4. [Problem Patterns](#problem-patterns)

---

## Core Templates

### Template 1: General Graph (Adjacency List)

**Use this when:** You have a list of nodes and weighted edges (standard graph).

```python
import heapq

def dijkstra(n, adj, start_node):
    """
    Find shortest paths from start_node to all other nodes.
    
    Args:
        n: Number of nodes (0 to n-1)
        adj: Adjacency list where adj[u] = [(v, weight), ...]
        start_node: Starting node
    
    Returns:
        Dictionary mapping node -> shortest distance from start
    """
    # 1. Initialize distances to infinity, start node to 0
    distances = {i: float('inf') for i in range(n)}
    distances[start_node] = 0
    
    # 2. Min-heap: (distance, node)
    # Always process the node with smallest distance first
    pq = [(0, start_node)]
    
    while pq:
        current_dist, u = heapq.heappop(pq)
        
        # 3. OPTIMIZATION: Skip if we've already found a better path
        # This prevents reprocessing stale heap entries
        if current_dist > distances[u]:
            continue
        
        # 4. Explore all neighbors
        for v, weight in adj[u]:
            distance = current_dist + weight
            
            # 5. RELAXATION: Update if we found a shorter path
            if distance < distances[v]:
                distances[v] = distance
                heapq.heappush(pq, (distance, v))
    
    return distances
```

**Example Usage:**
```python
# Graph: 0 --1--> 1 --2--> 2
#        |               ↑
#        +------4--------+
adj = {
    0: [(1, 1), (2, 4)],
    1: [(2, 2)],
    2: []
}
result = dijkstra(3, adj, 0)
# Output: {0: 0, 1: 1, 2: 3}
```

---

### Template 2: Grid (Matrix)

**Use this when:** You're working on a 2D grid (common in coding interviews).

```python
import heapq

def dijkstra_grid(grid, start=(0,0), end=None):
    """
    Find shortest path in a grid.
    
    Args:
        grid: 2D matrix where grid[r][c] is the cost to enter that cell
        start: Starting position (row, col)
        end: Ending position (if None, compute distances to all cells)
    
    Returns:
        Shortest distance to end (or distance matrix if end is None)
    """
    rows, cols = len(grid), len(grid[0])
    
    # Distance matrix: dist[r][c] = shortest path to (r, c)
    dist = [[float('inf')] * cols for _ in range(rows)]
    dist[start[0]][start[1]] = 0
    
    # Min-heap: (cost, row, col)
    pq = [(0, start[0], start[1])]
    
    # 4 directions: right, left, down, up
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    
    while pq:
        d, r, c = heapq.heappop(pq)
        
        # Skip if we've found a better path already
        if d > dist[r][c]:
            continue
        
        # Early termination if we have a specific destination
        if end and (r, c) == end:
            return d
        
        # Explore 4 neighbors
        for dr, dc in directions:
            nr, nc = r + dr, c + dc
            
            # Check bounds
            if 0 <= nr < rows and 0 <= nc < cols:
                # Calculate new distance
                # NOTE: This logic changes based on the problem!
                new_dist = d + grid[nr][nc]
                
                # Relaxation step
                if new_dist < dist[nr][nc]:
                    dist[nr][nc] = new_dist
                    heapq.heappush(pq, (new_dist, nr, nc))
    
    return dist if not end else -1
```

**Example Usage:**
```python
# Grid with cell costs
grid = [
    [1, 3, 1],
    [1, 5, 1],
    [4, 2, 1]
]
result = dijkstra_grid(grid, start=(0,0), end=(2,2))
# Output: 7 (path: 1→1→1→2→1→1)
```

---

## When to Use Dijkstra

| Problem Type | Use Dijkstra? | Alternative |
|-------------|---------------|-------------|
| **Shortest Path (Unweighted)** | ❌ No | BFS is faster $O(V+E)$ |
| **Shortest Path (Weighted)** | ✅ Yes | None better |
| **Shortest Path (Negative Weights)** | ❌ No | Bellman-Ford |
| **Min-Max Path (Effort)** | ✅ Yes | Modified Dijkstra |
| **Network Delay Time** | ✅ Yes | Standard Dijkstra |
| **All-Pairs Shortest Path** | ⚠️ Maybe | Floyd-Warshall for small graphs |

---

## Key Concepts

### The Relaxation Process

**Relaxation** is the core of Dijkstra: "Can I improve the shortest path to node V by going through node U?"

```python
# Current: start → ... → V (cost = 10)
# New path: start → ... → U → V (cost = 8)

if distance_to_U + edge_weight(U, V) < distance_to_V:
    distance_to_V = distance_to_U + edge_weight(U, V)  # Relax!
    add V to priority queue
```

**Visual Example:**
```
Initial: dist[A] = 0, dist[B] = ∞, dist[C] = ∞

Step 1: Process A
  - Neighbor B: 0 + 5 = 5 < ∞  →  dist[B] = 5
  - Neighbor C: 0 + 10 = 10 < ∞  →  dist[C] = 10

Step 2: Process B (smaller distance)
  - Neighbor C: 5 + 2 = 7 < 10  →  dist[C] = 7 (relaxed!)

Result: Shortest path to C is 7, not 10
```

### Why No Visited Set?

**You might ask:** "Why not mark nodes as visited?"

**Answer:** The optimization `if current_dist > distances[u]: continue` does the same thing!

**Explanation:**
```python
# Traditional approach (with visited set):
visited = set()
if u in visited:
    continue
visited.add(u)

# Optimized approach (no visited set):
if current_dist > distances[u]:
    continue

# Both achieve the same goal:
# - Prevent reprocessing nodes
# - Skip stale heap entries
# - But the second is simpler!
```

**Why it works:**
- When we pop node U with distance D
- If we've already found a better path (distances[U] < D), skip
- Otherwise, this is the optimal path to U (greedy choice)

### Complexity Analysis

**Time Complexity:** $O(E \log V)$
- Each edge relaxed once: $O(E)$
- Each relaxation involves heap push/pop: $O(\log V)$
- Total: $O(E \log V)$

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$
- Distance array: $O(V)$
- Priority queue: $O(V)$ in worst case

**Comparison with BFS:**
| Algorithm | Time | Use Case |
|-----------|------|----------|
| BFS | $O(V + E)$ | Unweighted graphs |
| Dijkstra | $O(E \log V)$ | Weighted graphs |

### Critical Limitation: No Negative Weights

**Dijkstra FAILS with negative weights!**

**Example:**
```
Graph: A --(-5)--> B --1--> C
       |                    ↑
       +--------10----------+

Dijkstra's approach:
1. Pop A (dist=0)
2. Update C: dist[C] = 10
3. Update B: dist[B] = -5
4. Pop B (dist=-5)
5. Update C: -5 + 1 = -4 < 10  →  dist[C] = -4

BUT we already processed C with dist=10!
If we used the visited set or skipped C,
we'd get the WRONG answer (10 instead of -4).

Solution: Use Bellman-Ford instead!
```

**Why Dijkstra assumes non-negative weights:**
- Once a node is popped from the heap, its distance is FINAL
- No future path can improve it (greedy assumption)
- Negative weights violate this assumption

---

## Problem Patterns

### Pattern 1: Standard Dijkstra

**Recognition:** Direct shortest path with weighted edges.

**Key Insight:** Sum up edge weights along the path.

#### Example Problems

**1. Network Delay Time (LeetCode 743)**

**Problem:** Signal sent from node K, find time for all nodes to receive it.

**Solution Pattern:**
```python
def networkDelayTime(times, n, k):
    # Build adjacency list
    adj = {i: [] for i in range(1, n+1)}
    for u, v, w in times:
        adj[u].append((v, w))
    
    # Standard Dijkstra from node k
    distances = dijkstra(n+1, adj, k)
    
    # Answer = max distance (when last node receives signal)
    max_dist = max(distances.values())
    return max_dist if max_dist != float('inf') else -1
```

**Why this works:** Signal propagates like waves, reaching nodes at different times.

---

**2. Path with Maximum Probability (LeetCode 1514)**

**Problem:** Find path with highest probability of success.

**Key Twist:** Instead of minimizing sum, maximize product!

**Solution Pattern:**
```python
def maxProbability(n, edges, prob, start, end):
    adj = {i: [] for i in range(n)}
    for i, (u, v) in enumerate(edges):
        adj[u].append((v, prob[i]))
        adj[v].append((u, prob[i]))
    
    # Use MAX-HEAP (negate for min-heap)
    probs = [0.0] * n
    probs[start] = 1.0
    
    pq = [(-1.0, start)]  # Negative for max-heap
    
    while pq:
        curr_prob, u = heapq.heappop(pq)
        curr_prob = -curr_prob  # Convert back
        
        if u == end:
            return curr_prob
        
        if curr_prob < probs[u]:
            continue
        
        for v, edge_prob in adj[u]:
            new_prob = curr_prob * edge_prob  # MULTIPLY!
            if new_prob > probs[v]:
                probs[v] = new_prob
                heapq.heappush(pq, (-new_prob, v))
    
    return 0.0
```

**Key Differences:**
- Use max-heap (negate values)
- Multiply probabilities instead of adding costs
- Initialize start to 1.0 (100% probability)
- Want maximum, not minimum

---

**3. Cheapest Flights Within K Stops (LeetCode 787)**

**Problem:** Find cheapest path but limit to at most K stops.

**Key Twist:** Track number of stops in addition to cost.

**Solution Pattern:**
```python
def findCheapestPrice(n, flights, src, dst, k):
    adj = {i: [] for i in range(n)}
    for u, v, price in flights:
        adj[u].append((v, price))
    
    # State: (cost, node, stops_remaining)
    pq = [(0, src, k+1)]
    
    # Track best cost for (node, stops) combination
    visited = {}
    
    while pq:
        cost, u, stops in heapq.heappop(pq)
        
        if u == dst:
            return cost
        
        if stops == 0:
            continue
        
        # Avoid revisiting same state
        if (u, stops) in visited and visited[(u, stops)] <= cost:
            continue
        visited[(u, stops)] = cost
        
        for v, price in adj[u]:
            heapq.heappush(pq, (cost + price, v, stops - 1))
    
    return -1
```

**Key Insight:** State includes both position and remaining stops!

---

### Pattern 2: Min-Max Path

**Recognition:** Optimize the worst edge along the path (not the sum).

**Key Insight:** Use `max(current, edge)` instead of `sum`.

#### Example Problems

**1. Path With Minimum Effort (LeetCode 1631)**

**Problem:** Find path where maximum height difference is minimized.

**Solution Pattern:**
```python
def minimumEffortPath(heights):
    # Use max() instead of sum!
    new_effort = max(current_effort, abs(height_diff))
```

**See full solution in:** [Path with minimum effort.md](Path with minimum effort.md)

---

**2. Swim in Rising Water (LeetCode 778)**

**Problem:** Grid with elevations, can only move when water level reaches cell height.

**Key Insight:** Water level must be at least the maximum elevation along your path.

**Solution Pattern:**
```python
def swimInWater(grid):
    n = len(grid)
    dist = [[float('inf')] * n for _ in range(n)]
    dist[0][0] = grid[0][0]
    
    pq = [(grid[0][0], 0, 0)]
    
    while pq:
        time, r, c = heapq.heappop(pq)
        
        if r == n-1 and c == n-1:
            return time
        
        if time > dist[r][c]:
            continue
        
        for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < n and 0 <= nc < n:
                # Time = max elevation along path
                new_time = max(time, grid[nr][nc])
                if new_time < dist[nr][nc]:
                    dist[nr][nc] = new_time
                    heapq.heappush(pq, (new_time, nr, nc))
    
    return -1
```

**Mental Model:** You need enough water to cover the tallest point on your path.

---

### Pattern 3: State-Based Dijkstra

**Recognition:** Position alone doesn't define state—you need extra information.

**Key Insight:** State = (position, additional_info)

**Common Additional Info:**
- Keys collected (bitmask)
- Obstacles removed
- Energy remaining
- Direction facing

#### Example Problems

**1. Shortest Path to Get All Keys (LeetCode 864)**

**Problem:** Navigate grid collecting keys to unlock doors, find shortest path.

**Key Insight:** State = (row, col, keys_collected_bitmask)

**Solution Pattern:**
```python
def shortestPathAllKeys(grid):
    rows, cols = len(grid), len(grid[0])
    
    # Find start and count keys
    start = None
    key_count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '@':
                start = (r, c)
            elif grid[r][c].islower():
                key_count += 1
    
    # State: (distance, row, col, keys_bitmask)
    # keys_bitmask: bit i = 1 if we have key i
    all_keys = (1 << key_count) - 1
    pq = [(0, start[0], start[1], 0)]
    
    # visited: {(row, col, keys): distance}
    visited = {(start[0], start[1], 0): 0}
    
    while pq:
        dist, r, c, keys = heapq.heappop(pq)
        
        # Found all keys!
        if keys == all_keys:
            return dist
        
        if (r, c, keys) in visited and visited[(r, c, keys)] < dist:
            continue
        
        for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
            nr, nc = r + dr, c + dc
            
            if 0 <= nr < rows and 0 <= nc < cols:
                cell = grid[nr][nc]
                
                # Wall
                if cell == '#':
                    continue
                
                # Door (need key)
                if cell.isupper():
                    key_bit = ord(cell) - ord('A')
                    if not (keys & (1 << key_bit)):
                        continue
                
                new_keys = keys
                # Key (collect it)
                if cell.islower():
                    key_bit = ord(cell) - ord('a')
                    new_keys = keys | (1 << key_bit)
                
                new_dist = dist + 1
                state = (nr, nc, new_keys)
                
                if state not in visited or visited[state] > new_dist:
                    visited[state] = new_dist
                    heapq.heappush(pq, (new_dist, nr, nc, new_keys))
    
    return -1
```

**Key Techniques:**
- Bitmask for key collection (6 keys max → 2^6 = 64 states)
- State space: rows × cols × 2^keys
- Can't just track (r, c) because having different keys matters!

---

**2. Minimum Obstacle Removal to Reach Corner (LeetCode 2290)**

**Problem:** Grid with obstacles (1) and empty cells (0), find path removing minimum obstacles.

**Key Insight:** This is 0-1 BFS! (Special case of Dijkstra)

**Solution Pattern:**
```python
from collections import deque

def minimumObstacles(grid):
    rows, cols = len(grid), len(grid[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    dist[0][0] = 0
    
    # Use deque for 0-1 BFS
    dq = deque([(0, 0, 0)])  # (obstacles, row, col)
    
    while dq:
        obstacles, r, c = dq.popleft()
        
        if r == rows-1 and c == cols-1:
            return obstacles
        
        if obstacles > dist[r][c]:
            continue
        
        for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
            nr, nc = r + dr, c + dc
            
            if 0 <= nr < rows and 0 <= nc < cols:
                # Cost = number of obstacles removed
                new_obstacles = obstacles + grid[nr][nc]
                
                if new_obstacles < dist[nr][nc]:
                    dist[nr][nc] = new_obstacles
                    
                    # 0-1 BFS optimization:
                    # If weight 0, add to front (higher priority)
                    # If weight 1, add to back
                    if grid[nr][nc] == 0:
                        dq.appendleft((new_obstacles, nr, nc))
                    else:
                        dq.append((new_obstacles, nr, nc))
    
    return dist[rows-1][cols-1]
```

**0-1 BFS Trick:**
- Weights are only 0 or 1
- Use deque instead of heap
- Weight 0: add to front (highest priority)
- Weight 1: add to back (lower priority)
- Faster than standard Dijkstra: $O(V+E)$ instead of $O(E \log V)$

---

### Pattern 4: Hidden Dijkstra

**Recognition:** Doesn't look like a graph, but can be modeled as one.

**Key Insight:** Define nodes and edges creatively.

#### Example Problems

**1. The Maze II (LeetCode 505)**

**Problem:** Ball rolls in a direction until hitting a wall. Find shortest path.

**Key Insight:** 
- Nodes = stopping positions (not every cell!)
- Edges = rolling distances between stops

**Solution Pattern:**
```python
def shortestDistance(maze, start, destination):
    rows, cols = len(maze), len(maze[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    dist[start[0]][start[1]] = 0
    
    pq = [(0, start[0], start[1])]
    
    while pq:
        d, r, c = heapq.heappop(pq)
        
        if [r, c] == destination:
            return d
        
        if d > dist[r][c]:
            continue
        
        # Try rolling in all 4 directions
        for dr, dc in [(0,1), (0,-1), (1,0), (-1,0)]:
            nr, nc = r, c
            steps = 0
            
            # Roll until hitting a wall
            while (0 <= nr + dr < rows and 
                   0 <= nc + dc < cols and 
                   maze[nr + dr][nc + dc] == 0):
                nr += dr
                nc += dc
                steps += 1
            
            # (nr, nc) is the stopping position
            new_dist = d + steps
            
            if new_dist < dist[nr][nc]:
                dist[nr][nc] = new_dist
                heapq.heappush(pq, (new_dist, nr, nc))
    
    return -1 if dist[destination[0]][destination[1]] == float('inf') else dist[destination[0]][destination[1]]
```

**Mental Model:** Think of "rolling" as edges between stopping points.

---

**2. K-Similar Strings (LeetCode 854)**

**Problem:** Minimum swaps to transform string A into string B.

**Key Insight:** 
- Nodes = string states
- Edges = swaps (weight = 1)

**Solution Pattern:**
```python
from collections import deque

def kSimilarity(A, B):
    if A == B:
        return 0
    
    # BFS since all edges have weight 1
    queue = deque([(A, 0)])
    visited = {A}
    
    while queue:
        curr, swaps = queue.popleft()
        
        # Find first position where curr differs from B
        i = 0
        while curr[i] == B[i]:
            i += 1
        
        # Try swapping with all valid positions
        for j in range(i+1, len(curr)):
            if curr[j] == B[i] and curr[j] != B[j]:
                # Perform swap
                next_str = list(curr)
                next_str[i], next_str[j] = next_str[j], next_str[i]
                next_str = ''.join(next_str)
                
                if next_str == B:
                    return swaps + 1
                
                if next_str not in visited:
                    visited.add(next_str)
                    queue.append((next_str, swaps + 1))
    
    return -1
```

**Why BFS not Dijkstra?** All edges have weight 1, so BFS is optimal!

---

## Quick Reference Table

| Pattern | Key Characteristic | Update Rule | Example |
|---------|-------------------|-------------|---------|
| **Standard** | Sum edge weights | `dist + weight` | Network Delay |
| **Min-Max** | Worst edge matters | `max(dist, weight)` | Minimum Effort |
| **Max-Product** | Multiply probabilities | `dist * probability` | Max Probability |
| **State-Based** | Extra state info | Depends on state | Collect Keys |
| **0-1 BFS** | Weights only 0 or 1 | Use deque | Obstacle Removal |
| **Hidden** | Non-obvious graph | Creative modeling | Maze Rolling Ball |

---

## Common Pitfalls & Tips

### ❌ Pitfall 1: Forgetting to Skip Stale Entries
```python
# Wrong!
current_dist, u = heapq.heappop(pq)
# Process immediately without checking
```

**Fix:**
```python
current_dist, u = heapq.heappop(pq)
if current_dist > distances[u]:
    continue  # Skip stale entry!
```

### ❌ Pitfall 2: Using Dijkstra for Unweighted Graphs
```python
# Wrong! Overkill for unweighted graph
pq = [(0, start)]
# Use BFS instead!
```

**Fix:** Use BFS for unweighted graphs (faster and simpler).

### ❌ Pitfall 3: Wrong Heap Ordering
```python
# Wrong! Forgetting distance comes first
heapq.heappush(pq, (node, distance))  # Will sort by node!
```

**Fix:**
```python
heapq.heappush(pq, (distance, node))  # Sort by distance
```

### ✅ Tip 1: Early Termination for Single Target
```python
if u == destination:
    return current_dist  # No need to explore further!
```

### ✅ Tip 2: For Max-Heap, Negate Values
```python
# Python only has min-heap
heapq.heappush(pq, (-value, node))  # Negate for max
value = -heapq.heappop(pq)[0]  # Negate back
```

### ✅ Tip 3: State Space Analysis
```
Total states = position_states × additional_info_states

Example (Collect Keys):
  Positions: 30 × 30 = 900
  Keys: 2^6 = 64
  Total: 900 × 64 = 57,600 states
```

---

## Interview Strategy

**Step 1: Identify if it's Dijkstra**
- ✅ Weighted graph?
- ✅ Finding shortest/minimum path?
- ✅ No negative weights?
- ✅ Not all weights are equal? (otherwise use BFS)

**Step 2: Choose the Right Template**
- Graph with edges → General template
- Grid → Grid template
- Need extra state → State-based template

**Step 3: Modify the Update Rule**
- Standard: `dist + weight`
- Min-max: `max(dist, weight)`
- Max-product: `dist * probability`
- Custom: depends on problem

**Step 4: Implement and Test**
- Start with simple case
- Check edge cases (single node, no path, etc.)
- Verify complexity

---

## Summary

**Key Takeaways:**

1. **Dijkstra = Greedy + Priority Queue**
   - Always explore minimum cost path first
   - Greedy choice is optimal for non-negative weights

2. **Template Variations:**
   - General graph: adjacency list
   - Grid: 2D matrix with directions
   - State-based: include extra info in state

3. **Pattern Recognition:**
   - Standard: sum of weights
   - Min-Max: maximum along path
   - State-based: position + additional info
   - Hidden: creative graph modeling

4. **Optimization:**
   - Skip stale entries: `if curr_dist > dist[u]: continue`
   - Early termination for single target
   - 0-1 BFS for binary weights

5. **Complexity:**
   - Time: $O(E \log V)$
   - Space: $O(V + E)$
   - No negative weights allowed!

**Mental Model:** Dijkstra explores paths like waves spreading from the source, always choosing the cheapest unexplored option. The priority queue ensures we find optimal paths greedily.

---

## Additional Problem Categories

### 1. The "Standard" Dijkstra Problems

These are direct applications where you are finding the minimum cumulative cost.

**Network Delay Time (743):** The literal "Hello World" of Dijkstra. You are given a list of travel times between nodes and asked how long it takes for a signal to reach all nodes.

**Path with Maximum Probability (1514):** A clever twist. Instead of minimizing sum, you maximize product.
- **Tip:** Use a Max-Heap and initialize distances to 0.

**Cheapest Flights Within K Stops (787):** A "limited" Dijkstra. You need to find the cheapest path but stop if you exceed $K$ edges.
- **Alternative:** Can also be solved with Bellman-Ford.

---

### 2. The "Min-Max" Pattern

In these problems, you aren't summing up weights; you are looking for a path where the single largest edge is minimized (or the single smallest edge is maximized).

**Path With Minimum Effort (1631):** You find a path where the maximum absolute difference between adjacent cells is minimized.

**Swim in Rising Water (778):** You can only swim through a cell if the water level is high enough. You want to find the minimum time (water level) to reach the end.

---

### 3. State-Based Dijkstra

These are "Hard" level problems because the "node" in your Dijkstra isn't just a coordinate (r, c), but a state (r, c, energy) or (r, c, keys_collected).

**Shortest Path to Get All Keys (864):** You need to find the shortest path while picking up keys to open doors. Your state in the PQ becomes (dist, r, c, bitmask_of_keys).

**Minimum Obstacle Removal to Reach Corner (2290):** Every obstacle has a weight of 1, and empty spaces have a weight of 0.
- **Pro Tip:** This is specifically called 0-1 BFS, a faster version of Dijkstra for when weights are only 0 and 1.

---

### 4. Advanced/Hidden Dijkstra

These problems don't look like graphs at first, but you can model them as such.

**The Maze II (505):** A ball rolls until it hits a wall. Each "stop" is a node, and the distance rolled is the edge weight.

**Reach a Number (754):** Can be modeled as a shortest path on a number line, though math solutions are faster.