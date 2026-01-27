

leetcode link : https://leetcode.com/problems/maximum-height-by-stacking-cuboids/

```python
class Solution:
    def maxHeight(self, cuboids: List[List[int]]) -> int:


        for c in cuboids:
            c.sort()
        # dp[i] = max height ending with cuboid i at the top
        cuboids.sort()

        n = len(cuboids)

        dp  = [0] *n

        for i in range(n):
            # Base case: the box stands alone
            dp[i] = cuboids[i][2]
            # Look back at all previous boxes
            for j in range(i):
                # Check if cuboid j can fit under cuboid i
            # (Since they are sorted, we check all 3 dimensions)
                if (cuboids[j][0] <= cuboids[i][0] and 
                    cuboids[j][1] <= cuboids[i][1] and 
                    cuboids[j][2] <= cuboids[i][2]):
                    dp[i] = max( dp[i] , dp[j] + cuboids[i][2])

        return max(dp)
        
```