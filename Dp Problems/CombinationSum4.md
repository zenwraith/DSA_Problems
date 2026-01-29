# Combination Sum IV

**LeetCode:** [Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/)

---

## Problem Statement

Given an array of distinct integers `nums` and a target integer `target`, return the number of possible combinations that add up to `target`. The order of numbers matters (i.e., this is counting **permutations**, not combinations).

**Example:**
- `nums = [1,2,3]`, `target = 4` → Output: 7
  - The possible combinations are: (1,1,1,1), (1,1,2), (1,2,1), (2,1,1), (2,2), (1,3), (3,1)
- `nums = [9]`, `target = 3` → Output: 0

---

## Strategy: Permutation DP (Target Outside)

**Key Insight:** Despite the name "Combination Sum IV", this problem actually counts **permutations** because order matters. `[1,2]` and `[2,1]` are counted as different ways.

We use `dp[i]` to represent the number of ways to form amount `i`.

### Why Target on Outside?

To get permutations, we iterate through **target amounts** in the outer loop and try all **nums** in the inner loop. This allows numbers to appear in any order.

**Target Outside (Permutations):**
```python
for i in range(1, target + 1):      # Target amounts outside
    for num in nums:                 # Try all nums for each amount
        if i >= num:
            dp[i] += dp[i - num]
```

**Nums Outside (Combinations):**
```python
for num in nums:                     # Process each num once
    for i in range(num, target + 1): # Update all reachable amounts
        dp[i] += dp[i - num]
```

---

## Solution

```python
class Solution:
    def combinationSum4(self, nums: List[int], target: int) -> int:
        # dp[i] = number of ways to form amount i
        dp = [0] * (target + 1)
        
        # Base case: one way to make 0 (select nothing)
        dp[0] = 1
        
        # Target on outside = PERMUTATIONS
        for i in range(1, target + 1):
            for num in nums:
                if i >= num:
                    # Add ways from (current amount - num)
                    dp[i] += dp[i - num]
        
        return dp[target]
```

---

## Permutations vs Combinations

### Example: `nums = [1, 2]`, `target = 3`

**Permutations (this problem):**
- Ways: `[1,1,1]`, `[1,2]`, `[2,1]`
- Count: 3
- Loop order: **Target outside, nums inside**

**Combinations (Coin Change 2):**
- Ways: `[1,1,1]`, `[1,2]` (treating `[1,2]` and `[2,1]` as same)
- Count: 2
- Loop order: **Nums outside, target inside**

---

## How It Works

For each amount `i`, we ask: "How many ways can I reach `i` by adding one more number?"

**Building `dp` for `nums = [1,2,3]`, `target = 4`:**

| i | Calculation | dp[i] |
|---|-------------|-------|
| 0 | Base case | 1 |
| 1 | dp[0] (using 1) | 1 |
| 2 | dp[1] (using 1) + dp[0] (using 2) | 2 |
| 3 | dp[2] (using 1) + dp[1] (using 2) + dp[0] (using 3) | 4 |
| 4 | dp[3] (using 1) + dp[2] (using 2) + dp[1] (using 3) | 7 |

---

## Complexity Analysis

- **Time Complexity:** $O(\text{target} \times n)$ where $n$ is the length of `nums`
- **Space Complexity:** $O(\text{target})$ for the DP array