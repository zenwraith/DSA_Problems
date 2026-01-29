
**LeetCode URL:** [https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)

## **Solution**

```python

class Solution:
def minInsertions(self, s: str) -> int:

        def solving_string_dp(s1, s2):
            n,m = len(s1) , len(s2)

            dp  = [ [0]*(m+1) for j in range(n+1) ]

            for i in range(1, n+1):
                for j in range(1, m+1):

                    if s1[i-1] == s2[j-1]:
                        dp[i][j] = 1+ dp[i-1][j-1]
                    else:
                        dp[i][j]  = max(dp[i-1][j] , dp[i][j-1])

            return dp[n][m]

        return len(s) - solving_string_dp(s, s[::-1])

```
# Minimum Insertion Steps to Make a String Palindrome

**LeetCode:** [Minimum Insertion Steps to Make a String Palindrome](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)

---

## Problem Statement

Given a string `s`, return the minimum number of insertions needed to make `s` a palindrome. You can insert characters at any position of the string.

**Example:**
- `s = "zzazz"` → Output: 0 (already a palindrome)
- `s = "mbadm"` → Output: 2 (insert "mb" to make "mbdadbm")
- `s = "leetcode"` → Output: 5 (insert 5 characters to make "eeetcodetoceeteee" or similar)

---

## Strategy: Reduction to LCS

**Key Insight:** The minimum number of insertions needed equals the number of characters that are NOT part of the longest palindromic subsequence.

**Formula:**
```
Minimum Insertions = len(s) - LPS(s)
```

where `LPS(s)` is the Longest Palindromic Subsequence of `s`.

**Why does this work?**
- The longest palindromic subsequence (LPS) represents the maximum number of characters already in palindromic order.
- The remaining characters (not in LPS) need to be "mirrored" by inserting their counterparts.
- LPS can be found using LCS: `LPS(s) = LCS(s, reverse(s))`

**Example:**
- `s = "mbadm"`, `reverse = "mdabm"`
- LCS(s, reverse) = 3 (e.g., "mam" or "mdm")
- Characters in palindrome order: 3
- Characters that need insertions: 5 - 3 = 2

---

## Solution

```python
class Solution:
    def minInsertions(self, s: str) -> int:
        def solving_string_dp(s1, s2):
            n, m = len(s1), len(s2)
            
            # dp[i][j] = LCS length of first i chars of s1 and first j chars of s2
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if s1[i - 1] == s2[j - 1]:
                        # Characters match: extend previous LCS
                        dp[i][j] = 1 + dp[i - 1][j - 1]
                    else:
                        # Characters don't match: take max of excluding one
                        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
            
            return dp[n][m]
        
        # Minimum insertions = Total length - Longest Palindromic Subsequence
        # LPS = LCS(s, reverse(s))
        return len(s) - solving_string_dp(s, s[::-1])
```

---

## Why do we use `range(1, n+1)` and `s1[i-1]`?

- `dp[i][j]` represents the LCS length for the first `i` characters of `s1` and first `j` characters of `s2`.
- We use 1-based indexing for the DP table to handle the base case (empty strings) naturally at `dp[0][...]` and `dp[...][0]`.
- Since the DP table is 1-indexed but the strings are 0-indexed, we access `s1[i-1]` and `s2[j-1]` to get the actual characters.

**Example:**
- When `i = 1`, we're considering the first character of `s1`, which is `s1[0]`.
- When `i = 2`, we're considering the second character of `s1`, which is `s1[1]`.

This indexing simplifies boundary handling by avoiding negative indices.

---

## Complexity Analysis

- **Time Complexity:** $O(n^2)$ where $n$ is the length of the string (since we compute LCS between `s` and its reverse).
- **Space Complexity:** $O(n^2)$ for the DP table.