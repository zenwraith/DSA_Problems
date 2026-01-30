# Sliding Window Maximum

**LeetCode 239** | **Difficulty:** Hard | **Category:** Sliding Window, Deque, Monotonic Queue

---

## Problem Statement

You are given an array of integers `nums`, there is a sliding window of size `k` which is moving from the very left of the array to the very right. You can only see the `k` numbers in the window. Each time the sliding window moves right by one position.

Return the **maximum** element in each sliding window.

---

## Examples

### Example 1:
```
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]

Explanation: 
Window position                Max
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

### Example 2:
```
Input: nums = [1], k = 1
Output: [1]
```

### Example 3:
```
Input: nums = [1,-1], k = 1
Output: [1,-1]
```

### Example 4:
```
Input: nums = [9,11], k = 2
Output: [11]
```

---

## Strategy

### The Monotonic Deque Approach

The key insight is using a **monotonic decreasing deque** (stores indices in decreasing order of their values).

**Why a Deque?**
- Need to track maximum element in current window
- Need to efficiently add elements to the right
- Need to efficiently remove elements from the left when they exit the window
- Need to remove elements that can never be the maximum

**Monotonic Deque Property:**
The deque stores **indices** in such a way that:
1. Elements are in **decreasing order** of their values
2. The **front** always contains the index of the maximum element in the current window
3. We remove smaller elements from the back because they can never be maximum if a larger element exists

**Algorithm:**

1. **For each element `nums[i]`:**
   - Remove elements from back while they're smaller than `nums[i]` (they'll never be max)
   - Add current index `i` to the back
   - Remove the front if it's outside the window (index < i - k + 1)
   - If we've processed at least `k` elements, record the maximum (front of deque)

**Why This Works:**
- **Monotonic property:** Ensures front is always the maximum
- **Remove from back:** Smaller elements behind a larger one are useless
- **Remove from front:** Elements outside the window are invalid

**Complexity:**
- **Time:** $O(n)$ - Each element is added and removed at most once
- **Space:** $O(k)$ - Deque stores at most k elements

---

## Solution

```python
from collections import deque
from typing import List

class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        dq = deque()  # Stores indices in decreasing order of values
        result = []
        
        for i in range(len(nums)):
            # Remove smaller elements from back (they can't be max)
            while dq and nums[dq[-1]] < nums[i]:
                dq.pop()
            
            # Add current index
            dq.append(i)
            
            # Remove elements outside the window from front
            if dq[0] == i - k:
                dq.popleft()
            
            # Once we have a full window, record the maximum
            if i >= k - 1:
                result.append(nums[dq[0]])
        
        return result
```

---

## Detailed Walkthrough

### Example: `nums = [1, 3, -1, -3, 5, 3, 6, 7]`, `k = 3`

**Visual Representation:**

| i | nums[i] | Deque (indices) | Deque (values) | Window | Max | Result | Action |
|---|---------|-----------------|----------------|--------|-----|--------|--------|
| 0 | 1 | [0] | [1] | [1] | - | [] | Add 0 |
| 1 | 3 | [1] | [3] | [1,3] | - | [] | Remove 0 (1<3), Add 1 |
| 2 | -1 | [1,2] | [3,-1] | [1,3,-1] | 3 | [3] | Add 2, Record max |
| 3 | -3 | [1,2,3] | [3,-1,-3] | [3,-1,-3] | 3 | [3,3] | Add 3, Record max |
| 4 | 5 | [4] | [5] | [-1,-3,5] | 5 | [3,3,5] | Remove all (smaller), Add 4, Record max |
| 5 | 3 | [4,5] | [5,3] | [-3,5,3] | 5 | [3,3,5,5] | Add 5, Record max |
| 6 | 6 | [6] | [6] | [5,3,6] | 6 | [3,3,5,5,6] | Remove all (smaller), Add 6, Record max |
| 7 | 7 | [7] | [7] | [3,6,7] | 7 | [3,3,5,5,6,7] | Remove all (smaller), Add 7, Record max |

**Detailed Step-by-Step:**

**Step 0 (i=0, nums[0]=1):**
- Deque empty, add index 0
- Not enough elements for window
- `dq = [0]`

**Step 1 (i=1, nums[1]=3):**
- Compare: nums[0]=1 < nums[1]=3, remove index 0
- Add index 1
- Not enough elements for window
- `dq = [1]`

**Step 2 (i=2, nums[2]=-1):**
- Compare: nums[1]=3 > nums[2]=-1, keep it
- Add index 2
- Window complete! Max = nums[1] = 3
- `dq = [1, 2]`, `result = [3]`

**Step 3 (i=3, nums[3]=-3):**
- Compare: nums[2]=-1 > nums[3]=-3, keep it
- Add index 3
- Check front: index 1 is still in window (3-3+1=1)
- Max = nums[1] = 3
- `dq = [1, 2, 3]`, `result = [3, 3]`

**Step 4 (i=4, nums[4]=5):**
- Compare: nums[3]=-3 < 5, remove index 3
- Compare: nums[2]=-1 < 5, remove index 2
- Compare: nums[1]=3 < 5, remove index 1
- Add index 4
- Max = nums[4] = 5
- `dq = [4]`, `result = [3, 3, 5]`

**Step 5 (i=5, nums[5]=3):**
- Compare: nums[4]=5 > 3, keep it
- Add index 5
- Max = nums[4] = 5
- `dq = [4, 5]`, `result = [3, 3, 5, 5]`

**Step 6 (i=6, nums[6]=6):**
- Compare: nums[5]=3 < 6, remove index 5
- Compare: nums[4]=5 < 6, remove index 4
- Add index 6
- Max = nums[6] = 6
- `dq = [6]`, `result = [3, 3, 5, 5, 6]`

**Step 7 (i=7, nums[7]=7):**
- Compare: nums[6]=6 < 7, remove index 6
- Add index 7
- Max = nums[7] = 7
- `dq = [7]`, `result = [3, 3, 5, 5, 6, 7]`

---

## Why Monotonic Decreasing?

### The Key Insight

**Question:** Why remove smaller elements from the back?

**Answer:** If `nums[i] > nums[j]` and `i > j`, then `nums[j]` can **never** be the maximum in any future window!

**Proof:**
- If both are in the window, `nums[i]` is larger
- When `nums[j]` leaves the window, `nums[i]` is still there (since i > j)
- Therefore, `nums[j]` is useless

**Example:**
```
nums = [1, 3, -1, 5]
k = 3

