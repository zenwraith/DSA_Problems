# Minimum Size Subarray Sum

**LeetCode 209** | **Difficulty:** Medium | **Category:** Sliding Window, Two Pointers

---

## Understanding Sliding Window Types

### The Strategy: Dynamic Sliding Window

Unlike a **fixed-size window** (where the window size is constant), this is a **dynamic window**. We expand it until we meet the condition, then shrink it to find the minimum possible size.

**The Logic:**

1. **Expand:** Move the right pointer to add elements to your `current_sum`.
2. **Contract:** Once `current_sum >= target`, you have a "valid" window. Now, try to make it smaller by moving the left pointer forward while the sum is still $\ge$ target.
3. **Update:** Every time the window is valid, record its length and keep the minimum.

### Window Behavior Comparison

| Problem Type | Window Behavior | Example Problem |
|-------------|-----------------|-----------------|
| **Fixed Size** | Right moves, left follows at a distance | Find max sum of subarray size K |
| **Dynamic (Expand)** | Right moves until valid | Longest Substring with K distinct chars |
| **Dynamic (Contract)** | Right moves, then left shrinks as much as possible | Minimum Size Subarray Sum |

---

## Problem Statement

Given an array of positive integers `nums` and a positive integer `target`, return the **minimal length** of a subarray whose sum is **greater than or equal to** `target`. If there is no such subarray, return `0` instead.

---

## Examples

### Example 1:
```
Input: target = 7, nums = [2,3,1,2,4,3]
Output: 2

Explanation: 
The subarray [4,3] has the minimal length under the problem constraint.
```

### Example 2:
```
Input: target = 4, nums = [1,4,4]
Output: 1

Explanation:
The subarray [4] has the minimal length.
```

### Example 3:
```
Input: target = 11, nums = [1,1,1,1,1,1,1,1]
Output: 0

Explanation:
No subarray has sum >= 11, so return 0.
```

---

## Strategy

### Sliding Window Approach

This problem is a classic **variable-size sliding window** problem:

**Key Insights:**
1. All numbers are positive → adding more elements always increases sum
2. We want the **smallest** subarray with sum ≥ target
3. Once we find a valid window, try to shrink it from the left

**Algorithm:**
```
1. Expand window by moving right pointer
2. Add current element to sum
3. While sum >= target:
   - Update minimum length
   - Shrink window from left (remove nums[left])
   - Move left pointer forward
4. Repeat until right pointer reaches end
```

**Why This Works:**
- **Expanding:** Ensures we explore all possible subarrays
- **Shrinking:** Finds the smallest valid subarray for each right position
- **Greedy:** Once sum >= target, we immediately try to minimize length

**Complexity:**
- **Time:** $O(n)$ - Each element is visited at most twice (once by right, once by left)
- **Space:** $O(1)$ - Only constant extra space

---

## Solution

```python
from typing import List

class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        left = 0
        current_sum = 0
        min_length = float('inf')
        
        # Expand window with right pointer
        for right in range(len(nums)):
            # Add current element to window
            current_sum += nums[right]
            
            # Shrink window while sum >= target
            # Try to find the smallest valid window
            while current_sum >= target:
                # Update minimum length
                min_length = min(min_length, right - left + 1)
                
                # Remove leftmost element and shrink window
                current_sum -= nums[left]
                left += 1
        
        # Return result (0 if no valid subarray found)
        return min_length if min_length != float('inf') else 0
```

---

## Detailed Walkthrough

### Example: `target = 7, nums = [2, 3, 1, 2, 4, 3]`

**Visual Representation:**

| Step | Left | Right | Window | Current Sum | Sum >= 7? | Min Length | Action |
|------|------|-------|--------|-------------|-----------|------------|--------|
| 0 | 0 | 0 | [2] | 2 | ❌ | ∞ | Expand |
| 1 | 0 | 1 | [2,3] | 5 | ❌ | ∞ | Expand |
| 2 | 0 | 2 | [2,3,1] | 6 | ❌ | ∞ | Expand |
| 3 | 0 | 3 | [2,3,1,2] | 8 | ✅ | 4 | Shrink |
| 4 | 1 | 3 | [3,1,2] | 6 | ❌ | 4 | Expand |
| 5 | 1 | 4 | [3,1,2,4] | 10 | ✅ | 4 | Shrink |
| 6 | 2 | 4 | [1,2,4] | 7 | ✅ | 3 | Shrink |
| 7 | 3 | 4 | [2,4] | 6 | ❌ | 3 | Expand |
| 8 | 3 | 5 | [2,4,3] | 9 | ✅ | 3 | Shrink |
| 9 | 4 | 5 | [4,3] | 7 | ✅ | **2** | Shrink |
| 10 | 5 | 5 | [3] | 3 | ❌ | 2 | Done |

