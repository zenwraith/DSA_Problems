# Maximal Rectangle

**LeetCode 85** | **Difficulty:** Hard | **Category:** Stack, Array, Dynamic Programming, Matrix, Monotonic Stack

---

## Problem Statement

Given a `rows x cols` binary matrix filled with `0`'s and `1`'s, find the largest rectangle containing only `1`'s and return its area.

**Constraints:**
- `rows == matrix.length`
- `cols == matrix[i].length`
- `1 <= rows, cols <= 200`
- `matrix[i][j]` is `'0'` or `'1'`

---

## Examples

### Example 1:
```
Input: matrix = [
  ["1","0","1","0","0"],
  ["1","0","1","1","1"],
  ["1","1","1","1","1"],
  ["1","0","0","1","0"]
]
Output: 6

Explanation:
The maximal rectangle is shown below:
  1 0 [1] 0 0
  1 0 [1 1 1]
  1 1 [1 1 1]
  1 0  0 1 0
Area = 2 rows * 3 cols = 6
```

### Example 2:
```
Input: matrix = [["0"]]
Output: 0
```

### Example 3:
```
Input: matrix = [["1"]]
Output: 1
```

### Example 4:
```
Input: matrix = [
  ["1","1","1","1"],
  ["1","1","1","1"],
  ["1","1","1","1"]
]
Output: 12

Explanation: Entire matrix is one rectangle
Area = 3 * 4 = 12
```

---

## Strategy

### Histogram-Based Approach with Monotonic Stack

**Key Insight:** Treat each row as the base of a histogram, where heights represent consecutive `1`'s above. Solve "Largest Rectangle in Histogram" for each row.

**Why This Works:**

```
Transformation Idea:
  For each row, build a histogram where:
    - Height at column i = number of consecutive 1's above (including current row)
    - If current cell is '0', height = 0 (breaks the chain)

Example:
  Matrix:        Heights after each row:
  1 0 1 0 0      Row 0: [1, 0, 1, 0, 0]
  1 0 1 1 1      Row 1: [2, 0, 2, 1, 1]
  1 1 1 1 1      Row 2: [3, 1, 3, 2, 2]
  1 0 0 1 0      Row 3: [4, 0, 0, 3, 0]

For each row's histogram, find largest rectangle using monotonic stack.

Why This Captures All Rectangles:
  - Every rectangle has a bottom row
  - When we process that row, the histogram captures that rectangle
  - Heights encode how far up we can extend
  - Width is determined by histogram algorithm

Algorithm Components:
1. Build heights array for each row (DP-like update)
2. For each row, solve largest rectangle in histogram
3. Track global maximum across all rows
```

**Algorithm Steps:**

1. **Initialize:**
   - `heights` array of size cols+1 (extra 0 at end for boundary)
   - `max_area` = 0

2. **For each row:**
   - **Update heights:**
     - If matrix[row][i] == '1': heights[i] += 1 (extend height)
     - If matrix[row][i] == '0': heights[i] = 0 (reset height)
   
   - **Find largest rectangle in histogram:**
     - Use monotonic increasing stack
     - For each height, calculate area when it's the minimum
     - Update max_area

3. **Return max_area**

**Histogram Algorithm (Monotonic Stack):**

```
Stack maintains indices of increasing heights.

For each position i:
  While current height < stack top height:
    - Pop the top (this is the minimum height in some rectangle)
    - Width = i - stack.top() - 1 (between current and new top)
    - Area = popped_height * width
    - Update max_area
  
  Push current index

Key: Extra 0 at end ensures all elements get popped
```

**Complexity:**
- **Time:** $O(rows \times cols)$ - Process each cell once, each histogram O(cols)
- **Space:** $O(cols)$ - Heights array and stack

---

## Solution

