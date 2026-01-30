# Two Sum II - Input Array Is Sorted

**LeetCode 167** | **Difficulty:** Medium | **Category:** Two Pointers, Binary Search

---

## Problem Statement

Given a **1-indexed** array of integers `numbers` that is already **sorted in non-decreasing order**, find two numbers such that they add up to a specific `target` number. Let these two numbers be `numbers[index1]` and `numbers[index2]` where `1 <= index1 < index2 <= numbers.length`.

Return the **indices** of the two numbers, `index1` and `index2`, **added by one** as an integer array `[index1, index2]` of length 2.

The tests are generated such that there is **exactly one solution**. You **may not** use the same element twice.

Your solution must use only **constant extra space**.

---

## Examples

### Example 1:
```
Input: numbers = [2,7,11,15], target = 9
Output: [1,2]

Explanation: 
The sum of 2 and 7 is 9. 
Therefore, index1 = 1, index2 = 2. 
We return [1, 2] (1-indexed).
```

### Example 2:
```
Input: numbers = [2,3,4], target = 6
Output: [1,3]

Explanation:
The sum of 2 and 4 is 6.
Therefore index1 = 1, index2 = 3.
```

### Example 3:
```
Input: numbers = [-1,0], target = -1
Output: [1,2]

Explanation:
The sum of -1 and 0 is -1.
Therefore index1 = 1, index2 = 2.
```

---

## Strategy

### Two Pointers Approach (Optimal)

**Key Insight:** Since the array is **sorted**, we can use two pointers starting from opposite ends. The sorted nature allows us to make intelligent decisions about which pointer to move.

**Algorithm Logic:**

1. **Initialize:** Place `left` at start (index 0), `right` at end (last index)
2. **Calculate Sum:** `current_sum = numbers[left] + numbers[right]`
3. **Make Decision:**
   - If `current_sum == target`: Found! Return `[left+1, right+1]` (1-indexed)
   - If `current_sum < target`: Sum too small → move `left++` to increase sum
   - If `current_sum > target`: Sum too large → move `right--` to decrease sum
4. **Repeat:** Continue until pointers meet

**Why This Works:**

The sorted property guarantees:
- Moving `left` right increases the sum (larger value)
- Moving `right` left decreases the sum (smaller value)
- We explore all valid pairs without duplicates
- Each pointer moves at most `n` times

**Visual Example:**
```
numbers = [2, 7, 11, 15], target = 9
           ↑           ↑
         left        right

Step 1: 2 + 15 = 17 > 9  → Move right left
Step 2: 2 + 11 = 13 > 9  → Move right left
Step 3: 2 + 7 = 9 == 9   → Found! Return [1, 2]
```

**Complexity:**
- **Time:** $O(n)$ - Each pointer moves at most n times
- **Space:** $O(1)$ - Only using two pointers

---

## Solution

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        """
        Two pointers approach for sorted array.
        
        Args:
            numbers: 1-indexed sorted array of integers
            target: Target sum to find
            
        Returns:
            [index1, index2] where numbers[index1-1] + numbers[index2-1] = target
        """
        left = 0
        right = len(numbers) - 1
        
        while left < right:
            current_sum = numbers[left] + numbers[right]
            
            # Found the target sum
            if current_sum == target:
                return [left + 1, right + 1]  # Convert to 1-indexed
            
            # Sum is too small, need a larger value
            elif current_sum < target:
                left += 1
            
            # Sum is too large, need a smaller value
            else:
                right -= 1
        
        # No solution found (shouldn't happen per problem constraints)
        return []
