# **Best Time to Buy and Sell Stock with Transaction Fee**

**LeetCode Link:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

---

## **Problem Statement**

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i`-th day, and an integer `fee` representing a transaction fee.

Find the maximum profit you can achieve. You may complete as many transactions as you like, but you need to pay the transaction fee for each transaction.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example:**
```
Input: prices = [1,3,2,8,4,9], fee = 2
Output: 8
Explanation: Maximum profit = ((8 - 1) - 2) + ((9 - 4) - 2) = 5 + 3 = 8
```

---

## **Strategy: Two-State DP with Fee Deduction**

This problem is similar to **Stock II** (unlimited transactions), but we must pay a fee for each transaction.

### **State Definition**

- **`hold`**: Maximum profit when currently holding a stock
- **`release`**: Maximum profit when not holding a stock

### **State Transitions**

```
release = max(release, hold + price - fee)  # Sell and pay fee
hold = max(hold, release - price)           # Buy
```

**Key Decision:** When to subtract the fee?

**Option 1:** Subtract fee when selling
```python
release = max(release, hold + price - fee)
```

**Option 2:** Subtract fee when buying
```python
hold = max(hold, release - price - fee)
```

Both work! We choose **Option 1** (subtract on sell) because it's more intuitive: you pay the fee when completing the transaction.

---

## **Problem Variants Comparison**

| Problem | Transaction Rule | Main DP Transition |
|---------|-----------------|-------------------|
| Stock I | Only 1 trade | `hold = max(hold, -price)` |
| Stock II | Infinite trades | `hold = max(hold, cash - price)` |
| Stock III/IV | Exactly K trades | `hold[k] = max(hold[k], release[k-1] - price)` |
| Cooldown | 1-day wait | `hold = max(hold, rest - price)` |
| Fee | Pay per trade | `cash = max(cash, hold + price - fee)` |

---

## **Solution**

```python
class Solution:
    def maxProfit(self, prices: List[int], fee: int) -> int:
        # Initialize states
        # hold: max profit when holding a stock (start by buying first stock)
        # release: max profit when not holding
        hold = -prices[0]  # Buy on day 0
        release = 0
        
        for i in range(1, len(prices)):
            # Update release: stay released OR sell stock (pay fee)
            release = max(release, hold + prices[i] - fee)
            
            # Update hold: stay holding OR buy stock
            hold = max(hold, release - prices[i])
        
        return release
```

---

## **Why Initialize `hold = -prices[0]`?**

We assume we **buy the stock on day 0**. This simplifies the logic:
- On day 1, we can either:
  - Sell the stock we bought on day 0: `release = hold + prices[1] - fee`
  - Or keep holding: `hold = max(-prices[0], release - prices[1])`

Alternatively, we could start with `hold = -inf`, but then we'd need to handle the first buy separately.

---

## **Example Walkthrough**

**Input:** `prices = [1, 3, 2, 8, 4, 9]`, `fee = 2`

### **Initial State**
```
hold = -1 (bought at price 1)
release = 0
```

### **Day 2: price = 3**
```
release = max(0, -1 + 3 - 2) = 0
hold = max(-1, 0 - 3) = -1
```

### **Day 3: price = 2**
```
release = max(0, -1 + 2 - 2) = 0
hold = max(-1, 0 - 2) = -1
```

### **Day 4: price = 8**
```
release = max(0, -1 + 8 - 2) = 5  (sell for profit 5)
hold = max(-1, 5 - 8) = -1
```

### **Day 5: price = 4**
```
release = max(5, -1 + 4 - 2) = 5
hold = max(-1, 5 - 4) = 1  (buy at 4 using profit 5)
```

### **Day 6: price = 9**
```
release = max(5, 1 + 9 - 2) = 8  (sell for total profit 8)
hold = max(1, 8 - 9) = 1
```

**Answer:** `8`

**Transactions:**
1. Buy at 1, sell at 8: profit = $8 - 1 - 2 = 5$
2. Buy at 4, sell at 9: profit = $9 - 4 - 2 = 3$
3. Total = $5 + 3 = 8$

---

## **Why Not Use Backward Loop?**

Unlike Stock III/IV, we don't need a backward loop here because:
- We're not tracking multiple transaction counts (no array of states)
- We only have two states: `hold` and `release`
- Order of updates doesn't matter since `hold` uses `release` and vice versa, but they reference each other's **previous** values naturally

**Proof:** Let's trace both orders:

**Order 1:**
```python
release = max(release, hold + price - fee)
hold = max(hold, release - price)
```

**Order 2:**
```python
hold = max(hold, release - price)
release = max(release, hold + price - fee)
```

Both work because we're not trying to prevent "same-day chaining" like in Stock III. The fee naturally limits greedy behavior.

---

## **Complexity Analysis**

**Time Complexity:** $O(n)$  
We iterate through the prices array once.

**Space Complexity:** $O(1)$  
We only use two variables: `hold` and `release`.