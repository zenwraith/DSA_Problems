
leetcode link : 



If you swap the loops (iterating through amount in the outer loop and coins in the inner loop), you will calculate permutations.

Correct (Combinations): [1, 2] and [2, 1] are counted as 1 way.

Incorrect (Permutations): [1, 2] and [2, 1] would be counted as 2 distinct ways.



Combinations (Coin Change II)Goal: $\{1, 2\}$ and $\{2, 1\}$ should be counted as the same thing.To achieve this, we pick a coin and use it as much as we want before ever moving to the next coin. Once we move from the 1-cent coin to the 2-cent coin, we never look back. This prevents us from creating a sequence like $2 + 1$.

```python
# COINS on the outside
for coin in coins:
    for i in range(coin, amount + 1):
        dp[i] += dp[i - coin]
```

How it works: You finish all possible ways to use the first coin, then you move to the second. Because you process coins one by one, you only ever produce "sorted" sequences (e.g., $1+1+2$, never $2+1+1$).

Permutations (Combination Sum IV)Goal: $\{1, 2\}$ and $\{2, 1\}$ are distinct ways.To achieve this, for every single "amount" step, we try every single coin available to us. This allows the coins to appear in any order.


```python
# AMOUNT on the outside
for i in range(1, amount + 1):
    for coin in coins:
        if i >= coin:
            dp[i] += dp[i - coin]
```
How it works: When calculating for amount 3, you ask: "Can I get here by adding a 1 to amount 2? Yes. Can I get here by adding a 2 to amount 1? Yes." This naturally counts both $1+2$ and $2+1$ because you check all coins at every step.

Think of the Combinations approach as building a path where you are only allowed to move forward through your coin tray. You can't reach back for a "1" once you've decided to start using "2s."

In the Permutations approach, you have the entire coin tray in front of you at every single step of the way.




```python

class Solution:
    def change(self, amount: int, coins: List[int]) -> int:

        # dp[i] stores the number of combinations to make amount i
        dp = [0] *(amount +1)

        # Base case: one way to make amount 0
        dp[0] = 1

        # Process each coin one by one to avoid duplicate combinations
        for coin in coins:
            for i in range(coin , amount + 1):
                dp[i] += dp[i-coin]

        return dp[amount]
        
```


 The Combination Trace (Outer Loop: Coins)Here, we pick a coin and update the entire dp table before moving to the next coin.Initial: dp = [1, 0, 0, 0] (1 way to make 0)Pick Coin 1:dp[1] = dp[1] + dp[0] $\rightarrow$ $0 + 1 = 1$ (Way: {1})dp[2] = dp[2] + dp[1] $\rightarrow$ $0 + 1 = 1$ (Way: {1,1})dp[3] = dp[3] + dp[2] $\rightarrow$ $0 + 1 = 1$ (Way: {1,1,1})Current Table: [1, 1, 1, 1]Pick Coin 2:dp[2] = dp[2] + dp[0] $\rightarrow$ $1 + 1 = 2$ (Ways: {1,1}, {2})dp[3] = dp[3] + dp[1] $\rightarrow$ $1 + 1 = 2$ (Ways: {1,1,1}, {1,2})Final Table: [1, 1, 2, 2]Result: 2 combinations ({1,1,1} and {1,2}). Notice we never produced {2,1} because by the time we started using 2s, we were already past the point of adding 1s to them.2. The Permutation Trace (Outer Loop: Amount)Here, for every amount, we try all coins.Initial: dp = [1, 0, 0, 0]Amount 1:Use 1: dp[1] += dp[0] ($0+1=1$)Result: {1}Amount 2:Use 1: dp[2] += dp[1] ($0+1=1$) $\rightarrow$ {1,1}Use 2: dp[2] += dp[0] ($1+1=2$) $\rightarrow$ {1,1}, {2}Amount 3:Use 1: dp[3] += dp[2] ($0+2=2$) $\rightarrow$ {1,1,1}, {2,1}Use 2: dp[3] += dp[1] ($2+1=3$) $\rightarrow$ {1,1,1}, {2,1}, {1,2}Final Table: [1, 1, 2, 3]Result: 3 permutations ({1,1,1}, {2,1}, and {1,2}). Because we checked every coin for Amount 3, we were able to "reach back" and add a 1 to a 2, and a 2 to a 1.



 Forward Loop: The "Infinite Supply" LogicUsed in: Coin Change IIWhen you loop forward, you allow the current item to be used multiple times for the same target.How it works:If you are at dp[i] and you look back at dp[i - coin], you are looking at a value that has already been updated in the current round.The Trace (Coin = 2):dp[0] is True.At i = 2: dp[2] looks at dp[0] $\rightarrow$ dp[2] becomes True.At i = 4: dp[4] looks at dp[2] $\rightarrow$ dp[4] becomes True.Result: You just used the number 2 twice to reach 4. This is "Unbounded."


 Backward Loop: The "Single Use" Logic
Used in: Partition Equal Subset Sum When you loop backward, you ensure that each item is used at most once.

How it works: When you calculate dp[i] by looking at dp[i - num], the value at dp[i - num] hasn't been touched yet in the current iteration. It still contains the result from the previous items only.