```

---

## Detailed Walkthrough

### Example: `numbers = [2, 7, 11, 15]`, `target = 9`

**Step-by-Step Execution:**

| Step | Left | Right | numbers[left] | numbers[right] | Sum | Comparison | Action | Result |
|------|------|-------|---------------|----------------|-----|------------|--------|--------|
| 0 | 0 | 3 | 2 | 15 | 17 | 17 > 9 | right-- | - |
| 1 | 0 | 2 | 2 | 11 | 13 | 13 > 9 | right-- | - |
| 2 | 0 | 1 | 2 | 7 | 9 | 9 == 9 | Found! | [1, 2] |

**Detailed Explanation:**

**Step 0:** `left=0, right=3`
- `numbers[0] = 2, numbers[3] = 15`
- `current_sum = 2 + 15 = 17`
- `17 > 9` → Sum too large, decrease it by moving right pointer left
- `right = 2`

**Step 1:** `left=0, right=2`
- `numbers[0] = 2, numbers[2] = 11`
- `current_sum = 2 + 11 = 13`
- `13 > 9` → Sum still too large, continue decreasing
- `right = 1`

**Step 2:** `left=0, right=1`
- `numbers[0] = 2, numbers[1] = 7`
- `current_sum = 2 + 7 = 9`
- `9 == 9` → Found target!
- Return `[0+1, 1+1] = [1, 2]` (converted to 1-indexed)

---

## Alternative Approach: Binary Search

For each element, use binary search to find the complement:

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        """
        Binary search approach: For each number, search for its complement.
        Less efficient than two pointers but demonstrates another valid approach.
        """
        for i in range(len(numbers)):
            complement = target - numbers[i]
            
            # Binary search for complement in the remaining array
            left, right = i + 1, len(numbers) - 1
            
            while left <= right:
                mid = (left + right) // 2
                
                if numbers[mid] == complement:
                    return [i + 1, mid + 1]  # 1-indexed
                elif numbers[mid] < complement:
                    left = mid + 1
                else:
                    right = mid - 1
        
        return []
```

**Complexity:**
- **Time:** $O(n \log n)$ - For each element, binary search takes O(log n)
- **Space:** $O(1)$ - Only using pointers

**Verdict:** Two pointers is **better** - simpler and faster!

---

## Why Two Pointers Beats Hash Map Here

### Hash Map Approach (Classic Two Sum)
```python
def twoSum_hashmap(numbers, target):
    seen = {}
    for i, num in enumerate(numbers):
        complement = target - num
        if complement in seen:
            return [seen[complement] + 1, i + 1]
        seen[num] = i
    return []
```

**Comparison:**

| Approach | Time | Space | Best For |
|----------|------|-------|----------|
| **Two Pointers** | $O(n)$ | $O(1)$ | Sorted arrays |
| **Hash Map** | $O(n)$ | $O(n)$ | Unsorted arrays |
| **Binary Search** | $O(n \log n)$ | $O(1)$ | When space is critical |

**For sorted arrays:** Two pointers is optimal because:
1. Same time complexity as hash map
2. No extra space needed
3. Simpler implementation
4. Leverages the sorted property

---

## Edge Cases

### 1. Negative Numbers
```python
numbers = [-5, -3, -1, 0, 3, 5], target = 0
# Output: [1, 6]
# -5 + 5 = 0
```

### 2. All Negative
```python
numbers = [-10, -5, -2, -1], target = -12
# Output: [1, 3]
# -10 + (-2) = -12
```

### 3. Minimum Array Size
```python
numbers = [1, 2], target = 3
# Output: [1, 2]
# Only two elements
```

### 4. Duplicate Values
```python
numbers = [1, 2, 2, 3, 4], target = 4
# Output: [1, 4] or [2, 3]
# Multiple valid pairs possible, return any
```

### 5. Large Numbers
```python
numbers = [1, 1000000000], target = 1000000001
# Output: [1, 2]
# Test overflow handling
```

### 6. Target at Extremes
```python
numbers = [1, 2, 3, 4, 5], target = 9
# Output: [4, 5]
# Sum of last two elements
```

---

## Common Mistakes

### ❌ Mistake 1: Forgetting 1-Indexed Return
```python
# Wrong! Returns 0-indexed
if current_sum == target:
    return [left, right]
```

**Fix:** Add 1 to both indices
```python
if current_sum == target:
    return [left + 1, right + 1]
```

### ❌ Mistake 2: Wrong Loop Condition
```python
# Wrong! Will miss the case where left == right
while left <= right:
    current_sum = numbers[left] + numbers[right]
```

**Fix:** Use `left < right` since we need two different elements
```python
while left < right:
    current_sum = numbers[left] + numbers[right]
```

### ❌ Mistake 3: Not Moving Pointers
```python
# Wrong! Infinite loop if not found
while left < right:
    if current_sum == target:
        return [left + 1, right + 1]
    # Missing: else cases to move pointers!
```

**Fix:** Always move a pointer in else cases
```python
while left < right:
    if current_sum == target:
        return [left + 1, right + 1]
    elif current_sum < target:
        left += 1
    else:
        right -= 1
```

### ❌ Mistake 4: Using Hash Map Unnecessarily
```python
# Works but wastes O(n) space!
seen = {}
for i, num in enumerate(numbers):
    # ... hash map logic
```

**Fix:** Use two pointers to leverage sorted property with O(1) space.

---

## Interview Insights

**Question Recognition:**
- "Sorted array" + "find two elements" + "constant space" → Two pointers
- If unsorted → Consider hash map approach

