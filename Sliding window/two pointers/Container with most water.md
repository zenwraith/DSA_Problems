# Container With Most Water

**LeetCode 11** | **Difficulty:** Medium | **Category:** Two Pointers, Greedy, Array

---

## Problem Statement

You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `i`th line are `(i, 0)` and `(i, height[i])`.

Find two lines that together with the x-axis form a container, such that the container contains the **most water**.

Return the **maximum amount of water** a container can store.

**Note:** You may not slant the container.

---

## Examples

### Example 1:
```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49

Explanation:
The vertical lines are:
8 |     █           █    
7 |     █           █   █
6 |     █   █       █   █
5 |     █   █   █   █   █
4 |     █   █   █ █ █   █
3 |     █   █   █ █ █ █ █
2 |     █   █ █ █ █ █ █ █
1 |   █ █   █ █ █ █ █ █ █
  +---------------------------
    0 1 2 3 4 5 6 7 8

Choose lines at index 1 (height=8) and index 8 (height=7):
- Width: 8 - 1 = 7
- Height: min(8, 7) = 7
- Area: 7 × 7 = 49 ✓
```

### Example 2:
```
Input: height = [1,1]
Output: 1

Explanation:
Only two lines, width = 1, height = min(1,1) = 1
Area = 1 × 1 = 1
```

### Example 3:
```
Input: height = [4,3,2,1,4]
Output: 16

Explanation:
Choose first and last lines:
- Width: 4 - 0 = 4
- Height: min(4, 4) = 4
- Area: 4 × 4 = 16
```

---

## Strategy

### Two Pointers (Greedy Approach)

**Key Insight:** The area is determined by:
$$\text{Area} = \text{width} \times \min(\text{height}[\text{left}], \text{height}[\text{right}])$$

**Critical Observation:**

Starting from the widest possible container (left=0, right=n-1), we have maximum width but possibly suboptimal heights. To potentially increase area:

- **Cannot increase width** (already at maximum)
- **Must increase the limiting height** (the shorter line)
- **Move the pointer at the shorter line** inward

**Why This Works:**

If we move the pointer at the **taller line**:
- Width decreases (guaranteed)
- Height can only decrease or stay same (limited by shorter line)
- Area **must decrease**

If we move the pointer at the **shorter line**:
- Width decreases (guaranteed)
- Height might increase (new line could be taller)
- Area might increase if new height gain outweighs width loss

**Mathematical Proof:**

```
Current area: A = w × min(h[l], h[r])

If h[l] < h[r] and we move right pointer:
    New area: A' = (w-1) × min(h[l], h[r-1])
    Since min(h[l], h[r-1]) ≤ h[l] < h[r]
    And w-1 < w
    Therefore: A' < A (definitely worse!)

If h[l] < h[r] and we move left pointer:
    New area: A' = (w-1) × min(h[l+1], h[r])
    If h[l+1] > h[l], we might get larger area
    Worth exploring!
```

**Algorithm:**

1. **Initialize:** Two pointers at both ends, track max area
2. **Calculate:** Current area with width and min height
3. **Update max:** Keep track of maximum area seen
4. **Move pointer:** Move the pointer pointing to shorter line
5. **Repeat:** Until pointers meet

**Complexity:**
- **Time:** $O(n)$ - Single pass through array
- **Space:** $O(1)$ - Only using pointers and variables

---

## Solution

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        """
        Find maximum water container area using two pointers.
        
        Key insight: Area = width × min(height[left], height[right])
        Move pointer at shorter height to potentially find taller line.
        
        Args:
            height: List of non-negative integers representing line heights
            
        Returns:
            Maximum area that can be formed between any two lines
        """
        left = 0
        right = len(height) - 1
        max_area = 0
        
        while left < right:
            # Calculate current container dimensions
            width = right - left
            current_height = min(height[left], height[right])
            current_area = width * current_height
            
            # Update maximum area if current is larger
            max_area = max(max_area, current_area)
            
            # Move pointer at shorter line (greedy choice)
            # This gives us the only chance to find a taller line
            if height[left] < height[right]:
                left += 1
            else:
                right -= 1
        
        return max_area
