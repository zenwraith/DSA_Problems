# Largest Rectangle in Histogram

**LeetCode 84** | **Difficulty:** Hard | **Category:** Stack, Array, Monotonic Stack

---

## Problem Statement

Given an array of integers `heights` representing the histogram's bar height where the width of each bar is `1`, return the area of the largest rectangle in the histogram.

**Constraints:**
- `1 <= heights.length <= 10^5`
- `0 <= heights[i] <= 10^4`

---

## Examples

### Example 1:
```
Input: heights = [2,1,5,6,2,3]
Output: 10

Explanation:
The largest rectangle is shown in the diagram.
Bars: [2, 1, 5, 6, 2, 3]

      6
    5 6
    5 6
    5 6   3
2   5 6 2 3
2 1 5 6 2 3

The rectangle with area 10 has height 5 and width 2 (bars at indices 2-3).
```

### Example 2:
```
Input: heights = [2,4]
Output: 4

Explanation:
  4
  4
2 4

The largest rectangle has height 2 and width 2, or height 4 and width 1.
Both give area 4.
```

### Example 3:
```
Input: heights = [2,1,2]
Output: 3

Explanation:
2   2
2 1 2

Can't use height 2 for all three bars (middle bar is only 1).
Best is height 1, width 3, area = 3.
```

---

## Strategy

### Monotonic Stack Approach

**Key Insight:** For each bar, find the left and right boundaries where it's the minimum height, then calculate the rectangle area. Use a monotonic increasing stack to efficiently find these boundaries.

**Why This Works:**

```
Challenge: For each bar, we need to know:
  1. How far left can we extend with this bar's height?
  2. How far right can we extend with this bar's height?

Naive approach: For each bar, scan left and right O(n²)

Better approach: Monotonic increasing stack
  - Stack maintains indices of bars in increasing height order
  - When we encounter a shorter bar, all taller bars have found their right boundary
  - Pop taller bars and calculate their areas
  - The bar below in stack is the left boundary (last bar shorter than popped)

Why this is optimal:
  - Each bar pushed once: O(n)
  - Each bar popped once: O(n)
  - Total: O(n)

Key observations:
1. For a bar at index i with height h:
   - Left boundary = first bar to the left shorter than h
   - Right boundary = first bar to the right shorter than h
   - Width = right_boundary - left_boundary - 1

2. Stack property:
   - Indices with increasing heights
   - When height[i] < height[stack[-1]]: found right boundary for stack top
   - When popped, stack[-1] is the left boundary

3. Boundary trick:
   - Add sentinel -1 at stack start (left boundary for first elements)
   - Append 0 to heights (forces all bars to be popped)
```

**Algorithm Steps:**

1. **Initialize:**
   - Stack with sentinel [-1]
   - Append 0 to heights (boundary sentinel)
   - max_area = 0

2. **Process each position:**
   - While current height < stack top height:
     - Pop index from stack (this bar has found right boundary)
     - Height = heights[popped_index]
     - Width = current_index - stack[-1] - 1
     - Update max_area
   - Push current index

3. **Return max_area**

**Complexity:**
- **Time:** $O(n)$ - Each element pushed and popped at most once
- **Space:** $O(n)$ - Stack in worst case (strictly increasing heights)

---

## Solution

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        """
        Find largest rectangle area in histogram using monotonic stack.
        
        Strategy: Monotonic increasing stack
        1. For each bar, use it as minimum height of rectangle
        2. Find how far left and right we can extend
        3. Stack maintains bars in increasing height order
        4. When shorter bar arrives, pop taller bars (they found right boundary)
        5. Width = current_position - left_boundary - 1
        
        Key insight: Each bar is popped when we find a shorter bar to its right.
        At pop time, we know both left and right boundaries.
        
        Edge cases:
        - Single bar: Area = height
        - All same height: Area = height * width
        - Strictly increasing: Each bar processed when 0 appended
        - Strictly decreasing: Each bar processed immediately when next arrives
        
        Args:
            heights: Array of bar heights
            
        Returns:
            Maximum rectangle area
        """
        # Sentinel for left boundary calculation
        stack = [-1]
        
        # Append 0 to force all remaining bars to be popped
        heights.append(0)
        
        max_area = 0
        
        for i in range(len(heights)):
            # While current bar is shorter than stack top
            # (stack top has found its right boundary)
            while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
                # Pop the bar whose rectangle we're calculating
                h = heights[stack.pop()]
                
                # Width between current position and new stack top
                w = i - stack[-1] - 1
                
                # Update maximum area
                max_area = max(max_area, h * w)
            
            # Push current index to stack
            stack.append(i)
        
        # Clean up (remove the appended 0)
        heights.pop()
        
        return max_area
