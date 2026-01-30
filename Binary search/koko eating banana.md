# Koko Eating Bananas

**LeetCode 875** | **Difficulty:** Medium | **Category:** Binary Search, Array

---

## Problem Statement

Koko loves to eat bananas. There are `n` piles of bananas, the `i`th pile has `piles[i]` bananas. The guards have gone and will come back in `h` hours.

Koko can decide her bananas-per-hour eating speed of `k`. Each hour, she chooses some pile of bananas and eats `k` bananas from that pile. If the pile has less than `k` bananas, she eats all of them instead and will not eat any more bananas during this hour.

Koko likes to eat slowly but still wants to finish eating all the bananas before the guards return.

Return the **minimum** integer `k` such that she can eat all the bananas within `h` hours.

---

## Examples

### Example 1:
```
Input: piles = [3,6,7,11], h = 8
Output: 4

Explanation:
With k=4:
- Pile 1 (3 bananas): ceil(3/4) = 1 hour
- Pile 2 (6 bananas): ceil(6/4) = 2 hours
- Pile 3 (7 bananas): ceil(7/4) = 2 hours
- Pile 4 (11 bananas): ceil(11/4) = 3 hours
Total: 1 + 2 + 2 + 3 = 8 hours ✓
```

### Example 2:
```
Input: piles = [30,11,23,4,20], h = 5
Output: 30

Explanation:
With k=30:
- Each pile takes ceil(pile/30) hours
- Since h=5 (number of piles), she needs to finish each pile in 1 hour
- Maximum pile is 30, so k must be at least 30
```

### Example 3:
```
Input: piles = [30,11,23,4,20], h = 6
Output: 23

Explanation:
With k=23:
- Pile 30: ceil(30/23) = 2 hours
- Pile 11: ceil(11/23) = 1 hour
- Pile 23: ceil(23/23) = 1 hour
- Pile 4: ceil(4/23) = 1 hour
- Pile 20: ceil(20/23) = 1 hour
Total: 2 + 1 + 1 + 1 + 1 = 6 hours ✓
```

### Example 4:
```
Input: piles = [3,6,7,11], h = 20
Output: 1

Explanation:
With k=1:
- Total bananas = 3 + 6 + 7 + 11 = 27
- Takes 27 hours, which is less than 20
Wait, let me recalculate...
- Pile 3: 3 hours
- Pile 6: 6 hours
- Pile 7: 7 hours
- Pile 11: 11 hours
Total: 27 hours, which is > 20

Actually with k=2:
- Pile 3: ceil(3/2) = 2 hours
- Pile 6: ceil(6/2) = 3 hours
- Pile 7: ceil(7/2) = 4 hours
- Pile 11: ceil(11/2) = 6 hours
Total: 15 hours ✓
```

---

## Strategy

### Binary Search on Answer Space

**Key Insight:** We're not searching in the array—we're searching for the **minimum eating speed k**.

**Why Binary Search?**

```
Eating speed k can range from 1 to max(piles)

Observation:
- If k works (finishes in ≤ h hours), then k+1 also works
- If k doesn't work, then k-1 also doesn't work
- This creates a monotonic property: [doesn't work... | works...]

Example: piles = [3,6,7,11], h = 8
k=1: 27 hours ✗
k=2: 14 hours ✓
k=3: 10 hours ✓
k=4: 8 hours ✓
k=5: 7 hours ✓
...
k=11: 4 hours ✓

We want the minimum k that works → Binary search!
```

**Search Space:**

```
Minimum possible k: 1 (slowest speed)
Maximum possible k: max(piles) (fastest needed)

Why max(piles)?
- If k = max(piles), largest pile takes 1 hour
- Any k > max(piles) is unnecessarily fast
- We want minimum, so search up to max(piles)
```

**Algorithm:**

1. **Set search range:** `left = 1, right = max(piles)`
2. **For each mid (potential k):**
   - Calculate total hours needed: `sum(ceil(pile / k) for pile in piles)`
   - If `total_hours <= h`: This k works, try smaller (right = k - 1)
   - If `total_hours > h`: This k too slow, need faster (left = k + 1)
3. **Track minimum valid k** and return it

**Time Calculation:**

