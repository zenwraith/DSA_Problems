
**LeetCode URL:** [https://leetcode.com/problems/longest-palindromic-subsequence/](https://leetcode.com/problems/longest-palindromic-subsequence/)

## **Solution**

```python
class Solution:
    def longestPalindromeSubseq(self, s: str) -> int:

        def solving_string_dp(s1,s2):

            n , m = len(s1) , len(s2)

            dp = [ [0]*(m+1) for i in range(n+1) ]

            for i in range(1,n+1):
                for j in range(1,m+1):

                    if s1[i-1]==s2[j-1]:
                        dp[i][j] = 1 + dp[i-1][j-1]
                    else:
                        dp[i][j]  = max( dp[i-1][j] , dp[i][j-1])

            return dp[n][m]


        return solving_string_dp(s, s[::-1])
```
# Longest Palindromic Subsequence

**LeetCode:** [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)

---

## Problem Statement

Given a string `s`, find the longest palindromic subsequence's length. A subsequence is a sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements.

**Example:**
- `s = "bbbab"` → Output: 4 (The longest palindromic subsequence is "bbbb")
- `s = "cbbd"` → Output: 2 (The longest palindromic subsequence is "bb")

---

## Strategy: Reduction to LCS

**Key Insight:** The longest palindromic subsequence of a string is the same as the longest common subsequence (LCS) between the string and its reverse.

**Why does this work?**
- A palindrome reads the same forwards and backwards.
- If we find characters that are common between `s` and `reverse(s)` in the same order, those characters form a palindromic subsequence.
- The LCS gives us the longest such sequence.

**Algorithm:**
1. Reverse the string: `s2 = s[::-1]`
2. Find the LCS between `s` and `s2`
3. The LCS length is the answer

---

## Solution

```python
class Solution:
    def longestPalindromeSubseq(self, s: str) -> int:
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
        
        # Find LCS between s and its reverse
        return solving_string_dp(s, s[::-1])
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

- **Time Complexity:** $O(n^2)$ where $n$ is the length of the string (since we compare `s` with its reverse of the same length).
- **Space Complexity:** $O(n^2)$ for the DP table.