```python
class Solution:
    def maximalRectangle(self, matrix: List[List[str]]) -> int:
        """
        Find largest rectangle of 1's in binary matrix.
        
        Strategy: Histogram approach with monotonic stack
        1. Treat each row as base of histogram
        2. Heights[i] = consecutive 1's above row at column i
        3. For each row, solve "Largest Rectangle in Histogram"
        4. Use monotonic increasing stack for histogram problem
        
        Key insight: Every rectangle has a bottom row. When we process
        that row, the histogram captures the rectangle's dimensions.
        
        Edge cases:
        - Empty matrix: Return 0
        - All 0's: Return 0
        - All 1's: Return rows * cols
        - Single row/column: Reduce to histogram problem
        
        Args:
            matrix: Binary matrix of '0' and '1' strings
            
        Returns:
            Area of largest rectangle containing only 1's
        """
        if not matrix or not matrix[0]:
            return 0
        
        cols = len(matrix[0])
        # Extra 0 at end acts as boundary to flush stack
        heights = [0] * (cols + 1)
        max_area = 0
        
        for row in matrix:
            # Update heights for current row
            for i in range(cols):
                if row[i] == '1':
                    heights[i] += 1  # Extend height upward
                else:
                    heights[i] = 0   # Break the chain
            
            # Solve largest rectangle in histogram for current heights
            stack = [-1]  # Sentinel for width calculation
            
            for i in range(cols + 1):
                # While current height is less than stack top
                # (we've found the right boundary for rectangles ending here)
                while heights[i] < heights[stack[-1]]:
                    # Pop the height that's ending
                    h = heights[stack.pop()]
                    
                    # Width between current position and new stack top
                    w = i - stack[-1] - 1
                    
                    # Calculate area with this height as minimum
                    max_area = max(max_area, h * w)
                
                stack.append(i)
        
        return max_area
```

---

## Detailed Walkthrough

### Example: Matrix with Maximal Rectangle = 6

```
Matrix:
  ["1","0","1","0","0"]
  ["1","0","1","1","1"]
  ["1","1","1","1","1"]
  ["1","0","0","1","0"]
```

**Row 0: `["1","0","1","0","0"]`**

Heights update:
```
Initial: [0, 0, 0, 0, 0, 0]
After:   [1, 0, 1, 0, 0, 0]
         (extra 0 at end)
```

Histogram for `[1, 0, 1, 0, 0, 0]`:

| i | height | stack | Action | Area Calculation |
|---|--------|-------|--------|------------------|
| 0 | 1 | [-1] → [-1,0] | Push 0 | - |
| 1 | 0 | [-1,0] | Pop 0: h=1, w=1-(-1)-1=1 | 1*1=1 |
|   |   | [-1] → [-1,1] | Push 1 | - |
| 2 | 1 | [-1,1] → [-1,1,2] | Push 2 | - |
| 3 | 0 | [-1,1,2] | Pop 2: h=1, w=3-1-1=1 | 1*1=1 |
|   |   | [-1,1] | Pop 1: h=0, w=3-(-1)-1=3 | 0*3=0 |
|   |   | [-1] → [-1,3] | Push 3 | - |
| 4 | 0 | [-1,3] | Pop 3: h=0, w=4-(-1)-1=4 | 0*4=0 |
|   |   | [-1] → [-1,4] | Push 4 | - |
| 5 | 0 | [-1,4] | Pop 4: h=0, w=5-(-1)-1=5 | 0*5=0 |
|   |   | [-1] → [-1,5] | Push 5 | - |

Max after row 0: **1**

---

**Row 1: `["1","0","1","1","1"]`**

Heights update:
```
Previous: [1, 0, 1, 0, 0, 0]
Current row: '1' '0' '1' '1' '1'
After:    [2, 0, 2, 1, 1, 0]
```

Histogram for `[2, 0, 2, 1, 1, 0]`:

```
Process histogram:
  i=0: Push 0, stack=[-1,0]
  i=1: height=0
    Pop 0: h=2, w=1-(-1)-1=1, area=2*1=2
    Push 1, stack=[-1,1]
  
  i=2: height=2, Push 2, stack=[-1,1,2]
  
  i=3: height=1
    Pop 2: h=2, w=3-1-1=1, area=2*1=2
    Push 3, stack=[-1,1,3]
  
  i=4: height=1, Push 4, stack=[-1,1,3,4]
  
  i=5: height=0
    Pop 4: h=1, w=5-3-1=1, area=1*1=1
    Pop 3: h=1, w=5-1-1=3, area=1*3=3
    Pop 1: h=0, w=5-(-1)-1=5, area=0*5=0
    Push 5
```