```

---

## Detailed Walkthrough

### Example: `height = [1,8,6,2,5,4,8,3,7]`

**Step-by-Step Execution:**

| Step | L | R | h[L] | h[R] | Width | Height | Area | Max | Move | Reason |
|------|---|---|------|------|-------|--------|------|-----|------|--------|
| 0 | 0 | 8 | 1 | 7 | 8 | 1 | 8 | 8 | L++ | 1 < 7 |
| 1 | 1 | 8 | 8 | 7 | 7 | 7 | 49 | 49 | R-- | 8 > 7 |
| 2 | 1 | 7 | 8 | 3 | 6 | 3 | 18 | 49 | R-- | 8 > 3 |
| 3 | 1 | 6 | 8 | 8 | 5 | 8 | 40 | 49 | L++ | 8 = 8 |
| 4 | 2 | 6 | 6 | 8 | 4 | 6 | 24 | 49 | L++ | 6 < 8 |
| 5 | 3 | 6 | 2 | 8 | 3 | 2 | 6 | 49 | L++ | 2 < 8 |
| 6 | 4 | 6 | 5 | 8 | 2 | 5 | 10 | 49 | L++ | 5 < 8 |
| 7 | 5 | 6 | 4 | 8 | 1 | 4 | 4 | 49 | L++ | 4 < 8 |
| End | 6 | 6 | - | - | - | - | - | 49 | Stop | L = R |

**Final Answer:** `49`

**Detailed Explanation of Key Steps:**

**Step 0 (L=0, R=8):**
- Width: 8, Heights: 1 and 7
- Area: 8 × min(1, 7) = 8 × 1 = 8
- Move left pointer since 1 < 7 (shorter line limits capacity)

**Step 1 (L=1, R=8):**
- Width: 7, Heights: 8 and 7
- Area: 7 × min(8, 7) = 7 × 7 = **49** ✓
- This is the maximum! Two tall lines with good width
- Move right pointer since 8 > 7

**Step 3 (L=1, R=6):**
- Width: 5, Heights: 8 and 8
- Area: 5 × min(8, 8) = 5 × 8 = 40
- Both lines equal height, but width too small
- Could move either pointer (we move left)

**Why Step 1 is Optimal:**
- Both lines are tall (8 and 7)
- Width is still large (7)
- Product gives maximum area (49)

---

## Why Greedy Works: Proof of Correctness

### The Greedy Choice

**Claim:** We never miss the optimal solution by always moving the pointer at the shorter line.

**Proof by Contradiction:**

Suppose the optimal solution uses lines at positions `i` and `j` where `i < j`.

When our algorithm's pointers are at positions `left` and `right`:

**Case 1:** `left < i` and `right > j`
- Our pointers haven't reached the optimal positions yet
- We'll eventually reach them as we move inward

**Case 2:** `left = i` and `right > j`
- Current left pointer is at optimal position
- Right pointer needs to move left to reach j
- Since we always move the shorter pointer, if `height[i] < height[right]`, we'd move left and miss it
- **BUT:** If `height[i] < height[right]`, then moving left is correct because:
  - Any container using `height[i]` with lines between `j` and `right` has width < (right - i)
  - Height is still limited by `height[i]`
  - So those containers can't be better than current one

**Case 3:** Similar reasoning for `left < i` and `right = j`

**Conclusion:** By always moving the shorter pointer, we explore all potentially optimal solutions without missing any.

---

## Alternative Approach: Brute Force

```python
def maxArea_bruteforce(height):
    """
    Try all possible pairs of lines.
    Time: O(n²), Space: O(1)
    """
    max_area = 0
    n = len(height)
    
    for i in range(n):
        for j in range(i + 1, n):
            width = j - i
            h = min(height[i], height[j])
            area = width * h
            max_area = max(max_area, area)
    
    return max_area
