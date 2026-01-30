# Find Median from Data Stream

**LeetCode 295** | **Difficulty:** Hard | **Category:** Heap, Design, Two Heaps

---

## Problem Statement

The **median** is the middle value in an ordered integer list. If the size of the list is even, there is no middle value, and the median is the mean of the two middle values.

- For example, for `arr = [2,3,4]`, the median is `3`.
- For example, for `arr = [2,3]`, the median is `(2 + 3) / 2 = 2.5`.

Implement the `MedianFinder` class:

- `MedianFinder()` initializes the `MedianFinder` object.
- `void addNum(int num)` adds the integer `num` from the data stream to the data structure.
- `double findMedian()` returns the median of all elements so far. Answers within `10^-5` of the actual answer will be accepted.

---

## Examples

### Example 1:
```
Input:
["MedianFinder", "addNum", "addNum", "findMedian", "addNum", "findMedian"]
[[], [1], [2], [], [3], []]

Output:
[null, null, null, 1.5, null, 2.0]

Explanation:
MedianFinder medianFinder = new MedianFinder();
medianFinder.addNum(1);    // arr = [1]
medianFinder.addNum(2);    // arr = [1, 2]
medianFinder.findMedian(); // return 1.5 (i.e., (1 + 2) / 2)
medianFinder.addNum(3);    // arr = [1, 2, 3]
medianFinder.findMedian(); // return 2.0
```

### Example 2:
```
Input:
["MedianFinder", "addNum", "findMedian", "addNum", "findMedian"]
[[], [5], [], [10], []]

Output:
[null, null, 5.0, null, 7.5]

Explanation:
medianFinder.addNum(5);    // arr = [5]
medianFinder.findMedian(); // return 5.0
medianFinder.addNum(10);   // arr = [5, 10]
medianFinder.findMedian(); // return 7.5
```

### Example 3:
```
Input:
["MedianFinder", "addNum", "addNum", "addNum", "findMedian"]
[[], [1], [3], [2], []]

Output:
[null, null, null, null, 2.0]

Explanation:
medianFinder.addNum(1);    // arr = [1]
medianFinder.addNum(3);    // arr = [1, 3]
medianFinder.addNum(2);    // arr = [1, 2, 3] (sorted)
medianFinder.findMedian(); // return 2.0
```

---

## Strategy

### Two Heaps: Max-Heap + Min-Heap

**Key Insight:** Use two heaps to maintain the lower and upper halves of the data stream, allowing O(1) median access.

**Why This Works:**

```
Goal: Find median efficiently without sorting every time

Observation: Median is in the "middle"
- For sorted array [1,2,3,4,5], median = 3 (middle element)
- For [1,2,3,4], median = (2+3)/2 (average of two middle)

Idea: Split numbers into two halves
- Small half: Contains smaller half of numbers (max-heap)
- Large half: Contains larger half of numbers (min-heap)

If we maintain:
1. All elements in small ≤ all elements in large
2. Heaps differ in size by at most 1

Then:
- Median is at the top of one (or both) heaps!
- O(1) access to median
- O(log n) insertion
```

**Data Structure Design:**

```python
small = []  # Max-heap (use negative values for Python's min-heap)
            # Contains smaller half of numbers
            # Top = largest of small half

large = []  # Min-heap
            # Contains larger half of numbers
            # Top = smallest of large half

Invariants:
1. max(small) ≤ min(large)  (small[0] ≤ large[0])
2. |len(small) - len(large)| ≤ 1  (balanced sizes)

Example: [1, 2, 3, 4, 5]
  small: [3, 2, 1]  (max-heap, top=3)
  large: [4, 5]     (min-heap, top=4)
  Median: 3 (from small since it has more elements)

Example: [1, 2, 3, 4]
  small: [2, 1]     (max-heap, top=2)
  large: [3, 4]     (min-heap, top=3)
  Median: (2 + 3) / 2 = 2.5
```

**Algorithm Steps:**

**addNum(num):**
1. Always add to `small` first (establishes default flow)
2. **Balance values:** If max(small) > min(large), move top of small to large
3. **Balance sizes:** If heaps differ by > 1, rebalance

**findMedian():**
- If `small` has more: return top of `small`
- If `large` has more: return top of `large`
- If equal sizes: return average of both tops

**Complexity:**
- **addNum:** $O(\log n)$ - Heap operations
- **findMedian:** $O(1)$ - Just peek at heap tops
- **Space:** $O(n)$ - Store all numbers

---

## Solution

