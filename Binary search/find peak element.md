# Find Peak Element

**LeetCode 162** | **Difficulty:** Medium | **Category:** Binary Search, Array

---

## Problem Statement

A **peak element** is an element that is **strictly greater** than its neighbors.

Given a **0-indexed** integer array `nums`, find a peak element, and return its index. If the array contains multiple peaks, return the index to **any of the peaks**.

You may imagine that `nums[-1] = nums[n] = -∞`. In other words, an element is always considered to be strictly greater than a neighbor that is outside the array.

You must write an algorithm that runs in **O(log n)** time.

---

## Examples

### Example 1:
```
Input: nums = [1,2,3,1]
Output: 2
Explanation: 3 is a peak element and your function should return the index number 2.
```

### Example 2:
```
Input: nums = [1,2,1,3,5,6,4]
Output: 5
Explanation: Your function can return either index number 1 where the peak element is 2, or index number 5 where the peak element is 6.
```

### Example 3:
```
Input: nums = [1,2,3,4,5]
Output: 4
Explanation: Array is strictly increasing, last element is peak.
```

### Example 4:
```
Input: nums = [5,4,3,2,1]
Output: 0
Explanation: Array is strictly decreasing, first element is peak.
```

---

## Strategy

### Binary Search on Unsorted Array

**Key Insight:** Even though the array is **not sorted**, we can still use binary search by following the **upward slope**.

**Visual Understanding:**

```
Array: [1, 2, 1, 3, 5, 6, 4]
       ↑  ↑  ↑  ↑  ↑  ↑  ↑
       
At mid=3 (value=3):
    nums[3]=3 < nums[4]=5  →  Upward slope!
    ↗️ Peak must exist to the right
    
At mid=5 (value=6):
    nums[5]=6 > nums[6]=4  →  Downward slope!
    Peak is here or to the left
```

**Core Logic:**

1. **Compare mid with mid+1** (not mid-1)
2. **If `nums[mid] < nums[mid+1]`** (Upward slope):
   - Peak **must** be in right half
   - Why? We're going uphill, and array ends are -∞, so we'll hit a peak
   - Action: `left = mid + 1`

3. **If `nums[mid] >= nums[mid+1]`** (Downward slope or plateau):
   - Peak is at mid or in left half
   - Why? Mid could be the peak, or we're coming down from one
   - Action: `right = mid`

**Why This Works (Mathematical Guarantee):**

