# Capacity To Ship Packages Within D Days

**LeetCode 1011** | **Difficulty:** Medium | **Category:** Binary Search, Array

---

## Problem Statement

A conveyor belt has packages that must be shipped from one port to another within `days` days.

The `i`th package on the conveyor belt has a weight of `weights[i]`. Each day, we load the ship with packages on the conveyor belt (in the order given by `weights`). We may not load more weight than the maximum weight capacity of the ship.

Return the **least weight capacity** of the ship that will result in all the packages on the conveyor belt being shipped within `days` days.

---

## Examples

### Example 1:
```
Input: weights = [1,2,3,4,5,6,7,8,9,10], days = 5
Output: 15

Explanation:
A ship capacity of 15 is the minimum to ship all packages in 5 days:
Day 1: 1, 2, 3, 4, 5 (total: 15)
Day 2: 6, 7 (total: 13)
Day 3: 8 (total: 8)
Day 4: 9 (total: 9)
Day 5: 10 (total: 10)
```

### Example 2:
```
Input: weights = [3,2,2,4,1,4], days = 3
Output: 6

Explanation:
A ship capacity of 6 is the minimum:
Day 1: 3, 2 (total: 5)
Day 2: 2, 4 (total: 6)
Day 3: 1, 4 (total: 5)
```

### Example 3:
```
Input: weights = [1,2,3,1,1], days = 4
Output: 3

Explanation:
Day 1: 1, 2 (total: 3)
Day 2: 3 (total: 3)
Day 3: 1, 1 (total: 2)
Day 4: (empty, but we have 4 days)

Actually, let me recalculate - we use 3 days only.
With capacity 3:
Day 1: 1, 2 (total: 3)
Day 2: 3 (total: 3)
Day 3: 1, 1 (total: 2)
Total: 3 days ≤ 4 days ✓
```

### Example 4:
```
Input: weights = [10,50,100,100,50,100,100,50], days = 5
Output: 160

Explanation:
With capacity 160:
Day 1: 10, 50, 100 (total: 160)
Day 2: 100, 50 (total: 150)
Day 3: 100 (total: 100)
Day 4: 100, 50 (total: 150)
Day 5: (empty if needed)
```

---

## Strategy

### Binary Search on Answer Space

**Key Insight:** We're searching for the **minimum ship capacity**, not searching in the array.

**Why Binary Search?**

```
Ship capacity can range from max(weights) to sum(weights)

Observation:
- If capacity C works (ships in ≤ days), then C+1 also works
- If capacity C doesn't work, then C-1 also doesn't work
- Monotonic property: [doesn't work... | works...]

Example: weights = [1,2,3,4,5,6,7,8,9,10], days = 5

Capacity 10: Need 10 days ✗
Capacity 15: Need 5 days ✓
Capacity 20: Need 4 days ✓
Capacity 30: Need 3 days ✓
...
Capacity 55: Need 1 day ✓

We want minimum capacity that works → Binary search!
```

**Search Space:**

```
Minimum possible capacity: max(weights)
- Why? Must be able to carry the heaviest single package
- If capacity < max(weights), that package can never ship!

Maximum possible capacity: sum(weights)
- Why? Can ship everything in 1 day
- Any capacity ≥ sum(weights) is unnecessarily large

Example: weights = [1,2,3,4,5]
- Min capacity: 5 (heaviest package)
- Max capacity: 15 (all packages)
```

**Algorithm:**

1. **Set search range:** `left = max(weights), right = sum(weights)`
2. **For each mid (potential capacity):**
   - Simulate shipping: Load packages sequentially until capacity reached
   - Count days needed with this capacity
   - If `days_needed <= days`: This capacity works, try smaller (right = mid - 1)
   - If `days_needed > days`: Capacity too small, need larger (left = mid + 1)
3. **Track minimum valid capacity** and return it

**Simulation Logic:**

```python
def canShip(capacity):
    ships = 1  # Start with first day
    curr_cap = capacity  # Current day's remaining capacity
    
    for weight in weights:
        if curr_cap - weight < 0:
            # Can't fit on current day, start new day
            ships += 1
            curr_cap = capacity
        
        curr_cap -= weight  # Load this package
    
    return ships <= days
```

