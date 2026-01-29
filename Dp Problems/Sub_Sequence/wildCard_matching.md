# Wildcard Matching

**LeetCode:** [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/)

---

## Problem Statement

Given an input string `s` and a pattern `p`, implement wildcard pattern matching with support for `'?'` and `'*'` where:
- `'?'` matches any single character
- `'*'` matches any sequence of characters (including the empty sequence)

**Example:**
- `s = "aa"`, `p = "a"` → Output: false
- `s = "aa"`, `p = "*"` → Output: true
- `s = "cb"`, `p = "?a"` → Output: false
- `s = "adceb"`, `p = "*a*b"` → Output: true

---

## Strategy: 2D DP Table

We use `dp[i][j]` to represent whether the first `i` characters of `s` match the first `j` characters of `p`.

### Recurrence Relations

For each position `(i, j)`:

1. **If pattern has '?' or characters match:** `p[j-1] == '?' or s[i-1] == p[j-1]`
   - Single character match: `dp[i][j] = dp[i-1][j-1]`

2. **If pattern has '*':** `p[j-1] == '*'`
   - `*` can match:
     - **Empty sequence:** `dp[i][j-1]` (ignore the `*`)
     - **One or more characters:** `dp[i-1][j]` (`*` matches `s[i-1]` and remains available)
   - `dp[i][j] = dp[i][j-1] or dp[i-1][j]`

### Base Cases

- `dp[0][0] = True` (empty string matches empty pattern)
- `dp[0][j] = True` if `p[0...j-1]` consists only of `*` (they can all match empty)
- `dp[i][0] = False` for `i > 0` (non-empty string cannot match empty pattern)

---

## Solution

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        n, m = len(s), len(p)
        
        # dp[i][j] = whether first i chars of s match first j chars of p
        dp = [[False] * (m + 1) for _ in range(n + 1)]
        
        # Base case 1: Empty string matches empty pattern
        dp[0][0] = True
        
        # Base case 2: Pattern with only '*' can match empty string
        for j in range(1, m + 1):
            if p[j - 1] == '*':
                dp[0][j] = dp[0][j - 1]
            else:
                # Once we hit a non-*, no further empty matches are possible
                break
        
        # Fill the DP table
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if p[j - 1] == '?' or s[i - 1] == p[j - 1]:
                    # Single character match (diagonal)
                    dp[i][j] = dp[i - 1][j - 1]
                elif p[j - 1] == '*':
                    # '*' matches empty (left) or one/more chars (top)
                    dp[i][j] = dp[i][j - 1] or dp[i - 1][j]
        
        return dp[n][m]
```

---

## Understanding the '*' Pattern

**Why `dp[i][j] = dp[i][j-1] or dp[i-1][j]`?**

1. **`dp[i][j-1]`** (use `*` to match empty)
   - We've matched `s[0...i-1]` with `p[0...j-2]`
   - Use `*` to match nothing, effectively ignoring it

2. **`dp[i-1][j]`** (use `*` to match one or more)
   - We've matched `s[0...i-2]` with `p[0...j-1]`
   - Use `*` to match `s[i-1]`, and `*` remains available for more matches
   - This is the **unbounded knapsack** pattern - same column means the item (`*`) can be reused

**Example:** `s = "adceb"`, `p = "*a*b"`
- First `*` matches "ad"
- "a" matches "c" (not matching, but second `*` will handle it)
- Second `*` matches "ce"
- "b" matches "b"
- Result: true

---

## Why do we use `range(1, n+1)` and `s[i-1]`?

- `dp[i][j]` represents matching for the first `i` characters of `s` and first `j` characters of `p`.
- We use 1-based indexing for the DP table to handle the base case (empty strings) naturally at `dp[0][...]` and `dp[...][0]`.
- Since the DP table is 1-indexed but the strings are 0-indexed, we access `s[i-1]` and `p[j-1]` to get the actual characters.

---

## Complexity Analysis

- **Time Complexity:** $O(n \times m)$ where $n$ is the length of `s` and $m$ is the length of `p`.
- **Space Complexity:** $O(n \times m)$ for the DP table.