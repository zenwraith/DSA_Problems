# Find First and Last Position of Element in Sorted Array

**LeetCode 34** | **Difficulty:** Medium | **Category:** Binary Search, Array

---

## Problem Statement

Given an array of integers `nums` sorted in **non-decreasing order**, find the **starting and ending position** of a given `target` value.

If `target` is not found in the array, return `[-1, -1]`.

You must write an algorithm with **O(log n)** runtime complexity.

---

## Examples

### Example 1:
```
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]

Explanation:
Target 8 appears at indices 3 and 4
First occurrence: index 3
Last occurrence: index 4
```

### Example 2:
```
Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]

Explanation:
Target 6 is not in the array
```

### Example 3:
```
Input: nums = [], target = 0
Output: [-1,-1]

Explanation:
Empty array
```

### Example 4:
```
Input: nums = [1], target = 1
Output: [0,0]

Explanation:
Single element is both first and last occurrence
```

---

## Strategy

### Two Binary Searches

**Key Insight:** We need to perform **two separate binary searches**:
1. **First search:** Find the **leftmost** (first) occurrence of target
2. **Second search:** Find the **rightmost** (last) occurrence of target

**Why Not Just Expand from Found Position?**

If we find target and then expand left/right linearly, we get O(n) in worst case (all elements are target). We need O(log n) for both searches.

**Modified Binary Search:**

**For First Occurrence:**
- When we find target, **don't stop!**
- Continue searching in the **left half** to find earlier occurrence
- Keep updating the bound when we find target

**For Last Occurrence:**
- When we find target, **don't stop!**
- Continue searching in the **right half** to find later occurrence
- Keep updating the bound when we find target

**Visual Example:**
```
nums = [5, 7, 7, 7, 7, 10], target = 7

Finding FIRST occurrence:
[5, 7, 7, 7, 7, 10]
    ↑     ↑
  Found at 1, continue left
  [5, 7, 7, 7, 7, 10]
      ↑
    First at index 1 ✓

Finding LAST occurrence:
[5, 7, 7, 7, 7, 10]
          ↑     ↑
    Found at 3, continue right
    [5, 7, 7, 7, 7, 10]
              ↑
          Last at index 4 ✓
```

**Algorithm:**

```
FindFirst(target):
    Initialize bound = -1
    While search space exists:
        Find mid
        If nums[mid] == target:
            Update bound = mid
            Search left half (right = mid - 1)
        Else if nums[mid] < target:
            Search right half (left = mid + 1)
        Else:
            Search left half (right = mid - 1)
    Return bound

FindLast(target):
    Similar, but search right half when found
```

**Complexity:**
- **Time:** $O(\log n)$ - Two binary searches, each O(log n)
- **Space:** $O(1)$ - Only using pointers

---

## Solution

```python
class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        """
        Find first and last position of target using two binary searches.
        
        Approach: Perform two separate binary searches:
        1. Find leftmost (first) occurrence
        2. Find rightmost (last) occurrence
        
        Args:
            nums: Sorted array in non-decreasing order
            target: Target value to find
            
        Returns:
            [first_index, last_index] or [-1, -1] if not found
        """
        def findBound(isFirst):
            """
            Helper function to find first or last occurrence.
            
            Args:
                isFirst: True to find first occurrence, False for last
                
            Returns:
                Index of bound, or -1 if not found
            """
            left, right = 0, len(nums) - 1
            bound = -1
            
            while left <= right:
                mid = (left + right) // 2
                
                if nums[mid] == target:
                    # Found target! Update bound and continue searching
                    bound = mid
                    
                    if isFirst:
                        # For first occurrence, search left half
                        right = mid - 1
                    else:
                        # For last occurrence, search right half
                        left = mid + 1
                
                elif nums[mid] < target:
                    # Target is in right half
                    left = mid + 1
                else:
                    # Target is in left half
                    right = mid - 1
            
            return bound
        
        # Find first and last positions
        return [findBound(True), findBound(False)]
```

---

## Detailed Walkthrough

### Example: `nums = [5,7,7,8,8,10]`, `target = 8`

**Finding First Occurrence:**

