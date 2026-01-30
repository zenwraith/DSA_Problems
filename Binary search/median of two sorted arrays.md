# Median of Two Sorted Arrays

**LeetCode 4** | **Difficulty:** Hard | **Category:** Binary Search, Array, Divide and Conquer

---

## Problem Statement

Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return **the median** of the two sorted arrays.

The overall run time complexity should be **O(log (m+n))**.

---

## Examples

### Example 1:
```
Input: nums1 = [1,3], nums2 = [2]
Output: 2.00000
Explanation: merged array = [1,2,3] and median is 2.
```

### Example 2:
```
Input: nums1 = [1,2], nums2 = [3,4]
Output: 2.50000
Explanation: merged array = [1,2,3,4] and median is (2 + 3) / 2 = 2.5.
```

### Example 3:
```
Input: nums1 = [0,0], nums2 = [0,0]
Output: 0.00000
```

### Example 4:
```
Input: nums1 = [], nums2 = [1]
Output: 1.00000
```

### Example 5:
```
Input: nums1 = [2], nums2 = []
Output: 2.00000
```

---

## Strategy

### Binary Search on the Smaller Array

**Key Insight:** Use binary search to partition both arrays such that:
- Elements on left side ≤ Elements on right side
- Both sides have equal count (or left has 1 more for odd total)

**Why This is Hard:**

Simple approach: Merge arrays → Find median → O(m+n) time ❌

Required: O(log(m+n)) → Must use binary search ✓

**Core Concept: Partitioning**

```
Array A: [1, 3, 8, 9, 15]
Array B: [7, 11, 18, 19, 21, 25]

Goal: Partition both arrays so:
Left side:  [1, 3, 7, 8, 9, 11]  (6 elements)
Right side: [15, 18, 19, 21, 25]  (5 elements)

Median = (max(left) + min(right)) / 2 = (11 + 15) / 2 = 13
```

**Partitioning Strategy:**

```
Array A: [1, 3, | 8, 9, 15]
           ↑ A_left=3  A_right=8 ↑
           
Array B: [7, 11, 18, | 19, 21, 25]
           ↑ B_left=18  B_right=19 ↑

Partition is valid when:
1. A_left ≤ B_right  (3 ≤ 19) ✓
2. B_left ≤ A_right  (18 ≤ 8) ✗  Invalid!

Need to adjust partition!
```

**Valid Partition:**

```
Array A: [1, 3, 8, 9, | 15]
           ↑ A_left=9  A_right=15 ↑
           
Array B: [7, 11, | 18, 19, 21, 25]
           ↑ B_left=11  B_right=18 ↑

Check:
1. A_left ≤ B_right  (9 ≤ 18) ✓
2. B_left ≤ A_right  (11 ≤ 15) ✓  Valid!

Median = (max(9, 11) + min(15, 18)) / 2 = (11 + 15) / 2 = 13
```

**Binary Search Logic:**

1. **Always binary search on smaller array** (for efficiency)
2. **Calculate partition index in larger array** based on smaller
3. **Check partition validity:**
   - If `A_left > B_right`: Too many from A on left → Move left
   - If `B_left > A_right`: Too few from A on left → Move right
   - Otherwise: Valid partition found!

**Mathematical Foundation:**

```
Total elements: m + n
Half count: (m + n + 1) // 2

If we take i elements from A:
We need (half - i) elements from B

This ensures:
- Odd total: Left has 1 more element
- Even total: Both sides equal
```

**Complexity:**
- **Time:** $O(\log(\min(m, n)))$ - Binary search on smaller array
- **Space:** $O(1)$ - Only using pointers and variables

---

## Solution