Max after row 1: **3**

---

**Row 2: `["1","1","1","1","1"]`**

Heights update:
```
Previous: [2, 0, 2, 1, 1, 0]
Current row: '1' '1' '1' '1' '1'
After:    [3, 1, 3, 2, 2, 0]
```

Histogram for `[3, 1, 3, 2, 2, 0]`:

```
Process histogram:
  i=0: Push 0, stack=[-1,0]
  
  i=1: height=1
    Pop 0: h=3, w=1-(-1)-1=1, area=3*1=3
    Push 1, stack=[-1,1]
  
  i=2: height=3, Push 2, stack=[-1,1,2]
  
  i=3: height=2
    Pop 2: h=3, w=3-1-1=1, area=3*1=3
    Push 3, stack=[-1,1,3]
  
  i=4: height=2, Push 4, stack=[-1,1,3,4]
  
  i=5: height=0
    Pop 4: h=2, w=5-3-1=1, area=2*1=2
    Pop 3: h=2, w=5-1-1=3, area=2*3=6 ✓ (This is it!)
    Pop 1: h=1, w=5-(-1)-1=5, area=1*5=5
    Push 5
```

Max after row 2: **6** ✓

The rectangle with area 6 is at columns 2-4, rows 1-2:
```
  [1 1 1]  ← Height 2 at columns 2,3,4
  [1 1 1]    Width 3
```

---

**Row 3: `["1","0","0","1","0"]`**

Heights update:
```
Previous: [3, 1, 3, 2, 2, 0]
Current row: '1' '0' '0' '1' '0'
After:    [4, 0, 0, 3, 0, 0]
```

Max remains **6**.

**Final Answer:** 6

---

## Why This Algorithm Works

### The Histogram Transformation

```
Question: Why does converting to histograms capture all rectangles?

Answer: Every rectangle has a bottom row!

Proof:
  Consider any rectangle R in the matrix.
  Let bottom_row = lowest row of R.
  
  When we process bottom_row:
    - Heights[i] will be ≥ R's height for all columns in R
    - Histogram algorithm will find rectangles of various heights
    - Including one with R's dimensions (or larger)
  
  Therefore, we find all rectangles by checking each row.
```

### Height Update Logic

```
Why heights[i] += 1 or = 0?

Example column over rows:
  Row 0: '1' → height = 1
  Row 1: '1' → height = 2 (consecutive 1's)
  Row 2: '0' → height = 0 (chain broken)
  Row 3: '1' → height = 1 (new chain starts)

This encodes: "How far up can we extend from current row?"

Why this works:
  - Height represents the tallest bar at that column
  - Histogram algorithm finds widest rectangle for each height
  - Combined, we find largest rectangle ending at this row
```

### Monotonic Stack for Histogram

```
Why monotonic increasing stack?

Stack property: indices with increasing heights

When we encounter smaller height:
  - All taller bars in stack have found their right boundary
  - Pop them and calculate their areas
  - Width = current_index - new_stack_top - 1

Why this formula?
  - Popped bar's height is h
  - Left boundary = stack top after popping (last smaller element)
  - Right boundary = current index (first smaller element)
  - Width = distance between boundaries
```

### Extra 0 at End

```
Why heights = [0] * (cols + 1)?

The extra 0 ensures all bars get processed.

Without it:
  heights = [3, 2, 1]
  Stack at end: [-1, 0, 1, 2]
  
  Problem: Bars never get popped!

With extra 0:
  heights = [3, 2, 1, 0]
  When we process the 0, all bars pop.
  
This is a common trick for monotonic stack problems.
```

---

## Edge Cases

### 1. Empty Matrix
```python
matrix = []
# Output: 0
# Handled by: if not matrix or not matrix[0]
```

### 2. Single Cell - '1'
```python
matrix = [["1"]]
# Heights: [1, 0]
# Histogram: area = 1*1 = 1
# Output: 1
```

### 3. Single Cell - '0'
```python
matrix = [["0"]]
# Heights: [0, 0]
# Histogram: area = 0
# Output: 0
```