When we see 3:
  - 1 is now useless (3 > 1 and 3 comes after 1)
  - Remove 1 from deque

When we see 5:
  - 3 is useless (5 > 3)
  - -1 is useless (5 > -1)
  - Remove both
```

**Visual Example:**
```
Window: [1, 3, -1]
         ^  ^
         |  |
         |  +-- This is larger and will stay longer
         +-- This will never be max, remove it!

After removing 1, deque = [3, -1]
  - Front (3) is the maximum
  - -1 might be max in a future window (when 3 leaves)
```

---

## Why Not Use a Heap?

### Comparison with Max Heap

**Heap Approach:**
```python
import heapq

def maxSlidingWindow_heap(nums, k):
    heap = []
    result = []
    
    for i in range(len(nums)):
        heapq.heappush(heap, (-nums[i], i))
        
        # Remove elements outside window
        while heap and heap[0][1] <= i - k:
            heapq.heappop(heap)
        
        if i >= k - 1:
            result.append(-heap[0][0])
    
    return result
```

**Complexity:** $O(n \log n)$ - Each push/pop is $O(\log n)$

**Comparison:**

| Approach | Time | Space | Why |
|----------|------|-------|-----|
| **Monotonic Deque** | $O(n)$ | $O(k)$ | Each element added/removed once |
| **Max Heap** | $O(n \log n)$ | $O(n)$ | Heap operations are logarithmic |
| **Brute Force** | $O(n \times k)$ | $O(1)$ | Check max in each window |

**Winner:** Monotonic Deque is **faster and uses less space**!

---

## Edge Cases

### 1. Single Element Window
```python
nums = [1, 2, 3, 4], k = 1
# Output: [1, 2, 3, 4]
# Each element is its own maximum
```

### 2. Window Size Equals Array Length
```python
nums = [1, 3, 2, 5], k = 4
# Output: [5]
# Only one window, maximum is 5
```

### 3. Decreasing Array
```python
nums = [5, 4, 3, 2, 1], k = 3
# Output: [5, 4, 3]
# First element of each window is always max
# Deque only has one element at a time
```

### 4. Increasing Array
```python
nums = [1, 2, 3, 4, 5], k = 3
# Output: [3, 4, 5]
# Last element of each window is always max
# Deque clears and adds new element each time
```

### 5. All Same Elements
```python
nums = [3, 3, 3, 3], k = 2
# Output: [3, 3, 3]
# All maximums are 3
```

---

## Common Mistakes

### ❌ Mistake 1: Storing Values Instead of Indices
```python
# Wrong! Can't check if element is outside window
while dq and dq[-1] < nums[i]:
    dq.pop()
dq.append(nums[i])  # Storing value, not index
```

**Fix:** Store indices, access values via `nums[dq[0]]`

### ❌ Mistake 2: Wrong Window Boundary Check
```python
# Wrong! Should be ==, not <=
if dq[0] <= i - k:
    dq.popleft()
```

**Fix:**
```python
# Correct! Remove if exactly at boundary
if dq[0] == i - k:
    dq.popleft()
