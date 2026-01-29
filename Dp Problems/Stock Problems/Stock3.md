# Best Time to Buy and Sell Stock III

**LeetCode:** [Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)

---

## Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day. Find the maximum profit you can achieve. You may complete **at most two transactions**.

**Note:** You may not engage in multiple transactions simultaneously (you must sell before buying again).

**Example:**
- `prices = [3,3,5,0,0,3,1,4]` → Output: 6 (Buy on day 4 at 0, sell on day 6 at 3, profit = 3. Buy on day 7 at 1, sell on day 8 at 4, profit = 3. Total = 6)
- `prices = [1,2,3,4,5]` → Output: 4 (Buy on day 1, sell on day 5)

---

## Strategy: State Machine with 2 Transactions

We track four states:
- **`hold1`**: Max profit after first buy
- **`release1`**: Max profit after first sell
- **`hold2`**: Max profit after second buy
- **`release2`**: Max profit after second sell

### State Transitions

```
Start → Buy1 (hold1) → Sell1 (release1) → Buy2 (hold2) → Sell2 (release2)
```

### Recurrence Relations

For each price, we update states in **reverse chronological order** (from transaction 2 to transaction 1):

1. **`release2`** (after 2nd sell): `max(release2, hold2 + price)`
2. **`hold2`** (after 2nd buy): `max(hold2, release1 - price)`
3. **`release1`** (after 1st sell): `max(release1, hold1 + price)`
4. **`hold1`** (after 1st buy): `max(hold1, -price)`

---

## Why Backward Order?

**Key Insight:** Updating in reverse order (from the last transaction to the first) ensures we use the **previous iteration's values**, preventing multiple transactions on the same day.

**Wrong Order (forward):**
```python
hold1 = max(hold1, -price)             # Updates hold1
release1 = max(release1, hold1 + price) # Uses NEW hold1 (wrong!)
```

**Correct Order (backward):**
```python
release2 = max(release2, hold2 + price)   # Uses old hold2
hold2 = max(hold2, release1 - price)      # Uses old release1
release1 = max(release1, hold1 + price)   # Uses old hold1
hold1 = max(hold1, -price)                # Updates hold1 last
```

This way, each transaction uses the profit from the previous **day's** state, not the current day.

---

## Solution

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # Initialize all states
        hold1 = hold2 = float('-inf')  # Can't hold without buying
        release1 = release2 = 0         # Start with $0
        
        for price in prices:
            # Update in REVERSE order: Transaction 2 → Transaction 1
            release2 = max(release2, hold2 + price)
            hold2 = max(hold2, release1 - price)
            release1 = max(release1, hold1 + price)
            hold1 = max(hold1, -price)
        
        return release2
```

---

## Example Walkthrough

**`prices = [1, 2, 4]`**

| Day | Price | hold1 | release1 | hold2 | release2 |
|-----|-------|-------|----------|-------|----------|
| Init | - | -∞ | 0 | -∞ | 0 |
| 1 | 1 | -1 | 0 | -1 | 0 |
| 2 | 2 | -1 | 1 | -1 | 1 |
| 3 | 4 | -1 | 3 | -1 | 3 |

**Result:** 3 (Buy at 1, sell at 4, one transaction is optimal)

---

## Complexity Analysis

- **Time Complexity:** $O(n)$ where $n$ is the number of days
- **Space Complexity:** $O(1)$ - only four variables