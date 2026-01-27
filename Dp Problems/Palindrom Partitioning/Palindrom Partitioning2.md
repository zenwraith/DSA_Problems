
# **Palindrome Partitioning II**

**LeetCode URL:** [https://leetcode.com/problems/palindrome-partitioning-ii/](https://leetcode.com/problems/palindrome-partitioning-ii/)

## **The Two-Step Strategy**

To solve this efficiently ($O(N^2)$), we break it down:

1. **Step 1 (Interval DP):** Pre-calculate every possible palindrome in the string.
2. **Step 2 (Linear DP):** Find the minimum cuts using the results from Step 1.

## **Pre-calculating Palindromes**

We create a 2D boolean table `isPal[i][j]` which is True if `s[i:j+1]` is a palindrome. This is classic Interval DP.

A string is a palindrome if:

- The characters at the ends are the same: `s[i] == s[j]`
- AND the string inside is a palindrome: `isPal[i+1][j-1]` is True.

## **Finding Minimum Cuts**

Now we use a 1D DP array, `cuts[i]`, which represents the minimum cuts needed for the prefix `s[0...i]`. For a position i, we look at all previous positions j:

- If `s[j...i]` is a palindrome (using our table from Step 1), then:
  $$cuts[i] = \min(cuts[i], cuts[j-1] + 1)$$

### **Solution**

class Solution:
    def minCut(self, s: str) -> int:

        n = len(s)

        # --- Step 1: Interval DP to find all palindromes ---
        isPal = [ [False]*n for i in range(n) ]

        for length in range(1,n+1):
            for i in range(n - length + 1):
                j = i+ length - 1
                if s[i] == s[j]:
                    if length <= 2 or isPal[i+1][j-1]:
                        isPal[i][j] = True
        
        # --- Step 2: Linear DP for min cuts ---
        # dp[i] = min cuts for substring s[0...i]
        dp = [0] * n 

        for i in range(n):
            # Maximum cuts needed for s[0...i] is i (cutting every char)
            min_cuts = i
            for j in range(i+1):
                # If the suffix s[j...i] is a palindrome
                if isPal[j][i]:
                    # If j is 0, the whole string s[0...i] is a palindrome (0 cuts)
                    # Else, it's (cuts for s[0...j-1]) + 1
                    min_cuts = 0 if j == 0 else min( min_cuts, dp[j-1]+1)

            dp[i] = min_cuts

        return dp[n-1]

            





             

````


(Note: This logic of "checking back to find the last valid state" is actually very similar to how we checked back in Stock DP or Longest Increasing Subsequence!)


Master Tip
When you see a problem asking for the "Minimum number of [something]" to partition a string, your first thought should be:

Can I solve this with a 1D DP? 2. Do I need a 2D table to check a property (like Palindrome) quickly?


The "Looking Back" Comparison
Let's look at the logic for three different problems side-by-side. Notice the identical structure:

Problem,The Question at Index i,"The ""Look Back"" Condition",The DP Transition
Longest Increasing Subsequence,What is the longest chain ending here?,Is nums[j] < nums[i]?,"dp[i] = max(dp[i], dp[j] + 1)"
Palindrome Partition II,What is the min cuts for this prefix?,Is s[j...i] a palindrome?,"dp[i] = min(dp[i], dp[j-1] + 1)"
Word Break,Can I form a sentence ending here?,Is s[j...i] a valid word?,dp[i] = dp[j] or (word_exists)


The "Jump" Visualization
Imagine you are standing at index i. You want to reach this point with the minimum cost.

You look back at all previous "platforms" j.

You ask: "Can I jump from platform j to where I am now?"

In LIS, the jump is allowed if the number is smaller.

In Palindrome II, the jump is allowed if the string between j and i is a palindrome.


Summary of the "Checking Back" Family
If you can solve one of these, you can solve them all. They all follow this template:

````
for i in range(n):
    for j in range(i):
        if condition_is_met(j, i):
            dp[i] = optimal(dp[i], dp[j] + current_value)
````