```

**Complexity:**
- **Time:** $O(n^2)$ - Check all pairs
- **Space:** $O(1)$ - Only variables

**Verdict:** Two pointers is **much better** - O(n) vs O(n²)!

---

## Comparison of Approaches

| Approach | Time | Space | Correctness | Interview |
|----------|------|-------|-------------|-----------|
| **Two Pointers** | $O(n)$ | $O(1)$ | Optimal | ✅ Best |
| **Brute Force** | $O(n^2)$ | $O(1)$ | Correct | ❌ Too slow |

---

## Edge Cases

### 1. Minimum Input
```python
height = [1, 1]
# Output: 1
# Only two lines, area = 1 × 1 = 1
```

### 2. All Same Height
```python
height = [5, 5, 5, 5, 5]
# Output: 20
# Widest container: 4 × 5 = 20
```

### 3. Increasing Heights
```python
height = [1, 2, 3, 4, 5]
# Output: 6
# Best: indices 0 and 4 → 4 × min(1, 5) = 4
# Or: indices 1 and 4 → 3 × min(2, 5) = 6 ✓
```

### 4. Decreasing Heights
```python
height = [5, 4, 3, 2, 1]
# Output: 6
# Best: indices 0 and 3 → 3 × min(5, 2) = 6
```

### 5. Two Tall Lines at Ends
```python
height = [9, 1, 1, 1, 1, 1, 9]
# Output: 54
# Widest with both tall: 6 × 9 = 54
```

### 6. Two Tall Lines in Middle
```python
height = [1, 9, 9, 1]
# Output: 9
# Best: indices 1 and 2 → 1 × 9 = 9
```

### 7. Single Tall Line
```python
height = [1, 100, 1, 1, 1]
# Output: 4
# Tall line doesn't help much without another tall line
# Best: indices 0 and 4 → 4 × 1 = 4
```

---

## Common Mistakes

### ❌ Mistake 1: Moving the Wrong Pointer
```python
# Wrong! Moving taller pointer guarantees worse result
if height[left] > height[right]:
    left += 1  # This will never improve area!
```

**Fix:** Always move the pointer at the shorter line:
```python
if height[left] < height[right]:
    left += 1
else:
    right -= 1
```

### ❌ Mistake 2: Forgetting to Calculate Area Before Moving
```python
# Wrong! Moving pointer before calculating
if height[left] < height[right]:
    left += 1
current_area = (right - left) * min(height[left], height[right])
```

**Fix:** Calculate area first, then move:
```python
current_area = (right - left) * min(height[left], height[right])
max_area = max(max_area, current_area)
if height[left] < height[right]:
    left += 1
```

### ❌ Mistake 3: Wrong Area Calculation
```python
# Wrong! Using max instead of min
current_area = width * max(height[left], height[right])
```

**Fix:** Water level is limited by shorter line:
```python
current_area = width * min(height[left], height[right])
```

### ❌ Mistake 4: Not Handling Equal Heights
```python
# Incomplete! What if heights are equal?
if height[left] < height[right]:
    left += 1
# Missing else case!
```

**Fix:** Handle equal case (can move either):
```python
if height[left] < height[right]:
    left += 1
else:
    right -= 1  # Handles both > and = cases
```

### ❌ Mistake 5: Wrong Loop Condition
```python
# Wrong! Will miss when left = right
while left <= right:
```

**Fix:** Need two distinct lines:
```python
while left < right:
```

---

## Interview Insights

**Question Recognition:**
- "Maximum area" + "two lines" + "container" → Two pointers
- "Cannot slant container" → Limited by shorter line

**Key Points to Mention:**

1. **Area formula:**
   - "Area equals width times the minimum of the two heights, since water level is limited by the shorter line."

2. **Greedy strategy:**
   - "We start with maximum width and move the pointer at the shorter line inward, seeking a taller line to compensate for width loss."

3. **Why move shorter pointer:**
   - "Moving the taller pointer guarantees a worse result since width decreases and height can't increase beyond the shorter line."

4. **Proof of correctness:**
   - "By moving the shorter pointer, we explore all potentially optimal solutions. Moving the taller pointer would only give us solutions we can prove are suboptimal."

**Follow-up Questions:**

1. **Q:** "What if we want to maximize the perimeter instead of area?"
   - **A:** Different problem! Would use different strategy, possibly considering sum of heights and width.

2. **Q:** "Can we use a stack instead?"
   - **A:** Stack doesn't help here. The two-pointer greedy approach is optimal for this specific problem.

3. **Q:** "What if lines have thickness?"
   - **A:** Adjust width calculation to account for thickness: `width = (right - left) - (thickness[left] + thickness[right]) / 2`.

4. **Q:** "How is this different from Trapping Rain Water?"
   - **A:** 
     - Container: Choose 2 lines, calculate area between them
     - Trapping: Use all bars, calculate water trapped between consecutive bars
     - Container: O(n) greedy, Trapping: O(n) two pointers with different logic

**Time Complexity Analysis:**
- Best case: O(n) - Must examine all or most elements
- Worst case: O(n) - Each pointer moves at most n times
- Average case: O(n) - Linear scan

---

## Variations

### Variation 1: Return the Indices
```python
def maxAreaWithIndices(height):
    """Return max area and the indices of two lines."""
    left, right = 0, len(height) - 1
    max_area = 0
    best_left, best_right = 0, len(height) - 1
    
    while left < right:
        width = right - left
        current_area = width * min(height[left], height[right])
        
        if current_area > max_area:
            max_area = current_area
            best_left = left
            best_right = right
        
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_area, best_left, best_right
```

### Variation 2: Maximum Area with K Lines
```python
def maxAreaKLines(height, k):
    """
    Find maximum area using exactly k lines (not a standard problem).
    This would require dynamic programming or more complex approach.
    Simple case k=2 is the original problem.
    """
    if k == 2:
        return maxArea(height)
    # For k > 2, would need DP approach
    # Complexity would be O(n^k) or O(n² × k) with DP
