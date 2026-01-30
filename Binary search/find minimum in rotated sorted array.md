# Find Minimum in Rotated Sorted Array

**LeetCode 153** | **Difficulty:** Medium | **Category:** Binary Search, Array

---

## Problem Statement

Suppose an array of length `n` sorted in ascending order is **rotated** between `1` and `n` times. For example, the array `nums = [0,1,2,4,5,6,7]` might become:

- `[4,5,6,7,0,1,2]` if it was rotated `4` times.
- `[0,1,2,4,5,6,7]` if it was rotated `7` times.

Notice that **rotating** an array `[a[0], a[1], a[2], ..., a[n-1]]` 1 time results in the array `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]`.

Given the sorted rotated array `nums` of **unique** elements, return the **minimum element** of this array.

You must write an algorithm that runs in **O(log n)** time.

---

## Examples

### Example 1:
```
Input: nums = [3,4,5,1,2]
Output: 1

Explanation:
Original array: [1,2,3,4,5]
Rotated 3 times: [3,4,5,1,2]
Minimum: 1
```

### Example 2:
```
Input: nums = [4,5,6,7,0,1,2]
Output: 0

Explanation:
Original array: [0,1,2,4,5,6,7]
Rotated 4 times: [4,5,6,7,0,1,2]
Minimum: 0
```

### Example 3:
```
Input: nums = [11,13,15,17]
Output: 11

Explanation:
Original array (no rotation): [11,13,15,17]
Minimum: 11
```

### Example 4:
```
Input: nums = [2,1]
Output: 1

Explanation:
Rotated once: [2,1]
Minimum: 1
```

---

## Strategy

### Modified Binary Search

**Key Insight:** The minimum element is the **only element** that is smaller than its previous element. It's also the **pivot point** where the rotation occurred.

**Visual Understanding:**

```
Original: [0, 1, 2, 4, 5, 6, 7]
Rotated:  [4, 5, 6, 7, 0, 1, 2]
                       ↑
                   Minimum (pivot)

Characteristics:
- Left side [4,5,6,7]: All elements > minimum
- Right side [0,1,2]: Contains minimum
- nums[mid] > nums[right] → Minimum in right half
- nums[mid] ≤ nums[right] → Minimum in left half or at mid
```

**Algorithm Logic:**

The key comparison is: **`nums[mid]` vs `nums[right]`**

1. **If `nums[mid] > nums[right]`:**
   - Array is rotated in this section
   - Minimum must be to the **right** of mid
   - Example: `[4, 5, 6, 7, 0, 1, 2]`, mid=7, right=2, 7>2
   - Search right: `left = mid + 1`

2. **If `nums[mid] ≤ nums[right]`:**
   - This section is sorted (or mid is minimum)
   - Minimum is to the **left** of mid or is mid itself
   - Example: `[4, 5, 6, 7, 0, 1, 2]`, mid=0, right=2, 0<2
   - Search left: `right = mid`

**Why Compare with Right, Not Left?**

Comparing with right gives us clear information:
- `nums[mid] > nums[right]` → Rotation is in right half
- `nums[mid] ≤ nums[right]` → Right half is sorted, min is left or mid

**Visual Decision Tree:**
```
At mid position:
├─ nums[mid] > nums[right]?
│   └─ Yes: Minimum in (mid, right]
│       Action: left = mid + 1
└─ No: Minimum in [left, mid]
    Action: right = mid
```

**Loop Condition:** Use `left < right` (not `<=`) because:
- We're looking for a position, not exact value
- When `left == right`, we've found the minimum
- No need for equality check in loop

**Complexity:**
- **Time:** $O(\log n)$ - Binary search
- **Space:** $O(1)$ - Only using pointers

---

