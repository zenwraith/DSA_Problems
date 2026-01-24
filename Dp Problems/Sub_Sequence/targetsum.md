
Leetcode Url : https://leetcode.com/problems/target-sum/


````python

class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:

        n = len(nums)

        @cache
        def helper( index, target):

            
            if index == n:
                return 1 if target ==0 else 0
    

            adding = helper(index+1, target - nums[index]) 
            removing = helper(index+1, target + nums[index])
            

            return adding + removing 
        
        return helper(0, target)

        
````


````python

class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:

        total = sum(nums) 
        
        if (target + total) % 2 != 0 or abs(target) > total:
            return 0

        
        subset_goal = (target + total) // 2
        dp = [0]*(subset_goal+1)
        dp[0] = 1

        # COMBINATION PATTERN: Items on outside
        for num in nums:
            # 0/1 PATTERN: Loop backward to use each num only once
            for i in range(subset_goal, num - 1, -1):
                dp[i] += dp[i-num]

        
        return dp[subset_goal]
        

````