```
For each pile p and eating speed k:
Hours needed = ceil(p / k)

Examples:
- pile=7, k=4: ceil(7/4) = ceil(1.75) = 2 hours
- pile=11, k=4: ceil(11/4) = ceil(2.75) = 3 hours
- pile=6, k=3: ceil(6/3) = ceil(2.0) = 2 hours

Math trick: ceil(a/b) = (a + b - 1) // b
```

**Complexity:**
- **Time:** $O(n \log m)$ where n = number of piles, m = max(piles)
  - Binary search: O(log m) iterations
  - Each iteration: O(n) to calculate total hours
- **Space:** $O(1)$ - Only using variables

---

## Solution

```python
import math

class Solution:
    def minEatingSpeed(self, piles: List[int], h: int) -> int:
        """
        Find minimum eating speed k to finish all bananas in h hours.
        
        Strategy: Binary search on the answer space [1, max(piles)].
        For each potential speed k, calculate if it's possible to finish
        in h hours. If yes, try slower; if no, must go faster.
        
        Key insight: If k works, all k' > k also work (monotonic property).
        We want the minimum k that works.
        
        Args:
            piles: Array of banana pile sizes
            h: Total hours available
            
        Returns:
            Minimum eating speed k
        """
        # Search space: k ranges from 1 to max(piles)
        left, right = 1, max(piles)
        res = right  # Worst case: eat at max speed
        
        while left <= right:
            # Try this eating speed
            k = (left + right) // 2
            
            # Calculate total hours needed at speed k
            total_time = 0
            for p in piles:
                # Each pile takes ceil(p/k) hours
                total_time += math.ceil(p / k)
            
            # Check if this speed works
            if total_time <= h:
                # k works! Try to find smaller k
                res = k
                right = k - 1
            else:
                # k too slow, need faster speed
                left = k + 1
        
        return res
```

### Alternative: Without math.ceil

```python
class Solution:
    def minEatingSpeed(self, piles: List[int], h: int) -> int:
        """Using ceiling division trick: ceil(a/b) = (a + b - 1) // b"""
        left, right = 1, max(piles)
        res = right
        
        while left <= right:
            k = (left + right) // 2
            
            total_time = 0
            for p in piles:
                # Ceiling division without math.ceil
                total_time += (p + k - 1) // k
            
            if total_time <= h:
                res = k
                right = k - 1
            else:
                left = k + 1
        
        return res
```

---

## Detailed Walkthrough

### Example 1: `piles = [3,6,7,11], h = 8`

**Setup:**
- left = 1, right = 11
- Need to find minimum k

**Step-by-Step Execution:**

| Step | Left | Right | k | Pile 3 | Pile 6 | Pile 7 | Pile 11 | Total | Check | Action | Update res |
|------|------|-------|---|--------|--------|--------|---------|-------|-------|--------|------------|
| 0 | 1 | 11 | 6 | 1 | 1 | 2 | 2 | 6 | 6≤8 ✓ | right=5 | res=6 |
| 1 | 1 | 5 | 3 | 1 | 2 | 3 | 4 | 10 | 10≤8 ✗ | left=4 | - |
| 2 | 4 | 5 | 4 | 1 | 2 | 2 | 3 | 8 | 8≤8 ✓ | right=3 | res=4 |
| 3 | 4 | 3 | - | - | - | - | - | - | left>right | Stop | - |

**Detailed Calculation for Step 2 (k=4):**

```
Pile 1 (3 bananas): ceil(3/4) = ceil(0.75) = 1 hour
Pile 2 (6 bananas): ceil(6/4) = ceil(1.5) = 2 hours
Pile 3 (7 bananas): ceil(7/4) = ceil(1.75) = 2 hours
Pile 4 (11 bananas): ceil(11/4) = ceil(2.75) = 3 hours

Total: 1 + 2 + 2 + 3 = 8 hours ≤ 8 ✓
```

**Final Answer:** `4` ✓

---

### Example 2: `piles = [30,11,23,4,20], h = 5`

**Setup:**
- left = 1, right = 30
- h = 5 (exactly number of piles!)

