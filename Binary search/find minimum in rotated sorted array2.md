# Find Minimum in Rotated Sorted Array II

**LeetCode 154** | **Difficulty:** Hard | **Category:** Binary Search, Array

---

## Problem Statement

Suppose an array of length `n` sorted in ascending order is **rotated** between `1` and `n` times. For example, the array `nums = [0,1,4,4,5,6,7]` might become:

- `[4,5,6,7,0,1,4]` if it was rotated `4` times.
- `[0,1,4,4,5,6,7]` if it was rotated `7` times.

Notice that **rotating** an array `[a[0], a[1], a[2], ..., a[n-1]]` 1 time results in the array `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]`.

Given the sorted rotated array `nums` that **may contain duplicates**, return the **minimum element** of this array.

You must **decrease the overall operation steps** as much as possible.

---

## Examples

### Example 1:
```
Input: nums = [1,3,5]
Output: 1
```

### Example 2:
```
Input: nums = [2,2,2,0,1]
Output: 0

Explanation:
Array with duplicates, minimum is 0
```

### Example 3:
```
Input: nums = [3,3,1,3]
Output: 1

Explanation:
[3,3,1,3] rotated, minimum is 1
```

### Example 4:
```
Input: nums = [10,1,10,10,10]
Output: 1

Explanation:
Multiple duplicates of 10, minimum is 1
```

---

## Strategy

### Modified Binary Search with Duplicate Handling

**Key Difference from Part I:** The presence of **duplicates** creates an **ambiguous case** when `nums[mid] == nums[right]`.

**Visual Understanding:**

```
Without duplicates: [4, 5, 6, 7, 0, 1, 2]
- nums[mid] vs nums[right] clearly indicates which half has minimum

With duplicates: [1, 3, 5, 1, 1, 1, 1]
                       ↑         ↑
                     mid=5    right=1
- If mid == right, we can't determine which half has minimum!
```

**Three Cases:**

1. **`nums[mid] > nums[right]`:**
   - Minimum must be in right half (same as Part I)
   - Action: `left = mid + 1`

2. **`nums[mid] < nums[right]`:**
   - Minimum in left half or at mid (same as Part I)
   - Action: `right = mid`

3. **`nums[mid] == nums[right]` (NEW - Ambiguous):**
   - Can't determine which half has minimum
   - Could be: `[1, 3, 1]` (min on left) or `[3, 1, 3]` (min on right)
   - Action: **Shrink search space by 1**: `right -= 1`

**Why Shrink Right by 1?**

When `nums[mid] == nums[right]`:
- We know `nums[right]` is not unique (at least one duplicate at mid)
- Even if `nums[right]` is the minimum, we have a copy at mid
- Safe to exclude `right` and continue searching

**Visual Decision Tree:**
```
At mid position:
├─ nums[mid] > nums[right]?
│   └─ Yes: Minimum in (mid, right]
│       Action: left = mid + 1
├─ nums[mid] < nums[right]?
│   └─ Yes: Minimum in [left, mid]
│       Action: right = mid
└─ nums[mid] == nums[right]? (AMBIGUOUS)
    └─ Yes: Can't determine, shrink right
        Action: right -= 1
```

**Complexity:**
- **Time:** 
  - Best case: $O(\log n)$ - Few or no duplicates
  - Worst case: $O(n)$ - All elements same (must check each)
  - Average case: $O(\log n)$ - Typical rotated array
- **Space:** $O(1)$ - Only using pointers

---

## Solution

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        """
        Find minimum element in rotated sorted array with duplicates.
        
        Key insight: Handle three cases instead of two:
        1. nums[mid] > nums[right]: minimum in right half
        2. nums[mid] < nums[right]: minimum in left half or at mid
        3. nums[mid] == nums[right]: ambiguous, shrink search space
        
        Args:
            nums: Rotated sorted array that may contain duplicates
            
        Returns:
            Minimum element in the array
        """
        left, right = 0, len(nums) - 1
        
        while left < right:
            mid = left + (right - left) // 2
            
            # Case 1: Mid element is greater than rightmost element
            # Minimum must be in right half
            if nums[mid] > nums[right]:
                left = mid + 1
            
            # Case 2: Mid element is less than rightmost element
            # Minimum is in left half or at mid
            elif nums[mid] < nums[right]:
                right = mid
            
            # Case 3: nums[mid] == nums[right] (AMBIGUOUS)
            # Can't determine which half has minimum
            # Shrink search space by excluding right
            else:
                right -= 1
        
        # When left == right, we've found the minimum
        return nums[left]
