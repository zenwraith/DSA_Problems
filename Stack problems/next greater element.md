# Next Greater Element I

**LeetCode 496** | **Difficulty:** Easy | **Category:** Stack, Array, Hash Table, Monotonic Stack

---

## Problem Statement

The **next greater element** of some element `x` in an array is the **first greater** element that is **to the right** of `x` in the same array.

You are given two **distinct 0-indexed** integer arrays `nums1` and `nums2`, where `nums1` is a subset of `nums2`.

For each `0 <= i < nums1.length`, find the index `j` such that `nums1[i] == nums2[j]` and determine the **next greater element** of `nums2[j]` in `nums2`. If there is no next greater element, then the answer for this query is `-1`.

Return an array `ans` of length `nums1.length` such that `ans[i]` is the **next greater element** as described above.

**Constraints:**
- `1 <= nums1.length <= nums2.length <= 1000`
- `0 <= nums1[i], nums2[i] <= 10^4`
- All integers in `nums1` and `nums2` are **unique**.
- All the integers of `nums1` also appear in `nums2`.

**Follow up:** Could you find an `O(nums1.length + nums2.length)` solution?

---

## Examples

### Example 1:
```
Input: nums1 = [4,1,2], nums2 = [1,3,4,2]
Output: [-1,3,-1]

Explanation:
The next greater element for each value of nums1 is as follows:
- 4 is underlined in nums2 = [1,3,4,2]. There is no next greater element, so the answer is -1.
- 1 is underlined in nums2 = [1,3,4,2]. The next greater element is 3.
- 2 is underlined in nums2 = [1,3,4,2]. There is no next greater element, so the answer is -1.
```

### Example 2:
```
Input: nums1 = [2,4], nums2 = [1,2,3,4]
Output: [3,-1]

Explanation:
The next greater element for each value of nums1 is as follows:
- 2 is underlined in nums2 = [1,2,3,4]. The next greater element is 3.
- 4 is underlined in nums2 = [1,2,3,4]. There is no next greater element, so the answer is -1.
```

### Example 3:
```
Input: nums1 = [1,3,5,2,4], nums2 = [6,5,4,3,2,1,7]
Output: [7,7,7,7,7]

Explanation:
All elements in nums1 have 7 as their next greater element in nums2.
```

---

## Strategy

### Monotonic Stack with Hash Map

**Key Insight:** Use a monotonic decreasing stack to efficiently find next greater elements, then store results in a hash map for O(1) lookup.

**Why This Works:**

```
Challenge: Naive approach is O(n²)
  - For each element in nums1, find it in nums2
  - Then scan right to find first greater element
  - Total: O(nums1.length * nums2.length)

Better Approach: Monotonic Stack
  - Process nums2 once to build next greater element mapping
  - Use stack to maintain elements waiting for their next greater
  - Store results in hash map for O(1) lookup
  - Query nums1 using the hash map

Key Property: Monotonic Decreasing Stack
  - Stack maintains elements in decreasing order (top to bottom)
  - When we see a larger element, it's the "next greater" for all smaller elements in stack
  - Pop all smaller elements and record their next greater element
  - Push current element onto stack

Why It Works:
  - Elements in stack are waiting for their next greater element
  - Current element is the first greater element they encounter
  - Once popped, they never need to be reconsidered
```

**Algorithm Steps:**

1. **Build Next Greater Map:**
   - Process nums2 left to right with a stack
   - For each element:
     - While stack is not empty AND current > stack top:
       - Pop element from stack
       - Record: next_greater[popped] = current
     - Push current element onto stack
   - Elements remaining in stack have no next greater → -1

2. **Query Results:**
   - For each element in nums1, lookup in hash map
   - Return mapped value or -1 if not found

**Complexity:**
- **Time:** $O(n + m)$ where n = len(nums2), m = len(nums1)
  - Each element in nums2 pushed/popped at most once: O(n)
  - Query nums1: O(m)
- **Space:** $O(n)$ for stack and hash map

---

## Solution