| Step | Left | Right | k | Hours | Check | Action | res |
|------|------|-------|---|-------|-------|--------|-----|
| 0 | 1 | 30 | 15 | 2+1+2+1+2=8 | 8≤5 ✗ | left=16 | 30 |
| 1 | 16 | 30 | 23 | 2+1+1+1+1=6 | 6≤5 ✗ | left=24 | 30 |
| 2 | 24 | 30 | 27 | 2+1+1+1+1=6 | 6≤5 ✗ | left=28 | 30 |
| 3 | 28 | 30 | 29 | 2+1+1+1+1=6 | 6≤5 ✗ | left=30 | 30 |
| 4 | 30 | 30 | 30 | 1+1+1+1+1=5 | 5≤5 ✓ | right=29 | 30 |
| 5 | 30 | 29 | - | - | left>right | Stop | - |

**Calculation for k=30:**

```
Pile 30: ceil(30/30) = 1 hour
Pile 11: ceil(11/30) = 1 hour
Pile 23: ceil(23/30) = 1 hour
Pile 4: ceil(4/30) = 1 hour
Pile 20: ceil(20/30) = 1 hour

Total: 5 hours = h ✓

When h equals number of piles, k must be ≥ max(piles)
```

**Final Answer:** `30` ✓

---

### Example 3: `piles = [3,6,7,11], h = 20` (Lots of time)

| Step | Left | Right | k | Total Hours | Check | Action | res |
|------|------|-------|---|-------------|-------|--------|-----|
| 0 | 1 | 11 | 6 | 1+1+2+2=6 | 6≤20 ✓ | right=5 | 6 |
| 1 | 1 | 5 | 3 | 1+2+3+4=10 | 10≤20 ✓ | right=2 | 3 |
| 2 | 1 | 2 | 1 | 3+6+7+11=27 | 27≤20 ✗ | left=2 | 3 |
| 3 | 2 | 2 | 2 | 2+3+4+6=15 | 15≤20 ✓ | right=1 | 2 |
| 4 | 2 | 1 | - | - | left>right | Stop | - |

**Calculation for k=2:**

```
Pile 3: ceil(3/2) = 2 hours
Pile 6: ceil(6/2) = 3 hours
Pile 7: ceil(7/2) = 4 hours
Pile 11: ceil(11/2) = 6 hours

Total: 15 hours ≤ 20 ✓
```

**Final Answer:** `2` ✓

---

## Why Binary Search Works

### The Monotonic Property

**Observation:** If k works, then any k' > k also works.

```
Proof:
If k can finish all piles in ≤ h hours,
then k' > k will finish even faster (≤ h hours).

Example: piles = [3,6,7,11], h = 8

k=4: 8 hours ✓ works
k=5: 7 hours ✓ still works (faster)
k=6: 6 hours ✓ still works (even faster)

This creates a sorted property:
[1:✗, 2:✗, 3:✗, 4:✓, 5:✓, 6:✓, ..., 11:✓]
        doesn't work | works

We want the first ✓ → Binary search!
```

**Visual Representation:**

```
Speed k:     1   2   3   4   5   6   7   8   9  10  11
Hours:      27  14  10   8   7   6   5   5   4   4   4
Works?      ✗   ✗   ✗   ✓   ✓   ✓   ✓   ✓   ✓   ✓   ✓
                        ↑
                    First True
                    Answer = 4
```

---

## Edge Cases

### 1. Single Pile
```python
piles = [30], h = 10
# Output: 3
# ceil(30/3) = 10 hours
```

### 2. h Equals Number of Piles
```python
piles = [3,6,7,11], h = 4
# Output: 11 (max pile)
# Must finish each pile in 1 hour
# k = max(piles)
```

### 3. Very Large h
```python
piles = [3,6,7,11], h = 100
# Output: 1
# Can eat 1 banana per hour
# Total: 3+6+7+11 = 27 hours < 100
```

### 4. All Piles Same Size
```python
piles = [5,5,5,5], h = 10
# Output: 2
# With k=2: each pile takes ceil(5/2) = 3 hours
# Total: 3*4 = 12 hours > 10 ✗
# With k=3: each pile takes ceil(5/3) = 2 hours
# Total: 2*4 = 8 hours ≤ 10 ✓
```

