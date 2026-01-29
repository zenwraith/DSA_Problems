
# **Best Time to Buy and Sell Stock with Cooldown**

**LeetCode Link:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

---

## **Problem Statement**

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i`-th day.

Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions:

- After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day).

**Example:**
```
Input: prices = [1,2,3,0,2]
Output: 3
Explanation: Buy on day 1 (price = 1), sell on day 2 (price = 2), profit = 1.
Then wait (cooldown on day 3), buy on day 4 (price = 0), sell on day 5 (price = 2), profit = 2.
Total profit = 1 + 2 = 3.
```

---

## **Strategy: Three-State Finite State Machine**

We track three distinct states:
1. **`hold`**: Currently holding a stock
2. **`sold`**: Just sold a stock today (entering cooldown)
3. **`rest`**: Resting (not holding, cooldown period over)

### **State Transitions**

```
hold → sold   (sell stock)
hold → hold   (keep holding)
sold → rest   (mandatory cooldown)
rest → rest   (stay resting)
rest → hold   (buy stock)
```

**Visual Diagram:**
```
    ┌──────────────┐
    │     HOLD     │────sell────►┌──────────────┐
    │  (has stock) │             │     SOLD     │
    └──────────────┘             │ (just sold)  │
         ▲                       └──────────────┘
         │                              │
        buy                        cooldown
         │                              │
         │                              ▼
    ┌──────────────┐              ┌──────────────┐
    │     REST     │◄─────────────│   (next day) │
    │ (can buy)    │              └──────────────┘
    └──────────────┘
```

---

## **Algorithm Steps**

### **Step 1: Capture 'Yesterday'**
```
prev_hold = hold
prev_sold = sold
prev_rest = rest
```

### **Step 2: Update 'Today' using only 'Yesterday'**

- **Door 1 - Buying:** You can only buy if you were RESTING yesterday
- **Door 2 - Selling:** You can only sell if you were HOLDING yesterday
- **Door 3 - Cooling Down / Resting:** You are free to buy if you were resting OR if you just finished selling

## **Three "Room" Analogy**

Imagine three rooms in a building. You can only move between them following specific doors:

- **Room 1: HOLDING** - You have a stock.
- **Room 2: SOLD** - You just sold a stock; you are currently standing at the cash register.
- **Room 3: RESTING** - You are empty-handed and allowed to buy.

## **Logic of the Doors**

- **To be in SOLD today:** You must have come from HOLDING yesterday.
- **To be in RESTING today:** You either were already RESTING yesterday, or you just finished your 1-day "penalty" from being in the SOLD room yesterday.
- **To be in HOLDING today:** You either stayed in the room from yesterday, or you entered from the RESTING room (you cannot enter from the SOLD room because of the cooldown!).

## **Solution**

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0
        
        # Initialize states
        # hold: max profit when holding a stock
        # sold: max profit when just sold (entering cooldown)
        # rest: max profit when resting (can buy)
        hold = -float('inf')  # Impossible to hold without buying first
        sold = 0
        rest = 0
        
        for price in prices:
            # Capture yesterday's values before updating
            prev_hold = hold
            prev_sold = sold
            prev_rest = rest
            
            # Update HOLD: stay holding OR buy from rest state
            # Can only buy if we were resting (not in cooldown)
            hold = max(prev_hold, prev_rest - price)
            
            # Update SOLD: must have held yesterday and sell today
            sold = prev_hold + price
            
            # Update REST: stay resting OR cooldown ends (from sold yesterday)
            rest = max(prev_rest, prev_sold)
        
        # Maximum profit is when we're not holding stock (sold or rest)
        return max(sold, rest)
```

---

## **Why Use `prev_state` Variables?**

**Critical Issue:** All three states depend on each other's **previous day** values.

If we update `hold` first without saving `prev_hold`, then when we calculate `sold = hold + price`, we'd use **today's** `hold` value instead of yesterday's!

**The Pattern:**
```python
# Save yesterday's snapshot
prev_hold, prev_sold, prev_rest = hold, sold, rest

# Now update today using only yesterday's values
hold = max(prev_hold, prev_rest - price)
sold = prev_hold + price
rest = max(prev_rest, prev_sold)
```

This guarantees we're implementing:
$$
\text{state}[\text{today}] = f(\text{states}[\text{yesterday}])
$$

---

## **Example Walkthrough**

**Input:** `prices = [1, 2, 3, 0, 2]`

### **Initial State**
```
hold = -inf, sold = 0, rest = 0
```

### **Day 1: price = 1**
```
hold = max(-inf, 0 - 1) = -1  (buy)
sold = -inf + 1 = -inf
rest = max(0, 0) = 0
```

### **Day 2: price = 2**
```
hold = max(-1, 0 - 2) = -1
sold = -1 + 2 = 1  (sell for profit 1)
rest = max(0, 0) = 0
```

### **Day 3: price = 3**
```
hold = max(-1, 0 - 3) = -1
sold = -1 + 3 = 2
rest = max(0, 1) = 1  (cooldown ends, profit = 1 saved)
```

### **Day 4: price = 0**
```
hold = max(-1, 1 - 0) = 1  (buy using profit 1)
sold = -1 + 0 = -1
rest = max(1, 2) = 2
```

### **Day 5: price = 2**
```
hold = max(1, 2 - 2) = 1
sold = 1 + 2 = 3  (sell for total profit 3)
rest = max(2, -1) = 2
```

**Answer:** `max(sold, rest) = max(3, 2) = 3`

---

## **Important Tip**

> **Whenever you see "Cooldown" or "Transaction Fee," stop trying to use the 1D array "Backward" trick.** It's too easy to make a mistake. Always use the `prev_state` variables. It makes the code 100% readable and prevents the computer from "teleporting" you between states on the same day.

---

## **Complexity Analysis**

**Time Complexity:** $O(n)$  
We iterate through the prices array once, performing constant-time operations for each price.

**Space Complexity:** $O(1)$  
We only use a fixed number of variables (hold, sold, rest, and their previous versions).