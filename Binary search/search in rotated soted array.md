# Search in Rotated Sorted Array

**LeetCode 33** | **Difficulty:** Medium | **Category:** Binary Search, Array

---

## Problem Statement

There is an integer array `nums` sorted in ascending order (with **distinct** values).

Prior to being passed to your function, `nums` is **possibly rotated** at an unknown pivot index `k` (`1 <= k < nums.length`) such that the resulting array is `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]` (**0-indexed**). For example, `[0,1,2,4,5,6,7]` might be rotated at pivot index `3` and become `[4,5,6,7,0,1,2]`.

Given the array `nums` **after** the possible rotation and an integer `target`, return the **index** of `target` if it is in `nums`, or `-1` if it is not in `nums`.

You must write an algorithm with **O(log n)** runtime complexity.

---

## Examples

### Example 1:
```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Explanation:
Original sorted array: [0,1,2,4,5,6,7]
Rotated at index 4: [4,5,6,7,0,1,2]
Target 0 is at index 4
```

### Example 2:
```
Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1

Explanation:
Target 3 is not in the array
```

### Example 3:
```
Input: nums = [1], target = 0
Output: -1

Explanation:
Single element array, target not found
```

### Example 4:
```
Input: nums = [4,5,6,7,0,1,2], target = 5
Output: 1

Explanation:
Target 5 is at index 1
```

---

## Strategy

### Modified Binary Search

**Key Insight:** Even though the array is rotated, **at least one half** of the array (from mid) is **always sorted**. We can use this property to determine which half to search.

**Visual Understanding:**

```
Original: [0, 1, 2, 4, 5, 6, 7]
Rotated:  [4, 5, 6, 7, 0, 1, 2]
           ↑        ↑  ↑     ↑
         left     mid        right

At any mid point:
- Either left half [left...mid] is sorted
- Or right half [mid...right] is sorted
- Never both unsorted!
```

**Algorithm Logic:**

1. **Find mid point** as in normal binary search
2. **Check if target found** at mid
3. **Identify which half is sorted:**
   - If `nums[left] <= nums[mid]` → Left half is sorted
   - Else → Right half is sorted
4. **Determine if target is in sorted half:**
   - If in sorted half: Search that half
   - Else: Search the other half
5. **Repeat** until found or search space exhausted

**Why This Works:**

When array is rotated, exactly one of the two halves from mid is sorted:
- **Left sorted:** `nums[left] <= nums[mid]` (monotonic increase)
- **Right sorted:** `nums[mid] <= nums[right]` (monotonic increase)

Once we know which half is sorted, we can check if target falls in that sorted range using simple comparisons.

**Decision Tree:**
```
Is nums[mid] == target? → Return mid

Is left half sorted? (nums[left] <= nums[mid])
├─ Yes: Is target in [nums[left], nums[mid])? 
│   ├─ Yes: Search left (right = mid - 1)
│   └─ No: Search right (left = mid + 1)
└─ No: Right half must be sorted
    ├─ Is target in (nums[mid], nums[right]]?
    │   ├─ Yes: Search right (left = mid + 1)
    │   └─ No: Search left (right = mid - 1)
```

**Complexity:**
- **Time:** $O(\log n)$ - Binary search with modified condition
- **Space:** $O(1)$ - Only using pointers

---

## Solution

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        """
        Search for target in rotated sorted array using modified binary search.
        
        Key insight: At least one half is always sorted in rotated array.
        We identify the sorted half and check if target lies in it.
        
        Args:
            nums: Rotated sorted array with distinct values
            target: Target value to search for
            
        Returns:
            Index of target if found, -1 otherwise
        """
        left, right = 0, len(nums) - 1
        
        while left <= right:
            mid = (left + right) // 2
            
            # Found the target
            if nums[mid] == target:
                return mid
            
            # Determine which half is sorted
            # Case 1: Left half is sorted [left...mid]
            if nums[left] <= nums[mid]:
                # Check if target is in the sorted left half
                if nums[left] <= target < nums[mid]:
                    right = mid - 1  # Target is in sorted left half
                else:
                    left = mid + 1   # Target is in right half (possibly unsorted)
            
            # Case 2: Right half is sorted [mid...right]
            else:
                # Check if target is in the sorted right half
                if nums[mid] < target <= nums[right]:
                    left = mid + 1   # Target is in sorted right half
                else:
                    right = mid - 1  # Target is in left half (possibly unsorted)
        
        # Target not found
        return -1
