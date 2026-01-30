# 3Sum

**LeetCode 15** | **Difficulty:** Medium | **Category:** Two Pointers, Sorting, Array

---

## Problem Statement

Given an integer array `nums`, return all the **triplets** `[nums[i], nums[j], nums[k]]` such that:
- `i != j`, `i != k`, and `j != k`
- `nums[i] + nums[j] + nums[k] == 0`

Notice that the solution set must **not contain duplicate triplets**.

---

## Examples

### Example 1:
```
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]

Explanation: 
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0.
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0.
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0.
The distinct triplets are [-1,0,1] and [-1,-1,2].
Notice that the order of the output and the order of the triplets does not matter.
```

### Example 2:
```
Input: nums = [0,1,1]
Output: []

Explanation: 
The only possible triplet does not sum up to 0.
```

### Example 3:
```
Input: nums = [0,0,0]
Output: [[0,0,0]]

Explanation: 
The only possible triplet sums up to 0.
```

---

## Strategy

### Fix One + Two Pointers (Optimal)

**Key Insight:** Convert 3Sum into multiple 2Sum problems by **fixing one element** and using **two pointers** for the remaining two elements.

**Algorithm Steps:**

1. **Sort the array** - Enables two pointers technique and easy duplicate handling
2. **Fix first element** - Iterate through array with index `i`
3. **Apply 2Sum** - For remaining elements after `i`, use two pointers to find pairs that sum to `-nums[i]`
4. **Skip duplicates** - Avoid duplicate triplets by skipping repeated values

**Why Sort First?**
- Enables two pointers approach
- Makes duplicate detection trivial (consecutive duplicates)
- Changes time from O(n³) brute force to O(n²)

**Visual Example:**
```
nums = [-1, 0, 1, 2, -1, -4]
After sorting: [-4, -1, -1, 0, 1, 2]

Fix i=0 (a=-4): Find two numbers that sum to 4 in [-1,-1,0,1,2]
  No valid pairs

Fix i=1 (a=-1): Find two numbers that sum to 1 in [-1,0,1,2]
  l=2, r=5: -1+2=1 ✓  → Found [-1,-1,2]

Fix i=2 (a=-1): Skip! Duplicate of previous

Fix i=3 (a=0): Find two numbers that sum to 0 in [1,2]
  l=4, r=5: 1+2=3 ✗  → No valid pairs

Fix i=4 (a=1): Find two numbers that sum to -1 in [2]
  Not enough elements
```

**Complexity:**
- **Time:** $O(n^2)$ - O(n log n) for sort + O(n) iterations × O(n) two pointers
- **Space:** $O(1)$ or $O(n)$ depending on sort implementation (ignoring output array)

---

## Solution

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        """
        Find all unique triplets that sum to zero.
        
        Approach: Sort + Fix one element + Two pointers
        
        Args:
            nums: List of integers
            
        Returns:
            List of triplets [a, b, c] where a + b + c = 0
        """
        res = []
        nums.sort()  # Step 1: Sort the array
        
        # Step 2: Fix the first element
        for i, a in enumerate(nums):
            # Skip positive numbers (since sorted, rest will be positive too)
            if a > 0:
                break
            
            # Skip duplicate values for first element
            if i > 0 and a == nums[i - 1]:
                continue
            
            # Step 3: Use two pointers for remaining elements
            l, r = i + 1, len(nums) - 1
            
            while l < r:
                three_sum = a + nums[l] + nums[r]
                
                if three_sum > 0:
                    # Sum too large, decrease it
                    r -= 1
                elif three_sum < 0:
                    # Sum too small, increase it
                    l += 1
                else:
                    # Found a valid triplet!
                    res.append([a, nums[l], nums[r]])
                    l += 1
                    
                    # Skip duplicates for second element
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1
        
        return res
```

---

## Detailed Walkthrough

### Example: `nums = [-1, 0, 1, 2, -1, -4]`

**Step 0: Sort the array**
```
Original: [-1, 0, 1, 2, -1, -4]
Sorted:   [-4, -1, -1, 0, 1, 2]
           i0  i1  i2 i3 i4 i5
