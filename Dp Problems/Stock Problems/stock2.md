
leetcode URl : https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/submissions/1895781943/



"We update the states in reverse order (from the last transaction to the first)
to ensure that we are using the previous state's values from the last time step.
This prevents us from completing multiple steps of the transaction chain on the same day."

````python

class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        

        hold = float('-inf')

        release = 0


        for price in prices:

            release = max(release , hold + price)

            hold = max(hold , release - price)

        return release
        

````