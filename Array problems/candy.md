# Candy

**LeetCode 135** | **Difficulty:** Hard | **Category:** Array, Greedy

---

## Problem Statement

There are `n` children standing in a line. Each child is assigned a rating value given in the integer array `ratings`.

You are giving candies to these children subjected to the following requirements:

- Each child must have at least one candy.
- Children with a higher rating get more candies than their neighbors.

Return the **minimum number of candies** you need to have to distribute the candies to the children.

**Constraints:**
- `n == ratings.length`
- `1 <= n <= 2 * 10^4`
- `0 <= ratings[i] <= 2 * 10^4`

---

## Examples

### Example 1:
```
Input: ratings = [1,0,2]
Output: 5

Explanation:
You can allocate to the first, second and third child with 2, 1, 2 candies respectively.
```

### Example 2:
```
Input: ratings = [1,2,2]
Output: 4

Explanation:
You can allocate to the first, second and third child with 1, 2, 1 candies respectively.
The third child gets 1 candy because it only needs to satisfy the right neighbor (who has lower rating).
```

### Example 3:
```
Input: ratings = [1,3,2,2,1]
Output: 7

Explanation:
Allocation: [1, 2, 1, 2, 1]
- Index 1: rating 3 > ratings[0] and ratings[2], so gets 2 candies
- Index 3: rating 2 > ratings[4], so gets 2 candies
- Others get 1 candy each
```

---

## Strategy

### Two-Pass Greedy Algorithm

**Key Insight:** Handle left-to-right and right-to-left constraints independently, then merge them.

**Why This Works:**

```
Challenge: A child can have BOTH left and right neighbors with lower ratings
  - Must satisfy: candy[i] > candy[i-1] if rating[i] > rating[i-1]
  - Must satisfy: candy[i] > candy[i+1] if rating[i] > rating[i+1]
  - Both constraints might apply simultaneously!

Problem: If we try to satisfy both in one pass, we get stuck
  Example: [1, 3, 2]
    - Child 1 needs more than child 0 (left constraint)
    - Child 1 needs more than child 2 (right constraint)
    - But we don't know child 2's value until we process it!

Solution: Two-Pass Approach
  Pass 1 (Left→Right): Satisfy left neighbor constraint only
  Pass 2 (Right→Left): Satisfy right neighbor constraint, take max to preserve pass 1

Key: max(candies[i], candies[i+1] + 1) ensures we don't violate earlier constraints
```

**Algorithm Steps:**

1. **Initialize:** Every child gets 1 candy (minimum requirement)
   ```python
   candies = [1] * n
   ```

2. **Pass 1 (Left → Right):** If child has higher rating than left neighbor, give them one more candy than left neighbor
   ```python
   for i in range(1, n):
       if ratings[i] > ratings[i-1]:
           candies[i] = candies[i-1] + 1
   ```

3. **Pass 2 (Right → Left):** If child has higher rating than right neighbor, ensure they have at least one more candy than right neighbor (take max to preserve pass 1 results)
   ```python
   for i in range(n-2, -1, -1):
       if ratings[i] > ratings[i+1]:
           candies[i] = max(candies[i], candies[i+1] + 1)
   ```

4. **Sum:** Return total candies needed

**Why Two Passes Work:**

```
After Pass 1: All left constraints satisfied
After Pass 2: All right constraints satisfied WITHOUT breaking left constraints

The max() in pass 2 is crucial:
  - If pass 1 already gave enough candies, keep it
  - If right constraint needs more, update it
  - This ensures BOTH constraints are met
```

**Complexity:**
- **Time:** $O(n)$ - Two linear passes
- **Space:** $O(n)$ - Candies array

---

## Solution

