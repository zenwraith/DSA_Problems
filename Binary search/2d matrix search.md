# Search a 2D Matrix

**LeetCode 74** | **Difficulty:** Medium | **Category:** Binary Search, Matrix, Array

---

## Problem Statement

You are given an `m x n` integer matrix `matrix` with the following two properties:

- Each row is sorted in non-decreasing order.
- The first integer of each row is greater than the last integer of the previous row.

Given an integer `target`, return `true` if `target` is in `matrix` or `false` otherwise.

You must write a solution in **O(log(m × n))** time complexity.

---

## Examples

### Example 1:
```
Input: matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
Output: true

Matrix:
[1,  3,  5,  7]
[10, 11, 16, 20]
[23, 30, 34, 60]

Target 3 is found at matrix[0][1]
```

### Example 2:
```
Input: matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
Output: false

Matrix:
[1,  3,  5,  7]
[10, 11, 16, 20]
[23, 30, 34, 60]

Target 13 is not in the matrix
```

### Example 3:
```
Input: matrix = [[1]], target = 1
Output: true
```

### Example 4:
```
Input: matrix = [[1]], target = 2
Output: false
```

---

## Strategy

### Treat 2D Matrix as 1D Sorted Array

**Key Insight:** The matrix properties make it equivalent to a **single sorted array**!

**Why This Works:**

```
Matrix:
[1,  3,  5,  7]
[10, 11, 16, 20]
[23, 30, 34, 60]

Think of it as:
[1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60]
 ↑                                          ↑
index 0                               index 11

Total elements: m × n = 3 × 4 = 12
```

**The Magic: Index Conversion**

```
1D index to 2D coordinates:
row = index // cols
col = index % cols

Example: index = 7 (value 20)
row = 7 // 4 = 1
col = 7 % 4 = 3
matrix[1][3] = 20 ✓

Example: index = 10 (value 34)
row = 10 // 4 = 2
col = 10 % 4 = 2
matrix[2][2] = 34 ✓
```

**Visual Mapping:**

```
Matrix indices:
[0,0] [0,1] [0,2] [0,3]
[1,0] [1,1] [1,2] [1,3]
[2,0] [2,1] [2,2] [2,3]

1D indices:
[0]   [1]   [2]   [3]
[4]   [5]   [6]   [7]
[8]   [9]   [10]  [11]

Formula:
1D → 2D: (index // cols, index % cols)
2D → 1D: row * cols + col
```

**Algorithm:**

1. Treat matrix as 1D array with `m × n` elements
2. Use standard binary search on indices `[0, m×n-1]`
3. Convert mid index to 2D coordinates to access value
4. Compare and adjust search space

**Complexity:**
- **Time:** $O(\log(m \times n))$ - Binary search on total elements
- **Space:** $O(1)$ - Only using pointers

---

## Solution

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        """
        Search for target in 2D matrix using binary search.
        
        Key insight: Matrix properties make it equivalent to a sorted 1D array.
        - Each row is sorted
        - First element of row i > last element of row i-1
        
        Strategy:
        1. Treat matrix as 1D array (virtual flattening)
        2. Binary search on indices [0, m*n-1]
        3. Convert mid index to 2D coordinates: (mid//cols, mid%cols)
        
        Args:
            matrix: 2D matrix with sorted rows
            target: Value to search for
            
        Returns:
            True if target exists in matrix, False otherwise
        """
        # Handle edge cases
        if not matrix or not matrix[0]:
            return False
        
        rows, cols = len(matrix), len(matrix[0])
        
        # Treat as 1D array: indices from 0 to (rows * cols - 1)
        left, right = 0, (rows * cols) - 1
        
        while left <= right:
            mid = (left + right) // 2
            
            # Convert 1D index to 2D coordinates
            mid_row = mid // cols
            mid_col = mid % cols
            mid_val = matrix[mid_row][mid_col]
            
            if mid_val == target:
                return True
            elif mid_val < target:
                # Target is in right half
                left = mid + 1
            else:
                # Target is in left half
                right = mid - 1
        
        return False