```

---

## Detailed Walkthrough

### Example 1: `heights = [2, 1, 5, 6, 2, 3]`

**Append 0:** `heights = [2, 1, 5, 6, 2, 3, 0]`

**Stack initialized:** `[-1]`

| i | height[i] | stack (before) | Comparison | Action | Area Calculation | max_area | stack (after) |
|---|-----------|----------------|------------|--------|------------------|----------|---------------|
| 0 | 2 | [-1] | - | Push 0 | - | 0 | [-1, 0] |
| 1 | 1 | [-1, 0] | 1 < 2 ✓ | Pop 0 | h=2, w=1-(-1)-1=1, area=2 | 2 | [-1] |
|   |   | [-1] | 1 < -1 ✗ | Push 1 | - | 2 | [-1, 1] |
| 2 | 5 | [-1, 1] | 5 < 1 ✗ | Push 2 | - | 2 | [-1, 1, 2] |
| 3 | 6 | [-1, 1, 2] | 6 < 5 ✗ | Push 3 | - | 2 | [-1, 1, 2, 3] |
| 4 | 2 | [-1, 1, 2, 3] | 2 < 6 ✓ | Pop 3 | h=6, w=4-2-1=1, area=6 | 6 | [-1, 1, 2] |
|   |   | [-1, 1, 2] | 2 < 5 ✓ | Pop 2 | h=5, w=4-1-1=2, area=10 ✓ | **10** | [-1, 1] |
|   |   | [-1, 1] | 2 < 1 ✗ | Push 4 | - | 10 | [-1, 1, 4] |
| 5 | 3 | [-1, 1, 4] | 3 < 2 ✗ | Push 5 | - | 10 | [-1, 1, 4, 5] |
| 6 | 0 | [-1, 1, 4, 5] | 0 < 3 ✓ | Pop 5 | h=3, w=6-4-1=1, area=3 | 10 | [-1, 1, 4] |
|   |   | [-1, 1, 4] | 0 < 2 ✓ | Pop 4 | h=2, w=6-1-1=4, area=8 | 10 | [-1, 1] |
|   |   | [-1, 1] | 0 < 1 ✓ | Pop 1 | h=1, w=6-(-1)-1=6, area=6 | 10 | [-1] |
|   |   | [-1] | 0 < -1 ✗ | Push 6 | - | 10 | [-1, 6] |

**Final max_area: 10** ✓

**The winning rectangle:**
- Height: 5 (from index 2)
- Width: 2 (indices 2-3, both have height ≥ 5)
- Area: 5 × 2 = 10

**Visualization:**
```
Index: 0  1  2  3  4  5
       2  1 [5][6] 2  3
             ^  ^
       Rectangle with height 5, width 2
```

---

### Example 2: `heights = [2, 4]`

**Append 0:** `heights = [2, 4, 0]`

```
i=0: height=2
  Stack: [-1]
  Push 0
  Stack: [-1, 0]

i=1: height=4
  Stack: [-1, 0]
  4 < 2? No
  Push 1
  Stack: [-1, 0, 1]

i=2: height=0
  Stack: [-1, 0, 1]
  
  0 < 4? Yes, pop 1
    h=4, w=2-0-1=1, area=4
    max_area=4
  Stack: [-1, 0]
  
  0 < 2? Yes, pop 0
    h=2, w=2-(-1)-1=2, area=4
    max_area=4
  Stack: [-1]
  
  Push 2
  Stack: [-1, 2]