```

---

## Detailed Walkthrough

### Example: `nums = [4,5,6,7,0,1,2]`, `target = 0`

**Initial State:**
```
Array: [4, 5, 6, 7, 0, 1, 2]
Index:  0  1  2  3  4  5  6
Target: 0
```

**Step-by-Step Execution:**

| Step | Left | Right | Mid | nums[mid] | Sorted Half | Target Range Check | Action | Reason |
|------|------|-------|-----|-----------|-------------|-------------------|--------|--------|
| 1 | 0 | 6 | 3 | 7 | Left (4≤7) | 0 in [4,7)? No | left=4 | Target not in sorted left |
| 2 | 4 | 6 | 5 | 1 | Right (1≤2) | 0 in (1,2]? No | right=4 | Target not in sorted right |
| 3 | 4 | 4 | 4 | 0 | - | 0==0? Yes | return 4 | Found! |

**Final Answer:** `4`

**Detailed Explanation:**

**Step 1:** `left=0, right=6, mid=3`
- `nums[mid] = 7`
- Check if left half sorted: `nums[0]=4 <= nums[3]=7` → Yes, left is sorted
- Is target in `[4, 7)`? No, `0 < 4`
- Search right half: `left = 4`

**Step 2:** `left=4, right=6, mid=5`
- `nums[mid] = 1`
- Check if left half sorted: `nums[4]=0 <= nums[5]=1` → No, so right is sorted
- Is target in `(1, 2]`? No, `0 < 1`
- Search left half: `right = 4`

**Step 3:** `left=4, right=4, mid=4`
- `nums[mid] = 0 == target` → Found at index 4!

---

### Example 2: `nums = [4,5,6,7,0,1,2]`, `target = 5`

| Step | Left | Right | Mid | nums[mid] | Sorted Half | Check | Action |
|------|------|-------|-----|-----------|-------------|-------|--------|
| 1 | 0 | 6 | 3 | 7 | Left (4≤7) | 5 in [4,7)? Yes | right=2 |
| 2 | 0 | 2 | 1 | 5 | - | 5==5? Yes | return 1 |

**Final Answer:** `1`

---

## Why At Least One Half is Always Sorted

**Proof:**

In a rotated sorted array, there exists a pivot point where the rotation occurs.

```
Case 1: Pivot is in right half
[... sorted ...][mid][... rotation ...]
    ↑           ↑
  Left half is fully sorted

Example: [4, 5, 6, 7, 0, 1, 2]
                    ↑ pivot
         [4,5,6,7] is sorted

Case 2: Pivot is in left half
[... rotation ...][mid][... sorted ...]
                   ↑           ↑
                Right half is fully sorted

Example: [6, 7, 0, 1, 2, 4, 5]
              ↑ pivot
         [2,4,5] is sorted (right of mid=1)
```

**Key Properties:**
1. Original array was fully sorted
2. Rotation creates exactly one "break" point (pivot)
3. Any mid point either includes the break or doesn't
4. The half without the break is sorted

**Check for Sorted Half:**
- **Left sorted:** `nums[left] <= nums[mid]` (no break between them)
- **Right sorted:** `nums[mid] <= nums[right]` (no break between them)

---

## Edge Cases

### 1. No Rotation (Already Sorted)
```python
nums = [1, 2, 3, 4, 5], target = 3
# Output: 2
# Works like standard binary search
```

### 2. Single Element
```python
nums = [1], target = 1
# Output: 0

