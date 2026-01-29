
# Path with Maximum Probability

**LeetCode Link:** [https://leetcode.com/problems/path-with-maximum-probability/](https://leetcode.com/problems/path-with-maximum-probability/)

---

## Problem Statement

You are given an undirected weighted graph of `n` nodes (0-indexed), represented by an edge list where `edges[i] = [a, b]` is an undirected edge connecting the nodes `a` and `b` with a probability of success of traversing that edge `succProb[i]`.

Given two nodes `start` and `end`, find the path with the **maximum probability** of success to go from `start` to `end` and return its success probability.

If there is no path from `start` to `end`, return `0`. Your answer will be accepted if it differs from the correct answer by at most `1e-5`.

**Example 1:**
```
Input: n = 3, edges = [[0,1],[1,2],[0,2]], succProb = [0.5,0.5,0.2], start = 0, end = 2
Output: 0.25000

Explanation:
Graph:
    0 --0.5--> 1
    |          |
  0.2        0.5
    |          |
    +--------→ 2

Path 1: 0 → 2 directly
  Probability = 0.2

Path 2: 0 → 1 → 2
  Probability = 0.5 × 0.5 = 0.25

Choose Path 2 (0.25 > 0.2)
```

**Example 2:**
```
Input: n = 3, edges = [[0,1],[1,2],[0,2]], succProb = [0.5,0.5,0.3], start = 0, end = 2
Output: 0.30000

Explanation:
Path 1: 0 → 2 directly
  Probability = 0.3

Path 2: 0 → 1 → 2
  Probability = 0.5 × 0.5 = 0.25

Choose Path 1 (0.3 > 0.25)
```

**Example 3:**
```
Input: n = 3, edges = [[0,1]], succProb = [0.5], start = 0, end = 2
Output: 0.00000

Explanation:
No path from 0 to 2, return 0
```

**Constraints:**
- $2 \leq n \leq 10^4$
- $0 \leq \text{start, end} < n$
- `start != end`
- $0 \leq a, b < n$
- `a != b`
- $0 \leq \text{succProb.length} == \text{edges.length} \leq 2 \times 10^4$
- $0 \leq \text{succProb[i]} \leq 1$
- There is at most one edge between every two nodes.

---

## Strategy: Modified Dijkstra with Max-Heap

### The Key Twist: Maximize Product, Not Minimize Sum

**Standard Dijkstra:** Find path with **minimum sum** of weights.
```
Path cost = edge1 + edge2 + edge3
Goal: Minimize sum
```

**This Problem:** Find path with **maximum product** of probabilities.
```
Path probability = prob1 × prob2 × prob3
Goal: Maximize product
```

### Why This Is Different

**Key Differences:**

| Aspect | Standard Dijkstra | This Problem |
|--------|------------------|--------------|
| **Operation** | Addition (`+`) | Multiplication (`×`) |
| **Goal** | Minimize | Maximize |
| **Heap Type** | Min-heap | Max-heap |
| **Initial Value** | Start = 0, Others = ∞ | Start = 1.0, Others = 0.0 |
| **Comparison** | `<` (smaller is better) | `>` (larger is better) |
| **Update Rule** | `dist + weight` | `prob × edge_prob` |

### Why Probabilities Multiply

**Probability Theory:**
```
Event A has probability P(A) = 0.5 (50%)
Event B has probability P(B) = 0.8 (80%)

Both events succeed (independent):
P(A and B) = P(A) × P(B) = 0.5 × 0.8 = 0.4 (40%)
```

**In our problem:**
```
Edge 0→1 succeeds with prob 0.5
Edge 1→2 succeeds with prob 0.5

Path 0→1→2 succeeds:
Both edges must work = 0.5 × 0.5 = 0.25 (25%)
```

### Visual Intuition

```
Graph with success probabilities:
    
    0 ---0.8--→ 1
    |           |
   0.3         0.9
    |           |
    ↓           ↓
    2 ←--0.7--- 3

Find maximum probability from 0 to 3:

Path A: 0 → 1 → 3
  Probability = 0.8 × 0.9 = 0.72

Path B: 0 → 2 → 3
  Probability = 0.3 × 0.7 = 0.21

Choose Path A (0.72 > 0.21)
```

### Algorithm Overview

1. **Build undirected adjacency list** (edges work both ways)
2. **Initialize max probabilities:**
   - Start node: `1.0` (100% chance we're already there)
   - All others: `0.0` (0% chance initially)
3. **Use max-heap** (negate values for Python's min-heap)
4. **Modified Dijkstra:**
   - Always process node with highest probability first
   - For each neighbor, calculate new probability by **multiplying**
   - Update if new probability is **greater** than current
5. **Early termination:** First time we pop `end` node, return its probability
6. **If queue exhausts:** No path exists, return `0.0`

---

## Solution: Max-Heap Dijkstra

```python
import heapq
from collections import defaultdict
from typing import List

class Solution:
    def maxProbability(self, n: int, edges: List[List[int]], succProb: List[float], start: int, end: int) -> float:
        # 1. Build Adjacency List (Undirected graph)
        adj = defaultdict(list)
        for i, (u, v) in enumerate(edges):
            prob = succProb[i]
            adj[u].append((v, prob))  # u → v with probability prob
            adj[v].append((u, prob))  # v → u with probability prob (undirected)
        
        # 2. Max Probabilities table initialized to 0.0
        # max_prob[i] = maximum probability to reach node i from start
        max_prob = [0.0] * n
        max_prob[start] = 1.0  # 100% chance we're at start
        
        # 3. Max-Priority Queue: (-probability, node)
        # Python only has min-heap, so negate to get max-heap behavior
        # Larger probability → more negative → pops first
        pq = [(-1.0, start)]  # Start with probability 1.0
        
        while pq:
            curr_prob, u = heapq.heappop(pq)
            curr_prob = -curr_prob  # Convert back to positive
            
            # EARLY TERMINATION: If we reached end, this is the maximum probability
            # Because we use max-heap, first pop of 'end' = optimal answer
            if u == end:
                return curr_prob
            
            # OPTIMIZATION: Skip if we've already found a better path to u
            # This prevents reprocessing stale heap entries
            if curr_prob < max_prob[u]:
                continue
            
            # 4. Explore all neighbors
            for v, edge_prob in adj[u]:
                # Calculate probability of reaching v through u
                # Multiply probabilities (not add!)
                new_prob = curr_prob * edge_prob
                
                # If this path has higher probability than previously known
                if new_prob > max_prob[v]:
                    max_prob[v] = new_prob
                    heapq.heappush(pq, (-new_prob, v))
        
        # If we exhausted the queue without reaching end, no path exists
        return 0.0
```

---

## Detailed Walkthrough

### Example: 3-Node Graph

**Input:** `n = 3`, `edges = [[0,1],[1,2],[0,2]]`, `succProb = [0.5,0.5,0.2]`, `start = 0`, `end = 2`

**Graph Visualization:**
```
    0 --0.5--> 1
    |          |
  0.2        0.5
    |          |
    +--------→ 2
```

#### Step-by-Step Execution

**Initial State:**
```python
max_prob = [1.0, 0.0, 0.0]  # Start at node 0 with prob 1.0
pq = [(-1.0, 0)]
```

| Step | Pop (-prob, node) | Convert prob | At End? | Explore Neighbors | Update max_prob | Heap After |
|------|------------------|--------------|---------|-------------------|-----------------|------------|
| 1 | (-1.0, 0) | prob=1.0 | No | • Node 1: 1.0×0.5=0.5<br>• Node 2: 1.0×0.2=0.2 | [1.0, 0.5, 0.2] | [(-0.5,1), (-0.2,2)] |
| 2 | (-0.5, 1) | prob=0.5 | No | • Node 0: skip (0.5×0.5=0.25 < 1.0)<br>• Node 2: 0.5×0.5=0.25 > 0.2 | [1.0, 0.5, 0.25] | [(-0.25,2), (-0.2,2)] |
| 3 | (-0.25, 2) | prob=0.25 | **YES!** ✓ | **Return 0.25** | - | - |

**Result:** `0.25`

**Optimal Path:** `0 → 1 → 2` with probability `0.5 × 0.5 = 0.25`

---

## Why This Works: The Mathematics

### Proof of Correctness

**Claim:** Modified Dijkstra with max-heap and multiplication finds maximum probability path.

**Key Properties:**
1. **Monotonicity:** Probabilities are between 0 and 1, so multiplying by more edges **decreases** probability
2. **Greedy Choice:** Processing highest probability first ensures we find the optimal path
3. **No "Negative" Probabilities:** All probabilities ≥ 0, similar to Dijkstra's non-negative weight assumption

**Why Greedy Works:**
```
If we pop node U with probability P:
  - This is the maximum probability to reach U
  - Any other path would have lower probability
  - Safe to process neighbors based on P
```

### Max-Heap Trick in Python

**Problem:** Python's `heapq` is a min-heap, but we need max-heap.

**Solution:** Negate the values!

```python
# Regular min-heap behavior:
heapq.heappush(pq, (0.8, 1))
heapq.heappush(pq, (0.5, 2))
heapq.heappop(pq)  # Returns (0.5, 2) ← smaller pops first

# Max-heap trick (negate):
heapq.heappush(pq, (-0.8, 1))  # Store -0.8
heapq.heappush(pq, (-0.5, 2))  # Store -0.5
prob, node = heapq.heappop(pq)  # Returns (-0.8, 1) ← more negative
prob = -prob  # Convert back: prob = 0.8
```

**Why this works:**
```
Larger probability → More negative → Pops first

Examples:
  0.9 → -0.9 ← pops first
  0.5 → -0.5 ← pops second
  0.1 → -0.1 ← pops last
```

---

## Comparison with Standard Dijkstra

### Side-by-Side Code

**Standard Dijkstra (Minimum Sum):**
```python
dist[start] = 0
dist[others] = float('inf')
pq = [(0, start)]  # Min-heap

while pq:
    current_dist, u = heapq.heappop(pq)
    
    if current_dist > dist[u]:  # Skip if worse
        continue
    
    for v, weight in adj[u]:
        new_dist = current_dist + weight  # ADD
        if new_dist < dist[v]:  # LESS THAN
            dist[v] = new_dist
            heapq.heappush(pq, (new_dist, v))
```

**This Problem (Maximum Product):**
```python
max_prob[start] = 1.0
max_prob[others] = 0.0
pq = [(-1.0, start)]  # Max-heap (negated)

while pq:
    current_prob, u = heapq.heappop(pq)
    current_prob = -current_prob  # Negate back
    
    if current_prob < max_prob[u]:  # Skip if worse
        continue
    
    for v, edge_prob in adj[u]:
        new_prob = current_prob * edge_prob  # MULTIPLY
        if new_prob > max_prob[v]:  # GREATER THAN
            max_prob[v] = new_prob
            heapq.heappush(pq, (-new_prob, v))  # Negate
```

**Key Differences:**
- `+` → `×` (addition to multiplication)
- `<` → `>` (minimize to maximize)
- `0` → `1.0` (start probability is 100%)
- `∞` → `0.0` (unknown probabilities start at 0%)
- Regular heap → Negated heap (min to max)

---

## Edge Cases

### 1. No Path Exists
```python
Input: n = 3, edges = [[0,1]], succProb = [0.5], start = 0, end = 2
Output: 0.0

Graph: 0 --0.5--> 1    2 (disconnected)

No path from 0 to 2, return 0.0
```

### 2. Direct Path vs Indirect Path
```python
Input: n = 3, edges = [[0,1],[1,2],[0,2]], succProb = [0.9,0.9,0.5], start = 0, end = 2
Output: 0.81

Direct: 0 → 2 = 0.5
Indirect: 0 → 1 → 2 = 0.9 × 0.9 = 0.81 ✓ Better!
```

### 3. Start = End (Not in Constraints)
```python
# Constraints guarantee start != end
# But if allowed:
start = end = 0
Return 1.0 (100% probability already there)
```

### 4. Probability = 0 Edge
```python
Input: edges = [[0,1],[1,2]], succProb = [0.0, 0.9]
Path: 0 → 1 → 2 = 0.0 × 0.9 = 0.0

Zero probability edge makes entire path impossible
```

### 5. Probability = 1 Edges
```python
Input: edges = [[0,1],[1,2]], succProb = [1.0, 1.0]
Path: 0 → 1 → 2 = 1.0 × 1.0 = 1.0

Guaranteed success!
```

---

## Common Mistakes

### ❌ Mistake 1: Using Addition Instead of Multiplication
```python
# Wrong!
new_prob = current_prob + edge_prob  # Should multiply!
```

**Fix:** Use `current_prob * edge_prob`.

### ❌ Mistake 2: Using Min-Heap Without Negation
```python
# Wrong! Will find minimum, not maximum
pq = [(1.0, start)]  # Regular min-heap
```

**Fix:** Negate: `pq = [(-1.0, start)]` and convert back when popping.

### ❌ Mistake 3: Wrong Initial Values
```python
# Wrong!
max_prob[start] = 0.0  # Should be 1.0!
max_prob[others] = float('inf')  # Should be 0.0!
```

**Fix:** Start = 1.0 (already there), others = 0.0 (unknown).

### ❌ Mistake 4: Using `<` Instead of `>`
```python
# Wrong! Looking for maximum, not minimum
if new_prob < max_prob[v]:  # Should be >
    max_prob[v] = new_prob
```

**Fix:** Use `>` for maximization.

### ❌ Mistake 5: Forgetting to Negate Back
```python
# Wrong!
curr_prob, u = heapq.heappop(pq)
# Use curr_prob directly (still negative!)
```

**Fix:** Convert back: `curr_prob = -curr_prob`.

---

## Complexity Analysis

**Time Complexity:** $O(E \log V)$
- Building adjacency list: $O(E)$
- Dijkstra's algorithm:
  - Each edge relaxed at most once: $O(E)$
  - Each relaxation involves heap operations: $O(\log V)$
  - Total: $O(E \log V)$
- Same as standard Dijkstra!

**Space Complexity:** $O(V + E)$
- Adjacency list: $O(E)$
- Max probability array: $O(V)$
- Priority queue: $O(V)$ in worst case
- Overall: $O(V + E)$

**For this problem:**
- $V = n \leq 10^4$
- $E = \text{edges.length} \leq 2 \times 10^4$
- Very efficient

---

## Interview Insights

### Q: Why multiply probabilities instead of add?

**A:** Probability of independent events succeeding together:
```
Flip coin: P(heads) = 0.5
Roll die: P(six) = 1/6

Both happen: P(heads AND six) = 0.5 × (1/6) = 0.083

In graphs:
  Edge A succeeds: 0.8
  Edge B succeeds: 0.9
  Both edges work: 0.8 × 0.9 = 0.72
```

### Q: What if we used logarithms?

**A:** Can convert multiplication to addition!
```python
# Convert to log space:
log_prob = log(prob)

# Multiplication becomes addition:
new_log = log_prob + log(edge_prob)

# Convert back:
actual_prob = exp(new_log)
```

**Advantage:** Avoids numerical underflow for very small probabilities.

**Disadvantage:** More complex, not necessary for this problem.

### Q: Why early termination works?

**A:** Max-heap guarantees:
```
When we pop node X with probability P:
  - P is the maximum probability to reach X
  - No future path can have higher probability
  - If X is the destination, we have the answer!

Early termination = first pop of destination = optimal
```

### Q: What if graph was directed?

**A:** Only add edge in one direction:
```python
for i, (u, v) in enumerate(edges):
    prob = succProb[i]
    adj[u].append((v, prob))  # Only u → v
    # Don't add v → u
```

---

## Related Problems

1. **1514. Path with Maximum Probability** (this problem)
2. **743. Network Delay Time** (standard min Dijkstra)
3. **1631. Path With Minimum Effort** (min-max Dijkstra)
4. **1368. Minimum Cost to Make at Least One Valid Path** (0-1 BFS)
5. **787. Cheapest Flights Within K Stops** (limited Dijkstra)

---

## Key Takeaways

1. **Modified Dijkstra for maximization** (not minimization)
2. **Multiply probabilities** along path (not add)
3. **Use max-heap** by negating values in Python
4. **Initialize start = 1.0, others = 0.0** (opposite of standard Dijkstra)
5. **Comparison operator flips:** `>` instead of `<`
6. **Early termination:** First pop of destination = optimal
7. **Undirected graph:** Add edges in both directions
8. **Time: $O(E \log V)$, same as standard Dijkstra**

**Mental Model:** Think of probability as "strength" that **decreases** as you multiply (like signal degradation). You want to find the path where the signal remains strongest!

 Why don't we use Logarithms?If you remember your math, $\log(a \times b) = \log(a) + \log(b)$.Some people solve this by taking the negative log of the probabilities and then using standard "Sum" Dijkstra. While mathematically elegant, it's often overkill for an interview and can lead to precision issues. Sticking to direct multiplication with a Max-Heap is cleaner.4. Critical Corner CasesDisconnected Start/End: The queue will empty, and we return 0.0.Cycles: Unlike "sum" Dijkstra with negative weights, multiplying by values between 0 and 1 will always make the probability smaller or equal. There is no "infinite loop" that increases the value, so Dijkstra remains stable