**Complexity:**
- **Time:** $O(n \log S)$ where n = len(weights), S = sum(weights)
  - Binary search: O(log S) iterations
  - Each iteration: O(n) to simulate shipping
- **Space:** $O(1)$ - Only using variables

---

## Solution

```python
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        """
        Find minimum ship capacity to ship all packages within days.
        
        Strategy: Binary search on answer space [max(weights), sum(weights)].
        For each capacity, simulate shipping to check if possible in ≤ days.
        
        Key insights:
        1. Capacity must be ≥ max(weights) to carry heaviest package
        2. Capacity ≤ sum(weights) because that ships everything in 1 day
        3. If capacity C works, any C' > C also works (monotonic)
        
        Args:
            weights: Array of package weights (in order)
            days: Maximum days allowed for shipping
            
        Returns:
            Minimum ship capacity needed
        """
        # Search space: capacity from max(weights) to sum(weights)
        left, right = max(weights), sum(weights)
        res = right  # Worst case: ship everything in 1 day
        
        def canShip(cap):
            """Check if we can ship all packages with given capacity."""
            ships = 1  # Start with day 1
            curr_cap = cap  # Remaining capacity for current day
            
            for w in weights:
                if curr_cap - w < 0:
                    # Package doesn't fit today, ship tomorrow
                    ships += 1
                    curr_cap = cap  # Reset to full capacity
                
                curr_cap -= w  # Load this package
            
            return ships <= days
        
        # Binary search for minimum capacity
        while left <= right:
            mid = (left + right) // 2
            
            if canShip(mid):
                # mid works! Try to find smaller capacity
                res = mid
                right = mid - 1
            else:
                # mid too small, need larger capacity
                left = mid + 1
        
        return res
```

---

## Detailed Walkthrough

### Example 1: `weights = [1,2,3,4,5,6,7,8,9,10], days = 5`

**Setup:**
- left = 10 (max weight)
- right = 55 (sum of weights)
- Need minimum capacity

**Step-by-Step Execution:**

| Step | Left | Right | Capacity | Simulation | Days Needed | Check | Action | res |
|------|------|-------|----------|------------|-------------|-------|--------|-----|
| 0 | 10 | 55 | 32 | 1-10(55 total) | 1 | 1≤5 ✓ | right=31 | 32 |
| 1 | 10 | 31 | 20 | Day1:1-8(17), Day2:9-10(19) | 2 | 2≤5 ✓ | right=19 | 20 |
| 2 | 10 | 19 | 14 | Day1:1-5(14), Day2:6-7(13), Day3:8(8), Day4:9(9), Day5:10(10) | 5 | 5≤5 ✓ | right=13 | 14 |
| 3 | 10 | 13 | 11 | Day1:1-5(11), Day2:6(6), Day3:7(7), Day4:8(8), Day5:9(9), Day6:10(10) | 6 | 6≤5 ✗ | left=12 | 14 |
| 4 | 12 | 13 | 12 | Day1:1-6(12), Day2:7(7), Day3:8(8), Day4:9(9), Day5:10(10) | 5 | 5≤5 ✓ | right=11 | 12 |
| 5 | 12 | 11 | - | - | - | left>right | Stop | - |

**Detailed Simulation for Capacity 15:**

```
Capacity = 15

Day 1: Load 1,2,3,4,5
- After 1: 15-1=14 left
- After 2: 14-2=12 left
- After 3: 12-3=9 left
- After 4: 9-4=5 left
- After 5: 5-5=0 left
- After 6?: 0-6 < 0, can't fit

Day 2: Load 6,7
- Start: 15 capacity
- After 6: 15-6=9 left
- After 7: 9-7=2 left
- After 8?: 2-8 < 0, can't fit

Day 3: Load 8
- Start: 15 capacity
- After 8: 15-8=7 left
- After 9?: 7-9 < 0, can't fit

Day 4: Load 9
- Start: 15 capacity
- After 9: 15-9=6 left
- After 10?: 6-10 < 0, can't fit

Day 5: Load 10
- Start: 15 capacity
- After 10: 15-10=5 left

Total: 5 days ✓
```

