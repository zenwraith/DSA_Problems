

# **Best Time to Buy and Sell Stock I**

**LeetCode URL:** [https://leetcode.com/problems/best-time-to-buy-and-sell-stock/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

```python

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
        

```


Since you start with $0, the moment you buy a stock for price, your balance becomes -price. You are doing max(hold, -price) to find the cheapest day to buy the stock.

Why use max for a negative number?
It feels weird to use max when you want the "cheapest" price, but remember: we are tracking your bank account balance.

If Stock A costs $10, your balance is -$10.

If Stock B costs $2, your balance is -$2.

Mathematically, -2 is greater than -10. So, max(-10, -2) returns -2. By using max, you are automatically picking the price that leaves you with the most money (the least debt) in your pocket.


Step-by-Step LogicImagine the prices are [10, 2, 5, 8]:Day 1 (Price 10):hold = max(-inf, -10) $\rightarrow$ -10.You spent $10. Your balance is negative 10.Day 2 (Price 2):hold = max(-10, -2) $\rightarrow$ -2.You "reset" your buy day to Day 2 because spending $2 is much better than having spent $10. You now have more money left in your account.Day 3 (Price 5):hold = max(-2, -5) $\rightarrow$ -2.You do nothing. Why buy for $5 when you already "held" it at $2?The "Holding" RuleThe variable hold is essentially asking: "What is the best financial state I can be in if I am currently forced to own a stock?" The answer is always: The state where I spent the least amount of money to get it.