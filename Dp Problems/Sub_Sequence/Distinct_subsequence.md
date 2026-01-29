# Distinct Subsequences

**LeetCode:** [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)

---

## Problem Statement

Given two strings `s` and `t`, return the number of distinct subsequences of `s` which equals `t`.

**Example:**
- `s = "rabbbit"`, `t = "rabbit"` → Output: 3 (There are 3 ways to form "rabbit" from "rabbbit")
- `s = "babgbag"`, `t = "bag"` → Output: 5

---

## Strategy: 2D DP Table

We use `dp[i][j]` to represent the number of distinct subsequences of `s[0...i-1]` that equal `t[0...j-1]`.

### Recurrence Relations

For each position `(i, j)`:

1. **If characters match:** `s[i-1] == t[j-1]`
   - We can either use this match or skip it
   - `dp[i][j] = dp[i-1][j-1] + dp[i-1][j]`
   - **Use the match:** `dp[i-1][j-1]` (match this character and continue with remaining)
   - **Skip it:** `dp[i-1][j]` (keep looking for matches in remaining s)

2. **If characters don't match:** `s[i-1] != t[j-1]`
   - Must skip `s[i-1]`: `dp[i][j] = dp[i-1][j]`

### Base Case

- `dp[i][0] = 1` for all `i` (empty `t` can be formed in exactly 1 way: select nothing)
- `dp[0][j] = 0` for all `j > 0` (empty `s` cannot form non-empty `t`)

---

## Solution

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        n, m = len(s), len(t)
        
        # dp[i][j] = number of ways to form t[0:j] from s[0:i]
        dp = [[0] * (m + 1) for _ in range(n + 1)]
        
        # Base case: empty t can be formed in 1 way
        for i in range(n + 1):
            dp[i][0] = 1
        
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if s[i-1] == t[j-1]:
                    # Use this match + skip this character
                    dp[i][j] = dp[i-1][j-1] + dp[i-1][j]
                else:
                    # Must skip s[i-1]
                    dp[i][j] = dp[i-1][j]
        
        return dp[n][m]
```

---

## Why do we use `range(1, n+1)` and `s[i-1]`?

- `dp[i][j]` represents the count for the first `i` characters of `s` and first `j` characters of `t`.
- We use 1-based indexing for the DP table to handle the base case (empty strings) naturally at `dp[0][...]` and `dp[...][0]`.
- Since the DP table is 1-indexed but the strings are 0-indexed, we access `s[i-1]` and `t[j-1]` to get the actual characters.

---

## Complexity Analysis (2D DP)

- **Time Complexity:** $O(n \times m)$ where $n$ is the length of `s` and $m$ is the length of `t`.
- **Space Complexity:** $O(n \times m)$ for the DP table.

---

## Space Optimized Approach - Backward Loop

### Key Concept: 0/1 Knapsack Logic

We can optimize space to $O(m)$ by using a 1D array and iterating backward.

**Why backward iteration?**
- When updating `dp[j]`, we need the old value of `dp[j-1]` (from the previous row)
- If we iterate forward, we'd overwrite `dp[j-1]` before using it for `dp[j]`
- By iterating backward, we preserve the old values until we need them

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        n, m = len(s), len(t)
        
        # dp[j] = number of ways to form t[0:j] from current prefix of s
        dp = [0] * (m + 1)
        
        # Base case: empty t can be formed in 1 way
        dp[0] = 1
        
        # Process each character of s
        for char_s in s:
            # Iterate backward to avoid overwriting values we need
            for j in range(m, 0, -1):
                if char_s == t[j-1]:
                    # Add ways using this match to existing ways without it
                    dp[j] += dp[j-1]
        
        return dp[m]
```

---

## Why Backward Iteration is Essential

**Example:** `s = "rabbit"`, `t = "ra"`

If we iterated forward:
- Update `dp[1]` first, which overwrites the old `dp[1]`
- When we update `dp[2]`, we'd use the new `dp[1]` instead of the old one
- This would give incorrect results!