```

**Iteration Table:**

| i | a | Skip? | l | r | nums[l] | nums[r] | Sum | Action | Found |
|---|---|-------|---|---|---------|---------|-----|--------|-------|
| 0 | -4 | No | 1 | 5 | -1 | 2 | -3 | l++ | - |
| 0 | -4 | No | 2 | 5 | -1 | 2 | -3 | l++ | - |
| 0 | -4 | No | 3 | 5 | 0 | 2 | -2 | l++ | - |
| 0 | -4 | No | 4 | 5 | 1 | 2 | -1 | l++ | - |
| 0 | -4 | No | 5 | 5 | - | - | - | Stop | - |
| 1 | -1 | No | 2 | 5 | -1 | 2 | 0 | ✓ Found | [-1,-1,2] |
| 1 | -1 | No | 3 | 5 | 0 | 2 | 1 | r-- | - |
| 1 | -1 | No | 3 | 4 | 0 | 1 | 0 | ✓ Found | [-1,0,1] |
| 1 | -1 | No | 4 | 4 | - | - | - | Stop | - |
| 2 | -1 | Yes | - | - | - | - | - | Skip dup | - |
| 3 | 0 | No | 4 | 5 | 1 | 2 | 3 | r-- | - |
| 3 | 0 | No | 4 | 4 | - | - | - | Stop | - |
| 4 | 1 | Break | - | - | - | - | - | a > 0 | - |

**Final Result:** `[[-1,-1,2], [-1,0,1]]`

**Detailed Step-by-Step:**

**i=0, a=-4:**
- Target: Find two numbers that sum to 4
- Search in: [-1, -1, 0, 1, 2]
- Two pointers can't find sum of 4 (max is -1+2=1)

**i=1, a=-1:**
- Target: Find two numbers that sum to 1
- Search in: [-1, 0, 1, 2]
- `l=2, r=5`: -1 + 2 = 1 ✓ → **Found [-1, -1, 2]**
- Move `l=3`, skip duplicate
- `l=3, r=4`: 0 + 1 = 1 ✓ → **Found [-1, 0, 1]**

**i=2, a=-1:**
- **Skip!** Same as previous (duplicate)

**i=3, a=0:**
- Target: Find two numbers that sum to 0
- Search in: [1, 2]
- `l=4, r=5`: 1 + 2 = 3 > 0 → No solution

**i=4, a=1:**
- `a > 0` → **Break!** All remaining numbers are positive, impossible to sum to 0

---

## Why Skip Duplicates?

### Without Duplicate Handling:
```
nums = [-1, -1, 2]

Fix i=0, a=-1: Find [-1, 2] → Result: [-1, -1, 2]
Fix i=1, a=-1: Find [-1, 2] → Result: [-1, -1, 2]  ← DUPLICATE!
```

### With Duplicate Handling:
```
Fix i=0, a=-1: Find [-1, 2] → Result: [-1, -1, 2]
Fix i=1, a=-1: Skip (same as previous)  ✓
```

**Three Levels of Duplicate Prevention:**

1. **First element (i):** Skip if `nums[i] == nums[i-1]`
2. **Second element (l):** Skip if `nums[l] == nums[l-1]` after finding a triplet
3. **Third element (r):** Automatically handled by moving `l` (since sorted)

---

## Alternative Approach: Hash Set

Using a hash set to track seen pairs:

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        """
        Hash set approach: For each pair, check if complement exists.
        Less efficient due to duplicate handling complexity.
        """
        nums.sort()
        res = []
        
        for i in range(len(nums)):
            # Skip duplicates for first element
            if i > 0 and nums[i] == nums[i - 1]:
                continue
            
            seen = set()
            for j in range(i + 1, len(nums)):
                complement = -nums[i] - nums[j]
                
                if complement in seen:
                    res.append([nums[i], complement, nums[j]])
                    # Skip duplicates for second element
                    while j + 1 < len(nums) and nums[j] == nums[j + 1]:
                        j += 1
                
                seen.add(nums[j])
        
        return res
```

**Complexity:**
- **Time:** $O(n^2)$ - Same as two pointers
- **Space:** $O(n)$ - Hash set storage

**Verdict:** Two pointers is **better** - cleaner duplicate handling, no extra space!

---

## Edge Cases

### 1. Empty or Small Array
```python
nums = []
# Output: []

nums = [1, 2]
# Output: []
# Need at least 3 elements
```

### 2. All Same Elements (Zero)
```python
nums = [0, 0, 0, 0]
# Output: [[0, 0, 0]]
# Only one unique triplet
```