Since `nums[-1] = nums[n] = -∞`:
- If we follow upward slope, we **must** eventually hit a peak (can't go up forever)
- If we're on downward slope, we're either at peak or moving toward one
- **At least one peak always exists**

**Visual Decision Tree:**
```
At mid position:
├─ nums[mid] < nums[mid+1]? (Going uphill)
│   └─ Yes: Peak guaranteed to the right
│       Action: left = mid + 1
│       
└─ nums[mid] >= nums[mid+1]? (Going downhill or flat)
    └─ Yes: Peak is at mid or to the left
        Action: right = mid
```

**Why O(log n)?**
- Each iteration eliminates half the search space
- Classic binary search reduction
- No need for sorted array because we follow slope direction

**Complexity:**
- **Time:** $O(\log n)$ - Binary search
- **Space:** $O(1)$ - Only using pointers

---

## Solution

```python
class Solution:
    def findPeakElement(self, nums: List[int]) -> int:
        """
        Find a peak element using binary search.
        
        Key insight: Follow the upward slope to guarantee finding a peak.
        Since array boundaries are treated as -∞, we're guaranteed to find
        a peak by always moving toward higher ground.
        
        Args:
            nums: Integer array where nums[-1] = nums[n] = -∞
            
        Returns:
            Index of any peak element
        """
        left, right = 0, len(nums) - 1
        
        while left < right:
            mid = (left + right) // 2
            
            # Compare mid with its right neighbor
            if nums[mid] < nums[mid + 1]:
                # Upward slope: Peak is to the right
                # We know a peak exists because array ends at -∞
                left = mid + 1
            else:
                # Downward slope: mid could be a peak or it's to the left
                # Keep mid in search space since it could be the peak
                right = mid
        
        # When left == right, we've found a peak
        return left
```

---

## Detailed Walkthrough

### Example 1: `nums = [1,2,1,3,5,6,4]`

**Step-by-Step Execution:**

| Step | Left | Right | Mid | nums[mid] | nums[mid+1] | Compare | Slope | Action | Reason |
|------|------|-------|-----|-----------|-------------|---------|-------|--------|--------|
| 0 | 0 | 6 | 3 | 3 | 5 | 3 < 5 | ↗️ Up | left=4 | Peak to right |
| 1 | 4 | 6 | 5 | 6 | 4 | 6 > 4 | ↘️ Down | right=5 | Peak at mid or left |
| 2 | 4 | 5 | 4 | 5 | 6 | 5 < 6 | ↗️ Up | left=5 | Peak to right |
| 3 | 5 | 5 | - | - | - | - | - | Stop | left == right |

**Final Answer:** `5` (value `6`) ✓

**Visualization:**
```
Step 0: [1, 2, 1, 3, 5, 6, 4]
                   ↑        
                  mid=3 < 4
                  Go right ➡️

Step 1: [1, 2, 1, 3, 5, 6, 4]
                      ↑     
                    mid=6 > 4
                    Go left ⬅️

Step 2: [1, 2, 1, 3, 5, 6, 4]
                   ↑  ↑     
                   4  5
                  mid=5 < 6
                  Go right ➡️

Found peak at index 5!
```

---

### Example 2: `nums = [1,2,3,4,5]` (Strictly Increasing)

| Step | Left | Right | Mid | nums[mid] | nums[mid+1] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 0 | 0 | 4 | 2 | 3 | 4 | 3 < 4 | left=3 |
| 1 | 3 | 4 | 3 | 4 | 5 | 4 < 5 | left=4 |
| 2 | 4 | 4 | - | - | - | - | Stop |

**Final Answer:** `4` (last element is peak) ✓

**Explanation:** Always going uphill, ends at rightmost element (which is > -∞)

---

### Example 3: `nums = [5,4,3,2,1]` (Strictly Decreasing)

| Step | Left | Right | Mid | nums[mid] | nums[mid+1] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 0 | 0 | 4 | 2 | 3 | 2 | 3 > 2 | right=2 |
| 1 | 0 | 2 | 1 | 4 | 3 | 4 > 3 | right=1 |
| 2 | 0 | 1 | 0 | 5 | 4 | 5 > 4 | right=0 |
| 3 | 0 | 0 | - | - | - | - | Stop |

**Final Answer:** `0` (first element is peak) ✓

**Explanation:** Always going downhill, ends at leftmost element (which is > -∞)

---

### Example 4: `nums = [1,3,2,1]` (Multiple Peaks)

| Step | Left | Right | Mid | nums[mid] | nums[mid+1] | Compare | Action |
|------|------|-------|-----|-----------|-------------|---------|--------|
| 0 | 0 | 3 | 1 | 3 | 2 | 3 > 2 | right=1 |
| 1 | 0 | 1 | 0 | 1 | 3 | 1 < 3 | left=1 |
| 2 | 1 | 1 | - | - | - | - | Stop |

**Final Answer:** `1` (value `3`) ✓

**Note:** Index 1 is a peak. There might be other peaks, but we only need to find one.

---

## Why This Algorithm Works

### Mathematical Proof

**Claim:** Following the upward slope guarantees finding a peak.

**Proof:**

1. **Base Case:** If `nums[mid] < nums[mid+1]`, we're on an upward slope.

2. **What happens when we go right?**
   - We move to higher ground: `left = mid + 1`
   - Two possibilities:
     - **a)** We continue upward: Keep going right
     - **b)** We start downward: We've found a peak!

3. **Boundary Guarantee:**
   - Right boundary: `nums[n] = -∞`
   - If we keep going right, we **must** eventually go down
   - When we go down, we've passed a peak

4. **Similarly for left:**
   - If `nums[mid] >= nums[mid+1]`, we're on downward slope
   - Mid could be peak, or peak is to the left
   - Left boundary: `nums[-1] = -∞`, so first element is peak if all decreasing

**Conclusion:** By always moving toward higher ground, we're guaranteed to hit a peak. ✓

---

### Visual Proof

