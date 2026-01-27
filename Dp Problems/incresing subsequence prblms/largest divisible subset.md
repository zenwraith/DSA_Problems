
leetcode link: https://leetcode.com/problems/largest-divisible-subset/

```python

class Solution:
    def largestDivisibleSubset(self, nums: List[int]) -> List[int]:

        if not nums:
            return []
        
        nums.sort()
        n = len(nums)

        dp = [ 1] * n

        parent = [-1] * n # to reconstruct the path

        max_size = 0
        best_idx = 0

        for i in range(n):
            for j in range(i):
                if nums[i] % nums[j] == 0:
                    if dp[i] < dp[j] + 1:
                        dp[i] = dp[j] + 1
                        parent[i] = j
            
            # Track where the overall largest subset ends
            if dp[i] > max_size:
                max_size = dp[i]
                best_idx = i

        # reconstruct the subset using the parent pointers
        result = [] 

        while best_idx != -1:
            result.append(nums[best_idx])
            best_idx = parent[best_idx]

        return result[::-1] # reverse the ascending order
        
``` 