### 5. Single Banana Piles
```python
piles = [1,1,1,1], h = 4
# Output: 1
# Each pile takes 1 hour
# Total: 4 hours
```

### 6. Large Max Pile, Small Other Piles
```python
piles = [1000000000], h = 2
# Output: 500000000
# ceil(1000000000 / 500000000) = 2 hours
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Search Range
```python
# Wrong! Right should be max(piles), not len(piles)
left, right = 1, len(piles)
```

**Fix:**
```python
left, right = 1, max(piles)
```

**Why?** k is eating speed (bananas/hour), not related to number of piles.

### ❌ Mistake 2: Integer Division Instead of Ceiling
```python
# Wrong! Will underestimate hours
total_time += p // k
```

**Fix:**
```python
total_time += math.ceil(p / k)
# Or: total_time += (p + k - 1) // k
```

**Why?** If pile has 7 bananas and k=4, it takes 2 hours, not 1.

### ❌ Mistake 3: Not Storing Result
```python
# Wrong! What if we never enter the if block?
while left <= right:
    # ...
    if total_time <= h:
        right = k - 1
    else:
        left = k + 1

return left  # or right? Confusing!
```

**Fix:**
```python
res = right  # Initialize
while left <= right:
    # ...
    if total_time <= h:
        res = k  # Store valid answer
        right = k - 1
    else:
        left = k + 1

return res
```

### ❌ Mistake 4: Wrong Comparison
```python
# Wrong! Should be <=, not ==
if total_time == h:
    return k
```

**Fix:**
```python
if total_time <= h:
    res = k
    right = k - 1
```

**Why?** We want to finish within h hours, not exactly h hours.

### ❌ Mistake 5: Returning left/right Instead of res
```python
# Wrong! left and right are search pointers
return left  # or return right
```

**Fix:**
```python
return res  # Return stored minimum valid k
```

---

## Interview Insights

**Question Recognition:**
- "Minimum" + "speed/rate/capacity" + array of values → Binary search on answer
- Not searching IN array, searching FOR answer

**Key Points to Mention:**

1. **Binary search on answer space:**
   - "We're not searching the array. We're searching for the minimum eating speed between 1 and max(piles)."

2. **The monotonic property:**
   - "If speed k works, any faster speed also works. This creates a monotonic property perfect for binary search."

3. **Ceiling division:**
   - "Each pile takes ceil(pile/k) hours because Koko finishes the pile in one sitting."

4. **Why max(piles) is the upper bound:**
   - "The maximum speed needed is max(piles) because that makes even the largest pile take exactly 1 hour."

**Follow-up Questions:**

1. **Q:** "What if Koko can switch between piles within an hour?"
   - **A:** Then we'd sum all bananas and divide by h: `ceil(sum(piles) / h)`. Much simpler!

2. **Q:** "Can you optimize the time calculation?"
   - **A:** Current O(n) per iteration is necessary. We need to check all piles each time.

3. **Q:** "What if we want maximum k instead of minimum?"
   - **A:** Just change condition: if works, go left=k+1 to find larger; store on else branch.

4. **Q:** "How does this relate to other problems?"
   - **A:** Classic "binary search on answer" pattern. Similar to capacity to ship packages, split array largest sum, etc.

**Complexity Analysis:**
- Time: O(n log m) where n = len(piles), m = max(piles)
- Space: O(1) - Only using variables

---

## Related Problems (Binary Search on Answer)

1. **Capacity To Ship Packages Within D Days (LeetCode 1011)** - Find minimum ship capacity
2. **Split Array Largest Sum (LeetCode 410)** - Minimize maximum subarray sum
3. **Minimum Speed to Arrive on Time (LeetCode 1870)** - Similar to Koko
4. **Magnetic Force Between Two Balls (LeetCode 1552)** - Maximize minimum distance
5. **Minimize Max Distance to Gas Station (LeetCode 774)** - Binary search on distance
6. **Find K-th Smallest Pair Distance (LeetCode 719)** - Binary search + counting

---

## Problem Pattern: Binary Search on Answer

### When to Use This Pattern:

1. **Question asks for minimum/maximum** of something
2. **There's a feasibility check** (can we do X with value k?)
3. **Monotonic property**: If k works, k+1 (or k-1) also works
4. **Search space is continuous** (1 to max, or some range)

### Template:

```python
def binary_search_on_answer(arr, target):
    """Generic template for binary search on answer."""
    # Define search space
    left, right = min_possible, max_possible
    result = right  # or left, depending on problem
    
    while left <= right:
        mid = (left + right) // 2
        
        # Feasibility check: Can we achieve target with mid?
        if is_feasible(arr, mid, target):
            result = mid
            # Look for better answer (min: right=mid-1, max: left=mid+1)
            right = mid - 1  # For minimum
        else:
            # mid doesn't work, adjust search space
            left = mid + 1  # For minimum
    
    return result

