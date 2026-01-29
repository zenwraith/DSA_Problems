# Best Time to Buy and Sell Stock II

**LeetCode:** [Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

---

## Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day. Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times).

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example:**
- `prices = [7,1,5,3,6,4]` → Output: 7 (Buy on day 2 at 1, sell on day 3 at 5, profit = 4. Buy on day 4 at 3, sell on day 5 at 6, profit = 3. Total = 7)
- `prices = [1,2,3,4,5]` → Output: 4 (Buy on day 1, sell on day 5, profit = 4)

---

## Strategy: State Machine DP (Unlimited Transactions)

We track two states:
- **`hold`**: Maximum profit if we are currently holding a stock
- **`release`**: Maximum profit if we are not holding a stock

### Key Difference from Stock I

In Stock I, we could only buy once: `hold = max(hold, -price)`

In Stock II, we can buy multiple times: `hold = max(hold, release - price)`

The difference is `release - price` instead of `-price`. This means when we buy, we use our current profit from previous transactions.

### Recurrence Relations

For each price:

1. **Update `release` FIRST:**
   - Either continue not holding: `release`
   - Or sell the stock we hold: `hold + price`
   - `release = max(release, hold + price)`

2. **Update `hold` SECOND:**
   - Either continue holding: `hold`
   - Or buy today using previous profit: `release - price`
   - `hold = max(hold, release - price)`

**Important:** We update `release` BEFORE `hold` to avoid using the same day's updated value.

---

## Solution

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # Initial states
        hold = float('-inf')  # Can't hold without buying first
        release = 0           # Starting with $0
        
        for price in prices:
            # Update release first (using old hold value)
            release = max(release, hold + price)
            
            # Update hold second (using NEW release value from this iteration)
            hold = max(hold, release - price)
        
        return release
```

---

## Why Update Order Matters

**Correct Order (release → hold):**
```python
release = max(release, hold + price)  # Uses old hold
hold = max(hold, release - price)      # Uses NEW release
```

This is actually **WRONG** for avoiding same-day transactions! Let me correct this:

**Actually, for Stock II, the order doesn't matter** because we're allowed unlimited transactions. However, if we want to be more explicit and avoid confusion:

```python
old_release = release
release = max(release, hold + price)
hold = max(hold, old_release - price)
```

But since Stock II allows unlimited transactions, the simpler version works fine.

---

## Alternative: Greedy Solution

For Stock II specifically, there's an even simpler greedy approach:

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        for i in range(1, len(prices)):
            # Add profit whenever price increases
            if prices[i] > prices[i-1]:
                profit += prices[i] - prices[i-1]
        return profit
```

**Why this works:** Every upward price movement is an opportunity to profit.

---

## Complexity Analysis

- **Time Complexity:** $O(n)$ where $n$ is the number of days
- **Space Complexity:** $O(1)$ - only two variables