## Solution

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        """
        Find minimum element in rotated sorted array using binary search.
        
        Key insight: Compare mid with right to determine which half contains minimum.
        - If nums[mid] > nums[right]: minimum is in right half
        - If nums[mid] <= nums[right]: minimum is in left half or at mid
        
        Args:
            nums: Rotated sorted array with unique elements
            
        Returns:
            Minimum element in the array
        """
        left, right = 0, len(nums) - 1
        
        # Continue until left and right converge
        while left < right:
            mid = (left + right) // 2
            
            # If mid element is greater than rightmost element,
            # the rotation point (minimum) must be to the right
            if nums[mid] > nums[right]:
                # Minimum is in right half
                left = mid + 1
            else:
                # nums[mid] <= nums[right]
                # Right half is sorted, minimum is in left half or at mid
                right = mid
        
        # When left == right, we've found the minimum
        return nums[left]
```

---

## Detailed Walkthrough

### Example 1: `nums = [4,5,6,7,0,1,2]`

**Initial State:**
```
Array: [4, 5, 6, 7, 0, 1, 2]
Index:  0  1  2  3  4  5  6
Target: Find minimum (0)
```

**Step-by-Step Execution:**

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action | Reason |
|------|------|-------|-----|-----------|-------------|---------|--------|--------|
| 1 | 0 | 6 | 3 | 7 | 2 | 7 > 2 | left=4 | Min in right half |
| 2 | 4 | 6 | 5 | 1 | 2 | 1 < 2 | right=5 | Min in left half or mid |
| 3 | 4 | 5 | 4 | 0 | 1 | 0 < 1 | right=4 | Min in left half or mid |
| 4 | 4 | 4 | - | - | - | - | Stop | left == right |

**Final Answer:** `nums[4] = 0` ✓

**Detailed Explanation:**

**Step 1:** `left=0, right=6, mid=3`
- `nums[3]=7, nums[6]=2`
- `7 > 2` → Array is rotated, minimum in right half
- `left = 4`

**Step 2:** `left=4, right=6, mid=5`
- `nums[5]=1, nums[6]=2`
- `1 < 2` → Right portion sorted, minimum in left half or at mid
- `right = 5`

**Step 3:** `left=4, right=5, mid=4`
- `nums[4]=0, nums[5]=1`
- `0 < 1` → Right portion sorted, minimum in left half or at mid
- `right = 4`

**Step 4:** `left=4, right=4`
- `left == right` → Found minimum!
- Return `nums[4] = 0`

---

### Example 2: `nums = [3,4,5,1,2]`

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 1 | 0 | 4 | 2 | 5 | 2 | 5 > 2 | left=3 |
| 2 | 3 | 4 | 3 | 1 | 2 | 1 < 2 | right=3 |
| 3 | 3 | 3 | - | - | - | - | Stop |

**Final Answer:** `nums[3] = 1` ✓

---

### Example 3: `nums = [11,13,15,17]` (No Rotation)

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 1 | 0 | 3 | 1 | 13 | 17 | 13 < 17 | right=1 |
| 2 | 0 | 1 | 0 | 11 | 13 | 11 < 13 | right=0 |
| 3 | 0 | 0 | - | - | - | - | Stop |

**Final Answer:** `nums[0] = 11` ✓

---

## Why Not Compare Mid with Left?

**Comparing with left doesn't work cleanly:**

```
Example: [4, 5, 6, 7, 0, 1, 2]
         mid=7

If nums[mid] > nums[left] (7 > 4):
    Could mean: Left is sorted OR mid is in rotated part
    Ambiguous!

If nums[mid] < nums[left]:
    Definitely rotated, min in left half
    
But this creates ambiguity in first case!
```

**Comparing with right is clearer:**

```
If nums[mid] > nums[right]:
    Definitely rotated, min in right half (clear!)

If nums[mid] <= nums[right]:
    Right is sorted, min in left half or mid (clear!)
```

---

## Edge Cases

### 1. No Rotation (Already Sorted)
```python
nums = [1, 2, 3, 4, 5]
# Output: 1
# Minimum at start
```

### 2. Single Element
```python
nums = [1]
# Output: 1
# Only one element
```

### 3. Two Elements (Rotated)
```python
nums = [2, 1]
# Output: 1