```python
class Solution:
    def candy(self, ratings: List[int]) -> int:
        """
        Distribute minimum candies satisfying rating constraints.
        
        Strategy: Two-pass greedy algorithm
        1. Left→Right pass: Satisfy left neighbor constraints
        2. Right→Left pass: Satisfy right neighbor constraints (with max)
        
        Key insight: Use max() in second pass to preserve first pass results
        
        Edge cases:
        - All same ratings: Each gets 1 candy
        - Strictly increasing: [1, 2, 3, ..., n]
        - Strictly decreasing: [n, n-1, ..., 2, 1]
        - Peak (both neighbors lower): Gets max(left+1, right+1)
        
        Args:
            ratings: Array of child ratings
            
        Returns:
            Minimum total candies needed
        """
        n = len(ratings)
        
        # Every child gets at least 1 candy
        candies = [1] * n
        
        # Pass 1: Left to Right
        # Ensure each child has more candies than left neighbor if rating is higher
        for i in range(1, n):
            if ratings[i] > ratings[i-1]:
                candies[i] = candies[i-1] + 1
        
        # Pass 2: Right to Left
        # Ensure each child has more candies than right neighbor if rating is higher
        # Use max() to preserve pass 1 results (don't break left constraints)
        for i in range(n-2, -1, -1):
            if ratings[i] > ratings[i+1]:
                # Take max to ensure we don't violate the left constraint
                candies[i] = max(candies[i], candies[i+1] + 1)
        
        return sum(candies)
```

---

## Detailed Walkthrough

### Example 1: `ratings = [1, 0, 2]`

**Pass 1: Left → Right (Check left neighbor)**

| i | rating | Left neighbor | Comparison | Action | candies |
|---|--------|---------------|------------|--------|---------|
| 0 | 1 | None | - | Base case | [**1**, 1, 1] |
| 1 | 0 | 1 | 0 < 1 ✗ | No change | [1, **1**, 1] |
| 2 | 2 | 0 | 2 > 0 ✓ | candies[2] = 1+1 | [1, 1, **2**] |

After Pass 1: `candies = [1, 1, 2]`

**Pass 2: Right → Left (Check right neighbor)**

| i | rating | Right neighbor | Comparison | Current | Right+1 | Action | candies |
|---|--------|----------------|------------|---------|---------|--------|---------|
| 2 | 2 | None | - | 2 | - | No change | [1, 1, **2**] |
| 1 | 0 | 2 | 0 < 2 ✗ | 1 | - | No change | [1, **1**, 2] |
| 0 | 1 | 0 | 1 > 0 ✓ | 1 | 2 | max(1, 2) = 2 | [**2**, 1, 2] |

After Pass 2: `candies = [2, 1, 2]`

**Result:** `sum([2, 1, 2]) = 5` ✓

**Why it works:**
- Child 0: rating 1 > rating 0 (right), gets 2 candies
- Child 1: rating 0 lowest, gets 1 candy
- Child 2: rating 2 > rating 0 (left), gets 2 candies

---

### Example 2: `ratings = [1, 2, 2]`

**Pass 1: Left → Right**

| i | rating | ratings[i] > ratings[i-1] | Action | candies |
|---|--------|---------------------------|--------|---------|
| 0 | 1 | - | Base | [**1**, 1, 1] |
| 1 | 2 | 2 > 1 ✓ | candies[1] = 2 | [1, **2**, 1] |
| 2 | 2 | 2 > 2 ✗ | No change | [1, 2, **1**] |

After Pass 1: `candies = [1, 2, 1]`

**Pass 2: Right → Left**

| i | rating | ratings[i] > ratings[i+1] | Current | Action | candies |
|---|--------|---------------------------|---------|--------|---------|
| 2 | 2 | - | 1 | Base | [1, 2, **1**] |
| 1 | 2 | 2 > 2 ✗ | 2 | No change | [1, **2**, 1] |
| 0 | 1 | 1 > 2 ✗ | 1 | No change | [**1**, 2, 1] |

After Pass 2: `candies = [1, 2, 1]`

**Result:** `sum([1, 2, 1]) = 4` ✓

**Key observation:** Equal ratings don't require more candies!

---

### Example 3: `ratings = [1, 3, 2, 2, 1]` (Peak Pattern)

**Pass 1: Left → Right**

```
i=0: rating=1, base case → [1, 1, 1, 1, 1]
i=1: rating=3 > rating=1 ✓ → candies[1] = 1+1 = 2 → [1, 2, 1, 1, 1]
i=2: rating=2 > rating=3 ✗ → no change → [1, 2, 1, 1, 1]
i=3: rating=2 > rating=2 ✗ → no change → [1, 2, 1, 1, 1]
i=4: rating=1 > rating=2 ✗ → no change → [1, 2, 1, 1, 1]

After Pass 1: [1, 2, 1, 1, 1]
```

