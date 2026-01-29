# Target Sum

**LeetCode:** [Target Sum](https://leetcode.com/problems/target-sum/)

---

## Problem Statement

You are given an integer array `nums` and an integer `target`. You want to build an expression by adding '+' or '-' before each integer in `nums` and concatenating them. Return the number of different expressions you can build that evaluates to `target`.

**Example:**
- `nums = [1,1,1,1,1]`, `target = 3` → Output: 5
  - Expressions: +1+1+1+1-1, +1+1+1-1+1, +1+1-1+1+1, +1-1+1+1+1, -1+1+1+1+1

---

## Solution 1 – Recursive with Memoization

### Strategy

Try all possible combinations of '+' and '-' for each number, using memoization to avoid recalculating the same state.

```python
class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        n = len(nums)
        
        @cache
        def helper(index, target):
            # Base case: reached end of array
            if index == n:
                return 1 if target == 0 else 0
            
            # Try adding + sign (subtract from target)
            adding = helper(index + 1, target - nums[index])
            
            # Try adding - sign (add to target)
            removing = helper(index + 1, target + nums[index])
            
            return adding + removing
        
        return helper(0, target)
```

---

## Solution 2 – DP Bottom-Up (Math Approach)

### Key Insight: Reduction to Subset Sum

This is actually a math problem! If we assign '+' to some numbers (subset $P$) and '-' to others (subset $N$), we get:

$$\text{Sum}(P) - \text{Sum}(N) = \text{target}$$

Since $\text{Sum}(P) + \text{Sum}(N) = \text{TotalSum}$, we can derive:

$$
\begin{align}
\text{Sum}(P) - \text{Sum}(N) &= \text{target} \\
\text{Sum}(P) + \text{Sum}(N) &= \text{TotalSum} \\
\hline
2 \cdot \text{Sum}(P) &= \text{target} + \text{TotalSum} \\
\text{Sum}(P) &= \frac{\text{target} + \text{TotalSum}}{2}
\end{align}
$$

**So the problem becomes:** Find the number of ways to form a subset that sums up to $\text{Sum}(P)$.

This is the classic **Subset Sum Count** problem!

### Edge Cases

- If `(target + total)` is odd, no solution exists (cannot divide evenly)
- If `|target| > total`, no solution exists (impossible to reach)

```python
class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        total = sum(nums)
        
        # Check if solution is possible
        if (target + total) % 2 != 0 or abs(target) > total:
            return 0
        
        # Calculate subset sum we need to find
        subset_goal = (target + total) // 2
        
        # dp[i] = number of ways to sum to i
        dp = [0] * (subset_goal + 1)
        dp[0] = 1  # One way to make 0: select nothing
        
        # COMBINATION PATTERN: Items on outside
        for num in nums:
            # 0/1 PATTERN: Loop backward to use each num only once
            for i in range(subset_goal, num - 1, -1):
                dp[i] += dp[i - num]
        
        return dp[subset_goal]
```

---

## Why Iterate Backward?

This is the **0/1 Knapsack pattern** - each number can only be used once.

- **Backward iteration** ensures we use values from the previous state (before updating)
- **Forward iteration** would allow using the same number multiple times (unbounded knapsack)

---

## Complexity Analysis

### Recursive with Memoization
- **Time Complexity:** $O(n \times \text{sum})$ where sum is the range of possible targets
- **Space Complexity:** $O(n \times \text{sum})$ for memoization + recursion stack

### DP Bottom-Up
- **Time Complexity:** $O(n \times \text{subset\_goal})$ where subset_goal = (target + total) / 2
- **Space Complexity:** $O(\text{subset\_goal})$ for the DP array