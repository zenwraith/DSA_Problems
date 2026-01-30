# Next Greater Element II

**LeetCode 503** | **Difficulty:** Medium | **Category:** Stack, Array, Monotonic Stack

---

## Problem Statement

Given a circular integer array `nums` (i.e., the next element of `nums[nums.length - 1]` is `nums[0]`), return the **next greater number** for every element in `nums`.

The next greater number of a number `x` is the first greater number to its traversing-order next in the array, which means you could search circularly to find its next greater number. If it doesn't exist, return `-1` for this number.

**Constraints:**
- `1 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`

---

## Examples

### Example 1:
```
Input: nums = [1,2,1]
Output: [2,-1,2]

Explanation:
- First 1's next greater number is 2
- Number 2 can't find next greater number (largest)
- Second 1's next greater number is 2 (wraps around to first element)
```

### Example 2:
```
Input: nums = [1,2,3,4,3]
Output: [2,3,4,-1,4]

Explanation:
- 1 → 2 (next element)
- 2 → 3 (next element)
- 3 → 4 (next element)
- 4 → -1 (largest element, no greater)
- 3 → 4 (wraps around to find 4)
```

### Example 3:
```
Input: nums = [5,4,3,2,1]
Output: [-1,5,5,5,5]

Explanation:
- 5 → -1 (largest)
- 4 → 5 (wraps to beginning)
- 3 → 5 (wraps to beginning)
- 2 → 5 (wraps to beginning)
- 1 → 5 (wraps to beginning)
```

---

## Strategy

### Monotonic Stack with Circular Array Processing

**Key Insight:** Process the array twice using modulo arithmetic to simulate circular behavior, but only store indices from the first pass.

**Why This Works:**

```
Challenge: Array is circular
  - Last element's next could be first element
  - Need to check "wrap around" for each element
  - Naive approach: For each element, scan right and wrap around (O(n²))

Better Approach: Process array twice
  - First pass: Find NGE using elements to the right
  - Second pass: Find NGE by wrapping to beginning
  - Use modulo (i % n) to access circular indices
  - Only store indices from first pass to avoid duplicates

Key Trick: Process 2*n elements
  - Iterate from i = 0 to 2*n-1
  - Access element: nums[i % n]
  - Only push indices when i < n (first pass only)
  
Why This Works:
  - Elements in second pass can pop elements from stack
  - This handles wrap-around cases
  - But we don't push indices from second pass (avoid duplicates)
  - Result array already initialized to -1 (handles no NGE case)
```

**Algorithm Steps:**

1. **Initialize:**
   - Result array with -1 (default for no NGE)
   - Empty monotonic stack (stores indices)

2. **Process 2*n elements:**
   - Use i % n to get circular index
   - While stack not empty and current > stack top:
     - Pop index and record its NGE as current element
   - Only push index if i < n (first pass)

3. **Return result**

**Why Process Twice:**

```
Example: [5, 4, 3, 2, 1]

First pass (i = 0 to 4):
  - 5: stack = [0]
  - 4 < 5: stack = [0, 1]
  - 3 < 4: stack = [0, 1, 2]
  - 2 < 3: stack = [0, 1, 2, 3]
  - 1 < 2: stack = [0, 1, 2, 3, 4]
  
No NGE found yet! All elements in decreasing order.

Second pass (i = 5 to 9, accessing i%5):
  - i=5: nums[0]=5, 5 > 1: pop 4, res[4] = 5
  - 5 > 2: pop 3, res[3] = 5
  - 5 > 3: pop 2, res[2] = 5
  - 5 > 4: pop 1, res[1] = 5
  - 5 = 5: stack = [0] (don't pop equal)
  
Now we found NGE for all except the largest (5)!
```

**Complexity:**
- **Time:** $O(n)$ - Each element pushed/popped at most once
- **Space:** $O(n)$ - Stack and result array

---

## Solution

