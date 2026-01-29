# Edit Distance

**LeetCode:** [Edit Distance](https://leetcode.com/problems/edit-distance/)

---

## Problem Statement

Given two strings `word1` and `word2`, return the minimum number of operations required to convert `word1` to `word2`. You have the following three operations permitted on a word:

1. **Insert** a character
2. **Delete** a character
3. **Replace** a character

**Example:**
- `word1 = "horse"`, `word2 = "ros"` → Output: 3
  - horse → rorse (replace 'h' with 'r')
  - rorse → rose (remove 'r')
  - rose → ros (remove 'e')
- `word1 = "intention"`, `word2 = "execution"` → Output: 5

---

## Strategy: 2D DP Table

We use `dp[i][j]` to represent the minimum number of operations to convert the first `i` characters of `word1` to the first `j` characters of `word2`.

### Recurrence Relations

For each position `(i, j)`:

1. **If characters match:** `word1[i-1] == word2[j-1]`
   - No operation needed: `dp[i][j] = dp[i-1][j-1]`

2. **If characters don't match:** `word1[i-1] != word2[j-1]`
   - We try all three operations and pick the minimum:
   
   | Operation | Meaning | Formula |
   |-----------|---------|---------|
   | **Insert** | Insert `word2[j-1]` into `word1` | `dp[i][j-1] + 1` |
   | **Delete** | Delete `word1[i-1]` from `word1` | `dp[i-1][j] + 1` |
   | **Replace** | Replace `word1[i-1]` with `word2[j-1]` | `dp[i-1][j-1] + 1` |
   
   `dp[i][j] = 1 + min(insert, delete, replace)`

### Base Cases

- `dp[0][j] = j` for all `j` (need `j` insertions to create first `j` chars of `word2` from empty string)
- `dp[i][0] = i` for all `i` (need `i` deletions to convert first `i` chars of `word1` to empty string)

---

## Solution

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        n, m = len(word1), len(word2)
        
        # dp[i][j] = min operations to convert first i chars of word1 to first j chars of word2
        dp = [[0] * (m + 1) for _ in range(n + 1)]
        
        # Base case: converting empty string to word2[0:j] requires j insertions
        for j in range(m + 1):
            dp[0][j] = j
        
        # Base case: converting word1[0:i] to empty string requires i deletions
        for i in range(n + 1):
            dp[i][0] = i
        
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if word1[i - 1] == word2[j - 1]:
                    # Characters match - no operation needed
                    dp[i][j] = dp[i - 1][j - 1]
                else:
                    # Try all three operations
                    insert = dp[i][j - 1]      # Insert word2[j-1]
                    delete = dp[i - 1][j]      # Delete word1[i-1]
                    replace = dp[i - 1][j - 1] # Replace word1[i-1] with word2[j-1]
                    dp[i][j] = 1 + min(insert, delete, replace)
        
        return dp[n][m]
```

---

## Why do we use `range(1, n+1)` and `word1[i-1]`?

- `dp[i][j]` represents the edit distance for the first `i` characters of `word1` and first `j` characters of `word2`.
- We use 1-based indexing for the DP table to handle the base cases (empty strings) naturally at `dp[0][...]` and `dp[...][0]`.
- Since the DP table is 1-indexed but the strings are 0-indexed, we access `word1[i-1]` and `word2[j-1]` to get the actual characters.

**Example:**
- When `i = 1`, we're considering the first character of `word1`, which is `word1[0]`.
- When `i = 2`, we're considering the second character of `word1`, which is `word1[1]`.

This indexing simplifies boundary handling by avoiding negative indices.

---

## Understanding the Three Operations

**Why these formulas?**

1. **Insert:** `dp[i][j-1] + 1`
   - We've already converted `word1[0...i-1]` to `word2[0...j-2]`
   - Now insert `word2[j-1]` to match up to `word2[0...j-1]`

2. **Delete:** `dp[i-1][j] + 1`
   - We've already converted `word1[0...i-2]` to `word2[0...j-1]`
   - Now delete `word1[i-1]` and we're done

3. **Replace:** `dp[i-1][j-1] + 1`
   - We've already converted `word1[0...i-2]` to `word2[0...j-2]`
   - Replace `word1[i-1]` with `word2[j-1]` to complete the match

---

## Complexity Analysis

- **Time Complexity:** $O(n \times m)$ where $n$ is the length of `word1` and $m$ is the length of `word2`.
- **Space Complexity:** $O(n \times m)$ for the DP table.