**Pass 2: Right → Left**

```
i=4: base case → [1, 2, 1, 1, 1]
i=3: rating=2 > rating=1 ✓ → max(1, 1+1) = 2 → [1, 2, 1, 2, 1]
i=2: rating=2 > rating=2 ✗ → no change → [1, 2, 1, 2, 1]
i=1: rating=3 > rating=2 ✓ → max(2, 1+1) = 2 → [1, 2, 1, 2, 1]
       (Already has 2 from pass 1, keep it)
i=0: rating=1 > rating=3 ✗ → no change → [1, 2, 1, 2, 1]

After Pass 2: [1, 2, 1, 2, 1]
```

**Result:** `sum([1, 2, 1, 2, 1]) = 7` ✓

**Analysis:**
- Index 1 (rating 3): Peak! Higher than both neighbors
  - Pass 1: Gets 2 (from left constraint)
  - Pass 2: Already has 2, which is enough for right constraint
- Index 3 (rating 2): Higher than right neighbor only
  - Pass 1: Gets 1 (no left constraint)
  - Pass 2: Gets 2 (from right constraint)

---

## Why This Algorithm Works

### Understanding the Two Passes

**Pass 1 establishes left-to-right order:**

```
Input: [1, 2, 4, 3]

After Pass 1: [1, 2, 3, 1]

Why?
  - Child 1: rating 2 > 1, so gets 1+1 = 2
  - Child 2: rating 4 > 2, so gets 2+1 = 3
  - Child 3: rating 3 < 4, so stays at 1

This satisfies: "If my left neighbor has lower rating, I have more candies"
```

**Pass 2 fixes right-to-left violations:**

```
After Pass 1: [1, 2, 3, 1]

Check child 2: rating 4 > rating 3
  - Child 2 has 3 candies, child 3 has 1
  - 3 > 1 ✓ Already satisfied!
  - Keep candies[2] = max(3, 1+1) = 3

Final: [1, 2, 3, 1]
```

**Why max() is Critical:**

```
Without max():

Input: [1, 3, 2]
Pass 1: [1, 2, 1]
Pass 2 without max:
  - i=0: rating 1 > rating 3? No
  - i=1: rating 3 > rating 2? Yes → candies[1] = 1+1 = 2
  
But wait! Pass 1 already gave child 1 TWO candies!
If we overwrite with 2, we're okay here, but consider:

Input: [1, 4, 3, 2]
Pass 1: [1, 2, 1, 1]
Pass 2 without max:
  - Child 1: 4 > 3, so candies[1] = 1+1 = 2 (wrong! Should be 4)

With max():
Pass 2: candies[1] = max(2, 1+1) = max(2, 2) = 2 ✓

The max() ensures we NEVER reduce candies established in pass 1.
```

### Why Not One Pass?

```
Try one pass on: [1, 3, 2]

i=0: candies[0] = 1
i=1: rating 3 > 1, candies[1] = 1+1 = 2
i=2: rating 2 < 3, candies[2] = ???

Problem: We need to know if child 1 should have MORE than 2 candies!
  - Child 1 has rating 3 > rating 2 (right neighbor)
  - So child 1 needs candies[2] + 1
  - But we don't know candies[2] yet!

Circular dependency! Can't solve in one pass forward.
```

---

## Edge Cases

### 1. All Same Ratings
```python
ratings = [2, 2, 2, 2]
# Output: 4
# candies = [1, 1, 1, 1]
# Equal ratings don't require more candies
```

### 2. Strictly Increasing
```python
ratings = [1, 2, 3, 4, 5]
# Output: 15 (1+2+3+4+5)
# candies = [1, 2, 3, 4, 5]
# Each child needs one more than the previous
```

### 3. Strictly Decreasing
```python
ratings = [5, 4, 3, 2, 1]
# Pass 1: [1, 1, 1, 1, 1] (no left constraints satisfied)
# Pass 2: [5, 4, 3, 2, 1] (all right constraints kick in)
# Output: 15
```