```
Case 1: Following uphill to the right
[1, 2, 3, 4, 5]  →  Peak at end (5 > -∞)
 ↗️ ↗️ ↗️ ↗️        

Case 2: Following uphill then downhill
[1, 2, 4, 3, 1]  →  Peak at 4
 ↗️ ↗️ ↘️ ↘️        Found it!

Case 3: Starting downhill
[5, 4, 3, 2, 1]  →  Peak at start (5 > -∞)
 ↘️ ↘️ ↘️ ↘️        

Impossible case: Can't keep going up forever
[1, 2, 3, 4, ...]  →  Must end at -∞, so must go down!
```

---

## Edge Cases

### 1. Single Element
```python
nums = [5]
# Output: 0
# Only element is peak (5 > -∞ on both sides)
```

### 2. Two Elements (Increasing)
```python
nums = [1, 2]
# Output: 1
# Last element is peak (2 > 1 and 2 > -∞)
```

### 3. Two Elements (Decreasing)
```python
nums = [2, 1]
# Output: 0
# First element is peak (2 > -∞ and 2 > 1)
```

### 4. All Same Elements (Not Possible)
```
Problem constraint: nums[i] != nums[i+1]
So this case doesn't exist
```

### 5. Multiple Peaks
```python
nums = [1, 3, 2, 4, 1]
# Output: 1 or 3 (both are peaks)
# Algorithm finds one, doesn't matter which
```

### 6. Plateau at Peak (Not Possible)
```
Problem constraint: nums[i] != nums[i+1]
No plateaus exist
```

### 7. Very Long Array
```python
nums = list(range(1000000))
# Output: 999999
# Still O(log n) = ~20 iterations
```

---

## Common Mistakes

### ❌ Mistake 1: Comparing with mid-1 Instead of mid+1
```python
# Wrong! Creates edge case issues
if nums[mid] > nums[mid - 1]:
    left = mid + 1
else:
    right = mid
```

**Problems:**
- When `mid=0`, accessing `nums[-1]` wraps to last element
- Logic becomes confusing

**Fix:** Always compare with `mid+1`
```python
if nums[mid] < nums[mid + 1]:
    left = mid + 1
else:
    right = mid
```

### ❌ Mistake 2: Using left <= right
```python
# Wrong! Can cause infinite loop
while left <= right:
    # ...
```

**Fix:** Use `left < right`
```python
while left < right:
    # ...
```

### ❌ Mistake 3: Wrong Comparison Direction
```python
# Wrong! Inverted logic
if nums[mid] > nums[mid + 1]:
    left = mid + 1  # Going toward downhill!
```

**Fix:** Go toward uphill
```python
if nums[mid] < nums[mid + 1]:
    left = mid + 1  # Following uphill
```

### ❌ Mistake 4: Excluding Potential Peak
```python
# Wrong! mid might be the peak
if nums[mid] >= nums[mid + 1]:
    right = mid - 1  # Excludes mid!
```

**Fix:** Keep mid in search space
```python
if nums[mid] >= nums[mid + 1]:
    right = mid  # mid could be peak
```

### ❌ Mistake 5: Trying to Find All Peaks
```python
# Wrong approach for this problem!
# We only need to find ONE peak
peaks = []
for i in range(len(nums)):
    if is_peak(i):
        peaks.append(i)
# This is O(n), not O(log n)
```

**Fix:** Use binary search to find one peak in O(log n)

---

## Interview Insights

**Question Recognition:**
- "Find peak element" + "O(log n)" → Binary search on unsorted array
- "Any peak" (not all peaks) → Can use greedy binary search

**Key Points to Mention:**

1. **Why binary search works on unsorted array:**
   - "We don't need sorted array. We follow the upward slope direction, which guarantees finding a peak because boundaries are -∞."

2. **The upward slope guarantee:**
   - "If we're going uphill, we must eventually hit a peak. We can't keep going up forever because the array ends at -∞."

3. **Why we compare with mid+1:**
   - "Comparing mid with mid+1 avoids edge case issues at index 0 and gives clearer logic about slope direction."

4. **Multiple peaks:**
   - "The problem allows returning any peak, so binary search naturally finds one without needing to check all elements."

**Follow-up Questions:**

1. **Q:** "What if you need to find ALL peaks?"
   - **A:** Would require O(n) linear scan. Binary search only finds one peak in O(log n).

2. **Q:** "Can you modify this for a 2D matrix to find a peak?"
   - **A:** Yes! LeetCode 1901 - use binary search on rows/columns. More complex but similar principle.