```python
class Solution:
    def nextGreaterElement(self, nums1: List[int], nums2: List[int]) -> List[int]:
        """
        Find next greater element for each element in nums1.
        
        Strategy: Monotonic stack + hash map
        1. Build next greater mapping for all elements in nums2
        2. Use monotonic decreasing stack to find NGE in one pass
        3. Query results using hash map
        
        Key insight: When we encounter a larger element, it's the NGE 
        for all smaller elements currently in the stack.
        
        Edge cases:
        - Element at end of nums2: No next greater (stack)
        - Largest element: No next greater
        - All decreasing: Stack keeps growing, all -1
        - All increasing: Stack always empty except current
        
        Args:
            nums1: Query array (subset of nums2)
            nums2: Reference array to find next greater elements
            
        Returns:
            Array of next greater elements for nums1
        """
        # Hash map to store next greater element for each value
        nge_map = {}
        
        # Monotonic decreasing stack
        stack = []
        
        # Process nums2 to build next greater element mapping
        for num in nums2:
            # Current num is next greater for all smaller elements in stack
            while stack and num > stack[-1]:
                val = stack.pop()
                nge_map[val] = num
            
            # Push current element (waiting for its next greater)
            stack.append(num)
        
        # Elements still in stack have no next greater element
        # (They're handled by get(n, -1) which returns -1 if not found)
        
        # Build result by querying hash map
        return [nge_map.get(n, -1) for n in nums1]
```

---

## Detailed Walkthrough

### Example 1: `nums1 = [4,1,2], nums2 = [1,3,4,2]`

**Building Next Greater Map:**

| Step | num | stack (before) | Action | stack (after) | nge_map |
|------|-----|----------------|--------|---------------|---------|
| 1 | 1 | [] | Push 1 | [1] | {} |
| 2 | 3 | [1] | 3 > 1: pop 1, map 1→3, push 3 | [3] | {1: 3} |
| 3 | 4 | [3] | 4 > 3: pop 3, map 3→4, push 4 | [4] | {1: 3, 3: 4} |
| 4 | 2 | [4] | 2 < 4: push 2 | [4, 2] | {1: 3, 3: 4} |

**Final State:**
- `nge_map = {1: 3, 3: 4}`
- `stack = [4, 2]` (elements with no next greater)

**Querying nums1:**

| Element | Lookup | Result |
|---------|--------|--------|
| 4 | nge_map.get(4, -1) | -1 (not in map) |
| 1 | nge_map.get(1, -1) | 3 |
| 2 | nge_map.get(2, -1) | -1 (not in map) |

**Output:** `[-1, 3, -1]` ✓

---

### Example 2: `nums1 = [2,4], nums2 = [1,2,3,4]`

**Building Next Greater Map:**

| Step | num | stack | Action | nge_map Updates |
|------|-----|-------|--------|-----------------|
| 1 | 1 | [] → [1] | Push 1 | {} |
| 2 | 2 | [1] → [2] | 2 > 1: pop 1, map 1→2 | {1: 2} |
| 3 | 3 | [2] → [3] | 3 > 2: pop 2, map 2→3 | {1: 2, 2: 3} |
| 4 | 4 | [3] → [4] | 4 > 3: pop 3, map 3→4 | {1: 2, 2: 3, 3: 4} |

**Stack Behavior:**
```
After 1: [1]
After 2: [2]      (popped 1, added 2→1's NGE)
After 3: [3]      (popped 2, added 3→2's NGE)
After 4: [4]      (popped 3, added 4→3's NGE)
```

**Querying nums1:**
- `2`: nge_map[2] = 3
- `4`: nge_map.get(4, -1) = -1

**Output:** `[3, -1]` ✓

---

### Example 3: `nums1 = [1,3,5,2,4], nums2 = [6,5,4,3,2,1,7]`

**Building Next Greater Map (Decreasing then 7):**

| Step | num | stack (before) | Action | nge_map |
|------|-----|----------------|--------|---------|
| 1 | 6 | [] | Push 6 | {} |
| 2 | 5 | [6] | 5 < 6: push 5 | {} |
| 3 | 4 | [6,5] | 4 < 5: push 4 | {} |
| 4 | 3 | [6,5,4] | 3 < 4: push 3 | {} |
| 5 | 2 | [6,5,4,3] | 2 < 3: push 2 | {} |
| 6 | 1 | [6,5,4,3,2] | 1 < 2: push 1 | {} |
| 7 | 7 | [6,5,4,3,2,1] | 7 > all! Pop all | {6:7, 5:7, 4:7, 3:7, 2:7, 1:7} |

**Stack Evolution:**
```
[6]
[6, 5]
[6, 5, 4]
[6, 5, 4, 3]
[6, 5, 4, 3, 2]
[6, 5, 4, 3, 2, 1]  ← Monotonic decreasing!

Then 7 arrives:
Pop 1: map 1→7
Pop 2: map 2→7
Pop 3: map 3→7
Pop 4: map 4→7
Pop 5: map 5→7
Pop 6: map 6→7

Stack becomes: [7]
```

