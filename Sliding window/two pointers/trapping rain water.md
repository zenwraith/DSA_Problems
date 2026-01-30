# Trapping Rain Water

**LeetCode 42** | **Difficulty:** Hard | **Category:** Two Pointers, Dynamic Programming, Stack

---

## Problem Statement

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

---

## Examples

### Example 1:
```
Input: height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Explanation:
The elevation map (represented by array [0,1,0,2,1,0,1,3,2,1,2,1]) can trap 6 units.

Visual representation:
    █
    █ █ █
█ █ █ █ █ █
█ █ █ █ █ █
0 1 0 2 1 0 1 3 2 1 2 1

Water trapped:
    █
  ▓ █ ▓ █
█ ▓ █ █ █ █ █
█ ▓ █ █ ▓ █ █
0 1 0 2 1 0 1 3 2 1 2 1

Water (▓) = 1 + 1 + 2 + 1 + 1 = 6
```

### Example 2:
```
Input: height = [4,2,0,3,2,5]
Output: 9

Visual:
█       █
█   █   █
█ █ █   █
█ █ █ █ █
4 2 0 3 2 5

With water:
█ ▓ ▓ ▓ ▓ █
█ ▓ ▓ █ ▓ █
█ █ ▓ █ █ █
█ █ ▓ █ █ █
4 2 0 3 2 5

Water = 2 + 4 + 1 + 2 = 9
```

### Example 3:
```
Input: height = [4,2,3]
Output: 1

Visual:
█
█   █
█ █ █
4 2 3

With water:
█
█ ▓ █
█ █ █
4 2 3

Water = 1
```

---

## Strategy

### Two Pointers Approach (Optimal)

**Key Insight:** Water trapped at any position depends on the **minimum** of the maximum heights to its left and right.

**Water at position i:**
$$\text{water}[i] = \min(\text{max\_left}[i], \text{max\_right}[i]) - \text{height}[i]$$

But we don't need to pre-calculate all max values!

**Critical Observation:**

When comparing two ends:
- If `left_max < right_max`, we **know** the water at `left` is limited by `left_max`
- The actual `right_max` doesn't matter as long as it's larger!
- We can safely calculate water at `left` position and move inward

**Why This Works:**

```
If left_max < right_max:
    water[left] = min(left_max, right_max) - height[left]
                = left_max - height[left]  (since left_max is smaller)
    
The actual right_max value doesn't affect this calculation!
```

**Algorithm:**

1. **Initialize:** Two pointers at both ends, track `left_max` and `right_max`
2. **Compare maxes:** Determine which side has smaller max height
3. **Calculate water:** On the side with smaller max, add trapped water
4. **Move pointer:** Move inward from the side we just processed
5. **Update max:** Update the max for the side we moved
6. **Repeat:** Continue until pointers meet

**Visual Example:**
```
height = [0,1,0,2,1,0,1,3,2,1,2,1]
          L →               ← R

left_max = 0, right_max = 1
0 < 1 → Process left side
Move left, water = max(0, height[1]) - height[1] = 0
```

**Complexity:**
- **Time:** $O(n)$ - Single pass through array
- **Space:** $O(1)$ - Only using pointers and variables

---

## Solution

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        """
        Calculate trapped rainwater using two pointers.
        
        Key insight: Water at position i = min(max_left, max_right) - height[i]
        We process from the side with smaller max to avoid needing both maxes.
        
        Args:
            height: List of non-negative integers representing elevation
            
        Returns:
            Total units of water trapped
        """
        if not height:
            return 0
        
        # Initialize two pointers
        left, right = 0, len(height) - 1
        
        # Track maximum heights seen so far from each side
        left_max = height[left]
        right_max = height[right]
        
        res = 0
        
        while left < right:
            # Process the side with smaller max height
            if left_max < right_max:
                # Move left pointer inward
                left += 1
                
                # Update left_max if current height is taller
                left_max = max(left_max, height[left])
                
                # Add trapped water at current position
                # Water = left_max - height[left]
                # (guaranteed non-negative since left_max >= height[left])
                res += left_max - height[left]
            
            else:
                # Move right pointer inward
                right -= 1
                
                # Update right_max if current height is taller
                right_max = max(right_max, height[right])
                
                # Add trapped water at current position
                res += right_max - height[right]
        
        return res
