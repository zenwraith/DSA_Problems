# Find K Pairs with Smallest Sums

**LeetCode 373** | **Difficulty:** Medium | **Category:** Heap, Two Pointers, Matrix

---

## Problem Statement

You are given two integer arrays `nums1` and `nums2` sorted in **ascending order** and an integer `k`.

Define a **pair** `(u, v)` which consists of one element from the first array and one element from the second array.

Return the `k` pairs `(u1, v1), (u2, v2), ..., (uk, vk)` with the **smallest sums**.

---

## Examples

### Example 1:
```
Input: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
Output: [[1,2],[1,4],[1,6]]

Explanation:
The first 3 pairs are returned from the sequence:
[1,2] sum=3
[1,4] sum=5
[1,6] sum=7
[7,2] sum=9
[7,4] sum=11
[11,2] sum=13
[7,6] sum=13
[11,4] sum=15
[11,6] sum=17
```

### Example 2:
```
Input: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
Output: [[1,1],[1,1]]

Explanation:
The first 2 pairs are:
[1,1] sum=2  (nums1[0] + nums2[0])
[1,1] sum=2  (nums1[1] + nums2[0])

Note: [1,2] (sum=3) and [1,2] (sum=3) come later
```

### Example 3:
```
Input: nums1 = [1,2], nums2 = [3], k = 3
Output: [[1,3],[2,3]]

Explanation:
All possible pairs are returned:
[1,3] sum=4
[2,3] sum=5

Only 2 pairs exist, so we return both (k=3 but only 2 available)
```

### Example 4:
```
Input: nums1 = [1,2,4,5,6], nums2 = [3,5,7,9], k = 3
Output: [[1,3],[2,3],[1,5]]

Explanation:
Pairs in sorted order by sum:
[1,3] sum=4  ✓
[2,3] sum=5  ✓
[1,5] sum=6  ✓
[4,3] sum=7
...
```

---

## Strategy

### Min-Heap with Smart Exploration

**Key Insight:** Don't generate all pairs (O(m×n)). Instead, use a min-heap to explore pairs in increasing sum order, similar to merging k sorted lists.

**Why This Works:**

```
Observation: Both arrays are sorted

All pairs form a virtual 2D matrix:
       nums2[0]  nums2[1]  nums2[2]
nums1[0]  [1,2]    [1,4]     [1,6]
nums1[1]  [7,2]    [7,4]     [7,6]
nums1[2] [11,2]   [11,4]    [11,6]

Key properties:
1. Each row is sorted (moving right increases sum)
2. Each column is sorted (moving down increases sum)
3. Top-left [1,2] is guaranteed smallest

Strategy: Start from first row, explore systematically
- Begin with pairs (0,0), (1,0), (2,0), ... (min from each row)
- When we pop (i,j), next candidate from that row is (i,j+1)
- Use min-heap to always get smallest unexplored sum
```

**Algorithm Steps:**

1. **Initialize heap with first column:**
   - Add pairs (0,0), (1,0), (2,0), ..., (min(m,k)-1, 0)
   - These are smallest sums from each row
   - Limit to k rows to avoid unnecessary work

2. **Extract k smallest:**
   - Pop minimum from heap → add to result
   - If popped pair was (i,j), push (i,j+1) to heap
   - Repeat until we have k pairs or heap is empty

3. **Return result**

**Why Initialize First Column Only?**

```
Proof: If (i,j) is not yet explored, (i,j-1) must have smaller sum
- sum(i,j) = nums1[i] + nums2[j]
- sum(i,j-1) = nums1[i] + nums2[j-1]
- Since nums2 is sorted: nums2[j-1] < nums2[j]
- Therefore: sum(i,j-1) < sum(i,j)

Conclusion: We can't reach (i,j) until we've processed (i,j-1)
So starting with j=0 for each row ensures we explore in correct order
```

**Complexity:**
- **Time:** $O(k \log k)$ - k heap operations, each O(log k)
- **Space:** $O(k)$ - Heap holds at most k elements