**Querying nums1 = [1,3,5,2,4]:**
- All map to 7!

**Output:** `[7, 7, 7, 7, 7]` ✓

---

## Why This Algorithm Works

### Monotonic Stack Property

**Definition:** A monotonic decreasing stack maintains elements in decreasing order from bottom to top.

```
Example stack during processing: [10, 8, 5, 3]
                                  ↑          ↑
                                bottom    top

Property: Each element in stack is smaller than the one below it.
```

**Why This Helps:**

```
When we encounter a number larger than stack top:
  - It's the FIRST greater number encountered so far
  - All elements in stack smaller than it have found their NGE
  - We can pop and record all of them

Example: Stack = [10, 8, 5, 3], current = 6

6 > 3: Pop 3, record NGE[3] = 6
6 > 5: Pop 5, record NGE[5] = 6
6 < 8: Stop! Push 6

New stack: [10, 8, 6]  ← Still monotonic decreasing
```

### Why One Pass is Sufficient

```
Question: How do we know we won't miss any NGE?

Answer: The stack maintains all elements waiting for their NGE.

Proof by cases:
1. Element gets popped → Its NGE is found and recorded
2. Element stays in stack → No element to the right is greater (no NGE)

Every element either:
  - Gets popped (NGE recorded)
  - Remains in stack (no NGE exists)
  
No element is missed!
```

### Why Push After Popping

```
After popping smaller elements, we push current element.

Why? Because current element might also need a next greater element!

Example: [3, 1, 4]

Process 1:
  - 1 < 3, so push 1
  - Stack: [3, 1]

Process 4:
  - Pop 1 (1 < 4), record 1→4
  - Pop 3 (3 < 4), record 3→4
  - Push 4 (it might need an NGE later)
  - Stack: [4]

If there were a 5 next, we'd pop 4 and record 4→5.
```

---

## Edge Cases

### 1. All Decreasing (Worst Case for Stack)
```python
nums1 = [5,4,3,2,1], nums2 = [5,4,3,2,1]
# Stack grows: [5] → [5,4] → [5,4,3] → [5,4,3,2] → [5,4,3,2,1]
# No elements popped, all stay in stack
# nge_map remains empty {}
# Output: [-1, -1, -1, -1, -1]
```

### 2. All Increasing (Best Case for Stack)
```python
nums1 = [1,2,3,4,5], nums2 = [1,2,3,4,5]
# Each element immediately pops previous
# Stack never grows beyond size 1
# nge_map: {1:2, 2:3, 3:4, 4:5}
# Output: [2, 3, 4, 5, -1]
```

### 3. Single Element
```python
nums1 = [1], nums2 = [1]
# Stack: [1], no pops
# nge_map: {}
# Output: [-1]
```

### 4. nums1 = nums2
```python
nums1 = [2,4], nums2 = [2,4]
# Same as example 2 but only querying elements in nums2
# nge_map: {2: 4}
# Output: [4, -1]
```

### 5. Last Element Query
```python
nums1 = [5], nums2 = [1,2,3,4,5]
# 5 is last in nums2, stays in stack
# nge_map: {1:2, 2:3, 3:4, 4:5}
# 5 not in map
# Output: [-1]
```

### 6. Mountain Pattern
```python
nums1 = [1,2,3,4,5], nums2 = [1,2,5,4,3]
# Process:
#   1: stack [1]
#   2: pop 1 (map 1→2), stack [2]
#   5: pop 2 (map 2→5), stack [5]
#   4: 4 < 5, stack [5,4]
#   3: 3 < 4, stack [5,4,3]
# nge_map: {1:2, 2:5}
# Output: [2, 5, -1, -1, -1]
```

### 7. Duplicate Lookups (Multiple elements in nums1)
```python
nums1 = [2,2,2], nums2 = [1,2,3]
# nge_map: {1:2, 2:3}
# Output: [3, 3, 3]  (all lookups return same value)
```

---

## Common Mistakes

### ❌ Mistake 1: Not Popping All Smaller Elements
```python
# Wrong! Only pops one element
while stack and num > stack[-1]:
    val = stack.pop()
    nge_map[val] = num
    break  # Wrong! Should continue popping

# Correct: Keep popping while condition holds
while stack and num > stack[-1]:
    val = stack.pop()
    nge_map[val] = num  # No break
```

### ❌ Mistake 2: Wrong Stack Ordering (Increasing Instead of Decreasing)
```python
# Wrong! This maintains increasing order
while stack and num < stack[-1]:  # Should be >
    val = stack.pop()
    nge_map[val] = num
```

