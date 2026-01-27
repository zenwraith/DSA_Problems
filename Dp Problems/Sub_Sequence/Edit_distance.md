
# **Edit Distance**

**LeetCode URL:** [https://leetcode.com/problems/edit-distance/](https://leetcode.com/problems/edit-distance/)

## **Solution**

```python

class Solution:
    def minDistance(self, word1: str, word2: str) -> int:

        n,m = len(word1) , len(word2)

        dp = [ [0] * (m+1) for i in range(n+1)]

        for i in range(n+1):
            dp[i][0] = i 
        
        for j in range(m+1):
            dp[0][j] = j

        for i in range(1,n+1):
            for j in range(1, m+1):

                if word1[i-1] == word2[j-1]:
                    dp[i][j] = dp[i-1][j-1]
                else:
                    insert = dp[i][j-1]
                    delete = dp[i-1][j]
                    replace = dp[i-1][j-1]
                    dp[i][j] = 1 + min( insert, delete, replace)

        return dp[n][m]
```

## **Key Insight**

> **Why dp[i-1][j] works for *?** This is the Unbounded Knapsack logic again! When we say `dp[i][j] = dp[i-1][j]`, we are saying: "The * already matched something at i-1, and we are using it again to match the character at i." Because we look at the same column j, the * remains available for the next row.