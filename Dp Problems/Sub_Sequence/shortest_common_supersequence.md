# Shortest Common Supersequence

**LeetCode:** [Shortest Common Supersequence](https://leetcode.com/problems/shortest-common-supersequence/)

---

## Problem Statement

Given two strings `s1` and `s2`, return the shortest string that has both `s1` and `s2` as subsequences. If there are multiple valid strings, return any of them.

**Example:**
- `s1 = "abac"`, `s2 = "cab"` → Output: "cabac"
- `s1 = "aaaaa"`, `s2 = "aaaaa"` → Output: "aaaaa"

---

## Strategy: LCS + Backtracking

**Key Insight:** The shortest common supersequence (SCS) can be constructed by:
1. Finding the LCS of both strings
2. Backtracking through the DP table to construct the SCS by including:
   - Common characters (from LCS) once
   - Non-common characters from both strings

**Formula:**
```
Length of SCS = len(s1) + len(s2) - LCS(s1, s2)
```

**Why?**
- We need all characters from both strings
- Characters in LCS appear in both, so we only need them once
- Total: `len(s1) + len(s2) - LCS` (subtract LCS once to avoid duplication)

---

## Solution

```python
class Solution:
    def shortestCommonSupersequence(self, s1: str, s2: str) -> str:
        n, m = len(s1), len(s2)
        
        # Step 1: Compute LCS table
        dp = [[0] * (m + 1) for _ in range(n + 1)]
        
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if s1[i - 1] == s2[j - 1]:
                    dp[i][j] = 1 + dp[i - 1][j - 1]
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        
        # Step 2: Backtrack to construct SCS
        res = []
        i, j = n, m
        
        while i > 0 and j > 0:
            if s1[i - 1] == s2[j - 1]:
                # Character is part of LCS, add it once
                res.append(s1[i - 1])
                i -= 1
                j -= 1
            elif dp[i - 1][j] > dp[i][j - 1]:
                # Character in s1 is not in LCS, but must be in SCS
                res.append(s1[i - 1])
                i -= 1
            else:
                # Character in s2 is not in LCS, but must be in SCS
                res.append(s2[j - 1])
                j -= 1
        
        # Add remaining characters from s1 (if any)
        while i > 0:
            res.append(s1[i - 1])
            i -= 1
        
        # Add remaining characters from s2 (if any)
        while j > 0:
            res.append(s2[j - 1])
            j -= 1
        
        # Reverse since we built it backwards
        return "".join(res[::-1])
```

---

## Understanding the Backtracking Logic

**Three cases during backtracking:**

1. **Characters match:** `s1[i-1] == s2[j-1]`
   - This character is in the LCS
   - Add it once and move diagonally: `i--, j--`

2. **LCS came from above:** `dp[i-1][j] > dp[i][j-1]`
   - Character `s1[i-1]` is not in LCS
   - Add `s1[i-1]` and move up: `i--`

3. **LCS came from left:** `dp[i][j-1] >= dp[i-1][j]`
   - Character `s2[j-1]` is not in LCS
   - Add `s2[j-1]` and move left: `j--`

**Example:** `s1 = "abac"`, `s2 = "cab"`
- LCS = "ab" or "ca"
- SCS construction merges both strings while including LCS characters once
- Result: "cabac"

---

## Complexity Analysis

- **Time Complexity:** $O(n \\times m)$ for LCS computation + $O(n + m)$ for backtracking = $O(n \\times m)$
- **Space Complexity:** $O(n \\times m)$ for the DP table + $O(n + m)$ for result string



        


