```

---

## Detailed Walkthrough

### Example: `height = [0,1,0,2,1,0,1,3,2,1,2,1]`

**Step-by-Step Execution:**

| Step | L | R | height[L] | height[R] | left_max | right_max | Compare | Side | Water Added | Total | Explanation |
|------|---|---|-----------|-----------|----------|-----------|---------|------|-------------|-------|-------------|
| 0 | 0 | 11 | 0 | 1 | 0 | 1 | 0 < 1 | Left | 0 | 0 | Initial state |
| 1 | 1 | 11 | 1 | 1 | 1 | 1 | 1 = 1 | Right | 0 | 0 | 1-1=0, no water |
| 2 | 1 | 10 | 1 | 2 | 1 | 2 | 1 < 2 | Left | 0 | 0 | 1-1=0 |
| 3 | 2 | 10 | 0 | 2 | 1 | 2 | 1 < 2 | Left | 1 | 1 | 1-0=1 ✓ |
| 4 | 3 | 10 | 2 | 2 | 2 | 2 | 2 = 2 | Right | 0 | 1 | 2-2=0 |
| 5 | 3 | 9 | 2 | 1 | 2 | 2 | 2 = 2 | Right | 1 | 2 | 2-1=1 ✓ |
| 6 | 3 | 8 | 2 | 2 | 2 | 2 | 2 = 2 | Right | 0 | 2 | 2-2=0 |
| 7 | 3 | 7 | 2 | 3 | 2 | 3 | 2 < 3 | Left | 1 | 3 | 2-1=1 ✓ |
| 8 | 4 | 7 | 1 | 3 | 2 | 3 | 2 < 3 | Left | 0 | 3 | 2-0=2... wait |
| 8 | 4 | 7 | 1 | 3 | 2 | 3 | 2 < 3 | Left | 1 | 4 | 2-1=1 ✓ |
| 9 | 5 | 7 | 0 | 3 | 2 | 3 | 2 < 3 | Left | 2 | 6 | 2-0=2 ✓ |
| 10 | 6 | 7 | 1 | 3 | 2 | 3 | 2 < 3 | Left | 1 | 7 | 2-1=1... hmm |

Let me recalculate more carefully:

| Step | L | R | height[L] | height[R] | left_max | right_max | Compare | Water | Total |
|------|---|---|-----------|-----------|----------|-----------|---------|-------|-------|
| Init | 0 | 11 | 0 | 1 | 0 | 1 | - | - | 0 |
| 1 | 1 | 11 | 1 | 1 | 1 | 1 | 0<1, L++ | 1-1=0 | 0 |
| 2 | 2 | 11 | 0 | 1 | 1 | 1 | 1=1, R-- | 1-1=0 | 0 |
| 3 | 2 | 10 | 0 | 2 | 1 | 2 | 1<2, L++ | 1-0=1 | 1 |
| 4 | 3 | 10 | 2 | 2 | 2 | 2 | 2=2, R-- | 2-2=0 | 1 |
| 5 | 3 | 9 | 2 | 1 | 2 | 2 | 2=2, R-- | 2-1=1 | 2 |
| 6 | 3 | 8 | 2 | 2 | 2 | 2 | 2=2, R-- | 2-2=0 | 2 |
| 7 | 3 | 7 | 2 | 3 | 2 | 3 | 2<3, L++ | 2-2=0 | 2 |
| 8 | 4 | 7 | 1 | 3 | 2 | 3 | 2<3, L++ | 2-1=1 | 3 |
| 9 | 5 | 7 | 0 | 3 | 2 | 3 | 2<3, L++ | 2-0=2 | 5 |
| 10 | 6 | 7 | 1 | 3 | 2 | 3 | 2<3, L++ | 2-1=1 | 6 |
| End | 7 | 7 | - | - | - | - | L=R | - | 6 |

**Final Answer:** `6`

**Detailed Explanation of Key Steps:**

**Step 3 (L=2):**
- `height[2] = 0`, `left_max = 1`, `right_max = 2`
- Since `1 < 2`, we process left side
- Water trapped: `1 - 0 = 1` unit
- There's a wall of height 1 on the left, so position 2 can hold 1 unit

**Step 9 (L=5):**
- `height[5] = 0`, `left_max = 2`, `right_max = 3`
- Since `2 < 3`, we process left side
- Water trapped: `2 - 0 = 2` units
- Position 5 is a valley, can hold 2 units of water

---

## Alternative Approaches

### Approach 1: Dynamic Programming (Precompute Maxes)

```python
def trap_dp(height):
    """
    Precompute max heights for all positions.
    Time: O(n), Space: O(n)
    """
    if not height:
        return 0
    
    n = len(height)
    
    # Precompute max height to the left of each position
    left_max = [0] * n
    left_max[0] = height[0]
    for i in range(1, n):
        left_max[i] = max(left_max[i - 1], height[i])
    
    # Precompute max height to the right of each position
    right_max = [0] * n
    right_max[n - 1] = height[n - 1]
    for i in range(n - 2, -1, -1):
        right_max[i] = max(right_max[i + 1], height[i])
    
    # Calculate trapped water
    water = 0
    for i in range(n):
        water += min(left_max[i], right_max[i]) - height[i]
    
    return water