```

---

## Detailed Walkthrough

### Example 1: `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3`

**Setup:**
- rows = 3, cols = 4
- Total elements = 12
- Search range: [0, 11]

**Step-by-Step Execution:**

| Step | Left | Right | Mid | Row | Col | mid_val | Compare | Action |
|------|------|-------|-----|-----|-----|---------|---------|--------|
| 0 | 0 | 11 | 5 | 1 | 1 | 11 | 11 > 3 | right=4 |
| 1 | 0 | 4 | 2 | 0 | 2 | 5 | 5 > 3 | right=1 |
| 2 | 0 | 1 | 0 | 0 | 0 | 1 | 1 < 3 | left=1 |
| 3 | 1 | 1 | 1 | 0 | 1 | 3 | 3 == 3 | Found! |

**Visual Trace:**

```
Step 0: mid=5 → matrix[5//4][5%4] = matrix[1][1] = 11
[1,  3,  5,  7]
[10, 11*, 16, 20]  ← mid_val = 11 > 3
[23, 30, 34, 60]
Go left

Step 1: mid=2 → matrix[2//4][2%4] = matrix[0][2] = 5
[1,  3,  5*, 7]    ← mid_val = 5 > 3
[10, 11, 16, 20]
[23, 30, 34, 60]
Go left

Step 2: mid=0 → matrix[0//4][0%4] = matrix[0][0] = 1
[1*, 3,  5,  7]    ← mid_val = 1 < 3
[10, 11, 16, 20]
[23, 30, 34, 60]
Go right

Step 3: mid=1 → matrix[1//4][1%4] = matrix[0][1] = 3
[1,  3*, 5,  7]    ← mid_val = 3 == 3 ✓
[10, 11, 16, 20]
[23, 30, 34, 60]
Found!
```

**Final Answer:** `True` ✓

---

### Example 2: `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13`

| Step | Left | Right | Mid | Row | Col | mid_val | Compare | Action |
|------|------|-------|-----|-----|-----|---------|---------|--------|
| 0 | 0 | 11 | 5 | 1 | 1 | 11 | 11 < 13 | left=6 |
| 1 | 6 | 11 | 8 | 2 | 0 | 23 | 23 > 13 | right=7 |
| 2 | 6 | 7 | 6 | 1 | 2 | 16 | 16 > 13 | right=5 |
| 3 | 6 | 5 | - | - | - | - | left > right | Not found |

**Visual:**

```
Step 0: mid=5 → matrix[1][1] = 11 < 13, go right
Step 1: mid=8 → matrix[2][0] = 23 > 13, go left
Step 2: mid=6 → matrix[1][2] = 16 > 13, go left
Step 3: left=6 > right=5, not found
```

**Final Answer:** `False` ✓

---

### Example 3: `matrix = [[1,3],[5,7]], target = 5`

**Setup:**
- rows = 2, cols = 2
- Total elements = 4
- Search range: [0, 3]

| Step | Left | Right | Mid | Row | Col | mid_val | Compare | Action |
|------|------|-------|-----|-----|-----|---------|---------|--------|
| 0 | 0 | 3 | 1 | 0 | 1 | 3 | 3 < 5 | left=2 |
| 1 | 2 | 3 | 2 | 1 | 0 | 5 | 5 == 5 | Found! |

**Visual:**

```
Matrix:
[1, 3]
[5, 7]

As 1D: [1, 3, 5, 7]
             ↑
           index 2, value 5
```

**Final Answer:** `True` ✓

---

## Index Conversion Deep Dive

### 1D to 2D Conversion

**Formula:** 
```python
row = index // cols
col = index % cols
```

**Examples with 3×4 matrix (12 elements):**

| 1D Index | Calculation | Row | Col | Value |
|----------|-------------|-----|-----|-------|
| 0 | 0//4=0, 0%4=0 | 0 | 0 | matrix[0][0] |
| 1 | 1//4=0, 1%4=1 | 0 | 1 | matrix[0][1] |
| 3 | 3//4=0, 3%4=3 | 0 | 3 | matrix[0][3] |
| 4 | 4//4=1, 4%4=0 | 1 | 0 | matrix[1][0] |
| 7 | 7//4=1, 7%4=3 | 1 | 3 | matrix[1][3] |
| 8 | 8//4=2, 8%4=0 | 2 | 0 | matrix[2][0] |
| 11 | 11//4=2, 11%4=3 | 2 | 3 | matrix[2][3] |

**Why It Works:**

```
Division (//): Which row?
- Indices 0-3: 0//4, 1//4, 2//4, 3//4 → all give 0 (row 0)
- Indices 4-7: 4//4, 5//4, 6//4, 7//4 → all give 1 (row 1)
- Indices 8-11: 8//4, 9//4, 10//4, 11//4 → all give 2 (row 2)

Modulo (%): Which column within the row?
- Indices 0,4,8: 0%4, 4%4, 8%4 → all give 0 (col 0)
- Indices 1,5,9: 1%4, 5%4, 9%4 → all give 1 (col 1)
- Indices 3,7,11: 3%4, 7%4, 11%4 → all give 3 (col 3)
```

### 2D to 1D Conversion (Bonus)

**Formula:**
```python
index = row * cols + col
```

**Examples:**

| Row | Col | Calculation | 1D Index |
|-----|-----|-------------|----------|
| 0 | 0 | 0*4+0 | 0 |
| 0 | 3 | 0*4+3 | 3 |
| 1 | 0 | 1*4+0 | 4 |
| 1 | 3 | 1*4+3 | 7 |
| 2 | 0 | 2*4+0 | 8 |
| 2 | 3 | 2*4+3 | 11 |

---

## Edge Cases

### 1. Single Element Matrix
```python
matrix = [[5]], target = 5
# Output: True
# rows=1, cols=1, total=1
# left=0, right=0, mid=0
# matrix[0][0] = 5 == 5 ✓
```

### 2. Single Row Matrix
```python
matrix = [[1, 3, 5, 7]], target = 5
# Output: True
# Equivalent to standard 1D binary search
```

### 3. Single Column Matrix
```python
matrix = [[1], [3], [5], [7]], target = 5
# Output: True
# 1D indices [0,1,2,3]
# mid=1 → matrix[1//1][1%1] = matrix[1][0] = 3
```

### 4. Empty Matrix
```python
matrix = [], target = 1
# Output: False
# Early return check
```

### 5. Empty Rows
```python
matrix = [[]], target = 1
# Output: False
# Early return check: not matrix[0]
```

### 6. Target Smaller Than All
```python
matrix = [[1,3,5],[7,9,11]], target = 0
# Output: False
# Binary search naturally handles
```

### 7. Target Larger Than All
```python
matrix = [[1,3,5],[7,9,11]], target = 20
# Output: False
# Binary search naturally handles
```

### 8. Target Between Rows
```python
matrix = [[1,3,5],[10,11,16]], target = 8
# Output: False
# 8 is between rows, not in matrix
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Index Conversion
```python
# Wrong! Row and col swapped
mid_row = mid % cols  # Should be mid // cols
mid_col = mid // cols  # Should be mid % cols
```

**Fix:**
```python
mid_row = mid // cols  # Division for row
mid_col = mid % cols   # Modulo for col
```

### ❌ Mistake 2: Off-by-One in Range
```python
# Wrong! Should be (rows * cols - 1)
right = rows * cols
```

**Fix:**
```python
right = (rows * cols) - 1  # Indices are 0-based
```

### ❌ Mistake 3: Not Handling Empty Matrix
```python
# Wrong! Will crash on empty matrix
rows, cols = len(matrix), len(matrix[0])
```

**Fix:**
```python
if not matrix or not matrix[0]:
    return False
rows, cols = len(matrix), len(matrix[0])
```

### ❌ Mistake 4: Using left < right
```python
# Wrong! Should use <=
while left < right:
    # ...
```

**Fix:**
```python
while left <= right:
    # ...
```

### ❌ Mistake 5: Accessing Matrix Before Conversion
```python
# Wrong! mid is 1D index, not 2D
mid_val = matrix[mid]  # This will fail!
```

**Fix:**
```python
mid_val = matrix[mid // cols][mid % cols]
```

---

## Interview Insights

**Question Recognition:**
- "2D matrix" + "sorted rows" + "first of row i > last of row i-1" → Treat as 1D array
- O(log(m×n)) requirement confirms binary search

**Key Points to Mention:**

1. **Why treat as 1D:**
   - "The matrix properties guarantee it's equivalent to a sorted 1D array, so we can use standard binary search."

2. **Index conversion:**
   - "We convert 1D indices to 2D coordinates using: row = mid // cols, col = mid % cols."

3. **Complexity:**
   - "O(log(m×n)) time because we're doing binary search on m×n total elements."
   - "O(1) space since we only use a few variables."

4. **Edge cases:**
   - "Need to check for empty matrix before accessing dimensions."

**Follow-up Questions:**

1. **Q:** "What if rows are sorted but first of row i is NOT necessarily greater than last of row i-1?"
   - **A:** That's LeetCode 240. Need different approach: start from top-right or bottom-left, O(m+n).

2. **Q:** "Can you do better than O(log(m×n))?"
   - **A:** No, this is optimal. Need to potentially check log(m×n) positions in worst case.

3. **Q:** "What if you need to find the position, not just check existence?"
   - **A:** Return (mid // cols, mid % cols) instead of True.

4. **Q:** "How would you search without treating as 1D?"
   - **A:** Two binary searches: first find row (O(log m)), then search in that row (O(log n)). Total: O(log m + log n) ≈ O(log(m×n)).

**Complexity Analysis:**
- Time: O(log(m × n)) - Binary search on total elements
- Space: O(1) - Only using pointers

---

## Alternative Approaches

### Approach 1: Two Binary Searches
```python
def searchMatrix_TwoSearches(matrix, target):
    """
    First find the correct row, then search within that row.
    Time: O(log m + log n) ≈ O(log(m×n))
    """
    if not matrix or not matrix[0]:
        return False
    
    # Binary search for correct row
    top, bottom = 0, len(matrix) - 1
    
    while top <= bottom:
        mid_row = (top + bottom) // 2
        
        if matrix[mid_row][0] <= target <= matrix[mid_row][-1]:
            # Target is in this row
            row = mid_row
            break
        elif target < matrix[mid_row][0]:
            bottom = mid_row - 1
        else:
            top = mid_row + 1
    else:
        return False  # No valid row found
    
    # Binary search within the row
    left, right = 0, len(matrix[0]) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if matrix[row][mid] == target:
            return True
        elif matrix[row][mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return False
```

### Approach 2: Linear Row Search + Binary Column Search
```python
def searchMatrix_LinearRow(matrix, target):
    """
    Linear search for row, binary search in row.
    Time: O(m + log n) - Less efficient
    """
    if not matrix or not matrix[0]:
        return False
    
    # Linear search for row
    for row in matrix:
        if row[0] <= target <= row[-1]:
            # Binary search in this row
            left, right = 0, len(row) - 1
            
            while left <= right:
                mid = (left + right) // 2
                
                if row[mid] == target:
                    return True
                elif row[mid] < target:
                    left = mid + 1
                else:
                    right = mid - 1
            
            return False
    
    return False
```

---

## Related Problems

1. **Search a 2D Matrix II (LeetCode 240)** - Different properties, O(m+n) solution
2. **Find Peak Element II (LeetCode 1901)** - 2D peak finding
3. **Kth Smallest Element in Sorted Matrix (LeetCode 378)** - Binary search + counting
4. **Valid Sudoku (LeetCode 36)** - 2D matrix traversal
5. **Set Matrix Zeroes (LeetCode 73)** - 2D matrix manipulation
6. **Spiral Matrix (LeetCode 54)** - 2D matrix traversal pattern

---

## Problem Variations

### LeetCode 240: Search a 2D Matrix II

**Different Properties:**
- Rows sorted left to right
- Columns sorted top to bottom
- But NO guarantee about row vs previous row

```
Example:
[1,  4,  7,  11, 15]
[2,  5,  8,  12, 19]
[3,  6,  9,  16, 22]
[10, 13, 14, 17, 24]
[18, 21, 23, 26, 30]

Note: 18 > 15, so can't treat as 1D array!
```

**Solution:** Start from top-right or bottom-left

```python
def searchMatrix_II(matrix, target):
    """
    Search in matrix where rows and columns are sorted.
    Time: O(m + n)
    """
    if not matrix or not matrix[0]:
        return False
    
    # Start from top-right
    row, col = 0, len(matrix[0]) - 1
    
    while row < len(matrix) and col >= 0:
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] > target:
            col -= 1  # Move left
        else:
            row += 1  # Move down
    
    return False
```

---

## Summary

**Key Takeaways:**

1. **Virtual 1D Array:**
   - Matrix properties allow treating it as sorted 1D array
   - First of row i > last of row i-1 is the key property
   - Total elements: m × n

2. **Index Conversion Magic:**
   - 1D → 2D: `(mid // cols, mid % cols)`
   - Division gives row, modulo gives column
   - Enables O(1) access conversion

3. **Standard Binary Search:**
   - Once converted to 1D, use standard template
   - Search range: [0, m×n-1]
   - Same logic: compare, adjust left/right

4. **Edge Cases:**
   - Empty matrix: early return
   - Single element: works naturally
   - Target out of range: binary search handles

5. **Optimal Complexity:**
   - O(log(m×n)) time - can't do better
   - O(1) space - only need pointers
   - More efficient than two separate searches

**Template:**
```python
if not matrix or not matrix[0]:
    return False

rows, cols = len(matrix), len(matrix[0])
left, right = 0, (rows * cols) - 1

while left <= right:
    mid = (left + right) // 2
    
    # Convert 1D to 2D
    mid_row = mid // cols
    mid_col = mid % cols
    mid_val = matrix[mid_row][mid_col]
    
    if mid_val == target:
        return True
    elif mid_val < target:
        left = mid + 1
    else:
        right = mid - 1

return False
```

**Mental Model:**
- Imagine "unrolling" the matrix into a single line
- The row-sorted property with first[i] > last[i-1] makes this valid
- Use division/modulo to "fold" the line back into 2D when accessing
- Standard binary search then works perfectly
- This problem beautifully demonstrates how clever indexing can simplify algorithms!
