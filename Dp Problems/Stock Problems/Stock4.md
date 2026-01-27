# **Best Time to Buy and Sell Stock IV**

**LeetCode URL:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

## **Problem Overview**

| Problem | Transaction Count | Logic |
|---------|------------------|-------|
| Stock I | K=1 | hold vs release |
| Stock II | K=∞ | cash vs hold (Forward is fine) |
| Stock III | K=2,4 | hardcoded variables (Backward or temp vars) |
| Stock IV | K=k | Array + Backward Loop |

## **Key Concept**

> We use a 1D DP array to save space. To maintain the 0/1 property (using each item/transaction only once per step), we must update the states in an order that ensures we are using values from the previous time-step ($i-1$) rather than values already updated in the current time-step ($i$).


````python


class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:

        if not prices or k == 0:
            return 0 

        
        if k >= len(prices) // 2:
            return sum( max(0, prices[i] - prices[i-1]) for i in range(1, len(prices)))

        
        hold = [ - float('inf') ]*(k+1)

        release = [0] *(k+1)

        for price in prices:

            for i in range(k,-0,-1):

                release[i] = max( release[i] , hold[i] + price)

                hold[i] = max( hold[i] , release[i-1] - price)

        return release[k]
        
````