3. **Q:** "What if array boundaries aren't -∞?"
   - **A:** Need to add explicit checks for first and last elements being peaks.

4. **Q:** "Could you use this algorithm if nums[i] == nums[i+1] is allowed?"
   - **A:** Would need modification. When equal, can't determine slope direction clearly.

**Complexity Analysis:**
- Time: O(log n) - Halving search space each iteration
- Space: O(1) - Only using pointers

---

## Variations

### Variation 1: Find First Peak (Leftmost)
```python
def findFirstPeak(nums):
    """Find the leftmost peak."""
    # Use linear scan - can't use binary search for "first" peak
    n = len(nums)
    
    for i in range(n):
        left_ok = (i == 0 or nums[i] > nums[i-1])
        right_ok = (i == n-1 or nums[i] > nums[i+1])
        
        if left_ok and right_ok:
            return i
    
    return -1  # No peak found
```

### Variation 2: Count All Peaks
```python
def countPeaks(nums):
    """Count total number of peaks."""
    if len(nums) < 1:
        return 0
    
    count = 0
    n = len(nums)
    
    for i in range(n):
        left_ok = (i == 0 or nums[i] > nums[i-1])
        right_ok = (i == n-1 or nums[i] > nums[i+1])
        
        if left_ok and right_ok:
            count += 1
    
    return count
```

### Variation 3: Find Peak Value (Not Index)
```python
def findPeakValue(nums):
    """Return the peak value instead of index."""
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        if nums[mid] < nums[mid + 1]:
            left = mid + 1
        else:
            right = mid
    
    return nums[left]  # Return value, not index
```

### Variation 4: Find Peak in Circular Array
```python
def findPeakCircular(nums):
    """Find peak in circular array."""
    n = len(nums)
    left, right = 0, n - 1
    
    while left < right:
        mid = (left + right) // 2
        
        # Compare with next element (circular)
        next_idx = (mid + 1) % n
        
        if nums[mid] < nums[next_idx]:
            left = mid + 1
        else:
            right = mid
    
    return left
```

---

## Related Problems

1. **Find Peak Element II (LeetCode 1901)** - 2D matrix version, O(m log n)
2. **Peak Index in Mountain Array (LeetCode 852)** - Guaranteed single peak
3. **Find Mountain in Array (LeetCode 941)** - Strictly increasing then decreasing
4. **Longest Mountain in Array (LeetCode 845)** - Count mountain length
5. **Search in Rotated Sorted Array (LeetCode 33)** - Similar binary search on unsorted
6. **Find Minimum in Rotated Sorted Array (LeetCode 153)** - Related rotation concept

---

## Summary

**Key Takeaways:**

1. **Binary Search Without Sorting:**
   - Don't need sorted array
   - Follow slope direction instead
   - Works because boundaries are -∞

2. **The Upward Slope Guarantee:**
   - Always move toward higher ground
   - Must eventually hit peak (can't go up forever)
   - Mathematical guarantee, not luck

3. **Compare with mid+1:**
   - Clearer logic than comparing with mid-1
   - Avoids edge case at index 0
   - Directly shows slope direction

4. **Any Peak is Valid:**
   - Don't need to find all peaks
   - Don't need to find specific peak
   - Makes O(log n) solution possible

5. **Why It's Efficient:**
   - Each iteration eliminates half search space
   - Classic binary search reduction
   - 20 iterations for 1 million elements!

**Template:**
```python
left, right = 0, len(nums) - 1

while left < right:
    mid = (left + right) // 2
    
    if nums[mid] < nums[mid + 1]:
        # Upward slope: go right
        left = mid + 1
    else:
        # Downward slope: mid or left
        right = mid

return left
```

**Mental Model:**
- Think of yourself as a hiker looking for any hilltop
- Always walk uphill when you can
- When you can't go higher, you're at a peak!
- The -∞ boundaries ensure you can't walk off forever
- No need to map the entire terrain (O(n)), just follow uphill (O(log n))


Key Interview Insights
Local vs. Global: This algorithm finds a local peak, not necessarily the highest peak in the entire array. The problem only asks for any peak.

Boundary Conditions: This is a "boundary-safe" version. By comparing mid with mid + 1, and knowing the loop ends when left == right, we avoid index out-of-bounds errors on the edges.