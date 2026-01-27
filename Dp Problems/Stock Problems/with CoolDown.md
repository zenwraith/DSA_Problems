
# **Best Time to Buy and Sell Stock with Cooldown**

**LeetCode Link:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

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

```python

class Solution:
    def maxProfit(self, prices: List[int]) -> int:


        if not prices:
            return 0

        
        hold = -float('inf')
        sold = 0
        rest = 0

        for price in prices:
            # We need yesterday's values to update today's
            prev_hold = hold
            prev_sold = sold
            prev_rest = rest


            # 1. Update HOLD: 
        # Stay holding OR Buy today (must come from rest state)
            hold = max(prev_hold, prev_rest- price)

            # 2. Update SOLD: 
        # Must have held yesterday and sold at today's price

            sold = prev_hold + price

            # 3. Update REST: 
        # Stay resting OR just finished cooldown from yesterday's sale
            rest = max(prev_rest, prev_sold)

        # Your final profit is the max of being in a 'sold' or 'rest' state
    # (Because you can't end the game while still holding a stock)
        return max(sold, rest)

```


## **Important Tip**

> **Whenever you see "Cooldown" or "Transaction Fee," stop trying to use the 1D array "Backward" trick.** It's too easy to make a mistake. Always use the `prev_state` variables. It makes the code 100% readable and prevents the computer from "teleporting" you between states on the same day.