def is_feasible(arr, value, target):
    """Check if value satisfies the problem constraints."""
    # Problem-specific logic
    pass
```

### Examples:

**Koko Eating Bananas:**
- Minimize: eating speed k
- Feasibility: Can finish in ≤ h hours?
- Range: [1, max(piles)]

**Ship Packages:**
- Minimize: ship capacity k
- Feasibility: Can ship all in ≤ D days?
- Range: [max(weights), sum(weights)]

**Split Array:**
- Minimize: largest subarray sum k
- Feasibility: Can split into ≤ m subarrays with max sum ≤ k?
- Range: [max(nums), sum(nums)]

---

## Variations

### Variation 1: Return Hours Taken with Optimal k
```python
def minEatingSpeedWithHours(piles, h):
    """Return both minimum k and hours taken."""
    left, right = 1, max(piles)
    res = right
    
    while left <= right:
        k = (left + right) // 2
        
        total_time = sum(math.ceil(p / k) for p in piles)
        
        if total_time <= h:
            res = k
            right = k - 1
        else:
            left = k + 1
    
    # Calculate actual hours with optimal k
    actual_hours = sum(math.ceil(p / res) for p in piles)
    return res, actual_hours
```

### Variation 2: Maximum Eating Speed with Constraint
```python
def maxEatingSpeed(piles, h):
    """Find maximum k that takes at least h hours."""
    left, right = 1, max(piles)
    res = 1
    
    while left <= right:
        k = (left + right) // 2
        
        total_time = sum(math.ceil(p / k) for p in piles)
        
        if total_time >= h:
            res = k
            left = k + 1  # Try larger k
        else:
            right = k - 1
    
    return res
```

### Variation 3: Check if Specific k Works
```python
def canFinishInTime(piles, k, h):
    """Check if eating speed k can finish all piles in h hours."""
    total_time = sum(math.ceil(p / k) for p in piles)
    return total_time <= h
```

---

## Summary

**Key Takeaways:**

1. **Binary Search on Answer:**
   - Not searching IN array, searching FOR answer
   - Range: [1, max(piles)]
   - Find minimum k that satisfies constraint

2. **Monotonic Property:**
   - If k works, any k' > k also works
   - Creates sorted structure: [✗✗✗✓✓✓]
   - Perfect for binary search

3. **Time Calculation:**
   - Each pile: `ceil(pile / k)` hours
   - Total: sum of all pile times
   - Must use ceiling, not floor!

4. **Why max(piles) is Upper Bound:**
   - At k = max(piles), largest pile takes 1 hour
   - Any larger k is unnecessarily fast
   - We want minimum, so no need to search higher

5. **Store Result Pattern:**
   - Initialize `res = right`
   - Update `res = k` when k works
   - Try to find smaller k: `right = k - 1`
   - Return stored result

**Template:**
```python
left, right = 1, max(piles)
res = right

while left <= right:
    k = (left + right) // 2
    
    # Calculate total hours at speed k
    total_time = sum(math.ceil(p / k) for p in piles)
    
    if total_time <= h:
        res = k  # k works, store it
        right = k - 1  # Try smaller k
    else:
        left = k + 1  # k too slow, need faster
        
return res
```

**Mental Model:**
- Imagine a speed dial from 1 to max(piles)
- Slower speeds take more time, faster take less
- There's a threshold speed where Koko finishes just in time
- Binary search efficiently finds this threshold
- This pattern appears in many "minimize/maximize with constraint" problems!
