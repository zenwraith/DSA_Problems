

# Best Time to Buy and Sell Stock I

**LeetCode:** [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

---

## Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day. You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock. Return the maximum profit you can achieve. If you cannot achieve any profit, return 0.

**Example:**
- `prices = [7,1,5,3,6,4]` → Output: 5 (Buy on day 2 at price 1, sell on day 5 at price 6, profit = 6-1 = 5)
- `prices = [7,6,4,3,1]` → Output: 0 (No transactions are done, max profit = 0)

**Constraint:** You can only hold at most one share of the stock at any time.

---

## Strategy: State Machine DP

We track two states as we iterate through prices:
- **`hold`**: Maximum profit if we are currently holding a stock
- **`release`**: Maximum profit if we are not holding a stock

### Recurrence Relations

For each price:

1. **Update `release` (not holding)**:
   - Either continue not holding (no change): `release`
   - Or sell the stock we hold: `hold + price`
   - `release = max(release, hold + price)`

2. **Update `hold` (holding stock)**:
   - Either continue holding (no change): `hold`
   - Or buy today's stock: `-price` (we spend money, so balance becomes negative)
   - `hold = max(hold, -price)`

**Key insight:** Since we can only do ONE transaction, buying costs `-price` (not `release - price`).

---

## Solution

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # Initial state: no stock held, starting with $0
        hold = -float('inf')  # Impossible to hold without buying first
        release = 0            # Starting cash
        
        for price in prices:
            # Update release: max of (doing nothing) or (selling the stock we hold)
            release = max(release, hold + price)
            
            # Update hold: max of (doing nothing) or (buying today's stock)
            # Note: -price because we spend money to buy (only one transaction allowed)
            hold = max(hold, -price)
        
        return release
```

---

## Why Use `max` with Negative Numbers?

Since you start with \\$0, when you buy a stock for `price`, your balance becomes `-price`.

**Intuition:**
- Stock A costs \\$10 → balance = -\\$10
- Stock B costs \\$2 → balance = -\\$2
- Mathematically: $-2 > -10$, so `max(-10, -2) = -2`

By using `max`, you automatically pick the price that leaves you with the most money (the least debt).

**Example:** `prices = [10, 2, 5, 8]`

| Day | Price | `hold` Calculation | `hold` | `release` Calculation | `release` |
|-----|-------|-------------------|--------|----------------------|-----------|
| 1 | 10 | max(-∞, -10) | -10 | max(0, -10+10) | 0 |
| 2 | 2 | max(-10, -2) | -2 | max(0, -2+2) | 0 |
| 3 | 5 | max(-2, -5) | -2 | max(0, -2+5) | 3 |
| 4 | 8 | max(-2, -8) | -2 | max(3, -2+8) | 6 |

**Result:** Maximum profit = 6

---

## The "Holding" Rule

The variable `hold` asks: **"What is the best financial state if I am currently forced to own a stock?"**

The answer: **The state where I spent the least amount of money to get it.**

By tracking `hold = max(hold, -price)`, we're always finding the cheapest day to buy, which maximizes our potential profit when we sell.

---

## Complexity Analysis

- **Time Complexity:** $O(n)$ where $n$ is the number of days
- **Space Complexity:** $O(1)$ - only two variables