**Final Answer:** `15` ✓

---

### Example 2: `weights = [3,2,2,4,1,4], days = 3`

**Setup:**
- left = 4, right = 16

| Step | Left | Right | Cap | Simulation | Days | Check | Action | res |
|------|------|-------|-----|------------|------|-------|--------|-----|
| 0 | 4 | 16 | 10 | [3,2,2], [4,1,4] | 2 | 2≤3 ✓ | right=9 | 10 |
| 1 | 4 | 9 | 6 | [3,2], [2,4], [1,4] | 3 | 3≤3 ✓ | right=5 | 6 |
| 2 | 4 | 5 | 4 | [3], [2,2], [4], [1], [4] | 5 | 5≤3 ✗ | left=5 | 6 |
| 3 | 5 | 5 | 5 | [3,2], [2], [4,1], [4] | 4 | 4≤3 ✗ | left=6 | 6 |
| 4 | 6 | 5 | - | - | - | left>right | Stop | - |

**Simulation for Capacity 6:**

```
Day 1: 3,2 (total: 5, next 2 would exceed 6)
Day 2: 2,4 (total: 6, next 1 would fit but then 4 wouldn't complete the pattern properly)
Actually:
- After 3: 6-3=3 left
- After 2: 3-2=1 left
- After 2?: 1-2 < 0, new day

Day 2: 2,4
- Start: 6
- After 2: 6-2=4 left
- After 4: 4-4=0 left
- After 1?: 0-1 < 0, new day

Day 3: 1,4
- Start: 6
- After 1: 6-1=5 left
- After 4: 5-4=1 left

Total: 3 days ✓
```

**Final Answer:** `6` ✓

---

### Example 3: `weights = [1,2,3,1,1], days = 4`

| Step | Left | Right | Cap | Days Needed | Check | Action | res |
|------|------|-------|-----|-------------|-------|--------|-----|
| 0 | 3 | 8 | 5 | 2 | 2≤4 ✓ | right=4 | 5 |
| 1 | 3 | 4 | 3 | 3 | 3≤4 ✓ | right=2 | 3 |
| 2 | 3 | 2 | - | - | left>right | Stop | - |

**Simulation for Capacity 3:**

```
Day 1: 1,2 (total: 3)
Day 2: 3 (total: 3)
Day 3: 1,1 (total: 2)

Total: 3 days ≤ 4 days ✓
```

**Final Answer:** `3` ✓

---

## Why Binary Search Works

### The Monotonic Property

**Observation:** If capacity C works, then any C' > C also works.

```
Proof:
If capacity C can ship all packages in ≤ days,
then larger capacity C' > C will ship even faster (≤ days).

Example: weights = [1,2,3,4,5,6,7,8,9,10], days = 5

Capacity 15: 5 days ✓
Capacity 16: 5 days or less ✓
Capacity 20: 4 days ✓
Capacity 55: 1 day ✓

This creates sorted property:
[10:✗, 11:✗, 12:✗, 13:✗, 14:✗, 15:✓, 16:✓, ..., 55:✓]
         doesn't work          |      works

We want first ✓ → Binary search!
```

**Visual Representation:**

```
Capacity:    10  11  12  13  14  15  16  17  18  ...  55
Days needed: 10  10   7   7   6   5   5   5   4  ...   1
Valid?       ✗   ✗   ✗   ✗   ✗   ✓   ✓   ✓   ✓       ✓
                              ↑
                          First True
                          Answer = 15
```

---

## Edge Cases

### 1. Single Package
```python
weights = [50], days = 1
# Output: 50
# Only one package, capacity must be 50
```

### 2. Days Equal to Number of Packages
```python
weights = [1,2,3,4,5], days = 5
# Output: 5 (max weight)
# Can ship one package per day
# Capacity must be ≥ heaviest package
```

### 3. Single Day Available
```python
weights = [1,2,3,4,5], days = 1
# Output: 15 (sum of weights)
# Must ship everything in 1 day
```

### 4. More Days Than Packages
```python
weights = [1,2,3], days = 10
# Output: 3 (max weight)
# Extra days don't reduce minimum capacity
# Still need to carry heaviest package
```