nums = [1, 2]
# Output: 1
```

### 4. Rotated Once
```python
nums = [7, 0, 1, 2, 4, 5, 6]
# Output: 0
# Minimum at index 1
```

### 5. Rotated to Last Position
```python
nums = [2, 3, 4, 5, 6, 7, 1]
# Output: 1
# Minimum at end
```

### 6. All Same Elements (Edge Case for LeetCode 154)
```python
# Not in this problem (unique elements)
# But variation: nums = [1, 1, 1, 1]
# Output: 1
```

### 7. Large Array
```python
nums = [15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14]
# Output: 1
# Works efficiently even with many elements
```

---

## Common Mistakes

### ❌ Mistake 1: Using `left <= right`
```python
# Wrong! Causes infinite loop or wrong answer
while left <= right:
    mid = (left + right) // 2
    # ...
```

**Fix:** Use `left < right` since we're converging to a position
```python
while left < right:
    mid = (left + right) // 2
```

### ❌ Mistake 2: Comparing Mid with Left
```python
# Wrong! Creates ambiguity
if nums[mid] > nums[left]:
    # Not clear which half has minimum
```

**Fix:** Compare with right for clarity
```python
if nums[mid] > nums[right]:
    left = mid + 1
```

### ❌ Mistake 3: Including Mid When Moving Left
```python
# Wrong! Might skip the minimum
if nums[mid] > nums[right]:
    left = mid  # Should be mid + 1
```

**Fix:** Move left pointer past mid
```python
if nums[mid] > nums[right]:
    left = mid + 1
```

### ❌ Mistake 4: Not Including Mid When Moving Right
```python
# Wrong! Might skip the minimum at mid
else:
    right = mid - 1  # Should be mid
```

**Fix:** Keep mid in consideration
```python
else:
    right = mid
```

### ❌ Mistake 5: Returning Wrong Value
```python
# Wrong! Returning index instead of value
return left
```

**Fix:** Return the value at that index
```python
return nums[left]
```

---

## Interview Insights

**Question Recognition:**
- "Rotated sorted array" + "find minimum" + "O(log n)" → Modified binary search
- "Pivot point" in rotation → Same as finding minimum

**Key Points to Mention:**

1. **Why compare mid with right:**
   - "Comparing nums[mid] with nums[right] clearly indicates which half contains the minimum."

2. **Loop condition:**
   - "We use left < right instead of left <= right because we're converging to a position, not searching for exact value match."

3. **Why mid+1 vs mid:**
   - "When minimum is in right half, we know mid itself isn't minimum (since nums[mid] > nums[right]), so we can skip it with left=mid+1."
   - "When minimum might be at mid, we keep it with right=mid."

4. **Time complexity:**
   - "O(log n) because we eliminate half the search space each iteration, just like standard binary search."

**Follow-up Questions:**

1. **Q:** "What if the array has duplicates?"
   - **A:** LeetCode 154. When nums[mid] == nums[right], we can't determine direction, need to decrement right by 1, worst case O(n).

2. **Q:** "How would you find the rotation count?"
   - **A:** The index of minimum element equals the rotation count.

3. **Q:** "Can you use this to search for a target?"
   - **A:** Yes! Find minimum first to identify rotation point, then decide which sorted half to search (LeetCode 33).

4. **Q:** "What if you need to return the index, not value?"
   - **A:** Simply return `left` instead of `nums[left]`.

**Complexity Analysis:**
- Best case: O(1) - Already sorted (but still need to verify)
- Worst case: O(log n) - Halve search space each time
- Average case: O(log n)

---

## Variations

### Variation 1: Find Minimum with Duplicates
```python
def findMin_duplicates(nums):
    """LeetCode 154: Find minimum with duplicates."""
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        if nums[mid] > nums[right]:
            # Minimum in right half
            left = mid + 1
        elif nums[mid] < nums[right]:
            # Minimum in left half or at mid
            right = mid
        else:
            # nums[mid] == nums[right]
            # Can't determine which half, shrink right
            right -= 1
    
    return nums[left]
```

**Complexity:** O(n) worst case when all elements are same

### Variation 2: Find Rotation Count
```python
def findRotationCount(nums):
    """Find number of times array was rotated."""
    left, right = 0, len(nums) - 1
    
    while left < right:
        # Already sorted, no rotation
        if nums[left] < nums[right]:
            return left
        
        mid = (left + right) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    
    return left  # Index of minimum = rotation count