```python
import heapq

class MedianFinder:
    """
    Find median from data stream using two heaps.
    
    Strategy: Maintain two heaps
    - small: Max-heap (negated) for smaller half
    - large: Min-heap for larger half
    
    Invariants:
    1. All elements in small ≤ all elements in large
    2. |len(small) - len(large)| ≤ 1
    
    This allows O(1) median access and O(log n) insertion.
    """

    def __init__(self):
        """Initialize two heaps."""
        # Max-heap for smaller half (use negative values)
        self.small = []
        # Min-heap for larger half
        self.large = []

    def addNum(self, num: int) -> None:
        """
        Add a number to the data structure.
        
        Process:
        1. Add to small (default path)
        2. Balance values: ensure max(small) ≤ min(large)
        3. Balance sizes: ensure heaps differ by at most 1
        
        Time: O(log n)
        """
        # Step 1: Add to small heap (as negative for max-heap behavior)
        heapq.heappush(self.small, -num)
        
        # Step 2: Balance values
        # If max of small > min of large, move it
        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        
        # Step 3: Balance sizes
        # Ensure heaps differ by at most 1
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        
        if len(self.large) > len(self.small) + 1:
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)

    def findMedian(self) -> float:
        """
        Return the median of all elements.
        
        Time: O(1)
        """
        # If small has more elements, median is its top
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        
        # If large has more elements, median is its top
        elif len(self.large) > len(self.small):
            return float(self.large[0])
        
        # If equal size, median is average of both tops
        else:
            return (-self.small[0] + self.large[0]) / 2.0


# Usage
# obj = MedianFinder()
# obj.addNum(num)
# param_2 = obj.findMedian()
```

---

## Detailed Walkthrough

### Example 1: `addNum(1), addNum(2), findMedian(), addNum(3), findMedian()`

**Step 1: addNum(1)**

| Action | Details |
|--------|---------|
| Add to small | Push -1 to small |
| small | [-1] (represents max-heap with max=1) |
| large | [] |
| Balance values | No large elements yet, skip |
| Balance sizes | len(small)=1, len(large)=0, diff=1 ✓ |

State: `small=[-1], large=[]`

**Step 2: addNum(2)**

| Action | Details |
|--------|---------|
| Add to small | Push -2 to small |
| small | [-2, -1] (max=2) |
| large | [] |
| Balance values | -small[0]=-(-2)=2, large empty, skip check |
| Balance sizes | len(small)=2, len(large)=0, diff=2 > 1! |
| Rebalance | Pop from small: -(-2)=2, push to large |
| Final | small=[-1], large=[2] |

State: `small=[-1], large=[2]`

**Step 3: findMedian()**

| Heap | Size | Details |
|------|------|---------|
| small | 1 | Max = 1 |
| large | 1 | Min = 2 |
| Same size | Yes | Median = (1 + 2) / 2 = 1.5 |

**Output:** `1.5`

**Step 4: addNum(3)**

| Action | Details |
|--------|---------|
| Add to small | Push -3 to small |
| small | [-3, -1] (max=3) |
| large | [2] (min=2) |
| Balance values | -small[0]=3 > large[0]=2? Yes! |
| Move | Pop 3 from small, push to large |
| After value balance | small=[-1], large=[2,3] |
| Balance sizes | len(small)=1, len(large)=2, diff=1 ✓ |

State: `small=[-1], large=[2,3]`

**Step 5: findMedian()**

| Heap | Size | Details |
|------|------|---------|
| small | 1 | Max = 1 |
| large | 2 | Min = 2 |
| large has more | Yes | Median = 2 (top of large) |

**Output:** `2.0`

---

### Example 2: Stream `[5, 10, 3, 15, 2]`

| Operation | small (max-heap) | large (min-heap) | Median |
|-----------|------------------|------------------|--------|
| addNum(5) | [5] | [] | 5.0 |
| addNum(10) | [5] | [10] | 7.5 |
| addNum(3) | [5, 3] | [10] | 5.0 |
| addNum(15) | [5, 3] | [10, 15] | 7.5 |
| addNum(2) | [5, 3, 2] | [10, 15] | 5.0 |

**Detailed trace for addNum(3) after [5, 10]:**

```
Initial: small=[5], large=[10]

1. Add -3 to small: small=[-5, -3]  (heap: [-5, -3], max=5)
   
2. Balance values: max(small)=5, min(large)=10
   5 > 10? No, OK
   
3. Balance sizes: len(small)=2, len(large)=1
   2 > 1+1? No, OK

Final: small=[5, 3], large=[10]
Median: len(small)=2 > len(large)=1
        Return max(small) = 5
```