### 3. All Same Elements (Non-Zero)
```python
nums = [1, 1, 1]
# Output: []
# 1 + 1 + 1 = 3 ≠ 0
```

### 4. No Valid Triplets
```python
nums = [1, 2, 3, 4, 5]
# Output: []
# All positive, can't sum to 0
```

### 5. All Negative
```python
nums = [-5, -4, -3, -2, -1]
# Output: []
# All negative, can't sum to 0
```

### 6. Multiple Duplicates
```python
nums = [-2, 0, 0, 2, 2]
# Output: [[-2, 0, 2]]
# Multiple ways to form same triplet, return unique only
```

### 7. Large Array with One Solution
```python
nums = [-1, 0, 1] + [2] * 1000
# Output: [[-1, 0, 1]]
# Efficiently handles duplicates
```

---

## Common Mistakes

### ❌ Mistake 1: Not Sorting First
```python
# Wrong! Can't use two pointers on unsorted array
def threeSum(nums):
    for i in range(len(nums)):
        l, r = i + 1, len(nums) - 1
        # Two pointers won't work correctly!
```

**Fix:** Always sort first: `nums.sort()`

### ❌ Mistake 2: Incomplete Duplicate Handling
```python
# Wrong! Only skips duplicates for first element
for i in range(len(nums)):
    if i > 0 and nums[i] == nums[i - 1]:
        continue
    # Missing: duplicate handling for l and r
```

**Fix:** Skip duplicates for all three positions:
```python
# First element
if i > 0 and nums[i] == nums[i - 1]:
    continue

# Second element (after finding triplet)
while l < r and nums[l] == nums[l - 1]:
    l += 1
```

### ❌ Mistake 3: Wrong Duplicate Check for l
```python
# Wrong! Skips valid first occurrence
if nums[l] == nums[l - 1]:
    l += 1
    continue
```

**Fix:** Only skip duplicates AFTER finding a triplet:
```python
res.append([a, nums[l], nums[r]])
l += 1
while l < r and nums[l] == nums[l - 1]:
    l += 1
```

### ❌ Mistake 4: Not Breaking on Positive First Element
```python
# Inefficient! Continues even when impossible
for i in range(len(nums)):
    # Missing optimization
```

**Fix:** Break when first element becomes positive:
```python
if nums[i] > 0:
    break
```

### ❌ Mistake 5: Using Set for Result Deduplication
```python
# Inefficient! O(n³) to create tuple keys
result_set = set()
# ... add tuples to set
return [list(t) for t in result_set]
```

**Fix:** Handle duplicates during iteration, not after.

---

## Interview Insights

**Question Recognition:**
- "Find triplets" + "sum to target" + "no duplicates" → Sort + Two Pointers
- Variation of Two Sum problem

**Key Points to Mention:**

1. **Why sort first?**
   - "Sorting enables two pointers technique and makes duplicate handling trivial by checking consecutive elements."

2. **Time complexity:**
   - "O(n²) because we iterate through n elements and for each, we do a two-pointer scan which is O(n)."

3. **Space complexity:**
   - "O(1) or O(n) depending on sorting algorithm (ignoring output space). Python's Timsort uses O(n) space."

4. **Duplicate handling:**
   - "We skip duplicates at three levels: first element (i), second element (l) after finding triplet, and third element is handled automatically."

**Follow-up Questions:**

1. **Q:** "Can you find triplets that sum to any target (not just 0)?"
   - **A:** Yes! Change condition from `three_sum == 0` to `three_sum == target`.

2. **Q:** "What about 4Sum or kSum?"
   - **A:** Extend to 4Sum with two nested loops + two pointers (O(n³)). For kSum, use recursion.

3. **Q:** "How to return count instead of actual triplets?"
   - **A:** Same algorithm, increment counter instead of appending to result.

4. **Q:** "What if array is too large to sort in memory?"
   - **A:** Use external sorting or streaming algorithm with multiple passes.

**Optimization Discussion:**
- Early termination when `nums[i] > 0`
- Early termination in two pointers when `nums[i] + nums[i+1] + nums[i+2] > 0`
- Skip to next distinct value instead of increment by 1

---

## Variations

