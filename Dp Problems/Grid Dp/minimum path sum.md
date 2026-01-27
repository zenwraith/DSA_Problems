leetcode link : https://leetcode.com/problems/minimum-path-sum/


```python

class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:

        n,m = len(grid) , len(grid[0])

        # Initialize dp array with infinity
        dp = [ float('inf')]*m

        # Base case: The cost to reach the start is just the value of the start cell
        dp[0] = 0

        for r in range(n):
            for c in range(m):
                if c == 0:
                    # In the first column, you can ONLY come from above
                    # so we just add the current cell value to the existing dp[0]
                    dp[c] += grid[r][c]
                else:
                    # You take the minimum of:
                    # dp[c] (the value from above)
                    # dp[c-1] (the value from the left)
                    # And add the current cell's cost
                    dp[c] = grid[r][c] + min(dp[c] , dp[c-1])

        return dp[m-1]
        
``` 