### 4. Single Child
```python
ratings = [1]
# Output: 1
# Only one child, gets minimum 1 candy
```

### 5. Two Children Same Rating
```python
ratings = [1, 1]
# Output: 2
# candies = [1, 1]
# Equal ratings, each gets 1
```

### 6. Peak in Middle
```python
ratings = [1, 5, 1]
# Pass 1: [1, 2, 1] (only left constraint)
# Pass 2: [1, 2, 1] (2 already > 1, so max(2, 2) = 2)
# Output: 4
```

### 7. Valley in Middle
```python
ratings = [5, 1, 5]
# Pass 1: [1, 1, 2] (right child higher than middle)
# Pass 2: [2, 1, 2] (left child higher than middle)
# Output: 5
```

### 8. Plateau Pattern
```python
ratings = [1, 2, 2, 3]
# Pass 1: [1, 2, 1, 2] (2=2 doesn't increase)
# Pass 2: [1, 2, 1, 2] (no right constraints)
# Output: 6
```

---

## Common Mistakes

### ❌ Mistake 1: Using Only One Pass
```python
# Wrong! Doesn't handle peaks (higher than both neighbors)
def candy(ratings):
    candies = [1] * len(ratings)
    for i in range(1, len(ratings)):
        if ratings[i] > ratings[i-1]:
            candies[i] = candies[i-1] + 1
    return sum(candies)

# Fails on: [1, 3, 2]
# Returns: [1, 2, 1] = 4
# But child 1 should have more candies than child 2!
# Correct: [1, 2, 1] (works by coincidence here, but try [1,4,3,2])
```

### ❌ Mistake 2: Not Using max() in Second Pass
```python
# Wrong! Overwrites first pass results
for i in range(n-2, -1, -1):
    if ratings[i] > ratings[i+1]:
        candies[i] = candies[i+1] + 1  # Missing max()!

# Fails on: [1, 4, 3, 2]
# Pass 1: [1, 2, 1, 1]
# Pass 2: candies[1] = 1+1 = 2 (should be max(2, 2) = 2, works)
#         candies[0] = 2+1 = 3 (should be max(1, 3) = 3, works)
# Actually works here, but try: [1, 5, 4, 3, 2]
# Pass 1: [1, 2, 1, 1, 1]
# Pass 2 wrong: candies[1] = 1+1 = 2 (loses the 2 from pass 1)
```

### ❌ Mistake 3: Comparing Wrong Neighbors
```python
# Wrong! Comparing with both neighbors in one pass
for i in range(1, n-1):
    if ratings[i] > ratings[i-1] and ratings[i] > ratings[i+1]:
        candies[i] = max(candies[i-1], candies[i+1]) + 1

# Problems:
# 1. Doesn't handle edges (i=0, i=n-1)
# 2. Only updates peaks, misses single-side increases
# 3. Order matters! Can't do it simultaneously
```

### ❌ Mistake 4: Incrementing Instead of Assigning
```python
# Wrong! Cumulative increments
for i in range(1, n):
    if ratings[i] > ratings[i-1]:
        candies[i] += 1  # Should be candies[i] = candies[i-1] + 1

# Fails on: [1, 2, 3]
# Result: [1, 2, 3] (happens to work!)
# Fails on: [1, 3, 5]
# Result: [1, 2, 3] (should be [1, 2, 3], works!)
# Actually this might work by coincidence, but conceptually wrong
# Try: [1, 10, 100] (all should be [1, 2, 3])
# With +=: [1, 2, 3] (works, but wrong logic)
```

### ❌ Mistake 5: Handling Equal Ratings Wrong
```python
# Wrong! Giving more candies for equal ratings
for i in range(1, n):
    if ratings[i] >= ratings[i-1]:  # Should be just >
        candies[i] = candies[i-1] + 1

# Fails on: [1, 1, 1]
# Returns: [1, 2, 3] = 6
# Correct: [1, 1, 1] = 3
```