### 5. All Same Weight
```python
weights = [5,5,5,5,5], days = 3
# Output: 10
# With capacity 10, can ship 2 per day
# Day 1: 5,5
# Day 2: 5,5
# Day 3: 5
```

### 6. One Heavy Package, Many Light
```python
weights = [1,1,1,1,1,100], days = 2
# Output: 100
# Must be able to carry the 100
# Day 1: 1,1,1,1,1 (total: 5)
# Day 2: 100
```

### 7. Decreasing Weights
```python
weights = [10,9,8,7,6,5,4,3,2,1], days = 5
# Output: 15
# Same as Example 1 but reversed
# Order matters for simulation!
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Minimum Capacity
```python
# Wrong! Should be max(weights), not min(weights)
left = min(weights)
```

**Fix:**
```python
left = max(weights)
```

**Why?** Must be able to carry the heaviest single package.

### ❌ Mistake 2: Wrong Maximum Capacity
```python
# Wrong! Should be sum(weights), not max(weights) * len(weights)
right = max(weights) * len(weights)
```

**Fix:**
```python
right = sum(weights)
```

### ❌ Mistake 3: Not Simulating in Order
```python
# Wrong! Packages must be loaded in order
weights.sort()  # Don't sort!
```

**Fix:**
```python
# Keep original order, simulate sequentially
for w in weights:  # Original order
    # ...
```

**Why?** Problem states packages are on conveyor belt in given order.

### ❌ Mistake 4: Off-by-One in Day Counting
```python
# Wrong! Starting with ships = 0
ships = 0
for w in weights:
    if curr_cap - w < 0:
        ships += 1
        curr_cap = capacity
    curr_cap -= w
```

**Fix:**
```python
ships = 1  # Start with day 1
# ...
```

### ❌ Mistake 5: Wrong Capacity Check
```python
# Wrong! Should check curr_cap - w < 0
if curr_cap < w:  # This is same, but less clear about the logic
    # ...
```

**Fix:**
```python
if curr_cap - w < 0:  # More explicit about exceeding capacity
    ships += 1
    curr_cap = capacity
curr_cap -= w
```

---

## Interview Insights

**Question Recognition:**
- "Minimum capacity" + "within days" + sequential processing → Binary search on answer
- Similar pattern to Koko Eating Bananas

**Key Points to Mention:**

1. **Binary search on answer space:**
   - "We're searching for the minimum ship capacity between max(weights) and sum(weights)."

2. **Why max(weights) is lower bound:**
   - "The ship must be able to carry at least the heaviest package, otherwise that package can never ship."

3. **Sequential constraint:**
   - "Packages must be shipped in order, so we simulate loading sequentially until capacity is reached."

4. **Monotonic property:**
   - "If capacity C works, any larger capacity also works. This monotonicity enables binary search."

**Follow-up Questions:**

1. **Q:** "What if packages can be shipped in any order?"
   - **A:** Sort by weight descending, use greedy first-fit. But this changes the problem fundamentally.

2. **Q:** "What if we want to maximize days instead of minimize capacity?"
   - **A:** Different problem. For fixed capacity, simulate to find maximum days needed.

3. **Q:** "Can you optimize the simulation?"
   - **A:** Current O(n) per check is necessary. We must process all packages sequentially.

4. **Q:** "How is this different from Koko Eating Bananas?"
   - **A:** Very similar pattern! Both are "minimize resource" problems with feasibility checks. Koko has independent piles, this has sequential constraint.

**Complexity Analysis:**
- Time: O(n log S) where n = len(weights), S = sum(weights)
- Space: O(1) - Only using variables

---

## Related Problems

1. **Koko Eating Bananas (LeetCode 875)** - Very similar binary search on answer
2. **Split Array Largest Sum (LeetCode 410)** - Minimize maximum subarray sum
3. **Minimize Max Distance to Gas Station (LeetCode 774)** - Binary search on answer
4. **Magnetic Force Between Two Balls (LeetCode 1552)** - Maximize minimum distance
5. **Cutting Ribbons (LeetCode 1891)** - Maximize length with count constraint
6. **Divide Chocolate (LeetCode 1231)** - Maximize minimum sweetness

---

## Comparison: Similar Problems

| Problem | Minimize/Maximize | Constraint | Range |
|---------|-------------------|------------|-------|
| **Ship Packages** | Minimize capacity | ≤ days | [max(w), sum(w)] |
| **Koko Bananas** | Minimize speed | ≤ hours | [1, max(piles)] |
| **Split Array** | Minimize max sum | ≤ m splits | [max(nums), sum(nums)] |
| **Magnetic Balls** | Maximize min dist | = m balls | [1, max position] |

**Common Pattern:**
- All use binary search on answer space
- All have feasibility check function
- All exploit monotonic property
- All are O(n log range)

---

## Variations

### Variation 1: Return Days Used with Optimal Capacity
```python
def shipWithDays(weights, days):
    """Return both minimum capacity and actual days used."""
    left, right = max(weights), sum(weights)
    res = right
    
    def countDays(cap):
        ships = 1
        curr_cap = cap
        for w in weights:
            if curr_cap - w < 0:
                ships += 1
                curr_cap = cap
            curr_cap -= w
        return ships
    
    while left <= right:
        mid = (left + right) // 2
        days_needed = countDays(mid)
        
        if days_needed <= days:
            res = mid
            right = mid - 1
        else:
            left = mid + 1
    
    return res, countDays(res)
