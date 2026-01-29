
# **Palindrome Partitioning IV**

**LeetCode Link:** [https://leetcode.com/problems/palindrome-partitioning-iv/](https://leetcode.com/problems/palindrome-partitioning-iv/)

```python

class Solution:
    def checkPartitioning(self, s: str) -> bool:

        n = len(s)
        # --- Step 1: Interval DP (Same as always!) ---
        isPal = [ [False] * n for i in range(n)]

        for length in range(1, n+1):
            for i in range(n - length +1):

                j = i+ length -1
                if s[i] == s[j]:
                    if length <= 2 or isPal[i+1][j-1]:
                        isPal[i][j] = True

    # --- Step 2: Try every possible pair of cuts ---
    # i is the end of the first part
    # j is the end of the second part
        for i in range( n -2):
            for j in range(i+1,n-1):
                # Check if all three parts are palindromes
                # Part 1: 0 to i
                # Part 2: i+1 to j
                # Part 3: j+1 to n-1
                if isPal[0][i] and isPal[i+1][j] and isPal[j+1][n-1]:
                    return True

        return False
        
```