```

### Variation 3: Find Maximum in Rotated Array
```python
def findMax(nums):
    """Find maximum element (element before minimum)."""
    # Find minimum index
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    
    min_idx = left
    
    # Maximum is just before minimum (circularly)
    max_idx = (min_idx - 1) % len(nums)
    return nums[max_idx]
```

### Variation 4: Check if Array is Rotated Sorted
```python
def isRotatedSorted(nums):
    """Check if array is a rotated version of sorted array."""
    n = len(nums)
    rotation_count = 0
    
    # Count rotation points (where nums[i] > nums[i+1])
    for i in range(n):
        if nums[i] > nums[(i + 1) % n]:
            rotation_count += 1
    
    # Valid rotated sorted array has exactly 0 or 1 rotation point
    return rotation_count <= 1
```

---

## Related Problems

1. **Search in Rotated Sorted Array (LeetCode 33)** - Search for target after finding minimum
2. **Find Minimum in Rotated Sorted Array II (LeetCode 154)** - With duplicates
3. **Find Peak Element (LeetCode 162)** - Similar binary search pattern
4. **Search in Rotated Sorted Array II (LeetCode 81)** - Search with duplicates
5. **Find First and Last Position (LeetCode 34)** - Modified binary search
6. **Single Element in Sorted Array (LeetCode 540)** - Binary search variant

---

## Comparison with Other Approaches

### Approach 1: Linear Search (Not Recommended)
```python
def findMin_linear(nums):
    return min(nums)
```
- **Time:** O(n) - Too slow
- **Space:** O(1)

### Approach 2: Two Pointer Scan (Not Recommended)
```python
def findMin_scan(nums):
    for i in range(len(nums) - 1):
        if nums[i] > nums[i + 1]:
            return nums[i + 1]
    return nums[0]
```
- **Time:** O(n) - Doesn't leverage sorted property
- **Space:** O(1)

### Approach 3: Binary Search (Optimal)
- **Time:** O(log n) - Exploits sorted structure
- **Space:** O(1)

---

## Summary

**Key Takeaways:**

1. **Core Comparison:** `nums[mid]` vs `nums[right]`
   - `nums[mid] > nums[right]` → Minimum in right half
   - `nums[mid] ≤ nums[right]` → Minimum in left half or at mid

2. **Loop Condition:** Use `left < right` (not `<=`)
   - Converging to position, not checking exact match
   - Stop when `left == right`

3. **Pointer Movement:**
   - Right half: `left = mid + 1` (skip mid, it's not minimum)
   - Left half: `right = mid` (keep mid, might be minimum)

4. **Why This Works:**
   - Minimum is the rotation point
   - Comparing with right tells us which half is rotated
   - Each iteration eliminates half the search space

5. **Complexity:**
   - Time: $O(\log n)$ - Binary search
   - Space: $O(1)$ - Only pointers

**Template:**
```python
left, right = 0, len(nums) - 1

while left < right:
    mid = (left + right) // 2
    
    if nums[mid] > nums[right]:
        # Minimum in right half
        left = mid + 1
    else:
        # Minimum in left half or at mid
        right = mid

return nums[left]
```

**Mental Model:** 
- Visualize the rotated array as two sorted subarrays
- The minimum is at the "break point" where rotation occurred
- By comparing mid with right, we determine which subarray contains this break point
- The comparison gives us a clear decision: go right (rotation is there) or go left (rotation isn't on right side)
- This elegant approach maintains O(log n) by halving the search space each iteration!



The Strategy: Identifying the Unshelved Side
In a rotated sorted array, the minimum is the only element whose left neighbor is greater than it. We can find it by comparing our mid value to the rightmost value of our current search space.

The Logic:

If nums[mid] > nums[right]: This means the "inflection point" (the pivot) must be to the right of mid. The left half is perfectly sorted and contains only larger numbers.

Action: left = mid + 1

If nums[mid] < nums[right]: This means the pivot is either at mid or to the left of mid. The right half is sorted, so the minimum can't be further right than mid.

Action: right = mid (Note: We don't do mid - 1 because mid itself could be the minimum).