### 4. All 1's
```python
matrix = [
  ["1","1","1"],
  ["1","1","1"]
]
# After row 0: heights = [1,1,1,0], max = 3
# After row 1: heights = [2,2,2,0], max = 6
# Output: 6 (entire matrix)
```

### 5. All 0's
```python
matrix = [
  ["0","0","0"],
  ["0","0","0"]
]
# Heights always [0,0,0,0]
# Output: 0
```

### 6. Single Row
```python
matrix = [["1","0","1","1","1","0","1"]]
# Reduces to largest rectangle in histogram
# heights = [1,0,1,1,1,0,1,0]
# Max consecutive = 3
# Output: 3
```

### 7. Single Column
```python
matrix = [
  ["1"],
  ["1"],
  ["1"],
  ["0"],
  ["1"]
]
# Heights progression:
#   [1,0], [2,0], [3,0], [0,0], [1,0]
# Max area = 3*1 = 3
# Output: 3
```

### 8. Diagonal Pattern
```python
matrix = [
  ["1","0","0"],
  ["0","1","0"],
  ["0","0","1"]
]
# Each row: max area = 1
# Output: 1
```

### 9. L-Shape
```python
matrix = [
  ["1","1","1"],
  ["1","0","0"],
  ["1","0","0"]
]
# Row 0: heights=[1,1,1,0], area=3
# Row 1: heights=[2,0,0,0], area=2
# Row 2: heights=[3,0,0,0], area=3
# Output: 3
```

### 10. Multiple Rectangles
```python
matrix = [
  ["1","1","0","1","1"],
  ["1","1","0","1","1"]
]
# Two 2x2 rectangles, both area = 4
# Output: 4
```

---

## Common Mistakes

### ❌ Mistake 1: Not Adding Extra 0 to Heights
```python
# Wrong! Bars at end never get popped
heights = [0] * cols  # Should be cols + 1

# Problem: Last bars remain in stack
# They're never processed, potentially missing max area
```

### ❌ Mistake 2: Wrong Width Calculation
```python
# Wrong! Incorrect width formula
w = i - stack[-1]  # Missing -1

# Should be:
w = i - stack[-1] - 1

# Why: Exclusive boundaries need adjustment
```

### ❌ Mistake 3: Not Using Sentinel in Stack
```python
# Wrong! Empty stack causes issues
stack = []  # Should be [-1]

# Problem: When calculating width for first element
# stack[-1] would fail if stack becomes empty
```

### ❌ Mistake 4: Resetting Heights Incorrectly
```python
# Wrong! Not resetting on '0'
if row[i] == '1':
    heights[i] += 1
# Missing: else: heights[i] = 0

# Problem: Heights keep accumulating even through 0's
```

### ❌ Mistake 5: Comparing Strings Incorrectly
```python
# Wrong! Using == 1 instead of == '1'
if row[i] == 1:  # row[i] is string!

# Correct:
if row[i] == '1':  # Compare with string
```

### ❌ Mistake 6: Not Handling Empty Matrix
```python
# Wrong! Missing edge case check
def maximalRectangle(self, matrix):
    cols = len(matrix[0])  # Crashes if matrix is []!

# Correct:
if not matrix or not matrix[0]:
    return 0
```

### ❌ Mistake 7: Wrong Histogram Loop Range
```python
# Wrong! Missing last position
for i in range(cols):  # Should be cols + 1
    while heights[i] < heights[stack[-1]]:
        # process

# Missing: The extra 0 at position cols needs to be processed
```

### ❌ Mistake 8: Updating Heights After Histogram
```python
# Wrong! Order matters
# Calculate histogram first
for i in range(cols + 1):
    # histogram logic

# Update heights (wrong order!)
for i in range(cols):
    if row[i] == '1':
        heights[i] += 1

# Correct: Update heights BEFORE histogram
```

---

## Interview Insights

**Question Recognition:**
- "Largest rectangle" + "binary matrix" → Histogram approach
- "Rectangle of 1's" → Convert to histogram problem
- Hard stack problem → Often involves monotonic stack

**Key Points to Mention:**

1. **Histogram transformation:**
   - "I'll treat each row as the base of a histogram, where heights represent consecutive 1's above."