nums = [1], target = 2
# Output: -1
```

### 3. Two Elements
```python
nums = [3, 1], target = 1
# Output: 1
# Rotated at index 1

nums = [1, 3], target = 3
# Output: 1
# No rotation
```

### 4. Target at Boundaries
```python
nums = [4,5,6,7,0,1,2], target = 4
# Output: 0
# Target at left boundary

nums = [4,5,6,7,0,1,2], target = 2
# Output: 6
# Target at right boundary
```

### 5. Target at Pivot
```python
nums = [4,5,6,7,0,1,2], target = 7
# Output: 3
# Last element of first sorted part

nums = [4,5,6,7,0,1,2], target = 0
# Output: 4
# First element of second sorted part
```

### 6. Complete Rotation
```python
nums = [2, 3, 4, 5, 6, 7, 0, 1], target = 0
# Output: 6
# Pivot near end
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Sorted Half Check
```python
# Wrong! Using < instead of <=
if nums[left] < nums[mid]:
```

**Fix:** Use `<=` to handle case where left == mid
```python
if nums[left] <= nums[mid]:
```

### ❌ Mistake 2: Incorrect Target Range Check
```python
# Wrong! Using <= on both sides
if nums[left] <= target <= nums[mid]:
```

**Fix:** Exclude mid from range (already checked)
```python
if nums[left] <= target < nums[mid]:
```

### ❌ Mistake 3: Not Handling Right Half Correctly
```python
# Wrong! Copy-paste error
if nums[mid] <= target <= nums[right]:
```

**Fix:** Exclude mid from range
```python
if nums[mid] < target <= nums[right]:
```

### ❌ Mistake 4: Swapping Search Direction
```python
# Wrong! Moving in wrong direction
if nums[left] <= target < nums[mid]:
    left = mid + 1  # Should be right = mid - 1!
```

**Fix:** Search the correct half
```python
if nums[left] <= target < nums[mid]:
    right = mid - 1  # Target is in left half
```

### ❌ Mistake 5: Not Returning -1
```python
# Wrong! Returning 0 or None
return 0
```

**Fix:** Return -1 when not found
```python
return -1
```

---

## Interview Insights

**Question Recognition:**
- "Rotated sorted array" + "O(log n)" → Modified binary search
- "Find element" in rotated array → Check sorted half

**Key Points to Mention:**

1. **Why binary search still works:**
   - "Even with rotation, at least one half is always sorted, allowing us to determine search direction."

2. **How to identify sorted half:**
   - "Compare nums[left] with nums[mid]. If nums[left] <= nums[mid], the left half is sorted; otherwise, the right half is sorted."

3. **Decision making:**
   - "Once we know which half is sorted, we check if target falls in that sorted range using simple comparisons."

4. **Time complexity:**
   - "Still O(log n) like standard binary search, just with additional logic to handle rotation."

**Follow-up Questions:**

1. **Q:** "What if the array has duplicates?"
   - **A:** Becomes harder (LeetCode 81). In worst case, we might need to skip duplicates at boundaries, leading to O(n) time.

2. **Q:** "Can you find the minimum element in rotated array?"
   - **A:** Yes! (LeetCode 153). Compare mid with right; if nums[mid] > nums[right], minimum is in right half.

3. **Q:** "How would you find the rotation point?"
   - **A:** Similar to finding minimum. The rotation point is where nums[i] > nums[i+1].

4. **Q:** "What if we need to find the number of rotations?"
   - **A:** Number of rotations equals the index of minimum element.

**Common Variations:**
- Search in Rotated Sorted Array II (with duplicates)
- Find Minimum in Rotated Sorted Array
- Find Peak Element
- Search in Rotated Sorted Array (different rotation)

---

## Variations

### Variation 1: Find Minimum in Rotated Array
```python
def findMin(nums):
    """LeetCode 153: Find minimum element."""
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        # If mid > right, minimum is in right half
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            # Minimum is in left half or at mid
            right = mid
    
    return nums[left]
```

