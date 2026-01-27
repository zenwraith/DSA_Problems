


leetcode link : https://leetcode.com/problems/longest-increasing-subsequence/description/


A Walkthrough: [10, 9, 2, 5, 3, 7, 101, 18]

Num,Action,tails state,Reason
10,Append,[10],First element.
9,Replace,[9],9 is better than 10 (easier to build on).
2,Replace,[2],2 is better than 9.
5,Append,"[2, 5]",5 is larger than 2.
3,Replace,"[2, 3]",3 is better than 5 (idx 1).
7,Append,"[2, 3, 7]",7 is larger than 3.
101,Append,"[2, 3, 7, 101]",101 is larger than 7.
18,Replace,"[2, 3, 7, 18]",18 is better than 101.

```python

class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:

        if not nums:
            return 0

        dp = [1] * len(nums)

        for i in range(len(nums)):
            for j in range(i):

                if nums[i] > nums[j]:

                    dp[i] = max(dp[i] , dp[j] +1)

        return max(dp)
        
```

```python

class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:

        if not nums:
            return 0

        tails = []

        for x in nums:

            idx = bisect.bisect_left(tails, x)

            if idx == len(tails):
                tails.append(x)
            else:
                tails[idx] = x
                
        return len(tails)
```