```

---

## Detailed Walkthrough

### Example 1: `nums = [2,2,2,0,1]`

**Step-by-Step Execution:**

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action | Reason |
|------|------|-------|-----|-----------|-------------|---------|--------|--------|
| 0 | 0 | 4 | 2 | 2 | 1 | 2 > 1 | left=3 | Min in right half |
| 1 | 3 | 4 | 3 | 0 | 1 | 0 < 1 | right=3 | Min in left or mid |
| 2 | 3 | 3 | - | - | - | - | Stop | left == right |

**Final Answer:** `nums[3] = 0` ✓

**Explanation:**
- Step 0: `nums[2]=2 > nums[4]=1`, so minimum must be in right half
- Step 1: `nums[3]=0 < nums[4]=1`, so minimum is in left or at mid
- Step 2: Converged to index 3, return 0

---

### Example 2: `nums = [3,3,1,3]` (Duplicates Create Ambiguity)

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action | Reason |
|------|------|-------|-----|-----------|-------------|---------|--------|--------|
| 0 | 0 | 3 | 1 | 3 | 3 | 3 == 3 | right=2 | Ambiguous, shrink |
| 1 | 0 | 2 | 1 | 3 | 1 | 3 > 1 | left=2 | Min in right half |
| 2 | 2 | 2 | - | - | - | - | Stop | left == right |

**Final Answer:** `nums[2] = 1` ✓

**Explanation:**
- Step 0: `nums[1]=3 == nums[3]=3`, can't determine direction, shrink right
- Step 1: `nums[1]=3 > nums[2]=1`, minimum in right half
- Step 2: Found minimum at index 2

---

### Example 3: `nums = [10,1,10,10,10]` (Many Duplicates)

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 0 | 0 | 4 | 2 | 10 | 10 | 10 == 10 | right=3 |
| 1 | 0 | 3 | 1 | 1 | 10 | 1 < 10 | right=1 |
| 2 | 0 | 1 | 0 | 10 | 1 | 10 > 1 | left=1 |
| 3 | 1 | 1 | - | - | - | - | Stop |

**Final Answer:** `nums[1] = 1` ✓

---

### Example 4: `nums = [1,1,1,1,1]` (All Same - Worst Case)

| Step | Left | Right | Mid | nums[mid] | nums[right] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 0 | 0 | 4 | 2 | 1 | 1 | 1 == 1 | right=3 |
| 1 | 0 | 3 | 1 | 1 | 1 | 1 == 1 | right=2 |
| 2 | 0 | 2 | 1 | 1 | 1 | 1 == 1 | right=1 |
| 3 | 0 | 1 | 0 | 1 | 1 | 1 == 1 | right=0 |
| 4 | 0 | 0 | - | - | - | - | Stop |

**Final Answer:** `nums[0] = 1` ✓

**Time:** O(n) - Had to shrink right pointer 4 times

---

## Why Duplicates Make This Hard

### The Ambiguous Case

**Without duplicates:**
```
[4, 5, 6, 7, 0, 1, 2]
         ↑        ↑
       mid=7    right=2

7 > 2 → Clear! Minimum in right half
```

**With duplicates:**
```
Case 1: [3, 1, 3, 3, 3]
             ↑        ↑
           mid=3    right=3
3 == 3 → Which way? Could be either side!

Case 2: [3, 3, 3, 1, 3]
             ↑        ↑
           mid=3    right=3
3 == 3 → Same! Can't tell direction!
```

**Why We Can't Just Pick a Direction:**

If we arbitrarily choose left or right when `nums[mid] == nums[right]`:
```
Example: [3, 1, 3]
              ↑  ↑
            mid right

If we go left: left = mid + 1 → Skip to index 2, miss minimum 1!
If we go right: right = mid - 1 → Skip to index 0, miss minimum 1!

Only safe option: Shrink by 1 (right -= 1)
```

---

## Why Shrink Right (Not Left)?

**Question:** Why `right -= 1` instead of `left += 1`?

**Answer:** Both work, but shrinking right is conventional:

**Shrinking Right:**
```python
if nums[mid] == nums[right]:
    right -= 1  # Standard approach
