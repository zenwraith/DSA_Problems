

Leetcode URL : https://leetcode.com/problems/best-time-to-buy-and-sell-stock/

````python

class Solution:
    def maxProfit(self, prices: List[int]) -> int:

        # Base Case: We start with no money and haven't bought anything
        hold = -float('inf')
        release = 0

        for price in prices:
            # Update release: Max of (doing nothing) or (selling the stock we hold)
            release = max( release , hold + price)
            
            # Update hold: Max of (doing nothing) or (buying today's stock)
            # Note: -price because we are spending money to buy

            hold = max(hold , -price)

        return release
        

````