
**LeetCode URL:** [https://leetcode.com/problems/longest-common-subsequence/](https://leetcode.com/problems/longest-common-subsequence/)

## **Solution**

```python

class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        
        n,m = len(text1) , len(text2)

        dp = [ [0] *(m+1) for i in range(n+1)]

        for i in range(1, n+1):
            for j in range(1, m+1):

                if text1[i-1] == text2[j-1]:
                    dp[i][j] = 1 + dp[i-1][j-1]
                else:
                    dp[i][j] = max( dp[i-1][j] , dp[i][j-1])
            
        return dp[n][m]
```
# Longest Common Subsequence

**LeetCode:** [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

---

## Problem Statement

Given two strings `text1` and `text2`, return the length of their longest common subsequence. A subsequence is a sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements.

**Example:**
- `text1 = "abcde"`, `text2 = "ace"` → Output: 3 (The longest common subsequence is "ace")

---

## Strategy: 2D DP Table

We use `dp[i][j]` to represent the length of the longest common subsequence between the first `i` characters of `text1` and the first `j` characters of `text2`.

### Recurrence Relation

For each position `(i, j)`:

1. **If characters match:** `text1[i-1] == text2[j-1]`
   - We can extend the LCS from the previous state: `dp[i][j] = 1 + dp[i-1][j-1]`

2. **If characters don't match:** `text1[i-1] != text2[j-1]`
   - Take the maximum from excluding either character: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

**Base Case:**
- `dp[0][j] = 0` for all `j` (empty `text1` has LCS of 0)
- `dp[i][0] = 0` for all `i` (empty `text2` has LCS of 0)

---

## Solution

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        n, m = len(text1), len(text2)
        
        # dp[i][j] = LCS length of first i chars of text1 and first j chars of text2
        dp = [[0] * (m + 1) for _ in range(n + 1)]
        
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if text1[i - 1] == text2[j - 1]:
                    # Characters match: extend previous LCS
                    dp[i][j] = 1 + dp[i - 1][j - 1]
                else:
                    # Characters don't match: take max of excluding one
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        
        return dp[n][m]
```

---

## Why do we use `range(1, n+1)` and `text1[i-1]`?

- `dp[i][j]` represents the LCS length for the first `i` characters of `text1` and first `j` characters of `text2`.
- We use 1-based indexing for the DP table to handle the base case (empty strings) naturally at `dp[0][...]` and `dp[...][0]`.
- Since the DP table is 1-indexed but the strings are 0-indexed, we access `text1[i-1]` and `text2[j-1]` to get the actual characters.

**Example:**
- When `i = 1`, we're considering the first character of `text1`, which is `text1[0]`.
- When `i = 2`, we're considering the second character of `text1`, which is `text1[1]`.

This indexing simplifies boundary handling by avoiding negative indices.

---

## Complexity Analysis

- **Time Complexity:** $O(n \times m)$ where $n$ is the length of `text1` and $m$ is the length of `text2`.
- **Space Complexity:** $O(n \times m)$ for the DP table.