By iterating backward:
- Update `dp[2]` first using old `dp[1]`
- Then update `dp[1]`
- This preserves the "previous row" semantics

---

## Complexity Analysis (Space Optimized)

- **Time Complexity:** $O(n \times m)$ - same as 2D solution
- **Space Complexity:** $O(m)$ - only one 1D array



Loop Direction,Equivalent to...,Result
Forward (0→m),Unbounded Knapsack,Uses the same item multiple times.
Backward (m→0),0/1 Knapsack,Uses each item exactly once.


The Interview Tip
Whenever you are doing Space Optimization in DP:If the value depends on the diagonal ($dp[i-1][j-1]$), 
you must loop backward or use a temporary variable to save the old diagonal value.
If the value only depends on the top and left (like in a basic Grid Path problem),
the loop direction depends on which side you want to "carry over."


Forward = Flow: The update "flows" into the next calculation in the same row. (Use again and again).
Backward = Barrier: You calculate the end before you update the beginning, creating a barrier between the item and itself. (Use only once).



#### The Setup
Your String (The "Store"): S = "A" (You have one letter 'A' to use).

The Target (The "Goal"): T = "AA" (You want to see how many ways you can make "AA").

The Rule: You can only use each letter from the "Store" once.


#### Scenario A: The Backward Loop (The Correct Way - "0/1 Logic")
Imagine your dp array is a row of boxes on a table: [1, 0, 0] (representing: ways to make "", "A", and "AA").

If we go Backward, we look at the goal ("AA") before we look at the middle ("A").

Check "AA": To make "AA", do I have a match? Yes, the 'A' from the store matches. But do I have another 'A' already? I look to my left to see if "A" was already made. In the old data, "A" has 0 ways. So, "AA" stays 0.

Check "A": Now I move left. To make "A", do I have a match? Yes. I look to my left to see if "" (empty) was made. Yes, there is 1 way. So "A" becomes 1.

Final Array: [1, 1, 0]. Result: 0 ways to make "AA". (Correct! Because you only had one 'A' in the store).



# Distinct Subsequences

**LeetCode:** [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)

---

## Solution

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        n, m = len(s), len(t)
        dp = [[0] * (m + 1) for _ in range(n + 1)]
        for i in range(n + 1):
            dp[i][0] = 1
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if s[i - 1] == t[j - 1]:
                    # sum of (using the match) + (skipping this s[i-1])
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]
                else:
                    # must skip s[i-1]
                    dp[i][j] = dp[i - 1][j]
        return dp[n][m]
```

## Space Optimized Approach – Backward Loop

### Key Concept: 0/1 Knapsack Logic

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        n, m = len(s), len(t)
        dp = [0] * (m + 1)
        dp[0] = 1
        for char_s in s:
            for j in range(m, 0, -1):
                if char_s == t[j - 1]:
                    dp[j] += dp[j - 1]
        return dp[m]
```

---

| Loop Direction      | Equivalent to         | Result                          |
|---------------------|----------------------|----------------------------------|
| Forward ($0\to m$)  | Unbounded Knapsack   | Uses the same item multiple times|
| Backward ($m\to 0$) | 0/1 Knapsack         | Uses each item exactly once      |

---

### The Interview Tip
Whenever you are doing Space Optimization in DP:
- If the value depends on the diagonal ($dp[i-1][j-1]$), you must loop backward or use a temporary variable to save the old diagonal value.
- If the value only depends on the top and left (like in a basic Grid Path problem), the loop direction depends on which side you want to "carry over."

**Forward = Flow:** The update "flows" into the next calculation in the same row. (Use again and again).

**Backward = Barrier:** You calculate the end before you update the beginning, creating a barrier between the item and itself. (Use only once).

---

#### The Setup
Your String (The "Store"): $S = "A"$ (You have one letter 'A' to use).

The Target (The "Goal"): $T = "AA"$ (You want to see how many ways you can make "AA").

The Rule: You can only use each letter from the "Store" once.