### ❌ Mistake 6: Wrong Range in Second Pass
```python
# Wrong! Starting from n-1 instead of n-2
for i in range(n-1, -1, -1):  # Should be range(n-2, -1, -1)
    if ratings[i] > ratings[i+1]:  # IndexError when i = n-1!
        candies[i] = max(candies[i], candies[i+1] + 1)
```

---

## Interview Insights

**Question Recognition:**
- "Minimum distribution" + "neighbor constraints" → Greedy with two passes
- "Higher value gets more" + "neighbors" → Local comparison problem

**Key Points to Mention:**

1. **Two-pass approach:**
   - "The key insight is that left and right constraints are independent. I handle them separately in two passes, then merge with max()."

2. **Why max() in second pass:**
   - "The max() is critical because it ensures I don't violate constraints established in the first pass. I only increase candies if the right constraint demands it."

3. **Edge case - equal ratings:**
   - "Equal ratings don't require more candies. The constraint is only for strictly greater ratings."

4. **Peak handling:**
   - "Peaks (higher than both neighbors) are automatically handled because both passes contribute, and max() preserves the higher value."

5. **One pass doesn't work:**
   - "I can't do this in one pass because a child might need more candies than both neighbors, creating a circular dependency."

**Follow-up Questions:**

1. **Q:** "Can you optimize space to O(1)?"
   - **A:** "Technically yes with a complex one-pass algorithm that tracks peak counts and valley lengths, but it's much more complex and error-prone. The O(n) space solution is clearer and still optimal for time."

2. **Q:** "What if we want to minimize the maximum candies given to any child?"
   - **A:** "That's a different problem. We'd need to use a different approach, possibly BFS from children with lowest ratings."

3. **Q:** "How would you handle if children could be in a circle (last child is neighbor of first)?"
   - **A:** "That's much harder! Linear arrangement lets us use two passes. A circle might require finding the minimum rating point and breaking the circle there."

4. **Q:** "What if there are multiple constraints (like ratings from multiple attributes)?"
   - **A:** "We'd need a pass for each constraint type, still using max() to merge results."

5. **Q:** "Can we use sorting?"
   - **A:** "No, because sorting loses the neighbor information. The problem fundamentally requires maintaining the original order."

