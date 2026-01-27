leetcode link : https://leetcode.com/problems/unique-paths-ii

# **Unique Paths II**   

```python

class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:

        m = len(obstacleGrid)
        n = len(obstacleGrid[0])

        if obstacleGrid[0][0] == 1:
            return 0
        
        # dp array representing one row
        dp = [0] *n
        dp[0] = 1 # base case: 1 way to be at the start

        for r in range(m):
            for c in range(n):
                if obstacleGrid[r][c] == 1:
                    dp[c] = 0 # obstacle : 0 ways to reach/pass through
                elif c>0:
                    # ways to reach = ways from above (already in dp[c])
                    # + ways from left ( dp[c-1])

                    dp[c] += dp[c-1] 
        return dp[n-1]
                
        
```


The "In-Place" Update Logic
When we use a single array dp of size n, the value dp[c] plays two roles during the update:

Before the update: dp[c] is the value from the row above (the cell directly on top).

After the update: dp[c] becomes the total ways for the current cell.

The math dp[c] = dp[c] + dp[c-1] literally translates to: Current Cell = Cell Above + Cell to Left


Why we didn't need the first row loop
In the first problem, we initialized the first row to 1 because there were no obstacles. In Unique Paths II, the first row might have an obstacle. If there is a rock at grid[0][3], then dp[3], dp[4], dp[5]... must all become 0 because you can't jump over it.

By using a single loop and the if obstacleGrid[r][c] == 1: dp[c] = 0 logic, the code handles the first row, the first column, and the middle of the grid all with the same logic!

```python

row = [0] * n
row[0] = 1
for r in range(m):
    new_row = [0] * n
    for c in range(n):
        if grid[r][c] == 1:
            new_row[c] = 0
        else:
            # combine current top (row[c]) and current left (new_row[c-1])
            new_row[c] = row[c] + (new_row[c-1] if c > 0 else 0)
    row = new_row
```