```

**Output: 4** ✓

---

### Example 3: `heights = [2, 1, 2]` (Minimum in Middle)

**Append 0:** `heights = [2, 1, 2, 0]`

```
i=0: Push 0, stack=[-1, 0]

i=1: height=1
  1 < 2? Yes, pop 0
    h=2, w=1-(-1)-1=1, area=2
  Push 1, stack=[-1, 1]

i=2: height=2
  2 < 1? No
  Push 2, stack=[-1, 1, 2]

i=3: height=0
  0 < 2? Yes, pop 2
    h=2, w=3-1-1=1, area=2
  
  0 < 1? Yes, pop 1
    h=1, w=3-(-1)-1=3, area=3 ✓
  
  Push 3, stack=[-1, 3]
```

**Output: 3** ✓

The best rectangle uses height 1 (the minimum) across all 3 bars.

---

## Why This Algorithm Works

### The Width Formula

```
Width = i - stack[-1] - 1

Why this formula?

When we pop index j:
  - i = current position (first bar shorter than heights[j])
  - stack[-1] = index below j in stack (last bar shorter than heights[j])
  
These are the boundaries!
  - Left boundary: stack[-1] (exclusive)
  - Right boundary: i (exclusive)
  - Width: i - stack[-1] - 1

Example:
  heights = [3, 1, 5, 6, 4]
  When processing i=4 (height=4), we pop index 3 (height=6)
  
  Stack after pop: [-1, 0, 1, 2]
  stack[-1] = 2 (height=5)
  
  Width = 4 - 2 - 1 = 1 ✓
  (Only the bar at index 3 can have height 6)
```

### Monotonic Increasing Stack Property

```
Stack maintains indices with increasing heights.

Example stack state: [0, 2, 4, 5]
Heights:             [1, 3, 7, 9]
                      ↑  ↑  ↑  ↑
                     Increasing!