```

### Variation 2: Find Maximum Days Possible
```python
def maxShippingDays(weights, capacity):
    """Find how many days needed with given capacity."""
    days = 1
    curr_cap = capacity
    
    for w in weights:
        if curr_cap - w < 0:
            days += 1
            curr_cap = capacity
        curr_cap -= w
    
    return days
```

### Variation 3: Minimum Capacity with Package Limit Per Day
```python
def shipWithPackageLimit(weights, days, max_packages_per_day):
    """Additional constraint: max packages per day."""
    left, right = max(weights), sum(weights)
    res = right
    
    def canShip(cap):
        ships = 1
        curr_cap = cap
        packages_today = 0
        
        for w in weights:
            if curr_cap - w < 0 or packages_today >= max_packages_per_day:
                ships += 1
                curr_cap = cap
                packages_today = 0
            
            curr_cap -= w
            packages_today += 1
        
        return ships <= days
    
    while left <= right:
        mid = (left + right) // 2
        
        if canShip(mid):
            res = mid
            right = mid - 1
        else:
            left = mid + 1
    
    return res
```

---

## Summary

**Key Takeaways:**

1. **Binary Search on Answer:**
   - Search for minimum capacity, not in array
   - Range: [max(weights), sum(weights)]
   - Find minimum that satisfies constraint

2. **Lower Bound: max(weights):**
   - Must carry heaviest single package
   - Any capacity less can't ship that package
   - This is the theoretical minimum

3. **Upper Bound: sum(weights):**
   - Can ship everything in 1 day
   - Any larger capacity is unnecessary
   - We want minimum, so this is the limit

4. **Sequential Constraint:**
   - Packages must be loaded in order
   - Can't rearrange or sort
   - Simulate loading sequentially

5. **Monotonic Property:**
   - If capacity C works, C+1 works
   - Creates [✗✗✗✓✓✓] pattern
   - Perfect for binary search

**Template:**
```python
left, right = max(weights), sum(weights)
res = right

def canShip(cap):
    ships = 1
    curr_cap = cap
    
    for w in weights:
        if curr_cap - w < 0:
            ships += 1
            curr_cap = cap
        curr_cap -= w
    
    return ships <= days

while left <= right:
    mid = (left + right) // 2
    
    if canShip(mid):
        res = mid
        right = mid - 1
    else:
        left = mid + 1

return res
```

**Mental Model:**
- Imagine a ship with adjustable capacity
- Too small: can't carry heavy packages or needs too many trips
- Too large: wastes capacity, not minimum
- Binary search finds the "Goldilocks" capacity
- Just enough to meet the deadline, no more
- Sequential loading constraint makes simulation necessary
- This pattern appears in many "resource allocation" problems!