### Variation 1: 3Sum with Target
```python
def threeSumTarget(nums, target):
    """Find triplets that sum to given target."""
    nums.sort()
    res = []
    
    for i in range(len(nums)):
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        l, r = i + 1, len(nums) - 1
        # Looking for nums[l] + nums[r] = target - nums[i]
        remaining = target - nums[i]
        
        while l < r:
            current_sum = nums[l] + nums[r]
            
            if current_sum == remaining:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                while l < r and nums[l] == nums[l - 1]:
                    l += 1
            elif current_sum < remaining:
                l += 1
            else:
                r -= 1
    
    return res
```

### Variation 2: 3Sum Closest
```python
def threeSumClosest(nums, target):
    """LeetCode 16: Find triplet with sum closest to target."""
    nums.sort()
    closest_sum = float('inf')
    
    for i in range(len(nums)):
        l, r = i + 1, len(nums) - 1
        
        while l < r:
            current_sum = nums[i] + nums[l] + nums[r]
            
            # Update closest if current sum is closer
            if abs(current_sum - target) < abs(closest_sum - target):
                closest_sum = current_sum
            
            if current_sum < target:
                l += 1
            elif current_sum > target:
                r -= 1
            else:
                return current_sum  # Exact match
    
    return closest_sum
```

### Variation 3: 3Sum Smaller
```python
def threeSumSmaller(nums, target):
    """LeetCode 259: Count triplets with sum < target."""
    nums.sort()
    count = 0
    
    for i in range(len(nums)):
        l, r = i + 1, len(nums) - 1
        
        while l < r:
            current_sum = nums[i] + nums[l] + nums[r]
            
            if current_sum < target:
                # All pairs (l, l+1...r) are valid
                count += r - l
                l += 1
            else:
                r -= 1
    
    return count
```

### Variation 4: 4Sum
```python
def fourSum(nums, target):
    """LeetCode 18: Find all unique quadruplets that sum to target."""
    nums.sort()
    res = []
    
    for i in range(len(nums)):
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        # Now it's 3Sum with target - nums[i]
        for j in range(i + 1, len(nums)):
            if j > i + 1 and nums[j] == nums[j - 1]:
                continue
            
            l, r = j + 1, len(nums) - 1
            
            while l < r:
                four_sum = nums[i] + nums[j] + nums[l] + nums[r]
                
                if four_sum == target:
                    res.append([nums[i], nums[j], nums[l], nums[r]])
                    l += 1
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1
                elif four_sum < target:
                    l += 1
                else:
                    r -= 1
    
    return res
```

---

## Related Problems

1. **Two Sum (LeetCode 1)** - Base case, use hash map
2. **Two Sum II (LeetCode 167)** - Sorted array, two pointers
3. **3Sum Closest (LeetCode 16)** - Find closest sum instead of exact
4. **3Sum Smaller (LeetCode 259)** - Count triplets with sum < target
5. **4Sum (LeetCode 18)** - Extend to quadruplets
6. **4Sum II (LeetCode 454)** - Four arrays, hash map approach

---

## Summary

**Key Takeaways:**

1. **Core Technique:** Sort + Fix one + Two pointers
   - Reduces from O(n³) brute force to O(n²)
   - Enables efficient duplicate handling

2. **Algorithm Pattern:**
   ```
   Sort array
   For each element i:
       Skip if duplicate
       Use two pointers (l, r) on remaining elements
       Find pairs that sum to -nums[i]
       Skip duplicates for l
   ```

3. **Duplicate Handling:** Three levels
   - **i:** Skip if `nums[i] == nums[i-1]`
   - **l:** Skip if `nums[l] == nums[l-1]` after finding triplet
   - **r:** Automatically handled by l movement

4. **Optimizations:**
   - Break when `nums[i] > 0` (all positive, impossible)
   - Early termination in two pointers
   - Skip to next distinct value

5. **Complexity:**
   - Time: $O(n^2)$ - O(n log n) sort + O(n) × O(n) two pointers
   - Space: $O(1)$ or $O(n)$ for sort (ignoring output)

**Template:**
```python
nums.sort()
res = []

for i in range(len(nums)):
    if nums[i] > 0:
        break
    if i > 0 and nums[i] == nums[i - 1]:
        continue
    
    l, r = i + 1, len(nums) - 1
    
    while l < r:
        total = nums[i] + nums[l] + nums[r]
        
        if total == 0:
            res.append([nums[i], nums[l], nums[r]])
            l += 1
            while l < r and nums[l] == nums[l - 1]:
                l += 1
        elif total < 0:
            l += 1
        else:
            r -= 1

return res
```
