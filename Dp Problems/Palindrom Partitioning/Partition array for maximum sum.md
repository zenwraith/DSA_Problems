
leetcode linke : https://leetcode.com/problems/partition-array-for-maximum-sum/description/


# **Partition Array for Maximum Sum**

**LeetCode Link:** [https://leetcode.com/problems/partition-array-for-maximum-sum/](https://leetcode.com/problems/partition-array-for-maximum-sum/)

## **Strategy: Linear DP with "Look Back" Window**

We define `dp[i]` as the maximum sum we can get for the prefix `arr[0...i-1]`.

To calculate `dp[i]`, we look at the last partition. This last partition could have a size anywhere from 1 to $k$.

### **Recurrence Relation**

| Last Partition Length | Calculation |
|---|---|
| Length 1 | `dp[i] = dp[i-1] + (arr[i-1] × 1)` |
| Length 2 | `dp[i] = dp[i-2] + (max(arr[i-2...i-1]) × 2)` |
| Length $len$ | `dp[i] = dp[i-len] + (max(arr[i-len...i-1]) × len)` |

We try all possible lengths from 1 to $k$ and pick the maximum.

## **Solution**

class Solution:
    def maxSumAfterPartitioning(self, arr: List[int], k: int) -> int:

        n = len(arr)
        # dp[i] is the max sum for the first i elements
        dp = [0] * (n+1)

        for i in range(1,n+1):

            current_max = 0
            # Try every possible length for the LAST subarray (up to k)
            # We ensure i - j >= 0 so we don't go out of bounds

            for j in range(1, min(k,i) +1):
                # Update the max element in the current window [i-j ... i-1]
                current_max = max(current_max , arr[i-j])
                # Recurrence: best sum of previous part + (max of current part * length)
                dp[i] = max(dp[i] , dp[i-j] + current_max * j)

        return dp[n]
```