```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        """
        Find median of two sorted arrays using binary search.
        
        Strategy:
        1. Binary search on smaller array for efficiency
        2. Partition both arrays into left/right halves
        3. Find partition where all left ≤ all right
        4. Median is boundary values of valid partition
        
        Args:
            nums1: First sorted array
            nums2: Second sorted array
            
        Returns:
            Median of combined sorted arrays
        """
        A, B = nums1, nums2
        total = len(A) + len(B)
        half = (total + 1) // 2  # Use +1 for odd case handling
        
        # Always binary search on smaller array for efficiency
        if len(B) < len(A):
            A, B = B, A
        
        l, r = 0, len(A)
        
        while l <= r:
            # Partition index in A (number of elements from A in left half)
            i = (l + r) // 2
            
            # Partition index in B (remaining elements needed for left half)
            j = half - i
            
            # Get boundary elements (use -inf/inf for edge cases)
            A_left = A[i - 1] if i > 0 else float('-inf')
            A_right = A[i] if i < len(A) else float('inf')
            B_left = B[j - 1] if j > 0 else float('-inf')
            B_right = B[j] if j < len(B) else float('inf')
            
            # Check if partition is valid
            if A_left <= B_right and B_left <= A_right:
                # Found valid partition!
                
                # Odd total: Median is max of left side
                if total % 2:
                    return max(A_left, B_left)
                
                # Even total: Median is average of middle two
                return (max(A_left, B_left) + min(A_right, B_right)) / 2
            
            # Invalid partition: Adjust search space
            elif A_left > B_right:
                # Too many elements from A on left side
                r = i - 1
            else:
                # Too few elements from A on left side
                l = i + 1
```

---

## Detailed Walkthrough

### Example 1: `nums1 = [1,3], nums2 = [2,4,5]`

**Setup:**
- Total = 5, half = 3
- A = [1,3] (smaller), B = [2,4,5]
- Binary search on A

**Step-by-Step Execution:**

| Step | l | r | i | j | A_left | A_right | B_left | B_right | Valid? | Action |
|------|---|---|---|---|--------|---------|--------|---------|--------|--------|
| 0 | 0 | 2 | 1 | 2 | 1 | 3 | 4 | 5 | A_left≤B_right ✓<br>B_left≤A_right ✗ | l=2 |
| 1 | 2 | 2 | 2 | 1 | 3 | ∞ | 2 | 4 | A_left≤B_right ✓<br>B_left≤A_right ✓ | Found! |

**Partition Visualization:**
```
Step 1:
A: [1, 3, | ]
    ↑  ↑
   A_left=3  A_right=∞

B: [2, | 4, 5]
    ↑    ↑
   B_left=2  B_right=4

Check: 3 ≤ 4 ✓ and 2 ≤ ∞ ✓

Total is odd (5), so median = max(3, 2) = 3
```

**Final Answer:** `3.0` ✓

---

### Example 2: `nums1 = [1,2], nums2 = [3,4]`

**Setup:**
- Total = 4, half = 2
- A = [1,2], B = [3,4]

| Step | l | r | i | j | A_left | A_right | B_left | B_right | Valid? |
|------|---|---|---|---|--------|---------|--------|---------|--------|
| 0 | 0 | 2 | 1 | 1 | 1 | 2 | 3 | 4 | ✓ Found! |

**Partition:**
```
A: [1, | 2]
    ↑    ↑
   A_left=1  A_right=2

B: [3, | 4]
    ↑    ↑
   B_left=3  B_right=4

Wait! Check: 1 ≤ 4 ✓ but 3 ≤ 2 ✗

Let me recalculate...

Step 0: i=1, j=1
A_left=1, A_right=2
B_left=3, B_right=4

Check: 1 ≤ 4 ✓ and 3 ≤ 2 ✗
So go to else case: l = i+1 = 2

Step 1: l=2, r=2, i=2, j=0
A_left=2, A_right=∞
B_left=-∞, B_right=3

Check: 2 ≤ 3 ✓ and -∞ ≤ ∞ ✓

Total is even (4):
median = (max(2, -∞) + min(∞, 3)) / 2 = (2 + 3) / 2 = 2.5
```

**Final Answer:** `2.5` ✓

---

### Example 3: `nums1 = [1,3,8,9,15], nums2 = [7,11,18,19,21,25]`

**Setup:**
- Total = 11, half = 6
- A = [1,3,8,9,15], B = [7,11,18,19,21,25]

| Step | l | r | i | j | A_left | A_right | B_left | B_right | Check | Action |
|------|---|---|---|---|--------|---------|--------|---------|-------|--------|
| 0 | 0 | 5 | 2 | 4 | 3 | 8 | 19 | 21 | 3≤21 ✓<br>19≤8 ✗ | l=3 |
| 1 | 3 | 5 | 4 | 2 | 9 | 15 | 11 | 18 | 9≤18 ✓<br>11≤15 ✓ | Found! |