```python
class Solution:
    def nextGreaterElements(self, nums: List[int]) -> List[int]:
        """
        Find next greater element in circular array.
        
        Strategy: Monotonic stack with two-pass simulation
        1. Process array twice (2*n iterations) using modulo
        2. First pass (i < n): Push indices to stack
        3. Second pass (i >= n): Only pop, don't push (handles wrap)
        4. Use modulo i%n to access circular indices
        
        Key insight: Processing twice simulates circular behavior
        - Elements can find their NGE by wrapping to start
        - Only push in first pass to avoid duplicate processing
        
        Edge cases:
        - All same: All return -1
        - Strictly decreasing: Need second pass for wrap-around
        - Single element: Returns [-1]
        - Largest element: No NGE, stays -1
        
        Args:
            nums: Circular integer array
            
        Returns:
            Array of next greater elements (or -1)
        """
        n = len(nums)
        
        # Initialize result with -1 (default when no NGE exists)
        res = [-1] * n
        
        # Monotonic decreasing stack (stores indices)
        stack = []
        
        # Process array twice to handle circular nature
        for i in range(2 * n):
            # Get circular index
            num = nums[i % n]
            
            # Pop all smaller elements (they found their NGE)
            while stack and nums[stack[-1]] < num:
                idx = stack.pop()
                res[idx] = num
            
            # Only push indices from first pass
            # (Second pass just helps find NGE for first pass elements)
            if i < n:
                stack.append(i)
        
        # Elements still in stack have no NGE (remain -1)
        return res
```

---

## Detailed Walkthrough

### Example 1: `nums = [1,2,1]`

**Iteration Details:**

| i | i%n | num | stack (before) | Action | res | stack (after) |
|---|-----|-----|----------------|--------|-----|---------------|
| 0 | 0 | 1 | [] | Push 0 | [-1,-1,-1] | [0] |
| 1 | 1 | 2 | [0] | 2>1: pop 0, res[0]=2, push 1 | [2,-1,-1] | [1] |
| 2 | 2 | 1 | [1] | 1<2: push 2 | [2,-1,-1] | [1,2] |
| 3 | 0 | 1 | [1,2] | 1<2: no pops, no push (i≥n) | [2,-1,-1] | [1,2] |
| 4 | 1 | 2 | [1,2] | 2>1: pop 2, res[2]=2 | [2,-1,2] | [1] |
| 5 | 2 | 1 | [1] | 1<2: no pops, no push | [2,-1,2] | [1] |

**Final:** `res = [2, -1, 2]`

**Visualization:**
```
Array: [1, 2, 1]
        ↓  ↓  ↓
NGE:    2 -1  2

Index 0 (value 1): Next greater is 2 at index 1
Index 1 (value 2): No next greater (largest)
Index 2 (value 1): Next greater is 2 (wraps to index 1)
                   Found in second pass when i=4, accessing nums[1]
```

---

### Example 2: `nums = [1,2,3,4,3]`

**First Pass (i = 0 to 4):**

```
i=0: nums[0]=1, stack=[] → push 0 → stack=[0]
i=1: nums[1]=2, 2>1 → pop 0, res[0]=2 → push 1 → stack=[1]
i=2: nums[2]=3, 3>2 → pop 1, res[1]=3 → push 2 → stack=[2]
i=3: nums[3]=4, 4>3 → pop 2, res[2]=4 → push 3 → stack=[3]
i=4: nums[4]=3, 3<4 → push 4 → stack=[3,4]

After first pass:
res = [2, 3, 4, -1, -1]
stack = [3, 4]  (indices 3 and 4 haven't found NGE yet)
```

**Second Pass (i = 5 to 9):**

