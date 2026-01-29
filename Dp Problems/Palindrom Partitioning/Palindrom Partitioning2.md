
# **Palindrome Partitioning II**

**LeetCode URL:** [https://leetcode.com/problems/palindrome-partitioning-ii/](https://leetcode.com/problems/palindrome-partitioning-ii/)

## **The Two-Step Strategy**

To solve this efficiently ($O(N^2)$), we break it down:

1. **Step 1 (Interval DP):** Pre-calculate every possible palindrome in the string.
2. **Step 2 (Linear DP):** Find the minimum cuts using the results from Step 1.

---
## Pre-calculating Palindromes

We create a 2D boolean table `isPal[i][j]` which is True if `s[i:j+1]` is a palindrome. This is classic Interval DP.

A string is a palindrome if:

- The characters at the ends are the same: `s[i] == s[j]`
- AND the string inside is a palindrome: `isPal[i+1][j-1]` is True.

We create a 2D boolean table `isPal[i][j]` which is True if `s[i:j+1]` is a palindrome. This is classic Interval DP.

A string is a palindrome if:

- The characters at the ends are the same: `s[i] == s[j]`
- AND the string inside is a palindrome: `isPal[i+1][j-1]` is True.

---
## Finding Minimum Cuts

Now we use a 1D DP array, `cuts[i]`, which represents the minimum cuts needed for the prefix `s[0...i]`. For a position i, we look at all previous positions j:

- If `s[j...i]` is a palindrome (using our table from Step 1), then:
    $$cuts[i] = \min(cuts[i], cuts[j-1] + 1)$$

Now we use a 1D DP array, `cuts[i]`, which represents the minimum cuts needed for the prefix `s[0...i]`. For a position i, we look at all previous positions j:

- If `s[j...i]` is a palindrome (using our table from Step 1), then:
  $$cuts[i] = \min(cuts[i], cuts[j-1] + 1)$$


```python
   # --- Step 1: Gap-based DP to find all palindromes ---
        # gap is the distance between i and j (j - i)
        for gap in range(n):
            for i in range(n - gap):
                j   = i + gap
                if s[i] == s[j]:
                     # If gap is 0, it's 1 char. If gap is 1, it's 2 chars.
                    # Both are palindromes if characters match.
                    if gap < 2 or isPal[i+1][j-1]:
                        isPal[i][j] = True

```python
# --- Step 1: Gap-based DP to find all palindromes ---
# gap is the distance between i and j (j - i)
for gap in range(n):
    for i in range(n - gap):
        j = i + gap
        if s[i] == s[j]:
            # If gap is 0, it's 1 char. If gap is 1, it's 2 chars.
            # Both are palindromes if characters match.
            if gap < 2 or isPal[i + 1][j - 1]:
                isPal[i][j] = True
```
```
### **Solution**
---
### Solution

```python
class Solution:
    def minCut(self, s: str) -> int:

        n = len(s)

        # --- Step 1: Interval DP to find all palindromes ---
        isPal = [[False] * n for _ in range(n)]

        for length in range(1, n + 1):
            for i in range(n - length + 1):
                j = i + length - 1
                if s[i] == s[j]:
                    if length <= 2 or isPal[i + 1][j - 1]:
                        isPal[i][j] = True

        # --- Step 2: Linear DP for min cuts ---
        # dp[i] = min cuts for substring s[0...i]
        dp = [0] * n

        for i in range(n):
            # Maximum cuts needed for s[0...i] is i (cutting every char)
            min_cuts = i
            for j in range(i + 1):
                # If the suffix s[j...i] is a palindrome
                if isPal[j][i]:
                    # If j is 0, the whole string s[0...i] is a palindrome (0 cuts)
                    # Else, it's (cuts for s[0...j-1]) + 1
                    min_cuts = 0 if j == 0 else min(min_cuts, dp[j - 1] + 1)

            dp[i] = min_cuts

        return dp[n - 1]

```

```python
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

```         





             

```



> **Note:** This logic of "checking back to find the last valid state" is very similar to how we checked back in Stock DP or Longest Increasing Subsequence!

---
## Master Tip

When you see a problem asking for the "Minimum number of [something]" to partition a string, your first thought should be:

1. Can I solve this with a 1D DP?
2. Do I need a 2D table to check a property (like Palindrome) quickly?


Master Tip
When you see a problem asking for the "Minimum number of [something]" to partition a string, your first thought should be:

Can I solve this with a 1D DP? 2. Do I need a 2D table to check a property (like Palindrome) quickly?


The "Looking Back" Comparison
Let's look at the logic for three different problems side-by-side. Notice the identical structure:


| Problem                     | The Question at Index i                | The "Look Back" Condition         | The DP Transition                          |
|-----------------------------|----------------------------------------|-----------------------------------|--------------------------------------------|
| Longest Increasing Subsequence | What is the longest chain ending here? | Is nums[j] < nums[i]?             | dp[i] = max(dp[i], dp[j] + 1)              |
| Palindrome Partition II     | What is the min cuts for this prefix?  | Is s[j...i] a palindrome?         | dp[i] = min(dp[i], dp[j-1] + 1)            |
| Word Break                  | Can I form a sentence ending here?     | Is s[j...i] a valid word?         | dp[i] = dp[j] or (word_exists)             |

| Problem                        | The Question at Index i                | The "Look Back" Condition         | The DP Transition                          |
|-------------------------------|----------------------------------------|-----------------------------------|--------------------------------------------|
| Longest Increasing Subsequence | What is the longest chain ending here? | Is nums[j] < nums[i]?             | dp[i] = max(dp[i], dp[j] + 1)              |
| Palindrome Partition II        | What is the min cuts for this prefix?  | Is s[j...i] a palindrome?         | dp[i] = min(dp[i], dp[j-1] + 1)            |
| Word Break                     | Can I form a sentence ending here?     | Is s[j...i] a valid word?         | dp[i] = dp[j] or (word_exists)             |



---
## The "Jump" Visualization

Imagine you are standing at index $i$. You want to reach this point with the minimum cost.

You look back at all previous "platforms" $j$ and ask: "Can I jump from platform $j$ to where I am now?"

- In LIS, the jump is allowed if the number is smaller.
- In Palindrome II, the jump is allowed if the string between $j$ and $i$ is a palindrome.

---
## Summary of the "Checking Back" Family

If you can solve one of these, you can solve them all. They all follow this template:

```python
for i in range(n):
    for j in range(i):
        if condition_is_met(j, i):
            dp[i] = optimal(dp[i], dp[j] + current_value)
```


Summary of the "Checking Back" Family
If you can solve one of these, you can solve them all. They all follow this template:



```python

for i in range(n):
    for j in range(i):
        if condition_is_met(j, i):
            dp[i] = optimal(dp[i], dp[j] + current_value)

```





---
## Some Whys

### Why `n - length + 1`?

- The outer loop tracks the length of the substring (from 1 up to $n$).
- For a substring of a specific length starting at index $i$, you must ensure you don't run off the end of the string.
- The last possible starting index $i$ is where the substring ends exactly at the last character ($n-1$).
- If a substring starts at $i$ and has a certain length, its last index is $i + \text{length} - 1$.
- To keep that last index within bounds ($< n$), the starting index $i$ must satisfy:
    $$i + \text{length} - 1 \leq n - 1$$
    Simplifying gives: $i \leq n - \text{length}$
- In Python's `range()`, the stop value is exclusive, so to include $n - \text{length}$, we write:
    `range(n - length + 1)`

### Why `j = i + length - 1`?

- This is just standard indexing.
- If $\text{length} = 1$ and $i = 0$, then $j$ should be 0 (the same character):
    $$0 + 1 - 1 = 0$$
    Correct.
- If $\text{length} = 3$ and $i = 0$, then $j$ should be 2 (the 3rd character):
    $$0 + 3 - 1 = 2$$
    Correct.


**Why `n - length + 1`?**

- The outer loop tracks the length of the substring (from 1 up to $n$).
- For a substring of a specific length starting at index $i$, you must ensure you don't run off the end of the string.
- The last possible starting index $i$ is where the substring ends exactly at the last character ($n-1$).
- If a substring starts at $i$ and has a certain length, its last index is $i + \text{length} - 1$.
- To keep that last index within bounds ($< n$), the starting index $i$ must satisfy:
    $$i + \text{length} - 1 \leq n - 1$$
    Simplifying gives: $i \leq n - \text{length}$
- In Python's `range()`, the stop value is exclusive, so to include $n - \text{length}$, we write:
    `range(n - length + 1)`

**Why `j = i + length - 1`?**

- This is just standard indexing.
- If $\text{length} = 1$ and $i = 0$, then $j$ should be 0 (the same character):
    $$0 + 1 - 1 = 0$$
    Correct.
- If $\text{length} = 3$ and $i = 0$, then $j$ should be 2 (the 3rd character):
    $$0 + 3 - 1 = 2$$
    Correct.


---
## Comparing "Length" vs. "Gap"

You mentioned gap. In a "gap-based" DP, we define the distance between the start and end pointers rather than the total count of characters.

| Logic        | Outer Loop                        | Inner Loop                        | End Index (j)         |
|--------------|-----------------------------------|-----------------------------------|-----------------------|
| Length-based | for length in range(1, n+1)       | for i in range(n - length + 1)    | j = i + length - 1    |
| Gap-based    | for gap in range(n)               | for i in range(n - gap)           | j = i + gap           |

---
**Why the Gap version is often preferred:**

- In the gap version, if gap = 0, then j = i + 0, which naturally represents a single character (length 1).
- If gap = 1, then j = i + 1, representing two characters.
- The math is slightly cleaner because you don't have the - 1 and + 1 offsets everywhere.

**Summary:**
- `n - length + 1` ensures your starting index leaves enough room for a substring of that size.
- `j = i + length - 1` translates a "count" (length) into a "coordinate" (index).

| Feature            | Length-Based (Your Code)         | Gap-Based                  |
|--------------------|----------------------------------|----------------------------|
| Variable Concept   | length: Total characters         | gap: Index distance (j−i)  |
| Inner Loop Limit   | n - length + 1                   | n - gap                    |
| Calculating j      | i + length - 1                   | i + gap                    |
| Base Case Check    | length <= 2                      | gap < 2                    |
You mentioned gap. In a "gap-based" DP, we define the distance between the start and end pointers rather than the total count of characters.


| Logic        | Outer Loop                        | Inner Loop                        | End Index (j)         |
|--------------|-----------------------------------|-----------------------------------|-----------------------|
| Length-based | for length in range(1, n+1)       | for i in range(n - length + 1)    | j = i + length - 1    |
| Gap-based    | for gap in range(n)               | for i in range(n - gap)           | j = i + gap           |


Why the Gap version is often preferred: In the gap version, if gap = 0, then j = i + 0, which naturally represents a single character (length 1). If gap = 1, then j = i + 1, representing two characters. The math is slightly cleaner because you don't have the - 1 and + 1 offsets everywhere.

Summary
n - length + 1 ensures your starting index leaves enough room for a substring of that size.

j = i + length - 1 translates a "count" (length) into a "coordinate" (index).



| Feature            | Length-Based (Your Code)         | Gap-Based                  |
|--------------------|----------------------------------|----------------------------|
| Variable Concept   | length: Total characters         | gap: Index distance (j−i)  |
| Inner Loop Limit   | n - length + 1                   | n - gap                    |
| Calculating j      | i + length - 1                   | i + gap                    |
| Base Case Check    | length <= 2                      | gap < 2                    |


Gap Logic: It's mathematically "cleaner." Because $j - i = gap$, the relationship between the pointers is direct. You don't have to worry about the "off-by-one" errors that come with calculating lengths.Length Logic: It's often easier to conceptualize for beginners because we naturally think "First, let's find all strings of length 1, then length 2, etc."


Feature,Length-Based (Your Code),Gap-Based
Variable Concept,length: Total characters in substring,gap: Index distance (j−i)
Inner Loop Limit,n - length + 1,n - gap
Calculating j,i + length - 1,i + gap
Base Case Check,length <= 2,gap < 2


```python

class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        # Add virtual balloons with value 1 at both ends
        nums = [1] + nums + [1]
        n = len(nums)
        
        # dp[i][j] is the max coins in the interval (i, j) exclusive
        dp = [[0] * n for _ in range(n)]

        # 'length' here is the size of the window [i, j]
        # Minimum window length to have a balloon 'k' inside is 3
        for length in range(3, n + 1):
            # i is the start of the window
            for i in range(n - length + 1):
                # j is the end of the window
                j = i + length - 1
                
                # k is the last balloon to burst in the range (i, j)
                for k in range(i + 1, j):
                    coins = nums[i] * nums[k] * nums[j]
                    dp[i][j] = max(dp[i][j], coins + dp[i][k] + dp[k][j])

        return dp[0][n - 1]

```

Key Adjustments Made
Outer Loop (length): We start at 3 because a window needs at least a left boundary i, a right boundary j, and one balloon k in the middle to burst. It goes up to n.

Inner Loop (i): Just like the palindrome problem, the limit is n - length + 1 to ensure j doesn't go out of bounds.

Boundary j: Calculated as i + length - 1.

The "K" Loop: Remains the same—it explores every balloon between the boundaries i and j.

Why the "Gap" version started at 2 but "Length" starts at 3?
This often confuses people! It’s just a difference in "counting" vs "distance":

Gap Logic: gap = j - i. If i=0 and j=2, the gap is 2, but there are 3 numbers involved (nums[0], nums[1], nums[2]).

Length Logic: The count of numbers is 3.


Approach,Outer Loop,j Calculation,Range of k
Gap,"range(2, n)",j = i + gap,"(i+1, j)"
Length,"range(3, n+1)",j = i + length - 1,"(i+1, j)"