```

**Shrinking Left (Alternative):**
```python
if nums[mid] == nums[left]:
    left += 1  # Also works
```

**Why right is preferred:**
- Consistent with comparing `nums[mid]` with `nums[right]` in other cases
- Slightly more intuitive: "We have a duplicate of mid at right, so right is expendable"

---

## Edge Cases

### 1. All Same Elements
```python
nums = [1, 1, 1, 1, 1]
# Output: 1
# Worst case: O(n) time
```

### 2. No Duplicates
```python
nums = [3, 4, 5, 1, 2]
# Output: 1
# Best case: O(log n) like Part I
```

### 3. Single Element
```python
nums = [5]
# Output: 5
```

### 4. Two Elements (Same)
```python
nums = [2, 2]
# Output: 2

nums = [2, 1]
# Output: 1
```

### 5. Duplicates at Ends Only
```python
nums = [3, 3, 4, 5, 1, 3]
# Output: 1
```

### 6. Minimum is Duplicated
```python
nums = [1, 1, 1, 0, 0, 1, 1]
# Output: 0
# Multiple copies of minimum
```

### 7. No Rotation
```python
nums = [1, 2, 3, 4, 5]
# Output: 1
# Already sorted
```

---

## Common Mistakes

### ❌ Mistake 1: Not Handling Equality Case
```python
# Wrong! Missing the nums[mid] == nums[right] case
if nums[mid] > nums[right]:
    left = mid + 1
else:
    right = mid  # This includes equality, but doesn't handle it correctly
```

**Fix:** Explicitly handle the equality case
```python
if nums[mid] > nums[right]:
    left = mid + 1
elif nums[mid] < nums[right]:
    right = mid
else:
    right -= 1  # Shrink when equal
```

### ❌ Mistake 2: Shrinking by More Than 1
```python
# Wrong! Might skip the minimum
if nums[mid] == nums[right]:
    right = mid - 1  # Too aggressive!
```

**Fix:** Shrink by exactly 1
```python
if nums[mid] == nums[right]:
    right -= 1
```

### ❌ Mistake 3: Trying to Maintain O(log n) Always
```python
# Wrong approach! Can't always avoid O(n) with duplicates
# No clever trick eliminates worst case
```

**Reality:** Accept that duplicates force O(n) worst case

### ❌ Mistake 4: Using left <= right
```python
# Wrong! Use left < right
while left <= right:
    # ...
```

**Fix:**
```python
while left < right:
    # ...
```

### ❌ Mistake 5: Comparing with Left Instead of Right
```python
# Wrong! Creates different ambiguities
if nums[mid] > nums[left]:
    # ...
```

**Fix:** Compare with right consistently
```python
if nums[mid] > nums[right]:
    # ...
```

---

## Interview Insights

**Question Recognition:**
- "Rotated sorted array" + "duplicates" + "find minimum" → Modified binary search with shrinking
- Harder version of LeetCode 153

**Key Points to Mention:**

1. **Why duplicates are problematic:**
   - "Duplicates create an ambiguous case where nums[mid] == nums[right], and we can't determine which half contains the minimum."

2. **The shrinking strategy:**
   - "When we encounter duplicates, we safely shrink the search space by 1 since we know nums[right] has at least one duplicate at mid."

3. **Time complexity:**
   - "Worst case is O(n) when all elements are the same, because we must check each element. Best/average case is O(log n) when duplicates are minimal."

4. **Difference from Part I:**
   - "Part I (no duplicates) is always O(log n). Part II has O(n) worst case due to the ambiguous equality case."

**Follow-up Questions:**

1. **Q:** "Can you achieve better than O(n) worst case?"
   - **A:** No. When all elements are identical, we must examine each element to confirm they're all the same.

2. **Q:** "What if you compared with left instead?"
   - **A:** Similar approach works, but convention is to compare with right for consistency.

3. **Q:** "Could you shrink both pointers when equal?"
   - **A:** Yes, `left++` and `right--`, but one is sufficient and clearer.

4. **Q:** "How does this differ from binary search with duplicates in unsorted array?"
   - **A:** Without rotation/sorted property, you'd need O(n) linear search. The sorted property still helps us achieve O(log n) average case.

**Complexity Analysis:**
- Best case: O(log n) - Few duplicates, behaves like Part I
- Worst case: O(n) - All elements same, must check each
- Average case: O(log n) - Duplicates don't dominate

---

## Variations

### Variation 1: Count Minimum Occurrences
```python
def countMinOccurrences(nums):
    """Count how many times minimum appears."""
    # First find minimum
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        elif nums[mid] < nums[right]:
            right = mid
        else:
            right -= 1
    
    minimum = nums[left]
    
    # Count occurrences
    return nums.count(minimum)
