
# **Best Time to Buy and Sell Stock III**

**LeetCode URL:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)

## **Key Concept**

The only reason `release1` was first in the previous code was because it followed the "chronological order" of a single transaction. But when you have multiple transactions (like Stock III or IV), the **Backward order** is the one that prevents the "Unbounded" logic (using the same day's price for multiple steps of the chain).


````python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:

        hold1 = hold2 = float('-inf')

        release1 = release2 = 0

        for price in prices:

            release2 = max(release2 , hold2 + price)

            hold2 = max(hold2, release1-price)

            release1 = max(release1 , hold1 + price)

            hold1 = max(hold1, -price)

            

        return release2
        
````