#### Scenario A: The Backward Loop (The Correct Way – "0/1 Logic")
Imagine your dp array is a row of boxes on a table: `[1, 0, 0]` (representing: ways to make "", "A", and "AA").

If we go Backward, we look at the goal ("AA") before we look at the middle ("A").

Check "AA": To make "AA", do I have a match? Yes, the 'A' from the store matches. But do I have another 'A' already? I look to my left to see if "A" was already made. In the old data, "A" has 0 ways. So, "AA" stays 0.

Check "A": Now I move left. To make "A", do I have a match? Yes. I look to my left to see if "" (empty) was made. Yes, there is 1 way. So "A" becomes 1.

Final Array: `[1, 1, 0]`. Result: 0 ways to make "AA". (Correct! Because you only had one 'A' in the store).
#### Scenario B: The Forward Loop (The "Infinite" Mistake - "Unbounded Logic")

Now, let’s see what happens if we move Forward.

Check "A": Does 'A' match? Yes. Look to the left (""). There is 1 way. So "A" becomes 1.

Check "AA": Does 'A' match? Yes. Look to the left ("A"). Wait! Because we are moving forward, we just updated "A" to 1. The code sees that "A" is finished, so it uses the 'A' from the store again to finish "AA". "AA" becomes 1.

Final Array: [1, 1, 1]. Result: 1 way to make "AA". (Wrong! The code thinks you have infinite 'A's).


1. The 0/1 Knapsack (Use items ONLY ONCE)
   This is for problems like Subset Sum or Distinct Subsequences.

```
# S = "A", T = "AA"
dp = [1, 0, 0]  # [Ways to make "", "A", "AA"]

for char in "A": # We only have one 'A'
    # GO BACKWARDS: j starts at 2, then goes to 1
    for j in range(2, 0, -1): 
        if char == "A":
            dp[j] = dp[j] + dp[j-1]
            # When j=2: dp[2] = dp[2] + dp[1] -> 0 + 0 = 0
            # When j=1: dp[1] = dp[1] + dp[0] -> 0 + 1 = 1
```

The result is 0 for "AA" because the update "stays behind" the loop.

2. The Unbounded Knapsack (Use items INFINITELY)
   This is for problems like Coin Change (where you have unlimited coins).

```

# S = "A", T = "AA"
dp = [1, 0, 0]

for char in "A":
    # GO FORWARD: j starts at 1, then goes to 2
    for j in range(1, 3): 
        if char == "A":
            dp[j] = dp[j] + dp[j-1]
            # When j=1: dp[1] = dp[1] + dp[0] -> 0 + 1 = 1
            # When j=2: dp[2] = dp[2] + dp[1] -> 0 + 1 = 1 (Uses the new dp[1]!)
            
 ```

The result is 1 for "AA" because the update "flows forward" into the next calculation.



#### Simple Rule of Thumb for your Interview:

Am I allowed to use the same character/item/coin more than once?

Yes $\rightarrow$ Loop Forward.

No $\rightarrow$ Loop Backward.



The short answer: We loop through S first because S is our "pool" of items, and T is our "goal."

By putting S in the outer loop, we are saying:
"I am going to pick up one character from S, see how it changes my progress toward the goal T, and then move on to the next character, never looking back."


1. Think of it like a "Build-a-Subsequence" Game
   Imagine you are building the word "CAT" (this is T) using a pile of letter-tiles: "C", "A", "T", "T" (this is S).

Outer Loop (S): You pick up the first tile from your pile.

Inner Loop (T): You walk through your goal "C-A-T" and ask: "Can I use this tile I just picked up to help finish this word?"

Once you finish the inner loop, you throw that tile away (or set it aside). You then pick up the next tile from the pile. This is why we go through S one character at a time.


2. What if we swapped them? (Loop T first, then S)
   If you put the goal T in the outer loop and the store S in the inner loop, the logic breaks for subsequences.

If you loop T first: You are saying: "I want to find the first letter 'C'. Let me look through all of 'S' to find it." Then, "I want to find 'A'. Let me look through all of 'S' to find it."