**Detailed trace for addNum(15) after [5, 3, 10]:**

```
Initial: small=[5, 3], large=[10]

1. Add -15 to small: small=[-15, -5, -3]  (max=15)
   
2. Balance values: max(small)=15, min(large)=10
   15 > 10? Yes! Move 15 to large
   Pop from small: 15
   Push to large: [10, 15] (min-heap, min=10)
   small=[5, 3]
   
3. Balance sizes: len(small)=2, len(large)=2
   Equal, OK

Final: small=[5, 3], large=[10, 15]
Median: Equal sizes
        Return (5 + 10) / 2 = 7.5
```

---

## Why This Algorithm Works

### The Two-Heap Invariant

**Invariant 1: Value Ordering**
```
All elements in small ≤ All elements in large

This means:
  max(small) ≤ min(large)

Visualization:
  small: [..., ..., MAX]
  large: [MIN, ..., ...]
         ↑         ↑
    These two are adjacent in sorted order
```

**Invariant 2: Size Balance**
```
|len(small) - len(large)| ≤ 1

Allowed states:
- len(small) = len(large)      (even total)
- len(small) = len(large) + 1  (odd total, small has extra)
- len(large) = len(small) + 1  (odd total, large has extra)

Not allowed:
- len(small) = len(large) + 2  (too unbalanced)
```

**Why These Invariants Give Us Median:**

```
Case 1: Even total (equal heap sizes)
  Sorted: [... small elements ... | ... large elements ...]
  Median: Average of max(small) and min(large)
  
  Example: [1, 2 | 3, 4]
           small  large
  Median: (2 + 3) / 2 = 2.5

Case 2: Odd total (one heap has +1)
  The extra element IS the median
  
  Example: [1, 2, 3 | 4, 5]
           small (3 elements)
  Median: max(small) = 3
  
  Or: [1, 2 | 3, 4, 5]
       small  large (3 elements)
  Median: min(large) = 3
```

### Why Always Add to Small First?

```
Design choice: Establish consistent flow

Alternative 1: Always add to small first
  ✓ Simple rule
  ✓ Fewer edge cases
  ✓ Balance checks fix any issues

Alternative 2: Add to correct heap directly
  ✗ Need to check if heaps are initialized
  ✗ More complex logic
  ✗ More edge cases

By always adding to small first, we simplify the logic
and rely on balance steps to maintain invariants.
```

---

## Edge Cases

### 1. Single Element
```python
mf = MedianFinder()
mf.addNum(5)
mf.findMedian()  # 5.0
# small=[5], large=[]
```

### 2. Two Elements
```python
mf = MedianFinder()
mf.addNum(5)
mf.addNum(10)
mf.findMedian()  # 7.5
# small=[5], large=[10]
```

### 3. Descending Order
```python
mf = MedianFinder()
mf.addNum(5)  # small=[5], large=[]
mf.addNum(4)  # small=[5,4], large=[] → small=[4], large=[5]
mf.addNum(3)  # small=[4,3], large=[5]
mf.findMedian()  # 4.0
```

### 4. Ascending Order
```python
mf = MedianFinder()
mf.addNum(1)  # small=[1], large=[]
mf.addNum(2)  # small=[1], large=[2]
mf.addNum(3)  # small=[1,2], large=[3] → small=[1], large=[2,3]
mf.findMedian()  # 2.0
```

### 5. Duplicate Values
```python
mf = MedianFinder()
mf.addNum(5)
mf.addNum(5)
mf.addNum(5)
mf.findMedian()  # 5.0
# Duplicates handled correctly
```

### 6. Negative Numbers
```python
mf = MedianFinder()
mf.addNum(-1)
mf.addNum(-2)
mf.addNum(-3)
mf.findMedian()  # -2.0
# Works with negatives (we negate again, so -(-2)=2)
```

---

## Common Mistakes

### ❌ Mistake 1: Using Single Heap
```python
# Wrong! Sorting every time
class MedianFinder:
    def __init__(self):
        self.heap = []
    
    def addNum(self, num):
        heapq.heappush(self.heap, num)
    
    def findMedian(self):
        sorted_nums = sorted(self.heap)  # O(n log n) every time!
        n = len(sorted_nums)
        if n % 2 == 1:
            return sorted_nums[n // 2]
        else:
            return (sorted_nums[n // 2 - 1] + sorted_nums[n // 2]) / 2
```

