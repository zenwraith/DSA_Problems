
leetcode link : https://leetcode.com/problems/unique-paths/


```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:

        dp = [[1] * n  for i in range(m)]

        for r in range(1,m):
            for c in range(1,n):

                dp[r][c] = dp[r-1][c] + dp[r][c-1]

        return dp[m-1][n-1]

        
```

```python

class Solution:
    def uniquePaths(self, m: int, n: int) -> int:

        row = [1] * n

        for i in range(m-1):
            new_row = [1]*n

            for j in range(1,n):
                new_row[j] = new_row[j-1] + row[j]
            row = new_row

        return row[-1]



```