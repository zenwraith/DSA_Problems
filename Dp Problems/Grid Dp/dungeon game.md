

leetcode link : https://leetcode.com/problems/dungeon-game/



The Strategy: Why we work BackwardIf you start from the top-left, you don't know how much health you'll need later. But if you start from the bottom-right (the Princess), you can calculate exactly how much health you need to survive that specific cell and make it to the end.The Rule: Your health must always be at least 1.Let $dp[r][c]$ be the minimum health needed before entering cell $(r, c)$.

At any cell, you want to pick the path (Down or Right) that requires the least amount of health.Calculate the health needed to survive the next step: next_health = min(dp[right], dp[down]).Subtract the current cell's value (if it's a health boost, you need less; if it's a demon, you need more): needed = next_health - grid[r][c].The Floor: If the cell gives you so much health that needed becomes $\le 0$, you still need at least 1 health to exist: dp[c] = max(1, needed).

```python
class Solution:
    def calculateMinimumHP(self, dungeon: List[List[int]]) -> int:

        n,m = len(dungeon) , len(dungeon[0])
        
        # dp[r][c] = min health needed to ENTER cell (r, c)
        # We add an extra row and column of infinity to handle edges
        dp = [ [ float('inf') ] * (m+1) for i in range(n+1)]

        dp[n][m-1] = dp[n-1][m] = 1

        for r in range(n-1,-1,-1):
            for c in range(m-1, -1,-1):
                # 1. Which next room is easier to enter?
                next_health_req = min(dp[r+1][c] , dp[r][c+1])

                # 2. How much health do I need before entering THIS room?
                 # If room is -10, I need (next_health + 10)
                # If room is +10, I need (next_health - 10)
                entrance_health = next_health_req - dungeon[r][c]
                
                # 3. You must always enter with at least 1 HP
                dp[r][c] = max( 1, entrance_health)

        return dp[0][0]

    
```


 The Step-by-Step LogicFor any cell, we look at the rooms to its Right and Down. We want to pick the "easier" room (the one that requires less entrance health).Look Ahead: next_step = min(Health_Needed_For_Right, Health_Needed_For_Down)Calculate Entrance Health: needed = next_step - current_room_valueThe "Safety" Check: * If the room has a demon (-10), needed becomes larger (e.g., $5 - (-10) = 15$).If the room has a huge potion (+100), needed might become negative (e.g., $5 - 100 = -95$).But wait! You can't enter a room with negative health; you'd be dead. So, if needed is 0 or less, you still must have at least 1 HP to be alive.final_needed = max(1, needed)


 Why this works
By the time you reach dp[0][0], the value represents the exact minimum health you must carry when you step into the first room so that no matter which path you take, you can satisfy the "Must stay above 0 HP" condition for the rest of the journey.

Summary of logic:
Demon (-): Increases the health you need to enter.

Potion (+): Decreases the health you need to enter (but not below 1).

