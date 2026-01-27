# **Wildcard Matching**

**LeetCode URL:** [https://leetcode.com/problems/wildcard-matching/](https://leetcode.com/problems/wildcard-matching/)

## **Solution**

```python

class Solution:
    def isMatch(self, s: str, p: str) -> bool:

        n,m = len(s) , len(p)

        # base 1:  initialize DP table with Fales
        dp = [ [ False]*(m+1) for i in range(n+1) ]

        # base 2:  Empty string matches empty pattern
        dp[0][0] = True

        # base 3 : patern has '*' that can match empty string
        for j in range(1,m+1):
            if p[j-1] == '*':
                dp[0][j] = dp[0][j-1]
            else:
                # once we hit a non-*, no feature empty matches are possible
                break
        
        for i in range(1, n+1):
            for j in range(1, m+1):

                if p[j-1] == '?' or s[i-1] == p[j-1]:
                    # Diagonal : Single character match
                    dp[i][j] = dp[i-1][j-1]
                elif p[j-1] == '*':
                    # Coince : match empty treating * as empty no meaning(left) or match one/more (top)
                    dp[i][j] = dp[i][j-1] or dp[i-1][j]

        return dp[n][m]

```