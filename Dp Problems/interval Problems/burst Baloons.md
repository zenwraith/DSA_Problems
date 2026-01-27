# **Burst Balloons**

**LeetCode Link:** [https://leetcode.com/problems/burst-balloons/](https://leetcode.com/problems/burst-balloons/)




````python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:

        nums = [1] + nums + [1]

        memo = {}

        def solve(i,j):
            # no ballons between i and j
            if j-i <= 1:
                return 0
            
            if (i,j) in memo:
                return memo[(i,j)]
            
            res = 0

            # trying every ballon 'k' as the LAST one to burst in this range
            for k in range(i+1, j):
                # coins = coins from bursting k LAST + coins from left side+ coins from right side.
                # bursting k last means its neighbors are nums[i] and nums[j]
                coins = nums[i] * nums[j] *nums[k]



                coins += solve(i,k) + solve(k,j)

                res = max(res , coins)

            memo[(i,j)] = res
            return res

        return solve(0, len(nums) - 1)

````

````python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:

        nums = [1] + nums + [1]

        n = len(nums)

        dp = [ [ 0] * n for i in range(n)]

        for gap in range(2,n):
            for i in range(n - gap):

                j = i + gap

                for k in range(i+1,j):
                    dp[i][j] = max( dp[i][j] , nums[i]*nums[k]*nums[j] + dp[i][k] + dp[k][j])


        return dp[0][n-1]
````
## **Summary of Interval DP**
The "Aha!" moment for these problems is realizing that:

- **Table Size:** Use the number of items (balloons/cuts), not the values.
- **The Split Point k:** In the inner loop, k represents the "divider" that splits your current interval into two smaller, already-solved intervals.
- **The Final Answer:** Usually sits at dp[0][n-1].