```

**Complexity:**
- **Time:** $O(n)$ - Three passes
- **Space:** $O(n)$ - Two arrays

**Verdict:** Two pointers is **better** - same time, O(1) space!

### Approach 2: Stack (Monotonic Decreasing)

```python
def trap_stack(height):
    """
    Use monotonic stack to calculate water layer by layer.
    Time: O(n), Space: O(n)
    """
    if not height:
        return 0
    
    stack = []
    water = 0
    
    for i in range(len(height)):
        # Pop shorter bars and calculate trapped water
        while stack and height[i] > height[stack[-1]]:
            top = stack.pop()
            
            if not stack:
                break
            
            # Calculate water between stack[-1] and i
            distance = i - stack[-1] - 1
            bounded_height = min(height[i], height[stack[-1]]) - height[top]
            water += distance * bounded_height
        
        stack.append(i)
    
    return water
```

**Complexity:**
- **Time:** $O(n)$ - Each element pushed/popped once
- **Space:** $O(n)$ - Stack storage

**Verdict:** Interesting approach but two pointers is simpler!

---

## Comparison of Approaches

| Approach | Time | Space | Difficulty | Best For |
|----------|------|-------|------------|----------|
| **Two Pointers** | $O(n)$ | $O(1)$ | Medium | Optimal solution |
| **Dynamic Programming** | $O(n)$ | $O(n)$ | Easy | Learning/visualization |
| **Stack** | $O(n)$ | $O(n)$ | Hard | Understanding layers |

**Best Choice:** Two pointers for interviews - optimal complexity, elegant logic!

---

## Edge Cases

### 1. Empty Array
```python
height = []
# Output: 0
# No elevation, no water
```

### 2. Single Element
```python
height = [5]
# Output: 0
# Need at least 2 walls to trap water
```

### 3. Flat Elevation
```python
height = [3, 3, 3, 3]
# Output: 0
# No valleys, can't trap water
```

### 4. Monotonic Increasing
```python
height = [1, 2, 3, 4, 5]
# Output: 0
# No valleys, all water flows off
```

### 5. Monotonic Decreasing
```python
height = [5, 4, 3, 2, 1]
# Output: 0
# No right wall to trap water
```

### 6. Valley Shape
```python
height = [3, 0, 0, 0, 3]
# Output: 9
# Perfect valley: 3 + 3 + 3 = 9
```

### 7. Two Peaks
```python
height = [5, 1, 5]
# Output: 4
# Simple case: 5 - 1 = 4
```

### 8. All Zeros
```python
height = [0, 0, 0, 0]
# Output: 0
# No walls at all
```

---

## Common Mistakes

### ❌ Mistake 1: Not Checking Empty Array
```python
# Wrong! Will crash on empty array
left, right = 0, len(height) - 1
left_max = height[left]  # IndexError if empty!
```

**Fix:** Add check at the beginning:
```python
if not height:
    return 0