The Problem: In a subsequence, order matters. If you look through all of S for 'C' and then look through all of `S' again for 'A', you might accidentally pick an 'A' that appeared before the 'C' in the original string!

By looping through S (the source) first, you naturally respect the order of the characters as they appear in the string.


3. The "Store and Customer" Analogy
   Think of S as a Store and T as a Customer's Shopping List.

Correct (S first): The store owner brings out items one by one from the back warehouse (S). For each item brought out, the customer checks their list (T) to see if they need it.

Incorrect (T first): The customer looks at the first item on their list and searches the entire warehouse. Then they look at the second item and search the entire warehouse again. (This doesn't work for subsequences because the customer needs to find items in the order they appear on the shelf).


Loop Order,What it means,Used for...
"Outer: S, Inner: T","""One character at a time, what can I build?""","Subsequences, Knapsack, Combinations."
"Outer: T, Inner: S","""For this goal, which items can I use?""",Permutations (where order doesn't matter or is handled differently).



This is one of the "Final Boss" questions of DP loop logic. Swapping the loop order doesn't just change the code; it fundamentally changes the question you are asking.

To understand this, let's use the Coin Change II problem (counting ways to make change).

Target: 3

Coins: [1, 2]


Case 1: Outer = Coins, Inner = Amount (Combinations)
When the coin is on the outside, you are saying: "I will finish using all my 1s before I even look at my 2s."

```python

for coin in [1, 2]:
   for j in range(coin, target + 1):
      dp[j] += dp[j - coin]

```

What happens?

Process all 1s: The DP finds it can make 3 using {1, 1, 1}.

Process all 2s: The DP finds it can make 3 by adding a 2 to a previously made 1. It finds {1, 2}.

Result: You get 2 ways: {1, 1, 1} and {1, 2}. Note that {2, 1} is impossible to find because by the time you are looking at 2, 
the 1s are already "behind" you. You can only add a 2 to an existing 1, not a 1 to an existing 2.


Case 2: Outer = Amount, Inner = Coins (Permutations)
When the amount is on the outside, you are saying: "For every single cent, I am willing to try any coin in any order."

````python
for j in range(1, target + 1):
    for coin in [1, 2]:
        if j >= coin:
            dp[j] += dp[j - coin]
````

What happens?

At j=1: Only coin 1 works. Result: {1}.

At j=2: * Use coin 1: Add it to dp[1]. Result: {1, 1}.

Use coin 2: Add it to dp[0]. Result: {2}.

At j=3: * Use coin 1: Add it to dp[2]. Since dp[2] has {1,1} and {2}, we get: {1,1,1} and {2,1}.

Use coin 2: Add it to dp[1]. Since dp[1] has {1}, we get: {1,2}.

Result: You get 3 ways: {1, 1, 1}, {2, 1}, and {1, 2}. Because the coins are in the inner loop, the code is allowed to pick a 2 and then pick a 1 later.


Loop Order,Logical Meaning,Result Type,"Example {1,2} vs {2,1}"
Outer: Items,"""Finish one item before moving to next.""",Combinations,Same thing (counts as 1)
Outer: Target,"""At every step, anything is possible.""",Permutations,Different things (counts as 2)

Interviewers love to ask this because it tests if you understand "State Dependency."

In Combinations, the state only depends on items seen so far (Order-Independent).

In Permutations, the state depends on the sum itself regardless of which item was picked last (Order-Dependent).



Problem 1: Combination Sum IVThe Goal: Given an array of distinct integers nums and a target, return the number of possible combinations that add
up to target.The "Catch": Different sequences are counted as unique (e.g., {1, 1, 2} and {1, 2, 1} are both counted).The Decision MatrixIs it Unbounded? Yes, you can use the same number as many times as you want.
$\rightarrow$ Loop Forward.Is it Permutations or Combinations? The problem says {1, 2, 1} is different from {1, 1, 2}. $\rightarrow$ Outer = Target, Inner = Nums.


```python

