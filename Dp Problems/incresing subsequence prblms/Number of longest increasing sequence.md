
leetcode link : https://leetcode.com/problems/number-of-longest-increasing-subsequence

```python

class Solution:
    def findNumberOfLIS(self, nums: List[int]) -> int:

        if not nums:
            return 0
        n = len(nums)

        lengths = [1]*n
        counts = [1]*n

        for i in range(n):
            for j in range(i):
                if nums[i] > nums[j]:
                    if lengths[j] + 1 > lengths[i]:

                        lengths[i] = lengths[j] + 1
                        counts[i] = counts[j]
                    elif lengths[j] +1 == lengths[i]:

                        counts[i] += counts[j]

        max_len = max(lengths)

        return sum(c for l,c in zip( lengths, counts) if l == max_len)

        
```

 The LogicAs we look back at $j < i$ where $nums[i] > nums[j]$, we have three scenarios:Scenario A: We found a longer path.If length[j] + 1 > length[i], it means we just found a new "best" length for $i$. We update length[i] and inherit the count: count[i] = count[j].Scenario B: We found an alternative path of the same "best" length.If length[j] + 1 == length[i], it means we found another way to reach the current maximum length. We add the counts together: count[i] += count[j].Scenario C: If length[j] + 1 < length[i], we ignore it; it's not contributing to the longest subsequence.