```
i=5: nums[0]=1, 1<3 → no pops, no push → stack=[3,4]
i=6: nums[1]=2, 2<3 → no pops, no push → stack=[3,4]
i=7: nums[2]=3, 3=3 → no pops, no push → stack=[3,4]
i=8: nums[3]=4, 4>3 → pop 4, res[4]=4 → no push → stack=[3]
i=9: nums[4]=3, 3<4 → no pops, no push → stack=[3]

Final:
res = [2, 3, 4, -1, 4]
stack = [3]  (index 3, value 4, is largest - no NGE)
```

**Output:** `[2, 3, 4, -1, 4]` ✓

---

### Example 3: `nums = [5,4,3,2,1]` (Strictly Decreasing)

**First Pass:**

```
All elements in decreasing order:
i=0: stack=[0]
i=1: 4<5, stack=[0,1]
i=2: 3<4, stack=[0,1,2]
i=3: 2<3, stack=[0,1,2,3]
i=4: 1<2, stack=[0,1,2,3,4]

res = [-1,-1,-1,-1,-1]
Stack full! No NGE found yet.
```

**Second Pass (Critical for decreasing arrays):**

```
i=5: nums[0]=5
  - 5 > 1 → pop 4, res[4]=5
  - 5 > 2 → pop 3, res[3]=5
  - 5 > 3 → pop 2, res[2]=5
  - 5 > 4 → pop 1, res[1]=5
  - 5 = 5 → stop (don't pop equal)
  - stack=[0]

i=6 to 9: nums[1] to nums[4] all < 5
  - No more pops
  
Final:
res = [-1, 5, 5, 5, 5]
stack = [0]  (index 0, value 5, is largest)
```

**Output:** `[-1, 5, 5, 5, 5]` ✓

**Key Observation:** Strictly decreasing arrays NEED the second pass!

---

## Why This Algorithm Works

### Circular Array Simulation

**Why process 2*n elements?**

```
Consider: [3, 1, 2]

Normal linear processing:
  i=0: 3, stack=[0]
  i=1: 1<3, stack=[0,1]
  i=2: 2>1, pop 1, res[1]=2, stack=[0,2]
  
Result: [-1, 2, -1]  Wrong! 2 should see 3.

With circular processing (i=0 to 5):
  First pass (i=0,1,2): Same as above
  
  Second pass:
  i=3: nums[0]=3, 3>2 → pop 2, res[2]=3, stack=[0]
  i=4: nums[1]=1<3, no pops
  i=5: nums[2]=2<3, no pops
  
Result: [-1, 2, 3]  Correct!

The second pass allows elements to find NGE at the beginning.
```

### Why Only Push in First Pass?

```
Question: Why not push indices in second pass?

Answer: To avoid duplicates and infinite loops.

If we pushed in second pass:
  i=5 (nums[0]): Would push index 0 again
  Now we have duplicates in stack!
  
Also, we don't need to:
  - First pass elements already in stack
  - Second pass only helps them find NGE
  - We're not looking for NGE of second pass elements
    (they're the same elements as first pass!)
```

### Stack Invariant

```
Throughout the algorithm:
1. Stack stores indices (not values)
2. Stack is monotonic decreasing (by values)
3. All indices in stack are from range [0, n-1]
4. Elements in stack haven't found NGE yet

After processing:
- Popped elements: Found their NGE (recorded in res)
- Remaining elements: No NGE exists (res already -1)
```

---

## Edge Cases

### 1. All Same Elements
```python
nums = [3, 3, 3, 3]
# No element is greater than any other
# Output: [-1, -1, -1, -1]
# Stack: [0,1,2,3] (no pops during both passes)
```

### 2. Single Element
```python
nums = [1]
# Only one element, can't be greater than itself
# Output: [-1]
# Process i=0,1: nums[0]=1 twice, but no greater
```

### 3. Two Elements Increasing
```python
nums = [1, 2]
# First pass: 1→2, stack=[1]
# Second pass: i=2 (nums[0]=1), 1<2, no pops
# Output: [2, -1]
```