When shorter bar arrives, it breaks the monotonic property.
All taller bars must be popped (they've found their right boundary).

Example: Next height = 5
  Pop 5 (height 9): width determined, calculate area
  Pop 4 (height 7): width determined, calculate area
  Don't pop 2 (height 3 < 5): this is the left boundary
  Push new bar
```

### Why Append 0 at End

```
Without the 0:
  heights = [1, 2, 3, 4, 5]
  Stack at end: [-1, 0, 1, 2, 3, 4]
  
  Problem: Bars never get popped!
  We miss calculating areas for these bars.

With the 0:
  heights = [1, 2, 3, 4, 5, 0]
  When we process the 0, all bars are popped.
  All areas are calculated.

The 0 acts as a universal right boundary.
```

### Why Sentinel -1 in Stack

```
Without sentinel:
  heights = [5, 4, 3]
  When we pop index 0 (height 5):
    w = 1 - stack[-1] - 1
    But stack is empty! Error!

With sentinel -1:
  Stack: [-1, 0]
  When we pop index 0:
    w = 1 - (-1) - 1 = 1 ✓
  
The -1 acts as a universal left boundary (before all bars).
```

---

## Edge Cases

### 1. Single Bar
```python
heights = [5]
# Append 0: [5, 0]
# Stack: [-1]
# i=0: Push 0, stack=[-1, 0]
# i=1: Pop 0, h=5, w=1-(-1)-1=1, area=5
# Output: 5
```

### 2. All Same Height
```python
heights = [3, 3, 3, 3]
# When 0 appended, all bars popped
# Last pop: h=3, w=4-(-1)-1=4, area=12
# Output: 12
```

### 3. Strictly Increasing
```python
heights = [1, 2, 3, 4, 5]
# Stack grows: [-1, 0, 1, 2, 3, 4]
# When 0 appended, all pop in sequence
# Last bar (height 1) gets full width: area=1*5=5
# But height 5 gets: area=5*1=5
# Best individual or full width with min height
# Output: 10 (actually height 2, width 5 = 10... let me recalculate)
# Actually: when all pop at end:
#   Pop 4: h=5, w=5-3-1=1, area=5
#   Pop 3: h=4, w=5-2-1=2, area=8
#   Pop 2: h=3, w=5-1-1=3, area=9
#   Pop 1: h=2, w=5-0-1=4, area=8
#   Pop 0: h=1, w=5-(-1)-1=5, area=5
# Output: 9
```

### 4. Strictly Decreasing
```python
heights = [5, 4, 3, 2, 1]
# Each bar immediately pops previous
# Each has width 1
# Output: 5 (the tallest bar)
```

### 5. Mountain Pattern
```python
heights = [1, 2, 3, 2, 1]
# Peak at index 2
# Output: 6 (height 2, width 3) or (height 1, width 5)
# Actually 6
```

### 6. Valley Pattern
```python
heights = [3, 1, 3]
# Middle is minimum
# Best: height 1, width 3, area=3
# Or: height 3, width 1, area=3
# Output: 3
```

### 7. All Zeros
```python
heights = [0, 0, 0]
# Output: 0
```

### 8. Zeros in Middle
```python
heights = [2, 3, 0, 4, 5]
# Zero breaks the histogram into two parts
# Left part: max 6 (height 2, width 3... no, height 2 width 2=4, height 3 width 1=3)
# Actually: [2,3]: best is 4
# Right part: [4,5]: best is 8
# Output: 8
```

### 9. Large Numbers
```python
heights = [10000]
# Output: 10000
```

### 10. Two Bars Same Height
```python
heights = [5, 5]
# Output: 10 (height 5, width 2)
```

---

## Common Mistakes

### ❌ Mistake 1: Not Appending 0 at End
```python
# Wrong! Bars at end never get popped
for i in range(len(heights)):  # Should be after append
    while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
        # process

# Missing: heights.append(0) before loop
```

### ❌ Mistake 2: Not Using Sentinel -1
```python
# Wrong! Empty stack causes index error
stack = []  # Should be [-1]

while stack and heights[i] < heights[stack[-1]]:
    h = heights[stack.pop()]
    w = i - stack[-1] - 1  # Error if stack becomes empty!
```

### ❌ Mistake 3: Wrong Width Calculation
```python
# Wrong! Off-by-one errors
w = i - stack[-1]  # Missing -1

# Should be:
w = i - stack[-1] - 1
```

### ❌ Mistake 4: Checking stack[-1] != -1 Incorrectly
```python
# Wrong! This doesn't prevent accessing sentinel
while stack and heights[i] < heights[stack[-1]]:
    # heights[-1] would access last element of heights array!

# Correct: Check stack[-1] != -1
while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
```

### ❌ Mistake 5: Not Cleaning Up Appended 0
```python
# Wrong! Modifies input (bad practice)
heights.append(0)
# ... algorithm ...
# Missing: heights.pop()

# While it works, it's better to clean up
```

### ❌ Mistake 6: Using Values Instead of Indices in Stack
```python
# Wrong! Can't calculate width without indices
stack = [-1]
for i, h in enumerate(heights):
    while stack[-1] != -1 and h < stack[-1]:  # Comparing height to index!
        # Can't work

# Must store indices in stack
```

### ❌ Mistake 7: Forgetting to Push Current Index
```python
# Wrong! Current bar not added to stack
while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
    # process and pop
# Missing: stack.append(i)
```

### ❌ Mistake 8: Wrong Comparison (>= Instead of >)
```python
# Wrong! Equal heights shouldn't pop
while stack[-1] != -1 and heights[i] <= heights[stack[-1]]:

# Should be strict less than:
while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
```

---

## Interview Insights

**Question Recognition:**
- "Largest rectangle" + "histogram" → Monotonic stack
- "Maximum area" + "bars" → Stack-based approach
- Classic hard stack problem

**Key Points to Mention:**

1. **Monotonic stack:**
   - "I use a monotonic increasing stack to track bars. When a shorter bar arrives, it's the right boundary for all taller bars in the stack."

2. **Width calculation:**
   - "For each bar, the width is determined by finding the nearest shorter bars on both sides. The stack naturally provides these boundaries."

3. **Sentinel values:**
   - "I use -1 as a left boundary sentinel and append 0 as a right boundary sentinel to handle edge cases cleanly."

4. **Time complexity:**
   - "Each bar is pushed and popped exactly once, giving us O(n) time complexity despite the nested while loop."

5. **Why this is optimal:**
   - "Naive approaches require O(n²) to check all possible rectangles. The monotonic stack exploits the structure to achieve O(n)."

**Follow-up Questions:**

1. **Q:** "Can you do this without modifying the input array?"
   - **A:** "Yes, I can iterate to len(heights)+1 and treat the extra position as 0 without actually appending."

2. **Q:** "What if bars have different widths?"
   - **A:** "I'd need to track both height and width for each bar, adjusting the width calculation accordingly."

3. **Q:** "How would you find the rectangle with second-largest area?"
   - **A:** "Track top 2 areas during the algorithm, or store all rectangle areas and sort."

4. **Q:** "What about in a 2D matrix of 1's?"
   - **A:** "That's Maximal Rectangle (LeetCode 85). Treat each row as a histogram base and apply this algorithm."

5. **Q:** "Can you use divide and conquer instead?"
   - **A:** "Yes, find minimum height, split into left/right subarrays, recursively solve. But it's O(n log n) average, O(n²) worst case. Stack is better."

**Complexity Analysis:**
- Time: O(n) - Each element pushed/popped once
- Space: O(n) - Stack in worst case

---

## Related Problems

1. **Maximal Rectangle (LeetCode 85)** - Applies this algorithm to 2D matrix
2. **Trapping Rain Water (LeetCode 42)** - Similar monotonic stack pattern
3. **Sum of Subarray Minimums (LeetCode 907)** - Uses next/previous smaller element
4. **Maximum Score of a Good Subarray (LeetCode 1793)** - Similar stack approach
5. **Minimum Cost Tree From Leaf Values (LeetCode 1130)** - Monotonic stack application

---

## Variations

### Variation 1: Return Rectangle Coordinates
```python
def largestRectangleCoords(self, heights):
    """
    Return (area, left_index, right_index, height).
    """
    stack = [-1]
    heights.append(0)
    max_area = 0
    result = (0, 0, 0, 0)
    
    for i in range(len(heights)):
        while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
            idx = stack.pop()
            h = heights[idx]
            w = i - stack[-1] - 1
            area = h * w
            
            if area > max_area:
                max_area = area
                left = stack[-1] + 1
                right = i - 1
                result = (area, left, right, h)
        
        stack.append(i)
    
    heights.pop()
    return result
```

### Variation 2: Count All Maximum Rectangles
```python
def countMaxRectangles(self, heights):
    """
    Count how many rectangles have the maximum area.
    """
    stack = [-1]
    heights.append(0)
    max_area = 0
    count = 0
    
    for i in range(len(heights)):
        while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
            h = heights[stack.pop()]
            w = i - stack[-1] - 1
            area = h * w
            
            if area > max_area:
                max_area = area
                count = 1
            elif area == max_area:
                count += 1
        
        stack.append(i)
    
    heights.pop()
    return max_area, count
```

### Variation 3: With Minimum Height Constraint
```python
def largestRectangleMinHeight(self, heights, min_height):
    """
    Find largest rectangle with height at least min_height.
    """
    stack = [-1]
    heights.append(0)
    max_area = 0
    
    for i in range(len(heights)):
        while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
            h = heights[stack.pop()]
            
            # Only count if meets minimum height
            if h >= min_height:
                w = i - stack[-1] - 1
                max_area = max(max_area, h * w)
        
        stack.append(i)
    
    heights.pop()
    return max_area
```

### Variation 4: With Width Constraint
```python
def largestRectangleMaxWidth(self, heights, max_width):
    """
    Find largest rectangle with width at most max_width.
    """
    stack = [-1]
    heights.append(0)
    max_area = 0
    
    for i in range(len(heights)):
        while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
            h = heights[stack.pop()]
            w = i - stack[-1] - 1
            
            # Only count if width constraint satisfied
            if w <= max_width:
                max_area = max(max_area, h * w)
        
        stack.append(i)
    
    heights.pop()
    return max_area
```

### Variation 5: All Rectangles Above Threshold
```python
def allRectanglesAboveThreshold(self, heights, threshold):
    """
    Return all rectangles with area above threshold.
    """
    stack = [-1]
    heights.append(0)
    rectangles = []
    
    for i in range(len(heights)):
        while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
            idx = stack.pop()
            h = heights[idx]
            w = i - stack[-1] - 1
            area = h * w
            
            if area > threshold:
                rectangles.append((area, stack[-1] + 1, i - 1, h))
        
        stack.append(i)
    
    heights.pop()
    return rectangles
```

### Variation 6: Divide and Conquer Approach
```python
def largestRectangleAreaDC(self, heights):
    """
    Divide and conquer approach (less efficient but alternative).
    """
    def helper(left, right):
        if left > right:
            return 0
        
        # Find minimum height in range
        min_idx = left
        for i in range(left, right + 1):
            if heights[i] < heights[min_idx]:
                min_idx = i
        
        # Max of: full width with min height, or split at min
        return max(
            heights[min_idx] * (right - left + 1),
            helper(left, min_idx - 1),
            helper(min_idx + 1, right)
        )
    
    return helper(0, len(heights) - 1)
    # Time: O(n log n) average, O(n²) worst case
```

### Variation 7: With Negative Heights (Below Ground)
```python
def largestRectangleNegative(self, heights):
    """
    Bars can go below ground (negative heights).
    Find largest absolute area.
    """
    # Separate into positive and negative parts
    positive = [max(0, h) for h in heights]
    negative = [abs(min(0, h)) for h in heights]
    
    # Solve for both
    area_pos = self.largestRectangleArea(positive)
    area_neg = self.largestRectangleArea(negative)
    
    return max(area_pos, area_neg)
```

---

## Summary

**Key Takeaways:**

1. **Monotonic Stack Pattern:**
   - Stack maintains indices of increasing heights
   - When shorter bar arrives, pop taller bars
   - Each pop represents a rectangle calculation
   - Push current index to maintain invariant

2. **Boundary Calculation:**
   - Left boundary: stack[-1] after pop (last shorter bar)
   - Right boundary: current index i (first shorter bar)
   - Width: i - stack[-1] - 1 (exclusive boundaries)

3. **Sentinel Values:**
   - Stack starts with -1 (left boundary for all)
   - Append 0 to heights (right boundary for all)
   - Ensures clean boundary handling

4. **Algorithm Invariants:**
   - Stack always has at least one element (sentinel -1)
   - Stack indices have strictly increasing heights
   - Each bar pushed exactly once
   - Each bar popped at most once

5. **Time Complexity Analysis:**
   - n iterations through bars
   - Each bar pushed once: O(n)
   - Each bar popped at most once: O(n)
   - Total: O(n) despite nested while

**Mental Model:**
```
Think of each bar asking: "How wide can my rectangle be?"

To answer:
  - Look left: Find first shorter bar (stack provides this)
  - Look right: Find first shorter bar (current iteration provides this)
  - Width = distance between boundaries

Stack is like a memory:
  - Remembers bars we haven't finished processing
  - When shorter bar arrives, time to finish taller bars
  - Stack top is always "current contender" for width calculation
```

**Template (Largest Rectangle in Histogram):**
```python
def largestRectangleArea(self, heights):
    stack = [-1]  # Sentinel for left boundary
    heights.append(0)  # Sentinel for right boundary
    max_area = 0
    
    for i in range(len(heights)):
        # Pop taller bars (they found right boundary)
        while stack[-1] != -1 and heights[i] < heights[stack[-1]]:
            h = heights[stack.pop()]
            w = i - stack[-1] - 1
            max_area = max(max_area, h * w)
        
        stack.append(i)
    
    heights.pop()  # Clean up
    return max_area
```

This is THE classic monotonic stack problem and a building block for many other problems!