The Trace (Num = 2):

dp[0] is True, all others False.

We start from the end (let's say target = 4) and move backward.

At i = 4: dp[4] looks at dp[2]. dp[2] is currently False. dp[4] stays False.

At i = 2: dp[2] looks at dp[0]. dp[0] is True. dp[2] becomes True.

Result: In this round, only dp[2] became True. To make dp[4], you'll have to wait for a different number in a future round.

💡 The "Ghost" of the 2D Table
Think of the 1D array as a "collapsed" 2D table.

In a 2D table, we have dp[item][weight].

To calculate the current row, we usually look at the row above it.

Backward Loop: Simulates looking at the row above by using values that haven't been overwritten yet.

Forward Loop: Simulates looking at the current row, allowing for "chain reactions" (using the same item again).

Loop Direction,Type of Problem,Analogy
"Forward (range(item, target+1))",Unbounded Knapsack,A vending machine (infinite items).
"Backward (range(target, item-1, -1))",0/1 Knapsack,A grocery basket (you only have one of that specific item).


Forward Loop (Unbounded: Use '3' as many times as you want)
Code: for i in range(3, 7): dp[i] = dp[i] or dp[i - 3]

Step,dp Array State,Explanation
Start,"[T, F, F, F, F, F, F]",dp[0] is True (base case).
i = 3,"[T, F, F, T, F, F, F]",dp[3] looks at dp[0] (True). dp[3] becomes True.
i = 4,"[T, F, F, T, F, F, F]",dp[4] looks at dp[1] (False).
i = 5,"[T, F, F, T, F, F, F]",dp[5] looks at dp[2] (False).
i = 6,"[T, F, F, T, F, F, T]",dp[6] looks at dp[3] (True). dp[6] becomes True.

Result: dp[6] is True because we used the '3' to get to 3, and then used that same '3' again to get from 3 to 6.

2. Backward Loop (0/1: Use '3' only ONCE)
Code: for i in range(6, 2, -1): dp[i] = dp[i] or dp[i - 3]

Step,dp Array State,Explanation
Start,"[T, F, F, F, F, F, F]",dp[0] is True.
i = 6,"[T, F, F, F, F, F, F]",dp[6] looks at dp[3] (False).
i = 5,"[T, F, F, F, F, F, F]",dp[5] looks at dp[2] (False).
i = 4,"[T, F, F, F, F, F, F]",dp[4] looks at dp[1] (False).
i = 3,"[T, F, F, T, F, F, F]",dp[3] looks at dp[0] (True). dp[3] becomes True.

Result: dp[6] is still False. Why? Because when we checked dp[6], dp[3] hadn't been updated yet! By the time we updated dp[3], we had already passed the opportunity to use it to update dp[6]. This effectively limits us to using the '3' only once per "round."



1. Coin Change II (Combinations)Pattern: Coins on the Outside, Amount on the Inside.The Logic: "Let's find all possible ways to reach every amount using only 1s. Done? Now let's see how many new ways we can find if we are allowed to use 2s."Why? By fixing the coin on the outside, you ensure you never pick a smaller coin after a larger one. This prevents $\{1, 2\}$ and $\{2, 1\}$ from being counted separately.

```python
for coin in coins: # Fix the coin
    for i in range(coin, target + 1): # Explore amounts
        dp[i] += dp[i - coin]
```

Combination Sum IV (Permutations)Pattern: Amount on the Outside, Coins on the Inside.The Logic: "For amount 1, what coins can I use? For amount 2, what coins can I use? For amount 3, what coins can I use?"Why? Since you check every coin for every amount, you allow coins to appear in any order. At amount 3, you'll find that you can reach it via $(amount=1 + coin=2)$ and $(amount=2 + coin=1)$.


```python
for i in range(1, target + 1): # Fix the amount
    for coin in coins: # Explore all coins for this specific amount
        if i >= coin:
            dp[i] += dp[i - coin]
```

2D String Problems (LCS, Edit Distance)
Pattern: String 1 on the Outside, String 2 on the Inside.

The Logic: "Let's compare the first character of String A against every character in String B. Now let's compare the first two characters of String A against String B."

Why? Here, the order of loops actually doesn't matter for the final result because you are filling a grid where each cell (i, j) depends on (i-1, j) and (i, j-1). However, String 1 is usually the outer loop by convention.

```python
for i in range(1, len(s1) + 1):
    for j in range(1, len(s2) + 1):
        if s1[i-1] == s2[j-1]:
            dp[i][j] = 1 + dp[i-1][j-1]
        else:
            dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```


Problem Type,Outer Loop,Inner Loop,Key Result
0/1 Knapsack,Items,Amount (Backward),Each item used once.
Unbounded Knapsack,Items,Amount (Forward),Each item used infinitely; unique combos.
Permutations,Amount,Items,"Order matters (e.g., {1,2}={2,1})."
Grid / Strings,Row (S1),Column (S2),Relationships between two sequences.

A Quick "Mental Trick"
Ask yourself: "If I change the order of my items, does the answer change?"

If the answer is NO (like in Partition Subset Sum), put the items on the outside.

If the answer is YES (like in Permutations), put the target/amount on the outside.