2. **Why this works:**
   - "Every rectangle has a bottom row. When we process that row, the histogram captures the rectangle's dimensions through the heights array."

3. **Monotonic stack:**
   - "For each histogram, I use a monotonic increasing stack to find the largest rectangle in O(n) time."

4. **Time complexity:**
   - "We process each cell once to build heights, and each histogram is O(cols). Total is O(rows × cols)."

5. **Space optimization:**
   - "We only need O(cols) space for the heights array and stack, processing row by row."

**Follow-up Questions:**

1. **Q:** "Can you do this without using the histogram approach?"
   - **A:** "Yes, there's a DP approach tracking left/right boundaries and heights, but it's more complex. Histogram approach is more intuitive."

2. **Q:** "What if we want to find all maximal rectangles, not just the largest?"
   - **A:** "We'd need to track all rectangles during the histogram phase, storing dimensions instead of just max area."

3. **Q:** "How would you modify this for '1's and '2's (two types)?"
   - **A:** "Would need separate histogram calculations for each value type, or modify height tracking to distinguish them."

4. **Q:** "Can you optimize space further?"
   - **A:** "We could modify the input matrix to store heights, but that's destructive. Current O(n) space is optimal for non-destructive solution."

5. **Q:** "What about finding the largest square instead of rectangle?"
   - **A:** "Different problem (LeetCode 221). Would use DP with min(left, top, diagonal) + 1."

**Complexity Analysis:**
- Time: O(rows × cols) - Process each cell once
- Space: O(cols) - Heights array and stack

---

## Related Problems

1. **Largest Rectangle in Histogram (LeetCode 84)** - Subproblem used here
2. **Maximal Square (LeetCode 221)** - Find largest square instead
3. **Count Submatrices With All Ones (LeetCode 1504)** - Count instead of max
4. **Max Sum of Rectangle No Larger Than K (LeetCode 363)** - Similar with sum constraint
5. **Number of Submatrices That Sum to Target (LeetCode 1074)** - Matrix subarray problem

---

## Variations

### Variation 1: Return Rectangle Coordinates
```python
def maximalRectangleCoords(self, matrix):
    """
    Return (area, top_row, left_col, height, width).
    """
    if not matrix or not matrix[0]:
        return (0, 0, 0, 0, 0)
    
    cols = len(matrix[0])
    heights = [0] * (cols + 1)
    max_area = 0
    result = None
    
    for row_idx, row in enumerate(matrix):
        for i in range(cols):
            heights[i] = heights[i] + 1 if row[i] == '1' else 0
        
        stack = [-1]
        for i in range(cols + 1):
            while heights[i] < heights[stack[-1]]:
                h = heights[stack.pop()]
                w = i - stack[-1] - 1
                area = h * w
                
                if area > max_area:
                    max_area = area
                    # Rectangle ends at current row
                    result = (area, row_idx - h + 1, stack[-1] + 1, h, w)
            
            stack.append(i)
    
    return result if result else (0, 0, 0, 0, 0)
```

### Variation 2: All Maximal Rectangles
```python
def allMaximalRectangles(self, matrix):
    """
    Find all maximal rectangles (not contained in larger ones).
    """
    if not matrix or not matrix[0]:
        return []
    
    rectangles = []
    # Similar to above, but store all during histogram processing
    # Filter out non-maximal ones at the end
    return rectangles
```

### Variation 3: With Minimum Size Constraint
```python
def maximalRectangleMinSize(self, matrix, min_height, min_width):
    """
    Find largest rectangle with at least min_height and min_width.
    """
    if not matrix or not matrix[0]:
        return 0
    
    cols = len(matrix[0])
    heights = [0] * (cols + 1)
    max_area = 0
    
    for row in matrix:
        for i in range(cols):
            heights[i] = heights[i] + 1 if row[i] == '1' else 0
        
        stack = [-1]
        for i in range(cols + 1):
            while heights[i] < heights[stack[-1]]:
                h = heights[stack.pop()]
                w = i - stack[-1] - 1
                
                # Only count if meets minimum requirements
                if h >= min_height and w >= min_width:
                    max_area = max(max_area, h * w)
            
            stack.append(i)
    
    return max_area
```