### ❌ Mistake 3: Not Handling Elements Not in Map
```python
# Wrong! KeyError if element not in map
return [nge_map[n] for n in nums1]

# Correct: Use get() with default value
return [nge_map.get(n, -1) for n in nums1]
```

### ❌ Mistake 4: Overwriting NGE Values
```python
# Wrong! Might overwrite correct NGE with later value
for num in nums2:
    while stack and num > stack[-1]:
        val = stack.pop()
        nge_map[val] = num  # First assignment is correct
        
# If we processed same element again somehow, we'd overwrite
# But this doesn't happen in correct implementation since
# each element is pushed exactly once
```

### ❌ Mistake 5: Not Pushing Current Element
```python
# Wrong! Forgets to add current element to stack
while stack and num > stack[-1]:
    val = stack.pop()
    nge_map[val] = num
# Missing: stack.append(num)

# This means current element won't be considered for future NGE
```

### ❌ Mistake 6: Using Array Index Instead of Value
```python
# Wrong! Storing indices instead of values
for i, num in enumerate(nums2):
    while stack and num > nums2[stack[-1]]:
        idx = stack.pop()
        nge_map[idx] = num  # Wrong key!
    stack.append(i)

# Problem: nums1 contains values, not indices
# We need to map values to values, not indices to values
```

### ❌ Mistake 7: Nested Loop O(n²) Approach
```python
# Wrong! Inefficient nested loop
result = []
for num in nums1:
    idx = nums2.index(num)
    nge = -1
    for j in range(idx + 1, len(nums2)):
        if nums2[j] > num:
            nge = nums2[j]
            break
    result.append(nge)

# Time: O(m * n) instead of O(m + n)
```

---

## Interview Insights

**Question Recognition:**
- "Next greater element" → Monotonic stack pattern
- "First element to the right that is larger" → Stack-based solution
- "For each element, find the next..." → Consider monotonic stack

**Key Points to Mention:**

1. **Monotonic stack pattern:**
   - "I'll use a monotonic decreasing stack to efficiently find next greater elements in one pass through nums2."

2. **Why stack works:**
   - "The stack maintains elements waiting for their next greater element. When we encounter a larger element, it's the answer for all smaller elements in the stack."

3. **Time complexity:**
   - "Each element is pushed and popped at most once, giving us O(n) for processing nums2, plus O(m) for querying nums1, so O(n+m) total."

4. **Hash map for lookup:**
   - "I store the results in a hash map for O(1) lookup when querying nums1."

5. **Space optimization:**
   - "We need O(n) space for the stack and hash map, which is optimal since we need to store results."

**Follow-up Questions:**

1. **Q:** "What if nums1 is not a subset of nums2?"
   - **A:** "I'd still use .get(n, -1) which returns -1 for missing elements. The algorithm handles this naturally."

2. **Q:** "Can you do this without extra space?"
   - **A:** "Not really. We need to store the NGE mapping somehow. Even if we modify arrays in place, we'd still need O(n) space for the stack."

3. **Q:** "What if we need the next SMALLER element instead?"
   - **A:** "I'd use a monotonic increasing stack instead of decreasing, and change the comparison from > to <."

4. **Q:** "What about next greater element in a circular array?"
   - **A:** "That's LeetCode 503. Process the array twice (or once with modulo) to handle wraparound."

5. **Q:** "How would you find the next greater element for ALL elements in nums2?"
   - **A:** "Same algorithm, but query all elements in nums2 instead of just nums1. Still O(n)."

**Complexity Analysis:**
- Time: O(n + m) where n = len(nums2), m = len(nums1)
- Space: O(n) for stack and hash map

---

## Related Problems

1. **Next Greater Element II (LeetCode 503)** - Circular array version
2. **Next Greater Element III (LeetCode 556)** - Find next greater permutation
3. **Daily Temperatures (LeetCode 739)** - Similar monotonic stack pattern
4. **Largest Rectangle in Histogram (LeetCode 84)** - Monotonic stack application
5. **Trapping Rain Water (LeetCode 42)** - Can use monotonic stack approach
6. **132 Pattern (LeetCode 456)** - Monotonic stack variant

---

## Variations

