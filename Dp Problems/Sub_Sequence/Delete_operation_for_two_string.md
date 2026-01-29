
# Delete Operation for Two Strings

**LeetCode:** [Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/)

---

## Problem Statement

Given two strings `word1` and `word2`, return the minimum number of steps required to make `word1` and `word2` the same. In one step, you can delete exactly one character in either string.

**Example:**
- `word1 = "sea"`, `word2 = "eat"` → Output: 2 (Delete 's' from "sea" and 't' from "eat")
- `word1 = "leetcode"`, `word2 = "etco"` → Output: 4

---

## Strategy: Reduction to LCS

**Key Insight:** The minimum number of deletions equals the total length minus twice the longest common subsequence.

**Formula:**
```
Min Deletions = len(word1) + len(word2) - 2 * LCS(word1, word2)
```

**Why does this work?**
- The LCS represents the maximum characters that can remain (no deletions needed)
- Characters not in LCS must be deleted from both strings
- From `word1`: delete `len(word1) - LCS` characters
- From `word2`: delete `len(word2) - LCS` characters
- Total: `len(word1) + len(word2) - 2 * LCS`

**Example:**
- `word1 = "sea"`, `word2 = "eat"`
- LCS = "ea" (length 2)
- Deletions = 3 + 3 - 2*2 = 2

---

## Solution – Using LCS

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        def solving_string_dp(s1, s2):
            n, m = len(s1), len(s2)
            
            # dp[i][j] = LCS length of first i chars of s1 and first j chars of s2
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if s1[i - 1] == s2[j - 1]:
                        # Characters match: extend LCS
                        dp[i][j] = 1 + dp[i - 1][j - 1]
                    else:
                        # Take max from excluding one character
                        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
            
            return dp[n][m]
        
        # Apply formula: total length - 2 * LCS
        return len(word1) + len(word2) - 2 * solving_string_dp(word1, word2)
```

---

## Complexity Analysis

- **Time Complexity:** $O(n \times m)$ where $n$ and $m$ are lengths of the two words
- **Space Complexity:** $O(n \times m)$ for the DP table

---

## Space Optimized

We can reduce space complexity to $O(\min(n, m))$ by using two 1D arrays.

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        n, m = len(word1), len(word2)
        
        # Ensure we use smaller dimension for space optimization
        if n < m:
            return self.minDistance(word2, word1)
        
        # prev[j] = LCS for previous row
        prev = [0] * (m + 1)
        
        for i in range(1, n + 1):
            curr = [0] * (m + 1)
            for j in range(1, m + 1):
                if word1[i - 1] == word2[j - 1]:
                    curr[j] = 1 + prev[j - 1]
                else:
                    curr[j] = max(curr[j - 1], prev[j])
            prev = curr
        
        return (n + m) - 2 * prev[m]
```

---

## Complexity Analysis (Space Optimized)

- **Time Complexity:** $O(n \times m)$
- **Space Complexity:** $O(\min(n, m))$ - only two 1D arrays