| Step | Left | Right | Mid | nums[mid] | Comparison | Action | Bound |
|------|------|-------|-----|-----------|------------|--------|-------|
| 0 | 0 | 5 | 2 | 7 | 7 < 8 | left=3 | -1 |
| 1 | 3 | 5 | 4 | 8 | 8 == 8 | right=3, bound=4 | 4 |
| 2 | 3 | 3 | 3 | 8 | 8 == 8 | right=2, bound=3 | 3 |
| 3 | 3 | 2 | - | - | Stop | - | 3 |

**First occurrence:** `3` ✓

**Finding Last Occurrence:**

| Step | Left | Right | Mid | nums[mid] | Comparison | Action | Bound |
|------|------|-------|-----|-----------|------------|--------|-------|
| 0 | 0 | 5 | 2 | 7 | 7 < 8 | left=3 | -1 |
| 1 | 3 | 5 | 4 | 8 | 8 == 8 | left=5, bound=4 | 4 |
| 2 | 5 | 5 | 5 | 10 | 10 > 8 | right=4 | 4 |
| 3 | 5 | 4 | - | - | Stop | - | 4 |

**Last occurrence:** `4` ✓

**Final Answer:** `[3, 4]`

**Detailed Explanation:**

**Finding First (Step 1):**
- `mid=4, nums[4]=8` → Found target!
- Update `bound=4` to remember this position
- Search left half: `right=3` (maybe there's an earlier 8)

**Finding First (Step 2):**
- `mid=3, nums[3]=8` → Found again!
- Update `bound=3` (better position)
- Search left half: `right=2` (check for even earlier 8)

**Finding First (Step 3):**
- `left > right` → Stop
- Return `bound=3` (first occurrence)

**Finding Last (Step 1):**
- `mid=4, nums[4]=8` → Found target!
- Update `bound=4`
- Search right half: `left=5` (maybe there's a later 8)

**Finding Last (Step 2):**
- `mid=5, nums[5]=10 > 8` → Target not here
- Search left half: `right=4`

**Finding Last (Step 3):**
- `left > right` → Stop
- Return `bound=4` (last occurrence)

---

## Alternative: Single Pass with Expansion

**Not Recommended** but worth understanding:

```python
def searchRange_expansion(nums, target):
    """
    Find target, then expand left and right.
    WARNING: O(n) worst case!
    """
    # Standard binary search
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            # Found! Now expand
            first = last = mid
            
            # Expand left
            while first > 0 and nums[first - 1] == target:
                first -= 1
            
            # Expand right
            while last < len(nums) - 1 and nums[last + 1] == target:
                last += 1
            
            return [first, last]
        
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return [-1, -1]
```

**Complexity:**
- **Time:** $O(n)$ - Linear expansion in worst case (all elements = target)
- **Space:** $O(1)$

**Verdict:** Two separate binary searches are **much better** - guaranteed O(log n)!

---

## Iterative vs Recursive

### Recursive Approach:

```python
def searchRange_recursive(nums, target):
    """Recursive implementation of finding bounds."""
    def findBound(left, right, isFirst):
        if left > right:
            return -1
        
        mid = (left + right) // 2
        
        if nums[mid] == target:
            if isFirst:
                # Check if this is the first occurrence
                if mid == 0 or nums[mid - 1] != target:
                    return mid
                return findBound(left, mid - 1, isFirst)
            else:
                # Check if this is the last occurrence
                if mid == len(nums) - 1 or nums[mid + 1] != target:
                    return mid
                return findBound(mid + 1, right, isFirst)
        
        elif nums[mid] < target:
            return findBound(mid + 1, right, isFirst)
        else:
            return findBound(left, mid - 1, isFirst)
    
    n = len(nums)
    return [findBound(0, n - 1, True), findBound(0, n - 1, False)]
```

**Complexity:**
- **Time:** $O(\log n)$
- **Space:** $O(\log n)$ - Recursion stack

**Verdict:** Iterative is **slightly better** due to O(1) space!

---

## Edge Cases

### 1. Empty Array
```python
nums = [], target = 5
# Output: [-1, -1]
```

### 2. Target Not Present
```python
nums = [1, 2, 3, 4, 5], target = 6
# Output: [-1, -1]

nums = [1, 2, 3, 4, 5], target = 0
# Output: [-1, -1]
```

### 3. Single Element
```python
nums = [1], target = 1
# Output: [0, 0]

nums = [1], target = 2
# Output: [-1, -1]
```

### 4. All Same Elements
```python
nums = [5, 5, 5, 5, 5], target = 5
# Output: [0, 4]
# First at 0, last at 4
```

### 5. Target at Boundaries
```python
nums = [1, 2, 3, 4, 5], target = 1
# Output: [0, 0]
# At start

nums = [1, 2, 3, 4, 5], target = 5
# Output: [4, 4]
# At end
```

### 6. Multiple Occurrences
```python
nums = [1, 2, 2, 2, 3, 4], target = 2
# Output: [1, 3]

nums = [1, 1, 1, 2, 2, 2], target = 1
# Output: [0, 2]

nums = [1, 1, 1, 2, 2, 2], target = 2
# Output: [3, 5]
```

### 7. Two Elements
```python
nums = [1, 1], target = 1
# Output: [0, 1]

nums = [1, 2], target = 1
# Output: [0, 0]

nums = [1, 2], target = 2
# Output: [1, 1]
```

---

## Common Mistakes

### ❌ Mistake 1: Not Continuing Search After Finding Target
```python
# Wrong! Stops immediately when found
if nums[mid] == target:
    return mid  # Only finds some occurrence, not first/last
```

**Fix:** Continue searching and update bound
```python
if nums[mid] == target:
    bound = mid
    if isFirst:
        right = mid - 1  # Continue left
    else:
        left = mid + 1   # Continue right
```

### ❌ Mistake 2: Linear Expansion
```python
# Wrong! O(n) worst case
if nums[mid] == target:
    while first > 0 and nums[first-1] == target:
        first -= 1
```

**Fix:** Use two separate binary searches

### ❌ Mistake 3: Wrong Search Direction
```python
# Wrong! Searching wrong half
if nums[mid] == target:
    bound = mid
    if isFirst:
        left = mid + 1  # Should be right = mid - 1!
```

**Fix:** For first occurrence, search left; for last, search right

### ❌ Mistake 4: Not Initializing Bound
```python
# Wrong! No default value
def findBound(isFirst):
    left, right = 0, len(nums) - 1
    # Missing: bound = -1
```

**Fix:** Initialize bound to -1
```python
bound = -1
```

### ❌ Mistake 5: Returning Mid Instead of Bound
```python
# Wrong! Returning temporary variable
while left <= right:
    mid = (left + right) // 2
    if nums[mid] == target:
        # ... search continues
return mid  # mid is out of scope or wrong value!
```

**Fix:** Return the bound variable that gets updated
```python
return bound
```

---

## Interview Insights

**Question Recognition:**
- "Find first and last" + "sorted array" + "O(log n)" → Two binary searches
- "Find range of target" → Modified binary search

**Key Points to Mention:**

1. **Why two searches:**
   - "We need two separate binary searches to find the leftmost and rightmost occurrences while maintaining O(log n) complexity."

2. **How it differs from standard binary search:**
   - "Standard binary search stops when target is found. Here, we continue searching to find the boundaries."

3. **Time complexity:**
   - "Two binary searches, each O(log n), gives us O(log n) total, which is better than finding one and expanding O(n)."

4. **Key modification:**
   - "When we find target, we update our bound and continue searching in one direction—left for first, right for last."

**Follow-up Questions:**

1. **Q:** "What if we want to count occurrences instead?"
   - **A:** `count = last_index - first_index + 1` if both are found, else 0.

2. **Q:** "Can this be done with a single binary search?"
   - **A:** Not efficiently. We'd need linear expansion, giving O(n) worst case.

3. **Q:** "What if the array has duplicates but isn't sorted?"
   - **A:** Would need O(n) linear scan. Binary search requires sorted array.

4. **Q:** "How would you handle a rotated sorted array?"
   - **A:** First find rotation point, then apply this algorithm to the appropriate sorted section.

**Complexity Analysis:**
- Finding first: O(log n)
- Finding last: O(log n)
- Total: O(log n) + O(log n) = O(log n)

---

## Variations

### Variation 1: Count Occurrences
```python
def countOccurrences(nums, target):
    """Count how many times target appears."""
    def findBound(isFirst):
        left, right = 0, len(nums) - 1
        bound = -1
        
        while left <= right:
            mid = (left + right) // 2
            
            if nums[mid] == target:
                bound = mid
                if isFirst:
                    right = mid - 1
                else:
                    left = mid + 1
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        
        return bound
    
    first = findBound(True)
    if first == -1:
        return 0
    
    last = findBound(False)
    return last - first + 1
```

### Variation 2: Find K-th Occurrence
```python
def findKthOccurrence(nums, target, k):
    """Find the k-th occurrence of target (1-indexed)."""
    def findBound(isFirst):
        left, right = 0, len(nums) - 1
        bound = -1
        
        while left <= right:
            mid = (left + right) // 2
            
            if nums[mid] == target:
                bound = mid
                if isFirst:
                    right = mid - 1
                else:
                    left = mid + 1
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        
        return bound
    
    first = findBound(True)
    if first == -1:
        return -1
    
    last = findBound(False)
    count = last - first + 1
    
    if k > count or k < 1:
        return -1
    
    return first + k - 1
```

### Variation 3: Find Range of Values (Not Just Exact Match)
```python
def findRangeOfValues(nums, lower, upper):
    """Find indices where values are in [lower, upper]."""
    def findFirst(target):
        left, right = 0, len(nums) - 1
        bound = -1
        
        while left <= right:
            mid = (left + right) // 2
            
            if nums[mid] >= target:
                bound = mid
                right = mid - 1
            else:
                left = mid + 1
        
        return bound
    
    def findLast(target):
        left, right = 0, len(nums) - 1
        bound = -1
        
        while left <= right:
            mid = (left + right) // 2
            
            if nums[mid] <= target:
                bound = mid
                left = mid + 1
            else:
                right = mid - 1
        
        return bound
    
    first = findFirst(lower)
    last = findLast(upper)
    
    if first == -1 or last == -1 or first > last:
        return [-1, -1]
    
    return [first, last]
```

### Variation 4: Find Insertion Position for Range
```python
def findInsertionRange(nums, target):
    """Find where to insert target to maintain sorted order."""
    def findLeftBound():
        # Find first position >= target
        left, right = 0, len(nums)
        
        while left < right:
            mid = (left + right) // 2
            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid
        
        return left
    
    def findRightBound():
        # Find first position > target
        left, right = 0, len(nums)
        
        while left < right:
            mid = (left + right) // 2
            if nums[mid] <= target:
                left = mid + 1
            else:
                right = mid
        
        return left
    
    return [findLeftBound(), findRightBound()]
```

---

## Related Problems

1. **First Bad Version (LeetCode 278)** - Find first occurrence with API calls
2. **Search Insert Position (LeetCode 35)** - Find insertion point
3. **Find Peak Element (LeetCode 162)** - Binary search variant
4. **Search in Rotated Sorted Array (LeetCode 33)** - Combine with rotation
5. **Find K Closest Elements (LeetCode 658)** - Use binary search for bounds
6. **Count Negative Numbers in Sorted Matrix (LeetCode 1351)** - 2D binary search

---

## Summary

**Key Takeaways:**

1. **Two Separate Binary Searches:** One for first, one for last
   - Continue searching after finding target
   - Update bound and search appropriate half

2. **Algorithm Pattern:**
   ```
   FindFirst:
       When target found:
           Update bound
           Search left half (right = mid - 1)
   
   FindLast:
       When target found:
           Update bound
           Search right half (left = mid + 1)
   ```

3. **Critical Difference from Standard Binary Search:**
   - Standard: Stop when found
   - This problem: Continue to find boundaries

4. **Complexity:**
   - Time: $O(\log n)$ - Two binary searches
   - Space: $O(1)$ - Only pointers

5. **Why Not Linear Expansion:**
   - Finding target then expanding: O(n) worst case
   - Two binary searches: O(log n) always

**Template:**
```python
def searchRange(nums, target):
    def findBound(isFirst):
        left, right = 0, len(nums) - 1
        bound = -1
        
        while left <= right:
            mid = (left + right) // 2
            
            if nums[mid] == target:
                bound = mid
                if isFirst:
                    right = mid - 1  # Continue left
                else:
                    left = mid + 1   # Continue right
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        
        return bound
    
    return [findBound(True), findBound(False)]
```

**Key Insight:** The elegance of this solution lies in recognizing that we need two independent searches, each with a slight modification: continuing the search after finding the target to locate the exact boundaries. This maintains the O(log n) efficiency while solving a more complex problem than standard binary search!