### Variation 4: Count All Rectangles
```python
def countRectangles(self, matrix):
    """
    Count all possible rectangles of 1's.
    """
    if not matrix or not matrix[0]:
        return 0
    
    count = 0
    # For each cell, count rectangles with that cell as top-left
    # More complex - requires different approach
    return count
```

### Variation 5: K Different Values
```python
def maximalRectangleKValues(self, matrix, target):
    """
    Find largest rectangle containing only 'target' value.
    Matrix can have multiple different values.
    """
    if not matrix or not matrix[0]:
        return 0
    
    cols = len(matrix[0])
    heights = [0] * (cols + 1)
    max_area = 0
    
    for row in matrix:
        for i in range(cols):
            # Only count if matches target
            heights[i] = heights[i] + 1 if row[i] == target else 0
        
        # Same histogram logic
        stack = [-1]
        for i in range(cols + 1):
            while heights[i] < heights[stack[-1]]:
                h = heights[stack.pop()]
                w = i - stack[-1] - 1
                max_area = max(max_area, h * w)
            stack.append(i)
    
    return max_area
```

### Variation 6: With Obstacles
```python
def maximalRectangleWithObstacles(self, matrix, obstacles):
    """
    Find largest rectangle avoiding obstacle positions.
    obstacles = [(row, col), ...]
    """
    # Mark obstacles as '0' in matrix copy
    matrix_copy = [row[:] for row in matrix]
    for r, c in obstacles:
        matrix_copy[r][c] = '0'
    
    # Use standard algorithm
    return self.maximalRectangle(matrix_copy)
```

### Variation 7: 3D Version (Cuboid)
```python
def maximalCuboid(self, matrix_3d):
    """
    Find largest cuboid in 3D binary matrix.
    Much more complex - needs 3D histogram approach.
    """
    # Significantly more complex
    # Would need to consider all depth slices
    pass
```

---

## Summary

**Key Takeaways:**

1. **Histogram Transformation:**
   - Treat each row as histogram base
   - Heights = consecutive 1's above (including current)
   - Reset height to 0 when encountering '0'
   - Every rectangle captured when processing its bottom row

2. **Monotonic Stack Pattern:**
   - Stack stores indices of increasing heights
   - When smaller height arrives, pop taller bars
   - Calculate area: height × width
   - Width = current - stack_top - 1

3. **Height Update Logic:**
   - If matrix[r][c] == '1': heights[c] += 1
   - If matrix[r][c] == '0': heights[c] = 0
   - This encodes "extend up" or "reset"

4. **Implementation Details:**
   - Extra 0 at end of heights (boundary sentinel)
   - Stack initialized with -1 (width calculation sentinel)
   - Process cols+1 positions (includes the extra 0)
   - Update heights before processing histogram

5. **Why This Is Optimal:**
   - Each cell visited once: O(rows × cols)
   - Each histogram solved in O(cols)
   - Total: O(rows × cols)
   - Space: O(cols) for heights and stack

**Mental Model:**
```
Think of layers stacking up:
  Layer 0: Original matrix
  Layer 1: Heights after row 0
  Layer 2: Heights after row 1
  ...

Each layer is a histogram problem.
Heights encode "how tall can bars be?"
Histogram algorithm finds "how wide can we go with each height?"

Combined: Largest rectangle!
```

**Template (Maximal Rectangle):**
```python
def maximalRectangle(self, matrix):
    if not matrix or not matrix[0]:
        return 0
    
    cols = len(matrix[0])
    heights = [0] * (cols + 1)  # Extra 0 at end
    max_area = 0
    
    for row in matrix:
        # Update heights
        for i in range(cols):
            heights[i] = heights[i] + 1 if row[i] == '1' else 0
        
        # Solve histogram
        stack = [-1]
        for i in range(cols + 1):
            while heights[i] < heights[stack[-1]]:
                h = heights[stack.pop()]
                w = i - stack[-1] - 1
                max_area = max(max_area, h * w)
            stack.append(i)
    
    return max_area
```

This is a beautiful combination of DP (height building) and monotonic stack (histogram solving)!
