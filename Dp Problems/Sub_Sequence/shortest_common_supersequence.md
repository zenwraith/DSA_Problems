
# **Shortest Common Supersequence**

**LeetCode URL:** [https://leetcode.com/problems/shortest-common-supersequence/](https://leetcode.com/problems/shortest-common-supersequence/)

## **Solution**

```python

class Solution:
def shortestCommonSupersequence(self, s1: str, s2: str) -> str:

        n,m  = len(s1) , len(s2)

        # standard lcs prblem
        dp = [ [0] * (m+1) for i in range(n+1)]

        for i in range(1,n+1):
            for j in range(1,m+1):

                if s1[i-1] == s2[j-1]:
                    dp[i][j] = 1 + dp[i-1][j-1]
                else:
                    dp[i][j] = max( dp[i-1][j]  , dp[i][j-1])

        # backtracking to construct string
        res = []
        i,j = n,m  
        # its n , m. which is already len(s1) which is +1 for dp array
        
        while i > 0 and j>0:

            if s1[i-1] == s2[j-1]:

                # character is part of lcs, add it once
                res.append( s1[i-1])
                i-=1
                j-=1
            
            elif dp[i-1][j] > dp[i][j-1]:
                # character is s1 is not in lcs, but must in SCS
                res.append( s1[i-1])
                i-=1
            else:
                res.append(s2[j-1])
                j-=1

        # add remaining characters (if any)
        while i > 0:
            res.append( s1[i-1])
            i-=1
        while j >0:
            res.append(s2[j-1])
            j-=1
        
        return "".join(res[::-1])
```



        


