**Partition:**
```
A: [1, 3, 8, 9, | 15]
              ↑      ↑
         A_left=9  A_right=15

B: [7, 11, | 18, 19, 21, 25]
        ↑      ↑
   B_left=11  B_right=18

Total is odd (11):
median = max(9, 11) = 11
```

**Final Answer:** `11.0` ✓

---

### Example 4: Empty Array - `nums1 = [], nums2 = [1]`

**Setup:**
- Total = 1, half = 1
- A = [] (smaller), B = [1]

| Step | l | r | i | j | A_left | A_right | B_left | B_right | Valid? |
|------|---|---|---|---|--------|---------|--------|---------|--------|
| 0 | 0 | 0 | 0 | 1 | -∞ | ∞ | 1 | ∞ | ✓ |

**Partition:**
```
A: [ | ]
   A_left=-∞  A_right=∞

B: [1, | ]
    ↑    
   B_left=1  B_right=∞

Total is odd (1):
median = max(-∞, 1) = 1
```

**Final Answer:** `1.0` ✓

---

## Why This Algorithm Works

### The Partitioning Guarantee

**Claim:** If we partition arrays correctly, the median is at the boundary.

**Proof:**

1. **Partition Definition:**
   ```
   Left half:  A[0..i-1] + B[0..j-1]  (exactly half elements)
   Right half: A[i..m-1] + B[j..n-1]  (remaining elements)
   ```

2. **Valid Partition Conditions:**
   ```
   A_left ≤ B_right  (All A elements on left ≤ All B elements on right)
   B_left ≤ A_right  (All B elements on left ≤ All A elements on right)
   ```

3. **If valid, then:**
   ```
   max(left side) ≤ min(right side)
   
   This means elements are properly ordered!
   ```

4. **Median Location:**
   - **Odd total:** Median = max(A_left, B_left) (largest in left half)
   - **Even total:** Median = (max(A_left, B_left) + min(A_right, B_right)) / 2

**Why Binary Search Finds It:**

- If A_left > B_right: We took too many from A, move partition left
- If B_left > A_right: We took too few from A, move partition right
- Converges to valid partition in O(log m) steps

---

### Visual Proof

```
Invalid Partition (A_left > B_right):

A: [1, 3, 8, | 9, 15]
           ↑ = 8
           
B: [7, | 11, 18, 19]
        ↑ = 7

8 > 11? No, but let's say A_left > B_right:
This means elements are out of order!
We need fewer elements from A on left side.
```

```
Valid Partition:

A: [1, 3, | 8, 9, 15]
        ↑
   A_left=3

B: [7, 11, | 18, 19]
        ↑
   B_left=11

Check: 3 ≤ 18 ✓ and 11 ≤ 8 ✗
Still invalid! Adjust...

Final:
A: [1, 3, 8, 9, | 15]
              ↑
         A_left=9

B: [7, 11, | 18, 19]
        ↑
   B_left=11

Check: 9 ≤ 18 ✓ and 11 ≤ 15 ✓
Valid! Median = max(9, 11) = 11
```

---

## Edge Cases

### 1. One Empty Array
```python
nums1 = [], nums2 = [1,2,3]
# Output: 2.0
# A is empty, all elements from B
```

### 2. Both Single Element
```python
nums1 = [1], nums2 = [2]
# Output: 1.5
# Median = (1 + 2) / 2
```

### 3. No Overlap
```python
nums1 = [1,2,3], nums2 = [4,5,6]
# Output: 3.5
# All of A before all of B
```

### 4. Complete Overlap
```python
nums1 = [1,3,5], nums2 = [2,4,6]
# Output: 3.5
# Interleaved: [1,2,3,4,5,6]
```

### 5. Duplicates
```python
nums1 = [1,1,1], nums2 = [1,1,1]
# Output: 1.0
# All same value
```

### 6. Negative Numbers
```python
nums1 = [-5,-3,-1], nums2 = [0,2,4]
# Output: -0.5
# Works with negatives
```

### 7. Very Different Sizes
```python
nums1 = [1], nums2 = [2,3,4,5,6,7,8]
# Output: 4.5
# Binary search still efficient
```

---

## Common Mistakes

