# Wildcard Matching

**LeetCode:** [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/)

---

## Solution

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        n, m = len(s), len(p)
        # Base 1: Initialize DP table with False
        dp = [[False] * (m + 1) for _ in range(n + 1)]
        # Base 2: Empty string matches empty pattern
        dp[0][0] = True
        # Base 3: Pattern has '*' that can match empty string
        for j in range(1, m + 1):
            if p[j - 1] == '*':
                dp[0][j] = dp[0][j - 1]
            else:
                # Once we hit a non-*, no further empty matches are possible
                break
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if p[j - 1] == '?' or s[i - 1] == p[j - 1]:
                    # Diagonal: Single character match
                    dp[i][j] = dp[i - 1][j - 1]
                elif p[j - 1] == '*':
                    # Coincide: match empty (left) or match one/more (top)
                    dp[i][j] = dp[i][j - 1] or dp[i - 1][j]
        return dp[n][m]
```