### Variation 2: Search with Duplicates
```python
def search_with_duplicates(nums, target):
    """LeetCode 81: Search in rotated array with duplicates."""
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            return True
        
        # Handle duplicates: can't determine which half is sorted
        if nums[left] == nums[mid] == nums[right]:
            left += 1
            right -= 1
            continue
        
        # Same logic as original problem
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return False
```

### Variation 3: Find Rotation Count
```python
def findRotationCount(nums):
    """Count number of rotations (index of minimum)."""
    left, right = 0, len(nums) - 1
    
    while left < right:
        # Already sorted, no rotation
        if nums[left] < nums[right]:
            return left
        
        mid = (left + right) // 2
        
        # Check if mid+1 is the minimum
        if mid < len(nums) - 1 and nums[mid] > nums[mid + 1]:
            return mid + 1
        
        # Check if mid is the minimum
        if mid > 0 and nums[mid] < nums[mid - 1]:
            return mid
        
        # Decide which half has the minimum
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    
    return left
```

### Variation 4: Find Target Range
```python
def searchRange(nums, target):
    """Find first and last position of target in rotated array."""
    def findFirst(nums, target):
        left, right, result = 0, len(nums) - 1, -1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                result = mid
                right = mid - 1  # Continue searching left
            elif nums[left] <= nums[mid]:
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1
            else:
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1
        return result
    
    def findLast(nums, target):
        left, right, result = 0, len(nums) - 1, -1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                result = mid
                left = mid + 1  # Continue searching right
            elif nums[left] <= nums[mid]:
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1
            else:
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1
        return result
    
    return [findFirst(nums, target), findLast(nums, target)]
```

---

## Related Problems

1. **Search in Rotated Sorted Array II (LeetCode 81)** - With duplicates, worst case O(n)
2. **Find Minimum in Rotated Sorted Array (LeetCode 153)** - Find pivot/minimum
3. **Find Minimum in Rotated Sorted Array II (LeetCode 154)** - With duplicates
4. **Find Peak Element (LeetCode 162)** - Similar binary search modification
5. **Search a 2D Matrix II (LeetCode 240)** - 2D binary search variant
6. **Find First and Last Position (LeetCode 34)** - Binary search for range

---

## Summary

**Key Takeaways:**

1. **Core Insight:** At least one half is always sorted in rotated array
   - Check which half is sorted: `nums[left] <= nums[mid]`
   - Use sorted half to determine search direction

2. **Algorithm Pattern:**
   ```
   While search space exists:
       Find mid
       If mid == target: return mid
       
       If left half sorted:
           If target in sorted left: search left
           Else: search right
       Else (right half sorted):
           If target in sorted right: search right
           Else: search left
   ```

3. **Critical Checks:**
   - Sorted half: `nums[left] <= nums[mid]`
   - Target in left sorted: `nums[left] <= target < nums[mid]`
   - Target in right sorted: `nums[mid] < target <= nums[right]`

4. **Complexity:**
   - Time: $O(\log n)$ - Binary search with constant work per iteration
   - Space: $O(1)$ - Only pointers

5. **Key Difference from Standard Binary Search:**
   - Standard: Compare target with mid
   - Rotated: First identify sorted half, then check if target in that half

**Template:**
```python
left, right = 0, len(nums) - 1

while left <= right:
    mid = (left + right) // 2
    
    if nums[mid] == target:
        return mid
    
    # Identify sorted half
    if nums[left] <= nums[mid]:  # Left sorted
        if nums[left] <= target < nums[mid]:
            right = mid - 1  # Search left
        else:
            left = mid + 1   # Search right
    else:  # Right sorted
        if nums[mid] < target <= nums[right]:
            left = mid + 1   # Search right
        else:
            right = mid - 1  # Search left

return -1
```

**Mental Model:** Think of the rotated array as two sorted subarrays concatenated. At each step, identify which subarray (or part of it) is fully sorted, and use that information to decide which direction to search. The beauty is that we maintain O(log n) complexity despite the rotation!