**Key Points to Mention:**

1. **Why two pointers over hash map?**
   - "Since the array is sorted, we can use two pointers with O(1) space instead of O(n) space for a hash map."

2. **Why start from opposite ends?**
   - "Starting from both ends allows us to adjust the sum up or down by moving the appropriate pointer based on the comparison with target."

3. **Proof of correctness:**
   - "We never miss a valid pair because when we move a pointer, we've exhausted all possibilities with the other pointer's current position."

4. **1-indexed return:**
   - "The problem requires 1-indexed output, so we return [left+1, right+1] instead of [left, right]."

**Follow-up Questions:**

1. **Q:** "What if the array is unsorted?"
   - **A:** Either sort first (O(n log n)) then two pointers, or use hash map (O(n) time, O(n) space).

2. **Q:** "What if there are multiple valid pairs?"
   - **A:** This solution returns the first pair found. To find all pairs, we'd continue searching after finding each match.

3. **Q:** "Can you find three numbers that sum to target?"
   - **A:** Yes! 3Sum problem - fix one element and use two pointers for the rest (O(n²)).

4. **Q:** "What if we can use the same element twice?"
   - **A:** Modify condition to `while left <= right` but check `if left != right` when found.

**Time Complexity Analysis:**
- Best case: O(1) - target at the extremes
- Average case: O(n/2) - target in middle
- Worst case: O(n) - no solution or target requires specific positions

---

## Variations

### Variation 1: Return All Pairs
```python
def twoSum_allPairs(numbers, target):
    """Find all unique pairs that sum to target."""
    result = []
    left = 0
    right = len(numbers) - 1
    
    while left < right:
        current_sum = numbers[left] + numbers[right]
        
        if current_sum == target:
            result.append([left + 1, right + 1])
            left += 1
            right -= 1
            # Skip duplicates
            while left < right and numbers[left] == numbers[left - 1]:
                left += 1
            while left < right and numbers[right] == numbers[right + 1]:
                right -= 1
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    
    return result
```

### Variation 2: Count Pairs Instead of Returning Indices
```python
def countPairs(numbers, target):
    """Count number of pairs that sum to target."""
    count = 0
    left = 0
    right = len(numbers) - 1
    
    while left < right:
        current_sum = numbers[left] + numbers[right]
        
        if current_sum == target:
            count += 1
            left += 1
            right -= 1
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    
    return count
```

### Variation 3: Two Sum Less Than Target
```python
def twoSumLessThanTarget(numbers, target):
    """Find two numbers with maximum sum less than target."""
    left = 0
    right = len(numbers) - 1
    max_sum = float('-inf')
    
    while left < right:
        current_sum = numbers[left] + numbers[right]
        
        if current_sum < target:
            max_sum = max(max_sum, current_sum)
            left += 1
        else:
            right -= 1
    
    return max_sum if max_sum != float('-inf') else -1
```

---

## Related Problems

1. **Two Sum (LeetCode 1)** - Unsorted array, use hash map
2. **3Sum (LeetCode 15)** - Find triplets, fix one + two pointers
3. **4Sum (LeetCode 18)** - Find quadruplets, fix two + two pointers
4. **Two Sum IV - BST (LeetCode 653)** - Two pointers on BST inorder
5. **3Sum Closest (LeetCode 16)** - Find sum closest to target
6. **Two Sum Less Than K (LeetCode 1099)** - Maximum sum < target

---

## Summary

**Key Takeaways:**

1. **Two Pointers Pattern:** Start from both ends, move based on comparison
   - `sum < target` → move left pointer right (increase sum)
   - `sum > target` → move right pointer left (decrease sum)
   - `sum == target` → found solution!

2. **Why This Works:** Sorted array property
   - Smaller values on left, larger on right
   - Moving pointers changes sum predictably
   - Each pointer moves at most n times

3. **Advantages Over Hash Map:**
   - O(1) space vs O(n) space
   - Simpler code
   - Leverages sorted property

4. **Complexity:**
   - Time: $O(n)$ - Linear scan with two pointers
   - Space: $O(1)$ - Only two pointer variables

5. **Remember:** Problem uses **1-indexed** output, add 1 to both indices!

**Template:**
```python
left, right = 0, len(numbers) - 1

while left < right:
    current_sum = numbers[left] + numbers[right]
    
    if current_sum == target:
        return [left + 1, right + 1]  # 1-indexed!
    elif current_sum < target:
        left += 1  # Need larger sum
    else:
        right -= 1  # Need smaller sum

return []
```