```

**Explanation:** If `dq[0] == i - k`, the front element is at position `i - k`, which is **just outside** the window `[i-k+1, i]`.

### ❌ Mistake 3: Not Using >= for First Result
```python
# Wrong! Should be >= k-1
if i == k - 1:
    result.append(nums[dq[0]])
```

**Fix:**
```python
# Correct! Process all windows from k-1 onwards
if i >= k - 1:
    result.append(nums[dq[0]])
```

### ❌ Mistake 4: Wrong Monotonic Condition
```python
# Wrong! Should be <, not <=
while dq and nums[dq[-1]] <= nums[i]:
    dq.pop()
```

**Why it matters:**
- Using `<=` removes equal elements
- This can cause issues when tracking indices
- Better to use `<` to maintain equal elements

---

## Interview Insights

**Question Recognition:**
- "Sliding window" + "maximum/minimum" + "each position" → Monotonic deque
- "Range queries" + "moving window" → Consider deque

**Key Points to Mention:**

1. **Why deque over other data structures?**
   - "A deque allows us to maintain a monotonic property with O(1) operations at both ends, giving us O(n) time overall."

2. **The monotonic invariant:**
   - "We maintain decreasing order so the front is always the maximum, and we remove smaller elements because they can never be the max once a larger element appears after them."

3. **Time complexity explanation:**
   - "Although we have a while loop inside the for loop, each element is added and removed from the deque at most once, giving us O(n) amortized time."

**Follow-up Questions:**

1. **Q:** "What if we want minimum instead of maximum?"
   - **A:** Use monotonic increasing deque instead (remove larger elements).

2. **Q:** "What if k = 1?"
   - **A:** Return the original array (each element is its own maximum).

3. **Q:** "Can you handle negative numbers?"
   - **A:** Yes, the algorithm works for any integers.

4. **Q:** "What about multiple queries with different k?"
   - **A:** Need to recompute for each k, or use a more complex data structure like segment tree.

**Optimization Discussion:**
- Already optimal at O(n) time
- Can't do better than O(n) since we need to examine each element

---

## Variations

### Variation 1: Sliding Window Minimum
```python
def minSlidingWindow(nums, k):
    dq = deque()
    result = []
    
    for i in range(len(nums)):
        # Monotonic increasing: remove larger elements
        while dq and nums[dq[-1]] > nums[i]:
            dq.pop()
        
        dq.append(i)
        
        if dq[0] == i - k:
            dq.popleft()
        
        if i >= k - 1:
            result.append(nums[dq[0]])
    
    return result
```

### Variation 2: Sliding Window Median (More Complex)
```python
# Requires two heaps (max heap and min heap)
# Or use two multisets in C++
# Time: O(n log k)
```

### Variation 3: First Negative in Every Window
```python
def firstNegativeInWindow(nums, k):
    dq = deque()
    result = []
    
    for i in range(len(nums)):
        # Store indices of negative numbers
        if nums[i] < 0:
            dq.append(i)
        
        # Remove elements outside window
        while dq and dq[0] <= i - k:
            dq.popleft()
        
        if i >= k - 1:
            if dq:
                result.append(nums[dq[0]])
            else:
                result.append(0)  # No negative in window
    
    return result
```

---

## Related Problems

1. **Min Stack (LeetCode 155)** - Similar monotonic property
2. **Shortest Subarray with Sum at Least K (LeetCode 862)** - Monotonic deque for prefix sums
3. **Longest Continuous Subarray With Absolute Diff <= Limit (LeetCode 1438)** - Two monotonic deques
4. **Jump Game VI (LeetCode 1696)** - DP with sliding window maximum
5. **Constrained Subsequence Sum (LeetCode 1425)** - Similar to Jump Game VI

---

## Summary

**Key Takeaways:**

1. **Data Structure:** Monotonic decreasing deque
   - Stores indices in decreasing order of values
   - Front always contains the maximum

2. **Core Operations:**
   - Remove smaller elements from back (they're useless)
   - Remove out-of-window elements from front
   - Add current index to back
   - Record maximum (front of deque)

3. **Why It's Optimal:**
   - Each element added once: O(n)
   - Each element removed once: O(n)
   - Total: O(n) amortized

4. **Complexity:**
   - Time: $O(n)$ - Linear scan with amortized constant operations
   - Space: $O(k)$ - Deque stores at most k elements

5. **Mental Model:** Think of the deque as a "sliding ranking board" where smaller elements behind larger ones are eliminated because they can never win.

**Template:**
```python
dq = deque()
result = []

for i in range(len(nums)):
    # Maintain monotonic property (remove smaller from back)
    while dq and nums[dq[-1]] < nums[i]:
        dq.pop()
    
    # Add current index
    dq.append(i)
    
    # Remove out-of-window indices from front
    if dq[0] == i - k:
        dq.popleft()
    
    # Record result once window is full
    if i >= k - 1:
        result.append(nums[dq[0]])

return result
```