
# Regular Expression Matching

**LeetCode:** [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/description/)

---

## Strategy: 2D Grid Matching

We create a DP table where `dp[i][j]` is a boolean representing: "Does the first $i$ characters of $s$ match the first $j$ characters of $p$?"

- **Rows ($n+1$):** String $s$
- **Cols ($m+1$):** Pattern $p$

### The Logic (The Three Rules)

**Rule 1: Direct Match**  
If $p[j-1]$ is the same as $s[i-1]$ or $p[j-1]$ is ".":
$$dp[i][j] = dp[i-1][j-1]$$
(We just inherit the result from the previous diagonal cell).

**Rule 2: The Star * (Zero occurrences)**  
We can treat $x*$ as empty. We look 2 cells back in the pattern:
$$dp[i][j] = dp[i][j-2]$$

**Rule 3: The Star * (One or more occurrences)**  
If the character before the star ($p[j-2]$) matches the current string character $s[i-1]$:
$$dp[i][j] = dp[i-1][j]$$
(This effectively "stays" in the same pattern position while the string moves forward, allowing the star to consume multiple characters).

### Visualizing the * Logic

Imagine $s = "aa"$ and $p = "a*"$.

When we hit the *, we look at $dp[i][j-2]$. This is the "Zero as" case.

Then we see $p[j-2]$ is 'a', which matches $s[i-1]$. We look at $dp[i-1][j]$. This is the "One or more as" case.

Because $dp[i-1][j]$ was already true from the previous a, the truth "flows" down.

### A Concrete Walkthrough

Let's match String: $s = "aa"$ with Pattern: $p = "a*"$

- **Empty vs Empty:** $dp[0][0] = True$
- **Empty string vs "a*":** Since * can mean zero as, we look back two spots to $dp[0][0]$. It's True! So $dp[0][2] = True$. (This means "" matches "a*").
- **String "a" vs Pattern "a*":**
    - Zero case: Look at $dp[1][0]$ (String "a" vs empty pattern). False.
    - More case: Does $s[0]$ ('a') match $p[0]$ ('a')? Yes. So look at $dp[0][2]$ (previous string state with same pattern). It's True!
    - Result: $dp[1][2] = True$.
- **String "aa" vs Pattern "a*":**
    - Zero case: Look at $dp[2][0]$. False.
    - More case: Does $s[1]$ ('a') match $p[0]$ ('a')? Yes. So look at $dp[1][2]$ (the state we just calculated). It's True!
    - Result: $dp[2][2] = True$.

---

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        n, m = len(s), len(p)
        dp = [[False] * (m + 1) for _ in range(n + 1)]
        dp[0][0] = True
        for j in range(2, m + 1):
            if p[j - 1] == '*':
                dp[0][j] = dp[0][j - 2]
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                # Normal match or dot match
                if p[j - 1] == s[i - 1] or p[j - 1] == '.':
                    dp[i][j] = dp[i - 1][j - 1]
                elif p[j - 1] == '*':
                    # Case 1: Treat '*' as zero of the preceding element
                    dp[i][j] = dp[i][j - 2]
                    # Case 2: If preceding element matches current string char
                    if p[j - 2] == s[i - 1] or p[j - 2] == '.':
                        dp[i][j] = dp[i][j] or dp[i - 1][j]
        return dp[n][m]
```