**Final Answer:** `2` (subarray [4, 3])

**Detailed Steps:**

1. **Steps 0-2:** Build window [2,3,1], sum = 6 < 7
2. **Step 3:** Add 2 → sum = 8 ≥ 7 ✓
   - Record length = 4
   - Try to shrink: remove 2 → sum = 6 < 7
3. **Step 5:** Add 4 → sum = 10 ≥ 7 ✓
   - Try shrinking twice:
     - Remove 3 → sum = 7 ≥ 7, length = 3
     - Remove 1 → sum = 6 < 7, stop
4. **Step 8:** Add 3 → sum = 9 ≥ 7 ✓
   - Remove 2 → sum = 7 ≥ 7, length = 2 (optimal!)

---

## Why Not Use Brute Force?

### Brute Force Approach:
```python
def minSubArrayLen_brute(target, nums):
    n = len(nums)
    min_length = float('inf')
    
    # Try all possible subarrays
    for i in range(n):
        current_sum = 0
        for j in range(i, n):
            current_sum += nums[j]
            if current_sum >= target:
                min_length = min(min_length, j - i + 1)
                break  # No need to expand further
    
    return min_length if min_length != float('inf') else 0
```

**Complexity:** $O(n^2)$ in worst case

**Example where brute force is slow:**
```python
target = 100
nums = [1, 1, 1, ..., 1, 100]  # 99 ones followed by 100
# Brute force checks 99 + 98 + 97 + ... + 1 = O(n²) subarrays
# Sliding window: O(n) - visits each element at most twice
```

---

## Edge Cases

### 1. Single Element Solution
```python
target = 5
nums = [5]
# Output: 1
```

### 2. No Valid Subarray
```python
target = 100
nums = [1, 2, 3, 4, 5]
# Output: 0 (sum of all elements = 15 < 100)
```

### 3. Entire Array is Answer
```python
target = 15
nums = [1, 2, 3, 4, 5]
# Output: 5 (need entire array)
```

### 4. Large Single Element
```python
target = 5
nums = [1, 2, 10, 3, 4]
# Output: 1 (subarray [10])
```

### 5. All Elements Same
```python
target = 10
nums = [2, 2, 2, 2, 2, 2]
# Output: 5 (need [2,2,2,2,2] with sum=10)
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Window Size Calculation
```python
# Wrong!
min_length = min(min_length, right - left)
```

**Fix:**
```python
# Correct! Window size = right - left + 1
min_length = min(min_length, right - left + 1)
```

**Explanation:** If left=2 and right=4, the window contains indices [2,3,4], which is 3 elements.

### ❌ Mistake 2: Not Using While Loop for Shrinking
```python
# Wrong! Only shrinks once
if current_sum >= target:
    min_length = min(min_length, right - left + 1)
    current_sum -= nums[left]
    left += 1
```

**Fix:**
```python
# Correct! Shrink as much as possible
while current_sum >= target:
    min_length = min(min_length, right - left + 1)
    current_sum -= nums[left]
    left += 1
```

**Explanation:** After adding a large number, you might be able to remove multiple elements from the left.

### ❌ Mistake 3: Updating Min Length After Shrinking
```python
# Wrong order!
while current_sum >= target:
    current_sum -= nums[left]
    left += 1
    min_length = min(min_length, right - left + 1)  # Too late!
```

**Fix:**
```python
# Correct! Update BEFORE shrinking
while current_sum >= target:
    min_length = min(min_length, right - left + 1)
    current_sum -= nums[left]
    left += 1
```

---

## Interview Insights

**Question Recognition:**
- "Minimum/Maximum subarray" + "sum/product condition" → Sliding Window
- "Contiguous elements" → Sliding Window
- "All positive numbers" → Variable-size window works well

**Time Complexity Analysis:**
- **Interviewer:** "Isn't there a nested loop? How is it O(n)?"
- **You:** "Each element is added once (by right pointer) and removed at most once (by left pointer). Total operations = 2n = O(n)."

**Follow-up Questions:**

1. **Q:** "What if numbers can be negative?"
   - **A:** Sliding window doesn't work directly. Use prefix sum with hash map (see Subarray Sum Equals K).

2. **Q:** "What if we want maximum length instead?"
   - **A:** Similar approach, but shrink when sum > target to maintain validity.

3. **Q:** "Can you solve in O(n log n)?"
   - **A:** Yes, use binary search + prefix sum, but O(n) sliding window is optimal.

4. **Q:** "What if we want exactly equal to target?"
   - **A:** This becomes "Subarray Sum Equals K" - needs hash map approach.

**Optimization Discussion:**
- Already optimal at O(n) time and O(1) space
- Can't do better than O(n) since we must examine each element

---

## Variations

### Variation 1: Maximum Length Subarray with Sum ≤ Target
```python
def maxSubArrayLen(target, nums):
    left = 0
    current_sum = 0
    max_length = 0
    
    for right in range(len(nums)):
        current_sum += nums[right]
        
        # Shrink if sum exceeds target
        while current_sum > target:
            current_sum -= nums[left]
            left += 1
        
        # Update max length
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