### ❌ Mistake 1: Not Using (total + 1) // 2
```python
# Wrong! Doesn't handle odd case correctly
half = total // 2
```

**Fix:** Use `(total + 1) // 2`
```python
half = (total + 1) // 2
```

**Why?**
- Odd total (5): half = 3 → Left has 3, right has 2 ✓
- Even total (4): half = 2 → Left has 2, right has 2 ✓

### ❌ Mistake 2: Not Searching on Smaller Array
```python
# Wrong! Can cause index out of bounds in larger array
# Always search on smaller array
```

**Fix:** Swap if needed
```python
if len(B) < len(A):
    A, B = B, A
```

### ❌ Mistake 3: Wrong Loop Condition
```python
# Wrong! Should use <=
while l < r:
    # ...
```

**Fix:** Use `l <= r`
```python
while l <= r:
    # ...
```

### ❌ Mistake 4: Not Handling Edge Cases with -inf/inf
```python
# Wrong! Index out of bounds when i=0 or i=len(A)
A_left = A[i - 1]
A_right = A[i]
```

**Fix:** Use conditional checks
```python
A_left = A[i - 1] if i > 0 else float('-inf')
A_right = A[i] if i < len(A) else float('inf')
```

### ❌ Mistake 5: Wrong Median Calculation
```python
# Wrong! Always using average
return (max(A_left, B_left) + min(A_right, B_right)) / 2
```

**Fix:** Check if odd or even
```python
if total % 2:
    return max(A_left, B_left)  # Odd: just max of left
return (max(A_left, B_left) + min(A_right, B_right)) / 2  # Even: average
```

---

## Interview Insights

**Question Recognition:**
- "Two sorted arrays" + "median" + "O(log n)" → Binary search on smaller array
- One of the hardest LeetCode problems

**Key Points to Mention:**

1. **Why not merge?**
   - "Merging gives O(m+n) time. We need O(log(m+n)), which suggests binary search."

2. **The partitioning insight:**
   - "The key is to partition both arrays so left half has exactly half the elements, and all left elements ≤ all right elements."

3. **Why search on smaller array:**
   - "We binary search on the smaller array to ensure the partition index in the larger array stays valid (0 to n)."

4. **Edge case handling:**
   - "Use -infinity and +infinity for boundary cases when partition is at array edges."

**Follow-up Questions:**

1. **Q:** "Can you do better than O(log(min(m,n)))?"
   - **A:** No. This is optimal. The median fundamentally requires examining both arrays.

2. **Q:** "What if arrays aren't sorted?"
   - **A:** Would need O((m+n) log(m+n)) to sort first, or use QuickSelect for O(m+n) average.

3. **Q:** "How would you find the kth smallest element?"
   - **A:** Similar approach, but adjust partition size to k instead of half.

4. **Q:** "What if one array is much larger?"
   - **A:** Algorithm handles this efficiently by binary searching on smaller array.

**Complexity Analysis:**
- Time: O(log(min(m, n))) - Binary search on smaller array
- Space: O(1) - Only using pointers

---

## Variations

### Variation 1: Find Kth Smallest Element
```python
def findKthElement(nums1, nums2, k):
    """Find the kth smallest element in two sorted arrays."""
    A, B = nums1, nums2
    
    if len(B) < len(A):
        A, B = B, A
    
    l, r = 0, len(A)
    
    while l <= r:
        i = (l + r) // 2
        j = k - i  # Need k elements total on left
        
        A_left = A[i - 1] if i > 0 else float('-inf')
        A_right = A[i] if i < len(A) else float('inf')
        B_left = B[j - 1] if j > 0 else float('-inf')
        B_right = B[j] if j < len(B) else float('inf')
        
        if A_left <= B_right and B_left <= A_right:
            return max(A_left, B_left)
        elif A_left > B_right:
            r = i - 1
        else:
            l = i + 1
```

### Variation 2: Handle Multiple Arrays
```python
def findMedianMultipleArrays(arrays):
    """Find median when merging multiple sorted arrays."""
    # Merge all arrays first (or use heap)
    import heapq
    merged = list(heapq.merge(*arrays))
    
    n = len(merged)
    if n % 2:
        return merged[n // 2]
    return (merged[n // 2 - 1] + merged[n // 2]) / 2
```