### 4. Two Elements Decreasing
```python
nums = [2, 1]
# First pass: stack=[0,1]
# Second pass: i=2 (nums[0]=2), 2>1, pop 1, res[1]=2
#              i=3 (nums[1]=1), 1<2, no pops
# Output: [-1, 2]  (Wraps around!)
```

### 5. All Increasing
```python
nums = [1, 2, 3, 4, 5]
# First pass: Each pops previous immediately
# res = [2, 3, 4, 5, -1]
# Second pass: No changes (already optimal)
# Output: [2, 3, 4, 5, -1]
```

### 6. Mountain (Peak in Middle)
```python
nums = [1, 3, 5, 3, 1]
# First pass: [3, 5, -1, 5, -1]
# Index 2 (value 5): Largest, no NGE
# Second pass: Index 4→1, finds 3 at wrap? No, 1<3 in stack
# Actually: res = [3, 5, -1, 5, 3]
# Let me trace:
#   i=0: push 0, stack=[0]
#   i=1: 3>1, pop 0, res[0]=3, push 1, stack=[1]
#   i=2: 5>3, pop 1, res[1]=5, push 2, stack=[2]
#   i=3: 3<5, push 3, stack=[2,3]
#   i=4: 1<3, push 4, stack=[2,3,4]
#   i=5: nums[0]=1<5, no pops
#   i=6: nums[1]=3<5, no pops
#   i=7: nums[2]=5>1, pop 4, res[4]=5
#                     5>3, pop 3, res[3]=5
#                     5=5, stop
# Output: [3, 5, -1, 5, 5]
```

### 7. Large Array with One Maximum
```python
nums = [1,2,3,4,5,6,7,8,9,10]
# Output: [2,3,4,5,6,7,8,9,10,-1]
# Only last element has no NGE
```

### 8. Alternating Pattern
```python
nums = [3, 1, 4, 1, 5]
# Output: [4, 4, 5, 5, -1]
# Each element finds something larger ahead or wrapped
```

---

## Common Mistakes

### ❌ Mistake 1: Processing Array Only Once
```python
# Wrong! Doesn't handle circular wrap-around
for i in range(n):  # Should be 2*n
    num = nums[i]
    while stack and nums[stack[-1]] < num:
        idx = stack.pop()
        res[idx] = num
    stack.append(i)

# Fails on: [5,4,3,2,1]
# Output: [-1,-1,-1,-1,-1]  (Should be [-1,5,5,5,5])
```

### ❌ Mistake 2: Pushing Indices in Second Pass
```python
# Wrong! Creates duplicates and incorrect behavior
for i in range(2 * n):
    num = nums[i % n]
    while stack and nums[stack[-1]] < num:
        idx = stack.pop()
        res[idx] = num
    stack.append(i % n)  # Wrong! Always pushes

# Problem: Second pass creates duplicate indices in stack
# Stack would have [0,1,2,0,1,2] instead of [0,1,2]
```

### ❌ Mistake 3: Not Using Modulo for Array Access
```python
# Wrong! Index out of bounds
for i in range(2 * n):
    num = nums[i]  # IndexError when i >= n!
    
# Correct:
for i in range(2 * n):
    num = nums[i % n]  # Wraps around
```

### ❌ Mistake 4: Not Initializing Result to -1
```python
# Wrong! Default 0 or None
res = [0] * n  # Should be [-1] * n

# Problem: 0 or None is wrong default
# We need -1 for "no next greater element"
```

### ❌ Mistake 5: Using Values Instead of Indices in Stack
```python
# Wrong! Can't update result array correctly
stack = []
for i in range(2 * n):
    num = nums[i % n]
    while stack and stack[-1] < num:
        val = stack.pop()  # We have value, not index!
        # How do we update res? We don't know which index!
    if i < n:
        stack.append(num)

# Must use indices to update result array
```

### ❌ Mistake 6: Wrong Condition (>= Instead of >)
```python
# Wrong! Equal values shouldn't pop
while stack and nums[stack[-1]] <= num:  # Should be <

# Problem: If nums=[3,3,3], first 3 would pop second 3
# But 3 is not GREATER than 3, just equal
```