def combinationSum4(nums, target):
    dp = [0] * (target + 1)
    dp[0] = 1 # One way to make zero (empty set)
    
    # PERMUTATION PATTERN: Target on outside
    for i in range(1, target + 1):
        # UNBOUNDED PATTERN: We look at all numbers for every i
        for num in nums:
            if i >= num:
                dp[i] += dp[i - num]
    return dp[target]

```


Problem 2: Target SumThe Goal: You are given an array nums and a target. You want to build an expression by adding a + or - before each integer. 
Return the number of ways to reach target.The "Aha!" MomentThis is actually a math problem.
If we assign + to some numbers (subset $P$) and - to others (subset $N$), we get:$Sum(P) - Sum(N) = \text{target}$Since we know $Sum(P) + Sum(N) = \text{TotalSum}$, we can derive:$2 \cdot Sum(P) = \text{target} + \text{TotalSum}$So the problem becomes: Find the number of ways to form a subset that sums up to $Sum(P)$.The Decision MatrixIs it Unbounded? No! You can only use each number in the array exactly once. $\rightarrow$ Loop Backward.Is it Permutations or Combinations? Picking the numbers {1, 2} is the same as {2, 1} for a subset. $\rightarrow$ Outer = Nums, Inner = Target.


```python

def findTargetSumWays(nums, target):
    total = sum(nums)
    # Math check: target must be reachable
    if (target + total) % 2 != 0 or abs(target) > total:
        return 0
    
    subset_goal = (target + total) // 2
    dp = [0] * (subset_goal + 1)
    dp[0] = 1
    
    # COMBINATION PATTERN: Items on outside
    for num in nums:
        # 0/1 PATTERN: Loop backward to use each num only once
        for i in range(subset_goal, num - 1, -1):
            dp[i] += dp[i - num]
            
    return dp[subset_goal]
```

Why This Clears InterviewsIf you can explain why Combination Sum IV uses a forward loop with Target on the outside, while Target Sum uses a backward loop with Nums on the outside, you have effectively proven that you understand $O(N)$ Space DP optimization better than 90% of candidates.

A Simple Test for Your InterviewIf an interviewer asks:
"Does {1, 2} count as one way or two ways?"One way? (Order doesn't matter) $\rightarrow$ Use Combinations Pattern (Outer = Nums).
Two ways? (Order matters) $\rightarrow$ Use Permutations Pattern (Outer = Target).


Problem 2: Target SumThe Goal: You are given an array nums and a target. 
You want to build an expression by adding a + or - before each integer. Return the number of ways to reach target.
The "Aha!" MomentThis is actually a math problem. If we assign + to some numbers (subset $P$) and - to others (subset $N$), we get:$Sum(P) - Sum(N) = \text{target}$Since we know $Sum(P) + Sum(N) = \text{TotalSum}$, we can derive:$2 \cdot Sum(P) = \text{target} + \text{TotalSum}$So the problem becomes: Find the number of ways to form a subset that sums up to $Sum(P)$.
The Decision MatrixIs it Unbounded? No! You can only use each number in the array exactly once. $\rightarrow$ Loop Backward.Is it Permutations or Combinations? Picking the numbers {1, 2} is the same as {2, 1} for a subset. $\rightarrow$ Outer = Nums, Inner = Target.


`````python

class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:

        total = sum(nums) 
        
        if (target + total) % 2 != 0 or abs(target) > total:
            return 0

        
        subset_goal = (target + total) // 2
        dp = [0]*(subset_goal+1)
        dp[0] = 1

        # COMBINATION PATTERN: Items on outside
        for num in nums:
            # 0/1 PATTERN: Loop backward to use each num only once
            for i in range(subset_goal, num - 1, -1):
                dp[i] += dp[i-num]

        
        return dp[subset_goal]
        
`````

Problem,Goal,Outer Loop,Inner Loop,Direction
Combination Sum IV,Permutations,Target,Nums,Forward
Target Sum,Combinations,Nums,Target,Backward