```

### Variation 2: Return All Minimum Indices
```python
def findAllMinIndices(nums):
    """Find all indices where minimum appears."""
    # Find minimum value
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        elif nums[mid] < nums[right]:
            right = mid
        else:
            right -= 1
    
    minimum = nums[left]
    
    # Find all indices
    return [i for i, val in enumerate(nums) if val == minimum]
```

### Variation 3: Handle Edge Case of Empty Array
```python
def findMin_safe(nums):
    """Handle empty array case."""
    if not nums:
        return None  # or raise exception
    
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        elif nums[mid] < nums[right]:
            right = mid
        else:
            right -= 1
    
    return nums[left]
```

### Variation 4: Find Maximum Instead
```python
def findMax(nums):
    """Find maximum in rotated array with duplicates."""
    # Find minimum index
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = left + (right - left) // 2
        
        if nums[mid] > nums[right]:
            left = mid + 1
        elif nums[mid] < nums[right]:
            right = mid
        else:
            right -= 1
    
    min_idx = left
    
    # Maximum is just before minimum (circularly)
    max_idx = (min_idx - 1) % len(nums)
    return nums[max_idx]
```

---

## Related Problems

1. **Find Minimum in Rotated Sorted Array (LeetCode 153)** - Without duplicates, O(log n)
2. **Search in Rotated Sorted Array II (LeetCode 81)** - Search with duplicates
3. **Search in Rotated Sorted Array (LeetCode 33)** - Search without duplicates
4. **Find Peak Element (LeetCode 162)** - Similar binary search with ambiguity
5. **Find First and Last Position (LeetCode 34)** - Modified binary search
6. **Single Element in Sorted Array (LeetCode 540)** - Binary search variant

---

## Comparison: Part I vs Part II

| Aspect | Part I (No Duplicates) | Part II (With Duplicates) |
|--------|------------------------|---------------------------|
| **LeetCode** | 153 | 154 |
| **Difficulty** | Medium | Hard |
| **Time (Best)** | O(log n) | O(log n) |
| **Time (Worst)** | O(log n) | **O(n)** |
| **Cases** | 2 (>, ≤) | **3** (>, <, ==) |
| **Ambiguity** | None | When mid == right |
| **Strategy** | Binary search | Binary search + shrinking |

---

## Summary

**Key Takeaways:**

1. **Three Cases Instead of Two:**
   - `nums[mid] > nums[right]` → Minimum in right half
   - `nums[mid] < nums[right]` → Minimum in left half or mid
   - **`nums[mid] == nums[right]`** → Ambiguous, shrink by 1

2. **Why Duplicates Matter:**
   - Create ambiguity when mid == right
   - Can't determine which half has minimum
   - Force worst case O(n) complexity

3. **The Shrinking Strategy:**
   - When ambiguous, shrink right by 1: `right -= 1`
   - Safe because mid has a duplicate at right
   - Gradually eliminates duplicates

4. **Complexity Trade-off:**
   - Best/Average: O(log n) - Few duplicates
   - Worst: O(n) - All elements same
   - No algorithm can do better than O(n) worst case

5. **Interview Strategy:**
   - Explain why duplicates create ambiguity
   - Show the three-case handling
   - Acknowledge O(n) worst case is unavoidable

**Template:**
```python
left, right = 0, len(nums) - 1

while left < right:
    mid = left + (right - left) // 2
    
    if nums[mid] > nums[right]:
        left = mid + 1
    elif nums[mid] < nums[right]:
        right = mid
    else:  # nums[mid] == nums[right]
        right -= 1

return nums[left]
```

**Mental Model:** 
- Think of duplicates as "fog" that obscures the rotation point
- When we can't see clearly (mid == right), we carefully step forward (shrink by 1)
- In worst case (all fog), we must step through all elements
- The algorithm gracefully degrades from O(log n) to O(n) based on duplicate density
- This is optimal—no algorithm can guarantee better than O(n) worst case with duplicates!