### ❌ Mistake 7: Processing 3*n or More
```python
# Inefficient but might work
for i in range(3 * n):  # Too many iterations
    # Same logic...

# Problem: Not incorrect, but wastes time
# Two passes (2*n) are sufficient
```

---

## Interview Insights

**Question Recognition:**
- "Circular array" + "next greater" → Process twice with modulo
- "Wrap around" → Circular array pattern
- Follow-up to Next Greater Element I

**Key Points to Mention:**

1. **Circular simulation:**
   - "I'll process the array twice using modulo arithmetic to simulate circular behavior."

2. **Only push in first pass:**
   - "The key trick is to only push indices during the first pass (i < n). The second pass just helps find NGE for those elements."

3. **Why 2n iterations:**
   - "Two passes ensure every element can check all other elements as potential next greater, including wrap-around."

4. **Time complexity:**
   - "Despite processing 2n elements, each index is pushed and popped at most once, so it's still O(n)."

5. **Space complexity:**
   - "We use O(n) space for the stack and result array, which is optimal."

**Follow-up Questions:**

1. **Q:** "What if we need the previous greater element in circular array?"
   - **A:** "Process from right to left (or backwards in the 2n loop), using similar logic."

2. **Q:** "Can we do this in one pass?"
   - **A:** "Not really. The circular nature means an element might find its NGE before it in the array, requiring a second look."

3. **Q:** "What about k circular passes?"
   - **A:** "For most cases, 2 passes suffice. More passes don't help since we've already checked all elements."

4. **Q:** "How would you modify for 'next greater or equal'?"
   - **A:** "Change the comparison from < to <= when popping from stack."

5. **Q:** "What if array can have duplicates and we want the closest next greater?"
   - **A:** "Same algorithm works! We pop when strictly greater, so we naturally find the closest."

**Complexity Analysis:**
- Time: O(n) - Each element pushed/popped once despite 2n iterations
- Space: O(n) - Stack and result array

---

## Related Problems

1. **Next Greater Element I (LeetCode 496)** - Non-circular version
2. **Daily Temperatures (LeetCode 739)** - Similar stack pattern
3. **Online Stock Span (LeetCode 901)** - Monotonic stack application
4. **Remove K Digits (LeetCode 402)** - Monotonic stack variant
5. **132 Pattern (LeetCode 456)** - Advanced stack pattern
6. **Largest Rectangle in Histogram (LeetCode 84)** - Next smaller element pattern

---

## Variations

### Variation 1: Distance to Next Greater (Circular)
```python
def nextGreaterDistance(self, nums):
    """
    Return distance to next greater element in circular array.
    """
    n = len(nums)
    res = [-1] * n
    stack = []
    
    for i in range(2 * n):
        num = nums[i % n]
        
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            # Calculate circular distance
            res[idx] = (i - idx) if i < n else (i % n + n - idx)
        
        if i < n:
            stack.append(i)
    
    return res
```

### Variation 2: Next Smaller Element (Circular)
```python
def nextSmallerElements(self, nums):
    """
    Find next smaller element in circular array.
    """
    n = len(nums)
    res = [-1] * n
    stack = []  # Monotonic increasing
    
    for i in range(2 * n):
        num = nums[i % n]
        
        # Pop larger elements (opposite of next greater)
        while stack and nums[stack[-1]] > num:
            idx = stack.pop()
            res[idx] = num
        
        if i < n:
            stack.append(i)
    
    return res
```

### Variation 3: Previous Greater Element (Circular)
```python
def previousGreaterElements(self, nums):
    """
    Find previous greater element in circular array.
    Process backwards.
    """
    n = len(nums)
    res = [-1] * n
    stack = []
    
    # Process backwards for "previous"
    for i in range(2 * n - 1, -1, -1):
        num = nums[i % n]
        
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            res[idx] = num
        
        if i >= n:  # Push in second pass (reversed)
            stack.append(i % n)
    
    return res
```

