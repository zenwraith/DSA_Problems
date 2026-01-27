# **Delete Operation for Two Strings**

**LeetCode URL:** [https://leetcode.com/problems/delete-operation-for-two-strings/](https://leetcode.com/problems/delete-operation-for-two-strings/)

## **Solution - Using LCS**

```python 

class Solution:
    def minDistance(self, word1: str, word2: str) -> int:

        def solving_string_dp(s1, s2):
            n,m = len(s1) , len(s2)


            dp = [[ 0] *(m+1) for j in range(n+1)]

            for i in range(1, n+1):
                for j in range(1, m+1):

                    if s1[i-1] == s2[j-1]:
                        dp[i][j] = 1+ dp[i-1][j-1]
                    else:
                        dp[i][j] = max(dp[i-1][j] , dp[i][j-1]  )

            return dp[n][m]


        return len(word1) + len(word2) - 2 * solving_string_dp(word1, word2)  
        
        
````


space optimized

````python

class Solution:
    def minDistance(self, word1: str, word2: str) -> int:

        n,m = len(word1) , len(word2)

        if n < m: return self.minDistance(word2, word1)

        prev = [ 0 ]*(m+1)

        for i in range(1,n +1):
            curr = [0] *(m+1)
            for j in range(1, m+1):

                if word1[i-1] == word2[j-1]:
                    curr[j] = 1 + prev[j-1]
                else:
                    curr[j] = max( curr[j-1] , prev[j])

            prev = curr

        return (n+m)-2*prev[m]




````
        