**Issues:** O(n log n) median access, defeats purpose

### ❌ Mistake 2: Forgetting to Negate for Max-Heap
```python
# Wrong! small is min-heap, not max-heap
heapq.heappush(self.small, num)  # Should be -num
```

**Fix:**
```python
heapq.heappush(self.small, -num)  # Negate for max-heap
```

### ❌ Mistake 3: Wrong Balance Order
```python
# Wrong! Balancing sizes before balancing values
def addNum(self, num):
    heapq.heappush(self.small, -num)
    
    # Size balance first (WRONG ORDER)
    if len(self.small) > len(self.large) + 1:
        val = -heapq.heappop(self.small)
        heapq.heappush(self.large, val)
    
    # Value balance second (TOO LATE)
    if self.small and self.large and -self.small[0] > self.large[0]:
        val = -heapq.heappop(self.small)
        heapq.heappush(self.large, val)
```

**Fix:** Balance values BEFORE balancing sizes
```python
def addNum(self, num):
    heapq.heappush(self.small, -num)
    
    # Value balance FIRST
    if self.small and self.large and -self.small[0] > self.large[0]:
        val = -heapq.heappop(self.small)
        heapq.heappush(self.large, val)
    
    # Size balance SECOND
    if len(self.small) > len(self.large) + 1:
        val = -heapq.heappop(self.small)
        heapq.heappush(self.large, val)
```

### ❌ Mistake 4: Not Checking Empty Heaps
```python
# Wrong! Accessing heap without checking if empty
if -self.small[0] > self.large[0]:  # Crashes if either empty
    # ...
```

**Fix:**
```python
if self.small and self.large and -self.small[0] > self.large[0]:
    # ...
```

### ❌ Mistake 5: Wrong Median Calculation
```python
# Wrong! Not handling even case correctly
def findMedian(self):
    if len(self.small) > len(self.large):
        return -self.small[0]
    else:  # Wrong: doesn't distinguish equal from large>small
        return self.large[0]
```

**Fix:**
```python
def findMedian(self):
    if len(self.small) > len(self.large):
        return float(-self.small[0])
    elif len(self.large) > len(self.small):
        return float(self.large[0])
    else:  # Equal sizes
        return (-self.small[0] + self.large[0]) / 2.0
```

---

## Interview Insights

**Question Recognition:**
- "Data stream" + "median" + "efficient" → Two heaps
- Need O(1) or O(log n) median access, not O(n)

**Key Points to Mention:**

1. **Why two heaps:**
   - "We split the data into two halves. Max-heap stores smaller half, min-heap stores larger half. This keeps median at the boundary."

2. **Invariants:**
   - "Two invariants: value ordering (max(small) ≤ min(large)) and size balance (differ by at most 1)."

3. **Time complexity:**
   - "addNum is O(log n) for heap operations. findMedian is O(1) since we just peek at heap tops."

4. **Space complexity:**
   - "O(n) to store all numbers in the two heaps."

**Follow-up Questions:**

1. **Q:** "What if 99% of queries are findMedian?"
   - **A:** This solution is optimal since findMedian is O(1). Can't do better.

2. **Q:** "What if the data is very large and doesn't fit in memory?"
   - **A:** External sorting or streaming algorithms. Could use approximation algorithms like reservoir sampling.

3. **Q:** "Can you find other percentiles (e.g., 75th percentile)?"
   - **A:** Yes, adjust heap sizes. For kth percentile, maintain k% in small, (100-k)% in large.

4. **Q:** "What if we also need to remove elements?"
   - **A:** More complex. Need lazy deletion with counters, or use TreeSet/balanced BST (O(log n) but more overhead).

5. **Q:** "What if numbers come in sorted order?"
   - **A:** Still works! Rebalancing ensures both heaps are used. Worst case for heap operations is still O(log n).

**Complexity Analysis:**
- addNum: O(log n) - Up to 3 heap operations
- findMedian: O(1) - Just peek
- Space: O(n) - Store all numbers

---

## Related Problems

1. **Sliding Window Median (LeetCode 480)** - Median in moving window (harder, with deletions)
2. **IPO (LeetCode 502)** - Two heaps for project selection
3. **Kth Largest Element in Stream (LeetCode 703)** - Similar but simpler (one heap)
4. **Top K Frequent Elements (LeetCode 347)** - Heap for top-k tracking
5. **Find K Pairs with Smallest Sums (LeetCode 373)** - Multi-heap problem
6. **Online Median (GeeksforGeeks)** - Same problem