```

### ❌ Mistake 2: Wrong Water Calculation
```python
# Wrong! This can be negative
res += height[left] - left_max
```

**Fix:** Always subtract current height from max:
```python
res += left_max - height[left]
```

### ❌ Mistake 3: Not Updating Max Before Calculation
```python
# Wrong! Using old max
if left_max < right_max:
    left += 1
    res += left_max - height[left]  # Old left_max!
    left_max = max(left_max, height[left])
```

**Fix:** Update max before calculating water:
```python
if left_max < right_max:
    left += 1
    left_max = max(left_max, height[left])
    res += left_max - height[left]
```

### ❌ Mistake 4: Wrong Loop Condition
```python
# Wrong! Will skip middle element
while left <= right:
```

**Fix:** Use `left < right` since we process after moving:
```python
while left < right:
```

### ❌ Mistake 5: Initializing Max to 0
```python
# Wrong! Should start with actual heights
left_max, right_max = 0, 0
```

**Fix:** Initialize with first/last heights:
```python
left_max = height[left]
right_max = height[right]
```

---

## Interview Insights

**Question Recognition:**
- "Trap water" + "elevation map" → Two pointers or DP
- "Calculate volume" between bars → Think about boundaries

**Key Points to Mention:**

1. **Water formula:**
   - "Water at each position equals the minimum of max heights on both sides, minus the current height."

2. **Why two pointers works:**
   - "We don't need both maxes simultaneously. We can process from the side with smaller max, since that's the limiting factor."

3. **Time and space:**
   - "Two pointers gives us O(n) time with O(1) space, which is optimal compared to DP's O(n) space."

4. **Why move from smaller side:**
   - "The smaller max determines the water level, so we know the exact water amount without needing the other max."

**Follow-up Questions:**

1. **Q:** "What if bars have different widths?"
   - **A:** Multiply water by width for each position. Formula becomes: `water = width * (min(left_max, right_max) - height)`.

2. **Q:** "How would you solve this in 2D (trap water in a 2D grid)?"
   - **A:** Use priority queue (min-heap) with boundary cells, similar to Dijkstra's algorithm. Process from outside in.

3. **Q:** "What if you need to return water at each position, not total?"
   - **A:** Use DP approach to precompute, or modify two pointers to store results in array.

4. **Q:** "Can you optimize for very sparse arrays (mostly zeros)?"
   - **A:** Skip consecutive zeros, only process positions between walls.

**Complexity Analysis:**
- Best case: O(n) - Must examine all elements
- Worst case: O(n) - Linear scan regardless of input
- Average case: O(n) - Consistent performance

---

## Variations

### Variation 1: Return Water at Each Position
```python
def trap_array(height):
    """Return array showing water at each position."""
    if not height:
        return []
    
    n = len(height)
    left_max = [0] * n
    right_max = [0] * n
    water = [0] * n
    
    # Compute left_max
    left_max[0] = height[0]
    for i in range(1, n):
        left_max[i] = max(left_max[i - 1], height[i])
    
    # Compute right_max
    right_max[n - 1] = height[n - 1]
    for i in range(n - 2, -1, -1):
        right_max[i] = max(right_max[i + 1], height[i])
    
    # Compute water at each position
    for i in range(n):
        water[i] = min(left_max[i], right_max[i]) - height[i]
    
    return water