---

## Solution

```python
import heapq

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        """
        Find k pairs with smallest sums from two sorted arrays.
        
        Strategy: Min-heap to explore pairs in increasing sum order
        1. Initialize heap with first column: (nums1[i] + nums2[0], i, 0)
        2. Pop minimum, add to result
        3. Push next pair from same row: (nums1[i] + nums2[j+1], i, j+1)
        4. Repeat until k pairs or heap empty
        
        Key insight: Both arrays sorted, so we can explore like
        merging k sorted lists. Start with smallest from each row.
        
        Args:
            nums1: First sorted array
            nums2: Second sorted array
            k: Number of pairs to find
            
        Returns:
            List of k pairs with smallest sums
        """
        if not nums1 or not nums2:
            return []
        
        res = []
        min_heap = []
        
        # Initialize heap with first column
        # Limit to min(len(nums1), k) to avoid extra work
        for i in range(min(len(nums1), k)):
            heapq.heappush(min_heap, (nums1[i] + nums2[0], i, 0))
        
        # Extract k smallest pairs
        while min_heap and len(res) < k:
            curr_sum, i, j = heapq.heappop(min_heap)
            res.append([nums1[i], nums2[j]])
            
            # Add next pair from same row
            if j + 1 < len(nums2):
                heapq.heappush(min_heap, (nums1[i] + nums2[j+1], i, j+1))
        
        return res
```

### Alternative: More Explicit Visited Tracking

```python
def kSmallestPairs(self, nums1, nums2, k):
    """
    Alternative with explicit visited set to prevent duplicates.
    Not needed if we follow the column-by-column pattern.
    """
    if not nums1 or not nums2:
        return []
    
    min_heap = [(nums1[0] + nums2[0], 0, 0)]
    visited = {(0, 0)}
    res = []
    
    while min_heap and len(res) < k:
        curr_sum, i, j = heapq.heappop(min_heap)
        res.append([nums1[i], nums2[j]])
        
        # Try next in same row
        if i + 1 < len(nums1) and (i+1, j) not in visited:
            heapq.heappush(min_heap, (nums1[i+1] + nums2[j], i+1, j))
            visited.add((i+1, j))
        
        # Try next in same column
        if j + 1 < len(nums2) and (i, j+1) not in visited:
            heapq.heappush(min_heap, (nums1[i] + nums2[j+1], i, j+1))
            visited.add((i, j+1))
    
    return res
```

---

## Detailed Walkthrough

### Example 1: `nums1 = [1,7,11], nums2 = [2,4,6], k = 3`

**Virtual Matrix (sum values):**
```
       j=0  j=1  j=2
i=0:    3    5    7     (1+2, 1+4, 1+6)
i=1:    9   11   13     (7+2, 7+4, 7+6)
i=2:   13   15   17    (11+2, 11+4, 11+6)
```

**Step 1: Initialize heap**

```
Add first column:
  (1+2, 0, 0) = (3, 0, 0)
  (7+2, 1, 0) = (9, 1, 0)
  (11+2, 2, 0) = (13, 2, 0)

Heap: [(3,0,0), (9,1,0), (13,2,0)]
```

**Step 2: Extract 1st smallest (k=1)**

| Action | Details |
|--------|---------|
| Pop | (3, 0, 0) → [1, 2] |
| Add to result | res = [[1,2]] |
| Push next | (1+4, 0, 1) = (5, 0, 1) |
| Heap | [(5,0,1), (9,1,0), (13,2,0)] |

**Step 3: Extract 2nd smallest (k=2)**

| Action | Details |
|--------|---------|
| Pop | (5, 0, 1) → [1, 4] |
| Add to result | res = [[1,2], [1,4]] |
| Push next | (1+6, 0, 2) = (7, 0, 2) |
| Heap | [(7,0,2), (9,1,0), (13,2,0)] |

**Step 4: Extract 3rd smallest (k=3)**