### Variation 1: Next Greater Element II (Circular Array)
```python
def nextGreaterElements(self, nums):
    """
    Find NGE in circular array (last element can use first elements).
    """
    n = len(nums)
    result = [-1] * n
    stack = []
    
    # Process array twice to handle circular nature
    for i in range(2 * n):
        num = nums[i % n]
        
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            result[idx] = num
        
        # Only push indices from first pass
        if i < n:
            stack.append(i)
    
    return result
```

### Variation 2: Previous Greater Element
```python
def previousGreaterElement(self, nums):
    """
    Find previous greater element (to the left).
    """
    result = []
    stack = []
    
    for num in nums:
        # Find previous greater
        while stack and stack[-1] <= num:
            stack.pop()
        
        result.append(stack[-1] if stack else -1)
        stack.append(num)
    
    return result
```

### Variation 3: Next Smaller Element
```python
def nextSmallerElement(self, nums):
    """
    Find next smaller element (opposite of next greater).
    """
    nse_map = {}
    stack = []  # Monotonic increasing stack
    
    for num in nums:
        # Current num is next smaller for all larger elements in stack
        while stack and num < stack[-1]:
            val = stack.pop()
            nse_map[val] = num
        
        stack.append(num)
    
    return [nse_map.get(n, -1) for n in nums]
```

### Variation 4: Distance to Next Greater Element
```python
def nextGreaterDistance(self, nums):
    """
    Return distance to next greater element, not the element itself.
    """
    result = [-1] * len(nums)
    stack = []  # Store indices
    
    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            result[idx] = i - idx  # Distance, not value
        
        stack.append(i)
    
    return result
```

### Variation 5: Next Greater or Equal Element
```python
def nextGreaterOrEqual(self, nums1, nums2):
    """
    Find next greater OR equal element.
    """
    nge_map = {}
    stack = []
    
    for num in nums2:
        # Use >= instead of >
        while stack and num >= stack[-1]:
            val = stack.pop()
            nge_map[val] = num
        
        stack.append(num)
    
    return [nge_map.get(n, -1) for n in nums1]
```

### Variation 6: K-th Next Greater Element
```python
def kthNextGreater(self, nums, k):
    """
    Find the k-th next greater element.
    Much harder! Need different approach.
    """
    result = []
    
    for i, num in enumerate(nums):
        count = 0
        nge = -1
        
        for j in range(i + 1, len(nums)):
            if nums[j] > num:
                count += 1
                if count == k:
                    nge = nums[j]
                    break
        
        result.append(nge)
    
    return result
    # Note: This is O(n*k), harder to optimize
```

### Variation 7: All Next Greater Elements
```python
def allNextGreater(self, nums):
    """
    Return NGE for every element (don't need nums1).
    """
    result = [-1] * len(nums)
    stack = []  # Store indices
    
    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            idx = stack.pop()
            result[idx] = num
        
        stack.append(i)
    
    return result
```

---

## Summary

**Key Takeaways:**

1. **Monotonic Stack Pattern:**
   - Maintains elements in specific order (decreasing for NGE)
   - Efficiently finds "next greater/smaller" in O(n) time
   - Each element pushed and popped at most once

2. **Algorithm Flow:**
   - Process array left to right
   - Pop smaller elements when larger element arrives
   - Record popped elements' NGE as current element
   - Push current element (waiting for its NGE)

3. **Why It's Optimal:**
   - Single pass through nums2: O(n)
   - Hash map for O(1) lookup: O(m) for nums1
   - Total: O(n + m), which is optimal

4. **Stack State Invariant:**
   - Stack always maintains monotonic decreasing order
   - Elements in stack are waiting for their NGE
   - Elements not in stack either:
     - Found their NGE (popped and recorded)
     - Or have no NGE (remaining in stack at end)

5. **Common Applications:**
   - Next/previous greater/smaller element
   - Temperature problems (days until warmer)
   - Stock span problems
   - Histogram problems
   - Expression evaluation

**Mental Model:**
```
Think of the stack as a "waiting room":
- Elements waiting for someone taller to arrive
- When someone taller arrives, all shorter people leave
- They record the taller person's height as their answer
- The taller person now waits for someone even taller
```

**Template (Next Greater Element):**
```python
def nextGreaterElement(self, nums):
    nge_map = {}
    stack = []  # Monotonic decreasing
    
    for num in nums:
        # Pop all smaller elements (they found their NGE)
        while stack and num > stack[-1]:
            val = stack.pop()
            nge_map[val] = num
        
        # Current element waits for its NGE
        stack.append(num)
    
    # Elements in stack have no NGE
    return [nge_map.get(n, -1) for n in query_array]
```

This is one of the most important stack patterns in competitive programming and interviews!
