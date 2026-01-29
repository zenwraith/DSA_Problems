# **Burst Balloons**

**LeetCode Link:** [https://leetcode.com/problems/burst-balloons/](https://leetcode.com/problems/burst-balloons/)



## **Solution 1: Bottom-Up (Tabulation)**

```python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        nums = [1] + nums + [1]

        n = len(nums)

        # dp[i][j] = max coins from bursting balloons between i and j
        dp = [[0] * n for i in range(n)]

        # Fill the DP table by increasing gap size
        for gap in range(2, n):
            for i in range(n - gap):
                j = i + gap

                # Try each balloon k as the last one to burst
                for k in range(i + 1, j):
                    dp[i][j] = max(dp[i][j], nums[i] * nums[k] * nums[j] + dp[i][k] + dp[k][j])

        return dp[0][n - 1]
```

## **Solution 2: Top-Down (Memoization)**

```python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        nums = [1] + nums + [1]

        memo = {}

        def solve(i, j):
            # no balloons between i and j
            if j - i <= 1:
                return 0

            if (i, j) in memo:
                return memo[(i, j)]

            res = 0

            # trying every balloon 'k' as the LAST one to burst in this range
            for k in range(i + 1, j):
                # coins = coins from bursting k LAST + coins from left side + coins from right side
                # bursting k last means its neighbors are nums[i] and nums[j]
                coins = nums[i] * nums[j] * nums[k]

                coins += solve(i, k) + solve(k, j)

                res = max(res, coins)

            memo[(i, j)] = res
            return res

        return solve(0, len(nums) - 1)
```



## **Why Do We Burst the Last Balloon?**

### **The Problem with Bursting First**

If we think "which balloon do I burst **first**?":

```
Array: [3, 1, 5, 8]

If we burst balloon 1 first (value 1):
- We get coins: 3 * 1 * 5 = 15
- Array becomes: [3, 5, 8]

Now we need to recurse on [3, 5, 8], but the structure changed!
The neighborhood relationships are now different.
```

This creates a **complex state tracking problem** because after each burst, the neighbors of remaining balloons change.

### **Why Bursting Last Works Better**

If we think "which balloon should I burst **last**?":

```
For range [left, right], consider bursting balloon k LAST:

Before bursting k:
[left] ... [everything between left and k] ... [k] ... [everything between k and right] ... [right]

After all the other balloons are burst:
[left] -------- EMPTY -------- [k] -------- EMPTY -------- [right]

When we finally burst k, its neighbors are ALWAYS left and right!
```

This is the **key insight**: When balloon $k$ is the **last** to be burst in range $(left, right)$, its neighbors are **guaranteed to be** `nums[left]` and `nums[right]` because all balloons between them are already gone.

### **The Clean Recurrence**

This leads to a beautiful recurrence relation:

$$dp[left][right] = \max_{k \in (left, right)} \{ dp[left][k] + dp[k][right] + nums[left] \times nums[k] \times nums[right] \}$$

Where:
- **$dp[left][k]$** = max coins from bursting balloons in range $(left, k)$
- **$dp[k][right]$** = max coins from bursting balloons in range $(k, right)$
- **$nums[left] \times nums[k] \times nums[right]$** = coins when bursting $k$ last (with **fixed** neighbors)

### **Comparison: Burst First vs. Burst Last**

| Approach | Neighbors After Burst | State Complexity |
|----------|----------------------|------------------|
| **Burst First** | **Change dynamically** | Need to track which neighbors are still alive → **Complicated** |
| **Burst Last** | **Always** left and right | Neighbors are **fixed** → **Clean DP formula** |

The **"burst last" perspective** transforms interval DP from tracking dynamic neighbors into a clean problem where we know exactly what the neighbors will be when we burst a balloon.

## **The Gap Pattern: Where It Appears**

The `gap` pattern used in this problem appears across many **Interval DP problems**. This pattern is used whenever you need to:

1. Partition an interval `[i, j]` into two subintervals `[i, k]` and `[k, j]`
2. Solve smaller problems before larger ones
3. Build up to the final answer

### **Common Problems Using This Pattern**

| Problem | What it Does |
|---------|--------------|
| **Burst Balloons** (This) | Maximize coins by choosing last balloon to burst |
| **Minimum Cost to Cut a Stick** | Minimize cost by choosing optimal cut point |
| **Matrix Chain Multiplication** | Minimize multiplications by choosing split point |
| **Palindrome Partitioning II** | Find minimum cuts to create palindromes |

### **The Loop Structure**

```python
for gap in range(2, n):               # LOOP 1: Interval size (2 to n)
    for i in range(n - gap):          # LOOP 2: Start position
        j = i + gap                   # End position is fixed by gap
        
        for k in range(i + 1, j):     # LOOP 3: Try each split point
            # Combine solutions from two subproblems
            dp[i][j] = optimal(dp[i][j], dp[i][k], dp[k][j])
```

### **Why This Pattern Works**

- **Gap = 2:** Smallest subproblems (intervals with just 1 item between endpoints). All have answer 0 or base case.
- **Gap increases:** Larger intervals build on already-solved smaller intervals.
- **Diagonal Filling:** Table fills from the center diagonal outward (like peeling an onion), ensuring dependencies are resolved.

### **Visual Pattern**

```
DP Table (n x n):

Gap=2:  X X X X X      Gap=3:      X X X X      Gap=4:          X X X
        X X X X X              X X X X              X X X
        X X X X X              X X X X              X X X
        X X X X X              X X X X
        X X X X X

Each diagonal represents one gap size. Compute fully before moving to next gap.
```

This pattern guarantees that whenever you compute `dp[i][j]`, both `dp[i][k]` and `dp[k][j]` are already calculated.

## **Summary of Interval DP**

The "Aha!" moment for these problems is realizing that:

- **Table Size:** Use the number of items (balloons/cuts), not the values.
- **The Split Point k:** In the inner loop, k represents the "divider" that splits your current interval into two smaller, already-solved intervals.
- **The Final Answer:** Usually sits at dp[0][n-1].