```

### Variation 2: Container With Most Water
```python
def maxArea(height):
    """LeetCode 11: Find max area between two bars (no filling)."""
    left, right = 0, len(height) - 1
    max_area = 0
    
    while left < right:
        # Area = width × min(height[left], height[right])
        width = right - left
        area = width * min(height[left], height[right])
        max_area = max(max_area, area)
        
        # Move pointer with shorter height
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_area
```

### Variation 3: Trapping Rain Water II (2D)
```python
def trapRainWater2D(heightMap):
    """LeetCode 407: Trap water in 2D grid."""
    import heapq
    
    if not heightMap or not heightMap[0]:
        return 0
    
    m, n = len(heightMap), len(heightMap[0])
    visited = [[False] * n for _ in range(m)]
    heap = []
    
    # Add all boundary cells to heap
    for i in range(m):
        for j in range(n):
            if i == 0 or i == m - 1 or j == 0 or j == n - 1:
                heapq.heappush(heap, (heightMap[i][j], i, j))
                visited[i][j] = True
    
    water = 0
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    
    # Process from outside to inside
    while heap:
        height, x, y = heapq.heappop(heap)
        
        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            
            if 0 <= nx < m and 0 <= ny < n and not visited[nx][ny]:
                # Water level is determined by boundary
                water += max(0, height - heightMap[nx][ny])
                # Update boundary with max of current cell and boundary
                heapq.heappush(heap, (max(height, heightMap[nx][ny]), nx, ny))
                visited[nx][ny] = True
    
    return water
```

---

## Related Problems

1. **Container With Most Water (LeetCode 11)** - Two pointers, max area (no filling)
2. **Trapping Rain Water II (LeetCode 407)** - 2D version, use priority queue
3. **Pour Water (LeetCode 755)** - Simulate water pouring
4. **Maximum Width Ramp (LeetCode 962)** - Similar max tracking
5. **Largest Rectangle in Histogram (LeetCode 84)** - Stack-based, similar concept
6. **Maximal Rectangle (LeetCode 85)** - 2D histogram

---

## Summary

**Key Takeaways:**

1. **Water Formula:** `water[i] = min(max_left[i], max_right[i]) - height[i]`

2. **Two Pointers Optimization:** 
   - Don't need both maxes simultaneously
   - Process from side with smaller max
   - The smaller max is the limiting factor

3. **Algorithm Pattern:**
   ```
   Initialize pointers at both ends
   Track max heights from both sides
   While left < right:
       Compare maxes
       Process side with smaller max
       Update max and add water
       Move pointer inward
   ```

4. **Why It Works:**
   - If `left_max < right_max`, water at left is limited by `left_max`
   - Actual `right_max` doesn't matter (we know it's larger)
   - Safe to calculate water and move left pointer

5. **Complexity:**
   - Time: $O(n)$ - Single pass
   - Space: $O(1)$ - Only pointers and variables

**Template:**
```python
if not height:
    return 0

left, right = 0, len(height) - 1
left_max = height[left]
right_max = height[right]
water = 0

while left < right:
    if left_max < right_max:
        left += 1
        left_max = max(left_max, height[left])
        water += left_max - height[left]
    else:
        right -= 1
        right_max = max(right_max, height[right])
        water += right_max - height[right]

return water
```

---

## Summary of Two Pointers Pattern

You've now mastered the main flavors of Two Pointers:

### 1. **Convergence** (Opposite Ends)
Moving toward the middle from both ends.
- **Two Sum II:** Find pair with target sum
- **3Sum:** Fix one + two pointers for remaining
- **Trapping Rain Water:** Process from side with smaller max
- **Container With Most Water:** Maximize area between bars

### 2. **Fast & Slow** (Same Direction, Different Speeds)
Detecting cycles or finding specific positions.
- **Linked List Cycle:** Detect cycles
- **Find Middle:** Slow moves 1, fast moves 2
- **Remove Nth from End:** Fast gets head start

### 3. **Sliding Window** (Adjacent Pointers)
Maintaining a contiguous segment.
- **Fixed Size:** Exact window size (Permutation in String)
- **Variable Size:** Expand/contract based on condition (Longest Substring)
- **Monotonic:** Maintain order property (Sliding Window Maximum)

**Common Characteristics:**
- O(n) or O(n²) time complexity
- O(1) space complexity
- Requires sorted array OR specific pattern
- Avoid O(n²) or O(n³) brute force