**Complexity Analysis:**
- Time: O(n) - Two passes through the array
- Space: O(n) - Candies array (required for output, can't optimize)

---

## Related Problems

1. **Jump Game II (LeetCode 45)** - Greedy with two-pass potential
2. **Trapping Rain Water (LeetCode 42)** - Two-pass approach (left max, right max)
3. **Best Time to Buy and Sell Stock II (LeetCode 122)** - Greedy local decisions
4. **Gas Station (LeetCode 134)** - Greedy with two-pass verification
5. **Task Scheduler (LeetCode 621)** - Greedy with constraints

---

## Variations

### Variation 1: Return Candy Distribution (Not Just Sum)
```python
def candyDistribution(self, ratings):
    """Return the actual candy distribution."""
    n = len(ratings)
    candies = [1] * n
    
    # Left to right
    for i in range(1, n):
        if ratings[i] > ratings[i-1]:
            candies[i] = candies[i-1] + 1
    
    # Right to left
    for i in range(n-2, -1, -1):
        if ratings[i] > ratings[i+1]:
            candies[i] = max(candies[i], candies[i+1] + 1)
    
    return candies  # Return array instead of sum
```

### Variation 2: Circular Arrangement (Advanced)
```python
def candyCircular(self, ratings):
    """Handle children in a circle (last is neighbor of first)."""
    n = len(ratings)
    if n == 1:
        return 1
    
    # Find minimum rating to break circle
    min_idx = ratings.index(min(ratings))
    
    # Rotate array so minimum is at start
    rotated = ratings[min_idx:] + ratings[:min_idx]
    
    # Solve linear problem
    candies = [1] * n
    for i in range(1, n):
        if rotated[i] > rotated[i-1]:
            candies[i] = candies[i-1] + 1
    
    for i in range(n-2, -1, -1):
        if rotated[i] > rotated[i+1]:
            candies[i] = max(candies[i], candies[i+1] + 1)
    
    # Rotate result back
    result = candies[n-min_idx:] + candies[:n-min_idx]
    return sum(result)
```

### Variation 3: With Minimum Candy Constraint
```python
def candyMinimum(self, ratings, minimum):
    """Each child must have at least 'minimum' candies."""
    n = len(ratings)
    candies = [minimum] * n  # Change base case
    
    for i in range(1, n):
        if ratings[i] > ratings[i-1]:
            candies[i] = candies[i-1] + 1
    
    for i in range(n-2, -1, -1):
        if ratings[i] > ratings[i+1]:
            candies[i] = max(candies[i], candies[i+1] + 1)
    
    return sum(candies)
```

### Variation 4: Space-Optimized One-Pass (Advanced)
```python
def candyOptimized(self, ratings):
    """
    O(1) space using peak/valley tracking.
    More complex, but demonstrates advanced greedy.
    """
    n = len(ratings)
    if n <= 1:
        return n
    
    total = 1
    up = 0  # Length of increasing sequence
    down = 0  # Length of decreasing sequence
    peak = 0  # Candies at peak
    
    for i in range(1, n):
        if ratings[i] > ratings[i-1]:
            # Going up
            down = 0
            up += 1
            peak = up + 1
            total += peak
        elif ratings[i] < ratings[i-1]:
            # Going down
            up = 0
            down += 1
            # Add down, and adjust peak if necessary
            total += down
            if down >= peak:
                total += 1
        else:
            # Plateau (equal ratings)
            up = down = peak = 0
            total += 1
    
    return total
```

### Variation 5: With Candy Limit Per Child
```python
def candyWithLimit(self, ratings, maxCandy):
    """
    Each child can receive at most maxCandy candies.
    Returns -1 if impossible.
    """
    n = len(ratings)
    candies = [1] * n
    
    for i in range(1, n):
        if ratings[i] > ratings[i-1]:
            candies[i] = candies[i-1] + 1
            if candies[i] > maxCandy:
                return -1  # Impossible
    
    for i in range(n-2, -1, -1):
        if ratings[i] > ratings[i+1]:
            needed = candies[i+1] + 1
            if needed > maxCandy:
                return -1  # Impossible
            candies[i] = max(candies[i], needed)
    
    return sum(candies)
```

---

## Summary

**Key Takeaways:**

1. **Two-Pass Greedy:**
   - Pass 1 (Left→Right): Satisfy left neighbor constraints
   - Pass 2 (Right→Left): Satisfy right neighbor constraints
   - max() merges both constraints without violating either

2. **Why Two Passes:**
   - Peaks (higher than both neighbors) create circular dependencies
   - Can't determine final value in one pass
   - Two passes decouple left and right constraints

3. **Critical Details:**
   - Initialize with `[1] * n` (everyone gets at least 1)
   - Only increase for strictly greater ratings (`>`, not `>=`)
   - Use `max(candies[i], candies[i+1] + 1)` in second pass
   - Range for pass 2: `range(n-2, -1, -1)` (avoid index out of bounds)

4. **Edge Cases:**
   - Equal ratings: No additional candies required
   - Strictly increasing: [1, 2, 3, ..., n]
   - Strictly decreasing: [n, n-1, ..., 2, 1]
   - Single child: Returns 1

5. **Why max() Matters:**
   - Preserves constraints from pass 1
   - Only updates if right constraint demands more
   - Ensures both left AND right constraints are satisfied

**Mental Model:**
```
Think of it as two independent "flows":
1. Left flow: Each person looks left and takes what they need
2. Right flow: Each person looks right and takes what they need
3. Final value: Take the maximum of both flows

This ensures both constraints are met without recalculation.
```

**Template:**
```python
def candy(self, ratings):
    n = len(ratings)
    candies = [1] * n
    
    # Pass 1: Left to Right
    for i in range(1, n):
        if ratings[i] > ratings[i-1]:
            candies[i] = candies[i-1] + 1
    
    # Pass 2: Right to Left (with max)
    for i in range(n-2, -1, -1):
        if ratings[i] > ratings[i+1]:
            candies[i] = max(candies[i], candies[i+1] + 1)
    
    return sum(candies)
```

This is a classic example of how sometimes the greedy approach requires multiple passes to handle bidirectional constraints!
