
# **Palindrome Partitioning III**

**LeetCode Link:** [https://leetcode.com/problems/palindrome-partitioning-iii/](https://leetcode.com/problems/palindrome-partitioning-iii/)

## **The Three-Step Strategy**

To solve this, we need to answer three questions in order:

1. **Cost to fix:** How many changes are needed to make s[i...j] a palindrome? (Interval DP)
2. **Split logic:** If I split the string at index i into j pieces, what's the best score? (Linear/Multi-stage DP)
3. **The Result:** The answer for the whole string with $k$ pieces.

## **Step 1: Pre-calculate "Cost to Palindrome"**

We create a 2D table `cost[i][j]` that stores the minimum characters we need to change to make s[i...j] a palindrome.

- If `s[i] == s[j]`, the cost is the same as the inner part: `cost[i+1][j-1]`.
- If `s[i] != s[j]`, we must change one of them, so the cost is `1 + cost[i+1][j-1]`.


````
# cost[i][j] calculation
for length in range(2, n + 1):
    for i in range(n - length + 1):
        j = i + length - 1
        cost[i][j] = (0 if s[i] == s[j] else 1) + (cost[i+1][j-1] if i+1 < j else 0)
````


### **Step 2: The Multi-Stage DP**
Now we need to split the string into k pieces.
This is similar to the "Minimum Cuts" problem, but now we have a budget of pieces. Let `dp[i][p]` = the minimum changes to split the prefix s[0...i] into p palindromes.

To find `dp[i][p]`, we try every possible position j to make the last cut. The logic is:

> The best way to make p pieces out of s[0...i] is to take the best way to make p-1 pieces out of s[0...j] and then add one more piece from s[j+1...i].

$$dp[i][p] = \min_{j < i} (dp[j][p-1] + cost[j+1][i])$$

### **Solution**

class Solution:
    def palindromePartition(self, s: str, k: int) -> int:

        n = len(s)

        # --- Step 1: Pre-calculate costs (Interval DP) ---
        cost = [ [0] * n for i in range(n)]

        for length in range(2, n+1):
            for i in range(n- length + 1):
                j = i+ length -1
                cost[i][j] = (0 if s[i] == s[j] else 1) + ( cost[i+1][j-1] if i+1 < j else 0)
        # --- Step 2: Multi-stage DP ---
        # dp[i][p] = min changes for s[:i+1] into p parts
        # Initialize with infinity
        dp  = [ [ float('inf')] * (k+1) for i in range(n) ]

        # Base case: splitting into 1 part is just the cost of the whole prefix
        for i in range(n):
            dp[i][1] = cost[0][i]

        # Fill for p = 2 up to k
        for p in range(2,k+1):
            for i in range(n):
                # Try splitting at every possible j before i
                for j in range(p-2,i):
                    dp[i][p] = min( dp[i][p] , dp[j][p-1] + cost[j+1][i])
        
        return dp[n-1][k]
```

## **Why this is the "Ultimate" Partition Problem**

It uses Interval DP to process the character properties.
It uses Linear DP with an extra state (p) to handle the "number of pieces" constraint.
It follows the "Looking Back" pattern: looking at all previous j to find the best previous state.