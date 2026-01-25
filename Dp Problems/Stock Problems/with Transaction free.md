

leetcode link : https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/



Problem,    Transaction Rule,   Main DP Transition
Stock I,    Only 1 trade,       "hold = max(hold, -price)"
Stock II,   Infinite trades,       "hold = max(hold, cash - price)"
Stock III/IV,Exactly K trades,  "hold[k] = max(hold[k], release[k-1] - price)"
Cooldown,   1-day wait,            "hold = max(hold, rest - price)"
Fee,        Pay per trade,              "cash = max(cash, hold + price - fee)"

``````python

class Solution:
    def maxProfit(self, prices: List[int], fee: int) -> int:

        hold = - prices[0]
        release = 0

        for i in range(1, len(prices)):

            release = max( release , hold + prices[i] - fee)

            hold = max(hold , release-prices[i])

        return release 
        
``````