| Action | Details |
|--------|---------|
| Pop | (7, 0, 2) → [1, 6] |
| Add to result | res = [[1,2], [1,4], [1,6]] |
| Push next | j+1 = 3 >= len(nums2), don't push |
| Heap | [(9,1,0), (13,2,0)] |

**Done:** `len(res) = 3 = k`

**Final Answer:** `[[1,2], [1,4], [1,6]]` ✓

---

### Example 2: `nums1 = [1,1,2], nums2 = [1,2,3], k = 2`

**Virtual Matrix:**
```
       j=0  j=1  j=2
i=0:    2    3    4     (1+1, 1+2, 1+3)
i=1:    2    3    4     (1+1, 1+2, 1+3)
i=2:    3    4    5     (2+1, 2+2, 2+3)
```

**Initialize:** Heap = [(2,0,0), (2,1,0), (3,2,0)]

| Iteration | Pop | Result | Push | Heap |
|-----------|-----|--------|------|------|
| 1 | (2,0,0) → [1,1] | [[1,1]] | (3,0,1) | [(2,1,0), (3,0,1), (3,2,0)] |
| 2 | (2,1,0) → [1,1] | [[1,1],[1,1]] | (3,1,1) | [(3,0,1), (3,1,1), (3,2,0)] |

**Done:** k=2 reached

**Answer:** `[[1,1], [1,1]]` ✓

---

### Example 3: `nums1 = [1,2,4,5,6], nums2 = [3,5,7,9], k = 3`

**Virtual Matrix (first few):**
```
       j=0  j=1  j=2  j=3
i=0:    4    6    8   10    (1+3, 1+5, 1+7, 1+9)
i=1:    5    7    9   11    (2+3, 2+5, 2+7, 2+9)
i=2:    7    9   11   13    (4+3, 4+5, 4+7, 4+9)
i=3:    8   10   12   14    (5+3, 5+5, 5+7, 5+9)
i=4:    9   11   13   15    (6+3, 6+5, 6+7, 6+9)
```

**Initialize:** 
```
Heap = [(4,0,0), (5,1,0), (7,2,0), (8,3,0), (9,4,0)]
But we limit to k=3, so:
Heap = [(4,0,0), (5,1,0), (7,2,0)]
```

| Iteration | Pop | Result | Push | Heap |
|-----------|-----|--------|------|------|
| 1 | (4,0,0) → [1,3] | [[1,3]] | (6,0,1) | [(5,1,0), (6,0,1), (7,2,0)] |
| 2 | (5,1,0) → [2,3] | [[1,3],[2,3]] | (7,1,1) | [(6,0,1), (7,1,1), (7,2,0)] |
| 3 | (6,0,1) → [1,5] | [[1,3],[2,3],[1,5]] | (8,0,2) | [(7,1,1), (7,2,0), (8,0,2)] |

**Done:** k=3 reached

**Answer:** `[[1,3], [2,3], [1,5]]` ✓

---

## Why This Algorithm Works

### The Matrix Exploration Pattern

**Conceptual Model:**

```
Think of all pairs as a sorted matrix (by sum):
       j=0  j=1  j=2  j=3
i=0:    3    5    7    9
i=1:    9   11   13   15
i=2:   13   15   17   19

Properties:
1. Top-left is global minimum
2. Each row is sorted left-to-right
3. Each column is sorted top-to-bottom

Question: How to find k smallest without exploring all cells?
Answer: Use min-heap to explore only necessary cells
```

**Exploration Strategy:**

```
Start with first column (leftmost in each row)
These are guaranteed smallest in their rows

When we select cell (i,j):
- It's currently the minimum among unexplored
- Next candidate from this row is (i, j+1)
- We don't need to explore (i, j+2) yet

Example path for k=5:
(0,0) → (0,1) → (1,0) → (0,2) → (1,1)
  3       5       9       7       11

Heap ensures we always pick globally smallest unexplored
```

**Why Not Explore Both Directions?**

```
Could we explore both right (i, j+1) and down (i+1, j)?

Yes, but:
1. More complex (need visited set)
2. More heap operations
3. Starting with full first column is simpler

Our approach:
- Initialize all of first column
- Only expand right
- Simpler, fewer states, same result
```

