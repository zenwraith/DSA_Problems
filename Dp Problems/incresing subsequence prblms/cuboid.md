

# **Maximum Height by Stacking Cuboids**

**LeetCode Link:** [https://leetcode.com/problems/maximum-height-by-stacking-cuboids/](https://leetcode.com/problems/maximum-height-by-stacking-cuboids/)

---

## **Problem Statement**

Given `n` cuboids where the dimensions of the `i`-th cuboid is `cuboids[i] = [width_i, length_i, height_i]` (0-indexed). Choose a subset of the cuboids and place them on each other.

You can place cuboid `i` on cuboid `j` if `width_i ≤ width_j` and `length_i ≤ length_j` and `height_i ≤ height_j`. You can rearrange any cuboid's dimensions by rotating it to put it on another cuboid.

Return the **maximum height** of the stacked cuboids.

**Example:**
```
Input: cuboids = [[50,45,20],[95,37,53],[45,23,12]]
Output: 190
Explanation: 
Cuboid 1 is placed on cuboid 0.
Cuboid 0 is placed on cuboid 2.
The height is 45 + 95 + 50 = 190.
```

---

## **Strategy: LIS Variant with Sorting**

**Key Insights:**

1. **We can rotate each cuboid** to any orientation, so we should orient it optimally
2. **Optimal orientation**: Sort each cuboid's dimensions so the largest is the "height"
3. **This becomes an LIS problem**: Find the longest "stackable" sequence

### **Why Sort Dimensions?**

Consider a cuboid `[50, 45, 20]`:
- We can orient it as: `[20, 45, 50]`, `[20, 50, 45]`, `[45, 20, 50]`, etc.
- **Best strategy**: Sort to `[20, 45, 50]` and use 50 as height
- **Why?** Using the largest dimension as height maximizes our total height

### **Why Sort Cuboids Array?**

After sorting each cuboid's dimensions, we sort the entire array:
```python
for c in cuboids:
    c.sort()  # [w, l, h] where w ≤ l ≤ h
cuboids.sort()  # Sort all cuboids
```

**Purpose:** This ensures that when we check if cuboid `j` can go under cuboid `i` (where `j < i`), we only need to verify all three dimensions, not worry about permutations.

---

## **Solution**

```python
class Solution:
    def maxHeight(self, cuboids: List[List[int]]) -> int:
        # Step 1: Sort each cuboid's dimensions (smallest to largest)
        for c in cuboids:
            c.sort()
        
        # Step 2: Sort all cuboids (lexicographically)
        # This ensures smaller cuboids come first
        cuboids.sort()
        
        n = len(cuboids)
        # dp[i] = maximum height achievable with cuboid i at the top
        dp = [0] * n
        
        for i in range(n):
            # Base case: cuboid i stands alone, height = cuboids[i][2]
            dp[i] = cuboids[i][2]
            
            # Look back at all previous cuboids
            for j in range(i):
                # Check if cuboid j can fit under cuboid i
                # All three dimensions must satisfy: j[dim] ≤ i[dim]
                if (cuboids[j][0] <= cuboids[i][0] and 
                    cuboids[j][1] <= cuboids[i][1] and 
                    cuboids[j][2] <= cuboids[i][2]):
                    # Stack j under i and take max
                    dp[i] = max(dp[i], dp[j] + cuboids[i][2])
        
        return max(dp)
```

---

## **Example Walkthrough**

**Input:** `cuboids = [[50,45,20], [95,37,53], [45,23,12]]`

### **Step 1: Sort Each Cuboid's Dimensions**
```
[50,45,20] → [20,45,50]
[95,37,53] → [37,53,95]
[45,23,12] → [12,23,45]
```

### **Step 2: Sort All Cuboids**
```
cuboids = [
    [12,23,45],  # Index 0
    [20,45,50],  # Index 1
    [37,53,95]   # Index 2
]
```

### **Step 3: DP Computation**

**For i=0:** `[12,23,45]`
```
dp[0] = 45 (stands alone)
```

**For i=1:** `[20,45,50]`
```
Check j=0: [12,23,45] vs [20,45,50]
  12 ≤ 20 ✓
  23 ≤ 45 ✓
  45 ≤ 50 ✓
  Can stack! dp[1] = max(50, 45 + 50) = 95
```

**For i=2:** `[37,53,95]`
```
Check j=0: [12,23,45] vs [37,53,95]
  12 ≤ 37 ✓
  23 ≤ 53 ✓
  45 ≤ 95 ✓
  Can stack! dp[2] = max(95, 45 + 95) = 140

Check j=1: [20,45,50] vs [37,53,95]
  20 ≤ 37 ✓
  45 ≤ 53 ✓
  50 ≤ 95 ✓
  Can stack! dp[2] = max(140, 95 + 95) = 190
```

**Final:** `max(dp) = 190`

**Stacking Order:** Cuboid 2 (height 45) → Cuboid 1 (height 50) → Cuboid 0 (height 95) = Total 190

---

## **Why This Sorting Works**

**Claim:** After sorting each cuboid and sorting the array, we can use a simple LIS-style DP.

**Proof Sketch:**
- **Sorting dimensions** ensures we use the maximum dimension as height
- **Sorting cuboids** creates a partial order: if cuboid `i` comes before `j`, then `i[0] ≤ j[0]`
- This means we only need to check the remaining two dimensions to determine stackability

**Example of Why Sorting Matters:**
```
Without sorting:
[95,37,53] comes before [20,45,50]
We'd need to check: can [95,37,53] go under [20,45,50]?
95 > 20 ✗ Immediately fails!

With sorting:
[20,45,50] comes before [37,53,95]
We'd need to check: can [20,45,50] go under [37,53,95]?
20 ≤ 37 ✓, 45 ≤ 53 ✓, 50 ≤ 95 ✓ Works!
```

---

## **Why Not Greedy?**

**Question:** Why not just sort by volume and stack the largest ones?

**Answer:** Because dimensions matter, not just volume!

**Example:**
```
Cuboid A: [1, 100, 100] → Volume = 10,000
Cuboid B: [50, 50, 50] → Volume = 125,000

Can B go under A? 
50 ≤ 1? ✗ No!

Can A go under B?
1 ≤ 50 ✓
100 ≤ 50? ✗ No!

They can't stack at all despite different volumes!
```

---

## **Comparison with Standard LIS**

| Aspect | Standard LIS | Stacking Cuboids |
|--------|--------------|------------------|
| **1D Comparison** | `nums[i] > nums[j]` | All 3 dims: `c[i] ≥ c[j]` |
| **DP Value** | Length of sequence | Total height |
| **Preprocessing** | None | Sort dimensions + sort array |
| **Base Case** | `dp[i] = 1` | `dp[i] = cuboids[i][2]` |
| **Transition** | `dp[j] + 1` | `dp[j] + cuboids[i][2]` |

---

## **Complexity Analysis**

**Time Complexity:** $O(n^2)$  
- Sorting each cuboid: $O(n \times 3 \log 3) = O(n)$
- Sorting all cuboids: $O(n \log n)$
- DP with nested loops: $O(n^2)$
- **Total:** $O(n^2)$

**Space Complexity:** $O(n)$  
DP array of size $n$.