```

### Variation 3: Minimum Area (Smallest Container)
```python
def minArea(height):
    """Find minimum non-zero area between two lines."""
    min_area = float('inf')
    
    # Try consecutive pairs (minimum width)
    for i in range(len(height) - 1):
        width = 1
        h = min(height[i], height[i + 1])
        area = width * h
        if area > 0:
            min_area = min(min_area, area)
    
    return min_area if min_area != float('inf') else 0
```

### Variation 4: Count Containers Above Threshold
```python
def countContainersAboveThreshold(height, threshold):
    """Count how many containers have area >= threshold."""
    count = 0
    left, right = 0, len(height) - 1
    
    # Need to check all pairs, O(n²)
    for i in range(len(height)):
        for j in range(i + 1, len(height)):
            width = j - i
            area = width * min(height[i], height[j])
            if area >= threshold:
                count += 1
    
    return count
```

---

## Related Problems

1. **Trapping Rain Water (LeetCode 42)** - Similar concept but different logic
2. **Largest Rectangle in Histogram (LeetCode 84)** - Use all bars, stack-based
3. **Maximal Rectangle (LeetCode 85)** - 2D version of histogram
4. **Maximum Width Ramp (LeetCode 962)** - Different constraint, similar pointers
5. **Max Consecutive Ones III (LeetCode 1004)** - Two pointers pattern
6. **Boats to Save People (LeetCode 881)** - Two pointers with weight limit

---

## Comparison: Container vs Trapping Rain Water

| Aspect | Container With Most Water | Trapping Rain Water |
|--------|---------------------------|---------------------|
| **Goal** | Find 2 lines with max area | Calculate total water trapped |
| **Lines Used** | Choose any 2 lines | Use all bars |
| **Water Calc** | `width × min(h[l], h[r])` | `sum of (min(max_l, max_r) - h[i])` |
| **Pointers** | Move shorter line | Move based on max comparison |
| **Greedy** | Yes (move shorter) | Yes (process smaller max) |
| **Visual** | Area between 2 lines | Water on top of each bar |

---

## Summary

**Key Takeaways:**

1. **Area Formula:** `width × min(height[left], height[right])`
   - Width: Distance between lines
   - Height: Limited by shorter line (water overflows)

2. **Greedy Strategy:** Always move pointer at shorter line
   - Moving taller pointer: **Always worse** (width↓, height can't improve)
   - Moving shorter pointer: **Might improve** (could find taller line)

3. **Algorithm Pattern:**
   ```
   Start with widest container (both ends)
   While pointers don't meet:
       Calculate current area
       Update maximum
       Move pointer at shorter line
   ```

4. **Proof of Optimality:**
   - By moving shorter pointer, we never miss optimal solution
   - Moving taller pointer only gives provably suboptimal solutions
   - Greedy choice is safe and optimal

5. **Complexity:**
   - Time: $O(n)$ - Single pass, each pointer moves at most n times
   - Space: $O(1)$ - Only pointers and variables

**Template:**
```python
left, right = 0, len(height) - 1
max_area = 0

while left < right:
    # Calculate current container area
    width = right - left
    h = min(height[left], height[right])
    current_area = width * h
    
    # Update maximum
    max_area = max(max_area, current_area)
    
    # Move pointer at shorter line (greedy)
    if height[left] < height[right]:
        left += 1
    else:
        right -= 1

return max_area
```

**Key Insight:** This problem beautifully demonstrates the power of greedy algorithms with two pointers. The intuition is simple: we can only improve area by finding a taller line, and we can only find it by moving the pointer at the currently shorter line!