### Variation 4: All Greater Elements in Circular Order
```python
def allGreaterElements(self, nums):
    """
    Return all greater elements in circular traversal order.
    """
    n = len(nums)
    result = [[] for _ in range(n)]
    
    for i in range(n):
        for j in range(1, n):
            idx = (i + j) % n
            if nums[idx] > nums[i]:
                result[i].append(nums[idx])
    
    return result
    # Note: O(n²), but returns all greater, not just next
```

### Variation 5: Next Greater with K Circular Rotations
```python
def nextGreaterKRotations(self, nums, k):
    """
    Process k full rotations instead of just 2 passes.
    Usually k=2 is enough, but this is more general.
    """
    n = len(nums)
    res = [-1] * n
    stack = []
    
    for i in range(k * n):
        num = nums[i % n]
        
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            res[idx] = num
        
        if i < n:
            stack.append(i)
    
    return res
```

### Variation 6: Count Next Greater Elements (Circular)
```python
def countNextGreater(self, nums):
    """
    Count how many greater elements exist after each element (circular).
    """
    n = len(nums)
    count = [0] * n
    
    for i in range(n):
        for j in range(1, n):
            idx = (i + j) % n
            if nums[idx] > nums[i]:
                count[i] += 1
    
    return count
    # Note: O(n²), could optimize with segment tree
```

### Variation 7: Next Greater Index (Not Value)
```python
def nextGreaterIndex(self, nums):
    """
    Return the index of next greater element, not the value.
    """
    n = len(nums)
    res = [-1] * n
    stack = []
    
    for i in range(2 * n):
        num = nums[i % n]
        
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            res[idx] = i % n  # Store index, not value
        
        if i < n:
            stack.append(i)
    
    return res
```

---

## Summary

**Key Takeaways:**

1. **Circular Array Simulation:**
   - Process 2*n elements using modulo: `nums[i % n]`
   - First n elements build stack
   - Second n elements help find wrap-around NGE
   - Only push indices when i < n

2. **Why Two Passes:**
   - Elements at end can have NGE at beginning
   - Can't determine NGE in single pass for circular arrays
   - Example: [5,4,3,2,1] → all except 5 have NGE of 5 (wrap)

3. **Algorithm Invariants:**
   - Stack stores indices from [0, n-1]
   - Stack is monotonic decreasing (by values)
   - Result initialized to -1 (default for no NGE)
   - Each index pushed/popped at most once

4. **Time Complexity Analysis:**
   - Process 2n elements: O(2n) iterations
   - Each index pushed once: n push operations
   - Each index popped at most once: n pop operations
   - Total: O(2n + n + n) = O(n)

5. **Key Differences from Non-Circular:**
   - Non-circular (NGE I): Single pass, O(n)
   - Circular (NGE II): Double pass with modulo, still O(n)
   - Only push in first pass to avoid duplicates

**Mental Model:**
```
Think of unrolling the circular array:
[1, 2, 3] circular becomes [1, 2, 3, 1, 2, 3] linear

But we only care about NGE for first 3 positions!
Second 3 positions just help find wrap-around NGE.

Stack behavior:
- First pass: Build stack with elements needing NGE
- Second pass: Pop stack as we find their NGE
- Don't push in second pass (already have those elements)
```

**Template (Circular Next Greater):**
```python
def nextGreaterElements(self, nums):
    n = len(nums)
    res = [-1] * n
    stack = []
    
    # Process twice for circular
    for i in range(2 * n):
        num = nums[i % n]  # Circular access
        
        # Pop smaller elements (they found NGE)
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            res[idx] = num
        
        # Only push in first pass
        if i < n:
            stack.append(i)
    
    return res
```

This circular variant is a classic extension of the monotonic stack pattern!