### Variation 3: Return Both Middle Elements (Even Case)
```python
def findMedianElements(nums1, nums2):
    """Return tuple of (left, right) for even case."""
    A, B = nums1, nums2
    total = len(A) + len(B)
    half = (total + 1) // 2
    
    if len(B) < len(A):
        A, B = B, A
    
    l, r = 0, len(A)
    
    while l <= r:
        i = (l + r) // 2
        j = half - i
        
        A_left = A[i - 1] if i > 0 else float('-inf')
        A_right = A[i] if i < len(A) else float('inf')
        B_left = B[j - 1] if j > 0 else float('-inf')
        B_right = B[j] if j < len(B) else float('inf')
        
        if A_left <= B_right and B_left <= A_right:
            if total % 2:
                return (max(A_left, B_left),)
            return (max(A_left, B_left), min(A_right, B_right))
        elif A_left > B_right:
            r = i - 1
        else:
            l = i + 1
```

### Variation 4: Find Median Without Sorting (Single Array)
```python
def findMedianUnsorted(nums):
    """Find median in unsorted array using QuickSelect."""
    def quickSelect(arr, k):
        pivot = arr[len(arr) // 2]
        left = [x for x in arr if x < pivot]
        mid = [x for x in arr if x == pivot]
        right = [x for x in arr if x > pivot]
        
        if k < len(left):
            return quickSelect(left, k)
        elif k < len(left) + len(mid):
            return mid[0]
        else:
            return quickSelect(right, k - len(left) - len(mid))
    
    n = len(nums)
    if n % 2:
        return quickSelect(nums, n // 2)
    return (quickSelect(nums, n // 2 - 1) + quickSelect(nums, n // 2)) / 2
```

---

## Related Problems

1. **Kth Smallest Element in Sorted Matrix (LeetCode 378)** - Similar binary search
2. **Find K Pairs with Smallest Sums (LeetCode 373)** - Two sorted arrays
3. **Merge Sorted Array (LeetCode 88)** - Simpler merge problem
4. **Search in Rotated Sorted Array (LeetCode 33)** - Binary search variant
5. **Find Minimum in Rotated Sorted Array (LeetCode 153)** - Binary search
6. **Median of Stream (LeetCode 295)** - Dynamic median with heaps

---

## Summary

**Key Takeaways:**

1. **Partitioning is Key:**
   - Split both arrays into left/right halves
   - Left half has exactly (m+n)/2 elements
   - All left ≤ all right

2. **Binary Search on Smaller:**
   - Always search on smaller array for efficiency
   - Prevents index out of bounds in larger array
   - Reduces complexity to O(log(min(m,n)))

3. **Edge Case Handling:**
   - Use -∞ for when partition at start (i=0 or j=0)
   - Use +∞ for when partition at end (i=m or j=n)
   - Handles empty arrays naturally

4. **Two Validation Checks:**
   - `A_left ≤ B_right` (A's left ≤ B's right)
   - `B_left ≤ A_right` (B's left ≤ A's right)
   - Both must pass for valid partition

5. **Median Calculation:**
   - Odd: `max(A_left, B_left)` (largest on left)
   - Even: `(max(left) + min(right)) / 2` (average of middle two)

**Template:**
```python
# Always search on smaller array
if len(B) < len(A):
    A, B = B, A

half = (total + 1) // 2
l, r = 0, len(A)

while l <= r:
    i = (l + r) // 2
    j = half - i
    
    # Get boundary values with -inf/inf
    A_left = A[i-1] if i>0 else float('-inf')
    A_right = A[i] if i<len(A) else float('inf')
    B_left = B[j-1] if j>0 else float('-inf')
    B_right = B[j] if j<len(B) else float('inf')
    
    # Check valid partition
    if A_left <= B_right and B_left <= A_right:
        # Found! Calculate median
        if total % 2:
            return max(A_left, B_left)
        return (max(A_left, B_left) + min(A_right, B_right)) / 2
    elif A_left > B_right:
        r = i - 1
    else:
        l = i + 1
```

**Mental Model:**
- Think of sliding a divider along array A
- For each position in A, calculate required position in B
- The dividers create left/right halves of the merged array
- When all elements on left ≤ all on right, we found the median!
- Binary search finds the correct divider position in O(log n) time
- This is one of the most elegant applications of binary search on unsorted data!