### Comparison with Alternatives

**Approach 1: Brute Force**
```python
# Generate all pairs, sort, take k
pairs = []
for i in range(len(nums1)):
    for j in range(len(nums2)):
        pairs.append([nums1[i], nums2[j]])
pairs.sort(key=lambda x: x[0] + x[1])
return pairs[:k]
```
**Time:** O(mn log(mn)) - Too slow for large arrays

**Approach 2: Two Pointers (Doesn't Work)**
```python
# Can't use simple two pointers because pairs aren't linearly ordered
# Example: After [1,2], we might want [1,4] OR [7,2]
# No single pointer advancement rule works
```

**Approach 3: Our Heap Solution**
```python
# O(k log k) - Optimal!
# Only explore k pairs (plus some in heap)
# Much better than O(mn)
```

---

## Edge Cases

### 1. Empty Arrays
```python
nums1 = [], nums2 = [1,2], k = 3
# Output: []
# No pairs possible
```

### 2. k Larger Than Total Pairs
```python
nums1 = [1,2], nums2 = [3,4], k = 10
# Output: [[1,3],[1,4],[2,3],[2,4]]
# Return all 4 pairs (k=10 but only 4 exist)
```

### 3. Single Element Arrays
```python
nums1 = [1], nums2 = [2], k = 1
# Output: [[1,2]]
# Only one pair
```

### 4. Large k, Small Arrays
```python
nums1 = [1,2,3], nums2 = [4,5], k = 100
# Output: [[1,4],[1,5],[2,4],[2,5],[3,4],[3,5]]
# Only 6 pairs exist
```

### 5. Negative Numbers
```python
nums1 = [-10,0,5], nums2 = [-5,0,5], k = 3
# Output: [[-10,-5],[-10,0],[0,-5]]
# Smallest sums: -15, -10, -5
# Works correctly with negatives
```

### 6. Duplicates
```python
nums1 = [1,1,1], nums2 = [1,1,1], k = 4
# Output: [[1,1],[1,1],[1,1],[1,1]]
# All pairs have same sum, returns first 4
```

### 7. k = 1
```python
nums1 = [1,7,11], nums2 = [2,4,6], k = 1
# Output: [[1,2]]
# Just the smallest pair
```

---

## Common Mistakes

### ❌ Mistake 1: Initializing Entire First Row
```python
# Wrong! Should initialize first column, not first row
for j in range(min(len(nums2), k)):
    heapq.heappush(min_heap, (nums1[0] + nums2[j], 0, j))
```

**Why wrong?** This only explores pairs with nums1[0], misses others.

**Fix:**
```python
# Correct: Initialize first column
for i in range(min(len(nums1), k)):
    heapq.heappush(min_heap, (nums1[i] + nums2[0], i, 0))
```

### ❌ Mistake 2: Not Limiting Initial Heap Size
```python
# Wrong! Adds all rows even if k is small
for i in range(len(nums1)):  # Should be min(len(nums1), k)
    heapq.heappush(min_heap, (nums1[i] + nums2[0], i, 0))
```

**Why wrong?** If k=3 but nums1 has 1000 elements, we add 1000 to heap unnecessarily.

### ❌ Mistake 3: Pushing Wrong Indices
```python
# Wrong! Pushes (i+1, j) instead of (i, j+1)
if i + 1 < len(nums1):
    heapq.heappush(min_heap, (nums1[i+1] + nums2[j], i+1, j))
```

**Why wrong?** This moves down in matrix instead of right, causing duplicates or misses.

### ❌ Mistake 4: Not Checking j+1 Bounds
```python
# Wrong! May access out of bounds
heapq.heappush(min_heap, (nums1[i] + nums2[j+1], i, j+1))
```

**Fix:**
```python
if j + 1 < len(nums2):
    heapq.heappush(min_heap, (nums1[i] + nums2[j+1], i, j+1))
```

### ❌ Mistake 5: Returning Too Early
```python
# Wrong! Stops at k pairs even if heap not empty
while len(res) < k:  # Missing: and min_heap
    # ...
```

**Why wrong?** If heap becomes empty before k pairs, infinite loop or error.

**Fix:**
```python
while min_heap and len(res) < k:
    # ...
```

---

## Interview Insights

**Question Recognition:**
- "k smallest" + "two sorted arrays" + "pairs" → Min-heap exploration
- Similar to "merge k sorted lists" pattern

**Key Points to Mention:**

1. **Why not brute force:**
   - "Generating all m×n pairs is O(mn) which is too slow. We only need k pairs."

2. **Matrix analogy:**
   - "Think of all pairs as a sorted matrix. We explore it systematically using a heap, similar to merging sorted lists."

3. **Why initialize first column:**
   - "Starting with the first column ensures we have the smallest candidate from each row. As we expand right, we maintain sorted order."

4. **Optimization:**
   - "We limit initial heap to min(m, k) rows because we only need k pairs total."

**Follow-up Questions:**

1. **Q:** "What if arrays aren't sorted?"
   - **A:** Would need to sort them first: O(m log m + n log n) + O(k log k). Or use different approach.

2. **Q:** "Can you find k largest pairs instead?"
   - **A:** Yes, start from bottom-right, use max-heap, expand left and up.

3. **Q:** "What if we need all pairs with sum < target?"
   - **A:** Two pointers: start with (0, n-1), if sum < target, move i right; else move j left.

4. **Q:** "Space optimization?"
   - **A:** Current O(k) is optimal for heap approach. Alternative: Two pointers for specific queries.

5. **Q:** "What if k is very large (close to m×n)?"
   - **A:** If k > mn/2, might be faster to generate all and sort: O(mn log(mn)).

**Complexity Analysis:**
- Time: O(k log k) - k heap pops, each O(log k)
- Space: O(k) - Heap size at most k
- Optimal for k << m×n

---

## Related Problems

1. **Find K-th Smallest Pair Distance (LeetCode 719)** - Binary search + two pointers
2. **Kth Smallest Element in Sorted Matrix (LeetCode 378)** - Similar heap exploration
3. **Merge k Sorted Lists (LeetCode 23)** - Same heap pattern
4. **Smallest Range Covering Elements from K Lists (LeetCode 632)** - Multi-array heap
5. **Find the Kth Smallest Sum of a Matrix With Sorted Rows (LeetCode 1439)** - Extended version
6. **Super Ugly Number (LeetCode 313)** - Multiple pointers with heap

---

## Variations

### Variation 1: Find K Largest Pairs
```python
def kLargestPairs(self, nums1, nums2, k):
    """Find k pairs with largest sums."""
    if not nums1 or not nums2:
        return []
    
    # Start from bottom-right corner
    res = []
    max_heap = []
    
    m, n = len(nums1), len(nums2)
    
    # Initialize with last column
    for i in range(max(0, m - k), m):
        heapq.heappush(max_heap, (-(nums1[i] + nums2[n-1]), i, n-1))
    
    while max_heap and len(res) < k:
        curr_sum, i, j = heapq.heappop(max_heap)
        res.append([nums1[i], nums2[j]])
        
        # Move left
        if j - 1 >= 0:
            heapq.heappush(max_heap, (-(nums1[i] + nums2[j-1]), i, j-1))
    
    return res
```

### Variation 2: Find All Pairs with Sum Less Than Target
```python
def pairsLessThanTarget(self, nums1, nums2, target):
    """Find all pairs with sum < target."""
    res = []
    i, j = 0, len(nums2) - 1
    
    while i < len(nums1) and j >= 0:
        curr_sum = nums1[i] + nums2[j]
        
        if curr_sum < target:
            # All pairs (i, 0), (i, 1), ..., (i, j) satisfy
            for k in range(j + 1):
                res.append([nums1[i], nums2[k]])
            i += 1
        else:
            j -= 1
    
    return res
```

### Variation 3: With Visited Set (Both Directions)
```python
def kSmallestPairsBiDirectional(self, nums1, nums2, k):
    """
    Explore both right and down directions.
    More general but requires visited set.
    """
    if not nums1 or not nums2:
        return []
    
    min_heap = [(nums1[0] + nums2[0], 0, 0)]
    visited = {(0, 0)}
    res = []
    
    while min_heap and len(res) < k:
        curr_sum, i, j = heapq.heappop(min_heap)
        res.append([nums1[i], nums2[j]])
        
        # Explore right
        if j + 1 < len(nums2) and (i, j+1) not in visited:
            heapq.heappush(min_heap, (nums1[i] + nums2[j+1], i, j+1))
            visited.add((i, j+1))
        
        # Explore down
        if i + 1 < len(nums1) and (i+1, j) not in visited:
            heapq.heappush(min_heap, (nums1[i+1] + nums2[j], i+1, j))
            visited.add((i+1, j))
    
    return res
```

### Variation 4: Three Arrays
```python
def kSmallestTriples(self, nums1, nums2, nums3, k):
    """Find k triples with smallest sums from three arrays."""
    if not nums1 or not nums2 or not nums3:
        return []
    
    # First, find top k pairs from nums1 and nums2
    pairs = self.kSmallestPairs(nums1, nums2, k * len(nums3))
    
    # Flatten pairs into single array
    pair_sums = [p[0] + p[1] for p in pairs]
    
    # Now find k smallest from pair_sums and nums3
    result = []
    min_heap = []
    
    for i in range(min(len(pair_sums), k)):
        heapq.heappush(min_heap, 
            (pair_sums[i] + nums3[0], i, 0, pairs[i]))
    
    while min_heap and len(result) < k:
        curr_sum, i, j, pair = heapq.heappop(min_heap)
        result.append([pair[0], pair[1], nums3[j]])
        
        if j + 1 < len(nums3):
            heapq.heappush(min_heap, 
                (pair_sums[i] + nums3[j+1], i, j+1, pair))
    
    return result
```

---

## Summary

**Key Takeaways:**

1. **Matrix Exploration Pattern:**
   - Treat pairs as 2D matrix sorted by rows and columns
   - Use min-heap to explore cells in increasing order
   - Start with first column (smallest in each row)
   - Expand rightward only (simpler than bidirectional)

2. **Heap Initialization:**
   - Add first element from each row (first column)
   - Limit to min(m, k) rows for efficiency
   - Each row's minimum is its first element (j=0)

3. **Expansion Strategy:**
   - When popping (i,j), next candidate is (i, j+1)
   - Don't need (i+1, j) since already in heap from initialization
   - This avoids duplicates and visited tracking

4. **Optimization:**
   - Only initialize k rows (if m > k)
   - Heap never exceeds k elements
   - Only explore O(k) cells instead of O(mn)

5. **Complexity:**
   - Time: O(k log k) - Much better than O(mn log(mn))
   - Space: O(k) - Heap size
   - Optimal when k << m×n

**Template:**
```python
def kSmallestPairs(self, nums1, nums2, k):
    if not nums1 or not nums2:
        return []
    
    res = []
    min_heap = []
    
    # Initialize first column
    for i in range(min(len(nums1), k)):
        heapq.heappush(min_heap, (nums1[i] + nums2[0], i, 0))
    
    # Extract k smallest
    while min_heap and len(res) < k:
        curr_sum, i, j = heapq.heappop(min_heap)
        res.append([nums1[i], nums2[j]])
        
        if j + 1 < len(nums2):
            heapq.heappush(min_heap, (nums1[i] + nums2[j+1], i, j+1))
    
    return res
```

**Mental Model:**
- Imagine a sorted 2D grid of sums
- Start at top-left (guaranteed minimum)
- Use heap as frontier to explore next smallest
- Expand systematically to avoid revisiting
- Similar to Dijkstra's algorithm or merge k sorted lists
- This pattern generalizes to k-way merges and sorted matrix problems!