### Variation 2: Count Subarrays with Sum ≥ Target
```python
def countSubarrays(target, nums):
    left = 0
    current_sum = 0
    count = 0
    
    for right in range(len(nums)):
        current_sum += nums[right]
        
        # All subarrays ending at right and starting from left to right are valid
        while current_sum >= target:
            count += len(nums) - right  # All subarrays from here to end
            current_sum -= nums[left]
            left += 1
    
    return count
```

---

## Related Problems

1. **Longest Substring Without Repeating Characters (LeetCode 3)** - Variable-size window
2. **Minimum Window Substring (LeetCode 76)** - More complex shrinking condition
3. **Subarray Product Less Than K (LeetCode 713)** - Similar sliding window
4. **Maximum Sum Subarray of Size K (LeetCode 643)** - Fixed-size window
5. **Fruit Into Baskets (LeetCode 904)** - At most K distinct elements

---

## Summary

**Key Takeaways:**

1. **Pattern:** Variable-size sliding window
   - Expand to find valid window
   - Shrink to minimize size

2. **Algorithm Steps:**
   - Use two pointers (left, right)
   - Expand with right pointer
   - Shrink with left pointer when condition met
   - Track minimum length

3. **Why O(n)?**
   - Each element visited at most twice
   - Right pointer: n iterations
   - Left pointer: at most n moves total

4. **When to Use:**
   - Contiguous subarray problems
   - Positive numbers (or monotonic condition)
   - Find minimum/maximum length

5. **Mental Model:** Think of a rubber band expanding and contracting. Expand to meet the target, then contract to find the minimum size.

**Template:**
```python
left = 0
window_sum = 0
result = float('inf')

for right in range(len(arr)):
    window_sum += arr[right]  # Expand
    
    while condition_met:       # Shrink
        result = update_result(result, window)
        window_sum -= arr[left]
        left += 1

return result if result != float('inf') else 0
```

---

## Advanced: Binary Search Approach

### A Hidden Follow-up: $O(n \log n)$ Solution

Interviewers sometimes ask if you can solve this in $O(n \log n)$. This is usually a hint to use **Binary Search**.

**Approach:**

1. Create a **Prefix Sum** array (which is monotonically increasing since all nums are positive).
2. For each starting index $i$, binary search for the smallest ending index $j$ such that:
   ```
   prefix[j] - prefix[i] >= target
   ```
3. Track the minimum length across all starting positions.

**Implementation:**

```python
def minSubArrayLen_binary_search(target, nums):
    n = len(nums)
    
    # Build prefix sum array
    prefix = [0] * (n + 1)
    for i in range(n):
        prefix[i + 1] = prefix[i] + nums[i]
    
    min_length = float('inf')
    
    # For each starting position
    for i in range(n):
        # Binary search for the smallest j where prefix[j] - prefix[i] >= target
        # We need: prefix[j] >= prefix[i] + target
        target_sum = prefix[i] + target
        
        # Binary search in prefix[i+1:]
        left, right = i + 1, n
        while left <= right:
            mid = (left + right) // 2
            if prefix[mid] >= target_sum:
                min_length = min(min_length, mid - i)
                right = mid - 1  # Try to find smaller
            else:
                left = mid + 1
    
    return min_length if min_length != float('inf') else 0
```

**Complexity:**
- **Time:** $O(n \log n)$ - For each of n positions, binary search takes $O(\log n)$
- **Space:** $O(n)$ - Prefix sum array

**Comparison:**

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| **Sliding Window** | $O(n)$ | $O(1)$ | **Preferred** - Optimal solution |
| **Binary Search** | $O(n \log n)$ | $O(n)$ | Good to know for follow-ups |
| **Brute Force** | $O(n^2)$ | $O(1)$ | Only for understanding |

**Note:** The $O(n)$ sliding window is almost always the **preferred answer**, but knowing the binary search alternative demonstrates deeper understanding of algorithmic techniques!