---

## Variations

### Variation 1: Support Remove Operation
```python
class MedianFinderWithRemove:
    """Support removing elements (lazy deletion)."""
    
    def __init__(self):
        self.small = []
        self.large = []
        self.small_count = {}  # Track invalid elements
        self.large_count = {}
    
    def addNum(self, num):
        # Same as before
        heapq.heappush(self.small, -num)
        
        if self.small and self.large and -self.small[0] > self.large[0]:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        
        # Balance sizes...
    
    def removeNum(self, num):
        """Mark number for removal."""
        if num <= -self.small[0]:
            self.small_count[num] = self.small_count.get(num, 0) + 1
        else:
            self.large_count[num] = self.large_count.get(num, 0) + 1
        
        # Clean up tops if needed
        self._clean_small()
        self._clean_large()
    
    def _clean_small(self):
        """Remove invalid elements from small top."""
        while self.small and self.small_count.get(-self.small[0], 0) > 0:
            val = -heapq.heappop(self.small)
            self.small_count[val] -= 1
    
    def _clean_large(self):
        """Remove invalid elements from large top."""
        while self.large and self.large_count.get(self.large[0], 0) > 0:
            val = heapq.heappop(self.large)
            self.large_count[val] -= 1
```

### Variation 2: Find kth Percentile
```python
class PercentileFinder:
    """Find any percentile, not just median."""
    
    def __init__(self, k):
        """k is percentile (0-100)."""
        self.k = k / 100.0  # Convert to ratio
        self.small = []  # Should contain k% of elements
        self.large = []  # Should contain (1-k)% of elements
    
    def addNum(self, num):
        heapq.heappush(self.small, -num)
        
        if self.small and self.large and -self.small[0] > self.large[0]:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        
        # Balance to maintain k% in small
        total = len(self.small) + len(self.large)
        target_small = int(total * self.k)
        
        while len(self.small) > target_small + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        
        while len(self.small) < target_small:
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)
    
    def findPercentile(self):
        """Return kth percentile value."""
        return float(-self.small[0]) if self.small else 0.0
```

### Variation 3: Sliding Window Median
```python
class SlidingWindowMedian:
    """Maintain median over sliding window."""
    
    def medianSlidingWindow(self, nums, k):
        """Return median for each window of size k."""
        result = []
        mf = MedianFinder()
        
        # Initialize first window
        for i in range(k):
            mf.addNum(nums[i])
        result.append(mf.findMedian())
        
        # Slide window (requires remove support)
        for i in range(k, len(nums)):
            mf.removeNum(nums[i - k])  # Remove oldest
            mf.addNum(nums[i])          # Add newest
            result.append(mf.findMedian())
        
        return result
```

---

## Summary

**Key Takeaways:**

1. **Two-Heap Pattern:**
   - Split data into two halves
   - Max-heap for smaller half (easy access to largest small value)
   - Min-heap for larger half (easy access to smallest large value)
   - Median is at the boundary between heaps

2. **Maintain Invariants:**
   - Value ordering: max(small) ≤ min(large)
   - Size balance: heaps differ by at most 1
   - These two invariants guarantee O(1) median access

3. **Balance Order Matters:**
   - Always balance VALUES before balancing SIZES
   - Value balance ensures correctness
   - Size balance ensures O(1) median access

4. **Max-Heap in Python:**
   - Python only has min-heap
   - Simulate max-heap by negating values
   - Push -x, pop and negate to get max

5. **Optimal Complexity:**
   - addNum: O(log n) - Heap operations
   - findMedian: O(1) - Peek at tops
   - Can't do better for both operations

**Template:**
```python
class MedianFinder:
    def __init__(self):
        self.small = []  # Max-heap (negated)
        self.large = []  # Min-heap
    
    def addNum(self, num):
        heapq.heappush(self.small, -num)
        
        # Balance values
        if self.small and self.large and -self.small[0] > self.large[0]:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        
        # Balance sizes
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        if len(self.large) > len(self.small) + 1:
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)
    
    def findMedian(self):
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        elif len(self.large) > len(self.small):
            return float(self.large[0])
        else:
            return (-self.small[0] + self.large[0]) / 2.0
```

**Mental Model:**
- Imagine sorting numbers as they arrive
- Two heaps represent left and right halves of sorted array
- Boundary between heaps is always the median(s)
- Rebalancing maintains this boundary as numbers are added
- This is a classic design pattern for streaming statistics!
