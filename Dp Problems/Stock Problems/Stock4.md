# **Best Time to Buy and Sell Stock IV**

**LeetCode URL:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

## **Problem Statement**

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the `i`-th day, and an integer `k`.

Find the maximum profit you can achieve. You may complete at most `k` transactions.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example:**
```
Input: k = 2, prices = [3,2,6,5,0,3]
Output: 7
Explanation: Buy on day 2 (price = 2) and sell on day 3 (price = 6), profit = 6-2 = 4.
Then buy on day 5 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.
Total profit = 4 + 3 = 7
```

---

## **Problem Overview**

| Problem | Transaction Count | Logic |
|---------|------------------|-------|
| Stock I | K=1 | hold vs release |
| Stock II | K=∞ | cash vs hold (Forward is fine) |
| Stock III | K=2,4 | hardcoded variables (Backward or temp vars) |
| Stock IV | K=k | Array + Backward Loop |

---

## **Strategy**

### **State Machine for K Transactions**

We track two states for each transaction number:
- **`hold[i]`**: Maximum profit when holding a stock after using at most `i` transactions
- **`release[i]`**: Maximum profit when not holding a stock after completing at most `i` transactions

### **State Transitions**

For each price on day `d`:
```
release[i] = max(release[i], hold[i] + price)
hold[i] = max(hold[i], release[i-1] - price)
```

**Interpretation:**
- **Release:** Either stay released, or sell the stock we're holding
- **Hold:** Either stay holding, or buy a new stock (using profit from `i-1` transactions)

### **Why Backward Loop Order?**

We must update in **reverse order** (`k` down to `0`) to maintain the 0/1 property:

```python
for i in range(k, 0, -1):  # BACKWARD
    release[i] = max(release[i], hold[i] + price)
    hold[i] = max(hold[i], release[i-1] - price)
```

**Reason:** When updating `hold[i]`, we need `release[i-1]` from the **previous day**, not the current day. If we go forward, we'd use the already-updated `release[i-1]` from today, allowing same-day transaction chaining.

**Example:** If we updated transaction 1 first, then transaction 2 would see transaction 1's profit from today's sale, allowing:
- Buy-Sell (transaction 1) → Buy-Sell (transaction 2) **on the same day**

Backward loop prevents this by ensuring each transaction uses only yesterday's state.

## **Key Concept**

> We use a 1D DP array to save space. To maintain the 0/1 property (using each item/transaction only once per step), we must update the states in an order that ensures we are using values from the previous time-step ($i-1$) rather than values already updated in the current time-step ($i$).

---

## **Optimization: When K ≥ n/2**

If $k \geq \frac{n}{2}$ (where $n$ is the number of days), we can make as many transactions as we want! This reduces to **Stock II** (unlimited transactions).

**Why?** Each transaction requires at least 2 days (buy + sell). If $k \geq \frac{n}{2}$, we have enough transactions to capture every profitable opportunity.

**Solution:** Simply sum all positive differences:
```python
return sum(max(0, prices[i] - prices[i-1]) for i in range(1, len(prices)))
```

---

## **Solution**

```python
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        if not prices or k == 0:
            return 0 
        
        # Optimization: if k >= n/2, reduce to unlimited transactions
        if k >= len(prices) // 2:
            return sum(max(0, prices[i] - prices[i-1]) for i in range(1, len(prices)))
        
        # hold[i]: max profit when holding stock after i transactions
        # release[i]: max profit when not holding after i transactions
        hold = [-float('inf')] * (k + 1)
        release = [0] * (k + 1)
        
        for price in prices:
            # Update in REVERSE to prevent same-day transaction chaining
            for i in range(k, 0, -1):
                # Option 1: Stay released OR sell stock we're holding
                release[i] = max(release[i], hold[i] + price)
                
                # Option 2: Stay holding OR buy stock using i-1 transaction profit
                hold[i] = max(hold[i], release[i-1] - price)
        
        return release[k]
```

---

## **Why `hold = [-float('inf')]`?**

Initializing `hold[i] = -inf` means "impossible to hold without making a purchase." 

On the first price, when we do:
```python
hold[i] = max(-inf, release[i-1] - price)
```

We force the algorithm to actually **buy** the stock, setting `hold[i] = -price`.

---

## **Example Walkthrough**

**Input:** `k = 2`, `prices = [3, 2, 6, 5, 0, 3]`

### **Initial State**
```
hold = [-inf, -inf, -inf]
release = [0, 0, 0]
```

### **Day 1: price = 3**
```
After updates (i=2, then i=1):
hold = [-inf, -3, -inf]    # Can buy 1 stock
release = [0, 0, 0]
```

### **Day 2: price = 2**
```
hold = [-inf, -2, -inf]    # Better to buy at 2
release = [0, 0, 0]
```

### **Day 3: price = 6**
```
hold = [-inf, -2, -6]      # Can start transaction 2
release = [0, 4, 4]        # Sell transaction 1: profit = 4
```

### **Day 5: price = 0**
```
hold = [-inf, 0, 4]        # Buy for transaction 2 using profit 4
release = [0, 4, 4]
```

### **Day 6: price = 3**
```
hold = [-inf, 0, 4]
release = [0, 4, 7]        # Sell transaction 2: total profit = 7
```

**Answer:** `7`

---

## **Complexity Analysis**

**Time Complexity:** $O(n \times k)$  
Where $n$ is the number of days. We iterate through each price and for each price, we iterate through all $k$ transactions.

**Space Complexity:** $O(k)$  
We use two arrays of size $k + 1$ to store the `hold` and `release` states.